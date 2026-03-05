# Pseudocode (AViRS-V2P)

This folder contains pseudocode describing the logic of the AViRS-V2P system.

## Files
- `vehicle_node.pseudo`  
  Vehicle-side sensing, Visibility Index computation, adaptive TTC/rate selection, message broadcast.
- `pedestrian_node.pseudo`  
  Pedestrian smartphone logic: receive message, compute TTC, compare with adaptive threshold, trigger alert.
- `visibility_index.pseudo`  
  Normalization of L/R/F and Visibility Index computation.
- `adaptive_policy.pseudo`  
  Mapping from Visibility Index + speed to TTC threshold, broadcast rate, safety radius, risk level.
- `congestion_control.pseudo`  
  DCC-style backoff: cap broadcast rate based on estimated channel load.
- `message_build_parse.pseudo`  
  Message fields + how vehicle builds and pedestrian parses AViRS message.
- `utils_geometry.pseudo`  
  Distance and relative motion helper functions (used by TTC estimation).

## Notes
- This is **concept-level** pseudocode (no hardware build required).
- Logic aligns with README sections: System Architecture (1), Functions (2), Parameters (3), Risk Model (4).