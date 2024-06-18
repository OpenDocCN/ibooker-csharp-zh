# 第四十一章：静态多态

在“静态虚拟/抽象接口成员”中，我们介绍了一个高级功能，即接口可以定义`static virtual`或`static abstract`成员，然后由类和结构体作为静态成员来实现。稍后，在“泛型约束”中，我们展示了将接口约束应用于类型参数使方法可以访问该接口成员的方式。在本节中，我们将演示如何通过这种方式实现*静态多态*，从而实现泛型数学功能。

为了说明这一点，考虑以下接口，它定义了一个静态方法来创建某种类型`T`的随机实例：

```cs
interface ICreateRandom<T>
{
  static abstract T CreateRandom();
}
```

现在假设我们希望在以下记录中实现此接口：

```cs
record Point (int X, int Y);
```

在`System.Random`类的帮助下（其`Next`方法生成随机整数），我们可以实现静态的`CreateRandom`方法如下：

```cs
record Point (int X, int Y) : ICreateRandom<Point>
{
  static Random rnd = new();

  public static Point CreateRandom() => 
    new Point (rnd.Next(), rnd.Next());
}
```

要通过接口调用此方法，我们使用*受限类型参数*。以下方法使用这种方法创建测试数据的数组：

```cs
T[] CreateTestData<T> (int count)
  where T : ICreateRandom<T>
{
  T[] result = new T[count];
  for (int i = 0; i < count; i++)
    result [i] = T.CreateRandom();
  return result;
}
```

此代码行展示了其使用方式：

```cs
Point[] testData = CreateTestData<Point>(50);
```

在`CreateTestData`中对静态`CreateRandom`方法的调用是*多态的*，因为它不仅适用于`Point`，还适用于任何实现`ICreateRandom<T>`的类型。这与*实例*多态不同，因为我们不需要在其上调用`CreateRandom`的`ICreateRandom<T>`的实例；我们在类型本身上调用`CreateRandom`。

## 多态操作符

因为操作符本质上是静态函数（参见“运算符重载”），所以操作符也可以声明为静态虚拟接口成员：

```cs
interface IAddable<T> where T : IAddable<T>
{
   abstract static T operator + (T left, T right);
}
```

###### 注意

在这个接口定义中的*自引用*类型约束是为了满足编译器对运算符重载的规则。回顾一下，当定义运算符函数时，操作数至少有一个必须是声明该运算符函数的类型。在这个例子中，我们的操作数是类型为`T`，而包含类型是`IAddable<T>`，因此我们需要一个自引用类型约束来允许将`T`视为`IAddable<T>`。

这是我们如何实现接口的方法：

```cs
record Point (int X, int Y) : IAddable<Point>
{
  public static Point operator +(Point left, Point right)
    => new Point (left.X + right.X, left.Y + right.Y);
}
```

通过受限类型参数，我们可以编写一个方法，以多态方式调用我们的加法运算符（为了简洁起见，省略了边缘情况处理）：

```cs
T Sum<T> (params T[] values) where T : IAddable<T>
{
  T total = values[0];
  for (int i = 1; i < values.Length; i++)
    total += values[i];
  return total;
}
```

我们对`+`运算符的调用（通过`+=`运算符）是多态的，因为它绑定到`IAddable<T>`，而不是`Point`。因此，我们的`Sum`方法适用于所有实现`IAddable<T>`的类型。

当然，像`IAddable<T>`这样的接口如果在.NET 运行时中定义，并且所有.NET 数值类型都实现了它，那将会更加有用。幸运的是，从.NET 7 开始，`System.Numerics`命名空间包含了（更复杂版本的）`IAddable`，以及许多其他算术接口——大部分都包含在`INumber<TSelf>`中。

## 泛型数学

在 .NET 7 之前，执行算术运算的代码必须硬编码为特定的数字类型，例如`int`或`double`。从 .NET 7 开始，新增了`INumber<TSelf>`接口，统一了各种数字类型的算术操作，允许编写如下泛型方法：

```cs
T Sum<T> (params T[] numbers) where T : INumber<T>
{
  T total = T.Zero;
  foreach (T n in numbers)
    total += n;  // Invokes addition for any numeric type
  return total;
}

int intSum = Sum (3, 5, 7);
double doubleSum = Sum (3.2, 5.3, 7.1);
decimal decimalSum = Sum (3.2m, 5.3m, 7.1m);
```

`INumber<TSelf>` 被 .NET 中所有的实数和整数数字类型（以及`char`）实现。它可以被视为一个总称接口，包括了其他更细粒度的接口，用于每种算术操作（加法、减法、乘法、除法、取模运算、比较等），以及用于解析和格式化的接口。这些接口定义了静态抽象运算符和方法——这些允许在我们的`Sum`方法中使用`+=`运算符（和对`T.Zero`的调用）。

