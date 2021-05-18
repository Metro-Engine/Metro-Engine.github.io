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



  
