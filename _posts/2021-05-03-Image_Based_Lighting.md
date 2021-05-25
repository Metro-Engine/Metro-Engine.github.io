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
 
 For us to be able to accomplish the first objective which is to be able to sample in any given direction, we will just unwrap the cubemap texture as if we were unwrapping it with the UVs, but instead of utilizing the Uvs, we utilize the `ωi` direction vector, this way we will be able to constantly sample the radiance in any given direction of the cubemap without any trouble.
 
```glsl
vec3 radiance = texture(environmentCubemap, w_i).rgb;
```
 
 As for the second issue, we are uncapable of processing all the incoming directions of the cubemap just like that, so as we always do in programming, we try to pre-compute. So we will be pre-computing a new cubemap that stores each sample direction `ωo` by pre-calculating it with the diffuse part of the integral. This effect will be solved through **convolution**.
 
 In essence the concept of convolution here is taking all the input data from the cubemap and calculate a final value while taking into consideration all the previous values of given cubemap, we pretty much take into consideration all the values in the `hemisphere`, think of it as blurring the cubemap or averaging the results altogether.
 
 ![Cubemap Convolution](https://user-images.githubusercontent.com/48097484/119525799-1bcf5180-bd7f-11eb-99d0-1a14539abc4b.png)

 This will serve for us to get the radiance and irradiance values given the direction `ωi` in an easier and pre-calculated manner, essentially fixing both the problems we had commented previously.
 
## HDR Environments

For instance all the cubemap images that we will most probably be utilizing if we **do not** have IBL are LDR(_Low Dynamic Range_) cubemap textures, this means that the color never surpass a certain treshold, they stay between `[0.0, 1.0]`, this results in a **los** of information for the color and lighting on the cubemap and whenever we utilize the cubemap texels as **light emitters** we need to consider that having a loss in information due to the range of the colors in the cubemap is going to affect the IBL technique and the realism it could provide us.

That is why whenever we sample a cubemap for the sole purpose of utilizing it for IBL we will be sampling it with the **.hdr** format, this format essentially encodes the colors in 8 bits per channel and alpha being the exponent.

Having this format is going to be able to help us with the **losing precision** issue but it comes with an exchange, the map itself is distorted, it does not show all the 6 faces individually just like cubemap faces, this kind of cubemaps are projected onto a sphere and afterwards projected in a flat plane, this means that we will need to utilize an **equirectangular map** for us to be able to sample the **hdr** file.

![Equirectangular Map](https://user-images.githubusercontent.com/48097484/119527520-ae242500-bd80-11eb-835f-7c68240616d6.png)

So for us to be able to load an **hdr** file to be able to treat the equirectangular map we do the following conde on a **render command**:

```cpp
ImageLoader::SetFlipVerticallyOnLoad(true);
  const u32 faceSize = 512;
  ScopedPtr<Texture> sourceHDRTexture;
  sourceHDRTexture.Alloc();
  //LOAD EQUIRECTANGULAR ENV MAP
  int width, height, nrChannels;
  if (tex->equirectangularCubemapFilePath_.find(std::string(".hdr")) != std::string::npos)
  {
    isHDR_ = true;
    float* data = ImageLoader::LoadImageFromFileF(tex->equirectangularCubemapFilePath_.c_str(), &width, &height, &nrChannels, 0);
    if (data)
    {
      unsigned int hdrTexture;
      glGenTextures(1, &hdrTexture);
      glBindTexture(GL_TEXTURE_2D, hdrTexture);
      glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB16F, width, height, 0, GL_RGB, GL_FLOAT, data);

      glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
      glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
      glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
      glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

      sourceHDRTexture->setTextureID(hdrTexture);
    }
    else
    {
      printf("Cubemap tex failed to load at path: %s\n", tex->equirectangularCubemapFilePath_.c_str());
    }

    ImageLoader::FreeImageData(data);
  }
```

{: .box-note}
**Note:** Note how we store the texture with the `GL_RGB16F` format so we can utilize the float format of the OpenGL textures to treat the information in the **hdr** file.

Now that we have the equirectangular map loaded thanks to **stbi_image**, we will be sampling it in an unitary cube in the scene, in this given cube we will be sampling the map into 6 different images that represent each cube's face, essentially doing a conversion to an oldschool cubemap as you can clearly see.

```glsl
#version 330

out vec4 FragColor;
in vec3 modelSpacePosition;
uniform sampler2D equirectangularMap;

const vec2 invAtan = vec2(0.1591, 0.3183);

vec2 SampleSphericalMap(vec3 v)
{
    vec2 uv = vec2(atan(v.z, v.x), asin(v.y));
    uv *= invAtan;
    uv += 0.5;
    return uv;
}

void main()
{		
    vec2 uv = SampleSphericalMap(normalize(modelSpacePosition)); // make sure to normalize localPos
    vec3 color = texture(equirectangularMap, uv).rgb;
    
    FragColor = vec4(color, 1.0);
}
```
If everything went correctly and as expected, you shall be seeing a unit cube in the center of the scene with all the 6 faces displayed correctly:

![Equirectangular Map Demonstration](https://user-images.githubusercontent.com/48097484/119528718-bb8ddf00-bd81-11eb-9d79-245633061f8a.png)

  

  

