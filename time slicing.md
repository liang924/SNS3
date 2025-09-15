<img width="260" height="502" alt="image" src="https://github.com/user-attachments/assets/1ba37b9b-af90-4336-a2fb-e82b1a338702" />



## Initialization (line 65-134)
```
SatFwdLinkSchedulerTimeSlicing::SatFwdLinkSchedulerTimeSlicing(Ptr<SatBbFrameConf> conf,
                                                               Mac48Address address,
                                                               double carrierBandwidthInHz)
    : SatFwdLinkScheduler(conf, address, carrierBandwidthInHz),
      m_lastSliceAssigned(1),
      m_lastSliceDequeued(1)
{
    NS_LOG_FUNCTION(this);

    ObjectBase::ConstructSelf(AttributeConstructionList());

    std::vector<SatEnums::SatModcod_t> modCods = conf->GetModCodsUsed();

    // Create control and broadcast container
    m_bbFrameContainers.insert(std::pair<uint8_t, Ptr<SatBbFrameContainer>>(
        0,
        CreateObject<SatBbFrameContainer>(modCods, m_bbFrameConf)));
    m_bbFrameContainers.at(0)->SetMaxSymbolRate(m_carrierBandwidthInHz);

    // Initialize containers
    for (uint8_t i = 0; i < m_numberOfSlices; i++)
    {
        Ptr<SatBbFrameContainer> container =
            CreateObject<SatBbFrameContainer>(modCods, m_bbFrameConf);
        container->SetMaxSymbolRate(m_carrierBandwidthInHz / m_numberOfSlices);
        m_bbFrameContainers.insert(std::pair<uint8_t, Ptr<SatBbFrameContainer>>(i + 1, container));
    }

    // Initialize number of symbols sent per slice
    for (uint8_t i = 0; i <= m_numberOfSlices; i++)
    {
        m_symbolsSent.insert(std::pair<uint8_t, uint32_t>(i, 0));
    }

    // Check if all symbol rates are high enough
    for (std::map<uint8_t, Ptr<SatBbFrameContainer>>::iterator it = m_bbFrameContainers.begin();
         it != m_bbFrameContainers.end();
         it++)
    {
        uint32_t maxSymbolPerCycle =
            it->second->GetMaxSymbolRate() * m_periodicInterval.GetSeconds();
        uint32_t symbolsMostRobustModcod;
        if (m_bbFrameConf->GetMostRobustModcod(SatEnums::NORMAL_FRAME) !=
            SatEnums::SAT_NONVALID_MODCOD)
        {
            symbolsMostRobustModcod = it->second->GetFrameSymbols(
                m_bbFrameConf->GetMostRobustModcod(SatEnums::NORMAL_FRAME));
        }
        else
        {
            // We are using only short Frames, as new ModCod exists for normal Frames.
            symbolsMostRobustModcod = it->second->GetFrameSymbols(
                m_bbFrameConf->GetMostRobustModcod(SatEnums::SHORT_FRAME));
        }
        if (symbolsMostRobustModcod > maxSymbolPerCycle)
        {
            NS_FATAL_ERROR("Symbol rate of slice " + std::to_string(it->first) + " (" +
                           std::to_string(it->second->GetMaxSymbolRate()) +
                           " Baud) is too low to allow at least one BBFrame of the most robust "
                           "ModCod. Must be at least " +
                           std::to_string((uint32_t)(symbolsMostRobustModcod /
                                                     m_periodicInterval.GetSeconds())) +
                           " Baud");
        }
    }

    Simulator::Schedule(m_periodicInterval,
                        &SatFwdLinkSchedulerTimeSlicing::PeriodicTimerExpired,
                        this);
}
```
Establish slice=0 (control/broadcast) and `BBFrameContainer` for each data slice, set the maximum symbol rate for each piece, initialize `m_symbolsSent`, check the minimum rate at which a Frame can be sent, and start the periodic timer.

## CollectServiceFlows 

```
SatFwdLinkSchedulerTimeSlicing::GetSchedulingObjects(std::vector<Ptr<SatSchedulingObject>>& output)
{
    NS_LOG_FUNCTION(this);

    if (GetTotalDuration() < m_periodicInterval)
    {
        // Get scheduling objects from LLC
        m_schedContextCallback(output);

        SortSchedulingObjects(output);
    }
}

```
Obtain `SatSchedulingObject` from LLC through `m_schedContextCallback(output)` and sort it.

## ComputeTimeSlices 
1. The first appearance of UT/Flow → Assign slice and issue subscription
```
 if ((m_slicesMapping.find(address) == m_slicesMapping.end()) &&
            (address != Mac48Address::GetBroadcast()))
        {
            m_slicesMapping.insert(std::pair<Mac48Address, uint8_t>(address, m_lastSliceAssigned));
            if (m_lastSliceAssigned == m_numberOfSlices)
            {
                m_lastSliceAssigned = 0;
            }
            m_lastSliceAssigned++;

            SendTimeSliceSubscription(address, std::vector<uint8_t>{m_slicesMapping.at(address)});

            // Begin again scheduling to insert slice subscription control packet.
            Simulator::Schedule(Seconds(0),
                                &SatFwdLinkSchedulerTimeSlicing::ScheduleBbFrames,
                                this);
            return;
        }
```
Newly appeared addresses are first assigned slices using Round-Robin method, and then immediately rescheduled after sending subscription messages.

2. Symbol rate/capacity constraints (whether new BBFrame can be opened)
```
SatFwdLinkSchedulerTimeSlicing::CanOpenBbFrame(Mac48Address address,
                                               uint32_t priorityClass,
                                               SatEnums::SatModcod_t modcod)
{
    NS_LOG_FUNCTION(this);

    if (priorityClass == 0)
    {
        // Always allow control messages to be send
        // TODO add a margin to take this into account ?
        return true;
    }

    uint8_t sliceId = (address == Mac48Address::GetBroadcast()) ? 0 : m_slicesMapping.at(address);
    double maxSymbolRate = m_bbFrameContainers.at(sliceId)->GetMaxSymbolRate();

    if (sliceId == 0)
    {
        // This is broadcast -> need to test all slices >= 1
        uint32_t symbols = GetSymbols(0, modcod);
        for (std::map<uint8_t, Ptr<SatBbFrameContainer>>::iterator it = m_bbFrameContainers.begin();
             it != m_bbFrameContainers.end();
             it++)
        {
            if (it->first == 0)
            {
                continue;
            }
            double symbolRate = (symbols + GetSymbols(it->first, SatEnums::SAT_NONVALID_MODCOD)) /
                                m_periodicInterval.GetSeconds();
            if (symbolRate > maxSymbolRate)
            {
                // One slice will not respect constraints
                return false;
            }
        }
        // Constraints are respected for all slices
        return true;
    }
    else
    {
        uint32_t symbols =
            GetSymbols(sliceId, modcod) + GetSymbols(0, SatEnums::SAT_NONVALID_MODCOD);
        double symbolRate = symbols / m_periodicInterval.GetSeconds();
        return symbolRate < maxSymbolRate;
    }
}

uint32_t
SatFwdLinkSchedulerTimeSlicing::GetSymbols(uint8_t sliceId, SatEnums::SatModcod_t modcod)
{
    NS_LOG_FUNCTION(this);

    uint32_t symbols =
        m_bbFrameContainers.at(sliceId)->GetTotalDuration().GetSeconds() * m_carrierBandwidthInHz;

    if (modcod != SatEnums::SAT_NONVALID_MODCOD)
    {
        symbols += m_bbFrameContainers.at(sliceId)->GetFrameSymbols(modcod);
    }

    return symbols;
}
```
Using the maximum symbol rate of each piece as a limitation, the current/planned number of symbols is calculated to determine whether a new BBFrame can be opened; when broadcasting slice=0, all data slices need to be checked simultaneously.

## AssignFlowsToSlices 
```
 uint8_t slice = (address == Mac48Address::GetBroadcast()) ? 0 : m_slicesMapping.at(address);
        SatEnums::SatModcod_t modcod =
            m_bbFrameContainers.at(slice)->GetModcod(flowId, GetSchedulingObjectCno(*it));

        uint32_t frameBytes =
            m_bbFrameContainers.at(slice)->GetBytesLeftInTailFrame(flowId, modcod);

        if ((m_bbFrameContainers.at(slice)->IsEmpty(flowId, modcod)) && (currentObBytes > 0) &&
            !CanOpenBbFrame(address, flowId, modcod))
        {
            continue;
        }

        while ((GetTotalDuration() < m_periodicInterval) && (currentObBytes > 0))
        {
            if (frameBytes < currentObMinReqBytes)
            {
                frameBytes =
                    m_bbFrameContainers.at(slice)->GetMaxFramePayloadInBytes(flowId, modcod) -
                    m_bbFrameConf->GetBbFrameHeaderSizeInBytes();

                if (!CanOpenBbFrame(address, flowId, modcod))
                {
                    break;
                }

                // if frame bytes still too small, we must have too long control message, so let's
                // crash
                if (frameBytes < currentObMinReqBytes)
                {
                    NS_FATAL_ERROR("Control package probably too long!!!");
                }
            }

            Ptr<Packet> p = m_txOpportunityCallback(frameBytes,
                                                    address,
                                                    flowId,
                                                    currentObBytes,
                                                    currentObMinReqBytes);

            if (p)
            {
                if ((flowId == 0) || (address == Mac48Address::GetBroadcast()))
                {
                    m_bbFrameContainers.at(0)->AddData(flowId, modcod, p);
                }
                else
                {
                    m_bbFrameContainers.at(slice)->AddData(flowId, modcod, p);
                    frameBytes =
                        m_bbFrameContainers.at(slice)->GetBytesLeftInTailFrame(flowId, modcod);
                }
            }
            else if (m_bbFrameContainers.at(slice)->GetMaxFramePayloadInBytes(flowId, modcod) !=
                     m_bbFrameContainers.at(slice)->GetBytesLeftInTailFrame(flowId, modcod))
            {
                frameBytes =
                    m_bbFrameContainers.at(slice)->GetMaxFramePayloadInBytes(flowId, modcod);

                if (!CanOpenBbFrame(address, flowId, modcod))
                {
                    break;
                }
            }
            else
            {
                NS_FATAL_ERROR("Packet does not fit in empty BB Frame. Control package too long or "
                               "fragmentation problem in user package!!!");
            }
        }

        m_bbFrameContainers.at(slice)->MergeBbFrames(m_carrierBandwidthInHz);
    }
}
```
Determine which slice this flow belongs to, select modcod, calculate the remaining payload, request packets from LLC (`m_txOpportunityCallback`), and join the corresponding slice or control slice.

## BuildBbFrames
```
m_bbFrameContainers.at(slice)->MergeBbFrames(m_carrierBandwidthInHz);
```
Merge the accumulated payloads within the slice into a sendable BB Frame.

## SendTimeSliceSubscription

```
SatFwdLinkSchedulerTimeSlicing::SendTimeSliceSubscription(Mac48Address address,
                                                          std::vector<uint8_t> slices)
{
    NS_LOG_FUNCTION(this);

    for (std::vector<uint8_t>::iterator it = slices.begin(); it != slices.end(); ++it)
    {
        Ptr<SatSliceSubscriptionMessage> sliceSubscription =
            CreateObject<SatSliceSubscriptionMessage>();
        sliceSubscription->SetSliceId(*it);
        sliceSubscription->SetAddress(address);

        m_sendControlMsgCallback(sliceSubscription, Mac48Address::GetBroadcast());
    }
}
```
`SendTimeSliceSubscription()` iterates through the slices vector:
- For each slice ID, it creates a SatSliceSubscriptionMessage object.
- It sets the slice ID and the user’s MAC address via SetSliceId() and SetAddress().
- It then calls the m_sendControlMsgCallback function to send the control message, typically using broadcast, so that the intended UTs know which slice they are assigned to.

## Transmission
```
std::pair<Ptr<SatBbFrame>, const Time>
SatFwdLinkSchedulerTimeSlicing::GetNextFrame()
{
    NS_LOG_FUNCTION(this);

    Ptr<SatBbFrame> frame;
    Time frameDuration;

    // Send slice control messages first if there is any.
    if (!m_bbFrameContainers.at(0)->IsEmpty(0, m_bbFrameConf->GetDefaultModCod()))
    {
        frame = m_bbFrameContainers.at(0)->GetNextFrame();
        if (frame != nullptr)
        {
            frame->SetSliceId(0);
            frameDuration = frame->GetDuration();
            m_symbolsSent.at(0) += ceil(frame->GetDuration().GetSeconds() * m_carrierBandwidthInHz);
        }
        else
        {
            frameDuration = m_bbFrameConf->GetDummyBbFrameDuration();
        }
    }
    else
    {
        uint8_t firstDeque = m_lastSliceDequeued;
        do
        {
            uint32_t symbols = m_symbolsSent.at(m_lastSliceDequeued) + m_symbolsSent.at(0);
            double maxSymbolRate = m_bbFrameContainers.at(m_lastSliceDequeued)->GetMaxSymbolRate();

            frame = m_bbFrameContainers.at(m_lastSliceDequeued)->GetNextFrame();
            if (frame != nullptr)
            {
                m_symbolsSent.at(m_lastSliceDequeued) +=
                    ceil(frame->GetDuration().GetSeconds() * m_carrierBandwidthInHz);
                symbols += ceil(frame->GetDuration().GetSeconds() * m_carrierBandwidthInHz);
                frame->SetSliceId(m_lastSliceDequeued);
                frameDuration = frame->GetDuration();

                if (symbols / m_periodicInterval.GetSeconds() > maxSymbolRate)
                {
                    NS_LOG_WARN("Symbol rate not respected for slice " +
                                std::to_string(m_lastSliceDequeued) + ". Got " +
                                std::to_string(symbols / m_periodicInterval.GetSeconds()) + "Baud" +
                                " while max is " + std::to_string(maxSymbolRate) + " Baud");
                }
            }
            if (m_lastSliceDequeued == m_numberOfSlices)
            {
                m_lastSliceDequeued = -1;
            }
            m_lastSliceDequeued++;
        } while (frame == nullptr && m_lastSliceDequeued != firstDeque);
    }

    // create dummy frame
    if (m_dummyFrameSendingEnabled && frame == nullptr)
    {
        frame = Create<SatBbFrame>(m_bbFrameConf->GetDefaultModCod(),
                                   SatEnums::DUMMY_FRAME,
                                   m_bbFrameConf);

        // create dummy packet
        Ptr<Packet> dummyPacket = Create<Packet>(1);

        // Add MAC tag
        SatMacTag mTag;
        mTag.SetDestAddress(Mac48Address::GetBroadcast());
        mTag.SetSourceAddress(m_macAddress);
        dummyPacket->AddPacketTag(mTag);

        // Add E2E address tag
        SatAddressE2ETag addressE2ETag;
        addressE2ETag.SetE2EDestAddress(Mac48Address::GetBroadcast());
        addressE2ETag.SetE2ESourceAddress(m_macAddress);
        dummyPacket->AddPacketTag(addressE2ETag);

        // Add dummy packet to dummy frame
        frame->AddPayload(dummyPacket);
        frame->SetSliceId(0);
        frameDuration = frame->GetDuration();
    }
    // If no bb frame available and dummy frames disabled
    else if (frame == nullptr)
    {
        frameDuration = m_bbFrameConf->GetDummyBbFrameDuration();
    }

    return std::make_pair(frame, frameDuration);
}
```
First send the control slice(0), then retrieve the frames from the data slice in a Round-Robin manner; update the symbol counts for each slice, and if there is no data to send, a Dummy Frame may be sent.

<img width="1704" height="545" alt="image" src="https://github.com/user-attachments/assets/b98c89cd-c0b7-4ccf-9503-7e3c5a479d64" />
