# Chapter 4\. Work Smart, Not Hard with Functional Code

Everything I’ve covered so far has been functional programming as intended by Microsoft’s C# team. You’ll find these features mentioned, along with examples on the Microsoft website. In this chapter however, I want to start being a bit more creative with C#.

I don’t know about you, but I like being lazy, or at least I don’t like wasting my time with tedious boilerplate code. One of the many wonderful things about functional programming is how concise it is, compared to Imperative code.

In this chapter I’m going to show you ways to push the Functional envelope further than out-of-the-box C# will allow, as well as ways to implement some of the more recent versions of C# in legacy versions, and hopefully allow you to get on with your day job an awful lot quicker.

There are broadly three categories which this chapter will explore:

Funcs in Enumerables

`Func` delegates don’t seem to get used all that much, but they’re incredibly powerful features of C#. I’ll show a few ways of using them that help extend C#’s capabilities. In this case by adding them to Enumerables and operating on them with Linq expressions.

Funcs as Filters

You can also use Funcs as filters - something that sits between you and the real value you’re trying to reach. There are a few neat bits of code you can write using these principles.

Custom Enumerables

I’ve discussed IEnumerables and how cool they are before, but did you know you can break them open and implement your own customised behavior? I’ll show you how.

# It’s Time to get Func-y

I’ve already talked about `Func` delegate types the introduction, but just to recap - they’re functions stored as variables. You define what parameters they take, what they return and call them like any other function. Here’s a quick example:

[PRE0]

The last generic type in the list between the two chevrons is the return value, all of the previous types are the parameters. My example above takes 2 string parameters and returns a string.

We’re going to be seeing an awful lot of Func delegates from now on, so please do make sure you’re comfortable with them before reading on.

## Funcs in Enumerables

I’ve seen plenty of examples of Funcs as parameters to functions, but I’m not sure many developers realise you can put them in Enumerable, and create some interesting behaviors.

First, is the obvious one - put them in an array to act on the same data mulitple times:

[PRE1]

Using this method, we can have a single original source of data (here, an Employee object) and have multiple records of the same type generated from it, and in my case I aggreated using the built-in .NET method `string.Join` to present a single, unified string to present to the end user.

There are a few advantages of this method over a simple StringBuilder.

Firstly, the array can be assembled dynamically. There could be multiple rules for each property and how it’s rendered, which could be selected from a set of local variables depending on some custom logic.

Secondly, this is an Enumerable, so by defining it this way, we’re taking advantage of a feature of Enumerables called *Lazy Evaluation*. The thing about Enumerables is that they aren’t arrays, they aren’t even data. They’re just pointers to something that will tell you how to extract the data. It might well be - and in fact usually is the case - that the source at the back of the Enumerable is a simple array, but not necessarily. An Enumerable requires a function to be executed each time the next item is accessed via a `ForEach` loop. Enumerables were developed so that it only transforms into actual data at the very last possible moment - typically when starting a `ForEach` loop iteration. Most of the time this doesn’t matter if there’s an array held in-memory somewhere feeding the Enumerable, but if there’s an expensive function or look-up to an external system powering it, then Lazy Loading can be incredibly useful to prevent unnecessary work.

The elements of an Enumerable are evaluated one at a time and only when their turn has come to be used by whatever process is performing the enumeration. For example, if we use the LINQ `Any` function to evaluate each element in an Enumerable, it will stop enumerating the first time that an element is found that matches the specified criteria, meaning the remaining elements will be left un-evaluated.

Lastly, from a maintanance perspective, this technique is easier to live with. Adding a new line to the final result is as easy as adding a new element to the array. It also acts as a restraint to future programmers, making it harder for them to try to put too much complex logic where it doesn’t belong.

## A Super-Simple Validator

Let’s imagine ourselves a quick validation function. They typically look something like this:

[PRE2]

Well, for a start, that’s a **lot** of code for what is, in fact, a fairly simple set of rules. There’s a whole heap of repetitive boilerplate code that Imperative code forces us to write here. On top of that, if we want to add in another rule, that’s potentially around 4 new lines of code to add when really only 1 is especially interesting to us.

If only there were a way to compact it into just a few simple lines of code…​

Well…​since you asked so nicely, here you go:

[PRE3]

Not so long now, is it? What is it I’ve done here? I’ve put all of the rules into an array of Funcs that turn a string into a bool - i.e. check a single validation rule. I’ve used a Linq statement - `.All()`. The purpose of this function is to evaluate whatever lambda expression I give it against all elements of the array it’s attached to. If a single one of these returns false, then the process is terminated early and false is returned from the `All` (As mentioned earlier, the subsequent values aren’t accesssed, so Lazy Evaluation saves us time by not evaluating them). If every single one of the items returns true, then the All also returns true.

We’ve effectively re-created the first code sample, but the boilerplate code we were forced to write - If statements and early returns - is now implicit in the structure.

This also has the advantage of once again being very easy to maintain, as a code structure. If you wanted, you could even generalize it into an extension method. Something I often do. Something like this:

[PRE4]

This reduces the size of the password validator yet further, and gives you a handy, generic structure to use elsewhere:

[PRE5]

At this point, I hope you’re re-considering ever writing something as long and ungainly as that first validation code sample ever again.

I think an `IsValid` check is easier to read and maintain, but if you wanted a piece of code that is much more in line with the original code sample, then a new extension method can be created using an Any instead of an All:

[PRE6]

This means that the boolean logic of each array element can be reversed, as they were originally:

[PRE7]

If you want to maintain both functions: `IsValid` and `IsInvalid` because each have their place in your codebase, it’s probably worth saving yourself some coding effort, and preventing a potential maintenance task in the future by simply referencing one in the other:

[PRE8]

Use it wisely, my young Functional Padawan Learner.

## Pattern Matching for Old Versions of C#

Pattern matching is one of the very best features of C# in recent years, along with record types, but it isn’t available in anything except the most recent .NET versions - See Chapter 3 for more details on native Pattern Matching in C# 7 and up.

Is there a way to allow pattern matching to happen, but without needing up upgrade to a newer version of C#?

There most certainly is. It is nowhere near as elegant as the native syntax in C# 8, but it provides a few of the same benefits.

In this example, I’ll calculate how much tax someone should pay based on a grossly simplified version of the UK Income Tax rules. Please note, these really are much simpler than the real thing. I don’t want to get too bogged down in the complexities of tax.

The rules I’m going to apply look like this:

*   Yearly income ⇐ £12,570, then no tax is taken

*   Yearly income is between £12,571 and £50,270, then take 20% tax

*   Yearly income is between £50,271 and £150,000, then take 40% tax

*   Yearly income is over £150,000, then take 45% tax

If you wanted to write this long-hand (i.e. non-functionally), it would look like this:

[PRE9]

Now, in C# 8 and onwards, switch expressions would compress this to a few lines. So long as you’re running at least C# 7 (i.e. .NET Framework 4.7) this is the style of pattern matching I can create:

[PRE10]

I’m passing in an array of tuples containing 2 lambda expressions. The first determines whether the input matches against the current pattern, the second is the transformation in value that occurs if the pattern is a match. There’s a final check to see whether the default pattern should be applied - i.e. because none of the other patterns were a match.

Despite being a fraction of the length of the original code sample, this contains all of the same functionality. My matching patterns on the left-hand side of the tuple are simple here, but they can contain expressions as complicated as you’d like, and could even be calls to whole functions containing detailed criteria to match on.

So, how did I make this work? This is an extremely simple version that provides most of the functionality required:

[PRE11]

I’m using the Linq method `FirstOrDefault` to first iterate through the left-hand functions to find one that returns true (i.e. one with the right criteria) then call the right-hand conversion Func to get my modified value.

This is fine, except that if *none* of the patterns match, we’ll be in a bit of a fix. There will most likely be a null reference exception.

To cover this, we need to force the need to provide a default match (the equivalent of a simple `else` statement, or the _ pattern match in switch expressions).

My answer is to have the `Match` function return a placeholder object that holds either a transformed value from the Match expressions, or else executes the Default pattern lambda expression. The improved version looks like this:

[PRE12]

This method is severely limited compared to what can be accomplished in the latest versions of C#. There is no object type matching, and the syntax isn’t as elegant, but it’s still usable and could save an awful lot of boilerplate, as well as encouraging good code standards.

Versions of C# that are older still, and which don’t include Tuples, can consider the use of `KeyValuePair<T,T>`, though the syntax is far from attractive.

What, you don’t want to take my word? Ok, here we go. Don’t say I didn’t warn you…​

The Extension method itself is just about the same, and just needs a small alteration to use `KeyValuePair` instead of Tuples:

[PRE13]

And here’s the ugly bit. The syntax for creating `KeyValuePair` objects is pretty awful:

[PRE14]

So you *can* still have a form of Pattern Matching in C# 4, but I’m not sure how much you’re gaining by doing it. That’s perhaps up to you to decide. At least I’ve shown you the way.

# Functional Filtering

Functions don’t only have to be used for turning one form of data into another, you can also use them as filters, extra layers, that sit between the developer and an original source of information or functionality.

This section looks at a few examples of how this method can be used to make your daily C# coding look simpler, and also be less error prone.

## Make Dictionaries More Useful

One of my absolute favorite things in C# by far is Dictionaries. Used appropriately they can reduce a heap of ugly, boilerplate-riddled code with a few simple elegant array-like lookups. They’re also very efficient to find data in, once created.

They have a problem, however, that makes it often necessary to add in a heap of boilerplate that invalidates the whole reason they’re so lovely to use. Consider the following code sample:

[PRE15]

What’s up with this code? It falls foul of a code feature of Dictionaries that I find inexplicable, if you try looking up an entry that doesn’t exist^([1](ch04.html#idm45400869322320)), it will trigger an Exception that has to be handled!

The only safe way to handle this is to use one of several methods available in C# to check against the available keys before compiling the string, like this:

[PRE16]

Alternatively, using slightly newer versions of C#, there is a `TryGetValue` function available as well to simplify this code a little:

[PRE17]

So, can we use functional programming techniques to reduce our boilerplate code, and give us all of the useful features of Dictionaries, but without the awful tendency to explode on us? You bet’cha!

First I need a quick extension method:

[PRE18]

I’ll explain further in a minute, but first, here’s how I’d use my extension methods:

[PRE19]

Notice the difference?

If you look carefully, I’m now using rounded function brackets, rather than square array/dictionary brackets to access values from the Dictionary. That’s because it’s technically not a dictionary any more! It’s a function.

If you look at my extension methods, they return functions, but they’re functions that keep the original Dictionary object in scope for as long as they exist. Basically, they’re like a filter layer sitting between the Dictionary and the rest of the codebase. The functions make a decision on whether use of the Dictionary is safe or not.

It means we can use a Dictionary, but the Exception that occurs when a key isn’t found will no longer be thrown, and we can either have the default for the type (usually Null) returned, or supply our own default value. Simples.

The only down side to this method is that it’s no longer a dictionary, in effect. This means you can’t modify it any further, or perform any LINQ operations on it. If you are a situation though, where you’re sure you won’t need to, then this is something you can use.

# Parsing Values

Another common cause of noisy, boilerplate code is parsing values from `string` to other forms. Something like this for parsing in a hypothetical settings object, in the event we were working in .NET Framework, and the appsettings.JSON and `IOption<T>` features aren’t available:

[PRE20]

That’s a lot of code to do something simple, isn’t it? There’s a lot of boilerplate code noise there that obscures the intention of the code to all but those familiar with these sorts of operations. Also, if a new setting were to be added, it would take 5 or 6 lines of new code for each and every one. That’s quite a waste.

Instead, we can do things a little more Functionally and hide the structure away somewhere, leaving just the intent of the code visible for us to see.

As usual, here’s an extension method to take care of business for me:

[PRE21]

This takes away all of the repetitive code from the first example, and allows us to move to a more readable, result-driven code sample, like this:

[PRE22]

It’s easy now to see at a glance what the code does, what the default values are, and how you’d add more settings with a single line of code.

Any other settings value types besides `int` and `string` would need another extension method creating, but that’s no great hardship.

# Custom Enumerations

Most of us have likely used Enumerables when coding, but did you know that there’s an engine under the surface that you can access and use to create all sorts of interesting custom behaviors?

With a custom iterator, we can drastically reduce the number of lines of code needed when more complicated behavior is needed when looping through data. First though, it’s necessary to understand just how an Enumerable works beneath the surface.

There’s a class sitting beneath the surface of the Enumerable, the engine which drives the Enumeration, which is what allows you to use a ForEach to loop through values. It’s called the Enumerator.

The Enumerator has basically two features:

*   Current: Get the current item out of the enumerable. This may be called as many times as you want, provided you don’t try moving to the next item. If you try getting the Current value before first calling MoveNext, an exception is thrown.

*   MoveNext: Move from the current item, and try to see whether there is another to be selected or not. Returns `True` if another value is found, `False` if we’ve reached the end of the Enumerable, or there were no elements in the first place. The first time this is called, it points the Enumerator at the first element in the Enumerable.

## Query Adjacent Elements

A relatively simple example to start with. Let’s imagine that I want to run through an Enumerable of Integers, to see whether it contains any numbers that are consecutive.

An Imperative solution would likely look something like this:

[PRE23]

To make this code functional, as is often the case, we need an extension method. Something that takes the Enumerable, extracts its Enumerator and controls the customised behavior.

To avoid use of an imperative-style loop, I’m using recursion here. In brief - recursion is a way of implementing an indefinite^([2](ch04.html#idm45400868337712)) loop by having a function call itself repeatedly.

I’ll revisit the concept of recursion in a later chapter. For now though, I’m just going to use the standard, simple version of recursion

[PRE24]

So, what’s happening here? It’s kind’ve like juggling, in a way. I start by extracting the Enumerator, and moving to the first item.

Inside the private function, I accept the enumerator (now pointing to the first item), the “are we done” evaluator function and a copy of that same first item.

What I do then is immediately move to the next item, and run the evaluator function, passing in the first item, and the new “current”, so they can be compared.

At this point either we find out we’ve run out of items or the evaluator returns true, in which case we can terminate the iteration. If MoveNext returned true then we check if the `previousValue` and `Current` match our requirement (as specified by `evaluator`). If they do then we finish and return true, otherwise we make a recursive call to check the rest of the values.

This is the updated version of the code to find consecutive numbers:

[PRE25]

It would also be easy enough to create an `All` method based on the same logic, like so:

[PRE26]

The only difference are the conditions for deciding whether or not to continue, and whether we need to return early. With an `All`, the point is to check every pair of values, and only return out of the loop early if one is found not to meet the criteria.

## Iterate Until a Condition is Met

This is basically a replacement for a while loop, so that there’s another statement we don’t necessarily need.

For my example, I want to imagine what the turn system might be like for a text-based adventure game. For younger readers - this is what we had in the old days, before graphics^([3](ch04.html#idm45400867718416)). You used to have to write what you wanted to do and the game would write what happened. Kind of like a book, except you wrote what happened yourself.

The basic structure of one of those games was something like:

*   Write a description of the current location

*   Receive user input

*   Execute requested command

Here’s how Imperative code might handle that situation:

[PRE27]

In principle what we want is a Linq-style Aggregate function, but one that doesn’t loop through all of the elements of an array, then finish. Instead we want it to loop continuously until our end condition (the player is dead) is met. I’m simplifying a little here, obviously our player in a proper game could *win* as well. But my example game is like life, and life’s not fair!

The extension method for this is another place where we’d benefit from tail recursion optimised calls, and I’ll be looking into options for that in a later chapter, but for now I’ll just use simple recursion - which may become an issue if there are a lot of turns, but for now it’d stop me from introducing too many ideas too soon.

[PRE28]

Using this, I can do away with the `while` loop entirely and transform the entire turn sequence into a single function, like so:

[PRE29]

This isn’t perfect, but it’s functional now. There are far better ways of handling the multiple steps of the update to the game’s state, and there’s the issue of how to handle user interaction in a functional manner, too. There’s a section coming up on that in chapter [X].

# Conclusion

In this chapter, we looked at ways to use Funcs, Enumerables and Extension methods to extend C# to make it easier to write Functional-style code, and to get around a few existing limitations of the language.

I’m certain that I’m barely scratching the surface with these techniques, and that there are plenty more out there to be discovered and used.

In our next chapter, we’ll be looking at Higher-order functions, and some structures that can be used to take advantage of them to create yet more useful functionality.

^([1](ch04.html#idm45400869322320-marker)) Incidentally, it was Peter Davison.

^([2](ch04.html#idm45400868337712-marker)) but hopefully not infinite!

^([3](ch04.html#idm45400867718416-marker)) go and check out the epic adventure game Zork if you’d like to see this for yourself. Try not to get eaten by a Grue!