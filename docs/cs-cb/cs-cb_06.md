# 第六章：异步编程

过去，大多数人编写的代码都是同步的。诸如并发性、线程池和并行编程等问题曾是专业专家的领域，有时甚至这些专家也会出错。历史上的互联网论坛、UseNet 甚至书籍上都充满了警告，不要尝试多线程，除非你知道你在做什么，并且确实有强烈的需求。然而，现在情况已经改变。

2010 年，微软推出了任务并行库（TPL），大大简化了编写多线程代码。这与多线程/多核 CPU 架构的普及同时出现。TPL 的一个基本组件是`Task`类，它表示承诺在单独的线程上执行某些工作，并返回结果。有趣的是，PLINQ 也在同一时间框架内推出，它在第 4.10 节中有详细介绍。TPL 仍然是开发人员工具箱中处理内部 CPU 密集型多线程的重要组成部分。

基于 TPL 的`Task`概念，微软在 C# 4 中通过专门的语言语法引入了异步。虽然自 C# 1 以来我们已经有了异步编程，通过委托，但它更复杂且效率更低。在 C# 5 中，通过引入`async/await`关键字，异步编程变得简化，使得代码及其执行顺序非常类似于同步代码。除了简化之外，C#异步的一个主要用例是跨进程通信，与 TPL 擅长处理的内部 CPU 密集型工作形成对比。当进行跨进程操作时，考虑访问文件系统、进行数据库查询或调用 REST API。在幕后，异步管理这些操作的线程，使其不会阻塞，并提高应用程序的性能和可伸缩性。使用异步，我们可以以简单的方式推理逻辑，同时享受异步操作的好处和复杂性。

自其推出以来，微软不断通过语言特性和.NET Framework 库改进异步。本章介绍了这些新功能，如异步`Main`方法、新的`ValueTask`类型、异步迭代器和异步处理。还有一些异步的原始能力值得特别关注，例如编写安全的异步库、管理并发的异步任务、取消和进度报告。

本章的主题是结账，客户在购物车中有产品，他们已经开始结账流程，代码需要处理每个结账请求。我们将从控制台应用程序中正确使用异步开始。

# 6.1 创建异步控制台应用程序

## 问题

在控制台应用程序中需要使用一个库，但它只有异步 API。

## 解决方案

该课程有异步方法：

```cs
public class CheckoutService
{
    public async Task<string> StartAsync()
    {
        await ValidateAddressAsync();
        await ValidateCreditAsync();
        await GetShoppingCartAsync();
        await FinalizeCheckoutAsync();

        return "Checkout Complete";
    }

    async Task ValidateAddressAsync()
    {
        // perform address validation
    }

    async Task ValidateCreditAsync()
    {
        // ensure credit is good
    }

    async Task GetShoppingCartAsync()
    {
        // get contents of shopping cart
    }

    async Task FinalizeCheckoutAsync()
    {
        // complete checkout transaction
    }
}
```

下面是编写异步控制台应用程序的旧方法：

```cs
class Program
{
    static void Main(string[] args)
    {
        var checkoutSvc = new CheckoutService();
        string result = string.Empty;

        Task<string> startedTask = checkoutSvc.StartAsync();
        startedTask.Wait();
        result = startedTask.Result;

        Console.WriteLine($"Result: {result}");
    }
}
```

下面是编写异步控制台应用程序的推荐方法：

```cs
static async Task Main()
{
    var checkoutSvc = new CheckoutService();

    string result = await checkoutSvc.StartAsync();

    Console.WriteLine($"Result: {result}");
}
```

## 讨论

初次引入时，异步几乎随处可见并且立即有用。然而，在某些边缘情况下，比如 `Main` 方法以及 `catch` 和 `finally` 块，不能使用异步。幸运的是，Microsoft 在 C# 7.1 中修复了这个问题，并在 .NET Framework 的其他部分增加了更多支持，例如 ASP.NET MVC 中的异步 `ActionResult`。Recipe 6.3 展示了异步迭代器如何解决另一个异步问题。

本节描述的一个显著的异步增强是异步 `Main`。问题在于，就像解决方案中的 `CheckoutService` 类一样，许多 .NET Framework 类型和第三方库都是为异步编写的。然而，没有异步 `Main`，开发人员必须编写问题代码。为了演示问题，解决方案包括 `Main` 方法的两个版本：旧的同步方式和新的异步方法。

在旧的同步技术中，开发人员被迫使用 `Wait()` 和 `Result`，这是典型的异步反模式，因为会导致线程阻塞、潜在的线程死锁和竞争条件。Recipe 6.4 解释了一种情况，如果编写这样的代码可能会导致死锁（以及如何避免）。这些都是异步方法返回的 `Task` 类型的成员。不幸的是，在第一次异步迭代中，如果想要编写命令行实用程序、文本应用程序或演示应用程序，这是唯一的选择。

解决方案中的第二个 `Main` 显示了新的语法，包括 `async` 修饰符和 `Task` 返回类型。我们只需 `await` 调用 `checkoutSvc.StartAsync()`，代码就可以正常工作。

###### 提示

如你所知，`Main` 可以返回 `void` 或 `int`。这个使用 `Task` 的示例是为了 `void` 返回而设计的。你可以将其改为 `Task<int>` 以返回 `int`。

本质上，Microsoft 没有推荐一种安全的方式从同步代码调用异步代码。因此，这是一个受欢迎的补充，大大简化了编写调用异步代码的控制台应用程序的过程。还要注意，从 `Main` 到 `CheckoutService.StartAsync` 再到其他 `CheckoutService` 方法的整个调用链都是异步的。理想情况下，整个调用链都应该是异步的，但偶尔你会有一个异步方法只调用同步方法；你可以在 Recipe 6.6 中了解更多相关信息。

## 参见

Recipe 6.3, “创建异步迭代器”

Recipe 6.4, “编写安全的异步库”

Recipe 6.6, “从异步代码调用同步代码”

# 6.2 减少异步返回值的内存分配

## 问题

你希望减少异步代码的内存消耗。

## 解决方案

这是在异步方法中使用 `ValueTask` 而不是 `Task` 的方法：

```cs
public class CheckoutService
{
    public async ValueTask<string> StartAsync()
    {
        await ValidateAddressAsync();
        await ValidateCreditAsync();
        await GetShoppingCartAsync();
        await FinalizeCheckoutAsync();

        return "Checkout Complete";
    }

    async ValueTask ValidateAddressAsync()
    {
        // perform address validation
    }

    async ValueTask ValidateCreditAsync()
    {
        // ensure credit is good
    }

    async ValueTask GetShoppingCartAsync()
    {
        // get contents of shopping cart
    }

    async ValueTask FinalizeCheckoutAsync()
    {
        // complete checkout transaction
    }
}
```

这是消耗该类的应用程序：

```cs
class Program
{
    static async Task Main()
    {
        var checkoutSvc = new CheckoutService();

        string result = await checkoutSvc.StartAsync();

        Console.WriteLine($"Result: {result}");
    }
}
```

## 讨论

自从异步开始以来，我们一直通过 `Task` 或 `Task<T>` 返回类型。这种方式一直有效，并且将来也会继续适用于任何异步代码。然而，随着时间的推移，人们发现了特定情况，这些情况打开了关于 `Task` 是引用类型以及运行时如何缓存 `Tasks` 的新性能机会。

`Task` 类根据定义是一个引用类型。这意味着每次异步方法返回 `Task` 时，运行时都会分配堆内存。正如你所知，值类型在定义它们的地方分配内存，但它们不会引起垃圾收集器的开销。

或许不太明显，`Tasks` 的另一个特性是运行时会对其进行缓存。而不是使用 `await` 方法，可以直接引用从 `async` 方法返回的 `Task`。有了这个 `Task` 的引用，你可以对多个任务进行并发调用。你也可以多次调用该任务。这里的重点是运行时已经缓存了任务，导致了更多的内存使用。

如前所述，在普通编码中，`Task` 的使用是正常的，也许你并不在意。然而，考虑到高性能场景，会分配大量的 `Task` 对象，你可能有兴趣找到提升性能和可扩展性的方法。该解决方案模拟了一个可能会关注这个问题的概念。想象一下，一个企业每天需要处理大量的购物车结账操作。在这种情况下，消除任何对象分配、垃圾收集和内存压力可能都是有益的。

为了解决这些问题，Microsoft 添加了对 `ValueTask`（以及 `ValueTask<T>`）作为异步返回类型的支持。顾名思义，`ValueTask` 是一个值类型。因为它是一个值类型，在这种情况下，它只会在栈上分配内存。根据值类型的定义，它并不会有独立的堆分配或者为了这个值而进行的垃圾收集。

此外，运行时不会缓存 `ValueType`，这导致了更少的内存分配和缓存管理。这在高性能/可扩展性场景中非常有效。解决方案中的 `CheckoutService` 展示了如何使用 `ValueTask`：只需在 `Task` 的位置使用它。这里的假设是代码总是会 `await` 方法，而不会尝试重用 `ValueTask`。在解决方案中，确实是这样。

###### 注意

如果你正在为其他开发人员编写可重用的库，请考虑 `ValueTask` 是否合适。通过使用 `ValueTask`，你消除了消费代码执行并发任务操作或其他高级场景的能力，这种情况下 `Task` 提供了最大的灵活性。

就像大多数事物一样，存在权衡。运行时`Task`缓存对于`ValueTask`不再适用的所有场景都不再是选项。使用`ValueTask`时，不能将操作组合或在第一次之后重用`ValueTask`。菜谱 6.7 和 6.8 展示了`ValueTask`无法使用的几种情况。

总结一下，当性能和可扩展性成为问题时，请使用`ValueTask`，在其他时间则可以自由使用`Task`。

## 参见

菜谱 6.7，“等待并行任务完成”

菜谱 6.8，“处理并行任务完成时的情况”

# 6.3 创建异步迭代器

## 问题

你正在处理异步代码，而经典的同步迭代器将不起作用。

## 解决方案

这是结帐过程的数据：

```cs
public class CheckoutRequest
{
    public Guid ShoppingCartID { get; set; }

    public string Name { get; set; }

    public string Card { get; set; }

    public string Address { get; set; }
}
```

这是每个请求的结帐过程：

```cs
public class CheckoutService
{
    public async ValueTask<string> StartAsync(CheckoutRequest request)
    {
        return
            $"Checkout Complete for Shopping " +
            $"Basket: {request.ShoppingCartID}";
    }
}
```

异步迭代器处理每个请求：

```cs
public class CheckoutStream
{
    public async IAsyncEnumerable<CheckoutRequest> GetRequestsAsync()
    {
        while (true)
        {
            IEnumerable<CheckoutRequest> requests =
                await GetNextBatchAsync();

            foreach (var request in requests)
                yield return request;

            await Task.Delay(1000);
        }
    }

    async Task<IEnumerable<CheckoutRequest>> GetNextBatchAsync()
    {
        return new List<CheckoutRequest>
        {
            new CheckoutRequest
            {
                ShoppingCartID = Guid.NewGuid(),
                Address = "123 4th St",
                Card = "1234 5678 9012 3456",
                Name = "First Card Name"
            },
            new CheckoutRequest
            {
                ShoppingCartID = Guid.NewGuid(),
                Address = "789 1st Ave",
                Card = "2345 6789 0123 4567",
                Name = "Second Card Name"
            },
            new CheckoutRequest
            {
                ShoppingCartID = Guid.NewGuid(),
                Address = "123 4th St",
                Card = "1234 5678 9012 3456",
                Name = "First Card Name"
            },
        };
    }
}
```

最后，应用程序消耗迭代器来处理每个请求：

```cs
static async Task Main()
{
    var checkoutSvc = new CheckoutService();
    var checkoutStrm = new CheckoutStream();

    await foreach (var request in checkoutStrm.GetRequestsAsync())
    {
        string result = await checkoutSvc.StartAsync(request);

        Console.WriteLine($"Result: {result}");
    }
}
```

## 讨论

虽然迭代器对于像`List<T>`这样的.NET Framework 集合或者你自己编写的自定义集合至关重要，但它们也可以是有用的抽象，隐藏复杂的数据获取逻辑。该解决方案演示了一个相关的场景，迭代器可能会有用的地方——处理一个`CheckoutRequests`流，就像它是一个集合一样。

解决方案的一个重要方面是，将太多的`Check​ou⁠t​Request`实例保存在内存中是不切实际的。如果一个系统不断接收订单，它需要进行扩展。在这个解决方案中，我们设想了一个轮询实现，持续获取下一批`CheckoutRequests`。这样可以减少内存压力，并且迭代器提供了一个抽象，隐藏了程序接收订单的复杂细节。

在异步的早期阶段，执行像这样的任务会更加复杂，因为轮询是异步的，会发出一个进程外的请求。显然可以找到一个让这种情况同步发生的库，但这忽略了异步的好处。该解决方案通过`IAsyncEnumerable`提供了一个新的异步流接口来解决这个问题。

`CheckoutStream`类有一个名为`GetRequestsAsync`的迭代器，返回`IAsyncEnumerable<CheckoutRequest>`。这相当于同步迭代器的异步版本`IEnumerable<T>`。虽然在此演示中，`while`循环永远继续下去，你需要手动停止应用程序，菜谱 6.9 展示了如何优雅地取消该过程。此迭代器获取一个新的`CheckoutRequests`批次，生成批次中的每个项目，并在获取下一批之前休眠一秒钟。为了演示目的，休眠使用了`Task.Delay`。

###### 注意

`yield` 关键字是语法糖，用于将类型成员转换为迭代器。包括 `IAsync​Enumer⁠able​<T>` 在内的 `IEnumerable<T>` 类型具有 `MoveNext` 和 `Current` 成员，其中 `MoveNext` 将 `Current` 加载为它读取的下一个值。在幕后，当 C# 编译器看到一个迭代器时，它会生成一个新的类，其中包含 `MoveNext` 和 `Current` 成员。当在 `GetRequestsAsync` 中使用 `yield return request` 等方式时，C# 编译器实例化该新类，调用 `MoveNext` 并返回 `Current`。

`GetNextBatchAsync` 方法仅返回 `CheckoutRequests` 列表。但是，请想象这实际上是一个异步调用到一个准备好下一组 `CheckoutRequest` 实例的网络端点、队列或服务总线。食谱 1.9、3.7 和 3.9 展示了在执行此操作时您关心的一些问题。通过将所有这些复杂性移入迭代器，应用代码可以以更简单的方式消耗数据。

`Main` 方法展示了如何消耗异步迭代器。首先要注意的是 `foreach` 循环上的 `async` 修饰符。这是 C# 中异步流的一个新补充。正如您所见，它允许 `foreach` 与 `IAsyncEnumerable<T>` 迭代器一起工作。

## 参见

第 1.9 节，“设计自定义异常”

第 3.7 节，“重新抛出异常”

第 3.9 节，“构建弹性网络连接”

第 6.9 节，“取消异步操作”

# 6.4 编写安全的异步库

## 问题

您的异步代码与 UI 线程引起死锁。

## 解决方案

此类将代码从 UI 线程上移：

```cs
public class CheckoutService
{
    public async Task<string> StartAsync()
    {
        await ValidateAddressAsync().ConfigureAwait(false);
        await ValidateCreditAsync().ConfigureAwait(false);
        await GetShoppingCartAsync().ConfigureAwait(false);
        await FinalizeCheckoutAsync().ConfigureAwait(false);

        return "Checkout Complete";
    }

    async Task ValidateAddressAsync()
    {
        // perform address validation
    }

    async Task ValidateCreditAsync()
    {
        // ensure credit is good
    }

    async Task GetShoppingCartAsync()
    {
        // get contents of shopping cart
    }

    async Task FinalizeCheckoutAsync()
    {
        // complete checkout transaction
    }
}
```

下面是调用它的程序：

```cs
static async Task Main()
{
    var checkoutSvc = new CheckoutService();

    string result = await checkoutSvc.StartAsync();

    Console.WriteLine($"Result: {result}");
}
```

## 讨论

UI 技术（如 Windows Forms、Windows Presentation Foundation（WPF）和 WinUI）运行在单线程上 —— UI 线程上。这简化了开发人员在处理 UI 代码时需要做的工作。但是，如果您使用异步或编写多线程逻辑，事情很容易出错。特别是，如果另一个线程尝试对 UI 进行任何操作或在 UI 线程的相同逻辑中运行，您就有可能发生竞态条件和死锁。要了解问题有多糟糕，请考虑您的应用程序通常在开发、QA 和生产环境中运行良好。然后，毫无征兆地，UI 卡住了，客户开始抱怨，而您无法重现问题。

###### 注意

在某些情况下，根据您使用的 UI 和 .NET 版本，当从非 UI 线程访问 UI 时可能会出现以下异常：

```cs
System.InvalidOperationException:
     'The calling thread cannot access this object
     because a different thread owns it.'
```

这很好，因为至少您知道存在问题。

Recipe 6.1 解释了如何通过调用`Wait`或在`Task`上分配`Result`可能会导致死锁。问题出在`Wait`和`Result`会阻塞 UI 线程，等待响应。异步调用的代码执行完毕并返回，试图在同一个线程上运行。然而，正如刚才提到的，UI 线程被阻塞，导致死锁。

解决方案在`CheckoutService.StartAsync`方法中修复了这个问题。请注意它如何调用`ConfigureAwait(false)`——这段代码与 Recipe 6.1 中的解决方案唯一的区别就是这样做可以将执行从调用线程（UI 线程）迁移到新线程上。现在，当线程从异步调用返回时，它不会导致死锁。

###### 注意

当等待一个`Task`时，默认条件是`ConfigureAwait(true)`。只有在实践工程之外的高级场景中才需要更改此默认设置。如果在代码中看到它，可能需要质疑为什么会有这种需求。

这里需要强调的一个重点是，问题陈述明确提到*libraries*。在编写库时，希望代码能够独立于调用方运行。因此，库代码必须独立于调用方，并且不知道调用方是谁。这就是 Recipe 1.5 中所述的例子，分离关注点非常重要。如果库代码不涉及 UI 操作（它绝对不应该涉及），就能避免线程问题，如竞态条件和死锁。

如果一个`await`使用了`ConfigureAwait(false)`，那么该方法中的所有`await`也应该使用。原因是有些方法执行速度非常快，会同步执行，并且`ConfigureAwait(false)`不会调度线程。如果另一个`await`异步运行但没有使用`ConfigureAwait(false)`，你会遇到与未调用`ConfigureAwait(false)`时相同的线程问题。

###### 警告

Visual Studio 分析器在所有缺少`ConfigureAwait(false)`的非 UI 代码上设置警告。可能会觉得增加这些配置很麻烦，但你仍然应该这样做。即使你认为方法的第一个`await`保证会异步执行，逻辑在维护过程中可能会发生变化，而你可能会无意中引发线程问题。最安全的方法是保持分析器启用并遵循推荐。

`ConfigureAwait(false)`的另一个好处是它稍微提高了效率。默认的`ConfigureAwait(true)`会为设置回调而产生开销，该回调会将已完成的线程调度到 UI 线程上。而`ConfigureAwait(false)`则避免了这种情况。

关于在库代码中使用 `ConfigureAwait(false)` 适当性的问题，有时您不希望使用它。特别是在事件处理程序中，特别是在 UI 代码中，您不希望调用 `ConfigureAwait(false)`。想一想事件处理程序及其功能。它是响应某些用户操作（如按钮点击）而调用的，它设置状态、更新等待指示器、禁用用户不应与之交互的控件，发起调用，然后重置 UI。所有这些工作都在 UI 线程上进行，正如它应该的那样。在这种情况下，您不希望使用 `ConfigureAwait(false)` 将其调度到 UI 线程之外，因为这将导致多线程 UI 问题。

虽然库代码不应该知道 UI 的存在，但有时代码应该传达进度或状态。与其直接访问 UI 代码，还有另一种通信状态的方式，如下一节讨论的内容。

## 另请参阅

食谱 1.5，“设计应用程序层”

食谱 6.5，“异步更新进度”

# 6.5 异步更新进度

## 问题

您需要在不阻塞 UI 线程的情况下显示来自异步任务的状态。

## 解决方案

此类包含进度状态信息：

```cs
public class CheckoutRequestProgress
{
    public int Total { get; set; }

    public string Message { get; set; }
}
```

此方法报告进度：

```cs
public async IAsyncEnumerable<CheckoutRequest>
    GetRequestsAsync(IProgress<CheckoutRequestProgress> progress)
{
    int total = 0;

    while (true)
    {
        List<CheckoutRequest> requests =
            await GetNextBatchAsync().ConfigureAwait(false);

        total += requests.Count;

        foreach (var request in requests)
            yield return request;

        progress.Report(
            new CheckoutRequestProgress
            {
                Total = total,
                Message = "New Batch of Checkout Requests"
            });

        await Task.Delay(1000).ConfigureAwait(false);
    }
}
```

下面是初始化和消耗进度更新的程序：

```cs
static async Task Main()
{
    var checkoutSvc = new CheckoutService();
    var checkoutStrm = new CheckoutStream();

    IProgress<CheckoutRequestProgress> progress =
        new Progress<CheckoutRequestProgress>(p =>
        {
            Console.WriteLine(
                $"\n" +
                $"Total: {p.Total}, " +
                $"{p.Message}" +
                $"\n");
        });

    await foreach (var request in
        checkoutStrm.GetRequestsAsync(progress))
    {
        string result = await checkoutSvc.StartAsync(request);

        Console.WriteLine($"Result: {result}");
    }
}
```

## 讨论

如 食谱 6.4 所述，库代码永远不应直接更新 UI。如果正确编写，它将在单独的线程上运行，并且对其调用者毫不知情。尽管如此，业务层或库代码可能希望通知调用者进度或状态。解决方案展示了一个场景，其中迭代器使用 `CheckoutRequestProgress` 类定义的进度信息更新 UI。基本上，库代码定义了它提供的进度信息类型，调用代码与之配合使用。在这种情况下，它是处理的订单总数和某些表示状态的消息。

`GetRequestAsync` 方法接受一个名为 `progress` 的 `IProgress<CheckoutRequestProgress>` 参数。`IProgress<T>` 是 .NET Framework 的一部分，`Progress<T>` 类实现了 `IProgress<T>` 接口。使用 `progress` 实例，`GetRequestsAsync` 调用 `Report` 方法，并传递一个已填充属性的 `CheckoutRequestProgress` 实例。这将进度发送到 UI 中的处理程序。

`Main` 方法通过实例化 `Progress<CheckoutRequest​Pro⁠gress>` 并将其分配给 `progress`，一个 `IProgress<CheckoutRequestProgress>`，设置了报告。`Progress<T>` 构造函数接受一个 `Action` 委托，并且 `Main` 分配了一个写入控制台进度的 lambda。每当 `GetRequestsAsync` 调用 `Report` 方法时，此 lambda 就会执行。回到起点，`Main` 将 `progress` 作为参数传递给 `Get​Re⁠questsAsync` 调用，以便它可以引用同一个对象进行报告。

你可能已经注意到`GetRequestAsync`在异步运行，并且`GetNextBatchAsync`和`Task.Delay`上的`await`也调用了`ConfigureAwait(false)`。如果该代码在除了 UI 线程之外的其他线程上运行，死锁的可能性是多少？没有，因为`Progress<T>`将回调调度到 UI 线程，所以代码可以安全地与 UI 进行交互。请记住，库代码`GetRequest​Async`对于`Process<T>`构造函数的`Action`参数的 lambda 参数没有任何了解。这意味着 lambda 可以安全地访问任何需要显示进度的 UI 代码。

## 参见

第 6.4 节，“编写安全的异步库”

# 6.6 调用同步代码从异步代码

## 问题

您的异步方法中唯一的代码是同步的，并且您希望以异步方式`await`它。

## 解决方案

这节课展示了如何从同步逻辑中返回异步结果：

```cs
public class CheckoutService
{
    public async Task<string> StartAsync()
    {
        await ValidateAddressAsync().ConfigureAwait(false);
        await ValidateCreditAsync().ConfigureAwait(false);
        await GetShoppingCartAsync().ConfigureAwait(false);
        await FinalizeCheckoutAsync().ConfigureAwait(false);

        return "Checkout Complete";
    }

    async Task<bool> ValidateAddressAsync()
    {
        bool result = true;
        return await Task.FromResult(result);
    }

    async Task<bool> ValidateCreditAsync()
    {
        bool result = true;
        return await Task.FromResult(result);
    }

    async Task<bool> GetShoppingCartAsync()
    {
        bool result = true;
        return await Task.FromResult(result);
    }

    async Task FinalizeCheckoutAsync()
    {
        await Task.CompletedTask;
    }
}
```

这是运行应用程序的代码：

```cs
static async Task Main()
{
    var checkoutSvc = new CheckoutService();

    string result = await checkoutSvc.StartAsync();

    Console.WriteLine($"Result: {result}");
}
```

## 讨论

为了简化，本章的前几节从异步代码中调用同步代码。你可能已经注意到，Visual Studio（与其他 IDE 相同）在异步方法没有任何`await`时会显示绿色波浪线。您还会收到以下警告：

```cs
CS1998: This async method lacks 'await' operators
and will run synchronously.
Consider using the 'await' operator to await non-blocking API calls,
or 'await Task.Run(...)' to do CPU-bound work on a background thread.
```

编译器发出这个警告是件好事，因为这可能是个错误。有可能你忘记了在异步方法调用上添加`await`修饰符。在这种情况下，程序不会在等待的方法处停止执行。异步方法和调用它的代码都会运行。如果没有等待的异步方法可能不会在程序退出时完成。

另一个问题是，如果未等待的异步方法抛出异常，那么它将无法捕获，因为调用代码继续运行。使用`async void`方法时也会遇到类似的问题，因为你无法`await`它们，也没有办法捕获异常。

###### 警告

本章中的几处地方描述了与异步代码相关的编译器警告。在许多情况下，这些警告代表错误条件。我经常遇到应用程序有着无法管理的警告墙。好像开发人员以某种方式不认为警告是个问题或者没有在意。了解忘记`await`异步方法或未添加`ConfigureAwait(false)`的严重后果，正如第 6.4 节所述，可能会促使您优先清理和维护警告。

有时异步方法内部的代码确实是同步的。它原本可能是异步的，但在维护时被修改了，或者你必须实现一个接口。在这种情况下，你有几种方法。一种方法是在调用链中删除`async/await`关键字，直到达到需要异步的更高级别方法。如果有多个调用者等待该方法，或者它是多个应用程序的公共接口的一部分，你可能不想立即进行重构。另一种方法，在解决方案中演示的是`await Task.FromResult<T>`。

你可以在`CheckoutService`中的`StartAsync`方法中看到它是如何工作的，其中每个方法返回等待`Task.FromResult<T>`的结果。`Task.FromResult<T>`方法是泛型的，因此可以用于任何类型。

当方法需要返回一个值时，`await Task.FromResult<T>`是有效的。然而，`FinalizeTaskAsync`方法只返回`Task`。请注意，该方法只是简单地等待`Task.CompletedTask`。

你可能会认为这是多余的工作，只是为了消除一个警告。虽然这是事实，但考虑一下其好处。你确实消除了警告，并且享受到了保持警告墙修剪带来的生产力提升。更重要的是，代码明确表达了其意图，进行维护的开发人员清楚地看到没有由于缺少`await`而导致的错误——代码是正确的。

## 参见

Recipe 6.4, “编写安全的异步库”

# 6.7 等待并行任务完成

## 问题

你有多个任务，同时运行，并且需要等待它们全部完成后才能继续。

## 解决方案

这段代码运行并行任务：

```cs
public class CheckoutService
{
    class WhenAllResult
    {
        public bool IsValidAddress { get; set; }
        public bool IsValidCredit { get; set; }
        public bool HasShoppingCart { get; set; }
    }

    public async Task<string> StartAsync()
    {
        var checkoutTasks =
            new List<Task<(string, bool)>>
            {
                ValidateAddressAsync(),
                ValidateCreditAsync(),
                GetShoppingCartAsync()
            };

        Task<(string method, bool result)[]> allTasks =
            Task.WhenAll(checkoutTasks);

        if (allTasks.IsCompletedSuccessfully)
        {
            WhenAllResult whenAllResult = GetResultsAsync(allTasks);

            await FinalizeCheckoutAsync(whenAllResult);

            return "Checkout Complete";
        }
        else
        {
            throw allTasks.Exception;
        }
    }

    WhenAllResult GetResultsAsync(
        Task<(string method, bool result)[]> allTasks)
    {
        var whenAllResult = new WhenAllResult();

        foreach (var (method, result) in allTasks.Result)
            switch (method)
            {
                case nameof(ValidateAddressAsync):
                    whenAllResult.IsValidAddress = result;
                    break;
                case nameof(ValidateCreditAsync):
                    whenAllResult.IsValidCredit = result;
                    break;
                case nameof(GetShoppingCartAsync):
                    whenAllResult.HasShoppingCart = result;
                    break;
            }

        return whenAllResult;
    }

    async Task<(string, bool)> ValidateAddressAsync()
    {
        //throw new ArgumentException("Testing!");

        return await Task.FromResult(
            (nameof(ValidateAddressAsync), true));
    }

    async Task<(string, bool)> ValidateCreditAsync()
    {
        return await Task.FromResult(
            (nameof(ValidateCreditAsync), true));
    }

    async Task<(string, bool)> GetShoppingCartAsync()
    {
        return await Task.FromResult(
            (nameof(GetShoppingCartAsync), true));
    }

    async Task<bool> FinalizeCheckoutAsync(WhenAllResult result)
    {
        Console.WriteLine(
            $"{nameof(WhenAllResult.IsValidAddress)}: " +
            $"{result.IsValidAddress}");
        Console.WriteLine(
            $"{nameof(WhenAllResult.IsValidCredit)}: " +
            $"{result.IsValidCredit}");
        Console.WriteLine(
            $"{nameof(WhenAllResult.HasShoppingCart)}: " +
            $"{result.HasShoppingCart}");

        bool success = true;
        return await Task.FromResult(success);
    }
}
```

这里是请求和处理并行任务结果的应用程序：

```cs
static async Task Main()
{
    try
    {
        var checkoutSvc = new CheckoutService();

        string result = await checkoutSvc.StartAsync();

        Console.WriteLine($"Result: {result}");
    }
    catch (AggregateException aEx)
    {
        foreach (var ex in aEx.InnerExceptions)
            Console.WriteLine($"Unable to complete: {ex}");
    }
}
```

## 讨论

在执行如购物车结账等操作时，你不希望用户等待应用程序太长时间才返回。依次运行太多操作可能会增加等待时间。改善用户体验的一种方法是并发地运行独立操作。

在解决方案中，`CheckoutService`有四个不同的异步服务。在这里我们假设其中三个操作，`ValidateAddressAsync`、`ValidateCreditAsync`和`GetShoppingCartAsync`，彼此之间没有任何依赖关系。这使它们成为同时运行的良好候选者。

`StartAsync`方法通过创建一个`List<Task>`来实现这一点。如果你回忆起来，等待一个方法实际上就是在返回的`Task`上进行`await`。没有`await`，每个方法都会返回一个`Task`，但其逻辑直到等待该任务后才会运行。

`Task`类有一个`WhenAll`方法，其目的是并发地运行由`checkoutTasks`参数指定的所有任务。`WhenAll`会等待所有`Tasks`完成后才返回。

等待具有返回类型的单个方法，从角度来看很简单，因为你可以将返回值分配给单个变量。然而，在并行运行任务时，你需要关联响应，因为`WhenAll`同时返回所有任务。假设哪些任务发生在集合的哪个位置可能会出现错误，并且在维护时可能会很繁琐。代码需要知道哪个响应对应哪个`Task`。

解决方案通过元组实现，其中`string`是方法的名称，`bool`是响应。该元组及其内容的选择是为了这个演示而特定的，你可以根据你的应用程序适当调整任务类型。这让我们知道哪个任务对应哪个结果。`GetResultsAsync`方法通过迭代任务数组，并根据每个响应的方法参数构建`WhenAllResult`来实现这一点。

注意，`ValidateAddressAsync`的第一行是一个被注释的语句，抛出了一个`ArgumentException`。取消注释并重新运行应用程序会在调用`WhenAll`时导致异常。`Main`方法通过对`AggregateException`的`catch`处理该异常。由于所有任务都在并行运行，其中一个或多个任务可能会抛出异常。`AggregateException`收集这些异常。通常情况下，你可以在`InnerException`属性中查找异常详细信息。然而，`AggregateException`还有另一个属性，即`InnerExceptions`。它们之间的区别在于`AggregateException`的属性是复数形式，这是有意为之的。为了正确调试，你可以在`InnerExceptions`属性中找到所有异常。

## 参见

Recipe 6.8, “处理并行任务的完成情况”

# 6.8 处理并行任务的完成情况

## 问题

你以为调用`Task.WhenAny`会高效利用资源来处理任务完成时的结果，但实际上成本和性能都很糟糕。

## 解决方案

这是一个顺序实现，用于调用多个任务：

```cs
public async Task<string> StartBigONAsync()
{
    (_, bool addressResult) = await ValidateAddressAsync();
    (_, bool creditResult) = await ValidateCreditAsync();
    (_, bool cartResult) = await GetShoppingCartAsync();

    await FinalizeCheckoutAsync(
        new AllTasksResult
        {
            IsValidAddress = addressResult,
            IsValidCredit = creditResult,
            HasShoppingCart = cartResult
        });

    return "Checkout Complete";
}
```

这是一个并行实现，用于调用多个任务：

```cs
public async Task<string> StartBigO1Async()
{
    var checkoutTasks =
        new List<Task<(string, bool)>>
        {
            ValidateAddressAsync(),
            ValidateCreditAsync(),
            GetShoppingCartAsync()
        };

    Task<(string method, bool result)[]> allTasks =
        Task.WhenAll(checkoutTasks);

    if (allTasks.IsCompletedSuccessfully)
    {
        AllTasksResult allResult = GetResults(allTasks);

        await FinalizeCheckoutAsync(allResult);

        return "Checkout Complete";
    }
    else
    {
        throw allTasks.Exception;
    }
}
```

下一个实现将任务并行处理，但在每个任务返回时处理它们：

```cs
public async Task<string> StartBigONSquaredAsync()
{
    var checkoutTasks =
        new List<Task<(string, bool)>>
        {
            ValidateAddressAsync(),
            ValidateCreditAsync(),
            GetShoppingCartAsync()
        };

    var allResult = new AllTasksResult();

    while (checkoutTasks.Any())
    {
        Task<(string, bool)> task = await Task.WhenAny(checkoutTasks);
        checkoutTasks.Remove(task);

        GetResult(task, allResult);
    }

    await FinalizeCheckoutAsync(allResult);

    return "Checkout Complete";
}
```

该方法展示了如何获取首个完成的任务：

```cs
async Task<(string method, bool result)> ValidateCreditAsync()
{
    var checkoutTasks =
        new List<Task<(string, bool)>>
        {
            CheckInternalCreditAsync(),
            CheckAgency1CreditAsync(),
            CheckAgency2CreditAsync()
        };

    Task<(string, bool)> task = await Task.WhenAny(checkoutTasks);

    (_, bool result) = task.Result;

    return await Task.FromResult(
        (nameof(ValidateCreditAsync), result));
}
```

`Main`方法提供了选择从哪个方法开始：

```cs
static async Task Main()
{
    try
    {
        var checkoutSvc = new CheckoutService();

        string result = await checkoutSvc.StartBigO1Async();
        //string result = await checkoutSvc.StartBigONAsync();
        //string result = await checkoutSvc.StartBigONSquaredAsync();

        Console.WriteLine($"Result: {result}");
    }
    catch (AggregateException aEx)
    {
        foreach (var ex in aEx.InnerExceptions)
            Console.WriteLine($"Unable to complete: {ex}");
    }
}
```

## 讨论

本节问题探讨了`Task.WhenAny`的作用。如果你尝试使用`Task.WhenAny`来处理任务返回时的处理，可能会感到意外，因为它的工作方式不符合你的预期。

在大部分情况下，这个解决方案的概念和组织方式与 Recipe 6.7 类似——不同之处在于，这个解决方案展示了运行任务的不同方法，并解释了你需要知道的以做出适当的设计决策。

`StartBigONAsync` 方法的运行方式类似本章前面顺序运行的部分。其性能为 O(N)，因为它依次处理 N 个任务。

Recipe 6.7 展示了如何在任务之间不互相依赖时加快程序执行。它使用了 `Task.WhenAll`，在 `StartBigO1Async` 中展示。性能提升来自其近似 O(1) 的性能——不需要执行 N 次操作，只执行 1 次。更准确地说，这是 O(2)，因为 `FinalizeCheckout​A⁠sync` 在其他三个任务完成后运行。

除了 `Task.WhenAll`，你还可以使用 `Task.WhenAny`。可能自然而然地认为 `Task.WhenAny` 是并行运行多个任务并能在其他任务运行时处理每个任务的好方法。然而，`Task.WhenAny` 并不像你想象的那样工作。看看 `StartBigONSquaredAsync` 并按照以下逻辑操作：

1.  `while` 循环在 `checkoutTasks` 仍有内容时迭代。

1.  `Task.WhenAny` 启动所有任务并行运行。

1.  最快的任务返回。

1.  自从该任务返回后，从 `checkoutTasks` 中移除它，以免再次运行。

1.  收集该任务的结果。

1.  再次在剩余任务上执行循环，或在 `checkoutTasks` 为空时停止。

在该算法中的第一个令人惊讶的心理障碍是错误地认为后续循环操作相同的任务，每个任务完成后返回。实际情况是每个后续循环都会启动一组全新的任务。这就是异步工作的方式——你可以多次 `await` 一个任务，但每个 `await` 都会启动一个新的任务。这意味着代码在每次循环时都会连续启动剩余任务的新实例。这种循环模式与 `Task.WhenAny`，不像 `Task.WhenAll` 那样，不会产生你可能预期的 O(1) 性能，而是 O(N²)。尽管此解决方案只有三个任务，但想象一下任务列表增长时性能会如何逐渐下降。

###### 注意

本章讨论了使用大 O 表示法的性能问题。特别是在查看 O(N²) 的算法时，存在一种过多操作会破坏性能的阈值。Recipe 3.10 展示了如何测量应用程序的性能，并根据你的性能要求找到该阈值。

另外，考虑在一段时间内对检查操作次数进行乘法运算，这些任务的数量会增加。你的应用性能不仅会变差，还可能通过过多的网络流量和端点服务器处理来减慢服务器。这不仅可能影响你自己的系统，还可能影响同时运行的其他系统。此外，考虑到网络请求可能是云服务在消耗计划上的情况，这可能会非常昂贵。在这种特定的用例中，除非使用较少的任务并且影响最小，否则可能被视为反模式。

###### 警告

在互联网上，您会找到解释`Task.WhenAny` 作为一种在并行运行任务并在完成时处理每个任务的技术的文章。虽然这对于一些任务可能有效，但本节讲述了在该用例中使用`Task.WhenAny` 的危险性。

话虽如此，`Task.WhenAny` 在某些情况下非常有效——第一个任务获胜。在解决方案中，`ValidateCreditAsync` 方法展示了这种策略。场景是您有多个源来判断客户是否有良好的信用，而且来自任何一个源的响应都是可靠的。每个服务具有不同的性能特征，您只关心返回最快的那个。您可以丢弃其余的响应。这保持了 O(1)的性能。

`ValidateCreditAsync` 包含要运行的任务列表。`Task.WhenAny` 并行运行这些任务，并返回第一个完成的任务。代码处理该任务并返回。

本解决方案的副作用是除了返回的第一个任务之外，其他任务仍在继续运行。但是，您无法访问它们，因为只返回一个任务。对于这种情况，您并不关心这些任务，但应停止它们以避免使用比必要更多的资源。您可以在下一节关于取消任务的部分中了解如何做到这一点。

## 参见

3.10 章节，“性能测量”的配方

6.7 章节，“等待并行任务完成”的配方

6.9 章节，“取消异步操作”

# 6.9 取消异步操作

## 问题

您正在进行异步过程，并需要停止它。

## 解决方案

该类演示了取消任务的多种方法：

```cs
public class CheckoutStream
{
    CancellationToken cancelToken;

    public CheckoutStream(CancellationToken cancelToken)
    {
        this.cancelToken = cancelToken;
    }

    public async IAsyncEnumerable<CheckoutRequest> GetRequestsAsync(
        IProgress<CheckoutRequestProgress> progress)
    {
        int total = 0;

        while (true)
        {
            var requests = new List<CheckoutRequest>();

            try
            {
                requests = await GetNextBatchAsync();
            }
            catch (OperationCanceledException)
            {
                break;
            }

            total += requests.Count;

            foreach (var request in requests)
            {
                if (cancelToken.IsCancellationRequested)
                    break;

                yield return request;
            }

            progress.Report(
                new CheckoutRequestProgress
                {
                    Total = total,
                    Message = "New Batch of Checkout Requests"
                });

            if (cancelToken.IsCancellationRequested)
                break;

            await Task.Delay(1000);
        }

        if (cancelToken.IsCancellationRequested)
            progress.Report(
                new CheckoutRequestProgress
                {
                    Total = total,
                    Message = "Process Cancelled!"
                });
    }

    async Task<List<CheckoutRequest>> GetNextBatchAsync()
    {
        if (cancelToken.IsCancellationRequested)
            throw new OperationCanceledException();

        var requests = new List<CheckoutRequest>
        {
            new CheckoutRequest
            {
                ShoppingCartID = Guid.NewGuid(),
                Address = "123 4th St",
                Card = "1234 5678 9012 3456",
                Name = "First Card Name"
            },
            new CheckoutRequest
            {
                ShoppingCartID = Guid.NewGuid(),
                Address = "789 1st Ave",
                Card = "2345 6789 0123 4567",
                Name = "Second Card Name"
            },
            new CheckoutRequest
            {
                ShoppingCartID = Guid.NewGuid(),
                Address = "123 4th St",
                Card = "1234 5678 9012 3456",
                Name = "First Card Name"
            },
        };

        return await Task.FromResult(requests);
    }
}
```

这是初始化取消并展示如何取消的应用程序：

```cs
static async Task Main()
{
    var cancelSource = new CancellationTokenSource();
    var checkoutStrm = new CheckoutStream(cancelSource.Token);
    var checkoutSvc = new CheckoutService();

    IProgress<CheckoutRequestProgress> progress =
        new Progress<CheckoutRequestProgress>(p =>
        {
            Console.WriteLine(
                $"\n" +
                $"Total: {p.Total}, " +
                $"{p.Message}" +
                $"\n");
        });

    int count = 1;

    await foreach (var request in
        checkoutStrm.GetRequestsAsync(progress))
    {
        string result = await checkoutSvc.StartAsync(request);

        Console.WriteLine($"Result: {result}");

        if (count++ >= 10)
            break;

        if (count >= 5)
            cancelSource.Cancel();
    }
}
```

## 讨论

6.3 章节具有一个永不结束的`while` 循环异步迭代器。这在演示中起作用，但实际应用程序通常需要一种停止长时间运行过程的方式。想象一下弹出具有正在进行的过程状态并提供取消按钮的对话框，允许您停止操作。任务取消自 TPL 引入以来一直存在，并且在取消异步操作中也是至关重要的。

在解决方案中，`Main` 方法展示了如何初始化取消。`Cancel​la⁠tionTokenSource`，`cancelSource`，提供了取消令牌和取消控制。请看`CheckoutStream` 构造函数的参数是通过`cancelSource` 的`Token` 属性设置的`Cancellation​To⁠ken`。

因为 `cancelSource` 可以管理其范围内所有代码的取消，所以可以将 `CancellationToken` 作为参数传递给任何具有 `CancellationToken` 参数的构造函数或方法，允许您从单个位置 `cancelSource` 取消任何操作。解决方案没有按钮，并在处理了 10 个 `CheckoutRequests` 后取消。您可以看到 `count` 变量在每个循环中增加，检查请求的数量，并在 `10` 之后中断循环。由于对 `count >= 5` 的检查，程序永远不会达到 `10`，因此调用 `cancelSource.Cancel()`。

对 `cancelSource.Cancel` 的调用发送了应取消处理的消息，但您仍然需要编写能够识别取消需求的代码。尽早取消是正确的，而 `GetRequestsAsync` 在多个检查中使用了 `cancelToken.IsCancellationRequested`。当在传递 `CancelToken` 的 `CancellationTokenSource` 实例上调用 `Cancel` 时，`IsCancellationRequested` 属性为 `true`。

在循环内部，`IsCancellationRequested` 中断。在循环外部，`IsCancellationRequested` 发送一个 `IProgress<T>` 状态消息，以通知调用者操作已经正确取消。

`GetNextBatchAsync` 方法展示了另一种处理取消的方式，即抛出 `OperationCancelledException`。如果你回想一下，方法抛出异常是因为它无法完成设计的操作。在这种情况下，`GetNextBatchAsync` 没有检索记录，因此这可能是一个语义上正确的响应方式。即使这不是您会做出的设计决策，也要考虑到 `GetNextBatchAsync` 可能会 `await` 另一个方法，传递其 `cancelToken`。当取消时，等待的异步方法可能会抛出 `OperationCancelledException`。因此，在处理取消时，可以安全地预期和处理 `OperationCancelledException`。解决方案通过将对 `GetNextBatchAsync` 的调用包装在 `try/catch` 中来执行此操作，从而中断循环，并让现有代码向调用者报告取消状态。

在取消操作时，您可能还需要清理资源。下一节 食谱 6.10 将讨论如何做到这一点。

## 参见

食谱 6.3，“创建异步迭代器”

食谱 6.10，“处理异步资源释放”

# 6.10 处理异步资源释放

## 问题

您有一个必须释放资源的异步处理过程。

## 解决方案

此类展示了如何正确实现异步释放模式：

```cs
public class CheckoutStream : IAsyncDisposable, IDisposable
{
    CancellationTokenSource cancelSource = new CancellationTokenSource();
    CancellationToken cancelToken;
    ILogger log = new ConsoleLogger();

    FileStream asyncDisposeObj = new FileStream(
        "MyFile.txt", FileMode.OpenOrCreate, FileAccess.Write);
    HttpClient syncDisposeObj = new HttpClient();

    public CheckoutStream()
    {
        this.cancelToken = cancelSource.Token;
    }

    public async IAsyncEnumerable<CheckoutRequest> GetRequestsAsync(
        IProgress<CheckoutRequestProgress> progress)
    {
        int total = 0;

        while (true)
        {
            var requests = new List<CheckoutRequest>();

            try
            {
                requests = await GetNextBatchAsync();
            }
            catch (OperationCanceledException)
            {
                break;
            }

            total += requests.Count;

            foreach (var request in requests)
            {
                if (cancelToken.IsCancellationRequested)
                    break;

                yield return request;
            }

            progress.Report(
                new CheckoutRequestProgress
                {
                    Total = total,
                    Message = "New Batch of Checkout Requests"
                });

            if (cancelToken.IsCancellationRequested)
                break;

            await Task.Delay(1000);
        }
    }

    async Task<List<CheckoutRequest>> GetNextBatchAsync()
    {
        if (cancelToken.IsCancellationRequested)
            throw new OperationCanceledException();

        var requests = new List<CheckoutRequest>
        {
            new CheckoutRequest
            {
                ShoppingCartID = Guid.NewGuid(),
                Address = "123 4th St",
                Card = "1234 5678 9012 3456",
                Name = "First Card Name"
            },
            new CheckoutRequest
            {
                ShoppingCartID = Guid.NewGuid(),
                Address = "789 1st Ave",
                Card = "2345 6789 0123 4567",
                Name = "Second Card Name"
            },
            new CheckoutRequest
            {
                ShoppingCartID = Guid.NewGuid(),
                Address = "123 4th St",
                Card = "1234 5678 9012 3456",
                Name = "First Card Name"
            },
        };

        return await Task.FromResult(requests);
    }

    public async ValueTask DisposeAsync()
    {
        await DisposeAsyncCore();

        Dispose(disposing: false);
        GC.SuppressFinalize(this);
    }

    public void Dispose()
    {
        Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (disposing)
        {
            syncDisposeObj?.Dispose();
            (asyncDisposeObj as IDisposable)?.Dispose();
        }

        DisposeThisObject();
    }

    protected virtual async ValueTask DisposeAsyncCore()
    {
        if (asyncDisposeObj is not null)
        {
            await asyncDisposeObj.DisposeAsync().ConfigureAwait(false);
        }

        if (syncDisposeObj is IAsyncDisposable disposable)
        {
            await disposable.DisposeAsync().ConfigureAwait(false);
        }
        else
        {
            syncDisposeObj.Dispose();
        }

        DisposeThisObject();

        await log.WriteAsync("\n\nDisposed!");
    }

    void DisposeThisObject()
    {
        cancelSource.Cancel();

        asyncDisposeObj = null;
        syncDisposeObj = null;
    }
}
```

这是演示如何使用异步可释放对象的应用程序：

```cs
static async Task Main()
{
    await using var checkoutStrm = new CheckoutStream();

    var checkoutSvc = new CheckoutService();

    IProgress<CheckoutRequestProgress> progress =
        new Progress<CheckoutRequestProgress>(p =>
        {
            Console.WriteLine(
                $"\n" +
                $"Total: {p.Total}, " +
                $"{p.Message}" +
                $"\n");
        });

    int count = 1;

    await foreach (var request in
        checkoutStrm.GetRequestsAsync(progress))
    {
        string result = await checkoutSvc.StartAsync(request);

        Console.WriteLine($"Result: {result}");

        if (count++ >= 10)
            break;
    }
}
```

## 讨论

Recipe 1.1 描述了释放资源的模式以及它如何解决对象生命周期结束时释放资源的问题。这对于同步代码很有效，但对于异步代码则不然。本节展示了如何使用异步释放模式释放异步资源。

在解决方案中，`CheckoutStream`有两个字段：一个`FileStream`，`asyncDisposeObj`，和一个`HttpClient`，`syncDisposeObj`。通常，它们应该具有表示它们在应用程序中用途的名称，但在这种情况下，它们的名称表示它们在解决方案中如何帮助遵循复杂的逻辑集。正如它们的名称所示，`asyncDisposeObj`引用必须异步处理的资源；`syncDisposeObj`引用必须同步处理的资源。同时考虑异步和同步处理非常重要，因为这解释了为什么它们的处理过程现在是交织在一起的。

对于异步和同步释放，`CheckoutService`分别实现了`IAsyncDisposable`和`IDisposable`。正如在 Recipe 1.1 中讨论的那样，`IDisposable`指定类必须实现没有参数的`Dispose`，并且我们添加了一个带有`bool`参数的虚拟`Dispose(bool)`以及一个可选的析构函数来实现该模式。解决方案没有实现可选的析构函数。对于`IAsyncDisposable`，`CheckoutService`实现了必需的`DisposeAsync`方法和一个虚拟的`DisposeAsyncCore`方法，两者都没有参数。

异步和同步两条处理路径都可能运行，因此它们都必须准备好释放资源。在同步路径上，`Dispose(bool)`不仅调用`syncDisposeObj`的`Dispose`，还尝试调用`asyncDisposeObj`的`Dispose`。请注意，`Dispose(bool)`还调用`DisposeThisObject`，其中包含异步路径需要调用的相同代码，从而减少了重复。

虽然`Dispose`和`DisposeAsync`是接口成员，但`Dispose(bool)`和`DisposeAsyncCore`是约定俗成的。还要注意它们都是`virtual`的。这是模式的一部分，派生类可以通过重写这些方法并调用它们，通过`base.Dispose(bool)`和`base.DisposeAsyncCore`来确保释放整个继承层次结构中的资源。

`Dispose`和`DisposeAsync`都调用`Dispose(bool)`，但`DisposeAsync`将`disposing`参数设置为`false`。如果回想一下，`disposing`是`Dispose(bool)`在设置为`true`时释放托管资源的标志。请记住，`Dispose(bool)`是同步路径。相反，`DisposeAsync`调用`DisposeAsyncCore`释放异步资源。

与`Dispose(true)`一样，`DisposeAsyncCore`尝试释放所有托管资源。异步情况很明显。然而，同步对象有几种可能性。如果同步对象当前或将来实现了`IAsyncDisposable`，那么在代码处于异步路径时尝试调用`DisposeAsync`是更好的选择。否则，调用同步路径，使用`Dispose`。

正如前面提到的，`Dispose(bool)`和`DisposeAsyncCore`都调用`DisposeThisObject`。在解决方案场景中，`GetRequestsAsync`迭代器实现了取消操作，如第 6.9 节中所解释的那样。根据情况，可能在处理释放过程中取消是个不错的选择。例如，如果代码需要保存其最新的良好状态或者与网络端点有闭包协议，思考清楚是很重要的，而且释放和异步释放模式能帮助到你。

最后，请注意`Main`方法如何在`CheckoutStream`实例上等待使用语句。这与第 2.2 节中讨论的相同的`using`语句类似，只是现在有一个`await`。这确保代码在`Main`方法结束时调用`DisposeAsync`。

## 参见

第 1.1 节，“管理对象生命周期结束”

第 2.2 节，“简化实例清理”

第 6.9 节，“取消异步操作”
