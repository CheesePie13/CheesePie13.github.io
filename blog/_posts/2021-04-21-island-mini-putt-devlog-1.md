---
layout: post
slug: island-mini-putt-devlog-1
title: "Island Mini Putt - Devlog #1 - The First Milestone"
image: /assets/2021-04-21-island-mini-putt-devlog-1/main.png
date: 2021-04-22 12:00:00 -0500
author: Matt Gibson
tags: IslandMiniPutt Unity Devlog
github_issue_id: 12
---

The inception of Island Mini Putt. A new video game experience that will revolutionize the game of golf as we know it... or maybe it's just a fun little mini golf game that takes place on an island :D

<!--more-->

## Art Direction

Mini golf has always been one of my favorite activities to do when ever I actually go outside. That's why when I was messing around in [MagicaVoxel](https://ephtracy.github.io/), trying to create some new weird "art" for my instagram account, this vision popped into my head. A lonely island in the middle of the ocean that has nothing on it except a classic windmill mini golf hole.

You can see the instagram post below I made on my account [CheesePie1313](https://www.instagram.com/cheesepie1313/). Be warned, there's a reason I call my instagram posts "art" (in quotes).

<blockquote class="instagram-media" data-instgrm-permalink="https://www.instagram.com/p/CL5NvDnBqpu/?utm_source=ig_embed&amp;utm_campaign=loading" data-instgrm-version="13" style=" background:#FFF; border:0; border-radius:3px; box-shadow:0 0 1px 0 rgba(0,0,0,0.5),0 1px 10px 0 rgba(0,0,0,0.15); margin: 1px; max-width:540px; min-width:326px; padding:0; width:99.375%; width:-webkit-calc(100% - 2px); width:calc(100% - 2px);"><div style="padding:16px;"> <a href="https://www.instagram.com/p/CL5NvDnBqpu/?utm_source=ig_embed&amp;utm_campaign=loading" style=" background:#FFFFFF; line-height:0; padding:0 0; text-align:center; text-decoration:none; width:100%;" target="_blank"> <div style=" display: flex; flex-direction: row; align-items: center;"> <div style="background-color: #F4F4F4; border-radius: 50%; flex-grow: 0; height: 40px; margin-right: 14px; width: 40px;"></div> <div style="display: flex; flex-direction: column; flex-grow: 1; justify-content: center;"> <div style=" background-color: #F4F4F4; border-radius: 4px; flex-grow: 0; height: 14px; margin-bottom: 6px; width: 100px;"></div> <div style=" background-color: #F4F4F4; border-radius: 4px; flex-grow: 0; height: 14px; width: 60px;"></div></div></div><div style="padding: 19% 0;"></div> <div style="display:block; height:50px; margin:0 auto 12px; width:50px;"><svg width="50px" height="50px" viewBox="0 0 60 60" version="1.1" xmlns="https://www.w3.org/2000/svg" xmlns:xlink="https://www.w3.org/1999/xlink"><g stroke="none" stroke-width="1" fill="none" fill-rule="evenodd"><g transform="translate(-511.000000, -20.000000)" fill="#000000"><g><path d="M556.869,30.41 C554.814,30.41 553.148,32.076 553.148,34.131 C553.148,36.186 554.814,37.852 556.869,37.852 C558.924,37.852 560.59,36.186 560.59,34.131 C560.59,32.076 558.924,30.41 556.869,30.41 M541,60.657 C535.114,60.657 530.342,55.887 530.342,50 C530.342,44.114 535.114,39.342 541,39.342 C546.887,39.342 551.658,44.114 551.658,50 C551.658,55.887 546.887,60.657 541,60.657 M541,33.886 C532.1,33.886 524.886,41.1 524.886,50 C524.886,58.899 532.1,66.113 541,66.113 C549.9,66.113 557.115,58.899 557.115,50 C557.115,41.1 549.9,33.886 541,33.886 M565.378,62.101 C565.244,65.022 564.756,66.606 564.346,67.663 C563.803,69.06 563.154,70.057 562.106,71.106 C561.058,72.155 560.06,72.803 558.662,73.347 C557.607,73.757 556.021,74.244 553.102,74.378 C549.944,74.521 548.997,74.552 541,74.552 C533.003,74.552 532.056,74.521 528.898,74.378 C525.979,74.244 524.393,73.757 523.338,73.347 C521.94,72.803 520.942,72.155 519.894,71.106 C518.846,70.057 518.197,69.06 517.654,67.663 C517.244,66.606 516.755,65.022 516.623,62.101 C516.479,58.943 516.448,57.996 516.448,50 C516.448,42.003 516.479,41.056 516.623,37.899 C516.755,34.978 517.244,33.391 517.654,32.338 C518.197,30.938 518.846,29.942 519.894,28.894 C520.942,27.846 521.94,27.196 523.338,26.654 C524.393,26.244 525.979,25.756 528.898,25.623 C532.057,25.479 533.004,25.448 541,25.448 C548.997,25.448 549.943,25.479 553.102,25.623 C556.021,25.756 557.607,26.244 558.662,26.654 C560.06,27.196 561.058,27.846 562.106,28.894 C563.154,29.942 563.803,30.938 564.346,32.338 C564.756,33.391 565.244,34.978 565.378,37.899 C565.522,41.056 565.552,42.003 565.552,50 C565.552,57.996 565.522,58.943 565.378,62.101 M570.82,37.631 C570.674,34.438 570.167,32.258 569.425,30.349 C568.659,28.377 567.633,26.702 565.965,25.035 C564.297,23.368 562.623,22.342 560.652,21.575 C558.743,20.834 556.562,20.326 553.369,20.18 C550.169,20.033 549.148,20 541,20 C532.853,20 531.831,20.033 528.631,20.18 C525.438,20.326 523.257,20.834 521.349,21.575 C519.376,22.342 517.703,23.368 516.035,25.035 C514.368,26.702 513.342,28.377 512.574,30.349 C511.834,32.258 511.326,34.438 511.181,37.631 C511.035,40.831 511,41.851 511,50 C511,58.147 511.035,59.17 511.181,62.369 C511.326,65.562 511.834,67.743 512.574,69.651 C513.342,71.625 514.368,73.296 516.035,74.965 C517.703,76.634 519.376,77.658 521.349,78.425 C523.257,79.167 525.438,79.673 528.631,79.82 C531.831,79.965 532.853,80.001 541,80.001 C549.148,80.001 550.169,79.965 553.369,79.82 C556.562,79.673 558.743,79.167 560.652,78.425 C562.623,77.658 564.297,76.634 565.965,74.965 C567.633,73.296 568.659,71.625 569.425,69.651 C570.167,67.743 570.674,65.562 570.82,62.369 C570.966,59.17 571,58.147 571,50 C571,41.851 570.966,40.831 570.82,37.631"></path></g></g></g></svg></div><div style="padding-top: 8px;"> <div style=" color:#3897f0; font-family:Arial,sans-serif; font-size:14px; font-style:normal; font-weight:550; line-height:18px;"> View this post on Instagram</div></div><div style="padding: 12.5% 0;"></div> <div style="display: flex; flex-direction: row; margin-bottom: 14px; align-items: center;"><div> <div style="background-color: #F4F4F4; border-radius: 50%; height: 12.5px; width: 12.5px; transform: translateX(0px) translateY(7px);"></div> <div style="background-color: #F4F4F4; height: 12.5px; transform: rotate(-45deg) translateX(3px) translateY(1px); width: 12.5px; flex-grow: 0; margin-right: 14px; margin-left: 2px;"></div> <div style="background-color: #F4F4F4; border-radius: 50%; height: 12.5px; width: 12.5px; transform: translateX(9px) translateY(-18px);"></div></div><div style="margin-left: 8px;"> <div style=" background-color: #F4F4F4; border-radius: 50%; flex-grow: 0; height: 20px; width: 20px;"></div> <div style=" width: 0; height: 0; border-top: 2px solid transparent; border-left: 6px solid #f4f4f4; border-bottom: 2px solid transparent; transform: translateX(16px) translateY(-4px) rotate(30deg)"></div></div><div style="margin-left: auto;"> <div style=" width: 0px; border-top: 8px solid #F4F4F4; border-right: 8px solid transparent; transform: translateY(16px);"></div> <div style=" background-color: #F4F4F4; flex-grow: 0; height: 12px; width: 16px; transform: translateY(-4px);"></div> <div style=" width: 0; height: 0; border-top: 8px solid #F4F4F4; border-left: 8px solid transparent; transform: translateY(-4px) translateX(8px);"></div></div></div> <div style="display: flex; flex-direction: column; flex-grow: 1; justify-content: center; margin-bottom: 24px;"> <div style=" background-color: #F4F4F4; border-radius: 4px; flex-grow: 0; height: 14px; margin-bottom: 6px; width: 224px;"></div> <div style=" background-color: #F4F4F4; border-radius: 4px; flex-grow: 0; height: 14px; width: 144px;"></div></div></a><p style=" color:#c9c8cd; font-family:Arial,sans-serif; font-size:14px; line-height:17px; margin-bottom:0; margin-top:8px; overflow:hidden; padding:8px 0 7px; text-align:center; text-overflow:ellipsis; white-space:nowrap;"><a href="https://www.instagram.com/p/CL5NvDnBqpu/?utm_source=ig_embed&amp;utm_campaign=loading" style=" color:#c9c8cd; font-family:Arial,sans-serif; font-size:14px; font-style:normal; font-weight:normal; line-height:17px; text-decoration:none;" target="_blank">A post shared by CheesePie1313 (@cheesepie1313)</a></p></div></blockquote> <script async src="//www.instagram.com/embed.js"></script>
<br>

After creating and posting this masterpiece, it didn't take me long to think "what if I turned this into a game?" My ability to paint high resolution sprites and create highly detailed 3D models are pretty lacking at the moment. I like to collaborate with actual artists for games that have those kinds of things. But with voxel art, I'm able to create something half decent myself, so I felt this could be a fun solo project to take on.

## Physics

Now that I had an art direction, the next step was to prove out the gameplay. So I hopped into Unity and spawned in a sphere and a plane.

I might go into this in more detail in a later blog post, but getting the physics of the sphere to behave like an actual golf ball was a bit of a process. The default physics Unity has felt pretty bad. I looked for some tutorials online for how other developers have implemented golf ball physics, but none of them felt right either. It took a lot of playing around with the Unity physics settings and overriding how collisions are processed. In the end I got something that could still use a lot of improvements, but at least feels decent.

![Golf Ball Physics GIF](/assets/2021-04-21-island-mini-putt-devlog-1/physics.gif)

## 3D Modeling

Having both an art direction and golf ball physics, it was time to put them together. However there was a problem, I can't directly use the model that I exported from MagicaVoxel for a pretty obvious reason. A golf ball can't roll up steps...

![Voxel Steps](/assets/2021-04-21-island-mini-putt-devlog-1/steps.png)

Luckily I had gone through some basic [Blender](https://www.blender.org/) tutorials in the past, so I had some base knowledge to start editing the meshes exported by MagicaVoxel.

![Blender Screenshot](/assets/2021-04-21-island-mini-putt-devlog-1/blender.png)

It took sometime to get the shape the way I wanted, especially since I needed to make sure a lot of the vertices were still aligned with the voxel grid. Changing the vertices also messed up the normals in the original mesh, so I had to find the right settings to clean up the mesh and align those normals so the shading looked right. After all that I ended up with a smooth ramp and playable level! #RampHype!

![Smooth Ramps](/assets/2021-04-21-island-mini-putt-devlog-1/model.png)

## Version 0.1

At this point I was pretty happy with how the game was going and was ready to start putting a plan together for how I would turn this prototype into a full game. I came up with a few milestones and started working away on the first one, Version 0.1. This first milestone included creating 3 playable holes to test out the physics of different elements, adding touch and mouse + keyboard controls so I could play the game on android and in a web browser and finally a basic menu system for navigating between the holes.

Those tasks for the first milestone ended up going pretty smoothly. Having made many menu systems before in Unity, that was quick to set up. The touch controls specifically were a little rough to get right due to having to make android builds between iterations, but overall was not too bad. Most importantly I worked on the gameplay logic, which I implemented mostly using state machines. I even snuck in an updated shader for the water, to make it really feel like the ocean. Once I had that all set up, it started to feel like an actual game.

![Version 0.1](/assets/2021-04-21-island-mini-putt-devlog-1/version0-1.png)

And the best part is, I exported it as a WebGL game so you can try it out right now in your web browser!

<h1 style="text-align: center;"><a href="/assets/island-mini-putt/index.html" target="_blank">Click Here to Play!</a></h1>
<p style="text-align: center;">(The link may be to a newer version of the game depending on when you're reading this)</p>

## Inspiration
Since I started developing this game I explored what other mini golf games are out there. One I've been playing a lot lately is [Golf Battle](https://play.google.com/store/apps/details?id=games.onebutton.golfbattle). It's a pretty fun multiplayer mini golf game where you battle other players to see who can get the lower score over a few holes. The holes have some cool obstacles and a good variety of environments. Now a lot of these things are out of scope for my game (especially multiplayer), but I did get some ideas for the aiming mechanics from this game.

Another game that I've been playing a lot of is [Walkabout Mini golf](https://www.mightycoconut.com/minigolf). It's a VR game that I've been playing on my Oculus Quest 2. Out of all the VR games I've played it's definitely the game that feels the most like it's real life counter part. I've had a lot of good times playing with friends and family in person and with the online multiplayer. Now turning my game into a VR game would be really cool, but that is definitely out of scope at least for version 1.0. Walkabout Mini Golf though has given me some good ideas for hole designs. The game also has a hard mode of all the courses in the game, which gave me the idea for doing hard modes of my golf holes too. If you have a VR headset I highly recommend trying that game out, it's personally my favorite VR game (with Beat Saber as a close second of course).

## The Future
There's a good amount of improvements that I want to make before I release a version 1.0. Most importantly is creating a full 18 hole course along with a hard version of every hole. On top of that there's some basic features like getting a regular play mode in the game with a viewable score card. And finally I want to make some further improvements to things like the physics, the UI and add transitions between holes.

So that's where I'm at now, this is the [link]("/assets/island-mini-putt/index.html") again if you wanna play. I'll probably have some more devlogs in a little, I've been using a lot of state machines for this game so that might be what the next one will be about. Thanks for reading, hope you enjoy the game in it's early form and stay tuned for more!

![Hole In One GIF](/assets/2021-04-21-island-mini-putt-devlog-1/hole_in_one.gif)
