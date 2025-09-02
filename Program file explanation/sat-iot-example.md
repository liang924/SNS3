# Purpose of the `sat-iot-example`

Sets up an **IoT-over-satellite** scenario with a single spot-beam, gateway, and user terminals, to test **narrowband return link transmissions** under constrained device power and bandwidth allocations.  
This example is useful for evaluating IoT device access to satellites, queue sizes, frequency/bandwidth allocations, and basic throughput/delay performance.

---

# What does this example do (High-level)

- Builds a satellite topology with:
  - **1 gateway (GW)**  
  - **1 beam (fixed ID)**  
  - Configurable **number of UTs per beam** and **end-users per UT**  
- Configures **queue size**, **application start time**, and **simulation length**.  
- Models **IoT device constraints**, such as:
  - Limited transmit power (`--MaxPowerTerminalW`)  
  - Narrow feeder/user link frequencies and bandwidths  
  - Frame-specific bandwidth and carrier parameters  
- Runs the simulation for a chosen duration and exports statistics to `--OutputPath`.

---

# Parameter Explanation (from `--PrintHelp`)

| Parameter | Type / Example | Function |
|-----------|----------------|----------|
| `--Beam` | Integer, e.g., `8` (default `8`) | Beam ID used for the simulation (only single beam supported). |
| `--NbGw` | Integer, default `1` | Number of gateways. |
| `--NbUtsPerBeam` | Integer, default `1` | Number of UTs per beam. |
| `--NbEndUsersPerUt` | Integer, default `1` | Number of end-user devices behind each UT. |
| `--QueueSize` | Integer, default `50` | Queue size (in packets) at the satellite. |
| `--AppStartTime` | Time string, default `+1ms` | Start time of the applications/traffic. |
| `--SimLength` | Time string, default `+1min` | Total simulation length. |
| `--MaxPowerTerminalW` | Float, default `0.3` | Maximum power of terminals in watts (models IoT device power constraint). |
| `--RtnFeederLinkBaseFrequency` | Float, default `1.77e+10` | Base frequency for the return feeder link band. |
| `--RtnUserLinkBaseFrequency` | Float, default `2.95e+10` | Base frequency for the return user link band. |
| `--RtnFeederLinkBandwidth` | Float, default `4.6848e+06` | Bandwidth of the return feeder link band. |
| `--Frame0_AllocatedBandwidthHz` | Integer, default `292800` | Allocated bandwidth (Hz) for the frame. |
| `--Frame0_CarrierAllocatedBandwidthHz` | Integer, default `292800` | Allocated carrier bandwidth (Hz) for the frame. |
| `--Frame0_CarrierRollOff` | Float, default `0.22` | Roll-off factor for the frame. |
| `--Frame0_CarrierSpacing` | Float, default `0` | Carrier spacing factor for the frame. |
| `--OutputPath` | Directory path | Directory where statistics/results are stored. |
| General (`--Print*`) | `--PrintHelp`, `--PrintGlobals`, `--PrintGroups`, `--PrintTypeIds`… | Queries help or lists ns-3 types/attributes; does not run the simulation. |

---

