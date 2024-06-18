# Chapter 10\. Cancellation

The .NET 4.0 framework introduced exhaustive and well-designed cancellation support. This support is cooperative, which means that cancellation can be requested but not enforced on code. Since cancellation is cooperative, it isn’t possible to cancel code unless it is written to support cancellation. For this reason, I recommend supporting cancellation in as much of your own code as possible.

Cancellation is a type of signal, with two different sides: a source that triggers the cancellation and a receiver that then responds to the cancellation. In .NET, the source is `CancellationTokenSource` and the receiver is `CancellationToken`. The recipes in this chapter cover both sources and receivers of cancellation in normal usage and describe how to use the cancellation support to interoperate with nonstandard forms of cancellation.

Cancellation is treated as a special kind of error. The convention is that canceled code will throw an exception of type `OperationCanceledException` (or a derived type, such as `TaskCanceledException`). This way the calling code knows that the cancellation was observed.

To indicate to calling code that your method supports cancellation, you should take a `CancellationToken` as a parameter. This parameter is usually the last parameter, unless your method also reports progress ([Recipe 2.3](ch02.html#recipe-async-progress)). You can also consider providing an overload or default parameter value for consumers that do not require cancellation:

[PRE0]

`CancellationToken.None` represents a cancellation token that will never be canceled, and is a special value that is equivalent to `default(CancellationToken)`. Consumers pass this value when they don’t ever want the operation to be canceled.

Asynchronous streams have a similar but more complex way of handling cancellation. Canceling asynchronous streams is covered in detail in [Recipe 3.4](ch03.html#recipe-async-streams-cancellation).

# 10.1 Issuing Cancellation Requests

## Problem

Your code calls cancelable code (that takes a `CancellationToken`) and you need to cancel it.

## Solution

The `CancellationTokenSource` type is the source for a `CancellationToken`. It only enables code to respond to cancellation requests; the `CancellationTokenSource` members enable code to request cancellation.

Each `CancellationTokenSource` is independent from every other one (unless you link them, as considered in [Recipe 10.8](#recipe-linked-cancellationtokens)). The `Token` property returns a `CancellationToken` for that source, and the `Cancel` method issues the actual cancellation request.

The following code illustrates creating a `CancellationTokenSource` and using `Token` and `Cancel`. The code uses an `async` method because it’s easier to illustrate in a short code sample; the same `Token`/`Cancel` pair is used to cancel *all* kinds of code:

[PRE1]

In the preceding example code, the `task` variable is ignored after it has started running; in real-world code, that task would probably be stored somewhere and awaited so that the end user is aware of the final result.

When you cancel code, there is almost always a race condition. The cancelable code may have been *just about to finish* when the cancel request is made, and if it doesn’t happen to check its cancellation token before finishing, it will actually complete successfully. In fact, when you cancel code, there are three possibilities: it may respond to the cancellation request (throwing `OperationCanceledException`), it may finish successfully, or it may finish with an error unrelated to the cancellation (throwing a different exception).

The following code is just like the last, except that it awaits the task, illustrating all three possible results:

[PRE2]

Normally, setting up the `CancellationTokenSource` and issuing the cancellation are in separate methods. Once you cancel a `CancellationTokenSource` instance, it is permanently canceled. If you need another source, you must create another instance. The following code is a more realistic GUI-based example that uses one button to start an asynchronous operation and another button to cancel it. It also disables and enables `StartButton` and `CancelButton` so that there can only be one operation at a time:

[PRE3]

## Discussion

The most realistic example in this recipe used a GUI application, but don’t get the impression that cancellation is just for user interfaces. Cancellation has its place on the server as well; for example, ASP.NET provides a cancellation token representing the request timeout or client disconnect. It’s true that cancellation token sources are rarer on the server side, but there’s no reason you can’t use them; they’re useful if you need to cancel for some reason not covered by ASP.NET cancellation, such as an additional timeout for a portion of the request processing.

## See Also

[Recipe 10.4](#recipe-cancel-async) covers passing tokens to `async` code.

[Recipe 10.5](#recipe-cancel-parallel) covers passing tokens to parallel code.

[Recipe 10.6](#recipe-cancel-rx) covers using tokens with reactive code.

[Recipe 10.7](#recipe-cancel-dataflow) covers passing tokens to dataflow meshes.

# 10.2 Responding to Cancellation Requests by Polling

## Problem

You have a loop in your code that needs to support cancellation.

## Solution

When you have a processing loop in your code, then there isn’t a lower-level API to which you can pass the `CancellationToken`. In this case, you should periodically check whether the token has been canceled. The following code observes the token periodically while executing a CPU-bound loop:

[PRE4]

If your loop is very tight (i.e., if the body of your loop executes very quickly), then you may want to limit how often you check your cancellation token. As always, measure your performance before and after a change like this before deciding which way is best. The following code is similar to the previous example, but it has more iterations of a faster loop, so I added a limit to how often the token is checked:

[PRE5]

The proper limit to use depends entirely on how much work you’re doing and how responsive the cancellation needs to be.

## Discussion

The majority of the time, your code should just pass through the `CancellationToken` to the next layer. There are examples of this in Recipes [10.4](#recipe-cancel-async), [10.5](#recipe-cancel-parallel), [10.6](#recipe-cancel-rx), and [10.7](#recipe-cancel-dataflow). The polling technique in this recipe should only be used if you have a processing loop that needs to support cancellation.

There’s another member on `CancellationToken` called `IsCancellationRequested`, which starts returning `true` when the token is canceled. Some people use this member to respond to cancellation, usually by returning a default or `null` value. I do not recommend this approach for most code. The standard cancellation pattern is to raise an `OperationCanceledException`, which is taken care of by `ThrowIfCancellationRequested`. If code further up the stack wants to catch the exception and act like the result is `null`, then that’s fine, but any code taking a `CancellationToken` should follow the standard cancellation pattern. If you do decide not to follow the cancellation pattern, at least document it clearly.

`ThrowIfCancellationRequested` works by *polling* the cancellation token; your code has to call it at regular intervals. There’s also a way to register a callback that is invoked when cancellation is requested. The callback approach is more about interoperating with other cancellation systems; [Recipe 10.9](#recipe-cancellationtoken-callbacks) covers using callbacks with cancellation.

## See Also

[Recipe 10.4](#recipe-cancel-async) covers passing tokens to `async` code.

[Recipe 10.5](#recipe-cancel-parallel) covers passing tokens to parallel code.

[Recipe 10.6](#recipe-cancel-rx) covers using tokens with reactive code.

[Recipe 10.7](#recipe-cancel-dataflow) covers passing tokens to dataflow meshes.

[Recipe 10.9](#recipe-cancellationtoken-callbacks) covers using callbacks instead of polling to respond to cancellation requests.

[Recipe 10.1](#recipe-cancellationtokensource) covers issuing a cancellation request.

# 10.3 Canceling Due to Timeouts

## Problem

You have some code that needs to stop running after a timeout.

## Solution

Cancellation is a natural solution for timeout situations. A timeout is just one type of cancellation request. The code that needs to be canceled merely observes the cancellation token just like any other cancellation; it should neither know nor care that the cancellation source is a timer.

There are some convenience methods for cancellation token sources that automatically issue a cancel request based on a timer. You can pass the timeout into the constructor:

[PRE6]

Alternatively, if you already have a `CancellationTokenSource` instance, you can start a timeout for that instance:

[PRE7]

## Discussion

To execute code with a timeout, use `CancellationTokenSource` and `CancelAfter` (or the constructor). There are other ways to do the same thing, but using the existing cancellation system is the easiest and most efficient option.

Remember that the code to be canceled needs to observe the cancellation token; it isn’t possible to easily cancel un-cancelable code.

## See Also

[Recipe 10.4](#recipe-cancel-async) covers passing tokens to `async` code.

[Recipe 10.5](#recipe-cancel-parallel) covers passing tokens to parallel code.

[Recipe 10.6](#recipe-cancel-rx) covers using tokens with reactive code.

[Recipe 10.7](#recipe-cancel-dataflow) covers passing tokens to dataflow meshes.

# 10.4 Canceling async Code

## Problem

You are using `async` code and need to support cancellation.

## Solution

The simplest way to support cancellation in asynchronous code is to just pass the `CancellationToken` through to the next layer. The following example code performs an asynchronous delay and then returns a value; it supports cancellation by passing the token to `Task.Delay`:

[PRE8]

Many asynchronous APIs support `CancellationToken`, so enabling cancellation yourself is usually a simple matter of taking a token and passing it along. As a general rule, if your method calls APIs that take `CancellationToken`, then your method should also take a `CancellationToken` and pass it to every API that supports it.

## Discussion

Unfortunately, some methods don’t support cancellation. When you’re in this situation, there’s no easy solution. It’s not possible to safely stop arbitrary code unless it’s wrapped in a separate executable. If your code calls code that doesn’t support cancellation, and if you don’t want to wrap that code in a separate executable, you do always have the option of *pretending* to cancel the operation by ignoring the result.

Cancellation should be provided as an option whenever possible. This is because proper cancellation at a higher level depends on proper cancellation at the lower level. So, when you’re writing your own `async` methods, try your best to include support for cancellation; you never know what higher-level method will want to call yours, and it might need cancellation.

## See Also

[Recipe 10.1](#recipe-cancellationtokensource) covers issuing a cancellation request.

[Recipe 10.3](#recipe-cancel-timeout) covers using cancellation as a timeout.

# 10.5 Canceling Parallel Code

## Problem

You are using parallel code and need to support cancellation.

## Solution

The easiest way to support cancellation is to pass the `CancellationToken` through to the parallel code. `Parallel` methods support this by taking a `ParallelOptions` instance. You can set the `CancellationToken` on a `ParallelOptions` instance in the following manner:

[PRE9]

Alternatively, it’s possible to observe the `CancellationToken` directly in your loop body:

[PRE10]

The alternative method is more work and doesn’t compose as well because the parallel loop will wrap the `OperationCanceledException` within an `AggregateException`. Also, if you pass the `CancellationToken` as part of a `ParallelOptions` instance, the `Parallel` class may make more intelligent decisions about how often to check the token. For these reasons, it’s best to pass the token as an option. If you pass the token as an option, you could *also* pass the token to the loop body, but you don’t want to *only* pass the token to the loop body.

Parallel LINQ (PLINQ) also has built-in support for cancellation, using the `WithCancellation` operator:

[PRE11]

## Discussion

Supporting cancellation for parallel work is important for a good user experience. If your application is doing parallel work, it’ll use a large amount of CPU at least for a short time. High CPU usage is something that users notice, even if it doesn’t interfere with other applications on the same machine. So, I recommend supporting cancellation whenever you do parallel computation (or any other CPU-intensive work), even if the total time spent with high CPU usage isn’t extremely long.

## See Also

[Recipe 10.1](#recipe-cancellationtokensource) covers issuing a cancellation request.

# 10.6 Canceling System.Reactive Code

## Problem

You have some reactive code, and you need it to be cancelable.

## Solution

The System.Reactive library has a notion of a *subscription* to an observable stream. Your code can dispose of the subscription to unsubscribe from the stream. In many cases, this is sufficient to logically cancel the stream. For example, the following code subscribes to mouse clicks when one button is pressed and unsubscribes (cancels the subscription) when another button is pressed:

[PRE12]

It’s quite convenient to make System.Reactive work with the `CancellationTokenSource`/`CancellationToken` system that everything else uses for cancellation. The rest of this recipe covers ways that System.Reactive observables interact with `CancellationToken`.

The first major use case is when the observable code is wrapped in asynchronous code. The basic approach was covered in [Recipe 8.5](ch08.html#recipe-async-interop-rx), and now you want to add `CancellationToken` support. In general, the easiest way to do this is to perform all operations using reactive operators and then call `ToTask` to convert the last resulting element to an awaitable task. The following code shows how to asynchronously take the last element in a sequence:

[PRE13]

Taking the first element is very similar; just modify the observable before calling `ToTask`:

[PRE14]

Asynchronously converting the entire observable sequence to a task is likewise similar:

[PRE15]

Finally, let’s consider the reverse situation. We’ve looked at several ways to handle situations where System.Reactive code responds to `CancellationToken`—that is, where a `CancellationTokenSource` cancel request is translated into a disposal of that subscription. It’s also possible to go the other way: issuing a cancellation request as a response to disposal.

The `FromAsync`, `StartAsync`, and `SelectMany` operators all support cancellation, as seen in [Recipe 8.6](ch08.html#recipe-rx-interop-async). These operators cover the vast majority of use cases. Rx also provides a `CancellationDisposable` type that cancels a `CancellationToken` when disposed. You can use `CancellationDisposable` directly, like this:

[PRE16]

## Discussion

System.Reactive (Rx) has its own notion of cancellation: disposing subscriptions. This recipe looked at several ways to make Rx play nicely with the universal cancellation framework introduced in .NET 4.0\. As long as you are in the Rx world portion of your code, use the Rx subscription/disposal system; it’s cleanest if you only introduce `CancellationToken` support at the boundaries.

## See Also

[Recipe 8.5](ch08.html#recipe-async-interop-rx) covers asynchronous wrappers around Rx code (without cancellation support).

[Recipe 8.6](ch08.html#recipe-rx-interop-async) covers Rx wrappers around asynchronous code (with cancellation support).

[Recipe 10.1](#recipe-cancellationtokensource) covers issuing a cancellation request.

# 10.7 Canceling Dataflow Meshes

## Problem

You are using dataflow meshes and need to support cancellation.

## Solution

The best way to support cancellation in your code is to pass the `CancellationToken` through to a cancelable API. Each block in a dataflow mesh supports cancellation as a part of its `DataflowBlockOptions`. If you want to extend your custom dataflow block with cancellation support, set the `CancellationToken` property on the block options:

[PRE17]

In this example, I applied the `CancellationToken` to every block in the mesh, which isn’t strictly necessary. Since I’m also propagating completion along the links, I could apply it to the first block and allow it to propagate through. Cancellations are considered a special form of error, so the blocks further down the pipeline would be completed with an error as that error propagates through. That said, if I’m canceling a mesh, I may as well cancel every block simultaneously, so in this case I usually set the `CancellationToken` option on every block.

## Discussion

In dataflow meshes, cancellation is *not* a form of flush. When a block is canceled, it drops all its input and refuses to take any new items. So if you cancel a block while it’s running, you *will* lose data.

## See Also

[Recipe 10.1](#recipe-cancellationtokensource) covers issuing a cancellation request.

# 10.8 Injecting Cancellation Requests

## Problem

You have a layer of your code that needs to respond to cancellation requests and also issue its own cancellation requests to the next layer.

## Solution

The .NET 4.0 cancellation system has built-in support for this scenario, known as *linked cancellation tokens*. A cancellation token source can be created linked to one (or many) existing tokens. When you create a linked cancellation token source, the resulting token is canceled when any of the existing tokens is canceled or when the linked source is explicitly canceled.

The following code performs an asynchronous HTTP request. The token passed into the `GetWithTimeoutAsync` method represents cancellation requested by the end user, and the `GetWithTimeoutAsync` method also applies a timeout to the request:

[PRE18]

The resulting `combinedToken` is canceled when either the user cancels the existing `cancellationToken` or when the linked source is canceled by `CancelAfter`.

## Discussion

Although the preceding example only used a single `CancellationToken` source, the `CreateLinkedTokenSource` method can take any number of cancellation tokens as parameters. This enables you to create a single combined token from which you can implement your logical cancellation. For example, ASP.NET provides a cancellation token that represents the user disconnecting (`HttpContext.RequestAborted`); handler code may create a linked token that responds to either a user disconnecting or its own cancellation reason, such as a timeout.

Keep in mind the lifetime of the linked cancellation token source. The previous example is the usual use case, where one or more cancellation tokens are passed into the method, which then links them together and passes them on as a combined token. Note also that the example code uses the `using` statement, which ensures that the linked cancellation token source is disposed of when the operation is complete (and the combined token is no longer being used). Consider what would happen if the code didn’t dispose of the linked cancellation token source: it’s possible that the `GetWithTimeoutAsync` method may be called multiple times with the same (long-lived) existing token, in which case the code would link a new token source each time the method is invoked. Even after the HTTP requests complete (and nothing is using the combined token), that linked source is still attached to the existing token. To prevent memory leaks like this, dispose of the linked cancellation token source when you no longer need the combined token.

## See Also

[Recipe 10.1](#recipe-cancellationtokensource) covers issuing cancellation requests in general.

[Recipe 10.3](#recipe-cancel-timeout) covers using cancellation as a timeout.

# 10.9 Interop with Other Cancellation Systems

## Problem

You have some external or legacy code with its own notion of cancellation, and you want to control it using a standard `CancellationToken`.

## Solution

The `CancellationToken` has two primary ways to respond to a cancellation request: polling (covered in [Recipe 10.2](#recipe-cancellationtoken)) and callbacks (the subject of this recipe). Polling is normally used for CPU-bound code, such as data processing loops; callbacks are normally used in all other scenarios. You can register a callback for a token using the `CancellationToken.Register` method.

For example, let’s say you’re wrapping the `System.Net.NetworkInformation.Ping` type and you want to be able to cancel a ping. The `Ping` class already has a `Task`-based API but does not support `CancellationToken`. Instead, the `Ping` type has its own `SendAsyncCancel` method that you can use to cancel a ping. To do this, register a callback that invokes that method:

[PRE19]

Now, when a cancellation is requested, the `CancellationToken` will invoke the `SendAsyncCancel` method for you, canceling the `SendPingAsync` method.

## Discussion

The `CancellationToken.Register` method can be used to interoperate with any kind of alternative cancellation system. But do bear in mind that when a method takes a `CancellationToken`, a cancellation request should only cancel that one operation. Some alternative cancellation systems implement a cancel by closing some resource, which can cancel multiple operations; this kind of cancellation system doesn’t map well to a `CancellationToken`. If you do decide to wrap that kind of cancellation in a `CancellationToken`, you should document its unusual cancellation semantics.

Keep in mind the lifetime of the callback registration. The `Register` method returns a disposable that should be disposed of when that callback is no longer needed. The preceding example code uses a `using` statement to clean up when the asynchronous operation completes. If the code didn’t have that `using` statement, then each time the code is called with the same (long-lived) `CancellationToken`, it would add another callback (which in turn keeps the `Ping` object alive). To avoid memory and resource leaks, dispose of the callback registration when you no longer need the callback.

## See Also

[Recipe 10.2](#recipe-cancellationtoken) covers responding to a cancellation token by polling rather than callbacks.

[Recipe 10.1](#recipe-cancellationtokensource) covers issuing cancellation requests in general.