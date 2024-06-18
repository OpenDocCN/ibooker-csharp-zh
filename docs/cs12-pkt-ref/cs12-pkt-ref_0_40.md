# 第四十章：异步函数

`await` 和 `async` 关键字支持 *异步编程*，这是一种编程风格，长时间运行的函数在返回给调用者后继续完成大部分或全部工作。这与正常的 *同步* 编程相反，长时间运行的函数会 *阻塞* 调用者直到操作完成。异步编程意味着 *并发*，因为长时间运行的操作在调用者的同时 *并行* 运行。异步函数的实现者通过多线程（用于计算绑定的操作）或通过回调机制（用于 I/O 绑定的操作）启动这种并发。

###### 注意

多线程、并发和异步编程是一个广泛的话题。我们在 *C# 12 in a Nutshell* 中专门为它们分配了两章，并在线上讨论它们，网址是 [*http://albahari.com/threading*](http://albahari.com/threading)。

例如，考虑以下长时间运行且计算绑定的 *同步* 方法：

```cs
int ComplexCalculation()
{
  double x = 2;
  for (int i = 1; i < 100000000; i++)
    x += Math.Sqrt (x) / i;
  return (int)x;
}
```

这个方法在运行时会阻塞调用者几秒钟，然后将计算结果返回给调用者：

```cs
int result = ComplexCalculation(); 
// Sometime later:
Console.WriteLine (result);   // 116
```

CLR 定义了一个称为 `Task<TResult>`（在 `System.Threading.Tasks` 中）的类来封装未来完成的操作概念。通过调用 `Task.Run`，你可以为计算绑定的操作生成一个 `Task<TResult>`，该方法指示 CLR 在一个单独的线程上执行指定的委托，并与调用者并行执行：

```cs
Task<int> ComplexCalculationAsync()
  => Task.Run ( () => ComplexCalculation() );
```

这个方法是 *异步* 的，因为它立即返回给调用者，同时并发执行。然而，我们需要一些机制来允许调用者指定在操作完成并且结果变得可用时应该发生什么。`Task<TResult>` 通过公开 `GetAwaiter` 方法来解决这个问题，该方法允许调用者附加一个 *延续*：

```cs
Task<int> task = ComplexCalculationAsync();
var awaiter = task.GetAwaiter();
awaiter.OnCompleted (() =>        // Continuation
{
  int result = awaiter.GetResult();
  Console.WriteLine (result);       // 116
});
```

这告诉操作：“当你完成时，执行指定的委托。” 我们的延续首先调用 `GetResult`，它返回计算的结果。（或者，如果任务 *失败* —— 抛出异常 —— 调用 `GetResult` 会重新抛出异常。）然后我们的延续通过 `Console.WriteLine` 输出结果。

## `await` 和 `async` 关键字

`await` 关键字简化了连接延续的操作。从一个基本场景开始考虑以下情况：

```cs
var *result* = await *expression*;
*statement(s)*;
```

编译器将这个展开为与以下功能类似的内容：

```cs
var awaiter = *expression*.GetAwaiter();
awaiter.OnCompleted (() => 
{
  var *result* = awaiter.GetResult();
  *statement(s)*;
});
```

###### 注意

编译器还会生成代码来优化操作立即完成（同步完成）的场景。异步操作立即完成的常见原因是它实现了内部缓存机制并且结果已经被缓存。

因此，我们可以像这样调用我们之前定义的 `ComplexCalculationAsync` 方法：

```cs
int result = await ComplexCalculationAsync();
Console.WriteLine (result);
```

要编译，我们需要在包含方法上添加 `async` 修饰符：

```cs
async void Test()
{
  int result = await ComplexCalculationAsync();
  Console.WriteLine (result);
}
```

`async` 修饰符指示编译器将 `await` 视为关键字，而不是标识符，以防在方法内出现歧义（这确保了在 C# 5.0 之前可能使用 `await` 作为标识符的代码仍能正常编译）。`async` 修饰符仅适用于返回 `void` 或（稍后您将看到的）`Task` 或 `Task<TResult>` 的方法（和 lambda 表达式）。

###### 注意

`async` 修饰符与 `unsafe` 修饰符类似，它不会影响方法的签名或公共元数据；它只影响方法**内部**发生的事情。

带有 `async` 修饰符的方法被称为*异步函数*，因为它们本身通常是异步的。要了解原因，请看异步函数如何通过执行过程。

遇到 `await` 表达式时，执行（通常）会返回给调用者——类似于迭代器中的 `yield return`。但在返回之前，运行时会将一个继续操作附加到等待的任务上，确保任务完成时，执行会跳回到方法中，并继续之前的位置。如果任务出现故障，则会重新抛出其异常（通过调用 `GetResult` 方法）；否则，其返回值将分配给 `await` 表达式。

###### 注意

CLR 对任务等待器的 `OnCompleted` 方法的实现默认确保，如果有当前的 *同步上下文*，则会通过它发布继续操作。实际上，这意味着在富客户端 UI 场景（如 WPF、WinUI 和 Windows Forms）中，如果在 UI 线程上进行 `await`，您的代码将继续在同一线程上执行。这简化了线程安全性。

您 `await` 的表达式通常是一个任务；然而，任何具有返回一个 *可等待对象* 的 `GetAwaiter` 方法的对象——实现了 `INotifyCompletion.On​Com⁠pleted` 和具有适当类型的 `GetResult` 方法（以及测试同步完成的 `bool IsCompleted` 属性）——都将满足编译器的要求。

注意我们的 `await` 表达式评估为 `int` 类型；这是因为我们等待的表达式是一个 `Task<int>`（其 `GetAwaiter().GetResult()` 方法返回一个 `int`）。

等待非泛型任务是合法的，并生成一个 `void` 表达式：

```cs
await Task.Delay (5000);
Console.WriteLine ("Five seconds passed!");
```

`Task.Delay` 是一个返回在指定毫秒数后完成的 `Task` 的静态方法。`Task.Delay` 的*同步*等效方法是 `Thread.Sleep`。

`Task` 是 `Task<TResult>` 的非泛型基类，并且在功能上等效于 `Task<TResult>`，除了它没有结果。

## 捕获局部状态

`await` 表达式真正的威力在于它们几乎可以出现在代码的任何地方。具体而言，在异步函数中，`await` 表达式可以出现在除了 `lock` 语句或 `unsafe` 上下文之外的任何表达式的位置。

在下面的示例中，我们在循环中使用了 `await`：

```cs
async void Test()
{
  for (int i = 0; i < 10; i++)
  {
    int result = await ComplexCalculationAsync();
    Console.WriteLine (result);
  }
}
```

第一次执行`ComplexCalculationAsync`时，由于`await`表达式，执行返回给调用者。当方法完成（或故障）时，执行会在离开时保持本地变量和循环计数器的值不变。编译器通过将这样的代码转换为状态机来实现这一点，就像它在迭代器中所做的那样。

如果没有`await`关键字，手动使用延续意味着你必须编写等效于状态机的代码。这通常是异步编程困难的根源。

## 编写异步函数

对于任何异步函数，可以将`void`返回类型替换为`Task`，使方法本身变得有用地异步（并且可`await`）。不需要进一步的更改：

```cs
async Task PrintAnswerToLife()
{
  await Task.Delay (5000);
  int answer = 21 * 2;
  Console.WriteLine (answer);  
}
```

注意，我们在方法体中没有显式返回任务。编译器会创建任务，并在方法完成时（或未处理异常时）发出信号。这使得创建异步调用链变得容易：

```cs
async Task Go()
{
  await PrintAnswerToLife();
  Console.WriteLine ("Done");
}
```

（因为`Go`返回一个`Task`，所以`Go`本身是可以`await`的。）编译器将返回任务的异步函数展开为使用`TaskCompletionSource`间接创建任务的代码。

###### 注意

`TaskCompletionSource`是一个 CLR 类型，允许你创建可以手动控制的任务，并使用结果（或异常）标记其完成。与`Task.Run`不同，`TaskCompletionSource`不会在操作期间阻塞线程。它还用于编写 I/O 绑定的任务返回方法（例如`Task.Delay`）。

目标是确保当返回任务的异步方法完成时，执行可以通过延续跳回到等待它的地方。

### 返回`Task<TResult>`

如果方法体返回`TResult`，可以返回一个`Task<TResult>`：

```cs
async Task<int> GetAnswerToLife()
{
  await Task.Delay (5000);
  int answer = 21 * 2;
  // answer is int so our method returns Task<int>
  return answer;    
}
```

我们可以通过从`Go`中调用`PrintAnswerToLife`来演示`GetAnswerToLife`：

```cs
async Task Go()
{
  await PrintAnswerToLife();
  Console.WriteLine ("Done");
}
async Task PrintAnswerToLife()
{
  int answer = await GetAnswerToLife();
  Console.WriteLine (answer);
}
async Task<int> GetAnswerToLife()
{
  await Task.Delay (5000);
  int answer = 21 * 2;
  return answer;
}
```

异步函数使异步编程类似于同步编程。以下是我们调用图的同步等效版本，调用`Go()`会在阻塞五秒后给出相同的结果：

```cs
void Go()
{
  PrintAnswerToLife();
  Console.WriteLine ("Done");
}
void PrintAnswerToLife()
{
  int answer = GetAnswerToLife();
  Console.WriteLine (answer);
}
int GetAnswerToLife()
{
  Thread.Sleep (5000);
  int answer = 21 * 2;
  return answer;
}
```

这也展示了如何在 C#中设计异步函数的基本原则，即先以同步方式编写方法，然后将同步方法调用替换为`await`的异步方法调用。

## 并行性

我们刚刚演示了最常见的模式，即在调用后立即`await`返回任务的函数。这导致了逻辑上类似于同步等效的顺序程序流。

调用异步方法而不等待它允许随后的代码并行执行。例如，以下代码并发执行`PrintAnswerToLife`两次：

```cs
var task1 = PrintAnswerToLife();
var task2 = PrintAnswerToLife();
await task1; await task2;
```

在之后等待这两个操作，我们在那一点上“结束”了并行性（并重新抛出这些任务的任何异常）。 `Task` 类提供了一个名为 `WhenAll` 的静态方法，以稍微更有效地实现相同的结果。 `WhenAll` 返回一个任务，在你传递给它的所有任务完成时完成：

```cs
await Task.WhenAll (PrintAnswerToLife(),
                    PrintAnswerToLife());
```

`WhenAll` 被称为一个*任务组合器*。（`Task` 类还提供了一个任务组合器称为 `WhenAny`，它在提供给它的任何任务完成时完成。）我们在*C# 12 in a Nutshell*中详细介绍任务组合器。

## 异步 Lambda 表达式

我们知道普通的*命名*方法可以是异步的：

```cs
async Task NamedMethod()
{
  await Task.Delay (1000);
  Console.WriteLine ("Foo");
}
```

同样地，*未命名*方法（lambda 表达式和匿名方法），如果前面加上 `async` 关键字，也可以是异步的：

```cs
Func<Task> unnamed = async () =>
{
  await Task.Delay (1000);
  Console.WriteLine ("Foo");
};
```

你可以以相同的方式调用和等待它们：

```cs
await NamedMethod();
await unnamed();
```

当附加事件处理程序时，你可以使用异步 lambda 表达式：

```cs
myButton.Click += async (sender, args) =>
{
  await Task.Delay (1000);
  myButton.Content = "Done";
};
```

这比以下的更简洁，但效果相同：

```cs
myButton.Click += ButtonHandler;
...
async void ButtonHandler (object sender, EventArgs args)
{
  await Task.Delay (1000);
  myButton.Content = "Done";
};
```

异步 lambda 表达式也可以返回 `Task​<TRe⁠sult>`：

```cs
Func<Task<int>> unnamed = async () =>
{
  await Task.Delay (1000);
  return 123;
};
int answer = await unnamed();
```

## 异步流

使用 `yield return`，你可以编写一个迭代器；使用 `await`，你可以编写一个异步函数。*异步流*（来自 C# 8）结合了这些概念，让你能够编写一个等待的迭代器，异步地生成元素。此支持建立在以下一对接口的基础上，它们是我们在“枚举和迭代器”中描述的枚举接口的异步对应物：

```cs
public interface IAsyncEnumerable<out T>
{
  IAsyncEnumerator<T> GetAsyncEnumerator (...);
}

public interface IAsyncEnumerator<out T>: IAsyncDisposable
{
  T Current { get; }
  ValueTask<bool> MoveNextAsync();
}
```

`ValueTask<T>` 是包装 `Task<T>` 的结构体，行为上等效于 `Task<T>`，除了当任务同步完成时能够实现更高效的执行（这在枚举序列时经常发生）。 `IAsyncDisposable` 是 `IDisposable` 的异步版本，提供了在手动实现接口时执行清理操作的机会：

```cs
public interface IAsyncDisposable
{
  ValueTask DisposeAsync();
}
```

###### 注意

从序列获取每个元素的操作 (`MoveNextAsync`) 是一个异步操作，因此当元素逐个到达时，异步流非常合适（例如处理来自视频流的数据）。相比之下，以下类型在序列*整体*延迟时更为合适，但元素到达时却是一次性到达：

```cs
Task<IEnumerable<T>>
```

要生成一个异步流，你需要编写一个结合了迭代器和异步方法原理的方法。换句话说，你的方法应该包括 `yield return` 和 `await`，并且应该返回 `IAsyncEnumerable<T>`：

```cs
async IAsyncEnumerable<int> RangeAsync (
  int start, int count, int delay)
{
  for (int i = start; i < start + count; i++)
  {
    await Task.Delay (delay);
    yield return i;
  }
}
```

要消费一个异步流，请使用 `await foreach` 语句：

```cs
await foreach (var number in RangeAsync (0, 10, 100))
  Console.WriteLine (number);
```

