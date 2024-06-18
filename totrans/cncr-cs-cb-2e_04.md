# Chapter 4\. Parallel Basics

This chapter covers patterns for parallel programming. Parallel programming is used to split up CPU-bound pieces of work and divide them among multiple threads. These parallel processing recipes only consider CPU-bound work. If you have naturally asynchronous operations (such as I/O-bound work) that you want to execute in parallel, then see [Chapter 2](ch02.html#async-basics), and [Recipe 2.4](ch02.html#recipe-async-whenall) in particular.

The parallel processing abstractions covered in this chapter are part of the Task Parallel Library (TPL). The TPL is built into the .NET framework.

# 4.1 Parallel Processing of Data

## Problem

You have a collection of data, and you need to perform the same operation on each element of the data. This operation is CPU-bound and may take some time.

## Solution

The `Parallel` type contains a `ForEach` method specifically designed for this problem. The following example takes a collection of matrices and rotates them all:

[PRE0]

There are some situations where you’ll want to stop the loop early, such as if you encounter an invalid value. The following example inverts each matrix, but if an invalid matrix is encountered, it’ll abort the loop:

[PRE1]

This code uses `ParallelLoopState.Stop` to stop the loop, preventing any further invocations of the loop body. Bear in mind that this is a parallel loop, so other invocations of the loop body may already be running, including invocations for items after the current item. In this code example, if the third matrix isn’t invertible, the loop is stopped and no new matrixes will be processed, but other matrixes (such as the fourth and fifth) may already be processing.

A more common situation is when you want the ability to cancel a parallel loop. This is different than stopping the loop; a loop is *stopped* from inside the loop, and it is *canceled* from outside the loop. To show an example, a cancel button may cancel a `CancellationTokenSource`, canceling a parallel loop as in this code example:

[PRE2]

One thing to keep in mind is that each parallel task may run on a different thread, so any shared state must be protected. The following example inverts each matrix and counts the number of matrices that couldn’t be inverted:

[PRE3]

## Discussion

The `Parallel.ForEach` method enables parallel processing over a sequence of values. A similar solution is Parallel LINQ (PLINQ), which provides much of the same capabilities with a LINQ-like syntax. One difference between `Parallel` and PLINQ is that PLINQ assumes it can use all the cores on the computer, while `Parallel` will dynamically react to changing CPU conditions.

`Parallel.ForEach` is a parallel `foreach` loop. If you need to do a parallel `for` loop, the `Parallel` class also supports a `Parallel.For` method. `Parallel.For` is especially useful if you have multiple arrays of data that all take the same index.

## See Also

[Recipe 4.2](#recipe-parallel-aggregate) covers aggregating a series of values in parallel, including sums and averages.

[Recipe 4.5](#recipe-parallel-plinq) covers the basics of PLINQ.

[Chapter 10](ch10.html#cancellation) covers cancellation.

# 4.2 Parallel Aggregation

## Problem

At the conclusion of a parallel operation, you need to aggregate the results. Examples of aggregation are summing up values or finding their average.

## Solution

The `Parallel` class supports aggregation through the concept of *local values*, which are variables that exist locally within a parallel loop. This means that the body of the loop can just access the value directly, without needing synchronization. When the loop is ready to aggregate each of its local results, it does so with the `localFinally` delegate. Note that the `localFinally` delegate *does* need to synchronize access to the variable that holds the final result. Here’s an example of a parallel sum:

[PRE4]

Parallel LINQ has more natural aggregation support than the `Parallel` class:

[PRE5]

OK, that was a cheap shot, since PLINQ has built-in support for many common operators (for example, `Sum`). PLINQ also has generic aggregation support via the `Aggregate` operator:

[PRE6]

## Discussion

If you’re already using the `Parallel` class, you may want to use its aggregation support. Otherwise, in most scenarios, the PLINQ support is more expressive and has shorter code.

## See Also

[Recipe 4.5](#recipe-parallel-plinq) covers the basics of PLINQ.

# 4.3 Parallel Invocation

## Problem

You have a number of methods to call in parallel, and these methods are (mostly) independent of one another.

## Solution

The `Parallel` class contains a simple `Invoke` member that is designed for this scenario. This example splits an array in half and processes each half independently:

[PRE7]

You can also pass an array of delegates to the `Parallel.Invoke` method if the number of invocations isn’t known until runtime:

[PRE8]

`Parallel.Invoke` supports cancellation just like the other members of the `Parallel` class:

[PRE9]

## Discussion

`Parallel.Invoke` is a great solution for simple parallel invocation. Note that it will not be a perfect fit if you want to invoke an action for each item of input data (use `Parallel.ForEach` instead) or if each action produces some output (use Parallel LINQ instead).

## See Also

[Recipe 4.1](#recipe-parallel-foreach) covers `Parallel.ForEach`, which invokes an action for each item of data.

[Recipe 4.5](#recipe-parallel-plinq) covers Parallel LINQ.

# 4.4 Dynamic Parallelism

## Problem

You have a more complex parallel situation where the structure and number of parallel tasks depend on information known only at runtime.

## Solution

The Task Parallel Library (TPL) is centered around the `Task` type. The `Parallel` class and Parallel LINQ are just convenience wrappers around the powerful `Task`. When you need dynamic parallelism, it’s easiest to use the `Task` type directly.

Here is an example in which some expensive processing needs to be done for each node of a binary tree. The structure of the tree won’t be known until runtime, so this is a good scenario for dynamic parallelism. The `Traverse` method processes the current node and then creates two child tasks, one for each branch underneath the node (for this example, I’m assuming that the parent nodes must be processed before the children). The `ProcessTree` method starts the processing by creating a top-level parent task and waiting for it to complete:

[PRE10]

The `AttachedToParent` flag ensures that the `Task` for each branch is linked to the `Task` for their parent node. This creates parent/child relationships among the `Task` instances that mirror the parent/child relationships in the tree nodes. Parent tasks execute their delegate and then wait for their child tasks to complete. Exceptions from child tasks are then propagated from the child tasks to their parent task. So, `ProcessTree` can wait for the tasks for the entire tree just by calling `Wait` on the single `Task` at the root of the tree.

If you don’t have a parent/child kind of situation, you can schedule any task to run after another by using a task *continuation*. The continuation is a separate task that executes when the original task completes:

[PRE11]

## Discussion

`CancellationToken.None` and `TaskScheduler.Default` are used in the preceding code example. Cancellation tokens are covered in [Recipe 10.2](ch10.html#recipe-cancellationtoken), and task schedulers are covered in [Recipe 13.3](ch13.html#recipe-parallel-scheduler). It’s always a good idea to explicitly specify the `TaskScheduler` used by `StartNew` and `ContinueWith`.

This arrangement of parent and child tasks is common with dynamic parallelism, although it’s not required. It’s equally possible to store each new task in a threadsafe collection and then wait for them all to complete using `Task.WaitAll`.

###### Warning

Using `Task` for parallel processing is completely different than using `Task` for asynchronous processing.

The `Task` type serves two purposes in concurrent programming: it can be a parallel task or an asynchronous task. Parallel tasks may use blocking members, such as `Task.Wait`, `Task.Result`, `Task.WaitAll`, and `Task.WaitAny`. Parallel tasks also commonly use `AttachedToParent` to create parent/child relationships between tasks. Parallel tasks should be created with `Task.Run` or `Task.Factory.StartNew`.

In contrast, asynchronous tasks should avoid blocking members, and prefer `await`, `Task.WhenAll`, and `Task.WhenAny`. Asynchronous tasks should not use `AttachedToParent`, but they can form an implicit kind of parent/child relationship by awaiting another task.

## See Also

[Recipe 4.3](#recipe-parallel-invoke) covers invoking a sequence of methods in parallel, when all the methods are known at the start of the parallel work.

# 4.5 Parallel LINQ

## Problem

You need to perform parallel processing on a sequence of data to produce another sequence of data or a summary of that data.

## Solution

Most developers are familiar with LINQ, which you can use to write pull-based calculations over sequences. Parallel LINQ (PLINQ) extends this LINQ support with parallel processing.

PLINQ works well in streaming scenarios, when you have a sequence of inputs and are producing a sequence of outputs. Here’s a simple example that just multiplies each element in a sequence by two (real-world scenarios will be much more CPU-intensive than a simple multiply):

[PRE12]

The example may produce its outputs in any order; this behavior is the default for Parallel LINQ. You can also specify the order to be preserved. The following example is still processed in parallel, but it preserves the original order:

[PRE13]

Another natural use of Parallel LINQ is to aggregate or summarize the data in parallel. The following code performs a parallel summation:

[PRE14]

## Discussion

The `Parallel` class is good for many scenarios, but PLINQ code is simpler when doing aggregation or transforming one sequence to another. Bear in mind that the `Parallel` class is more friendly to other processes on the system than PLINQ; this is especially a consideration if the parallel processing is done on a server machine.

PLINQ provides parallel versions of a wide variety of operators, including filtering (`Where`), projection (`Select`), and a variety of aggregations, such as `Sum`, `Average`, and the more generic `Aggregate`. In general, anything you can do with regular LINQ you can do in parallel with PLINQ. This makes PLINQ a great choice if you have existing LINQ code that would benefit from running in parallel.

## See Also

[Recipe 4.1](#recipe-parallel-foreach) covers how to use the `Parallel` class to execute code for each element in a sequence.

[Recipe 10.5](ch10.html#recipe-cancel-parallel) covers how to cancel PLINQ queries.