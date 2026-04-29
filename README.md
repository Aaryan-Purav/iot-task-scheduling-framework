# IoT Task Scheduling Framework with Energy Awareness

A simulation-based comparative study of 11 CPU scheduling algorithms on a mixed-criticality IoT workload, with a custom energy model and a novel Hybrid Adaptive Scheduler proposed as the main contribution.

Built as part of the Operating Systems and Computer Networks (OSCN) course at Vidyalankar Institute of Technology, Mumbai.

---

## Authors

- **Aaryan Purav** — [aaryan.purav@vit.edu.in](mailto:aaryan.purav@vit.edu.in)
- **Shailesh Pawar** — [shailesh.pawar@vit.edu.in](mailto:shailesh.pawar@vit.edu.in)

**Guide:** Prof. Swapnil Ashtekar, Department of Electronics and Telecommunication, VIT Mumbai

---

## What This Project Does

IoT processors handle heterogeneous workloads simultaneously — health alerts with hard deadlines, interactive motion sensors, heavy camera processing, and background loggers. Classical schedulers were not designed for this mixed-criticality context.

This project benchmarks 11 scheduling algorithms on a 100-unit IoT simulation with a 3× burst window (t = 30 to t = 65), measuring performance, energy, deadline compliance, and queue stability for each. It also proposes a novel **Hybrid Adaptive Scheduler** that dynamically switches scheduling policy based on task category and burst state.

---

## Algorithms Implemented

**Classical:**
| Algorithm | Type | Key Property |
|---|---|---|
| FCFS | Non-preemptive | Simple FIFO, convoy effect |
| SJF | Non-preemptive | Minimizes avg waiting time |
| SRTF | Preemptive | Min avg WT, highest context switches |
| Round Robin (q=4) | Preemptive | Bounded response time |

**Priority-Based:**
| Algorithm | Type | Key Property |
|---|---|---|
| Priority (NP) | Non-preemptive | Static priority, no preemption |
| Priority (P) | Preemptive | Immediate preemption on higher priority |
| Multilevel Queue | Semi-preemptive | Fixed per-category queues |
| MLFQ | Preemptive | Queue demotion, best response time |

**Real-Time:**
| Algorithm | Type | Key Property |
|---|---|---|
| EDF | Dynamic priority | Optimal when U ≤ 1 |
| RMS | Fixed priority | Optimal fixed-priority for periodic tasks |

**Novel Contribution:**
| Algorithm | Type | Key Property |
|---|---|---|
| Hybrid Adaptive | Context-aware | EDF + RR + SRTF with burst detection |

---

## Workload Model

8 sensor types are modeled across 4 task categories:

| Sensor | Category | Burst (units) | Period | Deadline |
|---|---|---|---|---|
| Health Alert | CRITICAL | 2–4 | 10 | 8 |
| Security Alarm | CRITICAL | 3–5 | 12 | 10 |
| Motion Detect | INTERACTIVE | 2–6 | 5 | 8 |
| Touch Input | INTERACTIVE | 1–3 | 3 | 5 |
| Camera Process | BATCH | 10–18 | 20 | 30 |
| Audio Analysis | BATCH | 8–15 | 15 | 25 |
| Temp Logger | BACKGROUND | 2–4 | 30 | 50 |
| Status Update | BACKGROUND | 1–3 | 25 | 40 |

Task arrival rate: 2/unit (normal), 6/unit (burst window t = 30–65).

---

## Energy Model

```
E(t) = P_active × α_category + N_context_switches × E_cs
```

| Parameter | Value |
|---|---|
| Base active power | 200 mW |
| Idle power | 50 mW |
| Context switch energy | 2.5 mJ per switch |
| CRITICAL multiplier (α) | 1.4 |
| INTERACTIVE multiplier (α) | 1.0 |
| BATCH multiplier (α) | 0.9 |
| BACKGROUND multiplier (α) | 0.8 |

---

## Novel Contribution: Hybrid Adaptive Scheduler

The Hybrid Adaptive Scheduler maintains three separate sub-queues and dynamically selects the scheduling policy at each time unit:

```
if CRITICAL queue not empty  →  EDF (run to completion)
else if INTERACTIVE queue not empty  →  Round Robin (q = 4)
else if BATCH/BACKGROUND queue not empty:
    if burst detected (> 3× window threshold)  →  Extended quantum (q = 8)
    else  →  SRTF
```

Additionally, BATCH tasks execute with 85% power scaling during low-load periods, reducing energy consumption further.

**Results vs best classical algorithm (SRTF):**
- 12% lower average waiting time (618.7 vs 543.6 — with 10× fewer context switches)
- 10.6% lower total energy (362,803 mJ vs ~400,600 mJ average)
- Only 372 context switches vs 3,808 for SRTF

---

## Results Summary

| Algorithm | Avg WT | Avg RT | Energy (mJ) | Miss Rate | Queue Var |
|---|---|---|---|---|---|
| FCFS | 951.9 | 951.9 | 400,603 | 98.8% | 8535.8 |
| SJF | 545.7 | 545.7 | 400,603 | 86.2% | 5395.0 |
| SRTF | 543.6 | 541.6 | 405,828 | 85.9% | 5327.7 |
| Round Robin | 939.0 | 504.6 | 400,603 | 99.1% | 6558.7 |
| Priority (NP) | 924.9 | 924.9 | 400,603 | 97.3% | 4735.2 |
| Priority (P) | 935.7 | 896.9 | 405,828 | 97.1% | 4754.8 |
| MLQ | 1053.1 | 833.9 | 400,603 | 97.7% | 4402.9 |
| MLFQ | 1008.2 | 278.0 | 400,613 | 99.4% | 8894.9 |
| EDF | 918.1 | 911.1 | 405,828 | 98.8% | 7623.4 |
| RMS | 899.6 | 866.1 | 405,828 | 95.0% | 4309.3 |
| **Hybrid Adaptive** | **618.7** | **612.6** | **362,803** | 97.7% | 8334.7 |

Key finding: All algorithms exceed 85% deadline miss rate under full CPU utilization, confirming that admission control is required for hard real-time guarantees — the scheduler alone cannot fix an overloaded system.

---

## Project Structure

```
Course Project/
│
├── AdvancedIoTScheduler.java     # Main simulation — all 11 algorithms
├── AdvancedIoTScheduler.class    # Compiled bytecode
├── scheduling_results.csv        # Raw output — all metrics per algorithm
├── results_table.tex             # LaTeX table of results
├── visualize_results.py          # Python visualization script (matplotlib)
│
├── fig1_performance_comparison.png
├── fig2_energy_analysis.png
├── fig3_performance_energy_tradeoff.png
├── fig4_realtime_analysis.png
├── fig5_queue_stability.png
└── fig6_radar_comparison.png
```

---

## How to Run

**Requirements:** Java 17 or higher (uses switch expressions)

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/iot-task-scheduling-framework.git
cd iot-task-scheduling-framework/Course\ Project/

# Compile
javac AdvancedIoTScheduler.java

# Run
java AdvancedIoTScheduler
```

Output printed to terminal includes four analysis sections:
- Performance comparison (all 11 algorithms)
- Energy consumption analysis
- Dynamic load and stability analysis
- Real-time deadline analysis

Results are also exported to `scheduling_results.csv`.

**To regenerate figures** (requires Python):

```bash
pip install matplotlib pandas numpy
python visualize_results.py
```

---

## Key Design Insights

1. **Preemption costs energy.** SRTF and EDF produce 3,800+ context switches, adding ~9.5 J in switching overhead — significant on battery-powered IoT hardware.

2. **MLFQ is best for responsiveness.** Average response time of 278 units — 3× better than any other algorithm — by rapidly completing short interactive tasks.

3. **No single policy wins on all metrics.** This is the core motivation for context-aware hybrid scheduling.

4. **Admission control is non-negotiable for hard real-time.** At CPU utilization = 100%, every algorithm fails to meet deadlines. The scheduler cannot fix an overloaded system.

---

## References

1. A. Silberschatz, P. B. Galvin, and G. Gagne, *Operating System Concepts*, 10th ed. Wiley, 2018.
2. C. L. Liu and J. W. Layland, "Scheduling algorithms for multiprogramming in a hard-real-time environment," *J. ACM*, vol. 20, no. 1, pp. 46–61, 1973.
3. G. C. Buttazzo, *Hard Real-Time Computing Systems*, 3rd ed. Springer, 2011.
4. L. Atzori, A. Iera, and G. Morabito, "The Internet of Things: A survey," *Comput. Netw.*, vol. 54, no. 15, pp. 2787–2805, 2010.

---

## License

Developed for academic purposes at Vidyalankar Institute of Technology. Free to use for learning and reference.
