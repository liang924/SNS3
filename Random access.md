# 🛰️ SNS-3 Random Access (RA) Summary

## 🔹 1. System Overview
- **Purpose:** Enables User Terminals (UTs) to transmit data **without pre-assigned timeslots**.
- **Difference from DAMA:** RA operates independently—UTs decide when to transmit based on protocol rules, while DAMA relies on NCC scheduling.
- **Use Cases:**
  - Initial logon procedure
  - Sending control or capacity request messages
  - Small or infrequent data transmissions
  - Requesting capacity before switching to DAMA

---

## 🔹 2. Random Access Schemes

| Scheme | Description | Best Use Case |
|--------|--------------|---------------|
| **Slotted ALOHA** | UT randomly selects timeslots; collisions trigger retries | Light traffic |
| **CRDSA** (Contention Resolution Diversity Slotted ALOHA) | Multiple replicas per packet; uses interference cancellation | Medium load, high reliability |
| **ESSA** (Enhanced Spread Spectrum ALOHA) | Asynchronous access; packets identified by unique ID | High load or asynchronous scenarios |

💡 **CRDSA** provides the best balance between throughput and reliability.

---

## 🔹 3. Key Components

| Component | Responsibility |
|------------|----------------|
| `SatUtMac` | UT-side MAC layer operations, including RA transmissions |
| `SatRandomAccess` | Configures and manages RA methods |
| `SatQueue` | Handles queue events that can trigger RA |
| `SatPhyRxCarrierConf` | Detects packet reception and collisions |

---

## 🔹 4. Process Flow
1. UT prepares a packet for transmission.  
2. RA mechanism (ALOHA / CRDSA / ESSA) determines slot selection.  
3. If collision occurs, UT enters a **backoff** phase and retries later.  
4. Successful transmissions (e.g., capacity requests) trigger DAMA resource allocation.

---

## 🔹 5. Integration with DAMA
RA often acts as the **initial phase** of DAMA:
1. UT sends a **capacity request** using RA.  
2. NCC allocates **dedicated bandwidth** (DAMA mode).  
3. Regular data flows use DAMA slots.  
4. Small, sporadic traffic may continue using RA for efficiency.

---

## 🔹 6. Key Configuration Parameters

| Parameter | Description | Default |
|------------|--------------|----------|
| `RandomAccessModel` | Specifies RA model | OFF |
| `RaCollisionModel` | Collision detection method (SINR-based) | SINR check |
| `RaService0_BackOffTimeInMilliSeconds` | Backoff time | 50 ms |
| `RaService0_NumberOfInstances` | CRDSA replica count | 3 |
| `EnableRandomAccessDynamicLoadControl` | Enables adaptive load control | False |

---

## 🔹 7. Dynamic Load Control
SNS-3 supports **dynamic adjustment** of RA parameters based on network load:
- When offered load exceeds threshold:
  - Increase **backoff probability**
  - Extend **backoff duration**
- Prevents congestion collapse during heavy traffic.

Key parameters:
- `EnableRandomAccessDynamicLoadControl`
- `AverageNormalizedOfferedLoadWindowSize`
- `HighLoadBackOffProbability`

---

## 🔹 8. Performance Optimization

| Factor | Recommendation |
|---------|----------------|
| **Scheme Selection** | ALOHA → low load / CRDSA → medium / ESSA → asynchronous |
| **CRDSA Replicas** | 2–3 replicas for balance between success rate and resource use |
| **Backoff Parameters** | Longer backoff = fewer collisions but higher latency; higher probability = less load but lower efficiency |

---

## 🔹 9. Summary
The **Random Access module in SNS-3** provides flexible, unscheduled uplink access mechanisms for satellite networks.  
It supports multiple schemes (ALOHA, CRDSA, ESSA), integrates with DAMA, and includes **dynamic load control** for optimal performance.

**Key Features:**
- Dynamic load control  
- SINR-based collision model  
- Event-driven triggering (queue, control, logon)  

