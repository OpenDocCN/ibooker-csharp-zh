# 第十章：取消

.NET 4.0 框架引入了详尽且设计良好的取消支持。这种支持是协作性的，这意味着可以请求取消但不能强制执行取消。由于取消是协作性的，除非编写支持取消的代码，否则不可能取消代码。因此，我建议尽可能在自己的代码中支持取消。

取消是一种信号类型，有两个不同的方面：触发取消的源和响应取消的接收器。在.NET 中，源是 `CancellationTokenSource`，接收器是 `CancellationToken`。本章的配方涵盖了取消的来源和接收器在正常使用中的应用，并描述了如何使用取消支持与非标准取消形式进行交互。

取消被视为一种特殊的错误。约定是取消的代码将抛出 `OperationCanceledException` 类型的异常（或其派生类型，如 `TaskCanceledException`）。这样调用代码就知道已观察到取消。

为了向调用代码指示您的方法支持取消，您应该将 `CancellationToken` 作为参数。该参数通常是最后一个参数，除非您的方法还报告进度（配方 2.3）。您还可以考虑为不需要取消的消费者提供重载或默认参数值：

```cs
public void CancelableMethodWithOverload(CancellationToken cancellationToken)
{
  // Code goes here.
}

public void CancelableMethodWithOverload()
{
  CancelableMethodWithOverload(CancellationToken.None);
}

public void CancelableMethodWithDefault(
    CancellationToken cancellationToken = default)
{
  // Code goes here.
}
```

`CancellationToken.None` 表示一个永远不会被取消的取消标记，是一个特殊值，等同于 `default(CancellationToken)`。当消费者不希望操作被取消时，会传递这个值。

异步流处理取消方式类似，但更复杂。有关异步流的取消详细信息，请参见配方 3.4。

# 10.1 发出取消请求

## 问题

您的代码调用可取消的代码（接受 `CancellationToken` 参数），而您需要取消它。

## 解决方案

`CancellationTokenSource` 类型是 `CancellationToken` 的源头。它仅使代码能够响应取消请求；`CancellationTokenSource` 的成员允许代码请求取消。

每个 `CancellationTokenSource` 都是独立的（除非将它们链接在一起，如配方 10.8 所述）。`Token` 属性返回该源的 `CancellationToken`，`Cancel` 方法则发出实际的取消请求。

以下代码演示了如何创建 `CancellationTokenSource`，以及如何使用 `Token` 和 `Cancel`。该代码使用了一个 `async` 方法，因为在短代码示例中更容易说明；相同的 `Token`/`Cancel` 对被用于取消*所有*类型的代码：

```cs
void IssueCancelRequest()
{
  using var cts = new CancellationTokenSource();
  var task = CancelableMethodAsync(cts.Token);

  // At this point, the operation has been started.

  // Issue the cancellation request.
  cts.Cancel();
}
```

在上面的示例代码中，`task` 变量在启动后被忽略；在真实的代码中，该任务可能会被存储在某个地方，并等待其完成，以便最终用户能够看到最终结果。

当您取消代码时，几乎总会存在竞态条件。可取消的代码可能在取消请求发出时*几乎要完成*，如果它在完成之前没有检查其取消令牌，它将实际上成功完成。实际上，当您取消代码时，有三种可能的结果：它可能响应取消请求（抛出 `OperationCanceledException`），它可能成功完成，或者它可能由于与取消无关的错误而完成（抛出其他异常）。

下面的代码与上一个示例相似，但它等待任务完成，展示了所有三种可能的结果：

```cs
async Task IssueCancelRequestAsync()
{
  using var cts = new CancellationTokenSource();
  var task = CancelableMethodAsync(cts.Token);

  // At this point, the operation is happily running.

  // Issue the cancellation request.
  cts.Cancel();

  // (Asynchronously) wait for the operation to finish.
  try
  {
    await task;
    // If we get here, the operation completed successfully
    //  before the cancellation took effect.
  }
  catch (OperationCanceledException)
  {
    // If we get here, the operation was canceled before it completed.
  }
  catch (Exception)
  {
    // If we get here, the operation completed with an error
    //  before the cancellation took effect.
    throw;
  }
}
```

通常，设置 `CancellationTokenSource` 和执行取消操作是在不同的方法中完成的。一旦取消了 `CancellationTokenSource` 实例，它就会永久取消。如果您需要另一个源，必须创建另一个实例。以下代码是一个更实际的基于 GUI 的示例，使用一个按钮启动异步操作，另一个按钮取消它。它还禁用和启用 `StartButton` 和 `CancelButton`，以确保一次只能进行一个操作：

```cs
private CancellationTokenSource _cts;

private async void StartButton_Click(object sender, RoutedEventArgs e)
{
  StartButton.IsEnabled = false;
  CancelButton.IsEnabled = true;
  try
  {
    _cts = new CancellationTokenSource();
    CancellationToken token = _cts.Token;
    await Task.Delay(TimeSpan.FromSeconds(5), token);
    MessageBox.Show("Delay completed successfully.");
  }
  catch (OperationCanceledException)
  {
    MessageBox.Show("Delay was canceled.");
  }
  catch (Exception)
  {
    MessageBox.Show("Delay completed with error.");
    throw;
  }
  finally
  {
    StartButton.IsEnabled = true;
    CancelButton.IsEnabled = false;
  }
}

private void CancelButton_Click(object sender, RoutedEventArgs e)
{
  _cts.Cancel();
  CancelButton.IsEnabled = false;
}
```

## 讨论

本配方中最真实的示例使用了一个 GUI 应用程序，但不要认为取消仅适用于用户界面。取消在服务器上同样适用；例如，ASP.NET 提供了一个表示请求超时或客户端断开连接的取消令牌。在服务器端，取消令牌源可能更为罕见，但您仍然可以使用它们；如果需要取消某些 ASP.NET 取消范围之外的操作（例如请求处理的某部分的额外超时），这些令牌非常有用。

## 参见

Recipe 10.4 讲述了如何在 `async` 代码中传递令牌。

Recipe 10.5 讲述了如何在并行代码中传递令牌。

Recipe 10.6 讲述了如何在响应式代码中使用令牌。

Recipe 10.7 讲述了如何在数据流网络中传递令牌。

# 10.2 通过轮询响应取消请求

## 问题

您的代码中有一个需要支持取消操作的循环。

## 解决方案

当您的代码中有一个处理循环时，没有更低级别的 API 可以传递 `CancellationToken`。在这种情况下，您应该定期检查令牌是否已取消。以下代码在执行 CPU 绑定的循环时定期观察令牌：

```cs
public int CancelableMethod(CancellationToken cancellationToken)
{
  for (int i = 0; i != 100; ++i)
  {
    Thread.Sleep(1000); // Some calculation goes here.
    cancellationToken.ThrowIfCancellationRequested();
  }
  return 42;
}
```

如果你的循环非常紧凑（即，循环体执行非常快），那么你可能希望限制检查取消令牌的频率。如常，在进行此类更改之前和之后，请先测量性能，然后决定哪种方式最佳。以下代码与之前的示例类似，但循环迭代更多，因此我添加了对令牌检查频率的限制：

```cs
public int CancelableMethod(CancellationToken cancellationToken)
{
  for (int i = 0; i != 100000; ++i)
  {
    Thread.Sleep(1); // Some calculation goes here.
    if (i % 1000 == 0)
      cancellationToken.ThrowIfCancellationRequested();
  }
  return 42;
}
```

应该使用的适当限制完全取决于你正在执行的工作量和取消需求的响应速度。

## 讨论

大多数情况下，你的代码应该将 `CancellationToken` 直接传递给下一层。在配方 10.4、10.5、10.6 和 10.7 中有此类示例。本配方中的轮询技术只有在需要支持取消的处理循环时才应使用。

`CancellationToken` 上还有另一个成员叫做 `IsCancellationRequested`，当令牌被取消时开始返回 `true`。有些人使用此成员来响应取消，通常通过返回默认值或 `null`。我不建议大多数代码使用此方法。标准的取消模式是引发 `OperationCanceledException`，由 `ThrowIfCancellationRequested` 处理。如果调用堆栈上游的代码想要捕获异常并像结果是 `null` 一样处理，那么可以这样做，但是任何使用 `CancellationToken` 的代码都应该遵循标准的取消模式。如果你决定不遵循取消模式，请务必清楚地记录下来。

`ThrowIfCancellationRequested` 通过 *轮询* 取消令牌来工作；你的代码必须定期调用它。还有一种方法可以注册在请求取消时调用的回调函数。回调方法更多地是为了与其他取消系统进行交互；10.9 配方 讲解了在取消时使用回调的方法。

## 另请参阅

10.4 配方 讲解了将令牌传递给 `async` 代码。

10.5 配方 讲解了将令牌传递给并行代码的方法。

10.6 配方 讲解了在响应式代码中使用令牌的方法。

10.7 配方 讲解了将令牌传递给数据流网络的方法。

10.9 配方 讲解了使用回调而非轮询来响应取消请求。

10.1 配方 讲解了发出取消请求。

# 10.3 由于超时而取消

## 问题

你有一些代码需要在超时后停止运行。

## 解决方案

取消是超时情况的自然解决方案。超时只是取消请求的一种类型。需要取消的代码只需像处理任何其他取消请求一样观察取消令牌；它既不应该知道也不关心取消源是定时器。

还有一些方便的取消令牌源方法，它们基于计时器自动发出取消请求。您可以将超时传递给构造函数：

```cs
async Task IssueTimeoutAsync()
{
  using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
  CancellationToken token = cts.Token;
  await Task.Delay(TimeSpan.FromSeconds(10), token);
}
```

或者，如果您已经有一个`CancellationTokenSource`实例，您可以为该实例启动超时：

```cs
async Task IssueTimeoutAsync()
{
  using var cts = new CancellationTokenSource();
  CancellationToken token = cts.Token;
  cts.CancelAfter(TimeSpan.FromSeconds(5));
  await Task.Delay(TimeSpan.FromSeconds(10), token);
}
```

## 讨论

要使用超时执行代码，请使用`CancellationTokenSource`和`CancelAfter`（或构造函数）。还有其他方法可以做同样的事情，但使用现有的取消系统是最简单和最有效的选择。

记住，需要取消的代码需要观察取消令牌；不可能轻易取消不可取消的代码。

## 另请参阅

配方 10.4 涵盖了向`async`代码传递令牌。

配方 10.5 涵盖了向并行代码传递令牌。

配方 10.6 涵盖了在响应式代码中使用令牌。

配方 10.7 涵盖了向数据流网格传递令牌。

# 10.4 取消异步代码

## 问题

您正在使用`async`代码并且需要支持取消。

## 解决方案

在异步代码中支持取消的最简单方法是将`CancellationToken`直接传递给下一层。以下示例代码执行异步延迟，然后返回一个值；通过将令牌传递给`Task.Delay`来支持取消：

```cs
public async Task<int> CancelableMethodAsync(CancellationToken cancellationToken)
{
  await Task.Delay(TimeSpan.FromSeconds(2), cancellationToken);
  return 42;
}
```

许多异步 API 支持`CancellationToken`，因此自己启用取消通常只需要简单地获取一个令牌并传递它。作为一般规则，如果您的方法调用使用`CancellationToken`的 API，则您的方法也应该接受一个`CancellationToken`并将其传递给每个支持它的 API。

## 讨论

不幸的是，一些方法不支持取消。当您遇到这种情况时，没有简单的解决方案。除非将代码包装在单独的可执行文件中，否则无法安全地停止任意代码。如果您的代码调用不支持取消的代码，并且不想将该代码包装在单独的可执行文件中，您始终可以通过*假装*取消操作来选择忽略结果。

尽可能地提供取消选项是很重要的。这是因为在更高级别正确地取消依赖于更低级别的正确取消。因此，当您编写自己的`async`方法时，请尽量包含取消支持；您永远不知道哪个更高级别的方法将要调用您的方法，而它可能需要取消功能。

## 另请参阅

配方 10.1 涵盖了发出取消请求。

配方 10.3 涵盖了使用取消作为超时。

# 10.5 取消并行代码

## 问题

您正在使用并行代码并且需要支持取消。

## 解决方案

支持取消操作的最简单方法是通过`CancellationToken`传递给并行代码。`Parallel`方法通过接受`ParallelOptions`实例来支持此操作。您可以通过以下方式在`ParallelOptions`实例上设置`CancellationToken`：

```cs
void RotateMatrices(IEnumerable<Matrix> matrices, float degrees,
    CancellationToken token)
{
  Parallel.ForEach(matrices,
      new ParallelOptions { CancellationToken = token },
      matrix => matrix.Rotate(degrees));
}
```

或者，可以直接在循环体中观察`CancellationToken`：

```cs
void RotateMatrices2(IEnumerable<Matrix> matrices, float degrees,
    CancellationToken token)
{
  // Warning: not recommended; see below.
  Parallel.ForEach(matrices, matrix =>
  {
    matrix.Rotate(degrees);
    token.ThrowIfCancellationRequested();
  });
}
```

另一种方法工作量更大，并且不太灵活，因为并行循环会在`AggregateException`中包装`OperationCanceledException`。此外，如果将`CancellationToken`作为`ParallelOptions`实例的一部分传递，`Parallel`类可能会更智能地决定多久检查该令牌。因此，最好将令牌作为选项传递。如果将令牌作为选项传递，还可以将令牌传递给循环体，但不要仅仅将令牌传递给循环体。

并行 LINQ（PLINQ）还具有使用`WithCancellation`操作符的内置取消支持：

```cs
IEnumerable<int> MultiplyBy2(IEnumerable<int> values,
    CancellationToken cancellationToken)
{
  return values.AsParallel()
      .WithCancellation(cancellationToken)
      .Select(item => item * 2);
}
```

## 讨论

对于良好的用户体验，支持并行工作的取消很重要。如果您的应用程序正在进行并行工作，至少在短时间内会使用大量 CPU。即使不会干扰同一台机器上的其他应用程序，用户也会注意到高 CPU 使用率。因此，建议在进行并行计算（或任何其他 CPU 密集型工作）时支持取消操作，即使高 CPU 使用率的总时间不会非常长。

## 参见

配方 10.1 涵盖了发出取消请求。

# 10.6 取消 System.Reactive 代码

## 问题

您有一些响应式代码，需要使其支持取消。

## 解决方案

System.Reactive 库中有一个对可观察流的*订阅*概念。您的代码可以释放该订阅以取消对流的订阅。在许多情况下，这已足以逻辑上取消流。例如，以下代码在按下一个按钮时订阅鼠标点击事件，并在按下另一个按钮时取消订阅（取消订阅）：

```cs
private IDisposable _mouseMovesSubscription;

private void StartButton_Click(object sender, RoutedEventArgs e)
{
  IObservable<Point> mouseMoves = Observable
      .FromEventPattern<MouseEventHandler, MouseEventArgs>(
          handler => (s, a) => handler(s, a),
          handler => MouseMove += handler,
          handler => MouseMove -= handler)
      .Select(x => x.EventArgs.GetPosition(this));
  _mouseMovesSubscription = mouseMoves.Subscribe(value =>
  {
    MousePositionLabel.Content = "(" + value.X + ", " + value.Y + ")";
  });
}

private void CancelButton_Click(object sender, RoutedEventArgs e)
{
  if (_mouseMovesSubscription != null)
    _mouseMovesSubscription.Dispose();
}
```

使用所有其他部分用于取消的`CancellationTokenSource`/`CancellationToken`系统使 System.Reactive 与之一起工作非常方便。本配方的其余部分介绍了 System.Reactive 可观察对象如何与`CancellationToken`交互。

主要用例之一是将可观察代码包装在异步代码中。基本方法已在配方 8.5 中介绍过，现在您想要添加`CancellationToken`支持。一般来说，最简单的方法是使用响应式操作执行所有操作，然后调用`ToTask`将最后产生的元素转换为可等待任务。以下代码展示了如何异步获取序列中的最后一个元素：

```cs
CancellationToken cancellationToken = ...
IObservable<int> observable = ...
int lastElement = await observable.TakeLast(1).ToTask(cancellationToken);
// or: int lastElement = await observable.ToTask(cancellationToken);
```

取第一个元素非常类似；只需在调用`ToTask`之前修改可观察对象即可：

```cs
CancellationToken cancellationToken = ...
IObservable<int> observable = ...
int firstElement = await observable.Take(1).ToTask(cancellationToken);
```

将整个可观察序列异步转换为任务同样类似：

```cs
CancellationToken cancellationToken = ...
IObservable<int> observable = ...
IList<int> allElements = await observable.ToList().ToTask(cancellationToken);
```

最后，让我们考虑相反的情况。我们已经讨论了几种处理方式，这些方式在 System.Reactive 代码响应 `CancellationToken` — 即，`CancellationTokenSource` 取消请求被转换为该订阅的处置。也可以反过来：响应处置而发出取消请求。

`FromAsync`、`StartAsync` 和 `SelectMany` 运算符都支持取消，就像在 Recipe 8.6 中所示的那样。这些运算符涵盖了绝大多数的使用情况。Rx 还提供了一个 `CancellationDisposable` 类型，当其被处置时取消一个 `CancellationToken`。你可以直接使用 `CancellationDisposable`，就像这样：

```cs
using (var cancellation = new CancellationDisposable())
{
  CancellationToken token = cancellation.Token;
  // Pass the token to methods that respond to it.
}
// At this point, the token is canceled.
```

## 讨论

System.Reactive (Rx) 有其自己的取消概念：处理订阅的释放。本文介绍了如何使 Rx 在 .NET 4.0 引入的通用取消框架中良好运作的几种方式。只要您在代码的 Rx 部分，使用 Rx 订阅/释放系统；如果仅在边界引入 `CancellationToken` 支持，则更为清晰。

## 参见

Recipe 8.5 讲述了围绕 Rx 代码的异步包装（不带取消支持）。

Recipe 8.6 讲述了围绕异步代码的 Rx 包装（带取消支持）。

Recipe 10.1 讲述了发出取消请求的过程。

# 10.7 取消数据流网格

## 问题

您正在使用数据流网格，并且需要支持取消。

## 解决方案

在您的代码中支持取消的最佳方式是通过将 `CancellationToken` 传递给可取消的 API。数据流网格中的每个块都支持取消作为其 `DataflowBlockOptions` 的一部分。如果您想扩展您的自定义数据流块以支持取消，设置块选项的 `CancellationToken` 属性：

```cs
IPropagatorBlock<int, int> CreateMyCustomBlock(
    CancellationToken cancellationToken)
{
  var blockOptions = new ExecutionDataflowBlockOptions
  {
    CancellationToken = cancellationToken
  };
  var multiplyBlock = new TransformBlock<int, int>(item => item * 2,
      blockOptions);
  var addBlock = new TransformBlock<int, int>(item => item + 2,
      blockOptions);
  var divideBlock = new TransformBlock<int, int>(item => item / 2,
      blockOptions);

  var flowCompletion = new DataflowLinkOptions
  {
    PropagateCompletion = true
  };
  multiplyBlock.LinkTo(addBlock, flowCompletion);
  addBlock.LinkTo(divideBlock, flowCompletion);

  return DataflowBlock.Encapsulate(multiplyBlock, divideBlock);
}
```

在这个例子中，我在网格中的每个块上应用了`CancellationToken`，虽然这并非完全必要。因为我同时在链接中传播完成，我可以将其应用于第一个块并允许其传播。取消被认为是错误的一种特殊形式，因此管道中更深层的块将会因为这个错误而完成。话虽如此，如果我取消一个网格，我可能也会同时取消每个块，所以在这种情况下，我通常会设置每个块的`CancellationToken`选项。

## 讨论

在数据流网格中，取消*不是*刷新的一种形式。当取消一个块时，它会丢弃其所有的输入并拒绝接收任何新项。因此，如果在块运行时取消它，您*将*会丢失数据。

## 参见

Recipe 10.1 讲述了发出取消请求的过程。

# 10.8 注入取消请求

## 问题

您的代码层需要响应取消请求，并向下一层发出自己的取消请求。

## 解决方案

.NET 4.0 取消系统内置支持这种情况，称为*链接取消标记*。可以创建一个与一个（或多个）现有标记相关联的取消标记源。当创建链接取消标记源时，当任何现有标记被取消或链接源被显式取消时，结果标记就会被取消。

下面的代码执行异步 HTTP 请求。传递给`GetWithTimeoutAsync`方法的标记表示用户请求的取消，并且`GetWithTimeoutAsync`方法还为请求应用超时：

```cs
async Task<HttpResponseMessage> GetWithTimeoutAsync(HttpClient client,
    string url, CancellationToken cancellationToken)
{
  using CancellationTokenSource cts = CancellationTokenSource
      .CreateLinkedTokenSource(cancellationToken);
  cts.CancelAfter(TimeSpan.FromSeconds(2));
  CancellationToken combinedToken = cts.Token;

  return await client.GetAsync(url, combinedToken);
}
```

当用户取消现有的`cancellationToken`或链接源通过`CancelAfter`取消时，生成的`combinedToken`会被取消。

## 讨论

尽管前面的示例仅使用了单个`CancellationToken`源，但`CreateLinkedTokenSource`方法可以接受任意数量的取消标记作为参数。这使您可以从中实现逻辑取消的单个组合标记。例如，ASP.NET 提供了一个取消标记，表示用户断开连接（`HttpContext.RequestAborted`）；处理程序代码可以创建一个链接标记，响应用户断开连接或自己的取消原因，如超时。

要记住链接取消标记源的生命周期。前面的示例是通常的用例，其中一个或多个取消标记传递给方法，然后将它们链接在一起并作为组合标记传递。还要注意，示例代码使用了`using`语句，确保在操作完成时（并且不再使用组合标记时），链接的取消标记源会被处理。考虑一下，如果代码没有处理链接的取消标记源会发生什么：可能`GetWithTimeoutAsync`方法会多次使用相同的（长期存在的）现有标记调用，这种情况下，每次调用方法都会链接一个新的标记源。即使 HTTP 请求完成（而且没有任何使用组合标记的东西），链接的源仍然附加到现有的标记上。为了防止这种内存泄漏，请在不再需要组合标记时处理链接的取消标记源。

## 参见

食谱 10.1 概述了一般情况下发出取消请求的操作。

食谱 10.3 涵盖了使用取消作为超时的情况。

# 10.9 与其他取消系统的互操作

## 问题

您有一些带有自己取消观念的外部或遗留代码，并且希望使用标准的`CancellationToken`来控制它。

## 解决方案

`CancellationToken` 有两种主要方式来响应取消请求：轮询（在 第 10.2 节 中讨论）和回调（本节的主题）。轮询通常用于 CPU 绑定的代码，如数据处理循环；而回调通常用于其他所有场景。您可以使用 `CancellationToken.Register` 方法为令牌注册回调。

例如，假设您正在封装 `System.Net.NetworkInformation.Ping` 类型，并且希望能够取消 ping。`Ping` 类已经具有基于 `Task` 的 API，但不支持 `CancellationToken`。相反，`Ping` 类具有自己的 `SendAsyncCancel` 方法，您可以使用该方法取消 ping。为此，请注册一个调用该方法的回调：

```cs
async Task<PingReply> PingAsync(string hostNameOrAddress,
    CancellationToken cancellationToken)
{
  using var ping = new Ping();
  Task<PingReply> task = ping.SendPingAsync(hostNameOrAddress);
  using CancellationTokenRegistration _ = cancellationToken
      .Register(() => ping.SendAsyncCancel());
  return await task;
}
```

现在，当请求取消时，`CancellationToken` 将为您调用 `SendAsyncCancel` 方法，取消 `SendPingAsync` 方法。

## 讨论

`CancellationToken.Register` 方法可用于与任何类型的替代取消系统进行交互。但请注意，当方法接受 `CancellationToken` 时，取消请求应仅取消该操作。某些替代取消系统通过关闭某些资源来实现取消，这可能会取消多个操作；这种取消系统与 `CancellationToken` 不太匹配。如果决定将此类取消封装在 `CancellationToken` 中，则应记录其不寻常的取消语义。

要记住回调注册的生命周期。`Register` 方法返回一个可处置对象，在不再需要该回调时应予以处理。前面的示例代码使用 `using` 语句在异步操作完成时进行清理。如果代码没有该 `using` 语句，那么每次使用相同（长期存在的）`CancellationToken` 调用代码时，都会添加另一个回调（这反过来会使 `Ping` 对象保持活动状态）。为避免内存和资源泄漏，请在不再需要回调时处置回调注册。

## 参见

第 10.2 节 涵盖了通过轮询而非回调来响应取消令牌。

第 10.1 节 概述了一般的取消请求发出。
