# III. LAYER: PHOTOLITHOGRAPHY & NANOFABRICATION (Complete)

## III-A. Optical Lithography Physics

```
Rayleigh criterion:
  CD_min = k₁ × λ / NA
  
  CD_min = minimum resolvable feature (critical dimension)
  k₁ = process complexity factor
    Theoretical minimum: 0.25 (single exposure, coherent illumination)
    Practical with OPC/SMO: 0.28-0.35
    With multi-patterning: effectively k₁ < 0.25 per exposure
  λ = wavelength of exposure light
  NA = numerical aperture of projection lens

Depth of focus:
  DOF = k₂ × λ / NA²
  
  Higher NA → smaller features BUT shallower depth of focus
  → Wafer must be flatter, resist must be thinner
  High-NA EUV (0.55): DOF ≈ 45nm — resist must be <20nm thick

Resolution Enhancement Techniques (RET):
  Off-axis illumination (OAI): dipole, quadrupole, freeform
  Phase-shift masks (PSM): alternating, attenuated
  Optical Proximity Correction (OPC): modify mask shapes
  Sub-Resolution Assist Features (SRAF): non-printing helper patterns
  Source-Mask Optimization (SMO): co-optimize light source + mask
  Inverse Lithography Technology (ILT): compute-optimal mask via inverse problem
```

## III-B. Lithography Generations

### g-line / i-line (legacy: >350nm)
- Mercury lamp: g-line 436nm, i-line 365nm
- Contact/proximity or simple projection
- Still used: MEMS, power devices, packaging

### KrF (DUV 248nm)
- Krypton fluoride excimer laser
- Enabled 250nm → 130nm nodes
- Still widely used for non-critical layers

### ArF (DUV 193nm)
- Argon fluoride excimer laser
- Dry: 193nm, NA ≈ 0.85 → CD ~68nm
- Immersion: water between lens and wafer, NA = 1.35 → CD ~43nm
  - Water n=1.44 at 193nm → effective NA exceeds 1.0
  - ASML TWINSCAN NXT:2050 — workhorse of the industry
  - With multi-patterning: extends to 7nm/5nm production

### EUV (13.5nm)
- Extreme Ultraviolet — not really "ultraviolet" (soft X-ray)
- Source: Sn (tin) droplet LPP (Laser-Produced Plasma)
  - 50,000 tin droplets per second, each ~25µm diameter
  - CO₂ laser (>30kW) hits each droplet twice:
    Pre-pulse: flattens droplet into pancake for better absorption
    Main pulse: heats to ~500,000°C → tin plasma emits 13.5nm photons
  - Source power: ~250-400W at intermediate focus
  - Collector mirror (5m diameter): captures and focuses EUV photons
  - Source availability: >90% uptime target (was a huge challenge, now mature)

- Optics: ALL REFLECTIVE (no material transmits 13.5nm light)
  - 11 multilayer mirrors: Mo/Si bilayers, ~40 pairs, ~7nm period
  - Each mirror ~70% reflective → total transmission: 0.70¹¹ ≈ 2%
  - 4× reduction optics (mask to wafer)
  - Mirror surface roughness: <0.1nm RMS (atomically smooth)
  - Operating environment: ultra-high vacuum (<1 Pa)
  - Hydrogen gas flow: cleans tin debris from mirrors
  - Mirror contamination/degradation: multi-year lifetime, field-replaceable

- Pellicle: thin membrane protecting mask from particles
  - EUV pellicle: ultra-thin (50nm) CNT or polysilicon membrane
  - Must transmit >90% of EUV while stopping particles
  - One of the hardest engineering challenges in EUV

- Resist:
  Chemically Amplified Resists (CAR):
    - Photoacid generator (PAG) absorbs EUV photon
    - Generates acid → catalytic chain reaction during post-exposure bake
    - Amplifies single photon event into large chemical change
    - Problem: acid diffusion blurs features (stochastic)
  Metal-oxide resists (inorganic):
    - HfO₂, ZrO₂, SnOx-based resists
    - Higher EUV absorption → fewer photons needed → faster throughput
    - Better etch resistance
    - Less diffusion blur
    - Inpria (now JSR): leading metal-oxide EUV resist

- ASML TWINSCAN NXE:3800E:
  NA = 0.33, 13.5nm wavelength
  CD_min ≈ 13nm (single exposure with k₁ = 0.32)
  Throughput: >185 wafers/hour (WPH)
  Overlay: <1.0nm
  Tool cost: $150-200M
  Used at: TSMC 5nm/3nm, Samsung 3nm, Intel 4/3

### High-NA EUV (0.55 NA)
- ASML TWINSCAN EXE:5200 (first tool shipped 2025 to Intel)
  - NA increased from 0.33 → 0.55 (+67%)
  - CD_min ≈ 8nm (single exposure)
  - Enables single-exposure patterning at 2nm/1.4nm nodes

- Anamorphic optics:
  - 4× demagnification in scan direction, 8× in cross-scan
  - Required because larger NA → larger mirror → mask would need to be impossibly large
  - Consequence: mask field is HALF the size of standard EUV
  - Half-field stitching: two exposures to cover full die area
  - Stitching accuracy: <1nm overlay at stitch boundary
  - Some designs avoid stitching by fitting in half-field (chiplets help here)

- New requirements:
  - Thinner resist: <20nm (reduced DOF demands it)
  - New resist platforms: metal-oxide or dry-deposited resists
  - Low-k₁ requires aggressive OPC/ILT on mask
  - Mask infrastructure: new mask blanks, new pellicle, new inspection tools
  - Mask cost: >$500M for a full set at High-NA nodes

- Who:
  Intel Clearwater Forest / Panther Lake (18A/14A): first High-NA products
  TSMC N1.4/A14: High-NA adoption ~2027
  Samsung SF1.4: similar timeline

- Cost: $350-400M per tool, Intel ordered multiple units

## III-C. Multi-Patterning (Complete Treatment)

### SADP (Self-Aligned Double Patterning)
```
Process:
1. Deposit mandrel material (SiN or amorphous Si) on target layer
2. Litho + etch: pattern mandrel with relaxed pitch (2× final pitch)
3. Deposit conformal spacer (ALD SiO₂, precisely controlled thickness)
4. Etch spacer: anisotropic RIE leaves sidewall spacers on mandrel edges
5. Remove mandrel (selective etch): only spacers remain
6. Spacers now have 2× the density of original mandrel pattern
7. Etch target layer using spacers as hard mask

Pitch: final_pitch = spacer_thickness × 2
  Example: 40nm mandrel pitch → 20nm spacer pitch

Used: TSMC 7nm/5nm metal layers, Intel 10nm
Advantage: self-aligned (no overlay error between features)
Limitation: only creates lines at regular pitch — no arbitrary shapes
```

### SAQP (Self-Aligned Quadruple Patterning)
```
Process: SADP done TWICE
1. First SADP → 2× density pattern (spacers on mandrel)
2. These spacers become the new mandrel
3. Second SADP → 4× density pattern
4. Etch target layer

Pitch: final_pitch = original_mandrel_pitch / 4
  Example: 80nm litho pitch → 20nm final pitch

Used: Intel 10nm metal layers, some TSMC 3nm layers
Advantage: enables sub-20nm pitch with 193i litho (no EUV!)
Limitation: very expensive (many deposition + etch steps), edge placement errors accumulate
```

### LELE (Litho-Etch-Litho-Etch)
```
Process:
1. First litho exposure + etch (pattern A)
2. Second litho exposure + etch (pattern B, interleaved with A)
3. Combined pattern has 2× density

Advantage: supports arbitrary 2D shapes (not just regular lines)
Limitation: requires <2nm overlay between exposures (expensive metrology)
Used: some via layers, cut masks
```

### EUV impact on multi-patterning:
```
TSMC N7: 4 EUV layers replaced 20+ DUV multi-patterning steps
TSMC N5: 14 EUV layers
TSMC N3: ~20 EUV layers, minimal multi-patterning
TSMC N2 / Intel 18A: 5-8 critical EUV layers + DUV for non-critical
High-NA EUV: aims to eliminate remaining EUV multi-patterning
```

## III-D. Computational Lithography (Complete)

### OPC (Optical Proximity Correction)
```
Problem: What you draw on the mask ≠ what prints on wafer
  - Dense lines print narrower than isolated lines
  - Line ends pull back (line-end shortening)
  - Corners round off
  - Features interact via diffraction

Solution: Pre-distort mask shapes to compensate
  - Rule-based OPC: lookup table of corrections (fast, less accurate)
  - Model-based OPC: simulate aerial image, iterate corrections (slow, accurate)
  - Add serifs to corners, bias line widths, add SRAFs

Runtime: Model-based OPC for a full chip: 10,000-100,000 CPU hours
  → Massive compute farms required (GPU acceleration emerging)
```

### ILT (Inverse Lithography Technology)
```
Instead of iterating corrections, solve the inverse problem:
  "Given desired wafer pattern, compute optimal mask shape"

Formulated as optimization problem:
  minimize |I_aerial(mask) - I_target|² + regularization
  
  I_aerial = simulated image on wafer from mask
  I_target = desired pattern
  Regularization: ensures mask is manufacturable (minimum feature size, etc.)

Result: curvilinear mask shapes (not Manhattan geometry)
  → Better process window, fewer defects, fewer hotspots
  → But curvilinear masks are harder to write (e-beam mask writer time increases)

ASML/Brion: full-chip ILT now feasible with GPU acceleration
  → Adopted at TSMC 3nm/2nm for critical layers
```

### SMO (Source-Mask Optimization)
```
Co-optimize illumination source shape AND mask pattern simultaneously
  → Freeform illumination (not just standard dipole/quadrupole)
  → Larger process window = better yield

Modern approach: Pixel-based illumination
  → Programmable illuminator with ~1000 individually controllable pixels
  → Different illumination per layer for optimal imaging
```

### Stochastic Effects (The Photon Budget Crisis)
```
At EUV, photon count per pixel is LOW:
  Energy per photon: E = hc/λ = 92 eV at 13.5nm
  Compare: DUV 193nm photon = 6.4 eV (14× less energetic)
  
  At a given dose, you get 14× fewer EUV photons than DUV photons
  
  For a 10nm × 10nm pixel at typical dose (30 mJ/cm²):
    ~20-50 EUV photons per pixel
  
  Statistical variation: σ = √N (Poisson statistics)
    √25 / 25 = 20% shot noise → significant line edge roughness (LER)

LER (Line Edge Roughness):
  LER ∝ 1/√(dose)
  More photons = smoother edges but slower throughput
  Target: LER < 1.5nm 3σ for sub-3nm nodes
  This is fundamentally limited by physics — no amount of OPC fixes it

RLS Triangle (Resolution-LER-Sensitivity tradeoff):
  Can optimize any two, but not all three simultaneously
  High-sensitivity resist: fast throughput but higher LER
  Low-sensitivity resist: smooth edges but slow throughput
  → Active area of research: metal-oxide resists, EUV sensitizers
```

## III-E. Mask / Reticle Technology

```
EUV mask structure (reflective — NOT transmissive):
  Substrate: ultra-low thermal expansion glass (ULE or Zerodur)
  Multilayer reflector: 40 pairs of Mo/Si (reflects 13.5nm at ~67% efficiency)
  Capping layer: Ru (protects multilayer from oxidation)
  Absorber: TaN or TaBN (absorbs EUV where pattern should be dark)
  Absorber thickness: ~60-70nm (attenuated PSM) or thicker for binary

Mask defects — the nightmare:
  Even a single particle (>25nm) on an EUV mask prints on EVERY die
  Mask blank defect density requirement: <0.003 defects/cm²
  Phase defects in multilayer: buried particles cause local phase shifts
  → Actinic (at-wavelength) mask inspection at 13.5nm required
  → Only tool: ASML/ZEISS AIMS™ EUV (one of the most expensive tools in a fab)

Mask write:
  Electron beam writer: Gaussian beam or shaped beam
  Variable Shaped Beam (VSB): NuFlare/JEOL — standard for DUV masks
  Multi-Beam Mask Writer: IMS MBMW (256,000 parallel beams)
    → Required for curvilinear ILT masks (10-100× more data than Manhattan)
    → Write time: 10-20 hours for a single EUV mask plate

Mask cost:
  DUV mask: $50-200K
  EUV mask: $300-500K per plate
  Full mask set (all layers):
    7nm: $10-15M
    3nm: $20-30M
    2nm: $30-50M
    High-NA: >$50M (estimated)
```

## III-F. Wafer Fabrication Process Flow (Complete)

### Front-End-of-Line (FEOL) — Building Transistors

```
1. Substrate preparation
   - 300mm (12") single-crystal Si wafer
   - Czochralski growth: seed crystal slowly pulled from melt
   - Crystal orientation: <100> for CMOS (optimal mobility)
   - p-type doping: boron-doped to ~10¹⁵ cm⁻³
   - CMP polish: <0.2nm surface roughness
   - Cost: $400-600 per blank wafer (prime grade)

2. STI (Shallow Trench Isolation)
   - Defines active device regions
   - Etch trenches ~250-300nm deep
   - Fill with SiO₂ (HARP or flowable CVD)
   - CMP planarize

3. Well formation
   - Ion implantation: n-well (phosphorus) and p-well (boron)
   - Retrograde well profile for latchup prevention
   - High-energy implant: 100keV-1MeV

4. Gate stack formation (GAA process — 2nm node example)
   - Epitaxial Si/SiGe superlattice growth (alternating layers)
     Si: future channel, SiGe: sacrificial (will be removed)
     Typically 3-4 Si/SiGe pairs
   - Fin patterning (SADP/EUV): define nanosheet width
   - Inner spacer formation: recess SiGe, fill with dielectric
   - SiGe release: selective etch removes SiGe, leaves Si nanosheets suspended
   - Gate dielectric: ALD HfO₂ (high-k, ~1-2nm, EOT ~0.7nm)
     Interfacial SiO₂ layer: ~0.3-0.5nm (cannot be eliminated)
   - Work function metal: TiN/TaN stack (tunes V_th)
     NMOS: TiAl or similar (low work function)
     PMOS: TiN (high work function)
   - Gate fill: tungsten (W) or cobalt (Co) or ruthenium (Ru)
   - Gate CMP: planarize

5. Source/drain formation
   - Epitaxial growth: raised source/drain for low contact resistance
     NMOS: SiP (silicon-phosphorus) for tensile strain
     PMOS: SiGe (silicon-germanium) for compressive strain → higher hole mobility
   - Contact etch: self-aligned contact (SAC) process
   - Silicide: TiSi or NiSi for low contact resistance (~1-10 Ω·µm)
   - Contact plug: tungsten (W) CVD → CMP
```

### Middle-of-Line (MOL) — Connecting Transistors to Wires

```
6. Local interconnect
   - Connects transistor contacts to first metal layer (M1)
   - Via-0: tungsten or cobalt plugs
   - Buried power rail (BPR): power lines underneath transistors
     → Frees routing tracks on metal layers for signals
     → Intel PowerVia: backside power delivery eliminates BPR compromise
```

### Back-End-of-Line (BEOL) — Building the Wire Stack

```
7. Metallization (10-15+ metal layers)
   - Dual damascene process (per layer):
     a. Deposit low-k dielectric (k ≈ 2.5-3.0, target <2.5)
        Materials: SiCOH, porous variants, air gaps for lowest layers
     b. Pattern trench + via (litho + etch)
     c. Barrier deposition: Ta/TaN liner (~2-3nm) — prevents Cu diffusion
     d. Cu seed layer: PVD (~5nm)
     e. Cu electroplating: fills trench + via
     f. CMP: remove excess Cu, planarize surface
     g. Capping: SiCN or SiN diffusion barrier on top

   - Metal stack hierarchy (bottom to top):
     M1-M3: Finest pitch (~20-28nm at 3nm node), shortest wires, local routing
       Cu resistivity problem: at <20nm width, R increases 5-10× due to
       grain boundary and surface scattering
       Emerging metals: Ru (no barrier needed), Mo, Co
     M4-M7: Intermediate pitch (~40-60nm), block-level routing
       Cu still viable here
     M8-M12: Semi-global (~100-200nm pitch)
     M13-M15: Global routing (~400nm-1µm pitch), power distribution
       Thick, low-resistance Cu

   - Air gaps:
     Remove dielectric between wires (k → 1.0)
     20-30% RC delay reduction at tightest pitches
     Mechanical fragility: must survive CMP and packaging

   - Backside power delivery network (BSDN):
     Intel PowerVia: power routed through BACKSIDE of wafer
     Process: after FEOL, flip wafer, thin to ~1µm, etch TSVs from back,
     build power grid on backside
     Benefit: signal routing on frontside is uncongested → 6-10% perf gain
     Who: Intel 18A (first production use), TSMC N2P backside power (2026+)

8. Passivation and pad formation
   - Final passivation: SiN/SiO₂ stack protects chip from moisture/contamination
   - Bond pad opening: expose Al or Cu pads for wire bonding or bumping
   - Redistribution layer (RDL): reroute pads for advanced packaging

9. Wafer-level testing (probe)
   - Probe card contacts each die pad
   - Run functional tests: verify basic operation
   - Mark bad dies (inkless mark or e-fuse)
   - Yield determination: good_dies / total_dies

10. Dicing and packaging (see Advanced Packaging section)
```

## III-G. Deposition Techniques (Complete)

```
CVD (Chemical Vapor Deposition):
  - Precursor gases react on heated wafer surface
  - LPCVD: low pressure, uniform films, 400-800°C
  - PECVD: plasma-enhanced, lower temperature (200-400°C)
  - Used for: SiO₂, SiN, SiCOH, amorphous Si, W
  - Conformality: moderate (depends on process)

ALD (Atomic Layer Deposition):
  - Sequential self-limiting reactions → one atomic layer per cycle
  - Cycle: precursor A → purge → precursor B → purge → repeat
  - Growth rate: 0.5-1.5 Å per cycle
  - Temperature: 150-350°C (thermal), 25-200°C (plasma-enhanced)
  - Conformality: PERFECT (coats inside 100:1 aspect ratio trenches)
  - Used for: HfO₂ (gate dielectric), Al₂O₃, TiN, TaN
  - Critical: the ONLY technique precise enough for gate dielectrics at <1nm EOT

PVD (Physical Vapor Deposition):
  - Sputtering: bombard target with Ar ions, atoms deposit on wafer
  - Evaporation: heat source material until it evaporates
  - Used for: metal seed layers (Cu, Al), barrier metals, hard masks
  - Poor conformality (line-of-sight deposition)

Epitaxy:
  - Grow crystalline material on crystalline substrate (lattice-matched)
  - MBE (Molecular Beam Epitaxy): ultra-precise, slow, research
  - MOCVD: faster, used for III-V semiconductors
  - Si/SiGe epitaxy: CVD-based (SiH₄ + GeH₄ precursors)
  - Used for: channel layers, source/drain stressors, superlattice for GAA
  - Temperature: 500-700°C (must not disturb earlier structures)

Electroplating:
  - Electrochemical deposition from solution
  - Used for: Cu metallization (fill damascene trenches)
  - Superfilling: additives ensure bottom-up fill without voids
  - Current density: 5-20 mA/cm²
```

## III-H. Etch Techniques (Complete)

```
RIE (Reactive Ion Etch):
  - Plasma generates reactive ions + radicals
  - Ions accelerate toward wafer → anisotropic (vertical) etch
  - Selectivity: etch target material faster than mask/underlying layer
  - Gases: CF₄/CHF₃ (SiO₂ etch), Cl₂/HBr (Si etch), BCl₃/Cl₂ (metal etch)
  - Profile control: balance chemical (isotropic) vs physical (anisotropic) etch
  - ARDE: Aspect Ratio Dependent Etch — deeper trenches etch slower

ALE (Atomic Layer Etch):
  - Remove exactly ONE atomic layer per cycle (inverse of ALD)
  - Cycle: surface modification → removal
  - Precision: sub-angstrom depth control
  - Used for: gate recess, fin trimming, nanosheet release
  - Critical for GAA: must stop precisely at Si/SiGe interface

Wet etch:
  - Chemical dissolution in liquid
  - Isotropic (etches in all directions equally)
  - Used for: cleaning, sacrificial layer removal, SiGe release
  - SC-1: NH₄OH/H₂O₂ — particle removal
  - SC-2: HCl/H₂O₂ — metal contamination removal
  - HF (hydrofluoric acid): SiO₂ removal, native oxide clean
  - Vapor HF: dry alternative for sensitive structures
```

## III-I. Metrology & Inspection

```
CD-SEM (Critical Dimension SEM):
  - Measures feature width/height/profile
  - Resolution: <1nm
  - Throughput: ~20 wafers/hour
  - Used: in-line CD monitoring at every critical litho step

Scatterometry / OCD (Optical CD):
  - Shines light on periodic structures, measures diffracted spectrum
  - Non-destructive, high throughput (>100 WPH)
  - Fits measured spectrum to model → extracts CD, sidewall angle, thickness
  - Used: in-line monitoring for litho and etch

Overlay metrology:
  - Measures alignment between successive litho layers
  - DBO (Diffraction-Based Overlay): <0.1nm measurement uncertainty
  - Target: <1nm overlay for EUV layers

Defect inspection:
  - Broadband plasma: KLA 39xx series — unpatterned and patterned inspection
  - E-beam inspection: ASML/HMI multi-beam — highest sensitivity but slow
  - Detects particles, pattern defects, bridge/opens
  - At EUV: stochastic defects (missing contacts, broken lines) are the new challenge

TEM (Transmission Electron Microscopy):
  - Cross-section imaging at atomic resolution
  - Used: failure analysis, process development, gate stack measurement
  - Sample prep: FIB (Focused Ion Beam) milling
  - Destructive but provides the "ground truth"

Atom Probe Tomography:
  - 3D compositional mapping at atomic scale
  - Used: dopant profiling, interface analysis
  - Research/development tool (not in-line)
```

---
