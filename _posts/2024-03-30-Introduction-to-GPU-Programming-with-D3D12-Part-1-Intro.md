---
layout: post
title:  "Introduction to GPU Programming with D3D12: Part 1 - Intro"
date:   2024-03-30 14:40:00 +0200
categories: Introduction_to_GPU_Programming_with_D3D12
lang: en
---

This is the first part of a series about GPU programming with D3D12. This series will cover how and why you would run code on a GPU, as well as some adjacent topics.

Note that some parts may be updated while whole series still coming out.

## Intended audience

This series is intended for people who would like an introduction to GPU programming, or who are already familiar with high level graphics API, but want an introduction for a low level one.

This series explicitly *does not* require you to know D3D11, in contrast to many other resources that can be found online.

## Prerequisites

* C++ – most Graphics APIs can only be directly used with C and C++. In this series we'll use C++ as it makes certain things simpler. This means that you'll need to have a certain level of proficiency with C++ before you can efficiently use said APIs.

* Math – this series will touch on both compute and graphics capabilities of modern GPUs. To program graphics you'll need a solid understanding of linear algebra.

* Modern GPU - we'll use certain features that are only available on fairly modern GPUs. Minimum Required GPUs: NVIDIA GeForce RTX 20 series, AMD Radeon RX 6000 series, Intel ARC.

## Why GPU Programming?

GPU is short for Graphics Processing Unit. They are a very common piece of hardware that is present in virtually any modern consumer computer/smartphone/game console/etc. In fact, you are most likely relying on one while reading this text right now.

Originally GPUs were designed to solve a specific problem: render 3D graphics. But over the years, as realism of 3D graphics advanced, GPUs became more and more generalized, with modern GPUs often used for computations that are no longer related to graphics: Scientific Simulations, Machine Learning, Cryptocurrency Mining, etc.

So, it is worth exploring how exactly are GPUs different from CPUs, why only certain use cases use them, and how to write and run your code on one.

## Why D3D12?

To interface with the GPU, you need to use what's called a “Graphics API”. Despite the name they allow you to run any kind of workload on the GPU, that GPU is capable of running, not just “Graphics”.

At the time of writing, there are many relevant Graphics APIs:

- D3D11
  - High level API
  - Windows exclusive
- D3D12
  - Low level API
  - Windows exclusive
- Metal
  - Low level API
  - Apple platforms exclusive
- Vulkan
  - Low level API
  - Cross platform
- WebGL
  - High level API
  - Web based API
- WebGPU
  - Low level API
  - Web based API

High level APIs were not selected, as many of modern GPU features, such as Hardware Accelerated Raytracing or Mesh Shaders, require low level APIs specifically.

Web-based APIs were not selected, as they are simply translated to other APIs, and so are more limited than the underlying API.

That leaves Vulkan, D3D12 and Metal, all of which are a perfectly fine choice. D3D12 was picked as it is slightly simpler compared to Vulkan, and more accessible compared to Metal. But many concepts and ideas are the same between those APIs.

[Next Post](Introduction-to-GPU-Programming-with-D3D12-Part-2-Why-do-we-need-GPUs.html)
