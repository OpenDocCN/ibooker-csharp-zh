# 第十六章：结构体

*结构体*类似于类，但具有以下关键区别：

+   结构体是一个值类型，而类是一个引用类型。

+   结构体不支持继承（除了隐式从`object`或更确切地说是`System.ValueType`派生）。

一个结构体可以拥有类似于类的所有成员，但不能有析构函数、虚拟或受保护成员。

###### 警告

在 C# 10 之前，结构体进一步禁止定义字段初始化程序和无参数构造函数。尽管现在这种禁止已经放宽，主要是为了记录结构体的利益（参见“记录”），但在定义这些构造函数之前值得仔细考虑，因为它们可能导致混乱的行为，我们将在“结构体构造语义”中描述。

当需要值类型语义时，结构体是合适的选择。良好的示例是数值类型，其中赋值复制值而不是引用更为自然。因为结构体是值类型，每个实例不需要在堆上实例化对象（以及随后的收集）；在创建许多类型实例时，这可以节省有用的开销。

与任何值类型一样，结构体可以间接地最终出现在堆上，通过装箱或者作为类中字段的一部分。如果我们在以下示例中实例化`SomeClass`，字段`Y`将引用堆上的一个结构体：

```cs
struct SomeStruct { public int X;        }
class SomeClass   { public SomeStruct Y; }
```

同样地，如果您声明了一个`SomeStruct`的数组，该实例将位于堆上（因为数组是引用类型），尽管整个数组只需要单个内存分配。

从 C# 7.2 开始，您可以将`ref`修饰符应用于结构体，以确保它只能以将其放置在堆栈上的方式使用。这不仅能够进一步优化编译器，还允许使用[`Span<T>`类型](https://oreil.ly/kCark)。

## 结构体构造语义

###### 注意

在 C# 11 之前，结构体中的每个字段都必须在构造函数（或字段初始化程序）中显式赋值。现在这个限制已经放宽。

除了您定义的任何构造函数外，结构体始终具有一个隐式的无参数构造函数，该构造函数对其字段执行位清零操作（将它们设置为它们的默认值）：

```cs
Point p = new Point();     // p.x and p.y will be 0
struct Point { int x, y; }
```

即使您定义了自己的无参数构造函数，隐式的无参数构造函数仍然存在，并且可以通过`default`关键字访问：

```cs
Point p1 = new Point();    // p1.x and p1.y will be 1
Point p2 = default;        // p2.x and p2.y will be 0

struct Point
{
  int x = 1; int y;
  public Point() => y = 1;
}
```

在本例中，我们通过字段初始化器将`x`初始化为 1，通过无参数构造函数将`y`初始化为 1。然而，使用`default`关键字，我们仍然能够创建一个绕过这两个初始化的`Point`。默认构造函数也可以通过其他方式访问，正如下面的例子所示：

```cs
var points = new Point[10];   // Each point will be (0,0)
var test = new Test();        // test.p will be (0,0)
class Test { Point p; }
```

对于结构体，一个好的策略是设计它们的`default`值是一个有效状态，从而使初始化变得多余。

## 只读结构体和函数

您可以将`readonly`修饰符应用于结构体，以强制所有字段都是`readonly`；这有助于声明意图，并允许编译器更多优化自由度：

```cs
readonly struct Point
{
  public readonly int X, Y;   // X and Y must be readonly
}
```

如果您需要在更精细的层次上应用`readonly`，可以将`readonly`修饰符（从 C# 8 开始）应用于结构的*函数*。这样可以确保如果函数尝试修改任何字段，将生成编译时错误：

```cs
struct Point
{
  public int X, Y;
  public readonly void ResetX() => X = 0;  // Error!
}
```

如果一个`readonly`函数调用一个非`readonly`函数，编译器会生成警告（并防御性地复制结构体，以避免突变的可能性）。

