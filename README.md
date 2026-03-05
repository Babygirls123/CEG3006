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

| Date (DD/MM) | Trigger / Problem | Options / Alternatives | Evaluation Criteria (Engineering Metrics) | Decision & Rationale | AI Usage (if any) | Team Member(s) |
|---|---|---|---|---|---|---|
| 05/03 | Many V2P ideas are generic “warning apps”; need stronger vehicular-network focus | (A) Simple fixed TTC alert app (B) V2P with only adaptive TTC (C) Adaptive TTC + adaptive broadcast + visibility-aware scaling | Originality, network relevance, measurable trade-offs (latency/bandwidth) | Chose **(C)** to make it a protocol-level design: adaptive TTC + adaptive rate + visibility index (Sections **1,2,4**) | ChatGPT used to brainstorm multiple concepts; team merged best parts | Sylvia, Fatin |
| 05/03 | Poor visibility is hard to quantify consistently | (A) Use raw sensor data only (lux/rain/fog) (B) Use a single Visibility Index (C) Use ML classifier | Message size, explainability, feasibility, integration simplicity | Chose **(B)** Visibility Index to compress sensing into one score while staying explainable (Sections **4.1, 4.2**) | ChatGPT suggested candidate formulas; team corrected sign/normalization | Chor Yi |
| 06/03 | Need to decide where risk computation happens | (A) Vehicle computes TTC for pedestrian (B) Phone computes everything (C) Vehicle computes visibility + threshold; phone computes TTC | Latency, compute load, battery use, privacy, message overhead | Chose **(C)** split: vehicle computes Visibility Index + TTC threshold; pedestrian computes TTC locally (Sections **2.1, 4.3**) | ChatGPT used to compare architecture splits; human sanity checks | Fatin, Shirin |
| 06/03 | Rate scaling risks channel congestion during dense traffic | (A) Always allow 10 Hz when low visibility (B) Add congestion-aware backoff (DCC-style) (C) Fixed 5 Hz maximum | Channel load, packet collision probability, scalability | Chose **(B)** DCC-style backoff: up to 10 Hz only when needed and channel load acceptable (Literature + **4.3**) | ChatGPT suggested congestion-control framing; team anchored to ETSI DCC | Sylvia, Jia Yi |
| 07/03 | Decide what fields must be in the message to support TTC + adaptation | (A) Minimal: position, speed only (B) Add visibility index + TTC threshold (C) Add all raw sensors | Message size (~200B), receiver usefulness, privacy risk | Chose **(B)** include Visibility Index + TTC threshold + Safety Radius (Sections **2.2, 3.2**) | ChatGPT proposed extra fields; team trimmed | Fatin |
| 07/03 | Risk Level definition is unclear (Low/Med/High) | (A) Risk from TTC only (B) Risk from visibility only (C) Combine TTC regime + visibility context | False positives, explainability, scenario consistency | Chose **(C)** define risk levels tied to adaptive TTC regimes and poor visibility (Sections **2.2, 4.3**) | ChatGPT suggested mapping; team simplified | Chor Yi |
| 08/03 | Safety Radius should not be fixed in poor conditions | (A) Fixed 30 m always (B) Scale with visibility only (C) Scale with visibility + speed | Reaction distance, nuisance alerts, realism | Chose **(C)** adaptive Safety Radius (30–90 m) increases under low visibility/high speed (Sections **3.2, 4.3, 5**) | ChatGPT suggested wide ranges; humans constrained | Shirin, Chor Yi |
| 08/03 | Equations must match the normalization tables | (A) Equation only (B) Equation + explicit normalization tables (C) Only lookup table | Consistency, readability, defensibility | Chose **(B)** keep equation and add normalization tables (Sections **4.2.1–4.2.3**) | ChatGPT helped format; team verified ranges | Chor Yi, Sylvia |
| 09/03 | Decide on parameter bounds that are physically plausible | (A) TTC up to 6–8 s (B) TTC 2–4 s (C) TTC 1–3 s only | Reaction time, braking distance realism, practicality | Chose **(B)** TTC thresholds 2–4 s (clear→night+rain), consistent with use case (Sections **3.2, 4.3, 5**) | ChatGPT proposed larger values; team selected defensible bounds | Sylvia |
| 10/03 | LaTeX may not render on GitHub consistently | (A) LaTeX only (B) LaTeX + code-block fallback (C) Remove formulas | Readability on GitHub, marking clarity | Chose **(B)** include readable code-block formula fallback (Section **4.1**) | ChatGPT suggested formatting approaches | Shirin, Sylvia |

*(More entries will be inside decisionlog.md .)*

---

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
