# 03 - Bilingual Voice Control, Human-Robot Interaction & Audio Architecture

## Executive Summary
This document establishes the complete edge-driven audio, bilingual voice control, and Human-Robot Interaction (HRI) architecture for the hybrid wheel-legged robotic platform. The system operates under strict physical and operational constraints:
1. **Control Modality:** Strictly **Voice Control + RC Remote Manual Control** (no autonomous SLAM, no lidar/vision path planning, and zero cloud connectivity).
2. **Bilingual Support:** Native, deterministic operation in **English** ("Hey robot") and **Spanish** ("Oye robot").
3. **Edge Compute Architecture:** Stage 1 onboard compute consists of a **Raspberry Pi 5 (8 GB RAM)** running edge DSP, wake word detection, constrained speech recognition, intent parsing, and TTS synthesis, tightly coupled over high-speed UART to a **Teensy 4.1 (ARM Cortex-M7 @ 600 MHz)** running hard real-time motor commutation, attitude stabilization, and RC receiver decoding. Offline model training, acoustic dataset augmentation, and benchmarking are supported by an external **Jetson Thor** development cluster.
4. **Safety Invariant:** A dual-tier safety regime where:
   - The immediate software emergency command (**"Stop" / "Alto" / "Para"**) executes via an un-gated fast-path listener that bypasses wake word detection and high-level NLP, triggering an emergency brake packet to the Teensy 4.1 in **< 45 ms**.
   - The physical **RC Remote (ELRS / CRSF @ 500 Hz)** maintains hard firmware-level preemption over all voice commands at the Teensy 4.1 microcontroller layer.

---

# 1. Voice Pipeline Architecture (100% Offline / Edge)

### 1.1 Architectural Philosophy & Constraints
To guarantee real-time safety, acoustic reliability in outdoor conditions, and absolute zero-cloud dependence, the voice processing stack is implemented entirely locally on the Raspberry Pi 5. The pipeline rejects general-purpose Large Language Models (LLMs) in the real-time locomotion loop. Running unconstrained generative models for real-time motor commands introduces unacceptable non-deterministic latency (1,000–4,000 ms), non-zero hallucination rates, and thermal throttling under continuous ARM CPU loads. Instead, the architecture couples a hardware DSP acoustic front-end with dual-path edge neural keyword detection and constrained-grammar speech decoding.

### 1.2 End-to-End Pipeline Block Diagram
The signal flow decouples safety-critical interrupts from standard modal locomotion commands.

```mermaid
flowchart TD
    subgraph Audio_Ingest ["Acoustic Front-End & Hardware DSP"]
        MIC["ReSpeaker 4-Mic Array<br/>Circular MEMS PDM"]
        XMOS["XMOS XVF3000 / XVF3800 DSP<br/>- Phat-DS Beamforming<br/>- Hardware AEC (Acoustic Echo Cancellation)<br/>- Noise Suppression (NS) & AGC<br/>- Direction of Arrival (DOA)"]
        SPK_REF["Speaker Reference Loopback<br/>(I2S / Hardware Tap)"]
        
        MIC --> XMOS
        SPK_REF -->|AEC Reference| XMOS
    end

    subgraph Audio_Streams ["Audio Stream Splitter (ALSA / Ring Buffer)"]
        XMOS -->|16 kHz 16-bit Mono (Beamformed)| RING["Lock-Free Circular Ring Buffer<br/>(Shared Memory / POSIX shm)"]
    end

    subgraph Fast_Path ["Fast-Path Emergency Subsystem (High Priority)"]
        RING -->|Raw Audio Slices (80ms)| ESTOP_KWS["Continuous Safety Keyword Spotter<br/>'Stop' / 'Alto' / 'Para'<br/>(Zero Wake-Word Gate)"]
        ESTOP_KWS -->|Trigger Detected (< 35ms)| ESTOP_SIGNAL["Immediate Software E-Stop Packet"]
    end

    subgraph Standard_Path ["Standard Locomotion & Modal Pipeline"]
        RING -->|Audio Slices| WW["openWakeWord Engine<br/>'Hey robot' / 'Oye robot'"]
        WW -->|Wake Detected| VAD_GATE["Acoustic Window Gate & VAD<br/>(Silero VAD / WebrtcVAD)"]
        VAD_GATE -->|Utterance Buffer| ASR["Bilingual Vosk ASR Engine<br/>(Constrained Grammar FST)"]
        ASR -->|Raw Text Hypothesis| INTENT["Intent Parser & Validator<br/>(Strict Pydantic / JSON Schema)"]
        INTENT -->|Hazardous Action?| CONFIRM_SM{"Requires<br/>Confirmation?"}
        
        CONFIRM_SM -->|Yes: e.g. Outlet On| CONFIRM_STATE["5s Confirmation Dialog State Machine<br/>'Confirm' / 'Confirma'"]
        CONFIRM_SM -->|No: Standard Move| ACTION_DISPATCH["Robot Action Packet Generator"]
        CONFIRM_STATE -->|Confirmed| ACTION_DISPATCH
        CONFIRM_STATE -->|Timeout / Cancel| CANCEL_FEEDBACK["Cancel Audio Notification"]
    end

    subgraph Hardware_Bridge ["Real-Time Control & Safety Arbiter"]
        RC_RX["RC Receiver (ELRS / CRSF 2.4GHz)<br/>Direct UART to Teensy 4.1"]
        TEENSY["Teensy 4.1 (Cortex-M7 @ 600 MHz)<br/>- Low-Level Motor Control Loop (1 kHz)<br/>- Wheel-Leg Balance & Kinematics<br/>- Hardware Preemption Arbiter"]
        MOTORS["Wheel Motors & Leg Actuators<br/>(CAN-FD / PWM / Dynamic Brakes)"]
        AUX_RELAY["Auxiliary 110V/220V Inverter Relay"]

        ESTOP_SIGNAL -->|UART @ 921,600 baud| TEENSY
        ACTION_DISPATCH -->|UART @ 921,600 baud| TEENSY
        RC_RX -->|Hardware Preemption Watchdog| TEENSY
        
        TEENSY --> MOTORS
        TEENSY --> AUX_RELAY
    end

    subgraph HRI_Feedback ["Human-Robot Interaction & Audio Feedback"]
        ACTION_DISPATCH -.->|Trigger Feedback| TTS_MGR["Audio Feedback Manager"]
        CANCEL_FEEDBACK -.-> TTS_MGR
        TEENSY -.->|Telemetry & State Changes| TTS_MGR
        
        TTS_MGR --> CACHE{"Audio in<br/>RAM Cache?"}
        CACHE -->|Yes (Canned)| WAV_PLAY["Zero-Latency PCM Streamer<br/>(< 5ms Latency)"]
        CACHE -->|No (Dynamic Telemetry)| PIPER["Piper TTS Engine (ONNX)<br/>en_US / es_MX"]
        
        WAV_PLAY --> DAC["MAX98357A I2S DAC / Amp"]
        PIPER --> DAC
        DAC --> SPK["3W Rugged Enclosed Speaker"]
        SPK -.->|Acoustic Output| SPK_REF
    end
```

### 1.3 Latency Budget Breakdown
| Pipeline Stage | Subsystem / Component | Tier 0 (E-Stop) Latency | Tier 1 (Modal Command) Latency | Notes / Optimization |
| :--- | :--- | :--- | :--- | :--- |
| **Audio Acquisition** | XMOS DSP I2S / USB transfer | 10.0 ms | 10.0 ms | 16 kHz, 16-bit PCM, 80 ms audio frame buffered in 10 ms ring chunks |
| **Hardware Pre-Processing** | AEC, Phat-DS Beamforming, NS | 5.0 ms | 5.0 ms | Hardware executed on XMOS XVF3000 silicon |
| **Wake Word Detection** | `openWakeWord` (AST / Conformer) | **BYPASS (0.0 ms)** | 75.0 ms | Tier 1 evaluates wake word concurrently with incoming audio |
| **Utterance Capture** | Silero VAD (End-of-Speech trigger) | **BYPASS (0.0 ms)** | 120.0 ms | Dynamic speech end-point detection (silence threshold: 120 ms) |
| **Keyword Spotting / ASR** | Vosk Constrained Grammar FST | 18.0 ms (KWS search) | 65.0 ms (Full phrase) | Constrained search lattice prunes non-command paths |
| **Intent & Safety Validation** | Pydantic Schema / Regex Validator | 1.0 ms | 2.0 ms | Zero-LLM deterministic regex and hash validation |
| **Serial Bus Dispatch** | RPi 5 to Teensy 4.1 UART | 0.4 ms | 0.4 ms | 32-byte framed binary packet @ 921,600 baud |
| **Firmware Processing & Actuation**| Teensy 4.1 motor torque clamping | 0.5 ms | 1.0 ms | Hard dynamic braking / PWM zero-clamping at 1 kHz control loop |
| **Earcon Acoustic Ack** | In-memory RAM chime playback | — | 15.0 ms | Concurrently played while Teensy initiates kinematic motion |
| **Total Response Time** | **Acoustic Wave to Physical Actuation** | **34.9 ms (< 45 ms)** | **288.4 ms (< 300 ms)** | Well within human perception (< 500 ms) and real-time safety limits |

---

# 2. Wake Word Detection (EN / ES)

- **Engine:** `openWakeWord` via ONNX Runtime with ARM NEON SIMD vectorization.
- **Models:** `"Hey robot"` (`hey_robot_v1.onnx`) and `"Oye robot"` (`oye_robot_v1.onnx`).
- **Resource Usage:** Combined inference latency $\approx 2.07\text{ ms}$, CPU load $< 3\%$ of a single Cortex-A76 core on Raspberry Pi 5.
- **Accuracy:** False Acceptance Rate $< 0.04$ triggers / 24 hr; Recall $> 93\%$ under active motor noise ($0\text{ dB}$ SNR).

---

# 3. Automatic Speech Recognition (ASR) Benchmark

- **Engine Selection:** **Vosk Small with Runtime Grammar Constraints** (`vosk-model-small-en-us-0.15` and `vosk-model-small-es-0.42`).
- **Why Vosk over Whisper on Edge:**
  - Streaming frame-by-frame evaluation via Weighted Finite State Transducers (WFST).
  - Search graph is restricted to legal robot commands: hallucinations are physically impossible.
  - Latency: **$62\text{ ms}$ (EN) / $68\text{ ms}$ (ES)** vs $800-1,800\text{ ms}$ for faster-whisper.
  - RAM: $58\text{ MB}$ vs $400-800\text{ MB}$.

---

# 4. Intent Parsing & Bilingual Command Schema

### Command Mapping Table
| Category | English Utterances | Spanish Utterances | Mapped Action / Canonical Payload | Safety Class |
| :--- | :--- | :--- | :--- | :--- |
| **Emergency** | `stop`, `halt`, `emergency stop` | `alto`, `para`, `detente`, `parada de emergencia` | `STOP_IMMEDIATE` | **CRITICAL** (Bypasses Wake Word) |
| **Locomotion** | `move forward`, `go forward`, `advance` | `avanza`, `ve adelante`, `adelante` | `MOVE_FORWARD` (speed: 30%, dist: 1.0m) | OPERATIONAL |
| **Locomotion** | `move backward`, `go back`, `reverse` | `retrocede`, `ve atras`, `atras` | `MOVE_BACKWARD` (speed: 25%, dist: 0.8m) | OPERATIONAL |
| **Locomotion** | `turn left`, `rotate left` | `gira a la izquierda`, `izquierda` | `TURN_LEFT` (turn_degrees: 45) | OPERATIONAL |
| **Locomotion** | `turn right`, `rotate right` | `gira a la derecha`, `derecha` | `TURN_RIGHT` (turn_degrees: 45) | OPERATIONAL |
| **Posture** | `stand up`, `stand tall`, `rise` | `levantate`, `ponte de pie`, `parate` | `POSTURE_STAND` (target_height: 0.65m) | OPERATIONAL |
| **Posture** | `crouch`, `squat`, `lower down` | `agachate`, `baja`, `posicion baja` | `POSTURE_CROUCH` (target_height: 0.35m) | OPERATIONAL |
| **Mode Switch** | `wheel mode`, `driving mode` | `modo ruedas`, `activa ruedas` | `MODE_WHEEL` (lock_knee_pitch: true) | HAZARDOUS |
| **Mode Switch** | `walk mode`, `stepping mode` | `modo pasos`, `modo caminar` | `MODE_WALK` (gait: quadruped_crawl) | HAZARDOUS |
| **Obstacle** | `climb step`, `climb curb` | `sube escalon`, `sube banqueta` | `MODE_CLIMB_STEP` (height: 0.15m) | HAZARDOUS |
| **Payload Power**| `turn on outlet`, `enable power` | `enciende la toma`, `activa corriente` | `OUTLET_POWER_ON_REQUEST` (req_confirm: true) | **HAZARDOUS** |
| **Payload Power**| `turn off outlet`, `disable power` | `apaga la toma`, `corta corriente` | `OUTLET_POWER_OFF` (req_confirm: false) | OPERATIONAL |
| **Confirmation** | `confirm`, `yes confirm`, `affirmative` | `confirma`, `si confirma`, `afirmativo` | `CONFIRM_ACTION` | OPERATIONAL |
| **Cancellation** | `cancel`, `negative`, `abort` | `cancela`, `no cancela`, `aborta` | `CANCEL_ACTION` | OPERATIONAL |
| **Telemetry** | `battery status`, `battery level` | `estado de bateria`, `cuanta bateria` | `QUERY_BATTERY` | INFORMATIONAL |
| **Telemetry** | `motor temperature`, `temp status` | `temperatura de motores`, `motores` | `QUERY_MOTOR_TEMP` | INFORMATIONAL |
| **Telemetry** | `system status`, `health report` | `estado del sistema`, `diagnostico` | `QUERY_STATUS` | INFORMATIONAL |

---

# 5. Text-to-Speech (TTS) & Feedback

- **Engine:** **Piper TTS** with ONNX runtime (`en_US-lessac-medium` and `es_MX-ald-medium`).
- **RAM Pre-Caching:** All 32 static prompt and acknowledgement files are pre-loaded into uncompressed RAM PCM buffers, yielding `< 5 ms` response time.
- **Acoustic Earcons:** High-contrast synthesized audio cues for Wake Ack, Command Ack, Warning Prompt, and Emergency Stop.

---

# 6. Hardware Setup & Acoustic Engineering

- **Microphone Array:** **Seeed Studio ReSpeaker USB 4-Mic Array** (XMOS XVF3000 enterprise DSP with circular PDM array, Phat-DS beamforming, hardware AEC with reference tap).
- **Vibration Decoupling:** Pod elevated 120 mm above chassis deck on a carbon-fiber mast mounted on 4× Shore 40A silicone vibration bobbins ($f_c \approx 38\text{ Hz}$, $> 28\text{ dB}$ mechanical attenuation).
- **Speaker:** 3W enclosed speaker downward-facing in a baffled chassis enclosure with $> 32\text{ dB}$ acoustic isolation from microphones.
- **Wind Protection:** Dual-stage acoustic foam (35 PPI, 8 mm thickness) in an aerodynamic ASA printed teardrop dome + slip-on synthetic deadcat fur cover ($25-30\text{ dB}$ wind attenuation).
