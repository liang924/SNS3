<img width="844" height="758" alt="image" src="https://github.com/user-attachments/assets/fc8e8780-f2e2-47c2-94e6-492b47bb5acb" />


## Running `sat-constellation-example`

**Command :**
```
cd ~/workspace/bake/source/ns-3.43
./ns3 run scratch/sat-constellation-example
```
**Output :**
### 1. Satellites
```
Satellites                               # Satellite system information
  SAT: ID = 0, at 0.013043,7.20835,3.57864e+07   # Satellite with ID=0, located at given coordinates (lat ,lon , alt)
    Devices to ground stations           # Devices linked to ground stations
      02-06-00:00:00:00:00:01            # Device identifier connected to satellite
        Feeder at 02-06-00:00:00:00:00:03, beam 30   # Feeder terminal at address ending :03, using beam 30
        Feeder at 02-06-00:00:00:00:00:09, beam 43   # Feeder terminal at address ending :09, using beam 43
      Feeder connected to                # Feeders connected to specific ground nodes
        00:00:00:00:00:05                # Ground node/device with ID ending :05
        00:00:00:00:00:15                # Ground node/device with ID ending :15
        User at 02-06-00:00:00:00:00:04, beam 30   # User terminal at :04, connected via beam 30
        User at 02-06-00:00:00:00:00:0a, beam 43   # User terminal at :0a, connected via beam 43
      User connected to                  # Users linked to downstream devices
        00:00:00:00:00:0c                # Device or node with ID ending :0c
    ISLs                                 # Inter-Satellite Links
      02-06-00:00:00:00:00:23 to SAT 1   # ISL from device :23 on this satellite to satellite ID=1
```

```
SAT: ID = 1, at -0.0219759,47.7047,3.5779e+07   # Satellite with ID=1, located at given coordinates (lat ,lon , alt)
    Devices to ground stations                  # Devices connected to ground stations
      02-06-00:00:00:00:00:02                   # Device identifier assigned to this satellite
        Feeder at 02-06-00:00:00:00:00:13, beam 30   # Feeder terminal at address ending :13, using beam 30
        Feeder at 02-06-00:00:00:00:00:18, beam 43   # Feeder terminal at address ending :18, using beam 43
      Feeder connected to                       # Feeders linked to ground nodes
        00:00:00:00:00:0b                       # Ground node/device with ID ending :0b
        00:00:00:00:00:1a                       # Ground node/device with ID ending :1a
        User at 02-06-00:00:00:00:00:14, beam 30   # User terminal at :14, connected via beam 30
        User at 02-06-00:00:00:00:00:19, beam 43   # User terminal at :19, connected via beam 43
      User connected to                         # User terminals further connected to other devices
        00:00:00:00:00:16                       # Device or node with ID ending :16
        00:00:00:00:00:17                       # Device or node with ID ending :17
    ISLs                                        # Inter-Satellite Links
      02-06-00:00:00:00:00:24 to SAT 0          # ISL from device :24 on this satellite to satellite ID=0

```

### 2. Gateways (GWs)
```
GWs                                         # Ground stations (Gateways) information
  GW: ID = 2, at 55.75,37.62,0              # Gateway with ID=2, located at latitude 55.75, longitude 37.62, altitude 0
  Devices                                   # Devices linked to this gateway
    02-06-00:00:00:00:00:0b, sat: 0, beam: 43   # Device ID :0b, connected to Satellite 0 via beam 43
    02-06-00:00:00:00:00:1a, sat: 1, beam: 43   # Device ID :1a, connected to Satellite 1 via beam 43
  GW: ID = 3, at 48.85,2.34,0               # Gateway with ID=3, located at latitude 48.85, longitude 2.34, altitude 0 
  Devices                                   # Devices linked to this gateway
    02-06-00:00:00:00:00:05, sat: 0, beam: 30   # Device ID :05, connected to Satellite 0 via beam 30
    02-06-00:00:00:00:00:15, sat: 1, beam: 30   # Device ID :15, connected to Satellite 1 via beam 30

```

### 3. User Terminals (UTs)
```
UTs                                              # User Terminals (UT) information
  UT: ID = 4, at 48.8534,2.3488,0                # User Terminal with ID=4, located at latitude 48.8534, longitude 2.3488, altitude 0 
  Devices                                        # Devices connected to this UT
    02-06-00:00:00:00:00:0c, sat: 0, beam: 43. Linked to GW 02-06-00:00:00:00:00:1a   # Device :0c, connected to Satellite 0 via beam 43, linked to Gateway device :1a
  UT: ID = 7, at 55.755,37.6218,0                # User Terminal with ID=7, located at latitude 55.755, longitude 37.6218, altitude 0 
  Devices
    02-06-00:00:00:00:00:16, sat: 1, beam: 30. Linked to GW 02-06-00:00:00:00:00:05   # Device :16, connected to Satellite 1 via beam 30, linked to Gateway device :05
  UT: ID = 8, at 55.754,37.621,0                 # User Terminal with ID=8, located at latitude 55.754, longitude 37.621, altitude 0 
  Devices
    02-06-00:00:00:00:00:17, sat: 1, beam: 30. Linked to GW 02-06-00:00:00:00:00:05   # Device :17, connected to Satellite 1 via beam 30, linked to Gateway device :05

```

### 4. GW Users & UT Users
```
GW users
  GW user: ID = 14
  GW user: ID = 15
  GW user: ID = 16
UT users
  GW user: ID = 5
  GW user: ID = 6
  GW user: ID = 9
  GW user: ID = 10
  GW user: ID = 11
  GW user: ID = 12
```
- GW users (IDs 14–16) → represent end-host applications behind the GWs.
- UT users (IDs 5–12) → represent end-host applications behind the UTs.
- 👉 These IDs are not physical devices but logical application nodes.

### 5. SatIdMapper (MAC → Satellite ID)
```
SatIdMapper GW/UT MAC to satellite ID map
00:00:00:00:00:05 0
00:00:00:00:00:0b 0
00:00:00:00:00:0c 0
00:00:00:00:00:15 0
00:00:00:00:00:16 1
00:00:00:00:00:17 1
00:00:00:00:00:1a 1
```
- This table maps ground MAC addresses to which satellite (ID) they are associated with.
- Example:
  - `...:05` → belongs to SAT 0.
  - `...:1a` → belongs to SAT 1.
  - `...:0c` (UT 4) → belongs to SAT 0.

<img width="1118" height="746" alt="image" src="https://github.com/user-attachments/assets/38b5b9d4-55ae-444c-bf2d-f9659e5ec202" />

## Running `sat-fwd-system-test-example`

**Command :**
```
cd ~/workspace/bake/source/ns-3.43

# 打開 Forward Link Scheduler 的所有訊息
export NS_LOG="SatFwdLinkScheduler=level_all|prefix_time"

# 如果要同時觀察 BBFrame
export NS_LOG+=":SatBbFrame=level_all|prefix_time"

# 如果要觀察 Beam Scheduler
export NS_LOG+=":SatBeamScheduler=level_all|prefix_time"

./ns3 run scratch/sat-fwd-system-test-example
```
<img width="368" height="711" alt="image" src="https://github.com/user-attachments/assets/3552ef42-1ecc-40fa-bde5-ed914c6560bb" />

**Output :** Internal Trace
### 1. Old Frame Handling and Release
```
+5.960551104s SatBbFrame:GetPayload(0x5e8f6e0527f0)
+5.960551104s SatBbFrame:GetPayload(0x5e8f6e0527f0)
+5.960551104s SatBbFrame:~SatBbFrame(0x5e8f6e0527f0)
```

- Action: The scheduler retrieves the payload (`GetPayload`) to pass it to the PHY/MAC layer for transmission.
- Destructor (`~SatBbFrame`): Indicates the frame is no longer needed, and memory is released.
- Meaning: The forward link transmission cycle finishes for this frame, and the system prepares for the next scheduling round.

### 2. New Scheduling Round Begins
```
+5.973331392s SatFwdLinkScheduler:SortSchedulingObjects(0x5e8f634c5f00)
+5.973331392s SatFwdLinkScheduler:GetSchedulingObjectCno(0x5e8f634c5f00, 0x5e8f6cdede80)
```

- SortSchedulingObjects: Orders all candidate scheduling objects (users/flows).
- GetSchedulingObjectCno: Fetches C/N₀ (Carrier-to-Noise density ratio), representing channel quality for each object.
- Meaning: This step decides who gets served first, usually based on channel quality (CQI) or fairness criteria.

### 3. Frame Space Check and Payload Filling
```
+5.973331392s SatBbFrame:GetSpaceUsedInBytes(0x5e8f6de91b50)
+5.973331392s SatBbFrame:AddPayload(0x5e8f6de91b50)
+5.973331392s SatBbFrame:GetSpaceLeftInBytes(0x5e8f6de91b50)
```

- GetSpaceUsedInBytes: Checks how much of the BBFrame is already occupied.
- AddPayload: Places a user’s packet into the frame.
- GetSpaceLeftInBytes: Updates the remaining capacity.
- Meaning: The scheduler translates the scheduling decision into an actual physical transmission unit (BBFrame).

### 4. Full Scheduling Cycle

A single Forward Link Scheduling cycle follows these steps:

1. Frame cleanup → old BBFrame payload is transmitted and freed.
2. Scheduling order → sort users/flows based on C/N₀ or policy.
3. Payload filling → selected users’ packets are added to the new BBFrame.
4. Capacity check → repeat until the frame is full or time expires.
5. Frame dispatch → completed BBFrame is sent, then cycle restarts.

### 5.Repeated Calls (Simplification)

- Multiple calls to GetPayload() → due to upper/lower layers accessing payload.
- Multiple calls to GetSpaceUsedInBytes() → triggered each time a packet is inserted.
- These can be summarized as “Frame check → payload insertion → update status.”

**Output** (bbframe summary)

> command:
> 1. Create a dedicated folder for logs
> ```
> mkdir -p ~/sns3_logs/fwd_test
> ```
> 2. Run the forward link test program
> ```
> ./ns3 run "sat-fwd-system-test-example --simLength=10 --traceFrameInfo=true"
> ```
> --simLength=10 → simulate for 10 seconds
> 
> --traceFrameInfo=true → enable BBFrame trace output (this is the key)
>
> 3. Save the output into a file
> ```
> ./ns3 run "sat-fwd-system-test-example --simLength=10 --traceFrameInfo=true" > ~/sns3_logs/fwd_test/run1.log
> ```
> 4. Check the log file
> ```
> cat ~/sns3_logs/fwd_test/run1.log
> ```

```
[BBFrameTx] Time: 9.99912, Frame Type: NORMAL_FRAME, ModCod: QPSK_1_TO_2, Occupancy: 1, Duration: +3.19507e+06ns, Space used: 4050, Space Left: 0 [Receivers: 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17, 00:00:00:00:00:17]
```
- Time: 9.99912
  Simulation time in seconds. This BB frame was transmitted at approximately t ≈ 9.99912 s.

- Frame Type: NORMAL_FRAME
  The BB frame is a Normal frame under DVB-S2/S2X (not Short or Dummy).

- ModCod: QPSK_1_TO_2
  Modulation and coding scheme: QPSK with code rate 1/2.
  This determines the frame’s payload capacity and duration (looked up by SatBbFrameConf based on ModCod + FrameType).

- Occupancy: 1
  Occupancy = 1.0, meaning the frame is completely filled (no remaining space).

- Duration: +3.19507e+06 ns
  Frame duration ≈ 3.19507 ms.
  This is derived from `GetBbFrameDuration`(modcod, type).

- Space used: 4050 B, Space Left: 0 B
  The frame payload carried 4050 bytes in total, with no remaining capacity (consistent with Occupancy = 1).

- Receivers: 00:00:00:00:00:17 (repeated multiple times)
  The frame contained multiple PDUs/packets, all directed to the same receiver MAC.
  In your topology dump earlier, 00:00:00:00:00:17 corresponds to UT ID = 8 (satellite 1, beam 30, linked to GW 00:...:05).
  So this frame is unicast to one UT, just carrying multiple packets for that single user.

## Running `sat-rtn-system-test-example`
```
cd ~/workspace/bake/source/ns-3.43
./ns3 run sat-rtn-system-test-example

```

**Output**
```
115 Bytes allocated within TBTP
130 Bytes allocated within TBTP
115 Bytes allocated within TBTP
115 Bytes allocated within TBTP
115 Bytes allocated within TBTP
130 Bytes allocated within TBTP 
```

### UT Scheduler (使用者端的排程)

```
NS_LOG="SatUtScheduler=level_all" ./ns3 run "sat-rtn-system-test-example --simLength=5"
```

- Call scheduling process
```
SatUtScheduler:GetPrioritizedRcIndexOrder(0x58ab35b0d1e0) UT scheduling RC: 0 with 130 bytes
SatUtScheduler:DoSchedulingForRcIndex(0x58ab35b0d1e0, 130, 1) UT scheduling RC: 0 with 130 bytes
SatUtScheduler:DoSchedulingForRcIndex(0x58ab35b0d1e0, 130, 2) UT scheduling RC: 0 with 130 bytes
SatUtScheduler:DoSchedulingForRcIndex(0x58ab35b0d1e0, 130, 3) Created a packet from RC: 3 size: 10 Created a packet from RC: 3 size: 119
SatUtScheduler:DoScheduling(0x58ab35b1e030, 130, 0, 1) UT scheduling RC: 0 with 130 bytes
SatUtScheduler:DoSchedulingForRcIndex(0x58ab35b1e030, 130, 0)
```
- 
