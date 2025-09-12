<img width="321" height="657" alt="image" src="https://github.com/user-attachments/assets/dfad3800-1be6-4792-891f-923d53b951ce" />


## Initialization (line 65-134)

This code represents the initialization phase of `SatFwdLinkSchedulerTimeSlicing`. It begins by invoking the base class constructor and setting up internal state, then creates a control/broadcast slice container along with multiple user slice containers, assigning each a maximum symbol rate. It initializes symbol counters for all slices and verifies that the most robust modulation and coding scheme (ModCod) can support at least one BBFrame within each slice’s symbol rate. Finally, it schedules a periodic timer to trigger subsequent scheduling operations.

## PeriodicTimerExpired 

```
SatFwdLinkSchedulerTimeSlicing::PeriodicTimerExpired()
{
    NS_LOG_FUNCTION(this);

    SendAndClearSymbolsSentStat();
    ScheduleBbFrames();

    Simulator::Schedule(m_periodicInterval,
                        &SatFwdLinkSchedulerTimeSlicing::PeriodicTimerExpired,
                        this);
}
```
`PeriodicTimerExpired()` is invoked whenever the periodic timer expires. It first clears and reports symbol transmission statistics (`SendAndClearSymbolsSentStat()`), then executes the core scheduling logic (`ScheduleBbFrames()`), and finally re-schedules itself for the next interval to ensure the process continues periodically.

## SendAndClearSymbols 
```
SatFwdLinkSchedulerTimeSlicing::SendAndClearSymbolsSentStat()
{
    NS_LOG_FUNCTION(this);

    for (std::map<uint8_t, uint32_t>::iterator it = m_symbolsSent.begin();
         it != m_symbolsSent.end();
         it++)
    {
        m_schedulingSymbolRateTrace(it->first, it->second / Seconds(1).GetSeconds());
    }

    m_symbolsSent.clear();

    for (uint8_t i = 0; i <= m_numberOfSlices; i++)
    {
        m_symbolsSent.insert(std::pair<uint8_t, uint32_t>(i, 0));
    }
}
```
`SendAndClearSymbolsSentStat()` iterates through all slice symbol counters and logs their transmission rates (symbols sent per second) to the tracing system for statistics. It then clears `m_symbolsSent` and re-initializes the counters for all slices, resetting them to zero in preparation for the next scheduling cycle.

## ScheduleBbFrames (line 287-395)

This function is the **core scheduling routine** of `SatFwdLinkSchedulerTimeSlicing`, responsible for assigning packets into the appropriate slices’ BBFrames.

- **Retrieve flows**: Calls `GetSchedulingObjects()` to obtain the current list of flows that need scheduling.  
- **Slice assignment**: If a user (identified by its MAC address) has not been assigned a slice yet, it allocates one and immediately sends a slice subscription control message.  
- **Select ModCod**: Determines the modulation and coding scheme based on the C/N₀ of the flow.  
- **Check available space**: If the tail BBFrame does not have enough space, attempts to open a new one; if still insufficient, raises a fatal error.  
- **Insert packets**: Uses `m_txOpportunityCallback()` to request packets from the LLC. If successful, the packets are placed into the slice’s BBFrame.  
- **Error handling**: If a packet cannot fit even in an empty BBFrame, a fatal error is raised (indicating overly large control packets or fragmentation problems).  
- **Merge frames**: After scheduling, calls `MergeBbFrames()` to consolidate frames within the container.  

## GetSchedulingObjects
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
GetSchedulingObjects() checks the current BBFrames total duration (GetTotalDuration()) and only runs if it is still below the periodic interval (m_periodicInterval).

- It invokes m_schedContextCallback(output), which is a callback function registered by the LLC layer. This callback fills the output vector with the flows available for scheduling.
- Then it calls SortSchedulingObjects(output) to arrange the flows based on predefined criteria (such as Flow ID, head-of-line delay, or buffer load).

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

