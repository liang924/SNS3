# Purpose of the `sat-vhts-example`

Sets up a **V/HTS (Very/High Throughput Satellite)** scenario with multiple spot-beams, gateways, and user terminals to evaluate **random access (RA) on the return link**, multi-beam loading, mobility, and super-frame configurations.  
It helps you study how **CRDSA/MARSALA**, **dynamic load control**, **burst length**, and **mobility** affect throughput, delay, and collision resolution in multi-beam systems.

---

# What does this example do (High-level)

- Builds a V/HTS topology with:
  - **N gateways (GWs)**, **selected beams**, and **UTs per beam**
  - Optional **UT mobility** loaded from a trajectory file
- Installs **end-user applications** behind each UT and starts them at a chosen time.
- Configures **Random Access (RA)** on the **return link**:
  - **CRDSA** (Contention Resolution Diversity Slotted ALOHA)
  - **MARSALA** (multi-replica interference cancellation / time-shift combining)
- Optionally enables **Dynamic Load Control** to adapt RA load.
- Chooses **burst length** (short/long) and **super-frame** settings.
- Runs for a given **simulation length**, and writes statistics to `--OutputPath`.

> Tip: Use multiple beams and UTs per beam to emulate realistic multi-beam congestion and test RA performance under load.

---

# Parameter Explanation (from `--PrintHelp`)

| Parameter | Type / Example | Function |
|-----------|----------------|----------|
| `--Beams` | String of IDs (e.g., `8` or `4_5_6`) | IDs of spot-beams to use. Separate multiple beams with underscores `_`. |
| `--NbGw` | Integer (e.g., `1`, `2`) | Number of gateways in the scenario. |
| `--NbUtsPerBeam` | Integer (e.g., `10`) | Number of UTs per selected spot-beam. |
| `--NbEndUsersPerUt` | Integer (e.g., `1`, `3`) | Number of end-user nodes behind each UT (e.g., CPEs). |
| `--AppStartTime` | Time (e.g., `+1ms`, `1s`) | When to start the traffic/applications. |
| `--SimLength` | Time (e.g., `+1min`, `120s`) | Total simulation time. |
| `--RaModel` | `CRDSA` or `MARSALA` | Random Access model for the return link. CRDSA uses multiple replicas and SIC; MARSALA extends SIC with time-shift/combining for heavy load. |
| `--DynamicLoadControl` | `true` / `false` | Enable closed-loop load control to adapt RA load and reduce collisions. |
| `--UtMobility` | `true` / `false` | Enable UT mobility (uses `--mobilityPath`). |
| `--mobilityPath` | Path | Folder containing UT trajectories (e.g., `contrib/satellite/data/utpositions/mobiles/scenario0/trajectory`). |
| `--BurstLength` | `ShortBurst` / `LongBurst` / `ShortAndLongBurst` | Select burst size used by RA. Long bursts improve robustness but consume more resources. |
| `--SuperFrameConfForSeq0` | Integer (e.g., `0`) | Super-frame configuration ID for sequence 0 (defines frame timing/structure). |
| `--FrameConfigType` | Integer (e.g., `0`) | Overall frame configuration type used by the super-frame. |
| `--OutputPath` | Directory path | Directory where simulation statistics/results are stored. |
| General (`--Print*`) | `--PrintHelp`, `--PrintGlobals`, `--PrintGroups`, `--PrintTypeIds`… | Inspect help and TypeIds; **does not** run the simulation. |

---

