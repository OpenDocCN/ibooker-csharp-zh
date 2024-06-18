# Preface

If you had to pick two popular technologies to focus on that emerged over the last 20 years, it would be tough to pick two better choices than C# and AWS. Anders Hejlsberg designed C# for Microsoft in 2000, and a few years later, the .NET Framework and Visual Studio launched. Next, in 2004, Mono, a cross-platform compiler and runtime for C#, became widely available. This ecosystem has enabled unique platforms and technologies, including the cross-platform mobile framework Xamarin.

In an alternate universe in the early 2000s, Amazon switched to a service-oriented architecture that culminated in a 2002 release of public web services. As the services grew in popularity, Amazon released SQS in 2004 and Amazon S3 and EC2 in 2006\. These storage, compute, and messaging services are still the core building blocks of Amazon Web Services (AWS). As of 2022, AWS has at least 27 geographic regions, 87 availability zones, millions of servers, and hundreds of services. These services include various offerings ranging from machine learning (SageMaker) to serverless (Lambda). This book shows what is possible with these two remarkable technology stacks. You’ll learn the theory surrounding cloud computing and how to implement it using AWS. You will also build many solutions with AWS technology, from serverless to DevOps implemented with the power of the .NET ecosystem, culminating in many elegant solutions that use these two powerful technologies.

# Who Should Read This Book

This book is for C# developers looking to explore the possibilities on AWS. The book could also be considered an introduction to AWS and cloud computing in general. We will introduce many cloud computing topics from a standing start, so if this is your first foray into cloud-based software development, then rest assured we will provide all the context you will need.

While C# is *technically* just a programming language and not a framework, we will be relating almost exclusively to .NET application developers building web applications. There will be passing references to Blazor and Xamarin but knowledge of these is absolutely not required. We also have content relevant to developers familiar with both the older .NET Framework, and to the newer .NET (previously *.NET Core*).

# How This Book Is Organized

AWS offers over 200 services to build all kinds of cloud-based applications, and we have organised this book to cover the services we feel are most important to a C# developer. The chapters are structured as follows.

[Chapter 1, “Getting Started with .NET on AWS”](ch01.xhtml#Chapter1), serves as an introduction to AWS and how to interact with and configure your services.

[Chapter 2, “AWS Core Services”](ch02.xhtml#Chapter2), covers Storage and Compute in detail, two of the most widely used services that will also feel most familiar to a .NET application developer used to deploying to web servers.

[Chapter 3, “Migrating a Legacy .NET Framework Application to AWS”](ch03.xhtml#Chapter3), serves as a brief overview of the many ways you can rapidly migrate an existing codebase to AWS, either from an existing physical or virtual machine, or another cloud provider.

[Chapter 4, “Modernizing .NET Applications to Serverless”](ch04.xhtml#Chapter4), takes you one step further than migrating existing applications and looks at ways you can modernize the way your code is architected to follow a serverless model, with examples on AWS.

[Chapter 5, “Containerization of .NET”](ch05.xhtml#Chapter5), is all about containers. If you are already using Docker containers for your .NET web application, this chapter will help you rapidly deploy these to a managed container service on AWS.

[Chapter 6, “DevOps”](ch06.xhtml#Chapter6), did you know AWS has a fully managed continuous delivery service called CodePipeline? This chapter shows you how you can simplify your application’s path to production using the tools provided on AWS.

[Chapter 7, “Logging, Monitoring, and Instrumentation for .NET”](ch07.xhtml#Chapter7), Monitoring and Instrumentation for .NET continues on from DevOps to demonstrate some of the logging and monitoring functionality you can integrate into your applications. We will also look at manually pushing performance metrics directly from your C# code.

[Chapter 8, “Developing with AWS C# SDK”](ch08.xhtml#Chapter8), is the final chapter of this book and offers a more in depth look at the SDK tools for C# that allow you to integrate AWS services into your code.

Finally, there are two appendixes that you might find helpful at the back of the book that didn’t fit in cleanly in any chapter. [Appendix A, “Benchmarking AWS”](app01.xhtml#appendix-benchmark-aws), walks through how to benchmark AWS machines and [Appendix B, “Getting Started with .NET”](app02.xhtml#appendix-csharp-tutorial), shows a brief C# and F# GitHub Codespaces tutorial.

Additionally, each chapter includes critical thinking questions and exercises. Critical thinking questions are a great starting point for team discussions, user groups, or preparation for a job interview or certification. The exercises help you learn by doing and applying the context of the chapter into a working code example. These examples could make excellent portfolio pieces.

# Additional Resources

For those with access to the O’Reilly platform, an optional resource that may help you with your AWS journey is [52 Weeks of AWS-The Complete Series](https://oreil.ly/kDf0Q); it contains hours of walk-throughs on all significant certifications. This series is also available as a free podcast on all [major podcast channels](https://podcast.paiml.com).

In addition to those resources, there are daily and weekly updates on AWS from Noah Gift at the following locations:

Noah Gift O’Reilly

On [Noah Gift’s O’Reilly profile](https://www.oreilly.com/pub/au/3039), there are hundreds of videos and courses covering a variety of technologies, including AWS, as well as frequent Live Trainings. Two other O’Reilly books covering AWS may also be helpful: [*Python for DevOps*](https://www.oreilly.com/library/view/python-for-devops/9781492057680) and [*Practical MLOps*](https://www.oreilly.com/library/view/practical-mlops/9781098103002).

Noah Gift Linkedin

On [Noah Gift’s LinkedIn](https://www.linkedin.com/in/noahgift), he periodically streams training on AWS and in-progress notes on O’Reilly books.

Noah Gift Website

[*Noahgift.com*](https://noahgift.com) is the best place to find the latest updates on new courses, articles, and talks.

Noah Gift GitHub

On [Noah Gift’s GitHub profile](https://github.com/noahgift), you can find hundreds of repositories and almost daily updates.

Noah Gift AWS Hero Profile

On [Noah Gift’s AWS Hero Profile](https://oreil.ly/FyFvf), you can find links to current resources involving work on the AWS platform.

Noah Gift Coursera Profile

On [Noah Gift’s Coursera profile](https://www.coursera.org/instructor/noahgift), you can find multiple specializations covering a wide range of topics in cloud computing with a heavy emphasis on AWS. These courses include:

*   [Cloud Computing Foundations](https://oreil.ly/XcxeC)

*   [Cloud Virtualization, Containers and APIs](https://oreil.ly/mcacu)

*   [Cloud Data Engineering](https://oreil.ly/pVHGm)

*   [Cloud Machine Learning Engineering and MLOps](https://oreil.ly/724Sz)

*   [Linux and Bash for Data Engineering](https://oreil.ly/Ps1Mz)

*   [Python and Pandas for Data Engineering](https://oreil.ly/07ywX)

*   [Scripting with Python and SQL for Data Engineering](https://oreil.ly/rSBMG)

*   [Web Applications and Command-Line Tools for Data Engineering](https://oreil.ly/gDDHy)

*   [Building Cloud Computing Solutions at Scale Specialization](https://oreil.ly/VZo7x)

*   [Python, Bash and SQL Essentials for Data Engineering Specialization](https://oreil.ly/TpIUU)

Pragmatic AI Labs Website

Many free resources related to AWS are available at the [*Pragmatic AI Labs and Solutions website*](https://paiml.com). These include several free books:

*   [*Cloud Computing for Data Analysis*](https://oreil.ly/I4ADa)

*   [*Minimal Python*](https://oreil.ly/JD2Ha)

*   [*Python Command Line Tools: Design Powerful Apps with Click*](https://oreil.ly/7zxNa)

*   [*Testing in Python*](https://oreil.ly/uJ5cC)

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

Supplemental material (code examples, exercises, etc.) is available for download at [*https://oreil.ly/AWS-with-C-Sharp-examples*](https://oreil.ly/AWS-with-C-Sharp-examples).

If you have a technical question or a problem using the code examples, please send email to [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com).

This book is here to help you get your job done. In general, if example code is offered with this book, you may use it in your programs and documentation. You do not need to contact us for permission unless you’re reproducing a significant portion of the code. For example, writing a program that uses several chunks of code from this book does not require permission. Selling or distributing examples from O’Reilly books does require permission. Answering a question by citing this book and quoting example code does not require permission. Incorporating a significant amount of example code from this book into your product’s documentation does require permission.

We appreciate, but generally do not require, attribution. An attribution usually includes the title, author, publisher, and ISBN. For example: “*Developing on AWS with C#* by Noah Gift and James Charlesworth (O’Reilly). Copyright 2023 O’Reilly Media, Inc., 978-1-492-09623-8.”

If you feel your use of code examples falls outside fair use or the permission given above, feel free to contact us at [*permissions@oreilly.com*](mailto:permissions@oreilly.com).

# O’Reilly Online Learning

###### Note

For more than 40 years, [*O’Reilly Media*](https://oreilly.com) has provided technology and business training, knowledge, and insight to help companies succeed.

Our unique network of experts and innovators share their knowledge and expertise through books, articles, and our online learning platform. O’Reilly’s online learning platform gives you on-demand access to live training courses, in-depth learning paths, interactive coding environments, and a vast collection of text and video from O’Reilly and 200+ other publishers. For more information, visit [*https://oreilly.com*](https://oreilly.com).

# How to Contact Us

Please address comments and questions concerning this book to the publisher:

*   O’Reilly Media, Inc.
*   1005 Gravenstein Highway North
*   Sebastopol, CA 95472
*   800-998-9938 (in the United States or Canada)
*   707-829-0515 (international or local)
*   707-829-0104 (fax)

We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at [*https://oreil.ly/AWS-with-C-sharp*](https://oreil.ly/AWS-with-C-sharp).

Email [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com) to comment or ask technical questions about this book.

For news and information about our books and courses, visit [*https://oreilly.com*](https://oreilly.com).

Find us on LinkedIn: [*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media).

Follow us on Twitter: [*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia).

Watch us on YouTube: [*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia).

# Acknowledgments

## Noah

It is always an honor to have the opportunity to work on an O’Reilly book. This book marks my fourth for O’Reilly, and I am working on the fifth book on Enterprise MLOps. Thank you to [Rachel Roumeliotis](https://oreil.ly/GbCM4) for allowing me to work on this book and [Zan McQuade](https://oreil.ly/TyZX7) for the excellent strategic direction.

Both my coauthor [James Charlesworth](https://oreil.ly/ReRkb) and our development editor [Melissa Potter](https://oreil.ly/Sm2Nn) are fantastic to work with, and I am lucky to work with two such talented people on a project this demanding. Thanks to Lior Kirshner-Shalom and from AWS, [Nivas Durairaj](https://oreil.ly/75Nab), David Pallmann, and Bryan Hogan, who provided many insightful comments and corrections during technical reviews.

Thanks as well to many of my current and former students as well as faculty and staff at [Duke MIDS](https://oreil.ly/2SWqF), [Duke Artificial Intelligence Masters in Engineering](https://oreil.ly/bTcbr), [UC Davis MSBA](https://oreil.ly/ZB0SH), [UC Berkeley](https://oreil.ly/cX4XH), [Northwestern MS in Data Science Program](https://oreil.ly/i9AIB) as well as [UTK](https://oreil.ly/u3rJi) and [UNC Charlotte](https://oreil.ly/yCydZ).

These people include in no particular order: [Dickson Louie](https://oreil.ly/gMxsP), [Andrew B. Hargadon](https://oreil.ly/IEnYf), [Hemant Bhargava](https://oreil.ly/Mgkru), [Amy Russell](https://oreil.ly/ABcjv), [Ashwin Aravindakshan](https://oreil.ly/fqVSg), [Ken Gilbert](https://oreil.ly/8aoUi), [Michel Ballings](https://oreil.ly/PouWU), [Jon Reifschneider](https://oreil.ly/IsCsh), [Robert Calderbank](https://oreil.ly/iQ0yY), and [Alfredo Deza](https://oreil.ly/HvsSl).

Finally, thank you to my family, Leah, Liam, and Theodore, who put up with me working on weekends and late at night to hit deadlines.

## James

This is my first book with O’Reilly, and I have been overwhelmed by the support we have received from the team, especially Melissa Potter, without whom you would have suffered a lot more of my convoluted British spellings. I’d also echo Noah’s thanks to the technical reviewers, specifically David Pallmann and Bryan Hogan at AWS.

The tech industry moves at such a rapid pace that you do not get anywhere without standing on the shoulders of giants. It is for this reason that I’d like to thank two of my former bosses, [James Woodley](https://oreil.ly/0ZmWS) and [Dave Rensin](https://oreil.ly/dlL6x), for all the advice, coaching, mentorship, and support over recent years.

Lastly, big shout out to my amazing wife, Kaja, who supports me through all my endeavors, I will always be your Richard Castle.