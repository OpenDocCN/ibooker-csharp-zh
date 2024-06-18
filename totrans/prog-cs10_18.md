# Chapter 18\. Memory Efficiency

As [Chapter 7](ch07.xhtml#ch_object_lifetime) described, the CLR is able to perform automatic memory management thanks to its garbage collector (GC). This comes at a price: when a CPU spends time on garbage collection, that stops it from getting on with more productive work. On laptops and phones, GC work drains power from the battery. In a cloud computing environment where you may be paying for CPU time based on consumption, extra work for the CPU corresponds directly to increased costs. More subtly, on a computer with many cores, spending too much time in the GC can dramatically reduce throughput, because many of the cores may end up blocked, waiting for the GC to complete before they can proceed.

In many cases, these effects will be small enough not to cause visible problems. However, when certain kinds of programs experience heavy load, GC costs can come to dominate the overall execution time. In particular, if you write code that performs relatively simple but highly repetitive processing, GC overhead can have a substantial impact on throughput.

To give you an example of the kinds of improvements that can sometimes be possible, early versions of Microsoft’s ASP.NET Core web server framework frequently ran into hard limits due to GC overhead. To enable .NET applications to break through these barriers, C# introduced various features that can enable dramatic reductions in the number of allocations. Fewer allocations means fewer blocks of memory for the GC to recover, so this translates directly to lower GC overhead. When ASP.NET Core first started making extensive use of these features, performance improved across the board, but for the simplest performance benchmark, known as *plaintext* (part of the TechEmpower suite of web performance tests), this release improved the request handling rate by over 25%.

In some specialized scenarios, the differences can be more dramatic. In 2019, I worked on a project that processed diagnostic information from a broadband provider’s networking equipment (in the form of RADIUS packets). Adopting the techniques described in this chapter boosted the rate at which a single CPU core in our system could process the messages from around 300,000/s to about 7 million/s.

There is a price to pay, of course: these GC-efficient techniques add significant complication to your code. And the payoff won’t always be so large—although the first ASP.NET Core release to be able to use these features improved over the previous version on all benchmarks, only the simplest shows a 25% boost, and most improved more modestly. The practical improvement will really depend on the nature of your workload, and for some applications you might find that applying these techniques delivers no measurable improvement. So before you even consider using them, you should use performance monitoring tools to find out how much time your code spends in the GC. If it’s only a few percent, then you might not be able to realize order-of-magnitude improvements. But if testing suggests that there’s room for significant improvement, the next step is to ask whether the techniques in this chapter are likely to help. So let’s start by exploring exactly how these new techniques can help you reduce GC overhead.

# (Don’t) Copy That

The way to reduce GC overhead is to allocate less memory on the heap. And the most important technique for minimizing allocations is to avoid making copies of data. For example, consider the URL *http://example.com/books/1323?edition=6&format=pdf*. There are several elements of interest in here, such as the protocol (`http`), the hostname (`example.com`), or the query string. The latter has its own structure: it is a sequence of name/value pairs. The obvious way to work with a URL in .NET is to use the `System.Uri` type, as [Example 18-1](#ex_basic_url_parsing) shows.

##### Example 18-1\. Deconstructing a URL

[PRE0]

It produces the following output:

[PRE1]

This is convenient, but by getting the values of these four properties, we have forced the `Uri` to provide four `string` objects in addition to the original one. You could imagine a smart implementation of `Uri` that recognized certain standard values for `Scheme`, such as `http`, and that always returned the same string instance for these instead of allocating new ones, but for all the other parts, it’s likely to have to allocate new strings on the heap.

There is another way. Instead of creating new `string` objects for each section, we could take advantage of the fact that all of the information we want was already in the string containing the whole URL. There’s no need to copy each section into a new string, when instead we can just keep track of the position and lengths of the relevant sections within the string. Instead of creating a string for each section, we would need just two numbers. And since we can represent numbers using value types (e.g., `int` or, for very long strings, `long`), we don’t need any additional objects on the heap beyond the single string with the full URL. For example, the scheme (`http`) is at position 0 and has length 4\. [Figure 18-1](#fig_UrlByOffsetAndSize) shows each of the elements by their offset and position within the string.

![URL elements by their offset and size in a string](assets/pc10_1801.png)

###### Figure 18-1\. URL substrings

This works, but already we can see the first problem with working this way: it is somewhat awkward. Instead of representing, say, the `Host` with a convenient `string` object, which is easily understood and readily inspected in the debugger, we now have a pair of numbers, and as developers, we now have to remember which string they point into. It’s not rocket science, but it makes it slightly harder to understand our code, and easier to introduce bugs. But there’s a payoff: instead of five strings (the original URL and the four properties), we just have one. And if you’re trying to process millions of events each second, that could easily be worth the effort.

Obviously this technique would work for a more fine-grained structure too. The offset and position `(25, 4)` locates the text `1323` in this URL. We might want to parse that as an `int`. But at this point we run into the second problem with this style of working: it is not widely supported in .NET libraries. The usual way to parse text into an `int` is to use the `int` type’s static `Parse` or `TryParse` methods. Unfortunately, these do not provide overloads that accept a position or offset within a `string`. They require a string containing only the number to be parsed. This means you end up writing code such as [Example 18-2](#ex_substring_defeating_the_purpose).

##### Example 18-2\. Defeating the point of the exercise by using `Substring`

[PRE2]

This works, but by using `Substring` to go from our (offset, length) representation back to the plain `string` that `int.Parse` wants, we’ve allocated a new `string`. The whole point of this exercise was to reduce allocations, so this doesn’t seem like progress. One solution might be for Microsoft to go through the entire .NET API surface area, adding overloads that accept offset and length parameters in any situation where we might want to work with something in the middle of something else (either a substring, as in this example, or perhaps a subrange of an array). In fact, there are examples of this already: the `Stream` API for working with byte streams has various methods that accept a `byte[]` array, and also offset and length arguments to indicate exactly which part of the array you want to work with.

However, there’s one more problem with this technique: it is inflexible about the type of container that the data lives in. Microsoft could add an overload to `int.Parse` that takes a `string`, an offset, and a length, but it would only be able to parse data inside a `string`. What if the data happens to be in a `char[]`? In that case, you’d have to convert it to a `string` first, at which point we’re back to additional allocations. Alternatively, every API that wants to support this approach would need multiple overloads to support all the containers that anyone might want to use, each potentially requiring a different implementation of the same basic method.

More subtly, what if the data you have is currently in memory that’s not on the CLR’s heap? This is a particularly important question when it comes to the performance of servers that accept requests over the network (e.g., a web server). Sometimes it is not possible to arrange for data received by a network card to be delivered directly into memory on .NET’s heap. Also, some forms of interprocess communication involve arranging for the OS to map a particular region of memory into two different processes’ address spaces. The .NET heap is local to the process and cannot use such memory.

C# has always supported use of external memory through *unsafe code*, which supports raw unmanaged pointers that work in a similar way to pointers in the C and C++ languages. However, there are a couple of problems with these. First, they would add yet another entry to the list of overloads that everything would need to support in a world where we can parse data in place. Second, code using pointers cannot pass .NET’s type safety verification rules. This means it becomes possible to make certain kinds of programming errors that are normally impossible in C#. It may also mean that the code will not be allowed to run in certain scenarios, since the loss of type safety would enable unsafe code to bypass certain security constraints.

To summarize, it has always been possible to reduce allocations and copying in .NET by working with offsets and lengths and either a reference to a containing string or array or an unmanaged pointer to memory, but there was considerable room for improvement on these fronts:

*   Convenience

*   Wide support across .NET APIs

*   Unified, safe handling of the following:

    *   Strings

    *   Arrays

    *   Unmanaged memory

.NET offers a type that addresses all three points: `Span<T>`. (See the next sidebar, [“Support Across Language and Runtime Versions”](#span_runtime_and_langauge_versions_sidebar), for more information on how the features described in this chapter relate to C# language and .NET runtime versions.)

# Representing Sequential Elements with Span<T>

The `System.Span<T>` value type represents a sequence of elements of type `T` stored contiguously in memory. Those elements can live inside an array, a string, a managed block of memory allocated in a stack frame, or unmanaged memory. Let’s look at how `Span<T>` addresses each of the requirements enumerated in the preceding section.

A `Span<T>` encapsulates three things: a pointer or reference to the containing memory (e.g., the `string` or array), the position of the data within that memory, and its length.^([1](ch18.xhtml#idm45884782468272)) To access the contents of a span, you use it much as you would an array, as [Example 18-3](#ex_sum_span_contents) shows. This makes it much more convenient to use than ad hoc techniques in which you define a couple of `int` variables and have to remember what they refer to.

##### Example 18-3\. Iterating over a `Span<int>`

[PRE3]

Since a `Span<T>` knows its own length, its indexer checks that the index is in range, just as the built-in array type does. And if you are running on .NET Core or .NET, the performance is very similar to using a built-in array. This includes the optimizations that detect certain loop patterns—for example, the CLR will recognize the preceding code as a loop that iterates over the entire contents, enabling it to generate code that doesn’t need to check that the index is in range each time around the loop. In some cases it is even able to generate code that uses the vector-oriented instructions available in some CPUs to accelerate the loop. (On .NET Framework, `Span<T>` is a little slower than an array, because its CLR does not include the optimizations that were added in .NET Core to support `Span<T>`.)

You may have noticed that the method in [Example 18-3](#ex_sum_span_contents) takes a `ReadOnlySpan<T>`. This is a close relative of `Span<T>`, and there is an implicit conversion enabling you to pass any `Span<T>` to a method that takes a `ReadOnlySpan<T>`. The read-only form enables a method to declare clearly that it will only read from the span, and not write to it. (This is enforced by the fact that the read-only form’s indexer offers just a `get` accessor, and no `set`.)

###### Tip

Whenever you write a method that works with a span and that does not mean to modify it, you should use `ReadOnlySpan<T>`.

There are implicit conversions from the various supported containers to `Span<T>` (and also to `ReadOnlySpan<T>`). This enables [Example 18-4](#ex_pass_int_array_as_span) to pass an array to the `SumSpan` method.

##### Example 18-4\. Passing an `int[]` as a `ReadOnlySpan<int>`

[PRE4]

Of course, we’ve gone and allocated an array on the heap there, so this particular example defeats the whole point of using spans, but if you already have an array on hand, this is a useful technique. `Span<T>` also works with stack-allocated arrays, as [Example 18-5](#ex_pass_stack_int_array_as_span) shows. (The `stackalloc` keyword enables you to create an array in memory allocated on the current stack frame.)

##### Example 18-5\. Passing a stack-allocated array as a `ReadOnlySpan<int>`

[PRE5]

Normally, C# won’t allow you to use `stackalloc` outside of code marked as `unsafe`. The keyword allocates memory on the current method’s stack frame, and it does not create a real array object. Arrays are reference types, so they must live on the GC heap. A `stackalloc` expression produces a pointer type, because it produces plain memory without the usual .NET object headers. In this case, it would be an `int*`. You can only use pointer types directly in unsafe code blocks. However, the compiler makes an exception to this rule if you assign the pointer produced by a `stackalloc` expression directly into a span. This is permitted because spans impose bounds checking, preventing undetected out-of-range access errors of the kind that normally make pointers unsafe. Also, `Span<T>` and `ReadOnlySpan<T>` are both defined as `ref struct` types, and as [“Stack Only”](#stack_only) describes, this means they cannot outlive their containing stack frame. This guarantees that the stack frame on which the stack-allocated memory lives will not vanish while there are still outstanding references to it. (.NET’s type safety verification rules include special handling for spans.)

Earlier I mentioned that spans can refer to strings as well as arrays. However, we can’t pass a `string` to this `SumSpan` for the simple reason that it requires a span with an element type of `int`, whereas a `string` is a sequence of `char` values. `int` and `char` have different sizes—they take 4 and 2 bytes each, respectively. Although an implicit conversion exists between the two (meaning you can assign a `char` value into an `int` variable, giving you the Unicode value of the `char`), that does not make a `ReadOnlySpan<char>` implicitly compatible with a `ReadOnlySpan<int>`.^([2](ch18.xhtml#idm45884782266080)) Remember, the entire point of spans is that they provide a view into a block of data without needing to copy or modify that data; since `int` and `char` have different sizes, converting a `char[]` to an `int[]` array would double its size. However, if we were to write a method accepting a `ReadOnlySpan<char>`, we would be able to pass it a `string`, a `char[]` array, a `stackalloc char[]`, or an unmanaged pointer of type `char*` (because the in-memory representation of a particular span of characters within each of these is the same).

###### Note

Since strings are immutable in .NET, you cannot convert a `string` to a `Span<char>`. You can only convert it to a `ReadOnlySpan<char>`.

We’ve examined two of our requirements from the preceding section: `Span<T>` is easier to use than ad hoc storing of an offset and length, and it makes it possible to write a single method that can work with data in arrays, strings, the stack, or unmanaged memory. This leaves our final requirement: widespread support throughout .NET’s runtime libraries. As [Example 18-6](#ex_parse_string_in_situ) shows, it is now supported in `int.Parse`, enabling us to fix the problem shown in [Example 18-2](#ex_substring_defeating_the_purpose).

##### Example 18-6\. Parsing integers in a string using `Span<char>`

[PRE6]

`Span<T>` is a relatively new type (it was introduced in 2018; .NET has been around since 2002), so although the .NET runtime libraries now support it widely, many third-party libraries do not yet support it, and perhaps never will. However, it has become increasingly well supported since being introduced, and the situation will only improve.

## Utility Methods

In addition to the array-like indexer and `Length` properties, `Span<T>` offers a few useful methods. The `Clear` and `Fill` methods provide convenient ways to initialize all the elements in a span either to the default value for the element type or a specific value. Obviously, these are not available on `ReadOnlySpan<T>`.

You may sometimes encounter situations in which you have a span and you need to pass its contents to a method that requires an array. Obviously there’s no avoiding an allocation in this case, but if you need to do it, you can use the `ToArray` method.

Spans (both normal and read-only) also offer a `TryCopyTo` method, which takes as its argument a (non-read-only) span of the same element type. This allows you to copy data between spans. This method handles scenarios where the source and target spans refer to overlapping ranges within the same container. As the `Try` suggests, it’s possible for this method to fail: if the target span is too small, this method returns `false`.

## Stack Only

The `Span<T>` and `ReadOnlySpan<T>` types are both declared as `ref struct`. This means that not only are they value types, they are value types that can live only on the stack. So you cannot have fields with span types in a `class`, or in any `struct` that is not also a `ref struct`. This also imposes some potentially more surprising restrictions. For example, it means you cannot use a span in a variable in an `async` method. (These store all their variables as fields in a hidden type, enabling them to live on the heap, because asynchronous methods often need to outlive their original stack frame. In fact, these methods can even switch to a completely different stack altogether, because asynchronous methods can end up running on different threads as their execution progresses.) For similar reasons, there are restrictions on using spans in anonymous functions and in iterator methods. You can use them in local methods, and you can even declare a `ref struct` variable in the outer method and use it from the nested one, but with one restriction: you must not create a delegate that refers to that local method, because this would cause the compiler to move shared variables into an object that lives on the heap. (See [Chapter 9](ch09.xhtml#ch_delegates_lambdas_events) for details.)

This restriction is necessary for .NET to be able to offer the combination of array-like performance, type safety, and the flexibility to work with multiple different containers. For situations in which this stack-only limitation is problematic, we have the `Memory<T>` type.

# Representing Sequential Elements with Memory<T>

The `Memory<T>` type and its counterpart, `ReadOnlyMemory<T>`, represent the same basic concept as `Span<T>` and `ReadOnlySpan<T>`: these types provide a uniform view over a contiguous sequence of elements of type `T` that could reside in an array, unmanaged memory, or, if the element type is `char`, a `string`. But unlike spans, these are *not* `ref struct` types, so they can be used anywhere. The downside is that this means they cannot offer the same high performance as spans. (It also means you cannot create a `Memory<T>` that refers to `stackalloc` memory.)

You can convert a `Memory<T>` to a `Span<T>`, and likewise a `ReadOnlyMemory<T>` to a `ReadOnlySpan<T>`, as long as you’re in a context where spans are allowed (e.g., in an ordinary method but not an asynchronous one). The conversion to a span has a cost. It is not massive, but it is significantly higher than the cost of accessing an individual element in a span. (In particular, many of the optimizations that make spans attractive only become effective with repeated use of the same span.) So if you are going to read or write elements in a `Memory<T>` in a loop, you should perform the conversion to `Span<T>` just once, outside of the loop, rather than doing it each time around. If you can work entirely with spans, you should do so since they offer the best performance. (And if you are not concerned with performance, then this is not the chapter for you!)

# ReadOnlySequence<T>

The types we’ve looked at so far in this chapter all represent contiguous blocks of memory. Unfortunately, data doesn’t always neatly present itself to us in the most convenient possible form. For example, on a busy server that is handling many concurrent requests, the network messages for requests in progress often become interleaved—if a particular request is large enough to need to be split across two network packets, it’s entirely possible that after receiving the first but before receiving the second of these, one or more packets for other, unrelated requests could arrive. So by the time we come to process the contents of the request, it might be split across two different chunks of memory. Since span and memory values can each represent only a contiguous range of elements, .NET provides another type, `ReadOnlySequence`, to represent data that is conceptually a single sequence but that has been split into multiple ranges.

###### Note

There is no corresponding `Sequence<T>`. Unlike spans and memory, this particular abstraction is available only in read-only form. That’s because it’s common to need to deal with fragmented data as a reader, where you don’t control where the data lives, but if you are producing data, you are more likely to be in a position to control where it goes.

Now that we’ve seen the main types for working with data while minimizing the number of allocations, let’s look at how these can all work together to handle high volumes of data. To coordinate this kind of processing, we need to look at one more feature: pipelines.

# Processing Data Streams with Pipelines

Everything we’re looking at in this chapter is designed to enable safe, efficient processing of large volumes of data. The types we’ve seen so far all represent information that is already in memory. We also need to think about how that data is going to get into memory in the first place. The preceding section hinted at the fact that this can be somewhat messy. The data will very often be split into chunks, and not in a way designed for the convenience of the code processing the data, because it will likely be arriving either over a network or from a disk. If we’re to realize the performance benefits made possible by `Span<T>` and its related types, we need to pay close attention to the job of getting data into memory in the first place and the way in which this data fetching process cooperates with the code that processes the data. Even if you are only going to be writing code that consumes data—perhaps you are relying on a framework such as ASP.NET Core to get the data into memory for you—it is important to understand how this process works.

The `System.Io.Pipelines` NuGet package defines a set of types in a namespace of the same name that provide a high-performance system for loading data from some source that tends to split data into inconveniently sized chunks, and passing that data over to code that wants to be able to process it in situ using spans. [Figure 18-2](#fig_PipelineOverview) shows the main participants in a pipeline-based process.

At the heart of this is the `Pipe` class. It offers two properties: `Writer` and `Reader`. The first returns a `PipeWriter`, which is used by the code that loads the data into memory. (This often doesn’t need to be application-specific. For example, in a web application, you can let ASP.NET Core control the writer on your behalf.) The `Reader` property’s type is, predictably, `PipeReader`, and this is most likely to be the part your code interacts with.

![An overview of the participants in a pipeline](assets/pc10_1802.png)

###### Figure 18-2\. Pipeline overview

The basic process for reading data from a pipe is as follows. First, you call `Pipe​Rea⁠der.⁠Rea⁠dAs⁠ync`. This returns a task,^([3](ch18.xhtml#idm45884782152672)) because if no data is available yet, you will need to wait until the data source supplies the writer with some data. Once data is available, the task will provide a `ReadResult` object. This supplies a `ReadOnlySequence<T>`, which presents the available data as one or more `ReadOnlySpan<T>` values. The number of spans will depend on how fragmented the data is. If it’s all conveniently in one place in memory, there will be just one span, but code using a reader needs to be able to cope with more. Your code should then process as much of the available data as it can. Once it has done this, it calls the reader’s `AdvanceTo` to tell it how much of the data your code has been able to process. Then, if the `ReadResult.IsComplete` property is false, we will repeat these steps again from the call to `ReadAsync`.

An important detail of this is that we are allowed to tell the `PipeReader` that we couldn’t process everything it gave us. This would normally be because the information got sliced into pieces, and we need to see some of the next chunk before we can fully process everything in the current one. For example, a JSON message large enough to need to be split across several network packets will probably end up with splits in inconvenient places. So you might find that the first chunk looks like this:

[PRE7]

And the second like this:

[PRE8]

In practice the chunks would be bigger, but this illustrates the basic problem: the chunks that a `PipeReader` returns are likely to slice across the middle of important features. With most .NET APIs, you never have to deal with this kind of mess because everything has been cleaned up and reassembled by the time you see it, but the price you pay for that is the allocation of new strings to hold the recombined results. If you want to avoid those allocations, you have to handle these challenges.

There are a couple of ways to deal with this. One is for code reading data to maintain enough state to be able to stop and later restart at any point in the sequence. So code processing this JSON might choose to remember that it is partway through an object and that it’s in the middle of processing a property whose name starts with `prope`. But `PipeReader` offers an alternative. Code processing these examples could report with its call to `AdvanceTo` that it has consumed everything up to the first comma. If you do that, the `Pipe` will remember that we’re not yet finished with this first block, and when the next call to `ReadAsync` completes, the `ReadOnlySequence<T>` in `ReadResult.Buffer` will now include at least two spans: the first span will point into the same block of memory as last time, but now its offset will be set to where we got to last time—that first span will refer to the `"prope` text at the end of the first block. And then the second span will refer to the text in the second chunk.

The advantage of this second approach is that the code processing the data doesn’t need to remember as much between calls to `ReadAsync`, because it knows it’ll be able to go back and look at the previously unprocessed data again once the next chunk arrives, at which point it should now be able to make sense of it.

In practice, this particular example is fairly easy to cope with because there’s a type in the runtime libraries called `Utf8JsonReader` that can handle all the awkward details around chunk boundaries for us. Let’s look at an example.

## Processing JSON in ASP.NET Core

Suppose you are developing a web service that needs to handle HTTP requests containing JSON. This is a pretty common scenario. [Example 18-7](#ex_json_http_handler) shows the typical way to do this in ASP.NET Core. This is reasonably straightforward, but it does not use any of the low-allocation mechanisms discussed in this chapter, so this forces ASP.NET Core to allocate multiple objects for each request.

##### Example 18-7\. Handling JSON in HTTP requests

[PRE9]

Before we look at how to change it, for readers not familiar with ASP.NET Core, I will quickly explain what’s happening in this example. The `CreateJob` method is annotated with attributes telling ASP.NET Core that this will handle HTTP POST requests where the URL path is `/jobs/create`. The `[FromBody]` attribute on the method’s argument indicates that we expect the body of the request to contain data in the form described by the `JobDescription` type. ASP.NET Core can be configured to handle various data formats, but if you go with the defaults, it will expect JSON.

This example is therefore telling ASP.NET Core that for each POST request to `/jobs/create`, it should construct a `JobDescription` object, populating its `Dep⁠art⁠ment​Id` and `JobCategory` from properties of the same name in JSON in the incoming request body.

In other words, we’re asking ASP.NET Core to allocate two objects—a `Job​Des⁠cri⁠pti⁠on` and a `string`—for each request, each of which will contain copies of information that was in the body of the incoming request. (The other property, `DepartmentId`, is an `int`, and since that’s a value type, it lives inside the `Job​Des⁠crip⁠tion` object.) And for most applications that will be fine—a couple of allocations is not normally anything to worry about in the course of handling a single web request. However, in more realistic examples with more complex requests, we might then be looking at a much larger number of properties, and if you need to handle a very high volume of requests, the copying of data into a `string` for each property can start to cause enough extra work for the GC that it becomes a performance problem.

[Example 18-8](#ex_frugal_json_http_handler) shows how we can avoid these allocations using the various features described in the preceding sections of this chapter. It makes the code a good deal more complex, demonstrating why you should only apply these kinds of techniques in cases where you have established that GC overhead is high enough that the extra development effort is justified by the performance improvements.

##### Example 18-8\. Handling JSON without allocations

[PRE10]

Instead of defining an argument with a `[FromBody]` attribute, this method works directly with the `this.Request.BodyReader` property. (Inside an ASP.NET Core MVC controller class, `this.Request` returns an object representing the request being handled.) This property’s type is `PipeReader`, the consumer side of a `Pipe`. ASP.NET Core creates the pipe, and it manages the data production side, feeding data from incoming requests into the associated `PipeWriter`.

As the property name suggests, this particular `PipeReader` enables us to read the contents of the HTTP request’s body. By reading the data this way, we make it possible for ASP.NET Core to present the request body to us in situ: our code will be able to read the data directly from wherever it happened to end up in memory once the computer’s network card received it. (In other words, no copies, and no additional GC overhead.)

The `while` loop in `CreateJobFrugalAsync` performs the same process you’ll see with any code that reads data from a `PipeReader`: it calls `ReadAsync`, processes the data that returns, and calls `AdvanceTo` to let the `PipeReader` know how much of that data it was able to process. We then check the `IsComplete` property of the `ReadResult` returned by `ReadAsync`, and if that is `false`, then we go round one more time.

[Example 18-8](#ex_frugal_json_http_handler) uses the `Utf8JsonReader` type to read the data. As the name suggests, this works directly with text in UTF-8 encoding. This alone can provide a significant performance improvement: JSON messages are commonly sent with this encoding, but .NET strings use UTF-16\. So one of the jobs that the simpler [Example 18-7](#ex_json_http_handler) forced ASP.NET to do was convert any strings from UTF-8 to UTF-16\. On the other hand, we’ve lost some flexibility. The simpler, slower approach has the benefit of being able to adapt to incoming requests in more formats: if a client chose to send its request in something other than UTF-8—perhaps UTF-16 or UCS-32, or even a non-Unicode encoding such as ISO-8859-1—our handler could cope with any of them, because ASP.NET Core can do the string conversions for us. But since [Example 18-8](#ex_frugal_json_http_handler) works directly with the data in the form the client transmitted, using a type that only understands UTF-8, we have traded off that flexibility in exchange for higher performance.

`Utf8JsonReader` is able to handle the tricky chunking issues for us—if an incoming request ends up being split across multiple buffers in memory because it was too large to fit in a single network packet, `Utf8JsonReader` is able to cope. In the event of an unhelpfully placed split, it will process what it can, and then the `JsonReaderState` value it returns through its `CurrentState` will report a `Position` indicating the first unprocessed character. We pass this to `PipeReader.AdvanceTo`. The next call to `PipeReader.ReadAsync` will return only when there is more data, but its `ReadResult.Buffer` will also include the previously unconsumed data.

Like the `ReadOnlySpan<T>` type it uses internally when reading data, `Utf8JsonReader` is a `ref struct` type, meaning that it cannot live on the heap. This means it cannot be used in an `async` method, because `async` methods store all of their local variables on the heap. That is why this example has a separate method, `ProcessBuffer`. The outer `CreateJobFrugalAsync` method has to be `async` because the streaming nature of the `PipeReader` type means that its `ReadAsync` method requires us to use `await`. But the `Utf8JsonReader` cannot be used in an `async` method, so we end up having to split our logic across two methods.

###### Note

When splitting your pipeline processing into an outer `async` reader loop and an inner method that avoids async in order to use `ref struct` types, it can be convenient to make the inner method a local method, as [Example 18-8](#ex_frugal_json_http_handler) does. This enables it to access variables declared in the outer method. You might be wondering whether this causes a hidden extra allocation—to enable sharing of variables in this way, the compiler generates a type, storing shared variables in fields in that type and not as conventional stack-based variables. With lambdas and other anonymous methods, this type will indeed cause an additional allocation, because it needs to be a heap-based type so that it can outlive the parent method. However, with local methods, the compiler uses a `struct` to hold the shared variables, which it passes by reference to the inner method, thus avoiding any extra allocation. This is possible because the compiler can determine that all calls to the local method will return before the outer method returns.

When using `Utf8JsonReader`, our code has to be prepared to receive the content in whatever order it happens to arrive. We can’t write code that tries to read the properties in an order that is convenient for us, because that would rely on something holding those properties and their values in memory. (If you tried to rely on going back to the underlying data to retrieve particular properties on demand, you might find that the property you wanted was in an earlier chunk that’s no longer available.) This defeats the whole goal of minimizing allocations. If you want to avoid allocations, your code needs to be flexible enough to handle the properties in whatever order they appear.

So the `ProcessBuffer` code in [Example 18-8](#ex_frugal_json_http_handler) just looks at each JSON element as it comes and works out whether it’s of interest. This means that when looking for particular property values, we have to notice the `PropertyName` element, and then remember that this was the last thing we saw, so that we know how to handle the `Number` or `String` element that follows, containing the value.

One strikingly odd feature of this code is the way it checks for particular strings. It needs to recognize properties of interest (`JobCategory` and `DepartmentId` in this example). But we can’t just use normal string comparison. While it’s possible to retrieve property names and string values as .NET strings, doing so defeats the main purpose of using `Utf8JsonReader`: if you obtain a `string`, the CLR has to allocate space for that string on the heap and will eventually have to garbage collect the memory. (In this example, every acceptable incoming string is known in advance. In some scenarios there will be user-supplied strings whose values you will need to perform further processing on, and in those cases, you may just need to accept the costs of allocating an actual `string`.) So instead we end up performing binary comparisons. Notice that we’re working entirely in UTF-8 encoding, and not the UTF-16 encoding used by .NET’s `string` type. (The various static fields, such as `Utf8TextJobCategory` and `Utf8TextDepartmentId`, are all byte arrays created through `Encoding.UTF8` from the `System.Text` namespace.) That’s because all of this code works directly against the request’s payload in the form in which it arrived over the network, in order to avoid unnecessary copying.

# Summary

APIs that break data down into the constituent components can be very convenient to use, but this convenience comes at a price. Each time we want some subelement represented either as a string or a child object, we cause another object to be allocated on the GC heap. The cumulative cost of these allocations (and the corresponding work to recover the memory once they are no longer in use) can be damaging in some very performance-sensitive applications. They can also be significant in cloud applications or high-volume data processing, where you might be paying for the amount of processing work you do—reducing CPU or memory usage can have a nontrivial effect on cost.

The `Span<T>` type and the related types discussed in this chapter make it possible to work with data wherever it already resides in memory. This typically requires rather more complex code, but in cases where the payoff justifies the work, these features make it possible for C# to tackle whole classes of problems for which it would previously have been too slow.

Thank you for reading this book, and congratulations for making it to the end. I hope you enjoy using C#, and I wish you every success with your future projects.

^([1](ch18.xhtml#idm45884782468272-marker)) .NET Core and .NET do not store the pointer and offset separately: instead, a span just points directly to the data of interest. The version of `Span<T>` available for .NET Framework needs to maintain the pointer separately to ensure GC handles spans correctly, because its CLR does not have the same modifications for supporting spans that .NET Core has.

^([2](ch18.xhtml#idm45884782266080-marker)) That said, it is possible to perform this kind of conversion explicitly—the `MemoryMarshal` class offers methods that can take a span of one type and return another span that provides a view over the same underlying memory, interpreted as containing a different element type. But it is unlikely to be useful in this case: converting a `ReadOnlySpan<char>` to a `ReadOnlySpan<int>` would produce a span with half the number of elements, where each `int` contained pairs of adjacent `char` values.

^([3](ch18.xhtml#idm45884782152672-marker)) It is a `ValueTask<ReadResult>` because the purpose of this exercise is to minimize allocations. `ValueTask<T>` was described in [Chapter 16](ch16.xhtml#ch_multithreading).