# 第四章\. 泛型

在 第三章 中，我展示了如何编写类型，并描述了它们可以包含的各种成员。然而，关于类、结构体、接口和方法还有一个额外的维度我没有展示。它们可以定义*类型参数*，这些占位符让你在编译时可以插入不同的类型。这使得你只需编写一个类型，然后就可以生成多个版本。执行此操作的类型称为*泛型类型*。例如，运行时库定义了一个名为 `List<T>` 的泛型类，它充当可变长度数组。这里的 `T` 是一个类型参数，你可以几乎使用任何类型作为参数，因此 `List<int>` 是一个整数列表，`List<string>` 是一个字符串列表，依此类推。¹ 你也可以编写*泛型方法*，它是一种具有自己类型参数的方法，与其包含的类型是否是泛型无关。

泛型类型和方法在视觉上是有区别的，因为它们的名称后面总是有尖括号 (`<` 和 `>`)。这些尖括号包含一个逗号分隔的参数或参数列表。与方法一样，这里也有参数/参数列表的区分：声明时指定参数列表，然后在使用方法或类型时为这些参数提供参数。因此，`List<T>` 定义了一个单一的类型参数 `T`，而 `List<int>` 提供了一个*类型参数* `int`。

对于类型参数，你可以使用任何喜欢的名称，只需符合 C#中标识符的通常约束即可，但有一些流行的约定。当只有一个参数时，通常（但不是普遍）使用 `T`。对于多参数泛型，你倾向于看到稍微更具描述性的名称。例如，运行时库定义了 `Dictionary<TKey, TValue>` 集合类。有时即使只有一个参数，你也会看到像这样的描述性名称，但无论如何，你通常会看到以 `T` 为前缀，这样在你的代码中使用它们时类型参数就会显眼。

# 泛型类型

类、结构体、记录和接口都可以是泛型，委托也可以，我们将在 第九章 中详细讨论它们。 示例 4-1 展示了如何定义一个泛型类。

##### 示例 4-1\. 定义泛型类

```cs
public class NamedContainer<T>
{
    public NamedContainer(T item, string name)
    {
        Item = item;
        Name = name;
    }

    public T Item { get; }
    public string Name { get; }
}
```

structs（结构体）、records（记录）和 interfaces（接口）的语法基本相同：类型名称紧跟着类型参数列表。 示例 4-2 展示了如何编写类似于 示例 4-1 中类的泛型记录。

##### 示例 4-2\. 定义泛型记录

```cs
public record NamedContainer<T>(T Item, string Name);
```

在通用类型的定义内部，我可以在通常会看到类型名称的任何地方使用类型参数`T`。在第一个示例中，我已将其用作构造函数参数的类型，在两个示例中都用作`Item`属性的类型。我也可以定义类型为`T`的字段。（实际上我已经这样做了，尽管不是显式地。自动属性会生成隐藏字段，因此我的`Item`属性将有一个关联的类型为`T`的隐藏字段。）你还可以定义类型为`T`的局部变量。并且你可以自由地将类型参数作为其他通用类型的参数。例如，我的`NamedContainer<T>`可以声明类型为`List<T>`的成员。

示例 4-1 和 4-2 定义的类型，像任何通用类型一样，都不是完整的类型。通用类型声明是*未绑定*的，意味着必须填入类型参数才能生成完整的类型。基本问题，例如`NamedContainer<T>`实例需要多少内存，无法在不知道`T`是什么的情况下回答——如果`T`是`int`，那么`Item`属性的隐藏字段将需要 4 字节，但如果是`decimal`，则需要 16 字节。如果 CLR 不知道如何安排内存中的内容，就不能为类型生成可执行代码。因此，为了使用这个或任何其他通用类型，我们必须提供类型参数。示例 4-3 展示了如何做到这一点。当提供类型参数时，结果有时被称为*构造类型*。（这与构造函数无关，我们在第三章中看过的一种特殊的成员类型。事实上，示例 4-3 也使用了它们——它调用了几个构造类型的构造函数。）

##### 示例 4-3\. 使用通用类

```cs
var a = new NamedContainer<int>(42, "The answer");
var b = new NamedContainer<int>(99, "Number of red balloons");
var c = new NamedContainer<string>("Programming C#", "Book title");
```

你可以在任何普通类型可以使用的地方使用构造的通用类型。例如，你可以将它们用作方法参数和返回值的类型，属性或字段的类型。你甚至可以将一个作为另一个通用类型的类型参数，就像示例 4-4 所示的那样。

##### 示例 4-4\. 作为类型参数的构造通用类型

```cs
// ...where a, and b come from Example 4-3. var namedInts = new List<NamedContainer<int>>() { a, b };
var namedNamedItem = new NamedContainer<NamedContainer<int>>(a, "Wrapped");
```

每次我向`NamedContainer<T>`提供不同的类型作为参数，都会构造一个不同的类型。（对于具有多个类型参数的通用类型，每个不同的类型参数组合将构造一个不同的类型。）这意味着`NamedContainer<int>`是一种不同于`NamedContainer<string>`的类型。这就是为什么在将`NamedContainer<int>`用作另一个`NamedContainer`的类型参数时不存在冲突的原因，正如示例 4-4 的最后一行所示——这里没有无限递归。

因为每组不同的类型参数产生不同的类型，所以在大多数情况下，同一通用类型的不同形式之间并没有隐含的兼容性。你不能将`NamedContainer<int>`赋给类型为`Nam⁠ed​Con⁠tai⁠ner⁠<str⁠ing>`的变量，反之亦然。这两种类型不兼容是有道理的，因为`int`和`string`是完全不同的类型。但如果我们使用`object`作为类型参数呢？正如第二章所述，几乎可以将任何东西放入一个`object`变量中。如果你写了一个参数类型为`object`的方法，传递一个`string`是可以的，因此你可能期望一个接受`NamedContainer<object>`的方法也接受`NamedContainer<string>`。然而这是行不通的，但某些通用类型（特别是接口和委托）可以声明它们希望具有这种兼容关系。支持这一点的机制（称为*协变*和*逆变*）与类型系统的继承机制密切相关。第六章详细讨论了继承和类型兼容性的这一方面，因此我将在那里讨论通用类型的这一方面。

类型参数的数量构成了未绑定通用类型的一部分身份。这使得可以引入具有相同名称但具有不同类型参数数量的多个类型。（类型参数数量的技术术语是*arity*。）

因此，你可以定义一个名为，比如说，`Operation<T>`的通用类，然后是另一个类，`Operation<T1, T2>`，还有`Operation<T1, T2, T3>`等等，都在同一命名空间中，而不会引入任何歧义。当你使用这些类型时，通过参数数量清楚地表明了使用的是哪种类型——例如，`Operation<int>`明显使用第一个，而`Operation<string, double>`使用第二个。出于同样的原因，一个非通用的`Operation`类会与具有相同名称的通用类型不同。

我的`NamedContainer<T>`示例对其类型参数`T`的实例不做任何操作——它从不调用任何方法或使用任何属性或其他成员的`T`。它所做的只是接受一个`T`作为构造函数参数，并将其存储以便稍后检索。运行时库中许多通用类型也是如此——我提到过一些集合类，它们都是对包含数据以便稍后检索这一主题的变体。

这是有原因的：通用类能够处理任何类型，因此对其类型参数的假设很少。然而，并非只能这样。你可以为你的类型参数指定*约束*。

# 约束

C#允许您声明类型实参必须满足某些要求。例如，假设您希望能够根据需要创建类型的新实例。示例 4-5 展示了一个简单的类，提供了延迟构造的功能——它通过静态属性提供了一个实例，但只有在第一次读取属性时才会尝试构造该实例。

##### 示例 4-5\. 创建参数化类型的新实例

```cs
// For illustration only. Consider using Lazy<T> in a real program. public static class Deferred<T>
    `where` `T` `:` `new``(``)`
{
    private static T? _instance;

    public static T Instance
    {
        get
        {
            if (_instance == null)
            {
                `_instance` `=` `new` `T``(``)``;`
            }
            return _instance;
        }
    }
}
```

###### 警告

在实践中，您不会编写这样的类，因为运行时库提供了`Lazy<T>`，它能够以更灵活的方式完成相同的工作。`Lazy<T>`可以在多线程代码中正确工作，而示例 4-5 则不行。示例 4-5 只是为了说明约束的工作原理。不要使用它！

为了使这个类能够完成它的工作，它需要能够构造一个供`T`类型实参使用的实例。`get`访问器使用了`new`关键字，由于没有传递参数，显然需要`T`提供无参构造函数。但并非所有类型都提供这样的构造函数，那么如果我们尝试使用一个没有适当构造函数的类型作为`Deferred<T>`的实参会发生什么？

编译器会拒绝这样做，因为它违反了这个泛型类型为`T`声明的约束条件。约束条件出现在类的开放大括号之前，并以`where`关键字开始。在示例 4-5 中的`new()`约束声明了`T`必须提供一个无参数的构造函数。

如果没有这个约束条件，示例 4-5 中的类将无法编译——在尝试构造`T`的新实例时会出错。泛型类型（或方法）只能使用通过约束指定的类型参数的特性，或者基本`object`类型定义的特性。（例如，`object`类型定义了`ToString`方法，因此您可以在任何类型的实例上调用它，而无需指定约束。）

C#仅提供了一套非常有限的约束条件。例如，您不能要求带有参数的构造函数。事实上，C#仅支持六种类型参数的约束：类型约束、引用类型约束、值类型约束、`notnull`、`unmanaged`以及`new()`约束。我们刚刚看到了最后一种，现在让我们来看看其他几种。

## 类型约束

您可以约束类型参数的实参与特定类型兼容。例如，您可以使用这个特性要求实参类型实现特定接口。示例 4-6 展示了相应的语法。

##### 示例 4-6\. 使用类型约束

```cs
public class GenericComparer<T> : IComparer<T>
    `where` `T` `:` `IComparable``<``T``>`
{
    public int Compare(T? x, T? y)
    {
        if (x == null) { return y == null ? 0 : -1; }
        return x.CompareTo(y);
    }
}
```

在描述如何利用类型约束之前，我将简要解释此示例的目的。这个类提供了.NET 中两种值比较风格之间的桥梁。某些数据类型提供它们自己的比较逻辑，但有时将比较作为独立函数实现在其自己的类中可能更有用。这两种风格分别由`IComparable<T>`和`IComparer<T>`接口代表，它们都是运行时库的一部分（分别位于`System`和`System.Collections.Generic`命名空间中）。我在第三章展示了`IComparer<T>`——这个接口的实现可以比较两个类型为`T`的对象或值。该接口定义了一个`Compare`方法，接受两个参数，根据第一个参数分别小于、等于或大于第二个参数时返回负数、0 或正数。`IComparable<T>`非常相似，但其`CompareTo`方法只接受一个参数，因为使用此接口时，你要求一个实例比较*它自己*与另一个实例。

运行时库的某些集合类要求你提供一个`IComparer<T>`来支持排序等操作。它们使用这种模式，其中一个单独的对象执行比较，因为这比`IComparable<T>`模式有两个优点。首先，它使你能够使用不实现`IComparable<T>`的数据类型。其次，它允许你插入不同的排序顺序。（例如，假设你希望使用不区分大小写的顺序对一些字符串进行排序。`string`类型实现了`IComparable<string>`，但它提供了区分大小写、具有特定区域设置的顺序。）因此，`IComparer<T>`是更灵活的模式。但是，如果你使用实现了`IComparable<T>`的数据类型，并且你对其提供的顺序非常满意，那么当你在使用要求`IComparer<T>`的 API 时，你会怎么做呢？

实际上，答案是你可能只需使用.NET 专为这种情况设计的功能：`Comparer<T>.Default`。如果`T`实现了`IComparable<T>`，该属性将返回一个精确满足你需求的`IComparer<T>`。因此，在实践中，你不需要编写示例 4-6 中的代码，因为 Microsoft 已经为你编写了。然而，看到如何编写自己版本仍然很有教育意义，因为它展示了如何使用类型约束。

以`where`关键字开头的那行声明，说明这个泛型类要求其类型参数`T`实现`IComparable<T>`。如果没有这个限制，`Compare`方法将无法编译——它在类型为`T`的参数上调用`CompareTo`方法。该方法并不适用于所有对象，C#编译器之所以允许这样做，只是因为我们约束`T`必须实现提供这种方法的接口。

接口约束有点奇怪：乍一看，似乎我们真的不应该需要它们。如果一个方法需要特定的参数来实现特定的接口，你通常会将该接口作为参数的类型。然而，示例 4-6 无法做到这一点。你可以通过尝试示例 4-7 来证明这一点。它将无法编译通过。

##### 示例 4-7\. 编译失败：未实现接口

```cs
public class GenericComparer<T> : IComparer<T>
{
    public int Compare(IComparable<T>? x, T? y)
    {
        if (x == null) { return y == null ? 0 : -1; }
        return x.CompareTo(y);
    }
}
```

编译器会抱怨我没有实现`IComparer<T>`接口的`Compare`方法。示例 4-7 有一个`Compare`方法，但其签名是错误的——第一个参数应该是`T`。我也可以尝试不指定约束的正确签名，就像示例 4-8 所示。

##### 示例 4-8\. 编译失败：缺少约束

```cs
public class GenericComparer<T> : IComparer<T>
{
    public int Compare(T? x, T? y)
    {
        if (x == null) { return y == null ? 0 : -1; }
        return x.CompareTo(y);
    }
}
```

这也无法通过编译，因为编译器找不到我试图使用的`CompareTo`方法。在示例 4-6 中对`T`的约束使编译器能够了解该方法的真正含义。

顺便说一句，类型约束不一定是接口。你可以使用任何类型。例如，你可以要求特定类型参数派生自特定基类。更微妙的是，你还可以根据另一个类型参数来定义一个参数的约束。例如，示例 4-9 要求第一个类型参数派生自第二个类型参数。

##### 示例 4-9\. 将一个参数约束为从另一个派生

```cs
public class Foo<T1, T2>
    where T1 : T2
...
```

类型约束非常具体——它们要求特定的继承关系或实现某些接口。然而，你可以定义稍微不那么具体的约束。

## 引用类型约束

可以将类型参数约束为引用类型。如示例 4-10 所示，这看起来类似于类型约束。你只需使用关键字`class`而不是类型名称。如果你处于启用的可空注解上下文中，此注解的含义会发生变化：它要求类型参数为非空引用类型。如果指定`class?`，则允许类型参数为可空或非空引用类型。

##### 示例 4-10\. 要求引用类型的约束

```cs
public class Bar<T>
    where T : class
...
```

这个约束会阻止将值类型，如 `int`、`double` 或任何 `struct`，用作类型参数。其存在使得你的代码能够做三件否则不可能做到的事情。首先，它意味着你可以编写测试相关类型变量是否为 `null` 的代码。² 如果你没有将类型约束为引用类型，它始终有可能是值类型，而这些类型不能有 `null` 值。第二个能力是你可以将其用作 `as` 运算符的目标类型，我们将在第六章中讨论这一点。这实际上只是第一个特性的变体——`as` 关键字需要一个引用类型，因为它可能产生一个 `null` 结果。

###### 注意

`class` 约束会阻止使用可空类型，如 `int?`（或 CLR 中称为 `Nullable<int>`）。虽然你可以测试 `int?` 是否为 `null` 并使用 `as` 运算符，但编译器对可空类型和引用类型在这两个操作上生成的代码完全不同。如果你使用这些特性，它就不能编译一个可以处理引用类型和可空类型的单个方法。

引用类型约束的第三个功能是能够使用某些其他泛型类型。对于泛型代码来说，将其中一个类型参数用作另一个泛型类型的参数通常很方便，如果另一个类型指定了约束，你需要在自己的类型参数上放置相同的约束。因此，如果其他某个类型指定了类约束，这可能要求你以相同的方式约束自己的某个参数。

当然，这确实引出了为什么你正在使用的类型首先需要这个约束的问题。也许它只是想要测试 `null` 或使用 `as` 运算符，但应用这个约束还有另一个原因。有时候，你只需要一个类型参数是引用类型——有些泛型方法可能能够在没有 `class` 约束的情况下编译，但如果与值类型一起使用，它将无法正确工作。为了说明这一点，我将描述我有时发现自己需要使用这种约束的场景。

我经常编写测试，创建被测试类的实例，并且也需要一个或多个虚拟对象来替代真实对象，以便与被测试对象交互。使用这些替身减少了每个单独测试需要执行的代码量，并且可以更轻松地验证被测试对象的行为。例如，我的测试可能需要验证我的代码在正确的时机向服务器发送消息，但我不想在单元测试期间运行真实服务器，因此我提供了一个对象，它实现了与负责传输消息的类相同的接口，但实际上并不会发送消息。被测试对象加上一个虚拟对象的组合是一种常见的模式，可能有助于将代码放入可重用的基类中。使用泛型意味着该类可以适用于被测试的类型和虚拟类型的任意组合。示例 4-11 展示了我在这些情况下有时编写的一种辅助类的简化版本。

##### 示例 4-11\. 受另一个约束的限制

```cs
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Moq;

public class TestBase<TSubject, TFake>
    where TSubject : new()
    where TFake : class
{
    public TSubject? Subject { get; private set; }
    public Mock<TFake>? Fake { get; private set; }

 [TestInitialize]
    public void Initialize()
    {
        Subject = new TSubject();
        Fake = new Mock<TFake>();
    }
}
```

有多种方法可以构建用于测试目的的虚拟对象。您可以编写实现与您的真实对象相同接口的新类，但也有第三方库可以生成它们。一个这样的库称为 Moq（一个 [免费提供的开源项目](https://github.com/Moq)），`Mock<T>` 类就是来自于这里，在 示例 4-11 中。它能够生成任何接口或任何未密封类的虚拟实现。(第六章 描述了 `sealed` 关键字。) 它默认提供所有成员的空实现，并且如果需要，您还可以配置更有趣的行为。您还可以验证代码在使用虚拟对象时是否按预期进行了使用。

这与约束有什么关系？`Mock<T>` 类在其自身类型参数 `T` 上指定了引用类型约束。这是因为它在运行时创建类型的动态实现的方式；这种技术仅适用于引用类型。Moq 在运行时生成类型，如果 `T` 是一个接口，则生成的类型将实现它；而如果 `T` 是一个类，则生成的类型将从它派生。³ 如果 `T` 是一个结构体，它将无法执行任何有用的操作，因为不能从值类型派生。这意味着当我在 示例 4-11 中使用 `Mock<T>` 时，我需要确保传递的任何类型参数不是结构体（即必须是引用类型）。但是，我使用的类型参数是我的类的类型参数之一：`TFake`。因此，我不知道那将是什么类型——这将取决于谁在使用我的类。

为了使我的类能够编译而不报错，我必须确保已满足我使用的任何泛型类型的约束。我必须保证`Mock<TFake>`是有效的，而唯一的方法就是在自己的类型上添加一个要求`TFake`为引用类型的约束。这就是在类定义的第三行中做的事情，在示例 4-11 中。如果没有这个，编译器会在引用`Mock<TFake>`的两行上报错。

总的来说，如果你想使用自己的类型参数作为泛型的类型参数，并指定一个约束条件，你需要在自己的类型参数上也指定相同的约束条件。

## 值类型约束

就像你可以约束一个类型参数为引用类型一样，你也可以约束它为值类型。如示例 4-12 所示，语法与引用类型约束类似，但使用`struct`关键字。

##### 示例 4-12\. 要求值类型的约束

```cs
public class Quux<T>
    where T : struct
...
```

到目前为止，我们只在自定义值类型的上下文中看到过`struct`关键字，但尽管它的外观如此，此约束允许`bool`、`enum`类型以及任何内置数值类型（如`int`），以及自定义结构体。

.NET 的`Nullable<T>`类型施加了这个约束。从第三章回忆起，`Nullable<T>`为值类型提供了一个包装器，允许变量既可以持有一个值，也可以没有值。（通常我们使用 C#提供的特殊语法，例如，我们会写`int?`而不是`Nullable<int>`。）这种类型存在的唯一理由是为不能持有 null 值的类型提供可空性。因此，只有将此类型用于值类型才有意义——引用类型变量已经可以被设置为`null`而不需要这个包装器。值类型约束阻止你将`Nullable<T>`用于不需要它的类型。

## 使用非托管约束实现全面的值类型

你可以指定`unmanaged`作为约束条件，这要求类型参数是一个值类型，但也要求它不包含引用。该类型的所有字段必须是值类型，如果任何字段不是内置基元类型，则其类型必须进一步仅包含值类型字段，以此类推直到底部。实际上，这意味着所有实际数据必须是固定集合中的一种内置类型（基本上是所有数值类型、`bool`或指针）或`enum`类型。这主要在互操作场景中非常重要，因为符合`unmanaged`约束的类型可以安全高效地传递给非托管代码。

## 非空约束

如果使用了第三章描述的可空引用特性（在创建新项目时默认启用），则可以指定`notnull`约束。这允许值类型或非可空引用类型，但不允许可空引用类型。

## 其他特殊类型约束

第三章描述了各种特殊类型，包括枚举类型（`enum`）和委托类型（在第九章中详细介绍）。有时将类型参数约束为这些类型之一是很有用的。不过，这没有什么特别的技巧：你只需使用类型约束即可。所有委托类型都派生自`System.Delegate`，所有枚举类型都派生自`System.Enum`。正如示例 4-13 所示，你可以编写一个约束类型参数必须派生自其中任何一个的类型约束。

##### 示例 4-13\. 要求委托和`enum`类型的约束

```cs
public class RequireDelegate<T>
    where T : Delegate
{
}

public class RequireEnum<T>
    where T : Enum
{
}
```

## 多重约束

如果你希望对单个类型参数施加多重约束，可以将它们放在一个列表中，正如示例 4-14 所示。有一些限制。你不能结合使用`class`、`struct`、`notnull`或`unmanaged`约束 —— 这些是互斥的。如果使用了其中一个关键字，必须将其放在列表的最前面。如果存在`new()`约束，则必须放在最后。

##### 示例 4-14\. 多重约束

```cs
public class Spong<T>
    where T : IEnumerable<T>, IDisposable, new()
...
```

当你的类型具有多个类型参数时，需要为每个想要约束的类型参数编写一个`where`子句。实际上，我们在前面看到了这一点 —— 示例 4-11 为其两个参数定义了约束。

# 零值类似的值

所有类型都支持的某些特性，因此不需要约束。这包括由`object`基类定义的方法集，详见第三章和第六章。但在泛型代码中有时可以使用更基本的特性。

任何类型的变量都可以初始化为默认值。正如在前面章节中所看到的，有些情况下 CLR 会为我们做这件事。例如，新构造对象中的所有字段将具有已知值，即使我们没有编写字段初始化器并且没有在构造函数中提供值。同样，任何类型的新数组将所有元素初始化为已知值。CLR 通过填充相关内存区域为零来完成此操作。这的确切含义取决于数据类型。对于任何内置数值类型，该值将几乎肯定是数字`0`，但对于非数值类型，情况则不同。对于`bool`，默认值是`false`，对于引用类型，则为`null`。

有时，对于通用代码而言，能够将变量设置为这种初始默认的零值可能非常有用。但在大多数情况下，您无法使用文字表达式来完成这一点。您不能将`null`赋给一个由类型参数指定的变量，除非该参数已被约束为引用类型。并且您也不能将字面量`0`赋给任何此类变量，因为当前没有一种方法来约束类型参数为数值类型。

相反，您可以使用`default`关键字请求任何类型的零值。（这与我们在第二章中在`switch`语句内部看到的相同关键字，但用法完全不同。C#保持了 C 家族传统，为每个关键字定义多个不相关的含义。）如果您编写`default(*SomeType*)`，其中`*SomeType*`可以是特定类型或类型参数，则会获得该类型的默认初始值：如果是数值类型，则为`0`，对于任何其他类型，则为其等效值。例如，表达式`default(int)`的值为`0`，`default(bool)`为`false`，`default(string)`为`null`。您可以将其与泛型类型参数一起使用，以获取相应类型参数的默认值，如示例 4-15 所示。

##### 示例 4-15\. 获取类型参数的默认（类似零值）值

```cs
static void ShowDefault<T>()
{
    Console.WriteLine(default(T));
}
```

在定义类型参数`T`的泛型类型或方法内部，表达式`default(T)`将产生类型`T`的默认零值——无论`T`是什么——而无需约束。因此，您可以使用示例 4-15 中的泛型方法来验证我所述的`int`、`bool`和`string`的默认值。

###### 注意

当启用了可空引用功能（在第三章中描述）时，编译器将考虑`default(T)`作为一个可能为空的值，除非您通过应用`struct`约束来排除引用类型的使用。

在编译器能够推断所需类型的情况下，您可以使用更简单的形式。而不是编写`default(T)`，您只需编写`default`。这在示例 4-15 中是行不通的，因为`Console.WriteLine`几乎可以接受任何东西，所以编译器无法缩小到一个选项，但在示例 4-16 中可以正常工作，因为编译器可以看到泛型方法的返回类型是`T`，所以这必须需要一个`default(T)`。由于它可以推断出来，我们只需写`default`。

##### 示例 4-16\. 获取推断类型的默认（类似零值）

```cs
static T? GetDefault<T>() => default;
```

而且，既然我刚刚向您展示了一个示例，这似乎是一个谈论泛型方法的好时机。

# 泛型方法

除了泛型类型，C# 还支持泛型方法。在这种情况下，泛型类型参数列表位于方法名之后，并且在方法的普通参数列表之前。示例 4-17 展示了一个具有单个类型参数的方法。它将该参数用作其返回类型，并将其用作将传递给方法的数组的元素类型。该方法返回数组中的最后一个元素，并且因为它是泛型的，所以对于任何数组元素类型都有效。

##### 示例 4-17\. 泛型方法

```cs
public static T GetLast<T>(T[] items) => items[¹];
```

###### 注意

您可以在泛型类型或非泛型类型中定义泛型方法。如果泛型方法是泛型类型的成员，则包含类型的所有类型参数都在方法内部有效，以及方法特定的类型参数。

与泛型类型类似，您可以通过指定方法名和类型参数来使用泛型方法，如示例 4-18 所示。

##### 示例 4-18\. 调用泛型方法

```cs
int[] values = { 1, 2, 3 };
int last = GetLast<int>(values);
```

泛型方法与泛型类型类似，但类型参数仅在方法声明和方法体内有效。您可以像处理泛型类型那样指定约束条件。如示例 4-19 所示，约束条件出现在方法参数列表后和方法体前。

##### 示例 4-19\. 具有约束条件的泛型方法

```cs
public static T MakeFake<T>()
    where T : class
{
    return new Mock<T>().Object;
}
```

然而，泛型方法与泛型类型有一个显著的区别：您不必总是显式指定泛型方法的类型参数。

## 类型推断

C# 编译器通常能够推断出泛型方法的类型参数。例如，我可以通过从方法调用中移除类型参数列表来修改示例 4-18，如示例 4-20 所示。这并不改变代码的含义。

##### 示例 4-20\. 泛型方法类型参数推断

```cs
int[] values = { 1, 2, 3 };
int last = GetLast(values);
```

当遇到这种普通的方法调用时，如果没有同名的非泛型方法可用，编译器会开始寻找合适的泛型方法。如果示例 4-17 中的方法在作用域内，则它将是一个候选项，并且编译器将尝试推断类型参数。这是一个相当简单的情况。该方法期望一个类型为`T`的数组，而我们传递了一个元素类型为`int`的数组，因此很容易推断出这段代码应该被视为对`GetLast<int>`的调用。

随着更复杂的案例的出现，情况变得更加复杂。C# 规范大约有六页专门讨论类型推断算法，但它的目标始终是：在类型参数冗余时让您可以省略类型参数。顺便说一句，类型推断始终在编译时进行，因此它基于方法参数的静态类型。

对于广泛使用泛型的 API（例如 LINQ，这是第 10 章的主题），显式列出每个类型参数可能会使代码非常难以理解，因此通常依赖类型推断。如果使用匿名类型，则类型参数推断变得至关重要，因为无法显式提供类型参数。

# 泛型和元组

C# 的轻量级元组具有独特的语法，但在运行时看来，它们并没有什么特别之处。它们都只是一组通用类型的实例。看看示例 4-21。这里使用 `(int, int)` 作为局部变量的类型，表示它是一个包含两个 `int` 值的元组。

##### 示例 4-21\. 正常方式声明元组变量

```cs
(int, int) p = (42, 99);
```

现在看看示例 4-22。这里使用了位于 `System` 命名空间中的 `ValueTuple<int, int>` 类型。但这与示例 4-21 中的声明完全等效。在 Visual Studio 或 VS Code 中，如果你将鼠标悬停在 `p2` 变量上，它将报告其类型为 `(int, int)`。

##### 示例 4-22\. 声明带有其底层类型的元组变量

```cs
ValueTuple<int, int> p2 = (42, 99);
```

C# 的特殊语法允许给元组元素命名，这是其特有的一点。`ValueTuple` 系列为其元素命名为 `Item1`、`Item2`、`Item3` 等，但在 C# 中我们可以选择其他名称。当你声明一个带有命名元组元素的局部变量时，这些名称在 C# 中实际上是虚构的——在运行时完全没有表现。但是，当一个方法返回一个元组时，例如在示例 4-23 中，情况就不同了：这些名称需要可见，以便消费此方法的代码可以使用相同的名称。即使此方法位于我代码已引用的某个库组件中，我也希望能够编写 `Pos().X`，而不是必须使用 `Pos().Item1`。

##### 示例 4-23\. 返回一个元组

```cs
public static (int X, int Y) Pos() => (10, 20);
```

为了使这一点实现，编译器将一个名为 `TupleElementNames` 的属性应用于方法的返回值，其中包含一个列出要使用的属性名称的数组。(第 14 章描述了属性。) 你实际上不能自己编写能够执行此操作的代码：如果你编写一个返回 `ValueTuple<int, int>` 的方法，并尝试将 `TupleElementNamesAttribute` 作为 `return` 属性应用，编译器将生成错误消息，告诉你不要直接使用此属性，而是使用元组语法。但是编译器正是通过该属性来报告元组元素的名称。

请注意，运行库中还有另一组元组类型，`Tuple<T>`、`Tuple<T1, T2>`等。这些几乎与`ValueTuple`系列看起来相同。不同之处在于，`Tuple`系列的泛型类型都是类，而所有`ValueTuple`类型都是结构体。C#的轻量级元组语法仅使用`ValueTuple`系列。尽管如此，`Tuple`系列在运行库中已经存在很长时间了，因此在旧代码中经常看到它们，这些代码需要将一组值捆绑在一起而不需要为此定义新类型。

# 在泛型内部

如果你熟悉 C++模板，现在应该已经注意到 C#泛型与模板有很大不同。表面上看，它们有一些相似之处，并且可以用类似的方式使用——例如，都适用于实现集合类。然而，有些基于模板的技术在 C#中根本行不通，比如示例 4-24 中的代码。

##### 示例 4-24\. C# 泛型中无法工作的模板技术

```cs
public static T Add<T>(T x, T y)
{
    return x + y;  // Will not compile
}
```

在 C++模板中可以做这种事情，但在 C#中不行，并且无法通过约束完全修复。你可以添加一个类型约束，要求`T`从某个类型派生或实现某个定义了自定义`+`运算符的接口，这样就可以编译通过，但这相当有限——它只适用于从该基类型派生的类型。在 C++中，你可以编写一个模板，它将任何支持加法的类型的两个项相加在一起，无论是内置类型还是自定义类型。此外，C++模板不需要约束；编译器能够自行判断特定类型是否适用作为模板参数。

这个问题并非特定于算术运算。根本问题在于，由于泛型代码依赖于约束来确定其类型参数上可用的操作，它只能使用作为接口成员或共享基类的特性。如果.NET 中的算术运算是基于接口的，那么可以定义一个需要该接口的约束。但是操作符都是静态方法，尽管接口可以包含静态成员，⁴ 但没有支持的方法让各个类型提供自己的实现——允许每个类型提供自己接口实现的动态调度机制仅适用于实例成员。⁵

C# 泛型的限制是由其设计原理决定的，因此理解其机制非常有用。（顺便说一下，这些限制并不特定于任何特定的 CLR 实现。它们是泛型如何融入.NET 运行时设计的必然结果。）

通用方法和类型在编译时并不知道将用作参数的具体类型。这是 C#泛型与 C++模板之间的根本区别——在 C++中，编译器可以看到模板的每个实例化。但在 C#中，你可以在编译代码很久之后，实例化泛型类型，而无需访问任何相关源代码。毕竟，微软多年前就写了通用的`List<T>`类，但你今天完全可以写一个全新的类，并作为类型参数嵌入其中。 （你可能会指出 C++标准库的`std::vector`存在更久。然而，C++编译器可以访问定义类的源文件，而对于 C#和`List<T>`来说则不然。C#只看到已编译的库。）

结果是，C#编译器需要足够的信息来在编译泛型代码时生成类型安全的代码。看看示例 4-24。它无法知道这里的+运算符具体是什么意思，因为对于不同的类型它可能是不同的。对于内置数值类型，该代码需要编译为执行加法的特定中间语言（IL）指令。如果该代码位于检查上下文中（即使用第 2 章中显示的`checked`关键字），我们可能已经遇到问题，因为使用溢出检查的整数加法代码会为有符号和无符号整数使用不同的 IL 操作码。此外，由于这是一个泛型方法，我们可能根本不处理内置数值类型——也许我们正在处理定义了自定义`+`运算符的类型，在这种情况下，编译器需要生成一个方法调用。（自定义运算符实际上就是方法。）或者如果相关类型不支持加法操作，编译器应该生成一个错误。

对于编译简单的加法表达式，存在几种可能的结果，这取决于实际涉及的类型。当编译器知道类型时，这很好，但它必须在不知道将用作参数的类型的情况下编译泛型类型和方法的代码。

或许你会认为微软可以支持一种类似于泛型代码的暂定半编译格式，从某种意义上说，它确实做到了。在引入泛型时，微软修改了类型系统、文件格式和 IL 指令，允许泛型代码使用代表类型参数的占位符，以便在类型完全构造时填充。那么为什么不扩展以处理运算符？为什么不让编译器在编译试图使用泛型类型的代码时生成错误，而不是坚持在编译泛型代码本身时生成错误呢？好吧，事实证明，你可以在运行时插入新的类型参数集合——我们将在第十三章看到的反射 API 允许你构造泛型类型。在明显出现错误的时间点可能没有可用的编译器，因为并非所有 .NET 版本都提供了 C# 编译器的副本。无论如何，如果一个泛型类是用 C# 编写的，但被完全不同的语言消费，也许这种语言不支持操作符重载，那么该使用哪种语言的规则来决定对`+`操作符的处理呢？应该是编写泛型代码的语言还是编写类型参数的语言呢？（如果有多个类型参数，并且每个参数使用不同语言编写的类型，那又该怎么办呢？）或者规则应该来自于决定将类型参数插入泛型类型或方法的语言，但是如果一段泛型代码将其参数传递给其他泛型实体呢？即使你能决定哪种方法最好，这也假设在运行时确定一行代码的含义所使用的规则是可用的，这一假设再次因为运行代码的机器上可能没有相关的编译器而遇到困难。

.NET 泛型通过在泛型代码编译时使用编写泛型代码的语言的规则，要求完全定义泛型代码的含义来解决这个问题。如果泛型代码涉及使用方法或其他成员，它们必须在编译时静态解析（即这些成员的标识必须在编译时精确确定）。关键在于，这意味着泛型代码本身的编译时间，而不是消费泛型代码的代码的编译时间。这些要求解释了为什么 C# 泛型不像 C++ 使用的消费者编译时替换模型那样灵活。回报是，你可以将泛型编译成二进制形式的库，并且它们可以被支持泛型的任何 .NET 语言使用，具有完全可预测的行为。

# 摘要

泛型使我们能够编写带有类型参数的类型和方法，在编译时可以填充这些参数，从而生成适用于特定类型的不同版本的类型或方法。在它们首次引入时，泛型的最重要用例之一是使得编写类型安全的集合类成为可能，比如`List<T>`。我们将在下一章节中查看一些这样的集合类型。

¹ 在说泛型类型名称时，惯例是使用“of”这个词，比如“List of T”或“List of int”。

² 即使在启用了可空注解上下文中使用了普通的`class`约束，也是允许的。可空引用特性并不能完全保证非空性，因此允许与`null`进行比较。

³ Moq 依赖于 Castle 项目的*动态代理*功能来生成这种类型。如果您想在您的代码中使用类似的东西，可以在[Castle 项目](http://castleproject.org)找到它。

⁴ 静态接口成员在.NET Framework 中不可用。

⁵ 已经有一个提案用于为静态接口成员添加动态调度。尽管它不是官方的 C# 10.0 的一部分，但.NET 6.0 SDK 包含了一个预览实现。您可以通过将`EnablePreviewFeatures`项目属性设置为 true 来尝试它。如果在未来版本中得到支持，也许我们会看到一个`IAddable<T>`。