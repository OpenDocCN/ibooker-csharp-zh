# Chapter 13\. Scheduling

When a piece of code executes, it has to run on some thread somewhere. A *scheduler* is an object that decides where a certain piece of code runs. There are a few different scheduler types in the .NET framework, and they’re used with slight differences by parallel and dataflow code.

I recommend that you *not* specify a scheduler whenever possible; the defaults are usually correct. For example, the `await` operator in asynchronous code will automatically resume the method within the same context unless you override this default, as described in [Recipe 2.7](ch02.html#recipe-async-configureawait). Similarly, reactive code has reasonable default contexts for raising its events, which you can override with `ObserveOn`, as described in [Recipe 6.2](ch06.html#recipe-rx-observeon).

If you need other code to execute in a specific context (e.g., a UI thread context or an ASP.NET request context), then you can use the scheduling recipes in this chapter to control the scheduling of your code.

# 13.1 Scheduling Work to the Thread Pool

## Problem

You have a piece of code that you explicitly want to execute on a threadpool thread.

## Solution

The vast majority of the time, you’ll want to use `Task.Run`, which is quite simple. The following code blocks a threadpool thread for 2 seconds:

[PRE0]

`Task.Run` also understands return values and asynchronous lambdas just fine. The task returned by `Task.Run` in the following code will complete after 2 seconds with a result of 13:

[PRE1]

`Task.Run` returns a `Task` (or `Task<T>`), which can be naturally consumed by asynchronous or reactive code.

## Discussion

`Task.Run` is ideal for UI applications, when you have time-consuming work to do that cannot be done on the UI thread. For example, [Recipe 8.4](ch08.html#recipe-async-interop-parallel) uses `Task.Run` to push parallel processing to a threadpool thread. However, do not use `Task.Run` on ASP.NET unless you’re *absolutely* sure you know what you’re doing. On ASP.NET, request handling code is already running on a threadpool thread, so pushing it onto *another* threadpool thread is usually counterproductive.

`Task.Run` is an effective replacement for `BackgroundWorker`, `Delegate.BeginInvoke`, and `ThreadPool.QueueUserWorkItem`. None of those older APIs should be used in new code; code using `Task.Run` is much easier to write correctly and maintain over time. Furthermore, `Task.Run` handles the vast majority of use cases for `Thread`, so most uses of `Thread` can also be replaced with `Task.Run` (with the rare exception of single-thread apartment threads).

Parallel and dataflow code executes on the thread pool by default, so there’s usually no need to use `Task.Run` with code executed by the `Parallel`, Parallel LINQ, or TPL Dataflow libraries.

If you’re doing dynamic parallelism, then use `Task.Factory.StartNew` instead of `Task.Run`. This is necessary because the `Task` returned by `Task.Run` has its default options configured for asynchronous use (i.e., to be consumed by asynchronous or reactive code). It also doesn’t support advanced concepts, such as parent/child tasks, which are more common in dynamic parallel code.

## See Also

[Recipe 8.6](ch08.html#recipe-rx-interop-async) covers consuming asynchronous code (such as the task returned from `Task.Run`) with reactive code.

[Recipe 8.4](ch08.html#recipe-async-interop-parallel) covers asynchronously waiting for parallel code, which is most easily done via `Task.Run`.

[Recipe 4.4](ch04.html#recipe-parallel-dynamicparallelism) covers dynamic parallelism, a scenario where you should use `Task.Factory.StartNew` instead of `Task.Run`.

# 13.2 Executing Code with a Task Scheduler

## Problem

You have multiple pieces of code that you need to execute in a certain way. For example, you may need all the pieces of code to execute on the UI thread, or you may need to execute only a certain number at a time.

This recipe deals with how to define and construct a scheduler for those pieces of code. Actually applying that scheduler is the subject of the next two recipes.

## Solution

There are quite a few different types in .NET that can handle scheduling; this recipe focuses on `TaskScheduler` because it’s portable and relatively easy to use.

The simplest `TaskScheduler` is `TaskScheduler.Default`, which queues work to the thread pool. You will seldomly specify `TaskScheduler.Default` in your own code, but it’s important to be aware of it, since it’s the default for many scheduling scenarios. `Task.Run`, parallel, and dataflow code all use `TaskScheduler.Default`.

You can capture a specific *context* and later schedule work back to it by using `TaskScheduler.FromCurrentSynchronizationContext`:

[PRE2]

This code creates a `TaskScheduler` to capture the current `SynchronizationContext` and schedule code onto that context. `SynchronizationContext` is a type that represents a general-purpose scheduling context. There are several different contexts in the .NET framework; most UI frameworks provide a `SynchronizationContext` that represents the UI thread, and ASP.NET before Core provided a `SynchronizationContext` that represented the HTTP request context.

`ConcurrentExclusiveSchedulerPair` is another powerful type introduced in .NET 4.5; this is actually *two* schedulers that are related to each other. The `ConcurrentScheduler` member is a scheduler that allows multiple tasks to execute at the same time, as long as no task is executing on the `ExclusiveScheduler`. The `ExclusiveScheduler` only executes code one task at a time, and only when there’s no task already executing on the `ConcurrentScheduler`:

[PRE3]

One common utilization for `ConcurrentExclusiveSchedulerPair` is to just use the `ExclusiveScheduler` to ensure only one task is executed at a time. Code that executes on the `ExclusiveScheduler` will run on the thread pool but will be restricted to executing exclusive of all other code using the same `ExclusiveScheduler` instance.

Another use for `ConcurrentExclusiveSchedulerPair` is as a throttling scheduler. You can create a `ConcurrentExclusiveSchedulerPair` that will limit its own concurrency. In this scenario, the `ExclusiveScheduler` is usually not used:

[PRE4]

Note that this kind of throttling only throttles code while it is *executing*; it’s quite different than the kind of logical throttling covered in [Recipe 12.5](ch12.html#recipe-throttling). In particular, asynchronous code is not considered to be executing while it is awaiting an operation. The `ConcurrentScheduler` throttles executing code; other throttling, such as `SemaphoreSlim`, throttles at a higher level (i.e., an entire `async` method).

## Discussion

You may have noticed that the last code example passed `TaskScheduler.Default` into the constructor for `ConcurrentExclusiveSchedulerPair`. This is because `ConcurrentExclusiveSchedulerPair` applies its concurrent/exclusive logic around an existing `TaskScheduler`.

This recipe introduces `TaskScheduler.FromCurrentSynchronizationContext`, which is useful for executing code on a captured context. It is also possible to use `SynchronizationContext` directly to execute code on that context; however, I don’t recommend this approach. Whenever possible, use the `await` operator to resume on an implicitly captured context, or use a `TaskScheduler` wrapper.

Don’t ever use platform-specific types to execute code on a UI thread. WPF, Silverlight, iOS, and Android all provide the `Dispatcher` type, Universal Windows uses the `CoreDispatcher`, and Windows Forms has the `ISynchronizeInvoke` interface (i.e., `Control.Invoke`). Do not use any of these types in new code; just pretend they don’t exist. Using them ties your code to a specific platform unnecessarily. `SynchronizationContext` is a general-purpose abstraction around these types.

System.Reactive (Rx) introduces a more general scheduler abstraction: `IScheduler`. An Rx scheduler is capable of wrapping any other kind of scheduler; the `TaskPoolScheduler` will wrap any `TaskFactory` (which contains a `TaskScheduler`). The Rx team also defined an `IScheduler` implementation that can be manually controlled for testing. If you do need to use a scheduler abstraction, I’d recommend the `IScheduler` from Rx; it’s well designed, well defined, and test friendly. However, most of the time you don’t need a scheduler abstraction, and earlier libraries, such as the Task Parallel Library (TPL) and TPL Dataflow, only understand the `TaskScheduler` type.

## See Also

[Recipe 13.3](#recipe-parallel-scheduler) covers applying a `TaskScheduler` to parallel code.

[Recipe 13.4](#recipe-dataflow-scheduler) covers applying a `TaskScheduler` to dataflow code.

[Recipe 12.5](ch12.html#recipe-throttling) covers higher-level logical throttling.

[Recipe 6.2](ch06.html#recipe-rx-observeon) covers System.Reactive schedulers for event streams.

[Recipe 7.6](ch07.html#recipe-rx-testscheduling) covers the System.Reactive test scheduler.

# 13.3 Scheduling Parallel Code

## Problem

You need to control how the individual pieces of code are executed in parallel code.

## Solution

Once you create an appropriate `TaskScheduler` instance (see [Recipe 13.2](#recipe-taskscheduler)), you can include it in the options that you pass to a `Parallel` method. The following code takes a sequence of sequences of matrices. It starts a bunch of parallel loops and wants to limit the *total* parallelism of all loops simultaneously, regardless of how many matrices are in each sequence:

[PRE5]

## Discussion

`Parallel.Invoke` also takes an instance of `ParallelOptions`, so you can pass a `TaskScheduler` to `Parallel.Invoke` the same way as `Parallel.ForEach`. If you’re doing dynamic parallel code, you can pass `TaskScheduler` directly to `TaskFactory.StartNew` or `Task.ContinueWith`.

There is no way to pass a `TaskScheduler` to Parallel LINQ (PLINQ) code.

## See Also

[Recipe 13.2](#recipe-taskscheduler) covers common task schedulers and how to choose between them.

# 13.4 Dataflow Synchronization Using Schedulers

## Problem

You need to control how the individual pieces of code are executed in dataflow code.

## Solution

Once you create an appropriate `TaskScheduler` instance (see [Recipe 13.2](#recipe-taskscheduler)), you can include it in the options that you pass to a dataflow block. When called from the UI thread, the following code creates a dataflow mesh that multiplies all of its input values by two (using the thread pool) and then appends the resulting values to the items of a list box (on the UI thread):

[PRE6]

## Discussion

Specifying a `TaskScheduler` is especially useful in coordinating the actions of blocks in different parts of your dataflow mesh. For example, you can utilize the `ConcurrentExclusiveSchedulerPair.ExclusiveScheduler` to ensure that blocks A and C never execute code at the same time, while allowing block B to execute whenever it wants.

Keep in mind that synchronization by `TaskScheduler` only applies while the code is *executing*. For example, if you have an action block that runs asynchronous code and apply an exclusive scheduler, the code isn’t considered running when it is awaiting.

You can specify a `TaskScheduler` for any kind of dataflow block. Even though a block may not execute your code (e.g., `BufferBlock<T>`), it still has housekeeping tasks that it needs to do, and it’ll use the provided `TaskScheduler` for all of its internal work.

## See Also

[Recipe 13.2](#recipe-taskscheduler) covers common task schedulers and how to choose between them.