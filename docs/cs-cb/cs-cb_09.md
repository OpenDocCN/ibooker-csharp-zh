# 第九章：检视最近的 C#语言亮点

C#编程语言不断发展。前几章讨论了从 C# 1 到 C# 8 的主题。例外是在第八章中的模式匹配，其中一些模式是在 C# 9 中引入的。本章主要关注 C# 9，例外是 Recipe 9.9 的主题，这是一个 C# 8 的功能。

本章的一个核心概念是不可变性——能够创建和操作不会改变的类型。不可变性对于安全的多线程操作非常重要，也有助于减轻认知负担，因为你知道，你将一个类型传递给的代码不会改变（突变）对象的内容。

示例场景是处理地址，如邮寄地址或送货地址。在许多情况下，地址是一个值，意味着它没有身份。地址作为与客户或公司等实体关联的一组数据（值）存在。另一方面，实体确实有一个身份，通常在数据库中建模为 ID 字段。因为我们将其视为一个值，所以地址成为不可变性的一个有用候选，因为我们不希望该值在设置后发生变化。

C# 9 的一个有趣特性称为*模块初始化*。想想我们如何使用构造函数初始化类型或`Main`初始化应用程序。模块初始化允许你在程序集范围内编写初始化代码，你将看到它是如何工作的。

另一个 C# 9 的主题是简化代码。你将看到如何写出不带命名空间或类的代码，消除在`Main`方法中启动应用程序时的繁文缛节。另一个简化是在实例化对象时，使用新的特性来推断类型上下文。让我们从简化应用程序启动开始。

# 9.1 简化应用程序启动

## 问题

你需要尽可能地消除应用程序入口的代码。

## 解决方案

这是一个顶层程序：

```cs
using System;

Console.WriteLine("Address Info:\n");

Console.Write("Street: ");
string street = Console.ReadLine();

Console.Write("City: ");
string city = Console.ReadLine();

Console.Write("State: ");
string state = Console.ReadLine();

Console.Write("Zip: ");
string zip = Console.ReadLine();

Console.WriteLine($@"
 Your address is:

 {street}
 {city}, {state} {zip}");
```

这是 C#编译器生成的代码：

```cs
using System;
using System.Runtime.CompilerServices;

[CompilerGenerated]
internal static class <Program>$
{
  private static void <Main>$(string[] args)
  {
    Console.WriteLine("Address Info:\n");
    Console.Write("Street: ");
    string street = Console.ReadLine();
    Console.Write("City: ");
    string city = Console.ReadLine();
    Console.Write("State: ");
    string state = Console.ReadLine();
    Console.Write("Zip: ");
    string zip = Console.ReadLine();
    Console.WriteLine(
      "\r\n    Your address is:\r\n\r\n    " + street +
      "\r\n    " + city + ", " + state + " " + zip);
  }
}
```

## 讨论

我们写的大部分代码都是样板代码——我们一遍又一遍地复制的标准语法。在具有`Main`方法的控制台应用程序中，你会有一个命名空间，通常与项目名称匹配，一个名为`Program`的类和一个`Main`方法。虽然你可以自由删除命名空间并重命名类，但人们很少这样做。开发人员多年来一直意识到这一点，在 C# 9 中，我们不再需要这些样板代码。

解决方案展示了新的顶级语句功能，代码中不需要命名空间、类或`Main`方法。这是启动应用程序所需的最少代码。示例代码请求地址详细信息并打印结果，其功能与所写的代码完全相同。

###### 提示

如果你在教授某人如何在 C# 中编程，顶级语句可以使任务更加简单。你不需要解释方法，因为你不写`Main`方法。你不需要解释一个类，它是一个具有成员和更多的对象。你可以省略命名空间的讨论以及所有关于命名和组织的微妙之处。与其浅显地横跨（或忽略）所有这些复杂细节，你可以在学生准备好时稍后讨论它们。

顶级语句可以替代`Main`方法。解决方案显示了编译器生成的代码。它具有`CompilerGenerated`属性、类和`Main`方法。命名约定与 Visual Studio、.NET CLI 和其他 IDE 为控制台应用程序生成的典型样板代码匹配。

有趣的是，你只能将顶级语句放在单个文件中。如果尝试将它们放在多个文件中，你将遇到以下编译器错误：

```cs
CS8802  Only one compilation unit can have top-level statements.
```

# 9.2 减少实例化语法

## 问题

对象实例化过于冗长和啰嗦。

## 解决方案

我们要实例化这个类：

```cs
public class Address
{
    public Address() { }

    public Address(
        string street,
        string city,
        string state,
        string zip)
    {
        Street = street;
        City = city;
        State = state;
        Zip = zip;
    }

    public string Street { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string Zip { get; set; }
}
```

以下是实例化该类的不同方式：

```cs
class Program
{
    // doesn't work at this level
    // var address = new Address();

    // this still works
    Address addressOld = new Address();

    // new target typed field
    Address addressNew = new();

    static void Main()
    {
        // these still work
        var addressLocalVar = new Address();
        Address addressLocalOld = new Address();

        // new target typed local variable
        Address addressLocalNew = new();

        // target typed with object ini
        Address addressObjectInit = new()
        {
            Street = "123 4th St.",
            City =   "My City",
            State =  "ZZ",
            Zip =    "55555-3333"
        };

        // target typed with ctor init
        Address addressCtorInit = new(
            street: "567 8th Ave.",
            city:   "Some Place",
            state:  "YY",
            zip:    "12345-7890");
    }
}
```

## 讨论

最初，C# 有一种实例化变量的方法：声明类型、变量名、new 操作符、类型和带括号的构造函数参数列表。你可以在解决方案中看到这一点，通过`addressOld`字段和`addressLocalOld`变量。

在 C# 3 的范例下，我们需要一个强类型变量（`var`关键字）来保存匿名类型，特别是用于 LINQ 查询。`var`变量需要赋值，并成为一个强类型的分配类型。有些人看到`var`看起来像 JavaScript 的`var`，并不习惯使用它。然而，正如前面所述，C# 的`var`变量是强类型的，这意味着你不能将变量声明为不同的类型。

除了 LINQ，`var`的一个方便用法是在类型实例化中出现。开发人员认识到通过使用`var`消除定义变量时的冗余是可行的。你可以在解决方案中看到这是如何工作的，例如`addressLocalVar`变量。

由于使用`var`在对象实例化中减少代码的流行，开发人员开始寻求在字段中使用相同的体验。然而，你不能在字段中使用`var`，这在解决方案中的`address`字段中有所示。

C# 9 通过称为*目标类型的新*特性解决了冗余问题。而不是使用`var`，目标类型的新声明了类型、标识符和带参数的`new`关键字。`addressNew`字段和`addressLocalNew`变量展示了这是如何工作的。现在，你可以实例化字段，而无需在同一语句中指定相同类型两次的冗余。

目标类型的新实例化是永远做的相同类型实例化的快捷语法。这意味着你仍然可以使用对象初始化程序和构造函数重载，分别显示在`addressObjectInit`和`addressCtorInit`中。

现在我们有了目标类型的新特性，有理由更喜欢使用它而不是`var`。第一个原因是开发者在避免使用`var`时存在认知犹豫，因为它与 JavaScript 的`var`同名 —— 尽管我们知道 C#的`var`是强类型的。另一个原因是，由于我们可以在变量和字段中都使用目标类型的新特性，我们在实例化类型时有了语法上的一致性。一些开发者认为在同一代码中混合使用`var`和目标类型的新特性会让人分心或感觉凌乱。

###### 注意

即使你不在类型实例化时使用`var`，它仍然很有用。在进行 LINQ 查询时，你可以在同一方法中用匿名类型投影重新塑造数据，这就需要用到`var`来存储结果。

最后，将目标类型的新特性引入语言中并不一定意味着偏好直接对象实例化。正如配方 1.2 中解释的那样，IoC 是一种强大的机制，用于解耦代码，促进关注点分离，并使代码更易于测试。

## 另请参阅

配方 1.2, “移除显式依赖”

# 9.3 初始化不可变状态

## 问题

你需要在实例化期间仅填充不可变的属性。

## 解决方案

这里有一个具有不可变状态的类：

```cs
public class Address
{
    public Address() { }

    public Address(
        string street,
        string city,
        string state,
        string zip)
    {
        Street = street;
        City = city;
        State = state;
        Zip = zip;
    }

    public string Street { get; init; }
    public string City { get; init; }
    public string State { get; init; }
    public string Zip { get; init; }
}
```

以下是实例化不可变类的几种方式：

```cs
static void Main(string[] args)
{
    Address addressObjectInit = new()
    {
        Street = "123 4th St.",
        City = "My City",
        State = "ZZ",
        Zip = "55555-3333"
    };

    // not allowed
    //addressObjectInit.City = "A Locality";

    // target typed with ctor init
    Address addressCtorInit = new(
        street: "567 8th Ave.",
        city: "Some Place",
        state: "YY",
        zip: "12345-7890");

    // not allowed
    //addressCtorInit.Zip = "98765";
}
```

## 讨论

不可变性，即创建和操作不会改变的类型的能力，对代码的质量和正确性越来越重要。想象一下这样的情况：你将一个对象传递给一个方法，并得到相同类型的对象作为返回值。假设你不拥有该方法的代码，那么你如何知道该方法对你给定的对象做了什么？除了反编译或信任文档，你不知道。然而，如果对象是不可变的，它就不会改变，你就知道该方法没有做任何改变。

另一个不可变性的用例是多线程。死锁和竞争条件的现实长期困扰着开发者。在死锁场景中，不同线程等待对方释放另一个线程需要的资源以进行更改。在竞争条件场景中，你无法知道哪个线程会首先修改对象，导致对象状态不一致。在每种情况下，不可变性简化了情景，因为任何线程都无法更改现有对象 —— 它们必须依赖自己的副本。多线程是如此复杂的话题，无法在此深入讨论，但关键是不可变性是解决方案的一部分。

在解决方案中，`Address`类是不可变的。你可以用需要的数据实例化它，但其内容在那之后不能改变。请注意，属性有 getter 但没有 setter。相反，它们有 initters。initters 允许你实例化对象，但随后不能更改它。

`Main` 方法展示了这是如何工作的。 `addressObjectInit` 变量可以正常实例化，但是设置其任何属性，包括 `City`，都将无法通过编译。 `addressCtorInit` 变量展示了类似的情况。

如果您有现有类，使属性成为只读可以很有用。但是，如果您正在使用 C# 9 构建新类型，您也可以定义记录类型，正如下一篇配方所讨论的那样。

## 参见

配方 9.4，“创建不可变类型”

# 9.4 创建不可变类型

## 问题

您需要一个不可变的引用类型，但不想编写所有的管道代码。

## 解决方案

这是一个 C# 记录：

```cs
record Address(
    string Street,
    string City,
    string State,
    string Zip);
```

此代码显示了如何使用该记录：

```cs
static void Main(string[] args)
{
    var addressClassic = new Address(
        Street: "567 8th Ave.",
        City: "Some Place",
        State: "YY",
        Zip: "12345-7890");

    // or

    Address addressCtorInit = new(
        Street: "567 8th Ave.",
        City: "Some Place",
        State: "YY",
        Zip: "12345-7890");

    // not allowed
    //addressCtorInit.Street = "333 2nd St.";

    Console.WriteLine(
        $"Value Equal:     " +
        $"{addressClassic == addressCtorInit}");
    Console.WriteLine(
        $"Reference Equal: " +
        $"{ReferenceEquals(addressClassic, addressCtorInit)}");

    Console.WriteLine(
        $"{nameof(addressClassic)}: {addressClassic}");
    Console.WriteLine(
        $"{nameof(Address)}:        {addressCtorInit}");
}
```

这是输出结果：

```cs
Value Equal:     True
Reference Equal: False
addressClassic: Address
{
    Street = 567 8th Ave., City = Some Place,
    State = YY, Zip = 12345-7890
}
Address:        Address
{
    Street = 567 8th Ave., City = Some Place,
    State = YY, Zip = 12345-7890
}
```

这是 C# 编译器生成的合成代码：

```cs
using System;
using System.Collections.Generic;
using System.Runtime.CompilerServices;
using System.Text;
using Section_09_04;

internal class Address : IEquatable<Address>
{
    protected virtual Type EqualityContract
    {
 [CompilerGenerated]
        get
        {
            return typeof(Address);
        }
    }

    public string Street { get; set; }

    public string City { get; set; }

    public string State { get; set; }

    public string Zip { get; set; }

    public Address(string Street, string City, string State, string Zip)
    {
        this.Street = Street;
        this.City = City;
        this.State = State;
        this.Zip = Zip;
        base..ctor();
    }

    public override string ToString()
    {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.Append("Address");
        stringBuilder.Append(" { ");
        if (PrintMembers(stringBuilder))
        {
            stringBuilder.Append(" ");
        }
        stringBuilder.Append("}");
        return stringBuilder.ToString();
    }

    protected virtual bool PrintMembers(StringBuilder builder)
    {
        builder.Append("Street");
        builder.Append(" = ");
        builder.Append((object?)Street);
        builder.Append(", ");
        builder.Append("City");
        builder.Append(" = ");
        builder.Append((object?)City);
        builder.Append(", ");
        builder.Append("State");
        builder.Append(" = ");
        builder.Append((object?)State);
        builder.Append(", ");
        builder.Append("Zip");
        builder.Append(" = ");
        builder.Append((object?)Zip);
        return true;
    }

    public static bool operator !=(Address? r1, Address? r2)
    {
        return !(r1 == r2);
    }

    public static bool operator ==(Address? r1, Address? r2)
    {
        return (object)r1 == r2 || (r1?.Equals(r2) ?? false);
    }

    public override int GetHashCode()
    {
        return
        (((EqualityComparer<Type>.Default.GetHashCode(EqualityContract)
        * -1521134295
        + EqualityComparer<string>.Default.GetHashCode(Street))
        * -1521134295
        + EqualityComparer<string>.Default.GetHashCode(City))
        * -1521134295
        + EqualityComparer<string>.Default.GetHashCode(State))
        * -1521134295
        + EqualityComparer<string>.Default.GetHashCode(Zip);
    }

    public override bool Equals(object? obj)
    {
        return Equals(obj as Address);
    }

    public virtual bool Equals(Address? other)
    {
        return (object)other != null
        && EqualityContract == other!.EqualityContract
        && EqualityComparer<string>.Default.Equals(Street, other!.Street)
        && EqualityComparer<string>.Default.Equals(City, other!.City)
        && EqualityComparer<string>.Default.Equals(State, other!.State)
        && EqualityComparer<string>.Default.Equals(Zip, other!.Zip);
    }

    public virtual Address <Clone>$()
    {
        return new Address(this);
    }

    protected Address(Address original)
    {
        Street = original.Street;
        City = original.City;
        State = original.State;
        Zip = original.Zip;
    }

    public void Deconstruct(
        out string Street, out string City,
        out string State, out string Zip)
    {
        Street = this.Street;
        City = this.City;
        State = this.State;
        Zip = this.Zip;
    }
}
```

## 讨论

配方 9.3 讨论了不可变性及其好处，以及如何创建不可变类。如果您有现有类型并希望将它们迁移到不可变状态，则这种方法非常有效。然而，对于想要使新代码和类型变为不可变的情况，建议考虑使用记录。

记录在 C# 9 中被引入为创建简单不可变类型的一种方式。解决方案展示了如何在 `Address` 记录中实现这一点。声明类型为记录，为其命名，列出属性，并以分号终止。尽管这看起来可能类似于定义构造函数或方法，但参数定义了这种新类型的属性，并遵循了常见的帕斯卡命名约定。

解决方案展示了如何实例化 `Address` 记录。请注意，`addressCtorInit` 变量不允许更改其状态，包括 `Street` 属性。

有关记录的一个有趣事实是它们是具有值语义的引用类型。解决方案显示，使用 `==` 比较 `addressClassic` 和 `addressCtorInit` 结果为 `true`。这表明了值相等性，因为两个记录的属性是相同的。但请注意 `ReferenceEquals` 的比较结果。它是 `false`，因为记录是引用类型，每个都指向内存中的不同对象。

虽然声明 `Address` 记录是简短快捷的，但这是 C# 编译器生成的实际代码的大幅简化。解决方案展示了带有许多成员的合成代码。该类型是一个类，并且具有一个构造函数重载，用于填充每个属性的参数。

值相等性的关键在于 `IEquatable<Address>` 的实现。该类具有弱类型和强类型 `Equals` 方法。配方 2.5 展示了如何实现 `IEquatable<T>`，这与此实现有些相似之处。一个区别是存储在 `EqualityContract` 属性中的类型。由于 C# 生成的类在 `Equals` 和 `GetHashCode` 中使用 `EqualityContract`，因此消除冗余是有道理的。

如果你读过 Recipe 3.6，你可能会觉得`ToString`和`PrintMembers`的实现很熟悉。这两个实现几乎完全相同。请注意，`PrintMembers`是`virtual`的，这允许派生类型将它们的值添加到输出中。

最后，生成的类包括一个`Clone`方法用于获取浅复制，一个用于将值表示为元组的解构方法，以及一个复制构造函数用于复制另一个记录。还有一个方便的方法可以获得当前对象的副本，但带有修改，下面将讨论这一点。

## 另请参阅

Recipe 2.5, “检查类型相等性”

Recipe 3.6, “自定义类字符串表示”

Recipe 9.3, “初始化不可变状态”

# 9.5 简化不可变类型赋值

## 问题

您需要更改对象的一个属性，但不想改变原始对象。

## 解决方案

我们将使用这个记录：

```cs
record Address(
    string Street,
    string City,
    string State,
    string Zip);
```

以下代码展示了如何复制该记录：

```cs
static void Main(string[] args)
{
    Address addressPre = new(
        Street: "567 8th Ave.",
        City: "Some Place",
        State: "YY",
        Zip: "12345-7890");

    Address addressPost =
        addressPre with
        {
            Street = "569 8th Ave."
        };

    Console.WriteLine($"Pre:  {addressPre}");
    Console.WriteLine($"Post: {addressPost}");

    Console.WriteLine(
        $"Value Equal: " +
        $"{addressPre == addressPost}");
}
```

## 讨论

如同在 Recipe 9.4 中讨论的那样，记录类型具有普通构造函数、复制构造函数和克隆方法，用于创建相同类型的新记录。这些选项很容易涵盖一个场景：获取一个具有修改的类型。也就是说，如果你希望对象上的所有内容都保持不变，只改变一个或两个属性，本节展示了一种简单的方法来实现这一点。

由于记录是不可变的，您不能修改它们的任何属性。您可以始终实例化一个新的记录并提供所有属性，但是如果只有一个属性发生变化，尤其是如果它是一个具有许多属性的对象，这种做法是很浪费的。

C# 9 中的解决方案使用了`with`表达式。您有一个现有记录，添加一个`with`表达式，仅更改需要更改的属性。这样可以获得一个带有所需更改的新记录类型的实例。

解决方案在`addressPre`变量上执行此操作。`with`表达式使用一个属性赋值块来指定需要更改的属性。此示例更改了一个属性。您也可以像使用对象初始化程序一样设置多个属性，通过逗号分隔的列表。

## 另请参阅

Recipe 9.4, “创建不可变类型”

# 9.6 为记录重用设计

## 问题

你需要避免重复功能。

## 解决方案

这是一个抽象基本记录：

```cs
public abstract record AddressBase(
    string Street,
    string City,
    string State,
    string Zip);
```

这两个记录派生自该抽象基本记录：

```cs
public record MailingAddress(
    string Street,
    string City,
    string State,
    string Zip,
    string Email,
    bool PreferEmail)
    : AddressBase(Street, City, State, Zip);

public record ShippingAddress : AddressBase
{
    public ShippingAddress(
        string street,
        string city,
        string state,
        string zip,
        string deliveryInstructions)
        : base(street, city, state, zip)
    {
        if (street.Contains("P.O. Box"))
            throw new ArgumentException(
                "P.O. Boxes aren't allowed");

        DeliveryInstructions = deliveryInstructions;
    }

    public string DeliveryInstructions { get; init; }
}
```

这是如何使用这些记录的方法：

```cs
static void Main(string[] args)
{
    MailingAddress mailAddress = new(
        Street: "567 8th Ave.",
        City: "Some Place",
        State: "YY",
        Zip: "12345-7890",
        Email: "me@example.com",
        PreferEmail: true);

    ShippingAddress shipAddress = new(
        street: "567 8th Ave.",
        city: "Some Place",
        state: "YY",
        zip: "12345-7890",
        deliveryInstructions: "Ring Doorbell");

    Console.WriteLine($"Mail: {mailAddress}");
    Console.WriteLine($"Ship: {shipAddress}");

    Console.WriteLine(
        $"Derived types equal: " +
        $"{mailAddress == shipAddress}");

    AddressBase mailBase = mailAddress;
    AddressBase shipBase = shipAddress;
    Console.WriteLine(
        $"Base types equal: " +
        $"{mailBase == shipBase}");
}
```

## 讨论

在 C#中实现重用的一种方式是通过继承。记录支持与类相同的继承方式。

解决方案有一个名为 `AddressBase` 的记录。顾名思义，`AddressBase` 旨在作为基础记录。`AddressBase` 还是抽象的，阻止直接实例化。它具有所有派生类型通用的属性。

`MailingAddress` 和 `ShippingAddress` 派生自 `AddressBase`，使用类似类的继承语法。不同之处在于继承的记录声明包括参数列表，指示从派生记录中匹配基础记录的哪些参数。

`MailingAddress` 是基于 `AddressBase` 特化的，具有两个新属性：`Email` 和 `PreferEmail`。`ShippingAddress` 也是基于 `AddressBase` 特化的，额外增加了一个 `DeliveryInstructions` 属性。

`ShippingAddress` 的定义不同，因为它显式定义了成员，而不是使用默认的记录语法。它有一个构造函数，类似于 C# 类，将参数传递给基类 `AddressBase`。`ShippingAddress` 构造函数中包含验证代码，用于防止无效初始化而抛出异常。在这种情况下，它强制执行逻辑，即邮政信箱不是交付商品的地方。构造函数还初始化了 `DeliveryInstructions` 属性。这表明，虽然默认记录语法简化了代码，但你仍然可以根据需要定制记录。

在定制记录时，可以添加任何类可能具有的成员。此外，还可以重写等式、`ToString` 输出或构造函数的默认实现。此外，像 `ShippingAddress` 一样的定制并不会阻止 C# 编译器生成默认记录实现。

# 9.7 返回不同的方法覆盖类型

## 问题

要覆盖基类方法，但需要返回更具体的类型。

## 解决方案

这是我们想要处理的记录：

```cs
public abstract record AddressBase(
    string Street,
    string City,
    string State,
    string Zip);

public record MailingAddress(
    string Street,
    string City,
    string State,
    string Zip,
    string Email,
    bool PreferEmail)
    : AddressBase(Street, City, State, Zip);

public record ShippingAddress : AddressBase
{
    public ShippingAddress(
        string street,
        string city,
        string state,
        string zip,
        string deliveryInstructions)
        : base(street, city, state, zip)
    {
        if (street.Contains("P.O. Box"))
            throw new ArgumentException(
                "P.O. Boxes aren't allowed");

        DeliveryInstructions = deliveryInstructions;
    }

    public string DeliveryInstructions { get; init; }
}
```

这个基类有一个方法，返回一个基础记录：

```cs
abstract class DeliveryBase
{
    public abstract AddressBase GetAddress(string name);
}
```

这些类具有返回派生记录的方法：

```cs
class Communications : DeliveryBase
{
    public override MailingAddress GetAddress(string name)
    {
        return new(
            Street: "567 8th Ave.",
            City: "Some Place",
            State: "YY",
            Zip: "12345-7890",
            Email: "me@example.com",
            PreferEmail: true);
    }
}

class Shipping : DeliveryBase
{
    public override ShippingAddress GetAddress(string name)
    {
        return new(
            street: "567 8th Ave.",
            city: "Some Place",
            state: "YY",
            zip: "12345-7890",
            deliveryInstructions: "Ring Doorbell");
    }
}
```

该代码展示了如何使用那些返回派生记录的派生类。

```cs
static void Main(string[] args)
{
    Communications comm = new();
    MailingAddress mailAddr = comm.GetAddress("Person A");
    Console.WriteLine(mailAddr);

    Shipping ship = new();
    ShippingAddress shipAddr = ship.GetAddress("Person B");
    Console.WriteLine(shipAddr);
}
```

## 讨论

以前方法覆盖要求返回与基类虚方法返回类型相同。问题在于，派生类经常需要从其覆盖中返回特定信息。替代方案很丑陋：

1.  创建一个新的非多态方法。

1.  返回基础类型。

1.  返回一个从基础返回类型派生的类型，并期望调用者进行转换。

这些选择都不是最佳的，幸运的是，C# 9 通过协变返回类型提供了解决方案。

解决方案有两组类型层次结构：一组用于返回类型，另一组用于方法多态性。`AddressBase` 及其两个派生记录 `MailingAddress` 和 `ShippingAddress` 代表返回类型。`DeliveryBase` 类及其派生类 `Communications` 和 `Shipping` 具有多态操作的 `GetAddress` 方法。

###### 注意

注意`GetAddress`的实现如何返回目标类型的新实例。编译器通过上下文推断类型，这些示例中的返回类型是它。您可以在食谱 9.2 中了解更多关于目标类型新实例的信息。

在 C# 9 之前，在 `Communications` 和 `Shipping` 中的 `GetAddress` 将被迫返回 `AddressBase`。然而，通过查看解决方案实现，在 `Communications` 和 `Shipping` 中的 `GetAddress` 分别返回 `MailingAddress` 和 `ShippingAddress`。

## 参见

食谱 9.2，“减少实例化语法”

# 9.8 实现迭代器作为扩展方法

## 问题

您需要在一个您无法访问代码的第三方类型上添加一个迭代器。

## 解决方案

这里有一个我们无法访问第三方库的类型的定义：

```cs
public record Address(
    string Street,
    string City,
    string State,
    string Zip);
```

这个类有一个针对该类型的枚举器扩展方法：

```cs
public static class AddressExtensions
{
    public static IEnumerator<string> GetEnumerator(
        this Address address)
    {
        yield return address.Street;
        yield return address.City;
        yield return address.State;
        yield return address.Zip;
        yield break;
    }
}
```

这是如何使用该枚举器的方法：

```cs
class Program
{
    static void Main()
    {
        IEnumerable<Address> addresses = GetAddresses();

        foreach (var address in addresses)
        {
            foreach (var line in address)
                Console.WriteLine(line);

            Console.WriteLine();
        }
    }

    static IEnumerable<Address> GetAddresses()
    {
        return new List<Address>
        {
            new Address(
                Street: "567 8th Ave.",
                City: "Some Place",
                State: "YY",
                Zip: "12345-7890"),
            new Address(
                Street: "569 8th Ave.",
                City: "Some Place",
                State: "YY",
                Zip: "12345-7890")
        };
    }
}
```

## 讨论

有时，将迭代器添加到对象中会很方便。这样做可以将解剖、转换和返回对象数据的关注点与希望集中在解决业务问题的消费代码分离开来。如果您拥有对象的代码并希望循环访问其内容，则添加迭代器。然而，如果对象来自第三方且您无法访问其代码，则通常被迫在业务代码中添加冗余逻辑。在 C# 9 中，您现在可以作为扩展方法添加 `GetEnumerator` 方法。

在解决方案中，`Address` 记录是我们想要迭代的对象。更具体地说，我们想要遍历 `Address` 记录的成员，类似于您可以遍历 JavaScript 对象的属性的方式。

`AddressExtensions` 方法有一个名为 `GetEnumerator` 的扩展方法，接受一个 `Address` 参数并返回一个 `IEnumerable<T>`。`this` 参数的使用方式与任何其他扩展方法相同，指定要操作的类型和实例。迭代器的模式是该方法必须命名为 `GetEnumerator`，并且必须返回一个 `IEnumerator<T>`。类型 `T` 可以是您选择的任何类型——您所需要的类型。在这个例子中，`T` 是 `string`。这意味着您需要将每个属性转换为一个字符串，在 `Address` 中这并不是问题，因为所有属性都已经是字符串。与 C# 迭代器实现一致，`AddressExtensions` 的 `GetEnumerator` 方法使用 `yield return` 返回每个值，并使用 `yield break` 表示迭代结束。

在获取 `Address` 列表之后，`Main` 方法有一个嵌套的 `foreach` 循环，其中内部的 `foreach` 在 `Address` 的一个实例上进行迭代。由于扩展方法的存在，`foreach` 在处理 `address` 时与数组和集合的操作方式相同——不需要额外的语法。

# 9.9 切片数组

## 问题

您想使用范围来浏览数据。

## 解决方案

我们将使用这个记录：

```cs
public record Address(
    string Street,
    string City,
    string State,
    string Zip);
```

这个方法填充了一个记录数组：

```cs
Address[] GetAddresses()
{
    int count = 15;
    List<Address> addresses = new();

    for (int i = 0; i < count; i++)
    {
        string streetSuffix =
            i switch
            {
                0 => "st",
                1 => "nd",
                2 => "rd",
                _ => "th"
            };

        addresses.Add(
            new(
            Street: $"{i+100} {i+1}{streetSuffix} St.",
            City: "My Place",
            State: "ZZ",
            Zip: "12345-7890"));
    }

    return addresses.ToArray();
}
```

这个方法通过切片数组记录进行分页：

```cs
public IEnumerable<Address[]> GetAddresses(int perPage)
{
    Address[] addresses = GetAddresses();

    for (int i = 0, j = i+perPage;
         i < addresses.Length;
         i+=perPage, j+=perPage)
    {
        yield return addresses[i..j];
    }
}
```

这段代码迭代记录的各个页面：

```cs
static void Main()
{
    AddressService addressSvc = new();

    foreach (var addresses in
        addressSvc.GetAddresses(perRow: 3))
    {
        foreach (var address in addresses)
        {
            Console.WriteLine(address);
        }

        Console.WriteLine("\nNew Page\n");
    }
}
```

## 讨论

自从 C# 8 以来，对数组进行切片变得更加容易。指定开始索引，连接两个点，并指定最后索引。

这个解决方案从分页的角度看待切片问题。我们使用的一些应用程序按行数或行中的列数分页。这个解决方案每页显示三个`Address`实例。

`GetAddresses`方法有两个重载版本。第一个无参数版本生成唯一的地址。

第二个`GetAddresses`重载是一个接受`int`参数`perPage`的迭代器，指示方法一次返回多少个`Address`实例。获取`Address`实例列表后，`for`循环控制对列表的迭代。`for`初始化器将`i`设置为第一个`Address`，`j`设置为最后一个`Address`加一。由于`i`是范围的开始，`for`条件确保`i`不超过数组的大小。`for`增量器调整`i`和`j`到下一组`Address`实例（即下一页）。

`GetAddresses(int` `perPage)`是一个迭代器，表明它返回类型为`IEnumerable<Address[]>`并使用`yield return`来返回结果。而 9.8 节展示了如何将迭代器作为扩展方法添加，本例假设你可以访问代码并直接将迭代器添加到代码中更为合适。

`Main`方法展示了如何使用`GetAddresses(int perPage)`迭代器，返回从原始`Address[]`中切片的页面。

## 参见

9.8 节，“将迭代器实现为扩展方法”

# 9.10 初始化整个模块

## 问题

你需要 IoC 来在类库上工作，而不依赖于调用者正确实现。

## 解决方案

这是一个返回记录的存储库：

```cs
public record Address(
    string Street,
    string City,
    string State,
    string Zip);

public interface IAddressRepository
{
    List<Address> GetAddresses();
}

public class AddressRepository : IAddressRepository
{
    public List<Address> GetAddresses() =>
        new List<Address>
        {
            new (
                Street: "123 4th St.",
                City: "My Place",
                State: "ZZ",
                Zip: "12345-7890"),
            new (
                Street: "567 8th Ave.",
                City: "Some Place",
                State: "YY",
                Zip: "12345-7890"),
            new (
                Street: "567 8th Ave.",
                City: "Some Place",
                State: "YY",
                Zip: "12345-7890")
        };
}
```

这个模块初始化器配置了一个 IoC 容器：

```cs
class Initializer
{
    internal static ServiceProvider Container { get; set; }

 [ModuleInitializer]
    internal static void InitAddressUtilities()
    {
        var services = new ServiceCollection();
        services.AddTransient<AddressService>();
        services.AddTransient<IAddressRepository, AddressRepository>();
        Container = services.BuildServiceProvider();
    }
}
```

这个服务依赖于 IoC 容器：

```cs
public class AddressService
{
    readonly IAddressRepository addressRep;

    public AddressService(IAddressRepository addressRep) =>
        this.addressRep = addressRep;

    public static AddressService Create() =>
        Initializer.Container.GetRequiredService<AddressService>();

    public List<Address> GetAddresses() =>
        (from address in addressRep.GetAddresses()
         select address)
        .Distinct()
        .ToList();
}
```

这个`Main`方法是一个使用该服务的客户端：

```cs
static void Main()
{
    AddressService addressSvc = AddressService.Create();

    addressSvc
        .GetAddresses()
        .ForEach(address =>
            Console.WriteLine(address));
}
```

## 讨论

C# 9 添加了一个称为模块初始化的功能。基本上，这允许您为一个程序集添加任何类型的初始化代码。此初始化代码在程序集中的任何其他代码之前运行。

乍一看，这听起来可能有些奇怪，因为控制台、Windows 窗体和 WPF 应用程序都有`Main`方法。即使所有版本的 ASP.NET 和 Web API 也有启动代码。我并不是说这些技术没有用武之地，但对于普通的专业开发人员来说，这似乎是一个罕见的事件。

###### 注意

自 C# 1 以来存在的另一种初始化技术是使用静态构造函数。静态构造函数只有在代码通过类型或实例访问类成员时才运行。因此，静态构造函数不能有效替代模块初始化，因为可能永远不会访问该类的成员，静态构造函数也永远不会运行。

话虽如此，有一组涉及类库的用例。问题始终是您不知道消费代码将如何使用您的库。您可以记录并设置一个合同，该合同表示用户必须以某种方式调用某些方法或启动库，这是您能够保证对初始化进行任何类型控制的最接近方法。

模块初始化的变化使得您可以更好地控制如何初始化库代码，而不管用户做了什么。该解决方案解决了确保 IoC 在任何代码运行之前得到初始化的问题。

`AddressService` 类提供了两种实例化自身的方式，可以通过 IoC 或使用 `Create` 方法。用户可以选择是否使用 IoC。好处在于 IoC 对于库开发者来说也成为一个选项，便于编写单元测试和编写可维护的代码。

Recipe 1.2 解释了 IoC 的工作原理，使用 `Microsoft.Extensions.DependencyInjection`，而此解决方案使用相同的库和技术。主要区别在于 IoC 容器的配置位置。

`Initializer` 类有一个名为 `InitAddressUtilities` 的方法。`ModuleInitializer` 属性表明 `InitAddressUtilities` 是此类库的模块初始化代码。`InitAddressUtilities` 方法将在类库中的任何其他代码之前运行。

###### 注意

在 .NET 早期，模块是将代码组合到单个文件中的一种方式，用于模块化。您可以将模块组合成一个装配体，其中装配体定义为一个或多个模块及一个额外的清单。清单包含 CLR 的元数据，其详细信息繁多且对当前重点不重要。

使用模块在很大程度上是一种理论能力，因为大多数代码都编译为一个装配体的单一模块。这是 C# 编译器和 Visual Studio 的默认行为。事实上，你必须费劲去创建一个潜在有用的模块。虽然 `ModuleInitializer` 中有“module”一词，但实际情况是它适用于装配体级别的初始化。

因为 `InitAddressUtilities` 方法已经运行，所以 `AddressService` 中的 `Create` 方法可以依赖于 `Initializer.Container` 具有有效的容器引用来解析 `AddressService` 实例。

## 参见

Recipe 1.2，“Removing Explicit Dependencies”
