# 第十三章 调度

当一段代码执行时，它必须在某个线程上运行。*调度器* 是一个决定某段代码在哪里运行的对象。在 .NET 框架中有几种不同的调度器类型，它们稍微有些不同地被并行和数据流代码使用。

我建议在可能的情况下*不*指定调度器；默认情况通常是正确的。例如，在异步代码中，`await` 操作符会自动在相同的上下文中恢复方法，除非你覆盖了这个默认行为，如 Recipe 2.7 中所述。类似地，响应式代码对于引发其事件有合理的默认上下文，你可以通过 `ObserveOn` 覆盖它们，如 Recipe 6.2 中所述。

如果你需要其他代码在特定上下文（例如 UI 线程上下文或 ASP.NET 请求上下文）中执行，则可以使用本章中的调度配方来控制代码的调度。

# 13.1 在线程池中安排工作

## 问题

当你有一段代码，明确希望在线程池线程上执行时。

## 解决方案

绝大多数情况下，你会想使用`Task.Run`，这非常简单。以下代码会阻塞线程池线程 2 秒：

```cs
Task task = Task.Run(() =>
{
  Thread.Sleep(TimeSpan.FromSeconds(2));
});
```

`Task.Run` 也完全理解返回值和异步 lambda。以下代码中 `Task.Run` 返回的任务将在 2 秒后完成，结果为 13：

```cs
Task<int> task = Task.Run(async () =>
{
  await Task.Delay(TimeSpan.FromSeconds(2));
  return 13;
});
```

`Task.Run` 返回一个 `Task`（或 `Task<T>`），可以自然地被异步或响应式代码消耗。

## 讨论

`Task.Run` 在 UI 应用程序中非常理想，当你有耗时的工作需要执行，而不能在 UI 线程上完成时。例如，Recipe 8.4 使用 `Task.Run` 将并行处理推送到线程池线程。然而，在 ASP.NET 上除非你非常确定自己知道在做什么，否则不要在 ASP.NET 上使用 `Task.Run`。在 ASP.NET 上，请求处理代码已经在线程池线程上运行，因此将其推送到*另一个*线程池线程通常是适得其反的。

`Task.Run` 是`BackgroundWorker`、`Delegate.BeginInvoke` 和 `ThreadPool.QueueUserWorkItem` 的有效替代品。这些旧的 API 不应在新代码中使用；使用`Task.Run` 的代码编写起来更容易且随时间维护起来也更简单。此外，`Task.Run` 处理了大多数 `Thread` 的使用案例，因此大多数 `Thread` 的使用也可以替换为 `Task.Run`（只有单线程公寓线程是个例外）。

并行和数据流代码默认在线程池上执行，因此通常不需要在使用`Parallel`、Parallel LINQ 或 TPL Dataflow 库执行的代码中使用`Task.Run`。

如果你正在进行动态并行处理，则应该使用`Task.Factory.StartNew`而不是`Task.Run`。这是必要的，因为`Task.Run`返回的`Task`已经配置为用于异步使用（即，被异步或响应式代码消耗）。它也不支持高级概念，如父/子任务，在动态并行代码中更为常见。

## 参见

Recipe 8.6 讲解了如何使用响应式代码消耗异步代码（例如从`Task.Run`返回的任务）。

Recipe 8.4 讲解了如何通过`Task.Run`异步等待并行代码，这是最简单的方式。

Recipe 4.4 讲解了动态并行处理，这种情况下应使用`Task.Factory.StartNew`而不是`Task.Run`。

# 13.2 使用任务调度器执行代码

## 问题

你有多段代码需要以某种特定的方式执行。例如，你可能需要所有代码在 UI 线程上执行，或者可能只需要同时执行一定数量的代码片段。

本篇介绍了如何定义和构造用于这些代码片段的调度器。如何实际应用该调度器是接下来两篇的主题。

## 解决方案

.NET 中有很多不同的类型可以处理调度；本篇重点介绍`TaskScheduler`，因为它是可移植且相对容易使用的。

最简单的`TaskScheduler`是`TaskScheduler.Default`，它将工作排队到线程池中。你很少会在自己的代码中指定`TaskScheduler.Default`，但要意识到它的重要性，因为它是许多调度场景的默认值。`Task.Run`、并行和数据流代码都使用`TaskScheduler.Default`。

你可以通过使用`TaskScheduler.FromCurrentSynchronizationContext`来捕获特定的*上下文*，然后稍后将工作安排回去：

```cs
TaskScheduler scheduler = TaskScheduler.FromCurrentSynchronizationContext();
```

这段代码创建了一个`TaskScheduler`来捕获当前的`SynchronizationContext`并将代码安排到该上下文中。`SynchronizationContext`是一个表示通用调度上下文的类型。在.NET 框架中有几种不同的上下文；大多数 UI 框架提供了代表 UI 线程的`SynchronizationContext`，而 ASP.NET Core 之前提供了代表 HTTP 请求上下文的`SynchronizationContext`。

`ConcurrentExclusiveSchedulerPair`是.NET 4.5 中引入的另一种强大类型；实际上，它是*两个*相关的调度器。`ConcurrentScheduler`成员是一个允许多个任务同时执行的调度器，只要没有任务在`ExclusiveScheduler`上执行。`ExclusiveScheduler`一次只执行一个任务，并且只有在`ConcurrentScheduler`上没有任务正在执行时才执行：

```cs
var schedulerPair = new ConcurrentExclusiveSchedulerPair();
TaskScheduler concurrent = schedulerPair.ConcurrentScheduler;
TaskScheduler exclusive = schedulerPair.ExclusiveScheduler;
```

`ConcurrentExclusiveSchedulerPair`的一个常见用途是只使用`ExclusiveScheduler`来确保一次只执行一个任务。在`ExclusiveScheduler`上执行的代码将在线程池上运行，但将被限制为仅对使用同一`ExclusiveScheduler`实例的所有其他代码执行独占。

另一个`ConcurrentExclusiveSchedulerPair`的用途是作为一个限流调度器。你可以创建一个`ConcurrentExclusiveSchedulerPair`来限制其并发性。在这种情况下，通常不使用`ExclusiveScheduler`：

```cs
var schedulerPair = new ConcurrentExclusiveSchedulerPair(
    TaskScheduler.Default, maxConcurrencyLevel: 8);
TaskScheduler scheduler = schedulerPair.ConcurrentScheduler;
```

注意，这种限流仅在*执行*时限流代码；这与 Recipe 12.5 中涵盖的逻辑限流有很大不同。特别是，异步代码在等待操作时不被认为正在执行。`ConcurrentScheduler`限制执行代码；其他限流，如`SemaphoreSlim`，在更高级别（即整个`async`方法）限流。

## 讨论

您可能已经注意到，最后一个代码示例将`TaskScheduler.Default`传递给`ConcurrentExclusiveSchedulerPair`的构造函数中。这是因为`ConcurrentExclusiveSchedulerPair`在现有的`TaskScheduler`周围应用其并发/独占逻辑。

本教程介绍了`TaskScheduler.FromCurrentSynchronizationContext`，用于在捕获的上下文中执行代码。也可以直接使用`SynchronizationContext`在该上下文中执行代码；然而，我不建议这种方法。尽可能使用`await`运算符在隐式捕获的上下文中恢复，或者使用`TaskScheduler`包装器。

永远不要使用特定于平台的类型在 UI 线程上执行代码。WPF、Silverlight、iOS 和 Android 都提供`Dispatcher`类型，Universal Windows 使用`CoreDispatcher`，而 Windows Forms 有`ISynchronizeInvoke`接口（即`Control.Invoke`）。不要在新代码中使用这些类型；只需假装它们不存在。`SynchronizationContext`是围绕这些类型的通用抽象。

System.Reactive (Rx)引入了更通用的调度器抽象：`IScheduler`。Rx 调度器能够包装任何其他类型的调度器；`TaskPoolScheduler`将包装任何`TaskFactory`（其中包含`TaskScheduler`）。Rx 团队还定义了一个可以手动控制用于测试的`IScheduler`实现。如果确实需要使用调度器抽象，我建议使用 Rx 中的`IScheduler`；它设计良好，定义明确，易于测试。然而，大多数情况下不需要调度器抽象，早期的库（如任务并行库（TPL）和 TPL Dataflow）仅理解`TaskScheduler`类型。

## 参见

Recipe 13.3 涵盖了将`TaskScheduler`应用于并行代码的方法。

Recipe 13.4 讲述了如何在数据流代码中应用 `TaskScheduler`。

Recipe 12.5 讲述了更高级别的逻辑限流。

Recipe 6.2 讲述了用于事件流的 System.Reactive 调度器。

Recipe 7.6 讲述了 System.Reactive 测试调度器。

# 13.3 调度并行代码

## 问题

你需要控制并行代码中各个代码片段的执行方式。

## 解决方案

一旦创建了适当的 `TaskScheduler` 实例（参见 Recipe 13.2），你可以将其包含在传递给 `Parallel` 方法的选项中。以下代码接受一个矩阵序列的序列。它启动了一些并行循环，并希望限制所有循环的总并行度，而不管每个序列中有多少矩阵：

```cs
void RotateMatrices(IEnumerable<IEnumerable<Matrix>> collections, float degrees)
{
  var schedulerPair = new ConcurrentExclusiveSchedulerPair(
      TaskScheduler.Default, maxConcurrencyLevel: 8);
  TaskScheduler scheduler = schedulerPair.ConcurrentScheduler;
  ParallelOptions options = new ParallelOptions { TaskScheduler = scheduler };
  Parallel.ForEach(collections, options,
      matrices => Parallel.ForEach(matrices, options,
          matrix => matrix.Rotate(degrees)));
}
```

## 讨论

`Parallel.Invoke` 还接受 `ParallelOptions` 的实例，因此你可以像对待 `Parallel.ForEach` 一样向 `Parallel.Invoke` 传递 `TaskScheduler`。如果你正在进行动态并行代码，你可以直接将 `TaskScheduler` 传递给 `TaskFactory.StartNew` 或 `Task.ContinueWith`。

无法将 `TaskScheduler` 传递给并行 LINQ（PLINQ）代码。

## 参见

Recipe 13.2 讲述了常见的任务调度器以及如何在它们之间进行选择。

# 13.4 数据流同步使用调度器

## 问题

你需要控制数据流代码中各个代码片段的执行方式。

## 解决方案

一旦创建了适当的 `TaskScheduler` 实例（参见 Recipe 13.2），你可以将其包含在传递给数据流块的选项中。当从 UI 线程调用时，以下代码创建一个数据流网格，通过线程池将其所有输入值乘以二，然后将结果值附加到列表框的项目上（在 UI 线程上）：

```cs
var options = new ExecutionDataflowBlockOptions
{
  TaskScheduler = TaskScheduler.FromCurrentSynchronizationContext(),
};
var multiplyBlock = new TransformBlock<int, int>(item => item * 2);
var displayBlock = new ActionBlock<int>(
    result => ListBox.Items.Add(result), options);
multiplyBlock.LinkTo(displayBlock);
```

## 讨论

在协调数据流网格不同部分的块操作时，指定 `TaskScheduler` 特别有用。例如，你可以使用 `ConcurrentExclusiveSchedulerPair.ExclusiveScheduler` 确保块 A 和 C 永远不会同时执行代码，同时允许块 B 在任何时候执行。

请注意，通过 `TaskScheduler` 进行同步仅适用于代码*执行*时。例如，如果你有一个运行异步代码并应用独占调度器的动作块，那么在等待时代码不被认为是正在运行。

你可以为任何类型的数据流块指定 `TaskScheduler`。即使一个块可能不执行你的代码（例如 `BufferBlock<T>`），它仍然有一些必要的内部任务需要完成，它将使用提供的 `TaskScheduler` 进行所有内部工作。

## 参见

Recipe 13.2 讲述了常见的任务调度器以及如何在它们之间进行选择。
