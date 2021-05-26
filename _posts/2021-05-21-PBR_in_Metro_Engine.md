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

  To be able to do a more precise calculation on the physics side of things, we are going to expand on the concept of capturing all incoming **irradiance** from all possible directions onto a specific point `p`. This will be known as the `hemisphere`, it is going to be centered around point `p` and it will capture and sum up all the radiance of the light that comes from all those directions, 360 degrees. It is important to remark that it is aligned with the surface's normal `n`.
  
  ![Hemisphere Exemplification](https://user-images.githubusercontent.com/48097484/119427689-bd1cc000-bd0b-11eb-9f32-dd6145ce5045.png)

  To be able to calculate all the values inside this hemispherical area, we utilize something that we might or might not know called an `integral`, it is shown in the reflectance equation with the `∫` shapeand it indicates the sum of all radiance in all the incoming directions ` dωi` withing the hemisphere `Ω`.
  
  All the calculations done for the sum of radiance onto point  `p` from direction `ωi` are also scaled by a factor `fr` that is mostly known as the **BRDF** or **bidirectional reflective distribution function** that essentially scaled the incoming radiance based on the surface's properties.
  
## BRDF (Bidirectional Reflective Distribution Funcion)

This function takes as input the following information:

- n (surface normal)
- ωo (outgoing view direction)
- ωi (incoming light direction)
- a (represents the roughness of the microsurface)

In essence this function will define how much each individual light ray with direction `ωi` has contributed to the final reflected light of an opaque surface given the properties of the object. If we have a mirror that fully reflects, the BRDF part of the reflectance function would return `0.0` for all incoming rays with direction `ωi`.

{: .box-warning}
**Warning:** If the incoming ray direction `ωi` is the same as the outgoing ray `ωo`, the BRDF function will return `1.0`, it only happens on that specific case.

Almost everyone nowadays utilizes a very famous BRDF called `Cook-Torrance BRDF` because it contains both the diffuse and specular parts and the formula is the following:

![BRDF Formula](https://user-images.githubusercontent.com/48097484/119431020-1851b100-bd12-11eb-8958-da0228dd5f67.png)

As previously mentioned at the beginning of the post, we know that we have a **refracted** and a **reflected** component for lighting, they both correspond to the `Kd` and `Ks` variables respectively, we notice that both of those variables in the formula are known to us, so no problem there, but we encounter a new one, the **lambertian diffuse** which is represented like this `flambert`, we need to remark that it is a constant factor that denotes the diffuse component and the formula is: 

![Lambertian Diffuse](https://user-images.githubusercontent.com/48097484/119431560-176d4f00-bd13-11eb-9cb7-58bed98cd685.png)

  In this case `c` is the albedo or surface color, in this case we divide by pi to be able to normalize the diffuse light as the reflectance equation is scaled by pi, so to normalize it back we divide it by the same quantity.
  
  In general, the previously mentioned `Cook-Torrance BRDF` is separated in various functions that each one of them do a specific thing to contribute to realistic lighting, `D`, `F` and `G` represent a function that is specialized in something very specific, and those are the following:
  
  - Normal distribution function.
  - Geometry function.
  - Fresnel equation.

  This equations and functions have their real life equivalent in physics, we can pick whatever approximated version we want such as [Trowbridge-Reiz GGX for `D` or the V_Smith GGX Fuction](https://google.github.io/filament/Filament.md.html). 
  
  So it is up to the developer to choose whichever function to utilize to represent the physical rendered lighting in the scene of the engine, in our case we went pretty default and utilized `Trowbridge-Reitz GGX`, `Fresnel-Schlick` for `F` and `Smith Schlick-GGX` for `G`.
  
  ![Cook Torrance Formula](https://user-images.githubusercontent.com/48097484/119432071-fa854b80-bd13-11eb-959c-5f3b903a40c6.png)

## Normal Distribution Function

  This function specifically approximates the surface area of a microfacets exactly aligned to the halfway vector `h`, it is an alignment corrector towards the normals given a specific roughness parameter. 
  
  ![Trowbridge-Reitz GGX Formula](https://user-images.githubusercontent.com/48097484/119432181-2dc7da80-bd14-11eb-8d11-f66d2325c77a.png)

The formula in its own seems complicated to understand but whenever we translate it to code it seems simpler:

```glsl

float DistributionGGX(float NdotH, float roughness)
{
    float a = roughness * roughness;
    float a2 = a*a;
    float NdotH2 = NdotH*NdotH;

    float num = a2;
    float denom = NdotH2 * (a2 - 1.0) + 1.0;
    denom = PI * denom * denom;

    return num / max(denom, 0.0000001);
}

```

## Geometry Function

![Geometry Function Example](https://user-images.githubusercontent.com/48097484/119432596-e4c45600-bd14-11eb-9fd6-8017873db660.png)

This function causes microsurfaces to overshadow each other, causing light rays to be occluded, giving a less uniform lighting look given a roughness parameter. The higher the actual roughness parameter is, the more chances your microsurface shave to overshadow other microsurfaces and the more shadowed the object is going to look like.

![Smith Schlick-GGX](https://user-images.githubusercontent.com/48097484/119432719-1c330280-bd15-11eb-9053-b52d5372aced.png)

Once again the formula might be daunting but it is not that difficult whenever represented in code, we need to consider that this is going to be a combination of two formulas Beckmann's approximation and GGX:

```glsl

// Schlick-Beckmann
float GeometrySchlickGGX(float valDot, float roughness)
{
    float r = roughness + 1.0;
    float k = (r * r) / 8.0;

    float num = valDot;
    float denom = valDot * (1.0 - k) + k;

    return num/denom;
}

// Smith GGX
float GeometrySmith(float NdotV, float NdotL, float roughness)
{
    
    float ggx2 = GeometrySchlickGGX(NdotV, roughness);
    float ggx1 = GeometrySchlickGGX(NdotL, roughness);

    return ggx1 * ggx2;
}


```

  The results will be clamped in a value between `[0.0, 1.0]` and the following changes can be appreciated:
  
  ![Geometry Function from 0.0 to 1.0](https://user-images.githubusercontent.com/48097484/119432905-73d16e00-bd15-11eb-8120-eb0fa23fdcec.png)

## Fresnel Equation

  ![Fresnel Example](https://user-images.githubusercontent.com/48097484/119433015-a713fd00-bd15-11eb-9170-46e62c3c39fa.png)

  The Fresnel equation is the ratio of ligth that gets reflected over the light that gets refracted and it varies depending on the `viewDir`, whenever we look at a surface from a specific perspective, straight in this case, we have a **base reflectivity**, whenever we start looking it from an angle, all the reflections start being more apparent compared to the base reflectivity when we looked straight, this effect can be approximated computationally wise with the `Fresnel-Schlick` approximation and it is the following one:
  
  ![Fresnel-Schlick](https://user-images.githubusercontent.com/48097484/119433154-f823f100-bd15-11eb-85e4-894a03a9973c.png)

  Obviously it looks daunting at first but whenever we represent this in code it simplifies a lot:
  
 ```glsl
 vec3 fresnelSchlick(float HdotV, vec3 baseReflectivity)
{
    return baseReflectivity + (1.0 - baseReflectivity) * pow(1.0 - HdotV, 5.0);
}

vec3 fresnelSchlickRoughness(float HdotV, vec3 baseReflectivity, float roughness)
{
    return baseReflectivity + 
    (max(vec3(1.0 - roughness), baseReflectivity) - baseReflectivity) * 
    pow(1.0 - HdotV, 5.0);
}

 ```
  We need to remark that this equation will only work for **Dielectric** (_non-metallic_) surfaces, for metallic surfaces the calculations are completely different.
  
## Full Cook-Torrance Reflectance Equation
 
 ![Full Cook-Torrance Reflectance Equation Example](https://user-images.githubusercontent.com/48097484/119433299-43d69a80-bd16-11eb-9003-2a3dfda25be2.png)

  As we can see, from the very simplistic form of calculating single rays towards point `p` with starting direction `ωo` towards light direction `ωi` we substituted it mostly with real life physical equation approximations that transform light in such a way to make it realistic. To be able to utilize this equation we need to have the output of our lights in our scene as previously explained in this post in case you have not read it, you can check it out [here](https://metro-engine.github.io/2021-05-13-Lighting_in_Metro_Engine/).
  
  For example to calculate the PBR lighting for the directional lights in our scene we have the following GLSL code:
  
```glsl

void main()
{

  vec3 camPos = u_camerapos.xyz;

	vec4 albedo = texture(u_color_texture, uvs);
	vec4 worldPosition = texture(u_position_texture, uvs);
	vec4 worldNormal = texture(u_normal_texture, uvs);
    
    float metallic = albedo.w;
    albedo.w = 1.0;

    float roughness = worldPosition.w;
    worldPosition.w = 1.0;

    float ao = worldNormal.w;
    worldNormal.w = 0.0;

    normalize(worldNormal);

    //Camera- fragment vector
    vec3 V = normalize(camPos - worldPosition.xyz);

    //0.04 is plastic metallic value (almost none) and albedo is the most metallic value. Depending on metallic would interpolate to one or another
    vec3 baseReflectivity = vec3(0.04);
    baseReflectivity = mix(baseReflectivity, vec3(albedo.xyz), metallic);
    vec3 radiance = vec3(0.0);

    vec4 lightSpacePositions[kMaxDirectionalLights];


    vec3 outputLuminance = vec3(0.0);
    int numOfLightsDirectional = int(numOfLights.x);
    int numOfLightsSpot = int(numOfLights.y);
    int numOfLightsPoint = int(numOfLights.z);

    vec3 N = worldNormal.xyz;
    float NdotV = max(dot(N, V), 0.0000001);

    for (int i = 0; i < numOfLightsDirectional; ++i)
    {
        lightSpacePositions[i] = u_light_p_matrix[i] * u_light_v_matrix[i] * worldPosition;
        radiance = CalculateDirectionalLights(directionalLights[i], worldNormal, lightSpacePositions[i], albedo, i).xyz;

        vec3 L = -directionalLights[i].direction.xyz;
        vec3 H = normalize(V + L);

        float NdotL = max(dot(N, L), 0.0000001);
        float NdotH = max(dot(N, H), 0.0);
        float HdotV = max(dot(H, V), 0.0);
        // Cook Torrance BRDF
        float NDF = DistributionGGX(NdotH, roughness);
        float G = GeometrySmith(NdotV, NdotL, roughness);
        vec3 F = fresnelSchlick(HdotV, baseReflectivity);

        vec3 kS = F;
        vec3 kD = vec3(1.0) - kS;
        kD *= 1.0 - metallic;

        vec3 numerator = NDF * G * F;
        float denominator = 4.0 * NdotV * NdotL;
        vec3 specular = numerator / max(denominator, 0.0000001);

        outputLuminance += (kD * albedo.xyz / PI + specular) * radiance * NdotL * (1 - CalculateShadowFactor(lightSpacePositions[i], vec4(-L,0.0), i));

    }
    
    
    vec3 color = ambient + outputLuminance;

    //Tone mapping
    color = color / (color + vec3(1.0));
    //Gama correction
    color = pow(color, vec3(1.0/2.2)); 

   	gColorResult = vec4(color, 1.0);

```

 Notice that we do some tone mapping and gamma correction as extra post-processing to have a more correct look on PBR. We later pass this information to the color g-buffer so it can be outputed to the final quad. In case you are interested in reading about **deferred rendering** you can check out this post [here](https://metro-engine.github.io/2021-05-03-Forward_Rendering_And_Deferred_Rendering/).
 
 
## Conclusion

  This rendering technique is not easy to explain mathematically speaking, there a lot of variables and factors that need to be considered, you need to have a lot of advanded knowledge about `radiance` or about the graphic-backend you are utilizing to be able to implement this without too much trouble, as far as the mathematic formulas for Cook-Torrance, you can find variations of those equations each one with slightly different effects that can enrich the look of your scene depending on your specific needs.
  
  As always, we need to thank [LearnOpenGL](https://learnopengl.com/PBR/Theory) and [PBR Filament](https://google.github.io/filament/Filament.md.html) for the images and the explanation on this rendering technique. 
