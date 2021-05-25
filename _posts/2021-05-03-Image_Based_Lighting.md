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

  We need to remark that for us to be able to represent a skybox regardless of the context of doing IBL or not, we need to sample each face a projection in itself, a lookAt matrix per face so it is pointing **inwards** towards the scene. Afterwards we will make sure that `glDepthFunc(GL_LEQUAL)` is being done in the depth comparison so the skybox is drawn behind all the other objects in the scene and lastly we will be drawing the projected cube through the shader.

(_This renders a specific cubemap face_)
```glsl
#version 330 

layout (location = 0) in vec3 a_position;

out vec3 modelSpacePosition;

uniform mat4 u_p_matrix;
uniform mat4 u_v_matrix;

void main()
{
    modelSpacePosition = a_position;  
    gl_Position =  u_p_matrix * u_v_matrix * vec4(modelSpacePosition, 1.0);
}
```
And this **Fragment Shader** samples the environment cubemap and outputs the color:
(_Notice how we are applying gamma correction before outputing the color to the screen_) 

```glsl
#version 330 core
out vec4 FragColor;

in vec3 localPos;
  
uniform samplerCube environmentMap;
  
void main()
{
    vec3 envColor = texture(environmentMap, localPos).rgb;
    
    // [Gamma Correction]
    envColor = envColor / (envColor + vec3(1.0)); 
    envColor = pow(envColor, vec3(1.0/2.2)); 
  
    FragColor = vec4(envColor, 1.0);
}
```
## Diffuse IBL

After we have the normal environment map calculated and displayed in the screen as a normal skybox we need to proceed as we previously stated, we need to pre-calculate the **irradiance** map and this is done through a **fragment shader** and as mentioned, to be able to calculate the final **irradiance** value we need to take into consideration all the previous **captured information**, so the code looks as following:

```glsl
#version 330

out vec4 FragColor;
in vec3 modelSpacePosition;
uniform samplerCube environmentMap;

const float PI = 3.14159265359;

void main()
{		
    vec3 normal = normalize(modelSpacePosition);
    vec3 irradiance = vec3(0.0);
    
    vec3 up = vec3(0.0, 1.0, 0.0);
    vec3 right = normalize(cross(up, normal));
    up = normalize(cross(normal, right));

    float sampleDelta = 0.025;
    float nrSamples = 0.0; 
    for(float phi = 0.0; phi < 2.0 * PI; phi += sampleDelta)
    {
        for(float theta = 0.0; theta < 0.5 * PI; theta += sampleDelta)
        {
            // spherical to cartesian (in tangent space)
            vec3 tangentSample = vec3(sin(theta) * cos(phi),  sin(theta) * sin(phi), cos(theta));
            // tangent space to world
            vec3 sampleVec = tangentSample.x * right + tangentSample.y * up + tangentSample.z * normal; 

            irradiance += texture(environmentMap, sampleVec).rgb * cos(theta) * sin(theta);
            nrSamples++;
        }
    }
    irradiance = PI * irradiance * (1.0 / float(nrSamples));

    FragColor = vec4(irradiance, 1.0);
}

```

  Notice how we are utilizing `cos`, `sin` and `pi` to be able to convolute, that is becuase we are going to use a spherical convolution like displayed in the following and we will be extracting an average for each final value while taking into consideration the other sampled values:
  
  ![Spherical Convolution Example](https://user-images.githubusercontent.com/48097484/119535350-2b9f6380-bd88-11eb-9841-15cab6d72805.png)

At the end of the day we will end up with an irradiance map that is the **blurred** version of the **environment map** if everything was calculated correctly and we will be able to utilize it in the **shader** to compute the **radiance** and **irradiance** values, which goes tightly hand to hand with the **PBR** rendering technique.

In this case for us to be able to calculate the diffuse part of the reflectance equation we can use the `FresnelSchlick` approximation function, although we will be adding a slight modification to it, adding a roughness value to it so at the end we do not end up with relatively smooth surfaces that reflect very strongly, with this little fix we get a more realistic approach because the fresnel calculations will not work as if the object was completely smooth, it will be determined by the roughness parameters of the microfacets.

```glsl
vec3 fresnelSchlickRoughness(float HdotV, vec3 baseReflectivity, float roughness)
{
    return baseReflectivity + 
    (max(vec3(1.0 - roughness), baseReflectivity) - baseReflectivity) * 
    pow(1.0 - HdotV, 5.0);
}

  
//Ambient
vec3 F = fresnelSchlickRoughness(NdotV, baseReflectivity, roughness);
vec3 kD = (1.0 - F) * (1.0 - metallic);
vec3 irradiance = texture(u_irradiance_map, N).rgb * albedo.xyz * kD;

```

At the end with the **diffuse IBL** we will end up with the following result: 

![Metro Engine Diffuse IBL](https://user-images.githubusercontent.com/48097484/119537675-9356ae00-bd8a-11eb-8503-abac594a1a9c.png)

As we can see we have done a linear scaling of the metallic and roughness values in the grid, that is why there is a clear visual difference between the spheres that are at the top right and the top left. Now to make it more realistic we are still lacking the specular component of the light to be able to see the reflection of the cubemap in our objects, this in other words is called the **Specular IBL**.

## Specular IBL

