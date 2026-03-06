# Pseudocode (AViRS-V2P)

This folder contains pseudocode describing the logic of the AViRS-V2P system.

## Files
- `vehicle_node.pseudo`  
  Vehicle-side sensing, Visibility Index computation, adaptive TTC/rate selection, message broadcast, smart-tag warning transmission, and driver warning triggering.
- `pedestrian_node.pseudo`  
  Pedestrian-side logic for both smartphone mode and advanced reflective smart tag mode. In smartphone mode, the device receives vehicle messages, computes TTC, compares with the adaptive threshold, and triggers alerts. In smart tag mode, the tag receives a simplified warning signal and triggers vibration or LED alerts.
- `visibility_index.pseudo`  
  Normalization of L/R/F and Visibility Index computation.
- `adaptive_policy.pseudo`  
  Mapping from Visibility Index + speed to TTC threshold, broadcast rate, safety radius, and risk level.
- `congestion_control.pseudo`  
  DCC-style backoff: cap broadcast rate based on estimated channel load.
- `message_build_parse.pseudo`  
  AViRS safety message fields, message construction/parsing, and simplified warning signal handling for advanced reflective smart tags.
- `utils_geometry.pseudo`  
  Distance and relative motion helper functions (used by TTC estimation).

## Notes
- This is **concept-level** pseudocode (no hardware build required).
- Logic aligns with README sections: System Architecture (1), Functions (2), Parameters (3), Risk Model (4), and Use Case / Smart Tag support (5).
- The system supports both **smartphone-based pedestrians** and **advanced reflective smart tags** as pedestrian-side devices.
