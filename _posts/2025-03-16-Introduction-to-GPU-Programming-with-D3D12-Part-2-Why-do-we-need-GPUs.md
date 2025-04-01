---
layout: post
title:  "Introduction to GPU Programming with D3D12: Part 2 - Why do we need GPUs?"
date:   2025-03-16 4:00:00 +0900
categories: Introduction_to_GPU_Programming_with_D3D12
lang: en
---

[Previous Post](Introduction-to-GPU-Programming-with-D3D12-Part-1-Intro.html)

Before diving into GPU Programming, you should first understand the need for it. Why do we need another kind of processor if we can already perform any kind of calculation with CPU? Let's look at a few sample problems.

## Problem 1

You have an array of 3D points and a transformation matrix. You need to apply transformation to each of those points and overwrite original array with the result.

### CPU Solution

What we can do to solve this problem with a CPU algorithm is to run a loop over all points and apply transformation to each one. This gets problem solved, but it will take a while to iterate over a large array.

Let's try to speed it up. Average CPU has about 6 cores/12 threads. That means you can run 12 threads in parallel, each getting equal part of an array to process. That is up to 12x speedup, which is better, but nowhere close to what we need.

### GPU Solution

Unlike CPU, we can't just directly run a code on a GPU. What we'll do instead is:

- Expose copy of that array to the GPU
- Write a program that takes transform and index of a point, applies transform to specified point and then writes result back to that array
- Launch however many instances of that program we need to on GPU
- When GPU is done, copy that array back

And here's the big difference: average GPU (Nvidia RTX 3060) has about 3500 threads. GPU will automatically spread-out requested workload among those threads, so it will be way faster compared to CPU approach.

## Problem 2

You have a memory block. You need to calculate its hash. Hash is defined as a function that takes the last byte of a memory block and hash of all bytes before that. If the memory block is empty, hash returns 0.

### CPU Solution

Very straightforward, we start with 0 as previous hash and calculate the hash for each subsequent byte. In the end we have hash for the whole block of memory. We can't use multithreading in this example, as each subsequent computation first needs result of a previous one.

### GPU Solution

Unfortunately, we can't do much better than a CPU algorithm. Since we are now required to perform all calculations in a series, the best we can do is:

- Expose copy of memory block to the GPU
- Write a program to calculate hash of whole memory block
- Launch 1 instance of it on the GPU
- When GPU is done, copy back the hash

It has multiple drawbacks:

- We waste memory, as we now need a copy for our data
- GPU threads are slower compared to CPU threads, so algorithm by itself is going to take longer to execute on GPU compared to CPU
- We need CPU to wait until the work is done on the GPU, introducing complexity and overhead of CPU/GPU synchronizations
- We still need to perform memory copy on the CPU, which introduce additional overhead

So, it's clearly better to this problem solve with CPU approach.

## Problem 3

You need to read 2 numbers from console and output their sum.

### CPU Solution

```  
#include <iostream>
int main() {  
int a;  
int b;  
std::cin >> a >> b;  
std::cout << (a + b) << std::endl;  
}  
```

That's it

### GPU Solution

You literally can't do that. GPUs are not designed for single threaded problems. That's why you just don't get access to many things: console, keyboard/mouse/controller input, file system, network, etc. The logic is that those things are best left to the CPU.

## CPU and GPU comparison

Several conclusions from those examples:

- Some problems can be solved by both the CPU and the GPU, but the GPU does so much faster
  - If you need to have high performance, and have such problem, you have a use case for a GPU
- Some problems can be solved by both the CPU and the GPU, but the GPU can't do so as efficiently as the CPU
  - Just because you can, doesn't mean you should
- Some problems cannot be solved by GPU
  - You will always need to have part of your program to be executed by the CPU

[Next Post](Introduction-to-GPU-Programming-with-D3D12-Part-3-History.html)
