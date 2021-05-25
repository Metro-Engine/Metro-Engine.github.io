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

We can see that if we have almost zero roughness the specular is sharper whereas if we get it close to `0.8` or `1.0` the specular tends to be more difuminated, implying that the light rays scatter more compared to when they are close to zero where the rays almost **do not** scatter.

### Energy Conservation

  Essentially this means that the light leaving a surface can never be brighter than the light that fell upon it. For us to have a clear distinction of this and have a real appreciation if there is really energy conservation we need to separate the light in their diffuse and specular components `Kd` and `Ks`. We need to make obvious that the moment an actual light ray hits a surface it splits in **two** components, the **REFRACTED** part and the **REFLECTED** part.
  
  We can deduct that the **reflected** part is obviously the part that hits the surface and does not get absorbed in it, it just purely bounces off the surface whereas the **refracted** one partially enters the surface and gets absorbed in the object, this is what we know as the **diffuse** component of the light, the one that gets absorbed in the object's surface.
  
  ![Refracted Vs Reflected](https://user-images.githubusercontent.com/48097484/119422838-30213900-bd02-11eb-927e-605cd3011a18.png)

  We need to consider that for our current implementation of **PBR** both components **Kd** and **Ks** are mutually **exclusive** ths means that light that gets reflected will no longer get absorbed by the object in the shape of **diffuse lighting** and likewise for the refracted one, it will not emerge towards the surface of the object to contribute to reflected light, this is just a simplification so we can have an easier distinction between both components. 
  
  To be able to **preserve** the energy we first calculate the **specular** fraction percentage of the incoming light and then substract it the leftover to get the diffuse.
  
```glsl
float kS = calculateSpecularComponent();
float kD = 1.0 - kS;
```

This way we make sure that **both** components never exceed the value of `1.0`, this allows us to control the energy conservation principle, something that was not taken into account for example in the `Blinn-Phong` rendering technique for lighting.

### The Reflectance Equation

(_Reference Table for some mathematic symbols_)
| Symbol | Short Description |
| :------ |:--- |
| L | Denoted as the **radiance** in the reflectance equation. | 
| Φ | Denoted as the radiant flux, it is the transmitted energy of a light source measured in **Watts**. | 
| ω | Denoted as the **solid angle**, it tells us the area of a shape projected onto a unit sphere. | 
| I | Denoted as the **radiant intensity**, it is used to measure the amount of **radiant flux** per solid angle. |

  Now comes one of the **hardest** parts of this post, breaking down the **reflectance equation** to be able to fully understand **PBR** and how does it work internally as far as mathematics goes. Just to get a bit friendly with the equation, here is a first glance at it, it might appear a bit daunting but we will try our best to break it down so you can understand it, and we hopefully can understand it a bit better (_haha..._).
  
  ![The Reflectance Equation PBR](https://user-images.githubusercontent.com/48097484/119423829-855e4a00-bd04-11eb-8f03-f892188bf72e.png)

  


