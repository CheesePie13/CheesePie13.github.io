---
layout: post
slug: csharp-delegates-memory-static-methods
title: "C# Delegates and Memory Allocations: Static Methods"
image: /assets/2021-02-16-csharp-delegates-memory-static-methods/main.png 
date: 2021-02-16 12:00:00 -0500
author: Matt Gibson
tags: Unity CSharp Delegates
github_issue_id: 8
---

In this blog post I look into memory allocations associated with creating delegates from static methods.

<!--more-->

If you haven't already make sure you read the introduction post [here](/blog/csharp-delegates-memory-intro), it goes over the basics of the data stored in a delegate and how I'll be testing the memory allocations. Also I recommend reading through my first delegate test with [class methods](/blog/csharp-delegates-memory-class-methods), I go into more detail on that test then I will with this one.

## The Test
```csharp
using System;

public class Test {
	private static int AddOne(int x) {
		return x + 1;
	}

	public Func<int, int> GetDelegate() {
		return AddOne;
	}
}
```

Keeping it simple again, I created a class with a single private static method that is returned as a delegate from the `GetDelegate()` method.

## The Results
### Memory Allocations
Using Rider's memory debugger, it shows that the first time the `GetDelegate()` method is called, one delegate is allocated. On every consecutive call a delegate is allocated as well. This is the exact same as the class method delegates, but if you understand how delegates work you may have noticed that there is room for optimization for this specific case. Let's look at the IL code.

### Compiled IL Code
<details>
  <summary>Click to view the IL Assembly (or skip to the IL Pseudo Code below)</summary>

  ```
  .class public auto ansi beforefieldinit
    Test
      extends [mscorlib]System.Object
  {

    .method private hidebysig static int32
      AddOne(
        int32 x
      ) cil managed
    {
      .maxstack 8

      // [5 3 - 5 16]
      IL_0000: ldarg.0      // x
      IL_0001: ldc.i4.1
      IL_0002: add
      IL_0003: ret

    } // end of method Test::AddOne

    .method public hidebysig instance class [mscorlib]System.Func`2<int32, int32>
      GetDelegate() cil managed
    {
      .maxstack 8

      // [9 3 - 9 17]
      IL_0000: ldnull
      IL_0001: ldftn        int32 Test::AddOne(int32)
      IL_0007: newobj       instance void class [mscorlib]System.Func`2<int32, int32>::.ctor(object, native int)
      IL_000c: ret

    } // end of method Test::GetDelegate

    .method public hidebysig specialname rtspecialname instance void
      .ctor() cil managed
    {
      .maxstack 8

      IL_0000: ldarg.0      // this
      IL_0001: call         instance void [mscorlib]System.Object::.ctor()
      IL_0006: ret

    } // end of method Test::.ctor
  } // end of class Test
  ```
</details>
<br/><br/>

### Compiled IL Pseudo Code
```c#
public class Test {
	private static int AddOne(int x) {
		return x + 1;
	}

	public Func<int, int> GetDelegate() {
		return new Func<int, int>(
			target: null,
			funcPtr: AddOne
		);
	}
}
```

As you can see, the IL code shows that a delegate is being created every time the `GetDelegate()` function is called. What you may have noticed as well is that the input to the `Func` constructor is constant. Since delegates are immutable, there is no need to create and return the exact same delegate object every time. There is only the need for one instance of the delegate but the compiler does not make this optimization. Let's look at how we can make this optimization ourselves.

## Cached Static Method Delegates
```csharp
using System;

public class Test {
	private static readonly Func<int, int> addOneDelegate = AddOne;

	private static int AddOne(int x) {
		return x + 1;
	}

	public Func<int, int> GetDelegate() {
		return addOneDelegate;
	}
}
```

In this new class instead of directly returning a delegate of the static method, we create a cached version of the delegate when the program starts and return that delegate each time `GetDelegate()` is called.

### Memory Allocations
With this new class there is only one delegate allocation needed for the entirety of the program.

### Compiled IL Code
<details>
  <summary>Click to view the IL Assembly (or skip to the IL Pseudo Code below)</summary>

  ```
  .class public auto ansi beforefieldinit
    Test
      extends [mscorlib]System.Object
  {

    .field private static initonly class [mscorlib]System.Func`2<int32, int32> addOneDelegate

    .method private hidebysig static int32
      AddOne(
        int32 x
      ) cil managed
    {
      .maxstack 8

      // [7 3 - 7 16]
      IL_0000: ldarg.0      // x
      IL_0001: ldc.i4.1
      IL_0002: add
      IL_0003: ret

    } // end of method Test::AddOne

    .method public hidebysig instance class [mscorlib]System.Func`2<int32, int32>
      GetDelegate() cil managed
    {
      .maxstack 8

      // [11 3 - 11 25]
      IL_0000: ldsfld       class [mscorlib]System.Func`2<int32, int32> Test::addOneDelegate
      IL_0005: ret

    } // end of method Test::GetDelegate

    .method public hidebysig specialname rtspecialname instance void
      .ctor() cil managed
    {
      .maxstack 8

      IL_0000: ldarg.0      // this
      IL_0001: call         instance void [mscorlib]System.Object::.ctor()
      IL_0006: ret

    } // end of method Test::.ctor

    .method private hidebysig static specialname rtspecialname void
      .cctor() cil managed
    {
      .maxstack 8

      // [4 2 - 4 65]
      IL_0000: ldnull
      IL_0001: ldftn        int32 Test::AddOne(int32)
      IL_0007: newobj       instance void class [mscorlib]System.Func`2<int32, int32>::.ctor(object, native int)
      IL_000c: stsfld       class [mscorlib]System.Func`2<int32, int32> Test::addOneDelegate
      IL_0011: ret

    } // end of method Test::.cctor
  } // end of class Test
  ```
</details>
<br/><br/>

### Compiled IL Pseudo Code

```c#
public class Test {
	private static readonly Func<int, int> AddOneDelegate = new Func<int, int>(
		target: null,
		funcPtr: AddOne
	);

	private static int AddOne(int x) {
		return x + 1;
	}

	public Func<int, int> GetDelegate() {
		return AddOneDelegate;
	}
}
```

You can see in the IL Pseudo Code that a new `Func` delegate is created only once now, not every time I call `GetDelegate()`.

## Conclusion
The lesson we can take away from this is that when using a delegate from a static method the compiler will not make the optimization for us. This is fine if we use a static method once to subscribe to an event or as a callback, but if we use it 2 or more times you can save on memory by caching the delegate for the static method in a static variable.

Thanks for reading this post! I was a bit surprised myself to learn that this optimization wasn't made automatically, especially now knowing how lambda delegates work. I'll be talking about those lambdas in my next post, so see you then!
