---
layout: post
title: "Tackling a CHIP-8 Interpreter, Part 5"
summary: "Opcodes Opcodes and More Opcodes"
featured-img: chip8init
---

# What is an Opcode?

Okay so there are 35 opcodes for Chip-8. You may be asking, what the hell is an [opcode](https://en.wikipedia.org/wiki/Opcode)? Well basically it's just an instruction for the CPU. It will operate on the memory, timers, registers, call stack, etc. In a normal emulator each opcode has a specific number of cycles it takes to run, in Chip-8 every opcode only takes one cycle. That means that every opcode will be treated equal in the sense that they all take 1 CPU cycle. 

All opcodes are 2 bytes long and are stored big endian. Something to note is that in memory each first byte of each instruction should be stored in an even address. I will be using the same verbiage as [other Chip-8 tutorials](http://devernay.free.fr/hacks/chip8/C8TECH10.HTM) to help describe the opcodes. The verbiage is below:

*nnn or addr* - 12 bit value, specifically the lower 12 bits of the opcode.
*n or nibble* - 4 bit value, specifically the lowest 4 bits of the opcode.
*x* - 4 bit value, specifically the lower 4 bits of the high byte of the opcode. That means the lower 4 bits of the first byte.
*y* - 4 bit value, specifically the upper 4 bits of the low byte of the opcode. That means the higher 4 bits of the second byte.
*kk or byte* - 8 bit value, specifically the lowest 8 bits of the opcode.

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



# WARNING! FUN TIMES AHEAD!

## How do we get our opcode?

Remember, an opcode is 2 bytes of data that (should, we won't add any validation) start at an even numbered memory location. So to get our opcode we will use this code:

<code>

    opcode = (short) (memory[programCounter] << 8 | memory[programCounter + 1]);

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
	

