<img width="805" height="732" alt="image" src="https://github.com/user-attachments/assets/ccb63340-4385-487f-bccd-a84331029b26" />

## Running `sat-constellation-example`

**Command :**
```
cd ~/workspace/bake/source/ns-3.43
./ns3 run scratch/sat-constellation-example
```
**Output :**
### 1. Satellites
```
Satellites
  SAT: ID = 0, at 0.013043,7.20835,3.57864e+07
    Devices to ground stations 
      02-06-00:00:00:00:00:01
        Feeder at 02-06-00:00:00:00:00:03, beam 30
        Feeder at 02-06-00:00:00:00:00:09, beam 43
      Feeder connected to
        00:00:00:00:00:05
        00:00:00:00:00:15
        User at 02-06-00:00:00:00:00:04, beam 30
        User at 02-06-00:00:00:00:00:0a, beam 43
      User connected to
        00:00:00:00:00:0c
    ISLs 
      02-06-00:00:00:00:00:23 to SAT 1
```
- ID = 0 → Satellite #0
- at (0.013043, 7.20835, 3.57864e+07) → longitude, latitude, altitude (≈35,786 km, GEO orbit).
- Devices to ground stations → list of radio interfaces that connect SAT 0 to GWs/UTs.
  - Feeder = satellite-to-Gateway interface (beam 30 & 43).
  - Feeder connected to ... = MAC addresses of ground GWs linked with this satellite.
  - User at ... beam 30/43 = satellite-to-User Terminal interfaces.
  - User connected to ...:0c = specific UT device currently attached to SAT 0.
- ISLs → Inter-Satellite Link. SAT 0 has an ISL (...:23) connected to SAT 1.

```
  SAT: ID = 1, at -0.0219759,47.7047,3.5779e+07
    Devices to ground stations 
      02-06-00:00:00:00:00:02
        Feeder at 02-06-00:00:00:00:00:13, beam 30
        Feeder at 02-06-00:00:00:00:00:18, beam 43
      Feeder connected to
        00:00:00:00:00:0b
        00:00:00:00:00:1a
        User at 02-06-00:00:00:00:00:14, beam 30
        User at 02-06-00:00:00:00:00:19, beam 43
      User connected to
        00:00:00:00:00:16
        00:00:00:00:00:17
    ISLs 
      02-06-00:00:00:00:00:24 to SAT 0
```
- ID = 1 → Satellite #1
- at (-0.0219759, 47.7047, 3.5779e+07) → longitude, latitude, altitude.
- Feeder interfaces → connected to beams 30 & 43.
- Feeder connected to ... → MAC addresses of GWs for this satellite.
- User connected to ...:16 and ...:17 → two UTs currently linked with SAT 1.
- ISLs → SAT 1 is connected back to SAT 0 via ISL (...:24).

### 2. Gateways (GWs)
```
GWs
  GW: ID = 2, at 55.75,37.62,0
  Devices 
    02-06-00:00:00:00:00:0b, sat: 0, beam: 43
    02-06-00:00:00:00:00:1a, sat: 1, beam: 43
  GW: ID = 3, at 48.85,2.34,0
  Devices 
    02-06-00:00:00:00:00:05, sat: 0, beam: 30
    02-06-00:00:00:00:00:15, sat: 1, beam: 30
```
- GW ID = 2 → Located at (55.75, 37.62).
- Connected to SAT 0 beam 43 and SAT 1 beam 43.
- GW ID = 3 → Located at (48.85, 2.34).
- Connected to SAT 0 beam 30 and SAT 1 beam 30.

### 3. User Terminals (UTs)
```
UTs
  UT: ID = 4, at 48.8534,2.3488,0
  Devices 
    02-06-00:00:00:00:00:0c, sat: 0, beam: 43. Linked to GW 02-06-00:00:00:00:00:1a
  UT: ID = 7, at 55.755,37.6218,0
  Devices 
    02-06-00:00:00:00:00:16, sat: 1, beam: 30. Linked to GW 02-06-00:00:00:00:00:05
  UT: ID = 8, at 55.754,37.621,0
  Devices 
    02-06-00:00:00:00:00:17, sat: 1, beam: 30. Linked to GW 02-06-00:00:00:00:00:05
```
- UT 4 → Connected via SAT 0, beam 43. Traffic is routed through GW (...:1a, belongs to SAT 1).
- UT 7 → Connected via SAT 1, beam 30. Linked to GW (...:05, belongs to SAT 0).
- UT 8 → Similar to UT 7, connected via SAT 1, beam 30 → GW (...:05).
- 👉 Notice cross-links: UTs can connect to one satellite, but the serving GW may belong to another satellite (via ISL).

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
**Output :** (Low-level internal trace, showing that objects are created, used, and released in memory.)
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

**Output** (High-level trace: shows when a complete BBFrame is actually packed by the scheduler and transmitted.)

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
