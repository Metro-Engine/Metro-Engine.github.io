---
layout: post
title: ECS in Metro Engine
subtitle: How does Metro Engine implement ECS?
tags: [ecs, entity, data-oriented]
---

## What is ECS?

  In game development we have heard about **Entity Component Systems** (_ECS_), there are numerous engines such as **Unreal Engine** and **Unity** that do utilize this system and it is a different ways to organize the data in your engine.
  
  Since a lot of time whenever we developed an engine we would follow an inheritance approach organization of data, **object oriented programming** is what takes us into this journey thanks to **inheritance** and **polymorphism**.
  
### The Two Problems (Flexibility & Misuse Of Cache)  

  The issue here is that there are two huge problems with this system, one of them being a problem with **flexibility** and the second one is the **not** so obvious **misuse of cache** which is not that apparent at first sight but really helps in performance in general. Image we had the following hierarchical structure for classes:
  
  ![Class Hierarchy Example #1](https://i.imgur.com/yUywn4D.png)

  As you can see we have a base class `Humanoid` from which two other classes, `Monster` and `Human` inherit, and from there we have another two that inherint from the previously mentioned classes and those are `Orc` and `Knight`.
  
  Now, you might think that this is a very stable structure for a game, right? It is done the usual way, through inheritance, now think about this:
  
- What if we want to have an Orc that wants to be a Knight?
- What if we want to have a Knight change his racial to Orc?

  This would mean that we have a pretty **static** structure that for us to be able to change we need to once again abstract it, re-organize it and lay down an even more solid base, we pretty much need to predict that such thing can happen.

  Another one of the problems is the **misuse of cache**, if we had a personal `Update()` method per each **object** and it updated their position, velocity, acceleration, etc, logically what we would do is iterate over **each and every single** object in our engine and call their `Update()` method to update the values of each individual object, take care that each individual object here is like an individual **container** that contains their own information.
  
  For us to be able to update every single one of the objects, we would need to be **constantly loading** all the different values separatedly, position, velocity, acceleration, etc. As you can see, per each new object we are not utilizing cache at all, usually the perfect workflow here is to pack up all the positions from all objects and process and update them all, in this sense we utilize cache to gain speed and performance and we avoid retrieving information from RAM >> CPU >> RAM constantly, we pack all the necessary information, bring it to CPU's Cache and reiteratively utilize it there without the need to be constantly loading and unloading the cache just because we do the data organization wrong.

### Solutions To The Problems

  To be able to fix the **flexibility** problem, the game industry has taken a shift towards a more **component-based** design, for example in Unity a game object is created entirely with components, you get a blank game object with a **TransformComponent** and you can add a **Rigidbody** Component to be able to make it work with the physics engine to add **gravity** for example, or you can add your own **script** which could act as a **ScriptCompoennt** that could make the object rotate constantly in a specific **axis**. This means that we are not bound to a specific structure anymore, we can have different game objects built with different components without the restrain of inheritance.
  
  For the second problem, is fixed by keeping all data that will be iterated packed up in the same place in memory just as I have mentioned above, by doing this an entire cache line of data can be loaded at once and whenever the next item is iterated, we will find it in the cache instead of doing the iterative loop of going back between the CPU and the RAM.
  
### ECS and Systems 

  In this new way of organization of data and processing information, we lose the concept of _object_ and we instead adopt a new concept that we call "**Entity**", which is just an ID, nothing else. This ID is going to be utilized to index in an array of components, think of it as a database where we have an **unique identifier** for each entity and we establish **relationships** between **component and entity ID**.
  
  As we know, an array is contiguous in memory, which in general helps on the way this data structurization works, it is the data structure of choice to store all the information about ECS in general because we seek **continuity in memory**, we do not want to jump between places to search for information, that breaks the cache.
  
  
