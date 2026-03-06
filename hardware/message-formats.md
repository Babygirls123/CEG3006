# Message Formats

AViRS-V2P mainly uses a vehicle-to-pedestrian broadcast safety message so nearby pedestrian devices can estimate collision risk and trigger alerts. In addition, the system may send a simplified warning signal to an advanced reflective smart tag.

---

## 1. AViRS_MSG — Vehicle to Pedestrian Smartphone

This is the main broadcast safety message transmitted by the vehicle.

| Field | Type | Unit | Description |
|---|---|---|---|
| msg_type | string | — | Constant identifier: `"AViRS_MSG"` |
| vehicle_id | string | — | Anonymous or pseudonymous vehicle identifier |
| timestamp | timestamp | ms | Message generation or transmit time |
| vehicle_position | lat/lon | degrees | Vehicle GNSS position |
| vehicle_speed | float | m/s or km/h | Current vehicle speed |
| heading | float | degrees | Vehicle travel direction |
| visibility_index | float | 0–1 | Computed visibility score, where 1 = best visibility |
| ttc_threshold | float | s | Adaptive collision warning threshold |
| safety_radius | float | m | Maximum alert distance / safety bubble |
| risk_level | enum | — | Risk classification: `LOW`, `MEDIUM`, `HIGH` |
| broadcast_rate | float | Hz | Current message broadcast rate used by the vehicle |

### Purpose
The pedestrian smartphone uses this message together with its own position and motion data to:
- estimate relative distance and trajectory,
- compute Time-to-Collision (TTC),
- compare TTC against the adaptive threshold,
- trigger audio, vibration, or visual alerts when necessary.

### Notes
- `vehicle_id` should be pseudonymous to reduce privacy exposure.
- `timestamp` is used to discard stale messages.
- `broadcast_rate` is adaptive and may increase under poor visibility, but can still be limited by congestion-control logic.

---

## 2. TAG_WARN — Vehicle to Reflective Smart Tag

This is a simplified warning signal for advanced reflective smart tags attached to pedestrians, wheelchairs, canes, bicycles, or school bags.

| Field | Type | Unit | Description |
|---|---|---|---|
| msg_type | string | — | Constant identifier: `"TAG_WARN"` |
| vehicle_id | string | — | Anonymous or pseudonymous vehicle identifier |
| timestamp | timestamp | ms | Warning transmission time |
| risk_level | enum | — | Risk classification: `LOW`, `MEDIUM`, `HIGH` |
| alert_code | enum | — | Warning command such as `VIBRATE`, `LED_FLASH`, or `VIBRATE_LED` |
| safety_radius | float | m | Alert radius associated with the warning |

### Purpose
The smart tag does not need to perform full TTC computation like the smartphone. Instead, it receives a simplified warning command from the vehicle and triggers:
- vibration,
- LED flashing,
- or both.

### Notes
- `TAG_WARN` is intentionally smaller and simpler than `AViRS_MSG`.
- This supports pedestrians who may not carry smartphones.
- The vehicle may send `AViRS_MSG` to smartphones and `TAG_WARN` to smart tags in parallel.

---

## 3. Design Rationale

The message structure is kept compact so that:
- important safety information is transmitted clearly,
- adaptive thresholds can be communicated directly,
- message size remains manageable for vehicular wireless communication,
- the system can scale better under dense traffic conditions.

The adaptive fields that make AViRS-V2P different from a fixed-threshold V2P system are:
- `visibility_index`
- `ttc_threshold`
- `safety_radius`
- `broadcast_rate`
