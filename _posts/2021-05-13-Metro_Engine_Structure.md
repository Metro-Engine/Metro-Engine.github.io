---
layout: post
title: MetroEngine General Structure
subtitle: Structure of the Engine
tags: [structure, resourcemanagement, tinyobj, stblibraries]
---


## The Inner Structure

  The inner structure of this engine is pretty interesting, considering that this engine is based off **multithreading** this means that the organization of how the code is ran is different from coding it in a single core.

  One of the first things that we were taught this year was multithreading and how we had to have at least **two** different threads running in the engine, one for the ```Update()``` of the engine, implying that all the logic of the engine was ran there and likewise another loop which we can call ```Draw()``` that is where all the rendering of the engine was happening.

  So we have two distinct threads in which our engine has its base layered on:
  
    - Update() Loop.
    - Draw() Loop.

 {: .box-note}
**Note:** The update loop is not ran at a fixed delta time so it is not suited for physics calculations, the delta time of the frame is not constant so the physics calculations can be often erratic.

  After having the base logic on the loops separated we had to be able to separate the requests that the user could do in the engine from the actual rendering, this situation can be compared to what OpenGL does, you request to draw a triangle for example, and the actual API subscribes you or adds you to a list, it does not immediately listen to the request of the user, it stores a backlog / queue and whenever it has time to fit it in the pipeline, then it is when it draws it. 

  This serves as a way to separate the actual graphic backend from the actual general logic of the engine, this means that whenever the user codes a new functionality in the engine, the changes that the user does are not immediate, they are queued and processed whenever the engine has time.

  ## Producer - Consumer Pattern

  


