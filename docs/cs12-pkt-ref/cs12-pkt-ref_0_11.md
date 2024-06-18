# 第十一章：语句

函数由按照它们出现的文本顺序依次执行的语句组成。*语句块*是出现在大括号之间的一系列语句（`{}`标记）。

## 声明语句

变量声明引入一个新变量，并可选地使用表达式进行初始化。您可以在逗号分隔的列表中声明多个相同类型的变量。例如：

```cs
bool rich = true, famous = false;
```

常量声明类似于变量声明，但在声明后不能更改，并且初始化必须与声明同时进行（更多内容请参阅“常量”）：

```cs
const double c = 2.99792458E08;
```

### 局部变量作用域

局部变量或局部常量的作用域延伸到当前块。您不能在当前块或任何嵌套块中声明另一个同名的局部变量。

## 表达式语句

表达式语句是有效的语句也是表达式。实际上，这意味着“做”某事的表达式；换句话说：

+   分配或修改变量

+   实例化对象

+   调用方法

不执行这些操作的表达式不是有效的语句：

```cs
string s = "foo";
s.Length;          // Illegal statement: does nothing!
```

当您调用返回值的构造函数或方法时，您不必使用该结果。但是，除非构造函数或方法更改状态，否则该语句是无用的：

```cs
new StringBuilder();     // Legal, but useless
x.Equals (y);            // Legal, but useless
```

## 选择语句

选择语句有条件地控制程序执行流程。

### `if`语句

`if`语句在`bool`表达式为`true`时执行一个语句。例如：

```cs
if (5 < 2 * 3)
  Console.WriteLine ("true");       // true
```

语句可以是一个代码块：

```cs
if (5 < 2 * 3)
{
  Console.WriteLine ("true");       // true
  Console.WriteLine ("...")
}
```

### else 子句

`if` 语句可以选择地包含 `else` 子句：

```cs
if (2 + 2 == 5)
  Console.WriteLine ("Does not compute");
else
  Console.WriteLine ("False");        // False
```

在 `else` 子句中，你可以嵌套另一个 `if` 语句：

```cs
if (2 + 2 == 5)
  Console.WriteLine ("Does not compute");
else
 if (2 + 2 == 4)
 Console.WriteLine ("Computes");    // Computes
```

### 通过移动大括号来改变执行流程

`else` 子句总是应用于语句块中紧接着的 `if` 语句。例如：

```cs
if (true)
  if (false)
    Console.WriteLine();
  else
    Console.WriteLine ("executes");
```

这在语义上与以下内容相同：

```cs
if (true)
{
  if (false)
    Console.WriteLine();
  else
    Console.WriteLine ("executes");
}
```

通过移动大括号，你可以改变执行流程：

```cs
if (true)
{
  if (false)
    Console.WriteLine();
}
else
  Console.WriteLine ("does not execute");
```

C# 没有“elseif”关键字；然而，以下模式可以达到相同的效果：

```cs
if (age >= 35)
  Console.WriteLine ("You can be president!");
else if (age >= 21)
  Console.WriteLine ("You can drink!");
else if (age >= 18)
  Console.WriteLine ("You can vote!");
else
  Console.WriteLine ("You can wait!");
```

### `switch` 语句

`switch` 语句允许你根据变量可能具有的一组可能值来分支程序执行。与多个 `if` 语句相比，`switch` 语句可以生成更干净的代码，因为 `switch` 语句只需要评估一次表达式。例如：

```cs
static void ShowCard (int cardNumber)
{
  switch (cardNumber)
  {
    case 13:
      Console.WriteLine ("King");
      break;
    case 12:
      Console.WriteLine ("Queen");
      break;
    case 11:
      Console.WriteLine ("Jack");
      break;
    default:    // Any other cardNumber
      Console.WriteLine (cardNumber);
      break;
  }
}
```

每个 `case` 表达式中的值必须是常量，这限制了它们允许的类型为内置的数值类型以及 `bool`、`char`、`string` 和 `enum` 类型。在每个 `case` 子句的末尾，你必须明确指定执行流向下一个跳转语句。以下是选项：

+   `break`（跳转到 `switch` 语句的末尾）

+   `goto case *x*`（跳转到另一个 `case` 子句）

+   `goto default`（跳转到 `default` 子句）

+   任何其他跳转语句，包括 `return`、`throw`、`continue` 或 `goto *label*`

当多个值应执行相同代码时，可以顺序列出公共的 `case`：

```cs
switch (cardNumber)
{
 case 13:
 case 12:
 case 11:
    Console.WriteLine ("Face card");
    break;
  default:
    Console.WriteLine ("Plain card");
    break;
}
```

`switch` 语句的这一特性在产生比多个 `if`-`else` 语句更清晰的代码方面可以起到关键作用。

### 在类型上进行 `switch`

从 C# 7 开始，你可以根据 *类型* 进行 `switch`：

```cs
static void TellMeTheType (object x)
{
  switch (x)
  {
    case int i:
      Console.WriteLine ("It's an int!");
      break;
    case string s:
      Console.WriteLine (s.Length);      // We can use s
      break;
    case bool b when b == true:   // Fires when b is true
      Console.WriteLine ("True");
      break;
    case null:    // You can also switch on null
      Console.WriteLine ("null");
      break;
  }
}
```

（`object` 类型允许变量具有任何类型 — 见 “继承” 和 “object 类型”。）

每个 *case* 子句指定了要匹配的类型以及要在匹配成功时分配的变量。与常量不同，你可以使用任何类型。可选的 `when` 子句指定了必须满足的条件以使 `case` 匹配。

当你在类型上进行 `switch` 时，`case` 子句的顺序是相关的（与在常量上进行 `switch` 不同）。一个例外是 `default` 子句，无论它出现在何处，都是最后执行的。

你可以堆叠多个 `case` 子句。以下代码中的 `Console.WriteLine` 将对大于 1,000 的任何浮点类型执行：

```cs
  switch (x)
  {
 case float f when f > 1000:
 case double d when d > 1000:
 case decimal m when m > 1000:
      Console.WriteLine ("f, d and m are out of scope");
      break;
```

在这个例子中，编译器允许我们在 `when` 子句中消耗变量 `f`、`d` 和 `m`，仅在调用 `Console.WriteLine` 时，无法确定将为这三个变量中的哪一个分配值，因此编译器使它们全部超出范围。

### `switch` 表达式

从 C# 8 开始，你也可以在 *表达式* 的上下文中使用 `switch`。假设 `cardNumber` 的类型是 `int`，下面展示了它的使用方法：

```cs
string cardName = cardNumber switch
{
  13 => "King",
  12 => "Queen",
  11 => "Jack",
  _ => "Pip card"   // equivalent to 'default'
};
```

注意，`switch`关键字出现在变量名之后，并且 case 子句是表达式（以逗号结尾），而不是语句。你也可以对多个值（*tuples*）进行 switch：

```cs
int cardNumber = 12; string suite = "spades";
string cardName = (cardNumber, suite) switch
{
  (13, "spades") => "King of spades",
  (13, "clubs") => "King of clubs",
  ...
};
```

## 迭代语句

C#允许使用`while`、`do-while`、`for`和`foreach`语句重复执行一系列语句。

### `while`和`do-while`循环

`while`循环在`bool`表达式为真时重复执行一段代码。表达式在执行循环体之前进行测试。例如，以下输出`012`：

```cs
int i = 0;
while (i < 3)
{                         // Braces here are optional
  Console.Write (i++);
}
```

`do`-`while`循环与`while`循环的功能不同之处仅在于它们在执行语句块后测试表达式（确保语句块至少执行一次）。以下是使用`do`-`while`循环重写的前述示例：

```cs
int i = 0;
do
{
  Console.WriteLine (i++);
}
while (i < 3);
```

### `for`循环

`for`循环类似于`while`循环，但具有用于循环变量的初始化和迭代的特殊子句。`for`循环包含以下三个子句：

```cs
for (*init-clause*; *condition-clause*; *iteration-clause*)
  *statement-or-statement-block*
```

*init-clause*在循环开始之前执行，通常初始化一个或多个*iteration*变量。

*condition-clause*是一个`bool`表达式，在每次循环迭代之前进行测试。当条件为真时，执行循环体。

*iteration-clause*在每次执行循环体之后执行。通常用于更新迭代变量。

例如，以下打印数字 0 至 2：

```cs
for (int i = 0; i < 3; i++)
  Console.WriteLine (i);
```

以下打印前 10 个斐波那契数（每个数字是前两个数字的和）：

```cs
for (int i = 0, prevFib = 1, curFib = 1; i < 10; i++)
{
  Console.WriteLine (prevFib);
  int newFib = prevFib + curFib;
  prevFib = curFib; curFib = newFib;
}
```

`for`语句的三个部分都可以省略。你可以实现无限循环，例如以下（虽然可以使用`while(true)`替代）：

```cs
for (;;) Console.WriteLine ("interrupt me");
```

### `foreach`循环

`foreach`语句在可枚举对象中迭代每个元素。大多数表示集合或列表的.NET 类型都是可枚举的。例如，数组和字符串都是可枚举的。以下是枚举字符串中字符的示例，从第一个字符到最后一个字符：

```cs
foreach (char c in "beer")
  Console.Write (c + " ");   // b e e r
```

我们在“Enumeration and Iterators”中定义可枚举对象。

## 跳转语句

C#跳转语句包括`break`、`continue`、`goto`、`return`和`throw`。我们在“try Statements and Exceptions”中讨论`throw`关键字。

### break 语句

`break`语句结束迭代或`switch`语句的执行：

```cs
int x = 0;
while (true)
{
  if (x++ > 5) break;      // break from the loop
}
// execution continues here after break
...
```

### continue 语句

`continue`语句放弃循环中剩余的语句，并提前开始下一次迭代。以下循环*跳过*偶数：

```cs
for (int i = 0; i < 10; i++)
{
  if ((i % 2) == 0) continue;
  Console.Write (i + " ");      // 1 3 5 7 9
}
```

### goto 语句

`goto`语句将执行转移至语句块内的标签（用冒号后缀表示）。以下迭代 1 至 5 的数字，模拟`for`循环：

```cs
int i = 1;
startLoop:
if (i <= 5)
{
  Console.Write (i + " ");   // 1 2 3 4 5
  i++;
 goto startLoop;
}
```

### return 语句

`return`语句退出方法，并且如果方法是非 void 类型，则必须返回方法返回类型的表达式：

```cs
decimal AsPercentage (decimal d)
{
  decimal p = d * 100m;
  return p;     // Return to calling method with value
}
```

`return`语句可以出现在方法中的任何位置（除了`finally`块），并且可以多次使用。

