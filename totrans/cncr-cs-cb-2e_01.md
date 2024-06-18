# Chapter 1\. Concurrency: An Overview

Concurrency is a key aspect of beautiful software. For decades, concurrency was possible but difficult to achieve. Concurrent software was difficult to write, difficult to debug, and difficult to maintain. As a result, many developers chose the easier path and avoided concurrency. With the libraries and language features available for modern .NET programs, concurrency is now much easier. Microsoft has led the way in significantly lowering the bar for concurrency. Previously, concurrent programming was the domain of experts; these days, every developer can (and should) embrace concurrency.

# Introduction to Concurrency

Before continuing, I’d like to clear up some terminology that I’ll be using throughout this book. These are my own definitions that I use consistently to disambiguate different programming techniques. Let’s start with *concurrency*.

Concurrency

Doing more than one thing at a time.

I hope it’s obvious how concurrency is helpful. End-user applications use concurrency to respond to user input *while* writing to a database. Server applications use concurrency to respond to a second request *while* finishing the first request. You need concurrency any time you need an application to do one thing *while* it’s working on something else. Almost every software application in the world can benefit from concurrency.

Most developers hearing the term “concurrency” immediately think of “multithreading.” I’d like to draw a distinction between these two.

Multithreading

A form of concurrency that uses multiple threads of execution.

Multithreading refers to literally using multiple threads. As demonstrated in many recipes in this book, multithreading is *one* form of concurrency, but certainly not the only one. In fact, direct use of low-level threading types has almost no purpose in a modern application; higher-level abstractions are more powerful and more efficient than old-school multithreading. For that reason, I’ll minimize my coverage of outdated techniques. None of the multithreading recipes in this book use the `Thread` or `BackgroundWorker` types; they have been replaced with superior alternatives.

###### Warning

As soon as you type `new Thread()`, it’s over; your project already has legacy code.

But don’t get the idea that multithreading is dead! Multithreading lives on in the *thread pool*, a useful place to queue work that automatically adjusts itself according to demand. In turn, the thread pool enables another important form of concurrency: *parallel processing*.

Parallel processing

Doing lots of work by dividing it up among multiple threads that run concurrently.

Parallel processing (or parallel programming) uses multithreading to maximize the use of multiple processor cores. Modern CPUs have multiple cores, and if there’s a lot of work to do, then it makes no sense to make one core do all the work while the others sit idle. Parallel processing splits the work among multiple threads, which can each run independently on a different core.

Parallel processing is one type of multithreading, and multithreading is one type of concurrency. There’s another type of concurrency that is important in modern applications but isn’t as familiar to many developers: *asynchronous programming*.

Asynchronous programming

A form of concurrency that uses futures or callbacks to avoid unnecessary threads.

A *future* (or *promise*) is a type that represents some operation that will complete in the future. Some modern future types in .NET are `Task` and `Task<TResult>`. Older asynchronous APIs use callbacks or events instead of futures. Asynchronous programming is centered around the idea of an *asynchronous operation*: some operation that is started that will complete some time later. While the operation is in progress, it doesn’t block the original thread; the thread that starts the operation is free to do other work. When the operation completes, it notifies its future or invokes its callback or event to let the application know the operation is finished.

Asynchronous programming is a powerful form of concurrency, but until recently, it required extremely complex code. The `async` and `await` support in modern languages make asynchronous programming almost as easy as synchronous (nonconcurrent) programming.

Another form of concurrency is *reactive programming*. Asynchronous programming implies that the application will start an operation that will complete once at a later time. Reactive programming is closely related to asynchronous programming but is built on *asynchronous events* instead of *asynchronous operations*. Asynchronous events may not have an actual “start,” may happen at any time, and may be raised multiple times. One example is user input.

Reactive programming

A declarative style of programming where the application reacts to events.

If you consider an application to be a massive state machine, the application’s behavior can be described as reacting to a series of events by updating its state at each event. This isn’t as abstract or theoretical as it sounds; modern frameworks make this approach quite useful in real-world applications. Reactive programming isn’t necessarily concurrent, but it is closely related to concurrency, so this book covers the basics.

Usually, a mixture of techniques is used when writing a concurrent program. Most applications at least use multithreading (via the thread pool) and asynchronous programming. Feel free to mix and match all the various forms of concurrency, using the appropriate tool for each part of the application.

# Introduction to Asynchronous Programming

Asynchronous programming has two primary benefits. The first benefit is for end-user GUI programs: asynchronous programming enables responsiveness. Everyone has used a program that temporarily locks up while it’s working; an asynchronous program can remain responsive to user input while it’s working. The second benefit is for server-side programs: asynchronous programming enables scalability. A server application can scale somewhat just by using the thread pool, but an asynchronous server application can usually scale an order of magnitude better than that.

Both benefits of asynchronous programming derive from the same underlying aspect: asynchronous programming frees up a thread. For GUI programs, asynchronous programming frees up the UI thread; this permits the GUI application to remain responsive to user input. For server applications, asynchronous programming frees up request threads; this permits the server to use its threads to serve more requests.

Modern asynchronous .NET applications use two keywords: `async` and `await`. The `async` keyword is added to a method declaration, and performs a double purpose: it enables the `await` keyword within that method and it signals the compiler to generate a state machine for that method, similar to how `yield return` works. An `async` method may return `Task<TResult>` if it returns a value, `Task` if it doesn’t return a value, or any other “task-like” type, such as `ValueTask`. In addition, an `async` method may return `IAsyncEnumerable<T>` or `IAsyncEnumerator<T>` if it returns multiple values in an enumeration. The task-like types represent futures; they can notify the calling code when the `async` method completes.

###### Warning

Avoid `async void`! It is possible to have an `async` method return `void`, but you should only do this if you’re writing an `async` event handler. A regular `async` method without a return value should return `Task`, not `void`.

With that background, let’s take a quick look at an example:

[PRE0]

An `async` method begins executing synchronously, just like any other method. Within an `async` method, the `await` keyword performs an *asynchronous wait* on its argument. First, it checks whether the operation is already complete; if it is, it continues executing (synchronously). Otherwise, it will pause the `async` method and return an incomplete task. When that operation completes some time later, the `async` method will resume executing.

You can think of an `async` method as having several synchronous portions, broken up by `await` statements. The first synchronous portion executes on whatever thread calls the method, but where do the other synchronous portions execute? The answer is a bit complicated.

When you `await` a task (the most common scenario), a *context* is captured when the `await` decides to pause the method. This is the current `SynchronizationContext` unless it’s `null`, in which case the context is the current `TaskScheduler`. The method resumes executing within that captured context. Usually, this context is the UI context (if you’re on the UI thread) or the threadpool context (most other situations). If you have an ASP.NET Classic (pre-Core) application, then the context could also be an ASP.NET request context. ASP.NET Core uses the threadpool context rather than a special request context.

So, in the preceding code, all the synchronous portions will attempt to resume on the original context. If you call `DoSomethingAsync` from a UI thread, each of its synchronous portions will run on that UI thread; but if you call it from a threadpool thread, each of its synchronous portions will run on any threadpool thread.

You can avoid this default behavior by awaiting the result of the `ConfigureAwait` extension method and passing `false` for the `continueOnCapturedContext` parameter. The following code will start on the calling thread, and after it is paused by an `await`, it’ll resume on a threadpool thread:

[PRE1]

###### Tip

It’s good practice to always call `ConfigureAwait` in your core “library” methods, and only resume the context when you need it—in your outer “user interface” methods.

The `await` keyword is not limited to working with tasks; it can work with any kind of *awaitable* that follows a certain pattern. As an example, the Base Class Library includes the `ValueTask<T>` type, which reduces memory allocations if the result is commonly synchronous; for example, if the result can be read from an in-memory cache. `ValueTask<T>` is not directly convertible to `Task<T>`, but it does follow the awaitable pattern, so you can directly `await` it. There are other examples, and you can build your own, but most of the time `await` will take a `Task` or `Task<TResult>`.

There are two basic ways to create a `Task` instance. Some tasks represent actual code that a CPU has to execute; these computational tasks should be created by calling `Task.Run` (or `TaskFactory.StartNew` if you need them to run on a particular scheduler). Other tasks represent a *notification*; these kinds of event-based tasks are created by `TaskCompletionSource<TResult>` (or one of its shortcuts). Most I/O tasks use `TaskCompletionSource<TResult>`.

Error handling is natural with `async` and `await`. In the code snippet that follows, `PossibleExceptionAsync` may throw a `NotSupportedException`, but `TrySomethingAsync` can catch the exception naturally. The caught exception has its stack trace properly preserved and isn’t artificially wrapped in a `TargetInvocationException` or `AggregateException`:

[PRE2]

When an `async` method throws (or propagates) an exception, the exception is placed on its returned `Task` and the `Task` is completed. When that `Task` is awaited, the `await` operator will retrieve that exception and (re)throw it in a way such that its original stack trace is preserved. Thus, code such as the following example would work as expected if `PossibleExceptionAsync` was an `async` method:

[PRE3]

There’s one other important guideline when it comes to `async` methods: once you start using `async`, it’s best to allow it to grow through your code. If you call an `async` method, you should (eventually) `await` the task it returns. Resist the temptation to call `Task.Wait`, `Task<TResult>.Result`, or `GetAwaiter().GetResult()`; doing so could cause a deadlock. Consider the following method:

[PRE4]

The code in this example will deadlock if called from a UI or ASP.NET Classic context because both of those contexts only allow one thread in at a time. `Deadlock` will call `WaitAsync`, which begins the delay. `Deadlock` then (synchronously) waits for that method to complete, blocking the context thread. When the delay completes, `await` attempts to resume `WaitAsync` within the captured context, but it cannot because there’s already a thread blocked in the context, and the context only allows one thread at a time. Deadlock can be prevented two ways: you can use `ConfigureAwait(false)` within `WaitAsync` (which causes `await` to ignore its context), or you can `await` the call to `WaitAsync` (making `Deadlock` into an `async` method).

###### Warning

If you use `async`, it’s best to use `async` all the way.

For a more complete introduction to `async`, the online documentation that Microsoft has provided for `async` is fantastic; I recommend reading at least the [Asynchronous Programming overview](http://bit.ly/async-prog) and the [Task-based Asynchronous Pattern (TAP)](http://bit.ly/task-async-patt) overview. If you want to go a bit deeper, there’s also the [Async in Depth](http://bit.ly/async-indepth) documentation.

Asynchronous streams take the groundwork of `async` and `await` and extend it to handle multiple values. Asynchronous streams are built around the concept of asynchronous enumerables, which are like regular enumerables, except that they enable asynchronous work to be done when retrieving the next item in the sequence. This is an extremely powerful concept that [Chapter 3](ch03.html#async-streams) covers in more detail. Asynchronous streams are especially useful whenever you have a sequence of data that arrives either one at a time or in chunks. For example, if your application processes the response of an API that uses paging with `limit` and `offset` parameters, then asynchronous streams are an ideal abstraction. As of the time of this writing, asynchronous streams are only available on the newest .NET platforms.

# Introduction to Parallel Programming

Parallel programming should be used any time you have a fair amount of computation work that can be split up into independent chunks. Parallel programming increases the CPU usage temporarily to improve throughput; this is desirable on client systems where CPUs are often idle, but it’s usually not appropriate for server systems. Most servers have some parallelism built in; for example, ASP.NET will handle multiple requests in parallel. Writing parallel code on the server may still be useful in some situations (if you *know* that the number of concurrent users will always be low), but in general, parallel programming on the server would work against its built-in parallelism and therefore wouldn’t provide any real benefit.

There are two forms of parallelism: *data parallelism* and *task parallelism*. Data parallelism is when you have a bunch of data items to process, and the processing of each piece of data is mostly independent from the other pieces. Task parallelism is when you have a pool of work to do, and each piece of work is mostly independent from the other pieces. Task parallelism may be dynamic; if one piece of work results in several additional pieces of work, they can be added to the pool of work.

There are a few different ways to do data parallelism. `Parallel.ForEach` is similar to a `foreach` loop and should be used when possible. `Parallel.ForEach` is covered in [Recipe 4.1](ch04.html#recipe-parallel-foreach). The `Parallel` class also supports `Parallel.For`, which is similar to a `for` loop, and can be used if the data processing depends on the index. Code that uses `Parallel.ForEach` looks like the following:

[PRE5]

Another option is PLINQ (Parallel LINQ), which provides an `AsParallel` extension method for LINQ queries. `Parallel` is more resource friendly than PLINQ; `Parallel` will play more nicely with other processes in the system, while PLINQ will (by default) attempt to spread itself over all CPUs. The downside to `Parallel` is that it’s more explicit; PLINQ in many cases has more elegant code. PLINQ is covered in [Recipe 4.5](ch04.html#recipe-parallel-plinq) and looks like this:

[PRE6]

Regardless of the method you choose, one guideline stands out when doing parallel processing.

###### Tip

The chunks of work should be as independent from one another as possible.

As long as your chunk of work is independent from all other chunks, you maximize your parallelism. As soon as you start sharing state between multiple threads, you have to synchronize access to that shared state, and your application becomes less parallel. [Chapter 12](ch12.html#synchronization) covers synchronization in more detail.

The output of your parallel processing can be handled in various ways. You can place the results in some kind of a concurrent collection, or you can aggregate the results into a summary. Aggregation is common in parallel processing; this kind of map/reduce functionality is also supported by the `Parallel` class method overloads. [Recipe 4.2](ch04.html#recipe-parallel-aggregate) looks at aggregation in more detail.

Now let’s turn to task parallelism. Data parallelism is focused on processing data; task parallelism is just about doing work. At a high level, data parallelism and task parallelism are similar; “processing data” is a kind of “work.” Many parallelism problems can be solved either way; it’s convenient to use whichever API is more natural for the problem at hand.

`Parallel.Invoke` is one type of `Parallel` method that does a kind of fork/join task parallelism. This method is covered in [Recipe 4.3](ch04.html#recipe-parallel-invoke); you just pass in the delegates you want to execute in parallel:

[PRE7]

The `Task` type was originally introduced for task parallelism, though these days it’s also used for asynchronous programming. A `Task` instance—as used in task parallelism—represents some work. You can use the `Wait` method to wait for a task to complete, and you can use the `Result` and `Exception` properties to retrieve the results of that work. Code using `Task` directly is more complex than code using `Parallel`, but it can be useful if you don’t know the structure of the parallelism until runtime. With this kind of dynamic parallelism, you don’t know how many pieces of work you need to do at the beginning of the processing; you find out as you go along. Generally, a dynamic piece of work should start whatever child tasks it needs and then wait for them to complete. The `Task` type has a special flag, `TaskCreationOptions.AttachedToParent`, which you could use for this. Dynamic parallelism is covered in [Recipe 4.4](ch04.html#recipe-parallel-dynamicparallelism).

Task parallelism should strive to be independent, just like data parallelism. The more independent your delegates can be, the more efficient your program can be. Also, if your delegates aren’t independent, then they need to be synchronized, and it’s harder to write correct code if that code needs synchronization. With task parallelism, be especially careful of variables captured in closures. Remember that closures capture references (not values), so you can end up with sharing that isn’t obvious.

Error handling is similar for all kinds of parallelism. Because operations are proceeding in parallel, it’s possible for multiple exceptions to occur, so they are wrapped up in an `AggregateException` that’s thrown to your code. This behavior is consistent across `Parallel.ForEach`, `Parallel.Invoke`, `Task.Wait`, etc. The `AggregateException` type has some useful `Flatten` and `Handle` methods to simplify the error handling code:

[PRE8]

Usually, you don’t have to worry about how the work is handled by the thread pool. Data and task parallelism use dynamically adjusting partitioners to divide work among worker threads. The thread pool increases its thread count as necessary. The thread pool has a single work queue, and each threadpool thread also has its own work queue. When a threadpool thread queues additional work, it sends it to its own queue first because the work is usually related to the current work item; this behavior encourages threads to work on their own work, and maximizes cache hits. If another thread doesn’t have work to do, it’ll steal work from another thread’s queue. Microsoft put a lot of work into making the thread pool as efficient as possible, and there are a large number of knobs you can tweak if you need maximum performance. As long as your tasks are not extremely short, they should work well with the default settings.

###### Tip

Tasks should neither be extremely short, nor extremely long.

If your tasks are too short, then the overhead of breaking up the data into tasks and scheduling those tasks on the thread pool becomes significant. If your tasks are too long, then the thread pool cannot dynamically adjust its work balancing efficiently. It’s difficult to determine how short is too short and how long is too long; it really depends on the problem being solved and the approximate capabilities of the hardware. As a general rule, I try to make my tasks as short as possible without running into performance issues (you’ll see your performance suddenly degrade when your tasks are too short). Even better, instead of using tasks directly, use the `Parallel` type or PLINQ. These higher-level forms of parallelism have partitioning built in to handle this automatically for you (and adjust as necessary at runtime).

If you want to dive deeper into parallel programming, the best book on the subject is *Parallel Programming with Microsoft .NET*, by Colin Campbell et al. (Microsoft Press).

# Introduction to Reactive Programming (Rx)

Reactive programming has a higher learning curve than other forms of concurrency, and the code can be harder to maintain unless you keep up with your reactive skills. If you’re willing to learn it, though, reactive programming is extremely powerful. Reactive programming enables you to treat a stream of events like a stream of data. As a rule of thumb, if you use any of the event arguments passed to an event, then your code would benefit from using System.Reactive instead of a regular event handler.

###### Tip

System.Reactive used to be called Reactive Extensions, which was often shortened to “Rx.” All three of these terms refer to the same technology.

Reactive programming is based on the notion of observable streams. When you subscribe to an observable stream, you’ll receive any number of data items (`OnNext`), and then the stream may end with a single error (`OnError`) or “end of stream” notification (`OnCompleted`). Some observable streams never end. The actual interfaces look like the following:

[PRE9]

However, you should never implement these interfaces. The System.Reactive (Rx) library by Microsoft has all the implementations you should ever need. Reactive code ends up looking very much like LINQ; you can think of it as “LINQ to Events.” System.Reactive has everything that LINQ does and adds in a large number of its own operators, particularly ones that deal with time. The following code starts with some unfamiliar operators (`Interval` and `Timestamp`) and ends with a `Subscribe`, but in the middle are some `Where` and `Select` operators that should be familiar from LINQ:

[PRE10]

The example code starts with a counter running off a periodic timer (`Interval`) and adds a timestamp to each event (`Timestamp`). It then filters the events to only include even counter values (`Where`), selects the timestamp values (`Timestamp`), and then as each resulting timestamp value arrives, writes it to the debugger (`Subscribe`). Don’t worry if you don’t understand the new operators, such as `Interval`: these are covered later in this book. For now, just keep in mind that this is a LINQ query very similar to the ones you’re already familiar with. The main difference is that LINQ to Objects and LINQ to Entities use a *“pull” model*, where the enumeration of a LINQ query pulls the data through the query, while LINQ to Events (System.Reactive) uses a *“push” model*, where the events arrive and travel through the query by themselves.

The definition of an observable stream is independent from its subscriptions. The last example is the same as the following code:

[PRE11]

It is normal for a type to define the observable streams and make them available as an `IObservable<TResult>` resource. Other types can then subscribe to those streams or combine them with other operators to create another observable stream.

A System.Reactive subscription is also a resource. The `Subscribe` operators return an `IDisposable` that represents the subscription. When your code is done listening to an observable stream, it should dispose its subscription.

Subscriptions behave differently with hot and cold observables. A *hot observable* is a stream of events that is always going on, and if there are no subscribers when the events come in, they are lost. For example, mouse movement is a hot observable. A *cold observable* is an observable that doesn’t have incoming events all the time. A cold observable will react to a subscription by starting the sequence of events. For example, an HTTP download is a cold observable; the subscription causes the HTTP request to be sent.

The `Subscribe` operator should always take an error handling parameter as well. The preceding examples do not; the following is a better example that will respond appropriately if the observable stream ends in an error:

[PRE12]

`Subject<TResult>` is one type that is useful when experimenting with System.Reactive. This “subject” is like a manual implementation of an observable stream. Your code can call `OnNext`, `OnError`, and `OnCompleted`, and the subject will forward those calls to its subscribers. `Subject<TResult>` is great for experimenting, but in production code, you should strive to use operators like those covered in [Chapter 6](ch06.html#rx-basics).

There are tons of useful System.Reactive operators, and I only cover a few selected ones in this book. For more information on System.Reactive, I recommend the excellent online book [*Introduction to Rx*](http://www.introtorx.com/).

# Introduction to Dataflows

TPL Dataflow is an interesting mix of asynchronous and parallel technologies. It’s useful when you have a sequence of processes that need to be applied to your data. For example, you may need to download data from a URL, parse it, and then process it in parallel with other data. TPL Dataflow is commonly used as a simple pipeline, where data enters one end and travels until it comes out the other. However, TPL Dataflow is far more powerful than this; it’s capable of handling any kind of mesh. You can define forks, joins, and loops in a mesh, and TPL Dataflow will handle them appropriately. Most of the time, though, TPL Dataflow meshes are used as a pipeline.

The basic building unit of a dataflow mesh is a *dataflow block*. A block can either be a target block (receiving data), a source block (producing data), or both. Source blocks can be linked to target blocks to create the mesh; linking is covered in [Recipe 5.1](ch05.html#recipe-dataflow-linking). Blocks are semi-independent; they will attempt to process data as it arrives and push the results downstream. The usual way of using TPL Dataflow is to create all the blocks, link them together, and then start putting data in at one end. The data then comes out of the other end by itself. Again, Dataflow is more powerful than this; it’s possible to break links and create new blocks and add them to the mesh *while* there is data flowing through it, but that is a very advanced scenario.

Target blocks have buffers for the data they receive. Having buffers enables them to accept new data items even if they aren’t ready to process them yet; this keeps data flowing through the mesh. This buffering can cause problems in fork scenarios, where one source block is linked to two target blocks. When the source block has data to send downstream, it starts offering it to its linked blocks one at a time. By default, the first target block would just take the data and buffer it, and the second target block would never get any. The fix for this situation is to limit the target block buffers by making them nongreedy; [Recipe 5.4](ch05.html#recipe-dataflow-throttling) covers this.

A block will fault when something goes wrong, for example, if the processing delegate throws an exception when processing a data item. When a block faults, it will stop receiving data. By default, it won’t take down the whole mesh; this enables you to rebuild that part of the mesh or redirect the data. However, this is an advanced scenario; most times, you want the faults to propagate along the links to the target blocks. Dataflow supports this option as well; the only tricky part is that when an exception is propagated along a link, it is wrapped in an `AggregateException`. So, if you have a long pipeline, you could end up with a deeply nested exception; the method `AggregateException.Flatten` can be used to work around this:

[PRE13]

[Recipe 5.2](ch05.html#recipe-dataflow-errors) covers dataflow error handling in more detail.

At first glance, dataflow meshes sound very much like observable streams, and they do have much in common. Both meshes and streams have the concept of data items passing through them. Also, both meshes and streams have the notion of a normal completion (a notification that no more data is coming), as well as a faulting completion (a notification that some error occurred during data processing). But System.Reactive (Rx) and TPL Dataflow do not have the same capabilities. Rx observables are generally better than dataflow blocks when doing anything related to timing. Dataflow blocks are generally better than Rx observables when doing parallel processing. Conceptually, Rx works more like setting up callbacks: each step in the observable directly calls the next step. In contrast, each block in a dataflow mesh is very independent from all the other blocks. Both Rx and TPL Dataflow have their own uses, with some amount of overlap. They also work quite well together; [Recipe 8.8](ch08.html#recipe-rx-interop-dataflow) covers interoperability between Rx and TPL Dataflow.

If you’re familiar with actor frameworks, TPL Dataflow will seem to share similarities with them. Each dataflow block is independent, in the sense that it will spin up tasks to do work as needed, like executing a transformation delegate or pushing output to the next block. You can also set up each block to run in parallel, so that it’ll spin up multiple tasks to deal with additional input. Due to this behavior, each block does have a certain similarity to an actor in an actor framework. However, TPL Dataflow is not a full actor framework; in particular, there’s no built-in support for clean error recovery or retries of any kind. TPL Dataflow is a library with an actor-like feel, but it isn’t a full-featured actor framework.

The most common TPL Dataflow block types are `TransformBlock<TInput, TOutput>` (similar to LINQ’s `Select`), `TransformManyBlock<TInput, TOutput>` (similar to LINQ’s `SelectMany`), and `ActionBlock<TResult>`, which executes a delegate for each data item. For more information on TPL Dataflow, I recommend the [MSDN documentation](http://bit.ly/dataflow-doc) and the [“Guide to Implementing Custom TPL Dataflow Blocks”](http://bit.ly/tpl-dataflow).

# Introduction to Multithreaded Programming

A *thread* is an independent executor. Each process has multiple threads in it, and each of those threads can be doing different things simultaneously. Each thread has its own independent stack but shares the same memory with all the other threads in a process. In some applications, there is one thread that is special. For example, user interface applications have a single special UI thread, and Console applications have a single special main thread.

Every .NET application has a thread pool. The thread pool maintains a number of worker threads that are waiting to execute whatever work you have for them to do. The thread pool is responsible for determining how many threads are in the thread pool at any time. There are dozens of configuration settings you can play with to modify this behavior, but I recommend that you leave it alone; the thread pool has been carefully tuned to cover the vast majority of real-world scenarios.

There is almost no need for you to ever create a new thread yourself. The only time you should ever create a `Thread` instance is if you need an STA thread for COM interop.

A thread is a low-level abstraction. The thread pool is a slightly higher level of abstraction; when code queues work to the thread pool, the thread pool itself will take care of creating a thread if necessary. The abstractions covered in this book are higher still: parallel and dataflow processing queues work to the thread pool as necessary. Code using these higher abstractions is easier to get right than code using low-level abstractions.

For this reason, the `Thread` and `BackgroundWorker` types are not covered at all in this book. They have had their time, and that time is over.

# Collections for Concurrent Applications

There are a couple of collection categories that are useful for concurrent programming: concurrent collections and immutable collections. Both of these collection categories are covered in [Chapter 9](ch09.html#collections). Concurrent collections allow multiple threads to update them simultaneously in a safe way. Most concurrent collections use *snapshots* to enable one thread to enumerate the values while another thread may be adding or removing values. Concurrent collections are usually more efficient than just protecting a regular collection with a lock.

Immutable collections are a bit different. An immutable collection cannot actually be modified; instead, to modify an immutable collection, you create a new collection that represents the modified collection. This sounds horribly inefficient, but immutable collections share as much memory as possible between collection instances, so it’s not as bad as it sounds. The nice thing about immutable collections is that all operations are pure, so they work very well with functional code.

# Modern Design

Most concurrent technologies have one similar aspect: they are functional in nature. I don’t mean *functional* as in “they get the job done,” but rather *functional* as a style of programming that is based on function composition. If you adopt a functional mindset, your concurrent designs will be less convoluted.

One principle of functional programming is *purity* (that is, avoiding side effects). Each piece of the solution takes some value(s) as input and produces some value(s) as output. As much as possible, you should avoid having these pieces depend on global (or shared) variables or update global (or shared) data structures. This is true whether the piece is an `async` method, a parallel task, a System.Reactive operation, or a dataflow block. Of course, sooner or later your computations will have to have an effect, but you’ll find your code is cleaner if you can handle the *processing* with pure pieces and then perform updates with the *results*.

Another principle of functional programming is *immutability*. Immutability means that a piece of data cannot change. One reason that immutable data is useful for concurrent programs is that you never need synchronization for immutable data; the fact that it cannot change makes synchronization unnecessary. Immutable data also helps you avoid side effects. Developers are beginning to use more immutable types, and this book has several recipes covering immutable data structures.

# Summary of Key Technologies

The .NET framework has had some support for asynchronous programming since the very beginning. However, asynchronous programming was difficult until 2012, when .NET 4.5 (along with C# 5.0 and VB 2012) introduced the `async` and `await` keywords. This book will use the modern `async`/`await` approach for all asynchronous recipes, and it has some recipes showing how to interoperate between `async` and the older asynchronous programming patterns. If you need support for older platforms, see [Appendix A](app01.html#legacy-platform-support).

The Task Parallel Library was introduced in .NET 4.0 with full support for both data and task parallelism. These days, it’s available even on platforms with fewer resources, such as mobile phones. The TPL is built in to .NET.

The System.Reactive team has worked hard to support as many platforms as possible. System.Reactive, like `async` and `await`, provide benefits for all sorts of applications, both client and server. System.Reactive is available in the [`System.Reactive`](http://bit.ly/sys-reactive) NuGet package.

The TPL Dataflow library is officially distributed within the NuGet package for [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df).

Most concurrent collections are built into .NET; there are some additional concurrent collections available in the [`System.Threading.Channels`](https://www.nuget.org/packages/System.Threading.Channels) NuGet package. Immutable collections are available in the [`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet package.