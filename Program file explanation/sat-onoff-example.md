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
這個類別是整個模擬的主控制器，所有場景建構、應用安裝、執行模擬等都透過它進行。

2. SatHelper

```
simulationHelper->CreateSatScenario(satScenario);
```
由 SimulationHelper 呼叫，用來節點建立（GW/UT/Sat),裝置安裝（NetDevice),通道連線（SatChannel）

3. SatTrafficHelper

```
simulationHelper->GetTrafficHelper()->AddOnOffTraffic(...);
```

> 這個 GetTrafficHelper() 回傳的物件就是 SatTrafficHelper，負責：
> - 安裝 OnOffApplication
> - 設定 PacketSink
> - 指定應用參數（資料率、on/off 時間等）

4. SatStatsHelperContainer

```
simulationHelper->SetOutputTag(scenario);
```
> 它負責記錄：
> - 每秒吞吐量
> - 封包數量
> - 延遲等資訊

5. NodeContainer 與節點

📍 在 SatHelper 中你可以找到：

```
Ptr<Node> gateway = CreateObject<Node>();
NodeContainer utUsers = CreateObject<NodeContainer>();
Ptr<Node> satellite = CreateObject<Node>();
```

6. SatNetDevice / SatGeoNetDevice

📍 這段在 SatHelper::InstallNetDevices() 裡：
```
Ptr<NetDevice> dev = CreateObject<SatNetDevice>();
node->AddDevice(dev);
```
衛星會安裝 SatGeoNetDevice，地面節點（UT/GW）會安裝 SatNetDevice。

7. SatChannel

📍 在 SatHelper::InstallChannels() 中：
```
Ptr<SatChannel> channel = CreateObject<SatChannel>();
satNetDevice->SetChannel(channel);
```
所有節點的裝置透過這個 Channel 連線，用來模擬延遲、頻寬、干擾等傳輸條件。
