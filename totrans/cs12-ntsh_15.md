# Chapter 15\. Streams and I/O

This chapter describes the fundamental types for input and output in .NET, with emphasis on the following topics:

*   The .NET stream architecture and how it provides a consistent programming interface for reading and writing across a variety of I/O types

*   Classes for manipulating files and directories on disk

*   Specialized streams for compression, named pipes, and memory-mapped files

This chapter concentrates on the types in the `System.IO` namespace, the home of lower-level I/O functionality.

# Stream Architecture

The .NET stream architecture centers on three concepts: backing stores, decorators, and adapters, as shown in [Figure 15-1](#stream_architecture-id00092).

A *backing store* is the endpoint that makes input and output useful, such as a file or network connection. Precisely, it is either or both of the following:

*   A source from which bytes can be sequentially read

*   A destination to which bytes can be sequentially written

![Stream architecture](assets/cn10_1501.png)

###### Figure 15-1\. Stream architecture

A backing store is of no use, though, unless exposed to the programmer. A `Stream` is the standard .NET class for this purpose; it exposes a standard set of methods for reading, writing, and positioning. Unlike an array, for which all the backing data exists in memory at once, a stream deals with data serially—either one byte at a time or in blocks of a manageable size. Hence, a stream can use a small, fixed amount of memory regardless of the size of its backing store.

Streams fall into two categories:

Backing store streams

These are hardwired to a particular type of backing store, such as `FileStream` or `NetworkStream`.

Decorator streams

These feed off another stream, transforming the data in some way, such as `DeflateStream` or `CryptoStream`.

Decorator streams have the following architectural benefits:

*   They liberate backing store streams from needing to implement such features as compression and encryption themselves.

*   Streams don’t suffer a change of interface when decorated.

*   You connect decorators at runtime.

*   You can chain decorators together (e.g., a compressor followed by an encryptor).

Both backing store and decorator streams deal exclusively in bytes. Although this is flexible and efficient, applications often work at higher levels such as text or XML. *Adapters* bridge this gap by wrapping a stream in a class with specialized methods typed to a particular format. For example, a text reader exposes a `ReadLine` method; an XML writer exposes a `WriteAttributes` method.

###### Note

An adapter wraps a stream, just like a decorator. Unlike a decorator, however, an adapter is not *itself* a stream; it typically hides the byte-oriented methods completely.

To summarize, backing store streams provide the raw data; decorator streams provide transparent binary transformations such as encryption; adapters offer typed methods for dealing in higher-level types such as strings and XML.

[Figure 15-1](#stream_architecture-id00092) illustrates their associations. To compose a chain, you simply pass one object into another’s constructor.

# Using Streams

The abstract `Stream` class is the base for all streams. It defines methods and properties for three fundamental operations: *reading*, *writing*, and *seeking*, as well as for administrative tasks such as closing, flushing, and configuring timeouts (see [Table 15-1](#stream_class_members)).

Table 15-1\. Stream class members

| Category | Members |
| --- | --- |
| Reading | `public abstract bool CanRead { get; }` |
|  | `public abstract int Read (byte[] buffer, int offset, int count)` |
|  | `public virtual int ReadByte();` |
| Writing | `public abstract bool CanWrite { get; }` |
|  | `public abstract void Write (byte[] buffer, int offset, int count);` |
|  | `public virtual void WriteByte (byte value);` |
| Seeking | `public abstract bool CanSeek { get; }` |
|  | `public abstract long Position { get; set; }` |
|  | `public abstract void SetLength (long value);` |
|  | `public abstract long Length { get; }` |
|  | `public abstract long Seek (long offset, SeekOrigin origin);` |
| Closing/flushing | `public virtual void Close();` |
|  | `public void Dispose();` |
|  | `public abstract void Flush();` |
| Timeouts | `public virtual bool CanTimeout { get; }` |
|  | `public virtual int ReadTimeout { get; set; }` |
|  | `public virtual int WriteTimeout { get; set; }` |
| Other | `public static readonly Stream Null; // "Null" stream` |
|  | `public static Stream Synchronized (Stream stream);` |

There are also asynchronous versions of the `Read` and `Write` methods, both of which return `Task`s and optionally accept a cancellation token, and overloads that work with `Span<T>` and `Memory<T>` types that we describe in [Chapter 23](ch23.html#spanless_thantgreater_than_and-id00089).

In the following example, we use a file stream to read, write, and seek:

[PRE0]

Reading or writing asynchronously is simply a question of calling `ReadAsync`/`WriteAsync` instead of `Read`/`Write`, and `await`ing the expression (we must also add the `async` keyword to the calling method, as we described in [Chapter 14](ch14.html#concurrency_and_asynchron)):

[PRE1]

The asynchronous methods make it easy to write responsive and scalable applications that work with potentially slow streams (particularly network streams), without tying up a thread.

###### Note

For the sake of brevity, we’ll continue to use synchronous methods for most of the examples in this chapter; however, we recommend the asynchronous `Read`/`Write` operations as preferable in most scenarios involving network I/O.

## Reading and Writing

A stream can support reading, writing, or both. If `CanWrite` returns `false`, the stream is read-only; if `CanRead` returns `false`, the stream is write-only.

`Read` receives a block of data from the stream into an array. It returns the number of bytes received, which is always either less than or equal to the `count` argument. If it’s less than `count`, it means that either the end of the stream has been reached or the stream is giving you the data in smaller chunks (as is often the case with network streams). In either case, the balance of bytes in the array will remain unwritten, their previous values preserved.

###### Warning

With `Read`, you can be certain you’ve reached the end of the stream only when the method returns `0`. So, if you have a 1,000-byte stream, the following code might fail to read it all into memory:

[PRE2]

The `Read` method could read anywhere from 1 to 1,000 bytes, leaving the balance of the stream unread.

Here’s the correct way to read a 1,000-byte stream via the `Read` method:

[PRE3]

To make this easier, from .NET 7, the `Stream` class includes helper methods called `ReadExactly` and `ReadAtLeast` (and async versions of each). The following reads exactly 1,000 bytes from the stream (throwing an exception if the stream ends before then):

[PRE4]

The last line is equivalent to:

[PRE5]

###### Note

The `BinaryReader` type provides another solution:

[PRE6]

If the stream is less than 1,000 bytes long, the byte array returned reflects the actual stream size. If the stream is seekable, you can read its entire contents by replacing `1000` with `(int)s.Length`.

We describe the `BinaryReader` type further in [“Stream Adapters”](#stream_adapters).

The `ReadByte` method is simpler: it reads just a single byte, returning −1 to indicate the end of the stream. `ReadByte` actually returns an `int` rather than a `byte` because the latter cannot return −1.

The `Write` and `WriteByte` methods send data to the stream. If they are unable to send the specified bytes, an exception is thrown.

###### Warning

In the `Read` and `Write` methods, the `offset` argument refers to the index in the `buffer` array at which reading or writing begins, not the position within the stream.

## Seeking

A stream is seekable if `CanSeek` returns `true`. With a seekable stream (such as a file stream), you can query or modify its `Length` (by calling `SetLength`) and at any time change the `Position` at which you’re reading or writing. The `Position` property is relative to the beginning of the stream; the `Seek` method, however, allows you to move relative to the current position or the end of the stream.

###### Note

Changing the `Position` on a `FileStream` typically takes a few microseconds. If you’re doing this millions of times in a loop, the `MemoryMappedFile` class might be a better choice than a `FileStream` (see [“Memory-Mapped Files”](#memory_mapped_files)).

With a nonseekable stream (such as an encryption stream), the only way to determine its length is to read it completely through. Furthermore, if you need to reread a previous section, you must close the stream and start afresh with a new one.

## Closing and Flushing

Streams must be disposed after use to release underlying resources such as file and socket handles. A simple way to guarantee this is by instantiating streams within `using` blocks. In general, streams follow standard disposal semantics:

*   `Dispose` and `Close` are identical in function.

*   Disposing or closing a stream repeatedly causes no error.

Closing a decorator stream closes both the decorator and its backing store stream. With a chain of decorators, closing the outermost decorator (at the head of the chain) closes the whole lot.

Some streams internally buffer data to and from the backing store to lessen round-tripping and so improve performance (file streams are a good example of this). This means that data you write to a stream might not hit the backing store immediately; it can be delayed as the buffer fills up. The `Flush` method forces any internally buffered data to be written immediately. `Flush` is called automatically when a stream is closed, so you never need to do the following:

[PRE7]

## Timeouts

A stream supports read and write timeouts if `CanTimeout` returns `true`. Network streams support timeouts; file and memory streams do not. For streams that support timeouts, the `ReadTimeout` and `WriteTimeout` properties determine the desired timeout in milliseconds, where `0` means no timeout. The `Read` and `Write` methods indicate that a timeout has occurred by throwing an exception.

The asynchronous `ReadAsync`/`WriteAsync` methods do not support timeouts; instead you can pass a cancellation token into these methods.

## Thread Safety

As a rule, streams are not thread-safe, meaning that two threads cannot concurrently read or write to the same stream without possible error. The `Stream` class offers a simple workaround via the static `Synchronized` method. This method accepts a stream of any type and returns a thread-safe wrapper. The wrapper works by obtaining an exclusive lock around each read, write, or seek, ensuring that only one thread can perform such an operation at a time. In practice, this allows multiple threads to simultaneously append data to the same stream—other kinds of activities (such as concurrent reading) require additional locking to ensure that each thread accesses the desired portion of the stream. We discuss thread safety fully in [Chapter 21](ch21.html#advanced_threadin).

###### Note

From .NET 6, you can use the `RandomAccess` class for performant thread-safe file I/O operations. `RandomAccess` also lets you pass in multiple buffers to improve performance.

## Backing Store Streams

[Figure 15-2](#backing_store_streams-id00053) shows the key backing store streams provided by .NET. A “null stream” is also available via the `Stream`’s static `Null` field. Null streams can be useful when writing unit tests.

![Backing store streams](assets/cn10_1502.png)

###### Figure 15-2\. Backing store streams

In the following sections, we describe `FileStream` and `MemoryStream`; in the final section in this chapter, we describe `IsolatedStorageStream`. In [Chapter 16](ch16.html#networking-id00041), we cover `NetworkStream`.

## FileStream

Earlier in this section, we demonstrated the basic use of a `FileStream` to read and write bytes of data. Let’s now examine the special features of this class.

###### Note

If you’re still using Universal Windows Platform [UWP], you can also do file I/O with the types in `Windows.Storage`. We describe this in the online supplement at [*http://www.albahari.com/nutshell*](http://www.albahari.com/nutshell).

### Constructing a FileStream

The simplest way to instantiate a `FileStream` is to use one of the following static façade methods on the `File` class:

[PRE8]

`OpenWrite` and `Create` differ in behavior if the file already exists. `Create` truncates any existing content; `OpenWrite` leaves existing content intact with the stream positioned at zero. If you write fewer bytes than were previously in the file, `OpenWrite` leaves you with a mixture of old and new content.

You can also directly instantiate a `FileStream`. Its constructors provide access to every feature, allowing you to specify a filename or low-level file handle, file creation and access modes, and options for sharing, buffering, and security. The following opens an existing file for read/write access without overwriting it (the `using` keyword ensures it is disposed when `fs` exits scope):

[PRE9]

We look closer at `FileMode` shortly.

### Specifying a filename

A filename can be either absolute (e.g., *c:\temp\test.txt—or in Unix, /tmp/test.txt*) or relative to the current directory (e.g., *test.txt* or *temp\test.txt*). You can access or change the current directory via the static `Environment.CurrentDirectory` property.

###### Warning

When a program starts, the current directory might or might not coincide with that of the program’s executable. For this reason, you should never rely on the current directory for locating additional runtime files packaged along with your executable.

`AppDomain.CurrentDomain.BaseDirectory` returns the *application base directory*, which in normal cases is the folder containing the program’s executable. To specify a filename relative to this directory, you can call `Path.Combine`:

[PRE10]

You can read and write across a Windows network via a Universal Naming Convention (UNC) path, such as *\\JoesPC\PicShare\pic.jpg* or *\\10.1.1.2\PicShare\pic.jpg*. (To access a Windows file share from macOS or Unix, mount it to your filesystem following instructions specific to your OS, and then open it using an ordinary path from C#).

### Specifying a FileMode

All of `FileStream`’s constructors that accept a filename also require a `FileMode` enum argument. [Figure 15-3](#choosing_a_filemode) shows how to choose a `FileMode`, and the choices yield results akin to calling a static method on the `File` class.

![Choosing a FileMode](assets/cn10_1503.png)

###### Figure 15-3\. Choosing a FileMode

###### Warning

`File.Create` and `FileMode.Create` will throw an exception if used on hidden files. To overwrite a hidden file, you must delete and re-create it:

[PRE11]

Constructing a `FileStream` with just a filename and `FileMode` gives you (with just one exception) a readable writable stream. You can request a downgrade if you also supply a `FileAccess` argument:

[PRE12]

The following returns a read-only stream, equivalent to calling `File.OpenRead`:

[PRE13]

`FileMode.Append` is the odd one out: with this mode, you get a *write-only* stream. To append with read-write support, you must instead use `FileMode.Open` or `FileMode.OpenOrCreate` and then seek the end of the stream:

[PRE14]

### Advanced FileStream features

Here are other optional arguments you can include when constructing a `FileStream`:

*   A `FileShare` enum describing how much access to grant other processes wanting to dip into the same file before you’ve finished (`None`, `Read` [default], `ReadWrite`, or `Write`).

*   The size, in bytes, of the internal buffer (default is currently 4 KB).

*   A flag indicating whether to defer to the operating system for asynchronous I/O.

*   A `FileOptions` flags enum for requesting operating system encryption (`Encrypted`), automatic deletion upon closure for temporary files (`DeleteOnClose`), and optimization hints (`RandomAccess` and `SequentialScan`). There is also a `WriteThrough` flag that requests that the OS disable write-behind caching; this is for transactional files or logs. Flags not supported by the underlying OS are silently ignored.

Opening a file with `FileShare.ReadWrite` allows other processes or users to simultaneously read and write to the same file. To avoid chaos, you can all agree to lock specified portions of the file before reading or writing, using these methods:

[PRE15]

`Lock` throws an exception if part or all of the requested file section has already been locked.

## MemoryStream

`MemoryStream` uses an array as a backing store. This partly defeats the purpose of having a stream because the entire backing store must reside in memory at once. `MemoryStream` is still useful when you need random access to a nonseekable stream. If you know the source stream will be of a manageable size, you can copy it into a `MemoryStream` as follows:

[PRE16]

You can convert a `MemoryStream` to a byte array by calling `ToArray`. The `GetBuffer` method does the same job more efficiently by returning a direct reference to the underlying storage array; unfortunately, this array is usually longer than the stream’s real length.

###### Note

Closing and flushing a `MemoryStream` is optional. If you close a `MemoryStream`, you can no longer read or write to it, but you are still permitted to call `ToArray` to obtain the underlying data. `Flush` does absolutely nothing on a memory stream.

You can find further `MemoryStream` examples in [“Compression Streams”](#compression_streams) and in [“Overview”](ch20.html#overview-id00067).

## PipeStream

`PipeStream` provides a simple means by which one process can communicate with another through the operating system’s *pipes* protocol. There are two kinds of pipe:

Anonymous pipe (faster)

Allows one-way communication between a parent and child process on the same computer

Named pipe (more flexible)

Allows two-way communication between arbitrary processes on the same computer or different computers across a network

A pipe is good for interprocess communication (IPC) on a single computer: it doesn’t rely on a network transport, which means no network protocol overhead, and it has no issues with firewalls.

###### Note

Pipes are stream-based, so one process waits to receive a series of bytes while another process sends them. An alternative is for processes to communicate via a block of shared memory; we describe how to do this in [“Memory-Mapped Files”](#memory_mapped_files).

`PipeStream` is an abstract class with four concrete subtypes. Two are used for anonymous pipes and the other two for named pipes:

Anonymous pipes

`AnonymousPipeServerStream` and `AnonymousPipeClientStream`

Named pipes

`NamedPipeServerStream` and `NamedPipeClientStream`

Named pipes are simpler to use, so we describe them first.

### Named pipes

With named pipes, the parties communicate through a pipe of the same name. The protocol defines two distinct roles: the client and server. Communication happens between the client and server as follows:

*   The server instantiates a `NamedPipeServerStream` and then calls `WaitForConnection`.

*   The client instantiates a `NamedPipeClientStream` and then calls `Connect` (with an optional timeout).

The two parties then read and write the streams to communicate.

The following example demonstrates a server that sends a single byte (100) and then waits to receive a single byte:

[PRE17]

Here’s the corresponding client code:

[PRE18]

Named pipe streams are bidirectional by default, so either party can read or write their stream. This means that the client and server must agree on some protocol to coordinate their actions, so both parties don’t end up sending or receiving at once.

There also needs to be agreement on the length of each transmission. Our example was trivial in this regard, because we bounced just a single byte in each direction. To help with messages longer than one byte, pipes provide a *message* transmission mode (Windows only). If this is enabled, a party calling `Read` can know when a message is complete by checking the `IsMessageComplete` property. To demonstrate, we begin by writing a helper method that reads a whole message from a message-enabled `PipeStream`—in other words, reads until `IsMessageComplete` is true:

[PRE19]

(To make this asynchronous, replace “`s.Read`” with “`await s.ReadAsync`”.)

###### Warning

You cannot determine whether a `PipeStream` has finished reading a message simply by waiting for `Read` to return 0\. This is because, unlike most other stream types, pipe streams and network streams have no definite end. Instead, they temporarily “dry up” between message transmissions.

Now we can activate message transmission mode. On the server, this is done by specifying `PipeTransmissionMode.Message` when constructing the stream:

[PRE20]

On the client, we activate message transmission mode by setting `ReadMode` after calling `Connect`:

[PRE21]

###### Note

Message mode is supported only on Windows. Other platforms throw `PlatformNotSupportedException`.

### Anonymous pipes

An anonymous pipe provides a one-way communication stream between a parent and child process. Instead of using a system-wide name, anonymous pipes tune in through a private handle.

As with named pipes, there are distinct client and server roles. The system of communication is a little different, however, and proceeds as follows:

1.  The server instantiates an `AnonymousPipeServerStream`, committing to a `PipeDirection` of `In` or `Out`.

2.  The server calls `GetClientHandleAsString` to obtain an identifier for the pipe, which it then passes to the client (typically as an argument when starting the child process).

3.  The child process instantiates an `AnonymousPipeClientStream`, specifying the opposite `PipeDirection`.

4.  The server releases the local handle that was generated in Step 2, by calling `DisposeLocalCopyOfClientHandle`.

5.  The parent and child processes communicate by reading/writing the stream.

Because anonymous pipes are unidirectional, a server must create two pipes for bidirectional communication. The following Console program creates two pipes (input and output) and then starts up a child process. It then sends a single byte to the child process, and receives a single byte in return:

[PRE22]

As with named pipes, the client and server must coordinate their sending and receiving and agree on the length of each transmission. Anonymous pipes don’t, unfortunately, support message mode, so you must implement your own protocol for message length agreement. One solution is to send, in the first four bytes of each transmission, an integer value defining the length of the message to follow. The `BitConverter` class provides methods for converting between an integer and an array of four bytes.

## BufferedStream

`BufferedStream` decorates, or wraps, another stream with buffering capability, and it is one of a number of decorator stream types in .NET, all of which are illustrated in [Figure 15-4](#decorator_streams).

![Decorator streams](assets/cn10_1504.png)

###### Figure 15-4\. Decorator streams

Buffering improves performance by reducing round trips to the backing store. Here’s how we wrap a `FileStream` in a 20 KB `BufferedStream`:

[PRE23]

In this example, the underlying stream advances 20,000 bytes after reading just one byte, thanks to the read-ahead buffering. We could call `ReadByte` another 19,999 times before the `FileStream` would be hit again.

Coupling a `BufferedStream` to a `FileStream`, as in this example, is of limited value because `FileStream` already has built-in buffering. Its only use might be in enlarging the buffer on an already constructed `FileStream`.

Closing a `BufferedStream` automatically closes the underlying backing store stream.

# Stream Adapters

A `Stream` deals only in bytes; to read or write data types such as strings, integers, or XML elements, you must plug in an adapter. Here’s what .NET provides:

Text adapters (for string and character data)

`TextReader`, `TextWriter`

`StreamReader`, `StreamWriter`

`StringReader`, `StringWriter`

Binary adapters (for primitive types such as `int`, `bool`, `string`, and `float`)

`BinaryReader`, `BinaryWriter`

XML adapters (covered in [Chapter 11](ch11.html#other_xml_and_json_technologies))

`XmlReader`, `XmlWriter`

[Figure 15-5](#readers_and_writers) illustrates the relationships between these types.

![Readers and writers](assets/cn10_1505.png)

###### Figure 15-5\. Readers and writers

## Text Adapters

`TextReader` and `TextWriter` are the abstract base classes for adapters that deal exclusively with characters and strings. Each has two general-purpose implementations in .NET:

`StreamReader`/`StreamWriter`

Uses a `Stream` for its raw data store, translating the stream’s bytes into characters or strings

`StringReader`/`StringWriter`

Implements `TextReader`/`TextWriter` using in-memory strings

[Table 15-2](#textreader_members) lists `TextReader`’s members by category. `Peek` returns the next character in the stream without advancing the position. Both `Peek` and the zero-argument version of `Read` return −1 if at the end of the stream; otherwise, they return an integer that can be cast directly to a `char`. The overload of `Read` that accepts a `char[]` buffer is identical in functionality to the `ReadBlock` method. `ReadLine` reads until reaching either a CR (character 13) or LF (character 10), or a CR+LF pair in sequence. It then returns a string, discarding the CR/LF characters.

Table 15-2\. TextReader members

| Category | Members |
| --- | --- |
| Reading one char | `public virtual int Peek(); // Cast the result to a char` |
|  | `public virtual int Read(); // Cast the result to a char` |
| Reading many chars | `public virtual int Read (char[] buffer, int index, int count);` |
|  | `public virtual int ReadBlock (char[] buffer, int index, int count);` |
|  | `public virtual string ReadLine();` |
|  | `public virtual string ReadToEnd();` |
| Closing | `public virtual void Close();` |
|  | `public void Dispose(); // Same as Close` |
| Other | `public static readonly TextReader Null;` |
|  | `public static TextReader Synchronized (TextReader reader);` |

###### Note

`Environment.NewLine` returns the new-line sequence for the current OS.

On Windows, this is `"\r\n"` (think “ReturN”) and is loosely modeled on a mechanical typewriter: a CR (character 13) followed by an LF (character 10). Reverse the order and you’ll get either two new lines or none!

On Unix and macOS, it’s simply `"\n"`.

`TextWriter` has analogous methods for writing, as shown in [Table 15-3](#textwriter_members). The `Write` and `WriteLine` methods are additionally overloaded to accept every primitive type, plus the `object` type. These methods simply call the `ToString` method on whatever is passed in (optionally through an `IFormatProvider` specified either when calling the method or when constructing the `TextWriter`).

Table 15-3\. TextWriter members

| Category | Members |
| --- | --- |
| Writing one char | `public virtual void Write (char value);` |
| Writing many chars | `public virtual void Write (string value);` |
|  | `public virtual void Write (char[] buffer, int index, int count);` |
|  | `public virtual void Write (string format, params object[] arg);` |
|  | `public virtual void WriteLine (string value);` |
| Closing and flushing | `public virtual void Close();` |
|  | `public void Dispose(); // Same as Close` |
|  | `public virtual void Flush();` |
| Formatting and encoding | `public virtual IFormatProvider FormatProvider { get; }` |
|  | `public virtual string NewLine { get; set; }` |
|  | `public abstract Encoding Encoding { get; }` |
| Other | `public static readonly TextWriter Null;` |
|  | `public static TextWriter Synchronized (TextWriter writer);` |

`WriteLine` simply appends the given text with `Environment.NewLine`. You can change this via the `NewLine` property (this can be useful for interoperability with Unix file formats).

###### Note

As with `Stream`, `TextReader` and `TextWriter` offer task-based asynchronous versions of their read/write methods.

### StreamReader and StreamWriter

In the following example, a `StreamWriter` writes two lines of text to a file, and then a `StreamReader` reads the file back:

[PRE24]

Because text adapters are so often coupled with files, the `File` class provides the static methods `CreateText`, `AppendText`, and `OpenText` to shortcut the process:

[PRE25]

This also illustrates how to test for the end of a file (viz. `reader.Peek()`). Another option is to read until `reader.ReadLine` returns null.

You can also read and write other types such as integers, but because `TextWriter` invokes `ToString` on your type, you must parse a string when reading it back:

[PRE26]

### Character encodings

`TextReader` and `TextWriter` are by themselves just abstract classes with no connection to a stream or backing store. The `StreamReader` and `StreamWriter` types, however, are connected to an underlying byte-oriented stream, so they must convert between characters and bytes. They do so through an `Encoding` class from the `System.Text` namespace, which you choose when constructing the `StreamReader` or `StreamWriter`. If you choose none, the default UTF-8 encoding is used.

###### Warning

If you explicitly specify an encoding, `StreamWriter` will, by default, write a prefix to the start of the stream to identity the encoding. This is usually undesirable, and you can prevent it by constructing the encoding as follows:

[PRE27]

The second argument tells the `StreamWriter` (or `StreamReader`) to throw an exception if it encounters bytes that do not have a valid string translation for their encoding, which matches its default behavior if you do not specify an encoding.

The simplest of the encodings is ASCII because each character is represented by one byte. The ASCII encoding maps the first 127 characters of the Unicode set into its single byte, covering what you see on a US-style keyboard. Most other characters, including specialized symbols and non-English characters, cannot be represented and are converted to the □ character. The default UTF-8 encoding can map all allocated Unicode characters, but it is more complex. The first 127 characters encode to a single byte, for ASCII compatibility; the remaining characters encode to a variable number of bytes (most commonly two or three). Consider the following:

[PRE28]

The word “but” is followed not by a stock-standard hyphen but by the longer em dash (—) character, U+2014\. This is the one that won’t get you into trouble with your book editor! Let’s examine the output:

[PRE29]

Because the em dash is outside the first 127 characters of the Unicode set, it requires more than a single byte to encode in UTF-8 (in this case, three). UTF-8 is efficient with the Western alphabet as most popular characters consume just one byte. It also downgrades easily to ASCII simply by ignoring all bytes above 127\. Its disadvantage is that seeking within a stream is troublesome because a character’s position does not correspond to its byte position in the stream. An alternative is UTF-16 (labeled just “Unicode” in the `Encoding` class). Here’s how we write the same string with UTF-16:

[PRE30]

And here’s the output:

[PRE31]

Technically, UTF-16 uses either two or four bytes per character (there are close to a million Unicode characters allocated or reserved, so two bytes is not always enough). However, because the C# `char` type is itself only 16 bits wide, a UTF-16 encoding will always use exactly two bytes per .NET `char`. This makes it easy to jump to a particular character index within a stream.

UTF-16 uses a two-byte prefix to identify whether the byte pairs are written in a “little-endian” or “big-endian” order (the least significant byte first or the most significant byte first). The default little-endian order is standard for Windows-based systems.

### StringReader and StringWriter

The `StringReader` and `StringWriter` adapters don’t wrap a stream at all; instead, they use a string or `StringBuilder` as the underlying data source. This means no byte translation is required—in fact, the classes do nothing you couldn’t easily achieve with a string or `StringBuilder` coupled with an index variable. Their advantage, though, is that they share a base class with `StreamReader`/`StreamWriter`. For instance, suppose that we have a string containing XML and want to parse it with an `XmlReader`. The `XmlReader.Create` method accepts one of the following:

*   A `URI`

*   A `Stream`

*   A `TextReader`

So, how do we XML-parse our string? Because `StringReader` is a subclass of `TextReader`, we’re in luck. We can instantiate and pass in a `StringReader` as follows:

[PRE32]

## Binary Adapters

`BinaryReader` and `BinaryWriter` read and write native data types: `bool`, `byte`, `char`, `decimal`, `float`, `double`, `short`, `int`, `long`, `sbyte`, `ushort`, `uint`, and `ulong`, as well as `string`s and arrays of the primitive data types.

Unlike `StreamReader` and `StreamWriter`, binary adapters store primitive data types efficiently because they are represented in memory. So, an `int` uses four bytes; a `double` uses eight bytes. Strings are written through a text encoding (as with `StreamReader` and `StreamWriter`) but are length-prefixed in order to make it possible to read back a series of strings without needing special delimiters.

Imagine that we have a simple type, defined as follows:

[PRE33]

We can add the following methods to `Person` to save/load its data to/from a stream using binary adapters:

[PRE34]

`BinaryReader` can also read into byte arrays. The following reads the entire contents of a seekable stream:

[PRE35]

This is more convenient than reading directly from a stream because it doesn’t require a loop to ensure that all data has been read.

## Closing and Disposing Stream Adapters

You have four choices in tearing down stream adapters:

1.  Close the adapter only

2.  Close the adapter and then close the stream

3.  (For writers) Flush the adapter and then close the stream

4.  (For readers) Close just the stream

###### Note

`Close` and `Dispose` are synonymous with adapters, just as they are with streams.

Options 1 and 2 are semantically identical because closing an adapter automatically closes the underlying stream. Whenever you nest `using` statements, you’re implicitly taking option 2:

[PRE36]

Because the nest disposes from the inside out, the adapter is first closed, and then the stream. Furthermore, if an exception is thrown within the adapter’s constructor, the stream still closes. It’s hard to go wrong with nested `using` statements!

###### Warning

Never close a stream before closing or flushing its writer—you’ll amputate any data that’s buffered in the adapter.

Options 3 and 4 work because adapters are in the unusual category of *optionally* disposable objects. An example of when you might choose not to dispose an adapter is when you’ve finished with the adapter but you want to leave the underlying stream open for subsequent use:

[PRE37]

Here, we write to a file, reposition the stream, and then read the first byte before closing the stream. If we disposed the `StreamWriter`, it would also close the underlying `FileStream`, causing the subsequent read to fail. The proviso is that we call `Flush` to ensure that the `StreamWriter`’s buffer is written to the underlying stream.

###### Note

Stream adapters—with their optional disposal semantics—do not implement the extended disposal pattern where the finalizer calls `Dispose`. This allows an abandoned adapter to evade automatic disposal when the garbage collector catches up with it.

There’s also a constructor on `StreamReader`/`StreamWriter` that instructs it to keep the stream open after disposal. Consequently, we can rewrite the preceding example as follows:

[PRE38]

# Compression Streams

Two general-purpose compression streams are provided in the `System.IO.Compression` namespace: `DeflateStream` and `GZipStream`. Both use a popular compression algorithm similar to that of the ZIP format. They differ in that `GZipStream` writes an additional protocol at the start and end—including a CRC to detect errors. `GZipStream` also conforms to a standard recognized by other software.

.NET also includes `BrotliStream`, which implements the *Brotli* compression algorithm. `BrotliStream` is more than 10 times slower than `DeflateStream` and `GZipStream` but achieves a better compression ratio. (The performance hit applies only to compression—decompression performs very well.)

All three streams allow reading and writing, with the following provisos:

*   You always *write* to the stream when compressing.

*   You always *read* from the stream when decompressing.

`DeflateStream`, `GZipStream`, and `BrotliStream` are decorators; they compress or decompress data from another stream that you supply in construction. In the following example, we compress and decompress a series of bytes using a `FileStream` as the backing store:

[PRE39]

With `DeflateStream`, the compressed file is 102 bytes: slightly larger than the original (`BrotliStream` would compress it to 73 bytes). Compression works poorly with “dense,” nonrepetitive binary data (and worst of all with encrypted data, which lacks regularity by design). It works well with most text files; in the next example, we compress and decompress a text stream composed of 1,000 words chosen randomly from a small sentence with the *Brotli* algorithm. This also demonstrates chaining a backing store stream, a decorator stream, an adapter (as depicted at the start of the chapter in [Figure 15-1](#stream_architecture-id00092)), and the use of asynchronous methods:

[PRE40]

In this case, `BrotliStream` compresses efficiently to 808 bytes—less than one byte per word. (For comparison, `DeflateStream` compresses the same data to 885 bytes.)

## Compressing in Memory

Sometimes, you need to compress entirely in memory. Here’s how to use a `MemoryStream` for this purpose:

[PRE41]

The `using` statement around the `DeflateStream` closes it in a textbook fashion, flushing any unwritten buffers in the process. This also closes the `MemoryStream` it wraps—meaning we must then call `ToArray` to extract its data.

Here’s an alternative that avoids closing the `MemoryStream` and uses the asynchronous read and write methods:

[PRE42]

The additional flag sent to `DeflateStream`’s constructor instructs it to not follow the usual protocol of taking the underlying stream with it in disposal. In other words, the `MemoryStream` is left open, allowing us to position it back to zero and reread it.

## Unix gzip File Compression

`GZipStream`’s compression algorithm is popular on Unix systems as a file compression format. Each source file is compressed into a separate target file with a *.gz* extension.

The following methods do the work of the Unix command-line gzip and gunzip utilities:

[PRE43]

The following compresses a file:

[PRE44]

And the following decompresses it:

[PRE45]

# Working with ZIP Files

The `ZipArchive` and `ZipFile` classes in `System.IO.Compression` support the ZIP compression format. The advantage of the ZIP format over `DeflateStream` and `GZipStream` is that it also acts as a container for multiple files and is compatible with ZIP files created with Windows Explorer.

`ZipArchive` works with streams, whereas `ZipFile` addresses the more common scenario of working with files. (`ZipFile` is a static helper class for `ZipArchive`.)

`ZipFile`’s `CreateFromDirectory` method adds all the files in a specified directory into a ZIP file:

[PRE46]

`ExtractToDirectory` does the opposite and extracts a ZIP file to a directory:

[PRE47]

(From .NET 8, you can also specify a `Stream` instead of a zip file path.)

When compressing, you can specify whether to optimize for file size or speed as well as whether to include the name of the source directory in the archive. Enabling the latter option in our example would create a subdirectory in the archive called *MyFolder* into which the compressed files would go.

`ZipFile` has an `Open` method for reading/writing individual entries. This returns a `ZipArchive` object (which you can also obtain by instantiating `ZipArchive` with a `Stream` object). When calling `Open`, you must specify a filename and indicate whether you want to `Read`, `Create`, or `Update` the archive. You can then enumerate existing entries via the `Entries` property or find a particular file by calling `GetEntry`:

[PRE48]

`ZipArchiveEntry` also has a `Delete` method, an `ExtractToFile` method (this is actually an extension method in the `ZipFileExtensions` class), and an `Open` method that returns a readable/writable `Stream`. You can create new entries by calling `CreateEntry` (or the `CreateEntryFromFile` extension method) on the `ZipArchive`. The following creates the archive *d:\zz.zip*, to which it adds *foo.dll*, under a directory structure within the archive called *bin\X86*:

[PRE49]

You could do the same thing entirely in memory by constructing `ZipArchive` with a `MemoryStream`.

# Working with Tar Files

The types in the `System.Formats.Tar` namespace (from .NET 7) support the *.tar* archive format, popular on Unix systems for bundling multiple files. To create a *.tar* file (a *tarball*), call `TarFile.CreateFromDirectory`:

[PRE50]

(The third argument indicates whether to include the base directory name in the archive entries.)

To extract a tarball, call `TarFile.ExtractToDirectory`:

[PRE51]

(The third argument indicates whether to overwrite existing files.)

Both of these methods let you specify a `Stream` instead of a *.tar* filepath. In the following example, we write the tarball to a memory stream, and then use `GZipStream` to compress that stream to a *.tar.gz* file:

[PRE52]

(Compressing a *.tar* into a *.tar.gz* is useful because the *.tar* format does not itself incorporate compression, unlike the *.zip* format.) We can extract the *.tar.gz* file as follows:

[PRE53]

You can also access the API at a more granular level with the `TarReader` and `TarWriter` classes. The following illustrates the use of `TarReader`:

[PRE54]

# File and Directory Operations

The `System.IO` namespace provides a set of types for performing “utility” file and directory operations, such as copying and moving, creating directories, and setting file attributes and permissions. For most features, you can choose between either of two classes, one offering static methods and the other instance methods:

Static classes

`File` and `Directory`

Instance-method classes (constructed with a file or directory name)

`FileInfo` and `DirectoryInfo`

Additionally, there’s a static class called `Path`. This does nothing to files or directories; instead, it provides string manipulation methods for filenames and directory paths. `Path` also assists with temporary files.

## The File Class

`File` is a static class whose methods all accept a filename. The filename can be either relative to the current directory or fully qualified with a directory. Here are its methods (all `public` and `static`):

[PRE55]

`Move` throws an exception if the destination file already exists; `Replace` does not. Both methods allow the file to be renamed as well as moved to another directory.

`Delete` throws an `UnauthorizedAccessException` if the file is marked read-only; you can tell this in advance by calling `GetAttributes`. It also throws that exception if the OS denies delete permission for that file to your process. Here are all the members of the `FileAttribute` enum that `GetAttributes` returns:

[PRE56]

Members in this enum are combinable. Here’s how to toggle a single file attribute without upsetting the rest:

[PRE57]

###### Note

`FileInfo` offers an easier way to change a file’s read-only flag:

[PRE58]

### Compression and encryption attributes

###### Note

This feature is Windows-only and requires the NuGet package `System.Management`.

The `Compressed` and `Encrypted` file attributes correspond to the compression and encryption checkboxes on a file or directory’s Properties dialog box in Windows Explorer. This type of compression and encryption is *transparent* in that the OS does all the work behind the scenes, allowing you to read and write plain data.

You cannot use `SetAttributes` to change a file’s `Compressed` or `Encrypted` attributes—it fails silently if you try! The workaround is simple in the latter case: you instead call the `Encrypt()` and `Decrypt()` methods in the `File` class. With compression, it’s more complicated; one solution is to use the Windows Management Instrumentation (WMI) API in `System.Management`. The following method compresses a directory, returning `0` if successful (or a WMI error code if not):

[PRE59]

To uncompress, replace `CompressEx` with `UncompressEx`.

Transparent encryption relies on a key seeded from the logged-in user’s password. The system is robust to password changes performed by the authenticated user, but if a password is reset via an administrator, data in encrypted files is unrecoverable.

###### Note

Transparent encryption and compression require special filesystem support. NTFS (used most commonly on hard drives) supports these features; CDFS (on CD-ROMs) and FAT (on removable media cards) do not.

You can determine whether a volume supports compression and encryption with Win32 interop:

[PRE60]

### Windows file security

###### Note

This feature is Windows-only and requires the NuGet package `System.IO.FileSystem.AccessControl`.

The `FileSecurity` class allow you to query and change the OS permissions assigned to users and roles (namespace `System.Security.AccessControl`).

In this example, we list a file’s existing permissions and then assign Write permission to the “Users” group:

[PRE61]

We give another example, later, in [“Special Folders”](#special_folders).

### Unix file security

From .NET 7, the `File` class includes the methods `GetUnix​Fi⁠leMode` and `SetUnix​Fi⁠leMode` to get and set file permissions on Unix systems. The `Directory.CreateDirectory` method is also now overloaded to accept a Unix file mode, and it’s possible to specify a file mode when creating a file, as follows:

[PRE62]

## The Directory Class

The static `Directory` class provides a set of methods analogous to those in the `File` class—for checking whether a directory exists (`Exists`), moving a directory (`Move`), deleting a directory (`Delete`), getting/setting times of creation or last access, and getting/setting security permissions. Furthermore, `Directory` exposes the following static methods:

[PRE63]

###### Note

The last three methods are potentially more efficient than the `Get*` variants because they’re lazily evaluated—fetching data from the file system as you enumerate the sequence. They’re particularly well suited to LINQ queries.

The `Enumerate*` and `Get*` methods are overloaded to also accept `search​Pat⁠tern` (string) and `searchOption` (enum) parameters. If you specify `SearchOp⁠tion​.SearchAllSubDirectories`, a recursive subdirectory search is performed. The `*FileSystemEntries` methods combine the results of `*Files` with `*Directories`.

Here’s how to create a directory if it doesn’t already exist:

[PRE64]

## FileInfo and DirectoryInfo

The static methods on `File` and `Directory` are convenient for executing a single file or directory operation. If you need to call a series of methods in a row, the `FileInfo` and `DirectoryInfo` classes provide an object model that makes the job easier.

`FileInfo` offers most of the `File`’s static methods in instance form—with some additional properties such as `Extension`, `Length`, `IsReadOnly`, and `Directory`—for returning a `DirectoryInfo` object. For example:

[PRE65]

Here’s how to use `DirectoryInfo` to enumerate files and subdirectories:

[PRE66]

## Path

The static `Path` class defines methods and fields for working with paths and filenames.

Assuming this setup code:

[PRE67]

we can demonstrate `Path`’s methods and fields with the following expressions:

| Expression | Result (Windows, then Unix) |
| --- | --- |
| `Directory.GetCurrentDirectory()` | `k:\demo\` or `/demo` |
| `Path.IsPathRooted (file)` | `False` |
| `Path.IsPathRooted (path)` | `True` |
| `Path.GetPathRoot (path)` | `c:\` or `/` |
| `Path.GetDirectoryName (path)` | `c:\mydir` or `/mydir` |
| `Path.GetFileName (path)` | `myfile.txt` |
| `Path.GetFullPath (file)` | `k:\demo\myfile.txt` or `/demo/myfile.txt` |
| `Path.Combine (dir, file)` | `c:\mydir\myfile.txt` or `/mydir/myfile.txt` |
| **File extensions:** |  |
| `Path.HasExtension (file)` | `True` |
| `Path.GetExtension (file)` | `.txt` |
| `Path.GetFileNameWithoutExtension (file)` | `myfile` |
| `Path.ChangeExtension (file, ".log")` | `myfile.log` |
| **Separators and characters:** |  |
| `Path.DirectorySeparatorChar` | `\` or `/` |
| `Path.AltDirectorySeparatorChar` | `/` |
| `Path.PathSeparator` | `;` or`:` |
| `Path.VolumeSeparatorChar` | `:` or `/` |
| `Path.GetInvalidPathChars()` | chars 0 to 31 and `"<>&#124;`eor 0 |
| `Path.GetInvalidFileNameChars()` | chars 0 to 31 and `"<>&#124;:*?\/` or 0 and `/` |
| **Temporary files:** |  |
| `Path.GetTempPath()` | *<local user folder>*\`Temp` or */tmp/* |
| `Path.GetRandomFileName()` | `*d2dwuzjf.dnp*` |
| `Path.GetTempFileName()` | *<local user folder>*\`Temp`\`*tmp14B.tmp*` or */tmp/*`*tmpubSUYO.tmp*` |

`Combine` is particularly useful: it allows you to combine a directory and filename—or two directories—without first having to check whether a trailing path separator is present, and it automatically uses the correct path separator for the OS. It provides overloads that accept up to four directory and/or filenames.

`GetFullPath` converts a path relative to the current directory to an absolute path. It accepts values such as *..\..\file.txt*.

`GetRandomFileName` returns a genuinely unique 8.3-character filename, without actually creating any file. `GetTempFileName` generates a temporary filename using an autoincrementing counter that repeats every 65,000 files. It then creates a zero-byte file of this name in the local temporary directory.

###### Warning

You must delete the file generated by `GetTempFileName` when you’re done; otherwise, it will eventually throw an exception (after your 65,000th call to `GetTempFileName`). If this is a problem, you can instead `Combine GetTempPath` with `GetRandomFileName`. Just be careful not to fill up the user’s hard drive!

## Special Folders

One thing missing from `Path` and `Directory` is a means to locate folders such as *My Documents*, *Program Files*, *Application Data*, and so on. This is provided instead by the `GetFolderPath` method in the `System.Environment` class:

[PRE68]

`Environment.SpecialFolder` is an enum whose values encompass all special directories in Windows, such as `AdminTools`, `ApplicationData`, `Fonts`, `History`, `SendTo`, `StartMenu`, and so on. Everything is covered here except the .NET runtime directory, which you can obtain as follows:

[PRE69]

###### Note

Most of the special folders have no path assigned on Unix systems. The following have paths on Ubuntu Linux 18.04 Desktop: `ApplicationData`, `CommonApplicationData`, `Desktop`, `DesktopDirectory`, `LocalApplicationData`, `MyDocuments`, `MyMusic`, `MyPictures`, `MyVideos`, `Templates`, and `UserProfile`.

Of particular value on Windows systems is `ApplicationData`, where you can store settings that travel with a user across a network (if roaming profiles are enabled on the network domain); `LocalApplicationData`, which is for nonroaming data (specific to the logged-in user); and `CommonApplicationData`, which is shared by every user of the computer. Writing application data to these folders is considered preferable to using the Windows Registry. The standard protocol for storing data in these folders is to create a subdirectory with the name of your application:

[PRE70]

There’s a horrible trap when using `CommonApplicationData`: if a user starts your program with administrative elevation and your program then creates folders and files in `CommonApplicationData`, that user might lack permissions to replace those files later, when run under a restricted Windows login. (A similar problem exists when switching between restricted-permission accounts.) You can work around it by creating the desired folder (with permissions assigned to everyone) as part of your setup.

Another place to write configuration and log files is to the application’s base directory, which you can obtain with `AppDomain.CurrentDomain.BaseDirectory`. This is not recommended, however, because the OS is likely to deny your application permissions to write to this folder after initial installation (without administrative elevation).

## Querying Volume Information

You can query the drives on a computer with the `DriveInfo` class:

[PRE71]

The static `GetDrives` method returns all mapped drives, including CD-ROMs, media cards, and network connections. `DriveType` is an enum with the following values:

[PRE72]

## Catching Filesystem Events

The `FileSystemWatcher` class lets you monitor a directory (and optionally, subdirectories) for activity. `FileSystemWatcher` has events that fire when files or subdirectories are created, modified, renamed, and deleted, as well as when their attributes change. These events fire regardless of the user or process performing the change. Here’s an example:

[PRE73]

###### Warning

Because `FileSystemWatcher` raises events on a separate thread, you must exception-handle the event handling code to prevent an error from taking down the application. For more information, see [“Exception Handling”](ch14.html#exception_handling).

The `Error` event does not inform you of filesystem errors; instead, it indicates that the `FileSystemWatcher`’s event buffer overflowed because it was overwhelmed by `Changed`, `Created`, `Deleted`, or `Renamed` events. You can change the buffer size via the `InternalBufferSize` property.

`IncludeSubdirectories` applies recursively. So, if you create a `FileSystemWatcher` on *C:\* with `IncludeSubdirectories true`, its events will fire when a file or directory changes anywhere on the hard drive.

###### Warning

A trap in using `FileSystemWatcher` is to open and read newly created or updated files before the file has been fully populated or updated. If you’re working in conjunction with some other software that’s creating files, you might need to consider some strategy to mitigate this, such as creating files with an unwatched extension and then renaming them after they’re fully written.

# OS Security

All applications are subject to OS restrictions, based on the user’s login privileges. These restrictions affect file I/O as well as other capabilities, such as access to the Windows Registry.

In Windows and Unix, there are two types of accounts:

*   An administrative/superuser account that imposes no restrictions in accessing the local computer

*   A limited permissions account that restricts administrative functions and visibility of other users’ data

On Windows, a feature called User Account Control (UAC) means that administrators receive two tokens or “hats” when logging in: an administrative hat and an ordinary user hat. By default, programs run wearing the ordinary user hat—with restricted permissions—unless the program requests *administrative elevation*. The user must then approve the request in the dialog box that’s presented.

On Unix, users typically log in with restricted accounts. That is also true for administrators to lessen the probability of inadvertently damaging the system. When a user needs to run a command that requires elevated permissions, they precede the command with `sudo` (short for “super-user do”).

*By default*, your application will run with restricted user privileges. This means that you must either:

*   Write your application such that it can run without administrative privileges.

*   Demand administrative elevation in the application manifest (Windows only), or detect the lack of required privileges and alert the user to restart the application as an administrator/super-user.

The first option is safer and more convenient for the user. Designing your program to run without administrative privileges is easy in most cases.

You can find out whether you’re running under an administrative account as follows:

[PRE74]

With UAC enabled on Windows, this returns `true` only if the current process has administrative elevation. On Linux, it returns `true` only if the current process is running as super-user (e.g., *sudo myapp*).

## Running in a Standard User Account

Here are the key things that you *cannot* do in a standard user account:

*   Write to the following directories:

    *   The OS folder (typically *\Windows* or */bin, /sbin, ...*) and subdirectories

    *   The program files folder (*\Program Files* or */usr/bin, /opt*) and subdirectories

    *   The root of the OS drive (e.g., *C:\* or */*)

*   Write to the HKEY_LOCAL_MACHINE branch of the Registry (Windows)

*   Read performance monitoring (WMI) data (Windows)

Additionally, as an ordinary Windows user (or even as an administrator), you might be refused access to files or resources that belong to other users. Windows uses a system of Access Control Lists (ACLs) to protect such resources—you can query and assert your own rights in the ACLs via types in `System.Security.AccessControl`. ACLs can also be applied to cross-process wait handles, described in [Chapter 21](ch21.html#advanced_threadin).

If you’re refused access to anything as a result of OS security, the CLR detects the failure and throws an `UnauthorizedAccessException` (rather than failing silently).

In most cases, you can deal with standard user restrictions as follows:

*   Write files to their recommended locations.

*   Avoid using the Registry for information that can be stored in files (aside from the HKEY_CURRENT_USER hive, which you will have read/write access to on Windows only).

*   Register ActiveX or COM components during setup (Windows only).

The recommended location for user documents is `SpecialFolder.MyDocuments`:

[PRE75]

The recommended location for configuration files that a user might need to modify outside of your application is `SpecialFolder.ApplicationData` (current user only) or `SpecialFolder.CommonApplicationData` (all users). You typically create subdirectories within these folders, based on your organization and product name.

## Administrative Elevation and Virtualization

With an *application manifest*, you can request that Windows prompt the user for administrative elevation whenever running your program (Linux ignores this request):

[PRE76]

(We describe application manifests in more detail in [Chapter 17](ch17.html#assemblies).)

If you replace `requireAdministrator` with `asInvoker`, it instructs Windows that administrative elevation is *not* required. The effect is almost the same as not having an application manifest at all—except that *virtualization* is disabled. Virtualization is a temporary measure introduced with Windows Vista to help old applications run correctly without administrative privileges. The absence of an application manifest with a `requestedExecutionLevel` element activates this backward-compatibility feature.

Virtualization comes into play when an application writes to the *Program Files* or *Windows* directory, or the HKEY_LOCAL_MACHINE area of the Registry. Instead of throwing an exception, changes are redirected to a separate location on the hard disk where they can’t affect the original data. This prevents the application from interfering with the OS—or other well-behaved applications.

# Memory-Mapped Files

*Memory-mapped files* provide two key features:

*   Efficient random access to file data

*   The ability to share memory between different processes on the same computer

The types for memory-mapped files reside in the `System.IO.MemoryMappedFiles` namespace. Internally, they work by wrapping the operating system’s API for memory-mapped files.

## Memory-Mapped Files and Random File I/O

Although an ordinary `FileStream` allows random file I/O (by setting the stream’s `Position` property), it’s optimized for sequential I/O. As a rough rule of thumb:

*   `FileStream`s are approximately 10 times faster than memory-mapped files for sequential I/O.

*   Memory-mapped files are approximately 10 times faster than `FileStream`s for random I/O.

Changing a `FileStream`’s `Position` can cost several microseconds—which adds up if done within a loop. A `FileStream` is also unsuitable for multithreaded access—because its position changes as it is read or written.

To create a memory-mapped file:

1.  Obtain a `FileStream` as you would ordinarily.

2.  Instantiate a `MemoryMappedFile`, passing in the file stream.

3.  Call `CreateViewAccessor` on the memory-mapped file object.

The last step gives you a `MemoryMappedViewAccessor` object that provides methods for randomly reading and writing simple types, structures, and arrays (more on this in [“Working with View Accessors”](#working_with_view_accessors)).

The following creates a one million–byte file and then uses the memory-mapped file API to read and then write a byte at position 500,000:

[PRE77]

You can also specify a map name and capacity when calling `CreateFromFile`. Specifying a non-null map name allows the memory block to be shared with other processes (see the following section); specifying a capacity automatically enlarges the file to that value. The following creates a 1,000-byte file:

[PRE78]

## Memory-Mapped Files and Shared Memory (Windows)

Under Windows, you can also use memory-mapped files as a means of sharing memory between processes on the same computer. One process creates a shared memory block by calling `MemoryMappedFile.CreateNew`, and then other processes subscribe to that same memory block by calling `MemoryMappedFile.OpenExisting` with the same name. Although it’s still referred to as a memory-mapped “file,” it resides entirely in memory and has no disk presence.

The following code creates a 500-byte shared memory-mapped file and writes the integer 12345 at position 0:

[PRE79]

The following code opens that memory-mapped file and reads that integer:

[PRE80]

## Cross-Platform Interprocess Shared Memory

Both Windows and Unix allow multiple processes to memory-map the same file. You must exercise care to ensure appropriate file sharing settings:

[PRE81]

## Working with View Accessors

Calling `CreateViewAccessor` on a `MemoryMappedFile` gives you a view accessor that lets you read/write values at random positions.

The `Read*`/`Write*` methods accept numeric types, `bool`, and `char`, as well as arrays and structs that contain value-type elements or fields. Reference types—and arrays or structs that contain reference types—are prohibited because they cannot map into unmanaged memory. So, if you want to write a string, you must encode it into an array of bytes:

[PRE82]

Notice that we wrote the length first. This means we know how many bytes to read back later:

[PRE83]

Here’s an example of reading/writing a struct:

[PRE84]

The `Read` and `Write` methods are surprisingly slow. You can get much better performance by directly accessing the underlying unmanaged memory via a pointer. Following on from the previous example:

[PRE85]

Your project must be configured to allow unsafe code. You can do that by editing your `.csproj` file:

[PRE86]

The performance advantage of pointers is even more pronounced when working with large structures because they let you work directly with the raw data rather than using `Read`/`Write` to *copy* data between managed and unmanaged memory. We explore this further in [Chapter 24](ch24.html#native_and_com_interoperabilit).