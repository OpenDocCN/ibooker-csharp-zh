# 附录 B. 识别和解释异步模式

异步代码的好处在 .NET 发明之前就已广为人知。在 .NET 早期，出现了几种不同的异步代码风格，被这里或那里使用，最终被废弃。这些并非全都是坏主意；其中许多为现代 `async`/`await` 方法铺平了道路。然而，现在有很多遗留代码使用了旧的异步模式。本附录将讨论更常见的模式，解释它们的工作原理以及如何与现代代码集成。

有时，同一类型多年来更新，支持多个异步模式。也许最好的例子是 `Socket` 类。以下是 `Socket` 类的核心 `Send` 操作的一些成员：

```cs
class Socket
{
  // Synchronous
  public int Send(byte[] buffer, int offset, int size, SocketFlags flags);

  // APM
  public IAsyncResult BeginSend(byte[] buffer, int offset, int size,
      SocketFlags flags, AsyncCallback callback, object state);
  public int EndSend(IAsyncResult result);

  // Custom, very close to APM
  public IAsyncResult BeginSend(byte[] buffer, int offset, int size,
      SocketFlags flags, out SocketError error,
      AsyncCallback callback, object state);
  public int EndSend(IAsyncResult result, out SocketError error);

  // Custom
  public bool SendAsync(SocketAsyncEventArgs e);

  // TAP (as an extension method)
  public Task<int> SendAsync(ArraySegment<byte> buffer,
      SocketFlags socketFlags);

  // TAP (as an extension method) using more efficient types
  public ValueTask<int> SendAsync(ReadOnlyMemory<byte> buffer,
      SocketFlags socketFlags, CancellationToken cancellationToken = default);
}
```

遗憾的是，由于大多数文档都是按字母顺序排列，并且有大量重载以试图简化使用，类型如 `Socket` 变得难以理解。希望本节的指南能有所帮助。

# 任务异步模式（TAP）

任务异步模式（TAP）是现代异步 API 模式，适用于 `await` 使用。每个异步操作由返回可等待对象的单个方法表示。"可等待对象" 是任何可以由 `await` 消耗的类型；通常是 `Task` 或 `Task<T>`，但也可能是 `ValueTask`、`ValueTask<T>`，一个框架定义的类型（例如，由通用 Windows 应用程序使用的 `IAsyncAction` 或 `IAsyncOperation<T>`），甚至是库定义的自定义类型。

TAP 方法通常以 `Async` 后缀命名。但这只是一种约定；并非所有 TAP 方法都带有 `Async` 后缀。如果 API 开发者认为异步上下文已充分暗示，可以省略此后缀；例如，`Task.WhenAll` 和 `Task.WhenAny` 就没有 `Async` 后缀。此外，请注意，*非* TAP 方法可能会带有 `Async` 后缀（例如，`WebClient.DownloadStringAsync` 不是 TAP 方法）。在这种情况下，通常 TAP 方法会带有 `TaskAsync` 后缀（例如，`WebClient.DownloadStringTaskAsync` 是 TAP 方法）。

返回异步流的方法也遵循类似于 TAP 的模式，使用 `Async` 作为后缀。即使它们不返回可等待对象，它们也会返回可等待流——可以使用 `await foreach` 消耗的类型。

可以通过以下特征识别任务异步模式（TAP）：

1.  操作由单个方法表示。

1.  方法返回可等待对象或可等待流。

1.  方法通常以 `Async` 结尾。

下面是一个具有 TAP API 的类型示例：

```cs
class ExampleHttpClient
{
  public Task<string> GetStringAsync(Uri requestUri);

  // Synchronous equivalent, for comparison
  public string GetString(Uri requestUri);
}
```

使用 `await` 可以实现任务型异步模式，并且本书的大部分内容都涵盖了这一点。如果你在没有理解如何使用 `await` 的情况下来到这个附录，那我不确定我能在这一点上帮助你，但你可以试着阅读第 1 和 2 章节，看看是否能唤起你的记忆。

# 异步编程模型（APM）

在 TAP 之后，异步编程模型（APM）模式可能是您会遇到的下一个最常见模式。这是第一个异步操作具有一级对象表示的模式。该模式的显著特征是与一对管理操作的方法一起使用的 `IAsyncResult` 对象，其中一个以 `Begin` 开头，另一个以 `End` 开头。

`IAsyncResult` 受 [本地重叠 I/O](http://bit.ly/sync-ipop) 强烈影响。APM 模式允许消费代码以同步或异步方式运行。消费代码可以从以下选项中选择：

+   阻塞操作完成。这通过调用 `End` 方法来完成。

+   在做其他事情的同时轮询操作是否完成。

+   提供一个回调委托，在操作完成时调用。

在所有情况下，消费代码必须最终调用 `End` 方法以检索异步操作的结果。如果在调用 `End` 时操作尚未完成，则会阻塞调用线程直到操作完成。

`Begin` 方法接受 `AsyncCallback` 参数和 `object` 参数（通常称为 `state`）作为其最后两个参数。这些参数由消费代码使用，以在操作完成时调用回调委托。`object` 参数可以是任何你想要的；这是在 .NET 的早期阶段之前使用的，甚至在 lambda 方法或匿名方法存在之前。它仅用于为 `AsyncCallback` 参数提供上下文。

APM 在微软库中相当普遍，但在更广泛的 .NET 生态系统中并不常见。这是因为从未有任何可重用的 `IAsyncResult` 实现，并且正确实现该接口相当复杂。此外，组合基于 APM 的系统也很困难。我只见过少数几个自定义的 `IAsyncResult` 实现；所有这些都是 Jeffrey Richter 发表在他的文章 “Concurrent Affairs: Implementing the CLR Asynchronous Programming Model” 中的通用 `IAsyncResult` 实现的某个版本，该文章发表在 [2007 年 3 月的 *MSDN Magazine*](http://bit.ly/conc-aff) 上。

可以通过以下特征识别异步编程模型模式：

1.  操作由一对方法表示，一个以 `Begin` 开头，另一个以 `End` 开头。

1.  `Begin` 方法返回一个 `IAsyncResult`，除了所有正常的输入参数外，还有额外的 `AsyncCallback` 参数和额外的 `object` 参数。

1.  `End`方法只接受一个`IAsyncResult`，并返回结果值（如果有）。

这是一个具有 APM API 的示例类型：

```cs
class MyHttpClient
{
  public IAsyncResult BeginGetString(Uri requestUri,
      AsyncCallback callback, object state);
  public string EndGetString(IAsyncResult asyncResult);

  // Synchronous equivalent, for comparison
  public string GetString(Uri requestUri);
}
```

通过将其转换为 TAP 来使用 APM，可以使用`Task.Factory.FromAsync`；参见 Recipe 8.2 和[Microsoft 文档](http://bit.ly/interop-async)。

有些情况下，代码几乎遵循了 APM 模式，但并非完全如此；例如，旧的`Microsoft.TeamFoundation`客户端库在其`Begin`方法中不包括`object`参数。在这些情况下，`Task.Factory.FromAsync`将不起作用，然后您可以选择两个选项。效率较低的选项是调用`Begin`方法并将`IAsyncResult`传递给`FromAsync`。不太优雅的选项是使用更灵活的`TaskCompletionSource<T>`；参见 Recipe 8.3。

# 基于事件的异步编程（EAP）

基于事件的异步编程（EAP）定义了一组匹配的方法/事件对。方法通常以`Async`结尾，并最终引发以`Completed`结尾的事件。

在处理 EAP 时有一些注意事项，使得其比最初看起来更加复杂。首先，必须记住在调用方法之前将处理程序添加到事件*之前*；否则，可能会出现竞争条件，事件可能在您订阅之前发生，然后您将永远看不到其完成。其次，按照 EAP 模式编写的组件通常在某个时刻捕获当前的`SynchronizationContext`，然后在该上下文中引发其事件。一些组件在构造函数中捕获`SynchronizationContext`，而其他组件则在调用方法并开始异步操作时捕获它。

基于事件的异步编程模式可以通过以下特征来识别：

1.  操作由事件和方法表示。

1.  事件以`Completed`结尾。

1.  `Completed`事件的事件参数类型可能是从`AsyncCompletedEventArgs`派生的。

1.  方法通常以`Async`结尾。

1.  方法返回`void`。

以`Async`结尾的 EAP 方法与以`Async`结尾的 TAP 方法有所区别，因为 EAP 方法返回`void`，而 TAP 方法返回可等待类型。

这是一个具有 EAP API 的示例类型：

```cs
class GetStringCompletedEventArgs : AsyncCompletedEventArgs
{
  public string Result { get; }
}

class MyHttpClient
{
  public void GetStringAsync(Uri requestUri);
  public event Action<object, GetStringCompletedEventArgs> GetStringCompleted;

  // Synchronous equivalent, for comparison
  public string GetString(Uri requestUri);
}
```

通过将其转换为 TAP 来消耗 EAP，可以使用`TaskCompletionSource<T>`；参见 Recipe 8.3 和[Microsoft 文档](http://bit.ly/EAP-MS)。

# 连续传递样式（CPS）

这是其他语言中更常见的一种模式，特别是 JavaScript 和 TypeScript，由 Node.js 开发人员使用。在这种模式中，每个异步操作都会接受一个回调委托，当操作完成时会调用该委托，无论是成功还是出错。此模式的变体使用 *两个* 回调委托，一个用于成功，另一个用于错误。这种类型的回调称为“continuation”，并且 continuation 作为参数传递，因此得名“continuation passing style”。这种模式在 .NET 世界中从未普及，但有几个较老的开源库使用了它。

通过以下特征可以识别 Continuation Passing Style 模式：

1.  操作由单个方法表示。

1.  该方法接受一个额外的参数，这是一个回调委托；回调委托接受两个参数，一个用于错误，另一个用于结果。

1.  或者，操作方法接受两个额外参数，都是回调委托；一个回调委托仅用于错误，另一个回调委托仅用于结果。

1.  回调委托通常命名为 `done` 或 `next`。

下面是一个具有 continuation-passing style API 的示例类型：

```cs
class MyHttpClient
{
  public void GetString(Uri requestUri, Action<Exception, string> done);

  // Synchronous equivalent, for comparison
  public string GetString(Uri requestUri);
}
```

通过使用 `TaskCompletionSource<T>` 将 CPS 转换为 TAP 来消耗，传递仅完成 `TaskCompletionSource<T>` 的回调委托；参见 Recipe 8.3。

# 自定义异步模式

非常专业化的类型有时会定义自己的自定义异步模式。其中最著名的例子是 `Socket` 类型，它定义了一个通过传递代表操作的 `SocketAsyncEventArgs` 实例的模式。引入此模式的原因是 `SocketAsyncEventArgs` 可以被重用，从而减少了对执行大量网络活动的应用程序的内存使用量。现代应用程序可以使用 [`ValueTask<T>` 和 `ManualResetValueTaskSourceCore<T>`](http://bit.ly/man-reset-type-doc) 来获得类似的性能增益。

自定义模式没有任何共同特征，因此最难识别。幸运的是，自定义异步模式并不常见。

下面是一个具有自定义异步 API 的示例类型：

```cs
class MyHttpClient
{
  public void GetString(Uri requestUri,
      MyHttpClientAsynchronousOperation operation);

  // Synchronous equivalent, for comparison
  public string GetString(Uri requestUri);
}
```

`TaskCompletionSource<T>` 是消耗自定义异步模式的唯一方式；参见 Recipe 8.3。

# ISynchronizeInvoke

所有之前的模式都是针对已启动的异步操作，并且一旦启动，它们就会完成。一些组件遵循订阅模型：它们代表基于推送的事件流，而不是一次启动并完成的单个操作。一个好的订阅模型示例是 `FileSystemWatcher` 类型。为了观察文件系统的变化，消费代码首先订阅多个事件，然后将 `EnableRaisingEvents` 属性设置为 `true`。一旦 `EnableRaisingEvents` 为 `true`，可能会引发多个文件系统变化事件。

一些组件为其事件使用`ISynchronizeInvoke`模式。它们公开一个`ISynchronizeInvoke`属性，消费者将该属性设置为允许组件调度工作的实现。这通常用于将工作安排到 UI 线程，以便在 UI 线程上引发组件的事件。按照惯例，如果`ISynchronizeInvoke`为`null`，则不进行事件同步，并且可能在后台线程上引发。

可以通过以下特征识别`ISynchronizeInvoke`模式：

1.  有一个`ISynchronizeInvoke`类型的属性。

1.  该属性通常称为`SynchronizingObject`。

这是使用`ISynchronizeInvoke`模式的一个示例类型：

```cs
class MyHttpClient
{
  public ISynchronizeInvoke SynchronizingObject { get; set; }
  public void StartListening();
  public event Action<string> StringArrived;
}
```

由于`ISynchronizeInvoke`暗示订阅模型中的多个事件，正确的消费这些组件的方法是将这些事件转换为可观察流，可以使用`FromEvent`（参见 Recipe 6.1）或`Observable.Create`。
