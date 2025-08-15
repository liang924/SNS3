> ### [satellite-node-info.h](https://github.com/sns3/sns3-satellite/blob/0fc2b8c74f0d9c2b0c3ee4ed132064a40ad2daf1/model/satellite-node-info.h)

| Name             | Type                               | Description                                                                |
|------------------|------------------------------------|----------------------------------------------------------------------------|
| `m_nodeId`       | `uint32_t`                         | Unique identifier for the node                                             |
| `m_nodeType`     | `SatEnums::SatNodeType_t`          | Node type (e.g., UT, SAT, GW, NCC, TER) defined via enumeration            |
| `m_macAddress`   | `Mac48Address`                     | MAC address of the node, used for network-level identification             |
