# Parallel Distance Joins using LSH on Apache Spark

This repository contains a scalable, distributed implementation of approximate distance joins for large-scale high-dimensional vectors. The project leverages **Locality-Sensitive Hashing (LSH)** with p-stable distributions and **Apache Spark's Map-Side Broadcast Join** to overcome the "Curse of Dimensionality" and network bottlenecks in Big Data environments.

## Overview
Finding similar vectors across two massive datasets is a computationally expensive Cartesian product problem `O(|R| x |S|)`. Traditional tree-based indices fail in high-dimensional spaces. 

This project implements an approximate distance join solution that evaluates only highly probable candidate pairs, completely bypassing full exhaustive comparisons. It was tested on the **SIFT1M dataset** (1 million base vectors, 10,000 query vectors, 128 dimensions), evaluating 10 billion potential pairs.

### Key Highlights
* **High Recall**: Consistently achieves > 98.5% recall compared to the exact brute-force baseline.
* **Significant Speedup**: Demonstrated a 6.1x performance acceleration over the exhaustive comparison baseline on a constrained cluster.
* **Network Optimization**: Replaced expensive Spark Shuffle operations with a **Map-Side Broadcast Join** architecture, broadcasting the smaller LSH index directly to worker memory.
* **Vectorized Processing**: Utilizes `NumPy` matrix multiplications over partitioned data chunks to avoid Out-Of-Memory (OOM) errors and exploit hardware vectorization.

## Architecture & Tuning
The implementation relies on three core parameters that were rigorously tuned via grid search:
1. **k (Hash functions per table)**: Set to `k=4` to balance bucket selectivity and collision probability.
2. **L (Number of Hash Tables)**: Tuned to `L=50` as the optimal trade-off between recall convergence (>95%) and computational overhead.
3. **w (Bucket Width)**: Dynamically calculated as `1.5 * θ` (where `θ` is the distance threshold) to regulate filtering strictness.

## Repository Structure
* `/notebooks`: Contains the PySpark implementations for tuning parameters (`L`, `w`, `ε`), evaluating scalability, and the `Grand Finale` full-scale execution.
* `/results`: Experimental output logs and execution times for various dataset scales and threshold tuning.
* `/docs`: The detailed academic report covering the mathematical background, Spark architecture, and performance analysis (in Greek).

## Getting Started

### Prerequisites
* Apache Spark (v4.0+)
* Python 3.x
* `pyspark`, `numpy`

### Dataset
The project uses the [SIFT1M dataset](ftp://ftp.irisa.fr/local/texmex/corpus/sift.tar.gz) from INRIA. You can download and extract it directly inside the project directory:
```bash
wget ftp://ftp.irisa.fr/local/texmex/corpus/sift.tar.gz
tar -xzf sift.tar.gz
