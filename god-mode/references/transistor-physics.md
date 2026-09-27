# II. LAYER: TRANSISTOR PHYSICS & DEVICE ENGINEERING

## II-A. Fundamental MOSFET Physics

```
Drain current (linear region):
  I_ds = µ × C_ox × (W/L) × [(V_gs - V_th)V_ds - V_ds²/2]

Drain current (saturation):
  I_ds = (µ × C_ox / 2) × (W/L) × (V_gs - V_th)²

Gate oxide capacitance:
  C_ox = ε_ox / t_ox = (ε₀ × k) / t_ox
  For HfO₂ (k ≈ 25): C_ox is ~6× higher than SiO₂ (k ≈ 3.9)
  → Can use thicker physical oxide while maintaining same electrical thickness
  → Equivalent Oxide Thickness (EOT) = t_physical × (3.9 / k_highk)

Maximum switching frequency:
  f_T = g_m / (2π × C_total)
  g_m = ∂I_ds/∂V_gs = µ × C_ox × (W/L) × V_ds  (linear)
  g_m = µ × C_ox × (W/L) × (V_gs - V_th)         (saturation)

Dynamic power:
  P_dynamic = α × C_load × V_dd² × f
  α = activity factor (fraction of gates switching per cycle, typically 0.1-0.3)

Leakage power:
  P_leak = I_off × V_dd
  I_off = I₀ × exp((V_gs - V_th) / (n × V_T))
  V_T = kT/q ≈ 26mV at room temperature
  n = subthreshold ideality factor (1.0 ideal, ~1.3 for planar, ~1.05 for GAA)

Subthreshold swing:
  SS = n × V_T × ln(10) ≈ n × 60 mV/decade at 300K
  Theoretical limit: 60 mV/dec (Boltzmann tyranny)
  Planar MOSFET: ~80-100 mV/dec
  FinFET: ~65-75 mV/dec
  GAA/Nanosheet: ~62-68 mV/dec
  Negative capacitance FET (NC-FET): potentially <60 mV/dec (sub-Boltzmann)
  Tunnel FET (TFET): <60 mV/dec via band-to-band tunneling (low I_on though)
```

## II-B. Transistor Architecture Evolution (Complete)

### Planar MOSFET (>22nm)
- Gate sits on TOP of flat silicon channel
- At small L_gate, gate loses control of channel → short-channel effects
- DIBL (Drain-Induced Barrier Lowering): V_th drops as V_ds increases
- Punch-through: source-drain depletion regions merge → uncontrollable leakage
- End of scaling: ~28nm for planar bulk, ~22nm for planar SOI (FDSOI)
- FDSOI (Fully Depleted SOI): thin Si on insulator, better electrostatics than bulk
  Used by: GlobalFoundries 22FDX, STMicroelectronics, Samsung 28FDS

### FinFET (22nm → 3nm)
- Channel is a vertical "fin" of silicon, gate wraps 3 sides
- Electrostatic control: gate controls from 3 sides vs 1 → better SS, lower leakage
- Fin dimensions at 5nm node: width ~5-7nm, height ~45-50nm, pitch ~25-30nm
- Multi-fin: parallel fins for more drive current (typical: 1-3 fins per transistor)
- Limitations: fin width quantization (can't have fractional fins), high parasitic cap
  between fins, reduced mobility from narrow fin confinement
- Timeline: Intel 22nm (2012) → TSMC 16nm → 10nm → 7nm → 5nm → 3nm (last FinFET node)
- Industry: TSMC N3/N3E, Samsung 3GAE — last generation before GAA transition

### GAA-FET / GAAFET / Nanosheet (3nm → 1.4nm)
- Channel: horizontal silicon nanosheets stacked vertically (typically 3-4 sheets)
- Gate wraps ALL 4 SIDES of each nanosheet → near-ideal electrostatic control
- Sheet dimensions: width 10-50nm (tunable!), thickness ~5-7nm, spacing ~10-12nm
- Key advantage over FinFET: width is continuously tunable (not quantized to fin count)
  → Better power/performance optimization per transistor
- TSMC N2 (2025): first production GAA, ~48nm contacted poly pitch (CPP),
  ~28nm minimum metal pitch, 4 nanosheets
- Intel 18A (2025): GAA + backside power delivery (PowerVia) + RibbonFET
- Samsung SF3/SF2: GAA with MBCFET (Multi-Bridge Channel FET)

### CFET — Complementary FET (sub-1nm research → production ~2028-2029)
- NMOS stacked DIRECTLY ON TOP of PMOS (or vice versa)
- Eliminates N-to-P spacing → ~40% cell area reduction
- Two integration approaches:
  a) Monolithic: build bottom device, then top device on same wafer
     Challenge: thermal budget — bottom device annealing during top device processing
     Solution: low-temperature epitaxy + laser annealing for top tier (<500°C)
  b) Sequential: build on separate wafers, bond, then process
     Challenge: alignment accuracy for bonded layers
- IMEC 2025: demonstrated functional CFET with 4-sheet NMOS over 4-sheet PMOS
- Critical for: sub-1nm logic density scaling without shrinking transistors further
- Required EDA upgrades: 3D-aware place and route, new standard cell libraries

### 2D Material FETs (research → targeted production ~2030+)
- Channel material: monolayer or few-layer 2D semiconductors
  MoS₂: bandgap 1.8eV (mono), 1.2eV (bulk), n-type
  WSe₂: bandgap 1.7eV (mono), ambipolar (n and p-type)
  WS₂: bandgap 2.1eV (mono), n-type
  Black phosphorus (BP): tunable bandgap 0.3-2.0eV, anisotropic
  Graphene: zero bandgap (semimetal) — for interconnects, not logic
- Why: at <1nm channel thickness, silicon surface roughness scattering
  destroys carrier mobility. 2D materials have atomically smooth surfaces.
  Atomically thin body → ultimate electrostatic control (SS → 60 mV/dec)
- Challenges (and 2025-2026 status):
  Contact resistance: metal-2D interface ~10× worse than Si contacts
    Breakthrough: TSMC/MIT 2025 — semimetal Bi₂Se₃ contacts, 123 µΩ·cm²
  Large-area growth: CVD on sapphire with uniform monolayer, then transfer
    Samsung 2025: wafer-scale MoS₂ growth with <5% thickness variation
  Doping: substitutional doping is hard — use electrostatic doping or
    molecular dopants (F₄-TCNQ for p-type)
  Integration with CMOS: use as BEOL (back-end-of-line) transistors
    for 3D monolithic stacking — don't need to replace silicon FEOL
- Near-term application: BEOL select transistors for 3D SRAM/memory

### Negative Capacitance FET (NC-FET)
- Ferroelectric layer (HfZrO₂) in gate stack creates negative capacitance effect
- Amplifies surface potential → internal voltage gain → sub-60mV/dec SS
- Reality: hysteresis issues, reliability of ferroelectric under cycling
- Status: research devices demonstrated, no production adoption yet
- Potential: could extend voltage scaling below 0.5V

### Tunnel FET (TFET)
- Operates on band-to-band tunneling instead of thermionic emission
- Sub-60mV/dec SS achievable (not limited by Boltzmann)
- Problem: very low ON current (I_on) — orders of magnitude below MOSFET
- Heterojunction designs (InAs/GaSb) improve I_on but add fab complexity
- Status: research, not near production

## II-C. Quantum Effects at Nanoscale

```
At sub-5nm channel lengths, quantum mechanics dominates:

Direct source-drain tunneling:
  T ∝ exp(-2 × d × √(2m*ΔE) / ℏ)
  d = barrier width (channel length)
  m* = effective mass
  ΔE = barrier height
  At L_gate < 3nm: tunneling current ≈ ON current → transistor becomes useless
  This is the HARD LIMIT of scaling for silicon

Quantum confinement:
  In nanosheets <5nm thick, carriers are quantum-confined
  Energy subbands form → effective bandgap increases
  Carrier mobility changes (can increase or decrease depending on orientation)
  Threshold voltage becomes thickness-dependent

Gate leakage tunneling:
  Electrons tunnel through the gate dielectric
  For SiO₂ < 1nm thick: leakage is catastrophic
  Solution: high-k dielectrics (HfO₂, ZrO₂) — physically thicker, electrically thin
  EOT target: 0.5-0.7nm for advanced nodes
```

---
