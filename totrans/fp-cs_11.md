# Chapter 11\. Practical Functional C#

I’m not just a pretty face^([1](ch11.html#idm45400846563120)) as well as spend my days slogging at the virtual IT coalface each day, I’ve also been priviliged enough to spent a lot of time over the years talking at various events on the subject too^([2](ch11.html#idm45400846562432)). While at talks, there are a few questions that come up on a fairly regular basis.

The most common is actually something like “Why don’t we just use F#”. See all the way back in [“What about F#? Should I be learning F#?”](ch01.html#Chapter1_what_about_Fsharp) for my answer to that particular question. That pretty much comes up at just about every event I’ve ever spoken at, which is one of the reasons I gave such a detailed answer.

Oddly, the second most common is to explain Monads (which I did in [Chapter 7](ch07.html#Chapter7_Beginning)). Hopefully after getting to this point you’re something of an expert on that yourself now.

After these, the next most common question is about performance. There’s a widespread belief that functional programming in C# is inefficient in production code compared with Object-Oriented (OO). I’d like to spend a section of the book now, talking about the issue of performance, and whether it’s something that you need to be concerned about before adopting functional C# in your everyday life. Or, at least, your everyday life that involves .NET code. For me at least there’s a rather large overlap between those two things.

# Functional C# and Performance

I’m going to continue on now with a look at Functional C# and performance. To do that, I’m going to need a bit of code to use as a test subject with which I can compare imperative (i.e. the programming paradigm that Object-Oriented Programming belongs to) code to the various flavors of functional C#.

Now, I’m a big fan of the annual coding event, the Advent of Code^([3](ch11.html#idm45400846556400)) and the very first challenge ever published in their first event in 2015 is one I often link people to as a good example of how functional thinking can make a difference.

The problem is this:

The input is a string consisting strictly of the characters *(* and *)*. These represent the movements of an elevator^([4](ch11.html#idm45400846552176)). A *(* represents a movement up a floor, and *(* represents a movement down a floor. We start on the Ground floor, represented not by the letter *G*, but a 0, meaning we can work exclusively with an integer value to represent the current floor, with negative numbers representing floors beneath the ground.

There are two parts, which will serve perfectly for performance tests. The first is to run a series of instructions and calculate the final floor. The second is to work out which character of the input string gets you to floor -1 (i.e. the basement).

Here’s an example for you. Given the input string "((((“, we go up 3 floors, then down 5, then again up 4 floors.

For part 1, the answer is 2, the final floor we end up on, because 3-5+4 = 2.

For part 2, the answer is 6 - the 7th character is the first to put us on floor -1, but the array location is 6, because it’s a zero-based array.

These two puzzles are great examples of both definite and indefinite loops, and the input provided by the puzzle is large enough (over 7,000 characters) that it’ll cause the code to need to work for a bit. Enough for me to gather some statistics.

If you care about spoilers, go ahead to the page^([5](ch11.html#idm45400846546672)) and solve the puzzle before continuing. Just know that if you use a functional approach, you can solve it in a single line!

OK, spoiler warning now, after this section I’m launching into a full-on spoiler heavy set of solutions to this - currently 8 year old - puzzle.

## Baseline - An Imperative Solution

Before I look at performance in the various functional solutions I have prepared, I want to look first at how performance looks in an imperative solution. This is something of a scientific experiment, and to be a proper experiment, we need a control. A baseline to compare all of the Functional results to.

For performance measuring, I’m using Benchmark.NET. For those of you not familiar with this tool, it’s similar in some ways to unit testing, except the point is that it’ll compare the performance between several versions of the same code. It runs the same code many times, in order to get a mean of things like time taken to run and the amount of memory used.

The following are solutions to the two puzzles entirely using Imperative style coding.

[PRE0]

There might be better solutions, but this one will serve, I feel.

## Performance Results

Now these tests were performed for a 7,000 character input on my developer’s laptop, so the actual numbers you might see when trying to replicate this experiment are likely to vary. The main point of the next few sections is to compare the results from the same test set-up.

### Imperative Baseline Results

As it happens, my results for this imperative solution were:

Table 11-1\. Object Orientated Performance Results

| Loop Type | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- |
| Definite | 10.59μs^([a](ch11.html#idm45400846388336)) | 0.108μs | 24b |
| Indefinite | 2.226μs | 0.0141μs | 24b |
| ^([a](ch11.html#idm45400846388336-marker)) These are Microseconds. |

Despite the size of the task, the actual time taken is very small indeed. Not too shabby. The Indefinite loop is faster, but you’d expect that - it’s not having to loop through the entire input string.

In the next sections, I’m going to go through a few different FP implementations of each loop type, and see what difference it makes to performance.

### Definite Loop Solutions

I did say that it was possible to solve this in a single line, didn’t I? It’s a fairly long line, but a single line nevertheless. Here’s my line:

[PRE1]

I’ve put some newline characters in there to make it readable, but it’s still technically a single line!

What We’re doing is performing a `Sum` aggregation method which adds either 1 or -1 at each iteration, based on the current character. Bear in mind that in C#, `string` is both a bit of text *and* an array, which is why I can apply LINQ operations to it like this.

What effect does this have on performance? Let’s have a look.

Table 11-2\. Sum Aggregation Performance Results

| Solution | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- |
| Imperative Baseline | 10.59μs | 0.108μs | 24b |
| Sum Aggregation | 60.75μs | 0.38μs | 56b |

There’s no avoiding the fact that performance here is indeed worse. This is just out of the box LINQ, so even using one of the Microsoft-provided tools, we still aren’t performant as imperative code.

What do you think, should I throw in the towel now? Not likely! I’ve got a few more things I’d like to try.

How about if we were to separate out the conversion of `char`→ `int` transformation into 2 separate lines?

[PRE2]

Does this makes any difference?

Table 11-3\. Select then Sum Aggregation Performance Results

| Solution | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- |
| Imperative Baseline | 10.59μs | 0.108μs | 24b |
| Select/Sum Aggregation | 84.89μs | 0.38μs | 112b |

Well, actually that’s a little worse. Ok, how about converting to another data structure, like a Dictionary, which has a fantastic reading speed.

[PRE3]

This time I’m creating a `Grouping` where each possible `char` value is one `Group` within it. In our example, there will only ever be 2 `Groups` with a key of either *(* or *)*. Once I’ve got the groupings, I’m deducting the size of one group from the other to get the final floor (i.e. deduct the total number of moves down from the total number of moves up).

Table 11-4\. Select then Sum Aggregation Performance Results

| Solution | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- |
| Imperative Baseline | 10.59μs | 0.108μs | 24b |
| Group/Dictionary Aggregation | 93.86μs | 0.333μs | 33.18Kb |

Not only is that even worse, but the amount of allocated memory is horrible.

I’d still like to try an indefinite loop, and a few more things besides before I wrap up with my thoughts on what all of this means. For now though - I don’t think this is as bad as it looks. Keep reading to see what I mean.

## Indefinite Loop Solutions

Now I’m going to try out a few solutions for the indefinite loop puzzle - i.e. which character of the input string is the first to bring us to floor -1\. I can’t say where that’ll be, but we’ll need to loop until the condition is met.

First, I’ve always heard that recursion in C# was a bad idea if you value your stack. Let’s see just how bad things could be. This is a recursive solution to the problem:

[PRE4]

Nice, neat and compact. but what’s the performance hit like?

Table 11-5\. Recursive Loop Performance Results

| Solution | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- |
| Imperative Baseline | 2.226μs | 0.0141μs | 24b |
| Recursive Loop | 1,030ms | 4.4733μs | 20.7Mb |

That’s honestly quite shocking. Note that those time results are in *milli* seconds, not *micro* seconds. that’s a thousand times worse. The memory usage is also too large to be stored on a box of old floppy discs.

You now have the evidence before you, if it were needed, that recursion is a *really* bad idea in C#. Unless you’re sure of what you’re doing, I’d avoid it alltogether.

What about non-recursive functional solutions? Well, these are the ones that require a compromise of some sort. let’s start with the use of my `IterateUntil` function from [“Trampolining”](ch09.html#Chapter_9_Trampolining). How does that affect performance?

This is my code:

[PRE5]

This time I need something to track state with. I’m trying a `record` type, since they’ve been provided to us to allow more functional code to be written. So, what are the results:

Table 11-6\. Trampolining Performance Results

| Solution | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- |
| Imperative Baseline | 2.226μs | 0.0141μs | 24b |
| Trampolining | 24.050μs | 0.3215μs | 55.6Kb |

Not so shocking this time, but still worse than the imperative version. there’s still quite a large amount of memory being stored during the operation too.

How about if I were to swap `record` out for the previous functional-style structure Microsoft provided - the Tuple. Would that improve performance?

[PRE6]

That’s unfortunately not quite as friendly to look at. I adore the lovely syntactic sugar that `record` brings to us, but if performance is our goal, then readability and maintainability may have to be sacrifices on its altar.

If you look at Table 11-7, you’ll see how the performance was:

Table 11-7\. Trampolining With Tuples Performance Results

| Solution | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- |
| Imperative Baseline | 2.226μs | 0.0141μs | 24b |
| Trampolining with Tuples | 17.132μs | 0.0584μs | 24b |

That’s actually really quite substantially better. The time taken is still a little worse, but the amount of memory allocated is actually exactly the same!

The last test I want to do is the custom `Enumerable` option I demonstrated in [“Custom Iterator”](ch09.html#Chapter_9_custom_Iterator). How does that compare to trampolining tuples^([6](ch11.html#idm45400845693904))

[PRE7]

That’s an awful lot of code, but is it any more performant?

Table 11-8\. Custom Enumerable Performance Results

| Solution | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- |
| Imperative Baseline | 2.226μs | 0.0141μs | 24b |
| Custom Enumerable | 24.033μs | 0.1072μs | 136b |

The time taken is pretty much identical to the Trampolining example with a `record`, but there’s more data allocated in this version. Not too bad, and still no-where near the calamity that is the recursive version.

Another option we’ve got is to try and interop F# into C# to have guaranteed functional code when C# really won’t play along. Let’s have a look at that.

## Interop with F# Performance

It’s actually possible to write code in an F# project and reference it in C# as if it were just another .NET library. It’s a fairly simple process, and I’ll walk you through it..

Create a new Project in your Visual Studio Solution, but instead of a C# Library, select F# from the dropdown.

You can reference the F# project in your C# code. Please note though, that the code you’ve written won’t be visible in C# unless you compile the F# project separately. I’m not sure why this is, but it’s a necessary step.

Since this is a C# book, so I’m not going to go into how to write F#, but it’s interesting to see how the performance would compare if we absolutely had to have a piece of code entirely functionally without having to compromise with the limitations of functional C#.

In case you’re curious, here is an F# solution to the problem - don’t worry if you don’t understand exactly how it works. I’m presenting it here as a curio, rather than something you need to learn^([7](ch11.html#idm45400845359936)).

[PRE8]

This code is purely functional. `Seq` is the F# equivalent of `Enumerable`, so there are some neat efficiency saving features at work here due to the lazy-loading nature of both of those types.

But what about the performance? Will the performant nature of F# be countered somehow by being referenced over a C# channel? Let’s have a look at table 11-9 to see the results…​

Table 11-9\. F# Interop Performance Results

| LoopType | Solution | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- | --- |
| Definite | Imperative Baseline | 10.59μs | 0.108μs | 24b |
| Definite | F# Interop | 63.63μs | 0.551μs | 326b |
| Indefinite | Imperative Baseline | 2.226μs | 0.0141μs | 24b |
| Indefinite | F# Interop | 32.873μs | 0.1002μs | 216b |

That’s still worse, it turnsout, but only a couple of times. It’s a viable option if you want to go down that route. It’ll mean learning F# first though, and that’s far outside the scope of this book. At least you know it’s there in the future if you enjoy C# enough that you want to take it further still.

## External Factors and Performance

Believe it or not, everything we’ve looked at so far is actually incidental in comparison to making any sort of interaction with the world outside your C# code. Let me show you what I mean.

In this next experiment I’ve modified both the original OO baseline functions and the most performant version of the FP solution, the one that used Tuples. I’ve set them not to accept any input strings, but instead to load the *same* data from a file stored in the local Windows file system. Functionally I’ve got the same input and same result, but there’s a file operation involved now.

What sort of difference could that make? Well, table 11-10 just so happens to contain the answer…​

Table 11-10\. File Process Performance Results

| LoopType | Solution | Mean Time Taken | Time Taken Standard Deviation | Memory Allocated |
| --- | --- | --- | --- | --- |
| Definite | Imperative Baseline | 10.59μs | 0.108μs | 24b |
| Definite | Imperative With File | 380.21μs | 15.187μs | 37.8Kb |
| Definite | FP with File | 450.28μs | 9.169μs | 37.9Kb |
| Indefinite | Imperative Baseline | 2.226μs | 0.0141μs | 24b |
| Indefinite | Imperative with File | 326.006μs | 1.7735μs | 37.8Kb |
| Indefinite | FP with File | 366.010μs | 2.2281μs | 93.22Kb |

How about that? It’s still the case that the functional solutions take a little longer, but the proportion of difference is far less. When we compare the in-memory version of the FP solution with a definite loop using Tuples to the imperative equivalent, the FP version takes about 8.5 times more time to complete. When we compare the two versions that include a file load, the proportional difference is only about 1.2 times more time required. Hardly that much more.

Imagine doing this if there were HTTP calls to Web APIs or Database connections established over a network? I can guarantee you that the time taken would be far, far worse than what we see here.

Join me, if you would, in the next section where I’ll put together my final thoughts on what conclusions we can draw from these experiments.

# What does all of this mean?

First-off - It’s an inescapable fact that functional C# is less efficient than well-written Object-Oriented code. That’s just the way it is.

We can also show that if possible, Pure LINQ operations, kept as compact as possible are the most effective of the functional features. If a state object of some kind is required, then a Tuple is currently the best choice. This assumes, though that performance is the most important goal of our code. Is it always? It depends entirely on what you’re trying to achieve, and who your customer is.

If you’re planning to make an elaborate VR application with high-definition 3D graphics, then honestly you want to stear well away from functional alltogether. For things like that, you’ll need to squeeze every last bit of performance from your code that you possibly can, and then some.

What about the rest of us? I’ve worked in a lot of different companies, and I would say that in nearly all of them of the many drivers behind my development work, performance wasn’t necessarily the most critical of them.

Mostly what I’ve worked on is bespoke web applications used by internal employees of the company to carry out some sort of business process as part of their daily workload. Whenever a requirement has been handed to me, it’s usually more imporant to get it completed and released to production quickly, rather than to spend time worrying about performance.

Even if there were issues with the app slowing down, these days it’s easy enough to pop onto Azure or AWS and click a button to add an extra virtual RAM chip to the virtual server, and the problem largely goes away.

But what about the cost to the business for that nasty old RAM chip?

Well, what about it? Let me put it to you this way. Functional style C# is far easier to understand, modify and maintain than the Object-Oriented equivalent. We can get changes made and shipped quicker, and with a greater chance that everything will work the first time and without errors.

Given that fact, what would cost the company more? One extra virtual RAM chip, or the amount per hour that your time costs as you do all of the extra development work, and resolve the unnecessary bugs that would otherwise end up in production?

If nothing else, functional C# is a more pleasant developer experience, and isn’t it important to ensure that the work environment is one that’s keeping everyone happy?

In any case - even though I’ve talked about the potential need for more RAM, would we necessarily even need it?

In the experiements I’ve done in the previous sections of this chapter, you can see that although the functional solutions need more time than the object-oriented equivalents - the proportional difference once we invoke a file operations isn’t really all that much. Hardly enough to start getting worried about.

Here are the performance results of the two solutions that used Files as input. The two sections of the each pie chart represent the time required to load the file and the time required to process the data that was loaded.

![Images/chapter10_pieChart.png](assets/chapter10_pieChart.png)

###### Figure 11-1\. A Comparison of Performance of Tests with File Operations

It doesn’t look quite as bad, when you see it in perspective, does it? Unless, that is, you really are working on a project where there are financial consequences to every tiniest loss in performance. If that’s the case, I’d suggest putting down this book, and pick up one instead on *High Performance .NET code*. There are a few about.

If you’re less utterly, singularly focused on performance, and you’d be happier with nicer code that’s easy to maintain, then I’d suggest that any marginal loss in perforance is a price worth paying to be able to write code in this style.

As is always the case, it’s down to you. Your personal coding preferences. The constraints of your working environment and the team you’re part of. And, of course, whatever it is you’re trying to develop.

You’re a grown up^([8](ch11.html#idm45400845090960)), so I’ll leave you to decide for yourself.

For the second part of this chapter, I’m going to consider a few different practical concerns you might have regarding functional C# in a production environment, and what my opinion is on each.

# Functional C# Concerns & Questions

Right, no time to waste. Let’s get cracking. First question, please!

## How functional should I make my codebase?

One of the beauties of functional C# is that the functional content isn’t part of a framework. You can pick and choose how much you want to do functional or not to and that sliding scale can move just about as far to either side as you want it to.

You can decide to make your entire codebase function. Or restrict it to a single project. A single class. A single function. Or - if necessary - a single line. Mix functional and non-functional in the same function. It can be done. I probably wouldn’t. But the point is you *can*.

I’m not a purist. I understand that sometimes in production there are all sorts of other concerns that need to be considered beyond making the code exactly the way you want it.

There’s not just *where* should you be functional, but to what *extent*. Do you want to go all-out. Use Partial Application, Monads, Discriminiated Unions, and the rest of it. Or - you might feel just using an awful lot of LINQ to replace `While` loops. That’s fine too. Whatever you feel comfortable with. There’s no need to feel like a traitor to the cause. We’re developers, not the Judean People’s Front^([9](ch11.html#idm45400845082960)).

Another consideration is how functional your colleagues are comfortable with. Do consider that however beautiful your functional code may be, it still has to be maintained by the team. I’d argue that Monads aren’t really all that hard to understand, but you still need to convince everyone to learn how to use them before they’ll be ready to support code containing them. Even if it might be in their best interests.

You can lead a horse to water, and all that…​

As I’m always saying - at the end of the day, the question you have to ask yourself is, “What am I trying to achieve?”. Once you’ve answered that truthfully, your decision-making process should be relatively easy.

You could always leave a few copies of this book lying around the office. See what happens. I don’t know how long after publication date you’re reading this, but there’s even a chance that I may still be around. Feel free to get folks to reach out to me with questions.

## How Should I Structure a Functional C# Solution?

In classes and Folders, largely the same as an existing OO C# project. Strictly speaking Classes are *not* a functional concept. If you look at F# projects, they don’t necessarily have classes at all.

You can contain all of your code in Modules, but they aren’t really the same thing. They’re convenient ways for F# developers to group code. More like a C# Namespace. There’s no functional meaning to a Module.

In C# on the other hand, there *have* to be classes. There’s no way around it, so that will still have to be a thing you do.

In pure functional style C#, you could make every Class and Function static. If you did that though, I feel sure you’re going to run into practical problems somewhere^([10](ch11.html#idm45400845076240)), especially when you have to try and interact with a non-functional Class from Nuget or one of the built-in Microsoft classes.

Lack of dependency injection would also make unit testing far harder. Languages like Haskell get around this by use of Monads of various kinds. In C# though, I’d make your life easy and just use standard Classes and an IoC container, as hopefully you already have been.

## How Do I Share my Functional Methods Between Applications?

I maintain a set of functional libraries which are full of classes and extension methods, and these provide all of my implementations of the Monads, Discriminiated Unions and other useful extension methods I want to use in my code. I usually have a separate project in the solution called “Common” where I place anything generic like that. That way it can all sit somewhere in the codebase where I can use it, but I don’t necessarily have to look at it again while I’m getting on with my work.

The contents of my Common project are copied from solution to solution for the time being.

At some point at work, we’re planning to set up our own local instance of Nuget, and when we do so we’ll probably have the Common project set up as a consumable Nuget package. That’ll make it easy to distribute bug fixes, improvements, new features, etc. For now though, copying over every time mostly works.

## Did you order this Pizza?

Erm, I’m not sure. That looks like it’s got anchovies on it, so probably not for me. I asked for a meat feast. Let me just check with Liam, I think it might be his. 5 seconds, I’ll be right back…​

## How do I Convince my Team Mates to do this too?

Good question. When you work out how to do this consistently, let me know how you managed it.

I’ve been giving talks on this subject for a long time now, and reactions to functional programming seem fall into one of a few categories:

*   OMG! This is the holy grail of development! I want Moooooore!

*   This is vaguely interesting, but I think I’ll carry on working the way I always have, thanks.

*   This is the worst. Boo!

I couldn’t give you any real statistics on how many people fall into each camp, but my feeling is it’s roughly a small number in the first and last group, and the vast majority in the middle.

That being the case, it’s our job as functional programming advocates to try and convince them of the benefit to changing their ways. Bearing in mind, most human beings are very much creatures of habit, and don’t really want to have to make huge changes to their daily lives like that - unless there’s a good reason.

It also depends where you work and what your project constraints are. If you’re creating mobile applications where every bit of memory is crucial, or a 3D graphics system for a VR device where you can’t afford even the tiniest bit of inefficiency in your code, then you probably will find FP quite a hard sell.

If, however, you don’t do those things, then there may well be a possibility you can convince everyone by talking about the benefits. Just don’t become a functional programming bore. There may come a point that it’s obvious there’s no traction with everyone else. At that point it’s best to hold off, and hope there’s a chance you can win the long game by attrition, rather than via a big, bold frontal attack.

The benefits I’d focus on, in roughly descending order, are:

*   Reliability - functional applications tend to be more robust. Tend to fail less. Functional programming also massively enables unit testing. So by following this paradigm, you’ll end up with a higher quality product that fails less. This will save the company a whole load of money in time that would otherwise be required to resolve bugs in production.

*   Development Speed - It’s usuall quicker and easier to write code using the functional paradigm (once you’re familiar with it, that is!) and enhancements tend to be especially easy due to the way that functional code is structured. I’d talk about the amount of time and money that will be saved in the initial development effort.

*   Microsoft Support - It’s a stated goal of the .NET team to support the functional paradigm. Talk about how this style of programming isn’t abusing .NET, it’s actually *using* it the way it was intended. Mostly…​

*   Easy to Learn - I hope you feel that way now, after getting to this point in the book. I don’t feel that functional programming is all that hard to learn, at least not once you dispence with all of the formal definitions and scary-sounding terminology. In fact, there is *less* to learn to adopt FP than it would take for a new developer to learn the object-oriented paradigm fully. I’d reassure everyone that the learning overhead isn’t all that big.

*   Pick and choose - Mention also that you can adopt as much, or as little of the paradigm as you want. It’s not like you need throw away your old codebase and move to an entirely different one. You can start small by refactoring a single function, and you might not even choose to use Monads (although I think you should!).

*   It’s not new! - It might finally be worth talking about how old the functional paradigm is. Many companies may be reluctant to adopt a new, trendy technology that hasn’t proven it’ll stay around long enough to be worth the investment. That’s a reasonable concern, but FP was first used as a software development concept in the 1960, and its roots go back to the late 1800s. It’s already been proven a million times over in production environments.

I really *wouldn’t* necesarilly start with talk right away of the 3 laws of the Monad, F#, currying or anything like that. Keep it all as familiar as you can.

Aside from talking about it yourself, there’s always this book, and many more other good books on the subject. There’s a section coming up shortly in which I’ll recommend the ones I like the most.

## Is it worth Including F# Projects in my Solution?

Up to you. Performance-wise there’s no real issue with it I’m aware of.

The best usage might be to consider making the deeper, rules-based parts of your codebase F#. The functions that convert data from one form to another based on a set of business requirements.

F# is likely to make those parts neater, more robust and more performant.

It’ll also be a whole ton more functional than anything C# can do, if that’s what you’re after.

The only thing I’d consider is whether your team is willing to support it or not. If they are - great. Go for it. If not, get together with them and have a discussion. That sort of decision needs to be made by everyone.

If your team aren’t comfortable with C#, you can at least take some comfort in the fact that most of the functional paradigm is still here in C#.

## Will Functional Coding Solve All of my Problems?

Depends what problems you’re facing?

Functional programming is unlikely to improve your Poker game, or make you cups of coffee when you’re in need.

It certainly will provide you with a better codebase, one that’ll work better in production, and that’ll be easier to support, going forward.

It’s still possible for someone that’s determined to, to write bad code, or to be lazy one day and make a mistake. There’s nothing on this planet^([11](ch11.html#idm45400845047216)) that’ll stop that happening. Well, besides the usual methods of enforcing automated testing, code reviews and manual quality checks. The sorts of things that have mostly been around since the beginning of development as an industry.

Functional style coding will make it easier to *spot* problems, however. It’s concise style makes it easy to tell at a glance what a function actually *does* and whether it’s actually doing what its name suggests.

Functional also - as stated earlier in this chapter - won’t be the *most* performant solution to your coding requirement, but it’s near enough that unless absolute peak performance matters to you, then it’s likely to be just fine.

None of this will help with any of your usual project management issues either. Issues with unclear requirements are between yourself and whichever business analyst you believe in.

Functional programming will make you cool, though. Kids might even give you the thumbs up in the street as you pass. Such is the street cred of FP. True fact.

## Connery, Moore or Craig?

None of the above, I’m a big fan of Timothy Dalton. He deserved more attention.

## Thinking a Problem through Functionally

There isn’t one way to do this any more than there’s *one* way to develop a piece of software. If it helps, though - I’ll briefly describe my process.

I would start by thinking through the *logical* steps of the code you’re trying to write. By which, I mean try splitting it into the steps you’d describe if you were talking through your work with someone else. i.e. “First I’d do X, then I’d to Y, then Z”. This isn’t a way you can work especially with Object-Oriented style code, but it’s actually the best way to split up functional code.

I’d then write each piece of functional code based on those steps.

Wherever possible as well, I’d consider making whatever you’re working on an `Enumerable` of some kind, whether that be of primitives, complex objects or `Func` delegates. Functional programming is often at its most potent when you’re running list-based operations, like in T-SQL.

I would advise against making chains of functions too long. You want opportunities when you’re developing to be able to examine the previous steps of a complex calculation, to make sure everything is working the way you expecct. It also gives you chances to apply meaningful names to what you store of each stage of the process.

Functional programming supports unit testing very well, so I’d also suggest breaking the entire process up into as many smaller functions as you can, and then make sure you’ve tested each one as thouroughly as you can.

If you’ve got the advantage of being able to break the steps up logically, then use it to the best effect you can.

# Conclusion

This chapter was a game of two halves.

In the first we considered the myth of poorly-performing functional code, and hopefully busted it to some extent. We saw that it’s a real phenomenon, but the difference is insignificant compared to any amount of I/O in your code.

In the second half, we considered some of the more esoteric, philosophical issues of functional programming in the industrial setting.

Functional Programming will return in You Only live Twice Chapter 12\. In which I’ll look into the options you have for doing functional programming using 3rd party packages from Nuget.

^([1](ch11.html#idm45400846563120-marker)) OK, I"m not even that, but leave me my illusions, won’t you!

^([2](ch11.html#idm45400846562432-marker)) It was that or bore my family about it.

^([3](ch11.html#idm45400846556400-marker)) Website available here: [*https://adventofcode.com/*](https://adventofcode.com/). Two coding challenges per day for 24 days leading up to Christmas. I’ve never managed to complete an event in real-time, but they’re fantastic puzzles.

^([4](ch11.html#idm45400846552176-marker)) Or “lift”, for my fellow countrymen

^([5](ch11.html#idm45400846546672-marker)) this is where you’ll find it: [*https://adventofcode.com/2015/day/1*](https://adventofcode.com/2015/day/1) have fun!

^([6](ch11.html#idm45400845693904-marker)) Now, try saying **that** ten times fast! I dare you!

^([7](ch11.html#idm45400845359936-marker)) Thanks once again to F# guru Ian Russell for this code.

^([8](ch11.html#idm45400845090960-marker)) Probably. Fair play to you if you aren’t and you’ve read this far. You’ll do well in life!

^([9](ch11.html#idm45400845082960-marker)) Splitters!

^([10](ch11.html#idm45400845076240-marker)) Over the Rainbow?

^([11](ch11.html#idm45400845047216-marker)) Probably nothing off it either, but who knows…​