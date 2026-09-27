# VI. LAYER: FPGA & RECONFIGURABLE COMPUTE (Complete)

```
Core:
  - Verilog / SystemVerilog: synthesizable RTL for FPGA
  - VHDL: alternative HDL, common in aerospace/defense
  - Synthesis: Vivado (Xilinx/AMD), Quartus (Intel/Altera)
  - Timing closure: meet Fmax across routing/placement
  - Resource utilization: LUTs, FFs, BRAMs, DSP blocks, UltraRAM
  - Systolic array design: matrix multiply engines in fabric
  - Pipeline optimization: insert registers to maximize clock frequency
  - Parallel compute: exploit spatial parallelism (vs temporal in CPUs)

Advanced:
  - DFX (Dynamic Function Exchange): swap logic regions at runtime
    Use case: reconfigure accelerator for different AI models without reboot
  - HLS (High-Level Synthesis): C/C++ → RTL (Vitis HLS, Intel HLS)
    Trade: faster development, ~20-30% area/timing overhead vs hand-coded RTL
  - Partial reconfiguration: change part of FPGA while rest runs
  - Multi-die FPGA: Xilinx/AMD Versal, Intel Agilex (chiplet-based FPGA)
  - Hardened AI engines: AMD AIE (AI Engine) tiles in Versal
  - Integrated ARM cores: Zynq UltraScale+, Intel Agilex SoC
  - Embedded processors: soft-core (MicroBlaze, Nios) vs hard-core (ARM)
  - High-speed transceivers: 32-112 Gbps SerDes for networking
  - Power analysis: dynamic + static, clock gating in FPGA fabric

Safety / verification:
  - Formal verification: prove properties on FPGA design
  - Fault tolerance: TMR (Triple Modular Redundancy), ECC on BRAMs
  - SEU (Single Event Upset) mitigation: scrubbing, rad-hard by design
  - Hardware validation pipelines: in-system debug (ILA, SignalTap)
  - DO-254 (avionics), IEC 61508 (industrial safety) compliance paths
```

---

# VII. LAYER: AI / ML SYSTEMS (Complete)

```
Core:
  - Transformer architecture: self-attention, MLP, LayerNorm, positional encoding
  - Multi-head attention: Q×K^T/√d_k → softmax → ×V, parallelized across heads
  - Hardware mapping: systolic arrays for matmul, special-function units for softmax/GELU
  - Quantization:
    INT8: 4× compute density vs FP32, <1% accuracy loss with calibration
    INT4: 8× density, requires quantization-aware training (QAT)
    FP8 (E4M3/E5M2): Nvidia H100 format, better dynamic range than INT8
    Posit arithmetic: tapered precision, better accuracy per bit than IEEE FP
      16-bit posit ≈ 32-bit float accuracy for many workloads
      Hardware: slower adoption, but ideal for medical AI (precision matters)
    Binary/ternary: extreme quantization for MCU-class devices
  - ONNX deployment: framework-agnostic model format → TFLite, TensorRT

Advanced:
  - Multimodal AI: fuse vision (camera) + biosignals (ECG/EEG) + audio (mic)
    Cross-attention between modalities, modality-specific encoders
  - Agentic systems: LLM-driven closed-loop control
    Observe → Think → Act → Verify cycle
    Tool use: GPIO control, sensor queries, actuator commands
    Safety: watchdog timers, action whitelists, human-in-the-loop gates
  - RAG (Retrieval-Augmented Generation):
    Embed → vector store → retrieve top-k → augment prompt → generate
    On-device: MiniLM embeddings + ChromaDB + llama.cpp (Phi-3)
  - Mixture of Experts (MoE): route tokens to specialized sub-networks
    → Larger model capacity with same compute (only k/n experts active)
  - Speculative decoding: draft model proposes tokens, main model verifies
    → 2-3× faster inference with identical output quality

Optimization:
  - Pruning: remove weights below threshold → sparse matrix → skip zero ops
    Unstructured: any weight (flexible, needs sparse hardware)
    Structured: entire channels/heads (simpler hardware, coarser)
    Magnitude pruning, movement pruning, lottery ticket hypothesis
  - Distillation: train small student model to mimic large teacher
    → Transfer knowledge without transfer of model size
    Label smoothing, intermediate layer matching, attention transfer
  - Deterministic inference: fixed-point arithmetic, no stochastic rounding
    → Bit-exact reproducibility required for medical/safety-critical AI
  - TinyML / Edge Impulse:
    Models <100KB for MCU deployment (Cortex-M4, ESP32)
    Keyword spotting, anomaly detection, gesture recognition
    Edge Impulse: cloud training → optimized TFLite Micro deployment
  - Neural Architecture Search (NAS):
    Automatically discover optimal model architecture for target hardware
    Hardware-aware NAS: constrain search to models that meet latency/power budget
```

---
