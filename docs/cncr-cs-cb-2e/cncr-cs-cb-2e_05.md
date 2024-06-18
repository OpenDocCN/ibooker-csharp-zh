# 第五章：数据流基础

TPL Dataflow 是一个强大的库，它允许你创建一个网格或管道，然后（异步地）通过它发送数据。Dataflow 是一种非常声明式的编程风格：通常情况下，你先完全定义网格，然后开始处理数据。网格最终成为一个结构，通过它你的数据流动。这需要你用一种不同的方式来思考你的应用，但一旦你跨越了这一步，数据流就变成了许多场景的自然选择。

每个网格由相互链接的各种块组成。单个块很简单，负责数据处理的一个步骤。当块完成对其数据的处理时，它会将结果传递给任何链接的块。

要使用 TPL Dataflow，在你的应用程序中安装 NuGet 包 [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df)。

# 5.1 链接块

## 问题

你需要将数据流块链接到彼此以创建一个网格。

## 解决方案

TPL Dataflow 库提供的块仅定义了最基本的成员。许多有用的 TPL Dataflow 方法实际上是扩展方法。`LinkTo` 扩展方法提供了一种将数据流块连接在一起的简单方法：

```cs
var multiplyBlock = new TransformBlock<int, int>(item => item * 2);
var subtractBlock = new TransformBlock<int, int>(item => item - 2);

// After linking, values that exit multiplyBlock will enter subtractBlock.
multiplyBlock.LinkTo(subtractBlock);
```

默认情况下，链接的数据流块仅传播数据；它们不传播完成状态（或错误）。如果你的数据流是线性的（像一个管道），那么你可能希望传播完成状态。要传播完成状态（和错误），你可以在链接上设置 `PropagateCompletion` 选项：

```cs
var multiplyBlock = new TransformBlock<int, int>(item => item * 2);
var subtractBlock = new TransformBlock<int, int>(item => item - 2);

var options = new DataflowLinkOptions { PropagateCompletion = true };
multiplyBlock.LinkTo(subtractBlock, options);

...

// The first block's completion is automatically propagated to the second block.
multiplyBlock.Complete();
await subtractBlock.Completion;
```

## 讨论

一旦链接，数据将自动从源块流向目标块。`PropagateCompletion` 选项可以在传递数据的同时传递完成状态；然而，在管道的每一步中，故障块会将其异常传播给下一个块，并封装在 `AggregateException` 中。因此，如果你有一个传播完成的长管道，原始错误可能会嵌套在多个 `AggregateException` 实例中。`AggregateException` 有几个成员，例如 `Flatten`，可以帮助处理这种情况中的错误。

可以用多种方式链接数据流块；你的网格可以有分支和汇合甚至循环。然而，对于大多数场景，简单的线性管道就足够了。我们主要处理管道（并简要涵盖分支）；更高级的场景超出了本书的范围。

`DataflowLinkOptions`类型为您提供了几种不同的选项，您可以在链接上设置这些选项（例如在此解决方案中使用的`PropagateCompletion`选项），并且`LinkTo`重载还可以接受一个谓词，您可以使用它来过滤通过链接的数据。如果数据未通过过滤器，则不会被丢弃。通过过滤器的数据通过该链接传输；未通过过滤器的数据尝试通过备用链接传输，并且如果没有其他链接可供其使用，则留在块中。如果数据项在块中被卡住，那么该块将不会生成任何其他数据项；直到移除该数据项为止，整个块都将处于停滞状态。

## 另请参阅

菜谱 5.2 涵盖了沿着链路传播错误的方法。

菜谱 5.3 介绍了如何移除块之间的链接。

菜谱 8.8 介绍了如何将数据流块链接到 System.Reactive 的可观测流。

# 5.2 传播错误

## 问题

您需要一种方法来响应数据流网格中可能发生的错误。

## 解决方案

如果传递给数据流块的委托引发异常，那么该块将进入故障状态。当块处于故障状态时，它将丢弃所有数据（并停止接受新数据）。下面的代码中的块永远不会生成任何输出数据；第一个值引发异常，第二个值则被丢弃：

```cs
var block = new TransformBlock<int, int>(item =>
{
  if (item == 1)
    throw new InvalidOperationException("Blech.");
  return item * 2;
});
block.Post(1);
block.Post(2);
```

要捕获数据流块的异常，您应该`await`其`Completion`属性。`Completion`属性返回一个`Task`，当块完成时完成，如果块故障，则`Completion`任务也将故障：

```cs
try
{
  var block = new TransformBlock<int, int>(item =>
  {
    if (item == 1)
      throw new InvalidOperationException("Blech.");
    return item * 2;
  });
  block.Post(1);
  await block.Completion;
}
catch (InvalidOperationException)
{
  // The exception is caught here.
}
```

当使用`PropagateCompletion`链接选项传播完成时，错误也会被传播。但是，异常会被包装在`AggregateException`中传递到下一个块。以下示例从管道末端捕获异常，因此如果从先前的块传播异常，则会捕获`AggregateException`：

```cs
try
{
  var multiplyBlock = new TransformBlock<int, int>(item =>
  {
    if (item == 1)
      throw new InvalidOperationException("Blech.");
    return item * 2;
  });
  var subtractBlock = new TransformBlock<int, int>(item => item - 2);
  multiplyBlock.LinkTo(subtractBlock,
      new DataflowLinkOptions { PropagateCompletion = true });
  multiplyBlock.Post(1);
  await subtractBlock.Completion;
}
catch (AggregateException)
{
  // The exception is caught here.
}
```

每个块都将传入的错误包装在`AggregateException`中，即使传入的错误已经是`AggregateException`。如果在管道的早期发生错误并在几个链接下行之前被观察到，则原始错误将被包装在多层`AggregateException`中。`AggregateException.Flatten`方法简化了这种情况下的错误处理。

## 讨论

在构建网格（或管道）时，请考虑如何处理错误。在较简单的情况下，最好只传播错误并在最后一次捕获它们。在更复杂的网格中，您可能需要在数据流完成时观察每个块。

或者，如果你希望你的块在面对异常时仍然可用，你可以选择将异常视为另一种数据，让它们与正确处理的数据一起流过网格。使用这种模式，你可以保持数据流网格的操作性，因为块本身不会故障，并且会继续处理下一个数据项。参见 配方 14.6 了解更多详情。

## 参见

配方 5.1 讲述了如何建立块之间的链接。

配方 5.3 讲述了如何断开块之间的链接。

配方 14.6 讲述了如何在数据流网格中同时传递异常和数据。

# 5.3 取消链接块

## 问题

在处理过程中，您需要动态更改数据流的结构。这是一个几乎不常见的高级场景。

## 解决方案

你可以随时链接或取消链接数据流块；数据可以自由地通过网格流动，随时链接或取消链接都是安全的。链接和取消链接都是完全线程安全的。

当您创建数据流块链接时，请保留 `LinkTo` 方法返回的 `IDisposable`，并在希望取消链接块时将其处理掉：

```cs
var multiplyBlock = new TransformBlock<int, int>(item => item * 2);
var subtractBlock = new TransformBlock<int, int>(item => item - 2);

IDisposable link = multiplyBlock.LinkTo(subtractBlock);
multiplyBlock.Post(1);
multiplyBlock.Post(2);

// Unlink the blocks.
// The data posted above may or may not have already gone through the link.
// In real-world code, consider a using block rather than calling Dispose.
link.Dispose();
```

## 讨论

除非你能保证链接是空闲的，否则在取消链接时可能会出现竞态条件。然而，通常这些竞态条件并不是一个问题；数据要么在断开链接之前流过链接，要么根本不会。没有竞态条件会导致数据的重复或丢失。

取消链接是一个高级场景，但在少数情况下非常有用。举例来说，没有办法改变链接的过滤器。要改变现有链接的过滤器，你需要取消旧的链接并创建一个新的链接，并可以选择设置 `DataflowLinkOptions.Append` 为 false。另一个例子是，在战略性的点上取消链接可以用来暂停数据流网格。

## 参见

配方 5.1 讲述了如何建立块之间的链接。

# 5.4 限流块

## 问题

在您的数据流网格中存在分叉场景，并希望数据以负载平衡的方式流动。

## 解决方案

默认情况下，当块产生输出数据时，它会检查所有链接（按创建顺序），并尝试逐个将数据流过每个链接。此外，每个块默认会维护一个输入缓冲区，在准备好处理数据之前可以接受任意数量的数据。

在分支场景中，这会造成问题，其中一个源块链接到两个目标块：然后第二个块就会饿死。当源块生成数据时，它会尝试将数据流向每个链接。第一个目标块将始终接受数据并缓冲它，因此源块永远不会尝试将数据流向第二个目标块。可以通过使用`BoundedCapacity`块选项节流目标块来解决此问题。默认情况下，`BoundedCapacity`设置为`DataflowBlockOptions.Unbounded`，这会导致第一个目标块即使没有准备好处理数据也缓冲*所有*数据。

`BoundedCapacity`可以设置为大于零的任意值（或者当然是`DataflowBlockOptions.Unbounded`）。只要目标块能够跟得上来自源块的数据，简单的值为 1 就足够了：

```cs
var sourceBlock = new BufferBlock<int>();
var options = new DataflowBlockOptions { BoundedCapacity = 1 };
var targetBlockA = new BufferBlock<int>(options);
var targetBlockB = new BufferBlock<int>(options);

sourceBlock.LinkTo(targetBlockA);
sourceBlock.LinkTo(targetBlockB);
```

## 讨论

节流在分支场景中进行负载平衡非常有用，但可以在任何需要节流行为的地方使用。例如，如果您正在从 I/O 操作中填充数据流网格的数据，可以在网格中的块上应用`BoundedCapacity`。这样，直到网格准备好处理数据之前，您都不会读取太多 I/O 数据，并且在能够处理它之前，您的网格不会最终缓冲所有输入数据。

## 另请参阅

Recipe 5.1 介绍如何将块链接在一起。

# 5.5 使用数据流块进行并行处理

## 问题

您希望在数据流网格中进行一些并行处理。

## 解决方案

默认情况下，每个数据流块彼此独立。当您将两个块链接在一起时，它们将独立处理。因此，每个数据流网格内建有一些自然的并行性。

如果需要进一步的操作，例如，如果有一个执行大量 CPU 计算的特定块，则可以通过设置`MaxDegreeOfParallelism`选项，使该块并行处理其输入数据。默认情况下，此选项设置为 1，因此每个数据流块一次只处理一个数据片段。

`BoundedCapacity`可以设置为`DataflowBlockOptions.Unbounded`或大于零的任何值。以下示例允许任意数量的任务同时乘以数据：

```cs
var multiplyBlock = new TransformBlock<int, int>(
    item => item * 2,
    new ExecutionDataflowBlockOptions
    {
      MaxDegreeOfParallelism = DataflowBlockOptions.Unbounded
    });
var subtractBlock = new TransformBlock<int, int>(item => item - 2);
multiplyBlock.LinkTo(subtractBlock);
```

## 讨论

`MaxDegreeOfParallelism`选项使得块内的并行处理变得容易。不那么容易的是确定哪些块需要它。一种技术是在调试器中暂停数据流执行，在那里您可以看到排队的数据项数量（即尚未由块处理的数据项）。意外数量的数据项可能表明某些重组或并行化将有所帮助。

如果数据流块进行异步处理，`MaxDegreeOfParallelism`也适用。在这种情况下，`MaxDegreeOfParallelism`选项指定并发级别——一定数量的*槽*。每个数据项在开始处理时占据一个槽，并且只有在异步处理完全完成时才释放该槽。

## 参见

食谱 5.1 涵盖了将块链接在一起。

# 5.6 创建自定义块

## 问题

您有要放置到自定义数据流块中的可重用逻辑。这样做可以创建包含复杂逻辑的较大块。

## 解决方案

您可以使用`Encapsulate`方法剪切具有单个输入和输出块的任何数据流网格的部分。`Encapsulate`将从两个端点创建一个单一的块。在这些端点之间传播数据和*完成*是您的责任。以下代码创建了一个自定义数据流块，将数据和完成状态传播到两个块之间：

```cs
IPropagatorBlock<int, int> CreateMyCustomBlock()
{
  var multiplyBlock = new TransformBlock<int, int>(item => item * 2);
  var addBlock = new TransformBlock<int, int>(item => item + 2);
  var divideBlock = new TransformBlock<int, int>(item => item / 2);

  var flowCompletion = new DataflowLinkOptions { PropagateCompletion = true };
  multiplyBlock.LinkTo(addBlock, flowCompletion);
  addBlock.LinkTo(divideBlock, flowCompletion);

  return DataflowBlock.Encapsulate(multiplyBlock, divideBlock);
}
```

## 讨论

当您将网格封装为自定义块时，请考虑要向用户公开的选项类型。考虑每个块选项应该（或不应该）传递给内部网格；在许多情况下，某些块选项不适用或没有意义。因此，常见的做法是自定义块定义自己的自定义选项，而不是接受`DataflowBlockOptions`参数。

`DataflowBlock.Encapsulate`仅封装具有一个输入块和一个输出块的网格。如果您具有具有多个输入和/或输出的可重用网格，则应将其封装在自定义对象中，并将输入和输出作为类型为`ITargetBlock<T>`（用于输入）和`IReceivableSourceBlock<T>`（用于输出）的属性公开。

这些示例都使用`Encapsulate`创建自定义块。也可以自己实现数据流接口，但这要困难得多。Microsoft 有[一篇论文](http://bit.ly/tpl-dataflow)描述了创建自定义数据流块的高级技术。

## 参见

食谱 5.1 涵盖了将块链接在一起。

食谱 5.2 涵盖了沿着块链接传播错误。
