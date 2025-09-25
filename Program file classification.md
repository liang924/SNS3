# Table of Contents
- [Table](#table)
- [Detail content](#detail-content)
  - [Satellite-side](#satellite-side)
  - [Gateway-side](#gateway-side)
  - [User Terminal (UT)](#user-terminal-ut)
    - [ARQ](#arq-automatic-repeat-request)
    - [CBR / Traffic](#cbr--traffic)
    - [DAMA](#dama-demand-assigned-multiple-access)
    - [Random Access](#random-access)
    - [LoRa / IoT](#lora--iot)
    - [Logon / Mobility](#logon--mobility)
- [Generic / Utility](#generic--utility)

## Table
<h3>SNS3 Example Programs (by Execution Side)</h3>

<table>
  <tr>
    <th style="width: 20%;">Category</th>
    <th style="width: 40%;">Example</th>
    <th style="width: 40%;">Description</th>
  </tr>

  <!-- Satellite -->
  <tr><td rowspan="17">🛰️ Satellite-side</td><td>sat-beam-position-tracer</td><td>Trace satellite beam pointing positions</td></tr>
  <tr><td>sat-dynamic-frequency-plan-example</td><td>Demonstrates dynamic frequency planning</td></tr>
  <tr><td>sat-essa-example</td><td>Environmental Satellite Systems Analysis example</td></tr>
  <tr><td>sat-fwd-link-beam-hopping-example</td><td>Forward link beam hopping scenario</td></tr>
  <tr><td>sat-fwd-system-test-example</td><td>Full system test for forward link</td></tr>
  <tr><td>sat-mobility-beam-tracer</td><td>Visualize moving beams over mobility patterns</td></tr>
  <tr><td>sat-mobility-example</td><td>Simulates satellite mobility</td></tr>
  <tr><td>sat-mobility-position-generator</td><td>Generates user/satellite positions</td></tr>
  <tr><td>sat-rayleigh-example</td><td>Apply Rayleigh fading channel model</td></tr>
  <tr><td>sat-constellation-example</td><td>Multi-satellite constellation setup</td></tr>
  <tr><td>sat-regeneration-example</td><td>Regenerative satellite payload demo</td></tr>
  <tr><td>sat-regeneration-collisions-example</td><td>Payload with collision modeling</td></tr>
  <tr><td>sat-vhts-example</td><td>Very High Throughput Satellite case</td></tr>
  <tr><td>sat-trace-input-external-fading-example</td><td>Use external fading trace as input</td></tr>
  <tr><td>sat-trace-input-fading-example</td><td>Generic fading trace input</td></tr>
  <tr><td>sat-trace-input-interference-example</td><td>Trace-based interference modeling</td></tr>
  <tr><td>sat-trace-input-rx-power-example</td><td>Trace-based received power input</td></tr>

  <!-- Gateway -->
  <tr><td rowspan="5">🌐 Gateway-side</td><td>sat-gw-handover-example</td><td>Gateway handover scenario</td></tr>
  <tr><td>sat-handover-example</td><td>General handover process</td></tr>
  <tr><td>sat-rtn-system-test-example</td><td>Return link end-to-end test</td></tr>
  <tr><td>sat-profiling-sim</td><td>Profiling of system performance</td></tr>
  <tr><td>sat-profiling-sim-tn8</td><td>Profiling scenario with TN8 parameters</td></tr>

  <!-- UT ARQ -->
  <tr><td rowspan="2">📡 UT – ARQ</td><td>sat-arq-fwd-example</td><td>ARQ on forward link</td></tr>
  <tr><td>sat-arq-rtn-example</td><td>ARQ on return link</td></tr>

  <!-- UT CBR -->
  <tr><td rowspan="11">📡 UT – CBR/Traffic</td><td>sat-cbr-example</td><td>Simple constant bit rate traffic</td></tr>
  <tr><td>sat-cbr-full-example</td><td>Full-featured CBR setup</td></tr>
  <tr><td>sat-cbr-stats-example</td><td>Collect statistics for CBR flows</td></tr>
  <tr><td>sat-cbr-user-defined-example</td><td>Custom CBR traffic definition</td></tr>
  <tr><td>sat-onoff-example</td><td>On/Off traffic pattern</td></tr>
  <tr><td>sat-nrtv-example</td><td>Non-real-time video traffic</td></tr>
  <tr><td>sat-multi-application-fwd-example</td><td>Multiple applications on forward link</td></tr>
  <tr><td>sat-multi-application-rtn-example</td><td>Multiple applications on return link</td></tr>
  <tr><td>sat-multicast-example</td><td>Multicast service example</td></tr>
  <tr><td>sat-http-example</td><td>HTTP traffic over satellite</td></tr>
  <tr><td>sat-iot-example</td><td>IoT traffic scenario</td></tr>

  <!-- UT DAMA -->
  <tr><td rowspan="4">📡 UT – DAMA</td><td>sat-dama-http-sim-tn9</td><td>HTTP with DAMA access (TN9)</td></tr>
  <tr><td>sat-dama-onoff-sim-tn9</td><td>On/Off with DAMA (TN9)</td></tr>
  <tr><td>sat-dama-sim-tn9</td><td>Generic DAMA simulation (TN9)</td></tr>
  <tr><td>sat-dama-verification-sim</td><td>Verification of DAMA scheduler</td></tr>

  <!-- UT Random Access -->
  <tr><td rowspan="10">📡 UT – Random Access</td><td>sat-random-access-crdsa-collision-example</td><td>CRDSA with collisions</td></tr>
  <tr><td>sat-random-access-crdsa-example</td><td>CRDSA access scheme</td></tr>
  <tr><td>sat-random-access-dynamic-load-control-example</td><td>Load control for random access</td></tr>
  <tr><td>sat-random-access-example</td><td>Basic random access scenario</td></tr>
  <tr><td>sat-random-access-slotted-aloha-collision-example</td><td>Slotted ALOHA with collisions</td></tr>
  <tr><td>sat-random-access-slotted-aloha-example</td><td>Basic slotted ALOHA</td></tr>
  <tr><td>sat-ra-sim-tn9</td><td>Random access simulation (TN9)</td></tr>
  <tr><td>sat-ra-sim-tn9-comparison</td><td>Compare RA configurations (TN9)</td></tr>
  <tr><td>sat-rtn-link-da-example</td><td>Return link with demand assignment</td></tr>
  <tr><td>sat-rtn-link-ra-example</td><td>Return link with random access</td></tr>

  <!-- UT LoRa -->
  <tr><td rowspan="4">📡 UT – LoRa/IoT</td><td>sat-lora-constellation-example</td><td>LoRa with satellite constellation</td></tr>
  <tr><td>sat-lora-example</td><td>Basic LoRa system</td></tr>
  <tr><td>sat-lora-handover-example</td><td>LoRa mobility with handover</td></tr>
  <tr><td>sat-lora-regenerative-example</td><td>LoRa with regenerative payload</td></tr>

  <!-- UT Logon -->
  <tr><td rowspan="2">📡 UT – Logon/Mobility</td><td>sat-log-example</td><td>General log tracing</td></tr>
  <tr><td>sat-logon-example</td><td>User terminal logon procedure</td></tr>

  <!-- Generic -->
  <tr><td rowspan="12">🛠️ Generic/Utility</td><td>sat-generic-launcher</td><td>General launcher for scenarios</td></tr>
  <tr><td>sat-group-example</td><td>Group communications</td></tr>
  <tr><td>sat-link-budget-example</td><td>Link budget calculation</td></tr>
  <tr><td>sat-link-results-plot</td><td>Plot link performance results</td></tr>
  <tr><td>sat-list-position-ext-fading-example</td><td>Position list with external fading</td></tr>
  <tr><td>sat-loo-example</td><td>Loo fading channel model</td></tr>
  <tr><td>sat-markov-fading-trace-example</td><td>Markov chain fading trace</td></tr>
  <tr><td>sat-markov-logic-example</td><td>Logic-based Markov process</td></tr>
  <tr><td>sat-per-packet-if-sim-tn9</td><td>Per-packet interface simulation (TN9)</td></tr>
  <tr><td>sat-trace-output-example</td><td>Example of trace file output</td></tr>
  <tr><td>sat-training-example</td><td>Training/demo scenario</td></tr>
  <tr><td>sat-tutorial-example</td><td>Tutorial for new users</td></tr>
</table>

## Detail content

### Satellite-side

* **sat-beam-position-tracer**: Trace satellite beam pointing positions.  
* **sat-dynamic-frequency-plan-example**: Demonstrates dynamic frequency planning.  
* **sat-essa-example**: Environmental Satellite Systems Analysis.  
* **sat-fwd-link-beam-hopping-example**: Forward link beam hopping scenario.  
* **sat-fwd-system-test-example**: Full forward link system test.  
* **sat-mobility-beam-tracer**: Visualize beam coverage under mobility.  
* **sat-mobility-example**: Simulate satellite or user mobility.  
* **sat-mobility-position-generator**: Generate positions/trajectories for users or satellites.  
* **sat-rayleigh-example**: Apply Rayleigh fading channel model.  
* **sat-constellation-example**: Multi-satellite constellation setup.  
* **sat-regeneration-example**: Regenerative satellite payload demonstration.  
* **sat-regeneration-collisions-example**: Regenerative payload with collision modeling.  
* **sat-vhts-example**: Very High Throughput Satellite case study.  
* **sat-trace-input-external-fading-example**: Use external fading trace as input.  
* **sat-trace-input-fading-example**: General fading trace input.  
* **sat-trace-input-interference-example**: Trace-based interference modeling.  
* **sat-trace-input-rx-power-example**: Trace-based received power input.  

---

### Gateway-side

* **sat-gw-handover-example**: Gateway handover scenario.  
* **sat-handover-example**: General handover process.  
* **sat-rtn-system-test-example**: Return link end-to-end test.  
* **sat-profiling-sim**: Profiling system performance.  
* **sat-profiling-sim-tn8**: Profiling with TN8 configuration.  

---

### User Terminal (UT)

#### ARQ (Automatic Repeat reQuest)
* **sat-arq-fwd-example**: ARQ on forward link.  
* **sat-arq-rtn-example**: ARQ on return link.  

#### CBR / Traffic
* **sat-cbr-example**: Simple constant bit rate traffic.  
* **sat-cbr-full-example**: Full CBR traffic setup.  
* **sat-cbr-stats-example**: Collect statistics for CBR flows.  
* **sat-cbr-user-defined-example**: User-defined CBR traffic.  
* **sat-onoff-example**: On/Off traffic pattern.  
* **sat-nrtv-example**: Non-real-time video traffic.  
* **sat-multi-application-fwd-example**: Multiple applications on forward link.  
* **sat-multi-application-rtn-example**: Multiple applications on return link.  
* **sat-multicast-example**: Multicast service example.  
* **sat-http-example**: HTTP traffic simulation.  
* **sat-iot-example**: IoT traffic scenario.  

#### DAMA (Demand Assigned Multiple Access)
* **sat-dama-http-sim-tn9**: HTTP with DAMA access (TN9).  
* **sat-dama-onoff-sim-tn9**: On/Off traffic with DAMA (TN9).  
* **sat-dama-sim-tn9**: Generic DAMA simulation (TN9).  
* **sat-dama-verification-sim**: Verification of DAMA scheduler.  

#### Random Access
* **sat-random-access-crdsa-collision-example**: CRDSA with collisions.  
* **sat-random-access-crdsa-example**: CRDSA access scheme.  
* **sat-random-access-dynamic-load-control-example**: Random access with load control.  
* **sat-random-access-example**: Basic random access example.  
* **sat-random-access-slotted-aloha-collision-example**: Slotted ALOHA with collisions.  
* **sat-random-access-slotted-aloha-example**: Basic slotted ALOHA.  
* **sat-ra-sim-tn9**: Random access simulation (TN9).  
* **sat-ra-sim-tn9-comparison**: Compare RA configurations (TN9).  
* **sat-rtn-link-da-example**: Return link with demand assignment.  
* **sat-rtn-link-ra-example**: Return link with random access.  

#### LoRa / IoT
* **sat-lora-constellation-example**: LoRa with satellite constellation.  
* **sat-lora-example**: Basic LoRa system.  
* **sat-lora-handover-example**: LoRa mobility with handover.  
* **sat-lora-regenerative-example**: LoRa with regenerative payload.  

#### Logon / Mobility
* **sat-log-example**: General log tracing.  
* **sat-logon-example**: User terminal logon procedure.  

---

### Generic / Utility

* **sat-generic-launcher**: General launcher for scenarios.  
* **sat-group-example**: Group communications.  
* **sat-link-budget-example**: Link budget calculation.  
* **sat-link-results-plot**: Plot link performance results.  
* **sat-list-position-ext-fading-example**: Position list with external fading.  
* **sat-loo-example**: Loo fading channel model.  
* **sat-markov-fading-trace-example**: Markov chain fading trace.  
* **sat-markov-logic-example**: Logic-based Markov process.  
* **sat-per-packet-if-sim-tn9**: Per-packet interface simulation (TN9).  
* **sat-trace-output-example**: Trace output example.  
* **sat-training-example**: Training/demo scenario.  
* **sat-tutorial-example**: Tutorial example for beginners.  
