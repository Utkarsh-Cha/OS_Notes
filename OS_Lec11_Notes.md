# Operating Systems — Lecture 11
## Swapping | Context Switching | Orphan Process | Zombie Process

> **Module:** Process Management
> **Why this lecture matters:** All four topics here are frequently asked in interviews. In particular, the **difference between an orphan process and a zombie process** is a classic interview question. The lecture also demonstrates both of these *live on a terminal*, so you can see exactly how they are created and how they look.

**Topics covered, in order:**
1. Quick recap of schedulers (LTS & STS) + introduction of the **Medium Term Scheduler (MTS)**
2. **Swapping** (swap-out / swap-in)
3. **Context Switching** (formal treatment using PCB)
4. **Orphan Process** (theory + live terminal demo)
5. **Zombie Process** (theory + live terminal demo)

---

# PART 0 — Recap: Long Term & Short Term Schedulers

Before starting the new material, the instructor recaps what was taught in the previous lecture, because MTS only makes sense once you know where LTS and STS sit.

## 0.1 The two schedulers and their alternate names

| Scheduler | Abbreviation | Alternate name | Where it operates |
|---|---|---|---|
| Long Term Scheduler | LTS | **Job Scheduler** | Job Queue → Ready Queue |
| Short Term Scheduler | STS | **CPU Scheduler** | Ready Queue → CPU |

## 0.2 The logical diagram

```
   [  JOB QUEUE  ]  ──── LTS ────►  [  READY QUEUE  ]  ──── STS ────►  ( CPU )
    (lives in                                            (dispatching)
   SECONDARY STORAGE)
```

**Key placement fact:** The **job queue physically resides in secondary storage**. All the programs the user has launched sit there first. The **ready queue resides in main memory**. The LTS is the component that decides *which* jobs get promoted from secondary storage into main memory.

## 0.3 What exactly does the LTS do? (The "mix of processes" idea)

The user may have fired off a very large number of processes — P1, P2, P3, and so on. The LTS cannot admit all of them. Its important job is to select a **good mix of processes**.

**What does "mix" mean here?**
Processes broadly fall into two behavioural categories:

- **I/O-intensive processes** — spend most of their time doing input/output, rarely need the CPU for long.
- **CPU-intensive processes** — spend most of their time computing, hold the CPU for long bursts.

**Why is a mix required? (the reasoning — this is the important part)**

- If the LTS loads **only CPU-intensive processes** into the ready queue, every process wants the CPU for a long time. Processes at the back of the queue keep waiting → **chance of starvation increases**.
- If the LTS loads **only I/O-intensive processes**, everyone piles up on the I/O devices while the CPU sits idle → again **chance of starvation increases** (and the CPU is wasted).

So a *homogeneous* ready queue (all one type) is bad in both directions. The instructor's point: *"I don't want only one type of job, I want a mix."* A balanced mixture keeps both the CPU and the I/O subsystem busy simultaneously, which is what makes the ready queue a **good** ready queue.

**So:** building this mixture is the responsibility of the **job scheduler (LTS)**. Based on this criterion it chooses processes and places them into the ready queue.

## 0.4 What does the STS do?

The STS (CPU scheduler) simply **picks one process from the ready queue and dispatches it to the CPU** — the choice depends on the **scheduling algorithm** you have selected (FCFS, SJF, Round Robin, etc.). This act of handing the process to the CPU is called **dispatching**.

---

# PART 1 — Medium Term Scheduler (MTS) and Swapping

## 1.1 Where MTS came from

- **Classic operating systems had only two schedulers: LTS and STS.**
- The **MTS was introduced later** ("lately"). Today's modern systems do have an MTS.
- It sits **in between** the long-term and short-term scheduler — hence *medium* term.

The instructor's larger point about OS design: *"The interesting thing about OS that I really like is that optimization has been applied on every small thing."* LTS + STS were sufficient for a while, but designers ran into a real-life problem in practice, and instead of going back and re-doing the LTS's work, they inserted a **new scheduler in the middle** to solve it. MTS is that optimization.

## 1.2 The problem MTS solves — step by step

**Step 1 — Degree of multiprogramming goes up.**

> **Degree of multiprogramming** = the number of processes that can be present in the ready queue (i.e. how many processes are held in main memory at one time).

Who increased it? **The LTS.** The long-term scheduler kept admitting more and more processes into the ready queue.

**Step 2 — The CPU starts running them, and memory runs out.**

While executing, it turns out that **two or three of these processes are extremely memory-hungry**. Because they are consuming so much memory, **memory itself is being exhausted** — there is no longer enough space to hold and maintain the entire ready queue.

**Step 3 — Something has to be removed.**

The only way out is to **swap some processes out**, i.e. remove them from the ready queue / main memory to free space.

### The worked example used in the lecture

```
READY QUEUE:  P1, P2, P3, P4, ...

P1, P2  →  became very memory-intensive
P3, P4  →  no space left for them in the ready queue
```

- P1 and P2 begin doing very memory-intensive work.
- As a result **there is no room left for P3 and P4** to even stay in the ready queue.
- So we take P3 and P4 and **swap them out** into a separate area called the **swap space**.

```
[ READY QUEUE ]                                  [   SWAP SPACE   ]
  P1, P2, P3, P4    ──── Swap-out (P3, P4) ────►   (secondary storage)

  (later: P1 terminates,
   memory freed)     ◄──── Swap-in (P3, P4) ────
```

**Important detail — these are partially executed processes.**
P3 and P4 are obviously **not finished** (if they were finished they wouldn't be in the ready queue). They may be **half-executed**, waiting at some particular point of their **program counter**. So the MTS does not throw them away — it **picks up their entire current state and saves it into the swap space**, so execution can later resume exactly from where it left off.

**Where is the swap space?**
The swap space is generally located in **secondary storage** — i.e. your **SSD or hard disk**.

**Why secondary storage?**
As covered in the storage/memory hierarchy: secondary storage is **much larger** because it is **cheap** (its access speed is far slower than RAM, which is exactly why it costs so much less per unit). Since it is cheap, you get lots of space — enough to park swapped-out processes.

**Step 4 — Effect of swapping out.**
For the time being, you have **shrunk the ready queue and saved memory from being exhausted**. P1 and P2 can now do their memory-heavy work comfortably.

**Step 5 — Swap-in.**
Suppose P1 (the memory-hungry one) finishes and moves to the **terminate state**. Now a large amount of memory is free again. The MTS picks up P3 and P4 — whose full saved state is sitting in secondary storage — and **brings them back into the ready queue**. This is **swap-in**.

**Step 6 — Naming.**

> **Swap-out + Swap-in together = SWAPPING.**
> Both are performed by the **Medium Term Scheduler (MTS)**.

## 1.3 The real-life intuition (very useful for remembering this)

The instructor gives two everyday examples:

1. **On a computer:** you keep opening program after program after program. Eventually memory can't hold everything.
2. **On an Android/iOS phone:** open a lot of apps. In the recent-apps view you'll notice that the **recently opened apps** are still there and open instantly. But some apps that are **further back** in the list — they're visible in the list, but when you tap them they take a long time to reload.
   **Why?** *Because that app was not actually in memory any more. It had been moved to some other area.* That "other area" is exactly the **swap space** in secondary storage. Tapping it forces a swap-in, which takes time because it's a disk read.

This is precisely why swapped-out apps feel slow to resume — you are paying a secondary-storage access cost.

## 1.4 Who physically carries out swapping?

- Operations like **context switching and swapping have their own dedicated instructions written inside the kernel**.
- The **kernel** performs them.
- But the ultimate executor of *any* instruction is always the **CPU**. So the kernel's swapping code is itself executed by the CPU, and the CPU is what actually moves the processes out and in.

## 1.5 Swapping is a form of "context saving"

The instructor makes a nice conceptual link: when a process is swapped out, its **entire current context** (program counter, state, etc.) is written into the swap area. So swapping is *a type of context saving* — not the same thing as CPU context switching between two processes, but conceptually the same "save now, restore later" mechanism.

## 1.6 The official notes diagram (as shown in the PDF on screen)

```
                     +---------------------------------------+
                     |  Partially executed swapped-out       |
                     |  processes                            |
                     +---------------------------------------+
                       ^                                 |
                       | Swap-in                         | Swap-out
                       |                                 v
+--------------+    +--------------+    +-----+    +--------------+
|      I/O     |<---|  Ready queue |--->| CPU |--->|     end      |
+--------------+    +--------------+    +-----+    +--------------+
       |                                   |
       v                                   v
+--------------+                           |
| I/O waiting  |<--------------------------+
| queue        |
+--------------+
```

**How to read this diagram:**
- The **ready queue** feeds the **CPU**; a finished process exits at **end**.
- A process that needs I/O leaves the CPU, goes into the **I/O waiting queue**, does its **I/O**, and comes back to the ready queue.
- Separately, the **MTS** can pull processes **out of the ready queue** (swap-out) into the "**partially executed swapped-out processes**" store in secondary storage, and later push them **back in** (swap-in).

## 1.7 Swapping — the written PDF notes (verbatim points, explained)

> **1. Swapping**
> a. Time-sharing system may have medium term scheduler (MTS).
> b. Remove processes from memory to reduce degree of multi-programming.
> c. These removed processes can be reintroduced into memory, and its execution can be continued where it left off. This is called Swapping.
> d. Swap-out and swap-in is done by MTS.
> e. Swapping is necessary to improve process mix or because a change in memory requirements has overcommitted available memory, requiring memory to be freed up.
> f. Swapping is a mechanism in which a process can be swapped temporarily out of main memory (or move) to secondary storage (disk) and make that memory available to other processes. At some later time, the system swaps back the process from the secondary storage to main memory.

**Explanation of the two key phrases in point (e):**
- **"to improve process mix"** — links straight back to Section 0.3. If the ready queue has become badly balanced (too many of one type), swapping some processes out and others in restores a healthy CPU-bound / I/O-bound mixture.
- **"memory requirements has overcommitted available memory"** — this is the P1/P2 scenario: processes ended up demanding more memory than actually exists, so memory *must* be freed up, and swapping is the mechanism that does it.

Note point (b): swapping **reduces the degree of multiprogramming** — it is literally the inverse of what the LTS did.

---

# PART 2 — Context Switching

Context switching has been mentioned informally since around the 2nd or 3rd lecture of the series. It is being covered **formally now** because we have since studied **process attributes, process states, and the PCB** — and you cannot describe context switching precisely without the PCB.

## 2.1 The real-life analogy

You are sitting and **listening to songs / playing PUBG**. From behind, your father calls: *"The Swiggy guy is downstairs, go get the food."*

What do you have to do?

1. **Pause** the song you were on.
2. Turn off the phone, take out the earphones.
3. Whatever **context was in your head** — the task you were focused on — you pick it up and **save it somewhere**; you put the phone down on the table.
4. Go downstairs, collect the delivery, come up, hand it to your father. **That is Task B, and it is now complete.**
5. Come back, plug the earphones in again, and **return to exactly where the song had been interrupted** — you **restore your old context**.

That whole thing is **context switching**. We do it many times every single day in ordinary life.

## 2.2 The OS version — the setup

We have two processes, **P1** and **P2**. Each process has **attributes**, held in its **PCB (Process Control Block)**.

```
      P1                            P2
  +---------+                   +---------+
  |  PCB 1  |                   |  PCB 2  |
  |  PID    |                   |  PID    |
  |  PC     |  (program counter)|  PC     |
  |  Regist.|  (registers)      |  Regist.|
  |  State  |                   |  State  |
  |  FDs    |  (file handles)   |  FDs    |
  +---------+                   +---------+

                 ( CPU )
              [ SP ] [ CP ] ...   ← CPU registers

        PROCESS TABLE
        +------+--------+
        |  P1  |  PCB1  |
        |  P2  |  PCB2  |
        +------+--------+
```

**The main PCB fields used in this explanation:**

| Field | Meaning |
|---|---|
| **PID** | Process ID |
| **PC (Program Counter)** | Address of the next instruction to execute |
| **Registers** | Saved values of the CPU's registers |
| **State** | Current process state (running, ready, waiting, …) |
| **FDs / File handles** | Which file descriptors (and device descriptors) are open |

**Where does the PCB live?**
There is a **process table**, and **each entry of the process table is a PCB**. The PCB is a **data structure that describes everything about a process**.

**On the CPU side:** the CPU also has its **own registers** (SP, CP, and so on). Note that **different CPUs / different CPU architectures have different types of registers, and even a different count of registers.** This fact becomes important in Section 2.5.

## 2.3 When does a context switch happen?

The CPU must context-switch when the currently running process gives up the CPU. Two triggers were given:

1. **The process goes into the WAIT state** — i.e. it has gone off to do **I/O**.
2. **The process's time quantum expires** (in a **time-sharing system**).

In both cases the CPU will no longer be with this process — some other process is about to run — so the current process's context **must be saved first**, otherwise it can never resume correctly.

## 2.4 The mechanics — save then restore

Suppose **P1** is currently on the CPU.

### Step A — SAVE the context of P1

Assume P1's program was executing at some particular memory location; call that address **1**. The next instruction is at address **2**.

The CPU (executing kernel code) does the following:

1. **Program counter:** writes **2** into the PC field of **PCB1** — the address from which P1 must resume later.
2. **Registers:** takes the **current values of all CPU registers** and copies them into the **registers section of PCB1**. (How many registers this is depends entirely on the CPU architecture.)
3. **State:** updates P1's **state** field (it is no longer *running* — it becomes *ready* or *waiting*).
4. **FDs:** saves which **file descriptors / device descriptors** were open.
5. In short: **every piece of information making up P1's current context is stored into P1's corresponding PCB.**

### Step B — RESTORE the context of P2

Now suppose the scheduler chose **P2** to run next. P2 was also saved at some earlier point in time, so its PCB already holds a snapshot.

1. Take the **register values stored in PCB2** and **copy them back into the corresponding CPU registers**.
2. Take **P2's saved program counter** and tell the CPU: *"from here onwards is where you must execute."*
3. Restore all the **file descriptors / device descriptors** that were open for P2, handing them back to the CPU.

**Net result:** P1's context has been **saved**, P2's context has been **restored**. That entire process is **context switching**.

## 2.5 Who performs context switching?

**The kernel.** All such low-level OS work is done by the kernel — user space obviously cannot do it. (And as always, the kernel's instructions are physically executed by the CPU.)

## 2.6 Context switching is PURE OVERHEAD ⚠️ (important)

This is a heavily emphasised point, and it's a common interview line.

**The reasoning:**
- Context switching is itself a program — instructions written inside the kernel that enable context switching. **That program is also executed by the CPU.**
- Therefore, **while a context switch is in progress, the CPU is not executing any process from the ready queue.**
- From the user's perspective, **none of the user-defined processes sitting in the ready queue are making any progress.** No useful work is happening at that moment.
- The system is simply "switching", not "doing".

> **Conclusion: context switching is pure overhead, because at that point in time no useful work is being done.**

This is exactly what the official notes say: *"It is pure overhead, because the system does no useful work while switching."*

## 2.7 What does context-switch speed depend on?

How fast a switch from P1 to P2 completes — how many milliseconds, how many instruction cycles — depends on:

| Factor | Why it matters |
|---|---|
| **Register performance & number of registers** | Every register value has to be copied out and copied in. More registers, or slower registers, = slower switch. |
| **Memory speed** | The PCB save/restore is memory traffic. Faster RAM = faster switch. |
| **Machine architecture** | Different machines have different architectures, so the cost differs machine to machine. |

**The RAM example given:** as RAM evolved — **DDR2 → DDR3 → DDR4** — RAM speed kept increasing. As RAM speed increased, **context switching started taking less time**, and because context switching got faster, **the whole system became faster**.

> **Takeaway line:** *Context switching speed varies from machine to machine.*

## 2.8 Context Switching — the written PDF notes (verbatim, explained)

> **2. Context-Switching**
> a. Switching the CPU to another process requires performing a **state save** of the current process and a **state restore** of a different process.
> b. When this occurs, the **kernel saves the context of the old process in its PCB** and **loads the saved context of the new process** scheduled to run.
> c. It is **pure overhead**, because the system does no useful work while switching.
> d. **Speed varies from machine to machine**, depending on the memory speed, the number of registers that must be copied, etc.

Points (a) and (b) are the formal statement of Steps A and B in Section 2.4 — note the exact terminology: **state save** and **state restore**.

---

# PART 3 — Orphan Process

## 3.1 Prerequisite: `fork()` and the process tree

- Every process in an operating system is created using the **`fork()`** system call.
- Some **parent process calls `fork()`** and thereby **creates a child process**.
- Consequence: **in any operating system, every process is somebody's child.**

**Then who is the very first process?**
There must be one process that has no parent — the "only one of its kind". On a **Linux system** this is the **`init`** process:

```
        ( init )   PID = 1
       /    |    \     \
    (...)  (...)  (...)  (...)
```

- `init` is the **very first process of the OS**.
- Its **PID is 1**.
- After `init`, all further kernel processes and user processes are created, forming a **tree** structure.

## 3.2 Definition

> **Orphan process** = a process **whose parent has terminated**, but which is **itself still running**.

("Orphan" in plain English = someone who has no parents.)

## 3.3 How an orphan is created

```
 ( P1 ) Running  ──── fork() ────►  ( P2 ) Child, Running
    |                                     |
    v                                     |  P1 is gone → P2 has no parent
 exception → terminate                    v
                                    ( init ) PID = 1
                                          |
                                       ( P2 )   ← adopted
```

1. **P1** is a process in the **running** state.
2. P1 calls `fork()` and creates child **P2**, which is also running.
3. **P1 dies.** Two possible reasons were given:
   - P1's code hit an **exception**, so P1 **terminated**; or
   - the programmer **never put a `wait()` in P1**, so P1 simply exited before its child.

> **Programmer's responsibility (a norm, not an option):** if a program `fork()`s a child, the **parent should `wait()` for that child**. This is a simple, standard rule of writing correct code. Orphans are typically the result of badly written code where this rule was broken — *"some innocent programmers write it this way."*

4. Now **P2 is orphaned** — its parent no longer exists, and nobody knows who its parent is.

## 3.4 What does the OS do about it? (The clever part)

**The problem:** the OS *must* do process management — it needs to **track where every process is**. Processes are organised as a **tree** rooted at `init`. If a parent is deleted, that subtree gets **detached from the tree**, and the OS loses its handle on it.

**Normally, this should never happen** — every process *should* have a parent; there should be no program whose parent has simply vanished. But since careless programs exist, **the OS is intelligent and plays a trick:**

> **The OS re-links the orphan's parent pointer to the `init` process.**
> So P2, whose parent used to be P1 (now exited), now has **`init` as its parent**.

This keeps the process tree intact and keeps every process trackable.

**Design observation made by the instructor:** *"Tree implementation is used a lot inside the OS"* — and here we can see the tree structure showing up in process management too.

## 3.5 Written PDF notes (verbatim)

> **3. Orphan process**
> a. The process whose parent process has been terminated and it is still running.
> b. Orphan processes are adopted by `init` process.
> c. `init` is the first process of OS.

---

## 3.6 LIVE TERMINAL DEMO — creating an orphan process

The demo is done on a Mac terminal (shell is **zsh**) using a small bash script.

### Demo Part 1 — The NORMAL case (no orphan)

**The script `orphan.sh`:**

```bash
lakshaykumar@9192 Desktop % cat orphan.sh
#!/bin/bash
# sleep 200 &
sleep 200
```

- Line 1 (`#!/bin/bash`) is just the shebang, telling the system it's a bash script.
- Line 2 is **commented out for now** — ignore it.
- **Line 3, `sleep 200`, is the active line** — the script sleeps for 200 seconds.

**First, find the terminal's own PID:**

```bash
lakshaykumar@9192 Desktop % echo $$
9161
```

> `echo $$` prints the PID of the current shell. Here the terminal's PID = **9161**.

**Now run the script:**

```bash
lakshaykumar@9192 Desktop % bash orphan.sh
```

**In a second terminal, inspect with `ps -al`:**

```text
  UID   PID  PPID   ...  S  ...  CMD
  501  9161  9159   ...  S+ ...  zsh
  501  9185  9161   ...  S+ ...  bash orphan.sh
  501  9186  9185   ...  S+ ...  sleep 200
```

**Reading this output — the parent/child chain:**

| PID | PPID | What it is |
|---|---|---|
| **9161** | 9159 | `zsh` — the **terminal** that is open |
| **9185** | **9161** | `bash orphan.sh` — launched **by the terminal** |
| **9186** | **9185** | `sleep 200` — the sleep system call launched **by the script** |

So the chain is exactly:

```
Terminal (9161)  ──►  orphan.sh (9185)  ──►  sleep 200 (9186)
   parent of ↑          parent of ↑           (currently working)
```

**What will happen naturally?**
- After 200 seconds, `sleep 200` exits.
- Its parent `orphan.sh` — which is **waiting** for it — gets notified that the child has finished.
- `orphan.sh` then exits, and **its** parent (the terminal) is likewise notified.
- One by one, **all these entries disappear** from the process table. This is the normal, healthy flow. **The parent is waiting here.**

### Why does a parent wait for its child? (crucial concept, reused in Part 4)

Whenever a parent creates a child process, **the parent waits until the child exits**. Why?

- The parent wants to know **what the child returned** — i.e. whether the child executed successfully or not.
- The parent **reads the child's return / exit status**.
- After reading it, the parent knows how that child behaved, and **notes the exit status**.
- **Only once the exit status has been read is the child's entry deleted from the process table** — the logic being: *"yes, the parent now knows what happened to the child, so remove it from the process table."*

Hold on to this — it is the entire foundation of the zombie process.

**Ending the demo:** the instructor tries to `kill` the process and finds it has already terminated on its own. Checking again, no such processes remain — only **9161** (the terminal) is still there, since the terminal is still open.

### Demo Part 2 — Creating an ACTUAL orphan (using `&`)

Now the script is edited with `vim orphan.sh` to swap which line is active:

```bash
#!/bin/bash
sleep 200 &
# sleep 200
```

**What does the `&` (ampersand) do?**

> The `&` **initiates the command as a new, separate process** — the process gets **detached**. The bash script keeps running as before, but `sleep 200` now runs **detached, as its own separate background process.**

**Run it and check `ps -al`:**

```text
  UID   PID  PPID   ...  S  ...  CMD
  501  9161  9159   ...  S+ ...  zsh
  501  9228     1   ...  S  ...  sleep 200
```

**Two things to notice:**

1. **`bash orphan.sh` is gone from the list** — the script already exited. (You can also see on the terminal that **control has returned to the prompt** immediately, instead of blocking for 200 seconds.)
2. **`sleep 200` has PID 9228 and PPID = 1.**

> **PPID 1 is the PID of the `init` process.** This is the *proof* that the process has been orphaned and adopted by `init`.

**What happened, step by step:**

```
Terminal (9161)  ── fork() ──►  orphan.sh  ── fork() ──►  ( sleep 200 & )
  (grandparent)                     |                            |
                                    v                            v
                            exited without wait()        ORPHAN, PPID becomes 1
```

1. The **terminal** forked/launched **`orphan.sh`**. (Terminal is the parent; relative to `sleep`, it's the grandparent.)
2. `orphan.sh` called **`sleep 200 &`** — because of the ampersand, this became a **detached, brand-new process** (a `fork()`).
3. Crucially, **`orphan.sh` did NOT `wait()` for `sleep 200`.**
4. Result of not waiting: **`orphan.sh` exited**, while `sleep 200` was still running.
5. `sleep 200` is now an **orphan process**.
6. The OS immediately re-parented it: its **PPID entry became 1**, i.e. it was **handed over to the `init` process**.

> **This is the live proof of the theory:** parent gone + child still running → PPID becomes 1 → adopted by `init`.

---

# PART 4 — Zombie Process

## 4.1 Another name

> A **zombie process** is also called a **defunct process**.
> Some operating systems / kernels use the name **"zombie"** in their implementation; others use **"defunct"**. Both mean the same thing.

## 4.2 Definition

> A **zombie process** is a process **whose execution is completed**, but which **still has an entry in the process table**.

## 4.3 How a zombie is created

```
   ( P1 ) Parent
      |
    fork()
      |
      v
   ( P2 ) Child ──── executing ──── exit() ────► ( ZOMBIE )
      ^                                              |
      |                                              |
      +──────── parent calls wait() later ───────────+
                (reads exit status → entry removed)
```

**The normal, correct flow:**
1. **P1** forks **P2**. P1 = parent, P2 = child.
2. The child executes.
3. The parent **waits** for the child: *"you exit first, and when you exit I will read your exit status."*
4. The mechanism used to read the exit status is the **`wait()` system call**.
5. Parent calls `wait()` → exit status is read → parent learns the child has exited → **parent can then exit too**.
6. Once both of these are done properly, **the child's entry is deleted from the process table.**

**Where the zombie comes from — the timing mismatch:**

Suppose the parent's `wait()` is set up such that it effectively waits for a **long time** — say the parent waits **5 minutes** — but the **child finishes and exits after only 2 minutes**.

> **For that 3-minute gap (between minute 2 and minute 5), P2 is a ZOMBIE process.**

The child is dead, but nobody has collected its exit status yet, so its process-table entry cannot be removed.

## 4.4 Why is it called a zombie? (the reasoning)

This is the "dead but not gone" intuition, and it's worth understanding precisely:

- When a process calls **`exit()`**, **all the resources it had requested/blocked from the OS are released.** Memory, files, devices — freed.
- So the resources are gone. **But the entry for that child inside the process table is still lying there.**
- The process is therefore **effectively dead** (no resources, no execution) but **still listed as existing**. Dead, yet present — a zombie.

*(You can literally see this in the demo output: zombie rows show **SZ = 0 and RSS = 0** — zero memory — confirming that the resources really were released and only the table entry remains.)*

## 4.5 Why zombies are a problem ⚠️

**The process table can fill up.**

- Every OS has a **limit on the size of the process table** — a maximum number of entries.
- If a lot of zombie processes accumulate, the **process table gets exhausted**.
- Once exhausted, **no new process can be created**. There is no free PID / no free slot in the process table, so that PID space simply cannot be used.

> **The process table is itself a resource.** So a build-up of zombies is a **resource leak** — you are leaking process-table entries.

This is a **badly problematic situation** for the system.

### 📌 Homework given in the lecture

> **H.W:** For different operating systems (**Linux, Windows**, etc.), find out **the maximum number of entries the process table can hold**. Google it and post the answer in the comment section.

## 4.6 Reaping a zombie

> **Reaping** = removing the zombie's entry from the process table.

How it happens:
1. A zombie's entry stays in the process table **until the parent calls `wait()` and reads its exit status**.
2. Once the parent reads the exit status (in our example, at the 5-minute mark), the **zombie's entry is removed from the process table**.
3. After that the parent can exit as well, and so on.

> **This whole act — parent comes, calls `wait()`, reads the exit status, entry deleted — is called REAPING OF THE ZOMBIE PROCESS.**

## 4.7 Two failure scenarios (edge cases — interview-relevant)

### Case A — The parent never calls `wait()` at all

Somebody wrote such bad code that `wait()` is never called.

- **No proper resource leak of the ordinary resources** — those were already released at `exit()`.
- **But the process table will get exhausted.** That is the very first problem.

### Case B — The parent itself exits

Forget `wait()` — the parent process has itself exited, so it isn't even *capable* of waiting.

- The instructor's framing: in that case, **"there is some bug in the operating system."**
- **Lots of zombie processes will keep accumulating**, and after a point, the same outcome: **the process table fills up**.
- Since the process table is itself a resource, this is a **resource leak**.

## 4.8 Written PDF notes (verbatim, with explanation)

> **4. Zombie process / Defunct process**
> a. A zombie process is a process whose execution is completed but it still has an entry in the process table.
> b. Zombie processes usually occur for **child processes**, as the parent process still needs to read its child's exit status. Once this is done using the **`wait` system call**, the zombie process is eliminated from the process table. This is known as **reaping the zombie process**.
> c. It is because parent process may call `wait()` on child process for a **longer time duration** and child process got **terminated much earlier**.
> d. As entry in the process table can only be removed **after the parent process reads the exit status of child process**, hence the child process remains a zombie till it is removed from the process table.

Note point (b): zombies **normally happen to child processes** — exactly as in the demo, where the zombies were the `sleep` children of the bash script.

---

## 4.9 LIVE TERMINAL DEMO — creating zombie processes

### The script `zombie.sh`

```bash
lakshaykumar@9192 Desktop % cat zombie.sh
#!/bin/bash
for i in {1..100}
do
    sleep 1&
done
exec sleep 100
```

### Line-by-line logic

**`for i in {1..100} … do sleep 1& … done`**
- Fires off **100 processes**, each of which just does `sleep 1` (runs for 1 second, then exits).
- The **`&`** makes each one a **detached background ("silent") process**.

**Why is the `&` essential here?**
> Without the ampersand, the loop would be **sequential**: run `sleep 1`, **wait for it to exit**, then start the next `sleep 1`, wait for that to exit, and so on — one after another.
> With the ampersand, the loop **fires all 100 sleeps rapidly as separate new processes** without waiting for any of them. That's how you get 100 children finishing almost simultaneously with nobody collecting their statuses.

**`exec sleep 100`**
- This **holds the same bash script busy for another 100 seconds**.
- **What `exec` does:** it **replaces** the current process's code with new code, **using the same PID**.
- Consequence you will literally see in `ps`: the script no longer shows up as `bash zombie.sh` — it shows up as **`sleep 100`**, but it is still **the same process, same PID**. *"That `sleep 100` is nothing but our particular bash script itself."*

### Step 1 — Confirm there are no zombies beforehand

```bash
ps -elf | grep Z
```

Output:

```text
  501  9568  9170  ... S+ ... grep Z
```

- **No zombie process is visible.** The single line shown is just the **`grep Z` command itself** appearing in the process list — not an actual zombie.

### Step 2 — Run the script and look again

```bash
bash zombie.sh
```

Then check with `ps -al`:

```text
  501  9571  9569  ...  0  0 ... Z+ ... (sleep)
  501  9572  9569  ...  0  0 ... Z+ ... (sleep)
  ...  (many more identical rows)
```

**What to observe in this output:**

| Observation | Meaning |
|---|---|
| **State column shows `Z+`** | `Z` = **zombie**. All these `sleep` processes are now zombies. |
| **CMD shown as `(sleep)` in parentheses** | The parenthesised form indicates a defunct/dead process. |
| **All of them share the same PPID = 9569** | One single parent — the bash script — spawned all of them. |
| **SZ = 0 and RSS = 0** | Zero memory held → all resources already released at `exit()`. Only the table entry survives. |

The parent (PID **9569**) appears in `ps` as **`sleep 100`**, launched by the terminal (PID **9161**), because of the `exec` replacement.

### Step 3 — Explanation of what happened

```
Terminal (9161)
     |
     ├─ fork ─►  bash zombie.sh  (PID 9569)
     |                 |
     |                 ├─ fork ─► sleep 1  ┐  each runs 1 second,
     |                 ├─ fork ─► sleep 1  ├─ then exit()  →  become ZOMBIES
     |                 ├─ ... (×100)       ┘  (entry stays in process table)
     |                 |
     |                 └─ exec ─► becomes "sleep 100", same PID 9569, RUNNING
```

1. The **terminal** launched **`zombie.sh`**.
2. `zombie.sh` did **two things**:
   - **First**, it created **many `sleep 1` child processes** — each of which ran for just **1 second and then exited**.
   - **Then**, it used **`exec`** to replace its own code with **`sleep 100`**, so the same process is now sitting in the **running** state for 100 seconds.
3. By the time we look, **that 1 second has long passed** for every child. All those children have already **called `exit()`** and are now just sitting there.
4. **Their entries are still in the process table.** Why? **Because the parent process — which is currently stuck in `sleep 100` — has not come along and read their exit statuses.** They are all waiting for the parent to call `wait()` and note their exit statuses so the process table entries can be removed.
5. Therefore, for this window of time, **they are zombie processes**.

> **Think of the `ps -al` output on your screen as the process table itself** — those `Z+` rows are the leftover entries.

### Step 4 — After 100 seconds

Once the parent finishes / reads the exit statuses of all the children it had spawned:

- Running `ps -al` again shows **only a couple of entries** — essentially just the **terminal's own entry**.
- **All the `Z+` sleep zombie entries have disappeared.**

> **That disappearance is reaping** — the parent came, read the exit status of every child it had forked, and all those zombie entries were removed from the process table.

**Summary of what the demo proved:** zombies exist **only for the interval between the child's `exit()` and the parent's reading of its exit status** — exactly the "5-minute vs 2-minute" gap from the theory.

---

# PART 5 — Consolidated Reference

## 5.1 Orphan vs Zombie (the classic interview comparison)

| | **Orphan Process** | **Zombie / Defunct Process** |
|---|---|---|
| **Definition** | Parent has **terminated**, but the process is **still running** | Process has **completed execution (exited)** but its **entry is still in the process table** |
| **Who is dead?** | The **parent** is dead | The **child** is dead |
| **Who is alive?** | The **child** is alive and running | The **parent** is alive (but hasn't read the exit status yet) |
| **Cause** | Parent exited / hit an exception without `wait()`ing for the child | Parent hasn't yet called `wait()` to read the child's exit status (e.g. it waits much longer than the child lives) |
| **Resources held** | It is a live process — holds normal resources | Resources **already released** on `exit()`; only the **process-table entry** remains |
| **How it is resolved** | **Adopted by `init`** (PPID becomes 1) | **Reaped** — parent calls `wait()`, reads exit status, entry deleted |
| **Danger** | Loss of tracking in the process tree (solved by `init` adoption) | **Process table exhaustion → resource leak → no new process can be created** |
| **How to spot it** | `ps -al` → **PPID = 1** | `ps -al` → state column = **`Z` / `Z+`**, CMD in parentheses, SZ/RSS = 0 |

## 5.2 All three schedulers at a glance

| | **LTS** | **STS** | **MTS** |
|---|---|---|---|
| **Also called** | Job Scheduler | CPU Scheduler | — |
| **Moves processes** | Job queue → Ready queue | Ready queue → CPU | Ready queue ↔ Swap space |
| **Key job** | Select a good **mix** of CPU-bound and I/O-bound processes | Pick the next process to run, per the scheduling algorithm; **dispatch** it | **Swapping**: swap-out / swap-in |
| **Effect on degree of multiprogramming** | **Increases** it | No direct effect | **Decreases** it |
| **Availability** | Classic + modern OS | Classic + modern OS | Introduced **later**; present in modern systems |

## 5.3 Commands and shell features used in the demos

| Command / feature | What it does |
|---|---|
| `echo $$` | Prints the **PID of the current shell/terminal** |
| `cat file.sh` | Displays the contents of the script |
| `vim file.sh` | Edits the script |
| `bash file.sh` | Runs the script (creates a child process of the terminal) |
| `ps -al` | Lists processes with **UID, PID, PPID, state (S), and CMD** |
| `ps -elf \| grep Z` | Filters the process list looking for zombie (`Z`) entries |
| `kill <pid>` | Terminates a process |
| `command &` | Runs the command **detached, as a separate background process** — the parent does **not** wait for it |
| `exec command` | **Replaces the current process's code** with the new command, **keeping the same PID** |
| `sleep N` | Sleeps for N seconds (used to keep processes alive for observation) |

**Process state letters seen in `ps` output:**

| Letter | Meaning |
|---|---|
| `S` | Sleeping |
| `S+` | Sleeping, in the **foreground** process group |
| `Z` / `Z+` | **Zombie** |

## 5.4 Final recap (as the instructor summarised at the end)

1. **Swapping** — performed by the **MTS (Medium Term Scheduler)**; swap-out removes partially executed processes to secondary storage, swap-in brings them back, execution continues from where it left off.
2. **Context switching** — save the state of the current process into its PCB, restore the state of the next process from its PCB; done by the **kernel**; it is **pure overhead**; speed varies machine to machine.
3. **Orphan process** — a process **still running** whose **parent has terminated**; it is **adopted by the `init` process**, which is the **first process of the OS** (PID 1).
4. **Zombie process (a.k.a. defunct process)** — a process **whose execution has completed but which still has an entry in the process table**. Normally these are **child processes**. When the **parent reads the exit status** (via `wait()`), the entry is removed from the process table — this is **reaping of the zombie process**.

> **Instructor's closing emphasis:** *orphan process and zombie process are very important topics — definitely keep them in mind.*
