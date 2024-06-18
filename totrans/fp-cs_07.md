# Chapter 7\. Functional Flow

Calls to external systems, whether they be databases, web APIs or whatever are an absolute pain, aren’t they? Before making use of your data - the most important part of the function you’re writing, you have to:

1.  Catch and handle any exceptions. Maybe the network was glitching, or the DB server offline?

2.  Check what’s come back from the database is not NULL

3.  Check that there’s an actual, reasonable set of data, even if it isn’t null.

That’s a lot of tedious boilerplate, and all of that gets in the way of your actual business logic.

Use of the Maybe Discriminated Union from the previous chapter will help somewhat with returning something other than Null for records not found or errors encountered, but there’s still boilerplate required even then.

What if I were to tell you there were a way you could never have to see another unhandled Exception again? Not only that, but you’d never even need to use a Try/Catch block again. As for Null checks? Forget ‘em. You won’t ever have to do that again either.

Don’t believe me? Well, strap in, I’m going to introduce you to one of my favorite features of functional programming. Something I use all the time in my day job, and I’m hoping that after reading this chapter, so too might you.

# Maybe, Revisited

Mentioning that Maybe discriminated union from the last chapter - I’d like to revisit it now, but this time I’m going to show you how it can be even more useful than you could ever have imagined.

What I’m going to do is add in a version of the Map extension method I used a couple of chapters ago. If you remember the `Map` combinator back in Chapter 5 [“Chaining Functions”](ch05.html#Chapter5_map_combinator) works similarly to the LINQ Select method, except it acts on the entire source object, not individual elements from it.

This time though, I’m going to add a bit of logic inside the Map, something that will determine which actual type is going to come out. I’m giving it a different name this time - Bind^([1](ch07.html#idm45400858339280))

[PRE0]

So, what’s happening here? One of a few possible things:

*   The current value of `This` - i.e. the current object being held by the Maybe is a `Something` - i.e. an actual value, and the value held is non-default^([2](ch07.html#idm45400858334112)), in which case the supplied function is run, and whatever came out of it is returned in a new `Something`.

*   The current value of `This` is a `Something`, but the value inside it is default (Null, in most cases), in which case instead of a *Something*, we now return a `Nothing`

*   The current value of `This` is a Nothing, in which case, return another Nothing. No point doing anything else.

*   The current value of `This` is an Error. Again, no point doing anything except passing it on.

What’s the point of all this? Well, imagine the following procedural code:

[PRE1]

When you look at it, the actual purpose of this code is incredibly simple. Fetch an employee. If there are no issues, say hello to them. But, since Null and unhandled exceptions are a thing, we have to write so much defensive code. Null checks and `Try/Catch` blocks. Not just here, but all over our codebase.

What’s worse is that we make it the problem of the code calling this function to know what to do with it. How do we signal that there was an error, or that the employee wasn’t found? In my example I just return a string for whatever application we’ve written to display blindly. The other option would be to return some sort of return object with meta data attached (bool DataFound, bool ExceptionOccurred, Exception CapturedException - that sort of thing).

Using a Maybe and Bind function though, none of that is necessary. The code could be re-written like this:

[PRE2]

Think about the possible results for each bind I listed above.

If the Employee Repository returned a Null value, the next Bind call would identify a Something with a default (i.e. Null) value, and not execute the function that constructs a greeting string, instead it would return a Nothing.

If an error occurred in the repository (maybe a network connection issue, something impossible to predict or prevent) then the error would simply be passed on, instead of executing the function.

The ultimate point I’m making is that the arrow function that assembles a greeting will only ever execute if the previous step a) returned an actual value and b) didn’t throw an unhandled exception.

This means that the small function written above with use of the Bind method is functionally identical to the previous version, covered with defensive code.

It gets better…​

We’re not returning a string any more, we’re returning a Maybe<string>. This is a descriminated union that can be used to inform whatever is calling our function what the result of execution was, whether it worked, etc. That can be used in the outside world to decide how to handle the resulting value, or it can be used in subsequent chains of Bind calls.

Either like this:

[PRE3]

Or alternatively you could adapt the UserInterface module so that it takes a Maybe as a parameter:

[PRE4]

Swapping out a concrete value in an Interface for a Maybe<T> is a sign to the class that consumes it that there is no certainty that the operation will work, and forces the consuming class to consider each of the possibilities and how to handle them. It also puts the onus on deciding how to respond entirely in the hands of the consuming class. There’s no need for the class returning the Maybe to have any interest in what happens next.

The best description I’ve ever encountered for this style of programming was in a talk and related articles by Scott Wlashin called Railway Oriented Programming ^([3](ch07.html#idm45400857765248))

Wlashin desribes the process as being like a railway line with a series of points. Each set of points is a Bind call. The train starts on the Something line, and every time a function passed to a Bind is executed, the train either carries on to the next set of points, or switches to the Nothing path, and simply glides along to the station at the end of the line without doing any more work.

It’s a beautiful, elegant way of writing code, and cuts out so much boilerplate it isn’t even funny.

If only there were a handy technical term for this structure. Oh wait, there is! It’s called a Monad!

I said way back at the beginning that they might pop up somewhere. If anyone ever says to you that Monads are complicated, I hope you can see now that they’re wrong.

Monads are like a wrapper around a value of some kind. Like an envelope, or a burrito. They hold the value, but make no comment about what it’s actually set to.

What they do is give you the ability to hang functions off them which provide a safe environment to do perform operations without having to worry about negative consequences - such as null reference exceptions.

The Bind function is like a relay race - each call does some sort of operation, then passes its value to the next runner. It also handles errors and nulls for you, so you don’t need to worry about writing so much defensive code.

If you like, imagine it being like an explosion proof box. You have a package you want to open, but you don’t know whether it’ll be something safe like a letter^([4](ch07.html#idm45400857754608)) or could it be something explosive, just waiting to take you down when you lift the lid. If you pop it inside the Monad container, it can open safely or explode, but the Monad will keep you safe from the consequences.

That’s really all there is too it. Well, mostly.

In the rest of this chapter I’ll consider what else we can do with Monads, and what other sorts of Monads there are. Don’t worry though, the “hard” part is over now, if you’ve passed this point, and you’re still with me, then the rest of this book is going to be a piece of cake^([5](ch07.html#idm45400857752832)).

## Maybe and Debugging

A comment I hear sometimes hear regarding strings of Bind statements is that it’s harder to use debug tools in Visual Studio to step through the changes. Especially when you have scenarios like this:

[PRE5]

It is actually possible in most versions of Visual Studio, but you’d need make sure you keep mashing the “Step-in” key to enter the nested arrow functions inside the Bind call. It’s still hardly the best for working out what’s happening if a value isn’t being calculated correctly. It’s even worse when you consider Step-in will enter the Maybe’s Bind function and need a few more steps to be taken before seeing the result of the arrow function.

I tend to write my Binds one to a line, each storing a variable containing their individual output:

[PRE6]

Functionally these two samples are identical, it’s just that we’re capturing each output separately, rather than immediately feeding it into another function and discarding them.

The second sample is easier to diagnose issues with though, as you can inspect each intermediate value. Made easier due to the functional programming technique of never modifying a variable once set - this means that every intermediate step in the process is set in stone to work through what exactly happened, as well as how and where things went wrong in the event of a bug.

These intermediate values will remain in scope for the entirety of the life of whatever larger function they’re part of, though. So, if it’s an especially large function, and one of the intermediate values is hefty, it might be worth merging them so that the large intermediate vaue is de-scoped as early as possible.

This decision is mostly a matter of personal style, and one or two codebase constraints. Whichever you choose is fine.

## Map vs Bind

Strictly speaking, I’m not implementing the Bind function in accordance with the functional paradigm.

There *should* be two functions attached to the Maybe: Map and Bind. They’re nearly the same, but with a small and subtle difference.

The Map function is like the one I described in the previous section - it attaches to a Maybe<T1> and needs a function that gives you the value of type T1 from inside the Maybe and requires you to turn it into a type T2.

An actual Bind needs you to pass in a function that returns a Maybe of the new type - i.e. Maybe<T2>. It still returns the same result as the Map function,

[PRE7]

If, for example, you had a function that calls a database and returns a type of `Maybe<IEnumerable<Customer>>` to represent a list of customers that may or may not have been found - then you’d call that with a Bind function.

Any subsequent chained functions to change the `Enumerable` of customers into some other form would be done with a Map call, since those changes are data-to-data, not data-to-maybe.

Here’s how you might go about implementing a proper Bind:

[PRE8]

And an example of how to use it:

[PRE9]

Use this new Bind function and rename the previous one Map, and you’re conforming to the functional paradigm a little closer.

In production code however, I don’t personally do this. I just use a function called Bind for both purposes.

Why, you might ask?

It’s mostly to prevent confusion, in all honesty. There is a Map function that’s native to JavaScript but which operates like a C# `Select` on individual elements of arrays. In C# there’s also a Map function in Jimmy Bogard’s AutoMapper library^([6](ch07.html#idm45400857310352)), which is used to convert an array of objects from one type to another.

With both of these instances of a Map function already in use in many C# codebases, I thought adding another Map function into the mix might confuse other folks looking at my code. For this reason, I use Bind for all purposes, as there isn’t already a Bind function anywhere in C# or JavaScript - except for in libraries implementing the functional paradigm.

You can please yourself whether you use the more strictly accurate version with both Map and Bind, or the route that - in my opinion - is less confusing and more pragmatic, which is to simply use multiple instances of the Bind function to serve every purpose.

I’m going to carry on assuming that second option for the rest of this book.

## Maybe and the Primitives

This sounds like the title of an amazing pulp adventure novel that was never written. Probably involving our heroine - Captain Maybe, swinging to the rescue somewhere in a lost civilisation populated by aggressive cave dwelling nasties.

In fact, a primitive type in C# is one of a set of built-in types that *don’t* default to Null. This is a list of them:

*   bool

*   byte

*   sbyte

*   char

*   decimal

*   double

*   float

*   int

*   uint

*   nint

*   nuint

*   long

*   ulong^([7](ch07.html#idm45400857292336))

*   short

*   ushort

The point here is that if I were to use any of these on my `Bind` function from the previous sections, and set their value to 0, it would fall foul of the check against `default`, because most of these default to 0^([8](ch07.html#idm45400857288640))

Here’s an example of a unit test that would fail (I’m using XUnit with FluentAssertions for a friendlier, human-readable assert style):

[PRE10]

This test stores an integer with the value of 0 in a Maybe, then attempts to `Bind` it into a value 10 higher - i.e. it should equal 10\. In the existing code, the switch operation inside `Bind` would consider the value 0 to be default, and switch the return type from `Something<int>` to `Nothing<int>` and the function to add 10 wouldn’t be carried out, meaning that *output* in my unit test would be switched into a null, and the test would fail with a null reference exception.

Arguably though, this isn’t correct behavior, 0 is a valid value of integer.

It’s easily fixed with an additional line in the `Bind` function, however:

[PRE11]

The new line checks the first generic argument of the `Maybe<T>` - i.e. the “real” type of `T`. All of the types I listed at the beginning of this section would have a value of `IsPrimitive` set to `true`.

If I were to re-run my unit test with this modified `Bind` function, then the 0-valued `int` still wouldn’t match on the check against not being `default`, but the next line would match, because int is a primitive.

This does now mean that all primitives are *incapable* of being a `Nothing<T>`. Whether that’s right or wrong is a matter for you to assess for yourself.

You might want to consider it a `Nothing<T>` if *T* is a `bool`, for example. If that’s the case, another case would need to be added to the switch between the first two lines to handle the specific case of *T* being `bool`.

It might also be important to a calculation that it be possible for a boolean `false` to be passed into a function to perform a calculation. As I said, it’s a question you can best answer yourself.

One way to avoid this situation altogether is to always pass a nullable class around as *T* so that you can be sure that you’re getting the correct behavior when trying to decide whether what you’re looking at is `Something` or `Nothing`.

## Maybe and Logging

Another thing worth considering for the use of Monads in a professional environment is the all-important developer’s tool - Logging. It’s often crucially important to log information about the status of the progress of a function. Not just errors, but also all sorts of important information.

It’s possible to do something like this, of course:

[PRE12]

This is likely to balloon out of hand if you do much of this, though. Especially if there are many binds in the process that all require logging.

It might be possible to leave out the error log until the very end, or even all the way out in the Controller, or whatever else it was that ultimately originated this request. The error message would be passed from hand to hand untouched. But that still leaves ocassional log messages for Information or Warning purposes.

I prefer to add extension methods to the Maybe to provide a set of event handler functions:

[PRE13]

The way I’d use this then, is more like this:

[PRE14]

This is fairly usable, although it does have a drawback.

The OnNothing and OnError states will proliferate from Bind to Bind unmodified, so if you have a long list of Bind calls with OnNothing or OnError handler functions, they’ll all fire every time. Like this:

[PRE15]

In the code sample above, all three OnNothings will fire, and three Warning logs will be written. You may want that, or you may not. It might not be all that interesting after the first nothing.

I **do** have a solution for this issue, but it means quite a lot more coding.

Create a new instance of Nothing and Error that descent from the originals:

[PRE16]

We’d also need to modify the Bind function so that these are the types that are returned when switching from the Something path to one of these.

[PRE17]

Then finally, the handler functions need to be updated:

[PRE18]

What all of this means is that when a switch happens from Something to one of the other states, the Maybe switches to a state that not only signals that either Nothing or an Exception occurred, but also that nothing has yet handled that state.

Once one of the handler functions is called, and an unhandled state is found, then the callback is triggered to log, or whatever, and a new object is returned that maintains the same state type, but this time indicating that isn’t no longer unhandled.

This means that in my example above with multiple Bind calls with OnNothing functions attached, only the first OnNothing will actually be triggered, the rest will be ignored.

There’s nothing stopping you still using a pattern matching statement to examine the type of the Maybe to perform some action or other once the Maybe reaches its final destination, elsewhere in the codebase.

## Maybe and Async

So, I know what you’re going to ask me next. Look, I’m going to have to let you down gently. I’m already married. Oh? My mistake. Async and Monads. Yeah, OK. Moving on…​

How do you handle calls inside a Monad to processes that are async? Honestly, it’s not all that hard. Leave the Maybe Bind functions you’ve already written, and add these to the codebase as well:

[PRE19]

All I’ve done here is wrap another layer around the value we’re passing around.

The first layer is the Maybe - representing that the operation we’re trying may not have worked

The second layer is the Task - representing that an `async` operation needs to be carried out first, before we can get to the Maybe.

Using this out in your wider codebase is probably best done a line at a time so you can avoid mixing up the async and non-async versions in the same chain of Bind calls, otherwise you could end up with a `Task<T>` being passed around as a type, rather than the actual type, `T`. Also, it means you can separate each Async call out and use an await statement to get the real value to pass along to the next Bind operation.

## Nested Maybes

There is a problem scenario with the Maybes I’ve shown so far. This was something I only really came to realise existed once I’d changed an awful lot my Interfaces to have Maybe<T> as the return type for anything involving an external interaction.

Here’s a scenario to consider. I’ve created a couple of different data loaders of some description. They could be databases, web APIs, or whatever. Doesn’t matter:

[PRE20]

In some other part of the code I want to orchestrate calls to each of these interfaces in turn using a Bind call.

Note that the point of the `Maybe<string>` return type is that I can reference them through `Bind` functions and if any of the dataLoader calls fail, the subsequent steps *wont’* be executed and I’ll get a `Nothing<string` or `Error<string>` at the end to examine.

Something like this:

[PRE21]

What I find though is that this code won’t compile. Why do you suppose that is?

There are three function calls at work here, and all three return the type `Maybe<string>`. Look at what happens a line at a time:

1.  First GetStringOne returns a `Maybe<string>`. So far, so good.

2.  Next the Bind call attaches to the `Maybe<string>` and unpacks it to a string to pass to GetStringTwo, whose return type is popped into a new `Maybe` for safe keeping.

3.  The next bind call unwraps the return type of that last bind so that it’s only the return type of GetStringTwo - but GetStringTwo didn’t return a `string`, it returned `Maybe<string>`. So, on this second Bind call, x is actually equal to `Maybe<string>` which can’t be passed into GetStringThree!

I *could* solve this by directly accessing the value out of the Maybe stored in x, but first I’d need to cast it to a Something. What if it weren’t a something, though? What if an error had occurred in GetStringOne talking to the database? What if no string could be found?

I basically need a way to unpack a nested Maybe, but *only* in the event that it returns a Something containing a real value, in all other cases, we need to match its unhappy path (Nothing or Error).

The way I’d do it is with another Bind function to sit alongside the other two we’ve already created, but this one specifically handles the issue of nested Maybes.

I’d do it like this:

[PRE22]

What we’re doing here is taking the nested bind (`Maybe<Maybe<string>>`) and calling Bind on it, which unwraps the first layer, leaving us simply with `Maybe<string>` inside the Bind callback function, from there we can just do the exact same logic on the Maybe as in previous Bind functions.

This would need to be done for the `async` version as well:

[PRE23]

If you want another way to think about this process. Think of the `SelectMany` function in LINQ. If you feed it an array of arrays - i.e. a multi-dimensional array, then you get back a single-dimension, flat array. This handling of nested `Maybe` objects now allows us to do the same thing with Monads. This is actually one of the “Laws” of Monads - the properties that anything that calls itself a Monad is expected to follow.

In fact, that leads me neatly onto the next topic. What exactly are Laws, what are they for, and how do we make sure our C# Monads follow them too…​

# The Laws

Strictly speaking, to be considered a true Monad, there are a set of rules (known as *laws*) that you have to conform to. I’ll briefly talk through each of these laws, so you’ll know for yourself whether you’re looking at a real Monad or not.

## Left Identity Law

This states that a Monad, given a function as a parameter to its Bind method will return something that is entirely the equivalent of just running the function directly with no side-effects.

Here’s some C# code to demonstrate that:

[PRE24]

## Right Identity Law

Before I explain this one, I need to move back a few steps…​

Firstly, I need to explain Functors. These are functions that convert a thing, or list of things from one form into another. `Map`, `Bind` and `Select` are all examples of Functors.

The very simplest functor that exists is the Identity functor. The identity is a function that given an input, returns it back again unaltered, and with no side-effects. It can have its uses, when you’re composing functions together.

The only reason I’m interested in it here and now is that it’s the basis of the second Monad law - the Right Identity Law. This means a Monad, when given an Identity Functor in its Bind function, will return the original value back with no side effects.

I could test the Maybe I created in the last chapter like this:

[PRE25]

All this means is that the Maybe takes a function that doesn’t result in an error or `Null`, executes it, then returns exactly whatever comes out of it, and nothing else.

The basic gist of both of these first two laws is simply that the Monad can’t interfere in any way with the data coming in or going out, or in the execution of the funtion provided as a parameter to the Bind method. A Monad is simply a pipe down which functions and data flow.

## Associativity law

The first two laws should be fairly trivial, and my Maybe implementation fulfills them both. The last law, the Associativity Law, is a bit harder to explain.

It basically means that it doesn’t matter how the Monads are nested, you always end up with a single Monad containing a value at the end.

Here’s a simple C# example:

[PRE26]

Look back a few sections to my description of how to deal with nested Maybes[“Nested Maybes”](#Chapter_7_nested_maybes) and you’ll see how this is implemented.

With any luck, now we’ve looked at the three Monad laws, I’ve proven that my Maybe is a proper, no-holds barred, honest to dog Monad.

In the next section, I’ll show you another Monad you might use to do away with the need for storing variables that need to be shared around.

# Reader

Let’s imagine for a moment that we’re putting together a report of some kind. It does a series of pulls of data from an SQL Server database.

First we need to grab the record for a given user.

Next, using that record, we get their most recent order from our entirely imaginary bookshop^([9](ch07.html#idm45400854867920)).

Finally, we turn the most recent Order record into a list of items from the order and return a few details from them in a report.

We want to take advantage of a monad-style Bind operation, so how do we ensure the data from each step is passed along with the database connection object? That’s no problem, we can throw together a Tuple and simply pass along both objects.

[PRE27]

This is a workable solution, but it’s a bit ugly. There’s a few repeated steps that exist only to alow the connection object to be persisted between Bind operations. It’s harming the readability of the function.

It’s also not pure, if you think about it. The function is having to create a database connection, which is a form of side-effect.

There is another functional structure we can use to solve all of these problems - the Reader Monad. It’s functional programming’s answer to dependency injection, but on the functionl-level, rather than into a class.

In the case of the function above, it’s the IDbConnection that we want to inject in, so it can be instantiated elsewhere, leaving the MakeOrderReport pure - i.e. free of any side effects.

Here’s an incredibly simple use of a Reader:

[PRE28]

What we’ve defined here is a Reader which takes a function that it stores, but doesn’t execute. The function has an “environment"variable type as its parameter - this is the currently unknown value we’re going to inject in the future, and it returns a value based on that parameter, in our case an integer.

The “int → string” function is stored in the Reader on the first line, then on the second line we call the “Run” function which provides the missing environment variable value, here 100\. Since the environment has finally been provided, the Reader can therefore use it to return a real value.

Since this is a Monad, that also means we should have Bind functions to provide a flow. This is how they’d be used:

[PRE29]

Note that the type of the “reader” variable is `Reader<int, string>`. That’s because every Bind call places a wrapper around the previous function which has the same alternate return type, but a different parameter.

In the first line with the parameter `e => e * 100`, that function wil be exuecute later, afer the Run And, so on…​

This is a more realistic use of the Reader:

[PRE30]

Alternatively, you can actually simply return the Reader, and allow the outside world to continue using Bind calls to modify it further before the Run function turns it into a proper value.

[PRE31]

This way, the same function can be called many times, with the Reader’s Bind function used to convert it to the actual type I want.

For example, if I wanted to get the customer’s order data:

[PRE32]

Think of it as you’re creating a box that can only be opened by inserting a variable of the correct type.

The use of the Reader also means it’s easy to inject a mock IDbConnection into these functions and write unit tests based on them.

Depending on how you want to structure your code, you could even consider exposing Readers on interfaces. It doesn’t have to be a depenency like a DbConnection that you pass in, it could be an Id value for a database table, or anything you like. Something like this, perhaps:

[PRE33]

There are all sorts of ways you could use this, it’s all a matter of what suits you, and what you’re trying to do.

In the next section, I’ll show a variation on this idea - the State Monad.

# State

In principle, a State Monad is very similar to a Reader. A container is defined that requires some form of state object to convert itself into a proper final piece of data. Binds are usable to provide additional data transformations, but nothing will happen until the State is provided.

What makes it difference is two things:

1.  Instead of an “Environment” type, it’s known as the “state” type

2.  There are two items, not one being passed between Bind operations.

In a Reader, the original Environment type is only seen at the beginning of the chain of Binds. With a State Monad, it persists all the way through to the end. The State type, and whatever the current value is set to are stored in a Tuple which is passed from one step to the next. Both the value and the State can be replaced with new values each time. The Value can change types, but the State is a single type throughout the whole process that can have its vaules updated, if required.

You can also arbitrarily fetch or replace the State object in the State Monad at any time using functions.

My implementation doesn’t strictly adhere to the way you’ll see it in languages like Haskell, but I’d argue that implementations of that kind are a pain in C#, and I’m not convinced that there’s any point to doing it. The version I’m showing you here could well have some use in daily C# coding.

[PRE34]

The State monad doesn’t have multiple states, so there’s no need for a base abstract class. It’s just a simple class with two properties - a Value and a State (i.e. the thing we’ll pass along through every instance).

The logic has to be implemented in extension methods:

[PRE35]

As usual, not a lot of code to implement, but plenty of interesting effects.

This is the way I’d use it:

[PRE36]

The idea here is that the state object is being passed along the chain as *s*, and the result of the last Bind is passed as *x*. Based on both of those values, you can determine what the next value should be.

This just leaves out the ability to update the current state. I’d do that, with this extension method:

[PRE37]

Here’s a simple example for the sake of illustration:

[PRE38]

Using this method, you can have arrow functions with a few bits of state that will flow from one Bind operation to the next, and which you can even update when needed. It prevents you from being either forced to turn your neat arrow function into a full-function with curly braces, or passing a large, ungainly Tuple containing the readonly data through each Bind.

This implementation has left off the form you''ll find in Haskell, where the initial State value is only passed in when the complete chain of Binds has been defined, but I’d argue that in a C# context this version is more useful, and certainly an awful lot easier to code!

## Maybe a State?

You may notice in the previous code sample, there is no functionality to use the Bind function like a Maybe, to capture error conditions and null results coming back in one or other of several possible states. Is it possible to merge the Maybe and the Reader into a single monad that both persists a State object **and** handles errors?

Yes, and there are several ways you could accomplish it, depending on how exactly you’re planning to use it. I’ll show you my preferred solution. First, I’d adjust the State class so that instead of a value, it stores a Maybe containing the value.

[PRE39]

Then Id adjust the Bind function to take into account the Maybe, but without changing the signature of the function:

[PRE40]

The usage is just about exactly the same, except that Value is now of type Maybe<T>, instead of simply T. That only really affects the return value from the container function, though:

[PRE41]

Whether you want to merge the concepts of the Maybe and the State Monad in this way, or would rather keep them separately is entirely down to you.

If you do follow this approach, you’d just need to make sure to use a Switch expression to translate the Maybe into a single, concrete value at some point.

A last thing to bear in mind too - The CurrentValue object of the State Monad doesn’t have to be data, it can be a `Func` delegate too, allowing you to have a bit of functionality ported betweeen Bind calls.

In the next section I’m going to look at what *else* might be a Monad you’ve already been using in C#.

# Examples You’re already using

Believe it or not, you’ve already most likely been using Monads for a while already if you’ve been working with C# for any amount of time. Let’s take a look at a few examples.

## Enumerable

If an Enumerable isn’t a Monad, it’s as close as it gets, at least once we invoke LINQ - which as we already know is developed based on functional programming concepts.

The Enumerable `Select` method operates on individual elements within an enumerable, but it still obeys the Left Identity law:

[PRE42]

and the Right Identity Law:

[PRE43]

That just leaves the Associativity law - which is still a necessity to be considered a true Monad. Does Enumerable follow that? It does, of course. By use of SelectMany.

Consider this:

[PRE44]

There we have it, nested Enumerables outputted as a single Enumerable. That’s the Associativity law. Ipso facto, QED, etc. Yeah. Enumerables are Monads.

## Task

What about Tasks? Are they monads too? I bet you a beer of your choice^([10](ch07.html#idm45400853379040)) that they’re absolutely monads, and I can prove it.

Let’s run through the laws once again.

The Left identity law, a function call with a Task should match calling the function call directly. That’s slightly tricky to prove, in that an async method always returns a type of `Task` or `Task<T>` - which is the same as `Maybe<T>` in many ways, if you think about it. It’s a wrapper around a type that might or might not resolve to actual data. But, if we move back a level of abstraction, I think we can still demonstrate that the law is obeyed:

[PRE45]

I’m not saying that this is C# code I’d necessarily be proud of, but it does at least prove the point, that whether I call `op` through an async wrapper method or not, the result is the same. That’s the Left Identity Law confirmed. How about the right Identity Law? Honestly, that’s roughly the same code again:

[PRE46]

That’s the Identity laws settled. What about the equally-important Associativity law? Believe it or not, there is a way to demonstrate this with Tasks.

[PRE47]

Here we have a `Task<int>` being passed into another `Task<int>` as a parameter, but with nested calls to `await`, it’s all possible to flatten out to a simple `int` - which is the actual type of result.

Hopefully I’ve earned my beer? Mine’s a pint^([11](ch07.html#idm45400853159696)), please. A European-style half-litre is fine as well.

# Other Structures

Honestly and truly - if you’re OK with my version of the Maybe monad, and you aren’t bothered about going further, then feel free to skip ahead to the next chapter. You can easily achieve most of what you’re likely to want to achieve with Maybe alone. I’m going to describe a few other kinds of Monad that exist out there in the wider functional programming language world, which you *might* want to consider implementing in C#.

These Monads might be of interest if squeezing out the last few vestiges of non-functional code from C# is your intention. They might also be of interest from a theoretical perspective. It’s entirely up to you though, if you want to take the Monad concept further and continue to implement these.

Now, strictly speaking the version of the Maybe monad I’ve been building up over this chapter and the previous chapter was a mix of two different monads.

A true Maybe monad has only two states - Something (or Just) and Nothing (or Empty). That’s it. The monad for handling error states is the Either (aka Result) monad. That has two states - Left and Right.

Right is the “happy” path, where every function passed into the Bind command worked, and all is right with the world.

Left is the “unhappy” path where an error of some kind occurred, and the error is contained in the Left.

The Left and Right naming convention is presumably coming from the recurring concept in many cultures that the Left hand is evil and the Right hand is good. This is even captured in bits of our language - the Latin word for “left” is literally “Sinister”. In these enlightened days however, we no longer drive out left-handed folks^([12](ch07.html#idm45400853120800)) from our homes or whatever it was they used to do.

I won’t spell out that implementation here, you can basically achieve it by taking my version of the Maybe and removing the “Nothing” class.

Similarly, you can make a true Maybe by removing the Error class - although I’d argue that by combining the two into a single entity, you’ve got something that can handle just about any situation you’re likely to encounter when interacting with external resources.

Is my approach pure and fully correct classical functional theory? No. Is it useful in production code? 100% yes.

There are many more monads beyond the Maybe and Either, and if you move into a programming language like Haskell you’ll likely make regular use of them. Here are a few examples:

*   Identity - a monad that simply returns back whatever value you feed in. These have their uses when getting into deeper functional theory in a more purely functional language, but don’t really have an application here in C#.

*   State - used to run a series of operations on a value. Not entirely unlike a `Bind` method, but there’s also a state object that’s passed along as well, that is used as an additional object to the calculations. In C#, we’re just as well using a LINQ `Aggregate` function, or a Maybe `Bind` function with a tuple or something similar to pass the necessary state objects through.

*   IO - Used to allow interactions with external resources without introducing impure functions. In C#, we can follow the Inversion of Control pattern (i.e. Dependency Injection) to allow us to get around any issues for testing, etc.

*   Environment- Known as the Reader monad in Haskell. Often used for processes like Logging, to wrap away the side-effects. This is useful if you’re trying to ensure your language is enforcing the strict rules of functional programming, but I don’t see any benefit in C#.

As you can see from the list above, there are many other monads available in the world of functional programming, but I’d argue that most or all of them don’t provide any real benefit to us. At the end of the day, C# is a hybrid functional/object-oriented language. It has been extended to allow support for functional programming concepts, but it’s never going to be a purely functional language, and there’s no benefit to trying to treat it as such.

I strongly recommend experimenting with the Maybe/Either monad, but beyond that, I honestly wouldn’t bother, unless you’re curious to see just how far you can push the idea of functional programming in C#^([13](ch07.html#idm45400853112320)). It’s not something for your production environment, though.

In the final section, I’ll provide a complete worked example of how to use monads in an application.

# A Worked Example

OK, here we go. Let’s put it all together in one great, epic heap of monady-functional goodness. We’ve already talked holidays in our examples this chapter, so this time I’m going to focus on how we’re actually going to get to the airport - this will need a series of look-ups, data transformations and all of which would normally require error handling and branching logic if we were to follow a more conventional, Object-oriented approach. I hope you’ll agree that using functional programming techniques and monads, it looks quite considerably more elegant.

First-off, we need our interfaces. I’m not actually going to code each and every single dependency our code has, so I’m just going to define the interfaces we’ll need:

[PRE48]

Having done that, I’ll write the code to consume them all. The specific scenario I’m imagining is this - It’s the near future. Driverless cars have become a thing. To the extent that most people no longer own personal vehicles, and simply use an app on their phones to have a car brought out of the cloud of automated vehicles, direct to their homes^([14](ch07.html#idm45400853081120)).

The process is going to be something like this:

1.  The initial inputs are the starting location & destination, provided by the user.

2.  Each of these locations need to be looked up in a mapping system and converted to proper addresses

3.  The user’s account needs to be fetched from the internal data store

4.  The route needs to be checked with a traffic service

5.  A pricing service has to be called to determine a charge for the journey

6.  The price is returned to the user.

In code, that process might look something like this:

[PRE49]

Far easier this was, isn’t it? Before I end this chapter, I want to unpick a few details of what’s happening in the code sample, above.

Firstly, there’s no error handling anywhere. Any of those external dependencies could result in an error being throwm, or no details being located in their respective data stores. The monad `Bind` function handles all of that logic. If - for example - the router was unable to determine a route (maybe a network error occurred), then the Maybe would be set to an `Error<Route>` at that point, and none of the subsequent operations would be executed. The final return type would be `Error<decimal>`, because the `Error` class is re-created at each step, but the actual `Exception` is passed between instances. The outside world is responsible for doing something with the final returned value, until that .

If we followed the OO approach to this code, then the function would most likely be 2-3 times longer, in order to include `Try/Catch` blocks and checks against each object to confirm that they’re valid.

I’ve used Tuples in cases where I want to build up a set of inputs. In the case of the Address objects, this means that in the event that the first address isn’t found, the lookup for the second won’t be attempted. It also means that both of the inputs required by the second function to be run are available in a single location, which we can then access with another `Bind` call (assuming there’s a real value returned by the address lookup).

The final few steps don’t actualy involve calls to external dependencies, but by continuing to use the `Bind` function, anything inside its parameter lambda expression can be written with the assumption that there is a real value available, because if there weren’t, the lambda wouldn’t be executed.

And there we have it, a pretty much fully functional bit of C# code. I hope it was to your liking.

# Conclusion

In this chapter, we explored the dreaded functional programming concept that has been known to make grown developers tremble in their inexpensive footware. All being well, this shouldn’t be a mystery to you any longer.

I’ve shown you how:

*   to massively reduce the amount of required code

*   to introduce an implicit error handling system.

All using the Maybe Monad, along with how to make one yourself.

In the next chapter, I’ll be taking a brief look at the concept of currying. I’ll see you on the next page.

^([1](ch07.html#idm45400858339280-marker)) I don’t know why that name gets used. I really don’t. It’s pretty common though, and something of a standard. Just bear with me.

^([2](ch07.html#idm45400858334112-marker)) I would say non-null, but integers default to 0 and booleans to false

^([3](ch07.html#idm45400857765248-marker)) See here to read the article, it’s worth your while: [*https://fsharpforfunandprofit.com/rop/*](https://fsharpforfunandprofit.com/rop/)

^([4](ch07.html#idm45400857754608-marker)) or living Schrödinger’s cat. Actually, *is* that the safe option? I’ve owned cats, I know what they’re like!

^([5](ch07.html#idm45400857752832-marker)) Cheesecake preferably. Paul Hollywood would be very disappointed to learn I don’t like many cakes, but New York style Cheesecake is most certainly one of them!

^([6](ch07.html#idm45400857310352-marker)) You can read about that here: [*https://automapper.org/*](https://automapper.org/) It can be used for quickly and easily converting between types, if that’s something you do a lot in your code

^([7](ch07.html#idm45400857292336-marker)) Not a kind of tea! Nor is it a porcine character from Dragonball

^([8](ch07.html#idm45400857288640-marker)) Except bool, which defaults to false and char which defaults to *\0*

^([9](ch07.html#idm45400854867920-marker)) I’m the imaginary owner, you can be the imaginary person that helps customers find what they want. Aren’t I generous?

^([10](ch07.html#idm45400853379040-marker)) I like dark ales, and european-style lagers. That strong stuff they make in Minnesota is terrific too

^([11](ch07.html#idm45400853159696-marker)) That’s 568ml. I’m aware other countries have other definitions of the word

^([12](ch07.html#idm45400853120800-marker)) My brother among them. Hi, Mark - you’ve been mentioned in a programming book! I wonder whether you’ll ever know about it?

^([13](ch07.html#idm45400853112320-marker)) C# rather than .NET in general - F# is .NET as well.

^([14](ch07.html#idm45400853081120-marker)) This all sounds great to me, I don’t much like driving & I get terribly car sick if I’m a passenger. I for one welcome our driverless car overlords.