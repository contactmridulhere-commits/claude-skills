# XIV. LAYER: WEB + HARDWARE INTEGRATION (Complete)

```
Core:
  - APIs for hardware control: REST, GraphQL endpoints for sensor/actuator access
  - Real-time dashboards: React/Vue frontends, Chart.js/D3 live data
  - WebSocket communication: bidirectional, <10ms latency on LAN

Advanced:
  - Remote device control: WebRTC for peer-to-peer, VPN tunneling
  - Cloud sync (optional fallback): MQTT → cloud broker → mobile app
  - Device telemetry systems: InfluxDB + Grafana time-series monitoring
  - Three.js / WebGL: 3D hardware visualization, digital twins in browser
  - Progressive Web Apps: offline-capable hardware dashboards
```

---

# XV. LAYER: CONNECTIVITY (Complete)

```
Core:
  - BLE 5.x: long range (coded PHY), high throughput (2 Mbps), advertising extensions
  - WiFi 6/6E: OFDMA, TWT (Target Wake Time), 6 GHz band
  - ESP-NOW: peer-to-peer, low-latency (~1ms), no router needed, 250 byte payload
  - MQTT: pub/sub, QoS 0/1/2, last will, retained messages

Advanced:
  - LoRaWAN mesh: 10+ km range, <50 kbps, battery life years
  - WiFi 7 (802.11be): 320 MHz channels, 4096-QAM, Multi-Link Operation
  - Microsecond synchronization:
    PTP (Precision Time Protocol) IEEE 1588: <1µs sync over Ethernet
    TSN (Time-Sensitive Networking): deterministic Ethernet for real-time
    White Rabbit: sub-ns sync (CERN, used in distributed physics experiments)

Medical:
  - HL7: health information exchange standard (v2 messages, FHIR RESTful)
  - FHIR (Fast Healthcare Interoperability Resources): modern health data API
    JSON-based, RESTful, resources (Patient, Observation, DiagnosticReport)
  - Secure transmission: TLS 1.3, certificate pinning, HIPAA-compliant encryption
  - Bluetooth Medical Device Profile: standardized health data streaming
  - ANT+: ultra-low-power health sensor protocol (heart rate, power meters)
```

---

# XIX. LAYER: CYBERSECURITY (Complete)

```
Core:
  - Encryption:
    AES-256: symmetric, block cipher, hardware-accelerated on most MCUs
    ECC (Elliptic Curve): asymmetric, smaller keys than RSA (256-bit ≈ 3072-bit RSA)
    ChaCha20-Poly1305: stream cipher, fast on ARM without AES hardware
  - Secure boot:
    Chain of trust: ROM bootloader verifies first-stage → first verifies second → ...
    Each stage cryptographically signed
    Root of trust in hardware (OTP fuses or ROM)
  - Hardware root of trust:
    Immutable boot ROM, secure key storage, hardware RNG
    ARM TrustZone, Intel SGX, RISC-V PMP

Advanced:
  - Side-channel resistance:
    Power analysis (SPA/DPA): measure power consumption → extract key bits
    Countermeasures: constant-time code, random delays, noise injection
    Electromagnetic analysis: near-field probe → EM emissions → key extraction
    Countermeasure: EM shielding, algorithmic masking
  - Fault injection protection:
    Voltage glitching, clock glitching, laser fault injection
    Countermeasures: voltage/clock monitors, redundant computation, error detection
  - Secure firmware updates:
    Signed firmware images (ECDSA signature verification)
    Rollback protection (monotonic counter in OTP)
    A/B partition scheme: fail-safe update (revert on failure)

  Post-quantum cryptography (PQC):
    - Current ECC/RSA vulnerable to Shor's algorithm on quantum computers
    - NIST PQC standards (finalized 2024):
      ML-KEM (CRYSTALS-Kyber): key encapsulation (lattice-based)
      ML-DSA (CRYSTALS-Dilithium): digital signatures (lattice-based)
      SLH-DSA (SPHINCS+): hash-based signatures (stateless)
    - Hardware implementation: PQC accelerators in secure elements
    - Hybrid approach: classical + PQC (defense in depth during transition)
    - Medical device mandate: QMSR requires considering future threats
    - Lattice-based crypto on FPGA: demonstrated at 100 MHz on Artix-7

  Physical Unclonable Function (PUF):
    - Each chip has unique physical fingerprint from manufacturing variation
    - SRAM PUF: random initial state of SRAM cells → unique ID per chip
    - Ring oscillator PUF: frequency variation → device-specific key
    - Use: secure key generation without storing keys in NVM
    - Challenge-response authentication: unclonable, tamper-evident
```

---
