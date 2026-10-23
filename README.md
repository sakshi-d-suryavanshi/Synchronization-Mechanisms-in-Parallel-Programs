# Synchronization Mechanisms in Parallel Programs

## 1. Introduction

Synchronization is an important concept in parallel programming. When multiple threads access and modify the same shared variable simultaneously, race conditions can occur and produce incorrect results.

This experiment implements and compares different synchronization mechanisms used in shared-memory parallel programs:

1. Race condition (no synchronization)
2. Critical section
3. Atomic operation
4. Barrier
5. Semaphore
6. Lock-based synchronization

The purpose of this experiment is to study the correctness, execution time, and scalability of different synchronization mechanisms as the number of threads increases.

The experiment uses C programming, OpenMP, POSIX threads, and semaphores. A shared counter is incremented repeatedly to observe the effects of synchronization.

---

# 2. Objectives

- To understand the need for synchronization in parallel programs.
- To demonstrate race conditions caused by unsynchronized access to shared variables.
- To implement critical sections and atomic operations using OpenMP.
- To understand barrier synchronization between threads.
- To implement semaphore-based synchronization using POSIX threads.
- To compare execution times for different synchronization mechanisms.
- To evaluate correctness and scalability using different thread counts.

---

# 3. Experimental Configuration

| Parameter | Configuration |
|---|---|
| Programming Language | C |
| Operating System | Ubuntu |
| Compiler | GCC |
| Parallel Programming Library | OpenMP |
| Threading Library | POSIX Threads (pthreads) |
| Synchronization Libraries | OpenMP locks and POSIX semaphores |
| Total Counter Increments | 10,000,000 |
| Thread Counts | 1, 2, 4, 8 |
| Semaphore Threads | 2 |
| Increments per Semaphore Thread | 100,000 |
| Initial Counter Value | 0 |
| Expected Benchmark Counter | 10,000,000 |

---

# 4. Program 1 - Race Condition (No Synchronization)

## 4.1 Description

The race-condition program demonstrates what happens when multiple threads update the same shared variable without synchronization.

The program uses an OpenMP parallel loop to increment a shared counter 10,000,000 times.

```c
#pragma omp parallel for
for (long i = 0; i < 10000000; i++) {
    counter++;
}
```

The operation `counter++` involves reading, incrementing, and writing the shared variable. Multiple threads may interfere with one another, causing some updates to be lost.

## 4.2 Execution

Compile the program:

```bash
gcc race.c -fopenmp -o race
```

Run the program:

```bash
./race
```

## 4.3 Result

Recorded output:

```text
Counter = 6939237
```

| Parameter | Result |
|---|---:|
| Expected Counter | 10,000,000 |
| Obtained Counter | 6,939,237 |
| Status | Incorrect |

## 4.4 Observation

The final counter value is smaller than the expected value because multiple threads update the shared variable without protection.

This demonstrates a race condition and shows why synchronization is necessary for correct shared-data updates.

---

# 5. Program 2 - Critical Section

## 5.1 Description

A critical section ensures that only one thread at a time executes a protected block of code.

OpenMP provides the `critical` directive to protect shared data from simultaneous updates.

```c
#pragma omp critical
{
    counter++;
}
```

Other threads attempting to enter the same critical section must wait until the current thread exits.

## 5.2 Execution

Compile:

```bash
gcc critical.c -fopenmp -o critical
```

Run:

```bash
./critical
```

## 5.3 Result

Recorded output:

```text
Counter = 10000000
```

| Parameter | Result |
|---|---:|
| Expected Counter | 10,000,000 |
| Obtained Counter | 10,000,000 |
| Status | Correct |

## 5.4 Observation

The critical section produces the correct result because it prevents multiple threads from updating the counter simultaneously.

However, the mechanism introduces waiting overhead because threads must enter the protected region one at a time.

---

# 6. Program 3 - Atomic Operation

## 6.1 Description

An atomic operation protects a simple shared-variable update by ensuring that the operation is performed indivisibly.

OpenMP provides the `atomic` directive for this purpose.

```c
#pragma omp atomic
counter++;
```

Unlike a general critical section, an atomic directive is designed for supported simple memory operations rather than arbitrary blocks of code.

## 6.2 Execution

Compile:

```bash
gcc atomic.c -fopenmp -o atomic
```

Run:

```bash
./atomic
```

## 6.3 Result

Recorded output:

```text
Counter = 10000000
```

| Parameter | Result |
|---|---:|
| Expected Counter | 10,000,000 |
| Obtained Counter | 10,000,000 |
| Status | Correct |

## 6.4 Observation

The atomic operation prevents lost updates and produces the correct counter value.

It generally performs better than a critical section for simple shared-variable updates, although actual performance depends on hardware, compiler implementation, and thread contention.

---

# 7. Program 4 - Barrier Synchronization

## 7.1 Description

A barrier is a synchronization point at which all threads in the parallel region must arrive before any of them can proceed beyond it.

The OpenMP directive used is:

```c
#pragma omp barrier
```

The program prints a message before and after the barrier to demonstrate synchronization among four threads.

## 7.2 Execution

Compile:

```bash
gcc barrier.c -fopenmp -o barrier
```

Run using four threads:

```bash
OMP_NUM_THREADS=4 ./barrier
```

## 7.3 Result

Recorded output:

```text
Thread 3 before barrier
Thread 1 before barrier
Thread 2 before barrier
Thread 0 before barrier
Thread 1 after barrier
Thread 3 after barrier
Thread 0 after barrier
Thread 2 after barrier
```

| Parameter | Result |
|---|---|
| Number of Threads | 4 |
| Before-Barrier Messages | 4 |
| After-Barrier Messages | 4 |
| Status | Successful |

## 7.4 Observation

All four threads print their messages before and after the barrier.

The order of messages may vary because threads execute independently. The barrier ensures that no thread proceeds past it until all threads have reached that synchronization point.

A barrier synchronizes execution phases but does not directly provide mutual exclusion for shared-variable updates.

---

# 8. Program 5 - Semaphore Synchronization

## 8.1 Description

A semaphore is a synchronization mechanism used to control access to a shared resource.

This program uses POSIX threads and a binary semaphore initialized to 1.

```c
sem_init(&semaphore, 0, 1);
```

The following operations control access to the shared counter:

```c
sem_wait(&semaphore);
counter++;
sem_post(&semaphore);
```

- `sem_wait()` acquires access to the protected section.
- `counter++` increments the shared counter.
- `sem_post()` releases access to the protected section.

Two threads perform 100,000 increments each.

## 8.2 Execution

Compile:

```bash
gcc semaphore.c -pthread -o semaphore
```

Run:

```bash
./semaphore
```

## 8.3 Result

Recorded output:

```text
Counter = 200000
```

| Parameter | Result |
|---|---:|
| Number of Threads | 2 |
| Increments per Thread | 100,000 |
| Expected Counter | 200,000 |
| Obtained Counter | 200,000 |
| Status | Correct |

## 8.4 Observation

The semaphore protects the shared counter and ensures that only one thread enters the protected section at a time.

The final value of 200,000 confirms that both threads completed their increments correctly.

---

# 9. Program 6 - Performance and Scalability Comparison

## 9.1 Description

The comparison program evaluates four counter-update strategies:

1. None (no synchronization)
2. Critical section
3. Atomic operation
4. OpenMP lock

Each mechanism performs 10,000,000 counter increments.

The benchmark is executed using 1, 2, 4, and 8 threads.

Execution time is measured using:

```c
double start = omp_get_wtime();

long result = fn(nthreads);

double elapsed = omp_get_wtime() - start;
```

Correctness is checked by comparing the final counter with the expected value of 10,000,000.

## 9.2 Compilation and Execution

Compile:

```bash
gcc compare_sync.c -fopenmp -o compare_sync
```

Run:

```bash
./compare_sync
```

The program automatically tests all four thread counts and prints the final counter, correctness status, and execution time.

## 9.3 Benchmark Results

The following table contains the actual recorded results from the experiment.

| Mechanism | Threads | Final Count | Correctness | Time (s) |
|---|---:|---:|---|---:|
| None | 1 | 10,000,000 | Correct | 0.0050 |
| Critical | 1 | 10,000,000 | Correct | 0.0959 |
| Atomic | 1 | 10,000,000 | Correct | 0.0545 |
| Lock | 1 | 10,000,000 | Correct | 0.0898 |
| None | 2 | 5,567,660 | Incorrect | 0.0073 |
| Critical | 2 | 10,000,000 | Correct | 0.2994 |
| Atomic | 2 | 10,000,000 | Correct | 0.1125 |
| Lock | 2 | 10,000,000 | Correct | 0.3407 |
| None | 4 | 5,075,835 | Incorrect | 0.0067 |
| Critical | 4 | 10,000,000 | Correct | 0.2912 |
| Atomic | 4 | 10,000,000 | Correct | 0.1215 |
| Lock | 4 | 10,000,000 | Correct | 0.3623 |
| None | 8 | 4,516,116 | Incorrect | 0.0085 |
| Critical | 8 | 10,000,000 | Correct | 0.2896 |
| Atomic | 8 | 10,000,000 | Correct | 0.1204 |
| Lock | 8 | 10,000,000 | Correct | 0.3552 |

## 9.4 Observations

### None - No Synchronization

The program produces the correct result with one thread but incorrect results with 2, 4, and 8 threads.

This happens because multiple threads update the shared counter simultaneously, causing lost updates.

It has the lowest recorded execution time, but the result is unreliable.

### Critical Section

The critical-section implementation produces the correct counter for every tested thread count.

However, it has higher execution time because threads must wait to enter the protected region.

### Atomic Operation

The atomic implementation produces the correct result for every tested thread count.

It is faster than both Critical and Lock in all the recorded benchmark runs, making it the most efficient of these three synchronized counter-update approaches in this experiment.

### OpenMP Lock

The lock implementation produces the correct result for every tested thread count.

Its execution time is relatively high because each increment requires acquiring and releasing the lock.

## 9.5 Performance Comparison at Eight Threads

| Mechanism | Execution Time (s) | Correctness |
|---|---:|---|
| None | 0.0085 | Incorrect |
| Critical | 0.2896 | Correct |
| Atomic | 0.1204 | Correct |
| Lock | 0.3552 | Correct |

At eight threads, Atomic has the lowest execution time among the three synchronized methods in this run.

The unsynchronized method is faster but does not produce the expected counter value.

---

# 10. Correctness Verification

The expected final counter for the comparison benchmark is:

```text
Expected Counter = 10000000
```

The synchronization mechanisms are verified by comparing their actual final counter values with this expected value.

| Mechanism | Expected Counter | Observed Result | Verification |
|---|---:|---|---|
| None (1 thread) | 10,000,000 | 10,000,000 | Correct |
| None (2 threads) | 10,000,000 | 5,567,660 | Incorrect |
| None (4 threads) | 10,000,000 | 5,075,835 | Incorrect |
| None (8 threads) | 10,000,000 | 4,516,116 | Incorrect |
| Critical | 10,000,000 | 10,000,000 | Correct |
| Atomic | 10,000,000 | 10,000,000 | Correct |
| Lock | 10,000,000 | 10,000,000 | Correct |

The standalone semaphore program produced the expected value of 200,000, and the barrier program successfully demonstrated synchronization between four threads.

The results confirm that synchronization is necessary to maintain correctness when multiple threads modify shared data.

---

# 11. Technologies Used

- C Programming Language
- GCC Compiler
- OpenMP
- POSIX Threads (pthreads)
- POSIX Semaphores
- OpenMP Locks
- Ubuntu
- Linux Terminal

---

# 12. Project Structure

```text
sync_experiment/
│
├── race.c
├── critical.c
├── atomic.c
├── barrier.c
├── semaphore.c
├── compare_sync.c
│
├── race
├── critical
├── atomic
├── barrier
├── semaphore
└── compare_sync
```

The `.c` files contain the source code, while the executable files are generated during compilation.

---

# 13. How to Run the Experiment

## 13.1 Install Required Tools

```bash
sudo apt update
sudo apt install build-essential -y
```

## 13.2 Create the Project Directory

```bash
mkdir -p ~/sync_experiment
cd ~/sync_experiment
```

## 13.3 Compile the Programs

Race condition:

```bash
gcc race.c -fopenmp -o race
```

Critical section:

```bash
gcc critical.c -fopenmp -o critical
```

Atomic operation:

```bash
gcc atomic.c -fopenmp -o atomic
```

Barrier:

```bash
gcc barrier.c -fopenmp -o barrier
```

Semaphore:

```bash
gcc semaphore.c -pthread -o semaphore
```

Comparison benchmark:

```bash
gcc compare_sync.c -fopenmp -o compare_sync
```

## 13.4 Run the Programs

```bash
./race
./critical
./atomic
OMP_NUM_THREADS=4 ./barrier
./semaphore
./compare_sync
```

The comparison program automatically tests 1, 2, 4, and 8 threads.

---


# 14. Conclusion

This experiment demonstrates the importance of synchronization in parallel programming.

The race-condition program produced incorrect counter values when multiple threads updated shared data without synchronization. The critical section, atomic operation, and OpenMP lock produced the correct result of 10,000,000 increments for all tested thread counts. The semaphore-based program also produced the correct result of 200,000 increments using two threads.

The benchmark results show that synchronization affects both execution time and scalability. The critical section and lock introduce waiting overhead, while the atomic operation performs better for simple shared-variable updates in the recorded runs. The barrier demonstrates how threads can be synchronized between different phases of execution.

Overall, the experiment shows that synchronization mechanisms help maintain correctness in parallel programs, and choosing an appropriate mechanism can reduce unnecessary overhead while improving performance.
