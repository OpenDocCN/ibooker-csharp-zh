# 第二章：什么我们已经可以做了？

本章讨论的一些代码和概念可能对某些人来说显得微不足道，但请耐心等待。更有经验的开发人员可能想跳到第三章，我在其中讨论 C#为函数式程序员提供的最新发展，或者跳到第四章，我在其中展示了一些使用你可能已经熟悉的特性来实现一些函数式特性的新颖方法。

在本章中，我将探讨几乎在今天所有使用中的 C#代码库中可能出现的函数式编程特性。我将假设至少.NET Framework 3.5，并且通过一些微小的修改，本章提供的所有代码示例都将在该环境中工作。即使你在更新版本的.NET 中工作，但对函数式编程不熟悉，我仍然建议阅读本章，因为它应该为你在函数式编程中提供一个很好的起点。

那些对函数式代码已经很熟悉，只想看看在最新版本的.NET 中有什么可用的人，最好跳到下一章。

# 入门

函数式编程真的很简单！尽管许多人认为它比面向对象编程难学，但实际上学起来更简单。需要学习的概念更少，而且你实际上要考虑的东西也更少。

如果你不相信我，试着向你家庭中的非技术人员解释多态性！那些对面向对象编程感到舒适的人往往已经做了很长时间，以至于可能已经忘记了刚开始时有多难理解。

函数式编程并不难理解，只是不同而已。我曾经与许多刚从大学毕业的学生交流过，他们对此充满热情。所以，如果*他们*可以做到...​

尽管如此，关于学习函数式编程需要学习很多东西的传言似乎仍在流传。但如果我告诉你，如果你已经用 C#写了一段时间，你很可能已经在写函数式代码了呢？让我告诉你我的看法...​

# 你的第一个函数式代码

在我们开始编写一些功能性代码之前，让我们先看看一些非功能性的内容。这是你在你的 C#职业生涯开始不久就可能学到的一种风格。

## 一个非功能性的电影查询

在我快速编造的例子中，我从我的想象数据存储中获取了所有电影的列表，并创建了一个新的列表，从第一个列表复制过来，但只包含动作类型的项目¹。

```cs
public IEnumerable<Film> GetFilmsByGenre(string genre)
{
 var allFilms = GetAllFilms();
 var chosenFilms = new List<Film>();

 foreach (var f in allFilms)
 {
  if (f.Genre == genre)
  {
      chosenFilms.Add((f));
  }
 }

 return chosenFilms;

}

var actionFilms = GetFilmsByGenre("Action");
```

这段代码有什么问题？至少，它不够优雅。我们写了很多代码来完成一些相对简单的事情。

我们还创建了一个新对象，只要这个函数在运行，它就会保持在作用域内。如果整个函数只是这么一小段代码，那就没什么好担心的了。但是，如果这只是一个非常长的函数的一部分呢？在那种情况下，allFilms 和 actionFilms 变量都会保持在作用域内，因此在内存中占据位置，即使它们没有被使用。

可能并不一定有所有数据的副本保留在正在复制的项目中，这取决于它是类、结构体还是其他类型。但至少，对于这两个项目在作用域内的时间，会不必要地在内存中保留一组重复的引用。这仍然比我们严格需要保留的内存更多。

我们还强制执行操作的顺序。我们指定了循环的时机，添加的时机等等。每个步骤的执行位置和时间都已经明确。如果数据转换中有任何中间步骤需要执行，我们也会指定它们，并将它们保存在更长寿的变量中。

我可以像这样用`yield`返回解决一些问题：

```cs
public IEnumerable<Film> GetFilmsByGenre(string genre)
{
 var allFilms = GetAllFilms();

 foreach (var f in allFilms)
 {
  if (f.Genre == genre)
  {
      yield return f;
  }
 }
}

var actionFilms = GetFilmsByGenre("Action");
```

不过，这并没有减少多少行代码。

如果有比我们已经决定的更优化的操作顺序呢？如果稍后的代码实际上意味着我们最终并不返回 actionFilms 的内容呢？我们会不必要地做了这些工作。

这是过程式代码的永恒问题。一切都必须一一列举。我们在函数式编程中的主要目标之一是远离这一点。不要对每件小事都如此具体。放松一点，接受声明性的代码。

## 一个函数式的电影查询

那么，上面的代码样本如果按照函数式风格编写会是什么样子？我希望你们中的许多人已经猜到了如何重新编写它。

```cs
public IEnumerable<Film> GetFilmsByGenre(IEnumerable<Film> source, string genre) =>
 source.Where(x => x.Genre == genre);

 var allFilms = GetAllFilms();
 var actionFilms = GetFilmsByGenre(allFilms, "Action");
```

如果此时有人说“这不就是 LINQ 吗？”，是的。没错。我会告诉你们一个小秘密 - LINQ 遵循函数式范式。

只是快速地，对于还不熟悉 LINQ 强大之处的人来说。这是一个自 C#早期就存在的库，为数据集合提供了丰富的过滤、修改和扩展功能。像`Select`、`Where`和`All`这样的函数来自 LINQ，在全球范围内广泛使用。

回想一下函数式编程特性的列表，看看 LINQ 实现了多少……

+   高阶函数 - 传递给 LINQ 函数的 Lambda 表达式都是函数，作为参数变量传入。

+   不变性 - LINQ 不会改变源数组，它返回基于旧数组的新的`Enumerable`。

+   表达式而非语句 - 我们已经消除了`ForEach`和`If`的使用。

+   引用透明性 - 我在这里写的 Lambda 表达式实际上确实符合引用透明性（即“无副作用”），尽管没有强制执行。我很容易可以引用 Lambda 外部的字符串变量。通过要求将源数据作为参数传入，我还使得测试更加容易，而不需要创建和设置某种 Mock 来代表数据存储连接。函数所需的一切都由其自己的参数提供。

迭代也可以通过递归完成，至少我不知道 Where 函数的源代码是什么样的。在没有相反证据的情况下，我只是相信它这样做。

这个微小的一行代码示例在很多方面是函数式方法的完美例子。我们传递函数来对一组数据执行操作，根据旧数据创建新的数据集。

通过遵循函数式范式，我们最终得到了更简洁、更易读，因此更易于维护的东西。

# 结果导向编程

函数式代码的一个共同特征是，它更加专注于最终结果，而不是达到结果的过程。构建复杂对象的完全过程化方法是在代码块开始时将其空实例化，然后在过程中逐步填充每个属性。

像这样：

```cs
var sourceData = GetSourceData();
var obj = new ComplexCustomObject();

obj.PropertyA = sourceData.Something + sourceData.SomethingElse;
obj.PropertyB = sourceData.Ping * sourceData.Pong;

if(sourceData.AlternateTuesday)
{
  obj.PropertyC = sourceData.CaptainKirk;
  obj.PropertyD = sourceData.MrSpock;
}
else
{
   obj.PropertyC = sourceData.CaptainPicard;
   obj.PropertyD = sourceData.NumberOne;
}

return obj;
```

这种方法的问题在于，它很容易被滥用。我在这里创建的这个虚构的小代码块短小且易于维护。然而，在实际的生产代码中经常发生的情况是，代码可能变得非常长，有多个数据源需要预处理、连接、重新处理等。你可能会看到长长的嵌套的 If 语句块，使得代码开始类似家谱的形状。

对于每个嵌套的 If 语句，复杂性实际上是成倍增加的。如果代码库中散布着多个返回语句，情况尤其如此。如果不仔细考虑逐渐复杂的代码库，很容易出现意外结束为 Null 或其他意外值的风险。函数式编程不鼓励这样的结构，并且不容易出现这种复杂性或潜在的意外后果。

在我们上面的代码示例中，我们在两个不同的地方定义了 PropertyC 和 PropertyD。在这里处理起来并不太难，但我见过一些例子，其中同一个属性在多个类和子类中定义了大约半打地方²。

我不知道你是否曾经不得不处理过这样的代码？我却经历过很多次。

这类庞大而笨重的代码库随着时间的推移只会变得更难处理。随着每次增加，开发人员实际完成工作的速度会下降，业务可能会感到沮丧，因为他们不明白为什么他们的“简单”更新会花费如此之长时间。

函数式代码理想情况下应该写成小而简洁的块，完全专注于最终产品。它偏好的表达式是基于数学工作的，所以你真的希望像小公式一样写，精确地定义一个值及其所有组成变量。不应该在代码库中上下搜索以找出值的来源。

类似这样的东西：

```cs
function ComplexCustomObject MakeObject(SourceData source) =>
    new ComplexCustomObject
    {
       PropertyA = source.Something + source.SomethingElse,
       PropertyB = source.Ping * source.Pong,
       PropertyC = source.AlternateTuesday
                     ? source.CaptainKirk
                     : source.CaptainPicard,
       PropertyD = source.AlternateTuesday
                     ? source.MrSpock,
                     : source.NumberOne
    };
```

我知道我现在在重复交替星期二标志，但这意味着所有决定返回属性的变量都在一个地方定义。这样将来处理起来会简单得多。

如果一个属性非常复杂，需要多行代码，或者一系列占用大量空间的 Linq 操作，那么我会创建一个独立的函数来包含这些复杂逻辑。尽管如此，我仍然会保留中心的、基于结果的返回在所有代码的核心位置。

# 关于枚举的几句话

我有时候觉得枚举是 C# 中最被低估和最不被理解的功能之一。一个枚举是数据集合的最抽象表示形式 - 如此抽象，以至于它本身不包含任何数据，实际上只是一个在内存中描述如何获取数据的说明。一个枚举甚至不知道有多少个可用项，直到遍历了所有内容 - 它只知道当前项在哪里，以及如何迭代到下一个。

这被称为 *惰性求值* 或 *延迟执行*。在开发中，偷懒是件好事情。不要让任何人告诉你相反³。

事实上，如果你想的话，甚至可以为枚举编写自定义行为。在表面下，有一个叫做 Enumerator 的对象。与其交互可以用来获取当前项，或者迭代到下一个。你不能用它确定列表的长度，迭代只能单向进行。

看看这段代码示例：

首先是一组简单的日志记录函数，它们将消息放入一个字符串列表中：

```cs
IList<string> c = new List<string>();

public int DoSomethingOne(int x)
{
	c.Add(DateTime.Now + " - DoSomethingOne (" + x + ")");
	return x;
}

public int DoSomethingTwo(int x)
{
	c.Add(DateTime.Now + " - DoSomethingTwo (" + x + ")");
	return x;
}

public int DoSomethingThree(int x)
{
	c.Add(DateTime.Now + " - DoSomethingThree (" + x + ")");
	return x;
}
```

然后是一小段代码，依次调用每个“DoSomething”函数，使用不同的数据。

```cs
var input = new[]
{
	75,
	22,
	36
};

var output = input.Select(x => DoSomethingOne(x))
	 .Select(x => DoSomethingTwo(x))
	 .Select(x => DoSomethingThree(x))
	 .ToArray();
```

你认为操作的顺序是什么？你可能会认为运行时会拿到原始输入数组，对所有 3 个元素应用 DoSomethingOne 来创建第二个数组，然后再对所有三个元素进行 DoSomethingTwo，依此类推。

如果我要检查那个字符串列表的内容，我会找到这样的东西：

```cs
 18/08/1982 11:24:00 - DoSomethingOne(75)
 18/08/1982 11:24:01 - DoSomethingTwo(75)
 18/08/1982 11:24:02 - DoSomethingThree(75)
 18/08/1982 11:24:03 - DoSomethingOne(22)
 18/08/1982 11:24:04 - DoSomethingTwo(22)
 18/08/1982 11:24:05 - DoSomethingThree(22)
 18/08/1982 11:24:06 - DoSomethingOne(36)
 18/08/1982 11:24:07 - DoSomethingTwo(36)
 18/08/1982 11:24:08 - DoSomethingThree(36)
```

实际上，它几乎与通过`For`/`ForEach`循环运行的效果相同，但我们已经有效地将操作顺序控制权交给了运行时。我们不关心临时变量的琐碎细节，不关心什么时候把什么放在哪里。相反，我们只是描述我们想要的操作，并期望在最后得到一个单一的答案。

它可能不总是看起来完全一样，这取决于调用它的代码是什么样的。但意图始终保持不变，即可枚举对象只在实际需要数据时才会产生数据。它们的定义位置并不重要，关键是它们何时被*使用*会有所不同。

通过使用可枚举而不是固定数组，我们实际上已经成功实现了一些需要编写声明性代码的行为。

令人难以置信的是，如果我像这样重写代码，我上面写的日志文件仍然会看起来一样：

```cs
 var input = new[]
 {
     1,
     2,
     3
 };

 var temp1 = input.Select(x => DoSomethingOne(x));
 var temp2 = input.Select(x => DoSomethingTwo(x));
 var finalAnswer = input.Select(x => DoSomethingThree(x));
```

temp1、temp2 和 finalAnswer 都是可枚举的，除非被迭代，否则它们都不包含任何数据。

这里有一个你可以尝试的实验。写一些类似这个示例的代码。不要完全复制，可能简单一些，比如一系列选择操作修改整数值。在 Visual Studio 中设置一个断点，移动操作指针直到 finalAnswer 被传递，然后悬停在 finalAnswer 上。你很可能会发现，即使已经通过了这一行，它也无法向你显示任何数据。因为它实际上还没有执行任何操作。

Things would change if I did something like this:

```cs
 var input = new[]
 {
     1,
     2,
     3
 };

 var temp1 = input.Select(x => DoSomethingOne(x)).ToArray();
 var temp2 = input.Select(x => DoSomethingTwo(x)).ToArray();
 var finalAnswer = input.Select(x => DoSomethingThree(x)).ToArray();
```

因为我现在明确调用`ToArray()`来强制执行每个中间步骤的枚举，那么我们确实会在移动到下一个停止之前为输入中的每个项目调用 DoSomethingOne。

现在日志文件看起来是这样的：

```cs
 18/08/1982 11:24:00 - DoSomethingOne(75)
 18/08/1982 11:24:01 - DoSomethingOne(22)
 18/08/1982 11:24:02 - DoSomethingOne(36)
 18/08/1982 11:24:03 - DoSomethingTwo(75)
 18/08/1982 11:24:04 - DoSomethingTwo(22)
 18/08/1982 11:24:05 - DoSomethingTwo(36)
 18/08/1982 11:24:06 - DoSomethingThree(75)
 18/08/1982 11:24:07 - DoSomethingThree(22)
 18/08/1982 11:24:08 - DoSomethingThree(36)
```

因此，我几乎总是建议在使用`ToArray()`或`ToList()` ⁴之前尽可能等待，因为这样我们可以尽可能地保持操作未执行。如果后续逻辑完全阻止枚举操作发生，甚至可能根本不执行。

有些情况例外。要么是为了性能，要么是为了避免多次迭代。当 Enumerable 保持未枚举状态时，它没有任何数据，但操作本身仍然保留在内存中。如果你将过多的这些操作叠加在一起 - 特别是如果开始执行递归操作，那么你可能会发现内存消耗过大，性能受到影响，甚至可能导致堆栈溢出。

# 更倾向于使用表达式而不是语句。

在本章的其余部分，我将提供更多例子，展示如何更有效地使用 Linq，避免使用诸如 If、Where、For 等语句或改变状态（即改变变量的值）的需要。

会有一些不可能或不理想的情况。但这正是本书其余部分要解决的问题。

## 谦卑的选择

如果你已经读到了本书的这一部分，你很可能已经意识到了 Select 语句以及如何使用它们。不过，大多数我谈话的人似乎并不知道一些功能，它们都是可以用来使我们的代码更加函数式的东西。

第一件事情是我在前一节已经展示过的 - 你可以将它们链接起来。可以作为一系列的 Select 函数调用 - 一个接一个地，或者在一行代码中；或者你可以将每个 Select 的结果存储在不同的本地变量中。从功能上讲，这两种方法是相同的。甚至在每次调用 ToArray 之后都不重要。只要你不修改任何结果数组或其中包含的对象，你就遵循了函数式范式。

重要的是要摆脱的是命令式的做法，即定义一个 List，通过 ForEach 循环遍历源对象，然后将每个新项目添加到 List 中。这样做冗长，阅读起来更困难，老实说相当乏味。为什么要走弯路呢？只需使用一个漂亮简单的 Select 语句。

### 通过元组传递工作值

元组是在 C#7 中引入的。Nuget 包确实存在，允许一些较旧版本的 C# 使用它们。它们基本上是一种快速而肮脏地收集属性的方式，而无需创建和维护一个类。

如果你有一些属性想要在一个地方保留一会儿，然后立即处理掉，元组对于这个很棒。

如果你有多个对象想要在 Select 之间传递，或者想要在一个 Select 中传入或传出多个项目，那么你可以使用元组。

```cs
 var filmIds = new[]
 {
     4665,
     6718,
     7101
 };

 var filmsWithCast = filmIds.Select(x => (
     film: GetFilm(x),
     castList: GetCastList(x)
 ));

var renderedFilmDetails = filmsWithCast.Select(x =>
                @$"
Title: {x.film.Title}
Director: {x.film.Director}
Cast: {string.Join(", ", x.castList)}
".Trim());
```

在我的示例中，我使用一个元组来配对每个给定电影 Id 的两个查找函数的数据，这意味着我可以运行一个后续的 Select 来简化这对对象为一个单一的返回值。

### 需要迭代器值

如果你正在将一个 Enumerable 选择成一个新形式，而且你需要迭代器作为转换的一部分，该怎么办呢？

```cs
 var films = GetAllFilmsForDirector("Jean-Pierre Jeunet")
                                .OrderByDescending(x => x.BoxOfficeRevenue);

 var i = 1;

 Console.WriteLine("The films of visionary French director");
 Console.WriteLine("Jean-Pierre Jeunet in descending order"
 Console.WriteLine(" of financial success are as follows:");

 foreach (var f in films)
 {
     Console.WriteLine($"{i} - {f.Title}");
     i++;
 }

 Console.WriteLine("But his best by far is Amelie");
```

我们可以使用 Select 语句的一个令人惊讶的少有人知道的特性 - 它有一个重载，允许我们访问迭代器作为 Select 的一部分。你所要做的就是提供一个带有 2 个参数的 Lambda 表达式，第二个参数是一个整数，表示当前项目的索引位置。

这是我们代码的函数式版本的样子：

```cs
 var films = GetAllFilmsForDirector("Jean-Pierre Jeunet")
                           .OrderByDescending(x => x.BoxOfficeRevenue);

 Console.WriteLine("The films of visionary French director");
 Console.WriteLine("Jean-Pierre Jeunet in descending order"
 Console.WriteLine(" of financial success are as follows:");

 var formattedFilms = films.Select((x, i) => $"{i} - {x.Title}");
 Console.WriteLine(string.Join(Environment.NewLine, formattedFilms));

 Console.WriteLine("But his best by far is Amelie");
```

使用这些技巧，几乎不存在需要在 List 上使用 `ForEach` 循环的情况。由于 C# 对函数式范式的支持，几乎总是有声明性方法可用来解决问题。

获取“i”索引位置变量的两种不同方法是命令式与声明式代码的一个很好的例子。命令式、面向对象的方法是开发者手动创建一个变量来保存 i 的值，并显式设置变量递增的位置。声明式代码不关心变量的定义位置，也不关心每个索引值是如何确定的。

注意 - 我使用了`string.Join`将字符串链接在一起。这不仅是 C#语言中的另一个隐藏宝石，而且也是聚合的一个例子，即将一组东西转换为单一的东西。我们将在接下来的几节中详细介绍。

### 没有起始数组

对于每次迭代获取 i 值的最后一个技巧非常适用于首先有一个数组 - 或者其他某种集合 - 的情况。如果没有数组呢？如果需要任意迭代一定次数呢？

这些情况 - 有点罕见 - 你需要一个老式的`For`循环而不是`ForEach`。如何从无中创建一个数组呢？

在这种情况下，你的两个最好的朋友是两个静态方法 - `Enumerable.Range` 和 `Enumerable.Repeat`。

Range 从一个起始整数值创建一个数组，并要求你告诉它数组应该有多少元素。然后根据这些规格创建一个整数数组。

例如：

```cs
var a = Enumerable.Range(8, 5);
var s = string.Join(", ", a);
// s = "8, 9, 10, 11, 12"
// That's 5 elements, each one higher than the last,
// starting with 8.
```

制作好一个数组后，我们可以应用 LINQ 操作来得到我们的最终结果。让我们想象我正在为我女儿们准备九乘表的描述⁵。

```cs
var nineTimesTable = Enumerable.Range(1,10)
 .Select(x => x + " times 9 is " + (x * 9));

var message = string.Join("\r\n", nineTimesTable);
```

再举一个例子，如果我想从某种类型的网格中获取所有值，其中需要 x 和 y 值来获取每个值。我想象有一个网格存储库，我可以用来获取值。

想象一下网格是一个 5x5 的，这是我如何获得每一个值的方式：

```cs
var coords = Enumerable.Range(1, 5)
	.SelectMany(x => Enumerable.Range(1, 5)
		.Select(y => (X: x, Y: y))
);

var values = coords.Select(x => this.gridRepo.GetVal(x.Item1,x.Item2);
```

这里的第一行生成一个整数数组，值为`[1, 2, 3, 4, 5]`。然后我使用另一个`Select`将这些整数转换为另一个数组，使用`Enumerable.Range`的另一个调用。这意味着我现在有一个包含 5 个元素的数组，每个元素本身都是一个包含 5 个整数的数组。在嵌套数组上使用 Select，我将这些子元素中的每一个转换为一个元组，该元组从父数组（x）和子数组（y）中取一个值。使用 SelectMany 来将整个结构展平为所有可能坐标的简单列表，看起来像这样：`(1, 1), (1, 2), (1, 3), (1, 4), (1, 5), (2, 1), (2, 2)`…等等。

可以通过将坐标数组选择到存储库的 GetVal 函数的一组调用中获取值，从我在上一行创建的坐标元组中传递 X 和 Y 的值。

我们可能会遇到的另一种情况是需要在每种情况下使用相同的起始值，但需要根据数组中的位置以不同方式进行转换。这就是 `Enumerable.Repeat` 的用武之地。

`Enumerable.Repeat` 创建每个值都完全相同的元素，您可以指定要重复的元素数量。

您无法使用 `Enumerable.Range` 逆向计数。如果我们想做前面的例子，但从 (5,5) 开始向后移动到 (1,1)，这里是如何做的示例：

```cs
var gridCoords = Enumerable.Repeat(5, 5).Select((x, i) => x - i)
	.SelectMany(x => Enumerable.Repeat(5, 5)
		.Select((y, i) => (x, y - i))
);

var values = coords.Select(x => this.gridRepo.GetVal(x.Item1,x.Item2);
```

这看起来更复杂了，但实际上并非如此。我所做的是将 `Enumerable.Range` 调用替换为一个两步操作。

首先调用 `Enumerable.Repeat`，它重复整数值 5 - 5 次。结果得到这样一个数组：`[5, 5, 5, 5, 5]`。

完成这一步后，我使用了 `Select` 的重载版本，其中包括了 i 的值，然后从数组的当前值中减去了该 i 的值。这意味着在第一次迭代中，返回值是数组中的当前值 i（5）减去 i 的值（第一次迭代为 0），因此简单地返回 5。在下一次迭代中，i 的值为 1，所以 5-1 就返回 4。依此类推。

最后，我们得到了一个看起来像这样的数组：`(5, 5), (5, 4), (5, 3), (5, 2), (5, 1), (4, 5), (4, 4)` …​等等。

还有更进一步的方法，但在本章中，我将专注于相对简单的情况，这些情况不需要对 C# 进行调试。这些都是每个人都可以立即使用的开箱即用功能。

## 多对一 - 聚合的微妙艺术

我们已经看过用于将一种东西转换为另一种东西的循环，X 项进入 → X 项新出。这样的事情。还有一个循环的用例我想要涵盖 - 将许多项减少为单个值。

这可能是进行总计、计算平均值、均值或其他统计数据，或者进行其他更复杂的聚合操作。

在过程化代码中，我们会有一个循环，一个状态跟踪值，并在循环内基于数组中的每个项不断更新状态。这里有一个我所说的简单示例：

```cs
 var total = 0;
 foreach(var x in listOfIntegers)
 {
   total += x;
 }
```

实际上，LINQ 中有一个内置的方法可以做到这一点：

```cs
 var total = listOfIntegers.Sum();
```

实际上，根本不应该有必要手动执行这种“长手计算”。即使我们要从对象数组中创建特定属性的总和，LINQ 也能够帮助我们完成：

```cs
 var films = GetAllFilmsForDirector("Alfred Hitchcock");
 var totalRevenue = films.Sum(x => x.BoxOfficeRevenue);
```

对于计算均值同样有一个函数，称为 Average。至少据我所知，没有计算中位数的函数。

我可以用一小段函数式风格的代码来计算中位数，看起来像这样：

```cs
var numbers = new [] {
    83,
    27,
    11,
    98
 };

 bool IsEvenNumber(int number) => number % 2 == 0;

 var sortedList = numbers.OrderBy(x => x).ToArray();
 var sortedListCount = sortedList.Count();

 var median = IsEvenNumber(sortedList.Count())
 				? sortedList.Skip((sortedListCount/2)-1).Take(2).Average()
				: sortedList.Skip((sortedListCount) / 2).First();

// median = 55.
```

有时候需要更复杂的聚合。例如，如果我们想从一个包含复杂对象的 Enumerable 中获取两个不同值的总和会怎样？

过程化代码可能如下所示：

```cs
 var films = GetAllFilmsForDirector("Christopher Nolan");

 var totalBudget = 0.0M;
 var totalRevenue = 0.0M;

 foreach (var f in films)
 {
     totalBudget += f.Budget;
     totalRevenue += f.BoxOfficeRevenue;
 }
```

我们可以使用两个单独的 Sum 函数调用，但那么我们将两次迭代 Enumerable，这绝对不是获取信息的有效方式。相反，我们可以使用 Linq 的另一个奇特而不为人知的特性 - 聚合函数。它包括以下组件：

+   种子 - 最终值的起始值。

+   一个聚合函数，它有两个参数 - 我们正在聚合的 Enumerable 中的当前项，和当前的累计值。

种子不一定是原始类型，比如整数或其他东西，它同样可以是一个复杂对象。然而，为了以函数式风格重写上面的代码示例，我们只需要一个简单的元组。

```cs
 var films = GetAllFilmsForDirector("Christopher Nolan");

 var (totalBudget, totalRevenue) = films.Aggregate(
	(Budget: 0.0M, Revenue: 0.0M),
	(runningTotals, x) => (
			runningTotals.Budget + x.Budget,
			runningTotals.Revenue + x.BoxOfficeRevenue
		)
);
```

在正确的位置，Aggregate 是 C#中一个非常强大的功能，值得花时间去探索和正确理解。

它还是函数式编程中另一个重要概念的示例 - 递归。

## 定制化迭代行为

递归存在于许多迭代的函数式版本中。为了不了解的人好处，它是一个重复调用自身的函数，直到满足某些条件为止。

这是一种非常强大的技术，但在 C#中有一些需要记住的限制。最重要的两个是：

+   如果开发不当，可能导致无限循环，这会一直运行直到用户终止应用程序，或者堆栈上的所有可用空间被消耗完。正如著名英国奇幻 RPG 游戏秀《Knightmare》的传奇地下城主 Treguard 所说：“哦，那真是个恶心的事情”⁶。

+   在 C#中，它们倾向于消耗比其他形式的迭代更多的内存。虽然有办法解决这个问题，但那是另一个章节的话题。

我对递归还有很多话要说，我们很快会讨论到这一点，但是对于这一章节的目的，我将给出我能想到的最简单的例子。

假设你想要遍历一个 Enumerable 但是你不知道要遍历多久。假设你有一个整数的增量值列表（即每次添加或减少的量），你想找出从起始值（无论是什么）到 0 需要多少步。

你可以很容易地通过一个 Aggregate 调用获得最终值，但我们不想要最终值。我们对所有中间值感兴趣，并且希望通过迭代提前停止。这是一个简单的算术操作，但如果涉及到复杂对象在真实场景中，提前终止过程可能会显著节省性能。

在过程式代码中，你可能会写成这样：

```cs
var deltas = GetDeltas().ToArray();
var startingValue = 10;
var currentValue = startingValue;
var i = -1;

foreach(var d in deltas)
{
   if(currentValue == 0)
   {
     break;
   }
   i++;
   currentValue = startingValue + d;

}

return i;
```

在这个例子中，我返回-1 来表明起始值已经是我们要找的值，否则我返回导致达到 0 的数组的基于零的索引。

这是我递归地完成它的方式：

```cs
 var deltas = GetDeltas().ToArray();

 int GetFirstPositionWithValueZero(int currentValue, int i = -1) =>
     currentValue == 0
         ? i
         : GetFirstPositionWithValueZero(currentValue + deltas[i], i + 1);

 return GetFirstPositionWithValueZero(10);
```

现在这已经是函数式的了，但实际上并不是理想的。嵌套函数有其存在的必要性，但我个人认为它在这里的使用方式并不像代码本可以那样可读。虽然递归很美妙，但我认为可以更清晰些。

另一个主要问题是，如果 delta 列表很大，这种方法将无法很好地扩展。我会给你展示一下我的意思。

让我们假设 Deltas 只有 3 个值：2，-12 和 9。在这种情况下，我们期望答案返回 1，因为数组的第二个位置（即索引=1）导致了 0（10+2-12）。我们也期望 9 永远不会被评估。这就是我们在这里寻找代码效率的节约。

不过实际上递归代码的情况是这样的。

首先，它以当前值为 10（即起始值）调用 GetFirstPositionWithValueZero，并允许 i 为默认值-1。

函数的主体是一个三元 if 语句。如果达到了零，返回 i，否则再次调用函数，但使用更新后的当前值和 i。

这就是第一个 delta（即 i=0，即 2）时会发生的情况，所以 GetFirstPositionWithValueZero 现在被调用，当前值更新为 12，并且 i 为 0。

新值不是 0，所以第二次调用 GetFirstPositionWithValueZero 将再次调用自身，这次当前值更新为 delta[1]并且 i 增加到 1。delta[1]为-12，这意味着第三次调用会返回 0，因此 i 可以简单地被返回。

不过，问题来了……

第三次调用得到了一个答案，但前两次调用仍然保留在内存中并存储在堆栈上。第三次调用返回 1，这个值传递到第二次调用 GetFirstPositionWithValueZero，现在它也返回 1，依此类推……直到最初的第一次调用 GetFirstPositionWithValueZero 返回 1。

如果你想稍微图形化地看一下，可以想象它看起来像这样：

```cs
GetFirstPositionWithValueZero(10, -1)
   GetFirstPositionWithValueZero(12, 0)
      GetFirstPositionWithValueZero(0, 1)
      return 1;
   return 1;
return 1;
```

这在我们的数组中有 3 个项目时是可以接受的，但如果有数百个呢！

递归，正如我所说的，是一个强大的工具，但在 C#中会有成本。更纯粹的函数式语言（包括 F#）有一个称为*尾递归优化*的特性，允许使用递归而不会出现内存使用问题。

尾递归是一个重要的概念，我将在后面专门的一章中回到它，所以我不会在这里进一步详细讨论它。

就目前而言，C#的默认设置不允许尾递归，尽管在.NET 公共语言运行时（CLR）中是可用的。我们可以尝试一些技巧来使其对我们可用，但这些技巧对本章来说有点复杂，所以我会在稍后的某个时候再谈论它们。

暂时，按照这里描述的方式考虑递归，并记住您可能希望在何时何地使用它时要小心。

# 不可变性

函数式编程在 C# 中不仅仅是 Linq。我想讨论的另一个重要特性是不可变性（即一旦声明，变量的值不会改变）。在 C# 中可能达到何种程度？

首先，关于 C# 8 及更高版本的不可变性有一些新的发展。请参阅下一章了解更多内容。对于本章来说，我限制自己只讨论.NET 的几乎所有版本都适用的内容。

首先，让我们考虑一下这个小的 C# 片段：

```cs
public class ClassA
{
  public string PropA { get; set; }
  public int PropB { get; set; }
  public DateTime PropC { get; set; }
  public IEnumerable<double> PropD { get; set; }
  public IList<string> PropE { get; set; }
}
```

这是不可变的吗？非常不是。任何这些属性都可以通过设置器替换为新值。IList 也提供一组函数，允许其底层数组进行添加或删除操作。

我们可以将设置器设为私有，这意味着我们必须通过详细的构造函数来实例化类：

```cs
public class ClassA
{
  public string PropA { get; private set; }
  public int PropB { get; private set; }
  public DateTime PropC { get; private set; }
  public IEnumerable<double> PropD { get; private set; }
  public IList<string> PropE { get; private set; }

  public ClassA(string propA, int propB, DateTime propC, IEnumerable<double> propD, IList<string> propE)
  {
    this.PropA = propA;
    this.PropB = propB;
    this.PropC = propC;
    this.PropD = propD;
    this.PropE = propE;
  }

}
```

现在是不是不可变的？不，老实说不是。确实，你不能在 ClassA 外部直接替换任何属性，这很好。属性可以在类内部替换，但开发人员可以确保永远不会添加这样的代码。希望你有某种代码审查系统来确保这一点。

PropA 和 PropC 都很好 - 字符串和 DateTime 在 C# 中都是不可变的。PropB 的 int 值也没问题 - int 类型除了其值外无法更改任何内容。

然而还存在一些问题。

PropE 是一个列表，即使我们不能替换整个对象，仍然可以添加、删除和替换值。如果我们实际上不需要持有 PropE 的可变副本，我们可以轻松地将其替换为 IEnumerable 或 IReadOnlyList。

`IEnumerable<double>` 类型的 PropD 乍一看似乎没问题，但如果作为一个 `List<double>` 传递给构造函数，而外部仍然引用这种类型，那么它仍然可以通过这种方式改变其内容。

还有可能引入类似于这样的东西：

```cs
public class ClassA
{
  public string PropA { get; private set; }
  public int PropB { get; private set; }
  public DateTime PropC { get; private set; }
  public IEnumerable<double> PropD { get; private set; }
  public IList<string> PropE { get; private set; }
  public SubClassB PropF { get; private set; }

  public ClassA(string propA, int propB, DateTime propC, IEnumerable<double> propD, IList<string> propE, SubClassB propF)
  {
    this.PropA = propA;
    this.PropB = propB;
    this.PropC = propC;
    this.PropD = propD;
    this.PropE = propE;
    this.PropF = propF
  }

}
```

PropF 的所有属性可能也会是可变的 - 除非在那里也遵循具有私有设置器的相同结构。

从外部代码库中获取的类有什么不同呢？微软的类或第三方 NuGet 包中的类呢？没有办法强制不可变性。

不幸的是，根本没有办法强制通用的不可变性，即使在最新版本的 C# 中也是如此。出于向后兼容的原因，我认为永远都不会有。

如果有一种本地化的 C# 方法可以默认确保不可变性，那将会很棒，但实际上并没有 - 也不太可能因为向后兼容的原因。我的解决方案是，在编码时，我简单地 *假装* 项目中存在不可变性，从不改变任何对象。在 C# 中，没有任何形式的强制执行，因此你只能自己或团队内部做出决定。

# 将所有内容整合在一起 - 完整的功能流程

我已经谈了很多关于如何立即使用一些简单技术使你的代码更具功能性。现在，我想展示一个完整的、虽小但完整的应用程序，用于展示一个端到端的功能流程。

我将编写一个非常简单的 CSV 解析器。在我的示例中，我想要读取包含有关《Doctor Who》前几个系列数据的完整 CSV 文件的文本⁷。我想要读取数据，将其解析为普通的 C#对象（POCO，即仅包含数据而没有逻辑的类），然后将其聚合成一个报告，该报告计算每个季度的剧集数以及已知丢失的剧集数。⁸。为了这个示例的目的，我简化了 CSV 解析。我不会担心字符串字段周围的引号，字段值中的逗号或需要额外解析的任何值。对于所有这些都有第三方库！我只是在证明一个观点。

这个完整的过程代表了一个很好的、典型的功能流。将单个项拆分成列表，应用列表操作，然后再次聚合为单个值。

这是我的 CSV 文件的结构：

+   [0] - 季节数。整数值介于 1 和 39 之间。现在看起来我有点冒险，因为到目前为止已经有 39 个季度了。

+   [1] - 故事名称 - 我不关心的字符串字段

+   [2] - 编剧 - 同上

+   [3] - 导演 - 同上

+   [4] - 剧集数 - 在《Doctor Who》中，所有故事包含 1 到 14 集。直到 1989 年，所有故事都是多集连续剧。

+   [5] - 缺失剧集数 - 这部连续剧中不知道存在的集数。任何非零数目对我来说都太多了，但这就是生活。

我想最终得到的报告只包括以下字段：

+   季节数

+   总剧集数

+   总缺失剧集数

+   缺失百分比

让我们继续编写一些代码……。

```cs
var text = File.ReadAllText(filePath);

// Split the string containing the whole contents of the
// file into an array where each line of the original file
// (i.e. each record) is an array element
var splitLines = text.Split(Environment.NewLine);

// Split each line into an array of fields, splitting the
// source array by the ',' character.  Convert to Array
// for each access.
var splitLinesAndFields = splitLines.Select(x => x.Split(",").ToArray());

// Convert each string array of fields into a data class.
// parse any non-string fields into the correct type.
// Not strictly necessary, based on the final aggregation
// that follows, but I believe in leaving behind easily
// extendible code
var parsedData = splitLinesAndFields.Select(x => new Story
{
   SeasonNumber = int.Parse(x[0]),
   StoryName = x[1],
   Writer = x[2],
   Director = x[3],
   NumberOfEpisodes = int.Parse(x[4]),
   NumberOfMissingEpisodes = int.Parse(x[5])
});

// group by SeasonNumber, this gives us an array of Story
// objects for each season of the TV series
var groupedBySeason = parsedData.GroupBy(x => SeasonNumber);

// Use a 3 field Tuple as the aggregate state:
// S (int) = the season number.  Not required for
//                the aggregation, but we need a way
//                to pin each set of aggregated totals
//                to a season
// NumEps (int) = the total number of episodes in all
//                serials in the season
// NumMisEps (int) = The total number of missing episodes
//                from the season
var aggregatedReportLines = groupedBySeason.Select(x =>
    x.Aggregate((S: x.Key, NumEps: 0, NumMisEps: 0),
      (acc, val) => (acc.S,
                       acc.NumEps + val.NumberOfEpisodes,
                        acc.NumMisEps + val.NumberOfMissingEpisodes)
    )
);

// convert the Tuple-based results set to a proper
// object and add in the calculated field PercentageMissing
// not strictly necessary, but makes for more readable
// and extendible code
var report = aggregatedReportLines.Select(x => new ReportLine
{
   SeasonNumber = x.S,
   NumberOfEpisodes = x.NumEps,
   NumberOfMIssingEpisodes = x.NumMisEps,
   PercentageMissing = (x.NumMisEps/x.NumEps)*100
});

// format the report lines to a list of strings
var reportTextLines = report.Select(x => $"{x.SeasonNumber}\t {x.NumberOfEpisodes}\t" +
$"{x.NumberofMissingEpisodes}\t{x.PercentageMissing}");

// join the lines into a large single string with New Line
// characters between each line
var reportBody = string.Join(Environment.NewLine, reportTextLines);
var reportHeader = "Season\tNo Episodes\tNo MissingEps\tPercentage Missing";

// the final report consists of the header, a new line, then the reportbody
var finalReport = $"{reportHeader}{Environment.NewLine}{reportTextLines}";
```

如果你感兴趣，结果看起来会像这样（“\t”字符是制表符，使其更易读）：

```cs
Season   No Episodes   No Missing Eps   Percentage Missing,
1         42               9                  21.4
2         39               2                  5.1
3         45               28                 62.2
4         43               33                 76.7
5         40               18                 45
6         44               8                  18.2
7         25               0                  0
8         25               0                  0
9         26               0                  0

...
```

注意，我本可以使代码示例更简洁，并且像这样将所有内容写在一个长长的连贯表达式中：

```cs
var reportTextLines = File.ReadAllText(filePath)
                      .Split(Environment.NewLine)
                      .Select(x => x.Split(",").ToArray())
                      .GroupBy(x => x[0])
                      .Select(x =>
    x.Aggregate((S: x.Key, NumEps: 0, NumMisEps: 0),
      (acc, val) => (acc.S,
                       acc.NumEps + int.Parse(va[4]),
                        acc.NumMisEps + int.Parse(val[5]))
    )
)
.Select(x => $"{x.S}, {x.NumEps},{x.NumMisEps},{(x.NumMisEps/x.NumEps)*100}");

var reportBody = string.Join(Environment.NewLine, reportTextLines);
var reportHeader = "Season,No Episodes,No MissingEps,Percentage Missing";

var finalReport = $"{reportHeader}{Environment.NewLine}{reportHeader}";
```

这种方法没有错，但我喜欢将其拆分成单独的行，原因有几个：

+   变量名提供了一些关于你的代码正在做什么的见解。我们有点像是在半强制性地进行代码注释。

+   可以检查中间变量，查看每个步骤中的内容。这使得调试更容易，因为正如我在前一章中所说的那样——就像在数学答案中回顾你的工作，看看哪一步错了。

没有最终的功能差异，没有任何会被最终用户注意到的地方，所以采用哪种风格更多是个人品味的问题。无论你以何种方式写作，都要保持可读性和易于跟踪。

# 更进一步 - 发展你的函数式编程技能

这里有一个挑战给你。如果这里描述的一些或全部技术对你来说是新的，那就去尝试一下，享受一番吧。

挑战自己按照以下规则编写代码：

+   将所有变量视为不可变的 - 一旦设置，不要更改任何变量值。基本上把所有东西都当作常量对待。

+   不允许使用以下语句 - `If`、`For`、`ForEach`、`While`。只有在三元表达式中可以接受`If` - 例如：someBoolean ? valueOne : valueTwo。

+   尽可能写尽可能多的函数，简洁的箭头函数（也称为 Lambda 表达式）。

要么将其作为你的生产代码的一部分，要么去寻找一个代码挑战网站，比如[The Advent of Code (https://adventofcode.com)](https://adventofcode.com/)或[Project Euler (https://projecteuler.net)](https://projecteuler.net/)。找些你感兴趣的事情来做。

如果你不想在 Visual Studio 中为这些练习创建整个解决方案，总还有 LINQPad（[*https://www.linqpad.net/*](https://www.linqpad.net/)）可以快速轻松地编写一些 C#代码。

当你掌握了这些之后，你就准备好迈向下一步了。希望你到目前为止玩得开心！

# 摘要

在本章中，我们探讨了一些基于简单 Linq 技术的方法，可立即在任何 C#代码库中使用至少.NET Framework 3.5 编写函数式风格的代码，因为这些功能是永恒的，并且在.NET 的每个后续版本中都无需更新或替换。

我们讨论了 Select 语句的更高级特性，Linq 的一些较少知名的特性以及聚合和递归方法。

在下一章中，我将探讨一些最新的 C#开发进展，可以在更新的代码库中使用。

¹ 说实话，我更喜欢科幻。

² 并且在一个例子中，一些定义也在数据库存储过程之外。

³ 除了你的雇主。他们支付你的账单。如果他们很好的话，每年还会送你一张生日卡。

⁴ 作为一名函数式程序员，我坚信应尽可能暴露最抽象的接口，因此我从来不使用`ToList()`。即使`ToList`稍微快一点，我也只用`ToArray()`。

⁵ 不行，Sophie。仅仅用手指操作是不够的！

⁶ 看看他本人的视频 [这里 (https://www.youtube.com/watch?v=OISR3al5Bnk)](https://www.youtube.com/watch?v=OISR3al5Bnk)

⁷ 对于那些不熟悉的人来说，这是一部自 1963 年以来断断续续播放的英国科幻系列。在我看来，这是有史以来最伟大的电视系列。关于这一点，我不接受任何异议。

⁸ 令人遗憾的是，BBC 在 1970 年代销毁了许多该系列的剧集。如果你有其中的任何剧集，请归还给我。
