# Chapter 5\. Dataflow Basics

TPL Dataflow is a powerful library that enables you to create a mesh or pipeline and then (asynchronously) send your data through it. Dataflow is a very declarative style of coding: normally, you completely define the mesh first and then start processing data. The mesh ends up being a structure through which your data flows. This requires you to think about your application a bit differently, but once you make that leap, dataflow becomes a natural fit for many scenarios.

Each mesh is comprised of various blocks that are linked to each other. The individual blocks are simple and are responsible for a single step in the data processing. When a block finishes working on its data, it will pass its result along to any linked blocks.

To use TPL Dataflow, install the NuGet package [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) into your application.

# 5.1 Linking Blocks

## Problem

You need to link dataflow blocks to one another to create a mesh.

## Solution

The blocks provided by the TPL Dataflow library define only the most basic members. Many of the useful TPL Dataflow methods are actually extension methods. The `LinkTo` extension method provides an easy way to link dataflow blocks together:

[PRE0]

By default, linked dataflow blocks only propagate data; they do not propagate completion (or errors). If your dataflow is linear (like a pipeline), then you will probably want to propagate completion. To propagate completion (and errors), you can set the `PropagateCompletion` option on the link:

[PRE1]

## Discussion

Once linked, data will flow automatically from the source block to the target block. The `PropagateCompletion` option flows completion in addition to data; however, at each step in the pipeline, a faulting block will propagate its exception to the next block wrapped in an `AggregateException`. So, if you have a long pipeline that propagates completions, the original error may be nested in multiple `AggregateException` instances. `AggregateException` has several members, such as `Flatten`, that assist with error handling in this situation.

It is possible to link dataflow blocks in many ways; your mesh can have forks and joins and even loops. However, the simple, linear pipeline is sufficient for most scenarios. We’ll be dealing mainly with pipelines (and briefly cover forks); more advanced scenarios are beyond the scope of this book.

The `DataflowLinkOptions` type gives you several different options you can set on a link (such as the `PropagateCompletion` option used in this solution), and the `LinkTo` overload can also take a predicate that you can use to filter which data can go over a link. If data doesn’t pass the filter, it is not dropped. Data that passes the filter travels over that link; data that doesn’t pass the filter attempts to pass over an alternate link, and stays in the block if there’s no other link for it to take. If a data item gets stuck in a block like this, then that block won’t produce any other data items; the entire block becomes stalled until that data item is removed.

## See Also

[Recipe 5.2](#recipe-dataflow-errors) covers propagating errors along links.

[Recipe 5.3](#recipe-dataflow-unlinking) covers removing links between blocks.

[Recipe 8.8](ch08.html#recipe-rx-interop-dataflow) covers how to link dataflow blocks to System.Reactive observable streams.

# 5.2 Propagating Errors

## Problem

You need a way to respond to errors that can happen in your dataflow mesh.

## Solution

If a delegate passed to a dataflow block throws an exception, then that block will enter a faulted state. When a block is in a faulted state, it will drop all of its data (and stop accepting new data). The block in the following code will never produce any output data; the first value raises an exception, and the second value is just dropped:

[PRE2]

To catch exceptions from a dataflow block, you should `await` its `Completion` property. The `Completion` property returns a `Task` that will complete when the block is completed, and if the block faults, the `Completion` task is also faulted:

[PRE3]

When you propagate completion using the `PropagateCompletion` link option, errors are also propagated. However, the exception is passed to the next block wrapped in an `AggregateException`. The following example catches the exception from the end of a pipeline, so it would catch `AggregateException` if an exception was propagated from earlier blocks:

[PRE4]

Each block wraps incoming errors in an `AggregateException`, even if the incoming error is already an `AggregateException`. If an error occurs early in a pipeline and travels down several links before it’s observed, the original error will be wrapped in multiple layers of `AggregateException`. The `AggregateException.Flatten` method simplifies error handling in this scenario.

## Discussion

When you build your mesh (or pipeline), consider how errors should be handled. In simpler situations, it can be best to just propagate the errors and catch them once at the end. In more complex meshes, you may need to observe each block when the dataflow has completed.

Alternatively, if you want your blocks to remain viable in the face of exceptions, you can choose to treat exceptions as another kind of data and let them flow through your mesh along with your correctly processed data items. Using that pattern, you can keep your dataflow mesh operational, since the blocks themselves don’t fault and continue processing the next data item. See [Recipe 14.6](ch14.html#recipe-railway-dataflow) for more details.

## See Also

[Recipe 5.1](#recipe-dataflow-linking) covers establishing links between blocks.

[Recipe 5.3](#recipe-dataflow-unlinking) covers breaking links between blocks.

[Recipe 14.6](ch14.html#recipe-railway-dataflow) covers flowing exceptions alongside data in a dataflow mesh.

# 5.3 Unlinking Blocks

## Problem

During processing, you need to dynamically change the structure of your dataflow. This is an advanced scenario that is hardly ever needed.

## Solution

You can link or unlink dataflow blocks at any time; data can be freely passing through the mesh and it’s still safe to link or unlink at any time. Both linking and unlinking are fully threadsafe.

When you create a dataflow block link, keep the `IDisposable` returned by the `LinkTo` method, and dispose of it when you want to unlink the blocks:

[PRE5]

## Discussion

Unless you can guarantee that the link is idle, there will be race conditions when you unlink it. However, these race conditions are usually not a concern; data will either flow over the link before the link is broken, or it won’t. There are no race conditions that would cause duplication or loss of data.

Unlinking is an advanced scenario, but it can be useful in a handful of situations. As one example, there’s no way to change the filter for a link. To change the filter on an existing link, you’d have to unlink the old one and create a new link with the new filter (optionally setting `DataflowLinkOptions.Append` to false). As another example, unlinking at a strategic point can be used to pause a dataflow mesh.

## See Also

[Recipe 5.1](#recipe-dataflow-linking) covers establishing links between blocks.

# 5.4 Throttling Blocks

## Problem

You have a fork scenario in your dataflow mesh and want the data to flow in a load-balancing way.

## Solution

By default, when a block produces output data, it’ll examine all of its links (in the order they were created) and attempt to flow the data down each link one at a time. Also, by default, each block will maintain an input buffer and accept any amount of data before it’s ready to process it.

This causes a problem in a fork scenario, where one source block is linked to two target blocks: the second block is then starved. As the source block produces data, it will try to flow the data down each link. The first target block would always accept the data and buffer it, and so the source block would never try to flow the data to the second target block. This problem can be fixed by throttling the target blocks using the `BoundedCapacity` block option. By default, `BoundedCapacity` is set to `DataflowBlockOptions.Unbounded`, which causes the first target block to buffer *all* the data even if it isn’t ready to process it yet.

`BoundedCapacity` can be set to any value greater than zero (or, of course, `DataflowBlockOptions.Unbounded`). As long as the target blocks can keep up with the data coming from the source blocks, a simple value of 1 will suffice:

[PRE6]

## Discussion

Throttling is useful for load balancing in fork scenarios, but it can be used anywhere else you want throttling behavior. For example, if you’re populating your dataflow mesh with data from an I/O operation, you can apply `BoundedCapacity` to the blocks in your mesh. That way, you won’t read too much I/O data until your mesh is ready for it, and your mesh won’t end up buffering all the input data before it’s able to process it.

## See Also

[Recipe 5.1](#recipe-dataflow-linking) covers linking blocks together.

# 5.5 Parallel Processing with Dataflow Blocks

## Problem

You want to perform some parallel processing within your dataflow mesh.

## Solution

By default, each dataflow block is independent from each other block. When you link two blocks together, they will process independently. So, every dataflow mesh has some natural parallelism built in.

If you need to go beyond this—for example, if you have one particular block that does heavy CPU computations—then you can instruct that block to operate in parallel on its input data by setting the `MaxDegreeOfParallelism` option. By default, this option is set to 1, so each dataflow block will only process one piece of data at a time.

`BoundedCapacity` can be set to `DataflowBlockOptions.Unbounded` or any value greater than zero. The following example permits any number of tasks to be multiplying data simultaneously:

[PRE7]

## Discussion

The `MaxDegreeOfParallelism` option makes parallel processing within a block easy to do. What is not so easy is determining which blocks need it. One technique is to pause dataflow execution in the debugger, where you can see the number of data items queued up (i.e., the ones that haven’t yet been processed by the block). An unexpected number of data items can be an indication that some restructuring or parallelization would be helpful.

`MaxDegreeOfParallelism` also works if the dataflow block does asynchronous processing. In this case, the `MaxDegreeOfParallelism` option specifies the level of concurrency—a certain number of *slots*. Each data item takes up a slot when the block begins processing it and only leaves that slot when the asynchronous processing is fully completed.

## See Also

[Recipe 5.1](#recipe-dataflow-linking) covers linking blocks together.

# 5.6 Creating Custom Blocks

## Problem

You have reusable logic that you want to place into a custom dataflow block. Doing so enables you to create larger blocks that contain complex logic.

## Solution

You can cut out any part of a dataflow mesh that has a single input and output block by using the `Encapsulate` method. `Encapsulate` will create a single block out of the two endpoints. Propagating data *and completion* between those endpoints is your responsibility. The following code creates a custom dataflow block out of two blocks, propagating data and completion:

[PRE8]

## Discussion

When you encapsulate a mesh into a custom block, consider what kind of options you want to expose to your users. Consider how each block option should (or shouldn’t) be passed on to your inner mesh; in many cases, some block options don’t apply or don’t make sense. For this reason, it’s common for custom blocks to define their own custom options instead of accepting a `DataflowBlockOptions` parameter.

`DataflowBlock.Encapsulate` will only encapsulate a mesh with one input block and one output block. If you have a reusable mesh with multiple inputs and/or outputs, you should encapsulate it within a custom object and expose the inputs and outputs as properties of type `ITargetBlock<T>` (for inputs) and `IReceivableSourceBlock<T>` (for outputs).

These examples all use `Encapsulate` to create a custom block. It is also possible to implement the dataflow interfaces yourself, but it’s much more difficult. Microsoft has [a paper](http://bit.ly/tpl-dataflow) that describes advanced techniques for creating your own custom dataflow blocks.

## See Also

[Recipe 5.1](#recipe-dataflow-linking) covers linking blocks together.

[Recipe 5.2](#recipe-dataflow-errors) covers propagating errors along block links.