# 第二章：编码算法

我们每天都在编码，思考我们要解决的问题，确保我们的算法正确运行。这就是应该的方式，现代工具和软件开发工具包越来越多地释放我们的时间，专注于这一点。即便如此，C#、.NET 和编码中的某些特性仍然显著影响效率、性能和可维护性。

# 性能

本章的几个主题讨论了应用程序性能，如高效处理字符串、缓存数据或延迟实例化类型直到需要时。在一些简单的场景中，这些事情可能并不重要。然而，在需要性能和规模的复杂企业应用程序中，关注这些技术可以帮助避免生产中的昂贵问题。

# 可维护性

如何组织代码显著影响其可维护性。在第一章的讨论基础上，您将看到一种新的模式和策略，并理解它们如何简化算法并使应用程序更易扩展。另一节讨论了如何在自然发生的分层数据中使用递归。收集这些技术，并思考最佳算法的方法，可以显著提升代码的可维护性和质量。

# 思维模式

本章的几个部分可能在特定环境下很有趣，展示了解决问题的不同思考方式。你可能不会每天都使用正则表达式，但在需要时它们非常有用。另一部分讨论了如何将时间转换为/从 Unix 时间，展望了.NET 作为跨平台语言的未来，我们需要一种特定的思维方式来设计算法，这可能是我们以前从未考虑过的环境。

# 2.1 高效处理字符串

## 问题

分析器指示您的代码中有问题，它迭代地构建了一个大字符串，您需要提升性能。

## 解决方案

这是我们将要使用的`InvoiceItem`类：

```cs
public class InvoiceItem
{
    public decimal Cost { get; set; }
    public string Description { get; set; }
}
```

这种方法生成演示的示例数据：

```cs
static List<InvoiceItem> GetInvoiceItems()
{
    var items = new List<InvoiceItem>();
    var rand = new Random();
    for (int i = 0; i < 100; i++)
        items.Add(
            new InvoiceItem
            {
                Cost = rand.Next(i),
                Description = "Invoice Item #" + (i+1)
            });

    return items;
}
```

有两种处理字符串的方法。首先是低效的方法：

```cs
static string DoStringConcatenation(List<InvoiceItem> lineItems)
{
    string report = "";

    foreach (var item in lineItems)
        report += $"{item.Cost:C} - {item.Description}\n";

    return report;
}
```

下面是更高效的方法：

```cs
static string DoStringBuilderConcatenation(List<InvoiceItem> lineItems)
{
    var reportBuilder = new StringBuilder();

    foreach (var item in lineItems)
        reportBuilder.Append($"{item.Cost:C} - {item.Description}\n");

    return reportBuilder.ToString();
}
```

`Main`方法将所有这些联系在一起：

```cs
static void Main(string[] args)
{
    List<InvoiceItem> lineItems = GetInvoiceItems();

    DoStringConcatenation(lineItems);

    DoStringBuilderConcatenation(lineItems);
}
```

## 讨论

我们有不同的原因需要将数据收集到更长的字符串中。报告，无论是基于文本还是通过 HTML 或其他标记格式化，都需要组合文本字符串。有时我们将项目添加到电子邮件中，或者手动构建 PDF 内容作为电子邮件附件。其他时候，我们可能需要以非标准格式导出数据用于遗留系统。开发人员在需要时经常使用字符串连接，而`StringBuilder`则是更优的选择。

字符串连接是直观且编码速度快的操作，这就是为什么有那么多人这样做。然而，字符串连接也可能会降低应用程序的性能。问题出在每次连接都需要进行昂贵的内存分配。让我们看看如何用错误的方法和正确的方法来构建字符串。

`DoStringConcatenation` 方法中的逻辑从每个 `InvoiceItem` 中提取 `Cost` 和 `Description` 并将其连接到一个增长的字符串中。连接几个字符串可能不会被注意到。但是，想象一下如果有 25、50 或 100 行甚至更多。使用类似本章节解决方案的示例，Recipe 3.10 展示了字符串连接是一个指数级耗时操作，会严重影响应用程序性能。

###### 注意

在同一个表达式内进行连接，例如，`string1 + string2`，C# 编译器可以优化这段代码。而循环连接则会导致性能急剧下降。

`DoStringBuilderConcatenation` 方法解决了这个问题。它使用了位于 `System.Text` 命名空间中的 `StringBuilder` 类。它使用了构建器模式，如 Recipe 1.10 中描述的那样，每个 `AppendText` 都将新字符串添加到 `StringBuilder` 实例 `reportsBuilder` 中。在返回之前，该方法调用 `ToString` 方法将 `StringBuilder` 的内容转换为字符串。

###### 小贴士

一般来说，一旦您超过四个字符串连接，使用 `StringBuilder` 就能获得更好的性能。

幸运的是，.NET 生态系统中有许多 .NET Framework 库和第三方库，可以帮助处理常见格式的字符串。尽可能使用这些库，因为它们通常经过优化，可以节省时间并使代码更易于阅读。例如，表 2-1 展示了一些常见格式的库。

表 2-1\. 数据格式和库

| Data format | Library |
| --- | --- |
| JSON.NET 5 | System.Text.Json |
| JSON ⇐ .NET 4.x | Json.NET |
| XML | LINQ to XML |
| CSV | LINQ to CSV |
| HTML | System.Web.UI.HtmlTextWriter |
| PDF | 各种商业和开源提供者 |
| Excel | 各种商业和开源提供者 |

还有一个想法：自定义搜索和过滤面板通常用于为用户提供查询企业数据的简单方式。开发人员经常使用字符串连接来构建结构化查询语言（SQL）查询。虽然字符串连接更简单，但除了性能之外，它的问题在于安全性。字符串连接的 SQL 语句打开了 SQL 注入攻击的机会。在这种情况下，`StringBuilder` 不是一个解决方案。相反，你应该使用一个数据库库来对用户输入进行参数化，以避免 SQL 注入。有 ADO.NET、LINQ 提供程序和其他第三方数据库库可以为你进行输入值参数化。对于动态查询，使用数据库库可能更难，但是可以做到。你可能需要认真考虑使用 LINQ，我在 第四章 中有讨论。

## 另请参阅

配方 1.10，“构造具有复杂配置的对象”

配方 3.10，“性能测量”

第四章，“*使用 LINQ 进行查询*”

# 2.2 简化实例清理

## 问题

旧的 `using` 语句会导致不必要的嵌套，你希望清理和简化代码。

## 解决方案

这个程序有用于读写文本文件的 `using` 语句：

```cs
class Program
{
    const string FileName = "Invoice.txt";

    static void Main(string[] args)
    {
        Console.WriteLine(
            "Invoice App\n" +
            "-----------\n");

        WriteDetails();

        ReadDetails();
    }

    static void WriteDetails()
    {
        using var writer = new StreamWriter(FileName);

        Console.WriteLine("Type details and press [Enter] to end.\n");

        string detail;
        do
        {
            Console.Write("Detail: ");
            detail = Console.ReadLine();
            writer.WriteLine(detail);
        }
        while (!string.IsNullOrWhiteSpace(detail));
    }

    static void ReadDetails()
    {
        Console.WriteLine("\nInvoice Details:\n");

        using var reader = new StreamReader(FileName);

        string detail;
        do
        {
            detail = reader.ReadLine();
            Console.WriteLine(detail);
        }
        while (!string.IsNullOrWhiteSpace(detail));
    }
}
```

## 讨论

在 C# 8 之前，`using` 语句的语法要求对 `IDisposable` 对象进行实例化并且包含一个封闭的代码块。在运行时，当程序执行到封闭的代码块时，会调用实例化对象的 `Dispose` 方法。如果需要多个 `using` 语句同时操作，开发人员通常会将它们嵌套，导致除了正常语句嵌套之外还有额外的空间。对一些开发人员来说，这种模式已经足够恼人，以至于微软为语言添加了一个功能来简化 `using` 语句。

在解决方案中，你可以看到新的 `using` 语句语法出现在几个地方：在 `WriteDetails` 中实例化 `StreamWriter` 和在 `ReadDetails` 中实例化 `StreamReader`。在这两种情况下，`using` 语句都是单行的。括号和花括号已经消失，每个语句以分号结尾。

新 `using` 语句的作用域是其封闭的代码块，在执行到封闭的代码块的末尾时调用 `using` 对象的 `Dispose` 方法。在解决方案中，封闭的代码块是方法，这会导致每个 `using` 对象的 `Dispose` 方法在方法结束时被调用。

单行 `using` 语句的不同之处在于它适用于既实现 `IDisposable` 接口又实现可释放模式的对象。在这个上下文中，可释放模式意味着对象不实现 `IDisposable`，但它有一个无参数的 `Dispose` 方法。

## 另请参阅

配方 1.1，“管理对象的生命周期”

# 2.3 保持逻辑局部化

## 问题

算法具有复杂的逻辑，最好将其重构为另一个方法，但这些逻辑实际上只在一个地方使用。

## 解决方案

程序使用了`CustomerType`和`InvoiceItem`：

```cs
public enum CustomerType
{
    None,
    Bronze,
    Silver,
    Gold
}

public class InvoiceItem
{
    public decimal Cost { get; set; }
    public string Description { get; set; }
}
```

此方法生成并返回一组演示发票：

```cs
static List<InvoiceItem> GetInvoiceItems()
{
    var items = new List<InvoiceItem>();
    var rand = new Random();
    for (int i = 0; i < 100; i++)
        items.Add(
            new InvoiceItem
            {
                Cost = rand.Next(i),
                Description = "Invoice Item #" + (i + 1)
            });

    return items;
}
```

最后，`Main`方法展示了如何使用局部函数：

```cs
static void Main()
{
    List<InvoiceItem> lineItems = GetInvoiceItems();

    decimal total = 0;

    foreach (var item in lineItems)
        total += item.Cost;

    total = ApplyDiscount(total, CustomerType.Gold);

    Console.WriteLine($"Total Invoice Balance: {total:C}");

    decimal ApplyDiscount(decimal total, CustomerType customerType)
    {
        switch (customerType)
        {
            case CustomerType.Bronze:
                return total - total * .10m;
            case CustomerType.Silver:
                return total - total * .05m;
            case CustomerType.Gold:
                return total - total * .02m;
            case CustomerType.None:
            default:
                return total;
        }
    }
}
```

## 讨论

当代码仅与单个方法相关且希望隔离该代码时，局部方法非常有用。隔离代码的原因包括赋予一组复杂逻辑以意义、重用逻辑和简化调用代码（也许是一个循环），或者允许异步方法在等待封闭方法之前抛出异常。

解决方案中的`Main`方法具有一个名为`ApplyDiscount`的局部方法。此示例演示了局部方法如何简化代码。如果您检查`ApplyDiscount`中的代码，可能不会立即清楚其目的是什么。然而，通过将该逻辑分离到自己的方法中，任何人都可以阅读方法名称并知道逻辑的目的是什么。这是通过表达意图并使该逻辑局部化来使代码更易于维护的一个很好的方法，而其他开发人员不需要搜索可能在未来维护后移动的类方法。

# 2.4 在多个类上执行相同操作

## 问题

应用程序必须是可扩展的，以添加新的插件功能，但不希望为新的类重写现有代码。

## 解决方案

这是几个类共同实现的常见接口：

```cs
public interface IInvoice
{
    bool IsApproved();

    void PopulateLineItems();

    void CalculateBalance();

    void SetDueDate();
}
```

这里有几个实现了`IInvoice`接口的类：

```cs
public class BankInvoice : IInvoice
{
    public void CalculateBalance()
    {
        Console.WriteLine("Calculating balance for BankInvoice.");
    }

    public bool IsApproved()
    {
        Console.WriteLine("Checking approval for BankInvoice.");
        return true;
    }

    public void PopulateLineItems()
    {
        Console.WriteLine("Populating items for BankInvoice.");
    }

    public void SetDueDate()
    {
        Console.WriteLine("Setting due date for BankInvoice.");
    }
}

public class EnterpriseInvoice : IInvoice
{
    public void CalculateBalance()
    {
        Console.WriteLine("Calculating balance for EnterpriseInvoice.");
    }

    public bool IsApproved()
    {
        Console.WriteLine("Checking approval for EnterpriseInvoice.");
        return true;
    }

    public void PopulateLineItems()
    {
        Console.WriteLine("Populating items for EnterpriseInvoice.");
    }

    public void SetDueDate()
    {
        Console.WriteLine("Setting due date for EnterpriseInvoice.");
    }
}

public class GovernmentInvoice : IInvoice
{
    public void CalculateBalance()
    {
        Console.WriteLine("Calculating balance for GovernmentInvoice.");
    }

    public bool IsApproved()
    {
        Console.WriteLine("Checking approval for GovernmentInvoice.");
        return true;
    }

    public void PopulateLineItems()
    {
        Console.WriteLine("Populating items for GovernmentInvoice.");
    }

    public void SetDueDate()
    {
        Console.WriteLine("Setting due date for GovernmentInvoice.");
    }
}
```

此方法使用实现了`IInvoice`接口的对象填充集合：

```cs
static IEnumerable<IInvoice> GetInvoices()
{
    return new List<IInvoice>
    {
        new BankInvoice(),
        new EnterpriseInvoice(),
        new GovernmentInvoice()
    };
}
```

`Main`方法具有操作`IInvoice`接口的算法：

```cs
static void Main(string[] args)
{
    IEnumerable<IInvoice> invoices = GetInvoices();

    foreach (var invoice in invoices)
    {
        if (invoice.IsApproved())
        {
            invoice.CalculateBalance();
            invoice.PopulateLineItems();
            invoice.SetDueDate();
        }
    }
}
```

## 讨论

随着开发人员职业的进展，他们很可能会遇到客户希望应用程序具有“可扩展性”的要求。尽管即使对经验丰富的架构师来说，确切的含义也不精确，但普遍理解的是，“可扩展性”应该成为应用程序设计的一个主题。我们通常通过识别随时间可以和将会发生变化的应用程序区域来朝这个方向发展。设计模式可以帮助实现这一点，比如食谱 1.3 中的工厂类、食谱 1.4 中的工厂方法以及食谱 1.10 中的构建器。类似地，本节中描述的策略模式有助于组织可扩展性的代码。

策略模式在同时处理多种对象类型并希望它们可互换，并且希望只编写一次可以对每个对象执行相同操作的代码时非常有用。从面向对象的角度来看，这是接口多态性。我们每天使用的软件是策略模式的典型例子。办公应用程序有不同的文档类型，并允许开发人员编写自己的插件。浏览器有开发人员可以编写的插件。您每天使用的编辑器和集成开发环境（IDE）具有插件功能。

该解决方案描述了在银行、企业和政府领域中操作不同类型发票的应用程序。每个领域都有其自己的与法律或其他要求相关的业务规则。使其可扩展的原因在于，将来我们可以添加另一个处理另一个领域发票的类。

使其工作的关键是`IInvoice`接口。它包含每个实现类必须定义的必需方法（或合同）。你可以看到，`BankInvoice`、`EnterpriseInvoice`和`GovernmentInvoices`都实现了`IInvoice`。

`GetInvoices`模拟了您将从数据源填充发票的情况。每当需要通过添加新的`IInvoice`派生类型来扩展框架时，这是唯一会改变的代码。因为所有类都是`IInvoice`，所以它们都可以通过同一个`IEnumerable<IInvoice>`集合返回。

###### 注意

即使`GetInvoices`实现是在`List<IInvoice>`上操作的，它却从`GetInvoices`中返回了一个`IEnumerable<IInvoice>`。通过在这里返回一个接口`IEnumerable<T>`，调用者不对底层集合实现做任何假设。这样一来，如果将来`GetInvoices`的另一个实现更适合另一种实现类型，那么代码就可以更改而不更改方法签名，并且不会破坏调用代码。

最后，请检查`Main`方法。它迭代每个`IInvoice`对象，调用其方法。`Main`不关心具体的实现是什么，因此其代码永远不需要改变以适应特定实例的逻辑。你不需要为特殊情况编写`if`或`switch`语句，这样会在维护时导致代码复杂难以维护。任何未来的更改将涉及`Main`如何与`IInvoice`接口一起工作。与发票相关的业务逻辑的任何更改都限于发票类型本身。这样易于维护，也容易确定逻辑的存在和应有的位置。此外，通过添加实现`IInvoice`的新插件类，还可以轻松扩展。

## 参见

Recipe 1.3，“将对象创建委托给一个类”

配方 1.4，“将对象创建委托给方法”

配方 1.10，“使用复杂配置构造对象”

# 2.5 检查类型的相等性

## 问题

你需要在集合中搜索对象，而默认相等性无法胜任。

## 解决方案

`Invoice` 类实现了 `IEquatable<T>` 接口：

```cs
public class Invoice : IEquatable<Invoice>
{
    public int CustomerID { get; set; }

    public DateTime Created { get; set; }

    public List<string> InvoiceItems { get; set; }

    public decimal Total { get; set; }

    public bool Equals(Invoice other)
    {
        if (ReferenceEquals(other, null))
            return false;

        if (ReferenceEquals(this, other))
            return true;

        if (GetType() != other.GetType())
            return false;

        return
            CustomerID == other.CustomerID &&
            Created.Date == other.Created.Date;
    }

    public override bool Equals(object other)
    {
        return Equals(other as Invoice);
    }

    public override int GetHashCode()
    {
        return (CustomerID + Created.Ticks).GetHashCode();
    }

    public static bool operator ==(Invoice left, Invoice right)
    {
        if (ReferenceEquals(left, null))
            return ReferenceEquals(right, null);

        return left.Equals(right);
    }

    public static bool operator !=(Invoice left, Invoice right)
    {
        return !(left == right);
    }
}
```

此代码返回一个 `Invoice` 类的集合：

```cs
static List<Invoice> GetAllInvoices()
{
    DateTime date = DateTime.Now;

    return new List<Invoice>
    {
        new Invoice { CustomerID = 1, Created = date },
        new Invoice { CustomerID = 2, Created = date },
        new Invoice { CustomerID = 1, Created = date },
        new Invoice { CustomerID = 3, Created = date }
    };
}
```

使用 `Invoice` 类的方法如下：

```cs
static void Main(string[] args)
{
    List<Invoice> allInvoices = GetAllInvoices();

    Console.WriteLine($"# of All Invoices: {allInvoices.Count}");

    var invoicesToProcess = new List<Invoice>();

    foreach (var invoice in allInvoices)
    {
        if (!invoicesToProcess.Contains(invoice))
            invoicesToProcess.Add(invoice);
    }

    Console.WriteLine($"# of Invoices to Process: {invoicesToProcess.Count}");
}
```

## 讨论

引用类型的默认相等性语义是引用相等性，而值类型的是值相等性。引用相等性意味着当比较对象时，这些对象仅在它们的引用指向同一个确切的对象实例时才相等。值相等性发生在比较对象的每个成员之前，这两个对象才被视为相等。引用相等性的问题在于，有时你有两个相同类的实例，但实际上想要比较它们的对应成员是否相等。值相等性可能也会带来问题，因为有时你可能只想检查对象的部分内容是否相等。

为解决默认相等性不足的问题，解决方案在 `Invoice` 上实现了自定义相等性。`Invoice` 类实现了 `IEquatable<T>` 接口，其中 `T` 是 `Invoice`。尽管 `IEquatable<T>` 要求实现 `Equals(T other)` 方法，你还应该实现 `Equals(object other)`、`GetHashCode()` 方法以及 `==` 和 `!=` 操作符，以确保在所有情况下都有一致的相等性定义。

在选择一个良好的哈希码时涉及很多科学问题，这超出了本书的范围，因此解决方案的实现是最小的。

###### 注意

C# 9.0 记录（Records）默认为你提供了 `IEquatable<T>` 逻辑。然而，记录（Records）提供了值相等性，如果需要更具体的实现 `IEquatable<T>`，你需要自行实现。例如，如果你的对象具有不影响对象标识的自由文本字段，为何要浪费资源进行不必要的字段比较？另一个问题（可能更少见）可能是记录的某些部分基于时间原因会有所不同，例如临时时间戳、状态或全局唯一标识符（GUID），这将导致对象在处理过程中永远不相等。

相等性实现避免了重复的代码。`!=` 操作符调用并取反 `==` 操作符。`==` 操作符检查引用并在两个引用都为 `null` 时返回 `true`，在只有一个引用为 `null` 时返回 `false`。`==` 操作符和 `Equals(object other)` 方法都调用 `Equals(Invoice other)` 方法。

当前实例显然不是 `null`，因此 `Equals(Invoice other)` 只检查 `other` 引用，如果它是 `null`，则返回 `false`。然后检查 `this` 和 `other` 是否具有引用相等性，这显然意味着它们是相等的。然后，如果对象不是相同类型，则不被认为是相等的。最后，返回要比较的值的结果。在这个例子中，唯一有意义的是 `CustomerID` 和 `Date`。

###### 注意

`Equals(Invoice other)` 方法中可以改变的一部分是类型检查。您可能会根据应用程序的要求有不同的看法。例如，如果希望即使 `other` 是派生类型也能检查相等性，则更改逻辑以接受派生类型。

`Main` 方法处理发票，确保我们不会将重复的发票添加到列表中。循环调用集合的 `Contains` 方法，检查对象的相等性。如果没有匹配的对象，`Contains` 将新的 `Invoice` 实例添加到 `invoicesToProcess` 列表中。运行程序时，在 `allInvoices` 中存在四张发票，但只有三张添加到 `invoicesToProcess` 中，因为在 `allInvoices` 中有一个重复（基于 `CustomerID` 和 `Created`）。

# 2.6 处理数据层次结构

## 问题

应用程序需要处理层次数据，而迭代方法过于复杂和不自然。

## 解决方案

这是我们要处理的数据格式：

```cs
public class BillingCategory
{
    public int ID { get; set; }
    public string Name { get; set; }
    public int? Parent { get; set; }
}
```

此方法返回一组层次相关的记录：

```cs
static List<BillingCategory> GetBillingCategories()
{
    return new List<BillingCategory>
    {
        new BillingCategory { ID = 1, Name = "First 1",  Parent = null },
        new BillingCategory { ID = 2, Name = "First 2",  Parent = null },
        new BillingCategory { ID = 4, Name = "Second 1", Parent = 1 },
        new BillingCategory { ID = 3, Name = "First 3",  Parent = null },
        new BillingCategory { ID = 5, Name = "Second 2", Parent = 2 },
        new BillingCategory { ID = 6, Name = "Second 3", Parent = 3 },
        new BillingCategory { ID = 8, Name = "Third 1",  Parent = 5 },
        new BillingCategory { ID = 8, Name = "Third 2",  Parent = 6 },
        new BillingCategory { ID = 7, Name = "Second 4", Parent = 3 },
        new BillingCategory { ID = 9, Name = "Second 5", Parent = 1 },
        new BillingCategory { ID = 8, Name = "Third 3",  Parent = 9 }
    };
}
```

这是将扁平数据转换为层次形式的递归算法：

```cs
static List<BillingCategory> BuildHierarchy(
     List<BillingCategory> categories, int? catID, int level)
{
    var found = new List<BillingCategory>();

    foreach (var cat in categories)
    {
        if (cat.Parent == catID)
        {
            cat.Name = new string('\t', level) + cat.Name;
            found.Add(cat);
            List<BillingCategory> subCategories =
                BuildHierarchy(categories, cat.ID, level + 1);
            found.AddRange(subCategories);
        }
    }

    return found;
}
```

`Main` 方法运行程序并打印层次数据：

```cs
static void Main(string[] args)
{
    List<BillingCategory> categories = GetBillingCategories();

    List<BillingCategory> hierarchy =
        BuildHierarchy(categories, catID: null, level: 0);

    PrintHierarchy(hierarchy);
}

static void PrintHierarchy(List<BillingCategory> hierarchy)
{
    foreach (var cat in hierarchy)
        Console.WriteLine(cat.Name);
}
```

## 讨论

很难判断您将如何多次遇到迭代算法，其复杂逻辑和循环操作的条件。`for`、`foreach` 和 `while` 这样的循环是熟悉且经常使用的，即使有更优雅的解决方案存在。我并不是在暗示循环有什么问题，它们是语言工具集的重要部分。然而，对于给定情况，扩展我们的思维以探索其他可能更优雅和可维护的代码技术是有用的。有时候，像集合的 `ForEach` 操作符上的 lambda 表达式这样的声明性方法是简单明了的。LINQ 是处理内存中对象集合的良好解决方案，这是 第四章 的主题。递归是本节的另一种选择。

我在这里要表达的主要观点是，我们需要根据具体情况编写使用最自然的技术的算法。很多算法确实自然地使用循环，如遍历集合。其他任务可能需要递归。处理层次结构的一类算法可能非常适合使用递归。

此解决方案展示了递归简化处理和使代码清晰的一个领域。它基于计费处理类别列表。请注意，`BillingCategory`类具有`ID`和`Parent`两个属性。这些属性管理层次结构，其中`Parent`标识父类别。任何具有`null` `Parent`的`BillingCategory`都是顶级类别。这是单表关系数据库（DB）表示的分层数据。

`GetBillingCategories`展示了`BillingCategories`如何从数据库中获取。它是一个平面结构。请注意，`Parent`属性引用它们的父`BillingCategory`的 ID。关于数据的另一个重要事实是父子之间没有明确的排序。在实际应用中，你将从给定的类别集开始，并随后添加新的类别。同样，随着时间在代码和数据的维护中变化，这会改变我们对算法设计的方法，从而复杂化迭代解决方案。

这个解决方案的目的是将平面类别表示转换为另一个列表，该列表表示类别之间的层次关系。这是一个简单的解决方案，但你可以想象一个基于对象的表示，其中父类别包含一个子类别集合。执行此操作的递归算法是`BuildHierarchy`方法。

`BuildHierarchy`方法接受三个参数：`categories`、`catID`和`level`。`categories`参数是来自数据库的平面集合，每次递归调用都会接收到对同一集合的引用。一个潜在的优化可能是移除已经处理过的类别，尽管演示避免了任何会分散注意力的内容。`catID`参数是当前`BillingCategory`的`ID`，代码正在寻找其`Parent`匹配`catID`的所有子类别，正如`foreach`循环内部的`if`语句所示。`level`参数有助于管理每个类别的视觉表示。`if`块内的第一条语句使用`level`确定在类别名称前加上多少制表符（`\t`）。每次递归调用`BuildHierarchy`时，我们会增加`level`，以便子类别比其父类别缩进更多。

算法使用相同的类别集合调用`BuildHierarchy`。此外，它使用当前类别的`ID`，而不是`catID`参数。这意味着它递归调用`BuildHierarchy`，直到达到最底层的类别。它会通过`foreach`循环完成且没有新的类别，因为当前（最底层）类别没有子类别时，确定它位于层次结构的底部。

到达底部后，`BuildHierarchy` 返回并继续 `foreach` 循环，收集 `catID` 下的所有类别——即它们的 `Parent` 是 `catID`。然后将任何匹配的子类别附加到调用 `BuildHierarchy` 的 `found` 集合中。这将继续，直到算法达到顶层并处理所有根类别。

###### 注意

此解决方案中的递归算法称为深度优先搜索（DFS）。

到达顶层后，`BuildHierarchy` 将整个集合返回给其原始调用者，即 `Main`。`Main` 最初使用整个平面 `categories` 集合调用 `BuildHierarchy`。它将 `catID` 设置为 `null`，表示 `BuildHierarchy` 应从根级别开始。`level` 参数为 `0`，表示我们不希望在根级别类别名称上使用任何制表符前缀。这是输出：

```cs
First 1
        Second 1
        Second 5
                Third 3
First 2
        Second 2
                Third 1
First 3
        Second 3
                Third 2
        Second 4
```

回顾 `GetBillingCategories` 方法时，您可以看到视觉表示与数据匹配的方式。

# 2.7 从/到 Unix 时间的转换

## 问题

服务将日期信息发送为自 Linux 纪元以来的秒或滴答，需要转换为 C#/.NET `DateTime`。

## 解决方案

这里是我们将使用的一些值：

```cs
static readonly DateTime LinuxEpoch =
    new DateTime(1970, 1, 1, 0, 0, 0, 0);
static readonly DateTime WindowsEpoch =
    new DateTime(0001, 1, 1, 0, 0, 0, 0);
static readonly double EpochMillisecondDifference =
    new TimeSpan(
        LinuxEpoch.Ticks - WindowsEpoch.Ticks).TotalMilliseconds;
```

这些方法转换为和从 Linux 纪元时间戳：

```cs
public static string ToLinuxTimestampFromDateTime(DateTime date)
{
    double dotnetMilliseconds = TimeSpan.FromTicks(date.Ticks).TotalMilliseconds;

    double linuxMilliseconds = dotnetMilliseconds - EpochMillisecondDifference;

    double timestamp = Math.Round(
        linuxMilliseconds, 0, MidpointRounding.AwayFromZero);

    return timestamp.ToString();
}

public static DateTime ToDateTimeFromLinuxTimestamp(string timestamp)
{
    ulong.TryParse(timestamp, out ulong epochMilliseconds);
    return LinuxEpoch + +TimeSpan.FromMilliseconds(epochMilliseconds);
}
```

`Main` 方法演示如何使用这些方法：

```cs
static void Main()
{
    Console.WriteLine(
        $"WindowsEpoch == DateTime.MinValue: " +
        $"{WindowsEpoch == DateTime.MinValue}");

    DateTime testDate = new DateTime(2021, 01, 01);

    Console.WriteLine($"testDate: {testDate}");

    string linuxTimestamp = ToLinuxTimestampFromDateTime(testDate);

    TimeSpan dotnetTimeSpan =
        TimeSpan.FromMilliseconds(long.Parse(linuxTimestamp));
    DateTime problemDate =
        new DateTime(dotnetTimeSpan.Ticks);

    Console.WriteLine(
        $"Accidentally based on .NET Epoch: {problemDate}");

    DateTime goodDate = ToDateTimeFromLinuxTimestamp(linuxTimestamp);

    Console.WriteLine(
        $"Properly based on Linux Epoch: {goodDate}");
}
```

## 讨论

有时开发人员在数据库中表示日期/时间数据为毫秒或滴答。滴答以 100 纳秒为单位计量。毫秒和滴答都表示从预定义纪元开始的时间，这是计算平台的最小日期。对于.NET，纪元是 01/01/0001 00:00:00，对应解决方案中的 `WindowsEpoch` 字段。这与 `DateTime.MinValue` 相同，但以这种方式定义使示例更加明确。对于 MacOS，纪元是 1904 年 1 月 1 日，对于 Linux，纪元是 1970 年 1 月 1 日，如解决方案中的 `LinuxEpoch` 字段所示。

###### 注意

关于将 `DateTime` 值表示为毫秒或滴答作为适当设计存在各种意见。但是，我将这场辩论留给其他人和场合。我的习惯是使用我正在使用的数据库的 `DateTime` 格式。我还将 `DateTime` 转换为 UTC，因为许多应用程序需要存在超出本地时区，并且您需要一致的可转换表示。

越来越多的开发人员可能会遇到需要构建跨平台解决方案或与基于不同纪元的第三方系统集成的情况，例如，Twitter API 在其 2020 年版本 2.0 中开始使用基于 Linux 纪元的毫秒数。解决方案示例受到处理来自 Twitter API 响应的毫秒的代码启发。.NET Core 的发布为 C#开发人员提供了控制台和 ASP.NET MVC Core 应用程序的跨平台能力。.NET 5 继续跨平台故事，.NET 6 的路线图包括第一个丰富的 GUI 界面，代号 Maui。如果您习惯于仅在 Microsoft 和.NET 平台上工作，这应表明事物继续沿着未来开发所需的类型思维发展。

`ToLinuxTimestampFromDateTime`接受一个.NET `DateTime`并将其转换为 Linux 时间戳。Linux 时间戳是从 Linux 纪元开始的毫秒数。由于我们在毫秒级别工作，`TimeSpan`将`DateTime`的刻度转换为毫秒。为了进行转换，我们从.NET 时间和等效 Linux 时间之间的毫秒数中减去了数量，在`EpochMillisecondDifference`中通过从 Linux 纪元减去.NET（Windows）纪元进行预计算。转换后，我们需要将值四舍五入以消除过多的精度。默认的`Math.Round`使用所谓的银行家舍入，这通常不是我们所需要的，因此使用带有`MidpointRounding.AwayFromZero`的重载进行我们期望的舍入。解决方案将最终值作为字符串返回，您可以根据您的实现需求进行更改。

`ToDateTimeFromLinuxTimestamp`方法非常简单。将其转换为`ulong`后，它从毫秒创建一个新的时间戳，并将其加到 LinuxEpoch 上。以下是`Main`方法的输出：

```cs
WindowsEpoch == DateTime.MinValue: True
testDate: 1/1/2021 12:00:00 AM
Accidentally based on .NET Epoch: 1/2/0052 12:00:00 AM
Properly based on Linux Epoch: 1/1/2021 12:00:00 AM
```

正如您所看到的，`DateTime.MinValue`与 Windows 纪元相同。使用 2021 年 1 月 1 日作为一个好日期（至少我们希望如此），`Main`通过将该日期正确转换为 Linux 时间戳来开始。然后显示了处理该日期的错误方法。最后，它调用`ToDateTimeFromLinuxTimestamp`，执行正确的转换。

# 2.8 缓存频繁请求的数据

## 问题

网络延迟导致应用程序运行缓慢，因为静态且经常使用的数据经常被获取。

## 解决方案

这是将被缓存的数据类型：

```cs
public class InvoiceCategory
{
    public int ID { get; set; }

    public string Name { get; set; }
}
```

这是检索数据的存储库的接口：

```cs
public interface IInvoiceRepository
{
    List<InvoiceCategory> GetInvoiceCategories();
}
```

这是检索和缓存数据的存储库：

```cs
public class InvoiceRepository : IInvoiceRepository
{
    static List<InvoiceCategory> invoiceCategories;

    public List<InvoiceCategory> GetInvoiceCategories()
    {
        if (invoiceCategories == null)
            invoiceCategories = GetInvoiceCategoriesFromDB();

        return invoiceCategories;
    }

    List<InvoiceCategory> GetInvoiceCategoriesFromDB()
    {
        return new List<InvoiceCategory>
        {
            new InvoiceCategory { ID = 1, Name = "Government" },
            new InvoiceCategory { ID = 2, Name = "Financial" },
            new InvoiceCategory { ID = 3, Name = "Enterprise" },
        };
    }
}
```

这是使用该存储库的程序：

```cs
class Program
{
    readonly IInvoiceRepository invoiceRep;

    public Program(IInvoiceRepository invoiceRep)
    {
        this.invoiceRep = invoiceRep;
    }

    void Run()
        List<InvoiceCategory> categories =
            invoiceRep.GetInvoiceCategories();

        foreach (var category in categories)
            Console.WriteLine(
                $"ID: {category.ID}, Name: {category.Name}");
    }

    static void Main()
    {
        new Program(new InvoiceRepository()).Run();
    }
}
```

## 讨论

根据您使用的技术，可能有很多通过 CDN、HTTP 和数据源解决方案等机制缓存数据的选项。每种方法都有其适用的场景和目的，本节仅介绍了一种快速简单的数据缓存技术，适用于许多情况。

您可能遇到过一种情况，即在许多不同地方使用一组数据。这些数据通常是查找列表或业务规则数据的性质。在日常工作中，我们构建包含这些数据的查询，可以直接选择查询，也可以作为数据库表连接的形式存在。直到有人开始抱怨应用程序性能时，我们才会注意到这一点。分析可能会显示，有大量查询不断请求相同的数据集。如果可行，您可以将这些数据缓存在内存中，以避免网络延迟因对相同数据集的过多查询而加剧。

这并不是一个适用于所有情况的通用解决方案，因为您必须考虑在您的情况下是否实际可行。例如，在内存中保存过多数据是不现实的，这会引起其他可扩展性问题。理想情况下，这是一个有限且相对较小的数据集，例如发票类别。这些数据不应该经常更改，因为如果您需要实时访问动态数据，这种方法就行不通了。如果基础数据源发生更改，则缓存可能会保留旧的陈旧数据。

解决方案展示了一个`InvoiceCategory`类，我们将对其进行缓存。它是一个查找列表，每个对象仅有两个值，是一个有限且相对较小的集合，并且不经常变化。可以想象，每次发票查询以及包含查找列表的管理或搜索界面都需要这些数据。通过删除额外的连接并在数据库查询后加入缓存数据，可以加快发票查询速度并减少数据传输量。

解决方案中有一个`InventoryRepository`，实现了`IInvoiceRepository`接口。尽管对于这个例子来说这并不是严格必要的，但它支持展示 IoC 的另一个例子，正如 Recipe 1.2 中讨论的那样。

`InvoiceRepository`类具有一个`invoiceCategories`字段，用于保存`InvoiceCategory`的集合。`GetInvoiceCategories`方法通常会进行数据库查询并返回结果。但是，在这个例子中，只有在`invoiceCategories`为`null`时才执行数据库查询，并将结果缓存到`invoiceCategories`中。这样，后续请求将获取缓存版本，而不需要进行数据库查询。

###### 注意

`invoiceCategories` 字段是静态的，因为你只想要一个单一的缓存。在无状态的 web 场景中，如 ASP.NET 中，Internet Information Services (IIS) 进程会不可预测地回收，并建议开发人员不要依赖静态变量。这种情况不同，因为如果回收清除了 `invoiceCategories`，使其为 `null`，下一个查询将重新填充它。

`Main` 方法使用 IoC 实例化 `InvoiceRepository` 并对 `InvoiceCategory` 集合执行查询。

## 另请参阅

第 1.2 节，“移除显式依赖”

# 2.9 延迟类型实例化

## 问题

一个类具有大量的实例化要求，通过延迟实例化只在必要时节省资源使用。

## 解决方案

这是我们将要处理的数据：

```cs
public class InvoiceCategory
{
    public int ID { get; set; }

    public string Name { get; set; }
}
```

这是存储库接口：

```cs
public interface IInvoiceRepository
{
    void AddInvoiceCategory(string category);
}
```

这是我们延迟实例化的存储库：

```cs
public class InvoiceRepository : IInvoiceRepository
{
    public InvoiceRepository()
    {
        Console.WriteLine("InvoiceRepository Instantiated.");
    }

    public void AddInvoiceCategory(string category)
    {
        Console.WriteLine($"for category: {category}");
    }
}
```

此程序展示了几种延迟初始化存储库的方法：

```cs
class Program
{
    public static ServiceProvider Container;

    readonly Lazy<InvoiceRepository> InvoiceRep =
        new Lazy<InvoiceRepository>();

    readonly Lazy<IInvoiceRepository> InvoiceRepFactory =
        new Lazy<IInvoiceRepository>(CreateInvoiceRepositoryInstance);

    readonly Lazy<IInvoiceRepository> InvoiceRepIoC =
        new Lazy<IInvoiceRepository>(CreateInvoiceRepositoryFromIoC);

    static IInvoiceRepository CreateInvoiceRepositoryInstance()
    {
        return new InvoiceRepository();
    }

    static IInvoiceRepository CreateInvoiceRepositoryFromIoC()
    {
        return Container.GetRequiredService<IInvoiceRepository>();
    }

    static void Main()
    {
        Container =
            new ServiceCollection()
                .AddTransient<IInvoiceRepository, InvoiceRepository>()
                .BuildServiceProvider();

        new Program().Run();
    }

    void Run()
    {
        IInvoiceRepository viaLazyDefault = InvoiceRep.Value;
        viaLazyDefault.AddInvoiceCategory("Via Lazy Default \n");

        IInvoiceRepository viaLazyFactory = InvoiceRepFactory.Value;
        viaLazyFactory.AddInvoiceCategory("Via Lazy Factory \n");

        IInvoiceRepository viaLazyIoC = InvoiceRepIoC.Value;
        viaLazyIoC.AddInvoiceCategory("Via Lazy IoC \n");
    }
}
```

## 讨论

有时您会遇到启动开销大的对象。它们可能需要一些初始计算，或者需要等待一段时间才能获取数据，因为网络延迟或依赖性于性能不佳的外部系统。这可能会对应用程序启动速度造成严重的负面影响。想象一下，一个应用程序由于启动太慢而失去潜在客户，甚至是企业用户因等待时间而受到影响。虽然您可能无法修复性能瓶颈的根本原因，但另一种选择可能是将该对象的实例化延迟到需要时。例如，如果您真的不需要立即使用该对象，可以立即显示启动屏幕。

解决方案演示了如何使用 `Lazy<T>` 延迟对象实例化。所涉及的对象是 `InvoiceRepository`，我们假设它在构造函数逻辑上有问题，导致延迟实例化。

`Program` 有三个字段，其类型为 `Lazy<InvoiceRepository>`，展示了三种不同的实例化方式。第一个字段 `InvoiceRep`，实例化一个没有参数的 `Lazy<Invoice​Re⁠pository>`。它假设 `InvoiceRepository` 有一个默认构造函数（无参数），并在代码访问 `Value` 属性时调用它来创建一个新实例。

`InvoiceRepFactory` 字段实例引用了 `CreateInvoiceRepository​In⁠stance` 方法。当代码访问此字段时，它调用 `CreateInvoiceRepositor⁠y​Instance` 来构造对象。由于它是一个方法，你在构建对象时有很大的灵活性。

除了其他两个选项之外，`InvoiceRepIoC` 字段显示了如何在 IoC 中使用延迟实例化。注意，`Main` 方法构建了一个 IoC 容器，如第 1.2 节中所述。`CreateInvoiceRepositoryFromIoC` 方法使用该 IoC 容器请求 `InvoiceRepository` 的实例。

最后，`Run` 方法展示如何通过 `Lazy<T>.Value` 属性访问字段。

## 另请参阅

菜谱 1.2，“移除显式依赖关系”

# 2.10 解析数据文件

## 问题

应用程序需要从自定义外部格式中提取数据，而字符串类型操作导致代码复杂且效率低下。

## 解决方案

这是我们将要处理的数据类型：

```cs
public class InvoiceItem
{
    public decimal Cost { get; set; }
    public string Description { get; set; }
}

public class Invoice
{
    public string Customer { get; set; }
    public DateTime Created { get; set; }
    public List<InvoiceItem> Items { get; set; }
}
```

此方法返回我们要提取并转换为发票的原始字符串数据：

```cs
static string GetInvoiceTransferFile()
{
    return
        "Creator 1::8/05/20::Item 1\t35.05\t" +
        "Item 2\t25.18\tItem 3\t13.13::Customer 1::Note 1\n" +
        "Creator 2::8/10/20::Item 1\t45.05" +
        "::Customer 2::Note 2\n" +
        "Creator 1::8/15/20::Item 1\t55.05\t" +
        "Item 2\t65.18::Customer 3::Note 3\n";
}
```

这些是用于构建和保存发票的实用方法：

```cs
static Invoice GetInvoice(
    string matchCustomer, ..., string matchItems)
{
    List<InvoiceItem> lineItems = GetLineItems(matchItems);

    DateTime.TryParse(matchCreated, out DateTime created);

    var invoice =
        new Invoice
        {
            Customer = matchCustomer,
            Created = created,
            Items = lineItems
        };
    return invoice;
}

static List<InvoiceItem> GetLineItems(string matchItems)
{
    var lineItems = new List<InvoiceItem>();

    string[] itemStrings = matchItems.Split('\t');

    for (int i = 0; i < itemStrings.Length; i += 2)
    {
        decimal.TryParse(itemStrings[i + 1], out decimal cost);
        lineItems.Add(
            new InvoiceItem
            {
                Description = itemStrings[i],
                Cost = cost
            });
    }

    return lineItems;
}

static void SaveInvoices(List<Invoice> invoices)
{
    Console.WriteLine($"{invoices.Count} invoices saved.");
}
```

此方法使用正则表达式从原始字符串数据中提取值：

```cs
static List<Invoice> ParseInvoices(string invoiceFile)
{
    var invoices = new List<Invoice>();

    Regex invoiceRegEx = new Regex(
        @"^.+?::(?<created>.+?)::(?<items>.+?)::(?<customer>.+?)::.+");

    foreach (var invoiceString in invoiceFile.Split('\n'))
    {
        Match match = invoiceRegEx.Match(invoiceString);

        if (match.Success)
        {
            string matchCustomer = match.Groups["customer"].Value;
            string matchCreated = match.Groups["created"].Value;
            string matchItems = match.Groups["items"].Value;

            Invoice invoice =
                GetInvoice(matchCustomer, matchCreated, matchItems);
            invoices.Add(invoice);
        }
    }

    return invoices;
}
```

`Main` 方法运行演示：

```cs
static void Main(string[] args)
{
    string invoiceFile = GetInvoiceTransferFile();

    List<Invoice> invoices = ParseInvoices(invoiceFile);

    SaveInvoices(invoices);
}
```

## 讨论

有时，我们会遇到不符合标准数据格式的文本数据。它可能来自现有的文档文件、日志文件或外部和遗留系统。通常，我们需要接收这些数据并处理以便存储到数据库中。本节将解释如何使用正则表达式进行处理。

解决方案展示了我们想要生成的数据格式是一个带有 `InvoiceItem` 集合的 `Invoice`。`GetInvoiceTransferFile` 方法展示了数据的格式。演示表明数据可能来自已经生成了该格式的遗留系统，使用 C# 代码来接收比在该系统中添加支持更好的格式更容易。我们要提取的具体数据是 `created` 日期、发票 `items` 和 `customer` 名称。注意换行符 (`\n`) 分隔记录，双冒号 (`::`) 分隔发票字段，制表符 (`\t`) 分隔发票项字段。

`GetInvoice` 和 `GetLineItems` 方法从提取的数据构造对象，并用于将对象构建与正则表达式提取逻辑分离。

`ParseInvoices` 方法使用正则表达式从输入字符串中提取数值。`RegEx` 构造函数参数包含用于提取数值的正则表达式字符串。

虽然讨论整个正则表达式的内容超出了范围，但这里是该字符串的功能：

+   `^` 表示从字符串的开头开始。

+   `.+?::` 匹配所有字符，直到下一个发票字段分隔符 (`::`)。换句话说，它忽略了匹配的内容。

+   `(?<created>.+?)::`、`(?<items>.+?)::` 和 `(?<customer>.+?)::` 类似于 `.+?)::`，但进一步通过给定名称提取值到组中。例如，`(?<created>.+?)::` 表示将提取所有匹配的数据并放入名为“created”的组中。

+   `.+` 匹配所有剩余字符。

`foreach` 循环依赖于字符串中的 `\n` 分隔符来处理每个发票。`Match` 方法执行正则表达式匹配，提取数值。如果匹配成功，代码从组中提取数值，调用 `GetInvoice` 方法，并将新发票添加到 `invoices` 集合中。

您可能已经注意到，我们使用`GetLineItems`从`matchItems`参数中提取数据，来自正则表达式`items`字段。我们本可以使用更复杂的正则表达式来处理这个问题。然而，这是有意为之，用来对比展示在这种情况下正则表达式处理更为优雅的解决方案。

###### 提示

作为增强功能，如果您关心数据丢失或者想知道正则表达式或原始数据格式中是否存在 bug，可以记录任何`match.Success`为`false`的情况。

最后，应用程序将新的行项目返回给调用代码`Main`，以便保存它们。
