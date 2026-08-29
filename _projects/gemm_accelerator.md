---
layout: project
title: "8×8 GEMM Accelerator"
tagline: "Hardware-verified INT8 matrix multiplication on a Cyclone V FPGA."
order: 1
status: "active"
role: "RTL design + verification + HPS integration"
organization: "Personal summer project"
date_range: "Summer 2026 - Present"
tech:
  - SystemVerilog
  - C
  - ModelSim
  - Embedded Linux
  - Python
  - Quartus Prime
repo: "https://github.com/Ae0lis/gemm-accelerator"
demo: ""
thumb: "projects/gemm-de1-soc.jpg"
header_image: "projects/gemm-de1-soc.jpg"
placeholder_color: "#34598a"
placeholder_label: "8×8 GEMM"
featured: true
category: "FPGA"
---

## Overview

I built an 8×8 signed-integer matrix-multiplication accelerator in synthesizable SystemVerilog and ran it on the Cyclone V FPGA of a DE1-SoC. The project covers the full path from RTL and verification to synthesis, timing analysis, memory-mapped integration, ARM-side software, and physical-board testing.

The current implementation is a **result-stationary broadcast array** rather than a systolic array. The broadcast array is fully hardware-verified. In the future, I plan to make it into a systolic array to improve performance.

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

## Verification

- **Component-level SystemVerilog tests** for all pieces of the design
- **1,000 randomized register and idle operations** to test the Avalon-MM slave's control interface
- **Seeded 8×8 golden model verification** in SystemVerilog 
- **ARM-side 8×8 golden model verification** using C on embedded Linux, all 64 outputs matched on hardware
- **GitHub repository** with all RTL, testbenches, integration software, and hardware evidence (plus information on how to use it yourself!)

## Context

I wanted to do some kind of FPGA project during summer 2026, so I poked around a bit and ended up interested in GEMM accelerators. My end goal was to make a systolic array that could accelerate signed INT8 × INT8 multiplication with INT32 accumulation. I picked this because I’m very interested in AI-related hardware as a field, and I wanted to get my toes wet.

First off, what is a GEMM accelerator? GEMM stands for GEneral Matrix Multiplication, so a GEMM accelerator is a piece of hardware custom-designed to speed up GEMM operations. The core of a GEMM accelerator is a MAC - Multiply ACcumulate - unit. A MAC will receive two numbers, multiply them together, and add them to a running total. This is the backbone of matrix multiplication. To multiply two matrices together, you take all the numbers in a row of the first and multiply them with the corresponding numbers in a column of the second, then add the results together to get a single number of the result matrix. This is known as a dot product. Suppose we're multiplying matrices A and B together to get matrix C. A Row 0 · B Col 0 gives `C[0][0]`, A Row 0 · B Col 1 gives `C[0][1]`, A Row 1 · B Col 0 gives `C[1][0]`, and so on. If that’s a confusing explanation, you can find a fantastic animation here: http://matrixmultiplication.xyz/. Assuming you’re multiplying 8×8 matrices, you’ll need a minimum of 8 MAC operations per result number (so a total of 512 MAC operations). 

CPUs and GPUs can do this fairly well, but there's some room for improvement. Dedicated hardware can provide both higher throughput and lower power draw, at the cost of reduced flexibility. Because GEMM is so important in the modern age (particularly for AI applications), many companies have designed ASICs to improve performance. These include Google’s TPU and AWS Trainium. Both take advantage of a really cool architecture called a systolic array for their matrix operations. In a systolic array, each MAC unit takes in numbers every cycle, does its calculations, then passes that number along to the next like how numbers flow in that matrix multiplication animation. This reduces fan-out substantially, especially for larger arrays, and supports high throughput through localized data movement.. My goal this summer was to build my own spin on one of these. Right now, I’ve built a MAC array similar to a systolic array, but the data is driven externally by a state machine rather than being pumped across in waves. This makes my design less scalable. Currently, with just 64 MACs it works perfectly fine. Actually, I suspect that with an array this small the two approaches may have comparable latency because broadcasting avoids the fill and drain time of a systolic array (though I haven't tested that yet). Larger iterations, on the other hand, would start to really struggle with pumping a number to every single MAC on every single clock cycle. For an array the size used in the first-generation TPU - a 256×256 array with a whopping 65,536 MAC units - this would be a serious limitation. To get more experience with scalable design, I’ll definitely be transforming my array into a systolic one in the near future.

## Approach and lessons learned

The first major design decision I had to make was how to design the MAC loading algorithm (designing the MAC was pretty easy). There are three main schools of thought. The first, weight-stationary, is used frequently in AI applications. The benefit here is that you can preload an array with weights before pumping in inputs to maximally reuse weights. In this case, you collect the result as it filters out the bottom. The second frequently-used design is result-stationary. This makes the feeding a bit trickier, but allows for very easy measurement at the end. Finally, input stationary allows for reuse of input data across multiple weights. Of the three, result-stationary was the one that was most useful for me. This means that on accumulation cycle `k`, the design broadcasts column `k` of A and row `k` of B, and MAC `(i,j)` accumulates `A[i,k] × B[k,j]` into its local INT32 result. This result can then be read directly, rather than waiting for it to filter out of the array. Especially when testing out calculations with only one MAC, result-stationary allowed for much easier automated testing. 

Speaking of which, time to talk about how I went about building this. I started by making a MAC unit, then testing that with `$readmemh` in SystemVerilog. I compared it against a set of known-good vectors generated by a Python golden model. After that worked out, I designed a basic state machine to pump numbers into the MAC in hardware and then read them out. I was imagining this to be an easier version of the array, but I didn’t end up finding it to be a particularly useful exercise in the long term. Then came the biggest hurdle - connecting the ARM chip on the DE1-SoC to the FPGA I was programming. I lost an ungodly amount of time getting Linux set up and writing C programs to test that the hardware I was programming onto the FPGA was actually functional. If I were to do this again, I would save hardware deployment until after I finished the systolic array in simulation. Debugging hardware was by far my biggest time loss.

Once I had the basics in place, it was time to build the array. I created a MAC array to do the calculations, then made some RAM blocks and designed the hardware to support the two. I ended up using the lightweight bridge for simplicity’s sake, so I had to design an Avalon-MM slave to receive inputs from the ARM core and funnel them into the MAC array. Due to having to send a column of A and a row of B, I found it easier to just store A transposed in the first place so I could use the same hardware for both arrays. A bit of timing wizardry and debugging later, and I had a broadcast array running correctly on the **physical FPGA**! 

Typing it out now, it sounds so easy! In reality, this project took me two months. Granted, I was working 40 hours a week so I could only work on it in the evenings and weekends, but it was seriously tricky at some points! By far the worst hurdle was just figuring out what to even do next. I hadn’t ever built something like this when I started - the closest I had done was making ‘Dancing With Your Fingers’ (writeup in progress, but you can find the GitHub repository [here](https://github.com/Ae0lis/dancing-with-your-thumbs)). Researching this, reading papers about what architecture was the best, and making all the design decisions myself was a really fun experience. I definitely learned a ton, especially about embedded C and software-to-hardware interfaces. Overall, I really enjoyed this project, and I look forward to whatever comes next!
