## Main Execution Flow of sat-onoff-example.cc

<img width="480" height="678" alt="image" src="https://github.com/user-attachments/assets/0a1bc924-c6cf-4b47-80da-c2fbeae2905a" />

### 1.Program starts

```
int main(int argc, char* argv[])
```

### 2.Create SimulationHelper

```
auto simulationHelper = CreateObject<SimulationHelper>("example-onoff");
```

### 3.Set simulation parameters

```
uint32_t packetSize = 512;
std::string dataRate = "500kb/s";
std::string onTime = "1.0";
std::string offTime = "0.5";
std::string scenario = "simple";
std::string sender = "both";
std::string simDuration = "11s";

// Default configuration settings
Config::SetDefault("ns3::SatEnvVariables::EnableSimulationOutputOverwrite", BooleanValue(true));
Config::SetDefault("ns3::SatHelper::ScenarioCreationTraceEnabled", BooleanValue(true));
Config::SetDefault("ns3::SatHelper::PacketTraceEnabled", BooleanValue(true));

// read command line parameters given by user
CommandLine cmd;
cmd.AddValue("packetSize", "Size of constant packet (bytes e.g 512)", packetSize);
cmd.AddValue("dataRate", "Data rate (e.g. 500kb/s)", dataRate);
cmd.AddValue("onTime", "Time for packet sending is on in seconds, (e.g. (1.0)", onTime);
cmd.AddValue("offTime", "Time for packet sending is off in seconds, (e.g. (0.5)", offTime);
cmd.AddValue("sender", "Packet sender (ut, gw, or both).", sender);
cmd.AddValue("scenario", "Test scenario to use. (simple, larger or full", scenario);
cmd.AddValue("simDuration", "Duration of the simulation (Time)", simDuration);
simulationHelper->AddDefaultUiArguments(cmd);
cmd.Parse(argc, argv);
```

### 4.Load satellite scenario

```
simulationHelper->LoadScenario("geo-33E");
```

### 5.Create satellite scenario

```
SatHelper::PreDefinedScenario_t satScenario = SatHelper::SIMPLE;  --66

if (scenario == "larger") {
    satScenario = SatHelper::LARGER;
} else if (scenario == "full") {
    satScenario = SatHelper::FULL;
}   --84~91

simulationHelper->CreateSatScenario(satScenario);  --105
```

### 6.Install network devices and connect channels

> These steps are called inside CreateSatScenario():

```
// Inside CreateSatScenario()  --105
SatHelper::InstallNetDevices();
SatHelper::InstallChannels();
```

### 7.Install application layer: OnOffApplication

```
if ((sender == "gw") || (sender == "both")) {
    simulationHelper->GetTrafficHelper()->AddOnOffTraffic(
        SatTrafficHelper::FWD_LINK,
        SatTrafficHelper::UDP,
        DataRate(dataRate),
        packetSize,
        NodeContainer(Singleton<SatTopology>::Get()->GetGwUserNode(0)),
        Singleton<SatTopology>::Get()->GetUtUserNodes(),
        "ns3::ConstantRandomVariable[Constant=" + onTime + "]",
        "ns3::ConstantRandomVariable[Constant=" + offTime + "]",
        Seconds(1.0),
        Time(simDuration),
        Seconds(0));
}

if (sender == "ut" || sender == "both") {
    simulationHelper->GetTrafficHelper()->AddOnOffTraffic(
        SatTrafficHelper::RTN_LINK,
        SatTrafficHelper::UDP,
        DataRate(dataRate),
        packetSize,
        NodeContainer(Singleton<SatTopology>::Get()->GetGwUserNode(0)),
        Singleton<SatTopology>::Get()->GetUtUserNodes(),
        "ns3::ConstantRandomVariable[Constant=" + onTime + "]",
        "ns3::ConstantRandomVariable[Constant=" + offTime + "]",
        Seconds(2.0),
        Time(simDuration),
        Seconds(0));
}
```

### 8.Create statistics module

> This is implicitly done inside SetOutputTag() or by AddOnOffTraffic():

```
simulationHelper->SetOutputTag(scenario);   --81
// internally triggers SatStatsHelperContainer creation
```

### 9.Run simulation

```
simulationHelper->RunSimulation();
```

## System architecture
<img width="869" height="656" alt="image" src="https://github.com/user-attachments/assets/ce271a73-76fc-4124-8cf3-8933b36f8686" />

1. SimulationHelper

```
auto simulationHelper = CreateObject<SimulationHelper>("example-onoff");
```
This class serves as the main controller of the entire simulation.  
All scenario construction, application installation, and simulation execution are performed through this component.

2. SatHelper

```
simulationHelper->CreateSatScenario(satScenario);
```
Called by `SimulationHelper`, this class is responsible for:

- Creating nodes (Gateway, User Terminal, Satellite)
- Installing devices (`NetDevice`)
- Connecting devices via channels (`SatChannel`)

3. SatTrafficHelper

```
simulationHelper->GetTrafficHelper()->AddOnOffTraffic(...);
```

>　This method returns a `SatTrafficHelper` object, which is responsible for:
>　- Installing `OnOffApplication`
>　- Setting up `PacketSink`
>　- Configuring application parameters (e.g., data rate, on/off time, etc.)

4. SatStatsHelperContainer

```
simulationHelper->SetOutputTag(scenario);
```
> It is responsible for recording:
> - Throughput per second  
> - Number of packets  
> - Latency and related information

5. NodeContainer & Node 
📍 You can find the following in `SatHelper`:

```
Ptr<Node> gateway = CreateObject<Node>();
NodeContainer utUsers = CreateObject<NodeContainer>();
Ptr<Node> satellite = CreateObject<Node>();
```

6. SatNetDevice / SatGeoNetDevice

📍 This section is in `SatHelper::InstallNetDevices()`:
```
Ptr<NetDevice> dev = CreateObject<SatNetDevice>();
node->AddDevice(dev);
```
The satellite installs `SatGeoNetDevice`, while ground nodes (UT/GW) install `SatNetDevice`.

7. SatChannel

📍 在 SatHelper::InstallChannels() 中：
```
Ptr<SatChannel> channel = CreateObject<SatChannel>();
satNetDevice->SetChannel(channel);
```
All node devices are connected through this `Channel`, which is used to simulate transmission conditions such as delay, bandwidth, and interference.

## Message Sequence Chart (MSC)

<img width="982" height="405" alt="image" src="https://github.com/user-attachments/assets/d60be440-580c-41d8-bce2-29582ffadc69" />

### 1️⃣ `SimulationHelper → SatHelper`: `CreateScenario()`

**Meaning:**  
The main simulation controller (`SimulationHelper`) calls `SatHelper` to begin building the scenario. This includes creating nodes and installing devices.

**Code:**
```cpp
simulationHelper->CreateSatScenario(satScenario);
```

---

### 2️⃣ `SatHelper → NetDevices`: `CreateNodes()`

**Meaning:**  
`SatHelper` creates the following nodes:
- Gateway Node (GW)
- User Terminal Node (UT)
- Satellite Node

**Code:**
```cpp
SatHelper::CreateNodes();  // Indirectly called in CreateSatScenario()
```

You can find `CreateObject<Node>()` in `sat-helper.cc`.

---

### 3️⃣ `SatHelper → NetDevices`: `InstallDevices()`

**Meaning:**  
Each node gets the appropriate NetDevice installed:
- GW/UT → `SatNetDevice`
- Satellite → `SatGeoNetDevice`

**Code:**
```cpp
SatHelper::InstallNetDevices();
netDevice = CreateObject<SatNetDevice>();
gwNode->AddDevice(netDevice);
```

---

### 4️⃣ `NetDevices → SatChannel`: `create channel`

**Meaning:**  
Devices are interconnected through `SatChannel`.

**Code:**
```cpp
Ptr<SatChannel> satChannel = CreateObject<SatChannel>();
netDevice->SetChannel(satChannel);
```

---

### 5️⃣ `TrafficHelper → GW/UT Nodes`: `InstallApps()`

**Meaning:**  
Applications are installed on GW/UT nodes:
- `OnOffApplication` (for data generation)
- `PacketSink` (for data reception)

**Code:**
```cpp
simulationHelper->GetTrafficHelper()->AddOnOffTraffic(...);
```

This is typically inside an `if (sender == "gw" || sender == "both")` block in the main program.

---

### 6️⃣ `StatsHelper → GW/UT Nodes`: `CollectStats()`

**Meaning:**  
The `SatStatsHelperContainer` begins to collect metrics such as throughput and delay.

**Code:**
```cpp
simulationHelper->SetOutputTag(scenario);  // Triggers statsHelperContainer creation/registration
```

---

### 7️⃣ `SimulationHelper → Simulator`: `Simulator::Run()`

**Meaning:**  
Starts the simulation execution.

**Code:**
```cpp
simulationHelper->RunSimulation();
```

---

### 8️⃣ `GW/UT/Sat Nodes → NetDevices → SatChannel`: `Send Packets using channel`

**Meaning:**  
`OnOffApplication` sends packets through `NetDevice` via `SatChannel`.

**Code:**
```cpp
onOffApp->SetStartTime(Seconds(1.0));
```

Actual chain:  
`OnOffApplication::StartApplication()` → `SendPacket()` → `Socket::SendTo()` → `NetDevice::Send()`

---

### 9️⃣ `SatChannel → NetDevices → GW/UT/Sat Nodes`: `Recv Packets using channel`

**Meaning:**  
Packets are received by the destination's `NetDevice` and passed to `PacketSink`.

**Code Chain:**
```cpp
SatChannel::Transmit() → NetDevice::Receive() → PacketSink::HandleRead()
```

---

### 🔟 `GW/UT/Sat Nodes → StatsHelper`: `Collect throughput/delay results`

**Meaning:**  
Packet arrival time and size are recorded to compute average throughput and delay.

**Code:**
```cpp
device->TraceConnectWithoutContext("PhyTxBegin", ...);
```
This is handled in the `TraceSink()` function inside `SatStatsHelperContainer`.

---
