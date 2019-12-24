---
layout: post
title: "Tackling a CHIP-8 Interpreter, Part 4"
summary: "Setting Up Defaults"
featured-img: chip8init
---

# Necessary Variables, Initialization, etc.

I _promise_ this is the last boring one. After this one we will only be doing fun things. Get excited because next we'll be tackling ALL of the opcodes. 

Everything here should be relatively self explanatory or at least explained well enough with my comments so I won't be writing in too much detail. Plus you can go back to look at part 0 for a bit more information. Basically when we start our program we want to ensure that we have all of the default values. We need to ensure that everything is set to the default values. This is to ensure that we always process the ROMs the same upon each startup. So this means we'll have to setup the timers, memory, call stack, etc. We will also load in the ROM.

<code>

	private final int CPU_FREQUENCY = 500; /* Speed of our CPU in Hz */
	private final int TIMER_FREQUENCY = 60; /* Speed of our timers in Hz */

	private final int MAX_MEMORY = 4096; /* How many bytes of memory we have */
	private final int NUM_PIXELS = 2048; /* The total number of pixels we can display (64 * 32) */
	private final int NUM_V_REGISTERS = 16; /* We have 16 total registers that we can write to, each one being one byte */
	private final int MAX_CALL_STACK_LEVEL = 48; /* How deep our call stack will go. This is typically anywhere from 16-48, I'm going to be generous and make it 48 */
	private final int NUM_KEYS = 16; /* How many different keyboard inputs we accept (0 - F) */
	private final short PROGRAM_COUNTER_START_LOCATION = 0x200; /* Where the program counter start in our memory */
	private final short INDEX_REGISTER_START_LOCATION = 0; /* Index register's initial location */
	private final short OPCODE_START = 0; /* Our initial opcode should be nothing */
	private final short STACK_POINTER_START = 0; /* Where we begin in the stack */
	private final int MEMORY_ROM_START_LOCATION = 512; /* Where we can begin to store the ROM into memory (4096 - 512) */
	private final int DELAY_TIMER_START = 0; /* What our delay timer should start at */
	private final int SOUND_TIMER_START = 0;/* What our sound timer should start at */

	private short programCounter, indexRegister, stackPointer, opcode, delayTimer, soundTimer;
	private boolean[] pixels;
	private byte[] vRegisters, memory, keys, callStack;

	private boolean drawFlag; /* true if we are ready to draw a new frame */

    public Emulator() {
		initialize();
	}

    /*
    Setup default values for everything, load ROM into memory, there are 16 vRegisters, 2048 pixels (64*32), 16 levels in the stack, and 4096 max memory, and program counter starts at 0x200
    */
    private void initialize() {
        programCounter = PROGRAM_COUNTER_START_LOCATION;
        indexRegister = INDEX_REGISTER_START_LOCATION;
        stackPointer = STACK_POINTER_START;
        opcode = OPCODE_START;
        delayTimer = DELAY_TIMER_START;
        soundTimer = SOUND_TIMER_START;

        pixels = new boolean[NUM_PIXELS];

        vRegisters = new byte[NUM_V_REGISTERS];
        memory = new byte[MAX_MEMORY];
        keys = new byte[NUM_KEYS];
        callStack = new byte[MAX_CALL_STACK_LEVEL];

        try {
            List < Byte > rom = ROM.loadROM( Paths.get( "C:\\Users\\Guard\\Documents\\libGDXChip8\\core\\assets\\roms\\TETRIS" ) );

            for ( int memoryIndex = MEMORY_ROM_START_LOCATION; memoryIndex < MAX_MEMORY && (memoryIndex - MEMORY_ROM_START_LOCATION) < rom.size(); memoryIndex++ ) {
                memory[memoryIndex] = rom.get( memoryIndex - MEMORY_ROM_START_LOCATION );
            }

            System.out.println( rom.size() );
        } catch ( Exception e ) {
            e.printStackTrace();
            System.err.println( "Failed to load ROM" );
        }

        drawFlag = true;
    }


