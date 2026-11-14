# 04 - Electronics, Power Distribution & Energy Export Architecture

## Executive Summary
This document specifies the electrical architecture, power distribution network (PDN), battery energy storage system (BESS), auxiliary power export subsystem, safety interlocks, and wiring harness engineering for the hybrid wheel-legged robotic platform. The architecture is developed across two deployment phases:
- **Stage 1 (Prototyping & Field Validation):** An unregulated ~11.1V–12.6V bus powered by a 3S4P 21700 Li-ion pack (~160–186 Wh), supporting 12 smart serial bus servos, 4 brushless/DC wheel actuators, dual-rail regulated logic (Raspberry Pi 5 and Teensy 4.1), and an auxiliary power export suite (100W USB-C PD, 12V switched ignition outlet, and 20W MPPT solar input).
- **Stage 3 (Full-Scale Utility & Mobile Microgrid):** A high-voltage 48V (51.2V nominal) 5.12 kWh LiFePO4 bus driving high-torque quasi-direct-drive (QDD) actuators, integrated with an isolated 1.2 kW pure sine wave AC inverter (120V/230V) for heavy appliances, a 400W semi-flexible solar canopy, and complete industrial ground fault protection (GFCI/RCD).

---

## 1. Power Budget & Battery Design (Stage 1)

### 1.1 Actuator & System Power Draw Breakdown
The Stage 1 robotic platform integrates:
- **Leg Articulation:** 12× high-torque smart serial bus servos (3 degrees of freedom [DOF] per leg: Hip Roll, Hip Pitch, Knee Pitch). Exemplar actuator: Feetech STS3215 / STS3250 (native 9.0V–12.6V rating).
- **Wheel Locomotion:** 4× high-efficiency geared DC/BLDC motors mounted at the feet (e.g., 25GA-370 encoder motors or planetary geared BLDC with FOC ESCs).
- **Compute & Control:** Raspberry Pi 5 (High-Level Vision, SLAM, ROS 2) + Teensy 4.1 (Low-Level Real-Time Kinematics & PDN Monitoring).
- **Sensors & Auxiliaries:** Pi Camera Module 3 / Intel RealSense D435i, BNO085 9-DOF IMU, joint magnetic encoders, cooling fans, and status indicators.

#### Detailed Subsystem Power Draw Table
| Subsystem | Component Details | Qty | Nominal Voltage | Idle / Standby Current | Nominal Dynamic Current | Peak / Stall Current (Burst <2s) |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **Leg Actuators** | Feetech STS3250 Smart Servos | 12 | 11.1V–12.6V (Direct 3S) | 50 mA each (0.6A / 7.2W) | 0.8A–1.2A active (7.0A / 80.5W) | 3.5A stall (42.0A / 483W) |
| **Wheel Drive** | 25GA-370 / Hub Motors | 4 | 11.1V–12.6V (Direct 3S) | 100 mA each (0.4A / 4.8W) | 1.8A each (7.2A / 82.8W) | 6.5A stall (26.0A / 299W) |
| **High-Level SBC** | Raspberry Pi 5 (8GB) | 1 | 5.1V Regulated (Buck) | 0.6A @ 5.1V (3.1W) | 2.5A @ 5.1V (12.8W) | 5.0A @ 5.1V (25.5W) |
| **Real-Time MCU** | Teensy 4.1 (600 MHz M7) | 1 | 5.0V Regulated (Buck) | 100 mA @ 5.0V (0.5W) | 140 mA @ 5.0V (0.7W) | 200 mA @ 5.0V (1.0W) |
| **Vision & Depth** | Pi Cam 3 / RealSense | 1 | 5.0V (via USB 3.0 / Pi 5) | 250 mA @ 5.0V (1.25W) | 700 mA @ 5.0V (3.5W) | 1.4A @ 5.0V (7.0W) |
| **Pose & Encoders** | BNO085 IMU + Encoders | 5 | 3.3V (LDO from Teensy) | 80 mA @ 3.3V (0.26W) | 120 mA @ 3.3V (0.4W) | 180 mA @ 3.3V (0.6W) |
| **Cooling & LEDs** | Noctua 5V Fans + LEDs | 2 | 5V | 150 mA @ 5V (0.75W) | 350 mA @ 5V (1.75W) | 500 mA @ 5V (2.5W) |

#### Operational State Power Totals (11.1V Equivalent Bus)
1. **Quiescent / System Standby:**
   - Actuators unpowered / holding zero torque; compute idling.
   - Total Bus Current: $I_{idle} \approx 1.55\text{ A}$ @ 11.1V. Total Idle Power: $P_{idle} \approx 17.2\text{ W}$.
2. **Nominal Operational Motion (Hybrid Walking / Rolling):**
   - 8 leg servos dynamically loaded during gait transitions, 4 wheels rolling on flat/moderate terrain.
   - Total Bus Current: $I_{nom} \approx 16.5\text{ A}$ @ 11.1V. Total Nominal Power: $P_{nom} \approx 183.2\text{ W}$.
3. **Worst-Case Transient Burst / Obstacle Climb:**
   - Dynamic acceleration, high-incline push, obstacle stepping with 3 servos per stance leg near peak torque.
   - Total Bus Current: $I_{peak} \approx 65.0\text{ A} - 72.0\text{ A}$ @ 11.1V.
   - Total Peak Power: $P_{peak} \approx 720\text{ W} - 800\text{ W}$ (Transient duration limited by software to $\le 2.0\text{ seconds}$).

---

### 1.2 Battery Pack Sizing (3S4P 21700 Configuration)
To maintain high energy density and mechanical compactness within the robot chassis, cylindrical 21700 lithium-ion cells are selected in a **3-Series, 4-Parallel (3S4P)** arrangement (12 cells total).

#### Cell Voltage Thresholds
- **Maximum Pack Voltage (100% SoC):** $3 \times 4.20\text{V} = 12.60\text{V}$
- **Nominal Pack Voltage (50% SoC):** $3 \times 3.65\text{V} - 3.70\text{V} = 10.95\text{V} - 11.10\text{V}$
- **Minimum Cutoff Voltage (0% SoC / UVLO):** $3 \times 3.00\text{V} = 9.00\text{V}$ (Software protection cutoff at $9.60\text{V}$ to prevent deep-discharge degradation).

#### Comparative Analysis: Molicel INR21700-P42A vs. Samsung INR21700-50S
| Parameter | Molicel INR21700-P42A | Samsung INR21700-50S | Trade-off Implication |
| :--- | :--- | :--- | :--- |
| **Nominal Cell Capacity** | 4,200 mAh | 5,000 mAh | 50S provides +19% nominal capacity |
| **Total 3S4P Pack Capacity** | **16.8 Ah (186.5 Wh)** | **20.0 Ah (222.0 Wh)** | 50S yields longer low-power runtimes |
| **Continuous Discharge Rating (Per Cell)** | **45 A** (without temp cut) | **25 A** (35 A with 80°C cut) | P42A handles +80% higher sustained current |
| **Total 3S4P Continuous Discharge** | **180 A** | **100 A** | Both satisfy robot's 72A max burst |
| **AC Internal Resistance ($R_{ac}$)** | $\le 10\text{ m}\Omega$ | $\approx 15 - 18\text{ m}\Omega$ | **P42A has 35–45% lower IR** |
| **Pack DC Resistance ($R_{pack} = \frac{3 \cdot R_{cell}}{4}$)** | **$\approx 7.5\text{ m}\Omega$** | **$\approx 12.5\text{ m}\Omega$** | P42A causes significantly less voltage sag |
| **Voltage Drop @ 60A Burst ($I \cdot R$)** | **$0.45\text{ V}$** | **$0.75\text{ V}$** | P42A avoids servo undervoltage reset |
| **Cycle Life (80% Retention)** | $\ge 500\text{ cycles}$ @ 10A/20A | $\approx 400\text{ cycles}$ @ 10A/25A | P42A delivers superior high-C cycle life |

> **Architectural Selection:** **Molicel INR21700-P42A** is selected. In a dynamic legged robot, servo stall pulses and acceleration transients generate large current spikes. The ultra-low internal resistance of the P42A minimizes bus voltage sag (preventing servo brownouts and logic resets) and significantly reduces internal heat within the enclosed chassis.

#### Estimated Runtime Performance (P42A Pack, 186.5 Wh Nominal)
- **Quiescent Monitoring / Standby:** $\frac{186.5\text{ Wh} \times 0.90\text{ DoD}}{17.2\text{ W}} \approx \mathbf{9.7\text{ hours}}$
- **Continuous Rolling (Wheeled Mode, Flat Ground, ~110W):** $\frac{186.5 \times 0.90}{110\text{ W}} \approx \mathbf{1.53\text{ hours (92 minutes)}}$
- **Aggressive Walking / Rough Terrain (~183W):** $\frac{186.5 \times 0.90}{183.2\text{ W}} \approx \mathbf{55\text{ minutes}}$

---

### 1.3 Battery Management System (BMS) Architecture
To ensure safety and telemetry reporting to the real-time controller, the pack incorporates an onboard **Smart 3S Protection & Fuel Gauge BMS** based on the Texas Instruments **BQ40Z50-R2** (or industrial equivalent Daly/JBD Smart BMS with SMBus/I2C/UART).

#### BMS Electrical & Operational Specifications
- **Continuous Current Rating:** 60 A (continuous thermal rating).
- **Overcurrent Protection (OCP 1 - Software):** 75 A (programmable delay: 200 ms).
- **Overcurrent Protection (OCP 2 - Hardware):** 110 A (trip delay: 10 ms).
- **Short-Circuit Protection (SCP):** 180 A (trip response time: $< 250\text{ }\mu\text{s}$).
- **Overvoltage Cutoff (OVP):** 4.25 V/cell (release at 4.15 V/cell).
- **Undervoltage Cutoff (UVP):** 2.85 V/cell (release at 3.00 V/cell).
- **Cell Balancing:** Integrated passive bleed balancing (50 mA bypass current per cell, active during CV charge stage for $\Delta V > 15\text{ mV}$).
- **Temperature Sensing:** Dual NTC thermistors (10k $\Omega$ B3950).
- **Telemetry Interface:** I2C / SMBus connected to Teensy 4.1: Cell Voltages ($V_1, V_2, V_3$), Total Current ($I_{pack}$), State of Charge ($SoC\%$), and Internal Temperature.

---

### 1.4 Primary Fusing & Wiring Gauge Strategy
Safety requires coordinated multi-stage overcurrent protection:

```
[3S4P Battery Pack] 
        │
   (10 AWG Silicone)
        │
  [60A MIDI Fuse] ── Main Pack Disconnect
        │
  [XT90-S Anti-Spark]
        │
  ┌─────┴───────────────────────────────────────────────────────┐
  │ Power Distribution Board (PDB)                              │
  ├───────────────────┬───────────────────┬────────────────────┤
  │ [30A ATO Fuse]    │ [25A ATO Fuse]    │ [10A Mini Fuse]    │ [5A Mini Fuse]
  │ (12 AWG)          │ (14 AWG)          │ (18 AWG)           │ (20 AWG)
  ▼                   ▼                   ▼                    ▼
[12x Servos Bus]    [4x Wheel ESCs]     [USB-C Buck-Boost]   [5.1V Pi Buck]
```

- **Main Primary Fuse:** Littelfuse MIDI bolt-down fuse (Rating: **60 A**, 32V DC, Interrupting Rating: 2,000 A @ 32 VDC). Bolted directly to the battery positive output terminal block.
- **Wiring Gauge Specifications:**
  - **Main Battery Harness:** **10 AWG** ultra-flexible silicone wire (200°C rated).
  - **Actuator Bus Branch (12 Servos):** **12 AWG** silicone wire into distribution hub, breaking into **20 AWG** feeds.
  - **Wheel Drive Bus Branch (4 ESCs):** **14 AWG** silicone wire.
  - **Logic & Sensor Buck Converters:** **20 AWG** silicone wire.
  - **Signals / I2C / CAN / UART:** **26–28 AWG** twisted-pair shielded cables.

---

## 2. Power Distribution Network (PDN) Schematic

### 2.1 High-Current Bus vs. Regulated Logic Rails Architecture
- **Design Decision: Unregulated Direct Battery Bus for Actuators.**
  - Regulating 60A–80A bursts down to a fixed 12.0V would require an enormous synchronous buck-boost converter weighing $>400\text{ g}$, introducing an extra 6–10% thermal efficiency penalty ($>50\text{ W}$ heat loss during peak dynamic walking).
  - Modern serial bus servos (STS3250) and brushless ESCs operate natively across $9.0\text{V} - 12.6\text{V}$. Closed-loop control in firmware handles voltage variation.
- **Regulated Logic Rails:**
  - Compute and microcontrollers are isolated from the high-current bus via dedicated, high-efficiency synchronous switching regulators.

```
+───────────────────────────────────────────────────────────────────────────────────────────+
│                                 POWER DISTRIBUTION SCHEMATIC                              │
+───────────────────────────────────────────────────────────────────────────────────────────+

  [3S4P Li-ion Pack (9.0V - 12.6V)]
                │
         [60A MIDI FUSE]
                │
       [XT90-S Anti-Spark]
                │
                ├───[P-MOSFET / Contactor E-STOP SWITCH]
                │            │
                │            ▼ (UNREGULATED DIRTY MOTOR BUS: 9.0V - 12.6V)
                │            ├─────────────────────────────────────────► To 12x Leg Servos (30A Fuse)
                │            │   [TVS: SMCJ14CA] + [3x 1000uF Low-ESR]
                │            │
                │            ├─────────────────────────────────────────► To 4x Wheel ESCs (25A Fuse)
                │            │
                │            └─────────────────────────────────────────► To Switched 12V Igniter
                │
                ▼ (CLEAN LOGIC BUS TIE - BEFORE E-STOP ACTUATOR CUTOFF)
                │
         [LC PI-FILTER: 10uH + 2x 22uF MLCC]
                │
                ├───────────────────────────────────────────────────────┐
                ▼                                                       ▼
      [TPS56C215 Buck (5.1V / 5A)]                            [TPS54302 Buck (5.0V / 3A)]
                │                                                       │
         [USB-C Port / GPIO]                                            ├────────► Teensy 4.1 (5V In)
                │                                                       │             │
                ▼                                                       │      [LP5907 Ultra-Low Noise LDO]
       Raspberry Pi 5 (8GB)                                             │             │
        (Vision, SLAM, ROS 2)                                           │             ▼
                                                                        │      Clean 3.3V Rail (IMU & ADCs)
                                                                        │
                                                                        └────────► Depth Camera / Aux Sensors
```

### 2.2 Logic Rail Converter Specifications
1. **Primary High-Power Logic Rail (Raspberry Pi 5 Dedicated):**
   - **Voltage / Current:** **5.10 V $\pm 1\%$ @ 5.0 A** (25.5 W).
   - **Topology:** Synchronous Step-Down Buck Converter using **TI TPS56C215**.
   - **Switching Frequency:** 800 kHz.
   - **Critical Engineering Requirement:** The Raspberry Pi 5 requires a true 5.1V rail capable of 5A delivery with $< 50\text{ mV}$ peak-to-peak ripple to avoid CPU throttling.
2. **Secondary Auxiliary Logic Rail (Teensy & Sensors):**
   - **Voltage / Current:** **5.00 V $\pm 1\%$ @ 3.0 A** (15.0 W).
   - **Topology:** Synchronous Buck Converter using **TI TPS54302**.
3. **Ultra-Low Noise Analog Rail (Sensor IMU & Magnetic Encoders):**
   - **Voltage / Current:** **3.30 V @ 250 mA**.
   - **Topology:** Ultra-Low-Noise LDO Regulator (**TI LP5907**), powered from the 5.0V clean rail. PSRR $> 82\text{ dB}$ at 1 kHz; output noise $< 6.5\text{ }\mu\text{V}_{RMS}$.

---

### 2.3 Isolation, Ground Loop Prevention & Inductive Noise Suppression
- **Star Grounding Architecture:**
  - A strict **Star Ground Point** is established directly at the negative terminal of the XT90-S battery connector.
  - `DIRTY_GND` (servos, ESCs, igniter) and `CLEAN_GND` (compute, sensors, analog) are physically segregated and joined at only one physical point through a ferrite bead (Murata BLM31PG121SN1, 120 $\Omega$ @ 100 MHz, 6A DC) and a 0 $\Omega$ bridging link.
- **LC PI-Filter Design for Logic Protection:**
  - Filter topology: 100 $\mu$F 25V Low-ESR electrolytic + 10 $\mu$H shielded power inductor + 2× 22 $\mu$F 25V X7R ceramic capacitors. Provides $> 42\text{ dB}$ attenuation across the 20 kHz to 500 kHz servo PWM band.
- **Back-EMF Clamping & Transient Voltage Suppression (TVS):**
  - 3× 1,000 $\mu$F 25V Low-ESR electrolytic capacitors (Panasonic FR series, ESR $< 18\text{ m}\Omega$) across servo power headers.
  - Bidirectional TVS Diode: **Littelfuse SMCJ14CA** ($V_{RWM} = 14.0\text{V}$, $V_{BR} = 15.6\text{V}-17.2\text{V}$, $V_C = 23.2\text{V}$ at 64.7A peak pulse).
- **Communication Bus Galvanic Isolation:**
  - The serial half-duplex UART bus driving the 12× Feetech servos is isolated from the Teensy 4.1 using a high-speed digital isolator (**TI ISO7721**).

---

## 3. Power Export Subsystem (Stage 1 Demo)

The Stage 1 platform includes an energy export and harvesting demonstration suite, validating the robot's role as a mobile micro-utility hub.

```
+───────────────────────────────────────────────────────────────────────────────────────────+
│                              POWER EXPORT SUBSYSTEM SCHEMATIC                             │
+───────────────────────────────────────────────────────────────────────────────────────────+

                                  +───────────────────────────────────+
                                  │ 3S4P Battery Bus (9.0V - 12.6V)   │
                                  +───────────────────────────────────+
                                                    │
                 ┌──────────────────────────────────┼──────────────────────────────────┐
                 │ (10A Fuse)                       │ (5A Fuse)                        │ (3A Fuse)
                 ▼                                  ▼                                  ▼
      +───────────────────────+          +───────────────────────+          +───────────────────────+
      │   IP2368 Synchronous  │          │ High-Side P-FET Switch│          │ TI BQ24650 Synchronous│
      │ 4-Switch Buck-Boost   │          │ (IRF7424 / Opto-Drv)  │          │ MPPT Solar Charger    │
      +───────────────────────+          +───────────────────────+          +───────────────────────+
                 │                                  │                                  ▲
       [USB-C PD 3.0 Port]                [12V Switched Barrel]                        │
       (5V-20V @ up to 100W)              (Burner Spark Igniter)                       │
                 │                                  │                                  │
                 ▼                                  ▼                         [18V / 20W Solar Panel]
       Laptop / Field Tools               Propane Camp Burner                 (External Harvesting)
                 │                                  │
                 └──────────────────┬───────────────┘
                                    │
                                    ▼
                         [INA226 Digital Monitors]
                          (I2C to Teensy 4.1 MCU)
```

### 3.1 USB-C Power Delivery (PD) Module: IP2368 Buck-Boost Architecture
- **Topology Selection:** A **4-Switch Synchronous Buck-Boost Converter** is mandatory because a 3S pack cannot generate 15V or 20V via step-down alone.
- **Core Controller:** **Injoinic IP2368** (Bidirectional high-power buck-boost controller with integrated USB PD3.0 protocol engine).
  - External Switching Power Stage: 4× N-Channel MOSFETs (AON6512, 30V, 65A, $R_{DS(on)} = 3.8\text{ m}\Omega$).
  - Power Output Capability: **Up to 100 W (20V @ 5A)** when pack $> 10.5\text{V}$; automatically negotiates down to **65 W (20V @ 3.25A)** if pack drops below $10.0\text{V}$.
  - Protocols: USB PD 3.0 (PPS: 3.3V–21V @ 5A), QC 4+, AFC, FCP.

### 3.2 Switched 12V DC Auxiliary Outlet (Burner Spark Igniter)
- Controlled 12V DC auxiliary outlet (5.5×2.1 mm locking barrel jack) for field stove spark ignition.
- High-side P-Channel MOSFET (**Infineon IRF7424**, 30V, 11A, $R_{DS(on)} = 13.5\text{ m}\Omega$) driven via optocoupler (**LTV-817**) from Teensy GPIO.
- Protection: Vishay SS34 Schottky freewheeling diode, 10 $\Omega$ + 100 nF RC snubber, and 2.0A PTC resettable fuse.

### 3.3 Digital Current Sensing & Power Telemetry via Teensy 4.1
- **Sensor:** Texas Instruments **INA226** (16-bit Delta-Sigma ADC with bidirectional high-side current and bus voltage sensing over I2C).
- **Allocations:**
  1. `INA226_BATTERY` (`0x40`): Total Battery inflow/outflow via $2.0\text{ m}\Omega$, 5W shunt.
  2. `INA226_ACTUATORS` (`0x41`): Leg + Wheel motor bus via $2.5\text{ m}\Omega$, 3W shunt.
  3. `INA226_EXPORT` (`0x44`): USB-C PD + 12V Igniter outlet via $10.0\text{ m}\Omega$, 2W shunt.
- Telemetry published over ROS 2 topics: `/power/battery_state` and `/power/export_telemetry`.

### 3.4 Solar Charge Controller (MPPT Input Subsystem)
- **Controller IC:** Texas Instruments **BQ24650** (Synchronous switch-mode buck battery charge controller with input voltage regulation loop).
- Sized for an external 18V / 20W portable solar panel ($V_{mp} \approx 18.0\text{V}$, $I_{mp} \approx 1.11\text{A}$).
- Dynamic MPPT tracking prevents solar panel voltage collapse under partial cloud cover.

---

## 4. Safety Interlocks & E-Stop Circuitry

Safety architecture mandates that actuator motion must be inhibited via autonomous hardware cutoffs without relying on high-level software operating systems.

```
+───────────────────────────────────────────────────────────────────────────────────────────+
│                            SAFETY & E-STOP INTERLOCK ARCHITECTURE                         │
+───────────────────────────────────────────────────────────────────────────────────────────+

  [Dual-Channel Physical E-Stop]      [Wireless RC Receiver Failsafe]
     (Red Twist Mushroom Button)        (Opto-Isolated Watchdog Relay)
          │                                  │
          │ (NC Contact 1)                   │ (NC Relay)
          └──────────────────┬───────────────┘
                             │
                             ▼
               [Hardware Safety Control Loop]
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
    [LM5060 High-Side Gate Driver]   [Optocoupler TLP2361]
              │                             │
              ▼                             ▼
    [4x Parallel Power MOSFETs]      [Teensy 4.1 Hardware INT]
   (IPT012N08N5, 0.3 mOhm Array)            │
              │                             ▼
              ▼                     Instant Software Halt:
    +───────────────────+           - Clear PWM registers
    │ CUTS ALL POWER TO:│           - Command 0 Torque
    │ - 12x Leg Servos  │           - Broadcast ROS 2 E-Stop
    │ - 4x Wheel Motors │
    +───────────────────+
    (Compute & Sensors Remain Powered!)
```

### 4.1 Dual-Channel Hardware E-Stop Architecture
- Conforms to ISO 13849-1 Category 3 / PL d principles.
- Channels: (1) Chassis 22 mm red mushroom twist-to-release switch (NC), and (2) Wireless RC ELRS failsafe watchdog relay.
- High-Side Disconnect: Drives **TI LM5060** controlling 4× parallel N-Channel power MOSFETs (**Infineon IPT012N08N5**, array $R_{DS(on)} \approx 0.3\text{ m}\Omega$). Cuts actuator power within $18\text{ }\mu\text{s}$.
- Compute rails (Pi 5 and Teensy) tap power **upstream** of the E-stop switch, maintaining diagnostics and telemetry during an emergency stop.

### 4.2 Drive-Away Interlock (Cable-Presence Detection)
- Detects physical connection of USB-C cable (via IP2368 `VBUS_DET` / `PG`), 12V barrel plug (via mechanical shunt switch in jack), or solar panel tether.
- Multi-modal signals feed a hardware OR gate (**74LVC1G32**) which clamps the hardware `DRIVE_ENABLE` lines of the wheel motor ESCs to ground. Wheel rotation is **physically prohibited** while plugged in.

---

## 5. Scaling to Stage 3 (48V Bus, Inverter & 1 kW Appliances)

Stage 3 transforms the platform into an expeditionary, utility-scale mobile microgrid.

### 5.1 Bus Architecture Transition: 12V vs. 48V
- At 1.0 kW AC output ($1,110\text{ W}$ DC from inverter):
  - 12V bus requires **92.5 A** (heavy 2 AWG cabling, high thermal loss).
  - 48V bus requires **23.1 A** (compact 12 AWG cabling, 16× lower ohmic losses).
- 48V natively interfaces with industrial quasi-direct-drive (QDD) actuators (e.g., T-Motor AK80 series or Unitree Go-M8010).

### 5.2 Battery Chemistry & Sizing: 5.12 kWh LiFePO4 (LFP) Architecture
- **Chemistry:** 16S 1P Prismatic LiFePO4 (CATL / EVE LF105, 100 Ah).
- **Nominal Voltage:** $51.2\text{ V}$ ($44.0\text{V} - 58.4\text{V}$ range).
- **Total Capacity:** $5,120\text{ Wh (5.12 kWh)}$. Total pack mass $\approx 46\text{ kg}$.
- LFP provides superior thermal safety (no oxygen release in thermal runaway, onset $> 270^\circ\text{C}$) and 3,500–6,000 cycle lifespan. Mounted as low-chassis ballast.

### 5.3 Pure Sine Wave Inverter Integration (1.2 kW Continuous)
- **Model:** **Victron Phoenix 48/1200 Smart** (1,000W cont., 2,200W peak surge pure sine wave, 94% efficiency).
- **Earthing & Ground Fault Protection:** Complies with ISO 60204-1 and NEC Article 250. Includes internal Neutral-to-Earth (N-E) bonding link to chassis and an industrial **Type B Residual Current Device (RCD / GFCI)** with a $10\text{ mA} - 30\text{ mA}$ trip threshold.

### 5.4 Flexible Solar Canopy Harvesting (300W–400W Array)
- 2S2P SunPower Maxeon Gen III semi-flexible monocrystalline panels on dorsal deck.
- Paired with **Victron SmartSolar MPPT 100/20** charge controller.

---

## 6. Wiring Harness, Connectors & Custom PCB Design Plan

### 6.1 Connector Standardization Hierarchy
| Tier | Connector Model | Rating | Key Feature |
| :--- | :--- | :---: | :--- |
| **Main Battery Pack** | AMASS XT90-S | 90A Peak / 45A Cont. | Integrated anti-spark pre-charge resistor |
| **Actuator Trunks** | AMASS XT60H | 60A Peak / 30A Cont. | Snap-on insulating sheath |
| **Auxiliary / Solar** | AMASS XT30U | 30A Peak / 15A Cont. | Compact nylon high-temp housing |
| **Sensors & Real-Time Bus** | JST-GH (1.25 mm) | 1.0 A | Positive acoustic click-latch (vibration-proof) |
| **Balance Header** | JST-XH (2.54 mm) | 3.0 A | Standard keyed multi-pin balance plug |
| **Solar Bulkhead** | Weipu SP13 (2-Pin) | 13.0 A | IP68 threaded waterproof circular connector |
| **Auxiliary 12V Outlet** | Locking 5.5×2.1 mm Barrel | 5.0 A | Threaded locking collar |
| **High-Power USB-C** | Amphenol LUSB IP67 Type-C | 5.0 A | Sealed chassis mount with silicone O-ring |

### 6.2 Leg Joint Wiring Harness Engineering
- **Flex-Life Conductors:** Ultra-fine stranded silicone wire (0.05 mm copper stranding) rated for $> 500,000$ flex cycles.
- **Service Loops:** Service loop routing with torsion rather than tight 90° bending radii along joint rotation axes.
- **Abrasion Protection:** Expandable braided PET sleeving with heat-shrink strain relief boots at all anchor points.
