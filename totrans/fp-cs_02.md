# Chapter 2\. What Can We Do Already?

Some of the code and concepts discussed in this chapter may seem trivial to some, but bear with me. I don’t want to introduce too much too soon. More experienced developers might like to skip ahead to Chapter 3, in which I talk about the more recent developments in C# for Functional progammers, or Chapter 4 where I demonstrate some novel ways to use features you might already be familiar with to achive some Functional features.

In this chapter, I’m going to look at the Functional Programming features that are possible in just about every C# codebase in use in production today. I’m going to assume at least .NET Framework 3.5, and with some minor alterations, all of the code samples provided in this chapter will work in that environment. Even if you work in a more recent version of .NET, but are unfamiliar with Functional Programming, I still recommend reading this chapter, as it should give you a decent starting point in programming with the Functional Paradigm.

Those of you familiar already with Functional code, and just want to see what’s available in the latest versions of .NET, it might be best to skip ahead to the next chapter.

# Getting Started

Functional Programming is easy, really it is! Despite what many people think, it’s easier to learn than Object-Oriented progrmanning. There are fewer concepts to learn, and actually less to get your head around.

If you don’t believe me, try explaining Polymorphism to a non-technical member of your family! Those of us that are comfortable with Object Orientation have often been doing it so long that we’ve forgotten how hard it may have been to get our heads around it at the beginning.

Functional programming isn’t hard to understand at all, just different. I’ve spoken to plenty of students coming out of university that embrace it with enthusiasm. So, if *they* can manage it…​

The myth does seem to persist though, that to get into Functional Programming, there’s a whole load of stuff that needs learning first. What if I told you though, that if you’ve been doing C# for any length of time, you’ve already most likely been writing Functional code for a while? Let me show you what I mean…​

# Your First Functional Code

Before we start with some functional code, let’s look at a bit of non-functional. A style you most likely learned somewhere very near the beginning of your C# career.

## A Non-Functional Film Query

In my quick, made-up example, I’m getting a list of all films from my imaginary data store and creating a new list, copied from the first, but only those items in the Action genre^([1](ch02.html#idm45400885975216))

[PRE0]

What’s wrong with this code? At the very least, it’s not very elegant. That’s a lot we’ve written to do something fairly simple.

We’ve also instantiated a new object that’s going to stay in scope for as long as this function is running. If there’s nothing more to the whole function than this, then there’s not much to worry about. But, what if this were just a short excerpt from a very long function? In that instance, the allFilms and actionFilms variables would both remain in scope, and thus in memory all that time, even if they aren’t in use.

There may not necessarily be copies of all of the data held within the item that’s being replicated, depending on whether it’s a class, a struct or whatever else. At the very least though, there’s a duplicate set of references being held unnecessarily in memory for as long as both items are in scope. That’s still more memory than we strictly need to hold.

We’re also forcing the order of operations. We’ve specified when to loop, when to add, etc. Both where and when each step should be carried out. If there were any intermediate steps in the data transformations to be carried out, we’d be specifying them too, and holding them in yet more potentially long-life variables.

I could solve a few problems with a `yield` return like this:

[PRE1]

This hasn’t done more than shave a few lines off, however.

What if there were a more optimal order of operations than the one we’ve decided on? What if a later bit of code actually meant that we don’t end up returning the contents of actionFilms? We’d have done the work unnecessarily.

This is the eternal problem of procedural code. Everything has to be spelled out. One of our major aims with Functional Programming is to move away from all that. Stop being so specific about every little thing. Relax a little, and embrace declaritive code.

## A Functional Film Query

So, what would that code sample above look like written in a Functional style? I’d hope many of you might already guess at how you would re-write it.

[PRE2]

If anyone at this point is saying “isn’t that just LINQ?”, then yes. Yes, it is. I’ll let you all in on a little secret - LINQ follows the Functional paradigm.

Just quickly, for anyone that’s not yet familiar with the awesomeness of LINQ. It’s a library that’s been part of C# since the early days, and provides a rich set of functions for filtering, altering and extending collections of data. Functions like `Select`, `Where` and `All` are from LINQ and commonly used around the world.

Think back for a moment to the list of features of Functional Programming, and see how many LINQ implements…​

*   Higher-order Functions - The lambda expressions passed to LINQ functions are all functions, being passed in as parameter variables.

*   Immutability - LINQ doesn’t change the source array, it returns a new `Enumerable` based on the old one.

*   Expressions instead of Statements - We’ve eliminated the use of a `ForEach` and an `If`

*   Referential Transparency - The Lambda Expression I’ve written here does actually conform to Referential Transparency (I.e. “no side effects”), though there’s nothing enforcing that. I could easily have referenced a string variable outside the Lambda. By requiring that the source data be passed in as a parameter, I’m also making it easier to test without requiring the creation & setup of a Mock of some kind to represent the data store connection. Everything the function needs is provided by its own parameters.

The iteration could well be done by recursion too, for all I know, but I have no idea what the source code of the Where function looks like. In the absence of evidence to the contrary, I’m just going to go on believing that it does.

This tiny little one-line code sample is a perfect example of the Functional approach in many ways. We’re passing around functions to perform operations against a collection of data, creating a new collection based on the old one.

What we’ve ended up with by following the Functional paradigm is something more concise, easier to read and therefore far easier to maintain.

# Results-Oriented Programming

A common feature of Functional code is that it focuses much more heavily on the end result, rather than on the process of getting there. An entirely Procedural method of building a complex object would be to instantiate it empty at the beginning of the code block, then fill in each property as we go along.

Something like this:

[PRE3]

The problem with this approach is that it’s very open to abuse. This silly little imaginary codeblock I’ve created here is short and easy to maintain. What often happens with production code however, is that the code can end up becoming incredibly long, with multiple data sources that all have to be pre-processed, joined, re-processed, etc. You can end up with long blocks of If-statements nested in If-statements, to the point that the code starts resembling the shape of a Family Tree.

For each nested If-statement, the complexity effectively doubles. This is especially true if there are multiple return statements scattered around the codebase. The risk increases of inadvertently ending up with a Null or some other unexpected value if the increasingtly complex codebase isn’t thought through in detail. Functional Programming discourages structures like this, and isn’t prone to this level of complexity, or of the potential unexpected consequences.

In our code sample above, we have PropertyC and PropertyD defined in 2 different places. It’s not too hard to work with here, but I’ve seen examples where the same property is defined in around half a dozen places across multiple classes and sub-classes^([2](ch02.html#idm45400886839824)).

I don’t know whether you’ve ever had to work with code like this? It’s happened to me an awful lot.

These sorts of large, unweildy codebases only ever get harder to work with over time. With each addition, the actual speed at which the developers can do the work goes down, and the business can end up getting frustrated because they don’t understand why their “simple” update is taking so long.

Functional code should ideally be written into small, concise blocks, focusing entirely on the end product. The expressions it prefers are modelled on mathematical working, so you really want to write it like small formulas, each precisely defining a value and all of the variables that make it up. There shouldn’t be any hunting up and down the codebase to work out where a value comes from.

Something like this:

[PRE4]

I know I’m now repeating the AlternateTuesday flag, but it means that all of the variables that determine a returned property are defined in a single place. It makes it much simpler to work with in the future.

In the event that a property is so complicated that it will either need multiple lines of code, or a series of Linq operations that takes up a lot of space, then I’d create a break-out function to contain that complex logic. I’d still have my central, result-based return at the heart of it all, though.

# A few words about Enumerables

I sometimes think Enumerables are one of the most under-used and least understood features of C#. An Enumerable is the most abstract representation of a collection of data - so abstract that it doesn’t contain any data itself, it’s actually just a description held in memory of how to go about getting the data. An Enumerable doesn’t even know how many items there are available until it iterates through everything - all it knows is where the current item is, and how to iterate to the next.

This is called *Lazy Evaluation* or *deferred Execution*. Being lazy is a good thing in development. Don’t let anyone tell you otherwise^([3](ch02.html#idm45400886206224)).

In fact, you can even write your own entire customised behaviour for an Enumerable if you wanted. Under the surface, there’s an object called an Enumerator. Interacting with that can be used to either get the current item, or iterate on to the next. You can’t use it to determine the length of the list, and the iteration only works in a single direction.

Have a look at this code sample:

First a set of simple logging functions that pop a message in a List of strings:

[PRE5]

Then a bit of code that calls each of those “DoSomething” functions in turn with different data.

[PRE6]

What do you think the order of operations is? You might think that the runtime would take the original input array, apply DoSomethingOne to all 3 elements to create a second array, then again with all three elements into DoSomethingTwo, and so on.

If I were to examine the content of that List of strings, I’d find something like this:

[PRE7]

It’s almost the exact same as you might get if you were running this through a `For`/`ForEach` loop, but we’ve effectively handed over control of the order of operations to the runtime. We’re not concerned with the nitty-gritty of temporary holding variables, what goes where and when. Instead we’re just describing the operations we want, and expecting a single answer back at the end.

It might not always look exactly like that, it depends on what the code that calls it looks like. But the intent always remains, that Enumerables only actually produce their data at the precise moment it’s needed. It doesn’t matter where they’re defined, it’s when they’re *used* that makes a difference.

Using Enumerables instead of solid arrays, we’ve actually managed to implement some of the behaviors we need to write Declarative code.

Incredibly, the log file I wrote above would still look the same if I were to re-write the code like this:

[PRE8]

temp1, temp2 and finalAnswer are all Enumerables, and none of them will contain any data until iterated.

Here’s an experiment for you to try. Write some code like this sample. Don’t copy it exactly, maybe something simpler like a series of selects amending an integer value somehow. Put a break point in and move the operation pointer on until final answer has been passed, then hover over finalAnswer in Visual Studio. What you’ll most likely find is that it can’t display any data to you, even though the line has been passed. That’s beause it hasn’t actually performed any of the operations yet.

Things would change if I did something like this:

[PRE9]

Because I’m specifically now calling `ToArray()` to force an enumeration of each intermediate step, then we really will call DoSomethingOne for each item in input before moving onto the next stop.

The log file would look something like this now:

[PRE10]

For this reason, I nearly always advocate for waiting as long as possible before using `ToArray()` or `ToList()` ^([4](ch02.html#idm45400883250816)), because this way we can leave the operations unperformed for as long as possible. And potentially even never performed if later logic prevents the enumeration from occurring at all.

There are some exceptions. Either for performance, or for avoiding multiple iterations. While the Enumerable remains un-enumerated it doesn’t have any data, but the operation itself remains in memory. If you pile too many of them on top of each other - especially if you start performing recursive operations, then you might find that you fill up far too much memory and performance takes a hit, and possibly even end up with a stack overflow.

# Prefer Expressions to Statements

In the rest of this chapter, I’m going to give more examples of how Linq can be used more effectively to avoid the need to use statements like If, Where, For, etc. or to mutate state (i.e. change the value of a variable).

There will be cases that aren’t possible, or aren’t ideal. But, that’s what the rest of this book is for.

## The Humble Select

If you’ve read this far in the book, you’re most likely aware of Select statements, and how to use them. There are a few features though, that most people I speak to don’t seem to be aware of, and they’re all things that can be used to make our code a little more functional.

The first thing was something I’ve already shown in the previous section - you can chain them. Either as a series of Select function calls - literally one after the other, or in a single code line; or else you can store the results of each Select in a different local variable. Functionally these two approaches are identical. It doesn’t even matter if you call ToArray after each one. So long as you don’t modify any resulting arrays or the object contained within them, you’re following the Functional paradigm.

The important thing is to get away from is the Imperative practice of defining a List, looping through the source objects with a ForEach and then adding each new item to the List. This is long-winded, harder to read, and honestly quite tedious. Why do things the hard way? Just use a nice, simple Select statement.

### Passing working values via tuples

Tuples were introduced in C#7\. Nuget packages do exist to allow some of the older versions of C# to use them too. They’re basically a way to throw together a quick-and-dirty collection of properties, without having to create and maintain a class.

If you’ve got a few properties you want to hold onto for a minute in one place, then dispose of immediately, Tuples are great for that.

If you have multiple objects you want to pass between Selects, or multiple items you want to pass in or out of one, then you can use a Tuple.

[PRE11]

In my example, above, I use a Tuple to pair up data from two look-up functions for each given film Id, meaning I can run a subsequent Select to simplify the pair of objects into a single return value.

### Iterator value is required

Som what if you’re Select-ing an Enumerable into a new form, and you need the iterator as part of the transformation? Something like this:

[PRE12]

We can use a feature of Select statements that surprisingly few people know about - that it has an override that allows us access to the iterator as part of the Select. All you have to do is provide a Lambda expression with 2 parameters, the second being an integer which represents the index position of the current item.

This is how our functional version of the code looks:

[PRE13]

Using these techniques, there’s nearly no circumstance that could exist where you need to use `ForEach` loop with a List. Thanks to C#’s support for the Functional paradigm, there are nearly always Declaritive methods available to solve problems.

The two different methods of getting the “i” index position variable are a great example of Imperative vs Delarative code. The Imperative, Object-Oriented method has the developer manually creating a variable to hold the value of i, and also explicitly set the place for the variable to be incremented. The declaritive code isn’t concerned with where the variable is defined, or in how each index value is determined.

N.b - Notice that I used `string.Join` to link the strings together. This is not only another one of those hidden gems of the C# language, but it’s also an example of Aggregation, that is - converting a list of things into a single thing. That’s what we’ll walk through in the next few sections.

### No Starting Array

The last trick for getting the value of i for each iteration is great if there’s an array - or collection of some other kind - available in the first place. What if there isn’t an array? What if you need to arbitrarily iterate for a set number of times?

These are the situations - somewhat rare - where you need a good, old-fashioned `For` loop instead of a `ForEach`. How do you create an array from nothing?

Your two best friends in this case are two static methods - `Enumerable.Range` and `Enumerable.Repeat`.

Range creates an array from a starting integer value, and requires you to tell it how many elements the array you want should have. It then creates an array of integers based on those specifications.

For example:

[PRE14]

Having whipped up an array, we can then apply LINQ operations to get our final result. Let’s imagine I was preparing a description of the 9 times table for one of my daughters^([5](ch02.html#idm45400882823232)).

[PRE15]

Here’s another example for you, what if I wanted to get all of the values from a grid of some kind, where an x and y value are required to get each value. I’ll imagine there’s a grid repository of some kind that I can use to get values.

Imagining that the grid is a 5x5, this is how I’d get every value:

[PRE16]

The first line here is generating an array of integers with the values `[1, 2, 3, 4, 5]`. I then use another `Select` to convert each of these integers into another array using another call to `Enumerable.Range`. This meant I now have an array of 5 elements, each of which was iteself an array of 5 integers. Using a Select on that nested array, I converted each of those sub-elements into a tuple which took one value from the parent array (x) and one from the sub-array (y). SelectMany is used to flatten the whole thing out to a simple list of all of the possible coordinates, which would look something like this: `(1, 1), (1, 2), (1, 3), (1, 4), (1, 5), (2, 1), (2, 2)`…​and so on.

Values can be obtained by Selecting this array of coordinates into a set of calls to the repository’s GetVal function, passing in the values of X and Y from the tuple of coordinates I created on the previous line.

Another situation we might be in is needing the same starting value in each case, but needing to transform it in different ways, depending on the position within the array. This is where `Enumerable.Repeat` comes in.

`Enumerable.Repeat` creates where each value is exactly the same, and you can specify exactly how many repeat elements you want.

You can’t use `Enumerable.Range` to count backwards. What if we wanted to do the previous example, but start at (5,5) and move backwards to (1,1). Here’s an example of how you’d do it:

[PRE17]

This looks a lot more complicated, but it isn’t really. What I’ve done is swap out the `Enumerable.Range` call for a 2-step operation.

First a call to `Enumerable.Repeat` which is repeating the integer value of 5 - 5 times. This results in an array like this: `[5, 5, 5, 5, 5]`.

Having done that, I’m then selecting using the overloaded version of `Select` which includes the value of i, then deducting that i value from the current value in the array. This means that in the first iteration, the return value is the current value i the array (5) minus the value of i (0 for the first iteration), this gives simply 5 back. In the next iteration, the value of i is 1, so 5-1 means 4 is returned. And so on.

At the end of it we get back an array that looks something like this: `(5, 5), (5, 4), (5, 3), (5, 2), (5, 1), (4, 5), (4, 4)` …​etc.

There are ways to take this further still, but for this chapter I’m sticking to the relatively simple cases, ones that don’t require hacking around with C#. This is all out-of-the-box functionality that anyone can use right away.

## Many to One - The subtle art of Aggregation

We’ve looked at loops for converting one thing into another, X items in → X new items out. That sort’ve thing. There’s another use case for loops that I’d like to cover - reducing many items into a single value.

This could be making a total count, calculating Averages, Means or other statistical data, or other more complex aggregations.

In Procedural code, we’d have a loop, a state tracking value and inside the loop we’d update the state constantly, based on each item from our array. Here’s a very simple example of what I’m talking about:

[PRE18]

There’s actually an in-built Linq method for doing this:

[PRE19]

There really shouldn’t ever be a need to do this sort of operation “long-hand”. Even if we’re creating the sum of a particular property from an array of Objects, Linq still has us covered:

[PRE20]

There’s another function for calculating Means in the same manner called Average. There’s nothing for calculating Median, so far as I’m aware.

I could calculate the Median with a quick bit of functional style code, however. It would look like this:

[PRE21]

There are more complex aggregations that are required sometimes. What if we wanted - for example - a sum of two different values from an Enumerable of complex objects?

Procedural code might look like this:

[PRE22]

We could use two separate Sum function calls, but then we’d be iterating twice through the Enumerable, hardly an efficient way to get our information. Instead, we can use another strangely little-known feature of Linq - the aggregate function. This consists of the following components:

*   Seed - a starting value for the final value.

*   An Aggregator function, this has two parameters - the current item from the Enumerable we’re aggregating down, and the current running total.

The seed doesn’t have to be a primitive type, like an integer or whatever, it can just as easily be a complex object. In order to re-write the code sample, above, in a Functional style, however, we just need a simple Tuple.

[PRE23]

In the right place, Aggregate is an incredibly powerful feature of C#, and one worth taking the time to explore and understand properly.

It’s also an example of another concept important to Functional Programming - recursion.

## Customised Iteration Behavior

Recursion sits at the back of a lot of Functional versions of Iteration. For the benefit of anyone that doesn’t know, it’s a function that calls itself repeatedly until some condition or other is met.

It’s a very powerful technique, but has some limitations to bear in mind in C#. The most important two being:

*   If developed improperly, it can lead to infinite loops, which will literally run until the user terminates the application, or all available space on the stack is consumed. As Treguard, the legendary Dungeon Master of the popular British Fantasy RPG gameshow *Knightmare* would put it: “Oooh, Nasty”^([6](ch02.html#idm45400882090672)).

*   In C# they tend to be consume a lot of memory compared to other forms of iteration. There are ways around this, but that’s a topic for another chapter.

I have a lot more to say about recursion, and we’ll get to that shortly, but this for the purposes of this chapter, I’ll give the simplest example I can think of.

Let’s say that you want to iterate through an Enumerable but you don’t know how long for. Let’s say you have a list of delta values for an integer (i.e. the amount to add or subtract each time) and you want to find out how many steps it is until you get from the starting value (whatever that might be) to 0.

You could quite easily get the final value with an Aggregate call, but we don’t want the final value. We’re interested in all of the intermediate values, and we want to stop prematurely through the iteration. This is a simple arithmetic operation, but if complex objects were involved in a real-world scenario, there might be a significant performance saving from the ability to terminate the process early.

In Procedural code, you’d probably write something like this:

[PRE24]

In this example I’m returning -1 to say that the starting value is already the one we’re looking for, otherwise I’m returning the zero-based index of the array that resulted in 0 being reached.

This is how I’d do it recursively:

[PRE25]

This is Functional now, but it’s not really ideal. Nested functions have their place, but I don’t personally find the way it has to be used here as readable as the code could be. Delightfully recursive, but I think it could be made clearer.

The other major problem is that this won’t scale up well if the list of deltas is large. I’ll show you what I mean.

Let’s imagine there are only 3 values for the Deltas: 2, -12 & 9\. In this case we’d expect our answer to come back as 1, because the second position (i.e. index=1) of the array resulted in a zero (10+2-12). We would also expect that the 9 will never be evaluated. That’s the efficiency saving we’re looking for from our code here.

What was actually happening with the recursive code, though.

First, it called GetFirstPositionWithValueZero with a current value of 10 (i.e. the starting value) and i was allowed to be the default of -1.

The body of the function is a ternary if statement. If zero has been reached, return i, otherwise call the function again but with updated values for current and i.

This is what’ll happen with the first delta (i.e. i=0, i.e. 2), so GetFirstPositionWithValueZero is called again with the current value now updated to 12 and i as 0.

The new value is not 0, so the second call to GetFirstPositionWithValueZero will call itself again, this time with the current value updated with delta[1] and i incremented to 1\. delta[1] is -12, which would mean the third call results in a 0, which means that i can simply be returned.

Here’s the problem though…​

The third call got an answer, but the first two calls are still open in memory and stored on the stack. The third call returns 1, which is passed up a level to the second call to GetFirstPositionWithValueZero, which now also returns 1, and so on…​ Until finally the original first call to GetFirstPositionWithValueZero returns the 1.

If you want to see that a little graphically, imagine it looking something like this:

[PRE26]

That’s fine with 3 items in our array, but what if there are hundreds!

Recursion, as I’ve said, is a powerful tool, but it comes with a cost in C#. Purer Functional languages (including F#) have a feature called *Tail Call Optimised Recursion* which allows the use of recursion without this memory usage problem.

Tail Recursion is an important concept, and one I’m going to return to later in a whole chapter dedicated to it, so I’m not going to dwell on it in any further detail here.

As it stands, out-of-the-box C# doesn’t permit Tail Recursion, even though it’s available in the .NET Common Language Runtime (CLR). There are a few tricks we can try to make it available to us, but they’re a little too complex for this chapter, so I’ll talk about them at a later juncture.

For now, consider recursion as it’s described here, and keep in mind that you might want to be careful where and when you use it.

# Immutability

There’s more to Functional Programming in C# than just Linq. Another important feature I’d like to discuss is Immutability (i.e. a variable may not change value once declared). To what extent is it possible in C#?

Firstly, there are some newer developments with regards to Immutability in C# 8 and upwards. See the next chapter for that. For this chapter, I’m restricting myself to what is true of just about any version of .NET.

To begin, let’s consider this little C# snippet:

[PRE27]

Is this immutable? It very much is not. Any of those properties can be replaced with new values via the setter. The IList also provides a set of functions that allows its underlying array to be added to or removed from.

We could make the setters private, meaning we’d have to instantiate the class via a detailed contructor:

[PRE28]

Is it immutable now? No, honestly it’s not. It’s true that you can’t outright replace any of the properties with new objects outside ClassA, which is great. The properties can be replaced inside the class, but the developer can ensure that no such code is ever added. You should hopefully have some sort of code review system to ensure that, as well.

PropA and PropC are fine - strings and DateTime are both immutable in C#. The int value of PropB is fine too - ints don’t have anything you can change except its value.

There are still several problems, however.

PropE is a List, which can still have values added, removed and replaced, even though we can’t replace the entire object. If we didn’t actually need to hold a mutable copy of PropE, we could easily replace it with an IEnumerable or IReadOnlyList.

The `IEnumerable<double>` value of PropD seems fine at first glance, but what if it was passed to the constructor as a `List<double` which is still referenced by that type in the outside world? It would still be possible to alter its contents that way.

There’s also the possibility of introducing something like this:

[PRE29]

All properties of PropF are also potentially going to be mutable - unless this same structure with private setters is followed there too.

What about classes from outside your codebase? What about Microsoft classes, or those from a 3rd party Nuget package? There’s no way to enforce immutability.

Unfortunately there simply isn’t any way to enforce universal immutability, not even in the most recent versions of C#. I would assume that for backwards compatibility reasons, there is never going to be.

It would be lovely to have a native C# method of ensuring immutability by default, but there isn’t one - and isn’t ever likely to be for reasons of backwards compatibility. My own solution is that when coding, I simply *pretend* that Immutability exists in the project, and never change any object. There’s nothing in C# that provides any level of enforcement whatsoever, so you’d simply have to make a decision for yourself, or within your team, to act as if it does.

# Putting it all Together - a Complete Functional Flow

I’ve talked a lot about some simple techniques you can use to make your code more functional right away. Now, I’d like to show a complete, if minute, application written to demonstrate an end-to-end functional process.

I’m going to write a very simple CSV parser. In my example, I want to read in the complete text of a CSV file containing data about the first few series of Doctor Who^([7](ch02.html#idm45400881422624)). I want to read the data, parse it into a Plain Old C# Object (POCO, i.e. a class containing only data and no logic) and then aggregate it into a report which counts the number of episodes, and the number of episodes known to be lost for each season. ^([8](ch02.html#idm45400881421904)). I’m simplifying CSV parsing for the purposes of this example. I’m not worrying about quotes around string fields, commas in field values or any values requiring additional parsing. There are 3rd party libraries for all of that! I’m just proving a point.

This complete process represents a nice, typical functional flow. Take a single item, break it up into a list, apply list operations, then aggregate back down into a single value again.

This is the structure of my CSV file:

*   [0] - Season Number. Integer value between 1 and 39\. I’m running the risk of dating this book now, but there are 39 seasons to date.

*   [1] - Story Name - a string field I don’t care about

*   [2] - Writer - ditto

*   [3] - Director - ditto

*   [4] - Number of Episodes - in Doctor Who, all stories comprise between 1 and 14 episodes. Until 1989, all stories were multi-part serials.

*   [5] - Number of Missing Episodes - the number of episodes of this serial not known to exist. Any non-zero number is too many for me, but such is life.

I want to end up with a report that has just these fields:

*   Season Number

*   Total Episodes

*   Total Missing Episodes

*   Percentage Missing

Let’s crack on with some code…​.

[PRE30]

In case you’re curious, the results would look something like this (the “\t” characters are tabs, which make it a bit more readable):

[PRE31]

Note, I could have made the code sample more concise and written just about all of this together in one long, continuous fluent expression like this:

[PRE32]

There’s nothing wrong with that sort of approach, but I like splitting it out into individual lines for a couple of reasons:

*   The variable names provide some insight into what your code is doing. We’re sort’ve semi-enforcing a form of code commenting.

*   It’s possible to inspect the intermediate variables, to see what’s in them at each step. This makes debugging easier, because like I said in the previous chapter - it’s like being able to look back on your working in a mathematics answer, to see which step it was that you went wrong on.

There isn’t any ultimate functional difference, nothing that would be noticed by the end user, so which style you adopt is more a matter of personal taste. Write in whatever way it seems best to you. Do try and keep it readable, and easy for everyone to follow, though.

# Taking it Further - Develop Your Functional Skills

Here’s a challenge for you. If some or all of the techniques described to you here were new, then go off and have fun with them for a bit.

Challenge yourself to writing code with the following rules:

*   Treat all variables as immutable - do not change any variable value once set. Basically treat everything as if it were a constant.

*   None of the following statements are permitted - `If`, `For`, `ForEach`, `While`. `If` is acceptable only in a Ternary expression - i.e. the single-line expression in the style: someBoolean ? valueOne : valueTwo.

*   Where possible write as many functions as small, concise arrow functions (a.k.a. Lambda Expressions).

Either do this as part of your production code, or else go out and look for a code challenge site, something like [The Advent of Code (https://adventofcode.com)](https://adventofcode.com/) or [Project Euler (https://projecteuler.net)](https://projecteuler.net/). Something you can get your teeth into.

If you don’t want to go through the bother of creating an entire solution for these exercises in Visual Studio, there’s always LINQPad ([*https://www.linqpad.net/*](https://www.linqpad.net/)) for a quick and easy way to rattle off some C# code.

After you’ve got the hang of this, you’ll be ready to move onto the next step. I hope you’re having fun so far!

# Summary

In this chapter, we looked at a variety of simple Linq-based techniques for writing Functional-style code immediately in any C# codebase using at least .NET Framework 3.5, because these features are ever-green and have been in place for all of those years in every subsequent version of .NET without needing to be updated or replaced.

We discussed the more advanced features of Select statements, some of the less well-known features of Linq and methods for Aggregating and Recursion.

In the next chapter, I’ll look at some of the most recent developments in C# that can be used in more up-to-date codebases.

^([1](ch02.html#idm45400885975216-marker)) I’m more of an SF (or Sci-fi, if you prefer) fan, truth be told.

^([2](ch02.html#idm45400886839824-marker)) And in one example, a couple of definitions were also outside the codebase in Database Stored Procedures

^([3](ch02.html#idm45400886206224-marker)) Except your employer. They pay your bills. They hopefully send you birthday cards once a year too, if they’re nice.

^([4](ch02.html#idm45400883250816-marker)) As a Functional programmer, and a believer in exposing the most abstract interface possible, I literally **never** use `ToList()`. Only ever `ToArray()`, even if `ToList` is every so slightly faster.

^([5](ch02.html#idm45400882823232-marker)) No, Sophie. It’s not good enough to just use your fingers!

^([6](ch02.html#idm45400882090672-marker)) Check out the man himself [here (https://www.youtube.com/watch?v=OISR3al5Bnk)](https://www.youtube.com/watch?v=OISR3al5Bnk)

^([7](ch02.html#idm45400881422624-marker)) For those of you unacquainted, this is a British SF series that has been running on-and-off since 1963\. It is, in my own opinion, the greatest TV series ever made. I’m not taking any arguments on that

^([8](ch02.html#idm45400881421904-marker)) Sad to say, the BBC junked many episodes of the series in the 1970s. If you have any of those, please do hand them back.