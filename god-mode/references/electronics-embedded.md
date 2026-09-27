# X. LAYER: ELECTRONICS (Complete)

```
Core:
  - Analog circuit design: op-amp circuits, filters, oscillators, references
  - Digital circuit design: combinational, sequential, FSM, timing
  - Signal integrity: reflection, crosstalk, ground bounce, decoupling
  - Power electronics: buck/boost/LDO regulators, MOSFET drivers, H-bridges

Advanced:
  - High-speed PCB design:
    Controlled impedance traces (50Ω single-ended, 100Ω differential)
    Via stitching, ground plane integrity, layer stack optimization
    Length matching for DDR/PCIe/USB3 differential pairs
  - Impedance matching: Smith chart, L/T/Pi networks, transmission line theory
  - EMI/EMC compliance: FCC Part 15, CISPR 32, IEC 61000
    Shielding, filtering, grounding strategy, common-mode chokes
  - Thermal management: heatsink design, thermal vias, copper pours, TIMs
  - Self-healing analog circuits:
    Graphene-PEDOT:PSS composites that recover >80% function in <10 seconds
    Liquid metal (gallium alloys) micro-channels as self-repairing conductors
    For implantable/wearable: survives body movement, mechanical stress

Power:
  - Battery management systems (BMS): cell balancing, SoC estimation, protection
  - Energy optimization: DVFS, clock gating, power gating, retention modes
  - Energy harvesting:
    Thermoelectric (TEG): body heat → electricity (Seebeck effect)
      ~20-40 µW/cm² from ΔT = 5°C (skin to ambient)
      Enough for: low-power sensors, BLE beacon
    Piezoelectric: mechanical vibration → electricity
      ~10-100 µW from walking, heartbeat
    RF harvesting: ambient WiFi/cellular → rectified DC
      ~1-10 µW at typical ambient RF levels
    Solar (indoor): amorphous Si or organic PV under indoor lighting
      ~10-100 µW/cm²
    Biofuel cell: glucose + oxygen → electricity (enzymatic)
      ~1-10 µW from interstitial fluid
    Energy neutrality: harvesting ≥ consumption → infinite battery life
```

---

# XI. LAYER: EMBEDDED SYSTEMS (Complete)

```
Platforms:
  - ESP32: dual-core 240MHz, WiFi/BLE, 520KB SRAM, ultra-low-power modes
  - STM32: M0 to M7, FPU, DSP, DMA, rich peripherals
  - Raspberry Pi (3/4/5/Zero): full Linux, GPU, camera, edge AI
  - Arduino: AVR/ARM, massive ecosystem, beginner to intermediate
  - RP2040 (Pico): dual M0+, PIO state machines, flexible pin mux

Core:
  - RTOS:
    FreeRTOS: most popular, small footprint, tasks/queues/semaphores
    Zephyr: modern, supports 500+ boards, device tree, BLE stack
    NuttX: POSIX-compliant, used in PX4 (drones)
    RT-Thread: Chinese ecosystem, growing globally
    ThreadX (Azure RTOS): deterministic, safety-certified
  - Bare-metal programming: direct register manipulation, no OS overhead
  - Interrupts: priority levels, nesting, ISR design (keep short!)
  - DMA: offload memory transfers from CPU, zero-copy data movement
  - Timers: PWM generation, input capture, one-shot/periodic, watchdog

Advanced:
  - Deterministic scheduling: rate-monotonic, EDF (Earliest Deadline First)
  - Power-aware firmware: sleep modes, wake sources, duty cycling
  - Low-level hardware control: register-level GPIO, peripheral configuration
  - Bootloader design: secure boot, firmware update (OTA/DFU)
  - Mixed-criticality systems: safety-critical + non-critical on same processor
    ARM TrustZone: hardware isolation between secure and non-secure worlds
```

---

# XVI. LAYER: OS & LOW-LEVEL (Complete)

```
Core:
  - Memory management: virtual memory, MMU, MPU (embedded), stack/heap
  - Drivers: character/block/network, device tree binding, interrupt handling
  - Kernel basics: process scheduling, IPC, file systems, syscalls

Advanced:
  - Embedded Linux: Buildroot, Yocto, kernel configuration, device trees
  - Custom OS design:
    Minimal kernel: scheduler + IPC + memory manager + driver framework
    Microkernel (seL4): formally verified, capability-based security
      seL4: mathematical proof of correctness, no bugs in kernel
      Used: DARPA-funded military systems, autonomous vehicles
    Unikernel: single-address-space OS for single application
      → Minimal attack surface, sub-second boot
  - Real-time kernels:
    PREEMPT_RT patch: make Linux fully preemptible
    Worst-case latency: <50µs with tuning (vs ~ms for standard Linux)
    Xenomai: dual-kernel approach (RT co-kernel alongside Linux)
    QNX: commercial RTOS, POSIX-compliant, ASIL-D automotive certified
```

---
