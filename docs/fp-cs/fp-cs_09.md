# 第九章。无限循环

我们在前几章中看到函数式编程如何用 LINQ 函数如`Select`或`Aggregate`替换`For`和`ForEach`循环。这绝对是太棒了 - 前提是您正在处理固定长度的数组或者一个`Enumerable`，它会自行决定何时结束迭代。

如果你根本不确定你想要迭代多久怎么办？如果你一直迭代直到满足某个条件呢？

这里有一个非功能性代码的例子，展示了我所说的那种事情，基于 Monopoly 棋盘游戏的松散基础。想象一下，你被关在监狱里，你这个淘气鬼！有以下几种方法可以让你脱困：

+   以您所在的货币支付 50（如果您在美国，则为 50 美元，或者在英国的我这里为 50 英镑¹)

+   掷一个双子

+   使用一个“出狱卡”

在真实的 Monopoly 游戏中，还需要考虑其他玩家的回合，但我简化为无限循环，直到满足其中一个条件。如果我真的这么做了，我可能会在这里添加一些验证逻辑，但再次强调，我要保持简单。

```cs
var inJail = true;
var inventory = getInventory();
var rnd = getRandomNumberGenerator();

while(inJail)
{
  var playerAction = getAction();
  if(playerAction == Actions.PayFine)
  {
   inventory.Money -= 50;
   inJail = false;
  }
  else if(playerAction == Actions.GetOutOfJailFree)
  {
   inventory.GetOutOfJailFree -= 1;
   inJail = false;
  }
  else if(playerAction == Actions.RollDice)
  {
   var dieOne = rnd.Random(1, 6);
   var dieTwo = rnd.Random(1,6);
   inJail = dieOne != dieTwo; // Stay in jail if the dice are different
  }
}
```

您无法使用`Select`语句执行上述操作。我们无法确定何时满足条件，因此我们将继续在`While`循环中迭代，直到其中一个条件被满足。

那么我们如何使其功能化？`While`循环是一个语句（具体说来是一个控制流语句），因此它不被函数式编程语言所青睐。

有几种选择，我会逐一描述每一种，但这是一个需要做出某种权衡的领域。每个选择都有后果，我会尽量考虑它们各自的利弊。

系好你的安全带，我们出发吧…​

# 递归

处理无限循环的经典函数式编程方法是使用递归。简而言之，对于那些不熟悉的人 - 递归是使用调用自身的函数。还会有某种条件确定是否应该进行另一次迭代，或者是否实际返回数据。

如果决策是在递归函数的末尾做出的，这被称为*尾递归*。

解决 Monopoly 问题的纯递归解决方案可能如下所示：

```cs
public record Inventory
{
 public int Money { get; set; }
 public int GetOutOfJail { get; set; }
}

// I'm making the Inventory object a Record to make it
// a bit easier to be functional
var inventory = getInventory();
var rnd = getRandomNumberGenerator();

var updatedInventory = GetOutOfJail(inventory);

private Inventory GetOutOfJail(Inventory oldInv)
{
 var playerAction = getAction();
 return playerAction switch
 {
  Actions.PayFine => oldInv with
  {
   Money = oldInv.Money - 50
  },
  Actions.GetOutOfJailFree => oldInv with
  {
   GetOutOfJail = oldInv.GetOutOfJail - 1
  },
  Actions.RollDice =>
  {
   var dieOne = rnd.Random(1, 6);
   var dieTwo = rnd.Random(1,6);
   // return unmodified state, or else
   // iterate again
   return dieOne == dieTwo
    ? oldInv
    : GetOutOfJail(oldInv);
  }
 };
}
```

任务完成了，对吧？嗯，并不完全是这样，我在使用上面的这种函数之前会非常谨慎地考虑。问题在于每个嵌套的函数调用都会在.NET 运行时的堆栈中添加一个新项目，如果递归调用很多，那么这可能会对性能产生负面影响，或者用堆栈溢出异常终止应用程序。

如果确保只有少数迭代，那么递归方法基本上没有什么问题。你还必须确保，如果代码的使用在增强后发生了显著变化，这一点会被重新考虑。可能会出现这样的情况，有一天这个很少使用的函数在某天变成了一个大量迭代的重度使用函数。如果这种情况发生了，业务可能会想知道为什么他们的优秀应用突然变得接近无响应。

所以，正如我所说的，在使用 C#中的递归算法之前，请非常谨慎。这样做的优点是相对简单，而且不需要编写任何样板代码来实现它。

###### 注意

F#以及许多其他更强调函数式的语言，具有称为*尾递归调用优化*的特性，这意味着可以编写递归函数而不会使堆栈溢出。然而，这在 C#中是不可用的，并且将来也没有计划将其提供。根据情况，F#优化将创建带有`while(true)`循环的中间语言（IL）代码，或者利用一个称为`goto`的 IL 命令将执行环境的指针物理地移回循环的开始位置。

我确实调查了从 F#引用通用的尾递归调用，并通过编译的 DLL 将其暴露给 C#的可能性，但这会有自己的性能问题，使其成为一种徒劳的努力。

我在网上看到另一个可能性的讨论，那就是添加一个后构建事件，直接操作 C#编译成的.NET 中间语言，使其回顾性地使用 F#的尾部优化特性。这非常聪明，但对我来说听起来太像辛苦的工作了。这也可能是一个额外的维护任务。

在接下来的部分中，我将探讨一种在 C#中模拟尾递归调用优化的技术。

# 立体跳板（Trampolining）

我并不完全确定术语*Trampolining*的起源，但它早于.NET。我能找到的最早参考文献是 90 年代的学术论文，研究在 C 中实现 LiSP 的一些特性。我猜这个术语甚至比那还要年长一些。

基本思想是你有一个以*thunk*作为参数的函数 - *thunk*是存储在变量中的一段代码。在 C#中，这些是作为`Func`或`Action`实现的。

得到*thunk*后，你创建一个带有`while(true)`的无限循环，并且通过某种方式评估一个条件，以确定循环是否应该终止。这可以通过返回`bool`的额外`Func`或者某种需要通过*thunk*在每次迭代中更新的包装对象来完成。

但归根结底，我们看到的基本上是在代码库的后面隐藏了一个`while`循环。`while`并非纯函数式，但这是我们可能需要妥协的地方之一。从根本上讲，C#是一种混合语言，支持 OO 和 FP 范式。总会有一些地方无法像 F#那样精确地控制其行为。这就是其中之一。

有几种方法可以实现 trampolining，但我倾向于选择这种方法：

```cs
public static class FunctionalExtensions
{
	public static T IterateUntil<T>(
		this T @this,
		Func<T, T> updateFunction,
		Func<T, bool> endCondition)
	{
		var currentThis = @this;

		while (!endCondition(currentThis))
		{
			currentThis = updateFunction(currentThis);
		}

		return currentThis;
	}
}
```

通过附加到类型`T`，它是一个通用的，这个扩展方法因此会附加到 C#代码库中的所有内容。第一个参数是一个`Func`委托，它更新`T`代表的类型到基于外部定义的任何规则的新形式。第二个是另一个`Func`，它返回导致循环终止的条件。

由于这是一个简单的`While`循环，不存在堆栈大小的问题。虽然它不是纯函数式编程，但这是一个折衷的方案。至少，它是代码库中深藏的一个`while`循环实例。也许有一天，微软会发布一个新功能，以便以某种方式实现适当的尾递归调用优化，那么这个函数就可以重新实现，代码应该继续像之前一样工作，但少一个命令式代码特性实例。

使用这个版本的不定迭代，Monopoly 代码现在看起来像这样：

```cs
// we need everything required to both update and
// assess whether we should continue or not in a
// single object, so I'm considering it "state" rather than
// simply inventory
var playerState = geState();
var rnd = getRandomNumberGenerator();

var playerState.IterateUntil(x => {
 var action = GetAction();
  return action switch
  {
   Actions.PayFine => x with
   {
    Money = x.Money - 50,
    LastAction = action
   },
   Actions.GetOutOfJailFree => x with
   {
    GetOutOfJail = x.GetOutOfJail - 1,
    LastAction = action
   },
   _ => x with
   {
    DieOne = rnd.Random(1, 6),
    DieTwo = rnd.Random(1, 6)
   }
  }
},

x => x.LastAction == Actions.PayFine ||
 x.LastAction == Actions.GetOutOfJailFree ||
 x.DieOne == x.DieTwo

);
```

还*有*另一种你可以用来实现 - Trampolining。从功能上看，它的行为与隐藏的`while`循环相同，性能方面也基本相同。

我并不确定是否有额外的好处，而且*个人*感觉它看起来比`while`循环不那么友好，但它可能略微更符合函数式编程范式，因为它省去了`while`语句。

如果你喜欢，可以使用这个，但在我看来，这是个人喜好的问题。

这个版本使用了一个 C#命令，通常在任何其他情况下我都*极力建议*你不要使用。自从 BASIC 时代以来，这种编码方式一直存在，今天仍以某种形式存在 - `goto`命令。

在 BASIC 中，你可以通过调用`goto`并指定行号来移动到任意的代码行。这通常是在 BASIC 中实现循环的方式。但是在 C#中，你需要创建标签，而`goto`只能移动到这些标签处。

这是使用两个标签重新实现`IterateUntil`的方法。一个称为*LoopBeginning*，相当于`while`循环开头的*{*字符。第二个标签称为*LoopEnding*，表示循环的结束，或`while`循环的*}*字符。

```cs
public static T IterateUntil<T>(
 this T @this,
 Func<T, T> updateFunction,
 Func<T, bool> endCondition)
{
	var currentThis = @this;

	LoopBeginning:

		currentThis = updateFunction(currentThis);
		if(endCondition(currentThis))
			goto LoopEnding;
		goto LoopBeginning;

	LoopEnding:

	return currentThis;
}
```

我会让你决定你更喜欢哪个版本。它们几乎是等效的。但无论你做什么，绝对不要在代码中的任何其他地方使用`goto`，除非你绝对，完全，彻底地知道你在做什么，以及为什么没有更好的选择。

像某位爱蛇，没鼻子的邪恶巫师 - `goto`命令既伟大又可怕，如果不明智地使用。

它很强大，可以通过其他任何方式无法实现的方式创建效果，并在某些情况下提高效率。但它也很危险，因为在执行期间，指针可以跳转到代码库中的任意点，而不管这样做是否有任何意义。如果使用不当，您可能会在代码库中遇到难以解释和难以调试的问题。

请非常谨慎地使用`goto`语句。

还有第三个选项，需要相当多的样板代码，但最终看起来比之前的版本友好一些。

看一看，看看你的想法如何。

# 自定义迭代器

第三个选项是在`IEnumerables`和`IEnumerators`上进行调试。关于`IEnumerables`的一点是它们实际上并不是数组，它们只是指向数据“当前”项的指针，并包含如何获取下一项的指令。因此，我们可以创建自己的`IEnumerable`接口实现，但具有自己的行为。

在我们的大富翁示例中，我们希望有一个`IEnumerable`，它将迭代直到用户选择了一个方法来摆脱监狱，或者投掷了双倍。

我们首先创建`IEnumereable`的实现，它只有一个必须实现的函数：`GetEnumerator()`。`IEnumerator`是实际进行枚举工作的类，这是我们接下来要讨论的内容。

## 枚举器的解剖

这实际上就是`IEnumerator`接口的样子（它继承了一些其他接口的函数，因此这里有一些函数是为了满足继承要求而存在的）：

```cs
public interface IEnumerator<T>
{
 object Current { get; }
 object IEnumerator.Current { get; }
 void Dispose();
 bool MoveNext();
 void Reset();
}
```

每个这些函数都有一个非常具体的工作要做：

表 9-1\. IEnumerator 的组成函数

| Function | Behavior | Returns |
| --- | --- | --- |
| Current | 获取当前数据项 | 当前项，或者如果迭代尚未开始则为 null |
| IEnumerator.Current | 作为当前项，这是从 IEnumerable 实现的另一个接口引用的相同项 | 作为当前项 |
| Dispose | 将 Enumerable 中的所有内容拆分以实现 IEnumerator | Void |
| MoveNext | 移动到下一项 | 如果找到另一个项则为 true，如果枚举过程完成则为 false |
| Reset | 回到数据集的开头 | void |

大多数情况下，`Enumerator`只是在数组上进行枚举，这种情况下，我想实现可能应该大致如下：

```cs
public class ArrayEnumerable<T> : IEnumerator<T>
{
	public readonly T[] _data;
	public int pos = -1;
	public ArrayEnumerable(T[] data)
	{
		this._data = data;
	}

	private T GetCurrent() => this.pos > -1 ? _data[this.pos] : default;

	T IEnumerator<T>.Current => GetCurrent();

	object IEnumerator.Current => GetCurrent();

	public void Dispose()
	{
		//  Run!  Run for your life!
		// Run before the GC gets us all!
	}
	public bool MoveNext()
	{
		this.pos++;
		return this.pos < this._data.Length;
	}
	public void Reset()
	{
		this.pos = -1;
	}
}
```

我认为微软的实际代码可能比这复杂得多——你希望有更多的错误处理和参数检查，但这个简单的实现给出了`Enumerator`的工作内容的一个概念。

## 实现自定义枚举器

知道它在表面下如何工作，你就可以看到如何在`Enumerable`中实现任何你想要的行为。我将通过创建一个`IEnumerable`实现来展示这种技术有多强大，它只通过在`MoveNext`中放入以下代码来遍历数组中每个*其他*项：

```cs
public bool MoveNext()
{
	pos += 2;
	return this.pos < this._data.Length;
}

// This turns { 1, 2, 3, 4 }
// into { 2, 4 }
```

怎么样，一个可以在枚举时每个项目循环两次的`Enumerator`，实际上创建了每个项目的副本：

```cs
public bool IsCopy = false;
public bool MoveNext()
{
 if(this.IsCopy)
 {
  this.pos = this.pos + 1;
 }
 this.IsCopy = !this.IsCopy;

 return this.pos < this._data.Length
}

// This turns { 1, 2, 3 }
// into { 1, 1, 2, 2, 3, 3 }
```

或者一个完整的实现，倒着开始，以`Enumerator`外部包装器为开始：

```cs
public class BackwardsEnumerator<T> : IEnumerable<T>
{
 private readonly T[] data;

 public BackwardsEnumerator(IEnumerable<T> data)
 {
  this.data = data.ToArray();
 }

 public IEnumerator<T> GetEnumerator()
 {
  return new BackwardsArrayEnumerable<T>(this.data);
 }

 IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

然后是驱动反向运动的实际`Enumerator`：

```cs
public class BackwardsArrayEnumerable<T> : IEnumerator<T>
{
  public readonly T[] _data;

 public int pos;

 public BackwardsArrayEnumerable(T[] data)
 {
  this._data = data ?? new T[0];
  this.pos = this._data.Length;
 }

 T Current => (this._data != null && this._data.Length > 0 &&
  this.pos >= 0 && this.pos < this._data.Length)
   ? _data[pos] : default;

 object IEnumerator.Current => this.Current;

 T IEnumerator<T>.Current => this.Current;

 public void Dispose()
 {
  // Nothing to dispose
 }

 public bool MoveNext()
 {

  this.pos = this.pos - 1;
  return this.pos >= 0;
 }

 public void Reset()
 {
  this.pos = this._data.Length;
 }
}
```

这个逆向枚举的使用方式几乎与普通枚举完全相同：

```cs
var data = new[] { 1, 2, 3, 4, 5, 6, 7, 8 };
var backwardsEnumerator = new BackwardsEnumerator<int>(data);
var list = new List<int>();
foreach(var d in backwardsEnumerator)
{
 list.Add(d);
}

// list = { 8, 7, 6, 5, 4, 3, 2, 1 }
```

所以，现在你已经看到了如何轻松地创建自己想要的任何自定义行为的`Enumerable`，应该很容易制造一个迭代无限的`Enumerable`。

## 无限循环的可枚举

试试看快速连续说十遍这个章节标题！

正如你在上一节中看到的，其实`Enumerable`并没有特别的原因必须从头开始循环到结尾。我们可以让它表现得任何我们想要的方式。

在这种情况下，我想要做的是——而不是一个数组——我想要传递一种单一的状态对象，以及一个用于确定循环是否应该继续的代码束（即 Thunk 或`Func`委托）。

逆向工作，我要做的第一件事是`Enumerator`。这是一个完全定制的枚举过程，所以我不会试图以任何方式使其通用化。我正在编写的逻辑在游戏状态对象之外是没有意义的。

虽然在我的假想的大富翁实现中，我可能想要进行几次不同的迭代，所以我会使操作和循环终止逻辑稍微通用化。

```cs
public class GameEnumerator : IEnumerator<Game>
{
// I need this in case of a restart
 private Game StartState;
 private Game CurrentState;
 // old game state -> new game state
 private readonly Func<Game, Game> iterator;
 // Should the iteration stop?
 private Func<Game, bool> endCondition;
 // some tricky logic required to ensure the final
 // game state is iterated.  Normal logic is that if
 // the MoveNext function returns false, then there isn't
 // anything pulled from Current, the loop simply terminates
 private bool stopIterating = false;

 public GameEnumerator(Func<Game, Game> iterator,
  Func<Game, bool> endCondition, Game state)
 {
  this.StartState = state;
  this.CurrentState = state;
  this.iterator = iterator;
  this.endCondition = endCondition;
 }

 public Game Current => this.CurrentState;

 object IEnumerator.Current => Current;

 public void Dispose()
 {
  // Nothing to dispose
 }

 public bool MoveNext()
 {
  var newState = this.iterator(this.CurrentState);
  // Not strictly functional here, but as always with
  // this topic, a compromise is needed
  this.CurrentState = newState;

 // Have we completed the final iteration?  That's done after
 // reaching the end condition
  if (stopIterating)
      return false;

  var endConditionMet = this.endCondition(this.CurrentState);
  var lastIteration = !this.stopIterating && endConditionMet;
  this.stopIterating = endConditionMet;
  return !this.stopIterating || lastIteration;
 }

 public void Reset()
 {
 // restore the initial state
  this.CurrentState = this.StartState;
 }
}
```

这就完成了困难的部分！我们有一个引擎在表面下，它允许我们迭代连续的状态，直到我们完成为止——无论我们决定“完成”意味着什么。

下一个需要的项目是运行`Enumerator`的`IEnumerable`。那非常简单：

```cs
public class GameIterator : IEnumerable<Game>
{
 private readonly Game _startState;
 private readonly Func<Game,Game> _iterator;
 private readonly Func<Game,bool> _endCondition;

 public GameIterator(Game startState, Func<Game, Game> iterator,
  Func<Game, bool> endCondition)
 {
  this._startState = startState;
  this._iterator = iterator;
  this._endCondition = endCondition;
 }

 public IEnumerator<Game> GetEnumerator() =>
  new GameEnumerator(this._startState, this._iterator, this._endCondition);
 }

 IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

现在一切都准备就绪，可以执行自定义迭代了。我只需要定义我的自定义逻辑，设置迭代器。

```cs
var playerState = getState();
var rnd = getRandomNumberGenerator();
var endCondition = (Game x) => x.x.LastAction == Actions.PayFine ||
  x.LastAction == Actions.GetOutOfJailFree ||
  x.DieOne == x.DieTwo);

var update = (Game x) => {
  var action = GetAction();
    return action switch
  {
   Actions.PayFine => x with
   {
    Money = x.Money - 50,
    LastAction = action
   },
   Actions.GetOutOfJailFree => x with
   {
    GetOutOfJail = x.GetOutOfJail - 1,
    LastAction = action
   },
   _ => x with
   {
    DieOne = rnd.Random(1, 6),
    DieTwo = rnd.Random(1, 6)
   }
  }
}

var gameIterator = new GameIterator(playerState, update, endCondition);
```

有几种处理迭代本身的选项，我想花点时间讨论每个选项的细节。

## 使用无限迭代器

严格来说，作为一个完全成熟的`Iterator`，可以应用任何 LINQ 操作，以及标准的`ForEach`迭代。

`ForEach` 可能是处理这种迭代最简单的方法，但它并不严格符合函数式编程的要求。这取决于你，如果你愿意通过添加一个有限的语句来妥协，或者想寻找一个更纯粹的函数式替代方案。可能会像这样：

```cs
foreach(var g in gameIterator)
{
// store the updated state outside of the loop.
 playerState = g;

 // Here you can do whatever logic you'd like to do
 // to message back to the player.  Write a message onto screen
 // or whatever is useful for them to be prompted to do another action
}

// At the end of the loop here, the player is now out of jail, and
// the game can continue with the updated version of playerState;
```

老实说，在生产代码中，这不会让我太担心。但是，我们所做的是否定了我们在试图从代码库中清除非函数式代码方面所做的所有工作。

另一种选择涉及使用 LINQ。作为一个完整的 `Enumerable`，我们的 `GameIterator` 可以应用任何 LINQ 操作。但哪些才是最好的呢？

`Select` 是一个明显的起点，但它可能不完全按照你的预期行为。用法基本上与你以前进行的任何普通 `Select` 列表操作一样：

```cs
var gameStates = gameIterator.Select(x => x);
```

这里的诀窍在于我们将 `gameIterator` 视为一个数组，因此从中进行 `Select` 将会得到一个游戏状态数组。基本上，你会得到一个包含用户经历的每一个中间步骤的数组，最后一个元素是最终状态。

将这简化为仅仅最终状态的简单方法是用 `Last` 替代 `Select`：

```cs
var endState = var gameStates = gameIterator.Last();
```

当然，这假设你对中间步骤不感兴趣。也许你想为每个状态更新向用户发送一条消息，那么你可能想选择并提供一个转换。

或许是这样的：

```cs
var messages = gameIterator.Select(x =>
 "You chose to do " + x.LastAction + " and are " +
  (x.InJail ? "In Jail" : "Free to go!");
);
```

不过，这会消除实际的游戏状态，因此可能 `Aggregate` 是一个更好的选择：

```cs
var stateAndMessages = (
 Messages: Enumerable.Empty<string>(),
 State: playerState
);

var updatedStateAndMessages =
 stateAndMessages.Aggregate(stateAndMessages, (acc, x) => (
  acc.Messages.Append("You chose to do " + x.LastAction + " and are " +
   (x.InJail ? "In Jail" : "Free to go!")),
   x
 ));
```

`Aggregate` 过程中每次迭代中的 *x* 是游戏状态的更新版本，它将继续聚合，直到满足声明的结束条件。每次迭代都会向列表附加一条消息，因此最终你得到的是一个包含要传递给玩家的消息数组和游戏状态的 `Tuple`。

请注意，任何使用 LINQ 语句的地方都会以某种方式提前终止迭代 - `First`、`Take` 等，这可能导致我们的实例中玩家仍然处于监狱中的迭代过程提前结束。

当然，这可能是你实际想要的行为！例如，也许你限制玩家在继续游戏的另一部分或另一位玩家的回合之前只能进行几个动作。类似这样的情况。

在这种技术中，你可以提出各种逻辑可能性。

# 结论

我们已经研究了如何在 C# 中实现无限迭代，而不使用 `ForEach` 语句，这将导致代码更清晰，执行过程中的副作用更少。

纯函数式地做这件事情是不太可能的，有几种选择可供选择 - 所有这些选项在某种程度上都对函数式范式进行了妥协，但这就是在 C# 中工作的本质。

你希望使用哪种选项（如果有的话），完全取决于个人选择以及适用于你的项目的任何约束条件。

但是，请在使用递归时非常小心。它是一种快速的迭代方法，完全函数化，但如果不小心，可能会导致内存使用方面的显著性能问题。

在接下来的章节中，我将介绍一种利用纯函数优化算法性能的好方法。

¹ 这在印度等国家可能行不通。50 印度卢比不会带给你太多。这也就意味着，你*真的*认为你能用 200 美元买整条街吗！
