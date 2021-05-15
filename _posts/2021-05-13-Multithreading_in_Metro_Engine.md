---
layout: post
title: Multithreading in Metro Engine
subtitle: Px-Sched and how it is used
tags: [multithreading, tasks, synchronization, px_sched, pplux]
---

## Multithreading <Px-Scheduler>
  
  Before actually reading about **multithreading** and how it is handled in the engine, I recommend you to read [Metro Engine Structure](https://metro-engine.github.io/2021-05-13-Metro_Engine_Structure/)  blog post just to have more information on the why are we doing it this specific way and what possibilities we do get from doing it this way specifically.

  As this was one of the requirements that need to be met for the subject that we were studying in university, our engine had to be for sure multithreaded, this meant that we had to at least separate the render and the logic loops in different threads for them to be ran simultaneously and having a healthy and synchronized communication without any kind of _race conditions_ or _thread collisions_.
  
  
  
  
  
  
  
  
  
  
  
