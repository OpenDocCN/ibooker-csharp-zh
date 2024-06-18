# Chapter 11\. Functional-Friendly OOP

Modern programs require asynchronous programming; these days servers must scale better than ever, and end-user applications must be more responsive than ever. Developers are finding that they must learn asynchronous programming, and as they explore this world, they find that it often clashes with the traditional object-oriented programming that they’re accustomed to.

The core reason for this is because asynchronous programming is functional. By “functional,” I don’t mean “it works”; I mean it’s a functional style of programming instead of a procedural style of programming. A lot of developers learned basic functional programming in college and have hardly touched it since. If code like `(car (cdr '(3 5 7)))` gives you a chill as repressed memories come flooding back, then you may be in that category. But don’t fear; modern asynchronous programming isn’t that hard once you get used to it.

The major breakthrough with `async` is that you can still think procedurally while programming asynchronously. This makes asynchronous methods easier to write and understand. However, under the covers, asynchronous code is still functional in nature, and this causes some problems when people try to force `async` methods into classical object-oriented designs. The recipes in this chapter deal with those friction points where asynchronous code clashes with object-oriented programming.

These friction points are especially noticeable when translating an existing OOP code base into an `async`-friendly code base.

# 11.1 Async Interfaces and Inheritance

## Problem

You have a method in your interface or base class that you want to make asynchronous.

## Solution

The key to understanding this problem and its solution is to realize that `async` is an implementation detail. The `async` keyword can only be applied to methods with implementations; it isn’t possible to apply it to abstract methods or interface methods (unless they have default implementations). However, you can define a method with the same signature as an `async` method, just without the `async` keyword.

Remember that *types* are awaitable, not *methods*. You can `await` a `Task` returned by a method, whether or not that method is implemented using `async`. So, an interface or abstract method can just return a `Task` (or `Task<T>`), and the return value of that method is awaitable.

The following code defines an interface with an asynchronous method (without the `async` keyword), an implementation of that interface (with `async`), and an independent method that consumes a method of the interface (via `await`):

[PRE0]

This same pattern works for abstract methods in base classes.

An asynchronous method signature only means that the implementation *may* be asynchronous. It is possible for the actual implementation to be synchronous if it has no real asynchronous work to do. For example, a test stub may implement the same interface (without `async`) by using something like `FromResult`:

[PRE1]

## Discussion

At the time of this writing, `async` and `await` are still gaining traction. As asynchronous methods become more common, asynchronous methods on interfaces and base classes will become more common as well. They’re not that hard to work with if you keep in mind that it is the return type that is awaitable (not the method), and that an asynchronous method definition may be implemented either asynchronously or synchronously.

## See Also

[Recipe 2.2](ch02.html#recipe-async-fromresult) covers returning a completed task, implementing an asynchronous method signature with synchronous code.

# 11.2 Async Construction: Factories

## Problem

You are coding a type that requires some asynchronous work to be done in its constructor.

## Solution

Constructors cannot be `async`, nor can they use the `await` keyword. It would certainly be useful to `await` in a constructor, but this would change the C# language considerably.

One possibility is to have a constructor paired with an `async` initialization method, so the type could be used like this:

[PRE2]

This approach has some disadvantages. It can be easy to forget to call the `InitializeAsync` method, and the instance isn’t usable immediately after it’s constructed.

A better solution is to make the type its own factory. The following type illustrates the asynchronous factory method pattern:

[PRE3]

The constructor and `InitializeAsync` method are `private` so that other code cannot possibly misuse them; so the only way of creating an instance is via the static `CreateAsync` factory method. Calling code cannot access the instance until after the initialization is complete.

Other code can create an instance like this:

[PRE4]

## Discussion

The primary advantage of this pattern is that there’s no way that other code can get an uninitialized instance of `MyAsyncClass`. That’s why I prefer this pattern over other approaches whenever I can use it.

Unfortunately, this approach does not work in some scenarios—in particular, when your code is using a dependency injection provider. No major dependency injection or inversion of control library works with `async` code. If you find yourself in one of these scenarios, there are a couple of alternatives that you can consider.

If the instance you’re creating is actually a shared resource, then you can use the asynchronous lazy type discussed in [Recipe 14.1](ch14.html#recipe-lazy). Otherwise, you can use the asynchronous initialization pattern discussed in [Recipe 11.3](#recipe-oop-constructors).

Here’s an example of what *not* to do:

[PRE5]

At first glance, this seems like a reasonable approach: you get a regular constructor that kicks off an asynchronous operation; however, there are several drawbacks that are due to the use of `async void`. The first problem is that when the constructor completes, the instance is still being asynchronously initialized, and there isn’t an obvious way to determine when the asynchronous initialization has completed. The second problem is with error handling: any exceptions raised from `InitializeAsync` can’t be caught by any `catch` clauses surrounding the object construction.

## See Also

[Recipe 11.3](#recipe-oop-constructors) covers the asynchronous initialization pattern, a way of doing asynchronous construction that works with dependency injection/inversion of control containers.

[Recipe 14.1](ch14.html#recipe-lazy) covers asynchronous lazy initialization, which is a viable solution if the instance is conceptually a shared resource or service.

# 11.3 Async Construction: The Asynchronous Initialization Pattern

## Problem

You are coding a type that requires some asynchronous work to be done in its constructor, but you cannot use the asynchronous factory pattern ([Recipe 11.2](#recipe-oop-factories)) because the instance is created via reflection (e.g., a dependency injection/inversion of control library, data binding, `Activator.CreateInstance`, and so on).

## Solution

When you encounter this scenario, you *have* to return an uninitialized instance, though you can mitigate this situation by applying a common pattern: the asynchronous initialization pattern. Every type that requires asynchronous initialization should define a property, like this:

[PRE6]

I usually like to define this in a marker interface for types that require asynchronous initialization:

[PRE7]

When you implement this pattern, you should start the initialization (and assign the `Initialization` property) in the constructor. The results of the asynchronous initialization (including any exceptions) are exposed via that `Initialization` property. Here’s an example implementation of a simple type using asynchronous initialization:

[PRE8]

If you’re using a dependency injection/inversion of control library, you can create and initialize an instance of this type using code like the following:

[PRE9]

You can extend this pattern to allow composition of types with asynchronous initialization. In the following example another type that depends on an `IMyFundamentalType` is defined:

[PRE10]

The composed type waits for all of its components to initialize before it proceeds with its initialization. The rule to follow is that every component should be initialized by the end of `InitializeAsync`. This ensures that all dependent types are initialized as part of the composed initialization. Any exceptions from a component initialization are propagated to the composed type’s initialization.

## Discussion

If you can, I recommend using asynchronous factories ([Recipe 11.2](#recipe-oop-factories)) or asynchronous lazy initialization ([Recipe 14.1](ch14.html#recipe-lazy)) instead of this solution. Those are the best approaches because you never expose an uninitialized instance. However, if your instances are created by dependency injection/inversion of control, data binding, and so on, then you’re forced to expose an uninitialized instance, and in that case I recommend using the asynchronous initialization pattern in this recipe.

Remember from the recipe on asynchronous interfaces ([Recipe 11.1](#recipe-oop-inheritance)) that an asynchronous method signature only means that the method *may* be asynchronous. The `MyComposedType.InitializeAsync` code is a good example of this: if the `IMyFundamentalType` instance does not also implement `IAsyncInitialization` and `MyComposedType` has no asynchronous initialization of its own, then its `InitializeAsync` method completes synchronously.

The code for checking whether an instance implements `IAsyncInitialization` and initializing it is a bit awkward, and it becomes more so when you have a composed type that depends on a larger number of components. It’s easy enough to create a helper method that can be used to simplify the code:

[PRE11]

You can call `InitializeAllAsync` and pass in whatever instances you want initialized; the method will ignore instances that don’t implement `IAsyncInitialization`. The initialization code for a composed type that depends on three injected instances can then look like the following:

[PRE12]

## See Also

[Recipe 11.2](#recipe-oop-factories) covers asynchronous factories, which are a way to do asynchronous construction without exposing uninitialized instances.

[Recipe 14.1](ch14.html#recipe-lazy) covers asynchronous lazy initialization, which can be used if the instance is a shared resource or service.

[Recipe 11.1](#recipe-oop-inheritance) covers asynchronous interfaces.

# 11.4 Async Properties

## Problem

You have a property that you want to make `async`. The property is not used in data binding.

## Solution

This is a problem that often comes up when converting existing code to use `async`; in this situation, you have a property whose getter invokes a method that is now asynchronous. However, there’s no such thing as an “asynchronous property.” It’s not possible to use the `async` keyword with a property, and that’s a good thing. Property getters should return current values; they shouldn’t be kicking off background operations:

[PRE13]

When you find that your code wants an “asynchronous property,” what your code really *needs* is something a little different. The solution depends on whether your property value needs to be evaluated once or multiple times; you have a choice between these semantics:

*   A value that is asynchronously evaluated each time it is read

*   A value that is asynchronously evaluated once and is cached for future access

If your “asynchronous property” needs to kick off a new (asynchronous) evaluation each time it’s read, then it’s not a *property*; it’s a *method* in disguise. If you encountered this situation when converting synchronous code to asynchronous, then it’s time to admit that the original design was actually incorrect; the property should have been a method all along:

[PRE14]

It is *possible* to return a `Task<int>` directly from a property, as the following code shows:

[PRE15]

I do not recommend this approach, however. If every access to a property is going to kick off a new asynchronous operation, then that “property” should really be a method. The fact that it’s an asynchronous method makes it clearer that a new asynchronous operation is initiated every time, so the API isn’t misleading. Recipes [11.3](#recipe-oop-constructors) and [11.6](#recipe-oop-dispose) do use task-returning properties, but those properties apply to the instance as a whole; they don’t start a new asynchronous operation every time they are read.

Sometimes you want the property value evaluated every time it’s retrieved. Other times you want the property to only kick off a single (asynchronous) evaluation and cache that resulting value for future use. In this case, you can use asynchronous lazy initialization. That solution is covered in detail in [Recipe 14.1](ch14.html#recipe-lazy), but in the meantime, here’s an example of what the code would look like:

[PRE16]

The code will only execute the asynchronous evaluation once and then return that same value to all callers. Calling code looks like the following:

[PRE17]

In this case, the property syntax is appropriate since there’s only one evaluation happening.

## Discussion

One of the important questions to ask yourself is whether reading the property should start a new asynchronous operation; if the answer is yes, then use an asynchronous *method* instead of a property. If the property should act as a lazy-evaluated cache, then use asynchronous initialization (see [Recipe 14.1](ch14.html#recipe-lazy)). In this recipe I didn’t cover properties that are used in data binding; I cover those in [Recipe 14.3](ch14.html#recipe-async-databinding).

When you’re converting a synchronous property to an “asynchronous property,” here’s an example of what *not* to do:

[PRE18]

While we’re on the subject of properties in `async` code, it’s worth thinking about how state relates to asynchronous code. This is especially true if you’re converting a synchronous code base to asynchronous. Consider any state that you expose in your API (e.g., via properties); for each piece of state, ask yourself, what is the current state of an object that has an asynchronous operation in progress? There’s no right answer, but it’s important to think about the semantics you want and to document them.

For example, consider `Stream.Position`, which represents the current offset of the stream pointer. With the synchronous API, when you call `Stream.Read` or `Stream.Write`, the reading/writing is done and `Stream.Position` is updated to reflect the new position before the `Read` or `Write` method returns. The semantics are clear for synchronous code.

Now, consider `Stream.ReadAsync` and `Stream.WriteAsync`: when should `Stream.Position` be updated? When the read/write operation is complete, or before it actually happens? If it’s updated before the operation completes, is it updated synchronously by the time `ReadAsync`/`WriteAsync` returns, or could it happen shortly after that?

This is a great example of how a property that exposes state has perfectly clear semantics for synchronous code but no obviously correct semantics for asynchronous code. It’s not the end of the world—you just need to think about your entire API when async-enabling your types and document the semantics you choose.

## See Also

[Recipe 14.1](ch14.html#recipe-lazy) covers asynchronous lazy initialization in detail.

[Recipe 14.3](ch14.html#recipe-async-databinding) covers “asynchronous properties” that need to support data binding.

# 11.5 Async Events

## Problem

You have an event that you need to use with handlers that might be `async`, and you need to detect whether the event handlers have completed. Note that this is a rare situation when raising an event; usually, when you raise an event, you don’t care when the handlers complete.

## Solution

It’s not feasible to detect when `async void` handlers have returned, so you need some alternative way to detect when the asynchronous handlers have completed. The Universal Windows platform introduced a concept called *deferrals* that you can use to track asynchronous handlers. An asynchronous handler allocates a deferral before its first `await` and later notifies the deferral when it completes. Synchronous handlers don’t need to use deferrals.

The `Nito.AsyncEx` library includes a type called a `DeferralManager`, which is used by the component raising the event. This deferral manager then permits event handlers to allocate deferrals and keeps track of when all the deferrals have completed.

For each of your events where you need to wait for the handlers to complete, you first extend your event arguments type:

[PRE19]

When you’re dealing with asynchronous event handlers, it’s best to make your event arguments type threadsafe. The easiest way to do this is to make it immutable (i.e., have all its properties be read-only).

Then, each time you raise the event, you can (asynchronously) wait for all asynchronous event handlers to complete. The following code will return a completed task if there are no handlers; otherwise, it’ll create a new instance of your event arguments type, pass it to the handlers, and wait for any asynchronous handlers to complete:

[PRE20]

Asynchronous event handlers can then use the deferral within a `using` block; the deferral notifies the deferral manager when it is disposed:

[PRE21]

This is slightly different than how Universal Windows deferrals work. In the Universal Windows API, each event that needs deferrals defines its own deferral type, and that deferral type has an explicit `Complete` method rather than being `IDisposable`.

## Discussion

There are logically two different kinds of events used in .NET, with very different semantics. I call these *notification events* and *command events*; this isn’t official terminology, just some terms that I chose for clarity. A notification event is an event that is raised to notify other components of some situation. A notification is purely one-way; the sender of the event doesn’t care whether there are any receivers of the event. With notifications, the sender and receiver can be entirely disconnected. Most events are notification events; one example is a button click.

In contrast, a command event is an event that is raised to implement some functionality on behalf of the sending component. Command events aren’t “events” in the true sense of the term, though they are often implemented as .NET events. The sender of a command must wait until the receiver handles it before moving on. If you use events to implement the Visitor pattern, then those are command events. Lifecycle events are also command events, so ASP.NET page lifecycle events and many UI framework events, such as Xamarin’s `Application.PageAppearing`, fall into this category. Any UI framework event that is actually an implementation is also a command event (e.g., `BackgroundWorker.DoWork`).

Notification events don’t require any special code to enable asynchronous handlers; the event handlers can be `async void` and work just fine. When the event sender raises the event, the asynchronous event handlers aren’t completed immediately, but that doesn’t matter because they’re just notification events. So, if your event is a notification event, the grand total amount of work you need to do to support asynchronous handlers is: nothing.

Command events are a different story. When you have a command event, you need a way to detect when the handlers have completed. The preceding solution with deferrals should only be used for command events.

###### Tip

The `DeferralManager` type is in the [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet package.

## See Also

[Chapter 2](ch02.html#async-basics) covers the basics of asynchronous programming.

# 11.6 Async Disposal

## Problem

You have a type that has asynchronous operations but also needs to enable disposal of its resources.

## Solution

There are a couple of common options for dealing with existing operations when disposing of an instance: you can either treat the disposal as a cancellation request that is applied to all existing operations, or you can implement an actual *asynchronous disposal*.

Treating disposal as a cancellation has a historic precedence on Windows; types such as file streams and sockets cancel any existing reads or writes when they are closed. By defining your own private `CancellationTokenSource` and passing that token to your internal operations, you can do something very similar in .NET. With the following code, `Dispose` will cancel the operations but won’t wait for those operations to complete:

[PRE22]

The code shows the basic pattern around `Dispose`. In a real-world app, you should put in checks that the object is not already disposed of and also enable the user to supply her own `CancellationToken` (using the technique from [Recipe 10.8](ch10.html#recipe-linked-cancellationtokens)):

[PRE23]

Calling code will have any existing operations canceled when `Dispose` is called:

[PRE24]

For some types, implementing `Dispose` as a cancellation request works just fine (e.g., `HttpClient` has these semantics). However, other types need to know when all the operations have completed. For these types, you need some kind of asynchronous disposal.

Asynchronous disposal is a technique introduced with C# 8.0 and .NET Core 3.0\. The BCL introduced a new `IAsyncDisposable` interface that is an asynchronous equivalent of `IDisposable`. The language simultaneously introduced an `await using` statement that is the asynchronous equivalent of `using`. So types that would like to do asynchronous work during disposal now have that capability:

[PRE25]

The return type of `DisposeAsync` is `ValueTask` and not `Task`, but the standard `async` and `await` keywords work just as well with `ValueTask` as they do with `Task`.

Types implementing `IAsyncDisposable` are usually consumed by `await using`:

[PRE26]

If you need to avoid context using `ConfigureAwait(false)`, that is possible, but it’s a bit more awkward because you have to declare your variable outside the `await using` statement:

[PRE27]

## Discussion

Asynchronous disposal is definitely easier than implementing `Dispose` as a cancellation request, and the more complex approach should only be used when you really need it. In fact, most of the time you can get away with not disposing anything at all, which is certainly the easiest approach because you don’t have to do anything.

This recipe has two patterns for handling disposal; it’s also possible to use *both* of them if you want. Using both would give your type the semantics of a clean shutdown if the client code uses `await using`, and a “cancel” if the client code uses `Dispose`. I wouldn’t recommend this in general, but it is an option.

## See Also

[Recipe 10.8](ch10.html#recipe-linked-cancellationtokens) covers linked cancellation tokens.

[Recipe 11.1](#recipe-oop-inheritance) covers asynchronous interfaces.

[Recipe 2.10](ch02.html#recipe-async-implement-valuetask) discusses implementing methods returning `ValueTask`.

[Recipe 2.7](ch02.html#recipe-async-configureawait) covers avoiding context using `ConfigureAwait(false)`.