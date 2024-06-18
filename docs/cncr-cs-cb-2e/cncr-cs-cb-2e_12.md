# 第十二章 同步

当您的应用程序使用并发（几乎所有.NET 应用程序都会这样）时，您需要注意一种情况：一段代码需要更新数据，同时其他代码需要访问相同的数据。每当发生这种情况时，您都需要*同步*对数据的访问。本章的配方涵盖了用于同步访问的最常见类型。但是，如果您适当地使用本书中的其他配方，您会发现许多常见的同步已经由相应的库为您完成。在深入研究同步配方之前，让我们更详细地看看可能需要或不需要同步的一些常见情况。

###### 提示

本节中的同步解释略有简化，但结论都是正确的。

同步有两种主要类型：*通信*和*数据保护*。当一段代码需要通知另一段代码某些条件（例如，新消息已到达）时使用通信。我将在本章的配方中更详细地讨论通信；本介绍的其余部分讨论数据保护。

当*以下所有三个条件*都为真时，您需要使用同步来保护共享数据：

+   多段代码同时运行。

+   这些段代码正在访问（读取或写入）相同的数据。

+   至少有一段代码正在更新（写入）数据。

第一个条件的原因显而易见；如果您的整个代码从头到尾顺序运行，并且从不并发发生，那么您永远不必担心同步。这对一些简单的控制台应用程序来说是成立的，但绝大多数.NET 应用程序确实会使用*某种*并发。第二个条件意味着，如果每段代码都有自己的本地数据，而它们不*共享*，则无需同步；本地数据从未从任何其他代码访问。如果存在共享数据但数据永远不会更改，比如使用不可变类型定义数据，也不需要同步。第三个条件涵盖的是像配置值之类的在应用程序开始时设置然后从不更改的情况。如果仅读取共享数据，则不需要同步；只有*共享*且*更新*的数据需要同步。

数据保护的目的是为每段代码提供数据的一致视图。如果一段代码正在更新数据，那么您可以使用同步使这些更新对系统的其他部分看起来是原子的。

学习何时需要同步需要一些实践，因此在开始本章的配方之前，我们将通过几个例子来讲解。首先，考虑以下代码：

```cs
async Task MyMethodAsync()
{
  int value = 10;
  await Task.Delay(TimeSpan.FromSeconds(1));
  value = value + 1;
  await Task.Delay(TimeSpan.FromSeconds(1));
  value = value - 1;
  await Task.Delay(TimeSpan.FromSeconds(1));
  Trace.WriteLine(value);
}
```

如果`MyMethodAsync`方法是从线程池线程调用的（例如，在`Task.Run`内部），那么访问`value`的代码行可能会在不同的线程池线程上运行。但是它是否需要同步？不需要，因为它们都不可能同时运行。该方法是异步的，但也是顺序执行的（意味着它一次进行一个部分的进展）。

好的，让我们稍微复杂化这个例子。这次我们将运行并发的异步代码：

```cs
private int value;

async Task ModifyValueAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1));
  value = value + 1;
}

// WARNING: may require synchronization; see discussion below.
async Task<int> ModifyValueConcurrentlyAsync()
{
  // Start three concurrent modifications.
  Task task1 = ModifyValueAsync();
  Task task2 = ModifyValueAsync();
  Task task3 = ModifyValueAsync();

  await Task.WhenAll(task1, task2, task3);

  return value;
}
```

上述代码正在启动三个并发运行的修改。它是否需要同步？这要看情况。如果你知道该方法是从 GUI 或 ASP.NET 上下文（或任何只允许一段代码同时运行的上下文）调用的，那么不需要同步，因为实际的`data`修改代码运行时，它会在不同的时间运行于其他两个`data`修改之外。例如，如果前面的代码在 GUI 上下文中运行，那么只有一个 UI 线程会执行每个`data`修改，因此*必须*一个接一个地执行它们。因此，如果你知道上下文是一次一个的上下文，那么就不需要同步。然而，如果相同的方法是从线程池线程（例如，从`Task.Run`）调用的，那么就需要同步。在这种情况下，三个`data`修改可以在不同的线程池线程上运行并同时更新`data.Value`，因此你需要同步访问`data.Value`。

现在让我们考虑另一个问题：

```cs
private int value;

async Task ModifyValueAsync()
{
  int originalValue = value;
  await Task.Delay(TimeSpan.FromSeconds(1));
  value = originalValue + 1;
}
```

考虑如果`ModifyValueAsync`被同时调用多次会发生什么。即使它是从一个一次一个的上下文中调用的，数据成员在每次`ModifyValueAsync`调用之间是共享的，并且当该方法执行`await`时，值可能随时更改。如果你想要避免这种共享，即使在一次一个的上下文中，你也可能需要应用同步。换句话说，为了确保每次调用`ModifyValueAsync`都等到所有先前的调用完成，你需要添加同步。即使上下文确保所有代码只使用一个线程（比如 UI 线程），在这种情况下也是如此。在这种情况下，同步是异步方法的一种*限流*方式（参见 Recipe 12.2）。

让我们再看一个`async`示例。你可以使用`Task.Run`来执行我称之为“简单并行处理”的操作——这是一种基本的并行处理方式，不提供`Parallel`/PLINQ 真正并行处理所具有的效率和可配置性。以下代码使用简单并行处理更新一个共享值：

```cs
// BAD CODE!!
async Task<int> SimpleParallelismAsync()
{
  int value = 0;
  Task task1 = Task.Run(() => { value = value + 1; });
  Task task2 = Task.Run(() => { value = value + 1; });
  Task task3 = Task.Run(() => { value = value + 1; });
  await Task.WhenAll(task1, task2, task3);
  return value;
}
```

这段代码在线程池上运行了三个单独的任务（通过`Task.Run`），它们都修改同一个`value`。因此，我们的同步条件适用，并且在这里确实需要同步。请注意，尽管`value`是一个局部变量，我们仍然需要同步；尽管它是局部于一个方法，但它仍然在多个线程之间*共享*。

转向真正的并行代码，让我们考虑一个使用`Parallel`类型的示例：

```cs
void IndependentParallelism(IEnumerable<int> values)
{
  Parallel.ForEach(values, item => Trace.WriteLine(item));
}
```

由于这段代码使用了`Parallel`，我们必须假设并行循环的主体(`item => Trace.WriteLine(item)`)可能在多个线程上运行。然而，循环的主体只从自己的数据中读取；这里没有线程之间的数据共享。`Parallel`类将数据分配给线程，以便它们中的任何一个不必共享其数据。每个运行其循环主体的线程都与运行相同循环主体的所有其他线程是独立的。因此，前述代码不需要同步。

让我们看一个聚合示例，类似于食谱 4.2 中涵盖的示例：

```cs
// BAD CODE!!
int ParallelSum(IEnumerable<int> values)
{
  int result = 0;
  Parallel.ForEach(source: values,
      localInit: () => 0,
      body: (item, state, localValue) => localValue + item,
      localFinally: localValue => { result += localValue; });
  return result;
}
```

在这个例子中，代码再次使用多个线程；这次，每个线程从其本地值初始化为 0 开始(`() => 0`)，并且对于每个线程处理的输入值，它将输入值添加到其本地值中(`(item, state, localValue) => localValue + item`)。最后，所有本地值都添加到返回值中(`localValue => { result += localValue; }`)。前两个步骤没有问题，因为没有线程之间共享的内容；每个线程的本地和输入值与所有其他线程的本地和输入值是独立的。然而，最后一步是有问题的；当每个线程的本地值添加到返回值时，这是一个共享变量(`result`)被多个线程访问并更新的情况。因此，在最后一步中，您需要使用同步（参见食谱 12.1）。

PLINQ、数据流和响应式库与`Parallel`示例非常相似：只要您的代码只处理自己的输入，就无需担心同步。我发现如果我适当地使用这些库，大多数代码几乎不需要我添加同步。

最后，让我们讨论集合。请记住，需要同步的三个条件是*多段代码*，*共享数据*和*数据更新*。

不可变类型自然是线程安全的，因为它们*无法*更改；不可能更新不可变集合，因此不需要同步。例如，以下代码不需要同步，因为每个单独的线程池线程将值推送到堆栈时，它正在创建一个新的具有该值的不可变堆栈，不会改变原始的`stack`：

```cs
async Task<bool> PlayWithStackAsync()
{
  ImmutableStack<int> stack = ImmutableStack<int>.Empty;

  Task task1 = Task.Run(() => Trace.WriteLine(stack.Push(3).Peek()));
  Task task2 = Task.Run(() => Trace.WriteLine(stack.Push(5).Peek()));
  Task task3 = Task.Run(() => Trace.WriteLine(stack.Push(7).Peek()));
  await Task.WhenAll(task1, task2, task3);

  return stack.IsEmpty; // Always returns true.
}
```

当您的代码使用不可变集合时，通常会有一个共享的“根”变量，该变量本身不是不可变的。在这种情况下，您*需要*使用同步。在以下代码中，每个线程将一个值推送到堆栈（创建一个新的不可变堆栈），然后更新共享的根变量；代码*需要*同步以更新`stack`变量：

```cs
// BAD CODE!!
async Task<bool> PlayWithStackAsync()
{
  ImmutableStack<int> stack = ImmutableStack<int>.Empty;

  Task task1 = Task.Run(() => { stack = stack.Push(3); });
  Task task2 = Task.Run(() => { stack = stack.Push(5); });
  Task task3 = Task.Run(() => { stack = stack.Push(7); });
  await Task.WhenAll(task1, task2, task3);

  return stack.IsEmpty;
}
```

线程安全集合（例如，`ConcurrentDictionary`）与不可变集合非常不同。与不可变集合不同，线程安全集合可以被更新。但它们内置了所有需要的同步，所以你不必担心同步集合更改。如果以下代码更新了一个`Dictionary`而不是`ConcurrentDictionary`，它将需要同步；但由于它更新了一个`ConcurrentDictionary`，所以不需要同步：

```cs
async Task<int> ThreadsafeCollectionsAsync()
{
  var dictionary = new ConcurrentDictionary<int, int>();

  Task task1 = Task.Run(() => { dictionary.TryAdd(2, 3); });
  Task task2 = Task.Run(() => { dictionary.TryAdd(3, 5); });
  Task task3 = Task.Run(() => { dictionary.TryAdd(5, 7); });
  await Task.WhenAll(task1, task2, task3);

  return dictionary.Count; // Always returns 3.
}
```

# 12.1 阻塞锁

## 问题

你有一些共享数据，需要安全地从多个线程读取和写入。

## 解决方案

这种情况的最佳解决方案是使用`lock`语句。当一个线程进入锁时，它将阻止任何其他线程进入该锁，直到锁被释放为止：

```cs
class MyClass
{
  // This lock protects the _value field.
  private readonly object _mutex = new object();

  private int _value;

  public void Increment()
  {
    lock (_mutex)
    {
      _value = _value + 1;
    }
  }
}
```

## 讨论

.NET 框架中还有许多其他类型的锁，例如`Monitor`、`SpinLock`和`ReaderWriterLockSlim`。在大多数应用程序中，几乎不应直接使用这些锁类型。特别是当开发人员没有必要使用`ReaderWriterLockSlim`时，会很自然地跳到它。基本的`lock`语句可以很好地处理 99%的情况。

在使用锁时有四个重要的指导原则：

+   限制锁的可见性。

+   记录锁保护的内容。

+   最小化锁定代码。

+   在持有锁时不要执行任意代码。

首先，你应该努力限制锁的可见性。在`lock`语句中使用的对象应该是一个私有字段，绝不应该暴露给类外的任何方法。通常每种类型最多只有一个锁成员；如果你有多个，请考虑将该类型重构为不同的类型。你*可以*锁定任何引用类型，但我更倾向于专门为`lock`语句使用一个字段，就像最后一个例子中那样。如果你在另一个实例上进行锁定，请确保它是私有的，只能在你的类内部使用；它不应该从构造函数传入或从属性获取器返回。你绝不应该`lock(this)`或锁定任何`Type`或`string`的实例；这些锁定可能会导致死锁，因为它们可以从其他代码中访问。

其次，记录锁保护的内容。在初次编写代码时很容易忽视这一步，但随着代码复杂性的增加，它变得更加重要。

第三，尽量减少在持有锁时执行的代码。要注意的一件事是阻塞调用；理想情况下，你的代码在持有锁时不应该阻塞。

最后，绝对不要在锁内调用任意代码。任意代码可能包括触发事件、调用虚方法或调用委托。如果必须执行任意代码，请在释放锁之后执行。

## 参见

Recipe 12.2 介绍了与`async`兼容的锁。`lock`语句不兼容`await`。

Recipe 12.3 介绍了线程间的信号传递。`lock`语句旨在保护共享数据，而不是在线程间发送信号。

食谱 12.5 讲解了节流，这是锁的一种泛化。锁可以被视为一次只允许一个线程通过。

# 12.2 异步锁

## 问题

如果你有一些共享数据，并且需要安全地从多个代码块中进行读写，这些代码块可能使用 `await`。

## 解决方案

.NET 框架中的 `SemaphoreSlim` 类型已在 .NET 4.5 中更新为与 `async` 兼容。以下是如何使用它的方法：

```cs
class MyClass
{
  // This lock protects the _value field.
  private readonly SemaphoreSlim _mutex = new SemaphoreSlim(1);

  private int _value;

  public async Task DelayAndIncrementAsync()
  {
    await _mutex.WaitAsync();
    try
    {
      int oldValue = _value;
      await Task.Delay(TimeSpan.FromSeconds(oldValue));
      _value = oldValue + 1;
    }
    finally
    {
      _mutex.Release();
    }
  }
}
```

你还可以使用 `Nito.AsyncEx` 库中的 `AsyncLock` 类型，它具有稍微更加优雅的 API：

```cs
class MyClass
{
  // This lock protects the _value field.
  private readonly AsyncLock _mutex = new AsyncLock();

  private int _value;

  public async Task DelayAndIncrementAsync()
  {
    using (await _mutex.LockAsync())
    {
      int oldValue = _value;
      await Task.Delay(TimeSpan.FromSeconds(oldValue));
      _value = oldValue + 1;
    }
  }
}
```

## 讨论

与 食谱 12.1 中提到的相同准则同样适用于这里，特别是：

+   限制锁的可见性。

+   记录锁保护的内容。

+   最小化在锁内的代码。

+   在持有锁时绝不执行任意代码。

保持你的锁实例私有；不要在类外部暴露它们。确保清楚地记录（并仔细考虑）锁实例到底保护什么。最小化在持有锁时执行的代码。特别地，不要调用任意代码，包括触发事件、调用虚方法和调用委托。

###### 提示

[`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中的 `AsyncLock` 类型。

## 参见

食谱 12.4 讲解了与 `async` 兼容的信号处理。锁的作用是保护共享数据，而不是作为信号。

食谱 12.5 讲解了节流，这是锁的一种泛化。锁可以被视为一次只允许一个线程通过。

# 12.3 阻塞信号

## 问题

你需要从一个线程向另一个线程发送通知。

## 解决方案

最常见和通用的跨线程信号是 `ManualResetEventSlim`。手动重置事件可以处于两种状态中的一种：信号或未信号。任何线程都可以将事件设置为信号状态或将事件重置为未信号状态。线程还可以等待事件信号。

下面两个方法由不同的线程调用；一个线程等待另一个线程的信号：

```cs
class MyClass
{
  private readonly ManualResetEventSlim _initialized =
      new ManualResetEventSlim();

  private int _value;

  public int WaitForInitialization()
  {
    _initialized.Wait();
    return _value;
  }

  public void InitializeFromAnotherThread()
  {
    _value = 13;
    _initialized.Set();
  }
}
```

## 讨论

`ManualResetEventSlim` 是一个很好的通用信号，用于一个线程向另一个线程发送信号，但只有在适当时才应使用它。如果“信号”实际上是通过线程之间发送某些数据的 *消息*，那么考虑使用生产者/消费者队列。另一方面，如果信号只是用于协调对共享数据的访问，则应使用锁。

.NET 框架中还有其他较少使用的线程同步信号类型。如果 `ManualResetEventSlim` 不符合你的需求，考虑使用 `AutoResetEvent`、`CountdownEvent` 或 `Barrier`。

`ManualResetEventSlim` 是一个同步信号，因此 `WaitForInitialization` 将阻塞调用线程，直到信号被发送。如果你想等待信号而不阻塞线程，则需要一个异步信号，如 食谱 12.4 中描述的那样。

## 参见

食谱 9.6 讲解了阻塞生产者/消费者队列。

12.1 配方 涵盖阻塞锁。

12.4 配方 涵盖 `async` 兼容的信号。

# 12.4 异步信号

## 问题

您需要从代码的一部分发送通知到另一部分，并且通知的接收者必须异步等待它。

## 解决方案

使用 `TaskCompletionSource<T>` 异步发送通知，如果通知只需发送一次。发送代码调用 `TrySetResult`，接收代码等待其 `Task` 属性：

```cs
class MyClass
{
  private readonly TaskCompletionSource<object> _initialized =
      new TaskCompletionSource<object>();

  private int _value1;
  private int _value2;

  public async Task<int> WaitForInitializationAsync()
  {
    await _initialized.Task;
    return _value1 + _value2;
  }

  public void Initialize()
  {
    _value1 = 13;
    _value2 = 17;
    _initialized.TrySetResult(null);
  }
}
```

`TaskCompletionSource<T>` 类型可用于异步等待任何类型的情况 —— 在本例中，来自代码其他部分的通知。如果信号仅发送一次，则此方法效果很好，但如果需要同时关闭和打开信号，则效果不佳。

`Nito.AsyncEx` 库包含 `AsyncManualResetEvent` 类型，这是用于异步代码的 `ManualResetEvent` 的近似等效物。以下示例是编造的，但展示了如何使用 `AsyncManualResetEvent` 类型：

```cs
class MyClass
{
  private readonly AsyncManualResetEvent _connected =
      new AsyncManualResetEvent();

  public async Task WaitForConnectedAsync()
  {
    await _connected.WaitAsync();
  }

  public void ConnectedChanged(bool connected)
  {
    if (connected)
      _connected.Set();
    else
      _connected.Reset();
  }
}
```

## 讨论

信号是一种通用的通知机制。但如果这个“信号”是*消息*，用于从一段代码发送数据到另一段代码，则考虑使用生产者/消费者队列。同样，不要仅仅为了协调对共享数据的访问而使用通用信号；在这种情况下，请使用异步锁。

###### 提示

`AsyncManualResetEvent` 类型位于 [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中。

## 另请参阅

9.8 配方 涵盖异步生产者/消费者队列。

12.2 配方 涵盖异步锁。

12.3 配方 涵盖阻塞信号，可用于跨线程的通知。

# 12.5 节流

## 问题

您有高度并发的代码，实际上是*过于*并发，需要一些方法来限制并发。

当应用程序的某些部分无法跟上其他部分时，导致数据项累积并消耗内存时，代码过于并发。在这种情况下，通过节流部分代码可以防止内存问题。

## 解决方案

解决方案根据代码正在执行的并发类型而异。这些解决方案都将并发限制为特定值。反应式扩展具有更强大的选项，例如滑动时间窗口；有关 System.Reactive observables 的节流更详尽地介绍在 6.4 配方 中。

Dataflow 和并行代码都具有内置选项用于节流并发：

```cs
IPropagatorBlock<int, int> DataflowMultiplyBy2()
{
  var options = new ExecutionDataflowBlockOptions
  {
    MaxDegreeOfParallelism = 10
  };

  return new TransformBlock<int, int>(data => data * 2, options);
}

// Using Parallel LINQ (PLINQ)
IEnumerable<int> ParallelMultiplyBy2(IEnumerable<int> values)
{
  return values.AsParallel()
      .WithDegreeOfParallelism(10)
      .Select(item => item * 2);
}

// Using the Parallel class
void ParallelRotateMatrices(IEnumerable<Matrix> matrices, float degrees)
{
  var options = new ParallelOptions
  {
    MaxDegreeOfParallelism = 10
  };
  Parallel.ForEach(matrices, options, matrix => matrix.Rotate(degrees));
}
```

并发异步代码可以通过使用 `SemaphoreSlim` 进行节流：

```cs
async Task<string[]> DownloadUrlsAsync(HttpClient client,
    IEnumerable<string> urls)
{
  using var semaphore = new SemaphoreSlim(10);
  Task<string>[] tasks = urls.Select(async url =>
  {
    await semaphore.WaitAsync();
    try
    {
      return await client.GetStringAsync(url);
    }
    finally
    {
      semaphore.Release();
    }
  }).ToArray();
  return await Task.WhenAll(tasks);
}
```

## 讨论

当您发现代码使用了过多资源（例如 CPU 或网络连接）时，可能需要进行节流。请记住，最终用户通常拥有比开发者更弱的计算机，因此最好进行稍微多一点的节流，而不是太少。

## 另请参阅

6.4 配方 涵盖反应式代码的节流。
