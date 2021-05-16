---
layout: post
title: ImGui in Metro Engine
subtitle: How does ImGui work in the engine?
tags: [structure, imgui, ocornut, ui/ux, interface, gui]
---

## Introduction

  The interface of the engine is a nice addition that any engine needs to have, currently in Metro Engine we are using **ImGui** in the PC version, we are planning to upgrade the engine with more interface as time goes because it is an overall nice way to interact directly with the engine without touching the actual inners of it.

  The interface of the engine is pretty simple, we went for the **docker** system that ImGui provides and tried to build it similarly to another engine that we had as reference that you most probably know about called **Unity**, so you can expect a lot of ressemblances.

  ![Metro Engine ImGui](https://i.imgur.com/fdzzfZi.png)
  
  You can see that the structure as previously mentioned is pretty straight forward and similar to Unity, you have in the left-most part the hierarchy of your scene, in the rightmost part you can find the properties editor for each entity and lastly in the center bottom part you can find the logger.
  
  
### Hierarchy

![Hierarchy Window](https://i.imgur.com/SSDh8ZX.png)

  In this window you can see all the entities that are currently in the scene and you can _left click_ them to **select** them to be able to edit their properties in other window.
  
  Aside from that you can select an entity, **hold left click** and drag it to another entity to be able to parent it to be able to create a parent-child relationship between entities.

  Lastly you have a few buttons such as the "**Add Entity**", "**Clear Scene**" which are pretty self explanatory. You also have a  selection box to choose which skybox you want to have currently drawn in the scene and you can debug the **irradiance skybox** for the IBL graphic effect to see if it is being done correctly.

### Inspector

![Inspector Window](https://i.imgur.com/259FVM7.png)

  The next ImGui Window we have in the engine is the inspector window, this window is pretty special because depending on the entity and which components it has it will show different information. It is a given that every single entity component will at least have a **TRANSFORM COMPONENT** , so whenever you create a new entity, expect it to have a transform component with its respective **translation**, **rotation** and **scale**.

  Another small component that each one of the entities will currently have is the "**Add Component**" selection box alongside the **Add** and **Destroy** buttons, obviously this is done so the user can add and remove whatever component from the given entity without touching any kind of code at all.
 
  This means that every entity will have different/same components from the following ones:

  - Mesh Component.
  - Material Component.
  - Rotation Component.
  - Light Component.
  - Script Component.

{: .box-error}
**Error:** The engine will give you an error message and will not let you add a component that the entity already has, it is useless to try and do it, in case you try to add a compoennt and you see that nothing happens in the inspector window, check if the entity **already has that component**.

