---
layout: post
title: "Tackling a CHIP-8 Interpreter, Part 3"
summary: "Accurately Emulating CPU Cycles"
featured-img: genericcpuchip8
---

# This is Where the Fun Begins

As I hinted at in the last post we are going to get to work on emulating the CPU cycles. This wasn't specified in part 0, because Chip-8 doesn't have a specified frequency, but we will be targeting 500 Hz. We will be emulating in a [separate thread](https://www.youtube.com/watch?v=KAHLwAxS7FI) from our rendering thread. I will assume working knowledge of threads so I will lightly explain what is going on. 

# The Basic Design

Since we will be making this it's own thread we will be implementing runnable which means that we need to have a run method. 
* run() will be used as our initialization. It will do anything on start up.
* update() will be ran at 100Hz (more detail below) and will be our main loop for emulating CPU cycles.
* emulateCycle() will be used to essentially grab the next opcode and then tell us to compute it. We may need this for other things, not quite sure yet and I'd rather start with it than not.
* executeOpcode() will execute the opcode. This is where most of the magic will happen down the line.

# Tick Tock Tick Tock

Chip-8 is forgiving in that it doesn't have a set frequency that we are required to emulate at. This means we don't have to worry about a ton of annoying timing stuff. As a note I will be using the Java Duration and Instant features to make my life easier and ensure better accuracy so make sure you're on Java 9 or later. 

If you followed my hint link in the last post you would see it took you here: <https://github.com/AfBu/haxe-chip-8-emulator/wiki/(Super)CHIP-8-Secrets>. See, we will want to ensure that we properly hit the set frequency. In order to do this we will be running all of our operations in set batches. We will calculate how many opcodes to run by using the delta time (in seconds) multiplied by the frequency. If you have ever done game programming you would recognize delta time as that is the common name used for the time between frames. Except instead of computing the time between frames we will be computing the time since we last ran calculations. 

This will give us:

<code>

	Instant currentComputeTime = Instant.now();
	double deltaTime = (double) Duration.between( timeOfLastCompute, currentComputeTime ).toNanos() / 1000000000;

	timeOfLastCompute = Instant.now();

	for ( int i = 0; i < Math.round( deltaTime * FREQUENCY ); i++ ) {
		emulateCycle();
	}

Now we can properly compute how many emulated CPU cycles we will need to calculate in our batch. There's only one problem.. Modern computers are EXTREMELY fast. Like they're insane. To the point where we are going to have to slow down our code in order to even properly batch these out. Since our lovely friend Petr Kratina found that 100Hz (see link above) is a good pace to run this code that's also what we'll be using. We can easily achieve this with a single line of code:

<code>

	TimeUnit.MILLISECONDS.sleep( 10 );

I use TimeUnit instea of Thread.Sleep() for readability and you should too. Soapboxing aside everything else is pretty much just boiler plate set up in preparation for the future.

# Here is the Current Class

<code>

	package com.codedchai.chip8;

	import java.time.Duration;
	import java.time.Instant;
	import java.util.concurrent.TimeUnit;

	public class Emulator implements Runnable {

		private static final int FREQUENCY = 500;

		Instant timeOfLastCompute = null;

		@Override
		public void run() {
			timeOfLastCompute = Instant.now();
			try {
				update();
			} catch ( Exception e ) {
				e.printStackTrace();
			}
		}

		/*
		Our main loop
		 */
		private void update() throws Exception {

			while ( true ) {
				Instant currentComputeTime = Instant.now();
				double deltaTime = (double) Duration.between( timeOfLastCompute, currentComputeTime ).toNanos() / 1000000000;

				System.out.println( Math.round( deltaTime * FREQUENCY ) );

				timeOfLastCompute = Instant.now();

				for ( int i = 0; i < Math.round( deltaTime * FREQUENCY ); i++ ) {
					emulateCycle();
				}

				TimeUnit.MILLISECONDS.sleep( 10 );
			}
		}

		/*
		This is each step of our processor, basically each tick
		 */
		private void emulateCycle() throws Exception {
			// Get opcode
			executeOpcode((byte) 0 );
		}

		/*
		Execute our opcode
		 */
		private void executeOpcode(byte opcode) throws Exception {
		}
	}
