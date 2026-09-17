# RISC-V CV32E40P Verification Plan

This document contains all the verification plan tables structured identically to the reference example, tailored to be straightforward for a standard UVM environment without over-engineering, and strictly within the scope of Vanilla RV32I & RV32M.

## 1. System Plan

| ID | Design Requirement Description |
| :--- | :--- |
| `risc_sys_0` | Verify core ports and connections |
| `risc_sys_1` | Output behavior when reset is asserted (should be low/inactive) |
| `risc_sys_2` | Pipeline hazards (RAW, Load-Use, Control Flushes) |
| `risc_sys_3` | Exceptions (Illegal Instruction for unsupported/undecodable opcodes and reserved shifts) |
| `risc_sys_4` | Misaligned memory access (Instruction Fetch and Data Load/Store) |
| `risc_sys_5` | fetch_enable_i behavior (Core should not fetch instructions until enabled) |
| `risc_sys_6` | Core stalls when slave (Memory) delays grant or valid signals |
| `risc_sys_7` | Master stalls (Handling when the core holds or drops OBI requests) |
| `risc_sys_8` | Instruction Fetch (IF) OBI handshake |
| `risc_sys_9` | Load/Store Unit (LSU) OBI handshake |

---

## 2. Checking Plan

| ID | Feature | Checking Method / Monitor |
| :--- | :--- | :--- |
| `chk_sys_01` | Reset behavior | Monitor: Check that all outputs are inactive during reset. |
| `chk_sys_02` | OBI Protocol | Monitor: Validate basic OBI handshake rules (req, gnt, rvalid). |
| `chk_alu_01` | RV32I ALU Operations | Scoreboard: Compare core result with a reference golden model. |
| `chk_m_01` | MUL/DIV normal operation | Scoreboard: Verify correct 32-bit product/quotient/remainder. |
| `chk_m_02` | DIV corner cases | Scoreboard: Verify ISA default values for divide-by-zero and overflow. |
| `chk_haz_01` | Pipeline Stalls | Monitor: Ensure IF/ID stages hold data during multi-cycle EX operations. |
| `chk_exc_01` | Misaligned / Illegal | Scoreboard: Verify core jumps to trap address without corrupting memory/registers. |

---

## 3. Coverage Plan

| ID | Coverpoint / Cross | Description |
| :--- | :--- | :--- |
| `cov_instr_01` | RV32I Instructions | Cover all base integer instructions. |
| `cov_instr_02` | RV32M Instructions | Cover all multiply, divide, and remainder instructions. |
| `cov_regs_01` | Register Access | Cover all 32 general-purpose registers as sources and destinations. |
| `cov_obi_01` | OBI Latency | Cover immediate memory response vs delayed response (wait states). |
| `cov_exc_01` | Exceptions | Cover Illegal Instructions (unimplemented/reserved) and Misaligned Addresses. |

---

## 4. Generation Plan

| ID | Sequence | Description |
| :--- | :--- | :--- |
| `gen_sys_01` | `sys_reset_sequence` | Generates reset signal and basic fetch enable. |
| `gen_sys_02` | `obi_memory_sequence` | Generates normal memory responses and randomized wait states. |
| `gen_alu_01` | `riscv_alu_sequence` | Generates random R-type and I-type integer instructions. |
| `gen_mem_01` | `riscv_lsu_sequence` | Generates Load/Store instructions with aligned and misaligned addresses. |
| `gen_ctrl_01` | `riscv_branch_sequence` | Generates conditional branches and unconditional jumps. |
| `gen_m_01` | `riscv_mul_div_sequence` | Generates multiply/divide instructions including zero and extreme values. |
| `gen_exc_01` | `riscv_illegal_sequence` | Generates undecodable opcodes and reserved shifts. |

---

## 5. Test List

| test name | sequance | run stimuls generated |
| :--- | :--- | :--- |
| `sys_reset_test` | `sys_reset_sequence` | Apply reset for a random number of cycles, then release. |
| `sys_obi_stall_test` | `obi_memory_sequence` | Generate random delays (wait states) on memory grant and valid signals. |
| `alu_r_type_test` | `riscv_alu_sequence` | Generate only R-type instructions (ADD, SUB, SLL, etc.) with random registers. |
| `alu_i_type_test` | `riscv_alu_sequence` | Generate only I-type instructions (ADDI, SLTI, etc.) with random immediates. |
| `branch_jump_test` | `riscv_branch_sequence` | Generate branches (BEQ, BNE, etc.) and jumps (JAL, JALR) with random offsets. |
| `load_store_test` | `riscv_lsu_sequence` | Generate Load and Store instructions targeting aligned memory addresses. |
| `mul_div_test` | `riscv_mul_div_sequence` | Generate MUL and DIV instructions, occasionally forcing division by zero or overflow. |
| `misaligned_exc_test` | `riscv_lsu_sequence` | Generate Load/Store instructions specifically targeting addresses not aligned to 4 bytes. |
| `illegal_instr_test` | `riscv_illegal_sequence` | Generate instructions with unimplemented opcodes, CSR instructions, and reserved shift encodings. |
| `hazard_test` | `riscv_hazard_sequence` | Generate back-to-back instructions that depend on each other's registers (RAW hazards). |
