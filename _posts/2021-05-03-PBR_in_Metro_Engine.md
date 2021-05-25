---
layout: post
title: Physically Based Rendering in Metro Engine
tags: [pbr, realistic, physics, math, engine, lighting]
---

## Physical Based Rendering

  This is a rendering technique that tries to mimimc real life lighting, that is why it is called "physically based rendering", we want to stray further from the classic "Blinn-Phong" rendering technique and we want to get closer to how really light interacts with everything in real life.
  
  This is why we first need to know about a few advanced topics such as `framebuffers`, `cubemaps`, `normal mapping` to be able to continue further with this technique without any mishaps. For us to be able to mimic reality we need to get to a very "microscopical" level and that is why the `microfacet model` exists.
  
### The Microfacet Model

  Pretty much all the PBR techniques that you will see around the internet are based on this theory of **microfacets**. In a microscopic scale we will have tiny reflective mirrors called **microfacets**, those mirros depending on given properties such as **roughness** or the alignment they have will define in which direction a ray might scatter for example.
  
  Here is an example of a rough surface and a smooth surface: 
  
  ![Rough Vs Smooth Surface Clean With No Rays](https://user-images.githubusercontent.com/48097484/119421020-0534e600-bcfe-11eb-9ef8-651c69b62c69.png)

  We need to consider that the rougher a surface is the more chaotic it will be and the more the light rays will scatter around the surface outwards whereas in a smoother surface the reflection will be more uniform and we will be able to see a reflection (_just like a real life mirror!_).
  
  ![Rough Vs Smooth Surface with Light Rays Casted](https://user-images.githubusercontent.com/48097484/119421090-30b7d080-bcfe-11eb-8375-cbc368dff809.png)

  {: .box-note}
**Note:** Consider that no surface is going to be perfectly 100% smooth at a microscopic level, but at least we can have a clear distinction between what is very rough and what is clearly smoothed out.

To be able to have this distinction between rough and smooth surfaces we need to introduce a **roughness parameter**, based on this parameter we will be calculating the ratio of microfacets that are aligned to a specific direction vector `h`, `h` being the **halfway vector** that sits between the light `l` and the view vector `v`.

![Half Vector Calculation](https://user-images.githubusercontent.com/48097484/119421251-93a96780-bcfe-11eb-91ea-6b6fc8b4f021.png)

  This means that the more the microfacets are aligned to the halfway vector `h` the sharper and stronger the specular reflection of the object will be, in addition to that we still have the roughness parameter that we can willingly tweak to approach the microfacets to our liking as in the following image:
  
  ![Roughness 0.1 to 1.0](https://user-images.githubusercontent.com/48097484/119421306-b5a2ea00-bcfe-11eb-95fc-a050b80dd409.png)

We can see that if we have almost zero roughness the specular is sharper whereas if we get it close to `0.8` or `1.0` the specular tends to be more difuminated, implying that the light rays scatter more compared to when they are close to zero where the rays almost do not scatter.


