<div align="center">

# RV32I RISC-V Processor with Custom Cryptographic (AES-128) Extension

### **Custom ISA Acceleration | 12-Cycle Register Marshalling FSM | AMD-Xilinx Artix-7**
**Architecture:** 32-Bit RV32I Integer Datapath + `custom-0` Cryptographic Extension  
**Operating Frequency:** 50.00 MHz ($T_{\text{clk}} = 20.00\text{ ns}$, Derived $F_{\max} = 79.35\text{ MHz}$)  
**Toolchain:** AMD-Xilinx Vivado Design Suite 2025.2

[![RISC-V](https://img.shields.io/badge/ISA-RISC--V%20(RV32I)-D41217?style=for-the-badge&logo=riscv&logoColor=white)](https://github.com/1dt23ec012-source/rv32i-aes128-custom-isa)
[![FPGA](https://img.shields.io/badge/FPGA-Artix--7%20(XC7A35T)-purple?style=for-the-badge&logo=amd&logoColor=white)](https://github.com/1dt23ec012-source/rv32i-aes128-custom-isa)
[![Timing Met](https://img.shields.io/badge/STA-50%20MHz%20Timing%20Met%20(%2B7.397ns%20WNS)-brightgreen?style=for-the-badge)](https://github.com/1dt23ec012-source/rv32i-aes128-custom-isa)
[![Zero BRAM/DSP](https://img.shields.io/badge/Logic-Pure%20Distributed%20LUT%2FFF-blue?style=for-the-badge)](https://github.com/1dt23ec012-source/rv32i-aes128-custom-isa)

</div>

---

## 📖 Executive Summary

Executing symmetric cryptography like AES-128 purely in scalar software on embedded processors incurs heavy execution overhead due to byte substitutions, Galois Field arithmetic, row shifts, and round key schedules. 

This project implements a functional **32-bit RV32I RISC-V processor core** augmented with a dedicated **custom cryptographic hardware extension**. The extension occupies the standard RISC-V `custom-0` opcode space (`7'b0001011`) and provides dedicated hardware acceleration for AES-128 encryption and decryption.

To transfer 128-bit cryptographic state and key blocks through a standard 32-bit processor architecture without modifying the 32-bit register file width or bloating instruction encodings, a **12-cycle register marshalling FSM** coordinates data collection, computation, and writeback across groups of four 32-bit architectural registers.

The system was synthesized, placed, and routed on an **AMD-Xilinx Artix-7 FPGA (XC7A35T)**, closing timing at **50.00 MHz with +7.397 ns of positive setup slack** ($F_{\max} = 79.35\text{ MHz}$).

---

## 🏛️ Processor Architecture & Custom ISA Extension

```text
 ┌─────────────────────────────────────────────────────────────────────────────────────────────────┐
 │                                 RV32I Processor Subsystem (top_edge_artix7)                     │
 │                                                                                                 │
 │       ┌──────────────────┐               ┌──────────────────┐                                   │
 │       │ Program Counter  ├──────────────►│Instruction Memory│                                   │
 │       │      (PC)        │               │   (On-Chip ROM)  │                                   │
 │       └────────┬─────────┘               └────────┬─────────┘                                   │
 │                │                                  │ 32-bit Instruction                          │
 │                ▼                                  ▼                                             │
 │       ┌──────────────────┐               ┌──────────────────┐                                   │
 │       │  Control Unit    ├──────────────►│Instruction Decode│                                   │
 │       │ (Multi-cycle)    │               └────────┬─────────┘                                   │
 │       └────────┬─────────┘                        │                                             │
 │                │ rs1, rs2, rd Base Pointers       ▼                                             │
 │                │                         ┌──────────────────┐                                   │
 │                │                         │  Register File   │ (x0 - x31, 32x32-bit)             │
 │                │                         └────────┬─────────┘                                   │
 │                │                                  │                                             │
 │                ▼                                  ▼                                             │
 │       ┌─────────────────────────────────────────────────────┐                                   │
 │       │               Multiplexed Execution Path            │                                   │
 │       │  ┌─────────────────────────┐  ┌──────────────────┐  │                                   │
 │       │  │    32-bit RV32I ALU     │  │ Custom AES-128   │  │                                   │
 │       │  │  ADD, SUB, XOR, Shifts  │  │ Cryptographic IP │  │                                   │
 │       │  └───────────┬─────────────┘  └────────┬─────────┘  │                                   │
 │       └──────────────┼─────────────────────────┼────────────┘                                   │
 │                      ▼                         ▼                                                │
 │       ┌─────────────────────────────────────────────────────┐                                   │
 │       │                 Writeback Multiplexer               ├────────► Register Update          │
 │       └─────────────────────────────────────────────────────┘                                   │
 └─────────────────────────────────────────────────────────────────────────────────────────────────┘
