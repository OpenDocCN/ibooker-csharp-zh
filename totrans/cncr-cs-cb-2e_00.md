# Preface

I think the animal on this cover, a common palm civet, is applicable to the subject of this book. I knew nothing about this animal until I saw the cover, so I looked it up. Common palm civets are considered pests because they defecate all over attics and make loud noises fighting with each other at the most inopportune times. Their anal scent glands emit a nauseating secretion. They have an endangered species rating of “Least Concern,” which is apparently the politically correct way of saying, “Kill as many of these as you want; no one will miss them.” Common palm civets enjoy eating coffee cherries, and they pass the coffee beans through. Kopi luwak, one of the most expensive coffees in the world, is made from the coffee beans extracted from civet excretions. According to the Specialty Coffee Association of America, “It just tastes bad.”

This makes the common palm civet a perfect mascot for concurrent and multithreaded development. To the uninitiated, concurrency and multithreading are undesirable. They make well-behaved code act up in the most horrendous ways. Race conditions and whatnot cause loud crashes (always, it seems, either in production or during a demo). Some have gone so far as to declare “threads are evil” and avoid concurrency completely. There are a handful of developers who have developed a taste for concurrency and use it without fear; but most developers have been burned in the past by concurrency, and that experience has left a bad taste in their mouth.

However, for modern applications, concurrency is quickly becoming a requirement. Users these days expect fully responsive interfaces, and server applications are having to scale to unprecedented levels. Concurrency addresses both of these trends.

Fortunately, there are many modern libraries that make concurrency *much* easier! Parallel processing and asynchronous programming are no longer exclusively the domains of wizards. By raising the level of abstraction, these libraries make responsive and scalable application development a realistic goal for every developer. If you have been burned in the past, when concurrency was extremely difficult, then I encourage you to give it another try with modern tools. We can probably never call concurrency easy, but it sure isn’t as hard as it used to be!

# Who Should Read This Book

This book is written for developers who want to learn modern approaches to concurrency. I do assume that you’ve got a fair amount of .NET experience, including an understanding of generic collections, enumerables, and LINQ. I do *not* expect that you have any multithreading or asynchronous programming knowledge. If you do have some experience in those areas, you may still find this book helpful because it introduces newer libraries that are safer and easier to use.

Concurrency is useful for any kind of application. It doesn’t matter whether you work on desktop, mobile, or server applications; these days concurrency is practically a requirement across the board. You can use the recipes in this book to make user interfaces more responsive and servers more scalable. We are already at the point where concurrency is ubiquitous, and understanding these techniques and their uses is essential knowledge for the professional developer.

# Why I Wrote This Book

Early in my career, I learned multithreading the hard way. After a couple of years, I learned asynchronous programming the hard way. While those were both valuable experiences, I do wish that back then I had some of the tools and resources that are available today. In particular, the `async` and `await` support in modern .NET languages is pure gold.

However, if you look around today at books and other resources for learning concurrency, they almost all start by introducing the most low-level concepts. There’s excellent coverage of threads and serialization primitives, and the higher-level techniques are put off until later, if they’re covered at all. I believe this is for two reasons. First, many developers of concurrency, such as myself, did learn the low-level concepts first, slogging through the old-school techniques. Second, many books are years old and cover now-outdated techniques; as the newer techniques have become available, these books have been updated to include them, but have unfortunately placed them at the end.

I think that’s backward. In fact, this book *only* covers modern approaches to concurrency. That’s not to say there’s no value in understanding all the low-level concepts. When I went to college for programming, I had one class where I had to build a virtual CPU from a handful of gates, and another class that covered assembly programming. In my professional career, I’ve never designed a CPU, and I’ve only written a couple dozen lines of assembly, but my understanding of the fundamentals still helps me every day. Still, it’s best to start with the higher-level abstractions; my first programming class wasn’t in assembly language.

This book fills a niche: it is an introduction to (and reference for) concurrency using modern approaches. It covers several different kinds of concurrency, including parallel, asynchronous, and reactive programming. It does not, however, cover any of the old-school techniques, which are adequately covered in many other books and online resources.

# Navigating This Book

Here’s how the book is broken down:

*   [Chapter 1](ch01.html#intro) is an introduction to the various kinds of concurrency covered by this book: parallel, asynchronous, reactive, and dataflow.

*   Chapters [2](ch02.html#async-basics)–[6](ch06.html#rx-basics) are a more thorough introduction to these kinds of concurrency.

*   The remaining chapters each deal with a particular aspect of concurrency, and they act as a reference for solutions to common problems.

I recommend reading (or at least skimming) the first chapter, even if you’re already familiar with some kinds of concurrency.

###### Warning

As this book goes to press, .NET Core 3.0 is still in beta, so some details around asynchronous streams may change.

# Online Resources

This book acts like a broad-spectrum introduction to several different kinds of concurrency. I’ve done my best to include techniques that I and others have found the most helpful, but this book isn’t exhaustive by any means. The following resources are the best ones I’ve found for a more thorough exploration of these technologies:

*   For parallel programming, the best resource I know of is *Parallel Programming with Microsoft .NET* by Microsoft Press, the text of which is [available online](http://bit.ly/parallel-prog). Unfortunately, it’s already a bit out of date. The section on futures should use asynchronous code instead, and the section on pipelines should use Channels or TPL Dataflow.

*   For asynchronous programming, MSDN is quite good, particularly the [“Asynchronous Programming”](http://bit.ly/async-prog) overview.

*   Microsoft has also made available [documentation for TPL Dataflow.](http://bit.ly/dataflow-tpl)

*   System.Reactive (Rx) is a library that is gaining a lot of traction online and continues evolving. In my opinion, the best resource today for Rx is [*Introduction to Rx*](http://www.introtorx.com/), an ebook by Lee Campbell.

# Conventions Used in This Book

The following typographical conventions are used in this book:

*Italic*

Indicates new terms, URLs, email addresses, filenames, and file extensions.

`Constant width`

Used for program listings, as well as within paragraphs to refer to program elements such as variable or function names, databases, data types, environment variables, statements, and keywords.

**`Constant width bold`**

Shows commands or other text that should be typed literally by the user.

*`Constant width italic`*

Shows text that should be replaced with user-supplied values or by values determined by context.

###### Tip

This element signifies a tip or suggestion.

###### Note

This element signifies a general note.

###### Warning

This element indicates a warning or caution.

# Using Code Examples

Supplemental material (code examples, exercises, etc.) is available for download at [*https://oreil.ly/concur-c-ckbk2*](https://oreil.ly/concur-c-ckbk2).

This book is here to help you get your job done. In general, if example code is offered with this book, you may use it in your programs and documentation. You do not need to contact us for permission unless you’re reproducing a significant portion of the code. For example, writing a program that uses several chunks of code from this book does not require permission. Selling or distributing a CD-ROM of examples from O’Reilly books does require permission. Answering a question by citing this book and quoting example code does not require permission. Incorporating a significant amount of example code from this book into your product’s documentation does require permission.

We appreciate, but do not require, attribution. An attribution usually includes the title, author, publisher, and ISBN. For example: “*Concurrency in C# Cookbook*, Second Edition, by Stephen Cleary (O’Reilly). Copyright 2019 Stephen Cleary, 978-1-492-05450-4.”

If you feel your use of code examples falls outside fair use or the permission given above, feel free to contact us at [*permissions@oreilly.com*](mailto:permissions@oreilly.com).

# O’Reilly Online Learning

###### Note

For almost 40 years, [*O’Reilly Media*](http://oreilly.com) has provided technology and business training, knowledge, and insight to help companies succeed.

Our unique network of experts and innovators share their knowledge and expertise through books, articles, conferences, and our online learning platform. O’Reilly’s online learning platform gives you on-demand access to live training courses, in-depth learning paths, interactive coding environments, and a vast collection of text and video from O’Reilly and 200+ other publishers. For more information, please visit [*http://oreilly.com*](http://oreilly.com).

# How to Contact Us

Please address comments and questions concerning this book to the publisher:

*   O’Reilly Media, Inc.
*   1005 Gravenstein Highway North
*   Sebastopol, CA 95472
*   800-998-9938 (in the United States or Canada)
*   707-829-0515 (international or local)
*   707-829-0104 (fax)

We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at [*https://oreil.ly/concur-c-ckbk2*](https://oreil.ly/concur-c-ckbk2).

To comment or ask technical questions about this book, send email to [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com).

For more information about our books, courses, conferences, and news, see our website at [*http://www.oreilly.com*](http://www.oreilly.com).

Find us on Facebook: [*http://facebook.com/oreilly*](http://facebook.com/oreilly)

Follow us on Twitter: [*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

Watch us on YouTube: [*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# Acknowledgments

This book simply would not exist without the help of so many people!

First and foremost, I’d like to acknowledge my Lord and Savior, Jesus Christ. Becoming a Christian was the most important decision of my life! If you want more information on this subject, feel free to contact me via [my personal web page](http://stephencleary.com/).

Second, I thank my family for allowing me to give up so much time with them. When I started writing, I had some author friends of mine tell me, “Say goodbye to your family for the next year!” and I thought they were joking. My wife, Mandy, and our children, SD and Emma, have been very understanding while I put in long days at work followed by writing on evenings and weekends. Thank you so much. I love you!

Of course, this book wouldn’t be nearly as good as it is without my editors and our technical reviewers: Stephen Toub, Petr Onderka (“svick”), Nick Paldino (“casperOne”), Lee Campbell, and Pedro Felix. So if any mistakes get through, it’s totally their fault. Just kidding! Their input has been invaluable in shaping (and fixing) the content, and any remaining mistakes are of course my own. Particular thanks go to Stephen Toub, who taught me the Boolean Argument Hack ([Recipe 14.5](ch14.html#recipe-sync-and-async)), as well as countless other `async` topics; and Lee Campbell, who has helped me learn System.Reactive and make my observable code more idiomatic.

Finally, I’d like to thank some of the people I’ve learned these techniques from: Stephen Toub, Lucian Wischik, Thomas Levesque, Lee Campbell, the members of Stack Overflow and the MSDN forums, and the attendees of the software conferences in and around my home state of Michigan. I appreciate being a part of the software development community, and if this book adds any value, it’s because so many have already shown the way. Thank you all!