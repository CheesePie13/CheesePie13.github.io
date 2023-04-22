---
layout: post
slug: lighting-demo-2d
title: "2D Lighting Demo"
image: /assets/lighting-demo-2d/main.png
date: 2023-02-08 12:00:00 -0500
author: Matt Gibson
tags: 
assets_folder: 
---

2D lighting using SDFs and raymarching.

<!--more-->

<canvas id="canvas" height="720" width="720"></canvas>
<script type="module">
	import init from "/assets/lighting-demo-2d/wasm.js";
	await init();
</script>

<h1 style="text-align: center;">Click and drag above to manually control the light!</h1>

### Implementation Details
- Written in Rust and can be built for the following platforms:
	- Windows using GLFW and OpenGL for rendering (Mac/Linux hasn't been tested)
	- WebAssembly using WebGL2 for rendering (this is the version running above)
- The light rendering process involves three main steps:
	1. A texture with the SDF data for the entire scene is created using the position and sizes of the shapes
	2. Using raymarching and the SDF texture, a path is traced from the light source to each pixel to see if the light ray is unobstructed
	3. The light intensity of the pixel is calculated using the inverse square of the distance to the light (an additional linear falloff is added so that the light doesn't extend infinitely)
- The text and buttons are rendered using my immediate mode GUI library, which is still in early development
