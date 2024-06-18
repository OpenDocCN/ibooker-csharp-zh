# Appendix B. Recognizing and Interpreting Asynchronous Patterns

The benefits of asynchronous code have been well understood for decades before .NET was invented. In the early days of .NET, several different styles of asynchronous code were developed, used here and there, and eventually discarded. These were not all bad ideas; many of them paved the way for the modern `async`/`await` approach. However, there’s a lot of legacy code out there that uses older asynchronous patterns. This appendix will discuss the more common patterns, explaining how they work and how to integrate them with modern code.

Sometimes, the same type is updated over the years, acquiring more and more members as it supports multiple asynchronous patterns. Perhaps the best example of this is the `Socket` class. Here are some of the members of the `Socket` class for the core `Send` operation:

[PRE0]

Sadly, with most documentation being alphabetical and with tons of overloads in an attempt to simplify usage, types like `Socket` become difficult to understand. Hopefully the guidelines in this section will help.

# Task-Based Asynchronous Pattern (TAP)

The Task-Based Asynchronous Pattern (TAP) is the modern asynchronous API pattern that is ready for use with `await`. Each asynchronous operation is represented by a single method that returns an awaitable. An “awaitable” is any type that can be consumed by `await`; this is usually `Task` or `Task<T>` but may also be `ValueTask`, `ValueTask<T>`, a type defined by a framework (e.g., `IAsyncAction` or `IAsyncOperation<T>`, used by Universal Windows applications), or even a custom type defined by a library.

It is common for TAP methods to have an `Async` suffix. However, this is just a convention; not all TAP methods have an `Async` suffix. It can be skipped if the API developer believes the asynchronous context is sufficiently implied; e.g., `Task.WhenAll` and `Task.WhenAny` do not have an `Async` suffix. Furthermore, keep in mind that the `Async` suffix may be present on *non*-TAP methods (e.g., `WebClient.DownloadStringAsync` is not a TAP method). The usual pattern in this case is for the TAP method to have a `TaskAsync` suffix (e.g., `WebClient.DownloadStringTaskAsync` is a TAP method).

Methods that return asynchronous streams also follow a TAP-like pattern, with `Async` used as a suffix. Even though they don’t return awaitables, they do return awaitable streams—types that can be consumed using `await foreach`.

The Task-Based Asynchronous Pattern can be recognized by these characteristics:

1.  The operation is represented by a single method.

2.  The method returns an awaitable or an awaitable stream.

3.  The method usually ends with `Async`.

Here’s an example of a type with a TAP API:

[PRE1]

Consuming the Task-Based Asynchronous Pattern is done using `await` and is covered by large portions of this book. If you somehow got to this appendix without knowing how to use `await`, then I’m not sure I can help you at this point, but you can try reading Chapters [1](ch01.html#intro) and [2](ch02.html#async-basics) anyway to see if they jog your memory.

# Asynchronous Programming Model (APM)

After TAP, the Asynchronous Programming Model (APM) pattern is probably the next most-common pattern you’ll encounter. It was the first pattern where asynchronous operations had first-class object representations. The telltale sign of this pattern is the `IAsyncResult` objects in conjunction with a pair of methods that manage the operation, one starting with `Begin` and the other starting with `End`.

`IAsyncResult` was strongly influenced by [native overlapped I/O](http://bit.ly/sync-ipop). The APM pattern allows consuming code to behave either synchronously or asynchronously. The consuming code can choose from these options:

*   Block for the operation to complete. This is done by calling the `End` method.

*   Poll for the operation to complete while doing something else.

*   Supply a callback delegate to invoke when the operation completes.

In all cases, the consuming code must eventually call the `End` method to retrieve the results of the asynchronous operation. If the operation is not completed when `End` is called, it’ll block the calling thread until the operation completes.

The `Begin` method takes an `AsyncCallback` parameter and an `object` parameter (usually called `state`) as its last two parameters. These are used by consuming code to provide a callback delegate to invoke when the operation completes. The `object` parameter can be whatever you want; this is a holdover from the very early days of .NET, before lambda methods or even anonymous methods existed. It is just used to provide context to the `AsyncCallback` parameter.

The APM is fairly widespread among Microsoft libraries, but is not as common in the wider .NET ecosystem. This is because there were never any `IAsyncResult` implementations made available for reuse, and implementing that interface correctly is fairly complex. In addition, it is difficult to compose APM-based systems. I’ve seen only a few custom `IAsyncResult` implementations in the wild; all of these were some version of Jeffrey Richter’s general-purpose `IAsyncResult` implementation, as published in his article, “Concurrent Affairs: Implementing the CLR Asynchronous Programming Model,” from [the March 2007 edition of *MSDN Magazine*](http://bit.ly/conc-aff).

The Asynchronous Programming Model pattern can be recognized by these characteristics:

1.  The operation is represented by a pair of methods, one starting with `Begin` and the other starting with `End`.

2.  The `Begin` method returns an `IAsyncResult`, and takes all normal input parameters, along with an extra `AsyncCallback` parameter and an extra `object` parameter.

3.  The `End` method only takes an `IAsyncResult`, and returns the result value, if any.

Here’s an example of a type with an APM API:

[PRE2]

Consume the APM by converting it to TAP using `Task.Factory.FromAsync`; see [Recipe 8.2](ch08.html#recipe-async-interop-apm) and [the Microsoft docs](http://bit.ly/interop-async).

There are some cases in which code *almost* follows the APM pattern, but not quite; e.g.,, the old `Microsoft.TeamFoundation` client libraries did not include the `object` parameter in their `Begin` methods. In these cases, `Task.Factory.FromAsync` will not work, and you then have the choice of two options. The less efficient option is to call the `Begin` method and pass the `IAsyncResult` to `FromAsync`. The less elegant option is to use the more flexible `TaskCompletionSource<T>`; see [Recipe 8.3](ch08.html#recipe-async-interop-any).

# Event-Based Asynchronous Programming (EAP)

The Event-Based Asynchronous Programming (EAP) defines a matching method/event pair. The method usually ends in `Async`, and it eventually causes an event to be raised that ends in `Completed`.

There are a few caveats when working with EAP that make it a bit more difficult than it first appears. First, you have to remember to add your handler to the event *before* calling the method; otherwise, you’d have a race condition where the event could happen before you subscribed, and then you’d never see it complete. Second, components written in the EAP pattern usually capture the current `SynchronizationContext` at some point and then raise their event in that context. Some components capture the `SynchronizationContext` in the constructor, and others capture it at the time the method is called and the asynchronous operation begins.

The Event-Based Asynchronous Programming pattern can be recognized by these characteristics:

1.  The operation is represented by an event and a method.

2.  The event ends in `Completed`.

3.  The event args type for the `Completed` event might be descended from `AsyncCompletedEventArgs`.

4.  The method usually ends in `Async`.

5.  The method returns `void`.

EAP methods ending in `Async` are distinguishable from TAP methods ending in `Async` because the EAP methods return `void`, while the TAP methods return an awaitable type.

Here’s an example of a type with an EAP API:

[PRE3]

Consume the EAP by converting it to TAP using `TaskCompletionSource<T>`; see [Recipe 8.3](ch08.html#recipe-async-interop-any) and [the Microsoft docs](http://bit.ly/EAP-MS).

# Continuation Passing Style (CPS)

This is a pattern that is much more common in other languages, particularly JavaScript and TypeScript as used by Node.js developers. In this pattern, each asynchronous operation takes a callback delegate that is invoked when the operation completes, either successfully or with error. A variant of this pattern uses *two* callback delegates, one for success and one for error. This kind of callback is called a “continuation,” and the continuation is passed as a parameter, hence the name “continuation passing style.” This pattern was never common in the .NET world, but there are a few older open source libraries that used it.

The Continuation Passing Style pattern can be recognized by these characteristics:

1.  The operation is represented by a single method.

2.  The method takes an extra parameter which is a callback delegate; the callback delegate takes two arguments, one for errors and the other for results.

3.  Alternatively, the operation method takes two extra parameters, both callback delegates; one callback delegate is only for errors, and the other callback delegate is only for results.

4.  The callback delegates are commonly named `done` or `next`.

Here’s an example of a type with a continuation-passing style API:

[PRE4]

Consume CPS by converting it to TAP using `TaskCompletionSource<T>`, passing callback delegates that just complete the `TaskCompletionSource<T>`; see [Recipe 8.3](ch08.html#recipe-async-interop-any).

# Custom Async Patterns

Very specialized types will sometimes define their own custom asynchronous patterns. The most famous example of this is the `Socket` type, which defined a pattern that passed around `SocketAsyncEventArgs` instances representing the operation. The reason this pattern was introduced was that `SocketAsyncEventArgs` could be reused, thus reducing memory churn for applications that do heavy network activity. Modern applications can use [`ValueTask<T>` with `ManualResetValueTaskSourceCore<T>`](http://bit.ly/man-reset-type-doc) to get similar performance gains.

Custom patterns do not have any common characteristics and are therefore the hardest to recognize. Thankfully, custom asynchronous patterns are rare.

Here’s an example of a type with a custom asynchronous API:

[PRE5]

`TaskCompletionSource<T>` is the only way to consume custom asynchronous patterns; see [Recipe 8.3](ch08.html#recipe-async-interop-any).

# ISynchronizeInvoke

All the previous patterns are for asynchronous operations that are started, and once they start, they complete once. Some components follow a subscription model: they represent a push-based stream of events rather than a single operation that starts once and completes once. A good example of a subscription model is the `FileSystemWatcher` type. To observe file system changes, the consuming code first subscribes to multiple events and then sets the `EnableRaisingEvents` property to `true`. Once `EnableRaisingEvents` is `true`, multiple file system change events may be raised.

Some components use an `ISynchronizeInvoke` pattern for their events. They expose a single `ISynchronizeInvoke` property, and consumers set that property to an implementation that allows the component to schedule work. This is most commonly used to schedule work to a UI thread so that the component’s events are raised on the UI thread. By convention, if `ISynchronizeInvoke` is `null`, then no synchronizing of the events is done, and they may be raised on background threads.

The `ISynchronizeInvoke` pattern can be recognized by these characteristics:

1.  There is a property of type `ISynchronizeInvoke`.

2.  The property is usually called `SynchronizingObject`.

Here’s an example of a type that uses the `ISynchronizeInvoke` pattern:

[PRE6]

Since `ISynchronizeInvoke` implies multiple events in a subscription model, the proper way to consume these components is to translate those events to an observable stream, either using `FromEvent` (see [Recipe 6.1](ch06.html#recipe-rx-events)) or `Observable.Create`.