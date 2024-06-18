# Chapter 12\. Exception Handling: *Putting out Fires Gets old*

![Images](assets/pg623.png)

**Programmers aren’t meant to be firefighters.**

You’ve worked your tail off, wading through technical manuals and a few engaging *Head First* books, and you’ve reached the pinnacle of your profession. But you’re still getting panicked phone calls in the middle of the night from work because **your program crashes**, or **doesn’t behave like it’s supposed to**. Nothing pulls you out of the programming groove like having to fix a strange bug...but with **exception handling**, you can write code to **deal with problems** that come up. Better yet, you can even plan for those problems, and **keep things running** when they happen.

# Your hex dumper reads a filename from the command line

At the end of [#reading_and_writing_files_save_the_last](ch10.html#reading_and_writing_files_save_the_last) you built a hex dumper that uses command-line arguments to dump any file. You used the project properties in the IDE to set the arguments for the debugger, and you saw how to call it from a Windows command prompt or macOS Terminal window.

![Images](assets/pg624-1.png)

## But what happens if you give HexDump an invalid filename?

When you modified your HexDump app to use command-line arguments, we asked you to be careful to specify a valid filename. What happens when you give it an invalid filename? Try running your app again from the command line, but this time give it the argument **`invalid-filename`**. Now it ***throws an exception.***

![Images](assets/pg624-2.png)

Use the project settings to set the program’s argument to an invalid filename and run the app in the IDE’s debugger. Now you’ll see it throw an exception with the same class name (System.IO.FileNotFoundException) and a similar “Could not find file” message.

![Images](assets/pg624-3.png)

###### Note

**You’d never actually get all these exceptions in a row—the program would throw the first exception and then halt. You’d only get to the second exception if you fixed the first.**

# When your program throws an exception, the CLR generates an Exception object

You’ve been looking at the CLR’s way of telling you something went wrong in your program: an **exception**. When an exception occurs in your code, an object is created to represent the problem. It’s called—no surprise here—Exception.

For example, suppose you have an array with four items, and you try to access the 16th item (index 15, since we’re zero-based here):

![Images](assets/pg628.png)

###### Note

ex-cep-tion, noun.

a person or thing that is excluded from a general statement or does not follow a rule. *While Jamie usually hates peanut butter, they made an **exception** for Parker’s peanut butter fudge.*

When the IDE halts because the code threw an exception, you can see the details of the exception by **expanding** **`$exception`** **in the Locals window**. The Locals window shows you all of the variables currently ***in scope*** (which means the current statement has access to them).

The CLR goes to the trouble of creating an object because it wants to give you all the information it has about what caused the exception. You may have code to fix, or you may just need to make some changes to how you handle a particular situation in your program.

This particular exception is an **IndexOutOfRangeException**, which gives you an idea of what the problem is: you’re trying to access an index in the array that’s out of range. You’ve also got information about exactly where in the code the problem occurred, making it easier to track down and solve (even if you’ve got thousands of lines of code).

# All Exception objects inherit from System.Exception

.NET has lots of different exceptions it may need to report. Since many of these have a lot of similar features, inheritance comes into play. .NET defines a base class, called Exception, that all specific exception types inherit from.

The Exception class has a couple of useful members. The Message property stores an easy-to-read message about what went wrong. StackTrace tells you what code was being executed when the exception occurred, and what led up to the exception. (There are others, too, but we’ll use those first.)

![Images](assets/pg629.png)![Images](assets/pg631.png)

**That’s right. Exceptions are a really useful tool that you can use to find places where your code acts in ways you don’t expect.**

A lot of programmers get frustrated the first time they see an exception. But exceptions are really useful, and you can use them to your advantage. When you see an exception, it’s giving you a lot of clues to help you figure out why your code is reacting in a way that you didn’t anticipate. That’s good for you: it lets you know about a new scenario that your program has to handle, and it gives you an opportunity to **do something about it.**

> **Exceptions are all about helping you find and fix situations where your code behaves in ways you didn’t expect.**

# There are some files you just can’t dump

In [#linq_and_lambdas_get_control_of_your_dat](ch09.html#linq_and_lambdas_get_control_of_your_dat) we talked about making your code **robust** so that it can deal with bad data, malformed input, user errors, and other unexpected situations. Dumping stdin if no file is passed on the command line or if the file doesn’t exist is a great start to making the hex dumper robust.

But are there still some cases that we need to handle? For example, what if the file exists but it’s not readable? Let’s see what happens if we remove read access from a file, then try to read it:

*   ***On Windows:*** Right-click the file in the Windows Explorer, go to the Security tab, and click Edit to change the permissions. Check all of the Deny boxes.

    ![Images](assets/pg632-1.png)
*   ***On a Mac:*** In a Terminal window, change to the folder with the file you want to dump, and run this command, replacing `binarydata.dat` with the name of your file): `chmod 000 binarydata.dat.`

Now that you’ve removed read permissions from your file, try running your app again, either in the IDE or from the command line.

You’ll see an exception—the stack trace shows that the **`using` statement called the GetInputStream method**, which eventually caused the FileStream to throw a System.UnauthorizedAccessException:

[PRE0]

![Images](assets/pg632-2.png)

**Actually, there *is* something you can do about it.**

Yes, users do, indeed, screw up all the time. They’ll give you bad data, weird input, click on things you didn’t even know existed. That’s a fact of life, but that doesn’t mean you can’t do anything about it. C# gives you really useful **exception handling tools** to help you make your programs more robust. Because while you *can’t* control what your users do with your app, you *can* make sure that your app doesn’t crash when they do it.

# What happens when a method you want to call is risky?

Users are unpredictable. They feed all sorts of weird data into your program, and click on things in ways you never expected. That’s just fine, because you can deal with exceptions that your code throws by adding **exception handling**, which lets you write special code that gets executed any time an exception is thrown.

1.  **Let’s say a method called in your program takes input from a user.**

    ![Images](assets/pg633-1.png)
2.  **That method could do something risky that might not work at runtime.**

    ![Images](assets/pg633-2.png)![Images](assets/pg633-3.png)
3.  **You need to know that the method you’re calling is risky.**

    ![Images](assets/pg633-4.png)

    ###### Note

    If you can come up with a way to do a less risky thing that avoids throwing the exception, that’s the best possible outcome! But some risks just can’t be avoided, and that’s when you want to do this.

4.  **You can then write code that can *handle* the exception if it does happen. You need to be prepared, just in case.**

    ![Images](assets/pg633-5.png)

# Handle exceptions with try and catch

When you add exception handling to your code, you’re using the `try` and `catch` keywords to create a block of code that gets run if an exception is thrown.

Your *try/catch* code basically tells the C# compiler: “**Try** this code, and if an exception occurs, **catch** it with this *other* bit of code.” The part of the code you’re trying is the `try` **block**, and the part where you deal with exceptions is called the `catch` **block**. In the `catch` block, you can do things like print a friendly error message instead of letting your program come to a halt.

Let’s have another look at the last three lines of the stack trace in our HexDump scenario to help us figure out where to put our exception handling code:

[PRE1]

The UnauthorizedAccessException is caused by the line in GetInputStream that calls File.OpenRead. Since we can’t prevent that exception, let’s modify GetInputStream to use a *try/catch* block:

![Images](assets/pg634.png)

We kept things simple in our exception handler. First we used Console.Error to write a line to the error output (stderr) letting the user know that an error occurred, then we fell back to reading data from standard input so the program still did something. Notice how **the `catch` block has a `return` statement**. The method returns a Stream, so if it handles an exception it still needs to return a Stream; otherwise you’ll get the “not all code paths return a value” compiler error.

# Use the debugger to follow the try/catch flow

An important part of exception handling is that when a statement in your `try` block throws an exception, the rest of the code in the block gets **short-circuited**. The program’s execution immediately jumps to the first line in the `catch` block. Let’s use the IDE’s debugger to explore how this works.

***Debug this!***

1.  Replace the GetInputStream method in your HexDump app with the one that we just showed you to handle an UnauthorizedAccessException.

2.  Modify your project options to set the argument so that it contains the path to an unreadable file.

3.  Place a breakpoint on the first statement in GetInputStream, then start debugging your project.

4.  When it hits the breakpoint, step over the next few statements until you get to File.OpenRead. Step over it—the app jumps to the first line in the `catch` block.

    ![Images](assets/pg635.png)
5.  Keep stepping through the rest of the `catch` block. It will write the line to the console, then return Console.OpenStandardInput and resume the Main method.

# If you have code that ALWAYS needs to run, use a finally block

When your program throws an exception, a couple of things can happen. If the exception ***isn’t*** handled, your program will stop processing and crash. If the exception ***is*** handled, your code jumps to the `catch` block. What about the rest of the code in your `try` block? What if you were closing a stream, or cleaning up important resources? That code needs to run, even if an exception occurs, or you’re going to make a mess of your program’s state. That’s where you’ll use a `**finally block**`. It’s a block of code that comes after the `try` and `catch`. The `**finally**` block always runs, whether or not an exception was thrown. Let’s use the debugger to explore how the `finally` block works.

***Debug this!***

1.  **Create a new Console App project.**

    Add `using System.IO;` to the top of the file, then add this Main method:

    ![Images](assets/pg636.png)

    ###### Note

    You’ll see the exception in the Locals window, just like you did earlier.

2.  **Add a breakpoint to the first line of the Main method.**

    Debug your app and step through it. The first line in the `try` block tries to access `args[0]`, but since you didn’t specify any command-line arguments the `args` array is empty and it throws an exception—specifically, a System.**IndexOutOfRangeException**, with the message *“Index was outside the bounds of the array.”* After it prints the message, it **executes the** `**finally**` **block**, and then the program exits.

3.  **Set a command-line argument with the path of a valid file.**

    Use the project properties to pass a command-line argument to the app. Give it the full path of a valid file. Make sure there are no spaces in the filename, otherwise the app will interpret it as two arguments. Debug your app again—after it finishes the `try` block, it **executes the** `**finally**` **block.**

4.  **Set a command-line argument with an invalid file path.**

    Go back to the project properties and change the command-line argument to pass the app the name of a file that does not exist. Run your app again. This time it catches a different exception: a System.IO.**FileNotFoundException**. Then it **executes the `finally` block.**

# Catch-all exceptions handle System.Exception

You just made your console app throw two different kinds of exception—an IndexOutOfRangeException and a FileNotFoundException—and they were both handled. Take a closer look at the `catch` block:

[PRE2]

This is a **catch-all exception**: the type after the `catch` block indicates what type of exception to handle, and since all exceptions extend the System.Exception class, specifying `Exception` as the type tells the `try/catch` block to catch any exception.

## Avoid catch-all exception with multiple catch blocks

It’s always better to try to anticipate the specific exceptions that your code will throw and handle them. For example, we know that this code can throw an IndexOutOfRange exception if no filename is specified, or a FileNotFound exception if an invalid file is found. We also saw earlier in the chapter that trying to read an unreadable file causes the CLR to throw an UnauthorizedAccessException. You can handle these different kinds of exceptions by adding **multiple `catch` blocks** to your code:

![Images](assets/pg637.png)

Now your app will write different error messages depending on which exception is handled. Notice that the first two `catch` blocks **did not specify a variable name** (like `ex`). You only need to specify a variable name if you’re going to use the Exception object.

# Pool Puzzle

Your ***job*** is to take code snippets from the pool and place them into the blank lines in the program. You can use the same snippet more than once, and you won’t need to use all the snippets. Your ***goal*** is to make the program produce the output below.

![Images](assets/pg639.png)

# Pool Puzzle Solution

![Images](assets/pg640.png)![Images](assets/pg641-1.png)

**Unhandled exceptions bubble up.**

Believe it or not, it can be really useful to leave exceptions unhandled. Real-life programs have complex logic, and it’s often difficult to recover correctly when something goes wrong, especially when a problem occurs very far down in the program. By only handling specific exceptions and avoiding catch-all exception handlers, you let unexpected exceptions ***bubble up***: instead of being handled in the current method, they’re caught by the next statement up the call stack. Anticipating and handling the exceptions that you expect and letting unhandled exceptions bubble up is a great way to build more robust apps.

Sometimes it’s useful to **rethrow** an exception, which means that you handle an exception in a method but *still bubble it up* to the statement that called it. All you need to do to rethrow an exception is call `throw;` inside a `catch` block, and the exception that it caught will immediately bubble up:

![Images](assets/pg641-2.png)

###### Note

Here’s a career tip: a lot of C# programming job interviews include a question about how you deal with exceptions in a constructor.

# Use the right exception for the situation

When you use the IDE to generate a method, it adds code that looks like this:

[PRE3]

The **NotImplementedException** is used any time you have an operation or method that hasn’t been implemented. It’s a great way to add a placeholder—as soon as you see one you know there’s code that you need to write. That’s just one of the many exceptions that .NET provides.

Choosing the right exception can make your code easier to read, and make exception handling cleaner and more robust. For example, code in a method that validates its parameters can throw an ArgumentException, which has an overloaded constructor with a parameter to specify which argument caused the problem. Consider the Guy class back in [#objectshellipget_orientedexclamation_mar](ch03.html#objectshellipget_orientedexclamation_mar), had a ReceiveCash method that checked the `amount` parameter to make sure it was receiving a positive amount. This is a good opportunity to throw an ArgumentException:

![Images](assets/pg642-1.png)

Take a minute and look over the list of exceptions that are part of the .NET API—you can throw any of them in your code: [https://docs.microsoft.com/en-us/dotnet/api/system.systemexception](https://docs.microsoft.com/en-us/dotnet/api/system.systemexception).

## Catch custom exceptions that extend System.Exception

Sometimes you want your program to throw an exception because of a special condition that could happen when it runs. Let’s go back to the Guy class from [#objectshellipget_orientedexclamation_mar](ch03.html#objectshellipget_orientedexclamation_mar). Suppose you’re using it in an app that absolutely depended on a Guy always having a positive amount of cash. You could add a custom exception that **extends System.Exception**:

![Images](assets/pg642-2.png)

Now you can throw that new exception, and catch it exactly like you’d catch any other exception:

![Images](assets/pg642-3.png)

# Exception Magnets

![Images](assets/Common-11.png)

Arrange the magnets so the application writes the following output to the console:

**when it thaws it throws.**

[PRE4]

![Images](assets/pg643.png)

# Exception Magnets Solution

![Images](assets/Common-11.png)

Arrange the magnets so the application writes the following output to the console:

**when it thaws it throws.**

[PRE5]

![Images](assets/pg644.png)![Images](assets/pg645-2.png)

# Exception filters help you create precise handlers

Let’s say we’re building a game set in classic 1930s mafia gangland, and we’ve got a LoanShark class that needs to collect cash from instances of Guy using the Guy.GiveCash method, and that handles any OutOfCashException with a good old gangland-style lesson.

The thing is, every loan shark knows the golden rule: don’t try to collect money from the big mob boss. That’s where **exception filters** can come in really handy. An exception filter uses the `when` keyword to tell your exception handler to catch an exception only under specific conditions.

Here’s an example of how an exception filter works:

![Images](assets/pg646-1.png)![Images](assets/pg646-2.png)![Images](assets/pg646-3.png)

**It’s always better to build the most precise exception handlers that we can.**

There’s more to exception handling than just printing out a generic error message. Sometimes you want to handle different exceptions differently—like how the hex dumper handled a FileNotFoundException differently from an UnauthorizedAccessException. Planning for exceptions always involves ***unexpected situations***. Sometimes those situations can be prevented, sometimes you want to handle them, and sometimes you want the exceptions to bubble up. A big lesson to learn here is that there’s no “one-size-fits-all” approach to dealing with the unexpected, which is why the IDE doesn’t just wrap everything in a `try/catch` block.

###### Note

This is why there are so many classes that inherit from Exception, and why you may even want to write your own classes to inherit from Exception.

# The worst catch block EVER: catch-all plus comments

A `catch` block will let your program keep running if you want it to. An exception gets thrown, you catch the exception, and instead of shutting down and giving an error message, you keep going. But sometimes, that’s not such a good thing.

Take a look at this `Calculator` class, which seems to be acting funny all the time. What’s going on?

![Images](assets/pg648.png)

## You should handle your exceptions, not bury them

Just because you can keep your program running doesn’t mean you’ve *handled* your exceptions. In the code above, the calculator won’t crash...at least, not in the Divide method. What if some other code calls that method, and tries to print the results? If the divisor was zero, then the method probably returned an incorrect (and unexpected) value.

Instead of just adding a comment and burying the exception, you need to **handle the exception**. If you’re not able to handle the problem, ***don’t leave empty or commented `catch` blocks!*** That just makes it harder for someone else to track down what’s going on. It’s better to let the program continue to throw exceptions, because then it’s easier to figure out what’s going wrong.

###### Note

Remember, when your code doesn’t handle an exception, the exception bubbles up the call stack. Letting an exception bubble up is a perfectly valid way of dealing with an exception, and in some cases it makes more sense to do that than to use an empty catch block to bury the exception.

# Temporary solutions are OK (temporarily)

Sometimes you find a problem, and know it’s a problem, but aren’t sure what to do about it. In these cases, you might want to log the problem and note what’s going on. That’s not as good as handling the exception, but it’s better than doing nothing.

Here’s a temporary solution to the calculator problem:

###### Note

...but in real life, “temporary” solutions have a nasty habit of becoming permanent.

###### Note

**Take a minute and think about this `catch` block. What happens if the `StreamWriter` can’t write to the C:\Logs\ folder? You can nest another `try/catch` block inside it to make it less risky. Can you think of a better way to do this?**

![Images](assets/pg649-1.png)![Images](assets/pg649-2.png)

**Handling exceptions doesn’t always mean the same thing as FIXING exceptions.**

It’s never good to have your program bomb out. It’s way worse to have no idea why it’s crashing or what it’s doing to users’ data. That’s why you need to be sure that you’re always dealing with the errors you can predict and logging the ones you can’t. While logs can be useful for tracking down problems, preventing those problems in the first place is a better, more permanent solution.