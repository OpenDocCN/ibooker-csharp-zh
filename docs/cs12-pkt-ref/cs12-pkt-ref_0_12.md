# 第十二章：命名空间

*命名空间*是类型名称必须唯一的域。类型通常组织在层次结构命名空间中，这样既可以避免命名冲突，又可以更容易找到类型名称。例如，处理公钥加密的`RSA`类型定义在以下命名空间中：

```cs
System.Security.Cryptography
```

命名空间是类型名称的一个重要部分。以下代码调用`RSA`的`Create`方法：

```cs
System.Security.Cryptography.RSA rsa =
  System.Security.Cryptography.RSA.Create();
```

###### 注意

命名空间独立于程序集，这些程序集是部署单位，例如*.exe*或*.dll*。

命名空间对成员的可访问性没有影响 — `public`、`internal`、`private`等等。

`namespace`关键字为该块内的类型定义了一个命名空间。例如：

```cs
namespace Outer.Middle.Inner
{
  class Class1 {}
  class Class2 {}
}
```

命名空间中的点表示嵌套命名空间的层次结构。接下来的代码在语义上与前面的示例相同：

```cs
namespace Outer
{
  namespace Middle
  {
    namespace Inner
    {
      class Class1 {}
      class Class2 {}
    }
  }
}
```

您可以使用*完全限定名称*引用类型，其中包括从最外层到最内层的所有命名空间。例如，您可以将前面示例中的`Class1`称为`Outer.Middle.Inner.Class1`。

未定义在任何命名空间中的类型被称为*全局命名空间*。全局命名空间还包括顶层命名空间，例如我们示例中的`Outer`。

## 文件范围的命名空间

通常，您希望文件中的所有类型都定义在一个命名空间中：

```cs
namespace MyNamespace
{
  class Class1 {}
  class Class2 {}
}
```

自 C# 10 起，您可以使用*文件范围的命名空间*实现此目的：

```cs
namespace MyNamespace;  // Applies to everything below

class Class1 {}         // inside MyNamespace
class Class2 {}         // inside MyNamespace
```

文件范围的命名空间减少了混乱，并消除了不必要的缩进级别。

## using 指令

`using`指令*导入*一个命名空间，是一种方便的方法，可以引用类型而不必使用完全限定名称。例如，您可以像下面这样引用前面示例中的`Class1`：

```cs
using Outer.Middle.Inner;

Class1 c;    // Don't need fully qualified name
```

`using`指令可以嵌套在命名空间内本身，以限制指令的作用范围。

## 全局 using 指令

自 C# 10 起，如果您在`using`指令前加上`global`关键字，则该指令将应用于项目或编译单元中的所有文件：

```cs
global using System;
global using System.Collection.Generic;
```

这使得您可以集中管理常见的导入，避免在每个文件中重复相同的指令。

`global using`指令必须在非全局指令之前，并且不能出现在命名空间声明内。全局指令可以与`using static`一起使用。

## 使用静态导入

`using static`指令导入的是一个*类型*而不是命名空间。该类型的所有静态成员可以在不用限定类型名称的情况下使用。在以下示例中，我们调用`Console`类的静态`WriteLine`方法：

```cs
using static System.Console;

WriteLine ("Hello");
```

`using static` 指令导入类型的所有可访问的静态成员，包括字段、属性和嵌套类型。你还可以将此指令应用于枚举类型（见“枚举”），在这种情况下，它们的成员被导入。如果在多个静态导入之间出现歧义，C# 编译器无法从上下文中推断出正确的类型，并将生成错误。

## 命名空间内的规则

### 名称作用域

在内部命名空间中，可以不带限定地使用外部命名空间中声明的名称。在此示例中，`Class1` 在 `Inner` 中不需要限定：

```cs
namespace Outer
{
  class Class1 {}

  namespace Inner
  {
    class Class2 : Class1 {}
  }
}
```

如果你想引用命名空间层次结构中不同分支的类型，可以使用部分限定名称。在以下示例中，我们基于 `Common.ReportBase` 创建 `SalesReport`：

```cs
namespace MyTradingCompany
{
  namespace Common
  {
    class ReportBase {}
  }
  namespace ManagementReporting
  {
    class SalesReport : Common.ReportBase {}
  }
}
```

### 名称隐藏

如果同一类型名称同时出现在内部和外部命名空间中，则内部名称优先。要引用外部命名空间中的类型，必须限定其名称。

###### 注意

所有类型名称在编译时转换为完全限定名称。中间语言（IL）代码不包含未限定或部分限定的名称。

### 重复的命名空间

可以重复命名空间声明，只要命名空间内的类型名称不冲突：

```cs
namespace Outer.Middle.Inner { class Class1 {} }
namespace Outer.Middle.Inner { class Class2 {} }
```

类可以跨源文件和程序集。

### 全局限定符 `global::`

偶尔，完全限定的类型名称可能与内部名称冲突。你可以通过在其前面加上 `global::` 强制 C# 使用完全限定的类型名称，如下所示：

```cs
global::System.Text.StringBuilder sb;
```

## 类型和命名空间别名

导入命名空间可能会导致类型名称冲突。与其导入整个命名空间，你可以只导入需要的特定类型，并为每个类型起一个别名。例如：

```cs
using PropertyInfo2 = System.Reflection.PropertyInfo;
class Program { PropertyInfo2 p; }
```

可以通过以下方式给整个命名空间起别名：

```cs
using R = System.Reflection;
class Program { R.PropertyInfo p; }
```

### 类型别名（C# 12）

自 C# 12 起，`using` 指令可以为任何类型和命名空间起别名，包括例如数组：

```cs
using NumberList = double[];
NumberList numbers = { 2.5, 3.5 };
```

你还可以给元组起别名（见“元组”）。

