## Use a table to list the files in each directory and briefly describe their functions. 

> ### help/

| File Name | Brief Description |
| --------- | ----------------- |
| `lora-forwarder-helper.h` | Installs LoRa forwarder on gateway nodes, supports terrestrial/satellite modes. |
| `lora-network-server-helper.h` | Sets up LoRa Network Server with ADR and P2P support. |
| `satellite-beam-helper.h` | Builds beam structures and connects UTs, GWs, and satellites; supports beam hopping, multicast, ISL. |
| `sat-beam-user-info.h` | Manages user count and location info per beam. |
| `satellite-cno-helper.h` | Configures C/N₀ values for GW/UT nodes, via fixed or time-series input. |
| `satellite-conf.h` | Manages simulation parameters like beam, TLE, and location. |
| `satellite-group-helper.h` | Creates UT groups by location or randomness for large-scale scenarios. |
| `satellite-gw-helper-dvb.h` | Sets up DVB gateway nodes with NetDevice, NCC, and links. |
| `satellite-gw-helper-lora.h` | Installs/configures LoRa satellite gateways. |
| `satellite-gw-helper.h` | General helper for gateway setup, including superframe and access model. |
| `satellite-helper.h` | Core helper to build full satellite topology including beams, GWs, UTs. |
| `satellite-isl-arbiter-unicast-helper.h` | Installs ISL unicast route management across satellites. |
| `sat-lora-conf.h` | Configures LoRaWAN MAC/PHY based on selected PHY standard. |
| `satellite-on-off-helper.h` | Installs apps generating satellite traffic with configurable rate and packet size. |
| `sat-orbiter-helper-dvb.h` | Sets up DVB-mode satellite nodes with forward/return channels and NCC link. |
| `sat-orbiter-helper-lora.h` | Sets up LoRa-mode satellite nodes with superframe and access configs. |
| `sat-orbiter-helper.h` | General tool for configuring satellite NetDevices and superframe structures. |
| `satellite-point-to-point-isl-helper.h` | Creates point-to-point ISL links with bandwidth and queue config. |
| `satellite-traffic-helper.h` | Creates bidirectional application traffic (e.g., VoIP, HTTP) for UT/GW evaluation. |
| `satellite-user-helper.h` | Deploys users, UTs, and GWs with routing setup; supports handover. |
| `satellite-ut-helper-dvb.h` | Sets up DVB UTs with NetDevice, channels, MAC, and NCC. |
| `satellite-ut-helper-lora.h` | Sets up LoRa UTs with MAC/channel and NCC integration. |
| `satellite-ut-helper.h` | General UT helper for NetDevice, MAC/PHY, and GW/NCC connection. |
| `simulation-helper.h` | Builds full satellite simulation scenario with traffic, topology, and statistics. |

## Use a table to list important libraries and parameter definitions (Configuration). 
## Use a main program flow chart to explain the sequence between the program architecture and key functions. 
## Use a system architecture diagram to explain the important functional blocks and modules of the program. 
## Use a Message Sequence Chart (MSC) to explain the interactive behavior of important functional blocks and modules.
