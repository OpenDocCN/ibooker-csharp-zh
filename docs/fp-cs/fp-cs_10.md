# 第十章。记忆化

纯函数的优点不仅在于产生可预测的结果。尽管这是一件好事，但我们还有另一种方法可以利用这种行为来获益。

记忆化有点像缓存，特别是从 `MemoryCache` 的 `GetOrAdd` 函数来说。它的作用是取某种键值，如果该键已存在于缓存中，则将对象返回出来。如果不存在，则需要传入一个生成所需值的函数。

记忆化的工作原理与此相同，只不过其范围可能不会延伸到单个计算之外。

这在某种需要递归进行的多步计算中是有用的，或者因某种原因需要多次执行相同的计算。

或许最好的方法是通过一个例子来解释这一点...​

# Bacon 数

曾经想过有一种娱乐方式来浪费一个或两个下午吗？试试 Bacon 数吧。它基于凯文·贝肯是演艺界的中心人物的想法，连接所有演员。就像所有道路通往罗马一样，所有演员在某个层面上都与凯文·贝肯有连接。一个演员的 Bacon 数是你需要经过的电影连接数，才能到达凯文·贝肯。我们来看几个例子：

**凯文·贝肯**：简单。Bacon 数为 0，因为他**就是**大贝肯本人。

**汤姆·汉克斯**：Bacon 数为 1。他与 KB 在我个人最爱的电影之一《阿波罗 13 号》中合作。经常与汤姆·汉克斯合作的**梅格·瑞恩**也是 1，因为她在《剪切中》中与 KB 合作。

**大卫·田纳特**：Bacon 数为 2。他与**科林·费斯**在电影《圣特里尼安学校 2》中合作出演。科林·费斯与凯文·贝肯合作出演了《真相何在》，这是两部电影，所以他们之间有连接，因此 Bacon 数为 2。令人难以置信的是，**玛丽莲·梦露**的分数也为 2，因为凯文·贝肯出演了《JFK》，与**杰克·莱蒙**合作出演了《热情如火》。

宝莱坞巨星**阿米尔·汗**的 Bacon 数为 3。他与传奇巨星**阿米塔布·巴赫查恩**合作出演了《宝莱坞谈话》。阿米塔布出演了《了不起的盖茨比》，与**托比·麦奎尔**合作出演了《超越所有界限》。

我的 Bacon 数是无穷大！这是因为我从未以演员身份出现在电影中²。而我所知的最高 Bacon 数的持有者是**威廉·鲁弗斯·沙夫特**，一位美国内战将军，他也出现在 1989 年制作的非虚构电影中，这使他的 Bacon 数高达 10！

对了，希望你能理解这些规则。

假设你想编写程序来找出这些演员中 Bacon 数最低的是谁。像这样：

```cs
var actors = new []
{
 "Tom Hanks",
 "Meg Ryan",
 "David Tennant",
 "Marilyn Monroe",
 "Aamir Khan"
};

var actorsWithBaconNumber = actors.Select(x => (a: x, b: GetBaconNumber(x)));

var report = string.Join("\r\n", actorsWithBaconNumber.Select(x =>
 x.a+ ": " + x.b);
```

有很多方法可以计算 GetBaconNumber。最有可能使用某种电影数据的 Web API。还有更高级的“最短路径”算法，但简单起见，我会说是这样的：

1.  获取凯文·贝肯的所有电影。将这些电影中的所有演员分配为 1 号。如果目标演员（例如汤姆·汉克斯）在其中，则返回答案为 1。否则继续。

1.  从前一步骤中的每个演员（不包括凯文·贝肯本人）获取他们的所有电影列表，这些电影尚未被检查。给这些电影中尚未分配号码的所有演员分配值 2。

1.  每次迭代时，将演员集合分配逐渐增加的值，直到最终到达目标演员并返回其编号。

由于存在用于计算这些数字的 API，我们下载演员的电影作品列表或下载演员名单的每部电影都需要大量的处理时间。

此外，这些演员和他们的电影之间有很多重叠，所以除非我们介入并采取措施，否则我们将多次检查同一部电影。

一种选择是创建一个状态对象，传递给某种聚合函数。这是一个不确定的循环，因此我们还需要选择一种妥协功能原则的选项，允许这种循环。

它可能看起来像这样（注意，我编造了 Web API，所以不要期望这在实际应用中能工作）：

```cs
public int CalculateBaconNumber(string actor)
{

	var initialState = (
		checkedActors: new Dictionary<string, int>(),
		actorsToCheck: new[] { "Kevin Bacon" },
		baconNumber: 0
	);

	var answer = initialState.IterateUntil(
		x => x.checkedActors.ContainsKey(actor),
		acc => {
			var filmsToCheck =
			 acc.actorsToCheck.SelectMany(GetAllActorsFilms);
			var newActorsFound = filmsToCheck.SelectMany(x => x.ActorList)
				.Distinct()
				.ToArray();

			return (
				acc.checkedActors.Concat(acc.actorsToCheck
					.Where(x => !acc.checkedActors.ContainsKey(x))
					.Select(x =>
					 new KeyValuePair<string, int>(x, acc.baconNumber)))
					.ToArray()
					.ToDictionary(x => x.Key, x => x.Value),

				newActorsFound.SelectMany(GetAllActorsFilms)
					.SelectMany(x => x.ActorList).ToArray(),

				acc.baconNumber + 1
			);
		});
	return answer.checkedActors[actor];
}
```

这个方法不错，但可以更好。有很多样板代码涉及跟踪演员是否已经被检查过。例如，有很多使用 Distinct 的地方。

使用 Memoization，我们得到一个通用版本的检查，就像一个缓存，但它存在于我们执行的计算范围内，并且不会在其之外持久存在。如果您确实希望保存计算值以在调用此函数时保持持久性，则`MemoryCache`可能是更好的选择。

我可以创建一个 Memoized 函数，以获取我上面列出的演员曾参演的电影列表，就像这样：

```cs
var getAllActorsFilms = (String a) => this._filmGetter.GetAllActorsFilms(a);
var getAllFilmsMemoized = getAllActorsFilms(getAllActorsFilms);

var kb1 = getAllFilmsMemoized("Kevin Bacon");
var kb2 = getAllFilmsMemoized("Kevin Bacon");
var kb3 = getAllFilmsMemoized("Kevin Bacon");
var kb4 = getAllFilmsMemoized("Kevin Bacon");
```

我在那里调用了相同的函数 4 次，按理说它应该去电影数据存储库，并获取数据的新副本 4 次。实际上，仅在填充`kb1`时执行了一次。从那时起，每次都返回相同数据的副本。

另外，请注意，Memoized 版本和原始版本位于不同的行上。这是 C#的限制。您无法在函数上调用扩展方法，只能在`Func`委托上调用，并且箭头函数在存储在变量中之前不是`Func`。

一些函数式语言在 Memoization 方面具有开箱即用的支持，但奇怪的是，F#没有。

这是 Bacon Number 计算的更新版本，这次利用了 Memoization 功能：

```cs
public int CalculateBaconNumber2(string actor)
{

	var initialState = (
		checkedActors: new Dictionary<string, int>(),
		actorsToCheck: new[] { "Kevin Bacon" },
		baconNumber: 0
	);

	var getActorsFilms = GetAllActorsFilms;
	var getActorsFilmsMem = getActorsFilms.Memoize();

	var answer = initialState.IterateUntil(
		x => x.checkedActors.ContainsKey(actor),
		acc => {
			var filmsToCheck = acc.actorsToCheck.SelectMany(getActorsFilmsMem);
			var newActorsFound = filmsToCheck.SelectMany(x => x.ActorList)
				.Distinct()
				.ToArray();

			return (
				acc.checkedActors.Concat(acc.actorsToCheck
						.Where(x => !acc.checkedActors.ContainsKey(x))
						.Select(x => new KeyValuePair<string, int>(x, acc.baconNumber)))
					.ToArray()
					.ToDictionary(x => x.Key, x => x.Value),

				newActorsFound.SelectMany(getActorsFilmsMem)
					.SelectMany(x => x.ActorList).ToArray(),

				acc.baconNumber + 1
			);
		});
	return answer.checkedActors[actor];
}
```

实际上唯一的区别在于我们创建了一个从远程资源获取电影数据的本地版本，然后进行了记忆化，并且以后只引用了记忆化的版本。这意味着保证了没有不必要的重复数据请求。

# 在 C#中实现记忆化

现在你已经了解了一些基础知识，这就是如何为简单的单参数函数创建一个记忆化函数的方法：

```cs
public static Func<T1, TOut> Memoize<T1, TOut>(this Func<T1, TOut> @this)
{
	var dict = new Dictionary<T1, TOut>();
	return x =>
	{
		if (!dict.ContainsKey(x))
			dict.Add(x, @this(x));
		return dict[x];
	};
}
```

这个版本的 Memoize 期望“live data”函数只有一个任意类型的参数。如果期望更多的参数，则需要进一步的 Memoize 扩展方法，像这样：

```cs
public static Func<T1, T2, TOut> Memoize<T1, T2, TOut>(this Func<T1, T2, TOut> @this)
{
	var dict = new Dictionary<string, TOut>();
	return (x, y) =>
	{
		var key = $"{x},{y}";
		if (!dict.ContainsKey(key))
			dict.Add(key, @this(x, y));
		return dict[key];
	};
}
```

现在，要使这个工作正常，我假设`ToString()`方法返回一些有意义的东西，这意味着它很可能必须是一个基本类型（如`string`或`int`）才能正常工作。类的`ToString()`方法通常只返回类的*类型*描述，而不是其属性。

如果你确实需要将类作为参数进行记忆化，那么需要一些创意。保持通用性的最简单方法可能是向`Memoize`函数添加参数，要求开发者提供自定义的`ToString`函数。像这样：

```cs
public static Func<T1, TOut> Memoize<T1, TOut>(
 this Func<T1, TOut> @this,
 Func<T1, string> keyGenerator)
{
	var dict = new Dictionary<string, TOut>();
	return x =>
	{
		var key = keyGenerator(x);
		if (!dict.ContainsKey(key))
			dict.Add(key, @this(x));
		return dict[key];
	};
}

public static Func<T1, T2, TOut> Memoize<T1, T2, TOut>(
 this Func<T1, T2, TOut> @this,
 Func<T1, T2, string> keyGenerator)
{
	var dict = new Dictionary<string, TOut>();
	return (x, y) =>
	{
		var key = keyGenerator(x, y);
		if (!dict.ContainsKey(key))
			dict.Add(key, @this(x, y));
		return dict[key];
	};
}
```

如果你愿意，你可以这样称呼它：

```cs
var getCastForFilm((Film x) => this.castRepo.GetCast(x.Filmid);
var getCastForFilmM = getCastForFilm.Memoize(x => x.Id.ToString());
```

这只有在你的函数保持*纯粹*的情况下才可能实现。如果你的“live”函数有任何副作用，那么你可能不会得到你预期的结果。这取决于那些副作用是什么。

从实际的角度来看，我不担心将日志添加到“live”函数中，但如果生成类的每个实例中预期存在唯一的属性，则我可能会担心。

在某些情况下，可能*希望*结果在对`Memoize`函数的多次调用之间保持持久化，这种情况下，你还需要添加一个 MemoryCache 参数，并从外部传入一个实例。不过我并不认为有很多情况下这是个好主意。

# 结论

在这一章中，我们探讨了记忆化，它是什么以及如何实现它。它是缓存的一种轻量级替代方案，可用于显著减少执行具有大量重复元素的复杂计算所需的时间。

现在理论部分就到这里了！这不仅是本章的结束，也是整本书的这一部分的结束。

第一部分是如何使用函数式思想和现成的 C#来改进你的日常编码。第二部分深入探讨了函数式编程背后的实际理论，以及如何通过一些创意的方法实现它。第三部分将会更加哲学性，为你提供一些关于如何利用在这里学到的知识的下一步方向的提示。

转过来，如果你敢的话，进入…第三部分…。

¹ 这可能并不正确，抱歉贝肯先生。但我仍然爱你！

² 虽然我不会拒绝。有人知道想要招聘一个年迈、超重、英国技术宅的电影导演吗？我可能不会演詹姆斯·邦德，但我愿意试试！
