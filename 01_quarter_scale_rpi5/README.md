# Project 1 — Quarter-Scale Wheel-Legged Validation Platform (`RPi 5` + `Teensy 4.1`)

**Role:** Low-cost, fast-iteration bench & backyard testbed to validate **Symmetric `< >` X-configuration kinematics**, wheel-leg mode switching, Teensy-to-SBC `200 Hz` telemetry, offline bilingual (`EN / ES`) voice command parsing, ELRS manual override, USB-C PD + 12 V spark igniter + 20 W solar charging, and **Autodesk Fusion 360 → Blender → MuJoCo** CAD-to-sim workflow.

## Target Specifications

- **Mass & Payload:** `~3.8 kg` empty + `1.0 kg` payload (`~4.8–5.0 kg` loaded)
- **Step Climbing Target:** `6 cm` curbs / mock stairs (`< >` X-configuration legs)
- **Actuators:** 12× Feetech STS3250 serial bus servos (`4.9 N·m` stall; `2:1` GT2 9 mm belt on knees for `~1.94 N·m` continuous / `~9.3 N·m` stall) + 4× 25GA-370 brushed geared wheel motors (`330 RPM`, `1,496` counts/rev quadrature encoders)
- **Compute & Vision:** Raspberry Pi 5 (8 GB) + Pi Camera Module 3 Wide (`IMX708`) + Teensy 4.1 (`Serial1/Serial2` @ 1 Mbps) + ICM-42688-P IMU
- **Voice & Radio:** ReSpeaker 4-Mic on isolated `120 mm` mast, openWakeWord + Vosk EN/ES small grammar + Piper TTS, RadioMaster ELRS 2.4 GHz override
- **Power & Solar:** 3S4P Molicel P42A (`186.5 Wh`), USB-C PD (`65–100 W`), 12 V DC spark igniter (`IRF7424`), `20 W` quick-disconnect solar panel on insulated box lid (`BQ24650` MPPT)
- **Estimated DIY Cost:** `~$1,880–$2,150 USD` component baseline (`$1,881.74` hybrid `8× 12V STS3215 + 4× STS3250` or `$2,149.74` all-`STS3250`, plus ~10–15% Mexico shipping/IVA; printer/dryer tooling separate)

## Documents in This Folder

0. [`00_10day_gated_schedule.md`](./00_10day_gated_schedule.md) — **10-Day Gated Execution Schedule (Nov 15–24, 2026)** with 2-day decision deadlines and mandatory approval hold points (`Gate 1` through `Gate 5`).
1. [`01_mechanical_and_cad.md`](./01_mechanical_and_cad.md) — Dimensions, `< >` X-config IK sign conventions, CNC `6061-T6` + annealed `PA-CF` rules, Fusion 360 → Blender → MuJoCo export workflow, and TPU 95A airless wheel design. *(Note: the starter MJCF snippet at the bottom of `01_mechanical_and_cad.md` uses a sample FL leg; when authoring RL/RR legs for symmetric `< >` X-config, mirror rear `knee_pitch_joint` range to `0.0 2.618` and flip `hip_pitch` stance nominal to `-45°`.)*
2. [`02_actuation_and_sizing.md`](./02_actuation_and_sizing.md) — Joint torque calculations, STS3250 thermal limits, Dual-UART `200 Hz` bus timing, and Stage 2/3 actuator scaling comparison.
3. [`03_voice_and_hri.md`](./03_voice_and_hri.md) — Bilingual EN/ES pipeline, `< 45 ms` fast-path emergency `Stop`/`Alto`/`Para`, acoustic mast mounting, and Jetson Thor dev parity notes.
4. [`04_power_and_electronics.md`](./04_power_and_electronics.md) — 3S4P BMS schematic notes, USB-C PD, 12 V spark igniter safety, hardware drive-away lockout (`74LVC1G32`), and Stage 3 electrical preview.
5. [`05_sourcing_and_bom.md`](./05_sourcing_and_bom.md) — PA-CF printer comparison (`QIDI Plus4` / `Bambu H2D`), itemized Project 1 BOM, Mexico DDP/SAT `33.5%` duty notes, and local 6061/21700 suppliers.

See also the shared industrial look and CAD rules in [`../shared/concept_design.md`](../shared/concept_design.md).
