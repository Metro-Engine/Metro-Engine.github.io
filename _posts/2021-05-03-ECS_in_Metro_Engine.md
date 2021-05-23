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
  
  This means that we will be extracing the information from the component array and utilizing it, but where do we utilize it? In something that we call a **System**, in ECS you can create different **systems** that treat different **combinations** of **components**. This means that for example we have a **physics system** that only runs on entities that have a **transform**, **rigidbody** component.
  
  Thanks to the indexing we are able to do through the entity because we have the ID of the entity at our disposal, we can obtain all the components associated to that entity, the system itself will process through all the information in the components and update the information on them.
  
### The Signature
  
  This is going to be the "**fingerprint**" of the entity to know which components the entity "has" as well as knowing which components a system takes care about.

  In our engine we specifically have a signature up to 32 bits, that means that we can have a maximum of **32** components per entity and systems that can act upon entities with up to **32** components.
  
```cpp
  #include <bitset>
  using Signature = std::bitset<32>;
```
  This in its essence is what we know as a **bitfield**, but instead renamed to **Signature**. Each and every one of the components is going to also have a unique identifier starting from `0` which is used to be a unique representation in the shape of a bit in the signature.
  
  A clear example could be the `Transform` as type 0, `RotationComponent` as type 1, the bitfield for an entity that "has" those two components would be `00b11`. In the same fashion, a system would register the components in which it will act upon, essentially what it does is a simple bitwise comparison to ensure that the entity's signature contains the system's signature, an entity might have **more** components than a system requires and that is **completely** fine, as long as it has all the necessary components it will run with the system itself.
  
### Entity Manager

  The entity manager is the one taht is going to be in charge of distributing all the entities in the engine, as we know an entity is just an **ID**, a number, so it is going to keep a record of which IDs are being used and which ones are not used, how does it do that? 
  
  For this to work you need to declare a `std::queue<u32>` and you need to fill it with the `MAX_ENTITIES` quantity like the following:
  
```cpp

namespace Metro {
	class EntityManager
	{
	public:
		EntityManager()
		{
			signatures_.Alloc(kSceneObjectsPoolSize);
			activeEntityCount_ = 0;
			for (u32 entity = 0; entity < kSceneObjectsPoolSize; ++entity)
			{
				availableEntities_.push(entity);
			}
			sceneMaxEnityDepthLevel_ = 0;
		}

		u32 CreateEntity()
		{
			u32 id = availableEntities_.front();
			availableEntities_.pop();
			++activeEntityCount_;

			return id;
		}

		void DestroyEntity(u32 entity)
		{
			signatures_[entity].reset();
			availableEntities_.push(entity);
			--activeEntityCount_;
		}

		void SetSignature(u32 entity, Signature signature)
		{
			signatures_[entity] = signature;
		}

		Signature GetSignature(u32 entity)
		{
			return signatures_[entity];
		}

		u32 sceneMaxEnityDepthLevel_;

	private:
		std::queue<u32> availableEntities_;
		ScopedArray<Signature, u32>	signatures_;
		uint32_t activeEntityCount_;
	};
}

```

  We can observe that the `EntityManager` class, in the **constructor**, does a for loop from 0 to the `MAX_ENTITIES` quantity and fills it with indexes so it is ready to be used. After filling the queue with the given indexes we need to be able to create entities:
  
```cpp
		u32 CreateEntity()
		{
			u32 id = availableEntities_.front();
			availableEntities_.pop();
			++activeEntityCount_;

			return id;
		}
```

 As you can see, we capture the front of the queue, pop the front of the queue and afterwards increment the counter on **current active entities** count, and lastly we return the value so whenever we create an entity we can utilize it in other functions in our engine.
 
In addition to this we have the `DestroyEntity(u32 entity)` function that is going to pretty much receive as an arguement the entity that is going to be added to the availableEntities queue once again and cleaned from all the other necessary arrays and managers.

```cpp
		void DestroyEntity(u32 entity)
		{
			signatures_[entity].reset();
			availableEntities_.push(entity);
			--activeEntityCount_;
		}

```
We can observe that here we reset the signature of the given entity, pretty much leaving the bitfield all to **zeros** and then pushing the **ID** of the entity back to the queue and we substract one from the **current active entities** count.

All the other functions in the **entity manager** are pretty self explanatory and do not require a major explanation, simple setter/getter for the signatures.

### The Component Array

We need a specific data structure that is **continuous** and keeps all the data **packed** without having any kind of **holes**, oh well, arrays are the best data structure to utilize in this scenario, the issue comes whenever we want to destroy an entity, we might leave a hole and that defeats the whole purpose of ECS.

Remember that the entire purpose of ECS is to keep data packe din memory so whenever a system iterates over all the specific components, it does **not** find any kind of holes and everything goes smoothly and as expected.

To solve this problem whenever you destroy an entity to **not** leave a hole is by pretty much take the last valid element in the array and move it to the destroyed entity spot and update the map so the entity ID points to the correct position, here I will show a visual example on how this works:

![Component Array Example #1](https://user-images.githubusercontent.com/48097484/119245000-17841800-bb76-11eb-8a21-2df407f6ad50.png)

