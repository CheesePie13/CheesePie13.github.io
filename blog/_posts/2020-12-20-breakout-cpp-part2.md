---
layout: post
slug: breakout-cpp-part2
title: "Making Breakout with Data-Oriented Design (Part 2)"
image: /assets/2020-12-20-breakout-cpp-part2/main.png 
date: 2020-12-20 12:00:00 -0500
author: Matt Gibson
tags: BreakoutCpp Cpp DataOrientedDesign
github_issue_id: 4
assets_folder: /assets/2020-12-20-breakout-cpp-part2
---

Part 2 in a series about re-creating the game breakout in C++ using data oriented design. In this episode, I talk about how I set up the project to build for multiple platforms.

![Input Action Asset]({{ page.assets_folder }}/main.png)

## How did it go?
All things considered, the set up of the project went relatively smoothly. I got a window up showing a square that you can move horizontally with the arrow keys (this will eventually be the paddle at the bottom of the screen). You can see the code (version 0.1) [here](https://github.com/CheesePie13/BreakoutCpp/tree/v0.1) or download the release version [here](https://github.com/CheesePie13/BreakoutCpp/releases/tag/v0.1). Let's get into my experience setting up the project and some of the decisions I made.

## Compiling
For compiling I wanted to keep it as simple as possible. I thought about using a build tool like CMake, but decided against it. It would be another program that you'd need to install to build and I didn't want to waste time learning how to set up a CMake project. Definitely something I'll spend a little time learning in the future but not right now. For this project I decided to go with good old reliable build scripts.

I've used Microsoft's Visual C++ compiler for test projects in the past, but setting it up was a pain. There's a special batch file you need to run before running the compiler to set up all the environment variables and there are a lot of compiler options I needed to add to get it compiling the way I want. For this project I decided to try out a different compiler option.

G++ was the compiler I used in university and I remember it being pretty straight forward. From a little research I found that there was a windows port called mingw-w64 and decided to try it out. Got it compiling a hello world program pretty easily so I decided to go with it. I vaguely used [this tutorial](https://code.visualstudio.com/docs/cpp/config-mingw) to set up building and debugging with VS Code.

After I got the project set up on windows I decided to try creating a mac version too since GLFW supports it. I created a bash script with the same g++ call as the windows build, and with a few tweaks (like including the mac frameworks needed for GLFW) I got it compiling. I didn't look into it but it seems like g++ is set up as an alias for clang on mac, you might need to have Xcode installed for it to work.

## GLFW 
In the past I've used 2 different options for the platform layer of a game (aka the code that handles interfacing with the operating system). I've used GLUT when first messing around with OpenGL a long time ago. A bit more recently I wrote the platform layer myself for windows using the Win32 API by following along with the [Handmade Hero](https://handmadehero.org/) series. My knowledge of the Windows API is very limited so I didn't want to go with writing it myself again, I know that there are other libraries out there that are better written and, most importantly, well tested against a larger sample of computers. 

I decided to go with the GLFW library base on recommendations from other game devs I've seen online. It went pretty smoothly, I went with the pre-compiled binaries for windows and mac (to avoid needing CMake) and followed the tutorial on their [website](https://www.glfw.org/). Got a blank window showing up pretty quickly and wasn't hard after that to get OpenGL working with it.

## Loading Shaders
The one cross platform feature that GLFW wasn't able to handle for me was setting up the working directory. I wanted the working directory to be the directory the executable is in because I plan on storing the game assets in that folder (for right now just the shaders). I believe there is an option to have GLFW use a mac apps resource folder as the working directory. However my mac build is just an executable, not a full app, so I needed a different solution. 

I found a function for each platform that would get me the path to the executable, I just stripped off the file name and set that to be the working directory. You can see the code here:

```cpp
uint32 exec_path_size = 1024;
char exec_path[exec_path_size];

#ifdef WINDOWS
GetModuleFileName(NULL, exec_path, exec_path_size);
*strrchr(exec_path, '\\') = '\0'; // Remove file name after last '\'
#elif MACOS
_NSGetExecutablePath(exec_path, &exec_path_size);
*strrchr(exec_path, '/') = '\0'; // Remove file name after last '/'
#endif

chdir(exec_path);
std::cout << "Working Directory: " << exec_path << std::endl;
```

## Platform Layer Separation
I got the idea from the Handmade Hero series to separate the platform layer from the actual game logic code. The platform layer is all the code responsible for interfacing with the OS. Like setting up the window, running the game loop and getting keyboard input. The [src/game.hpp](https://github.com/CheesePie13/BreakoutCpp/blob/v0.1/src/game.hpp) file defines the interface between the platform layer and game logic.

There are 3 functions that the platform layer calls to run the game. The `init()` function sets up the initial game and allocates data for the `State` struct. This State contains all the data for the game, the platform layer doesn't care what is in this struct but it needs to be passed back to the game when ever `update()` or `render()` is called. The Input struct is also passed into all these functions, this struct is populated by the platform layer and contains data like the frame time delta and the current state of the keyboard keys we care about.

```cpp
// Initialize the game
State* init(const Input* input);

// Update the game logic for a frame
void update(const Input* input, State* state);

// Render a frame
void render(const Input* input, State* state);
```

One thing I would like to do but may not for this project is separate out all the code for rendering. Currently it's fully mixed in with the game logic but this isn't ideal if I wanted to support multiple rendering APIs. For this small project I probably won't bother but I'm definitely intrigued to find out how to best architect that sort of separation. Probably something I will dive into more on a future project.

## Types
In the code I have the file [src/types.h](https://github.com/CheesePie13/BreakoutCpp/blob/v0.1/src/types.h) that redefines the numeric types. Ints and floats are so common in game code. I like having consistent and short type names for these types. It always seemed crazy to me that the size of and int was never guaranteed, so I prefer using type names like `int32` so I always know what size of int I'm using. I've also seen some people use even shorter type names like `i32`, `u16` and `f64`, maybe I'll switch to that in the future 🤔.

## Variable Naming
I pretty much only ever use PascalCase and camelCase when programming day to day, sometimes I use snake_case for config files but not usually in actual code. I have herd the case being made that snake_case is more readable in code so I wanted to try it out. So far I don't mind it. If I had to pick I would probably say I prefer snake_case for function names and camelCase for variable names. Honestly it doesn't really matter though, I think the most important thing is just being consistent within a project. But that's even hard when you're dealing with libraries that use a naming convention that doesn't match your project. You could go the route of some languages that define a convention for the whole language but then you might be forced into a style you don't like, like indenting with spaces, (no offence if you do) so really it's just all bad and I should stop even talking about this because there is no real solution just do whatever you want and what I want to do is just end this run on sentence so I'm gonna do that now.

## Conclusion
Overall the set up for this project went pretty smoothly and I like how I set most of it up (especially considering it's a small project and I didn't want to over engineer it). The next step now is to plan out how I'm gonna be implementing this thing and that's gonna be in my next blog post. I'll see ya then!

