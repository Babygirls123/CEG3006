# Hardware Components and Parameters

This file summarises the assumed hardware components and example system parameters used in the AViRS-V2P design.

---

## 1. Hardware Components

### 1.1 Vehicle Side

| Component | Purpose |
|---|---|
| GNSS receiver | Provides vehicle position and speed |
| Ambient light sensor or camera | Detects lighting level or night conditions |
| Rain sensor or wiper-state input | Estimates rain intensity |
| Camera-based visibility estimator | Detects fog or reduced visibility |
| Risk computation module | Computes Visibility Index, TTC threshold, and safety radius |
| Driver alert interface | Dashboard warning or audible alert |
| V2X On-Board Unit (OBU) | Broadcasts V2P safety messages using DSRC or C-V2X |

---

### 1.2 Pedestrian Side (Smartphone)

| Component | Purpose |
|---|---|
| Smartphone GNSS | Provides pedestrian position |
| Inertial sensors (accelerometer / IMU) | Detects pedestrian movement |
| V2P communication receiver | Receives AViRS safety messages |
| TTC estimation module | Estimates collision risk |
| Alert engine | Triggers audio, vibration, and visual warnings |

---

### 1.3 Alternative Pedestrian Device (Reflective Smart Tag)

| Component | Purpose |
|---|---|
| Low-power wireless beacon | Periodically advertises pedestrian presence |
| Warning receiver | Receives simplified warning signal from vehicle |
| Vibration motor | Provides tactile warning |
| LED indicator | Provides visual warning |
| Small battery or power module | Powers the wearable device |

---

## 2. Example System Parameters

These are example engineering assumptions for the prototype design.

| Parameter | Symbol | Example Value | Notes |
|---|---|---:|---|
| Default TTC threshold | T_default | 2.0 s | Used in clear conditions |
| Maximum TTC threshold | T_max | 4.0 s | Used in worst visibility conditions |
| Broadcast rate range | f_tx | 1–10 Hz | Adaptive transmission rate |
| Safety radius range | R_b | 30–90 m | Alert distance or safety bubble |
| Message size | S_msg | ~200 bytes | Approximate AViRS message size |
| Target latency | L_target | <100 ms | Desired end-to-end warning latency |
| Message stale limit | T_stale | 1–2 s | Old packets are ignored |
| Prediction horizon | H_pred | 3–5 s | Time horizon for TTC estimation |
| Alert cooldown | T_cooldown | 1–2 s | Prevents repeated alert spam |

---

## 3. Environmental Parameter Ranges

These values are example input ranges used to compute the Visibility Index.

| Parameter | Symbol | Unit | Example Range | Meaning |
|---|---|---|---|---|
| Ambient light | L_raw | lux | 0–10000 | Higher value = brighter environment |
| Rain intensity | R_raw | mm/hour or proxy level | 0–10+ | Higher value = heavier rain |
| Visibility distance | D_vis | m | 20–200+ | Lower value = denser fog or lower visibility |

---

## 4. Visibility Index Parameters

The Visibility Index combines normalized environmental factors into a single score.

### Formula

```math
V = w_L L + w_R (1 - R) + w_F (1 - F)
