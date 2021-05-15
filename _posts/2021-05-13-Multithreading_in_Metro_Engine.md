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
    printf("[SCHEDULER] Initialized scheduler with %d threads\n", numThreads);
  }

  void DoSomething() {
    printf("I am doing something!\n");
  }


  auto myNewTask = [] {
    DoSomething();
  };
  
  schedulerHandler_.run(myNewTask, &logicSync_);


```
  
  Of course this is just a dumb example on how a task can be executed, you can clearly see that you need an initialize function for the scheduler handler which in this case would be `InitializeScheduler()` and it would accept as parameters the number of threads that would be utilized by given scheduler to actually send the tasks to. 
  
  Aside from that we also hace the custom structure `px_sched::SchedulerParams customSchedParams_` which we need to fill with specific information so the handler can execute in a correct manner.
  
  We need to remark that the use of lambdas / anonymous functions is a very common way to send specific functions to the scheduler, it is a very common way to wrap functions or specific bits of code that you want to be ran within a specific scope by the scheduler.
  
  One of the weird things here is the `run(task, syncObject)` function that the scheduler has, we can see that the first parameter is probably a lambda or a `void*` that is later called by the scheduler, but what is the synchronization object?
  
  As this is a task scheduler, you need to have some kind of way of controlling the flow of the tasks and the order they are ran in, that is why this synchronization objects that we can refer as `fences` exist. They are used as the gatekeepers for the tasks to maintain a cohesive and coherent order between each other, it is a necessity to be able to control the flow of the tasks without them being in complete utter chaos without being executed in random order.
  
  ![FencesExample](https://i.imgur.com/wpes1Tt.png)
  
    
  In the image above we can see that we want to execute from `Task #1` to `Task #4` in a sequential way, notice that in this case we have the concept of `fences` which we can associate as "**synchronization objects**". The example code for the above image would be the following:

```cpp

  int main(int, char **) {
    px::Scheduler schd;
    schd.init();

    px::Sync fence1, fence2, fence3;
    
    auto task1 = [] {
      // ...
    };
    
    auto task2 = [] {
      // ...
    };
    
    auto task3 = [] {
      // ...
    };
    
    auto task4 = [] {
      // ...
    };

    schd.run(task1, fence1);
    schd.runAfter(fence1, task2, &fence2);
    schd.runAfter(fence2, task3, &fence3);
    schd.run(task4, fence3);
    
    schd.waitFor(fence3);
    printf("All tasks have been executed, finalizing.\n");

  }

```

  As you can see we have introduced a new function from the scheduler which in this case is `runAfter()` this functionl is ideally used to concurrently run a task, assign it to a specific fence and make it wait for other fence to finish up the tasks it has.
  
  This works pretty much like a _semaphore_ telling every single car when to pass to the next area whilst avoiding any kind of collision or misbehavior with the car, it is a pretty cool way to synchronize and run tasks while virtually losing 0% speed because your pipeline is completing all the concurrent tasks and moving on onto the next batch of tasks assigned to the next fence until it finally finishes, the previous example was pretty **sequential** and it did not shine in brigh on the capabilities of the library, the strengths of this library is that it can run for example `10,000 concurrent tasks` and run after a specific fence has finished.
  
   We also need to remark that at the end of the example we have another function that is called `waitFor`, this function pretty much halts the scheduler and waits for a specific fence to finish all the concurrent tasks it has for it to advance. 
   
   This is in fact **detrimental** for the performance of the scheduler and it is **heavily suggested** to avoid using `waitFor()` as much as you can, because this essentially kills the performance of your software, in threading you want a continuous pipeline that is running constantly and all the threads are working and are not halted or doing absolutely nothing to achieve **maximum** performance. Here is another _schema_ of multiple tasks being ran concurrently whilst waiting for a synchronization object (_fence_) to finish so it can run the following task.

![Fencing Example Concurrent](https://i.imgur.com/XktMhcm.png)

  Ideally you can see that this time in the leftmost part of the fence we have two tasks that are going to be ran before proceeding with the final task which in this case is number 3. Here is the code example for this scenario:

```cpp

  int main(int, char **) {
    px::Scheduler schd;
    schd.init();

    px::Sync fence1, fence2;
    
    auto task1 = [] {
      // ...
    };
    
    auto task2 = [] {
      // ...
    };
    
    auto task3 = [] {
      // ...
    };
    
    auto task4 = [] {
      // ...
    };

    schd.run(task1, &fence1);
    schd.run(task2, &fence1);
    schd.runAfter(fence1, task3, &fence2);
    
    schd.waitFor(fence2);
    printf("All tasks have been executed, finalizing.\n");

  }

```

  I am showing simple and stupid examples here, but remember that you can send a huge amount of tasks to a fencing object just like in this example:

```cpp
  int main(int, char **) {
    px::Scheduler schd;
    schd.init();

    px::Sync fence1;
    
    for(int i = 0; i < 10000; ++i) {
      auto job = [i] {
        printf("Task %d: Completed from %s\n",
          i, px::Scheduler::current_thread_name());
      };
      schd.run(job, &fence1);
    }

  }

```
  With the basic explanation on how the scheduler works and how the synchronization between tasks and fences is done, we have Metro Engine with the separation of the `Update()` and `Draw()` loops and the internal loop looks as following:

```cpp

void App::Run() {

  auto logicThread = [] {
    Upadte();
  }

  while (!window.ShouldClose()) {
    Input();
    {
      schd.run(logicThread, &logicSync);
    }
    DrawFromMainThread();
    {
      schd.waitFor(logicSync);
    }
  }
}

```
  To finalize and wrap it up you can see that the engine itself treats the logic in a separate thread and the drawing and input is kept on the main thread and it is constantly being ran while the window from `GLFW` is not closed. 

  This is very important because all things render-wise are being drawn in the main thread where all the OpenGL commands in this case are being executed and processed through the **Display Deque** and the logic is being ran in a separate thread (**Logic Commands**).

  Hopefully this clarified how the multithreading works in our engine and shade a light on how this **px_sched** single header library developed by **[pplux](https://github.com/pplux)** works and how easy it is to integrate with any kind of engine to start having a better performant engine with just the snap of a finger!

  We encourage you to try this library and do all the pertinent testing by yourself and hopefully you might like it and keep using it, remember it is open-source!   
