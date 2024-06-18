# 第八章：Currying 和 部分应用

Currying 和 部分应用 是从古老数学论文中直接衍生出来的两个更多的函数式概念。前者与印度食物毫不相关（尽管它确实很美味）¹，事实上，它是以卓越的美国数学家 Haskell Brooks Curry 命名的，他命名了不少于三种编程语言²。

Currying 源自 Haskell Curry 在组合逻辑上的工作，这为现代函数式编程提供了一个基础之一。我不会给出干燥的正式定义，我会通过例子来解释。这是一个有点像 C# 伪代码的加法函数示例：

```cs
public interface ICurriedFunctions
{
 decimal Add(decimal a, decimal b);
}

var curry = // some logic for obtaining an implementation of the interface

var answer = curry.Add(100, 200);
```

在这个例子中，我们期望 answer 简单地是 300（即 100+200），这确实是它会得到的结果。

不过，如果我只提供一个参数呢？像这样：

```cs
public interface ICurriedFunctions
{
 decimal Add(decimal a, decimal b);
}

var curry = // some logic for obtaining an implementation of the interface

var answer = curry.Add(100); // What could it be?
```

在这种情况下，如果这是一个假设的柯里化函数，你认为 *answer* 会返回什么？

在函数式编程中，我制定了一个经验法则 - 如果有一个问题，答案可能是“函数”。这在这里是适用的情况。

如果这是一个柯里化函数，那么 *answer* 变量将是一个函数。它将是原始 `Add` 函数的修改版本，但现在第一个参数已经固定为值 100 - 实际上这是一个新函数，它将 100 加到你提供的任何值上。

你可以像这样使用它：

```cs
public interface ICurriedFunctions
{
 decimal Add(decimal a, decimal b);
}

var curry = // some logic for obtaining an implementation of the interface

var add100 = curry.Add(100); // Func<decimal,decimal>, adds 100 to the input

var answerA = add100(200); // 300 -> 200+100
var answerB = add100(0); // 100 -> 0+100
var answerC = add100(900); // 1000 -> 900+100
```

基本上，这是一种从具有多个参数的函数开始，并从中创建多个更具体版本的方法。一个单一的基本函数可以成为许多不同的函数。如果你愿意的话，*可以*将它与 OO 概念的继承进行比较？但实际上它与继承完全*不*一样。实际上只有一个具有任何逻辑的基本函数 - 其余实际上是指向该基本函数的带参数的指针，准备向其输入。

不过，Currying 究竟有什么用？你怎么使用它？

让我解释一下……

# Currying 和大型函数

在上面我提供的“添加”示例中，我们只有一对参数，因此在可能进行 Currying 时，我们只有两种可能的处理方式：

1.  提供第一个参数，获取一个函数

1.  提供两个参数并获得一个值

Currying 如何处理具有超过 2 个基本参数的函数？为此，我将使用一个简单的 CSV 解析器的示例 - 即一个接收 CSV 文本文件，按行分割记录，然后使用一些分隔符（通常是逗号）再次分割记录内的各个属性。

让我们想象我写了一个用于加载一批书籍数据的解析器函数：

```cs
// Input in the format:
//
//title,author,publicationDate
//The Hitch-Hiker's Guide to the Galaxy,Douglas Adams,1979
//Dimension of Miracles,Robert Sheckley,1968
//The Stainless Steel Rat,Harry Harrison,1957
//The Unorthodox Engineers,Colin Kapp,1979

public IEnumerable<Book> ParseBooks(string fileName) =>
 File.ReadAllText(fileName)
  .Split("\r\n")
  .Skip(1) // Skip the header
  .Select(x => x.split(",").ToArray())
  .Select(x => new Book
  {
   Title = x[0],
   Author = x[1],
   PublicationDate = x[2]
  });

var bookData = parseBooks("books.csv");
```

这一切看起来都很好，但接下来的两组书籍有不同的格式。Books2.csv 使用管道符号而不是逗号分隔字段，而 Books3.csv 来自 Linux 环境，其行结束符是 "\n" 而不是 Windows 风格的 "\r\n"。

我们可以通过创建三个几乎相同的函数来解决这个问题。不过，我不喜欢不必要的复制，因为这会给未来想要维护代码库的开发人员带来太多问题。

一个更合理的解决方案是为可能发生变化的每一项添加参数，就像这样：

```cs
public IEnumerable<Book> ParseBooks(
 string lineBreak,
 bool skipHeader,
 string fieldDelimiter,
 string fileName
) =>
 File.ReadAllText(fileName)
  .Split(lineBreak)
  .Skip(skipHeader ? 1 : 0)
  .Select(x => x.split(fieldDelimiter).ToArray())
  .Select(x => new Book
  {
   Title = x[0],
   Author = x[1],
   PublicationDate = x[2]
  });

var bookData = ParseBooks(Environment.NewLine, true, ",", "books.csv");
```

现在，如果我想按照非函数化的方法使用这个函数，我将不得不填写每个可能的 CSV 文件风格的所有参数，就像这样：

```cs
var bookData1 = ParseBooks(Environment.NewLine, true,  ",", "books.csv");
var bookData2 = ParseBooks(Environment.NewLine, true, "|", "books2.csv");
var bookData3 = ParseBooks("\n", false, ",", "books3.csv");
```

实际上，柯里化意味着逐个提供参数。对柯里化函数的任何调用都会产生一个新函数，该新函数的参数少一个，或者如果所有基本函数的参数都已提供，则产生一个具体值。

来自前一个代码示例中已提供参数的调用可以替换成这样：

```cs
// First some magic that curries the parseBooks function
// I'll look into implementation details later, let's just
// understand the theory for now.

var curriedParseBooks = ParseBooks.Curry();

// these two have 3 parameters - string, string, string
var parseSkipHeader = curriedParseBooks(true);
var parseNoHeader = curriedParseBooks(false);

// 2 parameters
var parseSkipHeaderEnvNl = parseSkipHeader(Environment.NewLine);
var parseNoHeaderLinux = parseNoHeader("\n");

// 1 parameter each
var parseSkipHeaderEnvNlCommarDel = parseSkipHeaderEnvNl(",");
var parseSkipHeaderEnvNlPipeDel = parseSkipHeaderEnvNl("|");
var parseNoHeaderLinuxCommarDel = parseNoHeaderLinux(",");

// Actual data, Enumerables of Book data
var bookData1 = parseSkipHeaderEnvNlCommarDel("books.csv");
var bookData2 = parseSkipHeaderEnvNlPipeDel("books2.csv");
var bookData3 = parseNoHeaderLinuxCommarDel("books3.csv");
```

关键在于，柯里化将具有 X 个参数的函数转变为 X 个函数的序列，每个函数只有一个参数 - 最后一个函数返回最终结果。

如果你真的非常想的话，你甚至可以像上面那样写出这些函数调用！

```cs
var bookData1 = parseBooks(true)(Environment.NewLine)(",")("books.csv")
var bookData2 = parseBooks(true)(Environment.NewLine)("|")("books2.csv")
var bookData3 = parseBooks(true)("\n")(",")("books3.csv")
```

函数柯里化的第一个示例的要点是，我们逐步构建一个超特定版本的函数，它只接受文件名作为参数。除此之外，我们还将所有中间版本存储起来，以便在构建其他函数时重复使用。

我们实际上在这里做的是像用乐高积木搭建墙壁一样构建函数，其中每个积木都是一个函数。或者，如果你想从另一个角度来考虑，这是一个函数家族树，每个阶段的选择都导致家族中的一个分支：

![图像/第八章 _001.png](img/ch08_001.png)

###### 图 8-1\. A 解析书籍函数的家族树

另一个可能在实际中有用的例子是将日志函数分割为多个更具体的函数：

```cs
// For the sake of this exercise, the parameters are
// an enum (log type - warning, error, info, etc.) and a string
// containing a message to store in the log file
var logger = getLoggerFunction()
var curriedLogger = logger.Curry();

var logInfo = curriedLogger(LogLevel.Info);
var logWarning = curriedLogger(LogLevel.Warning);
var logError = curriedLogger(LogLevel.Error);

// You'd use them then, like this:

logInfo("This currying lark works a treat!");
```

这种方法有几个有用的特性：

+   事实上，我们最终只创建了一个单一的函数，但是从这个函数中，我们设法创建了至少 3 个可用的变体，这些变体可以传递并且只需一个文件名即可使用。这将代码复用提升到了一个新的水平！

+   还有所有中间函数也是可用的。这些函数可以直接使用，也可以用作创建额外新函数的起点。

在 C# 中，柯里化还有另一个用途。我将在下一节中讨论这个问题。

# 柯里化和高阶函数

如果我想用柯里化来创建几个函数来在摄氏度和华氏度之间转换，我会从像这样的柯里化基本算术操作开始：

```cs
// once again, the Currying process is just magic for now.
// Keep reading for the implementation

var add = ((x,y) => x + y).Curry();
var subtract = ((x,y) => y - x).Curry();
var multiply = ((x,y) => x * y).Curry();
var divide = ((x,y) => y / x).Curry();
```

使用这个方法，再加上前一章的映射函数，我们可以创建一组相当简洁的函数定义：

```cs
var celsiusToFahrenheit = x =>
 x.Map(multiply(9))
 .Map(divide(5))
 .Map(add(32));

var fahrenheitToCelsius = x=>
 x.Map(subtract(32))
  .Map(multiply(5))
  .Map(divide(9));
```

是否发现任何有用的内容很大程度上取决于用例 - 你实际上想要实现什么以及柯里化是否适用于它。

现在你可以在 C# 中使用它，就像你看到的那样。只要我们能找到在 C# 中实现它的方法……

# 在 .NET 中进行柯里化

所以，重要的问题是：更多基于函数式的语言可以在代码库中**所有**函数中本地实现这一点，那么在 .NET 中我们能做类似的事情吗？

简短的答案是差不多吧。

更长的答案是是的，有点。虽然不像在函数式语言（例如 F#）中那样优雅，那里这些功能都是开箱即用的。我们需要硬编码、创建静态类，或者在语言中稍微蹒跚并跳过一些 hoops。

硬编码的方法假设你只会以柯里化的方式使用函数，就像这样：

```cs
var Add = (decimal x) => (decimal y) => x + y;
var Subtract = (decimal x) => (decimal y) => y - x;
var Multiply = (decimal x) => (decimal y) => x * y;
var Divide = (decimal x) => (decimal y) => y / x;
```

注意每个函数中有两组箭头，意味着我们定义了一个返回另一个 `Func` 委托的 `Func` 委托 - 即实际类型是 `Func<decimal, Func<decimal, decimal>>`。只要你使用的是 C# 10 或更高版本，你就能利用 `var` 关键字隐式地获取类型，就像上面的例子一样。较旧版本的 C# 可能需要在代码示例中显式声明委托的类型。

第二个选项是创建一个静态类，可以在代码库中的任何地方引用。你可以随意命名它，但我选择用 `F` 来表示函数式。

```cs
public static class CurryingExtensions
{
  public static Func<T1, Func<T2, TOut>> Curry<T1, T2, TOut>(
   Func<T1, T2, TOut> functionToCurry) =>
    (T1 x) => (T2 y) => functionToCurry(x, y);

  public static Func<T1, Func<T2, Func<T3, TOut>>> Curry<T1, T2, T3, TOut>(
   Func<T1, T2, T3, TOut> functionToCurry) =>
    (T1 x) => (T2 y) => (T3 z) => functionToCurry(x, y, z);

  public static Func<T1, Func<T2, Func<T3, Func<T4, TOut>>>> Curry<T1, T2, T3, T4, TOut>(
   Func<T1, T2, T3, T4, TOut> functionToCurry) =>
    (T1 x) => (T2 y) => (T3 z) => (T4 a) => functionToCurry(x, y, z, a);
}
```

这实际上在调用被柯里化的最终函数和使用它的代码区域之间放置了多层 `Func` 委托。

这种方法的缺点是，我们必须为每种可能的参数数量创建一个柯里化方法。我的示例涵盖了具有 2、3 或 4 个参数的函数。具有更多参数的函数需要构建另一个 Curry 方法，根据相同的公式。

另一个问题是，Visual Studio 无法隐式确定传入的函数的类型，因此有必要在调用 F.Curry 时定义被柯里化的函数，并声明每个参数的类型，就像这样：

```cs
var Add = F.Curry((decimal x, decimal y) => x + y);
var Subtract = F.Curry((decimal x, decimal y) => y - x);
var Multiply = F.Curry((decimal x, decimal y) => x * y);
var Divide = F.Curry((decimal y, decimal y) => y / x);
```

最后一个选项 - 也是我更喜欢的选项 - 是使用扩展方法来减少必需的样板代码。对于具有 2、3 和 4 个参数的函数，定义如下：

```cs
public static class Ext
{
	public static Func<T1,Func<T2, T3>> Curry<T1,T2,T3>(
	 this Func<T1,T2,T3> @this) =>
		 (T1 x) => (T2 y) => @this(x, y);

	public static Func<T1,Func<T2,Func<T3,T4>>>Curry<T1,T2,T3,T4>(
	 this Func<T1,T2,T3,T4> @this) =>
		 (T1 x) => (T2 y) => (T3 z) => @this(x, y, z);

	public static Func<T1,Func<T2,Func<T3,Func<T4,T5>>>>Curry<T1,T2,T3,T4,T5>(
	 this Func<T1,T2,T3,T4,T5> @this) =>
		 (T1 x) => (T2 y) => (T3 z) => (T4 a) => @this(x, y, z, a);
}
```

那是一个相当丑陋的代码块，不是吗？好消息是，你可以把它放在代码库的深处，并且大部分时间都可以忘记它的存在。

使用方式如下：

```cs
// specifically define the function on one line
// it has to be stored as a `Func` delegate, rather than a
// Lambda expression
var Add = (decimal x, decimal y) => x + y;
var CurriedAdd = Add.Curry();

var add10 = CurriedAdd(10);
var answer = add10(100);
// answer = 110
```

那就是柯里化。你们中眼尖的可能已经注意到，这一章被称为“柯里化**和**部分应用”。

什么是部分应用？嗯……既然你这么客气地问了……

# 部分应用

部分应用的工作原理与柯里化非常类似，但它们之间有微妙的区别。这两个术语经常被误用，互换使用。

Currying **专门** 处理将带有一组参数的函数转换为一系列连续的函数调用，每个调用只有一个参数（技术术语是 *一元* 函数）。

部分应用，另一方面，允许您一次应用尽可能多的参数。有数据出现如果所有的参数都填写完毕。

返回到我之前的解析函数示例，这些是我们正在处理的格式：

+   book1 - Windows 行结束，头部，字段用逗号

+   book2 - Windows 行结束，头部，字段用管道符号

+   book3 - Linux 行结束，没有头部，字段用逗号

使用柯里化方法，我们为设置 Book3 的每个参数创建了中间步骤，即使它们最终只用于每个参数的唯一用途。我们还为 book1 和 book2 的 SkipHeader 和行结束参数做了同样的事情，尽管它们是相同的。

可以这样做来节省空间：

```cs
var curriedParseBooks = parseBooks.Curry();

var parseNoHeaderLinuxCommaDel = curriedParseBooks(false)("\n")(",");

var parseWindowsHeader = curriedParseBooks(true)(Environment.NewLine);
var parseWindowsHeaderComma = parseWindowsHeader(",");
var parseWindowsHeaderPipe = parseWindowsHeader("|");

// Actual data, Enumerables of Book data
var bookData1 = parseWindowsHeaderComma("books.csv");
var bookData2 = parseWindowsHeaderPipe("books2.csv");
var bookData3 = parseNoHeaderLinuxCommaDel("books3.csv");
```

但是，如果我们能够简洁地使用部分应用来应用这 2 个参数，那就更清晰了。

```cs
// I'm using an extension method called Partial to apply
// parameters.  Check out the next section for implementation details

var parseNoHeaderLinuxCommarDel = ParseBooks.Partial(false,"\n",",");

var parseWindowsHeader =
 curriedParseBooks.Partial(true,Environment.NewLine);
var parseWindowsHeaderComma = parseWindowsHeader.Partial(",");
var parseWindowsHeaderPipe = parseWindowsHeader.Partial("|");

// Actual data, Enumerables of Book data
var bookData1 = parseWindowsHeaderComma("books.csv");
var bookData2 = parseWindowsHeaderPipe("books2.csv");
var bookData3 = parseNoHeaderLinuxCommarDel("books3.csv");
```

我认为这是一个相当优雅的解决方案，它仍然允许我们在需要时具有可重用的中间函数，但仍然只有一个基本函数。

在下一节中，我将向您展示如何实际实现这一点。

# .NET 中的部分应用

这是个坏消息。没有任何一种方式可以优雅地在 C# 中实现部分应用。您需要做的是为每个参数组合创建一个扩展方法。

在我刚刚给出的示例中，我需要：

+   4 个参数变成 1 个参数，用于 `parseNoHeaderLinuxCommaDel`

+   4 个参数变成 2 个参数，用于 `parseWindowsHeader`

+   2 个参数变成 1 个参数，用于 `parseWindowsHeaderComma` 和 `parseWindowsHeaderPipe`

这是每个示例将会是什么样子：

```cs
public static class PartialApplicationExtensions
{
// 4 parameters to 1
 public static Func<T4,TOut> Partial<T1,T2,T3,T4,TOut>(
  this Func<T1,T2,T3,T4,TOut> f,
  T1 one, T2 two, T3 three) => (T4 four) => f(one, two, three, four);

// 4 parameters to 2
 public static Func<T3,T4,TOut>Partial<T1,T2,T3,T4,TOut>(
  this Func<T1,T2,T3,T4,TOut> f,
  T1 one, T2 two) => (T3 three, T4 four) => f(one, two, three, four);

 // 2 parameters to 1
 public static Func<T2, TOut> Partial<T1,T2,TOut>(
  this Func<T1,T2,TOut> f, T1 one) =>
   (T2 two) => f(one, two);

}
```

如果您决定部分应用是一种您想要追求的技术，那么您可以根据需要将部分方法添加到代码库中，或者将一段时间留出来创建您可能需要的尽可能多的部分方法。

# 结论

Currying 和部分应用是函数式编程中两个强大且相关的概念。不幸的是，它们在 C# 中并不原生支持，并且不太可能会支持。

它们可以通过使用静态类或扩展方法来实现，这为代码库增加了一些样板代码 - 这有些讽刺，考虑到这些技术部分是为了减少样板。

鉴于 C# 不像 F# 和其他函数式语言那样完全支持高阶函数。C# 不能像转换为 `Func` 委托那样传递函数。

即使函数被转换为 `Func`，Roslyn 编译器也不能始终正确确定参数类型。

在 C#领域，这些技术可能永远不如其他语言那么有用。尽管如此，它们在减少模板代码和实现比其他方式更高的代码可重用性方面仍然有其用处。

是否使用它们是个人偏好的问题。我不认为它们对于功能型 C#是必需的，但还是值得探索的。

在我们的下一章中，我们将探索在功能型 C#中无限循环的更深层奥秘，以及什么是尾递归调用。

¹ 美食提示：如果你曾经来到孟买，一定要尝试在 Shivaji Park 的 Tibb’s Frankie，你不会后悔的！

² 显然是 Haskell，但还有 Brook 和 Curry 这两种不那么出名的语言。
