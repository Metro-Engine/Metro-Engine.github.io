---
layout: post
title: Multithreading in Metro Engine
subtitle: Px-Sched and how it is used
tags: [multithreading, tasks, synchronization, px_sched, pplux]
---

## Multithreading <Px-Scheduler>
  
  Before actually reading about **multithreading** and how it is handled in the engine, I recommend you to read [Metro Engine Structure](https://metro-engine.github.io/2021-05-13-Metro_Engine_Structure/)  blog post just to have more information on the why are we doing it this specific way and what possibilities we do get from doing it this way specifically.

  As this was one of the requirements that need to be met for the subject that we were studying in university, our engine had to be for sure multithreaded, this meant that we had to at least separate the render and the logic loops in different threads for them to be ran simultaneously and having a healthy and synchronized communication without any kind of _race conditions_ or _thread collisions_.
  
  In this specific scenario we do utilize an auxiliary library that was coded by our teacher and it is called "**Px_sched**" it is a single header task oriented scheduler that serves the purpose to paralelyze all the little tasks that our engine can do.
  
  As mentioned previously in the other post we comment that we have an `Update()` and `Draw()` loops that are being both executed in different threads, this means that through this single header library we separate the execution of those loops in different tasks, it is very easy to create a task and it is done the following way:

```cpp

  px_sched::Scheduler schedulerHandler_;
  px_sched::SchedulerParams customSchedParams_;
  px_sched::Sync logicSync_;

  // Put this funciton in a main init() function.
  void InitializeScheduler(int numThreads) {
    // ...
    customSchedParams_.num_threads = numThreads;
    schedulerHandler_.init(customSchedParams_);
    printf("[SCHEDULER] Initialized scheduler with %d threads\n, numThreads);
  }

  void DoSomething() {
    printf("I am doing something!\n");
  }


  auto myNewTask = [] {
    DoSomething();
  }
  
  schedulerHandler_.run(myNewTask, &logicSync_);


```
  
  Of course this is just a dumb example on how a task can be executed, you can clearly see that you need an initialize function for the scheduler handler which in this case would be `InitializeScheduler()` and it would accept as parameters the number of threads that would be utilized by given scheduler to actually send the tasks to. Aside from that we also hace the custom structure `px_sched::SchedulerParams customSchedParams_` which we need to fill with specific information so the handler can execute in a correct manner.
  
  We need to remark that the use of lambdas / anonymous functions is a very common way to send specific functions to the scheduler, it is a very common way to wrap functions or specific bits of code that you want to be ran within a specific scope by the scheduler.
  
  One of the weird things here is the `run(task, syncObject)` function that the scheduler has, we can see that the first parameter is probably a lambda or a `void*` that is later called by the scheduler, but what is the synchronization object?
  
  As this is a task scheduler, you need to have some kind of way of controlling the flow of the tasks and the order they are ran in, that is why this synchronization objects that we can refer as `fences` exist. They are used as the gatekeepers for the tasks to maintain a cohesive and coherent order between each other, it is a necessity to be able to control the flow of the tasks without them being in complete utter chaos without being executed in random order.
  
  ![FencesExample](https://i.imgur.com/I9Q2TuF.png)
  
  
  
  
  
  
  
