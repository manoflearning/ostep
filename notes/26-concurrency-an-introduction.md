# 26. Concurrency: An Introduction

- **Virtual CPU**: enabling the illusion of multiple programs running at the same time.
- **Virtual Memory**: enalbes each program to behave as if it has its own memory when indeed the OS secretly multiplexing address spaces across physical memory.

**Thread**:
- A abstraction for a single running process.
- A **multi threaded** program has more than one point of execution (i.e., multiple PCs, each of which is being fetched and executed from).
    - Each thread is very much like a separate process, except for one difference: they share the same address space and thus can access the same data.
- The state of a single thread:
    - It has a program counter (PC) that tracks where the program is fetching instructions from.
    - Each thread has its own private set of registers it uses for computation; thus, if there are two threads that are running on a single processor, a **context switch** must take place.
        - Which processes, we saved state to a **process control block (PCB)**
        - Which threads, we saved state to a **thread control block (TCB)**
        - Major difference when switch the context (versus process): the address space remains the same.
- In multi-threaded process, we need multiple stacks (single stack for each thread).

## 26.1. Why Use Threads?

Why should we use threads at all?
- First reason: **parallelism**
- Second reason: to avoid blocking program progress due to slow I/O
    - Threading enables **overlap** of I/O with other activites within a single program
- You could also use multiple processes instead of threads. But threads share an address space and thus make it easy to share data.

## 26.2. An Example: Thread Creation

SKIPPED

## 26.3. Why It Gets Worse: Shared Data

How the threads interact when they access shared data?

SKIPPED

## 26.4. The Heart Of The Problem: Uncontrolled Scheduling

SKIPPED

- **Race Condition**: The results depend on the timing of the code's execution.
- **Critical Section**: A piece of code that accesses a shared variable, and must not be concurrently executed by more than one thread.
- **Mutual Exclusion**: If one thread is executing within the critical section, the others will be prevented from doing so.

## 26.5. The Wish For Atomicity

One way to solve this problem would be to have more powerful instructions that, in a single step, did exactly whatever we needed done and thus removed the possibility of an untimely interrupt.

```nasm
memory-add 0x8049a1c, $0xl
```

The hardware guarantees that it executes **atomically**: either the instruction has not run at all, or it has run to completion.

But we couldn't have the single instruction for all cases.
Thus, what we will instead do is ask the hardware for a few useful instructions upon which we can build a general set of what we call **synchronization primitives**.

The crux: how to support synchronization?
- What support do we need from the hardware in order to build useful synchronization primitives?
- What support do we need from the OS?
- How can we build these primitives correctly and efficiently?
- How can programs use them to get the desired results?

## 26.6. One More Problem: Waiting For Another

Think of the case: one thread must wait for another to complete some action before it continues.
