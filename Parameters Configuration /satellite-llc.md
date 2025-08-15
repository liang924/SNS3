> ### [satellite-llc.md](https://github.com/sns3/sns3-satellite/blob/0fc2b8c74f0d9c2b0c3ee4ed132064a40ad2daf1/model/satellite-llc.h)
| Parameter Name | Data Type | Description |
|----------------|-----------|-------------|
| `m_packetTrace` | `TracedCallback` | Callback function for packet tracing |
| `m_nodeInfo` | `Ptr<SatNodeInfo>` | Stores node-specific information such as type, ID, and MAC address |
| `m_encaps` | `EncapContainer_t` | Mapping of encapsulator objects |
| `m_decaps` | `EncapContainer_t` | Mapping of decapsulator objects |
| `m_fwdLinkArqEnabled` | `bool` | Enables ARQ (Automatic Repeat reQuest) for forward link |
| `m_rtnLinkArqEnabled` | `bool` | Enables ARQ for return link |
| `m_gwAddress` | `Mac48Address` | MAC address of the gateway (GW) |
| `m_satelliteAddress` | `Mac48Address` | MAC address of the satellite (used in regenerative mode) |
| `m_additionalHeaderSize` | `uint32_t` | Additional header size used during encapsulation/decapsulation |
| `m_rxCallback` | `ReceiveCallback` | Callback function for delivering received packets to upper layers |
| `m_readCtrlCallback` | `ReadCtrlMsgCallback` | Callback for reading control messages |
| `m_sendCtrlCallback` | `SatBaseEncapsulator::SendCtrlCallback` | Callback for sending control messages (used by encapsulator only) |
| `m_forwardLinkRegenerationMode` | `SatEnums::RegenerationMode_t` | Regeneration mode for the forward link |
| `m_returnLinkRegenerationMode` | `SatEnums::RegenerationMode_t` | Regeneration mode for the return link |
