# Project 3 — Full-Scale 80 kg Field Logistics & 1 kW Appliance Quadruped (`Jetson AGX Thor`)

**Status:** **Target Production Architecture (Locked for Stage 3 after Project 1 & Project 2 validation)**  
**Role:** Heavy-duty outdoor logistics carrier rated for **`80 kg` payload** (external insulated food container without active cooling, plus field appliances up to a **`1 kW` kitchen blender**), **`≥ 25–30 cm` stair/curb climbing**, **`4+ hours` endurance**, powered by your **NVIDIA Jetson AGX Thor** onboard.

---

## 1. Full-Scale Target Envelope

| Parameter | Stage 3 Target Specification | Notes |
| :--- | :--- | :--- |
| **Empty Chassis + Battery Mass** | `~85–95 kg` | Welded/CNC `6061-T6` / `7075-T6` aluminum spine, carbon tubes, steel/7075 joint housings |
| **Rated Cargo Payload** | **`80 kg`** (`~170–175 kg` loaded gross weight) | Bolted top rack fits large external insulated food container (no cooling) + appliance mount |
| **Step / Stair Climbing** | **`≥ 25–30 cm`** | Symmetric `< >` X-configuration legs (`L_thigh ≈ 420–450 mm`, `L_shin ≈ 420–450 mm`) |
| **Endurance Target** | **`≥ 4 hours`** mixed outdoor logistics + appliance duty | `48 V` (`16S` `LiFePO4`, `~4.5–5.1 kWh`, `~32–38 kg` pack) |
| **Onboard Compute** | **NVIDIA Jetson AGX Thor** *(user already owns)* | Runs vision-language navigation, full terrain-aware RL locomotion, Whisper + multi-billion-parameter local EN/ES LLM, real-time SLAM |
| **Perception** | Stereo RGB-D + optional 3D LiDAR (`Mid-360` / `Ouster` / `RoboSense`) | Protected inside front/rear tubular `6061-T6` / `4130` roll bars |
| **Real-Time Low-Level Control** | Dual CAN-FD (`5 Mbps`) or EtherCAT deterministic controller (`1 kHz`) + dual redundant IMUs | Hard hardware E-stop contactor (`300 A+`) + pre-charge + drive-away interlock |

---

## 2. Structural & Actuation Architecture (Scaled from Project 2)

1. **12× Heavy-Duty Leg Actuators (`120–180+ N·m` Peak):**
   - Frameless high-torque BLDC stators/rotors (e.g., CubeMars RI80/RI100 class, T-Motor AK10-9 / MyActuator RMD-X10/X12, or equivalent) paired with **CNC-machined steel/`7075-T6` dual cycloidal reducers (`15:1`–`25:1`) or strain-wave/planetary gearboxes** (`200–300 W` continuous per joint).
   - **No load-bearing 3D-printed plastics in primary hip/knee torque paths.** Annealed `PAHT-CF` is reserved for rock shields, wire ducts, camera/mic shrouds, and fairings.
2. **4× Hub Wheel Motors (`500–1,000 W` each):**
   - Geared planetary bicycle/cargo-scooter hub motors (`~40–60 N·m` peak torque at `200–250 mm` diameter) with custom molded or thick-wall `TPU 95A` off-road treads + optional mechanical parking brake on rear wheels (eliminates static motor heat while parked on hills with `80 kg` cargo!).
3. **Why Mechanical Wheel Parking Brakes Matter at `170 kg`:**
   - Holding a `170 kg` loaded robot still on a `10°` ramp for 30 minutes while blending food using BLDC holding current wastes battery heat. **12 V/24 V failsafe spring-applied electromagnetic brakes** (or servo mechanical pin locks) on rear wheels give zero-watt hillside parking.

---

## 3. Power, Appliance Inverter (`1 kW` Blender) & Solar

| Subsystem | Stage 3 Hardware Spec | Justification |
| :--- | :--- | :--- |
| **Main Traction Battery** | **`48 V 100 Ah` (`16S` `LiFePO4` prismatic / pack, `5.12 kWh`)** | Safe chemistry under outdoor heat/impact, `2,000–4,000` cycle life, supports `4+ hr` rolling (~450 W avg) + blender bursts (`~800–1,200 W` AC for 2–3 min runs = `< 50 Wh` per batch) |
| **AC Appliance Inverter** | **`Victron Phoenix 48/1200` (1,200 VA / `1,000 W` cont, `2,200 W` surge) + Type B GFCI outlet** | Clean pure-sine AC handles high inductive inrush motor startup of commercial kitchen blenders safely outdoors |
| **DC Auxiliary Outputs** | Dual `100 W` USB-C PD + `12 V / 15 A` momentary burner spark igniter jack | Same EN/ES confirmation + hardware dead-man safety logic verified in Stage 1/2 |
| **Solar Charging** | **`100–200 W`** quick-disconnect monocrystalline panels (lid-mounted + optional fold-out side wings) + Victron SmartSolar MPPT (`75/15` or `100/20`) | Extends camp/field standby and slowly tops off during stationary outdoor prep |
| **Drive-Away Safety Interlock** | Hardware interlock forces **`PARK BRAKE ON` + `LEG BLDCs DISABLED`** whenever AC inverter outlet cover is open, shore charger is connected, or igniter is armed | Required before operating `120 V AC` appliances onboard a mobile `170 kg` machine |

---

## 4. Stage 3 Budget & Readiness Gate

- **Estimated DIY Build Cost:** **`~$11,000–$18,000 USD`** *(excludes Jetson AGX Thor and CNC/printer equipment already owned; actuators + LFP pack + Victron inverter drive ~75% of cost)*.
- **Mandatory Entry Gate from Project 2 (`02_half_scale_jetson_oakd/`):**
  1. Proven zero-failure CAN-FD bus + E-stop + drive-away interlock under outdoors vibration.
  2. Verified CNC cycloidal/HTD knee gearbox endurance across `50+` stair climbs at `10 kg` payload.
  3. Validated Fusion 360 → Isaac Lab/MuJoCo sim-to-real RL transfer for `< >` symmetric X-config legs.
