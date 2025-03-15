---
layout: post
title:  "Introduction to GPU Programming with D3D12: Part 3 - History"
date:   2025-03-16 4:01:00 +0900
categories: Introduction_to_GPU_Programming_with_D3D12
lang: en
---

[Previous Post](Introduction-to-GPU-Programming-with-D3D12-Part-2-Why-do-we-need-GPUs.html)

Modern GPU programming has its roots in real-time 3D graphics. Understanding the history behind that will allow you to more easily understand why GPU Programming ended up being the way it is. At this point you don't need to understand all of used techniques in detail, but rather see the idea of how graphics progressed through the years.

Note that games in this list may not be the first ones to implement respective techniques. As it turns out it is surprisingly hard to figure out which game was first to use any of those things.

## Geometry transformation

Battlezone (1980) used wireframe rendering.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/Ctr54kopo8I?si=EscADUKrUjz3W9kv" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Objects are represented as points with lines between them. To render them, it accounts for position, rotation and scale of each object, as well as position, rotation and field of view of the camera, to project those points onto a screen. Then it can draw lines between those points to create an illusion of a 3D object.

The major downside of that render is that it is just lines. Objects in front do not obscure objects behind.

## Triangles

I, Robot (1984) used solid triangle rendering.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/gmvWxG2zvs8?si=C6YQvcufy_aO7bck" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

By using triangle instead of line, you can define area of a screen that can be filled. However, that by itself does not yet solve the occlusion problem. To get proper occlusion you also need to ensure that object that is closer to camera is drawn on top of objects that are further from camera.

We can achieve that by sorting objects based on distance to camera, so that in the resulting list we first have objects that are in the back, and in the end are objects that are in the front, aka back to front sorting. That way, any pixel filled by objects in the back can be overwritten by objects in the front, effectively implementing occlusion. This is also known as a [Painter's algorithm](https://en.wikipedia.org/wiki/Painter%27s_algorithm).

There's one more issue though, you also need to ensure correct occlusion between different triangles within your object. That means that you also need to also sort triangles within the object back to front. But there's one more trick we can use to cut down on number of triangles we need to sort and draw. For each triangle, we can define direction which it will be facing. With that information we can check whether it faces towards or away from camera. We know that all triangles that we'll end up seeing must face the camera, so we can just skip all the triangles facing away from camera. That is known as back-face culling.

## Texturing

Ultima Underworld: The Stygian Abyss (1992) used texture mapping.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/ee4PUcpGSn8?si=ZAKEYQxgPbdf7k2A" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This allows triangles to not just be solid color filled. This is done by mapping a Texture, which you can think of as a 2D grid of colors, to a triangle. Each vertex is mapped to a point on that texture, and when you draw a triangle, you can interpolate position on a texture based on distance to vertices of a triangle, read the value from that point on a texture and then use that color for the pixel.

## Depth Buffer

Quake (1996) used depth buffer.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/ZHT2TgMX7Rg?si=yVbRU-nClFMBJSQP" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Up to now you had to sort all objects and triangles back to front to get correct order. But there's another method. What if instead of sorting triangles, we'll just keep track of how far is each pixel from the camera? For that we'll create texture the size of the screen, in which each pixel will store float value corresponding to distance from the camera. That way we can skip the sorting of objects and triangles, as well as get additional information about resulting image that we can later use for some additional effects.

Here's an example of how depth buffer may look. Click for see full resolution image.

[![Depth Buffer]({{ site.baseurl }}/images/DepthBuffer.webp)]({{ site.baseurl }}/images/DepthBuffer.png)

Also, since you no longer need to have defined, back to front order of objects and triangles, with this method you can also handle cases of intersecting objects and triangles.

## Lighting

Quake (1996) also used shaded lighting.

Let's rehash how you draw a triangle: First you get your vertices, perform certain calculations for each vertex, specifically mapping to points on the screen, find out the set of pixels on the screen that belongs to the triangle, and for each pixel perform calculations required for texture mapping. That's good, but why not perform more computations so we can try getting more realistic results.

What Quake did was calculate lighting per vertex. Since you know the position of vertex in the scene, you can loop over all relevant light sources and for each one calculate how much light does the surface get near the vertex. Sum up results from all lights, and you get lighting for your vertex. Then, when drawing each pixel, in addition to interpolating texture coordinates, interpolate lighting result and you get basic lighting.

Unreal (1998) used per-pixel shaded lighting.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/J4fJWvckmFQ?si=wU7qFyi12XdbGJep" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Same idea as what you can see in Quake, but instead of calculating lighting per vertex and then interpolating results, Unreal does calculations per-pixel, which provides more accurate results. Fun fact, this is the first game that runs on an Unreal Engine and is the reason for engine's name.

## Shadow Mapping

Oddworld: Munch's Oddysee (2001) used shadow mapping.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/Aa_hZKSrck0?si=Vz6Rsdq7HiOl5s86" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Turns out, you can solve problem of calculating shadow with creative usage of depth buffer. First, you render scene from point of view of light with depth buffer. After that is done, you now have a texture that has all geometry that is directly visible from the light. Any other geometry, that can not be seen from lights point of view would then appear in shadow. Using that texture you can check whether a point on surface is visible or not.

Here's an example of how such shadowmap can look. Click for see full resolution image.

[![Shadowmap]({{ site.baseurl }}/images/Shadowmap.webp)]({{ site.baseurl }}/images/Shadowmap.png)

## Programmable Shaders

Nvidia's Tech Demo Chameleon (2001) used programmable shaders

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/Qt6Vb09qrrI?si=ctB7SIH2phpU3zcL" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Real-time graphics are only really feasible when rendered by a GPU. But up till this demo, GPUs were not able to perform arbitrary calculations. All of the calculations you needed for simple lighting and shadow map usage were performed by specialized hardware, which were not capable to run arbitrary programs.

This demo shows one of the first examples of "shaders". Shader is just a program that can run on a GPU. There are multiple types of shaders, but first one to appear are Vertex and Pixel shaders. Vertex shaders are executed per-vertex, and they calculate properties of a vertex the are executed for. While Pixel shaders are executed per-pixel, and in simplest terms, responsible for calculating final color of a pixel.

This demo shows what Pixel shaders can acomplish, specifically how pixel shaders allow surface to change its properties based on certain conditions.

## Compute Shaders

Battlefield: Bad Company 2 (2010) used Compute Shaders.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/-AxiUMiKGLY?si=OFAJoKVLLUJBJWL4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

As the name suggest Compute Shaders is a new type of a shader. Unlike Vertex or Pixel shaders, the are not tied to graphics directly, you don't need any geometry to start compute shaders. The only thing you need is the shaders itself, and a number of instances to run. That allows for great flexibility of what can be done with those.

The way it was used in Battlefield: Bad Company 2 is quite interesting. First, all of geometry is rendered, but instead of calculating lighting immediatelly, we save information about each pixel in textures, to calculate lighting later. After all geometry was rendered, we calculate lighting for all pixels with compute shader. One of the benefits for doing that is that you can pre-calculate list of lights that affect each portion of the screen, as by the time all geometry is rendered, you have complete depth buffer and can calculate bounding box of each portion of a screen.

## Hardware accelerated raytracing

Battlefield V (2018) used Hardware accelerated raytracing.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/F-ZlMl9L3kM?si=LXGd-3kmEj7v-G1r" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Up until now, real-time computer graphics used approach of going over all geometry to figure out what each pixel gets. With ray-tracing, you would go other way around, from pixel to geometry. First, you build an "Acceleration Structure", that contains geometry you are interested in, then, you are able to query it for intersection with rays. For example, to figure out which geometry is visible at a given pixel, you would cast a ray from camera, that corresponds to a given pixel, and the closest geometry hit by that ray, is the geometry that is visible for a given pixel.

## Why is all of this important?

Despite the fact that the history of real-time graphics spans many decades, all of the same concepts and ideas are still used today. Every single thing described here, is still available and used in practice in modern applications.
