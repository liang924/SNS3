
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

// Read CLI arguments
CommandLine cmd;
cmd.AddValue("packetSize", ...);
cmd.AddValue("dataRate", ...);
cmd.AddValue("onTime", ...);
cmd.AddValue("offTime", ...);
cmd.AddValue("sender", ...);
cmd.AddValue("scenario", ...);
cmd.AddValue("simDuration", ...);
simulationHelper->AddDefaultUiArguments(cmd);
cmd.Parse(argc, argv);
```

### 4.Load satellite scenario

```
simulationHelper->LoadScenario("geo-33E");
```

### 5.Create satellite scenario

```
SatHelper::PreDefinedScenario_t satScenario = SatHelper::SIMPLE;

if (scenario == "larger") {
    satScenario = SatHelper::LARGER;
} else if (scenario == "full") {
    satScenario = SatHelper::FULL;
}

simulationHelper->CreateSatScenario(satScenario);
```

### 6.Install network devices and connect channels

> These steps are called inside CreateSatScenario():

```
// Inside CreateSatScenario()
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
simulationHelper->SetOutputTag(scenario);
// internally triggers SatStatsHelperContainer creation
```

### 9.Run simulation

```
simulationHelper->RunSimulation();
```

