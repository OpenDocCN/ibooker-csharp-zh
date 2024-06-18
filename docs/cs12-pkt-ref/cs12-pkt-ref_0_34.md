# 第三十四章：模

早些时候，我们演示了如何使用 `is` 运算符测试引用转换是否成功，然后使用其转换后的值：

```cs
if (obj is string s)
  Console.WriteLine (s.Length);
```

这使用了一种称为 *类型模式* 的模式。`is` 运算符还支持其他在最近版本的 C# 中引入的模式。模式在以下上下文中受支持：

+   在 `is` 运算符之后（`*variable* is *pattern*`）

+   在 `switch` 语句中

+   在 `switch` 表达式中

我们已经在“在类型上进行切换”和“is 运算符”中介绍了类型模式。在本节中，我们将介绍在最近版本的 C# 中引入的更高级的模式。

一些更专业的模式旨在在 `switch` 语句/表达式中使用。在这里，它们减少了对 `when` 子句的需求，并让您在以前无法使用 `switch` 的地方使用它们。

## 变量模式

*var 模式* 是 *类型模式* 的一种变体，您可以用 `var` 关键字替换类型名称。转换总是成功的，因此它的目的仅仅是让您重用随后的变量：

```cs
bool IsJanetOrJohn (string name) => 
  name.ToUpper() is var upper && 
  (upper == "JANET" || upper == "JOHN");
```

这相当于：

```cs
bool IsJanetOrJohn (string name)
{
  string upper = name.ToUpper();
  return upper == "JANET" || upper == "JOHN";
}
```

## 常量模式

*常量模式* 允许您直接匹配到一个常量，对于与 `object` 类型一起工作时非常有用：

```cs
void Foo (object obj) 
{
  if (obj is 3) ...
}
```

这个加粗的表达式相当于以下内容：

```cs
obj is int && (int)obj == 3
```

正如我们将很快看到的那样，常量模式在使用 *模式组合器* 时会更加有用。

## 关系模式

自 C# 9 开始，您可以在模式中使用 `<`、`>`、`<=` 和 `>=` 运算符：

```cs
if (x is > 100) Console.Write ("x is greater than 100");
```

在 `switch` 中这变得非常有用：

```cs
string GetWeightCategory (decimal bmi) => bmi switch
{
  < 18.5m => "underweight",
  < 25m => "normal",
  < 30m => "overweight",
  _ => "obese"
};
```

## 模式组合器

自 C# 9 开始，您可以使用 `and`、`or` 和 `not` 关键字来组合模式：

```cs
bool IsJanetOrJohn (string name)
  => name.ToUpper() is "JANET" or "JOHN";

bool IsVowel (char c) 
  => c is 'a' or 'e' or 'i' or 'o' or 'u';

bool Between1And9 (int n) => n is >= 1 and <= 9;

bool IsLetter (char c) => c is >= 'a' and <= 'z'
                            or >= 'A' and <= 'Z';
```

与`&&`和`||`运算符一样，`and`比`or`具有更高的优先级。您可以使用括号覆盖这一点。一个很好的技巧是将`not`组合器与*类型模式*结合起来，以测试对象是否为（不是）某种类型：

```cs
if (obj is not string) ...
```

这看起来比以下方式更好：

```cs
if (!(obj is string)) ...
```

## 元组和位置模式

*元组模式*（C# 8 中引入）匹配元组：

```cs
var p = (2, 3);
Console.WriteLine (p is (2, 3));   // True
```

元组模式可以被视为*位置模式*（C# 8+）的特殊情况，它匹配任何公开`Deconstruct`方法的类型（参见“解构器”）。在以下示例中，我们利用`Point`记录的编译器生成的解构器：

```cs
var p = new Point (2, 2);
Console.WriteLine (p is (2, 2));  // True

record Point (int X, int Y);
```

您可以在匹配时解构，使用以下语法：

```cs
Console.WriteLine (p is (var x, var y) && x == y);
```

这是一个将类型模式与位置模式结合的 switch 表达式：

```cs
string Print (object obj) => obj switch 
{
  Point (0, 0)                      => "Empty point",
  Point (var x, var y) when x == y  => "Diagonal"
  ...
};
```

## 属性模式

*属性模式*（C# 8+）匹配对象的一个或多个属性值：

```cs
if (obj is string { Length:4 }) ...
```

然而，这与以下方式并没有太大区别：

```cs
if (obj is string s && s.Length == 4) ...
```

对于 switch 语句和表达式，属性模式更有用。考虑`System.Uri`类，它表示一个 URI。它具有的属性包括`Scheme`、`Host`、`Port`和`IsLoopback`。在编写防火墙时，我们可以通过使用使用属性模式的 switch 表达式来决定是否允许或阻止 URI：

```cs
bool ShouldAllow (Uri uri) => uri switch
{
  { Scheme: "http",  Port: 80  } => true,
  { Scheme: "https", Port: 443 } => true,
  { Scheme: "ftp",   Port: 21  } => true,
  { IsLoopback: true           } => true,
  _ => false
};
```

您可以嵌套属性，使以下子句合法：

```cs
  { Scheme: { Length: 4 }, Port: 80 } => true,
```

从 C# 10 开始，可以简化为：

```cs
  { Scheme.Length: 4, Port: 80 } => true,
```

您可以在属性模式中使用其他模式，包括关系模式：

```cs
  { Host: { Length: < 1000 }, Port: > 0 } => true,
```

你可以在子句末尾引入一个变量，然后在`when`子句中使用该变量：

```cs
  { Scheme: "http", Port: 80 } httpUri 
      when httpUri.Host.Length < 1000 => true,
```

您还可以在*属性*级别引入变量：

```cs
  { Scheme: "http", Port: 80, Host: var host }
      when host.Length < 1000 => true,
```

然而，在这种情况下，以下方式更短更简单：

```cs
  { Scheme: "http", Port: 80, Host: { Length: < 1000 } }
```

## 列表模式

列表模式（从 C# 11 开始）适用于任何可计数的集合类型（具有`Count`或`Length`属性）和可索引的集合类型（具有`int`或`System.Index`类型的索引器）。

列表模式匹配方括号中的一系列元素：

```cs
int[] numbers = { 0, 1, 2, 3, 4 };
Console.Write (numbers is [0, 1, 2, 3, 4]);   // True
```

下划线匹配任何值的单个元素：

```cs
Console.Write (numbers is [0, 1, _, _, 4]);   // True
```

`var` 模式也适用于匹配单个元素：

```cs
Console.Write (numbers is [0, 1, var x, 3, 4] && x > 1);
```

两个点表示一个*切片*。切片匹配零个或多个元素：

```cs
Console.Write (numbers is [0, .., 4]);    // True
```

对于支持索引和范围的数组和其他类型（参见“索引和范围”），您可以在切片后跟一个`var`模式：

```cs
Console.Write (numbers is [0, .. var mid, 4]
               && mid.Contains (2));           // True
```

列表模式最多可以包含一个切片。

