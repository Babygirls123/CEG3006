# Decision Log (chronological)

| Date | Trigger / Problem | Options / Alternatives | Evaluation criteria | Decision + rationale | AI usage | Member(s) | Repo links |
|---|---|---|---|---|---|---|---|
| 2026-03-03 | Need original V2P concept | Fixed TTC+rate vs adaptive risk+rate | Originality, network relevance, feasibility | Adaptive risk + congestion-bounded rate | Brainstorm support | ___ | README §1.1 |
| 2026-03-03 | Visibility modelling | Lux only vs wiper only vs lux+wiper+contrast | Robustness, explainability | Use V index with 3 inputs | Outline support | ___ | README §1.3 |
| 2026-03-04 | Rain proxy | rain sensor vs wiper proxy | availability, feasibility | use wiper proxy | options list | ___ | docs/assumptions.md |
| 2026-03-04 | Fog estimate | dedicated sensor vs camera contrast | complexity, realism | camera contrast heuristic | options list | ___ | docs/assumptions.md |
| 2026-03-05 | Risk check method | distance-only vs TTC-only vs closest-approach | false positives, interpretability | closest-approach (t*, d_min) | suggested | ___ | README §1.3 |
| 2026-03-05 | Who triggers alerts | vehicle-side only vs phone-side only vs split | privacy, latency, feasibility | split: vehicle broadcasts thresholds; phone computes TTC | suggested | ___ | README §1.3 |
| 2026-03-06 | Congestion control | none vs cap via CBR | channel stability, reliability | cap rate using CBR-based f_cap | suggested | ___ | tables/parameters.md |
| 2026-03-06 | Broadcast strategy | fixed 10 Hz vs adaptive 1–10 Hz | channel load, timeliness | adaptive desired rate then cap | suggested | ___ | tables/parameters.md |
| 2026-03-07 | Payload size | rich payload vs minimal fields | overhead, clarity | minimal AViRS fields only | refined | ___ | tables/message-formats.md |
| 2026-03-07 | Privacy | static ID vs pseudonym rotation | tracking risk | rotate pseudonymous sender_id | flagged | ___ | docs/assumptions.md |
