---
layout: post
slug: tick-functions
title: "Why you should avoid Tick()/Update() functions"
date: 2024-01-12 12:00:00 -0500
author: Matt Gibson
tags: Unreal Unity DataOrientedDesign
---

In the game engines I'm most familiar with (Unreal and Unity) and many others, there is the common notion of a `Tick()` (or `Update()`) function. It makes sense, all games have a game-loop which runs logic each frame to update the game world. So providing an easy way for all the Components/Actors (I'll call them entities) in your game world to run some logic of their own each frame is only natural. This seems just fine on the surface but in my experience it can sometimes make logic a lot more complicated than it needs to be, and can be a cause of performance problems. Let me explain...

<!--more-->

## The Problems
When using tick functions, you don't control the context of the tick function call. You know it was called sometime during the frame, but relative to all the other entities in the game you don't know in which order it was called. This becomes a problem when entities depend on other entities (like when one entity sets up data in their tick for another entity to use in their tick). Engines usually give tools to get around this. These can include the ablilty to specify the order entity types tick in, specify a tick group the entity is a part of, or directly specify dependencies between entities. These tools can help solve the problem, but for me they make the code harder to follow since the settings for these tools aren't always near the tick function in the code, or aren't in the code at all.

Another problem comes up when you need to do multiple tick passes over the entities. In many situations, I want to perform one set of updates to some entities and then, after they have all been processed, perform another set of updates. The easiest way would be to perform the first update on a tick and then perform the second update on a tick the next frame, but that would introduce a frame of delay. In Unity you could do the first update in the `Update()` function and then the second set in the `LateUpdate()` function, that will work but it doesn't scale if I want to do more passes in one frame. Tick functions just don't work well for this use case.

And finally there is performance. Depending on the engine, there might be a significant enough overhead cost to calling the tick functions themselves. It might not be that noticeable with a small number of entities, but as more ticking entities are added to the game world it can add up. In addition to that, using this kind of framework that provides a tick function prevents you from using some data oriented design patterns. I'll talk more about this below.

## The Solution
When you need to work in a engine or framework that provides a tick function, the solution to all these problems is pretty simple. Create a manager entity. The manager keeps a reference to all the entities it owns, when the manager ticks, it just needs to use a simple for loop to loop over all the entities and update them.

**Old Logic (Pseudocode)**
```c#
class ChildEntity : Entity {
	void Tick() {
		// Update the entity
	}
}
```

**New Logic (Pseudocode)**
```c#
class ChildEntityManager : Entity {
	ChildEntity[] childEntities;

	void Tick() {
		foreach(ChildEntity childEntity in childEntities) {
			// Update the childEntity
		}
	}
}
```

Now you have complete control! You can loop over the entities in whatever order you want. You can make sure they are always called in the same order, you can sort the entities then loop over them in that order. You can also easily do multiple passes, no more Update and LateUpdate, you just loop over the entities twice. It's all explicit: the order, the dependencies, and any multiple passes are all described directly in the code in one place. And last but not least, you no longer have the performance overhead that exists between tick calls, because you are directly calling the logic yourself on your entities.

If you want to step this up to the next level, you can employ some data oriented design techniques. Instead of storing all the data/state on your entities, you can store their state in tightly packed arrays and process that data using for loops (this is great for cache performance). Then just apply any visual updates to your entities based on the updated state. If you want to go further down this path, this video about optimization in Unity is a good one: [Practical Optimizations by Jason Booth](https://www.youtube.com/watch?v=NAVbI1HIzCE)

## Conclusion

Hopefully you can see why in many circumstances avoiding tick functions can be beneficial. They can be perfectly fine for one off logic that is isolated to an entity. However, for logic that needs to run on many entities or entities that have a lot of interconnected logic and dependencies, try using a manager entity to reduce the code complexity and increase the performance.
