---
layout: post
slug: csharp-delegates-memory-summary
title: "C# Delegates and Memory Allocations: Summary"
date: 2021-02-19 12:00:00 -0500
author: Matt Gibson
tags: Unity CSharp Delegates
github_issue_id: 11
---

Knowing how memory is managed under the hood is really important for writing high performance C# code. In this blog post I summarize how creating delegates from different types of functions can affect memory allocations.

<!--more-->

Over my past several blog posts I explored and tested what memory allocations are made from creating delegates. If you want to read through the entire series you can start with the intro [here](/blog/csharp-delegates-memory-intro). I'm going to try and keep this final summary post mostly self contained as a resource to come back to when optimizing delegate usage.

# The Tests
I conducted multiple tests using Rider's memory debugger and a C# console application (using .NETFramework v4.8 and C# 7.3) to find out how much memory is allocated by a delegate in multiple scenarios. This included creating a delegate from a class method, static method, lambdas and local functions. For each one I recorded how much memory was allocated the first time the delegate is created and how much is allocated every time afterwards. You can see the results of all these tests in the table below.

# The Results
<table>
	<thead>
		<tr>
			<th rowspan="3">Delegate Type</th>
			<th colspan="4">Memory Allocations</th>
		</tr>
		<tr>
			<th colspan="2">Test Results</th>
			<th colspan="2">Theoretical Results</th>
		</tr>
		<tr>
			<th>First Call</th>
			<th>Consecutive Calls</th>
			<th>First Call</th>
			<th>Consecutive Calls</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><a href="/blog/csharp-delegates-memory-class-methods">Class Method</a></td>
			<td>delegate</td>
			<td>delegate</td>
			<td>delegate</td>
			<td>none</td>
		</tr>
		<tr>
			<td><a href="/blog/csharp-delegates-memory-static-methods">Static Method</a></td>
			<td>delegate</td>
			<td>delegate</td>
			<td>delegate</td>
			<td>none</td>
		</tr>
		<tr>
			<td><a href="/blog/csharp-delegates-memory-lambdas#lambdas-with-class-member-references">Lambda: Class Member References</a></td>
			<td>delegate</td>
			<td>delegate</td>
			<td>delegate</td>
			<td>none</td>
		</tr>
		<tr>
			<td><a href="/blog/csharp-delegates-memory-lambdas#lambdas-with-local-scope-references">Lambda: Local scope references</a></td>
			<td>delegate + closure</td>
			<td>delegate + closure</td>
			<td>delegate + closure</td>
			<td>delegate + closure</td>
		</tr>
		<tr>
			<td><a href="/blog/csharp-delegates-memory-lambdas#lambdas-with-no-external-references">Lambda: No external references</a></td>
			<td>delegate + closure</td>
			<td>none</td>
			<td>delegate</td>
			<td>none</td>
		</tr>
		<tr>
			<td><a href="/blog/csharp-delegates-memory-local-functions#local-functions-with-class-member-references">Local Function: Class Member References</a></td>
			<td>delegate</td>
			<td>delegate</td>
			<td>delegate</td>
			<td>none</td>
		</tr>
		<tr>
			<td><a href="/blog/csharp-delegates-memory-local-functions#local-functions-with-local-scope-references">Local Function: Local scope references</a></td>
			<td>delegate + closure</td>
			<td>delegate + closure</td>
			<td>delegate + closure</td>
			<td>delegate + closure</td>
		</tr>
		<tr>
			<td><a href="/blog/csharp-delegates-memory-local-functions#local-functions-with-no-external-references">Local Function: No external References</a></td>
			<td>delegate + closure</td>
			<td>delegate</td>
			<td>delegate</td>
			<td>none</td>
		</tr>
	</tbody>
</table>

With each delegate type there are 2 possible allocations that can be made. One for the delegate object that stores the pointer to the function that will be invoked as well as a target object. Another for a closure object, which is sometimes needed to capture local variables and is used as the delegate's target. The test results show which objects are allocated in each case. You can follow the link for each type of delegate to read the details of that test. I also go into the compiled code for each test to see why the allocations are being made.

The theoretical results in the table show what I believe could be the smallest amount of memory allocations needed for each type of delegate. You can achieve these theoretical results through manually changing the code to cache the delegates because, in most cases, the compiler won't do this on it's own. One reason the compiler might purposefully avoid doing these optimizations is that in the case where the delegate is only used once, it's actually less optimized. Caching the delegate requires added logic and an additional memory reference. If in the majority of cases delegates only end up being used once the optimizations for the multiple use case would make things worse.

## The Key Takeaways
These are the main learnings I got from doing the tests and analyzing the results.

### Class Methods vs Lambdas
If you have a lambda that only references class members outside it's body, it will have the same memory allocation characteristics as a class method.

```csharp
public class ClassMethodTest {
	private int num = 1;

	private int Add(int x) {
		return x + num;
	}

	// First call:        1 memory allocation
	// Consecutive calls: 1 memory allocation
	public Func<int, int> GetDelegateClassMethod() {
		return Add;
	}

	// First call:        1 memory allocation
	// Consecutive calls: 1 memory allocation
	public Func<int, int> GetDelegateLambda() {
		return x => x + num;
	}
}
```

In the above code calling both the `GetDelegateClassMethod()` method and `GetDelegateLambda()` will result in the same amount of memory allocations. This means if you rather inline a class method because it's only used in one location, this will have no negative impact on the amount of memory used.

### Pure Function Caching
If you are using a function that makes no external references (aka a pure function) as a delegate, the delegate will only be cached when the pure function is a lambda. If you have a pure static method or local function, the delegate will not be cached and extra allocations will be made when used multiple times.

```csharp
public class ClassMethodTest {
	private static int AddOneStatic(int x) {
		return x + 1;
	}

	// First call:        1 memory allocation
	// Consecutive calls: 1 memory allocation
	public Func<int, int> GetDelegateStaticMethod() {
		return AddOneStatic;
	}

	// First call:        2 memory allocations
	// Consecutive calls: 1 memory allocation
	public Func<int, int> GetDelegateLocalFunction() {
		int AddOne(int x) {
			return x + 1;
		}

		return AddOne;
	}

	// First call:        2 memory allocations
	// Consecutive calls: 0 memory allocations
	public Func<int, int> GetDelegateLambda() {
		return x => x + 1;
	}
}
```

If your code only ever creates one delegate from a pure function, a static method is the way to go. However, when ever you need to create multiple delegates from a pure function (for use as a callback or to subscribe to events) a lambda will save you allocations every time after its first use. That's because the compiler will automatically cache and re-used delegates from lambdas that don't make external references.

### Manually Cache Static and Class Methods
You can manually cache a delegate of a static or class method to avoid new memory allocations every time you use the delegate.

```csharp
public class CachingTest {
	private Func<int, int> addOneDelegate = AddOne;
	private int AddOne(int x) {
		return x + 1;
	}

	// First call:        1 memory allocation
	// Consecutive calls: 1 memory allocation
	public Func<int, int> GetDelegate() {
		return AddOne;
	}

	// First call:        1 memory allocation
	// Consecutive calls: 0 memory allocations
	public Func<int, int> GetDelegateCached() {
		return AddOneDelegate;
	}
}

public class StaticCachingTest {
	private static Func<int, int> addOneStaticDelegate = AddOneStatic;
	private static int AddOneStatic(int x) {
		return x + 1;
	}

	// First call:        1 memory allocation
	// Consecutive calls: 1 memory allocation
	public Func<int, int> GetDelegate() {
		return AddOneStatic;
	}

	// First call:        1 memory allocation
	// Consecutive calls: 0 memory allocations
	public Func<int, int> GetDelegateCached() {
		return AddOneStaticDelegate;
	}
}
```

Having a private member (static or class) to store the delegate of a method will allow you to use the delegate multiple times without needing new allocations. You don't need to worry about what is done with the delegate since it is immutable. For example `delegate += x => x + 1` does not mutate the `delegate`, it allocates a new one and leaves the original unchanged.

## Overall Conclusions
At the end of the day, having done all these tests, the conclusion I arrive at is pretty simple. Use whatever kinds of delegates you are the most comfortable with. Some programmers prefer having all there callbacks and event handlers as separate class and static methods, while other programmers prefer to use lambdas or local functions where they can to keep the code inline. The cases here don't show any delegate types as being clearly better for memory for all situations, and I don't think it's worth memorizing which kind of delegates are best in each situation. There is no defined way that the compiler needs to handle memory, so more compiler optimizations may be made in the future that would change these results.

What is most important above all is that when you get to the optimization stage of developing your app or game, that you profile your game to see what areas of the code are causing the worst performance and most memory allocations. If those areas involve delegates, especially when used repeatedly in a loop or every frame, you can use the learnings from this post to improve the memory usage and put less strain on the garbage collector. If that's not the case, it's best you spend your effort with those areas of your program that are actually causing the big problems.

Anyways thanks for reading, hope you learned something! If you got any comments or if you've found any mistakes please let me know below.
