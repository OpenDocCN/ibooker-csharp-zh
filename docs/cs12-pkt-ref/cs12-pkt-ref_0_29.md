# 第二十九章：可空引用类型

尽管*可空值类型*将空性引入值类型，*可空引用类型*（来自 C# 8）则相反。启用时，它们将（一定程度的）*非空性*引入引用类型，旨在帮助避免`NullReference​Excep⁠tion`。

可空引用类型通过编译器纯粹强制执行安全级别，在检测到有可能生成`NullReference​Excep⁠tion`的代码时生成警告。

要启用可空引用类型，您必须将`Nullable`元素添加到您的*.csproj*项目文件中（如果要为整个项目启用它）：

```cs
<Nullable>enable</Nullable>
```

或者在代码中的适当位置使用以下指令来使其生效：

```cs
#nullable enable   // enables NRT from this point on
#nullable disable  // disables NRT from this point on
#nullable restore  // resets NRT to project setting
```

启用后，编译器将非空性设置为默认值：如果要使引用类型接受空值而不生成警告，必须添加`?`后缀以指示*可空引用类型*。在以下示例中，`s1`是非可空的，而`s2`是可空的：

```cs
#nullable enable    // Enable nullable reference types

string s1 = null;   // Generates a compiler warning!
string? s2 = null;  // OK: s2 is *nullable reference type*
```

###### 注意

因为可空引用类型是编译时构造，所以在`string`和`string?`之间没有运行时差异。相比之下，可空值类型在类型系统中引入了具体内容，即`Null​a⁠ble<T>`结构。

以下也会生成警告，因为`x`未初始化：

```cs
class Foo { string x; }
```

如果通过字段初始化器或构造函数中的代码初始化`x`，警告将消失。

如果编译器认为可能会发生`NullReferenceException`，则在对可空引用类型进行解引用时也会向您发出警告。在以下示例中，访问字符串的 `Length` 属性会生成警告：

```cs
void Foo (string? s) => Console.Write (s.Length);
```

要消除警告，可以使用*空值忽略运算符* (`!`)：

```cs
void Foo (string? s) => Console.Write (s!.Length);
```

在这个示例中使用的空值忽略运算符是危险的，因为我们最初试图避免的`NullReferen⁠ce​Exception`可能会被抛出。我们可以按如下方式修复它：

```cs
void Foo (string? s)
{
  if (s != null) Console.Write (s.Length);
}
```

现在请注意，我们不再需要空值忽略运算符。这是因为编译器执行静态分析，并且在简单情况下足够聪明以推断出解引用是安全的，没有`NullReferenceException`的可能。

编译器检测和警告的能力并不是绝对可靠的，而且在覆盖范围方面也存在限制。例如，它无法知道数组的元素是否已被填充，因此以下情况不会生成警告：

```cs
var strings = new string[10];
Console.WriteLine (strings[0].Length);
```

