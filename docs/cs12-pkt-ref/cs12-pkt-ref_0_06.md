# 第六章：字符串和字符

C#的`char`类型（别名为`System.Char`类型）表示一个 Unicode 字符，占据两个字节（UTF-16）。`char`字面值用单引号指定：

```cs
char c = 'A';       // Simple character
```

*转义序列*表示不能以字面方式表达或解释的字符。转义序列是一个反斜杠后跟具有特殊含义的字符。例如：

```cs
char newLine = '\n';
char backSlash = '\\';
```

有效的转义序列如下：

| Char | 含义 | 值 |
| --- | --- | --- |
| `\'` | 单引号 | `0x0027` |
| `\"` | 双引号 | `0x0022` |
| `\\` | 反斜杠 | `0x005C` |
| `\0` | 空字符 | `0x0000` |
| `\a` | 警报 | `0x0007` |
| `\b` | 退格 | `0x0008` |
| `\f` | 换页符 | `0x000C` |
| `\n` | 换行 | `0x000A` |
| `\r` | 回车 | `0x000D` |
| `\t` | 水平制表符 | `0x0009` |
| `\v` | 垂直制表符 | `0x000B` |

`\u`（或`\x`）转义序列允许您通过其四位十六进制代码指定任何 Unicode 字符：

```cs
char copyrightSymbol = '\u00A9';
char omegaSymbol     = '\u03A9';
char newLine         = '\u000A';
```

从`char`到能容纳无符号`short`的数值类型的隐式转换适用于其他数值类型，需要显式转换。

## 字符串类型

C#的`string`类型（别名`System.String`类型）表示一种不可变（不可修改）的 Unicode 字符序列。字符串字面量在双引号内指定：

```cs
string a = "Heat";
```

###### 注意

`string`是引用类型，而不是值类型。然而，它的相等运算符遵循值类型语义：

```cs
string a = "test", b = "test";
Console.Write (a == b);  // True
```

在字符串内也适用于`char`字面量的转义序列：

```cs
string a = "Here's a tab:\t";
```

代价是，每当您需要一个字面的反斜杠时，您必须写两次：

```cs
string a1 = "\\\\server\\fileshare\\helloworld.cs";
```

为了避免这个问题，C#允许*逐字*字符串字面量。逐字字符串字面量以`@`为前缀，不支持转义序列。以下逐字字符串与前述相同：

```cs
string a2 = @"\\server\fileshare\helloworld.cs";
```

逐字字符串字面量也可以跨越多行。您可以通过连续写两次来包含逐字文字中的双引号字符。

### 原始字符串字面量（C# 11）

用三个或更多引号字符（`"""`）包裹字符串会创建一个*原始字符串字面量*。原始字符串字面量几乎可以包含任何字符序列，无需转义或重复：

```cs
string raw = """<file path="c:\temp\test.txt"></file>""";
```

原始字符串字面量使得能够轻松表示 JSON、XML 和 HTML 字面量，以及正则表达式和源代码。如果需要在字符串本身中包含三个（或更多）引号字符，则可以用四个（或更多）引号字符包裹该字符串：

```cs
string raw = """"We can include """ in this string."""";
```

多行原始字符串字面量受到特殊规则的约束。我们可以将字符串`"Line 1\r\nLine 2"`表示如下：

```cs
string multiLineRaw = """
  Line 1
  Line 2
  """;
```

注意，开头和结尾的引号必须位于字符串内容的不同行。此外：

+   在*开头*的`"""`后面的空白会被忽略。

+   在*结尾*的`"""`前面的空白被视为*常见缩进*并从字符串的每一行中删除。这使得您可以包含源代码可读性的缩进（就像我们在示例中所做的那样），而不使该缩进成为字符串的一部分。

原始字符串字面量可以进行插值，受到“字符串插值”中描述的特殊规则的约束。

### 字符串连接

`+`运算符连接两个字符串：

```cs
string s = "a" + "b";
```

操作数中的一个可以是非字符串值，在这种情况下，将对该值调用`ToString`。例如：

```cs
string s = "a" + 5;  // a5
```

反复使用`+`运算符构建字符串可能效率低下；更好的解决方案是使用`System.Text.StringBuilder`类型——它表示可变（可编辑）字符串，并具有有效地`Append`、`Insert`、`Remove`和`Replace`子字符串的方法。

### 字符串插值

以`$`字符开头的字符串称为*插值字符串*。插值字符串可以在大括号内包含表达式：

```cs
int x = 4;
Console.Write ($"A square has {x} sides");
// Prints: A square has 4 sides
```

任何有效的 C#表达式可以出现在大括号内，并且 C#将通过调用其`ToString`方法或等效方法将表达式转换为字符串。您可以通过在表达式后附加冒号和*格式字符串*（我们在*C# 12 in a Nutshell*的第六章中描述了格式字符串）来更改格式：

```cs
string s = $"15 in hex is {15:X2}";
// Evaluates to "15 in hex is 0F"
```

从 C# 10 开始，插值字符串可以是常量，只要插值的值是常量即可：

```cs
const string greeting = "Hello";
const string message = $"{greeting}, world";
```

从 C# 11 开始，插值字符串可以跨多行（无论是标准还是逐字字符串）：

```cs
string s = $"this interpolation spans {1 +
1} lines";
```

原始字符串字面量（从 C# 11 开始）也可以进行插值：

```cs
string s = $"""The date and time is {DateTime.Now}""";
```

要在插值字符串中包含大括号文字：

+   使用标准和逐字字符串字面量，重复所需的大括号字符。

+   使用原始字符串字面量，通过重复`$`前缀来改变插值序列。

在原始字符串字面量的前缀中使用两个（或更多）`$`字符可以改变插值序列，从一个大括号变为两个（或更多）。考虑以下字符串：

```cs
$$"""{ "TimeStamp": "{{DateTime.Now}}" }"""
```

这将评估为：

```cs
{ "TimeStamp": "01/01/2024 12:13:25 PM" }
```

### 字符串比较

`string`不支持`<`和`>`操作符进行比较。您必须使用`string`的`CompareTo`方法，该方法返回正数、负数或零，具体取决于第一个值出现在第二个值之后、之前还是与第二个值并列：

```cs
Console.Write ("Boston".CompareTo ("Austin"));   // 1
Console.Write ("Boston".CompareTo ("Boston"));   // 0
Console.Write ("Boston".CompareTo ("Chicago"));  // -1
```

### 在字符串内搜索

`string`的索引器返回指定位置的字符：

```cs
Console.Write ("word"[2]);   // r
```

`IndexOf`和`LastIndexOf`方法用于在字符串中搜索字符。`Contains`、`StartsWith`和`EndsWith`方法用于在字符串中搜索子字符串。

### 操作字符串

因为`string`是不可变的，所有“操作”字符串的方法都返回一个新字符串，原始字符串保持不变：

+   `Substring`提取字符串的一部分。

+   `Insert`和`Remove`在指定位置插入和删除字符。

+   `PadLeft`和`PadRight`添加空格字符。

+   `TrimStart`、`TrimEnd`和`Trim`移除空白字符。

`string`类还定义了`ToUpper`和`ToLower`方法以更改大小写，`Split`方法以根据提供的分隔符将字符串拆分为子字符串，并且静态`Join`方法以将子字符串连接回字符串。

## UTF-8 字符串

从 C# 11 开始，您可以使用`u8`后缀创建使用 UTF-8 而不是 UTF-16 编码的字符串字面量。此功能旨在用于高级场景，例如性能热点中 JSON 文本的低级处理：

```cs
ReadOnlySpan<byte> utf8 = "ab→cd"u8;
Console.WriteLine (utf8.Length);      // 7
```

底层类型是`ReadOnlySpan<byte>`，我们在*C# 12 in a Nutshell*的第二十三章中介绍了它。您可以通过调用`ToArray()`方法将其转换为数组。

