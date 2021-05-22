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
