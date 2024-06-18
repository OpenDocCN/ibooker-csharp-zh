# 第三十三章：记录

*记录*（从 C# 9 开始）是一种特殊的类或结构体，设计用于与不可变（只读）数据良好配合。其最有用的特性是允许*非破坏性变异*，即要“修改”不可变对象，你创建一个新对象，并复制数据同时合并修改。

记录也非常有用，用于创建仅组合或保存数据的类型。在简单情况下，它们消除了样板代码，同时遵循*结构相等*语义（如果它们的数据相同，则两个对象相同），这通常是不可变类型所需的。

记录纯粹是 C#的编译时构造。在运行时，CLR 将它们视为类或结构体（由编译器添加了一堆额外的“合成”成员）。

## 定义记录

记录定义类似于类或结构体定义，可以包含相同类型的成员，包括字段、属性、方法等。记录可以实现接口，（基于类的）记录可以子类化其他（基于类的）记录。

默认情况下，记录的基础类型是一个类：

```cs
record Point { }          // Point is a class
```

从 C# 10 开始，记录的基础类型也可以是结构体：

```cs
record struct Point { }   // Point is a struct
```

（`record class`也是合法的，并具有与`record`相同的含义。）

一个简单的记录可能只包含一堆仅初始化的属性，也许还有一个构造函数：

```cs
record Point
{
  public Point (double x, double y) => (X, Y) = (x, y);

  public double X { get; init; }
  public double Y { get; init; }    
}
```

在编译时，C#将记录定义转换为类（或结构体）并执行以下额外步骤：

+   它编写了一个受保护的*复制构造函数*（和一个隐藏的*Clone*方法），以促进非破坏性变异。

+   它重写/重载了与相等性相关的函数，以实现结构相等性。

+   它重写了`ToString()`方法（扩展了记录的公共属性，就像匿名类型一样）。

前述记录声明会展开为类似于这样的内容：

```cs
class Point
{  
  public Point (double x, double y) => (X, Y) = (x, y);

  public double X { get; init; }
  public double Y { get; init; }    

 protected Point (Point original) // “Copy constructor”
 {
 this.X = original.X; this.Y = original.Y
 }

  // This method has a strange compiler-generated name:
 public virtual Point <Clone>$() => new Point (this);

 // Additional code to override Equals, ==, !=,
 // GetHashCode, ToString()...
}
```

### 参数列表

记录定义可以通过使用*参数列表*来简化：

```cs
record Point (double X, double Y)
{
  ...
}
```

参数可以包括`in`和`params`修饰符，但不能包括`out`或`ref`。如果指定了参数列表，则编译器会执行以下额外步骤：

+   它为每个参数编写了一个仅初始化的属性（或者在记录结构体的情况下为可写属性）。

+   它编写了一个*主构造函数*来填充属性。

+   它编写了一个解构器。

这意味着我们可以简单地如下声明我们的`Point`记录：

```cs
record Point (double X, double Y);
```

编译器最终生成（几乎）与我们在前述展开中列出的内容完全一致。一个小的区别是主构造函数中的参数名最终会成为`X`和`Y`，而不是`x`和`y`：

```cs
  public Point (double X, double Y)
  {
    this.X = X; this.Y = Y;
  }
```

###### 注意

另外，由于是*主构造函数*，参数`X`和`Y`会神奇地在记录中的任何字段或属性初始化器中可用。我们稍后在“主构造函数”中讨论这个细微之处。

定义参数列表时的另一个区别是，编译器还会生成一个解构器：

```cs
  public void Deconstruct (out double X, out double Y)
  {
    X = this.X; Y = this.Y;
  }
```

具有参数列表的记录可以使用以下语法进行子类化：

```cs
record Point3D (double X, double Y, double Z) 
  : Point (X, Y);
```

然后，编译器会像这样发出一个主构造函数：

```cs
class Point3D : Point
{
  public double Z { get; init; }

  public Point3D (double X, double Y, double Z)
    : base (X, Y)
    => this.Z = Z;
}
```

###### 注意

当你需要一个简单地将一堆值（在函数式编程中称为*乘积类型*）简单地组合在一起的类时，参数列表提供了一个不错的快捷方式，并且在原型设计中也可能非常有用。但是，当你需要向`init`访问器添加逻辑（例如参数验证）时，它们并不那么有用。

## 非破坏性变异

编译器对所有记录执行的最重要步骤是编写一个*复制构造函数*（以及一个隐藏的*Clone*方法）。这通过`with`关键字实现了非破坏性的变异：

```cs
Point p1 = new Point (3, 3);
Point p2 = p1 with { Y = 4 };
Console.WriteLine (p2);       // Point { X = 3, Y = 4 }

record Point (double X, double Y);
```

在本示例中，`p2`是`p1`的副本，但其`Y`属性设置为 4。当属性更多时，这带来的好处更大。

非破坏性变异分为两个阶段：

1.  首先，*复制构造函数*克隆记录。默认情况下，它复制记录的每个底层字段，创建一个忠实的副本，同时绕过（访问器的）任何逻辑。所有字段都包括在内（公共和私有的，以及支持自动属性的隐藏字段）。

1.  然后，更新*成员初始化器列表*中的每个属性（这次使用`init`访问器）。

编译器将以下内容翻译为：

```cs
Test t2 = t1 with { A = 10, C = 30 };
```

转换为以下等效的功能：

```cs
Test t2 = new Test(t1);  // Clone t1
t2.A = 10;               // Update property A
t2.C = 30;               // Update property C
```

（如果您明确编写代码，由于`A`和`C`是只读属性，则相同的代码不会编译。此外，复制构造函数是*受保护的*；C#通过调用一个写入到名为`<Clone>$`的记录中的公共隐藏方法来解决此问题。）

如果需要，您可以定义自己的*复制构造函数*。C#然后将使用您的定义而不是自己编写一个：

```cs
protected Point (Point original)
{
  this.X = original.X; this.Y = original.Y;
}
```

当子类化另一个记录时，复制构造函数负责仅复制其自己的字段。要复制基本记录的字段，请委托给基类：

```cs
protected Point (Point original) : base (original)
{
  ...
}
```

## 主构造函数

当您使用参数列表定义记录时，编译器会自动生成属性声明，以及*主构造函数*（和解构函数）。这在简单情况下效果很好，在更复杂的情况下，您可以省略参数列表并手动编写属性声明和构造函数。C#还提供了一个中间选项，即在写一些或所有属性声明的同时定义参数列表：

```cs
record Student(int ID, string Surname, string FirstName)
{
 public int ID { get; } = ID;
}
```

在这种情况下，我们“接管”了`ID`属性的定义，将其定义为只读（而不是只读初始化），防止它参与非破坏性变异。如果您从不需要非破坏性地变异特定属性，则将其设置为只读属性使您能够在记录中缓存计算数据，而无需编写刷新机制。

注意，我们需要包含*属性初始化器*（粗体）：

```cs
  public int ID { get; } = ID;
```

当您“接管”属性声明时，您将负责初始化其值；主构造函数不再自动执行此操作。（这与在类或结构上定义主构造函数时的行为完全匹配。）请注意，粗体中的`ID`指的是*主构造函数参数*，而不是`ID`属性。

与类和结构的主构造函数语义一致，主构造函数参数（在本例中为`ID`、`Surname`和`FirstName`）对所有字段和属性初始化器都是自动可见的。

您还可以接管属性定义并使用显式访问器：

```cs
int _id = ID;
public int ID { get => _id; init => _id = value; }
```

再次，粗体中的`ID`指的是主构造函数参数，而不是属性。（没有歧义的原因是从初始化程序访问属性是非法的。）

我们必须使用 `ID` 初始化 `_id` 属性，这使得这种“接管”变得不太有用，因为主构造函数中的任何逻辑（如验证）都将被绕过。

## 记录和相等性比较

正如对结构体、匿名类型和元组一样，记录提供了开箱即用的结构相等性，这意味着如果它们的字段（和自动属性）相等，则两个记录是相等的：

```cs
var p1 = new Point (1, 2);
var p2 = new Point (1, 2);
Console.WriteLine (p1.Equals (p2));   // True

record Point (double X, double Y);
```

*相等运算符* 也适用于记录（如同适用于元组一样）：

```cs
Console.WriteLine (p1 == p2);         // True
```

与类和结构体不同，如果要自定义相等性行为，您不需要（也不能）重写 `object.Equals` 方法。相反，您需要定义一个公共的 `Equals` 方法，具有以下签名：

```cs
record Point (double X, double Y)
{
  public virtual bool Equals (Point other) =>
  other != null && X == other.X && Y == other.Y;
}
```

`Equals` 方法必须是 `virtual`（而不是 `override`），并且它必须是 *强类型* 的，以便接受实际的记录类型（在这种情况下是 `Point`，而不是 `object`）。一旦您正确地设置了签名，编译器将自动插入您的方法。

与任何类型一样，如果您接管了相等比较，您也应该重写 `GetHashCode()`。记录的一个好处是，您不需要重载 `!=` 或 `==`，也不需要实现 `IEquatable<T>`：这一切都由编译器为您完成。我们在《C# 12 简明概述》的第六章“相等比较”中详细讨论了这个主题。

