CUDA SVGF
================
CIS 565: *GPU Programming and Architecture* Final Project
 - **Zheyuan Xie** [[GitHub](https://github.com/ZheyuanXie)] [[LinkedIn](https://www.linkedin.com/in/zheyuan-xie)]
 - **Yan Dong** [[GitHub](https://github.com/coffeiersama)] [[LinkedIn](https://www.linkedin.com/in/yan-dong-572b1113b)]
- **Weiqi Chen** [[GitHub](https://github.com/WaikeiChan)] [[LinkedIn](https://www.linkedin.com/in/weiqi-ricky-chen-2b04b2ab)]

![Demo (Cornell Box)](img/banner.png)

This is the final project for CIS 565: GPU Programming. The goal of the project is to denoise Monte-Carlo path traced images with low sample counts. We attempted two methods:

 - Spatiotemporal (Spatiotemporal Variance Guided Filtering)
 - Machine Learning (Kernel Predicting Convolutional Networks)

This repo contains our CUDA implementation of the Spatiotemporal method.

## Overview

Physically based monte-carlo path tracing can produce photo-realistc rendering of computer graphics scenes. However, even with today's hardware it is impossible to converge a scene quickly and meet the performance requirement for real-time interactive application such as games. To bring path tracing to real-time, we reduce sample counts per pixel to 1 and apply post-processing to eliminate noise.

*Signal processing* and *Accumulation* are two major techniques of denosing. Signal processing techniques blur out noise by applying spatial filters or machine learning to the output; Accumulation techniques make it possible to reuse samples in a moving scene by associating pixel between frames. *Spatio-Temporal Variance Guided Filter* [Schied 2017] combines these two techniques and enables high quaility real-time path tracing for dynamic scenes. 

## Demo

![](img/demo.gif)

## Path Tracing
The project is developed based on [CIS 565 Project 3](https://github.com/ZheyuanXie/Project3-CUDA-Path-Tracer). Addtion to the project 3, we implemented texture mapping and bounding volume hierarchy to accelerate path tracing for complex meshes. To reduce sample variance, if the ray hit matte surface, we trace a shadow ray directly to the light.

## Denoising - Spatiotemporal Method
![Pipeline](img/svgf.png)

### Temporal Accumulation
To reuse samples from the previous frame, we reproject each pixel sample to its prior frame and calculate its screen space coordinate. This is completed in the following steps:
1. Find world space position of the current frame in G-buffer. 
2. Transform from current world space to the previous clip space using the stored camera view matrix.
3. Transform from previous clip space to previous screen space using perspective projection.

For each reprojected sample we test its consistency by comparing current and previous G-buffer data (normals, positions, object IDs). If the test rejects, we discard the color and moment history of the corresponding pixel and set history length to zero.

### Spatial Filtering
The spatial filtering is accomplished by a-trous wavelet transform. As illustrated in the figure below, the a-trous wavelet transfrom hierarchically filters over multiple iterations with increasing kernel size but a constant number of non-zero elements. By inserting zeros between non-zero kernel weights, computational time does not increase quadratically.

![](img/atrous_kernel.png)

A set of edge stopping functions prevent the filter from overblurring important details. Three edge-stopping functions based on position, normal, and luminance are used as in *Edge-avoiding À-Trous wavelet transform for fast global illumination filtering*  [Dammertz et al. 2010]. The standard deviation term in luminance edge-stopping function is based on variance estimation. This will guide the filter to blur more in regions with more uncertainty, i.e. large variance.

## Build Instruction
 0. Make sure you have the required software installed:
  - Visual Studio (2017 or 2019)
  - CMake (Latest)
  - CUDA Toolkit (10.0)
 1. Clone this repository.
 ```
 $ git clone https://github.com/ZheyuanXie/CUDA-Path-Tracer-Denoising
 $ cd CUDA-Path-Tracer-Denoising
 ```
 2. Create a build folder.
 ```
 $ mkdir build && cd build
 ```
 3. Run CMake GUI.
 ```
 $ cmake-gui ..
 ```
 4. Configure the project in `Visual Studio 2017` or `Visual Stuio 2019` and `x64`, then click Generate.
 5. Open Visual Studio project and build in release mode.

## Project Timeline
### Milestone 1 (Nov. 18)
 - Revised codes from hw3 to generate data for next milestone.
 - Wrote framework code based on project 3.
 - Implemented A-Trous Wavelet Transform for spatial filtering.

[MS1 Slides](slides/MS1.pdf)
### Milestone 2 (Nov. 25)
 - Implemented texture mapping.
 - Built and tested denoising neural network for proof of concept.
 - Completed temporal accumulation and variance estimation with bugs.

[MS2 Slides](slides/MS2.pdf)
### Milestone 3 (Dec. 2)
 - Finised all major components in SVGF, achieved real-time denoising for basic scenes.
 - Constructed complex scenes
 - Batch-generated training data for denosing nerual network.
 - Added a GUI control panel for SVGF.

[MS3 Slides](slides/MS3.pdf)
### Final (Dec. 9)
 - Refactored path tracer, resolved bugs and optimized performance for SVGF.
 - Generated denoised images using trained network.

## Acknowledgments
 - [1] [Spatiotemporal Variance-Guided Filtering: Real-Time Reconstruction for Path-Traced Global Illumination](https://research.nvidia.com/publication/2017-07_Spatiotemporal-Variance-Guided-Filtering%3A): The SVGF part of the project is primarily implemented based on this paper.
 - [2] [Edge-avoiding À-Trous wavelet transform for fast global illumination filtering](https://dl.acm.org/citation.cfm?id=1921491): This paper with its code sample also helped a lot in the spatial filtering part.
 - [3] [Alain Galvan's Blog](https://alain.xyz/blog/raytracing-denoising): Alain's blog posts, as well as his Nov. 20 lecture talk at UPenn gave us a good overview and understanding of denosing technoligies.
 - [4] [Dear ImGui](https://github.com/ocornut/imgui): This library lets us create pretty-looking GUI with ease.
 - [5] [Kernel-Predicting Convolutional Networks for Denoising Monte Carlo Renderings](https://www.ece.ucsb.edu/~psen/Papers/SIGGRAPH17_KPCN_LoRes.pdf): The machine learning part of the project is implemented based on this paper.
