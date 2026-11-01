# RISC-V Single-Cycle Processor Project

## Project Overview

For this project, we designed and implemented a 32-bit RISC-V single-cycle processor using Verilog in Vivado. The processor executes instructions stored in instruction memory and performs operations such as arithmetic, logical operations, branching, jumping, and memory access.

We also connected the processor to the FPGA board so we can monitor different values using the LEDs and seven-segment display.

## Project Files

### 1. fullDataPathModule.v

This is the main module of our RISC-V processor. It connects all the components together and controls how instructions are executed.

The main components are:

- Program Counter (PC)
- Instruction Memory
- Control Unit
- Register File
- Immediate Generator
- ALU and ALU Control
- Data Memory
- Branch and Jump Logic
- Write-Back Logic

The processor reads an instruction, decodes it, executes the required operation, and updates the PC.

### 2. instructionMemory.v

This module stores the instructions that our processor executes.

We used a memory containing 64 instructions, with each instruction being 32 bits.

The instructions test different operations, including:

- ADD, ADDI, SUB
- LUI, AUIPC
- SLT, SLTU, SLTI, SLTIU
- SLL, SLLI, SRLI, SRAI
- XORI
- BEQ, BNE, BLT, BGE, BLTU, BGEU
- JAL, JALR
- LW, LH, LHU, LB, LBU
- SW, SH, SB

Some instructions are used to check that register x0 always stays zero, even when we try to write a value into it.

The last instruction is EBREAK, which stops the processor by preventing the PC from updating.

### 3. ControlUnit.v

The Control Unit reads the instruction opcode and generates the control signals needed by the datapath.

These include:

- Branch
- Jump
- JumpReg
- MemRead
- MemWrite
- ALUSrc
- RegWrite
- ASel
- WBSel

These signals decide which operation should happen and where the results should go.

### 4. RegFile.v

The Register File contains 32 registers, each 32 bits wide.

It allows the processor to read two registers and write a result into one register.

Register x0 must always contain zero.

### 5. ImmGen.v

The Immediate Generator extracts immediate values from instructions.

It handles the different immediate formats needed for arithmetic, load/store, branch, and jump instructions.

### 6. ALU.v and ALUControl.v

The ALU performs arithmetic and logical operations.

The ALU Control determines which operation the ALU should perform based on the instruction.

### 7. DataMem.v

The Data Memory is used for load and store instructions.

It supports reading and writing different data sizes, including bytes, halfwords, and words.

### 8. N_bitRegister.v

This module is used for the Program Counter.

The PC normally increases by 4 after each instruction. It can also change to a branch or jump address.

When EBREAK is reached, the PC stops updating.

### 9. N_bit_shift_left.v

This module shifts the immediate value to the left.

In our final datapath, the shifted immediate is only used for monitoring because the Immediate Generator already produces the correct branch and jump offsets.

### 10. Four_Digit_Seven_Segment_Driver_Optimized.v

This module displays values on the FPGA board's four-digit seven-segment display.

It includes a BCD converter that converts a binary number into decimal digits.

The display driver switches between the four digits quickly so that they appear to be displayed at the same time.

### 11. top_datapath_sevenSegement.v

This is the top module used to connect the RISC-V processor to the FPGA board.

It connects the processor's output to the seven-segment display driver.

The main inputs and outputs are:

- clk: Clock used for the seven-segment display.
- rclk: Clock used to execute processor instructions.
- rst: Resets the processor.
- ledSel: Selects which information appears on the LEDs.
- ssdSel: Selects which value appears on the seven-segment display.
- leds: The 16 LED outputs.
- LED_out: The seven-segment outputs.
- Anode: Selects the active display digit.

### 12. top_datapath_sevenSegement.xdc

This file contains the FPGA pin assignments.

It connects the inputs and outputs of the top module to the physical clock, switches, LEDs, and seven-segment display.

The constraints are written for the Basys 3 board.

### 13. fullDataPath_tb.v

This is the simulation testbench for the processor.

It runs the instructions stored in instruction memory and checks the behavior of the datapath.

The testbench is used to check:

- PC updates
- Branch and jump instructions
- Register values
- Memory operations
- EBREAK behavior

It also helps us identify errors before generating the bitstream.

## How the Processor Works

1. The PC provides the address of the current instruction.
2. Instruction Memory returns the instruction.
3. The Control Unit decodes the instruction and generates the control signals.
4. The Register File provides the required register values.
5. The Immediate Generator produces the immediate value when needed.
6. The ALU performs the operation.
7. Data Memory is accessed if the instruction is a load or store.
8. The result is written back to the Register File when required.
9. The PC is updated to the next instruction address.

All these steps happen within one clock cycle because this is a single-cycle processor.

## Branch and Jump Instructions

We implemented the six branch instructions:

- BEQ: Branch if equal.
- BNE: Branch if not equal.
- BLT: Branch if less than (signed).
- BGE: Branch if greater than or equal (signed).
- BLTU: Branch if less than (unsigned).
- BGEU: Branch if greater than or equal (unsigned).

For branch instructions, the processor compares the two register values and decides whether to change the PC.

We also implemented JAL and JALR.

JAL changes the PC using a PC-relative offset, while JALR uses a register value and an immediate.

Both can save the return address (PC + 4) in the destination register.

## Using the LEDs

We used the 16 LEDs to monitor different information from the processor.

The value of ledSel determines what is displayed.

| ledSel | Display |
|---|---|
| 00 | Lower 16 bits of the instruction |
| 01 | Upper 16 bits of the instruction |
| 10 | Control signals and ALU information |
| 11 | All LEDs off |

This makes it easier to observe the processor while running instructions.

## Using the Seven-Segment Display

The ssdSel input selects which datapath value appears on the seven-segment display.

| ssdSel | Display |
|---|---|
| 0000 | PC |
| 0001 | PC + 4 |
| 0010 | Branch Target |
| 0011 | Next PC |
| 0100 | ReadData1 |
| 0101 | ReadData2 |
| 0110 | WriteBackData |
| 0111 | Immediate |
| 1000 | ShiftedImmediate |
| 1001 | ALUInput2 |
| 1010 | ALUResult |
| 1011 | DataMemoryOut |

Only the lower 13 bits of each value are displayed, so larger values will not appear in full.


## Conclusion

In this project, we built a single-cycle RISC-V processor and connected its main components using Verilog.

We used instruction memory to test arithmetic, logical, branch, jump, and memory instructions. We also added LED and seven-segment monitoring to make it easier to see what is happening inside the processor.

The simulation testbench helps us check the processor's behavior, while the FPGA implementation allows us to observe its outputs on the board.
