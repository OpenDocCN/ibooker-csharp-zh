# 第二十六章：try 语句和异常

`try` 语句指定一个可能包含错误处理或清理代码的代码块。`try` *块* 后必须跟随一个或多个 `catch` *块* 和/或一个 `finally` *块*。当 `try` 块中抛出错误时，执行 `catch` 块。`finally` 块在执行离开 `try` 块（或如有的话，`catch` 块）后执行清理代码，无论是否抛出异常。

`catch` 块可以访问包含有关错误信息的 `Exception` 对象。你可以使用 `catch` 块来补偿错误或*重新抛出*异常。如果仅想记录问题，或者想要重新抛出新的更高级别的异常类型，则重新抛出异常。

`finally` 块通过始终执行来为程序添加确定性，无论如何。它对于如关闭网络连接等清理任务非常有用。

`try` 语句如下所示：

```cs
try
{
  ... // exception may get thrown within execution of
      // this block
}
catch (ExceptionA ex)
{
  ... // handle exception of type ExceptionA
}
catch (ExceptionB ex)
{
  ... // handle exception of type ExceptionB
}
finally
{
  ... // cleanup code
}
```

考虑以下代码：

```cs
int x = 3, y = 0;
Console.WriteLine (x / y);
```

因为 `y` 为零，运行时抛出 `DivideByZeroException` 并终止我们的程序。我们可以通过以下方式捕获异常来防止这种情况：

```cs
try
{
  int x = 3, y = 0;
  Console.WriteLine (x / y);
}
catch (DivideByZeroException)
{
  Console.Write ("y cannot be zero. ");
}
// Execution resumes here after exception...
```

###### 注意

这是一个简单的示例，用于说明异常处理。在实践中，我们可以通过在调用 `Calc` 之前显式检查除数是否为零来更好地处理这种情况。

处理异常相对昂贵，需要数百个时钟周期。

当在 `try` 语句内抛出异常时，CLR 进行测试：

*try* *语句有兼容的*catch*块吗？*

+   如果有，则执行跳转到兼容的`catch`块，然后是`finally`块（如果存在），然后执行正常继续。

+   如果不是，则执行直接跳转到 `finally` 块（如果存在），然后 CLR 查找调用堆栈中的其他 `try` 块，如果找到，则重复测试。

如果调用堆栈中没有任何函数负责异常，则向用户显示错误对话框并终止程序。

## catch 子句

`catch` 子句指定要捕获的异常类型。这必须是 `System.Exception` 或 `System.Exception` 的子类。捕获 `System.Exception` 可以捕获所有可能的错误。这在以下情况下非常有用：

+   你的程序可能会无论特定异常类型如何都能够恢复。

+   你计划重新抛出异常（可能在记录日志后）。

+   在程序终止之前，你的错误处理程序是最后的救援。

不过，更典型的情况是，你捕获*特定的异常类型*，以避免处理处理程序未设计的情况（例如 `OutOfMemoryException`）。

你可以通过多个 `catch` 子句处理多个异常类型：

```cs
try
{
  DoSomething();
}
catch (IndexOutOfRangeException ex) { ... }
catch (FormatException ex)          { ... }
catch (OverflowException ex)        { ... }
```

对于给定异常，只有一个 `catch` 子句执行。如果要包含一个安全网以捕获更一般的异常（如 `System.Exception`），则必须先放置更具体的处理程序*第一*。

不需要指定变量即可捕获异常：

```cs
catch (OverflowException)   // no variable
{ ... }
```

此外，你可以省略变量和类型（意味着捕获所有异常）：

```cs
catch { ... }
```

### 异常过滤器

你可以通过添加`when`子句在`catch`子句中指定异常过滤器：

```cs
catch (WebException ex) 
  when (ex.Status == WebExceptionStatus.Timeout)
{
  ...
}
```

如果在此示例中抛出 `WebException`，则会评估 `when` 关键字后面的布尔表达式。如果结果为假，则忽略相应的 `catch` 块，并考虑任何后续的 `catch` 子句。使用异常筛选器时，捕获相同的异常类型可能很有意义：

```cs
catch (WebException ex) when (ex.Status == *something*)
{ ... }
catch (WebException ex) when (ex.Status == *somethingelse*)
{ ... }
```

`when` 子句中的布尔表达式可能具有副作用，例如记录异常以进行诊断目的的方法。

## finally 块

`finally` 块始终执行 —— 无论是否抛出异常以及 `try` 块是否完整运行。`finally` 块通常用于清理代码。

`finally` 块执行：

+   在 `catch` 块完成后

+   由于 `jump` 语句（例如 `return` 或 `goto`）导致控制离开 `try` 块后。

+   在 `try` 块结束后

`finally` 块有助于为程序增加确定性。在以下示例中，无论是否：

+   `try` 块正常结束。

+   由于文件为空(`EndOfStream`)，执行提前返回。

+   在读取文件时抛出 `IOException`。

```cs
static void ReadFile()
{
  StreamReader reader = null;  // In System.IO namespace
  try
  {
    reader = File.OpenText ("file.txt");
    if (reader.EndOfStream) return;
    Console.WriteLine (reader.ReadToEnd());
  }
  finally
  {
    if (reader != null) reader.Dispose();
  }
}
```

在此示例中，我们通过在 `StreamReader` 上调用 `Dispose` 来关闭文件。在 .NET 中，调用对象的 `Dispose` 方法是一个标准惯例，并且在 C# 中通过 `using` 语句得到了显式支持。

### using 声明

许多类封装了不受管理的资源，例如文件句柄、图形句柄或数据库连接。这些类实现了 `System.IDisposable`，它定义了一个名为 `Dispose` 的无参数方法，用于清理这些资源。`using` 语句提供了一个优雅的语法，用于在 `finally` 块内调用 `IDisposable` 对象的 `Dispose` 方法。

以下：

```cs
using (StreamReader reader = File.OpenText ("file.txt"))
{
  ...
}
```

等价于：

```cs
{
  StreamReader reader = File.OpenText ("file.txt");
  try
  {
    ...
  }
  finally
  {
    if (reader != null) ((IDisposable)reader).Dispose();
  }
}
```

### using 声明

如果省略 `using` 语句后面的括号和语句块，它就成为*using 声明*（C# 8+）。当执行流离开*封闭的*语句块时，资源将被处理：

```cs
if (File.Exists ("file.txt"))
{
 using var reader = File.OpenText ("file.txt"); 
  Console.WriteLine (reader.ReadLine());
  ...
}
```

在这种情况下，当执行流离开 `if` 语句块时，`reader` 将被处理。

## 抛出异常

异常可以由运行时或用户代码抛出。在此示例中，`Display` 抛出 `System.ArgumentNul⁠l​Exception`：

```cs
static void Display (string name)
{
  if (name == null)
 throw new ArgumentNullException (nameof (name));

  Console.WriteLine (name);
}
```

### 抛出表达式

从 C# 7 开始，`throw` 可以作为表达式出现在表达体函数中：

```cs
public string Foo() => throw new NotImplementedException();
```

`throw` 表达式也可以出现在三元条件表达式中：

```cs
string ProperCase (string value) =>
  value == null ? throw new ArgumentException ("value") :
  value == "" ? "" :
  char.ToUpper (value[0]) + value.Substring (1);
```

### 重新抛出异常

可以按如下方式捕获并重新抛出异常：

```cs
try {  ...  }
catch (Exception ex)
{
  // Log error
  ...
 throw;          // Rethrow same exception
}
```

以这种方式重新抛出异常，可以在不*吞噬*异常的情况下记录错误。它还允许你在情况超出预期时退出异常处理。

###### 注意

如果我们将 `throw` 替换为 `throw ex`，示例仍将正常工作，但异常的 `StackTrace` 属性将不再反映原始错误。

另一种常见的情况是重新抛出更具体或有意义的异常类型：

```cs
try
{
  ... // parse a date of birth from XML element data
}
catch (FormatException ex)
{
  throw new XmlException ("Invalid date of birth", ex);
}
```

在重新抛出不同的异常时，可以将 `InnerException` 属性填充为原始异常以帮助调试。几乎所有类型的异常都提供了此目的的构造函数（例如我们的示例中）。

## `System.Exception` 的关键属性

`System.Exception` 的最重要属性如下：

`StackTrace`

一个字符串，表示从异常的起源到 `catch` 块调用的所有方法。

`Message`

一个带有错误描述的字符串。

`InnerException`

内部异常（如果有）引发了外部异常。这本身可能有另一个 `InnerException`。

