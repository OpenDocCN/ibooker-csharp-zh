# Chapter 10\. Reading and writing files: *Save the last byte for me!*

![Images](assets/pg529.png)

**Sometimes it pays to be a little persistent.**

So far, all of your programs have been pretty short-lived. They fire up, run for a while, and shut down. But that’s not always enough, especially when you’re dealing with important information. You need to be able to **save your work**. In this chapter, we’ll look at how to **write data to a file**, and then how to **read that information back in** from a file. You’ll learn about **streams**, and how to store your objects in files with **serialization**, and get down to the actual bits and bytes of **hexadecimal**, **Unicode**, and **binary data**

# .NET uses streams to read and write data

A **stream** is the .NET Framework’s way of getting data into and out of your program. Any time your program reads or writes a file, connects to another computer over a network, or generally does anything where it **sends or receives bytes**, you’re using streams. Sometimes you’re using streams directly , other times indirectly. Even when you’re using classes that don’t directly expose streams, under the hood they’re almost always using streams.

> **Whenever you want to read data from a file or write data to a file, you’ll use a Stream object.**

###### Note

**Let’s say you have a simple app that needs to read data from a file. A really basic way to do that is to use a `Stream` object.**

![Images](assets/pg530-1.png)

###### Note

**And if your app needs to write data out to the file, it can use another `Stream` object.**

![Images](assets/pg530-2.png)

# Different streams read and write different things

Every stream is a subclass of the **abstract Stream class**, and there are many subclasses of Stream that do different things. We’ll be concentrating on reading and writing regular files, but everything you learn about streams in this chapter can apply to compressed or encrypted files, or network streams that don’t use files at all.

![Images](assets/pg531.png)

## Things you can do with a stream:

1.  **Write to the stream.**

    You can write your data to a stream through a stream’s **Write method.**

2.  **Read from the stream.**

    You can use the **Read method** to get data from a file, or a network, or memory, or just about anything else using a stream. You can even read data from ***really big*** files, even if they’re too big to fit into memory.

3.  **Change your position within the stream.**

    Most streams support a `**Seek method**` that lets you find a position within the stream so you can read or insert data at a specific place. However, not every Stream class supports Seek—which makes sense, because you can’t always backtrack in some sources of streaming data.

> **Streams let you read and write data. Use the right kind of stream for the data you’re working with.**

# A FileStream reads and writes bytes in a file

When your program needs to write a few lines of text to a file, there are a lot of things that have to happen:

1.  Create a new FileStream object and tell it to write to the file.

    ![Images](assets/pg532-1.png)
2.  The FileStream attaches itself to a file.

    ![Images](assets/pg532-2.png)
3.  Streams write bytes to files, so you’ll need to convert the string that you want to write to an array of bytes.

    ![Images](assets/pg532-3.png)
4.  Call the stream’s Write method and pass it the byte array.

    ![Images](assets/pg532-4.png)
5.  Close the stream so other programs can access the file.

    ![Images](assets/pg532-5.png)

# Write text to a file in three simple steps

C# comes with a convenient class called **StreamWriter** that simplifies those things for you. All you have to do is create a new StreamWriter object and give it a filename. It ***automatically*** creates a FileStream and opens the file. Then you can use the StreamWriter’s Write and WriteLine methods to write everything to the file you want.

> **StreamWriter creates and manages a FileStream object for you automatically.**

1.  **Use the StreamWriter’s constructor to open or create a file.**

    You can pass a filename to the StreamWriter’s constructor. When you do, the writer automatically opens the file. StreamWriter also has an overloaded constructor that lets you specify its *append* mode: passing it `true` tells it to add data to the end of an existing file (or append), while `false` tells the stream to delete the existing file and create a new file with the same name.

    [PRE0]

    ![Images](assets/pg533-1.png)
2.  **Use the Write and WriteLine methods to write to the file.**

    These methods work just like the ones in the Console class: Write writes text, and WriteLine writes text and adds a line break to the end.

    [PRE1]

    ![Images](assets/pg533-2.png)
3.  **Call the Close method to release the file.**

    If you leave the stream open and attached to a file, then it’ll keep the file locked and no other program will be able to use it. So make sure you always close your files!

    [PRE2]

# The Swindler launches another diabolical plan

The citizens of Objectville have long lived in fear of the Swindler, Captain Amazing’s archnemesis. Now he’s using a StreamWriter to implement another evil plan. Let’s take a look at what’s going on. Create a new Console App project and **add this Main code**, starting with a `using` declaration because StreamWriter is in the **System.IO namespace**:

###### Note

**StreamWriter’s Write and WriteLine methods work just like Console’s: Write writes text, and WriteLine writes text with a line break. Both classes support {curly brackets} like this:**

[PRE3]

**When you include {0} in the text, it’s replaced by the first parameter after the string; {1} is replaced by the second, {2} by the third, etc.**

![Images](assets/pg534-1.png)![Images](assets/pg534-2.png)

###### Note

The Swindler is Captain Amazing’s arch-nemesis, a shadowy supervillain bent on the domination of Objectville.

![Images](assets/pg534-3.png)

Here’s the output that it writes to *secret_plan.txt*:

**Output**

[PRE4]

# StreamWriter Magnets

![Images](assets/pg535a.png)

Oops! These magnets were nicely arranged on the fridge with the code for the Flobbo class, but someone slammed the door and they all fell off. Can you rearrange them so the Main method produces the output below?

[PRE5]

**We added an extra challenge.**

Something weird is going on with the Blobbo method. See how it has two different declarations in the first two magnets? We defined Blobbo as an **overloaded method**—there are two different versions, each with its own parameters, just like the **overloaded methods** you’ve used in previous chapters.

![Images](assets/pg535.png)

# StreamWriter Magnets Solution

![Images](assets/pg535a.png)

Your job was to construct the Flobbo class from the magnets to create the desired output.

[PRE6]

![Images](assets/pg536.png)

###### Note

**Just a reminder: we picked intentionally weird variable names and methods in these puzzles because if we used really good names, the puzzles would be too easy! Don’t use names like this in your code, OK?**

# Use a StreamReader to read a file

Let’s read the Swindler’s secret plans with **StreamReader**, a class that’s a lot like StreamWriter—except instead of writing a file, you create a StreamReader and pass it the name of the file to read in its constructor. Its ReadLine method returns a string that contains the next line from the file. You can write a loop that reads lines from it until its EndOfStream field is true—that’s when it runs out of lines to read. Add this console app that uses a StreamReader to read one file, and a StreamWriter to write another file:

###### Note

**StreamReader is a class that reads characters from streams, but it’s not a stream itself. When you pass a filename to its constructor, it creates a stream for you, and closes it when you call its Close method. It also has an overloaded constructor that takes a reference to a Stream.**

![Images](assets/pg537.png)

###### Note

**Output**

[PRE7]

# Data can go through more than one stream

One big advantage to working with streams in .NET is that you can have your data go through more than one stream on its way to its final destination. One of the many types of streams in .NET Core is the CryptoStream class. This lets you encrypt your data before you do anything else with it. So instead of writing plain text to a regular old text file:

![Images](assets/pg538-1.png)![Images](assets/pg538-2.png)

the Swindler can **chain streams together** and send the text through a CryptoStream object before writing its output to a FileStream:

![Images](assets/pg538-3.png)

> **You can CHAIN streams. One stream can write to another stream, which writes to another stream...often ending with a network or file stream.**

# Pool Puzzle

![Images](assets/pg539-1.png)

Your ***job*** is to take code snippets from the pool and place them into the blank lines in the Pineapple, Pizza, and Party classes. You can use the same snippet more than once, and you won’t need to use all the snippets. Your ***goal*** is to make the program write a file called *order.txt* with the five lines listed in the output box below.

![Images](assets/pg539-2.png)

# Pool Puzzle Solution

![Images](assets/pg540.png)

# Use the static File and Directory classes to work with files and directories

Like StreamWriter, the File class creates streams that let you work with files behind the scenes. You can use its methods to do most common actions without having to create the FileStreams first. The Directory class lets you work with whole directories full of files.

## Things you can do with the static File class:

1.  **Find out if the file exists.**

    You can check to see if a file exists using the File.Exists method. It’ll return true if it does, and false if it doesn’t.

2.  **Read from and write to the file.**

    You can use the File.OpenRead method to get data from a file, or the File.Create or File.OpenWrite method to write to the file.

3.  **Append text to the file.**

    The File.AppendAllText method lets you append text to an already created file. It even creates the file if it’s not there when the method runs.

4.  **Get information about the file.**

    The File.GetLastAccessTime and File.GetLastWriteTime methods return the date and time when the file was last accessed and modified.

![Images](assets/pg542.png)

## Things you can do with the static Directory class:

1.  **Create a new directory.**

    Create a directory using the Directory.CreateDirectory method. All you have to do is supply the path; this method does the rest.

2.  **Get a list of the files in a directory.**

    You can create an array of files in a directory using the Directory.GetFiles method; just tell the method which directory you want to know about, and it will do the rest.

3.  **Delete a directory.**

    Need to delete a directory? Call the Directory.Delete method.

# IDisposable makes sure objects are closed properly

A lot of .NET classes implement a particularly useful interface called IDisposable. It **has only one member**: a method called Dispose. Whenever a class implements IDisposable, it’s telling you that there are important things that it needs to do in order to shut itself down, usually because it’s **allocated resources** that it won’t give back until you tell it to. The Dispose method is how you tell the object to release those resources.

## Use the IDE to explore IDisposable

You can use the Go To Definition feature in the IDE (or “Go to Declaration” on a Mac) to see the definition of IDisposable. Go to your project and type `IDisposable` anywhere inside a class. Then right-click on it and select Go To Definition from the menu. It’ll open a new tab with code in it. Expand all of the code and this is what you’ll see:

![Images](assets/pg545.png)

# Avoid filesystem errors with using statements

Throughout the chapter we’ve been stressing that you need to **close your streams**. That’s because some of the most common bugs that programmers run across when they deal with files are caused when streams aren’t closed properly. Luckily, C# gives you a great tool to make sure that never happens to you: IDisposable and the Dispose method. When you **wrap your stream code in a `using` statement**, it automatically closes your streams for you. All you need to do is **declare your stream reference** with a `using` statement, followed by a block of code (inside curly brackets) that uses that reference. When you do that, C# **automatically calls the Dispose method** as soon as it finishes running the block of code.

###### Note

These “using” statements are different from the ones at the top of your code.

![Images](assets/pg546.png)

###### Note

**This `using` statement declares a variable `sw` that references a new StreamWriter and is followed by a block of code. After all of the statements in the block are executed, the using block will automatically call sw.Dispose.**

## Use multiple using statements for multiple objects

You can pile `using` statements on top of each other—you don’t need extra sets of curly brackets or indents:

[PRE8]

###### Note

**Every stream has a Dispose method that closes the stream. When you declare your stream in a `using` statement, it will always get closed! And that’s important, because some streams *don’t write* all of their data *until they’re closed.***

> **When you declare an object in a using block, that object’s Dispose method is called automatically.**

# Use a MemoryStream to stream data to memory

We’ve been using streams to read and write files. What if you want to read data from a file and then, well, do something with it? You can use a **MemoryStream**, which keeps track of all data streamed to it by storing it in memory. For example, you can create a new MemoryStream and pass it as an argument to a StreamWriter constructor, and then any data you write with the StreamWriter will be sent to that MemoryStream. You can retrieve that data using the **MemoryStream.ToArray method**, which returns all of the data that’s been streamed to it in a byte array.

## Use Encoding.UTF8.GetString to convert byte arrays to strings

One of the most common things that you’ll do with byte arrays is convert them to strings. For example, if you have a byte array called bytes, here’s one way to convert it to a string:

![Images](assets/pg547-1.png)

Here’s a small console app that uses composite formatting to write a number to a MemoryStream, and then convert it to a byte array and then to a string. Just one problem... ***it doesn’t work!***

**Create a new console app** and add this code to it. Can you sleuth out the problem and fix it?

***Do this!***

![Images](assets/pg547-2.png)

###### Note

**This app doesn’t work! When you run it, it ’s supposed to write a line of text to the console, but it doesn’t write anything at all. We’ll explain what’s wrong, but before we do first see if you can sleuth it out on your own.**

***Here’s a hint: can you figure out when the streams are closed?***

**Q: Remind me why you put an @ in front of the strings that contained filenames in that “Sharpen your pencil” exercise?**

**A:** Because if we didn’t, the \S in “C:\SYP” would be interpreted as an invalid escape sequence and throw an exception. When you add a string literal to your program, the compiler converts escape sequences like `\n` and `\r` to special characters. Windows filenames have backslash characters in them, but C# strings normally use backslashes to start escape sequences. If you put @ in front of a string, it tells C# not to interpret escape sequences. It also tells C# to include line breaks in your string, so you can hit Enter halfway through the string and it’ll include that as a line break in the output.

**Q: And what exactly are escape sequences?**

**A:** An escape sequence is a way to include special characters in your strings. For example, `\n` is a line feed, `\t` is a tab, and `\r` is a return character, or half of a Windows return (in Windows text files, lines have to end with `\r\n`; for macOS and Linux, lines just end in `\n`). If you need to include a quotation mark inside a string, you can use `\"` and it won’t end the string. If you want to use an actual backslash in your string and not have C# interpret it as the beginning of an escape sequence, just do a double backslash: `\\`.

###### Note

We gave you a chance to sleuth this problem out on your own. Here’s how we fixed it.

![Images](assets/pg552.png)

**There’s an easier way to store your objects in files. It’s called serialization.**

**Serialization** means writing an object’s entire state to a file or string. **Deserialization** means reading the object’s state back from that file or string. So instead of painstakingly writing out each field and value to a file line by line, you can save your object the easy way by serializing it out to a stream. ***Serializing*** an object is like **flattening it out** so you can slip it into a file. On the other end, ***deserializing*** an object is like taking it out of the file and **inflating** it again.

###### Note

OK, just to come clean here: there’s also a method called Enum.Parse that will convert the string “Spades” to the enum value Suits.Spades. It even has a companion, Enum.TryParse, that works just like the int. TryParse method you’ve used throughout this book. But serialization still makes a lot more sense here. You’ll find out more about that shortly...

# What happens to an object when it’s serialized?

It seems like something mysterious has to happen to an object in order to copy it off of the heap and put it into a file, but it’s actually pretty straightforward.

1.  **Object on the heap**

    ![Images](assets/pg553-1.png)

    When you create an instance of an object, it has a **state**. Everything that an object “knows” is what makes one instance of a class different from another instance of the same class.

2.  **Object serialized**

    ![Images](assets/pg553-2.png)

    When C# serializes an object, it **saves the complete state of the object**, so that an identical instance (object) can be brought back to life on the heap later.

    ![Images](assets/pg553-3.png)
3.  **And later on...**

    Later—maybe days later, and in a different program—you can go back to the file and **deserialize** it. That pulls the original class back out of the file and restores it **exactly as it was**, with all of its fields and values intact.

    ![Images](assets/pg553-4.png)

# But what exactly IS an object’s state? What needs to be saved?

We already know that **an object stores its state in its fields and properties**. So when an object is serialized, each of those values needs to be saved to the file.

Serialization starts to get interesting when you have more complicated objects. Chars, ints, doubles, and other value types have bytes that can just be written out to a file as is. What if an object has an instance variable that’s an object *reference*? What about an object that has five instance variables that are object references? What if those object instance variables themselves have instance variables?

Think about it for a minute. What part of an object is potentially unique? Think about what needs to be restored in order to get an object that’s identical to the one that was saved. Somehow everything on the heap has to be written to the file.

###### Note

Brain Barbell is like a “juiced up” Brain Power. Take a few minutes and really think about this one.

# When an object is serialized, all of the objects it refers to get serialized, too...

...and all of the objects *they* refer to, and all of the objects *those other objects* refer to, and so on and so on. Don’t worry—it may sound complicated, but it all happens automatically. C# starts with the object you want to serialize and looks through its fields for other objects. Then it does the same for each of them. Every single object gets written out to the file, along with all the information C# needs to reconstitute it all when the object gets deserialized.

###### Note

**A group of objects connected to each other by references is sometimes referred to as a graph.**

![Images](assets/pg555.png)

# Use JsonSerialization to serialize your objects

You’re not just limited to reading and writing lines of text to your files. You can use **JSON serialization** to let your programs ***copy entire objects*** to strings (which you can write to files!) and read them back in...all in just a few lines of code! Let’s take a look at how this works. Start by **creating a new console app.**

***Do this!***

1.  **Design some classes for your object graph.**

    Add this `HairColor` enum and these Guy, Outfit, and HairStyle classes to your new console app:

    [PRE9]

2.  **Create a graph of objects to serialize.**

    Now create a small graph of objects to serialize: a new List<Guy> pointing to a couple of Guy objects. Add this code to your Main method. It uses a collection initializer and object initializers to build the object graph:

    [PRE10]

    ![Images](assets/pg556.png)
3.  **Use JsonSerializer to serialize the objects to a string.**

    First add a `using` directive to the top of your code file:

    [PRE11]

    Now you can **serialize the entire graph** with a single line of code:

    [PRE12]

    Run your app and look closely at what it prints to the console:

    [PRE13]

    That’s your object graph **serialized to JSON** (which some people pronounce “Jason” and others pronounce “JAY-sahn”). It’s *a human-readable data interchange format*, which means that it’s a way to store complex objects using strings that a person can make sense of. Because it’s human readable, you can see that it has all of the parts of the graph: names and clothes are encoded as strings (“Bob”, “t-shirt”) and enums are encoded as their integer values.

4.  **Use JsonSerializer to deserialize the JSON to a new object graph.**

    Now that we have a string that contains the object graph serialized to JSON, we can **deserialize** it. That just means using it to create new objects. JsonSerializer lets us do that in one line of code, too. Add this to the Main method:

    [PRE14]

    Run your app again. It deserializes the guys from the JSON string and writes them to the console:

    [PRE15]

###### Note

**When you use JsonSerializer to serialize an object graph to JSON, it generates a (somewhat) readable text representation of the data in each object.**

# JSON only includes data, not specific C# types

When you were looking through the JSON data, you saw human-readable versions of the data in your objects: strings like “Bob” and “slacks”, numbers like 8 and 3.5, and even lists and nested objects. What *didn’t* you see when you looked at the JSON data? JSON **does not include the names of types**. Look inside a JSON file and you won’t see class names like Guy, Outfit, HairColor, or HairStyle, or even the names of basic types like int, string, or double. That’s because JSON just contains the data, and JsonSerializer will do its best to deserialize the data into whatever properties it finds.

Let’s put this to the test. Add a new class to your project:

![Images](assets/pg559.png)

And run your code again. Since the JSON just has a list of objects, JsonSerializer.Deserialize will happily stick them into a Stack (or a Queue, or an array, or another collection type). Since Dude has public Name and Hair properties that match the data, it will fill in any data that it can. Here’s what it prints to the output:

[PRE16]

###### Note

One more thing! We showed you basic serialization with JsonSerializer. There are just a couple more things you need to know about it.

# Next up: we’ll take a deep dive into our data

You’ve been writing lots of code using value types like int, bool, and double, and creating objects that store data in fields. Now it’s time to take a lower-level view of things. The rest of this chapter is all about getting to know your data better by understanding the actual bytes that C# and .NET use to represent it.

Here’s what we’ll do.

![Images](assets/pg561-1.png)

> **We’ll explore how C# strings are encoded with Unicod—.NET uses Unicode to store characters and text.**

![Images](assets/pg561-2.png)

> **We’ll write values as binary data, then we’ll read them back in and see what bytes were written.**

![Images](assets/pg561-3.png)

> **We’ll build a hex dumper that lets us take a closer look at the bits and bytes in files.**

###### Note

Accessibility features are often included as an afterthought, but your games—and any other kind of program!—come out better if you think about these things from the very beginning.

# C# strings are encoded with Unicode

You’ve been using strings since you typed `"Hello, world!"` into the IDE at the start of [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin). Because strings are so intuitive, we haven’t really needed to dissect them and figure out what makes them tick. But ask yourself...***what exactly is a string?***

A C# string is a **read-only collection of chars**. So if you actually look at how a string is stored in memory, you’ll see the string “Elephant” stored as the chars ‘E’, ‘l’, ‘e’, ‘p’, ‘h’, ‘a’, ‘n’, and ‘t’. Now ask yourself...***what exactly is a char?***

A char is a character represented with **Unicode**. Unicode is an industry standard for **encoding** characters, or converting them into bytes so they can be stored in memory, transmitted across networks, included in documents, or pretty much anything else you want to do with them—and you’re guaranteed that you’ll always get the correct characters.

This is especially important when you consider just how many characters there are. The Unicode standard supports over 150 **scripts** (sets of characters for specific languages), including not just Latin (which has the 26 English letters and variants like é and ç but scripts for many languages used around the world. The list of supported scripts is constantly growing, as the Unicode Consortium adds new ones every year (here’s the current list: [http://www.unicode.org/standard/supported.html](http://www.unicode.org/standard/supported.html)).

![Images](assets/pg563-1.png)

Unicode supports another really important set of characters: **emoji**. All of the emoji, from the winking smiley face (![Images](assets/pg563-2.png) ) to the ever-popular pile of poo (![Images](assets/pg563-3.png) ), are Unicode characters.

![Images](assets/pg564.png)

**Every Unicode character—including emoji—has a unique number called a code point.**

The number for a Unicode character is called a **code point**. You can download a list of all of the Unicode characters here: [https://www.unicode.org/Public/UNIDATA/UnicodeData.txt](https://www.unicode.org/Public/UNIDATA/UnicodeData.txt).

That’s a large text file with a line for every Unicode character. Download it and search for “ELEPHANT” and you’ll find a line that starts like this: `1F418;ELEPHANT`. The numbers 1F418 represent a **hexadecimal** (or **hex**) value. Hex values are written with the numbers 0 to 9 and letters A to F, and it’s often easier to work with Unicode values (and binary values in general) in hex than in decimal. You can create a hex literal in C# by adding 0x to the beginning, like this: 0x1F418.

1F418 is the Elephant emoji’s ***UTF-8*** **code point**. UTF-8 is the most common way to **encode** a character as Unicode (or represent it as a number). It’s a variable-length encoding, using between 1 and 4 bytes. In this case, it uses 3 bytes: 0x01 (or 1), 0xF4 (or 244), and 0x18 (or 24).

But that’s not what the JSON serializer printed. It printed a longer hex number: D83DDC18\. That’s because **the C# char type uses *UTF-16***, which uses code points that consist of either one or two 2-byte numbers. The UTF-16 code point of the elephant emoji is 0xD83D 0xDC18\. UTF-8 is much more popular than UTF-16, especially on the Web, so when you look up code points you’re much more likely to find UTF-8 than UTF-16.

> **UTF-8 is a variable-length encoding used by most web pages and many systems. It can store characters using one, two, three, or more bytes. UTF-16 is a fixed-length encoding that always uses either one or two 2-byte numbers. .NET stores char values in memory as UTF-16 values.**

# Visual Studio works really well with Unicode

Let’s use Visual Studio to see how the IDE works with Unicode characters. You saw back in [#start_building_with_chash_build_somethin](ch01.html#start_building_with_chash_build_somethin) that you can use emoji in code. Let’s see what else the IDE can handle. Go to the code editor and enter this code:

[PRE17]

If you’re using Windows, open up the Character Map app. If you’re using a Mac, press Ctrl-![Images](assets/pg565a.png)-space to pop up the Character Viewer. Then search for the Hebrew letter shin (![Images](assets/pg565b.png)) and copy it to the clipboard.

![Images](assets/pg565.png)

Place your cursor at the end of the string between the space and the quotation mark, and paste in the shin character that you copied to the clipboard. Hmm, something looks weird:

![Images](assets/pg565-1.png)

Did you notice that the cursor is positioned to the *left* of the pasted letter? Well, let’s continue. Don’t click anywhere in the IDE—keep the cursor where it is, then switch over to Character Map or Character Viewer to search for the Hebrew letter lamed (![Images](assets/pg565-2.png)). Switch back to the IDE—make sure the cursor is still positioned just left of the shin—and paste in the lamed:

![Images](assets/pg565-3.png)

When you pasted the lamed, the IDE added it to the left of the shin. Now search for the Hebrew letters vav (`I`) and final mem (![Images](assets/pg565-4.png)). Paste each of them into the IDE—it will insert them to the left of the cursor:

![Images](assets/pg565-5.png)

The IDE knows that ***Hebrew is read right to left***, so it’s behaving accordingly. Click to select the text near the beginning of the statement, and slowly drag your cursor right to select `Hello` and then ![Images](assets/pg565-6.png). Watch carefully what happens when the selection reaches the Hebrew letters. It skips to the shin (![Images](assets/pg565-7.png)) and then selects from right to left—and that’s exactly what a Hebrew reader would expect it to do.

# .NET uses Unicode to store characters and text

The two C# types for storing text and characters—string and char—keep their data in memory as Unicode. When that data’s written out as bytes to a file, each of those Unicode numbers is written out to the file. Let’s get a sense of exactly how Unicode data is written out to a file. **Create a new console app.** We’ll use the File.WriteAllBytes and File.ReadAllBytes methods to start exploring Unicode.

***Do this!***

1.  **Write a normal string out to a file and read it back.**

    Add the following code to the Main method—it uses File.WriteAllText to write the string “Eureka!” out to a file called *eureka.txt* (so you’ll need `using System.IO;`). Then it creates a new byte array called `eurekaBytes`, reads the file into it, and prints out all of the bytes it read:

    ![Images](assets/pg566.png)

    You’ll see these bytes written to the output: `69 117 114 101 107 97 33`. The last line calls the method Encoding.UTF8.GetString, which converts a byte array with UTF-8-encoded characters to a string. Now **open the file in Notepad** (Windows) **or TextEdit** (Mac). It says “Eureka!”

2.  **Then add code to write the bytes as hex numbers.**

    When you’re encoding data you’lloften use hex, so let’s do that now. Add this code to the end of the Main method that writes the same bytes out, using `{0:x2}` to **format each byte as a hex number**:

    [PRE18]

    ###### Note

    Hex uses the numbers 0 through 9 and letters A through F to represent numbers in base 16, so 6B is equal to 107.

    That tells Write to print parameter 0 (the first one after the string to print) as a two-character hex code. So it writes the same seven bytes in hex instead of decimal: `45 75 72 65 6b-61 21`.

3.  **Modify the first line to write the Hebrew letters “![Images](assets/pg566-2.png)” instead of “Eureka!”**

    You just added the Hebrew text ![Images](assets/pg566-2.png) to another program using either Character Map (Windows) or Character Viewer (Mac). **Comment out the first line of the Main method and replace it with the following code** that writes ![Images](assets/pg566-2.png) to the file instead of “Eureka!” We’ve added an extra Encoding.Unicode parameter so it writes UTF-16 (the Encoding class is in the System.Text namespace, so also add `using System.Text;` to the top):

    ![Images](assets/pg566-1.png)

    Now run the code again, and look closely at the output: `ff fe e9 05 dc 05 d5 05 dd 05`. The first two characters are “FF FE”, which is the Unicode way of saying that we’re going to have a string of 2-byte characters. The rest of the bytes are the Hebrew letters—but they’re reversed, so U+05E9 appears as `**e9 05**`. Now open the file up in Notepad or TextEdit to make sure it looks right.

4.  **Use JsonSerializer to explore UTF-8 and UTF-16 code points.**

    When you serialized the elephant emoji, JsonSerializer generated `\uD83D\uDC18` – which we now know is the 4-byte UTF-16 code point in hex. Now let’s try that with the Hebrew letter shin. Add `using System.Text.Json;` to the top of your app and then add this line:

    ![Images](assets/pg567-1a.png)

    Run your app again. This time it printed a value with two hex bytes, “\u05E9”—that’s the UTF-16 code point for the Hebrew letter shin. It’s also the UTF-8 code point for the same letter.

    But wait a minute—we learned that the UTF-8 code point for the elephant emoji is `0x1F418`, which is ***different*** than the UTF-16 code point (0xD83D 0xDC18). What’s going on?

    It turns out that most of the characters with 2-byte UTF-8 code points have the same code points in UTF-16\. Once you reach the UTF-8 values that require three or more bytes—which includes the familiar emoji that we’ve used in this book—they differ. So while the Hebrew letter shin is 0x05E9 in both UTF-8 and UTF-16, the elephant emoji is 0x1F418 in UTF-8 and 0xD8ED 0xDC18 in UTF-16.

    ![Images](assets/pg567-1.png)
5.  **Use Unicode escape sequences to encode** ![Images](assets/pg567-2.png).

    Add these lines to your Main method to write the elephant emoji to two files using both the UTF-16 and UTF-32 escape sequences:

    [PRE19]

    Run your app again, then open both of those files in Notepad or TextEdit. You should see the correct character written to the file.

    ###### Note

    **You used UTF-16 and UTF-32 escape sequences to create your emoji, but the WriteAllText method writes a UTF-8 file. The Encoding.UTF8.GetString method you used in step 1 converts a byte array with UTF-8-encoded data back to a string.**

# C# can use byte arrays to move data around

Since all your data ends up encoded as **bytes**, it makes sense to think of a file as one **big byte array**...and you already know how to read and write byte arrays.

![Images](assets/pg568.png)

# Use a BinaryWriter to write binary data

###### Note

StreamWriter also encodes your data. It just specializes in text and text encoding—it defaults to UTF-8.

You ***could*** encode all of your strings, chars, ints, and floats into byte arrays before writing them out to files, but that would get pretty tedious. That’s why .NET gives you a very useful class called **BinaryWriter** that **automatically encodes your data** and writes it to a file. All you need to do is create a FileStream and pass it into the BinaryWriter’s constructor (they’re in the System.IO namespace, so you’ll need `using System.IO;`). Then you can call its methods to write out your data. Let’s practice using BinaryWriter to write binary data to a file.

***Do this!***

1.  Start by creating a console app and setting up some data to write to a file:

    ![Images](assets/pg569-1a.png)
2.  To use a BinaryWriter, first you need to open a new stream with File.Create:

    [PRE20]

3.  Now just call its Write method. Each time you do this, it adds new bytes onto the end of the file that contain an encoded version of whatever data you passed it as a parameter:

    ![Images](assets/pg569-1.png)
4.  Now use the same code you used before to read in the file you just wrote:

    [PRE21]

    Write down the output in the blanks below. Can you **figure out** **what bytes correspond** to each of the five `writer.Write(...)` statements? We put a bracket under the groups of bytes that correspond with each statement to help you figure out which bytes in the file correspond with data written by the app.

# Use BinaryReader to read the data back in

The BinaryReader class works just like BinaryWriter. You create a stream, attach the BinaryReader object to it, and then call its methods...but the reader **doesn’t know what data’s in the file**! It has no way of knowing. Your float value of 491.695F was encoded as d8 f5 43 45\. Those same bytes are a perfectly valid `int`—1,140,185,334—so you’ll need to tell the BinaryReader exactly what types to read from the file. Add the following code to your program, and have it read the data you just wrote.

###### Note

Don’t take our word for it. Replace the line that reads the float with a call to ReadInt32\. (You’ll need to change the type of floatRead to int.) Then you can see for yourself what it reads from the file.

1.  Start out by setting up the FileStream and BinaryReader objects:

    [PRE22]

2.  You tell BinaryReader what type of data to read by calling its different methods:

    ![Images](assets/pg570-1.png)
3.  Now write the data that you read from the file to the console:

    [PRE23]

    Here’s the output that gets printed to the console:

    [PRE24]

###### Note

**If you’re writing a string that only has Unicode characters with low numbers (such as Latin letters), it writes one byte per character. If it’s got high-numbered characters (like emoji characters), they’ll be written using two or more bytes each.**

# A hex dump lets you see the bytes in your files

A **hex dump** is a *hexadecimal* view of the contents of a file, and it’s a really common way for programmers to take a deep look at a file’s internal structure.

It turns out that hex is a convenient way to display bytes in a file. A byte takes two characters to display in hex: bytes range from 0 to 255, or 00 to ff in hex. That lets you see a lot of data in a really small space, and in a format that makes it easier to spot patterns. It’s useful to display binary data in rows that are 8, 16, or 32 bytes long because most binary data tends to break down in chunks of 4, 8, 16, or 32...like all the types in C#. (For example, an int takes up 4 bytes.) A hex dump lets you see exactly what those values are made of.

## How to make a hex dump of some plain text

Start with some familiar text using Latin characters:

[PRE25]

First, break up the text into 16-character segments, starting with the first 16: `When you have el`

Next, convert each character in the text to its UTF-8 code point. Since the Latin characters all have ***1-byte*** UTF-8 code points, each will be represented by a two-digit hex number from 00 to 7F. Here’s what each line of our dump will look like:

![Images](assets/pg572.png)

Repeat until you’ve dumped every 16-character segment in the file:

[PRE26]

And that’s our dump. There are many hex dump programs for various operating systems, and each of them has a slightly different output. Each line in our particular hex dump format represents 16 characters in the input that was used to generate it, with the offset at the start of each line and the text for each character at the end. Other hex dump apps might display things differently (for example, rendering escape sequences or showing values in decimal).

> **A hex dump is a hexadecimal view of data in a file or memory, and can be a really useful tool to help you debug binary data.**

# Use StreamReader to build a hex dumper

Let’s build a hex dump app that reads data from a file with StreamReader and writes its dump to the console. We’ll take advantage of the **ReadBlock method** in StreamReader, which reads a block of characters into a char array: you specify the number of characters you want to read, and it’ll either read that many characters or, if there are fewer than that many left in the file, it’ll read the rest of the file. Since we’re displaying 16 characters per line, we’ll read blocks of 16 characters.

**Create a new console app called HexDump**. Before you add code, **run the app** to create the folder with the binary. Use Notepad or TextEdit to **create a text file called** *textdata.txt*, add some text to it, and put it in the same folder as the binary.

Here’s the code from inside the Main method—it reads the *textdata.txt* file and writes a hex dump to the console. Make sure you add `using System.IO;` to the top.

###### Note

**The ReadBlock method reads the next characters from its input into a byte array (sometimes referred to as a buffer). It blocks, which means it keeps executing and doesn’t return until it’s either read all of the characters you asked for or run out of data to read.**

![Images](assets/pg573.png)

# Use Stream.Read to read bytes from a stream

The hex dumper works just fine for text files—but there’s a problem. **Copy the *binarydata.dat* file** you wrote with BinaryWriter into the same folder as your app, then change the app to read it:

[PRE27]

Now run your app. This time it prints something else—but it’s not quite right:

![Images](assets/pg574-1.png)

The text characters (“Hello`!`”) seem OK. But compare the output with the “Sharpen your pencil” solution—the bytes aren’t quite right. It looks like it replaced some bytes (86, e8, 81, f6, d8, and f5) with a different byte, fd. That’s because **StreamReader is built to read text files**, so it only reads **7-bit values**, or byte values up to 127 (7F in hex, or 1111111 in binary—which are 7 bits).

So let’s do this right—by ***reading the bytes directly from the stream***. Modify the `using` block so it uses **File.OpenRead**, which opens the file and **returns a FileStream**. You’ll use the Stream’s Length property to keep reading until you’ve read all of the bytes in the file, and its Read method to read the next 16 bytes into the byte array buffer:

![Images](assets/pg574-2.png)

The rest of the code is the same, except for the line that sets `bufferContents:`

[PRE28]

You used the Encoding class earlier in the chapter to convert a byte array to a string. This byte array contains a single byte per character—that means it’s a valid UTF-8 string. That means you can use Encoding.UTF8.GetString to convert it. Since the Encoding class is in the System.Text namespace, you’ll need to add `using System.Text;` to the top of the file.

Now run your app again. This time it prints the correct bytes instead of changing them to fd:

![Images](assets/pg574-3.png)

There’s just one more thing we can do to clean up the output. Many hex dump programs replace nontext characters with dots in the output. **Add this line to the end of the `for` loop:**

[PRE29]

Now run your app again—this time the question marks are replaced with dots:

[PRE30]

# Modify your hex dumper to use command-line arguments

Most hex dump programs are utilities that you run from the command line. You can dump a file by passing its name to the hex dumper as a **command-line argument**, like this: `C:\> HexDump myfile.txt`

Let’s modify the hex dumper to use command-line arguments. When you create a console app, C# makes the command-line arguments available as the **`args` string array** that gets passed to the Main method:

[PRE31]

We’ll modify the Main method to open a file and read its contents from a stream. The **File.OpenRead method** takes a filename as a parameter, opens it for reading, and returns a stream with the file contents.

Change these lines in your Main method:

![Images](assets/pg575.png)

Now let’s use the command-line argument in the IDE by **changing the debug properties** to pass a command line to the program. **Right-click on the project** in the solution, then:

*   ***On Windows,*** choose Properties, then click Debug, and enter the filename to dump in the Application arguments box (either the full path or the name of a file in the binary folder).

*   ***On macOS,*** choose Options, expand Run >> Configurations, click Default, and enter the filename in the Arguments box.

###### Note

Make sure you right-click on the project, not the solution.

Now when you debug your app, its `args` array will contain the arguments you set in the project settings. ***Make sure you specify a valid filename when you set up the command-line arguments.***

## Run your app from the command line

You can also run the app from the command line, replacing `[filename]` with the name of a file (either the full path or the name of a file in the current directory):

*   ***On Windows,*** Visual Studio builds an executable under the bin\Debug folder (in the same place you put your files to read), so you can run the executable directly from that folder. Open a command window, **`cd`** to the bin\Debug folder, and run `HexDump [filename].`

*   ***On a Mac,*** you’ll need to **build a self-contained application**. Open a Terminal window, go to the project folder, and run this command: `dotnet publish -r osx-x64.`

    The output will include a line like this: `HexDump -> /path-to-binary/osx-x64/publish/`

    Open a Terminal window, `cd` to the full path that it printed, and run `./HexDump [filename].`