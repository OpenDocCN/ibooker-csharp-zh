# Chapter 6\. Discriminated Unions

Discriminated unions (DUs) are a way of defining a type (or class in the OO world) that is actually one of a set of different types. Which type an instance of a DU actually is at any given moment has to be checked before use.

F# has DUs available natively, and it’s a feature used extensively by F# developers. Despite sharing a common runtime with C#, and the feature being there for us *in theory*, there are only plans in place to introduce them into C# at some point - but it’s not certain how or when. What we can do in the meantime is roughly simulate them with abstract classes, and that’s the technique I’m going to talk about in this chapter.

This chapter is our very first dabble into some of the more advanced areas of functional programming. Earlier chapters in the book were more focused on how you, the developer can work smart, not hard. We’ve also looked at ways to reduce boilerplate, and to make code more robust and maintainable.

Discriminated unions^([1](ch06.html#idm45400862303440)) are a programming structure that will do all of this too, but are more than a simple extension method, or a single-line fix to remove a little bit of boilerplate. DUs are closer in concept to a design pattern - in that they have a structure, and some logic that needs to be implemented around it.

# Holiday Time

Let’s imagine an old-school Object-Oriented problem where we’re creating a system for package holidays. You know - the sort where the travel agency arrange your travel, accomodation, etc. all in one. I’ll leave you to imagine what lovely destination you’re off too. Personally, I’m quite fond of the Greek islands.

[PRE0]

Now imagine we were creating, say, an account page for our customers^([2](ch06.html#idm45400862223056)), and we want to list everything they’ve bought so far. Not all that difficult, really. We can use some relatively-new `is` statement to build the necessary string. Here’s one way we could do it:

[PRE1]

If I wanted to quickly improve this with a few functional ideas, I could consider introducing a Fork combinator (see previous chapter), The basic type is Holiday, the sub-type is a Holiday with a Meal. Essentially the same thing, but with an extra field or two.

What if…​there was a project started up in the company. Now, they’re going to start offering other types of service, separate from holidays. They’re going to also start providing day trips that don’t involve hotels, flights, or anything else of that sort. Entrance into Tower Bridge^([3](ch06.html#idm45400862084240)) in London, perhaps. Or a quick jaunt up the Eiffel Tower in Paris. Whatever you fancy. World’s your oyster.

The object would look something like this:

[PRE2]

The point is though, that if we want to represent this new scenario with inheritance from a Holiday object, it doesn’t work. An approach I’ve seen some people follow is to merge all of the fields together, along with a boolean to indicate which fields are the ones you should be looking at.

Something like this:

[PRE3]

This is a poor idea for several reasons. For one, you’re breaking the Interface Segregation principle. Whichever type it really is, you’re forcing it to hold fields that are irrelevant to it. We’ve also doubled up the concepts of “Destination” and “Attraction”, as well as “DateOfTrip and “StartDate” here, to avoid duplication, but it means that we’ve lost some of the terminology that makes code dealing with day trips meaningful.

The other option is to maintain them as entirely separate types of object with no relationship between them at all. Doing that though, we lose the ability to have a nice, concise, simple loop through every object. We wouldn’t be able to list everything in a single table in date order. There would have to be multiple tables.

None of the possibilities seem all that good. But this is where DUs come charging to the rescue. In the next section, I’ll show you how to use them to provide an optimum solution to this problem.

# Holidays with Discriminated Unions

In F#, you can create a union type for our customer offering example, like this:

[PRE4]

What that means is that you can instantiate a new instance of CustomerOffering, but there are three separate types it could be, each potentially with their own entirely different properties.

This is the nearest we can get to this approach in C#:

[PRE5]

On the face of it, it doesn’t seem entirely different to the first version of this set of classes, but there’s an important difference. The base is abstract - you can’t actually create a CustomerOffering class. Instead of being a family tree of classes with one parent at the top that all others conform to, all of the sub-classes are different, but equal in the hiararchy.

Here’s a class hiararchy diagram, which makes the difference between the two approaches clearer:

![OO vs Discriminated Union](assets/Discriminatedunion.png)

The DayTrip class is in no way forced to conform to any concept that makes sense to the Holiday class. DayTrip is completely its own thing. It means it can use property names that correspond exactly to its own business logic, rather than having to retro-fit a few of the properties from Holiday. In other words - DayTrip isn’t an **extension** of Holiday, it’s an **alternative** to it.

This also means you can have a single array of all CustomerOfferings, even though they’re wildly different. No need for separate data sources.

We’d handle an array of CustomerOffering objects in code using a pattern matching statement:

[PRE6]

This simplifies the code everywhere the discriminated union is received, and gives rise to more descriptive code, and code that more accurately describes all of the possible outcomes of a function.

# Schrödinger’s Union

If you want an analogy of how these things work, think of poor old Schrödinger’s Cat. This was a thought experiment proposed by an Austrian physisist Erwin Schrödinger to highlight a paradox in quantum mechanics. The idea was that given a box containing a cat^([4](ch06.html#idm45400861606272)) and a radioactive isotope that had a 50-50 chance of decaying, which would kill the cat. The point was that, according to quantum physics^([5](ch06.html#idm45400861605424)) until someone opens the box to check on the cat, both states - alive and dead - exist at the same time. Meaning that the cat is both alive and dead at the same time.

This also means that if Herr Schrödinger were to send his cat/isotope box in the post to a friend^([6](ch06.html#idm45400861604272)) they have a box that could contain one of two states inside, and until they open it, they don’t know which. Of course, the postal service being what it is, chances are the cat would dead upon arrival *either way*. This is why you really shouldn’t try this one at home. Trust me I’m not a Doctor, nor do I play one on TV.

That’s kind’ve how a discriminated union is. A single returned value, but which may exist in two or more states. You don’t know which until you examine it.

If a class doesn’t care which state, you can even pass it along to its next destination unopened.

Schrödinger’s cat as code might look like this:

[PRE7]

I’m hoping you’re now clear on what exactly Discriminated Unions actually **are**. I’m going to spend the rest of this chapter demonstrating a few examples of what they are **for**.

# Naming Conventions

Let’s imagine a code module for writing out people’s names from the individual components. If you have a traditional British name, like my own, then this is fairly straightforward. A class to write a name like mine would look something like this:

[PRE8]

The code to render it would be as simple as this:

[PRE9]

All done, right? Well, this works for traditional British names, but what about Chinese names? They aren’t written in the same order as British names. Chinese names are written in the order <family name> <given name>, and many Chinese people take a “courtesy name” - a western-style name, which is used professionally.

Let’s take the example of the legendary actor, directory, writer, stunt-man, singer & all-round awesome Human being - Jackie Chan. His real name is Fang Shilong. In that set of names, his family name (i.e. Surname) is Fang. His personal name (often in English called the First name, or Christian Name) is Shilong. Jackie is a courtesy name he’s used since he was very young. This style of name doesn’t work whatsoever with the formatName function I created, above.

I *could* mangle the data a bit to make it work. Something like this:

[PRE10]

So fine, this correctly writes his two official names in the correct order. What about his courtesy name though? There’s nothing to write that out. Also, The Chinese equivalent of “Mr” - Xiānsheng ^([7](ch06.html#idm45400861305600)) - actually goes **after** the name, so this is really pretty shoddy - even if we try re-purposing the existing fields.

We could add an awful lot of `if` statements into the code to check for the nationality of the person being described, but that approach would rapidly turn into a nightmare if we tried to scale it up to include more than 2 nationalities.

Once again, a better approach would be to use a discriminated union to represent the radically different data structures in a form that mirrors the reality of the thing they’re trying to represent.

[PRE11]

In my imaginary scenario there are probably separate data sources for each name type - each with their own schema. Maybe a Web API for each country?

Using this union, we can actually create an array of names containing both me and Jackie Chan^([8](ch06.html#idm45400861328768))

[PRE12]

I coud then extend out my formatting function with a pattern matching expression:

[PRE13]

This same principle can be applied to any style of naming for anywhere in the world, and the names given to fields will always be meaningful to that country, as well as always being correctly styled without re-purposing existing fields.

# Database Lookup

The sort of area of a system I’d often consider using Discriminated Unions in C# is as return types from functions defined on an interface.

One area I’m especially likely to use this technique is in lookup functions to data sources. Let’s imagine for a moment you wanted to find someone’s details in a system of some kind somewhere. The function is going to take an integer Id value, and return a Person record.

At least that’s what you’d often find people doing. Something like this:

[PRE14]

But if you think about it, returning a Person object is only *one* of the possible return states of the function.

What if an Id is entered for a person that doesn’t exist? You *could* return `Null` I suppose, but that’s not descriptive of what actually happened. What if there were a handled `Exception` that resulted in nothing being returned? The `Null` doesn’t tell you *why* it was returned.

The other possibility is an `Exception` being raised. It might well not be the fault of your code, but nevertheless it could happen if there are network issues, or whatever. What would you return in this case?

Rather than returning an unexplained `Null` and forcing other parts of the codebase to handle it, or an alternative return type object with metadata fields in it containing Exceptions, etc. we could instead create a discriminated union:

[PRE15]

All of this means we can now return a single class from our GetPersonById function which tells the code utilizing the class that one of these three states has been returned, but which it is has already been determined. There’s no need for the returned object to have logic applied to it, to determine whether it worked or not, and the states are completely descriptive of each case that needs to be handled.

The function would look something like this:

[PRE16]

And consuming it is once again a matter of using a pattern matching expression to determine what to do:

[PRE17]

# Sending Email

The last example was fine for cases where you’re expecting a value back, but what about cases where there’s no return value? Let’s imagine I’d written some code to send an email to a customer, or family member I can’t be bothered to write a message to myself^([9](ch06.html#idm45400860602016)).

I don’t expect anything back, but I might like to know if an error has occured, so this time there are only two states I’m especially concerned with.

This is how I’d accomplish it:

[PRE18]

Use of this class in code might look like this:

[PRE19]

Usage of the function elsewhere in the codebase would look like this:

[PRE20]

This, once again, means I can return an error message and failure state from the function, but without anything anywhere depending on properties it doesn’t need.

# Console Input

Some time ago I came up with the mad idea to try out my functional programming skills by converting an old text-based game written in HP Timeshare BASIC to functional-style C#.

The game was called Oregon Trail, and dated all the way back to 1975\. Hard as it is to believe, even older than I am! Older even, than Star Wars. In fact, it even predates monitors, and had to effectively be played on something that looked like a typewriter. In those days, when the code said “print” - it meant it!

One of the most crucial things the game code had to do was to periodically take input from the user. Most of the time an integer was required - either to select a command from a list, or to enter an amount of goods to purchase. Other times, it was important to receive text, and to confirm what it was the user typed - such as in the hunting mini-game, where the user was required to type “BANG” as quickly as possible to simulate attempting to accurately hit a target.

I *could* have simply had a module in the codebase that returned raw user input from the console. This would mean that every place in the entire codebase that required an integer value would be required to carry out a check, then a parse to integer, before getting on with whatever logic was actually required.

A smarter idea is to use a discriminated union to represent the different states the logic of the game recognises from user input, and keep the necessary int check code in a single place.

Something like this:

[PRE21]

I’m not honestly sure what errors are possible from the console, but I don’t think it’s wise to rule it out, especially as it’s something beyond the control of our application code.

The idea here is that I’m gradually shifting from the impure area beyond the codebase, into the pure, controlled area within it. Like a multi-stage airlock.

![stages of text input](assets/textinput.png)

Speaking of the console being beyond our control…​ If we want to keep our codebase as functional as possible, then it’s best to hide it behind an interface, so that we can inject mocks during test, and push back the non-pure area of our code a little further.

Something like this:

[PRE22]

That was the most basic representation possible of an interaction with the user. That’s because that’s an area of the system with side effects, and I want to keep that as small as possible.

After that, I created another layer, but this time there was actually some logic applied to the text received from the player:

[PRE23]

This means that if I want to prompt the user for input, and guarantee that they gave me an interger, it’s now very easy to code:

[PRE24]

What I’m doing in this code block is prompting the player for input. Then, I check whether it’s the integer I expected - based on the check already done on it via a discriminated union. If it’s an integer, great. Job’s a good ‘un, return that interger.

If not, then the player needs to be prompted to try again, and I call this function again, recursively. I could add more detail in about capturing and logging any errors received, but I think this demonstrates the principle soundly enough.

Note also, that there isn’t a need for a Try/Catch in this function. That is already handled by the lower-level function.

There are many, many places this code checking for integer are needed in my Oregon Trail conversion. Imagine how much code I’ve saved myself by wrapping the integer check into the structure of the return object!

# Generic Unions

All of the discriminated unions so far are entirely situation specific. Before wrapping up this chapter, I want to discuss a few options for creating entirely generic, reusable versions of the same idea.

Firstly, let me re-iterate - we can’t have discriminated unions you can simply declare easily, on the fly, like the folks in F# can. It’s just not a thing we can do. Sorry. The best we can do is emulate it as closely as possible, with some sort of boilerplate trade-off.

Here are a couple of functional structures you can use. There are, incidentally, more advanced ways to use these coming up in the next chapter. Stay tuned for that.

## Maybe

If your intention with using a Discriminated Union is to represent that data might not have been found by a function, then the Maybe structure might be the one for you.

Implementations look like this:

[PRE25]

You’re basically using the Maybe abstract as a wrapper around another class, the actual class your function reutrns, but by wrapping it in this manner, you are signalling to the outside world that there may not necessarily be anything returned.

Here’s how you might use it for a function that returns a single object:

[PRE26]

You’d use that something like this:

[PRE27]

This doesn’t handle error situations especially well. A Nothing state at least prevents unhandled exceptions from occurring, and we are logging, but nothing useful has been passed back to the end user.

## Result

An alternative to a Maybe is a Result, which represents the possibility that a function might throw an error, instead of returning anything. It might look like this:

[PRE28]

Now, the Result version of the “Get Doctor” function looks like this:

[PRE29]

And you might consider using it, something like this:

[PRE30]

Now I’m covering the error scenario in one of the possible states of the Discriminated Union, but the burden of null checking falls to the receiving function.

## Maybe vs Result

A perfectly valid question at this point - which is better to use? Maybe or Result?

The Maybe gives a state that informs the user that no data has been found, removing the need for null checks, but effectively silently swallows errors. It’s better than an unhandled exception, but it could result in unreported bugs.

The Result handles errors elegantly, but puts a burden on the receiving function to check for null.

My personal preference? This might not be strictly within the standard definition of these structures, but I combine them into one. I usually have a 3-state Maybe - Something, Nothing, Error. That handles just about anything the codebase can throw at me.

This would be my personal solution to the problem:

[PRE31]

And I’d use it like this:

[PRE32]

This means the receiving function can now handle all three states elegantly with a pattern matching expression:

[PRE33]

I find this allows me to provide a full set of responses to any given scenario, when returning from a function that requires a connection to the cold, dark, hungry-wolf-filled world beyond my program, and easily allows a more informative response back to the end user.

Before we finish on this topic, here’s how I’d use that same structure to handle a return type of Enumerable:

[PRE34]

This allows me to handle the response from the function like this:

[PRE35]

Once again, nice and elegant, and everything has been considered. This is an approach I use all the time in my everyday coding, and I hope after reading this chapter that you will too!

## Either

Something and Result - in one form or another - now generically handle the idea of returning from a function where there’s some uncertainty as to how it might behave. What about scenarios where you might want to return two or more entirely different types?

This is where the Either type comes in. The syntax isn’t the nicest, but it does work.

[PRE36]

I could use it to create a type that might be left or right, like this:

[PRE37]

You could of course expand this out to have three or more different possible types. I’m not entirely sure what you’d call each of them, but it’s certainly possible. There’s just a lot of awkward boilerplate, in that you have to include all of the references to the generic types in a lot of places. At least it works, though…​

# Conclusion

In this chapter we discussed Discriminated Unions. What exactly they are, how they are used, and just how incredibly powerful they are as a code feature.

Discriminated Unions can be used to massively cut down on boilerplate code, and make use of a datatype that descriptively represents all possible states of the system, in a way that strongly encourages the receiving function to handle them appropriately.

Discriminated Unions can’t be implemented quite as easily as in F#, or other functional languages, but there are possibilities in C#, at least.

In the next chapter, I’ll be looking into some more advanced functional concepts, which will take Discriminated Unions up to the next level!

^([1](ch06.html#idm45400862303440-marker)) let me please re-assure everyone that despite being called “discriminated unions” they bear no connection to anyone’s view of love and/or marriage or to worker’s organizations.

^([2](ch06.html#idm45400862223056-marker)) Didn’t I tell you? We’re in the travel business now, you and I! Togther we’ll flog cheap holidays to unsuspecting punters until we retire rich and contented. That, or carry on doing what we’re doing now. Either way.

^([3](ch06.html#idm45400862084240-marker)) It’s not London Bridge, that famous one you’re thinking of. London Bridge is elsewhere. In Arizona, in fact. No, really. Look it up

^([4](ch06.html#idm45400861606272-marker)) N.B - no-one has ever done this. I’m not aware of a single cat ever being sacrificed in the name of quantum mechanics

^([5](ch06.html#idm45400861605424-marker)) Somehow. I’ve never really understood this part of it.

^([6](ch06.html#idm45400861604272-marker)) Wow. What a horrible birthday present that would be. Thanks, Schrödinger!

^([7](ch06.html#idm45400861305600-marker)) “先生” - It literally means “one who was born earlier”. Interestingly, if you were to write the same letters in Japanese, it would be pronounced “Sensei”. I’m a nerd - I love stuff like this!

^([8](ch06.html#idm45400861328768-marker)) Sadly the closest I’ll ever get to him for real. Do watch some of his Hong-Kong films, if you haven’t already! I’d start with the Police Story series.

^([9](ch06.html#idm45400860602016-marker)) Just kidding, folks, honest! Please don’t take me off your Christmas card lists!