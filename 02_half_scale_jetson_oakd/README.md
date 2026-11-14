# Project 2 — Half-Scale AI & Hybrid-BLDC Platform (`Jetson Orin Nano Super` + `OAK-D Lite`)

**Role:** The primary **Edge AI & Quasi-Direct-Drive (QDD) robotics development platform**. Upgrades from serial hobby servos to `48 V` torque-controlled BLDC actuators, **hybrid buy/DIY hip + CNC/PA-CF knee drives**, **Jetson Orin Nano Super (8 GB, 67 TOPS)**, and **Luxonis OAK-D Lite** stereo depth + on-VPU YOLO so your RL locomotion policies (trained on your Jetson AGX Thor), obstacle safety stops, "Follow Me" tracking, and bilingual Whisper + local LLM voice parsing are proven at meaningful mechanical stress (`10 kg` payload, `12–15 cm` stairs) before spending `$11k–$18k` on Stage 3.

## Target Specifications

- **Mass & Payload:** `~20 kg` empty + `10 kg` payload (`30 kg` loaded)
- **Step Climbing Target:** `12–15 cm` curbs & stairs (`< >` X-configuration legs)
- **Actuators (Hybrid):** 8× bought 8108-class planetary CAN-FD hip BLDCs (`25–35 N·m` peak) + 4× DIY co-axial `8108/6010` + HTD-5M belt or CNC cycloidal knees (`45–60 N·m` peak) + 4× `6010` wheel drives with FOC ESCs
- **Compute & Vision:** **Jetson Orin Nano Super (8 GB, `$249`)** + **Luxonis OAK-D Lite Fixed-Focus (`$149`)** + **Teensy 4.1 (`500 Hz` dual CAN-FD)** + `ICM-42688-P` SPI IMU
- **AI Suite:** On-VPU YOLOv8n obstacle/person stop (`~0` Jetson GPU load), TensorRT FP16 RL walking/climbing policy, `faster-whisper` GPU ASR + quantized 1B–1.5B local LLM JSON parser (`EN/ES`), Piper TTS, optional voice-toggled "Follow Me"
- **Power & Appliances:** `48 V 20 Ah` (`~960–1,000 Wh`), `19 V` Orin rail, USB-C PD (`100 W`), `12 V` DC igniter, `300 W` pure-sine AC inverter demo, `40–50 W` lid solar input, hardware drive-away interlock
- **Estimated DIY Cost:** **`~$3,500–$4,800 USD`** *(baseline BOM estimate `~$4,273 USD`)*

## Documents in This Folder

1. [`01_mechanical_and_actuation.md`](./01_mechanical_and_actuation.md) — `30 kg` torque sizing, hybrid buy-hip / CNC+PA-CF knee architectures, `6010` wheel drives, and `Teensy 4.1` dual CAN-FD topology.
2. [`02_ai_vision_and_software.md`](./02_ai_vision_and_software.md) — Jetson Orin Nano Super + OAK-D Lite ROS 2 Humble architecture, `8 GB` RAM budget across YOLO/RL/Whisper/LLM/Piper, and Thor → Orin TensorRT sim-to-real pipeline.
3. [`03_power_and_bom.md`](./03_power_and_bom.md) — `48 V` battery pack sizing (`~4.5 hr` target cruise), `19 V / 12 V / 5 V` DC-DC + `300 W` inverter demo + drive-away interlock, and itemized BOM with Mexico sourcing notes.

See also the shared industrial look and CAD rules in [`../shared/concept_design.md`](../shared/concept_design.md).
