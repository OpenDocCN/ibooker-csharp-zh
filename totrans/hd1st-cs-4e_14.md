# Chapter 9\. LINQ and lambdas: *Get control of your data*

![Images](assets/pg467.png)

**It’s a data-driven world...we all need to know how to live in it.**

Gone are the days when you could program for days, even weeks, without dealing with **loads of data**. Today, ***everything is about data***, and that’s where **LINQ** comes in. LINQ is a feature of C# and .NET that not only lets you **query data** in your .NET collections in an intuitive way, but lets you **group data** and **merge data from different data sources**. You’ll add **unit tests** to make sure your code is working the way you want. Once you’ve got the hang of wrangling your data into manageable chunks, you can use **lambda expressions** to refactor your C# code to make it even more expressive

# Jimmy’s a Captain Amazing super-fan...

Meet Jimmy, one of the most enthusiastic collectors of Captain Amazing comics, graphic novels, and paraphernalia. He knows all the Captain trivia, he’s got props from all the movies, and he’s got a comic collection that can only be described as, well, amazing.

![Images](assets/pg468.png)

# ...but his collection’s all over the place

Jimmy may be passionate, but he’s not exactly organized. He’s trying to keep track of the most prized “crown jewel” comics of his collection, but he needs help. Can you build Jimmy an app to manage his comics?

![Images](assets/pg469.png)

# Use LINQ to query your collections

In this chapter you’ll learn about **LINQ** (or **Language-**Integrated** Query**). LINQ combines really useful classes and methods with powerful features built directly into C#, all created to help you work with sequences of data—like Jimmy’s comic book collection.

Let’s use Visual Studio to start exploring LINQ. **Create a new Console App** (.NET Core) project and give it the name ***LinqTest***. Add this code, and when you get to the last line **add a period** and look at the IntelliSense window:

![Images](assets/pg470.png)

Let’s use some of those new methods to finish your console app:

[PRE0]

Now run your app. It prints this line of text to the console:

[PRE1]

So what did you just do?

> **LINQ (or Language INtegrated Query) is a combination of C# features and .NET classes that helps you work with sequences of data.**

# LINQ works with any IEnumerable<T>

When you added the `using System.Linq;` directive to your code, your List<int> suddenly got “superpowered”—a bunch of LINQ methods repeated appeared on it. You can do the same thing for ***any*** ***class that implements IEnumerable<T>.***

When a class implements IEnumerable<T>, any instance of that class is a **sequence:**

*   The list of numbers from 1 to 99 was a sequence.

*   When you called its Take method, it returned a reference to a sequence that contained the first five elements.

*   When you called its TakeLast method, it returned another five-element sequence.

*   And when you used Concat to combine the two five-element sequences, it created a new sequence with 10 elements and returned a reference to that new sequence.

> **Any time you have an object that implements the IEnumerable interface, you have a sequence that you can use with LINQ. Doing an operation on that sequence in order is called enumerating the sequence.**

## LINQ methods enumerate your sequences

You already know that `foreach` loops work with IEnumerable objects. Think about what a `foreach` loop does:

![Images](assets/pg472.png)

When a method goes through each item in a sequence in order, that’s called **enumerating** the sequence. And that’s how LINQ methods work.

###### Note

**Objects that implement the IEnumerable interface can be enumerated. That’s the job objects that implement the IEnumerable interface do.**

###### Note

**What if you want to find the first 30 issues in Jimmy’s collection starting with issue #118? LINQ provides a really useful method to help with that. The static Enumerable.Range method generates a sequence of integers. Calling Enumerable.Range(8, 5) returns a sequence of 5 numbers starting with 8: 8, 9, 10, 11, 12.**

![Images](assets/pg473-1.png)

###### Note

**The LINQ methods in this exercise have names that make it obvious what they do. Some LINQ methods, like Sum, Min, Max, Count, First, and Last, return a single value. The Sum method adds up the values in the sequence. The Average method returns their average value. The Min and Max methods return the smallest and largest values in the sequence. The First and Last methods do exactly what it sounds like they do.**

**Other LINQ methods, like Take, TakeLast, Concat, Reverse (which reverses the order in a sequence), and Skip (which skips the first elements in a sequence), return another sequence.**

# LINQ’s query syntax

The LINQ methods you’ve seen so far may not be enough on their own to answer the kinds of questions about data that we might have—or the questions that Jimmy has about his comic collection.

And that’s where the **LINQ declarative query syntax** comes in. It uses special keywords—including `where`, `select`, `groupby`, and `join`—to build **queries** directly into your code.

## LINQ queries are built from clauses

Let’s build a query that finds the numbers in an int array that are under 37 and puts those numbers in ascending order. It does that using four **clauses** that tell it what object to query how to determine which of its members to select, how to sort the results, and how the results should be returned.

###### Note

**LINQ queries work on sequences, or objects that implement IEnumerable<T>. LINQ queries start with a from clause:**

[PRE2]

**It tells the query what sequence to execute against, and assigns a name to use for each element being queried. It’s like the first line in a `foreach loop`: it declares a variable to use while iterating through the sequence that’s assigned each element in the sequence. So this:**

[PRE3]

**iterates through each element in the `values` array in order, assigning the first value in the array to v, then the second, then the third, etc.**

![Images](assets/pg475.png)![Images](assets/pg476-2.png)

**LINQ isn’t just for numbers. It works with objects, too.**

When Jimmy looks at stacks and stacks of disorganized comics, he might see paper, ink, and a jumbled mess. When us developers look at them, we see something else: **lots and lots of data** just waiting to be organized. How do we organize comic book data in C#? The same way we organize playing cards, bees, or items on Sloppy Joe’s menu: we create a class, then we use a collection to manage that class. So all we need to help Jimmy out is a Comic class, and code to help us bring some sanity to his collection. And LINQ will help!

# LINQ works with objects

Jimmy wanted to know how much some of his prize comics are worth, so he hired a professional comic book appraiser to give him prices for each of them. It turns out that some of his comics are worth a lot of money! Let’s use collections to manage that data for him.

***Do this!***

1.  **Create a new console app and add a Comic class.**

    Use two automatic properties for the name and issue number:

    ![Images](assets/pg477-1.png)
2.  **Add a List that contains Jimmy’s catalog.**

    Add this static Catalog field to the Comic class. It returns a sequence with Jimmy’s prized comics:

    ![Images](assets/pg477-2.png)
3.  **Use a Dictionary to manage the prices.**

    Add a static Comic.Prices field—it’s a Dictionary<int, decimal> that lets you look up the price of each comic using its issue number (using the collection initializer syntax for dictionaries that you learned in [#enums_and_collections_organizing_your_da](ch08.html#enums_and_collections_organizing_your_da)). Note that we’re using the **IReadOnlyDictionary interface** for encapsulation—it’s an interface that includes only the methods to read values (so we don’t accidentally change the prices):

    ![Images](assets/pg477-3.png)

    ###### Note

    **We used a Dictionary to store the prices for the comics. We could have included a property called Price. We decided to keep information about the comic and price separate. We did this because prices for collectors’ items change all the time, but the name and issue number will always be the same. Do you think we made the right choice?**

# Use a LINQ query to finish the app for Jimmy

We used the LINQ declarative query syntax earlier to create a query with four clauses: a `from` clause to create a range variable, a `where` clause to include only numbers under 37, an `orderby` clause to sort them in descending order, and a `select` clause to determine which elements to include in the resulting sequence.

Let’s **add a LINQ query to the Main method** that works exactly the same way—except using Comic objects instead of int values, so it writes a list of comics with a price > 500 in reverse order to the console. We’ll start with two `using` declarations so we can use IEnumerable<T> and LINQ methods. The query will return an IEnumerable<Comic>, then use a `foreach` loop to iterate through it and write the output.

4.  **Modify the Main method to use a LINQ query.**

    Here’s the entire Program class, including the `using` directives that you needed to add to the top:

    ![Images](assets/pg478-1.png)

    **Output:**

    [PRE4]

5.  **Use the descending keyword to make your orderby clause more readable.**

    Your `orderby` clause uses a minus sign to negate the comic prices before sorting, causing the query to sort them in descending order. But it’s easy to accidentally miss that minus sign when you’re reading the code and trying to figure out how it works. Luckily, there’s another way to get the same results. Remove the minus sign and **add the `descending` keyword** to the end of the clause:

    ![Images](assets/pg478-2.png)

# The var keyword lets C# figure out variable types for you

We just saw that when we made the small change to the `select` clause, the type of sequence that the query returned changed. When it was `select comic;` the return type was IEnumerable<Comic>. When we changed it to `select $"{comic} is worth {Comic.Prices[comic.Issue]:c}";` the return type changed to IEnumerable<string>. When you’re working with LINQ, that happens all the time—you’re constantly tweaking your queries. It’s not always obvious exactly what type they return. Sometimes going back and updating all of your declarations can get annoying.

Luckily, C# gives us a really useful tool to help keep variable declarations simple and readable. You can replace any variable declaration with the `var` **keyword**. So you can replace any of these declarations:

[PRE5]

###### Note

When you use the var keyword, you’re telling C# to use an implicitly typed variable. We saw that same word—implicit—back in [#enums_and_collections_organizing_your_da](ch08.html#enums_and_collections_organizing_your_da) when we talked about covariance. It means that C# figures out the types on its own.

with these declarations, which do exactly the same thing:

[PRE6]

And you don’t have to change any of your code. Just replace the types with var and everything works.

## When you use var, C# figures out the variable’s type automatically

Go ahead—try it right now. Comment out the first line of the LINQ query you just wrote, then replace IEnumerable<Comic> with var:

![Images](assets/pg480.png)

The IDE figured out the mostExpensive variable’s type—and it’s a type we haven’t seen before. Remember in [#interfacescomma_castingcomma_and_quotati](ch07.html#interfacescomma_castingcomma_and_quotati) when we talked about how interfaces can extend other interfaces? The IOrderedEnumerable interface is part of LINQ—it’s used to represent a *sorted* sequence—and it extends the IEnumerable<T> interface. Try commenting out the `orderby` clause and hovering over the mostExpensive variable—you’ll find that it turns into an IEnumerable<Comic>. That’s because C# looks at the code to ***figure out the type of any variable you declare with*** ***var***.

![Images](assets/pg483.png)

**You really can use var in your variable declarations.**

And yes, it really is that simple. A lot of C# developers declare local variables using var almost all the time, and include the type only when it makes the code easier to read. As long as you’re declaring the variable and initializing it in the same statement, you can use var.

But there are some important restrictions on using var. For example:

*   You can only declare one variable at a time with var.

*   You can’t use the variable you’re declaring in the declaration.

*   You can’t declare it equal to `null`.

If you create a variable named `var`, you won’t be able to use it as a keyword anymore:

*   You definitely can’t use var to declare a field or a property—you can only use it as a local variable inside a method.

*   If you stick to those ground rules, you can use var pretty much anywhere.

**So when you did this in [#types_and_references_getting_the_referen](ch04.html#types_and_references_getting_the_referen):**

[PRE7]

**Or this in [#inheritance_your_objectapostrophes_famil](ch06.html#inheritance_your_objectapostrophes_famil):**

[PRE8]

**Or this in [#enums_and_collections_organizing_your_da](ch08.html#enums_and_collections_organizing_your_da):**

[PRE9]

**You could have done this:**

[PRE10]

**Or this:**

[PRE11]

**Or this:**

[PRE12]

**...and your code would have worked exactly the same.**

**But you can’t use var to declare a field or property:**

[PRE13]

# LINQ is versatile

With LINQ, you can do a lot more than just pull a few items out of a collection. You can modify the items before you return them. Once you’ve generated a set of result sequences, LINQ gives you a bunch of methods that work with them. Top to bottom, LINQ gives you the tools you need to manage your data. Let’s do a quick review of some of the LINQ features that we’ve already seen.

*   **Modify every item returned from the query.**

    This code will add a string onto the end of each element in an array of strings. It doesn’t change the array itself—it creates a new sequence of modified strings.

    ![Images](assets/pg486-1.png)
*   **Perform calculations on sequences.**

    You can use the LINQ methods on their own to get statistics about a sequence of numbers.

    ###### Note

    **The static String.Join method concatenates all of the items in a sequence into a string, specifying the separator to use between them.**

    [PRE14]

    ![Images](assets/pg486-2.png)

# LINQ queries aren’t run until you access their results

When you include a LINQ query in your code, it uses **deferred evaluation** (sometimes called lazy evaluation). That means the LINQ query doesn’t actually do any enumerating or looping until your code executes a statement that ***uses the results of the query***. That sounds a little weird, but it makes a lot more sense when you see it in action. **Create a new console app** and add this code:

***Do this!***

![Images](assets/pg487-1.png)

###### Note

**Did you get a weird compiler error? Make sure you added the two using directives to your code!**

[PRE15]

Now run your app. Notice how the Console.WriteLine that prints `"Set up the query"` runs ***before*** the get accessor ever executes. That’s because the LINQ query won’t get executed until the `foreach` loop.

![Images](assets/pg487-2.png)

If you need the query to execute right now, you can force **immediate execution** by calling a LINQ method that needs to enumerate the entire list—for example, the ToList method, which turns it into a List<T>. Add this line, and change the foreach to use the new List:

[PRE16]

Run the app again. This time you’ll see the get accessors called before the `foreach` loop starts executing—which makes sense, because ToList needs to access every element in the sequence to convert it to a List. Methods like Sum, Min, and Max also need to access every element in the sequence, so when you use them you’ll force LINQ to do immediate execution as well.

# Use a group query to separate your sequence into groups

Sometimes you really want to slice and dice your data. For example, Jimmy might want to group his comics by the decade they were published. Or maybe he wants to separate them by price (cheap ones in one collection, expensive ones in another). There are lots of reasons you might want to group your data together. That’s where the **LINQ group query** comes in handy.

***Group this!***

![Images](assets/pg488.png)

1.  **Create a new console app and add the card classes and enums.**

    Create a **new .NET Core console app named *CardLinq***. Then go to the Solution Explorer panel, right-click on the project name, and choose Add >> Existing Items (or Add >>Existing Files on a Mac). Navigate to the folder where you saved the Two Decks project from [#enums_and_collections_organizing_your_da](ch08.html#enums_and_collections_organizing_your_da). Add the files with the **Suit and Value enums**, then add the **Deck, Card, and CardComparerByValue classes**.

    ###### Note

    **You created the Deck class in the downloadable “Two Decks” project at the end of [#enums_and_collections_organizing_your_da](ch08.html#enums_and_collections_organizing_your_da).**

    Make sure you **modify the namespace in each file you added** to match the namespace in *Program.cs* so your Main method can access the classes you added.

2.  **Make your Card class sortable with IComparable<T>.**

    We’ll be using a LINQ `orderby` clause to sort groups, so we need the Card class to be sortable. Luckily, this works exactly like the List.Sort method, which you learned about in [#interfacescomma_castingcomma_and_quotati](ch07.html#interfacescomma_castingcomma_and_quotati). Modify your Card class to **extend the IComparable interface**:

    ![Images](assets/pg487-2a.png)
3.  **Modify the Deck.Shuffle method to support method chaining.**

    The Shuffle class shuffles the deck. All you need to do to make it support method chaining is modify it to return a reference to the Deck instance that just got shuffled:

    ![Images](assets/pg489-1.png)
4.  **Use a LINQ query with group...by clause to group the cards by suit.**

    The Main method will get 16 random cards by shuffling the deck, then using the LINQ Take method to pull the first 16 cards. Then it will use a LINQ query with a `group...by` **clause** to separate the deck into smaller sequences, with one sequence for each suit in the 16 cards:

    ![Images](assets/pg489-2.png)

###### Note

**The group...by clause creates a sequence of groups that implement the IGrouping interface. IGrouping extends IEnumerable and adds exactly one member: a property called Key. So each group is a sequence of other sequences—in this case, it’s a group of Card sequences, where the key is the suit of the card (from the `Suits` enum). The full type of each group is IGrouping<Suits, Card>, which means it’s a sequence of Card sequences, each of which has a Suits value as its key.**

# Use join queries to merge data from two sequences

Every good collector knows that critical reviews can have a big impact on values. Jimmy’s been keeping track of reviewer scores from two big comic review aggregators, MuddyCritic and Rotten Tornadoes. Now he needs to match them up to his collection. How’s he going to do that?

LINQ to the rescue! Its `join` keyword lets you **combine data from two sequences** using a single query. It does it by comparing items in one sequence with their matching items in a second sequence. (LINQ is smart enough to do this efficiently—it doesn’t actually compare every pair of items unless it has to.) The final result combines every pair that matches.

1.  You’ll start your query with the usual `from` clause. But instead of following it up with the criteria to determine what goes into the results, you’ll add:

    [PRE17]

    The `join` clause tells LINQ to enumerate both sequences to match up pairs with one member from each. It assigns `name` to the member it’ll pull out of the joined collection in each iteration. You’ll use that name in the `where` clause.

    ![Images](assets/pg491.png)
2.  Next you’ll add the `**on**` clause, which tells LINQ how to match the two collections together. You’ll follow it with the name of the member of the first collection you’re matching, followed by `**equals**` and the name of the member of the second collection to match it up with.

3.  You’ll continue the LINQ query with `**where**` and `**orderby**` clauses as usual. You could finish it with a normal `**select**` clause, but you usually want to return results that pull some data from one collection and other data from the other.

The result is a sequence of objects that have Name and Issue properties from Comic, but Critic and Score properties from Review. The result can’t be a sequence of Comic objects, but it also can’t be a sequence of Review objects, because neither class has all of those properties. The result is a different kind of type: ***an anonymous type.***

# Use the new keyword to create anonymous types

You’ve been using the `new` keyword since [#objectshellipget_orientedexclamation_mar](ch03.html#objectshellipget_orientedexclamation_mar) to create instances of objects. Every time you use it, you include a type (so the statement `new Guy();` creates an instance of the type Guy). You can also use the `new` keyword without a type to create an **anonymous type**. That’s a perfectly valid type that has read-only properties, but doesn’t have a name. The type we returned from the query that joined Jimmy’s comics to reviews is an anonymous type. You can add properties to your anonymous type by using an object initializer. Here’s what that looks like:

[PRE18]

Try pasting that into a new console app and running it. You’ll see this output:

[PRE19]

Now hover over `whatAmI` in the IDE and have a look at the IntelliSense window:

![Images](assets/pg492-2a.png)

###### Note

**The LINQ query we just saw that joins Jimmy’s comics with reviews returns an anonymous type. You’ll add that query to an app later in the chapter.**

The `whatAmI` variable is a reference type, just like any other reference. It points to an object on the heap, and you can use it to access that object’s members—in this case, two of its properties:

[PRE20]

Besides the fact that they don’t have names, anonymous types are just like any other types.

![Images](assets/pg492-2.png)

**Right! You use var to declare anonymous types.**

In fact, that’s one of the most important uses of the `var` keyword.

![Images](assets/pg499.png)

# Unit tests help you make sure your code works

We intentionally left a bug in the code we gave you...but is that the *only* bug in the app? It’s easy to write code that doesn’t do exactly what you intended it to do. Luckily, there’s a way for us to find bugs so we can fix them. **Unit tests** are *automated* tests that help you make sure your code does what it’s supposed to do. Each unit test is a method that makes sure that a specific part of the code (the “unit” being tested) works. If the method runs without throwing an exception, it passes. If it throws an exception, it fails. Most large programs have a **suite** of tests that cover most or all of the code.

Visual Studio has built-in unit testing tools to help you write your tests and track which ones pass or fail. The unit tests in this book will use **MSTest**, a unit test framework (which means that it’s a set of classes that give you the tools to write unit tests) developed by Microsoft.

###### Note

Visual Studio also supports unit tests written in NUnit and xUnit, two popular open source unit test frameworks for C# and .NET code.

## Visual Studio for Windows has a Test Explorer window

Open the Test Explorer window by choosing ***View >> Test Explorer*** from the main menu bar. It shows you the unit tests on the left, and the results of the most recent run on the right. The toolbar has buttons to run all tests, run a single test, and repeat the last run.

![Images](assets/pg500-1.png)

###### Note

**When you add unit tests to your solution, you can **`run`** your tests by clicking the Run All Tests button. You can **`debug`** your unit tests on Windows by choosing Tests >> Debug all tests, and on Mac by clicking Debug All Tests in the Unit Tests tool window.**

![Images](assets/pg500-2.png)

## Visual Studio for Mac has the Unit Tests tool window

Open the Unit Test pad by choosing ***View >> Tool Windows >> Unit Tests*** from the menu bar. It has buttons to run or debug your tests. When you run the unit tests, the IDE displays the results in a Test Results tool window (usually at the bottom of the IDE window).

![Images](assets/pg500-3.png)

###### Note

Back in [#objectshellipget_orientedexclamation_mar](ch03.html#objectshellipget_orientedexclamation_mar) you learned about prototypes, or early versions of games that you can play, test, learn from, and improve—and you saw how that idea can work with any kind of project, not just games. The same thing applies to testing. Sometimes the idea of testing software can seem a little abstract. Thinking about how game developers test their games can help us get used to the idea, and can make the concept of testing feel more intuitive.

###### Note

Most development teams try to automate as much of their testing as possible, and run those tests before each commit. That way, they know if they inadvertently introduced a bug when they were making a fix or adding a new feature.

# Add a unit test project to Jimmy’s comic collection app

1.  **Add a new MSTest (.NET Core) project.**

    Right-click on the solution name in the Solution Explorer, then choose **Add >> New Project...** from the menubar. Make sure it’s an **MSTest Test Project (.NET Core)**: on Windows use the “Search for Templates” box to search for MSTest; on macOS select Tests under “Web and Console” to see the project template. Name your project *JimmyLinqUnitTests*.

2.  **Add a dependency on your existing project.**

    You’ll be building unit tests for the ComicAnalyzer class. When you have two different projects in the same solution, they’re *independent*—by default, the classes in one project can’t use classes in another project—so you’ll need to set up a dependency to let your unit tests use ComicAnalyzer.

    Expand the JimmyLinqUnitTests project in the Solution Explorer, then right-click on Dependencies and choose **Add Reference...** from the menu. Check the JimmyLinq project that you created for the exercise.

    ![Images](assets/pg502-1.png)
3.  **Make your ComicAnalyzer class public.**

    When Visual Studio added the unit test project, it created a class called UnitTest1\. Edit the *UnitTest1.cs* file and try adding the `using JimmyLinq;` directive inside the namespace:

    ![Images](assets/pg502-2.png)

    Hmm, something’s wrong—the IDE won’t let you add the directive. The reason is that the JimmyLinq project has no public classes, enums, or other members. Try modifying the `Critics` enum to make it public (`**public enum Critics**`), then go back and try adding the `using` directive. Now you can add it! The IDE saw that the JimmyLinq namespace has public members, and added it to the pop-up window.

    Now change the ComicAnalyzer declaration to make it public: `**public static class ComicAnalyzer**`

    ***Uh-oh—something’s wrong.*** Did you get a bunch of “Inconsistent accessibility” compiler errors?

    ![Images](assets/pg502-3.png)

    The problem is that ComicAnalyzer is public, but it exposes members that have no access modifiers, which makes them `**internal**`—so other projects in the solution can’t see them. **Add the `public` access modifier** to ***every*** ***class and enum*** in the JimmyLinq project. Now your solution will build again.

# Write your first unit test

The IDE added a class called UnitTest1 to your new MSTest project. Rename the class (and the file) ComicAnalyzerTests. The class contains a test method called TestMethod1\. Next, give it a very descriptive name: rename the method ComicAnalyzer_Should_Group_Comics. Here’s the code for your unit test class:

![Images](assets/pg503-1.png)

###### Note

**When you run your unit tests in the IDE, it looks for any class with `[TestClass]` above it. That’s called an attribute. A test class includes test methods, which must be marked with the `[TestMethod]` attribute.**

**MSTest unit tests use the Assert class, which has static methods that you can use to check that your code behaves the way you expect it to. This unit test uses the Assert.AreEqual method. It takes two parameters, an expected result (what you think the code should do) and an actual result (what it actually does), and throws an exception if they’re not equal.**

**This test sets up some very limited test data: a sequence of three comics and a dictionary with three prices. Then it calls GroupComicsByPrice and uses Assert.AreEqual to validate that the results match what we expect.**

Run your test by choosing **Test >> Run All Tests** (Windows) or **Run >> Run Unit Tests** (Mac) from the menubar. The IDE will pop up a Test Explorer window (Windows) or Test Results panel (Mac) with the test results:

[PRE21]

This is the result of a **failed unit test**. Look for the ![Images](assets/pg503-2.png) icon in Windows or the ![Images](assets/pg503-3.png) message at the bottom of the IDE window in Visual Studio for Mac—that’s how you see a count of your failed unit tests.

***Did you expect that unit test to fail? Can you figure out what went wrong with the test?***

# Write a unit test for the GetReviews method

The unit test for the GroupComicsByPrice method used MSTest’s static Assert.AreEqual method to check expected values against actual ones. The GetReviews method *returns a sequence of strings*, not an individual value. We *could* use Assert.AreEqual to compare individual elements in that sequence, just like we did with the last two assertions, using LINQ methods like First to get specific elements...but that would take a LOT of code.

Luckily, MSTest has a better way to compare collections: the **CollectionAssert class** has static methods for comparing expected versus actual collection results. So if you have a collection with expected results and a collection with actual results, you can compare them like this:

[PRE22]

If the expected and actual results don’t match the test will fail. Go ahead and **add this test** to validate the ComicAnalyzer.GetReviews method:

[PRE23]

Now run your tests again. You should see two unit tests pass.

# Write unit tests to handle edge cases and weird data

In the real world, data is messy. For example, we never really told you exactly what review data is supposed to look like. You’ve seen review scores between 0 and 100\. Did you assume those were the only values allowed? That’s definitely the way some review websites in the real world operate. What if we get some weird review scores—like negative ones, or really big ones, or zero? What if we get more than one score from a reviewer for an issue? Even if these things aren’t *supposed to* happen, they *might* happen.

We want our code to be **robust**, which means that it handles problems, failures, and especially bad input data well. So let’s build a unit test that passes some weird data to GetReviews and makes sure it doesn’t break:

![Images](assets/pg506.png)

###### Note

Always take time to write unit tests for edge cases and weird data—think of them as “must-have” and not “nice-to-have” tests. The point of unit testing is to cast the widest possible net to catch bugs, and these kinds of tests are really effective for that.

> **It’s really important to add unit tests that handle edge cases and weird data. They can help you spot problems in your code that you wouldn’t find otherwise.**

![Images](assets/pg507.png)

**Your projects actually go faster when you write unit tests.**

We’re serious! It may seem counterintuitive that it takes *less time* to write *more code*, but if you’re in the habit of writing unit tests, your projects go a lot more smoothly because you find and fix bugs early. You’ve written a lot of code so far in the first eight and a half chapters of this book, which means you’ve almost certainly had to track down and fix bugs in your code. When you fixed those bugs, did you have to fix other code in your project too? When we find an unexpected bug, we often have to stop what we’re doing to track it down and fix it, and switching back and forth like that—losing our train of thought, having to interrupt our flow—can really slow things down. Unit tests help you find those bugs early, before they have a chance to interrupt your work.

***Still wondering exactly when unit tests should be written? We included a downloadable project at the end of the chapter to help answer that question.***

# Use the => operator to create lambda expressions

***We left you hanging back at the beginning of the chapter.*** Remember that mysterious line we asked you to add to the Comic class? Here it is again:

[PRE24]

You’ve been using that ToString method throughout the chapter—you know it works. What would you do if we asked you to rewrite that method the way you’ve been writing methods so far? You might write something like this:

[PRE25]

And you’d basically be right. So what’s going on? What exactly is that => operator?

The => operator that you used in the ToString method is the **lambda operator**. You can use => to define a **lambda expression**, or an *anonymous function* defined within a single statement. Lambda expressions look like this:

[PRE26]

There are two parts to a lambda expression:

*   The `input-parameters` part is a list of parameters, just like you’d use when you declare a method. If there’s only one parameter, you can leave off the parentheses.

*   The `expression` is any C# expression: it can be an interpolated string, a statement that uses an operator, a method call—pretty much anything you would put in a statement.

Lambda expressions may look a little weird at first, but they’re just another way of using the ***same familiar C# expressions*** that you’ve been using throughout the book—just like the Comic.ToString method, which works the same way whether or not you use a lambda expression.

![Images](assets/pg508.png)

**Yes! You can use lambda expressions to refactor many methods and properties.**

You’ve written a lot of methods throughout this book that contain just a single statement. You could refactor most of them to use lambda expressions instead. In many cases, that could make your code easier to read and understand. Lambdas give you options—you can decide when using them improves your code.

# A lambda test drive

![Images](assets/coma.png)

Let’s kick the tires on lambda expressions, which give us a whole new way to write methods, including ones that return values or take parameters.

1.  **Create a new console app**. Add this Program class with the Main method:

    [PRE27]

    ![Images](assets/pg509-1.png)

    Run it a few times. Each time it prints a different random number, as in: `The value is 37.8709`

2.  **Refactor the GetRandomDouble and PrintValue methods** using the => operator:

    ![Images](assets/pg509-2.png)

    Run your program again—it should print a different random number, just like before.

    Before we do one more refactoring, **hover over the random field** and look at the IntelliSense pop-up:

    ![Images](assets/pg509-3.png)
3.  **Modify the random field** to use a lambda expression:

    [PRE28]

    The program still runs the same way. Now **hover over the random field** again:

    ![Images](assets/pg509-4.png)

    Wait a minute—random isn’t a field anymore. Changing it into a lambda turned it into a property! That’s because **lambda expressions always work like methods**. So when random was a field, it got instantiated once when the class was constructed. When you changed the = to a => and converted it to a lambda, it became a method—which means ***a new instance of Random is created every time the property is accessed***.

# Refactor a clown with lambdas

Back in [#interfacescomma_castingcomma_and_quotati](ch07.html#interfacescomma_castingcomma_and_quotati), you created an IClown interface with two members:

***Do this!***

![Images](assets/pg509-5a.png)

And you modified this class to implement that interface:

[PRE29]

Let’s do that same thing again—but this time we’ll use lambdas. **Create a new Console App** **project** and add the IClown interface and TallGuy class. Then modify TallGuy to implement IClown:

[PRE30]

Now open the Quick Actions menu and choose **“Implement interface.”** The IDE fills in all of the interface members, having them throw NotImplementedExceptions just like it does when you use Generate Method.

![Images](assets/pg510-2.png)

Let’s refactor these methods so they do the same thing as before, but now use lambda expressions:

[PRE31]

![Images](assets/pg510-3.png)

Now add the same Main method you used back in [#interfacescomma_castingcomma_and_quotati](ch07.html#interfacescomma_castingcomma_and_quotati):

[PRE32]

Run your app. The TallGuy class works just like it did in [#interfacescomma_castingcomma_and_quotati](ch07.html#interfacescomma_castingcomma_and_quotati), but now that we’ve refactored its members to use lambda expressions it’s more compact.

***We think the new and improved TallGuy class is easier to read. Do you?***

![Images](assets/pg510-4.png)

> **You can use the => operator to create a property with a get accessor that executes a lambda expression.**

# Use the ?: operator to make your lambdas make choices

What if you want your lambdas to do... more? It would be great if they could make decisions...and that’s where the **conditional operator** (which some people call the **ternary operator**) comes in. It works like this:

[PRE33]

which may look a little weird at first, so let’s have a look at an example. First of all, the ?: operator isn’t unique to lambdas—you can use it anywhere. Take this `if` statement from the AbilityScoreCalculator class in [#types_and_references_getting_the_referen](ch04.html#types_and_references_getting_the_referen):

![Images](assets/pg513-1.png)

Notice how we set Score equal to the results of the ?: expression. The ?: expression **returns a value**: it checks the *condition* (added < Minimum), and then it either returns the *consequent* (Minimum) or the *alternative* (added).

When you have a method that looks like that `if/else` statement, you can **use ?:** **to refactor it as a lambda**. For example, take this method from the PaintballGun class in [#encapsulation_keep_your_privateshellippr](ch05.html#encapsulation_keep_your_privateshellippr):

![Images](assets/pg513-2.png)

Let’s rewrite that as a more concise lambda expression:

Notice the slight change—in the `if/else` version, the BallsLoaded property was set inside the then- and else-statements. We changed this to use a conditional operator that checked balls against MAGAZINE_SIZE and returned the correct value, and used that return value to set the BallsLoaded property.

# Lambda expressions and LINQ

Add this small LINQ query to any C# app, then hover over the `select` keyword in the code:

![Images](assets/pg514-1.png)

The IDE pops up a tooltip window just like it does ***when you hover over a method.*** Let’s take a closer look at the first line, which shows the method declaration:

![Images](assets/pg514-2.png)

We can learn a few things from that method declaration:

*   The IEnumerable<int>.Select method returns an IEnumerable<int>.

*   It takes a single parameter of type Func<int, int>.

## Use lambda expressions with methods that take a Func parameter

When a method takes a Func<int, int> parameter, you can **call it with a lambda expression** that takes an int parameter and returns an int value. So you could refactor the select query like this:

[PRE34]

Go ahead—try that yourself in a console app. Add a foreach statement to print the output:

[PRE35]

When you print the results of the refactored query, you’ll get the sequence { 2, 4, 6, 8 }—exactly the same result as you got with the LINQ query syntax before you refactored it.

# LINQ queries can be written as chained LINQ methods

Take this LINQ query from earlier and **add it to an app** so we can explore a few more LINQ methods:

[PRE36]

## The OrderBy LINQ method sorts a sequence

Hover over the `**orderby**` keyword and take a look at its parameter:

![Images](assets/pg515-1.png)

When you use an `orderby` clause in a LINQ query, it calls a LINQ OrderBy method that sorts the sequence. In this case, we can pass it a lambda expression with an int parameter that **returns the sort key**, or any value (which must implement IComparer) that it can use to sort the results.

## The Where LINQ method pulls out a subset of a sequence

Now hover over the `**where**` keyword in the LINQ query:

![Images](assets/pg515-2.png)

The `where` clause in a LINQ query calls a LINQ Where method that can use a lambda that returns a Boolean. ***The Where method calls that lambda for each element in the sequence***. If the lambda returns true, the element is included in the results. If the lambda returns false, the element is removed.

# Use the `=>` operator to create switch expressions

You’ve been using `switch` statements since [#inheritance_your_objectapostrophes_famil](ch06.html#inheritance_your_objectapostrophes_famil) to check a variable against several options. It’s a really useful tool... but have you noticed its limitations? For example, try adding a case that tests against a variable:

[PRE37]

You’ll get a C# compiler error: *A constant value is expected.* That’s because you can only use constant values—like literals and variables defined with the `const` keyword—in the `switch` statements that you’ve been using.

But that all changes with the `=>` operator, which lets you create **switch expressions**. They’re similar to the `switch` statements that you’ve been using, but they’re *expressions* that return a value. A switch expression starts with a value to check and the `switch` keyword followed by a series of *switch arms* in curly brackets separated by commas. Each switch arm uses the => operator to check the value against an expression. If the first arm doesn’t match, it moves on to the next one, returning the value for the matching arm.

![Images](assets/pg517-1.png)

Let’s say you’re working on a card game that needs to assign a certain score based on suit, where spades are worth 6, hearts are worth 4, and other cards are worth 2\. You could write a `switch` statement like this:

![Images](assets/pg517-2.png)

The whole goal of this `switch` statement is to use the cases to set the `score` variable—and a lot of our `switch` statements work that way. We can use the => operator to create a switch expression that does the same thing:

![Images](assets/pg517-3.png)

# Explore the Enumerable class

We’ve been using sequences for a while. We know they work with `foreach` loops and LINQ. But what, exactly, makes sequences tick? Let’s take a deeper dive to find out. We’ll start with the **Enumerable class**—specifically, with its three static methods, Range, Empty, and Repeat. You already saw the Enumerable.Range method earlier in the chapter. Let’s use the IDE to discover how the other two methods work. Type `**Enumerable**`. and then hover over Range, Empty, and Repeat in the IntelliSense pop-up to see their declarations and comments.

![Images](assets/pg521.png)

## Enumerable.Empty creates an empty sequence of any type

Sometimes you need to pass an empty sequence to a method that takes an IEnumerable<T> (for example, in a unit test). The **Enumerable.Empty method** comes in handy in these cases:

[PRE38]

## Enumerable.Repeat repeats a value a number of times

Let’s say you need a sequence of 100 3s, or 12 “yes” strings, or 83 identical anonymous objects. You’d be surprised at how often that happens! You can use the **Enumerable.Repeat method** for this—it returns a sequence of repeated values:

[PRE39]

## So what exactly is an IEnumerable<T>?

We’ve been using IEnumerable<T> for a while now. We haven’t really answered the question of what an enumerable sequence *actually is*. A really effective way to understand something is to build it ourselves, so let’s finish the chapter by building some sequences from the ground up.

# Create an enumerable sequence by hand

Let’s say we have some sports:

[PRE40]

Obviously, we could create a new List<Sport> and use a collection initializer to populate it. But for the sake of exploring how sequences work, we’ll build one manually. Let’s create a new class called ManualSportSequence and make it implement the IEnumerable<Sport> interface. It just has two members that return an IEnumerator:

![Images](assets/pg522-1.png)

OK, so what’s an IEnumerator? It’s an interface that lets you enumerate a sequence, moving through each item in the sequence one after another. It has a property, Current, which returns the current item being enumerated. Its MoveNext method moves to the next element in the sequence, returning false if the sequence has run out. After MoveNext is called, Current returns that next element. Finally, the Reset method resets the sequence back to the beginning. Once you have those methods, you have an enumerable sequence.

![Images](assets/pg522-2.png)

So let’s implement an IEnumerator<Sport>:

![Images](assets/pg522-3.png)

And that’s all we need to create our own IEnumerable. Go ahead—give it a try. **Create a new console app**, add ManualSportSequence and ManualSportEnumerator, and then enumerate the sequence in a `foreach` loop:

[PRE41]

# Use yield return to create your own sequences

C# gives you a much easier way to create enumerable sequences: the `**yield return statement**`. The `yield return` statement is a kind of all-in-one automatic enumerator creator. A good way to understand it is to see an example. Let’s use a **multiproject solution**, just to give you a little more practice with that.

**Add a new Console App project to your solution**—this is just like what you did when you added the MSTest project earlier in the chapter, except this time instead of choosing the project type MSTest choose the same Console App project type that you’ve been using for most of the projects in the book. Then right-click on the project under the solution and **choose “Set as startup project.”** Now when you launch the debugger in the IDE, it will run the new project. You can also right-click on any project in the solution and run or debug it.

Here’s the code for the new console app:

![Images](assets/pg523-1.png)

Run the app—it prints four lines: `apples`, `oranges`, `bananas`, and `unicorns`. So how does that work?

## Use the debugger to explore yield return

Set a breakpoint on the first line of the Main method and launch the debugger. Then use **Step Into** (F11 / ![Images](assets/pg523-2a.png)) to debug the code line by line, right into the iterator:

*   Step into the code, and keep stepping into it until you reach the first line of the SimpleEnumerable method.

*   Step into that line again. It acts just like a `return` statement, returning control back to the statement that called it—in this case, back to the `foreach` statement, which calls Console.WriteLine to write `apples`.

*   Step two more times. Your app will jump back into the SimpleEnumerable method, but ***it skips the first statement in the method*** and goes right to the second line:

    ![Images](assets/pg523-2.png)
*   Keep stepping. The app returns to the `foreach` loop, then back to the ***third line*** of the method, then returns to the `foreach` loop, and goes back to the ***fourth line*** of the method.

So `yield return` makes a method **return an enumerable sequence** by returning the next element in the sequence each time it’s called, and keeping track of where it returned from so it can pick up where it left off.

# Use yield return to refactor ManualSportSequence

You can create your own IEnumerable<T> by **using `yield return` to implement the GetEnumerator method**. For example, here’s a BetterSportSequence class that does exactly the same thing as ManualSportSequence did. This version is much more compact because it uses `yield return` in its GetEnumerator implementation:

![Images](assets/pg524.png)

Go ahead and **add a new Console App project to your solution**. Add this new BetterSportSequence class, and modify the Main method to create an instance of it and enumerate the sequence.

## Add an indexer to BetterSportSequence

You’ve seen that you can use `yield return` in a method to create an IEnumerator<T>. You can also use it to create a class that implements IEnumerable<T>. One advantage of creating a separate class for your sequence is that you can add an **indexer**. You’ve already used indexers—any time you use brackets `[]` to retrieve an object from a list, array, or dictionary (like `myList[3]` or `myDictionary["Steve"]`), you’re using an indexer. An indexer is just a method. It looks a lot like a property, except it’s got a single named parameter.

The IDE has an ***especially useful code snippet*** to help you add your indexer. Type `**indexer**` followed by two tabs, and the IDE will add the skeleton of an indexer for you automatically.

Here’s an indexer for the SportCollection class:

[PRE42]

Calling the indexer with `[3]` returns the value `Hockey`:

[PRE43]

Take a close look when you use the snippet to create the indexer—it lets you set the type. You can define an indexer that takes different types, including strings and even objects. While our indexer only has a getter, you can also include a setter (just like the ones you’ve used to set items in a List).

# Collectioncross

![Images](assets/pg527.png)

[EclipseCrossword.com](http://EclipseCrossword.com)

**Across**

1\. Use the var keyword to declare an _____ typed variable

7\. A collection _____ combines the declaration with items to add

9\. What you’re trying to make your code when you have lots of tests for weird data and edge cases

11\. LINQ method to return the last elements in a sequence

12\. A last-in, first-out (LIFO) collection

18\. LINQ method to return the first elements in a sequence

19\. A method that has multiple constructors with different parameters

20\. The type of parameter that tells you that you can use a lambda

21\. What you take advantage of when you upcast an entire list

22\. What you’re using when you call myArray[3]

25\. What T gets replaced with when you see <T> in a class or interface definition

32\. The keyword you use to create an anonymous object

33\. A data type that only allows certain values

34\. The kind of collection that can store any type

35\. The interface that all sequences implement

36\. Another name for the ?: conditional operator

**Down**

1\. If you want to sort a List, its members need to implement this

2\. A collection class for storing items in order

3\. A collection that stores keys and values

4\. What you pass to List.Sort to tell it how to sort its items

**Down**

5\. What goes in the parentheses: ( _____ ) => expression;

6\. You can’t use the var keyword to declare one of these

8\. The access modifier for a class that can’t be accessed by another project in a multiproject solution

10\. The kind of expression the => operator creates

13\. LINQ method to append the elements from one sequence to the end of another

14\. Every collection has this method to put a new element into it

15\. What you can do with methods in a class that return the type of that class

16\. What kind of type you’re looking at when the IDE tells you this: `’a’ is a new string Color, int Height`

17\. An object’s namespace followed by a period followed by the class is a fully _____ class name

23\. The kind of evaluation that means a LINQ query isn’t run until its results are accessed

24\. The clause in a LINQ query that sorts the results

26\. Type of variable created by the from clause in a LINQ query

27\. The Enumerable method that returns a sequence with many copies of the same element

28\. The clause in a LINQ query that determines which elements in the input to use

29\. A LINQ query that merges data from two sequences

30\. A first-in, first-out (FIFO) collection

31\. The keyword a switch statement has that a switch expression doesn’t

# Collectioncross Solution

![Images](assets/pg528-3.png)