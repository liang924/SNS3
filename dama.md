<img width="271" height="552" alt="image" src="https://github.com/user-attachments/assets/45b1b6d8-4191-4a50-8adf-d80f971adc71" />

- SatDamaEntry/CR update – Process the received Capacity Requests (CR) within the previous superframe.
- Preliminary resource allocation – Pre-allocate a set of soft-symbols for each UT based on configured CRA, dynamic request type (RBDC, VBDC) and value, CNo conditions and frame configurations and load.
- Time slot generation – generate the time slots for each frame based on the pre-allocated soft-symbols for each UT and RC index. Fill in the TBTP on-the-fly.
- SatDamaEntry update – Update the allocated VBDC bytes for each UT context
- TBTP signaling – Send the TBTP message to the proper GW protocol stack handling the resources for this specific spot-beam.
- Schedule next scheduling time for the next SF.
