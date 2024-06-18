# 第十四章：继承

类可以从另一个类 *继承*，以扩展或自定义原始类。从类继承可以重用该类中的功能，而不是从头开始构建。一个类只能从一个类继承，但本身可以被多个类继承，从而形成类层次结构。在这个例子中，我们首先定义了一个名为 `Asset` 的类：

```cs
public class Asset { public string Name; }
```

接下来，我们定义了名为 `Stock` 和 `House` 的类，它们将继承自 `Asset`。`Stock` 和 `House` 获得 `Asset` 所有的东西，以及它们自己定义的任何额外成员：

```cs
public class Stock : Asset   // inherits from Asset
{
  public long SharesOwned;
}

public class House : Asset   // inherits from Asset
{
  public decimal Mortgage;
}
```

下面是如何使用这些类的示例：

```cs
Stock msft = new Stock { Name="MSFT",
                         SharesOwned=1000 };

Console.WriteLine (msft.Name);         // MSFT
Console.WriteLine (msft.SharesOwned);  // 1000

House mansion = new House { Name="Mansion",
                            Mortgage=250000 };

Console.WriteLine (mansion.Name);      // Mansion
Console.WriteLine (mansion.Mortgage);  // 250000
```

*子类* `Stock` 和 `House` 从 *基类* `Asset` 继承了 `Name` 字段。

子类也称为 *派生类*。

## 多态性

引用是 *多态的*。这意味着类型为 *x* 的变量可以引用一个子类 *x* 的对象。例如，考虑以下方法：

```cs
public static void Display (Asset asset)
{
  System.Console.WriteLine (asset.Name);
}
```

这个方法可以显示 `Stock` 和 `House`，因为它们都是 `Asset`。多态性基于子类（`Stock` 和 `House`）具有其基类（`Asset`）的所有特征。然而，反之则不成立。如果将 `Display` 重写为接受 `House`，则不能传递 `Asset`。

## 转换和引用转换

对象引用可以是：

+   隐式地 *向上转换* 到基类引用

+   显式地 *向下转换* 到子类引用

在兼容的引用类型之间进行上转换和下转换执行 *引用转换*：创建一个指向 *相同* 对象的新引用。上转换始终成功；下转换仅在对象适当类型时才成功。

### 上转换

上转换操作从子类引用创建一个基类引用。例如：

```cs
Stock msft = new Stock();    // From previous example
Asset a = msft;              // Upcast
```

上转换后，变量 `a` 仍然引用与变量 `msft` 相同的 `Stock` 对象。被引用的对象本身没有被修改或转换：

```cs
Console.WriteLine (a == msft);        // True
```

虽然 `a` 和 `msft` 引用同一个对象，但 `a` 对该对象的视图更为限制：

```cs
Console.WriteLine (a.Name);         // OK
Console.WriteLine (a.SharesOwned);  // Compile-time error
```

最后一行生成了一个编译时错误，因为变量 `a` 的类型是 `Asset`，即使它引用的是 `Stock` 类型的对象。要访问其 `SharesOwned` 字段，必须将 `Asset` *下转换* 为 `Stock`。

### 下转换

下转换操作从基类引用创建一个子类引用。例如：

```cs
Stock msft = new Stock();
Asset a = msft;                      // Upcast
Stock s = (Stock)a;                  // Downcast
Console.WriteLine (s.SharesOwned);   // <No error>
Console.WriteLine (s == a);          // True
Console.WriteLine (s == msft);       // True
```

与上转换类似，只影响引用而不影响底层对象。下转换需要显式转换，因为它在运行时可能失败：

```cs
House h = new House();
Asset a = h;          // Upcast always succeeds
Stock s = (Stock)a;   // Downcast fails: a is not a Stock
```

如果一个下转换失败，会抛出 `InvalidCastException`。这是一个 *运行时类型检查* 的例子（参见 “静态类型检查与运行时类型检查”）。

### `as` 操作符

`as` 操作符执行一个下转换，如果下转换失败则返回 `null`（而不是抛出异常）：

```cs
Asset a = new Asset();
Stock s = a as Stock;   // s is null; no exception thrown
```

当你要测试结果是否为 `null` 时，这是非常有用的：

```cs
if (s != null) Console.WriteLine (s.SharesOwned);
```

`as`运算符无法执行*自定义转换*（见“运算符重载”），也不能进行数值转换。

### `is`运算符

`is`运算符测试引用转换是否成功——换句话说，对象是否派生自指定的类（或实现接口）。通常用于在向下转换之前进行测试：

```cs
if (a is Stock) Console.Write (((Stock)a).SharesOwned);
```

如果拆箱转换会成功（见“对象类型”），`is`运算符也会返回 true。但它不考虑自定义或数值转换。

自 C# 7 开始，您可以在使用`is`运算符时引入变量：

```cs
if (a is Stock s)
  Console.WriteLine (s.SharesOwned);
```

引入的变量可以被“立即”使用，并且在`is`表达式外保持作用域内：

```cs
if (a is Stock s && s.SharesOwned > 100000)
  Console.WriteLine ("Wealthy");
else
 s = new Stock();   // s is in scope

Console.WriteLine (s.SharesOwned);  // Still in scope
```

`is`运算符与 C#的最新版本中引入的其他模式一起使用。有关详细讨论，请参阅“模式”。

## 虚函数成员

标记为`virtual`的函数可以被子类重写，提供特定的实现。方法、属性、索引器和事件都可以声明为`virtual`：

```cs
public class Asset
{
  public string Name;
  public virtual decimal Liability => 0;
}
```

(`Liability => 0`是`{ get { return 0; } }`的快捷方式。有关此语法的更多详细信息，请参阅“表达式体属性”。）子类通过应用`override`修饰符来重写虚方法：

```cs
public class House : Asset
{
  public decimal Mortgage;

  public override decimal Liability => Mortgage;
}
```

默认情况下，`Asset`的`Liability`为`0`。`Stock`不需要专门化此行为。但是，`House`专门化了`Liability`属性，以返回`Mortgage`的值：

```cs
House mansion = new House { Name="Mansion",
                            Mortgage=250000 };
Asset a = mansion;
Console.WriteLine (mansion.Liability);  // 250000
Console.WriteLine (a.Liability);        // 250000
```

虚方法和重写方法的签名、返回类型和可访问性必须相同。重写方法可以通过`base`关键字调用其基类实现（见“base 关键字”）。

### 协变返回

自 C# 9 开始，你可以重写一个方法（或属性`get`访问器），使其返回*更具体*（子类化）的类型。例如，你可以在`Asset`类中编写一个返回`Asset`的`Clone`方法，并在`House`类中重写该方法，使其返回`House`。

这是允许的，因为它不违反`Clone`必须返回`Asset`的约定：它返回一个`House`，而*House*是一个`Asset`（甚至更多）。

## 抽象类和抽象成员

声明为*abstract*的类永远不能被实例化。相反，只能实例化其具体的*子类*。

抽象类可以定义*抽象成员*。抽象成员类似于虚成员，但它们不提供默认实现。除非子类也声明为抽象类，否则必须由子类提供实现：

```cs
public abstract class Asset
{
  // Note empty implementation
  public abstract decimal NetValue { get; }
}
```

子类可以像重写虚方法一样重写抽象成员。

## 隐藏继承成员

基类和子类可以定义相同的成员。例如：

```cs
public class A      { public int Counter = 1; }
public class B : A  { public int Counter = 2; }
```

类`B`中的`Counter`字段被称为*隐藏*类`A`中的`Counter`字段。通常情况下，这种情况是偶然发生的，当在子类型中添加相同成员之后添加基类型时。因此，编译器生成警告，然后按以下方式解决歧义：

+   对`A`的引用（在编译时）绑定到`A.Counter`。

+   对`B`的引用（在编译时）绑定到`B.Counter`。

有时，您想要故意隐藏一个成员，在这种情况下，您可以在子类的成员上应用`new`修饰符。`new`修饰符只是抑制了否则会产生的编译器警告：

```cs
public class A     { public     int Counter = 1; }
public class B : A { public new int Counter = 2; }
```

`new`修饰符向编译器和其他程序员表明，重复的成员不是偶然事件。

## 封闭函数和类

重写的函数成员可以使用`sealed`关键字封闭其实现，防止其被后续的子类重写。在我们之前的虚函数成员示例中，我们可以封闭`House`对`Liability`的实现，从而防止从`House`派生的类重写`Liability`，如下所示：

```cs
public sealed override decimal Liability { get { ... } }
```

您还可以将`sealed`修饰符应用于类本身，以防止子类化。

## `base`关键字

`base`关键字类似于`this`关键字。它有两个基本目的：从子类访问重写的函数成员和调用基类构造函数（请参见下一节）。

在此示例中，`House`使用`base`关键字访问`Asset`的`Liability`实现：

```cs
public class House : Asset
{
  ...
  public override decimal Liability 
    => base.Liability + Mortgage;
}
```

使用`base`关键字，我们非虚拟地访问`Asset`的`Liability`属性。这意味着无论实例的实际运行时类型如何，我们始终访问`Asset`的此属性版本。

如果`Liability`是*隐藏*而不是*重写*，同样的方法也适用。（在调用函数之前，您可以通过将其强制转换为基类来访问隐藏成员。）

## 构造函数和继承

子类必须声明其自己的构造函数。例如，假设我们定义`Baseclass`和`Subclass`如下：

```cs
public class Baseclass
{
  public int X;
  public Baseclass () { }
  public Baseclass (int x) => X = x;
}
public class Subclass : Baseclass { }
```

随后的内容是非法的：

```cs
Subclass s = new Subclass (123);
```

`Subclass`必须“重新定义”它想要公开的任何构造函数。在这样做时，它可以使用`base`关键字调用基类的任何构造函数：

```cs
public class Subclass : Baseclass
{
  public Subclass (int x) : base (x) { ... }
}
```

`base`关键字工作方式类似于`this`关键字，不同之处在于它调用基类中的构造函数。基类构造函数总是首先执行；这确保了*基础*初始化发生在*专门化*初始化之前。

如果子类中的构造函数省略了`base`关键字，则会隐式调用基类型的*无参数*构造函数（如果基类没有可访问的无参数构造函数，则编译器会生成错误）。

### 必需成员（C# 11）

如果在大型类层次结构中，子类需要调用基类的构造函数可能会变得繁琐，特别是当有许多参数和多个构造函数时。有时，最好的解决方案是完全避免构造函数，而是完全依赖对象初始化器在构造过程中设置字段或属性。为了帮助解决这个问题，您可以将字段或属性标记为`required`（来自 C# 11）：

```cs
public class Asset
{
  public required string Name;
}
```

必需成员在构造时必须通过对象初始化器填充：

```cs
Asset a1 = new Asset { Name="House" };  // OK
Asset a2 = new Asset();                 // Error
```

如果您希望同时编写一个构造函数，您可以为该构造函数应用`[SetsRequiredMembers]`属性以绕过该构造函数的必需成员限制：

```cs
public class Asset
{
  public required string Name;

  public Asset() { }

  [System.Diagnostics.CodeAnalysis.SetsRequiredMembers]
  public Asset (string n) => Name = n;
}
```

### 构造函数和字段初始化顺序

当一个对象被实例化时，初始化按照以下顺序进行：

1.  从子类到基类：

    1.  字段被初始化。

    1.  调用基类构造函数调用的参数被评估。

1.  从基类到子类：

    1.  构造函数体执行。

### 具有主构造函数的继承

具有主构造函数的类可以使用以下语法进行子类化：

```cs
class Baseclass (int x) {...}
class Subclass (int x, int y) : Baseclass (x) {...}
```

## 重载和解析

继承对方法重载有着有趣的影响。考虑以下两个重载：

```cs
static void Foo (Asset a) { }
static void Foo (House h) { }
```

当调用一个重载时，最具体的类型优先：

```cs
House h = new House (...);
Foo(h);                      // Calls Foo(House)
```

静态地（在编译时）而不是在运行时决定调用特定的重载。下面的代码调用`Foo(Asset)`，尽管`a`的运行时类型是`House`：

```cs
Asset a = new House (...);
Foo(a);                      // Calls Foo(Asset)
```

###### 注意

如果将`Asset`转换为`dynamic`（参见“动态绑定”），则决定调用哪个重载将推迟到运行时，并基于对象的实际类型进行选择。

