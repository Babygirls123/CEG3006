# Message Formats

This project uses a single broadcast message from the vehicle to the pedestrian device.

## AViRS_MSG (Visibility-Scaled V2P Broadcast) — Vehicle → Pedestrian

| Field | Type | Unit | Description |
|---|---|---|---|
| msg_type | string | — | Constant `"AViRS_MSG"` |
| sender_id | string | — | Pseudonymous vehicle ID (rotate periodically) |
| t_tx | timestamp | s/ms | Transmit timestamp (freshness / staleness checking) |
| veh_pos | lat/lon or (x,y) | deg/m | Vehicle position from GNSS |
| veh_speed | float | m/s | Vehicle speed |
| veh_heading | float | deg/rad | Vehicle heading |
| V | float | — | Visibility Index in [0,1], where 1 = best visibility |
| T_th | float | s | Adaptive TTC threshold |
| R_b | float | m | Adaptive safety bubble radius (alert distance) |
| RL | enum/float | — | Risk level (LOW/MED/HIGH or 0..1) |
| P | int | — | Warning priority / intensity level |
| f_des | float | Hz | Desired broadcast rate from policy (before cap) |
| f_cap | float | Hz | Congestion-control cap derived from channel load (CBR) |
| f_tx | float | Hz | Actual broadcast rate used (min(f_des, f_cap)) |

### Notes
- **Purpose:** AViRS_MSG provides adaptive thresholds and bubble radius so the pedestrian phone can compute TTC and trigger warnings under low visibility.
- **Congestion-bounded:** even if visibility is poor, the transmit rate is capped by `f_cap` to avoid channel overload.
