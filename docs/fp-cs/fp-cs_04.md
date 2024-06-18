# 第四章：通过功能代码工作智能，而不是努力工作

到目前为止，我所涵盖的一切都是微软 C# 团队旨在实现的功能编程。您会在微软网站上找到这些功能的提及及其示例。然而，在本章中，我想开始对 C# 进行更富创意的探索。

我不知道你怎么看，但我喜欢偷懒，或者说我不喜欢浪费时间在冗长的样板代码上。函数式编程的许多精彩之处之一就是它的简洁性，相比命令式代码。

在本章中，我将展示如何推动功能编程的边界，超出 C# 的开箱即用功能，以及如何在旧版本中实现一些较新版本的 C#，希望能让您更快地完成日常工作。

本章将探讨以下三个广泛的类别：

枚举中的 Funcs

`Func` 委托似乎并没有被广泛使用，但它们是 C# 的非常强大的特性。我将展示一些使用它们来扩展 C# 能力的方法。在这种情况下，通过将它们添加到 Enumerable 并使用 Linq 表达式对它们进行操作。

Funcs 作为过滤器

您还可以将 Funcs 用作过滤器 - 这是一种位于您和真正想要达到的值之间的东西。您可以使用这些原则编写一些精彩的代码。

自定义枚举

我之前讨论过 IEnumerable 及其有多酷的功能，但你知道你可以打开它们并实现自定义行为吗？我会向你展示如何做。

# 是时候变得“Func-y”

我在介绍中已经谈到了 `Func` 委托类型，但简要回顾一下 - 它们是存储为变量的函数。您定义它们接受什么参数，返回什么，并像任何其他函数一样调用它们。这里有一个快速的例子：

```cs
private readonly Func<Person, DateTime, string> SayHello =
    (Person p, DateTime today) => today + " : " + "Hello " + p.Name;
```

在两个尖括号之间的列表中，最后一个泛型类型是返回值，之前的所有类型都是参数。我上面的例子接受两个字符串参数并返回一个字符串。

我们接下来将经常见到许多**Func 委托**，所以请确保在继续阅读之前对它们感到舒适。

## 枚举中的 Funcs

我见过很多将 Funcs 作为函数参数的例子，但我不确定有多少开发人员意识到你可以将它们放入 Enumerable，并创建一些有趣的行为。

首先，显而易见的是 - 将它们放入数组中以对同一数据进行多次操作：

```cs
private IEnumerable<Func<Employee, string>> descriptors = new []
{
    x => "First Name = " + x.firstName,
    x => "Last Name = " + x.lastName,
    x => "MiddleNames = string.Join(" ", x.MiddleNames)
}

public string DescribeEmployee(Employee emp) =>
   string.Join(Environment.NewLine, descriptors.Select(x => x(emp)));
```

使用这种方法，我们可以从单一的数据源（这里是一个 Employee 对象）生成多个相同类型的记录，在我的案例中，我使用内置的 .NET 方法 `string.Join` 聚合生成一个统一的字符串呈现给最终用户。

这种方法相对于简单的 StringBuilder 有一些优势。

首先，数组可以动态组装。每个属性和其呈现方式可能有多个规则，这些规则可以根据某些自定义逻辑从一组本地变量中选择。

其次，这是一个可枚举对象，因此通过这种方式定义它，我们利用了可枚举对象称为*惰性评估*的功能。关于可枚举对象的一点是，它们不是数组，甚至不是数据。它们只是指向某个东西的指针，这个东西将告诉你如何提取数据。很可能 - 实际上通常是这样的情况 - 可枚举对象背后的源头是一个简单的数组，但不一定如此。可枚举对象需要在每次通过 `ForEach` 循环访问下一个项目时执行一个函数。可枚举对象的开发目的是使其在最后可能的时刻才转换为实际数据 - 通常是在开始 `ForEach` 循环迭代时。大多数情况下，如果内存中有一个数组供可枚举对象使用，这并不重要，但如果有一个昂贵的函数或者用于驱动它的外部系统查找，则惰性加载可以非常有用，以防止不必要的工作。

可枚举对象的元素将逐一评估，并且只有当它们被某个执行枚举的过程使用时才会被使用。例如，如果我们使用 LINQ 的 `Any` 函数来评估可枚举对象中的每个元素，它将在找到满足指定条件的第一个元素时停止枚举，这意味着剩余的元素将保持未评估状态。

最后，从维护的角度来看，这种技术更容易维护。向最终结果添加新行就像向数组添加新元素一样简单。它还作为对未来程序员的一种限制，使他们更难在不适当的地方加入过多复杂的逻辑。

## 一个超级简单的验证器

让我们设想一个快速验证函数。它们通常看起来像这样：

```cs
public bool IsPasswordValid(string password)
{
   if(password.Length <= 6)
      return false;

   if(password.Length > 20)
       return false;

   if(!password.Any(x => Char.IsLower(x)))
      return false;

   if(!password.Any(x => Char.IsUpper(x)))
      return false;

   if(!password.Any(x => Char.IsSymbol(x)))
      return false;

   if(password.Contains("Justin", StringComparison.OrdinalIgnoreCase)
      && password.Contains("Bieber", StringComparison.OrdinalIgnoreCase))
       return false;

   return true;
}
```

嗯，首先，实际上这是一堆代码，用来实现一个相当简单的规则。在这里，命令式代码迫使我们写下一堆重复的样板代码。除此之外，如果我们想要添加另一条规则，那可能会多写大约 4 行代码，但其实只有 1 行对我们特别有意义。

要是能把它压缩成几行简单的代码就好了……

嗯……既然你这么客气地问了，那就给你看看：

```cs
public bool IsPasswordValid(string password) =>
    new Func<string, bool>[]
    {
        x => x.Length > 6,
        x => x.Length <= 20,
        x => x.Any(y => Char.IsLower(y)),
        x => x.Any(y => Char.IsUpper(y)),
        x => x.Any(y => Char.IsSymbol(y)),
        x => !x.Contains("Justin", StringComparison.OrdinalIgnoreCase)
            && !x.Contains("Bieber", StringComparison.OrdinalIgnoreCase)
    }.All(f => f(password));
```

现在不那么长了吧？这里我做了什么？我把所有规则放入了一个将字符串转换为布尔值的 Func 数组中 - 即检查单个验证规则。我使用了一个 Linq 语句 - `.All()`。这个函数的目的是评估我给它的任何 Lambda 表达式对其附加的数组的所有元素进行检查。如果其中一个返回 false，那么进程将提前终止，并且从`All`返回 false（如前所述，后续值不会被访问，因此惰性评估通过不评估它们来节省时间）。如果所有项都返回 true，则`All`也返回 true。

我们有效地重新创建了第一个代码示例，但我们被迫编写的样板代码 - If 语句和早期返回 - 现在在结构中是隐含的。

这也有一个优势，那就是作为代码结构非常容易维护。如果你愿意，甚至可以将其泛化为一个扩展方法。我经常这样做。类似于这样：

```cs
public static bool IsValid<T>(this T @this, params Func<T,bool>[] rules) =>
    rules.All(x => x(@this));
```

这进一步减少了密码验证器的大小，并为您提供了一个方便的通用结构可供其他用途使用：

```cs
public bool IsPasswordValid(string password) =>
    password.IsValid(
        x => x.Length > 6,
        x => x.Length <= 20,
        x => x.Any(y => Char.IsLower(y)),
        x => x.Any(y => Char.IsUpper(y)),
        x => x.Any(y => Char.IsSymbol(y)),
        x => !x.Contains("Justin", StringComparison.OrdinalIgnoreCase)
            && !x.Contains("Bieber", StringComparison.OrdinalIgnoreCase)
    )
```

此时此刻，我希望你重新考虑是否再写像第一个验证代码示例那样长而臃肿的东西了。

我认为`IsValid`检查更易读和维护，但如果你想要一个更符合原始代码示例的代码片段，那么可以创建一个新的扩展方法，使用`Any`代替`All`：

```cs
public static bool IsInvalid<T>(this T @this, params Func<string,bool>[] rules) =>
    rules.Any(x => @this);
```

这意味着每个数组元素的布尔逻辑可以被反转，因为它们最初是这样的：

```cs
public bool IsPasswordValid(string password) =>
    !password.IsInvalid(
        x => x.Length <= 6,
        x => x.Length > 20,
        x => !x.Any(y => Char.IsLower(y)),
        x => !x.Any(y => Char.IsUpper(y)),
        x => !x.Any(y => Char.IsSymbol(y)),
        x => x.Contains("Justin", StringComparison.OrdinalIgnoreCase)
            && x.Contains("Bieber", StringComparison.OrdinalIgnoreCase)
    )
```

如果您希望维护`IsValid`和`IsInvalid`两个函数，因为它们在代码库中各有用处，那么通过简单地引用其中一个来节省编码工作并避免未来的潜在维护任务可能是值得的：

```cs
public static bool IsValid<T>(this T @this, params Func<T,bool>[] rules) =>
 rules.All(x => x(@this));

public static bool IsInvalid<T>(this T @this, params Func<T,bool>[] rules) =>
 !@this.IsValid(rules);
```

聪明地使用它，我的年轻函数式学徒。

## 旧版本 C#的模式匹配：

模式匹配是近年来 C#中最好的功能之一，与记录类型一起，但除了最新的.NET 版本外，其他版本都不支持 - 更多有关 C# 7 及以上本机模式匹配的详情，请参见第三章。

有没有一种方法可以允许模式匹配发生，但不需要升级到较新版本的 C#？

当然有。它远不及 C# 8 中的本机语法那么优雅，但它提供了一些相同的好处。

在这个例子中，我将根据英国所得税规则的大大简化版本计算某人应缴纳的税款。请注意，这比真实情况简单得多。我不想陷入税务复杂性的泥沼。

我将要应用的规则如下：

+   年收入 ≤ £12,570，则不扣税。

+   年收入在£12,571 到£50,270 之间，则缴纳 20%的税款。

+   年收入在£50,271 到£150,000 之间，则缴纳 40%的税款。

+   年收入超过£150,000，则缴纳 45%的税款。

如果你想手写（即非功能性地）编写这段代码，看起来会像这样：

```cs
decimal ApplyTax(decimal income)
{
 if (income <= 12570)
  return income;
 else if (income <=50270)
	 return income * 0.8M;
 else if (income <= 150000)
	 return income * 0.6M;
 else
	 return income * 0.55M;
}
```

现在，在 C# 8 及以后的版本中，switch 表达式可以压缩为几行代码。只要你至少运行 C# 7（即.NET Framework 4.7），这就是我可以创建的模式匹配风格：

```cs
var inputValue = 25000M;
var updatedValue = inputValue.Match(
 (x => x <= 12570, x => x),
 (x => x <= 50270, x => x * 0.8M),
 (x => x <= 150000, x => x * 0.6M)
).DefaultMatch(x => x * 0.55M);
```

我传递了一个包含 2 个 lambda 表达式的元组数组。第一个确定输入是否与当前模式匹配，第二个是匹配时发生的值转换。最后检查是否应用默认模式 - 即因为没有其他模式匹配。

尽管长度只有原始代码样本的一小部分，但这包含了所有相同的功能。这里左侧元组的匹配模式很简单，但它们可以包含像您想要的那样复杂的表达式，甚至可以是调用包含详细匹配条件的整个函数。

那么，我是如何让这个工作的呢？这是一个非常简单的版本，提供了大部分所需的功能：

```cs
public static class ExtensionMethods
{
	public static TOutput Match<TInput, TOutput>(
	                             this TInput @this,
	                             params (Func<TInput, bool> IsMatch,
	                             Func<TInput, TOutput> Transform)[] matches)
	{
		var match = matches.FirstOrDefault(x => x.IsMatch(@this));
		var returnValue = match.Transform(@this);
		return returnValue;
	}
}
```

我使用 Linq 方法`FirstOrDefault`首先遍历左侧函数，以找到返回 true 的函数（即具有正确条件的函数），然后调用右侧的转换 Func 来获取我的修改后的值。

这很好，除非*没有*任何模式匹配，我们可能会遇到一些问题。很可能会出现空引用异常。

要覆盖这一点，我们需要强制提供默认匹配的需要（简单的`else`语句的等效或 switch 表达式中的 _ 模式匹配）。

我的答案是让`Match`函数返回一个占位符对象，该对象保存从匹配表达式转换得到的值，或执行默认的模式 lambda 表达式。改进后的版本如下：

```cs
public static MatchValueOrDefault<TInput, TOutput> Match<TInput, TOutput>(
  this TInput @this,
  params (Func<TInput, bool>,
  Func<TInput, TOutput>)[] predicates)
{
	var match = predicates.FirstOrDefault(x => x.Item1(@this));
	var returnValue = match?.Item2(@this);
	return new MatchValueOrDefault<TInput, TOutput>(returnValue, @this);
}

public class MatchValueOrDefault<TInput, TOutput>
{
 private readonly TOutput value;
 private readonly TInput originalValue;

 public MatchValueOrDefault(TOutput value, TInput originalValue)
 {
 	this.value = value;
 	this.originalValue = originalValue;
 }

public TOutput DefaultMatch(Func<TInput, TOutput> defaultMatch)
{
 if (EqualityComparer<TOutput>.Default.Equals(default, this.value))
 {
 	return defaultMatch(this.originalValue);
 }
 else
  {
  	return this.value;
  }
}
```

与最新版本的 C#相比，此方法受到严重限制。它没有对象类型匹配，并且语法不那么优雅，但仍然可用，并且可以节省大量样板代码，同时也鼓励良好的代码标准。

较旧版本的 C#，不包括元组的版本，可以考虑使用`KeyValuePair<T,T>`，尽管语法远非理想。

你不相信？好的，我们来试试。别说我没警告过你……

扩展方法本身几乎一样，并且只需少量修改即可使用`KeyValuePair`代替元组：

```cs
public static MatchValueOrDefault<TInput, TOutput> Match<TInput, TOutput>(
  this TInput @this,
  params KeyValuePair<Func<TInput, bool>, Func<TInput, TOutput>>[] predicates)
{
    var match = predicates.FirstOrDefault(x => x.Key(@this));
    var returnValue = match.Value(@this);
    return new MatchValueOrDefault<TInput, TOutput>(returnValue, @this);
}
```

这里有个丑陋的地方。创建`KeyValuePair`对象的语法非常糟糕：

```cs
var inputValue = 25000M;
var updatedValue = inputValue.Match(
	new KeyValuePair<Func<decimal, bool>, Func<decimal, decimal>>(
		x => x <= 12570, x => x),
	new KeyValuePair<Func<decimal, bool>, Func<decimal, decimal>>(
		x => x <= 50270, x => x * 0.8M),
	new KeyValuePair<Func<decimal, bool>, Func<decimal, decimal>>(
		x => x <= 150000, x => x * 0.6M)
).DefaultMatch(x => x * 0.55M);
```

因此，在 C# 4 中*仍然可以*使用某种形式的模式匹配，但我不确定你这样做能获得多少好处。这可能由你来决定。至少我已经向你展示了路径。

# 函数式过滤

函数不仅可以用于将一种形式的数据转换为另一种形式，还可以用作过滤器，额外的层，它们位于开发者和信息或功能的原始来源之间。

本节将介绍如何使用这种方法的几个示例，使您的日常 C#编码看起来更简单，同时更少出错。

## 使字典更加有用

在 C#中，我绝对喜欢的一件事情是字典。如果适当使用，它们可以通过几个简单而优雅的类似数组的查找来减少大量丑陋、样板化的代码。一旦创建，它们在查找数据时也非常高效。

然而，它们存在一个问题，这使得通常需要添加大量样板代码，这些代码无效化了它们使用的原因。考虑以下代码示例：

```cs
var doctorLookup = new []
{
	( 1, "William Hartnell" ),
	( 2, "Patrick Troughton" ),
	( 3, "Jon Pertwee" ),
	( 4, "Tom Baker" )
}.ToDictionary(x => x.Item1, x => x.Item2);

var fifthDoctorInfo = $"The 5th Doctor was played by {doctorLookup[5]}";
```

这段代码怎么了？它触犯了我发现的字典的一个令人费解的代码特性，如果尝试查找一个不存在的条目¹，它将触发一个必须处理的异常！

处理这种情况的唯一安全方法是使用 C#中提供的几种方法之一，在编译字符串之前检查可用键，就像这样：

```cs
var doctorLookup = new []
{
	( 1, "William Hartnell" ),
	( 2, "Patrick Troughton" ),
	( 3, "Jon Pertwee" ),
	( 4, "Tom Baker" )
}.ToDictionary(x => x.Item1, x => x.Item2);

var fifthDoctorActor = doctorLookup.ContainsKey(5) ? doctorLookup[5] : "An Unknown Actor";

var fifthDoctorInfo = $"The 5th Doctor was played by {fifthDoctorActor}";
```

或者，使用稍新版本的 C#，还可以使用`TryGetValue`函数来简化这段代码：

```cs
 var fifthDoctorActor = doctorLookup.TryGetValue(5, out string value) ? value : "An Unknown Actor";
```

那么，我们能否使用函数式编程技术来减少我们的样板代码，并为我们提供字典的所有有用功能，但又不会出现糟糕的膨胀倾向？你可以打赌！

首先，我需要一个快速的扩展方法：

```cs
public static class ExtensionMethods
{
	public static Func<TKey, TValue> ToLookup<TKey,TValue>(
	  this IDictionary<TKey,TValue> @this)
	{
		return x => @this.TryGetValue(x, out TValue? value) ? value : default;
	}

	public static Func<TKey, TValue> ToLookup<TKey,TValue>(
	  this IDictionary<TKey,TValue> @this,
	  TValue defaultVal)
	{
		return x => @this.ContainsKey(x) ? @this[x] : defaultVal;
	}
}
```

我稍后会进一步解释，但首先，这是我如何使用我的扩展方法：

```cs
var doctorLookup = new []
{
	( 1, "William Hartnell" ),
	( 2, "Patrick Troughton" ),
	( 3, "Jon Pertwee" ),
	( 4, "Tom Baker" )
}.ToDictionary(x => x.Item1, x => x.Item2)
	.ToLookup("An Unknown Actor");

var fifthDoctorInfo = $"The 5th Doctor was played by {doctorLookup(5)}";
// output = "The 5th Doctor was played by An Unknown Actor"
```

注意区别了吗？

仔细观察，现在我使用的是圆括号函数，而不是方括号访问字典/数组的值。这是因为它实际上不再是一个字典！它是一个函数。

如果你看我的扩展方法，它们返回函数，但它们是保持原始 Dictionary 对象在其存在期间有效的函数。基本上，它们就像是位于 Dictionary 和代码库其余部分之间的过滤器层。这些函数决定了是否安全使用 Dictionary。

这意味着我们可以使用一个字典，但是当找不到键时不会再抛出异常，我们可以返回类型的默认值（通常是 Null），或者提供我们自己的默认值。简单吧。

这种方法唯一的缺点是它不再是一个字典。这意味着你不能进一步修改它，或者在它上面执行任何 LINQ 操作。然而，如果你确信你不需要这样做，那么这是你可以使用的东西。

# 解析值

另一个导致冗长、样板代码的常见原因是将字符串解析为其他形式。例如，对于在假设我们在.NET Framework 中工作且不可用 appsettings.JSON 和`IOption<T>`功能的设置对象中进行解析：

```cs
public Settings GetSettings()
{
	var settings = new Settings();

	var retriesString = ConfigurationManager.AppSettings["NumberOfRetries"];
	var retriesHasValue = int.TryParse(retriesString, out var retriesInt);
	if(retriesHasValue)
		settings.NumberOfRetries = retriesInt;
	else
		settings.NumberOfRetries = 5;

	var pollingHourStr = ConfigurationManager.AppSettings["HourToStartPollingAt"];
	var pollingHourHasValue = int.TryParse(pollingHourStr, out var pollingHourInt);
	if(pollingHourHasValue)
		settings.HourToStartPollingAt = pollingHourInt;
	else
		settings.HourToStartPollingAt = 0;

	var alertEmailStr = ConfigurationManager.AppSettings["AlertEmailAddress"];
	if(string.IsNullOrWhiteSpace(alertEmailStr))
		settings.AlertEmailAddress = "test@thecompany.net";
	else
		settings.AlertEmailAddress = aea.ToString();

	var serverNameString = ConfigurationManager.AppSettings["ServerName"];
	if(string.IsNullOrWhiteSpace(serverNameString))
		settings.ServerName = "TestServer";
	else
		settings.ServerName = sn.ToString();

	return settings;
}
```

这样做一些简单的事情确实需要很多代码，对吧？在那里有很多样板代码噪音，使得代码的意图除了熟悉这类操作的人之外几乎无法理解。而且，如果要添加新的设置，每个设置可能需要新增 5 到 6 行代码。这是一种浪费。

相反，我们可以更加功能化地处理，将结构隐藏在某个地方，只留下代码的意图可见。

通常情况下，这里是一个扩展方法来为我处理业务：

```cs
public static class ExtensionMethods
{
	public static int ToIntOrDefault(this object @this, int defaultVal = 0) =>
		int.TryParse(@this?.ToString() ?? string.Empty, out var parsedValue)
		      ? parsedValue
		      : defaultVal;

	public static string ToStringOrDefault(this object @this, string defaultVal = "") =>
		string.IsNullOrWhiteSpace(@this?.ToString() ?? string.Empty)
	        ? defaultVal
	        : @this.ToString();
}
```

这消除了第一个示例中所有重复的代码，并允许我们转向更可读、以结果为导向的代码示例，如下所示：

```cs
public Settings GetSettings() =>
	new Settings
	{
		NumberOfRetries = ConfigurationManager.AppSettings["NumberOfRetries"]
	                                 .ToIntOrDefault(5),
		HourToStartPollingAt = ConfigurationManager.AppSettings["HourToStartPollingAt"]
		                                .ToIntOrDefault(0),
		AlertEmailAddress = ConfigurationManager.AppSettings["AlertEmailAddress"]
                                  .ToStringOrDefault("test@thecompany.net"),
		ServerName = ConfigurationManager.AppSettings["ServerName"]
		                                .ToStringOrDefault("TestServer"),

	};
```

现在一目了然代码的作用，缺省值是什么，以及如何通过一行代码添加更多的设置。

除了 `int` 和 `string` 之外的任何其他设置值类型都需要创建另一个扩展方法，但这并不是什么大困难。

# 自定义枚举

大多数人在编码时可能都使用过 Enumerables，但您知道表面下有一个引擎可以访问并用于创建各种有趣的自定义行为吗？

使用自定义迭代器时，可以大大减少在需要更复杂行为时循环数据所需的代码行数。不过，首先需要了解 Enumerable 在表面之下是如何工作的。

在 Enumerable 表面下有一个类，驱动枚举的引擎，这使得你可以使用 ForEach 来循环遍历值。它被称为 Enumerator。

枚举器基本上有两个特性：

+   当前：从可枚举中获取当前项。可以随意调用多次，前提是不尝试移动到下一项。如果在首次调用 MoveNext 之前尝试获取当前值，则会抛出异常。

+   MoveNext：从当前项移动，并尝试看看是否有另一个可选择的值。如果找到另一个值，则返回 `True`；如果已经到达 Enumerable 的末尾，或者根本没有元素，则返回 `False`。首次调用此方法时，它将 Enumerator 指向 Enumerable 的第一个元素。

## 查询相邻元素

一个相对简单的例子来开始。假设我想遍历一个整数的 Enumerable，看看是否包含任何连续的数字。

一种命令式的解决方案可能看起来像这样：

```cs
public IEnumerable<int> GenerateRandomNumbers()
{
	var rnd = new Random();
	var returnValue = new List<int>();
	for (var i = 0; i < 100; i++)
	{
		returnValue.Add(rnd.Next(1, 100));
	}
	return returnValue;
}

public bool ContainsConsecutiveNumbers(IEnumerable<int> data)
{
	// OK, you caught me out OrderBy isn't strictly Imperative, but
	// there's no way I'm going to write out a sorting algorithm out
	// here just to prove a point!
	var sortedData = data.OrderBy(x => x).ToArray();

	for (var i = 0; i < sortedData.Length - 1; i++)
	{
		if ((sortedData[i] + 1) == sortedData[i + 1])
			return true;
	}

	return false;
}

var result = ContainsConsecutiveNumbers(GenerateRandomNumbers());
Console.WriteLine(result);
```

要使这段代码功能化，通常情况下，我们需要一个扩展方法。这个方法接收 Enumerable，提取其 Enumerator，并控制定制的行为。

为了避免使用命令式风格的循环，我在这里使用了递归。简而言之，递归是通过让函数重复调用自身来实现无限循环的一种方式。

我将在后面的章节中重新讨论递归的概念。但现在，我只会使用标准的简单递归版本。

```cs
public static bool Any<T>(this IEnumerable<T> @this, Func<T, T, bool> evaluator)
{
	using var enumerator = @this.GetEnumerator();
	var hasElements = enumerator.MoveNext();
	return hasElements && Any(enumerator, evaluator, enumerator.Current);
}

private static bool Any<T>(IEnumerator<T> enumerator,
		Func<T, T, bool> evaluator,
		T previousElement)
{
	var moreItems = enumerator.MoveNext();
	return moreItems && (evaluator(previousElement, enumerator.Current)
		? true
		: Any(enumerator, evaluator, enumerator.Current));

}
```

那么，这里发生了什么？在某种程度上有点像杂耍。我首先提取枚举器，然后移到第一项。

在私有函数内部，我接受枚举器（现在指向第一个项目）、“我们完成了吗”的评估器函数以及同一个第一个项目的副本。

接着，我立即移到下一个项目，并运行评估器函数，传入第一个项目和新的“当前项目”，这样它们就可以进行比较。

此时，要么我们发现物品用完了，要么评估器返回 true，这种情况下我们可以终止迭代。如果 MoveNext 返回 true，那么我们检查`previousValue`和`Current`是否符合我们的要求（由`evaluator`指定）。如果符合，则完成并返回 true，否则我们递归调用以检查其余的值。

这是查找连续数字的代码的更新版本：

```cs
public IEnumerable<int> GenerateRandomNumbers()
{
 var rnd = new Random();

 var returnValue = Enumerable.Repeat(0, 100)
  .Select(x => rnd.Next(1, 100));
 return returnValue;
}

public bool ContainsConsecutiveNumbers(IEnumerable<int> data)
{
 var sortedData = data.OrderBy(x => x).ToArray();
 var result = sortedData.Any((prev, curr) => cur == prev + 1);
 return result;
}
```

这样，基于相同逻辑，创建一个`All`方法也相当容易，如下所示：

```cs
public static bool All<T>(this IEnumerator<T> enumerator, Func<T,T,bool> evaluator, T previousElement)
{
	var moreItems = enumerator.MoveNext();
	return moreItems
		? evaluator(previousElement, enumerator.Current)
				? All(enumerator, evaluator, enumerator.Current)
				: false
		: true;
}

public static bool All<T>(this IEnumerable<T> @this, Func<T,T,bool> evaluator)
{
	using var enumerator = @this.GetEnumerator();
	var hasElements = enumerator.MoveNext();
	return hasElements
	 ? All(enumerator, evaluator, enumerator.Current)
	 : true;
}
```

唯一的区别在于决定是否继续的条件以及我们是否需要提前返回。使用`All`，关键是检查每对值，并且仅在找到不满足条件的情况下提前退出循环。

## 迭代直到满足条件

这基本上是替换 while 循环的一种方法，所以这是我们不一定需要的另一种语句。

对于我的示例，我想象文本冒险游戏的轮换系统可能是什么样子。对于年轻读者来说 - 这就是我们在旧日子里没有图形之前的样子。你过去必须写下你想做的事情，然后游戏会写下发生的事情。有点像书，只不过你自己写发生了什么。

其中一个游戏的基本结构大致如下：

+   写下当前位置的描述

+   接收用户输入

+   执行请求的命令

这里展示了命令式代码如何处理这种情况：

```cs
var gameState = new State
{
  IsAlive = true,
  HitPoints = 100
};

while(gameState.IsAlive)
{
  var message = this.ComposeMessageToUser(gameState);
  var userInput = this.InteractWithUser(message);
  this.UpdateState(gameState, userInput);

  if(gameState.HitPoints <= 0)
    gameState.IsAlive = false;
}
```

原则上，我们想要的是一个类似 Linq 风格的聚合函数，但不是循环遍历数组的方式，然后结束。相反，我们希望它持续循环，直到满足我们的结束条件（玩家死亡）。我这里稍微简化了一下，显然在真正的游戏中，我们的玩家也可能会*赢*。但我的示例游戏就像生活一样，生活并不公平！

对于这种情况的扩展方法，我们可以从尾递归优化调用中受益，我将在后面的章节中研究这方面的选择，但目前我将仅使用简单的递归 - 如果回合很多可能会成为一个问题，但目前它会阻止我过早引入太多想法。

```cs
public static class ExtensionMethods
{
	public static T AggregateUntil<T>(
	  this T @this,
	  Func<T,bool> endCondition,
	  Func<T,T> update) =>
		endCondition(@this)
			 ? @this
			 : AggregateUntil(update(@this), endCondition, update);
}
```

使用这个方法，我可以完全摆脱`while`循环，并将整个回合序列转换为一个函数，如下所示：

```cs
var gameState = new State
{
  IsAlive = true,
  HitPoints = 100
};

var endState = gameState.AggregateUntil(
       x => x.HitPoints <= 0,
       x => {
          var message = this.ComposeMessageToUser(x);
          var userInput = this.InteractWithUser(message);
          return this.UpdateState(x, userInput);
       });
```

这还不完美，但现在它是可以运行的。有更好的方法来处理游戏状态更新的多个步骤，以及如何以函数式的方式处理用户交互的问题。在第[X]章中将有专门的部分讨论这个问题。

# 结论

在本章中，我们探讨了如何使用 Funcs、Enumerables 和扩展方法来扩展 C#，使得编写函数式代码更加容易，并且解决了语言中一些现有的局限性。

我相信我只是触及到了这些技术的皮毛，还有很多其他的技术等待被发现和应用。

在我们的下一章中，我们将讨论高阶函数，以及一些可以利用它们来创建更多有用功能的结构。

¹ 顺便提一句，那是彼得·戴维森。

² 但希望不是无限的！

³ 如果你想亲自体验，请去玩一下史诗冒险游戏 Zork。试着不要被 Grue 吃掉！
