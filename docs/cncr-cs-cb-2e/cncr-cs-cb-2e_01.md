# 第一章：并发：概述

并发是优秀软件的关键特征。几十年来，并发虽然可行，但难以实现。并发软件难以编写、难以调试，也难以维护。因此，许多开发者选择了更简单的路径，并避免使用并发。有了现代.NET 程序可用的库和语言特性，现在并发要容易得多了。微软在显著降低并发门槛方面走在了前列。以前，并发编程是专家领域；而今，每个开发者都可以（也应该）拥抱并发。

# 并发介绍

继续之前，我想澄清一些我将在本书中使用的术语。这些是我自己的定义，我一贯使用它们来消除不同编程技术的歧义。让我们从*并发*开始。

并发

同时进行多项任务。

我希望并发的帮助显而易见。端用户应用程序使用并发来在向数据库写入数据的同时响应用户输入。服务器应用程序使用并发来在完成第一个请求的同时响应第二个请求。任何时候你需要应用程序在做一件事的同时正在处理另一件事时，都需要并发。世界上几乎每一个软件应用程序都可以从并发中受益。

大多数开发者一听到“并发”就立刻想到“多线程”。我想要区分这两者。

多线程

使用多个执行线程的并发形式。

多线程是指确实使用多个线程。正如本书中许多示例所示，多线程是*一种*并发形式，但绝非唯一形式。事实上，在现代应用程序中，直接使用低级别的线程类型几乎没有意义；高级抽象比传统的多线程更强大、更高效。因此，我将尽量减少对过时技术的覆盖。本书中的多线程示例均未使用`Thread`或`BackgroundWorker`类型；它们已被更优秀的替代方案取代。

###### 警告

一旦你键入`new Thread()`，你的项目就已经有了遗留代码。

但不要认为多线程已经过时！多线程仍然存在于*线程池*中，这是一个有用的工作队列，可以根据需求自动调整自己。线程池进一步实现了另一种重要的并发形式：*并行处理*。

并行处理

通过将工作分配给多个同时运行的线程来完成大量工作。

并行处理（或并行编程）利用多线程来最大化多个处理器核心的使用。现代 CPU 具有多个核心，如果有很多工作要做，那么让一个核心独自完成所有工作而其他核心空闲是没有意义的。并行处理将工作分配给多个线程，每个线程可以独立地在不同的核心上运行。

并行处理是多线程的一种类型，而多线程是并发的一种类型。在现代应用程序中，还有另一种重要的并发类型，但对许多开发者来说并不那么熟悉：*异步编程*。

异步编程

一种使用未来或回调来避免不必要线程的并发形式。

*未来*（或*承诺*）是一种表示将来某些操作完成的类型。在.NET 中，一些现代的未来类型包括`Task`和`Task<TResult>`。旧的异步 API 使用回调或事件来代替未来。异步编程围绕*异步操作*的概念展开：启动的操作将在稍后某个时刻完成。在操作进行时，它不会阻塞原始线程；启动操作的线程可以自由进行其他工作。当操作完成时，它通过通知其未来或调用其回调或事件来告知应用程序操作已完成。

异步编程是一种强大的并发形式，但直到最近，它需要极其复杂的代码。现代语言中的`async`和`await`支持使异步编程几乎和同步（非并发）编程一样简单。

另一种并发形式是*响应式编程*。异步编程意味着应用程序将启动一个稍后会完成的操作。响应式编程与异步编程密切相关，但是建立在*异步事件*而不是*异步操作*的基础上。异步事件可能没有实际的“启动”，可能随时发生，并且可能被多次触发。例如用户输入是一个例子。

响应式编程

一种声明式编程风格，应用程序对事件做出响应。

如果将一个应用程序视为一个大型状态机，该应用程序的行为可以描述为在每个事件中通过更新其状态来响应一系列事件。这并不像听起来那么抽象或理论化；现代框架使这种方法在实际应用中非常有用。响应式编程并不一定是并发的，但它与并发密切相关，因此本书涵盖了基础知识。

通常，在编写并发程序时会混合使用各种技术。大多数应用程序至少使用多线程（通过线程池）和异步编程。可以根据应用程序的不同部分混合和匹配所有各种形式的并发，选择适当的工具。

# 异步编程介绍

异步编程有两个主要好处。第一个好处是对于终端用户 GUI 程序：异步编程能够提升响应性。每个人都使用过一个在工作时暂时锁定的程序；异步程序可以在工作时保持对用户输入的响应。第二个好处是对于服务器端程序：异步编程能够提升可伸缩性。服务器应用程序可以通过使用线程池来实现一定程度的扩展，但异步服务器应用程序通常可以比这更好地扩展一个数量级。

异步编程的两个好处都源自同一个基本方面：异步编程释放了一个线程。对于 GUI 程序，异步编程释放了 UI 线程；这使得 GUI 应用程序可以保持对用户输入的响应。对于服务器应用程序，异步编程释放了请求线程；这使得服务器可以使用其线程来处理更多的请求。

现代异步.NET 应用程序使用两个关键字：`async`和`await`。`async`关键字添加到方法声明中，起到双重作用：它在该方法内启用`await`关键字，并提示编译器为该方法生成状态机，类似于`yield return`的工作方式。如果异步方法返回值，它可以返回`Task<TResult>`，如果不返回值，则返回`Task`或任何其他“类似任务”的类型，如`ValueTask`。此外，如果异步方法返回枚举中的多个值，则可以返回`IAsyncEnumerable<T>`或`IAsyncEnumerator<T>`。类似任务的类型代表未来；它们可以在异步方法完成时通知调用代码。

###### 警告

避免使用`async void`！可能有一个`async`方法返回`void`，但只有在编写`async`事件处理程序时才应该这样做。没有返回值的常规`async`方法应该返回`Task`，而不是`void`。

在此背景下，让我们快速看一个例子：

```cs
async Task DoSomethingAsync()
{
  int value = 13;

  // Asynchronously wait 1 second.
  await Task.Delay(TimeSpan.FromSeconds(1));

  value *= 2;

  // Asynchronously wait 1 second.
  await Task.Delay(TimeSpan.FromSeconds(1));

  Trace.WriteLine(value);
}
```

`async`方法开始同步执行，就像任何其他方法一样。在`async`方法内部，`await`关键字对其参数执行*异步等待*。首先，它检查操作是否已经完成；如果完成，它将继续执行（同步）。否则，它将暂停`async`方法并返回一个未完成的任务。当操作稍后完成时，`async`方法将继续执行。

你可以把`async`方法看作有几个同步部分，这些部分由`await`语句分隔开。第一个同步部分在调用方法的任何线程上执行，但其他同步部分在哪里执行呢？答案有点复杂。

当您`await`一个任务（最常见的场景），当`await`决定暂停方法时，将捕获一个*上下文*。这是当前的`SynchronizationContext`，除非它为`null`，在这种情况下，上下文是当前的`TaskScheduler`。方法在捕获的上下文中恢复执行。通常，此上下文是 UI 上下文（如果您在 UI 线程上）或线程池上下文（大多数其他情况）。如果您有一个 ASP.NET 经典（非 Core）应用程序，则上下文也可以是 ASP.NET 请求上下文。ASP.NET Core 使用线程池上下文而不是特殊的请求上下文。

因此，在前面的代码中，所有同步部分都将尝试在原始上下文中恢复。如果你从 UI 线程调用`DoSomethingAsync`，那么它的每个同步部分都将在该 UI 线程上运行；但如果你从线程池线程调用它，那么它的每个同步部分将在任何线程池线程上运行。

您可以通过等待`ConfigureAwait`扩展方法的结果并将`continueOnCapturedContext`参数设置为`false`来避免此默认行为。以下代码将在调用线程上启动，并在被`await`暂停后在线程池线程上恢复：

```cs
async Task DoSomethingAsync()
{
  int value = 13;

  // Asynchronously wait 1 second.
  await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);

  value *= 2;

  // Asynchronously wait 1 second.
  await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);

  Trace.WriteLine(value);
}
```

###### 提示

在核心“库”方法中始终调用`ConfigureAwait`并仅在外部“用户界面”方法中恢复上下文是一个好习惯。

`await`关键字不仅限于与任务一起工作；它可以与任何符合特定模式的*awaitable*一起工作。例如，基类库包括`ValueTask<T>`类型，如果结果通常是同步的，则减少内存分配；例如，如果结果可以从内存中的缓存中读取。`ValueTask<T>`不直接转换为`Task<T>`，但它确实遵循可等待模式，因此您可以直接`await`它。还有其他示例，您也可以构建自己的示例，但大多数情况下，`await`将接受`Task`或`Task<TResult>`。

有两种基本方法可以创建`Task`实例。某些任务表示 CPU 必须执行的实际代码；这些计算任务应通过调用`Task.Run`（或者如果需要它们在特定调度程序上运行，则通过`TaskFactory.StartNew`）创建。其他任务表示*通知*；这些基于事件的任务类型是通过`TaskCompletionSource<TResult>`（或其快捷方式之一）创建的。大多数 I/O 任务使用`TaskCompletionSource<TResult>`。

使用`async`和`await`进行错误处理是自然的。在以下代码片段中，`PossibleExceptionAsync`可能会抛出`NotSupportedException`，但`TrySomethingAsync`可以自然地捕获异常。捕获的异常保留了其堆栈跟踪，并且没有被人为包装在`TargetInvocationException`或`AggregateException`中。

```cs
async Task TrySomethingAsync()
{
  try
  {
    await PossibleExceptionAsync();
  }
  catch (NotSupportedException ex)
  {
    LogException(ex);
    throw;
  }
}
```

当`async`方法抛出（或传播）异常时，异常会放置在其返回的`Task`上，并且`Task`已完成。当等待该`Task`时，`await`操作符将检索该异常并（重新）抛出它，以保留其原始堆栈跟踪。因此，如果`PossibleExceptionAsync`是一个`async`方法，则下面的示例代码会按预期工作：

```cs
async Task TrySomethingAsync()
{
  // The exception will end up on the Task, not thrown directly.
  Task task = PossibleExceptionAsync();

  try
  {
    // The Task's exception will be raised here, at the await.
    await task;
  }
  catch (NotSupportedException ex)
  {
    LogException(ex);
    throw;
  }
}
```

在使用`async`方法时还有一个重要的指导原则：一旦开始使用`async`，最好让它在整个代码中延续下去。如果调用了一个`async`方法，应该（最终）`await`其返回的任务。抵制调用`Task.Wait`、`Task<TResult>.Result`或`GetAwaiter().GetResult()`的诱惑；这样做可能会导致死锁。考虑以下方法：

```cs
async Task WaitAsync()
{
  // This await will capture the current context ...
  await Task.Delay(TimeSpan.FromSeconds(1));
  // ... and will attempt to resume the method here in that context.
}

void Deadlock()
{
  // Start the delay.
  Task task = WaitAsync();

  // Synchronously block, waiting for the async method to complete.
  task.Wait();
}
```

这个示例中的代码在从 UI 或 ASP.NET 经典环境中调用时会造成死锁，因为这些环境只允许一个线程进入。`Deadlock`将调用`WaitAsync`，开始延迟。然后`Deadlock`（同步地）等待该方法完成，阻塞了上下文线程。当延迟完成时，`await`试图在捕获的上下文中恢复`WaitAsync`，但由于上下文中已经有一个线程被阻塞，并且上下文只允许一个线程，所以无法执行。可以通过两种方式防止死锁：在`WaitAsync`中使用`ConfigureAwait(false)`（使`await`忽略其上下文），或者`await`调用`WaitAsync`（使`Deadlock`变成异步方法）。

###### 警告

如果使用`async`，最好全程使用`async`。

要了解更全面的`async`介绍，Microsoft 提供的在线文档非常棒；我建议至少阅读[异步编程概述](http://bit.ly/async-prog)和[基于任务的异步模式（TAP）](http://bit.ly/task-async-patt)概述。如果想深入了解，还有[深入异步](http://bit.ly/async-indepth)文档。

异步流利用了`async`和`await`的基础，并将其扩展到处理多个值。异步流围绕异步可枚举的概念构建，类似于常规可枚举，但允许在检索序列的下一个项目时进行异步工作。这是一个非常强大的概念，第三章详细介绍了此内容。异步流在处理单个或分块到达的数据序列时特别有用。例如，如果您的应用程序处理使用`limit`和`offset`参数进行分页的 API 响应，则异步流是一个理想的抽象。截至撰写本文时，异步流仅在最新的.NET 平台上可用。

# 并行编程简介

并行编程应该在任何你有相当数量的可以分成独立块的计算工作时使用。并行编程会暂时增加 CPU 使用率以提高吞吐量；这在客户端系统中是可取的，因为 CPU 通常是空闲的，但通常不适用于服务器系统。大多数服务器都内置了一些并行性；例如，ASP.NET 将并行处理多个请求。在服务器上编写并行代码可能在某些情况下仍然有用（如果你*知道*并发用户数始终很低），但一般情况下，在服务器上进行并行编程会与其内置的并行性相抵触，因此不会带来实际好处。

并行性有两种形式：*数据并行性*和*任务并行性*。数据并行性是指你有一堆数据项要处理，每个数据项的处理大多数是独立的。任务并行性是指你有一组工作要做，每个工作大多数也是独立的。任务并行性可能是动态的；如果一个工作导致产生几个额外的工作，则它们可以添加到工作池中。

有几种不同的数据并行方式。`Parallel.ForEach`类似于`foreach`循环，应在可能的情况下使用。`Parallel.ForEach`在 Recipe 4.1 中有所介绍。`Parallel`类还支持`Parallel.For`，类似于`for`循环，如果数据处理依赖于索引，则可以使用。使用`Parallel.ForEach`的代码如下：

```cs
void RotateMatrices(IEnumerable<Matrix> matrices, float degrees)
{
  Parallel.ForEach(matrices, matrix => matrix.Rotate(degrees));
}
```

另一种选择是 PLINQ（Parallel LINQ），它为 LINQ 查询提供了`AsParallel`扩展方法。`Parallel`比 PLINQ 更节约资源；`Parallel`在系统中更加友好，而 PLINQ（默认情况下）会尝试在所有 CPU 上进行分布。`Parallel`的缺点是它更加显式；在许多情况下，PLINQ 代码更加优雅。PLINQ 在 Recipe 4.5 中有所介绍，代码如下：

```cs
IEnumerable<bool> PrimalityTest(IEnumerable<int> values)
{
  return values.AsParallel().Select(value => IsPrime(value));
}
```

无论你选择的方法是什么，在进行并行处理时有一个指导原则非常重要。

###### 提示

工作块应尽可能彼此独立。

只要你的工作块与其他所有工作块都是独立的，就能最大化并行处理能力。一旦开始在多个线程之间共享状态，就必须同步对共享状态的访问，你的应用程序就会变得不那么并行。第十二章详细介绍了同步的内容。

并行处理的输出可以通过多种方式处理。你可以将结果放入某种并发集合中，或者将结果聚合到摘要中。在并行处理中，聚合很常见；这种 map/reduce 功能也被 `Parallel` 类的方法重载支持。Recipe 4.2 更详细地讨论了聚合。

现在让我们转向任务并行性。数据并行性专注于处理数据；任务并行性仅仅是关于完成工作。从高层次来看，数据并行性和任务并行性类似；“处理数据”就是一种“工作”。许多并行问题可以用两种方式解决；使用对于手头问题更自然的 API 是很方便的。

`Parallel.Invoke` 是 `Parallel` 方法的一种，它执行一种 fork/join 任务并行性。这个方法在 Recipe 4.3 中有所涵盖；你只需传入你想并行执行的委托：

```cs
void ProcessArray(double[] array)
{
  Parallel.Invoke(
      () => ProcessPartialArray(array, 0, array.Length / 2),
      () => ProcessPartialArray(array, array.Length / 2, array.Length)
  );
}

void ProcessPartialArray(double[] array, int begin, int end)
{
  // CPU-intensive processing...
}
```

`Task` 类型最初是为了任务并行性而引入的，尽管如今它也用于异步编程。`Task` 实例——作为任务并行性的一部分——代表了一些工作。你可以使用 `Wait` 方法等待任务完成，也可以使用 `Result` 和 `Exception` 属性获取工作的结果。直接使用 `Task` 的代码比使用 `Parallel` 更复杂，但如果在运行时不知道并行结构，它可能很有用。在这种动态并行性中，你不知道开始处理时需要做多少工作；随着进行，你才会找出。通常，动态工作应该启动需要的子任务，然后等待它们完成。`Task` 类型有一个特殊标志，`TaskCreationOptions.AttachedToParent`，可以用于此目的。动态并行性在 Recipe 4.4 中有所涵盖。

任务并行性应该力求独立，就像数据并行性一样。你的委托越独立，程序就越高效。此外，如果你的委托不独立，那么它们就需要同步，编写正确的代码就会更加困难。在任务并行性中，尤其要注意闭包中捕获的变量。记住，闭包捕获的是引用（而不是值），因此可能会出现不明显的共享问题。

所有类型的并行性都有类似的错误处理。因为操作是并行进行的，可能会发生多个异常，因此它们会被包装在抛给你的 `AggregateException` 中。这种行为在 `Parallel.ForEach`、`Parallel.Invoke`、`Task.Wait` 等中是一致的。`AggregateException` 类型有一些有用的 `Flatten` 和 `Handle` 方法来简化错误处理代码：

```cs
try
{
  Parallel.Invoke(() => { throw new Exception(); },
      () => { throw new Exception(); });
}
catch (AggregateException ex)
{
  ex.Handle(exception =>
  {
    Trace.WriteLine(exception);
    return true; // "handled"
  });
}
```

通常，你不必担心线程池如何处理工作。数据和任务并行使用动态调整的分区器来在工作线程之间分割工作。线程池会根据需要增加其线程数。线程池有一个单一的工作队列，每个线程池线程也有自己的工作队列。当线程池线程将额外的工作排队时，它首先发送到自己的队列，因为这个工作通常与当前工作项相关；这种行为鼓励线程处理自己的工作，并最大化缓存命中。如果另一个线程没有工作要做，它会从另一个线程的队列中窃取工作。微软在使线程池尽可能高效方面投入了大量工作，并且如果需要最大性能，你可以调整大量参数。只要你的任务不是过于短小，它们应该能够在默认设置下正常工作。

###### 提示

任务既不应该过于短小，也不应该过于长。

如果你的任务过于短小，那么将数据分解为任务，并在线程池上调度这些任务的开销会变得很大。如果任务过于长，则线程池无法有效地动态调整其工作负载平衡。很难确定什么长度算是太短，什么长度算是太长；这确实取决于所解决的问题以及硬件的大致能力。作为一般规则，我尽量让我的任务尽可能短，而不会遇到性能问题（当任务过于短时，性能会突然下降）。更好的方法是，不直接使用任务，而是使用`Parallel`类型或者 PLINQ。这些更高级别的并行形式已经内置了分区处理，可以自动处理（并在运行时根据需要调整）。

如果你想深入了解并行编程，关于这个主题最好的书是《使用 Microsoft .NET 进行并行编程》，作者是 Colin Campbell 等人（Microsoft Press）。

# 引入响应式编程（Rx）

响应式编程比其他形式的并发编程有更高的学习曲线，如果不跟上响应式技能的话，代码维护起来可能会更困难。但如果你愿意学习，响应式编程是非常强大的。响应式编程让你能够将事件流视为数据流。作为经验法则，如果你使用了传递给事件的任何事件参数，那么你的代码将受益于使用 System.Reactive 而不是常规事件处理程序。

###### 提示

System.Reactive 曾被称为 Reactive Extensions，通常简称为“Rx”。这三个术语都指的是同一项技术。

响应式编程基于可观察流的概念。当您订阅一个可观察流时，您将接收任意数量的数据项 (`OnNext`)，然后流可能以单个错误 (`OnError`) 或 “流结束” 通知 (`OnCompleted`) 结束。某些可观察流永远不会结束。实际的接口如下所示：

```cs
interface IObserver<in T>
{
  void OnNext(T item);
  void OnCompleted();
  void OnError(Exception error);
}

interface IObservable<out T>
{
  IDisposable Subscribe(IObserver<TResult> observer);
}
```

然而，您永远不应该实现这些接口。Microsoft 的 System.Reactive (Rx) 库已经包含了您可能需要的所有实现。响应式代码看起来非常类似于 LINQ；您可以将其视为 “LINQ to Events”。System.Reactive 拥有与 LINQ 相同的所有功能，并添加了大量自己的操作符，特别是处理时间的操作符。以下代码从一些不熟悉的操作符 (`Interval` 和 `Timestamp`) 开始，以 `Subscribe` 结束，但中间有一些您应该从 LINQ 中熟悉的 `Where` 和 `Select` 操作符：

```cs
Observable.Interval(TimeSpan.FromSeconds(1))
    .Timestamp()
    .Where(x => x.Value % 2 == 0)
    .Select(x => x.Timestamp)
    .Subscribe(x => Trace.WriteLine(x));
```

示例代码从一个周期性定时器 (`Interval`) 开始运行一个计数器，并为每个事件添加时间戳 (`Timestamp`)。然后，它会过滤事件，只包括偶数计数器值 (`Where`)，选择时间戳值 (`Timestamp`)，最后每个结果的时间戳值到达时，将其写入调试器 (`Subscribe`)。如果您不理解 `Interval` 等新操作符，不要担心：这些稍后在本书中会进行介绍。现在只需记住，这是一个与您已经熟悉的 LINQ 查询非常相似的 LINQ 查询。主要区别在于 LINQ to Objects 和 LINQ to Entities 使用 *“拉取”模型*，即 LINQ 查询的枚举通过查询拉取数据，而 LINQ to Events (System.Reactive) 使用 *“推送”模型*，即事件到达并自行通过查询传递。

可观察流的定义与其订阅是独立的。最后的例子与以下代码相同：

```cs
IObservable<DateTimeOffset> timestamps =
    Observable.Interval(TimeSpan.FromSeconds(1))
        .Timestamp()
        .Where(x => x.Value % 2 == 0)
        .Select(x => x.Timestamp);
timestamps.Subscribe(x => Trace.WriteLine(x));
```

类型定义可观察流并将其作为 `IObservable<TResult>` 资源提供是正常的。其他类型可以订阅这些流或与其他操作符结合以创建另一个可观察流。

`System.Reactive` 的订阅也是一种资源。`Subscribe` 操作符返回一个 `IDisposable`，表示订阅。当您的代码完成对可观察流的监听后，应该释放其订阅。

使用热和冷可观察流时，订阅的行为是不同的。*热可观察流* 是一个始终运行的事件流，如果没有订阅者在事件到达时，它们将丢失。例如，鼠标移动是一个热可观察流。*冷可观察流* 是不会始终有传入事件的可观察流。冷可观察流将通过订阅启动事件序列。例如，HTTP 下载是一个冷可观察流；订阅导致发送 HTTP 请求。

`Subscribe` 操作符应始终带有错误处理参数。前面的示例没有这样做；以下是一个更好的示例，如果可观察流以错误结束，将适当地做出响应：

```cs
Observable.Interval(TimeSpan.FromSeconds(1))
    .Timestamp()
    .Where(x => x.Value % 2 == 0)
    .Select(x => x.Timestamp)
    .Subscribe(x => Trace.WriteLine(x),
        ex => Trace.WriteLine(ex));
```

`Subject<TResult>` 是在使用 System.Reactive 进行实验时非常有用的一种类型。这个“主题”类似于手动实现的可观察流。你的代码可以调用 `OnNext`、`OnError` 和 `OnCompleted`，主题将把这些调用转发给它的订阅者。在实际生产代码中，`Subject<TResult>` 很适合进行实验，但你应该努力使用像在 Chapter 6 中涵盖的操作符。

System.Reactive 拥有大量有用的操作符，在本书中我只涵盖了一些精选的。关于 System.Reactive 的更多信息，我推荐阅读优秀的在线书籍 [*Introduction to Rx*](http://www.introtorx.com/)。

# 数据流介绍

TPL Dataflow 是异步和并行技术的有趣结合。当你有一系列需要应用到数据的过程时，它非常有用。例如，你可能需要从 URL 下载数据，解析数据，然后与其他数据并行处理。TPL Dataflow 常用作简单的管道，其中数据从一端进入，直到另一端出来。然而，TPL Dataflow 的能力远不止如此；它能处理任何类型的网格。你可以在网格中定义分支、汇合和循环，TPL Dataflow 会适当地处理它们。尽管如此，大多数时候，TPL Dataflow 网格被用作管道。

数据流网格的基本构建单元是 *数据流块*。一个块可以是目标块（接收数据）、源块（产生数据），或者两者兼而有之。源块可以与目标块链接以创建网格；链接的详细信息请参见 Recipe 5.1。块是半独立的；它们会尝试处理到达的数据并将结果推送到下游。使用 TPL Dataflow 的常规方式是创建所有块，将它们链接在一起，然后从一端开始输入数据。数据会自动从另一端输出。同样，Dataflow 的能力远超出此范围；在数据流动的同时，可以断开链接、创建新块并将其添加到网格中，但这是一种非常高级的场景。

目标块具有用于接收它们接收的数据的缓冲区。通过具有缓冲区，即使它们还没有准备好处理数据项，它们也能够接受新的数据项；这保持了数据通过网格的流动。这种缓冲可能会在分叉场景中引发问题，其中一个源块链接到两个目标块。当源块有数据要发送到下游时，它开始逐个向其链接的块提供数据。默认情况下，第一个目标块只会接受数据并缓冲它，而第二个目标块则永远不会得到任何数据。解决此情况的方法是通过使目标块成为非贪婪的来限制目标块的缓冲区；菜谱 5.4 详细介绍了这一点。

当某个块出现问题时，例如在处理数据项时处理委托引发异常时，该块将出现故障。当块出现故障时，它将停止接收数据。默认情况下，它不会导致整个网格崩溃；这使您能够重建网格的那一部分或重定向数据。但是，这是一个高级场景；大多数情况下，您希望故障沿着链路传播到目标块。数据流也支持此选项；唯一棘手的部分是当异常沿链路传播时，它会被包装在`AggregateException`中。因此，如果您有一个长的管道，可能会出现深度嵌套的异常；方法`AggregateException.Flatten`可用于解决此问题：

```cs
try
{
  var multiplyBlock = new TransformBlock<int, int>(item =>
  {
    if (item == 1)
      throw new InvalidOperationException("Blech.");
    return item * 2;
  });
  var subtractBlock = new TransformBlock<int, int>(item => item - 2);
  multiplyBlock.LinkTo(subtractBlock,
      new DataflowLinkOptions { PropagateCompletion = true });

  multiplyBlock.Post(1);
  subtractBlock.Completion.Wait();
}
catch (AggregateException exception)
{
  AggregateException ex = exception.Flatten();
  Trace.WriteLine(ex.InnerException);
}
```

菜谱 5.2 详细介绍了数据流错误处理。

乍一看，数据流网格听起来很像可观察流，它们确实有很多共同点。网格和流都有数据项通过的概念。而且，网格和流都有正常完成（通知没有更多数据到来）和故障完成（通知在数据处理过程中发生错误）的概念。但是，System.Reactive（Rx）和 TPL Dataflow 并没有相同的能力。在处理与时间相关的任何事务时，Rx 可观察流通常比数据流块更好。而在进行并行处理时，数据流块通常比 Rx 可观察流更好。从概念上讲，Rx 更像是设置回调：可观察流中的每个步骤直接调用下一个步骤。相比之下，数据流网格中的每个块都与所有其他块非常独立。Rx 和 TPL Dataflow 都有各自的用途，并存在一定的重叠。它们在一起工作也非常好；菜谱 8.8 详细介绍了 Rx 和 TPL Dataflow 之间的互操作性。

如果您熟悉 Actor 框架，TPL Dataflow 将似乎与其共享一些相似之处。每个数据流块是独立的，它会根据需要启动任务来执行工作，如执行转换委托或将输出推送到下一个块。您还可以设置每个块并行运行，以便它可以启动多个任务来处理额外的输入。由于这种行为，每个块确实与 Actor 框架中的 Actor 具有某种相似性。然而，TPL Dataflow 并不是一个完整的 Actor 框架；特别是，它没有内置支持干净的错误恢复或任何形式的重试。TPL Dataflow 是一个具有类似 Actor 感觉的库，但它不是一个功能完备的 Actor 框架。

最常见的 TPL Dataflow 块类型包括 `TransformBlock<TInput, TOutput>`（类似于 LINQ 的 `Select`）、`TransformManyBlock<TInput, TOutput>`（类似于 LINQ 的 `SelectMany`）和 `ActionBlock<TResult>`，它为每个数据项执行一个委托。有关 TPL Dataflow 的更多信息，请参阅[MSDN 文档](http://bit.ly/dataflow-doc)和[“实现自定义 TPL Dataflow 块指南”](http://bit.ly/tpl-dataflow)。

# 多线程编程简介

*线程* 是独立的执行器。每个进程中都有多个线程，在其中每个线程可以同时执行不同的任务。每个线程有其自己独立的堆栈，但与进程中的所有其他线程共享相同的内存。在某些应用程序中，有一个特殊的线程。例如，用户界面应用程序有一个特殊的 UI 线程，控制台应用程序有一个特殊的主线程。

每个.NET 应用程序都有一个线程池。线程池维护着一些工作线程，这些线程等待执行您给它们的工作。线程池负责确定线程池中任何时候有多少个线程。有许多配置设置可以调整这种行为，但我建议您不要去碰它；线程池已经经过精心调整，可以覆盖绝大多数实际场景。

几乎没有必要自己创建新线程。您唯一需要创建 `Thread` 实例的时候是如果您需要一个 STA 线程来进行 COM 互操作。

线程是低级抽象。线程池是稍高级的抽象；当代码将工作排入线程池时，线程池本身会负责根据需要创建线程。本书涵盖的抽象级别更高：并行和数据流处理根据需要将工作排入线程池。使用这些更高级别抽象的代码比使用低级抽象更容易正确实现。

因此，`Thread` 和 `BackgroundWorker` 类型在本书中完全没有涵盖。它们有过自己的时代，而那个时代已经结束了。

# 并发应用程序的集合

有几个集合类别对并发编程很有用：并发集合和不可变集合。这两个集合类别都在第九章中有所涵盖。并发集合允许多个线程同时安全地更新它们。大多数并发集合使用*快照*来使一个线程能够枚举值，而另一个线程可能在添加或删除值。并发集合通常比只用锁保护常规集合更有效率。

不可变集合有些不同。不可变集合实际上不能被修改；相反，要修改一个不可变集合，你需要创建一个代表修改后集合的新集合。这听起来效率非常低，但不可变集合在集合实例之间尽可能共享内存，所以情况没有听起来那么糟。不可变集合的好处在于所有操作都是纯粹的，因此它们与函数式代码非常配合。

# 现代设计

大多数并发技术具有一个相似的特点：它们都具有功能性质。我不是指“功能性”指的是它们能完成工作，而是指一种基于函数组合的编程风格。如果你采用功能性思维方式，你的并发设计将更加简洁。

函数式编程的一个原则是*purity*（即避免副作用）。解决方案的每一部分都将某些值作为输入并产生某些值作为输出。尽可能避免这些部分依赖全局（或共享）变量或更新全局（或共享）数据结构。无论这部分是一个`async`方法、一个并行任务、一个 System.Reactive 操作还是一个数据流块，这一点都是真实的。当然，迟早你的计算将不得不产生效果，但如果你能使用纯净的部分处理*processing*，然后使用*results*执行更新，你会发现你的代码更加清洁。

函数式编程的另一个原则是*immutability*。不可变性意味着数据片段不能改变。对于并发程序，不可变数据的一个有用原因是你永远不需要为不可变数据进行同步；它的不变性使得同步变得不必要。不可变数据还有助于避免副作用。开发者开始更多地使用不可变类型，本书包含了几个处理不可变数据结构的技巧。

# 关键技术摘要

.NET 框架自始至终对异步编程有一些支持。然而，直到 2012 年，也就是.NET 4.5（连同 C# 5.0 和 VB 2012）引入了`async`和`await`关键字之前，异步编程一直很困难。本书将使用现代的`async`/`await`方法来处理所有异步任务，并提供了一些示例，展示如何在`async`和旧的异步编程模式之间进行交互。如果需要支持旧平台，请参阅附录 A。

.NET 4.0 引入了任务并行库（Task Parallel Library，TPL），全面支持数据并行和任务并行。如今，即使在资源较少的平台如手机上，也可以使用。TPL 已经内置于.NET 中。

System.Reactive 团队努力支持尽可能多的平台。像`async`和`await`一样，System.Reactive 为各种应用程序（包括客户端和服务器）提供了诸多好处。System.Reactive 可以在[`System.Reactive`](http://bit.ly/sys-reactive) NuGet 包中找到。

TPL Dataflow 库正式分发在[`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) NuGet 包中。

大多数并发集合都内置于.NET 中；[`System.Threading.Channels`](https://www.nuget.org/packages/System.Threading.Channels) NuGet 包中还提供了一些额外的并发集合。不可变集合则可以在[`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet 包中找到。
