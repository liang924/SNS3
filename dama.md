<img width="271" height="552" alt="image" src="https://github.com/user-attachments/assets/45b1b6d8-4191-4a50-8adf-d80f971adc71" />

- SatDamaEntry/CR update – Process the received Capacity Requests (CR) within the previous superframe.
- Preliminary resource allocation – Pre-allocate a set of soft-symbols for each UT based on configured CRA, dynamic request type (RBDC, VBDC) and value, CNo conditions and frame configurations and load.
- Time slot generation – generate the time slots for each frame based on the pre-allocated soft-symbols for each UT and RC index. Fill in the TBTP on-the-fly.
- SatDamaEntry update – Update the allocated VBDC bytes for each UT context
- TBTP signaling – Send the TBTP message to the proper GW protocol stack handling the resources for this specific spot-beam.
- Schedule next scheduling time for the next SF.

[DAMA](https://github.com/sns3/sns3-satellite/blob/master/model/satellite-dama-entry.cc)\

### NCC receives CR → calls SatDamaEntry to update
- `SatDamaEntry::UpdateRbdcInKbps(uint8_t index, uint16_t rateInKbps)` -- Update the Rate-Based Dynamic Capacity (RBDC) request.
  - Capacity requests are made in terms of rate (kbps).
  - The NCC allocates the corresponding time slots to the user based on the RBDC request.
- `SatDamaEntry::UpdateVbdcInBytes(uint8_t index, uint32_t volumeInBytes)` -- Update the Volume-Based Dynamic Capacity (VBDC) backlog request.
  - The user tells the network: “I currently have X bytes waiting to be transmitted; please allocate me the corresponding capacity.”
- `SatDamaEntry::SetVbdcInBytes(uint8_t index, uint32_t volumeInBytes)`
  - Directly sets the backlog value and checks whether it exceeds the maximum limit.
 
### SatDamaEntry Update Status + Manage Lifecycle

```
SatDamaEntry::ResetDynamicRatePersistence()
{
    NS_LOG_FUNCTION(this);

    m_dynamicRatePersistence = m_llsConf->GetDynamicRatePersistence();
}

void
SatDamaEntry::DecrementDynamicRatePersistence()
{
    NS_LOG_FUNCTION(this);

    if (m_dynamicRatePersistence > 0)
    {
        m_dynamicRatePersistence--;
    }

    if (m_dynamicRatePersistence == 0)
    {
        std::fill(m_dynamicRateRequestedInKbps.begin(), m_dynamicRateRequestedInKbps.end(), 0.0);
    }
}
```
Control the duration of RBDC requests, automatically resetting upon expiration.

```
SatDamaEntry::ResetVolumeBacklogPersistence()
{
    NS_LOG_FUNCTION(this);

    m_volumeBacklogPersistence = m_llsConf->GetVolumeBacklogPersistence();
}

void
SatDamaEntry::DecrementVolumeBacklogPersistence()
{
    NS_LOG_FUNCTION(this);

    if (m_volumeBacklogPersistence > 0)
    {
        m_volumeBacklogPersistence--;
    }

    if (m_volumeBacklogPersistence == 0)
    {
        std::fill(m_volumeBacklogRequestedInBytes.begin(),
                  m_volumeBacklogRequestedInBytes.end(),
                  0);
    }
}
```
Control the duration of VBDC requests, automatically resetting upon expiration.

**The persistence mechanism ensures that the CR will not remain permanently effective.**

### Scheduler next round scheduling use (check CR status)
- `SatDamaEntry::GetRbdcInKbps(uint8_t index) const` -- Query the current RBDC value of a certain service.
- `SatDamaEntry::GetVbdcBasedBytes() const` -- Query the total sum of all backlogs.
- `SatDamaEntry::GetCraBasedBytes(Time duration) const` -- Calculate the total bytes that CRA can send over a certain period of time.
- `SatDamaEntry::GetRbdcBasedBytes(Time duration) const` -- Calculate the total bytes that RBDC can transmit over a certain period of time.
- `SatDamaEntry::GetMinRateBasedBytes(Time duration) const` -- Calculate the number of bytes that can be sent at the minimum rate.

**These query functions are called by SatBeamScheduler::DoSchedule() to determine how to allocate slots.**
