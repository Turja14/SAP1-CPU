# SAP-1 CPU Implementation

A complete Simple-As-Possible (SAP-1) CPU built in Logisim Evolution with a hardwired control unit, a ROM-based bootloader (data loader), and an ALU extended with shift/rotate units.

---

## Table of Contents
- [Abstract](#abstract)
- [Introduction](#introduction)
  - [Project Overview](#project-overview)
  - [Purpose and Goals](#purpose-and-goals)
  - [SAP-1 CPU Architecture](#sap-1-cpu-architecture)
- [Design and Implementation](#design-and-implementation)
  - [Overview of SAP-1 Architecture](#overview-of-sap-1-architecture)
  - [Key Components](#key-components)
    - [Program Counter (PC)](#program-counter-pc)
    - [Registers (A, B, Amount Reg)](#registers-a-b-amount-reg)
    - [Instruction Register (IR)](#instruction-register-ir)
    - [RAM](#ram)
    - [ALU + Shifter + Rotator](#alu--shifter--rotator)
    - [Control Sequencer](#control-sequencer)
    - [T-states](#t-states)
    - [Bootloader / Data Loader](#bootloader--data-loader)
  - [Compiler Interface](#compiler-interface)
- [Final Circuit](#final-circuit)
- [Control Signals (Selected)](#control-signals-selected)
- [Instruction Set & Example Program](#instruction-set--example-program)
- [Step-by-Step: Running](#step-by-step-running)
- [Testing and Validation](#testing-and-validation)
- [Features and Enhancements](#features-and-enhancements)
- [Future Work and Improvements](#future-work-and-improvements)
- [References](#references)
- [License](#license)

---

## Abstract
This project presents the design and implementation of a **Simple-As-Possible (SAP-1) CPU** in **Logisim Evolution**. Core units—**PC**, **IR**, **A/B registers**, **RAM**, and a hardwired **Control Unit**—cooperate to realize the classic **fetch–decode–execute** cycle. An extended **ALU** integrates **barrel shifter** and **rotator**, and a **ROM-based bootloader** automatically preloads programs into RAM, avoiding manual entry. The CPU executes basic instructions (load, add, store, shift, rotate, halt) and serves as a clear, hands-on learning platform for computer architecture.

---

## Introduction

### Project Overview
A complete SAP-1 CPU implemented in Logisim Evolution to demonstrate fundamental CPU concepts—fetching, decoding, executing—and to showcase clean control sequencing and datapath design.

### Purpose and Goals
- Building a fully functional SAP-1 with a **hardwired** control unit.
- **Automating program load** via a **ROM bootloader** into RAM.
- Demonstrating basic arithmetic and data-movement operations (e.g., LDA, ADD, STA, SHL/SHR, ROTL/ROTR, HLT).

### SAP-1 CPU Architecture
Essential blocks: **PC**, **IR**, **A/B registers**, **Shift/Rotate amount register**, **ALU (add/sub + shifter + rotator)**, **RAM**, and the **Control Sequencer**. The control unit generates precise signals per T-state to orchestrate data movement and operations.

---

## Design and Implementation

### Overview of SAP-1 Architecture
A minimal but complete datapath connected by a shared bus, driven by a **ring-counter–based** sequencer that steps through **T1…T6**. Instructions are fetched from RAM, decoded by IR bits, and executed through ALU/regs under control signals.

### Key Components

#### Program Counter (PC)
4-bit synchronous counter that holds the address of the next instruction. Signals: `pc_en` (increment), `pc_out_en` (drive address bus), `pc_reset` (clear).  
*Image:* `images/pc.png`

#### Registers (A, B, Amount Reg)
8-bit edge-triggered registers with `reg_in_en`, `reg_clk`, `reg_out_en`. The **amount register** stores shift/rotate counts used by the shifter/rotator.  
*Image:* `images/register.png`

#### Instruction Register (IR)
Splits an 8-bit instruction into **opcode (upper nibble)** and **operand (lower nibble)**; `ir_in_en` latches, `ir_out_en` exposes operand.  
*Image:* `images/ir.png`

#### RAM
16×8 SRAM built from register cells, a 4→16 decoder, and tri-state buses. Control: `sram_rd`, `sram_wr`, `cs`.  
*Images:* `images/ram_cell.png`, `images/ram.png`

#### ALU + Shifter + Rotator
- **ALU**: add/sub on `reg_a` and `reg_b`, outputs `alu_output`, `alu_carry_out`.
- **Shifter**: 8-bit **barrel shifter** using MUX stages; shifts by any amount in one cycle.
- **Rotator**: 8-bit **barrel rotator** for circular shifts without data loss.  
*Images:* `images/alu.png`, `images/barrel_shifter.png`, `images/barrel_rotator.png`

#### Control Sequencer
Hardwired sequencer with a **ring counter** (generates T1…T6), opcode **decoder**, and a **control matrix** (AND/OR/NAND) that asserts control pins at precise T-states.  
*Images:* `images/sequencer_block.png`, `images/sequencer_inner1.png`, `images/sequencer_inner2.png`

#### T-states
A **T-state** is a discrete timing step in the instruction cycle. The control logic combines T-states with opcode lines to produce outputs (e.g., `pc_out`, `mar_in_en`, `sram_rd`, `a_in`, `alu_out_en`).

#### Bootloader / Data Loader
A small ROM-driven loader that, when `enable=1`, iterates addresses, reads a ROM byte, asserts `mar_in_en` and `sram_wr`, and writes the byte to RAM. Includes **debug** mode for stepping.  
*Images:* `images/data_loader.png`, `images/data_loader_ctrl.png`

### Compiler Interface
A web tool (screenshot included) converts SAP-1 assembly (e.g., `LDA 13`, `ADD 14`, `STA 15`, `ROR 4`, `HLT`) into 16-byte hex for Logisim ROM init.  
*Image:* `images/compiler.png`

---

## Final Circuit
The complete top-level SAP-1 schematic integrating datapath, control, and bootloader.  
*Image:* `images/final_top.png`

---

## Control Signals (Selected)
Control pins are asserted by Boolean expressions of **T-states** and **decoded opcodes** realized by AND/OR/NAND gates in the control matrix.

- `pc_out` : **T1**  
- `mar_in_en` : **T1** (fetch) and **T4** when executing memory-operand instructions  
- `sram_rd` : **T2** (fetch) and **T5** (execute, memory read)  
- `ir_in_en` : **T2**  
- `pc_en` : **T3**  
- `a_in` : **T5** for `LDA`; **T6** for ALU/shifter/rotator results  
- `b_in` : **T5** for `ADD`/`SUB`  
- `alu_out_en` : **T6** for `ADD`/`SUB`  
- `sram_wr` : **T6** for `STA`  
- Shifter/Rotator enables follow T5 (capture operands/amount) and T6 (output result)

(See `docs/control_signals.md` for the full table if you export one.)

---

## Instruction Set & Example Program

| Mnemonic | Hex | Description                              |
|---------:|:---:|------------------------------------------|
| LDA x    | 0x1x| Load A from RAM[x]                      |
| ADD x    | 0x2x| A ← A + RAM[x]                          |
| SUB x    | 0x3x| A ← A − RAM[x]                          |
| STA x    | 0x4x| Store A → RAM[x]                        |
| SHL n    | 0x5n| Logical left shift A by n               |
| SHR n    | 0x6n| Logical right shift A by n              |
| ROTL n   | 0x7n| Rotate A left by n                      |
| ROTR n   | 0x8n| Rotate A right by n                     |
| HLT      | 0xF0| Halt                                    |

**Sample (16-byte ROM image):**  
`1D 2E 4F 84 F0 00 00 00 00 00 00 00 00 33 19` (as from the compiler screenshot)

---

## Step-by-Step: Running

1. **Preload Program (Bootloader)**
   - Assert `reset`, set `enable=1` (and `debug` as desired).  
   - The loader sequences: `address_en` → ROM read → `mar_in_en` → `sram_wr` → increment address, until all bytes are written.

2. **Execute Program**
   - Release loader, start CPU clock.
   - **Fetch**: `pc_out`→`mar_in_en` (T1), `sram_rd`+`ir_in_en` (T2), `pc_en` (T3).  
   - **Decode**: opcode lines assert (combinational).  
   - **Execute**: memory address out (T4), memory/regs/ALU actions (T5–T6) per instruction.

---

## Testing and Validation
- **Unit tests** of PC, registers, RAM, ALU/shifter/rotator, and control unit.  
- **Instruction verification** for LDA/ADD/SUB/STA/SHL/SHR/ROTL/ROTR/HLT.  
- **Program-level simulation** to check fetch–decode–execute sequencing.  
- **Bootloader tests** to confirm correct ROM→RAM transfer.  
- **Edge cases**: address bounds, arithmetic carry/borrow, shift/rotate extremes.

---

## Features and Enhancements
- **Hardwired control unit** with ring counter T-states.  
- **ROM-based bootloader** for automatic RAM initialization.  
- **Barrel shifter** and **barrel rotator** integrated with ALU.  
- **Debug probes** (address/data) for visibility during loading/execution.

---

## Future Work and Improvements
- **Expanding** the instruction set (JMP/CALL/RET, conditionals).  
- **Implementing** pipelining and **optimizing** the control matrix.  
- **Adding** stack/interrupt support and increasing memory size.  
- **Developing** richer debug tools (breakpoints, memory watch).  
- **Enhancing** visualization and documentation.

---

## References
- Ben Eater — *8-bit breadboard computer* playlist/search:  
  <https://www.youtube.com/results?search_query=8+bit+breadboard+computer>
- Tamanna Nazmin — *SAP-1 Controller Sequencer (Proteus)*  
- Touhidul Islam — *SAP-1 Sequencer (Proteus)*  
- Ahsanullah Khalid — *SAP-1 CPU Logisim* repo: <https://github.com/aukhalid/SAP-1-CPU-Logisim>

---

## License
This project is for educational use. Add your preferred license (e.g., MIT) here.

---

### Screenshots
Place your images under `images/` and keep these names or adjust links above:

