# Model Description
## Frame configuration
### Return Link
- Uses **Multi-Frequency Time Division Multiple Access (MF-TDMA)**.
- Composed of:
  - Superframe sequences
  - Superframes
  - Frames
  - Time slots
  - Bandwidth Time Units (BTUs)
- Frame structures are dynamically configured by **Network Control Center (NCC)** using:
  - **Superframe Composition Table (SCT)**
  - **Frame Composition Table v2 (FCT2)**
  - **Broadcast Composition Table (BCT)**
- Satellite module does not explicitly model SCT, FCT, BCT.
- Configurations can be changed via **parameterization**.
- Follows **DVB-RCS2** guidelines (section 5.2.3).
- **TBTPv2** (Terminal Burst Time Plan v2) is implemented:
  - Dynamically configures time slots (start time, duration, waveform, etc.)

### Forward Link
- Uses **DVB-S2 Time Division Multiplexing (TDM)**.
- 2 GHz forward feeder link bandwidth is:
  - Divided into **16 carriers**, each **125 MHz**.
  - Each carrier is **statically mapped** to a user link frequency color.
  - Only **one carrier per beam** is supported.

## Architecture
<img width="720" height="325" alt="image" src="https://github.com/user-attachments/assets/4599ea4d-a7f8-4f2b-b98b-b80486b138e2" />

### 1️⃣ **End User (Terrestrial User)**

* **Protocol Stack:**
  * **Application Layer:** User-facing applications (e.g., voice, video, data transmission)
  * **Transport Layer:** Uses TCP/UDP to ensure reliable or real-time transmission
  * **Network Layer:** Responsible for packet routing
  * **Data Link Layer (CSMA):** Uses Carrier Sense Multiple Access for shared media like LAN

* **Connection:**
  * Connected to User Terminal (UT) via **CSMA channel**

### 2️⃣ **UT (User Terminal)**

* **Modules:**
  * `SatNetDevice`: A new satellite network device, simulating a satellite-specific NIC for communication
  * `GeoPosition`: Records the terminal's geographic coordinates (latitude, longitude, altitude using WGS-84/GRS-84)

* **Communication:**
  * Connected to Satellite via **SatChannel**

### 3️⃣ **Satellite**

* **Modules:**
  * `SatGeoNetDevice`: An advanced satellite network device, responsible for relaying data between ground terminals and gateways
  * `GeoPosition`: Records the satellite’s orbital position using WGS-84 or GRS-84

* **Communication:**
  * Uses **SatChannel** to establish uplink/downlink with both UT and GW

### 4️⃣ **GW (Gateway)**

* **Modules:**
  * `SatNetDevice`: Handles data transmission to and from the satellite
  * `CSMA`: Communicates with end users on the ground via terrestrial networks (e.g., Wi-Fi, wired LAN)
  * `GeoPosition`: Records the gateway’s geographic coordinates

* **Communication:**
  * Communicates with end users via **Ideal channel**
 
### 5️⃣ **Opposite End User**

* Same structure as the initial end user, including Application, Transport, Network, and CSMA layers

* Communicates through an **Ideal Channel**, representing a simplified model without interference or delay


### 🔄 Communication Channels

| Channel Name     | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| **CSMA channel** | Communication between ground user and UT, simulates LAN/shared media        |
| **SatChannel**   | Simulates satellite uplink/downlink (UT ↔ SAT ↔ GW)                         |
| **Ideal channel**| Idealized channel between GW and ground user (no interference or latency)   |

## User terminal
<img width="401" height="487" alt="image" src="https://github.com/user-attachments/assets/d5727091-f0f8-47e8-8620-cb60bb0a9fe5" />

### SatNetDevice
- The **main container module** that integrates submodules such as LLC, MAC, and PHY.
- Inherits from `NetDevice`, tailored for satellite use.


### SatPacketClassifier
- The **packet classifier** module.
- Classifies packets based on features (e.g., QoS, destination) to determine handling paths (e.g., queues or modules).


### SatUtLlc (Logical Link Control - LLC Layer)
- **SatReturnLinkEncapsulator**: Encapsulates uplink IP packets using Return Link Encapsulation (RLE) format for return channel transmission.
- **SatGenericStreamEncapsulator**: Handles downstream packet unpacking and re-encapsulation.
- **SatQueue**: Temporary buffers for uplink and downlink data.
- **SatRequestManager**: 
  - Monitors queue status and length.
  - Sends **Capacity Request (CR)** messages to the **Network Control Center (NCC)** when needed.
  - Supports two types of dynamic capacity requests:
    - **RBDC** (Rate-Based Dynamic Capacity)
    - **VBDC** (Volume-Based Dynamic Capacity)

### SatUtMac (MAC Layer)
- **SatUtScheduler**: Schedules time slots based on TBTP (Terminal Burst Time Plan) information.
- **SatRandomAccess**: Supports random access mechanisms.
- **SatTbtpContainer**: Stores TBTP content received from NCC.

### SatUtPhy (PHY Layer)
- **SatPhyTx**: Handles physical layer transmission logic.
- **SatPhyRx**: Handles physical layer reception logic.
- **SatPhyRxCarrier**: Manages multi-carrier reception and interference checks.
  - **SatLinkResults**: Stores results of received packets.
  - **SatInterference**: Evaluates signal interference during reception.


### Typical Usage Flow

1. Packets enter the UT and are classified by `SatPacketClassifier`.
2. LLC layer processes uplink/downlink encapsulation and manages queues.
3. `SatRequestManager` sends CR messages to the NCC based on queue status.
4. MAC layer uses TBTP to schedule transmissions.
5. PHY layer executes actual transmission and reception of packets, including interference assessment.

## Geostationary satellite
<img width="697" height="396" alt="image" src="https://github.com/user-attachments/assets/2eb57f00-5b6c-495f-a82e-32b7084817ec" />

### 1. SatGeoUserPhy
Handles the **User Link PHY layer** for communication with end-users.

- `Tx`: Transmitter module responsible for sending signals to the user terminal (UE).
- `Rx`: Receiver module to receive signals from the user terminal.
- `RxC`: Receiver Control module that monitors the quality of reception.
- `I`: Interference Estimator used to calculate SINR (Signal-to-Interference-plus-Noise Ratio).

### 2. SatGeoFeederPhy
Handles the **Feeder Link PHY layer**, typically used for communication with the ground gateway (GW).

- Structure is similar to `SatGeoUserPhy`, including `Tx`, `Rx`, `RxC`, and `I`.
- Responsible for receiving signals from the GW or sending signals back to it.

### Additional Notes

- **Transparent Forwarding (bent-pipe)**:  
  The satellite does not process any packets; it only amplifies and forwards the signal.

- **SINR Calculation**:  
  Although the satellite does not process packets, it still calculates SINR. This is done **separately** at both the user and feeder ends and then **combined** at the final receiver (e.g., the gateway) using a composite formula.

- **No Processing Logic**:  
  No packet segmentation or reassembly is performed; only analog signal-level operations are involved.

## Gateway
<img width="301" height="368" alt="image" src="https://github.com/user-attachments/assets/eb5df9ec-62b4-4d55-8eb9-f60da015149a" />

### 1. `SatNetDevice`
- The container representing the entire gateway network device.
- Encapsulates all layers and functional modules of the Gateway transmission stack.

### 2. `SatPacketClassifier`
- Responsible for classifying incoming packets.
- Determines the appropriate routing path and scheduling strategy.
- Acts as the entry point to the internal Gateway logic.

### 3. `SatGwLlc` (Logical Link Control Layer)
Handles encapsulation and queueing of data.

#### Tx (Transmit Path)
- `SatGenericStreamEncapsulator`: Encapsulates higher-layer data into the Generic Stream format (GSE).
- `SatQueue`: Queues the packets to be transmitted.

#### Rx (Receive Path)
- `SatReturnLinkEncapsulator`: De-encapsulates incoming data from the return link.
- `SatQueue`: Buffers the received packets for further processing.


### 4. `SatGwMac` (Media Access Control Layer)
- Controls the transmission schedule and BBFrame assembly.
- `SatFwdLinkScheduler`: Forward link scheduler that determines which data to send and when.
- `SatBbFrameContainer`: Holds fully assembled Baseband Frames (BBFrames).


### 5. `SatGwPhy` (Physical Layer)
Interfaces with the physical transmission channels.

- `SatPhyTx`: Manages signal transmission.
- `SatPhyRx`: Manages signal reception.
  - `SatPhyRxCarrier`: Handles multiple carrier receptions.
  - `SatLinkResults`: Stores evaluation results of link quality.
  - `SatInterference`: Detects and handles interference data.

### Architecture Highlights

- `SatNetDevice` encapsulates the complete stack from classification to physical transmission.
- The **LLC layer** supports bidirectional operations via separate **Tx** and **Rx** modules, ensuring full-duplex communication.
- The **MAC layer** is responsible for generating BBFrames based on a MODCOD scheme and scheduling them efficiently.
- The **PHY layer** provides both transmission and multi-carrier reception functionality and supports:
  - Link quality monitoring
  - Interference assessment
- Even when no user data is available, the Gateway can generate **dummy BBFrames** to maintain synchronization and continuous frame transmission.

## Network control center
### Responsibilities
The **Network Control Centre (NCC)** is responsible for:
- Return link resource allocations
- Admission control
- Packet scheduling
- Adaptive Coding and Modulation (ACM)

---

### Global NCC and Scheduler Design
- A **global NCC** is implemented within the satellite module.
- Each **spot-beam** has its **own scheduler**, called `SatBeamScheduler`.

---

### GW and NCC Communication
- To simplify implementation, **NCC is directly attached to each GW** and its `SatNetDevice` using **NS-3 callbacks**.
- This setup enables:
  - An ideal simulation communication channel between NCC and GW.
  - Easy migration to a real protocol implementation in the future.

## Channel
### Basic Concept

- `SatChannel` represents a **frequency color** (bandwidth).
- The main purpose of a channel is to **pass packets within a specific frequency band** to all receivers sharing the same bandwidth.
- In other words:
  - **Co-channel beams** share the same channel and may **interfere** with each other.
  - Beams using **different frequency bands** are fully **separated into distinct channel instances**.

### User Link

- A total of **4 `SatChannel` instances per direction**.
- Each instance represents **125 MHz bandwidth**.
- In the reference system:
  - There are 72 beams in total.
  - 72 / 4 = **18 spot-beams** share the same user link channel.
  - These 18 spot-beams may **interfere** with each other due to frequency reuse.

### Feeder Link

- There are **16 `SatChannel` instances per direction**:
  - Computed as 2 GHz / 0.125 GHz = 16.
  - Each instance also represents **125 MHz bandwidth**.
- All **Gateways (GWs)** use the **same frequency band**.
  - Thus, a maximum of **5 GWs** can share the same channel instance.

##  Random Access in DVB-RCS2

###  Three RA Modes (Focused on DVB-RCS2 Usage)

- **CRDSA (Contention Resolution Diversity Slotted ALOHA)**: Primary method for initial data access and emergency transmission.
- **Slotted ALOHA**: Limited to small control message transmission.
- **MARSALA**: Enhances CRDSA decoding performance when interference cancellation fails.

### CRDSA 

CRDSA is the core RA technique in DVB-RCS2, providing efficient access for user terminals (UTs).

#### Use Cases (Defined in DVB-RCS2):
1. **RA Cold Start**: Initial request when UT powers on.
2. **RA-DAMA Top-Up**: Additional capacity before quota runs out.
3. **RA-DAMA Back-Up**: Backup communication.
4. **RA IP Queue**: Random access for queued IP data.
5. **RA Capacity Request**: General bandwidth request.
6. **RA for SCADA**: Used in industrial remote monitoring systems.

#### Key Features:
- Transmits multiple packet replicas to increase success rate.
- Uses SIC (Successive Interference Cancellation) to resolve collisions.
- Optimized for **RA cold start** to reduce packet delay when no dedicated resources are available.

### Role of Slotted ALOHA

- Supported in DVB-RCS2 but limited to:
  - **Capacity Request (CR)**
  - **ARQ ACK control messages**
- Not suitable for large payloads due to limited capacity.

###  MARSALA Enhancement

- Based on **Correlation-Based Localisation** technique.
- Activated when **CRDSA's SIC decoding fails**.
- Improves decoding accuracy and reduces packet loss.

## Return link packet scheduling

In DVB-RCS2, the **Return Link Scheduler** is centrally managed by the **Network Control Center (NCC)**. Each **spot-beam** has an **independent scheduler**, and there is **no interaction** between them.

### Three Time Slot Configuration Modes

| Mode     | Description                                                      |
|----------|------------------------------------------------------------------|
| Conf-0   | Fixed time slots with a static waveform (MODCOD + burst length) |
| Conf-1   | Fixed time slots with varying MODCOD                            |
| Conf-2   | Time slots dynamically generated based on user demand and channel conditions |


### MODCOD and Waveform Support

- **Supported waveforms:** 3 to 22  
- **MODCOD range:** QPSK 1/3 to 16QAM 5/6  
- **Burst lengths:** 536 or 1616 symbols  

> ✅ Based on C/No (Carrier-to-Noise ratio) measurements reported by the Gateway, the NCC selects the optimal MODCOD that balances **spectral efficiency** and **target error rate**.


### Six-Step Scheduling Procedure

1. **SatDamaEntry/CR Update**  
   → Process Capacity Requests (CR) received during the previous superframe.

2. **Preliminary Resource Allocation**  
   → Pre-allocate soft-symbols for each User Terminal (UT) based on CRA configuration, request type (RBDC, VBDC), channel C/No, load, and frame configuration.

3. **Time Slot Generation**  
   → Generate time slots for each UT and RC index according to allocated soft-symbols, and dynamically fill in the TBTP (Terminal Burst Time Plan).

4. **SatDamaEntry Update**  
   → Update VBDC bytes allocated per UT to reflect resource usage status.

5. **TBTP Signaling**  
   → Send TBTP messages to the respective Gateway protocol stack that manages the resources for the specific spot-beam.

6. **Schedule Next Frame**  
   → Determine the scheduling time for the next superframe.

##  Demand Assignment Multiple Access (DAMA)

**DAMA** is implemented in the **Request Manager (RM)** module to dynamically evaluate and allocate bandwidth based on demand. It operates per **RC index** (resource configuration index), with individual settings for each one.

###  Supported Capacity Allocation Types

The satellite module supports the following DAMA capacity types:

- **CRA**: Continuous Rate Assignment  
- **RBDC**: Rate-Based Dynamic Capacity  
- **VBDC**: Volume-Based Dynamic Capacity

###  Configuration and Operation

- **Request Manager (RM)** is configured through lower-layer service parameters.
- Each **RC index** may be configured independently for DAMA operation.

###  Evaluation and Capacity Request Process

- **RM periodically or on-demand evaluates** the need to send a **Capacity Request (CR)** for a given RC index.
- It monitors:
  - **UT packet queues** (user terminal)
  - **Incoming traffic rates**
  - **Previously received DA resources** from **TBTPs (Terminal Burst Time Plans)**
- **CRs are modeled as real signaling messages**, with a probability of transmission error considered.

## UT Scheduler

The **User Terminal (UT) Scheduler** is responsible for managing transmission opportunities (time slots) to the upper layers based on the **TBTP (Terminal Burst Time Plan)** messages received from the **Network Control Center (NCC)**.

###  Scheduling Behavior

- **Primary Operation**:  
  The UT scheduler primarily follows the **RC (Return Channel) indices** defined in the received TBTP messages.

- **Exception Handling**:  
  If no packets are available in the **RLE (Return Link Encapsulation)** queue for a given RC index, the UT scheduler is allowed to **freely choose another RC index** to serve, ensuring resource efficiency and avoiding idle time slots.

###  Summary

The UT scheduler combines centralized guidance from NCC with local flexibility. It follows the NCC instructions under normal conditions but maintains adaptability to maximize link utilization when certain queues are empty.

##  FWD Link Scheduler

- Periodically constructs multiple Baseband (BB) frames and fills them with GSE packets from the LLC in priority order.
- Assigns optimal MODCOD (Modulation and Coding) for each BB frame based on the UT-reported C/No (Carrier-to-Noise ratio).
- After each scheduling round, may downgrade the MODCOD to reduce the number of required BB frames and enhance spectral efficiency.

##  ARQ (Automatic Repeat reQuest)

- ARQ is not included in the official DVB-RCS2 specification.
- For research purposes, a **Selective Repeat ARQ** mechanism is implemented in the satellite module.
- ARQ operates at the **LLC layer**, and supports retransmissions for:
  - FWD link: using GSE packets
  - RTN link: using RLE packets

##  Mobility and Handover

- Two mobility models are implemented for UTs (User Terminals):
  - **Static Mode**: UT remains fixed.
  - **Trace-Based Mode**: UT moves according to trace files, with intermediate positions calculated using linear interpolation between points.
- Once the UT reaches the final position, it stays static until the simulation ends.

- A **handover module** can be attached to any UT:
  - It monitors the carrier's power level.
  - If the signal drops below a predefined threshold, it sends a request to the NCC (Network Control Center) to switch the UT to a new carrier or beam.
  - The NCC authorizes the handover to ensure continuous communication.


