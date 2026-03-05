# Adaptive Visibility & Risk Scaling V2P System (AViRS-V2P)

---

# Project Overview

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

# Literature Research

Vehicle-to-Pedestrian (V2P) safety has been widely studied as a way to protect vulnerable road users (VRUs) by broadcasting a vehicle’s state (e.g., position, speed, heading) and estimating collision risk on the receiving side. However, surveys highlight that practical V2P performance depends heavily on latency, reliability, positioning uncertainty, and scenario context, and that “one-size-fits-all” thresholds (e.g., a fixed TTC cut-off) may not generalize well across different pre-crash situations and environmental conditions (Sewalkar & Seitz, 2019). Beyond application logic, V2P also must remain network-feasible: raising beacon/message rates everywhere can increase channel load, collisions, and packet loss, which undermines safety messaging precisely when density is high.

On the standardization side, ETSI’s VRU awareness work defines a VRU awareness basic service and associated message concepts (e.g., VAM) that structure how VRU-related information can be communicated, including considerations around dissemination and redundancy mitigation (ETSI TS 103 300-3). At the access/network layer, ITS-G5 specifications incorporate Decentralized Congestion Control (DCC) concepts to manage channel congestion and maintain acceptable performance under load (ETSI EN 303 797). ETSI technical reports further analyze congestion-control behavior and potential improvements, reinforcing that congestion management is a first-class design constraint for safety communications (ETSI TR 104 073).

Building on this, AViRS-V2P contributes a protocol-level adaptation layer that couples:

1. **Visibility-aware risk scaling** (via a Visibility Index derived from light/rain/fog proxies)
2. **Adaptive TTC thresholds and adaptive broadcast rates**
3. **DCC-style backoff when the channel is busy**

This targets earlier warning under poor visibility without “always-on high-rate” transmissions that can degrade overall network reliability.

---

# 1. System Architecture

## 1.1 Overall Architecture

The AViRS-V2P system consists of two main components:

* **Vehicle Node**
* **Pedestrian Node**

---

## 1.2 Architecture Diagram

![Block Diagram](architecture/diagrams/block%20diagram.jpeg)

---

## 1.3 Vehicle Side

The vehicle continuously evaluates environmental conditions and computes a dynamic risk level.

### Main Components

* Global Navigation Satellite System receiver / **GNSS receiver** (vehicle position and speed)
* **Ambient light sensor / camera**
* **Rain intensity detection** (wiper speed proxy)
* **Fog or visibility estimator**
* **Risk computation module**
* **V2X On-Board Unit** (DSRC or C-V2X)

The vehicle broadcasts safety messages to nearby pedestrian devices.

---

## 1.4 Pedestrian Side

The pedestrian device (smartphone) receives V2P safety messages and evaluates collision risk.

### Main Components

* **Smartphone GNSS**
* **Inertial sensors** (accelerometer / motion detection)
* **V2P communication receiver**
* **TTC estimation module**
* **Alert engine** (audio / vibration warning)

---

# 2. Functions and Communication Messages

## 2.1 System Functions

### Vehicle Node

* Collect environmental sensor data
* Compute Visibility Index
* Adjust TTC threshold dynamically
* Adjust broadcast frequency
* Broadcast V2P safety message

---

### Pedestrian Node

* Receive vehicle message
* Estimate relative distance and speed
* Compute TTC
* Compare TTC with threshold
* Trigger alert if risk is detected

---

## 2.2 AViRS Safety Message Format

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

# 3. Hardware Components and System Parameters

## 3.1 Vehicle Sensors

| Sensor                    | Purpose                             |
| ------------------------- | ----------------------------------- |
| GNSS                      | Vehicle location and speed          |
| Ambient Light Sensor      | Detect night conditions             |
| Rain Sensor / Wiper State | Estimate rain intensity             |
| Camera                    | Detect fog / visibility degradation |
| V2X On-Board Unit         | Wireless V2P communication          |

---

## 3.2 Example System Parameters

| Parameter      | Example Value |
| -------------- | ------------- |
| Default TTC    | 2 seconds     |
| Maximum TTC    | 4 seconds     |
| Broadcast Rate | 1–10 Hz       |
| Safety Radius  | 30–90 m       |
| Message Size   | ~200 bytes    |
| Target Latency | <100 ms       |

---

## 3.3 Environmental Parameter Ranges

| Parameter           | Measurement Unit | Example Range |
| ------------------- | ---------------- | ------------- |
| Ambient Light       | lux              | 0 – 10000     |
| Rain Intensity      | mm/hour          | 0 – 10+       |
| Visibility Distance | meters           | 20 – 200+     |

---

# 4. Adaptive Risk Scaling Model

## 4.1 Visibility Index (Normalized)

We normalize each factor to **[0,1]**:

* **L** = normalized ambient light (0 = dark, 1 = bright)
* **R** = normalized rain intensity (0 = none, 1 = heavy)
* **F** = normalized fog level (0 = clear, 1 = dense)

### Visibility Index Formula

[
V = w_L L + w_R (1 - R) + w_F (1 - F)
]

[
V \in [0,1]
]

Risk scaling factor:

[
S = 1 - V
]

Lower visibility results in higher risk scaling.

### Example Weighting Factors

* wL = **0.4** (ambient light importance)
* wR = **0.35** (rain impact)
* wF = **0.25** (fog impact)

The weights can be calibrated using experimental data or simulation in future implementations.

---

## 4.2 Environmental Parameter Normalization

Environmental parameters are normalized so that measurements with different physical units can be combined into a single equation.

---

### 4.2.1 Ambient Light (L)

| Condition       | Light Level | Normalized L |
| --------------- | ----------- | ------------ |
| Bright daylight | >1000 lux   | 1.0          |
| Cloudy daylight | 200–500 lux | 0.8          |
| Street lighting | 10–50 lux   | 0.3          |
| Dark rural road | <5 lux      | 0.1          |

Lower light levels correspond to poorer visibility.

---

### 4.2.2 Rain Intensity (R)

| Wiper Speed   | Estimated Rain Intensity | Normalized R |
| ------------- | ------------------------ | ------------ |
| Off           | No rain                  | 0.0          |
| Intermittent  | Light rain               | 0.3          |
| Medium speed  | Moderate rain            | 0.6          |
| Maximum speed | Heavy rain               | 1.0          |

Higher rain intensity increases visual obstruction.

---

### 4.2.3 Fog Level (F)

| Visibility Distance | Fog Condition | Normalized F |
| ------------------- | ------------- | ------------ |
| >200 m              | Clear         | 0.0          |
| 100–200 m           | Light fog     | 0.3          |
| 50–100 m            | Moderate fog  | 0.6          |
| <50 m               | Dense fog     | 1.0          |

Lower visibility distances correspond to higher fog levels.

---

## 4.3 Adaptive TTC Threshold

| Condition    | TTC Threshold | Broadcast Rate |
| ------------ | ------------- | -------------- |
| Clear Day    | 2.0 s         | 1 Hz           |
| Night        | 3.0 s         | 5 Hz           |
| Heavy Rain   | 3.5 s         | 8 Hz           |
| Night + Rain | 4.0 s         | 10 Hz          |

This allows the system to increase safety margins during poor visibility.

---

# 5. Use Case Scenario

A vehicle travels at **60 km/h at night during heavy rain**. Environmental sensors detect low ambient light and high rain intensity, resulting in a low Visibility Index.

The AViRS-V2P system increases the TTC threshold from **2 seconds to 4 seconds** and raises the broadcast rate to **10 Hz** to improve detection reliability.

A pedestrian approaching a crossing receives the V2P message through their smartphone. The system calculates the relative trajectory and determines that the TTC falls below the adaptive threshold.

The pedestrian device immediately triggers a **vibration and audio alert**, allowing the pedestrian to stop before entering the vehicle’s path.

This adaptive approach provides earlier warnings compared to traditional fixed-threshold V2P systems.

---

# 6. Decision Log

This section records the **chronological evolution** of AViRS-V2P, including triggers, realistic alternatives, engineering criteria, decisions, and AI usage.

| Date (DD/MM) | Trigger / Problem (what caused this entry?) | Options / Alternatives (realistic) | Evaluation Criteria (engineering metrics) | Decision and Rationale (what changed?) | AI Usage (if any) (AI contribution + human correction) | Team members (lead) |
|---|---|---|---|---|---|---|
| 05/03 | **Project start:** needed to brainstorm multiple V2P concepts and select one that is original and fits the module scope | A) Generic crossing warning (fixed TTC) B) School-zone VRU alert C) Cyclist/wheelchair assist D) **Low-visibility adaptive V2P (night/rain/fog) with adaptive TTC + adaptive rate** | Originality; feasibility (no prototype required); network relevance (rate/congestion); safety impact; clarity of architecture | Selected **D** because it is not just an app warning: it links **visibility** to **risk scaling + adaptive communication** and includes network constraints (avoid congestion). | ChatGPT accelerated ideation by generating several concept directions; humans filtered out generic/unrealistic options and chose the one with strongest **vehicular-network reasoning**. | **Whole team** (brainstorm), Zhang Jia Yi (logged) |
| 05/03 | Early concept risked becoming a generic “warning app”; needed stronger vehicular-network/protocol focus | A) Fixed TTC + fixed 1 Hz beacons B) Always high-rate (e.g., 10 Hz) C) **Adaptive TTC + adaptive rate + congestion-aware behavior** | Warning reliability in low visibility; channel load; packet collision rate; latency; scalability in dense traffic | Picked **C** to make the system clearly **network-aware**: earlier warning when needed but avoid constant congestion from always-high-rate broadcasting. | AI suggested common V2P features; humans reframed it into **protocol adaptation + congestion constraint** instead of generic app logic. | Zhang Jia Yi, Sylvia |
| 05/03 | Needed a measurable definition of “visibility” using feasible sensors (avoid unrealistic hardware) | A) Ambient light only B) Weather API/infrastructure data C) **Fuse light + wiper proxy (rain) + camera fog estimation** | Sensor availability; cost/complexity; update rate; robustness; feasibility in assignment scope | Picked **C**: practical sensing proxies that can be normalized and combined; avoided dependency on external services. | AI proposed extra sensors (some unrealistic); humans replaced them with **light + wiper proxy + camera visibility/fog estimation** that fits vehicle platforms. | Loong Chor Yi |
| 05/03 | Visibility model must behave correctly (worse visibility → higher risk) and stay bounded | A) Unnormalized score B) Multiplicative model C) **Normalized weighted sum \(V\in[0,1]\), scaling \(S=1-V\)** | Bounded outputs; monotonic behavior; tuning simplicity; explainability | Picked **C** because it guarantees \(V\) stays within **[0,1]**, and \(S\) increases as visibility drops. | AI drafted formulas; humans corrected **sign direction** (using \(1-R\), \(1-F\)) and ensured bounds/clamping assumptions are clear. | Loong Chor Yi |
| 05/03 | Fixed TTC threshold may be unsafe in night/rain/fog; needed adaptive safety margin | A) TTC fixed at 2 s B) Manual “safety mode” C) **Adaptive TTC (2–4 s) driven by visibility index** | False negative risk; false alarm rate; reaction-time margin; stability (avoid oscillation) | Picked **C**: TTC increases under poor visibility (up to 4 s), improving safety margin without requiring manual action. | AI suggested sample TTC mappings; humans adjusted values to match use-case and kept a **reasonable clamp (2–4 s)**. | Loong Chor Yi, Fatin Farahin |
| 05/03 | Increasing message rate improves detection but can cause congestion/collisions | A) Constant 1 Hz B) Constant 10 Hz C) **Adaptive 1–10 Hz + DCC-style backoff when channel busy** | Channel busy ratio; packet delivery ratio; collision probability; latency; fairness | Picked **C**: raise rate only under high risk/low visibility, and reduce rate when channel is busy to protect reliability. | AI helped summarize congestion-control approach; humans simplified it to “**DCC-style backoff**” (credible and within scope). | Sylvia Goh Wen Wen |
| 05/03 | Message payload must support pedestrian TTC decision but avoid oversized frames | A) Minimal: position+speed only B) Very rich: raw sensor data C) **Compact message (~200 bytes) with only decision-critical fields** | Airtime; packet size; congestion impact; privacy; completeness for TTC + alert | Picked **C**: include position/speed/heading + visibility index + TTC threshold + safety radius + risk level; removed raw sensor readings not needed by decision logic. | AI proposed many fields; humans verified “**every transmitted field is used**” and trimmed overly complex formats. | Fatin Farahin |
| 05/03 | Pedestrian-side solution must be practical (no dedicated devices required) | A) Dedicated wearable V2X device B) **Smartphone GNSS + inertial + V2P receiver** C) Infrastructure-assisted tracking | Deployment practicality; cost; energy use; positioning uncertainty; accessibility | Picked **B**: smartphone-based node is realistic and within scope; uses GNSS + inertial to support motion state and TTC reasoning. | AI helped structure the workflow; humans ensured consistency across architecture, functions, and decision logic. | Shirin Nadia |
| 05/03 | Needed submission-ready consistency (tables/narrative/parameters must match use-case) | A) Leave sections independent B) Partial standardization C) **Standardize terms + cross-check values across sections** | Internal consistency; traceability; readability; defensibility | Picked **C**: standardized “Visibility Index / TTC threshold / broadcast rate / safety radius” and aligned tables with scenario (night+rain → TTC 4 s, 10 Hz). | AI generated checklists; humans fixed mismatches and removed claims that felt arbitrary without assumptions. | Sylvia Goh Wen Wen, Zhang Jia Yi |

# 7. AI Usage and Reflection

## 7.1 AI Tools Used

* **ChatGPT** – concept ideation and documentation

---

## 7.2 Example Prompts Used

**Example prompt 1**

> "Suggest a novel V2P application with adaptive communication for vehicular networks."

**Example prompt 2**

> "Generate a V2P message structure including TTC and visibility parameters."

**Example prompt 3**

> "Provide pseudocode for adaptive TTC calculation."

---

## 7.3 AI Limitations Identified

1. AI suggested unrealistic hardware sensors that were replaced with practical alternatives.
2. Some generated parameter values were unrealistic and were manually verified.
3. AI occasionally produced overly complex message formats that required simplification.

---

## 7.4 Individual Reflection

Each team member will include a short reflection describing:

* Their contribution to the project
* How AI tools assisted their work
* How results were verified and improved

---

## Fatin Farahin (2303542)

I contributed to defining the system scope and translating the project idea into clear vehicle-side and pedestrian-side responsibilities. I helped specify what must be included in the AViRS safety message so the pedestrian device can compute TTC and make a reliable alert decision. ChatGPT assisted by suggesting alternative message-field designs and helping us phrase the design clearly in the README.

I verified the design by checking that every message field we transmit is actually used in the pedestrian decision logic and is consistent with the functions described. I also reviewed edge cases, for example, clear day or at rainy day at night, to ensure our adaptive behaviour changes in the correct direction.

Where initial drafts were too complex, I simplified them into a form that is explainable and feasible for the assignment scope. Overall, my work focused on keeping the protocol definition clear, consistent, and implementable.

## Loong Chor Yi (2302793)

I contributed to the adaptive risk-scaling model, especially the Visibility Index definition and how it drives TTC thresholds and broadcast rates. I using ChatGPT helped generate candidate formulas and mapping rules, which I then refined to avoid ambiguity in normalization and sign direction. I verified the model by performing sanity checks across scenarios, for example I will reduced visibility should increase safety margins, and ensuring outputs remain within defined bounds using clamping assumptions.

I also aligned the condition table with the model so the examples remain consistent with the equations. When AI-generated text was generic, I rewrote it to be more precise and aligned with vehicular-network design reasoning. I improved the final version by ensuring the model section is self-contained. Overall, my work ensured the adaptive logic is coherent and defensible.

## Shirin Nadia (2302871)

I contributed to documenting the system workflow and ensuring the end-to-end communication flow is easy to follow from sensing to broadcast to pedestrian alert. ChatGPT helped structure our README sections and produce clear, concise descriptions of the pipeline and message-handling steps.

I verified the documentation by checking consistency across sections, so that no module is missing inputs or outputs. I also reviewed our diagrams and links to ensure they correctly represent the written workflow and can be opened from the README.

Where drafts were too long or unclear, I streamlined them while retaining the technical meaning. I improved the final README by ensuring the reader can understand the system without needing extra explanations. Overall, my work focused on clarity, completeness, and presentation quality.

## Sylvia Goh Wen Wen (2302759)

I contributed to integrating the README into a consistent submission-ready document and ensuring it aligns with CEG3006 expectations, like network focus, protocol reasoning, and required components. ChatGPT helped generate checklists for compliance and suggested ways to frame our design as adaptive communication plus adaptive risk thresholding.

I verified the final content by checking internal consistency, for example, the does tables match the narrative, parameters match the use-case, and the model behaves correctly under poor visibility. I also checked that our congestion-control idea is described in a way that is technically credible and fits within the scope of the module.

Where AI suggested values or claims that felt arbitrary, I tightened them and added explicit assumptions to make them defensible. I improved readability by standardizing terminology and formatting across sections. Overall, my work focused on coherence, compliance, and technical justification.

## Zhang Jia Yi (2302757)

I contributed to developing the decision log and ensuring our design decisions are framed as engineering trade-offs with measurable criteria.  I using ChatGPT to gather all the realistic alternatives, for example, message size and richness, rate scaling and congestion risk, privacy and  precision, which I refined into decision entries suitable for the README.

I verified the decision log by ensuring each entry includes clear options, criteria, and a justified choice that relates to network performance, for example, latency, channel load, reliability and energy. I also reviewed system parameters to ensure they remain plausible and consistent with the use-case scenario and adaptive table.

Where AI suggestions were vague, I rewrote them to be specific to our AViRS-V2P design and architecture. I improved the final documentation by making the decision log read as structured engineering reasoning rather than opinions. Overall, my work strengthened the project’s justification and completeness.
