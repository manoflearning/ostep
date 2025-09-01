# 7. Scheduling: Introduction

The crux: how to develop scheduling polcy
- How should we develop a basic framework for thinking about scheduling policies?
- What are the key assumptions?
- What metrics are important?
- What basic approaches have been used in the earliest of computer systems?

## 7.1. Workload Assumptions

**Workload**:
- Number of simplifying assumptions about the processes running in the system.
- Determining the workload is a critical part of building policies, and the more you know about workload, the more fine-tuned your policy can be.

Will will make the following assumptions about the processes (or **jobs**) (initial simplified version):
1. Each job runs for the same amount of time.
2. All jobs arrive at the same time.
3. Once started, each job runs to completion.
4. All jobs only use the CPU (i.e., they perform no I/O)
5. The run-time of each job is known.

## 7.2. Scheduling Metrics

**Scheduling metric**:
- **turnaround time**:
    - $T_{\text{turnaround}} = T_{\text{completion}} - T_{\text{arrival}}$
    - Because we have assumed that all jobs arrive at the same time, for now $T_{\text{arrival}} = 0$ and hence $T_{\text{turnaround}} = T_{\text{completion}}$.
- Turnaround time is a **performance** metric. Another metric of interest is **fairness**.

## 7.3. First In, First Out (FIFO)

**First In, First Out** (or sometimes **First Come, First Served (FCFS)**):
- Positive properties: simple and easy to implement

Let's relax assumption 1, and thus no longer assume that each job runs for the same amount of time.
How does FIFO perform now?

**Convoy Effect**:
- A number of relatively-short potential consumers of a resource get queued behind a heavyweight resource consumer.
- Results in poor performance.

## 7.4. Shortest Job First (SJF)

**Shortest Job First (SJF)**:
- It runs the shortest job first, then the next shortest, and so on.
- With regards to average turnaround time, SJF performs better than FIFO.
- Given out assumptions about jobs all arriving at the same time, we could prove that SJF is indeed an **optimal** scheduling algorithm.

Let's relax assumption 2, now assume that jobs can arrive at any time instead of all at once.

We still suffer the same convoy problem!

## 7.5. Shortest Time-to-Completion First (STCF)

To address this concern, we need to relax assumption 3.

Now the OS can **preempt** certain job and decide to run another job.

**Shortest Time-to-Completion First (STCF)** (or **Prremptive Shortest Job First (PSJF)**):
- Any time a new job enters the system, the STCF scheduler determines which of the remaining jobs has the least tmie left, and schedules that one.

## 7.6. A New Metric: Response Time

If we knew job lengths, and that jobs only used the CPU, and our only metric was turnaround time, STCF would be a great policy.
However, the introduction of time-shared machines changed all that.
Now users would sit at a terminal and demand interactive performance from the system as well.
A new metric was born: **response time**

**Response Time**:
- $T_{\text{response}} = T_{\text{firstrun}} - T_{\text{arrival}}$
- STCF and related disciplines are not good for response time

## 7.7. Round Robin

**Round-Robin (RR)**
- RR runs a job for a **time slice** (sometimes called a **scheduling quantum**) and then switches to the next job in the run queue.
- The length of a time slice must be a multiple of the timper-interrupt period.
- The length of the time slice is critical for RR:
    - The shorter it is, the better the performance of RR under the response-time metric.
    - The shorter it is, the cost of context switching is increase.

The cost of context switching does not arise solely from the OS actions of saving and restoring a few registers: state in CPU caches, TLBs, branch predictors, and other on-chip hardware.

RR is indeed one of the worst policies if turnaround time is our metric.

More generally, any policy (such as RR) that is **fair**, i.e., that evenly divides the CPU among active processes on a small time scale, will perform poorly on metrics such as turnaround time.

We still have two assumptions which need to be relaxed:
- Assumption 4 (that jobs do no I/O)
- Assumption 5 (that the run-time of each job is known)

## 7.8. Incorporating I/O

First we will relax assumption 4.
A scheduler has a decision to make when a job initiates an I/O request, because the currently-running job won't be using the CPU during the I/O; it is **blocked** waiting for I/O completion.
If the I/O is sent to a hard disk drive, the process might be blocked for a few milliseconds or longer, depending on the current I/O load of the drive.

The scheduler also has to make a decision when the I/O completes.
When that occurs, an interrupt is raised, and the OS runs and moves the process that issued the I/O from blocked back to the ready state.

**Overlap**: when I/O occurs, run other process on the CPU (for the meantime).

## 7.9. No More Oracle

Final assumption: that the scheduler knows the length of each job.

In fact, in a general-purpose OS, the OS usually knows very little about the length of each job.
Thus, how can we build an approach that behaves like SJF/STCF without such a priori knowledge?
