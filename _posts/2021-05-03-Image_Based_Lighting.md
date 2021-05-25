---
layout: post
title: Image-Based Lighting in Metro Engine
subtitle: IBL implementation in Metro Engine
tags: [lighting, technique, brdf, pbr, ibl, cubemaps]
---

### Image Based Lighting

This is yet another advanced technique to improve the lighting on the objects in the scene, it is most commonly known as **Image Based Lighting**, implied that we would sample the lighting from an image and then reflect it on the surface of an object in some kind of way...

  This means that we need to gather the lighting from the surrounding environment as one big light source, this means that we need to utilize something that is pretty common in graphics programming and it is a **cubemap**, it is essentially a texture that contains 6 individual 2d textures that each one of them form a side of a cube, they are pretty useful because they allow us to **sample** using a direction vector and this is going to be heavily used in the IBL technique, in case you want to read more about cubemaps you can go [here](https://learnopengl.com/Advanced-OpenGL/Cubemaps).
  
  Now that we know that we can sample from the cubemap having a specific direction and that we need to treat the cubemap itself as a big light source, we can sort individual **texels** of the cubemap as light emitters, this way we can capture the "environment" global lighting and feed it into the objects to make them feel like they pertain to the environment in which they are created in, if we had a cubemap of a blue sky, we would like the actual objects to be lit by the sky to have a blue-ish color, it would feel out of place to have the objects not being affected by the blue sky the slightest, it makes them feel out of place.
  
  So to remember a bit and in case you have not read about it, we recommend reading the [PBR post](https://metro-engine.github.io/2021-05-03-PBR_in_Metro_Engine/)  on our blog to be able to understand this technique better because they go hand in hand.
  
  ![Cook-Torrance BRDF Reflectance Equation](https://user-images.githubusercontent.com/48097484/119504743-8dea6b00-bd6c-11eb-82c8-040ebb01d4ac.png)

 As mentioned in the **PBR** post, this equation effectively solves the integral of all incoming light directions `ωi` over the hemisphere `Ω`, this was pretty easy to calculate considering that with the PBR rendering pipeline we knew that we had for example 5 lights and 5 directions from wihch we could sample the sum of the radiance in the hemisphere, this is not the case for IBL.
 
 Every single **texel** of the cubemap is going to act as a light emitter, this means that the amount of lights in the scene this time is going to be humongous, this quickly becomes unfeasible to take **every single individual** direction, this means that we need to find ways to approach this in a more feasible way such as finding a way to retrieve the radiance given any direction `ωi` and we need to agilize the calculation of the integral because this needs to be **fast** and in **real-time** because it is integrated in a graphic engine.
 
 
 
 
