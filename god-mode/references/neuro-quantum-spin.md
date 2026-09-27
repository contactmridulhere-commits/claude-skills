# VIII. LAYER: NEUROMORPHIC + QUANTUM + PHOTONIC COMPUTE (Complete)

## VIII-A. Neuromorphic Computing

```
Spiking Neural Networks (SNNs):
  - Neurons communicate via discrete spikes (events), not continuous values
  - Event-driven: zero power when no spikes → ultra-low power
  - Temporal coding: information in spike TIMING, not just rate
  - Hardware:
    Intel Loihi 2: 128 cores, 1M neurons, 120M synapses, 1W
    IBM NorthPole: inference-optimized, 256 cores, 22B transistors
    SynSense Speck: ultra-low-power neuromorphic vision (50µW)
    BrainChip Akida: edge neuromorphic IP for SoC integration
  - Programming: Lava (Intel), sPyNNaker (Manchester), Norse (PyTorch SNN)
  - STDP (Spike-Timing-Dependent Plasticity):
    On-chip learning: strengthen synapse if pre fires before post (causal)
    No backpropagation needed → online, local learning rule
  - Use cases: always-on perception (gesture, keyword), anomaly detection,
    robotics control, brain-computer interfaces

Analog in-memory computing:
  - Perform matrix-vector multiply INSIDE memory arrays
  - Weights stored as conductances in non-volatile memory (ReRAM, PCM, MRAM)
  - Input: voltage applied to rows, output: current summed on columns
  - Kirchhoff's law does the multiply-accumulate (MAC): I = Σ(V_i × G_ij)
  - Energy: ~10-100× more efficient than digital MAC
  - Challenges: device variation, noise, limited precision (4-8 bit effective)
  - Who: Mythic (acquired), Syntiant, IBM, TSMC eMRAM-based CIM
```

## VIII-B. Photonic Computing (FULL — not "next iteration")

```
Silicon photonics fundamentals:
  - Light (photons) carries data instead of electrons
  - Waveguides: sub-micron silicon strips on SiO₂ (SOI wafer)
    Si core: n=3.48, SiO₂ cladding: n=1.44 → strong confinement
    Single-mode waveguide width: ~400-500nm at 1550nm wavelength
    Propagation loss: 1-3 dB/cm (improving to <0.5 dB/cm)

  - Modulators: convert electrical signal to optical
    Ring resonator modulator: resonance shifts with applied voltage
      Q-factor >10,000, bandwidth >25 GHz, footprint ~10µm radius
      Mechanism: carrier injection/depletion changes refractive index
    Mach-Zehnder modulator (MZM): interference-based, broader bandwidth
      Length: ~1-5mm, bandwidth >40 GHz, higher power consumption

  - Photodetectors: convert optical signal back to electrical
    Germanium-on-Silicon (Ge-on-Si): absorption at 1550nm
    Responsivity: ~1 A/W, bandwidth >50 GHz
    Dark current: <1 nA at low bias

  - Wavelength Division Multiplexing (WDM):
    Multiple wavelengths on single waveguide → parallel data channels
    Dense WDM (DWDM): 50-100 GHz channel spacing
    Coarse WDM (CWDM): 20nm channel spacing
    Micro-ring resonator filters: wavelength-selective add/drop
    64+ channels on single fiber/waveguide demonstrated

  - Optical interconnects for chips:
    Replace electrical global interconnects with photonic links
    Advantages: no RC delay, no crosstalk, distance-independent bandwidth
    Co-packaged optics (CPO): photonics die in same package as compute die
    Intel Silicon Photonics: 100G/400G transceivers in production
    TSMC COUPE: co-packaged optics for AI interconnect

Photonic computing for AI (photonic tensor cores):
  - Photonic matrix multiplication (photonic tensor core architecture):
    Input vector encoded as light intensities across waveguides
    Weight matrix encoded as MZI (Mach-Zehnder Interferometer) mesh
    Output: matrix product at speed of light (single pass through mesh)
    Lightmatter Envise: photonic AI accelerator chip
    Luminous Computing: photonic transformer engine
    Energy: ~10fJ per MAC (vs ~1pJ for digital at 3nm)

  - Photonic spiking neural networks:
    All-optical neurons using ring resonator nonlinearity
    Optical STDP: weight update via optical feedback
    272 trainable parameters demonstrated (Optica 2026)
    THz bandwidth: orders of magnitude faster than electronic SNNs
    Target: always-on perception at femtojoule per spike

  - Coherent Ising machines:
    Optical parametric oscillator (OPO) network
    Solves combinatorial optimization (NP-hard problems)
    NTT: 100,000-spin Ising machine demonstrated
```

## VIII-C. Quantum Computing (FULL — not "next iteration")

```
Qubit technologies:
  Superconducting transmon (IBM, Google):
    - Josephson junction (Al/AlOx/Al) operates at 15mK
    - Coherence time: ~100-500µs (improving)
    - Gate fidelity: 99.5% single-qubit, 99.0% two-qubit
    - IBM Eagle: 127 qubits, IBM Condor: 1121 qubits
    - IBM Heron: 133 qubits with improved error rates
    - Google Willow: 105 qubits, below-threshold error correction demonstrated

  Trapped ions (IonQ, Quantinuum):
    - Individual ions held in electromagnetic trap
    - Coherence time: seconds to minutes (much better than SC)
    - Gate fidelity: >99.9% (best of any platform)
    - Quantinuum H2: 56 qubits, all-to-all connectivity
    - Slower gate speed than superconducting (~µs vs ~ns)

  Photonic (PsiQuantum, Xanadu):
    - Qubits encoded in photon states (polarization, time-bin)
    - Room temperature operation (no cryo!)
    - Challenge: probabilistic gates (need many attempts)
    - Xanadu Borealis: 216 squeezed-mode qubits

  Diamond NV-center (Quantum Brilliance):
    - Nitrogen-Vacancy defect in diamond crystal
    - ROOM TEMPERATURE quantum operation
    - Coherence time: ~1ms at RT (with dynamical decoupling)
    - Small qubit count currently (~5-10)
    - Quantum Brilliance: rack-mounted diamond quantum accelerator
    - Ideal for edge quantum computing (no dilution refrigerator)
    - ORNL validated: hybrid classical-quantum scheduling

  Topological (Microsoft):
    - Majorana fermions in nanowire heterostructures
    - Theoretically error-protected by topology
    - Status: Microsoft claimed first topological qubit (2025)
    - Still early, but if it works → exponentially fewer error correction resources

Quantum error correction (FULL — not "next iteration"):
  - Physical qubits have errors → need redundancy
  - Surface code: most studied, threshold ~1% error rate
    Logical qubit = O(d²) physical qubits (d = code distance)
    d=3: 17 physical → 1 logical (minimal protection)
    d=7: ~100 physical → 1 logical (moderate protection)
    d=17: ~600 physical → 1 logical (high protection)
    Google Willow (2025): demonstrated below-threshold behavior
      Error rate decreased as code distance increased (key milestone)
  - Other codes: color codes, LDPC codes, Floquet codes
  - Real-time decoding: classical decoder must keep up with qubit errors
    MWPM (Minimum Weight Perfect Matching) decoder
    Union-Find decoder: O(n) complexity, fast enough for real-time
    Neural network decoders: ML-assisted error correction
  - Fault-tolerant quantum computation:
    Goal: arbitrary-length computation with bounded error probability
    Requires: error rate below threshold + sufficient physical qubits
    Timeline: ~2028-2030 for first fault-tolerant demonstrations (1000+ logical ops)

Quantum-classical hybrid systems:
  - Variational Quantum Eigensolver (VQE): quantum circuit + classical optimizer
  - QAOA: quantum approximate optimization for combinatorial problems
  - Quantum machine learning: parameterized quantum circuits as ML models
  - Edge quantum advantage:
    Diamond NV-center (room temp) as co-processor for specific tasks
    Optimization, sampling, molecular simulation
    Classical CPU handles control, I/O, pre/post-processing
    Quantum Brilliance + ORNL: validated hybrid scheduling framework
```

## VIII-D. Spintronics (FULL — not "next iteration")

```
Spin-based devices:
  - Electron spin (up/down) as information carrier instead of charge
  - Non-volatile: spin state persists without power
  - Low switching energy: potentially <1 aJ per bit

STT-MRAM (Spin-Transfer Torque MRAM):
  - Spin-polarized current switches free layer magnetization
  - Read: TMR (Tunnel Magnetoresistance) — resistance depends on alignment
  - Write: 10-20ns, read: 3-5ns
  - Endurance: >10¹² cycles (vs ~10⁵ for Flash)
  - Embedded MRAM: TSMC 22nm eMRAM in production
  - Used: IoT MCU (replaces Flash + SRAM), automotive, cache replacement

SOT-MRAM (Spin-Orbit Torque MRAM):
  - Current flows through heavy metal layer adjacent to free layer
  - Spin-orbit coupling generates spin current → switches magnet
  - Advantages over STT: faster write (~1ns), separate read/write paths
  - 3-terminal device (vs 2-terminal STT)
  - Status: research/early production, Samsung demos

Magnonic computing:
  - Information encoded in spin waves (magnons) propagating through magnetic media
  - Wave-based computing: interference patterns perform logic operations
  - Wavelength: ~100nm, frequency: 1-100 GHz
  - Ultra-low power: magnons carry angular momentum, not charge
  - Status: research, proof-of-concept logic gates demonstrated

Spin-orbit torque logic:
  - All-spin logic: compute without converting to/from charge
  - MESO (Magneto-Electric Spin-Orbit): Intel's beyond-CMOS research
    Multiferroic material: voltage switches magnetization (no current needed)
    → 10-30× energy reduction vs CMOS for some logic functions
  - Status: research devices, not near production
```

---
