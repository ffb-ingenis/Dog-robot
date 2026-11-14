# Project 2 — Edge AI, Vision, RL & Bilingual HRI (`robot/02_half_scale_jetson_oakd`)

## 1. Compute & Vision Hardware Stack

Project 2 moves from basic Raspberry Pi telemetry to real onboard **Edge AI** using the **NVIDIA Jetson Orin Nano Super Developer Kit (8 GB, 67 TOPS INT8, `$249`)** paired with a **Luxonis OAK-D Lite (`$149`, USB-C)** stereo depth + VPU camera.

| Subsystem | Selected Hardware | Key Role & Justification |
| :--- | :--- | :--- |
| **Onboard AI Computer** | **Jetson Orin Nano Super (8 GB, 25 W Super mode)** | Runs JetPack 6 (Ubuntu 22.04), ROS 2 Humble, TensorRT RL locomotion policy, Whisper ASR + small local LLM intent parser, and Piper TTS |
| **Vision & Depth Sensor** | **Luxonis OAK-D Lite (USB-C)** | On-camera Myriad X VPU executes **YOLO person/obstacle detection + stereo depth map** (`~0` Jetson GPU load), leaving Orin Nano's 8 GB unified RAM free for speech + RL |
| **Primary Control Link** | **RadioMaster ELRS 2.4 GHz RX** (`CRSF` → Teensy) | Mandatory manual override, E-stop channel, stair-assist speed cap, and mode switch |
| **Real-Time Low-Level MCU** | **Teensy 4.1 + ICM-42688-P SPI IMU** | Executes `500 Hz` CAN-FD MIT torque loop, E-stop / BMS / drive-away interlock watchdog |
| **Optional Lab Training Rig** | **NVIDIA Jetson AGX Thor** *(already owned)* | Used **offboard** in workshop to train RL policies (`Isaac Lab` / `MuJoCo Playground`), quantize Whisper/LLM weights, and benchmark Vision-Language / navigation stacks before deploying TensorRT engines to Orin Nano |
| **Audio Frontend** | **ReSpeaker 4-Mic Array v2.0** on `120 mm` mast | Shore 40A elastomer isolation + `35 PPI` windscreen; outputs hardware AEC/beamformed `16 kHz` PCM over USB to Orin Nano |

> [!NOTE]
> **Why OAK-D Lite over Raspberry Pi Camera Module 3 (`IMX708`) on Jetson:** Jetson Orin Nano natively prioritizes IMX219/IMX477 CSI drivers; `IMX708` requires out-of-tree kernel patches and provides monocular RGB only. **OAK-D Lite** works plug-and-play over USB-C in ROS 2 (`depthai-ros`), supplies metric depth for step/curb detection, and runs YOLOv8n/YOLOv6n right on its VPU.

---

## 2. Software Architecture (`ROS 2 Humble` + `500 Hz` Teensy Bridge)

```text
  [ReSpeaker USB Audio]     [OAK-D Lite USB-C]        [ELRS 2.4GHz CRSF]
           |                        |                          |
           v                        v                          v
  +------------------+   +--------------------+       +-----------------+
  | openWakeWord     |   | DepthAI VPU Node   |       | Teensy 4.1 MCU  |
  | -> Whisper GPU   |   | (Depth + YOLO Det) |       | (500Hz CAN-FD)  |
  | -> Local LLM     |   +---------+----------+       +--------+--------+
  |    JSON Parser   |             |                           ^
  +--------+---------+             v                           |
           |             +--------------------+                |
           +------------>| Command Arbiter &  |--- USB-HS ---->+
                         | Safety Stop Node   |    500 Hz      |
                         +---------+----------+                |
                                   |                           |
                                   v                           |
                         +--------------------+                |
                         | RL Policy (TRT) +  |----------------+
                         | Wheel-Leg Planner  |
                         +--------------------+
```

### Priority & Safety Hierarchy (Strict Non-Overridable Order)

1. **Hardware Latching Mushroom E-Stop & Drive-Away Interlock** (cuts BLDC gate/bus relay in `< 5 ms`, independent of software).
2. **ELRS RC Override Channel** (`SF` arm switch + `SA` mode switch + `CH1–CH4` sticks always override autonomous/voice velocity within `10 ms`).
3. **OAK-D Stereo + YOLO Safety Stop** (halts forward velocity if uncommanded person or drop-off/cliff `< 0.8 m` ahead during voice-commanded movement).
4. **Bilingual Voice Intent (`EN / ES`) & Optional "Follow Me"** (only active when armed in Voice/Follow mode via RC switch and cleared by safety layer).

---

## 3. Practical Edge AI Pipeline (Fitting Comfortably in `8 GB` Unified RAM)

On Jetson Orin Nano Super (8 GB unified CPU/GPU RAM, ~6.2 GB usable after JetPack OS overhead), run lightweight FP16/INT8 engines so nothing triggers OOM swaps:

| Workload | Model / Runtime | Footprint (Approx.) | Latency Target | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Obstacle + Person Detection + Depth** | YOLOv8n / Depth on **OAK-D Lite VPU** | `~150 MB` host ROS node | `30 FPS` (`~33 ms`) | **Offloaded to OAK-D**—zero Orin GPU compute used |
| **RL Locomotion / Step-Climb Policy** | Compact MLP / Actor-Critic (`256×128×64` or LSTM) exported to **TensorRT FP16** | `~120 MB` | `< 2 ms` per `100–200 Hz` step | Reads IMU, `16×` joint positions/velocities, heightmap/command vector; outputs joint position + wheel velocity targets to Teensy `500 Hz` PD loop |
| **Wake Word + Fast Safety Stop** | `openWakeWord` (CPU) + acoustic keyword spotter (`Stop` / `Alto` / `Para`) | `~80 MB` | `< 45 ms` | Immediate brake without waiting for LLM |
| **Bilingual Speech-to-Text (EN/ES)** | `faster-whisper` (`small` or `base` **INT8** via CTranslate2 / TensorRT) on GPU | `~600–900 MB` | `~250–450 ms` | Runs on demand after wake word or push-to-talk |
| **Local Intent Parser (EN/ES → JSON)** | Quantized `Qwen2.5-1.5B-Instruct` or `Llama-3.2-1B` (`Q4_K_M` via `llama.cpp` CUDA) with strict GBNF grammar | `~1.0–1.2 GB` | `~150–300 ms` | Maps freeform EN/ES ("tráeme la caja despacio", "climb step", "follow me") into strict `{action, speed, duration, safety_confirm}` schema—never unconstrained navigation |
| **Bilingual Voice Confirmation** | **Piper TTS** (`en_US-lessac-medium` / `es_MX-claude-high`) | `~150 MB` | `< 120 ms` | Confirms hazardous actions ("Powering 12V igniter" / "Siguiendo al operador") |
| **System + ROS 2 + Headroom** | Ubuntu 22.04, ROS 2 Humble, buffers | `~2.0–2.5 GB` | — | **~1.5 GB headroom remaining** without zram thrashing |

---

## 4. Training Workflow on Jetson AGX Thor → Orin Nano Deployment

1. **Sim-to-Real Training (Offboard on Thor / Workstation):**
   - Export URDF/MJCF from Autodesk Fusion 360 + Blender into **Isaac Lab** or **MuJoCo Playground**.
   - Apply **domain randomization**: payload mass `0–12 kg`, friction `0.4–1.1`, joint backlash/damping variance, `10–25 ms` telemetry latency, and step heights `4–16 cm`.
   - Train symmetric `< >` X-configuration policies for three gait skills:
     1. *Wheel-cruise* (fast energy-efficient rolling with active suspension compliance).
     2. *Quasi-static curb step* (shift CoG back, lift front wheel onto `15 cm` ledge, pull body forward, lift rear wheel).
     3. *Recovery / Squat* (kneel low for cargo loading or battery swap).
2. **ONNX → TensorRT Compilation:**
   - Export trained policy to ONNX and compile `trtexec --fp16` **directly on the Orin Nano Super** so TensorRT kernels match Ampere SM `8.7` (`1024` CUDA cores).
3. **"Follow Me" Visual Mode (Voice-Toggled Only):**
   - Uses OAK-D person bounding box + stereo depth `(X_person, Z_person)` fed to a damped pure-pursuit follower (`v_x, omega_z`) capped at `0.8 m/s`, holding `1.5 m` standoff distance and auto-stopping if operator turns away out of FOV or another obstacle enters `< 0.7 m`.
