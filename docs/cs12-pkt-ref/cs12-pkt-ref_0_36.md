# 第三十六章：动态绑定

*动态绑定*延迟了*绑定*—解析类型、成员和运算符的过程—从编译时到运行时。动态绑定在编译时*您*知道某个函数、成员或运算符存在，但*编译器*不知道时非常有用。这通常发生在与动态语言（如 IronPython）和 COM 的互操作以及您可能使用反射的情况下。

动态类型通过使用上下文关键字`dynamic`声明：

```cs
dynamic d = GetSomeObject();
d.Quack();
```

动态类型指示编译器放松。我们期望`d`的运行时类型具有`Quack`方法。我们只是不能在静态上下文中证明它。因为`d`是动态的，编译器将`Quack`绑定到`d`直到运行时。理解这意味着需要区分*静态绑定*和*动态绑定*。

## 静态绑定与动态绑定

典型的绑定示例是在编译表达式时将名称映射到特定函数。要编译以下表达式，编译器需要找到名为`Quack`的方法的实现：

```cs
d.Quack();
```

假设`d`的静态类型是`Duck`：

```cs
Duck d = ...
d.Quack();
```

在最简单的情况下，编译器通过查找`Duck`上名为`Quack`的无参方法来进行绑定。如果失败，编译器会扩展其搜索到带有可选参数的方法、`Duck`的基类方法以及以`Duck`作为第一个参数的扩展方法。如果找不到匹配项，你将会得到一个编译错误。不管绑定了什么方法，最终的结果是绑定是由编译器完成的，并且绑定完全依赖于静态知道操作数的类型（在这种情况下是`d`）。这就是*静态绑定*。

现在让我们将`d`的静态类型改为`object`：

```cs
object d = ...
d.Quack();
```

调用`Quack`会导致编译错误，因为尽管存储在`d`中的值可能包含名为`Quack`的方法，但编译器无法知道这一点，因为它仅仅知道变量的类型，而在这种情况下是`object`。但现在让我们将`d`的静态类型改为`dynamic`：

```cs
dynamic d = ...
d.Quack();
```

`dynamic`类型就像`object`—它同样不描述类型。不同之处在于它允许你以编译时不知道的方式使用它。动态对象基于其运行时类型而不是编译时类型在运行时进行绑定。当编译器看到动态绑定表达式（通常是包含类型为`dynamic`的任何值的表达式），它只是打包表达式以便稍后在运行时进行绑定。

在运行时，如果动态对象实现了`IDynamicMeta​Ob⁠jectProvider`，那么这个接口将被用来执行绑定。如果没有，绑定几乎与编译器知道动态对象的运行时类型时的方式相同。这两种选择被称为*自定义绑定*和*语言绑定*。

## 自定义绑定

自定义绑定发生在一个动态对象实现了`IDynamicMetaObjectProvider`（IDMOP）时。虽然你可以在你用 C#编写的类型上实现 IDMOP，并且这样做很有用，但更常见的情况是你从在.NET 上实现的动态语言（如 IronPython 或 IronRuby）中获得了一个 IDMOP 对象。这些语言的对象隐式地实现了 IDMOP，作为直接控制在它们上执行的操作意义的一种手段。这里有一个简单的例子：

```cs
dynamic d = new Duck();
d.Quack();       // Quack was called
d.Waddle();      // Waddle was called

public class Duck : DynamicObject   // in System.Dynamic
{
  public override bool TryInvokeMember (
    InvokeMemberBinder binder, object[] args,
    out object result)
  {
    Console.WriteLine (binder.Name + " was called");
    result = null;
    return true;
  }
}
```

`Duck`类实际上没有`Quack`方法。相反，它使用自定义绑定来拦截和解释所有方法调用。我们在*C# 12 in a Nutshell*中详细讨论了自定义绑定器。

## 语言绑定

语言绑定发生在一个动态对象没有实现`IDynamicMetaObjectProvider`时。语言绑定在你处理.NET 类型系统中的不完美设计类型或固有限制时非常有用。例如，内置的数值类型不完美，因为它们没有共同的接口。我们已经看到方法可以动态绑定；对于运算符也是如此：

```cs
int x = 3, y = 4;
Console.WriteLine (Mean (x, y));

dynamic Mean (dynamic x, dynamic y) => (x+y) / 2;
```

这带来的好处很明显——您不需要为每种数字类型重复编写代码。但是，您失去了静态类型安全性，冒着运行时异常的风险，而不是编译时错误。

###### 注意

动态绑定规避了静态类型安全性，但没有规避运行时类型安全性。与反射不同，您不能通过动态绑定规避成员可访问性规则。

设计上，语言运行时绑定的行为尽可能类似于静态绑定，如果动态对象的运行时类型在编译时已知，则程序行为与硬编码 `Mean` 以适配 `int` 类型的行为相同。静态绑定和动态绑定之间在一致性方面最显著的例外是扩展方法，我们在“不可调用函数”中讨论了此问题。

###### 注意

动态绑定也会带来性能损失。但由于 DLR 的缓存机制，对相同动态表达式的重复调用进行了优化，允许您在循环中高效地调用动态表达式。这种优化使得在当今硬件上进行简单动态表达式的典型开销降低到少于 100 ns。

## RuntimeBinderException

如果成员绑定失败，将抛出 `RuntimeBinderException`。您可以将其视为运行时的编译时错误：

```cs
dynamic d = 5;
d.Hello();       // throws RuntimeBinderException
```

抛出异常是因为 `int` 类型没有 `Hello` 方法。

## 动态的运行时表示

`dynamic` 与 `object` 类型之间存在深刻的等价性。运行时将以下表达式视为 `true`：

```cs
typeof (dynamic) == typeof (object)
```

这个原则适用于构造类型和数组类型：

```cs
typeof (List<dynamic>) == typeof (List<object>)
typeof (dynamic[]) == typeof (object[])
```

类似于对象引用，动态引用可以指向任何类型的对象（除指针类型外）：

```cs
dynamic x = "hello";
Console.WriteLine (x.GetType().Name);  // String

x = 123;  // No error (despite same variable)
Console.WriteLine (x.GetType().Name);  // Int32
```

结构上，对象引用和动态引用没有区别。动态引用简单地允许对其指向的对象进行动态操作。您可以从 `object` 转换为 `dynamic`，以在 `object` 上执行任何动态操作。

```cs
object o = new System.Text.StringBuilder();
dynamic d = o;
d.Append ("hello");
Console.WriteLine (o);   // hello
```

## 动态转换

`dynamic` 类型与所有其他类型都有隐式转换。为了成功转换，动态对象的运行时类型必须隐式可转换为目标静态类型。

以下示例抛出 `RuntimeBinderException`，因为 `int` 不能隐式转换为 `short`：

```cs
int i = 7;
dynamic d = i;
long l = d;       // OK - implicit conversion works
short j = d;      // throws RuntimeBinderException
```

## `var` 与 `dynamic` 的区别很深。`var` 让 *编译器* 推断类型。

`var` 和 `dynamic` 类型在外表上相似，但其内在差异深远：

+   `var` 表示，“让 *编译器* 推断类型。”

+   `dynamic` 表示，“让 *运行时* 确定类型。”

举个例子：

```cs
dynamic x = "hello";  // Static type is dynamic
var y = "hello";      // Static type is string
int i = x;            // Runtime error
int j = y;            // Compile-time error
```

## 动态表达式

字段、属性、方法、事件、构造函数、索引器、操作符和转换都可以动态调用。

禁止使用 `void` 返回类型来消耗动态表达式的结果，就像使用静态类型表达式一样。不同之处在于错误发生在运行时。

通常涉及动态操作数的表达式本身也是动态的，因为缺少类型信息的影响是级联的：

```cs
dynamic x = 2;
var y = x * 3;       // Static type of y is dynamic
```

这个规则有几个明显的例外。首先，将动态表达式强制转换为静态类型会产生静态表达式。其次，构造函数调用始终产生静态表达式——即使使用动态参数调用时也是如此。

此外，还有一些边缘情况，在这些情况下，包含动态参数的表达式是静态的，包括将索引传递给数组和委托创建表达式。

## 动态成员重载解析

使用`dynamic`的经典用例涉及动态*接收器*。这意味着动态对象是动态函数调用的接收器：

```cs
dynamic x = ...;
x.Foo (123);          // x is the receiver
```

然而，动态绑定不限于接收器：方法参数也适用于动态绑定。使用动态参数调用函数的效果是将重载解析从编译时推迟到运行时：

```cs
static void Foo (int x)    => Console.WriteLine ("int");
static void Foo (string x) => Console.WriteLine ("str");

static void Main()
{
  dynamic x = 5;
  dynamic y = "watermelon";

  Foo (x);    // int
  Foo (y);    // str
}
```

运行时重载解析也称为*多分派*，在实现*访问者*等设计模式中非常有用。

如果没有涉及动态接收器，编译器可以静态地执行基本检查，以查看动态调用是否会成功：它检查是否存在一个名称和参数数目正确的函数。如果找不到候选项，则会得到编译时错误。

如果一个函数被调用时使用了动态和静态参数的混合方式，最终的方法选择将反映动态和静态绑定决策的混合：

```cs
static void X(object x, object y) =>Console.Write("oo");
static void X(object x, string y) =>Console.Write("os");
static void X(string x, object y) =>Console.Write("so");
static void X(string x, string y) =>Console.Write("ss");

static void Main()
{
  object o = "hello";
  dynamic d = "goodbye";
  X (o, d);               // os
}
```

对`X(o,d)`的调用是动态绑定的，因为它的一个参数`d`是`dynamic`类型。但因为`o`是静态已知的，尽管绑定是动态发生的，它将利用静态类型。在这种情况下，重载解析将由于`o`的静态类型和`d`的运行时类型而选择`X`的第二个实现。换句话说，编译器“尽可能静态”。

## 不可调用的函数

有些函数无法动态调用。以下情况下不能调用：

+   扩展方法（通过扩展方法语法）

+   通过接口的任何成员（通过接口）

+   子类隐藏的基类成员

这是因为动态绑定需要两个信息：要调用的函数的名称，以及要在其上调用函数的对象。然而，在这三种不可调用的情况中，涉及到一个*额外类型*，这种类型只有在编译时才知道。并且没有办法动态指定这些额外的类型。

当调用扩展方法时，这个额外的类型是一个扩展类，根据源代码中的`using`指令（编译后消失）隐式选择。当通过接口调用成员时，可以通过隐式或显式转换来传递这个额外类型。（对于显式实现来说，实际上无法在不进行接口转换的情况下调用成员。）当调用隐藏基类成员时，必须通过转换或`base`关键字来指定额外的类型——并且该额外类型在运行时丢失。

