# 📝 MY_WORK: Student Information, Development Log, Reflection & Answers

> This is the **only file** your instructor reads to grade Parts 3 and 4 (documentation and video). Everything you write here must be **in your own words**.

---

## 👤 Student Information

| Field | Your Answer |
|-------|-------------|
| **Full Name** | Dalia Abdullah Alhussain |
| **Student ID** | 445052058 |
| **University Email** | 445052058@std.psau.edu.sa |
| **GitHub Username** | daliaabdullah1 |
| **Repository Link** | https://github.com/daliaabdullah1/OS-Assignment3-Dalia-Abdullah |
 
---

## 🎥 Video Link

**Video Link**: [Paste your public Google Drive or YouTube Unlisted video link here]

---

# Part A: Development Log (0.5 mark)

## Your Development Log

### Entry 1 - [November 8, 2026, 5:15 PM]
**What I did**: Forked the repository, set my student ID, and ran the starter code.

**Details**:
- Forked the starter repository and renamed it to `OS-Assignment3-Dalia-Abdullah` (kept it public).
- Changed the student ID on line 304 of `SchedulerSimulationSync.java` to `445052058`.
- Ran the original starter code in VS Code to see how it fails before adding locks.
- Committed my changes: `Set my student ID: 445052058`.

**Challenges**: At first, I was surprised why both CPUs picked `P1` at the same time and caused an `IllegalThreadStateException`.

**Solution**: I checked `ReadyQueue.poll()` and saw `RaceWindow.pause()` between reading the first item and removing it, which let two CPUs grab the same thread.

**Time spent**: 40 minutes

---

### Entry 2 - [November 10, 2026, 7:00 PM]
**What I did**: Added a `ReentrantLock` to protect `ReadyQueue` (Feature 1).

**Details**:
- Imported `java.util.concurrent.locks.ReentrantLock`.
- Created `queueLock` inside `ReadyQueue` and wrapped `add()`, `poll()`, `processOf()`, and `snapshot()` with `lock()` and `try/finally`.
- Committed the update: `Feature 1: Protect ReadyQueue with ReentrantLock`.

**Challenges**: Making sure the lock is always unlocked even when `poll()` returns `null` if the queue is empty.

**Solution**: Put `queueLock.unlock()` inside a `finally` block so it always runs no matter how the method exits.

**Time spent**: 35 minutes

---

### Entry 3 - [November 12, 2026, 6:30 PM]
**What I did**: Protected the shared counters in `SharedStats` using a lock (Feature 2).

**Details**:
- Added `statsLock` to `SharedStats`.
- Protected all three update methods (`incrementContextSwitch`, `incrementCompletedProcess`, `addServiceTime`) and the three getter methods.
- Ran the code and saw that the context switches (25), completed processes (11), and service time (78695 ms) matched the expected numbers.
- Committed the update: `Feature 2: Protect SharedStats with ReentrantLock`.

**Challenges**: Deciding if I should use one lock for all three counters or one lock for each variable.

**Solution**: I used one lock (`statsLock`) because the updates are very fast and simple, so one lock keeps the code clean without slowing anything down.

**Time spent**: 30 minutes

---

### Entry 4 - [November 14, 2026, 8:20 PM]
**What I did**: Replaced busy waiting with a counting `Semaphore` (Feature 3).

**Details**:
- Added `Semaphore itemsAvailable = new Semaphore(0)` in `ReadyQueue`.
- Called `itemsAvailable.release()` inside `add()` whenever a process enters the queue.
- Replaced the `while` polling loop and `Thread.sleep(20)` in `CpuWorker` with `readyQueue.acquireProcess()`.
- Added `wakeAllCpus()` when the last process finishes so sleeping CPUs can exit cleanly.
- Committed the update: `Feature 3: Implement Semaphore for CPU synchronization`.

**Challenges**: After all 11 processes finished, one CPU exited while the second CPU stayed stuck waiting on `acquire()`, triggering the watchdog.

**Solution**: Made the CPU that finishes the last process call `release(totalCPUs)` so any waiting CPU wakes up, sees everything is done, and exits.

**Time spent**: 50 minutes

---

### Entry 5 - [November 16, 2026, 10:30 PM]
**What I did**: Tested the final code multiple times and completed `MY_WORK.md`.

**Details**:
- Ran the final code 5 times in a row in VS Code with my ID (`445052058`) and confirmed all 4 checks show `✔` and `Busy-waiting polls` is `0`.
- Wrote the development log, reflection, and technical answers in `MY_WORK.md`.

**Challenges**: Explaining the race condition timeline clearly in simple words using my own output numbers.

**Solution**: Traced step by step how `CPU-1` and `CPU-2` overlap inside `RaceWindow.pause()` and wrote it down clearly.

**Time spent**: 45 minutes

---

## Development Log Summary

**Total time spent on assignment**: 3 hours 20 minutes

**Most challenging part**: Making sure the waiting CPU thread wakes up and terminates cleanly when the last process finishes in Feature 3.

**Most interesting learning**: Seeing how a tiny 1ms pause (`RaceWindow.pause()`) completely breaks shared counters without locks, and how `ReentrantLock` fixes it immediately.

**What I would do differently next time**: I would save the terminal output after every commit in separate files so I can compare them faster.

---

# Part B: Reflection (0.5 mark)

## Question 1: What did you learn about synchronization?

**Your Answer:**
I learned how race conditions happen when multiple threads read and update shared data at the same time without protection. When I ran the starter code, `RaceWindow.pause()` showed how easily two CPUs can read the same value before either one saves its update, causing lost data and crashes. Using `ReentrantLock` taught me how mutual exclusion allows only one thread inside a critical section at a time. I also learned why putting `unlock()` inside a `finally` block is so important so a thread never leaves a lock stuck. Finally, using a `Semaphore` showed me how threads can sleep until work is ready instead of wasting CPU time checking an empty queue.

## Question 2: What was the most challenging part of this assignment?

**Your Answer:**
The hardest part for me was getting the CPUs to shut down properly after adding the counting semaphore in Feature 3. Before using the semaphore, the CPUs kept checking the queue in a loop, so they naturally stopped when `completedProcessCount` reached 11. Once I changed them to block on `itemsAvailable.acquire()`, one CPU finished the 11th process while the other CPU stayed asleep waiting for a permit that would never come. Fixing this required calling `release(totalCPUs)` when the last process finishes so all sleeping CPUs wake up and exit the loop cleanly.

## Question 3: How did you overcome the challenges you faced?

**Your Answer:**
I solved the problems by taking the assignment step by step and testing after every small change. First, I read `SchedulerSimulationSync.java` carefully to see which methods touched `ReadyQueue` and `SharedStats`. Whenever something went wrong, I traced the steps between `CpuWorker.run()` and `ReadyQueue.add()` to see how threads were locking and unlocking or waiting on the semaphore. Checking the verification lines at the bottom of the output after each feature made it easy to confirm that my fix worked before moving to the next step.

## Question 4: Where can synchronization concepts be applied in real-world applications?

**Your Answer:**
Synchronization is used everywhere in real apps that handle many users at the same time. For example, in banking apps, if two transfers happen on the same account at once, locks prevent the balance from being overwritten incorrectly. In flight or cinema booking websites, locks stop two people from booking the same seat at the exact same second. Also, in operating system print queues or web servers, semaphores help worker threads wait quietly when there are no tasks and wake up immediately when new requests arrive.

---

# Part C: Technical Answers (0.5 mark)

## Question 1: Race Condition in Your Own Output

**Your Answer:**
In the starter run with my student ID `445052058` (11 processes, 2 CPUs, 4000ms quantum), both CPUs accessed the queue and shared counters without locks, causing an `IllegalThreadStateException` and lost updates. On a real computer, the OS can switch between threads at any nanosecond between reading and writing a variable, even without `RaceWindow.pause()`. This bug is hard to catch because it depends on exact thread timing, so it happens randomly and might not show up in every single test run.

Verification block of the **original starter**:
