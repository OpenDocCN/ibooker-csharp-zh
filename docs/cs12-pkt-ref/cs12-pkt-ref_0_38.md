# 第三十八章：特性

您已经熟悉了使用修饰符（如 `virtual` 或 `ref`）为程序的代码元素添加属性的概念。这些构造已内置于语言中。*属性*是一种可扩展的机制，用于向代码元素（程序集、类型、成员、返回值和参数）添加自定义信息。此可扩展性对于深度集成到类型系统中而无需特殊关键字或构造的服务非常有用。

## 属性类

属性由从抽象类 `System.Attribute` 直接或间接继承的类定义。要将属性附加到代码元素，请在代码元素之前的方括号中指定属性的类型名称。例如，以下示例将 `ObsoleteAttribute` 附加到 `Foo` 类：

```cs
[ObsoleteAttribute]
public class Foo {...}
```

编译器识别此特定属性，并且如果引用了标记为过时的类型或成员，则会导致编译器警告。按照惯例，所有属性类型的名称以 *Attribute* 结尾。C# 识别此规则，并允许您在附加属性时省略后缀：

```cs
[Obsolete]
public class Foo {...}
```

`ObsoleteAttribute` 是在 `System` 命名空间中声明的类型，如下所示（为简洁起见已简化）：

```cs
public sealed class ObsoleteAttribute : Attribute {...}
```

## 命名和位置属性参数

属性可以具有参数。在以下示例中，我们将 `XmlElementAttribute` 应用于一个类。此属性指示 `XmlSerializer`（位于 `System.Xml.Serialization` 中）如何在 XML 中表示对象，并接受几个*属性参数*。以下属性将 `CustomerEntity` 类映射到名为 `Customer` 的 XML 元素，属于 `http://oreilly.com` 命名空间：

```cs
[XmlElement ("Customer", Namespace="http://oreilly.com")]
public class CustomerEntity { ... }
```

属性参数分为两类：位置参数或命名参数。在上述示例中，第一个参数是*位置参数*；第二个是*命名参数*。位置参数对应于属性类型的公共构造函数的参数。命名参数对应于属性类型的公共字段或公共属性。

当指定属性时，必须包含对应于属性构造函数之一的位置参数。命名参数是可选的。

## 属性目标

隐式地，属性的目标是它所紧随的代码元素，通常是类型或类型成员。但是，您也可以将属性附加到程序集。这要求您明确指定属性的目标。以下是使用 `CLSCompliant` 属性指定整个程序集的公共语言规范（CLS）兼容性的示例：

```cs
[assembly:CLSCompliant(true)]
```

从 C# 10 开始，您可以将属性应用于 lambda 表达式的方法、参数和返回值：

```cs
Action<int> a =
  [Description ("Method")]
  [return: Description ("Return value")]
  ([Description ("Parameter")]int x) => Console.Write (x);
```

## 指定多个属性

您可以为单个代码元素指定多个属性。您可以在同一对方括号中列出每个属性（用逗号分隔），也可以分别在不同的方括号对中列出（或两者结合）。以下两个示例在语义上是相同的：

```cs
[Serializable, Obsolete, CLSCompliant(false)]
public class Bar {...}

[Serializable] [Obsolete] [CLSCompliant(false)]
public class Bar {...}
```

## 编写自定义属性

您可以通过子类化`System.Attribute`来定义自己的属性。例如，您可以使用以下自定义属性来标记用于单元测试的方法：

```cs
[AttributeUsage (AttributeTargets.Method)]
public sealed class TestAttribute : Attribute
{
  public int     Repetitions;
  public string  FailureMessage;

  public TestAttribute () : this (1) { }
  public TestAttribute (int repetitions)
    => Repetitions = repetitions;
}
```

下面是如何应用该属性的示例：

```cs
class Foo
{
  [Test]
  public void Method1() { ... }

  [Test(20)]
  public void Method2() { ... }

  [Test(20, FailureMessage="Debugging Time!")]
  public void Method3() { ... }
}
```

`AttributeUsage` 本身就是一个属性，指示可以应用自定义属性的结构（或结构组合）。 `AttributeTargets` 枚举包括`Class`、`Method`、`Parameter`和`Constructor`（以及`All`，它结合了所有目标）等成员。

## 在运行时检索属性

在运行时检索属性的两种标准方法：

+   在任何`Type`或`MemberInfo`对象上调用`GetCustomAttributes`

+   调用`Attribute.GetCustomAttribute`或`Attribute.GetCustomAttributes`

这两个后续方法都已重载，以接受与有效属性目标（`Type`、`Assembly`、`Module`、`MemberInfo`或`ParameterInfo`）对应的任何反射对象。

下面是如何枚举前述`Foo`类中具有`TestAttribute`的每个方法：

```cs
foreach (MethodInfo mi in typeof (Foo).GetMethods())
{
  TestAttribute att = (TestAttribute)
    Attribute.GetCustomAttribute
     (mi, typeof (TestAttribute));

  if (att != null)
    Console.WriteLine (
      "{0} will be tested; reps={1}; msg={2}",
      mi.Name, att.Repetitions, att.FailureMessage);
}
```

下面是输出：

```cs
Method1 will be tested; reps=1; msg=
Method2 will be tested; reps=20; msg=
Method3 will be tested; reps=20; msg=Debugging Time!
```

