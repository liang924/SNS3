# Return link packet scheduling

對應程式檔: http://rtn-system-test-example.cc/

```bash
┌──────────────────────────────────────────────┐
│ 程式啟動：main()                             │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 初始化預設變數與 Logger (ex.模擬時間           │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 建立 SimulationHelper (控制整個模擬流程）      │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 組成 input XML 路徑                          │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 解析命令列 (CommandLine) 並覆寫參數          │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 讀取 ConfigStore (Load XML)                  │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 設定 SuperFrame / Scheduler 預設              │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ switch(testCase)                             │
│ └── case 0..5: 設定不同 Config 屬性          │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 設定 utsPerBeam / endUsersPerUt / on-off 模式 │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 設定 simulationHelper 參數                   │
│ （UT 數量、beam 數量、simTime 等）          │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ LoadScenario("geo-33E")                      │
│ 建立 GEO 衛星配置                           │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ CreateSatScenario()                          │
│ 建立節點、裝置與協定堆疊                    │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 連接 trace callbacks                         │
│ （RBDC/VBDC/AVBDC/DaResources）             │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 選擇 trafficModel                            │
│ ├── 0 → AddCbrTraffic() (CBR 模型)           │
│ └── 1 → AddOnOffTraffic() (On-Off 模型)      │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 設定統計輸出 (SatStatsHelperContainer)        │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 印出模擬參數摘要 (NS_LOG_INFO)                │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ RunSimulation()                              │
│ 啟動事件驅動模擬（執行 TBTP 排程流程）        │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 收集統計結果與輸出檔案                        │
│ （觸發 callbacks 驗證 TBTP 結果）             │
└──────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│ 模擬結束 → main() 回傳 0                     │
└──────────────────────────────────────────────┘

```

## 模擬啟動與初始化

**SNS3 Mapping:** `main()`, `SimulationHelper`, `SatConfigStore`, `SetDefaultSuperframeConf()`

### **7.5.1 Return Link Structure (MF-TDMA)**

The simulation initializes the **Return Link** system following the MF-TDMA (Multi-Frequency Time Division Multiple Access) structure defined in DVB-RCS2.

- Each **superframe** is divided into multiple **frames**, each containing several **timeslots**.
- The **NCC (Network Control Center)** maintains synchronization for all RCSTs through timing control.
- Each RCST (terminal) can transmit only during its assigned timeslot, following NCC scheduling.
- Superframe synchronization ensures deterministic uplink burst alignment.

> 🧠 In SNS3:
> 
> 
> `SimulationHelper::SetDefaultSuperframeConf()` sets up the MF-TDMA superframe layout.
> 
> It defines the number of frequencies, slots per frame, and total duration.
> 
> This structure is used later during TBTP allocation.
> 

---

### **7.5.1.3 Frame**

Frames are scheduled according to the **superframe counter**.

Each burst opportunity is linked to `(superframe, frame_id, slot_id)`.

- Transmission timing accuracy: ±90 ms (as defined by ETSI).
- Ensures proper slot alignment to prevent collisions.

> ⏱ SNS3 models these discrete timing events inside the Simulator::Run() event scheduler.
> 

---

### **Connection to Simulation Flow**

This phase:

- Initializes the **satellite time base** and **superframe hierarchy**.
- Loads XML configs (via `SatConfigStore::LoadXml()`).
- Sets all NCC/RCST timing parameters before link scheduling starts.

## 建立模擬場景 — `CreateSatScenario()`

**SNS3 Mapping:** `SatHelper::CreateSatScenario()`, `SatNetDevice`, `SatMac`, `SatPhy`

### **7.2.4 Lower Layer Addressing by the RCST**

This section defines how the NCC and RCST nodes communicate within the return link.

- Each RCST receives a **unique MAC ID** and is mapped to a corresponding uplink carrier.
- The NCC uses logical addressing for routing and control.
- The satellite PHY layer defines **carrier frequencies**, **beam coverage**, and **timing offsets**.

> 🧩 In SNS3:
> 
> 
> `SatHelper::CreateSatScenario()` builds the physical topology and installs network devices:
> 
> - `SatNetDevice` → node abstraction for NCC/RCST
> - `SatMac` → manages TDMA access control
> - `SatPhy` → defines modulation and propagation delays

---

### **7.2 Return Link Medium Access Control**

Defines how uplink bursts are generated, identified, and multiplexed.

- MAC headers contain fields such as **RCST ID**, **slot index**, and **burst type**.
- RCSTs must comply with NCC scheduling and acknowledge receipt of TBTP updates.

> 🧠 SNS3 Implementation:
> 
> 
> The MAC layer setup here ensures later callbacks (`RbdcRcvdCb`, `VbdcRcvdCb`) can record per-node transmission state.
> 

## 連接 trace callbacks

**SNS3 Mapping:** `RbdcRcvdCb()`, `VbdcRcvdCb()`, `AvbdcRcvdCb()`, `TbtpResources()`

### **7.2.6.3 Solicitation Methods**

The NCC’s timeslot allocation process must support at least one of the following methods:

- **RBDC (Rate Based Dynamic Capacity):** resource request based on bit rate
- **VBDC (Volume Based Dynamic Capacity):** resource request based on traffic volume

Each request is defined by the pair `(RCST, Request Class)`.

The RCST may use either method depending on NCC configuration.

> 📘 Resources granted through these methods are dedicated and delivered via TBTP2 to specific RCSTs.
> 

---

### **7.2.6.3.1 VBDC – Volume Based Dynamic Capacity**

The RCST reports its pending data volume (backlog) to the NCC.

The NCC allocates uplink timeslots accordingly.

- The NCC maintains a **backlog value per request class**.
- The backlog **expires automatically** if no update is received for 1–255 superframes.
- The timeout is indicated in the Lower Layer Service Descriptor within TIM-U.
- The NCC may enforce a **maximum backlog limit** and notify the RCST.

If VBDC support is not explicitly indicated in TIM-U, the RCST assumes VBDC is **not supported**.

> **Backlog** refers to the amount of data currently waiting to be transmitted by the RCST that has not yet been allocated transmission resources by the NCC.
> 

---

### **7.2.6.3.1.1 AVBDC – Absolute Volume Based Dynamic Capacity**

- Each new AVBDC request **replaces** the previous one, resetting the backlog.
- Used when the RCST knows the current backlog at the NCC is zero.

### **7.2.6.3.1.2 IVBDC – Incremental Volume Based Dynamic Capacity**

- Each new IVBDC request **adds** to the previous volume request.
- Used for continuous data flows (e.g., streaming uploads).

---

### **7.2.6.3.2 RBDC – Rate Based Dynamic Capacity**

The RCST requests a specific transmission rate from the NCC.

- The NCC computes the allocation based on `RBDC_rate` and `max_RBDC`.
- Each request is **absolute**, overwriting the previous rate request.
- If no update arrives within the timeout (1–255 superframes), the request expires.
- If TIM-U does not enable RBDC, the RCST assumes it is **not supported**.

---

### **7.2.6.3.3 Requests per Class**

The RCST can submit requests for multiple request classes simultaneously.

Each class is identified by an `R_index`.

ETSI requires support for **at least three concurrent request classes**.

> 📘 In SNS3, this corresponds to multi-flow traffic sources (e.g., separate applications or QoS levels).
> 

---

### **7.2.6.3.4 Limitation of Requested Resources**

The RCST must not request more capacity than needed.

When fully granted, the average frame payload must not exceed 110 % of its nominal traffic.

This limit applies to:

- Explicit requests (RBDC/VBDC)
- Unsolicited allocations (CRA)
- Random access (RA)

## 🚀 Overview: Role of the Burst Time Plan (BTP)

The **BTP (Burst Time Plan)** is a transmission schedule broadcast by the **NCC (Network Control Center)** to all **RCSTs (Return Channel Satellite Terminals)**.

It specifies:

- **When** the RCST should transmit,
- **On which carrier or frequency**, and
- **What type of burst** (control or user data) should be sent.

In other words, BTP defines the **uplink scheduling plan** for the **return link**.

In SNS3, this corresponds to the **TBTP allocation logic** handled by the NCC side.

---

## 🧩 Structure and Transmission Flow

1. **BTP Broadcast by the NCC**
    - Sent through control messages: `SCT / FCT2 / BCT / TIM-U / TBTP2`.
    - `TBTP2` is the *Terminal Burst Time Plan* message containing superframe and timeslot assignment information.
2. **RCST Behavior After Receiving TBTP2**
    - Uses the `superframe_count` field to determine when to start transmitting.
    - Controls uplink burst timing, waveform selection, and transmission scheduling.

---

## ⏱ Transmission Latency Requirements

| Type | Maximum Latency | Description |
| --- | --- | --- |
| From TBTP2 reception to transmission | ≤ 90 ms | RCST must be ready to transmit within 90 ms after receiving TBTP2 (for MF-TDMA) |
| From BCT update to new waveform usage | ≤ 90 ms | Switching delay after a waveform or parameter update |
| Switching between transmission modes (including continuous carrier) | ≤ 2 s | Switching into/out of Continuous Carrier mode (CC) must not exceed 2 seconds |

---

## ⚙️ Timeslot Assignment Methods

Timeslots may be assigned to RCSTs in several ways:

- **Permanent assignment:** fixed by frame type.
- **Dynamic assignment:** defined within TBTP2.
- **Periodic assignment:** configured through the *Control Assign Descriptor* in TIM-U, which specifies:
    - the **first superframe**,
    - the **timeslot position**, and
    - the **repeat interval** (in number of superframes).

In SNS3, this corresponds to the **`TbtpResources()`** function and the NCC scheduling modules responsible for TBTP generation.
