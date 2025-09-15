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
