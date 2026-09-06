# Operating Systems — Lecture 10
## Process States | Process Queues | Schedulers | Dispatcher | Degree of Multiprogramming

---

## 0. Where This Lecture Fits (Recap of Lecture 9)

In the previous lecture we discussed the **PCB (Process Control Block)** and the **attributes of a process**. The PCB is the data structure that the OS maintains for *every* process, and one of the fields/blocks inside the PCB was called **"Process State"**.

That field stores **what the process is currently doing** — i.e. its current state at this instant.

This lecture "double-clicks" on that one field: **what are all the possible values a process's state can take, and how does a process move from one to another?**

---

## 1. What Is a "Process State"?

A process is not a static thing. From the moment it is created until the moment it finishes, it keeps **changing state**.

> **Definition:** As a process executes, it changes state. At any given moment, a process is in exactly **one** of a fixed set of states, and this current state is recorded in the **Process State field of its PCB**.

### The Life Cycle Idea

On the board the instructor wrote:

```
process state :-
   - Life cycle
   New  ->  Generation  ->  Termination
```

The **life cycle of a process** spans:

- **Right from its New state** — i.e. from its **generation / creation**,
- **up to its Termination** — i.e. when it finishes and is removed.

Everything that happens to a process *in between* generation and termination — all the different states it passes through — is what we study here.

---

## 2. The Five Process States

The instructor first gave the five definitions verbally/on the board, then explained them again using the transition diagram. Both passes are merged below.

---

### 2.1 New State

**Board notes:**
```
1. New
   Program  ->  Process
   Being done
```

**Meaning:**

- The OS is **about to pick up a program** (which is lying on the disk/secondary storage) and **convert it into a process**.
- The conversion `Program → Process` is **in progress** — it is "being done", not yet done.
- **While this conversion is happening**, the Process State field inside that particular process's PCB literally reads **`New`**.

**Key intuition:** "New" is not "the process is waiting"; it is "the process is *being born*". The process is currently under construction.

**Formal definition (from the lecture's summary notes):**
> **New:** OS is about to pick the program & convert it into process. OR the process is being created.

---

### 2.2 Ready State

**Board notes:**
```
2. Ready :-
   Process is in Memory
   Ready Queue.
```

**Meaning:**

- The process has now been **fully created**.
- It has been **brought into main memory**.
- The area of memory where such processes sit is called the **Ready Queue** (a term we already saw in earlier lectures).

**Why is it called "Ready"?**
Because the process is **completely ready to execute** — there is nothing at all left to prepare. The **only** thing it lacks is the **CPU**.

- It is simply **waiting for the CPU**.
- The instant the CPU is allocated / the instant it gets scheduled, it will **immediately take off** and start running.

**Important consequence — multiple processes can be in Ready state simultaneously:**
In a **multiprogramming OS**, we deliberately keep **more than one process** in the ready queue at the same time. The reason is to **increase the degree of multiprogramming** (discussed in detail in Section 7).

**Formal definition:**
> **Ready:** The process is in memory, waiting to be assigned to a processor.

---

### 2.3 Running State

**Board notes:**
```
3. Running :-
   P1  ->  CPU allocate
```

**Meaning:**

- Finally, **one particular process has been picked from the ready queue and the CPU has been allocated to it**.
- Example: process `P1` has been given the CPU.
- Its **instructions are now actually being executed**.
- At this moment, the Process State field of `P1`'s PCB reads **`Running`**.

**Formal definition:**
> **Run:** Instructions are being executed, CPU is allocated.

---

### 2.4 Waiting State

**Board notes:**
```
4. Waiting :-
   Waiting I/O completion
```

**Meaning:**

- While a process is running, it very often hits an **I/O instruction** (read a file, read from keyboard, write to disk, etc.).
- I/O is extremely slow compared to the CPU, so the process **cannot continue** until the I/O finishes.
- Therefore the process is moved **out of the CPU** into the **Waiting state**.
- Here it does nothing except **wait for I/O completion**, so that afterwards it can continue from where it left off.

**Why this state must exist (intuition):** If the process stayed on the CPU while waiting for a slow device, the CPU would sit idle doing nothing. Instead we evict it, free the CPU, and let another process use it.

**Formal definition:**
> **Waiting:** Waiting for IO.

---

### 2.5 Terminated State

**Board notes:**
```
5. Terminated :-
   process finishes
```

**Meaning:**

- The instructor deliberately says: don't call it "Termination", call it **"Terminated"** — because it is a *completed* action; the process **has already terminated**.
- The process's **execution is finished**.
- The **existence of that particular job/process is no longer there in memory** — it is wiped out.
- Its **PCB entry is removed from the process table**.

**Formal definition:**
> **Terminated:** The process has finished execution. PCB entry removed from process table.

---

### 2.6 Quick Reference Table of the Five States

| # | State | What is happening | Where does the process live | PCB "Process State" field value |
|---|-------|-------------------|------------------------------|---------------------------------|
| 1 | **New** | Program is being converted into a process (process being created) | Being loaded from **secondary storage / disk** | `New` |
| 2 | **Ready** | Fully created, only the CPU is missing | **Main memory**, in the **Ready Queue** | `Ready` |
| 3 | **Running** | Instructions are actually executing; CPU allocated | On the **CPU** | `Running` |
| 4 | **Waiting** | Blocked until an I/O / event completes | **Waiting Queue** (in memory) | `Waiting` |
| 5 | **Terminated** | Execution over; PCB removed from process table | **Nowhere** — removed from memory | `Terminated` |

---

## 3. The Process State Transition Diagram

This is the single most important diagram of the lecture — the instructor explicitly said: **"burn this diagram into your mind."**

```
        ┌───────────┐
        │   DISK    │   (secondary storage — programs waiting to be executed)
        │  (pool)   │
        └─────┬─────┘
              │  program picked up by Job Scheduler (LTS)
              v
          ( NEW ) ──────── admitted ────────┐
                                            │
                                            v
                                       ( READY ) ◄──────── interrupt ──────── ( RUNNING ) ──── exit ──► ( TERMINATED )
                                          ▲   │                                   │
                                          │   └──── scheduler dispatch ───────────┘
                                          │                                       │
                                          │                                       │  I/O or event wait
                          I/O or event    │                                       v
                          completion      └──────────────────────────────── ( WAITING )
```

### 3.1 Reading Every Arrow

| From → To | Label on the arrow | What actually happens |
|-----------|--------------------|------------------------|
| **Disk → New** | (job scheduler picks it up) | The program is lying on the disk; the OS lifts it up and begins converting it into a process |
| **New → Ready** | `admitted` | Process creation is complete; the process is admitted into main memory and placed in the ready queue |
| **Ready → Running** | `scheduler dispatch` | The CPU Scheduler selects a process and the **Dispatcher** hands the CPU over to it |
| **Running → Waiting** | `I/O or event wait` | The running process issues an I/O instruction (or waits for some event) and gets blocked |
| **Waiting → Ready** | `I/O or event completion` | The I/O finished; the process is **not** put straight back on the CPU — it re-joins the **ready queue** |
| **Running → Ready** | `interrupt` | Its **time quantum expired** in a time-sharing system (a **software interrupt** is generated), so it is pushed back into the ready queue |
| **Running → Terminated** | `exit` | Execution completed; the process exits and is destroyed |

### 3.2 The Disk / Secondary Storage on the Left

The instructor drew a **cylinder (disk symbol) to the left of the New state**. The reasoning:

- Programs are **not** born in memory. They are sitting on the **disk**.
- Things are **picked up from the disk one by one** and brought forward.
- At the moment a program is being lifted off the disk and turned into a process, **it is in the New state**.

### 3.3 The `Running → Ready` (interrupt) Arrow — Explained

There are **two ways** a running process can leave the CPU **without finishing**:

1. **It goes off to do I/O** → moves to **Waiting** state.
2. **Its time quantum expires** (in a **time-sharing system**) → moves back to **Ready** state.

For case 2:
- This is handled **as an interrupt** — specifically a **software interrupt** (we discussed software interrupts in an earlier lecture, and this is exactly one of their uses).
- The interrupt effectively says: *"Hey P1, you were running here, your time is up — come back now."*
- Because of this interrupt, the process is sent **back into the ready queue**, not terminated and not blocked.

### 3.4 ⭐ Very Important Doubt (asked in the comments of a previous video)

> **Question:** "If a process is in the Running state, and then goes to Waiting because of I/O — once the I/O is done, does it go **directly back to Running**?"
>
> **Answer: NO.**

- After I/O completion, the process goes **first to the Ready Queue**.
- From there it is the **scheduler's responsibility** to decide **when** it gets its next turn.
- That decision depends on:
  - the process's **priority**, and
  - **which scheduling algorithm** the OS is using.
- So the process will get a chance **later**, not immediately.

**Why this matters:** There is **no arrow from Waiting directly to Running** in the diagram. Every entry into the Running state comes **only** from the Ready state, via `scheduler dispatch`. This is a classic exam/interview trap.

---

## 4. Full Dry Run / Worked Example (P1, P2, P3)

This is the example the instructor drew on top of the diagram. Follow it step by step.

**Initial situation:**
```
Ready Queue:  [ P1 | P2 | P3 ]
Running (CPU): empty
Waiting Queue: empty
```

**Step 1 — Schedule P3**

The scheduler picks **P3** and dispatches it to the CPU.

```
Ready Queue:  [ P1 | P2 ]      ← P3 is REMOVED from the ready queue
Running (CPU): P3
Waiting Queue: empty
```

> **Key point:** Once a process moves to the Running state, it is **removed from the ready queue**. A process is in exactly one place at a time.

**Step 2 — P3 requests I/O**

While running, P3 hits an I/O instruction, so it is moved to the **Waiting** state.

```
Ready Queue:  [ P1 | P2 ]
Running (CPU): (momentarily free)
Waiting Queue: [ P3 ]
```

Now the **CPU has become free** — and the CPU must never sit idle.

**Step 3 — P2 is scheduled almost instantly**

The very instant P3 left for I/O, **within a fraction of time**, the CPU Scheduler schedules **P2**.

```
Ready Queue:  [ P1 ]           ← P2 is REMOVED from the ready queue
Running (CPU): P2
Waiting Queue: [ P3 ]
```

> This "within a fraction of time" is precisely **why the CPU scheduler is called the *short-term* scheduler** (Section 6.3).

**Step 4 — P3 completes its I/O**

P3's I/O finishes. It does **not** go back to the CPU (P2 is using it). It **re-joins the ready queue**.

```
Ready Queue:  [ P1 | P3 ]
Running (CPU): P2
Waiting Queue: empty
```

P3 will get another turn on the CPU **later**, whenever the scheduling algorithm gives it one.

**Step 5 (extension used later in the lecture) — P2 also goes for I/O**

A little while later P2 also issues an I/O request:

```
Ready Queue:  [ P1 | P3 ]  →  P1 gets scheduled
Running (CPU): P1
Waiting Queue: [ P3 | P2 ]   ← more than one process can be waiting at once
```

This is exactly why the waiting area is also a **queue** — multiple processes can be blocked on I/O simultaneously.

---

## 5. Process Queues — The Three Queues

Since many processes can be in the same state at the same time, the OS maintains a **queue for each state group**.

### 5.1 Job Queue

**Board diagram:**
```
[ P1 | P2 | P3 ]   Job queue
        (New)
```

- Holds all processes that are in the **New state**.
- These are **present in secondary memory** (the disk pool) — they have not been loaded into main memory yet.
- The disk is described as a **very large pool** containing lots of **"ready to be executed" programs**.
- The module that picks a process out of this pool and loads it into memory is the **Job Scheduler**, also called the **Long-Term Scheduler (LTS)**.

**Summary-note form:**
> **Job Queue:** (i) Processes in new state. (ii) Present in secondary memory. (iii) **Job Scheduler (Long Term Scheduler, LTS)** picks process from the pool and loads them into memory for execution.

### 5.2 Ready Queue

- Holds all processes in the **Ready state**.
- These are **present in main memory**.
- The module that picks a process from here and gives it to the CPU (**Running state**) is the **CPU Scheduler**, also called the **Short-Term Scheduler (STS)**.

**Summary-note form:**
> **Ready Queue:** (i) Processes in Ready state. (ii) Present in main memory. (iii) **CPU Scheduler (Short-term scheduler)** picks process from ready queue and dispatch it to CPU.

### 5.3 Waiting Queue

*(The instructor initially forgot this one and came back to it in the summary — so pay attention, it is easy to miss.)*

- Holds all processes in the **Wait / Waiting state**.
- These are the processes that left the Running state to perform I/O.
- Since **more than one process can be doing I/O at the same time** (as in Step 5 above, where P2 and P3 were both blocked), these blocked processes also form a **queue**.

**Summary-note form:**
> **Waiting Queue:** (i) Processes in Wait state.

### 5.4 Mapping the Diagram to the Queues

| Region of the state diagram | What it really is |
|-----------------------------|-------------------|
| Left of `New` / the `New` box | **Job Queue** (in secondary storage) |
| The `Ready` box | **Ready Queue** (in main memory) |
| The `Running` box | Not a queue at all — **this is literally the CPU** (only one process at a time per CPU) |
| The `Waiting` box | **Waiting Queue** |

> **Note the asymmetry:** New, Ready and Waiting are *queues* (many processes). **Running is not a queue** — it is the CPU itself, and only one process occupies it at a time.

---

## 6. Schedulers

### 6.1 Job Scheduler = Long-Term Scheduler (LTS)

**Job:** Move processes from the **pool on the disk → New state → Ready Queue (main memory)**.

- The disk holds a huge pool of programs that are ready to be executed.
- The Job Scheduler comes, **picks one up**, **converts it into a process (New state)**, and then **puts it into the ready queue**.
- So the entire responsibility of *"getting a job from the disk into the ready queue"* belongs to the Job Scheduler.

**Board note:** `Job scheduler (LTS)`

### 6.2 CPU Scheduler = Short-Term Scheduler (STS)

**Job:** Move a process from the **Ready Queue → Running state (CPU)**.

- It picks a process from the ready queue **depending upon the scheduling algorithm and the priority** of the processes.
- It then **dispatches** that process to the CPU.
- The **decision of which process goes next** is entirely the CPU Scheduler's job.

**Board note:** `CPU scheduler (STS)`

### 6.3 ⭐ Why "Long-Term" and "Short-Term"? (The Frequency Argument)

This is the conceptual heart of the section. The names **long-term** and **short-term** have **nothing to do with how long the process runs**. They depend on the **frequency at which the scheduler itself runs** — i.e. **how much idle time there is between two invocations of that scheduler**.

#### The CPU Scheduler runs at very HIGH frequency (small gap) → Short-Term

**Reasoning through an example:**

1. Process `P1` is in the ready queue. The CPU scheduler schedules it → `P1` moves to Running.
2. After only **a few instruction cycles** (not even a few seconds), `P1` hits an I/O operation and leaves for I/O.
3. The **CPU is now completely free**.

Now ask: *what if the CPU scheduler were lazy?*

- Suppose after scheduling `P1` the CPU scheduler decided *"I'll come back and look at the ready queue after 1 minute."*
- Then for that entire minute the **CPU would sit idle**, doing absolutely nothing.
- And we already established in earlier lectures: **the CPU must never sit idle, no matter what.**

**Therefore:**
- The CPU scheduler's idle time / gap is kept as **small as possible**.
- It works at **millisecond-level differences** — extremely high frequency.
- Behaviourally: the moment it schedules a process, it **does not trust that process to keep the CPU busy**. It immediately goes back and keeps **checking "is the CPU free? is the CPU free?"**
- The instant it finds the CPU free, it picks another process (say `P3`) from the ready queue and puts it into the Running state.

**Hence: CPU Scheduler = Short-Term Scheduler.** Its inter-run gap is very short.

#### The Job Scheduler runs at LOW frequency (large gap) → Long-Term

- The Job Scheduler is comparatively **"lazy"** — its idle time between runs is **much larger**.
- Example given: it may check only **once every 1 minute** whether some new job needs to be put into the ready queue.

**Board annotation:**
```
t0        →  5 jobs
t0 + 1 min
```

Meaning:
- At time `t0` it picked up **5 jobs** and placed them into the ready queue.
- Then at `t0 + 1 minute` it comes back and checks: *"has any new job arrived at the door that I should push into the ready queue?"*

Because this **gap in between is large**, we call it the **Long-Term Scheduler**.

#### Side-by-side comparison

| Aspect | Job Scheduler (LTS) | CPU Scheduler (STS) |
|--------|---------------------|---------------------|
| Other name | Long-Term Scheduler | Short-Term Scheduler |
| Moves process from → to | Disk pool / New → **Ready Queue** | Ready Queue → **Running (CPU)** |
| Source location | Secondary memory | Main memory |
| Frequency of execution | **Low** (e.g. once a minute) | **Very high** (millisecond scale) |
| Idle time between invocations | **Large** | **Very small** |
| Selection basis | Which jobs to admit into memory | Scheduling algorithm + process priority |
| Controls degree of multiprogramming? | **Yes** | No |

---

## 7. Degree of Multiprogramming

**Definition (as given in the lecture):**
> The **degree of multiprogramming** is the **number of processes in memory** — i.e. how many processes can sit in the **ready queue at one time**.

**Example:** If only **5 processes** can be present in the ready queue at a time, then the **degree of multiprogramming = 5**.

### ⭐ Who Controls the Degree of Multiprogramming?

**Answer: the Job Scheduler, i.e. the Long-Term Scheduler (LTS).**

**Why (the reasoning):**
- The LTS is the one that **actually lifts jobs from the pool (secondary storage) and puts them into the ready queue**.
- Since it alone decides **how many** processes get admitted into memory, it alone decides **how many** processes are in memory at a time.
- Therefore it is the LTS that governs/manages the degree of multiprogramming.

> **Exam/interview question flagged by the instructor:** *"Who controls the degree of multiprogramming?"* → **LTS (Long-Term Scheduler).**

**Connection back to the Ready state:** Earlier we said that in a multiprogramming OS we keep more than one process in the ready queue *precisely so that the degree of multiprogramming increases*. This is the same idea seen from the other side.

---

## 8. Dispatcher

The instructor separates two things that are often confused:

- The **CPU Scheduler (STS)** = the module that **decides** *which* process should get the CPU next.
- The **Dispatcher** = the module that **actually hands over control of the CPU** to the process that STS selected.

**Formal definition:**
> **Dispatcher:** The module of the OS that gives control of the CPU to a process selected by the STS.

This is why the arrow in the state diagram from Ready → Running is labelled **"scheduler dispatch"** — the scheduler chooses, the dispatcher enables/performs the actual handover.

The instructor notes he mentioned the dispatcher explicitly so that **when you read the term elsewhere, it doesn't feel unfamiliar**.

---

## 9. Consolidated Summary (the lecture's own closing notes)

### 1. Process States
As a process executes, it changes state. Each process may be in one of the following states:

- **a. New:** OS is about to pick the program & convert it into process. OR the process is being created.
- **b. Run:** Instructions are being executed, CPU is allocated.
- **c. Waiting:** Waiting for IO.
- **d. Ready:** The process is in memory, waiting to be assigned to a processor.
- **e. Terminated:** The process has finished execution. PCB entry removed from process table.

*(Plus the state transition diagram — memorise it.)*

### 2. Process Queues

- **a. Job Queue**
  - i. Processes in new state.
  - ii. Present in secondary memory.
  - iii. **Job Scheduler (Long-Term Scheduler, LTS)** picks process from the pool and loads them into memory for execution.
- **b. Ready Queue**
  - i. Processes in Ready state.
  - ii. Present in main memory.
  - iii. **CPU Scheduler (Short-Term Scheduler)** picks process from ready queue and dispatches it to CPU.
- **c. Waiting Queue**
  - i. Processes in Wait state.

### 3. Degree of Multi-programming
The number of processes in the memory.
- **a.** **LTS controls the degree of multi-programming.**

### 4. Dispatcher
The module of OS that gives control of CPU to a process selected by STS.

---

## 10. Points Most Likely to Be Asked (Traps & Interview Angles)

1. **Waiting → Running is NOT a valid transition.** After I/O completes, a process goes to **Ready**, never straight to Running. Only `Ready → Running` (scheduler dispatch) enters the Running state.
2. **Running → Ready happens via an interrupt**, specifically a **software interrupt** caused by **time-quantum expiry** in a **time-sharing** system.
3. **Who controls the degree of multiprogramming?** → **LTS / Job Scheduler.**
4. **Difference between Scheduler and Dispatcher:** the **STS decides** which process runs next; the **Dispatcher actually gives the CPU** to that process.
5. **Why is STS "short-term"?** Not because processes are short — because **the scheduler runs very frequently (tiny idle gap)** so that the **CPU never sits idle**.
6. **Why is LTS "long-term"?** Because its **gap between two runs is large** (e.g. once a minute).
7. **New vs Ready:** In **New** the process is still **being created** and lives in **secondary memory**; in **Ready** it is **fully created**, sits in **main memory**, and lacks only the CPU.
8. **Running is not a queue** — it is the CPU; only one process at a time (per CPU).
9. **Terminated means the PCB entry is removed from the process table** — the process no longer exists in memory.
10. **A process exists in exactly one queue/state at a time** — when P3 is dispatched to Running, it is removed from the ready queue.
11. **The Process State field of the PCB** is the concrete place where all of this is recorded — this is the link back to Lecture 9.
