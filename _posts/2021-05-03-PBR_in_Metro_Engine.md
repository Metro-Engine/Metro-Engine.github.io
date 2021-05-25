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

  Diving straight into the reflectance equation we need to understand the first variable that we see, in this case the `L`, it is often referred as the **radiance** and it is used to quantify the magnitude or **strength** of luight coming from a single direction.
  
  Afterwards we have the next variable, `Φ` which is called **radiant flux** and it is the transmitted energy of a light source measured in **watts**. Our eyes can only perceive a specific area of the wavelength spectrum of light, it goes from 390 nm to 700nm. Those wavelengths are associated witha specific color and if we go out of those boundaries we will not be able to perceive them anymore, here is an image to clarify:
  
  ![Wavelength Light Spectrum](https://user-images.githubusercontent.com/48097484/119424215-4f6d9580-bd05-11eb-8111-81a2f79f48f7.png)

  The radiant flux in itself basically measures the total are of the different wavelengths we encounter in the image, but for the sake of simplification and simplicity for computer graphics, it is not going to act as a calculus for varying wavelength strengths but as an RGB triplet or as we say, the light's color.
  
  In addition to that we have the **solid angle** which can be defined with the `ω` symbol, as previously said it is going to help us calculate the area of a shape projected in an unit sphere, think of it the following way, imagine you are **inside** the sphere and you are looking **outwards** towards a specific direction where you see a **shape** of an object, the size of the **silhouette** of that given object makes out what we call the **solid angle**. 

![Solid Angle Example](https://user-images.githubusercontent.com/48097484/119424861-91e3a200-bd06-11eb-80eb-c27bc8ab7cfc.png)

Now we have the **radiant intensity** which is represented with the `I` symbol, this basically measures the amount of radiant flux there is per solid angle, in other words the strength of a light source over a specific projected area in a unit sphere.

![Radiant Intensity Example](https://user-images.githubusercontent.com/48097484/119424970-ceaf9900-bd06-11eb-9e85-c9887fecca22.png)

To be able to calculate this **radiant intensity** we utilize the following equation which in this case is:
![Radiant Intensity Equation](https://user-images.githubusercontent.com/48097484/119425036-ebe46780-bd06-11eb-860c-acc966aa7518.png)

  After having all the components of the puzzle, **radiant flux**, **radiant intensity**, **solid angle**, we can finally determine the **radiance** which is the total observed energy in an area `A` over the solid angle `ω` of a light radiant intensity of `Φ`.
  
  ![Radiance Calculation](https://user-images.githubusercontent.com/48097484/119425161-264e0480-bd07-11eb-9a57-7b2acf67bf51.png)

  If we split the sphere in halfs and we had a plane in the middle of it as you can observe in the image, radiance would be the amount of light in an area scaled by an indicent angle `θ` to the surface's normal as `cos θ`, the radiance is the strongest the more perpendicular it is to the surface's normal.
  
```glsl
float cosTheta = dot(lightDirection, Norm);
```
  Here we are interested in calculating a single ray's radiance value, we can do it if we convert the solid angle `ω` into a direction vector `ω` and the `A` which is the area over the solid angle into a point `p`, this way we will be able to utilize a single point's **radiance** value and calculate per-fragment radiance.
  
  In computer graphics all we care about is **all** the incoming light upon a point `p` which is the sum of all **radiance** known as **irradiance**.
  
  So going back to the reflectance equation:
  
![The Reflectance Equation PBR #2](https://user-images.githubusercontent.com/48097484/119425785-4cc06f80-bd08-11eb-88b7-c563ba3b9f44.png)

We now know that `L` is the radiance of some point `p` in an infinitely small solid angle (_the size of a pixel in the screen_) `ωi` which can be thought as the incoming direction vector `ωi`.

We also know that the closer the angle is to the perpendicular (_the normal of the plane_) the stronger the radiance is, so this way we know that `n * ωi` is the light's incident angle to the surface.

As we previously mentioned, our main goal here is to calculate something that we know as **irradiance** which is the sum of all the radiance onto a point `p` which represented with the equation terms is `Lo(p,ωo)`, it essentially translates to the sum of the reflected radiance of a point `p` in the direction `ωo` which is thought to be the outgoing direction in which the viewer is observing, in this case `the camera`, us.

Okay, hold up, this is starting to sound complicated, at this point we might have lost you for sure, a quick resume on what all this paragraph above means is that `Lo` pretty much measures the summatory of the light's irradiance onto a point `p` viewed from `ωo`, so think of it that you are the camera and you are casting a infinitely small light of ray to a specific fragment at position `p`, you are essentially calculating the **sum of the irradiance** at that specific point.



