# Project 2 — Power System, Safety Interlocks & Half-Scale BOM (`robot/02_half_scale_jetson_oakd`)

## 1. Power Architecture (`48 V` Nominal Bus)

Moving to a **`48 V` (`13S` Li-ion or `16S` LFP)** traction bus cuts motor bus current by `4×` relative to `12 V`, avoids thick heavy cabling down the thighs, and feeds the **Jetson Orin Nano Super** cleanly via its native **`9–20 V DC`** input (bypassing the tight `5.1 V / 5 A` USB-C bottleneck of Raspberry Pi 5).

### 1.1 Battery Pack Sizing

- **Target Runtime:** `2–4 hours` mixed rolling + stair-step testing at half scale.
- **Estimated Average Load:**
  - Rolling on gravel (`~1.2 m/s`): `~120–160 W` motors + `~25 W` Orin Nano/OAK-D/Teensy = `~150–185 W`.
  - Dynamic climbing bursts: `~450–900 W` brief peaks (`< 5 s`).
- **Recommended Pack:** **`48 V 20 Ah` (`13S4P` or `13S5P` 21700 Molicel P42A / Samsung 50S, `~960–1,000 Wh`)** or modular pair of **`48 V 10 Ah`** hot-swappable packs (`~4.5–5.5 kg`) with **`80 A` continuous / `150 A` peak smart BMS (`UART/CAN` telemetry to Teensy)**.
  - **Estimated Runtime:** `960 Wh × 0.85 DoD / 180 W ≈ 4.5 hours` continuous mixed cruising!

### 1.2 Voltage Rails & Appliance Demo Panel

| Rail / Output | Converter / Hardware | Rating | Loads |
| :--- | :--- | :--- | :--- |
| **`48 V` Motor Bus** | Direct from BMS via **200 A latching E-stop contactor/FET switch** + `4700 µF` anti-spark pre-charge | `48 V (39–54.6 V)` | 8× Hip BLDCs + 4× Knee BLDCs + 4× Wheel FOCs |
| **`19 V` Compute Rail** | Isolated `48 V → 19 V` DC-DC buck (`10 A / 190 W`) | `19 V @ 5 A+` | **Jetson Orin Nano Super** (25 W Super mode barrel jack) + auxiliary cooling fans |
| **`5 V` Logic / Vision Rail** | Isolated `48 V → 5 V` DC-DC (`6–10 A`) | `5 V @ 8 A` | Teensy 4.1, ELRS RX, IMU, ReSpeaker, OAK-D Lite powered USB-C hub |
| **`12 V` Igniter / Aux Rail** | `48 V → 12 V` DC-DC (`10 A`) + High-side FET (`IRF7424`) + `10 A` fuse | `12 V @ 10 A` | **Momentary push-to-spark stove igniter** (hard-wired hardware dead-man / interlock) |
| **USB-C PD Ports (×2)** | Dual `12–48 V` input buck-boost USB-C PD (`IP2368` / `SW3518` modules) | `Up to 100 W` each | Phones, radios, field tablets |
| **AC Inverter Demo** | **`300 W` compact pure-sine inverter (`48 V DC → 120 V AC`)** + compact GFCI plug | `300 W` cont | Laptop charger, small camp appliances *(tests safety & wiring ahead of Stage 3's 1 kW blender inverter)* |
| **Lid Solar Input** | Quick-disconnect XT60 + **`40–50 W` semi-flexible panel** on container lid + `48 V` boost MPPT (`MPT-7210A` or Genasun boost MPPT) | `~45 W` trickle | Extends standby/outdoor runtime without draining traction pack |

### 1.3 Drive-Away Hardware Interlock

Keep the hardware AND-gate interlock from Project 1:
- **Shore charge sense (`CHG_SENSE`)**, **AC inverter active (`AC_EN`)**, and **12 V igniter trigger (`IGN_ARM`)** feed a hardware interlock line (`74LVC1G32` / relay coil) that opens the main `48 V` motor enable loop so the robot **cannot drive away while plugged in or sparking a burner**.

---

## 2. Project 2 Estimated BOM (`$3,500–$4,800 USD`)

*(All prices are planning estimates in USD excluding shipping/Mexican import IVA unless noted; markups vary ~25–35% when importing electronics directly from China vs US/local MX distributors.)*

| Category | Item | Qty | Est. Unit | Est. Total | Sourcing Notes (Mexico) |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Compute & AI** | NVIDIA Jetson Orin Nano Super Dev Kit (8 GB) | 1 | $249 | $249 | NVIDIA / Digi-Key / Arrow DDP |
| **Vision** | Luxonis OAK-D Lite (Auto-Focus or Fixed-Focus — prefer **Fixed-Focus** for robot vibration!) | 1 | $149 | $149 | Luxonis / Digi-Key (avoid AF lens chatter!) |
| **Audio / HRI** | ReSpeaker 4-Mic Array v2.0 + windscreen + elastomer mounts | 1 | $75 | $75 | Seeed / Digi-Key |
| **MCU & Sensors** | Teensy 4.1 + ICM-42688-P IMU + 2× CAN-FD transceivers (`MCP2562FD`) | 1 set | $85 | $85 | PJRC / Digi-Key / Mouser |
| **RC Override** | RadioMaster Pocket / Boxer ELRS + RP3 diversity RX | 1 | $95 | $95 | Local RC / AliExpress |
| **Hip Actuators** | 8× 8108-class Planetary CAN BLDC (e.g., GIM8108-9 / RMD / AK-class) | 8 | $145 | $1,160 | Factory-direct / AliExpress (factor MX duty) |
| **Knee Actuators** | 4× 8108/6010 BLDC + 4× CAN-FD FOC ESC + HTD-5M pulleys/belts & bearings | 4 sets | $165 | $660 | CNC pulleys/housings in-house; bearings local MX |
| **Wheel Drives** | 4× 6010 high-torque BLDC + 4× compact CAN FOC ESC | 4 sets | $95 | $380 | Same supplier batch to reduce freight |
| **Structure** | CNC `6061-T6` aluminum plates + CF tubes (`20×20` & `22 mm`) + roll-bar tubing | Lot | $260 | $260 | Metalium / Inox Victoria / Sumetal (MX local) + online CF |
| **3D Printing Filament** | `PAHT-CF` / `PA12-CF` (`3 kg`) + `TPU 95A` (`1 kg`) + `7075`/steel pins/fasteners | Lot | $240 | $240 | Bambu / QIDI / local fastener supply (`M3–M8 12.9`) |
| **48 V Battery + BMS** | ~52× Molicel P42A/50S (`13S4P`) + smart `80 A` CAN/UART BMS + nickel + XT90 + charger | 1 pack | $420 | $420 | **Buy 21700 cells inside Mexico** (avoid IATA air ban) |
| **Power & Safety Panel** | E-stop contactor, `19 V/12 V/5 V` DC-DCs, `100 W` PD, `300 W` 48 V inverter demo, `50 W` solar + boost MPPT, fuse block | Lot | $320 | $320 | Amazon MX / MercadoLibre / Digi-Key |
| **Spares & Misc** | Silicone wire (`10–18 AWG`), XT60/XT30/GH1.25 crimps, extra belts, insulated cargo box | Lot | $180 | $180 | Local electronics / MercadoLibre |
| **Estimated Subtotal** | | | | **~$4,273** | **Fits comfortably inside `$3,500–$4,800` target band** *(excludes 3D printer capital cost budgeted separately)* |

> [!TIP]
> **Always pick Fixed-Focus (`FF`) over Auto-Focus on OAK-D Lite** for mobile legged robots. VCM autofocus coils blur violently under footsteps and high-frequency motor vibration.
