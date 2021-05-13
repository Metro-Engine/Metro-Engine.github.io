---
layout: post
title: Metro Engine A New Beginning
subtitle: OpenGL / PS4 Engine
tags: [engine, opengl, ps4, esat]
---

##The Beginning

  The decision to create a graphical engine such as **Metro Engine** was pretty straightforward, both of us are students in an university called **ESAT** (__Escuela Superior De Arte y Tecnología__). One of our subjects is "Graphic Engine Programming", in this subject we were tasked with the creation of a multithreading engine that at least ran in one platform such as Windows with **OpenGL** as the graphic backend.

  We are currently studying our last year in this university and we had somewhat some experience with OpenGL and the creation of an engine in general because they taught us the basics in our second year of studies there. The beginning was pretty harsh, something that was not easy for us to understand was the concept of **Multithreading**, being able to use more than one core of your processor was something that had to be taught to us because nowadays, everywhere multithreading is a must, so creating an engine that was ready to be used with multiple cores was a requirement for this subject.

  The first thing with which we started was the project organization through **GEnie** and the creation of a **private repository** in Github as per request of the teacher and the university. This were some crucial steps, to be able to lay a very strong foundation from which our engine could be built and started with ease so we did not need to do anything but execute a script to be able to build it with Visual Studio 2017 or Visual Studio 2019.

  After arranging all the basic libraries for OpenGL we decided to implement GLFW as our backend for the window previsualization for the engine using the GLEW backend for OpenGL. It was a pretty easy implementation that did not cost us too much time and we definitely enjoyed implementing the core skeleton of the engine with the respective loop.

~~~
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

~~~

{: .box-note}
**Note:** Notice how we wrapped the window creation functionality of GLFW in a class called "**app.h**" so we could maybe extend it in the future to multiple viewports / windows that could be opened simultaneously by the engine.



