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
		programCounter = (short) (opcode & 0x0FFF);
	}

	/*
	2nnn - Call address
	We will call the subroutine at address nnn. We will increment the stack pointer, put the current program counter on
	the top of the stack, and then set the program counter to nnn
	 */
	private void call() {
		stackPointer++;
		callStack[stackPointer] = programCounter;
		programCounter = (short) (opcode & 0x0FFF);
	}

	/*
	3xkk - SE
	We will skip the next opcode if Vx is equal to kk. If Vx == kk then increment the program counter by 4 (to skip the next
	opcode) otherwise we will increment the program counter by 2
	 */
	private void SE() {
		if ( vRegisters[(opcode & 0x0F00) >> 8] == (opcode & 0x0FF) ) {
			programCounter += 4;
		} else {
			programCounter += 2;
		}
	}

