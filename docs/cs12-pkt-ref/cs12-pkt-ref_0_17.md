# 第十七章：访问修饰符

为了促进封装性，类型或类型成员可以通过在声明中添加*访问修饰符*来限制其对其他类型和其他程序集的*可访问性*：

`public`

完全可访问。这是枚举或接口成员的隐式可访问性。

`internal`

只能在包含的程序集或友元程序集中访问。这是非嵌套类型的默认可访问性。

`private`

只能在包含类型内部访问。这是类或结构体成员的默认可访问性。

`protected`

只能在包含类型或子类中访问。

`protected internal`

`protected`和`internal`可访问性的*并集*（这比单独的`protected`或`internal`更*宽松*，因为它使成员在两个方面更易访问）。

`private protected`

`protected`和`internal`可访问性的*交集*（这比单独的`protected`或`internal`更*严格*）。

`file`（从 C# 11 开始）

只能从同一文件内部访问。用于源代码生成器（见“扩展局部方法”）。此修饰符只能应用于类型声明。

在以下示例中，`Class2`可以从其所在的程序集外部访问；`Class1`则不行：

```cs
class Class1 {}         // Class1 is internal (default)
public class Class2 {}
```

`ClassB`将字段`x`公开给同一程序集中的其他类型；`ClassA`则不会：

```cs
class ClassA { int x;          }  // x is private
class ClassB { internal int x; }
```

当您覆盖基类函数时，覆盖函数的可访问性必须相同。编译器阻止使用访问修饰符的不一致使用，例如，子类本身可以比基类不可访问，但不能更可访问。

## 友元程序集

您可以通过添加`System.Runtime.CompilerServices.InternalsVisibleTo`程序集属性，指定友元程序集的名称，来将`internal`成员暴露给其他*友元*程序集：

```cs
[assembly: InternalsVisibleTo ("Friend")]
```

如果友元程序集使用强名称签名，您必须指定其*完整*的 160 字节公钥。您可以通过语言集成查询（LINQ）提取此密钥—可以在 LINQPad 的* C# 12 简明教程*的第三章“访问修饰符”中的免费示例库中找到交互式示例。

## 可访问性封装

类型限制其声明成员的可访问性。最常见的封装示例是当您有一个带有`public`成员的`internal`类型时。例如：

```cs
class C { public void Foo() {} }
```

`C`的（默认）`internal`可访问性限制了`Foo`的可访问性，从效果上讲使得`Foo`是`internal`。将`Foo`标记为`public`的常见原因是为了更轻松地进行重构，如果以后将`C`更改为`public`。

