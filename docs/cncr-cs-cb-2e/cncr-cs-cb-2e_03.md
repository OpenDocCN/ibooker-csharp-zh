# 第三章：异步流

*异步流*是一种异步接收多个数据项的方式。它们建立在*异步可枚举*(`IAsyncEnumerable<T>`)之上。异步可枚举是可枚举的异步版本；也就是说，它可以按需为消费者生成项目，并且每个项目可能是异步生成的。

我发现将异步流与可能更为熟悉的其他类型进行对比并考虑它们之间的差异非常有用。这帮助我记住何时使用异步流以及其他类型何时更合适。

# 异步流和 Task<T>

使用`Task<T>`的标准异步方法仅足以异步处理单个数据值。一旦给定的`Task<T>`完成，就没了；单个`Task<T>`无法为其消费者提供超过一个`T`值。即使`T`是一个集合，该值也只能提供一次。有关使用`async`与`Task<T>`的更多信息，请参阅“异步编程简介”和第二章。

将`Task<T>`与异步流进行比较时，异步流更类似于可枚举。具体而言，`IAsyncEnumerator<T>`可以逐个提供任意数量的`T`值。与`IEnumerator<T>`类似，`IAsyncEnumerator<T>`的长度可以是无限的。

# 异步流和 IEnumerable<T>

`IAsyncEnumerable<T>`，正如其名称所示，类似于`IEnumerable<T>`。也许这并不奇怪；它们都允许消费者逐个检索元素。不同之处在于名称：一个是异步的，另一个不是。

当您的代码遍历`IEnumerable<T>`时，它会阻塞，因为它从可枚举中检索每个元素。如果`IEnumerable<T>`代表某种 I/O 绑定操作，例如数据库查询或 API 调用，那么消费代码最终会阻塞在 I/O 上，这并不理想。`IAsyncEnumerable<T>`的工作方式与`IEnumerable<T>`完全相同，只是它异步检索每个下一个元素。

# 异步流和 Task<IEnumerable<T>>

完全可以异步返回一个包含多个项目的集合；一个常见的例子是`Task<List<T>>`。但是，返回`List<T>`的异步方法只有一个`return`语句；在返回之前，必须完全填充集合。甚至返回`Task<IEnumerable<T>>`的方法可能会异步返回一个可枚举，但然后该可枚举会同步评估。请考虑 LINQ-to-Entities 具有`ToListAsync` LINQ 方法，该方法返回`Task<List<T>>`。当 LINQ 提供程序执行此操作时，它必须与数据库通信并获取*所有*匹配的响应，然后才能完成填充列表并返回它。

`Task<IEnumerable<T>>` 的限制在于它不能在获取到项目时返回它们；如果返回一个集合，它必须将所有项目加载到内存中，填充集合，然后一次性返回整个集合。即使返回 LINQ 查询，它也可以异步地构建该查询，但一旦返回查询，每个项目都是同步检索的。`IAsyncEnumerable<T>` 也异步返回多个项目，但不同之处在于 `IAsyncEnumerable<T>` 可以对每个返回的项目进行异步操作。这是真正的异步项目流。

# 异步流和 `IObservable<T>`

观察者是异步流的真正概念；它们一次产生一个通知，支持真正的异步生产（无阻塞）。但 `IObservable<T>` 的消费模式与 `IAsyncEnumerable<T>` 完全不同。有关 `IObservable<T>` 的更多详细信息，请参见 第六章。

要消费 `IObservable<T>`，代码需要定义一个类似 LINQ 的查询，通过该查询将可观察通知流动起来，然后订阅可观察对象以开始流动。在处理可观察对象时，代码首先定义如何对传入通知做出 *反应*，然后才将它们打开（因此称为“响应式”）。相比之下，消费 `IAsyncEnumerable<T>` 与消费 `IEnumerable<T>` 非常相似，只是消费是异步的。

还存在背压问题；System.Reactive 中的所有通知都是同步的，因此一旦将一个项目通知发送给其订阅者，可观察对象将继续执行并检索下一个要发布的项目，可能会再次调用 API。如果消费代码是异步消费流（即在每个通知到达时执行某些异步操作），则可观察对象将超前于消费代码。

关于它们之间的区别的一种很好的思考方式是 `IObservable<T>` 是推送式的，而 `IAsyncEnumerable<T>` 是拉取式的。可观察流将向您的代码推送通知，但异步流会被动地让您的代码（异步地）从中拉取数据项。只有在消费代码请求下一个项目时，可观察流才会恢复执行。

# 总结

可能会有一个理论示例很有用。许多 API 都接受 `offset` 和 `limit` 参数以启用结果的分页。假设我们想要定义一个方法，从进行分页的 API 中检索结果，并且我们希望我们的方法处理分页，使得我们的高级方法不必处理它。

如果我们的方法返回 `Task<T>`，我们只能返回单个 `T`。对于调用 API 并返回其结果的单个调用来说这是可以接受的，但如果我们希望我们的方法多次调用 API，则这种返回类型效果不佳。

如果我们的方法返回`IEnumerable<T>`，我们可以创建一个循环，通过多次调用来分页 API 结果。每次方法调用 API 时，它会使用`yield return`返回该页的结果。只有在枚举继续时才需要进一步的 API 调用。不幸的是，返回`IEnumerable<T>`的方法无法是异步的，因此我们所有的 API 调用都被迫是同步的。

如果我们的方法返回`Task<List<T>>`，那么我们可以通过调用 API 异步分页的循环。然而，代码无法在获取响应时返回每个项目；它必须积累所有结果并一次性返回它们。

如果我们的方法返回`IObservable<T>`，我们可以使用`System.Reactive`来实现一个可观察的流，该流在订阅时开始请求，并在获取每个项目时发布。这种抽象是基于推送的；它给消费代码的表现形式是 API 结果被推送给它们，这样处理起来更加笨拙。`IObservable<T>`更适合接收和响应 WebSocket/SignalR 消息等场景。

如果我们的方法返回`IAsyncEnumerable<T>`，我们可以使用`await`和`yield return`创建一个真正的基于拉取的异步流。`IAsyncEnumerable<T>`是这种场景的天然选择。

表格 3-1 总结了常见类型的不同角色。

表格 3-1 类型分类

| 类型 | 单个或多个值 | 异步或同步 | 推送或拉取 |
| --- | --- | --- | --- |
| `T` | 单个值 | 同步 | 不适用 |
| `IEnumerable<T>` | 多个值 | 同步 | 不适用 |
| `Task<T>` | 单个值 | 异步 | 拉取 |
| `IAsyncEnumerable<T>` | 多个值 | 异步 | 拉取 |
| `IObservable<T>` | 单个或多个 | 异步 | 推送 |

###### 警告

当本书付梓时，.NET Core 3.0 仍处于测试版阶段，因此关于异步流的细节可能会有所变化。

# 3.1 创建异步流

## 问题

你需要返回多个值，每个值可能需要一些异步工作。这一点通常从以下两个路径之一达到：

+   你有多个值要返回（作为`IEnumerable<T>`），然后需要添加异步工作。

+   你有一个单一的异步返回（作为`Task<T>`），然后需要添加其他返回值。

## 解决方案

从方法返回多个值可以通过`yield return`实现，而异步方法则使用`async`和`await`。有了异步流，你可以结合这两者；只需使用返回类型`IAsyncEnumerable<T>`：

```cs
async IAsyncEnumerable<int> GetValuesAsync()
{
  await Task.Delay(1000); // some asynchronous work
  yield return 10;
  await Task.Delay(1000); // more asynchronous work
  yield return 13;
}
```

这个简单的例子说明了如何使用`await`与`yield return`创建异步流。

一个更真实的例子是异步枚举 API 所有使用分页参数的结果：

```cs
async IAsyncEnumerable<string> GetValuesAsync(HttpClient client)
{
  int offset = 0;
  const int limit = 10;
  while (true)
  {
    // Get the current page of results and parse them.
    string result = await client.GetStringAsync(
        $"https://example.com/api/values?offset={offset}&limit={limit}");
    string[] valuesOnThisPage = result.Split('\n');

    // Produce the results for this page.
    foreach (string value in valuesOnThisPage)
      yield return value;

    // If this is the last page, we're done.
    if (valuesOnThisPage.Length != limit)
      break;

    // Otherwise, proceed to the next page.
    offset += limit;
  }
}
```

当 `GetValuesAsync` 开始时，它会对第一页的数据进行异步请求，然后生成第一个元素。当请求第二个元素时，`GetValuesAsync` 会立即生成，因为它也在同一页的数据中。下一个元素也在该页中，依此类推，直到 10 个元素。然后，当请求第 11 个元素时，`valuesOnThisPage` 中的所有值都已生成，因此在第一页上没有更多的元素了。`GetValuesAsync` 将继续执行其 `while` 循环，转到下一页，进行第二页数据的异步请求，接收新的一批值，然后生成第 11 个元素。

## 讨论

自从引入 `async` 和 `await` 以来，用户一直在思考如何与 `yield return` 结合使用。多年来，这是不可能的，但是异步流现在已经将这一能力带到了 C# 和现代版本的 .NET 中。

在更现实的示例中，您可能会注意到只有部分结果需要进行异步处理。在示例中，页面长度为 10 时，大约每 10 个元素中只有 1 个需要进行异步处理。如果页面大小为 20，则每 20 个元素中只有 1 个需要异步处理。

这是异步流的一种常见模式。对于许多流来说，大多数异步迭代实际上是同步的；异步流只是允许以异步方式检索任何下一个项。异步流旨在同时考虑异步和同步代码；这就是为什么异步流建立在 `ValueTask<T>` 上的原因。通过在底层使用 `ValueTask<T>`，异步流最大化了其效率，无论是同步还是异步地检索项目。有关 `ValueTask<T>` 更多信息和适用场景，请参见 食谱 2.10。

当您实现异步流时，请考虑支持取消操作。有关异步流取消的详细讨论，请参见 食谱 3.4。有些情况下并不需要真正的取消；消费代码始终可以选择不检索下一个元素。如果没有取消的外部源，则这是一个完全可以接受的方法。如果您有一个异步流，在该流中希望取消异步流，即使在获取下一个元素时也要支持正确的取消操作，则应使用 `CancellationToken`。

## 参见

食谱 3.2 讨论了如何消费异步流。

食谱 3.4 讨论了如何处理异步流的取消。

食谱 2.10 更详细地介绍了 `ValueTask<T>` 的使用场景。

# 3.2 消费异步流

## 问题

你需要处理异步流的结果，也称为异步可枚举。

## 解决方案

通过`await`来消耗异步操作，通常通过`foreach`来消耗可枚举对象。将这两者结合到`await foreach`中来消耗异步可枚举对象。例如，给定一个异步可枚举对象，用于分页 API 响应，你可以消耗它并将每个元素写入控制台：

```cs
IAsyncEnumerable<string> GetValuesAsync(HttpClient client);

public async Task ProcessValueAsync(HttpClient client)
{
  await foreach (string value in GetValuesAsync(client))
  {
    Console.WriteLine(value);
  }
}
```

在这里发生的概念上，是调用`GetValuesAsync`，它返回一个`IAsyncEnumerable<T>`。然后`foreach`从该异步可枚举对象创建一个异步枚举器。异步枚举器在逻辑上类似于常规枚举器，只是它们的“获取下一个元素”的操作可能是异步的。因此，`await foreach`将等待下一个元素到达或异步枚举器完成。如果元素到达，`await foreach`将执行其循环体；如果异步枚举器完成，则循环将退出。

对每个元素进行异步处理也是很自然的：

```cs
IAsyncEnumerable<string> GetValuesAsync(HttpClient client);

public async Task ProcessValueAsync(HttpClient client)
{
  await foreach (string value in GetValuesAsync(client))
  {
    await Task.Delay(100); // asynchronous work
    Console.WriteLine(value);
  }
}
```

在这种情况下，`await foreach`不会在循环体完成之前继续下一个元素。因此，`await foreach`将异步接收第一个元素，然后异步执行该第一个元素的循环体，然后异步接收下一个元素，然后异步执行该下一个元素的循环体，依此类推。

在`await foreach`中隐藏了一个`await`：即“获取下一个元素”的操作被等待。通过使用`ConfigureAwait(false)`，你可以避免在常规`await`中捕获上下文，正如 2.7 节中所述。异步流还支持`ConfigureAwait(false)`，它传递给隐藏的`await`语句中使用的。

```cs
IAsyncEnumerable<string> GetValuesAsync(HttpClient client);

public async Task ProcessValueAsync(HttpClient client)
{
  await foreach (string value in GetValuesAsync(client).ConfigureAwait(false))
  {
    await Task.Delay(100).ConfigureAwait(false); // asynchronous work
    Console.WriteLine(value);
  }
}
```

## 讨论

`await foreach`是消耗异步流的最自然方式。语言支持`ConfigureAwait(false)`来避免在`await foreach`中的上下文。

可以传入取消令牌；由于异步流的复杂性较高，因此这是更高级的内容，你可以在 3.4 节中找到相关内容。

虽然使用`await foreach`来消耗异步流既可能又自然，但是也有大量的异步 LINQ 操作符可用；其中一些较受欢迎的操作符在 3.3 节中有所涵盖。

`await foreach`的主体可以是同步的，也可以是异步的。对于特定的异步示例，这对于在处理其他流抽象（如`IObservable<T>`）时要正确处理的事情要困难得多。这是因为可观察的订阅必须是同步的，但`await foreach`允许自然的异步处理。

`await foreach`生成一个`await`用于“获取下一个元素”的操作；它还生成一个`await`用于异步处理可枚举对象的释放。

## 参见

Recipe 3.1 涵盖了生成异步流。

Recipe 3.4 涵盖了处理异步流的取消。

Recipe 3.3 涵盖了异步流的常见 LINQ 方法。

Recipe 11.6 涵盖了异步处理。

# 3.3 使用 LINQ 处理异步流

## 问题

您希望使用经过良好定义和经过充分测试的操作符处理异步流。

## 解决方案

`IEnumerable<T>`具有 LINQ to Objects，而`IObservable<T>`具有 LINQ to Events。这两者都有定义操作符的扩展方法库，您可以使用这些方法构建查询。`IAsyncEnumerable<T>`也具有 LINQ 支持，由.NET 社区在`System.Linq.Async` NuGet 包中提供。

举个例子，关于 LINQ 的一个常见问题是如何在`Where`的谓词是异步的情况下使用`Where`操作符。换句话说，您希望根据一些异步条件过滤序列，例如，您需要查找数据库或 API 中的每个元素，以查看它是否应包含在结果序列中。`Where`无法处理异步条件，因为`Where`操作符要求其委托返回即时、同步的答案。

异步流有一个支持库，定义了许多有用的操作符。在下面的示例中，`WhereAwait`是正确的选择：

```cs
IAsyncEnumerable<int> values = SlowRange().WhereAwait(
    async value =>
    {
      // Do some asynchronous work to determine
      //  if this element should be included.
      await Task.Delay(10);
      return value % 2 == 0;
    });

await foreach (int result in values)
{
  Console.WriteLine(result);
}

// Produce sequence that slows down as it progresses.
async IAsyncEnumerable<int> SlowRange()
{
  for (int i = 0; i != 10; ++i)
  {
    await Task.Delay(i * 100);
    yield return i;
  }
}
```

用于异步流的 LINQ 操作符也包括同步版本；将同步的`Where`（或`Select`，或其他操作符）应用于异步流是有意义的。结果仍然是一个异步流：

```cs
IAsyncEnumerable<int> values = SlowRange().Where(
    value => value % 2 == 0);

await foreach (int result in values)
{
  Console.WriteLine(result);
}
```

您所有熟悉的 LINQ 操作符都在这里：`Where`、`Select`、`SelectMany`，甚至`Join`。现在大多数 LINQ 操作符也接受异步委托，就像上面的`WhereAwait`示例一样。

## 讨论

异步流是基于拉取的，因此没有像可观察对象那样的与时间相关的操作符。在这个世界中，`Throttle`和`Sample`没有意义，因为元素是按需从异步流中拉取出来的。

用于异步流的 LINQ 方法也对常规可枚举对象有用。如果您发现自己处于这种情况下，您可以在任何`IEnumerable<T>`上调用`ToAsyncEnumerable()`，然后您将拥有一个异步流接口，您可以使用`WhereAwait`、`SelectAwait`和其他支持异步委托的操作符。

在深入研究之前，有必要谈一下命名。本示例中使用`WhereAwait`作为`Where`的异步等价物。当您探索用于异步流的 LINQ 操作符时，您会发现一些以`Async`结尾，而另一些以`Await`结尾。以`Async`结尾的操作符返回一个可等待的对象；它们代表一个常规值，而不是一个异步序列。以`Await`结尾的操作符接受一个异步委托；它们名称中的`Await`意味着它们实际上在您传递给它们的委托上执行了一个`await`。

我们已经看过带有 `Await` 后缀的 `Where` 和 `WhereAwait` 的示例。`Async` 后缀仅适用于*终结操作符*——提取某些值或执行某些计算并返回异步标量值而不是异步序列的操作符。终结操作符的示例是 `CountAsync`，异步流版本的 `Count`，它可以计算与某些谓词匹配的元素数：

```cs
int count = await SlowRange().CountAsync(
    value => value % 2 == 0);
```

该谓词也可以是异步的，在这种情况下，您将使用 `CountAwaitAsync` 操作符，因为它既接受异步委托（它将`await`）又生成单个终端值，即计数：

```cs
int count = await SlowRange().CountAwaitAsync(
    async value =>
    {
      await Task.Delay(10);
      return value % 2 == 0;
    });
```

简而言之，可以接受委托的操作符有两个名称：一个带有 `Await` 后缀，一个没有。此外，返回终端值而不是异步流的操作符以 `Async` 结尾。如果一个操作符既接受异步委托，*又*返回终端值，则它具有这两个后缀。

###### 提示

用于异步流的 LINQ 操作符位于 NuGet 包 [`System.Linq.Async`](http://bit.ly/sys-linq-async) 中。还可以在 NuGet 包 [`System.Interactive.Async`](http://bit.ly/sys-int-async) 中找到用于异步流的其他 LINQ 操作符。

## 参见

配方 3.1 涵盖了生成异步流。

配方 3.2 涵盖了消费异步流。

# 3.4 异步流与取消

## 问题

您需要一种取消异步流的方法。

## 解决方案

并非所有的异步流都需要取消。当达到条件时，可以简单地停止枚举。如果这是唯一需要的“取消”，那么不需要真正的取消，就像以下示例所示：

```cs
await foreach (int result in SlowRange())
{
  Console.WriteLine(result);
  if (result >= 8)
    break;
}

// Produce sequence that slows down as it progresses.
async IAsyncEnumerable<int> SlowRange()
{
  for (int i = 0; i != 10; ++i)
  {
    await Task.Delay(i * 100);
    yield return i;
  }
}
```

话虽如此，取消异步流通常很有用，因为某些操作符将取消标记传递给它们的源流。在这种情况下，您希望使用 `CancellationToken` 来停止外部代码中的 `await foreach`。

返回 `IAsyncEnumerable<T>` 的 `async` 方法可以通过定义标记有 `EnumeratorCancellation` 属性的参数来接受取消令牌。然后可以自然地使用该令牌，通常是通过将其传递给其他接受取消令牌的 API，如下所示：

```cs
using var cts = new CancellationTokenSource(500);
CancellationToken token = cts.Token;
await foreach (int result in SlowRange(token))
{
  Console.WriteLine(result);
}

// Produce sequence that slows down as it progresses.
async IAsyncEnumerable<int> SlowRange(
 [EnumeratorCancellation] CancellationToken token = default)
{
  for (int i = 0; i != 10; ++i)
  {
    await Task.Delay(i * 100, token);
    yield return i;
  }
}
```

## 讨论

此处的示例解决方案直接将 `CancellationToken` 传递给返回异步枚举器的方法。这是最常见的用法。

还有其他情景，您的代码将获得一个异步枚举器，并希望对其使用`CancellationToken`。在启动可枚举对象的新枚举时使用取消令牌是有意义的。可枚举对象本身是通过`SlowRange`方法*定义*的，但直到被消费之前都不会启动。甚至有些情况下，应该为可枚举对象的不同枚举传递不同的取消令牌。

简言之，可枚举对象本身是不可取消的，但由此创建的枚举器是可以取消的。这是一个不常见但重要的用例，也是为什么异步流支持`WithCancellation`扩展方法，您可以使用它将`CancellationToken`附加到异步流的特定迭代中。

```cs
async Task ConsumeSequence(IAsyncEnumerable<int> items)
{
  using var cts = new CancellationTokenSource(500);
  CancellationToken token = cts.Token;
  await foreach (int result in items.WithCancellation(token))
  {
    Console.WriteLine(result);
  }
}

// Produce sequence that slows down as it progresses.
async IAsyncEnumerable<int> SlowRange(
 [EnumeratorCancellation] CancellationToken token = default)
{
  for (int i = 0; i != 10; ++i)
  {
    await Task.Delay(i * 100, token);
    yield return i;
  }
}

await ConsumeSequence(SlowRange());
```

通过`EnumeratorCancellation`参数属性，编译器负责将令牌从`WithCancellation`传递到标记为`EnumeratorCancellation`的`token`参数，现在取消请求会导致`await foreach`在处理了少量项目后引发`OperationCanceledException`。

`WithCancellation`扩展方法不会阻止`ConfigureAwait(false)`。这两个扩展方法可以链式使用：

```cs
async Task ConsumeSequence(IAsyncEnumerable<int> items)
{
  using var cts = new CancellationTokenSource(500);
  CancellationToken token = cts.Token;
  await foreach (int result in items
      .WithCancellation(token).ConfigureAwait(false))
  {
    Console.WriteLine(result);
  }
}
```

## 另请参阅

Recipe 3.1 讲述了生成异步流。

Recipe 3.2 讲述了消费异步流。

第十章 讲述了跨多种技术的协作取消。
