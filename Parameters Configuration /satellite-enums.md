> ### [satellite-enums.h](https://github.com/sns3/sns3-satellite/blob/0fc2b8c74f0d9c2b0c3ee4ed132064a40ad2daf1/model/satellite-enums.h)

#### Node Types (SatNodeType_t)
| Parameter Name   | Data Type       | Description            |
| ---------------- | --------------- | ---------------------- |
| `NODE_TYPE_NONE` | `SatNodeType_t` | Undefined node type    |
| `NODE_TYPE_UT`   | `SatNodeType_t` | User Terminal (UT)     |
| `NODE_TYPE_SAT`  | `SatNodeType_t` | Satellite node         |
| `NODE_TYPE_GW`   | `SatNodeType_t` | Gateway node           |
| `NODE_TYPE_NCC`  | `SatNodeType_t` | Network Control Center |
| `NODE_TYPE_TER`  | `SatNodeType_t` | Terrestrial node       |

#### QoS Types (SatQosType_t)
| Parameter Name             | Data Type      | Description          |
| -------------------------- | -------------- | -------------------- |
| `QOS_BEST_EFFORT`          | `SatQosType_t` | Best effort service  |
| `QOS_ASSURED_FORWARDING`   | `SatQosType_t` | Assured forwarding   |
| `QOS_EXPEDITED_FORWARDING` | `SatQosType_t` | Expedited forwarding |

#### MAC Queue State (SatMacQueueState_t)
| Parameter Name            | Data Type            | Description                |
| ------------------------- | -------------------- | -------------------------- |
| `QUEUE_STATE_ENQUEUING`   | `SatMacQueueState_t` | Queue is currently filling |
| `QUEUE_STATE_READY`       | `SatMacQueueState_t` | Ready for transmission     |
| `QUEUE_STATE_TRANSMITTED` | `SatMacQueueState_t` | Transmission completed     |

#### Uplink Scheduling Type (SatMacQueueSchedType_t)
| Parameter Name             | Data Type                | Description                |
| -------------------------- | ------------------------ | -------------------------- |
| `QUEUE_SCHED_TYPE_DEFAULT` | `SatMacQueueSchedType_t` | Default scheduling         |
| `QUEUE_SCHED_TYPE_AF`      | `SatMacQueueSchedType_t` | Assured Forwarding         |
| `QUEUE_SCHED_TYPE_EF`      | `SatMacQueueSchedType_t` | Expedited Forwarding       |
| `QUEUE_SCHED_TYPE_CONTROL` | `SatMacQueueSchedType_t` | Control message scheduling |

#### MAC Header Type (SatMacHeaderType_t)
| Parameter Name            | Data Type            | Description    |
| ------------------------- | -------------------- | -------------- |
| `MAC_HEADER_TYPE_DATA`    | `SatMacHeaderType_t` | Data header    |
| `MAC_HEADER_TYPE_CONTROL` | `SatMacHeaderType_t` | Control header |

#### Delay Type (SatDelayType_t)
| Parameter Name          | Data Type        | Description      |
| ----------------------- | ---------------- | ---------------- |
| `DELAY_TYPE_ONE_WAY`    | `SatDelayType_t` | One-way delay    |
| `DELAY_TYPE_LINK`       | `SatDelayType_t` | Link delay       |
| `DELAY_TYPE_END_TO_END` | `SatDelayType_t` | End-to-end delay |

#### Packet Type (SatPacketType_t)
| Parameter Name        | Data Type         | Description    |
| --------------------- | ----------------- | -------------- |
| `PACKET_TYPE_DATA`    | `SatPacketType_t` | Data packet    |
| `PACKET_TYPE_CONTROL` | `SatPacketType_t` | Control packet |

#### MAC Scheduler Type (SatMacSchedType_t)
| Parameter Name    | Data Type           | Description                   |
| ----------------- | ------------------- | ----------------------------- |
| `SCHED_TYPE_RR`   | `SatMacSchedType_t` | Round Robin                   |
| `SCHED_TYPE_FAIR` | `SatMacSchedType_t` | Fair scheduling               |
| `SCHED_TYPE_PF`   | `SatMacSchedType_t` | Proportional Fair             |
| `SCHED_TYPE_TDMA` | `SatMacSchedType_t` | Time Division Multiple Access |

#### Statistics Tag Type (SatStatsTagType_t)
| Parameter Name           | Data Type           | Description              |
| ------------------------ | ------------------- | ------------------------ |
| `STATS_TAG_QOS`          | `SatStatsTagType_t` | QoS-related statistics   |
| `STATS_TAG_DELAY`        | `SatStatsTagType_t` | Delay-related statistics |
| `STATS_TAG_ENQUEUE_TIME` | `SatStatsTagType_t` | Packet enqueue time tag  |

