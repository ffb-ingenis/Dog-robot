# Wheel-Legged Outdoor Logistics Quadruped — Project Tree

This workspace organizes the **3-stage wheel-legged quadruped robot program** (Autodesk Fusion 360 + Blender CAD/sim, CNC 6061-T6 aluminum + carbon fiber + annealed PA-CF manufacturing, bilingual EN/ES voice + ELRS RC control, outdoor food & appliance logistics).

## Directory Structure

- [`shared/concept_design.md`](./shared/concept_design.md) — Locked visual/industrial concept design (**open-frame CNC 6061 + CF + PA-CF, front/rear roll bars, symmetric `< >` X-configuration legs, removable insulated box + lid-mounted quick-disconnect solar panel**) with exterior render & architectural blueprint.
- [`01_quarter_scale_rpi5/`](./01_quarter_scale_rpi5/README.md) — **Project 1 (Stage 1, ~$1.2k–$1.7k):** Quarter-scale (`4.8 kg` loaded, `1 kg` payload, `6 cm` step) bench & backyard validator using 12× STS3250 servos, Teensy 4.1, **Raspberry Pi 5 (8 GB) + Pi Camera Module 3 Wide**, Vosk EN/ES voice + ELRS RC, 3S4P 21700 battery, USB-C PD + 12 V spark igniter + 20 W solar demo.
- [`02_half_scale_jetson_oakd/`](./02_half_scale_jetson_oakd/README.md) — **Project 2 (Stage 2, ~$3.5k–$4.8k):** Half-scale (`30 kg` loaded, `10 kg` payload, `12–15 cm` step) Edge AI & QDD platform using **hybrid bought planetary hips + DIY CNC/PA-CF knees**, 6010 wheel drives, Teensy 4.1 dual CAN-FD, **Jetson Orin Nano Super (8 GB, 67 TOPS, $249) + Luxonis OAK-D Lite ($149)**, TensorRT RL policies (trained offboard on **Jetson AGX Thor**), on-VPU YOLO safety stop, GPU Whisper + local LLM intent parser, 48 V battery, and 300 W inverter demo.
- [`03_full_scale_thor/`](./03_full_scale_thor/README.md) — **Project 3 (Stage 3, ~$11k–$18k DIY):** Full-scale (`80 kg` payload, `~170 kg` loaded, `≥ 25–30 cm` steps, `4+ hr` endurance) field logistics robot running **Jetson AGX Thor** onboard, frameless BLDC + metal cycloidal/harmonic drives, `5.12 kWh` 48 V LFP battery, **`1,000 W` pure-sine AC inverter + GFCI (blender)**, 12 V igniter, USB-C PD, and `100–200 W` solar array.

## Immediate Next Execution Steps

1. Order/commission PA-CF 3D printer (`QIDI Plus4` ~$799 or `Bambu Lab H2D`) + Sunlu S4 dry box + local Mexico `6061-T6` plates & 21700 cells.
2. Build **Project 1 (`01_quarter_scale_rpi5`)** in Fusion 360 using the symmetric `< >` X-configuration kinematics and open-frame chassis + roll bars, validate CAD-to-MuJoCo export via Blender, and verify voice/RC/interlock safety on the bench.
3. Advance to **Project 2 (`02_half_scale_jetson_oakd`)** to qualify hybrid 48 V BLDC actuators, OAK-D Lite obstacle safety stops, and Jetson Thor → Orin Nano TensorRT sim-to-real walking/curb-climbing policies before scaling to Stage 3.
