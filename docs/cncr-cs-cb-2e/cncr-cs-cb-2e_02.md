# 第二章：异步基础

本章介绍了使用 `async` 和 `await` 进行异步操作的基础知识。在这里，我们将只处理自然异步操作，例如 HTTP 请求、数据库命令和网络服务调用。

如果你有一个 CPU 密集型操作，希望将其视为异步操作（例如，以免阻塞 UI 线程），请参阅第四章和 Recipe 8.4。此外，本章仅处理启动一次并完成一次的操作；如果需要处理事件流，请参阅第三章[ch03.html#async-streams]和第六章[ch06.html#rx-basics]。

# 2.1 暂停一段时间

## 问题

你需要（异步地）等待一段时间。这是单元测试或实现重试延迟时常见的情况。编写简单超时时，也会遇到这种情况。

## 解决方案

`Task` 类型有一个静态方法 `Delay`，返回在指定时间后完成的任务。

下面的示例代码定义了一个完成异步的任务。在伪造异步操作时，测试同步成功、异步成功以及异步失败非常重要。以下示例返回用于异步成功案例的任务：

```cs
async Task<T> DelayResult<T>(T result, TimeSpan delay)
{
  await Task.Delay(delay);
  return result;
}
```

指数退避是一种策略，其中你增加重试之间的延迟。在使用网络服务时，请确保服务器不会被重试淹没。下一个示例是指数退避的简单实现：

```cs
async Task<string> DownloadStringWithRetries(HttpClient client, string uri)
{
  // Retry after 1 second, then after 2 seconds, then 4.
  TimeSpan nextDelay = TimeSpan.FromSeconds(1);
  for (int i = 0; i != 3; ++i)
  {
    try
    {
      return await client.GetStringAsync(uri);
    }
    catch
    {
    }

    await Task.Delay(nextDelay);
    nextDelay = nextDelay + nextDelay;
  }

  // Try one last time, allowing the error to propagate.
  return await client.GetStringAsync(uri);
}
```

###### 提示

对于生产代码，建议使用更彻底的解决方案，例如[`Polly`](http://www.thepollyproject.org/) NuGet 库；此代码只是演示了 `Task.Delay` 的使用。

你还可以将 `Task.Delay` 用作简单的超时。 `CancellationTokenSource` 是实现超时的常规类型（Recipe 10.3）。你可以在无限 `Task.Delay` 中包装一个取消标记，以提供在指定时间后取消的任务。最后，将该定时器任务与 `Task.WhenAny` 结合使用（Recipe 2.5）实现“软超时”。以下示例代码在服务在三秒内未响应时返回 `null`：

```cs
async Task<string> DownloadStringWithTimeout(HttpClient client, string uri)
{
  using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(3));
  Task<string> downloadTask = client.GetStringAsync(uri);
  Task timeoutTask = Task.Delay(Timeout.InfiniteTimeSpan, cts.Token);

  Task completedTask = await Task.WhenAny(downloadTask, timeoutTask);
  if (completedTask == timeoutTask)
    return null;
  return await downloadTask;
}
```

虽然可以使用 `Task.Delay` 作为“软超时”，但这种方法有局限性。如果操作超时，它不会被取消；在前面的示例中，下载任务将继续下载并在丢弃之前下载完整的响应。首选方法是使用取消标记作为超时，并直接将其传递给操作（在最后一个示例中的 `GetStringAsync`）。尽管如此，有时操作是不可取消的，在这种情况下，其他代码可能会使用 `Task.Delay` *模拟* 操作超时。

## 讨论

`Task.Delay`是用于单元测试异步代码或实现重试逻辑的良好选择。但是，如果需要实现超时，通常使用`CancellationToken`会更好。

## 参见

Recipe 2.5 讲解了如何使用`Task.WhenAny`来确定哪个任务首先完成。

Recipe 10.3 讲解了如何使用`CancellationToken`作为超时的示例。

# 2.2 返回已完成的任务

## 问题

你需要实现一个具有异步签名的同步方法。如果你从一个异步接口或基类继承但希望同步实现它，可能会遇到这种情况。这种技术在单元测试异步代码时特别有用，当你需要一个简单的存根或模拟异步接口时。

## 解决方案

你可以使用`Task.FromResult`创建并返回一个已经完成并带有指定值的新`Task<T>`：

```cs
interface IMyAsyncInterface
{
  Task<int> GetValueAsync();
}

class MySynchronousImplementation : IMyAsyncInterface
{
  public Task<int> GetValueAsync()
  {
    return Task.FromResult(13);
  }
}
```

对于没有返回值的方法，可以使用`Task.CompletedTask`，这是一个成功完成的缓存任务：

```cs
interface IMyAsyncInterface
{
  Task DoSomethingAsync();
}

class MySynchronousImplementation : IMyAsyncInterface
{
  public Task DoSomethingAsync()
  {
    return Task.CompletedTask;
  }
}
```

`Task.FromResult`仅为成功结果提供已完成的任务。如果需要具有不同类型结果（例如，使用`NotImplementedException`完成的任务），则可以使用`Task.FromException`：

```cs
Task<T> NotImplementedAsync<T>()
{
  return Task.FromException<T>(new NotImplementedException());
}
```

类似地，有一个`Task.FromCanceled`用于创建已从给定的`CancellationToken`取消的任务：

```cs
Task<int> GetValueAsync(CancellationToken cancellationToken)
{
  if (cancellationToken.IsCancellationRequested)
    return Task.FromCanceled<int>(cancellationToken);
  return Task.FromResult(13);
}
```

如果你的同步实现可能失败，那么应该捕获异常并使用`Task.FromException`将其返回，如下所示：

```cs
interface IMyAsyncInterface
{
  Task DoSomethingAsync();
}

class MySynchronousImplementation : IMyAsyncInterface
{
  public Task DoSomethingAsync()
  {
    try
    {
      DoSomethingSynchronously();
      return Task.CompletedTask;
    }
    catch (Exception ex)
    {
      return Task.FromException(ex);
    }
  }
}
```

## 讨论

如果你正在使用同步代码实现异步接口，请避免任何形式的阻塞。当方法可以异步实现时，异步方法阻塞然后返回完成的任务并不理想。举个反例，考虑一下.NET BCL 中的`Console`文本读取器。`Console.In.ReadLineAsync`会阻塞调用线程直到读取到一行，然后返回一个完成的任务。这种行为并不直观，已经让很多开发者感到意外。如果一个异步方法阻塞了，它会阻止调用线程启动其他任务，这会影响并发性甚至可能导致死锁。

如果你经常使用相同值的`Task.FromResult`，考虑缓存实际任务。例如，如果你创建一个带有零结果的`Task<int>`，那么你避免创建额外的实例，这些实例将需要被垃圾回收：

```cs
private static readonly Task<int> zeroTask = Task.FromResult(0);
Task<int> GetValueAsync()
{
  return zeroTask;
}
```

从逻辑上讲，`Task.FromResult`、`Task.FromException`和`Task.FromCanceled`都是通用的帮助方法和`TaskCompletionSource<T>`的快捷方式。`TaskCompletionSource<T>`是一个低级别类型，对于与其他形式的异步代码进行交互非常有用。一般情况下，如果要返回一个已经完成的任务，应该使用`Task.FromResult`等快捷方式。使用`TaskCompletionSource<T>`返回在将来某个时间完成的任务。

## 参见

第 7.1 节 讲述了如何对异步方法进行单元测试。

第 11.1 节 讲述了 `async` 方法的继承。

第 8.3 节 展示了如何利用 `TaskCompletionSource<T>` 进行与其他异步代码的通用交互。

# 2.3 报告进度

## 问题

您需要在操作执行时响应进度。

## 解决方案

使用提供的 `IProgress<T>` 和 `Progress<T>` 类型。您的 `async` 方法应接受一个 `IProgress<T>` 参数；`T` 是您需要报告的进度类型：

```cs
async Task MyMethodAsync(IProgress<double> progress = null)
{
  bool done = false;
  double percentComplete = 0;
  while (!done)
  {
    ...
    progress?.Report(percentComplete);
  }
}
```

调用代码可以如此使用：

```cs
async Task CallMyMethodAsync()
{
  var progress = new Progress<double>();
  progress.ProgressChanged += (sender, args) =>
  {
    ...
  };
  await MyMethodAsync(progress);
}
```

## 讨论

根据约定，如果调用者不需要进度报告，则 `IProgress<T>` 参数可能为 `null`，因此请务必在您的 `async` 方法中进行检查。

请记住，`IProgress<T>.Report` 方法通常是异步的。这意味着在报告进度之前，`MyMethodAsync` 可能会继续执行。因此，最好将 `T` 定义为*不可变类型*，或者至少是值类型。如果 `T` 是可变引用类型，则每次调用 `IProgress<T>.Report` 都必须手动创建一个单独的副本。

`Progress<T>` 在构造时会捕获当前上下文，并在该上下文中调用其回调函数。这意味着如果在 UI 线程上构造 `Progress<T>`，则可以从其回调函数更新 UI，即使异步方法从后台线程调用 `Report`。

当方法支持进度报告时，它也应尽力支持取消。

`IProgress<T>` 不仅适用于异步代码；长时间运行的同步代码中也应使用进度和取消。

## 参见

第 10.4 节 讲述了如何在异步方法中支持取消。

# 2.4 等待一组任务完成

## 问题

您有多个任务，需要等待它们全部完成。

## 解决方案

框架提供了 `Task.WhenAll` 方法来实现此目的。此方法接受多个任务，并返回一个在所有这些任务完成时完成的任务：

```cs
Task task1 = Task.Delay(TimeSpan.FromSeconds(1));
Task task2 = Task.Delay(TimeSpan.FromSeconds(2));
Task task3 = Task.Delay(TimeSpan.FromSeconds(1));

await Task.WhenAll(task1, task2, task3);
```

如果所有任务具有相同的结果类型，并且它们都成功完成，则 `Task.WhenAll` 任务将返回包含所有任务结果的数组：

```cs
Task<int> task1 = Task.FromResult(3);
Task<int> task2 = Task.FromResult(5);
Task<int> task3 = Task.FromResult(7);

int[] results = await Task.WhenAll(task1, task2, task3);

// "results" contains { 3, 5, 7 }
```

存在一个接受任务 `IEnumerable` 的 `Task.WhenAll` 的重载；然而，我不建议您使用它。每当我将异步代码与 LINQ 混合时，我发现在明确“实体化”序列（即评估序列，创建集合）时代码更清晰：

```cs
async Task<string> DownloadAllAsync(HttpClient client,
    IEnumerable<string> urls)
{
  // Define the action to do for each URL.
  var downloads = urls.Select(url => client.GetStringAsync(url));
  // Note that no tasks have actually started yet
  //  because the sequence is not evaluated.

  // Start all URLs downloading simultaneously.
  Task<string>[] downloadTasks = downloads.ToArray();
  // Now the tasks have all started.

  // Asynchronously wait for all downloads to complete.
  string[] htmlPages = await Task.WhenAll(downloadTasks);

  return string.Concat(htmlPages);
}
```

## 讨论

如果任何任务抛出异常，那么 `Task.WhenAll` 将使用该异常使其返回的任务失败。如果多个任务抛出异常，则所有这些异常都被放置在 `Task.WhenAll` 返回的任务上。然而，在等待该任务时，只会抛出其中一个异常。如果需要每个具体的异常，可以检查 `Task.WhenAll` 返回的 `Task` 上的 `Exception` 属性：

```cs
async Task ThrowNotImplementedExceptionAsync()
{
  throw new NotImplementedException();
}

async Task ThrowInvalidOperationExceptionAsync()
{
  throw new InvalidOperationException();
}

async Task ObserveOneExceptionAsync()
{
  var task1 = ThrowNotImplementedExceptionAsync();
  var task2 = ThrowInvalidOperationExceptionAsync();

  try
  {
    await Task.WhenAll(task1, task2);
  }
  catch (Exception ex)
  {
    // "ex" is either NotImplementedException or InvalidOperationException.
    ...
  }
}

async Task ObserveAllExceptionsAsync()
{
  var task1 = ThrowNotImplementedExceptionAsync();
  var task2 = ThrowInvalidOperationExceptionAsync();

  Task allTasks = Task.WhenAll(task1, task2);
  try
  {
    await allTasks;
  }
  catch
  {
    AggregateException allExceptions = allTasks.Exception;
    ...
  }
}
```

大多数情况下，当使用 `Task.WhenAll` 时，我*不*观察所有的异常。通常只需响应第一个抛出的错误即可，而不是所有的错误。

在前面的例子中，请注意，`ThrowNotImplementedExceptionAsync` 和 `ThrowInvalidOperationExceptionAsync` 方法不会直接抛出异常；它们使用了 `async` 关键字，因此它们的异常被捕获并放置在一个正常返回的任务上。这是返回可等待类型的方法的正常和预期行为。

## 参见

方案 2.5 涵盖了等待一组任务中的*任意一个*完成的方法。

方案 2.6 涵盖了等待一组任务完成并在每个任务完成后执行操作的方法。

方案 2.8 涵盖了对`async Task`方法进行异常处理的方法。

# 2.5 等待任意任务完成

## 问题

你有多个任务，并且只需响应其中一个完成的情况。当你有多个独立的操作尝试时，这种问题最常见，其中有一种“先到先得”的结构。例如，你可以同时从多个 Web 服务请求股票报价，但你只关心第一个响应的情况。

## 解决方案

使用 `Task.WhenAny` 方法。`Task.WhenAny` 方法接受一系列任务，并返回一个在任何任务完成时完成的任务。返回的任务的结果是首个完成的任务。如果这听起来让人困惑，不要担心；这是一种难以解释但通过代码更容易理解的情况：

```cs
// Returns the length of data at the first URL to respond.
async Task<int> FirstRespondingUrlAsync(HttpClient client,
    string urlA, string urlB)
{
  // Start both downloads concurrently.
  Task<byte[]> downloadTaskA = client.GetByteArrayAsync(urlA);
  Task<byte[]> downloadTaskB = client.GetByteArrayAsync(urlB);

  // Wait for either of the tasks to complete.
  Task<byte[]> completedTask =
      await Task.WhenAny(downloadTaskA, downloadTaskB);

  // Return the length of the data retrieved from that URL.
  byte[] data = await completedTask;
  return data.Length;
}
```

## 讨论

`Task.WhenAny` 返回的任务永远不会以故障或取消状态完成。这个“外部”任务始终成功完成，其结果值是第一个完成的 `Task`（即“内部”任务）。如果内部任务以异常完成，那么该异常不会传播到 `Task.WhenAny` 返回的外部任务。通常应在内部任务完成后 `await` 内部任务以确保观察到任何异常。

当第一个任务完成时，请考虑是否取消其余任务。如果其他任务没有被取消，但也从未被等待，那么它们就被放弃了。被放弃的任务会继续运行到完成，并且它们的结果将被忽略。这些被放弃的任务的任何异常也将被忽略。如果这些任务没有被取消，它们将继续运行并可能浪费资源，如 HTTP 连接、数据库连接或定时器。

可以使用 `Task.WhenAny` 来实现超时（例如，将 `Task.Delay` 作为其中一个任务），但并不推荐。使用取消来表达超时更为自然，并且取消还具有一个额外的好处，即如果超时则可以*取消*操作。

另一个反模式是对 `Task.WhenAny` 处理任务完成。起初，保持任务列表并在每个任务完成后从列表中移除似乎是合理的。但这种方法的问题在于它的执行时间为 O(N²)，而存在 O(N) 的算法。正确的 O(N) 算法在 Recipe 2.6 中进行了讨论。

## 另请参阅

Recipe 2.4 描述了异步等待集合中*所有*任务完成的方法。

Recipe 2.6 描述了等待集合中的任务全部完成并在每个任务完成时执行操作的方法。

Recipe 10.3 描述了使用取消令牌实现超时的方法。

# 2.6 在任务完成时处理任务

## 问题

你有一个任务集合需要等待，并且希望在每个任务完成后对其进行处理。但是，你希望在每个任务完成时立即进行处理，而不等待其他任何任务。

以下示例代码启动了三个延迟任务，然后等待每个任务完成：

```cs
async Task<int> DelayAndReturnAsync(int value)
{
  await Task.Delay(TimeSpan.FromSeconds(value));
  return value;
}

// Currently, this method prints "2", "3", and "1".
// The desired behavior is for this method to print "1", "2", and "3".
async Task ProcessTasksAsync()
{
  // Create a sequence of tasks.
  Task<int> taskA = DelayAndReturnAsync(2);
  Task<int> taskB = DelayAndReturnAsync(3);
  Task<int> taskC = DelayAndReturnAsync(1);
  Task<int>[] tasks = new[] { taskA, taskB, taskC };

  // Await each task in order.
  foreach (Task<int> task in tasks)
  {
    var result = await task;
    Trace.WriteLine(result);
  }
}
```

当前代码按顺序等待每个任务，尽管序列中的第三个任务最先完成。你希望代码在每个任务完成时执行处理（例如，`Trace.WriteLine`），而不等待其他任务。

## 解决方案

有几种不同的方法可以解决这个问题。在本配方中首先描述的方法是推荐的方法；另一种方法在“讨论”部分中有所描述。

最简单的解决方案是通过引入一个处理等待任务并处理其结果的更高级别 `async` 方法来重构代码。一旦将处理分离出来，代码就会显著简化：

```cs
async Task<int> DelayAndReturnAsync(int value)
{
  await Task.Delay(TimeSpan.FromSeconds(value));
  return value;
}

async Task AwaitAndProcessAsync(Task<int> task)
{
  int result = await task;
  Trace.WriteLine(result);
}

// This method now prints "1", "2", and "3".
async Task ProcessTasksAsync()
{
  // Create a sequence of tasks.
  Task<int> taskA = DelayAndReturnAsync(2);
  Task<int> taskB = DelayAndReturnAsync(3);
  Task<int> taskC = DelayAndReturnAsync(1);
  Task<int>[] tasks = new[] { taskA, taskB, taskC };

  IEnumerable<Task> taskQuery =
      from t in tasks select AwaitAndProcessAsync(t);
  Task[] processingTasks = taskQuery.ToArray();

  // Await all processing to complete
  await Task.WhenAll(processingTasks);
}
```

或者，可以像这样重写此代码：

```cs
async Task<int> DelayAndReturnAsync(int value)
{
  await Task.Delay(TimeSpan.FromSeconds(value));
  return value;
}

// This method now prints "1", "2", and "3".
async Task ProcessTasksAsync()
{
  // Create a sequence of tasks.
  Task<int> taskA = DelayAndReturnAsync(2);
  Task<int> taskB = DelayAndReturnAsync(3);
  Task<int> taskC = DelayAndReturnAsync(1);
  Task<int>[] tasks = new[] { taskA, taskB, taskC };

  Task[] processingTasks = tasks.Select(async t =>
  {
    var result = await t;
    Trace.WriteLine(result);
  }).ToArray();

  // Await all processing to complete
  await Task.WhenAll(processingTasks);
}
```

所示的重构是解决此问题最清晰和最便携的方式。请注意，它与原始代码略有不同。此解决方案将会并发执行任务处理，而原始代码将会逐个执行任务处理。通常这不是问题，但如果对您的情况不可接受，请考虑使用锁定（Recipe 12.2）或以下替代解决方案。

## 讨论

如果重构不是一种可接受的解决方案，那么还有一个替代方案。Stephen Toub 和 Jon Skeet 都开发了一个扩展方法，返回一个按顺序完成的任务数组。Stephen Toub 的解决方案可在 [.NET 并行编程博客](http://bit.ly/toub-task) 上找到，Jon Skeet 的解决方案可在 [他的编程博客](http://bit.ly/skeet_blog) 上找到。

###### 提示

`OrderByCompletion` 扩展方法也可以在开源 [`AsyncEx` 库](https://github.com/StephenCleary/AsyncEx) 中找到，在 [`Nito.AsyncEx` NuGet 包](http://bit.ly/nito-async) 中也是如此。

使用像 `OrderByCompletion` 这样的扩展方法可以尽量减少对原始代码的更改：

```cs
async Task<int> DelayAndReturnAsync(int value)
{
  await Task.Delay(TimeSpan.FromSeconds(value));
  return value;
}

// This method now prints "1", "2", and "3".
async Task UseOrderByCompletionAsync()
{
  // Create a sequence of tasks.
  Task<int> taskA = DelayAndReturnAsync(2);
  Task<int> taskB = DelayAndReturnAsync(3);
  Task<int> taskC = DelayAndReturnAsync(1);
  Task<int>[] tasks = new[] { taskA, taskB, taskC };

  // Await each one as they complete.
  foreach (Task<int> task in tasks.OrderByCompletion())
  {
    int result = await task;
    Trace.WriteLine(result);
  }
}
```

## 参见

Recipe 2.4 讲述了异步等待一系列任务完成。

# 2.7 避免为继续执行设置上下文

## 问题

当一个 `async` 方法在一个 `await` 后恢复时，默认情况下它会在相同的上下文中继续执行。如果该上下文是 UI 上下文，并且有大量的 `async` 方法在 UI 上下文中恢复执行，这可能会导致性能问题。

## 解决方案

要避免在上下文中继续执行，在 `ConfigureAwait` 的结果上 `await` 并传递 `false` 给它的 `continueOnCapturedContext` 参数：

```cs
async Task ResumeOnContextAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1));

  // This method resumes within the same context.
}

async Task ResumeWithoutContextAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);

  // This method discards its context when it resumes.
}
```

## 讨论

在 UI 线程上运行太多的继续执行会导致性能问题。这种性能问题很难诊断，因为不是单个方法导致系统变慢。相反，随着应用程序变得更加复杂，UI 性能开始遭受“成千上万次的纸疤”。

真正的问题是，*在* UI 线程上的 *多少* 个继续执行是*太多*？没有明确的答案，但微软的 Lucian Wischik [公布了指南](http://bit.ly/new-async)，用于 Universal Windows 团队：每秒大约一百个左右是可以接受的，但每秒约一千个就太多了。

最好在一开始就避免这个问题。对于每个你编写的 `async` 方法，如果它不需要*恢复*到原始上下文，那么使用 `ConfigureAwait`。这样做没有任何不利之处。

写 `async` 代码时，注意上下文也是一个好主意。通常，一个 `async` 方法应*要么*需要上下文（处理 UI 元素或 ASP.NET 请求/响应），*要么*不需要上下文（执行后台操作）。如果你有一个 `async` 方法，其中一部分需要上下文，另一部分不需要上下文，考虑将其拆分为两个（或更多） `async` 方法。这种方法有助于更好地将你的代码组织成层次。

## 参见

第一章 讲述了异步编程的简介。

# 2.8 异步任务方法中的异常处理

## 问题

异常处理是任何设计中的关键部分。设计能够处理成功情况是容易的，但直到它也能处理失败情况，设计才是正确的。幸运的是，处理 `async Task` 方法的异常是直接的。

## 解决方案

异常可以通过简单的 `try/catch` 捕获，就像你为同步代码所做的那样：

```cs
async Task ThrowExceptionAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1));
  throw new InvalidOperationException("Test");
}

async Task TestAsync()
{
  try
  {
    await ThrowExceptionAsync();
  }
  catch (InvalidOperationException)
  {
  }
}
```

从 `async Task` 方法中引发的异常会被放置在返回的 `Task` 上。只有在等待返回的 `Task` 时才会引发它们：

```cs
async Task ThrowExceptionAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1));
  throw new InvalidOperationException("Test");
}

async Task TestAsync()
{
  // The exception is thrown by the method and placed on the task.
  Task task = ThrowExceptionAsync();
  try
  {
    // The exception is re-raised here, where the task is awaited.
    await task;
  }
  catch (InvalidOperationException)
  {
    // The exception is correctly caught here.
  }
}
```

## 讨论

当从 `async Task` 方法中抛出异常时，该异常会被捕获并放置在返回的 `Task` 上。由于 `async void` 方法没有 `Task` 可以放置其异常，它们的行为会有所不同；如何捕获 `async void` 方法中的异常在 2.9 节 中有所涉及。

当您 `await` 一个故障的 `Task` 时，该任务上的第一个异常将被重新抛出。如果您熟悉重新抛出异常的问题，您可能会想到堆栈跟踪。请放心：当异常被重新抛出时，原始堆栈跟踪会被正确保留。

这种设置听起来有些复杂，但所有这些复杂性都是为了让简单的场景有简单的代码。大多数情况下，您的代码应该从它调用的异步方法中传播异常；它所需做的只是 `await` 来自该异步方法的返回任务，异常将会自然传播。

有些情况下（例如 `Task.WhenAll`），一个 `Task` 可能有多个异常，而 `await` 只会重新抛出第一个异常。参见 2.4 节，以查看处理所有异常的示例。

## 参见

2.4 节 讲述了等待多个任务的方法。

2.9 节 讲述了从 `async void` 方法捕获异常的技术。

7.2 节 讲述了从 `async Task` 方法抛出异常的单元测试。

# 2.9 处理异步 `void` 方法的异常

## 问题

您有一个 `async void` 方法，并且需要处理从该方法传播出的异常。

## 解决方案

没有很好的解决方案。如果可能的话，请将方法更改为返回 `Task` 而不是 `void`。在某些情况下，这样做是不可能的；例如，假设您需要对 `ICommand` 实现进行单元测试（该方法 *必须* 返回 `void`）。在这种情况下，您可以为 `Execute` 方法提供一个返回 `Task` 的重载：

```cs
sealed class MyAsyncCommand : ICommand
{
  async void ICommand.Execute(object parameter)
  {
    await Execute(parameter);
  }

  public async Task Execute(object parameter)
  {
    ... // Asynchronous command implementation goes here.
  }

  ... // Other members (CanExecute, etc.)
}
```

最好避免从 `async void` 方法中传播异常。如果必须使用 `async void` 方法，请考虑将其所有代码都包装在 `try` 块中，并直接处理异常。

处理 `async void` 方法异常的另一种解决方案是，当 `async void` 方法传播异常时，该异常会在该方法开始执行时处于活动状态的 `SynchronizationContext` 上引发。如果您的执行环境提供了 `SynchronizationContext`，则通常可以在全局范围内处理这些顶级异常。例如，WPF 具有 `Application.DispatcherUnhandledException`，Universal Windows 具有 `Application.UnhandledException`，而 ASP.NET 则具有 `UseExceptionHandler` 中间件。

也可以通过控制`SynchronizationContext`来处理`async void`方法的异常。编写自己的`SynchronizationContext`并不容易，但可以使用免费的`Nito.AsyncEx` NuGet 助手库中的`AsyncContext`类型。`AsyncContext`对于没有内置`SynchronizationContext`的应用程序特别有用，如控制台应用程序和 Win32 服务。下一个示例使用`AsyncContext`来运行并处理`async void`方法中的异常：

```cs
static class Program
{
  static void Main(string[] args)
  {
    try
    {
      AsyncContext.Run(() => MainAsync(args));
    }
    catch (Exception ex)
    {
      Console.Error.WriteLine(ex);
    }
  }

  // BAD CODE!!!
  // In the real world, do not use async void unless you have to.
  static async void MainAsync(string[] args)
  {
    ...
  }
}
```

## 讨论

更倾向于使用`async Task`而不是`async void`的一个原因是，返回`Task`的方法更容易进行测试。至少，通过使用返回`Task`的方法重载返回`void`的方法，可以得到一个可测试的 API 表面。

如果确实需要提供自己的`SynchronizationContext`类型（例如`AsyncContext`），请确保不要将该`SynchronizationContext`安装在不属于您的任何线程上。一般规则是，不应将此类型放置在已经具有`SynchronizationContext`的任何线程上（如 UI 或 ASP.NET 经典请求线程）；也不应将`SynchronizationContext`放置在线程池线程上。控制台应用程序的主线程属于您，您手动创建的任何线程也属于您。

###### 提示

`AsyncContext`类型位于[`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中。

## 参见

2.8 菜谱涵盖了使用`async Task`方法进行异常处理。

7.3 菜谱涵盖了对`async void`方法进行单元测试。

# 2.10 创建一个 ValueTask

## 问题

您需要实现一个返回`ValueTask<T>`的方法。

## 解决方案

`ValueTask<T>`在通常存在同步结果且需要返回异步行为的情况下作为返回类型使用。作为一般规则，对于您自己的应用程序代码，应使用`Task<T>`作为返回类型，而不是`ValueTask<T>`。仅在分析显示您可以获得性能提升时，才考虑在您自己的应用程序中使用`ValueTask<T>`作为返回类型。尽管如此，确实有需要实现返回`ValueTask<T>`的情况。一个这样的情况是`IAsyncDisposable`，其`DisposeAsync`方法返回`ValueTask`。参见 11.6 菜谱以获取关于异步处理的更详细讨论。

实现返回`ValueTask<T>`的方法最简单的方式是像普通的`async`方法一样使用`async`和`await`：

```cs
public async ValueTask<int> MethodAsync()
{
  await Task.Delay(100); // asynchronous work.
  return 13;
}
```

许多情况下，返回`ValueTask<T>`的方法能够立即返回值；在这种情况下，您可以使用`ValueTask<T>`构造函数进行优化，然后仅在必要时转发到慢速异步方法：

```cs
public ValueTask<int> MethodAsync()
{
  if (CanBehaveSynchronously)
    return new ValueTask<int>(13);
  return new ValueTask<int>(SlowMethodAsync());
}

private Task<int> SlowMethodAsync();
```

对于非泛型的 `ValueTask`，也可以采用类似的方法。在这里，使用 `ValueTask` 的默认构造函数返回一个成功完成的 `ValueTask`。下面的示例展示了一个只运行其异步处理逻辑一次的 `IAsyncDisposable` 实现；在后续调用中，`DisposeAsync` 方法会成功且同步地完成：

```cs
private Func<Task> _disposeLogic;

public ValueTask DisposeAsync()
{
  if (_disposeLogic == null)
    return default;

  // Note: this simple example is not threadsafe;
  //  if multiple threads call DisposeAsync,
  //  the logic could run more than once.
  Func<Task> logic = _disposeLogic;
  _disposeLogic = null;
  return new ValueTask(logic());
}
```

## 讨论

大多数方法应返回 `Task<T>`，因为消耗 `Task<T>` 比消耗 `ValueTask<T>` 更少出错。详见 2.11 消耗 ValueTask 的详细信息。

大多数情况下，如果你只是实现使用 `ValueTask` 或 `ValueTask<T>` 的接口，那么可以简单地使用 `async` 和 `await`。更高级的实现是为了当你自己使用 `ValueTask<T>` 时。

本文涵盖的方法是创建 `ValueTask<T>` 和 `ValueTask` 实例的更简单和更常见的方法。还有一种更适合更高级场景的方法，当你需要绝对最小化使用的分配时。这种更高级的方法允许你缓存或池化 `IValueTaskSource<T>` 实现，并在多个异步方法调用中重复使用它。要开始使用高级场景，请参阅 [Microsoft docs 上的 `ManualResetValueTaskSourceCore<T>` 类型](http://bit.ly/man-reset-type-doc)。

## 另请参阅

2.11 使用 ValueTask 的限制 讨论了消耗 `ValueTask<T>` 和 `ValueTask` 类型的限制。

11.6 异步处理 讨论了异步处理。

# 2.11 消耗 ValueTask

## 问题

你需要消耗 `ValueTask<T>` 的值。

## 解决方案

使用 `await` 是消耗 `ValueTask<T>` 或 `ValueTask` 值最简单和常见的方法。大部分情况下，这就是你需要做的：

```cs
ValueTask<int> MethodAsync();

async Task ConsumingMethodAsync()
{
  int value = await MethodAsync();
}
```

在执行并发操作后，也可以执行 `await` 操作，就像处理 `Task<T>` 一样：

```cs
ValueTask<int> MethodAsync();

async Task ConsumingMethodAsync()
{
  ValueTask<int> valueTask = MethodAsync();
  ... // other concurrent work
  int value = await valueTask;
}
```

这两种方法都适合，因为 `ValueTask` 只被等待了一次。这是 `ValueTask` 的限制之一。

###### 警告

只能一次性等待 `ValueTask` 或 `ValueTask<T>`。

要执行更复杂的操作，请通过调用 `AsTask` 将 `ValueTask<T>` 转换为 `Task<T>`：

```cs
ValueTask<int> MethodAsync();

async Task ConsumingMethodAsync()
{
  Task<int> task = MethodAsync().AsTask();
  ... // other concurrent work
  int value = await task;
  int anotherValue = await task;
}
```

可以放心地多次 `await` 一个 `Task<T>`。你也可以做其他事情，比如异步等待多个操作完成（参见 2.4 异步等待多个操作的方法）：

```cs
ValueTask<int> MethodAsync();

async Task ConsumingMethodAsync()
{
  Task<int> task1 = MethodAsync().AsTask();
  Task<int> task2 = MethodAsync().AsTask();
  int[] results = await Task.WhenAll(task1, task2);
}
```

然而，对于每个 `ValueTask<T>`，只能调用一次 `AsTask`。通常的方法是立即将其转换为 `Task<T>`，然后忽略 `ValueTask<T>`。还要注意，不能同时 `await` 和调用 `AsTask` 同一个 `ValueTask<T>`。

大多数情况下，代码应该立即 `await` 一个 `ValueTask<T>` 或将其转换为 `Task<T>`。

## 讨论

`ValueTask<T>` 的其他属性适用于更高级的用法。它们与你可能熟悉的其他属性不太相似；特别是，`ValueTask<T>.Result` 比 `Task<T>.Result` 有更多限制。从 `ValueTask<T>` 同步检索结果的代码可以调用 `ValueTask<T>.Result` 或 `ValueTask<T>.GetAwaiter().GetResult()`，但这些成员不能在 `ValueTask<T>` 完成之前调用。从 `Task<T>` 同步获取结果会阻塞调用线程，直到任务完成；`ValueTask<T>` 不提供这样的保证。

###### 警告

从 `ValueTask` 或 `ValueTask<T>` 同步获取结果只能在 `ValueTask` 完成后进行一次，并且不能再等待或转换为任务。

为了避免重复，当你的代码调用返回 `ValueTask` 或 `ValueTask<T>` 的方法时，应立即 `await` 这个 `ValueTask` 或立即调用 `AsTask` 将其转换为 `Task`。这个简单的准则虽然不能涵盖所有高级场景，但大多数应用程序不会需要更多。

## 参见

示例 2.10 讲述了如何从你的方法中返回 `ValueTask<T>` 和 `ValueTask` 类型的值。

2.4 和 2.5 的示例介绍了同时等待多个任务的方法。
