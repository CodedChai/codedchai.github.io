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

