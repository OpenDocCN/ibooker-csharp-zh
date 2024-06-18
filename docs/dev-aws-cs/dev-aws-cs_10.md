# 附录 B. 使用 .NET 入门

从浏览器开始交互式文档的 [Hello World](https://oreil.ly/nExkF)。

# GitHub Codespaces 中的 C# Hello World

接下来，尝试使用 GitHub Codespaces 和 [Visual Studio Code 教程](https://oreil.ly/G6CcZ)。

首先，从 CLI 创建一个新的控制台应用，并命名为 `HelloWorld`：

```cs
dotnet new console -n HelloWorld
```

这一步创建了一个控制台项目。接下来，将 *Program.cs* 中的代码更改为以下示例：

```cs
using System;

namespace HelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World C# on AWS!");
        }
    }
}
```

接下来，进入目录（**`cd HelloWorld`**）并运行代码 **`dotnet run`**。该命令的输出如下：

```cs
Hello World C# on AWS!
```

# GitHub Codespaces 中的 F# Hello World

创建一个新的 F# 控制台应用：

```cs
dotnet new console -lang "F#" -n FSharpHelloWorld
```

将代码更改为以下内容：

```cs
open System

// Define a function to construct a message to print
let from whom =
    sprintf "from %s" whom

[<EntryPoint>]
let main argv =
    let message = from "F# on AWS" // Call the function
    printfn "Hello world %s" message
    0 // return an integer exit code
```

接下来，进入目录（**`cd FSharpHelloWorld`**）并运行 **`dotnet run`**。输出将是：`Hello world from F# on AWS`。

###### 注意

您可以在 [O'Reilly 平台](https://oreil.ly/Zqv9z) 或 [YouTube](https://youtu.be/7kkDXf3I4sQ) 上观看此过程的演示。
