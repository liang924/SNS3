# Purpose of the `sat-constellation-example`

This example sets up a **satellite constellation scenario** with multiple satellites (possibly including Inter-Satellite Links, ISLs) and ground nodes, running a **Constant Bit Rate (CBR) traffic flow** to validate that the overall network operates correctly.  
It is mainly used to quickly verify multi-satellite routing/forwarding, measure throughput and latency, and check whether the output files are successfully generated.

---

# What does this example do (High-level)

- Loads the specified **scenario folder (`--scenarioFolder`)** to build a simulation topology of a satellite constellation plus ground nodes (which may include ISLs and routing tables).  
- Creates one (or more) **CBR traffic flows** in the topology, transmitting with fixed packet size and fixed interval:  
  - Packet size: `--packetSize` (bytes)  
  - Sending interval: `--interval` (e.g., `20ms`, `1s`)  
- Collects statistics (throughput, latency, packet loss, etc.) and stores them in the directory specified by `--OutputPath` for later analysis or plotting.


---

# Parameter Explanation (from `--PrintHelp`)

| Parameter | Type / Example | Function |
|-----------|----------------|----------|
| `--packetSize` | Integer (bytes), default `512` | Size of each CBR packet. |
| `--interval` | Time string, default `20ms` (e.g., `20ms`, `1s`) | Packet sending interval; smaller interval = higher traffic load. |
| `--scenarioFolder` | Folder name, default `constellation-eutelsat-geo-2-sats-isls` | Specifies the **constellation scenario folder** (containing satellite/ground/link configurations and routing info). Different folders correspond to different constellations and link setups. |
| `--OutputPath` | Directory path | Directory where simulation statistics are stored. Recommended to use different subfolders for each run. |
| General arguments (`--Print*`) | `--PrintHelp`, `--PrintGlobals`, `--PrintGroups`, `--PrintTypeIds`… | Displays help or lists ns-3 types/attributes. Does not actually run the simulation. |

---
