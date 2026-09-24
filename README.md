**Description**
GPU architecture profiling tools using NVBit (CS623). Analyzes thread/memory divergence, instruction footprints, and simulates L1 data cache behavior (LRU, 32-way) across varying SM configurations.

---

# IITK CS623: GPU NVBit Profiling

An analytical toolset built on top of the [NVBit](https://www.google.com/search?q=https://github.com/NVlabs/nvbit&utm_source=gemini) (NVidia Binary Instrumentation Tool) framework to extract, trace, and simulate execution metrics of compiled CUDA program binaries (`app1`, `app2`). This repository contains custom instrumentation tools and C++ simulators developed for the CS623 GPU Architecture course[cite: 7].

## 📊 Core Features & Analysis

This project is divided into three primary profiling objectives:

### 1. Thread Hierarchy & Control Divergence (`control_div`)

* Extracts the exact thread hierarchy (blocks per dimension, threads per block) of executing kernels.
* Calculates overall **Thread Divergence** by monitoring active threads per warp instruction (excluding predicates).

### 2. Instruction Footprint Analysis (`pc_trace`)

* Logs the Program Counter (PC) trace of all executed warp instructions.
* Computes the number of **unique instruction cache blocks** (assuming 32-byte cache lines) and unique instructions accessed during execution.

### 3. Memory Divergence & L1 Cache Simulation (`mem_trace` & Parsers)

* **Memory Divergence (Part A):** Analyzes memory address traces to compute the average divergence per memory reference (unique 128-byte data cache blocks accessed divided by active threads in the warp).
* **L1 Cache Modeling (Part B):** A custom C++ LRU cache simulator modeling an array of L1 caches.
* **Specs:** 128 KB capacity, 32-way associativity, 128-byte block size.
* **Scaling:** Simulates cache hit/miss rates across dynamically scaling Streaming Multiprocessor (SM) configurations (1, 2, 4, 8, 16, 32, and 64 SMs) using round-robin CTA scheduling.



## 🛠️ Toolchain Structure

```text
├── tools/
│   ├── control_div/       # NVBit tool for thread hierarchy and divergence
│   ├── pc_trace/          # NVBit tool for instruction PC tracing
│   ├── mem_trace/         # Modified NVBit tool for streamlined memory address tracing
│   ├── p2.cpp             # C++ parser for Instruction Cache analysis
│   ├── p3a.cpp            # C++ parser for Memory Divergence
│   └── p3b.cpp            # C++ L1 Cache Simulator (LRU, 32-way)

```

## 🚀 Building & Running

### Prerequisites

* Linux x86_64 environment
* NVIDIA GPU with appropriate CUDA drivers
* NVBit v1.7.1
* GCC/G++ for parser compilation

### 1. Compile the NVBit Tools

Navigate to any of the tool directories (`control_div`, `pc_trace`, `mem_trace`) and compile the shared object (`.so`) file:

```bash
cd tools/mem_trace
make clean && make

```

### 2. Generate Traces using LD_PRELOAD

Run the target CUDA application while injecting the compiled NVBit instrumentation tool:

```bash
LD_PRELOAD=./tools/mem_trace/mem_trace.so ./app2 file2.1 > trace2.1.txt

```

### 3. Compile and Run the C++ Analyzers

Compile the C++ parsers with high optimization (`-O3`) for handling massive trace files efficiently:

```bash
g++ -O3 p3a.cpp -o p3a
./p3a trace2.1.txt

```
