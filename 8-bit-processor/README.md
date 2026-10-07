# 8-Bit Educational Computer Architecture

[![Logisim-Evolution](https://img.shields.io/badge/Simulator-Logisim--Evolution%20v5.0-blue.svg)](https://github.com/logisim-evolution/logisim-evolution)
[![Architecture](https://img.shields.io/badge/Architecture-8--Bit%20Custom%20ISA-orange.svg)](#-architectural-specifications)
[![Format](https://img.shields.io/badge/Format-Self--Contained%20.circ-brightgreen.svg)](#-subcircuit-hierarchy)

A fully functional, gate-level hardware implementation of an **8-bit educational computer architecture (Custom ISA)** designed in **Logisim-Evolution**. 

The entire processor—including the Arithmetic Logic Unit (ALU), multi-register bank, sequencing control logic, program counter, and RAM—is consolidated into a **single, self-contained project file** (`8_bit_computer.circ`) with zero external file dependencies and full VHDL-naming compliance for modern Logisim-Evolution environments.

---

## 📐 Architectural Specifications

| Component | Bit Width | Description |
| :--- | :---: | :--- |
| **Main Memory (RAM)** | 16 × 8-bit | 16 addressable memory words, 8-bit word length |
| **Address Bus** | 4-bit | Direct address space for $2^4 = 16$ bytes of memory |
| **Data Bus** | 8-bit | Shared bidirectional data transfer bus |
| **Program Counter (PC)** | 4-bit | Holds memory address of the next instruction to fetch |
| **Address Register (AR)** | 4-bit | Holds effective memory address for read/write access |
| **Data Register (DR)** | 4-bit | Buffer register for data transfers with memory |
| **Accumulator (AC)** | 4-bit | Primary register for arithmetic and logic operations |
| **Instruction Register (IR)**| 8-bit | Holds fetched instruction (IR[7:4] = Opcode, IR[3:0] = Address) |
| **Input Register (INPR)** | 4-bit | Latches incoming data from input peripherals |
| **Output Register (OUTR)** | 4-bit | Holds data directed to output devices |
| **Carry Flag (E)** | 1-bit | Stores carry/borrow output from ALU operations |
| **Start/Stop Flag (S)** | 1-bit | Run/Halt execution flip-flop |
| **Sequence Counter (SC)** | 4-bit | Generates timing state cycle pulses ($T_0, T_1, T_2, T_3, T_4$) |

---

## 🧩 Instruction Format

Every instruction word is 8 bits wide, organized into a 4-bit operation code and a 4-bit operand address:

```text
 7            4   3            0
+---------------+---------------+
| Opcode (4-bit)| Address (4-bit)|
+---------------+---------------+
```

* **IR[7:4] (Opcode):** Decoded by a 4-to-16 line decoder into active signals $D_0 - D_{15}$.
* **IR[3:0] (Address / Immediate):** Transferred to the Address Register ($AR$) during decode for memory access.

---

## ⚡ Instruction Set Architecture (ISA) & RTL Micro-Operations

### 1. Common Fetch & Decode Cycles

All instructions execute through a standardized 3-cycle fetch and decode sequence:

| Timing State | RTL Micro-Operations | Operation Description |
| :---: | :--- | :--- |
| **$T_0$** | $AR \leftarrow PC$ | Transfer Program Counter to Address Register |
| **$T_1$** | $IR \leftarrow M[AR], \quad PC \leftarrow PC + 1$ | Fetch instruction from RAM into $IR$; increment $PC$ |
| **$T_2$** | $D_0 \dots D_{15} \leftarrow \text{Decode}(IR[7:4]), \quad AR \leftarrow IR[3:0]$ | Decode Opcode; load target address operand into $AR$ |

---

### 2. Memory-Reference Instructions ($D_0 - D_6$)

| Mnemonic | Opcode (Hex) | Micro-Operations (RTL) | Description |
| :---: | :---: | :--- | :--- |
| **AND** | `0` | $D_0 T_3: DR \leftarrow M[AR]$<br>$D_0 T_4: AC \leftarrow AC \land DR, \quad SC \leftarrow 0$ | Bitwise AND memory word with Accumulator |
| **ADD** | `1` | $D_1 T_3: DR \leftarrow M[AR]$<br>$D_1 T_4: AC \leftarrow AC + DR, \quad E \leftarrow C_{out}, \quad SC \leftarrow 0$ | Add memory word to $AC$; update carry flag $E$ |
| **LDA** | `2` | $D_2 T_3: DR \leftarrow M[AR]$<br>$D_2 T_4: AC \leftarrow DR, \quad SC \leftarrow 0$ | Load memory word into Accumulator |
| **STA** | `3` | $D_3 T_3: M[AR] \leftarrow AC, \quad SC \leftarrow 0$ | Store Accumulator content into memory location |
| **BUN** | `4` | $D_4 T_3: PC \leftarrow AR, \quad SC \leftarrow 0$ | Branch unconditionally to specified address |
| **BSA** | `5` | $D_5 T_3: M[AR] \leftarrow PC, \quad AR \leftarrow AR + 1$<br>$D_5 T_4: PC \leftarrow AR, \quad SC \leftarrow 0$ | Branch to subroutine and save return address |
| **RET** | `6` | $D_6 T_3: PC \leftarrow M[AR], \quad SC \leftarrow 0$ | Return from subroutine by restoring $PC$ |

---

### 3. Register-Reference & I/O Instructions ($D_7 - D_{15}$)

| Mnemonic | Opcode (Hex) | Micro-Operations ($T_3$) | Description |
| :---: | :---: | :--- | :--- |
| **CLA** | `7` | $AC \leftarrow 0, \quad SC \leftarrow 0$ | Clear Accumulator to zero |
| **CLE** | `8` | $E \leftarrow 0, \quad SC \leftarrow 0$ | Clear carry flag $E$ to zero |
| **CMA** | `9` | $AC \leftarrow \overline{AC}, \quad SC \leftarrow 0$ | One's complement (invert) Accumulator bits |
| **INC** | `A` | $AC \leftarrow AC + 1, \quad SC \leftarrow 0$ | Increment Accumulator by 1 |
| **CIR** | `B` | $AC \leftarrow \text{shr } AC, \quad AC[3] \leftarrow E, \quad E \leftarrow AC[0], \quad SC \leftarrow 0$ | Circular shift right $AC$ and $E$ |
| **CIL** | `C` | $AC \leftarrow \text{shl } AC, \quad AC[0] \leftarrow E, \quad E \leftarrow AC[3], \quad SC \leftarrow 0$ | Circular shift left $AC$ and $E$ |
| **INP** | `D` | $AC \leftarrow INPR, \quad SC \leftarrow 0$ | Input peripheral data into Accumulator |
| **OUT** | `E` | $OUTR \leftarrow AC, \quad SC \leftarrow 0$ | Output Accumulator content to peripheral |
| **HLT** | `F` | $S \leftarrow 0, \quad SC \leftarrow 0$ | Halt processor execution clock |

---

## 🗂️ Subcircuit Hierarchy & Modular Design

The processor is organized cleanly across 20 self-contained subcircuits within [`8_bit_computer.circ`](8_bit_computer.circ):

1. **Top-Level Computer:**
   * `main`: Interconnects the ALU, Register Bank, 16×8 RAM, Tri-state bus buffers, and control logic.
2. **Control & Timing Logic:**
   * `main_kontrol`: Decodes timing signals ($T_0 - T_4$) and opcode lines ($D_0 - D_{15}$).
   * `register_kontrol`: Multiplexes bus control lines and load/increment/clear signals.
   * `sequence_counter`: 4-bit timing counter tracking micro-operation execution.
   * `sc_clear`: Combinational reset logic clearing the sequence counter ($SC \leftarrow 0$) at cycle end.
3. **Dedicated Register Controllers:**
   * `AR`, `PC`, `DR`, `AC`, `OUTR`, `IR`: Individual control logic generating `LOAD`, `INC`, and `CLR` pulses.
   * `mem_control`: Read and Write strobing for RAM.
   * `e_flag_control` (`E_kontrol`): Carry flag set/clear logic for arithmetic and shift instructions.
   * `ALU`: Control selector lines for arithmetic/logic functions.
4. **Datapath & Functional Units:**
   * `register_4bit`: 4-bit parallel-load register with synchronous enable.
   * `instruction_register`: 8-bit instruction register built from paired 4-bit stages.
   * `alu_4bit_carry`: 4-bit adder and logic unit with carry lookahead logic.
   * `alu_4bit` & `alu_1bit`: Modular 1-bit and 4-bit bitwise logic blocks (AND, ADD, CMA, shifts).
   * `E`: Dedicated D/SR flip-flop stage storing the carry output.

---

## 🧪 Sample Test Program

A sample program calculating $3 + 2 = 5$ and storing the result back into RAM:

### Assembly & Machine Code Walkthrough
```text
Address  Machine Code  Assembly     Operation Description
-------------------------------------------------------------------------------
0x0      27            LDA 0x7      ; Load operand A from RAM[0x7] (value: 3) into AC
0x1      18            ADD 0x8      ; Add operand B from RAM[0x8] (value: 2) to AC (AC = 5)
0x2      39            STA 0x9      ; Store result (5) into RAM[0x9]
0x3      FF            HLT          ; Halt computer execution
0x7      03            DATA 3       ; Constant operand A
0x8      02            DATA 2       ; Constant operand B
0x9      00            RESULT       ; Memory location holding computed sum (0x05)
```

### Ready-to-Load Logisim Raw Hex Code
To load this program into the RAM in Logisim-Evolution, save the following block into a text file (e.g., `program.hex`), or right-click the RAM module in Logisim and enter the values:

```text
v2.0 raw
27 18 39 ff 0 0 0 3 2
```

---

## 🚀 How to Run and Simulate

1. **Launch Logisim-Evolution:**
   ```bash
   java -jar logisim-evolution-5.0.0-all.jar
   ```
2. **Open Project:**
   * Navigate to `File > Open...` and select [`8_bit_computer.circ`](8_bit_computer.circ).
3. **Load Program into RAM:**
   * In the `main` circuit, locate the **RAM** component.
   * Right-click the RAM module and choose **Load Image...**, then select your saved `.hex` file.
   * *Alternatively:* Double-click the RAM component to open its hex editor and manually type the values:
     * `Addr 0`: `27`, `Addr 1`: `18`, `Addr 2`: `39`, `Addr 3`: `FF`, `Addr 7`: `03`, `Addr 8`: `02`.
4. **Run Simulation:**
   * Enable the clock generator: `Simulate > Ticks Enabled` (`Ctrl + K`).
   * Set tick frequency to 4 Hz or 8 Hz via `Simulate > Tick Frequency`.
   * To step cycle-by-cycle manually, use `Simulate > Tick Once` (`Ctrl + T`).
   * **Verification:** Observe that once the execution reaches `0x3` (`HLT`), the Accumulator ($AC$) holds `5`, and RAM location `0x9` contains `05`.
