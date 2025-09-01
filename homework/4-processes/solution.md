## Problem 1

#### Problem

Run `process-run.py` with the following flags: `-l 5:100,5:100`. What should the CPU utilization be (e.g., the percent of the CPU is in use?) Why do you know this? Use the `-c` and `-p` flags to see if you were right.

#### My Answer

The percent of time the CPU is in use must be $100\% (= 10/10)$.
We don't have any reason to block, so it will just finish the process0 (or process1) and then finish the other one.

#### Correct Answer

```bash
$ uv run process-run.py -l 5:100,5:100 -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2        RUN:cpu         READY             1          
  3        RUN:cpu         READY             1          
  4        RUN:cpu         READY             1          
  5        RUN:cpu         READY             1          
  6           DONE       RUN:cpu             1          
  7           DONE       RUN:cpu             1          
  8           DONE       RUN:cpu             1          
  9           DONE       RUN:cpu             1          
 10           DONE       RUN:cpu             1          

Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
```

## Problem 2

#### Problem

Now run with these flags: `./process-run.py -l 4:100,1:0`. These flags specify one process with 4 instructions (all to use the CPU), and one that simply issues an I/O and waits for it to be done.
How long does it take to complete both processes?
Use `-c` and `-p` to find out if you were right.

#### My Answer

Ideally takes 5 (if the time for waiting is less than 5).

If we run process1 first and run process0 while waiting for the I/O, then it takes 5.

#### Correct Answer

```bash
$ uv run process-run.py -l 4:100,1:0 -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2        RUN:cpu         READY             1          
  3        RUN:cpu         READY             1          
  4        RUN:cpu         READY             1          
  5           DONE        RUN:io             1          
  6           DONE       BLOCKED                           1
  7           DONE       BLOCKED                           1
  8           DONE       BLOCKED                           1
  9           DONE       BLOCKED                           1
 10           DONE       BLOCKED                           1
 11*          DONE   RUN:io_done             1          

Stats: Total Time 11
Stats: CPU Busy 6 (54.55%)
Stats: IO Busy  5 (45.45%)
```

## Problem 3

#### Problem

Switch the order of the processes: `-l 1:0,4:100`. What happens now? Does switching the order matter? Why? (As always, use `-c` and `-p` to see if you were right)

#### My Answer

Based on the result of problem 2, we can observe that:
- process with smaller PID comes first
- I/O block takes 5, and `RUN:io_done` takes additional 1. Total of 6.

My speculation:
- time 1: process0 runs first
- time 2: process1 runs since process0 is blocked
- time 6: process1 done, but process0 is still blocked
- time 7: process0 runs and do `RUN:io_done`

#### Correct Answer

```bash
$ uv run process-run.py -l 1:0,4:100 -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1          
  2        BLOCKED       RUN:cpu             1             1
  3        BLOCKED       RUN:cpu             1             1
  4        BLOCKED       RUN:cpu             1             1
  5        BLOCKED       RUN:cpu             1             1
  6        BLOCKED          DONE                           1
  7*   RUN:io_done          DONE             1          

Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
```

## Problem 4

#### Problem

We’ll now explore some of the other flags. One important flag is `-S`, which determines how the system reacts when a process issues an I/O. With the flag set to SWITCH_ON_END, the system will NOT switch to another process while one is doing I/O, instead waiting until the process is completely finished. What happens when you run the following two processes (`-l 1:0,4:100 -c -S SWITCH_ON_END`), one doing I/O and the other doing CPU work?

#### My Answer

| Time | PID: 0 | PID: 1 | CPU | IOs |
| --- | --- | --- | --- | --- |
| 1 | RUN:io | READY | 1 | |
| 2 | BLOCKED | READY | | 1 |
| 3 | BLOCKED | READY | | 1 |
| 4 | BLOCKED | READY | | 1 |
| 5 | BLOCKED | READY | | 1 |
| 6 | BLOCKED | READY | | 1 |
| 7 | RUN:io_done | READY | 1 | |
| 8 | DONE | RUN:cpu | 1 | |
| 9 | DONE | RUN:cpu | 1 | |
| 10 | DONE | RUN:cpu | 1 | |
| 11 | DONE | RUN:cpu | 1 | |

- Stats: Total Time 11
- Stats: CPU Busy 6
- Stats: IO Busy  5

#### Correct Answer

```bash
$ uv run process-run.py -l 1:0,4:100 -c -p -S SWITCH_ON_END
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1          
  2        BLOCKED         READY                           1
  3        BLOCKED         READY                           1
  4        BLOCKED         READY                           1
  5        BLOCKED         READY                           1
  6        BLOCKED         READY                           1
  7*   RUN:io_done         READY             1          
  8           DONE       RUN:cpu             1          
  9           DONE       RUN:cpu             1          
 10           DONE       RUN:cpu             1          
 11           DONE       RUN:cpu             1          

Stats: Total Time 11
Stats: CPU Busy 6 (54.55%)
Stats: IO Busy  5 (45.45%)
```

## Problem 5

#### Problem

Now, run the same processes, but with the switching behavior set to switch to another process whenever one is WAITING for I/O (`-l 1:0,4:100 -c -S SWITCH_ON_IO`). What happens now? Use `-c` and `-p` to confirm that you are right.

#### My Answer

| Time | PID: 0 | PID: 1 | CPU | IOs |
| --- | --- | --- | --- | --- |
| 1 | RUN:io | READY | 1 | |
| 2 | BLOCKED | RUN:cpu | 1 | 1 |
| 3 | BLOCKED | RUN:cpu | 1 | 1 |
| 4 | BLOCKED | RUN:cpu | 1 | 1 |
| 5 | BLOCKED | RUN:cpu | 1 | 1 |
| 6 | BLOCKED | DONE | | 1 |
| 7 | RUN:io_done | DONE | 1 | |

- Stats: Total Time 7
- Stats: CPU Busy 6
- Stats: IO Busy  5

#### Correct Answer

```bash
$ uv run process-run.py -l 1:0,4:100 -c -p -S SWITCH_ON_IO
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1          
  2        BLOCKED       RUN:cpu             1             1
  3        BLOCKED       RUN:cpu             1             1
  4        BLOCKED       RUN:cpu             1             1
  5        BLOCKED       RUN:cpu             1             1
  6        BLOCKED          DONE                           1
  7*   RUN:io_done          DONE             1          

Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
```

## Problem 6

#### Problem

One other important behavior is what to do when an I/O completes. With `-I IO_RUN_LATER`, when an I/O completes, the process that issued it is not necessarily run right away; rather, whatever was running at the time keeps running. What happens when you run this combination of processes? (`./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -c -p -I IO_RUN_LATER`) Are system resources being effectively utilized?

#### My Answer

When it comes to CPU utilization, yes it is effective. We use CPU in all the 17 times.

| Time | PID: 0     | PID: 1     | PID: 2     | PID: 3     | CPU | IOs |
| --- | --- | --- | --- | --- | --- | --- |
| 1    | RUN:io     | READY      | READY      | READY      | 1   |     |
| 2    | BLOCKED    | RUN:cpu    | READY      | READY      | 1   | 1   |
| 3    | BLOCKED    | RUN:cpu    | READY      | READY      | 1   | 1   |
| 4    | BLOCKED    | RUN:cpu    | READY      | READY      | 1   | 1   |
| 5    | READY      | RUN:cpu    | READY      | READY      | 1   |     |
| 6    | READY      | RUN:cpu    | READY      | READY      | 1   |     |
| 7    | RUN:io_done| DONE       | READY      | READY      | 1   |     |
| 8    | DONE       | DONE       | RUN:cpu    | READY      | 1   |     |
| 9    | DONE       | DONE       | RUN:cpu    | READY      | 1   |     |
| 10   | DONE       | DONE       | RUN:cpu    | READY      | 1   |     |
| 11   | DONE       | DONE       | RUN:cpu    | READY      | 1   |     |
| 12   | DONE       | DONE       | RUN:cpu    | READY      | 1   |     |
| 13   | DONE       | DONE       | DONE       | RUN:cpu    | 1   |     |
| 14   | DONE       | DONE       | DONE       | RUN:cpu    | 1   |     |
| 15   | DONE       | DONE       | DONE       | RUN:cpu    | 1   |     |
| 16   | DONE       | DONE       | DONE       | RUN:cpu    | 1   |     |
| 17   | DONE       | DONE       | DONE       | RUN:cpu    | 1   |     |

- Stats: Total Time 17
- Stats: CPU Busy 17
- Stats: IO Busy  3

#### Correct Answer

```bash
$ uv run process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -c -p -I IO_RUN_LATER
Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
  1         RUN:io         READY         READY         READY             1          
  2        BLOCKED       RUN:cpu         READY         READY             1             1
  3        BLOCKED       RUN:cpu         READY         READY             1             1
  4        BLOCKED       RUN:cpu         READY         READY             1             1
  5        BLOCKED       RUN:cpu         READY         READY             1             1
  6        BLOCKED       RUN:cpu         READY         READY             1             1
  7*         READY          DONE       RUN:cpu         READY             1          
  8          READY          DONE       RUN:cpu         READY             1          
  9          READY          DONE       RUN:cpu         READY             1          
 10          READY          DONE       RUN:cpu         READY             1          
 11          READY          DONE       RUN:cpu         READY             1          
 12          READY          DONE          DONE       RUN:cpu             1          
 13          READY          DONE          DONE       RUN:cpu             1          
 14          READY          DONE          DONE       RUN:cpu             1          
 15          READY          DONE          DONE       RUN:cpu             1          
 16          READY          DONE          DONE       RUN:cpu             1          
 17    RUN:io_done          DONE          DONE          DONE             1          
 18         RUN:io          DONE          DONE          DONE             1          
 19        BLOCKED          DONE          DONE          DONE                           1
 20        BLOCKED          DONE          DONE          DONE                           1
 21        BLOCKED          DONE          DONE          DONE                           1
 22        BLOCKED          DONE          DONE          DONE                           1
 23        BLOCKED          DONE          DONE          DONE                           1
 24*   RUN:io_done          DONE          DONE          DONE             1          
 25         RUN:io          DONE          DONE          DONE             1          
 26        BLOCKED          DONE          DONE          DONE                           1
 27        BLOCKED          DONE          DONE          DONE                           1
 28        BLOCKED          DONE          DONE          DONE                           1
 29        BLOCKED          DONE          DONE          DONE                           1
 30        BLOCKED          DONE          DONE          DONE                           1
 31*   RUN:io_done          DONE          DONE          DONE             1          
```
## Problem 7

#### Problem

Now run the same processes, but with `-I IO_RUN_IMMEDIATE` set, which immediately runs the process that issued the I/O. How does this behavior differ? Why might running a process that just completed an I/O again be a good idea?

#### My Answer

In this case, running a process that just completed an I/O again is a good idea, because it makes it possible to run CPU while doing I/O simultaneously.

| Time | PID: 0     | PID: 1     | PID: 2     | PID: 3     | CPU | IOs |
| --- | --- | --- | --- | --- | --- | --- |
| 1    | RUN:io     | READY      | READY      | READY      | 1   |     |
| 2    | BLOCKED    | RUN:cpu    | READY      | READY      | 1   | 1   |
| 3    | BLOCKED    | RUN:cpu    | READY      | READY      | 1   | 1   |
| 4    | BLOCKED    | RUN:cpu    | READY      | READY      | 1   | 1   |
| 5    | BLOCKED    | RUN:cpu    | READY      | READY      | 1   | 1   |
| 6    | BLOCKED    | RUN:cpu    | READY      | READY      | 1   | 1   |
| 7    | RUN:io_done| DONE       | READY      | READY      | 1   |     |
| 8    | RUN:io     | DONE       | READY      | READY      | 1   |     |
| 9    | BLOCKED    | DONE       | RUN:cpu    | READY      | 1   | 1   |
| 10   | BLOCKED    | DONE       | RUN:cpu    | READY      | 1   | 1   |
| 11   | BLOCKED    | DONE       | RUN:cpu    | READY      | 1   | 1   |
| 12   | BLOCKED    | DONE       | RUN:cpu    | READY      | 1   | 1   |
| 13   | BLOCKED    | DONE       | RUN:cpu    | READY      | 1   | 1   |
| 14   | RUN:io_done| DONE       | DONE       | READY      | 1   |     |
| 15   | RUN:io     | DONE       | DONE       | READY      | 1   |     |
| 16   | BLOCKED    | DONE       | DONE       | RUN:cpu    | 1   | 1   |
| 17   | BLOCKED    | DONE       | DONE       | RUN:cpu    | 1   | 1   |
| 18   | BLOCKED    | DONE       | DONE       | RUN:cpu    | 1   | 1   |
| 19   | BLOCKED    | DONE       | DONE       | RUN:cpu    | 1   | 1   |
| 20   | BLOCKED    | DONE       | DONE       | RUN:cpu    | 1   | 1   |
| 21   | RUN:io_done| DONE       | DONE       | DONE       | 1   |     |

- Stats: Total Time 21
- Stats: CPU Busy 21
- Stats: IO Busy 15

#### Correct Answer

```bash
$ uv run process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -c -p -I IO_RUN_IMMEDIATE
Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
  1         RUN:io         READY         READY         READY             1          
  2        BLOCKED       RUN:cpu         READY         READY             1             1
  3        BLOCKED       RUN:cpu         READY         READY             1             1
  4        BLOCKED       RUN:cpu         READY         READY             1             1
  5        BLOCKED       RUN:cpu         READY         READY             1             1
  6        BLOCKED       RUN:cpu         READY         READY             1             1
  7*   RUN:io_done          DONE         READY         READY             1          
  8         RUN:io          DONE         READY         READY             1          
  9        BLOCKED          DONE       RUN:cpu         READY             1             1
 10        BLOCKED          DONE       RUN:cpu         READY             1             1
 11        BLOCKED          DONE       RUN:cpu         READY             1             1
 12        BLOCKED          DONE       RUN:cpu         READY             1             1
 13        BLOCKED          DONE       RUN:cpu         READY             1             1
 14*   RUN:io_done          DONE          DONE         READY             1          
 15         RUN:io          DONE          DONE         READY             1          
 16        BLOCKED          DONE          DONE       RUN:cpu             1             1
 17        BLOCKED          DONE          DONE       RUN:cpu             1             1
 18        BLOCKED          DONE          DONE       RUN:cpu             1             1
 19        BLOCKED          DONE          DONE       RUN:cpu             1             1
 20        BLOCKED          DONE          DONE       RUN:cpu             1             1
 21*   RUN:io_done          DONE          DONE          DONE             1          

Stats: Total Time 21
Stats: CPU Busy 21 (100.00%)
Stats: IO Busy  15 (71.43%)
```

## Problem 8

#### Problem

Now run with some randomly generated processes using flags `-s 1 -l 3:50,3:50` or `-s 2 -l 3:50,3:50` or `-s 3 -l 3:50,3:50`. See if you can predict how the trace will turn out. What happens when you use the flag `-I IO_RUN_IMMEDIATE` versus that flag `-I IO_RUN_LATER`? What happens when you use the flag `-S SWITCH_ON_IO` versus `-S SWITCH_ON_END`?

#### My Answer

<!-- Think of the four cases:
1. `IO_RUN_IMMEDIATE` and `SWITCH_ON_IO`
2. `IO_RUN_LATER` and `SWITCH_ON_IO`
3. `IO_RUN_IMMEDIATE` and `SWITCH_ON_END`
4. `IO_RUN_LATER` and `SWITCH_ON_END` -->

In terms of total time, I guess:
- `-S SWITCH_ON_IO` is faster than `-S SWITCH_ON_END`.
- no difference on `IO_RUN_IMMEDIATE` and `IO_RUN_LATER`

#### Correct Answer

```bash
$ uv run process-run.py -s 1 -l 3:50,3:50 -I IO_RUN_IMMEDIATE -S SWITCH_ON_IO -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2         RUN:io         READY             1          
  3        BLOCKED       RUN:cpu             1             1
  4        BLOCKED       RUN:cpu             1             1
  5        BLOCKED       RUN:cpu             1             1
  6        BLOCKED          DONE                           1
  7        BLOCKED          DONE                           1
  8*   RUN:io_done          DONE             1          
  9         RUN:io          DONE             1          
 10        BLOCKED          DONE                           1
 11        BLOCKED          DONE                           1
 12        BLOCKED          DONE                           1
 13        BLOCKED          DONE                           1
 14        BLOCKED          DONE                           1
 15*   RUN:io_done          DONE             1          

Stats: Total Time 15
Stats: CPU Busy 8 (53.33%)
Stats: IO Busy  10 (66.67%)
```

```bash
$ uv run process-run.py -s 1 -l 3:50,3:50 -I IO_RUN_LATER -S SWITCH_ON_IO -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2         RUN:io         READY             1          
  3        BLOCKED       RUN:cpu             1             1
  4        BLOCKED       RUN:cpu             1             1
  5        BLOCKED       RUN:cpu             1             1
  6        BLOCKED          DONE                           1
  7        BLOCKED          DONE                           1
  8*   RUN:io_done          DONE             1          
  9         RUN:io          DONE             1          
 10        BLOCKED          DONE                           1
 11        BLOCKED          DONE                           1
 12        BLOCKED          DONE                           1
 13        BLOCKED          DONE                           1
 14        BLOCKED          DONE                           1
 15*   RUN:io_done          DONE             1          

Stats: Total Time 15
Stats: CPU Busy 8 (53.33%)
Stats: IO Busy  10 (66.67%)
```

```bash
$ uv run process-run.py -s 1 -l 3:50,3:50 -I IO_RUN_IMMEDIATE -S SWITCH_ON_END -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2         RUN:io         READY             1          
  3        BLOCKED         READY                           1
  4        BLOCKED         READY                           1
  5        BLOCKED         READY                           1
  6        BLOCKED         READY                           1
  7        BLOCKED         READY                           1
  8*   RUN:io_done         READY             1          
  9         RUN:io         READY             1          
 10        BLOCKED         READY                           1
 11        BLOCKED         READY                           1
 12        BLOCKED         READY                           1
 13        BLOCKED         READY                           1
 14        BLOCKED         READY                           1
 15*   RUN:io_done         READY             1          
 16           DONE       RUN:cpu             1          
 17           DONE       RUN:cpu             1          
 18           DONE       RUN:cpu             1          

Stats: Total Time 18
Stats: CPU Busy 8 (44.44%)
Stats: IO Busy  10 (55.56%)
```

```bash
$ uv run process-run.py -s 1 -l 3:50,3:50 -I IO_RUN_LATER -S SWITCH_ON_END -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2         RUN:io         READY             1          
  3        BLOCKED         READY                           1
  4        BLOCKED         READY                           1
  5        BLOCKED         READY                           1
  6        BLOCKED         READY                           1
  7        BLOCKED         READY                           1
  8*   RUN:io_done         READY             1          
  9         RUN:io         READY             1          
 10        BLOCKED         READY                           1
 11        BLOCKED         READY                           1
 12        BLOCKED         READY                           1
 13        BLOCKED         READY                           1
 14        BLOCKED         READY                           1
 15*   RUN:io_done         READY             1          
 16           DONE       RUN:cpu             1          
 17           DONE       RUN:cpu             1          
 18           DONE       RUN:cpu             1          

Stats: Total Time 18
Stats: CPU Busy 8 (44.44%)
Stats: IO Busy  10 (55.56%)
```