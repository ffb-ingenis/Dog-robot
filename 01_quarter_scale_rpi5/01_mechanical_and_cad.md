# 01 - Mechanical Design, CAD Pipeline & 3D Printing Specification (Project 1: Stage 1)

**Author:** Mechanical & 3D Printing Specialist  
**Project:** Project 1 — Quarter-Scale Wheel-Legged Quadruped (Raspberry Pi 5 + Pi Camera 3)  
**Path:** `robot/01_quarter_scale_rpi5/01_mechanical_and_cad.md`  

---

```
========================================================================================
             STAGE 1 WHEELED QUADRUPED ROBOT MECHANICAL SPECIFICATION
========================================================================================
 Total Mass (Loaded):  4.8 kg nominal (max 5.0 kg with 1.0 kg payload)
 Kinematic Topology:   4 legs × [3-DOF hip/knee + 1-DOF driven foot wheel] = 16 Actuators
 Leg Configuration:    Symmetric X-Configuration (< >): Front knees back, Rear knees forward
 Leg Link Lengths:     Thigh (L1) = 105 mm | Shank (L2) = 105 mm | Wheel Radius (Rw) = 32.5 mm
 Standing Height:      181 mm (nominal crouch) | 242.5 mm (max extension) | 137.5 mm (cruise)
 Actuation (Legs):     12× Feetech STS3250 Serial Bus Servos (12V, 4.9 N·m stall, magnetic enc)
 Actuation (Knee):     GT2 2mm Pitch Timing Belt Stage (2:1 Reduction, 9.8 N·m peak)
 Actuation (Wheels):   4× 25GA-370 12V DC Gearmotors (300 RPM, Hall quadrature encoders)
 Chassis Architecture: Open-frame industrial utility: CNC 6061-T6 plates + CF tubes + PA-CF
 Protection & Cargo:   Front/rear tubular aluminum roll bars + top rack for cargo & lid solar
 Foot Contact:         65 mm Compliant Airless TPU 95A Tire on PA-CF Rim (4-5 mm shock travel)
 Step Climbing Limit:  6.0 cm vertical riser (Stage 1) | 12-15 cm (Project 2) | 25-30 cm (Stage 3)
========================================================================================
```

---

## 1. Executive Summary & Kinematics Baseline

### 1.1 Kinematic Topology and Spatial Dimensions
Project 1 (Stage 1) is a quarter-scale wheeled quadruped designed to validate rolling locomotion, active-suspension compliance, and discrete step-climbing prior to Project 2 (half-scale Jetson Orin Nano + OAK-D Lite) and Stage 3 (80 kg payload, Jetson Thor).

**Symmetric X-Configuration (`< >`):** Front legs bend knees backward (`FL`, `FR` with $q_3 < 0$), and rear legs bend knees forward (`RL`, `RR` with $q_3 > 0$ symmetrically mirrored). This prevents front knees from striking vertical step risers during ascent and balances front/rear pitch stiffness.

```
                   [Insulated Box + Lid Solar Panel]
                   +-------------------------------+
          Roll Bar |       TOP BOLTED RACK         | Roll Bar
           (Front) +-------------------------------+ (Rear)
        [Cam/Mic]  |  OPEN-FRAME 6061 CHASSIS      | [Power Panel]
                   +---+-----------------------+---+
                      /                         \
                     /   Symmetric X-Config      \
                    * (Knee)               (Knee) *
                     \                           /
                    ( O )                     ( O )
                   =======                   =======
```

#### Link Lengths & Coordinate Offsets
* **Hip Abduction Offset ($L_{hip}$):** $48.0\text{ mm}$ (lateral distance from hip ab/ad roll axis to the hip flexion pitch plane).
* **Thigh Link Length ($L_1$):** $105.0\text{ mm}$ (center-to-center distance from hip pitch axis to knee pitch axis).
* **Shank / Calf Link Length ($L_2$):** $105.0\text{ mm}$ (center-to-center distance from knee pitch axis to wheel axle center).
* **Wheel Radius ($R_w$):** $32.5\text{ mm}$ ($65.0\text{ mm}$ outer diameter including TPU tread).
* **Chassis Dimensions:** Length = $320\text{ mm}$, Width = $160\text{ mm}$ (inner frame) / $256\text{ mm}$ (hip pivot to hip pivot), Height = $85\text{ mm}$.
* **Wheelbase ($L_{wb}$):** $280\text{ mm}$ (front axle to rear axle in nominal stance).
* **Track Width ($W_{tr}$):** $256\text{ mm}$ (lateral distance between left and right wheel contact centerlines).

#### Range of Motion (ROM) & Joint Envelopes
| Joint Name | Anatomical Equivalent | Axis of Rotation | Min Angle | Max Angle | Total ROM | Nominal Stance |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`hip_roll`** | Hip Abduction/Adduction | X (Forward) | $-30.0^\circ$ ($-0.52\text{ rad}$) | $+30.0^\circ$ ($+0.52\text{ rad}$) | $60.0^\circ$ | $0.0^\circ$ |
| **`hip_pitch` (Front / Rear)**| Hip Flexion/Extension | Y (Lateral Left) | $-90.0^\circ$ / $-90.0^\circ$ | $+90.0^\circ$ / $+90.0^\circ$ | $180.0^\circ$ | $+45.0^\circ$ / $-45.0^\circ$ |
| **`knee_pitch` (Front / Rear)**| Knee Flexion/Extension | Y (Lateral Left) | $-150.0^\circ$ / $0.0^\circ$ | $0.0^\circ$ / $+150.0^\circ$ | $150.0^\circ$ | $-90.0^\circ$ / $+90.0^\circ$ |
| **`wheel_rev`**| Wheel Rolling Axis | Y (Lateral Left) | $-\infty$ | $+\infty$ | Continuous | Velocity Controlled |

#### Ground Clearance Envelopes
1. **Fully Extended (Standing Stork Pose):**
   $$Z_{max} = L_1 + L_2 + R_w = 105.0 + 105.0 + 32.5 = 242.5\text{ mm}$$
   Belly clearance: $\approx 185\text{ mm}$.
2. **Nominal Crouched Stance ($|q_2| = 45^\circ, |q_3| = 90^\circ$):**
   $$h_{nom} = L_1 \cos(45^\circ) + L_2 \cos(45^\circ) + R_w = 105(0.7071) + 105(0.7071) + 32.5 = 181.0\text{ mm}$$
   Belly clearance: $\approx 125\text{ mm}$.
3. **High-Speed Wheeled Cruise Pose ($|q_2| = 60^\circ, |q_3| = 120^\circ$):**
   $$h_{cruise} = 105 \cos(60^\circ) + 105 \cos(60^\circ) + 32.5 = 52.5 + 52.5 + 32.5 = 137.5\text{ mm}$$
   Belly clearance: $\approx 80\text{ mm}$. Low center of gravity prevents roll-over during $1.0\text{ m/s}$ cornering.
4. **Minimum Clearance (Full Leg Retraction / Transport):**
   $$h_{min} \approx 65.0\text{ mm}$$
   Belly clearance: $\approx 15\text{ mm}$ (skid plates touch ground).

---

### 1.2 Mathematical Kinematics Formulation

#### Forward Kinematics (FK)
Given joint angles $\mathbf{q} = [q_1, q_2, q_3]^T$ and lateral hip offset $s \cdot L_{hip}$ ($s=+1$ Left, $s=-1$ Right):
1. In the leg pitch plane ($\theta_{23} = q_2 + q_3$):
   $$\begin{aligned}
   x_{leg} &= L_1 \sin(q_2) + L_2 \sin(q_2 + q_3) \\
   z_{leg} &= -L_1 \cos(q_2) - L_2 \cos(q_2 + q_3)
   \end{aligned}$$
2. Projected across Hip Roll $q_1$:
   $$\begin{aligned}
   x_w &= x_{leg} = L_1 \sin(q_2) + L_2 \sin(q_2 + q_3) \\
   y_w &= s \cdot L_{hip} \cos(q_1) - z_{leg} \sin(q_1) \\
   z_w &= s \cdot L_{hip} \sin(q_1) + z_{leg} \cos(q_1)
   \end{aligned}$$

#### Analytical Inverse Kinematics (IK)
Given target foot position $\mathbf{P}_{target} = [x, y, z]^T$ relative to hip origin ($k_{knee} = -1$ for front legs bending back, $k_{knee} = +1$ for rear legs bending forward):
1. **Hip Roll ($q_1$):**
   $$R_{yz} = \sqrt{y^2 + z^2}$$
   $$q_1 = \text{atan2}(y, -z) - \text{atan2}(s \cdot L_{hip}, \sqrt{R_{yz}^2 - L_{hip}^2})$$
2. **Effective Planar Reach ($D$):**
   $$z_{plane} = -\sqrt{R_{yz}^2 - L_{hip}^2}, \quad x_{plane} = x, \quad D = \sqrt{x_{plane}^2 + z_{plane}^2}$$
3. **Knee Flexion ($q_3$):**
   $$q_3 = k_{knee} \cdot \arccos\left(\text{clamp}\left(\frac{D^2 - L_1^2 - L_2^2}{2 L_1 L_2}, -1.0, 1.0\right)\right)$$
4. **Hip Pitch ($q_2$):**
   $$\alpha = \text{atan2}(x_{plane}, -z_{plane}), \quad \beta = \arccos\left(\frac{L_1^2 + D^2 - L_2^2}{2 L_1 D}\right), \quad q_2 = \alpha - k_{knee} \cdot \beta$$

#### Dynamic Ground Reaction Torque Verification
In dynamic 2-leg trot support ($m=4.8\text{ kg}$), peak vertical reaction force per leg is $F_z \approx 49\text{ N}$ ($2\times$ static). With a $55\text{ mm}$ knee lever arm in nominal stance, peak knee torque is $\tau_{knee,peak} \approx 2.7\text{ N}\cdot\text{m}$ ($\sim 1.85\text{ N}\cdot\text{m}$ RMS during stance).
Since the Feetech STS3250's continuous thermal torque rating is $\sim 1.02\text{ N}\cdot\text{m}$ ($\sim 21\%$ of stall), the **2:1 GT2 timing belt reduction** (95% efficiency) boosts continuous joint output to **$1.94\text{ N}\cdot\text{m}$** ($9.8\text{ N}\cdot\text{m}$ stall), keeping steady-state servo winding temperature safely at $\sim 56^\circ\text{C}$ (see [`02_actuation_and_sizing.md`](./02_actuation_and_sizing.md)).

---

### 1.3 Step-Climbing Kinematics (Stage 1 vs Stage 3)

```
        STEP CLIMBING PROFILE: 6.0 cm OBSTACLE (STAGE 1 QUARTER-SCALE)
        
                  Phase 1: Rear Tilt & Front Lift    Phase 2: Front Wheel Touchdown & Pull
               +-----------------------------+     +-----------------------------+
               | CoM shifted backward        |     | Front wheels torque up      |
               | Front leg clears 85 mm apex |     | Rear legs push upward       |
               +-----------------------------+     +-----------------------------+
                                                             ___
                                                            |   |  Step Surface (Z = +60 mm)
                                                            |   |
                                       ( O ) Swing Foot     |   |
                     +-------------------+  ^               |   |
                     |     CHASSIS       |  | Z = +85 mm    |   |
                     +-------------------+  v               |   |
                    /                                       |   |
                   /                                        |   |
                 ( O ) Rear Support                         +---+ Ground (Z = 0 mm)
               =============================================+
```

* **Quarter-Scale Obstacle Mechanics ($H_{step} = 60\text{ mm}$):**
  A $65\text{ mm}$ wheel has $R_w = 32.5\text{ mm}$. Because $H/R_w = 1.846 > 1.0$, direct rolling over a vertical step is physically impossible due to backward reaction vectors. The robot executes a hybrid stepping trajectory:
  1. Pitch bias $+12^\circ$ rearward.
  2. Front swing leg reaches apex $Z_{apex} = 60\text{ mm} + 25\text{ mm} = 85\text{ mm}$ via leg contraction ($q_2 = +65^\circ, q_3 = -135^\circ$).
  3. Foot places forward $\Delta X = +75\text{ mm}$ onto step ledge.
  4. 25GA-370 gearmotors drive active traction on TPU 95A high-friction tread ($\mu \ge 0.85$) while rear legs push.
  5. Rear legs lift sequentially over the riser.
* **Scaling to Stage 3 Full-Scale ($H_{step} = 250 - 300\text{ mm}$):**
  With $L_1 = 400\text{ mm}$, $L_2 = 420\text{ mm}$, and $D_w = 200\text{ mm}$, maximum vertical extension is $920\text{ mm}$. Standard architectural steps ($175-190\text{ mm}$ risers) are effortlessly traversed with $> 450\text{ mm}$ apex clearance.

---

## 2. Autodesk Fusion & Blender to URDF / Simulation Pipeline

### 2.1 Complete Step-by-Step Fusion 360 Export Workflow

1. **Assembly Structuring in Fusion 360:**
   * Ground `base_link` component.
   * Model all 16 joints as explicit Fusion **Revolute Joints** aligned with exact rotation axes (Z-axis of joint origin).
   * Define hard software limits directly in Joint limits.
2. **Assign Physical Materials & Inertias:**
   * 6061-T6 Aluminum ($2.70\text{ g/cm}^3$) for plates; PAHT-CF ($1.18\text{ g/cm}^3$) for printed links; TPU 95A ($1.21\text{ g/cm}^3$) for tires.
   * Extract Mass $m$, Center of Mass $\mathbf{c}$, and $3 \times 3$ Moments of Inertia $\mathbf{I}_{CoM}$.
3. **Mesh Export to Blender for Decimation:**
   * Export visual meshes as STEP/STL into Blender.
   * Apply **Decimate Modifier** (`Ratio: 0.08 - 0.12`). Target: $3,000 - 6,000$ triangles per link for visual fidelity.
   * For collision meshes: Run **V-HACD** convex decomposition (max 4 convex hulls per link, $< 200$ triangles total).
   * For wheels: Replace mesh collision in simulation with exact analytical cylinders (`size="0.0325 0.012"`).

### 2.2 Coordinate Frames & Joint Naming (REP-103)
* **Axes:** X = Forward, Y = Left, Z = Up.
* **Joints:** `[FL/FR/RL/RR]_hip_roll_joint`, `[FL/FR/RL/RR]_hip_pitch_joint`, `[FL/FR/RL/RR]_knee_pitch_joint`, `[FL/FR/RL/RR]_wheel_joint`.
* **Links:** `base_link`, `[prefix]_hip_link`, `[prefix]_thigh_link`, `[prefix]_shank_link`, `[prefix]_wheel_link`.

---

## 3. Structural Design & Materials Trade-off

```
+---------------------------------------------------------------------------------------+
|                    STRUCTURAL DIVISION: HYBRID CHASSIS TOPOLOGY                       |
+---------------------------------------------------------------------------------------+
|                                                                                       |
|      +-------------------------------------------------------------------+            |
|      | TOP CHASSIS DECK: CNC 6061-T6 Aluminum (1.5 mm Plate Stock)       |            |
|      | - Carries battery & payload tie-downs; high in-plane shear plate  |            |
|      +---------------------------------+---------------------------------+            |
|                                        | Standoff Columns (M3 Alum)                   |
|      +---------------------------------+---------------------------------+            |
|      | 3D PRINTED INNER BULKHEADS: FDM PA-CF (Bambu PAHT-CF)             |            |
|      | - Absorbs electronics vibration, routes wiring, cradles battery   |            |
|      +---------------------------------+---------------------------------+            |
|                                        |                                              |
|      +---------------------------------+---------------------------------+            |
|      | BOTTOM CHASSIS DECK: CNC 6061-T6 Aluminum (2.5 mm Plate Stock)    |            |
|      | - Primary structural spine; mounts 4x Hip Ab/Ad bearing housings |            |
|      +-------------------------------------------------------------------+            |
|                                   |                 |                                 |
|            +----------------------+                 +----------------------+          |
|            v                                                               v          |
|  +-----------------------------+                             +-----------------------------+
|  | THIGH LINK: FDM PA-CF       |                             | SHANK LINK: FDM PA-CF       |
|  | - Boxed hollow beam truss   |                             | - Dual I-beam clevis        |
|  | - Integrates STS3250 servo  |                             | - Houses 25GA-370 gearmotor |
|  | - GT2 belt idler channel    |                             | - Dual 686-2RS wheel bearings|
|  +-----------------------------+                             +-----------------------------+
+---------------------------------------------------------------------------------------+
```

### 3.1 Material Allocation Matrix
* **Bottom Chassis Deck (CNC 6061-T6, 2.5 mm):** High bending and torsion resistance across hip modules.
* **Top Chassis Deck (CNC 6061-T6, 1.5 mm):** Forms torsion box with bottom deck via aluminum standoffs; secures payload.
* **Thigh Link (Bambu PAHT-CF Nylon):** Boxed truss housing the STS3250 servo and GT2 belt slot.
* **Shank / Calf Link (Bambu PAHT-CF Nylon):** High-aspect I-beam clevis; houses 25GA-370 motor and wheel bearings.
* **Wheel Core (Bambu PAHT-CF):** Rigid structural hub with captive M3 brass inserts.
* **Tire Tread (TPU 95A):** Airless compliant lattice absorbing 3G landing shocks with $\mu \ge 0.85$.

### 3.2 Bearing Selection & Servo Spline Load Isolation
* **Core Rule:** The Feetech STS3250 output spline must NEVER take radial or axial shear loads.
* **Straddle Bearing Architecture:**
  - Hip Ab/Ad: Dual **6802-2RS** bearings ($15\text{ mm ID} \times 24\text{ mm OD} \times 5\text{ mm W}$) with hollow aluminum tube for wiring pass-through.
  - Hip Pitch & Knee: Dual **6800-2RS** bearings ($10\text{ mm ID} \times 19\text{ mm OD} \times 5\text{ mm W}$).
  - Wheel Axle: Dual **686-2RS** bearings ($6\text{ mm ID} \times 13\text{ mm OD} \times 5\text{ mm W}$).

### 3.3 Knee 2:1 GT2 Timing Belt Sizing
* Pitch: $2.0\text{ mm}$ (GT2). Width: $6.0\text{ mm}$.
* Driver Pulley: 16T ($pcd = 10.19\text{ mm}$).
* Driven Pulley: 32T ($pcd = 20.37\text{ mm}$).
* Belt: 100T closed loop ($200.0\text{ mm}$ pitch length) at nominal center distance $C \approx 75.8\text{ mm}$.
* Integrated M3 slotted jack-screw tensioner for resonance tuning at $65-75\text{ Hz}$.

---

## 4. Carbon Fiber Nylon (PA-CF / PAHT-CF) Printing Guidelines

### 4.1 Production Slicing Profiles (Bambu Lab H2D / QIDI Plus4)
* **Nozzle:** Hardened Steel / Tungsten Carbide, $0.6\text{ mm}$ (preferred) or $0.4\text{ mm}$.
* **Layer Height:** $0.20 - 0.22\text{ mm}$ ($0.6\text{mm}$ nozzle) / $0.18\text{ mm}$ ($0.4\text{mm}$).
* **Wall Loops:** **6 Walls Minimum** (structural parts are quasi-solid).
* **Infill:** **50% Gyroid** or 3D Cubic (isotropic shear resistance).
* **Temperatures:** Nozzle $285^\circ\text{C} - 295^\circ\text{C}$; Bed $95^\circ\text{C} - 100^\circ\text{C}$; Chamber $55^\circ\text{C} - 65^\circ\text{C}$.
* **Cooling Fan:** **0% (Disabled)**. Part cooling causes immediate inter-layer cleavage failure.

### 4.2 Annealing & Post-Processing
* Dry filament at $85^\circ\text{C}$ for 10 hours prior to print.
* Post-print thermal annealing in convection oven:
  1. Ramp to $110^\circ\text{C} - 115^\circ\text{C}$ at $1^\circ\text{C/min}$.
  2. Hold for 6 to 8 hours (secondary polymer recrystallization).
  3. Furnace cool slowly ($< 0.5^\circ\text{C/min}$) to ambient room temperature.
  Increases HDT to $> 190^\circ\text{C}$ and interlaminar shear strength by up to 35%.

---

## 5. Leg & Foot Wheel Mechanism

```
                   FOOT & WHEEL MOTOR INTEGRATION (CROSS-SECTION)
                   
          Shank PA-CF Fork Lower Leg
         +---------------------------------------------+
         |                                             |
         |  [25GA-370 12V DC Gearmotor]                |
         |  - Outer Dia: 25 mm                         |
         |  - Gear Ratio: ~34:1 (300 RPM)              |
         |  - Integrated Hall Quadrature Encoder       |
         |  +--------------------+                     |
         |  | D-Cut Motor Shaft  |                     |
         |  +---------+----------+                     |
         |            |                                |
         |     [Brass Hex Coupler]                     |
         |            |                                |
         |      +-----+------+                         |
         |      | 686-2RS Brg|                         |
         |      +-----+------+                         |
         |            |                                |
         |      +-----+--------------------------+     |
         |      | PA-CF 50mm Structural Rim Hub  |     |
         |      +-----+--------------------------+     |
         |            |                                |
         |      +-----+--------------------------+     |
         |      | Compliant TPU 95A Airless Tire |     |
         |      | (OD = 65 mm, Width = 24 mm)    |     |
         |      +--------------------------------+     |
         +---------------------------------------------+
```

* **Motor Selection:** 25GA-370 12V 300 RPM geared DC motor ($m \approx 95\text{ g}$).
* **Wheel Speed:** $\omega = 300\text{ RPM} = 31.4\text{ rad/s} \implies v = \omega R_w = 31.4 \times 0.0325 \approx \mathbf{1.02\text{ m/s}}$ cruise speed.
* **Tire Construction:** 3D-printed TPU 95A with internal gyroid cell airless matrix and chevron knobby tread. Absorbs micro-vibrations, preventing encoder jitter.
* **Axle Retention:** Output shaft couples to a $12\text{ mm}$ aluminum hex drive supported by dual outboard **686-2RS** ball bearings pressed into the calf fork. Zero motor gearbox shaft bending.

---

## 6. Recommendations, BOM Checklist & Open Risks

### 6.1 Mechanical Sourcing & Hardware Checklist
* 12× Feetech STS3250 Smart Bus Servos (12V, 4.9 N·m, aluminum horns).
* 4× 25GA-370 12V 300 RPM Gearmotors with Hall Encoders.
* 4× GT2 100T 6mm timing belts + 4× 16T driver pulleys + 4× 32T driven pulleys.
* 8× 6802-2RS bearings (hip ab/ad).
* 16× 6800-2RS bearings (hip pitch and knee).
* 8× 686-2RS bearings (wheel axles).
* 100× M3 knurled brass heat-set inserts ($4.6\text{ mm OD} \times 5.0\text{ mm L}$).
* 50× M2.5 brass heat-set inserts (servo faces).
* 1× 6061-T6 Aluminum sheet ($300 \times 400 \times 2.5\text{ mm}$) for CNC bottom deck.
* 1× 6061-T6 Aluminum sheet ($300 \times 400 \times 1.5\text{ mm}$) for CNC top deck.
* 2× 1 kg Spools Bambu PAHT-CF or Polymaker Fiberon PA6-CF20 filament.
* 1× 1 kg Spool Bambu TPU 95A filament.

### 6.2 Key Risks & Mitigations
1. **Risk:** Servo thermal throttling under continuous crouching.  
   *Mitigation:* 2:1 GT2 reduction keeps holding torque $< 1.8\text{ N}\cdot\text{m}$ (under continuous thermal threshold). Standby postures use mechanically locked knees ($q_3 = -90^\circ$).
2. **Risk:** PA-CF moisture absorption altering dimensional tolerances in Mexico humidity.  
   *Mitigation:* Dry parts at $80^\circ\text{C}$ before final assembly; calibrate hole X/Y compensation by $+0.12\text{ mm}$ in slicer for bearing press-fits.
3. **Risk:** Step climbing dynamic wheel slip.  
   *Mitigation:* Compliant TPU chevron tread provides mechanical interlock over concrete curbs and 6 cm step lips.
