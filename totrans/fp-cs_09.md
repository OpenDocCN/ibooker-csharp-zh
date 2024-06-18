# Chapter 9\. Indefinite Loops

We’ve seen in previous chapters how functional programming replaces `For` and `ForEach` loops with LINQ functions like `Select` or `Aggregate`. That’s absolutely terrific - provided you are working with a fixed-length array, or an `Enumerable` that will determine for itself when it’s time to finish iterating.

What do you do though, when you aren’t at all sure how long you’ll want to iterate for? What if you’re iterating indefinitely until a condition is met?

Here’s an example of some non-functional code that shows the sort of thing I’m talking about, losely based on the board game Monopoly. Let’s imaging you’re stuck in jail, you naughty individual! The rules for getting out are one of the following:

*   Pay 50 in whichever currency you play in (That’d be $50 if you’re in the US, or £50 for me, here in the UK^([1](ch09.html#idm45400850401728)))

*   Roll a double

*   Use a “Get out of Jail Free” card

In a real game of Monopoly, there are other player’s turns to consider, but I’m simplifying it down to looping indefinitely until one of these conditions are met. I’d probably add some validation logic in here too if I were doing this for real, but once again, I’m keeping it simple.

[PRE0]

You can’t possibly do the above with a `Select` statement. We can’t say when the criteria will be met, and we’ll continue to iterate around the `While` loop until one of them are.

So how can we make this functional? A `While` loop is a statement (specifically a control flow statement), and as such it’s not preferred by functional programming languages.

There are a few options, and I’ll describe each of them, but this is one of those areas where some sort of trade-off is required. Each of the choices have consequences, and I’ll do my best to consider their respective pros and cons.

Buckle up your seatbelts, here we go…​

# Recursion

The classic functional programming method for handling indefinite loops is to use recursion. In brief, for those of you unfamiliar with it - Recursion is the use of a function that calls itself. There will be a condition of some sort too that determines whether there should be another iteration, or whether to actually return data.

If the decision is made at the end of the recursive function, this is known as *tail recursion*.

A purely recusive solution to the Monopoly problem might look like this:

[PRE1]

Job done, right? Well, not really, and I would think very carefully before using a function like the one above. The issue is that every nested function call adds a new item onto the Stack in the .NET runtime, and if there are a lot of recursive calls, then that can either negatively effect performance or else kill the application with a Stack Overflow Exception.

If there are guaranteed only to be a handfull of iterations, then there’s nothing fundamentally wrong with the recursive approach. You’d also have to be sure that this is revisited if the code’s usage is ever significantly changed following an enhancement. It could turn out that this rarely used function with a few iterations one day turns into something heavily used with hundreds of iterations. If that ever happens, then the business might wonder why their wonderful application suddenly becomes near unresponsive almost overnight.

So, like I said, think very carefully before using a recursive algorithm in C#. This has the advantage of being relatively simple and not requiring you to write any boilerplate to make it happen.

###### Note

F#, and many other more strongly functional languages, have a feature called *Tail Optimised Recursion Calls*, which means it’s possible to write recursive functions without them exploding the stack. This isn’t available in C# however, and there are no plans to make it available in the future, either. Depending on the situation, the F# optimization will either create Intermediate Language (IL) code with a `while(true)` loop, or else make use of an IL command called `goto` to physically move the execution environment’s pointer back to the beginning of the loop.

I did investigate the possibility of referencing a generic Tail Optimised Recursion Call from F# and exposing it via a compiled DLL to C#, but that has its own performance issues that makes it a waste of effort.

There’s another possibility I’ve seen discussed online, and that’s to add a post-build event that directly manipulates the .NET Intermediate Language that C# compiles to so that it makes retrospective use of the F# Tail Optimization feature. That’s very clever, but sounds too much like hard work to me. It would be a potential extra maintainance task too.

In the next section, I’ll look into a technique to simulate Tail Optimised Recursion Calls in C#.

# Trampolining

I’m not entirely sure where the term *Trampolining* came from, but it pre-dates .NET. The earliest references I could find were academic papers from the 90s, looking at implementing some of the feaures of LiSP in C. I’d guess it’s a little older even than that, though.

The basic idea is that you have a function that takes a *thunk* as a parameter - a thunk being a block of code stored in a variable. In C# these are implemented as `Func` or `Action`.

Having got the thunk, you create an indefinite loop with `while(true)` and some way of assessing a condition that determines whether the loop should terminate. This might be done with an additional `Func` that returns a `bool` or else some sort of wrapper object that needs to be updated with each iteration by the thunk.

But at the end of the day, what we’re looking at is basically hiding a `while` loop at the back of our codebase. It’s true that `while` isn’t purely functional, but this is one of those places where we might need to compromise. Fundamentally, C# is a hybrid language, supporting both OO and FP paradigms. There are always going to be places where it’s not going to be possible to have it behave in exactly the way F# does. This is one of them.

There are a number of ways that you could implement trampolining, but this is the one I’d tend to go for:

[PRE2]

By attaching to type `T`, which is a generic, this extension method therefore attaches to everything in the C# codebase. The first parameter is a `Func` delegate that updates the type that `T` represents to a new form based on whatever rules the outside world defines. The second is another `Func` which returns the condition that will cause the loop to terminate.

Since this is a simple `While` loop, there aren’t any issues with the size of the stack. It’s not pure functional programming, though. It’s a compromise. At the very least though, it’s at least a single instance of a `while` loop that’s hidden somewhere, deep in the codebase. It may also be that one day, Microsoft will release a new feature that enables proper tail optimized recursion calls to be implemented somehow, in which case this function can be re-implemented and the code should continue to work as it did, but with one instance fewer of imperative code features.

Using this version of indefinite iteration, the Monopoly code would now look like this:

[PRE3]

There *is* another you are free to use to implement - Trampolining. Functionally it behaves the same as a hidden `while` loop, and it is pretty much the same in terms of performance as well.

I’m not necessarily convinced there’s any additional benefit, and I *personally* feel it looks a little less friendly than a `while` loop, but it is probably ever so slightly more in accordance with the functional paradigm, in that it dispenses with the `while` statement.

Use this if you prefer, but it’s a matter of personal preference, in my view.

This version is implemented using a C# command which under just about any other circumstance I’d *implore* you not to use. It’s something that has existed in coding since at least the days of BASIC, and is still around in some form today - the `goto` command.

In BASIC, you could literally move to any arbitrary line of code you wanted by calling `goto` and specifying the line number. That’s often how loops were implemented in BASIC. In c# however, you need to create tags, and it’s only to these tags that `goto` will move to.

Here’s a re-implementation of `IterateUntil` using two tags. One called *LoopBeginning* which is the equivalent of the *{* character at the beginning of a `while` loop. The second tag is called *LoopEnding* and represents the end of the loop, or the *}* character of the `while` loop.

[PRE4]

I’ll leave it to decide which version you prefer. They’re pretty much equivalent. Whatever you do though, don’t go using `goto` anywhere else in your code unless you absolutely, positively, completely and utterly know what you’re doing and why there’s literally *no* better option.

Like a certain snake-loving, noseless, evil wizard - the `goto` command is both great and yet also terrible if used unwisely.

It’s powerful, in that you can create effects, and improve efficiency (in some cases) in ways that are impossible via any other means. It’s dangerous also, in that you’ve no longer got a predictable order of operations - during execution the pointer can jump to an arbitrary point in the codebase, irrespective of whether it makes any sense at all to do so. Used improperly, you could end up with inexplicable, hard to debug issues in your codebase.

Use the `goto` statement with great caution.

There is a third option too, which requires quite a bit more boilerplate, but ultimately looks a little friendlier than the previous versions.

Have a look, and see what you think.

# Custom Iterator

The third option is to hack around with `IEnumerables` and `IEnumerators`. The thing about `IEnumerables` is they aren’t actually arrays, they’re just pointers to the “current” item of data, and an instruction on how to go and fetch the next item. That being the case, we can create our own implementation of the IEnumerable interface, but with our own behavior.

In the case of our Monopoly example, we want an `IEnumerable` that’s going to iterate until the user has selected a method to get out of jail, or else rolled a double.

We start by creating an implementation of `IEnumereable` itself, which only has a single function that has to be impemented: `GetEnumerator()`. The `IEnumerator` is the class that sits behind the scenes and actually does the work of enumerating. That’s what we’ll move onto next.

## Anatomy of an Enumerator

This is what the `IEnumerator` Interface effectively looks like (It inherits a few functions from other Interfaces, so a couple of these funtions are here to satisfy the inheritance requirements):

[PRE5]

Each of these functions has a very specific job to do:

Table 9-1\. Component functions of IEnumerator

| Function | Behavior | Returns |
| --- | --- | --- |
| Current | Get the current data item | The current item, or null if iteration hasn’t started yet |
| IEnumerator.Current | As Current, this is the same item, referenced from another Interface that IEnumerable implements | As Current |
| Dispose | break down everything in the Enumerable to implement IEnumerator | Void |
| MoveNext | Move to the next item | True if another item is found, false if the enumeration process is complete |
| Reset | Move back to the beginning of the data set | void |

Most of the time, an `Enumerator` is simply enumerating over an array, in which case I’d imagine the implementation *probably* should probably be something vaguely like this:

[PRE6]

I expect the real code by Microsoft is likely far more complicated than this - you’d hope there would be a lot more error handling and parameter checking, but this simple implementation gives you an idea of what sort of a job it is that the `Enumerator` does.

## Implementing Custom Enumerators

Knowing how it works under the surface, you can see how it’s possible to implement any behavior whatsoever that you’d like in an `Enumerable`. I’ll show you a few examples of how powerful this technique can be by by creating an `IEnumerable` implementation that only iterates through every *other* item in an array by putting the following code in `MoveNext`:

[PRE7]

How about an `Enumerator` that loops over every item twice, effectively creating a duplicate of each item when enumerating:

[PRE8]

Or an entire implementation that goes backwards, starting with an Enumerator outer wrapper:

[PRE9]

And aftwards the actual `Enumerator` that drives the backwards motion:

[PRE10]

The usage of this backwards enumerable is pretty much exactly the same as a normal enumerable:

[PRE11]

So, now that you’ve seen how easy it is to create your own `Enumerable` with whatever custom behavior you want, it should be easy enough to conjure up an `Enumerable` that iterates indefinitely.

## Indefinitely Looping Enumerables

Try saying this section title ten times fast!

As you saw in the previous section though, there’s no special reason that an `Enumerable` has to start at the beginning and loop to the end. We can make it behave in absolutely any way we care to.

What I want to do in this case, is - instead of an array - I want to pass in a single state object of some kind, along with a bundle of code (i.e. a Thunk, or `Func` delegate) for determining whether the loop should continue or not.

Working backwards, the first thing I’ll make is the `Enumerator`. This is an entirely bespoke enumeration process, so I’m not going to make any effort to make it generic in any way. The logic I’m writing wouldn’t make sense outside of a game state object.

I might want to do several different iterations in my hypothetical Monopoly implementation though, so I’ll make the operation and loop termination logic somewhat generic.

[PRE12]

That’s the hard bit done!! We have an engine under the surface that’ll allow us to iterate through successive states until we’re finished - whatever we decide “finished” means.

Next item required is the `IEnumerable` to run the `Enumerator`. That’s pretty straightforward:

[PRE13]

Everything is now in place to carry out a custom iteration. I just need to define my custom logic, set up the iterator.

[PRE14]

There are a couple of options for how to handle the iteration itself, and I’d like to take a little time out to discuss each of those options in more detail.

## Using Indefinite Iterators

Strictly speaking, as a fully-fledged `Iterator` any LINQ operation can be applied, as well as a standard `ForEach` iteration.

`ForEach` would probably be the simplest way to handle this iteration, but it wouldn’t be strictly functional. It’s up to you, if you want to compromise by adding a statement in a limited capacity, or want to seek out a more purely functional alternative. It might look like this:

[PRE15]

That wouldn’t give me too many causes for concern in production code, honestly. But, what we’ve done is negated all of the work we’ve put into attempting to get rid of non-functional code from our codebase.

The other options involve the use of LINQ. As a fully-fledged `Enumerable`, our GameIterator can have any LINQ operations applied to it. Which ones would be the best, though?

`Select` would be an obvious starting place, but it might not entirely behave as you’d expect. Usage is pretty much the same as any normal `Select` list operation you’ve ever done before:

[PRE16]

The trick here is that we’re treating gameIterator as an array, so `Select` -ing from it will result in an array of game states. What you’ll basically have is an array of every intermediate step the user has gone through, finishing with the final state in the last element.

The easy way to reduce this down to simply the final state is to substitute `Select` for `Last`:

[PRE17]

This assumes, of course that you aren’t interested in the intermediate steps. It might be that you want to compose a message to the user for each state update, in which case you might want to select, and provide a transformation.

Something like this, perhaps:

[PRE18]

That eradicates the actual game state, though, so possibly `Aggregate` might be a better option:

[PRE19]

The *x* in each iteration of the `Aggregate` process is an updated version of the Game State, and it’ll carry on aggregating until the declared end condition is met. Each pass appends a message to the list, so what you finally get at the end is a `Tuple` containing an array of strings, which are messages to pass to the player, and the final version of the game state.

Bear in mind that any use of LINQ statements that will terminate the iteration early in some manner - `First`, `Take`, etc. will also premeturely end this iteration proces, possibly in our instance with the player still in jail.

Of course, this might be a behavior you actually want! For example, maybe you’re restricting the player to just a couple of actions before moving onto another part of the game, or another player’s turn. Something like that.

There are all sorts of possibilities for the logic you could come up with, playing with this technique.

# Conclusion

We’ve looked into how we can implement indefinite iteration in C# without making use of the `ForEach` statement, which results in cleaner code with fewer possible side effects of execution.

It’s not strictly possible to do this purely functionally, and several options are available - all of which carry some level of compromise to the functional paradigm, but this is the nature of working in C#.

Which option - if any - you wish to make use of is entirely a matter of personal choice and whatever constraints apply to your project.

Please be very cautious with the use of recursion, though. It’s a fast method of iteration that’s purely functional, but if you aren’t careful, it can lead to significant performance issues when it comes to memory usage.

In the next chapter, I’ll look at a nice way to take advantage of pure functions to improve performance in your algorithms.

^([1](ch09.html#idm45400850401728-marker)) This probably doesn’t work in countries like india. INR 50 won’t get you much. This said, do you *really* think you can buy a whole street for even as much as $200!