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
  
  To be able to start working with **Lua** we first need to declare our `lua_State` variable, this variable defines the context and scripts with which we run **Lua** in our engine and in an actual `.h` file we need to deploy the necessary **lua header files**.
  
```cpp
// # in a .h where you will be declaring lua c api code:
extern "C" {
  #include "lua.h"
  #include "lauxlib.h"
  #include "lualib.h"
}

 // [ ... ]

```
  
```cpp
  // # Creation of a lua state:
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

  Now that we konw how to actually open a **Lua VM** state, we need to learn how to communicate between the script and C. For example if we want to do a simple multiplication function we would do the following with C:
  
```cpp

int multiplication(int a, int b) {
  return a * b;
}

```
  Pretty simple right? Now, how would we establish a connection between lua and this code so we can use this function internally in a **lua script**? Well, it is pretty easy, as I have mentioned previously, a stack is the middle point of communication between lua and C, so we will be utilizing it.
  
  Every function that is going to be utilized in Lua needs to have only **one arguement** and it has to be the **lua_State** which in essence is the **context** of **lua**. So we proceed with the following code to adapt the function to lua:
  
```cpp
  
int multiplication(lua_State *L) {
  int a = luaL_checkinteger(L, 1); // Notice how we pass the context and as a second arguement we check the specific position in the stack, in this case position 1.
  int b = luaL_checkinteger(L, 2); // Likewise for this, we instead of checking the 1st, check the 2nd.

  // # We store the result in a special typedef value for lua integers:
  lua_Integer c = a * b; 
  
  // # We push the variable to the stack.
  lua_pushinteger(L, c);

  // # Exit Code Successfully, otherwise we error.
  return 1;
}


```
  
  To clarify, if we called the following function `multiplication(5, 5)` the following would happen in the lua stack:
  
![Multiplication Stack Example](https://i.imgur.com/6mL9y4k.png)

  Remember that this example is executed from **Lua**, implying that the exemplified function from lua pushes the numbers into the stack so in C they can be `checked` and returned back to the stack, essentially doing a `LUA >> C >> LUA` communication.
  
  We can see that the indexing of the stack is pretty easy, both numbers are deposited at `position 1` and `position 2` and we utilize the `luaL_checkinteger` function to be able to check if those numbers are integers so we can later capture them in variables and utilize them to do the operations in **C**, afterwards we push the results in the stack, ideally the variable above would get returned as a `luaL_integer`.
  
  Afterwards we compute the result and push the given **integer** to the stack in the next position available because we have **not** cleaned up on ourselves, so the other two numbers, `(a, b)` still stay in their respective positions, so this means that the result will be pushed to `position 3` resulting in the following stack:
  
  ![Resultant Stack for Multiplication Operation](https://i.imgur.com/FTG41ao.png)
  
  So this way we have essentially got an efficient communication from Lua to C and back to Lua, the number `25` in this case would be available in the lua script to be utilized for whatever operation it is needed for.
  
  Now you might be wondering, how do we utilize this function in lua, as far as we know there is no custom `multiplication` function... We need to **register** it to the given lua context so it is available for use in the lua script. This means that we will be utilizing the function `lua_register(luaL_state*, const char*, lua_CFunction f)` function, from the signature we can figure out that it is pretty easy to utilize, in the case of the multiplication we can do the following registration:
  
```cpp
int multiplication(lua_State *L) {
  int a = luaL_checkinteger(L, 1);
  int b = luaL_checkinteger(L, 2);

  
  lua_Integer c = a * b; 
  
  
  lua_pushinteger(L, c);

  
  return 1;
}

  // #Register the function
  lua_register(L, "multiplication", multiplication);

```
  After we have the function registered to the current context, we will be able to utilize it inside a script as shown in the previous example, we can do a `multiplication(5, 5)` in lua and get a resultant `25` as the return value of the given function. Now we can see that something is **lacking**, our stack is a complete **disaster**, so far in the previous example we have learnt how to **push** numbers to the stack, but we need to learn to **clean up** after ourselves and learn how to **pop** irrelevant or unnecessary information so whenever we work with the stack once again the actual stack is clean and ready to go, it is a good **habit** to clean up the stack every time you utilize it for an operation.
  
  Given the **previous stack** that we did **not** clean up, we can do the following operations to clean it up:
  
  ![Resultant Stack for Multiplication Operation Example #2 Pop](https://i.imgur.com/FTG41ao.png)
  
```cpp

  lua_pop(L, 1); // We extract number 5.
  lua_pop(L, 1); // We extract number 5.
  
```
  As you can see, we only need to leave out the result of the actual operation in the stack so **lua** can utilize it in the script! This means that the following multiplication code would be the following:
  
  ```cpp
int multiplication(lua_State *L) {
  int a = luaL_checkinteger(L, 1);
  int b = luaL_checkinteger(L, 2);

  
  lua_Integer c = a * b; 
  
  lua_pop(L, 1);
  lua_pop(L, 1);
  
  // # The resultant number c would be deposited in position 1 of the stack:
  lua_pushinteger(L, c);

  return 1;
}

  lua_register(L, "multiplication", multiplication);

```

  So far so good, this is a rather more manual approach to cleaning the stack, we can always utilize the `lua_gettop(L)` function to get the highest position in the stack, in this case we would always capture `position 1`, because remember whenever you extract something from the **stack**, the 2nd becomes the 1st and so forth, indexes do change.
  
  We can also **negatively** index the stack and work the **inverse** way, so for example:
  
```cpp

  lua_pushinteger(L, 40);
  lua_pushinteger(L, 50);
  lua_pushinteger(L, 60);
  
  // # Would look like:
    // 40; -3
    // 50; -2
    // 60; -1

  // And if we popped number 2 from the stack, it should remove number 50:
  
  lua_pop(L, -2);
  
  // # Would look like:
    // 40; -2
    // 60; -1
  
  // In the normal order of the stack the above stack would look like this:
  
    // 40; 1
    // 60; 2
  

```
  You can get the idea of how to push and pop in the **normal order** and the **inverse order**, whichever suits you the most is the one you should use, in terms of functionality it is pretty much the **same**.
  
  Now the last thing left out for the finish up with the basic functionality to communicate between **lua** and **C** is the calling of a **lua function** in C with arguements and a return value. If you had the following lua script with the given function:
  
```lua
-- script0.lua
function my_function(a, b)
  return a * b;
end
```
  It is a pretty simple function that once again, does the same functionality as the `multiplication(a, b)` function we had in **C** but this time we have it declared in **Lua** and we want to use it in our **C** code in the engine.
  
  For this to be possible aside from using the stack to communicate we will be introducing a new function called `lua_pcall(lua_State *L, int nargs, int nresults, int errfunc)`, as you can see this function is pretty easy to utilize, we pass as first arguement the given **lua context**, the number of arguements that the function has, the expected returns (_in lua you can do a return with multiple values_) and an **error function** in case the operation is not a **success**.
  
  To remark we need to consider that every function declared plainly in a **lua script** is in the **global scope** of the script, this means that we can utilize a function called `lua_getglobal(lua_State *L, const char *functionName)` that will literally retrieve the pointer to that function for you to utilize it and call it from **C**.
  
  To be able to run the given function in C we do the following piece of code:
  
```cpp
int main(int argc, char ** argv) {
  
  lua_State *L = luaL_newstate();
  luaL_openlibs(L);
  
  // # luaL_dofile (lua_State *L, const char *filename); --> Loads and runs the given file, if it returns 0 there are no errors otherwise 1 is returned.
  if (luaL_dofile(L, "script0.lua") == LUA_OK) {
    lua_pop(L, lua_gettop(L)); // You pop the value 0 or 1 for successful or unsuccessful execution of the given file depending on if it was found or not.
  }
  
  // # We retrieve the function "my_function"
  lua_getglobal(L, "my_function"); // # This function inserts the function ptr in the stack.
  lua_pushinteger(L, 2); // # First Arguement
  lua_pushinteger(L, 5); // # Second Arguement
  
  // # As we have previously mentioned, we will be using lua_pcall to execute the function safely in the C environment:
  if (lua_pcall(L, 2, 1, 0) == LUA_OK) {
      
      // # Check if the return is an integer:
      if (lua_isinteger(L, -1)) {
      
        // # If it is an integer, convert the return value from lua typedef to integer:
        int result = lua_tointeger(L, -1);
        
        // # Pop the return value [Clean the stack]
        lua_pop(L, 1);
        printf("Result: %d\n", result);
      
      }
      // # Remove function from stack:
      lua_pop(L, lua_gettop(L));
      
  }
  lua_close(L);
  
}

```

  In essence this would be the process that you would follow to be able to **call** a **lua function** from **C** without having too much trouble, in this previous example you learnt how to retrieve global variables and functions and utilize them at your will, executing a lua file, checking if the returned value is an integer, converting from lua typedefs to C typedefs and using the `lua_pcall` function to execute the function with extra safe measures.
  
### LUA API in Metro Engine
  
  Now you can guess that **most** of our **LUA API** is built around this basic concepts explained above, this means that our API to communicate with our graphic backend and all the nifty features that our engine has are built on this foundation, of course we have not explained anything about the **strongest advantage** of lua which in this case is **tables** but I guess you can do a bit of self research [here](https://www.lua.org/pil/24.html) to learn all the necessary to be able to integrate more complex behaviors, with only what has been shown here is enough to build a rather simplistic but **useful** API that any user can use in **LUA** and in **C**.
  
```cpp
// # Metro Engine User API:

// # Geometry Creation Functions:

int CreateCube(lua_State* L);
int CreateSphere(lua_State* L);
int CreateMonkey(lua_State* L);

// # Debugging Tool (Utilized for printing in the logger of our engine):

int printStuff(lua_State* L);

// # Entity Modification Functions:

int SetPosition(lua_State* L);
int SetRotation(lua_State* L);
int SetScale(lua_State* L);
int SetTransform(lua_State* L);


// # PBR Values Functions:

int SetRoughness(lua_State* L);
int SetMetallic(lua_State* L);

// # Rotation Component Modification Functions:

int SetRotationSpeed(lua_State* L);
int SetRotationAxis(lua_State* L);

// # Entity Hierarchical Parenting Function (Change parent of Entity):

int SetParent(lua_State* L);

// # Creation & Setting of Texture to given entity:

int SetTexture(lua_State* L);
int CreateTexture(lua_State* L);

// # Set Color of given entity:

int SetColor(lua_State* L);

// # Clear of all entities in scene:

int ClearScene(lua_State* L);

```
  
  
