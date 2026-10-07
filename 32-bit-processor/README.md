# 32-Bit Single-Cycle RISC-V (RV32I) Processor

A native, modular **32-Bit Single-Cycle RISC-V (RV32I)** computer designed and simulated in **Logisim-Evolution v5.0.0**. This processor implements the standard open-source RISC-V Base Integer Instruction Set Architecture (RV32I) as defined by RISC-V International, following the canonical Patterson & Hennessy textbook design.

---

## 1. Architectural Overview

```
                      +-------------------+
                      |   Program Counter | <-----------+
                      +---------+---------+             |
                                |                       | (Next PC Mux)
                                v                       |
                      +-------------------+             |
                      | Instruction ROM   |             |
                      +---------+---------+             |
                                | [31:0]                |
           +--------------------+-------------------+   |
           |                    |                   |   |
           v                    v                   v   |
+---------------------+ +----------------+ +------------+-----+
|    Control Unit     | |  Imm Generator | |   Register File  |
|  (Opcode/Funct3/7)  | +--------+-------+ | (32x 32-bit Reg) |
+----------+----------+          |         +-----+------+-----+
           |                     |               |      |
           | ALUSrc, MemToReg... |               | rs1  | rs2
           |                     v               v      v
           |                 +-------+       +---+------+---+
           |                 | Mux B | <-----+  ALU Operand |
           |                 +---+---+       +------+-------+
           |                     |                  |
           |                     +--------+         |
           |                              |         |
           v                              v         v
+---------------------+             +-------------------+
|     Branch Eval     |             |    32-Bit ALU     |
|   (BEQ, BNE, BLT..) |             +---------+---------+
+----------+----------+                       | ALUResult
           |                                  v
           |                        +-------------------+
           |                        |     Data RAM      |
           |                        +---------+---------+
           |                                  |
           v                                  v
+---------------------+             +-------------------+
|   Next-PC Logic     | ----------->|   Writeback Mux   | ---> rd
+---------------------+             +-------------------+
```

### Key Architectural Specifications:
- **ISA**: RISC-V RV32I Base Integer Instruction Set (Unprivileged Spec).
- **Execution Model**: Single-Cycle Datapath (1 instruction per clock cycle).
- **Word Length**: 32-bit datapath, registers, instructions, and arithmetic.
- **Register File**: 32 general-purpose 32-bit registers ($x0 - x31$).
  - $x0$ is hardwired to constant `0x00000000` (writes to $x0$ are discarded).
  - 2 asynchronous read ports (`rs1`, `rs2`), 1 synchronous write port (`rd`, `RegWrite`).
- **Memory Architecture**:
  - **Instruction Memory (ROM)**: 10-bit word address ($1024 \times 32$-bit = 4 KB addressable code space).
  - **Data Memory (RAM)**: 10-bit word address ($1024 \times 32$-bit = 4 KB addressable data space), byte/word address translation via `Addr[11:2]`.
- **Immediate Formats Supported**: I-type, S-type, B-type, U-type, and J-type sign-extension.

---

## 2. Register File & ABI Conventions

The processor provides the complete set of 32 RISC-V integer registers:

| Register | ABI Name | Description | Saver |
|:---|:---|:---|:---|
| **$x0$** | `zero` | Hardwired constant zero | — |
| **$x1$** | `ra` | Return address | Caller |
| **$x2$** | `sp` | Stack pointer | Callee |
| **$x3$** | `gp` | Global pointer | — |
| **$x4$** | `tp` | Thread pointer | — |
| **$x5 - x7$** | `t0 - t2` | Temporaries | Caller |
| **$x8$** | `s0 / fp` | Saved register / Frame pointer | Callee |
| **$x9$** | `s1` | Saved register | Callee |
| **$x10 - x17$** | `a0 - a7` | Function arguments / Return values | Caller |
| **$x18 - x27$** | `s2 - s11`| Saved registers | Callee |
| **$x28 - x31$** | `t3 - t6` | Temporaries | Caller |

> [!NOTE]
> The top-level circuit `main` includes dedicated hexadecimal debug displays for registers **$x1$ (`ra`)**, **$x2$ (`sp`)**, **$x3$ (`gp`)**, and **$x4$ (`tp`)** for direct inspection during simulation.

---

## 3. Supported Instruction Set (RV32I)

### 3.1 R-Type Instructions (`opcode = 0110011` / `0x33`)
| Mnemonic | `funct3` | `funct7` | Operation | Description |
|:---|:---:|:---:|:---|:---|
| `add rd, rs1, rs2` | `000` | `0000000` | $R[rd] = R[rs1] + R[rs2]$ | Addition |
| `sub rd, rs1, rs2` | `000` | `0100000` | $R[rd] = R[rs1] - R[rs2]$ | Subtraction |
| `sll rd, rs1, rs2` | `001` | `0000000` | $R[rd] = R[rs1] \ll R[rs2][4:0]$ | Shift Left Logical |
| `slt rd, rs1, rs2` | `010` | `0000000` | $R[rd] = (R[rs1] < R[rs2]) \,?\, 1 : 0$ | Set on Less Than (signed) |
| `sltu rd, rs1, rs2`| `011` | `0000000` | $R[rd] = (R[rs1] <_{u} R[rs2]) \,?\, 1 : 0$ | Set Less Than Unsigned |
| `xor rd, rs1, rs2` | `100` | `0000000` | $R[rd] = R[rs1] \oplus R[rs2]$ | Bitwise XOR |
| `srl rd, rs1, rs2` | `101` | `0000000` | $R[rd] = R[rs1] \gg_{L} R[rs2][4:0]$ | Shift Right Logical |
| `sra rd, rs1, rs2` | `101` | `0100000` | $R[rd] = R[rs1] \gg_{A} R[rs2][4:0]$ | Shift Right Arithmetic |
| `or rd, rs1, rs2`  | `110` | `0000000` | $R[rd] = R[rs1] \mid R[rs2]$ | Bitwise OR |
| `and rd, rs1, rs2` | `111` | `0000000` | $R[rd] = R[rs1] \ \\& \ R[rs2]$ | Bitwise AND |

### 3.2 I-Type Instructions
| Mnemonic | `opcode` | `funct3` | Operation | Description |
|:---|:---:|:---:|:---|:---|
| `addi rd, rs1, imm` | `0010011` | `000` | $R[rd] = R[rs1] + \text{imm}$ | Add Immediate |
| `slli rd, rs1, shamt`| `0010011` | `001` | $R[rd] = R[rs1] \ll \text{shamt}$ | Shift Left Logical Imm |
| `slti rd, rs1, imm` | `0010011` | `010` | $R[rd] = (R[rs1] < \text{imm}) \,?\, 1 : 0$ | Set Less Than Imm |
| `sltiu rd, rs1, imm`| `0010011` | `011` | $R[rd] = (R[rs1] <_{u} \text{imm}) \,?\, 1 : 0$| Set Less Than Imm Unsigned |
| `xori rd, rs1, imm` | `0010011` | `100` | $R[rd] = R[rs1] \oplus \text{imm}$ | Bitwise XOR Immediate |
| `srli rd, rs1, shamt`| `0010011` | `101` | $R[rd] = R[rs1] \gg_{L} \text{shamt}$ | Shift Right Logical Imm |
| `srai rd, rs1, shamt`| `0010011` | `101` | $R[rd] = R[rs1] \gg_{A} \text{shamt}$ | Shift Right Arithmetic Imm |
| `ori rd, rs1, imm`  | `0010011` | `110` | $R[rd] = R[rs1] \mid \text{imm}$ | Bitwise OR Immediate |
| `andi rd, rs1, imm` | `0010011` | `111` | $R[rd] = R[rs1] \ \\& \ \text{imm}$ | Bitwise AND Immediate |
| `lw rd, offset(rs1)`| `0000011` | `010` | $R[rd] = M[R[rs1] + \text{offset}]$ | Load Word |
| `jalr rd, offset(rs1)`| `1100111`| `000` | $R[rd] = PC + 4;\ PC = (R[rs1] + \text{offset}) \ \\& \ \sim 1$ | Jump and Link Register |

### 3.3 S-Type Store Instructions
| Mnemonic | `opcode` | `funct3` | Operation | Description |
|:---|:---:|:---:|:---|:---|
| `sw rs2, offset(rs1)` | `0100011` | `010` | $M[R[rs1] + \text{offset}] = R[rs2]$ | Store Word |

### 3.4 B-Type Branch Instructions (`opcode = 1100011` / `0x63`)
| Mnemonic | `funct3` | Condition Evaluated | Description |
|:---|:---:|:---|:---|
| `beq rs1, rs2, label` | `000` | if $R[rs1] == R[rs2]$ then $PC = PC + \text{imm}$ | Branch if Equal |
| `bne rs1, rs2, label` | `001` | if $R[rs1] \neq R[rs2]$ then $PC = PC + \text{imm}$ | Branch if Not Equal |
| `blt rs1, rs2, label` | `100` | if $R[rs1] < R[rs2]$ then $PC = PC + \text{imm}$ | Branch if Less (signed) |
| `bge rs1, rs2, label` | `101` | if $R[rs1] \ge R[rs2]$ then $PC = PC + \text{imm}$ | Branch if Greater/Equal (signed) |
| `bltu rs1, rs2, label`| `110` | if $R[rs1] <_{u} R[rs2]$ then $PC = PC + \text{imm}$ | Branch if Less Unsigned |
| `bgeu rs1, rs2, label`| `111` | if $R[rs1] \ge_{u} R[rs2]$ then $PC = PC + \text{imm}$ | Branch if Greater/Equal Unsigned |

### 3.5 J-Type & U-Type Instructions
| Mnemonic | `opcode` | Operation | Description |
|:---|:---:|:---|:---|
| `jal rd, label` | `1101111` | $R[rd] = PC + 4;\ PC = PC + \text{imm}$ | Jump and Link |
| `lui rd, imm`   | `0110111` | $R[rd] = \text{imm} \ll 12$ | Load Upper Immediate |
| `auipc rd, imm` | `0010111` | $R[rd] = PC + (\text{imm} \ll 12)$ | Add Upper Imm to PC |

---

## 4. Modular Subcircuit Breakdown

The circuit is organized into clean, self-contained subcircuits within [`32_bit_computer.circ`](file:///home/ulgen/Documents/ozlem_repolar/-16-bit-computer-hardware-design/32-bit-processor/32_bit_computer.circ):

1. **`main`**:
   - Integrates the Program Counter (PC), Instruction ROM, Register File, Immediate Generator, ALU, Data RAM, Writeback Multiplexer, and Status Indicators.
   - Includes manual system clock (`SYS_CLK`) and asynchronous reset (`SYS_RESET`).
2. **`register_file`**:
   - Houses 32 independent 32-bit registers ($x0$ to $x31$).
   - A 5-to-32 decoder enables register writing when `RegWrite = 1`.
   - Dual 32-to-1 32-bit multiplexers provide dual simultaneous asynchronous read ports for `rs1` and `rs2`.
   - Dedicated wire routing channels prevent signal cross-talk or bus contention.
3. **`alu_32bit`**:
   - Implements 32-bit arithmetic and logic functions using Logisim-Evolution arithmetic primitives (`Adder`, `Subtractor`, `Shifter`, `Comparator`, gates).
   - Generates status flags: `Zero`, `Sign` (MSB), `Less` (signed), and `LessU` (unsigned).
4. **`imm_gen`**:
   - Decodes 32-bit instruction words into 32-bit sign-extended immediates based on `ImmSel` (I, S, B, U, J types).
5. **`control_unit`**:
   - Main instruction decoder taking `opcode[6:0]`, `funct3[2:0]`, and `funct7[6:0]` and generating single-cycle control signals (`RegWrite`, `ALUSrc`, `MemWrite`, `MemRead`, `MemToReg`, `ALUControl`, `Branch`, `Jump`, `JALR`, `ImmSel`, `AUIPC`).
6. **`branch_eval`**:
   - Computes conditional branch resolution based on `funct3` and ALU flags (`Zero`, `Less`, `LessU`), outputting `BranchTaken`.

---

## 5. Sample Program & Verification

The Instruction ROM is pre-loaded with a verification program testing arithmetic, memory store/load, and register writeback:

### Assembly Code:
```assembly
# RISC-V Verification Program: Arithmetic + Memory Load/Store
.text
main:
    addi x1, x0, 15        # x1 = 0 + 15 = 15      (0x0000000F)
    addi x2, x0, 27        # x2 = 0 + 27 = 27      (0x0000001B)
    add  x3, x1, x2        # x3 = 15 + 27 = 42     (0x0000002A)
    sw   x3, 4(x0)         # Mem[4] = 42           (Store into RAM[1])
    lw   x4, 4(x0)         # x4 = Mem[4] = 42      (Load back into x4)
halt:
    beq  x0, x0, 0         # Infinite loop / Halt
```

### Machine Code Hex:
```hex
v2.0 raw
00f00093
01b00113
002081b3
00302223
00402203
00000063
```

### Step-by-Step Execution Trace:
| Cycle | PC | Instruction | Mnemonic | Datapath Action | Register State After Tick |
|:---:|:---:|:---:|:---|:---|:---|
| **0** | `0x00` | `0x00F00093` | `addi x1, x0, 15` | $0 + 15 \to x1$ | $x1 = 15$ (`0x0000000F`) |
| **1** | `0x04` | `0x01B00113` | `addi x2, x0, 27` | $0 + 27 \to x2$ | $x2 = 27$ (`0x0000001B`) |
| **2** | `0x08` | `0x002081B3` | `add x3, x1, x2`  | $15 + 27 \to x3$ | $x3 = 42$ (`0x0000002A`) |
| **3** | `0x0C` | `0x00302223` | `sw x3, 4(x0)`    | $42 \to \text{RAM}[1]$ | $\text{RAM}[1] = 42$ (`0x0000002A`) |
| **4** | `0x10` | `0x00402203` | `lw x4, 4(x0)`    | $\text{RAM}[1] \to x4$ | $x4 = 42$ (`0x0000002A`) |
| **5** | `0x14` | `0x00000063` | `beq x0, x0, 0`   | Branch taken ($PC \to 0x14$) | Processor halts on self-loop |

---

## 6. How to Simulate in Logisim-Evolution

1. **Launch Logisim-Evolution**:
   ```bash
   java -jar logisim-evolution-5.0.0-all.jar 32-bit-processor/32_bit_computer.circ
   ```
2. **Observe the Circuit Layout**:
   - The top-level circuit `main` will open automatically.
   - The Instruction ROM is already pre-loaded with the verification program.
3. **Step Clock Manually**:
   - Press `Ctrl + T` (or click **Simulate -> Step Simulation**) to step each half-clock cycle.
   - Watch the PC register increment by 4 (`0x0`, `0x4`, `0x8`, `0xC`, `0x10`, `0x14`).
   - Watch `x1_ra` become `0xF`, `x2_sp` become `0x1B`, `x3_gp` become `0x2A`, and `x4_tp` become `0x2A`!
4. **Auto-Tick Execution**:
   - Press `Ctrl + K` (or check **Simulate -> Ticks Enabled**) to run continuously at your selected frequency (e.g. 1 Hz, 4 Hz, 16 Hz).
