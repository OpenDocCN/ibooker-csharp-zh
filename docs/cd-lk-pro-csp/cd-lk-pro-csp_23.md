# 附录 C 安装指南

本附录包含以下内容的快速安装指南：

+   .NET Framework 4.x

+   .NET 5

+   Visual Studio

+   Visual Studio for Mac

+   Visual Studio Code

.NET Framework 4.x（仅限 Windows）

.NET Framework 仅在 Windows 上受支持。要安装 .NET Framework 4 的最新版本，请访问 [`dotnet.microsoft.com/download/dotnet-framework`](https://dotnet.microsoft.com/download/dotnet-framework)，并从发布列表中选择顶部选项。请注意，你需要运行至少 .NET Framework 4.8 才能支持本书提供的源代码。当你点击一个发布版时，你将被带到该发布版的下载链接页面。点击“下载 .NET Framework 4.[版本] 开发者包”。这将下载一个安装程序，你可以运行它来在你的机器上安装 .NET Framework。

.NET 5 (Windows, Linux 和 macOS)

.NET 5 支持 Windows、Linux 和 macOS（仅限 64 位）。要安装 .NET 5 的最新版本，请导航至 [`dotnet.microsoft.com/download/dotnet/5.0`](https://dotnet.microsoft.com/download/dotnet/5.0)，并选择最新的 SDK 发布版。在撰写本文时，最新发布版是 SDK 5.0.203。当你点击适当的发布版时，你将被引导到相应发布版的下载页面。如果你希望这样做，还有一个选项可以下载所有平台的二进制文件。运行下载的安装程序将在你的平台上安装 .NET 5。

Visual Studio (Windows)

Visual Studio 是 Windows 上开发 C# 的首选 IDE。Visual Studio 有以下三种版本：

+   社区版

+   专业版

+   企业版

社区版是免费的，允许你开发（商业）软件，除非你是拥有超过 250 台电脑或年收入超过 100 万美元的组织（并为该组织工作）。三个版本之间有一些功能差异，但本书中我们做的所有事情都可以使用 Visual Studio 社区版完成。

要下载 Visual Studio Community，请访问 [`visualstudio.microsoft.com/vs/`](https://visualstudio.microsoft.com/vs/)，并在“下载 Visual Studio”下拉列表中选择 Visual Studio Community。请确保版本至少为 Visual Studio 2019 v16.7，因为 .NET 5 在旧版本的 Visual Studio（包括前一年的版本，如 Visual Studio 2017）上无法正确运行。当你启动安装程序时，你会看到一系列 Visual Studio 安装选项。这些选项被称为“工作负载”，对于本书，你需要安装以下这些：

+   ASP.NET 和 Web 开发

+   .NET Core 跨平台开发

通过选择工作负载，右下角将启用一个下载按钮。点击此按钮，Visual Studio 将安装所选的工作负载。请注意，Visual Studio 通常需要超过 8 GB 的空间来安装。

Visual Studio for Mac

Visual Studio for Mac 是 Visual Studio 的一个独立产品。它是微软将 Visual Studio 体验带到 macOS 的一次尝试。要安装 Visual Studio for Mac，请访问[`visualstudio.microsoft.com/vs/mac/`](https://visualstudio.microsoft.com/vs/mac/)。点击下载 Visual Studio for Mac 按钮，并运行下载的安装程序。现在您可以使用 Visual Studio for Mac 了。在 macOS 上还可以考虑的其他 IDE 有 VS Code 和 JetBrains 的 Rider。请确保您始终使用最新版本的 Visual Studio for Mac。

Visual Studio Code (Windows, Linux, macOS)

您不需要使用 Visual Studio 来阅读这本书或进行日常工作。从理论上讲，您可以使用任何支持 C#的编辑器并通过命令行进行编译。在实践中，这可能会有些痛苦。微软在 Visual Studio Code 中开发了一个轻量级的 Visual Studio 替代品。它是免费的，并且更像是一个文本编辑器而不是一个完整的 IDE。要下载 Visual Studio Code，请访问[`code.visualstudio.com/`](https://code.visualstudio.com/)，并点击[平台]的下载按钮。运行安装程序，然后 Visual Studio Code 就准备好使用了。当您在 Visual Studio Code 中第一次编写 C#代码（或打开 C#解决方案）时，它将提示您下载 C#包。接受此提示，您将能够使用 Visual Studio Code（或 VS Code）进行 C#开发。

在您的本地机器上运行飞剪航空公司数据库

如果您不想（或不能）在书中使用飞剪航空公司项目的部署数据库，您可以在本地 SQL Server 实例中运行 SQL 数据库。

要这样做，您需要安装以下软件：

+   SQL Server Developer Edition

+   Microsoft SQL Server Management Studio

要下载 SQL Server，请访问[`www.microsoft.com/en-us/sql-server/sql-server-downloads`](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)，并下载 SQL Server 的开发者版本。安装 SQL Server 后，您可以继续从[`docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15`](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15)安装 SQL Server Management Studio (SSMS)。SQL Server Developer 和 SSMS 都是免费的。

在安装 SSMS 之后，如果您还没有这样做，请通过 SQL Server 创建一个新的本地 SQL Server 实例。要安装一个新的本地 SQL Server 实例，打开与您的 SQL Server Developer Edition 应用程序一起安装的 SQL Server 安装中心。在安装中心中，点击安装 > 新建 SQL Server 独立安装或向现有安装添加功能。

按照安装向导进行操作，注意您为 SQL Server 实例提供的登录凭证。您需要这些信息来连接到 SQL Server 实例。

现在启动 SSMS。你会看到一个连接对话框，你可以在这里浏览你的 SQL 实例并填写你的连接信息。连接后，你会看到 SSMS 的主屏幕。在这里，在对象资源管理器中，右键单击数据库，然后选择导入数据层应用程序。

点击导入数据层应用程序的上下文菜单选项会弹出一个向导，允许你导入提供的飞剪航空公司数据库。在导入设置窗口中，选择从本地磁盘导入，并浏览到数据库文件（FlyingDutchmanAirlinesDatabase.bacpac）。在以下窗口中，如果你愿意，可以重命名数据库。完成向导后，数据库应该已导入并准备好与本书中的代码一起使用。

有可能你的数据库导入会失败。这通常是因为 Microsoft Azure 中包含的数据库设置与 SSMS 不匹配。如果导入失败，请在你的 SQL 实例（为你自动生成）的主要数据库上运行以下命令：

```
sp_configure 'contained database authentication', 1; GO RECONFIGURE; GO;
```

有些人报告说，围绕 GO 关键字的语法问题有时会出现。如果你遇到了上一个命令的问题，这个下一个命令应该对你有效：

```
EXEC sp_configure 'contained database authentication', 1; RECONFIGURE;
```

最后一点注意事项：无论何时你在书中遇到连接字符串，请确保将其替换为你的本地 SQL 实例数据库的正确连接字符串。
