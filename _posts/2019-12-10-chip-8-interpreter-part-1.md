---
layout: post
title: "Tackling a CHIP-8 Interpreter, Part 1"
summary: "How to read the ROMs"
featured-img: c8disassembler
---

# ROM Layout

ROMs consist solely of byte data. This means they won't open in all text viewing software as you might expect. Go ahead and download a [pack of ROMs](https://www.zophar.net/pdroms/chip8/chip-8-games-pack.html). Try opening one of them, I personally used Tetris, in a text editor like Sublime. You'll notice that it consists of a bunch of random letters and numbers similar to the image in my header above. These are the opcodes. Essentially we will read this into memory and use this while our game/program processes to determine what to do next. 

# Disassembler Design

From my understanding Chip-8 is designed to load in the entire ROM into memory which is what we will be doing. We will need to read in the file's bytes and then we will need to format the bytes into hexadecimal. We will then create the opcode which comprises of two bytes and we will store have that stored into our memory. We'll start with something like below to ensure we read the bytes in. Since we are only reading in 2 bytes that is a short. We will also ensure that the byte buffer is in the Big Endian notation as specified by Chip-8.

<code>

	byte[] fileBytes = Files.readAllBytes( filePath );

	ByteBuffer byteBuffer = ByteBuffer.wrap( fileBytes );
	byteBuffer.order( ByteOrder.BIG_ENDIAN);

	while(byteBuffer.hasRemaining()){
		short v = byteBuffer.getShort();
		System.out.println( v );
	}

</code>


Well if we look at the output we're getting all sorts of numbers, ranging from negative to positive. While byte wise this is expected we're going to translate this to be human readable to make more sense for us once we begin implementing opcodes. So we will want to convert our short to hexadecimal. With a [quick Google search](https://stackoverflow.com/questions/13356984/short-tohexstring) we can find out how to convert shorts to hexadecimal in Java. We haven't covered opcodes yet but they will be 2 bytes represented as String in our interpreter. This will make it rather easy going forward. We will not be using Integer.toHexString because it [trims leading zeroes](https://rules.sonarsource.com/java/RSPEC-4425). Instead we are going to grab each two bytes, build an Opcode object and then keep going until we run out of bytes.

For now we're going to keep our Opcode class simple and basically just have it hold the hexadecimal short. So our opcode class will look like:

<code>

	public class Opcode {

		private String opcode;

		public Opcode(String firstByte, String secondByte){
			opcode = firstByte + secondByte;
		}

		public Opcode(String opcodeBytes){
			opcode = opcodeBytes;
		}

		public String getOpcode(){
			return opcode;
		}

	}


</code>

Now that we can store the Opcode we will want to loop through 2 bytes at a time and save our Opcodes for later.

<code>
	
	public List<Opcode> loadROM(Path filePath) throws Exception{
        
        List<Opcode> opcodes = new ArrayList<>();

        ByteBuffer byteBuffer = ByteBuffer.wrap( Files.readAllBytes( filePath ) );
        byteBuffer.order( ByteOrder.BIG_ENDIAN);

        while(byteBuffer.hasRemaining()){
            try{
                String twoBytes = String.format("%04X",( byteBuffer.getShort() & 0xFFFF));

                Opcode opcode = new Opcode(twoBytes);
                opcodes.add(opcode);

                System.out.println( opcode.getOpcode() );
            } catch ( Exception e ){
                throw new Exception( "Erroneous byte found in file: " + filePath.getFileName()  );
            }
        }

        return opcodes;
    }

</code>


This concludes the basics of how to read to ROMs into memory and for later use. 

### Next Up we will cover the design of the Opcode class.

