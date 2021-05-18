---
layout: post
title: Normal Mapping & Parallax Mapping
tags: [normals, normalmapping, parallax, parallaxmapping, technique, opengl]
---

## Normal Mapping

  To be able to add some extra detail to quads or meshes in our scene, in this engine we decided to add normal mapping, this technique in fact adds an extra layer of realism due to lighting being a very important factor in any engine.
  
  The technique in itself is just a direct interaction with **lighting and the normals of a given mesh** to emphasize on the depth of a surface so it does not look like a flat surface. In essence this technique basically converts the normals from the mesh into **per-fragment** (_per-pixel_) normals, this way we can have little variations on the normals of a mesh and trick the lighting system in the engine to think it has some kind of change in light in specific parts of the mesh otherwise if it was a per-surface it is going to be pretty much the same for each fragment in the screen and it is not going to have any variation, in essence we give the **illusion** that the mesh is way more complex than it appears without adding any extra geometry.
  
  ![Example Normal Mapping](https://learnopengl.com/img/advanced-lighting/normal_mapping_surfaces.png)
  
  Giving this micro-level details of normals per pixel, we in essence create an illusion where have for example a flat brick plane to look like it is indented where it needs to be like in the following image:
  
  ![Brick Wall Example](https://learnopengl.com/img/advanced-lighting/normal_mapping_compare.png)

  This way we avoid doing an **interpolation** between surface normals and instead of that we utilize a specific normal per pixel in the screen, the lighting does the rest of the job to create this effect.
  
  Now, to be able to introduce in the engine we need to follow the usual **art pipeline**, usually the normals of a geometry are stored in what we know as a **normal map**, this will hold all the per-fragment information for the normals for given geometry and will allow us to add that realism to the geometry.
  
  This means that we will be loading a normal texture in the engine through our **texture builder** and usually a normal map looks very blue-ish in color with reds and greens in between, indicating that the colors correspond to the axis, `Blue (0, 0, 1)`, `Red (1, 0, 0)` and lastly `Top (0, 1, 0)` this corresponds to what we see in the texture, greens appear in the top-most part of geometry whereas reds tend to appear in the left-right most parts and in general the faces of the geometry are blue because they are pointing to `+Z`.
  
  ![Brick Normal Map Example](https://learnopengl.com/img/advanced-lighting/normal_mapping_normal_map.png)
  
  We do not need to forgive that whenever we are processing the texture through the shader we need to normalize in the `[0, 1]` space instead of the space in which they come `[-1, 1]` to be able to utilize them correctly. _Remember that we can represent the information from a normal texture through the **x, y, z** coordinates instead of the usual **RGB** that we do whenever we want to get information from the **diffuse**_.

  This means that to be able to normalize the information from the texture in the given limits we do the following shader code:
  
    
```glsl
  vec3 transformed_normal = normal * 0.5 + 0.5; // ( This essentially transforms from [-1,1] to [0, 1] )
```
  This allows us to transform from a normal space coordinates to an RGB coordinate space, essentially allowings us to read it without having too much trouble with random normal artifacts. Now, to be able to utilize the normal map we need to sample it and use them for lighting calculations and this is done the following way:
  
  
 ```glsl
 uniform sampler2D normalMap;
 
 in vec2 uvs;
 
 void main()
 {
    normal = texture(normalMap, uvs).rgb
    normal = normalize(normal * 2.0 - 1.0) // We convert them back to range [-1, 1]
    
    // [......] Proceed with the lighting calculations (Pointlight, Directional, Spotlight)
 
 }

```

  This all will work perfectly if we have surface sof the geometry or a simple flat plane with normal mapping that is pointing to the `+Z` direction, in case we rotate it or make it point in any other direction, normal mapping gets ruined, because the **normals** will still be pointing to the `+Z` coordinates, think of it as normals being absolute towards the `+Z` direction no matter how much you rotate the geometry or mesh, this implies that the lighting is going to be done incorrectly and we have officially broken the technique.
  
  ![Z Positive Normals Issue](https://learnopengl.com/img/advanced-lighting/normal_mapping_ground_normals.png)
  
  Luckily we have a solution for this, to be able to keep the normals relative to the geometry / mesh we utilize what we call `tangent space`, this in essence brings the normals to a local space where they are relative to the individual triangles of the geometry, of course always pointing `+Z` but this time in local coordinates, implying that even if we rotate it, it is going to rotate in the **positive Z** coordinate taking into account the rotation of the mesh.
  
  To be able to transform from **tangent space** to **world space** and viceversa we need to be able to construct what we call a **TBN Matrix** to be able to utilize the inverse and the matrix itself to do the transformations of normals comfortably.
  
  As we know, the normals in **tangent space** point towards the direction of the surface, this leaves us knowing one piece of information, **the normal**, we are left out with more information that we need to discover, in this case the **bitangent** and the **tangent**. 
  
  ![TBN Coordinates](https://learnopengl.com/img/advanced-lighting/normal_mapping_tbn_vectors.png)
  
  The issue here is that we do not have any other piece of information other than the normal, it is not like we can magically do a **cross product** and get it done, this means that we will need to do some simple math and it is the following one:
  
  ![Normal Mapping TBN Math](https://learnopengl.com/img/advanced-lighting/normal_mapping_surface_edges.png)
  
  You first need to calculate the edges and delta UV coordinates so you can get the desired vectors, to ease out the mathematics we will do a manual approach so it feels more integral, if we had the following positions, normals and texture coordinates:
  
 ```cpp
  glm::vec3 pos1(-1.0f, 1.0f, 0.0f);
  glm::vec3 pos2(-1.0f, -1.0f, 0.0f);
  glm::vec3 pos3(1.0f, -1.0f, 0.0f);
  glm::vec3 pos4(1.0f, 1.0f, 0.0f);
  
  // tex coords
  glm::vec2 uv1(0.0f, 1.0f);
  glm::vec2 uv2(0.0f, 0.0f);
  glm::vec2 uv3(1.0f, 0.0f);
  glm::vec2 uv4(1.0f, 1.0f);
 
 // normal vector +Z
 glm::vec3 nm(0.0f, 0.0f, 1.0f);
 
 ```
 
  We need to calculate the delta UV coordinates which correspond to `ΔU2` and `ΔV2` which is doing a substraction to the positions:
  
 ```cpp
  glm::vec2 deltaUV1 = uv2 - uv1;
  glm::vec2 deltaUV2 = uv3 - uv1;
  
```
  
  You can see that `deltaUV1` corresponds to the horizontal `ΔU2` delta and the `deltaUV2` corresponds to the vertical `ΔV2` delta, afterwards the next calculations we will need to do is the edges to be able to calculate the triangle, in this case `E1` and `E2`.
  

 ```cpp
  glm::vec3 edge1 = pos2 - pos1;
  glm::vec3 edge2 = pos3 - pos1;
  
```
  
  Once again we can correlate the horizontal and vertical edges with `E1` (_edge1_) and `E2` (_edge2_) respectively, this way we can see that with the edge vectors we essentially have constructed a triangle, having the delta for UVs and the edges of the triangle we can start calculating the vectors we want which in this case are the **tangents** and **bitangents**.
  
```cpp
float f = 1.0f / (deltaUV1.x * deltaUV2.y - deltaUV2.x * deltaUV1.y);

tangent1.x = f * (deltaUV2.y * edge1.x - deltaUV1.y * edge2.x);
tangent1.y = f * (deltaUV2.y * edge1.y - deltaUV1.y * edge2.y);
tangent1.z = f * (deltaUV2.y * edge1.z - deltaUV1.y * edge2.z);

bitangent1.x = f * (-deltaUV2.x * edge1.x + deltaUV1.x * edge2.x);
bitangent1.y = f * (-deltaUV2.x * edge1.y + deltaUV1.x * edge2.y);
bitangent1.z = f * (-deltaUV2.x * edge1.z + deltaUV1.x * edge2.z);

```
  With this you would already have all the necessary vectors to calculate the **TBN matrix**.

  {: .box-note}
**Note:** You need to do the same process for the tangent and bitangent calculation for each triangle in your geometry, usually you would incorporate this piece of code in your **object loader** in your engine and store them in a **vector** so you can later **build** the **TBN matrix** and **pass it to the respective shader**.

  You have two options here, you can construct the TBN here in the **CPU** or you can do it in the respective **shader**, in our case we have it built in the shader itself and you would pass the information through vertex attributes to the vertex shader as following:
  
```glsl

layout (location = 0) in vec4 a_position;
layout (location = 1) in vec4 a_normal;
layout (location = 2) in vec4 a_uvs;
layout (location = 3) in vec4 a_tangent;
layout (location = 4) in vec4 a_bitangent;

void main()
{
  vec3 T = normalize(vec3(u_m_matrix * a_tangent));
  vec3 B = normalize(vec3(u_m_matrix * a_bitangent));
  vec3 N = normalize(vec3(u_m_matrix * a_normal));
  mat3 TBN = mat3(T, B, N);
}

```
  
  {: .box-note}
**Note:** We do not need to calculate all the three vectors, with only two of the vectors, one of them being the **normals** we can do a **cross product** `vec3 B = cross (N, T)`.

 Now after we have the TBN matrix we can take two approaches, either taking the sampled normal from tangent space to world space using the TBN matrix or the opposite, using the inverse of the TBN to transform any vector to tangent space, the rule that we need to abide by is that **the calculations need to be in the same coordinate space** to avoid any kind of artifacts or weird / unexpected behavior.
 
 In our specific case in Metro Engine we utilize the second option, we bring all the vectors to tangent space to do all the necessary calculations, this means that we first need to inverse the matrix before sending it to the fragment shader.
 
```glsl

in mat3 TBN_in;

void main()
{
  mat3 inv_tbn = transpose(TBN_in);
  
}

```
 
 Of course, we get the TBN from the vertex shader as input into the fragment shader to later transpose it to be able to transform any vector in the fragment shader to tangent space for the lighting calculations.
 
{: .box-note}
  **Note:** The transpose of an orthogonal matrix equals the inverse of it.
  
  And now we need to transform the necessary vectors `lighDir` and `viewDir` to tangent space.
  
```glsl

in mat3 TBN_in;
in vec2 in_uvs;

void main()
{
  mat3 inv_tbn = transpose(TBN_in);
  vec3 normal = texture(normalMap, in_uvs).rgb;
  normal = normalize(normal * 2.0 - 1.0);
  
  vec3 lightDir = inv_tbn * normalize(lightPos - fragPos);
  vec3 viewDir = inv_tbn * normalize(viewPos - fragPos);
  // ...
}

```

  And with this, we can calculate all the **lighting** variables in tangent space given that the normal is in tangent space too, it is a perfect fit for normal mapping to shine and show the technique fully working without caring about the rotation of the geometry/mesh.

![Rotated Plane with Normal Mapping](https://learnopengl.com/img/advanced-lighting/normal_mapping_correct_tangent.png)

  We extracted this technique from **[LearnOpenGL](https://learnopengl.com/Advanced-Lighting/Normal-Mapping)** and all the images and mathematics done here are extracted from this website.



