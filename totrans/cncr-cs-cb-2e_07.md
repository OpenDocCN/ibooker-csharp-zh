# Chapter 7\. Testing

Testing is an essential part of software quality. Unit testing advocates have become common in recent years; it seems that you read or hear about it everywhere. Some promote *test-driven development*, a style of coding that ensures you have comprehensive tests when the application is complete. The benefits of unit testing on code quality and overall time to completion are well known, and yet many developers still don’t write unit tests.

I encourage you to write at least some unit tests. Start with the code in which you feel the least confidence. In my experience, unit tests have given me two main advantages:

*   **Better understanding of the code.** You know that part of the application that works but you have no idea how? It’s always kind of in the back of your mind when the really weird bug reports come in. Writing unit tests for code you find difficult is a great way to get a clear understanding of how it works. After writing unit tests describing its behavior, the code is no longer mysterious; you end up with a set of unit tests that describe its behavior and the dependencies that code has on the rest of the code.

*   **Greater confidence to make changes.** Sooner or later, you’ll get that feature request that requires you to change the code that scares you, and you’ll no longer be able to pretend it isn’t there (I know how that feels; I’ve been there!). It’s best to be proactive: write the unit tests for the scary code before the feature request comes in. Once your unit tests are complete, you’ll have an early warning system that will alert you immediately if your changes break existing behavior. When you have a pull request, unit tests also give you greater confidence that the code changes don’t break existing behavior.

Both of these advantages apply to your own code just as much as others’ code. I’m sure there are other advantages, too. Does unit testing decrease the frequency of bugs? Most likely. Does unit testing reduce the overall time on a project? Possibly. But the advantages I’ve described are definite; I experience them every time I write unit tests. So, that’s my sales pitch for unit testing.

This chapter contains recipes that are all about testing. A lot of developers (even ones who normally write unit tests) shy away from testing concurrent code because they assume it’s hard. However, as these recipes will show, unit testing concurrent code isn’t as difficult as they think. Modern features and libraries, such as `async` and System.Reactive, have put a lot of thought into testing, and it shows. I encourage you to use these recipes to write unit tests, especially if you’re new to concurrency (i.e., the new concurrent code appears hard or scary).

# 7.1 Unit Testing async Methods

## Problem

You have an `async` method that you need to unit test.

## Solution

Most modern unit test frameworks support `async Task` unit test methods, including MSTest, NUnit, and xUnit. MSTest began support for these tests with Visual Studio 2012\. If you use another unit test framework, you may have to upgrade to the latest version.

Here is an example of an `async` MSTest unit test:

[PRE0]

The unit test framework will notice that the return type of the method is `Task` and will intelligently wait for the task to complete before marking the test “successful” or “failed.”

If your unit test framework doesn’t support `async Task` unit tests, then it’ll need some help to wait for the asynchronous operation under test. One option is that you can use `GetAwaiter().GetResult()` to synchronously block on the task; if you then use `GetAwaiter().GetResult()` instead of `Wait()`, it avoids the `AggregateException` wrapper if the task has an exception. However, I prefer to use the `AsyncContext` type from the `Nito.AsyncEx` NuGet package:

[PRE1]

`AsyncContext.Run` will wait until all asynchronous methods complete.

## Discussion

Mocking asynchronous dependencies can be a bit awkward at first. It’s a good idea to at least test how your methods respond to synchronous success (mocking with `Task.FromResult`), synchronous errors (mocking with `Task.FromException`), and asynchronous success (mocking with `Task.Yield` and a return value). You’ll find coverage of `Task.FromResult` and `Task.FromException` in [Recipe 2.2](ch02.html#recipe-async-fromresult). `Task.Yield` can be used to force asynchronous behavior, and is primarily useful for unit tests:

[PRE2]

When testing asynchronous code, deadlocks and race conditions may surface more often than when testing synchronous code. I find the per-test timeout setting useful; in Visual Studio, you can add a test settings file to your solution that enables you to set individual test timeouts. The default value is quite high; I usually have a per-test timeout setting of two seconds.

###### Tip

The `AsyncContext` type is in the [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet package.

## See Also

[Recipe 7.2](#recipe-async-testfail) covers unit testing asynchronous methods expected to fail.

# 7.2 Unit Testing async Methods Expected to Fail

## Problem

You need to write a unit test that checks for a specific failure of an `async Task` method.

## Solution

If you’re doing desktop or server development, MSTest does support failure testing via the regular `ExpectedExceptionAttribute`:

[PRE3]

However, this solution isn’t the best: `ExpectedException` is actually a poor design. The exception it expects may be thrown by *any* of the methods called by your unit test method. A better design checks that a *particular* piece of code throws that exception, not the unit test as a whole.

Most modern unit test frameworks include `Assert.ThrowsAsync<TException>` in some form. For example, you can use xUnit’s `ThrowsAsync` like this:

[PRE4]

###### Warning

Do not forget to `await` the task returned by `ThrowsAsync`! The `await` will propagate any assertion failures that it detects. If you forget the `await` and ignore the compiler warning, your unit test will always silently succeed regardless of your method’s behavior.

Unfortunately, several other unit test frameworks don’t include an equivalent `async`-compatible `ThrowsAsync`. If you find yourself in this boat, create your own:

[PRE5]

You can use the method just like it was any other `Assert.ThrowsAsync<TException>` method. Don’t forget to `await` the return value!

## Discussion

Testing error handling is just as important as testing the successful scenarios. Some would even say more important, since the successful scenario is the one that everyone tries before the software is released. If your application behaves strangely, it will be due to an unexpected error situation.

However, I encourage developers to move away from `ExpectedException`. It’s better to test for an exception thrown at a specific point rather than testing for an exception at any time during the test. Instead of `ExpectedException`, use `ThrowsAsync` (or its equivalent in your unit test framework), or use the `ThrowsAsync` implementation, as in the last code example.

## See Also

[Recipe 7.1](#recipe-async-test) covers the basics of unit testing asynchronous methods.

# 7.3 Unit Testing async void Methods

## Problem

You have an `async void` method that you need to unit test.

## Solution

Stop.

Rather than solving this problem, you should do your dead-level best to avoid it. If it’s possible to change your `async void` method to an `async Task` method, then do so.

If your method *must* be `async void` (e.g., to satisfy an interface method signature), then consider writing two methods: an `async Task` method that contains all the logic, and an `async void` wrapper that just calls the `async Task` method and awaits the result. The `async void` method satisfies the architecture requirements, while the `async Task` method (with all the logic) is testable.

If it’s impossible to change your method and you *must* unit test an `async void` method, then there is a way to do it. You can use the `AsyncContext` class from the `Nito.AsyncEx` library:

[PRE6]

The `AsyncContext` type will wait until all asynchronous operations complete (including `async void` methods) and will propagate exceptions that they raise.

###### Tip

The `AsyncContext` type is in the [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet package.

## Discussion

One of the key guidelines in `async` code is to avoid `async void`. I strongly recommend you refactor your code instead of using `AsyncContext` for unit testing `async void` methods.

## See Also

[Recipe 7.1](#recipe-async-test) covers unit testing `async Task` methods.

# 7.4 Unit Testing Dataflow Meshes

## Problem

You have a dataflow mesh in your application, and you need to verify it works correctly.

## Solution

Dataflow meshes are independent: they have a lifetime of their own and are asynchronous by nature. So, the most natural way to test them is with an asynchronous unit test. The following unit test verifies the custom dataflow block from [Recipe 5.6](ch05.html#recipe-dataflow-custom):

[PRE7]

Unit testing failures isn’t quite as straightforward, unfortunately. This is because exceptions in dataflow meshes are wrapped in another `AggregateException` each time they are propagated to the next block. The following example uses a helper method to ensure that an exception will discard data and propagate through the custom block:

[PRE8]

## Discussion

Unit testing of dataflow meshes directly is doable, but somewhat awkward. If your mesh is a part of a larger component, then you may find that it’s easier to just unit test the larger component (implicitly testing the mesh). But if you’re developing a reusable custom block or mesh, then unit tests like the preceding ones should be used.

## See Also

[Recipe 7.1](#recipe-async-test) covers unit testing `async` methods.

# 7.5 Unit Testing System.Reactive Observables

## Problem

Part of your program is using `IObservable<T>`, and you need to find a way to unit test it.

## Solution

System.Reactive has a number of operators that produce sequences (e.g., `Return`) and other operators that can convert a reactive sequence into a regular collection or item (e.g., `SingleAsync`). You can use operators like `Return` to create stubs for observable dependencies, and operators like `SingleAsync` to test the output.

Consider the following code, which takes an HTTP service as a dependency and applies a timeout to the HTTP call:

[PRE9]

The system under test is `MyTimeoutClass`, which consumes an observable dependency and produces an observable as output.

The `Return` operator creates a cold sequence with a single element in it; you can use `Return` to build a simple stub. The `SingleAsync` operator returns a `Task<T>` that is completed when the next event arrives. `SingleAsync` can be used for simple unit tests like the following:

[PRE10]

Another operator important in stub code is `Throw`, which returns an observable that ends with an error. The operator enables us to unit test the error case as well. The following example uses the `ThrowsAsync` helper from [Recipe 7.2](#recipe-async-testfail):

[PRE11]

## Discussion

`Return` and `Throw` are great for creating observable stubs, and `SingleAsync` is an easy way to test observables with `async` unit tests. They’re a good combination for simple observables, but they don’t hold up well once you start dealing with *time*. For example, if you wanted to test the timeout capability of `MyTimeoutClass`, the unit test would have to wait for that amount of time. That, however, would be a poor approach: it makes your unit tests unreliable by introducing a race condition, and it doesn’t scale well as you add more unit tests. [Recipe 7.6](#recipe-rx-testscheduling) covers a special way that System.Reactive empowers you to stub out time itself.

## See Also

[Recipe 7.1](#recipe-async-test) covers unit testing `async` methods, which is very similar to unit tests that await `SingleAsync`.

[Recipe 7.6](#recipe-rx-testscheduling) covers unit testing observable sequences that depend on time passing.

# 7.6 Unit Testing System.Reactive Observables with Faked Scheduling

## Problem

You have an observable that is dependent on time, and want to write a unit test that is not dependent on time. Observables that depend on time include ones that use timeouts, windowing/buffering, and throttling/sampling. You want to unit test these but do not want your unit tests to have excessive runtimes.

## Solution

It’s certainly possible to put delays in your unit tests; however, there are two problems with that approach: 1) the unit tests take a long time to run, and 2) there are race conditions because the unit tests all run at the same time, making timing unpredictable.

The System.Reactive (Rx) library was designed with testing in mind; in fact, the Rx library itself is extensively unit tested. To enable thorough unit testing, Rx introduced a concept called a *scheduler*, and *every* Rx operator that deals with time is implemented using this abstract scheduler.

To make your observables testable, you need to allow your caller to specify the scheduler. For example, you can take the `MyTimeoutClass` from [Recipe 7.5](#recipe-rx-test) and add a scheduler:

[PRE12]

Next, you can modify your HTTP service stub so that it also understands scheduling, then introduce a variable delay:

[PRE13]

Now you can go ahead and use `TestScheduler`, a type included in the System.Reactive library. `TestScheduler` gives you powerful control over (virtual) time.

###### Tip

`TestScheduler` is in a separate NuGet package from the rest of System.Reactive; you’ll need to install the [`Microsoft.Reactive.Testing`](http://bit.ly/ms-react-test) NuGet package.

`TestScheduler` gives you complete control over time, but you often just need to set up your code and then call `TestScheduler.Start`. `Start` will virtually advance time until everything is done. A simple success test case could look like the following:

[PRE14]

The code simulates a network delay of half a second. It’s important to note that this unit test *does not* take half a second to run; on my machine, it takes about 70 milliseconds. The half-second delay only exists in virtual time. The other notable difference in this unit test is that it isn’t asynchronous; since you’re using `TestScheduler`, all your tests can complete immediately.

Now that everything is using test schedulers, it’s easy to test timeout situations:

[PRE15]

Once again, the preceding unit test does not take 1 second (or 1.5 seconds) to run; it executes immediately using virtual time.

## Discussion

In this recipe we’ve just scratched the surface on System.Reactive schedulers and virtual time. I recommend that you start unit testing when you start writing System.Reactive code; as your code grows more and more complex, you can rest assured that `Microsoft.Reactive.Testing` is capable of handling it.

`TestScheduler` also has `AdvanceTo` and `AdvanceBy` methods, which enable you to gradually step through virtual time. These may be useful in some situations, but you should strive to have your unit tests only test one thing. To test a timeout, you could write a single unit test that partially advanced the `TestScheduler` and ensured that the timeout didn’t happen early, and then advanced the `TestScheduler` past the timeout value and ensured that the timeout did happen. However, I prefer to run separate unit tests as much as possible; for example, one unit test ensuring that the timeout didn’t happen early, and a different unit test ensuring that the timeout did happen later.

## See Also

[Recipe 7.5](#recipe-rx-test) covers the basics of unit testing observable sequences.