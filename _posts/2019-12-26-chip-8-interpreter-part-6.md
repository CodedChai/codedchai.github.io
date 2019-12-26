---
layout: post
title: "Tackling a CHIP-8 Interpreter, Part 6"
summary: "How to execute code using our opcode"
featured-img: chip8init
---

# How are we going to use our opcode?

A giant switch statement. I'm not even joking. Sure, it's a little weird to see and most production environments really try to avoid these but when it comes to writing an interpreter, this is the way to go. Trust me, I've looked into doing a more OO way for this but it all requires more work, more overhead and can be much messier. So we'll stick with the age old tradition of a giant freaking switch statement. 

Now that we have that out of the way...

# What do our cases look like?

Honestly? They look a little strange. Here's an example:

<code>
    switch ( opcode & 0xF000 ) {
        case 0x0000:
            switch ( opcode & 0x000F ) {
                case 0x0000:
                    cls(); /* 00E0 */
                    break;
                
                case 0x000E:
                    ret(); /* 00EE */
                    break;
                ...

Now that's pretty, isn't it? Besides the greatness of how it looks, what are we even doing? Well if you paid attention to when we implemented our opcodes you would remember that we had things like _x, y, n, kk, and nnn_ all to represent an arbitrary hex value. Instead of writing a case for literally every single scenario from 0x0000 - 0xFFFF we will be using masking to account for the scenarios where we don't know what those arbitrary values are. We'll start by masking just the first hex value, you can see this with ```opcode & 0xF000```. Since we know we can separate most opcodes with this first value this is an extremely helpful mask. It means we can make cases where we only care to search for that first value. 

To help this make sense let me walk through my above example where we are looking for 0x00E0 to run cls().

First we will mask the opcode so we only look at the first value. We can do this by doing a bitwise and where the first hex value is F and all of the other hex values are 0. The 0s will remove the lower 3 hex values and the F will keep the upper hex value. From here we can look for the case that we want. Since we're looking at the first hex value only let's look for the case where the first hex value is 0. We can easily do this since we know the last three values are going to be 0 by just making the first value, in this case, also 0. So we'll look for case 0x0000. Now we know that we have 2 opcodes that start with 0, 0x00E0 and 0x00EE. To choose which of these two we would like to run we be looking at the last hex value since that is what differs here. To do this we will simply bitwise & the opcode with 0x000F. Now that we have done that we can call the correct method and break out of the switch statement. 

# Below is the beautiful code

<code>

	private void executeOpcode() throws Exception {

		switch ( opcode & 0xF000 ) {
			/* Begin case 0x0000 */
			case 0x0000:
				switch ( opcode & 0x000F ) {
					case 0x0000:
						cls(); /* 00E0 */
						break;
					case 0x000E:
						ret(); /* 00EE */
						break;
					default:
						throw new Exception( "Unknown opcode: " + opcode + " in [0x0000]" );
				}
				break;
			/* End case 0x0000 */

			/* Begin case 0x1000 */
			case 0x1000:
				jmp(); /* 0x1NNN */
				break;
			/* End case 0x1000 */

			/* Begin case 0x2000 */
			case 0x2000:
				call(); /* 0x2NNN */
				break;
			/* End case 0x2000 */

			/* Begin case 0x3000 */
			case 0x3000:
				SEVxIsKK(); /* 3xkk */
				break;
			/* End case 0x3000 */

			/* Begin case 0x4000 */
			case 0x4000:
				SNEVxIsNotKK(); /* 4xkk */
				break;
			/* End case 0x4000 */

			/* Begin case 0x5000 */
			case 0x5000:
				SEVxIsVy(); /* 5xy0 */
				break;
			/* End case 0x5000 */

			/* Begin case 0x6000 */
			case 0x6000:
				loadKKToVx(); /* 6xkk */
				break;
			/* End case 0x6000 */

			/* Begin case 0x7000 */
			case 0x7000:
				addVxKK(); /* 7xkk */
				break;
			/* End case 0x7000 */

			/* Begin case 0x6000 */
			case 0x8000:

				switch ( opcode & 0x000F ) {
					case 0x0000:
						loadVxVy(); /* 8xy0*/
						break;

					case 0x0001:
						orVxVy(); /* 8xy1 */
						break;

					case 0x0002:
						andVxVy(); /* 8xy2 */
						break;

					case 0x0003:
						xorVxVy(); /* 8xy3 */
						break;

					case 0x0004:
						addVxVy(); /* 8xy4 */
						break;

					case 0x0005:
						subVxVy(); /* 8xy5 */
						break;

					case 0x0006:
						shiftRightVx(); /* 8xy6 */
						break;

					case 0x0007:
						subnVxVy(); /* 8xy7 */
						break;

					case 0x000E:
						shiftLeftVx(); /* 8xyE */
						break;

					default:
						throw new Exception( "Unknown opcode: " + opcode + " in [0x8000]" );
				}
				break;
			/* End case 0x8000 */

			/* Begin case 0x9000 */
			case 0x9000:
				SNEVxIsNotVy(); /* 9xy0 */
				break;
			/* End case 0x9000 */

			/* Begin case 0xA000 */
			case 0xA000:
				loadINNN(); /* Annn */
				break;
			/* End case 0xA000 */

			/* Begin case 0xB000 */
			case 0xB000:
				jumpNNNV0(); /* Bnnn */
				break;
			/* End case 0xB000 */

			/* Begin case 0xC000 */
			case 0xC000:
				randomVxKK(); /* Cxkk */
				break;
			/* End case 0xC000 */

			/* Begin case 0xD000 */
			case 0xD000:
				displayVxVyN(); /* Dxyn */
				break;
			/* End case 0xD000 */

			/* Begin case 0xE000 */
			case 0xE000:
				switch ( opcode & 0x00FF ) {
					case 0x009E:
						skipKeyPressed(); /* Ex9E */
						break;

					case 0x00A1:
						skipKeyReleased(); /* ExA1 */
						break;
					default:
						throw new Exception( "Unknown opcode: " + opcode + " in [0xE000]" );
				}
				displayVxVyN(); /* Dxyn */
				break;
			/* End case 0xE000 */

			/* Begin case 0xF000 */
			case 0xF000:
				switch ( opcode & 0x00FF ) {
					case 0x0007:
						loadVxDisplayTimer(); /* Fx07 */
						break;

					case 0x000A:
						loadKeyPress(); /* Fx0A */
						break;

					case 0x0015:
						loadDelayTimer(); /* Fx15 */
						break;

					case 0x0018:
						loadSoundTimer(); /* Fx18 */
						break;

					case 0x001E:
						addIVx(); /* Fx1E */
						break;

					case 0x0029:
						setFontLocationInI(); /* Fx29 */
						break;

					case 0x0033:
						setBCD(); /* Fx33 */
						break;

					case 0x0055:
						storeRegisters(); /* Fx55 */
						break;

					case 0x0065:
						loadRegisters(); /* Fx65 */
						break;

					default:
						throw new Exception( "Unknown opcode: " + opcode + " in [0xF000]" );
				}
				/* End case 0xF000 */

			default:
				throw new Exception( "Unknown opcode: " + opcode );
		}

	}

# On the Brightside...

You can now actually run your code!!!! You won't be seeing any output and you won't be able to interact but you've technically finished the interpreter part. Next up we'll discuss displaying our pixels on the screen!