# 附录。Learning Blazor 应用程序项目

在本书中，我们研究了 Learning Blazor，这是为本书创建的一个应用程序。该应用程序由几个项目组成，这些项目作为独立的功能部分。架构在“浏览“Learning Blazor”示例应用程序”中进行了讨论。源代码可以在[GitHub](https://oreil.ly/learning-blazor-code)上找到。

*learning-blazor.sln*解决方案文件包含了几个项目，这些项目共同构成了整个应用程序作为一个统一的单元。虽然解决方案中的每个项目负责其核心功能，但协调具有不同功能的项目以形成一个统一的整体是任何成功应用程序的要求。以下部分列出了解决方案中的主要项目，并提供了关于它们的主题详细信息。

# Web 客户端

客户端应用程序，简称为 Web.Client，是一个 Blazor WebAssembly 项目，目标是使用`Microsoft.NET.Sdk.BlazorWebAssembly`软件开发工具包（SDK）。该 Web 项目负责所有用户交互和体验。通过页面、客户端路由、表单验证、模型绑定和基于组件的用户界面，Web.Client 项目展示了 Blazor 的大部分主要特性。该应用程序定义了一个`Learning.Blazor`命名空间。

# Web API

如果没有数据，客户端应用程序会变得相当无聊。你可能会问，Web 应用程序如何获取数据？HTTP 是最常见的方法，但除此之外，我们的应用程序还将使用 ASP.NET Core SignalR 和 Web Sockets 来实现实时网络功能。

###### 提示

ASP.NET Core SignalR 是一个开源库，简化了向应用程序添加实时网络功能的过程。它在示例源代码中用于展示实时功能。有关 SignalR 的概述，请参阅 Microsoft 的[ASP.NET Core SignalR 概述](https://oreil.ly/TrV3W)。

同样，示例应用程序使用 Blazor WebAssembly 托管模型，但展示实时网络功能仍然非常有价值。因此，ASP.NET Core SignalR 被使用，但使用方式与以前在 Blazor Server 托管模型中描述的方式不同。

有一个名为 Web.Api 的 ASP.NET Core Web API 项目，目标是使用`Microsoft.NET.Sdk.Web`。该项目将提供客户端应用程序依赖的各种端点。API 和 SignalR 端点将由 Azure Active Directory（Azure AD）的业务对消费者（B2C）认证保护。

Web API 项目使用内存缓存来确保响应迅速的体验。某些端点依赖于服务，这些服务会从缓存或原始的 HTTP 依赖端点中确定性地获取数据。

# Pwned Web API

Pwned Web API 项目还依赖于`Microsoft.NET.Sdk.Web` SDK。该项目从 Troy Hunt 提供的“Have I Been Pwned”服务中公开功能。用户在同意允许应用程序使用其电子邮件地址后，将其发送到 Pwned 服务。API 提供的详细信息用于通知用户其电子邮件是否曾经参与数据泄露。

# Web Abstractions

使用一个简单的 C#类库项目，目标为`Microsoft.NET.Sdk`，Web.Abstractions 项目定义了一些在客户端和服务器应用程序之间共享的抽象。这些契约将作为 SignalR 端点的粘合剂。从客户端的角度来看，这些抽象将提供一个可发现的 API 集合，客户端可以订阅事件和方法，并通过这些方法与服务器进行通信。从服务器的角度来看，这些抽象将巩固方法和事件名称，确保没有任何可能的不对齐。这非常重要，并且是所有基于 JavaScript 的单页面应用程序开发中的一个常见陷阱。

# Web Extensions

在现代 C#应用程序开发中，将重复的子程序封装为扩展是很常见的。由于它们的重复性质，实用的扩展方法是共享类库风格项目的完美候选者。在我们的情况下，我们将使用目标为`Microsoft.NET.Sdk`的 Web.Extensions 项目。该项目提供的功能将在我们解决方案中的大多数其他项目中使用，特别是客户端和服务器应用程序场景。

# Web HTTP Extensions

另一个扩展类库专注于定义`HttpClient`类型的默认值。有几个共享的类库，它们都在进行 HTTP 调用—​我希望所有失败的 HTTP 调用都有一个特定的重试策略来处理瞬态错误。这些策略在目标为`Microsoft.NET.Sdk`的 Web.Http.Extensions 项目中定义。

# Web Functions

在过去的十年中，无服务器编程已经变得非常普遍。不可变基础设施、弹性和可扩展性始终是非常受欢迎的特性。Azure Functions 用于封装我的天气服务。我决定使用 Open Weather Map API，它是免费的，支持多种语言，并且相当准确。通过 Azure Function 应用程序，我可以封装我的配置，保护我的 API 密钥，使用依赖注入，并将调用委托给天气 API。该项目名为 Web.Functions，目标为`Microsoft.NET.Sdk`。

# Web Joke Services

生活太短暂，我们需要更多的笑声，微笑，不要那么严肃地对待自己。Web.JokeServices 库负责按照伪随机的时间表聚合笑话。在此项目中，有三个独立且免费的笑话 API 被聚合起来：

+   [互联网查克·诺里斯数据库](https://oreil.ly/Dmf7N)

+   [我可以有爸爸笑话](https://oreil.ly/LMitC)

+   [随机编程笑话 API](https://oreil.ly/U67QS)

# Web Models

Web.Models 项目是解决方案中许多其他项目共享的库。它包含用于表示各种领域实体的所有模型，例如服务和客户端共享的模型。任何与应用程序交互的内容都会指定一个形状，并具有帮助唯一标识自身的成员。这当然是面向对象编程的核心。

# Web Twitter 组件

为了举例说明组件库的功能，我选择创建了一个名为 Web.TwitterComponents 的 Twitter 组件 Razor 库。这个项目依赖于`Microsoft.NET.Sdk.Razor` SDK。它提供了两个组件，一个代表一条推文，另一个代表推文集合。这个项目将演示组件如何被模板化；展示了父子层次关系。它展示了组件如何使用 JavaScript 互操作性并从异步事件中更新。

# Web Twitter 服务

Web.Twit​terServices 项目被 Web.Api 项目所消费，而不是 Web.TwitterComponents 项目。Twitter 服务在后台服务的上下文中使用。后台服务提供了一种管理长时间运行操作的手段，其功能超出了请求和响应管道。就像推文流处理那样，当实时过滤的推文发生时，我们的服务将相应地传播它们。
