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

We proceed to add a component with value `A` to Entity with Index `0` in the arrray:

![Component Array Example #2](https://user-images.githubusercontent.com/48097484/119245065-8497ad80-bb76-11eb-90c9-65967bac1596.png)

Now Imagine we have the array almost full with the same, we add components `B`, `C` to their respective entity with indexes `1` and `2`:

![Component Array Example #3](https://user-images.githubusercontent.com/48097484/119245110-df310980-bb76-11eb-8452-edafd8815b81.png)

Now imagine we delete entity with index `1` in this array, this means that the one that should **theoretically** occupy the spot is the last entity in the array which in this case is the one currently with index `2`.  And the resultant table would look as following:

![Component Array Example Destroy](https://user-images.githubusercontent.com/48097484/119245390-5798ca00-bb79-11eb-9847-6f2a927e5a3e.png)

That is how we would keep reference to the specific indexes so we never end up with holes, we interconnect everything and whenever we want to consult components we keep the **entity -> index** and **index -> entity** so we have immediate conversion between each other.

```cpp
namespace Metro {

	class IComponentArray
	{
	public:
		virtual ~IComponentArray() = default;
		virtual void EntityDestroyed(u32 entity) = 0;
	};

	template<typename T>
	class ComponentArray : public IComponentArray
	{
	public:

		ComponentArray() {
			componentArray_.Alloc(kMaxEntities);
			size_ = 0;
		}

		void InsertData(u32 entity, T component)
		{
			// Put new entry at end
			size_t newIndex = size_;
			entityToIndexMap_[entity] = newIndex;
			indexToEntityMap_[newIndex] = entity;
			componentArray_[newIndex] = component;
			++size_;
		}

		void RemoveData(u32 entity)
		{
			// Copy element at end into deleted element's place to maintain density
			size_t indexOfRemovedEntity = entityToIndexMap_[entity];
			size_t indexOfLastElement = size_ - 1;
			componentArray_[indexOfRemovedEntity] = componentArray_[indexOfLastElement];

			// Quick reorder of the array, changing the element we want to remove with the last one,
			// and deleting last one.
			u32 entityOfLastElement = indexToEntityMap_[indexOfLastElement];
			entityToIndexMap_[entityOfLastElement] = indexOfRemovedEntity;
			indexToEntityMap_[indexOfRemovedEntity] = entityOfLastElement;

			entityToIndexMap_.erase(entity);
			indexToEntityMap_.erase(indexOfLastElement);

			--size_;
		}

		T& GetData(u32 entity)
		{
			if (entityToIndexMap_.find(entity) == entityToIndexMap_.end())
			{
				printf("[ERROR] : Tried to get an unexisting component from entity %d\n", entity);
				return T();
				//assert(entityToIndexMap_.find(entity) != entityToIndexMap_.end());
			}
			return componentArray_[entityToIndexMap_[entity]];
		}

		void EntityDestroyed(u32 entity) override
		{
			if (entityToIndexMap_.find(entity) != entityToIndexMap_.end())
			{
				RemoveData(entity);
			}
		}

		bool Exist(u32 entity) {
			return entityToIndexMap_.find(entity) != entityToIndexMap_.end();
		}

	private:
		ScopedArray<T, u32> componentArray_;
		std::unordered_map<u32, size_t> entityToIndexMap_;
		std::unordered_map<size_t, u32> indexToEntityMap_;
		size_t size_;
	};


}

```

The IComponentArray is a virtual inheritance that is required, it pretty much our bridge of commuincation between all the component arrays (_one per component type_), this will basically serve to update all the other component arrays with the correct information once an entity is deleted.

We need to consider that we use an `std::unordered_map<u32, size_t>` and that has its penalty because as the name implies, it is an unordered data structure that does not have a set **continuity** hence why the better option here would be **arrays** to still keep that continuity but as of now, we tend to be more flexible towards the unordered_map because it has utility functions such as `find()`, `insert()` and `delete()` which allow for a more comfortable and flexible use of the data structure compared to an array.

### The Component Manager

This is going to be the manager that is going to be communicating with all the `ComponentArrays` in the ECS and will be in charge of capturing when a component needs to be added or removed, this means that this manager needs to know the **unique ID** of every type of component so that it can have a bit in a signature.

Similarly to the **entity manager** we will have a component type variable that will increment by one for each component type registered.

```cpp
namespace Metro {
	class ComponentManager
	{
	public:

		ComponentManager() {
			nextComponentType_ = 0;
		}

		template<typename T>
		void RegisterComponent(bool isUserAccesibleInUI)
		{
			const char* typeName = typeid(T).name();
			if (isUserAccesibleInUI)
			{
				std::string thisName = std::string(typeName);
				thisName.erase(0, 13);
				uiAccessibleComponentNames_.push_back(thisName);
			}
			componentTypes_.insert({ typeName, nextComponentType_ });
			componentArrays_.insert({ typeName, std::make_shared<ComponentArray<T>>() });
			++nextComponentType_;
		}

		template<typename T>
		u8 GetComponentType()
		{
			const char* typeName = typeid(T).name();
			return componentTypes_[typeName];
		}

		template<typename T>
		void AddComponent(u32 entity, T component)
		{
			GetComponentArray<T>()->InsertData(entity, component);
		}

		template<typename T>
		void RemoveComponent(u32 entity)
		{
			GetComponentArray<T>()->RemoveData(entity);
		}

		template<typename T>
		T& GetComponent(u32 entity)
		{
			return GetComponentArray<T>()->GetData(entity);
		}

		template<typename T>
		bool HasComponent(u32 entity)
		{
			auto componentArray = GetComponentArray<T>();
			return componentArray->Exist(entity);
		}

		void EntityDestroyed(u32 entity)
		{
			for (auto const& pair : componentArrays_)
			{
				auto const& component = pair.second;

				component->EntityDestroyed(entity);
			}
		}

		u8 NumOfExistingComponents() {
			return nextComponentType_;
		}

		std::vector<std::string> uiAccessibleComponentNames_;

	private:
		std::unordered_map<const char*, u8> componentTypes_;
		std::unordered_map<const char*, std::shared_ptr<IComponentArray>> componentArrays_;
		u8 nextComponentType_;

		template<typename T>
		std::shared_ptr<ComponentArray<T>> GetComponentArray()
		{
			const char* typeName = typeid(T).name();
			return std::static_pointer_cast<ComponentArray<T>>(componentArrays_[typeName]);
		}
	};
}
```

Aside from that we have a `std::vector<std::string>` with the UI accessible components that is of use for us in our engine for the ImGui interface in the engine. Thanks to C++ and templarization we can modularize all the functions for creation and deletion of components so we can use the same code for different types of components altogether.

### The System (How it Works)












