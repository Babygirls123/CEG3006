# Hardware components and parameters (assumptions)

## Hardware components

### Vehicle side
- GNSS: vehicle position
- Vehicle speed/heading: CAN or onboard sensors
- Ambient light: lux sensor or camera-derived proxy
- Rain intensity proxy: wiper speed/state
- Fog/visibility proxy: camera contrast estimate
- V2X OBU: DSRC or C-V2X broadcast transmitter
- Risk + Visibility Module: computes V, thresholds, rates, safety bubble

### Pedestrian side
- Smartphone GNSS: pedestrian position
- IMU (accel/gyro): motion estimation / smoothing
- V2P receiver + parser: receives AViRS_MSG
- TTC estimator: relative trajectory + closest approach
- Alert engine: vibration + audio + UI

## Network / performance parameters (example assumptions)

| Parameter | Symbol | Example value | Notes |
|---|---|---:|---|
| Desired tx rate range | f_min..f_max | 1..10 Hz | visibility-driven |
| CBR soft/hard thresholds | CBR_SOFT / CBR_HARD | 0.4 / 0.7 | tune as assumption |
| Cap rates | F_CAP_HIGH/MED/LOW | 10 / 5 / 2 Hz | piecewise cap |
| Packet size | S_pkt | ~200 bytes | used for channel load estimate |
| Prediction horizon | horizon | 3–5 s | TTC prediction |
| Message stale limit | STALE_LIMIT | 1–2 s | ignore old packets |
| Alert cooldown | ALERT_COOLDOWN | 1–2 s | avoid alert spam |

## Adaptive policy bands (example)

| Visibility V | Interpretation | TTC threshold T_th | Bubble radius R_b | Desired rate f_des | Priority |
|---:|---|---:|---:|---:|---|
| 0.80–1.00 | clear | ~2 s | ~30 m | ~1 Hz | Low |
| 0.50–0.79 | moderate | ~3 s | ~50 m | ~5 Hz | Medium |
| 0.00–0.49 | poor | ~4 s | ~70 m | up to ~10 Hz | High |

Actual transmit rate: f_tx = min(f_des, f_cap(CBR)).
