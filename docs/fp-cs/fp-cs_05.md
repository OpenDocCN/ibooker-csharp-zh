# 第五章。高阶函数

欢迎回到我的朋友们，这场永无止境的表演。

本章，我们将探讨高阶函数的用途。我将探讨在 C#中使用它们的新颖方式，以节省您的工作量，并使代码不太可能失败。

但是，*什么是*高阶函数呢？

高阶函数对于一些非常简单的事情来说是一个稍微奇怪的名字。事实上，如果你花了很多时间使用 LINQ，你很可能已经在使用它们了。它们有两种风味，这是第一种：

```cs
var liberatorCrew = new []
{
 "Roj Blake",
 "Kerr Avon",
 "Vila Restal",
 "Jenna Stannis",
 "Cally",
 "Olag Gan",
 "Zen"
};
var filteredList = liberatorCrew.Where(x => x.First() > 'M');
```

传递到`Where`函数中的是一个箭头表达式 - 这只是一种用于编写匿名函数的简写。长格式版本将如下所示：

```cs
function bool IsGreaterThanM(char c)
{
 return c > 'm';
}
```

因此，在这里，函数已作为参数传递给另一个函数，在其内部的其他地方执行。

这是高阶函数的另一个使用示例：

```cs
public Func<int, int> MakeAddFunc(int x) => y => x + y;
```

请注意，这里有两个箭头，而不是一个。我们正在获取一个整数 *x*，并从中返回一个新函数。在该新函数中，对 *x* 的引用将使用在最初调用 MakeAddFunc 时提供的任何内容填充。

例如：

```cs
var addTenFunction = MakeAddFunc(10);
var answer = addTenFunction(5);
// answer is 15
```

通过将 10 传递到`MakeAddFunc`中，在上面的示例中，我创建了一个新函数，其功能只是将 10 添加到您传递给它的任何其他整数中。

简而言之，高阶函数是具有以下一项或多项属性的函数：

+   接受一个函数作为参数

+   将函数作为其返回类型返回

在 C#中，这通常通过`Func`（用于具有返回类型的函数）或`Action`（用于返回 void 的函数）委托类型完成。

这是一个相当简单的想法，甚至更容易实现 - 但它们对你的代码库可能产生的影响是不可思议的。

在本章中，我将介绍如何使用高阶函数来改进您的日常编码方式。

我还将深入研究称为组合子的高阶函数的下一级用法。这些允许以一种创建更复杂和有用行为的方式传递函数。它们之所以被称为组合子，是因为它们源自一种称为组合逻辑的数学技术。你以后不需要担心再听到这个术语，或者关于任何高级数学的引用 - 我不会去那里。只是以防你好奇...​

# 问题报告

要开始，我们将查看一些问题代码。假设您的公司要求您编写一个函数来获取某种数据存储（XML 文件、JSON 文件，谁知道。无所谓），总结每种可能值的数量，然后将该数据传输到其他地方。除此之外，他们希望在找不到任何数据时发送一个单独的消息。我管理一个非常宽松的公司，所以让我们保持有趣，想象你在邪恶银河帝国™工作，并且你正在对你的雷达上的反抗联盟飞船进行分类。

代码可能如下所示：

```cs
 public void SendEnemyShipWeaponrySummary()
 {
  try
  {
    var enemyShips = this.DataStore.GetEnemyShips();
    var summaryNumbers = enemyShips.GroupBy(x => x.Type)
                                    .Select(x => (Type: x.Key, Count: x.Count()));
    var report = new Report
    {
        Title = "Enemy Ship Type",
        Rows = summaryNumbers.Select(X => new ReportItem
        {
            ColumnOne = X.Type,
            ColumnTwo = X.Count.ToString()
        })
    };

    if (!report.Rows.Any())
        this.CommunicationSystem.SendNoDataWarning();
    else
        this.CommunicationSystem.SendReport(report);
  }
  catch (Exception e)
  {
   this.Logger.LogError(e,
   $"An error occurred in {nameof(SendEnemyShipWeaponrySummary)}: {e.Message}");
  }
 }
```

这没问题，对吧？对吧？好吧，想想这种情景。你坐在桌前，吃着你的每日速食面¹，突然发现——就像《侏罗纪公园》一样——你的咖啡里开始有了节奏感的涟漪。这意味着你的噩梦来了。你的老板！让我们假设你的老板是——我随便说的——一个高个子、深沉嗓音的绅士，穿着黑色斗篷，患有可怕的哮喘。而且他真的讨厌人们惹他生气。*非常*讨厌。

他对你创建的第一个函数感到满意。你可以松一口气了。但现在他想要第二个函数。这个函数将创建另一个摘要，但这次是关于每艘飞船的武器水平。无论它们是无武装、轻装、重装还是能毁灭行星的。那种情况。

简单啊，你想。老板会对我多快地完成这个任务感到印象深刻的。所以，你做了看起来最简单的事情 `Ctrl+C`，然后 `Ctrl+V` 复制和粘贴原始内容，改变名称，改变你要总结的属性，最后得到了这样的东西：

```cs
 public void GenerateEnemyShipWeaponrySummary()
 {
  try
  {
    var enemyShips = this.DataStore.GetEnemyShips();
    var summaryNumbers = enemyShips.GroupBy(x => x.WeaponryLevel)
                                    .Select(x => (Type: x.Key, Count: x.Count()));
    var report = new Report
    {
        Title = "Enemy Ship Weaponry Level",
        Rows = summaryNumbers.Select(X => new ReportItem
        {
            ColumnOne = X.Type,
            ColumnTwo = X.Count.ToString()
        })
    };

    if (!report.Rows.Any())
        this.CommunicationSystem.SendNoDataWarning();
    else
        this.CommunicationSystem.SendReport(report);
  }
  catch (Exception e)
  {
   this.Logger.LogError(e,
   $"An error occurred in {nameof(GenerateEnemyShipWeaponrySummary)}: {e.Message}");
  }
 }
```

五秒钟的工作，一天或两天在你象征性的铲子上使劲，还时不时地抱怨这里的工作有多难，而你暗中又在玩今天的 Wordle。完成任务，到处拍背，对吧？对吧？

嗯……这种方法存在几个问题。

首先，让我们考虑单元测试。作为优秀的、正直的代码公民，我们对所有的代码进行单元测试。想象一下，我们对第一个函数进行了彻底的单元测试。当我们复制和粘贴第二个函数时，此时单元测试覆盖率是多少呢？

我给你一个提示——它介于零和零之间。你也可以复制并粘贴测试，这也可以，但这样我们每次复制和粘贴的代码量就更多了。

这种方法不适合扩展。如果我们的老板在这之后还想要另一个函数，再一个，再一个。如果我们最终被要求做 50 个函数？或者 100 个？！那就是大量的代码。你最终会得到一些上千行长的东西，这不是我乐意支持的。

当你考虑到我职业生涯初期发生的一件事情时，情况变得更糟。我曾在一家组织工作，他们有一个桌面应用程序，根据几个输入参数为每个客户进行一系列复杂的计算。每年规则都会改变，但旧的规则基础必须复制，因为可能需要查看以前年份的计算结果。

所以，在我加入团队之前，一直在开发该应用程序的人每年都复制了一大块代码。做了一些小改动，然后在某处添加了指向新版本的链接，完成了。

有一年，我被委派去做这些年度更改，于是我开始了，年轻、无邪，怀揣改变世界的热情。在进行更改时，我注意到了一些奇怪的现象。有一个与我的更改毫无关系的字段出现了错误。我修复了这个错误，但接着我产生了一个让我心情沉重的想法…​

我查看了每个之前版本的代码库，每一年的版本，几乎所有的版本都有同样的 bug。这个 bug 大约 10 年前引入，从那以后每位开发者都精确复制了这个 bug。因此，我不得不多次修复它，使得测试工作的工作量成倍增加。

思考一下这个问题——复制粘贴真的节省了你多少时间吗？我经常处理那些可能在几十年后仍然存在且毫无放弃迹象的应用程序。

当我决定在编码工作中节省时间时，我尝试审视整个应用程序的生命周期，并考虑一个决策在十年后可能产生的后果。

要回到我们的主题，我如何使用高阶函数来解决这个问题？好了，你们准备好了吗？那么，我就开始说了…​

# Thunks

一个代码束，其中包含一个存储计算的存储计算，可以在请求时执行，被正式称为*Thunk*。就像一块木板打在你头上发出的声音一样。关于这是否比读这本书更伤脑筋，这是一个有争议的问题！

在 C#中，我们可以使用`Func`委托来实现这一点。我们可以编写接受`Func`委托作为参数值的函数，以允许我们的函数中某些计算留空，这些空缺可以通过外部世界，通过箭头函数来填补。

虽然这个技术有一个严肃的、正式的数学术语，我喜欢称之为“甜甜圈函数”，因为这更具描述性。它们就像普通函数，但中间有一个空洞！这个空洞我会请别人填补必要的功能。

这是重构问题报告函数的一种潜在方式：

```cs
public void SendEnemyShipWeaponrySummary() =>
  GenerateSummary(x => x.Type, "Enemy Ship Type Summary");

public void GenerateEnemyShipWeaponryLevelSummary() =>
  GenerateSummary(x => x.WeaponryLevel, "Enemy Ship WeaponryLevel");

private void GenerateSummary(Func<EnemyShip, string> summarySelector, string reportName)
{
  try
  {
    var enemyShips = this.DataStore.GetEnemyShips();
    var summaryNumbers = enemyShips.GroupBy(summarySelector)
                                    .Select(x => (Type: x.Key, Count: x.Count()));
    var report = new Report
    {
      Title = reportName,
      Rows = summaryNumbers.Select(X => new ReportItem
      {
          ColumnOne = X.Type,
          ColumnTwo = X.Count.ToString()
      })
  };

  if (!report.Rows.Any())
   this.CommunicationSystem.SendNoDataWarning();
  else
   this.CommunicationSystem.SendReport(report);
  }
  catch (Exception e)
  {
    this.Logger.LogError(e,
    $"An error occurred in {nameof(GenerateSummary)}, report: {reportName}, message: {e.Message}");
  }
}
```

在这个修订版本中，我们获得了一些优势。

首先，每个新报告的额外行数仅为一行！这使得代码库更加整洁，更易于阅读。代码与新函数的意图非常接近——即与第一个函数相同，但有一些变化。

其次，在对第一个功能进行单元测试后，当我们创建第二个功能时，单元测试水平仍然接近 100%。从功能上讲，唯一的区别是报告名称和要汇总的字段。

最后，对基础函数的任何增强或错误修复将同时应用于所有报表函数。这对相对较少的工作量来说带来了很多好处。还有非常高的信心度，如果一个报表函数测试通过了，其他所有报表函数也将会是一样。

有可能会对这个版本感到满意。但如果是我，我实际上会考虑进一步，将带有其`Func`参数的私有版本暴露在接口上，供希望使用它的任何人使用。

像这样：

```cs
public interface IGenerateReports
{
 void GenerateSummary(Func<EnemyShip, string> summarySelector, string reportName)
}
```

实现方式是将前面代码示例中的私有函数公开化。这样一来，至少在希望为不同字段添加额外报告时，不需要修改接口或实现类。

这使得创建报告的工作完全可以由任何消耗这个类的代码模块任意完成。这样做不仅节省了我们这样的开发者在维护报告集方面的大量负担，而且更多地放在关心报告本身的团队手中。想象一下，现在不再需要向开发团队提交多少变更请求。

如果你真的想要野心勃勃，你可以进一步将`Func`参数公开为`Func<ReportLine,string>`，以允许报告类的用户定义自定义格式。你也可以使用`Action`参数来实现定制的日志记录或事件处理。这只是我那个傻乎乎、虚构的报告类。通过这种方式使用高阶函数的可能性是无限的。

尽管这是一个功能编程的特性，但这确实使我们牢牢地遵循了面向对象设计的 SOLID 原则中的*O* - 开闭原则²，即模块应该对扩展开放，对修改关闭。

令人惊讶的是，在 C#中，面向对象和功能编程如何能够互补。我经常认为，开发人员应该确保自己在两种范式中都能够熟练运用，这样才能有效地将它们结合使用。

# 函数链

请允许我介绍你可能从未意识到需要的最好朋友 - Map 函数。这个函数通常也被称为 Chain 和 Pipe，但为了保持一致性，在本书中我们将统一称它为 Map。恐怕很多功能性结构会根据编程语言和实现方式有很多不同的名称，我会尽量在适当时指出。

现在，我是英国人，有一个关于英国人的俗语是我们喜欢谈论天气。这完全是真的。我们的国家有时一天内会经历四季，所以天气对我们来说是一个持续引发兴趣的话题。

曾经我为一家美国公司工作，那时候，当我与同事通过视频通话讨论天气时，话题往往不可避免地转向了天气。他们告诉我外面的温度大约是 100 度。我使用摄氏度工作，所以对我来说这听起来非常像水的沸点。考虑到我的同事并没有因为血液沸腾而尖叫，我怀疑是其他因素在起作用。当然，他们使用的是华氏度，因此我需要将其转换为我理解的单位，用以下公式：

+   减去 32

+   然后，乘以 5

+   然后，除以 9

这将给出大约 38 度的摄氏温度，温暖而舒适，对于人类生活大多数时间是安全的。

我如何在完全这种多步操作中编码这个过程，然后返回一个格式化的字符串呢？我*可以*将它们全部拼接成一行，就像这样：

```cs
public string FahrenheitToCelcius(decimal tempInF) =>
  Math.Round(((tempInF-32) *5 / 9), 2) + "°C";
```

虽然不是很易读，对吧？说实话，在实际编码中，我可能不会对此太过挑剔，但我正在展示一种技术，不想深陷其中，所以请耐心等待。

编写这个多步骤操作的方式如下：

```cs
string FahrenheitToCelcius(decimal tempInF)
{
 var a = tempInF - 32;
 var b = a * 5;
 var c = b / 9;
 var d = Math.Round(c, 2);
 var returnValue = d + "°C";
 return returnValue;
}
```

这样更易读，更易于维护，但仍然存在一个问题。我们正在创建打算仅使用一次然后丢弃的变量。在这个小函数中，这并不是太重要，但如果这是一个庞大的千行函数呢？如果不是这些小小的十进制变量，而是一个大型复杂对象呢？在第 1000 行，那个不打算再次使用的变量仍然在作用域中，并占用内存。创建一个在下一行之后不打算再使用的变量也有点混乱。这就是 Map 发挥作用的地方。

Map 类似于 LINQ `Select` 函数，但不是作用于可枚举的每个元素，而是作用于对象。任何对象。你传递一个 Lambda 箭头函数，方式与 `Select` 相同，只是你的 *x* 参数指的是基础对象。如果你将其应用于可枚举对象，*x* 参数将指整个可枚举对象，而不是其中的单个元素。

这是我修改后的华氏转摄氏度函数的样子：

```cs
public string FahrenheitToCelcius(decimal tempInF)  =>
	tempInF.Map(x => x - 32)
			.Map(x => x * 5)
			.Map(x => x / 9)
			.Map(x => Math.Round(x, 2))
			.Map(x => x + "°C");
```

完全相同的功能，友好的多阶段操作，但没有丢弃的变量。每个箭头函数执行后，它们的内容就会被垃圾回收。被乘以 5 的十进制 *x* 在下一个箭头函数获取其结果并将其除以 9 时，也将被处理掉。

这是实现 Map 的方法：

```cs
public static class MapExtensionMethods
{
	public static TOut Map<TIn, TOut>(this TIn @this, Func<TIn, TOut> f) =>
		f(@this);
}
```

它很小，不是吗？尽管如此，我经常使用这种特定方法。每当我想要对数据进行多步转换时，这使得将整个函数体转换为简单的箭头函数变得更容易，就像我上面基于 Map 的 FahrenheitToCelcius 函数一样。

这个方法还有更高级的版本，包括错误处理等，我将在第七章中详细介绍。但目前，这是一个您可以立即开始玩耍的奇妙小玩具。大叔西蒙送给你的提前圣诞礼物。嘿，嘿，嘿。

如果您不想在每次转换时更改类型，那么可能存在一种更简洁的 Map 实现。如果符合您的需求，这样更清晰、更简洁。

可以这样实现：

```cs
public static T Map<T>(this T @this, params Func<T,T>[] transformations) =>
 transformations.Aggregate(@this, (agg, x) => x(agg));
```

使用它，基本的华氏温度到摄氏温度的转换将会像这样：

```cs
public decimal FahrenheitToCelcius(decimal tempInF)  =>
	tempInF.Map(
	 x => x - 32,
	 x => x * 5,
	 x => x / 9
	 x => Math.Round(x, 2);
```

这可能值得使用，以节省一些简单情况下的样板代码，比如温度转换。请参阅第八章有关柯里化的一些想法，了解如何使其看起来更好。

# 分支组合器

我也听说过这个被称为“Converge”。不过我更喜欢“Fork”，它更详细地描述了它的工作原理。Fork 组合器用于接收单个值，然后同时以多种方式处理它，然后将所有这些单独的分支合并为一个单一的最终值。它可以将一些相当复杂的多步计算简化为一行代码。

这个过程大致会像这样运行：

+   从一个单一值开始

+   将其输入一组“分支”函数 - 每个函数都独立作用于原始输入以产生某种输出

+   “join”函数将分支的结果合并为最终结果。

下面是我可能使用它的几个示例。

如果您想在函数定义中指定参数的数量 - 而不是从数组中具有未指定数量的分支，则可以使用 Fork 来计算平均值：

```cs
var numbers = new [] { 4, 8, 15, 16, 23, 42 }
var average = numbers.Fork(
 x => x.Sum(),
 x => x.Count(),
 (s, c) => s / c
);
// average = 18
```

或者这里有个来自过去的东西，一个用于计算三角形斜边的 Fork：

```cs
var triangle = new Triangle(100, 200);
var hypotenuse = triangle.Fork(
 x => Math.Pow(x.A, 2),
 x => Math.Pow(x.B, 2),
 (a2, b2) => Math.Sqrt(a2 + b2)
);
```

实现看起来像这样：

```cs
public static class ext
{
	public static TOut Fork<TIn, T1, T2, TOut>(
	  this TIn @this,
	  Func<TIn, T1> f1,
	  Func<TIn, T2> f2,
	  Func<T1,T2,TOut> fout)
	{
		var p1 = f1(@this);
		var p2 = f2(@this);
		var result = fout(p1, p2);
		return result;
	}
}
```

请注意，拥有两个泛型类型，每个分支一个，意味着这些函数可以返回任何类型的组合。

你也可以轻松地为任意数量的参数编写版本，但你想考虑的每个额外参数都需要一个额外的扩展方法。

如果您想进一步，并且有无限数量的“分支”，那么只要您愿意使用每个生成的相同中间类型，这很容易实现：

```cs
public static class ForkExtensionMethods
{
 public static TEnd Fork<TStart, TMiddle, TEnd>(
  this TStart @this,
  Func<TMiddle, TEnd> joinFunction,
  params Func<TStart, TMiddle>[] prongs
 )
{
 var intermediateValues = prongs.Select(x => x(@this));
 var returnValue = joinFunction(intermediateValues);
 return returnValue;
}
```

例如，我们可以用它来基于对象创建一个文本描述：

```cs
var personData = this.personRepository.GetPerson(24601);
var description = personData.Fork(
 prongs => string.Join(Environment.NewLine, prongs),
 x => "My name is " + x.FirstName + " " + x.LastName,
 x => "I am " + x.Age + " years old.",
 x => "I live in " + x.Address.Town
)

// This might, for example, produce:
//
// My name is Jean Valjean
// I am 30 years old
// I live in Montreuil-sur-mer
```

使用这个分支示例，我们可以轻松地添加更多描述性的行，但保持相同的复杂性和可读性。

# Alt 组合器

我也见过这被称为“Or”、“Alternate”和“Alternation”。它用于将一组函数绑定在一起以实现相同的目标，但应该依次尝试，直到其中一个返回一个值。

将其视为“尝试方法 A，如果不行，则尝试方法 B，如果不行，则尝试方法 C，如果还不行，我想我们没办法了”的工作方式。

让我们试着想象一种情景，我们可能希望通过尝试多种方法来查找某物：

```cs
var jamesBond = "007"
 .Alt(x => this.hotelService.ScanGuestsForSpies(x),
 x => this.airportService.CheckPassengersForSpies(x),
 x => this.barService.CheckGutterForDrunkSpies(x));

 if(jamesBond != null)
  this.deathTrapService.CauseHorribleDeath(jamesBond);
```

+   只要这三种方法中的一种返回与英国政府的一个酒鬼、边缘厌恶女性主义者、凶恶的雇员相对应的值，那么 jamesBond 变量就不会为空。哪个函数首先返回值就是最后一个要运行的函数。

那么在找到我们的敌人已经逃跑之前，我们如何实现这个函数呢？像这样：

```cs
public static TOut Alt<TIn, TOut>(this TIn @this, params Func<TIn, TOut>[] args) =>
	args.Select(x => x(@this))
	.First(x => x != null);
```

请记住，LINQ 的`Select`函数采用延迟加载的原则运行，所以即使我看起来在将整个`Func`数组转换为具体类型，实际上我并没有，因为`First`函数将阻止在其中一个返回非空值后执行任何元素。LINQ 真是太棒了，不是吗？

# Compose

函数式语言的一个共同特性是能够从一组较小的函数构建出一个高阶函数。任何涉及组合函数的过程都称为*组合*。

JavaScript 库如 RamdaJS³具有出色的组合功能，但是在这种情况下，C#的强类型实际上对其起到了反作用。

在 C#中有几种组合函数的方法。第一种是最简单的，只是使用基本的 Map 函数，如本章前面描述的那样：

```cs
var input = 100M;
var f = (decimal x) => x.Map(x => x - 32)
  .Map(x => x * 5)
  .Map(x => x / 9)
  .Map(x => Math.Round(x, 2))
  .Map(x => $"{x} degrees");
var output = f(input);
// output = "37.78 degrees"
```

在这里，*f*是一个组合的高阶函数。有 5 个函数（例如 x ⇒ x - 32，计算的那些步骤）用于创建它，这些函数被描述为匿名的 lambda 表达式。它们像乐高积木一样组合成一个更大、更复杂的行为。

此时一个有效的问题是 - 组合函数的意义何在？

答案是，您不一定要一次完成整个过程。您可以分步构建它，然后最终使用相同的基础部件创建许多函数。

现在想象一下，我还想拥有一个表示相反转换的`Func`委托 - 我们最终会得到两个这样的函数：

```cs
var input = 100M;
var fahrenheitToCelcius = (decimal x) => x.Map(x => x - 32)
  .Map(x => x * 5)
  .Map(x => x / 9)
  .Map(x => Math.Round(x, 2))
  .Map(x => $"{x} degrees");
var output = fahrenheitToCelcius(input);
Console.WriteLine(output);.
// 37.78 degrees

var input2 = 37.78M;
var celciusToFahrenheit =	(decimal x) =>
 x.Map(x => x * 9)
 .Map(x => x / 5)
 .Map(x => x + 32)
 .Map(x => Math.Round(x, 2))
 .Map(x => $"{x} degrees");
var output2 = celciusToFahrenheit(input2);
Console.WriteLine(output2);
// 100.00 degrees
```

每个函数的最后两行实际上是相同的。重复每次都重复它们有点浪费吗？我们可以使用 Compose 函数消除这种重复：

```cs
var formatDecimal = (decimal x) => x
 .Map(x => Math.Round(x, 2))
 .Map(x => $"{x} degrees");

var input = 100M;
var celciusToFahrenheit = (decimal x) => x.Map(x => x - 32)
  .Map(x => x * 5)
  .Map(x => x / 9);
var fToCFormatted = celciusToFahrenheit.Compose(formatDecimal);
var output = fToCFormatted(input);
Console.WriteLine(output);

var input2 = 37.78M;
var celciusToFahrenheit =	(decimal x) =>
 x.Map(x => x * 9)
 .Map(x => x / 5)
 .Map(x => x + 32);
var cToFFormatted = celciusToFahrenheit.Compose(formatDecimal);
var output2 = cToFFormatted(input2);
Console.WriteLine(output2);
```

从功能上讲，这些使用 Compose 的新版本与仅使用 Map 的先前版本是相同的。

Compose 函数执行的任务与 Map 几乎相同，不同之处在于我们最终生成的是一个`Func`委托，而不是最终值。这是执行 Compose 过程的代码：

```cs
public static class ComposeExtensionMethods
{
	public static Func<TIn, NewTOut> Compose<TIn, OldTOut, NewTOut>(
	 this Func<TIn, OldTOut> @this,
	 Func<OldTOut, NewTOut> f) =>
		x => f(@this(x));
}
```

使用 Compose，我们已经消除了一些不必要的复制。任何对格式化过程的改进都将同时应用于`Func`委托对象。

然而存在一个限制。在 C#中，不能将扩展方法附加到 lambda 表达式或直接到函数上。如果我们将 lambda 表达式引用为`Func`或`Action`委托，则可以将扩展方法附加到 lambda 表达式，但是在此之前，它需要被分配到一个变量中，以便自动设置为委托类型。这就是为什么在上面的示例中，在调用`Compose`之前需要将`Map`函数链分配给变量的原因 - 否则，可以简单地在`Map`链的末尾调用`Compose`并节省变量分配。

这个过程与面向对象编程中通过继承重用代码类似，只是在单行级别进行，而且需要的样板代码要少得多。它还将这些类似的相关代码放在一起，而不是必须分散在不同的类和文件中。

# 转换

Transducer 是一种将基于列表的操作（如 Select 和 Where）与某种形式的聚合结合起来，对值列表执行多个转换，最后将其折叠为单个最终值的方法。

虽然 Compose 是一个有用的功能，但它也有一些限制。它实际上只替代了一个 Map 函数的位置 - 即它作用于整个对象，并且无法对可枚举对象执行 LINQ 操作。*你可以*在数组中组合并放入 Select 和 Where 操作，但老实说，这看起来非常凌乱：

```cs
var numbers = new [] { 4, 8, 15, 16, 23, 42 };
var add5 = (IEnumerable<int> x) => x.Select(y => y + 5);
var Add5MultiplyBy10 = add5.Compose(x => x.Select(y => y * 10));

var numbersGreaterThan100 = Add5MultiplyBy10.Compose(x => x.Where(y => y > 100));

var composeMessage = numbersGreaterThan100.Compose(x => string.Join(",", x));
Console.WriteLine("Output = " + composeMessage(numbers));
// Output = 130,200,210,280,470
```

如果你对此感到满意，那么尽管使用它。本质上并没有什么错，除了相当不雅。

还有另一种结构可以使用 - Transduce。Transduce 操作作用于数组，并代表功能流的所有阶段：

+   过滤（即`.Where`）- 减少元素的数量

+   转换（即`.Select`）- 将它们转换为新的形式

+   聚合（即，嗯... 实际上就是*聚合*）- 使用这些规则将许多项的集合缩减为单个项。

在 C#中可以有许多实现方式，但这是一种可能性：

```cs
public static TFinalOut Transduce<TIn, TFilterOut, TFinalOut>(
	this IEnumerable<TIn> @this,
	Func<IEnumerable<TIn>, IEnumerable<TFilterOut>> transformer,
	Func<IEnumerable<TFilterOut>, TFinalOut> aggregator) =>
		aggregator(transformer(@this));
```

此扩展方法采用一个转换方法 - 用户定义的`Select`和`Where`的任何组合，最终将可枚举对象从一种形式和大小转换为另一种形式。该方法还接受一个聚合器，将转换器的输出转换为单个值。

这是我上面定义的组合函数如何使用此版本的 Transduce 方法实现的方式：

```cs
var numbers = new [] { 4, 8, 15, 16, 23, 42 };

// N.B - I could make this a single line with brackets, but
// I find this more readable, and it's functionally identical due
// to lazy evaluation of Enumerables
var transformer = (IEnumerable<int> x) => x
	.Select(y => y + 5)
	.Select(y => y * 10)
	.Where(y => y > 100);

var aggregator = (IEnumerable<int> x) => string.Join(", ", x);

var output = numbers.Transduce(transformer, aggregator);
Console.WriteLine("Output = " + output);
// Output = 130, 200, 210, 280, 470
```

或者，如果您更喜欢将所有东西都处理为`Func`委托，以便可以重用 Transducer 函数，则可以这样编写：

```cs
var numbers = new [] { 4, 8, 15, 16, 23, 42 };
var transformer = (IEnumerable<int> x) => x
	.Select(y => y + 5)
	.Select(y => y * 10)
	.Where(y => y > 100);

var aggregator = (IEnumerable<int> x) => string.Join(", ", x);

var transducer = transformer.ToTransducer(aggregator);
var output2 = transducer(numbers);
Console.WriteLine("Output = " + output2);
```

这是更新后的扩展方法：

```cs
public static class TransducerExtensionMethod
{
	public static Func<IEnumerable<TIn>, NewTOut> ToTransducer<TIn, OldTOut, NewTOut>(
		this Func<IEnumerable<TIn>,
		IEnumerable<OldTOut>> @this,
		Func<IEnumerable<OldTOut>, NewTOut> aggregator) =>
			x => aggregator(@this(x));
}
```

现在我们生成了一个`Func`委托变量，可以作为函数在任意多个整数数组上使用，该单一`Func`将执行所需数量的转换和过滤，然后将数组聚合为单个最终值。

# 点击

我经常听到有人对函数链提出的一个普遍关注是，在其中执行记录日志是不可能的 - 除非你将链中的一个链接指向一个带有记录调用的单独函数。

函数式编程中有一种技术，可以用来在任何函数链的某个点检查函数链的内容 - Tap 函数。

Tap 函数有点像旧侦探片中的窃听器⁴。它允许监视和处理信息流，但不会干扰或改变它。

实现 Tap 的方式如下：

```cs
public static class Extensions
{
 public static T Tap<T>(this T @this, Action<T> action)
 {
  action(@this);
  return @this;
}
```

一个`Action`委托实际上就像一个无返回值的函数。在这个实例中，它接受一个参数 - 一个泛型类型 T。Tap 函数将链中当前对象的当前值传递给 Action，在那里可以进行记录，然后返回相同对象的未修改副本。

你可以像这样使用它：

```cs
var input = 100M;
var fahrenheitToCelcius = (decimal x) => x.Map(x => x - 32)
  .Map(x => x * 5)
  .Map(x => x / 9)
  .Tap(x => this.logger.LogInformation("the un-rounded value is " + x))
  .Map(x => Math.Round(x, 2))
  .Map(x => $"{x} degrees");
var output = fahrenheitToCelcius(input);
Console.WriteLine(output);
// 37.78 degrees
```

在这个新版本的华氏度转换为摄氏度的函数链中，我现在在基本计算完成后开始窥探它，但在我开始四舍五入和格式化字符串之前。

我在 Tap 中添加了一个调用记录器的调用，但你可以将其换成`Console.WriteLine`或者其他你想要的东西。

# 尝试/捕获

函数式编程中有几种更高级的结构来处理错误。如果你只是想要一些快速简单的东西，你可以在几行代码中快速实现它，但它有其局限性，请继续阅读。否则，试着提前看看下一章的歧视联盟，以及在高级函数结构之后的章节。在那里可以找到大量关于处理没有副作用的错误的内容。

但是现在，让我们看看我们可以用几行简单的代码做些什么……

理论上，在函数风格的代码中间不应该有任何错误。如果一切都按照无副作用的代码、不可变变量等函数式原则进行，你应该是安全的。然而，在边缘处总是可能有一些被认为是不安全的交互。

假设你有一个场景，你想要在一个整数 Id 的外部系统中进行查找。这个外部系统可以是数据库，Web API，网络共享上的平面文件，或者任何其他东西。这些可能性的共同点是，它们中的任何一个都可能由于多种原因而失败，其中很少有任何一个是开发者的错。

可能会出现网络问题，本地或远程计算机上的硬件问题，无意中的人为干预。问题的列表还在继续……

这就是你通常在面向对象代码中处理这种情况的方式：

```cs
pubic IEnumerable<Snack> GetSnackByType(int typeId)
{
 try
 {
  var returnValue = this.DataStore.GetSnackByType(typeId);
  return returnValue;
 }
 catch(Exception e)
 {
  this.logger.LogError(e, $"There aren't any pork scratchings left!");
  return Enumerable.Empty<Snack>()
 }
}
```

关于这个代码块，有两件事我不喜欢。首先是我们必须用多少样板代码来填充代码。我们必须添加大量强大的编码以保护自己，以防我们没有引起的问题。

另一个问题是 try/catch 块本身。它打破了操作顺序，将程序执行从原来的位置移动到一些可能难以找到的地方。在这种情况下，这是一个很好、简单、紧凑的小函数，并且很容易确定 Catch 的位置。不过，我曾在代码库中工作过，其中 Catch 位于比故障发生位置高几层的函数中。在那个代码库中，由于假设某些代码行会被执行而实际上未被执行，经常出现错误。

说实话，我可能对上面的代码块在生产中并不会有太多问题，但如果不加以检查，不良编码实践可能会渗入其中。代码中没有任何阻止未来编码人员在此处引入多级嵌套函数的机制。

我认为最好的解决方案是采用一种方法，消除所有样板文件，并使引入不良代码结构变得困难，甚至不可能。

类似这样：

```cs
pubic IEnumerable<Snack> GetSnackByType(int typeId)
{
 var result = typeId.MapWithTryCatch(this.DataStore.GetSnackByType)
   ?? Enumerable.Empty<Snack>();
 return result;
}
```

我正在执行一个带有内嵌 Try/Catch 的 Map 函数。新的 Map 函数如果一切正常则返回一个值，如果失败则返回 `null`。

扩展方法如下：

```cs
public static class Extensions
{
 public static TOut MapWithTryCatch<TIn,TOut>(this TIn @this, Func<TIn,TOut> f)
 {
  try
  {
   return f(@this);
  }
  catch()
  {
   return default;
  }
 }
}
```

不过这并不是完美的解决方案。那么错误日志记录呢？这是未记录错误消息的重大错误。

有几种方法可以考虑解决这个问题。任何一种都可以，所以按照你的兴致去做。

一种选择是改为使用一个接受 `ILogger` 实例并返回包含 Try/Catch 功能的 `Func` 委托的扩展方法。类似这样：

```cs
public static class TryCatchExtensionMethods
{
 public static TOut CreateTryCatch<TIn,TOut>(this TIn @this, ILogger logger)
 {
  Func<TIn,TOut> f =>
  {
   try
   {
    return f(@this);
   }
   catch(Exception e)
   {
    logger.LogError(e, "An error occurred");
    return default;
   }
  }
 }
}
```

使用方法基本相似：

```cs
public IEnumerable<Snack> GetSnackByType(int typeId)
{
 var tryCatch = typeId.CreateTryCatch(this.logger);
 var result = tryCatch(this.DataStore.GetSnackByType)
   ?? Enumerable.Empty<Snack>();
 return result;
}
```

只增加了一行样板文件，现在开始记录。可惜的是，除了错误本身外，我们无法在消息中添加任何具体内容。扩展方法不知道它被调用的位置或错误的上下文，这使得在整个代码库中重复使用该方法非常方便。

如果你不希望 Try/Catch 意识到 `ILogger` 接口，或者每次都想提供自定义错误消息，那么我们需要考虑一些更复杂的方法来处理错误消息。

另一种选择是返回一个包含正在执行的函数的返回值以及一些关于是否工作、是否存在错误及其内容的元数据对象。类似这样：

```cs
public class ExecutionResult<T>
{
 public T Result { get; init; }
 public Exception Error { get; init; }
}

public static class Extensions
{
 public static ExtensionResult<TOut> MapWithTryCatch<TIn,TOut>(this TIn @this, Func<TIn,TOut> f)
 {
  try
  {
   var result = f(@this);
   return new ExecutionResult<TOut>
   {
    Result = result
   };
  }
  catch(Exception e)
  {
   return new ExecutionResult<TOut>
   {
    Error = e
   };
  }
 }
}
```

我真的不喜欢这种方法。它违反了面向对象设计的 SOLID 原则之一 - 接口隔离原则。嗯，有点。从技术上讲，这适用于接口，但我尽量在任何地方都应用它。即使我写函数式代码。理念是，我们不应被迫在类或接口中包含我们实际上不需要的内容。在这里，我们强制一个成功运行包含一个 `Exception` 属性，它永远不会需要，同样，一个失败运行将不得不包含它永远不会需要的 Result 属性。

还有其他方法可以做到这一点，但我选择了简单的方法，并返回了一个带有结果的 `ExecutionResult` 类的版本，或者一个带有默认值的 Result 和返回的异常。

这意味着我可以像这样调用它：

```cs
pubic IEnumerable<Snack> GetSnackByType(int typeId)
{
 var result = typeId.MapWithTryCatch(this.DataStore.GetSnackByType);
 if(result.Value == null)
 {
  this.Logger.LogException(result.Error, "We ran out of jammy dodgers!");
  return Enumerable.Empty<Snack>();
 }

 return result.Result;
}
```

除了不必要的字段外，这种方法还有另一个问题 - 现在开发者使用 Try/Catch 函数需要增加额外的样板代码来检查错误。

跳到下一章节，以更纯函数式的方式处理此类返回值的替代方法。但现在，这里有一种稍微更清洁的处理方式。

首先，我将添加另一个扩展方法。这次是附加到 ExecutionResult 对象：

```cs
public static T OnError<T>(this ExecutionResult<T> @this, Action<Exception> errorHandler)
 {
if (@this.Error != null)
errorHandler(@this.Error);
return @this.Result;
 }
```

我这里做的第一步是先检查是否有错误。如果有，那么执行用户定义的 `Action` - 这可能是一个日志操作。最后，将 ExecutionResult 解包成其实际返回的数据对象。

所有这些意味着你现在可以这样处理 Try/Catch：

```cs
public IEnumerable<Snack> GetSnackByTypeId(int typeId) =>
	typeId.MapWithTryCatch(DataStore.GetSnackByType)
		.OnError(e => this.Logger.LogError(e, "We ran out of custard creams!"));
```

虽然这远非完美的解决方案，但在不深入函数理论的情况下，它是可行且优雅的，足以不触发我的内部完美主义。这也迫使用户在使用时考虑错误处理，这只能是一件好事！

# 处理空值

空引用异常很烦人，不是吗？如果你想责怪某人，那就是一个叫 Tony Hoare 的家伙，他在 60 年代发明了 Null 的概念。不过，我们最好不要责怪任何人。我相信他是一个可爱的人，所有认识他的人都喜欢他。无论如何，我们可以希望大家都同意，空引用异常确实是一个很大的麻烦。

那么，有没有一种函数式的方式来处理它们？如果你读到这里，你可能知道答案将会是一个响亮的“是！”⁵。

*Unless* 函数接受一个布尔条件和一个 `Action` 委托，并且仅在布尔条件为假时执行 `Action` - 即 `Action` 总是执行，*除非* 条件为真。

这样的用法最常见的情况就是 - 你猜对了 - 检查空值。

这是一个我试图替换的代码示例。这是一个很少见的 Dalek 的源代码片段⁶：

```cs
public void BusinessAsUsual()
{
 var enemies = this.scanner.FindLifeforms('all');
 foreach(var e in enemies)
 {
  this.Gun.Blast(e.Coordinates.Longitude, e.Coordinates.Latitude);
  this.Speech.ScreamAt(e, "EXTERMINATE");
 }
}
```

这一切都很好，也许会留下许多人被一个移动的胡椒罐形状的精神病突变体杀死。但是，如果 Coordinates 对象因某种原因为空呢？没错——空引用异常。

这就是我们使其功能化并引入 Unless 函数以防止异常发生的地方。这就是 Unless 的样子：

```cs
public static class UnlessExtensionMethods
{
 public void Unless<T>(this T @this, Func<bool> condition, Action<T> f)
 {
  if(!condition(@this)
  {
   f(@this);
  }
 }
}
```

很遗憾，它必须是一个 void。如果我们将 `Action` 换成 `Func`，那么从扩展方法返回 `Func` 的结果是可以的。然而，当条件为真时，我们不执行时怎么办？那我返回什么？这个问题真的没有一个答案。

这是我如何用它来制作我的新的、超级、更致命的功能达雷克：

```cs
public void BusinessAsUsual()
{
 var enemies = this.scanner.FindLifeforms('all');

 foreach(var e in enemies)
 {
  e.unless(
   x => x.Coordinates == null,
   x => this.Gun.Blast(e.Coordinates.Longitude, e.Coordinates.Latitude)
  )

 // May as well do this anyway, since we're here.
  this.Speech.ScreamAt(e, "EXTERMINATE");
 }
}
```

使用这个方法，一个空的 Coordinates 对象不会导致异常，枪根本不会被开火。

在接下来的几章中，有更多方法可以预防空指针异常——这些方法需要更高级的编码和一些理论，但在工作方式上更加彻底。敬请期待。

# 更新一个 Enumerable

我将用一个有用的示例结束这一节。它涉及更新一个 Enumerable 中的元素，而不改变任何数据！

关于可枚举的要记住的一点是，它们被设计用来利用“惰性评估”——即直到最后可能的时刻才实际从指向数据源的一组函数转换为实际数据。很多时候，使用 `Select` 函数并不会触发评估，因此我们可以用它们有效地创建过滤器，坐落在数据源和代码中枚举数据的位置之间。

这是修改 Enumerable 的一个示例，使位置 x 处的项目被替换：

```cs
var sourceData = new []
{
 "Hello", "Doctor", "Yesterday", "Today", "Tomorrow", "Continue"
}

var updatedData = sourceData.ReplaceAt(1, "Darkness, my old friend");
var finalString = string.Join(" ", updatedData);
// Hello Darkness, my old friend Yesterday Today Tomorrow Continue
```

我所做的是调用一个函数来替换位置 1（即“Doctor”）的元素为一个新值。尽管有两个变量，在这段代码片段结束后，对源数据实际上并没有做任何操作。此外，直到调用 `string.Join` 时，才会进行实际的替换，因为那是需要具体值的时刻。

这是如何完成的：

```cs
public static class Extensions
{
  public static IEnumerable<T> ReplaceAt(this IEnumerable<T> @this,
    int loc,
    T replacement) =>
    @this.Select((x, i) => i == loc ? replacement : x);
}
```

这里返回的 Enumerable 实际上指向原始 Enumerable，并从那里获取其值，但有一个关键的区别。如果元素的索引等于用户定义的值（在我们的示例中是第二个元素，即 1），那么所有其他值都将不变地传递。

如果你愿意的话，你可以提供一个函数来执行更新——让用户能够基于正在被替换的旧数据项来生成新版本的数据项。

这是你会做到的：

```cs
public static class Extensions
{
  public static IEnumerable<T> ReplaceAt(this IEnumerable<T> @this,
    int loc,
    Func<T, T> replacement) =>
    @this.Select((x, i) => i == loc ? replacement(x) : x);
}
```

使用起来也很简单：

```cs
var sourceData = new []
{
 "Hello", "Doctor", "Yesterday", "Today", "Tomorrow", "Continue"
}

var updatedData = sourceData.ReplaceAt(1, x => x + " Who");
var finalString = string.Join(" ", updatedData);
// Hello Doctor Who Yesterday Today Tomorrow Continue
```

我们也可能不知道要更新的元素的 Id - 实际上可能有多个要更新的项目。这是一种基于提供 T 到 Bool 转换 `Func` 的替代 Enumerable 更新函数，用于标识应该更新的记录。

这个例子是基于桌游 - 我最喜欢的爱好之一 - 让我永远耐心的妻子很恼火！在这种情况下，BoardGame 对象上有一个 Tag 属性，其中包含描述游戏的元数据标签（“家庭”，“合作”，“复杂”等），这将被搜索引擎应用程序使用。已决定为适合单人游戏的游戏添加另一个标签 - “独奏”。

```cs
var sourceData = this.DataStore.GetBoardGames();

var updatedData = sourceData.ReplaceWhen(
		x => x.NumberOfPlayersAllowed.Contains(1),
		x => x with { Tags = x.Tags.Append("solo") });
this.DataStore.Save(updatedData);
```

实现是我们已经讨论过的代码的变体：

```cs
public static class ReplaceWhenExtensions
{
 public static IEnumerable<T> ReplaceWhen<T>(this IEnumerable<T> @this,
  Func<T, bool> shouldReplace,
  Func<T, T> replacement) =>
  @this.Select(x => shouldReplace(x) ? replacement(x) : x);
}
```

此函数可用于替代许多 If 语句的需求，并将它们简化为更简单、更可预测的操作。

# 结论

在本章中，我们探讨了使用高阶函数概念开发丰富功能以避免需要面向对象风格语句的各种方法。

如果您有任何关于自己用于高阶函数用途的想法，请随时与我们联系。您永远不知道，它可能会出现在本书的未来版本中！

在接下来的章节中，我们将探讨**辨识联合**，以及这个函数式概念如何帮助更好地模拟代码库中的概念，并消除通常在非函数式项目中所需的大量防御性代码。享受吧！

¹ 理想情况下，您可以找到最热辣的口味。吃时火焰应从您的嘴里冒出！

² 在此处了解更多信息：[*https://en.wikipedia.org/wiki/SOLID*](https://en.wikipedia.org/wiki/SOLID) - 或者如果您更喜欢视频，这里有一个由我亲自主持的视频：[*https://www.youtube.com/watch?v=0vJb_B47J6U*](https://www.youtube.com/watch?v=0vJb_B47J6U)

³ 自己看看这里：[*https://ramdajs.com/*](https://ramdajs.com/)

⁴ 我猜那可能就是它们得名的原因

⁵ 还有，祝贺您走到了这一步。虽然你花的时间可能没有我多！

⁶ 对于未了解的人来说，它们是英国科幻电视系列《Doctor Who》中的主要反派。在这里看看它们的表现：[*https://www.youtube.com/watch?v=d77jOE2Cjx8*](https://www.youtube.com/watch?v=d77jOE2Cjx8)
