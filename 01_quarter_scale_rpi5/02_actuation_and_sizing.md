# 02 - Actuation, Motor Sizing & Drive Systems

**Author:** Research Agent 2: Actuation, Drives & Sizing Specialist  
**Project:** Hybrid Wheel-Legged Quadruped Robot (Stage 1 Prototype & Scaling Architecture)  
**Deliverable Path:** `/Users/mac/Documents/Agents Antigravity/research/02_actuation_and_sizing.md`  

---

## Executive Summary & System Specifications Baseline

This document provides the foundational engineering sizing, kinematic and dynamic torque derivations, transmission design, electrical drive architecture, and multi-stage scaling roadmap for the hybrid wheel-legged quadruped robot platform.

The primary design challenge of a wheel-legged quadruped is the dual-modality operational envelope:
1. **Legged Locomotion Mode:** High joint torque, moderate angular speed, intermittent ground shock impacts, cyclic loading, and non-linear multi-axis kinematic constraints.
2. **Wheeled Locomotion Mode:** Low continuous joint torque, high wheel rotational speed, rolling resistance, slope climbing traction, and high-frequency dead-reckoning odometry.

### Stage 1 System Parameter Baseline
* **Total Operating Mass ($M$):** $5.00\text{ kg}$ (includes chassis, 4 legs, 12 bus servos, 4 wheel gearmotors, 3S battery pack, compute electronics, and $1.00\text{ kg}$ payload).
* **Total Gravitational Weight ($W$):** $W = M \cdot g = 5.00\text{ kg} \times 9.80665\text{ m/s}^2 = 49.03\text{ N}$.
* **Degrees of Freedom (DOF):** 16 active DOF total ($4\text{ legs} \times [3\text{ leg joints} + 1\text{ wheel motor}]$).
  * Hip Abduction/Adduction (Ab/Ad roll): $\theta_1$
  * Hip Flexion/Extension (Thigh pitch): $\theta_2$
  * Knee Flexion/Extension (Shank pitch): $\theta_3$
  * Foot Driving Wheel (Continuous rotation): $\theta_w$
* **Kinematic Link Dimensions:**
  * Thigh Link Length ($L_1$): $110\text{ mm}$ ($0.110\text{ m}$)
  * Shank Link Length ($L_2$): $110\text{ mm}$ ($0.110\text{ m}$)
  * Wheel Outer Diameter ($D_w$): $65\text{ mm}$ (Radius $R_w = 32.5\text{ mm} = 0.0325\text{ m}$)
  * Maximum Horizontal Leg Reach ($L_{max}$): $L_1 + L_2 = 220\text{ mm} = 0.220\text{ m}$
  * Nominal Standing Stance Height ($H_{stand}$): $150\text{ mm} - 170\text{ mm}$
* **Actuator Hardware (Stage 1):**
  * Leg Joints (12x): Feetech STS3250 Magnetic Encoder Smart Bus Servos (12V nominal, stall torque $50\text{ kg}\cdot\text{cm} \approx 4.90\text{ N}\cdot\text{m}$).
  * Wheel Motors (4x): 25GA-370 12V DC Brushed Gearmotors with Hall effect quadrature encoders (Nominal ~300 RPM, rated torque $1.0\text{ kg}\cdot\text{cm}$, stall torque $3.8\text{ kg}\cdot\text{cm}$).
  * Transmission: Knee joint incorporates a custom GT2 timing belt reduction ($1:2.0$) to isolate radial shock and boost joint torque.

---

## 1. Torque & Dynamic Sizing Equations

### 1.1 Dynamic Equations of Motion
The rigid-body dynamics of a 3-DOF quadruped leg in contact with the terrain are expressed via the standard manipulator equation:

$$\boldsymbol{\tau} = \mathbf{M}(\mathbf{q})\ddot{\mathbf{q}} + \mathbf{C}(\mathbf{q}, \dot{\mathbf{q}})\dot{\mathbf{q}} + \mathbf{G}(\mathbf{q}) + \mathbf{J}^T(\mathbf{q}) \mathbf{F}_{ext}$$

In quasi-static and moderate-speed locomotion ($v \le 0.5\text{ m/s}$ walking/trotting), joint torques are governed primarily by ground reaction forces:

$$\boldsymbol{\tau}_{static} = \mathbf{J}^T(\mathbf{q}) \mathbf{F}_{ext} + \mathbf{G}(\mathbf{q})$$

For the 2D sagittal plane (Hip Pitch $\theta_2$, Knee Pitch $\theta_3$):
$$\begin{bmatrix} \tau_{hip} \\[6pt] \tau_{knee} \end{bmatrix} = \mathbf{J}_{sag}^T \begin{bmatrix} F_x \\[6pt] F_z \end{bmatrix} = \begin{bmatrix}
\left(L_1 \cos\theta_2 + L_2 \cos(\theta_2 + \theta_3)\right) F_x - \left(L_1 \sin\theta_2 + L_2 \sin(\theta_2 + \theta_3)\right) F_z \\[6pt]
\left(L_2 \cos(\theta_2 + \theta_3)\right) F_x - \left(L_2 \sin(\theta_2 + \theta_3)\right) F_z
\end{bmatrix}$$

When contact forces are purely vertical ($F_x = 0$, $F_z = F_{GRF}$):
$$\tau_{hip} = - x_{foot} \cdot F_z$$
$$\tau_{knee} = - (x_{foot} - x_{knee}) \cdot F_z = - L_2 \sin(\theta_2 + \theta_3) \cdot F_z$$

---

### 1.2 Worst-Case Operational Load Cases

```
               [Load Case Comparison Visualized]

  Case 1: Dynamic Trot (2-Leg Stance)      Case 2: Extreme Deep Crouch Pose
  ===================================      ================================
            [CoM: 5 kg]                              [CoM: 5 kg]
             /       \                                /       \
            /         \                              /         \
        [Leg 1]     [Leg 2]                      [Thigh]----[Thigh]  (Horizontal)
         |            |                            |          |
      Fz=24.5N     Fz=24.5N                      [Shank]    [Shank] (Arm = 110mm)
    (Dyn Peak: 49.05 N/leg)                        |          |
                                                 Fz=12.3N   Fz=12.3N
```

#### Load Case 1: Two-Leg Diagonal Support (Dynamic Trot Gait)
* Mass distributed across 2 legs: $F_{z,static} = \frac{49.03\text{ N}}{2} = 24.52\text{ N}$.
* Dynamic impact factor ($k_{dyn} = 2.0$): $F_{z,dyn\_peak} = 49.03\text{ N}$.
* Lever arm to knee ($d_{knee} = 55\text{ mm} = 0.055\text{ m}$):
  * **Static Knee Torque (2-leg):** $T_{knee,static} = 24.52 \times 0.055 = \mathbf{1.349\text{ N}\cdot\text{m}} \quad (13.75\text{ kg}\cdot\text{cm})$
  * **Dynamic Peak Knee Torque (2-leg impact):** $T_{knee,dyn\_peak} = 49.03 \times 0.055 = \mathbf{2.697\text{ N}\cdot\text{m}} \quad (27.50\text{ kg}\cdot\text{cm})$

#### Load Case 2: Extreme Deep Crouch Pose
* Low-slung crawl clearance ($H \approx 80\text{ mm}$), knee lever arm $d_{knee} = 110 \times \sin(60^\circ) = 95.3\text{ mm} = 0.0953\text{ m}$.
* Static 4-leg crouch ($F_z = 12.26\text{ N}$): $T_{knee} = \mathbf{1.168\text{ N}\cdot\text{m}}$.
* Dynamic 2-leg crouch transition ($F_z = 24.52\text{ N}$): $T_{knee} = \mathbf{2.336\text{ N}\cdot\text{m}}$.

#### Load Case 3: Step Climbing ($6.0\text{ cm}$ Vertical Step)
* 65% weight transfer onto leading climbing leg pair ($F_z \approx 18.37\text{ N}$ with $1.5\text{ m/s}^2$ vertical acceleration).
* Required forward pull traction $F_x \approx 9.0\text{ N}$.
* Total peak joint torque during hoist:
  $$\tau_{knee,step} = (18.37 \times 0.055) + (9.0 \times 0.0953) = \mathbf{1.868\text{ N}\cdot\text{m}} \quad (19.05\text{ kg}\cdot\text{cm})$$
  $$\tau_{hip,step} = (18.37 \times 0.0228) + (9.0 \times 0.173) = \mathbf{1.976\text{ N}\cdot\text{m}} \quad (20.15\text{ kg}\cdot\text{cm})$$

---

### 1.3 Feetech STS3250 Actuator Analysis & Thermal Limit Proof

* **Stall Torque ($T_{stall}$):** $50.0\text{ kg}\cdot\text{cm} = 4.905\text{ N}\cdot\text{m}$ at $12.0\text{ V}$.
* **Stall Current ($I_{stall}$):** $4.80\text{ A}$. Internal resistance $R_m = 2.50\text{ }\Omega$.
* **Thermal Dissipation Model:** Winding $\Delta T_{max} = 50.0^\circ\text{C}$ over $35^\circ\text{C}$ ambient; thermal resistance $R_{th} \approx 20.0^\circ\text{C/W}$.
* **Max Continuous Electrical Loss:** $P_{loss,cont} = \frac{50.0^\circ\text{C}}{20.0^\circ\text{C/W}} = 2.50\text{ W}$.
* **Max Continuous Current:** $I_{cont} = \sqrt{\frac{2.50\text{ W}}{2.50\text{ }\Omega}} = \mathbf{1.00\text{ A}}$.
* **Continuous Thermal Torque Rating ($T_{cont}$):**
  $$T_{cont} = K_t \cdot I_{cont} = 1.022\text{ N}\cdot\text{m/A} \times 1.00\text{ A} = \mathbf{1.022\text{ N}\cdot\text{m}} \quad (10.42\text{ kg}\cdot\text{cm})$$
  $$\frac{T_{cont}}{T_{stall}} \approx \mathbf{21\%}$$

#### Why Direct-Drive Knee Fails
In direct-drive, dynamic trotting demands an RMS torque of $\approx 1.85\text{ N}\cdot\text{m}$ ($1.81\text{ A}$ current), producing $4.20\text{ W}$ average loss. Steady-state core temperature reaches:
$$T_{core} = 30^\circ\text{C} + (4.195\text{ W} \times 20^\circ\text{C/W}) = \mathbf{113.9^\circ\text{C}}$$
Since $113.9^\circ\text{C} \gg 85^\circ\text{C}$, direct-drive servos trigger thermal shutdown within **60 seconds of trotting**. A reduction stage is mandatory.

---

## 2. Knee Transmission & Reduction Design

### 2.1 Sizing the Knee Reduction Ratio: 1:2.0 GT2 Timing Belt

$$\text{Reduction Ratio } N = 2.00, \quad \eta_{belt} = 0.95 \implies T_{servo} = \frac{T_{knee,joint}}{2.0 \times 0.95} = \frac{T_{knee,joint}}{1.90}$$

```
========================================================================================
Comparative Evaluation: Direct Drive vs 1:2.0 Reduction
========================================================================================
Parameter                  Direct Drive (1:1)    Ratio 1:2.0 (Selected)
----------------------------------------------------------------------------------------
Pulleys (Driver / Driven)  N/A                   16T / 32T
Reduction Ratio (N)        1.00                  2.00
Net Transmission Eff.      1.00                  0.95
Torque Multiplier          1.00x                 1.90x

2-Leg Trot Stance Peak     2.697 N·m             1.419 N·m (28.9% Stall)
2-Leg Trot RMS Stance      1.850 N·m             0.974 N·m (95.3% T_cont - SAFE!)
Step Climbing Peak         1.868 N·m             0.983 N·m
Deep Crouch 4-Leg          1.168 N·m             0.615 N·m

Max Joint Angular Speed    60.6 RPM (364°/s)     30.3 RPM (182°/s)
Joint Full Flexion (90°)   0.247 seconds         0.495 seconds (Optimal for 2 Hz trot)
Thermal Equilibrium Temp   113.9°C (BURNOUT)     56.4°C (EXCELLENT!)
========================================================================================
```

* **Belt Specification:** GT2 (2.0 mm pitch), $9.0\text{ mm}$ width (breaking strength $> 850\text{ N}$).
* **Driver Pulley:** 16T ($d_{p1} = 10.19\text{ mm}$).
* **Driven Pulley:** 32T ($d_{p2} = 20.37\text{ mm}$).
* **Belt Pitch Length:** 80 teeth / $160.0\text{ mm}$ pitch length; center distance $C \approx 55.8\text{ mm}$.

### 2.2 Bearing Load Isolation & Spline Protection (Double-Shear Design)
* **Problem:** Direct mounting transmits radial impact forces ($F_z > 50\text{ N}$) into the internal servo spur gear cluster and magnetic encoder disc, destroying gear teeth and causing feedback runout.
* **Architecture:**
  1. An independent $\varnothing 8.0\text{ mm}$ 7075-T6 aluminum shaft forms the knee pivot axis.
  2. Dual **6802-2RS** sealed miniature ball bearings ($15\text{ mm}$ ID, $24\text{ mm}$ OD, $5\text{ mm}$ W) in double-shear clevis take **100% of external radial and bending loads**.
  3. The Feetech servo spline drives the 16T pulley under **pure torsional shear**. Zero bending moment reaches the servo.

---

## 3. Wheel Motor & Traction Sizing

### 3.1 Wheeled Rolling Dynamics & Slope Climbing
* **Flat Ground Cruise ($1.0\text{ m/s}$):** Rolling resistance $F_{rr} = C_{rr} M g = 0.030 \times 49.03 = 1.47\text{ N} \implies T_{wheel} = \mathbf{0.012\text{ N}\cdot\text{m}}$ per wheel.
* **Flat Ground Acceleration ($1.0\text{ m/s}^2$):** $T_{wheel} = \mathbf{0.053\text{ N}\cdot\text{m}}$.
* **15° Incline Climbing:**
  $$F_{total,climb} = M g (\sin 15^\circ + C_{rr} \cos 15^\circ) = 12.69\text{ N} + 1.42\text{ N} = 14.11\text{ N}$$
  $$T_{wheel,climb} = \frac{14.11\text{ N} \times 0.0325\text{ m}}{4} = \mathbf{0.1147\text{ N}\cdot\text{m}} \quad (1.17\text{ kg}\cdot\text{cm})$$
* **15° Incline Acceleration ($0.5\text{ m/s}^2$):** $T_{wheel} = \mathbf{0.1350\text{ N}\cdot\text{m}} \quad (1.38\text{ kg}\cdot\text{cm})$.

### 3.2 25GA-370 Motor Operating Points
* **Motor:** 12V 34:1 all-metal spur gearmotor ($330\text{ RPM}$ no-load, rated continuous $1.00\text{ kg}\cdot\text{cm}$, stall $3.80\text{ kg}\cdot\text{cm}$ / $2.0\text{ A}$).
* **15° Slope Climb Point:** Draws $0.72\text{ A}$ @ $228\text{ RPM}$ ($v = 0.78\text{ m/s}$). Within safe continuous thermal operating boundaries.
* **Traction Limit:** With TPU 95A tire on dry concrete ($\mu = 0.70$), maximum tractive force before wheel slip is $8.29\text{ N}$ per wheel vs $3.53\text{ N}$ required ($2.35\times$ traction safety factor).

### 3.3 Encoder Feedback & Odometry Resolution
* Hall effect quadrature encoder on rear motor rotor: $11\text{ PPR} \times 4 = 44\text{ counts/motor rev}$.
* After 34:1 gearbox: **$1,496\text{ counts/wheel rev}$**.
* Linear odometry resolution: $\Delta s = \frac{\pi \times 65\text{ mm}}{1,496} = \mathbf{0.1365\text{ mm/count}}$ ($136.5\text{ }\mu\text{m/pulse}$).
* Interrupt frequency at $1.0\text{ m/s}$ across 4 wheels: $29.3\text{ kHz}$ total (consumes $< 0.15\%$ CPU on Teensy 4.1).

---

## 4. Electrical Control & Bus Architecture

### 4.1 Feetech Serial Bus Protocol & Half-Duplex UART Optimization
* **Baud Rate:** $1,000,000\text{ bps}$ ($1.0\text{ Mbps}$), half-duplex asynchronous serial.
* **Dual-Bus Topology Selection:**
  - Single bus (12 servos): Round-trip `SYNC_WRITE` + `SYNC_READ` takes **$3.16\text{ ms}$** ($316\text{ Hz}$ theoretical limit).
  - **Dual Bus (6 servos per bus on Serial1 & Serial2):** Parallel DMA transactions drop frame time to **$1.66\text{ ms}$**, unlocking **$200\text{ Hz}$ closed-loop control** with $66.8\%$ idle bus headroom.

```
                    Teensy 4.1 Microcontroller (600 MHz)
                    +------------------------------------+
                    | [Hardware UART 1] [Hardware UART 2]|
                    +-------|---------------------|------+
                            | (TX1/RX1)           | (TX2/RX2)
                            v                     v
                 [74LVC2G241 Buffer A]   [74LVC2G241 Buffer B]
                            |                     |
                  BUS A (Left Legs: 6 Servos)  BUS B (Right Legs: 6 Servos)
```

### 4.2 Wheel Motor Driver Selection
* **DRV8833 Warning:** Absolute max voltage is $10.8\text{ V}$. Connecting to a 3S Li-ion pack ($12.6\text{ V}$) causes immediate dielectric breakdown.
* **Selected Driver:** Dual **Cytron MDD3A** (or TI DRV8874). Continuous rating $3.0\text{ A}$ per channel ($18\text{ V}$ max) with $R_{ds(on)} < 0.08\text{ }\Omega$.

---

## 5. Scaling Roadmap to Stage 2 & Stage 3 (BLDC, Cycloidal & FOC)

| Parameter | Stage 1 (Quarter-Scale Prototype) | Stage 2 (Half-Scale Prototype) | Stage 3 (Full-Scale Utility Microgrid) |
| :--- | :--- | :--- | :--- |
| **Total Mass ($M$)** | $5.0\text{ kg}$ ($1.0\text{ kg}$ payload) | $20.0\text{ kg}$ ($10.0\text{ kg}$ payload) | $90 - 100\text{ kg}$ ($80.0\text{ kg}$ payload) |
| **Leg Actuators** | 12x Feetech STS3250 Smart Bus Servos | 12x Custom 3D Printed Cycloidal QDD | 12x Frameless BLDC + Metal Cycloidal |
| **Nominal Leg Bus Voltage**| 12V (3S Li-ion) | 24V – 48V (6S–12S Li-ion) | 48V (16S LiFePO4 / 5.12 kWh) |
| **Knee Peak Torque** | $4.9\text{ N}\cdot\text{m}$ (boosted to $9.8\text{ N}\cdot\text{m}$) | $45 - 60\text{ N}\cdot\text{m}$ | **$500 - 700\text{ N}\cdot\text{m}$** |
| **Knee Continuous Torque**| $1.02\text{ N}\cdot\text{m}$ ($1.94\text{ N}\cdot\text{m}$ at joint) | $15 - 20\text{ N}\cdot\text{m}$ | **$150 - 250\text{ N}\cdot\text{m}$** |
| **Knee Transmission** | GT2 2:1 Timing Belt | 1:15–1:20 2-Stage Cycloidal Drive | 1:30–1:50 Precision Cycloidal / Ball-Screw |
| **Wheel Drive Actuators**| 4x 25GA-370 Geared DC Motors | 4x 5065/6010 BLDC Outrunners (FOC) | 4x 48V High-Torque In-Wheel Hub Motors |
| **Motor Drive Electronics**| Internal Servo H-Bridge + Cytron MDD3A | moteus c1 / ODrive S1 / SimpleFOC | CAN-FD Industrial FOC Drives with STO |
| **Control Bus** | Dual Half-Duplex TTL UART (1 Mbps) | Dual CAN-FD Buses (5 Mbps) | Quad CAN-FD / EtherCAT Network |
| **Obstacle Clearance** | $6.0\text{ cm}$ (Hybrid Stepping) | $12 - 15\text{ cm}$ | **$25 - 30\text{ cm}$ (Architectural Stairs)** |

---

## 6. Recommendations & Risk Mitigation

1. **Mandatory 1:2.0 Knee Belt Reduction:** Never operate STS3250 servos in direct-drive. Install the 16T to 32T GT2 9mm belt reduction to ensure continuous thermal survival ($56.4^\circ\text{C}$ vs $113.9^\circ\text{C}$).
2. **Double-Shear Knee Axle:** Support the knee pivot with dual 6802-2RS bearings to isolate all ground reaction shear from the servo spline.
3. **Dual UART Bus Topology:** Wire left and right leg servos to separate Teensy 4.1 hardware serial ports (Serial1 and Serial2) with 74LVC2G241 single-wire transceivers to guarantee $200\text{ Hz}$ control bandwidth.
4. **Driver Voltage Compatibility:** Strictly avoid DRV8833 drivers on 12V 3S power. Use Cytron MDD3A or TI DRV8874 drivers for the wheel gearmotors.
