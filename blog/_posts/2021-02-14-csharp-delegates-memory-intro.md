---
layout: post
slug: csharp-delegates-memory-intro
title: "C# Delegates and Memory Allocations: Introduction"
date: 2021-02-14 12:00:00 -0500
author: Matt Gibson
tags: Unity CSharp Delegates
github_issue_id: 6
---

Knowing how memory is managed under the hood is really important for writing high performance C# code. In this blog post series I dive into how memory is used in relation to delegates and how creating delegates from different types of functions can affect memory allocations.

<!--more-->

## What is a delegate?
If you haven't used delegates before you can read about how they are used [here](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/). A delegate can basically be thought of as a class that holds two main pieces of data: A function pointer and a target object. You can get the function pointer indirectly by using the `Delegate.Method` property and can get the target object by using the `Delegate.Target` property. The delegate class also has other properties for Multicast Delegates but they aren't relevant to the memory tests I'll be doing.

The "function pointer" of a delegate is used to store the location of the function in the compiled code. When the delegate is invoked, the program jumps to this location in the code to run the function.

The "target object" of a delegate is the object that is passed in as the first hidden parameter of a function. You may not have known that there was a hidden parameter passed into many of your functions but you have probably used it before. You can refer to it directly using the `this` keyword and is used implicitly whenever you reference a member variable. Some functions don't support the hidden `this` parameter (such as static functions), for these the target object is null.

### Delegates, Actions and Funcs
I often use the `Action` and `Func` generic types for delegates (the tests in this series primarily use `Func`). Using these types can make the code easier to read since the delegate's parameter and return types are known without having to look up the delegate declaration. However, you can't provide parameter names for Actions and Funcs, so delegates with a lot of parameters may be better suited for a custom delegate.

This is what the declarations for Actions and Funcs look like:
```c#
// These are the decompiled definitions of Action and Func
// Action
public delegate void Action();
public delegate void Action<in T>(T obj);
public delegate void Action<in T1, in T2>(T1 arg1, T2 arg2);
// ...

// Func
public delegate TResult Func<out TResult>();
public delegate TResult Func<in T, out TResult>(T arg);
public delegate TResult Func<in T1, in T2, out TResult>(T1 arg1, T2 arg2);
// ...
```

## Testing
To test the allocations of each kind of delegate I will be writing simple `Test` classes that return a delegate from a `GetDelegate()` method. Each test will create a delegate from a different kind of function.

### Memory Allocations
I use a program called [Rider](https://www.jetbrains.com/rider/) as my IDE for Unity, it has a built in feature to view memory allocations that occurred between statements or breakpoints. Unfortunately it doesn't work for Unity projects but it does with regular C# projects, so I'll be using a C# console application to do the tests.

### Compiled Code
To help understand why memory allocations are being made, I'll be looking at the Intermediate Language (IL) that is generated by the compiler. I'll be converting the IL back by hand to a kind of C# pseudo code so we can see what's happening. If you have Rider you can see the IL of your code by opening `Tools -> IL Viewer`.

### Setup
To run the tests I'll be using the following code to create an instance of the `Test` class and get the delegate multiple times. Each iteration of the loop I'll be checking how much memory is allocated. This should simulate passing around a delegate between classes or subscribing to events of another class.

```c#
Test test = new Test();
for (int i = 0; i < 5; i++) {
	Func<int, int> func = test.GetDelegate();
	int result = func(1);
}
```

## The Tests
I've broken up the tests in to multiple blog posts, I'll eventually link to them all here:
- Class Methods (Coming Soon)
- Static Methods (Coming Soon)
- Lambdas (Coming Soon)
- Local Functions (Coming Soon)
- Summary (Coming Soon)

If you got any questions or found any mistakes in these blog posts please comment them down below!