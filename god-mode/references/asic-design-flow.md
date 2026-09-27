# IV. LAYER: ASIC DESIGN FLOW (RTL to GDSII — Complete)

```
SPECIFICATION & ARCHITECTURE
  │
  ├── System-level modeling (SystemC, TLM)
  ├── Architecture exploration (Synopsys Platform Architect)
  ├── Memory hierarchy design (cache levels, scratchpad, HBM interface)
  ├── Interconnect: bus (AXI/AHB) vs NoC (mesh/ring/tree)
  ├── Power domain partitioning (voltage islands, always-on domains)
  ├── Clock domain planning (synchronous boundaries, CDC crossings)
  │
  ↓
RTL DESIGN
  │
  ├── Verilog / SystemVerilog
  ├── Finite state machines, datapaths, control logic
  ├── Pipeline design (depth vs throughput vs latency tradeoff)
  ├── Design-for-Test (DFT):
  │     Scan chains: make all flip-flops observable/controllable
  │     BIST (Built-In Self-Test): SRAM, MROM, logic BIST
  │     Boundary scan (JTAG): IEEE 1149.1
  │     ATPG (Automatic Test Pattern Generation): stuck-at, transition faults
  │     Compression: DFTMAX/TESSENT — reduce test time 10-100×
  ├── Design-for-Debug: trace buffers, trigger logic
  ├── Low-power RTL: clock gating insertion, operand isolation
  │
  ↓
FUNCTIONAL VERIFICATION
  │
  ├── Simulation: VCS (Synopsys), Xcelium (Cadence), Questa (Siemens)
  │     Directed tests → constrained random → coverage-driven
  ├── UVM (Universal Verification Methodology):
  │     Layered testbench: driver, monitor, scoreboard, coverage
  │     Reusable VIPs (Verification IP) for standard interfaces
  ├── Formal verification:
  │     Model checking: prove properties hold for ALL inputs (JasperGold)
  │     Equivalence checking: RTL ↔ netlist (Formality, Conformal)
  │     Connectivity checking: verify SoC-level interconnect
  ├── Assertion-based verification (SVA):
  │     `assert property @(posedge clk) req |-> ##[1:3] ack;`
  ├── Coverage targets:
  │     Code coverage: >95% line, branch, condition, toggle
  │     Functional coverage: >99.9% of all coverage points
  ├── Emulation: Palladium (Cadence), ZeBu (Synopsys), Veloce (Siemens)
  │     Run RTL at ~MHz speed (vs kHz for simulation)
  │     Boot OS, run drivers, test full software stack on pre-silicon hardware
  ├── FPGA prototyping: Xilinx VU19P, Intel Agilex — real-time SW development
  │
  ↓
SYNTHESIS
  │
  ├── Tool: Design Compiler (Synopsys) / Genus (Cadence)
  ├── Input: RTL + SDC (timing constraints) + PDK standard cell library
  ├── Output: gate-level netlist (AND, OR, MUX, FF cells from foundry library)
  ├── Optimizations:
  │     Timing-driven: restructure logic to meet frequency target
  │     Area optimization: share resources, merge redundant logic
  │     Power optimization: clock gating, multi-Vt cell selection
  │       HVT (High-Vt): slow but low leakage — for non-critical paths
  │       SVT (Standard-Vt): balanced
  │       LVT (Low-Vt): fast but high leakage — for critical paths
  │       ULVT: fastest, highest leakage — sparingly on worst paths
  ├── Timing: setup/hold analysis at this stage (pre-layout, optimistic)
  │
  ↓
PLACE AND ROUTE (P&R)
  │
  ├── Tool: Innovus (Cadence) / ICC2 (Synopsys)
  ├── Floorplanning:
  │     Place macros (SRAM, PLL, ADC, I/O pads)
  │     Define power domains, voltage areas
  │     Plan power grid (VDD/VSS mesh)
  │     Define pin locations for inter-block interfaces
  ├── Placement:
  │     Global placement: coarse positioning of millions of cells
  │     Detailed placement: legalize to grid, optimize wirelength
  │     Timing-driven placement: pull critical-path cells closer
  ├── Clock Tree Synthesis (CTS):
  │     Build balanced clock distribution network
  │     Target: <10ps skew across clock domain
  │     Structures: H-tree, fishbone, mesh
  │     Useful skew: intentionally skew clock to help timing on critical paths
  │     CTS for multiple clock domains + generated clocks
  ├── Routing:
  │     Global routing: plan paths through routing channels
  │     Detailed routing: assign exact metal tracks and vias
  │     10-15 metal layers, each with preferred direction (alternating H/V)
  │     DRC-clean routing: no shorts, spacing violations, min-width violations
  │     Antenna rule fixing: add diodes to protect gate oxide during etch
  │     Via optimization: redundant vias for reliability
  ├── Post-route optimization:
  │     Buffer insertion, gate sizing, wire re-routing for timing
  │     Filler cells: fill empty spaces (maintain density, prevent litho issues)
  │     Decap cells: add decoupling capacitance to power grid
  │
  ↓
STATIC TIMING ANALYSIS (STA)
  │
  ├── Tool: PrimeTime (Synopsys)
  ├── Reads: gate-level netlist + parasitics (SPEF from extraction)
  ├── Multi-Corner Multi-Mode (MCMM):
  │     Corners: (Process, Voltage, Temperature) combinations
  │       SS (slow-slow): worst-case setup (cold corner: low V, high T — or vice versa)
  │       FF (fast-fast): worst-case hold (opposite corner)
  │       TT (typical): nominal
  │       Typically 5-20+ PVT corners analyzed
  │     Modes: functional, test (scan), sleep, different clock frequencies
  ├── Setup check: data arrives before clock edge + setup time
  │     Slack = T_clock - T_data - T_setup > 0
  ├── Hold check: data stable after clock edge for hold time
  │     Slack = T_data - T_clock - T_hold > 0
  ├── Clock uncertainty: jitter + OCV (on-chip variation) derating
  ├── AOCV / POCV: path-based statistical derating (more accurate than flat OCV)
  ├── SI (Signal Integrity) analysis:
  │     Crosstalk-induced delay changes
  │     Crosstalk-induced glitches (functional failures)
  │     Tool: PrimeTime SI
  │
  ↓
PHYSICAL VERIFICATION
  │
  ├── Tool: Calibre (Siemens) / ICV (Synopsys)
  ├── DRC (Design Rule Check):
  │     Minimum width, spacing, enclosure, density rules
  │     Thousands of rules per layer at advanced nodes
  │     Multi-patterning DRC: color assignment validation
  ├── LVS (Layout vs Schematic):
  │     Extract devices and connections from layout
  │     Compare to schematic netlist — must match exactly
  ├── ERC (Electrical Rule Check):
  │     Floating gates, shorted supplies, missing connections
  ├── Antenna check:
  │     Long metal connected to gate during plasma etch → charge damage
  │     Fix: add reverse-biased diodes to discharge
  ├── Density checks:
  │     Metal density must be within range (for CMP uniformity)
  │     Fill patterns inserted automatically
  │
  ↓
SIGNOFF
  │
  ├── Final STA with extracted parasitics (post-route, post-OPC SPEF)
  ├── IR drop analysis (static + dynamic):
  │     Static: average current through power grid
  │     Dynamic: switching activity → instantaneous voltage drops
  │     Tool: Voltus (Cadence) / RedHawk (ANSYS-Synopsys)
  │     Target: <5% IR drop (V_dd × 0.05)
  ├── Electromigration (EM) check:
  │     Current density on every wire segment vs allowed J_max
  │     Black's equation: MTTF = A × J⁻ⁿ × exp(Ea/kT)
  │     Signoff condition: MTTF > 10 years at worst-case temperature
  ├── Thermal analysis:
  │     Power map → thermal simulation → hotspot identification
  │     Self-heating in FinFET/GAA devices
  ├── Aging analysis:
  │     NBTI, HCI, TDDB → V_th shifts over 10-year lifetime
  │     Age-aware timing libraries: re-run STA with degraded cells
  │
  ↓
TAPEOUT
  │
  ├── GDSII / OASIS file generation (layout database, 10-100+ GB)
  ├── Final DRC/LVS clean signoff
  ├── Metal fill (dummy fills for CMP uniformity)
  ├── Chip ID / revision marking
  ├── Reticle fracturing: partition layout for mask writer
  ├── Submit to foundry
  ├── Mask fabrication: 4-8 weeks
  ├── Wafer fabrication: 10-16 weeks (60-100 mask layers)
  ├── First silicon → bring-up → characterization → debug
  │     Silicon debug: probe station, logic analyzer, scan dump analysis
  │     Speed binning: sort dies by max operating frequency
  │     Yield ramp: iterative process/design fixes to improve yield
  ├── Production qualification → volume ramp
```

---
