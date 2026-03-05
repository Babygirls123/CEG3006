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

| Date (DD/MM) | Trigger / Problem | Options / Alternatives | Evaluation Criteria (Engineering Metrics) | Decision & Rationale | AI Usage (if any) | Team Member(s) |
|---|---|---|---|---|---|---|
| 05/03 | Many V2P ideas are generic “warning apps”; need stronger vehicular-network focus | (A) Simple fixed TTC alert app (B) V2P with only adaptive TTC (C) Adaptive TTC + adaptive broadcast + visibility-aware scaling | Originality, network relevance, measurable trade-offs (latency/bandwidth) | Chose **(C)** to make it a protocol-level design: adaptive TTC + adaptive rate + visibility index (README **1,2,4**) | ChatGPT used to brainstorm multiple concepts; team merged best parts | Sylvia, Fatin |
| 05/03 | Poor visibility is hard to quantify consistently | (A) Use raw sensor data only (lux/rain/fog) (B) Use a single Visibility Index (C) Use ML classifier | Message size, explainability, feasibility, integration simplicity | Chose **(B)** Visibility Index approach to compress sensing into one score while staying explainable (README **4.1, 4.2**) | ChatGPT suggested candidate formulas; team corrected sign/normalization direction | Chor Yi |
| 06/03 | Need to decide where risk computation happens | (A) Vehicle computes TTC for pedestrian (B) Phone computes everything (C) Vehicle computes visibility + threshold; phone computes TTC | Latency, compute load, battery use, privacy, message overhead | Chose **(C)** split computation: vehicle computes Visibility Index + TTC threshold; pedestrian computes TTC locally (README **2.1, 4.3**) | Used ChatGPT to compare architecture splits; human sanity checks | Fatin, Shirin |
| 06/03 | Rate scaling risks channel congestion during dense traffic | (A) Always allow 10 Hz when low visibility (B) Add congestion-aware backoff (DCC-style) (C) Fixed 5 Hz maximum | Channel load, packet collision probability, scalability | Chose **(B)** DCC-style backoff: allow up to 10 Hz only when conditions require and channel load acceptable (Literature mentions DCC; README **Literature, 3.2, 4.3**) | ChatGPT suggested congestion-control framing; team anchored it to ETSI DCC concept | Sylvia, Jia Yi |
| 07/03 | Decide what fields must be in the message to support TTC + adaptation | (A) Minimal: position, speed only (B) Add visibility index + TTC threshold (C) Add all raw sensors (lux/rain/fog) | Message size (~200B target), receiver usefulness, privacy risk | Chose **(B)** include Visibility Index + TTC threshold + Safety Radius; keep message compact (README **2.2**, **3.2**) | ChatGPT proposed extra fields; team trimmed to maintain feasibility | Fatin |
| 07/03 | Risk Level definition is unclear (Low/Med/High) | (A) Derive risk purely from TTC (B) Derive risk purely from Visibility Index (C) Combine TTC threshold regime + visibility context | False positives, explainability, consistent behavior across scenarios | Chose **(C)** define risk levels tied to adaptive TTC regimes and poor visibility context so alert intensity is justified (README **2.2**, **4.3**) | ChatGPT suggested mapping; team simplified to 3-level output | Chor Yi |
| 08/03 | Safety Radius should not be fixed in poor conditions | (A) Fixed 30 m always (B) Scale with visibility only (C) Scale with visibility + speed | Reaction distance, nuisance alerts, realism (night+rain needs larger radius) | Chose **(C)** Safety Radius is adaptive (30–90 m) and increases under low visibility / higher speed (README **3.2**, **4.3**, **5**) | ChatGPT suggested wide ranges; humans constrained to stated parameter table | Shirin, Chor Yi |
| 08/03 | Equations must match the normalization tables | (A) Keep equation as-is without normalization explanation (B) Add explicit normalization tables for L/R/F (C) Remove equation and only keep lookup table | Consistency, readability, ability to defend to prof/marker | Chose **(B)** keep equation + add normalization tables for Ambient Light, Rain, Fog (README **4.2.1–4.2.3**) | ChatGPT helped format tables; team verified ranges are realistic | Chor Yi, Sylvia |
| 09/03 | Decide on parameter bounds that are physically plausible | (A) TTC up to 6–8 s (B) TTC 2–4 s (C) TTC 1–3 s only | Driver reaction time, braking distance realism, practicality | Chose **(B)** TTC thresholds 2–4 s (clear→night+rain), matching use-case at 60 km/h and network limits (README **3.2**, **4.3**, **5**) | ChatGPT proposed larger values; team selected defensible bounds | Sylvia |
| 09/03 | GitHub repository needs a clear plan and structure early | (A) Write everything first then organize later (B) Build README skeleton now and fill sections iteratively (C) Only commit final README near deadline | Traceability, professionalism, progress evidence | Chose **(B)** create README with full headings early + add diagrams/tables iteratively to show authentic development (README structure) | ChatGPT assisted with formatting and section ordering | Shirin |
| 10/03 | Decide how to present formulas since GitHub may not render LaTeX | (A) Keep LaTeX blocks only (B) Provide both LaTeX and code-block formula (C) Remove formulas | Readability on GitHub, clarity during marking | Chose **(B)** keep the formula text and include a code-block fallback so it’s readable on GitHub (README **4.1**) | ChatGPT suggested formatting approaches; team used code-block fallback | Shirin, Sylvia |
| 10/03 | Decide how to connect to standards and networking literature | (A) No standards mention (B) Mention ETSI VRU/VAM only (C) Mention ETSI VRU/VAM + ITS-G5 DCC + congestion report | Credibility, network engineering alignment, evidence of research | Chose **(C)** include ETSI VRU awareness + ITS-G5 DCC + TR on congestion behavior (README **Literature Research**) | ChatGPT drafted summary; team verified relevance and kept it concise | Sylvia |
| 11/03 | Decision log must be “realistic alternatives”, not fake diversity | (A) Add random options just to fill 10 entries (B) Document only real design trade-offs we actually discussed | Authenticity, engineering defensibility | Chose **(B)** write entries tied directly to architecture/messages/parameters and show evaluation criteria (this file + README **6**) | ChatGPT suggested topics; team rewrote as concrete decisions | Jia Yi |
| 11/03 | Need to ensure “message fields are used” (avoid dead fields) | (A) Keep extra fields “just in case” (B) Keep only fields used by pedestrian TTC + threshold logic | Message overhead, clarity, feasibility | Chose **(B)** keep only fields used by pedestrian logic (position/speed/heading/visibility/TTC threshold/safety radius/risk level) (README **2.2**) | ChatGPT suggested extra; human check removed unused | Fatin |
| 12/03 | Alert intensity needs to be consistent and not annoying | (A) Always audio+vibrate (B) Risk-level based alert intensity (C) Only alert in critical | User acceptance, false positives, safety effectiveness | Chose **(B)** alert intensity based on risk level (Low/Med/High) to balance false alarms and safety (README **2.1**, **2.2**) | ChatGPT suggested alert mapping; team aligned to risk level table | Shirin |

---

## Notes / Next Steps (Planned Entries)

These are the **next decisions** we expect to log as the repo develops further:
- DSRC vs C-V2X assumption justification (latency/reliability trade-offs)
- Simple channel-load calculation example (vehicles × Hz × bytes)
- Privacy enhancement (rotating pseudonymous IDs)
- Edge-case handling (pedestrian stationary, vehicle moving away)

---
