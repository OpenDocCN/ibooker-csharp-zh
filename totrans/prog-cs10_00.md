# Preface

C# has now existed for around two decades. It has grown steadily in both power and size, but Microsoft has always kept the essential characteristics intact. Each new capability is designed to integrate cleanly with the rest, enhancing the language without turning it into an incoherent bag of miscellaneous features.

Even though C# continues to be a fairly straightforward language at its heart, there is a great deal more to say about it now than in its first incarnation. Because there is so much ground to cover, this book expects a certain level of technical ability from its readers.

# Who This Book Is For

I have written this book for experienced developers—I’ve been programming for years, and I set out to make this the book I would want to read if that experience had been in other languages, and I were learning C# today. Whereas earlier editions explained some basic concepts such as classes, polymorphism, and collections, I am assuming that readers will already know what these are. The early chapters still describe how C# presents these common ideas, but the focus is on the details specific to C#, rather than the broad concepts.

# Conventions Used in This Book

The following typographical conventions are used in this book:

*Italic*

Indicates new terms, URLs, email addresses, filenames, and file extensions.

`Constant width`

Used for program listings, as well as within paragraphs to refer to program elements such as variable or function names, databases, data types, environment variables, statements, and keywords.

**`Constant width bold`**

Shows commands or other text that should be typed literally by the user. In examples, highlights code of particular interest.

*`Constant width italic`*

Shows text that should be replaced with user-supplied values or by values determined by context.

###### Tip

This element signifies a tip or suggestion.

###### Note

This element signifies a general note.

###### Warning

This element indicates a warning or caution.

# Using Code Examples

Supplemental material (code examples, exercises, etc.) is available for download at [*https://oreil.ly/prog-cs-10-repo*](https://oreil.ly/prog-cs-10-repo).

If you have a technical question or a problem using the code examples, please send email to [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com).

This book is here to help you get your job done. In general, if example code is offered with this book, you may use it in your programs and documentation. You do not need to contact us for permission unless you’re reproducing a significant portion of the code. For example, writing a program that uses several chunks of code from this book does not require permission. Selling or distributing examples from O’Reilly books does require permission. Answering a question by citing this book and quoting example code does not require permission. Incorporating a significant amount of example code from this book into your product’s documentation does require permission.

We appreciate, but generally do not require, attribution. An attribution usually includes the title, author, publisher, and ISBN. For example: “*Programming C# 10* by Ian Griffiths (O’Reilly). Copyright 2022 by Ian Griffiths, 978-1-098-11781-8.”

If you feel your use of code examples falls outside fair use or the permission given above, feel free to contact us at [*permissions@oreilly.com*](mailto:permissions@oreilly.com).

# O’Reilly Online Learning

###### Note

For more than 40 years, [*O’Reilly*](http://oreilly.com) has provided technology and business training, knowledge, and insight to help companies succeed.

Our unique network of experts and innovators share their knowledge and expertise through books, articles, conferences, and our online learning platform. O’Reilly’s online learning platform gives you on-demand access to live training courses, in-depth learning paths, interactive coding environments, and a vast collection of text and video from O’Reilly and 200+ other publishers. For more information, please visit [*http://oreilly.com*](http://www.oreilly.com).

# How to Contact Us

Please address comments and questions concerning this book to the publisher:

*   O’Reilly Media, Inc.
*   1005 Gravenstein Highway North
*   Sebastopol, CA 95472
*   800-998-9938 (in the United States or Canada)
*   707-829-0515 (international or local)
*   707-829-0104 (fax)

We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at [*https://oreil.ly/prgrmg-c-10*](https://oreil.ly/prgrmg-c-10).

Email us with comments or technical questions at [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com).

For news and information about our books and courses, visit [*https://oreilly.com*](https://oreilly.com).

Find us on LinkedIn: [*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media).

Follow us on Twitter: [*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia).

Watch us on YouTube: [*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia).

# Acknowledgments

Many thanks to the book’s official technical reviewers: Stephen Toub, Howard van Rooijen, and Glyn Griffiths. I’d also like to give a big thank-you to those who reviewed individual chapters or otherwise offered help or information that improved this book: Brian Rasmussen, Eric Lippert, Andrew Kennedy, Daniel Sinclair, Brian Randell, Mike Woodring, Mike Taulty, Bart De Smet, Matthew Adams, Jess Panni, Jonathan George, Mike Larah, Carmel Eve, Ed Freeman, Elisenda Gascon, Jessica Hill, Liam Mooney, Nehemiah Campbell, and Shahryar Saljoughi. Thanks in particular to endjin, both for allowing me to take time out from work to write this book and for creating such a great place to work.

Thank you to everyone at O’Reilly whose work brought this book into existence. In particular, thanks to Corbin Collins for his support in making this book happen, and to Amanda Quinn for her support in getting this project started. Thanks also to Elizabeth Faerm, Cassandra Furtado, Ron Bilodeau, Nick Adams, Kate Dullea, Karen Montgomery, and Kristen Brown, for their help in bringing the work to completion. Further thanks to Sue Klefstad and WordCo Indexing Services, Inc. for their work on the index. Thanks also to Kim Cofer for her thorough and thoughtful copy editing and to Kim Sandoval for her diligent proofreading. Finally, thank you to John Osborn, for taking me on as an O’Reilly author back when I wrote my first book.