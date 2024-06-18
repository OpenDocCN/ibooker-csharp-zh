# 第八章：互操作性

异步、并行、响应式——每种方法都有其适用的场合，但它们如何一起工作呢？

在本章中，我们将探讨各种*互操作*场景，在这些场景中，您将学习如何结合这些不同的方法。您将了解到它们是互补的，而不是竞争的；在一个方法遇到另一个方法的边界处，几乎没有摩擦。

# 8.1 **异步包装器用于带有“Completed”事件的“Async”方法

## 问题

存在一种较旧的异步模式，使用名为``*`Operation`*Async``的方法以及名为``*`Operation`*Completed``的事件。您希望使用旧的异步模式执行操作并等待结果。

###### 提示

``*`Operation`*Async``和``*`Operation`*Completed``模式称为事件驱动的异步模式（EAP）。您将把它们封装成遵循任务异步模式（TAP）的返回`Task`方法。

## 解决方案

通过使用`TaskCompletionSource<TResult>`类型，您可以创建异步操作的包装器。`TaskCompletionSource<TResult>`类型控制`Task<TResult>`，并使您能够在适当的时候完成任务。

此示例定义了一个用于下载`string`的`WebClient`的扩展方法。`WebClient`类型定义了`DownloadStringAsync`和`DownloadStringCompleted`。使用这些，您可以定义一个`DownloadStringTaskAsync`方法，如下所示：

```cs
public static Task<string> DownloadStringTaskAsync(this WebClient client,
    Uri address)
{
  var tcs = new TaskCompletionSource<string>();

  // The event handler will complete the task and unregister itself.
  DownloadStringCompletedEventHandler handler = null;
  handler = (_, e) =>
  {
    client.DownloadStringCompleted -= handler;
    if (e.Cancelled)
      tcs.TrySetCanceled();
    else if (e.Error != null)
      tcs.TrySetException(e.Error);
    else
      tcs.TrySetResult(e.Result);
  };

  // Register for the event and *then* start the operation.
  client.DownloadStringCompleted += handler;
  client.DownloadStringAsync(address);

  return tcs.Task;
}
```

## 讨论

这个特定的例子并不是很有用，因为`WebClient`已经定义了`DownloadStringTaskAsync`，而且有一个更加支持`async`的`HttpClient`可以使用。然而，这种技术同样适用于接口未更新为使用`Task`的旧异步代码。

###### 提示

对于新代码，始终使用`HttpClient`。仅在使用旧代码时使用`WebClient`。

通常，用于下载字符串的 TAP 方法将命名为``*`Operation`*Async``（例如，`DownloadStringAsync`）；但是，在这种情况下，该命名约定不起作用，因为 EAP 已经定义了具有该名称的方法。在这里，约定是将 TAP 方法命名为``*`Operation`*TaskAsync``（例如，`DownloadStringTaskAsync`）。

在包装 EAP 方法时，存在“启动”方法可能会抛出异常的可能性；在前面的示例中，`DownloadStringAsync`可能会抛出异常。在这种情况下，您需要决定是允许异常传播还是捕获异常并调用`TrySetException`。大多数情况下，这些点抛出的异常是使用错误，所以无论选择哪种选项都没关系。如果不确定异常是否是使用错误，那么建议捕获异常并调用`TrySetException`。

## 参见

Recipe 8.2 介绍了对 APM 方法（``Begin*`Operation`*``和``End*`Operation`*``）的 TAP 包装器。

Recipe 8.3 介绍了任何类型通知的 TAP 包装器。

# 8.2 **异步包装器用于“Begin/End”方法

## 问题

旧的异步模式使用一对名为``Begin*`Operation`*``和``End*`Operation`*``的方法，`IAsyncResult`表示异步操作。您有一个遵循旧的异步模式的操作，并希望使用`await`消费它。

###### 提示

``Begin*`Operation`*``和``End*`Operation`*``模式称为异步编程模型（APM）。您将把它们包装成遵循基于任务的异步模式（TAP）的返回`Task`的方法。

## 解决方案

包装 APM 的最佳方法是使用`TaskFactory`类型上的`FromAsync`方法之一。`FromAsync`在内部使用`TaskCompletionSource<TResult>`，但在包装 APM 时，使用`FromAsync`要简单得多。

此示例定义了一个为`WebRequest`定义扩展方法的例子，该方法发送 HTTP 请求并获取响应。`WebRequest`类型定义了`BeginGetResponse`和`EndGetResponse`；您可以像这样定义一个`GetResponseAsync`方法：

```cs
public static Task<WebResponse> GetResponseAsync(this WebRequest client)
{
  return Task<WebResponse>.Factory.FromAsync(client.BeginGetResponse,
      client.EndGetResponse, null);
}
```

## 讨论

`FromAsync`有令人困惑的多种重载！

通常最好像示例中那样调用`FromAsync`。首先，传递``Begin*`Operation`*``方法（不调用它），然后传递``End*`Operation`*``方法（不调用它）。接下来，传递``Begin*`Operation`*``所需的所有参数，但最后的`AsyncCallback`和`object`参数除外。最后，传递`null`。

特别是，在调用`FromAsync`之前不要调用``Begin*`Operation`*``方法。您可以调用`FromAsync`，传递从``Begin*`Operation`*``获取的`IAsyncOperation`，但如果以这种方式调用它，`FromAsync`将被迫使用较低效的实现。

也许你会想知道为什么推荐的模式总是在最后传递一个`null`。在.NET 4.0 中引入了`FromAsync`和`Task`类型，而`async`还不存在。那时，在异步回调中常见使用`state`对象，而`Task`类型通过其`AsyncState`成员支持此功能。在新的`async`模式中，不再需要状态对象，因此对于`state`参数总是传递`null`是正常的。如今，`state`仅用于在优化内存使用时避免闭包实例。

## 另请参阅

配方 8.3 涵盖了为任何类型的通知编写 TAP 包装器。

# 8.3 任意内容的异步包装器

## 问题

您有一个不寻常或非标准的异步操作或事件，并希望通过`await`消费它。

## 解决方案

`TaskCompletionSource<T>`类型可用于在任何场景中构造`Task<T>`对象。使用`TaskCompletionSource<T>`，您可以以三种不同的方式完成任务：成功的结果、故障或取消。

在`async`出现之前，Microsoft 推荐了另外两种异步模式：APM（食谱 8.2）和 EAP（食谱 8.1）。然而，APM 和 EAP 都相当笨拙，并且在某些情况下很难正确使用。因此，出现了一种非官方的约定，使用回调方法，如以下方法：

```cs
public interface IMyAsyncHttpService
{
  void DownloadString(Uri address, Action<string, Exception> callback);
}
```

这类方法遵循这样的约定，即`DownloadString`将启动（异步）下载，并在完成时通过回调调用`callback`，传递结果或异常。通常情况下，`callback`在后台线程上被调用。

类似前面示例的非标准异步方法可以使用`TaskCompletionSource<T>`来进行包装，以便它自然地与`await`一起工作，如下一个示例所示：

```cs
public static Task<string> DownloadStringAsync(
    this IMyAsyncHttpService httpService, Uri address)
{
  var tcs = new TaskCompletionSource<string>();
  httpService.DownloadString(address, (result, exception) =>
  {
    if (exception != null)
      tcs.TrySetException(exception);
    else
      tcs.TrySetResult(result);
  });
  return tcs.Task;
}
```

## 讨论

您可以使用相同的`TaskCompletionSource<T>`模式来包装*任何*非标准的异步方法，无论多么非标准。首先创建`TaskCompletionSource<T>`实例。接下来，安排一个回调，使`TaskCompletionSource<T>`适当地完成其任务。然后，启动实际的异步操作。最后，返回附加到该`TaskCompletionSource<T>`的`Task<T>`。

为了确保这种模式的重要性，您必须确保`TaskCompletionSource<T>`始终被完成。特别是要仔细考虑您的错误处理，并确保`TaskCompletionSource<T>`会得到适当的完成。在最后一个示例中，异常明确地传递到回调中，因此您不需要`catch`块；但有些非标准模式可能需要您在回调中捕获异常并将其放置在`TaskCompletionSource<T>`上。

## 参见

食谱 8.1 涵盖了用于 EAP 成员的 TAP 包装器（``*`Operation`*Async``，``*`Operation`*Completed``）。

食谱 8.2 涵盖了用于 APM 成员的 TAP 包装器（``Begin*`Operation`*``，``End*`Operation`*``）。

# 8.4 用于并行代码的异步包装器

## 问题

您有（CPU 绑定的）并行处理，希望使用`await`消耗它。通常情况下，这是希望的，以使您的 UI 线程不会因等待并行处理完成而阻塞。

## 解决方案

`Parallel` 类型和并行 LINQ 使用线程池来进行并行处理。它们还会将调用线程作为其中一个并行处理线程，因此如果从 UI 线程调用并行方法，则 UI 将在处理完成之前无响应。

为了保持 UI 的响应性，在`Task.Run`中包装并`await`结果：

```cs
await Task.Run(() => Parallel.ForEach(...));
```

本食谱的关键在于并行代码*包括调用线程*在其用于并行处理的线程池中。这对于并行 LINQ 和 `Parallel` 类都是如此。

## 讨论

这是一个简单的配方，但经常被忽视。通过使用 `Task.Run`，您将所有并行处理推送到线程池。`Task.Run` 返回一个代表该并行工作的 `Task`，UI 线程可以（异步地）等待其完成。

本配方仅适用于 UI 代码。在服务器端（例如 ASP.NET）很少进行并行处理，因为服务器主机已经执行了并行处理。因此，服务器端代码不应执行并行处理，也不应将工作推送到线程池。

## 另请参阅

第四章介绍了并行代码的基础知识。

第二章介绍了异步代码的基础知识。

# 8.5 用于 System.Reactive 可观察对象的异步包装器

## 问题

您有一个您希望使用 `await` 消耗的可观察流。

## 解决方案

首先，您需要决定您对事件流中的哪些可观察事件感兴趣。这些是常见的情况：

+   流结束前的最后一个事件

+   下一个事件

+   所有事件

要捕获流中的*最后*一个事件，您可以 `await` `LastAsync` 的结果，或者直接 `await` 可观察对象：

```cs
IObservable<int> observable = ...;
int lastElement = await observable.LastAsync();
// or:  int lastElement = await observable;
```

当您 `await` 可观察对象或 `LastAsync` 时，代码（异步地）等待直到流完成并返回最后一个元素。在底层，`await` 是订阅流。

要捕获流中的*下一个*事件，请使用 `FirstAsync`。在以下代码中，`await` 订阅流，然后在第一个事件到达时完成（并取消订阅）：

```cs
IObservable<int> observable = ...;
int nextElement = await observable.FirstAsync();
```

要捕获流中的*所有*事件，您可以使用 `ToList`：

```cs
IObservable<int> observable = ...;
IList<int> allElements = await observable.ToList();
```

## 讨论

System.Reactive 库提供了使用 `await` 消耗流所需的所有工具。唯一棘手的部分是您必须考虑 awaitable 是否会等待直到流完成。在本配方的示例中，`LastAsync`、`ToList` 和直接 `await` 将等待直到流完成；`FirstAsync` 仅等待下一个事件。

如果这些示例不满足您的需求，请记住，您可以完全利用 LINQ 的强大功能以及 System.Reactive 操纵器。诸如 `Take` 和 `Buffer` 的运算符也可以帮助您在不必等待整个流完成的情况下异步等待所需的元素。

一些与 `await` 一起使用的运算符——如 `FirstAsync` 和 `LastAsync`——实际上并不返回 `Task<T>`。如果您计划使用 `Task.WhenAll` 或 `Task.WhenAny`，那么您需要一个实际的 `Task<T>`，您可以通过在任何可观察对象上调用 `ToTask` 来获得。`ToTask` 将返回一个在流中完成的 `Task<T>`。

## 另请参阅

配方 8.6 介绍了在可观察流中使用异步代码的方法。

配方 8.8 介绍了将可观察流用作数据流块输入的方法（可以执行异步工作）。

6.3 节介绍了用于可观察流的窗口和缓冲区。

# 8.6 System.Reactive 对异步代码的可观察包装

## 问题

您有一个想要与可观察对象结合的异步操作。

## 解决方案

任何异步操作都可以被视为执行以下两种操作之一的可观察流：

+   生成单个元素然后完成

+   未产生任何元素的故障

要实现这种转换，System.Reactive 库可以简单地将`Task<T>`转换为`IObservable<T>`。以下代码开始异步下载网页，并将其视为可观察序列：

```cs
IObservable<HttpResponseMessage> GetPage(HttpClient client)
{
  Task<HttpResponseMessage> task =
      client.GetAsync("http://www.example.com/");
  return task.ToObservable();
}
```

`ToObservable`方法假定您已经调用了`async`方法并有一个`Task`可以转换。

另一种方法是调用`StartAsync`。`StartAsync`也会立即调用`async`方法，但支持取消：如果取消订阅，则会取消`async`方法：

```cs
IObservable<HttpResponseMessage> GetPage(HttpClient client)
{
  return Observable.StartAsync(
      token => client.GetAsync("http://www.example.com/", token));
}
```

`ToObservable`和`StartAsync`立即启动异步操作，无需等待订阅；可观察对象是“热的”。要创建一个“冷”的可观察对象，仅在订阅时开始操作，请使用`FromAsync`（它也支持像`StartAsync`一样的取消）：

```cs
IObservable<HttpResponseMessage> GetPage(HttpClient client)
{
  return Observable.FromAsync(
      token => client.GetAsync("http://www.example.com/", token));
}
```

`FromAsync`与`ToObservable`和`StartAsync`显著不同，后两者返回已经开始的`async`操作的可观察对象。`FromAsync`每次订阅时启动一个新的独立`async`操作。

最后，您可以使用`SelectMany`的特殊重载来为源流中的每个事件启动异步操作。`SelectMany`也支持取消。

以下示例获取现有的 URL 事件流，然后在每个 URL 到达时初始化请求：

```cs
IObservable<HttpResponseMessage> GetPages(
    IObservable<string> urls, HttpClient client)
{
  return urls.SelectMany(
      (url, token) => client.GetAsync(url, token));
}
```

## 讨论

System.Reactive 在引入`async`之前就已存在，但添加了这些操作符（及其他操作符），以便与`async`代码良好地互操作。建议您使用描述的操作符，即使您可以使用其他 System.Reactive 操作符构建相同的功能。

## 参见

8.5 节介绍了如何使用异步代码消耗可观察流。

8.8 节涵盖了使用数据流块（可能包含异步代码）作为可观察流源的方法。

# 8.7 异步流与数据流网格

## 问题

您的解决方案的一部分使用了异步流，另一部分使用了数据流网格，并且需要在它们之间传递数据。

## 解决方案

将通道作为异步流来消耗是内置在通道类型中的；有关详细信息，请参见 9.8 节。将 TPL Dataflow 块作为异步流消耗则稍微复杂一些，但当然是可行的。我发现最简单的方法是首先为数据流块定义一个扩展方法，使其 API 更类似于通道，然后使用该扩展方法将它们作为异步流消耗：

```cs
public static class DataflowExtensions
{
  public static bool TryReceiveItem<T>(this ISourceBlock<T> block, out T value)
  {
    if (block is IReceivableSourceBlock<T> receivableSourceBlock)
      return receivableSourceBlock.TryReceive(out value);

    try
    {
      value = block.Receive(TimeSpan.Zero);
      return true;
    }
    catch (TimeoutException)
    {
      // There is no item available right now.
      value = default;
      return false;
    }
    catch (InvalidOperationException)
    {
      // The block is complete and there are no more items.
      value = default;
      return false;
    }
  }

  public static async IAsyncEnumerable<T> ReceiveAllAsync<T>(
      this ISourceBlock<T> block,
 [EnumeratorCancellation] CancellationToken cancellationToken = default)
  {
    while (await block
        .OutputAvailableAsync(cancellationToken).ConfigureAwait(false))
    {
      while (block.TryReceiveItem(out var value))
      {
        yield return value;
      }
    }
  }
}
```

详细内容请参见第 3.4 节，关于`EnumeratorCancellation`属性的详细信息。

使用前面代码示例中的扩展方法，可以将任何输出数据流块消耗为异步流：

```cs
var multiplyBlock = new TransformBlock<int, int>(value => value * 2);

multiplyBlock.Post(5);
multiplyBlock.Post(2);
multiplyBlock.Complete();

await foreach (int item in multiplyBlock.ReceiveAllAsync())
{
  Console.WriteLine(item);
}
```

还可以将异步流用作数据流块的项目来源。您只需循环获取项目并将其放入块中。以下代码中有几个假设可能不适用于每种场景。首先，代码假设您希望在流完成时完成块。其次，它始终在其调用线程上运行；某些场景可能希望始终在线程池线程上运行整个循环：

```cs
public static async Task WriteToBlockAsync<T>(
    this IAsyncEnumerable<T> enumerable,
    ITargetBlock<T> block, CancellationToken token = default)
{
  try
  {
    await foreach (var item in enumerable
        .WithCancellation(token).ConfigureAwait(false))
    {
      await block.SendAsync(item, token).ConfigureAwait(false);
    }

    block.Complete();
  }
  catch (Exception ex)
  {
    block.Fault(ex);
  }
}
```

## 讨论

此处的扩展方法旨在作为一个起点。特别是，`WriteToBlockAsync`扩展方法确实做了一些假设；在使用之前，请务必考虑这些方法的行为，并确保它们在您的场景中的行为是适当的。

## 查看也可参考

第 9.8 节介绍了如何将通道作为异步流进行消耗。

第 3.4 节介绍了取消异步流的相关内容。

第五章介绍了 TPL Dataflow 的相关技巧。

第三章介绍了异步流的相关技巧。

# 8.8 System.Reactive 可观察对象和数据流网格

## 问题

您的解决方案的一部分使用了 System.Reactive 的可观察对象，另一部分使用了数据流网格，您需要它们进行通信。

System.Reactive 的可观察对象和数据流网格各自具有自己的用途，部分概念重叠；此处演示了它们如何轻松地协同工作，以便您可以在作业的每个部分使用最佳工具。

## 解决方案

首先，让我们考虑将数据流块用作可观察流的输入。以下代码创建了一个缓冲块（不进行任何处理），并通过调用`AsObservable`从该块创建了一个可观察接口：

```cs
var buffer = new BufferBlock<int>();
IObservable<int> integers = buffer.AsObservable();
integers.Subscribe(data => Trace.WriteLine(data),
    ex => Trace.WriteLine(ex),
    () => Trace.WriteLine("Done"));

buffer.Post(13);
```

缓冲块和可观察流可以正常完成或出现错误，而`AsObservable`方法将块的完成（或故障）转换为可观察流的完成。但是，如果块因异常而故障，则在传递给可观察流时该异常将被包装在`AggregateException`中。这类似于链接块传播它们的故障的方式。

将网格视为可观察流的目的地只是稍微复杂了一点。以下代码调用`AsObserver`以使块能够订阅可观察流：

```cs
IObservable<DateTimeOffset> ticks =
    Observable.Interval(TimeSpan.FromSeconds(1))
        .Timestamp()
        .Select(x => x.Timestamp)
        .Take(5);

var display = new ActionBlock<DateTimeOffset>(x => Trace.WriteLine(x));
ticks.Subscribe(display.AsObserver());

try
{
  display.Completion.Wait();
  Trace.WriteLine("Done.");
}
catch (Exception ex)
{
  Trace.WriteLine(ex);
}
```

与之前一样，可观察流的完成被转换为块的完成，而可观察流的任何错误被转换为块的故障。

## 讨论

数据流块和可观察流在概念上有很多共同之处。它们都通过它们传递数据，并且都理解完成和故障。它们设计用于不同的场景；TPL 数据流适用于异步和并行编程的混合，而 System.Reactive 则适用于反应式编程。然而，概念上的重叠足够兼容，它们能够非常自然地很好地协同工作。

## 参见

Recipe 8.5 讲解了如何使用异步代码消耗可观察流。

Recipe 8.6 讲解了如何在可观察流中使用异步代码。

# 8.9 将 System.Reactive 可观察对象转换为异步流

## 问题

您的解决方案的一部分使用了 System.Reactive 的可观察对象，并且希望将它们作为异步流消耗。

## 解决方案

System.Reactive 的可观察对象是推送型的，而异步流是拉取型的。因此，一开始就需要意识到这种概念上的不匹配。您需要一种方法来保持对可观察流的响应性，存储其通知直到消费代码请求它们。

最简单的解决方案已经包含在 `System.Linq.Async` 库中：

```cs
IObservable<long> observable =
    Observable.Interval(TimeSpan.FromSeconds(1));

// WARNING: May consume unbounded memory; see discussion!
IAsyncEnumerable<long> enumerable =
    observable.ToAsyncEnumerable();
```

###### 提示

`ToAsyncEnumerable` 扩展方法位于 [`System.Linq.Async`](http://bit.ly/sys-linq-async) NuGet 包中。

然而，需要注意的是，这个简单的 `ToAsyncEnumerable` 扩展方法在内部使用了一个无界的生产者/消费者队列。本质上，这与您可以自己编写的使用通道作为无界生产者/消费者队列的扩展方法相同：

```cs
// WARNING: May consume unbounded memory; see discussion!
public static async IAsyncEnumerable<T> ToAsyncEnumerable<T>(
    this IObservable<T> observable)
{
  Channel<T> buffer = Channel.CreateUnbounded<T>();
  using (observable.Subscribe(
      value => buffer.Writer.TryWrite(value),
      error => buffer.Writer.Complete(error),
      () => buffer.Writer.Complete()))
  {
    await foreach (T item in buffer.Reader.ReadAllAsync())
      yield return item;
  }
}
```

这些都是简单的解决方案，但它们使用了无界队列，因此只有在消费者能够（最终）跟上可观察事件时才应使用它们。如果生产者在一段时间内运行得比消费者快，是可以接受的；在此期间，可观察事件进入缓冲区。只要生产者最终赶上，前述解决方案就会起作用。但是，如果生产者始终比消费者运行得快，可观察事件将继续到达，扩展缓冲区，并最终耗尽进程的所有内存。

您可以通过使用有界队列来避免内存问题。其中的折衷是，如果可观察事件填满队列，您必须决定如何处理额外的项。一种选择是丢弃额外的项；以下示例代码使用有界通道，在缓冲区满时丢弃最旧的可观察通知：

```cs
// WARNING: May discard items; see discussion!
public static async IAsyncEnumerable<T> ToAsyncEnumerable<T>(
    this IObservable<T> observable, int bufferSize)
{
  var bufferOptions = new BoundedChannelOptions(bufferSize)
  {
    FullMode = BoundedChannelFullMode.DropOldest,
  };
  Channel<T> buffer = Channel.CreateBounded<T>(bufferOptions);
  using (observable.Subscribe(
      value => buffer.Writer.TryWrite(value),
      error => buffer.Writer.Complete(error),
      () => buffer.Writer.Complete()))
  {
    await foreach (T item in buffer.Reader.ReadAllAsync())
      yield return item;
  }
}
```

## 讨论

当您的生产者运行速度快于消费者时，您有两个选择：要么缓冲生产者项目（假设生产者最终能够赶上），要么限制生产者的项目数量。本食谱的第二种解决方案通过丢弃不适合缓冲区的项目来限制生产者的项目。您还可以通过使用专为此设计的可观察操作符，如 `Throttle` 或 `Sample` 来限制生产者的项目；详细信息请参见 食谱 6.4。根据您的需求，在将输入可观察对象转换为 `IAsyncEnumerable<T>` 之前，最好使用 `Throttle` 或 `Sample` 技术中的一种来限制输入可观察对象。

除了有界队列和无界队列，还有第三个选项未在此处介绍：使用背压来通知可观察流，在缓冲区准备好接收通知之前必须停止生成通知。不幸的是，截至撰写本文时，System.Reactive 尚未标准化背压模式，因此这不是一个可行的选项。背压是复杂而微妙的，其他语言的响应式库已经实现了不同的背压模式。尚不清楚 System.Reactive 是否会采纳其中一种，发明自己的背压模式，还是干脆放弃解决背压的问题。

## 另请参阅

食谱 6.4 介绍了用于节流输入的 System.Reactive 操作符。

食谱 9.8 介绍了如何使用 Channel 作为无限制的生产者/消费者队列。

食谱 9.10 介绍了使用 Channel 作为采样队列，在其满时丢弃项目。
