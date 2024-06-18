# Chapter 10\. Memoization

There are more advantages to pure functions than just producing predictable results. Granted that’s a good thing to have, but there’s another way to use that behavior to our advantage.

Memoization is somewhat like caching, specifically the `GetOrAdd` function from `MemoryCache`. What this does is takes a key value of some kind, and if that key is already present in the cache, it returns the object out. If it isn’t, then you need to pass in a function that will generate the required value.

Memoization works to the exact same principle, except its scope might not extend beyond a single calculation.

Where this is useful is in a multi-step calculation of some kind that might be recursive, or involve the same calculations being performed multiple times for some reason.

Maybe the best way to explain this is with an example…​

# Bacon Numbers

Ever wanted an entertaining way to waste an afternoon or two? Have a look into Bacon Numbers. It’s based on the idea that Kevin Bacon is the center of the acting universe, connecting all actors together. Like all roads lead to Rome, all actors somehow connect at some level to Kevin Bacon^([1](ch10.html#idm45400847828272)). An actor’s Bacon number is the number of film connections you have to work through in order to reach Kevin Bacon. Let’s work through a few examples:

**Kevin Bacon**: easy. Bacon number of 0, because he **is** the Big Bacon himself.

**Tom Hanks**: Bacon number of 1\. He was with KB in one of my personal favorites, *Apollo 13*. Frequent Tom Hanks collaborator **Meg Ryan** is also 1, because she appeared with KB in *In The Cut*.

**David Tennant**: Bacon number of 2\. He appeared with **Colin Firth** in *St. Trinian’s 2*. Colin Firth appeared in *Where the Truth Lies* with Kevin Bacon. That’s 2 films before we find a connection, so a Bacon number of 2\. Believe it or not **Marilyn Monroe** also has a score of 2 due to KB appearing in *JFK* with **Jack Lemon**, who was also in *Some Like it Hot*.

Bollywood superstar **Aamir Khan** has a bacon number of 3\. He was with living legend **Amitabh Bacchhan** in *Bollywood Talkies*. Amitabh was in *The Great Gatsby* with **Toby McGuire**. Toby McGuire was in *Beyond all Boundaries* with Kevin Bacon.

My Bacon number is Infinity! This is because I’ve never appeared in a film as an actor^([2](ch10.html#idm45400847816624)). Also, the holder of the highest Bacon number I’m aware of is **William Rufus Shafter**, an American Civil War general, who also appeared in a non-fiction film made in 1989, which still secures him a Bacon Number. It’s a whopping great 10!!

Right, hopefully you understand the rules.

Let’s imagine you wanted to work out which out of these actors had the lowest Bacon Number programatically. Something like this:

[PRE0]

There are any numbers of ways that GetBaconNumber could be calculated. Most likely using a web API of flim data of some kind. There are more advanced “shortest path” algorithms out there, but for the sake of simplicity I’ll say it’s something like this:

1.  Get all of Kevin Bacon’s films. Assign all actors in these films a bacon number of 1\. If the target actor (e.g. Tom Hanks) is among them, return an answer of 1\. Otherwise continue

2.  Take each of the actors from the previous step (excluding Kevin Bacon himself), get a list of all of their flims not already checked. Assign all actors from these films not already assigned a number a value of 2.

3.  And so on in iterations, with each set of actors being assigned progressively higher values until we finally reach the target actor and return their number.

Since there’s an API at work to calculate these numbers, every actor whose filmography we download, or every film whose cast list we download, all have a significant cost in processing time.

Further to that, there is an awful lot of overlap between these actors and their films, so unless we step in and do something, we’re going to be checking the same film multiple times.

One option is to create a state object to pass into an Aggregate function of some kind. It’s an indefinite loop, so we’d also need to select one of the options for compromising on the functional principles and allowing a loop of this kind.

It might look something like this (N.B. I’ve made up the web API, so don’t expect this to work in a real application):

[PRE1]

This is fine, but could be better. There’s a lot of boilerplate code that’s concerned with tracking whether or not an actor has already been checked or not. For instance, there are a lot of uses of Distinct.

With Memoization we get a generic version of the check, like a cache, but it exists within the scope of the calculation we’re performing, and doesn’t persist beyond it. If you do want the saved, calculated value to persist between calls to this function, then `MemoryCache` maybe a better choice.

I could create a Memoized function to get a list of films the aactors I listed above have been in, like this:

[PRE2]

I called the same function there 4 times, and by rights it should have gone away to the film data respository and fetched a fresh copy of the data 4 times. In fact it did it only a single line when `kb1` was populated. Every time since then a copy of the same data was returned.

Note also, by the way, that the Memoized version and original version are on separate lines. That’s a limitation of C#. You can’t call an extension method on a function, only on a `Func` delegate, and the arrow function isn’t a `Func` until it has been stored in a variable.

Some functional languages have out-of-the-box support for Memoization, but F# doesn’t, oddly enough.

This is an updated version of the Bacon Number calculation, this time taking advantage of the Memoization feature:

[PRE3]

Literally the only difference here is that we’ve created a local version of the call to get film data from a remote resource, then memoized it, and thereafter referenced only the memoized version. This means that there are guaranteed to be no wasteful repeat requests for data.

# Implementing Memoization in C#

Now that you understand some of the basics, this is how you’d make a memoization function for a simple, single parameter function:

[PRE4]

This version of Memoize expects the “live data” function has only a single parameter of any type. To expect more parameters, further Memoize extension methods would be necessary, like this:

[PRE5]

Now, to make this work, I’ve assumed that `ToString()` returns something meaningful, meaning that most likely it’ll have to be a primitive type (like a `string` or `int`) in order to work. The `ToString()` method on a class tends to simply return a description of the *Type* of the class, not its properties.

If you absolutely have to memoize classes as parameters, then some creativity is needed. The easiest way to keep it generic is probably to add parameters to the `Memoize` function which require the developer to provide a custom `ToString` function. Like this:

[PRE6]

you might call it like this, in that case:

[PRE7]

This is only possible though, if you keep your functions *Pure*. If there are side effects of any kind in your “live” func, then you might not necessariy get the results you expect. Depending on what those side effects are.

From a practical perspective, I wouldn’t be worried about adding logging into the “live” function, but I might be concerned if there were properties that were expected to be unique in every instance of the generated class.

There may be cases that you *want* the results to persist between calls to the `Memoize` function, in which case you’ll also need to add a MemoryCache parameter and pass in an instance from the outside world. I’m not convinced there are many circumstances where that’s a good idea, though.

# Conclusion

In this chapter we looked at Memoization, what it is and how to implement it. It’s a lightweight alternative to caching which can be used to drastically reduce the amount of time taken to complete a complex calculation with a lot of repetitive elements.

That’s it for theory now! This isn’t just the end of this chapter, but also of this entire part of the book.

Part one was a look at how to use functional ideas and out-of-the-box C# to improve your daily coding. Part two was a deeper dive into the actual theory behind Functional Programming and how to implement it with a little creative hacking about. Part three is going to be a little more philosophical, and to give you a few hints as to where to go next with what you’ve learned here with me.

Turn over, if you dare, to enter…​Part Three…​

^([1](ch10.html#idm45400847828272-marker)) It probbaly isn’t true, sorry Mr. Bacon. I still love you, though!

^([2](ch10.html#idm45400847816624-marker)) I wouldn’t say no, though. Anyone know a film director wanting to cast an aging, overweight, British tech-nerd? I probably couldn’t play James Bond, but I’m willing to give it a go!