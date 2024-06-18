# 第八章：变量和参数

*变量* 表示具有可修改值的存储位置。变量可以是 *局部变量*、*参数*（*value*、*ref*、*out* 或 *in*）、*字段*（实例或静态）、或 *数组元素*。

## 堆栈与堆

*堆栈* 和 *堆* 是变量所在的地方。它们各自具有非常不同的生存期语义。

### 堆栈

*堆栈* 是用于存储局部变量和参数的内存块。堆栈在逻辑上随着方法或函数的进入和退出而增长和收缩。考虑以下方法（为避免分心，忽略输入参数检查）：

```cs
static int Factorial (int x)
{
  if (x == 0) return 1;
  return x * Factorial (x-1);
}
```

此方法是 *递归* 的，意味着它会调用自身。每次进入方法时，在堆栈上分配一个新的 `int`，每次退出方法时，该 `int` 被释放。

### 堆

*堆* 是对象（即引用类型实例）所在的内存。每当创建一个新对象时，它被分配到堆上，并返回对该对象的引用。在程序执行期间，堆会随着创建新对象而填满。运行时具有垃圾收集器，定期从堆中释放对象，以便程序不会耗尽内存。只要一个对象不被任何活动对象引用，就有资格进行释放。

值类型实例（和对象引用）存在于变量声明的位置。如果实例被声明为类类型的字段或数组元素，则该实例存在于堆上。

###### 注意

在 C# 中不能像在 C++ 中那样显式删除对象。一个未引用的对象最终会被垃圾收集器收集。

堆还存储静态字段和常量。与在堆上分配的对象（可以进行垃圾收集）不同，这些对象一直存在，直到应用程序域被拆除。

## 明确赋值

C# 强制实施明确赋值策略。在实践中，这意味着在 `unsafe` 上下文之外，不可能访问未初始化的内存。明确赋值有三个含义：

+   局部变量必须在读取之前赋值。

+   调用方法时必须提供函数参数（除非标记为可选—参见 “可选参数”）。

+   所有其他变量（例如字段和数组元素）都由运行时自动初始化。

例如，以下代码导致编译时错误：

```cs
int x;                   // x is a local variable
Console.WriteLine (x);   // Compile-time error
```

然而，以下输出 `0`，因为字段隐式分配了默认值（无论是实例还是静态）：

```cs
Console.WriteLine (Test.X);   // 0
class Test { public static int X; }   // Field
```

## 默认值

所有类型实例都有一个默认值。预定义类型的默认值是通过内存按位清零的结果，对于引用类型是 `null`，对于数值和枚举类型是 `0`，对于 `char` 类型是 `'\0'`，对于 `bool` 类型是 `false`。

您可以使用`default`关键字获取任何类型的默认值（这在与泛型一起使用时特别有用，稍后您将看到）。自定义值类型（即`struct`）的默认值与自定义类型定义的每个字段的默认值相同：

```cs
Console.WriteLine (default (decimal));   // 0
decimal d = default;
```

## 参数

一个方法可以有一系列参数。参数定义了必须为该方法提供的参数集。在此示例中，方法`Foo`有一个名为`p`的单一参数，类型为`int`：

```cs
Foo (8);                        // 8 is an argument
static void Foo (int p) {...}   // p is a parameter
```

您可以使用`ref`、`out`和`in`修饰符控制参数的传递方式：

| 参数修改器 | 通过 | 变量必须明确赋值 |
| --- | --- | --- |
| None | 值 | 进入*中* |
| `ref` | 引用 | 进入*中* |
| `in` | 引用（只读） | 进入*中* |
| `out` | 引用 | 进入*出* |

### 通过值传递参数

默认情况下，在 C#中，参数是*按值传递*的，这是最常见的情况。这意味着当参数传递给方法时会创建一个值的副本：

```cs
int x = 8;
Foo (x);                  // Make a copy of x
Console.WriteLine (x);    // x will still be 8

static void Foo (int p)
{
  p = p + 1;                // Increment p by 1
  Console.WriteLine (p);    // Write p to screen
}
```

给`p`赋一个新值不会更改`x`的内容，因为`p`和`x`位于不同的内存位置。

通过值传递引用类型参数会复制*引用*而不是对象。在下面的示例中，`Foo`看到了我们实例化的相同`StringBuilder`对象(`sb`)，但对它有独立的*引用*。换句话说，`sb`和`fooSB`是引用同一`StringBuilder`对象的不同变量：

```cs
StringBuilder sb = new StringBuilder();
Foo (sb);
Console.WriteLine (sb.ToString());    // test

static void Foo (StringBuilder fooSB)
{
  fooSB.Append ("test");
  fooSB = null;
}
```

因为`fooSB`是引用的*副本*，将其设置为`null`不会使`sb`为 null。（但是，如果`fooSB`被声明并用`ref`修饰符调用，则`sb`将会变为 null。）

### `ref`修饰符

要*按引用传递*，C# 提供了`ref`参数修改器。在下面的示例中，`p`和`x`引用相同的内存位置：

```cs
int x = 8;
Foo (ref  x);              // Ask Foo to deal
                           // directly with x
Console.WriteLine (x);     // x is now 9

static void Foo (ref int p)
{
  p = p + 1;               // Increment p by 1
  Console.WriteLine (p);   // Write p to screen
}
```

现在给`p`赋一个新值会改变`x`的内容。请注意，写入和调用方法时都需要`ref`修饰符。这使得正在发生的事情非常清晰。

###### 注意

无论参数类型是引用类型还是值类型，参数都可以按引用或按值传递。

### `out`修饰符

`out`参数类似于`ref`参数，但以下几点不同：

+   不需要在进入函数之前赋值。

+   它必须在函数*退出*之前赋值。

`out`修饰符最常用于从方法中获取多个返回值。

### 输出变量和丢弃

从 C# 7 开始，您可以在调用具有`out`参数的方法时即时声明变量：

```cs
int.TryParse ("123", out int x);
Console.WriteLine (x);
```

这等同于：

```cs
int x;
int.TryParse ("123", out x);
Console.WriteLine (x);
```

在调用具有多个`out`参数的方法时，您可以使用下划线“丢弃”您不感兴趣的任何参数。假设`SomeBigMethod`已定义为具有五个`out`参数，则可以忽略除第三个之外的所有参数，如下所示：

```cs
SomeBigMethod (out _, out _, out int x, out _, out _);
Console.WriteLine (x);
```

### `in`修饰符

从 C# 7.2 开始，你可以在参数前加上`in`修饰符，以防止其在方法内被修改。这允许编译器避免在传递之前复制参数的开销，这在大型自定义值类型的情况下尤为重要（参见“Structs”）。

### params 修饰符

如果将`params`修饰符应用于方法的最后一个参数，该方法将允许接受特定类型的任意数量的参数。参数类型必须声明为（单维）数组。例如：

```cs
int Sum (params int[] ints)
{
  int sum = 0;
  for (int i = 0; i < ints.Length; i++) sum += ints[i];
  return sum;
}
```

你可以这样调用它：

```cs
Console.WriteLine (Sum (1, 2, 3, 4));    // 10
```

如果在`params`位置上没有参数，则创建一个长度为零的数组。

你还可以将`params`参数作为普通数组提供。前述调用在语义上等同于：

```cs
Console.WriteLine (Sum (new int[] { 1, 2, 3, 4 } ));
```

### 可选参数

方法、构造函数和索引器可以声明 *可选参数*。如果在其声明中指定了 *默认值*，则该参数是可选的：

```cs
void Foo (int x = 23) { Console.WriteLine (x); }
```

调用方法时可以省略可选参数：

```cs
Foo();     // 23
```

*默认参数* `23` 实际上 *传递* 给了可选参数 `x` —— 编译器将值 `23` 嵌入到调用方的编译代码中。前述对 `Foo` 的调用在语义上与以下内容相同：

```cs
Foo (23);
```

因为编译器简单地在任何使用处替换可选参数的默认值。

###### 警告

向另一个程序集中的公共方法添加可选参数需要重新编译两个程序集——就像该参数是必需的一样。

可选参数的默认值必须由常量表达式、值类型的无参构造函数或`default`表达式指定。你不能用`ref`或`out`标记可选参数。

强制参数必须在方法声明和方法调用中 *之前* 出现可选参数（例外情况是`params`参数，它们始终出现在最后）。在以下示例中，显式值`1`被传递给`x`，默认值`0`被传递给`y`：

```cs
Foo(1);    // 1, 0

void Foo (int x = 0, int y = 0)
{
  Console.WriteLine (x + ", " + y);
}
```

通过结合可选参数和*命名参数*，你可以进行反向操作（向`x`传递默认值，向`y`传递显式值）。

### 命名参数

你可以通过名称而不是位置标识一个参数。例如：

```cs
Foo (x:1, y:2);  // 1, 2

void Foo (int x, int y)
{
  Console.WriteLine (x + ", " + y);
}
```

命名参数可以以任何顺序出现。对`Foo`的以下调用在语义上是相同的：

```cs
Foo (x:1, y:2);
Foo (y:2, x:1);
```

你可以混合使用命名参数和位置参数，只要命名参数出现在最后：

```cs
Foo (1, y:2);
```

命名参数在与可选参数结合使用时特别有用。例如，考虑以下方法：

```cs
void Bar (int a=0, int b=0, int c=0, int d=0) { ... }
```

你可以这样调用它，只提供`d`的值：

```cs
Bar (d:3);
```

在调用 COM API 时特别有用。

## var —— 隐式类型的局部变量

通常情况下，你会一步声明和初始化一个变量。如果编译器能够从初始化表达式中推断出类型，则可以使用 `var` 替代类型声明。例如：

```cs
var x = "hello";
var y = new System.Text.StringBuilder();
var z = (float)Math.PI;
```

这与以下完全等价：

```cs
string x = "hello";
System.Text.StringBuilder y = 
  new System.Text.StringBuilder();
float z = (float)Math.PI;
```

因为这种直接的等价性，隐式类型变量是静态类型的。例如，以下代码会生成编译时错误：

```cs
var x = 5;
x = "hello";    // Compile-time error; x is of type int
```

在“匿名类型”章节中，我们描述了使用`var`是强制性的情况。

## 目标类型化的新表达式

另一种减少词汇重复的方法是使用*C# 9*中的*目标类型化*`new` *表达式*：

```cs
StringBuilder sb1 = new();
StringBuilder sb2 = new ("Test");
```

这与以下完全等价：

```cs
StringBuilder sb1 = new StringBuilder();
StringBuilder sb2 = new StringBuilder ("Test");
```

原则是，如果编译器能够明确地推断出来，你可以调用`new`而不需要指定类型名称。当变量声明和初始化在代码的不同部分时，目标类型化`new`表达式尤其有用。一个常见的例子是当你想在构造函数中初始化一个字段时：

```cs
class Foo
{
  System.Text.StringBuilder sb;

  public Foo (string initialValue)
  {
    sb = new (initialValue);
  }
}
```

目标类型化`new`表达式在以下场景中也非常有帮助：

```cs
MyMethod (new ("test"));
void MyMethod (System.Text.StringBuilder sb) { ... }
```

