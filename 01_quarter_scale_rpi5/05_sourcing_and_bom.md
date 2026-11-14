# 05 - Comprehensive Bill of Materials (BOM), Sourcing & Procurement Guide (Stage 1)

**Deliverable Reference:** `/Users/mac/Documents/Agents Antigravity/research/05_sourcing_and_bom.md`  
**Author:** Research Agent 5 (Sourcing, BOM & Supplier Guide Specialist)  
**Target Platform:** Dynamic Wheel-Legged Robot (Stage 1 Prototype)  
**Budget Window:** $1,200.00 – $1,700.00 USD  
**Geographic Operations:** Mexico (UTC-6) / North America / Global Direct Procurement  

---

## Executive Summary & Procurement Philosophy

Building a dynamic wheel-legged robot requires a disciplined balance between low unsprung inertia, high peak-torque density, robust real-time compute, and rapid mechanical iteration. 

Stage 1 is designed as a **deterministic, ground-up validation platform**:
1. **Kinematics & Balance Control:** Evaluates Linear Quadratic Regulator (LQR) and Model Predictive Control (MPC) balance algorithms using high-speed IMU feedback.
2. **Hybrid Actuation:** Couples high-torque serial bus smart servos (Feetech STS series) for hip/knee 2-DOF linkage geometry with high-efficiency motors driving lightweight wheels via GT2 timing belts to keep heavy motors located near the hip/chassis pivot.
3. **Dual-Tier Compute:** Distributes hard real-time balance loops (1 kHz) to an ARM Cortex-M7 microcontroller (PJRC Teensy 4.1) while offloading vision, state estimation, ROS 2, and telemetry to a single-board computer (Raspberry Pi 5 8GB).
4. **Resilient Local Sourcing:** Optimizes procurement for an engineering team based in Mexico—mitigating high courier brokerage fees, navigating SAT cross-border e-commerce tax regulations (33.5% flat duty/IVA rules on non-FTA parcel imports), leveraging DDP (Delivered Duty Paid) shipping via Mouser/Digi-Key, and sourcing raw 6061-T6 aluminum plates from domestic metal service centers in CDMX, Guadalajara, and Monterrey.

---

## 1. Tooling & 3D Printing Procurement

Additive manufacturing represents 60% to 70% of Stage 1 structural chassis components, sensor enclosures, wheel rims, and bearing housings. Structural robot links experience severe shock impacts, high vibrational loads, and cyclic bending stresses during balance recovery and drop tests; standard PLA or unreinforced PETG are strictly unacceptable. Tooling must support abrasive, high-temperature composite filaments: **Carbon Fiber Reinforced Nylon (PA-CF)**, **UV/weather-resistant ASA**, and **vibration-damping TPU 95A**.

### 1.1 Printer Model Comparison: Bambu Lab H2D vs. Bambu Lab H2S vs. QIDI Plus4

| Feature / Metric | Bambu Lab H2D (Combo / Pro) | Bambu Lab H2S | QIDI Plus4 |
| :--- | :--- | :--- | :--- |
| **Motion Architecture** | CoreXY (Dual Toolhead / IDEX) | CoreXY (Single Toolhead) | CoreXY (Single Toolhead) |
| **Build Volume (X×Y×Z)** | $350 \times 320 \times 325 \text{ mm}^3$ | $340 \times 320 \times 340 \text{ mm}^3$ | $305 \times 305 \times 280 \text{ mm}^3$ |
| **Chamber Heating** | **Active Heating up to 65 °C** | **Active Heating up to 65 °C** | **Active 400W PTC Heater up to 65 °C** |
| **Max Hotend Temp** | 350 °C (Dual independent hotends) | 350 °C (Hardened steel/ceramic) | 370 °C (Bimetal / Hardened steel 80W) |
| **Max Bed Temp** | 120 °C | 120 °C | 120 °C |
| **Extruder System** | Dual Closed-Loop Servo Extruders | Closed-Loop Servo Extruder | High-Torque Dual-Gear Direct Drive |
| **Firmware Ecosystem** | Proprietary Bambu OS (Closed, AI vision) | Proprietary Bambu OS (Closed, AI vision) | **Open-source Klipper** (Fluidd/Mainsail web UI) |
| **Multi-Material Capability**| **Native Dual-Material / IDEX** (No waste purges; supports soluble PVA/HIPS + PA-CF) | Single nozzle (AMS multi-color/purge tower) | Single nozzle (Single material per job) |
| **Auxiliary Modules** | Optional 10W/40W Laser cutter & vinyl blade | Optional 10W Laser cutter module | Dedicated 3D printing only |
| **Price (USD)** | **$1,850 – $2,450 USD** | **$1,250 – $1,450 USD** | **$799 USD** |
| **Price (MXN, Approx.)** | $38,000 – $48,000 MXN + IVA | $25,000 – $30,000 MXN + IVA | $16,500 – $18,000 MXN (Imported / Amazon MX) |
| **Authorized MX Vendors** | 3D Market Mexico, México Makers | 3D Market Mexico, México Makers | QIDI Global Direct, Amazon MX, Mercado Libre |

#### Recommendations:
1. **The QIDI Plus4 ($799 USD) is the Primary Engineering Recommendation for Stage 1:**
   - Active 65 °C heated chamber, 370 °C hotend, and hardened nozzle out of the box at under half the cost of the Bambu H2D.
   - Native Klipper provides direct access to custom acceleration tuning, pressure advance, and macros without cloud lock-in.
2. **The Bambu Lab H2D ($1,850+ USD) is the Premium Multi-Material Solution:**
   - Independent dual-extrusion (IDEX) allows printing complex geometries with soluble supports or co-molding rigid PA-CF chassis hubs with integrated flexible TPU bumpers.

---

### 1.2 Tooling Accessories: Dryers, Nozzles & Build Plates

1. **Filament Dryers (Mandatory for PA-CF):**
   - **Sunlu S4 Filament Dryer (4-Spool Capacity, 350W PTC, Up to 70 °C):** ~$140 USD / $2,800 MXN. Polyamide (Nylon) absorbs moisture rapidly, leading to micro-voids and degraded tensile strength. S4 keeps spools at $\le 15\%$ RH and feeds directly into the printer during multi-hour prints.
2. **Hardened Nozzles:**
   - Standardize on **0.6 mm orifice diameter** (Hardened steel or Tungsten Carbide, $15–$30 USD). Prevents chopped fiber clogging, improves layer weld strength, and reduces total printing time by 35%.
3. **Build Plates & Adhesion Interfaces:**
   - Double-Sided Textured PEI Spring Steel Sheet ($25–$35 USD) + Magigoo PA Adhesive Pen (~$24 USD / $480 MXN).

---

## 2. Stage 1 Detailed Itemized BOM Table

### 2.1 Actuation & Transmission (12 Leg Servos + 4 Wheel Drives)
| Category | Component Name | Part Number / Model | Vendor / Platform | Unit Price (USD) | Qty | Total Cost (USD) | Lead Time | Purchase Search Query |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- |
| Actuation | High-Torque Smart Bus Servo (Full 12-DOF Baseline) | **Feetech STS3250** (12V, 50 kg·cm / 4.9 N·m, Mag Enc)* | AliExpress Feetech / Factory | $62.00 | 12 | $744.00 | 18–25 days | `Feetech STS3250 12V 50kg.cm servo` |
| Actuation | CNC Aluminum Servo Horns & Dual Brackets | Feetech Standard Servo Horn & Bracket Kit | AliExpress / Amazon MX | $6.50 | 12 | $78.00 | 15–20 days | `Feetech STS aluminum bracket horn kit` |
| Actuation | Brushed Geared Wheel Drive Motors + Encoders | **25GA-370 12V 300RPM** w/ Hall Quadrature Encoder | AliExpress / Amazon MX | $17.00 | 4 | $68.00 | 15–22 days | `25GA-370 gear motor encoder 12V 300rpm` |
| Electronics | Dual Wheel H-Bridge Driver Boards | **Cytron MDD3A** Dual DC Motor Driver (or DRV8874) | Mouser / Cytron / AliExpress | $18.00 | 2 | $36.00 | 4–18 days | `Cytron MDD3A dual DC motor driver` |
| Electronics | Half-Duplex UART Servo Bus Adapters (Bus A & B) | **74LVC2G241** Buffer / Waveshare Serial Bus Adapter | Digi-Key / AliExpress | $8.00 | 2 | $16.00 | 4–15 days | `Waveshare serial bus servo driver board` |
| Transmission | GT2 Motor Timing Pulleys (Driver) | GT2 16T, 5 mm Bore, 9/10 mm Belt Width (Alum) | Amazon MX / AliExpress | $3.50 | 4 | $14.00 | 3–15 days | `GT2 pulley 16T 5mm bore 10mm width` |
| Transmission | GT2 Driven Pulleys (Knee Joint) | GT2 32T, 8 mm Bore, 9/10 mm Belt Width (Alum) | Amazon MX / AliExpress | $7.00 | 4 | $28.00 | 3–15 days | `GT2 pulley 32T 8mm bore 10mm belt` |
| Transmission | GT2 Reinforced Timing Belts | GT2 Closed Loop Belt **100T / 200 mm** (6–9 mm width) | Amazon MX / AliExpress | $3.50 | 4 | $14.00 | 3–15 days | `GT2 timing belt closed loop 200mm` |
| Transmission | Wheel Couplers & Fastening Hardware | Brass Hex Couplers (for printed PA-CF hub + TPU 95A tire) | In-House Print / Amazon MX | $6.00 | 4 | $24.00 | 2–5 days | `robot wheel hex coupling 4mm 6mm` |
| **Subtotal** | **Actuation & Transmission** | | | | | **$1,022.00** | | |

*\*Budget Optimization Note: If targeting ~\$1,700 total, replace the 8 hip servos with **12V-rated Feetech STS3215** (`$28.50 × 8 = $228.00`) and keep `4× STS3250` (`$248.00`) on the knees, saving **`$268.00`** (`$754.00` Actuation Subtotal).*

### 2.2 Compute, Sensors & Radio
| Category | Component Name | Part Number / Model | Vendor / Platform | Unit Price (USD) | Qty | Total Cost (USD) | Lead Time | Purchase Search Query |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- |
| Compute | High-Level SBC (Vision, Navigation, ROS 2) | **Raspberry Pi 5 (8GB RAM)** | Digi-Key / Mouser / Amazon MX | $80.00 | 1 | $80.00 | 3–5 days | `Raspberry Pi 5 8GB SC1112` |
| Compute | Pi 5 Active Heatsink Cooler | Official Raspberry Pi 5 Active Cooler | Digi-Key / Mouser / Amazon MX | $7.50 | 1 | $7.50 | 3–5 days | `Raspberry Pi 5 Active Cooler SC1148` |
| Compute | Pi 5 CSI Ribbon Adapter Cable | **22-pin (0.5mm) to 15-pin (1mm)** CSI Cable (200–300 mm) | Digi-Key / Amazon MX | $5.00 | 1 | $5.00 | 2–5 days | `Raspberry Pi 5 camera cable 22 pin to 15 pin` |
| Compute | High-Endurance MicroSD Storage | SanDisk Extreme Pro 128GB MicroSDXC A2 V30 | Amazon MX | $18.00 | 1 | $18.00 | 1–2 days | `SanDisk Extreme Pro 128GB microSDXC` |
| Compute | Real-Time Microcontroller (200–1000 Hz Loop)| **PJRC Teensy 4.1** (ARM Cortex-M7 600MHz) | PJRC Direct / Digi-Key / Mouser | $36.75 | 1 | $36.75 | 3–5 days | `PJRC Teensy 4.1 DEV-16771` |
| Sensors | High-Precision 9-DOF IMU | **Adafruit BNO085** VR IMU Breakout (or ICM-42688-P) | Digi-Key / Mouser / Adafruit | $32.50 | 1 | $32.50 | 3–5 days | `Adafruit BNO085 4754` |
| Sensors | Low-Latency Vision Camera | **Raspberry Pi Camera Module 3 Wide** (Sony IMX708) | Digi-Key / Mouser / Amazon MX | $35.00 | 1 | $35.00 | 3–5 days | `Raspberry Pi Camera Module 3 Wide` |
| Sensors | Far-Field Voice & Audio DSP Array | **Seeed Studio ReSpeaker USB 4-Mic Array v2.0** | Digi-Key / Seeed Studio | $44.00 | 1 | $44.00 | 4–10 days | `Seeed Studio ReSpeaker USB 4-Mic Array` |
| Radio | ELRS 2.4GHz Handheld Transmitter | **RadioMaster Pocket ELRS 2.4GHz** (M2, Hall Gimbals) | Amazon MX / AliExpress | $64.99 | 1 | $64.99 | 4–18 days | `RadioMaster Pocket ELRS transmitter` |
| Radio | Ultra-Low-Latency Nano RC Receiver | **RadioMaster RP1 / RP3 Nano** 2.4GHz ELRS Receiver | Amazon MX / AliExpress | $16.50 | 1 | $16.50 | 4–18 days | `RadioMaster RP1 ExpressLRS receiver` |
| **Subtotal** | **Compute, Sensors & Radio** | | | | | **$340.24** | | |

### 2.3 Power, Battery & Auxiliary Boards
| Category | Component Name | Part Number / Model | Vendor / Platform | Unit Price (USD) | Qty | Total Cost (USD) | Lead Time | Purchase Search Query |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- |
| Power | High-Drain 21700 Li-ion Cells (3S4P Pack) | **Molicel INR21700-P42A** (4200mAh, 45A) | Domestic MX Battery Vendor | $12.50 | 12 | $150.00 | 2–5 days | `Molicel 21700 P42A original Mexico` |
| Power | Smart Battery Management System (3S) | **Daly / JBD Smart BMS 3S** 12V 40A–60A (UART / BT) | AliExpress Daly Official Store | $34.00 | 1 | $34.00 | 18–25 days | `Daly Smart BMS 3S 12V bluetooth` |
| Power | 3S Mains Battery Charger | 12.6V 5A Li-ion Charger (XT60 plug) | Amazon MX / MercadoLibre | $22.00 | 1 | $22.00 | 2–4 days | `12.6V 5A 3S li-ion charger XT60` |
| Power | 100W Bidirectional USB-C PD Board | **IP2368** 100W 2-Way Fast Charge Buck-Boost Module | AliExpress / Amazon MX | $22.00 | 1 | $22.00 | 12–20 days | `IP2368 100W PD buck boost module` |
| Power | Semi-Flexible Monocrystalline Solar Panel | 20W ETFE Semi-Flexible 18V Panel (lid-mounted) | Amazon MX / AliExpress | $45.00 | 1 | $45.00 | 4–15 days | `20W flexible solar panel ETFE 18V` |
| Power | Synchronous MPPT Solar Charger Board | **BQ24650 / CN3795** 3S MPPT Solar Charge Module | AliExpress / Amazon MX | $24.00 | 1 | $24.00 | 12–20 days | `BQ24650 MPPT solar charge module` |
| Power | Primary 5.1V Buck Converter (Pi 5) | 12V to **5.1V 5A** Synchronous Buck (**TPS56C215** class) | Digi-Key / Amazon MX | $14.50 | 1 | $14.50 | 2–5 days | `12V to 5V 5A step down buck converter` |
| Power | Secondary 5.0V Logic Buck (Teensy/Sensors) | 12V to **5.0V 3A** Buck (**TPS54302** class) | Digi-Key / Amazon MX | $8.00 | 1 | $8.00 | 2–5 days | `TPS54302 5V 3A buck module` |
| Telemetry | I2C Current & Voltage Sensors | **TI INA226** 16-bit I2C Power Monitor Breakouts | Amazon MX / Digi-Key | $4.50 | 3 | $13.50 | 2–5 days | `INA226 I2C current sensor module` |
| Power | Switched 12V Igniter Driver + Jack | High-side FET module (**IRF7424**/Opto) + locking 5.5×2.1 mm jack | Digi-Key / Amazon MX | $10.00 | 1 | $10.00 | 2–5 days | `opto isolated high side mosfet switch 12V` |
| Safety | Emergency Stop Pushbutton Switch (E-Stop) | 16/19 mm IP65 Latching E-Stop (Mushroom Head, NC) | Amazon MX / Mouser | $11.50 | 1 | $11.50 | 2–5 days | `16mm emergency stop latching switch` |
| Safety | Solid-State Power Disconnect & Interlock | Flipsky Anti-Spark Switch Pro (or **LM5060** stage + `74LVC1G32`) | AliExpress / Amazon MX | $39.00 | 1 | $39.00 | 10–18 days | `Flipsky anti spark switch pro` |
| Power | Wiring, Connectors & Fuse Distribution Kit | AMASS XT90-S / XT60H / XT30U + Silicone Wire + 60A MIDI & ATO Fuses | Amazon MX | $30.00 | 1 | $30.00 | 1–2 days | `Amass XT60 XT90 connector kit silicone wire` |
| **Subtotal** | **Power, Battery & Auxiliary Boards** | | | | | **$423.50** | | |

### 2.4 Mechanical, Hardware, Bearings & Filaments
| Category | Component Name | Part Number / Model | Vendor / Platform | Unit Price (USD) | Qty | Total Cost (USD) | Lead Time | Purchase Search Query |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- |
| Mechanical | CNC 6061-T6 Aluminum Plates & Roll Bars | 2.5 mm bottom deck, 1.5 mm top deck, 12–16 mm roll-bar tube | Metalium MX / In-House CNC | $135.00 | 1 | $135.00 | 5–8 days | Local 6061-T6 plate & tube stock |
| Mechanical | Sealed Deep Groove Ball Bearings (32 pcs total) | 8× `6802-2RS`, 16× `6800-2RS`, 8× `686-2RS` | Amazon MX / AliExpress | $28.00 | 1 | $28.00 | 2–15 days | `6802-2RS 6800-2RS 686-2RS bearings` |
| Hardware | Knurled Brass Threaded Heat-Set Inserts | M2.5 / M3 / M4 Assortment Kit | Amazon MX | $18.50 | 1 | $18.50 | 1–2 days | `brass heat set inserts M2.5 M3 M4 kit` |
| Hardware | High-Tensile Socket Head Screws Assortment | Class 12.9 Alloy Steel M2.5 / M3 / M4 | Amazon MX / McMaster-Carr | $28.00 | 1 | $28.00 | 1–2 days | `M2.5 M3 M4 socket head cap screw kit 12.9` |
| Hardware | Aluminum & Brass Hex Standoffs | M2.5 & M3 Hex Standoff Kit | Amazon MX | $15.50 | 1 | $15.50 | 1–2 days | `M2.5 M3 hex standoff brass aluminum kit` |
| Mechanical | Carbon Fiber Structural Tubes | 16 mm / 12 mm OD 3K Carbon Fiber Roll-Wrapped Tube | AliExpress / Amazon MX | $24.00 | 1 | $24.00 | 4–15 days | `16mm carbon fiber tube 3K roll wrapped` |
| Printing | Structural & Tire Filaments | 2× 1 kg Bambu `PAHT-CF` / `PA6-CF` + 1× 1 kg `TPU 95A` | Bambu / Polymaker / Amazon MX | $115.00 | 1 lot | $115.00 | 2–5 days | `Polymaker PA6-CF TPU 95A` |
| **Subtotal** | **Mechanical, Hardware & Filaments** | | | | | **$364.00** | | |

### 2.5 Grand Total Prototype Cost Summary (Reconciled Full 16-DOF)
```
┌─────────────────────────────────────────────────────────────┬─────────────────┐
│ Category                                                    │ Total Cost (USD)│
├─────────────────────────────────────────────────────────────┼─────────────────┤
│ 1. Actuation & Transmission (12× STS3250 + 4× 25GA-370)     │   $1,022.00     │
│    (Budget Alt w/ 8× 12V STS3215 hips + 4× STS3250 knees)   │    ($754.00)    │
│ 2. Compute, Sensors & Radio (RPi 5 + Teensy 4.1 + ELRS)     │     $340.24     │
│ 3. Power, Battery & Auxiliary Boards (3S4P + Solar + PD)    │     $423.50     │
│ 4. Mechanical, Hardware, Bearings & Filaments               │     $364.00     │
├─────────────────────────────────────────────────────────────┼─────────────────┤
│ COMPONENT HARDWARE TOTAL (All-STS3250 / Hybrid Servo Alt)   │$2,149.74 / $1,881.74│
│ Plus Shipping / SAT Import Duty Allowance (~10–15% blended) │ ~$200 – $260    │
└─────────────────────────────────────────────────────────────┴─────────────────┘
*(Excludes PA-CF 3D printer & dryer capital equipment: ~$940 for QIDI Plus4 + Sunlu S4.)*
```

---

## 3. Mexico / North America Sourcing Strategy

### 3.1 Handling Mexican Import Taxes & Logistics Channels
1. **Digi-Key & Mouser (Silicon, Microcontrollers & ICs):**
   - **Always select DDP (Delivered Duty Paid)** at checkout.
   - Pre-pays 16% Mexican IVA. Zero unexpected broker fees upon arrival via UPS/FedEx in 3–5 days.
   - USMCA (T-MEC) allows 0% duty on North American origin goods.
2. **AliExpress & Asian Suppliers (Servos, BMS, Pulleys):**
   - Under current SAT e-commerce rules, ~33.5% flat duty/IVA is collected directly at checkout for Choice/Standard shipping.
   - Avoid DHL Express on sub-$60 orders to prevent flat customs handling fees ($500–$800 MXN).
3. **Domestic Battery Procurement (IATA Hazmat Rules):**
   - Do NOT air-freight raw lithium-ion cells internationally.
   - Source genuine Molicel INR21700-P42A cells domestically through established Mexican distributors (e.g., *All 4 Vaping MX*, *TheMflight MX*) arriving in 2–3 days via Estafeta/DHL ground.

### 3.2 Domestic Mexican Metal Suppliers for 6061-T6 Aluminum Plate
- **Metalium** (Monterrey, national shipping): `metalium.mx`
- **Inox Victoria** (CDMX, Vallejo, Guadalajara): `inoxidablesvictoria.com`
- **Sumetal** (Tultitlán, Estado de México): `smtultitlan.com`
- **Metales MarChina / Metmar** (Zapopan, Guadalajara): `metmar.com.mx`
- *Machining Tip:* For complex 2D plate profiles, supply DXF files to local waterjet cutting services in CDMX or Guadalajara ($15–$25 USD/meter cut path).

---

## 4. Phased Purchasing Schedule (Week 1 vs. Week 3)

- **Week 1 (Long-Lead Overseas Orders & Tooling, ~$850 USD):** Place AliExpress orders (Feetech servos, Daly BMS, pulleys, IP2368); order Digi-Key parts (Teensy 4.1, IMU BNO085) via DDP; acquire 3D printer (QIDI Plus4) and filament dryer.
- **Week 2 (Domestic Metals & Additive Fabrication, ~$250 USD):** Source 6061-T6 plate drops; send DXF files for waterjet cutting; print skeleton in PA-CF, enclosures in ASA, and tires in TPU 95A.
- **Week 3 (Compute, Power Cells & Final Integration, ~$470 USD):** Acquire Raspberry Pi 5, Pi Cam 3, ReSpeaker array, and RadioMaster Pocket; buy Molicel 21700 cells domestically and spot-weld battery pack.
