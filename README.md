# Claude Skills

Two Claude Agent Skills I built and use daily. Each skill is a folder with a `SKILL.md` and a set of reference files that Claude loads only when a question needs them.

**Video:** [Claude Skills That Saved My Projects](https://youtube.com/shorts/7lVso38MYGs)

| Skill | Trigger | What it covers |
|---|---|---|
| [`embed-master`](embed-master) | `/embed` | Embedded systems end to end: PCB design, wiring, sensors, motor control, I²C/SPI/UART, power, and firmware for Arduino, ESP32, Raspberry Pi and STM32. Also edge AI (TFLite, ONNX, llama.cpp, Ollama, RAG), computer vision, web dashboards for hardware, 3D printing and parametric CAD, Three.js digital twins and waveguide optics. |
| [`god-mode`](god-mode) | `/god-mode` | Deep engineering and biomedical reference: transistor physics, photolithography, ASIC and FPGA design, advanced packaging, AI hardware, neuromorphic and photonic compute, biomedical sensing and BCIs, synthetic biology and therapeutics. |

## Install

- **Claude Code:** copy a skill folder into `~/.claude/skills/` for yourself, or `.claude/skills/` inside a project.
- **Claude apps:** upload the packaged `embed-master.skill` or `god-mode.skill` file wherever your Claude app lets you add a custom skill.

The folders and the `.skill` packages contain the same files. The folders are there so you can read them on GitHub.

## License

MIT. See [LICENSE](LICENSE).
