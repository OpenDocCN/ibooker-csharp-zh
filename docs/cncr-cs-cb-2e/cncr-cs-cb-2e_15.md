# 附录 A. Legacy 平台支持

本书讨论的许多技术也对旧版平台有一定的支持。如果您不得不支持这些平台，本附录中的信息可能帮助您确定可用的技术。在旧版平台上使用这些技术并非理想；即使您能让它们运行，也要记住唯一的长期解决方案是更新代码的平台目标。本附录主要作为历史参考，而非推荐；尽管如此，旧代码的维护者可能会发现它有用。

Table A-1 总结了不同技术在 legacy 平台上的支持情况。

Table A-1\. Legacy 平台支持

| 平台 | async | Parallel | Reactive | Dataflow | Concurrent collections | Immutable collections |
| --- | --- | --- | --- | --- | --- | --- |
| .NET 4.5 | ✓ | ✓ | NuGet | NuGet | ✓ | NuGet |
| .NET 4.0 | NuGet | ✓ | NuGet | ✗ | ✓ | ✗ |
| Windows Phone Apps 8.1 | ✓ | ✓ | NuGet | NuGet | ✓ | NuGet |
| Windows Phone SL 8.0 | ✓ | ✗ | NuGet | NuGet | ✗ | NuGet |
| Windows Phone SL 7.1 | NuGet | ✗ | NuGet | ✗ | ✗ | ✗ |
| Silverlight 5 | NuGet | ✗ | NuGet | ✗ | ✗ | ✗ |

# Legacy 平台支持 Async

如果您需要在旧的 legacy 平台上支持 `async`，请安装 [`Microsoft.Bcl.Async`](http://bit.ly/micro-async) 的 NuGet 包。

###### 警告

不要使用 `Microsoft.Bcl.Async` 在运行于 .NET 4.0 的 ASP.NET 上启用 `async` 代码！.NET 4.5 中已更新 ASP.NET 管道以支持 `async`，您必须使用 .NET 4.5 或更新版本进行 `async` ASP.NET 项目。`Microsoft.Bcl.Async` 仅适用于非 ASP.NET 应用程序。

Table A-2\. Async 的 Legacy 平台支持

| 平台 | Async 支持 |
| --- | --- |
| .NET 4.5 | ✓ |
| .NET 4.0 | NuGet: `Microsoft.Bcl.Async` |
| Windows Phone Apps 8.1 | ✓ |
| Windows Phone SL 8.0 | ✓ |
| Windows Phone 7.1 | NuGet: `Microsoft.Bcl.Async` |
| Silverlight 5 | NuGet: `Microsoft.Bcl.Async` |

使用 `Microsoft.Bcl.Async` 时，现代 `Task` 类型的许多成员位于 `TaskEx` 类型上，包括 `Delay`、`FromResult`、`WhenAll` 和 `WhenAny`。

# Legacy 平台支持 Dataflow

要使用 TPL Dataflow，请将 NuGet 包 [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) 安装到您的应用程序中。TPL Dataflow 库对较旧的平台的支持有限（Table A-3）。

###### 警告

不要使用旧版的 `Microsoft.Tpl.Dataflow` 包。它已不再维护。

Table A-3\. TPL Dataflow 的 Legacy 平台支持

| 平台 | Dataflow 支持 |
| --- | --- |
| .NET 4.5 | NuGet: `System.Threading.Tasks.Dataflow` |
| .NET 4.0 | ✗ |
| Windows Phone Apps 8.1 | NuGet: `System.Threading.Tasks.Dataflow` |
| Windows Phone SL 8.0 | NuGet: `System.Threading.Tasks.Dataflow` |
| Windows Phone SL 7.1 | ✗ |
| Silverlight 5 | ✗ |

# Legacy 平台支持 System.Reactive

若要使用 System.Reactive，请在你的应用程序中安装 NuGet 包[`System.Reactive`](http://bit.ly/sys-reactive)。System.Reactive 一直以来都具有广泛的平台支持（表格 A-4）；然而，大多数旧平台已不再受支持：

表格 A-4\. System.Reactive 的旧平台支持

| 平台 | 响应式支持 |
| --- | --- |
| .NET 4.7.2 | NuGet: `System.Reactive` |
| .NET 4.5 | NuGet: `System.Reactive v3.x` |
| .NET 4.0 | NuGet: `Rx.Main` |
| Windows Phone Apps 8.1 | NuGet: `System.Reactive v3.x` |
| Windows Phone SL 8.0 | NuGet: `System.Reactive v3.x` |
| Windows Phone SL 7.1 | NuGet: `Rx.Main` |
| Silverlight 5 | NuGet: `Rx.Main` |

###### 警告

旧的 `Rx.Main` 包已不再维护。
