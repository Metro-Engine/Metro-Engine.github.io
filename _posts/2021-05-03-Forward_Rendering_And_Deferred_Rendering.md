---
layout: post
title: Metro Engine Rendering Technique
subtitle: Forward & Deferred Rendering
tags: [rendering, forwardrendering, deferredrendering, opengl]
---

## Rendering Technique

  The initial stages of how we rendered everything in Metro Engine was through forward rendering, which meant that for each **fragment**(_pixel_) we calculated lighting from all the light sources in the scene.
  
  This is a pretty straightforward rendering technique that is used in a lot of engines that do not require a lot of heavy lighting and the amount of lightings are controlled and limited.
  
  In contraposition it is very bad because each fragment gets overwritten by the next lighting and all the lighting calculations that were previously done are completely wasted, this means that a lof of computation is just wasted because information is being constantly overwritten in the fragment.
 
 {: .box-warning}
**Warning:** The specific cost for forward rendering is a **constant** cost of `num_fragments * num_lights`

  The pipeline in forward rendering is pretty linear, you need to feed to the GPU the geometry, project it with the **view_projection matrix** and transform it giving it a position, rotation and scale so it can later be split into fragments and the lighting calculation can be done per fragment.

![Linear Example of Forward Rendering](https://i.imgur.com/qztQhS2.png)

  Imagine that if you have 5 objects in a scene and 5 lights you will need to render every objects 5 times to see the image with the objects being affected by lighting from 5 different light sources, this means that it needs to do 25 operations per frame and only the uttermost calculation is the one that will be staying in the screen because it is constantly being **overwritten**. 
  
  Now imagine that instead of 5 lights you had 50, the number goes up to 250 operations per frame, this proves that this rendering technique despite being most used because of its simplicity, it is inefficient in the case of having a scene with a high number of light sources.

  This means that if your engine is going to be utilizing a lot of lighting which is done in most of the games, you will most probably be better utilizing other rendering techniques such as **Deferred Rendering**.



## Deferred Rendering

  As previously mentioned, deferred rendering is the perfect case for whenever you have a software in which you render lighting such as an engine and you do want to have multiple light sources without having any operations wasted.
  
  This of course comes with a drastic change in how we render objects, given that this technique allows us to render **hundreds** or even **thousands** of lights with a rather acceptable framerate without any kind of trottle.
  
  The idea of this rendering technique is to postpone the lighting calculations to a later stage, this means that we need to do passes, first we need to do what is called the **geometry pass** in which we render the scene and retrieve all the geometry visible in the clipping space of the camera, this means that we will be retrieving all the geometrical information and storing them in a collection of textures called the **G-Buffer**.

  This essentially means that we will be sort of doing a snapshot of specific parts of the scene and store them in a canvas, in one single frame, we will be storing values such as:
  
  - Position Vectors.
  - Color Vectors.
  - Normal Vectors.
  - Specular Values (Optionally).

  Once again this implies that we will be constantly doing snapshots and storing the information in a canvas and we will have this information separated in different canvas, think of having like 4 different sheets of paper size A4 in which you draw stuff, in this case whenever you do the geometry pass, in the first sheet of paper you store the positions, in the next one the normals, the other one the albedo calculations and the last one the specular values. 
  
   I know this might be a dumb way to explain it but this is a way to store information separatedly in different places and that allows us as the developers to utilize that information whenever needed for our own purposes like doing a postprocess that requires the normals of our geometries or a postprocess that requires the position of our geometries. It is a very useful way to separate the information and to not be overwriting data anywhere, you centralize and localize it in sheets of paper (_textures_) and use them for your specific purposes.
   
   With that said we know now that we have a G-Buffer (_which has all those sheets filled with different information_) and we use them for what we know as a **second pass** which in this case is the **lighting pass** where we generate a plane/quad and calculate the scene's lighting for each fragment using all the geometrical information that we have stored in the **G-Buffer**.
   
   ![Position,Color,Normals](https://i.imgur.com/eV4SpI8.png)
   
   `(Positions: Up-Left, Normals: Up-Right, Color: Middle)`
   
  For example in our engine we have the following textures (_sheets of paper_) filled with information about positions, color(_albedo_) and normals. This allows us to do a second pass 
  
  
  
