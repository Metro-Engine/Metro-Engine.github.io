---
layout: post
title: Scripting in Metro Engine
subtitle: How does the LUA API work in the engine?
tags: [lua, api, scripting, engine]
---

## Lua in Metro Engine

  Our scripting language of choice in this case is Lua, it is a pure C scripting language that can be embedded pretty much in any piece of software without too much trouble, making it ideal for our engine. We also have considered that the actual scripting language is pretty easy to learn to start doing pretty much anything as it is not a **hard** typed language and that allows for a better comprehension for a user that is not that **technical** with programming in general.
  
  To be able to **understand** how **Lua** communicates with **C** and viceversa, you first need to understand the concept of a **stack**, it is a frequently used **data structure** in programming.
  
### Stack

  A Stack is a data structure that is very useful whenever you want to do inserts and extracts to the latter part of the structure. It is specialized in those operations because it works as a conventional computer stack as it works in LIFO mode (LAST IN FIRST OUT)

This container is a wrapper of a Vector data structure, the stack is a vector in disguise essentially with a few functionalities removed, you are only allowed to do insertLast and extract Last, those functionalities are renamed to “push” and “pop” to accomplish the functionalities of the stack. 

It is efficient whenever doing a lot of extracts and inserts to the latter part of the stack. It has all the advantages of the vector (continuous memory) and it removes a huge disadvantage of the vector, inserting and extracting from the head of the vector. But it is pretty limited on the inserting and extracting.

![Stack Example Image](https://i.imgur.com/IETs4na.png)

  To be able to **extract** from a **LIFO** (_LAST IN FIRST OUT_) we start from the top node it is coloquially called **pop** whenever you see it, you will easily identify when information is being extracted from the **stack**, here is an example: 

![Stack Pop Example Image](https://i.imgur.com/FcHqIY3.png)

  And to be able to **insert** into a **LIFO** stack, we insert the **new data** to the top part where we previosly pop from, in this case we call this action **push** and it is shown in the following example:
  
 ![Stack Push Example Image](https://i.imgur.com/gOoJVjP.png)
  
  Now that we know how this data structure works, we are pretty much ready to understand how **LUA** communicates with **C** and how we can connect that with our **engine** to be able to do fancy features.
  
### Lua 
  
  
