<img width="299" height="680" alt="image" src="https://github.com/user-attachments/assets/ee49d20e-fc28-4fda-a1ed-11287e1728b6" />

1. Initialization
```
SatBeamScheduler::SatBeamScheduler()   // Constructor (282-296)
SatBeamScheduler::~SatBeamScheduler()  // Destructor  (298)
SatBeamScheduler::GetTypeId()          // Register TypeId and attributes  (192-279)
SatBeamScheduler::Initialize(...)      // Initialize satId, beamId, GW, Orbiter, superframe, etc.  (352-444)
```
This part sets up the basic environment for the scheduler. It defines the satellite ID, beam ID, gateway device, orbital device, and the superframe sequence. It also registers configurable attributes (like C/N0 estimation mode, control slot interval) and ensures the scheduler starts in sync with the simulation timeline.

2. UT Management
```
SatBeamScheduler::AddUt(...)             // Add a new UT (447-481)
SatBeamScheduler::AddUtInfo(...)         // Store UT information  (484-512)
SatBeamScheduler::RemoveUt(...)          // Remove a UT (1053-1066)
SatBeamScheduler::RemoveUtInfo(...)      // Remove UT from internal map  (515-535)
SatBeamScheduler::HasUt(...)             // Check if UT exists  (538-544)
SatBeamScheduler::TransferUtToBeam(...)  // Perform UT handover to another beam  (980-1016)
```
This section manages User Terminals (UTs) in the beam.
It allows adding new UTs, removing existing ones, and transferring them across beams (handover). It ensures that UTs are properly tracked and that their minimum required resources are reserved.

3. Control & Measurement
```
SatBeamScheduler::UpdateUtCno(...)         // Update UT C/N₀ estimation  (555-563)
SatBeamScheduler::UpdateSatelliteCno(...)  // Update satellite C/N₀ estimation (566-574)
SatBeamScheduler::UtCrReceived(...)        // Receive Capacity Request (CR) message (577-585)
SatBeamScheduler::CreateCnoEstimator()     // Create C/N₀ estimator (588-608)
SatBeamScheduler::SendCnoToSatellite()     // Send C/N₀ report to satellite  (611-624)
```
This part deals with signal quality estimation (C/N₀) and capacity requests (CR) from UTs.
The scheduler updates the C/N₀ samples, processes CR messages, and generates reports. These measurements directly influence resource allocation decisions in scheduling.
4. Scheduling Core
```
SatBeamScheduler::Schedule()                      // Main scheduling loop  (627-712)
SatBeamScheduler::AddRaChannels(...)              // Add Random Access (RA) time slots (715-744)
SatBeamScheduler::UpdateDamaEntriesWithReqs()     // Update UT demands (CRA, RBDC, VBDC) (777-863)
SatBeamScheduler::DoPreResourceAllocation()       // Pre-allocation before final scheduling (866-887)
SatBeamScheduler::UpdateDamaEntriesWithAllocs(...)// Update results with actual allocations (890-977)
```
This is the heart of the beam scheduler.
It collects all UT requests (CRA = Constant Rate Assignment, RBDC = Rate-Based Dynamic Capacity, VBDC = Volume-Based Dynamic Capacity), performs pre-allocation, and generates Terminal Burst Time Plans (TBTPs). These plans specify which UT can transmit in which time slot. It also ensures RA (Random Access) channels are properly included.

5. Messaging
```
SatBeamScheduler::Send(...)              // Broadcast control message (312-318)
SatBeamScheduler::SendTo(...)            // Send message to a specific UT (321-332)
SatBeamScheduler::SendToSatellite(...)   // Send message to satellite (335-341)
SatBeamScheduler::SetSendTbtpCallback()  // Set callback for TBTP delivery (344-349)
SatBeamScheduler::CreateTimu()           // Create TIMU (handover info) message (1069-1079)
```
This part handles control plane messaging.
It sends scheduling results (like TBTPs) and other control messages to UTs, gateways, or satellites. It ensures proper communication so that UTs can follow the schedule.

6. Connectivity
```
SatBeamScheduler::ConnectUt(...)     // Connect a UT (1019-1025)
SatBeamScheduler::DisconnectUt(...)  // Disconnect a UT (1028-1034)
SatBeamScheduler::ConnectGw(...)     // Connect a Gateway (1037-1042)
SatBeamScheduler::DisconnectGw(...)  // Disconnect a Gateway (1045-1050)
```
This section manages the connectivity state of UTs and gateways.
It ensures UTs are registered to the beam, that gateways are connected properly, and cleans up connections when UTs/GWs leave.

```
+--------------------------------------------------+
| 192-279  GetTypeId()                             |
|   Register TypeId and attributes                 |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 282-296  Constructor                             |
| 298      Destructor                              |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 312-318  Send()                                  |
| 321-332  SendTo()                                |
| 335-341  SendToSatellite()                       |
| 344-349  SetSendTbtpCallback()                   |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 352-444  Initialize(...)                         |
|   Setup satId, beamId, GW, Orbiter, superframe   |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 447-481  AddUt()                                 |
| 484-512  AddUtInfo()                             |
| 515-535  RemoveUtInfo()                          |
| 538-544  HasUt()                                 |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 555-563  UpdateUtCno()                           |
| 566-574  UpdateSatelliteCno()                    |
| 577-585  UtCrReceived()                          |
| 588-608  CreateCnoEstimator()                    |
| 611-624  SendCnoToSatellite()                    |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 627-712  Schedule()                              |
| 715-744  AddRaChannels()                         |
| 777-863  UpdateDamaEntriesWithReqs()             |
| 866-887  DoPreResourceAllocation()               |
| 890-977  UpdateDamaEntriesWithAllocs()           |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 980-1016 TransferUtToBeam()                      |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 1019-1025 ConnectUt()                            |
| 1028-1034 DisconnectUt()                         |
| 1037-1042 ConnectGw()                            |
| 1045-1050 DisconnectGw()                         |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 1053-1066 RemoveUt()                             |
+--------------------------------------------------+
                |
                v
+--------------------------------------------------+
| 1069-1079 CreateTimu()                           |
+--------------------------------------------------+
```
