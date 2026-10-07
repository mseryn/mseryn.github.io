---
layout: page
title: COMP 464 practice answers
description: MPI and OpenMP practice problems for the midterm, with answers
permalink: /teaching/comp464/practice-answers/
giscus_comments: false
---

## MPI (41)

### Level 1: Recall and single calls

**M1.** Write the four calls nearly every MPI program uses: start MPI, get this process's rank, get the total number of processes, and shut down.

**Answer:**
```c
MPI_Init(&argc, &argv);
MPI_Comm_rank(MPI_COMM_WORLD, &rank);
MPI_Comm_size(MPI_COMM_WORLD, &size);
MPI_Finalize();
```

**M2.** Give the command to compile `hello.c` into an executable called `hello`, and the command to run it with 4 processes.

**Answer:**
```
mpicc hello.c -o hello
mpirun -np 4 ./hello
```
(`mpiexec -n 4 ./hello` is also correct.)

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

**Answer:** 3 lines, for ranks 0, 1, and 2, each saying "of 3," in any order.

**M4.** Give the MPI datatype for each C type: `int`, `double`, `char` used as text, `unsigned long`, `float`.

**Answer:** `MPI_INT`, `MPI_DOUBLE`, `MPI_CHAR`, `MPI_UNSIGNED_LONG`, `MPI_FLOAT`.

**M5.** Write an `MPI_Send` that sends one `int` named `x` to rank 1, with tag 0, on `MPI_COMM_WORLD`.

**Answer:**
```c
MPI_Send(&x, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
```

**M6.** Write the matching `MPI_Recv` on rank 1 that receives the value from M5 into an `int` named `y`. You don't need the status.

**Answer:**
```c
MPI_Recv(&y, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
```

**M7.** Write an `MPI_Send` that sends an array `arr` of 100 floats to rank 3, with tag 7.

**Answer:**
```c
MPI_Send(arr, 100, MPI_FLOAT, 3, 7, MPI_COMM_WORLD);
```

**M8.** Write the matching receive on rank 3, into an array `buf` of 100 floats. Assume the sender is rank 0.

**Answer:**
```c
MPI_Recv(buf, 100, MPI_FLOAT, 0, 7, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
```

**M9.** Name each argument in this call:
`MPI_Send(a, 10, MPI_DOUBLE, 2, 5, MPI_COMM_WORLD);`

**Answer:** `a` is the buffer address, `10` is the count, `MPI_DOUBLE` is the datatype, `2` is the destination rank, `5` is the tag, and `MPI_COMM_WORLD` is the communicator.

**M10.** What does `MPI_COMM_WORLD` contain?

**Answer:** Every process that was launched.

**M11.** What is `MPI_STATUS_IGNORE` for?

**Answer:** It goes in the status argument of a receive when you don't need the extra information (source, tag, count) about the incoming message.

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

**Answer:** Rank 0 sends to rank 1, which doesn't exist, so MPI aborts with an invalid-rank error.

### Level 2: Short programs and predicting output

**M13.** Write a program where rank 0 sends the integer 42 to rank 1, and rank 1 prints what it received.

**Answer:**
```c
int n;
if (rank == 0) {
    n = 42;
    MPI_Send(&n, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
} else if (rank == 1) {
    MPI_Recv(&n, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    printf("rank 1 received %d\n", n);
}
```

**M14.** Run the ping-pong code from M12 with `-np 2`. What prints?

**Answer:** `rank 0 got back 43`

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

**Answer:**
```
rank 1 sent back 100
rank 2 sent back 400
rank 3 sent back 900
```
Always in this order, since rank 0 receives from rank 1, then 2, then 3.

**M16.** Write code where rank 0 sends the value `i * 2` to each rank `i` from 1 to size−1, and each of those ranks prints what it receives.

**Answer:**
```c
if (rank == 0) {
    for (int i = 1; i < size; i++) {
        int v = i * 2;
        MPI_Send(&v, 1, MPI_INT, i, 0, MPI_COMM_WORLD);
    }
} else {
    int v;
    MPI_Recv(&v, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    printf("rank %d received %d\n", rank, v);
}
```

**M17.** Write an `MPI_Bcast` that sends an `int n` from rank 0 to every rank.

**Answer:**
```c
MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);
```

**M18.** This code is run with `-np 4`. What does each rank print?
```c
int n = 0;
if (rank == 0) n = 7;
printf("before: rank %d has n = %d\n", rank, n);
MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);
printf("after: rank %d has n = %d\n", rank, n);
```

**Answer:** Before the broadcast, rank 0 prints 7 and ranks 1, 2, and 3 print 0. After the broadcast, every rank prints 7. Lines from different ranks can appear in any order, but each rank prints its "before" line ahead of its "after" line.

**M19.** Write an `MPI_Reduce` that sums each process's rank into `total` on rank 0. With `-np 4`, what is `total`?

**Answer:**
```c
MPI_Reduce(&rank, &total, 1, MPI_INT, MPI_SUM, 0, MPI_COMM_WORLD);
```
0 + 1 + 2 + 3 = 6.

**M20.** Using the reduce from M19, what is `total` with `-np 5`? And with `MPI_MAX` instead of `MPI_SUM`, still with `-np 5`?

**Answer:** 10 for `MPI_SUM`, and 4 for `MPI_MAX`.

**M21.** Rewrite M19 using `MPI_Allreduce`. After the call, which ranks have the result?

**Answer:**
```c
MPI_Allreduce(&rank, &total, 1, MPI_INT, MPI_SUM, MPI_COMM_WORLD);
```
All ranks.

**M22.** What does `MPI_Barrier` do? Where would you place it when timing a section of code?

**Answer:** It blocks until every rank in the communicator reaches it. Put it right before the first `MPI_Wtime()` call.

**M23.** Every rank runs this loop. Add `MPI_Wtime` calls to time it, and print each rank's elapsed time.
```c
double x = 0.0;
for (long i = 0; i < 100000000L; i++)
    x += 1.0 / (i + 1);
```

**Answer:**
```c
MPI_Barrier(MPI_COMM_WORLD);
double t0 = MPI_Wtime();
double x = 0.0;
for (long i = 0; i < 100000000L; i++)
    x += 1.0 / (i + 1);
double elapsed = MPI_Wtime() - t0;
printf("rank %d: %f s (x = %f)\n", rank, elapsed, x);
```

**M24.** The ranks are arranged in a ring: each rank's right neighbor is the next rank up, and the last rank wraps around to rank 0. Write C expressions for `left` and `right` using `rank` and `size`. With `size = 4`, what are the left and right neighbors of rank 0 and of rank 3?

**Answer:**
```c
int right = (rank + 1) % size;
int left  = (rank - 1 + size) % size;
```
Rank 0 has left 3 and right 1. Rank 3 has left 2 and right 0.

**M25.** Each rank computes `rank * rank`, and the values are summed onto rank 0 with `MPI_Reduce`. What is the result with `-np 4`?

**Answer:** 0 + 1 + 4 + 9 = 14.

**M26.** Rank 0 has `int data[4] = {10, 20, 30, 40};`. Write an `MPI_Scatter` that sends one element to each of 4 ranks. Each rank stores its element in `int mine`. What does rank 2 receive?

**Answer:**
```c
MPI_Scatter(data, 1, MPI_INT, &mine, 1, MPI_INT, 0, MPI_COMM_WORLD);
```
Rank 2 receives 30.

**M27.** Each rank sets `int mine = rank * 10;`. Write an `MPI_Gather` that collects these into `int all[4]` on rank 0. With `-np 4`, what does `all` contain?

**Answer:**
```c
MPI_Gather(&mine, 1, MPI_INT, all, 1, MPI_INT, 0, MPI_COMM_WORLD);
```
`{0, 10, 20, 30}`

**M28.** Given `for (int i = rank; i < N; i += size)` with `N = 10` and `size = 3`, which values of `i` does each rank handle?

**Answer:** Rank 0 handles 0, 3, 6, 9. Rank 1 handles 1, 4, 7. Rank 2 handles 2, 5, 8.

### Level 3: Bugs, non-blocking, and full programs

**M29.** Block distribution: with `N = 100` and `size = 4`, use `start = rank * N / size` and `end = (rank + 1) * N / size`, where each rank handles `start <= i < end`. Which indices does rank 2 handle?

**Answer:** 50 through 74.

**M30.** Two ranks each run this code, with `other` set to the other rank. What happens, and how do you fix it?
```c
MPI_Recv(in, 1, MPI_INT, other, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
MPI_Send(out, 1, MPI_INT, other, 0, MPI_COMM_WORLD);
```

**Answer:** Deadlock: both ranks block in `MPI_Recv`, waiting for a send that never happens. Fix: have one rank send first, or use `MPI_Sendrecv`.

**M31.** This exchange from lecture runs on two ranks. Why can it work for small N but hang for large N? Rewrite it safely.
```c
MPI_Send(out, N, MPI_DOUBLE, other, 0, MPI_COMM_WORLD);
MPI_Recv(in, N, MPI_DOUBLE, other, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
```

**Answer:** Small messages are usually buffered (eager), so `MPI_Send` returns right away. Large ones wait for the matching receive (rendezvous), so both ranks block in `MPI_Send`. Safe version:
```c
MPI_Sendrecv(out, N, MPI_DOUBLE, other, 0,
             in,  N, MPI_DOUBLE, other, 0,
             MPI_COMM_WORLD, MPI_STATUS_IGNORE);
```

**M32.** Write a non-blocking exchange with a partner rank `other`: start the receive and the send, then sum `1.0 / (i + 1)` for i from 0 to N−1 into `double local` while the messages are in flight, then wait for both to finish.

**Answer:**
```c
MPI_Request reqs[2];
MPI_Irecv(in,  N, MPI_DOUBLE, other, 0, MPI_COMM_WORLD, &reqs[0]);
MPI_Isend(out, N, MPI_DOUBLE, other, 0, MPI_COMM_WORLD, &reqs[1]);
double local = 0.0;
for (int i = 0; i < N; i++) local += 1.0 / (i + 1);
MPI_Waitall(2, reqs, MPI_STATUSES_IGNORE);
```

**M33.** What is wrong here?
```c
MPI_Isend(out, N, MPI_DOUBLE, other, 0, MPI_COMM_WORLD, &req);
out[0] = 99.0;
MPI_Wait(&req, MPI_STATUS_IGNORE);
```

**Answer:** `out` is modified before the send completes. Move the assignment after `MPI_Wait`.

**M34.** Rank 1 sends 10 ints. Rank 0 receives with a buffer `int buf[5]` and a count of 5. What happens?

**Answer:** An error: the incoming message is larger than the receive buffer.

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

**Answer:** `sizeof(vals)` is 64 bytes, but the count is in elements, so both calls use a count of 64 doubles. Rank 0 sends whatever memory follows `vals`, and rank 1 writes 512 bytes into a 64-byte array, overwriting other memory. MPI doesn't report an error because the counts match, so the program may crash or silently corrupt other variables. Fix: use a count of 8.

**M36.** What is wrong here?
```c
if (rank == 0)
    MPI_Reduce(&local, &total, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
```

**Answer:** Only rank 0 calls the collective, so the program hangs. Move the reduce outside the `if`.

**M37.** Rank 0 should receive one int from each other rank, in whatever order they arrive, and print who sent it. Use an `MPI_Status` variable to find out which rank sent each message. Write rank 0's loop.

**Answer:**
```c
for (int i = 1; i < size; i++) {
    int v;
    MPI_Status st;
    MPI_Recv(&v, 1, MPI_INT, MPI_ANY_SOURCE, 0, MPI_COMM_WORLD, &st);
    printf("received %d from rank %d\n", v, st.MPI_SOURCE);
}
```

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

**Answer:**
```
rank 0: receive posted, doing other work
rank 0: got 3 ints with tag 42 and data: 4 5 6
```

**M39.** Write an MPI program that estimates pi by integrating 4/(1+x²) from 0 to 1 using N steps. Use a cyclic loop and `MPI_Reduce`.

**Answer:**
```c
long N = 1000000;
double h = 1.0 / N, local = 0.0, pi = 0.0;
for (long i = rank; i < N; i += size) {
    double x = (i + 0.5) * h;
    local += 4.0 / (1.0 + x * x);
}
local *= h;
MPI_Reduce(&local, &pi, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
if (rank == 0) printf("pi = %f\n", pi);
```

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

**Answer:** Rank 3, at about 4× rank 0's time (4× the iterations). The load is unbalanced, so the job waits on the slowest rank.

**M41.** Rank 0 builds `int a[1000]` with `a[i] = i`. Scatter equal chunks to all ranks (assume 1000 divides evenly by `size`), have each rank sum its chunk, and reduce the total to rank 0. What is the final answer?

**Answer:**
```c
int chunk = 1000 / size;
int *local = malloc(chunk * sizeof(int));
MPI_Scatter(a, chunk, MPI_INT, local, chunk, MPI_INT, 0, MPI_COMM_WORLD);
long mysum = 0, total = 0;
for (int i = 0; i < chunk; i++) mysum += local[i];
MPI_Reduce(&mysum, &total, 1, MPI_LONG, MPI_SUM, 0, MPI_COMM_WORLD);
if (rank == 0) printf("total = %ld\n", total);
free(local);
```
The total is 499500.

## OpenMP (30)

### Level 1: Recall and basic directives

**O1.** What compiler flag enables OpenMP with gcc?

**Answer:** `-fopenmp`, as in `gcc -fopenmp prog.c -o prog`.

**O2.** Which header do you include to call OpenMP runtime routines?

**Answer:** `#include <omp.h>`

**O3.** Give two ways to set the number of threads to 4.

**Answer:** Call `omp_set_num_threads(4);` in code, or run `export OMP_NUM_THREADS=4` in the shell.

**O4.** Write a parallel region where each thread prints its thread number and the team size.

**Answer:**
```c
#pragma omp parallel
{
    int id = omp_get_thread_num();
    int n  = omp_get_num_threads();
    printf("Hello from thread %d of %d\n", id, n);
}
```

**O5.** O4 is run with 4 threads. How many lines print, and in what order?

**Answer:** 4 lines, for threads 0 through 3. The order is not guaranteed.

**O6.** What range of values can `omp_get_thread_num()` return in a team of size T?

**Answer:** 0 through T − 1.

**O7.** What does `omp_get_num_threads()` return outside a parallel region? Inside a region with 4 threads?

**Answer:** 1 outside, and 4 inside.

**O8.** When would you use `omp_get_max_threads()` instead of `omp_get_num_threads()`?

**Answer:** Outside a parallel region, to get how many threads the next region will use, for example to size a per-thread array.

**O9.** Parallelize this loop:
```c
for (int i = 0; i < N; i++) a[i] = b[i] + c[i];
```

**Answer:**
```c
#pragma omp parallel for
for (int i = 0; i < N; i++) a[i] = b[i] + c[i];
```

**O10.** Describe the fork-join model in three or fewer sentences.

**Answer:** One thread runs the serial code and forks a team of threads at a parallel region. At the end of the region the team joins back, and only the original thread continues.

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

**Answer:** `thread_id` and `num_threads` are shared, so threads overwrite each other's `thread_id` and may print the wrong number. Declare them inside the region, or make them `private`.

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

**Answer:** `private`: each copy is uninitialized, so it prints garbage. `firstprivate`: every thread prints 15.

**O13.** What is wrong with this sum, and how do you fix it with `critical`?
```c
double sum = 0;
#pragma omp parallel for
for (int i = 0; i < N; i++) sum += a[i];
```

**Answer:** There's a race on `sum`, so updates get lost. A correct but slow fix:
```c
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    #pragma omp critical
    sum += a[i];
}
```

**O14.** The fix in O13 is correct but slow. Rewrite it so each thread keeps a local partial sum and enters the critical section only once.

**Answer:**
```c
double sum = 0;
#pragma omp parallel
{
    double partial = 0;
    #pragma omp for
    for (int i = 0; i < N; i++) partial += a[i];
    #pragma omp critical
    sum += partial;
}
```

**O15.** Compute the same sum as O13 without `critical`. Allocate an array `partial` with one entry per thread, sized with `omp_get_max_threads()`. In the parallel loop, each thread adds into its own entry, `partial[omp_get_thread_num()]`. After the parallel region, add the entries together serially to get `sum`.

**Answer:**
```c
int T = omp_get_max_threads();
double *partial = calloc(T, sizeof(double));
#pragma omp parallel
{
    int id = omp_get_thread_num();
    #pragma omp for
    for (int i = 0; i < N; i++) partial[id] += a[i];
}
double sum = 0;
for (int t = 0; t < T; t++) sum += partial[t];
free(partial);
```

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

**Answer:** In the first version, only the outer `i` loop is split. In the second, `collapse(2)` merges both loops into one N × M iteration space, and that combined space is split.

**O17.** Use `collapse(2)` to parallelize the initialization of an N-by-M matrix to zero.

**Answer:**
```c
#pragma omp parallel for collapse(2)
for (int i = 0; i < N; i++)
    for (int j = 0; j < M; j++)
        A[i][j] = 0.0;
```

**O18.** Explain these OpenMP schedules: `static`, `dynamic`, and `guided`.

**Answer:**
- `static`: iterations are split into equal chunks and assigned to threads before the loop starts. Lowest overhead; best when every iteration costs the same.
- `dynamic`: each thread is assigned a chunk (default size 1), and requests the next one when it finishes. Balances uneven work, at the cost of more overhead.
- `guided`: like `dynamic`, but chunks start large and shrink as the loop runs out of work. Fewer chunk requests than `dynamic`, while still balancing the end of the loop.

**O19.** Parallelize this loop with a dynamic schedule and chunks of 4:
```c
for (int i = 0; i < N; i++) {
    double s = 0.0;
    for (int j = 0; j < i; j++) s += 1.0 / (j + 1);
    a[i] = s;
}
```

**Answer:**
```c
#pragma omp parallel for schedule(dynamic, 4)
for (int i = 0; i < N; i++) {
    double s = 0.0;
    for (int j = 0; j < i; j++) s += 1.0 / (j + 1);
    a[i] = s;
}
```

**O20.** With `schedule(static)`, 4 threads, and 16 iterations (0 to 15), which iterations does thread 1 get?

**Answer:** 4 through 7.

**O21.** Parallelize this loop, time it with OpenMP's timer, and print the elapsed time.
```c
for (int i = 0; i < N; i++) a[i] = b[i] * 2.0;
```

**Answer:**
```c
double t0 = omp_get_wtime();
#pragma omp parallel for
for (int i = 0; i < N; i++) a[i] = b[i] * 2.0;
double elapsed = omp_get_wtime() - t0;
printf("%f s\n", elapsed);
```

**O22.** Inside a parallel region, print "starting" exactly once, from thread 0. (Hint: this can be done more than one way.)

**Answer:**
```c
#pragma omp parallel
{
    #pragma omp master
    printf("starting\n");
}
```
Or, inside the region: `if (omp_get_thread_num() == 0) printf("starting\n");`

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

**Answer:**
```c
int vals[64];   /* assume at most 64 threads */
#pragma omp parallel
{
    int t = omp_get_thread_num();
    int T = omp_get_num_threads();
    vals[t] = t * t;
    #pragma omp barrier
    printf("thread %d sees %d\n", t, vals[(t + 1) % T]);
}
```

**O24.** Fill `a[i] = i * i` and `b[i] = 2 * i` for i from 0 to N−1 at the same time, with one thread filling each array. (Hint: use sections.)

**Answer:**
```c
#pragma omp parallel sections
{
    #pragma omp section
    for (int i = 0; i < N; i++) a[i] = i * i;
    #pragma omp section
    for (int i = 0; i < N; i++) b[i] = 2 * i;
}
```

**O25.** This code computes `i * i` in parallel for i from 0 to 7. What prints?
```c
#pragma omp parallel for ordered
for (int i = 0; i < 8; i++) {
    int sq = i * i;
    #pragma omp ordered
    printf("%d\n", sq);
}
```

**Answer:** 0, 1, 4, 9, 16, 25, 36, 49, in that order, one per line.

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

**Answer:** 4.

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

**Answer:** All unnamed `critical` sections share one lock, so only one thread can be in either section at a time. A thread updating `misses` has to wait for a thread updating `hits`, even though they touch different variables. Give each section a name, so each gets its own lock:
```c
int hits = 0, misses = 0;
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    if (a[i] > 0) {
        #pragma omp critical(hits)
        hits++;
    } else {
        #pragma omp critical(misses)
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

**Answer:**
```c
int count = 0;
#pragma omp parallel
{
    int mine = 0;
    #pragma omp for schedule(dynamic, 100)
    for (int i = 2; i < N; i++)
        if (is_prime(i)) mine++;
    #pragma omp critical
    count += mine;
}
```
Dynamic (or guided), since cost grows with i.

**O29.** Estimate pi with OpenMP by integrating 4/(1+x²) from 0 to 1 using N steps. Use partial sums.

**Answer:**
```c
long N = 1000000;
double h = 1.0 / N, pi = 0.0;
#pragma omp parallel
{
    double partial = 0.0;
    #pragma omp for
    for (long i = 0; i < N; i++) {
        double x = (i + 0.5) * h;
        partial += 4.0 / (1.0 + x * x);
    }
    #pragma omp critical
    pi += partial * h;
}
printf("pi = %f\n", pi);
```

**O30.** Parallelize the matrix-vector multiply `y = A * x` for an N-by-N matrix. Which loop do you parallelize, and why that one?

**Answer:**
```c
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    double s = 0.0;
    for (int j = 0; j < N; j++) s += A[i][j] * x[j];
    y[i] = s;
}
```
Parallelize the outer `i` loop. Each thread computes whole rows, and each row writes only its own `y[i]`, so no two threads write to the same variable. If you parallelized the inner `j` loop instead, all threads would add into the same `s`, which is a race condition.

## Combined (15)

**C1.** What is a hybrid MPI + OpenMP program, and how do you compile one?

**Answer:** MPI distributes processes across nodes, and inside each process OpenMP spawns threads across that node's cores. Compile with `mpicc -fopenmp prog.c -o prog`.

**C2.** Write a hybrid hello where each MPI rank starts 3 OpenMP threads, and each thread prints its rank and thread number. With `-np 2`, how many lines print?

**Answer:**
```c
MPI_Init(&argc, &argv);
MPI_Comm_rank(MPI_COMM_WORLD, &rank);
omp_set_num_threads(3);
#pragma omp parallel
{
    printf("rank %d, thread %d\n", rank, omp_get_thread_num());
}
MPI_Finalize();
```
6 lines (2 ranks × 3 threads), in no guaranteed order.

**C3.** You run a hybrid MPI + OpenMP program on 4 nodes with 16 cores each. Give two ways to choose the number of MPI ranks and the number of OpenMP threads per rank so that every core is used, but no node runs more threads than it has cores.

**Answer:** Any choice where (ranks per node) × (threads per rank) = 16 works. For example:
- 4 ranks × 16 threads (one rank per node)
- 8 ranks × 8 threads (two ranks per node)
- 64 ranks × 1 thread (pure MPI)

**C4.** A program has a deadlock in its MPI section and a race condition in its OpenMP section. Explain the difference in symptoms.

**Answer:** Deadlock: the program hangs. Race: it finishes, but the results are wrong or vary between runs.

**C5.** A large array of doubles has been split across the MPI ranks. Each rank holds its piece in an array `local` of length `chunk`. Find the sum of the whole array in two steps:
1. Inside each rank, use OpenMP threads to add up `local` into a variable `mysum`. Each thread should sum its share of the loop into its own `partial` variable, then add `partial` into `mysum` once, inside a `critical` section (as in O14).
2. Use `MPI_Reduce` to add up every rank's `mysum` into `total` on rank 0. Rank 0 prints `total`.

**Answer:**
```c
double mysum = 0.0, total = 0.0;

/* Step 1: this rank's threads sum its piece of the array */
#pragma omp parallel
{
    double partial = 0.0;              /* each thread has its own copy */
    #pragma omp for
    for (int i = 0; i < chunk; i++) partial += local[i];
    #pragma omp critical
    mysum += partial;                  /* one update per thread */
}

/* Step 2: add every rank's mysum together onto rank 0 */
MPI_Reduce(&mysum, &total, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
if (rank == 0) printf("total = %f\n", total);
```
OpenMP splits the work among threads inside one rank, and MPI combines the results across ranks.

**C6.** Estimate pi by integrating 4/(1+x²) from 0 to 1 using N steps, the same calculation as M39 and O29, but now with both MPI and OpenMP:
1. Split the N steps across ranks with a block distribution (as in M29): each rank handles steps `start` through `end − 1`, where `start = rank * N / size` and `end = (rank + 1) * N / size`.
2. Inside each rank, use OpenMP threads to sum that block, with each thread keeping its own partial sum (as in O29). Store the rank's result in `mine`.
3. Use `MPI_Reduce` to add every rank's `mine` into `pi` on rank 0. Rank 0 prints `pi`.

**Answer:**
```c
long N = 1000000;
double h = 1.0 / N, mine = 0.0, pi = 0.0;

/* Step 1: this rank's block of steps */
long start = rank * N / size, end = (rank + 1) * N / size;

/* Step 2: this rank's threads sum the block */
#pragma omp parallel
{
    double partial = 0.0;
    #pragma omp for
    for (long i = start; i < end; i++) {
        double x = (i + 0.5) * h;
        partial += 4.0 / (1.0 + x * x);
    }
    #pragma omp critical
    mine += partial * h;
}

/* Step 3: add every rank's result together onto rank 0 */
MPI_Reduce(&mine, &pi, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
if (rank == 0) printf("pi = %f\n", pi);
```

**C7.** Rank 0 reads an integer N from the user. Every rank then runs an OpenMP loop over its share of the iterations 0 through N−1. Write it in three steps:
1. Rank 0 reads N with `scanf`. At this point, the other ranks don't know N.
2. Use `MPI_Bcast` to send N from rank 0 to every rank (as in M17).
3. Each rank loops over its share with a cyclic distribution (as in M28): rank r handles i = r, r + size, r + 2·size, and so on. Parallelize this loop with OpenMP, and print the rank, thread number, and `i` for each iteration.

**Answer:**
```c
int N;

/* Step 1: only rank 0 reads the input */
if (rank == 0) scanf("%d", &N);

/* Step 2: every rank gets N from rank 0 */
MPI_Bcast(&N, 1, MPI_INT, 0, MPI_COMM_WORLD);

/* Step 3: cyclic share of the iterations, split among this rank's threads */
#pragma omp parallel for
for (int i = rank; i < N; i += size)
    printf("rank %d, thread %d: i = %d\n", rank, omp_get_thread_num(), i);
```
For example, with N = 6 and 2 ranks, rank 0 prints i = 0, 2, 4 and rank 1 prints i = 1, 3, 5. The lines appear in no guaranteed order.

**C8.** In a hybrid program, every rank runs the OpenMP loop below. Ranks can finish at different times, and the job isn't done until the slowest rank finishes, so the job's runtime is the slowest rank's time. Measure it in three steps:
1. Use `MPI_Barrier` so all ranks start together, then record the start time with `MPI_Wtime()` (as in M23).
2. Run the loop, then compute this rank's `elapsed` time.
3. Use `MPI_Reduce` with `MPI_MAX` to put the largest `elapsed` across all ranks into `slowest` on rank 0. Rank 0 prints `slowest`.
```c
#pragma omp parallel for
for (int i = 0; i < N; i++) a[i] = b[i] * 2.0;
```
Then answer: would the OpenMP timer, `omp_get_wtime()`, also work here? Why or why not?

**Answer:**
```c
double slowest;

/* Step 1: start all ranks together */
MPI_Barrier(MPI_COMM_WORLD);
double t0 = MPI_Wtime();

/* Step 2: run the loop and time it on this rank */
#pragma omp parallel for
for (int i = 0; i < N; i++) a[i] = b[i] * 2.0;
double elapsed = MPI_Wtime() - t0;

/* Step 3: the largest elapsed time across ranks goes to rank 0 */
MPI_Reduce(&elapsed, &slowest, 1, MPI_DOUBLE, MPI_MAX, 0, MPI_COMM_WORLD);
if (rank == 0) printf("runtime: %f s\n", slowest);
```
Yes, `omp_get_wtime()` would also work: both timers return wall-clock time in seconds, and each rank subtracts two readings taken on itself. `MPI_Wtime()` is the usual choice in MPI code, and it still works if the program is compiled without OpenMP, for example to compare against an MPI-only run.

**C9.** The ranks are arranged in a ring (as in M24). Each rank sends its own rank number to its right neighbor and receives a number from its left neighbor, then uses that number to fill an array. Write it in three steps:
1. Compute `left` and `right` for this rank.
2. Send `rank` to `right` and receive from `left` into `int in`. Every rank sends and receives at the same time, so use `MPI_Sendrecv` to avoid the deadlock from M30 and M31.
3. Use OpenMP to fill `a[i] = in * i` for i from 0 to N−1.

**Answer:**
```c
/* Step 1: neighbors in the ring */
int right = (rank + 1) % size, left = (rank - 1 + size) % size;

/* Step 2: send to the right and receive from the left in one call */
int out = rank, in;
MPI_Sendrecv(&out, 1, MPI_INT, right, 0,
             &in,  1, MPI_INT, left,  0,
             MPI_COMM_WORLD, MPI_STATUS_IGNORE);

/* Step 3: this rank's threads fill the array */
#pragma omp parallel for
for (int i = 0; i < N; i++) a[i] = in * i;
```
For example, with 4 ranks, rank 0 receives 3 from its left neighbor, so its array is 0, 3, 6, 9, and so on.

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

**Answer:**

| Run | Workers | Speedup | Efficiency |
|---|---|---|---|
| 4 ranks | 4 | 80 / 22 ≈ 3.64 | 3.64 / 4 ≈ 0.91 |
| 4 ranks × 4 threads | 16 | 80 / 7 ≈ 11.4 | 11.4 / 16 ≈ 0.71 |

The 16-worker run is faster overall, but each worker does less useful work, so efficiency drops.

**C11.** A program takes 120 s to run serially. You run it with 8 MPI ranks and measure a parallel efficiency of 0.75. How long did the parallel run take? (Hint: find the speedup first.)

**Answer:** Speedup = efficiency × workers = 0.75 × 8 = 6. Parallel time = serial time ÷ speedup = 120 / 6 = 20 s.

**C12.** A program takes 64 s to run serially. Here are its parallel run times:

| Workers | Time |
|---|---|
| 2 | 33 s |
| 4 | 17 s |
| 8 | 10 s |
| 16 | 9 s |

Compute the speedup and efficiency for each run. Is it worth going from 8 workers to 16?

**Answer:**

| Workers | Speedup | Efficiency |
|---|---|---|
| 2 | 64 / 33 ≈ 1.94 | 1.94 / 2 ≈ 0.97 |
| 4 | 64 / 17 ≈ 3.76 | 3.76 / 4 ≈ 0.94 |
| 8 | 64 / 10 = 6.4 | 6.4 / 8 = 0.80 |
| 16 | 64 / 9 ≈ 7.11 | 7.11 / 16 ≈ 0.44 |

Probably not. Doubling the workers from 8 to 16 only cuts the time from 10 s to 9 s, and efficiency falls to 0.44, so more than half of the 16 workers' total time is wasted.

**C13.** A program takes 200 s to run serially. On a cluster with 64 cores, you try two layouts:
- 64 MPI ranks × 1 thread each: 5 s
- 8 MPI ranks × 8 OpenMP threads each: 4 s

Compute the speedup and efficiency for each. Which layout uses the cores better?

**Answer:**

| Layout | Workers | Speedup | Efficiency |
|---|---|---|---|
| 64 × 1 | 64 | 200 / 5 = 40 | 40 / 64 ≈ 0.63 |
| 8 × 8 | 64 | 200 / 4 = 50 | 50 / 64 ≈ 0.78 |

The 8 × 8 hybrid layout. Both use all 64 cores, but it finishes sooner, so its efficiency is higher.

**C14.** You don't know a program's serial time. A run with 4 workers takes 30 s, with a parallel efficiency of 0.8. What is the serial time? (Hint: use the rearranged formulas.)

**Answer:** Speedup = efficiency × workers = 0.8 × 4 = 3.2. Serial time = speedup × parallel time = 3.2 × 30 = 96 s.

**C15.** A program takes 96 s to run serially. You compare two runs:
- Run A: 16 workers, 10 s
- Run B: 8 workers, 14 s

(a) Which run finishes first?
(b) Compute the speedup and efficiency of each run.
(c) You have 16 cores and need to run the program twice, on two different inputs. Is it faster to run A twice, one after the other, or to run two copies of B at the same time, each on 8 cores?

**Answer:**
(a) Run A, at 10 s versus 14 s.

(b)

| Run | Workers | Speedup | Efficiency |
|---|---|---|---|
| A | 16 | 96 / 10 = 9.6 | 9.6 / 16 = 0.60 |
| B | 8 | 96 / 14 ≈ 6.86 | 6.86 / 8 ≈ 0.86 |

(c) Two copies of B at the same time. Running A twice takes 10 + 10 = 20 s, while two copies of B side by side finish in 14 s. The faster run isn't always the better choice when you have more than one job to run.
