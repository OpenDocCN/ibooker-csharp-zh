# 第二章：语法

C#语法的灵感来源于 C 和 C++语法。在本节中，我们描述了 C#语法的各个元素，使用以下程序：

```cs
using System;

int x = 12 * 30;
Console.WriteLine (x);
```

## 标识符和关键字

*标识符*是程序员为其类、方法、变量等选择的名称。以下是我们示例程序中标识符的顺序：

```cs
System   x   Console   WriteLine
```

标识符必须是一个完整的单词，基本上由以字母或下划线开头的 Unicode 字符组成。C# 标识符是区分大小写的。按照惯例，参数、局部变量和私有字段应该是*小驼峰命名法*（例如，`myVariable`），而其他所有标识符应该是*帕斯卡命名法*（例如，`MyMethod`）。

*关键字*是对编译器有特殊意义的名称。在我们的示例程序中有两个关键字，`using` 和 `int`。

大多数关键字是*保留*的，这意味着你不能把它们用作标识符。以下是所有 C# 保留关键字的完整列表：

| `abstract` `as`

`base`

`bool`

`break`

`byte`

`case`

`catch`

`char`

`checked`

`class`

`const`

`continue`

`decimal`

`default`

`delegate`

`do`

`double`

`else`

`enum` | `event` `explicit`

`extern`

`false`

`finally`

`fixed`

`float`

`for`

`foreach`

`goto`

`if`

`implicit`

`in`

`int`

`interface`

`internal`

`is`

`lock`

`long`

`namespace` | `new` `null`

`object`

`operator`

`out`

`override`

`params`

`private`

`protected`

`public`

`readonly`

`record`

`ref`

`return`

`sbyte`

`sealed`

`short`

`sizeof`

`stackalloc`

`static` | `string` `struct`

`switch`

`this`

`throw`

`true`

`try`

`typeof`

`uint`

`ulong`

`unchecked`

`unsafe`

`ushort`

`using`

`virtual`

`void`

`volatile`

`while` |

### 避免冲突

如果你确实想使用与保留关键字冲突的标识符，可以通过在其前面加上 `@` 前缀来使用它。例如：

```cs
class class  {...}      // Illegal
class @class {...}      // Legal
```

`@` 符号本身不是标识符的一部分。因此 `@myVariable` 和 `myVariable` 是相同的。

### 上下文关键字

有些关键字是*上下文*关键字，这意味着它们也可以作为标识符使用——无需 `@` 符号。这些上下文关键字如下：

| `add` `alias`

`and`

`ascending`

`async`

`await`

`by`

`descending`

`dynamic`

`equals` | `file` `from`

`get`

`global`

`group`

`init`

`into`

`join`

`let`

`managed` | `nameof` `nint`

`not`

`notnull`

`nuint`

`on`

`or`

`orderby`

`partial`

`remove` | `required` `select`

`set`

`unmanaged`

`value`

`var`

`with`

`when`

`where`

`yield` |

使用上下文关键字，可以在使用它们的上下文中消除歧义。

## 字面量、标点符号和操作符

*字面量*是程序中词法上嵌入的原始数据片段。我们在示例程序中使用的字面量是`12`和`30`。*标点符号*帮助标明程序的结构。例如，分号用于终止语句。语句可以跨多行：

```cs
Console.WriteLine
  (1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10);
```

*操作符*用于转换和组合表达式。C# 中的大多数操作符用符号表示，例如乘法操作符 `*`。以下是我们程序中的操作符：

```cs
=  *  .  ()
```

句点表示某物的成员（或数字字面量的小数点）。在声明或调用方法时使用括号；当方法不接受参数时使用空括号。等号执行*赋值*（双等号 `==` 执行相等比较）。

## 注释

C#提供了两种不同的源代码文档风格：*单行注释*和*多行注释*。单行注释以双斜杠开头，直到行尾。例如：

```cs
int x = 3;   // Comment about assigning 3 to x
```

多行注释以`/*`开头，以`*/`结尾。例如：

```cs
int x = 3;   /* This is a comment that
                spans two lines */
```

注释可以嵌入 XML 文档标签（见“XML 文档”）。

