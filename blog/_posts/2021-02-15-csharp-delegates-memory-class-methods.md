---
layout: post
slug: csharp-delegates-memory-class-methods
title: "C# Delegates and Memory Allocations: Class Methods"
image: /assets/2021-02-15-csharp-delegates-memory-class-methods/main.png 
date: 2021-02-15 12:00:00 -0500
author: Matt Gibson
tags: Unity CSharp Delegates
github_issue_id: 7
---

In this blog post I look into memory allocations associated with creating delegates from class methods.

<!--more-->

If you haven't already make sure you read the introduction post [here](/blog/csharp-delegates-memory-intro), it goes over the basics of the data stored in a delegate and how I'll be testing the memory allocations.

## The Test
Here is the test code I'll be using:

```csharp
using System;

public class Test {
	private int num = 1;

	private int Add(int x) {
		return x + num;
	}

	public Func<int, int> GetDelegate() {
		return Add;
	}
}
```

As you can see, I'm keeping it very simple: a class with 2 methods. The first one is an `Add()` method that simply adds the private member `num` to the value `x` that's passed in. This is the function that will become the delegate. The second method, `GetDelegate()`, is the method that will be called repeatedly as described in the introduction post. Each time it's called I'll be checking how much memory is allocated.

## The Results
### Memory Allocations
Using Rider's memory debugger I was able to find out that one delegate allocation is made every time the `GetDelegate()` method is called. It showed that it was 32 bytes per delegate however the size may vary.

What this means is that every time you create a delegate from a class method, whether you pass it to another class as a callback or you use that method to subscribe to an event, it will make a single memory allocation for the delegate.

### Compiled IL Code
To find out why this is the case, let's look at the compiled code. C# is compiled into a binary format called IL (Intermediate Language), we can convert this into a readable format called IL Assembly using a disassembler. Rider has an [IL Viewer](https://www.jetbrains.com/help/rider/Viewing_Intermediate_Language.html) feature built in, so I'll be using that.

IL Assembly logically works differently than C# code, I've learned enough about it to manually convert the IL into C# pseudo code for better understanding. If you know IL or want to learn, you can click bellow to view the IL Assembly.

<details>
  <summary>Click to view the IL Assembly</summary>

  ```
  .class public auto ansi beforefieldinit
    Test
      extends [mscorlib]System.Object
  {

    .field private int32 num

    .method private hidebysig instance int32
      Add(
        int32 x
      ) cil managed
    {
      .maxstack 8

      // [7 3 - 7 18]
      IL_0000: ldarg.1      // x
      IL_0001: ldarg.0      // this
      IL_0002: ldfld        int32 Test::num
      IL_0007: add
      IL_0008: ret

    } // end of method Test::Add

    .method public hidebysig instance class [mscorlib]System.Func`2<int32, int32>
      GetDelegate() cil managed
    {
      .maxstack 8

      // [11 3 - 11 14]
      IL_0000: ldarg.0      // this
      IL_0001: ldftn        instance int32 Test::Add(int32)
      IL_0007: newobj       instance void class [mscorlib]System.Func`2<int32, int32>::.ctor(object, native int)
      IL_000c: ret

    } // end of method Test::GetDelegate

    .method public hidebysig specialname rtspecialname instance void
      .ctor() cil managed
    {
      .maxstack 8

      // [4 2 - 4 22]
      IL_0000: ldarg.0      // this
      IL_0001: ldc.i4.1
      IL_0002: stfld        int32 Test::num
      IL_0007: ldarg.0      // this
      IL_0008: call         instance void [mscorlib]System.Object::.ctor()
      IL_000d: ret

    } // end of method Test::.ctor
  } // end of class Test

  ```
</details>
<br/><br/>

### Compiled IL Pseudo Code
Now here is the pseudo code I manually generated in hopes of explaining what the IL code does.

```c#
public class Test {
	private int num = 1;

	private int Add(int x) {
		return x + num;
	}

	public Func<int, int> GetDelegate() {
		return new Func<int, int>(
			target: this,
			functionPointer: Add
		);
	}
}
```

As you can see, each time the `GetDelegate()` method is called a new delegate object (`Func`) is allocated. The function pointer passed in to the delegate's constructor is obviously the `Add()` method, this is the location in the code that the program will jump to when the delegate is invoked. The function pointer is the same for every delegate created, the target parameter on the other hand can be different depending on how `GetDelegate()` is called. This is what makes the memory allocation necessary.

Passing in the `this` parameter as the target means that when the delegate is invoked, the `this` parameter inside the the `Add()` method will be the target. This is important especially because we refer to the `num` member variable inside the `Add()` method, we need to know which instance of the `Test` class to get `num` from.

Something to note from further testing is that even if the  `Add()` class method didn't refer to any member variables, `this` would still be passed in and the delegate allocation would still be needed every time.

## Conclusion
If you are creating delegates from class methods you should avoid creating them repeatedly. Each time you create one it will cause a new memory allocation. If you need to use a lot of delegates for the same class method and instance, you can try instead caching the delegate by creating one in the constructor and using that delegate instead. It is safe to use just one delegate object everywhere since delegates are immutable once created.

Here's an example of how you could cache the delegate:
```csharp
using System;

public class Test {
	private int num = 1;
	private Func<int, int> addDelegate;

	public Test() {
		addDelegate = Add;
	}

	private int Add(int x) {
		return x + num;
	}

	public Func<int, int> GetDelegate() {
		return addDelegate;
	}
}
```

Now you should have an understanding of how memory is allocated for a delegate that uses a class method. If you have any questions or comments please leave them below. In my next post I'll be testing how memory is allocated for a static method. See you then!
