# Purpose of the `sat-lora-example`

This example sets up a **LoRa-over-satellite** scenario to evaluate IoT-style low-power, long-range communications in a satellite environment.  
It focuses on **uplink transmissions from many UTs (IoT devices)** using LoRa modulation and timing windows, modeling **acknowledgment delays**, **receive windows**, and **packet collisions** under constrained bandwidth.

---

# What does this example do (High-level)

- Builds a scenario with a configurable number of **UTs per beam** (default 100).  
- Each UT periodically transmits **small LoRa packets**:  
  - Packet size: `--packetSize` (bytes, default 24)  
  - Interval: `--loraInterval` (default +10s)  
- Models **LoRa physical parameters**: spreading factor, roll-off, carrier spacing, and bandwidth allocations.  
- Implements **receive windows** and **acknowledgment timings** for end devices and gateways:  
  - First and second receive windows with configurable delays and durations  
  - Corresponding gateway answer delays for acknowledgments  
- Optionally displays traces and models interference per packet.  
- Simulation runs for the configured length and writes statistics to `--OutputPath`.

---

# Parameter Explanation (from `--PrintHelp`)

| Parameter | Type / Example | Function |
|-----------|----------------|----------|
| `--modelPP` | `true` / `false`, default `true` | Enable interference modeling per packet. |
| `--traces` | `true` / `false`, default `true` | Enable trace display/logging. |
| `--utsPerBeam` | Integer, default `100` | Number of UTs (LoRa end devices) per beam. |
| `--simLength` | Time string, default `+15s` | Simulation duration. |
| `--packetSize` | Integer (bytes), default `24` | Size of each transmitted LoRa packet. |
| `--loraInterval` | Time string, default `+10s` | Interval between two transmissions per UT. |
| `--frameAllocatedBandwidthHz` | Integer (Hz), default `15000` | Allocated bandwidth for the frame. |
| `--frameCarrierAllocatedBandwidthHz` | Integer (Hz), default `15000` | Allocated carrier bandwidth for the frame. |
| `--frameCarrierRollOff` | Float, default `0.22` | Roll-off factor of the carrier. |
| `--frameCarrierSpacing` | Float, default `0` | Carrier spacing factor. |
| `--frameSpreadingFactor` | Integer, default `256` | LoRa spreading factor (chips per symbol). |
| `--firstWindowDelay` | Time string, default `+1.5s` | Delay between transmission end and opening of the first receive window on the end device. |
| `--secondWindowDelay` | Time string, default `+2s` | Delay between transmission end and opening of the second receive window. |
| `--firstWindowDuration` | Time string, default `+400ms` | Duration of the first receive window on the end device. |
| `--secondWindowDuration` | Time string, default `+400ms` | Duration of the second receive window. |
| `--firstWindowAnswerDelay` | Time string, default `+1s` | Delay between end of reception and start of acknowledgment on the gateway (first window). |
| `--secondWindowAnswerDelay` | Time string, default `+2s` | Delay for acknowledgment on the gateway (second window). |
| `--OutputPath` | Directory path | Directory where simulation statistics are stored. |
| General (`--Print*`) | `--PrintHelp`, `--PrintGlobals`, `--PrintGroups`, `--PrintTypeIds`… | Prints ns-3 help/type information; does not run the simulation. |

---

