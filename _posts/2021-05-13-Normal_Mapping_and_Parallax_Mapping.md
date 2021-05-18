---
layout: post
title: Normal Mapping & Parallax Mapping
tags: [normals, normalmapping, parallax, parallaxmapping, technique, opengl]
---

## Normal Mapping

  To be able to add some extra detail to quads or meshes in our scene, in this engine we decided to add normal mapping, this technique in fact adds an extra layer of realism due to lighting being a very important factor in any engine.
  
  The technique in itself is just a direct interaction with lighting and the normals of a given mesh to emphasize on the depth of a surface so it does not look like a flat surface. In essence this technique basically converts the normals from the mesh into per-fragment (per-pixel) normals, this way we can have little variations on the normals of a mesh and trick the lighting system in the engine to think it has some kind of change in light in specific parts of the mesh otherwise if it was a per-surface it is going to be pretty much the same for each fragment in the screen and it is not going to have any variation, in essenc ewe give the illusion that the mesh is way more complex than it appears without adding any extra geometry.
  
  ![Example Normal Mapping](https://learnopengl.com/img/advanced-lighting/normal_mapping_surfaces.png)
  
  
