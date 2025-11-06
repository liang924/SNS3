<img width="339" height="685" alt="image" src="https://github.com/user-attachments/assets/cbc67494-6d55-4a52-a037-713a388f4c4f" />

1. Initialization
```
SatFwdLinkScheduler::GetTypeId(void)
{
    static TypeId tid =
        TypeId("ns3::SatFwdLinkScheduler")
            .SetParent<Object>()
            .AddConstructor<SatFwdLinkScheduler>()
            .AddAttribute("Interval",
                          "The time for periodic scheduling",
                          TimeValue(MilliSeconds(20)),
                          MakeTimeAccessor(&SatFwdLinkScheduler::m_periodicInterval),
                          MakeTimeChecker())
            .AddAttribute("BBFrameConf",
                          "BB Frame configuration for this scheduler.",
                          PointerValue(),
                          MakePointerAccessor(&SatFwdLinkScheduler::m_bbFrameConf),
                          MakePointerChecker<SatBbFrameConf>())
            .AddAttribute("DummyFrameSendingEnabled",
                          "Flag to tell, if dummy frames are sent or not.",
                          BooleanValue(false),
                          MakeBooleanAccessor(&SatFwdLinkScheduler::m_dummyFrameSendingEnabled),
                          MakeBooleanChecker())
            .AddAttribute("AdditionalSortCriteria",
                          "Sorting criteria after priority for scheduling objects from LLC.",
                          EnumValue(SatFwdLinkScheduler::NO_SORT),
                          MakeEnumAccessor<SatFwdLinkScheduler::ScheduleSortingCriteria_t>(
                              &SatFwdLinkScheduler::m_additionalSortCriteria),
                          MakeEnumChecker(SatFwdLinkScheduler::NO_SORT,
                                          "NoSorting",
                                          SatFwdLinkScheduler::BUFFERING_DELAY_SORT,
                                          "DelaySort",
                                          SatFwdLinkScheduler::BUFFERING_LOAD_SORT,
                                          "LoadSort"))
            .AddAttribute("CnoEstimationMode",
                          "Mode of the C/N0 estimator",
                          EnumValue(SatCnoEstimator::LAST),
                          MakeEnumAccessor<SatCnoEstimator::EstimationMode_t>(
                              &SatFwdLinkScheduler::m_cnoEstimatorMode),
                          MakeEnumChecker(SatCnoEstimator::LAST,
                                          "LastValueInWindow",
                                          SatCnoEstimator::MINIMUM,
                                          "MinValueInWindow",
                                          SatCnoEstimator::AVERAGE,
                                          "AverageValueInWindow"))
            .AddAttribute("CnoEstimationWindow",
                          "Time window for C/N0 estimation.",
                          TimeValue(Seconds(5000)),
                          MakeTimeAccessor(&SatFwdLinkScheduler::m_cnoEstimationWindow),
                          MakeTimeChecker())
            .AddTraceSource(
                "SymbolRate",
                "Scheduler symbol rate for a given packet",
                MakeTraceSourceAccessor(&SatFwdLinkScheduler::m_schedulingSymbolRateTrace),
                "ns3::SatTypedefs::FwdLinkSchedulerSymbolRateCallback")

        ;
    return tid;
}
```
Register Attributes (Cycle Time, Frame Settings, DummyFrame flag, Sorting Method, C/N0 Mode)

2. Set Callbacks
```
SatFwdLinkScheduler::SetSchedContextCallback(SatFwdLinkScheduler::SchedContextCallback cb)
{
    NS_LOG_FUNCTION(this << &cb);
    m_schedContextCallback = cb;
}

void
SatFwdLinkScheduler::SetTxOpportunityCallback(SatFwdLinkScheduler::TxOpportunityCallback cb)
{
    NS_LOG_FUNCTION(this << &cb);
    m_txOpportunityCallback = cb;
}

void
SatFwdLinkScheduler::SetSendControlMsgCallback(SatFwdLinkScheduler::SendControlMsgCallback cb)
{
    NS_LOG_FUNCTION(this << &cb);
    m_sendControlMsgCallback = cb;
}
```
Set up three callback functions to facilitate communication between the scheduler and the external MAC/LLC.

3. Scheduling Interfaces (abstract)
```
std::pair<Ptr<SatBbFrame>, const Time>
SatFwdLinkScheduler::GetNextFrame()
{
    NS_FATAL_ERROR("SatFwdLinkScheduler::GetNextFrame: should not be here");
    Ptr<SatBbFrame> f = nullptr;
    return std::make_pair(f, m_bbFrameConf->GetDummyBbFrameDuration());
}

void
SatFwdLinkScheduler::ClearAllPackets()
{
    NS_FATAL_ERROR("Must use subclasses");
}

void
SatFwdLinkScheduler::PeriodicTimerExpired()
{
    NS_FATAL_ERROR("SatFwdLinkScheduler::ScheduleBbFrames: should not be here");
}

void
SatFwdLinkScheduler::SendAndClearSymbolsSentStat()
{
    NS_FATAL_ERROR("SatFwdLinkScheduler::SendAndClearSymbolsSentStat: should not be here");
}

void
SatFwdLinkScheduler::ScheduleBbFrames()
{
    NS_FATAL_ERROR("SatFwdLinkScheduler::ScheduleBbFrames: should not be here");
}

void
SatFwdLinkScheduler::GetSchedulingObjects(std::vector<Ptr<SatSchedulingObject>>& output)
{
    NS_FATAL_ERROR("SatFwdLinkScheduler::GetSchedulingObjects: should not be here");
}
```
These are all abstract interfaces, which cannot be used in the base class and will throw a Fatal Error; they must be implemented by subclasses (Default / TimeSlicing).

4. Sorting Logic
```
bool
SatFwdLinkScheduler::CompareSoFlowId(Ptr<SatSchedulingObject> obj1, Ptr<SatSchedulingObject> obj2)
{
    return (bool)(obj1->GetFlowId() < obj2->GetFlowId());
}

bool
SatFwdLinkScheduler::CompareSoPriorityLoad(Ptr<SatSchedulingObject> obj1,
                                           Ptr<SatSchedulingObject> obj2)
{
    bool result = CompareSoFlowId(obj1, obj2);

    if (obj1->GetFlowId() == obj2->GetFlowId())
    {
        result = (bool)(obj1->GetBufferedBytes() > obj2->GetBufferedBytes());
    }

    return result;
}

bool
SatFwdLinkScheduler::CompareSoPriorityHol(Ptr<SatSchedulingObject> obj1,
                                          Ptr<SatSchedulingObject> obj2)
{
    bool result = CompareSoFlowId(obj1, obj2);

    if (obj1->GetFlowId() == obj2->GetFlowId())
    {
        result = (bool)(obj1->GetHolDelay() > obj2->GetHolDelay());
    }

    return result;
}

void
SatFwdLinkScheduler::SortSchedulingObjects(std::vector<Ptr<SatSchedulingObject>>& so)
{
    NS_LOG_FUNCTION(this);

    // sort only if there is need to sort
    if ((so.empty() == false) && (so.size() > 1))
    {
#ifdef SAT_FWD_LINK_SCHEDULER_PRINT_SORT_RESULT
        PrintSoContent("Before sort", so);
#endif

        switch (m_additionalSortCriteria)
        {
        case SatFwdLinkScheduler::NO_SORT:
            std::sort(so.begin(), so.end(), CompareSoFlowId);
            break;

        case SatFwdLinkScheduler::BUFFERING_DELAY_SORT:
            std::sort(so.begin(), so.end(), CompareSoPriorityHol);
            break;

        case SatFwdLinkScheduler::BUFFERING_LOAD_SORT:
            std::sort(so.begin(), so.end(), CompareSoPriorityLoad);
            break;

        default:
            NS_FATAL_ERROR("Not supported sorting criteria!!!");
            break;
        }

#ifdef SAT_FWD_LINK_SCHEDULER_PRINT_SORT_RESULT
        PrintSoContent("After sort", so);
#endif
    }
}
```

Provide three sorting methods:
- FlowId
- Load (Buffer Size)
- HolDelay (Head of Line Delay)

5. C/N0 Estimation
```
SatFwdLinkScheduler::CnoInfoUpdated(Mac48Address utAddress, double cnoEstimate)
{
    NS_LOG_FUNCTION(this << utAddress << cnoEstimate);

    CnoEstimatorMap_t::const_iterator it = m_cnoEstimatorContainer.find(utAddress);

    if (it == m_cnoEstimatorContainer.end())
    {
        Ptr<SatCnoEstimator> estimator = CreateCnoEstimator();

        std::pair<CnoEstimatorMap_t::const_iterator, bool> result =
            m_cnoEstimatorContainer.insert(std::make_pair(utAddress, estimator));
        it = result.first;

        if (result.second == false)
        {
            NS_FATAL_ERROR("Estimator cannot be added to container!!!");
        }
    }

    it->second->AddSample(cnoEstimate);
}

SatFwdLinkScheduler::CnoMatchWithFrame(double cno, Ptr<SatBbFrame> frame) const
{
    NS_LOG_FUNCTION(this << cno << frame);

    bool match = false;

    SatEnums::SatModcod_t modCod = m_bbFrameConf->GetBestModcod(cno, frame->GetFrameType());

    if (modCod >= frame->GetModcod())
    {
        match = true;
    }

    return match;
}

double
SatFwdLinkScheduler::GetSchedulingObjectCno(Ptr<SatSchedulingObject> ob)
{
    NS_LOG_FUNCTION(this << ob);

    double cno = NAN;

    CnoEstimatorMap_t::const_iterator it = m_cnoEstimatorContainer.find(ob->GetMacAddress());

    if (it != m_cnoEstimatorContainer.end())
    {
        cno = it->second->GetCnoEstimation();
    }

    return cno;
}

Ptr<SatCnoEstimator>
SatFwdLinkScheduler::CreateCnoEstimator()
{
    NS_LOG_FUNCTION(this);

    Ptr<SatCnoEstimator> estimator = nullptr;

    switch (m_cnoEstimatorMode)
    {
    case SatCnoEstimator::LAST:
    case SatCnoEstimator::MINIMUM:
    case SatCnoEstimator::AVERAGE:
        estimator = Create<SatBasicCnoEstimator>(m_cnoEstimatorMode, m_cnoEstimationWindow);
        break;

    default:
        NS_FATAL_ERROR("Not supported C/N0 estimation mode!!!");
        break;
    }

    return estimator;
}
```
Functions:
- Update C/N0 (CnoInfoUpdated)
- Check whether a frame matches the C/N0 (CnoMatchWithFrame)
- Get the C/N0 estimation of a UT (GetSchedulingObjectCno)
- Create a C/N0 estimator (CreateCnoEstimator)


## Note
# 🛰️ Forward Link Scheduling — Key Summary

## 🔹 1. Function and Role
The **Forward Link Scheduler** in SNS-3 controls **data transmission from the Gateway to User Terminals (UTs)** using **BBFrames (Base-Band Frames)**.

It determines:
- How packets are packed into BBFrames  
- Which **Modulation and Coding (MODCOD)** scheme to use  
- When to transmit frames according to traffic demand and link quality  

> *(The return link’s capacity control is covered separately under “Resource Management.”)*

---

## 🔹 2. System Architecture and Core Classes

| Class | Description |
|--------|-------------|
| `SatFwdLinkScheduler` | Abstract base class defining the common scheduler interface |
| `SatFwdLinkSchedulerDefault` | Default scheduler with a single BBFrame container |
| `SatFwdLinkSchedulerTimeSlicing` | Advanced scheduler supporting time slicing |
| `SatBbFrameConf` | Defines BBFrame configuration (MODCODs, frame types) |
| `SatBbFrameContainer` | Holds BBFrames ready for transmission |
| `SatDvbS2Waveform` | Represents a DVB-S2/S2X waveform (MODCOD, frame type, duration) |

---

## 🔹 3. Default Scheduler

### ⚙️ Operation
1. The **periodic timer** expires → triggers `PeriodicTimerExpired()`.  
2. Calls `ScheduleBbFrames()` to prepare new frames.  
3. Requests scheduling objects from the **LLC layer**.  
4. For each object:
   - Selects the proper **MODCOD**.
   - Checks for available space in the current BBFrame.
   - Requests packets and appends them into the BBFrame.
5. When the Gateway MAC requests a frame (`GetNextFrame()`), it returns the next ready BBFrame from the container.

### 📊 Configuration Parameters

| Parameter | Description | Default |
|------------|-------------|----------|
| `SchedulingStartThresholdTime` | Time threshold to start scheduling | 5 ms |
| `SchedulingStopThresholdTime` | Time threshold to stop scheduling | 15 ms |
| `PeriodicInterval` | Periodic scheduling interval | Custom |
| `BBFrameContainer` | Frame container used by scheduler | — |

> ✅ Best for **basic or single-user simulations** where time slicing is not needed.

---

## 🔹 4. Time Slicing Scheduler

### 🧩 Concept
- Divides total **carrier bandwidth** into multiple **time slices**.  
- Each slice owns a portion of the total **symbol rate**.  
- UTs are assigned to specific slices → prevents interference and ensures fairness.  

| Slice | Purpose |
|--------|----------|
| Slice 0 | Control and broadcast (shared by all UTs) |
| Slice 1–N | User data transmission (each UT mapped to one slice) |

### ⚙️ Implementation Details
- When a new UT joins, it is assigned to a slice via **Round-Robin** allocation.  
- The scheduler tracks the number of symbols per slice to **maintain the symbol rate limit**.  
- If a slice exceeds its allocation, new BBFrames are not opened.

> **Formula:**  
> `Slice Symbol Rate = Total Carrier Bandwidth / Number of Slices`

### 📊 Configuration Parameters

| Parameter | Description | Default |
|------------|-------------|----------|
| `NumberOfSlices` | Number of slices | 1 |
| `PeriodicInterval` | Scheduling interval | Custom |

> ✅ Used for **multi-user or multi-service simulations** to allocate bandwidth fairly.

---

## 🔹 5. BBFrame Configuration

### 📦 Description
A **BBFrame** is the fundamental transmission unit in DVB-S2/DVB-S2X.  
The `SatBbFrameConf` class defines its properties:
- MODCOD scheme  
- Frame type (NORMAL / SHORT / DUMMY)  
- Duration and modulation settings  

### 💡 Supported MODCOD Examples

| Modulation | Coding Rate | DVB Version |
|-------------|--------------|--------------|
| QPSK 1/2 | DVB-S2 |
| QPSK 3/5 | DVB-S2 |
| 8PSK 3/5 | DVB-S2 |
| 16APSK 2/3 | DVB-S2 |
| 32APSK 3/4 | DVB-S2 |
| … | … | … |

### 🧩 Frame Types

| Type | Description |
|-------|--------------|
| **NORMAL_FRAME** | Standard frame, larger payload |
| **SHORT_FRAME** | Shorter frame, smaller payload |
| **DUMMY_FRAME** | Empty frame used when no data is available |

### 📘 DVB Version
Supports both **DVB-S2** and **DVB-S2X**, selectable via:
```cpp
typedef enum { DVB_S2, DVB_S2X } DvbVersion_t;
