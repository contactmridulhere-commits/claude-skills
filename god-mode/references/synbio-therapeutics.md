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

---

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

