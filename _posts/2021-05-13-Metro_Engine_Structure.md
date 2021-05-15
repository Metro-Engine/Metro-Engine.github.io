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

## Producer Consumer Pattern

  This is a very interesting pattern that was brought to us whenever multithreading and how we should treat the actual handling of all the rendering and logic of the engine in its entirety. Basically the pattern goes as following, you have in the middle an _intermediate_ **buffer** that is going to be used as the core communication between the left and right sides of this pattern, the **producer** and the **consumer**.

  The Producer in this case is the one that will be qualified as the supplier or the one that is going to be filling up the actual buffer constantly whereas the consumer is the one that will be taking those "items" that are being dropped into the _intermediate buffer_ and using them either for logic or rendering.

  ![Producer Consumer Pattern](http://1.bp.blogspot.com/-ve5pbciTlBQ/UR1fzTt_BoI/AAAAAAAAAs0/jk6P3ce1fpE/s1600/Screen+Shot+2013-02-14+at+22.05.37.png)

  The concept is pretty similar to what a restaurant would work like, think of it as the following scenario, imagine that you are in a restaurant with your friends, you all see a waiter approach and take your orders for food, each one being different in size. The food that you guys order will be deposited on a big table in the middle of the restaurant and you guys will be responsible for taking those items from the table to eat them.

  In this case from what you could guess, the consumers in this scenario is you guys, the people that have done the orders to the restaurant, in contraposition the actual producer is the chef/s in the restaurant that are constantly producing the dishes for you guys to eat constantly and the _intermediate_ **buffer** in this case is the table in the middle of the restaurant where the dishes get deposited in.

  This implies that there are dishes with a variety of sizes and colors and each one gets consumed responsibly by **one** person, it is a clean way to avoid conflic, everyone knows his dish and everything gets consumed slowly but surely without any kind of confusion.

## DisplayList

 In our engine the intermediate **buffer** in this case is what we call a "DisplayList" it is a **list** alongside an atomic for multithreaded access, as previously explained, many people will be accessing the table in the restaurant, so having an atomic in this case would prevent race conditions or collisions between threads, it is an **extra** safe measurement that needs to be taken at the expense of slowing down the speed.

  In our engine the item that is going to be put in the _intermediate_ **buffer** is what we call the "DisplayList" it is a **list** alongside an atomic for multithreaded access, we expect many consumers to access this item to either look or process it thoroughly so an atomic is just a preventive measure in case we have race conditions or thread collisions that we obviously want to avoid, it is just an **extra safety** measurement that needs to be taken at the expense of slowing down the whole process of item consumption by the consumers. (_If this was the restaurant scenario you would not want to eat too fast or eat without looking you might get a stomach ache or food posoning! _)

 ```cpp

  class DisplayList() {
    
  public:
    DisplayList();
    ~DisplayList();

    DisplayList(DisplayList const&) = delete;
    DisplayList& operator=(DisplayList const&) = delete;

    DisplayList& operator=(DisplayList&& o) {
      // ...
    }
    DisplayList(DisplayList&& o) {
       // ...
    }

    void Add( Command && newRC ) ;
    void Run();
    
    inline void SetSwap(bool isSwap) {
      // ...
    }
    
    inline bool swap() const {
      // ...
    }

    u32 Size() const {
      // ...
    }    

   private:
    bool swap_;
    std::atomic<int> lockValue_;
    std::list< Command > internalList_;
  }


```

  In this specific case, the items that will be placed in the _intermediate_ buffer are pretty simple to work with, as seen in the previous piece of code, you can see that those lists work with what we call "Commands" in our engine, we can understand that this list can hold various commands that the actual engine needs to execute thorougly in a very specific and concatenated order so everything works as expected and nothing breaks.

  The `Add()` function pretty much adds a Command to the actual internal list and the `Run()` flushes the whole list from index `0 to size-1` of the list to be execute each one of the individual commands. 

  Now you might be wondering about the concept of `Command`, it is a very broad one and in a graphical engine you might expect it to be pretty much anything, in this case in our engine we have it split up in two distinct types of commands:

  - Render Command.
  - Logic Command.

  I will get back to those later on in this blog post, but firstly I will be explaining the function of the _intermediate_ buffer and what it is in our engine.
  

## DisplayList Deque

  The _intermediate_ **buffer** I was talking about in this case is the **double-ended list** that we know as "DisplayList Deque", this can be understood as the table or foundation in which all the items (_DisplayList_) are deposited in and processed by the consumers. 

  The main function it has is to extract the actual item from the front of list and process it through the internal `Run()` command that the item has. Of course we do not need to forgive that the actual buffer needs to be protected with an atomic too because this item is going to be heavily requested by a lot of consumers, hence why the access to it has to be highly individualized per thread.

 ```cpp

  class DisplayList() {
    
  public:
    DisplayList();
    ~DisplayList();

    DisplayList(DisplayList const&) = delete;
    DisplayList& operator=(DisplayList const&) = delete;

    DisplayList& operator=(DisplayList&& o) {
      // ...
    }
    DisplayList(DisplayList&& o) {
       // ...
    }

    inline void SubmitListWithSwap( DisplayList && dl) {
      // ...
    }
    
    inline void SubmitList(DisplayList && dl) {
      // ...
    }

    inline u32 Size() const {
      // ...
    }
    
    inline DisplayList ExtractList() {
      // ...
    }

   private:
   
   inline void lock() {
      // ...
   }

   inline void unlock() {
      // ...
   }
   
    std::atomic<int> lockValue_;
    std::deque < DisplayList > data_;
  }


```
 


