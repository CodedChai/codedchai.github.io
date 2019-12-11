---
layout: post
title: "Tackling a CHIP-8 Interpreter, Part 1"
summary: "How to read the ROMs"
featured-img: c8disassembler
---

# ROM Layout

ROMs consist solely of byte data. This means they won't open in all text viewing software as you might expect. Go ahead and download a [pack of ROMs](https://www.zophar.net/pdroms/chip8/chip-8-games-pack.html). Try opening one of them, I personally used Tetris, in a text editor like Sublime. You'll notice that it consists of a bunch of random letters and numbers similar to the image in my header above. These are the opcodes. Essentially we will read this into memory and use this while our game/program processes to determine what to do next. 

# Disassembler Design

From my understanding Chip-8 is designed to load in the entire ROM into memory which is what we will be doing. We will need to read in the file's bytes and then we will need to format the bytes into hexadecimal. We will then create the opcode which comprises of two bytes and we will store have that stored into our memory. The code will look something like what I have below.

<code>
	
	List<Opcode> opcodes = new ArrayList<>();

	byte[] fileBytes = Files.readAllBytes(filePath);

	for (int index = 0; index < fileBytes.length; index++) {
	    String firstByte = String.format("%02X",( fileBytes[index] & 0xFF));

	    if(fileBytes.length >= index + 1){
	        String secondByte = String.format("%02X",( fileBytes[++index] & 0xFF));

	        Opcode opcode = new Opcode(firstByte, secondByte);
	        System.out.println(opcode.getOpcode());
	        opcodes.add(opcode);
	    } else {
	        System.err.println("Erroneous byte " + firstByte + " found in file." );
	    }
	}

<code>

### Next Up we will cover the design of the Opcode class.

