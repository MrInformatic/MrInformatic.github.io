---
layout: post
title: "Sphere Audio Visualizer"
date: 2023-05-01
image: /assets/images/sphere_audio_visualizer_preview.jpg
summary: 
categories:
  - featured
  - free time project
links:
  - name: GitHub
    url: https://github.com/MrInformatic/sphere-audio-visualizer
    color: red
  - name: Demo Video
    url: https://www.youtube.com/watch?v=hiAd1o6ydsU
    color: blue
---

## Demo Video

<iframe width="1024" height="576" src="https://www.youtube.com/embed/hiAd1o6ydsU" title="Sphere Audio Visualizer" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## About

<!--excerpt.start-->

The Sphere Audio Visualizer is a cross-platform audio visualizer based on GPU-accelerated visualizations. The shader used is implemented in both: rust, using [rust-gpu](https://github.com/EmbarkStudios/rust-gpu), and [WGSL](https://gpuweb.github.io/gpuweb/wgsl/). Thanks to [WGPU](https://wgpu.rs/) the software is compatible with Vulkan, D3D12, D3D11, Metal, WebGPU, WebGL and OpenGLES, and due to [GStreamer](https://gstreamer.freedesktop.org/) the software can export the results to a variety of encodings.

<!--excerpt.end-->

## Architecture

The basis of the program's architecture is a Pipeline, of which you can see a basic breakdown in the following picture:

![pipeline](/assets/images/sphere_audio_visualizer_pipeline.svg)

Let's go though the different steps.

### 1. Audio Analysis

The audio analysis takes the samples of the source audio and generates a range of volumes. I divided the frequency range into bands which grow in size exponentially with increasing frequency to mimic the human ear's frequency perception. To analyze the bands' individual volumes I used high- and low-pass IIR (infinite impulse response) filters to isolate each band. I also used an envelope filter with low attack and high release to smooth out the signal.

I also tried using FFT(fast Fourier transform) instead of IIR-Filters but in my experience, the result is less exciting. Especially the important bass frequency bands suffer the most. This is mainly caused by two things; First, due to the linear distribution of sampling points, multiple bass bands have to share the same FFT band. Second, due to the low time resolution, we might miss peaks in loudness which could be captured with a low attack envelope.  

### 2. / 4. Resampling

I use two resampling steps: One before the physics simulation, and one before the scene building. This is done to run the physics simulation in an independent, constant framerate to deliver consistent quality physics simulations. Also, the audio sample rate is way too high for physics simulation.

### 3. Physics simulation

For the physics simulation, I used [Rapier](https://rapier.rs/) as my physics engine of choice. Mainly because it is platform independent, uses SIMD instructions to accelerate computation, and is able to be compiled to WASM to be used in a browser. I implemented the Physics simulation for 2D and 3D.

### 5. Scene Building

The scene-building step exists mainly to convert the data from the physics simulation into a scene for the renderer. The converter usually adds the background or background elements and converts the spheres.

### 6. Rendering

I implemented two different renderers: a path tracer and a metaballs renderer. I used WGPU as the rendering backend for both renderers. WGPU is also platform-independent and can be used in browsers using WASM. It is also inspired by the new WebGPU browser API and is able to use it. As the metaballs renderer is rather uninteresting, I will skip a detailed explanation, and will instead focus on the path-tracing algorithm in the following paragraphs. 

## Path Tracing

Out of the two renderers, the Path-tracing renderer is the more interesting. Therefore I will break down how it works in the following sections. Since all renderers are implemented in WGSL and Rust, thanks to [rust-gpu](https://github.com/EmbarkStudios/rust-gpu), I will excerpt code from both implementations.

For rendering, I split the lighting into diffusive and reflective shading. The reflective shading uses path-tracing. The diffusing uses lambert shading and SDF(signed distance field) ambient occlusion. This is done to save performance by eliminating the noise introduced by raytraced diffuse lighting and by reducing the amount of supersampling. After that, I used Filmic tone mapping as a post-processing effect.

I could have used the new and shiny hardware raytracing APIs. But sadly you usually need hardware that supports this feature in APIs like Vulkan. Since I didn't own a graphics card with this ability, I decided to implement the ray tracing algorithm in software instead.

I tried to split up the path-tracing steps to save performance when vectorizing branches. Nonetheless, it seems to be way faster not splitting them. This is caused by continuously writing and reading intermediate results from and to memory. Since the memory bandwidth is limited, choosing a memory bandwidth-saving approach is faster than choosing a computation-saving approach. Therefore the whole rendering pipeline is implemented in a single monolithic shader.

Since I only used a limited amount of primitive shapes, like spheres and rectangles, I implemented the SDF and raytracing calculations for these shapes rather than converting them to triangles first. For simplicity, I haven't implemented acceleration methods like an acceleration structure for the raytracing or 3D textures for the SDF. I only used a bounding box around all shapes of each primitive type.

## GStreamer

GStreamer is my multimedia multi-tool. Everything from recording a device, like a microphone, capturing system audio, decoding audio in all different codecs, to decoding and muxing audio and video is done using GStreamer. Because it is not easy to install GStreamer inside a Github Actions runner, I published my code in a [separate repository](https://github.com/MrInformatic/install-gstreamer-action).

## P.S.

This was just a side project of mine. I do not think it has any other application other than being a toy project of mine. Therefore I will most likely not continue supporting it. Thanks for reading, if this was useful for you I would be happy to hear.