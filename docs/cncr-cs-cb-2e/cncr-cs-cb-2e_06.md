# 第六章：System.Reactive 基础

LINQ 是一组语言功能，使开发人员能够查询序列。最常见的两个 LINQ 提供程序是内置的 LINQ to Objects（基于 `IEnumerable<T>`）和 LINQ to Entities（基于 `IQueryable<T>`）。还有许多其他提供程序可用，大多数提供程序具有相同的一般结构。查询是惰性评估的，序列根据需要生成值。在概念上，这是一种拉模型；在评估过程中，逐个从查询中获取值项。

System.Reactive (Rx) 将事件视为随时间到达的数据序列。因此，你可以将 Rx 视为基于 `IObservable<T>` 的事件 LINQ。观察者和其他 LINQ 提供程序之间的主要区别在于，Rx 是一种“推”模型，即查询定义了程序在事件到达时如何响应。Rx 在 LINQ 基础上构建，作为扩展方法添加了一些强大的新操作符。

本章介绍了一些常见的 Rx 操作。请记住，所有的 LINQ 操作符也都可用，因此像过滤 (`Where`) 和投影 (`Select`) 这样的简单操作在概念上与任何其他 LINQ 提供程序的工作方式相同。我们不会在这里介绍这些常见的 LINQ 操作；我们将专注于 Rx 在 LINQ 之上构建的新功能，特别是涉及*时间*的功能。

要使用 System.Reactive，请将 NuGet 包 [`System.Reactive`](http://bit.ly/sys-reactive) 安装到你的应用程序中。

# 6.1 转换 .NET 事件

## 问题

你有一个事件，需要将其视为 System.Reactive 的输入流，每次触发事件时通过 `OnNext` 产生一些数据。

## 解决方案

`Observable` 类定义了几个事件转换器。大多数 .NET 框架事件都兼容 `FromEventPattern`，但如果你有不遵循常规模式的事件，可以使用 `FromEvent`。

如果事件委托类型为 `EventHandler<T>`，则 `FromEventPattern` 的效果最佳。许多较新的框架类型使用此事件委托类型。例如，`Progress<T>` 类型定义了一个 `ProgressChanged` 事件，类型为 `EventHandler<T>`，因此可以轻松用 `FromEventPattern` 包装：

```cs
var progress = new Progress<int>();
IObservable<EventPattern<int>> progressReports =
    Observable.FromEventPattern<int>(
        handler => progress.ProgressChanged += handler,
        handler => progress.ProgressChanged -= handler);
progressReports.Subscribe(data => Trace.WriteLine("OnNext: " + data.EventArgs));
```

注意这里，`data.EventArgs` 的类型强制为 `int`。`FromEventPattern` 的类型参数（如前面的例子中的 `int`）与 `EventHandler<T>` 中的 `T` 类型相同。`FromEventPattern` 的两个 lambda 参数使 System.Reactive 能够订阅和取消订阅事件。

较新的用户界面框架使用 `EventHandler<T>`，可以轻松与 `FromEventPattern` 配合使用，但旧类型通常为每个事件定义一个独特的委托类型。这些也可以与 `FromEventPattern` 一起使用，但需要更多工作。例如，`System.Timers.Timer` 类定义了一个 `Elapsed` 事件，类型为 `ElapsedEventHandler`。你可以像这样用 `FromEventPattern` 包装旧事件：

```cs
var timer = new System.Timers.Timer(interval: 1000) { Enabled = true };
IObservable<EventPattern<ElapsedEventArgs>> ticks =
    Observable.FromEventPattern<ElapsedEventHandler, ElapsedEventArgs>(
        handler => (s, a) => handler(s, a),
        handler => timer.Elapsed += handler,
        handler => timer.Elapsed -= handler);
ticks.Subscribe(data => Trace.WriteLine("OnNext: " + data.EventArgs.SignalTime));
```

请注意，在此示例中，`data.EventArgs`仍然是强类型的。`FromEventPattern`的类型参数现在是唯一的处理程序类型和派生的`EventArgs`类型。`FromEventPattern`的第一个 Lambda 参数是从`EventHandler<ElapsedEventArgs>`到`ElapsedEventHandler`的转换器；该转换器除了传递事件之外不应执行其他操作。

那种语法确实变得笨拙了。这里有另一种选择，使用反射：

```cs
var timer = new System.Timers.Timer(interval: 1000) { Enabled = true };
IObservable<EventPattern<object>> ticks =
    Observable.FromEventPattern(timer, nameof(Timer.Elapsed));
ticks.Subscribe(data => Trace.WriteLine("OnNext: "
    + ((ElapsedEventArgs)data.EventArgs).SignalTime));
```

使用这种方法，调用`FromEventPattern`会更加简单。请注意，此方法存在一个缺点：消费者无法获得强类型的数据。因为`data.EventArgs`的类型是`object`，您必须自己将其转换为`ElapsedEventArgs`。

## 讨论

事件是 System.Reactive 流的常见数据源。本文介绍了如何包装符合标准事件模式（第一个参数是发送者，第二个参数是事件参数类型）的任何事件。如果您有不寻常的事件类型，仍然可以使用`Observable.FromEvent`方法重载将它们包装成可观察对象。

当事件被包装成可观察对象时，每次事件被触发时都会调用`OnNext`。当处理`AsyncCompletedEventArgs`时，这可能会导致令人惊讶的行为，因为任何异常都作为数据（`OnNext`）传递，而不是作为错误（`OnError`）。例如，考虑`WebClient.DownloadStringCompleted`的这个包装器：

```cs
var client = new WebClient();
IObservable<EventPattern<object>> downloadedStrings =
    Observable.
    FromEventPattern(client, nameof(WebClient.DownloadStringCompleted));
downloadedStrings.Subscribe(
    data =>
    {
      var eventArgs = (DownloadStringCompletedEventArgs)data.EventArgs;
      if (eventArgs.Error != null)
        Trace.WriteLine("OnNext: (Error) " + eventArgs.Error);
      else
        Trace.WriteLine("OnNext: " + eventArgs.Result);
    },
    ex => Trace.WriteLine("OnError: " + ex.ToString()),
    () => Trace.WriteLine("OnCompleted"));
client.DownloadStringAsync(new Uri("http://invalid.example.com/"));
```

当`WebClient.DownloadStringAsync`以错误完成时，事件会通过`AsyncCompletedEventArgs.Error`中的异常来触发。不幸的是，System.Reactive 将此视为数据事件，因此如果随后运行前述代码，则会打印出`OnNext: (Error)`而不是`OnError:`。

有些事件的订阅和取消订阅必须在特定的上下文中完成。例如，许多 UI 控件上的事件必须从 UI 线程订阅。System.Reactive 提供了一个操作符来控制订阅和取消订阅的上下文：`SubscribeOn`。在大多数情况下，UI 基础的订阅都是从 UI 线程进行的，所以`SubscribeOn`操作符在大多数情况下并不是必需的。

###### 提示

`SubscribeOn`控制添加和移除事件处理程序的代码的上下文。不要将其与`ObserveOn`混淆，后者控制可观察通知的上下文（传递给`Subscribe`的委托）。

## 参见

Recipe 6.2 介绍了如何更改引发事件的上下文。

Recipe 6.4 介绍了如何节流事件，以防止订阅者被压倒。

# 6.2 将通知发送到上下文

## 问题

System.Reactive 尽力保持线程无关性。因此，它会在当前线程中引发通知（例如，`OnNext`）。每个`OnNext`通知都将按顺序发生，但不一定在同一线程上。

您通常希望在特定上下文中引发这些通知。例如，UI 元素应仅从拥有它们的 UI 线程进行操作，因此如果您在响应在线程池线程上到达的通知时更新 UI，则需要切换到 UI 线程。

## 解决方案

System.Reactive 提供了 `ObserveOn` 操作符，用于将通知移动到另一个调度程序上。

考虑以下示例，它使用 `Interval` 操作符每秒创建一个 `OnNext` 通知：

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
  Trace.WriteLine($"UI thread is {Environment.CurrentManagedThreadId}");
  Observable.Interval(TimeSpan.FromSeconds(1))
      .Subscribe(x => Trace.WriteLine(
          $"Interval {x} on thread {Environment.CurrentManagedThreadId}"));
}
```

在我的机器上，输出看起来如下：

```cs
UI thread is 9
Interval 0 on thread 10
Interval 1 on thread 10
Interval 2 on thread 11
Interval 3 on thread 11
Interval 4 on thread 10
Interval 5 on thread 11
Interval 6 on thread 11
```

由于 `Interval` 基于定时器（没有特定的线程），通知是在线程池线程上引发的，而不是在 UI 线程上。如果您需要更新 UI 元素，您可以通过 `ObserveOn` 传递通知，并传递表示 UI 线程的同步上下文。

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
  SynchronizationContext uiContext = SynchronizationContext.Current;
  Trace.WriteLine($"UI thread is {Environment.CurrentManagedThreadId}");
  Observable.Interval(TimeSpan.FromSeconds(1))
      .ObserveOn(uiContext)
      .Subscribe(x => Trace.WriteLine(
          $"Interval {x} on thread {Environment.CurrentManagedThreadId}"));
}
```

`ObserveOn` 的另一个常见用法是在必要时将 *UI 线程* 切换到其他线程。考虑这样一种情况：每当鼠标移动时，您需要进行一些耗费 CPU 的计算。默认情况下，所有鼠标移动都在 UI 线程上引发，因此您可以使用 `ObserveOn` 将这些通知移动到线程池线程上，执行计算，然后将结果通知移回 UI 线程：

```cs
SynchronizationContext uiContext = SynchronizationContext.Current;
Trace.WriteLine($"UI thread is {Environment.CurrentManagedThreadId}");
Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
        handler => (s, a) => handler(s, a),
        handler => MouseMove += handler,
        handler => MouseMove -= handler)
    .Select(evt => evt.EventArgs.GetPosition(this))
    .ObserveOn(Scheduler.Default)
    .Select(position =>
    {
      // Complex calculation
      Thread.Sleep(100);
      var result = position.X + position.Y;
      var thread = Environment.CurrentManagedThreadId;
      Trace.WriteLine($"Calculated result {result} on thread {thread}");
      return result;
    })
    .ObserveOn(uiContext)
    .Subscribe(x => Trace.WriteLine(
        $"Result {x} on thread {Environment.CurrentManagedThreadId}"));
```

如果您执行此示例，您会看到计算在线程池线程上进行，并且结果在 UI 线程上打印出来。但是，您也会注意到计算和结果会滞后于输入；它们会排队，因为鼠标位置更新频率超过每 100 毫秒。System.Reactive 提供了几种处理此情况的技术；其中一种常见的技术在 Recipe 6.4 中介绍，即节流输入。

## 讨论

`ObserveOn` 实际上将通知移动到 System.Reactive 的 *调度程序* 上。本文介绍了默认的（线程池）调度程序以及创建 UI 调度程序的一种方法。`ObserveOn` 操作符的最常见用途是在 UI 线程上移动或移出，但调度程序在其他场景中也很有用。调度程序在更高级的场景中也很有用，例如在单元测试时模拟时间流逝，您可以在 Recipe 7.6 中找到相关内容。

###### 提示

`ObserveOn` 控制观察通知的上下文。这与控制添加和移除事件处理程序的代码上下文的 `SubscribeOn` 不同。

## 参见

Recipe 6.1 讲解了如何从事件创建序列，并使用 `SubscribeOn`。

Recipe 6.4 讲解了对事件流进行节流处理。

Recipe 7.6 介绍了用于测试 System.Reactive 代码的特殊调度程序。

# 6.3 使用窗口和缓冲区分组事件数据

## 问题

你有一系列事件，并且希望在事件到达时对其进行分组。例如，您需要对输入的成对事件作出反应。另一个例子是，您需要在两秒的时间窗口内对所有输入作出反应。

## 解决方案

`System.Reactive` 提供了一对操作符来分组输入序列：`Buffer` 和 `Window`。`Buffer` 会保存输入事件，直到组合完成，然后一次性将它们作为事件集合转发。`Window` 会逻辑上分组输入事件，但会随着它们的到来立即传递。`Buffer` 的返回类型是 `IObservable<IList<T>>`（事件流的集合）；`Window` 的返回类型是 `IObservable<IObservable<T>>`（事件流的事件流）。

以下示例使用 `Interval` 操作符每秒创建一个 `OnNext` 通知，并以两个为一组进行缓冲：

```cs
Observable.Interval(TimeSpan.FromSeconds(1))
    .Buffer(2)
    .Subscribe(x => Trace.WriteLine(
        $"{DateTime.Now.Second}: Got {x[0]} and {x[1]}"));
```

在我的机器上，此代码每两秒产生一对输出：

```cs
13: Got 0 and 1
15: Got 2 and 3
17: Got 4 and 5
19: Got 6 and 7
21: Got 8 and 9
```

以下是使用 `Window` 创建两个事件组的类似示例：

```cs
Observable.Interval(TimeSpan.FromSeconds(1))
    .Window(2)
    .Subscribe(group =>
    {
      Trace.WriteLine($"{DateTime.Now.Second}: Starting new group");
      group.Subscribe(
          x => Trace.WriteLine($"{DateTime.Now.Second}: Saw {x}"),
          () => Trace.WriteLine($"{DateTime.Now.Second}: Ending group"));
    });
```

在我的机器上，此 `Window` 示例产生以下输出：

```cs
17: Starting new group
18: Saw 0
19: Saw 1
19: Ending group
19: Starting new group
20: Saw 2
21: Saw 3
21: Ending group
21: Starting new group
22: Saw 4
23: Saw 5
23: Ending group
23: Starting new group
```

这些示例说明了 `Buffer` 和 `Window` 之间的区别。`Buffer` 等待其组内的所有事件，然后发布单个集合。`Window` 以相同的方式分组事件，但会在事件到达时即刻发布它们；`Window` 立即发布一个可观察对象，用于发布该窗口的事件。

`Buffer` 和 `Window` 也适用于时间跨度。以下代码示例中，所有鼠标移动事件都在一秒钟的窗口内收集：

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Buffer(TimeSpan.FromSeconds(1))
      .Subscribe(x => Trace.WriteLine(
          $"{DateTime.Now.Second}: Saw {x.Count} items."));
}
```

根据鼠标的移动方式，您应该看到类似以下的输出：

```cs
49: Saw 93 items.
50: Saw 98 items.
51: Saw 39 items.
52: Saw 0 items.
53: Saw 4 items.
54: Saw 0 items.
55: Saw 58 items.
```

## 讨论

`Buffer` 和 `Window` 是您用来管理输入并将其形成您想要的形式的工具之一。另一种有用的技术是限流，您将在 Recipe 6.4 中了解更多。

`Buffer` 和 `Window` 都有其他重载版本，可用于更高级的场景。带有 `skip` 和 `timeShift` 参数的重载允许您创建与其他组重叠或在组之间跳过元素的组。还有带有委托参数的重载，允许您动态定义组的边界。

## 参见

Recipe 6.1 讲解了如何从事件中创建序列。

Recipe 6.4 讲解了如何限流事件流。

# 6.4 通过限流和采样来控制事件流

## 问题

编写响应式代码时常见的问题是事件到达速度过快。快速移动的事件流可能会超出程序的处理能力。

## 解决方案

`System.Reactive` 提供了专门用于处理大量事件数据的操作符。`Throttle` 和 `Sample` 操作符为我们提供了两种不同的方法来控制快速输入事件。

`Throttle` 操作符建立了一个滑动超时窗口。当接收到新事件时，它会重置超时窗口。当超时窗口到期时，它会发布窗口内最后到达的事件值。

以下示例监控鼠标移动，并使用 `Throttle` 仅在鼠标静止一秒钟后报告更新：

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Select(x => x.EventArgs.GetPosition(this))
      .Throttle(TimeSpan.FromSeconds(1))
      .Subscribe(x => Trace.WriteLine(
          $"{DateTime.Now.Second}: Saw {x.X + x.Y}"));
}
```

输出因鼠标移动而大不相同，但在我的机器上的一个示例运行看起来是这样的：

```cs
47: Saw 139
49: Saw 137
51: Saw 424
56: Saw 226
```

`Throttle`经常用于像自动完成这样的情况，当用户在文本框中输入文本时，您不希望在用户停止输入之前进行实际查找。

`Sample` 采用了不同的方法来控制快速移动的序列。`Sample` 建立了一个常规的超时周期，并在每次超时到期时发布该窗口内的最新值。如果在抽样周期内没有收到值，则不会发布任何结果。

下面的示例捕获鼠标移动并在一秒间隔内对其进行抽样。与 `Throttle` 示例不同，这个 `Sample` 示例不需要您保持鼠标静止以查看数据：

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Select(x => x.EventArgs.GetPosition(this))
      .Sample(TimeSpan.FromSeconds(1))
      .Subscribe(x => Trace.WriteLine(
          $"{DateTime.Now.Second}: Saw {x.X + x.Y}"));
}
```

当我第一次让鼠标静止几秒钟然后持续移动时，这是我在我的机器上的输出：

```cs
12: Saw 311
17: Saw 254
18: Saw 269
19: Saw 342
20: Saw 224
21: Saw 277
```

## 讨论

对于驯服输入的洪流来说，节流和抽样是必不可少的工具。不要忘记您还可以使用标准 LINQ `Where` 运算符轻松进行过滤。您可以将 `Throttle` 和 `Sample` 运算符视为类似于 `Where`，只是它们基于时间窗口而不是事件数据进行过滤。这三个运算符各自以不同的方式帮助您驯服快速移动的输入流。

## 参见

Recipe 6.1 介绍了如何从事件创建序列。

Recipe 6.2 介绍了如何改变事件触发的上下文。

# 6.5 超时

## 问题

您期望在一定时间内收到事件，并确保您的程序能够及时响应，即使事件没有及时到达。最常见的情况是，这种期望的事件是单个异步操作（例如，期待来自 Web 服务请求的响应）。

## 解决方案

`Timeout` 运算符在其输入流上建立了一个滑动超时窗口。每当新事件到达时，超时窗口就会被重置。如果在该窗口内没有看到事件而超时，则 `Timeout` 运算符将使用一个 `TimeoutException` 的 `OnError` 通知结束流。

下面的示例发出对示例域的网页请求，并设置了一秒钟的超时。为了启动网页请求，代码使用 `ToObservable` 将 `Task<T>` 转换为 `IObservable<T>`（参见 Recipe 8.6）：

```cs
void GetWithTimeout(HttpClient client)
{
  client.GetStringAsync("http://www.example.com/").ToObservable()
      .Timeout(TimeSpan.FromSeconds(1))
      .Subscribe(
          x => Trace.WriteLine($"{DateTime.Now.Second}: Saw {x.Length}"),
          ex => Trace.WriteLine(ex));
}
```

`Timeout` 对于异步操作（例如 Web 请求）非常理想，但它可以应用于任何事件流。下面的示例将 `Timeout` 应用于鼠标移动，这样更容易玩耍：

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Select(x => x.EventArgs.GetPosition(this))
      .Timeout(TimeSpan.FromSeconds(1))
      .Subscribe(
          x => Trace.WriteLine($"{DateTime.Now.Second}: Saw {x.X + x.Y}"),
          ex => Trace.WriteLine(ex));
}
```

在我的电脑上，我移动了鼠标一下然后静止了一秒钟，得到了以下结果：

```cs
16: Saw 180
16: Saw 178
16: Saw 177
16: Saw 176
System.TimeoutException: The operation has timed out.
```

请注意，一旦`TimeoutException`被发送到`OnError`，流就结束了。不再传递更多的鼠标移动。也许您并不希望出现这种行为，因此`Timeout`操作符有多个重载版本，当超时发生时，会用第二个流替代结束流，并不抛出异常。

下面示例中的代码观察鼠标移动直到超时。超时后，代码观察鼠标点击：

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
  IObservable<Point> clicks =
      Observable.FromEventPattern<MouseButtonEventHandler, MouseButtonEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseDown += handler,
          handler => MouseDown -= handler)
      .Select(x => x.EventArgs.GetPosition(this));

  Observable.FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Select(x => x.EventArgs.GetPosition(this))
      .Timeout(TimeSpan.FromSeconds(1), clicks)
      .Subscribe(
          x => Trace.WriteLine($"{DateTime.Now.Second}: Saw {x.X},{x.Y}"),
          ex => Trace.WriteLine(ex));
}
```

在我的机器上，我稍微移动了一下鼠标，然后静止了一秒钟，然后点击了几个不同的点。以下输出显示了鼠标移动快速通过直到超时，然后显示了两次点击：

```cs
49: Saw 95,39
49: Saw 94,39
49: Saw 94,38
49: Saw 94,37
53: Saw 130,141
55: Saw 469,4
```

## 讨论

在复杂应用程序中，`Timeout`是一个必不可少的操作符，因为即使世界其他地方没有响应，您始终希望程序响应。当您有异步操作时，它尤其有用，但它也可以应用于任何事件流。请注意，底层操作实际上并没有被取消；在超时的情况下，操作将继续执行，直到成功或失败。

## 参见

6.1 菜谱介绍了如何从事件创建序列。

8.6 菜谱介绍了如何将异步代码包装为可观察事件流。

10.6 菜谱介绍了如何因`CancellationToken`取消订阅序列。

10.3 菜谱介绍了如何使用`CancellationToken`作为超时。
