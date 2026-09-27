# IX. LAYER: BIOMEDICAL ENGINEERING (Complete)

## IX-A. Biosignal Sensing

```
ECG (Electrocardiogram):
  - Measures heart electrical activity via skin electrodes
  - Frequency range: 0.05-150 Hz (diagnostic), 0.5-40 Hz (monitoring)
  - Amplitude: 0.1-5 mV
  - Lead configurations: 3-lead (basic), 12-lead (diagnostic)
  - HRV (Heart Rate Variability): time-domain (SDNN, RMSSD), frequency-domain (LF/HF ratio)
  - Arrhythmia detection: atrial fibrillation, VT, PVC — ML classification
  - AFE: ADS1292R (TI), MAX30003 (Maxim), AD8232 (Analog Devices)

EEG (Electroencephalogram):
  - Measures brain electrical activity via scalp electrodes
  - Frequency bands: Delta (0.5-4Hz), Theta (4-8Hz), Alpha (8-13Hz),
    Beta (13-30Hz), Gamma (30-100Hz)
  - Amplitude: 1-100 µV (very small — high-gain, low-noise amp required)
  - Electrode count: 1-256 channels
  - BCI applications: motor imagery classification, SSVEP, P300
  - AFE: ADS1299 (TI) — 8-channel, 24-bit, <1µVpp noise

EMG (Electromyogram):
  - Measures muscle electrical activity
  - Frequency: 20-500 Hz
  - Amplitude: 0.01-10 mV (surface), up to 30mV (needle)
  - Prosthetic control: classify hand gestures from forearm EMG
  - Fatigue detection: median frequency shift during sustained contraction

SpO2 (Pulse Oximetry):
  - Red (660nm) + IR (940nm) LED → photodetector
  - Beer-Lambert law: ratio of absorption at two wavelengths → oxygen saturation
  - MAX30102 (Maxim): integrated SpO2 + heart rate sensor
  - Accuracy: ±2% SpO2 (consumer), ±1% (medical grade)

Respiratory sensing:
  - Impedance pneumography: chest impedance changes with breathing
  - Strain gauge: chest belt with resistive stretch sensor
  - Thermistor: nasal airflow temperature change
  - Radar-based: 24GHz/60GHz FMCW radar detects chest wall movement (non-contact)
  - WiFi CSI: channel state information changes from chest movement (contactless)

Additional biosignals:
  - EDA/GSR (Electrodermal Activity): skin conductance → stress/arousal
  - PPG (Photoplethysmography): optical pulse wave → heart rate, BP estimation
  - EOG (Electrooculography): eye movement tracking via periocular electrodes
  - Body temperature: thermistor, IR thermopile, in-ear (tympanic)
  - Blood pressure: cuff-based (oscillometric), cuff-less (PPG + ML estimation)
  - Glucose: CGM (continuous glucose monitor), ISF (interstitial fluid) sampling
```

## IX-B. Signal Processing (Complete)

```
Bandpass filtering:
  - Butterworth: maximally flat passband
  - Chebyshev: sharper cutoff, passband ripple
  - Bessel: best group delay (preserves waveform shape)
  - FIR (Finite Impulse Response): linear phase, higher order needed
  - IIR (Infinite Impulse Response): lower order, may introduce phase distortion
  - Implementation: biquad cascade (second-order sections) for numerical stability

Notch filtering:
  - Remove powerline interference (50/60 Hz and harmonics)
  - Narrow notch: Q > 30 for minimal signal distortion
  - Adaptive notch: tracks frequency drift in real-time (LMS algorithm)

Adaptive filtering:
  - LMS (Least Mean Squares): simple, robust, slow convergence
  - RLS (Recursive Least Squares): fast convergence, higher computation
  - Kalman filter: optimal for linear systems with Gaussian noise
  - Used for: motion artifact removal, reference noise cancellation

Artifact removal:
  - Motion artifacts: high-pass filter (>0.5Hz for ECG), adaptive cancellation
    with accelerometer reference signal
  - Baseline wander: high-pass filter or polynomial fitting + subtraction
  - Muscle artifact in EEG: ICA (Independent Component Analysis) decomposition
  - Electrode pop: detect + interpolate, or reject segment
  - EMG contamination in ECG: Wiener filtering, wavelet denoising

Advanced noise suppression:
  - Wavelet denoising: decompose signal, threshold detail coefficients, reconstruct
    Wavelet selection: Daubechies db4/db8 for ECG, Symlet for EEG
  - Empirical Mode Decomposition (EMD): data-driven, no basis function assumed
  - Deep learning denoising: autoencoder trained on clean/noisy signal pairs
  - Ensemble averaging: average multiple beats/epochs to reduce random noise
```

## IX-C. Bioelectronics Hardware

```
Low-noise biopotential amplifiers:
  - Input-referred noise: <1 µVpp (0.5-100 Hz) for EEG
  - Input impedance: >1 GΩ (to work with high-impedance dry electrodes)
  - CMRR (Common-Mode Rejection Ratio): >110 dB
  - Gain: 100-10,000× (adjustable)
  - Architecture: instrumentation amplifier (3-op-amp or single-chip)
  - Chopper stabilization: eliminates 1/f noise (flicker noise)
  - Right-leg drive (RLD): active common-mode cancellation circuit

Analog Front-End (AFE) design:
  - Anti-aliasing filter → amplifier → ADC
  - Sigma-delta ADC: 24-bit resolution, inherent anti-aliasing
  - SAR ADC: faster, lower resolution (12-16 bit), for high-speed applications
  - Sample rate: 250-1000 Hz (ECG), 250-2000 Hz (EEG), 1-10 kHz (EMG)
  - Multiplexing: switch between electrodes to share ADC (reduces channel count)

Electrode technology:
  - Wet (Ag/AgCl): gold standard, low impedance, requires gel
  - Dry (metal, conductive polymer): no gel, higher impedance, motion artifacts
  - Textile: conductive yarn woven into fabric, comfortable for long-term
  - Microneedle: penetrate stratum corneum, very low impedance, semi-invasive
  - Capacitive: non-contact through clothing, highest noise, good for long-term

Skin-electrode impedance:
  - Model: R_s (spreading resistance) + [R_ct || C_dl] (charge transfer + double layer)
  - Wet electrode: 1-10 kΩ at 10 Hz
  - Dry electrode: 10-1000 kΩ at 10 Hz
  - Impedance decreases with frequency (capacitive component)
  - Pre-treatment: alcohol wipe, light abrasion → reduces impedance 10×
```

## IX-D. Implantable Systems

```
Sub-dermal MEMS sensors:
  - Pressure sensors: capacitive MEMS, implanted in arteries/brain ventricles
  - Chemical sensors: ISFET (Ion-Sensitive FET) for pH, glucose
  - Accelerometers: fall detection in hip implants
  - Packaging: hermetic sealing (ceramic, titanium, glass-silicon)
  - Telemetry: inductive coupling, ultrasonic power/data
  - Lifetime: 5-15 years (limited by battery or bio-fouling)

Flexible electronics:
  - Substrate: polyimide (Kapton), PDMS, parylene-C
  - Conductors: thin-film Au, Pt, PEDOT:PSS (conductive polymer)
  - Transistors: organic TFTs, oxide TFTs (IGZO), thinned silicon
  - Applications: conformable neural arrays, e-skin, strain sensors
  - Mechanical: bend radius <1mm, >100,000 flex cycles without failure

Neural interfaces (BCI — complete):
  - Utah array (Blackrock Neurotech): 96-electrode silicon microarray
    - Penetrating, cortical implant, FDA cleared for research
    - Signal: single-unit action potentials (spikes)
    - Limitation: tissue encapsulation over months (signal degrades)
  - Neuralace (Blackrock): ultra-high-channel flexible arrays
    - 10,000+ channels on flexible polymer substrate
    - Reduced tissue damage, longer signal longevity
    - Research stage (2025-2026)
  - Neuralink N1: 1024 electrodes, 64 threads, robotic insertion
    - PRIME study: 21 participants by 2026
    - Wireless: BLE telemetry, inductive charging
    - Brain-to-cursor control demonstrated in human subjects
  - Stentrode (Synchron): endovascular BCI
    - Inserted via blood vessels (jugular vein → superior sagittal sinus)
    - No open brain surgery required
    - 16 electrodes on stent scaffold
    - Records EEG-like signals from cortical surface via vasculature
    - Nvidia collaboration: AI-powered signal decoding
    - COMMAND study: FDA IDE trial underway
  - ECoG (Electrocorticography): surface electrode grids on cortex
    - Higher SNR than EEG, less invasive than penetrating arrays
    - Used: speech decoding (UCSF), motor decoding
  - Closed-loop neuromodulation:
    Sense → decode → stimulate → verify → adapt
    Example: detect pre-seizure activity → apply electrical stimulation → abort seizure
    RNS System (NeuroPace): FDA-approved responsive neurostimulation

Biocompatible materials:
  - Silicone (PDMS): flexible encapsulation, USP Class VI
  - Parylene-C: CVD-deposited conformal coating, FDA Class VI
  - Titanium: hermetic packages, osseointegration capability
  - Platinum/Iridium: electrode material, corrosion-resistant
  - Hydrogel: tissue-mimetic, drug-eluting coatings
  - Graphene: biocompatible conductor, transparent, flexible
  - Carbon nanotubes: neural electrode coating (lower impedance)

Circulatronics & Injectable Bioelectronics (MIT 2025):
  - Floating, wireless, injectable nano-implants in the bloodstream
  - Self-powered: harvest energy from blood flow or glucose
  - Self-navigate: travel through vasculature to target tissue
  - Functions: localized drug delivery, sensing, stimulation
  - No surgery: injected via standard IV
  - Status: animal studies, proof-of-concept
```

## IX-E. Therapeutics (Complete)

```
Closed-loop drug delivery:
  - Sensor (glucose) → controller (algorithm) → actuator (insulin pump)
  - Artificial pancreas: Medtronic 780G, Omnipod 5, DIY (OpenAPS/Loop)
  - PID or MPC (Model Predictive Control) algorithms
  - Safety: low-glucose suspend, max dose limits, redundant sensors
  - Beyond insulin: closed-loop analgesia, chemotherapy dosing

Neurostimulation:
  - TENS (Transcutaneous Electrical Nerve Stimulation):
    Surface electrodes, pain management, 2-150 Hz, non-invasive
  - DBS (Deep Brain Stimulation):
    Implanted electrodes in subthalamic nucleus or GPi
    Treats: Parkinson's, essential tremor, dystonia, OCD
    Adaptive DBS: adjust stimulation based on neural biomarkers
    Devices: Medtronic Percept PC (sensing + stimulation)
  - tDCS (Transcranial Direct Current Stimulation):
    1-2 mA DC through scalp, modulates cortical excitability
    Non-invasive, research for depression, motor learning
  - Vagus Nerve Stimulation (VNS):
    Electrode on cervical vagus nerve
    FDA approved: epilepsy, depression
    Emerging: bioelectronic medicine — treat inflammation, autoimmune
  - Electroceuticals:
    Replace drugs with targeted electrical stimulation
    Galvani Bioelectronics (GSK + Verily): miniature neural implants
    Target: rheumatoid arthritis, diabetes, hypertension via nerve modulation

Smart prosthetics:
  - Myoelectric control: EMG signals → proportional actuator control
  - Pattern recognition: ML classifies EMG → multiple grip patterns
  - Sensory feedback: vibrotactile or electrotactile → user feels grip force
  - Osseointegration: titanium implant fused to bone → direct skeletal attachment
    Eliminates socket → better proprioception, comfort, range of motion
    OPRA system (Integrum): FDA cleared, 20+ year track record
  - Targeted Muscle Reinnervation (TMR):
    Reroute residual nerves to chest muscles → more EMG sites → finer control
  - Osseointegration 2.0:
    Hardware physically bonds with bone and tissue at molecular level
    Bioactive coatings: hydroxyapatite, growth-factor-eluting surfaces
    Goal: zero rejection, lifelong integration

Lab-on-a-Chip / Microfluidics:
  - EWOD (Electrowetting-on-Dielectric): programmable droplet manipulation
    Move, split, merge microliter droplets on electrode array
    → Digital microfluidics: no pumps, no channels, software-defined
  - Integration with VLSI: sensor + processor + fluidics on same die
  - Real-time biochemistry: glucose, cortisol, lactate, pH, electrolytes
  - Interstitial fluid sampling: microneedle patch → EWOD chip → sensor
  - Point-of-care diagnostics: blood-to-result in <15 minutes
  - Organ-on-chip: microphysiological systems for drug testing
    Lung-on-chip, heart-on-chip, liver-on-chip (Emulate Inc.)
```

## IX-F. Compliance (Complete)

```
ISO 13485:2016: QMS for medical device design and manufacturing
IEC 62304: Software lifecycle for medical devices
  - Class A (no injury), B (non-serious), C (death/serious injury)
  - Class C requires: formal requirements, architecture, detailed design,
    unit testing, integration testing, system testing
IEC 60601-1: Safety and essential performance of medical electrical equipment
  - Part 2-47: ambulatory ECG systems
  - Part 1-11: home healthcare environment
FDA Design Controls (21 CFR 820.30):
  - Design input → output → review → verification → validation → transfer
  - Design History File (DHF): complete documentation trail
FDA 510(k) / De Novo / PMA pathways
  - 510(k): substantially equivalent to predicate device
  - De Novo: new device type, low-moderate risk
  - PMA: highest risk, clinical evidence required
QMSR (Quality Management System Regulation):
  - FDA final rule effective Feb 2, 2026
  - Aligns 21 CFR 820 with ISO 13485:2016
  - Eliminates dual compliance burden (one QMS serves both)
EU MDR (Medical Device Regulation 2017/745):
  - Class I (low risk) → Class III (highest risk)
  - Clinical evaluation required for all classes
  - Notified Body assessment for Class IIa/IIb/III
  - Technical documentation: risk management (ISO 14971),
    biocompatibility (ISO 10993), usability (IEC 62366)
HIPAA (US health data privacy)
GDPR (EU data protection — applies to medical devices processing personal data)
```

## IX-G. Synthetic Biology & Biofabrication (Complete — from v6.0)

```
Organoid Intelligence (OI):
  - Brain organoids: 3D structures grown from human iPSCs
  - Contain neurons that form functional networks (fire, learn, adapt)
  - DishBrain (Cortical Labs): organoid learned to play Pong
  - As co-processor: organoid networks for pattern recognition,
    adaptive control, tasks where biological neural efficiency exceeds silicon
  - Integration: microelectrode arrays (MEAs) interface organoid ↔ electronics
  - Northwestern 2025: organoid-bioelectronics mesh — flexible electronics
    conformally wrap around organoid for high-density recording/stimulation
  - Ethical considerations: sentience boundaries, informed consent frameworks

Living electrodes:
  - Neurons grown along engineered scaffolds to create biological "wires"
  - Interface between implanted electronics and neural tissue
  - Advantages: biocompatible, self-repairing, integrate with host tissue
  - Status: Penn Medicine, in vitro demonstrations

Bacterial-silicon neural interfaces:
  - Engineered bacteria as biosensors (sense neurotransmitters, metabolites)
  - Bacteria attached to silicon sensor chip → biological + electronic sensing
  - Macquarie University: semisynbio perspective on living electronics

CRISPR-engineered cells as bio-actuators:
  - Cells engineered to respond to electronic signals
  - Optogenetics: light-sensitive proteins → control cell behavior with LED
  - Electrogenetics: cells respond to applied voltage → produce therapeutic proteins
  - In-vivo CRISPR actuation: gene editing triggered by electronic signal

Semisynbio / DNA data storage:
  - Store digital data in synthetic DNA
  - Density: theoretical 1 exabyte per gram
  - Write: enzymatic DNA synthesis (Twist Bioscience, Catalog)
  - Read: nanopore sequencing (Oxford Nanopore)
  - Durability: DNA stable for >1000 years in cold/dry storage
  - Cost: still expensive (~$3500/MB write), but dropping fast
  - Use case: archival storage, long-term medical records
```

---
