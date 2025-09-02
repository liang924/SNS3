# Purpose of the `sat-ncr-example`

This example simulates a **Network Clock Recovery (NCR)** scenario in a satellite system, focusing on the impact of **UT (User Terminal) clock drift** and **guard time** on the return link.  
It is designed to test how multiple UTs transmitting simultaneously affect packet delay, synchronization errors, and overall system performance, with results exported for further analysis.

---

# What does this example do (High-level)

- Builds a scenario with a **single beam** containing a configurable number of **UTs and end users**.  
- Each UT sends fixed-size **UDP packets** at a defined interval:  
  - Packet size: `--packetSize` (bytes)  
  - Packet interval: `--interval` (e.g., `100ms`)  
- The system simulates **UT clock drift**, while **guard time** is introduced to reduce collisions and interference.  
- Runs for the specified simulation time and stores performance data in `--OutputPath`.

---

# Parameter Explanation (from `--PrintHelp`)

| Parameter | Type / Example | Function |
|-----------|----------------|----------|
| `--simLength` | Integer (seconds), default `60` | Total simulation duration. |
| `--beamId` | Integer, default `1` | ID of the beam used. |
| `--utsPerBeam` | Integer, default `10` | Number of UTs per beam. |
| `--endUsersPerUt` | Integer, default `1` | Number of end users behind each UT. |
| `--packetSize` | Integer (bytes), default `512` | Size of each UDP packet. |
| `--interval` | Time string (e.g., `100ms`), default `100ms` | Interval between packet transmissions per UT. |
| `--guardTime` | Integer (symbols), default `4` | Guard time (in symbols) to mitigate interference from clock drift. |
| `--clockDrift` | Integer (ticks/second), default `50` | Drift rate of UT clocks. |
| `--OutputPath` | Directory path | Directory where simulation statistics are stored. |
| General (`--Print*`) | `--PrintHelp`, `--PrintGlobals`, `--PrintGroups`, `--PrintTypeIds`… | Displays ns-3 help or type information; does not run the simulation. |

---

