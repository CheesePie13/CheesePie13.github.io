---
layout: post
slug: csharp-delegates-memory-local-functions
title: "C# Delegates and Memory Allocations: Local Functions"
image: /assets/2021-02-18-csharp-delegates-memory-local-functions/main.png 
date: 2021-02-18 12:00:00 -0500
author: Matt Gibson
tags: Unity CSharp Delegates
github_issue_id: 10
---

In this blog post I look into memory allocations associated with creating delegates from local functions.

<!--more-->

If you haven't already make sure you read the introduction post [here](/blog/csharp-delegates-memory-intro), it goes over the basics of the data stored in a delegate and how I'll be testing the memory allocations. Also I recommend reading through my first delegate test with [class methods](/blog/csharp-delegates-memory-class-methods), I go into more detail on that test then I will with this one.

This blog post is very similar to the [lambda post](/blog/csharp-delegates-memory-class-lambdas) except for the final case, feel free to skip down to that one.

## Local Functions with Class Member References
```csharp
using System;

public class Test {
	private int num = 1;

	public Func<int, int> GetDelegate() {
		int AddOne(int x) {
			return x + num;
		}

		return AddOne;
	}
}
```

This one is pretty much the same as the lambda case with class member references. Here we define a local function that references the `num` member variable.

### Memory Allocations
Each call to `GetDelegate()` allocates one delegate object.


### Compiled IL Code
<details>
  <summary>Click to view the IL Assembly (or skip to the IL Pseudo Code below)</summary>

  ```
  .class public auto ansi beforefieldinit
    Test
      extends [mscorlib]System.Object
  {

    .field private int32 num

    .method public hidebysig instance class [mscorlib]System.Func`2<int32, int32>
      GetDelegate() cil managed
    {
      .maxstack 8

      // [11 3 - 11 17]
      IL_0000: ldarg.0      // this
      IL_0001: ldftn        instance int32 Test::'<GetDelegate>g__AddOne|1_0'(int32)
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

    .method private hidebysig instance int32
      '<GetDelegate>g__AddOne|1_0'(
        int32 x
      ) cil managed
    {
      .custom instance void [mscorlib]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
        = (01 00 00 00 )
      .maxstack 8

      // [8 4 - 8 19]
      IL_0000: ldarg.1      // x
      IL_0001: ldarg.0      // this
      IL_0002: ldfld        int32 Test::num
      IL_0007: add
      IL_0008: ret

    } // end of method Test::'<GetDelegate>g__AddOne|1_0'
  } // end of class Test
  ```
</details>
<br/><br/>

### Compiled IL Pseudo Code
```csharp
public class Test {
	private int num = 1;

	private int AddOne(int x) {
		return x + num;
	}

	public Func<int, int> GetDelegate() {
		return new Func<int, int>(
			target: this,
			funcPtr: AddOne
		);
	}
}
```

The compiled version behaves exactly the same as with the lambdas. The local function is turned into a class method and a delegate is created every time with the `Test` class instance as the target.

## Local Functions with Local Scope References
```csharp
using System;

public class Test {
	public Func<int, int> GetDelegate() {
		int num = 1;

		int AddOne(int x) {
			return x + num;
		}

		return AddOne;
	}
}
```

This one as well is the same as the lambda with local scope references. Here we have a `num` variable in the same scope as the local function.

### Memory Allocations
Every time `GetDelegate()` is called 2 allocations are made, one for the delegate and one for the closure object.

### Compiled IL Code
<details>
  <summary>Click to view the IL Assembly (or skip to the IL Pseudo Code below)</summary>

  ```
  .class public auto ansi beforefieldinit
    Test
      extends [mscorlib]System.Object
  {

    .class nested private sealed auto ansi beforefieldinit
      '<>c__DisplayClass0_0'
        extends [mscorlib]System.Object
    {
      .custom instance void [mscorlib]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
        = (01 00 00 00 )

      .field public int32 num

      .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
      {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [mscorlib]System.Object::.ctor()
        IL_0006: ret

      } // end of method '<>c__DisplayClass0_0'::.ctor

      .method assembly hidebysig instance int32
        '<GetDelegate>g__AddOne|0'(
          int32 x
        ) cil managed
      {
        .maxstack 8

        // [8 4 - 8 19]
        IL_0000: ldarg.1      // x
        IL_0001: ldarg.0      // this
        IL_0002: ldfld        int32 Test/'<>c__DisplayClass0_0'::num
        IL_0007: add
        IL_0008: ret

      } // end of method '<>c__DisplayClass0_0'::'<GetDelegate>g__AddOne|0'
    } // end of class '<>c__DisplayClass0_0'

    .method public hidebysig instance class [mscorlib]System.Func`2<int32, int32>
      GetDelegate() cil managed
    {
      .maxstack 8

      IL_0000: newobj       instance void Test/'<>c__DisplayClass0_0'::.ctor()

      // [5 3 - 5 15]
      IL_0005: dup
      IL_0006: ldc.i4.1
      IL_0007: stfld        int32 Test/'<>c__DisplayClass0_0'::num

      // [11 3 - 11 17]
      IL_000c: ldftn        instance int32 Test/'<>c__DisplayClass0_0'::'<GetDelegate>g__AddOne|0'(int32)
      IL_0012: newobj       instance void class [mscorlib]System.Func`2<int32, int32>::.ctor(object, native int)
      IL_0017: ret

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
```csharp
using System;

public class Test {
	private class Closure {
		public int num;

		public int AddOne(int x) {
			return x + num;
		}
	}

	public Func<int, int> GetDelegate() {
		Closure closure = new Closure();
		closure.num = 1;
		return new Func<int, int>(
			target: closure,
			funcPtr: Closure.AddOne
		);
	}
}
```

As with the lambda case, the compiler generates a closure class to capture the `num` variable and `AddOne` function. A new closure is allocated and passed in as the target to the new delegate every time `GetDelegate()` is called.


## Local Functions with No External References
```csharp
using System;

public class Test {
	public Func<int, int> GetDelegate() {
		int AddOne(int x) {
			return x + 1;
		}

		return AddOne;
	}
}
```

The test for this case is once again very similar to the lambda case. However, the results are not quite the same.

### Memory Allocations
On the first call a closure and delegate are allocated and on consecutive calls only a delegate is allocated. For the lambda case, no allocation were needed for consecutive calls, what's going on here? Let's look at the IL.


### Compiled IL Code
<details>
  <summary>Click to view the IL Assembly (or skip to the IL Pseudo Code below)</summary>

  ```
  .class public auto ansi beforefieldinit
    Test
      extends [mscorlib]System.Object
  {

    .class nested private sealed auto ansi serializable beforefieldinit
      '<>c'
        extends [mscorlib]System.Object
    {
      .custom instance void [mscorlib]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
        = (01 00 00 00 )

      .field public static initonly class Test/'<>c' '<>9'

      .method private hidebysig static specialname rtspecialname void
        .cctor() cil managed
      {
        .maxstack 8

        IL_0000: newobj       instance void Test/'<>c'::.ctor()
        IL_0005: stsfld       class Test/'<>c' Test/'<>c'::'<>9'
        IL_000a: ret

      } // end of method '<>c'::.cctor

      .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
      {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [mscorlib]System.Object::.ctor()
        IL_0006: ret

      } // end of method '<>c'::.ctor

      .method assembly hidebysig instance int32
        '<GetDelegate>g__AddOne|0_0'(
          int32 x
        ) cil managed
      {
        .maxstack 8

        // [6 4 - 6 17]
        IL_0000: ldarg.1      // x
        IL_0001: ldc.i4.1
        IL_0002: add
        IL_0003: ret

      } // end of method '<>c'::'<GetDelegate>g__AddOne|0_0'
    } // end of class '<>c'

    .method public hidebysig instance class [mscorlib]System.Func`2<int32, int32>
      GetDelegate() cil managed
    {
      .maxstack 8

      // [9 3 - 9 17]
      IL_0000: ldsfld       class Test/'<>c' Test/'<>c'::'<>9'
      IL_0005: ldftn        instance int32 Test/'<>c'::'<GetDelegate>g__AddOne|0_0'(int32)
      IL_000b: newobj       instance void class [mscorlib]System.Func`2<int32, int32>::.ctor(object, native int)
      IL_0010: ret

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
```csharp
public class Test {
	private class Closure {
		public static readonly Closure instance = new Closure();
		public int AddOne(int x) {
			return x + 1;
		}
	}

	public Func<int, int> GetDelegate() {
		return return new Func<int, int>(
			target: Closure.instance,
			funcPtr: Closure.AddOne
		);
	}
}
```

As you can see, like the lambda case the closure instance is created once. However, the delegate ends up being created every time `GetDelegate()` is called. This is what causes that extra allocation on consecutive calls.

## Conclusion
The main take away here is that local functions behave the same as lambdas except in the case that they make no external references. In that case only the lambdas make the optimization of caching the delegates, the local function only caches the closure object. This is very interesting and I can't see a reason for it. Local functions seem pretty similar to lambdas except for the fact that they can reference themselves inside the body. Maybe there is something about that that makes the optimization harder to implement, or maybe it was just an oversight? It would probably take a look into the compiler's source code to find out, might be a good idea for another blog post 🤔

Anyways thanks for reading, I'll see you in my next and last post of this series that will summarize all the learnings I got from doing these tests!
