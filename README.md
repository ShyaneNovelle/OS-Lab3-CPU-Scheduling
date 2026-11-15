# OS Laboratory Activity 3
## Multilevel Queue and Real-Time CPU Scheduling

### Student Information
- **Name:** Shyane Novelle R. Canayan
- **Course and Section:** BCA151 / B186
- **Instructor:** Ma'am Fe Bahinting
- **Date Submitted:** November 14, 2026

---

### I. Introduction
This laboratory activity focuses on the implementation and simulation of Multilevel Queue (MLQ) and real-time CPU scheduling algorithms—specifically Rate Monotonic Scheduling (RMS) and Earliest Deadline First (EDF)—using Python and GitHub. Through 1 ms discrete-time simulations, this activity demonstrates how operating systems classify and prioritize processes across separate queues, enforce preemptive fixed-priority and dynamic-priority rules for periodic real-time tasks, generate Gantt charts, and evaluate performance metrics such as turnaround time, waiting time, CPU utilization, and deadline compliance.

---

### II. Objectives
- Explain the operation of Multilevel Queue, RMS, and EDF scheduling algorithms.
- Implement MLQ scheduling using Round Robin and First Come, First Served (FCFS).
- Implement preemptive RMS and EDF scheduling simulations using Python.
- Generate and interpret Gantt charts showing process and task execution.
- Calculate CPU utilization, waiting time, turnaround time, and deadline compliance.
- Create and manage a GitHub repository containing Python source code and laboratory documentation.

---

### III. Scheduling Algorithms

1. **Multilevel Queue (MLQ) Scheduling**
   MLQ partitions the ready queue into separate, permanent queues based on process characteristics and priority requirements. In this simulation, **Queue 1** handles higher-priority interactive processes using **Round Robin (RR)** with a time quantum of $2\text{ ms}$, while **Queue 2** handles lower-priority batch processes using **First Come, First Served (FCFS)**. Queue 1 holds strict preemptive priority over Queue 2: whenever a Queue 1 process arrives, any running Queue 2 process is immediately preempted and resumes only after Queue 1 becomes completely empty.

2. **Rate Monotonic Scheduling (RMS)**
   RMS is a preemptive, **fixed-priority** real-time scheduling algorithm designed for periodic tasks. Task priorities are assigned statically before execution based on task periods ($T$): the task with the shortest period receives the highest priority. If a higher-priority (shorter-period) task is released while a lower-priority task is executing, the lower-priority task is preempted immediately.

3. **Earliest Deadline First (EDF)**
   EDF is a preemptive, **dynamic-priority** real-time scheduling algorithm that prioritizes tasks based on their **absolute deadlines** ($\text{Absolute Deadline} = \text{Release Time} + \text{Relative Deadline}$). At every scheduling point, the ready task instance with the earliest absolute deadline is selected to execute. If a newly released task has an earlier absolute deadline than the currently running task, preemption occurs immediately.

---

### IV. Program Implementation
All three simulations were implemented in Python 3 using standard built-in libraries (`collections.deque`) and simulated over discrete $1\text{ ms}$ time intervals with context-switching overhead ignored:

- **`multilevel_queue/mlq.py`**: Uses two `deque` structures (`q1` and `q2`) to manage ready processes sorted by arrival time and PID. At each $1\text{ ms}$ tick, arriving processes are routed to their assigned queue. Queue 1 executes using a $2\text{ ms}$ countdown (`quantum_left`), re-queuing unfinished processes to the tail of `q1`. Queue 2 executes strictly when `current_q1` is `None` and `q1` is empty, pausing immediately if a Queue 1 process arrives.
- **`real_time_scheduling/rms.py`**: Generates individual periodic job instances across a $20\text{ ms}$ simulation horizon (`SIMULATION_TIME = 20`). At each $1\text{ ms}$ step, the scheduler filters active ready jobs (`release <= time < deadline` and `remaining > 0`) and selects the job belonging to the task with the smallest period $T$ (with release time and task ID as tie-breakers). It also computes total CPU utilization against the Liu & Layland sufficient utilization bound.
- **`real_time_scheduling/edf.py`**: Generates periodic task instances over a $20\text{ ms}$ horizon and evaluates ready jobs at each $1\text{ ms}$ interval. Instead of static period sorting, it dynamically selects the ready job with the minimum `deadline` value (breaking ties using earlier `release` time and `id`), verifying deadline compliance (`completion <= deadline`) at the end of the horizon.

---

### V. Test Data

#### 1. Multilevel Queue (MLQ) Dataset
- **Queue 1 Policy:** Round Robin ($\text{Quantum} = 2\text{ ms}$), Higher Priority
- **Queue 2 Policy:** FCFS, Lower Priority

| Process | Arrival Time (ms) | Burst Time (ms) | Queue |
| :--- | :---: | :---: | :---: |
| **P1** | 0 | 5 | 1 |
| **P2** | 1 | 4 | 1 |
| **P3** | 0 | 6 | 2 |
| **P4** | 2 | 3 | 2 |
| **P5** | 3 | 2 | 1 |

#### 2. Real-Time Scheduling (RMS & EDF) Dataset
- **Simulation Duration:** $20\text{ ms}$ (all tasks initially released at $t = 0\text{ ms}$, with $D_i = T_i$)

| Task | Execution Time ($C$) | Period ($T$) | Relative Deadline ($D$) | RMS Priority |
| :--- | :---: | :---: | :---: | :---: |
| **T1** | 1 ms | 4 ms | 4 ms | Highest |
| **T2** | 1 ms | 5 ms | 5 ms | Second |
| **T3** | 2 ms | 10 ms | 10 ms | Lowest |

---

### VI. Results and Discussion

#### 1. Multilevel Queue Scheduling (`mlq.py`)
**Gantt Chart:**
```text
| P1 [0-2] | P2 [2-4] | P1 [4-6] | P5 [6-8] | P2 [8-10] | P1 [10-11] | P3 [11-17] | P4 [17-20] |
