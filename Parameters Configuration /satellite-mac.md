> ### [satellite-mac.h](https://github.com/sns3/sns3-satellite/blob/0fc2b8c74f0d9c2b0c3ee4ed132064a40ad2daf1/model/satellite-mac.h)

| Parameter Name | Type | Description |
|----------------|------|-------------|
| `m_satId` | `uint32_t` | Satellite ID the MAC belongs to |
| `m_beamId` | `uint32_t` | Beam ID associated with the MAC |
| `m_nodeInfo` | `Ptr<SatNodeInfo>` | Node information (e.g., MAC address, type) |
| `m_handoverModule` | `Ptr<SatHandoverModule>` | Module responsible for managing handovers |
| `m_txEnabled` | `bool` | Whether PHY layer transmission is enabled |
| `m_beamEnabledTime` | `Time` | Last time the beam was activated |
| `m_lastDelay` | `Time` | Last measured delay (used for jitter calculation) |
| `m_lastLinkDelay` | `Time` | Last measured link delay |
| `m_isRegenerative` | `bool` | Whether this is a regenerative satellite node |
| `m_satelliteAddress` | `Address` | MAC address of the peer satellite (for regenerative link) |
| `m_ncrV2` | `bool` | Whether NCR uses version 2 (frame N-2) |
| `m_ncrMessagesToSend` | `std::queue<Ptr<SatNcrMessage>>` | Queue of NCR control messages pending transmission |
| `m_lastSOF` | `std::queue<Time>` | Last 3 Start-of-Frame timestamps used for NCR generation |
| `m_txCallback` | `TransmitCallback` | Callback for transmitting a packet to the PHY layer |
| `m_rxCallback` | `ReceiveCallback` | Callback for receiving a packet with source/destination MAC |
| `m_rxLoraCallback` | `LoraReceiveCallback` | Callback for receiving a packet (LoRa-specific) |
| `m_readCtrlCallback` | `ReadCtrlMsgCallback` | Reads control messages from control container |
| `m_reserveCtrlCallback` | `ReserveCtrlMsgCallback` | Reserves control message ID and stores it |
| `m_sendCtrlCallback` | `SendCtrlMsgCallback` | Sends a control message |
| `m_beamSchedulerCallback` | `BeamSchedulerCallback` | Retrieves the scheduler for a specific beam |
| `m_updateIslCallback` | `UpdateIslCallback` | Updates ISL routes during handover |
| `m_routingUpdateCallback` | `RoutingUpdateCallback` | Updates routing and ARP table during handover |
| `m_forwardLinkRegenerationMode` | `SatEnums::RegenerationMode_t` | Regeneration mode for forward link |
| `m_returnLinkRegenerationMode` | `SatEnums::RegenerationMode_t` | Regeneration mode for return link |
