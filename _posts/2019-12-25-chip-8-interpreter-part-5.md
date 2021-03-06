---
layout: post
title: "Tackling a CHIP-8 Interpreter, Part 5"
summary: "Opcodes Opcodes and More Opcodes"
featured-img: chip8opcodes
---

# What is an Opcode?

Okay so there are 35 opcodes for Chip-8. You may be asking, what the hell is an [opcode](https://en.wikipedia.org/wiki/Opcode)? Well basically it's just an instruction for the CPU. It will operate on the memory, timers, registers, call stack, etc. In a normal emulator each opcode has a specific number of cycles it takes to run, in Chip-8 every opcode only takes one cycle. That means that every opcode will be treated equal in the sense that they all take 1 CPU cycle. 

All opcodes are 2 bytes long and are stored big endian. Something to note is that in memory each first byte of each instruction should be stored in an even address. I will be using the same verbiage as [other Chip-8 tutorials](http://devernay.free.fr/hacks/chip8/C8TECH10.HTM) to help describe the opcodes. The verbiage is below:

*nnn or addr* - 12 bit value, specifically the lower 12 bits of the opcode.
*kk* - 8 bit value, specifically the lowest 8 bits of the opcode.
*x* - 4 bit value, specifically the lower 4 bits of the high byte of the opcode. That means the lower 4 bits of the first byte.
*y* - 4 bit value, specifically the upper 4 bits of the low byte of the opcode. That means the higher 4 bits of the second byte.
*n* - 4 bit value, specifically the lowest 4 bits of the opcode.

To make this even easier I used the following methods to get these values.

<code>

    short getNNN() {
		return (short) (opcode & 0x0FFF);
	}

	byte getKK() {
		return (byte) (opcode & 0x00FF);
	}

	byte getX() {
		return (byte) ((opcode & 0x0F00) >> 8);
	}

	byte getY() {
		return (byte) ((opcode & 0x00F0) >> 4);
	}

    byte getN() {
		return (byte) (opcode & 0x00F);
	}



# WARNING! FUN TIMES AHEAD!

## How do we get our opcode?

Remember, an opcode is 2 bytes of data that (should, we won't add any validation) start at an even numbered memory location. So to get our opcode we will have to bit shift the first byte over by 8 bits and then bitwise or the second byte. We will also have to mask the bytes with hex values to ensure that we don't delete any data and cause issues.

<code>

	opcode = (((memory[programCounter] & 0xFFFF) << 8)) | (memory[programCounter + 1] & 0xFF);


We will be grabbing the first byte at the first memory location, bit shifting it left 8 bits and then filling in the final 8 bits with the second byte which is at the memory location right after the program counter.

## How should I interpret the next part?

Here are all of the opcodes in Java Code form. They include comments that describe what they do along with their standard instruction name. The comments will be laid out like :

> Opcode Value - Opcode Name
> Description of Opcode

You'll notice that most of these opcodes end with *programCounter += 2;*. This is because we have to know what our next instruction is going to be. In most cases we will increment this by 2, for 2 bytes. 

<code>

	/*
	00E0 - Clear Screen
	Reset all pixels to 0, set drawFlag to true so we know that pixels were updated
	 */
	private void cls() {
		for ( int i = 0; i < pixels.length; i++ ) {
			pixels[i] = 0;
		}
		drawFlag = true;
		programCounter += 2;
	}

	/*
	00EE - Return from a subroutine
	We will set the program counter to the address at the top of the stack, then subtract 1 from the stack pointer
	 */
	private void ret() {
		programCounter = callStack[stackPointer];
		stackPointer--;
	}

	/*
	1nnn - Jump to address
	We will set the program counter to address nnn, we will do this by masking the first bit in the opcode
	 */
	private void jmp() {
		programCounter = getNNN();
	}

	/*
	2nnn - Call address
	We will call the subroutine at address nnn. We will increment the stack pointer, put the current program counter on
	the top of the stack, and then set the program counter to nnn
	 */
	private void call() {
		stackPointer++;
		callStack[stackPointer] = programCounter;
		programCounter = getNNN();
	}

	/*
	3xkk - SE Vx, kk
	We will skip the next opcode if Vx is equal to kk. If Vx == kk then increment the program counter by 4 (to skip the next
	opcode) otherwise we will increment the program counter by 2
	 */
	private void SEVxIsKK() {
		if ( vRegisters[(opcode & 0x0F00) >> 8] == (opcode & 0x00FF) ) {
			programCounter += 4;
		} else {
			programCounter += 2;
		}
	}

	/*
	4xkk - SNE Vx, kk
	We will skip the next opcode if Vx is not equal to kk. If Vx != kk then increment the program counter by 4 (to skip the next
	opcode) otherwise we will increment the program counter by 2
	 */
	private void SNEVxIsNotKK() {
		if ( vRegisters[getX()] != (getKK()) ) {
			programCounter += 4;
		} else {
			programCounter += 2;
		}
	}

	/*
	5xy0 - SE vx, vy
	We will skip the next opcode if Vx is equal to Vy. If Vx == Vy then increment the program counter by 4 (to skip the next
	opcode) otherwise we will increment the program counter by 2
	 */
	private void SEVxIsVy() {
		if ( vRegisters[getX()] == (vRegisters[getY()]) ) {
			programCounter += 4;
		} else {
			programCounter += 2;
		}
	}

	/*
	6xkk - LD Vx, kk
	Load the value kk into Vx
	 */
	private void loadKKToVx() {
		vRegisters[getX()] = getKK();
		programCounter += 2;
	}

	/*
	7xkk - Add Vx, kk
	Add kk to the value in Vx and store in Vx
	 */
	private void addVxKK() {
		// TODO: Figure out if we need to set the carry
		vRegisters[getX()] = (byte) (vRegisters[getX()] + getKK());
		programCounter += 2;
	}

	/*
	8xy0 - LD Vx, Vy
	Load the value Vy into the Vx register
	 */
	private void loadVxVy() {
		vRegisters[getX()] = vRegisters[getY()];
		programCounter += 2;
	}

	/*
	8xy1 - OR Vx, Vy
	Set Vx to the bitwise OR of the values of Vx and Vy
	 */
	private void orVxVy() {
		vRegisters[getX()] = (byte) (vRegisters[getX()] | vRegisters[getY()]);
		programCounter += 2;
	}

	/*
	8xy2 - AND Vx, Vy
	Set Vx to the bitwise AND of the values of Vx and Vy
	 */
	private void andVxVy() {
		vRegisters[getX()] = (byte) (vRegisters[getX()] & vRegisters[getY()]);
		programCounter += 2;
	}

	/*
	8xy3 - XOR Vx, Vy
	Set Vx to the bitwise XOR of the values of Vx and Vy
	 */
	private void xorVxVy() {
		vRegisters[getX()] = (byte) (vRegisters[getX()] ^ vRegisters[getY()]);
		programCounter += 2;
	}

	/*
	8xy4 - Add Vx, Vy
	Add Vx and Vy together. If the value is greater than a byte (>255) VF is set to 1, otherwise VF is set to 0. The lowest 8 bits are kept and stored in Vx.
	 */
	private void addVxVy() {
		int sum = vRegisters[getX()] + vRegisters[getY()];
		if ( sum > 255 ) {
			vRegisters[0x0F] = 1;
			sum -= 255;
		} else {
			vRegisters[0x0F] = 0;
		}
		vRegisters[getX()] = (byte) (sum);
		programCounter += 2;
	}

	/*
	8xy5 - SUB Vx, Vy
	If Vx > Vy then VF is set to 1, otherwise it is set to 0. Then Vy is subtracted from Vx and stored in Vx.
	*/
	private void subVxVy() {
		if ( vRegisters[getX()] > vRegisters[getY()] ) {
			vRegisters[0x0F] = 1;
		} else {
			vRegisters[0x0F] = 0;
		}
		vRegisters[getX()] = (byte) (vRegisters[getX()] - vRegisters[getY()]);
		programCounter += 2;
	}

	/*
	8xy6 - SHR Vx
	If the least significant bit is 1 then set VF to 1, otherwise set VF to 0. Then Vx is shifted right once (divided by two).
	*/
	private void shiftRightVx() {
		vRegisters[0xF] = (byte) (vRegisters[getX()] & 0x1); // Set based on LSB
		vRegisters[getX()] >>= 1;
		programCounter += 2;
	}

	/*
	8xy7 - SUBN Vx, Vy
	If Vy > Vx then set VF to 1, otherwise set to 0. Then Vx is subtracted from Vy and the results are stored in Vx.
	 */
	private void subnVxVy() {
		if ( vRegisters[getY()] > vRegisters[getX()] ) {
			vRegisters[0x0F] = 1;
		} else {
			vRegisters[0x0F] = 0;
		}
		vRegisters[getX()] = (byte) (vRegisters[getY()] - vRegisters[getX()]);
		programCounter += 2;
	}

	/*
	8xyE - SHL Vx
	If the most significant bit of Vx is 1 then set VF to 1, otherwise set to 0. Then shift Vx left once (multiply by two).
	 */
	private void shiftLeftVx() {
		vRegisters[0x0F] = (byte) (vRegisters[getX()] >> 7); // Set based on MSB
		vRegisters[getX()] <<= 1;
		programCounter += 2;
	}

	/*
	9xy0 - SNE Vx, Vy
	We will skip the next opcode if Vx is not equal to Vy. If Vx != Vy then increment the program counter by 4 (to skip the next
	opcode) otherwise we will increment the program counter by 2.
	 */
	private void SNEVxIsNotVy() {
		if ( vRegisters[getX()] != vRegisters[getY()] ) {
			programCounter += 4;
		} else {
			programCounter += 2;
		}
	}

	/*
	Annn - LD I, nnn
	Set register I to nnn
	 */
	private void loadINNN() {
		indexRegister = getNNN();
		programCounter += 2;
	}

	/*
	Bnnn - JP V0, nnn
	The program counter is set to nnn + V0
	 */
	private void jumpNNNV0() {
		programCounter = (short) (getNNN() + vRegisters[0]);
	}

	/*
	Cxkk - RND Vx, kk
	We will generate a random number between 0 and 255. We will then bitwise AND that random number with Vx and store that in Vx.
	 */
	private void randomVxKK() {
		vRegisters[getX()] = (byte) (random.nextInt( 256 ) & vRegisters[getX()]);
		programCounter += 2;
	}

	/*
	Dxyn - DRW Vx, Vy, nibble
	Display n-byte sprite starting at memory location I and screen coordinate (Vx, Vy), set VF if there is a collision.
	We will read in n bytes from memory, starting at the address stored in I. These bytes are then displayed as sprites on the screen
	at location (Vx, Vy). Sprites are XORed onto the existing screen. If this causes any pixels to disappear, VF is set to 1, otherwise
	it is set to 0. If the sprite is positioned so that some of it is outside of the display it wraps around to the other side of the screen.

	Remember that sprites are always 8 pixels wide
	 */
	private void displayVxVyN() {
		int x = getX();
		int y = getY();
		int spriteHeight = getN();
		int pixelValue;

		for ( int yLine = 0; yLine < spriteHeight; yLine++ ) {
			pixelValue = memory[indexRegister + yLine];
			for ( int xLine = 0; xLine < 8; xLine++ ) {
				if ( (pixelValue & (0x80 >> xLine)) != 0 ) {
					if ( pixels[x + xLine + ((y + yLine) * 64)] == 1 ) {
						vRegisters[0x0F] = 1;
					} else {
						vRegisters[0x0F] = 0;
					}
					pixels[x + xLine + ((y + yLine) * 64)] ^= 1;
				}
			}
		}

		drawFlag = true;
		programCounter += 2;
	}

	/*
	Ex9E - SKP Vx
	Skip the next opcode if the key with the value of Vx is currently pressed.
	 */
	private void skipKeyPressed() {
		if ( keys[vRegisters[getX()]] != 0 ) {
			programCounter += 4;
		} else {
			programCounter += 2;
		}
	}

	/*
	ExA1 - SKNP Vx
	Skip the next opcode if the key with the value of Vx is currently NOT pressed.
	 */
	private void skipKeyReleased() {
		if ( keys[vRegisters[getX()]] == 0 ) {
			programCounter += 4;
		} else {
			programCounter += 2;
		}
	}

	/*
	Fx07 - LD Vx, Delay Timer
	The value of the delay timer is put into Vx
	 */
	private void loadVxDisplayTimer() {
		vRegisters[getX()] = (byte) delayTimer;
		programCounter += 2;
	}

	/*
	Fx0A - LD Vx, Key
	All execution STOPS until a key is pressed. Store the value of the key in Vx and then continue processing like normal.
	 */
	private void loadKeyPress() {
		for ( int i = 0; i < keys.length; i++ ) {
			if ( keys[i] != 0 ) {
				vRegisters[getX()] = keys[i];
				programCounter += 2;
			}
		}
	}

	/*
	Fx15 - LD Delay Timer, Vx
	Delay timer is set to the value of Vx
	 */
	private void loadDelayTimer() {
		delayTimer = vRegisters[getX()];
		programCounter += 2;
	}

	/*
	Fx18 - LD Sound Timer, Vx
	Sound timer is set to the value of Vx
	 */
	private void loadSoundTimer() {
		soundTimer = vRegisters[getX()];
		programCounter += 2;
	}

	/*
	Fx1E - Add I, Vx
	Add the values of I and Vx, store the results in I
	 */
	private void addIVx() {
		// TODO: Figure out if I need to set carry flag
		indexRegister += vRegisters[getX()];
		programCounter += 2;
	}

	/*
	Fx29 - LD F, Vx
	The value of I is set to the location for the hexadecimal sprite corresponding to the value of Vx.
	 */
	private void setFontLocationInI() {
		indexRegister = (short) (vRegisters[getX()] * 5); // Multiply by 5 since we have 5 values for each font
		programCounter += 2;
	}

	/*
	Fx33 - LD B, Vx
	Store binary coded decimal  of Vx at address I, I + 1 and I + 2. I gets the hundreds digit, I + 1 gets the tens digit and I + 2 gets the ones digit.
	 */
	private void setBCD() {
		memory[indexRegister] = (byte) (vRegisters[getX()] / 100);
		memory[indexRegister + 1] = (byte) ((vRegisters[getX()] / 10) % 10);
		memory[indexRegister + 2] = (byte) ((vRegisters[getX()] % 100) % 10);
		programCounter += 2;
	}

	/*
	Fx55 - LD I, Vx
	Store registers V0 through Vx in memory starting at location I
	 */
	private void storeRegisters() {
		for ( int registerIndex = 0; registerIndex <= getX(); registerIndex++ ) {
			memory[indexRegister + registerIndex] = vRegisters[registerIndex];
		}
		programCounter += 2;
	}

	/*
	Fx65 - LD Vx, I
	Read the memory values starting at I and load them into registers V0 through Vx
	 */
	private void loadRegisters() {
		for ( int registerIndex = 0; registerIndex <= getX(); registerIndex++ ) {
			vRegisters[registerIndex] = memory[indexRegister + registerIndex];
		}
		programCounter += 2;
	}


# Holy Opcodes, Batman!!

Right?! It's a lot even for such a simple machine. On our next episode we'll be covering something that I briefly mentioned in the opcode setFontLocationInI() which is the font set. I will also be covering how we are going to be selecting our opcodes. As a hint it may have something to do with Nintendo's latest hit console...