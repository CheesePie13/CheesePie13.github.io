---
layout: post
slug: csharp-delegates-memory-lambdas
title: "C# Delegates and Memory Allocations: Lambdas"
image: /assets/2021-02-17-csharp-delegates-memory-lambdas/main.png 
date: 2021-02-17 12:00:00 -0500
author: Matt Gibson
tags: Unity CSharp Delegates Lambdas
github_issue_id: 9
---

In this blog post I look into memory allocations associated with creating delegates from lambdas.

<!--more-->

If you haven't already make sure you read the introduction post [here](/blog/csharp-delegates-memory-intro), it goes over the basics of the data stored in a delegate and how I'll be testing the memory allocations. Also I recommend reading through my first delegate test with [class methods](/blog/csharp-delegates-memory-class-methods), I go into more detail on that test then I will with this one.

There are 3 main lambda cases that I will be testing in this post. Each one compiles into different IL code and has different memory allocation behaviours, so they are all worth testing.

## Lambdas with Class Member References
```csharp
using System;

public class Test {
	private int num = 1;

	public Func<int, int> GetDelegate() {
		return x => x + num;
	}
}
```

I'm starting off with the simplest case for lambdas, when a lambda includes a class member reference in its body. You can see in the test that the lambda returned from the `GetDelegate()` method references the `num` member of the `Test` class.

### Memory Allocations
Each time the `GetDelegate()` function is called one delegate is allocated. This is the same pattern we saw when creating a delegate from a class method. Looking at the IL we can see why this is the case.

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

      // [7 3 - 7 23]
      IL_0000: ldarg.0      // this
      IL_0001: ldftn        instance int32 Test::'<GetDelegate>b__1_0'(int32)
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
      '<GetDelegate>b__1_0'(
        int32 x
      ) cil managed
    {
      .custom instance void [mscorlib]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
        = (01 00 00 00 )
      .maxstack 8

      // [7 15 - 7 22]
      IL_0000: ldarg.1      // x
      IL_0001: ldarg.0      // this
      IL_0002: ldfld        int32 Test::num
      IL_0007: add
      IL_0008: ret

    } // end of method Test::'<GetDelegate>b__1_0'
  } // end of class Test
  ```
</details>
<br/><br/>

### Compiled IL Pseudo Code
```csharp
public class Test {
	private int num = 1;

	private int LambdaFunction(int x) {
		return x + num;
	}
	
	public Func<int, int> GetDelegate() {
		return new Func<int, int>(
			target: this,
			funcPtr: LambdaFunction
		);
	}
}
```

The compiler basically turns the lambda into a class method. Just like the case where a delegate is created from a class method, each time the `GetDelegate()` method is called a new delegate is created using the class instance as the target and the new class method as the function pointer.

This means that if you were to take the code from a class method and inline it as a lambda to create a delegate, it would have no affect on the amount of memory allocated.

## Lambdas with Local Scope References
```csharp
using System;

public class Test {
	public Func<int, int> GetDelegate() {
		int num = 1;
		return x => x + num;
	}
}
```

Now let's look at when the lambda references a variable in its local scope. As you can see we have a `num` variable that is used in the body of the lambda and is created in the same scope as the lambda. The case would also be the same if the variable was passed in as a parameter, for example: `GetDelegate(int num)`.

### Memory Allocations
In this case every time `GetDelegate()` is called, 2 allocations are made. One allocation is the delegate and the other is a closure object. We can see what this closure object is by looking at the IL.

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
        '<GetDelegate>b__0'(
          int32 x
        ) cil managed
      {
        .maxstack 8

        // [6 15 - 6 22]
        IL_0000: ldarg.1      // x
        IL_0001: ldarg.0      // this
        IL_0002: ldfld        int32 Test/'<>c__DisplayClass0_0'::num
        IL_0007: add
        IL_0008: ret

      } // end of method '<>c__DisplayClass0_0'::'<GetDelegate>b__0'
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

      // [6 3 - 6 23]
      IL_000c: ldftn        instance int32 Test/'<>c__DisplayClass0_0'::'<GetDelegate>b__0'(int32)
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
public class Test {
	private class Closure {
		public int num;

		public int LambdaFunction(int x) {
			return x + num;
		}
	}
	
	public Func<int, int> GetDelegate() {
		Closure closure = new Closure();
		closure.num = 1;
		return new Func<int, int>(
			target: closure,
			funcPtr: Closure.LambdaFunction
		);
	}
}
```

Looking at the IL we can see that the closure object that is allocated is actually a new class the compiler has generated for us. This `Closure` class (called `<>c__DisplayClass0_0` in the actual IL Assembly) is what is used as the target of the delegate and is used to store the value of the `num` variable. This is how lambdas are able to use variables from local scope even if the lambda is called after it exits that scope.

Since lambdas that use local variables need this extra closure class to hold the variables' values, the `GetDelegate()` method makes 2 allocations every time it's called. One for the closure and one for the delegate.

## Lambdas with No External References
```csharp
using System;

public class Test {
	public Func<int, int> GetDelegate() {
		return x => x + 1;
	}
}
```

Now on to the last kind of lambda, one that doesn't use any references to variables outside it's body. These lambdas are pure functions, although I believe the case would be the same if the lambdas referenced static data. This happens to look the simplest in terms of the test code, however the compiled code that is generated happens to be the most complex. That's why I saved it for last.

### Memory Allocations
This case is unique compared to the other kinds of delegates we've looked at so far. Similar to the previous test with lambdas that have local references, the memory allocated on the first call to the `GetDelegate()` method includes 1 delegate and 1 closure. However, every subsequent call actually makes no memory allocations! Let's dive into the IL to find out why.

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

      .field public static class [mscorlib]System.Func`2<int32, int32> '<>9__0_0'

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
        '<GetDelegate>b__0_0'(
          int32 x
        ) cil managed
      {
        .maxstack 8

        // [5 15 - 5 20]
        IL_0000: ldarg.1      // x
        IL_0001: ldc.i4.1
        IL_0002: add
        IL_0003: ret

      } // end of method '<>c'::'<GetDelegate>b__0_0'
    } // end of class '<>c'

    .method public hidebysig instance class [mscorlib]System.Func`2<int32, int32>
      GetDelegate() cil managed
    {
      .maxstack 8

      // [5 3 - 5 21]
      IL_0000: ldsfld       class [mscorlib]System.Func`2<int32, int32> Test/'<>c'::'<>9__0_0'
      IL_0005: dup
      IL_0006: brtrue.s     IL_001f
      IL_0008: pop
      IL_0009: ldsfld       class Test/'<>c' Test/'<>c'::'<>9'
      IL_000e: ldftn        instance int32 Test/'<>c'::'<GetDelegate>b__0_0'(int32)
      IL_0014: newobj       instance void class [mscorlib]System.Func`2<int32, int32>::.ctor(object, native int)
      IL_0019: dup
      IL_001a: stsfld       class [mscorlib]System.Func`2<int32, int32> Test/'<>c'::'<>9__0_0'
      IL_001f: ret

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
	private class Closure {
		public static readonly Closure instance = new Closure();
		public static Func<int, int> cachedDelegate = null;
		
		public int LambdaFunction(int x) {
			return x + 1;
		}
	}
	
	public Func<int, int> GetDelegate() {
		if (Closure.cachedDelegate != null) {
			return Closure.cachedDelegate;
		}

		Closure.cachedDelegate = new Func<int, int>(
			target: Closure.instance,
			funcPtr: ClosureLambdaFunction
		);
		return cachedDelegate;
	}
}
```

The compiler has generated a closure class but this time it contains 2 static variables: one for a closure and one for a delegate. The Closure's `instance` is created when the program starts and the `cachedDelegate` is created the first time the `GetDelegate()` method is called. You can see that the compiler added a check in the `GetDelegate()` method to check if the `cachedDelegate` exists. If it doesn't exist it creates one, otherwise it returns the existing one. This means that after the first time `GetDelegate()` is called it will no longer allocate any additional memory.

This is very interesting to me for 2 reasons. First, pure function lambdas have better memory savings then static methods if you use them multiple times. Why don't static methods get compiled with the same caching mechanism? It might be that the compiler assumes that lambdas are more likely to be used multiple times compared to static methods. It could also be that they just never looked into adding that optimization to static methods.

The second thing that's interesting to me is the fact that there is a Closure class at all. The closure class doesn't actually have any fields. In this case the `LambdaFunction()` could have easily been made static and then all you would need to do is pass `null` in as the target of the delegate. This would have removed the need for the Closure instance and would have save a memory allocation. My guess is that this was just an optimization they never got to.

## Conclusion
One of the reasons I set out to do these comparisons in the first place was that I like to use lambdas instead of class methods in some cases for subscribing to events or as callbacks. I wanted to see if I was causing extra memory allocations by using lambdas in these cases. It's good to know now that I am not, delegates from lambdas that reference only class variables have the same memory allocations as delegates from class methods. The memory allocations do increase if you reference local variables inside the lambda(because of the closure class) but if you do make local references it couldn't have been a class method anyway.

It's also good to know that using a lambda instead of a static method can decrease memory usage if it is used multiple times. You can get less memory allocations by manually caching a static method delegate, but by default lambdas win when used multiple times. (That extra unneeded closure allocation is annoying though).

Hope that was interesting to you, it sure was for me. I'll see you in my next post on local functions, that will be my last test case before the summary post wrapping up all the information I learned from these tests. See you then!
