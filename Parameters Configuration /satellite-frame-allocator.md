> ### [satellite-frame-allocator.h](https://github.com/sns3/sns3-satellite/blob/0fc2b8c74f0d9c2b0c3ee4ed132064a40ad2daf1/model/satellite-frame-allocator.h)

| Parameter Name                 | Data Type                                | Description                                                     |
|-------------------------------|-------------------------------------------|-----------------------------------------------------------------|
| `m_totalSymbolsInFrame`       | `double`                                  | Total number of available symbols in the frame                 |
| `m_availableSymbolsInFrame`   | `double`                                  | Currently remaining symbols in the frame                       |
| `m_preAllocatedCraSymbols`    | `double`                                  | Pre-allocated CRA type symbols                                 |
| `m_preAllocatedMinRdbcSymbols`| `double`                                  | Pre-allocated minimum RBDC type symbols                        |
| `m_preAllocatedRdbcSymbols`   | `double`                                  | Pre-allocated RBDC type symbols                                |
| `m_preAllocatedVdbcSymbols`   | `double`                                  | Pre-allocated VBDC type symbols                                |
| `m_maxSymbolsPerCarrier`      | `double`                                  | Maximum available symbols per carrier                          |
| `m_maxCarrierCount`           | `uint16_t`                                | Maximum number of carriers in the frame                        |
| `m_carriersOffset`            | `uint16_t`                                | Number of carriers skipped in the current frame                |
| `m_configType`                | `SatSuperframeConf::ConfigType_t`         | Frame configuration type                                       |
| `m_frameId`                   | `uint8_t`                                 | Identifier of the frame                                        |
| `m_burstLenghts`              | `SatWaveformConf::BurstLengthContainer_t` | Configured burst lengths used                                 |
| `m_waveformConf`              | `Ptr<SatWaveformConf>`                    | Pointer to waveform configuration                             |
| `m_frameConf`                 | `Ptr<SatFrameConf>`                       | Pointer to frame configuration                                |
| `m_parent`                    | `Ptr<SatFrameAllocator>`                  | Pointer to parent allocator (if this is a sub-frame)          |
| `m_utAllocs`                  | `UtAllocContainer_t`                      | Stores user terminal (UT) allocation data                     |
| `m_rcAllocs`                  | `RcAllocContainer_t`                      | Stores resource class (RC) allocation data                    |
| `m_mostRobustWaveform`        | `Ptr<SatWaveform>`                        | Most robust supported waveform                                |
| `m_guardTimeSymbols`          | `uint8_t`                                 | Number of guard symbols between slots                         |
| `m_allocationDenied`          | `bool`                                    | Whether allocation for this frame is denied                  |
