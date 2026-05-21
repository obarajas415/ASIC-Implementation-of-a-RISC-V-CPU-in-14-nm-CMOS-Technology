# ASIC Implementation of a RISC-V Processor in 14-nm CMOS Technology
**Master's Thesis Summary — Oscar Benjamin Barajas**
San Francisco State University | Master of Science in Electrical and Computer Engineering | Spring 2026

---

## Abstract

This project presents the complete ASIC implementation of a 5-stage pipelined RISC-V CPU processor using the Synopsys EDA tool suite and a 14-nm CMOS technology library. The design flow covers RTL development, functional verification, logic synthesis, physical implementation, and post-layout static timing analysis (STA).

---

## 1. Background & Motivation

The rise of on-board machine learning has driven demand for application-specific hardware. General-purpose CPUs and GPUs are insufficient for the power, latency, and customization needs of embedded autonomous systems. Proprietary ISAs (x86, ARM) impose licensing constraints that limit innovation. RISC-V — an open, license-free ISA developed at UC Berkeley in 2010 — offers a compelling alternative. With over 1,000 contributing organizations in RISC-V International and a demonstrated NASA space mission (Microchip PolarFire), RISC-V is proving viable for demanding real-world applications.

---

## 2. Project Goals

- Design a Verilog RTL 5-stage RISC-V CPU (Fetch → Decode → Execute → Memory → Write Back)
- Functionally verify RTL using Synopsys VCS + waveform analysis with Verdi
- Synthesize to a gate-level netlist via Synopsys Design Compiler (SAED 14-nm library)
- Perform pre-layout STA using Synopsys PrimeTime
- Complete physical implementation (floorplan, placement, CTS, routing) using IC Compiler II (ICC2)
- Perform post-layout STA to confirm timing closure with parasitics

---

## 3. ASIC Design Flow Overview

The project follows the industry-standard ASIC design flow:

1. **Specification Definition** — 5 ns clock period target (200 MHz); R-type and I-type instruction formats
2. **RTL Design** — Verilog implementation with pipelining, hazard detection, and forwarding logic
3. **Functional Verification** — Synopsys VCS simulation + Verdi waveform inspection
4. **Logic Synthesis** — Synopsys Design Compiler → gate-level netlist
5. **Pre-Layout STA** — Synopsys PrimeTime timing sign-off
6. **Physical Implementation** — Synopsys ICC2 (floorplan → placement → CTS → routing)
7. **Post-Layout STA** — Timing verification on extracted parasitic netlist

---

## 4. RISC-V Architecture & RTL Design

The processor uses a **custom 7-bit opcode encoding** (not standard RISC-V binary encoding) mapping to 8 instruction classes:

| Opcode | Format | Instructions |
|--------|--------|-------------|
| 0000001 | R | ADD, SUB, SLT, AND, OR, XOR, SLL, SRL, SRA |
| 0000010 | I | ADDI, SLTI, SLTIU |
| 0000011 | I | ANDI, ORI, XORI |
| 0000100 | I | SLLI, SRLI, SRAI |
| 0000101 | B | BEQ (branch if equal) |
| 0000110 | J | JUMP (unconditional) |
| 0000111 | S | STORE (32-bit word) |
| 0001000 | I | LOAD (32-bit word) |

**Five Pipeline Stages:**
- **IF** — Fetch instruction from memory via Program Counter
- **ID** — Decode instruction, read register file, sign-extend immediates
- **EX** — ALU performs arithmetic/logical operation
- **MEM** — Load/store access to data memory
- **WB** — Write result back to register file

Key RTL modules include: ALU (32-bit adder + comparator), Control Unit, Forwarding Unit, Hazard Detection Unit, 4 pipeline registers (IFID, IDEX, EXMEM, MEMWB), and Data Memory (4096-entry, 32-bit).

---

## 5. Functional Verification

Verification was performed using **Synopsys VCS** with waveform analysis in **Synopsys Verdi**.

- Simulation completed at **5,100,000 ps (5.1 μs)** with no runtime errors
- All major blocks exercised: PC, Instruction Fetch/Decode, Register File, ALU, Branch/Jump, Hazard Detection, Forwarding Unit, all pipeline registers, and Data Memory
- Pre-synthesis setup slack: **+7.31 ns** (timing met with margin)
- Pre-synthesis hold slack: **+0.05 ns** (timing met)

---

## 6. Pre-Layout Static Timing Analysis (PrimeTime)

An independent PrimeTime run was performed on the synthesized netlist as an industry-standard sign-off check.

| Report | Result |
|--------|--------|
| Setup slack | **+3.88 ns** (MET — 3.88 ns margin at 5 ns target) |
| Hold slack | No violations |

**Power & Area Summary (PrimeTime):**

| Metric | Value |
|--------|-------|
| Total Dynamic Power | 1.2118 mW |
| Cell Leakage Power | 9.0082 mW |
| Register power share | ~50.25% |
| Combinational power share | ~49.71% |
| Total Cell Area | 9,803 units² |
| Total Area (with interconnect) | 28,226 units² |
| Number of cells | 20,943 |
| Sequential cells | 5,520 |
| Combinational cells | 15,356 |

---

## 7. Physical Implementation (IC Compiler II)

Physical design was performed in ICC2 through the following stages:

**Floorplanning** — Die boundary, core area, and I/O pin placement defined via `floorplan_icc.tcl`.

**Power/Ground Network** — VDD/VSS ring constructed around the core via `pg_net.tcl` + `power.tcl`.

**Placement** — Standard cells placed using `place_opt` to minimize wirelength and meet timing. Post-placement timing confirmed to still pass.

**Clock Tree Synthesis (CTS)** — Ideal clock replaced with a balanced physical clock tree using `clock_opt`. Buffers inserted to balance latency to all flip-flops.

**Routing** — Full auto-routing via `route_auto` followed by `optimize_routes` and `check_routes`.

**Post-Layout Timing Results:**

| Report | Slack | Status |
|--------|-------|--------|
| Setup (post-CTS) | **+2.83 ns** | MET |
| Hold (post-CTS) | **+0.02 ns** | MET |

> Note: Setup slack decreased from 3.88 ns (pre-layout) to 2.83 ns (post-layout) due to actual parasitics and routing delays — still comfortably meeting the 5 ns clock target.

---

## 8. Key Results Summary

| Stage | Outcome |
|-------|---------|
| RTL Simulation | Passed — no errors at 5.1 μs |
| Synthesis (Design Compiler) | Timing closed at 200 MHz |
| Pre-Layout STA (PrimeTime) | Setup: +3.88 ns, Hold: met |
| Physical Implementation (ICC2) | Floorplan → CTS → Routing complete |
| Post-Layout STA | Setup: +2.83 ns, Hold: +0.02 ns |

---

## 9. Tools & Technology

| Tool | Purpose |
|------|---------|
| Synopsys VCS + Verdi | RTL simulation & waveform analysis |
| Synopsys Design Compiler | Logic synthesis |
| Synopsys PrimeTime | Static timing analysis (pre & post layout) |
| Synopsys IC Compiler II | Physical implementation (P&R) |
| SAED 14-nm EDK | Standard cell library (HVT/LVT/RVT/SLVT) |

---

## References

1. Waterman et al., *RISC-V Instruction Set Manual*, RISC-V Foundation, 2019
2. Patterson & Hennessy, *Computer Organization and Design RISC-V Edition*, 2020
3. T. K. Prakasam, *ASIC Implementation of a RISC-V Processor*, SFSU, 2018
4. L. Singh, *Tutorial on ASIC Design Flow Using Synopsys Tools with 14-nm*, SFSU, 2020
5. Weste & Harris, *CMOS VLSI Design*, 4th ed., Addison-Wesley, 2011
6–9. Synopsys User Guides: Design Compiler, IC Compiler II, PrimeTime, VCS
