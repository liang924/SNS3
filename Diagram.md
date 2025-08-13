# SNS3 Simulation Flow for DVB-S2X/RCS2

<img width="721" height="991" alt="sns3 drawio" src="https://github.com/user-attachments/assets/c181e6b5-2f69-4566-891e-15f1838367c0" />

##  General Simulation Initialization

1. **Start Simulation (SNS3)**  
   Initialize the simulation environment.

2. **Configure Scenario Parameters**  
   Define key parameters such as bandwidth, number of nodes, and simulation time.

3. **Enable Traffic Generation at UE**  
   Activate traffic generation from the User Equipment (UE).

4. **Determine Traffic Direction**  
   Check whether the packet is for **Forward Link (GW → UE)** or **Return Link (UE → GW)**.

##  Forward Link (Gateway to UE)

1. **Apply GSE Encapsulation**  
   Use `SatGenericStreamEncapsulator` to encapsulate packets with the GSE format.

2. **Forward to SatQueue (Tx)**  
   Push encapsulated data into the satellite transmission queue.

3. **Schedule BBFrame**  
   `SatFwdLinkScheduler` schedules the Base Band Frame (BBFrame).

4. **Transmit via Physical Layer**  
   Transmit from `SatPhyTx` through the channel to `SatPhyRx`.

5. **Receive and Extract BBFrame**  
   `SatPhyRx` extracts BBFrame and delivers it to the receiving queue.

6. **Decapsulate**  
   `SatReturnLinkDecapsulation` decapsulates the BBFrame to retrieve the original data.

##  Return Link (UE to Gateway)

1. **Apply RLE Encapsulation**  
   Use `SatReturnLinkEncapsulator` to apply Return Link encapsulation.

2. **Forward to SatQueue (Rx)**  
   Send the encapsulated packet to the return path queue.

3. **Check Resource Allocation (TBTP)**  
   Verify if transmission time is allocated via TBTP (Terminal Burst Time Plan).

4. **Was a Resource Allocated?**
   - **YES** → Transmit in the assigned timeslot.
   - **NO** → Send a **Capacity Request (CR)** to the **Network Control Center (NCC)**.

5. **NCC Computes Resources and Sends TBTP**  
   NCC performs CRA (Continuous Rate Assignment) or VBDC (Volume-Based Dynamic Capacity) and responds with an updated TBTP.

6. **Transmit in Allocated Timeslot**  
   Transmit packet based on the assigned time window.

7. **Gateway Receives and Processes Packet**  
   Gateway receives the return data and processes it.

## Simulation Output

- **Simulation Statistics / Output**  
  Final step where SNS3 provides performance metrics and statistics of the simulation (e.g., throughput, delay, efficiency).
  
