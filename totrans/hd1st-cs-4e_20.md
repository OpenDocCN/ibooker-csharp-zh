# Appendix A. ASP.NET Core Blazor projects: *Visual Studio for Mac Learner’s Guide*

![Images](assets/pg663.png)

**Your Mac is a first-class citizen of the C# and .NET world.**

We wrote *Head First C#* with our Mac readers in mind, and that’s why we created this special **learner’s guide** just for you. Most projects in this book are .NET Core console apps, which work on **both Windows and Mac**. Some chapters have a project built with a technology used for desktop Windows apps. This learner’s guide has **replacements** for all of those projects—including a *complete replacement for [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin)*—that use **C# to create Blazor WebAssembly apps** that run in your browser and are equivalent to the Windows apps. You’ll do it all with **Visual Studio for Mac**, a great tool for writing code and a **valuable learning tool** for exploring C#. Let’s dive right in and get coding!

# Why you should learn C#

C# is a simple, modern language that lets you do incredible things. When you learn C#, you’re learning more than just a language. C# unlocks the whole world of .NET, an incredibly powerful open-source platform for building all sorts of applications.

## Visual Studio is your gateway to C#

If you haven’t installed Visual Studio 2019 yet, this is the time to do it.

Go to [https://visualstudio.microsoft.com](https://visualstudio.microsoft.com) and **download Visual Studio for Mac**. (If it’s already installed, run the Visual Studio for Mac Installer to update your installed options.)

## Install .NET Core

Once you’ve downloaded the Visual Studio for Mac installer, run it to install Visual Studio. Make sure the **.NET Core** target is checked.

![Images](assets/pg664.png)

###### Note

Make sure you’re installing Visual Studio for Mac, and not installing Visual Studio Code.

###### Note

Visual Studio Code is an amazing open source, crossplatform code editor, but it’s not tailored to .NET development the way Visual Studio is. That’s why we can use Visual Studio throughout this book as a tool for learning and exploration.

## You can also use Visual Studio for Windows to build Blazor web applications

Most of the projects in *Head First C#* are .NET Core console apps, which you can create using either macOS or Windows. Some of the chapters also include a Windows desktop app project that’s built using Windows Presentation Foundation (WPF). Since WPF is a technology that only works with Windows, we wrote this *Visual Studio for Mac Learner’s Guide* so you can create equivalent projects on your Mac using web technologies—specifically, ASP.NET Core Blazor WebAssembly projects.

What if you’re a Windows reader and want to learn to build rich web applications using Blazor? Then you’re in luck! ***You can build the projects in this guide using Visual Studio for Windows***. Go to the Visual Studio installer and make sure that **“ASP.NET and web development” option is checked**. Your IDE screenshots won’t exactly match the ones in this guide, but all of the code will be the same.

# Visual Studio is a tool for writing code and exploring C#

You could use TextEdit or another text editor to write your C# code, but there’s a better way. An **IDE**—that’s short for ***integrated development environment***—is a text editor, visual designer, file manager, debugger...it’s like a multitool for everything you need to write code.

These are just a few of the things that Visual Studio helps you do:

1.  **Build an application, FAST.** The C# language is flexible and easy to learn, and the Visual Studio IDE makes it easier by doing a lot of manual work for you automatically. Here are just a few things that Visual Studio does for you:

    *   Manages all your project files

    *   Makes it easy to edit your project’s code

    *   Keeps track of your project’s graphics, audio, icons, and other resources

    *   Helps you debug your code by stepping through it line by line

        ![Images](assets/pg665.png)
2.  **Write and run your C# code.** The Visual Studio IDE is one of the easiest-to-use tools out there for writing code. The team at Microsoft who develop it have put a huge amount of work into making your job of writing code as easy as possible.

3.  **Build visually stunning web applications.** In this Visual Studio for Mac Learner’s Guide, you’ll be building web applications that run in your browser. You’ll use **Blazor**, a technology that lets you build interactive web apps using C#. When you **combine C# with HTML and CSS**, you’ve got an impressive toolkit for web development.

4.  **Learn and explore C# and .NET.** Visual Studio is a world-class development tool, but lucky for us it’s also a fantastic learning tool. ***We’re going to use the IDE to explore C#***, which gives us a fast track for getting important programming concepts into your brain.

    ###### Note

    We’ll often refer to Visual Studio as just “the IDE” throughout this book.

    > **Visual Studio is an amazing development environment, but we’re also going to use it as a learning tool to explore C#.**

# Create your first project in Visual Studio for Mac

The best way to learn C# is to start writing code, so we’re going to use Visual Studio to **create a new project**... and start writing code immediately!

###### Note

***Do this!***

###### Note

**When you see Do this! (or Now do this!, or Debug this!, etc.), go to Visual Studio and follow along. We’ll tell you exactly what to do, and point out what to look for to get the most out of the example we show you.**

1.  **Create a new Console Project.**

    Start up Visual Studio 2019 for Mac. When it starts, it shows you a window that lets you create a new project or open an existing one. **Click New** to create a new project. Don’t worry if you dismiss the window—you can always get it back by choosing *File >> New Solution...* (![Images](assets/pg666-3.png)) from the menu.

    ![Images](assets/pg666-1.png)

    **Select .NET** from the panel on the left, then choose **Console Project**:

    ![Images](assets/pg666-2.png)
2.  **Name your project MyFirstConsoleApp.**

    Enter **MyFirstConsoleApp** in the Project Name box and click the **Create button** to create the project.

    ![Images](assets/pg667-1.png)
3.  **Look at the code for your new app.**

    When Visual Studio creates a new project, it gives you a starting point that you can build on. As soon as it finishes creating the new files for the app, it opens and displays a file called *Program.cs* with this code:

    ![Images](assets/pg667-2.png)

# Use the Visual Studio IDE to explore your app

1.  **Explore the Visual Studio IDE—and the files that it created for you.**

    When you created the new project, Visual Studio created several files for you automatically and bundled them into a **solution**. The Solution window on the left side of the IDE shows you these files, with the solution (MyFirstConsoleApp) at the top. The solution contains a **project** that has the same name as the solution.

    ![Images](assets/pg668.png)
2.  **Run your new app.**

    The app that Visual Studio for Mac created for you is ready to run. At the top of the Visual Studio IDE, find the Run button (with a “play” triangle). **Click that button** to run your app:

    ![Images](assets/pg669-1.png)
3.  **Look at your program’s output.**

    When you run your program, the **Terminal window** will appear at the bottom of the IDE and display the output of the program:

    ![Images](assets/pg669-2.png)

    The best way to learn a language is to write a lot of code in it, so you’re going to build a lot of programs in this book. Many of them will be Console App projects, so let’s have a closer look at what you just did.

    At the top of the Terminal window is the **output of the program:**

    [PRE0]

    Click anywhere in the code to hide the Terminal window. Then press the ![Images](assets/pg669-3.png) button at the bottom of the IDE to open it again—you’ll see the same output from your program. The IDE automatically hides the Terminal window after your app exits.

    Press the Run button to run your program again. Then choose Start Debugging from the Run menu, or use its shortcut (![Images](assets/pg669-4.png)). This is how you’ll run all of the Console App projects throughout the book.

# Let’s build a game!

You’ve built your first C# app, and that’s great! Now that you’ve done that, let’s build something a little more complex. We’re going to build an **animal matching game**, where a player is shown a grid of 16 animals and needs to click on pairs to make them disappear.

![Images](assets/pg670.png)

## Your animal matching game is a Blazor WebAssembly app

Console apps are great if you just need to input and output text. If you want a visual app that’s displayed on a browser page, you’ll need to use a different technology. That’s why your animal matching game will be a **Blazor WebAssembly app**. Blazor lets you create rich web applications that can run in any modern browser. Most of the chapters in this book will feature a Blazor app. The goal of this project is to introduce you to Blazor and give you tools to build rich web applications as well as console apps.

###### Note

**Building different kinds of projects is an important tool in your C# learning toolbox. We chose Blazor for the Mac projects in this book because it gives you tools to design rich web applications that run on any modern browser.**

**But C# isn’t just for web development and console apps! Every project in this Mac learner’s guide has an equivalent Windows project.**

**Are you a Windows user, but still want to learn Blazor and build web applications with C#? Well then, you’re in luck! All of the projects in the Mac Learner’s Guide can also be done with Visual Studio for Windows.**

> **By the time you’re done with this project, you’ll be a lot more familiar with the tools that you’ll rely on throughout this book to learn and explore C#.**

# Here’s how you’ll build your game

The rest of this chapter will walk you through building your animal matching game, and you’ll be doing it in a series of separate parts:

1.  First you’ll create a new Blazor WebAssembly App project in Visual Studio.

2.  Then you’ll lay out the page and write C# code to shuffle the animals.

3.  The game needs to let the user click on pairs of emoji to match them.

4.  You’ll write more C# code to detect when the player has won the game.

5.  Finally, you’ll make the game more exciting by adding a timer.

###### Note

This project can take anywhere from 15 minutes to an hour, depending on how quickly you type. We learn better when we don’t feel rushed, so give yourself plenty of time.

![Images](assets/pg671.png)

###### Note

Keep an eye out for these “Game design... and beyond” elements scattered throughout the book. We’ll use game design principles as a way to learn and explore important programming concepts and ideas that apply to any kind of project, not just video games.

# Create a Blazor WebAssembly App in Visual Studio

The first step in building your game is to create a new project in Visual Studio.

1.  Choose **File >> New Solution... (![Images](assets/pg672-3a.png))** from the menu to bring up the New Project window. It’s the same way you started out your Console App project.

    ![Images](assets/pg672-1.png)

    Click **App** under “Web and Console” on the left, then choose **Blazor WebAssembly App** and click **Next**.

2.  The IDE will give you a page with options.

    ![Images](assets/pg672-2.png)

    Leave all of the options set to their default values and click **Next.**

    ![Images](assets/pg672-3.png)

    **If you run into any trouble with this project, go to our GitHub page and look for a link to a video walkthrough:**

    [https://github.com/head-first-csharp/fourth-edition](https://github.com/head-first-csharp/fourth-edition)

3.  Enter **BlazorMatchGame** as the project name, just like you did with your Console App project.

    ![Images](assets/pg673-1.png)

    Then **click Create** to create the project solution.

    ![Images](assets/pg673-2.png)
4.  The IDE will create a new BlazorMatchGame project and show you its contents, just like it did with your first console app. **Expand the Pages folder** in the Solution window to view its contents, then **double-click *Index.razor*** to open it in an editor.

    ![Images](assets/pg673-3.png)

# Run your Blazor web app in a browser

When you run a Blazor web app, there are two parts: a **server** and a **web application**. Visual Studio launches them both with one button.

###### Note

***Do this!***

1.  **Choose the browser to run your web application.**

    Find the triangle-shaped Run button at the top of the Visual Studio IDE:

    ![Images](assets/pg674-1.png)

    Your default browser should be listed next to `Debug >`. Click the browser name to see a dropdown of installed browsers and **choose either Microsoft Edge or Google Chrome**.

2.  **Run your web application.**

    Click the **Run button** to start your application. You can also choose Start Debugging ![Images](assets/pg674-2.png) from the Run menu. The IDE will first open a Build Output window (at the bottom, just like it opened the Terminal window), and then an Application Output window. After that, it will pop up a browser running your app.

    ![Images](assets/pg674-3.png)
3.  **Compare the code in *Index.razor* with what you see in your browser.**

    The web app in your browser has two parts: a **navigation menu** on the left side with links to different pages (Home, Counter, and Fetch data), and a page displayed on the right side. Compare the HTML markup in the *Index.razor* file with the app displayed in the browser.

    ![Images](assets/pg675-1.png)
4.  **Change “Hello, world!” to something else.**

    Change the third line of the *Index.razor* file so it says something else:

    [PRE1]

    Now go back to your browser and reload the page. Wait a minute, nothing changed—it still says “Hello, world!” That’s because you changed your code, ***but you never updated the server.***

    **Click the Stop button** ![Images](assets/pg675-2.png) or choose Stop ![Images](assets/pg675-3.png) from the Run menu. Now go back and reload your browser—since you stopped your app, it displays its “Site can’t be reached” page.

    **Start your app again**, then reload your page in the browser. Now you’ll see the updated text.

    ![Images](assets/pg675-4.png)

    **Do you have extra instances of your browser open? Visual Studio opens a new browser each time you run your Blazor web app. Get in the habit of closing the browser (![Images](assets/pg675-5.png)) before you stop your app (![Images](assets/pg675-6.png)).**

![Images](assets/pg676-1.png)

# Now you’re ready to start writing code for your game

You’ve created a new app, and Visual Studio generated a bunch of files for you. Now it’s time to add C# code to start making your game work (as well as HTML markup to make it look right).

![Images](assets/pg676-2.png)![Images](assets/pg677-1.png)

## How the page layout in your animal matching game will work

Your animal matching game is laid out in a grid—or, at least, that’s how it looks. It’s actually made up of 16 square buttons. If you make your browser very narrow, it will rearrange them so they’re in one long column.

![Images](assets/pg677-2.png)

You’ll lay out the page by creating a container that’s 400 pixels wide (a CSS “pixel” is 1/96 inch when the browser is at default scale) that contains 100-pixel-wide buttons. We’ll give you all of the C# and HTML code to enter into the IDE. **Keep an eye out for this code** that you’ll add to your project ***soon***—it’s where the “magic” happens, by mixing C# code with HTML:

![Images](assets/pg677-3.png)

# Visual Studio helps you write C# code

Blazor lets you create rich, interactive apps that combine HTML markup and C# code. Luckily, the Visual Studio IDE has useful features to help you write that C# code.

1.  **Add C# code to your Index.razor file.**

    Start by **adding a @code block** to the end of your *Index.razor* file. (Keep the existing contents of the file there for now—you’ll delete them later.) Go to the last line of the file and type `@code {`. The IDE will fill in the closing curly bracket `}` for you. Press Enter to add a line between the two brackets:

    ![Images](assets/pg678-1.png)
2.  **Use the IDE’s IntelliSense window to help you write C#.**

    Position your cursor on the line between the `{` brackets `}` and type the letter `**L**`. The IDE will pop up an **IntelliSense window** with autocomplete suggestions. Choose `List<>` from the pop-up:

    ![Images](assets/pg678-2.png)

    The IDE will fill in `List`. Add an **opening angle bracket** (greater-than sign) `<`—the IDE will automatically fill in the closing bracket `>` and leave your cursor positioned between them.

3.  **Start creating a List to store your animal emoji.**

    **Type s** to bring up another IntelliSense window:

    ![Images](assets/pg678-3.png)

    Choose `string`—the IDE will add it between the brackets. Press the **right arrow and then the space bar**, then **type** `animalEmoji = new`. Press the space bar again to pop up another IntelliSense window. **Press Enter** to choose the default value, `List<string>`, from the options.

    ![Images](assets/pg678-4.png)

    Your code should now look like this: `List<string> animalEmoji = new List<string>`

4.  **Finish creating the List of animal emoji.**

    Start by **adding a `@code` block** to the end of your *Index.razor* file. Go to the last line and **type `@code {.`** The IDE will fill in the closing curly bracket `}` for you. Press Enter to add a line between the brackets, then:

    *   Type an **opening parenthesis **(**—the IDE will fill in the closing one.**

    *   **Press the right arrow** to move past the parentheses.

    *   Type an **opening curly bracket** `{`—again, the IDE will fill in the closing one.

    *   Press Enter to add a line between the brackets, then **add a semicolon ;** after the closing bracket.

    The last six lines at the very bottom of your *Index.razor* file should now look like this:

    ![Images](assets/pg679-1.png)
5.  **Use the Character Viewer to enter emoji.**

    Next, **choose Edit >> Emoji & Symbols** (![Images](assets/pg679-2a.png)Space) from the menu to bring up the macOS Character Viewer. Position your cursor between the quotes, then go to the Character Viewer and **search for “dog”:**

    ![Images](assets/pg679-2.png)

    The last six lines at the bottom of your *Index.razor* file should now look like this:

    ![Images](assets/pg679-3.png)

# Finish creating your emoji list and display it in the app

You just added a dog emoji to your `animalEmoji` list. Now add a **second dog emoji** by adding a comma after the second quote, then a space, another quote, another dog emoji, another quote, and a comma:

![Images](assets/pg680-1.png)

Now **add a second line right after it** that’s exactly the same, except with a pair of wolf emoji instead of dogs. Then add six more lines with pairs of cows, foxes, cats, lions, tigers, and hamsters. You should now have eight pairs of emoji in your `animalEmoji` list:

![Images](assets/pg680-2.png)

## Replace the contents of the page

**Delete these lines** from the top of the page:

![Images](assets/pg680-3.png)

Then put your cursor on the third line of the page and **type** `<st`—the IDE will pop up an IntelliSense window:

![Images](assets/pg680-4.png)

Choose **`style`** from the list, then **type >**. The IDE will add a closing *HTML tag*: **`<style></style>`**

Put your cursor between `<style>` and `</style>` and press Enter, then **carefully enter all of the following code**. Make sure the code in your app matches it exactly.

![Images](assets/pg681-1.png)

Go to the next line and use the IntelliSense to **enter an opening and closing <div> tag**, just like you did with `<style>` earlier. Then **carefully enter the code below**, making sure it matches exactly:

![Images](assets/pg681-2.png)

###### Note

**Make sure your app looks like this screenshot when you run it. Once it does, you’ll know you entered all of the code without any typos.**

# Shuffle the animals so they’re in a random order

Our match game would be too easy if the pairs of animals were all next to each other. Let’s add C# code to shuffle the animals so they appear in a different order each time the player reloads the page.

1.  Place your cursor just after the semicolon `;` just above the closing bracket `}` near the bottom of *Index.razor* and **press Enter twice**. Then use the IntelliSense pop-ups just like you did earlier to enter the following line of code:

    [PRE2]

2.  Next **type `protected override`** (the IntelliSense can autocomplete those keywords). As soon as you enter that and type a space, you’ll get an IntelliSense pop-up—**select OnInitialized()** from the list:

    ![Images](assets/pg682-1.png)

    The IDE will fill in code for a **method** called OnInitialized (we’ll talk more about methods in [#dive_into_chash_statementscomma_classesc](ch02.html#dive_into_chash_statementscomma_classesc)):

    ![Images](assets/pg682-2a.png)
3.  **Replace `base.OnInitialized() with SetUpGame()`** so your method looks like this:

    [PRE3]

    Then **add this SetUpGame method** just below your OnInitialized method—again, the IntelliSense window will help you get it right:

    ![Images](assets/pg682-2.png)

    As you type in the SetUpGame method, you’ll notice that the IDE pops up many IntelliSense windows to help you enter your code more quickly. The more you use Visual Studio to write C# code, the more helpful these windows will become—you’ll eventually find that they significantly speed things up. For now, use them to keep from entering typos—your code needs to ***match our code exactly*** or your app won’t run.

4.  Scroll back up to the HTML and find this code: `@foreach (var animal in animalEmoji)`

    **Double-click `animalEmoji`** to select it, then **type s**. The IDE will pop up an IntelliSense window. Choose `shuffledAnimals` from the list:

    ![Images](assets/pg683-1.png)

    Now **run your app again**. Your animals should be shuffled so they’re in a random order. **Reload the page** in the browser—they’ll be shuffled in a different order. Each time you reload, it reshuffles the animals.

    ![Images](assets/pg683-2.png)

    ###### Note

    **Again, make sure your app looks like this screenshot when you run it. Once it does, you’ll know you entered all of the code without any typos. Don’t move on until your game is reshuffling the animals every time you reload the browser page.**

# You’re running your game in the debugger

When you click the Run button ![Images](assets/pg684-1.png) or choose Start Debugging ![Images](assets/pg684-2.png) from the Run menu to start your program running, you’re putting Visual Studio into **debugging mode**.

You can tell that you’re debugging an app when you see the **debug controls** appear in the toolbar. The Start button has been replaced with the square Stop button ![Images](assets/pg684-3.png), the dropdown to choose which browser to launch is grayed out, and an extra set of controls has appeared.

Hover your mouse cursor over the Pause Execution button to see its tooltip:

![Images](assets/pg684-4.png)

You can stop your app clicking the Stop button or choosing Stop ![Images](assets/pg684-5.png) from the Run menu.

![Images](assets/pg684-6.png)

**You’ve set the stage for the next part that you’ll add.**

When you build a new game, you’re not just writing code. You’re also running a project. A really effective way to run a project is to build it in small increments, taking stock along the way to make sure things are going in a good direction. That way you have plenty of opportunities to change course.

###### Note

Here’s a pencil-and-paper exercise. It’s absolutely worth your time to do all of them because they’ll help get important C# concepts into your brain faster.

![Images](assets/pg687.png)

**Working on your code comprehension skills will make you a better developer.**

The pencil-and-paper exercises are **not optional**. They give your brain a different way to absorb the information. But they do something even more important: they give you opportunities to ***make mistakes***. Making mistakes is a part of learning, and we’ve all made plenty of mistkaes (you may even find one or two typos in this book!). Nobody writes perfect code the first time—really good programmers always assume that the code that they write today will probably need to change tomorrow. In fact, later in the book you’ll learn about *refactoring*, or programming techniques that are all about improving your code after you’ve written it.

###### Note

We’ll add bullet points like this to give a quick summary of many of the ideas and tools that you’ve seen so far.

# Add your new project to source control

You’re going to be building a lot of different projects in this book. Wouldn’t it be great if there was an easy way to back them up and access them from anywhere? What if you make a mistake—wouldn’t it be super convenient if you could roll back to a previous version of your code? Well, you’re in luck! That’s exactly what **source control** does: it gives you an easy way to back up all of your code, and keeps track of every change that you make. Visual Studio makes it easy for you to add your projects to source control.

**Git** is a popular version control system, and Visual Studio will publish your source to any Git **repository** (or **repo**). We think **GitHub** is one of the easiest Git providers to use. You’ll need a GitHub account to push code to it, so if you don’t already have one, go to [https://github.com](https://github.com) and create it now.

Once you have your GitHub account set up, you can use the built-in version control features of the IDE. **Choose *Version Control >> Publish in Version Control...* from the menu** to bring up the Clone Repository window:

![Images](assets/pg688.png)

###### Note

**The Visual Studio for Mac documentation has a complete guide to creating projects on GitHub and publishing them from Visual Studio. It includes step-by-step instructions for creating a remote repo on GitHub and publishing your projects to Git directly from Visual Studio. We think it’s a great idea to publish all of your *Head First C#* projects to GitHub—that way you can easily go back to them later. [https://docs.microsoft.com/en-us/visualstudio/mac/set-up-git-repository](https://docs.microsoft.com/en-us/visualstudio/mac/set-up-git-repository).**

![Images](assets/pg689-1.png)

# Add C# code to handle mouse clicks

You’ve got buttons with random animal emoji. Now you need them to do something when the player clicks them. Here’s how it will work:

![Images](assets/pg689-2.png)

# Add click event handlers to your buttons

When you click a button, it needs to do something. In web pages, a click is an **event**. Web pages have other events, too, like when a page finishes loading, or when an input changes. An **event handler** is C# code that gets executed any time a specific event happens. We’ll add an event handler that implements the button functionality.

## Here’s the code for the event handler

Add this code to the bottom of your Razor page, just above the closing `**}**` at the bottom:

![Images](assets/pg690-1.png)

## Hook up your event handler to the buttons

Now you just need to modify your buttons to call the ButtonClick method when clicked:

![Images](assets/pg690-2.png)

###### Note

**When we ask you to update one thing in a block of code, we might make the rest of the code a lighter shade and make the part of the code you change boldface**.

###### Note

**Uh-oh—there’s a bug in this code! Can you spot it? We’ll track it down and fix it in the next section.**

# Test your event handler

Run your app again. When it comes up, test your event handler by clicking on a button, then clicking on the button with the matching emoji. They should both disappear.

![Images](assets/pg692-1.png)

Click on another, then another, then another. You should be able to keep clicking on pairs until all of the buttons are blank. Congratulations, you found all the pairs!

![Images](assets/pg692-2.png)

## But what happens if you click on the same button twice?

Reload the page in your browser to reset the game. But this time instead of finding a pair, **click twice on the same button**. Hold on—***there’s a bug in the game!*** It should have ignored the click, but instead, it acted like you got a match.

![Images](assets/pg692-3.png)

# Use the debugger to troubleshoot the problem

You might have heard the word “bug” before. You might have even said something like this to your friends at some point in the past: “That game is really buggy, it has so many glitches.” Every bug has an explanation—everything in your program happens for a reason—but not every bug is easy to track down.

***Understanding a bug is the first step in fixing it.*** Luckily, the Visual Studio debugger is a great tool for that. (That’s why it’s called a debugger: it’s a tool that helps you get rid of bugs!)

1.  **Think about what’s going wrong.**

    The first thing to notice is that your bug is **reproducible**: any time you click on the same button twice, it always acts like you clicked a matching pair.

    The second thing to notice is that you have a **pretty good idea** where the bug is. The problem only happened *after* you added the code to handle the Click event, so that’s a great place to start.

2.  **Add a breakpoint to the Click event handler code that you just wrote.**

    Click on the first line of the ButtonClick method and **choose Run >> Toggle Breakpoint** (![Images](assets/box1.png)) from the menu. The line will change color and you’ll see a dot in the left margin:

    ![Images](assets/pg693-1.png)

# Keep debugging your event handler

Now that your breakpoint is set, use it to get a handle on what’s going on with your code.

3.  **Click on an animal to trigger the breakpoint.**

    If your app is already running, stop it and close all browser windows. Then **run your app** again and **click any animal button**. Visual Studio should pop into the foreground. The line where you toggled the breakpoint should now be highlighted in a different color:

    ![Images](assets/pg694-1.png)

    Move your mouse to the first line of the method, which starts `private void`, and **hover your cursor over animal**. A small window will pop up that shows you the animal that you clicked on:

    ![Images](assets/pg694-2.png)

    Press the **Step Over** button or choose Run >> Step Over (![Images](assets/pg693-2.png)) from the menu. The highlight will move down to the `**{**` line. Step over again to move the highlight to the next statement:

    ![Images](assets/pg694-3.png)

    Step over one more time to execute that statement, then hover over `lastAnimalFound`:

    ![Images](assets/pg694-4.png)

    The statement that you stepped over set the value of `lastAnimalFound` so it matches `animal`.

    ***That’s how the code keeps track of the first animal that the player clicked.***

4.  **Continue execution.**

    Press the **Continue Execution** button or choose Run >> Continue Debugging (![Images](assets/pg694-5.png)) from the menu. Switch back to the browser—your game will keep going until it hits the breakpoint again.

5.  **Click the matching animal in the pair.**

    Find the button with the matching emoji and **click it**. The IDE will trigger the breakpoint and pause the app again. Press **Step Over**—it will skip the first block and jump to the second:

    ![Images](assets/pg695-1.png)

    **Hover over `lastAnimalFound` and `animal`**—they should both have the same emoji. That’s how the event handler knows that you found a match. **Step over three more times**:

    ![Images](assets/pg695-2.png)

    Now **hover over `shuffledAnimals`. You’ll see several items in the window that pops up. Click the triangle next to `shuffledAnimals` to expand it, then **expand** `_items` to see all the animals:**

    ![Images](assets/pg695-3.png)

    Press **Step Over** one more time to execute the statement that removes the matches from the list. Then **hover over** `**shuffledAnimals**` **again** and look at its items. There are now two (*null*) values where the matched emoji used to be:

    ![Images](assets/pg695-4.png)

    **We’ve sifted through a lot of evidence and gathered some important clues. What do you think is causing the problem?**

# Track down the bug that’s causing the problem...

So what happens if you click the same animal button twice? Let’s find out! **Repeat the same steps you just did**, except this time **click the same animal twice**. Watch what happens when you get to step ![Images](assets/pg696a.png).

Hover over `animal` and `lastAnimalFound`, just like you did before. They’re the same! That’s because the event handler ***doesn’t have a way to distinguish between different buttons with the same animal***.

## ...and fix the bug!

Now that we know what’s causing the bug, we know how to fix it: give the event handler a way to distinguish between the two buttons with the same emoji.

First, **make these changes** to the ButtonCick event handler (make sure you don’t miss any changes):

![Images](assets/pg696-1.png)

Then **replace the `foreach loop`** with a different kind of loop, a `for` loop—this for loop counts the animals:

![Images](assets/pg696-2.png)

Now debug through the app again, just like you did before. This time when you click the same animal twice it will skip down to the end of the event handler. ***The bug is fixed!***

![Images](assets/pg698-1.png)

# Add code to reset the game when the player wins

The game is coming along—your player starts out with a grid full of animals to match, and they can click on pairs of animals that disappear when they’re matched. But what happens when all of the matches are found? We need a way to reset the game so the player gets another chance.

![Images](assets/pg698-2.png)

###### Note

When you see a Brain Power element, take a minute and really think about the question that it’s asking.

![Images](assets/pg701-1.png)

# Finish the game by adding a timer

Your animal matching game will be more exciting if players can try to beat their best time. We’ll add a **timer** that “ticks” after a fixed interval by repeatedly calling a method.

![Images](assets/pg701-3.png)![Images](assets/pg701-2.png)

###### Note

**Timers “tick” every time interval by calling methods over and over again. You’ll use a timer that starts when the player starts the game and ends when the last animal is matched.**

# Add a timer to your game’s code

***Add this!***

1.  Start by finding this line at the very top of the *Index.razor* file: `@page "/"`

    **Add this line just below it**—you need it in order to use a Timer in your C# code:

    [PRE4]

2.  **You’ll need to update the HTML markup to display the time. Add this just below the first block that you added in the exercise:**

    [PRE5]

3.  Your page will need a timer. It will also need to keep track of the elapsed time:

    [PRE6]

4.  You need to tell the timer how frequently to “tick” and what method to call. You’ll do this in the OnInitialized method, which is called once after the page is loaded:

    [PRE7]

5.  Reset the timer when you set up the game:

    [PRE8]

6.  You need to stop and start your timer. Add this line of code near the top of the ButtonClick method to start the timer when the player clicks the first button:

    [PRE9]

    And finally, add these two lines further down in the ButtonClick method to stop the timer and display a “Play Again?” message after the player finds the last match:

    [PRE10]

7.  Finally, your timer needs to know what to do each time it ticks. Just like buttons have Click event handlers, timers have Tick event handlers: methods that get executed every time the timer ticks.

    **Add this code at the very bottom of the page**, just above the closing bracket `}`:

    [PRE11]

###### Note

**The timer starts when the player clicks the first animal and stops when the last match is found. This doesn’t fundamentally change the way the game works, but makes it more exciting.**

# Clean up the navigation menu

Your game is working! But did you notice that there are other pages in your app? Try clicking on “Counter” or “Fetch data” in the navigation menu on the left side. When you created the Blazor WebAssembly App project, Visual Studio added these additional sample pages. You can safely remove them.

Start by expanding the **wwwroot** **folder** and editing *index.html*. Find the line that starts `<title>` and **modify it** so it looks like this: `<title> Animal Matching Game </title>`

Next, expand the **Shared folder** in the solution and **double-click *NavMenu.razor***. Find this line:

[PRE12]

and **replace it with this:**

[PRE13]

Then **delete these lines:**

![Images](assets/pg704-1.png)

Finally, hold down ![Images](assets/pg704-2.png) (Command) and **click to multiselect these files** in the Solution window: ***Counter.razor*** and ***FetchData.razor*** in the Pages folder, ***SurveyPrompt.razor*** in the Shared folder, and the **entire sample-data** folder inside the wwwroot folder. Once they’re all selected, right-click on one of them and **choose Delete** (![Images](assets/pg704-3.png)) from the menu to delete them.

***And now your game is done!***

![Images](assets/pg704.png)

**Whenever you have a large project, it’s always a good idea to break it into smaller pieces.**

One of the most useful programming skills that you can develop is the ability to look at a large and difficult problem and break it down into smaller, easier problems.

It’s really easy to be overwhelmed at the beginning of a big project and think, “Wow, that’s just so...big!” But if you can find a small piece that you can work on, then you can get started. Once you finish that piece, you can move on to another small piece, and then another, and then another. As you build each piece, you learn more and more about your big project along the way.

# Even better ifs...

Your game is pretty good! But every game—in fact, pretty much every program—can be improved. Here are a few things that we thought of that could make the game better:

*   Add different kinds of animals so the same ones don’t show up each time.

*   Keep track of the player’s best time so they can try to beat it.

*   Make the timer count down instead of counting up so the player has a limited amount of time.

###### Note

We’re serious—take a few minutes and do this. Stepping back and thinking about the project you just finished is a great way to seal the lessons you learned into your brain.

![Images](assets/pg705.png)

# *from [#dive_into_chash_statementscomma_classesc](ch02.html#dive_into_chash_statementscomma_classesc)* dive into C#

###### Note

This is the Blazor version of the Windows desktop project in [#dive_into_chash_statementscomma_classesc](ch02.html#dive_into_chash_statementscomma_classesc).

###### Note

**The last part of [#dive_into_chash_statementscomma_classesc](ch02.html#dive_into_chash_statementscomma_classesc) is a Windows project to experiment with different kinds of controls. We’ll use Blazor to build a similar project to experiment with web controls.**

# Controls drive the mechanics of your user interfaces

In the last chapter, you built a game using Button **controls**. But there are a lot of different ways that you can use controls, and the choices you make about what controls to use can really change your app. Does that sound weird? It’s actually really similar to the way we make choices in game design. If you’re designing a tabletop game that needs a random number generator, you can choose to use dice, a spinner, or cards. If you’re designing a platformer, you can choose to have your player jump, double jump, wall jump, or fly (or do different things at different times). The same goes for apps: if you’re designing an app where the user needs to enter a number, you can choose from different control to let them do that—***and that choice affects how your user experiences the app.***

![Images](assets/pg706-1.png)

*   A **text box** lets a user enter any text they want. But we need a way to make sure they’re only entering numbers and not just any text.

    ![Images](assets/pg706-2.png)
*   **Sliders** are used exclusively to choose a number. Phone numbers are just numbers, too, so *technically* you could use a slider to choose a phone number. Do you think that’s a good choice?

    ![Images](assets/pg706-3.png)
*   **Pickers** are controls that are specially built to pick a specific type of value from a list. For example, **date pickers** let you specify a date by picking its year, month, and day, and **color pickers** let you choose a color using a spectrum slider or by its numeric value.

    ![Images](assets/pg706-4.png)
*   **Radio buttons** let you restrict the user’s choice. They often look like circles with dots in them, but you can style them to look like regular buttons too.

    ![Images](assets/pg706-5.png)![Images](assets/pg706-6.png)

# Create a new Blazor WebAssembly App project

Earlier in this ***Visual Studio for Mac Learner’s Guide***, you created a Blazor WebAssembly App project for your animal matching game. You’ll do the same thing for this project, too.

***Here’s a concise set of steps to follow to create a Blazor WebAssembly App project, change the title text for the main page, and remove the extra files that Visual Studio creates. We won’t repeat this for every additional project in this guide—you should be able to follow these same instructions for all of the future Blazor WebAssembly App projects.***

1.  **Create a new Blazor WebAssembly App project.**

    Either start up Visual Studio 2019 for Mac or choose *File >> New Solution...* (![Images](assets/pg707-01.png)) from the menu to **bring up the New Project window. Click New** to create a new project. Name it **ExperimentWithControlsBlazor**.

2.  **Change the title and navigation menu.**

    At the end of the animal matching game project, you modified the title and navigation bar text. Do the same for this project. Expand the **wwwroot folder** and edit *Index.html*. Find the line that starts `<title>` and **modify it** so it looks like this: `<title> **Experiment with Controls** </title>`

    Expand the **Shared folder** in the solution and **double-click on *NavMenu.razor*.** Find this line:

    [PRE14]

    and **replace it with this:**

    [PRE15]

3.  **Remove the extra navigation menu options and their corresponding files.**

    This is just like what you did at the end of the animal matching game project. **Double-click on *NavMenu.razor* and delete these lines:**

    ![Images](assets/pg707.png)

    Then hold down ![Images](assets/pg707-1.png) (Command) and **click to multiselect these files** in the Solution window: ***Counter.razor*** and ***FetchData.razor*** in the Pages folder, ***SurveyPrompt.razor*** in the Shared folder, and the **entire sample-data** folder inside the wwwroot folder. Once they’re all selected, right-click on one of them and **choose Delete** ( ![Images](assets/pg707-2.png) ) from the menu to delete them.

# Create a page with a slider control

Many of your programs will need the user to input numbers, and one of the most basic controls to input a number is a **slider**, also known as a **range input**. Let’s create a new Razor page that uses a slider to update a value.

1.  **Replace the Index.razor page.**

    Open *Index.razor* and **replace** all of its contents with **this HTML markup:**

    ![Images](assets/pg708-1.png)
2.  **Run your app.**

    Run your app just like you did with the app in [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin). Compare the HTML markup with the page displayed in the browser—match up the individual `<div>` blocks with what’s displayed on the page.

    ![Images](assets/pg708-2.png)
3.  **Add C# code to your page.**

    Go back to *Index.razor* and **add this C# code** to the bottom of the file:

    ![Images](assets/pg709-1.png)
4.  **Hook up your range control to the Change event handler you just added.**

    Add an `@onchange` attribute to your range control:

    ![Images](assets/pg709-2.png)

# Add a text input to your app

The goal of this project is to experiment with different kinds of controls, so let’s add a **text input control** so users can type text into the app and have it display at the bottom of the page.

1.  **Add a text input control to your page’s HTML markup.**

    **Add an** `**<input ... />**` **tag** that’s almost identical to the one you added for the slider. The only difference is that you’ll set the `type` attribute to `"text"` instead of `"range"`. Here’s the HTML markup:

    ![Images](assets/pg710-1.png)

    **Run your app again**—now it has a text input control. Any text you enter will show up at the bottom of the page. Try changing the text, then moving the slider, then changing the text again. The value at the bottom will change each time you modify a control.

    ![Images](assets/pg710-2.png)
2.  **Add an event handler method that only accepts numeric values.**

    What if you only want to accept numeric input from your users? **Add this method** to the code between the brackets at the bottom of the Razor page:

    ![Images](assets/pg711-1.png)
3.  **Change the text input to use the new event handler method.**

    Modify your text control’s `@onchange` attribute to call the new event handler:

    [PRE16]

    Now try entering text into the text input—it won’t update the value at the bottom of the page unless the text that you enter is an integer value.

![Images](assets/pg712.png)

# Add color and date pickers to your app

Pickers are just different types of inputs. A **date picker** has the input type `"date"` and a **color picker** has the input type `"color"`—other than that, the HTML markup for those input types is identical.

Modify your app to **add a date picker and a color picker**. Here’s the HTML markup—add it just above the `<div>` tag that contains the display value:

![Images](assets/pg713-1.png)![Images](assets/pg713-2.png)

###### Note

**That’s the end of the project—great job! You can pick up [#dive_into_chash_statementscomma_classesc](ch02.html#dive_into_chash_statementscomma_classesc) at the very end, where there’s a person in a chair thinking this: THERE ARE SO MANY DIFFERENT WAYS FOR USERS TO CHOOSE NUMBERS!**

# from [#objectshellipget_orientedexclamation_mar](ch03.html#objectshellipget_orientedexclamation_mar) objects...get oriented!

###### Note

This is the Blazor version of the Windows desktop project in [#objectshellipget_orientedexclamation_mar](ch03.html#objectshellipget_orientedexclamation_mar).

###### Note

**Partway through [#objectshellipget_orientedexclamation_mar](ch03.html#objectshellipget_orientedexclamation_mar), there’s a project where you build a Windows version of the card picker app. We’ll use Blazor to build a web-based version of the same app.**

# Up next: build a Blazor version of your card picking app

In the next project, you’ll build a Blazor app called PickACardBlazor. It will use a slider to let you choose the number of random cards to pick and display those cards in a list. Here’s what it will look like:

![Images](assets/pg714.png)

## Reuse your CardPicker class in a new Blazor app

***Reuse this!***

If you’ve written a class for one program, you’ll often want to use the same behavior in another. That’s why one of the big advantages of using classes is that they make it easier to **reuse** your code. Let’s give your card picker app a shiny new user interface, but keep the same behavior by reusing your CardPicker class.

1.  **Create a new Blazor WebAssembly App project called PickACardBlazor.**

    You’ll follow exactly the same steps you used to create your animal matching game in [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin):

    *   Open Visual Studio and create a new project.

    *   Select **Blazor WebAssembly App**, just like you did with your previous Blazor apps.

    *   Name your new app **PickACardBlazor**. Visual Studio will create the project.

2.  **Add the CardPicker class that you created for your Console App project.**

    Right-click on the project name and choose **Add >> Existing Files...** from the menu:

    ![Images](assets/pg715-1.png)

    Navigate to the folder with your console app and **click on** ***CardPicker.cs*** to add it to your project. Visual Studio will ask you if you want to copy, move, or link to the file. Tell Visual Studio to **copy the file**. Your project should now have a copy of the *CardPicker.cs* file from your console app.

3.  **Change the namespace for the CardPicker class.**

    **Double-click on *CardPicker.cs*** in the Solution window. It still has the namespace from the console app. **Change the namespace** to match your project name:

    ![Images](assets/pg715-2.png)

    ***Congratulations, you’ve reused your CardPicker class!*** You should see the class in the Solution window, and you’ll be able to use it in the code for your Blazor app.

# The page is laid out with rows and columns

The Blazor apps in [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin) and [#dive_into_chash_statementscomma_classesc](ch02.html#dive_into_chash_statementscomma_classesc) used HTML markup to create rows and columns, and this new app does the same thing. Here’s a picture that shows you how your app will be laid out:

![Images](assets/pg716.png)

# The slider uses data binding to update a variable

The code at the bottom of the page will start with a variable called `numberOfCards`:

[PRE17]

You *could* use an event handler to update `numberOfCards`, but Blazor has a better way: **data binding**, which lets you set up your input controls to automatically update your C# code, and can automatically insert values from your C# code back into the page.

Here’s the HTML markup for the header, the range input, and the text next to it that shows its value:

![Images](assets/pg717-1.png)

Take a closer look at the attributes for the `input` tag. The `min` and `max` attributes restrict the input to values from 1 to 15\. The `**@bind**` attribute sets up the data binding, so any time the slider changes Blazor automatically updates `numberOfCards`.

The `input` tag is followed by `<div class="col-2">**@numberOfCards</div>**`—that markup adds text (with `ml-2` adding space to the left margin). This also uses data binding, but to go in the other direction: every time the `numberOfCards` field is updated, Blazor automatically updates the text inside that `div` tag.

###### Note

**That’s the end of the project—nice work! You can go back to [#objectshellipget_orientedexclamation_mar](ch03.html#objectshellipget_orientedexclamation_mar) and resume at the section with this heading: *Ana’s prototypes look great...***

# from [#types_and_references_getting_the_referen](ch04.html#types_and_references_getting_the_referen) types and references

###### Note

This is the Blazor version of the Windows desktop project in [#types_and_references_getting_the_referen](ch04.html#types_and_references_getting_the_referen).

**At the end of [#types_and_references_getting_the_referen](ch04.html#types_and_references_getting_the_referen) there’s a Windows project. We’ll build a Blazor version of it.**

## Welcome to Sloppy Joe’s Budget House o’ Discount Sandwiches!

Sloppy Joe has a pile of meat, a whole lotta bread, and more condiments than you can shake a stick at. What he doesn’t have is a menu! Can you build a program that makes a new *random* menu for him every day? You definitely can... with a **new Blazor WebAssembly App project**, some arrays, and a couple of useful new techniques.

***Do this!***

![Images](assets/pg720.png)

1.  **Add a new MenuItem class to your project and add its fields.**

    Have a look at the class diagram. It has six fields: an instance of Random, three arrays to hold the various sandwich parts, and string fields to hold the description and price. The array fields use **collection initializers**, which let you define the items in an array by putting them inside curly braces.

    [PRE18]

2.  **Add the Generate method to the MenuItem class.**

    This method uses the same Random.Next method you’ve seen many times to pick random items from the arrays in the Proteins, Condiments, and Breads fields and concatenate them together into a string.

    [PRE19]

    ###### Note

    **The Generate method makes a random price between 2.01 and 5.97 by converting two random ints to decimals. Have a close look at the last line—it returns `price.ToString("c")`. The parameter to the ToString method is a format. In this case, the `"c"` format tells ToString to format the value with the local currency: if you’re in the United States you’ll see a $; in the UK you’ll get a £, in the EU you’ll see €, etc.**

3.  **Add the page layout to your Index.razor file.**

    The menu page is made up of a series of Bootstrap rows, one for each menu item. Each row has two columns, a `col-9` with the menu item description and a `col-3` with the price. There’s one last row on the bottom with a centered `col-6` for the guacamole.

    ![Images](assets/pg721.png)

###### Note

**That’s the end of the project! Resume at the Bullet Points at the very end of [#types_and_references_getting_the_referen](ch04.html#types_and_references_getting_the_referen).**