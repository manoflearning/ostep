# 6. Mechanism: Limited Direct Execution

The crux: how to efficiently virtualize the CPU with control?
- The OS must virtualize the CPU in an efficient manner while retaining control over the system.
- To do so, both hardware and operating-system support will be required.
- The OS will often use a judicious bit of hardware support in order to accomplish its work effectively.

## 6.1. Basic Technique: Limited Direct Execution

**Direct Execution Protocol**:
- Just run the program directly on the CPU
- When the OS wishes to start a program running:
    1. It creates a process entry for it in a process list,
    2. allocates some memory for it,
    3. loads the program code into memory (from disk),
    4. locates its entry point (i.e., the `main()` routine or something similar),
    5. jumps to the entry point, and starts running the user's code.
- This approach gives rise to a few problems in our quest to virtualize the CPU:
    - If we just run a program, how can the OS make sure the program doesn't do anything that we don't want it to do, while still running it efficiently?
    - When we are running a process, how does the operating system stop it from running and switch to another process, thus implementing the **time sharing** we require to virtualize the CPU?
- This is where we need the "limited" part: **Limited Direct Execution**!
    - Without limits on running programs, the OS wouldn't be in control of anything and thus would be "just a library".

## 6.2. Problem #1: Restricted Operations

The Crux: How to perform restricted operations?
- A process must be able to perform I/O and some other restricted operations, but without giving the process complete control over the system.
- How can the OS and hardware work together to do so?

The approach we take:
- **User Mode**:
    - Code that runs in user mode is restricted in what it can do, e.g., when running in user mode, a process can't issue I/O requests; doing so would result in the processor raising an exception; the OS would then likely kill the process.
- **Kernel Mode**:
    - The operating system (or kernel) runs in kernel mode.
    - Code that runs can do what it likes, including privileged operations such as issuing I/O requests and executing all types of restricted instructions.

**System Call**:
- What should a user process do when it wishes to perform some kind of privileged operation, such as reading from disk?
- To enable this, virtually all modern hardware provides the ability for user programs to perform a **system call**.

**Trap Instruction**:
- To execute a system call, a program must execute a special **trap** insturction.
- This instruction simultaneously jumps into the kernel and raises the privilege level to kernel mode.
- When finished, the OS calls a special **return-from-trap** instruction, which returns into the calling user program while simultaneously reducing the privilege level back to user mode.

When executing a trap, the hardware needs to make sure to save enough of the caller's registers in order to be able to return correctly when the OS issues the return-from-trap instruction.
- E.g., on x86, the processor will push the program counter, flags, and a few other registers onto a per-process **kernel stack**.

**Trap Table**
- How does the trap know which code to run inside the OS? The calling process can't specify an address to jump to. Thus the kernel must carefully control what code executes upon a trap.
- The kernel does so by setting up a **trap table** at boot time:
    - When the machine boots up, it does so in privileged (kernel) mode, and thus is free to configure machine hardware as need be.
    - One of the first things the OS thus does is to tell the hardware what code to run when certain exceptional events occur.
        - E.g., what code should run when a hard-disk interrupt takes place, when a keyboard interrupt occurs, or when a program makes a system call?
    - The OS informs the hardware of the locations of these **trap handlers**, usually with some kind of special instruction.
    - Once the hardware is informed, it remembers the location of these handlers until the machine is next rebooted, and thus the hardware knows what to do (i.e., what code to jump to) when system calls and other exceptional events take place.

**System-call Number**
- To specify the exact system call, a **system-call number** is usually assigned to each system call.
- The user code is responsible for placing the desired system-call number in a register of at a specified location on the stack.
- The OS, when handling the system call inside the trap handler, examines this number, ensures it is valid, and if it is, executes the corresponding code.
- This level of indirection serves as a form of **protection**; user code cannot specify an exact address to jump to, but rather must request a particular service via number.

## 6.3. Problem #2: Switching Between Processes

The Crux: How to Regain Control of the CPU
- How can the operating system **regain control** of the CPU so that it can switch between processes?

### A Cooperative Approach: Wait For System Calls

The OS trusts the processes of the system to behave reasonably.
Processes that run for too long are assumed to periodically give up the CPU so that the OS can decide to run some other task.

Systems like this often include an explicit **yield** system call, which does nothing except to transfer control to the OS so it can run other processes.

Applications also transfer control to the OS when they do something illegal, e.g., divides by zero, or tries to access memory that it shouldn't be able to access.

### A Non-Cooperative Approach: The OS Takes Control

Without some additional help from the hardware, the OS can't do much at all when a process refuses to make system calls (or mistakes).

The crux: how to gain control without cooperation
- How can the OS gain control of the CPU even if processes are not being cooperative?
- What can the OS do to ensure a rogue process does not take over the machine?

**Timer Interrupt**:
- A timer device can be programmed to raise an interrupt every so many milliseconds; when the interrupt is raised, the currently running process is halted, and a pre-configured **interrupt handler** in the OS runs.
- At boot time, the OS informs the hardware of which code to run when the timer interrupt occurs.
- Second, also during the boot sequence, the OS must start the timer, which is a privileged operation.
- The timer can also be turned off (also a privileged operation).
- The hardware has some responsibility when an interrupt occurs, in particular to save enough of the state of the program that was running when the interrupt occurred such that a subsequent return-from-trap instruction will be able to resume the running program correctly.
    - Similar to the behavior of the hardware during an explicit system-call trap into the kernel.

### Saving and Restoring Context

- To save the context of the currently-running process, the OS will execute some low-level assembly code to save the general purpose registers, PC, and the kernel stack pointer of the currently-running process, and then restore said registers, PC, and switch to the kernel stack for the soon-to-be-executing process.
- By switching stacks, the kernel enters to the call to the switch doe in the context of one process and returns in the context of another.

There are two types of register saves/restores that happen during this protocol:
- When the timer interrupt occurs; in this case, the user registers of the running process are implicitly saved by the hardware, using the kernel stack of that process.
- When the OS decides to switch from A to B; in this case, the kernel registers are explicitly saved by the software (i.e., the OS), but this time into memory in the process structure of the process.

## 6.4. Worried About Concurrency?

Problem:
- What happens when, during a system call, a timer interrupt occurs?
- What happens when you're handling one interrupt and another one happens?

Some basics of how the OS handles these tricky situations:
- **Disable interrupts** during interrupt processing; doing so ensures that when one interrupt is being handled, no other one will be delivered to the CPU.
- Sophisticated **locking** schemes to protect concurrent access to internal data structures.
