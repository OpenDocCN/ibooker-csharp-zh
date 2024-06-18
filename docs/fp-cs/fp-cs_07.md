# 第七章：函数式流程

调用外部系统，无论是数据库、Web API 还是其他，都是一件很头疼的事情，不是吗？在使用你的数据 - 函数中最重要的部分之前，你必须：

1.  捕获并处理任何异常。也许是网络故障，或者数据库服务器离线了？

1.  检查从数据库返回的内容不是 NULL

1.  检查是否有一个实际合理的数据集，即使它不是 null。

这就是许多繁琐的样板代码，所有这些都妨碍了你的实际业务逻辑。

使用上一章的 Maybe 判别联合将在处理未找到记录或遇到错误时有所帮助，但即便如此，仍然需要一些样板代码。

如果我告诉你有一种方法，你永远不用再见到未处理的异常了？不仅如此，你甚至不需要再使用 Try/Catch 块了。至于 Null 检查？忘了吧。你再也不用做那些事了。

不相信我？好吧，准备好，我要向你介绍函数式编程中我最喜欢的一个特性。这是我在日常工作中经常使用的，希望读完这一章后，你也会用到。

# 再次审视 Maybe

提到上一章的 Maybe 判别联合 - 我现在想重新讨论它，但这次我要向你展示它可以比你想象的更加有用。

我要做的是在前几章使用过的 Map 扩展方法中加入一个版本。如果你还记得第五章“链式函数”中的`Map`组合子，它类似于 LINQ 的 Select 方法，只不过它作用于整个源对象，而不是其个别元素。

这一次，我要在 Map 里面加入一些逻辑，这将决定将产生哪种实际类型。这一次我给它起了一个不同的名字 - Bind¹

```cs
public static Maybe<TOut> Bind<TIn, TOut>(this Maybe<TIn> @this, Func<TIn, TOut> f)
{
	try
	{
		Maybe<TOut> updatedValue = @this switch
		{
			Something<TIn> s when !EqualityComparer<TIn>.Default.Equals(s.Value, default) =>
				new Something<TOut>(f(s.Value)),
			Something<TIn> _ => new Nothing<TOut>(),
			Nothing<TIn> _ => new Nothing<TOut>(),
			Error<TIn> e => new Error<TOut>(e.ErrorMessage),
			_ => new Error<TOut>(new Exception("New Maybe state that isn't coded for!: " + @this.GetType()))
		};
		return updatedValue;
	}
	catch (Exception e)
	{
		return new Error<TOut>(e);
	}
}
```

那么，这里发生了什么？可能有几种情况：

+   当前值为`This` - 即被 Maybe 持有的当前对象是一个`Something` - 即一个实际值，并且持有的值是非默认值²，在这种情况下，执行提供的函数，并将其输出作为新的`Something`返回。

+   当前值为`This`是一个`Something`，但其内部的值是默认值（在大多数情况下是 Null），在这种情况下，我们现在返回一个`Nothing`。

+   当前值为`This`是一个 Nothing，在这种情况下，返回另一个 Nothing。没有必要再做其他事情。

+   当前值为`This`是一个错误。同样，除了将其传递下去外，没有任何其他做的必要。

这一切的意义何在？好吧，想象一下以下的过程化代码：

```cs
public string MakeGreeting(int employeeId)
{
 try
 {
  var e = this.empRepo.GetById(employeeId);
  if(e != null)
  {
   return "Hello " + e.Salutation + " " + e.Name
  }

  return "Employee not found";
 }

 catch(Exception e)
 {
   return "An error occurred: " + e.Message;
 }
}
```

当你看它时，这段代码的实际目的非常简单。获取一个员工。如果没有问题，向他们打招呼。但是，由于空值和未处理的异常存在，我们不得不编写大量的防御性代码。空检查和 `Try/Catch` 块。不仅在这里，在我们的整个代码库中都是如此。

更糟糕的是，我们让调用此函数的代码知道该如何处理它。我们如何表示发生了错误，或者找不到员工？在我的例子中，我只是返回一个字符串，供我们编写的任何应用程序盲目显示。另一个选择是返回带有元数据附加的返回对象（例如 `bool DataFound`，`bool ExceptionOccurred`，`Exception CapturedException` - 这样的东西）。

然而，使用 `Maybe` 和 `Bind` 函数，这些都是不必要的。可以将代码重写如下：

```cs
public Maybe<string> MakeGreeting(int employeeId) =>
 new Something(employeeId)
  .Bind(x => this.empRepo.GetById(x))
  .Bind(x => "Hello " + x.Salutation + " " + x.Name);
```

想想我列出的每个 `Bind` 的可能结果。

如果员工存储库返回一个空值，则下一个 `Bind` 调用将标识为某个具有默认值（即空）的东西，并且不执行构造问候字符串函数，而是返回一个 `Nothing`。

如果存储库发生错误（可能是网络连接问题，无法预测或防止的问题），则简单地传递错误，而不执行函数。

我要表达的最终观点是，组装问候语的箭头函数只有在前一步 a) 返回了实际值且 b) 没有抛出未处理的异常时才会执行。

这意味着上面使用 `Bind` 方法编写的小函数在功能上与以前的版本完全相同，覆盖了防御性代码。

情况会变得更好…​

我们不再返回字符串，而是返回 `Maybe<string>`。这是一个可以用来通知调用我们函数的结果的辨别联合，告诉它是否工作等等。这可以在外部世界用于决定如何处理结果值，或者可以在后续的 `Bind` 调用链中使用。

要么像这样：

```cs
public Interface IUserInterface
{
 void WriteMessage(string s);
}

// Bit of magic here because it doesn't matter
this.UserInterface = Factory.MakeUserInterface();

var message = makeGreetingResult switch
{
 Something s => s.Value,
 Nothing _ => "Hi, but I've never heard of you.",
 Error _ => "An error occurred, try again"
};

this.UserInterface.WriteMessage(message);
```

或者，您可以调整 `UserInterface` 模块，以便它以 `Maybe` 作为参数：

```cs
public Interface IUserInterface
{
 void WriteMessage(Maybe<string> s);
}

// Bit of magic here because it doesn't matter
this.UserInterface = Factory.MakeUserInterface();

var logonMessage = MakeGreeting(employeeId)
 .Bind(x => x + Environment.NewLine + MakeUserInfo(employeeId));
this.UserInterface.WriteMessage(logonMessage);
```

在接口中用 `Maybe<T>` 替换具体值表明，消费它的类不能确定操作是否有效，并迫使消费类考虑每种可能性及其处理方式。它还完全让消费类决定如何响应。没有必要让返回 `Maybe` 的类对接下来发生的事情感兴趣。

我遇到的这种编程风格的最佳描述是在 Scott Wlashin 的讲座和相关文章中，称为铁路导向编程 ³。

Wlashin 将这个过程描述为像一个铁路线，有一系列的道岔。每组道岔都是一个 Bind 调用。火车从 Something 线开始，每次执行传递给 Bind 的函数时，火车要么继续前进到下一组道岔，要么切换到 Nothing 路径，直接滑向线路末端的车站，而无需做更多工作。

这是一种美丽、优雅的编码方式，大大减少了样板代码。

要是这种结构有一个方便的技术术语就好了。哦等等，有！它叫做 Monad！

我在开头就说过它们可能会突然出现在某处。如果有人告诉你 Monad 很复杂，我希望你现在能看到他们错了。

Monad 就像是围绕某种值的包装。就像一个信封或者一个墨西哥卷饼。它们保存值，但并不评论它到底设置了什么。

它们的作用是让你能够挂接函数，提供一个安全的环境进行操作，而不必担心负面后果，比如空引用异常。

Bind 函数就像接力赛一样 - 每次调用都执行某种操作，然后将其值传递给下一个运行者。它还为您处理错误和空值，因此您无需担心编写太多的防御性代码。

如果你愿意，可以想象它就像一个防爆箱。你有一个想要打开的包裹，但你不知道它是安全的，像一封信⁴，还是可能是爆炸品，等待你打开盖子时将你击倒。如果你把它放在 Monad 容器中，它可以安全地打开或爆炸，但 Monad 会保护你免受后果。

那基本上就是这样。嗯，大多数情况下是这样的。

在本章的其余部分，我将考虑我们可以用 Monad 做什么，以及还有哪些其他类型的 Monad。不过不用担心，现在“难”部分已经过去了，如果你已经通过了这一点，仍然与我在一起，那么这本书的其余部分将会很轻松⁵。

## Maybe 和调试

有时关于 Bind 语句串的评论是，在 Visual Studio 中使用调试工具逐步跟踪更改变得更难。特别是当你有这样的场景时：

```cs
var returnValue = idValue.ToMaybe()
  .Bind(transformationOne)
  .Bind(transformationTwo)
  .Bind(transformationThree);
```

实际上，在大多数 Visual Studio 的版本中都是可能的，但你需要确保你不断地按“Step-in”键进入 Bind 调用内部的箭头函数。如果值没有正确计算，这对于了解正在发生的事情还不是最好的。当你考虑 Step-in 将进入 Maybe 的 Bind 函数并需要执行更多步骤才能看到箭头函数的结果时，情况会更糟。

我倾向于每行写一个 Bind，每个 Bind 存储一个包含它们各自输出的变量：

```cs
var idMaybe = idValue.ToMaybe();
var transOne = idMaybe.Bind(x => transformationOne(x));
var transTwo = transOne.Bind(x => transformationTwo(x));
var returnValue = transTwo.Bind(x => transformationThree(x));
```

从功能上讲，这两个示例是相同的，只是我们分别捕获每个输出，而不是立即将其馈送到另一个函数并丢弃它们。

第二个示例更容易诊断问题，因为您可以检查每个中间值。由于函数式编程技术的使用，一旦设置变量就不修改它 - 这意味着处理过程中的每个中间步骤都是固定的，可以详细了解发生了什么事情，以及在出现错误时如何以及在哪里出错。

这些中间值将在它们所属的更大函数的整个生命周期内保持在作用域内。因此，如果是一个特别大的函数，并且其中一个中间值很大，那么合并它们以尽早去除这个大中间值可能是值得的。

这个决定主要是个人风格的问题，以及一个或两个代码库的约束。无论您选择哪个都可以。

## Map vs Bind

严格来说，按照函数范式实现 Bind 函数时，我并没有这样做。

*应该* 有两个附加到 Maybe 的函数：Map 和 Bind。它们几乎相同，但有一个小而微妙的区别。

Map 函数与我在上一节中描述的函数类似 - 它连接到 Maybe<T1>，需要一个函数来从 Maybe 内部获取类型 T1 的值，并要求您将其转换为类型 T2。

实际的 Bind 需要您传入一个返回新类型的 Maybe 的函数 - 即 Maybe<T2>。它仍然返回与 Map 函数相同的结果，

```cs
public static Maybe<TOut> Map<TIn, TOut>(this Maybe<TIn> @this, Func<TIn, TOut> f) => // Some implementation here

public static Maybe<TOut> Bind<TIn, TOut>(this Maybe<TIn> @this, Func<TIn, Maybe<TOut>> f) => // Some Other implementation here
```

例如，如果您有一个调用数据库并返回 `Maybe<IEnumerable<Customer>>` 类型的函数，表示可能找到或未找到客户的列表 - 那么您将使用 Bind 函数调用它。

将 `Enumerable` 中的客户端转换为其他形式的任何后续链式函数调用都可以使用 Map 调用，因为这些变化是数据到数据的转换，而不是数据到可能性的转换。

这是如何实现一个正确的 Bind 的方法：

```cs
public static Maybe<TOut> Bind<TIn, TOut>(this Maybe<TIn> @this, Func<TIn, Maybe<TOut>> f)
{
	try
	{
		var returnValue = @this switch
		{
			Something<TIn> s => f(s.Value),
			_ => new Nothing<TOut>()
		};
		return returnValue;
	}
	catch (Exception _)
	{
		return new Nothing<TOut>();
	}
}
```

下面是如何使用它的示例：

```cs
public Interface CustomerDataRepo
{
 Maybe<Customer> GetCustomerById(int customerId);
}

public string DescribeCustomer(int customerId) =>
 new Something<int>(customerId)
  .Bind(x => this.customerDataRepo.GetCustomerById(x))
  .Map(x => "Hello " + x.Name);
```

使用这个新的 Bind 函数并将之前的一个命名为 Map，您将更接近函数范式。

但是在生产代码中，我个人不这样做。我只是为两个目的都使用一个名为 Bind 的函数。

你可能会问，为什么呢？

这主要是为了防止混淆，说实话。JavaScript 有一个名为 Map 的原生函数，但它的操作类似于 C# 中的 `Select`，作用于数组的各个元素。在 C# 中，Jimmy Bogard 的 AutoMapper 库⁶ 中也有一个 Map 函数，用于将一个对象数组从一种类型转换为另一种类型。

由于许多 C#代码库中已经在使用两个 Map 函数的情况下，我认为在其中添加另一个 Map 函数可能会让查看我的代码的其他人感到困惑。因此，我为所有目的使用 Bind，因为在 C#或 JavaScript 中没有任何地方已经存在 Bind 函数 - 除了实现函数式范式的库中。

你可以自行选择使用更严格准确的同时使用 Map 和 Bind 的版本，或者在我看来更少混淆和更实用的路线，即简单地使用多个 Bind 函数实现每个目的。

我将继续假设这本书的其余部分采用第二种选项。

## Maybe 和原始类型

这听起来像是一本从未写成的惊险故事小说的标题。可能涉及我们的女英雄 - Maybe 船长，在一个由侵略性洞穴居民族群组成的失落文明中挥舞着救援。

实际上，在 C#中，原始类型是一组不默认为 Null 的内置类型之一。以下是其中的一些：

+   布尔型

+   字节型

+   有符号字节型

+   字符型

+   十进制

+   双精度浮点型

+   单精度浮点型

+   整型

+   无符号整型

+   有符号自然数型

+   无符号自然数型

+   长整型

+   无符号长整型⁷

+   短整型

+   无符号短整型

这里的重点是，如果我在之前章节的`Bind`函数中使用任何这些类型，并将它们的值设置为 0，它们将违反对`default`的检查，因为大多数这些类型的默认值为 0⁸

这里有一个单元测试的示例，会失败（我正在使用 XUnit 和 FluentAssertions，以获得更友好、易读的断言风格）：

```cs
[Fact]
public Task primitive_types_should_not_default_to_nothing()
{
	var input = new Something<int>(0);
	var output = input.Bind(x => x + 10);
	(output as Something<int>).Value.Should().Be(10);
}
```

这个测试将一个值为 0 的整数存储在一个 Maybe 中，然后尝试将其`Bind`为一个比原值高 10 的值 - 即应该等于 10。在现有代码中，`Bind`内部的 switch 操作会将值 0 视为默认值，并将返回类型从`Something<int>`切换为`Nothing<int>`，并且不会执行加 10 的函数，这意味着在我的单元测试中*输出*会被切换为 null，并且测试将因为空引用异常而失败。

有人可能会说，这并不是正确的行为，0 是一个整数的有效值。

只需在`Bind`函数中添加一行代码即可轻松解决：

```cs
public static Maybe<TOut> Bind<TIn, TOut>(this Maybe<TIn> @this, Func<TIn, TOut> f)
{
	try
	{
		Maybe<TOut> updatedValue = @this switch
		{
			Something<TIn> s when !EqualityComparer<TIn>.Default.Equals(s.Value, default) =>
				new Something<TOut>(f(s.Value)),
				// This is the new line
			Something<TIn> s when s.GetType().GetGenericArguments()[0].IsPrimitive => new Something<TOut>(f(s.Value)),
			Something<TIn> _ => new Nothing<TOut>(),
			Nothing<TIn> _ => new Nothing<TOut>(),
			Error<TIn> e => new Error<TOut>(e.ErrorMessage),
			_ => new Error<TOut>(new Exception("New Maybe state that isn't coded for!: " + @this.GetType()))
		};
		return updatedValue;
	}
	catch (Exception e)
	{
		return new Error<TOut>(e);
	}
}
```

新行检查`Maybe<T>`的第一个泛型参数 - 即`T`的“真实”类型。我在本节开头列出的所有类型的`IsPrimitive`值都将设置为`true`。

如果我使用修改后的`Bind`函数重新运行我的单元测试，那么值为 0 的`int`仍然不会匹配不是`default`的检查，但下一行会匹配，因为 int 是一个原始类型。

现在，这意味着所有原始类型都*不能*成为`Nothing<T>`。这是对错是你自己评估的问题。

如果 *T* 是一个 `bool`，你可能会认为它是 `Nothing<T>`。如果是这种情况，需要在第一两行之间的 switch 中添加另一种情况来处理 *T* 特定的情况为 `bool` 的情况。

如果一个布尔 `false` 被传递到执行计算的函数中可能也很重要。正如我所说的，这是一个你最好自己回答的问题。

完全避免这种情况的一种方法是始终传递一个可空的类作为 *T*，这样你就可以确保在尝试判断你所看到的是 `Something` 还是 `Nothing` 时能得到正确的行为。

## Maybe 和 日志记录

在专业环境中考虑使用单子的另一件事情是至关重要的开发者工具 - 日志记录。关于函数进度状态的日志信息通常至关重要，不仅仅是错误信息，还包括各种重要信息。

当然可以做类似这样的事情：

```cs
var idMaybe = idValue.ToMaybe();
var transOne = idMaybe.Bind(x => transformationOne(x));
if(transOne is Something<MyClass> s)
{
 this.Logger.LogInformation("Processing item " + s.Value.Id);
}
else if (transOne is Nothing<MyClass>)
{
 this.Logger.LogWarning("No record found for " + idValue");
}
else if (transOne is Error<MyClass> e)
{
 this.Logger.LogError(e, "An error occurred for " + idValue);
}
```

如果你做了很多这样的操作，这可能会失控。特别是在整个过程中有许多需要记录的 Bind。

也许可以直到最后才留出错误日志，或者甚至在控制器中，或者最终发起此请求的任何其他地方。错误消息将不经修改地从一方传递到另一方。但这仍然会留下偶尔的信息或警告用途的日志消息。

我更喜欢添加扩展方法到 Maybe 中以提供一组事件处理函数：

```cs
public static class MaybeLoggingExtensions
{
 public static Maybe<T> OnSomething(this Maybe<T> @this, Action<T> a)
 {
  if(@this is Something<T>)
  {
   a(@this);
  }

  return @this;
 }

 public static Maybe<T> OnNothing(this Maybe<T> @this, Action a)
 {
  if(@this is Nothing<T> _)
  {
   a();
  }

  return @this;
 }
}

 public static Maybe<T> OnError(this Maybe<T> @this, Action<Exception> a)
 {
  if(@this is Error<T> e)
  {
   a(e.CapturedError);
  }

  return @this;
 }
```

那么，我使用它的方式更像是这样：

```cs
var idMaybe  idValue.ToMaybe();
var transOne = idMaybe.Bind(x => transformationOne(x))
 .OnSomething(x => this.Logger.LogInformation("Processing item " + x.Id))
 .OnNothing(() => this.Logger.LogWarning("No record found for " + idValue))
 .OnError(e => this.Logger.LogError(e, "An error occurred for " + idValue));
```

这相当可用，尽管它确实有一个缺点。

OnNothing 和 OnError 状态将从 Bind 传播到未修改的 Bind，因此如果你有一长串带有 OnNothing 或 OnError 处理程序函数的 Bind 调用，它们每次都会触发。就像这样：

```cs
var idMaybe  idValue.ToMaybe();
var transOne = idMaybe.Bind(x => transformationOne(x))
 .OnNothing(() => this.Logger.LogWarning("Nothing happened one");
var transTwo = transOne.Bind(x => transformationTwo(x))
 .OnNothing(() => this.Logger.LogWarning("Nothing happened two");
var returnValue = transTwo.Bind(x => transformationThree(x))
  .OnNothing(() => this.Logger.LogWarning("Nothing happened three");
```

在上面的代码示例中，所有三个 OnNothing 将会触发，并写入三条警告日志。你可能希望这样，也可能不希望。在第一个 Nothing 之后，可能就不再那么有趣了。

我确实对这个问题有一个解决方案，但这意味着需要编写更多的代码。

创建一个从原始的 Nothing 和 Error 下降的新实例：

```cs
public class UnhandledNothing<T> : Nothing<T>
{
}

public class UnhandledError<T> : Error<T>
{
}
```

我们还需要修改 Bind 函数，以便在从 Something 路径切换到其中之一时返回这些类型。

```cs
public static Maybe<TOut> Bind<TIn, TOut>(this Maybe<TIn> @this, Func<TIn, TOut> f)
{
	try
	{
		Maybe<TOut> updatedValue = @this switch
		{
			Something<TIn> s when !EqualityComparer<TIn>.Default.Equals(s.Value, default) =>
				new Something<TOut>(f(s.Value)),
			Something<TIn> s when s.GetType().GetGenericArguments()[0].IsPrimitive => new Something<TOut>(f(s.Value)),
			Something<TIn> _ => new UnhandledNothing<TOut>(),
			Nothing<TIn> _ => new Nothing<TOut>(),
			UnhandledNothing<TIn> _ => new UnhandledNothing<TOut>(),
			Error<TIn> e => new Error<TOut>(e.ErrorMessage),
			UnhandledError<TIn> e => new UnhandledError<TOut>(e.CapturedError),
			_ => new Error<TOut>(new Exception("New Maybe state that isn't coded for!: " + @this.GetType()))
		};
		return updatedValue;
	}
	catch (Exception e)
	{
		return new UnhandledError<TOut>(e);
	}
}
```

然后最后，需要更新处理函数：

```cs
public static class MaybeLoggingExtensions
{

 public static Maybe<T> OnNothing(this Maybe<T> @this, Action a)
 {
  if(@this is UnhandledNothing<T> _)
  {
   a();
   return new Nothing<T>();
  }

  return @this;
 }
}

 public static Maybe<T> OnError(this Maybe<T> @this, Action<Exception> a)
 {
  if(@this is UnhandledError<T> e)
  {
   a(e.CapturedError);
   return new Error<T>(e.CapturedError);
  }

  return @this;
 }
```

所有这些意味着，当从 Something 切换到其他状态时，Maybe 会切换到一个状态，不仅表示发生了 Nothing 或异常，而且还表示尚未处理该状态。

一旦调用了处理函数之一，并发现了未处理的状态，那么回调就会触发日志记录，或者其他操作，并返回一个新对象，保持相同的状态类型，但这次指示它不再是未处理的状态。

这意味着在我上面示例中多次使用 `Bind` 调用并附加了 `OnNothing` 函数时，只有第一个 `OnNothing` 会被触发，其余的会被忽略。

在代码库的其他地方，你仍然可以使用模式匹配语句来检查 `Maybe` 的类型，以便在 `Maybe` 达到最终目的地时执行某些动作或其他操作。

## `Maybe` 和异步

所以，我知道你接下来要问我什么。看，我得委婉地拒绝你了。我已经结婚了。哦？我的错。异步和 Monad。是的，好的。继续...​

如何处理 Monad 内部调用到异步进程？老实说，这并不难。保留你已经编写的 `Maybe Bind` 函数，并将这些内容添加到代码库中：

```cs
public static async Task<Maybe<TOut>> BindAsync<TIn, TOut>(this Maybe<TIn> @this, Func<TIn, Task<TOut>> f)
{
	try
	{
		Maybe<TOut> updatedValue = @this switch
		{
			Something<TIn> s when EqualityComparer<TIn>.Default.Equals(s.Value, default) =>
				new Something<TOut>(await f(s.Value)),
			Something<TIn> _ => new Nothing<TOut>(),
			Nothing<TIn> _ => new Nothing<TOut>(),
			Error<TIn> e => new Error<TOut>(e.ErrorMessage),
			_ => new Error<TOut>(new Exception("New Maybe state that isn't coded for!: " + @this.GetType()))
		};
		return updatedValue;
	}
	catch (Exception e)
	{
		return new Error<TOut>(e);
	}
}
```

我在这里做的所有事情就是在我们传递的值周围再包装一层。

第一层是 `Maybe` - 表示我们尝试的操作可能没有成功

第二层是 `Task` - 表示首先需要执行一个 `async` 操作，然后才能到达 `Maybe`。

在你更广泛的代码库中使用这个最好一次一行地进行，这样你可以避免在同一个 `Bind` 调用链中混合使用异步和非异步版本，否则你可能会得到一个 `Task<T>` 作为一种类型传递，而不是实际的类型 `T`。此外，这意味着你可以将每个异步调用分离出来，并使用 `await` 语句获取真实的值以传递给下一个 `Bind` 操作。

## 嵌套的 `Maybe`

到目前为止，我展示的 `Maybe` 存在一些问题。这是我真正意识到的，一旦我将许多接口更改为使用 `Maybe<T>` 作为涉及外部交互的返回类型后。

下面是一个需要考虑的场景。我创建了几种不同类型的数据加载器。它们可以是数据库、Web API 或其他。并不重要：

```cs
public interface DataLoaderOne
{
	Maybe<string> GetStringOne();
}

public interface DataLoaderTwo
{
	Maybe<string> GetStringTwo(string stringOne);
}

public interface DataLoaderThree
{
	Maybe<string> GetStringThree(string stringTwo);
}
```

在代码的其他部分，我想通过 `Bind` 调用依次调用这些接口。

注意 `Maybe<string>` 返回类型的重点在于，我可以通过 `Bind` 函数引用它们，如果任何 `dataLoader` 调用失败，后续步骤 *将不会* 执行，并且我会得到一个 `Nothing<string>` 或 `Error<string>` 以供检查。

像这样：

```cs
	var finalString = dataLoaderOne.GetStringOne()
		.Bind(x => dataLoaderTwo.GetStringTwo(x))
		.Bind(x => dataLoaderThree.GetStringThree(x));
```

不过我发现这段代码不会编译。你觉得为什么会这样？

这里有三个函数调用在工作，并且所有三个返回类型都是 `Maybe<string>`。一次看一行，看看会发生什么：

1.  `GetStringOne` 首先返回一个 `Maybe<string>`。目前为止，一切顺利。

1.  然后 `Bind` 调用附加到 `Maybe<string>` 并解压缩到一个字符串，传递给 `GetStringTwo`，其返回类型被弹出到一个新的 `Maybe` 中以便安全保持。

1.  下一个绑定调用展开了上一个绑定的返回类型，使其仅为`GetStringTwo`的返回类型 - 但`GetStringTwo`并未返回`string`，而是返回了`Maybe<string>`。因此，在第二个绑定调用中，`x`实际上等于`Maybe<string>`，无法传递给`GetStringThree`！

*我可以*通过直接访问存储在`x`中的 Maybe 值来解决这个问题，但首先我需要将其转换为`Something`。但如果不是`Something`呢？如果在`GetStringOne`与数据库交互时发生错误呢？如果找不到任何字符串呢？

我基本上需要一种方法来解包一个嵌套的 Maybe，但*仅*在它返回包含真实值的 Something 时，对其进行处理，否则我们需要匹配其不成功路径（Nothing 或 Error）。

我的做法是创建另一个 Bind 函数，与我们已经创建的其他两个并列，但这个函数专门处理嵌套的 Maybes 问题。

我会这样做：

```cs
public static Maybe<TOut> Bind<TIn, TOut>(
	this Maybe<Maybe<TIn>> @this, Func<TIn, TOut> f)
{
	try
	{
		var returnValue = @this switch
		{
			Something<Maybe<TIn>> s => s.Value.Bind(f),
			Error<Maybe<TIn>> e => new Error<TOut>(e.ErrorMessage),
			Nothing<Maybe<TIn>> => new Nothing<TOut>(),
			_ => new Error<TOut>(new Exception("New Maybe state that isn't coded for!: " + @this.GetType()))

		};
		return returnValue;

	}
	catch (Exception e)
	{
		return new Error<TOut>(e);
	}
}
```

我们在这里做的是获取嵌套绑定（`Maybe<Maybe<string>>`）并对其调用 Bind，在回调函数中展开第一层，使我们仅剩下`Maybe<string>`，从而可以在 Maybe 上执行与之前 Bind 函数中相同的逻辑。

这也需要针对`async`版本完成：

```cs
public static async Task<Maybe<TOut>> BindAsync<TIn, TOut>(this Maybe<Maybe<TIn>> @this, Func<TIn, Task<TOut>> f)
{
 try
 {
 	var returnValue = await @this.Bind(async x =>
	{
		  var updatedValue = @this switch
		  {
		   Something<TIn> s when
		    EqualityComparer<TIn>.Default.Equals(s.Value, default(TIn))  =>
		     new Something<TOut>(await f(s.Value)),
		   Something<TIn> _ => new Nothing<TOut>(),
		   Nothing<TIn> _ => new Nothing<TOut>(),
		   Error<TIn> e => new Error<TOut>(e.CapturedError)
		  }
		  return updatedValue;
	});
  return returnValue;
 }
 catch(Exception e)
 {
  return new Error<TOut>(e);
 }
}
```

如果你希望以另一种方式思考这个过程。想想 LINQ 中的`SelectMany`函数。如果你向它提供一个数组的数组 - 即多维数组，则你会得到一个单维的扁平数组。现在，处理嵌套的`Maybe`对象允许我们在 Monads 中执行相同的操作。这实际上是 Monads 的一个“法则” - 任何自称为 Monad 的东西都应遵循的特性。

事实上，这使我顺利过渡到下一个主题。什么是法则，它们的用途是什么，以及我们如何确保我们的 C# Monad 也遵循它们…​

# 法则

严格来说，要被认为是一个真正的 Monad，必须遵循一组规则（称为*法则*）。我将简要讨论每个法则，这样你就能自己判断你是否在看一个真正的 Monad。

## 左单位元法则

这表明，一个 Monad，如果给定一个函数作为其 Bind 方法的参数，将返回与直接运行函数相同且没有副作用的等效结果。

这里有一些 C#代码来演示：

```cs
Func<int, int> MultiplyByTwo = x => x * 2;
var input = 100;
var runFunctionOutput = MultiplyByTwo(input);

var monadOutput = new Something<int>(input).Bind(MultiplyByTwo);

// runFunctionOutput and monadOutput.Value should both
// be identical - 200 - to conform to the Left Identity Law.
```

## 右单位元法则

在我解释这个之前，我需要往回退几步…​

首先，我需要解释一下 Functor。这些是将一个事物或一组事物从一种形式转换为另一种形式的函数。`Map`、`Bind`和`Select`都是 Functor 的示例。

最简单的函子是 Identity 函子。Identity 是一个函数，给定一个输入，返回未更改且没有副作用的同一输入。当你组合函数时，它可能会有用。

我在此处感兴趣的唯一原因是，它是第二个 Monad 定律的基础 - 右单位元法则。这意味着当 Monad 在其 Bind 函数中给定一个 Identity Functor 时，将无副作用地返回原始值。

我可以像这样测试我上一章中创建的 Maybe：

```cs
Func<int,int> identityInt = (int x) => x;
var input = 200;
var result = new Something<int>(input).Bind(identityInt);
// result = 200
```

这只意味着 Maybe 接受一个不会产生错误或`Null`的函数，执行它，然后准确地返回其输出，而不是其他任何内容。

这两个定律的基本要点很简单，即 Monad 不能以任何方式干预传入或传出的数据，也不能干预传递给 Bind 方法作为参数的函数的执行。Monad 只是一个管道，函数和数据通过它流动。

## 结合律

前两个定律应该相对琐碎，而我的 Maybe 实现同时满足这两个定律。最后一个定律，即结合律，更难以解释。

它基本上意味着嵌套的 Monad 如何并不重要，最终你总是得到一个包含值的单一 Monad。

这里是一个简单的 C# 示例：

```cs
var input = 100;
var op1 = (int x) => x * 2;
var op2 = (int x) => x + 100;

var versionOne = new Something<int>(input)
 .Bind(op1)
 .Bind(op2);

 // versionOne.Value = 100 * 2 + 100 = 300

 var versionTwo = new Something<int>(input)
  .Bind(x => new Something<int>(x).Bind(op1)).Bind(op2);

  // If we don't implement something to fulfill the
  // associativity law, we'll end up with a type of
  // Something<Something<int>>, where we want this to be
  // the exact same type and value as versionOne
```

回顾前几节关于如何处理嵌套 Maybe 的描述“嵌套 Maybe”，你会看到这是如何实现的。

幸运的话，现在我们已经看过了三个 Monad 定律，我已经证明了我的 Maybe 是一个真正的、毫不吝啬的、诚实的 Monad。

在接下来的部分，我将向您展示另一个 Monad，您可能会用它来消除需要在各处共享的变量。

# Reader

让我们想象一下，我们正在组织某种报告。它从 SQL Server 数据库中进行一系列数据拉取。

首先，我们需要获取特定用户的记录。

接下来，使用该记录，我们从我们完全虚构的书店中获取他们最近的订单⁹。

最后，我们将最近的订单记录转换为订单中的物品列表，并在报告中返回其中的一些细节。

我们想利用一种 Monad 风格的 Bind 操作，那么如何确保每个步骤中的数据与数据库连接对象一起传递？这没问题，我们可以组合一个 Tuple，并简单地传递两个对象。

```cs
public string MakeOrderReport(string userName) =>
 (
  Conn: this.connFactory.MakeDbConnection(),
  userid
 )
  .Bind(x => (
   x.Conn,
   Customer: this.customerRepo.GetCustomer(x.Conn, x.userName)
  )
  .Bind(x => (
   x.Conn,
   Order: this.orderRepo.GetCustomerOrders(x.Conn, x.Customer.Id)
  ),
  .Bind(x => this.Order.Items.First())
  .Bind(x => string.Join("\r\n", x));
```

这是一个可行的解决方案，但有点丑陋。存在一些重复的步骤，仅用于在 Bind 操作之间持久化连接对象。这影响了函数的可读性。

从纯粹性方面来说，这也不纯粹。这个函数必须创建数据库连接，这是一种副作用。

还有另一种功能结构，我们可以用来解决所有这些问题 - Reader Monad。这是函数式编程对依赖注入的回答，但在函数级别而不是类级别。

在上述函数的情况下，我们希望注入 IDbConnection，以便可以在其他地方实例化它，使 MakeOrderReport 保持纯净 - 即没有任何副作用。

这里是一个非常简单的读取器的使用示例：

```cs
var reader = new Reader<int, string>(e => e.ToString());
var result = reader.Run(100);
```

我们在这里定义的是一个读取器，它接受一个存储但不执行的函数。该函数以其参数作为“环境”变量类型 - 这是我们将来将要注入的当前未知值，并根据该参数返回一个值，在我们的案例中是一个整数。

“int → string” 函数存储在第一行的读取器中，然后在第二行我们调用“Run”函数，该函数提供了缺失的环境变量值，这里是 100。由于环境最终已提供，因此读取器因此可以使用它来返回一个实际的值。

由于这是一个单子，这也意味着我们应该有 Bind 函数来提供一个流程。这是它们将如何被使用的方式：

```cs
var reader = new Reader<int, int>(e => e * 100)
 .Bind(x => x / 50)
 .Bind(x => x.ToString());

var result = reader.Run(100);
```

注意，“reader” 变量的类型是 `Reader<int, string>`。这是因为每个 Bind 调用都在前一个函数周围放置了一个包装器，该函数具有相同的替代返回类型，但不同的参数。

在带有参数 `e => e * 100` 的第一行中，该函数将在稍后执行，然后运行等等...​

这是读取器的一个更现实的用法：

```cs
public Customre GetCustomerData(string userName, IDbConnection db) =>
 new Reader(this.customerRepo.GetCustomer(userName, x))
 .Run(db);
```

或者，您实际上可以简单地返回读取器，并允许外部世界继续使用 Bind 调用来进一步修改它，然后再运行函数将其转换为正确的值。

```cs
public Reader<IdbConnection, User> GetCustomerData(string userName) =>
  new Reader(this.customerRepo.GetCustomer(userName, x));
```

这种方式，同一个函数可以被多次调用，使用读取器的 Bind 函数将其转换为我想要的实际类型。

例如，如果我想获取客户的订单数据：

```cs
var dbConn = this.DbConnectionFactory.GetConnection();

var orders = this._customerRepo.GetCustomerData("simon.painter")
 .Bind(X => x.OrderData.ToArray())
 .Run(dbConn);
```

想象一下，您正在创建一个只能通过插入正确类型的变量来打开的盒子。

使用读取器还意味着可以轻松地将模拟的 IDbConnection 注入到这些函数中，并基于它们编写单元测试。

根据您希望如何构建代码的方式，甚至可以考虑在接口上公开读取器。它不必像传递进来的 DbConnection 那样成为一个依赖项，它可以是数据库表的 Id 值，或者任何你喜欢的东西。也许是这样：

```cs
public interface IDataStore
{
 Reader<int,Customer> GetCustomerData();
 Reader<Guid,Product> GetProductData();
 Reader<int,IEnumerable<Order>> GetCustomerOrders();
}
```

有各种各样的方法可以使用这个，这完全取决于适合你的是什么，以及你试图做什么。

在下一节中，我将展示这个想法的变种 - 状态单子。

# 状态

原则上，状态单子与读取器非常相似。定义了一个容器，它需要某种形式的状态对象将自身转换为一个正确的最终数据。绑定可用于提供额外的数据转换，但直到提供状态之前什么都不会发生。

它与众不同的两个方面是：

1.  不再是“环境”类型，而是称为“状态”类型。

1.  在 Bind 操作之间传递的不是一个而是两个项。

在 Reader 中，原始的 Environment 类型仅在绑定链的开头可见。使用 State 单子，它一直持续到最后。状态类型及其当前值设置为一个元组，从一个步骤传递到下一个。值和状态都可以替换为新值。值可以更改类型，但状态在整个过程中是一个单一类型，如果需要，可以更新其值。

你还可以随意提取或替换状态单子中的状态对象，使用函数随时。

我的实现并不严格遵循你在 Haskell 等语言中看到的方式，但我认为这种类型的实现在 C# 中很麻烦，我不确定这样做有什么意义。我在这里展示的版本在日常 C# 编码中可能会有所用处。

```cs
public class State<TS, TV>
{
	public TS CurrentState { get; init; }
	public TV CurrentValue { get; init; }
	public State(TS s, TV v)
	{
		CurrentValue = v;
		CurrentState = s;
	}
}
```

State 单子没有多个状态，因此不需要一个基础抽象类。它只是一个简单的类，有两个属性 - 一个值和一个状态（即我们将通过每个实例传递的东西）。

逻辑必须实现为扩展方法：

```cs
public static class StateMonadExtensions
{

public static State<TS, TV> ToState<TS, TV>(this TS @this, TV value) =>
	new(@this, value);

public static State<TS, TV> Update<TS, TV>(
	 this State<TS, TV> @this,
	 Func<TS, TS> f
	) => new(f(@this.CurrentState), @this.CurrentValue);
}
```

通常情况下，实现代码不多，但有很多有趣的效果。

这是我使用它的方式：

```cs
public IEnumerable<Order> MakeOrderReport(string userName) =>
 this.connFactory.MakeDbConnection().ToState(userName)
 .Bind((s, x) => this.customerRepo.GetCustomer(s, x))
 .Bind((s, x) => this.orderRepo.GetCustomreOrders(s, x.Id))
```

这里的想法是，状态对象作为 *s* 传递到链中，最后一个 Bind 的结果作为 *x* 传递进来。基于这两个值，你可以确定下一个值应该是什么。

这仅留下了更新当前状态的功能。我会用这个扩展方法来实现：

```cs
public static State<TS, TV>Update<TS,TV>(
 this State<TS,TV> @this,
 Func<TS, TS> f
) => new(@this.CurrentState, f(@this.CurrentState));
```

为了说明的简单例子，这里是一个简单的例子：

```cs
var result = 10.ToState(10)
	.Bind((s, x) => s * x)
	.Bind((s, x) => x - s) // s=10, x = 90
	.Update(s => s - 5) // s=5, x = 90
	.Bind((s, x) => x / 5); // s=5, x = 18
```

使用这种方法，你可以有箭头函数，带有几个状态位，这些状态位将从一个 Bind 操作流到下一个，需要时甚至可以更新。它可以防止你被迫将整洁的箭头函数转换为带大括号的完整函数，或者通过每个 Bind 传递一个大而笨重的只读数据元组。

这个实现已经去掉了你在 Haskell 中会找到的形式，即只有在定义完整的绑定链时才传递初始状态值，但我认为在 C# 上下文中，这个版本更有用，而且编码起来更容易！

## 也许一个状态？

你可能会注意到在上一个代码示例中，并没有使用 Bind 函数的功能，比如 Maybe，来捕获错误条件和返回的空结果在其中一个或多个可能状态中。是否可以将 Maybe 和 Reader 合并为一个单子，既持久化一个状态对象 **又** 处理错误？

是的，有几种方法可以实现它，具体取决于你打算如何使用它。我将展示我首选的解决方案。首先，我会调整 State 类，以便它不再存储一个值，而是存储一个 Maybe 包含的值。

```cs
public class State<TS, TV>
{
	public TS CurrentState { get; init; }
	public Maybe<TV> CurrentValue { get; init; }
	public State(TS s, TV v)
	{
		CurrentValue = new Something<TV>(v);
		CurrentState = s;
	}
}
}
```

然后我会调整 Bind 函数，以考虑 Maybe，但不改变函数的签名：

```cs
public static State<TS, TNew> Bind<TS, TOld, TNew>(
	this State<TS, TOld> @this, Func<TS, TOld, TNew> f) =>
	new(@this.CurrentState, @this.CurrentValue.Bind(x => f(@this.CurrentState, x)));
```

使用方法几乎完全相同，只是现在 Value 的类型是 Maybe<T>，而不只是 T。不过，这实际上只影响容器函数的返回值：

```cs
public Maybe<IEnumerable<order>> MakeOrderReport(string userName) =>
 this.connFactory.MakeDbConnection().ToState(userName)
 .Bind((s, x) => this.customerRepo.GetCustomer(s, x)
 .Bind((s, x) => this.orderRepo.GetCustomerOrders(s, x.Id))
```

是否希望以这种方式合并 Maybe 和 State Monad 的概念，或者更愿意将它们分开，完全取决于您。

如果您遵循这种方法，您只需要确保在某个时候使用 Switch 表达式将 Maybe 转换为单一的具体值。

还有一件事也要记住 - State Monad 的 CurrentValue 对象不一定是数据，它也可以是一个 `Func` 委托，允许您在 Bind 调用之间传递一些功能性。

在接下来的部分中，我将探讨在 C# 中可能已经在使用的其他单子。

# 您已经在使用的示例

信不信由你，如果您已经使用 C# 工作了一段时间，那么您很可能已经在使用单子。让我们看看一些例子。

## Enumerable

如果 Enumerable 不是一个单子，那么它几乎是最接近的了，至少一旦我们调用 LINQ - 正如我们已经知道的那样，LINQ 是基于函数编程概念开发的。

Enumerable 的 `Select` 方法在可枚举对象中操作各个元素，但它仍然遵循左恒等法则：

```cs
var op = x => x * 2;
var input = new [] { 100 };

var enumerableResult = input.Select(op);
var directResult = new [] { op(input.First()) };
// both equal the same value - { 200 }
```

和右恒等法则：

```cs
var op = x => x;
var input = new [] { 100 };

var enumerableResult = input.Select(op);
var directResult = new [] { op(input.First()) };
// both equal the same value - { 100 }
```

那么仅剩下结合律 - 它仍然是被认为是真正的单子所必需的。Enumerable 遵循这一法则吗？当然是的。通过使用 SelectMany。

考虑一下这个：

```cs
var createEnumerable = (int x) => Enumerable.Range(0, x);
var input = new [] { 100, 200 }
var output = input.SelectMany(createEnumerable);
// output = single dimension array with 300 elements
```

在这里，我们有嵌套的可枚举对象作为单个可枚举对象输出。这就是结合律。因此，QED，等等。是的。可枚举对象是单子。

## Task

那么任务呢？它们也是单子吗？我敢打赌，如果你已经使用 C# 一段时间，它们绝对是单子，而且我可以证明。

让我们再次审视这些法则。

左恒等法则，使用任务的函数调用应该与直接调用函数调用匹配。证明这一点稍微有些棘手，因为异步方法始终返回 `Task` 或 `Task<T>` 类型 - 这在许多方面与 `Maybe<T>` 相同，如果你仔细想想的话。它是围绕可能或可能不会解析为实际数据的类型的包装器。但是，如果我们回到抽象层次，我认为我们仍然可以证明法则是遵守的：

```cs
public Func<int> op = x => x * 2;
public async Task<int> asyncOp(int x) => await Task.FromResult(op(x));

var taskResult = await asyncOp(100);
var nonTaskResult = op(100);
// The result is the same - 200
```

我并不是说这是我必须引以为豪的 C# 代码，但至少它确实证明了一个观点，即无论我是通过异步包装方法调用还是直接调用 `op`，结果都是相同的。这就是左恒等法则的确认。那么右恒等法则呢？老实说，这几乎是相同的代码：

```cs
// Notice the function is simply returning x back again unchanged this time.
public Func<int> op = x => x;
public async Task<int> asyncOp(int x) => await Task.FromResult(op(x));

var taskResult = await asyncOp(100);
var nonTaskResult = op(100);
// The result is the same as the initial input - 100
```

这就是恒等法则的解决方案。那么同样重要的结合律呢？信不信由你，我们可以用任务来演示这一点。

```cs
async Task<int> op1(int x) => await Task.FromResult(10 * x);
async Task<int> pp2() => await Task.FromResult(100);
var result = await op1(await pp2());
// result = 1,000
```

在这里，我们有一个将 `Task<int>` 作为参数传递给另一个 `Task<int>`，但是通过嵌套调用 `await`，可以将其平铺为一个简单的 `int` - 这才是实际的结果类型。

希望我赢得了我的啤酒？我的是一品脱¹¹，请。欧式风格的半升也可以。

# 其他结构

诚实地说——如果你对我的 Maybe 单子版本满意，并且不介意深入进一步，那么请随意跳到下一章节。你可以仅通过 Maybe 轻松实现你可能想要实现的大多数功能。我将描述一些存在于更广泛函数编程语言世界中的其他单子类型，你*可能*想考虑在 C# 中实现它们。

如果你打算从 C# 中挤出最后几个非函数式代码的残余，那么这些单子可能会引起你的兴趣。它们也可能会从理论的角度引起兴趣。然而，如果你想进一步探讨 Monad 概念并继续实现这些内容，完全取决于你。

现在，严格来说，我在本章和上一章中一直在构建的 Maybe 单子版本是两种不同单子的混合。

一个真正的 Maybe 单子只有两种状态——Something（或 Just）和 Nothing（或 Empty）。就是这样。用于处理错误状态的单子是 Either（也称为 Result）单子。它有两种状态——Left 和 Right。

Right 是“幸福”路径，其中每个传递给 Bind 命令的函数都有效，一切都是正确的。

Left 是“不幸”路径，表示发生了某种错误，而该错误包含在 Left 中。

左右命名惯例可能源自许多文化中的一个重要概念，即左手是邪恶的，右手是善良的。这甚至在我们的语言中也有所体现——拉丁语中“left”的字面意思就是“Sinister（邪恶）”。然而，在这些启蒙时代，我们不再驱逐左撇子¹²离开家园，或者无论他们以前做过什么。

我不会在这里详细描述这个实现，你可以通过采用我版本的 Maybe 并移除“Nothing”类基本实现它。

同样，你可以通过移除错误类（Error class）来创建一个真正的 Maybe——尽管我认为通过将两者合并成一个单一实体，你可以处理与外部资源交互时可能遇到的几乎所有情况。

我的方法是纯粹和完全正确的经典函数理论吗？不是。它在生产代码中有用吗？100% 是。

除了 Maybe 和 Either 之外，还有许多单子，如果你转向像 Haskell 这样的编程语言，你很可能会经常使用它们。以下是一些例子：

+   Identity — 一个单子，简单地返回你输入的任何值。当深入学习更纯粹函数理论的更深层时，这些单子有其用处，但在 C# 中并没有真正的应用场景。

+   State - 用于对一个值运行一系列操作。与`Bind`方法有些类似，但也有一个状态对象被传递，作为额外的对象用于计算。在 C#中，我们可以使用 LINQ 的`Aggregate`函数，或者使用 Maybe 的`Bind`函数与元组或类似的结构一起传递必要的状态对象。

+   IO - 用于允许与外部资源交互而不引入不纯的函数。在 C#中，我们可以遵循控制反转模式（即依赖注入）来解决测试等问题。

+   环境 - 在 Haskell 中被称为 Reader 单子。通常用于像日志记录这样的过程，以封装掉副作用。如果你试图确保你的语言正在强制执行函数式编程的严格规则，这是有用的，但我在 C#中看不到任何好处。

正如你从上面的列表中看到的，函数式编程世界中有许多其他的单子，但我认为大多数或全部都对我们没有实际好处。归根结底，C#是一种混合的函数式/面向对象语言。它已经扩展到支持函数式编程概念，但它永远不会成为纯函数式语言，也没有尝试将其视为这样的好处。

我强烈建议尝试使用 Maybe/Either 单子，但除此之外，我实在不会打扰，除非你对在 C#中的函数式编程的想法有兴趣，看看你能推动这个想法到什么程度¹³。不过，这不适用于你的生产环境。

在最后一节中，我将提供一个完整的示例，展示如何在应用程序中使用单子。

# 一个示例

好的，我们开始吧。让我们把所有内容整合到一个伟大而史诗般的单子功能堆中。我们在本章的示例中已经谈论过假期，所以这次我要集中讨论我们实际上如何去机场 - 这将需要一系列的查找、数据转换，所有这些通常需要错误处理和分支逻辑，如果我们要遵循更传统的面向对象方法。我希望您会同意，使用函数式编程技术和单子，看起来要优雅得多。

首先，我们需要我们的接口。我实际上不会编写每一个我们的代码所需的依赖项，所以我只是定义我们将需要的接口：

```cs
public interface IMappingSystem
{
 Maybe<Address> GetAddress(Location l);
}

public interface IRoutePlanner
{
 Task<Maybe<Route>> DetermineRoute(Address a, Address b);
}

public interface ITrafficMonitor
{
 Maybe<TrafficAdvice> GetAdvice(Route r);
}

public interface IPricingCalculator
{
 decimal PriceRoute(Route r);
}
```

做完这些，我将编写代码来消耗它们。我想象中的具体场景是这样的 - 这是不久的将来。无人驾驶汽车已经成为一种事物。以至于大多数人不再拥有个人车辆，而只需使用手机上的应用程序从自动驾驶汽车的云中直接将车辆带到他们的家中¹⁴。

这个过程大致如下：

1.  最初的输入是用户提供的起始位置和目的地。

1.  需要查找映射系统中的每个位置，并转换为正确的地址。

1.  需要从内部数据存储中获取用户的账户。

1.  路由需要通过交通服务进行检查。

1.  必须调用定价服务来确定旅程的费用。

1.  价格返回给用户。

在代码中，这个过程可能看起来像这样：

```cs
public Maybe<decimal> DeterminePrice(Location from, Location to)
{
 var addresses = this.mapping.GetAddress(from).Bind(x =>
  (From: x,
   To: this.mapping.GetAddress(to)));

 var route = await addresses.BindAsync (async x => await this.router.DetermineRoute(x.From, x,To));

 var trafficInfo = route.Bind(x => this.trafficAdvisor.GetAdvice(x));
 var hasRoadWorks = trafficInfo is Something<TrafficAdvice> s &&
  s.Value.RoadworksOnRoute;

 var price = route.Bind(x => this.pricing.PriceRoute(x));
 var finalPrice = route.Bind(x => hasRoadWorks ? x *= 1.1 : x);

 return finalPrice;
}
```

这样更容易，不是吗？在我结束本章之前，我想解释一下代码示例中发生的一些细节。

首先，这里没有任何错误处理。任何这些外部依赖可能导致错误被抛出，或者在它们各自的数据存储中找不到详细信息。Monad `Bind` 函数处理所有这些逻辑。例如，如果路由器无法确定路由（可能发生网络错误），那么此时 Maybe 将被设置为 `Error<Route>`，并且不会执行后续操作。最终的返回类型将是 `Error<decimal>`，因为 `Error` 类在每一步都会重新创建，但实际的 `Exception` 在实例之间传递。外部世界负责处理最终返回的值，直到那时为止。

如果我们按照面向对象的方法编写此代码，那么该函数很可能会长两到三倍，以包括 `Try/Catch` 块和针对每个对象的检查以确认它们是否有效。

我在需要建立一组输入的情况下使用了元组。对于地址对象的情况，这意味着如果第一个地址找不到，那么不会尝试查找第二个地址。这也意味着第二个函数所需的两个输入都在一个地方可用，然后我们可以使用另一个 `Bind` 调用来访问它们（假设地址查找返回了一个真实的值）。

最后几个步骤实际上并不涉及对外部依赖的调用，但通过继续使用 `Bind` 函数，可以假定其参数 Lambda 表达式内部有一个真实值可用，因为如果没有，Lambda 就不会被执行。

至此，我们几乎有一个完全功能的 C# 代码片段了。希望你喜欢。

# 结论

在本章中，我们探讨了一个可怕的函数式编程概念，已经众所周知，它使成年开发人员在他们廉价的鞋子上颤抖。一切顺利的话，这对你来说应该不再是一个谜。

我已经向你展示了如何：

+   大幅减少所需代码量。

+   介绍一个隐式错误处理系统。

所有都使用 Maybe Monad，以及如何自己创建一个。

在下一章中，我将简要讨论柯里化的概念。我们在下一页见。

¹ 我不知道为什么会使用那个名字。真的不知道。尽管如此，这是相当普遍的，也是一种标准。请稍作等待。

² 我会说非空值，但整数默认为 0，布尔值默认为 false。

³ 查看这里阅读文章，这绝对值得您花时间：[*https://fsharpforfunandprofit.com/rop/*](https://fsharpforfunandprofit.com/rop/)

⁴ 或活着的薛定谔的猫。实际上，*这*是安全的选择吗？我养过猫，我知道它们是什么样子！

⁵ 最好是芝士蛋糕。保罗·荷莱伍德会很失望地得知我不喜欢很多种蛋糕，但纽约风格的芝士蛋糕绝对是我喜欢的一种！

⁶ 您可以在这里阅读更多信息：[*https://automapper.org/*](https://automapper.org/) 如果在您的代码中经常需要快速简单地在类型之间进行转换，它是非常有用的工具。

⁷ 不是一种茶！也不是《龙珠》中的一个猪角色。

⁸ 除了布尔值，默认为 false，和字符值默认为*\0*。

⁹ 我是虚构的所有者，你可以是虚构的帮助客户找到他们想要的东西的人。我真是慷慨呢，不是吗？

¹⁰ 我喜欢黑色艾尔啤酒和欧式风格的拉格啤酒。他们在明尼苏达州酿造的浓烈的东西也非常棒。

¹¹ 那是 568 毫升。我知道其他国家对这个词有不同的定义。

¹² 其中包括我的兄弟。嗨，马克 - 你在一本编程书中被提到了！我想知道你是否会知道这件事？

¹³ C# 而不是.NET 的普遍性 - F# 也是.NET。

¹⁴ 对我来说这听起来都很棒，我不太喜欢开车，而且如果我是乘客，我会非常晕车。我非常欢迎我们的无人驾驶汽车统治者。
