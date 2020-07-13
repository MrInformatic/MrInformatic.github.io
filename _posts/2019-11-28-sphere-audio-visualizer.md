---
layout: post
title: "Rendy Sphere Visualizer"
date: 2019-11-28
image: /assets/images/sphere_audio_visualizer_preview.jpg
summary: 
categories:
  - featured
  - free time project
links:
  - name: GitHub
    url: https://github.com/MrInformatic/rendy-sphere-visualizer
    color: red
  - name: Demo Video
    url: https://www.youtube.com/watch?v=Hfbo6E0vXDM
    color: blue
---

## Demo Video

### Raytraceing Version

<iframe width="1024" height="576" src="https://www.youtube.com/embed/qYGZSUkc3L0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Realtime Version

<iframe width="1024" height="576" src="https://www.youtube.com/embed/BZBFrcSU1_U" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## About

<!--excerpt.start-->

Rendy Sphere Visualizer is a render optimized for rendering spheres it uses different technics like distance field ambient occlusion and raytraced shadows to archive high fidelity graphics in real-time. I mainly use it as a visualizer for music. As the name implies I have used the Rendy rendering framework. Rendy uses gfx-hal which is a hardware abstraction layer that abstracts many graphics APIs like Vulkan, D3D, Metal, or OpenGL. Therefore, this project can run on a variety of platforms.

<!--excerpt.end-->

## Story

### Chapter 1: Sphere Audio Visualizer in Blender

I wanted to make an visualizer that uses spheres and physics to visualize the spectrum of audio. Therefor I created this spectrum audio visualizer. I wasn't realy pleased with the render time. I needed 11s per frame in Cycles. If I wanted to for example render 5 min of audio in 30 fps, It would take 27.5 hour to render this animation. This wasn't acceptable for me.

### Chapter 2: Raytracer in Java

The Project started as part of the bachelor module computer graphics. One part of the module was to develop a physical based monte carlo raytracer using Java. At the end there was a rendereing competition. The participation in the competition is on volentary basis. I was excited to participate at the competition. You could choose if you want to submit an image or an animation. I wanted to submit  the animation from the first chapter. I also wanted to use the whole 30s time limit. 30s * 30 fps results in 900 frames that have to be rendered somehow. At that time I needed approximately 120s per frame to get an acceptable result. That was not acceptable for me. Therefor I have to make my renderer faster.

My idea for getting better rendering performace was to reduce the ammount of samples I need per pixel without getting a noisier image. But what introduces the noise in a Monte carlo based raytracer you might ask. Well glad you ask essentialy everything that introduces random number generation. 

One source of noise are material where rays are reflected in 2 or more different ways, like glass where a ray could be reflected or refracted depending on the angle between the direction of the ray and the surface normal. I avoided this by returning all relected rays and weighting them according to the individual probability. 

Another source of noise are diffuse materials. Avoiding the noise introduced by diffuse matrials is a little bit more difficult. You could simulate diffuse lightning from point and directional lights with lambert's cosine law and simulate shadows with raytraceing. But you can't simulate indirect lightning and ambient occlusion that well without introducing much noise. Indirect lightning isn't that big of a deal in this scene but ambient occlusion is. It was at that time I heared of a new rendering technic called Raymarching. Raymarching is great for things like ambient occlusion because you could approximate it very ceaply useing this method. This method is also known as distance field ambient occlusion. 

The procedure works as follows for a hit you take an set ammount of points offset from the hit position in the direction of the hit normal and estimate the minimum distance for them. you then take the minimum distance and substract the offset. The result is zero if the hit position is the nearest point to the probe point. If the result is positive an other point is nearer to the probe point than the hit point meaning there must be an object near by blocking ambient light and therefore. 

And intigrating it in the raytracer was easy because I only use Spheres in that scene.

All these optimizations leads to a reduction of my rendering time from 120s to 6s per frame therfore 20 times faster. This was far more acceptable for me. I've also reached 1st place in the rendering competition.

### Chapter 3: Realtime Rendering using OpenGL

But 6s per frame wasn't enoght for me. I wanted to go realtime. Therefor I need to use a 3D Rendering API. The API of choise for this Project was OpenGL. I could have used Vulkan or Gfx Hal but this should be more of an proof of concept than an full featured rendering Engine. I wanted to combine my approatch with an defered rendering pipeline. Thats what I came up with.

#### G-Buffer Generation

Like every deffered Renderer I needed some kind of G-Buffer Generation.

#### Distance Field Ambient Occlusion

The Idea is the same as in Chapter 2. But now i had to split it up into 3 steps. The preperation, the distance estimation and the ambient occlusion step. The preperation step takes the position and moves it in the direction of the normal by a certain offset. The distance estimation step is performed for every sphere seperatly and takes the position and estimates the distance to the curent sphere. After that the distances get blended together with the min opperation. The ambient occlusion Step takes the estimated distances substracts the offset and multiplies the result with an factor. The results of this step gets blendet together with the reversed substraction opperation. But this can be optimized further because we know that the maximum distance our distance estimation can produce is the offset we only need to look at the pixels in that range. 

#### Raytraced Shadows

Yes no shadow mapping. This step looks for every sphere seperatly if a position could see the light. The results get blended together with the min opperation. But this operation also could get optimized further by only checking the pixels in the frustum of the light through the current sphere.

#### Composition

There is much going on in the final step first I decide if the point is a forground or background pixel. Next I calculate the direction of the camera to a point. Then I calculate the difuse light useing the lamert's cosine law. After that I approximate the frenell factor using the shlick equasion. I also sample the reflection from an cubemap. Finaly I bring everithing together.

#### Result

Does it run in realtime yes (1920x1080 60fps on my GTX 960 on Linux using the Nvidia 440 driver). Does it look pretty not so much.

### Chapter 4: Raytracer in Rust

Acutually this project is a little bit older than the Realtime renderer in OpenGL but it was finished after the realtime renderer. It is a port of the Java raytracer to Rust with some Rusty source and voila 0.5s per frame resulting 6 times increase in performace. But how did I achive that level of performace.

#### Performance analysis

It looks like that the traversal of the scene graph is one of the most used opperation be it for intersection or distance estimation. In the Java raytracer I used the compositum pattern to represent my scene graph. See a problem? OPP Guy: "No" Real Programmer: "What?". Therefor there are 3 strategies for dealing with this problem vectorization, more power, or some of this delicios rusty source.

#### Delicios rusty source 

Rust has this thing they call enum but the thing with the name it is complicated. It essentialy lets you treat an enum value as type and you could use pattern matching on an enum value to determin the enum value and extract its containing data. It is one very performant way of implementing polymorphism in rust. These enums have a size known at compile time. It is calculate from the biggest type in the enum plus the size of the discrimination value.

#### Results

Thats it replaceing all the Java polymorphism with Rust Enum polymorphism yields 6 times the performace of the Java Raytracer.

### Chapter 5: Further Thinking

I thought about different ways to make this raytracer faster. I could for example use the GPU to speed up my calculations using the optix framework from NVidia or raw opencl. Or I could try to convince my GPU to run rust llvm code. But I didn't look into these matters that deeply because GPU performance is relatet to dark magic, hours of trial and error and the good old guessing game. Thats why the optix route is the most appealing way becaus it was developed by nvidia engineers who should know their craft.