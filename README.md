<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/alchemist_logo.png?raw=true"/>
</p>

Alchemist is a raytracer I wrote for my final solo project in the computer graphics course at the University of Waterloo [CS488](https://student.cs.uwaterloo.ca/~cs488/gallery.html). I was interested in soft lighting, caustic light effects, and emulating realistic textures. To do so, I implemented [photon mapping](https://en.wikipedia.org/wiki/Photon_mapping), distributed raytracing techniques, and an in-depth material system among other features. Since the renderer was CPU bound, I also leveraged multithreading techniques to massively speed up the rendering process. 

This project was written over the course of a couple weeks, and despite it's rough edges, I'm very proud of the final results!

# Index 
- [Sample Images](#sample-images)
- [Building](#building)
- [Code Structure](#code-structure)
- [Features](#features)
- [Next Steps](#next-steps)

# Sample Images
<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/main_collage.png?raw=true" width="700"/>
</p>

Some more renders can be found in `./SampleImages`. All of the scenes were composed using `lua` and can be found under `./Assets/SampleScenes`.

# Building
This project currently depends on some Lua scene description boilerplate provided by the university. This is copyrighted code that I was not allowed to redstribute, so for the time being, this repo is not in a buildable state.

# Code Structure
Class headers can be found under `include`, and implmentations can be found under `libs`.

# Features
Here is the list of features I implemetented for this renderer.

## Primitives
<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/primitives.png?raw=true" width="700"/>
</p>

I implmented intersection algorithms for spheres, cubes, cylinders and cones. Along with primitives, I also implemented rendering for `.obj` models.


## Bounding Boxes
<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/bounding_boxes.png?raw=true" width="700"/>
</p>

To speed up rendering scenes with `.obj` models, I added rectangular prism bounding boxes. This allows skipping intersection calculations from being run unecessarily.


## Texture Mapping & Normal Mapping
<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/mapping.png?raw=true" width="700"/>
</p>

I added texture mapping and normal mapping for primitives. This allowed for creating some more interesting scenes! This involved making a mapping function from the coordinates on each primitive to a coordinate on a square grid representing the texture.


## Adaptive Antialiasing
I implemented adaptive antialiasing using jittered supersampling to smooth out rough edges. 

Using a templated image kernel class (`include/ImageKernelHelpers.hpp`) I performed a simple edge detection pass using variance. This pass returned all the pixel coordinates corresponding to edges. 

<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/edge_detection.png?raw=true" width="700"/>
</p>

I then re-rendered these pixels using multiple rays distributed within the original pixel area and took their average value. Here's the before and after!

<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/antialiasing.png?raw=true" width="700"/>
</p>

## Reflective And Refractive Materials
<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/reflections_refractions.png?raw=true" width="700"/>
</p>

Since I was interested in rendering caustics, I added support for reflective and refractive materials. I also added distribution-based ray redirection which allowed for modeling soft reflections and refractions!

## Multithreading
I implemented this renderer purely using the CPU. I used a pool-based queuing system and many threads to massively speed up the rendering process.

## Caustics (via Photon Mapping)
<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/caustics.png?raw=true" width="700"/>
</p>

Caustics was the most involved feature that I implemented for this project. I achieved this effect using photon mapping! This technique allows for caustic effects to converge much faster than with other rendering techniques such as path tracing.

In short, before doing the main render pass, "photons" are traced exiting from all the light sources in the scene. These photons interact with the scene (passting through refractive materials, and bouncing off of reflective ones), and are collected on diffuse surfaces. Here's a visualization of this photon step where each one is represented by a sphere:

<p align="center">
  <img src="https://github.com/shraacha/alchemist-render-engine/blob/main/ReadmeImages/photons.png?raw=true" width="400"/>
</p>

During the main rendering pass, i.e. after tracing the path of a ray from the camera out into the scene and hitting a surface, the photons near the intersection are collected and used in the lighting calculation.

You may notice that there are **alot** of photons that are produced by this step! To efficiently store the photon information, I wrote a (templated) KD tree class `/include/KDTree.hpp`. This allowed me to quickly look up the `n` closest photons to an intersection point.

# Next Steps
Here's a list of things I'd like to add to this project next!
- [ ] hand-rolled scene-loading code 
