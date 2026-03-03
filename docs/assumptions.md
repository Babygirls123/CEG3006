# Assumptions (ANVA-V2P)

This project is a conceptual V2P design. Any numerical values are treated as engineering assumptions unless verified by cited sources.

## A) Variables and units
- Vehicle speed: v (m/s)
- Visibility inputs:
  - L: ambient light (lux or camera-derived proxy)
  - R: rain intensity proxy from wiper speed/state
  - F: fog/visibility proxy from camera contrast
- Visibility Index: V ∈ [0,1], where **1 = best visibility**
- Adaptive TTC threshold: T_th (seconds)
- Safety bubble radius: R_b (meters)
- Desired broadcast rate: f_des (Hz)
- Congestion cap: f_cap (Hz)
- Actual broadcast rate: f_tx (Hz)

## B) Normalisation assumptions
Inputs are normalised to [0,1]:
- L_n = clip((L - L_min)/(L_max - L_min), 0, 1)
- R_n = clip((R - R_min)/(R_max - R_min), 0, 1)
- F_n = clip((F - F_min)/(F_max - F_min), 0, 1)

## C) Visibility Index equation
V = clip( α·L_n + β·(1 − R_n) + γ·F_n, 0, 1 )
where α + β + γ = 1.

## D) Adaptive policy form (tunable)
- T_th = clamp(T_base + kT(1 − V) + kS·v, T_min, T_max)
- R_b  = clamp(R_base + kR(1 − V) + kV·v², R_min, R_max)
- f_des = clamp(f_base + kF(1 − V), f_min, f_max)
- P = clamp(round(P_base + kP(1 − V)), P_min, P_max)

## E) Congestion control assumption
Channel Busy Ratio (CBR) ∈ [0,1] is measured by the radio stack.

A simple piecewise cap is assumed:
- if CBR ≥ CBR_HARD → f_cap = F_CAP_LOW
- else if CBR ≥ CBR_SOFT → f_cap = F_CAP_MED
- else → f_cap = F_CAP_HIGH

Actual transmit rate:
f_tx = min(f_des, f_cap)

## F) Message-related assumptions
- Vehicle periodically broadcasts a single message type `AViRS_MSG` containing:
  position, speed, heading, V, T_th, R_b, priority P, and rates (f_des, f_cap, f_tx).
- Pedestrian phone ignores stale messages older than STALE_LIMIT.
- Alerts are rate-limited with ALERT_COOLDOWN to avoid repeated spam.

## G) Sensor availability assumptions
- Wiper state/speed is available as a rain proxy.
- Camera contrast is available as a simple fog/visibility heuristic.
- Phone GNSS + IMU can estimate VRU motion sufficiently for TTC estimation.
