
# Purpose of the `sat-regeneration-example`

Performs a configurable satellite communication test with **different regeneration modes** on both forward and return links.  
It allows comparison between **transparent (bent-pipe)** and **regenerative (PHY / Link / Network)** satellite payloads.  
This example is used to study how different regeneration strategies affect throughput, latency, error correction, and resource allocation in satellite networks.

---

# What does this example do (High-level)

- Builds a satellite scenario: Satellite node (with payload model), Gateway (GW), and User Terminals (UTs) using `SimulationHelper/SatHelper`.  
- Creates a **CBR flow** between GW and UT with configurable packet size and interval.  
  - Packet size: `--packetSize` (bytes)  
  - Sending interval: `--interval` (e.g., 10ms, 1s)  
- Allows the user to choose regeneration strategies for both **forward** and **return** links:  
  - Transparent (bent-pipe, simple forwarding)  
  - Regeneration at PHY, Link, or Network layer  
- Outputs statistics (throughput, latency, packet loss, etc.) to the folder specified by `--OutputPath` for analysis.  

> **CBR bitrate calculation**: `bitrate ≈ packetSize × 8 / interval`  
> Example: `packetSize=512`, `interval=10ms` → **409.6 kbps**  
> Example: `packetSize=1024`, `interval=1s` → **8 kbps**

---

# Parameter Explanation (from `--PrintHelp`)

| Parameter | Type / Example | Function |
|-----------|----------------|----------|
| `--scenario` | `simple` / `larger` / `full` (default `simple`) | Selects the scenario topology: defines number of nodes, beams/links, and default paths. Recommended to start with `simple`. |
| `--packetSize` | Integer (bytes), default `512` | Size of each CBR packet. |
| `--interval` | Time string (e.g., `10ms`, `1s`), default `10ms` | Interval between packets. Smaller interval = higher sending frequency = higher effective bitrate. |
| `--beamIdInFullScenario` | Integer (default `10`) | Only used when `--scenario=full`: selects which beam’s UT/GW pair to use for the traffic. |
| `--forwardRegeneration` | `transparent` / `regeneration_phy` / `regeneration_network` (default `regeneration_network`) | Sets the regeneration mode on the forward link. |
| `--returnRegeneration` | `transparent` / `regeneration_phy` / `regeneration_link` / `regeneration_network` (default `regeneration_network`) | Sets the regeneration mode on the return link. |
| `--OutputPath` | Directory path | Directory where simulation statistics are stored. Recommended to use different subfolders for each run. |
| `--PrintHelp` and other General | `--PrintHelp`, `--PrintGlobals`, `--PrintGroups`, `--PrintTypeIds`… | Queries help or lists ns-3 type/attribute information. Does not actually run the simulation. |
