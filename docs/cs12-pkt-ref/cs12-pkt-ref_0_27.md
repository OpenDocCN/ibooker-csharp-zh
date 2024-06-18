# 第二十七章：枚举和迭代器

## 枚举

*枚举器* 是一种只读的、单向的值序列游标。如果 C# 做了以下任何一件事情，则将类型视为枚举器：

+   具有名为 `MoveNext` 的公共无参数方法和名为 `Current` 的属性

+   实现了 `System.Collections.Generic.IEnumerator<T>`

+   实现了 `System.Collections.IEnumerator`

`foreach` 语句迭代 *可枚举* 对象。可枚举对象是序列的逻辑表示。它本身不是游标，而是生成自身上的游标的对象。如果 C# 做了以下任何一件事情，则将类型视为可枚举（按照此顺序进行检查）：

+   具有公共的无参数方法 `GetEnumerator`，返回一个枚举器

+   实现了 `System.Collections.Generic.IEnumerable<T>`

+   实现了 `System.Collections.IEnumerable`

+   （自 C# 9 起）可以绑定到名为 `GetEnumerator` 的 *扩展方法*，该方法返回一个枚举器（参见 “扩展方法”）

枚举模式如下：

```cs
class *Enumerator*   // Typically implements IEnumerator<T>
{
  public *IteratorVariableType* Current { get {...} }
  public bool MoveNext() {...}
}
class *Enumerable*   // Typically implements IEnumerable<T>
{
  public *Enumerator* GetEnumerator() {...}
}
```

这是使用 `foreach` 语句迭代单词 *beer* 中字符的高级方法：

```cs
foreach (char c in "beer") Console.WriteLine (c);
```

这是在不使用 `foreach` 语句的情况下低级迭代单词 *beer* 中字符的方法：

```cs
using (var enumerator = "beer".GetEnumerator())
  while (enumerator.MoveNext())
  {
    var element = enumerator.Current;
    Console.WriteLine (element);
  }
```

如果枚举器实现了 `IDisposable`，则 `foreach` 语句还充当 `using` 语句，隐式处理枚举器对象。

## 集合初始化器和集合表达式

您可以一步实例化和填充可枚举对象。例如：

```cs
using System.Collections.Generic;

List<int> list = new List<int> {1, 2, 3};
```

自 C# 12 起，您可以使用 *集合表达式* 进一步缩短最后一行（请注意方括号）：

```cs
List<int> list = [1, 2, 3];
```

###### 注意

集合表达式是 *目标类型化* 的，这意味着 `[1,2,3]` 的类型取决于它被分配的类型（在本例中为 `List<int>`）。在下面的示例中，目标类型是数组：

```cs
int[] array = [1, 2, 3];
```

目标类型化意味着您可以在编译器可以推断类型的其他情况下省略类型，例如在调用方法时：

```cs
Foo ([1, 2, 3]);
void Foo (List<int> numbers) { ... }
```

编译器将其转换为以下内容：

```cs
List<int> list = new List<int>();
list.Add (1); list.Add (2); list.Add (3);
```

这要求可枚举对象实现 `System.Collections.IEnumerable` 接口，并且具有 `Add` 方法，该方法对于调用具有适当数量参数的方法非常重要。您可以像下面这样初始化字典（实现了 `System.Collections.IDictionary` 接口的类型）：

```cs
var dict = new Dictionary<int, string>()
{
  { 5, "five" },
  { 10, "ten" }
};
```

或者更简洁地说：

```cs
var dict = new Dictionary<int, string>()
{
  [5] = "five",
  [10] = "ten"
};
```

后者不仅适用于字典，还适用于任何具有索引器的类型。

## 迭代器

而 `foreach` 语句是枚举器的 *消费者*，迭代器则是枚举器的 *生产者*。在这个例子中，我们使用迭代器来返回斐波那契数列（其中每个数字是前两个数字的和）：

```cs
foreach (int fib in Fibs (6))
  Console.Write (fib + "  ");

IEnumerable<int> Fibs (int fibCount)
{
  for (int i=0, prevFib=1, curFib=1; i<fibCount; i++)
  {
 yield return prevFib;
    int newFib = prevFib+curFib;
    prevFib = curFib;
    curFib = newFib;
  }
}
*OUTPUT: 1  1  2  3  5  8*
```

而 `return` 语句表达“这是你要求我从这个方法返回的值”，`yield return` 语句则表达“这是你要求我从这个枚举器中 `yield` 的下一个元素”。在每个 `yield` 语句上，控制权返回给调用者，但被调用者的状态保持不变，以便方法可以在调用者枚举下一个元素时继续执行。此状态的生命周期与枚举器绑定，因此当调用者完成枚举时，状态可以被释放。

###### 注意

编译器将迭代器方法转换为实现 `IEnumerable<T>` 和/或 `IEnumerator<T>` 的私有类。迭代器块内部的逻辑被“反转”并插入到编译器生成的枚举器类的 `MoveNext` 方法和 `Current` 属性中，它们实际上成为了状态机。这意味着当您调用迭代器方法时，实际上只是实例化了编译器生成的类；您的代码只有在您开始枚举生成的序列时才会运行，通常是使用 `foreach` 语句。

## 迭代器语义

迭代器是一个方法、属性或索引器，其中包含一个或多个 `yield` 语句。迭代器必须返回以下四个接口之一（否则编译器将生成错误）：

```cs
System.Collections.IEnumerable
System.Collections.IEnumerator
System.Collections.Generic.IEnumerable<T>
System.Collections.Generic.IEnumerator<T>
```

返回 *enumerator* 接口的迭代器通常使用较少。它们在编写自定义集合类时很有用：通常，您将迭代器命名为 `GetEnumerator` 并让您的类实现 `IEnumerable<T>`。

返回 *enumerable* 接口的迭代器更常见，并且使用起来更简单，因为您不需要编写集合类。编译器在幕后会生成一个实现 `IEnumerable<T>`（以及 `IEnumerator<T>`）的私有类。

### 多个 yield 语句

一个迭代器可以包含多个 `yield` 语句：

```cs
foreach (string s in Foo())
  Console.Write (s + " ");    // One Two Three

IEnumerable<string> Foo()
{
  yield return "One";
  yield return "Two";
  yield return "Three";
}
```

### yield break

在迭代器块中使用 `return` 语句是非法的；相反，您必须使用 `yield break` 语句来指示迭代器块应该提前退出，而不返回更多元素。我们可以修改 `Foo` 如下所示来演示：

```cs
IEnumerable<string> Foo (bool breakEarly)
{
  yield return "One";
  yield return "Two";
  if (breakEarly) yield break;
  yield return "Three";
}
```

## 组合序列

迭代器非常易于组合。我们可以通过将以下方法添加到类中来扩展我们的斐波那契示例：

```cs
Enumerable<int> EvenNumbersOnly (
  IEnumerable<int> sequence)
  {
    foreach (int x in sequence)
      if ((x % 2) == 0)
        yield return x;
  }
}
```

我们可以如下输出偶数斐波那契数：

```cs
foreach (int fib in EvenNumbersOnly (Fibs (6)))
  Console.Write (fib + " ");   // 2 8
```

每个元素直到最后一刻才计算——在被请求的 `MoveNext()` 操作时。图 5 展示了随时间推移的数据请求和数据输出。

![组合序列](img/c12p_0105.png)

###### 图 5\. 组合序列

迭代器模式的组合性在构建 LINQ 查询中至关重要。

