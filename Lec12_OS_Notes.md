# Lecture 12: Introduction to Process Scheduling, FCFS, and the Convoy Effect

*Operating Systems, Process Management*

This lecture starts the topic of **process scheduling algorithms**, one of the most important parts of process management. Before any algorithm is covered, the lecture introduces the **terms and jargon** (AT, BT, CT, TAT, WT, Response Time, Throughput) that every later scheduling lecture uses. It then covers the first and simplest algorithm, **FCFS**, and its main problem, the **Convoy Effect**. The Convoy Effect is a common interview question.

---

## 1. Process Scheduling: Quick Recap

### 1.1 The overall flow of a process

```
              LTS                     STS (Dispatch)
  New jobs ─────────►  ┌───────────────┐ ──────────────►  ┌─────────┐
                       │  Ready Queue  │                  │   CPU   │ ──► exit
                       │  P1, P2, P3…  │ ◄──────────────  │  (RUN)  │
                       └───────────────┘   (preempted /   └─────────┘
                              ▲             time quantum        │
                              │             expired)            │ I/O
                              │                                 ▼
                              └─────────────────────────── ( wait )
```

How this works, step by step:

1. **The Long-Term Scheduler (LTS)** brings processes into the **Ready Queue**. Many processes (P1, P2, P3, …) can be waiting there.
2. **The OS picks one process** from the Ready Queue and gives it to the CPU. The **Short-Term Scheduler (STS)** does this, and the actual hand-off is called **dispatching**.
3. The process **runs** on the CPU (RUN state). Then one of three things happens:
   - It **finishes** and exits.
   - Its **time quantum expires** (depending on the algorithm), so it goes **back to the Ready Queue**.
   - It needs **I/O**, so it goes to the **wait state**. After the I/O completes, it returns to the Ready Queue.
4. The cycle repeats.

### 1.2 CPU scheduling vs. the dispatcher

| Term | What it does |
|---|---|
| **CPU Scheduling / Process Scheduling** | Deciding **which** process in the Ready Queue gets the CPU next. Whenever the CPU becomes idle, one process must be picked from the Ready Queue. This selection is done by the **process scheduling algorithm**. |
| **CPU Scheduler** | The component that performs this selection (the short-term scheduler). |
| **Dispatcher** | The module that **actually gives control of the CPU** to the selected process (covered in earlier lectures). |

The scheduler decides who runs next, and the dispatcher hands the CPU over to that process. **The scheduling algorithm defines the rule used to pick the process.**

---

## 2. The Two Types of Scheduling

There are two main paradigms of scheduling algorithms.

### 2.1 Non-Preemptive Scheduling

```
 (P1) ──dispatch──► (CPU) ──► terminate
                       │
                       └────► wait (I/O)
```

- Once the dispatcher gives the CPU to a process, the process **keeps the CPU until one of two things happens**:
  1. The process **terminates**, or
  2. The process goes to the **wait state** to do I/O.
- There is **no other way** for the process to give up the CPU. Once it gets the CPU, it holds on to it.
- **What is missing:** the **time quantum / time sharing** idea. In time sharing, after a process runs for X units of time (the time quantum), the CPU takes it away and sends it back to the Ready Queue. **This does not happen in non-preemptive scheduling.**

### 2.2 Preemptive Scheduling

A process gives up the CPU in any of these cases:

1. It **terminates**. This also happens in non-preemptive scheduling.
2. It goes to **wait** for I/O. This also happens in non-preemptive scheduling.
3. **Its time quantum expires.** This case is **extra** and exists only in preemptive scheduling. The CPU is taken away and the process goes **back to the Ready Queue**.

### 2.3 Comparison: Preemptive vs. Non-Preemptive

| Property | Non-Preemptive | Preemptive | Reason |
|---|---|---|---|
| **Starvation** | **High** ↑ | **Low** ↓ | See below |
| **CPU Utilization** | Lower | **High** ↑ | More processes get a chance to work |
| **Overhead** | Lower | **High** ↑ | Frequent process switching |

#### What is starvation?

Starvation happens when a process **does not get the CPU at all for a long time**. For example, a high-priority job or a very long job keeps holding the CPU. The other jobs in the Ready Queue (for example, low-priority ones) get no time. Those waiting processes are said to be **starved**.

#### Why starvation is higher in non-preemptive scheduling

- The time-sharing part is missing.
- **Example:** A very **CPU-intensive** job arrives that takes 20–30 minutes and has **no I/O instructions** in it. Under non-preemptive scheduling, it holds the CPU for the full 20–30 minutes until it terminates. Every other process in the Ready Queue starves for that whole period.

#### Why starvation is lower in preemptive scheduling

- The time quantum **will** expire, so every process gets **some chance** after a while.
- The instructor corrects himself here: starvation is not *eliminated* in preemptive scheduling, it is **reduced**.

#### Why CPU utilization is higher in preemptive scheduling

- More processes get a chance to do their work, so at any given period the CPU is working on more processes. The reasoning follows directly from the starvation point.

#### Why overhead is higher in preemptive scheduling

- In addition to switching for I/O and termination, there is a **switch every time the quantum expires**.
- **Example:** With a time quantum of 1 second, the running process changes every second. Each time, the short-term scheduler must pick a new process and give it to the CPU. In 10 seconds, that is about 10 process switches.
- This constant switching is the extra overhead.

---

## 3. Goals of CPU Scheduling Algorithms

Suppose you are **designing an OS** and writing its process management code. Before choosing a CPU scheduling algorithm, you decide **what parameters it should optimize**, meaning what goals it should fulfil.

| # | Goal | Meaning / Why |
|---|---|---|
| 1 | **Maximum CPU Utilization** | The CPU is the most important resource. We do not want it to sit idle even for a few seconds. This has been the goal since day 1. |
| 2 | **Minimum Turnaround Time (TAT)** | Once a process enters the Ready Queue, it should finish and exit as quickly as possible. |
| 3 | **Minimum Waiting Time (WT)** | Each process should wait for the CPU as little as possible. Long waiting leads to starvation. |
| 4 | **Minimum Response Time** | A process in the Ready Queue should get the CPU **for the first time** as soon as possible. |
| 5 | **Maximum Throughput** | The system should complete as many processes per unit time as possible. |

The typed PDF notes also list **Minimum Starvation** as a goal. It fits with the waiting-time goal and the starvation discussion above.

### Details of each goal

**Minimum Turnaround Time (TAT)**
```
Ready Queue (P1) ──────── … (runs, waits for I/O, gets preempted, re-queued) … ────────► exit
|◄────────────────────────────── Turnaround Time ─────────────────────────────────►|
```
- This is the time from when the process **first entered the Ready Queue** until it **finally terminates**.
- It includes everything in between: running, going to wait for I/O, being preempted and sent back to the Ready Queue, and so on.

**Minimum Waiting Time**
- A process can be sent back to the Ready Queue, for example when the CPU preempts it because the algorithm says another process should run now. We want every process's total waiting time to be as small as possible.

**Minimum Response Time**
```
Ready Queue (P1) ─────────► CPU (1st time)
|◄──── Response Time ────►|
```
- This covers only the **first** time the process gets the CPU.
- The goal is that the program at least **starts executing** (its instruction cycles begin) soon. We do not want a process to sit in the Ready Queue for a long time without getting the CPU even once.

**Maximum Throughput**
- Throughput measures how good a scheduling algorithm is. It tells us how many processes the whole system (the process management system together with the CPU) completes per unit time.
- **Why we care about all these goals:** we want maximum CPU utilization and a fast, efficient system. That is why these goals are fixed before designing the scheduling algorithm.

---

## 4. Scheduling Terminology (Formal Definitions)

These terms appear in every scheduling problem in the upcoming lectures.

| Term | Abbrev. | Definition | Formula |
|---|---|---|---|
| **Throughput** | — | Number of processes completed per unit time | — |
| **Arrival Time** | **AT** | The time at which the process arrives in the Ready Queue | — |
| **Burst Time** | **BT** | The time the process needs for its execution | — |
| **Completion Time** | **CT** | The time at which the process terminates | — |
| **Turnaround Time** | **TAT** | Time from when the process first enters the ready state until it terminates | **TAT = CT − AT** |
| **Waiting Time** | **WT** | Time the process spends waiting for the CPU | **WT = TAT − BT** |
| **Response Time** | **RT** | Time between the process entering the Ready Queue and getting the CPU for the first time | — |

### 4.1 Arrival Time (AT)
- Think of the system as having a **clock/timeline**. It starts at **t = 0** when the system is switched on and keeps running.
- **AT** is the point on that clock when the process entered the Ready Queue.

### 4.2 Burst Time (BT)
- BT is how much time the process **actually needs** to execute.
- **Intuition:** imagine there is **no other process** in the system, so this process is the only owner of the CPU. The time it would take to run is its burst time.
- BT is the pure execution time with no waiting included.

### 4.3 Completion Time (CT) vs. Turnaround Time (TAT)

These two are easy to confuse:

- **CT** is the **clock time** at which the process terminates. A process moves through many states along the way: it arrives, gets scheduled for the first time, may go to wait, comes back to the Ready Queue, gets scheduled again, has its quantum expire, returns to the Ready Queue, and so on. CT is the clock reading when it finally exits.
- **TAT** is the **duration** from arrival to termination:

$$\text{TAT} = \text{CT} - \text{AT}$$

> ⚠️ **Watch out:** In the video, the instructor describes CT as "the time from when the LTS put the process in the Ready Queue until it exited." That describes a *duration*, which is actually TAT. The correct understanding, confirmed by the written notes and by how the tables are filled in, is this: **CT is a point on the timeline, and TAT = CT − AT is the duration.** If a process arrives at t = 0, the two values are equal. That is why they look the same for P1 in the example below.

### 4.4 Waiting Time (WT)

$$\text{WT} = \text{TAT} - \text{BT}$$

**Why this formula works:**
- **TAT** is how much time the process *actually* spent in the system, from entering the Ready Queue to terminating.
- **BT** is how much time it *ideally* needed if it were the only process.
- The difference, (actual time taken) − (time actually needed), is the time spent **waiting** for the CPU.
- We want this difference to be as small as possible.

---

## 5. FCFS (First Come, First Serve)

### 5.1 The idea
- This is the **simplest** scheduling algorithm and very easy to understand.
- **Rule:** the process that comes into the Ready Queue **first** is given the CPU **first**.
- It is **non-preemptive** as described. A process runs until it finishes, and then the next process runs.

```
Ready Queue (arrival order):  P1 → P2 → P3
                               1     2     3

P1 ──► "Selected first" ──► dispatcher ──► CPU
```

- The FCFS scheduler selects the process that arrived earliest (P1) and tells the dispatcher to give it the CPU.
- When P1 finishes, P2 runs. When P2 finishes, P3 runs.

FCFS is simple, but it has a major problem called the **Convoy Effect**. The two scenarios below show it.

### 5.2 Gantt Chart
To compute CT, TAT, and WT, we draw a **Gantt Chart**. This is a bar showing the **complete timeline**: which process runs during which time interval. From now on, time is measured in **units**, not seconds.

### 5.3 Scenario I: The long job arrives first

**Given:**

| Process | AT | BT |
|:---:|:---:|:---:|
| P1 | 0 | 20 |
| P2 | 1 | 2 |
| P3 | 2 | 2 |

**Dry run:**
1. **t = 0:** Only P1 has arrived, and it has the earliest AT. FCFS schedules **P1**. BT = 20, so it runs until **t = 20**.
2. **t = 20:** P1 is done. P2 arrived at t = 1 and has been waiting. **P2** runs for 2 units, until 20 + 2 = **22**.
3. **t = 22:** **P3**, which has been waiting since t = 2, runs for 2 units, until 22 + 2 = **24**.

**Gantt Chart:**
```
|        P1        |  P2  |  P3  |
0                  20     22     24
```

**Calculations (TAT = CT − AT, WT = TAT − BT):**

| Process | AT | BT | CT | TAT = CT − AT | WT = TAT − BT |
|:---:|:---:|:---:|:---:|:---:|:---:|
| P1 | 0 | 20 | 20 | 20 − 0 = **20** | 20 − 20 = **0** |
| P2 | 1 | 2 | 22 | 22 − 1 = **21** | 21 − 2 = **19** |
| P3 | 2 | 2 | 24 | 24 − 2 = **22** | 22 − 2 = **20** |

- P1 waited **0**. It got the CPU as soon as it arrived.
- P2 had to wait **19** units just to run for 2 units.
- P3 waited **20** units to run for 2 units.

The instructor briefly wrote wrong numbers here and then corrected them. The values in the table above are the correct ones.

$$\text{Average WT} = \frac{0 + 19 + 20}{3} = \frac{39}{3} = \mathbf{13 \text{ units}}$$

### 5.4 Scenario II: The same jobs, with the long job last

We change **only the order** in which the processes arrive. The job with the largest burst time (P1) now comes last. The two short jobs come first.

**Given:**

| Process | AT | BT |
|:---:|:---:|:---:|
| P2 | 0 | 2 |
| P3 | 1 | 2 |
| P1 | 2 | 20 |

**Dry run:**
1. **t = 0:** **P2** arrives first and runs for 2 units, until **t = 2**.
2. **t = 2:** **P3** (arrived at t = 1) runs for 2 units, until **t = 4**.
3. **t = 4:** **P1** (arrived at t = 2) runs for 20 units, until **t = 24**.

**Gantt Chart:**
```
|  P2  |  P3  |        P1        |
0      2      4                  24
```

**Calculations:**

| Process | AT | BT | CT | TAT = CT − AT | WT = TAT − BT |
|:---:|:---:|:---:|:---:|:---:|:---:|
| P2 | 0 | 2 | 2 | **2** | **0** |
| P3 | 1 | 2 | 4 | **3** | **1** |
| P1 | 2 | 20 | 24 | **22** | **2** |

$$\text{Average WT} = \frac{0 + 1 + 2}{3} = \mathbf{1 \text{ unit}}$$

### 5.5 Comparing the two scenarios

| | Scenario I (long job first) | Scenario II (long job last) |
|---|:---:|:---:|
| Order | P1(20) → P2(2) → P3(2) | P2(2) → P3(2) → P1(20) |
| Total time | 24 | 24 |
| **Average WT** | **13 units** | **1 unit** |

The jobs, the total work, and the total time are all the same. Only the order changed, and the average waiting time dropped from **13 to 1**. The reason is that in Scenario I the one heavy job at the front made every short job behind it wait. This is the **Convoy Effect**.

---

## 6. The Convoy Effect ⭐ (Interview Question)

### 6.1 Definition

> **In FCFS, if one process has a longer burst time (BT), it has a major effect on the average waiting time (WT) of the other processes. This is called the Convoy Effect.**

**More general definition (from the notes):**

> **The Convoy Effect is a situation where many processes that need to use a resource for only a short time are blocked by one process that holds that resource for a long time.**
> - It causes **poor resource management**.

### 6.2 Intuition
- When a heavy job with the largest BT comes first, all the other jobs have to wait behind it.
- If the first job runs for 20 units, P2 and P3 sit in the queue that whole time. The average waiting time of the whole Ready Queue rises to **13** because of that one job.
- Put the same job in 3rd position and the average WT becomes **1 unit**.
- It is like a slow truck at the front of a convoy on a narrow road. Every faster vehicle behind it is stuck at its speed.

### 6.3 It is not limited to the CPU or FCFS
- The Convoy Effect is not restricted to CPU scheduling or to FCFS. It can happen with **any resource**.
- If one process holds a resource for a very long time and other processes also want to use it, the waiting processes' waiting time increases.
- That is why it is also described as **poor management of a resource**.

---

## 7. Quick Revision Sheet

**Scheduling basics**
- The scheduler **selects** a process from the Ready Queue, and the dispatcher **gives it the CPU**.
- **Non-preemptive:** a process leaves the CPU only when it **terminates** or goes to **wait for I/O**. There is no time quantum.
- **Preemptive:** a process also leaves the CPU when its **time quantum expires**.

**Preemptive vs. non-preemptive**
- **Starvation:** non-preemptive ↑, preemptive ↓ (reduced, not zero).
- **CPU utilization:** preemptive ↑.
- **Overhead:** preemptive ↑, because of frequent switching.

**Goals of scheduling**
- Maximize: **CPU utilization**, **throughput**.
- Minimize: **TAT**, **WT**, **response time**, **starvation**.

**Formulas**
```
TAT = CT − AT
WT  = TAT − BT
Response Time = (first time the process gets the CPU) − AT
```

**FCFS**
- The process that arrived first gets the CPU first. It is simple and non-preemptive.
- Problem: the **Convoy Effect**. A long-BT job at the front drastically increases the average WT. In the example, it went from 1 to 13 for the same set of jobs.

**Likely interview questions**
- What is the Convoy Effect?
- Preemptive vs. non-preemptive scheduling: which has more starvation, which has more overhead?
- What is the difference between TAT and WT, and between CT and TAT?
