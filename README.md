# Adaptive Visibility & Risk Scaling V2P System (AViRS-V2P)

## Project Overview

AViRS-V2P (Adaptive Visibility & Risk Scaling Vehicle-to-Pedestrian system) is a context-aware V2P communication system designed to improve pedestrian safety in **low-visibility conditions such as night, heavy rain, and fog**.

Traditional V2P systems typically rely on **fixed time-to-collision (TTC) thresholds and fixed broadcast rates**, which may not provide sufficient warning when environmental visibility deteriorates. At the same time, simply increasing message frequency can lead to **channel congestion and packet collisions** in vehicular networks.

Our proposed system introduces **adaptive communication and risk scaling**, where safety parameters and communication behaviour dynamically adjust based on environmental conditions and vehicle speed. The system computes a **Visibility Index** derived from ambient light levels, rain intensity (estimated through wiper activity), and fog detection from camera-based visibility estimation.

Using this index, AViRS-V2P dynamically adjusts:

* TTC thresholds
* Broadcast frequency
* Safety radius (alert distance)
* Alert intensity

This transforms V2P from a **static warning system into a context-aware adaptive communication protocol**, allowing safer operation while maintaining network efficiency.

---
## Literature Research

Vehicle-to-Pedestrian (V2P) safety has been widely studied as a way to protect vulnerable road users (VRUs) by broadcasting a vehicle’s state (e.g., position, speed, heading) and estimating collision risk on the receiving side. However, surveys highlight that practical V2P performance depends heavily on latency, reliability, positioning uncertainty, and scenario context, and that “one-size-fits-all” thresholds (e.g., a fixed TTC cut-off) may not generalize well across different pre-crash situations and environmental conditions (Sewalkar & Seitz, 2019). Beyond application logic, V2P also must remain network-feasible: raising beacon/message rates everywhere can increase channel load, collisions, and packet loss, which undermines safety messaging precisely when density is high.

On the standardization side, ETSI’s VRU awareness work defines a VRU awareness basic service and associated message concepts (e.g., VAM) that structure how VRU-related information can be communicated, including considerations around dissemination and redundancy mitigation (ETSI TS 103 300-3). At the access/network layer, ITS-G5 specifications incorporate Decentralized Congestion Control (DCC) concepts to manage channel congestion and maintain acceptable performance under load (ETSI EN 303 797). ETSI technical reports further analyze congestion-control behavior and potential improvements, reinforcing that congestion management is a first-class design constraint for safety communications (ETSI TR 104 073).

Building on this, AViRS-V2P contributes a protocol-level adaptation layer that couples (i) visibility-aware risk scaling (via a Visibility Index derived from light/rain/fog proxies) with (ii) adaptive TTC thresholds and adaptive broadcast rates, while (iii) applying a DCC-style backoff when the channel is busy. This targets earlier warning under poor visibility without “always-on high-rate” transmissions that can degrade overall network reliability.

# 1 System Architecture

## Overall Architecture

## Architecture Diagram
[![Block Diagram](<architecture/diagrams/block diagram.jpeg>)](<architecture/diagrams/block diagram.jpeg>)

## Communication Flow
![Communication Flow](architecture/diagrams/communication_flow.png)

## Sequence Flow
![Sequence Flow](<architecture/diagrams/sequence flow.jpeg>)

The AViRS-V2P system consists of two main components:

### Vehicle Side

The vehicle continuously evaluates environmental conditions and computes a dynamic risk level.

Main components:

* Global Navigation Satellite System receiver / GNSS receiver (vehicle position and speed)
* Ambient light sensor / camera
* Rain intensity detection (wiper speed proxy)
* Fog or visibility estimator
* Risk computation module
* V2X On-Board Unit (DSRC or C-V2X)

The vehicle broadcasts safety messages to nearby pedestrian devices.

---

### Pedestrian Side

The pedestrian device (smartphone) receives V2P safety messages and evaluates collision risk.

Main components:

* Smartphone GNSS
* Inertial sensors (accelerometer / motion detection)
* V2P communication receiver
* TTC estimation module
* Alert engine (audio / vibration warning)

---

# 2 Functions and Communication Messages

## System Functions

### Vehicle Node

* Collect environmental sensor data
* Compute Visibility Index
* Adjust TTC threshold dynamically
* Adjust broadcast frequency
* Broadcast V2P safety message

### Pedestrian Node

* Receive vehicle message
* Estimate relative distance and speed
* Compute TTC
* Compare TTC with threshold
* Trigger alert if risk is detected

---

## AViRS Safety Message Format

| Field            | Description                             |
| ---------------- | --------------------------------------- |
| Vehicle ID       | Anonymous identifier                    |
| Timestamp        | Message generation time                 |
| Vehicle Position | GNSS latitude & longitude               |
| Vehicle Speed    | Current speed                           |
| Heading          | Vehicle direction                       |
| Visibility Index | Computed environmental visibility score |
| TTC Threshold    | Dynamic collision threshold             |
| Safety Radius    | Maximum alert range                     |
| Risk Level       | Low / Medium / High                     |

---

# 3 Hardware Components and System Parameters

## Vehicle Sensors

| Sensor                    | Purpose                             |
| ------------------------- | ----------------------------------- |
| GNSS                      | Vehicle location and speed          |
| Ambient Light Sensor      | Detect night conditions             |
| Rain Sensor / Wiper State | Estimate rain intensity             |
| Camera                    | Detect fog / visibility degradation |
| V2X On-Board Unit         | Wireless V2P communication          |

---

## Example System Parameters

| Parameter      | Example Value |
| -------------- | ------------- |
| Default TTC    | 2 seconds     |
| Maximum TTC    | 4 seconds     |
| Broadcast Rate | 1–10 Hz       |
| Safety Radius  | 30–90 m       |
| Message Size   | ~200 bytes    |
| Target Latency | <100 ms       |

---

## Environmental Parameter Ranges

| Parameter | Measurement Unit | Example Range |
|----------|------------------|--------------|
| Ambient Light | lux | 0 – 10000 |
| Rain Intensity | mm/hour | 0 – 10+ |
| Visibility Distance | meters | 20 – 200+ |

---

# 4 Adaptive Risk Scaling Model

## Visibility Index (normalized)

We normalize each factor to [0,1]:

- L = normalized ambient light (0 = dark, 1 = bright)
- R = normalized rain intensity (0 = none, 1 = heavy)
- F = normalized fog level (0 = clear, 1 = dense)

Visibility Index (higher = better visibility):

$$
V = w_L L + w_R (1 - R) + w_F (1 - F), \quad V \in [0,1]
$$

Risk scaling factor (higher = higher risk due to low visibility):

$$
S = 1 - V
$$

Lower visibility results in higher risk scaling.

Example weighting factors used in this model:

- wL = 0.4 (ambient light importance)
- wR = 0.35 (rain impact)
- wF = 0.25 (fog impact)

The weights can be calibrated using experimental data or simulation in future implementations.

---

## Environmental Parameter Normalization

The environmental parameters used in the Visibility Index are derived from real-world sensor measurements and normalized to a range between **0 and 1** so that they can be combined in a single equation.

Normalization is commonly used in engineering systems when different sensors measure different physical units (e.g., lux, precipitation rate, visibility distance).

### Ambient Light (L)

Ambient light is measured using a vehicle light sensor or camera and expressed in **lux**.

| Condition | Light Level | Normalized L |
|----------|-------------|--------------|
| Bright daylight | >1000 lux | 1.0 |
| Cloudy daylight | 200–500 lux | 0.8 |
| Street lighting | 10–50 lux | 0.3 |
| Dark rural road | <5 lux | 0.1 |

Lower light levels correspond to poorer visibility and therefore lower values of **L**.

---

### Rain Intensity (R)

Rain intensity can be estimated using **vehicle wiper speed** as a proxy for precipitation level.

| Wiper Speed | Estimated Rain Intensity | Normalized R |
|-------------|--------------------------|--------------|
| Off | No rain | 0.0 |
| Intermittent | Light rain | 0.3 |
| Medium speed | Moderate rain | 0.6 |
| Maximum speed | Heavy rain | 1.0 |

Higher rain intensity increases visual obstruction and therefore increases the risk factor.

---

### Fog Level (F)

Fog severity can be estimated using **camera-based visibility detection**, where object contrast decreases as fog density increases.

| Visibility Distance | Fog Condition | Normalized F |
|---------------------|---------------|--------------|
| >200 m | Clear | 0.0 |
| 100–200 m | Light fog | 0.3 |
| 50–100 m | Moderate fog | 0.6 |
| <50 m | Dense fog | 1.0 |

Lower visibility distances correspond to higher fog levels.

---

Using these normalized parameters allows different environmental measurements to be combined into a **single visibility index**, enabling the AViRS-V2P system to adapt communication and safety parameters dynamically.

---

## Adaptive TTC Threshold

| Condition    | TTC Threshold | Broadcast Rate |
| ------------ | ------------- | -------------- |
| Clear Day    | 2.0 s         | 1 Hz           |
| Night        | 3.0 s         | 5 Hz           |
| Heavy Rain   | 3.5 s         | 8 Hz           |
| Night + Rain | 4.0 s         | 10 Hz          |

This allows the system to increase safety margins during poor visibility.

---

# 5 Use Case Scenario

A vehicle travels at **60 km/h at night during heavy rain**. Environmental sensors detect low ambient light and high rain intensity, resulting in a low Visibility Index.

The AViRS-V2P system increases the TTC threshold from **2 seconds to 4 seconds** and raises the broadcast rate to **10 Hz** to improve detection reliability.

A pedestrian approaching a crossing receives the V2P message through their smartphone. The system calculates the relative trajectory and determines that the TTC falls below the adaptive threshold.

The pedestrian device immediately triggers a **vibration and audio alert**, allowing the pedestrian to stop before entering the vehicle’s path.

This adaptive approach provides earlier warnings compared to traditional fixed-threshold V2P systems.

---

# 6 Decision Log

(Chronological record of design decisions)

| Date  | Trigger                    | Options                         | Criteria           | Decision              | AI Usage         | Team Member |
| ----- | -------------------------- | ------------------------------- | ------------------ | --------------------- | ---------------- | ----------- |
| XX/XX | Initial design idea        | Fixed vs adaptive TTC           | Safety performance | Chose adaptive TTC    | AI brainstorming | Member A    |
| XX/XX | Network congestion concern | Fixed vs dynamic rate           | Bandwidth          | Dynamic rate scaling  | AI suggestion    | Member B    |
| XX/XX | Environmental sensing      | Raw sensors vs visibility index | Message size       | Visibility index used | AI assisted      | Member C    |

*(More entries will be added during system development.)*

---

# 7 AI Usage and Reflection

## AI Tools Used

* ChatGPT – concept ideation and system design
* AI diagram tools – architecture diagrams
* AI writing support – documentation structuring

---

## Example Prompts Used

Example prompt 1:

> "Suggest a novel V2P application with adaptive communication for vehicular networks."

Example prompt 2:

> "Generate a V2P message structure including TTC and visibility parameters."

Example prompt 3:

> "Provide pseudocode for adaptive TTC calculation."

---

## AI Limitations Identified

1. AI suggested unrealistic hardware sensors that were replaced with practical alternatives.
2. Some generated parameter values were unrealistic and were manually verified.
3. AI occasionally produced overly complex message formats that required simplification.

---

## Individual Reflection

Each team member will include a short reflection describing:

* Their contribution to the project
* How AI tools assisted their work
* How results were verified and improved

---
