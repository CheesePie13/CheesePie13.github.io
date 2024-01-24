---
layout: post
slug: unreal-replication-settings
title: "Unreal Replication: Update Frequency, Relevancy and Priority"
date: 2024-01-24 12:00:00 -0500
author: Matt Gibson
tags: Unreal Networking
---

Replication is the process of synchronizing state between the server and clients to create the illusion that players are all participating in one consistent game world. In Unreal, replication occurs at the level of Actors, and there are three primary settings that control how an Actor replicates: Update Frequency, Relevancy and Priority.

<!--more-->

> Note: This post is only related to the built-in replication system, the Replication Graph plugin and the new Iris Replication System handle things differently than described here.

To properly understand these settings, you'll want to know the general process Unreal undergoes on the server every tick to determine which Actors to replicate to which client connections. The process begins with the list of all the Actors that have replication enabled. Through the replication process, this list is filtered down to the subset of Actors that actually replicate to a connection. The first filter in this process is based on the Update Frequency setting.

## Update Frequency
> How often an Actor replicates.

If your level has a lot of replicating Actors in it, having each of them send replication updates every tick could potentially saturate the bandwidth of the connection to the clients. Lowering the Update Frequency of Actors can prevent them from sending replication updates every frame, which will help mitigate this issue.

The `AActor::NetUpdateFrequency` property is used to determine how often the Actor replicates per second. For example, if the value is set to 5, the Actor will attempt replication only after at least 200ms have elapsed since the last replication attempt. If this time interval hasn't elapsed, the Actor will be filtered out and not considered for replication during that tick.

As this is the first step in the Actor filtering process, more Actors being filtered out at this stage means fewer Actors need processing in subsequent steps. So in addition to reducing the bandwidth usage, turning down the Update Frequency on Actors is an ideal strategy for reducing the CPU usage of the replication system each tick on the server.

However, reducing the Update Frequency can make your game feel less responsive. Longer intervals between replication updates increase the potential latency from the time a state change happens on the server to the time it appears on a client. Additionally, it increases the chances for predicted client state to become out of sync with the server, requiring larger and more noticeable corrections. It's a good idea to give Actors that the player is directly interacting with a higher Update Frequency than other. You can even dynamically change the `AActor::NetUpdateFrequency` property at run-time based on the player's likelihood of interacting with that Actor at a given time. The goal is to strike the right balance between bandwidth limitations, CPU performance and responsiveness. 

As an additional note, there is the Adaptive Net Update Frequency setting, which can be enabled with the `net.UseAdaptiveNetUpdateFrequency` console variable. Once enabled, the `AActor::NetUpdateFrequency` property is treated as the maximum update frequency and the `AActor::MinNetUpdateFrequency` property is the minimum update frequency. The replication system dynamically adjusts the Update Frequency within this range depending on how often the Actor's replicated properties actually change. This feature helps reduce the processing of Actors that don't end up having any state changes to replicate.

## Relevancy
> Should an Actor replicate to a connection.

After filtering out Actors based on their Update Frequency, the next step involves filtering out Actors based on Relevance. As opposed to Update Frequency, Relevancy is checked separately per client connection.

The relevancy of an Actor to a connection is determined using the `AActor::IsNetRelevantFor()` function. Typically, the client's PlayerController, Pawn and location are passed into the function and are used to determine relevance. It's a virtual function that can be overridden and optimized for your game's use case, but the base implementation accounts for various factors by default such as the Actor's owner, instigator and if the Actor is hidden.

One thing to note is that a client connection can have associated child connections. These child connections are used when a client has multiple viewports, enabling features such as local split-screen for online multiplayer. During the relevancy filtering, the Actor's relevancy is checked against the PlayerController and Pawn of the connection and all child connections. If any of them are relevant, the connection as a whole passes the relevancy filter.

There are a few additional settings related to relevancy:

### `AActor::bOnlyRelevantToOwner`

This property adds an additional check that makes the Actor only relevant to the connection that owns it. More specifically it checks if the `AActor::GetNetOwner()` is any of the connection's (or child connection's) PlayerController, Pawn or ViewTarget. The PlayerController is an example of a class that has this property enabled. It's the reason a PlayerControllers is only available on the server and the client it belongs to, not other clients. An important note is that you should avoid changing this property at runtime, if you do you'll need to manually close the Actor Channel assosiated with the Actor for the change to take effect.

### `AActor::bNetUseOwnerRelevancy`

This property is fairly simple, if it's enabled, the Actor will return the Relevance of its owner (`Actor->Owner->IsNetRelevantFor()`) as its own. It also makes the Actor use the Priority of its owner. Both of these behaviours depend on the default logic of the `AActor::IsNetRelevantFor()` and `AActor::GetNetPriority()` functions, so overriding them can break this property.

### `AActor::bAlwaysRelevant`

This property makes the Actor always relevant to all connections. It does depend on the logic in the `AActor::IsNetRelevantFor()` function, so overriding it could break this property.

![bAlwayRelevant Comment](/assets/2024-01-24-unreal-replication-settings/bAlwaysRelevant-comment.png)

There is a comment above the property in the engine code stating that it will override the `AActor::bOnlyRelevantToOwner` property, that is in fact **not** true. This is because when `AActor::bOnlyRelevantToOwner` is enabled, it adds an additional independent Actor filter that will apply regardless of whether `AActor::bAlwaysRelevant` is enabled.

## Priority
> If bandwidth is limited, which Actors should replicate first.

Now, with the final list of Actors we want to replicate for a connection, we have to take into account the bandwidth limit of the connection. Due to this limit, we may only be able to replicate a portion of the Actors before the connection becomes saturated. The Actors that are chosen to replicate are the ones with the highest Priority. The Actors that don't make the cut are marked so that they bypass the Update Frequency filtering on the next tick.

The Priority of an Actor is obtained by calling it's `AActor::GetNetPriority()` function. The function takes in a bunch of parameters which it can use to determine the Actor's priority. It is also a virtual function that can be overwritten for your use case. The default implementation primarily uses the position of the Actor relative to the player's view to determine the priority.

## Simplified Overview

Now that we've gone over the settings, here is a very simplified description of the replication process to sum everything up. This process runs every tick on the server to determine which Actors are replicated to which connection:

- For all Actors that have replication enabled:
	- Filter out Actors based on their **Update Frequency** settings and how long it has been since their last replication
- For each connection:
	- Filter out Actors that are not **Relevant** to the connection or any child connections
	- If `AActor::bOnlyRelevantToOwner` is true, then filter out the actor based on their net owner
	- Sort the Actors based on their **Priority** from highest to lowest
	- Send the replication data for Actors in order until the connection is saturated
		- Actors that don't make the cut will bypass the Update Frequency filter next tick

## Conclusion

Now, you should have a good understanding of the Update Frequency, Relevancy and Priority settings in Unreal's built-in replication system. Of course, this was a very simplified explanation of what actually happens, but it should be good enough to make some informed decisions to improve replication in your game. I've omitted details such as dormancy and connection throttling, and the whole replication process itself is much more intricate. If you want to grasp the whole picture, follow the execution flow of the `UNetDriver::ServerReplicateActors()` function, that's where all the magic happens!