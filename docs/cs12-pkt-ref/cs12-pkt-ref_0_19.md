# 第十九章：枚举

*枚举*是一种特殊的值类型，允许您指定一组命名的数字常量。例如：

```cs
public enum BorderSide { Left, Right, Top, Bottom }
```

我们可以如下使用这种枚举类型：

```cs
BorderSide topSide = BorderSide.Top;
bool isTop = (topSide == BorderSide.Top);   // true
```

每个枚举成员都有一个基础的整数类型值。默认情况下，基础值的类型为`int`，枚举成员被分配为常量`0`、`1`、`2`...（按其声明顺序）。您可以指定另一种整数类型，如下所示：

```cs
public enum BorderSide : byte { Left,Right,Top,Bottom }
```

您还可以为每个成员指定显式整数值：

```cs
public enum BorderSide : byte
 { Left=1, Right=2, Top=10, Bottom=11 }
```

编译器还允许您显式分配*一些*枚举成员。未分配的枚举成员从最后一个显式值递增。前面的示例等效于：

```cs
public enum BorderSide : byte
 { Left=1, Right, Top=10, Bottom }
```

## 枚举转换

您可以使用显式转换将`enum`实例转换为其基础整数值，反之亦然：

```cs
int i = (int) BorderSide.Left;
BorderSide side = (BorderSide) i;
bool leftOrRight = (int) side <= 2;
```

您还可以将一种枚举类型明确转换为另一种；翻译将使用成员的基础整数值。

数字文字`0`在特殊处理中不需要显式转换：

```cs
BorderSide b = 0;    // No cast required
if (b == 0) ...
```

在此特定示例中，`BorderSide`没有具有整数值`0`的成员。这不会生成错误：枚举的限制是，编译器和 CLR 不会阻止分配超出成员范围的整数值：

```cs
BorderSide b = (BorderSide) 12345;
Console.WriteLine (b);              // 12345
```

## 标志枚举

您可以组合枚举成员。为了避免歧义，可组合枚举的成员需要显式分配的值，通常是二的幂次方。例如：

```cs
[Flags]
public enum BorderSides
 { None=0, Left=1, Right=2, Top=4, Bottom=8 }
```

根据惯例，可组合的枚举类型应该使用复数而不是单数名称。要使用组合枚举值，您可以使用按位操作符，例如`|`和`&`。这些操作符适用于基础整数值：

```cs
BorderSides leftRight =
  BorderSides.Left | BorderSides.Right;

if ((leftRight & BorderSides.Left) != 0)
  Console.WriteLine ("Includes Left");   // Includes Left

string formatted = leftRight.ToString(); // "Left, Right"

BorderSides s = BorderSides.Left;
s |= BorderSides.Right;
Console.WriteLine (s == leftRight);      // True
```

对于可组合的枚举类型，应将`Flags`属性应用于其上；如果未执行此操作，则在`enum`实例上调用`ToString`会输出数字而不是一系列名称。

为方便起见，您可以在枚举声明本身中包含组合成员：

```cs
[Flags] public enum BorderSides
{
  None=0,
  Left=1, Right=2, Top=4, Bottom=8,
 LeftRight = Left | Right, 
 TopBottom = Top  | Bottom,
 All       = LeftRight | TopBottom
}
```

## 枚举操作符

适用于枚举的操作符包括：

```cs
=   ==   !=   <   >   <=   >=   +   -   ^  &  |   ˜
+=  -=   ++   --   sizeof
```

按位、算术和比较操作符返回处理基础整数值的结果。枚举与整数类型之间允许进行加法运算，但不允许两个枚举之间进行加法运算。

