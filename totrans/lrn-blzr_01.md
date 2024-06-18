# Preface

Welcome to *Learning Blazor*. You’re probably here because you’ve heard some cool things about Blazor and you want to try it out. So, what is it? Blazor is an open source web framework for building interactive client-side web UI components using C# (pronounced “see sharp”), HTML, and cascading style sheets (CSS).^([1](preface01.html#idm46365047943984)) As a feature of ASP.NET Core, Blazor extends the .NET developer platform with tools and libraries for building web apps.

WebAssembly enables numerous non-JavaScript-based programming languages to run on the browser. Blazor takes full advantage of WebAssembly and allows C# developers to build UI components and client-side experiences with .NET. Blazor is a single-page application (SPA) framework, similar to Angular, React, VueJS, and Svelte, for example, but it’s based on C# instead of JavaScript.

Okay, it’s a web framework, but what makes it different from any other client-side framework for building web UI?

# Why Blazor?

Blazor is a game-changer for .NET developers and web developers alike! In this book, you’ll learn how you can use the Blazor WebAssembly hosting model to create compelling real-time web experiences. There are seemingly countless reasons to choose Blazor as your next web app development framework. Let’s start with what it does for web development.

Back in the early ’90s, surfing the web was like reading a series of linked text documents—basic HTML. It was hardly an immersive or cohesive experience. When CSS and JavaScript came onto the scene, the ability to dynamically respond to user interactions added a lot more flair to the web experience. Though the web pages started to look more interesting, they were also very slow to load, and people expected a sluggish user experience, with visible page rendering/buffering. It was completely acceptable to watch images render in segments as the underlying image data was buffered over HTTP to the browser at dial-up connection speeds. This patience didn’t last. It’s human nature to want things right now, am I right? If you’re sitting on a browser for more than a few seconds, you start to feel a bit uneasy. As web content became more complex, development frameworks appeared to tame the complexity.

Among such disruptive frameworks is Blazor with WebAssembly. With Blazor, you can share C# code on both client and server scenarios, all while leveraging tooling with the Visual Studio family of products, the robust .NET CLI, and other popular .NET integrated development environments (IDEs). The .NET ecosystem is thriving, adoption is soaring, and the appeal of Long Term Support (LTS) continues to be a driving factor for enterprise development. When compared to the LTS of other SPA frameworks, such as Angular and React, .NET stands out as the clear winner. This is because the support policy that .NET extends is three years from each LTS version. It’s very beneficial to stay current with each release. For more information, see the [.NET support policy](https://oreil.ly/sQE70).

Just like any other web app, Blazor web apps can be created as progressive web apps (PWAs) to support offline experiences. They can also be hosted inside native desktop applications and installed on the user’s device. Your Blazor WebAssembly apps can define native dependencies, such as that from C and C++. Anything compiled with [Emscripten](https://emscripten.org) can be used in Blazor. There aren’t many trade-offs to be made, in my opinion; the web development platform is in high demand and enjoyable to program for.

When WebAssembly was introduced, it initially received only moderate developer community attention and anticipation. In 2017, WebAssembly was openly standardized, which allowed developers to explore new possibilities for interactivity and functionality beyond JavaScript alone. This is important to web developers, as they could more easily compete with the lucrative App Store development platform. JavaScript continues to evolve, adding features beyond the ECMAScript standard. With .NET’s creation of Blazor, C# became a true competitor to JavaScript.

As a developer with more than a decade of real-world web app development experience, I can attest that I have used .NET for enterprise development of production applications time and time again. The API surface area of .NET alone is massive and has been used on billions of computer systems around the world. I’ve built a lot of web apps through the years using various technologies including ASP.NET WebForms, AngularJS, Angular, VueJS, Svelte, yes, and even React, then ASP.NET Core Model View Controller, Razor Pages, and Blazor. Blazor melds together the strength of an established ecosystem with the flexibility and poise of the web, and it has a lot to offer to both .NET and web developers.

# Who Should Read This Book

This book is for .NET developers and web developers with a basic understanding of HTML, CSS, Document Object Model, and JavaScript, as well as some experience developing applications in .NET. This book is not for people who are complete beginners to programming. For instance, when I told my mom that I was writing a book, she asked what it was about and if she’d enjoy reading it. I said, “No.” She’s neither a .NET developer nor a web developer, so I don’t think she’d find much value in this book. If you’re a .NET developer or web developer, however, you’re in for a treat.

## For .NET Developers

If you’re a .NET developer who is curious about web app development, this book will detail how you can harness your existing .NET skills and apply them to Blazor development. The web app platform is a major opportunity for .NET developers. All the popular JavaScript SPA frameworks, such as Angular, React, VueJS, and Svelte, have a true rival in Blazor. Blazor app development should be familiar to you as Blazor is based on .NET and C#. You can share libraries between client and server, making development truly enjoyable.

## For Web Developers

If you’re a web developer who has worked with .NET before, this book extends two sets of learned programming skills. All of your .NET experience carries over, as does your knowledge of web fundamentals. If you’re a SPA developer, this book will open your eyes to a better set of tooling than you’re accustomed to. We also go over many new C# features. If you’re unfamiliar with C#, this book will provide an idiomatic view of C# and a strongly opinionated experience.

###### Tip

If you’re asking yourself, “What does idiomatic C# mean?,” C#, like all programming languages, has a set of programming idioms. Programming idioms are a way of writing smarter and better code to get something done. Idiomatic C# is a set of idioms that are used to make your code more readable and maintainable.

Your JavaScript and developer experience of client-side routing and a deep understanding of HTTP, microservice architecture, dependency injection, and component-based application mindset—all these things are directly applicable to Blazor development. Application development shouldn’t be so difficult, and I truly believe that Blazor makes it easier. With feature-rich data binding, strongly typed templating, component hierarchy eventing, logging, localization, authentication, support for PWA, and hosting, you have all the building blocks to orchestrate compelling web experiences.

# Why I Wrote This Book

When someone asks me, “Why did you want to write a book?” I pause, feigning deep thought, before replying, “O’Reilly asked me to.” Simple as that. But in all seriousness, when I got a friendly email from an O’Reilly acquisitions editor to see if I was interested in writing a book about Blazor, I gave it a lot of thought. First, it was pretty cool to be asked! But I also knew taking on this kind of project would mean putting a few things on hold. I’d have to take a hiatus from speaking events, which have been a major part of my life over the past several years. Yet I thrive on helping others, so writing a book would be helping people differently. Writing a book would also mean taking time away from my young family. My family and my wife specifically have been extremely warm-hearted and supportive. She believes in my ability to help others and shares my passion. In the end, I decided, “Yes! I want to write a book!”

To me, helping the developer community also helps strengthen my understanding of a specific technology. I love Blazor! Blazor is (and has been) a major investment for the .NET and ASP.NET Microsoft development teams. They continue to drive innovation, extending the reach of C# and the .NET ecosystem as a whole. This book is a *developer must-have*, and it’s my way of giving back to the developer community I’ve grown to love. I have poured myself into this book, and I know my enthusiasm for Blazor shines through.

# How to Use This Book

This isn’t your typical “introduction to X” kind of book. It’s a technical book that’ll introduce you to using Blazor to build SPAs with WebAssembly and C#. There are plenty of books out there that use the step-by-step approach—this book is not one of them.

As you read this book, I want you to have an experience that is similar to the one you’d have when joining a new team. You’ll experience a bit of onboarding, you’ll be brought up to speed on an existing application, and you’ll learn various domain bits along the way. The “Learning Blazor” sample app is a decent-sized solution with well over a dozen projects of varying sizes. Each project contains or contributes to specific functionality in the Learning Blazor app. We will examine these projects as examples of how to do things in Blazor. As I take you through the inner workings of the app, you’ll learn Blazor app development along the way. By the end, you’ll gain experience with what goes into Blazor app development and understand why certain development decisions were made, and you’ll have working examples of how to get things done. You’ll close the book and have inspiration for your apps.

All of the examples in this book are shown using the Learning Blazor application (or model app). The source code from the model app, along with this book, makes for a great learning resource and future point of reference. The source code repository is available on GitHub and shared in [“The Code Must Live On”](ch01.html#github-repo).

# Roadmap and Goals of This Book

This book is structured as follows:

*   [Chapter 1, “Blazing into Blazor”](ch01.html#chapter-one), introduces the core concepts and fundamentals of Blazor for web app development as a platform. It also introduces the example app for this book and discusses its architecture.

*   [Chapter 2, “Executing the App”](ch02.html#chapter-two), dives into how the execution of the app functions starting from the first client request to the static website’s URL. You’ll learn how the HTML renders, how the subsequent requests for additional resources are called, and how Blazor bootstraps itself.

*   [Chapter 3, “​​Componentizing”](ch03.html#chapter-three), goes into how the user is represented within the app. You’ll learn how to use third-party authentication providers to verify a user’s identity. You’ll learn about customization of the authentication state UX and about various data-binding approaches with Razor control structures.

*   [Chapter 4, “Customizing the User Login Experience”](ch04.html#chapter-four), details how the client services are registered for dependency injection. You’ll learn about *componentization* and how to use the `RenderFragment` approach for customizing components. You’ll also learn how to write and use parameterized client-native speech synthesis that is fully functional and configurable in Blazor WebAssembly.

*   [Chapter 5, “Localizing the App”](ch05.html#chapter-five), demonstrates how you can use a free AI-based automated continuous delivery pipeline to support localization. You’ll learn how to use the framework-provided `IStringLocalizer<T>` type and corresponding services.

*   [Chapter 6, “Exemplifying Real-Time Web Functionality”](ch06.html#chapter-six), introduces real-time web functionality and shows a notification system, live tweet stream page, and alert capabilities. Additionally, you’ll learn how to build a chat app using ASP.NET Core SignalR.

*   [Chapter 7, “Using Source Generators”](ch07.html#chapter-seven), creates a case for source generators to improve the Blazor JavaScript interoperability (interop) experience. You’ll learn why C# source generators are so useful in app development and how they’ll save you loads of time.

*   [Chapter 8, “Accepting Form Input with Validation”](ch08.html#chapter-eight), explores how forms work. We’ll go through an advanced `<form>` of input validation. We’ll also look at how to incorporate native speech recognition into the form to give users another input option. You’ll learn how to use `EditContext` and form-model binding. [Chapter 8](ch08.html#chapter-eight) also demonstrates a pattern for custom state validation that receives live updates using Reactive Extensions for .NET.

*   [Chapter 9, “Testing All the Things”](ch09.html#chapter-nine), teaches you how to write unit tests, component tests, and even end-to-end tests to make sure your app works. These tests can be automated to run each time that the app is pushed to the GitHub repository using GitHub Actions.

# Conventions Used in This Book

The following typographical conventions are used in this book:

*Italic*

Indicates new terms, URLs, email addresses, filenames, and file extensions

`Constant width`

Used for program listings, as well as within paragraphs to refer to program elements such as variable or function names, databases, data types, environment variables, statements, and keywords

###### Tip

This element signifies a tip or suggestion.

###### Note

This element signifies a general note.

###### Warning

This element indicates a warning or caution.

# Using Code Examples

Supplemental material (code examples, exercises, etc.) is available for download at [*https://oreil.ly/learning-blazor-code*](https://oreil.ly/learning-blazor-code).

If you have a technical question or a problem using the code examples, please send email to [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com).

This book is here to help you get your job done. In general, if an example code is offered with this book, you may use it in your programs and documentation. You do not need to contact us for permission unless you’re reproducing a significant portion of the code. For example, writing a program that uses several chunks of code from this book does not require permission. Selling or distributing examples from O’Reilly books does require permission. Answering a question by citing this book and quoting example code does not require permission. Incorporating a significant amount of example code from this book into your product’s documentation does require permission.

We appreciate but generally do not require attribution. An attribution usually includes the title, author, publisher, and ISBN. For example: “*Learning Blazor* by David Pine (O’Reilly). Copyright 2023 David Pine, 978-1-098-11324-7.”

If you feel your use of code examples falls outside fair use or the permission given above, feel free to contact us at [*permissions@oreilly.com*](mailto:permissions@oreilly.com).

# O’Reilly Online Learning

###### Note

For more than 40 years, [*O’Reilly Media*](http://oreilly.com) has provided technology and business training, knowledge, and insight to help companies succeed.

Our unique network of experts and innovators share their knowledge and expertise through books, articles, and our online learning platform. O’Reilly’s online learning platform gives you on-demand access to live training courses, in-depth learning paths, interactive coding environments, and a vast collection of text and video from O’Reilly and 200+ other publishers. For more information, visit [*https://oreilly.com*](https://oreilly.com).

# How to Contact Us

Please address comments and questions concerning this book to the publisher:

*   O’Reilly Media, Inc.
*   1005 Gravenstein Highway North
*   Sebastopol, CA 95472
*   800-998-9938 (in the United States or Canada)
*   707-829-0515 (international or local)
*   707-829-0104 (fax)

We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at [*https://oreil.ly/learning-blazor*](https://oreil.ly/learning-blazor).

Email [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com) to comment or ask technical questions about this book.

For news and information about our books and courses, visit [*https://oreilly.com*](https://oreilly.com).

Find us on LinkedIn: [*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

Follow us on Twitter: [*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)

Watch us on YouTube: [*https://www.youtube.com/oreillymedia*](https://www.youtube.com/oreillymedia)

# Acknowledgments

I once traveled to Serbia as part of the ITkonekt developer conference. I shared a travel van with three amazing individuals. One was Jon Galloway, who at the time was the executive director of the .NET Foundation. The second was Jonathan LeBlanc, who is the only person I know who has won an Emmy Award for technology (and who is now a fellow O’Reilly author). The third individual was Håkon Wium Lie, who is known for being the creator of CSS and is the former CTO of Opera. It was a great opportunity to learn from all of them.

Anyway, during the trip, it came to light that, of the four of us in the van, I was the only one who hadn’t written a book. They immediately encouraged me to rectify that. They told me to share my knowledge with the world and write a book. I was touched to hear that my esteemed friends and colleagues believed in me. I didn’t write a book right away, but I did give it a lot of thought and waited until the time was right. Which is now! I’d like to thank Jon, Jonathan, and Håkon for believing in me and being inspirations to the developer community.

Please allow me to thank a few contributors to some of the source code that’s referenced in this book. Thank you, Ben Felda, for contributing SVGs and styling updates to the model app’s landing page tile components. Thank you, Max Schmitt, for helping me simplify my usage of Playwright testing framework from the model app’s build validation workflow. Thank you to Billy Mumby for your extensive work on the model app’s task-list feature by supporting the underlying data store. The work you’ve done to strengthen the [Azure Cosmos DB Repository .NET SDK](https://oreil.ly/1rmND) that we maintain is a huge asset to the developer community. Thank you to Weihan Li for his contributions to the model app’s consumption of the Blazor source generator, namely [Blazorators](https://oreil.ly/uBU3o). Thank you, Vsevolod Šliachtenko, for your collaboration and work with me on the [Azure resource translator GitHub Action](https://oreil.ly/1pGdr). You helped to implement request batching beautifully. Thank you, GitHub bot, for automating more than 73,000 lines of code as of July 2022 to the Learning Blazor project.

I’d like to thank my mentor and good friend David Fowler. David has been mentoring me for a long time, and I hold all of his valuable lessons near and dear to my heart. David contributed code to my “Have I Been Pwned” .NET HTTP Client open source project to simplify the Minimal API example. Our exchanges are often the highlight of my week; I share code, experiences, career challenges, and thoughts with him, and he reflects his brightness. He’s an inspiration to me and so many others, and I’m immensely grateful to learn from him.

Thank you to the O’Reilly team for their support and encouragement. I want to formally thank all of the reviewers of this book: Rita Fernando, Carol Tumey, Erik Hopf, Gerald Versluis, John Kennedy, Chad Olson, and Egil Hansen. Without their tirelessness and thorough reviews—from editorial reviews hanging on every word to in-depth technical reviews ensuring that every line of code is as simple and elegant as possible—this book would not have become as profound and helpful. The quality is backed by decades of professional real-world experience, and I’m thrilled by the result.

I would like to thank Steve Sanderson for creating Blazor. I thoroughly enjoy writing apps using this technology. I would also like to thank the numerous contributors of open source and .NET communities around the world. You’re inspiring—thank you!

Finally, I want to thank my family. Without the support of my amazing wife, Jennifer, none of this would have been possible. She encourages me to be the best possible version of myself. She’s believed in me far longer than I’ve believed in myself. I want to thank my three sons, Lyric, Londyn, and Lennyx. They’re a constant reminder of the future and the good we find in the world. Each child uniquely carries a little piece of inquisitive nature, curiosity, and joy. Without their spark and support, you wouldn’t be reading this right now. Thank you!

^([1](preface01.html#idm46365047943984-marker)) “Blazor: Build Client Web Apps with C#: .NET,” Microsoft, [*https://oreil.ly/iIaWE*](https://oreil.ly/iIaWE).