This is an 8-Bit computer built within [Logisim](http://www.cburch.com/logisim/index.html)
based on the excelent [video series by Ben Eater](https://www.youtube.com/playlist?list=PLowKtXNTBypGqImE405J2565dvjafglHU)
.

![The SAP1p schematic](i/sap1p.jpg??raw=true)

The architecture is based on the [Simple as Possible 1](http://drghimire.com.np/simple-as-possible-computer-1-sap1-architecture/)
computer. It has an 8-bit bus, and features an 8-bit by 16 row memory. Most
operations use the 'A' register, and can perform addition and subtraction via
its ALU. Hexadecimal output is via two seven-segment displays. 8bit commands
on the computer contain a 4bit opcode, with a 4bit argument (usually a memory
address). I've called the computer 'plus' because it contains some extra
functionality such as the JC (Jump if carry bit set) command.

I built this by following along with Ben Eater's video series as a way to
do some hands-on learning while progressing through the series without
carrying around the set of kit to physically build the computer.

## Usage

Once you have loaded the computer in logisim, click the 'MAIN' circuit -
and use the 'hand' tool to interact with the computer.

In the Upper-Left corner of the clock is the 'step/run toggle'. This toggles
between manaully stepping through a program using the 'ST' button - or running
a program using the internal clock to step through the program. Once a program
has completed (or has errored) you can use the 'RSET' button to reset the
computer. This leaves memory/ram intact.

To program the computer, use the program interface above the RAM module.
Click the 'P/R' toggle to place the computer in 'PROG' mode. Then enter a
4bit 'addr', and below that an 8bit value. Press the 'set' value for 2 seconds.
You should see the value indicator lights below the RAM module change
indicating the new value is set. Continue this process for all memory values
you need to change, and then toggle back to 'RUN' mode. You should then
click 'RSET' to initialize the computer, and toggle the 'STEP/RUN' to run your
program (if it is not already running)

## Instruction Set

Instructions are passed into the computer using the opcodes below. The computer
uses the first four bits as the opcode, and the second four bits as an
argument to the opcode. Frequently the argument is a memory address, but may
also be a 4-bit value.

%0000 $0 NOOP: Null operation - do nothing!
%0001 $1 LDA [X]: load the A register with the value in memory location X
%0010 $2 ADD [X]: Set A to the value of A plus the value from mem loc X
%0011 $3 SUB [X]: Set A to the alue of A minus the value from mem loc X
%0100 $4 STA [X]: Set memory location X to the value in register A
%0110 $6 JMP [X]: Jump to a new address for the next command execution
%0111 $7 LDI X: Load the A register with the passed 4-bit value
%1000 $8 JC [X]: Jump to new address if carry bit set (branching)
%1110 $e OUT: Transfer the value in the A register into the display/output
%1111 $f HLT: Halt computer execution

There is room for more commands/opcodes - if you desire to expand the computer.


## Example Programs

### Add two numbers

    0x0 LDA 14  0000  00011110
    0x1 ADD 15  0001  00101111
    0x2 OUT     0010  11100000
    0x3 HLT     0011  11110000
    0xe 14      1110  00001110
    0xf 28      1111  00011100

Should output '2a' (aka 42)

### Fibonacci Sequence

This is the largest program that can fit in the computer. The program uses
thirteen bytes, and the final three bytes are used for storing values.

    0x0 ldi 0x1    0000  01110001
    0x1 sta [0xe]  0001  01001110
    0x2 ldi 0x0    0010  01110000
    0x3 out        0011  11100000
    0x4 add [0xe]  0100  00101110
    0x5 sta [0xf]  0101  01001111
    0x6 lda [0xe]  0110  00011110
    0x7 sta [0xd]  0111  01001101
    0x8 lda [0xf]  1000  00011111
    0x9 sta [0xe]  1001  01001110
    0xa lda [0xd]  1010  00011101
    0xb jc  0x0    1011  10000000
    0xc jmp 0x3    1100  01100011

## Microcode Reference

The microcode for each opcode is below. The system uses 5 ticks per command.
The first two ticks are used to load the next command into the command register
and to increment the program counter. There are sixteen control lines which the
are manipulated by the microcode.

Flags

* HLT: Halt system
* MI: Memory (address) Register input
* RI: RAM Input
* RO: RAM Output
* IO: Instruction Register Output
* II: Instruction Register Input
* AI: A register input
* A0: A register output
* EO: ALU output
* SU: ALU subtract
* BI: B register input
* BO: B register output
* OI: Output (display) Input
* CE: Instruction Counter Increment
* CO: Instruction Counter output
* J: Jump to address (Counter Input)


             0 1 2 3  4 5 6 7  8 9 0 1  2 3 4 5
        i s  H M R R  I I A A  E S B O  C C   J
    ins c t  T I I 0  0 I I 0  0 U I I  E 0 J C hexc  
                                                      
    --- x 0    1                          1     4004
    --- x 1  0     1    1               1       1408
    0000:
    NOP 0 2                                     0000
    0001:
    LDA 1 2    1      1                         4800
    LDA 1 3        1      1                     1200
    0010:
    ADD 2 2    1      1                         4800
    ADD 2 3        1               1            1020
    ADD 2 4               1    1                0280
    0011:
    SUB 3 2    1      1                         4800
    SUB 3 3        1               1            1020
    SUB 3 4               1    1 1              02c0
    0100:
    STA 4 2    1      1                         4800
    STA 4 3      1          1                   2100
    0110:
    JMP 6 2           1                     1   0802
    0111:
    LDI 7             1   1                     0a00
    1000:
    JC  8             1                       1 0801
    1110:
    OUT e 2                 1        1          0110
    1111:
    HLT f 2  1                                  8000

## Reference - hex to binary

Values are loaded into the computer using binary. Here's a handy reference
for converting between hex and binary:

    0 - 0000
    1 - 0001
    2 - 0010
    3 - 0011
    4 - 0100
    5 - 0101
    6 - 0110
    7 - 0111
    8 - 1000
    9 - 1001
    A - 1010
    B - 1011
    C - 1100
    D - 1101
    E - 1110
    F - 1111
