---
layout: post
slug: breakout-cpp-part1
title: "Making Breakout with Data-Oriented Design (Part 1)"
image: /assets/2020-12-12-breakout-cpp-part1/main.png 
date: 2020-12-12 12:00:00 -0500
author: Matt Gibson
tags: BreakoutCpp Cpp DataOrientedDesign
github_issue_id: 3
assets_folder: /assets/2020-12-12-breakout-cpp-part1
---

This is part 1 in a series about re-creating the game breakout in C++ using data oriented design. In this episode, I talk about the reason for starting the project and the rules for implementing it.

![Input Action Asset]({{ page.assets_folder }}/main.png)

## But Why?
As someone who primarily codes in the world of object oriented programming (OOP), I've been looking for a excuse to try out other programming styles. The one that jumps out to me the most is data oriented design, specifically because it's all about writing programs that leverage the way a computer actually runs code.  

From courses in university, I have some understanding of how operating systems and underlying computer hardware work. But built on top of that are several layers of abstraction, one of them sometimes being OOP. The main advantage of OOP is that it let's us reason about our code the way a lot of us think about the real world: as the interactions between objects. However, this mental model takes us away from how the computer is really managing and operating on our data. I want to try shifting my own mental model back towards that. 

Some of my inspiration for trying out a more data oriented approach came from watching content from Jonathan Blow (creator of Braid and The Witness) and Casey Muratori (creator of Handmade Hero). They are both pretty outspoken of there dislike of OOP, and from watching them program a bit over the past few years it's gotten me pretty interested in there style of programing. The idea for this project specifically came from a [GDC talk](https://youtu.be/GQx20iDBx9c) by Elizabeth Baumel. In it she describes creating breakout using data oriented design and I thought that would be a good starting project for me to do. I've only watched through it once and haven't looked at her slides in detail, I want to try and make those discoveries myself through actually doing the project.

It's also worth mentioning that I haven't programmed in c++ in a long time, I don't plan on using many of the modern C++ features but I am looking forward and a bit scared of getting back into managing my own memory (rather than letting the GC handle all that for me).

Having said all that, let's get into the plan for this project.

## The Plan
Objective: Create a very simple breakout game in C++ using data oriented design.

**Gameplay and Features**: Gonna keep this very simple with low quality graphics (lots of white rectangles on a black background) and no menus. The game will just begin when the player starts moving and every time the player clears the screen the tiles reset and gain one extra health. The player will have 3 lives and get 1 point for every tile hit.

### Restriction
I'm gonna be placing some restrictions on myself so that I don't fall back into programming the way I'm use to.

**No Member Functions**: From the CPU's perspective data cannot perform it's own actions, actions are instead performed on data. For this reason I don't want to have any functions inside my classes/structs, instead I'll need to create functions that take the classes/structs as a parameter. Member functions are just regular functions that take in a hidden `this` argument, but I'm gonna make this restriction anyways to get my self out of that object oriented mindset.

**No Private Variables**: If I can't have member functions then there's no point in having private variables, since there won't be any code that is able to access them. OOP puts a strong emphasis on encapsulation and abstraction, this goes directly against that. I'm pretty curious to see how I find working with all data being public.

**Use Minimal C++ Features**: This project isn't meant for learning a bunch of new language features. On top of that, Jonathan Blow (one of the inspiration for this project) started creating his own programming language specifically because of how complicated C++ has gotten with the addition of so many new features. For this reason, I'm planning on using very little C++ features and sticking mostly to features already in C.

**No Heap Allocations Every Frame**: In the game breakout there is a finite set of data to represent every possible state that the game is in, so the amount of memory I need can probably be set on initialization of the game. I shouldn't need to be doing things like allocating new memory on the heap every frame. Additionally random heap allocations can make for poor cache performance since data can become fragmented, so I want to avoid that.

## Next Steps
The next steps of the project are going to be mostly set up. Getting a program compiling that displays a window and some simple graphics. I'm thinking of using the GLFW library to hopefully get the project running on windows and mac, but we'll see how it goes. 

See you all in Part 2!
