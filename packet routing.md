## Flowchart
<img width="857" height="560" alt="image" src="https://github.com/user-attachments/assets/a3a044d2-ad9a-47ba-bfa2-3aa8243ac477" />

1. Example main
> In the examples/ folder, there are multiple example programs.Each of these examples represents a different simulation scenario, but the flow always calls SatHelper and SatBeamHelper to install devices and assign beams.In other words, regardless of which example is used, the entry point is always main(), with the difference lying in the simulation configuration (Lora / Return link / Forward link).

2. SatHelper::CreateScenario()
```
SatHelper::CreateSimpleScenario()
{
    NS_LOG_FUNCTION(this);

    SatBeamUserInfo beamInfo = SatBeamUserInfo(1, 1);
    BeamUserInfoMap_t beamUserInfos;
    beamUserInfos[std::make_pair(0, 8)] = beamInfo;

    DoCreateScenario(beamUserInfos, 1);

    m_creationSummaryTrace("*** Simple Scenario Creation Summary ***");
}

void
SatHelper::CreateLargerScenario()
{
    NS_LOG_FUNCTION(this);

    // install one user for UTs in beams 12 and 22
    SatBeamUserInfo beamInfo = SatBeamUserInfo(1, 1);
    BeamUserInfoMap_t beamUserInfos;

    beamUserInfos[std::make_pair(0, 12)] = beamInfo;
    beamUserInfos[std::make_pair(0, 22)] = beamInfo;

    // install two users for UT1 and one for UT2 in beam 3
    beamInfo.SetUtUserCount(0, 2);
    beamInfo.AppendUt(1);

    beamUserInfos[std::make_pair(0, 3)] = beamInfo;

    DoCreateScenario(beamUserInfos, 1);

    m_creationSummaryTrace("*** Larger Scenario Creation Summary ***");
}

void
SatHelper::CreateFullScenario()
{
    NS_LOG_FUNCTION(this);

    uint32_t beamCount = m_satConf->GetBeamCount();
    BeamUserInfoMap_t beamUserInfos;

    for (uint32_t i = 1; i < (beamCount + 1); i++)
    {
        BeamUserInfoMap_t::iterator beamInfo = m_beamUserInfos.find(std::make_pair(0, i));
        SatBeamUserInfo info;

        if (beamInfo != m_beamUserInfos.end())
        {
            info = beamInfo->second;
        }
        else
        {
            info = SatBeamUserInfo(m_utsInBeam, this->m_utUsers);
        }

        beamUserInfos[std::make_pair(0, i)] = info;
    }

    DoCreateScenario(beamUserInfos, m_gwUsers);

    m_creationSummaryTrace("*** Full Scenario Creation Summary ***");
}

void
SatHelper::CreateUserDefinedScenario(BeamUserInfoMap_t& infos)
{
    NS_LOG_FUNCTION(this);

    // create as user wants
    DoCreateScenario(infos, m_gwUsers);

    m_creationSummaryTrace("*** User Defined Scenario Creation Summary ***");
}
```
`SatHelper` will divide the setup into different Scenarios (Simple, Larger, Full, UserDefined) depending on the situation you want.

3. SatHelper::DoCreateScenario()
> `satellite-helper.cc` line 678-980
> 
> `SetNetworkAddresses()` line 691  → Assigns IP addresses and establishes the logical network topology.
>
> `SetGwMobility() / SetUtMobility()` line 708 → Determines the positions and mobility behavior of the Gateways (GW) and User Terminals (UT), affecting beam coverage and handover.
>
> `m_beamHelper->Install(...)` line 798 → Installs NetDevices, assigns beams/frequencies, and triggers routing updates.


### satellite-beam-helper
1. Distribute beamID for UT/GW
   
```
std::pair<std::map<std::pair<uint32_t, uint32_t>, uint32_t>::iterator, bool> beam =
m_beam.insert(std::make_pair(std::make_pair(satId, beamId), gwId));
NS_ASSERT(beam.second == true);
```
Map (satId, beamId) to gwId to ensure that each beam corresponds to the correct Gateway.

2. Distribute freqID for UL/DL

```
SatChannelPair::ChannelPair_t feederLink =
GetChannelPair(satId, beamId, fwdFlFreqId, rtnFlFreqId, false);
SatChannelPair::ChannelPair_t userLink =
GetChannelPair(satId, beamId, fwdUlFreqId, rtnUlFreqId, true);
```
Create feeder link (GW ↔ Sat) and user link (UT ↔ Sat) channels, and allocate uplink/downlink paths according to the input freqId (UL/DL frequency IDs).

3. InstallFeeder()/InstallUser()/Return (GW NetDev, UT Devices)

```
line 677-732
```
Create and install the Gateway NetDevice (gwNd) and the User Terminal NetDevices (utNd), and finally return these two.

4. Call UpdateUtRoutes()

```
Ptr<NetDevice> gwNd = InstallFeeder(DynamicCast<SatOrbiterNetDevice>(satNode->GetDevice(0)),
                                        gwNode,
                                        gwId,
                                        satId,
                                        beamId,
                                        feederSatId,
                                        feederBeamId,
                                        feederLink,
                                        rtnFlFreqId,
                                        fwdFlFreqId,
                                        routingCallback);
```
```
utNd = InstallUser(DynamicCast<SatOrbiterNetDevice>(satNode->GetDevice(0)),
                           ut,
                           gwNd,
                           satId,
                           beamId,
                           userLink,
                           rtnUlFreqId,
                           fwdUlFreqId,
                           routingCallback);
```
`routingCallback` is passed into InstallFeeder / InstallUser.

5. Update IPv4 Routing Table

```
Ipv4StaticRoutingHelper ipv4RoutingHelper;
Ptr<Ipv4StaticRouting> routing = ipv4RoutingHelper.GetStaticRouting(protocol);
routing->RemoveRoute(routing->GetNRoutes() - 1);
routing->SetDefaultRoute(ip, utIfIndex);  //Add a default route, so that all packets from the UT that cannot be delivered will be forwarded to the Gateway’s IP (ip).
```
