# Decision Log — AViRS-V2P (Adaptive Visibility & Risk Scaling V2P System)

This file is a **chronological record** of the evolution of the AViRS-V2P concept.  
Each entry documents the **trigger/problem**, realistic **alternatives**, **engineering evaluation criteria**, the final **decision and rationale**, **AI usage**, and the **team member(s)** leading the work.

> References to design details are aligned to sections in `README.md`:
- System Architecture: **Section 1**
- Functions + Messages: **Section 2**
- Hardware + Parameters: **Section 3**
- Risk Scaling Model: **Section 4**
- Use Case Scenario: **Section 5**
- AI Usage & Reflection: **Section 7**

---

## Decision Log Entries

| Date (DD/MM) | Trigger / Problem (what caused this entry?) | Options / Alternatives (realistic) | Evaluation Criteria (engineering metrics) | Decision and Rationale (what changed?) | AI Usage (if any) (AI contribution + human correction) | Team members (lead) |
|---|---|---|---|---|---|---|
| 05/03 | Brainstorm multiple V2P concepts and select one that is original and fits the module scope | A) “Night Visibility Assist” for Pedestrians B) “Distracted Pedestrian” Protection Mode C) Wheelchair Ramp Guardian (comfort + accessibility) D) School-Zone Group Bubble (clustered VRU beacons to reduce congestion) E) Intent-Aware Smart Crossing F) Dynamic Safety Aura (Personal Adaptive Risk Field) H) Smart Crosswalk Guardian (For Pedestrians) I) Blind Spot Shield for Cyclists J) Reversing Vehicle Early Warning for Pedestrians K) “Silent Vehicle Alert” (EV & Quiet Car Awareness System) | Originality; feasibility (no prototype required); network relevance (rate/congestion); safety impact; clarity of architecture | Selected **D** because it is not just an app warning: it links **visibility** to **risk scaling + adaptive communication** and includes network constraints (avoid congestion). | ChatGPT accelerated ideation by generating several concept directions; humans filtered out generic/unrealistic options and chose the one with strongest **vehicular-network reasoning**. | **Whole team** (brainstorm), Zhang Jia Yi (logged) |
| 07/03 | Concept still risked sounding like a general safety app; needed stronger vehicular-network/protocol framing | A) Fixed TTC + fixed 1 Hz beacons B) Always high-rate (e.g., 10 Hz) C) **Adaptive TTC + adaptive rate + congestion-aware behavior** | Warning reliability in low visibility; channel load; packet collision rate; latency; scalability in dense traffic | Picked **C** to make the system clearly **network-aware**: earlier warning when needed but avoid constant congestion from always-high-rate broadcasting. | AI suggested common V2P features; humans reframed it into **protocol adaptation + congestion constraint** rather than generic app logic. | Zhang Jia Yi, Sylvia |
| 10/03 | Needed a measurable definition of “visibility” using feasible sensors (avoid unrealistic hardware) | A) Ambient light only B) Weather API/infrastructure data C) **Fuse light + wiper proxy (rain) + camera fog estimation** | Sensor availability; cost/complexity; update rate; robustness; feasibility in assignment scope | Picked **C**: practical sensing proxies that can be normalized and combined; avoided dependency on external services. | AI proposed extra sensors (some unrealistic); humans replaced them with **light + wiper proxy + camera visibility/fog estimation** that fits vehicle platforms. | Loong Chor Yi |
| 12/03 | Visibility model must behave correctly (worse visibility → higher risk) and stay bounded | A) Unnormalized score B) Multiplicative model C) **Normalized weighted sum \(V\in[0,1]\), scaling \(S=1-V\)** | Bounded outputs; monotonic behavior; tuning simplicity; explainability | Picked **C** because it guarantees \(V\) stays within **[0,1]**, and \(S\) increases as visibility drops. | AI drafted formulas; humans corrected **sign direction** (using \(1-R\), \(1-F\)) and ensured bounds/clamping assumptions are clear. | Loong Chor Yi |
| 14/03 | Fixed TTC threshold may be unsafe in night/rain/fog; needed adaptive safety margin | A) TTC fixed at 2 s B) Manual “safety mode” C) **Adaptive TTC (2–4 s) driven by visibility index** | False negative risk; false alarm rate; reaction-time margin; stability (avoid oscillation) | Picked **C**: TTC increases under poor visibility (up to 4 s), improving safety margin without requiring manual action. | AI suggested sample TTC mappings; humans adjusted values to match use-case and kept a **reasonable clamp (2–4 s)**. | Loong Chor Yi, Fatin Farahin |
| 17/03 | Increasing message rate improves detection but can cause congestion/collisions | A) Constant 1 Hz B) Constant 10 Hz C) **Adaptive 1–10 Hz + DCC-style backoff when channel busy** | Channel busy ratio; packet delivery ratio; collision probability; latency; fairness | Picked **C**: raise rate only under high risk/low visibility, and reduce rate when channel is busy to protect reliability. | AI helped summarize congestion-control approach; humans simplified it to “**DCC-style backoff**” (credible and within scope). | Sylvia Goh Wen Wen |
| 19/03 | Message payload must support pedestrian TTC decision but avoid oversized frames | A) Minimal: position+speed only B) Very rich: raw sensor data C) **Compact message (~200 bytes) with only decision-critical fields** | Airtime; packet size; congestion impact; privacy; completeness for TTC + alert | Picked **C**: include position/speed/heading + visibility index + TTC threshold + safety radius + risk level; removed raw sensor readings not needed by decision logic. | AI proposed many fields; humans verified “**every transmitted field is used**” and trimmed overly complex formats. | Fatin Farahin |
| 21/03 | Pedestrian-side solution must be practical (no dedicated devices required) | A) Dedicated wearable V2X device B) **Smartphone GNSS + inertial + V2P receiver** C) Infrastructure-assisted tracking | Deployment practicality; cost; energy use; positioning uncertainty; accessibility | Picked **B**: smartphone-based node is realistic and within scope; uses GNSS + inertial to support motion state and TTC reasoning. | AI helped structure the workflow; humans ensured consistency across architecture, functions, and decision logic. | Shirin Nadia |
| 26/03 | Final integration: ensure submission-ready consistency (tables, narrative, parameters match use-case) | A) Leave sections independent B) Partial standardization C) **Standardize terms + cross-check values across sections** | Internal consistency; traceability; readability; defensibility | Picked **C**: standardized “Visibility Index / TTC threshold / broadcast rate / safety radius” and aligned tables with scenario (night+rain → TTC 4 s, 10 Hz). | AI generated checklists; humans fixed mismatches and removed claims that were arbitrary without stated assumptions. | Sylvia Goh Wen Wen, Zhang Jia Yi |

---

## Notes / Next Steps (Planned Entries)

These are the **next decisions** we expect to log as the repo develops further:
- DSRC vs C-V2X assumption justification (latency/reliability trade-offs)
- Simple channel-load calculation example (vehicles × Hz × bytes)
- Privacy enhancement (rotating pseudonymous IDs)
- Edge-case handling (pedestrian stationary, vehicle moving away)

---
