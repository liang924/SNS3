> ### [satellite-packet-classifier.h](https://github.com/sns3/sns3-satellite/blob/0fc2b8c74f0d9c2b0c3ee4ed132064a40ad2daf1/model/satellite-packet-classifier.h)
| Parameter Name | Data Type | Description |
|----------------|-----------|-------------|
| `Classify(SatControlMsgTag::SatControlMsgType_t type, const Address& dest)` | `uint8_t` | Classifies control messages into a flow ID based on message type and destination. |
| `Classify(const Ptr<Packet> packet, const Address& dest, uint16_t protocolNumber)` | `uint8_t` | Classifies packets into flow IDs using the IP header’s DSCP field. |
