# SAP-1 CPU Implementation
---

## 1. Introduction

### 1.1 Project Overview
This project involves the design and implementation of a Simple-As-Possible (SAP-1) CPU using **Logisim Evolution**. The SAP-1 architecture is a minimalistic CPU model used for educational purposes to demonstrate basic computer architecture principles.

### 1.2 Purpose and Goals
The primary objective of this project is to create a fully functional SAP-1 CPU that automates the **fetch-decode-execute** cycle. By adding a **ROM-based bootloader**, the CPU can load machine code programs into memory, simplifying the process and reducing human error. The project aims to execute basic operations like addition, subtraction, shift, rotate and demonstrate the core operations of a CPU.

### 1.3 SAP-1 CPU Architecture
The SAP-1 CPU architecture features a simple design with essential components such as a **Program Counter**, **Instruction Register**, **Accumulator**, **Arithmetic Logic Unit (ALU)**, and a **Control Unit**. The processor follows a straightforward fetch-decode-execute cycle, with a hardwired control unit to manage the flow of operations. The addition of the ROM bootloader enhances the functionality by enabling automatic program loading.

---

