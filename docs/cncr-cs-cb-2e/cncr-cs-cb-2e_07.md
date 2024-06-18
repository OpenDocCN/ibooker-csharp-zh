# 第七章 测试

测试是软件质量的重要组成部分。近年来，单元测试的支持者已经变得司空见惯；似乎你无论读什么或听什么都会提到它。一些人推广*测试驱动开发*，这是一种编码风格，确保在应用程序完成时有全面的测试。单元测试对代码质量和整体完成时间的好处是众所周知的，但许多开发者仍然不写单元测试。

我鼓励你至少写一些单元测试。从你感到最缺乏信心的代码开始。根据我的经验，单元测试给了我两个主要优势：

+   **更好地理解代码。** 你知道应用程序的那一部分能工作但你不知道为什么吗？当真正奇怪的 bug 报告出现时，它总是潜在地存在于你的脑海中。为你觉得困难的代码编写单元测试是了解它如何工作的一个绝佳方法。在编写描述其行为的单元测试后，代码不再神秘；你最终会得到一组描述其行为及其对其他代码的依赖关系的单元测试。

+   **更大的改动信心。** 迟早你会收到需要修改令你害怕的代码的功能请求，你将无法再假装它不存在（我知道那种感觉，我也经历过！）。最好是主动出击：在功能请求到来之前为可怕的代码编写单元测试。一旦你的单元测试完成，你就会有一个早期警报系统，如果你的修改破坏了现有行为，它会立即提醒你。当你有一个拉取请求时，单元测试也会让你更有信心，确保代码变更不会破坏现有行为。

这两个优势同样适用于你自己的代码，不仅仅是别人的代码。我相信还有其他优点。单元测试减少 bug 的频率吗？很可能是。单元测试减少项目总体时间吗？可能是。但我描述的优势是确定的；每次我写单元测试时，我都能感受到它们。所以，这是我对单元测试的推销。

本章包含的配方全都是关于测试的。很多开发者（即使通常编写单元测试的那些人）都会避开测试并发代码，因为他们认为这很难。然而，正如这些配方将展示的那样，单元测试并发代码并不像他们想象的那么难。现代特性和库，如`async`和 System.Reactive，在测试方面都进行了大量的思考，这一点很明显。我鼓励你使用这些配方来编写单元测试，特别是如果你对并发编程还很陌生（即新的并发代码看起来很难或令人害怕）。

# 7.1 异步方法的单元测试

## 问题

你有一个`async`方法需要进行单元测试。

## 解决方案

大多数现代单元测试框架支持`async Task`单元测试方法，包括 MSTest、NUnit 和 xUnit。MSTest 从 Visual Studio 2012 开始支持这些测试。如果你使用其他单元测试框架，可能需要升级到最新版本。

这里是一个`async` MSTest 单元测试的例子：

```cs
[TestMethod]
public async Task MyMethodAsync_ReturnsFalse()
{
  var objectUnderTest = ...;
  bool result = await objectUnderTest.MyMethodAsync();
  Assert.IsFalse(result);
}
```

单元测试框架会注意到方法的返回类型是`Task`，并智能地等待任务完成，然后标记测试为“成功”或“失败”。

如果你的单元测试框架不支持`async Task`单元测试，那么它需要一些帮助来等待正在测试的异步操作。一种选择是使用`GetAwaiter().GetResult()`来同步阻塞任务；如果你使用`GetAwaiter().GetResult()`而不是`Wait()`，它会避免`AggregateException`包装器（如果任务有异常的话）。然而，我更喜欢使用`Nito.AsyncEx` NuGet 包中的`AsyncContext`类型：

```cs
[TestMethod]
public void MyMethodAsync_ReturnsFalse()
{
  AsyncContext.Run(async () =>
  {
    var objectUnderTest = ...;
    bool result = await objectUnderTest.MyMethodAsync();
    Assert.IsFalse(result);
  });
}
```

`AsyncContext.Run`会等待所有异步方法完成。

## 讨论

最初，模拟异步依赖可能有点笨拙。建议至少测试你的方法如何响应同步成功（使用`Task.FromResult`进行模拟）、同步错误（使用`Task.FromException`进行模拟）和异步成功（使用`Task.Yield`进行模拟并返回值）。你可以在第 2.2 节中找到有关`Task.FromResult`和`Task.FromException`的覆盖率。`Task.Yield`可用于强制异步行为，主要用于单元测试：

```cs
interface IMyInterface
{
  Task<int> SomethingAsync();
}

class SynchronousSuccess : IMyInterface
{
  public Task<int> SomethingAsync()
  {
    return Task.FromResult(13);
  }
}

class SynchronousError : IMyInterface
{
  public Task<int> SomethingAsync()
  {
    return Task.FromException<int>(new InvalidOperationException());
  }
}

class AsynchronousSuccess : IMyInterface
{
  public async Task<int> SomethingAsync()
  {
    await Task.Yield(); // Force asynchronous behavior.
    return 13;
  }
}
```

在测试异步代码时，死锁和竞争条件可能比测试同步代码更容易出现。我发现每个测试设置超时时间非常有用；在 Visual Studio 中，你可以向解决方案添加一个测试设置文件，以便设置单独的测试超时时间。默认值相当高；我通常将每个测试的超时设置为两秒。

###### 提示

`AsyncContext`类型位于[`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中。

## 参见

第 7.2 节涵盖了预期失败的异步方法的单元测试。

# 7.2 测试失败预期的异步方法

## 问题

你需要编写一个单元测试来检查`async Task`方法的特定失败情况。

## 解决方案

如果你在桌面或服务器开发中，MSTest 确实支持通过常规的`ExpectedExceptionAttribute`进行失败测试：

```cs
// Not a recommended solution; see below.
[TestMethod]
[ExpectedException(typeof(DivideByZeroException))]
public async Task Divide_WhenDenominatorIsZero_ThrowsDivideByZero()
{
  await MyClass.DivideAsync(4, 0);
}
```

然而，这个解决方案并不是最佳选择：`ExpectedException`实际上设计不佳。它期望的异常可能由单元测试方法调用的*任何*方法抛出。更好的设计是检查*特定*代码块是否抛出了该异常，而不是整个单元测试。

大多数现代单元测试框架都以某种形式包含`Assert.ThrowsAsync<TException>`。例如，你可以像这样使用 xUnit 的`ThrowsAsync`：

```cs
[Fact]
public async Task Divide_WhenDenominatorIsZero_ThrowsDivideByZero()
{
  await Assert.ThrowsAsync<DivideByZeroException>(async () =>
  {
    await MyClass.DivideAsync(4, 0);
  });
}
```

###### 警告

不要忘记`await` `ThrowsAsync` 返回的任务！`await` 将传播任何检测到的断言失败。如果您忽略`await`并忽略编译器警告，那么您的单元测试将始终在不管方法行为如何的情况下静默成功。

不幸的是，其他几个单元测试框架不包括与 `async` 兼容的 `ThrowsAsync` 等效项。如果您发现自己处于这种情况中，请创建您自己的：

```cs
/// <summary>
/// Ensures that an asynchronous delegate throws an exception.
/// </summary>
/// <typeparam name="TException">
/// The type of exception to expect.
/// </typeparam>
/// <param name="action">The asynchronous delegate to test.</param>
/// <param name="allowDerivedTypes">
/// Whether derived types should be accepted.
/// </param>
public static async Task<TException> ThrowsAsync<TException>(Func<Task> action,
    bool allowDerivedTypes = true)
    where TException : Exception
{
  try
  {
    await action();
    var name = typeof(Exception).Name;
    Assert.Fail($"Delegate did not throw expected exception {name}.");
    return null;
  }
  catch (Exception ex)
  {
    if (allowDerivedTypes && !(ex is TException))
      Assert.Fail($"Delegate threw exception of type {ex.GetType().Name}" +
          $", but {typeof(TException).Name} or a derived type was expected.");
    if (!allowDerivedTypes && ex.GetType() != typeof(TException))
      Assert.Fail($"Delegate threw exception of type {ex.GetType().Name}" +
          $", but {typeof(TException).Name} was expected.");
    return (TException)ex;
  }
}
```

您可以像使用任何其他 `Assert.ThrowsAsync<TException>` 方法一样使用此方法。不要忘记`await`返回值！

## 讨论

测试错误处理与测试成功场景一样重要。有人甚至会说更重要，因为成功的场景是软件发布前每个人都会尝试的场景。如果您的应用行为异常，那可能是由于意外的错误情况。

然而，我鼓励开发人员不要再使用 `ExpectedException`。最好在特定点测试抛出异常，而不是在测试过程中的任何时候测试异常。请使用 `ThrowsAsync`（或您的单元测试框架中的等效项），或者像最后一个代码示例中一样使用 `ThrowsAsync` 的实现。

## 另请参阅

Recipe 7.1 涵盖了单元测试异步方法的基础知识。

# 7.3 单元测试 `async void` 方法

## 问题

您有一个需要进行单元测试的 `async void` 方法。

## 解决方案

停止。

而不是解决这个问题，您应尽全力避免它。如果可能将您的 `async void` 方法更改为 `async Task` 方法，则应这样做。

如果您的方法*必须*是 `async void`（例如，为了满足接口方法签名），则考虑编写两个方法：一个是包含所有逻辑的 `async Task` 方法，另一个是只调用 `async Task` 方法并等待结果的 `async void` 包装器。`async void` 方法满足架构要求，而 `async Task` 方法（包含所有逻辑）是可测试的。

如果无法更改您的方法并且*必须*对 `async void` 方法进行单元测试，则有一种方法可以实现。您可以使用 `Nito.AsyncEx` 库中的 `AsyncContext` 类：

```cs
// Not a recommended solution; see the rest of this section.
[TestMethod]
public void MyMethodAsync_DoesNotThrow()
{
  AsyncContext.Run(() =>
  {
    var objectUnderTest = new Sut(); // ...;
    objectUnderTest.MyVoidMethodAsync();
  });
}
```

`AsyncContext` 类型将等待所有异步操作完成（包括 `async void` 方法）并传播它们引发的异常。

###### 提示

[`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中包含 `AsyncContext` 类型。

## 讨论

在 `async` 代码中的一个关键指导原则是避免使用 `async void`。我强烈建议您重构代码，而不是为了单元测试 `async void` 方法而使用 `AsyncContext`。

## 另请参阅

Recipe 7.1 涵盖了单元测试异步方法的基础知识。

# 7.4 单元测试数据流网格

## 问题

您的应用程序中有一个数据流网格，并且您需要验证其正常工作。

## 解决方案

数据流网格是独立的：它们有自己的生命周期，并且本质上是异步的。因此，测试它们的最自然方式是使用异步单元测试。以下单元测试验证来自 Recipe 5.6 的自定义数据流块：

```cs
[TestMethod]
public async Task MyCustomBlock_AddsOneToDataItems()
{
  var myCustomBlock = CreateMyCustomBlock();

  myCustomBlock.Post(3);
  myCustomBlock.Post(13);
  myCustomBlock.Complete();

  Assert.AreEqual(4, myCustomBlock.Receive());
  Assert.AreEqual(14, myCustomBlock.Receive());
  await myCustomBlock.Completion;
}
```

单元测试失败并不是那么简单。这是因为数据流网格中的异常每次传播到下一个块时都会被包装在另一个`AggregateException`中。以下示例使用一个辅助方法来确保异常将丢弃数据并通过自定义块传播：

```cs
[TestMethod]
public async Task MyCustomBlock_Fault_DiscardsDataAndFaults()
{
  var myCustomBlock = CreateMyCustomBlock();

  myCustomBlock.Post(3);
  myCustomBlock.Post(13);
  (myCustomBlock as IDataflowBlock).Fault(new InvalidOperationException());

  try
  {
    await myCustomBlock.Completion;
  }
  catch (AggregateException ex)
  {
    AssertExceptionIs<InvalidOperationException>(
        ex.Flatten().InnerException, false);
  }
}

public static void AssertExceptionIs<TException>(Exception ex,
    bool allowDerivedTypes = true)
{
  if (allowDerivedTypes && !(ex is TException))
    Assert.Fail($"Exception is of type {ex.GetType().Name}, but " +
        $"{typeof(TException).Name} or a derived type was expected.");
  if (!allowDerivedTypes && ex.GetType() != typeof(TException))
    Assert.Fail($"Exception is of type {ex.GetType().Name}, but " +
        $"{typeof(TException).Name} was expected.");
}
```

## 讨论

直接对数据流网格进行单元测试是可行的，但有些笨拙。如果您的网格是较大组件的一部分，那么您可能会发现仅对较大组件进行单元测试（隐式测试网格）更容易。但如果您正在开发可重用的自定义块或网格，则应使用类似前面的单元测试。

## 参见

Recipe 7.1 涵盖了对`async`方法的单元测试。

# 7.5 单元测试 System.Reactive 可观察序列

## 问题

您的程序的一部分正在使用`IObservable<T>`，您需要找到一种方法来对其进行单元测试。

## 解决方案

System.Reactive 有许多操作符来生成序列（例如`Return`）和其他可以将响应序列转换为常规集合或项的操作符（例如`SingleAsync`）。你可以使用诸如`Return`的操作符来创建可观察依赖的存根，并使用诸如`SingleAsync`的操作符来测试输出。

考虑以下代码，它将 HTTP 服务作为依赖项，并对 HTTP 调用应用超时：

```cs
public interface IHttpService
{
  IObservable<string> GetString(string url);
}

public class MyTimeoutClass
{
  private readonly IHttpService _httpService;

  public MyTimeoutClass(IHttpService httpService)
  {
    _httpService = httpService;
  }

  public IObservable<string> GetStringWithTimeout(string url)
  {
    return _httpService.GetString(url)
        .Timeout(TimeSpan.FromSeconds(1));
  }
}
```

受测试系统是`MyTimeoutClass`，它消耗一个可观察依赖项并产生一个可观察输出。

`Return`操作符创建一个包含单个元素的冷序列；您可以使用`Return`来构建一个简单的存根。`SingleAsync`操作符返回一个在下一个事件到达时完成的`Task<T>`。`SingleAsync`可以用于像以下这样的简单单元测试：

```cs
class SuccessHttpServiceStub : IHttpService
{
  public IObservable<string> GetString(string url)
  {
    return Observable.Return("stub");
  }
}

[TestMethod]
public async Task MyTimeoutClass_SuccessfulGet_ReturnsResult()
{
  var stub = new SuccessHttpServiceStub();
  var my = new MyTimeoutClass(stub);

  var result = await my.GetStringWithTimeout("http://www.example.com/")
      .SingleAsync();

  Assert.AreEqual("stub", result);
}
```

在存根代码中另一个重要的操作符是`Throw`，它返回以错误结束的可观察序列。该操作符使我们能够对错误情况进行单元测试。以下示例使用了来自 Recipe 7.2 的`ThrowsAsync`辅助方法：

```cs
private class FailureHttpServiceStub : IHttpService
{
  public IObservable<string> GetString(string url)
  {
    return Observable.Throw<string>(new HttpRequestException());
  }
}

[TestMethod]
public async Task MyTimeoutClass_FailedGet_PropagatesFailure()
{
  var stub = new FailureHttpServiceStub();
  var my = new MyTimeoutClass(stub);

  await ThrowsAsync<HttpRequestException>(async () =>
  {
    await my.GetStringWithTimeout("http://www.example.com/")
        .SingleAsync();
  });
}
```

## 讨论

`Return`和`Throw`非常适合创建可观察存根，而`SingleAsync`是测试具有`async`单元测试的简单方法。它们对于简单的可观察序列是一个很好的组合，但是一旦涉及到*时间*，它们就无法很好地支持。例如，如果您想测试`MyTimeoutClass`的超时功能，单元测试将需要等待一段时间。然而，这是一个不好的方法：它通过引入竞争条件使您的单元测试不可靠，并且随着添加更多单元测试，它不会很好地扩展。Recipe 7.6 介绍了 System.Reactive 如何特殊处理使您能够模拟时间本身的方法。

## 参见

Recipe 7.1 讨论了对 `async` 方法进行单元测试，这与等待 `SingleAsync` 的单元测试非常相似。

Recipe 7.6 讨论了依赖于时间推移的可观测序列的单元测试。

# 7.6 使用伪造调度器对 System.Reactive 可观测对象进行单元测试

## 问题

你有一个依赖于时间的可观测对象，并且想编写一个不依赖于时间的单元测试。依赖于时间的可观测对象包括使用超时、窗口/缓冲以及节流/抽样的对象。你希望对这些进行单元测试，但不希望单元测试运行时间过长。

## 解决方案

当然，你可以在单元测试中添加延迟；然而，这种方法存在两个问题：1）单元测试运行时间长，2）由于单元测试同时运行，造成了竞争条件，使得时间难以预测。

System.Reactive（Rx）库设计时考虑了测试；事实上，Rx 库本身经过了大量单元测试。为了实现彻底的单元测试，Rx 引入了一个称为 *scheduler* 的概念，并且 *每个* 处理时间的 Rx 运算符都是使用这个抽象调度器实现的。

为了使你的可观测对象可测试，你需要允许调用者指定调度器。例如，你可以从 Recipe 7.5 中获取 `MyTimeoutClass` 并添加一个调度器：

```cs
public interface IHttpService
{
  IObservable<string> GetString(string url);
}

public class MyTimeoutClass
{
  private readonly IHttpService _httpService;

  public MyTimeoutClass(IHttpService httpService)
  {
    _httpService = httpService;
  }

  public IObservable<string> GetStringWithTimeout(string url,
      IScheduler scheduler = null)
  {
    return _httpService.GetString(url)
        .Timeout(TimeSpan.FromSeconds(1), scheduler ?? Scheduler.Default);
  }
}
```

接下来，你可以修改你的 HTTP 服务存根，使其也能理解调度，然后引入可变的延迟：

```cs
private class SuccessHttpServiceStub : IHttpService
{
  public IScheduler Scheduler { get; set; }
  public TimeSpan Delay { get; set; }

  public IObservable<string> GetString(string url)
  {
    return Observable.Return("stub")
        .Delay(Delay, Scheduler);
  }
}
```

现在你可以继续使用 `TestScheduler`，这是 System.Reactive 库中包含的一种类型。 `TestScheduler` 让你强大地控制（虚拟）时间。

###### 提示

`TestScheduler` 与 System.Reactive 的其余部分不在同一个 NuGet 包中；你需要安装 [`Microsoft.Reactive.Testing`](http://bit.ly/ms-react-test) NuGet 包。

`TestScheduler` 让你完全控制时间，但通常你只需设置好你的代码，然后调用 `TestScheduler.Start`。 `Start` 会虚拟推进时间，直到所有操作完成。一个简单的成功测试用例如下所示：

```cs
[TestMethod]
public void MyTimeoutClass_SuccessfulGetShortDelay_ReturnsResult()
{
  var scheduler = new TestScheduler();
  var stub = new SuccessHttpServiceStub
  {
    Scheduler = scheduler,
    Delay = TimeSpan.FromSeconds(0.5),
  };
  var my = new MyTimeoutClass(stub);
  string result = null;

  my.GetStringWithTimeout("http://www.example.com/", scheduler)
      .Subscribe(r => { result = r; });

  scheduler.Start();

  Assert.AreEqual("stub", result);
}
```

该代码模拟了半秒钟的网络延迟。需要注意的是，这个单元测试 *不会* 花费半秒钟来运行；在我的机器上，大约只需 70 毫秒。半秒钟的延迟仅存在于虚拟时间中。这个单元测试的另一个显著区别是它不是异步的；因为你使用了 `TestScheduler`，所有测试可以立即完成。

现在一切都在使用测试调度器，很容易测试超时情况：

```cs
[TestMethod]
public void MyTimeoutClass_SuccessfulGetLongDelay_ThrowsTimeoutException()
{
  var scheduler = new TestScheduler();
  var stub = new SuccessHttpServiceStub
  {
    Scheduler = scheduler,
    Delay = TimeSpan.FromSeconds(1.5),
  };
  var my = new MyTimeoutClass(stub);
  Exception result = null;

  my.GetStringWithTimeout("http://www.example.com/", scheduler)
      .Subscribe(_ => Assert.Fail("Received value"), ex => { result = ex; });

  scheduler.Start();

  Assert.IsInstanceOfType(result, typeof(TimeoutException));
}
```

再次强调，前面的单元测试不需要花费 1 秒（或 1.5 秒）来运行；它立即执行，使用虚拟时间。

## 讨论

在这个配方中，我们只是浅尝了一下 System.Reactive 的调度器和虚拟时间。我建议您在开始编写 System.Reactive 代码时就开始进行单元测试；随着代码变得越来越复杂，您可以放心使用 `Microsoft.Reactive.Testing` 来处理它。

`TestScheduler` 还有 `AdvanceTo` 和 `AdvanceBy` 方法，这些方法使您能够逐步在虚拟时间中前进。在某些情况下，这可能很有用，但您应该努力让您的单元测试只测试一件事情。要测试超时，您可以编写一个单元测试，部分前进 `TestScheduler` 并确保超时不会过早发生，然后前进 `TestScheduler` 超过超时值并确保超时确实发生。然而，我更喜欢尽可能运行分开的单元测试；例如，一个单元测试确保超时不会过早发生，另一个单元测试确保超时稍后发生。

## 另请参见

配方 7.5 覆盖了观察序列单元测试的基础知识。
