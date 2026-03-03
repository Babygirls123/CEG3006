# ANVA-V2P: Adaptive Night/Rain/Fog Visibility Safety Bubble (V2P)

## 0) Quick links
- Decision log: docs/decision-log.md
- Assumptions: docs/assumptions.md
- References: docs/references.md
- Message formats: tables/message-formats.md
- Parameters: tables/parameters.md

---

## 1) System Integration (required)

### 1.1 Brief description + mini literature review (200–300 words)
ANVA-V2P is a context-aware Vehicle-to-Pedestrian (V2P) safety concept designed for low-visibility conditions such as night, heavy rain, and fog. In these conditions, driver perception degrades due to glare, scattering, and reduced contrast, while vehicle stopping distance increases. Many baseline V2P concepts apply fixed Time-to-Collision (TTC) thresholds and fixed broadcast rates, which may be insufficient when visibility worsens; however, naively increasing message rates everywhere can overload the wireless channel in dense traffic and increase packet collisions. ANVA-V2P addresses this by combining adaptive risk thresholding with congestion-aware communication control. A vehicle computes a Visibility Index V from ambient light, rain intensity (wiper proxy), and a fog/visibility estimate, and uses V together with vehicle speed to scale the TTC threshold, safety bubble radius, broadcast frequency, and warning priority. To prevent channel overload, the transmit rate is capped by a congestion control mechanism based on measured channel busy ratio (CBR). A pedestrian smartphone receives the vehicle broadcast, estimates relative motion and TTC, compares it against the adaptive TTC threshold, and triggers an alert (haptic/audio) when the predicted closest approach falls within the safety bubble and time horizon.

Key novelty: context-aware risk scaling that is bounded by congestion control, enabling improved VRU safety under degraded visibility without destabilising channel load.

(See docs/references.md for sources and docs/assumptions.md for stated parameter assumptions.)

### 1.2 System architecture
![Architecture](diagrams/architecture.png)

### 1.3 Functions and messages
- Message definitions: tables/message-formats.md
- Pseudocode: (paste your vehicle + pedestrian pseudocode here, or link to a section below)

#### Vehicle-side pseudocode (OBU)
[PASTE VEHICLE PSEUDOCODE HERE]

#### Pedestrian-side pseudocode (Smartphone)
[PASTE PEDESTRIAN PSEUDOCODE HERE]

### 1.4 Hardware components + parameters
See hardware/parameters.md

### 1.5 Use case (100–200 words)
A vehicle travels at 60 km/h during heavy rain at night. Ambient light is low, wiper speed indicates intense rain, and camera contrast suggests reduced visibility. The vehicle computes a low Visibility Index V and scales its safety bubble radius and TTC threshold upward, increasing warning priority. The desired message rate rises to improve timeliness, but the radio measures channel busy ratio (CBR) as moderate, so congestion control caps the rate to prevent overload. A pedestrian begins crossing near a junction. The pedestrian phone receives the vehicle’s broadcast, computes relative motion, predicts the closest approach within the adaptive TTC window, and triggers a strong haptic and audio warning. As the vehicle slows and visibility improves, the system reduces warning intensity and returns to a lower beacon rate.

---

## 2) Decision Log (required)
See docs/decision-log.md

## 3) AI Usage and Individual Reflection (required)
### 3.1 AI tools used
- ChatGPT was used for: ideation, structuring the README, drafting message tables, and drafting pseudocode.

### 3.2 Prompts + outputs (≥3)
(1) Prompt:
[PASTE]
Output used:
[PASTE SUMMARY]

(2) Prompt:
[PASTE]
Output used:
[PASTE SUMMARY]

(3) Prompt:
[PASTE]
Output used:
[PASTE SUMMARY]

### 3.3 Weaknesses / hallucinations found + verification (≥3)
1) [Example] AI suggested an unrealistic range/latency → corrected and recorded as an assumption with a cited source.
2) [Example] AI mixed up message types → corrected to align with selected standard terminology.
3) [Example] AI proposed unverified numeric thresholds → treated as assumptions and bounded by a congestion-control cap.

### 3.4 Individual reflection (5–10 sentences each)
- Member A:
- Member B:
- Member C:
