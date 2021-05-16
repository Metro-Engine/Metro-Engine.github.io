---
layout: post
title: ImGui in Metro Engine
subtitle: How does ImGui work in the engine?
tags: [structure, imgui, ocornut, ui/ux, interface, gui]
---

  The interface of the engine is a nice addition that any engine needs to have, currently in Metro Engine we are using **ImGui** in the PC version, we are planning to upgrade the engine with more interface as time goes because it is an overall nice way to interact directly with the engine without touching the actual inners of it.

  The interface of the engine is pretty simple, we went for the **docker** system that ImGui provides and tried to build it similarly to another engine that we had as reference that you most probably know about called **Unity**, so you can expect a lot of ressemblances.

  ![Metro Engine ImGui](https://i.imgur.com/fdzzfZi.png)
  
  You can see that the structure as previously mentioned is pretty straight forward and similar to Unity, you have in the left-most part the hierarchy of your scene, in the rightmost part you can find the properties editor for each entity and lastly in the center bottom part you can find the logger.
  
  
