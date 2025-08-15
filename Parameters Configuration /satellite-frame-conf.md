> ### [satellite-frame-conf.h](https://github.com/sns3/sns3-satellite/blob/0fc2b8c74f0d9c2b0c3ee4ed132064a40ad2daf1/model/satellite-frame-conf.h)

## SatBtuConf Parameters

| Name                        | Type         | Description                                         |
|-----------------------------|--------------|-----------------------------------------------------|
| `m_allocatedBandwidthInHz` | `double`     | Total allocated bandwidth for the BTU in Hertz     |
| `m_occupiedBandwidthInHz`  | `double`     | Occupied bandwidth of the BTU in Hertz             |
| `m_effectiveBandwidthInHz` | `double`     | Effective bandwidth (symbol rate) in Hertz         |
| `m_spreadingFactor`        | `double`     | Spreading factor used in modulation                |
| `m_duration`               | `Time`       | Reserved duration field (currently unused)         |

---

## SatTimeSlotConf Parameters

| Name               | Type                               | Description                                            |
|--------------------|------------------------------------|--------------------------------------------------------|
| `m_startTime`      | `Time`                             | Start time of the time slot within a frame             |
| `m_waveFormId`     | `uint32_t`                         | Waveform ID used in this time slot                     |
| `m_frameCarrierId` | `uint16_t`                         | Carrier ID the time slot is mapped to                 |
| `m_rcIndex`        | `uint8_t`                          | Resource coding (RC) index of the time slot            |
| `m_slotType`       | `SatTimeSlotConf::SatTimeSlotType_t` | Type of time slot: Control, Traffic, or Both         |

---

## SatFrameConf Parameters

| Name                            | Type                                   | Description                                               |
|----------------------------------|----------------------------------------|-----------------------------------------------------------|
| `m_bandwidthHz`                | `double`                               | Bandwidth of the frame in Hertz                           |
| `m_duration`                   | `Time`                                 | Duration of the frame                                     |
| `m_isRandomAccess`             | `bool`                                 | Whether this frame is a random access frame               |
| `m_isLogon`                    | `bool`                                 | Whether this frame is for logon                           |
| `m_parent`                     | `Ptr<SatFrameConf>`                    | Parent frame if this is a subdivided frame                |
| `m_btuConf`                    | `Ptr<SatBtuConf>`                      | BTU configuration                                         |
| `m_waveformConf`              | `Ptr<SatWaveformConf>`                | Waveform configuration                                    |
| `m_allocationChannel`          | `uint8_t`                              | ID of allocation channel                                  |
| `m_carrierCount`               | `uint16_t`                             | Number of carriers in the frame                           |
| `m_maxSymbolsPerCarrier`       | `uint32_t`                             | Maximum number of symbols per carrier                     |
| `m_minPayloadPerCarrierInBytes`| `uint32_t`                             | Minimum payload size per carrier in bytes                 |
| `m_timeSlotConfMap`            | `std::map<uint16_t, SatTimeSlotConfContainer_t>` | Maps carrier IDs to their time slots       |
| `m_guardTimeSymbols`           | `uint8_t`                              | Number of guard time symbols used                         |

---

## SatSuperframeConf Parameters

| Name                               | Type                          | Description                                                   |
|------------------------------------|-------------------------------|---------------------------------------------------------------|
| `m_usedBandwidthHz`               | `double`                      | Total allocated bandwidth for the superframe                  |
| `m_duration`                      | `Time`                        | Total duration of the superframe                              |
| `m_frameCount`                    | `uint8_t`                     | Number of frames in the superframe                            |
| `m_configType`                    | `ConfigType_t`                | Superframe configuration type                                 |
| `m_maxCarrierSubdivision`         | `uint8_t`                     | Maximum allowed carrier subdivisions                          |
| `m_frameAllocatedBandwidth`       | `double[]`                    | Bandwidth allocated per frame                                 |
| `m_frameCarrierAllocatedBandwidth`| `double[]`                    | Bandwidth allocated per carrier in each frame                 |
| `m_frameCarrierSpacing`           | `double[]`                    | Carrier spacing per frame                                     |
| `m_frameCarrierRollOff`           | `double[]`                    | Roll-off factor per frame                                     |
| `m_frameCarrierSpreadingFactor`   | `uint32_t[]`                  | Spreading factor per frame                                    |
| `m_frameIsRandomAccess`           | `bool[]`                      | Flags indicating random access per frame                      |
| `m_frameIsLogon`                  | `bool[]`                      | Flags indicating logon per frame                              |
| `m_frameAllocationChannel`        | `uint8_t[]`                   | Allocation channel ID per frame                               |
| `m_frameGuardTimeSymbols`         | `uint8_t[]`                   | Guard time symbols per frame                                  |
| `m_frames`                        | `SatFrameConfList_t`          | List of frame configuration objects                           |
| `m_raChannels`                    | `std::vector<RaChannelInfo_t>`| Info about random access channels                             |
| `m_carrierCount`                  | `uint32_t`                    | Total number of carriers in the superframe                    |
| `m_logonEnabled`                  | `bool`                        | Indicates if logon is enabled                                 |
| `m_logonChannelIndex`             | `uint32_t`                    | Index of the logon channel                                    |

