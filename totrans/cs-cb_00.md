# Preface

# Why I Wrote This Book

In the course of a career, we collect many tools. Whether concepts, techniques, patterns, or reusable code, these tools help us get our job done. The more we collect, the better, because we have so many problems to solve and applications to build. *C# Cookbook* contributes to your toolset by providing you with a variety of recipes.

Things change over time, including programming languages. As of this writing, the C# programming language is over 20 years old, and software development has changed during its lifetime. There are a lot of recipes that could be written; this book acknowledges the evolution of C# over time and the fact that modern C# code makes us more productive.

This book is full of recipes that I’ve used throughout my career. In addition to stating a problem, presenting code, and explaining the solution, each discussion includes deeper insight into why each recipe is important. Throughout the book, I’ve avoided advocacy of process or absolute declarations of “you must do it this way” because much of what we do in creating software requires trade-offs. In fact, you’ll find several discussions of what the consequences or trade-offs are with a recipe. This respects the fact that you can consider to what extent a recipe applies to you.

# Who This Book Is For

This book assumes that you already know basic C# syntax. That said, there are recipes for various levels of developers. Whether you’re a beginner, intermediate, or senior developer, there should be something for you. If you’re an architect, there might be some interesting recipes that help you get back up to speed on the latest C# techniques.

# How This Book Is Organized

When brainstorming for this book, the entire focus was on answering the question “What do C# developers need to do?” Looking at the list, certain patterns emerged and evolved into chapters:

*   One of the first things I do when writing code is to build the types and organize the application. So I wrote [Chapter 1](ch01.xhtml#constructing_apps_and_types) to show how to create and organize types. You’ll see recipes dealing with patterns because that’s how I code.

*   After creating types, we add type members, like methods, and the logic they contain, which is a natural category of recipes for [Chapter 2](ch02.xhtml#coding_algorithms).

*   What good is code unless it works well? That’s why I added [Chapter 3](ch03.xhtml#ensuring_quality), which contains recipes that help improve the quality of code. While this chapter is packed with useful recipes, you’ll want to check out the recipe that shows how to use nullable reference types.

While Chapters [1](ch01.xhtml#constructing_apps_and_types) through [3](ch03.xhtml#ensuring_quality) follow the “What do C# developers need to do?” theme, from [Chapter 4](ch04.xhtml#querying_with_linq) to the end of the book I break away, taking a technology-specific focus:

*   Many people think of Language Integrated Query (LINQ) as a database technology. While LINQ is useful for working with databases, it’s also excellent for in-memory data manipulation and querying. That’s why [Chapter 4](ch04.xhtml#querying_with_linq) discusses what you can do with the in-memory provider, called LINQ to Objects.

*   Reflection was part of C# 1, but dynamic programming came along later in C# 4\. I think it’s important to discuss both technologies in [Chapter 5](ch05.xhtml#implementing_dynamic_and_reflection) and even show how dynamic programing can be better than reflection in some situations. Also, be sure to check out the Python interop recipes.

*   Async programming was a great addition to C# and seems straightforward, on the surface. [Chapter 6](ch06.xhtml#programming_asynchronously) covers async with recipes that explain several important features you might not be aware of.

*   All apps use data, whether securing, parsing, or serializing. [Chapter 7](ch07.xhtml#manipulating_data) includes several recipes covering different things you want to do with data. It focuses on some of the newer libraries and algorithms you might want to use for working with data.

*   One of the largest transformations of the C# language occurred over the last few versions in the area of pattern matching. There are so many that I was able to fill [Chapter 8](ch08.xhtml#matching_with_patterns) with only pattern-matching recipes.

*   C# continues to evolve and [Chapter 9](ch09.xhtml#examining_recent_csharp_langauge_highlights) captures recipes dedicated to C# 9\. We’ll look at some of the new features and discuss how to apply them. While I provide insight in the discussion, remember that sometimes new features can become more integral to a language in later versions. If you’re into the cutting edge, these recipes are pretty interesting.

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

Supplemental material (code examples, exercises, etc.) is available for download at [*https://github.com/JoeMayo/csharp-nine-cookbook*](https://github.com/JoeMayo/csharp-nine-cookbook).

If you have a technical question or a problem using the code examples, please send email to [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com).

This book is here to help you get your job done. In general, if example code is offered with this book, you may use it in your programs and documentation. You do not need to contact us for permission unless you’re reproducing a significant portion of the code. For example, writing a program that uses several chunks of code from this book does not require permission. Selling or distributing examples from O’Reilly books does require permission. Answering a question by citing this book and quoting example code does not require permission. Incorporating a significant amount of example code from this book into your product’s documentation does require permission.

We appreciate, but generally do not require, attribution. An attribution usually includes the title, author, publisher, and ISBN. For example: “*C# Cookbook* by Joe Mayo (O’Reilly). Copyright 2022 Mayo Software, LLC, 978-1-492-09369-5.”

If you feel your use of code examples falls outside fair use or the permission given above, feel free to contact us at [*permissions@oreilly.com*](mailto:permissions@oreilly.com).

# O’Reilly Online Learning

###### Note

For more than 40 years, [*O’Reilly Media*](http://oreilly.com) has provided technology and business training, knowledge, and insight to help companies succeed.

Our unique network of experts and innovators share their knowledge and expertise through books, articles, and our online learning platform. O’Reilly’s online learning platform gives you on-demand access to live training courses, in-depth learning paths, interactive coding environments, and a vast collection of text and video from O’Reilly and 200+ other publishers. For more information, visit [*http://oreilly.com*](http://oreilly.com).

# How to Contact Us

Please address comments and questions concerning this book to the publisher:

*   O’Reilly Media, Inc.
*   1005 Gravenstein Highway North
*   Sebastopol, CA 95472
*   800-998-9938 (in the United States or Canada)
*   707-829-0515 (international or local)
*   707-829-0104 (fax)

We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at [*https://oreil.ly/c-sharp-cb*](https://oreil.ly/c-sharp-cb).

Email [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com) to comment or ask technical questions about this book.

For news and information about our books and courses, visit [*http://oreilly.com*](http://oreilly.com).

Find us on Facebook: [*http://facebook.com/oreilly*](http://facebook.com/oreilly)

Follow us on Twitter: [*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

Watch us on YouTube: [*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# Acknowledgments

From concept to delivery, there are many people involved in creating a new book. I would like to recognize the people who helped on *C# Cookbook*.

Amanda Quinn, Senior Content Acquisitions Editor, helped form the concept for the book and provided feedback as I outlined its contents. Angela Rufino, Content Development Editor, got me started on standards and tools, provided feedback on my writing, and was incredibly helpful during the entire process. Bassam Alugili, Octavio Hernandez, and Shadman Kudchikar were tech editors, correcting errors, adding excellent insight, and sharing new ideas. Katherine Tozer, Production Editor and Vendor Coordinator, kept me up to date on new early releases and coordinated other production items. Justin Billing, Copyeditor, did a great job at improving my writing. There are many other people behind the scenes that made this book possible.

I would like to thank all of you. I am grateful for your contributions and wish you all the best.