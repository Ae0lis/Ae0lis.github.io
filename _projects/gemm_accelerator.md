---
layout: project
title: "8x8 GEMM Accelerator"
tagline: "Hardware-verified INT8 matrix multiplication on a Cyclone V FPGA."
order: 1
status: "active"
role: "RTL design + verification + HPS integration"
organization: "Personal summer project"
date_range: "Summer 2026 - Present"
tech:
  - SystemVerilog
  - C
  - Python
  - Quartus Prime
repo: "https://github.com/Ae0lis/gemm-accelerator"
demo: ""
thumb: "projects/gemm-de1-soc.jpg"
header_image: "projects/gemm-de1-soc.jpg"
placeholder_color: "#34598a"
placeholder_label: "8x8 GEMM"
featured: true
category: "FPGA"
---

## Overview

I built an 8×8 signed-integer matrix-multiplication accelerator in synthesizable SystemVerilog and ran it on the Cyclone V FPGA of a DE1-SoC. The project covers the full path from RTL and verification to synthesis, timing analysis, memory-mapped integration, ARM-side software, and physical-board testing.

The current implementation is a **result-stationary broadcast array** rather than a systolic array. On accumulation cycle `k`, the design broadcasts column `k` of A and row `k` of B; MAC `(i,j)` accumulates `A[i,k] * B[k,j]` into its local INT32 result. The broadcast array is fully hardware-verified. In the future, I plan to make it into a systolic array to improve performance.

## Results

- **64 parallel MAC units** operating on signed INT8 inputs with INT32 accumulation
- **9-cycle core latency:** one initialization cycle plus eight accumulation cycles
- **50 MHz fabric clock** with **77.48 MHz post-fit Fmax** reported by TimeQuest
- **64 of 87 DSP blocks** used on the Cyclone V device
- All **64 hardware outputs matched** the expected matrix on the physical DE1-SoC
- About **2 µs observed by the ARM** from issuing start to detecting completion, excluding input loading and output reads

## Architecture

<figure>
  <img src="/images/projects/gemm-architecture.png" alt="Block diagram of the GEMM accelerator, Avalon-MM interface, matrix storage, and 8 by 8 MAC array">
  <figcaption>The accelerator connects an Avalon-MM control interface and matrix storage to a 64-MAC broadcast array.</figcaption>
</figure>

Software stores A transposed so its columns can be read efficiently, while B remains row-major. The ARM writes four INT8 values per word into the input memories, starts the accelerator through its control register, polls for completion, and reads the 64 INT32 results.

## Verification and integration

I wrote focused SystemVerilog testbenches for the MAC, matrix storage, MAC array, and Avalon-MM slave. The integrated interface test performs **1,000 randomized register, memory, and idle operations** before executing a seeded 8×8 GEMM; every result is checked against a golden reference vector.

For board-level verification, a C program on the ARM maps the lightweight HPS-to-FPGA bridge through `/dev/mem`, verifies the accelerator's identity and scratch registers, loads the matrices, starts the computation, and compares all 64 outputs with the expected matrix. The repository includes the RTL, testbenches, integration software, compilation reports, and captured hardware evidence.

## What I learned

This project made the distinction between a working compute core and a usable accelerator concrete. The MAC array itself is compact; the larger engineering task is defining a reliable memory map, proving signed arithmetic and control behavior, closing timing, and making the hardware straightforward for software to drive.
