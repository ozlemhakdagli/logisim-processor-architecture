# Computer Architecture & Processor Hardware Designs in Logisim-Evolution

[![Simulator: Logisim-Evolution](https://img.shields.io/badge/Simulator-Logisim--Evolution%20v5.0-blue.svg)](https://github.com/logisim-evolution/logisim-evolution)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Architectures](https://img.shields.io/badge/Architectures-8--Bit%20%7C%2016--Bit%20%7C%2032--Bit-orange.svg)](#-architectural-comparison)
[![Format](https://img.shields.io/badge/Format-Self--Contained%20.circ-brightgreen.svg)](#-repository-structure)

A comprehensive, gate-level hardware implementation repository featuring complete, working computer systems designed in **Logisim-Evolution**. 

The repository covers an educational progression across three distinct computing eras: an **8-bit accumulator-based educational machine**, the classic **16-bit M. Morris Mano Common Bus computer**, and a modern **32-bit Single-Cycle RISC-V (RV32I)** processor. All circuits have been natively designed, migrated, and verified for modern **Logisim-Evolution (v5.0.0)** with **zero compatibility warnings**, clean VHDL-compliant entity identifiers, and self-contained modularity.

---

## 🏛️ Architectural Comparison

| Architectural Feature | [8-Bit Educational Computer](8-bit-processor/) | [16-Bit Mano Machine](16-bit-processor/) | [32-Bit RISC-V Processor](32-bit-processor/) |
| :--- | :--- | :--- | :--- |
| **Reference Architecture** | BLM320 Custom Micro-Architecture | M. Morris Mano *Basic Computer* | RISC-V RV32I Base Integer ISA |
| **Word Size** | 8-bit | 16-bit | 32-bit |
| **Instruction Format** | Fixed 8-bit (Opcode + Operand) | Fixed 16-bit (Mode + Opcode + Addr)| Fixed 32-bit (R, I, S, B, U, J types) |
| **Register Set** | 4-bit AC, AR, PC, DR, IR, INPR, OUTR | 16-bit AC, DR, TR, IR, 12-bit AR, PC | 32 × 32-bit Registers ($x0 - x31$) |
| **Memory Organization** | 16 × 8-bit RAM (16 Bytes) | 4096 × 16-bit RAM (64 Kbit) | 4 KB ROM (Code) + 4 KB RAM (Data) |
| **Address Bus Width** | 4-bit | 12-bit | 32-bit (10-bit word addressing) |
| **Datapath / Bus Topology** | Bidirectional 8-bit shared bus | 16-bit Common Bus (Tri-State Buffers) | Single-Cycle Point-to-Point Datapath |
| **ALU Operations** | ADD, AND, XOR, INC, DEC, Pass | ADD, AND, XOR, INC, NOT, SHR, SHL | ADD, SUB, SLL, SLT, SLTU, XOR, SRL, SRA, OR, AND |
| **Addressing Modes** | Direct memory | Direct & Indirect memory | Register, Immediate, Base+Offset, PC-Relative |
| **Subcircuit Modules** | 20 self-contained units | 30 self-contained units | 6 modular units (`main`, `regfile`, `alu`, `imm_gen`, `control`, `branch`) |
| **Simulator Compatibility** | Logisim-Evolution v5.0+ | Logisim-Evolution v5.0+ | Logisim-Evolution v5.0+ |

---

## 📂 Repository Structure

```text
computer-architecture-logisim/
├── .gitignore                         # Build, temporary, and OS ignore patterns
├── LICENSE                            # MIT Open-Source License
├── README.md                          # Main repository overview & comparison matrix
├── logisim-evolution-5.0.0-all.jar    # Standalone Logisim-Evolution v5.0 runtime
├── 8-bit-processor/
│   ├── README.md                      # Complete 8-bit ISA, RTL micro-operations, and test program
│   └── 8_bit_computer.circ            # Self-contained 8-bit computer schematic (20 subcircuits)
├── 16-bit-processor/
│   ├── README.md                      # Complete 16-bit Mano Machine architecture, ISA, and test program
│   └── 16_bit_computer.circ           # Self-contained 16-bit computer schematic (30 subcircuits)
└── 32-bit-processor/
    ├── README.md                      # Complete 32-bit RV32I ISA, RTL, and simulation walkthrough
    └── 32_bit_computer.circ           # Self-contained 32-bit RISC-V computer schematic (6 subcircuits)
```

---

## 🚀 Getting Started & Simulation Guide

### 1. Prerequisites
To run and simulate the circuits, ensure you have Java 21 or newer installed:
```bash
java -version
```

### 2. Launching Logisim-Evolution
Launch the included standalone runtime JAR:
```bash
java -jar logisim-evolution-5.0.0-all.jar
```

### 3. Simulating the Processors

#### 8-Bit Educational Computer
1. In Logisim-Evolution, open [`8-bit-processor/8_bit_computer.circ`](8-bit-processor/8_bit_computer.circ).
2. Right-click the **RAM** module in `main`, select **Load Image...**, and paste the hex values from [`8-bit-processor/README.md`](8-bit-processor/README.md).
3. Enable clock execution via `Simulate > Ticks Enabled` (`Ctrl + K`).
4. Result: Arithmetic sum is computed and written to RAM address `0x9` (`0x03 + 0x02 = 0x05`).

#### 16-Bit Mano Basic Computer
1. In Logisim-Evolution, open [`16-bit-processor/16_bit_computer.circ`](16-bit-processor/16_bit_computer.circ).
2. Right-click the **RAM** module in `main` and load the test program from [`16-bit-processor/README.md`](16-bit-processor/README.md).
3. Start simulation via `Simulate > Ticks Enabled` (`Ctrl + K`) or step manually with `Ctrl + T`.
4. Result: Arithmetic sum is computed and written to RAM address `0x006` (`0x0053 + 0x0029 = 0x007C`).

#### 32-Bit RISC-V (RV32I) Processor
1. In Logisim-Evolution, open [`32-bit-processor/32_bit_computer.circ`](32-bit-processor/32_bit_computer.circ).
2. The Instruction ROM is **pre-loaded** with a verification program testing arithmetic, memory store/load, and branches.
3. Step each clock cycle manually with `Ctrl + T` (or enable auto-ticking with `Ctrl + K`).
4. Watch the live hexadecimal debug probes on `main`:
   - `x1_ra` = `0x0000000F` (15)
   - `x2_sp` = `0x0000001B` (27)
   - `x3_gp` = `0x0000002A` (42)
   - `x4_tp` = `0x0000002A` (loaded from Data RAM `Mem[4]`)
   - Program halts cleanly in an infinite loop branch `beq x0, x0, 0` at PC `0x14`.

---

## 🗺️ Architectural Roadmap

- [x] **8-Bit Educational Computer:** Gate-level design, custom ISA, single-file modular integration, verified with Logisim-Evolution v5.0.
- [x] **16-Bit Mano Machine:** Complete 16-bit Common Bus architecture, 30 subcircuit hierarchy, full 25-instruction set.
- [x] **32-Bit RISC-V Architecture (RV32I):** Complete single-cycle implementation with 32 registers, 10-op ALU, immediate generator, instruction ROM, data RAM, branch evaluation, verified with Logisim-Evolution v5.0.

---

## 📄 License

This repository is licensed under the [MIT License](LICENSE). You are free to use, modify, and distribute this work for academic, educational, and personal projects.
