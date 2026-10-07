# 16-Bit Mano Machine Computer Architecture

[![Logisim-Evolution](https://img.shields.io/badge/Simulator-Logisim--Evolution%20v5.0-blue.svg)](https://github.com/logisim-evolution/logisim-evolution)
[![Architecture](https://img.shields.io/badge/Architecture-16--Bit%20Mano%20Machine-orange.svg)](#-architectural-overview)
[![Format](https://img.shields.io/badge/Format-Self--Contained%20.circ-brightgreen.svg)](#-subcircuit-organization-30-modules)

A complete, faithful gate-level hardware implementation of **M. Morris Mano's Classic Basic Computer (Mano Machine)** from *Computer System Architecture* (3rd Edition).

The entire system—including the 16-bit Common Bus, Arithmetic Logic Unit (ALU), 4096 × 16 RAM, register file, and microprogrammed timing and control decoders—is implemented as a **single, self-contained project file** ([`16_bit_computer.circ`](16_bit_computer.circ)) containing 30 modular subcircuits natively formatted for modern **Logisim-Evolution**.

---

## 🏛️ Architectural Overview

| Component | Bit Width | Functional Description |
| :--- | :---: | :--- |
| **Main Memory (RAM)** | 4096 × 16-bit | 4K words of memory with 16-bit data width |
| **Common Bus** | 16-bit | 16-bit shared bus driven by 3-state Controlled Buffers |
| **Address Register (AR)** | 12-bit | Holds memory address (0 to 4095) for read/write operations |
| **Program Counter (PC)** | 12-bit | Holds address of next instruction to be fetched |
| **Data Register (DR)** | 16-bit | Holds operand read from or written to memory |
| **Accumulator (AC)** | 16-bit | General-purpose processing and arithmetic register |
| **Instruction Register (IR)**| 16-bit | Holds the current instruction code being executed |
| **Temporary Register (TR)** | 16-bit | Holds temporary data during multi-step transfers |
| **Input Register (INPR)** | 8-bit | Holds 8-bit character data from input device |
| **Output Register (OUTR)** | 8-bit | Holds 8-bit character data directed to output device |
| **Sequence Counter (SC)** | 4-bit | Decoded into timing signals $T_0 - T_{15}$ |
| **Extended Flip-Flop (E)** | 1-bit | Carry/overflow flag for ALU and circular shifts |
| **Interrupt Request (R)** | 1-bit | Signals an active interrupt cycle |
| **Interrupt Enable (IEN)** | 1-bit | Masks or unmasks hardware interrupt handling |
| **I/O Flags (FGI / FGO)** | 1-bit | Input ready and Output ready peripheral status flags |
| **Start/Stop Flag (S)** | 1-bit | Clock run enable / halt latch |

---

## 🗂️ Decoders & Control Unit

* **3-to-8 Line Opcode Decoder ($D_0 - D_7$):** Decodes instruction bits `IR[14:12]` to identify the active operation.
* **4-to-16 Line Timing Decoder ($T_0 - T_{15}$):** Decodes the Sequence Counter ($SC$) outputs into discrete clock cycles for micro-operation execution.

---

## 🧩 Instruction Word Formats

The computer uses three distinct 16-bit instruction formats:

### 1. Memory-Reference Format
```text
 15   14      12 11                                 0
+---+-----------+------------------------------------+
| I |  Opcode   |          Address (12-bit)          |
+---+-----------+------------------------------------+
```
* **Bit 15 ($I$):** Addressing mode selector:
  * $I = 0$: Direct addressing (effective address is given in bits 11–0).
  * $I = 1$: Indirect addressing (bits 11–0 point to a pointer address holding the effective address).
* **Bits 14–12 (Opcode):** Operation codes `000` through `110` ($D_0 - D_6$).

### 2. Register-Reference Format ($D_7 = 1, I = 0$)
* Operation code is `0111` (Hex prefix `7`). Bits 11–0 specify which register operation is performed.

### 3. Input-Output Format ($D_7 = 1, I = 1$)
* Operation code is `1111` (Hex prefix `F`). Bits 11–0 specify the I/O or interrupt operation.

---

## ⚡ Complete Instruction Set Architecture (ISA)

### 1. Memory-Reference Instructions

| Symbol | Hex ($I=0$) | Hex ($I=1$) | RTL Micro-Operation Description |
| :---: | :---: | :---: | :--- |
| **AND** | `0xxx` | `8xxx` | $AC \leftarrow AC \land M[AR]$ (Bitwise logical AND) |
| **ADD** | `1xxx` | `9xxx` | $AC \leftarrow AC + M[AR], \quad E \leftarrow C_{out}$ (Addition with carry) |
| **LDA** | `2xxx` | `Axxx` | $AC \leftarrow M[AR]$ (Load memory into Accumulator) |
| **STA** | `3xxx` | `Bxxx` | $M[AR] \leftarrow AC$ (Store Accumulator into memory) |
| **BUN** | `4xxx` | `Cxxx` | $PC \leftarrow AR$ (Branch unconditionally) |
| **BSA** | `5xxx` | `Dxxx` | $M[AR] \leftarrow PC, \quad PC \leftarrow AR + 1$ (Branch and save return address) |
| **ISZ** | `6xxx` | `Exxx` | $M[AR] \leftarrow M[AR] + 1$, if $M[AR] = 0$ then $PC \leftarrow PC + 1$ |

---

### 2. Register-Reference Instructions (`7xxx`)

Executed at timing state $T_3$ when $D_7 = 1$ and $I = 0$:

| Symbol | Hex Code | RTL Micro-Operation | Description |
| :---: | :---: | :--- | :--- |
| **CLA** | `7800` | $AC \leftarrow 0$ | Clear Accumulator |
| **CLE** | `7400` | $E \leftarrow 0$ | Clear carry bit |
| **CMA** | `7200` | $AC \leftarrow \overline{AC}$ | Complement (invert) Accumulator |
| **CME** | `7100` | $E \leftarrow \overline{E}$ | Complement carry bit |
| **CIR** | `7080` | $AC \leftarrow \text{shr } AC, \quad AC[15] \leftarrow E, \quad E \leftarrow AC[0]$ | Circular shift right $AC$ and $E$ |
| **CIL** | `7040` | $AC \leftarrow \text{shl } AC, \quad AC[0] \leftarrow E, \quad E \leftarrow AC[15]$ | Circular shift left $AC$ and $E$ |
| **INC** | `7020` | $AC \leftarrow AC + 1$ | Increment Accumulator by 1 |
| **SPA** | `7010` | If $AC[15] = 0$ then $PC \leftarrow PC + 1$ | Skip next instruction if $AC$ is positive |
| **SNA** | `7008` | If $AC[15] = 1$ then $PC \leftarrow PC + 1$ | Skip next instruction if $AC$ is negative |
| **SZA** | `7004` | If $AC = 0$ then $PC \leftarrow PC + 1$ | Skip next instruction if $AC$ is zero |
| **SZE** | `7002` | If $E = 0$ then $PC \leftarrow PC + 1$ | Skip next instruction if $E$ is zero |
| **HLT** | `7001` | $S \leftarrow 0$ | Halt computer clock |

---

### 3. Input-Output Instructions (`Fxxx`)

Executed at timing state $T_3$ when $D_7 = 1$ and $I = 1$:

| Symbol | Hex Code | RTL Micro-Operation | Description |
| :---: | :---: | :--- | :--- |
| **INP** | `F800` | $AC[7:0] \leftarrow INPR, \quad FGI \leftarrow 0$ | Transfer input character to Accumulator |
| **OUT** | `F400` | $OUTR \leftarrow AC[7:0], \quad FGO \leftarrow 0$ | Transfer Accumulator character to output |
| **SKI** | `F200` | If $FGI = 1$ then $PC \leftarrow PC + 1$ | Skip if input flag is set |
| **SKO** | `F100` | If $FGO = 1$ then $PC \leftarrow PC + 1$ | Skip if output flag is set |
| **ION** | `F080` | $IEN \leftarrow 1$ | Turn interrupt on (enable) |
| **IOF** | `F040` | $IEN \leftarrow 0$ | Turn interrupt off (disable) |

---

## 🗂️ Subcircuit Organization (30 Modules)

All modular components are organized in [`16_bit_computer.circ`](16_bit_computer.circ):

1. **System & Datapath:**
   * `main`: Top-level processor schematic (RAM, Common Bus, all registers, ALU, probes, control logic).
   * `alu_16bit`: Full 16-bit ALU supporting ADD, AND, CMA, and bit-level shifts.
   * `alu_1bit`: 1-bit arithmetic/logic slice replicated across the ALU datapath.
   * `carry_logic`: Carry propagation and circular shifting stage.
2. **Registers:**
   * `register_16bit`: 16-bit parallel load register (used for DR, TR, AC, IR).
   * `register_12bit`: 12-bit register (used for PC, AR).
   * `register_8bit`: 8-bit register (used for INPR, OUTR).
   * `register_4bit`: 4-bit building block register.
3. **Control & Sequencers:**
   * `main_control`: Top-level control unit routing decoder outputs to functional units.
   * `register_kontrol`: Multiplexes bus select and load enable lines.
   * `FLAG_KONTROL`, `FGO_FF`, `FGI_FF`, `IEN_FF`: Flip-flop latches for I/O and interrupt flags.
   * `counter`: 4-bit synchronous sequence counter.
   * `SC_kontrol`: Sequence counter reset and increment logic.
   * `AR_kontrol`, `PC_kontrol`, `PC_KONTROL_BUF`, `DR_kontrol`, `OUTR_kontrol`, `IR_kontrol`, `TR_kontrol`, `AC_kontrol`, `ALU_kontrol`, `R_kontrol`, `e_control`, `mem_control`, `S_kontrol`, `I_kontrol`: Dedicated micro-operation control circuits.

## 🧪 Sample Test Program

A sample Mano Machine program calculating $83 + 41 = 124$ (`0x0053 + 0x0029 = 0x007C`) and storing the resulting sum back into RAM:

### Assembly & Machine Code Walkthrough
```text
Memory Address  Machine Code (Hex)  Assembly Instruction  Operation Description
---------------------------------------------------------------------------------------------------------
0x000           2004                LDA 0x004             ; Load operand A from RAM[0x004] (0x0053 = 83) into AC
0x001           1005                ADD 0x005             ; Add operand B from RAM[0x005] (0x0029 = 41) to AC
0x002           3006                STA 0x006             ; Store computed sum into RAM[0x006] (Result = 0x007C = 124)
0x003           7001                HLT                   ; Halt computer clock
0x004           0053                DATA 0x0053           ; Constant operand A (Decimal 83)
0x005           0029                DATA 0x0029           ; Constant operand B (Decimal 41)
0x006           0000                RESULT 0x0000         ; Memory location reserved for result (0x007C)
```

### Ready-to-Load Logisim Raw Hex Code
To load this program into the RAM in Logisim-Evolution, save the following block into a text file (e.g., `program.hex`), or right-click the RAM module in Logisim and load/type the values:

```text
v2.0 raw
2004 1005 3006 7001 0053 0029 0000
```

---

## 🚀 How to Run and Simulate

1. **Launch Logisim-Evolution:**
   ```bash
   java -jar logisim-evolution-5.0.0-all.jar
   ```
2. **Open Project:**
   * Select `File > Open...` and choose [`16_bit_computer.circ`](16_bit_computer.circ).
3. **Load Program into RAM:**
   * In the `main` circuit, locate the **RAM** component (4096 × 16).
   * Right-click the RAM component and select **Load Image...**, then choose your saved `.hex` file.
   * *Alternatively:* Double-click the RAM component to open its hex editor and manually type the hex values into addresses `000` to `005`.
4. **Explore Schematics:**
   * In the project explorer tree on the left, double-click `main` to inspect the full computer.
   * Explore sub-units such as `alu_16bit` or `main_control` to inspect gate-level implementations.
5. **Run Simulation:**
   * Enable clock generation: `Simulate > Ticks Enabled` (`Ctrl + K`).
   * Set tick frequency to 8 Hz or 16 Hz via `Simulate > Tick Frequency`.
   * To step cycle-by-cycle manually, use `Simulate > Tick Once` (`Ctrl + T`).
   * **Verification:** When execution reaches address `0x003` (`HLT`), observe that Accumulator ($AC$) holds `007C` (124 decimal) and RAM location `0x006` contains `007C`.
