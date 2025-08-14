> ### [satellite-phy.md](https://github.com/sns3/sns3-satellite/blob/0fc2b8c74f0d9c2b0c3ee4ed132064a40ad2daf1/model/satellite-phy.h)

| Parameter Name | Data Type | Description |
|----------------|-----------|-------------|
| `m_rxNoiseTemperatureDbk` | `double` | Receiver noise temperature in decibels Kelvin (dBK) |
| `m_rxMaxAntennaGainDb` | `double` | Maximum gain of the receiver antenna (dB) |
| `m_rxAntennaLossDb` | `double` | Loss of the receiver antenna (dB) |
| `m_txMaxAntennaGainDb` | `double` | Maximum gain of the transmitter antenna (dB) |
| `m_txMaxPowerDbw` | `double` | Maximum transmission power of the transmitter (dBW) |
| `m_txOutputLossDb` | `double` | Output loss of the transmitter (dB) |
| `m_txPointingLossDb` | `double` | Pointing loss of the transmitter antenna (dB) |
| `m_txOboLossDb` | `double` | Output back-off (OBO) loss of the transmitter (dB) |
| `m_txAntennaLossDb` | `double` | Antenna loss on the transmitting side (dB) |
| `m_defaultFadingValue` | `double` | Default fading value applied to signal strength |
| `m_isStatisticsTagsEnabled` | `bool` | Indicates whether statistical tracing tags are enabled |
| `m_lastDelay` | `Time` | Last delay measurement for jitter calculation |
| `m_lastLinkDelay` | `Time` | Last link delay measurement for link jitter calculation |
| `m_eirpWoGainW` | `double` | Equivalent Isotropically Radiated Power (EIRP) without antenna gain (in watts) |
| `m_satId` | `uint32_t` | Satellite identifier |
| `m_beamId` | `uint32_t` | Beam identifier |
| `m_phyTx` | `Ptr<SatPhyTx>` | Pointer to transmit-side PHY layer object |
| `m_phyRx` | `Ptr<SatPhyRx>` | Pointer to receive-side PHY layer object |
| `m_nodeInfo` | `Ptr<SatNodeInfo>` | Pointer to satellite node info (type, ID, MAC, etc.) |
| `m_rxCallback` | `ReceiveCallback` | Callback function when a packet is received from PHY |
| `m_retrieveChannelPair` | `ChannelPairGetterCallback` | Callback to retrieve channel pair by beam ID |
| `m_packetTrace` | `TracedCallback` | Trace callback for tracking transmitted and received packets |
| `m_rxTrace` | `TracedCallback` | Trace callback for received packets (with source address) |
| `m_rxDelayTrace` | `TracedCallback` | Trace callback for recording receive delay info |
| `m_rxLinkDelayTrace` | `TracedCallback` | Trace callback for recording link delay info |
| `m_rxJitterTrace` | `TracedCallback` | Trace callback for recording packet jitter info |
| `m_rxLinkJitterTrace` | `TracedCallback` | Trace callback for recording link jitter info |
| `m_rxLinkModcodTrace` | `TracedCallback` | Trace callback for recording link MODCOD info |
| `m_cnoCallback` | `CnoCallback` | Callback for reporting Carrier-to-Noise density (C/N₀) |
| `m_avgNormalizedOfferedLoadCallback` | `AverageNormalizedOfferedLoadCallback` | Callback for reporting average normalized offered load |
