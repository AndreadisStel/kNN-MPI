# Distributed k-NN Search using MPI

## Overview
This project implements a distributed, brute-force all-k-Nearest Neighbors (k-NN) search algorithm using the Message Passing Interface (MPI). The algorithm is designed to find the $k$ nearest neighbors for every point $x$ within a set $X$.

By leveraging distributed computing, this implementation addresses the limited memory problem associated with large datasets, where storing a full $X \times X$ distance matrix in a single system's RAM is impossible.

## Features
* **Sequential Baseline:** A standard brute-force implementation used for performance comparison.
* **Distributed Ring Architecture:** Data is distributed across $p$ MPI processes in disjoint blocks. The implementation uses a ring-based communication pattern to pass data blocks between processes, updating the nearest neighbors at each step.
* **Memory Efficiency:** Instead of storing the global distance matrix, each process only allocates memory for its local query set and the current corpus set being evaluated.
* **Asynchronous Parallelism:** The parallel version utilizes asynchronous communication to find all $k$ nearest neighbors across distributed nodes.

## Implementation Details

### Data Distribution
* The dataset $X$ is partitioned into local matrices $x_i$ of size $(n/p \times d)$, where $p$ is the number of processes.
* Every process $P_i$ calculates the distances between its own points and all other points in the dataset by rotating "corpus" sets $y_i$ around the ring.

### Communication & Synchronization
* **Ring Transfer:** Data is moved along the ring (receiving a new $y_i$ from the previous process and sending the current one to the next) until every process has seen the entire dataset.
* **Index Management:** The algorithm handles the mapping between local and global indices to ensure that the final neighbor IDs correctly reference the original dataset.
* **Result Aggregation:** Once the ring transfer is complete, results are collected by the root process (Process 0) for final output.

## Performance Analysis

The project was evaluated using 2D and 3D regular grids.
While the sequential version remains highly efficient for smaller datasets due to lower communication overhead, the distributed MPI approach is designed to scale for datasets that exceed the memory limits of a single machine.

### Experimental Results (Execution Time)
| Dataset | Sequential | MPI (n=2) | MPI (n=3) | MPI (n=4) |
| :--- | :--- | :--- | :--- | :--- |
| 3D 1k Elements | 0.05s	| 0.44s |	0.45s	| 0.43s |
| 2D 10k Elements | 10.72s	| 20.98s	| 15.24s|	13.83s |

Performance benchmarks indicate that as the dataset size increases, the overhead of MPI communication is mitigated by the distributed processing power.\
**While communication overhead is more visible in smaller datasets, the performance trajectory indicates that on high-performance computing (HPC) infrastructures with massive datasets, this distributed approach will likely achieve high efficiency and effective load balancing.**

## How to Run

1. Prerequisites
Ensure you have an MPI implementation (like OpenMPI) installed on your system.

2. Compilation
```bash
make clean
make
```
3. Running the Sequential Version
To run the serial version for 2D or 3D grids:
```bash
./bin/knn_serial [2 or 3]
```
4. Running the Parallel Version
To run the MPI version with 4 processes for 2D or 3D grids:
```bash
mpirun -n 4 ./bin/knn_mpi [2 or 3]
```



