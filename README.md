# NCDC RISC-V Microarchitecture Module — Cache Simulator

## Course
**NCDC Cohort 02/2025 — Design Verification (DV)**
NUST Chip Design Centre (NCDC), NUST

## Module
**RISC-V Microarchitecture Module** — Cache Architecture Project

---

## Overview

This project implements a **cache simulator** in C that models the behaviour of hardware caches with configurable parameters. The simulator reads real memory-access traces (generated from actual programs) and simulates cache hits, misses, and evictions using the **Least Recently Used (LRU)** replacement policy. It is derived from the CMU 15-213 / CS:APP CacheLab framework and extended with custom analysis.

---

## Background: Cache Memory

A cache is a small, fast memory that sits between the CPU and main memory (DRAM). It exploits **temporal** and **spatial locality** to reduce average memory access time. Key parameters include:

| Parameter | Symbol | Description |
|-----------|--------|-------------|
| Number of sets | S = 2^s | Total sets in the cache |
| Lines per set | E | Associativity (E=1: direct-mapped, E=∞: fully-associative) |
| Block size | B = 2^b | Bytes per cache line |
| Total capacity | S × E × B bytes | |

An address is split into **tag** bits, **set index** bits, and **block offset** bits.

---

## Repository Structure

```
cachelab-handout/
├── csim.c             # Cache simulator source — main implementation
├── cachelab.c         # Helper functions (printSummary, etc.)
├── cachelab.h         # Header — data structures and function declarations
├── csim              # Compiled simulator binary
├── csim-ref           # Reference simulator binary (for comparison)
├── Makefile           # Build system
├── driver.py          # Automated grading/testing script
├── test-csim          # Shell script — runs simulator against reference
├── README             # Original CacheLab README
└── traces/            # Real memory-access trace files
    ├── yi.trace        # Trace from the yi benchmark
    ├── yi2.trace       # Variant trace
    ├── dave.trace      # Trace from the dave benchmark
    ├── trans.trace     # Matrix transpose trace (cache-sensitive)
    ├── long.trace      # Extended trace for stress testing
    └── debug.trace     # Short trace for debugging
NCDC_Lab_Report_Cache_Simulator.pdf   # Full lab report with results and analysis
```

---

## Cache Simulator Features

- **Configurable geometry:** Set via command-line flags (`-s`, `-E`, `-b`).
- **LRU eviction:** When a set is full and a miss occurs, the least recently used line is evicted.
- **Trace-driven simulation:** Accepts Valgrind-format memory trace files (`.trace`).
- **Statistics output:** Reports total **hits**, **misses**, and **evictions** at the end of each run.
- **Reference comparison:** Results can be validated against the provided `csim-ref` binary.

---

## How to Build and Run

```bash
# Build the simulator
make

# Run the simulator
# Usage: ./csim -s <s> -E <E> -b <b> -t <tracefile>
./csim -s 4 -E 1 -b 4 -t traces/yi.trace       # 16 sets, direct-mapped, 16-byte blocks
./csim -s 4 -E 2 -b 4 -t traces/dave.trace     # 2-way set-associative
./csim -s 4 -E 4 -b 4 -t traces/trans.trace    # 4-way set-associative

# Compare with reference
./csim-ref -s 4 -E 1 -b 4 -t traces/yi.trace

# Run automated tests
python3 driver.py
```

### Example Output
```
hits:1891  misses:737  evictions:695
```

---

## Trace File Format

Each line in a trace file describes one memory operation:

```
I 0400d7d4,8        # Instruction load at address 0x0400d7d4, size 8 bytes
 L 10,1             # Data load at address 0x10, size 1 byte
 M 20,1             # Data modify (load then store) at address 0x20
 S 18,1             # Data store at address 0x18
```

Instruction fetches (`I`) are ignored; only data accesses (`L`, `S`, `M`) are simulated.

---

## Concepts Demonstrated
- Cache geometry and address decomposition (tag / index / offset)
- LRU replacement policy using a timestamp or linked-list ordering
- Trace-driven simulation of memory hierarchies
- Hit/miss/eviction counting and performance analysis
- Impact of cache parameters (S, E, B) on hit rate for real workloads
