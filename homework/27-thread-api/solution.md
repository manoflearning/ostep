## Problem 1

### Problem

First build main-race.c. Examine the code so you can see the (hopefully obvious) data race in the code. Now run helgrind (by `typing valgrind --tool=helgrind main-race`) to see how it reports the race. Does it point to the right lines of code? What other information does it give to you?

### Answer

```bash
$ valgrind --tool=helgrind ./main-race
==3772293== Helgrind, a thread error detector
==3772293== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==3772293== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3772293== Command: ./main-race
==3772293== 
==3772293== ---Thread-Announcement------------------------------------------
==3772293== 
==3772293== Thread #1 is the program's root thread
==3772293== 
==3772293== ---Thread-Announcement------------------------------------------
==3772293== 
==3772293== Thread #2 was created
==3772293==    at 0x49A2A23: clone (clone.S:76)
==3772293==    by 0x49A2BA2: __clone_internal_fallback (clone-internal.c:64)
==3772293==    by 0x49A2BA2: __clone_internal (clone-internal.c:109)
==3772293==    by 0x491554F: create_thread (pthread_create.c:297)
==3772293==    by 0x49161A4: pthread_create@@GLIBC_2.34 (pthread_create.c:836)
==3772293==    by 0x4854975: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772293==    by 0x109209: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772293== 
==3772293== ----------------------------------------------------------------
==3772293== 
==3772293== Possible data race during read of size 4 at 0x10C014 by thread #1
==3772293== Locks held: none
==3772293==    at 0x109236: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772293== 
==3772293== This conflicts with a previous write of size 4 by thread #2
==3772293== Locks held: none
==3772293==    at 0x1091BE: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772293==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772293==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3772293==    by 0x49A2A33: clone (clone.S:100)
==3772293==  Address 0x10c014 is 0 bytes inside data symbol "balance"
==3772293== 
==3772293== ----------------------------------------------------------------
==3772293== 
==3772293== Possible data race during write of size 4 at 0x10C014 by thread #1
==3772293== Locks held: none
==3772293==    at 0x10923F: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772293== 
==3772293== This conflicts with a previous write of size 4 by thread #2
==3772293== Locks held: none
==3772293==    at 0x1091BE: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772293==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772293==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3772293==    by 0x49A2A33: clone (clone.S:100)
==3772293==  Address 0x10c014 is 0 bytes inside data symbol "balance"
==3772293== 
==3772293== 
==3772293== Use --history-level=approx or =none to gain increased speed, at
==3772293== the cost of reduced accuracy of conflicting-access information
==3772293== For lists of detected and suppressed errors, rerun with: -s
==3772293== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

## Problem 2

### Problem

What happens when you remove one of the offending lines of code? Now add a lock around one of the updates to the shared variable, and then around both. What does helgrind report in each of these cases?

### Answer

Remove one of the offending lines of code:
```bash
$ valgrind --tool=helgrind ./main-race
==3772600== Helgrind, a thread error detector
==3772600== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==3772600== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3772600== Command: ./main-race
==3772600== 
==3772600== 
==3772600== Use --history-level=approx or =none to gain increased speed, at
==3772600== the cost of reduced accuracy of conflicting-access information
==3772600== For lists of detected and suppressed errors, rerun with: -s
==3772600== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

Add a lock around one of the updates to the shared variable:
```bash
$ valgrind --tool=helgrind ./main-race
==3772766== Helgrind, a thread error detector
==3772766== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==3772766== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3772766== Command: ./main-race
==3772766== 
==3772766== ---Thread-Announcement------------------------------------------
==3772766== 
==3772766== Thread #1 is the program's root thread
==3772766== 
==3772766== ---Thread-Announcement------------------------------------------
==3772766== 
==3772766== Thread #2 was created
==3772766==    at 0x49A2A23: clone (clone.S:76)
==3772766==    by 0x49A2BA2: __clone_internal_fallback (clone-internal.c:64)
==3772766==    by 0x49A2BA2: __clone_internal (clone-internal.c:109)
==3772766==    by 0x491554F: create_thread (pthread_create.c:297)
==3772766==    by 0x49161A4: pthread_create@@GLIBC_2.34 (pthread_create.c:836)
==3772766==    by 0x4854975: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772766==    by 0x10926B: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772766== 
==3772766== ----------------------------------------------------------------
==3772766== 
==3772766==  Lock at 0x10C060 was first observed
==3772766==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772766==    by 0x109207: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772766==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772766==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3772766==    by 0x49A2A33: clone (clone.S:100)
==3772766==  Address 0x10c060 is 0 bytes inside data symbol "lock"
==3772766== 
==3772766== Possible data race during read of size 4 at 0x10C040 by thread #1
==3772766== Locks held: none
==3772766==    at 0x109298: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772766== 
==3772766== This conflicts with a previous write of size 4 by thread #2
==3772766== Locks held: 1, at address 0x10C060
==3772766==    at 0x109211: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772766==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772766==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3772766==    by 0x49A2A33: clone (clone.S:100)
==3772766==  Address 0x10c040 is 0 bytes inside data symbol "balance"
==3772766== 
==3772766== ----------------------------------------------------------------
==3772766== 
==3772766==  Lock at 0x10C060 was first observed
==3772766==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772766==    by 0x109207: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772766==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772766==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3772766==    by 0x49A2A33: clone (clone.S:100)
==3772766==  Address 0x10c060 is 0 bytes inside data symbol "lock"
==3772766== 
==3772766== Possible data race during write of size 4 at 0x10C040 by thread #1
==3772766== Locks held: none
==3772766==    at 0x1092A1: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772766== 
==3772766== This conflicts with a previous write of size 4 by thread #2
==3772766== Locks held: 1, at address 0x10C060
==3772766==    at 0x109211: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-race)
==3772766==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3772766==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3772766==    by 0x49A2A33: clone (clone.S:100)
==3772766==  Address 0x10c040 is 0 bytes inside data symbol "balance"
==3772766== 
==3772766== 
==3772766== Use --history-level=approx or =none to gain increased speed, at
==3772766== the cost of reduced accuracy of conflicting-access information
==3772766== For lists of detected and suppressed errors, rerun with: -s
==3772766== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

Add a lock around the both:
```bash
$ valgrind --tool=helgrind ./main-race
==3772835== Helgrind, a thread error detector
==3772835== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==3772835== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3772835== Command: ./main-race
==3772835== 
==3772835== 
==3772835== Use --history-level=approx or =none to gain increased speed, at
==3772835== the cost of reduced accuracy of conflicting-access information
==3772835== For lists of detected and suppressed errors, rerun with: -s
==3772835== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 7 from 7)
```

## Problem 3

### Problem

Now let’s look at `main-deadlock.c`. Examine the code. This code has a problem known as deadlock (which we discuss in much more depth in a forthcoming chapter). Can you see what problem it might have?

### Answer

The deadlock occurs when one thread (probably `p1`) acquires the lock `m1`, and the other thread (probably `p2`) acquires the lock `m2`.

## Problem 4

### Problem

Now run helgrind on this code. What does helgrind report?

### Answer

```bash
$ valgrind --tool=helgrind ./main-deadlock
==3773091== Helgrind, a thread error detector
==3773091== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==3773091== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3773091== Command: ./main-deadlock
==3773091== 
==3773091== ---Thread-Announcement------------------------------------------
==3773091== 
==3773091== Thread #3 was created
==3773091==    at 0x49A2A23: clone (clone.S:76)
==3773091==    by 0x49A2BA2: __clone_internal_fallback (clone-internal.c:64)
==3773091==    by 0x49A2BA2: __clone_internal (clone-internal.c:109)
==3773091==    by 0x491554F: create_thread (pthread_create.c:297)
==3773091==    by 0x49161A4: pthread_create@@GLIBC_2.34 (pthread_create.c:836)
==3773091==    by 0x4854975: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x1093F4: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock)
==3773091== 
==3773091== ----------------------------------------------------------------
==3773091== 
==3773091== Thread #3: lock order "0x10C040 before 0x10C080" violated
==3773091== 
==3773091== Observed (incorrect) order is: acquisition of lock at 0x10C080
==3773091==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x109288: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock)
==3773091==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773091==    by 0x49A2A33: clone (clone.S:100)
==3773091== 
==3773091==  followed by a later acquisition of lock at 0x10C040
==3773091==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x1092C3: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock)
==3773091==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773091==    by 0x49A2A33: clone (clone.S:100)
==3773091== 
==3773091== Required order was established by acquisition of lock at 0x10C040
==3773091==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x10920E: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock)
==3773091==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773091==    by 0x49A2A33: clone (clone.S:100)
==3773091== 
==3773091==  followed by a later acquisition of lock at 0x10C080
==3773091==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x109249: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock)
==3773091==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773091==    by 0x49A2A33: clone (clone.S:100)
==3773091== 
==3773091==  Lock at 0x10C040 was first observed
==3773091==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x10920E: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock)
==3773091==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773091==    by 0x49A2A33: clone (clone.S:100)
==3773091==  Address 0x10c040 is 0 bytes inside data symbol "m1"
==3773091== 
==3773091==  Lock at 0x10C080 was first observed
==3773091==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x109249: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock)
==3773091==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773091==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773091==    by 0x49A2A33: clone (clone.S:100)
==3773091==  Address 0x10c080 is 0 bytes inside data symbol "m2"
==3773091== 
==3773091== 
==3773091== 
==3773091== Use --history-level=approx or =none to gain increased speed, at
==3773091== the cost of reduced accuracy of conflicting-access information
==3773091== For lists of detected and suppressed errors, rerun with: -s
==3773091== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 8 from 8)
```

## Problem 5

### Problem

Now run helgrind on `main-deadlock-global.c`. Examine the code; does it have the same problem that `main-deadlock.c` has? Should helgrind be reporting the same error? What does this tell you about tools like helgrind?

### Answer

It doesn't catch the context with lock `g`.

```bash
$ valgrind --tool=helgrind ./main-deadlock-global
==3773350== Helgrind, a thread error detector
==3773350== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==3773350== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3773350== Command: ./main-deadlock-global
==3773350== 
==3773350== ---Thread-Announcement------------------------------------------
==3773350== 
==3773350== Thread #3 was created
==3773350==    at 0x49A2A23: clone (clone.S:76)
==3773350==    by 0x49A2BA2: __clone_internal_fallback (clone-internal.c:64)
==3773350==    by 0x49A2BA2: __clone_internal (clone-internal.c:109)
==3773350==    by 0x491554F: create_thread (pthread_create.c:297)
==3773350==    by 0x49161A4: pthread_create@@GLIBC_2.34 (pthread_create.c:836)
==3773350==    by 0x4854975: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x10946A: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock-global)
==3773350== 
==3773350== ----------------------------------------------------------------
==3773350== 
==3773350== Thread #3: lock order "0x10C080 before 0x10C0C0" violated
==3773350== 
==3773350== Observed (incorrect) order is: acquisition of lock at 0x10C0C0
==3773350==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x1092C3: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock-global)
==3773350==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773350==    by 0x49A2A33: clone (clone.S:100)
==3773350== 
==3773350==  followed by a later acquisition of lock at 0x10C080
==3773350==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x1092FE: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock-global)
==3773350==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773350==    by 0x49A2A33: clone (clone.S:100)
==3773350== 
==3773350== Required order was established by acquisition of lock at 0x10C080
==3773350==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x109249: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock-global)
==3773350==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773350==    by 0x49A2A33: clone (clone.S:100)
==3773350== 
==3773350==  followed by a later acquisition of lock at 0x10C0C0
==3773350==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x109284: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock-global)
==3773350==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773350==    by 0x49A2A33: clone (clone.S:100)
==3773350== 
==3773350==  Lock at 0x10C080 was first observed
==3773350==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x109249: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock-global)
==3773350==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773350==    by 0x49A2A33: clone (clone.S:100)
==3773350==  Address 0x10c080 is 0 bytes inside data symbol "m1"
==3773350== 
==3773350==  Lock at 0x10C0C0 was first observed
==3773350==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x109284: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-deadlock-global)
==3773350==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773350==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773350==    by 0x49A2A33: clone (clone.S:100)
==3773350==  Address 0x10c0c0 is 0 bytes inside data symbol "m2"
==3773350== 
==3773350== 
==3773350== 
==3773350== Use --history-level=approx or =none to gain increased speed, at
==3773350== the cost of reduced accuracy of conflicting-access information
==3773350== For lists of detected and suppressed errors, rerun with: -s
==3773350== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 8 from 8)
```

## Problem 6

### Problem

Let’s next look at `main-signal.c`. This code uses a variable (done) to signal that the child is done and that the parent can now continue. Why is this code inefficient? (what does the parent end up spending its time doing, particularly if the child thread takes a long time to complete?)

### Answer

The parent spends unnecessary CPU resources in the while loop

## Problem 7

### Problem

Now run helgrind on this program. What does it report? Is the code correct?

### Answer

```bash
$ valgrind --tool=helgrind ./main-signal
==3773695== Helgrind, a thread error detector
==3773695== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==3773695== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3773695== Command: ./main-signal
==3773695== 
this should print first
==3773695== ---Thread-Announcement------------------------------------------
==3773695== 
==3773695== Thread #2 was created
==3773695==    at 0x49A2A23: clone (clone.S:76)
==3773695==    by 0x49A2BA2: __clone_internal_fallback (clone-internal.c:64)
==3773695==    by 0x49A2BA2: __clone_internal (clone-internal.c:109)
==3773695==    by 0x491554F: create_thread (pthread_create.c:297)
==3773695==    by 0x49161A4: pthread_create@@GLIBC_2.34 (pthread_create.c:836)
==3773695==    by 0x4854975: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773695==    by 0x109217: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-signal)
==3773695== 
==3773695== ---Thread-Announcement------------------------------------------
==3773695== 
==3773695== Thread #1 is the program's root thread
==3773695== 
==3773695== ----------------------------------------------------------------
==3773695== 
==3773695== Possible data race during write of size 4 at 0x10C014 by thread #2
==3773695== Locks held: none
==3773695==    at 0x1091C8: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-signal)
==3773695==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773695==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773695==    by 0x49A2A33: clone (clone.S:100)
==3773695== 
==3773695== This conflicts with a previous read of size 4 by thread #1
==3773695== Locks held: none
==3773695==    at 0x109245: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-signal)
==3773695==  Address 0x10c014 is 0 bytes inside data symbol "done"
==3773695== 
==3773695== ----------------------------------------------------------------
==3773695== 
==3773695== Possible data race during read of size 4 at 0x10C014 by thread #1
==3773695== Locks held: none
==3773695==    at 0x109245: main (in /home/wookyung/programming/ostep/homework/27-thread-api/main-signal)
==3773695== 
==3773695== This conflicts with a previous write of size 4 by thread #2
==3773695== Locks held: none
==3773695==    at 0x1091C8: worker (in /home/wookyung/programming/ostep/homework/27-thread-api/main-signal)
==3773695==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==3773695==    by 0x4915AA3: start_thread (pthread_create.c:447)
==3773695==    by 0x49A2A33: clone (clone.S:100)
==3773695==  Address 0x10c014 is 0 bytes inside data symbol "done"
==3773695== 
this should print last
==3773695== 
==3773695== Use --history-level=approx or =none to gain increased speed, at
==3773695== the cost of reduced accuracy of conflicting-access information
==3773695== For lists of detected and suppressed errors, rerun with: -s
==3773695== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 59 from 33)
```

## Problem 8

### Problem

Now look at a slightly modified version of the code, which is found in `main-signal-cv.c`. This version uses a condition variable to do the signaling (and associated lock). Why is this code preferred to the previous version? Is it correctness, or performance, or both?

### Answer

Better in both correctness and performance (the previous code also works in expected way, but it has data race problem)

## Problem 9

### Problem

Once again run helgrind on `main-signal-cv`. Does it report any errors?

### Answer

```bash
$ valgrind --tool=helgrind ./main-signal-cv
==3773973== Helgrind, a thread error detector
==3773973== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==3773973== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3773973== Command: ./main-signal-cv
==3773973== 
this should print first
this should print last
==3773973== 
==3773973== Use --history-level=approx or =none to gain increased speed, at
==3773973== the cost of reduced accuracy of conflicting-access information
==3773973== For lists of detected and suppressed errors, rerun with: -s
==3773973== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 12 from 12)
```