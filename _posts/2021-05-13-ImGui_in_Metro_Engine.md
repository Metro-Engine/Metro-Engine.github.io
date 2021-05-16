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

  - **Mesh Component**: You can add this component to an entity to define which kind of mesh it will be using like for example a **quad**, **cube**, **sphere**, etc.
  - **Material Component**: Depending if the engine is ran in **normal rendering** or **PBR** you will be able to change specific settings of the material for each entity.
  - **Rotation Component**: If you ever want to make an entity rotate in a specific axis with a given speed, this component is ideal for that purpose.
  - **Light Component**: A component utilized to determine if an entity is going to be a light emitter like for example a **pointlight**, **spotlight**, **directional light**.
  - **Script Component**: Like in Unity, an entity can have a script that will be executed, this is present in the shape of a component.

{: .box-error}
The engine will give you an error message and will not let you add a component that the entity already has, it is useless to try and do it, in case you try to add a compoennt and you see that nothing happens in the inspector window, check if the entity **already has that component**.

  

### Lua Tab

![Lua Window Tab](https://i.imgur.com/5lBaewI.png)

  I completely forgot to show the other tab that was next to the hierarchy tab due to the fps counter blocking the view of it, this is the lua tab of the engine, here you can see that you can either write code inside the text box in runtime and see the results, you have the logger in the center bottom part of the engine to support your actual scripting with prints to see that everything is working as expected.
  
  Aside from having the runtime option you can also create a file and run it through the engine as long as you deposit it in the `lua_scripts` folder of the engine and then insert the name of the script such as `HelloWorld` if the file was called `HelloWorld.lua`.

### Logger Window

![Logger Window](https://i.imgur.com/19qwL8C.png)

  Lastly and to finalize with the introduction of all the interface that is shown in the engine we can see that we have the logger, in this window you can see that you can receive information from the status of the engine in the shape of color coded messages, yellow for warnings, grey for plain command messages such as setting the position of an entity and lastly red for errors that require the attention of the user.
  
  This logger is rather _useful_ whenever you do want to script in the engine or want to execute a lua file, it is a perfect way to print and debug through the logger, it is after all a necessity for the developer to see which values are being printed whenever coding in the scripting language in the engine.

  In this case we have the **Clear** , **Copy** and **Add Debug Text** buttons which each one of them are useful in their own way, you can clear the whole logger, copy the whole logger output to your clipboard and test the logger by adding a debug text in case you are wondering if it is still working properly. We do not need to forgive about the **auto-scroll** checkbox which is a commodity feature for the developer, so every time a new message gets added to the logger, it gets auto scrolled to the bottom.
  
  You need to take care that the logger has a limit at **50 messages**, after that each message starting from above gets deleted and is lost forever.
  
  
### Last Words

  Once again, thanks to [Ocornut](https://github.com/ocornut) for having such a great **immediate mode** graphic library so it is easier for us developers to build the interface of our engine or software in just a snap of a finger without having too much trouble!
  
  

