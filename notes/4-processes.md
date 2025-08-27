# 4. The Abstraction: The Process

The crux of the problem:
- How to provide the illusion of many CPUs?
- Although there are only a few physical CPUs available, how can the OS provide the illusion of a nearly-endless supply of said CPUs?

**Time Sharing**:
- The OS creates this illusion by **virtualizing** the CPU.
- By running one process, then stopping it and running another, and so forth, the OS can promote the illusion that many virtual CPUs exist when in fact there is only one physical CPU (or a few).

To implement virtualization of the CPU (and to implement it well), the OS will need both:
- **mechanisms**:
    - low-level machinery
    - low-level methods or protocols that implement a needed piece of functionality
    - e.g., context switch
- **Policies**:
    - Algorithms for makign some kind of a decision within the OS
        - e.g., scheduling policy: given a number of possible programs to run on a CPU, which program should the OS run?

## 4.1. The Abstraction: A Process

**Process**:
- the abstraction provided by the OS of a running program
- at any instant in time, we can summarize a process by taking an inventory of the different pieces of the system it accesses or affects during the course of its execution

**Machine state**:
- what a program can read or update when it is running
- Memory:
    - instructions lie in memory
    - the data that the running program reads and writes sits in memory
    - Thus, the memory that the process can address (called its **address space**) is part of the process
- Registers:
    - Many instructions explicitly read or update registers and thus clearly they are important to the execution of the process
    - special registers that form part of this machine state:
        - **program counter (PC)**: tells us which instructino of the program will execute next
        - **stack pointer** and associated **frame pointer**: used to manage the stack for function parameters, local variables, and return addresses.
- Programs often access persistent storage devices too. Such I/O informatino might include a list of the files the process currently has open.

## 4.2. Process API

- **Create**: An operating system must include some method to create new processes.When you type a command into the shell, or double-click on an application icon, the OS is invoked to create a new process to run the program you have indicated.
- **Destroy**: Systems also provide an interface to destroy processes forcefully. Of course, many processes will run and just exit by themselves when complete; when they don't, however, the user may wish to kill them, and thus an interface to halt a runaway process is quite useful.
- **Wait**: Wait for a process to stop running.
- **Miscellaneous Control**: e.g., suspend a process and then resume it.
- **Status**: Get some status information about a process, such as how long it has run for, or what state it is in.

## 4.3. Process Creation: A Little More Detail

How programs are transformed into processes?

First thing that the OS must do to run a program is to **load** its code and any static data into memory, into the address space of the process.

In early operating systems, the loading process is done **eagarly**; modern OSes perform the process **lazily**.
To truly understand how lazy loading of pieces of code and data works, you'll have to understand more about the machinery of **paging** and **swapping**.

Once the code and static data are loaded into memory, there are a few other things the OS needs to do before running the process:
- Some memory must be allocated for the program's **run-time stack** (or just **stack**)

The OS may also allocate some memory for the program's **heap**. In C programs, the heap is used for explicitly requested dynamically-allocated data; programs request such space by calling `malloc()` and free it explicitly by calling `free()`.
The heap will be small at first; as the program runs, and requests more memory via the `malloc()` library API, the OS may get involved and allocate more memory to the process to help satisfy such calls.

The OS will also do some other initialization tasks, particularly as related to input/output (I/O).

One last task: to start the program running at the entry point, namely `main()`. By jumping to the `main()` routine, the OS transfers control of the CPU to the newly-created process, and thus the program begins its execution.

## 4.4. Process States

In a simplified view, a process can be in one of three states:
- **Running**: A process is running on a processor, means it is executing instructions.
- **Ready**: A process is ready to run but for some reason the OS has chosen not to run at this given moment.
- **Blocked**: A process has performed some kind of operation that makes it not ready to run until some other event takes place. E.g., when a process initiates an I/O request to a disk, it becoms blocked and thus some other process can use the processor.

A process can be moved between the ready and running states at the discretion of the OS.
- The process has been **scheduled**: being moved from ready to running
- The process has been **descheduled**: being moved from running to ready
- Once a process has become **blocked**, the OS will keep it as such until some event occurs; at that point, the process moves to the ready state again

## 4.5. Data Structures

The OS has some key data structures that track various relevant pieces of information.
- To track the state of each process, the OS likely will keep some kind of **process list** for all processes that are ready and some additional information to track which process is currently running.
- The OS must also track blocked processes; when an I/O event completes, the OS should make sure to wake the correct process and ready it to run again.

Example of what type of information an OS needs to track about each process in the xv6 kernel. (NOTE: xv6 is a small, UNIX-style kernel designed for educational purposes)
```cpp
// the registers xv6 will save and restore
// to stop and subsequently restart a process
struct context {
    int eip;
    int esp;
    int ebx;
    int ecx;
    int edx;
    int esi;
    int edi;
    int ebp;
};

// the different states a process can be in
enum proc_state { UNUSED, EMBRYO, SLEEPING, RUNNABLE, RUNNING, ZOMBIE };

// the information xv6 tracks about each process
// including its register context and state
struct proc {
    char *mem;                  // Start of process memory
    uint sz;                    // Size of process memory
    char *kstack;               // Bottom of kernel stack for this process
    enum proc_state state;      // Process state
    int pid;                    // Process ID
    struct proc *parent;        // Parent process
    void *chan;                 // If !zero, sleeping on chan
    int killed;                 // if !zero, has been killed
    struct file *ofile[NOFILE]; // Open files
    struct inode *cwd;          // Current directory
    struct context context;     // Switch here to run process
    struct trapframe *tf;       // Trap frame for the current interrupt
};
```

The **register context** will hold, for a stopped process, the contents of its registers.
When a process is stopped, its registers will be saved to this memory location; by restoring these registers (i.e., placing their values back into the actual physical registers), the OS can resume running the process.

More states a process can be in, beyond running, ready and blocked:
- **initial**: when it is being created
- **final**: it has exited but has not yet been cleaned up (in UNIX-based systems, this is called the **zombie** state)
    - This final state can be useful as it allows other processes to examine the return code of the process and see if the just-finished process executed successfully.
    - When finished, the parent will make one final call (e.g., `wait()`) to wait for the completion of the child, and to also indicate to the OS that it can clean up any relevant data structures that referred to the now-extinct process.
