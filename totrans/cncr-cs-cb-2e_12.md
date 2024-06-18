# Chapter 12\. Synchronization

When your application makes use of concurrency (as practically all .NET applications do), then you need to watch out for situations in which one piece of code needs to update data while other code needs to access the same data. Whenever this happens, you need to *synchronize* access to the data. The recipes in this chapter cover the most common types used to synchronize access. However, if you use the other recipes in this book appropriately, you’ll find that a lot of the more common synchronization is already done for you by the respective libraries. Before diving into the synchronization recipes, let’s take a closer look at some common situations where synchronization may or may not be required.

###### Tip

The synchronization explanations in this section are slightly simplified, but the conclusions are all correct.

There are two major types of synchronization: *communication* and *data protection*. Communication is used when one piece of code needs to notify another piece of code of some condition (e.g., a new message has arrived). I’ll cover communication more thoroughly in this chapter’s recipes; the remainder of this introduction discusses data protection.

You need to use synchronization to protect shared data when *all three* of these conditions are true:

*   Multiple pieces of code are running concurrently.

*   These pieces are accessing (reading or writing) the same data.

*   At least one piece of code is updating (writing) the data.

The reason for the first condition should be obvious; if your entire code runs from top to bottom and nothing ever happens concurrently, then you never have to worry about synchronization. This is the case for some simple Console applications, but the vast majority of .NET applications do use *some* kind of concurrency. The second condition means that if each piece of code has its own local data that it doesn’t *share*, then there’s no need for synchronization; the local data is never accessed from any other pieces of code. There’s also no need for synchronization if there is shared data but the data never changes, such as if the data is defined using immutable types. The third condition covers scenarios like configuration values and the like that are set at the beginning of the application and then never change. If the shared data is only read, then it doesn’t need synchronization; only data that is both *shared* and *updated* needs synchronization.

The purpose of data protection is to provide each piece of code with a consistent view of the data. If one piece of code is updating the data, then you can use synchronization to make those updates appear atomic to the rest of the system.

It takes some practice to learn when synchronization is necessary, so we’ll walk through a few examples before starting the recipes in this chapter. As our first example, consider the following code:

[PRE0]

If the `MyMethodAsync` method is called from a threadpool thread (e.g., from within `Task.Run`), then the lines of code accessing `value` may run on separate threadpool threads. But does it need synchronization? No, because none of them can be running at the same time. The method is asynchronous, but it’s also sequential (meaning it progresses one part at a time).

OK, let’s complicate the example a bit. This time we’ll run concurrent asynchronous code:

[PRE1]

This code above is starting three modifications that run concurrently. Does it need synchronization? It depends. If you know that the method is called from a GUI or ASP.NET context (or any context that only allows one piece of code to run at a time), synchronization won’t be necessary because when the actual `data` modification code runs, it runs at a different time than the other two `data` modifications. For example, if the preceding code is run in a GUI context, there’s only one UI thread that will execute each of the `data` modifications, so it *must* do them one at a time. So, if you know the context is a one-at-a-time context, then there’s no synchronization needed. However, if that same method is called from a threadpool thread (e.g., from `Task.Run`), then synchronization *would* be necessary. In that case, the three `data` modifications could run on separate threadpool threads and update `data.Value` simultaneously, so you would need to synchronize access to `data.Value`.

Now let’s consider one more wrinkle:

[PRE2]

Consider what happens if `ModifyValueAsync` is called multiple times concurrently. Even if it is called from a one-at-a-time context, the data member is shared between each invocation of `ModifyValueAsync`, and the value may change any time that method does an `await`. You may want to apply synchronization even in a one-at-a-time context if you want to avoid that kind of sharing. Put another way, to make it so that each call to `ModifyValueAsync` waits until all previous calls have completed, you’ll need to add synchronization. This is true even if the context ensures that only one thread is used for all the code (i.e., the UI thread). Synchronization in this scenario is a kind of *throttling* for asynchronous methods (see [Recipe 12.2](#recipe-async-locks)).

Let’s look at one more `async` example. You can use `Task.Run` to do what I call “simple parallelism”—a basic kind of parallel processing that doesn’t provide the efficiency and configurability that the true parallelism of `Parallel`/PLINQ does. The following code updates a shared value using simple parallelism:

[PRE3]

This code has three separate tasks running on the thread pool (via `Task.Run`), all modifying the same `value`. So, our synchronization conditions apply, and we certainly do need synchronization here. Note that we do need synchronization even though `value` is a local variable; it’s still *shared* between threads even though it’s local to the one method.

Moving on to true parallel code, let’s consider an example that uses the `Parallel` type:

[PRE4]

Since this code uses `Parallel`, we must assume the body of the parallel loop (`item => Trace.WriteLine(item)`) can be running on multiple threads. However, the body of the loop only reads from its own data; there’s no data sharing between threads here. The `Parallel` class divides the data among threads so that none of them has to share its data. Each thread running its loop body is independent from all the other threads running the same loop body. So, no synchronization of the preceding code is necessary.

Let’s look at an aggregation example similar to the one covered in [Recipe 4.2](ch04.html#recipe-parallel-aggregate):

[PRE5]

In this example, the code is again using multiple threads; this time, each thread starts with its local value initialized to 0 (`() => 0`), and for each input value processed by that thread, it adds the input value to its local value (`(item, state, localValue) => localValue + item`). Finally, all the local values are added to the return value (`localValue => { result += localValue; }`). The first two steps aren’t problematic because there’s nothing shared between threads; each thread’s local and input values are independent from all other threads’ local and input values. The final step is problematic, however; when each thread’s local value is added to the return value, this is a situation where there’s a shared variable (`result`) that is accessed by multiple threads and updated by all of them. So, you’d need to use synchronization in that final step (see [Recipe 12.1](#recipe-locks)).

The PLINQ, dataflow, and reactive libraries are very similar to the `Parallel` examples: as long as your code is just dealing with its own input, it doesn’t have to worry about synchronization. I find that if I use these libraries appropriately, there’s very little need for me to add synchronization to most of my code.

Lastly, let’s discuss collections. Remember that the three conditions requiring synchronization are *multiple pieces of code*, *shared data*, and *data updates*.

Immutable types are naturally threadsafe because they *cannot* change; it’s not possible to update an immutable collection, so no synchronization is necessary. For example, the following code doesn’t require synchronization because when each separate threadpool thread pushes a value onto the stack, it’s creating a new immutable stack with that value, leaving the original `stack` unchanged:

[PRE6]

When your code uses immutable collections, it’s common to have a shared “root” variable that is not itself immutable. In that case, you *do* have to use synchronization. In the following code, each thread pushes a value onto the stack (creating a new immutable stack) and then updates the shared root variable; the code *does* need synchronization to update the `stack` variable:

[PRE7]

Threadsafe collections (e.g., `ConcurrentDictionary`) are quite different. Unlike immutable collections, threadsafe collections can be updated. But they have all the synchronization they need built in, so you don’t have to worry about synchronizing collection changes. If the following code updated a `Dictionary` instead of a `ConcurrentDictionary`, it would need synchronization; but since it’s updating a `ConcurrentDictionary`, it doesn’t need synchronization:

[PRE8]

# 12.1 Blocking Locks

## Problem

You have some shared data and need to safely read and write it from multiple threads.

## Solution

The best solution for this situation is to use the `lock` statement. When a thread enters a lock, it’ll prevent any other threads from entering that lock until the lock is released:

[PRE9]

## Discussion

There are many other kinds of locks in the .NET framework, such as `Monitor`, `SpinLock`, and `ReaderWriterLockSlim`. In most applications, these lock types should almost never be used directly. In particular, it’s natural for developers to jump to `ReaderWriterLockSlim` when there is no need for that level of complexity. The basic `lock` statement handles 99% of cases quite well.

There are four important guidelines when using locks:

*   Restrict lock visibility.

*   Document what the lock protects.

*   Minimize code under lock.

*   Never execute arbitrary code while holding a lock.

First, you should strive to restrict lock visibility. The object used in the `lock` statement should be a private field and never should be exposed to any method outside the class. There’s usually at most one lock member per type; if you have more than one, consider refactoring that type into separate types. You *can* lock on any reference type, but I prefer to have a field specifically for use with the `lock` statement, as in the last example. If you do lock on another instance, be sure that it is private to your class; it should not have been passed in to the constructor or returned from a property getter. You should never `lock(this)` or lock on any instance of `Type` or `string`; these locks can cause deadlocks because they are accessible from other code.

Second, document what the lock protects. This step is easy to overlook when initially writing the code but becomes more important as the code grows in complexity.

Third, do your best to minimize the code that is executed while holding a lock. One thing to watch for is blocking calls; ideally, your code should never block while holding a lock.

Finally, do not ever call arbitrary code under lock. Arbitrary code can include raising events, invoking virtual methods, or invoking delegates. If you must execute arbitrary code, do so after the lock is released.

## See Also

[Recipe 12.2](#recipe-async-locks) covers `async`-compatible locks. The `lock` statement is not compatible with `await`.

[Recipe 12.3](#recipe-signals) covers signaling between threads. The `lock` statement is intended to protect shared data, not to send signals between threads.

[Recipe 12.5](#recipe-throttling) covers throttling, which is a generalization of locking. A lock can be thought of as throttling to one at a time.

# 12.2 Async Locks

## Problem

You have some shared data and need to safely read and write it from multiple code blocks, which may be using `await`.

## Solution

The .NET framework `SemaphoreSlim` type has been updated in .NET 4.5 to be compatible with `async`. Here’s how you can use it:

[PRE10]

You can also use the `AsyncLock` type from the `Nito.AsyncEx` library, which has a slightly more elegant API:

[PRE11]

## Discussion

The same guidelines from [Recipe 12.1](#recipe-locks) also apply here, specifically:

*   Restrict lock visibility.

*   Document what the lock protects.

*   Minimize code under lock.

*   Never execute arbitrary code while holding a lock.

Keep your lock instances private; do not expose them outside the class. Be sure to clearly document (and carefully think through) exactly what a lock instance protects. Minimize code that is executed while holding a lock. In particular, do not call arbitrary code; this includes raising events, invoking virtual methods, and invoking delegates.

###### Tip

The `AsyncLock` type is in the [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet package.

## See Also

[Recipe 12.4](#recipe-async-signals) covers `async`-compatible signaling. Locks are intended to protect shared data, not act as signals.

[Recipe 12.5](#recipe-throttling) covers throttling, which is a generalization of locking. A lock can be thought of as throttling to one at a time.

# 12.3 Blocking Signals

## Problem

You have to send a notification from one thread to another.

## Solution

The most common and general-purpose cross-thread signal is `ManualResetEventSlim`. A manual-reset event can be in one of two states: signaled or unsignaled. Any thread may set the event to a signaled state or reset the event to an unsignaled state. A thread may also wait for the event to be signaled.

The following two methods are invoked by separate threads; one thread waits for a signal from the other:

[PRE12]

## Discussion

`ManualResetEventSlim` is a great general-purpose signal from one thread to another, but you should only use it when appropriate. If the “signal” is actually a *message* sending some piece of data across threads, then consider using a producer/consumer queue. On the other hand, if the signals are just used to coordinate access to shared data, then you should use a lock instead.

There are other thread synchronization signal types in the .NET framework that are less commonly used. If `ManualResetEventSlim` doesn’t suit your needs, consider `AutoResetEvent`, `CountdownEvent`, or `Barrier`.

`ManualResetEventSlim` is a synchronous signal, so `WaitForInitialization` will block the calling thread until the signal is sent. If you want to wait for a signal without blocking a thread, then you want an asynchronous signal, as described in [Recipe 12.4](#recipe-async-signals).

## See Also

[Recipe 9.6](ch09.html#recipe-blocking-queues) covers blocking producer/consumer queues.

[Recipe 12.1](#recipe-locks) covers blocking locks.

[Recipe 12.4](#recipe-async-signals) covers `async`-compatible signals.

# 12.4 Async Signals

## Problem

You need to send a notification from one part of the code to another, and the receiver of the notification must wait for it asynchronously.

## Solution

Use `TaskCompletionSource<T>` to send the notification asynchronously, if the notification only needs to be sent once. The sending code calls `TrySetResult`, and the receiving code awaits its `Task` property:

[PRE13]

The `TaskCompletionSource<T>` type can be used to asynchronously wait for any kind of situation—in this case, a notification from another part of the code. This works well if the signal is only sent once, but doesn’t work as well if you need to turn the signal off as well as on.

The `Nito.AsyncEx` library contains a type `AsyncManualResetEvent`, which is an approximate equivalent of `ManualResetEvent` for asynchronous code. The following example is fabricated, but it shows how to use the `AsyncManualResetEvent` type:

[PRE14]

## Discussion

Signals are a general-purpose notification mechanism. But if that “signal” is a *message*, used to send data from one piece of code to another, then consider using a producer/consumer queue. Similarly, do not use general-purpose signals just to coordinate access to shared data; in that situation, use an asynchronous lock.

###### Tip

The `AsyncManualResetEvent` type is in the [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet package.

## See Also

[Recipe 9.8](ch09.html#recipe-async-queues) covers asynchronous producer/consumer queues.

[Recipe 12.2](#recipe-async-locks) covers asynchronous locks.

[Recipe 12.3](#recipe-signals) covers blocking signals, which can be used for notifications across threads.

# 12.5 Throttling

## Problem

You have highly concurrent code that is actually *too* concurrent, and you need some way to throttle the concurrency.

Code is too concurrent when parts of the application are unable to keep up with other parts, causing data items to build up and consume memory. In this scenario, throttling parts of the code can prevent memory issues.

## Solution

The solution varies based on the type of concurrency your code is doing. These solutions all restrict concurrency to a specific value. Reactive Extensions has more powerful options, such as sliding time windows; throttling for System.Reactive observables is covered more thoroughly in [Recipe 6.4](ch06.html#recipe-rx-throttle).

Dataflow and parallel code all have built-in options for throttling concurrency:

[PRE15]

Concurrent asynchronous code can be throttled by using `SemaphoreSlim`:

[PRE16]

## Discussion

Throttling may be necessary when you find your code is using too many resources (for example, CPU or network connections). Bear in mind that end users usually have less powerful machines than developers, so it’s better to throttle by a little too much than not enough.

## See Also

[Recipe 6.4](ch06.html#recipe-rx-throttle) covers throttling for reactive code.