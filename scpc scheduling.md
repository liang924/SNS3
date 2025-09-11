## Architecture

<img width="695" height="762" alt="image" src="https://github.com/user-attachments/assets/601385af-308e-42e8-abe8-a76aab6a829b" />


1. SatScpcScheduler

```
SatScpcScheduler::SatScpcScheduler(Ptr<SatBbFrameConf> conf,
                                   Mac48Address address,
                                   double carrierBandwidthInHz)
    : SatFwdLinkScheduler(conf, address, carrierBandwidthInHz),
      m_symbolsSent(0)
{
    NS_LOG_FUNCTION(this);

    ObjectBase::ConstructSelf(AttributeConstructionList());

    // Obtain the available modulation and coding set (ModCods).
    std::vector<SatEnums::SatModcod_t> modCods = conf->GetModCodsUsed();

    // Create a BB Frame container to store the baseband frames to be transmitted.
    m_bbFrameContainer = CreateObject<SatBbFrameContainer>(modCods, m_bbFrameConf);

    // Start the periodic scheduler (PeriodicTimerExpired).
    Simulator::Schedule(m_periodicInterval, &SatScpcScheduler::PeriodicTimerExpired, this);
}
```
Initialize the scheduler by creating the BB frame container and starting the periodic timer to trigger the scheduling process.

2. PeriodicTimerExpired()
```
SatScpcScheduler::PeriodicTimerExpired()
{
    NS_LOG_FUNCTION(this);

    SendAndClearSymbolsSentStat();
    ScheduleBbFrames();

    Simulator::Schedule(m_periodicInterval, &SatScpcScheduler::PeriodicTimerExpired, this);
}
```
On each periodic trigger, it clears and reports symbol statistics, schedules new BB frames, and reschedules the timer to continue the process.On each periodic trigger, it clears and reports symbol statistics, schedules new BB frames, and reschedules the timer to continue the process.

3. SendAndClearSymbols

```
SatScpcScheduler::SendAndClearSymbolsSentStat()
{
    NS_LOG_FUNCTION(this);

    m_schedulingSymbolRateTrace(0, m_symbolsSent / Seconds(1).GetSeconds());

    m_symbolsSent = 0;
}
```
Reports and clears the number of transmitted symbols, resetting the counter for the next round.

4. ScheduleBbFrames()
   
```
void
SatScpcScheduler::ScheduleBbFrames()
{
  NS_LOG_FUNCTION(this);

  // If enough frames already exist (exceeding the stop threshold) → exit immediately.
  if (m_bbFrameContainer->GetTotalDuration() >= m_schedulingStopThresholdTime)
    {
      return;
    }

  // Obtain the flows that need to be scheduled (via callback).
  SchedulingObjectList_t schedulingObjects = GetSchedulingObjects();

  // Process each flow one by one.
  for (auto& schedulingObject : schedulingObjects)
    {
      // Obtain the flow’s buffer, minimum transmission size (minTx), and carrier-to-noise ratio (C/N0).
      uint32_t buffer = schedulingObject->GetBuffer();
      uint32_t minTx  = schedulingObject->GetMinTx();
      double cno      = schedulingObject->GetCno();

      // While there is still data and the stop threshold has not been exceeded.
      while (buffer > 0 &&
             m_bbFrameContainer->GetTotalDuration() < m_schedulingStopThresholdTime)
        {
          // Check whether the frame has sufficient space.
          if (m_bbFrameContainer->GetTailFrameFreeSpace() < minTx)
            {
              // Not enough → open a new frame.
              m_bbFrameContainer->OpenNewFrame();
            }

          // Request a packet from the LLC callback.
          Ptr<Packet> p = m_txOpportunityCallback(minTx);

          if (p)
            {
              // Success → insert the packet into the frame.
              m_bbFrameContainer->AddData(schedulingObject, p);
              buffer -= p->GetSize();
            }
          else
            {
              // No packet obtained → check the frame status.
              if (m_bbFrameContainer->GetTailFrame()->IsEmpty())
                {
                  NS_FATAL_ERROR("Cannot get packet for an empty frame!");
                }
              else
                {
                  // Open a new frame and continue.
                  m_bbFrameContainer->OpenNewFrame();
                }
            }
        }
    }

  // Merge the frames and organize the output.
  m_bbFrameContainer->MergeFrames();
}
```
Schedules packets from each flow into the BB frame container based on demand and constraints, until the stop threshold is reached or no packets remain.
