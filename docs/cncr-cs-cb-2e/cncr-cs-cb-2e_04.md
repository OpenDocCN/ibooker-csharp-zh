# 第四章：并行基础

本章涵盖了并行编程的模式。并行编程用于拆分 CPU 绑定的工作并将其分配给多个线程。这些并行处理的示例仅考虑 CPU 绑定的工作。如果你有自然异步操作（如 I/O 绑定的工作），希望并行执行，请参阅第二章，特别是食谱 2.4。

本章涵盖的并行处理抽象是任务并行库（TPL）的一部分。TPL 是内置于 .NET 框架中的。

# 4.1 并行处理数据

## 问题

你有一组数据，并且需要对数据的每个元素执行相同的操作。这个操作是 CPU 绑定的，可能需要一些时间。

## 解决方案

`Parallel` 类型包含一个专门为此问题设计的 `ForEach` 方法。以下示例接受一组矩阵并对它们进行旋转：

```cs
void RotateMatrices(IEnumerable<Matrix> matrices, float degrees)
{
  Parallel.ForEach(matrices, matrix => matrix.Rotate(degrees));
}
```

有些情况下，你可能希望尽早停止循环，例如遇到无效值时。以下示例反转每个矩阵，但如果遇到无效矩阵，它将中止循环：

```cs
void InvertMatrices(IEnumerable<Matrix> matrices)
{
  Parallel.ForEach(matrices, (matrix, state) =>
  {
    if (!matrix.IsInvertible)
      state.Stop();
    else
      matrix.Invert();
  });
}
```

此代码使用 `ParallelLoopState.Stop` 来停止循环，防止进一步调用循环体。请注意，这是一个并行循环，因此可能已经在运行其他循环体调用，包括当前项之后的项目。在这个代码示例中，如果第三个矩阵不可逆，循环将被停止，不会处理新的矩阵，但其他矩阵（如第四和第五个）可能已经在处理中。

更常见的情况是希望能够取消并行循环。这与停止循环不同；停止循环是从循环内部*停止*，而取消是从循环外部*取消*。举例来说，取消按钮可以取消 `CancellationTokenSource`，从而取消并行循环，如下代码示例：

```cs
void RotateMatrices(IEnumerable<Matrix> matrices, float degrees,
    CancellationToken token)
{
  Parallel.ForEach(matrices,
      new ParallelOptions { CancellationToken = token },
      matrix => matrix.Rotate(degrees));
}
```

需要注意的一点是，每个并行任务可能在不同的线程上运行，因此任何共享状态必须受到保护。以下示例反转每个矩阵并计算无法反转的矩阵的数量：

```cs
// Note: this is not the most efficient implementation.
// This is just an example of using a lock to protect shared state.
int InvertMatrices(IEnumerable<Matrix> matrices)
{
  object mutex = new object();
  int nonInvertibleCount = 0;
  Parallel.ForEach(matrices, matrix =>
  {
    if (matrix.IsInvertible)
    {
      matrix.Invert();
    }
    else
    {
      lock (mutex)
      {
        ++nonInvertibleCount;
      }
    }
  });
  return nonInvertibleCount;
}
```

## 讨论

`Parallel.ForEach` 方法允许对值序列进行并行处理。类似的解决方案是并行 LINQ（PLINQ），它提供了与 LINQ 类似的语法和大部分相同的功能。`Parallel` 和 PLINQ 之间的一个区别是，PLINQ 假定可以使用计算机上的所有核心，而 `Parallel` 将动态响应 CPU 条件的变化。

`Parallel.ForEach` 是一个并行的 `foreach` 循环。如果需要执行并行的 `for` 循环，`Parallel` 类还支持 `Parallel.For` 方法。如果你有多个数据数组都使用相同的索引，`Parallel.For` 尤其有用。

## 参见

食谱 4.2 包括并行聚合一系列值，包括求和和平均值。

食谱 4.5 介绍了 PLINQ 的基础知识。

第十章涵盖了取消。

# 4.2 并行聚合

## 问题

在并行操作结束时，您需要对结果进行聚合。聚合的示例包括对值求和或找到它们的平均值。

## 解决方案

`Parallel`类通过*本地值*的概念支持聚合，这些变量在并行循环内部存在。这意味着循环体可以直接访问该值，而无需同步。当循环准备好聚合每个本地结果时，它会使用`localFinally`委托来执行。请注意，`localFinally`委托需要同步访问保存最终结果的变量。以下是并行求和的示例：

```cs
// Note: this is not the most efficient implementation.
// This is just an example of using a lock to protect shared state.
int ParallelSum(IEnumerable<int> values)
{
  object mutex = new object();
  int result = 0;
  Parallel.ForEach(source: values,
      localInit: () => 0,
      body: (item, state, localValue) => localValue + item,
      localFinally: localValue =>
      {
        lock (mutex)
          result += localValue;
      });
  return result;
}
```

并行 LINQ 比`Parallel`类具有更自然的聚合支持：

```cs
int ParallelSum(IEnumerable<int> values)
{
  return values.AsParallel().Sum();
}
```

好吧，这有点取巧，因为 PLINQ 内置支持许多常见操作符（例如`Sum`）。PLINQ 还通过`Aggregate`操作符支持通用聚合：

```cs
int ParallelSum(IEnumerable<int> values)
{
  return values.AsParallel().Aggregate(
      seed: 0,
      func: (sum, item) => sum + item
  );
}
```

## 讨论

如果您已经在使用`Parallel`类，可能希望使用其聚合支持。否则，在大多数场景中，PLINQ 支持更具表现力且代码更短。

## 另请参阅

食谱 4.5 介绍了 PLINQ 的基础知识。

# 4.3 并行调用

## 问题

您有许多方法可以并行调用，这些方法（大多数）彼此独立。

## 解决方案

`Parallel`类包含一个简单的`Invoke`成员，专为这种场景设计。以下示例将数组分成两半，并分别处理每一半：

```cs
void ProcessArray(double[] array)
{
  Parallel.Invoke(
      () => ProcessPartialArray(array, 0, array.Length / 2),
      () => ProcessPartialArray(array, array.Length / 2, array.Length)
  );
}

void ProcessPartialArray(double[] array, int begin, int end)
{
  // CPU-intensive processing...
}
```

如果要调用的次数直到运行时才知道，则还可以将委托数组传递给`Parallel.Invoke`方法：

```cs
void DoAction20Times(Action action)
{
  Action[] actions = Enumerable.Repeat(action, 20).ToArray();
  Parallel.Invoke(actions);
}
```

`Parallel.Invoke`支持与`Parallel`类的其他成员一样的取消：

```cs
void DoAction20Times(Action action, CancellationToken token)
{
  Action[] actions = Enumerable.Repeat(action, 20).ToArray();
  Parallel.Invoke(new ParallelOptions { CancellationToken = token }, actions);
}
```

## 讨论

`Parallel.Invoke`对于简单的并行调用是一个很好的解决方案。请注意，如果要为每个输入数据项调用一个动作（请改用`Parallel.ForEach`），或者如果每个动作产生一些输出（请改用 Parallel LINQ），那么它将不是一个完美的选择。

## 另请参阅

食谱 4.1 介绍了`Parallel.ForEach`，它为每个数据项调用一个动作。

食谱 4.5 涵盖了并行 LINQ。

# 4.4 动态并行性

## 问题

您有一个更复杂的并行情况，其中并行任务的结构和数量取决于仅在运行时可知的信息。

## 解决方案

任务并行库（TPL）以`Task`类型为中心。`Parallel`类和并行 LINQ 只是强大的`Task`的便利包装。当您需要动态并行性时，直接使用`Task`类型是最简单的。

下面是一个示例，其中需要对二叉树的每个节点进行一些昂贵的处理。直到运行时才知道树的结构，因此这是动态并行性的一个良好场景。`Traverse` 方法处理当前节点，然后创建两个子任务，分别处理节点下面的两个分支（对于本示例，假设必须先处理父节点再处理子节点）。`ProcessTree` 方法通过创建顶级父任务并等待其完成来启动处理：

```cs
void Traverse(Node current)
{
  DoExpensiveActionOnNode(current);
  if (current.Left != null)
  {
    Task.Factory.StartNew(
        () => Traverse(current.Left),
        CancellationToken.None,
        TaskCreationOptions.AttachedToParent,
        TaskScheduler.Default);
  }
  if (current.Right != null)
  {
    Task.Factory.StartNew(
        () => Traverse(current.Right),
        CancellationToken.None,
        TaskCreationOptions.AttachedToParent,
        TaskScheduler.Default);
  }
}

void ProcessTree(Node root)
{
  Task task = Task.Factory.StartNew(
      () => Traverse(root),
      CancellationToken.None,
      TaskCreationOptions.None,
      TaskScheduler.Default);
  task.Wait();
}
```

`AttachedToParent` 标志确保每个分支的任务与其父节点的任务链接在一起。这创建了任务实例之间与树节点中父/子关系相对应的父/子关系。父任务执行其委托，然后等待其子任务完成。子任务中的异常随后从子任务传播到其父任务。因此，`ProcessTree` 可以通过对树根上的单个 `Task` 调用 `Wait` 来等待整个树的任务。

如果没有父/子关系的情况，可以通过使用任务*继续*将任何任务安排在另一个任务之后执行。继续是一个单独的任务，在原始任务完成时执行：

```cs
Task task = Task.Factory.StartNew(
    () => Thread.Sleep(TimeSpan.FromSeconds(2)),
    CancellationToken.None,
    TaskCreationOptions.None,
    TaskScheduler.Default);
Task continuation = task.ContinueWith(
    t => Trace.WriteLine("Task is done"),
    CancellationToken.None,
    TaskContinuationOptions.None,
    TaskScheduler.Default);
// The "t" argument to the continuation is the same as "task".
```

## 讨论

`CancellationToken.None` 和 `TaskScheduler.Default` 在上面的代码示例中被使用。取消令牌在 Recipe 10.2 中有详细介绍，任务调度器则在 Recipe 13.3 中有介绍。明确指定 `StartNew` 和 `ContinueWith` 使用的 `TaskScheduler` 是个不错的主意。

这种父子任务的安排在动态并行性中很常见，尽管不是必需的。同样可以将每个新任务存储在线程安全的集合中，然后使用 `Task.WaitAll` 等待它们全部完成。

###### 警告

使用 `Task` 进行并行处理与使用 `Task` 进行异步处理完全不同。

`Task` 类型在并发编程中有两个用途：它可以是并行任务或异步任务。并行任务可能使用阻塞成员，例如 `Task.Wait`、`Task.Result`、`Task.WaitAll` 和 `Task.WaitAny`。并行任务通常也使用 `AttachedToParent` 在任务之间创建父/子关系。应使用 `Task.Run` 或 `Task.Factory.StartNew` 创建并行任务。

相反地，异步任务应避免使用阻塞成员，而应偏向于使用 `await`、`Task.WhenAll` 和 `Task.WhenAny`。异步任务不应使用 `AttachedToParent`，但它们可以通过等待另一个任务来形成一种隐式的父/子关系。

## 参见

Recipe 4.3 描述了如何在并行工作开始时并行调用一系列方法。

# 4.5 并行 LINQ

## 问题

您需要对数据序列进行并行处理，以生成另一个数据序列或该数据的摘要。

## 解决方案

大多数开发人员都熟悉 LINQ，您可以使用它来对序列进行拉取式计算。并行 LINQ（PLINQ）通过并行处理扩展了这种 LINQ 支持。

PLINQ 在流式场景中表现良好，当您有一系列输入并产生一系列输出时。以下是一个简单的例子，仅将序列中的每个元素乘以二（实际场景比简单的乘法更加 CPU 密集）：

```cs
IEnumerable<int> MultiplyBy2(IEnumerable<int> values)
{
  return values.AsParallel().Select(value => value * 2);
}
```

示例可以以任何顺序生成其输出；这是并行 LINQ 的默认行为。您还可以指定要保留的顺序。下面的例子仍然是并行处理的，但保留了原始顺序：

```cs
IEnumerable<int> MultiplyBy2(IEnumerable<int> values)
{
  return values.AsParallel().AsOrdered().Select(value => value * 2);
}
```

并行 LINQ 的另一个自然用途是并行聚合或汇总数据。以下代码执行了并行求和操作：

```cs
int ParallelSum(IEnumerable<int> values)
{
  return values.AsParallel().Sum();
}
```

## 讨论

`Parallel` 类在许多场景下表现良好，但在聚合或将一个序列转换为另一个序列时，PLINQ 代码更简单。请记住，与 PLINQ 相比，`Parallel` 类对系统上的其他进程更加友好；尤其是在服务器机器上进行并行处理时，这是一个考虑因素。

PLINQ 提供了许多运算符的并行版本，包括过滤器（`Where`）、投影（`Select`）以及各种聚合，如 `Sum`、`Average` 和更通用的 `Aggregate`。总体而言，您可以使用普通 LINQ 可以做的任何事情，也可以使用 PLINQ 并行处理。如果您有现有的 LINQ 代码，可以受益于并行运行，那么 PLINQ 是一个很好的选择。

## 参见

配方 4.1 讲述了如何使用 `Parallel` 类来对序列中的每个元素执行代码。

配方 10.5 讲述了如何取消 PLINQ 查询。
