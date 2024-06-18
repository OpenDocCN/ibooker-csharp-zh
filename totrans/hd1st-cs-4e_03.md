# Chapter 2\. Dive into C#: *Statements, Classes, and Code*

![Images](assets/049fig01.png)

**You’re not just an IDE user. You’re a **developer**.**

You can get a lot of work done using the IDE, but there’s only so far it can take you. Visual Studio is one of the most advanced software development tools ever made, but a **powerful IDE** is only the beginning. It’s time to **dig in to C# code**: how it’s structured, how it works, and how you can take control of it…because there’s no limit to what you can get your apps to do.

(And for the record, you can be a **real developer** no matter what kind of keyboard you prefer. The only thing you need to do is **write code**!)

# Let’s take a closer look at the files for a console app

In the last chapter, you created a new .NET Core Console App project and named it MyFirstConsoleApp. When you did that, Visual Studio created two folders and three files.

![Images](assets/050fig01.png)

Let’s take a closer look at the Program.cs file that it created. Open it up in Visual Studio:

![Images](assets/050fig02.png)

*   At the top of the file is a `**using directive**`. You’ll see `using` lines like this in all of your C# code files.

*   Right after the `using` directives comes the `namespace` **keyword**. Your code is in a namespace called MyFirstConsoleApp. Right after it is an opening curly bracket `**{**`, and at the end of the file is the closing bracket `**}**`. Everything between those brackets is in the namespace.

*   Inside the namespace is a **class**. Your program has one class called Program. Right after the class declaration is an opening curly bracket, with its pair in the second-to-last line of the file.

*   Inside your class is a **method** called Main—again, followed by a pair of brackets with its contents.

*   Your method has one **statement**: `Console.WriteLine("Hello World!");`

## A statement performs one single action

Every method is made up of **statements** like your Console.WriteLine statement. When your program calls a method, it executes the first statement, then the next, then the next, etc. When the method runs out of statements—or hits a `**return**` statement—it ends, and the program execution resumes after the statement that originally called the method.

# Two classes can be in the same namespace (and file!)

Take a look at these two C# code files from a program called PetFiler2\. They contain three classes: a Dog class, a Cat class, and a Fish class. Since they’re all in the same PetFiler2 namespace, statements in the Dog.Bark method can call Cat.Meow and Fish.Swim ***without adding a*** `using` ***directive***.

![Images](assets/052fig01.png)

A class can span multiple files too, but you need to use the `partial` keyword when you declare it. It doesn’t matter how the various namespaces and classes are divided up between files. They still act the same when they’re run.

![Images](assets/052fig02.png)![Images](assets/053fig01.png)

**The IDE helps you build your code right.**

A long, long, LONG time ago, programmers had to use simple text editors like Windows Notepad or macOS TextEdit to edit their code. In fact, some their features would have been cutting-edge (like search and replace, or Notepad’s Ctrl+G for “go to line number” ). We had to use a lot of complex command-line applications to build, run, debug, and deploy our code.

Over the years, Microsoft (and, let’s be fair, a lot of other companies, and a lot of individual developers) figured out how to add *`many`* helpful things like error highlighting, IntelliSense, WYSIWYG click-and-drag window UI editing, automatic code generation, and many other features.

After years of evolution, Visual Studio is now one of the most advanced code-editing tools ever built. And lucky for you, it’s also a ***great tool for learning and exploring C# and app development***.

# Statements are the building blocks for your apps

Your app is made up of classes, and those classes contain methods, and those methods contain statements. So if we want to build apps that do a lot of things, we’ll need a few **different kinds of statements** to make them work. You’ve already seen one kind of statement:

[PRE0]

This is a **statement that calls a method**—specifically, the Console.WriteLine method, which prints a line of text to the console. We’ll also use a few other kinds of statements in this chapter and throughout the book. For example:

| ![Images](assets/055fig01.png) | We use variables and variable declarations to let our app store and work with data. |
| ![Images](assets/055fig02.png) | Lots of programs use math, so we use mathematical operators to add, subtract, multiply, divide, and more. |
| ![Images](assets/055fig03.png) | Conditionals let our code choose between options, either executing one block of code or another. |
| ![Images](assets/055fig04.png) | Loops let our code run the same block over and over again until a condition is satisfied. |

# Your programs use variables to work with data

Every program, no matter how big or how small, works with data. Sometimes the data is in the form of a document, or an image in a video game, or a social media update—but it’s all just data. That’s where **variables** come in. A variable is what your program uses to store data.

![Images](assets/056fig01.png)

## Declare your variables

Whenever you **declare** a variable, you tell your program its *`type`* and its *`name`*. Once C# knows your variable’s type, it will generate errors that stop your program from building if you try to do something that doesn’t make sense, like subtract `"Fido"` from `48353`. Here’s how to declare variables:

![Images](assets/056fig02.png)

> **Whenever your program needs to work with numbers, text, true/false values, or any other kind of data, you’ll use variables to keep track of them.**

## Variables vary

A variable is equal to different values at different times while your program runs. In other words, a variable’s value ***varies***. (Which is why “variable” is such a good name.) This is really important, because that idea is at the core of every program you’ll write. Say your program sets the variable `myHeight` equal to 63:

[PRE1]

Any time `myHeight` appears in the code, C# will replace it with its value, 63\. Then, later on, if you change its value to 12:

[PRE2]

C# will replace `myHeight` with 12 from that point onwards (until it gets set again)—but the variable is still called `myHeight`.

## You need to assign values to variables before you use them

Try typing these statements just below the “Hello World” statement in your new console app:

[PRE3]

***Do this!***

Go ahead, try it right now. You’ll get an error, and the IDE will refuse to build your code. That’s because it checks each variable to make sure that you’ve assigned it a value before you use it. The easiest way to make sure you don’t forget to assign your variables values is to combine the statement that declares a variable with a statement that assigns its value:

![Images](assets/057fig01.png)

## A few useful types

Every variable has a type that tells C# what kind of data it can hold. We’ll go into a lot of detail about the many different types in C# in [#types_and_references_getting_the_referen](ch04.html#types_and_references_getting_the_referen). In the meantime, we’ll concentrate on the three most popular types. `int` holds integers (or whole numbers), `string` holds text, and `bool` holds **Boolean** true/false values.

###### Note

var-i-a-ble, noun.

an element or feature likely to change. *Predicting the weather would be a whole lot easier if meteorologists didn’t have to take so many **variables** into account.*

> If you write code that uses a variable that hasn’t been assigned a value, your code won’t build. It’s easy to avoid that error by combining your variable declaration and assignment into a single statement.

###### Note

Once you’ve assigned a value to your variable, that value can change. So there’s no disadvantage to assigning a variable an initial value when you declare it.

# Generate a new method to work with variables

In the last chapter, you learned that Visual Studio will **generate code for you**. This is quite useful when you’re writing code and ***it’s also a really valuable learning tool***. Let’s build on what you learned and take a closer look at generating methods.

***Do this!***

1.  **Add a method to your new MyFirstConsoleApp project.**

    **Open the Console App project** that you created in the last chapter. The IDE created your app with a Main method that has exactly one statement:

    [PRE4]

    Replace this with a statement that calls a method:

    [PRE5]

2.  **Let Visual Studio tell you what’s wrong.**

    As soon as you finish replacing the statement, Visual Studio will draw a red squiggly underline beneath your method call. Hover your mouse cursor over it. The IDE will display a pop-up window:

    ![Images](assets/058fig01.png)

    Visual Studio is telling you two things: that there’s a problem—you’re trying to call a method that doesn’t exist (which will prevent your code from building)—and that it has a potential fix.

3.  **Generate the OperatorExamples method**.

    On **Windows**, the pop-up window tells you to press Alt+Enter or Ctrl+. to see the potential fixes. On **macOS**, it has a “Show potential fixes” link—press Option+Return to see the potential fixes. So go ahead and press either of those key combinations (or click on the dropdown to the left of the pop-up).

    ![Images](assets/058fig02.png)

    The IDE has a solution: it will generate a method called OperatorExamples in your Program class. **Click “Preview changes”** to display a window that has the IDE’s potential fix—adding a new method. Then **click Apply** to add the method to your code.

# Add code that uses operators to your method

Once you’ve got some data stored in a variable, what can you do with it? Well, if it’s a number, you might want to add or multiply it. If it’s a string, you might join it together with other strings. That’s where **operators** come in. Here’s the method body for your new OperatorExamples method. **Add this code to your program**, and read the `**comments**` to learn about the operators it uses.

![Images](assets/059fig01.png)

[PRE6]

###### Note

String variables hold text. When you use the + operator with strings it joins them together, so adding “abc” + “def” results in a single string, “abcdef” . When you join strings like that it’s called concatenation.

# Use the debugger to watch your variables change

When you ran your program earlier, it was executing in the **debugger**—and that’s an incredibly useful tool for understanding how your programs work. You can use **breakpoints** to pause your program when it hits certain statements and add **watches** to look at the value of your variables. Let’s use the debugger to see your code in action. We’ll use these three features of the debugger, which you’ll find in the toolbar:

![Images](assets/060fig01.png)

If you end up in a state you don’t expect, just use the Restart button (![Images](assets/060fig02.png)) to restart the debugger.

***Do this!***

1.  **Add a breakpoint and run your program.**

    Place your mouse cursor on the method call that you added to your program’s Main method and **choose Toggle Breakpoint (F9) from the Debug menu**. The line should now look like this:

    ![Images](assets/060fig03.png)

    ###### Note

    **The debugging shortcut keys for Mac are Step Over (![Images](assets/060fig03b.png)), Step In (![Images](assets/060fig03c.png)), and Step Out (![Images](assets/060fig03d.png)). The screens will look a little different, but the debugger operates exactly the same, as you saw in [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin) in the *Mac Learner’s Guide*.**

    Then press the ![Images](assets/060fig03a.png) button to run your program in the debugger, just like you did earlier.

2.  **Step into the method.**

    Your debugger is stopped at the breakpoint on the statement that calls the OperatorExamples method.

    ![Images](assets/060fig04.png)

    **Press *Step Into (F11)***—the debugger will jump into the method, then stop before it runs the first statement.

3.  **Examine the value of the width variable.**

    When you’re **stepping through your code**, the debugger pauses after each statement that it executes. This gives you the opportunity to examine the values of your variables. Hover over the `width` variable.

    ![Images](assets/060fig05.png)

    The IDE displays a pop-up that shows the current value of the variable—it’s currently 0\. Now **press Step Over (F10)**—the execution jumps over the comment to the first statement, which is now highlighted. We want to execute it, so **press Step Over (F10) again**. Hover over `width` again. It now has a value of 3.

4.  **The Locals window shows the values of your variables.**

    The variables that you declared are **local** to your OperatorExamples method—which just means that they exist only inside that method, and can only be used by statements in the method. Visual Studio displays their values in the Locals window at the bottom of the IDE when it’s debugging.

    ![Images](assets/061fig01.png)
5.  **Add a watch for the height variable.**

    A really useful feature of the debugger is the **Watch window**, which is typically in the same panel as the Locals window at the bottom of the IDE. When you hover over a variable, you can add a watch by right-clicking on the variable name in the pop-up window and choosing Add Watch. Hover over the `height` variable, then right-click and choose **Add Watch** from the menu.

    ![Images](assets/061fig02.png)

    Now you can see the `height` variable in the Watch window.

    ![Images](assets/061fig03.png)

    > **The debugger is one of the most important features in Visual Studio, and it’s a great tool for understanding how your programs work.**

6.  **Step through the rest of the method.**

    Step over each statement in OperatorExamples. As you step through the method, keep an eye on the Locals or Watch window and watch the values as they change. On **Windows,** press **Alt+Tab** before and after the Console.WriteLine statements to switch back and forth to the Debug Console to see the output. On **macOS**, you’ll see the output in the Terminal window so you don’t need to switch windows.

# Use operators to work with variables

Once you have data in a variable, what do you do with it? Well, most of the time you’ll want your code to do something based on the value. That’s where **equality operators**, **relational operators**, and **logical operators** become important:

**Equality Operators**

The == operator compares two things and is true if they’re equal.

The != operator works a lot like ==, except it’s true if the two things you’re comparing are not equal.

**Relational Operators**

Use > and < to compare numbers and see if a number in one variable one is bigger or smaller than another.

You can also use >= to check if one value is greater than or equal to another, and <= to check if it’s less than or equal.

**Logical Operators**

You can combine individual conditional tests into one long test using the && operator for ***and*** and the || operator for ***or***.

Here’s how you’d check if `i` equals 3 ***or*** `j` is less than 5:`(i == 3) || (j < 5)`

###### Note

**Use operators to compare two int variables**

You can do simple tests by checking the value of a variable using a comparison operator. Here’s how you compare two ints, x and y:

[PRE7]

These are the ones you’ll use most often.

# “if” statements make decisions

Use `**if**` **statements** to tell your program to do certain things only when the **conditions** you set up are (or aren’t) true. The `if` statement ***tests the condition*** and executes code if the test passed. A lot of `if` statements check if two things are equal. That’s when you use the `==` operator. That’s different from the single equals sign (=) operator, which you use to set a value.

![Images](assets/063fig01.png)![Images](assets/063fig02.png)

## if/else statements also do something if a condition isn’t true

`**if/else**` **statements** are just what they sound like: if a condition is true they do one thing ***or else*** they do the other. An `if/else` statement is an `if` statement followed by the `**else**` **keyword** followed by a second set of statements to execute. If the test is true, the program executes the statements between the first set of brackets. Otherwise, it executes the statements between the second set.

![Images](assets/063fig03.png)

# Loops perform an action over and over

Here’s a peculiar thing about most programs (*`especially`* games!): they almost always involve doing certain things over and over again. That’s what **loops** are for—they tell your program to keep executing a certain set of statements as long as some condition is true or false.

![Images](assets/064fig01.png)

## while loops keep looping statements while a condition is true

In a **while loop**, all of the statements inside the curly brackets get executed as long as the condition in the parentheses is true.

[PRE8]

## do/while loops run the statements then check the condition

A **do/while** loop is just like a while loop, with one difference. The while loop does its test first, then runs its statements only if that test is true. The do/while loop runs the statements first, ***then*** runs the test. So if you need to make sure your loop always runs at least once, a do/while loop is a good choice.

[PRE9]

## for loops run a statement after each loop

A **for loop** runs a statement after each time it executes a loop.

![Images](assets/064fig04.png)

###### Note

**The parts of the for statement are called the** initializer **(int i = 0), the** conditional test **(i < 8), and the** iterator **(i = i + 2). Each time through a for loop (or any loop) is called an** iteration.

**The conditional test always runs at the beginning of each iteration, and the iterator always runs at the end of the iteration.**

###### Note

When you use the for snippet, press Tab to switch between i and length. If you change the name of the variable i, the snippet will automatically change the other two occurrences of it.

###### Note

When we give you pencil-and-paper exercises, we’ll usually give you the solution on the next page.

# Use code snippets to help write loops

**Do this!**

You’ll be writing a lot of loops throughout this book, and Visual Studio can help speed things up for you with **snippets**, or simple templates that you can use to add code. Let’s use snippets to add a few loops to your OperatorExamples method.

If your code is still running, choose **Stop Debugging (Shift+F5)** from the Debug menu (or press the square Stop button ![Images](assets/067fig02.png) in the toolbar). Then find the line `Console.WriteLine(area);` in your OperatorExamples method. Click at the end of that line so your cursor is after the semicolon, then press Enter a few times to add some extra space. Now start your snippet. **Type** `**while**` **and press the Tab key twice.** The IDE will add a template for a while loop to your code, with the conditional test highlighted:

![Images](assets/067fig03.png)

Type `**area < 50**`—the IDE will replace `true` with the text. **Press Enter** to finish the snippet. Then add two statements between the brackets:

[PRE10]

###### Note

IDE Tip: Brackets

If your brackets (or braces, either name will do) don’t match up, your program won’t build, which leads to frustrating bugs. Luckily, the IDE can help with this! Put your cursor on a bracket, and the IDE highlights its match.

Next, use the `**do/while**` **loop snippet** to add another loop immediately after the while loop you just added. Type `**do**` **and press Tab twice**. The IDE will add this snippet:

![Images](assets/067fig04.png)

Type `area > 25` and press Enter to finish the snippet. Then add two statements between the brackets:

[PRE11]

Now **use the debugger** to really get a good sense of how these loops work:

1.  Click on the line just above the first loop and choose **Toggle Breakpoint (F9)** from the Debug menu to add a breakpoint. Then run your code and **press F5** to skip to the new breakpoint.

2.  Use **Step Over (F10)** to step through the two loops. Watch the Locals window as the values for `height`, `width`, and `area` change.

3.  Stop the program, then change the while loop test to `**area < 20**` so both loops have conditions that are false. Debug the program again. The while checks the condition first and skips the loop, but the do/while executes it once and then checks the condition.

Some useful things to keep in mind about C# code

*   **Don’t forget that all your statements need to end in a semicolon.**

    [PRE12]

*   **Add comments to your code by starting a line with two slashes.**

    [PRE13]

*   **Use /* and */ to start and end comments that can include line breaks.**

    [PRE14]

*   **Variables are declared with a *type* followed by a *name*.**

    [PRE15]

*   **Most of the time, extra whitespace is fine.**

    [PRE16]

*   **If/else, while, do, and for are all about testing conditions.**

    Every loop we’ve seen so far keeps running as long as a condition is true.

    ![Images](assets/069fig01.png)

**Then your loop runs forever.**

Every time your program runs a conditional test, the result is either `**true**` or `**false**`. If it’s `**true**`, then your program goes through the loop one more time. Every loop should have code that, if it’s run enough times, should cause the conditional test to eventually return `**false**`. If it doesn’t, then the loop will keep running until you kill the program or turn the computer off!

###### Note

This is sometimes called an **infinite loop**, and there are definitely times when you’ll want to use one in your code.

![Images](assets/070fig01.png)

**Definitely! Every program has its own kind of mechanics.**

There are mechanics at every level of software design. They’re easier to talk about and understand in the context of video games. We’ll take advantage of that to help give you a deeper understanding of mechanics, which is valuable for designing and building any kind of project.

Here’s an example. The mechanics of a game determine how hard or easy it is to play. Make Pac Man faster or the ghosts slower and the game gets easier. That doesn’t necessarily make it better or worse—just different. And guess what? The same exact idea applies to how you design your classes! You can think of ***how you design your methods and fields*** as the mechanics of the class. The choices you make about how to break up your code into methods or when to use fields make them easier or more difficult to use.

# Controls drive the mechanics of your user interfaces

In the last chapter, you built a game using TextBlock and Grid **controls**. But there are a lot of different ways that you can use controls, and the choices you make about what controls to use can really change your app. Does that sound weird? It’s actually really similar to the way we make choices in game design. If you’re designing a tabletop game that needs a random number generator, you can choose to use dice, a spinner, or cards. If you’re designing a platformer, you can choose to have your player jump, double jump, wall jump, or fly (or do different things at different times). The same goes for apps: if you’re designing an app where the user needs to enter a number, you can choose from different controls to let them do that—***and that choice affects how your user experiences the app***.

![Images](assets/071fig01.png)

*   A **text box** lets a user enter any text they want. But we need a way to make sure they’re only entering numbers and not just any text.

    ![Images](assets/071fig04.png)
*   **Radio buttons** let you restrict the user’s choice. You can use them for numbers if you want, and you can choose how you want to lay them out.

    ![Images](assets/071fig05.png)
*   The other controls on this page can be used for other types of data, but **sliders** are used exclusively to choose a number. Phone numbers are just numbers, too. So *`technically`* you could use a slider to choose a phone number. Do you think that’s a good choice?

    ![Images](assets/071fig02.png)
*   A **list box** gives users a way to choose from a list of items. If the list is long, it will show a scroll bar to make it easier for the user to find an item.

    ![Images](assets/071fig03.png)
*   A **combo box** combines the behavior of a list box and a text box. It looks like a normal text box, but when the user clicks it a list box pops up underneath it.

    ![Images](assets/071fig06.png)

###### Note

**Controls are common user interface (UI) components, the building blocks of your UI. The choices you make about what controls to use change the mechanics of your app.**

###### Note

We can borrow the idea of mechanics from video games to understand our options, so we can make great choices for any of our own apps—not just games.

![Images](assets/071fig07.png)

**The rest of this chapter contains a project to build a WPF desktop app for Windows. Go to the Visual Studio for Mac Learner’s Guide for the corresponding macOS project.**

# Create a WPF app to experiment with controls

If you’ve filled out a form on a web page, you’ve seen the controls we just showed you (even if you didn’t know all of their official names). Now let’s **create a WPF app** to get some practice using those controls. The app will be really simple—the only thing it will do is let the user pick a number, and display the number that was picked.

***Do this!***

![Images](assets/072fig01.png)![Images](assets/074fig02.png)

**“Save early, save often.”**

That’s an old saying from a time before video games had an autosave feature, and when you had to stick one of these in your computer to back up your projects…but it’s still great advice! Visual Studio makes it easy to add your project to source control and keep it up to date—so you’ll always be able to go back and see all the progress you’ve made.

![Images](assets/074fig03.png)

# Add a TextBox control to your app

A **TextBox control** gives your user a box to type text into, so let’s add one to your app. But we don’t just want a TextBox sitting there without a label, so first we’ll add a **Label control** (which is a lot like a TextBlock, except it’s specifically used to add labels to other controls).

1.  **Drag a Label out of the Toolbox into the top-left cell of the grid.**

    This is exactly how you added TextBlock controls to your animal matching game in [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin), except this time you’re doing it with a Label control. It doesn’t matter where in the cell you drag it, as long as it’s in the upper-left cell.

2.  **Set the text size and content of the Label.**

    While the Label control is selected, go to the Properties window, expand the Text section, and set the font size to `**18px**`. Then expand the Common section and set the Content to the text `Enter a number`.

    ![Images](assets/075fig01.png)![Images](assets/075fig02.png)
3.  **Drag the Label to the upper-left corner of the cell.**

    Click on the Label in the designer and drag it to the upper-left corner. When it’s 10 pixels away from the left or top cell wall, you’ll see gray bars appear and it will snap to a 10px margin.

    The XAML for your window should now contain a Label control:

    [PRE17]

    ![Images](assets/075fig03.png)
4.  **Drag a TextBox into the top-left cell of the grid.**

    Your app will have a TextBox positioned just underneath the Label so the user can type in numbers. Drag it so it’s on the left side and below the Label—the same gray bars will appear to position it just underneath the Label with a 10px left margin. Set its name to `**numberTextBox**`, font size to `**18px**`, and text to `**0**`.

    ![Images](assets/076fig01.png)

    Now run your app. Oops! Something went wrong—it threw an exception.

    ![Images](assets/077fig01.png)

    Take a look at the bottom of the IDE. It has an Autos window that shows you any defined variables.

    So what’s going on—***and, more importantly, how do we fix it?***

###### Note

Moving the TextBlock tag in the XAML so it’s above the TextBox, causes the TextBlock to get initialized first.

# Add C# code to update the TextBlock

In [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin) you added **event handlers**—methods that are called when a certain event is **raised** (sometimes we say the event is **triggered** or **fired**)—to handle mouse clicks in your animal matching game. Now we’ll add an event handler to the code-behind that’s called any time the user enters text into the TextBox and copies that text to the TextBlock that you added to the upper-right cell in the mini-exercise.

###### Note

**When you double-click on a TextBox control, the IDE adds an event handler for the TextChanged event that’s called any time the user changes its text. Double-clicking on other types of controls might add other event handlers—and in some cases (like with TextBlock) doesn’t add any event handlers at all.**

1.  **Double-click on the TextBox control to add the method.**

    As soon as you double-click on the TextBox, the IDE will **automatically add a C# event handler method** hooked up to its TextChanged event. It generates an empty method and gives it a name that consists of the name of the control (`numberTextBox`) followed by an underscore and the name of the event being handled—numberTextBox_TextChanged:

    [PRE18]

2.  **Add code to the new TextChanged event handler.**

    Any time the user enters text into the TextBox, we want the app to copy it into the TextBlock that you added to the upper-right cell of the grid. Since you gave the TextBlock a name (`number`) and you also gave the TextBox a name (`numberTextBox`), you just need one line of code to copy its contents:

    [PRE19]

    ###### Note

    This line of code sets the text in the TextBlock so it’s the same as the text in the TextBox, and it gets called any time the user changes the text in the TextBox.

3.  **Run your app and try out the TextBox.**

    Use the Start Debugging button (or choose Start Debugging (F5) from the Debug menu) to start your app, just like you did with the animal matching game in [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin). (If the runtime tools appear, you can disable them just like you did in [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin).) Type any number into the TextBox and it will get copied.

    ![Images](assets/078fig02.png)

    But something’s wrong—you can enter any text into the TextBox, not just numbers!

    ![Images](assets/078fig03.png)

# Add an event handler that only allows number input

When you added the MouseDown event to your TextBlock in [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin), you used the buttons in the upper-right corner of the Properties window to switch between properties and events. Now you’ll do the same thing, except this time you’ll use the **PreviewTextInput event** to only accept input that’s made up of numbers, and reject any input that isn’t a number.

If your app is currently running, stop it. Then go to the designer, click on the TextBox to select it, and switch the Properties window to show you its events. Scroll down and **double-click inside the box next to PreviewTextInput** to make the IDE generate an event handler method.

![Images](assets/079fig01.png)![Images](assets/079fig02.png)

Your new event handler method will have one statement in it:

[PRE20]

###### Note

You’ll learn all about int.TryParse later in the book—for now, just enter the code exactly as it appears here.

Here’s how this event handler works:

1.  The event handler is called when the user enters text into the TextBox, but ***before*** the TextBox is updated.

2.  It uses a special method called `int.TryParse` to check if the text that the user entered is a number.

3.  If the user entered a number, it sets `e.Handled` to `true`, which tells WPF to ignore the input.

Before you run your code, go back and look at the XAML tag for the TextBox:

[PRE21]

Now it’s hooked up to two event handlers: the TextChange event is hooked up to an event handler method called numberTextBox_TextChanged, and right below it the PreviewTextInput event is hooked up to a method called numberTextBox_PreviewTextInput.

# Add sliders to the bottom row of the grid

Let’s add two sliders to the bottom row and then hook up their event handlers so they update the TextBlock in the upper-right corner.

1.  **Add a slider to your app.**

    Drag a Slider out of the Toolbox and into the lower-right cell. Drag it to the upper-left corner of the cell and use the gray bars to give it left and top margins of 10.

    ![Images](assets/083fig01.png)![Images](assets/083fig02.png)

    Use the Common section of the Properties window to set AutoToolTipPlacement to `**TopLeft**`, Maximum to `**5**`, and Minimum to `**1**`. Give it the name `**smallSlider**`. Then double-click on the slider to add this event handler:

    [PRE22]

    ###### Note

    The value of the Slider control is a fractional number with a decimal point. This “0” converts it to a whole number.

2.  **Add a ridiculous slider to choose phone numbers.**

    There’s an old saying: *`“Just because an idea is terrible and also maybe stupid, that doesn’t mean you shouldn’t do it.”`* So let’s do something that’s just a bit stupid: add a slider to select phone numbers.

    ![Images](assets/083fig04.png)![Images](assets/083fig05.png)

    Drag another slider into the bottom row. Use the Layout section of the Properties window to **reset its width**, set its ColumnSpan to `**2**`, set all of its margins to `**10**`, and set its vertical alignment to `**Center**` and horizontal alignment to `**Stretch**`. Then use the Common section to set AutoToolTipPlacement to `**TopLeft**`, Minimum to `**1111111111**`, Maximum to `**9999999999**`, and Value to `**7183876962**`. Give it the name `**bigSlider**`. Then double-click on it and add this ValueChanged event handler:

    ![Images](assets/083fig06.png)

# Add C# code to make the rest of the controls work

You want each of the controls in your app to do the same thing: update the TextBlock in the upper-right cell with a number, so when you check one of the radio buttons or pick an item from a ListBox or ComboBox, the TextBlock is updated with whatever value you chose.

1.  **Add a Checked event handler to the first RadioButton control.**

    Double-click on the first RadioButton. The IDE will add a new event handler method called RadioButton_Checked (since you never gave the control a name, it just uses the type of control to generate the method). Add this line of code:

    [PRE23]

2.  **Make the other RadioButtons use the same event handler.**

    Look closely at the XAML for the RadioButton that you just modified. The IDE added the property `Checked="RadioButton_Checked"`—this is exactly like how the other event handlers were hooked up. **Copy this property to the other RadioButton tags** so they all have identical Checked properties—and ***now they’re all connected to the same Checked event handler***. You can use the Events view in the Properties window to check that each RadioButton is hooked up correctly.

    ![Images](assets/084fig02.png)
3.  **Make the ListBox update the TextBlock in the upper-right cell.**

    When you did the exercise, you named your ListBox `myListBox`. Now you’ll add an event handler that fires any time the user selects an item and uses the name to get the number that the user selected.

    Double-click inside the **empty** space in the ListBox **below** the items to make the IDE add an event handler method for the SelectionChanged event. Add this statement to it:

    ###### Note

    Make sure you click on the empty space below the list items. If you click on an item, it will add an event handler for that item and not for the entire ListBox.

    [PRE24]

4.  **Make the read-only combo box update the TextBlock.**

    Double-click on the read-only ComboBox to make Visual Studio add an event handler for the SelectionChanged event, which is raised any time a new item is selected in the ComboBox. Here’s the code—it’s really similar to the code for the ListBox:

    [PRE25]

    ###### Note

    You can also use the Properties window to add a SelectionChanged event. If you accidentally do this, you can hit “undo” (but make sure you do it in both files).

5.  **Make the editable combo box update the TextBlock.**

    An editable combo box is like a cross between a ComboBox and a TextBox. You can choose items from a list, but you can also type in your own text. Since it works like a TextBox, we can add a PreviewTextInput event handler to make sure the user can only type numbers, just like we did with the TextBox. In fact, you can **reuse the same event handler** that you already added for the TextBox.

    Go to the XAML for the editable ComboBox, put your cursor just before the closing caret `**>**` and **start typing *PreviewTextInput***. An IntelliSense window will pop up to help you complete the event name. Then **add an equals sign**—as soon as you do, the IDE will prompt you to either choose a new event handler or select the one you already added. Choose the existing event handler.

    ![Images](assets/085fig02.png)

    The previous event handlers used the list items to update the TextBlock. But users can enter any text they want into an editable ComboBox, so this time you’ll **add a different kind of event handler**.

    Edit the XAML again to add a new tag below `ComboBox`. This time, **type** `**TextBoxBase**`.—as soon as you type the period, the autocomplete will give suggestions. Choose **TextBoxBase.TextChanged** and type an equals sign. Now choose <New Event Handler> from the dropdown.

    ![Images](assets/085fig03.png)

    The IDE will add a new event handler to the code-behind. Here’s the code for it:

    [PRE26]

    ***Now run your program. All of the controls should work. Great job!***

![Images](assets/086fig01.png)

**Controls give you the flexibility to make things easy for your users.**

When you’re building the UI for an app, there are so many choices that you make: what controls to use, where to put each one, what to do with their input. Picking one control instead of another gives your users an *`implicit`* message about how to use your app. For example, when you see a set of radio buttons, you know that you need to pick from a small set of choices, while an editable combo box tells you that there your choices are nearly unlimited. So don’t think of UI design as a matter of making “right” or “wrong” choices. Instead, think of it as your way to make things as easy as possible for your users.