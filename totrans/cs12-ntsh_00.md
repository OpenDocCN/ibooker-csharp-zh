# Preface

C# 12 represents the ninth major update to Microsoft’s flagship programming language, positioning C# as a language with unusual flexibility and breadth. At one end, it offers high-level abstractions such as query expressions and asynchronous continuations, whereas at the other end, it allows low-level efficiency through constructs such as custom value types and optional pointers.

The price of this growth is that there’s more than ever to learn. Although tools such as Microsoft’s IntelliSense—and online references—are excellent in helping you on the job, they presume an existing map of conceptual knowledge. This book provides exactly that map of knowledge in a concise and unified style—free of clutter and long introductions.

Like the past seven editions, *C# 12 in a Nutshell* is organized around concepts and use cases, making it friendly both to sequential reading and to random browsing. It also plumbs significant depths while assuming only basic background knowledge, making it accessible to intermediate as well as advanced readers.

This book covers C#, the Common Language Runtime (CLR), and the .NET 8 Base Class Library (BCL). We’ve chosen this focus to allow space for difficult and advanced topics without compromising depth or readability. Features recently added to C# are flagged so that you can also use this book as a reference for C# 11 and C# 10.

# Intended Audience

This book targets intermediate to advanced audiences. No prior knowledge of C# is required, but some general programming experience is necessary. For the beginner, this book complements, rather than replaces, a tutorial-style introduction to programming.

This book is an ideal companion to any of the vast array of books that focus on an applied technology such as ASP.NET Core or Windows Presentation Foundation (WPF). *C# 12 in a Nutshell* covers the areas of the language and .NET that such books omit, and vice versa.

If you’re looking for a book that skims every .NET technology, this is not for you. This book is also unsuitable if you want to learn about APIs specific to mobile device development.

# How This Book Is Organized

[Chapter 2](ch02.html#chash_language_basics) through [Chapter 4](ch04.html#advanced_chash) concentrate purely on C#, starting with the basics of syntax, types, and variables, and finishing with advanced topics such as unsafe code and preprocessor directives. If you’re new to the language, you should read these chapters sequentially.

The remaining chapters focus on .NET 8’s Base Class Libraries, covering such topics as Language-Integrated Query (LINQ), XML, collections, concurrency, I/O and networking, memory management, reflection, dynamic programming, attributes, cryptography, and native interoperability. You can read most of these chapters randomly, except for Chapters [5](ch05.html#dotnet_overview) and [6](ch06.html#dotnet_fundamentals), which lay a foundation for subsequent topics. You’re also best off reading the three chapters on LINQ in sequence, and some chapters assume some knowledge of concurrency, which we cover in [Chapter 14](ch14.html#concurrency_and_asynchron).

# What You Need to Use This Book

The examples in this book require .NET 8\. You will also find Microsoft’s .NET documentation useful to look up individual types and members (which is available [online](https://oreil.ly/y5bAc)).

Although it’s possible to write source code in a simple text editor and build your program from the command line, you’ll be much more productive with a *code scratchpad* for instantly testing code snippets, plus an *integrated development environment* (IDE) for producing executables and libraries.

For a Windows code scratchpad, download LINQPad 8 from [*www.linqpad.net*](http://www.linqpad.net) (free). LINQPad fully supports C# 12 and is maintained by the author.

For a Windows IDE, download *Visual Studio 2022*: any edition is suitable for what’s taught in this book. For a cross-platform IDE, download *Visual Studio Code*.

###### Note

All code listings for all chapters are available as interactive (editable) LINQPad samples. You can download the entire lot in a single click: at the bottom left, click the LINQPad’s Samples tab, click “Download more samples,” and then choose “C# 12 in a Nutshell.”

# Conventions Used in This Book

The book uses basic UML notation to illustrate relationships between types, as shown in [Figure P-1](#sample_diagram). A slanted rectangle means an abstract class; a circle means an interface. A line with a hollow triangle denotes inheritance, with the triangle pointing to the base type. A line with an arrow denotes a one-way association; a line without an arrow denotes a two-way association.

![Sample diagram](assets/cn10_0001.png)

###### Figure P-1\. Sample diagram

The following typographical conventions are used in this book:

*Italic*

Indicates new terms, URIs, filenames, and directories

`Constant width`

Indicates C# code, keywords and identifiers, and program output

`**Constant width bold**`

Shows a highlighted section of code

`*Constant width italic*`

Shows text that should be replaced with user-supplied values

# Using Code Examples

Supplemental material (code examples, exercises, etc.) is available for download at [*http://www.albahari.com/nutshell*](http://www.albahari.com/nutshell).

This book is here to help you get your job done. In general, you may use the code in this book in your programs and documentation. You do not need to contact us for permission unless you’re reproducing a significant portion of the code. For example, writing a program that uses several chunks of code from this book does not require permission. Selling or distributing examples from O’Reilly books *does* require permission. Answering a question by citing this book and quoting example code does not require permission (although we appreciate attribution). Incorporating a significant amount of example code from this book into your product’s documentation *does* require permission.

We appreciate, but generally do not require, attribution. An attribution usually includes the title, author, publisher, and ISBN. For example: “*C# 12 in a Nutshell* by Joseph Albahari (O’Reilly). Copyright 2024 Joseph Albahari, 978-1-098-14744-0.”

If you feel your use of code examples falls outside fair use or the permission given here, feel free to contact us at [permissions@oreilly.com](mailto:permissions@oreilly.com).

# How to Contact Us

Please address comments and questions concerning this book to the publisher:

*   O’Reilly Media, Inc.
*   1005 Gravenstein Highway North
*   Sebastopol, CA 95472
*   800-889-8969 (in the United States or Canada)
*   707-829-7019 (international or local)
*   707-829-0104 (fax)
*   [*support@oreilly.com*](mailto:support@oreilly.com)
*   [*https://www.oreilly.com/about/contact.html*](https://www.oreilly.com/about/contact.html)

We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at [*https://oreil.ly/c-sharp-nutshell-12*](https://oreil.ly/c-sharp-nutshell-12).

Code listings and additional resources are provided at:

*   [*http://www.albahari.com/nutshell/*](http://www.albahari.com/nutshell/)

For news and information about our books and courses, visit [*https://oreilly.com*](https://oreilly.com).

Find us on LinkedIn: [*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

Follow us on Twitter: [*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)

Watch us on YouTube: [*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)

# Acknowledgments

## Joseph Albahari

Since its first incarnation in 2007, this book has relied on input from some superb technical reviewers. For their input into recent editions, I’d like to extend particular thanks to Stephen Toub, Paulo Morgado, Fred Silberberg, Vitek Karas, Aaron Robinson, Jan Vorlicek, Sam Gentile, Rod Stephens, Jared Parsons, Matthew Groves, Dixin Yan, Lee Coward, Bonnie DeWitt, Wonseok Chae, Lori Lalonde, and James Montemagno.

And for their input into earlier editions, I’m most grateful to Eric Lippert, Jon Skeet, Stephen Toub, Nicholas Paldino, Chris Burrows, Shawn Farkas, Brian Grunkemeyer, Maoni Stephens, David DeWinter, Mike Barnett, Melitta Andersen, Mitch Wheat, Brian Peek, Krzysztof Cwalina, Matt Warren, Joel Pobar, Glyn Griffiths, Ion Vasilian, Brad Abrams, and Adam Nathan.

I appreciate that many of the technical reviewers are accomplished individuals at Microsoft, and I particularly thank you for taking the time to raise this book to the next quality bar.

I want to thank Ben Albahari and Eric Johannsen, who contributed to previous editions, and the O’Reilly team—particularly my efficient and responsive editor Corbin Collins. Finally, my deepest thanks to my wonderful wife, Li Albahari, whose presence kept me happy throughout the project.