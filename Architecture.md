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

> ### module/

| File Name                      | Description |
|-------------------------------|-------------|
| `satellite-net-device.h`      | Defines the `SatNetDevice` class, integrating PHY, MAC, and LLC layers. Supports packet transmission, control message handling, error modeling, callbacks, and link/interface information for satellite terminals and gateways. |
| `satellite-mac.h`             | Handles uplink/downlink data and control messages. Manages MAC initialization, activation, scheduling, and mode switching between Transparent and Regenerative. |
| `satellite-phy.h`             | Provides the base PHY class for satellite systems. Manages antenna gain, Tx power, noise/interference models, packet reception/transmission, and performance metrics such as SINR/C/N₀. |
| `satellite-llc.h`             | Defines the LLC layer base class (`SatLlc`) for packet encapsulation, segmentation, ARQ, and upper/lower layer data exchange. Connects NetDevice with MAC for both UT and GW implementations. |
| `satellite-packet-classifier.h` | Classifies packets into different QoS flows based on IP DSCP or control type, aiding flow scheduling and resource handling. |
| `satellite-node-info.h`       | Container for node-specific info like node ID, MAC address, and node type (UT, SAT, GW, etc.), accessible by PHY/MAC/LLC. |
| `satellite-control-message.h` | Defines all satellite control message types (e.g., ARQ ACK, CR, TBTP, C/N₀ reports). Includes base class, type tags, and message container for managing control message lifecycles. |
| `satellite-frame-conf.h`      | Contains time/frame configuration classes for BTU, slots, frames, and superframes. Enables multi-layer time-frequency resource management. |
| `satellite-frame-allocator.h` | Implements frame-level resource allocation logic. Handles CRA/RBDC/VBDC requests, waveform assignment, symbol mapping, and TBTP generation per frame. |
| `satellite-enums.h`           | Defines enums for various satellite types, node roles, message formats, and protocol settings used across the system. |

## Use a table to list important libraries and parameter definitions (Configuration). 

> ### [satellite-net-device.h](https://github.com/sns3/sns3-satellite/blob/0fc2b8c74f0d9c2b0c3ee4ed132064a40ad2daf1/model/satellite-net-device.h)

| Parameter Name (`.h`)         | Data Type                         | Description |
|-------------------------------|------------------------------------|-------------|
| `m_phy`                       | `Ptr<SatPhy>`                      | Pointer to the physical layer (PHY) component. |
| `m_mac`                       | `Ptr<SatMac>`                      | Pointer to the MAC layer component. |
| `m_llc`                       | `Ptr<SatLlc>`                      | Pointer to the LLC (Logical Link Control) layer component. |
| `m_isStatisticsTagsEnabled`   | `bool`                             | Indicates whether statistical tags are enabled (derived from the `EnableStatisticsTags` attribute). |
| `m_classifier`                | `Ptr<SatPacketClassifier>`         | Pointer to the packet classifier. |
| `m_rxCallback`                | `NetDevice::ReceiveCallback`       | Callback function triggered when a packet is received. |
| `m_promiscCallback`           | `NetDevice::PromiscReceiveCallback`| Callback for receiving packets in promiscuous mode. |
| `m_node`                      | `Ptr<Node>`                        | Pointer to the node this NetDevice is attached to. |
| `m_mtu`                       | `uint16_t`                         | Maximum Transmission Unit (MTU) in bytes. |
| `m_ifIndex`                   | `uint32_t`                         | Index of the network interface. |
| `m_address`                   | `Mac48Address`                     | MAC address of the device. |
| `m_receiveErrorModel`         | `Ptr<ErrorModel>`                  | Error model for simulating packet loss (though not actively used in the satellite module). |
| `m_nodeInfo`                  | `Ptr<SatNodeInfo>`                 | Additional satellite node-specific information. |
| `m_lastDelay`                 | `Time`                             | Last recorded packet delay, used for jitter computation. |
| `m_lastLinkDelay`             | `Time`                             | Last recorded link delay, used for link jitter computation. |

## Use a main program flow chart to explain the sequence between the program architecture and key functions. 
## Use a system architecture diagram to explain the important functional blocks and modules of the program. 
## Use a Message Sequence Chart (MSC) to explain the interactive behavior of important functional blocks and modules.
