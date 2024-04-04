---
layout: post
title:  "Introduction to GPU Programming with D3D12: Intro"
date:   2024-03-30 14:40:00 +0200
categories: Introduction_to_GPU_Programming_with_D3D12
lang: en
---

This is the first part of a series about GPU programming with D3D12. This series will cover how and why you would run code on a GPU, as well as some adjacent topics.

## Intended audience

This series is intended for people who would like an introduction to GPU programming, or who are already familiar with high level graphics API, but want an introduction for a low level one.

## Prerequisites

C++ – most Graphics APIs only have native bindings for C and C++. This series focuses on C++ as it makes certain things simpler. This means that you’ll need to have a certain level of proficiency with C++ before you can efficiently use said APIs.

Math – this series will touch on both compute and graphics capabilities of modern GPUs. To program graphics you’ll need a solid understanding of linear algebra.

Modern GPU - we'll use certain features that are only available on fairly modern GPUs. Minimum Required GPUs: NVIDIA GeForce RTX 20 series, AMD Radeon RX 6000 series, Intel ARC.

With that out of the way we can finally get to the introduction.

## Why GPU Programming?

GPU is short for Graphics Processing Unit. They are a very common piece of hardware that is present in virtually any modern consumer computer/smartphone/game console/etc. In fact, you are most likely relying on one while reading this text right now.

Originally GPUs were designed to solve a specific problem: render 3D graphics. But over the years, as realism of 3D graphics advanced, GPUs became more and more generalized, with modern GPUs often used for computations that are no longer related to graphics: Scientific Simulations, Machine Learning, Cryptocurrency Mining, etc.

So, it is worth exploring how exactly are GPUs different from CPUs, why only certain use cases use them, and how to write and run your code on one.

## Why D3D12?

To interface with the GPU, you need to use what’s called a “Graphics API”. Despite the name they allow you to run any kind of workload on the GPU, that GPU is capable of running, not just “Graphics”.

At the time of writing, there are many relevant Graphics APIs:

- D3D11 – high level, Windows specific API
- Vulkan – low level, cross platform API
- D3D12 – low level, Windows specific API
- Metal – can be both high level and low level API, Apple specific API
- WebGL – high level, web-based API
- WebGPU – low level, web-based API

High level APIs were not selected, as many of modern GPU features, such as Hardware Accelerated Raytracing or Mesh Shaders, require low level APIs specifically.

Web-based APIs were not selected, as they are simply translated to other APIs, and so are more limited than the underlying API.

That leaves Vulkan, D3D12 and Metal, all of which are a perfectly fine choice. D3D12 was picked as it is slightly simpler compared to Vulkan, and more accessible compared to Metal. But many concepts and ideas are the same between those APIs.

## How exactly are GPUs different from CPUs?

GPU’s original purpose is to render 3D graphics. If we oversimplify that process, you need to calculate what color each pixel on the screen gets. This calculation has 2 important properties:

1. Each pixel gets calculated independently, there are no data dependencies between any 2 pixels.
2. Calculations performed for each pixel are the same, there is no excessive divergence between them.

And that’s where GPU power comes from. While CPUs are largely designed to efficiently execute any sort of calculations with limited parallelism, GPUs are purposely made to execute highly parallel programs.

At the time of writing, according to [Steam Hardware Survey](https://store.steampowered.com/hwsurvey/Steam-Hardware-Software-Survey-Welcome-to-Steam?platform=pc), 98% of users has CPUs with 16 or less cores. Even if we assume that each CPU has [Simultaneous Multithreading](https://en.wikipedia.org/wiki/Simultaneous_multithreading), it means that practically any modern CPU can’t process more than 32 tasks in parallel.

On the other hand, the most common GPU at the time of writing is RTX 3060. That GPU has 3584 shading units, meaning that it can process 3584 tasks in parallel.

We can see that an average GPU can process over 100 times more tasks in parallel than high end CPUs. This is exactly how GPUs get dramatically higher performance for highly parallel computations, such as rendering 3D graphics.

However, there are computations that are impractical on a GPU. It can’t freely allocate its own memory, it’s the CPU that is responsible for memory management. You don’t have access to disk, network or input. GPUs can't efficiently run algorithms that are completely sequential and can't be parallelized. Due to those, and other similar limitations, only certain types of workloads can efficiently run on the GPU.
