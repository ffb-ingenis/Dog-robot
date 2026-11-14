# Project 1 — 10-Day Gated Execution Schedule (`robot/01_quarter_scale_rpi5`)

**Start Date:** Sunday, November 15, 2026 (Day 1)  
**Completion of 10-Day Gated Plan:** Tuesday, November 24, 2026 (Day 10)  
**Cadence:** 5 Stage Gates (1 Gate every 2 days)  
**Governance Rule:** **Strict Hold Point at Every Gate.** No work or purchasing for Gate $N+1$ proceeds until the user explicitly confirms decisions and approves Gate $N$.

---

## Gate Overview Roadmap

```mermaid
flowchart LR
    G0["Today (Nov 14)\nSchedule Locked"] --> G1["GATE 1 (Day 2: Nov 16)\nProcurement & Tooling Lock"]
    G1 -->|"User Approval"| G2["GATE 2 (Day 4: Nov 18)\nFusion 360 CAD & DXF Lock"]
    G2 -->|"User Approval"| G3["GATE 3 (Day 6: Nov 20)\nBlender/MuJoCo Sim & 1-Leg Fit"]
    G3 -->|"User Approval"| G4["GATE 4 (Day 8: Nov 22)\nWiring, BMS & Safety Interlock"]
    G4 -->|"User Approval"| G5["GATE 5 (Day 10: Nov 24)\nTeensy + RPi 5 Voice & Bring-Up"]
```

| Gate | Window | Deadline | Core Focus | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Gate 1** | Days 1–2 | **Mon, Nov 16, 2026** | Servo Option, Printer/Dryer Selection & Wave-1 Long-Lead Orders | `PENDING` |
| **Gate 2** | Days 3–4 | **Wed, Nov 18, 2026** | Fusion 360 Master Envelope, `< >` X-Leg CAD & CNC 6061-T6 DXFs | `LOCKED UNTIL G1` |
| **Gate 3** | Days 5–6 | **Fri, Nov 20, 2026** | Blender/MuJoCo `< >` Digital Twin & Single-Leg PA-CF/TPU Print Spec | `LOCKED UNTIL G2` |
| **Gate 4** | Days 7–8 | **Sun, Nov 22, 2026** | 3S4P Battery/BMS, Drive-Away Interlock Schematic & Teensy Pinout | `LOCKED UNTIL G3` |
| **Gate 5** | Days 9–10 | **Tue, Nov 24, 2026** | Teensy 200 Hz Firmware, RPi 5 EN/ES Voice Stack & Bench Test Plan | `LOCKED UNTIL G4` |

---

## Detailed Two-Day Stage Gates

### GATE 1 — Procurement Lock & Tooling Selection (Days 1–2: Nov 15–16)
- **Objective:** Lock exact actuator and tooling SKUs so overseas/DDP lead times start ticking and CAD mounting footprints are frozen.
- **Agent Prep Deliverables:**
  - Exact cart breakdown split by **AliExpress** (servos, 25GA-370 motors, GT2 pulleys/belts, Daly 3S BMS, IP2368, BQ24650), **Digi-Key/Mouser DDP** (RPi 5 8GB, Pi Cam 3 Wide + 22-pin CSI cable, Teensy 4.1, BNO085, ReSpeaker 4-Mic, Cytron MDD3A, 5.1V/5.0V bucks, INA226), and **Mexico Domestic** (Molicel P42A 21700 cells, 6061-T6 plates, filaments).
- **Decisions to Confirm by Mon, Nov 16:**
  - [ ] **Decision 1.1 (Leg Servos):** Full `12× Feetech STS3250` (`$744`) **OR** cost-optimized `8× 12V-STS3215` (hips) + `4× STS3250` (knees) (`$476`)?
  - [ ] **Decision 1.2 (PA-CF Printer & Dryer):** Order `QIDI Plus4 ($799)` **OR** `Bambu Lab H2D / H2S` + `Sunlu S4 Dryer ($140)`?
  - [ ] **Decision 1.3 (Wave-1 Order Release):** Confirm placing long-lead actuator & electronics orders.
- **Gate 1 Approval Criterion:** User signs off on Decisions 1.1–1.3 to unlock **Gate 2 (CAD & DXFs)**.

---

### GATE 2 — Master CAD Envelope & Symmetric `< >` X-Leg Geometry (Days 3–4: Nov 17–18)
- **Objective:** Freeze all parametric dimensions in **Autodesk Fusion 360** for the open-frame chassis, roll bars, top cargo/solar rack, and symmetric `< >` legs.
- **Agent Prep Deliverables:**
  - Parametric Fusion 360 dimension sheet & joint table (`L_hip = 48 mm`, `L1 = 105 mm`, `L2 = 105 mm`, `Rw = 32.5 mm`, `100T / 200 mm` GT2 `2:1` knee belt at `C = 75.8 mm`).
  - 2D CNC layout specification for `2.5 mm` bottom `6061-T6` plate and `1.5 mm` top `6061-T6` plate + front/rear roll-bar mounting brackets.
- **Decisions to Confirm by Wed, Nov 18:**
  - [ ] **Decision 2.1 (Chassis & Rack Dimensions):** Confirm `320 × 160 mm` spine footprint and top rack dimensions to match your external insulated food box + `20 W` solar panel lid.
  - [ ] **Decision 2.2 (CNC vs. Print Split):** Confirm which brackets/plates you will machine in-house on your CNC vs. print in `PAHT-CF` / `PA6-CF`.
  - [ ] **Decision 2.3 (Bearing Fit Tolerances):** Confirm press-fit (`6802-2RS`, `6800-2RS`, `686-2RS`) and `M2.5 / M3` brass heat-set insert hole sizes.
- **Gate 2 Approval Criterion:** User approves master CAD geometry to unlock **Gate 3 (Simulation & Single-Leg Prototyping)**.

---

### GATE 3 — Blender/MuJoCo Digital Twin & Single-Leg Prototype Spec (Days 5–6: Nov 19–20)
- **Objective:** Validate symmetric `< >` kinematics, CoG balance, and `6 cm` step-climbing in simulation before committing to full 4-leg manufacturing.
- **Agent Prep Deliverables:**
  - Complete **MuJoCo XML (`robot_x_config.xml`)** + Python inverse-kinematics & `6 cm` curb-climb test script for symmetric `< >` legs (`FL/FR` knees backward $q_3 < 0$, `RL/RR` knees forward $q_3 > 0$).
  - Slicer & annealing checklist for the first `FL` leg (`PA-CF` thigh/shank + `TPU 95A` airless wheel lattice).
- **Decisions to Confirm by Fri, Nov 20:**
  - [ ] **Decision 3.1 (Sim Clearance & Stance):** Confirm standing height (`181 mm`), cruise crouch (`137.5 mm`), and `6 cm` step-climb clearance in MuJoCo.
  - [ ] **Decision 3.2 (Tire Compliance & Tread):** Confirm `65 mm × 24 mm` `TPU 95A` airless lattice wall count and chevron tread pattern.
  - [ ] **Decision 3.3 (4-Leg Print & CNC Go-Ahead):** Authorize machining the `6061-T6` decks and printing all 4 leg assemblies.
- **Gate 3 Approval Criterion:** User approves digital twin & single-leg design to unlock **Gate 4 (Electrical & Safety Architecture)**.

---

### GATE 4 — Electrical Harness, 3S4P Battery & Hardware Interlocks (Days 7–8: Nov 21–22)
- **Objective:** Lock pin-for-pin wiring, fusing, star grounding, and hardware safety cutoffs before soldering or spot-welding.
- **Agent Prep Deliverables:**
  - Complete **Teensy 4.1 Pinout & Wiring Table** (`Serial1` Left 6 servos, `Serial2` Right 6 servos @ `1 Mbps`, 4× quadrature wheel encoders, 2× `Cytron MDD3A`, `BNO085` IMU, `ELRS CRSF` RX, 3× `INA226` I2C monitors).
  - Hardware safety interlock diagram: Latching `NC` Mushroom E-Stop + `LM5060`/Anti-Spark disconnect + `74LVC1G32` drive-away inhibit (`USB-C PD`, `12 V` spark igniter, and `20 W` solar tether lockout).
- **Decisions to Confirm by Sun, Nov 22:**
  - [ ] **Decision 4.1 (Harness & Connector Layout):** Confirm side utility panel layout (E-stop button, `USB-C PD` port, locking `12 V` igniter jack, `XT30/SP13` lid solar plug).
  - [ ] **Decision 4.2 (Battery Pack Assembly):** Confirm `3S4P Molicel P42A` physical layout, `60 A` MIDI main fuse + `ATO` branch fuses, and `Daly 3S` BMS placement.
  - [ ] **Decision 4.3 (Interlock Verification):** Confirm hardware drive-away lockout logic before any actuator power-on.
- **Gate 4 Approval Criterion:** User approves electrical & safety schematics to unlock **Gate 5 (Firmware, Voice Stack & Bench Bring-Up)**.

---

### GATE 5 — Teensy 200 Hz Firmware, RPi 5 Bilingual Voice & Bench Bring-Up Plan (Days 9–10: Nov 23–24)
- **Objective:** Deliver ready-to-flash Teensy 4.1 firmware and Raspberry Pi 5 ROS 2 / bilingual (`EN/ES`) voice packages, plus a step-by-step hardware bring-up procedure.
- **Agent Prep Deliverables:**
  - **Teensy 4.1 Real-Time Firmware Skeleton:** `200 Hz` Dual-UART `SYNC_WRITE`/`SYNC_READ` servo loop, wheel velocity PID, IMU fusion, RC override arbiter, and watchdog damping.
  - **Raspberry Pi 5 Software Package:** `openWakeWord` + `Vosk` (`EN/ES` strict grammar) + `< 45 ms` fast-path emergency `Stop`/`Alto`/`Para` + `Piper TTS` confirmation + USB serial bridge to Teensy.
  - **Staged Bench Commissioning Checklist:** Power-rail smoke test $\rightarrow$ Servo ID & zero-offset calibration $\rightarrow$ Hanging-frame IK test $\rightarrow$ Grounded standing/rolling/step test.
- **Decisions to Confirm by Tue, Nov 24:**
  - [ ] **Decision 5.1 (Voice & RC Command Map):** Confirm final English/Spanish voice phrases and RadioMaster Pocket switch assignments (`Arm`, `Mode`, `E-Stop`, `12V Spark Enable`).
  - [ ] **Decision 5.2 (Calibration & Test Protocol):** Approve bench bring-up sequence to execute as parts arrive.
- **Gate 5 Approval Criterion:** Final sign-off to begin physical assembly and live bench testing.
