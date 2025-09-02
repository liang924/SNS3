
# Purpose of the `sat-cbr-example`

Performs a minimal **CBR (Constant Bit Rate) traffic test** with fixed packet size and fixed sending interval on the **sns3-satellite** topology.  
It is used to quickly verify whether the forward/return links operate correctly, to measure throughput and latency, and to check if the output files are successfully generated.


# What does this example do (High-level)

- Builds a **minimal satellite scenario**: Satellite (transponder/channel), Gateway (GW), User Terminal (UT) nodes, using `SimulationHelper/SatHelper` to load a given **scenario**.  
- Creates a **CBR flow** (Constant Bit Rate) between GW and one UT with fixed packet size and fixed sending interval.  
  - Packet size: `--packetSize` (bytes)  
  - Sending interval: `--interval` (e.g., `1s`, `500ms`)  
- Outputs collected statistics to the folder specified by `--OutputPath` for later analysis/plotting.  

> **CBR bitrate calculation**: `bitrate ≈ packetSize × 8 / interval`  
> Example: `packetSize=1024`, `interval=1s` → **8 kbps**  
> Example: `packetSize=1200`, `interval=20ms` → **480 kbps**

---

# Parameter Explanation (from `--PrintHelp`)

| Parameter | Type / Example | Function |
|-----------|----------------|----------|
| `--scenario` | `simple` / `larger` / `full` (default `simple`) | Selects the scenario topology: simple / larger / full. The choice determines number of nodes, beams/links, and default paths. Recommended to start with `simple`. |
| `--packetSize` | Integer (bytes), default `512` | Size of each CBR packet. |
| `--interval` | Time string, e.g. `1s`, `500ms` | Packet sending interval. Smaller interval = higher frequency = higher effective bitrate. |
| `--beamIdInFullScenario` | Integer (default `10`) | **Only used in `--scenario=full`**: selects which beam’s UT/GW pair to use for the CBR traffic. In full scenarios with many beams, this identifies the target beam. |
| `--OutputPath` | Directory path | Directory where statistics/results are stored. It is recommended to use different subfolders for different runs. |
| `--PrintHelp` and other General | `--PrintHelp`, `--PrintGlobals`, `--PrintGroups`, `--PrintTypeIds`… | Queries help or lists ns-3 type/attribute information. **Does not actually run the simulation.** |
