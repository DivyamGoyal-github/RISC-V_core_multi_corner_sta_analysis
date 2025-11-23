# RISC-V_core_multi_corner_sta_analysis
---

### Week-3 (Post-Synthesis) vs Week-8 (Post-Route) Timing Comparison  
### Using OpenLANE, OpenROAD & OpenSTA

---

## 1. Overview

This repository documents the complete **Static Timing Analysis (STA)** of the **riscv_core** design across multiple **PVT corners** using:

- **OpenLANE** for synthesis, placement and routing  
- **OpenROAD** for loading post-route ODB and extracting SPEF  
- **OpenSTA** for multi-corner timing analysis  

The objective is to compare:

- **Week-3:** Post-Synthesis STA (ideal interconnect, no parasitics)  
- **Week-8:** Post-Route STA (real parasitics from SPEF, routed design)  

This report includes:  
✔ Timing tables  
✔ All PVT corner results  
✔ Graphs (WNS, TNS, Setup, Hold)  
✔ Detailed explanation of timing behavior after routing  

---

## 2. Environment & Inputs

### Software
- OpenLANE  
- OpenROAD  
- OpenSTA (v2.7.0)  
- Sky130 HD PDK (Liberty timing models)

### Required Files
- `riscv_core.odb` — Post-route OpenROAD database  
- `riscv_core.nom.spef` — Extracted parasitics  
- `riscv_base_post_cts.sdc` — Timing constraints  
- 16 × Liberty files across TT, FF, SS corners  

---

## 3. Post-Route STA TCL Script (Week-8)

This script loads the post-route design, SPEF parasitics, all 16 PVT Liberty files, runs setup/hold analysis for each corner, and writes WNS/TNS/Worst-Slack results.

```tcl
set list_of_lib_files(1)  "sky130_fd_sc_hd__tt_025C_1v80.lib"
set list_of_lib_files(2)  "sky130_fd_sc_hd__tt_100C_1v80.lib"
set list_of_lib_files(3)  "sky130_fd_sc_hd__ff_100C_1v65.lib"
set list_of_lib_files(4)  "sky130_fd_sc_hd__ff_100C_1v95.lib"
set list_of_lib_files(5)  "sky130_fd_sc_hd__ff_n40C_1v56.lib"
set list_of_lib_files(6)  "sky130_fd_sc_hd__ff_n40C_1v65.lib"
set list_of_lib_files(7)  "sky130_fd_sc_hd__ff_n40C_1v76.lib"
set list_of_lib_files(8)  "sky130_fd_sc_hd__ff_n40C_1v95.lib"
set list_of_lib_files(9)  "sky130_fd_sc_hd__ss_100C_1v40.lib"
set list_of_lib_files(10) "sky130_fd_sc_hd__ss_100C_1v60.lib"
set list_of_lib_files(11) "sky130_fd_sc_hd__ss_n40C_1v28.lib"
set list_of_lib_files(12) "sky130_fd_sc_hd__ss_n40C_1v35.lib"
set list_of_lib_files(13) "sky130_fd_sc_hd__ss_n40C_1v40.lib"
set list_of_lib_files(14) "sky130_fd_sc_hd__ss_n40C_1v44.lib"
set list_of_lib_files(15) "sky130_fd_sc_hd__ss_n40C_1v60.lib"
set list_of_lib_files(16) "sky130_fd_sc_hd__ss_n40C_1v76.lib"

set run_tag $::env(RUN_TAG)

set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um
set_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um

read_db /openlane/designs/riscv_core/runs/$run_tag/results/routing/riscv_core.odb
read_spef /openlane/designs/riscv_core/runs/$run_tag/results/final/spef/multicorner/riscv_core.nom.spef
current_design

if {![file exists ./sta_output]} {
    file mkdir ./sta_output
}

for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {

    read_liberty $::env(PDK_ROOT)/sky130A/libs.ref/sky130_fd_sc_hd/lib/$list_of_lib_files($i)
    read_sdc /openlane/designs/riscv_core/src/riscv_base_post_cts.sdc

    if {$i == 1} {
        exec echo -n > ./sta_output/sta_wns.txt
        exec echo -n > ./sta_output/sta_tns.txt
        exec echo -n > ./sta_output/sta_worst_max_slack.txt
        exec echo -n > ./sta_output/sta_worst_min_slack.txt
    }

    exec echo -ne "$list_of_lib_files($i)    " >> ./sta_output/sta_worst_max_slack.txt
    report_worst_slack -max >> ./sta_output/sta_worst_max_slack.txt

    exec echo -ne "$list_of_lib_files($i)    " >> ./sta_output/sta_worst_min_slack.txt
    report_worst_slack -min >> ./sta_output/sta_worst_min_slack.txt

    exec echo -ne "$list_of_lib_files($i)    " >> ./sta_output/sta_tns.txt
    report_tns >> ./sta_output/sta_tns.txt

    exec echo -ne "$list_of_lib_files($i)    " >> ./sta_output/sta_wns.txt
    report_wns >> ./sta_output/sta_wns.txt
}

exit
````

---

## 4. Week-3 Timing Results (Post-Synthesis)

### STA Analysis

<div align="center" >
  <img src="./assets/week_3/week3_sta_data.png" alt="week3_sta_data" width="80%">
</div>

### Worst Hold Slack

<div align="center" >
  <img src="./assets/week_3/WHS.jpg" alt="Worst_Hold_Stack" width="80%">
</div>

### Worst Setup Slack

<div align="center" >
  <img src="./assets/week_3/WSS.jpg" alt="Worst_Setup_Stack" width="80%">
</div>

### WNS

<div align="center" >
  <img src="./assets/week_3/WNS.jpg" alt="WNS" width="80%">
</div>

### TNS

<div align="center" >
  <img src="./assets/week_3/TNS.jpg" alt="TNS" width="80%">
</div>

---

## 5. Week-8 Timing Results (Post-Route)

### Week-8 Summary Table

![](/mnt/data/excel_data.png)

### Worst Setup Slack

![](/mnt/data/D19_worst_setup_slack.png)

### Worst Hold Slack

![](/mnt/data/D19_worst_hold_slack.png)

### WNS

![](/mnt/data/D19_wns.png)

### TNS

![](/mnt/data/D19_tns.png)

### STA Trends Across PVT

![](/mnt/data/D19_riscv_core_sta_across_pvt.png)

---

## 6. Week-3 vs Week-8 Comparison Table

| PVT Corner   | WNS (W3) | WNS (W8) | TNS (W3) | TNS (W8)   | WHS (W3) | WHS (W8) | Setup (W3) | Setup (W8) |
| ------------ | -------- | -------- | -------- | ---------- | -------- | -------- | ---------- | ---------- |
| tt_025C_1v80 | 0        | -7.4403  | 0        | -7477.7690 | 0.3096   | -2.8904  | 1.106      | -7.4403    |
| ff_100C_1v65 | 0        | -6.4158  | 0        | -6048.2212 | 0.2491   | -2.9509  | 2.4466     | -6.4158    |
| ff_100C_1v95 | 0        | -4.7488  | 0        | -4155.6294 | 0.196    | -3.0040  | 3.8366     | -4.7488    |
| ff_n40C_1v56 | 0        | -6.2602  | 0        | -6312.1372 | 0.2915   | -2.9085  | 1.127      | -6.2602    |
| ff_n40C_1v65 | 0        | -5.1682  | 0        | -5053.1134 | 0.2551   | -2.9449  | 2.1219     | -5.1682    |
| ff_n40C_1v76 | 0        | -4.2362  | 0        | -4004.4890 | 0.2243   | -2.9757  | 2.9919     | -4.2362    |
| ss_100C_1v40 | -13.0402 | -27.9272 | -7518    | -32115     | 0.9053   | -2.2947  | -13.0402   | -27.9272   |
| ss_100C_1v60 | -6.2777  | -18.2358 | -2911    | -20155     | 0.6420   | -2.5580  | -6.2777    | -18.2358   |
| ss_n40C_1v28 | -52.9031 | -51.4456 | -36775   | -72154     | 1.8296   | -1.8463  | -52.9031   | -51.4456   |
| ss_n40C_1v35 | -33.1984 | -37.8016 | -23279   | -51688     | 1.3475   | -1.9116  | -33.1984   | -37.8016   |
| ss_n40C_1v40 | -24.6564 | -31.4799 | -17173   | -42060     | 1.1249   | -2.0751  | -24.6564   | -31.4799   |
| ss_n40C_1v44 | -19.961  | -27.4636 | -13603   | -36005     | 0.9909   | -2.2091  | -19.961    | -27.4636   |
| ss_n40C_1v76 | -3.9606  | -12.3806 | -1905    | -14293     | 0.5038   | -2.6962  | -3.9606    | -12.3806   |

---

## 7. Interpretation of Results

### Post-Route Timing Is Worse

Because Week-8 includes **actual interconnect parasitics** from SPEF:

* Wire capacitance
* Wire resistance
* Via parasitics
* Coupling between adjacent nets

These increase the delay of long critical paths.

### Setup Slack Degradation

Worst corners (SS) show large setup violations:

* Slow silicon
* Low voltage
* High wire RC
* Long routes

### Hold Slack Degradation

Fast corners show worst hold slack:

* Fast silicon
* Higher supply voltage
* Shorter delays → races between FF stages

### SS_n40C_x corners dominate setup

As expected: slow + low voltage = maximum delay.

### FF_n40C_x corners dominate hold

Fast + low temperature = shortest delays.

---

## 8. Final Summary

* Week-3 timing is optimistic because it ignores routed RC parasitics.
* Week-8 timing is realistic and shows significant degradation in both setup and hold slack.
* SS corners show severe setup violations due to large wire delays.
* FF corners produce hold-time violations due to short propagation delays.
* Post-route timing highlights the need for buffering, cell upsizing, and net restructuring before sign-off.

---
