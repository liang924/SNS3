<img width="213" height="539" alt="image" src="https://github.com/user-attachments/assets/44d428d7-3434-412b-88ca-888d48fd49c9" />

1. Initialization
```
SatFwdLinkSchedulerDefault::SatFwdLinkSchedulerDefault(Ptr<SatBbFrameConf> conf,
                                                       Mac48Address address,
                                                       double carrierBandwidthInHz)
    : SatFwdLinkScheduler(conf, address, carrierBandwidthInHz),
      m_symbolsSent(0)
{
    NS_LOG_FUNCTION(this);

    ObjectBase::ConstructSelf(AttributeConstructionList());

    std::vector<SatEnums::SatModcod_t> modCods = conf->GetModCodsUsed();

    m_bbFrameContainer = CreateObject<SatBbFrameContainer>(modCods, m_bbFrameConf);

    Simulator::Schedule(m_periodicInterval,
                        &SatFwdLinkSchedulerDefault::PeriodicTimerExpired,
                        this);
}

SatFwdLinkSchedulerDefault::~SatFwdLinkSchedulerDefault()
{
    NS_LOG_FUNCTION(this);
}
```
Set the Start/Stop Threshold, create the `BBFrameContainer`, and start the cycle timer.

2. PeriodicTimerExpired
```
SatFwdLinkSchedulerDefault::PeriodicTimerExpired()
{
    NS_LOG_FUNCTION(this);

    SendAndClearSymbolsSentStat();
    ScheduleBbFrames();

    Simulator::Schedule(m_periodicInterval,
                        &SatFwdLinkSchedulerDefault::PeriodicTimerExpired,
                        this);
}
```
Scheduled call, clear statistics, execute a new round of scheduling.

3. SendAndClearSymbols
```
SatFwdLinkSchedulerDefault::SendAndClearSymbolsSentStat()
{
    NS_LOG_FUNCTION(this);

    m_schedulingSymbolRateTrace(0, m_symbolsSent / Seconds(1).GetSeconds());

    m_symbolsSent = 0;
}
```

4. ScheduleBbFrames
```
SatFwdLinkSchedulerDefault::ScheduleBbFrames()
{
    NS_LOG_FUNCTION(this);

    // Get scheduling objects from LLC
    std::vector<Ptr<SatSchedulingObject>> so;
    GetSchedulingObjects(so);

    for (std::vector<Ptr<SatSchedulingObject>>::const_iterator it = so.begin();
         (it != so.end()) &&
         (m_bbFrameContainer->GetTotalDuration() < m_schedulingStopThresholdTime);
         it++)
    {
        uint32_t currentObBytes = (*it)->GetBufferedBytes();
        uint32_t currentObMinReqBytes = (*it)->GetMinTxOpportunityInBytes();
        uint8_t flowId = (*it)->GetFlowId();
        SatEnums::SatModcod_t modcod =
            m_bbFrameContainer->GetModcod(flowId, GetSchedulingObjectCno(*it));

        uint32_t frameBytes = m_bbFrameContainer->GetBytesLeftInTailFrame(flowId, modcod);

        while (((m_bbFrameContainer->GetTotalDuration() < m_schedulingStopThresholdTime)) &&
               (currentObBytes > 0))
        {
            if (frameBytes < currentObMinReqBytes)
            {
                frameBytes = m_bbFrameContainer->GetMaxFramePayloadInBytes(flowId, modcod) -
                             m_bbFrameConf->GetBbFrameHeaderSizeInBytes();

                // if frame bytes still too small, we must have too long control message, so let's
                // crash
                if (frameBytes < currentObMinReqBytes)
                {
                    NS_FATAL_ERROR("Control package too probably too long!!!");
                }
            }

            Ptr<Packet> p = m_txOpportunityCallback(frameBytes,
                                                    (*it)->GetMacAddress(),
                                                    flowId,
                                                    currentObBytes,
                                                    currentObMinReqBytes);

            if (p)
            {
                m_bbFrameContainer->AddData(flowId, modcod, p);
                frameBytes = m_bbFrameContainer->GetBytesLeftInTailFrame(flowId, modcod);
            }
            else if (m_bbFrameContainer->GetMaxFramePayloadInBytes(flowId, modcod) !=
                     m_bbFrameContainer->GetBytesLeftInTailFrame(flowId, modcod))
            {
                frameBytes = m_bbFrameContainer->GetMaxFramePayloadInBytes(flowId, modcod);
            }
            else
            {
                NS_FATAL_ERROR("Packet does not fit in empty BB Frame. Control package too long or "
                               "fragmentation problem in user package!!!");
            }
        }

        m_bbFrameContainer->MergeBbFrames(m_carrierBandwidthInHz);
    }
}
```
Retrieve Scheduling Objects from LLC and pack them into BBFrame as required.

5. GetSchedulingObjects
```
SatFwdLinkSchedulerDefault::GetSchedulingObjects(std::vector<Ptr<SatSchedulingObject>>& output)
{
    NS_LOG_FUNCTION(this);

    if (m_bbFrameContainer->GetTotalDuration() < m_schedulingStopThresholdTime)
    {
        // Get scheduling objects from LLC
        m_schedContextCallback(output);

        SortSchedulingObjects(output);
    }
}
```
Obtain objects from LLC and sort them.

6. GetNextFrame
```
std::pair<Ptr<SatBbFrame>, const Time>
SatFwdLinkSchedulerDefault::GetNextFrame()
{
    NS_LOG_FUNCTION(this);

    if (m_bbFrameContainer->GetTotalDuration() < m_schedulingStartThresholdTime)
    {
        ScheduleBbFrames();
    }

    Ptr<SatBbFrame> frame = m_bbFrameContainer->GetNextFrame();
    Time frameDuration;

    if (frame != nullptr)
    {
        m_symbolsSent += ceil(frame->GetDuration().GetSeconds() * m_carrierBandwidthInHz);
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

        frameDuration = frame->GetDuration();
    }
    // If no bb frame available and dummy frames disabled
    else if (frame == nullptr)
    {
        frameDuration = m_bbFrameConf->GetDummyBbFrameDuration();
    }

    if (frame != nullptr)
    {
        frameDuration = frame->GetDuration();
        frame->SetSliceId(0);
    }

    return std::make_pair(frame, frameDuration);
}
```
- If the Frame is insufficient, call ScheduleBbFrames().
- Return the next Frame.
- If no data is available → possibly create a Dummy Frame.

7. SendTimeSliceSubscription
There is no direct `SendTimeSliceSubscription` function in the code.  
  - This block in the flowchart usually corresponds to the **system issuing time slicing control messages**.  
  - In the implementation, this is indirectly achieved through the interaction between **`m_txOpportunityCallback` and the LLC**.

8. CanOpenBbFrame / GetSymbols
- **Code location**:
  - `m_bbFrameContainer->GetTotalDuration()`
  - `frame->GetDuration()`
  - `m_symbolsSent` (accumulated symbol count)

- Check whether the Frame has enough capacity and calculate/return the number of symbols.
