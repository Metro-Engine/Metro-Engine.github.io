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

  
