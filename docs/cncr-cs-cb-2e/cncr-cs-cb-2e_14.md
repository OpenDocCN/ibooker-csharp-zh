# 第十四章：场景

在本章中，我们将介绍各种类型和技术，以解决编写并发程序时的一些常见场景。这些类型的情况可能填满另一本完整的书，因此我只选择了一些我认为最有用的情况。

# 14.1 初始化共享资源

## 问题

您有一个在代码的多个部分之间共享的资源。第一次访问该资源时需要对其进行初始化。

## 解决方案

.NET 框架包括一种专门用于此目的的类型：`Lazy<T>`。您可以使用用于初始化实例的工厂委托构造`Lazy<T>`类型的实例。然后，通过`Value`属性使实例可用。以下代码演示了`Lazy<T>`类型：

```cs
static int _simpleValue;
static readonly Lazy<int> MySharedInteger = new Lazy<int>(() => _simpleValue++);

void UseSharedInteger()
{
  int sharedValue = MySharedInteger.Value;
}
```

无论多少线程同时调用`UseSharedInteger`，工厂委托只执行一次，并且所有线程都等待相同的实例。创建后，实例被缓存，并且所有对`Value`属性的未来访问都返回相同的实例（在上面的示例中，`MySharedInteger.Value`始终为`0`）。

如果初始化需要异步工作，可以使用`Lazy<Task<T>>`，可以使用类似的方法：

```cs
static int _simpleValue;
static readonly Lazy<Task<int>> MySharedAsyncInteger =
    new Lazy<Task<int>>(async () =>
    {
      await Task.Delay(TimeSpan.FromSeconds(2)).ConfigureAwait(false);
      return _simpleValue++;
    });

async Task GetSharedIntegerAsync()
{
  int sharedValue = await MySharedAsyncInteger.Value;
}
```

在这个示例中，委托返回一个`Task<int>`，即一个确定整数值的异步操作。无论代码的哪些部分同时调用`Value`，`Task<int>`只创建一次并返回给所有调用者。然后，每个调用者可以选择（异步地）等待任务完成，方法是将任务传递给`await`。

前面的代码是一种可接受的模式，但还有一些额外的考虑因素。首先，异步委托可能在调用`Value`的任何线程上执行，并且该委托将在该上下文内执行。如果可能有不同类型的线程调用`Value`（例如，UI 线程和线程池线程，或两个不同的 ASP.NET 请求线程），则始终在线程池线程上执行异步委托可能更好。通过将工厂委托包装在`Task.Run`调用中，可以很容易地实现这一点：

```cs
static int _simpleValue;
static readonly Lazy<Task<int>> MySharedAsyncInteger =
  new Lazy<Task<int>>(() => Task.Run(async () =>
  {
    await Task.Delay(TimeSpan.FromSeconds(2));
    return _simpleValue++;
  }));

async Task GetSharedIntegerAsync()
{
  int sharedValue = await MySharedAsyncInteger.Value;
}
```

另一个考虑因素是，`Task<T>`实例只创建一次。如果异步委托抛出异常，则`Lazy<Task<T>>`将缓存该失败的任务。这很少是可取的；通常最好的做法是在下次请求懒惰值时重新执行委托，而不是缓存异常。没有办法“重置”`Lazy<T>`，但可以创建一个新的类来处理重新创建`Lazy<T>`实例的情况：

```cs
public sealed class AsyncLazy<T>
{
  private readonly object _mutex;
  private readonly Func<Task<T>> _factory;
  private Lazy<Task<T>> _instance;

  public AsyncLazy(Func<Task<T>> factory)
  {
    _mutex = new object();
    _factory = RetryOnFailure(factory);
    _instance = new Lazy<Task<T>>(_factory);
  }

  private Func<Task<T>> RetryOnFailure(Func<Task<T>> factory)
  {
    return async () =>
    {
      try
      {
        return await factory().ConfigureAwait(false);
      }
      catch
      {
        lock (_mutex)
        {
          _instance = new Lazy<Task<T>>(_factory);
        }
        throw;
      }
    };
  }

  public Task<T> Task
  {
    get
    {
      lock (_mutex)
        return _instance.Value;
    }
  }
}

static int _simpleValue;
static readonly AsyncLazy<int> MySharedAsyncInteger =
  new AsyncLazy<int>(() => Task.Run(async () =>
  {
    await Task.Delay(TimeSpan.FromSeconds(2));
    return _simpleValue++;
  }));

async Task GetSharedIntegerAsync()
{
  int sharedValue = await MySharedAsyncInteger.Task;
}
```

## 讨论

此配方中的最终代码示例是异步延迟初始化的通用代码模式，有些笨拙。`AsyncEx` 库包含一个名为 `AsyncLazy<T>` 的类型，它就像一个 `Lazy<Task<T>>`，在线程池上执行其工厂委托，并具有失败重试选项。它也可以直接等待，因此声明和使用看起来如下所示：

```cs
static int _simpleValue;
private static readonly AsyncLazy<int> MySharedAsyncInteger =
  new AsyncLazy<int>(async () =>
  {
    await Task.Delay(TimeSpan.FromSeconds(2));
    return _simpleValue++;
  },
  AsyncLazyFlags.RetryOnFailure);

public async Task UseSharedIntegerAsync()
{
  int sharedValue = await MySharedAsyncInteger;
}
```

###### 提示

`AsyncLazy<T>` 类型位于 [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中。

## 另请参阅

第一章 涵盖了基本的 `async`/`await` 编程。

Recipe 13.1 涵盖了将工作调度到线程池的方法。

# 14.2 System.Reactive 延迟评估

## 问题

您希望每次有人订阅时都创建一个新的源可观察对象。例如，您希望每个订阅都代表对 web 服务的不同请求。

## 解决方案

System.Reactive 库具有一个名为 `Observable.Defer` 的操作符，该操作符每次订阅可观察对象时都会执行一个委托。该委托充当创建可观察对象的工厂。以下代码使用 `Defer` 来在每次有人订阅可观察对象时调用一个异步方法：

```cs
void SubscribeWithDefer()
{
  var invokeServerObservable = Observable.Defer(
      () => GetValueAsync().ToObservable());
  invokeServerObservable.Subscribe(_ => { });
  invokeServerObservable.Subscribe(_ => { });

  Console.ReadKey();
}

async Task<int> GetValueAsync()
{
  Console.WriteLine("Calling server...");
  await Task.Delay(TimeSpan.FromSeconds(2));
  Console.WriteLine("Returning result...");
  return 13;
}
```

如果执行此代码，应该会看到以下输出：

```cs
Calling server...
Calling server...
Returning result...
Returning result...
```

## 讨论

您自己的代码通常不会多次订阅可观察对象，但某些 System.Reactive 操作符在其实现中会这样做。例如，`Observable.While` 操作符会在条件为 true 时重新订阅源序列。`Defer` 允许您定义一个每次新订阅时都会重新评估的可观察对象。如果需要刷新或更新该可观察对象的数据，则这非常有用。

## 另请参阅

Recipe 8.6 涵盖了在可观察对象中包装异步方法。

# 14.3 异步数据绑定

## 问题

您正在异步检索数据，并需要将结果数据绑定（例如，在 Model-View-ViewModel 设计的 ViewModel 中）。

## 解决方案

当数据绑定使用属性时，必须立即且同步返回某种结果。如果实际值需要异步确定，可以返回默认结果，稍后使用正确的值更新属性。

请记住，异步操作可能会失败，也可能会成功。由于您正在编写 ViewModel，因此可以使用数据绑定来更新 UI 以反映错误条件。

`Nito.Mvvm.Async library` 中有一个名为 `NotifyTask` 的类型可用于此目的：

```cs
class MyViewModel
{
  public MyViewModel()
  {
    MyValue = NotifyTask.Create(CalculateMyValueAsync());
  }

  public NotifyTask<int> MyValue { get; private set; }

  private async Task<int> CalculateMyValueAsync()
  {
    await Task.Delay(TimeSpan.FromSeconds(10));
    return 13;
  }
}
```

可以将数据绑定到`NotifyTask<T>`属性的各种属性，如本示例所示：

```cs
<Grid>
  <Label Content="Loading..."
      Visibility="{Binding MyValue.IsNotCompleted,
 Converter={StaticResource BooleanToVisibilityConverter}}"/>
  <Label Content="{Binding MyValue.Result}"
      Visibility="{Binding MyValue.IsSuccessfullyCompleted,
 Converter={StaticResource BooleanToVisibilityConverter}}"/>
  <Label Content="An error occurred" Foreground="Red"
      Visibility="{Binding MyValue.IsFaulted,
 Converter={StaticResource BooleanToVisibilityConverter}}"/>
</Grid>
```

MvvmCross 库中有一个 `MvxNotifyTask`，与 `NotifyTask<T>` 非常相似。

## 讨论

您也可以编写自己的数据绑定包装器，而不使用库中的一个。以下代码提供了基本思路：

```cs
class BindableTask<T> : INotifyPropertyChanged
{
  private readonly Task<T> _task;

  public BindableTask(Task<T> task)
  {
    _task = task;
    var _ = WatchTaskAsync();
  }

  private async Task WatchTaskAsync()
  {
    try
    {
      await _task;
    }
    catch
    {
    }

    OnPropertyChanged("IsNotCompleted");
    OnPropertyChanged("IsSuccessfullyCompleted");
    OnPropertyChanged("IsFaulted");
    OnPropertyChanged("Result");
  }

  public bool IsNotCompleted { get { return !_task.IsCompleted; } }
  public bool IsSuccessfullyCompleted
  {
    get { return _task.Status == TaskStatus.RanToCompletion; }
  }
  public bool IsFaulted { get { return _task.IsFaulted; } }
  public T Result
  {
    get { return IsSuccessfullyCompleted ? _task.Result : default; }
  }

  public event PropertyChangedEventHandler PropertyChanged;

  protected virtual void OnPropertyChanged(string propertyName)
  {
    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
  }
}
```

注意，这里有一个空的`catch`子句是有意为之：该代码明确希望捕获所有异常，并通过数据绑定处理这些情况。此外，该代码明确不希望使用`ConfigureAwait(false)`，因为应在 UI 线程上引发`PropertyChanged`事件。

###### 提示

`NotifyTask`类型位于[`Nito.Mvvm.Async`](http://bit.ly/nito-m-async) NuGet 包中。`MvxNotifyTask`类型位于[`MvvmCross`](http://bit.ly/m-cross) NuGet 包中。

## 另请参阅

第一章 讨论了基本的`async`/`await`编程。

配方 2.7 讨论了如何使用`ConfigureAwait`。

# 14.4 隐式状态

## 问题

你有一些状态变量，需要在调用堆栈的不同点访问。例如，你有一个当前操作标识符，你希望用于日志记录，但不想将其添加为每个方法的参数。

## 解决方案

最佳解决方案是向方法添加参数，将数据存储为类的成员，或使用依赖注入为代码的不同部分提供数据。然而，在某些情况下，这样做会使代码变得过于复杂。

`AsyncLocal<T>`类型使您能够为状态提供一个对象，可以在逻辑“上下文”中存储它。以下代码展示了如何使用`AsyncLocal<T>`设置稍后由日志记录方法读取的操作标识符：

```cs
private static AsyncLocal<Guid> _operationId = new AsyncLocal<Guid>();

async Task DoLongOperationAsync()
{
  _operationId.Value = Guid.NewGuid();

  await DoSomeStepOfOperationAsync();
}

async Task DoSomeStepOfOperationAsync()
{
  await Task.Delay(100); // Some async work

  // Do some logging here.
  Trace.WriteLine("In operation: " + _operationId.Value);
}
```

许多时候，在单个`AsyncLocal<T>`实例中拥有更复杂的数据结构（如值堆栈）是很有用的。这是可能的，但有一个重要注意事项：您应该只在`AsyncLocal<T>`中存储不可变数据。每当需要更新数据时，应该覆盖现有值。通常有助于将`AsyncLocal<T>`隐藏在一个助手类型中，以确保存储的数据是不可变的并且正确更新：

```cs
internal sealed class AsyncLocalGuidStack
{
  private readonly AsyncLocal<ImmutableStack<Guid>> _operationIds =
      new AsyncLocal<ImmutableStack<Guid>>();

  private ImmutableStack<Guid> Current =>
      _operationIds.Value ?? ImmutableStack<Guid>.Empty;

  public IDisposable Push(Guid value)
  {
    _operationIds.Value = Current.Push(value);
    return new PopWhenDisposed(this);
  }

  private void Pop()
  {
    ImmutableStack<Guid> newValue = Current.Pop();
    if (newValue.IsEmpty)
      newValue = null;
    _operationIds.Value = newValue;
  }

  public IEnumerable<Guid> Values => Current;

  private sealed class PopWhenDisposed : IDisposable
  {
    private AsyncLocalGuidStack _stack;

    public PopWhenDisposed(AsyncLocalGuidStack stack) =>
        _stack = stack;

    public void Dispose()
    {
      _stack?.Pop();
      _stack = null;
    }
  }
}

private static AsyncLocalGuidStack _operationIds = new AsyncLocalGuidStack();

async Task DoLongOperationAsync()
{
  using (_operationIds.Push(Guid.NewGuid()))
    await DoSomeStepOfOperationAsync();
}

async Task DoSomeStepOfOperationAsync()
{
  await Task.Delay(100); // some async work

  // Do some logging here.
  Trace.WriteLine("In operation: " +
      string.Join(":", _operationIds.Values));
}
```

封装类型确保底层数据是不可变的，并且新值被推送到堆栈上。它还提供了一种便利的`IDisposable`方法来从堆栈中弹出值。

## 讨论

旧代码可能使用`ThreadStatic`属性来处理同步代码使用的上下文状态。将旧代码转换为异步时，`AsyncLocal<T>`是替换`ThreadStaticAttribute`的首选。`AsyncLocal<T>`可用于同步和异步代码，并应该是现代应用程序中隐式状态的默认选择。

## 另请参阅

第一章 讨论了基本的`async`/`await`编程。

第九章 讨论了几种不可变集合，用于在需要将复杂数据存储为隐式状态时使用。

# 14.5 同步和异步代码相同

## 问题

你有一些代码需要通过同步和异步 API 公开，但你不想重复逻辑。在更新代码以支持异步时，经常会遇到这种情况，但现有的同步消费者不能（暂时）改变。

## 解决方案

如果可能的话，请尝试按照现代设计指南组织您的代码，例如端口和适配器（六边形架构），将业务逻辑与 I/O 等副作用分离。如果能够达到这种情况，那么不需要为任何事情同时暴露同步和异步 API；您的业务逻辑总是同步的，而 I/O 总是异步的。

然而，这是一个非常崇高的目标，在**现实世界**中，棕地代码可能会很混乱，在采用异步代码之前很少有时间使其完美。即使现有的 API 设计不佳，也经常需要维护以保持向后兼容性。

在这种情况下，没有完美的解决方案。许多开发人员尝试使同步代码调用异步代码，或使异步代码调用同步代码，但这两种方法都是反模式。在这种情况下，我倾向于使用布尔参数黑客。这是一种在单个方法中保持所有逻辑的方法，同时暴露同步和异步 API。

布尔参数黑客的主要思想是，有一个包含逻辑的私有核心方法。该核心方法具有异步签名，并带有布尔参数，确定核心方法是否应该是异步的。如果布尔参数指定核心方法应该是同步的，那么它必须返回一个已完成的任务。然后，您可以编写同时转发到核心方法的异步和同步 API 方法：

```cs
private async Task<int> DelayAndReturnCore(bool sync)
{
  int value = 100;

  // Do some work.
  if (sync)
    Thread.Sleep(value); // Call synchronous API.
  else
    await Task.Delay(value); // Call asynchronous API.

  return value;
}

// Asynchronous API
public Task<int> DelayAndReturnAsync() =>
    DelayAndReturnCore(sync: false);

// Synchronous API
public int DelayAndReturn() =>
    DelayAndReturnCore(sync: true).GetAwaiter().GetResult();
```

异步 API `DelayAndReturnAsync` 调用带有布尔参数 `sync` 设置为 `false` 的 `DelayAndReturnCore`；这意味着 `DelayAndReturnCore` 可能会异步执行，并使用底层异步的“延迟”API `Task.Delay`。从 `DelayAndReturnCore` 返回的任务会直接返回给 `DelayAndReturnAsync` 的调用者。

同步 API `DelayAndReturn` 调用带有布尔参数 `sync` 设置为 `true` 的 `DelayAndReturnCore`；这意味着 `DelayAndReturnCore` 必须同步执行，并使用底层同步的“延迟”API `Thread.Sleep`。`DelayAndReturnCore` 返回的任务必须已经完成，因此可以安全地提取结果。`DelayAndReturn` 使用 `GetAwaiter().GetResult()` 从任务中检索结果；这样做可以避免使用 `Task<T>.Result` 属性时可能出现的 `AggregateException` 包装器。

## 讨论

这不是一个理想的解决方案，但它可以帮助处理现实世界的应用场景。

对于这个解决方案，现在需要注意一些注意事项。如果`Core`方法未能正确地尊重其`sync`参数，可能会出现最严重的问题。如果`Core`方法在`sync`为`true`时返回了一个不完整的任务，那么同步 API 很容易会发生死锁；同步 API 可以阻塞其任务的唯一原因是它知道任务已经完成。类似地，如果`Core`方法在`sync`为`false`时阻塞了线程，那么应用程序的效率就不如预期。

可以对这个解决方案进行改进的一个方法是在同步 API 中添加一个检查，验证返回的任务实际上是已完成的。如果它曾经未完成过，那么这就是一个严重的编码错误。

## 另请参见

第一章介绍了基本的`async`/`await`编程，包括讨论一般情况下在异步代码中阻塞可能导致的死锁问题。

# 14.6 数据流网格中的铁路编程

## 问题

您已经建立了一个数据流网格，但有些数据项未能处理。您希望以一种方式响应这些错误，以保持数据流网格的正常运行。

## 解决方案

默认情况下，如果一个块在处理数据项时遇到异常，那么该块将会故障，导致无法继续处理任何数据项。这个解决方案的核心思想是将异常视为另一种数据。如果数据流网格操作的类型可以是*异常*或*数据*，那么即使出现异常，网格仍然可以继续运行并处理其他数据项。

这种编程方式有时被称为“铁路”编程，因为网格中的项目可以被视为沿着两条单独的轨道行驶。第一条是正常的“数据”轨道：如果一切顺利，项目将留在“数据”轨道上，并通过网格进行转换和操作，直到到达网格的末端。第二条轨道是“错误”轨道；在任何块中，如果处理项目时出现异常，该异常将转移到“错误”轨道并通过网格传递。异常项目不会被处理；它们只是从块传递到块，因此它们也会到达网格的末端。网格中的终端块最终会接收到一系列项目，每个项目都是数据项或异常项；数据项表示已成功完成整个网格的数据，异常项表示网格某个点的处理错误。

要设置这种“铁路”编程，首先需要定义一个表示数据项或异常的类型。如果要使用预先构建的类型，有几种可用。这种类型在函数式编程社区中很常见，通常称为`Try`或`Error`或`Exceptional`，是`Either`单子的特例。我定义了自己的`Try<T>`类型作为示例；它在[`Nito.Try` NuGet 包](https://www.nuget.org/packages/Nito.Try/)中，源代码在[GitHub 上](https://github.com/StephenCleary/Try)。

一旦您有某种`Try<T>`类型，设置网格有点繁琐，但并不可怕。每个数据流块的类型应从`T`更改为`Try<T>`，并且该块中的任何处理都应通过将一个`Try<T>`值映射到另一个来完成。使用我的`Try<T>`类型，通过调用`Try<T>.Map`来完成这一点。我发现定义小工厂方法用于铁路导向数据流块而不是在行内添加额外代码会很有帮助。以下代码是一个帮助方法的示例，它构造一个在`Try<T>`值上操作的`TransformBlock`，通过调用`Try<T>.Map`：

```cs
private static TransformBlock<Try<TInput>, Try<TOutput>>
    RailwayTransform<TInput, TOutput>(Func<TInput, TOutput> func)
{
  return new TransformBlock<Try<TInput>, Try<TOutput>>(t => t.Map(func));
}
```

有了这些帮助程序，数据流网格创建代码会更加简单：

```cs
var subtractBlock = RailwayTransform<int, int>(value => value - 2);
var divideBlock = RailwayTransform<int, int>(value => 60 / value);
var multiplyBlock = RailwayTransform<int, int>(value => value * 2);

var options = new DataflowLinkOptions { PropagateCompletion = true };
subtractBlock.LinkTo(divideBlock, options);
divideBlock.LinkTo(multiplyBlock, options);

// Insert data items into the first block.
subtractBlock.Post(Try.FromValue(5));
subtractBlock.Post(Try.FromValue(2));
subtractBlock.Post(Try.FromValue(4));
subtractBlock.Complete();

// Receive data/exception items from the last block.
while (await multiplyBlock.OutputAvailableAsync())
{
  Try<int> item = await multiplyBlock.ReceiveAsync();
  if (item.IsValue)
    Console.WriteLine(item.Value);
  else
    Console.WriteLine(item.Exception.Message);
}
```

## 讨论

铁路编程是避免数据流块故障的好方法。由于铁路编程是基于单子的函数式编程构造，将其转换为.NET 时有些笨拙，但可用。如果您有一个需要容错的数据流网格，那么铁路编程绝对值得一试。

## 参见

Recipe 5.2 讲述了异常如何影响块的正常方式，并可以通过网格传播，如果不使用铁路编程。 

# 14.7 节流进度更新

## 问题

您有一个长时间运行的操作，报告进度，并在 UI 中显示进度更新。但进度更新过于频繁，导致 UI 无响应。

## 解决方案

考虑以下代码，它非常快速地报告进度：

```cs
private string Solve(IProgress<int> progress)
{
  // Count as quickly as possible for 3 seconds.
  var endTime = DateTime.UtcNow.AddSeconds(3);
  int value = 0;
  while (DateTime.UtcNow < endTime)
  {
    value++;
    progress?.Report(value);
  }
  return value.ToString();
}
```

你可以通过将其包装在`Task.Run`中并传入`IProgress<T>`，从 GUI 应用程序执行此代码。以下示例代码适用于 WPF，但相同的概念适用于任何 GUI 平台（WPF、Xamarin 或 Windows Forms）：

```cs
// For simplicity, this code updates a label directly.
// In a real-world MVVM application, those assignments
//  would instead be updating a ViewModel property
//  which is data-bound to the actual UI.
private async void StartButton_Click(object sender, RoutedEventArgs e)
{
  MyLabel.Content = "Starting...";
  var progress = new Progress<int>(value => MyLabel.Content = value);
  var result = await Task.Run(() => Solve(progress));
  MyLabel.Content = $"Done! Result: {result}";
}
```

这段代码会导致 UI 在我的机器上变得无响应相当长的时间，大约 20 秒，然后突然 UI 重新响应并且只显示`"Done! Result:"`消息。中间的进度报告从未被看到。发生的情况是后台代码非常快地向 UI 线程发送进度报告，以至于在运行仅 3 秒后，UI 线程需要大约额外的 17 秒来处理所有这些进度报告，一遍又一遍地更新标签。最后，UI 线程最后一次更新标签的值为`"Done! Result:"`，然后*最终*有时间重新绘制屏幕，向用户显示更新后的标签值。

首先要意识到的是，我们需要节流进度报告。这是确保 UI 在进度更新之间有足够时间重新绘制自身的唯一方法。接下来要意识到的是，我们希望基于*时间*而不是*报告数量*来进行节流。虽然您可能会试图通过仅发送每百个或更多报告中的一个来节流进度报告，但出于“讨论”部分所述的原因，这并不理想。

我们希望处理*时间*表明我们应该考虑 System.Reactive。事实上，System.Reactive 具有专门设计用于按时间节流的操作符。因此，System.Reactive 似乎在这个解决方案中将发挥作用。

要开始，您可以定义一个`IProgress<T>`实现，该实现为每个进度报告触发一个事件，然后通过包装该事件创建一个可观察对象来接收这些进度报告：

```cs
public static class ObservableProgress
{
  private sealed class EventProgress<T> : IProgress<T>
  {
    void IProgress<T>.Report(T value) => OnReport?.Invoke(value);
    public event Action<T> OnReport;
  }

  public static (IObservable<T>, IProgress<T>) Create<T>()
  {
    var progress = new EventProgress<T>();
    var observable = Observable.FromEvent<T>(
        handler => progress.OnReport += handler,
        handler => progress.OnReport -= handler);
    return (observable, progress);
  }
}
```

方法`ObservableProgress.Create<T>`将创建一对：一个`IObservable<T>`和一个`IProgress<T>`，其中所有发送到`IProgress<T>`的进度报告将发送到`IObservable<T>`的订阅者。现在我们有了进度报告的可观察流；下一步是对其进行节流。

我们希望更新 UI 的速度足够慢，以使其保持响应，并且我们希望更新 UI 的速度足够快，以便用户能看到更新。人类感知远远慢于计算机显示，因此有很大的可能性。如果您更喜欢真实的可读性，每秒节流一次更新可能足够了。如果您更喜欢更实时的反馈，我发现每 100 或 200 毫秒（ms）节流一次更新足够快，以至于用户看到事情正在迅速发生，并获得进度详细信息的一般感觉，同时仍然足够慢以使 UI 保持响应。

另一个要记住的点是，进度报告可能会从其他线程引发—在这种情况下，它们是从后台线程引发的。节流应尽可能靠近源头完成，因此我们希望在后台线程上保持节流。然而，更新 UI 的代码需要在 UI 线程上运行。考虑到这一点，您可以定义一个 `CreateForUi` 方法，处理节流和转换到 UI 线程：

```cs
public static class ObservableProgress
{
  // Note: this must be called from the UI thread.
  public static (IObservable<T>, IProgress<T>) CreateForUi<T>(
      TimeSpan? sampleInterval = null)
  {
    var (observable, progress) = Create<T>();
    observable = observable
        .Sample(sampleInterval ?? TimeSpan.FromMilliseconds(100))
        .ObserveOn(SynchronizationContext.Current);
    return (observable, progress);
  }
}
```

现在，您有一个辅助方法，可以在更新到达用户界面之前对其进行节流。您可以在前面的代码示例中的按钮点击处理程序中使用这个辅助方法：

```cs
// For simplicity, this code updates a label directly.
// In a real-world MVVM application, those assignments
//  would instead be updating a ViewModel property
//  which is data-bound to the actual UI.
private async void StartButton_Click(object sender, RoutedEventArgs e)
{
  MyLabel.Content = "Starting...";
  var (observable, progress) = ObservableProgress.CreateForUi<int>();
  string result;
  using (observable.Subscribe(value => MyLabel.Content = value))
    result = await Task.Run(() => Solve(progress));
  MyLabel.Content = $"Done! Result: {result}";
}
```

新代码调用了我们的辅助方法 `ObservableProgress.CreateForUi`，它创建了 `IObservable<T>` 和 `IProgress<T>` 对。代码订阅进度更新，并保持到 `Solve` 完成为止。最后，它将 `IProgress<T>` 传递给长时间运行的 `Solve` 方法。当 `Solve` 调用 `IProgress<T>.Report` 时，这些报告首先在 100 毫秒的时间窗口内进行采样，每 100 毫秒转发一次更新到 UI 线程，并用于更新标签文本。现在，UI 完全响应！

## 讨论

这个配方是本书中其他配方的有趣组合！没有引入新技术；我们只是介绍了如何组合这些配方以得出这个解决方案。

在野外经常见到的这个问题的另一种替代解决方案是“模数解决方案”。这个解决方案的背后思想是 `Solve` 本身必须节流自己的进度更新；例如，如果代码只想处理每 100 个实际更新的一个更新，那么代码可能使用一些模数技术，如 `if (value % 100 == 0) progress?.Report(value);`。

采用模数方法存在几个问题。首先，没有“正确”的模数值；通常，开发人员会尝试各种值，直到在他们自己的笔记本电脑上运行良好。然而，同样的代码在运行在客户的大型服务器或不足的虚拟机内时可能表现不佳。此外，不同的平台和环境缓存方式差异很大，这可能导致代码运行比预期快得多（或慢得多）。当然，“最新”的计算机硬件的能力随时间而变化。因此，模数值最终只是一个猜测；它不会在所有地方和所有时间都正确。

模数方法的另一个问题是，它试图在代码的错误部分修复问题。这个问题纯粹是一个 UI 问题；UI 存在问题，并且 UI 层应该为其提供解决方法。在这个配方的示例代码中，`Solve` 表示一些后台业务处理逻辑；它不应关心 UI 特定的问题。控制台应用程序可能希望使用与 WPF 应用程序非常不同的模数。

模数方法正确的一点是，在将更新发送到 UI 线程之前最好对更新进行节流。本示例中的解决方案也是如此：在将更新发送到 UI 线程之前，它立即在后台线程上同步地节流更新。通过注入自己的`IProgress<T>`实现，UI 能够在不需要对`Solve`方法本身进行任何更改的情况下进行自己的节流。

## 参见

Recipe 2.3 涵盖了使用`IProgress<T>`从长时间运行的操作报告进度。

Recipe 13.1 涵盖了使用`Task.Run`在线程池线程上运行同步代码。

Recipe 6.1 涵盖了使用`FromEvent`将.NET 事件包装成可观察对象。

Recipe 6.4 涵盖了使用`Sample`按时间节流可观察对象。

Recipe 6.2 涵盖了使用`ObserveOn`将可观察通知移动到另一个上下文。
