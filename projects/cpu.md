---
layout: page
title: Superscalar In-order CPU
description: >
  How you install Hydejack depends on whether you start a new site,
  or change the theme of an existing site.
hide_description: true
sitemap: false
permalink: /cpu
---
This project was for my CSE 148 class at UCSD. As a team of two, we were given a basic five stage pipeline cpu design and were tasked with adding at least four optimization to the design. The time frame was 10 weeks including time to analyze the given base design thus this cpu design is not fully optimized. We considered adding out-of-order execution to the base design, but with only 10 weeks we believed it was just too tedious to fully implement, thus opted for other different solutions.

Project Presentation *[here](https://docs.google.com/presentation/d/1Gu_lHgepVhmzd-aFqks3AUkmhkOy-SfV8bvoyNtcZ04/edit?usp=sharing)*.

Project Paper *[here](/projects/assets/files/cpu_paper.pdf)*.

### Equipment Used
* Intel Quartus Prime
* Modelsim

### Overview

The baseline soft-core given at the beginning of the project had:
* 5-stage H&P pipeline
  * Forwarding
  * Branches resolved in the execute stage
    * NOTE: Although we have a branch delay slot, we still take a one cycle hazard for mispredictions due to not resolving in the decode stage. Branch delay slots were hardcoded into the MIPS ISA (stupid idea)
* No branch prediction
* Very simple caches

This soft-core implemented a subset of the MIPS I ISA with:
* No floating-point instructions
* No multiply / divide instructions

![Baseline CPU Diagram](/projects/assets/img/baseline-cpu-diagram.jpeg)

Although this cpu was not implemented on real hardware, the target device is the Cyclone V SoC FPGA. 

The SDRAM controller, the PLL to generate the clock, and arbiters / buffers between mem_read/write and the SDRAM controller were generated using the Platform Designer.

### Superscalar (2-Way) Optimization

![2-Way superscalar CPU diagram](/projects/assets/img/superscalar-cpu-diagram.png)
NOTE: The diagram is a little large so open the image in a new tab for a better view

* In/out of each module in cpu, pipeline registers, ALU doubled
* Hazard controller expanded
* Forward unit expanded to handle lane forwarding
  * Lw and sw hazard
  * Double the lw hazards
  * Stall pipeline on hazards
* Still only one read/write to mem at a time
* Resolve branches in IF stage
  * First instruction might be branch and second might be BDS
  * Need branch prediction in next cycle

### Hardware Prefetching (Stream Buffer) Optimization

![Stream Buffer](/projects/assets/img/stream-buffer.png)
* On a Cache miss, check the stream buffer's first line
* If the cache line is there, move to cache and retrieve next line
* If the cache line is not there, flush stream buffer, fetch cache line, and then fetch extra cache lines to fill space
* Simple FIFO queue 

Another proposed implementation was the Multi-way stream buffer (different strides per stream buffer)

### Branch Prediction (Direct Mapped YAGS) Optimization

#### YAGS Branch Prediction
YAGS works by taking the branch instruction's address and adding it with the global history register (GHR). This value is then routed to a pattern history table which determines whether to jump or not. Then the index value is used to index the secondary table (one each for taken / not taken). If the first table predicted taken, then the secondary table for not taken is accessed for special cases and vice versa. If the tag of that index matches the branch address, the prediction from the secondary table is used. 

The first table is only updated when the branch prediction is correct. The secondary table is updated when the corresponding branch result happens after branch resolve.

The GHR is carried to the execute stage via a separate pipeline that also contains branch mispredict recovery info so that on a mispredict, we can change the GHR info to be the actual branch result.

#### Issues
The branching system in the MIPS I ISA used a branch-delay slot which meant an instruction after the branch was ran regardless of taken or not. This allows for the branch to be ran in the decode stage while the BDS instruction is in the fetch stage. When predicting branch's in a superscalar design, two problematic scenarios arose:
* If the first instruction in the pair was a branch that means the BDS is in the same pair. Once the branch reaches the decode stage, the fetch stage instructions would be ran which is invalid because they are not in the BDS.

This was fixed by decoding instructions in the fetch stage and if it is a branch, a prediction would happen and the PC would change all in one stage.

* When the second instruction in the pair is a branch, that means the BDS instruction is in the next pair of instructions. If we immediately changed the PC, the BDS instruction would not be ran.

This was fixed by stalling the PC change until the decode stage. Also, after predicting to jump, the second instruction of the BDS pair is discarded because it should not be ran. 

All this information must be kept until the execute stage because on a mispredict it needs to know which instructions to wipe. Thus, a separate pipeline containing this info moves from the fetch to execute stages always in the same stage and pipeline as the branch instruction.

### Value Prediction (Load Value) Optimization
Our motivation for doing load value prediction was that with a superscalar design, there were alot more load word hazards (when a decode instruction needs a value from a register currently used in the execute stage). 

Due to decoding instructions in the fetch stage, we did our load value prediction in the fetch stage similar to our branch prediction. The predicted value was always the last value that the same instruction loaded. This was because many instructions tend to load in the same values every time (constant values). 

The prediction was resolved in the memory stage and if mispredicted, we would flush all instructions prior and restart.