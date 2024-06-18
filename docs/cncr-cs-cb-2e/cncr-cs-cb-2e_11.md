# 第十一章：友好的函数式面向对象编程

现代程序需要异步编程；如今，服务器必须比以往更好地扩展，终端用户应用程序必须比以往更具响应性。开发人员发现他们必须学习异步编程，当他们探索这个世界时，他们发现它经常与他们习惯的传统面向对象编程相冲突。

这样做的核心原因是因为异步编程是函数式的。通过“函数式”，我不是指“它有效”；我指的是它是一种函数式编程风格，而不是过程式编程风格。很多开发人员在大学学习了基本的函数式编程，之后几乎没有再碰过。如果像`(car (cdr '(3 5 7)))`这样的代码让你感到不安，因为被压抑的记忆涌入脑海，那么你可能属于这一类别。但不要害怕；一旦习惯了，现代异步编程并不那么难。

`async`的主要突破在于你仍然可以在编写和理解异步方法时以过程化方式思考。这使得编写和理解异步方法变得更容易。然而，在底层，异步代码仍然具有函数式的特性，当人们试图将`async`方法强行融入传统面向对象设计时，这会导致一些问题。本章的示例处理异步代码与面向对象编程相冲突的摩擦点。

当将现有的面向对象编码基础转换为友好的`async`编码基础时，这些摩擦点尤为明显。

# 11.1 异步接口和继承

## 问题

你有一个在接口或基类中的方法，你想要将其变成异步的。

## 解决方案

理解这个问题及其解决方案的关键是意识到`async`是一个实现细节。`async`关键字只能应用于具有实现的方法；不可能将其应用于抽象方法或接口方法（除非它们有默认实现）。但是，你可以定义一个与`async`方法具有相同签名的方法，只是没有`async`关键字。

记住*类型*是可等待的，而不是*方法*。你可以`await`一个方法返回的`Task`，无论该方法是否使用`async`实现。因此，一个接口或抽象方法可以直接返回一个`Task`（或`Task<T>`），并且该方法的返回值是可等待的。

以下代码定义了一个带有异步方法的接口（不带`async`关键字），该接口的实现（带有`async`），以及一个独立的方法，该方法通过`await`消耗接口的方法：

```cs
interface IMyAsyncInterface
{
  Task<int> CountBytesAsync(HttpClient client, string url);
}

class MyAsyncClass : IMyAsyncInterface
{
  public async Task<int> CountBytesAsync(HttpClient client, string url)
  {
    var bytes = await client.GetByteArrayAsync(url);
    return bytes.Length;
  }
}

async Task UseMyInterfaceAsync(HttpClient client, IMyAsyncInterface service)
{
  var result = await service.CountBytesAsync(client, "http://www.example.com");
  Trace.WriteLine(result);
}
```

这个模式也适用于基类中的抽象方法。

异步方法签名仅意味着实现*可能*是异步的。如果实际实现没有真正的异步工作要做，那么它也可能是同步的。例如，一个测试存根可以通过使用类似`FromResult`的东西来实现相同的接口（不带`async`）：

```cs
class MyAsyncClassStub : IMyAsyncInterface
{
  public Task<int> CountBytesAsync(HttpClient client, string url)
  {
    return Task.FromResult(13);
  }
}
```

## 讨论

在撰写本文时，`async` 和 `await` 仍在不断普及中。随着异步方法变得更加普遍，接口和基类上的异步方法也将变得更加常见。只要记住可等待的是返回类型（而不是方法），以及异步方法定义可以是异步的或同步的，它们并不难处理。

## 参见

Recipe 2.2 介绍了返回已完成任务，使用同步代码实现异步方法签名。

# 11.2 异步构造：工厂

## 问题

您正在编写一个需要在其构造函数中完成一些异步工作的类型。

## 解决方案

构造函数不能是`async`，也不能使用`await`关键字。在构造函数中使用`await`肯定会很有用，但这将大大改变 C#语言。

一种可能性是拥有一个构造函数和一个`async`初始化方法，以便可以像这样使用该类型：

```cs
var instance = new MyAsyncClass();
await instance.InitializeAsync();
```

这种方法有一些缺点。很容易忘记调用`InitializeAsync`方法，并且在构造完成后实例不能立即可用。

更好的解决方案是使类型成为其自身的工厂。以下类型展示了异步工厂方法模式：

```cs
class MyAsyncClass
{
  private MyAsyncClass()
  {
  }

  private async Task<MyAsyncClass> InitializeAsync()
  {
    await Task.Delay(TimeSpan.FromSeconds(1));
    return this;
  }

  public static Task<MyAsyncClass> CreateAsync()
  {
    var result = new MyAsyncClass();
    return result.InitializeAsync();
  }
}
```

构造函数和`InitializeAsync`方法都是`private`的，以防其他代码误用它们；因此，创建实例的唯一方式是通过静态的`CreateAsync`工厂方法。调用代码在初始化完成之前无法访问实例。

其他代码可以像这样创建一个实例：

```cs
MyAsyncClass instance = await MyAsyncClass.CreateAsync();
```

## 讨论

此模式的主要优点是其他代码无法获得未初始化的`MyAsyncClass`实例。这就是为什么我在可以使用它时更喜欢这种模式而不是其他方法的主要原因。

不幸的是，这种方法在某些场景下不起作用，特别是当您的代码使用依赖注入提供程序时。没有主要的依赖注入或控制反转库能与`async`代码配合工作。如果您发现自己处于这些情况之一，那么可以考虑几种替代方案。

如果您正在创建的实例实际上是一个共享资源，那么您可以使用 Recipe 14.1 中讨论的异步延迟类型。否则，您可以使用 Recipe 11.3 中讨论的异步初始化模式。

这是一个*不*推荐的示例：

```cs
class MyAsyncClass
{
  public MyAsyncClass()
  {
    InitializeAsync();
  }

  // BAD CODE!!
  private async void InitializeAsync()
  {
    await Task.Delay(TimeSpan.FromSeconds(1));
  }
}
```

乍一看，这似乎是一个合理的方法：您得到一个启动异步操作的常规构造函数；然而，由于使用了`async void`，存在几个缺点。第一个问题是当构造函数完成时，实例仍在异步初始化中，并且没有明显的方法来确定异步初始化何时完成。第二个问题是错误处理：`InitializeAsync`引发的任何异常不能被围绕对象构造的任何`catch`子句捕获。

## 参见

Recipe 11.3 讲述了异步初始化模式，这是一种与依赖注入/控制反转容器一起使用的异步构造方式。

Recipe 14.1 讲述了异步延迟初始化，这是如果实例在概念上是共享资源或服务，则是一种可行的解决方案。

# 11.3 异步构造：异步初始化模式

## 问题

您正在编写一个类型，其构造函数需要进行一些异步工作，但不能使用异步工厂模式（Recipe 11.2）因为实例是通过反射创建的（例如，依赖注入/控制反转库，数据绑定，`Activator.CreateInstance`等）。

## 解决方案

当您遇到这种情况时，*必须*返回一个未初始化的实例，尽管可以通过应用常见模式来缓解这种情况：异步初始化模式。每个需要异步初始化的类型都应定义一个属性，如下所示：

```cs
Task Initialization { get; }
```

我通常喜欢在需要异步初始化的类型的标记接口中定义这个：

```cs
/// <summary>
/// Marks a type as requiring asynchronous initialization
/// and provides the result of that initialization.
/// </summary>
public interface IAsyncInitialization
{
  /// <summary>
  /// The result of the asynchronous initialization of this instance.
  /// </summary>
  Task Initialization { get; }
}
```

当您实现此模式时，应在构造函数中启动初始化（并分配`Initialization`属性）。异步初始化的结果（包括任何异常）通过该`Initialization`属性公开。以下是使用异步初始化实现简单类型的示例实现：

```cs
class MyFundamentalType : IMyFundamentalType, IAsyncInitialization
{
  public MyFundamentalType()
  {
    Initialization = InitializeAsync();
  }

  public Task Initialization { get; private set; }

  private async Task InitializeAsync()
  {
    // Asynchronously initialize this instance.
    await Task.Delay(TimeSpan.FromSeconds(1));
  }
}
```

如果您使用依赖注入/控制反转库，可以使用以下代码创建和初始化此类型的实例：

```cs
IMyFundamentalType instance = UltimateDIFactory.Create<IMyFundamentalType>();
var instanceAsyncInit = instance as IAsyncInitialization;
if (instanceAsyncInit != null)
  await instanceAsyncInit.Initialization;
```

您可以将此模式扩展到允许异步初始化的类型组合。在以下示例中，定义了另一种依赖于`IMyFundamentalType`的类型：

```cs
class MyComposedType : IMyComposedType, IAsyncInitialization
{
  private readonly IMyFundamentalType _fundamental;

  public MyComposedType(IMyFundamentalType fundamental)
  {
    _fundamental = fundamental;
    Initialization = InitializeAsync();
  }

  public Task Initialization { get; private set; }

  private async Task InitializeAsync()
  {
    // Asynchronously wait for the fundamental instance to initialize,
    //  if necessary.
    var fundamentalAsyncInit = _fundamental as IAsyncInitialization;
    if (fundamentalAsyncInit != null)
      await fundamentalAsyncInit.Initialization;

    // Do our own initialization (synchronous or asynchronous).
    ...
  }
}
```

组合类型在所有组件初始化完成之前会等待。遵循的规则是每个组件都应在`InitializeAsync`结束时初始化完成。这确保了所有依赖类型作为组合初始化的一部分被初始化。任何组件初始化引发的异常会传播到组合类型的初始化过程中。

## 讨论

如果可能的话，我建议使用异步工厂（Recipe 11.2）或异步延迟初始化（Recipe 14.1）而不是这个解决方案。这些是最佳方法，因为您永远不会暴露未初始化的实例。然而，如果您的实例是由依赖注入/控制反转、数据绑定等创建的，那么您将被迫暴露未初始化的实例，在这种情况下，我建议使用本配方中的异步初始化模式。

从异步接口的配方（Recipe 11.1）记住，异步方法签名只意味着该方法可能是异步的。`MyComposedType.InitializeAsync`代码是一个很好的例子：如果`IMyFundamentalType`实例不同时实现`IAsyncInitialization`且`MyComposedType`本身没有异步初始化，则其`InitializeAsync`方法将同步完成。

检查实例是否实现`IAsyncInitialization`并对其进行初始化的代码有点笨拙，当有一个依赖于更多组件的组合类型时，情况会变得更加复杂。可以很容易地创建一个辅助方法来简化代码：

```cs
public static class AsyncInitialization
{
  public static Task WhenAllInitializedAsync(params object[] instances)
  {
    return Task.WhenAll(instances
        .OfType<IAsyncInitialization>()
        .Select(x => x.Initialization));
  }
}
```

您可以调用`InitializeAllAsync`并传入您想要初始化的任何实例；该方法将忽略不实现`IAsyncInitialization`的实例。然后，依赖于三个注入实例的组合类型的初始化代码可以看起来像以下内容：

```cs
private async Task InitializeAsync()
{
 // Asynchronously wait for all 3 instances to initialize, if necessary.
 await AsyncInitialization.WhenAllInitializedAsync(_fundamental,
     _anotherType, _yetAnother);

 // Do our own initialization (synchronous or asynchronous).
 ...
}
```

## 另请参阅

Recipe 11.2 介绍了异步工厂，这是一种进行异步构建而不暴露未初始化实例的方法。

Recipe 14.1 介绍了异步延迟初始化，如果实例是共享资源或服务，则可以使用该方法。

Recipe 11.1 介绍了异步接口。

# 11.4 异步属性

## 问题

您有一个您想要使`async`的属性。该属性未用于数据绑定。

## 解决方案

这是在将现有代码转换为使用`async`时经常遇到的问题；在这种情况下，您有一个其 getter 调用现在是异步的方法的属性。然而，“异步属性”这种东西并不存在。不能在属性上使用`async`关键字，而这是一个好事。属性的 getter 应该返回当前值；它们不应该启动后台操作：

```cs
// What we think we want (does not compile).
public int Data
{
  async get
  {
    await Task.Delay(TimeSpan.FromSeconds(1));
    return 13;
  }
}
```

当您发现您的代码需要一个“异步属性”时，您的代码真正*需要*的是略有不同。解决方案取决于您的属性值是否需要被评估一次或多次；您可以在以下语义之间做出选择：

+   每次读取时异步评估的值

+   一次异步评估并为将来访问缓存的值

如果你的“异步属性”在每次读取时都需要启动一个新的（异步）评估，那么它不是一个*属性*；它是一个伪装成*方法*的属性。如果在将同步代码转换为异步时遇到了这种情况，那么现在是时候承认原始设计实际上是错误的了；该属性本来应该是一个方法：

```cs
// As an asynchronous method.
public async Task<int> GetDataAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1));
  return 13;
}
```

返回`Task<int>`直接从属性中是*可能*的，如下面的代码所示：

```cs
// This "async property" is an asynchronous method.
// This "async property" is a Task-returning property.
public Task<int> Data
{
  get { return GetDataAsync(); }
}

private async Task<int> GetDataAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1));
  return 13;
}
```

然而，我不建议这种方法。如果每次访问属性都会启动一个新的异步操作，那么这个“属性”实际上应该是一个方法。它是一个异步方法使得每次都启动一个新的异步操作更加清晰，因此 API 不会误导。食谱 11.3 和 11.6 确实使用返回任务的属性，但这些属性适用于整个实例；它们不会在每次读取时启动新的异步操作。

有时候你希望每次检索属性值时都对其进行评估。其他时候，你希望属性仅启动一次（异步）评估并缓存该结果以供将来使用。在这种情况下，你可以使用异步懒初始化。这种解决方案在食谱 14.1 中有详细说明，但与此同时，这里是一个示例，展示了代码的样子：

```cs
// As a cached value
public AsyncLazy<int> Data
{
  get { return _data; }
}

private readonly AsyncLazy<int> _data =
    new AsyncLazy<int>(async () =>
    {
      await Task.Delay(TimeSpan.FromSeconds(1));
      return 13;
    });
```

代码将仅执行一次异步评估，然后将同一值返回给所有调用者。调用代码看起来像下面这样：

```cs
int value = await instance.Data;
```

在这种情况下，属性语法是适当的，因为只有一个评估在进行。

## 讨论

自问一个重要的问题是是否读取属性应该启动一个新的异步操作；如果答案是肯定的，那么使用异步*方法*而不是属性。如果属性应该作为延迟评估的缓存，则使用异步初始化（参见食谱 14.1）。在这个食谱中，我没有涵盖用于数据绑定的属性；我在食谱 14.3 中涵盖了这些内容。

当你将同步属性转换为“异步属性”时，这里有一个*不*要做的示例：

```cs
private async Task<int> GetDataAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(1));
  return 13;
}

public int Data
{
  // BAD CODE!!
  get { return GetDataAsync().Result; }
}
```

当谈论在`async`代码中的属性时，值得考虑状态如何与异步代码相关联。如果你正在将同步代码库转换为异步，这一点尤为重要。考虑你在 API 中暴露的任何状态（例如通过属性）；对于每个状态，问自己，具有异步操作进行中的对象的当前状态是什么？并没有正确的答案，但重要的是考虑你想要的语义，并进行文档记录。

例如，考虑`Stream.Position`，它表示流指针的当前偏移量。使用同步 API 时，当你调用`Stream.Read`或`Stream.Write`时，读取/写入完成并且`Stream.Position`更新以反映新位置，然后`Read`或`Write`方法返回。这对同步代码的语义很清晰。

现在，考虑`Stream.ReadAsync`和`Stream.WriteAsync`：何时应更新`Stream.Position`？当读取/写入操作完成时，还是在实际发生之前？如果在操作完成之前更新它，`Stream.Position`会在`ReadAsync`/`WriteAsync`返回时同步更新，还是可能稍后才会发生？

这是一个很好的例子，展示了一个公开状态的属性在同步代码中具有完全清晰的语义，但在异步代码中却没有明显正确的语义。这并不是世界末日——你只需要在使你的类型支持异步时考虑整个 API，并记录你选择的语义。

## 另请参阅

Recipe 14.1 详细介绍了异步延迟初始化。

Recipe 14.3 介绍了需要支持数据绑定的“异步属性”。

# 11.5 异步事件

## 问题

当你需要使用可能是`async`的处理程序处理事件，并且需要检测处理程序是否已完成时，你有一个事件。请注意，在引发事件时，这是一个罕见的情况；通常在引发事件时，你并不关心处理程序何时完成。

## 解决方案

检测`async void`处理程序何时返回是不可行的，因此你需要一些替代方法来检测异步处理程序何时完成。Universal Windows 平台引入了一个称为*deferrals*的概念，你可以使用它来跟踪异步处理程序。异步处理程序在其第一个`await`之前分配一个延迟，并在完成时通知延迟。同步处理程序不需要使用延迟。

`Nito.AsyncEx`库包含一个名为`DeferralManager`的类型，被引发事件的组件使用。这个延迟管理器允许事件处理程序分配延迟，并跟踪所有延迟何时完成。

对于每个需要等待处理程序完成的事件，你首先扩展你的事件参数类型：

```cs
public class MyEventArgs : EventArgs, IDeferralSource
{
  private readonly DeferralManager _deferrals = new DeferralManager();

  ... // Your own constructors and properties

  public IDisposable GetDeferral()
  {
    return _deferrals.DeferralSource.GetDeferral();
  }

  internal Task WaitForDeferralsAsync()
  {
    return _deferrals.WaitForDeferralsAsync();
  }
}
```

当处理异步事件处理程序时，最好使你的事件参数类型是线程安全的。实现这一点的最简单方法是使其不可变（即，其所有属性都是只读的）。

然后，每次你引发事件时，你可以（异步地）等待所有异步事件处理程序完成。以下代码将在没有处理程序时返回一个已完成的任务；否则，它将创建你事件参数类型的新实例，传递给处理程序，并等待任何异步处理程序完成：

```cs
public event EventHandler<MyEventArgs> MyEvent;

private async Task RaiseMyEventAsync()
{
  EventHandler<MyEventArgs> handler = MyEvent;
  if (handler == null)
    return;

  var args = new MyEventArgs(...);
  handler(this, args);
  await args.WaitForDeferralsAsync();
}
```

然后，异步事件处理程序可以在`using`块内使用延期；延期在被处理时通知延期管理器：

```cs
async void AsyncHandler(object sender, MyEventArgs args)
{
  using IDisposable deferral = args.GetDeferral();
  await Task.Delay(TimeSpan.FromSeconds(2));
}
```

这与 Universal Windows 延期的工作方式略有不同。在 Universal Windows API 中，每个需要延期的事件定义其自己的延期类型，并且该延期类型有一个显式的`Complete`方法，而不是`IDisposable`。

## 讨论

在 .NET 中，有两种逻辑上不同的事件类型，它们的语义差别很大。我称之为*通知事件*和*命令事件*；这不是官方术语，只是我为了清晰起见选择的术语。通知事件是为了通知其他组件某种情况而引发的事件。通知是单向的；事件的发送者不在乎是否有任何事件接收者。在通知事件中，发送者和接收者可以完全断开连接。大多数事件都是通知事件；一个例子是按钮点击。

相反，命令事件是为了代表发送组件实现某些功能而引发的事件。命令事件在术语的真正意义上并不是“事件”，尽管它们通常被实现为 .NET 事件。命令的发送者必须等待接收者处理完它才能继续。如果你使用事件来实现访问者模式，那么这些就是命令事件。生命周期事件也是命令事件，因此 ASP.NET 页面生命周期事件和许多 UI 框架事件，如 Xamarin 的`Application.PageAppearing`，属于这一类别。任何实际上是实现的 UI 框架事件也是命令事件（例如`BackgroundWorker.DoWork`）。

通知事件不需要任何特殊的代码来启用异步处理程序；事件处理程序可以是`async void`并且能够正常工作。当事件发送者引发事件时，异步事件处理程序不会立即完成，但这并不重要，因为它们只是通知事件。所以，如果你的事件是通知事件，你需要做的工作总量是：什么都不用做。

命令事件是另一回事。当你有一个命令事件时，你需要一种方法来检测处理程序何时完成。前面使用延期的解决方案应该仅用于命令事件。

###### 提示

[`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中的`DeferralManager`类型。

## 参见

第二章介绍了异步编程的基础知识。

# 11.6 异步处理

## 问题

你有一个具有异步操作但还需要启用其资源释放的类型。

## 解决方案

在处理实例的释放时，有几个常见的选项：你可以将释放视为应用于所有现有操作的取消请求，或者你可以实现真正的*异步释放*。

将释放视为取消在 Windows 上有着历史悠久的先例；例如文件流和套接字在关闭时取消任何现有的读取或写入。通过定义自己的私有`CancellationTokenSource`并将该令牌传递给内部操作，你可以在 .NET 中实现类似的效果。通过以下代码，`Dispose`将取消操作但不会等待这些操作完成：

```cs
class MyClass : IDisposable
{
  private readonly CancellationTokenSource _disposeCts =
      new CancellationTokenSource();

  public async Task<int> CalculateValueAsync()
  {
    await Task.Delay(TimeSpan.FromSeconds(2), _disposeCts.Token);
    return 13;
  }

  public void Dispose()
  {
    _disposeCts.Cancel();
  }
}
```

代码展示了围绕`Dispose`的基本模式。在实际应用程序中，你应该进行检查以确保对象尚未被释放，并允许用户提供自己的`CancellationToken`（使用来自菜谱 10.8 的技术）：

```cs
public async Task<int> CalculateValueAsync(CancellationToken cancellationToken)
{
  using CancellationTokenSource combinedCts = CancellationTokenSource
      .CreateLinkedTokenSource(cancellationToken, _disposeCts.Token);
  await Task.Delay(TimeSpan.FromSeconds(2), combinedCts.Token);
  return 13;
}
```

当调用`Dispose`时，正在进行的操作将被取消：

```cs
async Task UseMyClassAsync()
{
  Task<int> task;
  using (var resource = new MyClass())
  {
    task = resource.CalculateValueAsync(default);
  }

  // Throws OperationCanceledException.
  var result = await task;
}
```

对于某些类型，将`Dispose`实现为取消请求完全可以（例如，`HttpClient`具有这些语义）。然而，其他类型需要知道所有操作何时完成。对于这些类型，你需要某种形式的异步处理。

异步释放是在 C# 8.0 和 .NET Core 3.0 中引入的一种技术。BCL 引入了一个新的`IAsyncDisposable`接口，它是`IDisposable`的异步等价物。同时，语言还引入了一个`await using`语句，它是`using`的异步等价物。因此，现在希望在释放期间执行异步工作的类型现在具备了这种能力：

```cs
class MyClass : IAsyncDisposable
{
  public async ValueTask DisposeAsync()
  {
    await Task.Delay(TimeSpan.FromSeconds(2));
  }
}
```

`DisposeAsync`的返回类型是`ValueTask`而不是`Task`，但标准的`async`和`await`关键字在处理`ValueTask`时与处理`Task`一样有效。

实现`IAsyncDisposable`的类型通常由`await using`来消耗：

```cs
await using (var myClass = new MyClass())
{
  ...
} // DisposeAsync is invoked (and awaited) here.
```

如果需要避免使用`ConfigureAwait(false)`，是可行的，但有点麻烦，因为你必须在`await using`语句之外声明变量：

```cs
var myClass = new MyClass();
await using (myClass.ConfigureAwait(false))
{
  ...
} // DisposeAsync is invoked (and awaited) here with ConfigureAwait(false).
```

## 讨论

异步释放肯定比将`Dispose`实现为取消请求更容易，只有在确实需要时才应使用更复杂的方法。事实上，大多数情况下你甚至可以不释放任何东西，这显然是最简单的方法，因为你不必做任何事情。

本篇介绍了两种处理释放的模式；如果需要，也可以*同时*使用它们。同时使用可以给你的类型赋予使用`await using`时的干净关闭语义，以及使用`Dispose`时的“取消”语义。总体上我不建议这样做，但这确实是一种选择。

## 另请参阅

菜谱 10.8 介绍了链接的取消令牌。

菜谱 11.1 介绍了异步接口。

菜谱 2.10 讨论了实现返回`ValueTask`的方法。

菜谱 2.7 介绍了如何使用`ConfigureAwait(false)`来避免上下文。
