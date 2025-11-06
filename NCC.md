# 🛰️ Network Control Center (NCC) — Key Summary

## 🔹 1. Core Role
The **Network Control Center (NCC)** is the **central management entity** for satellite resource allocation in SNS-3.  
It is responsible for:
- Receiving and processing **capacity requests (CRs)** from user terminals (UTs).  
- Making bandwidth and timeslot allocation decisions using the **DAMA (Demand Assigned Multiple Access)** mechanism.  
- Coordinating multiple **beam schedulers** to ensure fair resource allocation.  
- Optimizing allocations based on **C/N₀ (Carrier-to-Noise density)**, **QoS class**, and **traffic demand**.

---

## 🔹 2. System Architecture

| Component | Function |
|------------|-----------|
| `SatNcc` | Core NCC controller, manages all beams and UTs |
| `SatBeamScheduler` | Manages resource allocation for a single beam |
| `SatDamaEntry` | Tracks resource usage for each UT |
| `SatLowerLayerServiceConf` | Defines service configurations for different traffic classes |

➡️ NCC creates one **beam scheduler per beam** and routes UT requests to the corresponding scheduler by beam ID.

---

## 🔹 3. DAMA-Based Resource Management

NCC uses the **DAMA protocol** to dynamically manage and allocate satellite capacity.  
Each capacity request type serves different traffic needs and persistence levels:

| Type | Description | Persistence | Config Parameter |
|------|--------------|--------------|------------------|
| **CRA** | Constant Rate Assignment (fixed bandwidth) | Permanent | `DaConstantServiceRateInKbps` |
| **RBDC** | Rate-Based Dynamic Capacity | Configurable | `DynamicRatePersistence` |
| **VBDC** | Volume-Based Dynamic Capacity (based on backlog) | Configurable | `VolumeBacklogPersistence` |
| **AVBDC** | Absolute Volume-Based Dynamic Capacity | None | N/A |

---

## 🔹 4. Scheduling and TBTP Generation

The **beam scheduler** executes periodic allocation every **superframe**:
1. Collects UT capacity requests.  
2. Updates corresponding **DAMA entries**.  
3. Performs resource allocation (bandwidth, timeslots, MODCOD).  
4. Generates **TBTP (Terminal Burst Time Plan)** messages.  
5. Sends TBTP to UTs defining when and where to transmit.

> 📡 **TBTP = “who transmits, when, on which frequency.”**

---

## 🔹 5. Control Message Flow

| Message | Direction | Purpose |
|----------|------------|----------|
| **CR (Capacity Request)** | UT → NCC | Bandwidth request |
| **TBTP** | NCC → UT | Timeslot allocation |
| **CNO** | UT → NCC | Report channel quality (C/N₀) |
| **RA (Random Access)** | NCC → UT | Control random access/backoff parameters |
| **TIMU** | NCC → UT | Provide UT info (beam ID, satellite ID) |

These messages maintain synchronization and coordination between NCC and terminals.

---

## 🔹 6. Channel Quality (C/N₀) Management
- NCC tracks C/N₀ reports from UTs.  
- Beam schedulers use these values to select appropriate **MODCOD** schemes.  
- This ensures adaptive coding/modulation and optimal throughput.

---

## 🔹 7. Key Classes and Responsibilities

| Class | Role | Key Methods |
|--------|------|-------------|
| `SatNcc` | Central control entity | `AddBeam()`, `AddUt()`, `UtCrReceived()`, `UtCnoUpdated()` |
| `SatBeamScheduler` | Beam-level scheduler | `Schedule()`, `UpdateDamaEntriesWithReqs()`, `Send()` |
| `SatDamaEntry` | UT resource tracking | `GetRbdcInKbps()`, `GetVbdcInBytes()`, `UpdateRbdcInKbps()` |
| `SatRequestManager` | UT-side request generation | `DoEvaluation()`, `DoRbdc()`, `DoVbdc()` |
| `SatTbtpMessage` | TBTP message structure | `SetDaTimeslot()`, `GetDaTimeslots()`, `GetSizeInBytes()` |
| `SatCrMessage` | Encapsulates capacity requests | `AddControlElement()`, `GetCapacityRequestContent()` |

---

## 🔹 8. Random Access Control
- NCC monitors **random access channel load**.  
- Adjusts **backoff probability and delay** dynamically to prevent congestion.  
- Ensures fairness and stability in random access operations.

---

## 🔹 9. Handover Support
- NCC supports seamless **handover between beams**.  
- When a UT switches beams:
  - Its **DAMA entry** and related data are transferred to the new beam scheduler.  
  - The service continuity is preserved without disconnection.

---

## 🔹 10. Summary

The **NCC** is the **central brain** of the SNS-3 satellite system:
- Handles all **capacity requests** and **resource scheduling**.  
- Tracks resource usage via **DAMA entries**.  
- Generates **TBTP messages** to inform UTs of transmission schedules.  
- Dynamically adapts allocation to QoS, C/N₀, and user demand.

> 🧠 **In essence:**  
> NCC dynamically manages “who transmits, when, and with how much bandwidth,” ensuring efficient multi-beam coordination across the entire satellite system.

