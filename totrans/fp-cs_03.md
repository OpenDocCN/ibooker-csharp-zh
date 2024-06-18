# Chapter 3\. Functional Coding in C# 7 and Beyond

I’m not sure when exactly the decision was made to make C# a hybrid Object-Oriented/Functional language. The very first foundation work was laid in C# 3\. That was when features like Lambda Expressions and Anonymous Types were introduced, which later went on to form parts of Linq in .NET 3.5.

After that though, there wasn’t much new in terms of Functional features for quite some time. In fact, it wasn’t really until the release on C# 7 in 2017 that Functional Programming seemed to become relevant again to the C# team.

From C# 7 onwards, every version of C# has contained something new and exciting to do more Functional style coding, a trend that doesn’t currently show any signs of stopping!

In the last chapter, we looked at Functional features that could be implemented in just about any C# codebase likely to still be in use out in the wild. In this chapter, we’re going to throw away that assumption and look at all the features you can make use of if your codebase is allowed to use any of the very latest features - or at least those released since C# 7.

# Tuples

Tuples were introduced in C#7\. Nuget packages do exist to allow some of the older versions of C# to use them too. They’re basically a way to throw together a quick-and-dirty collection of properties, without having to create and maintain a class.

If you’ve got a few properties you want to hold onto for a minute in one place, then dispose of immediately, Tuples are great for that.

If you have multiple objects you want to pass between Selects, or multiple items you want to pass in or out of one, then you can use a Tuple.

This is an example of the sort of thing you might consider using Tuples for:

[PRE0]

In my example, above, I use a Tuple to pair up data from two look-up functions for each given film Id, meaning I can run a subsequent Select to simplify the pair of objects into a single return value.

# Pattern Matching

Switch statements have been around for longer than just about any developers still working today. They have their uses, but they’re quite limited in what can be done with them. Functional Programming has taken that concept and moved it up a few levels. That’s what Pattern Matching is.

It was C# 7 that started to introduce this feature to the C# language, and it has been subsequently enhanced multiple times in later versions, and most likely there will be yet more features added in the future.

Pattern matching is an amazing way to save yourself an awful lot of work. To show you what I mean, I’ll now show you a bit of Procedural code, and then how pattern matching is implemented in a few different versions of C#.

## Procedural Bank Accounts

For our example, let’s imagine one of the classic Object-Oriented worked examples - Bank Accounts. I’m going to create a set of bank account types, each with different rules for how to calculate the amount of interest. These aren’t really based on real banking, they’re straight out of my imagination.

These are my rules:

*   A standard bank account calculates interest by multiplying the balance by the interest Rate for the account

*   A premium bank account with a balance of 10,000 or less is a standard Bank account

*   A premium bank account with a balance over 10,000 applies an interest rate augmented by a bonus additional rate

*   A Millionaire’s bank account, who owns so much money it’s larger than the largest value a decimal can hold (It’s a really, really big number - around 8*10^28, so they must be very wealthy indeed. Do you think they’d be willing to lend me a little, if I were to ask? I could do with a new pair of shoes). They have an overflow balance property to add in all that money they own that is over the max decimal value, that they can’t store in the standard balance property like us plebs. They need the interest to be calculated based on both balances.

*   A Monopoly player’s bank account. They get an extra 200 for passing Go. I’m not implementing the “Go Direct to Jail” Logic, there are only so many hours in a day.

These are my classes:

[PRE1]

The procedural approach to implementing the CalculateInterest feature for bank accounts - or as I think of it the “long-hand” approach, could possibly look like this:

[PRE2]

As is typical with Procedural code, the above code isn’t very concise, and might take a little bit of reading to understand its intent. It’s also wide open to abuse if many more new rules are added once the system goes into production.

The Object-Oriented approach would be either to use an interface, or polymorphism - i.e. create an abstract base class with a virtual method for the CalculateNewBalance function. The issue with that is that the logic is now split over many places, rather than being contained in a single, easy-to-read function.

In the sections that follow, I’ll show how each subsequent version of C# handled this problem.

## Pattern Matching in C# 7

C# 7 gave us two different ways of solving this problem. The first was the new `is` operator - a much more convenient way of checking types than had previously been available. An `is` operator can also be used to automatically cast the source variable to the correct type.

Our updated source would look something like this:

[PRE3]

Note in the code sample above, that with the `is` operator, we can also automatically wrap the source variable into a new local variable of the correct type.

This isn’t bad, it’s a little more elegant, and we’ve saved ourselves a few redundant lines, but we could do better, and that’s where another feature of C# 7 comes in - type switching.

[PRE4]

Pretty cool, right? Pattern Matching seems to be one of the most developed features of C# in recent years. As I’m about to show, every major version of C# since this has continued to add to it.

# Pattern Matching in C# 8

Things moved up a notch in C# 8, pretty much the same concept, but with a new, updated matching syntax that more closely matches JSON, or a C# object initializer expression. Any number of clauses to properties or sub-properties of the object under examination can be put inside the curly braces, and the default case is now represented by the *_* discard character.

[PRE5]

Also, switch can now **also** be an expression, which you can use as the body of a small, single-purpose function with surprisingly rich functionality. This means it can also be stored in a Func delegate for potential passing around as a Higher-Order function.

This is an example using an old childhood game: Scissor, Paper Stone. Known in the US as Rock, Paper, Scissors and in Japan as Janken. I’ve created a `Func` delegate in the following example with the following rules:

1.  Both players drawing the same = draw

2.  Scissors beats paper

3.  Paper beats stone

4.  Stone beats scissors

This function is specifically determining what the result is from *my* perspective against my imaginary adversary.

[PRE6]

Having stored it in a ‘Func<SPS,SPS>’ typed variable, I can pass it around to wherever needs it.

This can be as a parameter to a function, so that the functionality can be injected at run-time:

[PRE7]

If I wanted to test the logic of this function without putting the actual logic into it, I could easily inject my own `Func` from a test method instead, so I wouldn’t have to care what the real logic is - that can be tested in a dedicated test elsewhere.

It’s another small way to make the structure even more useful.

# Pattern Matching in C# 9

Nothing major added in C# 9, but a couple of nice little features. The `and` and `not` keywords from `is` expressions now work inside the curly braces of one of the patterns in the list, and it’s not necessary any longer to have a local variable for a cast type if the properties of it aren’t needed.

Although not ground breaking, this does continue to reduce the amount of necessary boilerplate code, and gives us an extra few pieces of more expressive syntax.

I’ve added a few more rules into the next example using these features. Now there are two categories of PremiumBankAccounts with different levels of special interest rates^([1](ch03.html#idm45400879635904)) and another bank account type for a Closed account, which shouldn’t generate any interest.

[PRE8]

Not bad, is it?

# Pattern Matching in C# 10

Like C# 9, C# 10 includes just the addition of another nice time-and-boilerplate-saving feature. A simple syntax for comparing the properties of sub-objects belonging to the type being examined.

[PRE9]

In this slightly silly example, it’s now possible to exclude all “Simon"s from earning so much money in Monopoly when passing Go. Poor, old me.

I’d suggest taking another moment at this point to examine the function, above. Think just how much code would have to be written if it weren’t done as a Pattern Matching expression! As it is, it **technically** comprises just a single line of code. One…​really long…​line of code, with a whole ton of NewLines in to make it readable. Still, the point stands.

# C# 11

C# 11 contains a new pattern matching feature which probably has a somewhat limited scope of useage, but will be devestatingly useful when something fits into that scope.

The .NET team have added in the ability to match based on the contents of an Enumerable and even to deconstruct elements from it into separate variables.

Let’s imagine we were creating a very simple text-based adventure game. These were a big thing when I was very young. Adventure games you played by typing in commands. Imagine something like Monkey Island, but with no graphics, just text. You had to use your imagination a lot more back then.

The first task would be to take the input from the user and decide what it is they’re trying to do. In English commands will just about universially have their relevant verbs as the first word of the sentance. “GO WEST”, “KILL THE GOBLIN”, “EAT THE SUSPICIOUS-LOOKING MUSHROOM”. The relevant verbs here are GO, KILL and EAT respectively.

Here’s how we’d use C# 11 pattern matching:

[PRE10]

The “..” in the above switch expression means “I don’t care what else is in the array, ignore it”. Putting a variable after it is used to contain everything else in the array besides those bits that’re specifically matched for.

In my example above, if I were to enter the text “GO WEST”, then the GoTo action would be called with a single-element array ["WEST"] as a parameter, because “GO” was part of the match.

Here’s another neat way of using it. Imagine I’m processing people’s names into data structures and I want 3 of them to be FirstName, LastName and an array - MiddleNames (I’ve only got one middle name, but plenty of folks have many).

[PRE11]

In this example, the Person class is instantiated with:

FirstName = “Percy”, LastName = “Kent-Smith”, MiddleNames = [ “James”, “Patrick” ]

I’m not sure I’ll find many uses for this, but it’ll probably get me very excited when I do. It’s a very powerful feature.

## Discriminated Unions

I’m not sure whether this is something we’ll ever get in C# or not. I’m aware of at least 2 attempts to implement this concept that are available currently in Nuget:

1.  Harry McIntyre’s OneOf ([*https://github.com/mcintyre321/OneOf*](https://github.com/mcintyre321/OneOf))

2.  Kim Hugener-Olsen’s Sundew.DiscriminatedUnions ([*https://github.com/sundews/Sundew.DiscriminatedUnions*](https://github.com/sundews/Sundew.DiscriminatedUnions))

I cover Discriminated Unions and how they could be implemented in C# in an awful lot more detail in Chapter 6, so skip ahead to there if you want to see more.

In brief: they’re a way of having a type that might actually be one of several types. They’re available natively in F#, but C# doesn’t have them to date and it’s anyone’s guess whether they ever will be available.

In the meantime there are discussions happening over on GitHub ([*https://github.com/dotnet/csharplang/issues/113*](https://github.com/dotnet/csharplang/issues/113)) and proposals in existence ([*https://github.com/dotnet/csharplang/blob/main/proposals/discriminated-unions.md*](https://github.com/dotnet/csharplang/blob/main/proposals/discriminated-unions.md)).

I’m not aware of any serious plans to add them to C# 12, so for now we’ll just have to keep watching the skies!

## Active Patterns

This an F# feature I can see being added to C# sooner or later. It’s an enhancement to Pattern Matching that allows functions to be executed in the left hand “pattern” side of the expression. This is an F# example:

[PRE12]

What F# developers are able to do, as in this example, is to provide their own custom functions to go on the left hand “pattern” side of the expression.

“IsDateTime” is the custom function here, defined on the first line. It takes a string, and returns a value if the parse worked, and what is effectively a null result if it doesn’t.

The pattern match expression “tryParseDateTime” uses IsDateTime as the pattern, if a value is returned from IsDateTime, then that case on the pattern match expression is selected and the resulting DateTime is returned.

Don’t worry too much about the intricacies of F# syntax, I’m not expecting you to learn about that here. There are other books for F#, and you could probably do worse than one or more of these:

1.  Get Programming with F# by Isaac Abraham (Manning)

2.  Essential F# by Ian Russell ([*https://leanpub.com/essential-fsharp*](https://leanpub.com/essential-fsharp))

3.  F# for Fun and Profit by Scott Wlaschin ([*https://fsharpforfunandprofit.com/*](https://fsharpforfunandprofit.com/))

Whether either of these F# features becomes available in a later version of C# remains to be seen, but C# and F# share a common language runtime, so it’s not beyond imagining that they might be ported over.

## Read-only Structs

I’m not going to discuss Structs a great deal here, there are other excellent books that talk about the features of C#^([2](ch03.html#idm45400872249424)) in far more detail. What’s great about them from a C# perspective is that they’re passed between functions by value, not reference - i.e. a copy is passed in, leaving the original untouched. The old OO technique of passing an object into a function for it to be modified there, away from the function that instantiated it - this is anathema to a Functional Programmer. We instantiate an object based on a class, and never change it again.

Structs have been around for an awfully long time, and although they’re passed by value, they can still have their properties modified, so they aren’t immutable as such. At least until C# 7.2.

Now, it’s possible to add a readonly modifier to a Struct definition, which will enforce all properties of the struct as readonly at design time. Any attempt to add a setter to a property will result in a Compiler Error.

Since all properties are enforced as readonly, in C# 7.2 itself, all properties need to be included in the constructor to be set. It would look like this:

[PRE13]

This is still a little clunky, forcing us to update the constructor with every property as they’re added to the struct, but it’s still better than nothing.

It’s also worth discussing this case, where I’ve added in a List to the struct:

[PRE14]

This will compile, and the application will run, but an error will be thrown when the `Add()` function is called. It’s nice that the read-only nature of the struct is being enforced, but I’m not a fan of having to worry about another potential unhandled exception.

But it’s a good thing that the developer can now add the readonly modifier to clarify intent, and it will prevent any easily-avoidable mutability being added to the struct. Even if it does mean that there has to be another layer of error handling.

## Init only setters

C# 9 introduced a new kind of auto-property type. We’ve already got `Get` and `Set`, but now there’s also `Init`.

If you have a class property with `Get` and `Set` attached to it, that means the property and be retrieved or replaced at any time.

If instead it has `Get` and `Init`, the it can have its value set when the object it’s part of is instantiated, but can’t then be changed again.

That means our read-only structs (and, indeed all of our classes too) can now have a slightly nicer syntax to be instantiated and then exist in a read-only state:

[PRE15]

This means we don’t have to maintain a convoluted constructor (i.e. one with a parameter for literally every single property - and there could be dozens of them), along with the properties themselves, which has removed a potential source of annoying boilerplate code.

We still have the issue with exceptions being thrown when attempting to modify Lists and sub-objects, though.

## Record types

In C# 9, one of my favorite features since Pattern Matching was added - record types. If you’ve not had a chance to play with these yourself yet, then do yourself a favor and do it as soon as possible. They’re fantastic.

On the face of it, they look about the same as a struct. In C# 9, a record type is based on a class, and as such is passed around by reference.

As of C# 10 and onwards that’s no longer the case, and records are treated more like structs, meaning they can be passed by value. Unlike a struct however, there is no readonly modifier, so immutability has to be enforced by the developer. This is an updated version of the Blade Runner code:

[PRE16]

It doesn’t look all that different, does it? Where records come into their own though, is when you want to create a modified version. Let’s imagine for a moment that in our C# 10 application, we wanted to create a new movie record for the Director’s Cut of Blade Runner^([3](ch03.html#idm45400871809840)). This is exactly the same for our purposes, except that it has a different title. To save defining data, we’ll literally copy over data from the original record, but with one modification. With a read-only struct, we’d have to do something like this:

[PRE17]

That’s following the Functional paradigm, and it’s not too bad, but it’s another heap of boilerplate we have to include in our applications if we want to enforce Immutability.

This becomes important if we’ve got something like a state object that needs to be updated regularly following interactions with the user, or external dependencies of some sort. That’s a lot of copying of properties we’d have to do using the read-only struct approach.

Record types gives us an absolutely amazing new keyword - `with`. This is a quick, convenient way of creating a replica of an existing record but with a modification. The updated version of the Director’s Cut code with record types looks like this:

[PRE18]

Isn’t that cool? The sheer amount of boilerplate you can save yourself with record types is staggering.

I recently wrote a text adventure game in Functional C#. I made a central GameState record type, containing all of the progress the player has made so far. I used a massive Pattern Matching statement to work out what the player was doing this turn, and a simple **with** statement to update state by returning a modified duplicate record. It’s an elegant way to code state machines, and clarifies intent massively by cutting away so much of the uninteresting boilerplate.

A last neat feature of Records is that you can even define them simply in a single line like this:

[PRE19]

Creating instances of Movie using this style of definition can’t be done with curly braces, it has to be done with a function:

[PRE20]

Note that all properties have to be supplied and in order, unless you use constructor tags like this:

[PRE21]

You *still* have to provide all of the properties, but you can put them in any order you like. For all the good that does you…​

Which syntax you prefer is a matter of preference. In most circumstances they’re equivalent.

## Nullable Reference Types

Despite what it sounds like, this isn’t actually a new type, like with record types. This is effectively a compiler option, which was introduced in C# 8\. This option is set in the CSPROJ file, like in this extract:

[PRE22]

If you prefer using a UI, then the option can also be set in the Build section of the project’s properties.

Strictly speaking, activating the Null Reference Types feature doesn’t change the behavior of the code generated by the compiler, but it does add an extra set of warnings to the IDE and the compiler to help avoid a situation where NULL might end up assigned. Here are some that are added to my Movie record type, warning me that it’s possible for properties to end up null:

![Warnings for nullable properties on a Record](assets/ch03_001.png)

Another example occurs if I try to set the title of the Blade Runner Director’s Cut to NULL:

![Warning for setting a property to NULL](assets/ch03_002.png)

Do bear in mind that these are only compiler warnings. The code will still execute without any errors at all. It’s just guiding you to writing code that’s less likely to contain null reference exceptions - which can only be a good thing.

Avoiding the use of a NULL value is generally a good practice, functional programming or not. NULL is the so-called “Billion dollar mistake”. It was invented by Tony Hoare in the mid-60s, and it’s been one of the leading causes of bugs in producion ever since. An object being passed into something that turned out unexpextectly to be NULL. This gives rise to a Null-Reference Exception, and you don’t need to have been in this business long before you encounter your first one of those!

Having NULL as a value adds unneeded complexity to your codebase, and introduces another source of potential errrors. This is why it’s worth paying attention to the compiler warning and keeping NULL out of your codebase wherever possible.

If there’s a perfectly good reason for a value to be NUL then you can do so by adding *?* characters to properties like this:

[PRE23]

The only circumstance under which I’d ever consider deliberately adding a nullable property to my codebase is where a 3rd party library requires it. Even then, I wouldn’t allow the Nullable to be persisted through the rest of my code - I’d probably tuck it away somewhere where the code that parses the external data can see it, then convert it into a safer, more controlled structure for passing to other areas of the system.

# The Future

As of the time of writing, C# 11 is out and well established as part of .NET 7\. The plans are only starting to be put together for .NET 8 and C# 12, but it’s not clear what - if anything - they’ll be including for functional programmers. Given that it’s a stated intent by the C# team to continue to add more Functional features with every version, it’s a safe bet that there’ll be at least *something* new for us to do more Functional Programming with.

# Summary

In this chapter, we looked at all the features of C# that have been released since Functional Programming began to be integrated in C# 3 and 4\. We looked at what they are, how they can be used, and why they’re worth considering.

Broadly these fall into two categories:

*   Pattern Matching, implemented in C# as an advanced form of switch statement that allows for incredibly powerful, code-saving logic to be written briefly and simply. We saw how every version of C# has contributed more pattern matching features to the developer.

*   Immutability, the ability to prevent variables from being altered once instantiated. It’s highly unlikely that true Immutability will ever be made available in C# for reasons of backwards-compatability, but new features are being added to C#, such as readonly structs and record types that make it easier for a developer to work in such a way that it’s easy to pretend that immutablility exists without having to add a lot of tedious boilerplate code to the application.

In the next chapter, we’re going to take things a step further, and demonstrate some ways to use some existing features of C# in novel ways to add to your Functional Programming tool belt.

^([1](ch03.html#idm45400879635904-marker)) Which, frankly no bank would ever offer

^([2](ch03.html#idm45400872249424-marker)) You could do worse than *C# in a Nutshell*, also published by O’Reilly

^([3](ch03.html#idm45400871809840-marker)) Vastly superior to the theatrical cut, in my opinion.