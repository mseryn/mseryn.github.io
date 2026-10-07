---
layout: page
title: COMP 464 practice problems
description: MPI and OpenMP practice problems for the midterm
permalink: /teaching/comp464/practice-questions/
giscus_comments: false
---

## MPI (41)

### Level 1: Recall and single calls

**M1.** Write the four calls nearly every MPI program uses: start MPI, get this process's rank, get the total number of processes, and shut down.

**M2.** Give the command to compile `hello.c` into an executable called `hello`, and the command to run it with 4 processes.

**M3.** This program from lecture is run with `-np 3`. How many lines print, what rank numbers appear, and in what order?
```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char **argv) {
    int rank, size;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    printf("Hello from rank %d of %d\n", rank, size);
    MPI_Finalize();
    return 0;
}
```

**M4.** Give the MPI datatype for each C type: `int`, `double`, `char` used as text, `unsigned long`, `float`.

**M5.** Write an `MPI_Send` that sends one `int` named `x` to rank 1, with tag 0, on `MPI_COMM_WORLD`.

**M6.** Write the matching `MPI_Recv` on rank 1 that receives the value from M5 into an `int` named `y`. You don't need the status.

**M7.** Write an `MPI_Send` that sends an array `arr` of 100 floats to rank 3, with tag 7.

**M8.** Write the matching receive on rank 3, into an array `buf` of 100 floats. Assume the sender is rank 0.

**M9.** Name each argument in this call:
`MPI_Send(a, 10, MPI_DOUBLE, 2, 5, MPI_COMM_WORLD);`

**M10.** What does `MPI_COMM_WORLD` contain?

**M11.** What is `MPI_STATUS_IGNORE` for?

**M12.** This ping-pong code from lecture is run with `-np 1`. What goes wrong?
```c
int n = 0;
if (rank == 0) {
    n = 42;
    MPI_Send(&n, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
    MPI_Recv(&n, 1, MPI_INT, 1, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    printf("rank 0 got back %d\n", n);
}
else if (rank == 1) {
    MPI_Recv(&n, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    n += 1;
    MPI_Send(&n, 1, MPI_INT, 0, 0, MPI_COMM_WORLD);
}
```

### Level 2: Short programs and predicting output

**M13.** Write a program where rank 0 sends the integer 42 to rank 1, and rank 1 prints what it received.

**M14.** Run the ping-pong code from M12 with `-np 2`. What prints?

**M15.** This program from lecture is run with `-np 4`. What prints, and in what order?
```c
if (rank == 0) {
    /* hand each worker a number */
    for (int dest = 1; dest < size; dest++) {
        int n = dest * 10;
        MPI_Send(&n, 1, MPI_INT, dest, 0, MPI_COMM_WORLD);
    }
    /* collect the answers, in rank order */
    for (int src = 1; src < size; src++) {
        int result;
        MPI_Recv(&result, 1, MPI_INT, src, 0, MPI_COMM_WORLD,
                 MPI_STATUS_IGNORE);
        printf("rank %d sent back %d\n", src, result);
    }
}
else {
    int n;
    MPI_Recv(&n, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    n = n * n;
    MPI_Send(&n, 1, MPI_INT, 0, 0, MPI_COMM_WORLD);
}
```

**M16.** Write code where rank 0 sends the value `i * 2` to each rank `i` from 1 to size−1, and each of those ranks prints what it receives.

**M17.** Write an `MPI_Bcast` that sends an `int n` from rank 0 to every rank.

**M18.** This code is run with `-np 4`. What does each rank print?
```c
int n = 0;
if (rank == 0) n = 7;
printf("before: rank %d has n = %d\n", rank, n);
MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);
printf("after: rank %d has n = %d\n", rank, n);
```

**M19.** Write an `MPI_Reduce` that sums each process's rank into `total` on rank 0. With `-np 4`, what is `total`?

**M20.** Using the reduce from M19, what is `total` with `-np 5`? And with `MPI_MAX` instead of `MPI_SUM`, still with `-np 5`?

**M21.** Rewrite M19 using `MPI_Allreduce`. After the call, which ranks have the result?

**M22.** What does `MPI_Barrier` do? Where would you place it when timing a section of code?

**M23.** Every rank runs this loop. Add `MPI_Wtime` calls to time it, and print each rank's elapsed time.
```c
double x = 0.0;
for (long i = 0; i < 100000000L; i++)
    x += 1.0 / (i + 1);
```

**M24.** The ranks are arranged in a ring: each rank's right neighbor is the next rank up, and the last rank wraps around to rank 0. Write C expressions for `left` and `right` using `rank` and `size`. With `size = 4`, what are the left and right neighbors of rank 0 and of rank 3?

**M25.** Each rank computes `rank * rank`, and the values are summed onto rank 0 with `MPI_Reduce`. What is the result with `-np 4`?

**M26.** Rank 0 has `int data[4] = {10, 20, 30, 40};`. Write an `MPI_Scatter` that sends one element to each of 4 ranks. Each rank stores its element in `int mine`. What does rank 2 receive?

**M27.** Each rank sets `int mine = rank * 10;`. Write an `MPI_Gather` that collects these into `int all[4]` on rank 0. With `-np 4`, what does `all` contain?

**M28.** Given `for (int i = rank; i < N; i += size)` with `N = 10` and `size = 3`, which values of `i` does each rank handle?

### Level 3: Bugs, non-blocking, and full programs

**M29.** Block distribution: with `N = 100` and `size = 4`, use `start = rank * N / size` and `end = (rank + 1) * N / size`, where each rank handles `start <= i < end`. Which indices does rank 2 handle?

**M30.** Two ranks each run this code, with `other` set to the other rank. What happens, and how do you fix it?
```c
MPI_Recv(in, 1, MPI_INT, other, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
MPI_Send(out, 1, MPI_INT, other, 0, MPI_COMM_WORLD);
```

**M31.** This exchange from lecture runs on two ranks. Why can it work for small N but hang for large N? Rewrite it safely.
```c
MPI_Send(out, N, MPI_DOUBLE, other, 0, MPI_COMM_WORLD);
MPI_Recv(in, N, MPI_DOUBLE, other, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
```

**M32.** Write a non-blocking exchange with a partner rank `other`: start the receive and the send, then sum `1.0 / (i + 1)` for i from 0 to N−1 into `double local` while the messages are in flight, then wait for both to finish.

**M33.** What is wrong here?
```c
MPI_Isend(out, N, MPI_DOUBLE, other, 0, MPI_COMM_WORLD, &req);
out[0] = 99.0;
MPI_Wait(&req, MPI_STATUS_IGNORE);
```

**M34.** Rank 1 sends 10 ints. Rank 0 receives with a buffer `int buf[5]` and a count of 5. What happens?

**M35.** This code is run with `-np 2`. Rank 0 should send its 8 doubles to rank 1. What is wrong, and how do you fix it? (Hint: what unit does `sizeof` return, and what unit does the count argument expect?)
```c
double vals[8];
if (rank == 0) {
    for (int i = 0; i < 8; i++) vals[i] = i;
    MPI_Send(vals, sizeof(vals), MPI_DOUBLE, 1, 0, MPI_COMM_WORLD);
} else if (rank == 1) {
    MPI_Recv(vals, sizeof(vals), MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
}
```

**M36.** What is wrong here?
```c
if (rank == 0)
    MPI_Reduce(&local, &total, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
```

**M37.** Rank 0 should receive one int from each other rank, in whatever order they arrive, and print who sent it. Use an `MPI_Status` variable to find out which rank sent each message. Write rank 0's loop.

**M38.** This program from lecture is run with `-np 2`. What does rank 0 print?
```c
if (rank == 1) {
    int data[3] = {4, 5, 6};
    MPI_Send(data, 3, MPI_INT, 0, 42, MPI_COMM_WORLD);
}
else if (rank == 0) {
    int buf[10];
    int count;
    MPI_Request req;
    MPI_Status status;
    /* post the receive, accept any tag */
    MPI_Irecv(buf, 10, MPI_INT, 1, MPI_ANY_TAG, MPI_COMM_WORLD, &req);
    printf("rank 0: receive posted, doing other work\n");
    /* block here until the message is in buf */
    MPI_Wait(&req, &status);
    MPI_Get_count(&status, MPI_INT, &count);
    printf("rank 0: got %d ints with tag %d and data: %d %d %d\n",
           count, status.MPI_TAG, buf[0], buf[1], buf[2]);
}
```

**M39.** Write an MPI program that estimates pi by integrating 4/(1+x²) from 0 to 1 using N steps. Use a cyclic loop and `MPI_Reduce`.

**M40.** This program from lecture is run with `-np 4`. Which rank is slowest, and about how much slower is it than rank 0? What does this tell you about the work distribution?
```c
int main(int argc, char **argv) {
    int rank;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    /* higher ranks get more work */
    long iters = (rank + 1) * 20000000L;
    MPI_Barrier(MPI_COMM_WORLD); /* start everyone together */
    double t0 = MPI_Wtime();
    double x = 0.0;
    for (long i = 0; i < iters; i++)
        x += 1.0 / (i + 1);
    double elapsed = MPI_Wtime() - t0;
    printf("rank %d: %.3f s (x = %f)\n", rank, elapsed, x);
    double fastest, slowest;
    MPI_Reduce(&elapsed, &fastest, 1, MPI_DOUBLE, MPI_MIN, 0, MPI_COMM_WORLD);
    MPI_Reduce(&elapsed, &slowest, 1, MPI_DOUBLE, MPI_MAX, 0, MPI_COMM_WORLD);
    if (rank == 0)
        printf("fastest %.3f s, slowest %.3f s, timer resolution %g s\n",
               fastest, slowest, MPI_Wtick());
    MPI_Finalize();
    return 0;
}
```

**M41.** Rank 0 builds `int a[1000]` with `a[i] = i`. Scatter equal chunks to all ranks (assume 1000 divides evenly by `size`), have each rank sum its chunk, and reduce the total to rank 0. What is the final answer?

## OpenMP (30)

### Level 1: Recall and basic directives

**O1.** What compiler flag enables OpenMP with gcc?

**O2.** Which header do you include to call OpenMP runtime routines?

**O3.** Give two ways to set the number of threads to 4.

**O4.** Write a parallel region where each thread prints its thread number and the team size.

**O5.** O4 is run with 4 threads. How many lines print, and in what order?

**O6.** What range of values can `omp_get_thread_num()` return in a team of size T?

**O7.** What does `omp_get_num_threads()` return outside a parallel region? Inside a region with 4 threads?

**O8.** When would you use `omp_get_max_threads()` instead of `omp_get_num_threads()`?

**O9.** Parallelize this loop:
```c
for (int i = 0; i < N; i++) a[i] = b[i] + c[i];
```

**O10.** Describe the fork-join model in three or fewer sentences.

### Level 2: Data sharing, scheduling, and synchronization

**O11.** This code is from lecture. What is wrong with it, and how do you fix it?
```c
int num_threads, thread_id;
#pragma omp parallel
{
    num_threads = omp_get_num_threads();
    thread_id = omp_get_thread_num();
    printf("Hello from thread %d out of %d\n", thread_id, num_threads);
}
```

**O12.** This code is from lecture. With 4 threads, what does each version print?
```c
int initial_value = 15;
#pragma omp parallel private(initial_value)
{
    printf("Thread %d sees initial_value = %d\n",
           omp_get_thread_num(), initial_value);
}
```
```c
int initial_value = 15;
#pragma omp parallel firstprivate(initial_value)
{
    printf("Thread %d sees initial_value = %d\n",
           omp_get_thread_num(), initial_value);
}
```

**O13.** What is wrong with this sum, and how do you fix it with `critical`?
```c
double sum = 0;
#pragma omp parallel for
for (int i = 0; i < N; i++) sum += a[i];
```

**O14.** The fix in O13 is correct but slow. Rewrite it so each thread keeps a local partial sum and enters the critical section only once.

**O15.** Compute the same sum as O13 without `critical`. Allocate an array `partial` with one entry per thread, sized with `omp_get_max_threads()`. In the parallel loop, each thread adds into its own entry, `partial[omp_get_thread_num()]`. After the parallel region, add the entries together serially to get `sum`.

**O16.** This code is from lecture. Which loop gets split across threads in each version?
```c
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    for (int j = 0; j < M; j++) {
        ...
    }
}
```
```c
#pragma omp parallel for collapse(2)
for (int i = 0; i < N; i++) {
    for (int j = 0; j < M; j++) {
        ...
    }
}
```

**O17.** Use `collapse(2)` to parallelize the initialization of an N-by-M matrix to zero.

**O18.** Explain these OpenMP schedules: `static`, `dynamic`, and `guided`.

**O19.** Parallelize this loop with a dynamic schedule and chunks of 4:
```c
for (int i = 0; i < N; i++) {
    double s = 0.0;
    for (int j = 0; j < i; j++) s += 1.0 / (j + 1);
    a[i] = s;
}
```

**O20.** With `schedule(static)`, 4 threads, and 16 iterations (0 to 15), which iterations does thread 1 get?

**O21.** Parallelize this loop, time it with OpenMP's timer, and print the elapsed time.
```c
for (int i = 0; i < N; i++) a[i] = b[i] * 2.0;
```

**O22.** Inside a parallel region, print "starting" exactly once, from thread 0. (Hint: this can be done more than one way.)

**O23.** In this code, each thread writes its own entry of `vals`, then prints its neighbor's entry. A thread may read its neighbor's entry before the neighbor has written it. Fix it.
```c
int vals[64];   /* assume at most 64 threads */
#pragma omp parallel
{
    int t = omp_get_thread_num();
    int T = omp_get_num_threads();
    vals[t] = t * t;
    printf("thread %d sees %d\n", t, vals[(t + 1) % T]);
}
```

**O24.** Fill `a[i] = i * i` and `b[i] = 2 * i` for i from 0 to N−1 at the same time, with one thread filling each array. (Hint: use sections.)

**O25.** This code computes `i * i` in parallel for i from 0 to 7. What prints?
```c
#pragma omp parallel for ordered
for (int i = 0; i < 8; i++) {
    int sq = i * i;
    #pragma omp ordered
    printf("%d\n", sq);
}
```

### Level 3: Locks and full programs

**O26.** This code protects a shared counter with an OpenMP lock, and each thread increments it once. With 4 threads, what is the final value of `count`?
```c
omp_lock_t lock;
int count = 0;
omp_init_lock(&lock);
#pragma omp parallel
{
    omp_set_lock(&lock);
    count++;
    omp_unset_lock(&lock);
}
omp_destroy_lock(&lock);
```

**O27.** This code counts the positive and non-positive entries of `a`. The two counters are unrelated, but both are updated in unnamed `critical` sections. The result is correct, but the code is slower than it needs to be. Why, and how do you fix it? (Hint: can a thread updating `hits` and a thread updating `misses` be in their critical sections at the same time?)
```c
int hits = 0, misses = 0;
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    if (a[i] > 0) {
        #pragma omp critical
        hits++;
    } else {
        #pragma omp critical
        misses++;
    }
}
```

**O28.** Count the primes below N in parallel using this function, which takes longer for larger n. Use a partial count with one critical update, and choose a schedule.
```c
int is_prime(int n) {
    for (int d = 2; d * d <= n; d++)
        if (n % d == 0) return 0;
    return 1;
}
```

**O29.** Estimate pi with OpenMP by integrating 4/(1+x²) from 0 to 1 using N steps. Use partial sums.

**O30.** Parallelize the matrix-vector multiply `y = A * x` for an N-by-N matrix. Which loop do you parallelize, and why that one?

## Combined (15)

**C1.** What is a hybrid MPI + OpenMP program, and how do you compile one?

**C2.** Write a hybrid hello where each MPI rank starts 3 OpenMP threads, and each thread prints its rank and thread number. With `-np 2`, how many lines print?

**C3.** You run a hybrid MPI + OpenMP program on 4 nodes with 16 cores each. Give two ways to choose the number of MPI ranks and the number of OpenMP threads per rank so that every core is used, but no node runs more threads than it has cores.

**C4.** A program has a deadlock in its MPI section and a race condition in its OpenMP section. Explain the difference in symptoms.

**C5.** A large array of doubles has been split across the MPI ranks. Each rank holds its piece in an array `local` of length `chunk`. Find the sum of the whole array in two steps:
1. Inside each rank, use OpenMP threads to add up `local` into a variable `mysum`. Each thread should sum its share of the loop into its own `partial` variable, then add `partial` into `mysum` once, inside a `critical` section (as in O14).
2. Use `MPI_Reduce` to add up every rank's `mysum` into `total` on rank 0. Rank 0 prints `total`.

**C6.** Estimate pi by integrating 4/(1+x²) from 0 to 1 using N steps, the same calculation as M39 and O29, but now with both MPI and OpenMP:
1. Split the N steps across ranks with a block distribution (as in M29): each rank handles steps `start` through `end − 1`, where `start = rank * N / size` and `end = (rank + 1) * N / size`.
2. Inside each rank, use OpenMP threads to sum that block, with each thread keeping its own partial sum (as in O29). Store the rank's result in `mine`.
3. Use `MPI_Reduce` to add every rank's `mine` into `pi` on rank 0. Rank 0 prints `pi`.

**C7.** Rank 0 reads an integer N from the user. Every rank then runs an OpenMP loop over its share of the iterations 0 through N−1. Write it in three steps:
1. Rank 0 reads N with `scanf`. At this point, the other ranks don't know N.
2. Use `MPI_Bcast` to send N from rank 0 to every rank (as in M17).
3. Each rank loops over its share with a cyclic distribution (as in M28): rank r handles i = r, r + size, r + 2·size, and so on. Parallelize this loop with OpenMP, and print the rank, thread number, and `i` for each iteration.

**C8.** In a hybrid program, every rank runs the OpenMP loop below. Ranks can finish at different times, and the job isn't done until the slowest rank finishes, so the job's runtime is the slowest rank's time. Measure it in three steps:
1. Use `MPI_Barrier` so all ranks start together, then record the start time with `MPI_Wtime()` (as in M23).
2. Run the loop, then compute this rank's `elapsed` time.
3. Use `MPI_Reduce` with `MPI_MAX` to put the largest `elapsed` across all ranks into `slowest` on rank 0. Rank 0 prints `slowest`.
```c
#pragma omp parallel for
for (int i = 0; i < N; i++) a[i] = b[i] * 2.0;
```
Then answer: would the OpenMP timer, `omp_get_wtime()`, also work here? Why or why not?

**C9.** The ranks are arranged in a ring (as in M24). Each rank sends its own rank number to its right neighbor and receives a number from its left neighbor, then uses that number to fill an array. Write it in three steps:
1. Compute `left` and `right` for this rank.
2. Send `rank` to `right` and receive from `left` into `int in`. Every rank sends and receives at the same time, so use `MPI_Sendrecv` to avoid the deadlock from M30 and M31.
3. Use OpenMP to fill `a[i] = in * i` for i from 0 to N−1.

### Speedup and efficiency (C10–C15)

Use these formulas for C10 through C15:

| Quantity | Formula |
|---|---|
| Number of workers | p = MPI ranks × OpenMP threads per rank |
| Speedup | S = T_serial / T_parallel |
| Efficiency | E = S / p |

T_serial is the run time of the serial program, and T_parallel is the run time with p workers. An efficiency of 1.0 means every worker is fully used. Real programs usually come in below 1.0.

The same formulas, rearranged:
- S = E × p
- T_parallel = T_serial / S
- T_serial = S × T_parallel

**C10.** A program takes 80 s to run serially. With 4 MPI ranks it takes 22 s, and with 4 ranks × 4 OpenMP threads it takes 7 s. For each parallel run, compute the speedup and the efficiency.

**C11.** A program takes 120 s to run serially. You run it with 8 MPI ranks and measure a parallel efficiency of 0.75. How long did the parallel run take? (Hint: find the speedup first.)

**C12.** A program takes 64 s to run serially. Here are its parallel run times:

| Workers | Time |
|---|---|
| 2 | 33 s |
| 4 | 17 s |
| 8 | 10 s |
| 16 | 9 s |

Compute the speedup and efficiency for each run. Is it worth going from 8 workers to 16?

**C13.** A program takes 200 s to run serially. On a cluster with 64 cores, you try two layouts:
- 64 MPI ranks × 1 thread each: 5 s
- 8 MPI ranks × 8 OpenMP threads each: 4 s

Compute the speedup and efficiency for each. Which layout uses the cores better?

**C14.** You don't know a program's serial time. A run with 4 workers takes 30 s, with a parallel efficiency of 0.8. What is the serial time? (Hint: use the rearranged formulas.)

**C15.** A program takes 96 s to run serially. You compare two runs:
- Run A: 16 workers, 10 s
- Run B: 8 workers, 14 s

(a) Which run finishes first?
(b) Compute the speedup and efficiency of each run.
(c) You have 16 cores and need to run the program twice, on two different inputs. Is it faster to run A twice, one after the other, or to run two copies of B at the same time, each on 8 cores?
