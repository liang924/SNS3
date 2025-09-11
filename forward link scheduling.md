<img width="626" height="760" alt="image" src="https://github.com/user-attachments/assets/b9461e94-3860-4ad8-a1ea-ddc3f739505f" />

# Forward Link Scheduler – Detailed Flow

## 1) Initialization → Idle
- Create the **Forward Link Scheduler** with attributes:  
  scheduling interval, dummy frame option, C/N₀ estimation mode, etc.  
- Bind callbacks (requesting packets from LLC, requesting scheduling objects, sending control messages).  
- Create **BBFrame Container** and **C/N₀ Estimator** container.  
- Start the **periodic timer**.  
- After initialization, enter **Idle**: waiting for either “timer expiry” or “MAC request for next frame”.
```
SatFwdLinkScheduler::SatFwdLinkScheduler(Ptr<SatBbFrameConf> conf,
                                         Mac48Address address,
                                         double carrierBandwidthInHz)
    : m_macAddress(address),
      m_bbFrameConf(conf),
      m_additionalSortCriteria(SatFwdLinkScheduler::NO_SORT),
      m_cnoEstimatorMode(SatCnoEstimator::LAST),
      m_carrierBandwidthInHz(carrierBandwidthInHz)
{
    NS_LOG_FUNCTION(this);

    ObjectBase::ConstructSelf(AttributeConstructionList());

    // Random variable used in scheduling
    m_random = CreateObject<UniformRandomVariable>();
}
```
---

## 2) Path A: Timer Expiry → Scheduling Round
This path is **proactive frame creation (filling the inventory)**.

1. **SchedulingRound**  
   - Periodic timer expires.  
   - Scheduler starts one scheduling round to “refill” the BBFrame Container.

2. **Call ScheduleBbFrames()**  
   - Enter the main loop of frame creation (stop when we reach the threshold).

3. **RequestObjects → Get scheduling objects from LLC**  
   - Callback to LLC to get a batch of **Scheduling Objects** (flow summaries).  
   - Each object includes:  
     - buffer size  
     - minimum transmission unit (**minTx**)  
     - flowId  
     - UT’s MAC  
     - LLC’s **HOL delay**, etc.  
   - Scheduler may sort them (by flowId, load, HOL, etc.).

4. **ProcessObjects (loop over each object)**  
   - For every scheduling object:  
     - **SelectModcod / Determine MODCOD based on C/N₀**  
       Use C/N₀ estimator, select the best ModCod so the frame can be reliably decoded.  
     - **CheckSpace**  
       Check if current tail frame has enough space ≥ minTx.  
       - If not: **Open new frame** (with selected ModCod, deduct header overhead).  
     - **RequestPacket (LLC callback)**  
       Request packets from LLC according to available space/minTx.  
       - Success → **AddToFrame** (insert packet into frame, update counters).  
       - Fail:  
         - If current frame is non-empty → **switch/open new frame**.  
         - If current frame is empty → **fatal error** (cannot even fit minTx).  
     - **Stop condition**  
       Continue while: buffer > 0 **and** container total duration < StopThreshold.  
       When reaching StopThreshold → stop this round.

5. **MergeFrames**  
   - Merge fragmented tail frames to align with BBFrame configuration.

6. **Return to Idle**  
   - If more objects remain and below StopThreshold → continue.  
   - Otherwise → finish the round and return Idle, wait for next timer.

---

## 3) Path B: MAC Requests Frame (On-demand)
This path is **MAC consumes inventory (on-demand delivery)**.

1. **ProvideFrame → Call GetNextFrame()**  
   - When GW MAC needs a frame, it calls `GetNextFrame()`.

2. **Get frame from container**  
   - If container below **StartThreshold** → quickly run `ScheduleBbFrames()` to refill.  
   - Fetch one frame from container.  
   - If none available:  
     - If dummy allowed → create **Dummy Frame**.  
     - Else → return empty duration (no data to transmit).  

3. **ReturnFrame → Return Idle**  
   - Give frame to MAC.  
   - Scheduler returns to Idle until next event (timer expiry or another MAC request).

---

## 4) Data & Control Exchanges
- **LLC ↔ Scheduler**  
  - Scheduler requests scheduling objects & packets.  
  - LLC provides buffer/minTx/HOL info and delivers packets.  

- **Scheduler ↔ BBFrame Container**  
  - Scheduler creates frames with ModCod, inserts them into container.  
  - Container stores frames, tracks duration, provides next frame.  

- **Scheduler ↔ MAC/PHY**  
  - MAC requests frames → Scheduler supplies them.  
  - Scheduler may also send control messages (e.g., slot/coding info).  

---

## 5) When to Stop / Resume
- **Scheduling round stops if:**  
  - Container total duration ≥ StopThreshold, OR  
  - No scheduling objects with data available.  

- **Scheduling resumes when:**  
  - **Periodic timer** expires, OR  
  - **MAC requests frame** but container < StartThreshold (trigger immediate refill).  

---

## ✅ Summary
- **Forward Link Scheduler** = decides *what data goes into downlink frames*.  
- Two main triggers: **Timer expiry (proactive)** and **MAC request (on-demand)**.  
- Ensures UTs receive continuous BBFrames with adaptive ModCod, while controlling buffer thresholds for efficiency.
