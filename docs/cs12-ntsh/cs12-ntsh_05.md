# 第五章 .NET 概述

几乎所有 .NET 8 运行时的功能都通过大量的托管类型暴露出来。这些类型按层次结构命名空间进行组织，并打包到一组程序集中。

一些 .NET 类型由 CLR 直接使用，并且对托管托管环境至关重要。这些类型位于一个名为 *System.Private.CoreLib.dll*（在 .NET Framework 中为 *mscorlib.dll*）的程序集中，包括 C# 的内置类型以及基本的集合类、用于流处理、序列化、反射、线程处理和本地互操作的类型。

在此之上是补充类型，这些类型“丰富”了 CLR 级别的功能，提供 XML、JSON、网络和语言集成查询等功能。它们构成了基础类库（BCL）。在其上层是*应用程序层*，为开发特定类型的应用程序（如 Web 或丰富客户端）提供 API。

在本章中，我们提供以下内容：

+   本书其余部分中会详细介绍 BCL 概述。

+   应用程序层的高级摘要

# 运行时目标和 TFMs

在项目文件中，`<TargetFramework>` 元素确定项目构建的运行时目标（其*框架目标*或*运行时目标*），并由*目标框架标识符*（TFM）表示。有效值包括 `net8.0`、`net7.0`、`net6.0`、`net5.0`（用于 .NET 版本 8、7、6 和 5）、`netcoreapp3.1`（用于 .NET Core 3.1）、`net48`（用于 .NET Framework 4.8）和 `netstandard2.0`（我们将在接下来的部分中介绍）。例如，以下是如何针对 .NET 8 进行目标设置的：

```cs
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  <PropertyGroup>
```

您可以通过指定 `<TargetFrameworks>` 元素（复数形式）来针对多个运行时。每个 TFM 由分号分隔：

```cs
    <TargetFrameworks>net8.0;net48</TargetFrameworks>
```

当您进行多目标时，编译器会为每个目标生成单独的输出程序集。

运行时目标通过 `TargetFramework` 属性编码到输出程序集中。一个程序集可以在比其目标更高的（但不是更低的）运行时上运行。

# .NET Standard

公共图书馆的丰富资源（位于 NuGet 上）如果只支持 .NET 8 将不会那么有价值。编写库时，通常需要支持多种平台和运行时版本。为了在不为每个运行时版本创建单独的构建（多目标）的情况下实现该目标，必须以最低公分母为目标。如果您的项目针对 .NET 6 (`net6.0`)，则您的库将在 .NET 6、.NET 7 和 .NET 8 上运行。

如果您还希望支持 .NET Framework（或 Xamarin 等旧版运行时），情况将变得更加混乱。原因是每个运行时都具有 CLR 和 BCL，其功能有重叠—没有一个运行时是其他运行时的纯子集。

*.NET Standard* 通过定义跨多个运行时工作的人工子集来解决此问题。通过针对 .NET Standard，您可以轻松编写具有广泛覆盖范围的库。

###### 注意

.NET Standard 不是运行时；它仅仅是描述一组运行时兼容性所需的最低基线功能（类型和成员）的规范。这个概念类似于 C# 接口：.NET Standard 就像一个接口，具体类型（运行时）可以实现它。

## .NET Standard 2.0

最有用的版本是 *.NET Standard 2.0*。一个针对 .NET Standard 2.0 而不是特定运行时的库将在现代 .NET（.NET 8/7/6/5，以及 .NET Core 2 至 4.6.1+）和 .NET Framework（4.6.1+）上无需修改即可运行。它还支持传统的 UWP（从 10.0.16299+ 开始）和 Mono 5.4+（老版本 Xamarin 使用的 CLR/BCL）。

要将目标设置为 .NET Standard 2.0，请将以下内容添加到您的 *.csproj* 文件中：

```cs
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  <PropertyGroup>
```

本书描述的大多数 API 都受 .NET Standard 2.0 的支持（那些不支持的大多数可以作为 NuGet 包提供）。

## 其他 .NET 标准

.NET Standard 2.1 是 .NET Standard 2.0 的超集，仅支持以下平台：

+   .NET Core 3+

+   Mono 6.4+

.NET Standard 2.1 不受任何 .NET Framework 版本支持，因此比 .NET Standard 2.0 要不实用得多。

还有旧版的 .NET Standard，如 1.1、1.2、1.3 和 1.6，其兼容性延伸到古老的运行时，如 .NET Core 1.0 或 .NET Framework 4.5。1.x 标准缺少成千上万的 API，在 2.0 中都有（包括本书中描述的大部分内容），因此已基本废弃。

## .NET Framework 和 .NET 8 的兼容性

由于 .NET Framework 存在时间很长，遇到仅适用于 .NET Framework（没有 .NET Standard、.NET Core 或 .NET 8 等同等版本）的库并不罕见。为了帮助缓解这种情况，.NET 5+ 和 .NET Core 项目被允许引用 .NET Framework 程序集，但必须注意以下几点：

+   如果 .NET Framework 程序集调用了不受支持的 API，则会抛出异常。

+   复杂的依赖关系可能（并经常）无法解析。

在实践中，它最有可能在简单情况下起作用，例如包装非托管 DLL 的程序集。

# 引用程序集

当您目标 .NET Standard 时，您的项目隐式引用了一个名为 *netstandard.dll* 的程序集，其中包含您选择版本的 .NET Standard 中允许的所有类型和成员。这称为 *引用程序集*，因为它仅存在于编译器的利益中，不包含任何编译代码。在运行时，通过程序集重定向属性来识别“真实”的程序集（选择的程序集取决于最终运行的运行时和平台）。

有趣的是，当您的目标是 .NET 8 时，您的项目隐式引用了一组引用程序集，其类型镜像了所选 .NET 版本的运行时程序集。这有助于版本控制和跨平台兼容性，并允许您将目标设置为与您的机器上安装的 .NET 版本不同的 .NET 版本。

# 运行时和 C#语言版本

默认情况下，项目的运行时目标决定使用的 C#语言版本：

| 运行时目标 | C# 版本 |
| --- | --- |
| .NET 8 | C# 12 |
| .NET 7 | C# 11 |
| .NET 6 | C# 10 |
| .NET 5 | C# 9 |
| .NET Core 3.x & 2.x | C# 8 |
| .NET Framework | C# 7.3 |
| .NET Standard 2.0 | C# 7.3 |

这是因为后续版本的 C#包含依赖于后续运行时中引入的类型的功能。

您可以通过`<LangVersion>`元素在项目文件中覆盖语言版本。在使用较旧的运行时（如.NET Framework）和较新的语言版本（如 C# 12）时，依赖于较新.NET 类型的语言特性将不起作用（尽管在某些情况下，您可以自定义这些类型，或从 NuGet 包导入）。

# CLR 和 BCL

索引器请注意：请跳过本节中的所有内容（除了一级标题）。本节的所有内容在书中的后续部分中有更详细的涵盖。

## 系统类型

最基本的类型直接位于`System`命名空间中。这些包括 C#内置类型；`Exception`基类；`Enum`、`Array`和`Delegate`基类；以及`Nullable`、`Type`、`DateTime`、`TimeSpan`和`Guid`。`System`命名空间还包括执行数学函数（`Math`）、生成随机数（`Random`）以及类型转换（`Convert`和`BitConverter`）的类型。

第六章还描述了这些类型，以及定义跨.NET 用于格式化（`IFormattable`）和顺序比较（`IComparable`）等任务的标准协议的接口。

`System`命名空间还定义了与垃圾回收器交互的`IDisposable`接口和`GC`类，我们在第十二章中进行了详细讨论。

## 文本处理

`System.Text`命名空间包含`StringBuilder`类（`string`的可编辑或*可变*版本）以及用于处理文本编码（如 UTF-8 的`Encoding`及其子类型）的类型。我们在第六章中进行了详细介绍。

`System.Text.RegularExpressions`命名空间包含执行高级基于模式的搜索和替换操作的类型；我们在第二十五章中描述了这些功能。

## 集合

.NET 提供了多种用于管理项集合的类。这些包括基于列表和字典的结构；它们与一组标准接口结合工作，统一它们的共同特征。所有集合类型均定义在以下命名空间中，我们在第七章中进行了涵盖：

```cs
System.Collections              // Nongeneric collections
System.Collections.Generic      // Generic collections
System.Collections.Frozen       // High-performance read-only collections
System.Collections.Immutable    // General-purpose read-only collections
System.Collections.Specialized  // Strongly typed collections
System.Collections.ObjectModel  // Bases for your own collections
System.Collections.Concurrent   // Thread-safe collection (Chapter 22)
```

## 查询

语言集成查询（LINQ）允许您在本地和远程集合（例如 SQL Server 表）上执行类型安全的查询，详细介绍见第八章、9 章和 10 章。LINQ 的一个重要优势是它在各种领域提供了一致的查询 API。关键类型位于以下命名空间中：

```cs
System.Linq                  // LINQ to Objects and PLINQ
System.Linq.Expressions      // For building expressions manually
System.Xml.Linq              // LINQ to XML
```

## XML 和 JSON

XML 和 JSON 在.NET 中得到了广泛支持。第十章专注于 LINQ to XML——一种轻量级的 XML 文档对象模型（DOM），可以通过 LINQ 构建和查询。第十一章涵盖了高性能的低级 XML 读取器/写入器类、XML 模式和样式表，以及处理 JSON 的类型：

```cs
System.Xml                // XmlReader, XmlWriter
System.Xml.Linq           // The LINQ to XML DOM
System.Xml.Schema         // Support for XSD
System.Xml.Serialization  // Declarative XML serialization for .NET types
System.Xml.XPath          // XPath query language
System.Xml.Xsl            // Stylesheet support

System.Text.Json          // JSON reader/writer and DOM
System.Text.Json.Nodes    // JsonNode API (DOM)
```

在在线补充材料中[*http://www.albahari.com/nutshell*](http://www.albahari.com/nutshell)，我们讨论了 JSON 序列化器。

## 诊断

在第十三章中，我们涵盖了日志记录和断言，并描述了如何与其他进程交互，写入 Windows 事件日志以及处理性能监控。相关类型在`System.Diagnostics`中定义和使用。

## 并发与异步

许多现代应用程序需要同时处理多个事务。自 C# 5.0 以来，通过异步函数和任务等高级结构变得更加容易。第十四章详细解释了这些内容，从多线程基础开始讲起。与线程和异步操作相关的类型位于`System.Threading`和`System.Threading.Tasks`命名空间中。

## 流和输入/输出

.NET 提供了基于流的低级输入/输出（I/O）模型。流通常用于直接读写文件和网络连接，并且可以通过链式或包装流添加压缩或加密功能。第十五章描述了流架构以及特定支持文件和目录处理、压缩、管道和内存映射文件的 I/O。`Stream`和 I/O 类型在`System.IO`命名空间中定义。

## 网络

通过`System.Net`中的类型，您可以直接访问大多数标准网络协议，如 HTTP、TCP/IP 和 SMTP。在第十六章中，我们展示了如何使用每个协议进行通信，从简单的任务如从网页下载开始，到直接使用 TCP/IP 检索 POP3 电子邮件为止。以下是我们涵盖的命名空间：

```cs
System.Net
System.Net.Http          // HttpClient
System.Net.Mail          // For sending mail via SMTP
System.Net.Sockets       // TCP, UDP, and IP
```

## 程序集、反射和属性

C#程序编译生成的程序集包含可执行指令（存储为 IL）和元数据，描述程序的类型、成员和属性。通过反射，您可以在运行时检查这些元数据，并动态调用方法。使用`Reflection.Emit`，您可以动态生成新的代码。

在第十七章中，我们描述了程序集的组成及如何动态加载和隔离它们。在第十八章中，我们涵盖了反射和属性，描述了如何检查元数据、动态调用函数、编写自定义属性、生成新类型以及解析原始 IL。用于反射和处理程序集的类型位于以下命名空间中：

```cs
System
System.Reflection
System.Reflection.Emit
```

## 动态规划

在第十九章中，我们研究了一些动态编程模式，并利用了动态语言运行时（DLR）。我们描述了如何实现*Visitor*模式、编写自定义动态对象以及与 IronPython 进行互操作。动态编程的类型位于`System.Dynamic`中。

## 密码学

.NET 为流行的哈希和加密协议提供了广泛的支持。在第二十章，我们涵盖了哈希、对称和公钥加密以及 Windows 数据保护 API。这些类型的定义如下：

```cs
System.Security
System.Security.Cryptography
```

## 高级线程

C#的异步函数显著简化了并发编程，因为它们减少了对低级技术的需求。然而，在某些情况下，仍然需要信号构造、线程本地存储、读者/写者锁等。第二十一章深入解释了这些内容。线程类型位于`System.Threading`命名空间中。

## 并行编程

在第二十二章，我们详细介绍了利用多核处理器的库和类型，包括任务并行性的 API、命令式数据并行性以及函数并行性（PLINQ）。

## `Span<T>`和`Memory<T>`

为了帮助微调性能热点，CLR 提供了许多类型，帮助您以减少内存管理器负担的方式编程。其中两个关键类型是`Span<T>`和`Memory<T>`，我们在第二十三章中进行了介绍。

## 本地和 COM 互操作性

您可以与本地代码和组件对象模型（COM）代码进行互操作。本地互操作性允许您调用未管理的 DLL 中的函数，注册回调，映射数据结构，并与本地数据类型进行互操作。COM 互操作性允许您调用 COM 类型（在 Windows 机器上）并将.NET 类型暴露给 COM。支持这些功能的类型位于`System.Runtime.InteropServices`中，我们在第二十四章中进行了详细介绍。

## 正则表达式

在第二十五章，我们讨论了如何使用正则表达式在字符串中匹配字符模式。

## 序列化

.NET 提供了多个系统，用于将对象保存和恢复为二进制或文本表示。这些系统可用于通信以及将对象保存和恢复为文件。在在线补充资料[*http://www.albahari.com/nutshell*](http://www.albahari.com/nutshell)，我们涵盖了所有四个序列化引擎：二进制序列化器，（新更新的）JSON 序列化器，XML 序列化器和数据合同序列化器。

## Roslyn 编译器

C# 编译器本身是用 C# 编写的 —— 这个项目称为“Roslyn”，并且这些库可以作为 NuGet 包提供。利用这些库，您可以以多种方式使用编译器的功能，例如编写代码分析和重构工具。我们在在线补充资料中介绍了 Roslyn，网址为[*http://www.albahari.com/nutshell*](http://www.albahari.com/nutshell)。

# 应用程序层

用户界面（UI）应用程序可以分为两类：*轻客户端*，相当于网站，和*丰富客户端*，用户必须在计算机或移动设备上下载和安装的程序。

要在 C# 中编写轻客户端应用程序，可以使用 ASP.NET Core，在 Windows、Linux 和 macOS 上运行。ASP.NET Core 还专为编写 Web API 而设计。

对于丰富客户端应用程序，有多种 API 可供选择：

+   Windows 桌面层包括流行的 WPF 和 Windows Forms API，在 Windows 7/8/10/11 桌面上运行。

+   WinUI 3（Windows App SDK）是 UWP 的继任者，仅在 Windows 10+ 桌面上运行。

+   UWP 允许您编写运行在 Windows 10+ 桌面以及 Xbox 或 HoloLens 等设备上的 Windows Store 应用程序。

+   MAUI（前身为 Xamarin）运行在 iOS 和 Android 移动设备上。MAUI 还允许跨平台桌面应用程序，目标是 macOS（通过 Catalyst）和 Windows（通过 Windows App SDK）。

还有第三方跨平台 UI 库，如 Avalonia。与 MAUI 不同，Avalonia 还可以在 Linux 上运行，并且不依赖于 Catalyst/WinUI 间接层用于桌面平台，简化了开发和调试。

## ASP.NET Core

ASP.NET Core 是 ASP.NET 的轻量模块化后继者，适用于创建网站、基于 REST 的 Web API 和微服务。它还可以与两个流行的单页应用程序框架一起运行：React 和 Angular。

ASP.NET 支持流行的*模型-视图-控制器*（MVC）模式，以及一种名为 Blazor 的新技术，在其中客户端代码用 C# 编写而不是 JavaScript。

ASP.NET Core 在 Windows、Linux 和 macOS 上运行，并且可以自托管在自定义进程中。与其 .NET Framework 前身（ASP.NET）不同，ASP.NET Core 不依赖于 `System.Web` 和 Web Forms 的历史包袱。

与任何轻客户端架构一样，ASP.NET Core 相对于丰富客户端提供以下一般优势：

+   在客户端端没有零部署。

+   客户端可以在支持 Web 浏览器的任何平台上运行。

+   更新可以轻松部署。

## Windows 桌面

Windows 桌面应用层提供了两种 UI API 选择，用于编写丰富客户端应用程序：WPF 和 Windows Forms。这两种 API 都可以在 Windows 桌面/服务器 7 到 11 上运行。

### WPF

WPF 于 2006 年推出，并且自那时以来一直在增强。与其前身 Windows Forms 不同，WPF 明确使用 DirectX 渲染控件，带来以下好处：

+   它支持复杂的图形，如任意变换、3D 渲染、多媒体和真正的透明度。通过样式和模板支持皮肤定制。

+   其主要测量单位不是基于像素的，因此应用程序可以在任何 DPI 设置下正确显示。

+   它具有广泛和灵活的布局支持，这意味着可以本地化应用程序而无需担心元素重叠的问题。

+   其使用 DirectX 使得渲染快速，并能利用图形硬件加速。

+   它提供可靠的数据绑定。

+   可以在 XAML 文件中声明性地描述 UI，这些文件可以独立于“代码后台”文件进行维护，有助于将外观与功能分离。

由于其规模和复杂性，学习 WPF 需要一些时间。用于编写 WPF 应用程序的类型位于`System.Windows`命名空间及其所有子命名空间中，除了`System.Windows.Forms`。

### Windows Forms

Windows Forms 是一个富客户端 API，随着 2000 年第一个.NET Framework 版本一同发布。与 WPF 相比，Windows Forms 是一种相对简单的技术，提供了编写典型 Windows 应用程序所需的大多数功能。它在维护遗留应用程序方面也具有重要意义。但与 WPF 相比，它有许多缺点，大部分源于其作为 GDI+和 Win32 控件库包装器的特性：

+   虽然 Windows Forms 提供了 DPI 感知机制，但仍然很容易编写在客户端 DPI 设置与开发者不同的应用程序，从而导致应用程序崩溃。

+   用于绘制非标准控件的 API 是 GDI+，虽然相对灵活，但在渲染大面积时速度较慢（并且没有双缓冲，可能会闪烁）。

+   控件缺乏真正的透明度。

+   大多数控件都是非组合的。例如，您不能将图像控件放在选项卡控件标题中。在 Windows Forms 中以一种对 WPF 来说微不足道的方式自定义列表视图、组合框和选项卡控件是耗时且痛苦的。

+   动态布局很难正确可靠地实现。

最后一点是支持 WPF 而不是 Windows Forms 的绝佳理由，即使你只编写需要 UI 而不是“用户体验”的业务应用程序。WPF 中的布局元素，如 `Grid`，使得组合标签和文本框变得简单，即使在语言更改后的本地化中，它们始终对齐，无需混乱的逻辑和任何闪烁。此外，你不需要迎合屏幕分辨率的最低公分母——WPF 布局元素从一开始就被设计为能够正确调整大小。

正面看，Windows Forms 相对简单易学且仍有大量第三方控件。

Windows Forms 类型位于 `System.Windows.Forms`（在 *System.Windows.Forms.dll* 中）和 `System.Drawing`（在 *System.Drawing.dll* 中）命名空间中。后者还包含用于绘制自定义控件的 GDI+ 类型。

## UWP 和 WinUI 3

UWP 是为编写面向触摸优先的 UI 的富客户端 API，目标是 Windows 10+ 桌面和设备。词汇“Universal”指其能够在一系列 Windows 10 设备上运行，包括 Xbox、Surface Hub、HoloLens 和（当时）Windows Phone。

UWP API 使用 XAML 并且在某种程度上类似于 WPF。以下是其主要区别：

+   UWP 应用程序的主要分发模式是 Windows Store。

+   UWP 应用程序在沙盒中运行，以减少恶意软件的威胁，这意味着它们不能执行诸如读取或写入任意文件之类的任务，并且不能以管理员权限运行。

+   UWP 依赖于操作系统（Windows）中的 WinRT 类型，而不是托管运行时。这意味着在编写应用程序时，你必须指定一个 Windows *版本范围*（例如 Windows 10 版本 17763 到 Windows 10 版本 18362）。这意味着你必须要么针对旧的 API，要么要求客户安装最新的 Windows 更新。

由于这些差异所带来的限制，UWP 从未成功地匹配 WPF 和 Windows Forms 的流行度。为了解决这个问题，Microsoft 将 UWP 转变为一种名为 Windows App SDK 的新技术（具有称为 WinUI 3 的 UI 层）。

Windows App SDK 将 WinRT API 从操作系统传输到运行时，从而暴露出完全托管的接口，并且不需要针对特定操作系统版本范围进行目标设定。它还执行以下操作：

+   与 Windows 桌面 API（Windows Forms 和 WPF）更好地集成

+   允许你编写在 Windows Store 沙箱之外运行的应用程序

+   运行在最新的 .NET 之上（而不是像 UWP 那样绑定于 .NET Core 2.2）

尽管有这些改进，WinUI 3 在经典 Windows 桌面 API 中并没有获得广泛的流行。截至撰写本文时，Windows App SDK 也不支持 Xbox 或 HoloLens，并且需要单独的最终用户下载。

## MAUI

MAUI（前身为 Xamarin）允许您使用 C#开发针对 iOS 和 Android 的移动应用（以及通过 Catalyst 和 Windows App SDK 针对 macOS 和 Windows 的跨平台桌面应用）。

运行在 iOS 和 Android 上的 CLR/BCL 被称为 Mono（开源 Mono 运行时的一个派生）。历史上，Mono 与.NET 并不完全兼容，能在 Mono 和.NET 上运行的库通常会选择目标为.NET Standard。然而，从.NET 6 开始，Mono 的公共接口与.NET 合并，使得 Mono 实际上成为了.NET 的一个*实现*。

MAUI 包括统一的项目接口、热重载功能，以及对 Blazor 桌面和混合应用的支持。详细信息请参阅[*https://github.com/dotnet/maui*](https://github.com/dotnet/maui)。
