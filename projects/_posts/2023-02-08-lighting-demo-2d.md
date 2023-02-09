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

### Demo Details
- Written in rust and can be built to the following targets:
	- Windows (and probably Mac/Linux) using GLFW and OpenGL for rendering
	- WebAssembly using WebGL2 for rendering (this is the version running above)
- Rendering steps:
	1. A texture with the SDF data for the entire scene is created using the position and sizes of the shapes
	2. Using raymarching and the SDF texture, a path is traced from the light source to each pixel to see if the light ray is unobstructed
	3. The light intensity of the pixel is calculating using the inverse square of the distance (with an additional linear falloff so that the light doesn't extend infinitely)
