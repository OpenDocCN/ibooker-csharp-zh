# 第五章：实现动态和反射

反射允许代码查看类型的内部并检查其细节和成员。这对于希望给用户最大灵活性以提交对象以执行某些自动操作的库和工具非常有用。一个常见的进行反射的代码示例是单元测试框架。如 Recipe 3.1 所述，单元测试使用具有属性的类来指示哪些方法是测试方法。单元测试框架使用反射来查找测试类，定位测试方法并执行测试。

本章示例基于一个动态报表构建应用程序。它使用反射来读取类的属性，访问类型成员并执行方法。本章的前四节展示了如何做到这一点。

除了反射之外，另一种灵活处理代码的方式是 C# 中的一个特性，称为动态。在 C# 中，我们编写的大部分代码都是强类型的，这对于提高生产力和可维护性非常有益。尽管如此，C# 中有一个`dynamic`关键字，允许开发人员假设对象具有某种结构。这很像动态编程语言（如 JavaScript 和 Python），开发人员根据文档访问对象，该文档指定了对象具有哪些成员。因此，他们只需编写使用这些成员的代码。动态代码允许 C# 做到同样的事情。

在执行需要 COM 互操作的操作时，动态特别有用，有一个部分专门解释了其工作原理。您将看到动态如何能够显著减少和简化代码，相比反射的冗长和复杂性。还有一些类型允许我们构建本质上是动态的类型。此外，还有一个动态语言运行时（DLR），它允许在 C# 和动态语言（如 Python）之间进行互操作，您将看到两个关于 C# 和 Python 互操作的部分。

# 5.1 使用反射读取属性

## 问题

您希望您的库的消费者在传递对象时具有最大的灵活性，但他们仍然需要传达对象的重要细节。

## 解决方案

这是一个`Attribute`类，表示报表列元数据：

```cs
[AttributeUsage(
 AttributeTargets.Property | AttributeTargets.Method,
 AllowMultiple = false)]
public class ColumnAttribute : Attribute
{
    public ColumnAttribute(string name)
    {
        Name = name;
    }

    public string Name { get; set; }

    public string Format { get; set; }
}
```

这个类表示要显示的记录，并使用了以下属性：

```cs
public class InventoryItem
{
 [Column("Part #")]
    public string PartNumber { get; set; }

 [Column("Name")]
    public string Description { get; set; }

 [Column("Amount")]
    public int Count { get; set; }

 [Column("Price")]
    public decimal ItemPrice { get; set; }

 [Column("Total")]
    public decimal CalculateTotal()
    {
        return ItemPrice * Count;
    }
}
```

`Main` 方法展示了如何实例化和传递数据：

```cs
static void Main()
{
    var inventory = new List<object>
    {
        new InventoryItem
        {
            PartNumber = "1",
            Description = "Part #1",
            Count = 3,
            ItemPrice = 5.26m
        },
        new InventoryItem
        {
            PartNumber = "2",
            Description = "Part #2",
            Count = 1,
            ItemPrice = 7.95m
        },
        new InventoryItem
        {
            PartNumber = "3",
            Description = "Part #3",
            Count = 2,
            ItemPrice = 23.13m
        },
    };

    string report = new Report().Generate(inventory);

    Console.WriteLine(report);
}
```

这个`Report`类有用于构建报表头和生成报表的方法：

```cs
public class Report
{
    // contains Generate and GetHeaders methods
}
```

这个方法是`Report`类的一个成员，使用反射来找到所有类型成员：

```cs
public string Generate(List<object> items)
{
    _ = items ??
        throw new ArgumentNullException(
            $"{nameof(items)} is required");

    MemberInfo[] members =
        items.First().GetType().GetMembers();

    var report = new StringBuilder("# Report\n\n");

    report.Append(GetHeaders(members));

    return report.ToString();
}
```

这个方法是`Report`类的一个成员，使用反射来读取类型的属性：

```cs
const string ColumnSeparator = " | ";

StringBuilder GetHeaders(MemberInfo[] members)
{
    var columnNames = new List<string>();
    var underscores = new List<string>();

    foreach (var member in members)
    {
        var attribute =
            member.GetCustomAttribute<ColumnAttribute>();

        if (attribute != null)
        {
            string columnTitle = attribute.Name;
            string dashes = "".PadLeft(columnTitle.Length, '-');

            columnNames.Add(columnTitle);
            underscores.Add(dashes);
        }
    }

    var header = new StringBuilder();

    header.AppendJoin(ColumnSeparator, columnNames);
    header.Append("\n");

    header.AppendJoin(ColumnSeparator, underscores);
    header.Append("\n");

    return header;
}
```

下面是输出结果：

```cs
# Report

Total | Part # | Name | Amount | Price
----- | ------ | ---- | ------ | -----
```

## 讨论

属性，即元数据，通常存在于代码工具中。本节中的解决方案采用类似的方法，其中 `ColumnAttribute` 是报表中数据列的元数据。你可以看到 `AttributeUsage` 指定了可以将 `ColumnAttribute` 应用于属性或方法。考虑报表列可能支持的功能，这个属性归结为两个典型的特征：`Name` 和 `Format`。因为 C# 属性名称可能不代表列标题的文本，`Name` 允许你指定任何你想要的内容。此外，如果不指定字符串格式，`DateTime` 和 `decimal` 列将采用默认显示方式，通常这并非你所期望的。这本质上解决了报表库的使用者想要传递任何类型对象的问题，通过使用 `ColumnAttribute` 共享重要的细节。

`InventoryItem` 展示了 `ColumnAttribute` 的工作原理。注意位置属性 `Name` 如何与属性和方法的名称不同。Recipe 5.2 展示了 `Format` 属性如何工作的例子，而本节仅集中于如何提取和显示 Markdown 格式列的元数据。

###### 注意

从架构上讲，你应该把这个项目看作两个独立的应用程序。有一个可重用的报表库，任何人都可以提交对象。报表库包括一个 `Report` 类和 `ColumnAttribute` 属性。然后有一个消费应用程序，即 `Main` 方法。为简单起见，本演示的源代码将所有代码放在同一个项目中，但在实践中，这些应该是分开的。

`Main` 方法实例化一个包含 `InventoryItem` 实例的 `List<object>`。这些数据通常来自数据库或其他数据源。它实例化 `Report`，传递数据，并打印结果。

`Generate` 方法属于 `Report` 类。注意它接受一个 `List<object>`，这就是为什么 `Main` 传递了一个 `List<object>`。实质上，`Report` 希望能够操作任何对象类型。

在验证输入项后，`Generate` 使用反射来发现传递对象中存在的成员。你看，我们再也无法知道因为对象不是强类型的，而且我们希望能够在可以传递任何类型时具有最大的灵活性。这是反射的一个好案例。话虽如此，我们不再保证所有项实例是相同类型，这必须是一个隐含的约定，而不是由代码强制执行的。Recipe 5.3 通过展示如何使用泛型来同时具有类型安全性和使用泛型的能力来解决这个问题，使用接口可能是另一种方法。

我们假设所有对象都相同，并且 `Generate` 在 `items` 上调用 `First`，因为所有对象在 `items` 中具有相同的属性。然后 `Generate` 在第一个项目上调用 `GetType`。 `Type` 实例是执行反射的门户。

获得 `Type` 实例之后，您可以查询有关类型的任何信息，并使用特定实例。此示例调用 `GetMembers` 获取 `MemberInfo[]`。 `MemberInfo` 包含有关特定类型成员的所有信息，例如其名称和类型。在此示例中，`MemberInfo[]` 包含从 `Main` 传入的 `InventoryItem` 的属性和方法：`PartNumber`、`Description`、`Count`、`ItemPrice` 和 `CalculateTotal`。

因为报告是 Markdown 文本的字符串，并且有很多连接操作，解决方案使用 `StringBuilder`。第 2.1 节 解释了为什么这是一个好方法。

因为我们关注属性，此解决方案仅打印报告头部，本章后面的部分将详细解释根据您的需求生成报告主体的多种不同方法。 `GetHeader` 方法接收 `MemberInfo[]` 并使用反射来确定这些标题应该是什么。

在 Markdown 中，我们用管道符号 `|` 分隔表头，并添加下划线，这就是为什么我们在 `columnNames` 和 `underscores` 中有两个数组。 `foreach` 循环检查每个 `MemberInfo`，调用 `GetCustomAttribute`。注意 `GetCustomAttribute` 的类型参数是 `ColumnAttribute` —— 成员可能有多个属性，但我们只需要这一个。从 `GetCustomAttribute` 返回的实例是 `ColumnAttribute`，所以我们可以访问它的属性，比如 `Name`。代码使用 `Name` 填充 `columnNames`，并添加与 `Name` 长度相同的下划线。

最后，`GetHeaders` 使用管道符号 `|` 连接值并返回结果头部。通过调用链追溯回来，`Generate` 添加了 `GetHeaders` 的结果，而 `Main` 打印了标题，您可以在解决方案输出中看到。

## 参见

第 2.1 节，“高效处理字符串”

第 5.2 节，“使用反射访问类型成员”

# 5.2 使用反射访问类型成员

## 问题

您需要检查对象以查看可以读取哪些属性。

## 解决方案

这个类表示要显示的记录：

```cs
public class InventoryItem
{
 [Column("Part #")]
    public string PartNumber { get; set; }

 [Column("Name")]
    public string Description { get; set; }

 [Column("Amount")]
    public int Count { get; set; }

 [Column("Price", Format = "{0:c}")]
    public decimal ItemPrice { get; set; }
}
```

这是一个包含每个报告列元数据的类：

```cs
public class ColumnDetail
{
    public string Name { get; set; }

    public ColumnAttribute Attribute { get; set; }

    public PropertyInfo PropertyInfo { get; set; }
}
```

此方法收集数据以填充列元数据：

```cs
Dictionary<string, ColumnDetail> GetColumnDetails(
    List<object> items)
{
     object itemInstance = items.First();
     Type itemType = itemInstance.GetType();
     PropertyInfo[] itemProperties = itemType.GetProperties();

    return
        (from prop in itemProperties
         let attribute = prop.GetCustomAttribute<ColumnAttribute>()
         where attribute != null
         select new ColumnDetail
         {
             Name = prop.Name,
             Attribute = attribute,
             PropertyInfo = prop
         })
        .ToDictionary(
            key => key.Name,
            val => val);
}
```

这是使用 LINQ 更简洁获取头部数据的方式：

```cs
StringBuilder GetHeaders(
    Dictionary<string, ColumnDetail> details)
{
    var header = new StringBuilder();

    header.AppendJoin(
        ColumnSeparator,
        from detail in details.Values
        select detail.Attribute.Name);

    header.Append("\n");

    header.AppendJoin(
        ColumnSeparator,
        from detail in details.Values
        let length = detail.Attribute.Name.Length
        select "".PadLeft(length, '-'));

    header.Append("\n");

    return header;
}
```

此方法使用反射从对象属性中提取值：

```cs
(object, Type) GetReflectedResult(
    object item, PropertyInfo property)
{
    object result = property.GetValue(item);
    Type type = property.PropertyType;

    return (result, type);
}
```

此方法使用反射检索和格式化属性数据：

```cs
List<string> GetColumns(
    IEnumerable<ColumnDetail> details,
    object item)
{
    var columns = new List<string>();

    foreach (var detail in details)
    {
        PropertyInfo member = detail.PropertyInfo;
        string format =
            string.IsNullOrWhiteSpace(
                detail.Attribute.Format) ?
                "{0}" :
                detail.Attribute.Format;

        (object result, Type columnType) =
            GetReflectedResult(item, member);

        switch (columnType.FullName)
        {
            case "System.Decimal":
                columns.Add(
                    string.Format(format, (decimal)result));
                break;
            case "System.Int32":
                columns.Add(
                    string.Format(format, (int)result));
                break;
            case "System.String":
                columns.Add(
                    string.Format(format, (string)result));
                break;
            default:
                break;
        }
    }

    return columns;
}
```

此方法组合和格式化所有数据行：

```cs
StringBuilder GetRows(
    List<object> items,
    Dictionary<string, ColumnDetail> details)
{
    var rows = new StringBuilder();

    foreach (var item in items)
    {
        List<string> columns =
            GetColumns(details.Values, item);

        rows.AppendJoin(ColumnSeparator, columns);

        rows.Append("\n");
    }

    return rows;
}
```

最后，这个方法使用所有其他方法来构建完整的报告：

```cs
const string ColumnSeparator = " | ";

public string Generate(List<object> items)
{
    var report = new StringBuilder("# Report\n\n");

    Dictionary<string, ColumnDetail> columnDetails =
        GetColumnDetails(items);
    report.Append(GetHeaders(columnDetails));
    report.Append(GetRows(items, columnDetails));

    return report.ToString();
}
```

这是输出：

```cs
|| Total | Part # | Name | Amount | Price ||
| $15.78 | 1 | Part #1 | 3 | 5.26 |
| $7.95 | 2 | Part #2 | 1 | 7.95 |
| $46.26 | 3 | Part #3 | 2 | 23.13 |
```

## 讨论

解决方案中的报表库接收一个`List<object>`，以便消费者可以发送任何类型的对象。由于输入对象没有强类型，`Report`类需要执行反射以从每个对象中提取数据。Recipe 5.1 解释了`Main`方法如何传递这些数据以及解决方案如何生成标题。本节关注数据，解决方案不会重复来自 Recipe 5.1 的确切代码。

`InventoryItem`类使用`ColumnAttribute`属性。请注意，`ItemPrice`现在具有名为`Format`的属性，指定该列应在报表中格式化为货币。

在反射过程中，我们需要从对象中提取一组数据，以帮助报表布局和格式化。`ColumnDetail`对此很有帮助，因为在处理每列时，我们需要知道：

+   `Name`以确保我们正在处理正确的列

+   用于格式化列数据的`Attribute`

+   用于获取属性数据的`PropertyInfo`

`GetColumnDetails`方法为每列填充一个`ColumnDetail`。获取数据中的第一个对象，获取其类型，然后在类型上调用`GetProperties`以获取`PropertyInfo[]`。与 Recipe 5.1 不同，后者调用`GetMembers`以获取`MemberInfo[]`，此方法仅从类型获取属性，而不是其他任何成员。

###### 提示

除了`GetMembers`和`GetProperties`之外，`Type`还具有其他反射方法，仅获取构造函数、字段或方法。如果需要限制正在处理的成员类型，则这些方法会很有用。

因为反射返回一组对象（在本解决方案中为`PropertyInfo[]`），我们可以使用 LINQ to Objects 进行更声明性的方法。这就是`GetColumnDetails`所做的事情，将投影到`ColumnDetails`实例中，并返回以列名为键，`ColumnDetail`为值的`Dictionary`。

###### 注意

如您稍后在解决方案中看到的那样，代码通过`Dictionary<string, ColumnDetail>`迭代，假设列及其数据按反射查询返回的顺序排列。但是，请想象一下未来的实现，其中`ColumnAttribute`具有`Order`属性，或者消费者可以传递`include/exclude`列元数据，这并不保证列的顺序与反射返回的顺序相匹配。在这种情况下，具有字典是至关重要的，可以根据您正在处理的列查找`ColumnDetail`元数据。尽管这个示例中没有涉及这一点以减少复杂性并集中于原始问题陈述，但它可能会给您一些如何扩展类似功能的想法。

`GetHeaders`方法与 Recipe 5.1 完全相同，只是它以 LINQ 语句编写，以减少和简化代码。

`GetReflectedResult` 返回一个元组，`(object, Type)`。其任务是从其 `PropertyInfo` 中提取属性的值和属性的类型。在这里，`item` 是实际的对象实例，`property` 是该属性的反射元数据。使用 `property`，代码使用 `item` 作为参数调用 `GetValue` —— 从 `item` 中读取该属性。再次强调，我们使用反射，并不知道属性的类型，因此将其放在类型对象中。`PropertyInfo` 还具有 `PropertyType`，我们可以从中获取 `Type` 对象。

###### 警告

该应用程序使用反射将属性数据放入 `object` 类型的变量中。如果属性类型是值类型（例如 `int`、`double`、`decimal`），则会产生装箱开销，这会影响应用程序的性能。如果您执行这种操作数百万次，您可能需要重新审视您的需求，并分析是否这对您的情况是一个好方法。尽管如此，这只是一份报告。考虑一下您可能为了向人显示数据而在报告中包含多少记录。在这种情况下，任何性能问题都将是微不足道的。这是灵活性与性能之间的经典权衡；您只需考虑它如何影响您的情况。

`GetColumns` 方法在遍历给定对象的每一列时使用 `GetReflectedResult`。`ColumnDetail` 的集合很有用，为当前列提供 `PropertyInfo`。如果列的 `ColumnAttribute` 不包括 `Format` 属性，则格式默认为无格式。`switch` 语句根据 `GetReflectedResult` 返回的 `Type` 对象对对象应用格式。

###### 注意

为简单起见，在 `GetColumns` 的 `switch` 语句中仅包含解决方案中的类型，尽管您可以想象它包含所有内置类型。我们可以使用反射调用 `ToString`，并带有格式说明符和类型，在 Recipe 5.4 中讨论，以减少代码量。然而，某些情况下，额外的复杂性并不增加价值。在这种情况下，我们只涵盖了一组有限的内置类型，一旦编写了该代码，它将不太可能更改。我对这种权衡的看法是，有时过于聪明会导致难以阅读且写作时间更长的代码。

最后，`GetRows` 为每一行调用 `GetColumns` 并返回给 `Generate`。然后，在调用了 `GetHeaders` 和 `GetRows` 后，`Generate` 将结果追加到 `StringBuilder` 并将字符串返回给调用者，其中包含整个报告，您可以在解决方案输出中看到。

## 参见

Recipe 5.1, “使用反射读取属性”

Recipe 5.4, “使用反射调用方法”

# 5.3 使用反射实例化类型成员

## 问题

您需要实例化泛型类型，但事先不知道类型或类型参数。

## 解决方案

解决方案生成根据此枚举唯一格式化的报告：

```cs
public enum ReportType
{
    Html,
    Markdown
}
```

这里是一个可重用的生成报告的基类：

```cs
public abstract class GeneratorBase<TData>
{
    public string Generate(List<TData> items)
    {
        StringBuilder report = GetTitle();

        Dictionary<string, ColumnDetail> columnDetails =
            GetColumnDetails(items);
        report.Append(GetHeaders(columnDetails));
        report.Append(GetRows(items, columnDetails));

        return report.ToString();
    }

    protected abstract StringBuilder GetTitle();

    protected abstract StringBuilder GetHeaders(
        Dictionary<string, ColumnDetail> details);

    protected abstract StringBuilder GetRows(
        List<TData> items,
        Dictionary<string, ColumnDetail> details);

    Dictionary<string, ColumnDetail> GetColumnDetails(
        List<TData> items)
    {
        TData itemInstance = items.First();
        Type itemType = itemInstance.GetType();
        PropertyInfo[] itemProperties = itemType.GetProperties();

        return
            (from prop in itemProperties
             let attribute = prop.GetCustomAttribute<ColumnAttribute>()
             where attribute != null
             select new ColumnDetail
             {
                 Name = prop.Name,
                 Attribute = attribute,
                 PropertyInfo = prop
             })
            .ToDictionary(
                key => key.Name,
                val => val);
    }

    protected List<string> GetColumns(
        IEnumerable<ColumnDetail> details,
        TData item)
    {
        var columns = new List<string>();

        foreach (var detail in details)
        {
            PropertyInfo member = detail.PropertyInfo;
            string format =
                string.IsNullOrWhiteSpace(
                    detail.Attribute.Format) ?
                    "{0}" :
                    detail.Attribute.Format;

            (object result, Type columnType) =
                GetReflectedResult(item, member);

            switch (columnType.Name)
            {
                case "Decimal":
                    columns.Add(
                        string.Format(format, (decimal)result));
                    break;
                case "Int32":
                    columns.Add(
                        string.Format(format, (int)result));
                    break;
                case "String":
                    columns.Add(
                        string.Format(format, (string)result));
                    break;
                default:
                    break;
            }
        }

        return columns;
    }

    (object, Type) GetReflectedResult(TData item, PropertyInfo property)
    {
        object result = property.GetValue(item);
        Type type = property.PropertyType;

        return (result, type);
    }
}
```

此类使用基类生成 Markdown 报告：

```cs
public class MarkdownGenerator<TData> : GeneratorBase<TData>
{
    const string ColumnSeparator = " | ";

    protected override StringBuilder GetTitle()
    {
        return new StringBuilder("# Report\n\n");
    }

    protected override StringBuilder GetHeaders(
        Dictionary<string, ColumnDetail> details)
    {
        var header = new StringBuilder();

        header.AppendJoin(
            ColumnSeparator,
            from detail in details.Values
            select detail.Attribute.Name);

        header.Append("\n");

        header.AppendJoin(
            ColumnSeparator,
            from detail in details.Values
            let length = detail.Attribute.Name.Length
            select "".PadLeft(length, '-'));

        header.Append("\n");

        return header;
    }

    protected override StringBuilder GetRows(
        List<TData> items,
        Dictionary<string, ColumnDetail> details)
    {
        var rows = new StringBuilder();

        foreach (var item in items)
        {
            List<string> columns =
                GetColumns(details.Values, item);

            rows.AppendJoin(ColumnSeparator, columns);

            rows.Append("\n");
        }

        return rows;
    }
}
```

而这个类则使用基类生成 HTML 报告：

```cs
public class HtmlGenerator<TData> : GeneratorBase<TData>
{
    protected override StringBuilder GetTitle()
    {
        return new StringBuilder("<h1>Report</h1>\n");
    }

    protected override StringBuilder GetHeaders(
        Dictionary<string, ColumnDetail> details)
    {
        var header = new StringBuilder("<tr>\n");

        header.AppendJoin(
            "\n",
            from detail in details.Values
            let columnName = detail.Attribute.Name
            select $"  <th>{columnName}</th>");

        header.Append("\n</tr>\n");

        return header;
    }

    protected override StringBuilder GetRows(
        List<TData> items,
        Dictionary<string, ColumnDetail> details)
    {
        StringBuilder rows = new StringBuilder();
        Type itemType = items.First().GetType();

        foreach (var item in items)
        {
            rows.Append("<tr>\n");

            List<string> columns =
                GetColumns(details.Values, item);

            rows.AppendJoin(
                "\n",
                from columnValue in columns
                select $"  <td>{columnValue}</td>");

            rows.Append("\n</tr>\n");
        }

        return rows;
    }
}
```

这种方法，来自`Report`类，管理报告生成过程：

```cs
public string Generate(List<TData> items, ReportType reportType)
{
    GeneratorBase<TData> generator = CreateGenerator(reportType);

    string report = generator.Generate(items);

    return report;
}
```

这是一个从`Report`类中使用枚举来确定要生成的报告格式的方法：

```cs
GeneratorBase<TData> CreateGenerator(ReportType reportType)
{
    Type generatorType;

    switch (reportType)
    {
        case ReportType.Html:
            generatorType = typeof(HtmlGenerator<>);
            break;
        case ReportType.Markdown:
            generatorType = typeof(MarkdownGenerator<>);
            break;
        default:
            throw new ArgumentException(
                $"Unexpected ReportType: '{reportType}'");
    }

    Type dataType = typeof(TData);
    Type genericType = generatorType.MakeGenericType(dataType);

    object generator = Activator.CreateInstance(genericType);

    return (GeneratorBase<TData>)generator;
}
```

这是另一种通过约定来确定要生成的报告格式的方法：

```cs
GeneratorBase<TData> CreateGenerator(ReportType reportType)
{
    Type dataType = typeof(TData);

    string generatorNamespace = "Section_05_03.";
    string generatorTypeName = $"{reportType}Generator`1";
    string typeParameterName = $"[[{dataType.FullName}]]";

    string fullyQualifiedTypeName =
        generatorNamespace +
        generatorTypeName +
        typeParameterName;

    Type generatorType = Type.GetType(fullyQualifiedTypeName);

    object generator = Activator.CreateInstance(generatorType);

    return (GeneratorBase<TData>)generator;
}
```

`Main`方法传递数据并指定要生成的报告格式：

```cs
static void Main()
{
    var inventory = new List<InventoryItem>
    {
        new InventoryItem
        {
            PartNumber = "1",
            Description = "Part #1",
            Count = 3,
            ItemPrice = 5.26m
        },
        new InventoryItem
        {
            PartNumber = "2",
            Description = "Part #2",
            Count = 1,
            ItemPrice = 7.95m
        },
        new InventoryItem
        {
            PartNumber = "3",
            Description = "Part #3",
            Count = 2,
            ItemPrice = 23.13m
        },
    };

    string report =
        new Report<InventoryItem>()
        .Generate(inventory, ReportType.Markdown);

    Console.WriteLine(report);
}
```

这是输出结果：

```cs
# Report

Part # | Name | Amount | Price
------ | ---- | ------ | -----
1 | Part #1 | 3 | $5.26
2 | Part #2 | 1 | $7.95
3 | Part #3 | 2 | $23.13
```

## 讨论

Recipe 5.2 基于一个通用对象类型创建报告，这导致我们失去了我们习惯的类型安全性。本节通过使用泛型来修复这个问题，并展示如何使用反射来实例化具有泛型类型参数的对象。

前面几节的概念是生成 Markdown 格式的报告。但是，如果报告生成器能够根据您选择的任何格式生成报告，它可能会更有用。本示例重构了 Recipe 5.2 中的示例，以提供 Markdown 和 HTML 输出报告。

枚举`ReportType`指定要生成的报告输出类型：`Html`或`Markdown`。因为我们可以生成多种格式，所以我们需要为每种格式单独创建类：`HtmlGenerator`和`MarkdownGenerator`。此外，我们不想重复代码，所以每个格式生成类都继承自`GeneratorBase`。

注意，`GeneratorBase`是一个抽象类（无法实例化），具有抽象和已实现的方法。在`GeneratorBase`中已实现的方法具有与输出格式无关的代码，并且所有派生的生成器类都将使用这些方法：`GetColumns`、`GetColumnDetails`和`GetReflectedResult`。根据定义，派生的生成器类必须覆盖这些特定于格式的抽象方法：`GetTitle`、`GetHeaders`和`GetRows`。查看`HtmlGenerator`和`MarkdownGenerator`，你可以看到这些抽象方法的覆盖实现。

现在，让我们将这一切整合起来，使其有意义。当程序启动时，在 `Report` 实例上调用的第一个方法是 `Generate`，在 `GeneratorBase` 中。注意 `Generate` 调用的顺序：`GetTitle`、`GetColumnDetails`、`GetHeaders`，然后是 `GetRows`。这基本上与 Recipe 5.2 中描述的顺序相同。你可以想象通过编写标题、获取其余报告的元数据、编写标题，并逐行编写报告的每一行来生成报告。为了实现代码重用并创建未来添加报告格式的可扩展框架，我们有一个通用的抽象基类 `GeneratorBase`，以及理解格式的派生类。以 `MarkdownGenerator` 为例，这是一个示例：

1.  外部代码调用 `GeneratorBase.Generate`。

1.  `Generator.Generate` 调用 `MarkdownGenerator.GetTitle`。

1.  `Generator.Generate` 调用 `Generator.GetColumnDetails`。

1.  `Generator.Generate` 调用 `MarkdownGenerator.GetHeader`。

1.  `Generator.Generate` 调用 `MarkdownGenerator.GetRows`。

1.  `MarkdownGenerator.GetRows` 调用 `Generator.GetColumns`。

1.  `Generator.GetColumns` 调用 `Generator.GetReflectedResult`。

1.  `MarkdownGenerator.GetRows` 完成，返回至 `Generator.Generate`。

1.  `Generator.Generate` 将报告返回给调用代码。

`HtmlGenerator` 的工作方式完全相同，未来的任何报告格式也将如此。事实上，Recipe 5.6 通过添加第三种格式来扩展此示例，以支持创建 Excel 报告。

###### 注意

该解决方案使用一种称为*模板模式*的模式。在这种模式中，基类实现通用逻辑，并将实现特定工作委托给派生类。这体现了面向对象的多态原则。

我们可以扩展这个框架，而无需重写样板逻辑，这使得这种方法变得可行。Recipe 5.6 展示了其工作原理。

`GenerateBase` 类故意声明为 `abstract`，因为唯一的工作方式是通过派生类的实例。`Report.Generate` 方法调用 `GeneratorBase.Generate`。在此之前，它必须确定通过 `CreateGenerator` 实例化哪个具体的 `GeneratorBase` 派生类，其中有两个示例。

`CreateGenerator`的第一个示例检查`ReportType`枚举，以查看通过`switch`语句生成哪种类型的报告。正如前面部分所述，执行反射需要`Type`对象，`typeof`操作符就是这样做的。请注意，我们传递了一个带有`<>`后缀的泛型类型，而没有泛型类型。之后，我们使用`typeof`操作符获取传递给`Report`类的类型参数`TData`的类型。现在我们有了泛型类型和其类型参数的类型。接下来，我们需要将泛型类型和其参数类型结合在一起，以获得一个完全构造的类型（例如，对于`Html`是`HtmlGenerator<TData>`）。一旦你有了完全构造的类型，你可以使用`Activator`类调用`CreateInstance`来实例化该类型。通过`GeneratorBase`派生类型的新实例，`CreateGenerate`返回到`ReportGenerate`，在新实例上调用`Generate`。正如你之前学到的，`GeneratorBase`为所有派生实例实现了`Generate`。

这是使用反射来实例化泛型类型的一种方法，如问题陈述所指定的。不过，需要考虑的一点是，你是否希望在将来添加更多格式支持？你将不得不返回到`Report`类并更改`switch`语句，这是一种通过代码更改的配置。如果你希望只编写一次`Report`类并且永远不再修改它呢？此外，如果你更喜欢通过约定优于配置的原则进行设计呢？.NET 中约定优于配置的一个很好的例子是 ASP.NET MVC。ASP.NET MVC 的一些约定包括控制器放在`Controllers`文件夹中，视图放在`Views`文件夹中。另一个约定是控制器名称是 URL 路径，其名称带有`Controller`后缀。这些事情只是因为这是约定而工作。`CreateGenerator`的第二个例子使用了约定优于配置的方法。

注意，`CreateGenerator`的第二个实现会构建一个带有命名空间和类型名称的完全限定类型名（例如，对于`Html`是`Section_05_03.HtmlGenerator`）。还请注意，`ReportType`枚举成员与类名完全匹配。这意味着将来任何时候，你都可以创建一个新格式，从`GeneratorBase`派生，并将前缀添加到`ReportType`，以`Generator`作为后缀，它将起作用。除非添加新功能，否则不需要再次触及`Report`类。

在获取类型对象之后，两个`CreateGenerator`示例都调用`Activator.CreateInstance`返回一个新实例给`Report.Generate`。

最后，看看`Main`方法，这个报告库的用户所需做的就是传入数据和他们想要生成的`ReportType`。

## 参见

5.2 节，“使用反射访问类型成员”

5.6 节，“与 Office 应用程序进行 Interop”

# 5.4 使用反射调用方法

## 问题

您收到的对象具有需要调用的方法。

## 解决方案

列元数据类具有`MemberInfo`属性：

```cs
public class ColumnDetail
{
    public string Name { get; set; }

    public ColumnAttribute Attribute { get; set; }

    public MemberInfo MemberInfo { get; set; }
}
```

这个类，要进行反射，有属性和一个方法：

```cs
public class InventoryItem
{
 [Column("Part #")]
    public string PartNumber { get; set; }

 [Column("Name")]
    public string Description { get; set; }

 [Column("Amount")]
    public int Count { get; set; }

 [Column("Price", Format = "{0:c}")]
    public decimal ItemPrice { get; set; }

 [Column("Total", Format = "{0:c}")]
    public decimal CalculateTotal()
    {
        return ItemPrice * Count;
    }
}
```

此方法调用`GetMembers`以处理`MemberInfo`实例：

```cs
Dictionary<string, ColumnDetail> GetColumnDetails(
    List<object> items)
{
    return
        (from member in
            items.First().GetType().GetMembers()
         let attribute =
            member.GetCustomAttribute<ColumnAttribute>()
         where attribute != null
         select new ColumnDetail
         {
             Name = member.Name,
             Attribute = attribute,
             MemberInfo = member
         })
        .ToDictionary(
            key => key.Name,
            val => val);
}
```

此方法使用`MemberInfo`类型来确定如何检索值：

```cs
(object, Type) GetReflectedResult(
    Type itemType, object item, MemberInfo member)
{
    object result;
    Type type;

    switch (member.MemberType)
    {
        case MemberTypes.Method:
            MethodInfo method =
                itemType.GetMethod(member.Name);
            result = method.Invoke(item, null);
            type = method.ReturnType;
            break;
        case MemberTypes.Property:
            PropertyInfo property =
                itemType.GetProperty(member.Name);
            result = property.GetValue(item);
            type = property.PropertyType;
            break;
        default:
            throw new ArgumentException(
                "Expected property or method.");
    }

    return (result, type);
}
```

## 讨论

本章前几节主要使用属性作为报表输入。在本节中，我们将修改[Recipe 5.2](https://example.org/#accessing_type_members_with_reflection)中的示例，并添加我们需要通过反射调用的方法。

第一个变化是`ColumnDetail`有一个`MemberInfo`属性，该属性保存了任何类型成员的元数据。

`InventoryItem`类有一个`CalculateTotal`方法。它将`ItemPrice`和`Count`相乘，显示该数量物品的总价格。

在 LINQ 语句中，`GetColumnDetails`的变化在于它迭代了`GetMembers`的结果，这是一个`MemberInfo[]`。与[Recipe 5.2](https://example.org/#accessing_type_members_with_reflection)不同，我们使用`MemberInfo`。这对于解决方案是必需的，因为我们希望获取属性和方法的信息。

最后，`GetReflectedResult`有一个`switch`语句来确定如何获取成员的值。由于参数是`MemberInfo`，我们查看`MemberType`属性来确定是否在处理属性或方法。无论哪种情况，我们都必须调用`GetProperty`或`GetMethod`来获取`PropertyInfo`或`MethodInfo`。对于方法，调用`Invoke`方法，参数是`item`作为要调用方法的对象实例。`Invoke`的第二个参数是`null`，表示此示例中的方法`CalculateTotal`没有参数。如果需要传递参数，请将`object[]`放入`Invoke`的第二个参数中，按照方法期望的顺序传递成员。如同[Recipe 5.2](https://example.org/#accessing_type_members_with_reflection)一样，在`Proper⁠ty​Info`实例上调用`GetValue`，使用`item`作为对象引用来获取该属性的值。

总结一下，每当您需要通过反射调用对象的方法时，获取其`Type`对象，获取一个`MethodInfo`（即使您需要从`MemberInfo`中获取中间步骤），然后调用`Invoke`方法，并将对象实例作为参数传递给`MethodInfo`。

## 参见

[Recipe 5.2，“使用反射访问类型成员”](https://example.org/#accessing_type_members_with_reflection)

# 5.5 用动态代码替换反射

## 问题

当您使用反射时，但了解某个类型的一些成员，并希望简化代码时。

## 解决方案

此类包含报表的数据列表：

```cs
public class Inventory
{
    public string Title { get; set; }

    public List<object> Data { get; set; }
}
```

这是填充数据的`Main`方法：

```cs
static void Main()
{
    var inventory = new Inventory
    {
        Title = "Inventory Report",
        Data = new List<object>
        {
            new InventoryItem
            {
                PartNumber = "1",
                Description = "Part #1",
                Count = 3,
                ItemPrice = 5.26m
            },
            new InventoryItem
            {
                PartNumber = "2",
                Description = "Part #2",
                Count = 1,
                ItemPrice = 7.95m
            },
            new InventoryItem
            {
                PartNumber = "3",
                Description = "Part #3",
                Count = 2,
                ItemPrice = 23.13m
            },
        }
    };

    string report = new Report().Generate(inventory);

    Console.WriteLine(report);
}
```

此方法使用反射来提取属性的值：

```cs
public string Generate(object reportDetails)
{
    Type reportType = reportDetails.GetType();
    PropertyInfo titleProp = reportType.GetProperty("Title");
    string title = (string)titleProp.GetValue(reportDetails);

    var report = new StringBuilder($"# {title}\n\n");

    PropertyInfo dataProp = reportType.GetProperty("Data");
    List<object> items =
        (List<object>)dataProp.GetValue(reportDetails);

    Dictionary<string, ColumnDetail> columnDetails =
        GetColumnDetails(items);
    report.Append(GetHeaders(columnDetails));
    report.Append(GetRows(items, columnDetails));

    return report.ToString();
}
```

这个类提取相同的属性值，但使用`dynamic`：

```cs
public string Generate(dynamic reportDetails)
{
    string title = reportDetails.Title;

    var report = new StringBuilder(
        $"# {title}\n\n");

    List<object> items = reportDetails.Data;

    Dictionary<string, ColumnDetail> columnDetails =
        GetColumnDetails(items);
    report.Append(GetHeaders(columnDetails));
    report.Append(GetRows(items, columnDetails));

    return report.ToString();
}
```

## 讨论

这个解决方案的概念再次是为报告库的用户提供对他们想要使用的类型的最大控制权。但是，如果你确实有一些限制怎么办？例如，必须有一种设置报告标题的方法，并且你需要知道那个属性是什么。这个解决方案通过告诉用户提供一个具有`Title`和`Data`属性的对象来与用户妥协。`Title`包含报告标题，`Data`包含报告行。只要他们提供这些属性，他们可以使用任何他们想要的对象。如果输入对象上有其他我们不关心的属性，它不会影响报告库。

我们将使用的类是`Inventory`，包含一个`Title`字符串和一个`Data`集合。`Main`方法填充一个`Inventory`实例并将其传递给`Generate`。

我们有两个`Generate`的示例：一个使用反射，另一个使用动态。在获取类型后，第一个示例调用`GetProperty`和`GetValue`来获取每个属性的值。方法的其余部分与配方 5.2 中的操作方式相同。

正如你所见，反射可能会很冗长，进行许多方法调用和类型转换。这是使用`dynamic`的一个好案例。我们知道`Title`和`Data`是存在的，那为什么不直接访问它们呢？这就是第二个示例的做法。首先，注意`reportDetails`参数的类型是`dynamic`。然后观察代码如何调用`Title`和`Data`，并将它们放入强类型变量中。

###### 注意

`dynamic`类型仍然是`object`类型，但是由 DLR 在后台执行了一些额外的魔法。

在开发过程中，由于`dynamic`不知道它正在处理的类型，所以你不能获得 IntelliSense，但是你可以得到可读的代码。在你了解到传递给代码的类型的成员时，`dynamic`是反射的更好机制。

## 参见

配方 5.2，“使用反射访问类型成员”

# 5.6 使用 Office 应用程序进行互操作

## 问题

您需要使用尽可能简单的代码填充 Excel 电子表格中的对象数据。

## 解决方案

这是一个为 Excel 添加额外成员的枚举：

```cs
public enum ReportType
{
    Html,
    Markdown,
    ExcelTyped,
    ExcelDynamic
}
```

不使用`dynamic`的 Excel 报告生成器：

```cs
public class ExcelTypedGenerator<TData> : GeneratorBase<TData>
{
    ApplicationClass excelApp;
    Workbook wkBook;
    Worksheet wkSheet;

    public ExcelTypedGenerator()
    {
        excelApp = new ApplicationClass();
        excelApp.Visible = true;

        wkBook = excelApp.Workbooks.Add(Missing.Value);
        wkSheet = (Worksheet)wkBook.ActiveSheet;
    }

    protected override StringBuilder GetTitle()
    {
        wkSheet.Cells[1, 1] = "Report";

        return new StringBuilder("Added Title...\n");
    }

    protected override StringBuilder GetHeaders(
        Dictionary<string, ColumnDetail> details)
    {
        ColumnDetail[] values = details.Values.ToArray();

        for (int i = 0; i < values.Length; i++)
        {
            ColumnDetail detail = values[i];
            wkSheet.Cells[3, i+1] = detail.Attribute.Name;
        }

        return new StringBuilder("Added Header...\n");
    }

    protected override StringBuilder GetRows(
        List<TData> items,
        Dictionary<string, ColumnDetail> details)
    {
        const int DataStartRow = 4;

        int rows = items.Count;
        int cols = details.Count;

        var data = new string[rows, cols];

        for (int i = 0; i < rows; i++)
        {
            List<string> columns =
                GetColumns(details.Values, items[i]);

            for (int j = 0; j < cols; j++)
            {
                data[i, j] = columns[j];
            }
        }

        int FirstCol = 'A';
        int LastExcelCol = FirstCol + cols - 1;
        int LastExcelRow = DataStartRow + rows - 1;
        string EndRangeCol = ((char)LastExcelCol).ToString();
        string EndRangeRow = LastExcelRow.ToString();

        string EndRange = EndRangeCol + EndRangeRow;
        string BeginRange = "A" + DataStartRow.ToString();

        var dataRange = wkSheet.get_Range(BeginRange, EndRange);
        dataRange.Value2 = data;

        wkBook.SaveAs(
            "Report.xlsx", Missing.Value, Missing.Value,
            Missing.Value, Missing.Value, Missing.Value,
            XlSaveAsAccessMode.xlShared, Missing.Value, Missing.Value,
            Missing.Value, Missing.Value, Missing.Value);

        return new StringBuilder(
            "Added Data...\n" +
            "Excel file created at Report.xlsx");
    }
}
```

使用动态生成 Excel 报告：

```cs
public class ExcelDynamicGenerator<TData> : GeneratorBase<TData>
{
    ApplicationClass excelApp;
    dynamic wkBook;
    Worksheet wkSheet;

    public ExcelDynamicGenerator()
    {
        excelApp = new ApplicationClass();
        excelApp.Visible = true;

        wkBook = excelApp.Workbooks.Add();
        wkSheet = wkBook.ActiveSheet;
    }

    protected override StringBuilder GetTitle()
    {
        wkSheet.Cells[1, 1] = "Report";

        return new StringBuilder("Added Title...\n");
    }

    protected override StringBuilder GetHeaders(
        Dictionary<string, ColumnDetail> details)
    {
        ColumnDetail[] values = details.Values.ToArray();

        for (int i = 0; i < values.Length; i++)
        {
            ColumnDetail detail = values[i];
            wkSheet.Cells[3, i+1] = detail.Attribute.Name;
        }

        return new StringBuilder("Added Header...\n");
    }

    protected override StringBuilder GetRows(
        List<TData> items,
        Dictionary<string, ColumnDetail> details)
    {
        const int DataStartRow = 4;

        int rows = items.Count;
        int cols = details.Count;

        var data = new string[rows, cols];

        for (int i = 0; i < rows; i++)
        {
            List<string> columns =
                GetColumns(details.Values, items[i]);

            for (int j = 0; j < cols; j++)
            {
                data[i, j] = columns[j];
            }
        }

        int FirstCol = 'A';
        int LastExcelCol = FirstCol + cols - 1;
        int LastExcelRow = DataStartRow + rows - 1;
        string EndRangeCol = ((char)LastExcelCol).ToString();
        string EndRangeRow = LastExcelRow.ToString();

        string EndRange = EndRangeCol + EndRangeRow;
        string BeginRange = "A" + DataStartRow.ToString();

        var dataRange = wkSheet.get_Range(BeginRange, EndRange);
        dataRange.Value2 = data;

        wkBook.SaveAs(
            "Report.xlsx",
            XlSaveAsAccessMode.xlShared);

        return new StringBuilder(
            "Added Data...\n" +
            "Excel file created at Report.xlsx");
    }
}
```

## 讨论

这个示例基于配方 5.3 中的多报告格式生成代码，简要说明了如何添加另一种报告类型。这个解决方案展示了如何实现。

首先，注意`ReportType`枚举有两个额外的成员：`ExcelTyped`和`ExcelDynamic`。两者都使用一致的约定，其中`ExcelTyped`创建一个`ExcelTypedGenerator`实例，而`ExcelDynamic`创建一个`ExcelDynamicGenerator`实例。区别在于，`ExcelTypedGenerator`使用强类型代码生成 Excel 报告，而`ExcelDynamicGenerator`使用动态代码生成 Excel 报告。

###### 提示

您可以使用这样的技术来自动化任何 Microsoft Office 应用程序。关键是确保您已通过 Visual Studio Installer 安装了 Visual Studio Tools for Office（VSTO）。这将安装所谓的*主互操作程序集*（PIAs）。安装后，您可以在您的 Visual Studio 安装文件夹下找到这些 PIAs（例如，我的机器上的文件夹是*C:\Program Files (x86)\Microsoft Visual Studio\Shared\Visual Studio Tools for Office\PIA*），并使用与您安装的 Microsoft Office 版本对应的版本。如果您的 Office 版本较旧，无法安装 VSTO，请搜索[下载选项](https://oreil.ly/vbMvL)。

要查看这两个示例之间的区别，请逐个成员地查看。特别是 `ExcelTypedGenerator` 具有强类型字段，因此每当不使用参数并且需要对返回类型执行转换时，必须使用 `Missing.Value` 占位符。注意 `GetRows` 方法末尾的 `SaveAs` 方法调用，这特别繁琐。

相比之下，将这些示例与 `ExcelDynamicGenerator` 代码进行比较。将 `wkBook` 字段设置为 `dynamic`，而不是强类型，可以改变代码。不再需要 `Missing.Value` 占位符或类型转换。代码编写起来更容易，阅读起来也更容易。

## 参见

Recipe 5.3，“使用反射实例化类型成员”

# 5.7 创建一个固有动态类型

## 问题

您的数据以专有格式存在，但希望通过对象访问成员而无需自行解析。

## 解决方案

这个类保存要显示在报告中的数据：

```cs
public class LogEntry
{
 [Column("Log Date", Format = "{0:yyyy-MM-dd hh:mm}")]
    public DateTime CreatedAt { get; set; }

 [Column("Severity")]
    public string Type { get; set; }

 [Column("Location")]
    public string Where { get; set; }

 [Column("Message")]
    public string Description { get; set; }
}
```

这些方法获取日志数据并返回具有该数据的 `DynamicObject` 类型列表：

```cs
static List<dynamic> GetData()
{
    string headers = "Date|Severity|Location|Message";

    string logData = GetLogData();

    return
        (from line in logData.Split('\n')
         select new DynamicLog(headers, line))
        .ToList<dynamic>();
}

static string GetLogData()
{
    return
"2022-11-12 12:34:56.7890|INFO|Section_05_07.Program|Got this far\n" +
"2022-11-12 12:35:12.3456|ERROR|Section_05_07.Report|Index out of range\n" +
"2022-11-12 12:55:34.5678|WARNING|Section_05_07.Report|Please check this";
}
```

这个类是一个 `DynamicObject`，它知道如何读取日志文件并动态公开属性：

```cs
public class DynamicLog : DynamicObject
{
    Dictionary<string, string> members =
        new Dictionary<string, string>();

    public DynamicLog(string headerString, string logString)
    {
        string[] headers = headerString.Split('|');
        string[] logData = logString.Split('|');

        for (int i = 0; i < headers.Length; i++)
            members[headers[i]] = logData[i];
    }

    public override bool TryGetMember(
        GetMemberBinder binder, out object result)
    {
        result = members[binder.Name];
        return true;
    }

    public override bool TryInvokeMember(
        InvokeMemberBinder binder, object[] args, out object result)
    {
        return base.TryInvokeMember(binder, args, out result);
    }

    public override bool TrySetMember(
        SetMemberBinder binder, object value)
    {
        members[binder.Name] = (string)value;
        return true;
    }
}
```

`Main` 方法消耗动态数据，填充数据对象，并获取新报告：

```cs
static void Main()
{
    List<dynamic> logData = GetData();

    var tempDateTime = DateTime.MinValue;
    List<object> inventory =
        (from log in logData
         let canParse =
            DateTime.TryParse(
                log.Date, out tempDateTime)
         select new LogEntry
         {
             CreatedAt = tempDateTime,
             Type = log.Severity,
             Where = log.Location,
             Description = log.Message
         })
        .ToList<object>();

    string report = new Report().Generate(inventory);

    Console.WriteLine(report);
}
```

## 讨论

`DynamicObject` 类型是 .NET Framework 的一部分，支持用于与动态语言的互操作性的 DLR。这是一种特殊的类型，允许任何人调用类型成员，并且它可以拦截调用并根据您编程的方式行为。与其挥手列举使用 `DynamicObject` 的几种方法，这个解决方案专注于需要一个对象来处理专有数据的问题。在这个解决方案中，数据是日志文件格式。在这里，我们将使用 `DynamicObject` 提供数据，并使用来自 Recipe 5.2 的报告库来显示日志数据。

`LogEntry` 类表示报告中的一行。我们无法将 `DynamicObject` 实例传递给 `Report`，因为没有办法反射它并提取属性。任何解决方法都很麻烦，使用 `DynamicObject` 处理数据并生成 `LogEntry` 对象的集合，然后将它们传递给 `Report` 更为简单。

`GetLogData` 方法展示了日志文件的样子。`GetData` 定义了一个 `headers` 字符串，它是每个日志文件条目的元数据。LINQ 查询遍历日志的每一行，结果是一个 `List<dynamic>`。投影使用头部和日志条目实例化一个新的 `DynamicLog` 实例。

`DynamicLog` 类型继承自 `DynamicObject`，仅实现了它需要的方法。`DynamicLog` 的实现展示了其中一些成员：`TryGetMember`、`TryInvokeMember` 和 `TrySetMember`。解决方案没有使用 `TryInvokeMember`，但我把它保留在那里以展示 `DynamicObject` 不仅仅与属性一起工作，还有其他重载。`Dictionary<string, string>` `members` 中保存着日志中每个字段的值，键来自头部，值来自日志文件中相同位置的字符串。

构造函数填充成员。它在每个字段的 `(|)` 分隔符上分割，并通过头部迭代，直到 `members` 每列都有一个条目。`TryGetMembers` 方法通过 `out object result` 参数从字典读取返回值。记得在成功时返回 `true`，因为返回 `false` 表示无法执行操作，用户将收到运行时异常。`TrySetMember` 用值填充字典。

`GetMemberBinder` 和 `SetMemberBinder` 包含被访问属性的元数据。例如，以下代码将调用 `TryGetMember`：

```cs
string severity = log.Severity;
```

假设 `log` 是 `DynamicLog` 的一个实例，则 `GetMemberBinder` 的 `Name` 属性将是 `Severity`。它将索引字典并返回分配给该键的任何值。类似地，以下代码将调用 `TrySetMember`：

```cs
log.Severity = "ERROR";
```

在这种情况下，`binder.Name` 将是 `Severity`，它将更新字典中该键的值为 `ERROR`。

这意味着现在我们有一个对象，您可以在其中设置您选择的属性名称，并提供相同格式的任何日志文件（管道分隔符）。每次您想要适应管道分隔格式的日志文件时，无需自定义类。

`GetData` 返回一个 `List<dynamic>`。因为它是一个动态对象，并且我们已经知道属性名称应该是什么（它们与头部匹配），我们可以通过在动态对象上仅指定属性名称来投影为 `LogEntry` 实例。此外，您可以在配置文件或数据库中指定这些头部，这些头部可以是数据驱动的，并且每次都可以更改。也许您甚至想要能够更改文件的分隔符来处理更多文件类型。正如您所看到的，这在 `DynamicObject` 中非常容易实现。

## 参见

Recipe 5.2, “使用反射访问类型成员”

# 5.8 动态添加和删除类型成员

## 问题

您希望一个完全动态的对象，可以在运行时添加成员，就像在 JavaScript 中一样。

## 解决方案

此方法使用 `ExpandoObject` 收集数据：

```cs
static List<dynamic> GetData()
{
    const int Date = 0;
    const int Severity = 1;
    const int Location = 2;
    const int Message = 3;

    var logEntries = new List<dynamic>();

    string logData = GetLogData();

    foreach (var line in logData.Split('\n'))
    {
        string[] columns = line.Split('|');

        dynamic logEntry = new ExpandoObject();

        logEntry.Date = columns[Date];
        logEntry.Severity = columns[Severity];
        logEntry.Location = columns[Location];
        logEntry.Message = columns[Message];

        logEntries.Add(logEntry);
    }

    return logEntries;
}

static string GetLogData()
{
    return
        "2022-11-12 12:34:56.7890|INFO" +
        "|Section_05_07.Program|Got this far\n" +
        "2022-11-12 12:35:12.3456|ERROR" +
        "|Section_05_07.Report|Index out of range\n" +
        "2022-11-12 12:55:34.5678|WARNING" +
        "|Section_05_07.Report|Please check this";
}
```

`Main` 方法将 `List<dynamic>` 转换为 `List<LogEntry>` 并获取报告：

```cs
static void Main()
{
    List<dynamic> logData = GetData();

    var tempDateTime = DateTime.MinValue;
    List<object> inventory =
        (from log in logData
         let canParse =
            DateTime.TryParse(
                log.Date, out tempDateTime)
         select new LogEntry
         {
             CreatedAt = tempDateTime,
             Type = log.Severity,
             Where = log.Location,
             Description = log.Message
         })
        .ToList<object>();

    string report = new Report().Generate(inventory);

    Console.WriteLine(report);
}
```

## 讨论

这类似于 Recipe 5.7 中的 `DynamicObject` 示例，但它涵盖了一个更简单的情况，您不需要那么多的灵活性。如果您预先知道文件格式，并且知道它不会更改，但希望以一种简单的方式将数据提取到动态对象中，而不必每次都创建一个新类型发送到报告中，那该怎么办？

在这种情况下，您可以使用 `ExpandoObject`，它是 .NET Framework 的一种类型，允许您动态添加和移除类型成员，与 JavaScript 中的操作相同。

在解决方案中，`GetData` 方法实例化一个 `ExpandoObject`，将其分配给 `dynamic` 类型的 `logEntry`。然后，它会动态添加属性，并使用解析的日志文件数据填充这些属性。

`Main` 方法从 `GetData` 接受一个 `List<dynamic>`。只要每个对象具有它所期望的属性，一切都会很好。

## 参见

Recipe 5.7, “Creating an Inherently Dynamic Type”

# 5.9 从 C# 调用 Python 代码

## 问题

您有一个 C# 程序，并且想要使用一些 Python 代码，但不想重写它。

## 解决方案

此 Python 文件包含我们需要使用的代码：

```cs
import sys
sys.path.append(
    "/System/Library/Frameworks/Python.framework" +
    "/Versions/Current/lib/python2.7")

from random import *

class SemanticAnalysis:
    @staticmethod
    def Eval(text):
        val = random()
        return val < .5
```

此类表示社交媒体数据：

```cs
public class Tweet
{
 [Column("Screen Name")]
    public string ScreenName { get; set; }

 [Column("Date")]
    public DateTime CreatedAt { get; set; }

 [Column("Text")]
    public string Text { get; set; }

 [Column("Semantic Analysis")]
    public string Semantics { get; set; }
}
```

`Main` 方法获取数据并生成报告：

```cs
static void Main()
{
    List<object> tweets = GetTweets();

    string report = new Report().Generate(tweets);

    Console.WriteLine(report);
}
```

这些是 `IronPython` NuGet 包的必需命名空间的一部分：

```cs
using IronPython.Hosting;
using Microsoft.Scripting.Hosting;
```

此方法设置了 Python 互操作：

```cs
static List<object> GetTweets()
{
    ScriptRuntime py = Python.CreateRuntime();
    dynamic semantic = py.UseFile("../../../Semantic.py");
    dynamic semanticAnalysis = semantic.SemanticAnalysis();

    DateTime date = DateTime.UtcNow;

    var tweets = new List<object>
    {
        new Tweet
        {
            ScreenName = "SomePerson",
            CreatedAt = date.AddMinutes(5),
            Text = "Comment #1",
            Semantics = GetSemanticText(semanticAnalysis, "Comment #1")
        },
        new Tweet
        {
            ScreenName = "SomePerson",
            CreatedAt = date.AddMinutes(7),
            Text = "Comment #2",
            Semantics = GetSemanticText(semanticAnalysis, "Comment #2")
        },
        new Tweet
        {
            ScreenName = "SomePerson",
            CreatedAt = date.AddMinutes(12),
            Text = "Comment #3",
            Semantics = GetSemanticText(semanticAnalysis, "Comment #3")
        },
    };

    return tweets;
}
```

此方法通过 `dynamic` 实例调用 Python 代码：

```cs
static string GetSemanticText(dynamic semantic, string text)
{
    bool result = semantic.Eval(text);
    return result ? "Positive" : "Negative";
}
```

## 讨论

在这个示例中，您正在处理社交媒体数据。报告中的一个项目是语义分析，告诉用户的推文是积极的还是消极的。您有一个很棒的语义分析 AI 模型，但它是用 TensorFlow 在 Python 模块中构建的。能够重用该代码而不是重写它将非常有帮助。

这就是 DLR 的作用所在，因为它允许您从 C# 中调用 Python（以及其他动态语言）。考虑到建立一个机器学习模型（或任何其他类型的模块）可能需要很多个月的时间，跨语言重用代码的优势是巨大的。

Python 文件中的 `SemanticAnalysis` 类模拟一个模型，返回正面结果为 `true` 或负面结果为 `false`。

`Main` 方法调用 `GetTweets` 来获取数据，并使用 `Report` 类，与 Recipe 5.2 中相同。从 `GetTweets` 返回的 `List<object>` 包含可以与报告生成器一起工作的 `Tweet` 对象。

###### 提示

要设置这一点，您需要引用 `IronPython` 包，您可以在 NuGet 上找到它。您还可能发现通过 Visual Studio 安装程序安装 Python 工具对于 Visual Studio 会很有用。

`GetTweets`方法需要一个对 Python `SemanticAnalysis`类的引用。调用`CreateRuntime`创建一个 DLR 引用。然后，您需要通过`UseFile`指定 Python 文件的位置。之后，您可以实例化`SemanticAnalysis`类。每个`Tweet`实例通过调用`GetSemanticText`，传递`SemanticAnalysis`引用和`text`来设置`Semantics`属性。

`GetSemanticText`方法调用`Eval`并以`text`作为其参数，返回一个`bool`结果，然后将其翻译为报告友好的“Positive”或“Negative”字符串。

在几行代码中，您看到了重用用动态语言编写的重要代码有多容易。DLR 支持的语言包括 Ruby 和 JavaScript 等。

## 参见

Recipe 5.2，“使用反射访问类型成员”

# 5.10 从 Python 调用 C#代码

## 问题

您有一个 Python 程序，并且想要使用 C#代码，但不想重写它。

## 解决方案

这是需要使用报告生成器的主要 Python 应用程序：

```cs
import clr, sys

sys.path.append(
    r"C:\Path Where You Cloned The Project" +
    "\Chapter05\Section-05-10\bin\Debug")
clr.AddReference(
    r"C:\Path Where You Cloned The Project" +
    \Chapter05\Section-05-10\bin\Debug\PythonToCS.dll")

from PythonToCS import Report
from PythonToCS import InventoryItem
from System import Decimal

inventory = [
    InventoryItem("1", "Part #1", 3, Decimal(5.26)),
    InventoryItem("2", "Part #2", 1, Decimal(7.95)),
    InventoryItem("3", "Part #1", 2, Decimal(23.13))]

rpt = Report()

result = rpt.GenerateDynamic(inventory)

print(result)
```

这个类有一个构造函数，以便在 Python 中更容易使用：

```cs
public class InventoryItem
{
    public InventoryItem(
        string partNumber, string description,
        int count, decimal itemPrice)
    {
        PartNumber = partNumber;
        Description = description;
        Count = count;
        ItemPrice = itemPrice;
    }

 [Column("Part #")]
    public string PartNumber { get; set; }

 [Column("Name")]
    public string Description { get; set; }

 [Column("Amount")]
    public int Count { get; set; }

 [Column("Price", Format = "{0:c}")]
    public decimal ItemPrice { get; set; }
}
```

这是 Python 代码调用以生成报告的 C#方法：

```cs
public string GenerateDynamic(dynamic[] items)
{
    List<object> inventory =
        (from item in items
         select new InventoryItem
         (
             item.PartNumber,
             item.Description,
             item.Count,
             item.ItemPrice
         ))
        .ToList<object>();

    return Generate(inventory);
}
```

## 讨论

在 Recipe 5.9 中，场景是从 C#调用 Python。在这个问题中的场景相反，我有一个 Python 应用程序，并且需要能够生成报告。然而，报告生成器是用 C#编写的。报告库已经投入了大量工作，因此重写为 Python 没有意义。幸运的是，DLR 允许我们用 Python 调用那个 C#代码。

报告与 Recipe 5.2 中使用的相同，而且 C#代码具有相同的`InventoryItem`类。

###### 小贴士

要设置这个，您可能需要安装[`pythonnet` package](https://oreil.ly/hY9bZ)：

```cs
>pip install pythonnet
```

通过导入`clr`和`sys`来设置 Python 代码，并调用`sys.path.append`来引用 C# DLL 所在的路径，然后调用`clr.AddReference`来添加对要使用的 C# DLL 的引用。

在 Python 中，每当您需要从框架或自定义程序集中使用.NET 类型时，请使用`from Namespace import type`语法，它大致相当于 C#的`using`声明。在 C#源代码中，命名空间是`PythonToCS`，代码使用它来导入`Report`和`InventoryItem`的引用。它还使用`System`命名空间来获取对`Decimal`类型的引用，该类型别名为 C#的`decimal`类型。

在 Python 中，每当您使用方括号`[]`时，您正在创建一个名为`list`的数据结构。它是一个具有 Python 语义的对象集合。在这个例子中，我们正在创建一个`InventoryItem`列表，并将其赋值给名为`inventory`的变量。

注意，在`InventoryItem`构造函数的最后一个参数`itemPrice`中我们使用了`Decimal`。Python 没有`decimal`的概念，会将该值作为`float`传递，这会导致错误，因为 C# 中的 `InventoryItem` 将该参数定义为`decimal`。

接下来，Python 代码实例化了`Report`，`rpt`，并调用了`GenerateDynamic`，传递了`inventory`。这将在`Report`中调用`GenerateDynamic`，并自动将 Python 的`list`转换为 C# 的`dynamic[]`，`items`。因为`items`中的每个对象都是`dynamic`，我们可以使用 LINQ 语句查询它，动态访问投影中每个对象的名称。

最后，`GenerateDynamically`调用`Generate`，应用程序返回一个报告，Python 代码打印该报告。

## 另请参见

Recipe 5.2，“使用反射访问类型成员”

Recipe 5.9，“从 C# 调用 Python 代码”
