# 第四章：数值类型

C# 具有以下预定义的数值类型：

| C# 类型 | 系统类型 | 后缀 | 大小 | 范围 |
| --- | --- | --- | --- | --- |
| **整数—有符号** |  |
| `sbyte` | `SByte` |  | 8 位 | –2⁷ 到 2⁷–1 |
| `short` | `Int16` |  | 16 位 | –2¹⁵ 到 2¹⁵–1 |
| `int` | `Int32` |  | 32 位 | –2³¹ 到 2³¹–1 |
| `long` | `Int64` | `L` | 64 位 | –2⁶³ 到 2⁶³–1 |
| `nint` | `IntPtr` |  | 32/64 位 |  |
| **整数—无符号** |  |
| `byte` | `Byte` |  | 8 位 | 0 到 2⁸–1 |
| `ushort` | `UInt16` |  | 16 位 | 0 到 2¹⁶–1 |
| `uint` | `UInt32` | `U` | 32 位 | 0 到 2³²–1 |
| `ulong` | `UInt64` | `UL` | 64 位 | 0 到 2⁶⁴–1 |
| `nuint` | `UIntPtr` |  | 32/64 位 |  |
| **实数** |  |
| `float` | `Single` | `F` | 32 位 | ±（~10^(–45) 到 10³⁸） |
| `double` | `Double` | `D` | 64 位 | ±（~10^(–324) 到 10³⁰⁸） |
| `decimal` | `Decimal` | `M` | 128 位 | ±（~10^(–28) 到 10²⁸） |

在*整型*类型中，`int`和`long`是一流公民，并且在 C#和运行时中受到青睐。其他整型类型通常用于互操作性或在空间效率至关重要时使用。`nint`和`nuint`本地大小的整型被调整为与运行时进程的地址空间匹配（32 位或 64 位）。当处理指针时，这些类型可能很有用——我们在《C# 12 权威指南》（O’Reilly）的第四章中描述了它们的微妙之处。

在*实数*类型中，`float`和`double`被称为*浮点类型*，通常用于科学和图形计算。`decimal`类型通常用于需要基于十进制的精确算术和高精度的财务计算。 （技术上，`decimal`也是一种浮点类型，尽管通常不这样称呼。）

## 数值文字

*整型文字*可以使用十进制、十六进制或二进制表示法；十六进制以`0x`前缀表示（例如，`0x7f`等同于`127`），二进制以`0b`前缀表示。*实数文字*可以使用十进制或指数表示法，如`1E06`。数字文字中可以插入下划线以提高可读性（例如，`1_000_000`）。

### 数值文字类型推断

默认情况下，编译器*推断*数值文字为`double`或整型：

+   如果文字包含小数点或指数符号（`E`），它是`double`。

+   否则，文字的类型是列表中可以容纳文字值的第一个类型：`int`、`uint`、`long`和`ulong`。

例如：

```cs
Console.Write (       1.0.GetType());  // Double *(double)*
Console.Write (      1E06.GetType());  // Double *(double)*
Console.Write (         1.GetType());  // Int32  *(int)*
Console.Write (0xF0000000.GetType());  // UInt32 *(uint)*
Console.Write (0x100000000.GetType()); // Int64  *(long)*
```

### 数值后缀

在上表中列出的*数值后缀*明确定义了文字的类型：

```cs
decimal d = 3.5M;   // M = decimal (case-insensitive)
```

`U`和`L`后缀很少必要，因为`uint`、`long`和`ulong`类型几乎总是可以从`int`推断或隐式转换：

```cs
long i = 5;     // Implicit conversion from int to long
```

技术上，带有`D`后缀的文字是多余的，因为带有小数点的所有文字都被推断为`double`（你可以随时给数字文字加上小数点）。`F`和`M`后缀是最有用的，在指定分数`float`或`decimal`文字时是必需的。没有后缀的话，以下代码不会编译通过，因为`4.5`将被推断为`double`类型，而`double`类型无法隐式转换为`float`或`decimal`：

```cs
float f = 4.5F;       // Won't compile without suffix
decimal d = -1.23M;   // Won't compile without suffix
```

## 数值转换

### 整型到整型的转换

当目标类型能够表示源类型的每一个可能值时，整数转换是*隐式*的。否则，需要*显式*转换。例如：

```cs
int x = 12345;       // int is a 32-bit integral type
long y = x;          // Implicit conversion to 64-bit int
short z = (short)x;  // Explicit conversion to 16-bit int
```

### 实数到实数的转换

`float` 可以隐式转换为 `double`，因为 `double` 能够表示每一个可能的 `float` 值。反向转换必须显式进行。

在 `decimal` 和其他实数类型之间的转换必须是显式的。

### 实数到整数的转换

从整数类型到实数类型的转换是隐式的，而反向转换必须是显式的。将浮点数转换为整数类型会截断任何小数部分；要执行四舍五入转换，请使用静态的 `System.Convert` 类。

一个注意事项是，将大整数类型隐式转换为浮点数类型会保留*大小*，但偶尔可能会失去*精度*：

```cs
int i1 = 100000001;
float f = i1;      // Magnitude preserved, precision lost
int i2 = (int)f;   // 100000000
```

## 算术运算符

算术运算符（`+`、`-`、`*`、`/`、`%`）适用于除了 8 位和 16 位整数类型之外的所有数值类型。`%` 运算符在除法后求余数。

## 自增和自减运算符

自增和自减运算符（`++`、`--`）会使数值类型增加或减少 1。操作符可以在变量前或后出现，具体取决于您希望在表达式评估*之前*还是*之后*更新变量。例如：

```cs
int x = 0;
Console.WriteLine (x++);   // Outputs 0; x is now 1
Console.WriteLine (++x);   // Outputs 2; x is now 2
Console.WriteLine (--x);   // Outputs 1; x is now 1
```

## 特定的整数操作

### 除法

对整数类型的除法操作总是会消除余数（朝零舍入）。除以一个值为零的变量会生成运行时错误（`DivideByZeroException`）。除以*字面值*或*常量* 0 会生成编译时错误。

### 溢出

在运行时，整数类型的算术操作可能会溢出。默认情况下，这种情况会静默发生 —— 不会抛出异常，并且结果会表现为环绕行为，就好像在较大的整数类型上进行计算并且丢弃了额外的有效位。例如，将最小可能的 `int` 值递减会导致最大可能的 `int` 值：

```cs
int a = int.MinValue; a--;
Console.WriteLine (a == int.MaxValue); // True
```

### checked 和 unchecked 操作符

当整型表达式或语句超出其类型的算术限制时，`checked` 操作符会指示运行时生成 `OverflowException` 而不是静默溢出。`checked` 操作符影响具有 `++`、`--`、（一元）`-`、`+`、`-`、`*`、`/` 和整型类型之间的显式转换运算符的表达式。溢出检查会带来一定的性能成本。

您可以在表达式或语句块周围使用 `checked`。例如：

```cs
int a = 1000000, b = 1000000;

int c = checked (a * b);   // Checks just the expression

checked                    // Checks all expressions
{                          // in statement block
   c = a * b;
   ...
}
```

您可以通过使用 `/checked+` 命令行开关（在 Visual Studio 中，转到高级构建设置）使算术溢出检查成为程序中所有表达式的默认设置。如果您需要仅针对特定表达式或语句禁用溢出检查，则可以使用 `unchecked` 操作符。

### 位运算符

C# 支持以下位运算符：

| 运算符 | 含义 | 示例表达式 | 结果 |
| --- | --- | --- | --- |
| `~` | 补码 | `~0xfU` | `0xfffffff0U` |
| `&` | 与 | `0xf0 & 0x33` | `0x30` |
| ` | ` | 或 | `0xf0 | 0x33` | `0xf3` |
| `^` | 异或 | `0xff00 ^ 0x0ff0` | `0xf0f0` |
| `<<` | 左移 | `0x20 << 2` | `0x80` |
| `>>` | 右移 | `0x20 >> 1` | `0x10` |

从 C# 11 开始，还有一个无符号右移操作符 (`>>>`)。而右移操作符 (`>>`) 在操作有符号整数时会复制高位位。

## 8 和 16 位整数类型

8 和 16 位整数类型包括 `byte`、`sbyte`、`short` 和 `ushort`。这些类型缺乏自己的算术运算符，因此 C# 根据需要将它们隐式转换为较大的类型。当试图将结果重新赋给小整数类型时可能会导致编译错误：

```cs
short x = 1, y = 1;
short z = x + y;          // Compile-time error
```

在这种情况下，`x` 和 `y` 隐式转换为 `int` 以便执行加法运算。这意味着结果也是一个 `int`，无法隐式转换回 `short`（因为可能会造成数据丢失）。为了使其编译通过，必须添加显式转换：

```cs
short z = (short) (x + y);   // OK
```

## 特殊浮点和双精度值

与整数类型不同，浮点类型有些特定操作会对特殊值进行处理。这些特殊值包括 NaN（非数字）、+∞、−∞ 和 −0\. `float` 和 `double` 类有用于 NaN、+∞ 和 −∞ 的常量（以及包括 `MaxValue`、`MinValue` 和 `Epsilon` 在内的其他值）。例如：

```cs
Console.Write (double.NegativeInfinity);   // -Infinity
```

将非零数除以零会得到一个无限值：

```cs
Console.WriteLine ( 1.0 /  0.0);   //  Infinity
Console.WriteLine (−1.0 /  0.0);   // -Infinity
Console.WriteLine ( 1.0 / −0.0);   // -Infinity
Console.WriteLine (−1.0 / −0.0);   //  Infinity
```

将零除以零，或从无穷大中减去无穷大，结果是 NaN：

```cs
Console.Write ( 0.0 / 0.0);                 //  NaN
Console.Write ((1.0 / 0.0) − (1.0 / 0.0));  //  NaN
```

当使用 `==` 时，NaN 值永远不等于另一个值，甚至是另一个 NaN 值。要测试一个值是否为 NaN，必须使用 `float.IsNaN` 或 `double.IsNaN` 方法：

```cs
Console.WriteLine (0.0 / 0.0 == double.NaN);    // False
Console.WriteLine (double.IsNaN (0.0 / 0.0));   // True
```

然而，当使用 `object.Equals` 时，两个 NaN 值是相等的：

```cs
bool isTrue = object.Equals (0.0/0.0, double.NaN);
```

## double 和 decimal 的比较

`double` 适用于科学计算（如计算空间坐标）。`decimal` 适用于金融计算和制造值，而不是实际测量结果。以下是它们之间的主要区别总结：

| 特性 | double | decimal |
| --- | --- | --- |
| 内部表示 | 二进制 | 十进制 |
| 精度 | 15–16 有效数字 | 28–29 有效数字 |
| 范围 | ±(~10^(−324) 到 ~10³⁰⁸) | ±(~10^(−28) 到 ~10²⁸) |
| 特殊值 | +0、−0、+∞、−∞ 和 NaN | 无 |
| 速度 | 本地处理器 | 非本地处理器（比 `double` 慢约 10 倍） |

## 实数舍入误差

`float` 和 `double` 在内部以二进制表示数字。因此，大多数带有小数部分的文字常量（以十进制为基础）不能精确表示，这使它们在财务计算中不合适。相比之下，`decimal` 使用十进制工作，因此可以精确表示如 0.1 的分数，其十进制表示不会循环。

