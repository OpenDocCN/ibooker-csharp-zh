# 第十章 LINQ

语言集成查询（LINQ）是一组强大的 C# 语言功能，用于处理信息集合。它在任何需要处理多个数据片段的应用程序中都很有用（即几乎所有应用程序）。尽管其最初的目标之一是提供对关系数据库的简单访问，但 LINQ 适用于许多类型的信息。例如，它还可以与内存中的对象模型、基于 HTTP 的信息服务、JSON 和 XML 文档一起使用。正如我们将在 第十一章 中看到的那样，它还可以与实时数据流一起使用。

LINQ 不是单一的功能。它依赖于多个语言元素共同工作。与 LINQ 相关的最显著的语言特性是*查询表达式*，这是一种类似数据库查询的表达形式，但可以用来对任何支持的源执行查询，包括普通的旧对象。正如你将看到的那样，查询表达式在很大程度上依赖于一些其他语言特性，例如 lambda 表达式、扩展方法和表达式对象模型。

语言支持只是故事的一半。LINQ 需要类库来实现一组查询原语，称为*LINQ 操作符*。每种不同类型的数据都需要其自己的实现，而某种类型信息的一组操作符被称为*LINQ 提供者*。（顺便说一句，这些也可以从 Visual Basic 和 F# 中使用，因为这些语言也支持 LINQ。）Microsoft 提供了几个提供者，其中一些内置于运行库中，一些作为独立的 NuGet 包提供。例如，Entity Framework Core 就提供了一个 LINQ 提供者，这是一个用于与数据库交互的对象/关系映射系统。Cosmos DB 云数据库（Microsoft Azure 的一个特性）也提供了一个 LINQ 提供者。还有在 .NET 反应式扩展 中描述的反应式扩展，提供了对数据实时流的 LINQ 支持。简而言之，LINQ 是.NET 中广泛支持的习语，并且它是可扩展的，因此你也会发现开源和其他第三方提供者。

本章大部分示例使用 LINQ to Objects。这部分是因为它避免了示例中的额外细节，例如数据库或服务连接，但还有一个更重要的原因。LINQ 的引入在 2007 年显著改变了我编写 C# 代码的方式，这完全是由于 LINQ to Objects。尽管 LINQ 的查询语法使它看起来主要是一种数据访问技术，但我发现它远比那更有价值。在任何对象集合上都可以使用 LINQ 的服务使其在代码的每个部分中都非常有用。

# 查询表达式

LINQ 最显著的特点是查询表达式语法。这并非最重要——我们稍后会看到，完全可以在不编写查询表达式的情况下有效地使用 LINQ。但是，对于许多类型的查询来说，这是一种非常自然的语法。

乍一看，查询表达式大致类似于关系数据库查询，但语法适用于任何 LINQ 提供程序。示例 10-1 展示了一个使用 LINQ to Objects 搜索特定`CultureInfo`对象的查询表达式。（`CultureInfo`对象提供了一组特定于文化的信息，如本地货币使用的符号、使用的语言等。有些系统称其为*区域设置*。）这个特定的查询查看了表示英文中所谓的小数点的字符。许多国家实际上使用逗号而不是句点，而在这些国家中，100,000 意味着将数字 100 写成三位小数；在英语文化中，我们通常将其写作 100.000\. 查询表达式搜索系统中所有已知的文化，并返回那些使用逗号作为小数分隔符的文化。

##### 示例 10-1\. LINQ 查询表达式

```cs
`IEnumerable``<``CultureInfo``>` `commaCultures` `=`
    `from` `culture` `in` `CultureInfo``.``GetCultures``(``CultureTypes``.``AllCultures``)`
    `where` `culture``.``NumberFormat``.``NumberDecimalSeparator` `=``=` `","`
    `select` `culture``;`

foreach (CultureInfo culture in commaCultures)
{
    Console.WriteLine(culture.Name);
}
```

在这个例子中，`foreach`循环显示了查询的结果。在我的系统上，这列出了 354 个文化的名称，表明大约 813 个可用文化中有一半使用逗号而不是小数点。当然，我完全可以不使用 LINQ 来轻松实现这一点。示例 10-2 将产生相同的结果。

##### 示例 10-2\. 非 LINQ 等效方法

```cs
CultureInfo[] allCultures = CultureInfo.GetCultures(CultureTypes.AllCultures);
foreach (CultureInfo culture in allCultures)
{
    if (culture.NumberFormat.NumberDecimalSeparator == ",")
    {
        Console.WriteLine(culture.Name);
    }
}
```

两个示例都有八行非空代码，尽管如果忽略只包含括号的行，示例 10-2 只包含四行，比示例 10-1 少两行。另一方面，如果我们计算语句，LINQ 示例只有三个，而循环示例有四个。因此很难有说服力地论证哪种方法比另一种更简单。

然而，示例 10-1 有一个显著的优势：决定选择哪些项的代码与决定如何处理这些项的代码有很好的分离。示例 10-2 把这两个问题搞混了：选择对象的代码一半在循环外部，一半在循环内部。

另一个区别是，示例 10-1 具有更声明性的风格：它关注我们想要的内容，而不是如何获得它。查询表达式描述了我们想要的项，而不强制规定以任何特定方式实现。对于这个非常简单的例子，这并不重要，但对于更复杂的例子，特别是在使用 LINQ 提供程序进行数据库访问时，允许提供程序自由决定如何执行查询非常有用。示例 10-2 的方法遍历`foreach`循环中的所有内容并选择它想要的项，如果我们在与数据库交互，这种做法通常是不好的——通常希望服务器来处理这种过滤工作。

示例 10-1 中的查询有三个部分。所有查询表达式都必须以 `from` 子句开头，该子句指定查询的源。在本例中，源是由 `CultureInfo` 类的 `GetCultures` 方法返回的 `CultureInfo[]` 类型的数组。除了为查询定义源外，`from` 子句还包含一个名称，在这里是 `culture`。这称为*范围变量*，我们可以在查询的其余部分中使用它来代表源中的单个项目。子句可以多次运行——在 示例 10-1 中的 `where` 子句会针对集合中的每个项运行一次，因此范围变量每次都会有不同的值。这与 `foreach` 循环中的迭代变量类似。事实上，`from` 子句的整体结构相似——我们有一个将代表集合中的项目的变量，然后是 `in` 关键字，然后是该变量将代表单个项的源。正如 `foreach` 循环中的迭代变量仅在循环内部有效一样，范围变量 `culture` 仅在此查询表达式内部有效。

###### 注意

尽管与 `foreach` 的类比有助于理解 LINQ 查询的意图，但不应该过于字面化。例如，并非所有的提供程序直接执行查询中的表达式。一些 LINQ 提供程序将查询表达式转换为数据库查询，在这种情况下，查询中各种表达式中的 C# 代码并不以传统意义上的方式运行。因此，虽然可以说范围变量表示源中的单个值，但并不总是可以说各个子句会针对其处理的每个项执行一次，并且范围值取该项的值。这在 示例 10-1 中是正确的，因为它使用的是 LINQ to Objects，但并非所有提供程序都是如此。

查询的第二部分在 示例 10-1 中是一个 `where` 子句。此子句是可选的，或者您也可以在一个查询中有多个。`where` 子句用于过滤结果，在本示例中说明，我只希望具有指示小数分隔符为逗号的 `NumberFormat` 的 `CultureInfo` 对象。

查询的最后部分是 `select` 子句。所有查询表达式都以 `select` 子句或 `group` 子句结尾。这确定查询的最终输出。这个示例表明，我们希望保留未被查询过滤掉的每个 `CultureInfo` 对象。在 示例 10-1 中展示查询结果的 `foreach` 循环仅使用 `Name` 属性，因此我可以编写一个仅提取该属性的查询。如 示例 10-3 所示，如果我这样做，还需要更改循环，因为生成的查询现在会生成字符串而不是 `CultureInfo` 对象。

##### 示例 10-3\. 在查询中提取单个属性

```cs
IEnumerable<string> commaCultures =
    from culture in CultureInfo.GetCultures(CultureTypes.AllCultures)
    where culture.NumberFormat.NumberDecimalSeparator == ","
    `select` `culture``.``Name``;`

foreach (string cultureName in commaCultures)
{
    Console.WriteLine(cultureName);
}
```

这引出了一个问题：一般来说，查询表达式有什么类型？在示例 10-1 中，`commaCultures`是`IEnumerable<CultureInfo>`；在示例 10-3 中，它是`IEnumerable<string>`。输出项的类型由查询的最后一个子句确定——`select`或者在某些情况下是`group`子句。然而，并非所有的查询表达式都会得到`IEnumerable<T>`。这取决于你使用的 LINQ 提供程序——我最终得到了`IEnumerable<T>`，因为我使用的是 LINQ to Objects。

###### 注意

在声明用于保存 LINQ 查询的变量时，通常使用`var`关键字是非常常见的。如果`select`子句生成匿名类型的实例，则这是必需的，因为没有办法编写结果查询类型的名称。即使不涉及匿名类型，`var`仍然被广泛使用，有两个原因。一个原因是一致性问题：一些人认为，因为你必须对某些 LINQ 查询使用`var`，所以应该对所有 LINQ 查询都使用它。另一个论点是，LINQ 查询类型通常具有冗长和丑陋的名称，而`var`可以使代码更清晰。在书籍布局的严格限制环境中，这可能是一个特别迫切的问题，因此在本章的许多示例中，我离开了我通常偏爱显式类型的偏好，而是使用了`var`来使事物更协调。

C#如何知道我想要使用 LINQ to Objects？这是因为我在`from`子句中使用了数组作为源。更普遍地说，当你指定任何`IEnumerable<T>`作为源时，LINQ to Objects 将被使用，除非存在更专门的提供程序。然而，这并不能真正解释 C#如何首先发现提供程序的存在以及如何在它们之间进行选择。要理解这一点，你需要知道编译器如何处理查询表达式。

## 查询表达式的扩展方式

编译器将所有的查询表达式转换成一个或多个方法调用。一旦这样做了，LINQ 提供程序通过与 C#用于任何其他方法调用相同的机制来选择。编译器没有任何内置的概念来定义什么是 LINQ 提供程序。它仅仅依赖于约定。示例 10-4 展示了编译器如何处理示例 10-3 中的查询表达式。

##### 示例 10-4\. 查询表达式的影响

```cs
IEnumerable<string> commaCultures =
    CultureInfo.GetCultures(CultureTypes.AllCultures)
    .Where(culture => culture.NumberFormat.NumberDecimalSeparator == ",")
    .Select(culture => culture.Name);
```

`Where`和`Select`方法是 LINQ 操作符的示例。LINQ 操作符实际上就是符合标准模式的方法。我稍后将在“标准 LINQ 操作符”中描述这些模式。

示例 10-4 中的代码都是一个语句，并且我正在链接方法调用——我在 `GetCultures` 的返回值上调用 `Where` 方法，并在 `Where` 的返回值上调用 `Select` 方法。格式看起来有点奇怪，但它太长了，无法一行展示；尽管这不是特别优雅，但我更喜欢在多行上拆分链式调用时在每一行的开头放置 `.`，因为这样更容易看出每一行是从上一行的哪里继续的。把句点留在前一行的末尾看起来更整洁，但也更容易误读代码。

编译器已将 `where` 和 `select` 子句的表达式转换为 lambda 表达式。注意，范围变量最终成为每个 lambda 的参数。这是一个例子，说明为什么不应过于字面地将查询表达式与 `foreach` 循环类比。与 `foreach` 迭代变量不同，范围变量并不存在作为单个常规变量。在查询中，它只是一个表示源中项的标识符，在扩展查询为方法调用时，C# 可能会为单个范围变量创建多个实际变量，就像在这里为两个分离的 lambda 的参数创建的那样。

所有查询表达式归结为这种形式——带有 lambda 的链式方法调用。（这就是为什么我们不严格需要查询表达式语法——你可以使用方法调用来写任何查询。）有些比其他的更复杂。尽管看起来几乎相同，示例 10-1 中的表达式最终具有更简单的结构。示例 10-5 展示了它是如何展开的。原来，当查询的 `select` 子句直接传递范围变量时，编译器解释为我们希望直接传递前一个子句的结果，而无需进一步处理，因此不会添加 `Select` 调用。（这有一个例外：如果编写一个查询表达式，其中只包含 `from` 和 `select` 子句，它将生成一个 `Select` 调用，即使 `select` 子句是微不足道的。）

##### 示例 10-5\. `select` 子句如何展开

```cs
IEnumerable<CultureInfo> commaCultures =
    CultureInfo.GetCultures(CultureTypes.AllCultures)
    .Where(culture => culture.NumberFormat.NumberDecimalSeparator == ",");
```

如果在查询的范围内引入多个变量，编译器将需要更多工作。可以使用 `let` 子句来实现这一点。示例 10-6 执行与 示例 10-3 相同的作业，但我引入了一个名为 `numFormat` 的新变量来引用数字格式。这使得我的 `where` 子句更短更易读，在需要多次引用该格式对象的更复杂查询中，这种技术可以减少很多混乱。

##### 示例 10-6\. 带有 `let` 子句的查询

```cs
IEnumerable<string> commaCultures =
    from culture in CultureInfo.GetCultures(CultureTypes.AllCultures)
    `let` `numFormat` `=` `culture``.``NumberFormat`
    where numFormat.NumberDecimalSeparator == ","
    select culture.Name;
```

当你编写引入额外变量的查询时，编译器会自动生成一个隐藏类，为每个变量创建一个字段，以便在每个阶段都能使它们可用。为了在普通的方法调用中达到同样的效果，我们需要做类似的事情，引入一个匿名类型来包含它们，就像 示例 10-7 中展示的那样。

##### 示例 10-7\. 多变量查询表达式如何扩展（近似）

```cs
IEnumerable<string> commaCultures =
    CultureInfo.GetCultures(CultureTypes.AllCultures)
    .Select(culture => new { culture, numFormat = culture.NumberFormat })
    .Where(vars => vars.numFormat.NumberDecimalSeparator == ",")
    .Select(vars => vars.culture.Name);
```

无论查询表达式多么简单或复杂，它们都不过是方法调用的一种特殊语法。这表明了我们编写自定义查询表达式源的方法。

## 支持查询表达式

因为 C# 编译器只是将查询表达式的各个子句转换为方法调用，所以我们可以编写一个类型来参与这些表达式，定义一些合适的方法。为了说明 C# 编译器实际上并不关心这些方法做了什么，示例 10-8 展示了一个完全没有意义的类，但在从查询表达式中使用时，仍然能让 C# 保持愉快。编译器只是机械地将查询表达式转换为一系列方法调用，因此如果存在合适的方法，代码将成功编译。

##### 示例 10-8\. 毫无意义的 `Where` 和 `Select`

```cs
public class SillyLinqProvider
{
    public SillyLinqProvider Where(Func<string, int> pred)
    {
        Console.WriteLine("Where invoked");
        return this;
    }

    public string Select<T>(Func<DateTime, T> map)
    {
        Console.WriteLine($"Select invoked, with type argument {typeof(T)}");
        return "This operator makes no sense";
    }
}
```

我可以使用这个类的实例作为查询表达式的源。这太疯狂了，因为这个类根本不代表数据的集合，但编译器不关心。它只需要某些方法存在，所以如果我在 示例 10-9 中编写代码，尽管代码毫无意义，编译器仍会完全满意。

##### 示例 10-9\. 一个无意义的查询

```cs
var q = from x in new SillyLinqProvider()
        where int.Parse(x)
        select x.Hour;
```

编译器将这些内容转换为方法调用的方式与示例 10-1 中更合理的查询完全相同。示例 10-10 展示了结果。如果你注意到了，你会发现我的范围变量在中间实际上改变了类型——我的 `Where` 方法需要一个接受字符串的委托，所以在第一个 Lambda 中，`x` 的类型是 `string`。但是我的 `Select` 方法要求它的委托接受一个 `DateTime`，所以在那个 Lambda 中，`x` 的类型就是 `DateTime`。 （而这些都不重要，因为我的 `Where` 和 `Select` 方法甚至根本不使用这些 Lambda。）再次强调，这是无意义的，但它展示了 C# 编译器如何机械地将查询转换为方法调用。

##### 示例 10-10\. 编译器如何转换这个无意义的查询

```cs
var q = new SillyLinqProvider().Where(x => int.Parse(x)).Select(x => x.Hour);
```

显然，编写毫无意义的代码是没有用的。我展示这个的原因是为了演示查询表达式语法不了解语义——编译器对其调用的任何方法都没有特定的期望。它所要求的只是它们接受 lambda 作为参数并返回非`void`。

明显，真正的工作是在别处进行的。正是 LINQ 提供程序自己使事情发生。现在我将概述，如果没有 LINQ to Objects，我们需要编写什么来使我在前面的几个示例中显示的查询工作。

你已经看到了 LINQ 查询如何转换为像示例 10-4 中显示的代码，但这并不是全部故事。`where` 子句变成了对 `Where` 方法的调用，但我们是在 `CultureInfo[]` 类型的数组上调用它，这种类型实际上没有 `Where` 方法。这只能工作，因为 LINQ to Objects 定义了一个适当的扩展方法。就像我在第 3 章中展示的那样，可以向现有类型添加新方法，LINQ to Objects 就是为 `IEnumerable<T>` 定义这些方法的。（由于大多数集合实现了 `IEnumerable<T>`，这意味着几乎可以在任何类型的集合上使用 LINQ to Objects。）要使用这些扩展方法，您需要为 `System.Linq` 命名空间添加一个 `using` 指令；在 .NET 6.0 中，新创建的项目启用了*隐式全局 using*功能（在“命名空间”中描述），它会自动生成适合的全局 `using` 指令，因此，除非您禁用了该功能，或者您的项目是在 .NET 6.0 之前创建的并且之后未启用该设置，否则您不需要自己编写指令。（顺便说一下，所有这些扩展方法都由该命名空间中名为 `Enumerable` 的静态类定义。）如果您尝试在没有该指令的情况下使用 LINQ，编译器会为 示例 10-1 或 示例 10-3 的查询表达式生成以下错误：

```cs
error CS1935: Could not find an implementation of the query pattern for source
type 'CultureInfo[]'.  'Where' not found.  Are you missing required assembly
references or a using directive for 'System.Linq'?
```

一般来说，该错误消息的建议可能很有帮助，但在这种情况下，我想编写自己的 LINQ 实现。示例 10-11 就是这样做的，我展示了整个源文件，因为扩展方法对命名空间和`using`指令的使用很敏感。（如果你下载这些示例，你会发现我没有为这个特定项目启用隐式全局`using`，这样就完全清楚发生了什么。）`Main`方法的内容应该看起来很熟悉——这类似于示例 10-3，但这一次，它将使用我的`CustomLinqProvider`类的扩展方法，而不是使用 LINQ 到对象提供程序。（通常情况下，你可以通过`using`指令使扩展方法可用，但由于`CustomLinqProvider`与`Program`类在同一个命名空间中，它的所有扩展方法都会自动对`Main`可用。）

###### 警告

虽然示例 10-11 表现如预期，但你不应将其视为 LINQ 提供程序通常执行其查询的示例。这确实展示了 LINQ 提供程序如何置身事外，但正如我稍后将展示的那样，这段代码在执行查询时存在一些问题。而且，它相当简约——LINQ 不仅仅是`Where`和`Select`，大多数真实的提供程序提供的不仅仅是这两个操作符。

##### 示例 10-11\. 一个用于`CultureInfo[]`的自定义 LINQ 提供程序

```cs
using System;
using System.Globalization;

namespace CustomLinqExample;

public static class CustomLinqProvider
{
    public static CultureInfo[] Where(this CultureInfo[] cultures,
                                        Predicate<CultureInfo> filter)
    {
        return Array.FindAll(cultures, filter);
    }

    public static T[] Select<T>(this CultureInfo[] cultures,
                                Func<CultureInfo, T> map)
    {
        var result = new T[cultures.Length];
        for (int i = 0; i < cultures.Length; ++i)
        {
            result[i] = map(cultures[i]);
        }
        return result;
    }
}

class Program
{
    static void Main(string[] args)
    {
        var commaCultures =
            from culture in CultureInfo.GetCultures(CultureTypes.AllCultures)
            where culture.NumberFormat.NumberDecimalSeparator == ","
            select culture.Name;

        foreach (string cultureName in commaCultures)
        {
            Console.WriteLine(cultureName);
        }
    }
}
```

正如你现在很清楚的那样，在`Main`中的查询表达式将首先在源上调用`Where`，然后在`Where`的返回值上调用`Select`。与之前一样，源是`GetCultures`的返回值，它是一个`CultureInfo[]`类型的数组。这是`CustomLinqProvider`定义的扩展方法的类型，因此这将调用`CustomLinqProvider.Where`。它使用`Array`类的`FindAll`方法来查找源数组中与谓词匹配的所有元素。`Where`方法将自己的参数直接传递给`FindAll`作为谓词，正如你所知道的，当 C#编译器调用`Where`时，它会传递一个基于 LINQ 查询中`where`子句表达式的 lambda 表达式。该谓词将匹配使用逗号作为其小数分隔符的区域设置，因此`Where`子句返回一个仅包含这些区域设置的`CultureInfo[]`类型的数组。

接下来，编译器为查询创建的代码将在`Where`返回的`CultureInfo[]`数组上调用`Select`。数组没有`Select`方法，因此将使用`CustomLinqProvider`中的扩展方法。我的`Select`方法是泛型的，因此编译器将需要推断类型参数，它可以从`select`子句中的表达式中推断出来。

首先，编译器将其转换为一个 lambda 表达式：`culture => culture.Name`。因为这成为 `Select` 的第二个参数，编译器知道我们需要一个 `Func<CultureInfo, T>`，因此它知道 `culture` 参数必须是 `CultureInfo` 类型。这使它能够推断 `T` 必须是 `string`，因为 lambda 返回 `culture.Name`，而 `Name` 属性的类型是 `string`。所以编译器知道它正在调用 `CustomLinqProvider.Select<string>`。（顺便说一句，我刚刚描述的推断并不特定于查询表达式。类型推断发生在查询被转换为方法调用之后。如果我们从 [Example 10-4](https://wiki.example.org/the_effect_of_a_query_expression) 中的代码开始，编译器将经历完全相同的过程。）

现在，`Select` 方法将生成一个 `string[]` 类型的数组（因为这里的 `T` 是 `string`）。它通过迭代传入的 `CultureInfo[]` 中的元素，将每个 `CultureInfo` 作为参数传递给提取 `Name` 属性的 lambda 表达式来填充该数组。因此，我们最终得到一个包含每个使用逗号作为其十进制分隔符的文化的名称的字符串数组。

这个例子比我的 `SillyLinqProvider` 稍微现实一些，因为它现在提供了预期的行为。然而，虽然查询产生了与使用真正的 LINQ to Objects 提供程序时相同的字符串，但它所采用的机制有些不同。我的 `CustomLinqProvider` 立即执行了每个操作——`Where` 和 `Select` 方法都返回完全填充的数组。LINQ to Objects 做了完全不同的事情。实际上，大多数 LINQ 提供程序也是如此。

# 延迟评估

如果 LINQ to Objects 的工作方式与我在 [Example 10-11](https://wiki.example.org/a_custom_linq_provider_for_cultureinfo) 中的自定义提供程序相同，它将无法很好地处理 [Example 10-12](https://wiki.example.org/query_with_an_infinite_source_sequence)。这有一个 `Fibonacci` 方法，返回一个永无止境的序列——只要代码继续请求，它将继续提供斐波那契数列中的数字。我已经使用这个方法返回的 `IEnumerable<BigInteger>` 作为查询表达式的源。由于我们在开头附近放置了 `System.Linq` 的 `using` 指令，我现在回到使用 LINQ to Objects。 （在可下载的示例中，我已禁用了该项目的隐式全局 `using` 指令，以清楚地了解使用了哪些命名空间。）

##### Example 10-12\. 具有无限源序列的查询

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Numerics;

static IEnumerable<BigInteger> Fibonacci()
{
    BigInteger n1 = 1;
    BigInteger n2 = 1;
    yield return n1;
    while (true)
    {
        yield return n2;
        BigInteger t = n1 + n2;
        n1 = n2;
        n2 = t;
    }
}

var evenFib = from n in Fibonacci()
              where n % 2 == 0
              select n;

foreach (BigInteger n in evenFib)
{
    Console.WriteLine(n);
}
```

这里将使用 LINQ to Objects 为 `IEnumerable<T>` 提供的 `Where` 扩展方法。如果它的工作方式与我在 示例 10-11 中为 `CultureInfo[]` 编写的 `CustomLinqExtension` 类的 `Where` 方法相同，那么这个程序将永远无法显示单个数字。我的 `Where` 方法在未过滤其整个输入并生成完全填充的数组作为输出之前不会返回。如果 LINQ to Objects 的 `Where` 方法尝试对我的无限斐波那契枚举器进行相同的操作，它将永远无法完成。

实际上，示例 10-12 运行完美——它产生了一系列由 2 整除的斐波那契数输出。这意味着在我们调用 `Where` 时它不会尝试执行所有的过滤工作。相反，它的 `Where` 方法返回一个 `IEnumerable<T>`，按需过滤项目。它不会尝试从输入序列获取任何内容，直到某些东西要求一个值，此时它将从源中一个接一个地检索值，直到过滤委托说找到匹配项。然后它会生成该匹配项，并在被要求下一个项目之前不会再从源中检索任何内容。示例 10-13 展示了如何利用 C# 的 `yield return` 特性实现这种行为。

##### 示例 10-13\. 自定义延迟 `Where` 操作符

```cs
public static class CustomDeferredLinqProvider
{
    public static IEnumerable<T> Where<T>(this IEnumerable<T> src,
                                          Func<T, bool> filter)
    {
        foreach (T item in src)
        {
            if (filter(item))
            {
                yield return item;
            }
        }
    }
}
```

真正的 LINQ to Objects 的 `Where` 实现要复杂一些。它检测到某些特殊情况，比如数组和列表，并以略微比通用实现更高效的方式处理它们。然而，对于 `Where` 和所有其他操作符来说，原则都是一样的：这些方法不执行指定的工作。相反，它们返回将按需执行工作的对象。只有在尝试检索查询结果时才会真正发生任何事情。这被称为*延迟评估*，有时也称为*惰性评估*。

延迟评估的好处是在需要时才执行工作，并且可以处理无限序列。然而，它也有缺点。您可能需要小心避免多次评估查询。示例 10-14 犯了这个错误，导致执行比必要做的工作多得多。这段代码循环遍历几个不同的数字，并使用每个使用逗号作为小数分隔符的文化的货币格式写出每一个数字。

###### 注意

如果您在 Windows 上运行此代码，则可能会发现大多数行显示的是包含`?`字符的行，表示控制台无法显示大多数货币符号。实际上，它可以——只是需要权限。默认情况下，Windows 控制台出于向后兼容性的原因使用 8 位代码页。如果您从命令提示符运行命令`chcp 65001`，它将将该控制台窗口切换到 UTF-8 代码页，从而使其能够显示您选择的控制台字体支持的任何 Unicode 字符。您可能希望配置控制台使用具有对不常见字符全面支持的字体——例如 Consolas 或 Lucida Console，以充分利用它。

##### 示例 10-14\. 延迟查询的意外重新评估

```cs
var commaCultures =
    from culture in CultureInfo.GetCultures(CultureTypes.AllCultures)
    where culture.NumberFormat.NumberDecimalSeparator == ","
    select culture;

object[] numbers = { 1, 100, 100.2, 10000.2 };

foreach (object number in numbers)
{
    foreach (CultureInfo culture in commaCultures)
    {
        Console.WriteLine(string.Format(culture, "{0}: {1:c}",
                          culture.Name, number));
    }
}
```

此代码的问题在于，即使`commaCultures`变量在数字循环外部初始化，我们也会为每个数字迭代它。由于 LINQ to Objects 使用延迟评估，这意味着每次外部循环周围都会重新执行查询的实际工作。因此，而不是为每种文化（在我的系统上为 813 次）评估一次`where`子句，它最终为每种文化执行四次（3,252 次），因为整个查询针对`numbers`数组的四个项之一每次都会评估。这并不是灾难性的——代码仍然能正常工作。但是，如果您在运行在负载较重的服务器上的程序中执行此操作，它将损害您的吞吐量。

如果您知道您将需要多次迭代查询的结果，请考虑使用 LINQ to Objects 提供的`ToList`或`ToArray`扩展方法之一。这些方法立即评估整个查询一次，分别生成`IList<T>`或`T[]`数组（因此显然不应在无限序列上使用这些方法）。然后，您可以随意多次迭代它，而不会产生进一步的成本（除了读取数组或列表元素所固有的最小成本）。但在只迭代一次查询的情况下，通常最好不要使用这些方法，因为它们将比必要的消耗更多内存。

# LINQ、泛型和 IQueryable<T>

大多数 LINQ 提供程序使用泛型类型。虽然没有强制要求这样做，但这种做法非常普遍。LINQ to Objects 使用`IEnumerable<T>`。几个数据库提供程序使用称为`IQueryable<T>`的类型。更广泛地说，模式是具有某种泛型类型`*Source*<T>`，其中`*Source*`表示某些项目的来源，而`T`是单个项目的类型。具有 LINQ 支持的源类型使得操作方法在`*Source*<T>`上对任何`T`都可用，并且这些操作符通常还返回`*Source*<TResult>`，其中`TResult`可能与`T`不同。

`IQueryable<T>`很有趣，因为它设计用于多个提供程序使用。在示例 10-15 中显示了这个接口、它的基本`IQueryable`和相关的`IQueryProvider`。

##### 示例 10-15\. `IQueryable` 和 `IQueryable<T>`

```cs
public interface IQueryable : IEnumerable
{
    Type ElementType { get; }
    Expression Expression { get; }
    IQueryProvider Provider { get; }
}

public interface IQueryable<out T> : IEnumerable<T>, IQueryable
{
}

public interface IQueryProvider
{
    IQueryable CreateQuery(Expression expression);
    IQueryable<TElement> CreateQuery<TElement>(Expression expression);
    object? Execute(Expression expression);
    TResult Execute<TResult>(Expression expression);
}
```

`IQueryable<T>` 最明显的特点是，它不向其基类添加任何成员。这是因为它完全通过扩展方法来使用。`Sys⁠tem.​Li⁠nq` 命名空间定义了所有标准 LINQ 操作符，这些操作符是由 `Queryable` 类提供的扩展方法，适用于 `IQueryable<T>`。然而，所有这些操作符都简单地推迟到由 `IQueryable` 基类定义的 `Provider` 属性。因此，与 LINQ to Objects 不同，在那里，`IEnumerable<T>` 上的扩展方法定义了行为，`IQueryable<T>` 的实现能够决定如何处理查询，因为它可以提供执行实际工作的 `IQueryProvider`。

然而，所有基于 `IQueryable<T>` 的 LINQ 提供程序有一个共同点：它们将 lambda 解释为表达式对象，而不是委托。Example 10-16 展示了为 `IEnumerable<T>` 和 `IQueryable<T>` 定义的 `Where` 扩展方法的声明。比较 `predicate` 参数。

##### 示例 10-16\. `Enumerable` versus `Queryable`

```cs
public static class Enumerable
{
    public static IEnumerable<TSource> Where<TSource>(
        this IEnumerable<TSource> source,
        `Func``<``TSource``,` `bool``>` `predicate``)`
    ...
}

public static class Queryable
{
    public static IQueryable<TSource> Where<TSource>(
        this IQueryable<TSource> source,
        `Expression``<``Func``<``TSource``,` `bool``>``>` `predicate``)`
    ...
}
```

`IEnumerable<T>` 的 `Where` 扩展方法（LINQ to Objects）接受 `Func<TSource, bool>`，如您在 Chapter 9 中所见，这是一种委托类型。但是 `IQueryable<T>` 的 `Where` 扩展方法（许多 LINQ 提供程序使用）接受 `Exp⁠res⁠sion​<Fu⁠nc<T⁠Sou⁠rce,⁠ bool>>`，正如您在 Chapter 9 中也看到的，这会导致编译器构建表达式的对象模型并将其作为参数传递。

如果 LINQ 提供程序需要这些表达式树，通常会使用 `IQueryable<T>`。这通常是因为它将检查您的查询并将其转换为其他形式，例如 SQL 查询。

在 LINQ 中还有一些其他常见的泛型类型。一些 LINQ 特性保证按特定顺序生成项目，而另一些则不保证。更微妙的是，一些运算符生成的项目顺序取决于其输入的顺序。这可以反映在定义运算符的类型和它们返回的类型中。LINQ to Objects 定义了 `IOrderedEnumerable<T>` 来表示有序数据，而对于基于 `IQueryable<T>` 的提供程序，则有相应的 `IOrderedQueryable<T>` 类型。（使用自己类型的提供程序往往会做类似的事情——例如，Parallel LINQ 在 Chapter 16 中定义了 `Ord⁠ered​Par⁠all⁠elQ⁠uery<T>`。）这些接口从它们的无序对应接口派生，如 `IEnumerable<T>` 和 `IQueryable<T>`，因此所有常见的运算符都是可用的，但它们使得定义需要考虑其输入顺序的运算符或其他方法成为可能。例如，在 “Ordering” 中，我将展示一个称为 `ThenBy` 的 LINQ 运算符，该运算符仅在已经有序的源上可用。

在查看 LINQ to Objects 时，有序/无序的区分可能看起来是不必要的，因为 `IEnumerable<T>` 总是以某种顺序生成项目。但某些提供程序并不一定以任何特定的顺序进行操作，也许是因为它们并行执行查询，或者因为它们让数据库为它们执行查询，并且在某些情况下，数据库保留在启用更有效地工作时干预顺序的权利。

# 标准 LINQ 操作符

在本节中，我将描述 LINQ 提供程序可以提供的标准操作符。在适用的情况下，我还将描述查询表达式的等效形式，尽管许多操作符没有相应的查询表达式形式。一些 LINQ 功能只能通过显式方法调用来使用。即使在某些可以在查询表达式中使用的操作符中也是如此，因为大多数操作符都是重载的，而查询表达式无法使用一些更高级的重载形式。

###### 注意

LINQ 操作符不是通常 C# 中的符号运算符，它们不是如 `+` 或 `&&` 的符号。LINQ 有自己的术语，对于本章而言，操作符是 LINQ 提供程序提供的查询功能。在 C# 中，它看起来像是一个方法。

所有这些操作符都有一个共同点：它们都设计用于支持组合。这意味着您几乎可以以任何方式组合它们，从而能够从简单元素构建复杂查询。为了实现这一点，操作符不仅接受某种类型的项目集合（例如 `IEnumerable<T>`）作为它们的输入，而且大多数操作符还返回某种代表项目集合的结果。如前所述，项目类型并不总是相同的——一个操作符可能会以某种 `IEnumerable<T>` 作为输入，并生成 `IEnumerable<TResult>` 作为输出，其中 `TResult` 不必与 `T` 相同。尽管如此，您仍然可以以任意数量的方式将它们链接在一起。这样做的部分原因是 LINQ 操作符类似于数学函数，它们不会修改它们的输入；相反，它们会基于它们的操作数产生一个新的结果。 （函数式编程语言通常具有相同的特征。）这意味着不仅您可以自由地以任意组合方式连接操作符，而且您还可以自由地将同一源用作多个查询的输入，因为没有任何 LINQ 查询会修改其输入。每个操作符都返回基于其输入的新查询。

没有任何东西强制执行这种函数式风格。就像您在我的 `SillyLinqProvider` 中看到的那样，编译器并不关心表示 LINQ 操作符的方法做了什么。但是，约定是操作符应该是函数式的，以支持组合。内置的 LINQ 提供程序都是这样工作的。

并非所有提供商都全面支持所有操作符。微软提供的主要支持包括 LINQ to Objects 或 Entity Framework Core 和 Rx 中的 LINQ 支持，尽可能全面，但某些情况下，某些操作符可能无意义。

为了演示操作符的作用，我需要一些源数据。以下部分的许多示例将使用 示例 10-17 中的代码。

##### 示例 10-17\. LINQ 查询的示例输入数据

```cs
public record Course(
    string Title,
    string Category,
    int Number,
    DateOnly PublicationDate,
    TimeSpan Duration)
{
    public static readonly Course[] Catalog =
    {
            new Course(
                Title: "Elements of Geometry",
                Category: "MAT", Number: 101, Duration: TimeSpan.FromHours(3),
                PublicationDate: new DateOnly(2009, 5, 20)),
            new Course(
                Title: "Squaring the Circle",
                Category: "MAT", Number: 102, Duration: TimeSpan.FromHours(7),
                PublicationDate: new DateOnly(2009, 4, 1)),
            new Course(
                Title: "Recreational Organ Transplantation",
                Category: "BIO", Number: 305, Duration: TimeSpan.FromHours(4),
                PublicationDate: new DateOnly(2002, 7, 19)),
            new Course(
                Title: "Hyperbolic Geometry",
                Category: "MAT", Number: 207, Duration: TimeSpan.FromHours(5),
                PublicationDate: new DateOnly(2007, 10, 5)),
            new Course(
                Title: "Oversimplified Data Structures for Demos",
                Category: "CSE", Number: 104, Duration: TimeSpan.FromHours(2),
                PublicationDate: new DateOnly(2021, 11, 8)),
            new Course(
                Title: "Introduction to Human Anatomy and Physiology",
                Category: "BIO", Number: 201, Duration: TimeSpan.FromHours(12),
                PublicationDate: new DateOnly(2001, 4, 11)),
        };
}
```

## 过滤器

最简单的操作符之一是 `Where`，它用于过滤其输入。你提供一个谓词，即一个接受单个项目并返回 `bool` 的函数。`Where` 返回一个表示输入中谓词为 true 的项目的对象。（从概念上讲，这与 `List<T>` 和数组类型上可用的 `FindAll` 方法非常相似，但使用延迟执行。）

正如你已经看到的，查询表达式使用 `where` 子句表示这一点。然而，`Where` 操作符有一种重载，提供了一个从查询表达式中无法访问的额外功能。你可以编写一个过滤器 lambda，它接受两个参数：输入中的一个项目和表示该项目在源中位置的索引。示例 10-18 使用此形式从输入中移除每隔一个数字，并且还移除长度小于三小时的课程。

##### 示例 10-18\. 带索引的 `Where` 操作符

```cs
IEnumerable<Course> q = Course.Catalog.Where(
    (course, index) => (index % 2 == 0) && course.Duration.TotalHours >= 3);
```

带索引的过滤只对有序数据有意义。它在 LINQ to Objects 中总是有效，因为它使用的是 `IEnumerable<T>`，会一个接一个地生成项目，但并非所有 LINQ 提供程序都按顺序处理项目。例如，使用 Entity Framework Core (EF Core)，你在 C# 中编写的 LINQ 查询将在数据库上处理。除非查询明确要求特定顺序，否则数据库通常可以自由地按照它认为合适的顺序处理项目，甚至可能并行处理。在某些情况下，数据库可能有优化策略，使其能够使用与原始查询极为不同的过程生成查询所需的结果。因此，说“由 `WHERE` 子句处理的第 14 个项目”可能并无意义。因此，如果你像 示例 10-18 类似的查询在 EF Core 中执行，将会引发异常，指出索引的 `Where` 操作符不适用。如果你想知道为什么提供程序不支持却还存在重载，因为 EF Core 使用 `IQueryable<T>`，所以所有标准操作符在编译时都是可用的；选择使用 `IQueryable<T>` 的提供程序只能在运行时报告操作符的不可用性。

###### 注意

实现部分或全部查询逻辑在服务器端的 LINQ 提供程序通常限制您可以在查询的 Lambda 中执行的操作。相反，LINQ 到对象在进程中运行查询，因此允许您在过滤 Lambda 中调用任何方法—如果您想在谓词中调用`Console.WriteLine`或从文件中读取数据，LINQ 到对象无法阻止您。但是数据库提供程序仅提供非常有限的方法选择。这些提供程序需要能够将您的 Lambda 表达式转换为服务器可以处理的内容，并且会拒绝尝试调用没有服务器端等效方法的表达式。

尽管如此，您可能期望异常在调用`Where`时出现，而不是在尝试执行查询时（即当您首次尝试检索一个或多个项目时）。然而，将 LINQ 查询转换为其他形式（例如 SQL 查询）的提供程序通常会推迟所有验证直到您执行查询。这是因为某些操作符可能仅在特定情况下有效，这意味着提供程序可能不知道任何特定操作符是否有效，直到您完成整个查询的构建。如果由于不可行查询而导致的错误有时在构建查询时出现，有时在执行时出现，这将是不一致的，因此即使在提供程序可以较早确定特定操作符将失败的情况下，通常也会等到执行查询时才告知您。

您提供给`Where`操作符的过滤 Lambda 必须接受项目类型的参数（例如`IEnumerable<T>`中的`T`），并且必须返回`bool`类型。您可能还记得第九章中运行时库定义了一个称为`Predicate<T>`的适当委托类型，但我在该章节中还提到 LINQ 避免使用它，现在我们可以看到原因了。`Where`操作符的索引版本不能使用`Predicate<T>`，因为有额外的参数，因此该重载使用`Func<T, int, bool>`。没有什么阻止非索引形式的`Where`使用`Predicate<T>`，但 LINQ 提供程序倾向于全面使用`Func`以确保具有类似意义的操作符具有类似的签名。因此，大多数提供程序使用`Func<T, bool>`，以与索引版本保持一致。（C#不在乎您使用哪个—如果提供程序使用`Predicate<T>`，查询表达式仍然有效，如我在示例 10-11 中展示的自定义`Where`操作符，但微软的提供程序没有这样做。）

###### 警告

C# 编译器的空值分析并不理解 LINQ 运算符。如果你有一个 `IEnumerable<string?>`，你可以写 `xs.Where(s => s is not null)` 来移除任何空值，但是 `Where` 仍然会返回一个 `IEnumerable<string?>`。编译器对 `Where` 的行为没有预期，因此它不理解输出实际上是一个 `IEnumerable<string>`。可以说让编译器做这样的推断可能是一个错误：就像 示例 10-8 中展示的那样，可以提供一个违反预期的 `Where`。

LINQ 定义了另一个过滤运算符：`OfType<T>`。如果您的源包含不同类型的项目混合——可能源是 `IEnumerable<object>`，您想将其过滤为仅为 `string` 类型的项目。 示例 10-19 展示了 `OfType<T>` 运算符如何实现这一点。

##### 示例 10-19\. `OfType<T>` 运算符

```cs
static void ShowAllStrings(IEnumerable<object> src)
{
    foreach (string s in src.OfType<string>())
    {
        Console.WriteLine(s);
    }
}
```

当你使用 `OfType<T>` 运算符与引用类型一起使用时，它将过滤掉任何 `null` 值。如果你启用了可空引用类型，`OfType` 避免了 `Where(s => s is not null)` 遇到的问题：如果你在 `IEnumerable<string?>` 上调用 `OfType<string>`，结果类型将是 `IEnumerable<string>`。但这并不是因为 `OfType` 被设计时考虑了可空引用类型。相反，它在使用引用类型作为类型参数时有效地忽略了空值。它之所以在这种情况下做我们想要的事情，是因为它总是寻找积极的匹配（它实际上执行与 `o is string` 类似的测试）。令人惊讶的推论是 `OfType<string?>` 也会过滤掉 `null` 项，稍微奇怪的是，它返回一个 `IEnumerable<string?>`，但永远不会产生 `null`。

如果源中没有任何对象符合要求，`Where` 和 `OfType<T>` 都会产生空序列。这不被视为错误——在 LINQ 中，空序列非常正常。许多运算符可以产生它们作为输出，大多数运算符可以处理它们作为输入。

## Select

在编写查询时，我们可能只想从源项目中提取特定的数据片段。大多数查询末尾的 `select` 子句允许我们提供一个 lambda 表达式，用于生成最终的输出项，我们可能希望使我们的 `select` 子句不仅仅是直接传递每个项。我们可能只想从每个项中挑选一个特定的信息片段，或者我们可能希望将其转换为完全不同的东西。

您已经看到了几个 `select` 子句，并且我在 Example 10-3 中展示了编译器如何将它们转换为对 `Select` 的调用。然而，与许多 LINQ 操作符一样，通过查询表达式访问的版本并不是唯一的选择。还有另一种重载形式，不仅提供用于生成输出项的输入项，还提供该项的索引。Example 10-20 使用这一点生成了一个课程标题的编号列表。

##### 示例 10-20\. 带有索引的 `Select` 操作符

```cs
IEnumerable<string> nonIntro = Course.Catalog.Select((course, index) =>
      $"Course {index}: {course.Title}");
```

请注意，传入 lambda 表达式的从零开始的索引将基于进入 `Select` 操作符的内容，并且不一定代表底层数据源中项的原始位置。这可能不会产生您在诸如 Example 10-21 中编写的代码中所期望的结果。

##### 示例 10-21\. `Where` 操作符下游的索引化 `Select`

```cs
IEnumerable<string> nonIntro = Course.Catalog
    .Where(c => c.Number >= 200)
    .Select((course, index) => $"Course {index}: {course.Title}");
```

此代码将选择在`Course.Catalog`数组中分别位于索引 2、3 和 5 的课程，因为这些课程的`Number`属性满足`Where`表达式。然而，此查询将会将这三门课程编号为 0、1 和 2，因为`Select`操作符只能看到`Where`子句允许通过的项目。就它而言，只有三个项目，因为`Select`子句从未访问过原始来源。如果您希望相对于原始集合提取索引，您需要在`Where`子句上游进行提取，就像 Example 10-22 中展示的那样。

##### 示例 10-22\. `Where` 操作符上游的索引化 `Select`

```cs
IEnumerable<string> nonIntro = Course.Catalog
    `.``Select``(``(``course``,` `index``)` `=``>` `new` `{` `course``,` `index` `}``)`
    .Where(vars => vars.course.Number >= 200)
    .Select(vars => $"Course {vars.index}: {vars.course.Title}");
```

你可能会想为什么我在这里使用了匿名类型而不是元组。我可以用`(course, index)`替换`new { course, index }`，代码同样可以工作。 （甚至可能更有效，因为元组是值类型，而匿名类型是引用类型。元组在这里会给 GC 带来更少的工作）。然而，一般来说，在 LINQ 中元组并不总是有效的。轻量级元组语法是在 C# 7.0 引入的，因此在 C# 3.0 引入表达式树时它们并不存在。表达式对象模型尚未更新以支持此语言特性，因此如果您尝试在基于`IQueryable<T>`的 LINQ 提供程序中使用元组，您将收到编译器错误 CS8143，提示`An expression tree may not contain a tuple literal`。¹ 因此，在这一章中我倾向于使用匿名类型，因为它们与基于查询的提供程序兼容。但是，如果您使用的是纯本地 LINQ 提供程序（例如 Rx 或 LINQ 到对象），请随意使用元组。

索引化的 `Select` 操作符与索引化的 `Where` 操作符类似。因此，正如您可能期望的那样，并非所有的 LINQ 提供程序都在所有情况下都支持它。

### 数据塑形和匿名类型

如果你正在使用 LINQ 提供程序来访问数据库，`Select` 运算符可以提供一个减少获取数据量的机会，这可能会减少服务器的负载。当你使用像 EF Core 这样的数据访问技术来执行返回表示持久化实体集合的查询时，存在着一种在一开始做过多工作和需要执行大量额外延迟工作之间的权衡。这些框架是否应完全填充与各个数据库表中列对应的对象属性？它们是否还应加载相关对象？通常来说，不获取你不会使用的数据更有效率，而未在一开始获取的数据随时可以后续按需加载。然而，如果你在初始请求中过于节约，最终可能会导致需要大量额外请求来填补空白，这可能会抵消避免不必要工作带来的任何好处。

当涉及到相关实体时，EF Core 允许你配置哪些相关实体应预取，哪些应按需加载，但对于获取的任何特定实体，通常会完全填充与列相关的所有属性。这意味着请求整个实体的查询最终会获取它们所触及的任何行的所有列。

如果你只需要使用一两列，获取它们所有都是相对昂贵的。示例 10-23 使用了这种效率较低的方法。它展示了一个相当典型的 EF Core 查询。

##### 示例 10-23\. 获取比所需更多的数据

```cs
var pq = from product in dbCtx.Product
         where product.ListPrice > 3000
         select product;
foreach (var prod in pq)
{
    Console.WriteLine($"{prod.Name} ({prod.Size}): {prod.ListPrice}");
}
```

这个 LINQ 提供程序将 `where` 子句翻译为一个高效的 SQL 等价物。然而，SQL `SELECT` 子句从表中检索所有列。与 示例 10-24 对比一下。这只修改了查询的一部分：LINQ `select` 子句现在返回一个匿名类型的实例，该实例仅包含我们需要的那些属性。（随后的循环仍然可以保持不变。它使用 `var` 作为其迭代变量，这对于匿名类型来说可以正常工作，因为匿名类型提供了循环所需的三个属性。）

##### 示例 10-24\. 匿名类型的 `select` 子句

```cs
var pq = from product in dbCtx.Product
         where product.ListPrice > 3000
         `select` `new` `{` `product``.``Name``,` `product``.``ListPrice``,` `product``.``Size` `}``;`
```

代码产生了完全相同的结果，但生成了一个更加紧凑的 SQL 查询，仅请求`Name`、`ListPrice` 和 `Size` 列。如果你正在使用具有许多列的表，这将产生一个显著较小的响应，因为它不再被我们不需要的数据所主导。这减少了与数据库服务器的网络连接负载，并且由于数据到达时间更短，还会导致更快的处理。这种技术称为*数据整形*。

这种方法并不总是一种改进。首先，这意味着你直接在数据库中使用数据，而不是使用实体对象。这可能意味着你在抽象级别上工作的比使用实体类型时更低，这可能会增加开发成本。另外，在某些环境中，数据库管理员不允许使用即席查询，强制你使用存储过程，在这种情况下，你将无法使用这种技术来获得灵活性。

将查询结果投影到匿名类型并不限于数据库查询。你可以在任何 LINQ 提供程序（如 LINQ to Objects）中自由使用这个功能。有时这是一种在不需要专门定义类的情况下从查询中获取结构化信息的有用方式。（正如我在 第三章 中提到的，匿名类型可以在 LINQ 之外使用，但这是它们设计的主要场景之一。按复合键分组是另一个场景，我将在 “分组” 中描述。）

### 投影和映射

`Select` 操作符有时被称为*投影*，它与许多语言称为*映射*的操作相同，提供了一种略有不同的看待 `Select` 操作符的方式。到目前为止，我已经介绍了 `Select` 作为选择查询结果的一种方式，但你也可以把它看作是将变换应用到源中的每个项的一种方式。示例 10-25 使用 `Select` 生成修改后的数字列表。它分别将数字加倍、求平方，并将它们转换为字符串。

##### 示例 10-25\. 使用 `Select` 转换数字

```cs
int[] numbers = { 0, 1, 2, 3, 4, 5 };

IEnumerable<int> doubled = numbers.Select(x => 2 * x);
IEnumerable<int> squared = numbers.Select(x => x * x);
IEnumerable<string> numberText = numbers.Select(x => x.ToString());
```

## SelectMany

`SelectMany` LINQ 操作符用于具有多个 `from` 子句的查询表达式。它之所以被称为 `SelectMany`，是因为它不是为每个输入项选择单个输出项，而是为每个输入项提供一个生成整个集合的 lambda 表达式。生成的查询将来自所有这些集合的对象，就好像你的 lambda 返回的所有集合被合并成一个一样。（这不会移除重复项。在 LINQ 中，序列可以包含重复项。你可以使用 “集合操作” 中描述的 `Distinct` 操作符来移除它们。）有几种思考这个操作符的方式。一种是它提供了将两个层次结构（集合的集合）展平为单一级别的方法。另一种看待它的方式是作为笛卡尔积——即从一些输入集合中生成每一种可能的组合的方法。

示例 10-26 展示了如何在查询表达式中使用此运算符。此代码突出显示了类似于笛卡尔积的行为。它显示了字母 A、B 和 C 与数字 1 到 5 的每个组合，即 A1、B1、C1、A2、B2、C2 等（如果您对这两个输入序列的表现不兼容感到疑惑，此查询的`select`子句依赖于一个事实，即如果您使用`+`运算符将一个`string`和某种其他类型相加，C#会为您生成调用非 string 操作数的`ToString`的代码）。

##### 示例 10-26\. 使用查询表达式中的`SelectMany`

```cs
int[] numbers = { 1, 2, 3, 4, 5 };
string[] letters = { "A", "B", "C" };

IEnumerable<string> combined = from number in numbers
                               from letter in letters
                               select letter + number;
foreach (string s in combined)
{
    Console.WriteLine(s);
}
```

示例 10-27 展示了如何直接调用运算符。这相当于示例 10-26 中的查询表达式。

##### 示例 10-27\. `SelectMany`运算符

```cs
IEnumerable<string> combined = numbers.SelectMany(
    number => letters,
    (number, letter) => letter + number);
```

示例 10-26 使用了两个固定集合——第二个`from`子句每次返回相同的`letters`集合。然而，您可以使第二个`from`子句中的表达式基于第一个`from`子句的当前项返回一个值。您可以在示例 10-27 中看到，`SelectMany`的第一个 Lambda 表达式（实际上对应第二个`from`子句的最终表达式）通过其`number`参数接收第一个集合的当前项，因此您可以用它来选择每个第一个集合项的不同集合。我可以利用这一点来利用`SelectMany`的展平行为。

我已经从示例 5-16 在第 5 章中复制了一个嵌套数组到示例 10-28，然后使用包含两个`from`子句的查询处理它。请注意，第二个`from`子句中的表达式现在是`row`，即第一个`from`子句的范围变量。

##### 示例 10-28\. 展平嵌套数组

```cs
int[][] arrays =
{
    new[] { 1, 2 },
    new[] { 1, 2, 3, 4, 5, 6 },
    new[] { 1, 2, 4 },
    new[] { 1 },
    new[] { 1, 2, 3, 4, 5 }
};

IEnumerable<int> flattened = from row in arrays
                             from number in row
                             select number;
```

第一个`from`子句要求迭代顶层数组中的每个项。这些项中的每一个也是一个数组，第二个`from`子句要求迭代每个嵌套数组。这个嵌套数组的类型是`int[]`，因此第二个`from`子句的范围变量`number`表示来自该嵌套数组的一个`int`。`select`子句只是返回每个这些`int`值。

结果序列依次提供数组中的每个数字。它将嵌套数组展平为简单的线性数字序列。这种行为在概念上类似于编写一个嵌套的循环对，一个循环遍历外部`int[][]`数组，另一个内部循环遍历每个单独的`int[]`数组的内容。

编译器对于 示例 10-28 和 示例 10-27 使用相同的 `SelectMany` 重载，但在这种情况下存在另一种选择。在 示例 10-28 中，最终的 `select` 子句更简单—它仅传递来自第二个集合的项目，这意味着 示例 10-29 中显示的更简单的重载同样能胜任。使用这种重载，我们只需提供一个 lambda，它选择 `SelectMany` 将为输入集合中的每个项目扩展的集合。

##### 示例 10-29\. 没有项目投影的 `SelectMany`

```cs
var flattened = arrays.SelectMany(row => row);
```

这是一段略显简洁的代码，因此如果不太清楚它如何最终展平数组，示例 10-30 展示了如果必须自己编写的话，您可能如何为 `IEnumerable<T>` 实现 `SelectMany`。

##### 示例 10-30\. `SelectMany` 的一个实现

```cs
static IEnumerable<T2> MySelectMany<T, T2>(
    this IEnumerable<T> src, Func<T, IEnumerable<T2>> getInner)
{
    foreach (T itemFromOuterCollection in src)
    {
        IEnumerable<T2> innerCollection = getInner(itemFromOuterCollection);
        foreach (T2 itemFromInnerCollection in innerCollection)
        {
            yield return itemFromInnerCollection;
        }
    }
}
```

编译器为什么不使用在 示例 10-29 中展示的更简单的选项？C# 语言规范定义了查询表达式如何转换为方法调用，并且仅提到了 示例 10-26 中显示的重载。也许规范之所以没有提及更简单的重载，是为了减少 C# 对想要支持这种双 `from` 查询形式的类型的要求—您只需编写一个方法即可启用此语法。然而，.NET 的各种 LINQ 提供程序更为慷慨，为选择直接使用运算符的开发人员提供了这种更简单的重载。事实上，一些提供程序定义了另外两个重载版本：迄今为止我们看到的 `SelectMany` 的这两种形式也会将项目索引传递给第一个 lambda。当然，对于索引运算符，通常的警告仍然适用。

虽然 示例 10-30 给出了 LINQ to Objects 在 `SelectMany` 中的一个合理想法，但这并不是确切的实现。对于特殊情况，存在优化。此外，其他提供程序可能使用非常不同的策略。数据库通常内置支持笛卡尔积，因此某些提供程序可能会基于此实现 `SelectMany`。

## Chunking

而 `SelectMany` 将多个序列展平为一个，LINQ 的 `Chunk` 操作符（在 .NET 6.0 中添加）则朝相反方向工作，将单个序列转换为一系列固定大小的序列。在涉及到 I/O 的情况下，这可能更有效，因为写入数据到磁盘或通过网络发送数据通常具有固定的最低成本，这往往意味着写入或发送单个记录的成本仅比写入或发送 10 条记录稍微低一点。

示例 10-31 使用 `Range` 方法（稍后在 “序列生成” 中描述）创建了一个从 1 到 50 的数字序列，然后要求 `Chunk` 将其分成每个包含 15 个数字的块。虽然 `Range` 生成了一个 `IEnumerable<int>`—一系列 `int` 值—`Chunk` 返回了一个 *数组* 序列，类型为 `int[]`。

##### 示例 10-31\. 使用 `Chunk` 将序列分成批次

```cs
IEnumerable<int> lotsOfNumbers = Enumerable.Range(1, 50);

IEnumerable<int[]> chunked = lotsOfNumbers.Chunk(15);
foreach(int[] chunk in chunked)
{
    Console.WriteLine(
        $"Chunk (length {chunk.Length}): {String.Join(", ", chunk)}");
}
```

查看 示例 10-31 的输出，我们可以看到 `Chunk` 将所有数字按顺序分割成了块：

```cs
Chunk (length 15): 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15
Chunk (length 15): 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30
Chunk (length 15): 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45
Chunk (length 5): 46, 47, 48, 49, 50
```

在本例中，源序列的长度不是块大小的精确倍数。`Chunk` 通过使最后一个块更小来处理这个问题。

###### 注意

一些 LINQ 提供程序使用不同的名称来表示此操作符：`Buffer`。在约 10 年前，Rx 库在介绍这种类型的操作符时选择了这个名称。.NET 6.0 选择了 `Chunk` 这个名称，但在此之前编写的库通常遵循了 Rx 的先例，将其版本的这个操作符称为 `Buffer`。

## 排序

一般来说，LINQ 查询不保证按任何特定顺序生成项目，除非您明确定义所需的顺序。您可以在查询表达式中使用 `orderby` 子句来实现这一点。正如 示例 10-32 所示，您可以指定定义项目顺序及其方向的表达式—因此，这将生成按发布日期升序排列的课程集合。恰巧的是，默认情况下是 `ascending`，因此您可以省略该限定词而不改变含义。正如您可能已经猜到的那样，您可以指定 `descending` 来反转顺序。

##### 示例 10-32\. 带有 `orderby` 子句的查询表达式

```cs
var q = from course in Course.Catalog
        orderby course.PublicationDate ascending
        select course;
```

编译器将 示例 10-32 中的 `orderby` 子句转换为对 `OrderBy` 方法的调用，如果指定了 `descending` 排序顺序，它将使用 `OrderByDescending`。对于那些区分有序和无序集合的源类型，这些操作符返回有序类型（例如，LINQ to Objects 返回 `IOrderedEnumerable<T>`，而基于 `IQueryable<T>` 的提供程序返回 `IOrderedQueryable<T>`）。

###### 警告

对于 LINQ to Objects，这些操作符必须从输入中检索每个元素，然后才能生成任何输出元素。升序的 `OrderBy` 只有在找到最低项之后才能确定要返回哪个项，并且直到看到所有项之后才能确定最低项。它仍然使用延迟评估—直到您请求第一个项目时才会执行任何操作。但是一旦您请求了某些内容，它必须立即完成所有工作。某些提供程序可能对数据具有额外的知识，从而能够实现更有效的策略（例如，数据库可以使用索引按照所需的顺序返回值）。

LINQ to Objects 的`OrderBy`和`OrderByDescending`运算符各有两个重载，其中只有一个可以从查询表达式中使用。如果直接调用这些方法，您可以提供一个额外的`IComparer<TKey>`类型的参数，其中`TKey`是正在排序的表达式的类型。如果您基于`string`属性进行排序，这可能很重要，因为文本有几种不同的排序方式，您可能需要根据应用程序的区域设置选择其中一种，或者您可能希望指定一个与文化无关的排序方式，以确保在所有环境中保持一致性。

示例 10-32 中确定顺序的表达式非常简单—它只是从源项目中检索`PublicationDate`属性。如果您愿意，可以编写更复杂的表达式。如果您使用将 LINQ 查询转换为其他内容的提供程序，可能会有一些限制。如果查询在数据库上运行，您可能可以引用其他表—提供程序可能能够将诸如`product.ProductCategory.Name`之类的表达式转换为适当的连接。但是，您无法在该表达式中运行任何旧代码，因为它必须是数据库可以执行的内容。但是，LINQ to Objects 只是为每个对象调用一次表达式，因此您确实可以在其中放入任何代码。

您可能希望按多个标准进行排序。您*不应该*通过编写多个`orderby`子句来实现这一点。示例 10-33 犯了这个错误。

##### 示例 10-33\. 如何不应用多个排序标准

```cs
var q = from course in Course.Catalog
        orderby course.PublicationDate ascending
        orderby course.Duration descending // BAD! Could discard previous order
        select course;
```

此代码按发布日期和持续时间对项目进行排序，但是将其作为两个独立且不相关的步骤进行。第二个`orderby`子句仅保证结果将按照该子句中指定的顺序排列，并不保证保留有关元素原始顺序的任何信息。如果您实际上想要项目按发布日期顺序排列，并且具有相同发布日期的项目按持续时间降序排列，您需要编写示例 10-34 中的查询。

##### 示例 10-34\. 查询表达式中的多个排序标准

```cs
var q = from course in Course.Catalog
        orderby course.PublicationDate ascending, course.Duration descending
        select course;
```

LINQ 为这种多级排序定义了单独的运算符：`ThenBy`和`The⁠nBy​Des⁠cen⁠ding`。示例 10-35 展示了如何通过直接调用 LINQ 运算符来实现与示例 10-34 中查询表达式相同的效果。对于那些类型区分有序和无序集合的 LINQ 提供程序，`ThenBy`和`ThenByDescending`运算符仅在有序形式上可用，例如`IOrderedQueryable<T>`或`IOrderedEnumerable<T>`。如果您尝试直接在`Course.Catalog`上调用`ThenBy`，编译器将报告错误。

##### 示例 10-35\. 使用 LINQ 操作符进行多重排序标准

```cs
var q = Course.Catalog
    .OrderBy(course => course.PublicationDate)
    .ThenByDescending(course => course.Duration);
```

您会发现，即使您没有要求，一些 LINQ 操作符也会保留某些排序方面。例如，LINQ to Objects 通常会按照它们在输入中出现的顺序生成项目，除非您编写了一个引起顺序改变的查询。但这只是 LINQ to Objects 工作方式的副产品，您不应该普遍依赖它。事实上，即使在使用该特定 LINQ 提供程序时，您也应该查看文档，以了解您得到的顺序是有保证的还是仅仅是实现的偶然。在大多数情况下，如果您关心顺序，应该编写一个明确表达这一点的查询。

## 包含性测试

LINQ 定义了各种用于发现集合内容的标准操作符。一些提供程序可能能够实现这些操作符而无需检查每个项目。（例如，基于数据库的提供程序可能使用 `WHERE` 子句，数据库可以使用索引来评估，而无需查看每个元素。）但没有限制 - 您可以按自己的喜好使用这些操作符，以及提供程序发现是否可以利用快捷方式。

###### 注意

不同于大多数 LINQ 操作符，在大多数提供程序中，它们既不返回集合也不返回输入中的项目。它们通常只返回 `true` 或 `false`，或者在某些情况下返回一个计数。Rx 是一个显著的例外：它这些操作符的实现将 `bool` 或 `int` 包装在产生结果的单元素 `IObservable<T>` 中。它这样做是为了保持 Rx 中处理的响应式特性。

最简单的操作符是 `Contains`。您传递一个项目，一些提供程序（包括 LINQ to Objects）提供了一个重载，还接受一个 `IEqualityComparer<T>`，以便您可以自定义操作符如何确定源中的项目与指定项目是否相同。`Contains` 如果源包含指定的项目则返回 `true`，如果不包含则返回 `false`。如果您使用具有 `ICollection<T>` 实现的集合的单参数版本（其中包括所有 `IList<T>` 和 `ISet<T>` 实现），LINQ to Objects 将检测到这一点，并且其 `Contains` 的实现只是将其延迟到集合。如果您使用非 `ICollection<T>` 集合，或者提供自定义的相等性比较器，它将不得不检查集合中的每个项目。

如果你不是在寻找特定的值，而是想知道一个集合是否包含任何满足某些特定条件的值，你可以使用`Any`运算符。它接受一个谓词，并且在源中至少有一个项满足谓词时返回`true`。如果你想知道有多少项满足某些条件，你可以使用`Count`运算符。它也接受一个谓词，而不是返回`bool`，而是返回一个`int`。如果你正在处理非常大的集合，`int`的范围可能不够，这时你可以使用`LongCount`运算符，它返回一个 64 位的计数。（在大多数 LINQ to Objects 应用程序中，这可能过于复杂，但在集合存在于数据库中时可能很重要。）

`Any`、`Count`和`LongCount`运算符有些重载不接受任何参数。对于`Any`来说，这告诉你源序列是否至少包含一个元素，而对于`Count`和`LongCount`，这些重载告诉你源序列包含多少元素。

你应该警惕像`if (q.Count() > 0)`这样的代码。计算确切的计数可能需要评估整个源查询（在这种情况下是`q`），而且通常比简单地回答“它是空的吗？”更耗费功夫。如果`q`是 LINQ 查询，写成`if (q.Any())`可能更有效率。话虽如此，在 LINQ 之外，对于类似列表的集合来说，并不是这样，那里获取元素计数是廉价的，而且实际上可能比`Any`运算符更有效率。

有些情况下，你可能只希望在能够高效计算时才使用计数。（例如，用户界面可能希望在可以轻松确定的情况下显示可用项的总数，但在计算成本太高时可能选择不显示。）对于这些情况，.NET 6.0 添加了一个新的`TryGetNonEnumeratedCount`方法。它将在可以在不必迭代整个集合的情况下确定计数时返回`true`，否则返回`false`。当它返回`true`时，通过`out int`类型的单个参数将计数传递回去。

与`Any`运算符密切相关的是`All`运算符。这个运算符没有重载——它接受一个谓词，并且仅当源序列不包含任何不匹配谓词的项时返回`true`。在上述句子中我使用了尴尬的双重否定是有原因的：`All`应用于空序列时返回`true`，因为空序列显然不包含任何元素无法匹配谓词，简单来说，它根本没有任何元素。

这可能看起来像一种顽固的逻辑形式。这让人想起了一个孩子的情形，当问到“你吃了你的蔬菜吗？”时，他无助地回答道：“我吃了我放在盘子上的所有蔬菜”，却忽略了他根本没把任何蔬菜放在盘子上这一事实。从技术上讲，这并不是不真实的，但它未能提供父母寻找的信息。尽管如此，这些运算符之所以工作如此，是有其原因的：它们对应一些标准的数学逻辑运算符。`Any`是*存在量词*，通常写作倒立的 E (∃)，读作“存在”，而`All`是*全称量词*，通常写作倒置的 A (∀)，读作“对于所有”。数学家们很久以前就对适用于空集的全称量词陈述达成了一致的约定。例如，定义𝕍为所有蔬菜的集合，我可以断言 ∀{v : (v ∈ 𝕍) ∧ putOnPlateByMe(v)} eatenByMe(v)，或者用英语说，“对于我放在盘子上的每一个蔬菜，我吃了那个蔬菜。” 如果我放在盘子上的蔬菜集合是空的，这个陈述被认为是真实的。（也许数学家也不喜欢蔬菜。）令人愉悦的是，这种陈述的正式术语是*空真*。

## 特定项目和子范围

编写仅产生单个项目的查询可能很有用。也许你正在寻找满足某些条件的列表中的第一个对象，或者你想通过特定键标识的数据库获取信息。LINQ 定义了几个可以实现此目的的运算符，以及一些处理查询可能返回的子范围的相关运算符。

使用`Single`运算符时，你认为应该只生成一个结果的查询。示例 10-36 展示了这样的一个查询—它通过类别和编号查找课程，在我的样本数据中，这唯一确定了一个课程。

##### 示例 10-36\. 将`Single`运算符应用于查询

```cs
var q = from course in Course.Catalog
        where course.Category == "MAT" && course.Number == 101
        select course;

Course geometry = q.Single();
```

因为 LINQ 查询是通过链接操作符构建的，我们可以取出由查询表达式构建的查询，然后添加另一个运算符—在这种情况下是`Single`运算符。虽然大多数运算符会返回代表另一个查询的对象（这里是`IEnumerable<T>`，因为我们使用 LINQ 来处理对象），但`Single`不同。像`ToArray`和`ToList`一样，`Single`运算符立即评估查询，然后返回查询产生的唯一对象。如果查询未能产生正好一个对象—可能没有生成任何项，或者生成了两个—这将引发`InvalidOperationException`。（由于这是另一个立即产生结果的运算符，一些提供程序提供了`SingleAsync`，如侧边栏“即时评估和异步”中所述。）

`Single` 操作符还有一个带有谓词的重载。正如 示例 10-37 所示，这使我们能够更紧凑地表达与 示例 10-36 整体相同的逻辑。（与 `Where` 操作符一样，本节中所有基于谓词的操作符都使用 `Func<T, bool>`，而不是 `Predicate<T>`。）

##### 示例 10-37\. 带有谓词的 `Single` 操作符

```cs
Course geometry = Course.Catalog.Single(
    course => course.Category == "MAT" && course.Number == 101);
```

`Single` 操作符是严格的：如果你的查询没有精确返回一个项，它会抛出异常。还有一个略微更灵活的变体叫做 `SingleOrDefault`，允许查询返回一个或零个项。如果查询没有结果，这个方法会返回该项类型的默认值（比如引用类型返回 `null`，数值类型返回 `0` 等）。多个匹配项仍会引发异常。和 `Single` 一样，它们有两个重载：一个不带参数，用于你认为源中不会有多个对象的情况；另一个带有谓词 lambda。

LINQ 定义了两个相关的操作符 `First` 和 `FirstOrDefault`，它们分别提供了不带参数或带有谓词的重载。对于包含零个或一个匹配项的序列，它们的行为与 `Single` 和 `SingleOrDefault` 完全相同：如果存在一个项，则返回该项；如果没有，`First` 会抛出异常，而 `FirstOrDefault` 会返回 `null` 或等效值。然而，当存在多个结果时，这些操作符的响应不同——它们会选择第一个结果并返回，忽略其余结果。如果你想从列表中找出最昂贵的物品，这可能会很有用——你可以按价格降序排序查询，然后选择第一个结果。示例 10-38 使用了类似的技术来从我的示例数据中选择最长的课程。

##### 示例 10-38\. 使用 `First` 选择最长的课程

```cs
var q = from course in Course.Catalog
        orderby course.Duration descending
        select course;
Course longest = q.First();
```

如果你的查询结果没有特定的顺序保证，这些操作符会任意选择一个项。

###### 提示

不要使用 `First` 或 `FirstOrDefault`，除非你期望有多个匹配项并且只想处理其中一个。有些开发者在期望只有一个匹配项时也使用这些操作符。当然，这些操作符可以工作，但 `Single` 和 `SingleOrDefault` 操作符更准确地表达了你的期望。它们会在有多个匹配项时抛出异常，让你知道你的期望是错误的。如果你的代码存在错误的假设，通常最好是知道而不是无视它们继续执行。

`First` 和 `FirstOrDefault` 的存在引出了一个明显的问题：我能选出最后一项吗？答案是肯定的；还有 `Last` 和 `LastOrDefault` 操作符，同样，每个都提供两个重载——一个不带参数，一个带有谓词。

.NET 6.0 对`SingleOrDefault`、`FirstOrDefault`和`LastOrDefault`进行了优化。这些方法新增了重载，使你能够提供一个返回默认值的值，而不是通常的零值。如果你有一个包含整数元素的集合，其中零是有效值，这可能非常有用。示例 10-39 展示了如何使用新的`SingleOrDefault`重载，在列表为空时获取一个值为-1 的结果。这样可以区分空列表和只包含单个零值的列表。当然，如果你的应用程序中所有可能的整数值都是有效的，这就不起作用了，你需要用其他方式检测空集合。但是，在你可以指定一些特殊值来表示“不在这里”的情况时（例如，在这种情况下是-1），这些新的重载是一个有用的补充。

##### 示例 10-39\. 使用显式默认值的`SingleOrDefault`

```cs
int valueOrNegative = numbers.SingleOrDefault(-1);
```

下一个显而易见的问题是：如果我想要一个既不是第一个也不是最后一个的特定元素怎么办？在这种情况下，LINQ 的指令非常实用，因为它提供了`ElementAt`和`ElementAtOrDefault`操作符，两者都只接受一个索引。这提供了一种通过索引访问任何`IEnumerable<T>`元素的方式。你可以指定索引为一个`int`。另外，.NET 6.0 添加了使用`Index`的重载，正如你可能从“使用索引和范围语法访问元素”了解到的那样，它允许使用相对末尾的位置。例如，`²`表示倒数第二个元素。（奇怪的是，`ElementAtOrDefault`没有新增重载来指定默认值，不像上一段讨论的三个操作符。）

你需要小心使用`ElementAt`和`ElementAtOrDefault`，因为它们可能会出乎意料地昂贵。如果你要求第 10,000 个元素，这些操作符可能需要请求并丢弃前 9,999 个元素才能到达那里。如果你通过写`source.ElementAt(⁵⁰⁰)`来指定一个相对末尾的位置，操作符可能需要读取每一个元素才能找到最后一个元素，并且对于这个特定的示例，它还可能需要保持已经看到的最后 500 个元素，因为直到到达末尾时，它才知道最终需要返回哪个元素。

正如情况所示，LINQ to Objects 会检测源对象是否实现了`IList<T>`接口，如果是，则直接使用索引器来直接获取元素，而不是绕一个慢速的方式。但并不是所有的`IEnumerable<T>`实现都支持随机访问，因此这些操作符可能会非常慢。特别是，即使你的源实现了`IList<T>`，一旦你对其应用了一个或多个 LINQ 操作符，这些操作符的输出通常就不支持索引访问了。因此，在像示例 10-40 中展示的循环中使用`ElementAt`将会特别灾难性。

##### 示例 10-40\. 不正确使用`ElementAt`的例子

```cs
var mathsCourses = Course.Catalog.Where(c => c.Category == "MAT");
for (int i = 0; i < mathsCourses.Count(); ++i)
{
    // Never do this!
    Course c = mathsCourses.ElementAt(i);
    Console.WriteLine(c.Title);
}
```

尽管`Course.Catalog`是一个数组，我已经用`Where`运算符过滤了它的内容，返回了一个类型为`IEnumerable<Course>`的查询，该类型不实现`IList<Course>`接口。第一次迭代不会太糟糕——我将`ElementAt`的索引设为`0`，因此它只返回第一个匹配项，在我的样本数据中，`Where`检查的第一个项目将匹配。但是在循环的第二次迭代中，我们再次调用`ElementAt`。`mathsCourses`引用的查询并不跟踪我们在上一个循环中的位置——它是一个`IEnumerable<T>`，而不是`IEnumerator<T>`——因此这将重新开始。`ElementAt`会要求该查询返回第一个项目，它会立即丢弃它，然后请求下一个项目，这将成为返回值。因此，`Where`查询现在已经执行了两次——第一次，`ElementAt`只要求它返回一个项目，然后第二次它要求它返回两个项目，因此它现在已经处理了第一个课程两次。第三次循环（也是最后一次），我们再次重复这一过程，但这次，`ElementAt`将丢弃前两个匹配项，并返回第三个匹配项，因此现在它已经查看了第一个课程三次，第二个课程两次，第三和第四个课程各一次。（在我的样本数据中，第三个课程不属于`MAT`类别，因此当要求第三个项目时，`Where`查询会跳过它。）因此，为了检索三个项目，我已经评估了`Where`查询三次，导致它评估我的过滤 lambda 函数七次。

事实上，情况比这更糟，因为`for`循环每次还会调用`Count`方法，而对于像`Where`返回的非可索引源，`Count`必须评估整个序列——`Where`运算符告诉你有多少项匹配的唯一方法就是查看所有这些项。因此，这段代码除了`ElementAt`进行的三次部分评估外，还完全评估了`Where`返回的查询三次。在这里我们得以侥幸，因为集合很小，但如果我有一个包含 1,000 个元素的数组，所有元素都匹配过滤器，我们将完全评估`Where`查询 1,000 次，并进行另外 1,000 次部分评估。每次完全评估都会调用过滤器谓词 1,000 次，而这里的部分评估平均会这样做 500 次，因此代码最终会执行 1,500,000 次过滤。通过`foreach`循环迭代`Where`查询只会评估一次查询，执行 1,000 次过滤表达式，结果将会是一样的。

因此，在使用`Count`和`ElementAt`时要小心。如果你在迭代调用它们的集合的循环中使用它们，结果代码的复杂度将会是 O(*n*²)（即，运行代码的成本与项目数量的平方成正比增长）。

所有我刚刚描述的操作符都从源中返回单个项目。还有四个操作符也会有选择地使用项目，但可以返回多个项目：`Skip`、`Take`、`SkipLast` 和 `TakeLast`。这些操作符每个接受一个 `int` 参数。顾名思义，`Skip` 丢弃序列开头指定数量的元素，然后返回源中的所有其他元素。`Take` 从序列开头返回指定数量的元素，然后丢弃其余部分（因此类似于 SQL 中的 `TOP`）。`SkipLast` 和 `TakeLast` 作用于序列末尾，例如，您可以使用 `TakeLast` 获取源中的最后 5 个项目，或者使用 `SkipLast` 跳过最后 5 个项目。

.NET 6.0 添加了一个重载的 `Take` 方法，接受一个 `Range`，使得可以使用在“使用索引和范围语法访问元素”中描述的范围语法。例如，`source.Take(10..¹⁰)` 跳过了前 10 个和最后 10 个项目（因此等效于 `source.Skip(10).SkipLast(10)`）。由于范围语法允许您在范围的起始和结束位置使用起始或结束相对索引，我们可以使用这个 `Take` 的重载来表示其他组合。例如，`source.Take(10..20)` 的效果与 `source.Skip(10).Take(10)` 相同；`source.Take(¹⁰..²)` 相当于 `source.TakeLast(10).SkipLast(2)`。

还有基于条件的版本，`SkipWhile` 和 `TakeWhile`。`SkipWhile` 将丢弃序列中的项目，直到找到与谓词匹配的项目，此时它将返回该项目及其后续项目直至序列结束（无论剩余项目是否匹配谓词）。相反，`TakeWhile` 将返回项目，直到遇到第一个不匹配谓词的项目，此时它将丢弃该项目及其后续序列。

虽然 `Skip`、`Take`、`SkipLast`、`TakeLast`、`SkipWhile` 和 `TakeWhile` 显然都是有序敏感的，但它们并不限于仅限于有序类型，比如 `IOr⁠der⁠ed​Enu⁠mer⁠abl⁠e<T>`。它们也适用于 `IEnumerable<T>`，这是合理的，因为即使没有特定的顺序保证，`IEnumerable<T>` 总是以某种顺序产生元素。（你可以从 `IEnumerable<T>` 中逐个提取项，因此总会有一种顺序，即使是任意的。每次枚举项时可能不会相同，但对于单个评估，项必须以某种顺序出现。）此外，`IOrderedEnumerable<T>` 在 LINQ 之外并没有广泛实现，因此通常有些不了解 LINQ 的对象，虽然它们以已知的顺序产生项目，但仅实现了 `IEnumerable<T>`。这些运算符在这些场景中非常有用，因此限制得以放宽。更令人惊讶的是，`IQueryable<T>` 也支持这些操作，但这与许多数据库支持对无序查询应用 `TOP`（大致相当于 `Take`）是一致的。正如以往一样，单个提供程序可能选择不支持某些操作，因此在没有这些运算符合理解释的情况下，它们将引发异常。

## 聚合

`Sum` 和 `Average` 运算符将所有源项的值相加。`Sum` 返回总和，`Average` 返回总和除以项数。通常支持这些运算符的 LINQ 提供程序会使它们适用于这些数值类型的项目集合：`decimal`、`double`、`float`、`int` 和 `long`。还有一些重载版本，与 lambda 表达式一起工作，该 lambda 接受一个项目并返回其中一个这些数值类型，这使得我们可以编写像 示例 10-41 这样的代码，它处理 `Course` 对象集合，并计算从对象中提取的特定值的平均值：课程时长（以小时计算）。

##### 示例 10-41\. 带有投影的 `Average` 运算符

```cs
Console.WriteLine("Average course length in hours: {0}",
    Course.Catalog.Average(course => course.Duration.TotalHours));
```

LINQ 还定义了 `Min` 和 `Max` 运算符。你可以将它们应用于任何类型的序列，尽管不能保证一定成功——你使用的特定提供程序可能在不知道如何比较你使用的类型时报告错误。例如，LINQ to Objects 要求序列中的对象实现 `IComparable`。

`Min` 和 `Max` 都有重载版本，接受一个从源项目获取值的 lambda 表达式。示例 10-42 使用这一特性来找出最近发布的课程的日期。

##### 示例 10-42\. `Max` 与投影

```cs
DateOnly m = mathsCourses.Max(c => c.PublicationDate);
```

注意，此方法并不返回最近发布日期的课程；它返回的是该课程的发布日期。如果您想选择某个属性具有最大值的对象，可以使用`MaxBy`。示例 10-43 将找到具有最高`PublicationDate`的课程，但与示例 10-42 不同，它返回相关课程而不是日期。（正如您所预料的那样，还有一个`MinBy`。）

##### 示例 10-43\. 用于标准的投影`Max`但不用于结果

```cs
Course? mostRecentlyPublished = mathsCourses.MaxBy(c => c.PublicationDate);
```

您可能已经在示例中注意到了`?`，表示`MaxBy`可能返回一个`null`结果。在输入集合为空且输出类型是引用类型或其支持的支持的数值类型的可空形式（例如`int?`或`double?`）的情况下，`Max`和`MaxBy`会发生这种情况。当输出是非空的结构（例如`DateOnly`，如示例 10-42）时，这些运算符无法返回`null`，并且会抛出`InvalidOperationException`。如果您使用的是引用类型，并且希望像值类型输出那样在输入为空时引发异常，唯一的方法是自行检查是否存在`null`结果并抛出异常。示例 10-44 展示了一种实现方式。

##### 示例 10-44\. 用于标准的投影`Max`但不用于结果，输入为空时出错

```cs
Course mostRecentlyPublished = mathsCourses.MaxBy(c => c.PublicationDate)
    ?? throw new InvalidOperationException("Collection must not be empty");
```

LINQ to Objects 为返回与`Sum`和`Average`处理相同数值类型的特定序列的`Min`和`Max`定义了专用重载（即`decimal`、`double`、`float`、`int`、`long`及其可空形式）。它还为使用 lambda 表达式的形式定义了类似的专用化。这些重载存在是为了通过避免装箱来提高性能。通用形式依赖于`IComparable`，并且获取一个值的接口类型引用总是涉及装箱该值。对于大集合，装箱每个值会对 GC 造成相当大的额外压力。

LINQ 定义了一个称为`Aggregate`的运算符，它泛化了`Min`、`Max`、`Sum`和`Average`所使用的模式，即使用涉及考虑每个源项的过程来生成单个结果。可以通过`Aggregate`来实现这四个运算符（及其`...By`对应运算符）。示例 10-45 使用`Sum`运算符计算所有课程的总持续时间，然后展示如何使用`Aggregate`运算符执行完全相同的计算。

##### 示例 10-45\. `Sum` 和 `Aggregate` 的等效形式

```cs
double t1 = Course.Catalog.Sum(course => course.Duration.TotalHours);
double t2 = Course.Catalog.Aggregate(
    0.0, (hours, course) => hours + course.Duration.TotalHours);
```

聚合通过建立一个值来表示到目前为止检查过的所有项目的知识，称为*累加器*。我们使用的类型取决于我们要累积的知识。在这里，我只是将所有数字相加，所以我使用了一个`double`（因为`TimeSpan`类型的`TotalHours`属性也是一个`double`）。

最初我们没有知识，因为我们还没有查看任何项目。我们需要提供一个累加器值来表示这个起始点，因此`Aggregate`运算符的第一个参数是*seed*，累加器的初始值。在 Example 10-45 中，累加器只是一个运行总数，因此种子是`0.0`。

第二个参数是一个 lambda 表达式，描述如何更新累加器以包含单个项目的信息。由于我这里的目标只是计算总时间，所以我只是将当前课程的持续时间添加到运行总数中。

一旦`Aggregate`查看了每个项目，这个特定的重载将直接返回累加器。在这种情况下，它将是所有课程中的总小时数。如果我们使用不同的累积策略，我们可以实现`Max`。而不是维护一个运行总数，表示到目前为止关于数据的所有知识的值只是看到的最高值。Example 10-46 显示了与 Example 10-42 的大致等价物。（它不完全相同，因为 Example 10-46 没有尝试检测空源。如果此源为空，`Max`会抛出异常，但这只会返回日期 0/0/0000。）

##### Example 10-46\. 使用`Aggregate`实现`Max`

```cs
DateOnly m = mathsCourses.Aggregate(
    new DateOnly(),
    (date, c) => date > c.PublicationDate ? date : c.PublicationDate);
```

这说明了`Aggregate`并不对累积知识的值强加任何单一含义——你使用它的方式取决于你要做什么。一些操作需要一个稍微有结构的累加器。Example 10-47 使用`Aggregate`计算了平均课程持续时间。

##### Example 10-47\. 使用`Aggregate`实现`Average`

```cs
double average = Course.Catalog.Aggregate(
    new { TotalHours = 0.0, Count = 0 },
    (totals, course) => new
    {
        TotalHours = totals.TotalHours + course.Duration.TotalHours,
        Count = totals.Count + 1
    },
    totals => totals.Count > 0
        ? totals.TotalHours / totals.Count
        : throw new InvalidOperationException("Sequence was empty"));
```

平均持续时间要求我们知道两件事：总持续时间和项目数。因此，在这个例子中，我的累加器使用了一个可以包含两个值的类型，一个用来保存总和，一个用来保存项目计数。我使用了匿名类型，因为正如前面提到的，在 LINQ 中有时这是唯一的选择，并且我想展示最一般的情况。然而，值得一提的是，在这种特定情况下，元组可能更好。它会起作用，因为这是 LINQ 到对象，而轻量级元组是值类型，而匿名类型是引用类型，元组会减少被分配的对象数量。

###### 注意

示例 10-47 基于同一组件中的两个独立方法创建两个结构相同的匿名类型实例时的事实，编译器会生成一个用于两者的单一类型。种子生成了一个由`TotalHours`（`double`）和`Count`（`int`）组成的匿名类型实例。累加 lambda 也返回了一个具有相同成员名称和类型的匿名类型实例，并且顺序也相同。C# 编译器认为这些将是相同的类型，这很重要，因为`Aggregate`要求 lambda 接受并返回累加器类型的实例。

示例 10-47 使用了与先前示例不同的重载。它采用了额外的 lambda 函数，用于从累加器中提取返回值—累加器积累了我需要生成结果所需的信息，但在这个示例中，累加器本身不是结果。

当然，如果你只想计算总和、最大值或平均值，你不会使用`Aggregate`—你会使用专门设计用于执行这些任务的操作符。它们不仅更简单，而且通常更高效。 （例如，数据库的 LINQ 提供程序可能能够生成一个查询，使用数据库的内置功能计算最小或最大值。）我只是想展示灵活性，使用易于理解的例子。但现在我已经做到了，示例 10-48 展示了一个特别简洁的`Aggregate`示例，它不对应任何其他内置操作符。它接受一个矩形集合并返回包含所有这些矩形的边界框。

##### 示例 10-48\. 聚合边界框

```cs
public static Rect GetBounds(IEnumerable<Rect> rects) =>
    rects.Aggregate(Rect.Union);
```

本示例中的`Rect`结构来自`System.Windows`命名空间。这是 WPF 的一部分，它是一个非常简单的数据结构，只包含四个数字—`X`、`Y`、`Width`和`Height`—因此，即使你喜欢，你也可以在非 WPF 应用中使用它。² 示例 10-48 使用了`Rect`类型的静态`Union`方法，它接受两个`Rect`参数并返回一个包含两个输入矩形的边界框的单个`Rect`（即包含两个输入矩形的最小矩形）。

我在这里使用`Aggregate`的最简单重载。它与我在示例 10-45 中使用的方法相同，但它不需要我提供一个种子——它只使用列表中的第一项。示例 10-49 相当于示例 10-48，但使步骤更明确。我已经提供了序列中第一个`Rect`作为显式种子值，使用`Skip`来聚合除了第一个元素之外的所有内容。我还编写了一个 lambda 来调用该方法，而不是直接传递方法本身。如果你使用这种 lambda，它只是将其参数直接传递给 LINQ 到对象的现有方法，你可以直接传递方法名称，它将直接调用目标方法，而不经过你的 lambda。（你不能在基于表达式的提供程序中这样做，因为它们要求一个 lambda。）

直接使用该方法更为简洁和略微更有效，但也会导致代码略显晦涩，这就是为什么我在示例 10-49 中详细解释它的原因。

##### 示例 10-49\. 更详细和不那么晦涩的边界框聚合

```cs
public static Rect GetBounds(IEnumerable<Rect> rects)
{
    IEnumerable<Rect> theRest = rects.Skip(1);
    return theRest.Aggregate(rects.First(), (r1, r2) => Rect.Union(r1, r2));
}
```

这两个示例的工作方式相同。它们以第一个矩形作为种子。对于列表中的下一个项，`Aggregate`将调用`Rect.Union`，传递种子和第二个矩形。结果——前两个矩形的边界框——成为新的累加器值。然后将其与第三个矩形一起传递给`Union`，依此类推。示例 10-50 展示了在四个`Rect`值的集合上执行此`Aggregate`操作的效果。（我在这里表示四个值为`r1`、`r2`、`r3`和`r4`。要将它们传递给`Aggregate`，它们需要在像数组这样的集合内。）

##### 示例 10-50\. `Aggregate`的效果

```cs
Rect bounds = Rect.Union(Rect.Union(Rect.Union(r1, r2), r3), r4);
```

`Aggregate`是 LINQ 中对其他一些语言称为*reduce*的操作的称呼。有时你也会看到它被称为*fold*。LINQ 选择使用`Aggregate`这个名字的原因与其将投影运算符称为`Select`而不是*map*（函数式编程语言中更常见的名称）相同：LINQ 的术语更多受到 SQL 的影响，而不是函数式编程语言。

## 集合操作

LINQ 定义了三个运算符，使用一些常见的集合操作来合并两个源。`Intersect`生成一个结果，其中包含仅存在于两个输入源中的项。`Except`包含仅来自第一个输入源中不在第二个输入源中的项。`Union`³的输出包含存在于任一（或两者）输入源中的项。

虽然 LINQ 定义了这些集合操作，大多数 LINQ 源类型并不直接对应集合的抽象。在数学集合中，任何特定项都要么属于集合，要么不属于，没有固有的顺序概念或特定项在集合中出现的次数。`IEnumerable<T>` 不是这样的——它是一系列项，因此可能存在重复项，`IQueryable<T>` 也是如此。这并不一定是问题，因为有些集合永远不会处于包含重复项的情况中，而且在某些情况下，重复项的存在也不会导致问题。但有时，将包含重复项的集合转换为不包含重复项可能很有用。为此，LINQ 定义了 `Distinct` 操作符，用于删除重复项。示例 10-51 包含一个查询，从所有课程中提取类别名称，并将其传递给 `Distinct` 操作符，以确保每个唯一的类别名称只出现一次。

##### 示例 10-51\. 使用 `Distinct` 删除重复项

```cs
var categories = Course.Catalog.Select(c => c.Category).Distinct();
```

所有这些集合操作符都有两种形式可用，因为你可以选择向其中任何一个传递一个 `IEqualityComparer<T>`。这允许你定制操作符如何决定两个项是否相同。

.NET 6.0 添加了 `IntersectBy`、`ExceptBy`、`UnionBy` 和 `DistinctBy` 操作符。它们的基本目的与 `Intersect`、`Except`、`Union` 和 `Distinct` 相同，但用于确定等效性的机制不同。你可以提供一个 lambda，它接受源集合中的一个元素作为输入，并产生任何你想要的输出。如果这个 lambda 对两个项产生相同的结果，则认为它们是相同的。（例如，你可以编写 `courses.DistinctBy(c => c.Title)`，如果两个课程具有相同的 `Title`，则它们被视为相同。）你也可以通过编写自定义的 `IEq⁠ual⁠ity​Com⁠par⁠er⁠<T>` 来实现相同的效果，但使用投影通常更简单。（这四种方法的所有重载还接受一个 `IEqualityComparer<T>`。如果你的投影产生一个字符串，并且你想指定字符串比较机制，这可能很有用。）

## 整个序列、保持顺序的操作

LINQ 定义了一些操作符，它们的输出包括源中的每个项，并保留或者反转顺序。并非所有集合都一定有顺序，因此这些操作符并不总是被支持。不过，LINQ 对对象支持它们全部。最简单的是 `Reverse`，它反转了元素的顺序。

`Concat`操作符组合两个序列。它返回一个序列，该序列产生第一个序列中所有元素（以该序列返回它们的任何顺序），然后是第二个序列中所有元素（再次保持顺序）。在需要仅将单个元素添加到第一个序列末尾的情况下，可以使用`Append`。还有`Prepend`，它在开头添加单个项目。`Repeat`操作符有效地连接源的指定数量的副本。

`DefaultIfEmpty`操作符返回其源的所有元素。但是，如果源为空，它将返回单个元素。这个方法有两个重载版本：您可以指定源为空时返回的默认值，或者如果不传递参数，则使用元素类型的默认值，类似于零。

`Zip`操作符也可以组合两个序列，但不是依次返回每个元素，它是逐对元素进行操作。因此，它返回的第一个项目将基于第一个序列和第二个序列的第一个项目。zipped 序列中的第二个项目将基于每个序列的第二个项目，依此类推。名称`Zip`旨在让人联想到服装上拉链的作用，将两个物品完美对齐在一起。（这并不是一个精确的类比。当拉链将两部分连接时，两半部分的齿会交错连接。但`Zip`操作符不会像物理拉链的齿那样交错处理其输入。它将两个源的项目成对组合在一起。）

我们需要告诉`Zip`如何组合项目。它接受一个带有两个参数的 lambda 函数，将来自两个源的项目对作为这些参数传递，并生成你的 lambda 函数返回的输出项目。示例 10-52 使用一个选择器，通过字符串连接组合每对项目。

##### 示例 10-52\. 使用`Zip`组合列表

```cs
string[] firstNames = { "Elisenda", "Jessica", "Liam" };
string[] lastNames = { "Gascon", "Hill", "Mooney" };
`IEnumerable``<``string``>` `fullNames` `=` `firstNames``.``Zip``(``lastNames``,`
    `(``first``,` `last``)` `=``>` `first` `+` `" "` `+` `last``)``;`
foreach (string name in fullNames)
{
    Console.WriteLine(name);
}
```

此示例中`Zip`组合在一起的两个列表包含名字和姓氏。输出如下：

```cs
Elisenda Gascon
Jessica Hill
Liam Mooney
```

如果输入源包含不同数量的项目，`Zip`将在达到较短集合的末尾时停止，并且不会尝试从较长的集合中获取更多项目。它不会将不匹配的长度视为错误。

`Zip`还有一些不需要 lambda 函数的重载。这些重载只返回一个元组序列。有两个版本：一个是组合一对序列，产生 2 元组的版本，另一个是接受三个序列，将它们组合成 3 元组的版本。（没有对应的三个输入 lambda-based `Zip`。）

`SequenceEqual` 运算符与 `Zip` 类似，它对两个序列进行操作，并处理两个序列中在相同位置上找到的项目对。但是，`SequenceEqual` 只是比较每对项目是否相等，而不是将它们传递给 lambda 表达式进行组合。如果比较过程发现两个源包含相同数量的项，并且对于每一对，两个项目都相等，则返回 `true`。如果源长度不同，或者仅有一对项目不相等，则返回 `false`。`SequenceEqual` 有两个重载，一个只接受用于与源比较的列表，另一个还接受一个 `IEqualityComparer<T>` 以自定义相等的含义。

## 分组

有时候，你会想要将具有共同特点的所有项目作为一组进行处理。示例 10-53 使用查询按类别对课程进行分组，在列出该类别下的所有课程之前写出每个类别的标题。

##### 示例 10-53\. 分组查询表达式

```cs
var subjectGroups = from course in Course.Catalog
                    group course by course.Category;

foreach (var group in subjectGroups)
{
    Console.WriteLine("Category: " + group.Key);
    Console.WriteLine();

    foreach (var course in group)
    {
        Console.WriteLine(course.Title);
    }
    Console.WriteLine();
}
```

`group` 子句接受一个表达式，用于确定组成员资格——在本例中，任何返回相同值的 `Category` 属性的课程都将被视为同一组的成员。 `group` 子句生成一个集合，其中每个项实现表示组的类型。由于我正在使用 LINQ 对象，且按类别字符串进行分组，在 示例 10-53 中 `subjectGroup` 变量的类型将为 `IEnumerable<IGrouping<string, Course>>`。此特定示例生成了三个组对象，如 图 10-1 所示。

每个 `IGrouping<string, Course>` 项都有一个 `Key` 属性，由于查询通过课程的 `Category` 属性对项进行分组，每个键包含来自该属性的字符串值。在 示例 10-17 中的示例数据中有三个不同的类别名称：`MAT`、`BIO` 和 `CSE`，因此这些是三个组的 `Key` 值。

`IGrouping<TKey, TItem>` 接口派生自 `IEnumerable<TItem>`，因此可以枚举每个组对象以查找它包含的项。因此，在 示例 10-53 中，外部 `foreach` 循环遍历查询返回的三个组，然后内部 `foreach` 循环遍历每个组中的 `Course` 对象。

![](img/pc10_1001.png)

###### 图 10-1\. 评估分组查询的结果

查询表达式变成了 示例 10-54 中的代码。

##### 示例 10-54\. 扩展简单分组查询

```cs
var subjectGroups = Course.Catalog.GroupBy(course => course.Category);
```

查询表达式在分组主题上提供了一些变体。通过对原始查询进行轻微修改，我们可以安排每个组中的项目不再是原始的`Course`对象。在示例 10-55 中，我已将`group`关键字后面的表达式从`course`改为了`course.Title`。

##### 示例 10-55\. 使用项目投影的分组查询

```cs
var subjectGroups = from course in Course.Catalog
                    group course.Title by course.Category;
```

这仍然具有相同的分组表达式`course.Category`，因此仍然会生成三个组，但现在它的类型是`IGrouping<string, string>`。如果您迭代其中一个组的内容，您会发现每个组提供了一个字符串序列，其中包含课程名称。正如示例 10-56 所示，编译器会将此查询扩展为`GroupBy`操作符的另一个重载。

##### 示例 10-56\. 使用项目投影扩展的分组查询

```cs
var subjectGroups = Course.Catalog
    .GroupBy(course => course.Category, course => course.Title);
```

查询表达式要求其最后一个子句必须是`select`或`group`之一。然而，如果一个查询包含`group`子句，它不必是最后一个子句。在示例 10-55 中，我修改了查询如何表示每个组内的每个项目（即图 10-1 右侧的框），但我也可以自定义表示每个组的对象（左侧的项目）。默认情况下，我会得到`IGrouping<TKey, TItem>`对象（或者对于查询使用的任何 LINQ 提供程序的等效对象），但我可以更改这一点。示例 10-57 在其`group`子句中使用了可选的`into`关键字。这引入了一个新的范围变量，可以遍历组对象，我可以继续在查询的其余部分使用它。我可以跟随其他子句类型，比如`orderby`或`where`，但在这种情况下，我选择使用了一个`select`子句。

##### 示例 10-57\. 使用组投影的分组查询

```cs
var subjectGroups =
    from course in Course.Catalog
    group course by course.Category into category
    select $"Category '{category.Key}' contains {category.Count()} courses";
```

此查询的结果是一个`IEnumerable<string>`，如果显示它生成的所有字符串，会得到以下内容：

```cs
Category 'MAT' contains 3 courses
Category 'BIO' contains 2 courses
Category 'CSE' contains 1 courses
```

如示例 10-58 所示，这会扩展为调用与示例 10-54 相同的`GroupBy`重载，然后在最后一个子句中使用普通的`Select`操作符。

##### 示例 10-58\. 扩展的组投影分组查询

```cs
IEnumerable<string> subjectGroups = Course.Catalog
    .GroupBy(course => course.Category)
    .Select(category =>
        $"Category '{category.Key}' contains {category.Count()} courses");
```

LINQ to Objects 定义了一些更多的`GroupBy`操作符重载，这些重载不能从查询语法中访问。示例 10-59 展示了一个提供稍微更直接等效于示例 10-57 的重载。

##### 示例 10-59\. 使用键和组投影的`GroupBy`

```cs
IEnumerable<string> subjectGroups = Course.Catalog.GroupBy(
    course => course.Category,
    (category, courses) =>
        $"Category '{category}' contains {courses.Count()} courses");
```

此重载采用两个 lambda 表达式。第一个是用于分组项目的表达式。第二个用于生成每个组对象。与之前的示例不同，这不使用`IGrouping<TKey, TItem>`接口。相反，最后一个 lambda 接收关键字作为一个参数，然后作为第二个参数接收组中项目的集合。这与`IGrouping<TKey, TItem>`封装的信息完全相同，但因为此操作符的此形式可以将它们作为单独的参数传递，所以它消除了需要创建用于表示组的对象的必要性。

在示例 10-60 中还展示了该操作符的另一个版本。它结合了所有其他变体的功能。

##### 示例 10-60\. 带有键、项目和组投影的`GroupBy`操作符

```cs
IEnumerable<string> subjectGroups = Course.Catalog.GroupBy(
    course => course.Category,
    course => course.Title,
    (category, titles) =>
         $"Category '{category}' contains {titles.Count()} courses: " +
             string.Join(", ", titles));
```

此重载采用三个 lambda 表达式。第一个是用于分组项目的表达式。第二个确定如何表示组内各个项目——这次我选择提取课程标题。第三个 lambda 用于生成每个组对象，就像示例 10-59 一样，最后一个 lambda 将关键字作为一个参数传递，并将其它参数作为组项目传递，由第二个 lambda 转换。因此，第二个参数不再是原始的`Course`项目，而是包含课程标题的`IEnumerable<string>`，因为这是本示例中第二个 lambda 请求的内容。`GroupBy`操作符的结果再次是一个字符串集合，但现在看起来像这样：

```cs
Category 'MAT' contains 3 courses: Elements of Geometry, Squaring the Circle, Hy
perbolic Geometry
Category 'BIO' contains 2 courses: Recreational Organ Transplantation, Introduct
ion to Human Anatomy and Physiology
Category 'CSE' contains 1 courses: Oversimplified Data Structures for Demos
```

我展示了`GroupBy`操作符的四个版本。所有四个版本都接受一个 lambda，用于选择用于分组的键，而最简单的重载仅接受键本身。其他版本让您控制组内各个项目的表示形式，或者每个组的表示形式，或者两者兼而有之。这个操作符还有另外四个版本，它们提供了与我已展示的四个版本完全相同的功能，但还接受一个`IEqualityComparer<T>`，让您可以自定义用于分组目的的逻辑来确定两个键是否被视为相同。

有时按多个值分组很有用。例如，假设您想按类别和出版年份分组课程。您可以链接操作符，首先按类别分组，然后按类别内的年份分组（或反之）。但您可能不希望这种嵌套水平——您可能希望将课程分组到每个唯一的`Category`和出版年份组合下。做法很简单，只需将两个值放入键中，可以通过使用匿名类型实现，如示例 10-61 所示。

##### 示例 10-61\. 复合组键

```cs
var bySubjectAndYear =
    from course in Course.Catalog
    group course by new { course.Category, course.PublicationDate.Year };
foreach (var group in bySubjectAndYear)
{
    Console.WriteLine($"{group.Key.Category} ({group.Key.Year})");
    foreach (var course in group)
    {
        Console.WriteLine(course.Title);
    }
}
```

这利用了匿名类型为我们实现了 `Equals` 和 `GetHashCode` 的事实。它适用于所有形式的 `GroupBy` 操作符。对于不将它们的 lambda 表达式视为表达式的 LINQ 提供程序（例如 LINQ to Objects），您可以改用元组，这样会更加简洁，但效果相同。

还有另一个分组输出的运算符称为 `GroupJoin`，但它作为联接操作的一部分执行，我们将先看一些更简单的联接。

## 联接操作

LINQ 定义了一个 `Join` 操作符，使得查询可以使用来自其他源的相关数据，就像数据库查询可以将一张表中的信息与另一张表中的数据联接一样。假设我们的应用程序存储了哪些学生报名了哪些课程的列表。如果将该信息存储在文件中，您不希望将课程或学生的完整详细信息复制到每一行中，而是只希望有足够的信息来识别学生和特定的课程。在我的示例数据中，课程通过类别和编号的组合唯一标识。为了跟踪谁报名了什么课程，我们需要记录包含三个信息：课程类别、课程编号以及用于识别学生的某些信息。在 示例 10-62 中的记录类型显示了我们可以如何在内存中表示这种关联。

##### Example 10-62\. 记录类型关联学生和课程

```cs
public record CourseChoice(int StudentId, string Category, int Number);
```

一旦我们的应用程序将这些信息加载到内存中，我们可能希望访问 `Course` 对象，而不仅仅是识别课程的信息。我们可以通过 `join` 子句实现这一点，如 示例 10-63 所示（它还使用 `CourseChoice` 类提供了一些额外的示例数据，以便查询有可用的数据）。

##### Example 10-63\. 使用 `join` 子句查询

```cs
CourseChoice[] choices =
{
    new CourseChoice(StudentId: 1, Category: "MAT", Number: 101),
    new CourseChoice(StudentId: 1, Category: "MAT", Number: 102),
    new CourseChoice(StudentId: 1, Category: "MAT", Number: 207),
    new CourseChoice(StudentId: 2, Category: "MAT", Number: 101),
    new CourseChoice(StudentId: 2, Category: "BIO", Number: 201),
};

var studentsAndCourses = from choice in choices
                         `join` `course` `in` `Course``.``Catalog`
                           `on` `new` `{` `choice``.``Category``,` `choice``.``Number` `}`
                           `equals` `new` `{` `course``.``Category``,` `course``.``Number` `}`
                         select new { choice.StudentId, Course = course };

foreach (var item in studentsAndCourses)
{
    Console.WriteLine(
        $"Student {item.StudentId} will attend {item.Course.Title}");
}
```

这显示了 `choices` 数组中每个条目的一行。它显示了每门课程的标题，因为尽管在输入集合中未提供该信息，但 `join` 子句定位了课程目录中的相关条目。示例 10-64 展示了编译器如何将 示例 10-63 中的查询转换。

##### Example 10-64\. 直接使用 `Join` 操作符

```cs
var studentsAndCourses = choices.Join(
    Course.Catalog,
    choice => new { choice.Category, choice.Number },
    course => new { course.Category, course.Number },
    (choice, course) => new { choice.StudentId, Course = course });
```

`Join`操作符的作用是查找第二个序列中与第一个项目对应的项目。这种对应关系由前两个 lambda 表达式决定；如果这两个 lambda 返回的值相等，则来自两个源的项目将被视为相互对应。本示例使用匿名类型，并依赖于同一程序集中两个结构上相同的匿名类型实例具有相同类型的事实。换句话说，这两个 lambda 都生成具有相同类型的对象。编译器为任何匿名类型生成一个`Equals`方法，逐个比较每个成员，因此该代码的效果是，如果它们的`Category`和`Number`属性相等，则认为两行相对应。（再次强调，对于基于`IQueryable<T>`的提供程序，我们必须使用匿名类型，而不是元组，因为这些 lambda 将转换为表达式树。但由于此示例使用非表达式的提供程序，LINQ 到对象，您可以稍微简化此代码，改用元组。）

我已经设置了这个示例，以便只能有一个匹配项，但如果课程类别和编号由于某种原因不能唯一标识课程会发生什么？如果任何单个输入行有多个匹配项，`Join`操作符将为每个匹配项生成一个输出项，因此在这种情况下，输出项数量将超过`choices`数组中的条目数。相反，如果第一个源中的项目在第二个集合中没有对应的项目，`Join`将不会为该项目生成任何输出项——它实际上会忽略该输入项。

LINQ 提供了一种替代的连接类型，用于处理具有零个或多个相应行的输入行，其方式与`Join`操作符不同。示例 10-65 展示了修改后的查询表达式。（区别在于`join`子句末尾添加了`into courses`，最终的`select`子句引用该子句而不是`course`范围变量。）这会以不同的形式生成输出，因此我还修改了编写结果的代码。

##### 示例 10-65\. 分组连接

```cs
var studentsAndCourses =
    from choice in choices
    join course in Course.Catalog
      on new { choice.Category, choice.Number }
      equals new { course.Category, course.Number }
      `into` `courses`
    select new { choice.StudentId, Courses = courses };

foreach (var item in studentsAndCourses)
{
    Console.WriteLine($"Student {item.StudentId} will attend " +
        string.Join(",", item.Courses.Select(course => course.Title)));
}
```

如示例 10-66 所示，这导致编译器生成对`GroupJoin`操作符的调用，而不是`Join`。

##### 示例 10-66\. `GroupJoin`操作符

```cs
var studentsAndCourses = choices.GroupJoin(
    Course.Catalog,
    choice => new { choice.Category, choice.Number },
    course => new { course.Category, course.Number },
    (choice, courses) => new { choice.StudentId, Courses = courses });
```

这种连接形式通过调用最终的 lambda 为输入集合中的每个项目生成一个结果。它的第一个参数是输入项，第二个参数将是来自第二个集合的所有相应对象的集合。（与`Join`相比，后者为每个匹配项调用最终的 lambda 一次，逐个传递相应的项目。）这提供了一种表示第二个集合中没有相应项目的输入项的方法：操作符可以简单地传递一个空集合。

`Join`和`GroupJoin`还有重载，接受`IEqualityComparer<T>`以便您可以为前两个 lambda 返回的值定义自定义的相等性意义。

## 转换

有时您需要将一个类型的查询转换为另一种类型。例如，您可能已经得到一个集合，其中类型参数指定了某个基本类型（例如`object`），但您有充分理由相信该集合实际上包含某些更具体类型的项目（例如`Course`）。在处理单个对象时，您可以使用 C#的转型语法将引用转换为您认为正在处理的类型。不幸的是，这对于诸如`IEnumerable<T>`或`IQueryable<T>`之类的类型并不适用。

虽然协变意味着`IEnumerable<Course>`可以隐式转换为`IEnumerable<object>`，但即使使用显式向下转换也不能反向转换。如果您有一个类型为`IEnumerable<object>`的引用，试图将其转换为`IEnumerable<Course>`将仅在对象实现`IEnumerable<Course>`时成功。很可能最终得到一个完全由`Course`对象组成但不实现`IEnumerable<Course>`的序列。注意，示例 10-67 创建了这样的序列，当试图转换为`IEnumerable<Course>`时将抛出异常。

##### 示例 10-67\. 如何避免对序列进行强制转换

```cs
IEnumerable<object> sequence = Course.Catalog.Select(c => (object) c);
var courseSequence = (IEnumerable<Course>) sequence; // InvalidCastException
```

当然，这是一个刻意设计的示例。我通过将`Select` lambda 的返回类型强制转换为`object`来强制创建一个`IEnumerable<object>`。然而，在稍微复杂的情况下，很容易陷入这种情况。幸运的是，有一个简单的解决方案。您可以使用`Cast<T>`运算符，如示例 10-68 所示。

##### 示例 10-68\. 如何对序列进行强制转换

```cs
var courseSequence = sequence.Cast<Course>();
```

这返回一个查询，按顺序产生其来源中的每个项目，但在执行此操作时将每个项目转换为指定的目标类型。这意味着尽管最初的`Cast<T>`可能成功，但在稍后尝试从序列中提取值时可能会得到`InvalidCastException`。毕竟，通常来说，`Cast<T>`运算符能够验证您提供的序列确实只产生类型为`T`的值的唯一方法是提取所有这些值并尝试转换它们。它无法预先评估整个序列，因为您可能提供了一个无限序列。如果您的序列首次产生的十亿个项目是正确类型的，但之后返回一个不兼容类型的项目，那么`Cast<T>`发现这一点的唯一方法就是逐个尝试转换项目。

###### 小贴士

`Cast<T>` 和 `OfType<T>` 看起来相似，有时开发人员在应该使用另一个时使用了一个（通常是因为他们不知道两者都存在）。`OfType<T>` 几乎与 `Cast<T>` 做的事情一样，但它会静默地过滤掉任何错误类型的项目，而不是抛出异常。如果您期望并希望忽略错误类型的项目，请使用 `OfType<T>`。如果您不期望错误类型的项目出现，请使用 `Cast<T>`，因为如果您错了，它会通过抛出异常来告诉您，减少隐藏潜在 bug 的风险。

LINQ to Objects 定义了一个 `AsEnumerable<T>` 运算符。它只是返回源而没有任何修改——在运行时没有任何效果。其目的是即使处理可能由不同的 LINQ 提供程序处理的内容，也强制使用 LINQ to Objects。例如，假设您有一个实现了 `IQueryable<T>` 的东西。该接口从 `IEnumerable<T>` 派生，但是适用于 `IQueryable<T>` 的扩展方法将优先于 LINQ to Objects。如果您的意图是在数据库上执行特定查询，然后使用 LINQ to Objects 进行进一步的客户端处理结果，您可以使用 `AsEnumerable<T>` 来划定界限，表示“这是我们将事务移到客户端的地方”。

相反，还有 `AsQueryable<T>`。它设计用于具有静态类型 `IEnumerable<T>` 变量的场景，您认为该变量可能包含对也实现 `IQueryable<T>` 的对象的引用，并且您希望任何创建的查询都使用该引用而不是 LINQ to Objects。如果您在一个实际上不实现 `IQueryable<T>` 的源上使用此运算符，则返回一个实现 `IQueryable<T>` 的包装器，但在内部使用 LINQ to Objects。

另一个选择不同 LINQ 口味的运算符是 `AsParallel`。它返回 `ParallelQuery<T>`，允许您构建由并行 LINQ 执行的查询，这是一个能够利用多个 CPU 核心并行执行某些操作以提高性能的 LINQ 提供程序。

有一些运算符可以将查询转换为其他类型，同时也会立即执行查询，而不是在先前查询的基础上构建新的查询链。`ToArray`，`ToList` 和 `ToHashSet` 分别返回包含执行输入查询完整结果的数组、列表或哈希集合。`ToDictionary` 和 `ToLookup` 同样如此，但它们不是产生简单的项目列表，而是生成支持关联查找的结果。`ToDictionary` 返回 `Dictionary<TKey, TValue>`，因此适用于键对应于唯一值的场景。`ToLookup` 设计用于键可能关联多个值的场景，因此返回不同类型 `ILookup<TKey, TValue>`。

我在 Chapter 5 中没有提及此查找接口，因为它特定于 LINQ。它与只读字典接口本质上相同，只是索引器返回 `IEnumerable<TValue>` 而不是单个 `TValue`。

数组和列表转换不需要参数，但字典和查找转换需要告诉每个源项要使用的键值。正如 Example 10-69 所示，通过传递 lambda 来告诉它们。这里使用课程的 `Category` 属性作为键。

##### Example 10-69\. 创建查找

```cs
ILookup<string, Course> categoryLookup =
    Course.Catalog.ToLookup(course => course.Category);
foreach (Course c in categoryLookup["MAT"])
{
    Console.WriteLine(c.Title);
}
```

`ToDictionary` 操作符提供了一个重载，参数与 `ToLookup` 相同，但返回字典而不是查找表。如果你像在 Example 10-69 中调用 `ToLookup` 一样调用它，会抛出异常，因为多个课程对象共享类别，它们会映射到相同的键。`ToDictionary` 要求每个对象具有唯一的键。要从课程目录生成字典，你需要首先按类别分组数据，并使每个字典条目引用整个组，或者需要一个 lambda 返回基于课程类别和编号的复合键，因为该组合对于课程是唯一的。

这两个操作符还提供了一个重载，接受一对 lambda 表达式——一个提取键，另一个选择用作相应值的内容（您无需使用源项作为值）。最后，还有接受 `IEqualityComparer<T>` 的重载。

现在您已经看到所有标准 LINQ 操作符，但由于这占据了相当多的页面，您可能会发现具有简洁摘要很有用。Table 10-1 列出了这些操作符，并简要描述了每个操作符的用途。

Table 10-1\. LINQ 操作符摘要

| 操作符 | 目的 |
| --- | --- |
| `Aggregate` | 通过用户提供的函数组合所有项以产生单个结果。 |
| `All` | 如果对所有项条件均不满足，返回`true`。 |
| `Any` | 如果对至少一个项满足条件，返回`true`。 |
| `Append` | 返回具有所有输入序列中的项及末尾添加的一项的序列。 |
| `AsEnumerable` | 将序列作为 `IEnumerable<T>` 返回。（强制使用 LINQ to Objects 很有用。） |
| `AsParallel` | 返回用于并行查询执行的`ParallelQuery<T>`。 |
| `AsQueryable` | 确保在可用时使用 `IQueryable<T>` 处理。 |
| `Average` | 计算项的算术平均值。 |
| `Cast` | 将序列中的每个项转换为指定类型。 |
| `Chunk` | 将序列分割成相等大小的批次。 |
| `Concat` | 通过连接两个序列形成一个新序列。 |
| `Contains` | 如果序列中包含指定的项，则返回`true`。 |
| `Count`, `LongCount` | 返回序列中的项数。 |
| `DefaultIfEmpty` | 生成源序列的元素，除非没有元素，此时生成一个具有默认值的单个元素。 |
| `Distinct` | 删除重复值。 |
| `DistinctBy` | 删除投影生成重复值的值。 |
| `ElementAt` | 返回指定位置的元素（如果超出范围则抛出异常）。 |
| `ElementAtOrDefault` | 返回指定位置的元素（如果超出范围则生成元素类型的默认值）。 |
| `Except` | 过滤掉在另一个提供的集合中的项目。 |
| `First` | 返回第一个项目，如果没有项目则抛出异常。 |
| `FirstOrDefault` | 如果没有项目，则返回第一个项目或默认值。 |
| `GroupBy` | 将项目分组。 |
| `GroupJoin` | 根据它们与输入序列中项目的关系，对另一个序列中的项目进行分组。 |
| `Intersect` | 过滤掉不在另一个提供的集合中的项目。 |
| `IntersectBy` | 使用投影进行比较的`Intersect`。 |
| `Join` | 对两个输入序列中每个匹配对的项目生成一个项目。 |
| `Last` | 返回最后一个项目，如果没有项目则抛出异常。 |
| `LastOrDefault` | 如果没有项目，则返回最后一个项目或默认值。 |
| `Max` | 返回最高值。 |
| `MaxBy` | 返回投影产生最高值的项目。 |
| `Min` | 返回最低值。 |
| `MinBy` | 返回投影产生最低值的项目。 |
| `OfType` | 过滤掉不是指定类型的项目。 |
| `OrderBy` | 以升序生成项目。 |
| `OrderByDescending` | 以降序生成项目。 |
| `Prepend` | 返回以指定单个项目开始，后跟其输入序列中所有项目的序列。 |
| `Reverse` | 以与输入相反的顺序生成项目。 |
| `Select` | 通过函数对每个项目进行投影。 |
| `SelectMany` | 将多个集合合并为一个。 |
| `SequenceEqual` | 仅当所有项目与另一个提供的序列中的项目相等时返回`true`。 |
| `Single` | 返回唯一项目，如果没有项目或多于一个项目则抛出异常。 |
| `SingleOrDefault` | 返回唯一项目或默认值（如果没有项目则抛出异常）；如果存在多个项目则抛出异常。 |
| `Skip` | 从开头过滤指定数量的项目。 |
| `SkipLast` | 从末尾过滤掉指定数量的项目。 |
| `SkipWhile` | 从开头开始过滤项目，直到项目不匹配为止。 |
| `Sum` | 返回所有项目相加的结果。 |
| `Take` | 生成指定数量或范围的项目，丢弃其余项目。 |
| `TakeLast` | 从输入的末尾生成指定数量的项目（丢弃之前的所有项目）。 |
| `TakeWhile` | 只要项目匹配谓词，就生成项目；一旦有一个项目不匹配，就丢弃其余序列。 |
| `ToArray` | 返回包含所有项目的数组。 |
| `ToDictionary` | 返回包含所有项目的字典。 |
| `ToHashSet` | 返回包含所有项目的`HashSet<T>`。 |
| `ToList` | 返回包含所有项目的`List<T>`。 |
| `ToLookup` | 返回包含所有项目的多值关联查找。 |
| `Union` | 生成位于任一输入中或两者中的所有项目。 |
| `UnionBy` | 与`Union`相同，但使用投影进行比较。 |
| `Where` | 过滤掉不符合提供的谓词的项目。 |
| `Zip` | 将来自两个或三个输入的相同位置的项目组合在一起。 |

# 序列生成

`Enumerable`类定义了扩展方法，用于`IEnumerable<T>`，构成了 LINQ to Objects。它还提供了一些额外的（非扩展）静态方法，用于创建新的序列。`Enumerable.Range`接受两个`int`参数，并返回一个`IEnumerable<int>`，产生从第一个参数值开始的连续递增数字系列，包含第二个参数指定的数字个数。例如，`Enumerable.Range(15, 10)`生成包含数字 15 到 24（包括）的序列。

`Enumerable.Repeat<T>`接受类型为`T`的值和计数。它返回一个序列，该序列将产生指定次数的该值。

`Enumerable.Empty<T>`返回一个不包含任何元素的`IEnumerable<T>`。这听起来可能不是很有用，因为有一个更简洁的替代方案。你可以写`new T[0]`，它创建一个不包含任何元素的数组（类型为`T`的数组实现了`IEnumerable<T>`）。然而，`Enumerable.Empty<T>`的优点在于，对于任何给定的`T`，它每次返回相同的实例。这意味着如果由于任何原因你需要在执行许多迭代的循环中重复使用空序列，`Enumerable.Empty<T>`更高效，因为它对 GC 的压力较小。

# 其他 LINQ 实现

本章中大多数我展示的示例都使用了 LINQ to Objects，除了少数几个引用了 EF Core。在这最后一节中，我将快速描述一些其他基于 LINQ 的技术。这并不是一个详尽的列表，因为任何人都可以编写 LINQ 提供程序。

## 实体框架核心

我展示的数据库示例使用了 Entity Framework Core（EF Core）的 LINQ 提供程序。EF Core 是一种数据访问技术，以 NuGet 包`Microsoft.EntityFrameworkCore`的形式提供。（EF Core 的前身，Entity Framework，仍内置于.NET Framework 中，但不包括在较新版本的.NET 中。）EF Core 可以在数据库和对象层之间进行映射。它支持多个数据库供应商。

EF Core 依赖于 `IQueryable<T>`。对于数据模型中的每个持久化实体类型，EF 可以提供一个实现 `IQueryable<T>` 的对象，作为构建检索该类型和相关类型实体查询的起点。由于 `IQueryable<T>` 不仅仅适用于 EF，您将使用 `System.Linq` 命名空间中提供的标准扩展方法集，但该机制设计用于允许每个提供程序插入其自己的行为。

因为 `IQueryable<T>` 将 LINQ 操作符定义为接受 `Expression<T>` 参数的方法，而不是普通的委托类型，所以您在查询表达式或作为底层操作符方法的 lambda 参数中编写的任何表达式都将转换为由编译器生成的代码，创建表示表达式结构的对象树。EF 依赖于此能力来生成检索所需数据的数据库查询。这意味着您必须使用 lambda；与 LINQ to Objects 不同，您不能在 EF 查询中使用匿名方法或委托。

###### 警告

因为 `IQueryable<T>` 派生自 `IEnumerable<T>`，所以可以在任何 EF 源上使用 LINQ to Objects 操作符。您可以通过 `AsEnumerable<T>` 操作符明确地执行此操作，但如果使用的重载支持 LINQ to Objects 而不支持 `IQueryable<T>`，也可能会发生意外情况。例如，如果尝试使用委托而不是 lambda 作为 `Where` 操作符的谓词，这将回退到 LINQ to Objects。这里的要点是，EF 最终会下载整个表的内容，然后在客户端上评估 `Where` 操作符。这不太可能是一个好主意。

## 并行 LINQ（PLINQ）

并行 LINQ 与 LINQ to Objects 相似，因为它基于对象和委托，而不是表达式树和查询翻译。但是，当您开始从查询请求结果时，它将尽可能使用多线程评估，利用线程池来有效地使用可用的 CPU 资源。第十六章 将展示多线程操作的实际效果。

## XML 的 LINQ

LINQ to XML 不是一个 LINQ 提供程序。我在这里提到它，因为它的名称听起来像一个。它真正是一个用于创建和解析 XML 文档的 API。它被称为*LINQ to XML*，因为它旨在通过 .NET 对象模型轻松执行对 XML 文档的 LINQ 查询，但它通过 .NET 对象模型来呈现 XML 文档来实现这一点。运行库提供了两个单独的 API 来实现这一点：除了 LINQ to XML 外，它还提供了 XML 文档对象模型（DOM）。DOM 基于一个平台无关的标准，因此与 .NET 习惯用法不太匹配，并且与大多数运行库相比感觉不必要地古怪。LINQ to XML 纯粹是为 .NET 设计的，因此它与普通的 C# 技术集成得更好。这包括与 LINQ 良好地配合工作，它通过提供从文档中提取特性的方法来推迟到 LINQ to Objects 来定义和执行查询。

## `IAsyncEnumerable<T>`

正如第五章所述，.NET 定义了`IAsyncEnumerable<T>`接口，这是`IEnumerable<T>`的异步等价物。第十七章将描述语言特性，使您能够使用这个接口。虽然 .NET 运行库中没有内置完整的 LINQ 操作符集，但它们在一个名为`System.Linq.Async`的 NuGet 包中可用。

## 响应式扩展

.NET 的响应式扩展（或简称为 Rx）是下一章的主题，因此我不会在这里过多介绍它们，但它们很好地说明了 LINQ 操作符如何在各种类型上工作。Rx 反转了本章展示的模型，我们可以在准备好并且需要数据时调用一个 Rx 源，而不是编写一个`foreach`循环来迭代查询，或者调用诸如`ToArray`或`SingleOrDefault`等评估查询的运算符。

尽管如此，Rx 有一个 LINQ 提供程序，支持大多数标准的 LINQ 操作符。

# 总结

在本章中，我展示了支持一些最常用的 LINQ 特性的查询语法。这使我们能够在 C# 中编写类似于数据库查询的查询，但可以查询任何 LINQ 提供程序，包括 LINQ to Objects，使我们能够针对我们的对象模型运行查询。我展示了用于查询的标准 LINQ 操作符，所有这些操作符都可以在 LINQ to Objects 中使用，大多数可以在数据库提供程序中使用。我还提供了一些常见的 .NET 应用程序的 LINQ 提供程序的快速概述。

我提到的最后一个提供程序是 Rx。但在我们查看 Rx 的 LINQ 提供程序之前，下一章将从如何使用 Rx 本身开始。

¹ 当我写这篇文章时，.NET 7.0 的初步功能集包括修复这个问题，因此有一些希望这可能会得到改善。

² 如果你这样做，请注意不要将其与另一种 WPF 类型`Rectangle`混淆。那是一个更为复杂的实体，支持动画、样式、布局、用户输入、数据绑定以及其他各种 WPF 功能。请勿在 WPF 应用程序之外尝试使用`Rectangle`。

³ 这与在前面示例中使用的`Rect.Union`方法无关。
