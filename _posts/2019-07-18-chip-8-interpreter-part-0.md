---
layout: post
title: "Tackling a CHIP-8 Interpreter, Part 0"
summary: "Going through a basic outline of what the CHIP-8 specification is and what my future posts will be covering"
featured-img: chip8
---

# What is CHIP-8?

CHIP-8 is an interpreted programming language created by Joseph Weisbecker back in the 70s. It was initially created as an easier way to program video games. However for our case we will be looking to get a better understanding of how an emulator works. CHIP-8 is often treated as an intro into emulators since it shares a lot of the same concepts but is much simpler. For example, there are only 35 opcodes needed to implement CHIP-8. Which leads us to the next section.

# Specifications

I mostly used the Wikipedia page to gather the information on this for reference

## Memory

We will have 4096 bytes of memory available to us. This sounds like a true 4k system to me! Since this is a modern implementation of CHIP-8 we will only need to reserve memory for the video out data and the call stack. This means we will reserve the highest 256 bytes (0xF00-0xFFFF) for our display information and the 96 bytes below that to the call stack information. 

## Registers

There is a whopping total of 16 8-bit registers for CHIP-8 to use. The registers are named from V0-VF. VF is typically used as a flag which we will cover better later on. Essentially this means we want to avoid writing to it if necessary.

## Call Stack

The stack is used to store return addresses when when subroutines are called. It seems like systems will typically have 48 bytes reserved specifically for this.

## Timers

We will only have to worry about 2 times which both luckily operate at the same frequency of 60 hertz. The first timer is the *Delay Timer* which is used for timing events in games. This timer can be set or read at any point but needs to be increments at 60Hz. The second timer is the *Sound Timer*. This timer is going to be set to play a rather annoying beeping sound while the timer is greater than zero. 

## Input

Input is typically handled by a "hex keyboard" which basically just means the only valid inputs are 0-F. Not very ergonomic but it can definitely simplify coding. We will have three opcodes dedicated to detect input. One of them will skip an instruction if a specified key is pressed, another one will skip an instruction if a specified key is not pressed and the final one will store the key value into a register for use later.

## Graphics

We will be displaying a 64x32 monochrome image. Graphics are completely handled by drawing sprites which are always 8 pixels wide but can vary between 1 and 15 pixels high. We will be XORing these sprite pixels with what is already in the display registers to display the images. This basically means that if a sprite is set we will be flipping the pixel from either on to off or vice versa. Note that we'll have to set the VF flag register if any bits change from being set to unset as this is how games handle collision detection.

## Sound

We'll make an annoying beep when our high tech Sound Timer's value is greater than 0.

## Opcodes

There are 35 opcodes all of which are two bytes long and stored in big-endian notation. That means the most significant bit is first. We will be covering this in much greater detail down the road.

# On the Next Episode

I will be tackling this using Java but you can follow along in any language you see fit. We will first write the disassembler to read through the ROMs. From there we will look at implementing the registers, timers and memory. Shortly after that we will implement the specific opcode logic. After that we will work on displaying everything to the screen. 
