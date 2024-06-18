# 第一章：第一个 C# 程序

以下是一个将 12 乘以 30 并将结果 360 打印到屏幕的程序。双斜线表示行的剩余部分是*注释*：

```cs
int x = 12 * 30;                  // Statement 1
System.Console.WriteLine (x);     // Statement 2
```

我们的程序由两个*语句*组成。C# 中的语句按顺序执行，并以分号终止。第一个语句计算*表达式* `12 * 30` 并将结果存储在名为 `x` 的*变量*中，其类型是 32 位整数 (`int`)。第二个语句在名为 `Console` 的*类*上调用 `WriteLine` *方法*，该类定义在名为 `System` 的*命名空间*中。这将变量 `x` 打印到屏幕上的文本窗口中。

方法执行一个功能；类将函数成员和数据成员组合成面向对象的构建块。`Console` 类组合了处理命令行输入/输出（I/O）功能的成员，比如 `WriteLine` 方法。类是一种*类型*，我们在“类型基础”中讨论过。

在最外层，类型被组织到*命名空间*中。许多常用类型，包括 `Console` 类，驻留在 `System` 命名空间中。.NET 库被组织成嵌套命名空间。例如，`System.Text` 命名空间包含用于处理文本的类型，而 `System.IO` 包含用于输入/输出的类型。

在每次使用时用 `System` 命名空间限定 `Console` 类会增加混乱。`using` 指令允许您通过*导入*一个命名空间来避免这种混乱：

```cs
using System;            // Import the System namespace

int x = 12 * 30;
Console.WriteLine (x);   // No need to specify System
```

代码重用的基本形式是编写调用低级函数的高级函数。我们可以通过一个可重用的*方法* `FeetToInches` 来*重构*我们的程序，该方法将整数乘以 12，如下所示：

```cs
using System;

Console.WriteLine (FeetToInches (30));      // 360
Console.WriteLine (FeetToInches (100));     // 1200

int FeetToInches (int feet)
{
 int inches = feet * 12;
 return inches;
}
```

我们的方法包含一系列被大括号包围的语句。这被称为*语句块*。

方法可以通过指定*参数*从调用者那里接收*输入*数据，并通过指定*返回类型*向调用者返回*输出*数据。我们的`FeetToInches`方法具有输入英尺的参数和输出英寸的返回类型：

```cs
int FeetToInches (int feet)
...
```

*字面量* `30` 和 `100` 是传递给`FeetToInches`方法的*参数*。

如果方法不接收输入，请使用空括号。如果方法不返回任何内容，请使用`void`关键字：

```cs
using System;
SayHello();

void SayHello()
{
  Console.WriteLine ("Hello, world");
}
```

方法是 C#中几种函数的一种。我们示例程序中使用的另一种函数是`*` *操作符*，它执行乘法运算。还有*构造函数*、*属性*、*事件*、*索引器*和*终结器*。

## 编译

C#编译器将源代码（一组扩展名为*.cs*的文件）编译成一个*程序集*。程序集是.NET 中的打包和部署单元。程序集可以是*应用程序*或*库*。普通控制台或 Windows 应用程序具有*入口点*，而库则没有。库的目的是被应用程序或其他库*引用*。.NET 本身是一组库（以及运行时环境）。

在前面的节中，每个程序都直接以一系列语句开始（称为*顶级语句*）。顶级语句的存在隐式创建了控制台或 Windows 应用程序的入口点。（没有顶级语句时，*Main 方法*表示应用程序的入口点—参见“预定义类型和自定义类型的对称性”。）

要调用编译器，您可以使用集成开发环境（IDE）如 Visual Studio 或 Visual Studio Code，也可以从命令行手动调用它。要使用.NET 手动编译控制台应用程序，首先下载.NET 8 SDK，然后按以下步骤创建新项目：

```cs
dotnet new console -o MyFirstProgram
cd MyFirstProgram
```

这将创建一个名为*MyFirstProgram*的文件夹，其中包含一个名为*Program.cs*的 C#文件，您可以随后编辑。要调用编译器，请调用`dotnet build`（或`dotnet run`，它将编译然后运行程序）。输出将写入*bin\debug*子目录下，其中包括*MyFirstProgram.dll*（输出程序集）以及直接运行编译程序的*MyFirstProgram.exe*。

