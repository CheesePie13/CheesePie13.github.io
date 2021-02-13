---
layout: post
slug: breakout-cpp-part3
title: "Making Breakout with Data-Oriented Design (Part 3)"
image: /assets/2021-02-12-breakout-cpp-part3/main.png 
date: 2021-02-12 12:00:00 -0500
author: Matt Gibson
tags: BreakoutCpp Cpp DataOrientedDesign
github_issue_id: 5
assets_folder: /assets/2021-02-12-breakout-cpp-part3
---

The final part in my series about re-creating the game breakout in C++ using data oriented design. In this episode, I talk about the final results of the project.

<!--more-->

![Input Action Asset]({{ page.assets_folder }}/gameplay.png)

In my last post, I said I would be talking about my plan for implementing the game next. I had a full plan layed out and about half a blog post written. But instead of finishing the blog post, I ended up just diving in to programming it according to my plan. It would feel kind of weird now talking about my plan for the project after I've already finished it. I also think the plan I had written out can be better explained by just looking at the source code. I'd rather talk about the experience writing the code and what I got out of the project overall.

If you'd like to see the final version 1.0 of the source code [click here](https://github.com/CheesePie13/BreakoutCpp/tree/v1.0) and you can download the release version [here](https://github.com/CheesePie13/BreakoutCpp/releases/tag/v1.0).

## How was it?
I had a good time writing the code for this game. It feels good to have completed a full project even when it's small. It can definitely be hard to find the energy and motivation to work on a side project when you already spend most of your day programming at work. It's also really easy to let the scope of side projects get too large. Even in this project, I still have some `todo` comments left in the code for things I would improve. I'm glad though that I put an end to the project and released a 1.0 version, it's not perfect but at least it's done. Sometimes I can still feel guilty for abandoning a project when I feel I've gotten what I wanted out of it or feel it's not leading anywhere. Putting a definitive end to it can release that guilt and even give a sense of accomplishment to motivate me for the next project.

One benefit of doing any project by yourself is the feeling of ownership, the feeling of being able to point at something and say "that was all me". That isn't to take away from how rewarding it can be to work on a team and build something greater than you could alone. I think I would miss either one of those feelings if I spent to long without it, but I'm glad this project got to scratch my solo programing itch. One of the great things about working on a solo project, is it allows a lot more opportunities for exploration. You don't need to worry about another programmer having to deal with your experimental ideas when you're the only one touching the code. I know some programmers can be very set in there ways as to what is the best way to program. It's nice to have an opportunity to be as creative as you want, and if you stumble across any good ideas at that point you can share them with other programmers.

Something I'm realizing from this project is that my biggest source of pain when programming is interfacing with code that I didn't write. For this project it was really minimal. There was GLFW for handling calls to the OS, this involved some pretty simple set up and then a couple more calls to get the keyboard input values I wanted. There was OpenGL for rendering the graphics, I've messed around with it a bit before and new the basics. The graphics in the game are really minimal anyways so I didn't have to do more than just render a bunch of quads. And finally there was the compiler, and for that I only chose to use a hand full of C/C++ features. Having given myself that restriction really saved me from having to get into all the complexity that C++ has gained over the years with it's newer features. Because of all of this, very little went wrong. My plan for how all the logic would operate on the data just worked as planned, that almost never happens when programming, there are pretty much always bugs and quirks in the code I have to interact with that need work arounds. For this project that didn't happen at all, everything I planned just worked because most of the code was my own and I could directly fix it when there were any issues. It was a nice change but for larger projects writing all the code yourself can take a lot of time and effort, that's the trade off.

A final thing I wanted to note was the collision detection. That was the hardest part that I had to write, and probably the most fun for my brain to figure out. It's cool to have a bit more knowledge about how physics ray-casting and collision detection potentially work. It could give me more insight when working with physics engines in the future.

## Was it worth it?
So was this project worth it? Well first I think the project was too small to get some of the learnings I wanted. It's clear to me that data oriented design works well on small projects. The code for this project was really easy to work on, the logic flow was very readable and all the data for the project was contained within a few main structs. This made understanding all the state of the game real easy. The real question here is, does it scale?

A lot of the motivation I see behind the main principles of Object Oriented Programing (OOP) is being able to interface with other code without knowing how it's implemented. For me, that ends up being one of the biggest problems I run into with OOP. When I'm calling into an interface where I don't know what exact code it will run, I have to make a lot of assumptions. First I need to hope I understand the meaning of the function and that I am using it for the right purpose. Beyond that I need to assume what side effects it may cause and what are the performance implications of calling it. For this project, using data oriented design, I had a lot less interfaces. My main interface was between the game code and the application layer that handled OS calls. I could also see adding a layer for the graphics to handle swapping out different graphics APIs.

OOP and Data Oriented Programming seem to be on opposite sides of the spectrum. OOP tries to have a lot of interfaces to allow the programmer to program against abstractions. Data Oriented tries to have few interfaces and allow the programmer to program against the actual code and data. Specifically for small projects, working on this one made me enjoy programming without so many interfaces and operating directly on the data. However, it didn't really helped me understand if it would work for bigger projects. Do the interfaces of OOP make programming at a larger scale manageable or would I still be able to wrap my head around all the data and code without the interfaces? That is something I still have to find out.

Overall I'm glad I took on this project, if nothing else it got me more familiar with OpenGL and got me use to programming in C/C++ again. But also, it let me know that if I really want to know if Data Oriented Programming is better than Object Oriented Programming, I'm gonna have to test that out on bigger projects in the future.

## What's next?
So what's next? My plan for now is to switch my focus back to Unity. I have some projects to finish in Unity and a couple blog post in the works for Unity related concepts. I'll eventually come back to C++ and data oriented design because there is a lot more I want to explore, but I think even in Unity there is more for me to explore. I really want to find and hone my Unity programming style right now, so that's my goal for the near future. Aspects of data oriented design may even work there way into my style. Unity is after all still working on there Entity Component System, I'm very interested in seeing how that turns out.

Anyway thanks for reading, hope you are having a great day and see you in my next post!
