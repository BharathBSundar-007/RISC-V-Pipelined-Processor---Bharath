# 5-Stage Pipelined RISC-V Processor (RV32I)



Welcome to the repository for my 5-stage pipelined RISC-V processor! This project implements the RV32I base integer instruction set using Verilog HDL, designed and simulated in Xilinx/AMD Vivado. 
By migrating from a single-cycle architecture to a 5-stage pipeline, this core significantly improves instruction throughput by executing multiple instructions concurrently.

---

## Architecture Overview

The datapath is divided into the classic 5-stage RISC pipeline, separated by pipeline registers to prevent data collision between concurrent instructions. 

| Stage | Abbreviation | Primary Function |
| :--- | :---: | :--- |
| **Instruction Fetch** | `IF` | Fetches the next instruction from memory using the Program Counter. |
| **Instruction Decode** | `ID` | Decodes the opcode/funct, reads registers, and extends immediate values. |
| **Execute** | `EX` | Performs ALU operations (math/logic) and calculates memory addresses. |
| **Memory** | `MEM` | Reads from or writes to the Data Memory (RAM). |
| **Write Back** | `WB` | Writes the final ALU or Memory result back into the Register File. |

---

## Project File Structure

Below is the structured breakdown of the design sources, categorized by their pipeline stage and function.

### 1. Top-Level Wrapper
> **`RISCV_Top.v`** (or `Datapath.v`)
> *The master file. Contains no logic of its own, but instantiates and wires all the modules below together to form the complete processor.*

### 2. Stage 1: Instruction Fetch (IF)
| Module | Description |
| :--- | :--- |
| 📄 **`PC.v`** | **Program Counter**: A register that holds the 32-bit address of the current instruction. |
| 📄 **`Instr_Mem.v`** | **Instruction Memory (ROM)**: Takes the PC address and outputs the 32-bit instruction code. |
| 📦 **`IF_ID.v`** | **Pipeline Register 1**: Catches the Instruction and PC address, holding them steady for the Decode stage. |

### 3. Stage 2: Instruction Decode (ID)
| Module | Description |
| :--- | :--- |
| 📄 **`Instr_Decode.v`** | **Control Unit**: Decodes the instruction (opcode, funct3, funct7) and generates control signals (ALU Op, Write Enables, etc.). |
| 📄 **`Reg_File.v`** | **Register File**: Contains the 32 general-purpose integer registers (`x0` to `x31`). |
| 📄 **`Imm_Gen.v`** | **Immediate Generator**: Extracts and sign-extends immediate values from I, S, B, U, and J type instructions. |
| 📦 **`ID_EX.v`** | **Pipeline Register 2**: Carries register data, immediate values, PC, and *all* control signals to the Execute stage. |

### 4. Stage 3: Execute (EX)
| Module | Description |
| :--- | :--- |
| 📄 **`ALU.v`** | **Arithmetic Logic Unit**: Performs arithmetic (ADD, SUB) and logical (AND, OR, XOR) operations. |
| 📦 **`EX_MEM.v`** | **Pipeline Register 3**: Catches the ALU result, store data, and remaining control signals for the Memory stage. |

### 5. Stage 4: Memory (MEM) & Write Back (WB)
| Module | Description |
| :--- | :--- |
| 📄 **`Data_Mem.v`** | **Data Memory (RAM)**: Handles load (`lw`) and store (`sw`) instructions. |
| 📦 **`Mem_wb.v`** | **Pipeline Register 4**: Catches the memory read data or ALU result to pass back to the `Reg_File.v`. |

### 6. Pipeline Control (Traffic Controllers)
> Pipelining introduces data hazards. These modules ensure the processor calculates correct values without stalling unnecessarily.

| Module | Description |
| :--- | :--- |
| 🛡️ **`Forward.v`** | **Forwarding Unit**: Intercepts newly calculated ALU data and routes it directly back to the ALU input if the next instruction needs it immediately (bypassing the register file delay). |
| 🛑 **`Hazard_Unit.v`** | **Hazard Detection Unit**: Detects situations where forwarding isn't enough (like a Load-Use hazard) and inserts a "bubble" (NOP) to stall the pipeline for one cycle. |

### 7. Verification
| Module | Description |
| :--- | :--- |
| 🔍 **`tb.v`** | **Testbench**: Generates clock and reset signals to simulate `RISCV_Top.v` and verify waveforms. |

---

## Tools & Technologies
* **Hardware Description Language:** Verilog HDL
* **IDE & Simulation:** Xilinx / AMD Vivado
* **Architecture:** 32-bit RISC-V (RV32I)

---
*Developed by Bharath*
