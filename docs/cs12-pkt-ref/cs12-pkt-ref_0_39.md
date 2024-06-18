# 第三十九章：调用者信息属性

您可以使用三种*调用者信息属性*之一来标记可选参数，这些属性指示编译器将来自调用方源代码的信息传递到参数的默认值中：

+   `[CallerMemberName]` 应用于调用方的成员名称。

+   `[CallerFilePath]` 应用于调用方的源代码文件路径。

+   `[CallerLineNumber]` 应用于调用方的源代码文件中的行号。

下面的程序中的`Foo`方法演示了所有三种方法：

```cs
using System;
using System.Runtime.CompilerServices;

class Program
{
  static void Main() => Foo();

  static void Foo (
    [CallerMemberName] string memberName = null,
    [CallerFilePath] string filePath = null,
    [CallerLineNumber] int lineNumber = 0)
  {
    Console.WriteLine (memberName);
    Console.WriteLine (filePath);
    Console.WriteLine (lineNumber);
  }
}
```

假设我们的程序位于*c:\source\test\Program.cs*，则输出将是：

```cs
Main
c:\source\test\Program.cs
6
```

与标准可选参数一样，替换是在*调用点*完成的。因此，我们的`Main`方法对于此是语法糖：

```cs
static void Main()
  => Foo ("Main", @"c:\source\test\Program.cs", 6);
```

调用者信息属性对编写日志函数和实现更改通知模式非常有用。例如，我们可以在属性的`set`访问器中调用以下方法，而无需指定属性的名称：

```cs
void RaisePropertyChanged (
  [CallerMemberName] string propertyName = null)
  {  
    ...
  }
```

## CallerArgumentExpression

将`[CallerArgumentExpression]`属性应用于方法参数，以捕获调用点的参数表达式：

```cs
Print (Math.PI * 2);

void Print (double number,
  [CallerArgumentExpression("number")] string expr = null)
    => Console.WriteLine (expr);

// Output: Math.PI * 2
```

此功能的主要应用程序是在编写验证和断言库时。在以下示例中，抛出异常，其消息包含文本“2 + 2 == 5”。这有助于调试：

```cs
Assert (2 + 2 == 5);

void Assert (bool condition,
  [CallerArgumentExpression ("condition")]
  string msg = null)
{
  if (!condition)
   throw new Exception ("Assert failed: " + msg);
}
```

您可以多次使用`[CallerArgumentExpression]`以捕获多个参数表达式。

