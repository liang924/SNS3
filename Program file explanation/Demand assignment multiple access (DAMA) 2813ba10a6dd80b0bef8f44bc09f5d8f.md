# Demand assignment multiple access (DAMA)

對應程式檔: satellite-dama-entry

## DAMA Overview

![image.png](image.png)

### 1.Capacity Request

The user terminal sends a **capacity request** to the NCC based on the amount of data currently waiting in its buffer that has not yet been transmitted.

This request can be one of the following types:

- **RBDC (Rate-Based Dynamic Capacity):** Requests bandwidth based on the required data rate.
- **VBDC (Volume-Based Dynamic Capacity):** Requests bandwidth based on the amount of data pending transmission (backlog).

### 2.Schedule Resources

After receiving the capacity requests from all RCSTs, the NCC calculates and schedules the available timeslots and bandwidth based on the overall system load, user priority, and QoS level.

This step can be regarded as the **TBTP generation phase.**

👉 In **SNS3**, this corresponds to the functions `TbtpResources()` and `SatUtScheduler::DoSchedulingForRcIndex()` .

### 3.TBTP

The NCC then broadcasts the **transmission timeslot allocation table (TBTP)** to all terminals.

The TBTP instructs each RCST on:

- **When** (which superframe/frame/slot) it is allowed to transmit
- **Which carrier frequency** to use
- **What type of packet** to transmit (control or user data)

### 4.Data Transmission in Allocated Slots

The RCST transmits data to the satellite according to the timeslots specified in the TBTP, and the satellite forwards it to the ground gateway.
These transmissions follow the MF-TDMA superframe structure assigned by the NCC, ensuring that no collisions occur between terminals.

## **DAMA Components in SNS-3**

![image.png](image%201.png)

1. RCST generates a **capacity request** (RBDC or VBDC) based on its data rate or backlog.
2. **NCC** collects all incoming requests and groups them by beam, passing them to the beam schedulers for processing.
3. Each beam scheduler determines the bandwidth allocation for individual users, and the **frame allocator** maps these allocations to actual **frames** and **timeslots**.
4. The **TBTP** message is generated, containing details such as the **frame ID, slot ID, carrier frequency, and burst type**.
5. The **TBTP** is transmitted to each RCST, which then performs uplink transmissions exactly according to the assigned schedule.
6. After completing data transmission, the terminal reports its remaining **backlog** and initiates the **next request cycle**, continuing the DAMA process.

<aside>
❔

### 🔍 What is MF-TDMA?

In the **return link** of a DVB-RCS2 system, the standard specifies the use of **MF-TDMA (Multi-Frequency Time Division Multiple Access)**.

- Multiple carriers (multi-frequency) × time division → multiple users transmit in different “frequency + time-slot” combinations.
- This defines the **physical/link-layer multiple-access structure**, i.e., when each RCST (Return Channel Satellite Terminal) may transmit, on which carrier, and in which timeslot.
- In the DVB-RCS2 specification, the MF-TDMA structure forms the **fundamental framework** of the return link.

So, MF-TDMA provides the **low-level foundation that determines how resources are divided (frequency + time) and when each terminal can transmit.**

---

### 🔍 What is DAMA?

**DAMA (Demand Assignment Multiple Access)** is a **resource-allocation policy or scheduling algorithm** that operates on top of the MF-TDMA structure.

- It manages user requests based on their needs (for example, *how much backlog* they have or *what data rate* they require).
- The **NCC (Network Control Center)** then decides **which timeslots to allocate** based on those requests, system load, and QoS policies.
- In other words, DAMA defines **who transmits, when, and how much bandwidth** each user receives.
- It relies on request mechanisms such as **RBDC (Rate-Based Dynamic Capacity)** and **VBDC (Volume-Based Dynamic Capacity)** to implement dynamic scheduling.

---

### ✅ Relationship Between the Two (Different Layers, Not Different Links)

| Layer | Function | Description |
| --- | --- | --- |
| **MF-TDMA** | Physical / Link Layer | Defines the available structure of multi-frequency timeslots. |
| **DAMA** | Control / Scheduling Layer | Runs **on top of** that structure to decide how the slots are **dynamically assigned** to users. |

In simpler terms:

> MF-TDMA is the stage and props, while DAMA is the actor’s choreography on that stage.
> 
</aside>

## DAMA Configuration Types

### **RBDC（Rate-Based Dynamic Capacity)**

RBDC requests specify a rate (in kbps) that the terminal needs. The allocation is maintained until a new request is received or a timeout occurs.

Configuration in SNS-3:

```cpp
Config::SetDefault("ns3::SatLowerLayerServiceConf::DaService3_RbdcAllowed", BooleanValue(true));
Config::SetDefault("ns3::SatLowerLayerServiceConf::DaService3_MinimumServiceRate", UintegerValue(16));
```

Sources:

[**examples/sat-dama-sim-tn9.cc130-139**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-sim-tn9.cc#L130-L139)

[**examples/sat-dama-onoff-sim-tn9.cc157-165**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-onoff-sim-tn9.cc#L157-L165)

### **VBDC (Volume-Based Dynamic Capacity)**

VBDC requests specify a volume (in bytes) that the terminal needs to transmit. Once this volume is transmitted, the allocation expires.

Configuration in SNS-3:

```cpp
Config::SetDefault("ns3::SatLowerLayerServiceConf::DaService3_VolumeAllowed", BooleanValue(true));
```

Sources: [**examples/sat-dama-onoff-sim-tn9.cc169-175**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-onoff-sim-tn9.cc#L169-L175)

### **CRA (Constant Rate Assignment)**

CRA provides a fixed allocation of bandwidth regardless of usage. It's configured through a constant service rate:

```cpp
Config::SetDefault("ns3::SatLowerLayerServiceConf::DaService0_ConstantAssignmentProvided", BooleanValue(true));
Config::SetDefault("ns3::SatLowerLayerServiceConf::DaService0_ConstantServiceRate", StringValue("ns3::ConstantRandomVariable[Constant=1]"));
```

Sources: [**examples/sat-dama-onoff-sim-tn9.cc210-217**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-onoff-sim-tn9.cc#L210-L217)

## **Capacity Request Transmission Methods**

In SNS-3, capacity requests can be transmitted using different methods:

### **Periodical Control Slots**

Control slots are dedicated time slots for transmitting control messages, including capacity requests:

```cpp
Config::SetDefault("ns3::SatBeamScheduler::ControlSlotsEnabled", BooleanValue(true));
Config::SetDefault("ns3::SatBeamScheduler::ControlSlotInterval", Value("+1000000000.0ns"))
```

Sources: [**examples/sat-dama-sim-tn9.cc138**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-sim-tn9.cc#L138-L138)

 [**examples/sat-dama-onoff-sim-tn9.cc202-205**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-onoff-sim-tn9.cc#L202-L205)

 [**examples/sat-dama-http-sim-tn9.cc144-145**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-http-sim-tn9.cc#L144-L145)

### **Random Access with Slotted ALOHA**

Slotted ALOHA is a random access method where terminals transmit in predefined slots:

```cpp
Config::SetDefault("ns3::SatBeamHelper::RandomAccessModel", EnumValue(SatEnums::RA_MODEL_SLOTTED_ALOHA));
Config::SetDefault("ns3::SatBeamScheduler::ControlSlotsEnabled", BooleanValue(false));
```

Sources: [**examples/sat-dama-onoff-sim-tn9.cc188-190**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-onoff-sim-tn9.cc#L188-L190) 

[**examples/sat-dama-http-sim-tn9.cc150-152**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-http-sim-tn9.cc#L150-L152)

### **CRDSA (Contention Resolution Diversity Slotted ALOHA)**

CRDSA is an enhanced random access method with multiple replicas:

```cpp
Config::SetDefault("ns3::SatBeamHelper::RandomAccessModel", EnumValue(SatEnums::RA_MODEL_CRDSA));
Config::SetDefault("ns3::SatBeamScheduler::ControlSlotsEnabled", BooleanValue(false));
```

Sources: [**examples/sat-dama-onoff-sim-tn9.cc194-198**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-onoff-sim-tn9.cc#L194-L198) 

[**examples/sat-dama-http-sim-tn9.cc156-160**](https://github.com/sns3/sns3-satellite/blob/0fc2b8c7/examples/sat-dama-http-sim-tn9.cc#L156-L160)