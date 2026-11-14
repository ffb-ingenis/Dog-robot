# Project 2 — Mechanical Sizing & Hybrid Actuation (`robot/02_half_scale_jetson_oakd`)

## 1. Design Targets & Mechanical Envelope

Project 2 scales the platform to **half-scale outdoor utility (~20 kg empty, 10 kg payload, 30 kg total loaded mass)** to test dynamic BLDC torque control, real reinforcement-learning terrain traversal, and half-scale CNC aluminum + annealed `PA-CF` structural fabrication before committing to full-scale 80 kg payload hardware.

| Parameter | Target Value | Notes |
| :--- | :--- | :--- |
| **Dry Robot Mass** | `~20 kg` | Aluminum 6061-T6 + CF tube + PA-CF + 48 V battery (~6 kg) |
| **Rated Payload** | `10 kg` | External insulated container or 300 W inverter test load |
| **Total Loaded Mass ($M_{tot}$)** | `30 kg` (`294.3 N` static weight) | Design load case uses `2.0g` dynamic impact factor |
| **Chassis Footprint** | `~580 mm × 340 mm × 160 mm` | Fits bolted top rack (`400 × 300 mm`) + removable insulated box |
| **Nominal Standing Height** | `400 mm` | `L_thigh = 230 mm`, `L_shin = 240 mm` |
| **Step / Curb Climbing Target** | `12–15 cm` | Symmetric `< >` X-configuration legs |
| **Max Wheel Speed (Flat)** | `2.0–2.5 m/s` (`~7–9 km/h`) | Gear/wheel KV sized for hill torque over top speed |
| **Wheel Diameter** | `120–130 mm` (`TPU 95A` airless tread) | Bolted onto hub/geared wheel motor |

---

## 2. Static & Dynamic Torque Sizing ($M_{tot} = 30\text{ kg}$)

Assume $L_2 = 0.23\text{ m}$, $L_3 = 0.24\text{ m}$, maximum knee moment arm $r_{\max} = 0.21\text{ m}$ at deep squat or step lift, and tripod support (1 swing leg + 3 stance legs taking load, with worst-case single corner share = `0.45 × W`):

$$F_{z,\text{corner}} = 0.45 \times 30\text{ kg} \times 9.81\text{ m/s}^2 \approx 132.4\text{ N}$$

- **Static Knee Torque (Normal Squat):**
  $$\tau_{\text{knee,static}} = 132.4\text{ N} \times 0.21\text{ m} \approx 27.8\text{ N}\cdot\text{m}$$
- **Dynamic Climbing / Impact Knee Torque (`2.0×` factor):**
  $$\tau_{\text{knee,peak}} \approx 55.6\text{ N}\cdot\text{m}$$
- **Static Hip-Pitch Torque (`1.5×` stance share):**
  $$\tau_{\text{hipP,static}} \approx 15\text{–}22\text{ N}\cdot\text{m},\quad \tau_{\text{hipP,peak}} \approx 35\text{–}45\text{ N}\cdot\text{m}$$
- **Hip Roll (Ab/Adduction):**
  $$\tau_{\text{hipR,static}} \approx 12\text{–}18\text{ N}\cdot\text{m},\quad \tau_{\text{hipR,peak}} \approx 30\text{ N}\cdot\text{m}$$
- **Wheel Drive Torque (`r_wheel = 0.0625 m`, `15°` gravel grade + `1.5×` acceleration):**
  - Grade force: $F_{grade} = 294.3 \sin(15^\circ) \approx 76.2\text{ N}$ total, plus rolling resistance (~0.06) $\approx 17.6\text{ N}$ $\Rightarrow$ `~94 N` total (`~23.5 N` nominal / `40 N` worst-case single wheel).
  - Torque per wheel: $40\text{ N} \times 0.0625\text{ m} = 2.5\text{ N}\cdot\text{m}$ continuous, **`6–10 N·m` peak** for curb nudge / low-speed hold.

---

## 3. Hybrid Actuation Strategy (Buy Hips, DIY CNC/PA-CF Knees)

To keep total Project 2 cost inside **`$3,500–$4,800 USD`** while exploiting your **CNC mill + PA-CF printer**, actuation uses a **hybrid buy/build architecture**:

### 3.1 Hips (8× Purchased Integrated Planetary BLDCs)

Buy 8 sealed off-the-shelf planetary quasi-direct-drive (QDD) actuators (e.g., **8108 / 8112-class 6:1–9:1 planetary CAN-FD actuators** such as CubeMars AK70-10 / AK80-9 clones, MyActuator RMD-X6/X8, or SteadyWin GIM8108-9):

| Metric | Target Spec per Hip Actuator | Rationale |
| :--- | :--- | :--- |
| **Gear Ratio** | `8:1` to `10:1` single-stage planetary | Low reflected inertia (`I_ref ∝ N^2`), backdrivable for RL compliance |
| **Continuous / Peak Torque** | `10–15 N·m` cont / `25–35 N·m` peak | Covers `hip_roll` and `hip_pitch` without custom gearbox tolerance headaches |
| **Voltage / Bus** | `24–48 V` nominal, **CAN-FD / CAN 2.0B (1–5 Mbps)** | Standard MIT-mode (`p, v, kp, kd, tau_ff`) telemetry feedback |
| **Mass** | `~450–580 g` each | Keeps hip inertia tucked close to torso |

### 3.2 Knees (4× DIY Co-Axial Belt or CNC Cycloidal Drives)

Co-mount a raw **`8108` or `6010` frameless/pancake BLDC + ODrive/VESC-class CAN-FD FOC driver (e.g., Moteus / ODrive Pro / Mini-FOC)** directly at the **hip pitch axis**, driving the knee via a **2.5:1–3:1 high-torque HTD-5M belt (`15 mm` width) or CNC 7075/steel-pin cycloidal stage**:

1. **Co-Axial Belt Option (Recommended First Build):**
   - Motor sits inside hip pitch block (`0 g` added distal mass on thigh/knee!).
   - Motor has internal `6:1` planetary stage (`~15–20 N·m` peak) + **`2.5:1` HTD-5M (`15 mm` kevlar/fiberglass belt)** down the thigh channel $\Rightarrow$ **`37.5–50 N·m` peak at the knee joint**, backdrivable, quiet, and easy to CNC tensioners in `6061-T6`.
2. **CNC Cycloidal Option (Upgrade Path):**
   - CNC-milled `7075-T6` or case-hardened steel eccentric + dual cycloidal discs (`11:1`–`15:1`) driven by bare `5010`/`6010` motor directly at knee for zero-backlash stair climbing practice ahead of Stage 3.
   - **Rule:** Use metal bearings, hardened dowel pins (`HRC 60`), and `6061/7075` housing rings—never rely solely on `PA-CF` teeth for cycloidal lobes.

### 3.3 Wheels (4× `6010`-Class / Geared Hub BLDCs)

- **4× `6010` or `5010` geared/direct-drive wheel motors** (`~4:1–6:1` planetary or `150–200 KV` high-pole-count outrunner inside custom CNC aluminum / `PA-CF` wheel hub with `TPU 95A` airless lattice tire).
- Dedicated compact FOC ESC (`CAN 2.0B / CAN-FD` velocity + current mode) mounted cleanly inside the upper thigh pocket or torso power bay with silicone 16 AWG motor phase leads routed through the shin tube.

---

## 4. Structure & Autodesk Fusion 360 / Blender Workflow

- **Chassis Spine & Bulkheads:** CNC `6061-T6` upper and lower deck plates (`4 mm` thickness, weight-relieved pockets) linked by `20 mm × 20 mm` carbon-fiber square tubes and CNC aluminum corner blocks.
- **Roll Bars:** `16 mm OD × 2 mm` `6061-T6` tube bent/welded or bolted into CNC corner clamps front & rear; provides protected camera bar and handle bars for two-person lift.
- **Thigh & Shin Links:**
  - **Thigh:** Dual `2.5 mm` CF cheek plates + annealed `PAHT-CF` structural spacer core enclosing the HTD-5M knee belt and phase wire channel.
  - **Shin:** `22 mm` CF round tube (`2 mm` wall) bonded/clamped into CNC `6061-T6` knee clevis and wheel fork.
- **Simulation Export (Fusion 360 → Blender → MuJoCo / Isaac Lab):**
  - Export rigid assemblies as STEP/OBJ; decimate visual meshes and run V-HACD convex decomposition in Blender (`<= 16` hulls per body) so the RL trainer on your **Jetson AGX Thor** runs at high parallel throughput.

---

## 5. Dual CAN-FD Bus Topology (`Teensy 4.1`)

Teensy 4.1 provides **3× native FlexCAN controllers (2× CAN-FD + 1× CAN 2.0B)**:

- **Bus A (`CAN1`, Left Side — FL + RL):** 6× leg joints (FL hipR, FL hipP, FL knee, RL hipR, RL hipP, RL knee) + 2× wheel FOCs (`8` nodes @ `5 Mbps` CAN-FD or `1 Mbps` CAN 2.0B).
- **Bus B (`CAN2`, Right Side — FR + RR):** 6× leg joints + 2× wheel FOCs (`8` nodes).
- **Host Link:** Teensy 4.1 ↔ **Jetson Orin Nano Super** over **USB-HS (CDC / `micro_ros_agent` or raw COBS binary frames @ `500 Hz`)** + hardware **BNO085 / ICM-42688-P IMU** read via SPI on Teensy at `1 kHz`.
- **Fallback Safety:** If Jetson crashes or misses `> 20 ms` heartbeat (`10` frames at `500 Hz`), Teensy drops seamlessly into **damped stand** (`kp=0, kd=safe_damping` or zero velocity hold) and accepts **ELRS RC override** directly.
