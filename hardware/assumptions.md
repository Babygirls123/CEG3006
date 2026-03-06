
---

## `assumptions.md`

```md
# Assumptions

This file records the main design assumptions used in the AViRS-V2P project. These assumptions help keep the scope realistic and explain how the proposed system operates even though it is presented as a design concept rather than a fully deployed commercial implementation.

---

## 1. General Project Scope Assumptions

1. AViRS-V2P is presented as a **conceptual and protocol-level design** for a low-visibility Vehicle-to-Pedestrian (V2P) safety system.
2. The project focuses on **adaptive communication and adaptive risk scaling**, rather than full production-grade hardware implementation.
3. The system is intended to support **pedestrian warning and driver warning**, not to replace existing onboard safety systems such as AEB or camera-based pedestrian detection.
4. Example values shown in the README, parameters file, and message definitions are **engineering assumptions for demonstration purposes** and would require further calibration in a real deployment.
5. The system is designed mainly for **low-visibility scenarios**, such as night, heavy rain, and fog.

---

## 2. Vehicle-Side Assumptions

1. The vehicle is assumed to have access to:
   - GNSS position and speed,
   - heading information,
   - ambient light sensing,
   - rain estimation through rain sensor input or wiper-state proxy,
   - fog or visibility estimation using a camera-based method,
   - a V2X communication module.

2. Vehicle speed and heading are assumed to be available from onboard systems or a vehicle network such as CAN.
3. The rain level does not need to be measured as exact mm/hour in the prototype. It may be approximated using **wiper state or wiper speed**.
4. Fog level does not need to be measured using a dedicated fog sensor. It may be approximated using **camera-derived visibility distance or contrast reduction**.
5. The vehicle can compute:
   - Visibility Index,
   - adaptive TTC threshold,
   - adaptive broadcast rate,
   - adaptive safety radius,
   - risk level.

6. The vehicle is assumed capable of triggering a **driver-facing warning** when risk becomes critical.

---

## 3. Pedestrian Smartphone Assumptions

1. The pedestrian is assumed to carry a smartphone capable of:
   - receiving V2P messages,
   - obtaining GNSS position,
   - estimating movement using phone sensors,
   - generating audio, vibration, or visual alerts.

2. The smartphone does not need to perform heavy environmental sensing because the **vehicle already sends adaptive risk information**.
3. The pedestrian smartphone uses:
   - its own position and motion data,
   - the vehicle broadcast data,
   - TTC estimation logic,
   to determine whether a warning should be triggered.

4. Smartphone GNSS accuracy is assumed to be sufficient for a prototype-level collision warning concept, even though real-world urban environments may introduce positioning error.
5. The phone is assumed to reject stale messages using timestamps.

---

## 4. Reflective Smart Tag Assumptions

1. Some pedestrians may not carry smartphones, so the system may support an **advanced reflective smart tag**.
2. The smart tag is assumed to be:
   - low-power,
   - wearable,
   - capable of broadcasting a simple beacon,
   - capable of receiving a simplified warning signal.

3. The smart tag does not perform full TTC estimation.
4. Instead, the vehicle computes the main risk and sends a simpler warning command to the tag.
5. The tag is assumed to provide alerts through:
   - vibration,
   - LED flashing,
   - or both.

6. Example attachment points include:
   - white canes,
   - wheelchairs,
   - school bags,
   - jackets,
   - bicycles,
   - strollers.

---

## 5. Communication Assumptions

1. The communication technology is assumed to be **DSRC, ITS-G5, or C-V2X**, depending on deployment context.
2. The project does not require implementation of a specific physical-layer standard; it focuses on the **message logic and adaptive policy**.
3. Broadcast frequency is assumed to be adaptive within the approximate range of **1 Hz to 10 Hz**.
4. The wireless channel may become congested in dense traffic, so the system is assumed to support **congestion-aware rate limiting**.
5. The actual transmit rate may therefore be lower than the desired adaptive rate in order to avoid excessive channel load.
6. Message size is assumed to remain compact, around **200 bytes**, to reduce communication overhead.
7. End-to-end target latency is assumed to be **less than 100 ms** for timely warnings.

---

## 6. Risk Model Assumptions

1. Visibility is represented using a normalized **Visibility Index** between 0 and 1.
2. Higher visibility corresponds to lower risk scaling, while poorer visibility corresponds to higher risk scaling.
3. Ambient light, rain, and fog are assumed to be sufficient as the main environmental factors in the model.
4. The example weight values:
   - `w_L = 0.40`
   - `w_R = 0.35`
   - `w_F = 0.25`
   are assumed design choices for this prototype and are not yet experimentally optimized.

5. The TTC threshold is assumed to increase under poor visibility conditions.
6. The safety radius is also assumed to increase under poor visibility conditions.
7. Broadcast rate is assumed to increase under poor visibility conditions unless congestion-control logic limits it.
8. Risk level is simplified into broad categories such as:
   - Low,
   - Medium,
   - High.

---

## 7. Scenario and Use-Case Assumptions

1. The system is especially intended for situations where pedestrian detection is harder, such as:
   - night driving,
   - heavy rain,
   - fog,
   - reduced roadside illumination.

2. The example scenario in the README, involving a vehicle travelling at **60 km/h at night during heavy rain**, is used as an illustrative case and not as a fixed operational limit.
3. The project assumes that earlier warnings are beneficial in poor visibility because braking distance and reaction demands increase.
4. The system is assumed to complement existing driver assistance systems rather than replace them.

---

## 8. Safety and Limitation Assumptions

1. If the pedestrian carries neither a smartphone nor a compatible smart tag, the pedestrian will not receive direct V2P warnings.
2. In such a case, safety improvement may still come indirectly through vehicle-side driver alerts.
3. If sensors or communication modules fail, the AViRS-V2P system may not operate correctly.
4. GNSS positioning error, packet loss, and interference are assumed possible in real environments.
5. To reduce the effect of uncertainty, the design uses:
   - safety radius margins,
   - TTC threshold margins,
   - stale-message filtering,
   - congestion-aware behaviour.

6. The project assumes that these measures improve robustness, but they do not eliminate all real-world failure modes.

---

## 9. Accessibility Assumptions

1. Different pedestrians may require different alert modalities.
2. The smartphone application is assumed to support:
   - audio warnings for visually impaired users,
   - vibration and visual warnings for hearing-impaired users,
   - multi-modal alerts for general accessibility.

3. The smart tag is assumed to support tactile and visual warnings, but not full spoken guidance.
4. Accessibility support is considered part of the design intent of AViRS-V2P.

---

## 10. Documentation Assumptions

1. The README, diagrams, hardware notes, and parameter tables are intended to remain internally consistent.
2. Terms such as:
   - Visibility Index,
   - TTC threshold,
   - safety radius,
   - broadcast rate,
   - risk level,
   should be used consistently across all project files.

3. Example values may be refined later if simulation or testing is performed.
4. Until then, the current values are treated as reasonable prototype assumptions for explaining the concept clearly.
