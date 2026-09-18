# risc_v (CV32E40P) — Final Verification Plan
**QRE Final Project 2026 — RV32I + M top-level verification of `cv32e40p_top`**

This folder contains the final Verification Plan, generated **strictly on the
structure of the provided template** (`plantemplet/*.html`). The HTML skeleton of
every sheet (CSS link, inline `.ritz .waffle` style block, 26-column thead,
title row, header row, row-header markup, cell classes) is copied verbatim from
the template; only the data rows and the project title (`risc_v`) are new.

| Sheet | Columns (exactly as in the template) | Rows |
|---|---|---|
| `System.html` | ID · Design Requirement Description · (note) · component name | 55 requirements |
| `Generation_.html` | ID · Design Requirement Description · Generation · component/Object name | 42 rows |
| `Checking_.html` | ID · Design Requirement Description · check · property name · comment | 45 rows |
| `Coverage_.html` | ID · Design Requirement Description · Coverage · covergroup | 40 rows |
| `test list.html` | test name · sequance run · stimuls generated | 27 tests |

**Traceability model (as in the template):** the `System` sheet is the
requirement inventory; rows in `Generation_` / `Checking_` / `Coverage_` reuse the
**same ID and repeat the requirement description verbatim**, so every
stimulus/check/covergroup traces back 1:1 to a System requirement. Verified:
all gen/chk/cov IDs exist in System; zero description mismatches.

Rebuild after editing the master requirement table:

```bash
python3 build_plan.py
```

---

## Sources used (and their role)

| Source | Role |
|---|---|
| `DATABOOK/` (CV32E40P user manual, v2) | Authoritative specification |
| `rtl/*.sv` (actual core RTL) | Authoritative implementation details (all timing/protocol/hazard/misaligned claims in the plan were individually verified against the RTL) |
| `guidelines/Questions QRE Final project status 2026/Team {1..6}.html` | Verification scope & methodology constraints |
| `ourplan(on progress)/Verification plan on progress/` | Starting point — preserved and massively extended (was: 10 system / 9 gen / 9 chk / 9 cov / 11 test rows) |
| `plantemplet/` | Authoritative format |
| `parameters/parameters.png` | DUT parameter set (all defaults) |
| `arcitecture/uvm_architecture_pro_updated.*` | Agent/component naming used in the plan (`sys_ctrl_agent`, `if_agt`, `lsu_agt`, `if_pas_mon`, `lsu_pas_mon`, `rvfi_pas_mon`, `sb`, SVA bind module) |
| `standard/riscv-spec-20191213.pdf` | ISA semantics for the golden RV32IM reference model (div-by-zero, overflow, SLTIU, …) |
| `بلان ايديا` | **⚠ NOT FOUND IN THE REPO** — folder was referenced but is not present in the current checkout. If it is added, every idea in it must be validated/merged (see below). |

## Scope (from the guidelines — binding)

- Verify **Vanilla RV32I + RV32M only**; everything else gets the minimal work
  that keeps the core running normally (T1 Q3, T4 Q3, T5 Q3).
- **Top-level** verification; scoreboards at major-group granularity (T2 Q2).
- The whole TB — including instruction/data memory models — is ours (T1 Q5).
- Build everything in-house; no third-party reference models (T2 Q7).
- Misaligned memory behavior is worth verifying; ecall/ebreak/hints are not
  (T2 Q5, T4 Q2). No CSR/interrupt verification (T3).

## Key engineering findings that shaped the plan

1. **Misaligned accesses are NOT exceptions in this core.**
   The in-progress plan (`chk_exc_01`, `exception_cg`) framed misaligned
   load/store as a trap to `mtvec`. The databook LSU chapter states the LSU
   *never* raises address-misaligned exceptions — accesses are split into two
   OBI transactions (lowest address first). This is confirmed in
   `rtl/cv32e40p_load_store_unit.sv` (be-pattern tables, `data_misaligned_o`,
   `rdata_q` merge). The plan therefore checks **split/order/byte-enable/data
   merge** (`risc_lsu_03..05`) plus an anti-check that no trap occurs
   (`risc_lsu_04` / `sb_no_exc_on_misaligned`).

2. **Open guideline questions answered directly from the RTL:**
   - *T2 Q9 (misaligned handling?)* → split into 2 transactions, lowest first,
     never an exception (see 1).
   - *T2 Q10 (all unimplemented encodings illegal? incl. reserved shifts)* →
     yes; `cv32e40p_decoder.sv` raises `illegal_insn` for every undefined
     encoding incl. funct3=001/101 with `imm[11:5]` ∉ {0000000, 0100000}
     (categories (a)–(f) in `risc_dec_02`).
   - *T4 Q4 (multicycle MULH/DIV in EX + taken-branch flush → write-back?)* →
     the multicycle instruction is architecturally *older* than the branch; it
     completes and writes back normally; only younger (IF/ID) instructions are
     flushed (`risc_m_04`).
   - *T4 Q6 (load in WB vs younger ALU in EX writing same rd — who wins?)* →
     reachable scenario; `register_file_ff` gives port **b (EX, younger)**
     priority over port **a (WB, load)**, which is the architecturally correct
     ordering. Scoreboard expects the younger result (`risc_haz_03` /
     `sb_waw_younger_wins`).

3. **Timing facts verified against RTL** (used in checks/coverage):
   OBI TRANSPARENT→REGISTERED request-stability FSM; prefetch `DEPTH=2`
   (≤2 outstanding on IF); data-side ≤2 outstanding (`cnt_q`); MUL=1 cycle,
   MULH/MULHSU/MULHU=5 cycles (STEP0..FINISH); DIV/REM=3..35 cycles with
   divisor-leading-zero early termination; load-use stall = 1 cycle
   (`load_stall`); JALR-consumer stall = 1 cycle (`jr_stall`); jumps resolved
   in ID (2 cycles), taken branches in EX (3 cycles); exceptions always to
   `mtvec` base; `mtvec` init = `{mtvec_addr_i[31:8], 6'b0, 2'b01}`; FF
   register file resets to 0; priority of `irq`/`debug`/`sleep` tie-off
   behavior.

4. **Stimulus delivery:** instruction memory is pre-loaded per test (compliant
   option (b), T4 Q5); the OBI slaves add programmable wait states /
   gnt-before-req windows independently on both buses.

## Review passes applied (per task methodology)

1. **RTL pass** — every meaningful module/FSM/decision point (OBI, prefetch,
   aligner pass-through, LSU, ALU/shifter, mult, div, controller FSM,
   forwarding/stall logic, register file, int_controller, sleep_unit) mapped
   to a requirement row. 
2. **Databook pass** — every chapter (intro, integration, pipeline, IF, LSU,
   HW loops, custom ISA, CSR map, exceptions/interrupts, debug, sleep, perf
   counters, verification) crossed against the plan; in-scope chapters fully
   covered, out-of-scope chapters present as tie-off/anti-check rows.
3. **Guidelines pass** — every Q&A constraint enforced as scope_row /
   tie-off / exclusion (no requirement silently lost).
4. **Corner-case pass** — operand/immediate/shamt/offset/sign/divisor-latency/
   register-sweep/x0/boot/mtvec corners in Generation + Coverage.
5. **Interaction pass** — branch×wait-states, ctrl×memory, multicycle×flush,
   misaligned×wait-states, reset×activity, dual-bus stress, illegal×wait-states.
6. **Negative pass** — illegal encoding categories (a)–(f), gnt-before-req,
   reset mid-transaction, anti-checks (no trap on misaligned, no unexpected
   trap on legal streams, no spurious WB, no spurious LSU traffic from flushed
   path).
7. **Temporal pass** — latency checks (MULH=5, DIV 3..35, load-use 1-cycle at
   zero-wait) + stall retention + flush/refetch timing + teardown drain.
8. **Gap pass** — "how would I break this RTL?" additions: WAW with misaligned
   producer, JALR odd-immediate `& ~1`, AUIPC-at-jump-target, back-to-back
   traps, stall during misaligned second beat, reset during misaligned pair.

## Deliberate limits (documented, not silent assumptions)

- Programs are 32-bit word-aligned only (C extension out of goal, T2 Q1);
  no self-modifying code (fence.i excluded, T4 Q2); bounded programs with an
  end-of-program sentinel.
- CSR *state* (mepc/mcause/mstatus) is not verified (guideline T3); trap
  redirect and side-effect-freedom are. An **optional** hierarchical probe for
  mepc/mcause is noted as `sb_csr_sideband_opt`.
- `NO error injection` on the data bus — the interface has no error signal in
  this configuration (load_err/store_err tied inactive in RTL).
- Initial-register-zero check (`sb_rf_reset_zero_opt`) is flagged as an
  implementation-specific property of the FF register file (not an ISA
  requirement) — kept as an optional one-shot check.

## Revision 2 — content review pass (scope tightening + human text)

- **All guideline/team references removed** from every description, check and
  comment. The sheets now read as self-contained engineering requirements; the
  scope decisions simply *are*, without citing where they came from.
- **Functional checks for outside-the-goal features deleted entirely**
  (nothing about them needs code in the TB beyond static driving):
  - `irq_ack_o`/`irq_id_o` checks, debug-status exclusivity check,
    `core_sleep_o` check — removed; only the static tie-off driving row
    remains (`risc_sys_06`).
  - CSR-side checks removed: the optional `mepc/mcause` probe and the CSR
    encodings illegal-category. The trap *redirect* check (fetch observable)
    stays because illegal instructions are in the I&M verification scope.
  - Illegal-encoding stimulus narrowed to the RV32I/M opcode space: categories
    (a) undefined major opcodes, (b) undefined funct3/funct7, (c) reserved
    shift encodings. F/A/custom/CSR opcode families are never generated at
    all, so no check is needed for them (`risc_dec_04` constraint).
- **Component names aligned to the UVM architecture diagram** (was ad-hoc
  short names): `sys_ctrl_agent`, `if_agent`, `if_passive_agent`,
  `lsu_agent`, `lsu_passive_agent` (each with its
  sequencer/driver/input_monitor or output_monitor), `rvfi_monitor`
  (internal_state_monitor), `ref_model`, `uvm_scoreboard`,
  `sva_module (bind cv32e40p_top)`, `v_sqr`, `virtual_sequences`, `env_cfg`;
  interfaces `sys_ctrl_vif / if_vif / lsu_vif / rvfi_vif`.
- Row counts after this pass: System 55 (was 59) / Generation 42 / Checking 45
  (was 49) / Coverage 40 / Tests 27. Traceability re-verified (all
  gen/chk/cov IDs exist in System, descriptions verbatim).

## Outstanding item

- **`بلان ايديا` folder missing** — when it lands in the repo, run a merge
  pass: validate each idea against databook/RTL/guidelines, then keep / expand /
  merge / correct / reject it into `build_plan.py` (todo list reserved in the
  master table). The current plan is already built from the authoritative
  sources, so ideas can only add scenarios, not change scope.
