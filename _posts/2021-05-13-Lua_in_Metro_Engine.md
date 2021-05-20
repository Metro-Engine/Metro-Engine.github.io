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
  
  With that said, we now need to clarify that all the communication done with **Lua** and **C** is done through a `virtual stack`, essentially what you do is push data from C for global variable definitions, tables, functions and function arguments, this causes those variables to be available in the specific lua script, this means that whenever we want to call a C function, we need to recover the arguments and push the result back again to **Lua**.
  
  To be able to start working with **Lua** we first need to declare our `lua_State` variable, this variable defines the context and scripts with which we run **Lua** in our engine.
  
```cpp

  lua_State *L = luaL_newstate();

```
  After creating this variable, we also do not need to forgive that we can open **axuiliary** libraries that come with **Lua** as a support for the scripting experience such as the **math library** and the **string library**.
  
```cpp

  lua_State *L = luaL_newstate();
  lua_openlibs(L); // Open all libraries
  
  // Optionally if we decided to open specific libraries we could utilize this functions instead of lua_openlibs(L);
  luaopen_math(L);
  luaopen_string(L);

```

  At the end of the execution of the engine we do not need to forgive to close the **Lua Virtual Machine**(_Lua VM_) and it is done with the following function:
  
  ```cpp
    lua_close(L);
  ```


