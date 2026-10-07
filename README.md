# PGC-02: Multithreaded Programming Using Pthreads and OpenMP

> **Course:** Parallel and GPU Computing (PGC)
> **Module / Repo:** `PGC-02-Multithread-Pthreads-OpenMP`
> **Laboratory Assignment:** *Develop Multithreaded Programs Using Parallel Programming Libraries to Understand Thread Creation, Management, and Coordination*

---

## Table of Contents

* [1. Aim & Objectives](#1-aim--objectives)
* [2. Theoretical Background](#2-theoretical-background)

  * [Sequential Execution (Single Worker)](#sequential-execution-single-worker)
  * [Multithreaded Parallel Execution (Multiple Workers)](#multithreaded-parallel-execution-multiple-workers)
* [3. Software Environment & System Setup](#3-software-environment--system-setup)
* [4. Part A — POSIX Threads (Pthreads)](#4-part-a--posix-threads-pthreads)

  * [Step 1: Single Thread Creation (`thread1.c`)](#step-1-single-thread-creation-thread1c)
  * [Step 2: Spawning Multiple Threads (`thread2.c`)](#step-2-spawning-multiple-threads-thread2c)
  * [Step 3: Work Distribution & Array Chunking (`thread_sum.c`)](#step-3-work-distribution--array-chunking-thread_sumc)
  * [Step 4: Concurrency Hazards & Race Condition (`race.c`)](#step-4-concurrency-hazards--race-condition-racec)
  * [Step 5: Mutual Exclusion Using Mutex (`mutex.c`)](#step-5-mutual-exclusion-using-mutex-mutexc)
* [5. Part B — OpenMP Compiler Directives](#5-part-b--openmp-compiler-directives)

  * [Step 6: OpenMP Parallel Region & Thread Identification (`omp1.c`)](#step-6-openmp-parallel-region--thread-identification-omp1c)
  * [Step 7: Work-Sharing Loops & Reduction (`omp_sum.c`)](#step-7-work-sharing-loops--reduction-omp_sumc)
  * [Step 8: OpenMP Data Hazards & Race Condition (`omp_race.c`)](#step-8-openmp-data-hazards--race-condition-omp_racec)
  * [Step 9: OpenMP Critical Section Synchronization (`omp_critical.c`)](#step-9-openmp-critical-section-synchronization-omp_criticalc)
  * [Step 10: Phased Barrier Coordination (`omp_barrier.c`)](#step-10-phased-barrier-coordination-omp_barrierc)
* [6. Part C — Performance Analysis & Empirical Scalability](#6-part-c--performance-analysis--empirical-scalability)

  * [Step 11: Single-Threaded Sequential Baseline (`sequential.c`)](#step-11-single-threaded-sequential-baseline-sequentialc)
  * [Step 12 & 13: Pthreads Scalability Benchmarks (`pthread_perf.c`)](#step-12--13-pthreads-scalability-benchmarks-pthread_perfc)
  * [Step 14: OpenMP Scalability Benchmarks (`omp_perf.c`)](#step-14-openmp-scalability-benchmarks-omp_perfc)
  * [Step 15: Measured Execution Time Comparison Table](#step-15-measured-execution-time-comparison-table)
  * [Step 16: Speedup Calculation & Scaling Analysis](#step-16-speedup-calculation--scaling-analysis)
  * [Step 17: Parallel Efficiency Analysis](#step-17-parallel-efficiency-analysis)
  * [Step 18: Architectural Bottlenecks & Hardware Interpretation](#step-18-architectural-bottlenecks--hardware-interpretation)
* [7. Comprehensive Comparison: Pthreads vs. OpenMP](#7-comprehensive-comparison-pthreads-vs-openmp)
* [8. Terminology Glossary](#8-terminology-glossary)
* [9. Repository File Structure](#9-repository-file-structure)

---

## 1. Aim & Objectives

The purpose of this laboratory is to implement and study multithreaded programs using **POSIX Threads (Pthreads)** and **OpenMP (Open Multi-Processing)**. The experiments demonstrate how a computational problem can be divided among multiple threads and how these threads can be coordinated safely.

The major objectives of the experiment are:

1. **Thread Creation and Management:** Learn how individual threads are created, assigned a task, and terminated or joined by the main thread.
2. **Work Distribution:** Divide an input workload into smaller sections so that multiple threads can process different portions simultaneously.
3. **Data Hazards (Race Conditions):** Observe how concurrent access to shared variables can lead to inconsistent or incorrect results.
4. **Synchronization Primitives:** Apply mutexes and critical sections to prevent multiple threads from modifying shared data at the same time.
5. **Thread Coordination:** Understand how barriers can be used to synchronize threads between different stages of a computation.
6. **Empirical Scalability:** Compare execution time, speedup, and efficiency for different numbers of threads using a large numerical workload of **1,000,000,000 iterations**.

---

## 2. Theoretical Background

A **thread** represents an independent flow of execution within a process. Multiple threads belonging to the same process can access shared memory, which makes communication between threads efficient but also introduces synchronization challenges.

Multithreading is particularly useful for computationally intensive tasks where independent portions of the workload can be executed concurrently.

### Sequential Execution (Single Worker)

In a sequential program, instructions are executed one after another by a single execution stream. A new task generally starts only after the previous task has completed.

```text
Sequential Execution (1 Core / Single Worker):
Main Thread ─── Task 1 ─── Task 2 ─── Task 3 ─── Task 4 ─── Finish
```

Although sequential execution is simple to implement, it may take longer for workloads that contain many independent operations.

### Multithreaded Parallel Execution (Multiple Workers)

In a multithreaded program, the overall workload can be divided into smaller portions. Multiple threads can then process these portions concurrently, potentially reducing the total execution time.

```text
Multithreaded Parallel Architecture:
                ┌─── Thread 1 ─── Chunk 1 ───┐
                ├─── Thread 2 ─── Chunk 2 ───┤
Master Spawn ───┼─── Thread 3 ─── Chunk 3 ───┼─── Join & Reduction ─── Aggregated Result
                └─── Thread 4 ─── Chunk 4 ───┘
```

The actual improvement depends on factors such as the number of available CPU cores, workload size, memory bandwidth, synchronization overhead, and operating-system scheduling.

---

## 3. Software Environment & System Setup

| Component                      | Specification                                                         |
| :----------------------------- | :-------------------------------------------------------------------- |
| **Operating System**           | Windows 11 with WSL (Windows Subsystem for Linux - Ubuntu)            |
| **Terminal Host**              | `user@DESKTOP-9BP6J7D:~/parallel_lab$`                                |
| **Compiler**                   | GCC 15.2.0 (`gcc (Ubuntu 15.2.0-16ubuntu1) 15.2.0`)                   |
| **Compilation Flags**          | `-pthread` (POSIX Threads), `-fopenmp` (OpenMP), `-O2` (Optimization) |
| **Parallel Hardware Capacity** | **32 Logical Threads** detected & utilized                            |

### System & Environment Verification

The WSL environment was first prepared and the compiler installation was verified before executing the multithreading experiments.

```bash
wsl
mkdir -p ~/parallel_lab && cd ~/parallel_lab
gcc --version
gcc -fopenmp --version
```

The GCC version check confirms that the required C compiler is available, while the OpenMP compiler check verifies support for OpenMP compilation.

<img width="1145" height="335" alt="00_wsl_ubuntu_gcc_environment" src="https://github.com/user-attachments/assets/ed4b2818-ed4c-4e3c-abcb-d99ee9815d3f" />

---

## 4. Part A — POSIX Threads (Pthreads)

**POSIX Threads (Pthreads)** is a thread programming interface that gives the programmer direct control over individual threads. Thread creation, argument passing, synchronization, and completion are explicitly handled by the programmer.

Important Pthreads functions used in these experiments include `pthread_create()` for creating a thread, `pthread_join()` for waiting for a thread to finish, and mutex operations for protecting shared data.

---

### Step 1: Single Thread Creation (`thread1.c`)

* **Concept:** This program creates one additional worker thread that executes `thread_function`. The main thread waits for the worker to finish by calling `pthread_join()`.
* **Source File:** [`thread1.c`](./thread1.c)
* **Compile & Run:**

  ```bash
  gcc thread1.c -o thread1 -pthread
  ./thread1
  ```

The `-pthread` compiler option enables the required POSIX thread support while compiling and linking the program.

This experiment provides the basic understanding required before moving to multiple concurrent threads.

<img width="635" height="82" alt="01_pthread_single_thread" src="https://github.com/user-attachments/assets/1eb45469-8ed3-4d27-8e7f-8f1201057138" />

---

### Step 2: Spawning Multiple Threads (`thread2.c`)

* **Concept:** Four worker threads are created using a loop. Each thread receives its own rank or identifier so that it can be distinguished during execution.

* **Source File:** [`thread2.c`](./thread2.c)

* **Compile & Run:**

  ```bash
  gcc thread2.c -o thread2 -pthread
  ./thread2
  ```

* **Concurrency Insight:** The four threads are not guaranteed to execute in the same order in which they are created. For example, an output sequence such as `1 -> 3 -> 2 -> 4` may be observed.

The reason is that thread execution is controlled by the operating-system scheduler. Depending on CPU availability and scheduling decisions, a different thread may execute first during another run.

Therefore, the order of the output should not be treated as a fixed sequence.

<img width="756" height="130" alt="02_pthread_multiple_threads" src="https://github.com/user-attachments/assets/1a2c376a-f4f9-4935-bf64-4f9083bc8f8f" />

---

### Step 3: Work Distribution & Array Chunking (`thread_sum.c`)

* **Concept:** An 8-element array `[10, 20, 30, 40, 50, 60, 70, 80]` is divided into four continuous chunks. Each thread processes two elements and calculates a local partial sum.
* **Source File:** [`thread_sum.c`](./thread_sum.c)
* **Compile & Run:**

  ```bash
  gcc thread_sum.c -o thread_sum -pthread
  ./thread_sum
  ```

Each thread works independently on its assigned portion of the array. After all worker threads finish, the main thread combines the partial results.

* **Mathematical Invariant Check:**

$$
\text{Total} = 30 + 70 + 110 + 150 = 360
$$

This experiment demonstrates **static work distribution**, where the programmer explicitly decides which portion of the data belongs to each thread.

<img width="742" height="145" alt="03_pthread_work_distribution_sum" src="https://github.com/user-attachments/assets/ef5ec344-3a3d-4b35-958f-38b8cf950f9b" />

---

### Step 4: Concurrency Hazards & Race Condition (`race.c`)

* **Concept:** Four threads attempt to increment a common global variable `counter` 100,000 times each without using any synchronization mechanism.
* **Source File:** [`race.c`](./race.c)
* **Compile & Run:**

  ```bash
  gcc race.c -o race -pthread
  ./race
  ```

The theoretically expected result is:

$$
4 \times 100000 = 400000
$$

However, the actual output can be smaller because the threads are accessing and modifying the same variable concurrently.

* **Root Cause Analysis:**

The operation:

```c
counter++;
```

should not be considered an indivisible operation in a multithreaded program. Conceptually, it involves:

1. Reading the current value of `counter`.
2. Adding one to the value.
3. Writing the new value back.

If two threads read the same old value before either thread writes its result, one increment can overwrite the other increment. This causes **lost updates**.

In the recorded execution, **262,410 updates were lost**, showing how severe the effect of a race condition can become when many concurrent updates occur.

<img width="694" height="90" alt="04_pthread_race_condition" src="https://github.com/user-attachments/assets/2770b0be-1d92-4757-85b1-c830feab336a" />

---

### Step 5: Mutual Exclusion Using Mutex (`mutex.c`)

* **Concept:** This experiment modifies the race-condition program by introducing a POSIX mutex to protect the shared counter.

* **Source File:** [`mutex.c`](./mutex.c)

* **Compile & Run:**

  ```bash
  gcc mutex.c -o mutex -pthread
  ./mutex
  ```

* **Protection Mechanism:**

```c
pthread_mutex_lock(&lock);
counter++;
pthread_mutex_unlock(&lock);
```

The mutex provides **mutual exclusion**, meaning only one thread can execute the protected critical section at a particular time.

A thread first acquires the lock before modifying `counter`. Other threads attempting to enter the same section must wait until the lock is released.

As a result, the expected value of **400,000** is obtained and the lost-update problem is eliminated.

<img width="647" height="93" alt="05_pthread_mutex_fixed" src="https://github.com/user-attachments/assets/6e1bf8cd-6113-43ac-882d-736c31c694e0" />

---

## 5. Part B — OpenMP Compiler Directives

**OpenMP** provides a higher-level approach to shared-memory parallel programming. Instead of explicitly creating every thread, the programmer can use compiler directives such as `#pragma omp parallel` and `#pragma omp parallel for`.

The OpenMP runtime system handles much of the thread creation, scheduling, and coordination, making it especially convenient for numerical and loop-based applications.

---

### Step 6: OpenMP Parallel Region & Thread Identification (`omp1.c`)

* **Concept:** A parallel region is created using `#pragma omp parallel`. Each thread identifies itself using `omp_get_thread_num()`, while `omp_get_num_threads()` returns the total number of threads in the team.
* **Source File:** [`omp1.c`](./omp1.c)
* **Compile & Run:**

  ```bash
  gcc omp1.c -o omp1 -fopenmp
  ./omp1
  ```

The experiment demonstrates the basic OpenMP execution model. When the program enters the parallel region, multiple threads participate in executing the enclosed statements.

<img width="723" height="599" alt="06_omp_parallel_hello_32threads" src="https://github.com/user-attachments/assets/5bb1220b-0d33-4cf0-b83b-d6e14574c9ea" />

---

### Step 7: Work-Sharing Loops & Reduction (`omp_sum.c`)

* **Concept:** The array-processing loop is distributed among multiple threads using `#pragma omp parallel for`. The `reduction(+:total_sum)` clause is used to safely combine the partial sums.

* **Source File:** [`omp_sum.c`](./omp_sum.c)

* **Compile & Run:**

  ```bash
  gcc omp_sum.c -o omp_sum -fopenmp
  ./omp_sum
  ```

* **Mechanism:** Instead of allowing every thread to directly update the same variable, OpenMP provides each participating thread with a private partial value. At the end of the loop, these partial results are combined automatically.

This approach avoids the race condition associated with multiple threads modifying the same accumulator simultaneously.

<img width="661" height="206" alt="07_omp_sum_reduction" src="https://github.com/user-attachments/assets/a9398f3d-d83c-407e-be41-b5b0414956f8" />

---

### Step 8: OpenMP Data Hazards & Race Condition (`omp_race.c`)

* **Concept:** This program demonstrates that simply adding OpenMP parallelism does not automatically make shared data operations safe.
* **Source File:** [`omp_race.c`](./omp_race.c)
* **Compile & Run:**

  ```bash
  gcc omp_race.c -o omp_race -fopenmp
  ./omp_race
  ```

The expected result for four threads performing 100,000 increments each is:

$$
400000
$$

However, because the shared accumulator is modified without synchronization, multiple threads can interfere with each other.

In the recorded execution, **299,825 updates were dropped** out of 400,000.

This experiment highlights the importance of identifying shared variables and protecting them when necessary.

<img width="689" height="93" alt="08_omp_race_condition" src="https://github.com/user-attachments/assets/7a1e676e-c173-4242-b645-c114cc9f2edc" />

---

### Step 9: OpenMP Critical Section Synchronization (`omp_critical.c`)

* **Concept:** The race condition is corrected using `#pragma omp critical`, which allows only one thread at a time to execute the enclosed code.

* **Source File:** [`omp_critical.c`](./omp_critical.c)

* **Compile & Run:**

  ```bash
  gcc omp_critical.c -o omp_critical -fopenmp
  ./omp_critical
  ```

* **Outcome:** The program produces the correct result of **400,000**.

The critical directive provides a simple way of protecting a shared operation. However, if a large portion of a program is placed inside a critical section, threads may spend more time waiting, reducing the advantage of parallel execution.

<img width="662" height="93" alt="09_omp_critical_section" src="https://github.com/user-attachments/assets/9970aee7-eb47-4a86-a40d-c4ae5f1ae7b3" />

---

### Step 10: Phased Barrier Coordination (`omp_barrier.c`)

* **Concept:** A barrier is used to synchronize multiple threads between different phases of execution.
* **Source File:** [`omp_barrier.c`](./omp_barrier.c)
* **Compile & Run:**

  ```bash
  gcc omp_barrier.c -o omp_barrier -fopenmp
  ./omp_barrier
  ```

The statement:

```c
#pragma omp barrier
```

forces each participating thread to wait until all other threads have reached the same synchronization point.

* **Observation:** All threads complete Stage 1 before any thread proceeds to Stage 2.

This is useful in algorithms where the output of one phase must be available before the next phase can safely begin.

<img width="642" height="187" alt="10_omp_barrier_synchronization" src="https://github.com/user-attachments/assets/e723d368-7399-44c3-9ba3-ab59e00e7ea0" />

---

## 6. Part C — Performance Analysis & Empirical Scalability

The final part of the experiment evaluates how execution time changes when the number of threads is increased.

A compute-intensive numerical workload containing **$N = 1,000,000,000$ iterations ($10^9$)** was executed using three approaches:

* Sequential execution
* Pthreads
* OpenMP

The same mathematical computation is used for all implementations so that their performance can be compared using a common workload.

The mathematical invariant is:

$$
\text{Mathematical Invariant: }
\sum_{i=0}^{N-1} (i \times 10^{-6})
=
\mathbf{499999999500.00}
$$

The final result is used to verify that parallel execution produces the same mathematical result as sequential execution.

---

### Step 11: Single-Threaded Sequential Baseline (`sequential.c`)

* **Source File:** [`sequential.c`](./sequential.c)

* **Compile & Run:**

  ```bash
  gcc sequential.c -o sequential_program
  ./sequential_program
  ```

* **Baseline Reference Runtime ($T_{\text{seq}}$):** **`1.418018 seconds`**

The sequential execution time acts as the reference point for evaluating the benefits of parallel execution.

A parallel implementation is considered faster when its execution time is lower than this baseline.

<img width="680" height="88" alt="11_sequential_baseline" src="https://github.com/user-attachments/assets/380d2ca1-1717-4162-b4b4-f751ee722e92" />

---

### Step 12 & 13: Pthreads Scalability Benchmarks (`pthread_perf.c`)

* **Source File:** [`pthread_perf.c`](./pthread_perf.c)
* **Compile & Run:**

  ```bash
  gcc pthread_perf.c -o pthread_perf -pthread
  ./pthread_perf
  ```

The program measures the execution time of the same workload using different numbers of Pthreads.

The tested configurations are **1, 2, 4, 6, and 16 threads**. These measurements are then used to determine how well the workload scales as more threads are introduced.

<img width="694" height="357" alt="12_pthread_perf_all_threads" src="https://github.com/user-attachments/assets/e05b51fd-d52d-4213-8a9b-74fa16c7c025" />

---

### Step 14: OpenMP Scalability Benchmarks (`omp_perf.c`)

* **Source File:** [`omp_perf.c`](./omp_perf.c)
* **Compile & Run:**

  ```bash
  gcc omp_perf.c -o omp_perf -fopenmp
  ./omp_perf
  ```

The OpenMP version performs the same computation and is tested using the same thread counts.

Using identical thread counts and workload sizes makes it possible to compare Pthreads and OpenMP more fairly.

<img width="664" height="355" alt="13_omp_perf_all_threads" src="https://github.com/user-attachments/assets/c2e37b2a-9f2d-401a-af3f-ba225cf65747" />

---

### Step 15: Measured Execution Time Comparison Table

| Thread Count ($P$) | Sequential Baseline | Pthreads Execution Time | OpenMP Execution Time | Time Reduction vs Sequential |
| :----------------: | :-----------------: | :---------------------: | :-------------------: | :--------------------------: |
|    **1 Thread**    |      1.418018 s     |      **1.405171 s**     |     **1.393317 s**    |             ~1.7%            |
|    **2 Threads**   |          —          |      **0.718377 s**     |     **0.717785 s**    |            ~49.4%            |
|    **4 Threads**   |          —          |      **0.358913 s**     |     **0.359875 s**    |            ~74.7%            |
|    **6 Threads**   |          —          |      **0.240157 s**     |     **0.240754 s**    |          **~83.1%**          |
|   **16 Threads**   |          —          |      **0.136414 s**     |     **0.136195 s**    |          **~90.4%**          |

The table shows a clear reduction in execution time as the number of threads increases.

For lower thread counts, the improvement is close to ideal because the workload can be divided efficiently among the available processing resources. At 16 threads, the execution time is still significantly lower, although the improvement is no longer perfectly proportional to the number of threads.

Pthreads and OpenMP produce very similar execution times for this workload, indicating that both approaches can effectively utilize the available shared-memory CPU resources.

<img width="1231" height="703" alt="execution_time_vs_threads" src="https://github.com/user-attachments/assets/9f7526af-9c5c-45ac-a0fe-7cbd159529cb" />

---

### Step 16: Speedup Calculation & Scaling Analysis

Speedup represents the performance improvement obtained from parallel execution compared with the sequential implementation.

$$
\text{Speedup } (S) =
\frac{T_{\text{sequential}}}{T_{\text{parallel}}}
$$

For example, if a parallel program takes half the time of the sequential program, its speedup is approximately **2×**.

|  Threads ($P$) | Pthreads Speedup | OpenMP Speedup | Scaling Assessment                                   |
| :------------: | :--------------: | :------------: | :--------------------------------------------------- |
|  **1 Thread**  |    **1.009×**    |   **1.018×**   | Single-core baseline                                 |
|  **2 Threads** |    **1.974×**    |   **1.976×**   | Near-linear dual-core speedup (~2×)                  |
|  **4 Threads** |    **3.951×**    |   **3.940×**   | Near-linear quad-core speedup (~4×)                  |
|  **6 Threads** |    **5.905×**    |   **5.890×**   | Near-linear six-thread scaling                       |
| **16 Threads** |    **10.395×**   |   **10.412×**  | High parallel throughput with reduced linear scaling |

The results show strong scaling from 1 to 6 threads. At 16 threads, the speedup continues to increase, but it does not reach the ideal value of 16×.

This difference between actual and ideal speedup is expected in practical parallel systems.

<img width="1239" height="709" alt="speedup_vs_threads" src="https://github.com/user-attachments/assets/d47a0267-ed4b-4426-b422-1ad840cc839d" />

---

### Step 17: Parallel Efficiency Analysis

Parallel efficiency indicates how close the measured speedup is to the theoretical ideal speedup for a given number of threads.

$$
\text{Efficiency } (E) =
\frac{\text{Speedup}}{P} \times 100\%
=
\frac{T_{\text{sequential}}}{P \times T_{\text{parallel}}}
\times 100\%
$$

|  Threads ($P$) | Pthreads Efficiency | OpenMP Efficiency | Operating State                                          |
| :------------: | :-----------------: | :---------------: | :------------------------------------------------------- |
|  **1 Thread**  |     **100.91%**     |    **101.77%**    | Single thread baseline                                   |
|  **2 Threads** |      **98.70%**     |     **98.78%**    | Very high efficiency (~99%)                              |
|  **4 Threads** |      **98.77%**     |     **98.51%**    | Near-optimal multicore scaling                           |
|  **6 Threads** |      **98.41%**     |     **98.16%**    | High sustained utilization                               |
| **16 Threads** |      **64.97%**     |     **65.07%**    | Reduced efficiency due to hardware and parallel overhead |

The results indicate excellent efficiency up to 6 threads. At 16 threads, efficiency decreases to approximately 65%.

This does not mean that the parallel program becomes ineffective. Instead, it means that each additional thread contributes less performance than it would under ideal linear scaling.

The slightly greater-than-100% values for the 1-thread measurements can occur because the sequential and parallel programs are separate executions and are affected by normal system and runtime variations.

<img width="1241" height="706" alt="efficiency_vs_threads" src="https://github.com/user-attachments/assets/dc7ea1f1-289a-4515-9129-09c5cf0439cb" />

---

### Step 18: Architectural Bottlenecks & Hardware Interpretation

#### Why does 16 threads achieve ~10.4× speedup instead of 16×?

If perfect linear scaling were possible, 16 threads would theoretically produce:

$$
\frac{1.418}{16}
\approx
0.0886\text{ seconds}
$$

However, the measured execution time is approximately **0.136 seconds**.

This gives a speedup of approximately **10.4×** rather than the ideal 16×. The difference can be explained by several hardware and software factors.

1. **Memory Bandwidth & Cache Contention:**

   Multiple threads access data at the same time. As more threads become active, they compete for shared cache and memory bandwidth. Once the memory subsystem becomes a limiting factor, adding more threads produces smaller performance gains.

2. **Simultaneous Multithreading (SMT / Hyper-Threading):**

   The operating system may expose more logical threads than physical CPU cores. Two logical threads running on the same physical core share several hardware resources.

   Therefore, 16 logical threads do not necessarily provide the performance of 16 independent physical cores.

3. **Thread Creation and Management Overhead:**

   Parallel programs require additional work to create, schedule, synchronize, and join threads. Although this overhead is small compared with a large workload, it becomes relevant when evaluating scalability.

4. **Operating System Scheduling:**

   The operating system decides when each runnable thread receives CPU time. Background processes and scheduling decisions can introduce small variations in measured execution time.

5. **Synchronization and Final Combination:**

   Parallel programs may need synchronization or final aggregation of partial results. These operations cannot always be performed completely in parallel.

6. **Amdahl's Law:**

   Amdahl's Law states that the non-parallel portion of a program places an upper limit on the speedup that can be achieved.

Overall, the experiment demonstrates an important principle of parallel computing:

> **Increasing the number of threads generally improves performance, but the improvement eventually becomes limited by hardware resources, synchronization, memory access, and sequential portions of the program.**

---

## 7. Comprehensive Comparison: Pthreads vs. OpenMP

| Feature                  | POSIX Threads (Pthreads)                                       | OpenMP                                                          |
| :----------------------- | :------------------------------------------------------------- | :-------------------------------------------------------------- |
| **Programming Paradigm** | Explicit, library-based API                                    | High-level, compiler directive-based (`#pragma`)                |
| **Thread Creation**      | Programmer explicitly creates threads using `pthread_create()` | Threads are managed by the OpenMP runtime                       |
| **Thread Lifecycle**     | Programmer must explicitly manage creation and joining         | Runtime manages the thread team and parallel regions            |
| **Work Partitioning**    | Programmer calculates indexes and chunk boundaries             | Loop iterations can be automatically distributed                |
| **Synchronization**      | Mutex locks and other Pthread synchronization functions        | `critical`, `barrier`, `reduction`, and other OpenMP constructs |
| **Barrier Coordination** | Requires explicit Pthread barrier mechanisms                   | Available using `#pragma omp barrier`                           |
| **Reduction Support**    | Usually implemented manually                                   | Directly supported using the `reduction` clause                 |
| **Code Footprint**       | Usually more verbose                                           | Usually shorter and easier to read                              |
| **Control Granularity**  | Provides detailed control over individual threads              | Provides higher-level parallelism                               |
| **Ease of Use**          | Requires more manual programming                               | Easier for loop-based parallel workloads                        |
| **Best Suited For**      | Applications requiring detailed thread-level control           | Numerical, scientific, and loop-oriented shared-memory programs |

### Overall Observation

Pthreads and OpenMP both support shared-memory parallelism, but they follow different programming approaches.

**Pthreads** provides more direct control because the programmer explicitly manages threads and synchronization mechanisms. This makes it suitable when detailed control over thread behavior is required.

**OpenMP** simplifies parallel programming by allowing the programmer to add directives around existing sequential code. It is particularly convenient for loops and numerical computations.

For the performance workload used in this experiment, both methods achieved very similar execution times, demonstrating that the choice between them often depends more on programming complexity and required control than on raw performance alone.

---

## 8. Terminology Glossary

* **Thread:** An independent execution flow that can be scheduled by the operating system.
* **Main Thread:** The initial thread that starts execution of the `main()` function.
* **Worker Thread:** A secondary thread created to perform a particular part of the overall computation.
* **Work Sharing:** Dividing a large task into smaller portions and assigning those portions to multiple threads.
* **Race Condition:** A condition where the final result depends on the timing or ordering of concurrent thread operations.
* **Mutex (Mutual Exclusion):** A synchronization mechanism that allows only one thread at a time to access a protected critical section.
* **Critical Section:** A section of code that accesses shared data and must be executed in a controlled manner.
* **Barrier:** A synchronization point at which threads wait until all participating threads have reached the same point.
* **Reduction:** The process of combining multiple partial results into one final result.
* **Speedup ($S$):** The ratio between sequential execution time and parallel execution time.
* **Parallel Efficiency ($E$):** A measure of how effectively multiple threads are being utilized compared with ideal linear scaling.
* **Amdahl's Law:** A principle that describes the theoretical limit on speedup based on the sequential portion of a program.
* **Shared Memory:** A memory model in which multiple threads belonging to the same process can access common data.
* **Logical Thread:** A hardware-supported execution unit visible to the operating system. Multiple logical threads may share resources within one physical CPU core.
* **Scalability:** The ability of a program to obtain performance improvements as additional processing resources are added.
* **Synchronization:** The coordination of multiple threads to ensure correct ordering and safe access to shared resources.
* **Thread Pool:** A group of reusable threads maintained by a runtime system for executing parallel tasks.

---

## 9. Repository File Structure

```text
PGC-02-Multithread-Pthreads-OpenMP/
│
├── README.md
├── .gitignore
│
├── pthreads/
│   ├── thread1.c
│   ├── thread2.c
│   ├── thread_sum.c
│   ├── race.c
│   └── mutex.c
│
├── openmp/
│   ├── omp1.c
│   ├── omp_sum.c
│   ├── omp_race.c
│   ├── omp_critical.c
│   └── omp_barrier.c
│
├── performance/
│   ├── sequential.c
│   ├── pthread_perf.c
│   └── omp_perf.c
│
└── images/
    ├── 00_wsl_ubuntu_gcc_environment.png
    ├── 01_pthread_single_thread.png                      
    ├── 02_pthread_multiple_threads.png                   
    ├── 03_pthread_work_distribution_sum.png              
    ├── 04_pthread_race_condition.png                   
    ├── 05_pthread_mutex_fixed.png                    
    ├── 06_omp_parallel_hello_32threads.png           
    ├── 07_omp_sum_reduction.png                    
    ├── 08_omp_race_condition.png                       
    ├── 09_omp_critical_section.png                     
    ├── 10_omp_barrier_synchronization.png            
    ├── 11_sequential_baseline.png                          
    ├── 12_pthread_perf_all_threads.png                
    ├── 13_omp_perf_all_threads.png                      
    ├── execution_time_vs_threads.png                     
    ├── speedup_vs_threads.png                               
    └── efficiency_vs_threads.png                           
```
