# Chapter 9\. Collections

Using the proper collections is essential in concurrent applications. I’m not talking about the standard collections like `List<T>`; I assume you already know about those. The purpose of this chapter is to introduce newer collections that are specifically intended for concurrent or asynchronous use.

*Immutable collections* are collection instances that can never change. At first glance, this sounds completely useless; but they’re actually very useful, even in single-threaded, nonconcurrent applications. Read-only operations (such as enumeration) act directly on the immutable instance. Write operations (such as adding an item) return a new immutable instance instead of changing the existing instance. This isn’t as wasteful as it first sounds because most of the time immutable collections share most of their memory. Furthermore, immutable collections have the advantage of being implicitly safe to access from multiple threads; since they cannot change, they are threadsafe.

###### Tip

Immutable collections are in the [`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet package.

Immutable collections are new, but they should be considered for new development unless you *need* a mutable instance. If you’re not familiar with immutable collections, I recommend that you start with [Recipe 9.1](#recipe-immutable-stacks-queues), even if you don’t need a stack or queue, because I’ll cover several common patterns that all immutable collections follow.

There are special ways to more efficiently construct an immutable collection with lots of existing elements; the example code in these recipes only adds elements one at a time. The MSDN documentation has details on how to efficiently construct immutable collections if you need to speed up your initialization.

Threadsafe collections

These mutable collection instances can be changed by multiple threads simultaneously. Threadsafe collections use a mixture of fine-grained locks and lock-free techniques to ensure that threads are blocked for a minimal amount of time (and usually aren’t blocked at all). For many threadsafe collections, enumeration of the collection creates a snapshot of the collection and then enumerates that snapshot. The key advantage of threadsafe collections is that they can be accessed safely from multiple threads, yet the operations will only block your code for a short time, if at all.

Producer/consumer collections

These mutable collection instances are designed with a specific purpose in mind: to allow (possibly multiple) producers to push items to the collection while allowing (possibly multiple) consumers to pull items out of the collection. So they act as a bridge between producer code and consumer code, and they also have an option to limit the number of items in the collection. Producer/consumer collections can either have a blocking or asynchronous API. For example, when the collection is empty, a blocking producer/consumer collection will block the calling consumer thread until another item is added; but an asynchronous producer/consumer collection will allow the calling consumer thread to asynchronously wait until another item is added.

There are a number of different producer/consumer collections used in the recipes in this chapter, and different producer/consumer collections have different advantages. [Table 9-1](#producer_consumer_collect) may be helpful in determining which one you should use.

Table 9-1\. Producer/consumer collections

| Feature | Channels | BlockingCollection<T> | BufferBlock<T> | AsyncProducer-ConsumerQueue<T> | AsyncCollection<T> |
| --- | --- | --- | --- | --- | --- |
| Queue semantics | ✓ | ✓ | ✓ | ✓ | ✓ |
| Stack/bag semantics | ✗ | ✓ | ✗ | ✗ | ✓ |
| Synchronous API | ✓ | ✓ | ✓ | ✓ | ✓ |
| Asynchronous API | ✓ | ✗ | ✓ | ✓ | ✓ |
| Drop items when full | ✓ | ✗ | ✗ | ✗ | ✗ |
| Tested by Microsoft | ✓ | ✓ | ✓ | ✗ | ✗ |

###### Tip

Channels can be found in the [`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet package, `BufferBlock<T>` in the NuGet package for [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df), and `AsyncProducerConsumerQueue<T>` and `AsyncCollection<T>` in the NuGet package for [`Nito.AsyncEx`](http://bit.ly/nito-async).

# 9.1 Immutable Stacks and Queues

## Problem

You need a stack or queue that does not change very often and can be accessed by multiple threads safely.

For example, a queue can be used as a sequence of operations to perform, and a stack can be used as a sequence of undo operations.

## Solution

Immutable stacks and queues are the simplest immutable collections. They behave very similarly to the standard `Stack<T>` and `Queue<T>`. Performance-wise, immutable stacks and queues have the same time complexity as standard stacks and queues; however, in simple scenarios where the collections are updated frequently, the standard stacks and queues are faster.

Stacks are a first-in, last-out data structure. The following code creates an empty immutable stack, pushes two items, enumerates the items, and then pops an item:

[PRE0]

Note in the example that we keep overwriting the local variable `stack`. Immutable collections follow a pattern where they return an updated collection; the original collection reference is unchanged. This means that once you have a reference to a particular immutable collection instance, it’ll never change. Consider the following example:

[PRE1]

Under the covers, the two stacks are sharing the memory used to contain the item `13`. This kind of implementation is very efficient while enabling you to easily snapshot the current state. Each immutable collection instance is naturally threadsafe, but immutable collections can also be used in single-threaded applications. In my experience, immutable collections are especially useful when the code is more functional or when you need to store a large number of snapshots and want them to share memory as much as possible.

Queues are similar to stacks, except they are a first-in, first-out data structure. The following code creates an empty immutable queue, enqueues two items, enumerates the items, and then dequeues an item:

[PRE2]

## Discussion

This recipe introduced the two simplest immutable collections, the stack and the queue. It also covered several important design philosophies that are true for *all* immutable collections:

*   An instance of an immutable collection never changes.

*   Since it never changes, it is naturally threadsafe.

*   When you call a modifying method on an immutable collection, the new modified collection is returned.

###### Warning

Even though immutable collections are threadsafe, *references* to immutable collections are *not* threadsafe. A variable that refers to an immutable collection needs the same synchronization protections as any other variable (see [Chapter 12](ch12.html#synchronization)).

Immutable collections are ideal for sharing state. They don’t, however, work as well as communication conduits. In particular, don’t use an immutable queue to communicate between threads; producer/consumer queues work much better for that.

###### Tip

`ImmutableStack<T>` and `ImmutableQueue<T>` can be found in the [`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet package.

## See Also

[Recipe 9.6](#recipe-blocking-queues) covers threadsafe (blocking) mutable queues.

[Recipe 9.7](#recipe-blocking-stacks-bags) covers threadsafe (blocking) mutable stacks.

[Recipe 9.8](#recipe-async-queues) covers async-compatible mutable queues.

[Recipe 9.11](#recipe-async-stacks-bags) covers async-compatible mutable stacks.

[Recipe 9.12](#recipe-blocking-async-queues) covers blocking/asynchronous mutable queues.

# 9.2 Immutable Lists

## Problem

You need a data structure you can index into that does not change very often and can be accessed by multiple threads safely.

## Solution

A list is a general-purpose data structure that can be used for all kinds of application states. Immutable lists do allow indexing, but you need to be aware of the performance characteristics. They’re not just a drop-in replacement for `List<T>`.

`ImmutableList<T>` does support similar methods as `List<T>`, as the following example shows:

[PRE3]

The immutable list is internally organized as a binary tree so that immutable list instances may maximize the amount of memory they share with other instances. As a result, there are performance differences between `ImmutableList<T>` and `List<T>` for some common operations ([Table 9-2](#performance_imm_lists)).

Table 9-2\. Performance difference of immutable lists

| Operation | List<T> | ImmutableList<T> |
| --- | --- | --- |
| Add | amortized O(1) | O(log N) |
| Insert | O(N) | O(log N) |
| RemoveAt | O(N) | O(log N) |
| Item[index] | O(1) | O(log N) |

Of note, the indexing operation for `ImmutableList<T>` is O(log N), not O(1), as you may expect. If you’re replacing `List<T>` with `ImmutableList<T>` in existing code, you’ll need to consider how your algorithms access items in the collection.

This means that you should use `foreach` instead of `for` whenever possible. A `foreach` loop over an `ImmutableList<T>` executes in O(N) time, while a `for` loop over the same collection executes in O(N * log N) time:

[PRE4]

## Discussion

`ImmutableList<T>` is a good general-purpose data structure, but because of its performance differences, you can’t blindly replace all your `List<T>` uses with it. `List<T>` is commonly used by default—it’s the one you use unless you *need* a different collection. `ImmutableList<T>` isn’t quite that ubiquitous; you’ll need to consider the other immutable collections carefully and choose the one that makes the most sense for your situation.

###### Tip

`ImmutableList<T>` is in the [`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet package.

## See Also

[Recipe 9.1](#recipe-immutable-stacks-queues) covers immutable stacks and queues, which are like lists that only allow certain elements to be accessed.

The [MSDN documentation on `ImmutableList<T>.Builder`](http://bit.ly/msdn-iml) covers an efficient way to populate an immutable list.

# 9.3 Immutable Sets

## Problem

You need a data structure that does not need to store duplicates, does not change very often, and can be accessed by multiple threads safely.

For example, an index of words from a file would be a good use case for a set.

## Solution

There are two immutable set types: `ImmutableHashSet<T>` is a collection of unique items, and `ImmutableSortedSet<T>` is a *sorted* collection of unique items. Both types have a similar interface:

[PRE5]

Only the sorted set allows indexing into it like a list:

[PRE6]

Unsorted sets and sorted sets have similar performance (see [Table 9-3](#performance_of_sets)).

Table 9-3\. Performance of immutable sets

| Operation | ImmutableHashSet<T> | ImmutableSortedSet<T> |
| --- | --- | --- |
| Add | O(log N) | O(log N) |
| Remove | O(log N) | O(log N) |
| Item[index] | n/a | O(log N) |

However, I recommend you use an unsorted set unless you know it needs to be sorted. Many types only support basic equality and not full comparison, so an unsorted set can be used for many more types than a sorted set.

One important note about the sorted set is that its indexing is O(log N), not O(1), just like `ImmutableList<T>`, which is covered in [Recipe 9.2](#recipe-immutable-lists). This means that the same caveat applies in this situation: you should use `foreach` instead of `for` whenever possible with an `ImmutableSortedSet<T>`.

## Discussion

Immutable sets are useful data structures, but populating a large immutable set can be slow. Most immutable collections have special builders that can be used to construct them quickly in a mutable way and then convert them into an immutable collection. This is true for many immutable collections, but I’ve found them most useful for immutable sets.

###### Tip

`ImmutableHashSet<T>` and `ImmutableSortedSet<T>` are in the NuGet [`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) package.

## See Also

[Recipe 9.7](#recipe-blocking-stacks-bags) covers threadsafe mutable bags, which are similar to sets.

[Recipe 9.11](#recipe-async-stacks-bags) covers async-compatible mutable bags.

The [MSDN documentation on `ImmutableHashSet<T>.Builder`](http://bit.ly/msdn-imh) covers an efficient way to populate an immutable hash set.

The [MSDN documentation on `ImmutableSortedSet<T>.Builder`](http://bit.ly/msdn-ims) covers an efficient way to populate an immutable sorted set.

# 9.4 Immutable Dictionaries

## Problem

You need a key/value collection that does not change very often and can be accessed by multiple threads safely. For example, you may want to store reference data in a *lookup collection*; the reference data rarely changes but should be available to different threads.

## Solution

There are two immutable dictionary types: `ImmutableDictionary<TKey, TValue>` and `ImmutableSortedDictionary<TKey, TValue>`. As you may be able to guess from their names, while the items in `ImmutableDictionary` have an unpredictable order, `ImmutableSortedDictionary` ensures that its elements are sorted.

Both of these collection types have very similar members:

[PRE7]

Note the use of `SetItem`. In a mutable dictionary, you could try doing something like `dictionary[key] = item`, but immutable dictionaries must return the updated immutable dictionary, so they use the `SetItem` method instead:

[PRE8]

Unsorted dictionaries and sorted dictionaries have similar performance, but I recommend you use an unordered dictionary unless you need your elements to be sorted (see [Table 9-4](#performance_dictionaries)). Unsorted dictionaries can be a little faster overall. Furthermore, unsorted dictionaries can be used with any key types, whereas sorted dictionaries require their key types to be fully comparable.

Table 9-4\. Performance of immutable dictionaries

| Operation | ImmutableDictionary<TK,TV> | ImmutableSortedDictionary<TK,TV> |
| --- | --- | --- |
| Add | O(log N) | O(log N) |
| SetItem | O(log N) | O(log N) |
| Item[key] | O(log N) | O(log N) |
| Remove | O(log N) | O(log N) |

## Discussion

In my experience, dictionaries are a common and useful tool when dealing with application state. They can be used in any kind of key/value or lookup scenario.

Like other immutable collections, immutable dictionaries have a builder mechanism for efficient construction if the dictionary contains many elements. For example, if you load your initial reference data at startup, you should use the builder mechanism to construct the initial immutable dictionary. On the other hand, if your reference data is gradually built up during your application’s execution, then using the regular immutable dictionary `Add` method is likely acceptable.

###### Tip

`ImmutableDictionary<TK, TV>` and `ImmutableSortedDictionary<TK, TV>` are in the [`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet package.

## See Also

[Recipe 9.5](#recipe-threadsafe-dictionaries) covers threadsafe mutable dictionaries.

The [MSDN documentation on `ImmutableDictionary<TK,TV>.Builder`](http://bit.ly/msdn-imd) covers an efficient way to populate an immutable dictionary.

The [MSDN documentation on `ImmutableSortedDictionary<TK,TV>.Builder`](http://bit.ly/msdn-isd) covers an efficient way to populate an immutable sorted dictionary.

# 9.5 Threadsafe Dictionaries

## Problem

You have a key/value collection (e.g., an in-memory cache) that you need to keep in sync, even though multiple threads are both reading from and writing to it.

## Solution

The `ConcurrentDictionary<TKey, TValue>` type in the .NET framework is a true gem of a data structure. It’s threadsafe, using a mixture of fine-grained locks and lock-free techniques to ensure fast access in the vast majority of scenarios.

Its API does take a bit of getting used to. It’s very different from the standard `Dictionary<TKey, TValue>` type, since it must deal with concurrent access from multiple threads. But once you have learned the basics in this recipe, you’ll find `ConcurrentDictionary<TKey, TValue>` to be one of the most useful collection types.

First, let’s learn how to write a value to the collection. To set the value of a key, you can use `AddOrUpdate`:

[PRE9]

`AddOrUpdate` is a bit complex because it must do several things, depending on the current contents of the concurrent dictionary. The first method argument is the key. The second argument is a delegate that transforms the key (in this case, `0`) into a value to be added to the dictionary (in this case, `"Zero"`). This delegate is only invoked if the key doesn’t exist in the dictionary. The third argument is another delegate that transforms the key (`0`) and the old value into an updated value to be stored in the dictionary (`"Zero"`). This delegate is only invoked if the key does exist in the dictionary. `AddOrUpdate` returns the new value for that key (the same value that was returned by one of the delegates).

Now for the part that really bends your brain: in order for the concurrent dictionary to work properly, `AddOrUpdate` *might* have to invoke either (or both) delegates multiple times. This is very rare, but it *can* happen. So your delegates should be simple and fast and not cause side effects. This means that your delegate should only create the value; it shouldn’t change any other variables in your application. The same principle holds for all delegates you pass to methods on `ConcurrentDictionary<TKey, TValue>`.

There are several other ways to add values to a dictionary. One shortcut is to just use indexing syntax:

[PRE10]

Indexing syntax is less powerful; it doesn’t give you the ability to update a value based on the existing value. The syntax is simpler and works fine, however, if you already have the value you want to store in the dictionary.

Let’s look at how to read a value. This can be easily done via `TryGetValue`:

[PRE11]

`TryGetValue` will return `true` and set the `out` value if the key was found in the dictionary. If the key wasn’t found, `TryGetValue` will return `false`. You can also use indexing syntax to read values, but I find that much less useful because it’ll throw an exception if a key isn’t found. Keep in mind that a concurrent dictionary has multiple threads reading, updating, adding, and removing values; in many situations, it’s difficult to know whether a key exists or not until you attempt to read it.

Removing values is just as easy as reading them:

[PRE12]

`TryRemove` is almost identical to `TryGetValue`, except (of course) it removes the key/value pair if the key was found in the dictionary.

## Discussion

Although `ConcurrentDictionary<TKey, TValue>` is threadsafe, that doesn’t mean its operations are atomic. If multiple threads call `AddOrUpdate` concurrently, it’s possible for both of them to detect that the key isn’t present, and both of them concurrently execute their delegate that creates a new value.

I think `ConcurrentDictionary<TKey, TValue>` is awesome, mainly because of the incredibly powerful `AddOrUpdate` method. However, it doesn’t fit the bill in every situation. `ConcurrentDictionary<TKey, TValue>` is best when you have multiple threads reading and writing to a shared collection. If the updates are not constant (if they’re more rare), then `ImmutableDictionary<TKey, TValue>` may be a better choice.

`ConcurrentDictionary<TKey, TValue>` is best in a shared-data situation, where multiple threads share the same collection. If some threads only add elements and other threads only remove elements, you’d be better served by a producer/consumer collection.

`ConcurrentDictionary<TKey, TValue>` isn’t the only threadsafe collection. The BCL also provides `ConcurrentStack<T>`, `ConcurrentQueue<T>`, and `ConcurrentBag<T>`. Threadsafe collections are commonly used as producer/consumer collections, which will be covered in the rest of this chapter.

## See Also

[Recipe 9.4](#recipe-immutable-dictionaries) covers immutable dictionaries, which are ideal if the contents of the dictionary change very rarely.

# 9.6 Blocking Queues

## Problem

You need a conduit to pass messages or data from one thread to another. For example, one thread could be loading data, which it pushes down the conduit as it loads; meanwhile, there are other threads on the receiving end of the conduit that receive the data and process it.

## Solution

The .NET type `BlockingCollection<T>` was designed to be this kind of conduit. By default, `BlockingCollection<T>` is a blocking queue, providing first-in, first-out behavior.

A blocking queue needs to be shared by multiple threads, and it’s usually defined as a private, read-only field:

[PRE13]

Usually, a thread will *either* add items to the collection *or* remove items from the collection, but not both. Threads that add items are called *producer threads*, and threads that remove items are called *consumer threads*.

Producer threads can add items by calling `Add`, and when the producer thread is finished (when all items have been added), it can then finish the collection by calling `CompleteAdding`. This notifies the collection that no more items will be added to it, and the collection can then inform its consumers that there are no more items.

Here’s a simple example of a producer that adds two items and then marks the collection complete:

[PRE14]

Consumer threads usually run in a loop, waiting for the next item and then processing it. If you take the producer code and put it in a separate thread (e.g., via `Task.Run`), then you can consume those items like this:

[PRE15]

If you want to have multiple consumers, `GetConsumingEnumerable` can be called from multiple threads at the same time. However, each item is only passed to one of those threads. When the collection is completed, the enumerable completes.

## Discussion

The preceding examples all use `GetConsumingEnumerable` for the consumer threads; this is the most common scenario. However, there’s also a `Take` member that enables a consumer to just consume a single item rather than run a loop consuming all the items.

When you use conduits like this, you do need to consider what happens if your producers run faster than your consumers. If you’re producing items faster than you can consume them, then you may need to throttle your queue.

Blocking queues are great when you have a separate thread (such as a threadpool thread) acting as a producer or consumer. They’re not as great when you want to access the conduit asynchronously—for example, if a UI thread wants to act as a consumer. [Recipe 9.8](#recipe-async-queues) covers asynchronous queues.

###### Tip

Whenever you introduce a conduit like this into your application, consider switching to the TPL Dataflow library. A lot of the time, using TPL Dataflow is simpler than building your own conduits and background threads.

`BufferBlock<T>` from TPL Dataflow can act like a blocking queue, and TPL Dataflow allows building a pipeline or mesh for processing. In many simpler cases, though, ordinary blocking queues like `BlockingCollection<T>` are the appropriate design choice.

You could also use `AsyncEx` library’s `AsyncProducerConsumerQueue<T>`, which can act like a blocking queue.

## See Also

[Recipe 9.7](#recipe-blocking-stacks-bags) covers blocking stacks and bags, if you want a similar conduit without first-in, first-out semantics.

[Recipe 9.8](#recipe-async-queues) covers queues that have asynchronous rather than blocking APIs.

[Recipe 9.12](#recipe-blocking-async-queues) covers queues that have both asynchronous and blocking APIs.

[Recipe 9.9](#recipe-queue-throttle) covers queues that throttle their number of items.

# 9.7 Blocking Stacks and Bags

## Problem

You need a conduit to pass messages or data from one thread to another, but you don’t want (or need) the conduit to have first-in, first-out semantics.

## Solution

The .NET type `BlockingCollection<T>` acts as a blocking queue by default, but it can also act like any kind of producer/consumer collection. It’s actually a wrapper around a threadsafe collection that implements `IProducerConsumerCollection<T>`.

So, you can create a `BlockingCollection<T>` with last-in, first-out (stack) semantics or unordered (bag) semantics:

[PRE16]

It’s important to keep in mind that there are now race conditions around the ordering of the items. If you let the same producer code execute before any consumer code, and then execute the consumer code after the producer code, then the order of the items will be exactly like a stack:

[PRE17]

When the producer code and consumer code are on different threads (which is the usual case), the consumer always gets the most recently added item next. For example, the producer could add `7`, the consumer could take `7`, the producer could add `13`, and the consumer could take `13`. The consumer does *not* wait for `CompleteAdding` to be called before it returns the first item.

## Discussion

The same considerations around throttling that apply to blocking queues also apply to blocking stacks and bags. If your producers run faster than your consumers and you need to limit the memory usage of your blocking stack/bag, you can use throttling as shown in [Recipe 9.9](#recipe-queue-throttle).

This recipe uses `GetConsumingEnumerable` for the consumer code; this is the most common scenario. There is also a `Take` member that enables a consumer to just consume a single item rather than run a loop consuming all the items.

If you want to access shared stacks or bags asynchronously rather than by blocking (for example, having your UI thread act as a consumer), see [Recipe 9.11](#recipe-async-stacks-bags).

## See Also

[Recipe 9.6](#recipe-blocking-queues) covers blocking queues, which are much more commonly used than blocking stacks or bags.

[Recipe 9.11](#recipe-async-stacks-bags) covers asynchronous stacks and bags.

# 9.8 Asynchronous Queues

## Problem

You need a conduit to pass messages or data from one part of code to another in a first-in, first-out manner, without blocking threads.

For example, one piece of code could be loading data, which it pushes down the conduit as it loads; meanwhile, the UI thread is receiving the data and displaying it.

## Solution

What you need is a queue with an asynchronous API. There is no type like this in the core .NET framework, but there are a couple of options available from NuGet.

The first option is to use Channels. Channels are a modern library for asynchronous producer/consumer collections, with a nice emphasis on high performance for high-volume scenarios. Producers generally write items to a channel using `WriteAsync`, and when they are all done producing, one of them calls `Complete` to notify the channel that there won’t be any more items in the future, like this:

[PRE18]

This more natural consumer code uses asynchronous streams; see [Chapter 3](ch03.html#async-streams) for more information. As of this writing, asynchronous streams are only available on the newest .NET platforms; older platforms can use the following pattern:

[PRE19]

Note the double `while` loop in the consumer code for older platforms; this is normal. `WaitToReadAsync` will asynchronously wait until an item is available or the channel has been marked complete; it returns `true` when there is an item available to be read. `TryRead` will attempt to read an item (immediately and synchronously), returning `true` if an item was read. If `TryRead` returns `false`, this could be because there’s no item available *right now*, or it could be because the channel has been marked complete and there will never be any more items. So, when `TryRead` returns `false`, the inner `while` loop exits and the consumer again calls `WaitToReadAsync`, which will return `false` if the channel has been marked complete.

Another producer/consumer queue option is to use `BufferBlock<T>` from the TPL Dataflow library. `BufferBlock<T>` is quite similar to a channel. The following example shows how to declare a `BufferBlock<T>`, what the producer code looks like, and what the consumer code looks like:

[PRE20]

The example consumer code uses `OutputAvailableAsync`, which is really only useful if you have just a single consumer. If you have multiple consumers, it is possible that `OutputAvailableAsync` will return `true` for more than one consumer even though there is only one item. If the queue is completed, then `ReceiveAsync` will throw `InvalidOperationException`. So if you have multiple consumers, the consumer code usually looks more like the following:

[PRE21]

You can also use the `AsyncProducerConsumerQueue<T>` type from the `Nito.AsyncEx` NuGet library. The API is similar to but not exactly the same as `BufferBlock<T>`:

[PRE22]

This consumer code also uses `OutputAvailableAsync` and has the same problems as `BufferBlock<T>`. If you have multiple consumers, the consumer code usually looks more like the following:

[PRE23]

## Discussion

I recommend using Channels for asynchronous producer/consumer queues whenever possible. They have multiple sampling options in addition to throttling, and they are highly optimized. However, if your application logic can be expressed as a “pipeline” through which data flows, then TPL Dataflow may be a more natural fit. The final option is `AsyncProducerConsumerQueue<T>`, which may make sense if your application is already using other types from `AsyncEx`.

###### Tip

Channels can be found in the [`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet package. The `BufferBlock<T>` type is in the [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) NuGet package. The `AsyncProducerConsumerQueue<T>` type is in the [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet package.

## See Also

[Recipe 9.6](#recipe-blocking-queues) covers producer/consumer queues with blocking semantics rather than asynchronous semantics.

[Recipe 9.12](#recipe-blocking-async-queues) covers producer/consumer queues that have *both* blocking and asynchronous semantics.

[Recipe 9.7](#recipe-blocking-stacks-bags) covers asynchronous stacks and bags if you want a similar conduit without first-in, first-out semantics.

# 9.9 Throttling Queues

## Problem

You have a producer/consumer queue, and your producers might run faster than your consumers, which would cause undesired memory usage. You also want to keep all the queue items, so you need a way to throttle the producers.

## Solution

When you use producer/consumer queues, you do need to consider what happens if your producers run faster than your consumers, unless you’re sure that your consumers will *always* run faster. If you’re producing items faster than you can consume them, then you may need to throttle your queue. You can throttle a queue by designating a maximum number of elements. When a queue is “full,” it applies backpressure to the producers, blocking them until there is more room in the queue.

Channels can be throttled by creating a bounded channel rather than an unbounded channel. Since channels are asynchronous, producers will be asynchronously throttled:

[PRE24]

`BufferBlock<T>` has built-in support for throttling, explored in more detail in [Recipe 5.4](ch05.html#recipe-dataflow-throttling). With dataflow blocks, you set the `BoundedCapacity` option:

[PRE25]

The producer in the preceding code snippet uses the asynchronous `SendAsync` API; the same approach works for the synchronous `Post` API.

The `AsyncEx` type `AsyncProducerConsumerQueue<T>` has support for throttling. Just construct the queue with the appropriate value:

[PRE26]

Blocking producer/consumer queues also support throttling. You can use `BlockingCollection<T>` to throttle the number of items by passing the appropriate value when you create it:

[PRE27]

## Discussion

Throttling is necessary whenever producers can run faster than consumers. One scenario you must consider is whether it’s possible for producers to run faster than consumers if your application is running on different hardware than yours. Some throttling is usually necessary to ensure your application will run on future hardware and/or cloud instances, which are generally more constrained than developer machines.

Throttling will cause backpressure on the producers, slowing them down to ensure that consumers are able to process all items, without causing undue memory pressure. If you don’t need to process *every* item, you can choose to sample instead of throttle. See [Recipe 9.10](#recipe-queue-sample) for sampling producer/consumer queues.

###### Tip

Channels are in the [`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet package. The `BufferBlock<T>` type is in the [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) NuGet package. The `AsyncProducerConsumerQueue<T>` type is in the [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet package.

## See Also

[Recipe 9.8](#recipe-async-queues) covers basic asynchronous producer/consumer queue usage.

[Recipe 9.6](#recipe-blocking-queues) covers basic synchronous producer/consumer queue usage.

[Recipe 9.10](#recipe-queue-sample) covers sampling producer/consumer queues, an alternative to throttling.

# 9.10 Sampling Queues

## Problem

You have a producer/consumer queue, but your producers may run faster than your consumers, which is causing undesired memory usage. You don’t need to keep all the queue items; you need a way to filter the queue items so that the slower producers only need to process the important ones.

## Solution

Channels are the easiest way to apply sampling to input items. One common example is to always take the latest *n* items, discarding the oldest items once the queue is full:

[PRE28]

This is an easy way to tame input streams, keeping them from flooding your consumers.

There are other `BoundedChannelFullMode` options as well. For example, if you wanted the *oldest* items to be preserved, you could discard any new items once the channel is full:

[PRE29]

## Discussion

Channels are great for doing simple sampling like this. A particularly useful option in many situations is `BoundedChannelFullMode.DropOldest`. More complex sampling would need to be done by the consumers themselves.

If you need to do time-based sampling, such as “only 10 items per second,” use System.Reactive. System.Reactive has natural operators for working with time.

###### Tip

Channels are located in the [`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet package.

## See Also

[Recipe 9.9](#recipe-queue-throttle) covers throttling channels, which limits the number of items in the channel by blocking producers rather than dropping items.

[Recipe 9.8](#recipe-async-queues) covers basic channel usage, including producer and consumer code.

[Recipe 6.4](ch06.html#recipe-rx-throttle) covers throttling and sampling using `System.Reactive`, which supports time-based sampling.

# 9.11 Asynchronous Stacks and Bags

## Problem

You need a conduit to pass messages or data from one part of code to another, but you don’t want (or need) the conduit to have first-in, first-out semantics.

## Solution

The `Nito.AsyncEx` library provides a type `AsyncCollection<T>`, which acts like an asynchronous queue by default, but it can also act like any kind of producer/consumer collection. The wrapper around an `IProducerConsumerCollection<T>`, `AsyncCollection<T>` is also the `async` equivalent of the .NET `BlockingCollection<T>`, which is covered in [Recipe 9.7](#recipe-blocking-stacks-bags).

`AsyncCollection<T>` supports last-in, first-out (stack) or unordered (bag) semantics, based on whatever collection you pass to its constructor:

[PRE30]

Note that there’s a race condition around the ordering of items in the stack. If all producers complete before consumers start, then the order of items is like a regular stack:

[PRE31]

When both producers and consumers are executing concurrently (which is the usual case), the consumer will always get the most recently added item next. This will cause the collection as a whole to act not quite like a stack. Of course, the bag collection has no ordering at all.

`AsyncCollection<T>` has support for throttling, which is necessary if producers may add to the collection faster than the consumers can remove from it. Just construct the collection with the appropriate value:

[PRE32]

Now the same producer code will asynchronously wait as needed:

[PRE33]

The example consumer code uses `OutputAvailableAsync`, which has the same limitation described in [Recipe 9.8](#recipe-async-queues). If you have multiple consumers, the consumer code usually looks more like the following:

[PRE34]

## Discussion

`AsyncCollection<T>` is just the asynchronous equivalent of `BlockingCollection<T>` with a slightly different API.

###### Tip

The `AsyncCollection<T>` type is in the [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet package.

## See Also

[Recipe 9.8](#recipe-async-queues) covers asynchronous queues, which are much more common than asynchronous stacks or bags.

[Recipe 9.7](#recipe-blocking-stacks-bags) covers synchronous (blocking) stacks and bags.

# 9.12 Blocking/Asynchronous Queues

## Problem

You need a conduit to pass messages or data from one part of code to another in a first-in, first-out manner, and you need the flexibility to treat either the producer end or the consumer end as synchronous or asynchronous.

For example, a background thread may be loading data and pushing it into the conduit, and you want the background thread to synchronously block if the conduit is too full. At the same time, the UI thread is receiving data from the conduit, and you want the UI thread to asynchronously pull data from the conduit so the UI remains responsive.

## Solution

After looking at blocking queues in [Recipe 9.6](#recipe-blocking-queues) and asynchronous queues in [Recipe 9.8](#recipe-async-queues), now we’ll learn about a few queue types that support both blocking and asynchronous APIs.

The first is `BufferBlock<T>` and `ActionBlock<T>` from the TPL Dataflow NuGet library. `BufferBlock<T>` can be easily used as an asynchronous producer/consumer queue (see [Recipe 9.8](#recipe-async-queues) for more details):

[PRE35]

As you can see in the following example, `BufferBlock<T>` also supports a synchronous API for both producers and consumers:

[PRE36]

The consumer code using `BufferBlock<T>` is rather awkward, since it isn’t the “dataflow way” of writing code. The TPL Dataflow library includes a number of blocks that can be linked together, enabling you to define a reactive mesh. In this case, a producer/consumer queue completing with a particular action can be defined using `ActionBlock<T>`:

[PRE37]

If the TPL Dataflow library isn’t available on your desired platform(s), then there is an `AsyncProducerConsumerQueue<T>` type in `Nito.AsyncEx` that also supports both synchronous and asynchronous methods:

[PRE38]

## Discussion

I recommend using `BufferBlock<T>` or `ActionBlock<T>` if possible because the TPL Dataflow library has been more extensively tested than the `Nito.AsyncEx` library. However, `AsyncProducerConsumerQueue<T>` may be useful if your application is already using other types from the `AsyncEx` library.

It is also possible to use `System.Threading.Channels` synchronously, but only indirectly. Their natural API is asynchronous, but since they are threadsafe collections, you can force them to work synchronously by wrapping your production or consumption code inside a `Task.Run` and then blocking on the task returned from `Task.Run`, like this:

[PRE39]

TPL Dataflow blocks, `AsyncProducerConsumerQueue<T>`, and Channels all support throttling by passing options during construction. Throttling is necessary when you have producers that push items faster than your consumers can consume them, which could cause your application to take up large amounts of memory.

###### Tip

The `BufferBlock<T>` and `ActionBlock<T>` types are in the [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) NuGet package. The `AsyncProducerConsumerQueue<T>` type is in the [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet package. Channels are in the [`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet package.

## See Also

[Recipe 9.6](#recipe-blocking-queues) covers blocking producer/consumer queues.

[Recipe 9.8](#recipe-async-queues) covers asynchronous producer/consumer queues.

[Recipe 5.4](ch05.html#recipe-dataflow-throttling) covers throttling dataflow blocks.