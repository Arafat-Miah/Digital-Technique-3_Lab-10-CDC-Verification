# CDC Unit Formal Verification and Gate-Level Synthesis (Lab 10)

## Repository
**Recommended name:** `Digital-Technique-3_Lab-10-CDC-Verification`

## Overview
This repository contains the advanced verification and synthesis results of a **Clock Domain Crossing (CDC) unit** developed for **Digital Technique 3**. 

Unlike previous labs focused on RTL code creation, this week's focus was entirely on **verifying structural safety, protocol compliance, and physical implementation**. Using industry-standard tools (Questa CDC, Synopsys Design Compiler, and Formality), the RTL design was mathematically proven to safely transfer data and control signals across asynchronous clock boundaries without metastability failures, and successfully synthesized into physical logic gates.

## Files Included
- `cdc_unit_static_results.jpg` — Questa CDC Static Analysis GUI showing 0 structural violations.
- `cdc_unit_dynamic_results.jpg` — Questa CDC Protocol Simulation showing 100% coverage and 0 firings.
- `cdc_unit_gatelevel_waves.jpg` — QuestaSim Waveform showing the clean gate-level simulation of the synthesized netlist.

---

## 10.1 Static and Dynamic CDC Analysis
The design was subjected to strict Clock Domain Crossing analysis using **Questa CDC** to ensure no unsafe combinational logic or unregistered signals crossed clock boundaries.

### Static Analysis
The tool mathematically analyzed the RTL structure without simulation:
- **Violations:** 0 (The architecture perfectly adhered to safe CDC guidelines).
- **Proven:** 2 checks (e.g., the Pulse Synchronizer and simple 1-bit syncs were mathematically proven to be structurally flawless).
- **Evaluations:** 4 checks (The multi-bit handshake and reset synchronizers were structurally correct, but required dynamic simulation to verify data stability protocols).

### Dynamic Protocol Simulation
To verify the "Evaluations" from the static phase, automatically generated SystemVerilog Assertions (SVAs) were simulated.
- **Coverage:** Achieved 100% SVA coverage for `cdc_sync` and `cdc_handshake` protocols.
- **Firings:** 0 Firings. The testbench data remained perfectly stable during the entire 4-phase Request/Acknowledge cycles, proving the design is dynamically safe.

**Figure 10.1 — Static CDC Results** ![Static CDC Results](cdc_unit_static_results.jpg)

---

## 10.2 Gate-Level Synthesis & Equivalence Checking

### Logic Synthesis
The RTL was mapped to a standard cell library using **Synopsys Design Compiler** under strict `.sdc` timing constraints (10.0ns `clk`, 54.2ns `mclk`).
- **Optimization:** The tool intelligently optimized away a redundant constant register (`rx_state[1]`), reducing the expected 114 flip-flops down to exactly **113 DFFR_X1 cells**.
- **Timing:** Met all constraints with a generous worst-case Reg-to-Reg setup slack of **+9.37 ns**.
- **Equivalence:** **Synopsys Formality** mathematically proved that the synthesized gates were 100% functionally equivalent to the original RTL (`Verification SUCCEEDED`).

### Gate-Level Simulation (Metastability Handling)
Simulating the physical gate delays revealed the true nature of asynchronous clocks drifting past one another.
- **Observation:** Setup and hold timing violations (`#Error: $setup`) intentionally occurred *only* on the first-stage synchronizer flip-flops (e.g., `play_sff1`, `rff1`, `aff1`). 
- **Resolution:** This perfectly mimicked real-world metastability. By correctly identifying these boundaries and configuring the simulator to disable timing checks on these specific first-stage registers (`set VSIM_DISABLE_TIMINGCHECKS { "*sff1*" "*rff1*" "*aff1*" }`), the final simulation ran flawlessly with no unknown (`'X'`) states propagating into the system.

**Figure 10.2 — Dynamic CDC Results** ![Dynamic CDC Results](cdc_unit_dynamic_results.jpg)

**Figure 10.3 — Gate-Level Simulation Waveforms** ![Gate-Level Waves](cdc_unit_gatelevel_waves.jpg)

---

## 10.3 Observations and Learning
### What I observed
- **The Limits of RTL Simulation:** Standard RTL simulation assumes ideal clocks and instant data transfers. It wasn't until the Gate-Level Simulation that the physical realities of clock drift and metastability (setup/hold violations) became visible.
- **Smart Synthesis Optimization:** I observed how Design Compiler actively analyzes state machines. It detected that my RX FSM only utilized states `00` and `01`, effectively making the MSB a constant `0`. It stripped this flip-flop from the physical silicon to save area and power.

### What I learned
- The fundamental difference between **Static CDC Analysis** (checking the structural drawing/schematic of the code) and **Dynamic CDC Analysis** (verifying that the data isn't moving too fast for the synchronizer to catch).
- How to properly handle intentional setup/hold violations in gate-level simulations by isolating and disabling timing checks on the first stage of a 2-FF synchronizer chain.

---

## Notes
In this week's lab, no new RTL code was authored; the focus was strictly on mastering advanced verification methodologies. When I needed clarification on interpreting the dense synthesis reports and configuring Questa CDC directives, I utilized AI as a guidance tool to help me understand the files and polish the formatting of this report. However, the tool execution, constraint configuration, debugging, and overall comprehension of the CDC verification flow were completed by me.

## Author
Arafat Miah
