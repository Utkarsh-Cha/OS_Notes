# OS Lecture 13 — CPU Scheduling Algorithms (Part 2)
### SJF (Non-preemptive & Preemptive/SRTF) · Priority Scheduling (Non-preemptive & Preemptive) · Round Robin

---

## 0. Where This Lecture Fits

- **Previous lecture (Lec 12):** FCFS (First Come First Serve), the simplest CPU scheduling algorithm, and the problem it causes: the **convoy effect**.
- **This lecture:** more algorithms that try to fix FCFS's problems:
  1. **SJF – Shortest Job First** (non-preemptive and preemptive versions)
  2. **Priority Scheduling** (non-preemptive and preemptive versions)
  3. **Round Robin (RR)**, called the most important/"special" algorithm of the lot
- **Next lecture:** Multi-Level Queue (MLQ) scheduling and Multi-Level Feedback Queue (MLFQ) scheduling. These are more complex and closer to how CPU scheduling is actually implemented in real-world operating systems. The algorithms in this lecture are comparatively simple to implement.

> **Interview note:** Product-based companies (Amazon, Adobe, Walmart, etc.) focus on OS in technical interviews, so these algorithms, their drawbacks, and their trade-offs are important.

---

## 1. Quick Recap of Terminology (from Lec 12)

These terms are used in every numerical below. If any is unclear, revise Lec 12.

| Term | Full form | Meaning / Formula |
|---|---|---|
| **AT** | Arrival Time | Time at which the process arrives in the ready queue |
| **BT** | Burst Time | CPU time the process needs to complete execution |
| **CT** | Completion Time | Time at which the process finishes execution |
| **TAT** | Turn-Around Time | `TAT = CT − AT` (total time from arrival to completion) |
| **WT** | Waiting Time | `WT = TAT − BT` (time spent waiting in the ready queue) |
| **Avg WT** | Average Waiting Time | Sum of all WTs ÷ number of processes. This is the main metric used to compare algorithms. |
| **Gantt chart** | — | Timeline showing which process runs on the CPU in which interval. The instructor stresses that drawing it is very important for solving problems. |

**Convoy effect (recap):** In FCFS, if a process with a very large burst time gets the CPU first, it holds the CPU for a long time and all the other (possibly short) processes wait behind it. This greatly increases the average waiting time and causes starvation of the other jobs.

**Non-preemptive vs Preemptive (recap):**
- **Non-preemptive:** once a process gets the CPU, it keeps it until it finishes (or blocks). The CPU is never taken away forcibly.
- **Preemptive:** the OS can take the CPU away from the running process (e.g., when a "better" process arrives or a time quantum expires) and put it back in the ready queue.

---

## 2. SJF — Shortest Job First (Non-Preemptive)

### 2.1 Idea

> **The process with the least burst time gets the CPU.**

- Among all the jobs currently in the ready queue, the scheduler checks which one has the **smallest burst time**. Smallest BT means "the job that will finish fastest".
- That job is given the CPU first; jobs with higher burst times are given the CPU later.
- **Non-preemptive version:** once a process gets the CPU, it runs to completion. No preemption.

### 2.2 Why do this?

In FCFS, when a process with a very high burst time was scheduled first, the average waiting time increased a lot (the convoy effect). SJF tries to resolve this by running short jobs first, so fewer jobs are stuck waiting behind a long one.

### 2.3 The Biggest Problem: Burst Time Cannot Be Known in Advance

- Suppose processes P1, P2, P3 arrive in the ready queue. To apply SJF, the OS must know their burst times **before** scheduling them, i.e., before they have even run.
- The OS can only **estimate** BT using **heuristics** (educated guesses / logic built into the algorithm), for example:
  - the **size of the process / its code size**, or
  - **history**: if the same job ran earlier, how long did it take then?
- So SJF is **completely estimation-based**. Knowing the *ideal/actual* BT beforehand is **nearly impossible**.
- The OS schedules in the *hope* that its estimate is correct.

**Example of how the estimate can go wrong:**

```text
( P1  P2  P3 )  ->  ?? BT
  ↓   ↓   ↓
  2   5   10      (estimated BT in seconds)  -> P1 goes to CPU first
```

The OS estimates P1 = 2 s, P2 = 5 s, P3 = 10 s, so it picks P1 as the shortest. But P1 may actually take **20 s**. The OS has not looked through the whole code, and actual execution time cannot be known until the process runs. This estimation problem is a key **drawback** of SJF.

> ⚠️ In all numericals below, the BT values given in the table are **estimated** values. We don't actually know the real BT.

### 2.4 Selection Criteria

```text
Criteria: AT + BT
```

- **AT:** only processes that have *arrived* by the current time are in the ready queue and can be considered.
- **BT:** among those, the one with the smallest BT is picked.

### 2.5 Solved Example (Non-Preemptive SJF)

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 8 |
| P2 | 1 | 4 |
| P3 | 2 | 9 |
| P4 | 3 | 5 |

**Dry run:**

1. **t = 0:** Only P1 has arrived, so P1 gets the CPU. Being non-preemptive, it runs its full BT of 8 s → finishes at **t = 8**.
2. **t = 8:** By now, P2, P3, and P4 have all arrived (the latest arrival was at t = 3). Ready queue = {P2 (4), P3 (9), P4 (5)}. Smallest BT = **P2 (4)** → runs 8 → **12**.
3. **t = 12:** Remaining = {P3 (9), P4 (5)}. Smallest BT = **P4 (5)** → runs 12 → **17**.
   - Note: **FCFS would have picked P3 here** (it arrived earlier), but SJF picks P4 because its BT is smaller.
4. **t = 17:** Only P3 remains → runs 17 → **26**.

**Gantt chart:**

```text
|  P1  |  P2  |  P4  |  P3  |
0      8      12     17     26
```

**Final table:**

| P | AT | BT | CT | TAT = CT − AT | WT = TAT − BT |
|---|---|---|---|---|---|
| P1 | 0 | 8 | 8 | 8 | 0 |
| P2 | 1 | 4 | 12 | 11 | 7 |
| P3 | 2 | 9 | 26 | 24 | 15 |
| P4 | 3 | 5 | 17 | 14 | 9 |

**Average WT = (0 + 7 + 15 + 9) / 4 = 31 / 4 = 7.75 s**

### 2.6 Problem With Non-Preemptive SJF: Convoy Effect Can Still Happen

- Imagine P1 (arriving at t = 0) had BT = **80** instead of 8.
- Since it's the only process at t = 0, it gets the CPU, and because there is no preemption it runs for **all 80 seconds**.
- Every other job waits 80 seconds, even though they are shorter. They starve and don't get the CPU even once during that time.
- So **non-preemptive SJF can be a victim of the convoy effect**. SJF only helps when choosing among jobs that are *already waiting*; it cannot help if a long job has already grabbed the CPU.

---

## 3. SJF — Preemptive Version (SRTF: Shortest Remaining Time First)

On the lecture's summary notes, the preemptive version of SJF is labelled **SRTF**.

### 3.1 Idea

- Criteria is still **AT + BT**, but now **preemption is added**.
- Whenever a new process arrives in the ready queue, the scheduler compares its burst time with the **remaining** burst time of the currently running process.
- If the new process has a **smaller** BT, the running process is **preempted** (sent back to the ready queue with its remaining BT) and the new one gets the CPU.

### 3.2 Solved Example (same processes as before)

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 8 |
| P2 | 1 | 4 |
| P3 | 2 | 9 |
| P4 | 3 | 5 |

**Dry run:**

1. **t = 0:** Only P1 is in the ready queue → P1 is scheduled (the others haven't arrived yet).
2. **t = 1:** P1 has run for 1 s → **P1's remaining BT = 8 − 1 = 7**. P2 arrives with BT = 4, which is less than 7 → **P1 is preempted**, and P2 is scheduled.
3. **t = 2:** P3 arrives (BT 9). P2's remaining = 3. 9 > 3 → P2 continues.
4. **t = 3:** P4 arrives (BT 5). P2's remaining = 2. 5 > 2 → P2 continues.
   - So by the end of P2's run, P3 and P4 are in the queue, but their BTs (9 and 5) are **greater** than P2's remaining time, so P2 is never preempted and runs to termination at **t = 5**.
   - (In the lecture, the instructor says "9 and 5 are lesser than P2" and "P1 will run completely" at this point. These are slips of the tongue; he means their BTs are *greater*, and *P2* runs to completion, as the Gantt chart shows.)
5. **t = 5:** Ready queue = {P1 (remaining 7), P3 (9), P4 (5)}. Smallest = **P4 (5)** → runs 5 → **10**.
6. **t = 10:** Ready queue = {P1 (7), P3 (9)}. Smallest = **P1 (7)** → no new processes are arriving, so it runs fully 10 → **17**.
7. **t = 17:** Only P3 remains → runs 17 → **26**.

**Gantt chart:**

```text
|  P1  |  P2  |  P4  |  P1  |  P3  |
0      1      5      10     17     26
```

**Final table:**

| P | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| P1 | 0 | 8 | 17 | 17 | 9 |
| P2 | 1 | 4 | 5 | 4 | 0 |
| P3 | 2 | 9 | 26 | 24 | 15 |
| P4 | 3 | 5 | 10 | 7 | 2 |

**Average WT = (9 + 0 + 15 + 2) / 4 = 26 / 4 = 6.5 units**

✅ With this optimization, average waiting time dropped from **7.75 → 6.5**.

### 3.3 Is There a Convoy Effect in Preemptive SJF? → **No**

**Question (think about it):** Will the convoy effect occur in preemptive SJF?

**Answer: No.** Reasoning:

- First understand *why* the convoy effect occurs: a job with a very large BT gets the CPU first and **hogs** it for a long time while all other jobs starve.
- In preemptive SJF, even if the first job has BT = 80 s, it only runs **until a job with a lower BT arrives**. As soon as a shorter job arrives, the long job is preempted and the short job gets the CPU. If an even shorter job arrives later, that one is scheduled next, and so on.
- So long (high-BT) jobs keep **shifting down** the order, and short jobs are never stuck behind them.
- Result: **average waiting time is always low. Preemptive SJF gives the lowest average waiting time.**

### 3.4 Biggest Challenge of SJF (both versions): Nearly Impossible to Implement Ideally

- The burst time **cannot be known beforehand**.
- Example: a programmer writes code expecting it to finish in about 70 s, but by mistake a `while(1)` infinite loop is left inside. The program is erroneous and keeps running. The estimated BT was 70 s, but the real BT is **unlimited**.
- So any scheduling decision based on estimated BT can be wrong.

### 3.5 SJF Summary

| | Non-preemptive SJF | Preemptive SJF (SRTF) |
|---|---|---|
| Criteria | AT + BT | AT + BT + preemption on arrival of a shorter job |
| Convoy effect | **Yes**, possible (a long job that starts first holds the CPU) | **No** |
| Starvation | More | Less |
| Avg WT (example) | 7.75 | 6.5 (lowest) |
| Main problem | BT must be estimated (nearly impossible to know) | Same |

**Conclusion:** the preemptive version is better than the non-preemptive version because it has no convoy effect and less starvation.

---

## 4. Priority Scheduling

### 4.1 Idea

- In the algorithms so far, no explicit "priority" was assigned. In SJF, a job's importance was decided *only* by its burst time.
- **SJF is a special case of priority scheduling:** no separate priority is assigned, but effectively the job with the **lowest burst time has the highest priority** at that point of time.
- In **Priority Scheduling**, we **explicitly assign a priority to each job**. When a job enters the ready queue, it is given a priority number. For example:

```text
( P1   P2 )
  ↓    ↓
  2    4      <- assigned priorities
```

- **The job with the highest priority is scheduled first.**
- In the lecture's examples, **a larger number means a higher priority** (e.g., 12 is higher than 2). Always check the convention in a question, since some textbooks use "smaller number = higher priority".

### 4.2 Non-Preemptive Priority Scheduling

- **Criteria:** priority. At each scheduling point, the process with the highest priority among the arrived processes gets the CPU.
- **Non-preemptive:** once a process gets the CPU, it keeps it until completion.
- The instructor's remark: "By now you can see that the non-preemptive versions are generally poor, and this one is too."

#### Solved Example

| P | Priority | AT | BT |
|---|---|---|---|
| P1 | 2 | 0 | 4 |
| P2 | 4 | 1 | 2 |
| P3 | 6 | 2 | 3 |
| P4 | 10 | 3 | 5 |
| P5 | 8 | 4 | 1 |
| P6 | 12 | 5 | 4 |
| P7 | 9 | 6 | 6 |

**Dry run:**

1. **t = 0:** Only P1 is present → P1 runs fully (non-preemptive), 0 → **4**.
2. **t = 4:** Arrived and waiting: P2 (4), P3 (6), P4 (10), P5 (8). Highest priority = **P4 (10)** → runs 4 → **9**.
3. **t = 9:** All processes have arrived (last AT = 6). Waiting: P2 (4), P3 (6), P5 (8), P6 (12), P7 (9). Highest = **P6 (12)** → runs 9 → **13**.
4. **t = 13:** Next highest = **P7 (9)** → runs 6 s, 13 → **19**.
5. **t = 19:** Next = **P5 (8)** → runs just 1 s, 19 → **20**.
6. **t = 20:** Next = **P3 (6)** → runs 3 s, 20 → **23**.
7. **t = 23:** Last = **P2 (4)** → runs 2 s, 23 → **25**.

**Gantt chart:**

```text
|  P1  |  P4  |  P6  |  P7  |  P5  |  P3  |  P2  |
0      4      9      13     19     20     23     25
```

**Final table:**

| P | Priority | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|---|
| P1 | 2 | 0 | 4 | 4 | 4 | 0 |
| P2 | 4 | 1 | 2 | 25 | 24 | 22 |
| P3 | 6 | 2 | 3 | 23 | 21 | 18 |
| P4 | 10 | 3 | 5 | 9 | 6 | 1 |
| P5 | 8 | 4 | 1 | 20 | 16 | 15 |
| P6 | 12 | 5 | 4 | 13 | 8 | 4 |
| P7 | 9 | 6 | 6 | 19 | 13 | 7 |

**Average WT (as written in the lecture) = 9.714 s**

> ⚠️ **Arithmetic check:** Adding the WT column gives 0 + 22 + 18 + 1 + 15 + 4 + 7 = **67**, and 67 / 7 = **9.571 s**. The 9.714 shown in the lecture corresponds to a sum of 68, so it appears to be a small calculation slip. The CT, TAT, and WT values in the table above are all correct; use 9.571 if you verify it yourself.

**Observation (keep this in mind):** every time, the **low-priority jobs get pushed to the back**, and high-priority jobs keep getting scheduled. This creates a serious problem, discussed in section 4.4.

### 4.3 Preemptive Priority Scheduling

- Priorities are still assigned to each process.
- As time moves forward and new processes arrive, **if a process with a higher priority than the currently running one arrives, the current process is preempted** and the higher-priority one gets the CPU.
- **Criteria:** whichever process has the highest priority at that moment.

#### Solved Example (same table as above)

| P | Priority | AT | BT |
|---|---|---|---|
| P1 | 2 | 0 | 4 |
| P2 | 4 | 1 | 2 |
| P3 | 6 | 2 | 3 |
| P4 | 10 | 3 | 5 |
| P5 | 8 | 4 | 1 |
| P6 | 12 | 5 | 4 |
| P7 | 9 | 6 | 6 |

**Dry run** (remaining BT is tracked as processes get preempted):

| Time | Event | Decision | Remaining BT after this slice |
|---|---|---|---|
| 0 → 1 | Only P1 present | P1 runs | P1: 4 → **3** |
| 1 | P2 (prio 4) arrives; 4 > 2 | **Preempt P1**, run P2 | |
| 1 → 2 | P2 runs | | P2: 2 → **1** |
| 2 | P3 (prio 6) arrives; 6 > 4 | **Preempt P2**, run P3 | |
| 2 → 3 | P3 runs | | P3: 3 → **2** |
| 3 | P4 (prio 10) arrives; 10 > 6 | **Preempt P3**, run P4 | |
| 3 → 5 | P4 runs. At t = 4, P5 (prio 8) arrives, but 8 < 10, so no preemption. The next higher-priority job (P6) arrives 2 s later. | | P4: 5 → **3** |
| 5 | P6 (prio 12) arrives; 12 > 10 | **Preempt P4**, run P6 | |
| 5 → 9 | P6 is the highest priority of all, so it runs to completion. P7 (prio 9) arrives at t = 6 but can't preempt it. | | P6: 4 → **0** ✅ (CT 9) |
| 9 → 12 | Highest remaining = P4 (10) | P4 runs its remaining 3 s | P4: 3 → **0** ✅ (CT 12) |
| 12 → 18 | Highest remaining = P7 (9) | P7 runs fully (6 s) | P7 → **0** ✅ (CT 18) |
| 18 → 19 | Highest remaining = P5 (8) | P5 runs 1 s | P5 → **0** ✅ (CT 19) |
| 19 → 21 | Highest remaining = P3 (6) | P3 runs remaining 2 s | P3 → **0** ✅ (CT 21) |
| 21 → 22 | Highest remaining = P2 (4) | P2 runs remaining 1 s | P2 → **0** ✅ (CT 22) |
| 22 → 25 | Only P1 (2) left | P1 runs remaining 3 s | P1 → **0** ✅ (CT 25) |

(In the lecture, the instructor first wrote P4 as running 9 → 11, then corrected himself: P4 had already run 2 s, so **3 s remain**, and it runs 9 → **12**.)

**BT changes over execution (as tracked on screen):**

```text
P1: 4 -> 3 -> 0
P2: 2 -> 1 -> 0
P3: 3 -> 2 -> 0
P4: 5 -> 3 -> 0
P5: 1 -> 0
P6: 4 -> 0
P7: 6 -> 0
```

**Gantt chart:**

```text
| P1 | P2 | P3 | P4 | P6 | P4 | P7 | P5 | P3 | P2 | P1 |
0    1    2    3    5    9    12   18   19   21   22   25
```

**Final table:**

| P | Priority | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|---|
| P1 | 2 | 0 | 4 | 25 | 25 | 21 |
| P2 | 4 | 1 | 2 | 22 | 21 | 19 |
| P3 | 6 | 2 | 3 | 21 | 19 | 16 |
| P4 | 10 | 3 | 5 | 12 | 9 | 4 |
| P5 | 8 | 4 | 1 | 19 | 15 | 14 |
| P6 | 12 | 5 | 4 | 9 | 4 | 0 |
| P7 | 9 | 6 | 6 | 18 | 12 | 6 |

**Average WT = (21 + 19 + 16 + 4 + 14 + 0 + 6) / 7 = 80 / 7 ≈ 11.4 s**

(The instructor left this as **homework**: compute the average WT yourself and verify it comes to 11.4.)

#### Doubt: Why does the scheduler check every 1 second? Why not every 0.5 s?

- In these examples, we **assume** the algorithm checks the ready queue every 1 second to see whether a new (higher-priority) process has arrived.
- This is an **assumption that depends on implementation**: how you implement your scheduling algorithm decides how often it checks.
- Since all arrival times in the problems are whole numbers, checking every second gives the correct result.

### 4.4 Observations: Convoy Effect Returns (Even Worse)

- Look at **P2**: its burst time is only **2 s**. Under SJF, it would have been executed early and finished quickly. Under preemptive priority scheduling, it completes at **t = 22**, because its priority is low.
- **Why this creates a convoy effect:** if high-priority jobs keep arriving (and their burst times are also large), the low-priority jobs keep shifting down. Even if a low-priority job has a tiny BT (1–2 s), it has to keep waiting. This pushes the average waiting time up.
- In this example, the **preemptive version (≈11.4)** has an even **higher average WT than the non-preemptive version (≈9.6)**.
- So yes: priority scheduling has a convoy effect, and a "terrible" one, because low-priority jobs never get a chance while high-priority jobs keep coming.

### 4.5 Biggest Drawback: Indefinite Waiting (Extreme Starvation)

```text
Biggest Drawback => (both Non-preemptive AND Preemptive)
Indefinite waiting  OR  Extreme starvation
```

> **Interview point:** If asked for the biggest drawback of priority scheduling, the answer is **indefinite waiting / indefinite blocking (extreme starvation)**. It exists in **both** the non-preemptive and preemptive versions.

**What it means:**

```text
( P1  P2  P3   P4   P5  )
  ↓   ↓   ↓    ↓    ↓
  1   2   100  101  102     <- priorities
```

- P1 and P2 arrive with priorities 1 and 2 and may start running a little.
- Then P3, P4, P5 arrive with very high priorities (100, 101, 102). These high-priority jobs keep getting scheduled among themselves.
- When they terminate, a new job P6 arrives with an even higher priority (say 200), and so on. (The numbers are arbitrary; the point is that high-priority jobs keep arriving.)
- The poor low-priority jobs (priority 1 and 2) get **blocked forever**. If the system keeps running for a very long time and high-priority jobs keep arriving, the low-priority jobs go into **indefinite waiting** and **never get the CPU**.
- The instructor calls indefinite waiting the **extreme version of the convoy effect**: only the highest-priority jobs keep getting CPU time; low-priority jobs (even ones with small BT) never get a chance, so the waiting time of the whole ready queue keeps rising.

#### Extreme Example: The IBM 7094 Rumor

```text
Rumor => IBM 7094 at MIT
ON -> 1967 -> Job submitted
1973 -> checked -> Lowest-priority job still in Ready Queue
```

- This is described as a **rumor**. Even the books present it as a rumor, and it's not known whether it's true.
- The story: an **IBM 7094** system at **MIT** was turned on, and jobs were submitted to it in **1967**.
- Because of this drawback of priority scheduling, the **lowest-priority job(s) stayed in the ready queue for years and never got the CPU**.
- When the system was checked in **1973**, the low-priority job(s) submitted in 1967 were still sitting in the ready queue.
- This shows how serious indefinite waiting can be.

**Why keep priority scheduling at all, then?** Because the algorithm itself is good and necessary. When studying types of operating systems, one requirement was that **high-priority jobs should execute first** (e.g., the **anti-virus example**: an antivirus job needing urgent attention must run first). So instead of discarding the algorithm, we fix its drawback.

### 4.6 Solution to Indefinite Waiting: **Ageing**

```text
Ageing => Gradually increasing the priorities of the lowest-priority jobs
e.g., every 15 min -> lowest-priority jobs -> priority + 1
```

- Ageing adds **one extra criterion**: the priority of waiting (low-priority) jobs is **gradually increased** over time.
- **Example:** every 15 minutes, add **+1** to the priority of the low-priority jobs that are still waiting. This interval is **customizable** (the instructor says 15 minutes and later 15 seconds; the exact value is chosen by the OS designer).
- As time passes, a low-priority job's priority slowly keeps rising. After some time, it **will definitely get a chance**: no matter how high-priority a newly arrived job is, the oldest ("most aged") waiting process will have accumulated a very high priority by then.
- If the IBM 7094 system had used ageing, the low-priority jobs would eventually have run.

### 4.7 Priority Scheduling Summary

| Aspect | Details |
|---|---|
| Criteria | Explicitly assigned priority (highest priority runs first) |
| Relation to SJF | SJF is a special case (priority = inverse of BT) |
| Non-preemptive avg WT (example) | 9.571 (9.714 written in lecture) |
| Preemptive avg WT (example) | ≈ 11.4 (even higher) |
| Convoy effect | **Yes, extreme** |
| Biggest drawback | **Indefinite waiting / extreme starvation** (in both versions) |
| Solution | **Ageing** |

---

## 5. Round Robin (RR) Scheduling

### 5.1 Key Features

```text
* Round-Robin (RR)
  - Most Popular
  - FCFS (Preemptive) version
  - Criteria: AT + TQ  (doesn't depend on BT)
  - Designed for Time-sharing
  - Easy to implement
```

1. **Why "Round Robin"?** It gives **every process in the ready queue an equal chance**. Every process gets the CPU fairly soon, in turn.
2. **Waiting time is quite low**, and it has the **lowest starvation** of all these algorithms, because after a short while, every other process's turn is bound to come.
3. Since starvation is low, there is **no convoy effect**.
4. **Most popular algorithm.** It is still used at different levels of operating systems today.
5. **It is the preemptive version of FCFS.** There is no priority-like criterion.
6. **Criteria = AT + TQ (Time Quantum).** It does **not depend on BT**.
   - This removes SJF's most problematic area: SJF needed the burst time (which is ideally not possible to know) before it could schedule. RR needs only arrival time and the time quantum.
7. **Designed for time-sharing systems.** Recall: in time-sharing, any process is **preempted after a fixed time quantum** and put back in the ready queue. RR is exactly this.
   - Time-sharing is used in **multitasking OSes**, so RR is the most popular choice there.
8. **Easy to implement**, since its criteria (AT and TQ) are very simple (see the flowchart below).

**Time Quantum (TQ):** a fixed time slice decided by the OS designer/developer. Each process can run for at most one TQ at a time before it is preempted.

### 5.2 How RR Works (Flowchart)

```text
                  [ Ready Queue ]
                         │
                         ▼
             [ Pick a process (FCFS) ]
                         │
                         ▼
             /                      \
            /  BT < TQ ?             \  No
           /                          \
          ▼ Yes                        ▼
[ Execute till termination ]   [ Execute for TQ ]
          │                            │ (TQ expires → preempt)
          ▼                            ▼
   [ Terminate state ]          /              \
                               /  Process       \ No
                              /  execution done? \
                             /                    \
                            ▼ Yes                  ▼
                     [ Terminate state ]   [ Re-enter Ready Queue ]
```

**Step-by-step explanation:**

1. Processes keep entering the **ready queue** as their arrival times come.
2. The scheduler **picks a process in FCFS order** (the one at the front of the queue, i.e., the earliest arrival).
3. **If the process's (remaining) burst time is less than (or equal to) the TQ**, it can finish within the time quantum → it executes and goes to the **terminate state**.
   - Example: TQ = 2 and a process has exactly 2 s of BT left → it runs 2 s and goes straight to termination.
4. **Otherwise (BT > TQ)**, it executes for exactly **one TQ**. When the TQ expires, it is **preempted** and put back at the **end of the ready queue**.

**Important practical intuition:** whether the BT is known or not **doesn't matter** in RR. No estimation is done. In real life, the process just runs for the TQ (say 2 s), then the OS checks: *has it terminated?*
- If **yes** → move it to the terminate state.
- If **no** → put it back in the ready queue.

Even if a process's BT turns out to be larger than expected, RR is unaffected. This is why RR is easy to implement.

### 5.3 Solved Example (TQ = 2 s)

> While solving this, notice how many overheads (context switches) occur.

| P | AT | BT |
|---|---|---|
| P1 | 0 | 4 |
| P2 | 1 | 5 |
| P3 | 2 | 2 |
| P4 | 3 | 1 |
| P5 | 4 | 6 |
| P6 | 6 | 3 |

**Queue convention used:** when a process is preempted at the same moment new processes arrive, the **newly arrived processes are placed in the queue before the preempted process**.

**Dry run:**

| Time slice | Running | What happens | Ready queue after this slice (front → back) |
|---|---|---|---|
| 0 → 2 | **P1** | Runs 2 s (TQ). Remaining 4 → **2**. Not terminated → preempted. P2 (AT 1) and P3 (AT 2) have arrived. | P2, P3, **P1** |
| 2 → 4 | **P2** | Runs 2 s. Remaining 5 → **3** → preempted. P4 (AT 3) and P5 (AT 4) have arrived. | P3, P1, P4, P5, **P2** |
| 4 → 6 | **P3** | Runs 2 s. Remaining 2 → **0** → **terminates (CT 6)**. P6 (AT 6) arrives. | P1, P4, P5, P2, P6 |
| 6 → 8 | **P1** | Runs its remaining 2 s → **0** → **terminates (CT 8)**. No new arrivals (last AT was 6). | P4, P5, P2, P6 |
| 8 → 9 | **P4** | Tries to run 2 s, but its BT is only 1 → finishes before TQ expires → **terminates (CT 9)**. | P5, P2, P6 |
| 9 → 11 | **P5** | Runs 2 s. Remaining 6 → **4** → preempted. | P2, P6, **P5** |
| 11 → 13 | **P2** | Runs 2 s. Remaining 3 → **1** → preempted. | P6, P5, **P2** |
| 13 → 15 | **P6** | Runs 2 s. Remaining 3 → **1** → preempted. | P5, P2, **P6** |
| 15 → 17 | **P5** | Runs 2 s. Remaining 4 → **2** → preempted. | P2, P6, **P5** |
| 17 → 18 | **P2** | Runs remaining 1 s → **terminates (CT 18)**. | P6, P5 |
| 18 → 19 | **P6** | Runs remaining 1 s → **terminates (CT 19)**. | P5 |
| 19 → 21 | **P5** | Last process left; runs remaining 2 s → **terminates (CT 21)**. | — |

**Gantt chart:**

```text
| P1 | P2 | P3 | P1 | P4 | P5 | P2 | P6 | P5 | P2 | P6 | P5 |
0    2    4    6    8    9    11   13   15   17   18   19   21
```

**Order in which processes were taken from the queue:**

```text
P1 -> P2 -> P3 -> P1 -> P4 -> P5 -> P2 -> P6 -> P5 -> P2 -> P6 -> P5
```

**BT changes over execution:**

```text
P1: 4 -> 2 -> 0
P2: 5 -> 3 -> 1 -> 0
P3: 2 -> 0
P4: 1 -> 0
P5: 6 -> 4 -> 2 -> 0
P6: 3 -> 1 -> 0
```

**Results table.** The lecture stops at the Gantt chart. The CT values come directly from it; TAT and WT are computed here with the same formulas, for practice.

| P | AT | BT | CT | TAT = CT − AT | WT = TAT − BT |
|---|---|---|---|---|---|
| P1 | 0 | 4 | 8 | 8 | 4 |
| P2 | 1 | 5 | 18 | 17 | 12 |
| P3 | 2 | 2 | 6 | 4 | 2 |
| P4 | 3 | 1 | 9 | 6 | 5 |
| P5 | 4 | 6 | 21 | 17 | 11 |
| P6 | 6 | 3 | 19 | 13 | 10 |

Average WT = 44 / 6 ≈ 7.33

### 5.4 Drawback of RR: High Overhead (Context Switching)

- Drawing this Gantt chart took a lot of effort because **after every TQ, the scheduler must decide whether to preempt** the process or not.
- In the OS, this translates to **overheads**: **context switching happens very frequently in Round Robin.** In the example above, the CPU switched between processes 11 times in just 21 s.
- **Trade-off:**
  - ✅ Starvation is (nearly) completely removed.
  - ✅ Convoy effect is removed.
  - ❌ But overheads (context switches) have increased.
- We ideally want a **balance**: a minimal convoy effect *and* minimal overhead. Here, trying to bring the convoy effect to zero increased the overhead.

### 5.5 Effect of the Time Quantum on Overhead

```text
TQ = 2s  -> some context switching
TQ = 1s  -> even MORE context switching
```

> **Interview question:** *In the RR algorithm, what determines the overhead?*
> **Answer: the Time Quantum (TQ).**

| TQ | Number of context switches | Overhead |
|---|---|---|
| **Larger TQ** | Fewer | **Lower** |
| **Smaller TQ** | More | **Higher** |

If TQ were reduced from 2 s to 1 s in the example above, processes would be preempted even more often, increasing context switching further.

---

## 6. Overall Comparison of the Algorithms

| Algorithm | Selection criteria | Preemptive? | Convoy effect | Starvation | Key problem |
|---|---|---|---|---|---|
| FCFS (Lec 12) | AT | No | Yes | Yes | Long first job delays everyone |
| SJF (Non-preemptive) | AT + BT | No | **Yes** | Yes | BT must be estimated; long job that starts first hogs the CPU |
| SJF Preemptive (SRTF) | AT + BT (+ preemption) | Yes | **No** | Less | BT must be estimated (nearly impossible to know) |
| Priority (Non-preemptive) | Priority | No | Yes | Yes | **Indefinite waiting** |
| Priority (Preemptive) | Priority (+ preemption) | Yes | Yes (extreme) | Extreme | **Indefinite waiting** → fixed by **ageing** |
| Round Robin | AT + TQ | Yes (after every TQ) | **No** | **Lowest** | High context-switching overhead (controlled by TQ) |

**Average WT in the lecture's examples:**
- SJF (same 4 processes): Non-preemptive **7.75** → Preemptive **6.5**
- Priority (same 7 processes): Non-preemptive **9.571** (9.714 written in lecture) → Preemptive **≈11.4**

---

## 7. Homework / Practice Suggestions from the Instructor

1. **Compute the average WT for preemptive priority scheduling** and verify that it is **11.4**.
2. **Implement these scheduling algorithms in code**, using data structures from your DSA course:
   - Take the Gantt-chart problems solved above as problem statements and write programs that **simulate** these algorithms.
   - **SJF → Min-heap:** keep inserting processes as they arrive; the min-heap always keeps the process with the **lowest BT at the top** (e.g., if P3 has the lowest BT at that moment, it stays at the top and gets scheduled next).
   - **Priority Scheduling → Priority queue.**
   - **Round Robin → figure out which data structure fits.** (Hint: RR repeatedly takes from the front and puts preempted processes at the back, which is FIFO **queue** behaviour.)
   - The instructor says he implemented these himself when he was learning and really enjoyed it.

---

## 8. Quick Revision Checklist

- [ ] SJF = least BT gets the CPU; criteria **AT + BT**.
- [ ] BT is only **estimated** (heuristics: code size, past history). Actual BT is nearly impossible to know (`while(1)` example).
- [ ] Non-preemptive SJF **can** have a convoy effect (BT = 80 example).
- [ ] Preemptive SJF (SRTF) has **no convoy effect** and the **lowest average WT**; long jobs shift down.
- [ ] SJF is a **special case of priority scheduling**.
- [ ] Priority scheduling: explicitly assigned priorities; highest runs first (larger number = higher priority in these examples).
- [ ] Priority scheduling has an **extreme convoy effect**; low-priority short jobs wait a long time.
- [ ] **Biggest drawback of priority scheduling (both versions): indefinite waiting / extreme starvation.**
- [ ] IBM 7094 at MIT rumor: jobs submitted in 1967 were still waiting in 1973.
- [ ] **Solution: ageing**, i.e., gradually increase the priority of waiting low-priority jobs (e.g., +1 every 15 min, customizable).
- [ ] RR = **preemptive FCFS**; criteria **AT + TQ**; does **not** depend on BT.
- [ ] RR: most popular; designed for **time-sharing / multitasking**; easy to implement; lowest starvation; no convoy effect.
- [ ] RR drawback: **high context-switching overhead**; **TQ determines overhead** (larger TQ → less overhead, smaller TQ → more overhead).
- [ ] Next lecture: **Multi-Level Queue** and **Multi-Level Feedback Queue** scheduling.
