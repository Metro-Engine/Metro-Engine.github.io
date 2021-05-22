---
layout: post
title: Shadow Mapping
subtitle: How shadows work in Metro Engine
tags: [shadow, texture, lighting, engine]
---

## Shadow Mapping

  We all know that this technique revolves around **shadows** just because of the name, and shadows happen whenever there is absence of light due to some kind of **occlusion**, this means that whenever a light source does not hit an object because it is occluded behind a wall or some other object, the object is in shadow.
  
  The idea is rather simple, we basically **render the scene from the light's perspective**, because we know that everything that is **seen** by the light is **lit**, this means that everything that **does** not appear from the light's perspective needs to be in **shadow**.
  
  ![Light Shadow Mapping Example #1](https://user-images.githubusercontent.com/48097484/119214127-a173a880-bac4-11eb-81fa-b2d16c27d296.png)

  From this basic principle and the image shown above we can see that the light is illuminating the blue surfaces of the red squares and the black color represents what has to be in shadow. This whenever applied to computer graphics you logically might think that we need to imitate real life **light sources**, this quickly becomes unfeasible because if we cast **rays** from our **light sources** to everywhere in the **scene** and start comparing the closest point to the other points in the **ray**, it impies that we need to iterate through roughly **thousands or millions** of light rays per **light source**, you can quickly realize that this becomes unfeasible whenever we try to apply it to computer graphics.
  
  The goal is to have an algorithm that can be ran at **runtime**, implying that we need to seek for a more optimized and better approach to shadow mapping, and luckily there is a few techniques that will allow us to create shadows and be able to execute them in **runtime** with little to no cost at all!

  To be able to achieve this technique we need to utilize something called the **depth buffer**, it is a buffer that is filled with information of the depth of a fragment in a scene clamped to `[0, 1]`, this everything is done from the **camera's point of view**.
  
  What if I told you that we could pretty much do the same with a depth buffer but instead of rendering the scene from camera we render it from each one of the **light sources** and store that information in a texture. This way we are storing all the information about depth from each perspective of the light, implying that we now can **sample** the **closest depth** and see it from the light's perspective, the values that are the furthest are not stored. Instead of calling it a depth map now, those textures can be called "**shadow maps**".
  
  ![Projected Shadow Directional Light](https://user-images.githubusercontent.com/48097484/119215217-81e07e00-bacc-11eb-9cd1-42932bc898ca.png)

  In the image above we see a **directional light** being casted in the scene on a red square, we can see that all the values that are being stored in the **shadow map** are the yellow part, and the black part represents the projected shadow, take care that this is from the **light's perspective**.
  
  How do we determine if it is a valid value or a shadow? Using the depth values that are stored in the texture you find the **closest** point and use that to determine whether the fragment is in shadow, we render the scene from the **light's perspective** using a **view and projection matrix** specific to that light source, essentially transforming any _3D coordinates_ into **coordinate space**.
  
  In the image we can see that we have two points, `P` and `C`, P being the furthest one and C being the closest one, to determine which one is in shadow and which one is being lit by the light, we need to do a simple comparison between them, the one that is smaller is the one that is being lit, the other one is immediately identified as a shadow.
  
  Of course we need to transform the point `P` to **coordinate space** to be able to compare with point `C` and we can use point `P` to index the shadow map to obtain the closest visible depth from the light's perspective, otherwise known as `C`.
  
  Essentiallyw hat we are doing is doing two passes, we render the depth map and in the second map we render the scene as usual but using the previously generated depth map in the previous pass to calculate whether fragments are in shadow or not.
  
## Implementation
  
  With that explained, we will start explaining how is the actual **implementation** of the actual technique in our **engine**.
  
  
  
