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
