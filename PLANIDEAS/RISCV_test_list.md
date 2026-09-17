# RISC-V Test List

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
