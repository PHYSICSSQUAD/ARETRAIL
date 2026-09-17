# CV32E40P Core Verification Plan (RV32I & RV32M)

## 1. Introduction and Scope
This document outlines the detailed verification plan for the CV32E40P RISC-V core. In accordance with the project guidelines, the scope is strictly limited to:
*   **RV32I (Base Integer Instruction Set)**
*   **RV32M (Standard Extension for Integer Multiplication and Division)**
*   **Simple Exceptions** (Specifically, Misaligned Memory Accesses)

**Out of Scope:** FPU (RV32F), Compressed Instructions (RV32C), PULP Custom Extensions, COREV Clusters, CSRs, Interrupts, and System/Debug Instructions (EBREAK, ECALL, MRET).

---

## 2. Interface and Signal Flow

The core operates as a black-box at the top level for this verification, but we monitor the datapath using the following Open Bus Interfaces (OBI) and internal pipeline behavior:

### Instruction Fetch (OBI IF)
*   **Output Signals:** `instr_req_o` (Fetch Request), `instr_addr_o` (Address of Instruction)
*   **Input Signals:** `instr_gnt_i` (Memory Granted Request), `instr_rvalid_i` (Data is Valid), `instr_rdata_i` (Fetched Instruction Data - 32-bit)
*   **Flow:** The `IF` stage drives `instr_req_o` and `instr_addr_o`. Once granted (`instr_gnt_i`), it waits for `instr_rvalid_i` to latch `instr_rdata_i` into the Instruction Decode (`ID`) stage.

### Data Memory Access (OBI LSU)
*   **Output Signals:** `data_req_o` (Access Request), `data_addr_o` (Memory Address), `data_we_o` (Write Enable: 1 for Write, 0 for Read), `data_be_o` (Byte Enable 4-bit), `data_wdata_o` (Write Data)
*   **Input Signals:** `data_gnt_i` (Memory Granted Request), `data_rvalid_i` (Data is Valid), `data_rdata_i` (Read Data)
*   **Flow:** Generated in the `EX` stage and executed by the LSU. Loads wait for `data_rvalid_i` to write back to the Register File (`WB` stage).

### Pipeline Flow (General)
1.  **IF Stage:** Fetches instruction via OBI IF.
2.  **ID Stage:** Decodes instruction, reads operands from the Register File (Ports A & B).
3.  **EX Stage:** Executes ALU/MAC operations, evaluates branch conditions, or computes memory addresses.
4.  **WB Stage:** Writes the result back to the Register File (Port C).

---

## 3. RV32I: Integer Computational Instructions

These instructions process data entirely within the ALU and Register File. They do not access data memory.

### 3.1. R-Type Instructions (ADD, SUB, SLL, SLT, SLTU, XOR, SRL, SRA, OR, AND)
*   **Description:** Operations between two source registers (`rs1`, `rs2`), storing the result in a destination register (`rd`).
*   **Input Data:** `instr_rdata_i` containing the specific opcode, `rs1`, `rs2`, and `rd` fields. Values of `rs1` and `rs2` pre-loaded in the Register File.
*   **Flow:** 
    *   **IF:** Fetches the instruction.
    *   **ID:** Reads `rs1` and `rs2` from the Register File.
    *   **EX:** ALU performs the arithmetic/logical operation.
    *   **WB:** Result is written back to `rd` in the Register File.
*   **Example (ADD):**
    *   **Input:** `rs1` (x1) = 0x0000_0005, `rs2` (x2) = 0x0000_000A. Instruction: `ADD x3, x1, x2`
    *   **Output:** Register `x3` is updated to 0x0000_000F in the WB stage. No external data bus activity.

### 3.2. I-Type Instructions (ADDI, SLTI, SLTIU, XORI, ORI, ANDI, SLLI, SRLI, SRAI)
*   **Description:** Operations between one source register (`rs1`) and a sign-extended 12-bit immediate, storing the result in `rd`.
*   **Flow:** Similar to R-Type, but the ID stage extracts the immediate value from `instr_rdata_i` and passes it to the EX stage instead of a second register read.
*   **Example (ADDI):**
    *   **Input:** `rs1` (x5) = 0xFFFF_FFF0. Instruction: `ADDI x6, x5, 15` (Imm = 0x00F).
    *   **Output:** Register `x6` is updated to 0xFFFF_FFFF.

### 3.3. U-Type Instructions (LUI, AUIPC)
*   **Description:** Load Upper Immediate (`LUI`) places a 20-bit immediate in the top 20 bits of `rd`. Add Upper Immediate to PC (`AUIPC`) adds a 20-bit immediate to the current PC and stores it in `rd`.
*   **Flow:** 
    *   **ID:** Extracts the 20-bit immediate.
    *   **EX:** (For AUIPC) ALU adds Immediate to PC. (For LUI) ALU passes Immediate directly.
    *   **WB:** Writes to `rd`.
*   **Example (LUI):**
    *   **Input:** Instruction: `LUI x7, 0x12345`
    *   **Output:** Register `x7` is updated to 0x1234_5000.

---

## 4. RV32I: Control Transfer Instructions

These instructions alter the Program Counter (PC).

### 4.1. Unconditional Jumps (JAL, JALR)
*   **Description:** Jumps to a target address and saves the return address (PC + 4) in `rd`.
*   **Flow:**
    *   **IF:** Fetches JAL/JALR.
    *   **ID:** Reads `rs1` (for JALR).
    *   **EX:** Calculates target address (PC + offset for JAL, `rs1` + offset for JALR). Flushes the IF stage (Pipeline Flush). Computes Return Address (PC + 4).
    *   **WB:** Writes Return Address to `rd`.
    *   **IF (Next Cycle):** `instr_addr_o` updates to the new calculated target address.
*   **Example (JALR):**
    *   **Input:** PC = 0x100, `rs1` (x1) = 0x200. Instruction: `JALR x0, x1, 0x10`
    *   **Output:** EX stage updates PC to 0x210. Next `instr_addr_o` = 0x210. Pipeline is flushed.

### 4.2. Conditional Branches (BEQ, BNE, BLT, BGE, BLTU, BGEU)
*   **Description:** Compares `rs1` and `rs2`. If the condition is met, PC branches to (PC + offset).
*   **Flow:**
    *   **ID:** Reads `rs1` and `rs2`.
    *   **EX:** ALU performs comparison. If taken, calculates target address, flushes IF, and updates PC. If not taken, execution continues normally.
*   **Example (BEQ - Taken):**
    *   **Input:** PC = 0x100, `rs1` (x1) = 5, `rs2` (x2) = 5. Instruction: `BEQ x1, x2, 0x20`
    *   **Output:** Condition true. IF flushed. Next `instr_addr_o` = 0x120. No WB to register file.

---

## 5. RV32I: Load/Store Instructions

These instructions interact with the OBI Data interface (LSU).

### 5.1. Store Instructions (SB, SH, SW)
*   **Description:** Stores a byte (SB), half-word (SH), or word (SW) from `rs2` to memory at address (`rs1` + offset).
*   **Flow:**
    *   **ID:** Reads `rs1` (base address) and `rs2` (data).
    *   **EX:** ALU computes absolute memory address.
    *   **LSU/OBI:** Drives `data_req_o` = 1, `data_addr_o` = computed address, `data_we_o` = 1, `data_be_o` = depends on size (e.g., 4'b1111 for SW, 4'b0011 for SH), `data_wdata_o` = `rs2` aligned to byte lanes.
    *   **Wait:** Stalls pipeline if `data_gnt_i` or `data_rvalid_i` (if write response is checked) is delayed.
*   **Example (SW):**
    *   **Input:** `rs1` (x1) = 0x1000, `rs2` (x2) = 0xDEADBEEF. Instruction: `SW x2, 0x4(x1)`
    *   **Output:** `data_req_o` = 1, `data_addr_o` = 0x1004, `data_we_o` = 1, `data_be_o` = 4'b1111, `data_wdata_o` = 0xDEADBEEF.

### 5.2. Load Instructions (LB, LH, LW, LBU, LHU)
*   **Description:** Loads data from memory address (`rs1` + offset) into `rd`.
*   **Flow:**
    *   **ID:** Reads `rs1`.
    *   **EX:** ALU computes memory address.
    *   **LSU/OBI:** Drives `data_req_o` = 1, `data_addr_o` = computed address, `data_we_o` = 0. 
    *   **Wait:** Stalls EX/WB stages until memory responds with `data_rvalid_i` = 1 and `data_rdata_i`.
    *   **WB:** Sign/Zero extends the read data and writes it to `rd`.
*   **Example (LW):**
    *   **Input:** `rs1` (x1) = 0x2000. Instruction: `LW x3, 0x0(x1)`. Memory returns `data_rdata_i` = 0xAABBCCDD.
    *   **Output:** `data_addr_o` = 0x2000. Upon valid read, Register `x3` is updated to 0xAABBCCDD.

---

## 6. RV32M: Multiply and Divide Instructions

These are multi-cycle instructions executed in the EX stage.

### 6.1. Multiplication (MUL, MULH, MULHSU, MULHU)
*   **Description:** Multiplies `rs1` and `rs2`. `MUL` stores lower 32-bits. `MULH*` store upper 32-bits.
*   **Flow:**
    *   **ID:** Reads `rs1` and `rs2`.
    *   **EX:** The MAC (Multiply-Accumulate) unit operates for multiple cycles (depending on multiplier implementation). Stalls the pipeline until calculation is complete.
    *   **WB:** Result written to `rd`.
*   **Example (MUL):**
    *   **Input:** `rs1` (x1) = 2, `rs2` (x2) = 3. Instruction: `MUL x3, x1, x2`
    *   **Output:** EX stalls for a few cycles, then Register `x3` is updated to 6.

### 6.2. Division & Remainder (DIV, DIVU, REM, REMU)
*   **Description:** Performs integer division or remainder.
*   **Flow:**
    *   **ID:** Reads `rs1` (dividend) and `rs2` (divisor).
    *   **EX:** The Divider unit initiates an iterative division process (can take up to ~32 cycles). The pipeline is completely stalled.
    *   **WB:** Result written to `rd`.
    *   **Corner Cases:** Division by zero (returns -1 for DIV, sets REM to dividend). Overflow (e.g., INT_MIN / -1).

---

## 7. Exception Verification (Misaligned Accesses)

As per guidelines, only simple exceptions like misaligned memory access need verification.

### 7.1. Misaligned Load/Store
*   **Scenario:** Attempting to execute a Word operation (LW/SW) on an address not divisible by 4, or a Half-word operation (LH/SH) on an odd address.
*   **Flow:**
    *   **EX/LSU:** Computes address. Detects misalignment.
    *   **Action:** The LSU cancels the OBI `data_req_o`. It triggers a Trap inside the core.
    *   **Trap Handling:** PC jumps to `mtvec_addr_i` (Trap Handler Base Address). The Instruction Fetch OBI will immediately request the instruction at `mtvec_addr_i`.
*   **Example (Misaligned SW):**
    *   **Input:** `rs1` (x1) = 0x1001 (Unaligned). Instruction: `SW x2, 0(x1)`
    *   **Output:** No OBI data request is sent. `instr_addr_o` updates to the trap vector address (`mtvec_addr_i`).

### 7.2. Misaligned Instruction Fetch
*   **Scenario:** Executing a branch or jump to an address not aligned to a 32-bit boundary (since Compressed instructions RV32C are out of scope, all instructions must be 32-bit aligned).
*   **Flow:**
    *   **EX:** Branch taken to an unaligned address (e.g., 0x1002).
    *   **Action:** Triggers an Instruction Address Misaligned Exception. IF stage is flushed, no OBI `instr_req_o` is made for the unaligned address. PC jumps to `mtvec_addr_i`.
