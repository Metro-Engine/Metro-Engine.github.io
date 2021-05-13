---
layout: post
title: Metro Engine A New Beginning
subtitle: OpenGL / PS4 Engine
tags: [engine, opengl, ps4, esat]
---

## The Beginning

  The decision to create a graphical engine such as **Metro Engine** was pretty straightforward, both of us are students in an university called **ESAT** (__Escuela Superior De Arte y Tecnología__). One of our subjects is "Graphic Engine Programming", in this subject we were tasked with the creation of a multithreading engine that at least ran in one platform such as Windows with **OpenGL** as the graphic backend.

  We are currently studying our last year in this university and we had somewhat some experience with OpenGL and the creation of an engine in general because they taught us the basics in our second year of studies there. The beginning was pretty harsh, something that was not easy for us to understand was the concept of **Multithreading**, being able to use more than one core of your processor was something that had to be taught to us because nowadays, everywhere multithreading is a must, so creating an engine that was ready to be used with multiple cores was a requirement for this subject.

  The first thing with which we started was the project organization through **GEnie** and the creation of a **private repository** in Github as per request of the teacher and the university. This were some crucial steps, to be able to lay a very strong foundation from which our engine could be built and started with ease so we did not need to do anything but execute a script to be able to build it with Visual Studio 2017 or Visual Studio 2019.

  After arranging all the basic libraries for OpenGL we decided to implement GLFW as our backend for the window previsualization for the engine using the GLEW backend for OpenGL. It was a pretty easy implementation that did not cost us too much time and we definitely enjoyed implementing the core skeleton of the engine with the respective loop.

```cpp
// Metro Engine Main

#include <app.h>

using namespace Metro;


void Metro_Main() {
  srand(time(NULL));
  App application;

  application.Initialize();
  application.Run();
  application.End();
}

int main() {
  Metro_Main();
}

```

{: .box-note}
**Note:** Notice how we wrapped the window creation functionality of GLFW in a class called "**app.h**" so we could maybe extend it in the future to multiple viewports / windows that could be opened simultaneously by the engine.


## Expanding Horizons


  We did want to learn a lot from this engine so our future perspectives were high, to be able to get a higher score in our subject we had to be able to run our engine in other platform that was not **Windows**, that is why we had in mind porting the engine to platforms such as Mac or Linux.

  Nonetheless porting an engine to another platform is a challenge in itself, the code needs to be adapted and alongside GENie it needs to be able to work flawlessly in both platforms that it is meant to be working in. This idea kinda clicked for us because expanding our horizons and learning how to port our engine to different platforms was a valuable learning experience that we had to have in our pocket because we **for sure** know that in the future we will stumble upon porting a piece of software, we need to be ready for the challenges we might stumble upon in the near future as junior/senior developers.

  As any young developer, we both are pretty ambitious people and we meticulously thought about what platform we should port the engine, whenever we scrolled through LinkedIn we noticed that a lot of companies asked as a **bonus** / **extra** to have experience with **consoles**, so we both thought that having that little bonus on our CV / Portfolio would increase our odds to get a job to dive into the industry, so we decided to go for that.

  Luckily the university is a **Sony Partner** and have access to the **PS4 Devkit**, so we signed up an **NDA** with **Sony** and **ESAT** and began our journey to port the engine to PS4, it was a pretty rough and difficult journey due to **COVID** we could not be everyday developing with the development kit so we had to do a lot of **blind coding** without any kind of way of checking it it was correct, at the end of the day we got a product that we were happy with and we are happy to say that we are proud with the porting to this console, it was in general an amazing experience that both of us will not forgive because it was our first experience fighting with a very large API and a completely different architecture from what we are used to, which is the PC.



