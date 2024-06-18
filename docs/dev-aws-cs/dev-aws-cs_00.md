# 前言

如果你必须选择过去 20 年中出现的两种流行技术进行重点关注，那么很难挑选出比 C#和 AWS 更好的选择。Anders Hejlsberg 于 2000 年为微软设计了 C#，几年后，.NET 框架和 Visual Studio 推出。接下来，在 2004 年，跨平台的 C#编译器和运行时环境 Mono 广泛可用。这个生态系统已经实现了包括跨平台移动框架 Xamarin 在内的独特平台和技术。

在 2000 年代初的另一个宇宙中，亚马逊转向了面向服务的架构，并在 2002 年发布了公共 Web 服务。随着服务的流行，亚马逊在 2004 年发布了 SQS，2006 年发布了 Amazon S3 和 EC2。这些存储、计算和消息传递服务仍然是亚马逊 Web 服务(AWS)的核心构建模块。截至 2022 年，AWS 至少拥有 27 个地理区域、87 个可用区、数百万台服务器和数百种服务。这些服务包括从机器学习（SageMaker）到无服务器计算（Lambda）的各种产品。本书展示了利用这两个卓越技术堆栈的可能性。你将学习云计算的理论以及如何使用 AWS 实现它。你还将使用 AWS 技术构建许多解决方案，从无服务器到利用.NET 生态系统实现的 DevOps，最终实现许多优雅的解决方案。

# 适合读者

这本书适合希望在 AWS 上探索可能性的 C#开发者。这本书也可以被看作是 AWS 和云计算的简介。我们将从零开始介绍许多云计算话题，所以如果这是你第一次涉足基于云的软件开发，那么放心，我们将提供所有你需要的背景信息。

虽然 C# *技术上* 只是一种编程语言而不是框架，我们将几乎完全关注于构建 Web 应用程序的.NET 应用程序开发人员。会稍微提到 Blazor 和 Xamarin，但对这些的了解绝对不是必需的。我们还有内容适用于熟悉旧版.NET 框架和新版.NET（以前是*.NET Core*）的开发人员。

# 书籍组织方式

AWS 提供超过 200 种服务来构建各种基于云的应用程序，我们将本书组织成为覆盖对 C#开发者最重要的服务。各章节结构如下。

第一章，“在 AWS 上开始使用.NET”，作为 AWS 的介绍以及如何与服务进行交互和配置。

第二章，“AWS 核心服务”，详细讨论了存储和计算，这两个最广泛使用的服务对于习惯于部署到 Web 服务器的.NET 应用程序开发人员来说也会感觉最为熟悉。

第三章，“将传统 .NET 框架应用迁移到 AWS”，作为你可以迅速将现有代码库从现有物理或虚拟机或其他云提供商迁移到 AWS 的简要概述。

第四章，“现代化 .NET 应用程序为无服务器”，比迁移现有应用程序更进一步，探讨了如何将你的代码架构现代化为无服务器模型，并提供 AWS 上的示例。

第五章，“.NET 容器化”，关于容器的一切。如果你已经在为你的 .NET Web 应用使用 Docker 容器，这一章将帮助你快速将它们部署到 AWS 的托管容器服务中。

第六章，“DevOps”，你知道 AWS 有一个完全托管的持续交付服务叫做 CodePipeline 吗？本章将向你展示如何使用 AWS 提供的工具简化应用程序通向生产的路径。

第七章，“.NET 的日志记录、监控和仪表化”，继续从 DevOps 中展示你可以集成到应用程序中的日志记录和监控功能。我们还将看看如何直接从你的 C# 代码手动推送性能指标。

第八章，“使用 AWS C# SDK 进行开发”，这本书的最后一章，深入介绍了允许你将 AWS 服务集成到你的代码中的 C# SDK 工具。

最后，在书的后面有两个附录可能会对你有帮助，它们没有清晰地适应任何章节。附录 A，“基准测试 AWS”，详细介绍了如何对 AWS 机器进行基准测试，以及附录 B，“开始使用 .NET”，展示了一个简短的 C# 和 F# GitHub Codespaces 教程。

此外，每一章还包括批判性思维问题和练习。批判性思维问题是团队讨论、用户组或准备工作面试或认证的良好起点。练习通过实践学习并将章节内容应用到工作代码示例中。这些示例可能成为出色的作品集件。

# 附加资源

对于那些可以访问 O’Reilly 平台的人来说，一个可能对你 AWS 旅程有帮助的可选资源是 [52 周 AWS-完整系列](https://oreil.ly/kDf0Q)；它包含了所有重要认证的数小时实操视频。这个系列也可以作为免费播客在所有 [主要播客渠道](https://podcast.paiml.com) 上获得。

除了这些资源外，Noah Gift 每天和每周都在以下位置更新 AWS：

Noah Gift O’Reilly

在 [Noah Gift 的 O’Reilly 个人资料页面](https://www.oreilly.com/pub/au/3039)，有数百个视频和课程，涵盖多种技术，包括 AWS，并经常进行在线培训。另外两本涵盖 AWS 的 O’Reilly 书籍可能也会有所帮助：[*Python for DevOps*](https://www.oreilly.com/library/view/python-for-devops/9781492057680) 和 [*Practical MLOps*](https://www.oreilly.com/library/view/practical-mlops/9781098103002)。

Noah Gift Linkedin

在 [Noah Gift 的 LinkedIn 页面](https://www.linkedin.com/in/noahgift)，他定期直播 AWS 培训和正在进行的 O’Reilly 书籍笔记。

Noah Gift 网站

[*Noahgift.com*](https://noahgift.com) 是获取最新课程、文章和演讲的最佳途径。

Noah Gift GitHub

在 [Noah Gift 的 GitHub 个人资料页面](https://github.com/noahgift)，您可以找到数百个仓库和几乎每日更新。

Noah Gift AWS Hero 个人资料

在 [Noah Gift 的 AWS Hero 个人资料页面](https://oreil.ly/FyFvf)，您可以找到涉及 AWS 平台工作的当前资源链接。

Noah Gift Coursera 个人资料页面

在 [Noah Gift 的 Coursera 个人资料页面](https://www.coursera.org/instructor/noahgift)，您可以找到涵盖广泛的云计算主题，重点是 AWS 的多个专项课程。这些课程包括：

+   [云计算基础](https://oreil.ly/XcxeC)

+   [云虚拟化、容器和 APIs](https://oreil.ly/mcacu)

+   [云数据工程](https://oreil.ly/pVHGm)

+   [云机器学习工程和 MLOps](https://oreil.ly/724Sz)

+   [Linux 和 Bash 用于数据工程](https://oreil.ly/Ps1Mz)

+   [Python 和 Pandas 用于数据工程](https://oreil.ly/07ywX)

+   [用于数据工程的 Python 和 SQL 脚本编写](https://oreil.ly/rSBMG)

+   [Web 应用程序和命令行工具用于数据工程](https://oreil.ly/gDDHy)

+   [构建大规模云计算解决方案专项](https://oreil.ly/VZo7x)

+   [Python、Bash 和 SQL 用于数据工程专项](https://oreil.ly/TpIUU)

Pragmatic AI Labs 网站

有关 AWS 的许多免费资源可在 [*Pragmatic AI Labs and Solutions 网站*](https://paiml.com) 上找到。其中包括几本免费书籍：

+   [*用于数据分析的云计算*](https://oreil.ly/I4ADa)

+   [*精简 Python*](https://oreil.ly/JD2Ha)

+   [*Python 命令行工具：使用 Click 设计强大的应用程序*](https://oreil.ly/7zxNa)

+   [*Python 测试*](https://oreil.ly/uJ5cC)

# 本书中使用的约定

本书中使用了以下印刷约定：

*斜体*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`恒定宽度`

用于程序清单，以及段落内引用程序元素如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`恒定宽度粗体`**

显示应由用户直接键入的命令或其他文本。

*`恒定宽度斜体`*

显示应由用户提供的文本或根据上下文确定的值替换。

###### 提示

这个元素表示提示或建议。

###### 注意

这个元素表示一般性的注意事项。

###### 警告

这个元素表示警告或注意事项。

# 使用代码示例

补充资料（代码示例、练习等）可在 [*https://oreil.ly/AWS-with-C-Sharp-examples*](https://oreil.ly/AWS-with-C-Sharp-examples) 下载。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至 *bookquestions@oreilly.com*。

本书旨在帮助您完成工作任务。一般而言，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分内容，否则不需要联系我们以获取许可。例如，编写使用本书多个代码片段的程序不需要许可。出售或分发奥莱利书籍中的示例代码需要许可。引用本书并引用示例代码回答问题不需要许可。将本书中大量示例代码整合到产品文档中需要许可。

我们感谢，但通常不需要署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*使用 C#开发 AWS* by Noah Gift and James Charlesworth (O’Reilly). 版权所有 2023 O’Reilly Media, Inc., 978-1-492-09623-8.”

如果您认为您使用的代码示例超出了合理使用范围或上述许可，请随时联系我们：*permissions@oreilly.com*。

# 奥莱利在线学习

###### 注意

超过 40 年来，[*奥莱利传媒*](https://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。奥莱利的在线学习平台让您随时访问直播培训课程、深入学习路径、交互式编码环境，以及奥莱利和其他 200 多个出版商的大量文本和视频内容。更多信息，请访问 [*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们有一个网页专门用于本书，列出勘误、示例和任何额外信息。您可以访问此页面：[*https://oreil.ly/AWS-with-C-sharp*](https://oreil.ly/AWS-with-C-sharp)。

通过电子邮件 *bookquestions@oreilly.com* 发表评论或询问有关本书的技术问题。

欲了解关于我们书籍和课程的最新消息，请访问 [*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)。

在 Twitter 关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)。

在 YouTube 关注我们：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)。

# 致谢

## Noah

有机会能够参与 O’Reilly 的图书创作总是一种荣幸。这本书是我为 O’Reilly 出版的第四本书，目前正在撰写第五本关于企业 MLOps 的书籍。感谢 [雷切尔·鲁米奥提斯](https://oreil.ly/GbCM4) 让我有机会参与这本书的创作，以及 [赞·麦克奎德](https://oreil.ly/TyZX7) 提供的出色战略指导。

我的合著者 [詹姆斯·查尔斯沃思](https://oreil.ly/ReRkb) 和我们的开发编辑 [梅丽莎·波特](https://oreil.ly/Sm2Nn) 都是非常棒的合作伙伴，我很幸运能和这两位才华横溢的人一起完成这个需求量巨大的项目。感谢来自 AWS 的 Lior Kirshner-Shalom，以及 Nivas Durairaj、David Pallmann 和 Bryan Hogan，他们在技术审阅期间提供了许多有见地的评论和修正。

我也要感谢我的许多现任和前任学生，以及 [杜克大学数据科学硕士项目](https://oreil.ly/2SWqF)，[杜克大学人工智能工程硕士项目](https://oreil.ly/bTcbr)，[加州大学戴维斯分校商业分析硕士项目](https://oreil.ly/ZB0SH)，[加州大学伯克利分校](https://oreil.ly/cX4XH)，[西北大学数据科学硕士项目](https://oreil.ly/i9AIB)，以及 [田纳西大学](https://oreil.ly/u3rJi) 和 [北卡罗来纳大学夏洛特分校](https://oreil.ly/yCydZ) 的教职员工。 

这些人包括（排名不分先后）：[迪克森·刘易斯](https://oreil.ly/gMxsP)，[安德鲁·B·哈加顿](https://oreil.ly/IEnYf)，[海曼特·巴尔加瓦](https://oreil.ly/Mgkru)，[艾米·拉塞尔](https://oreil.ly/ABcjv)，[阿什温·阿拉温达克尚](https://oreil.ly/fqVSg)，[肯·吉尔伯特](https://oreil.ly/8aoUi)，[米歇尔·巴林斯](https://oreil.ly/PouWU)，[乔恩·赖夫施奈德](https://oreil.ly/IsCsh)，[罗伯特·卡尔德班克](https://oreil.ly/iQ0yY) 和 [阿尔弗雷多·德扎](https://oreil.ly/HvsSl)。

最后，感谢我的家人，Leah，Liam 和 Theodore，在周末和深夜为了赶项目截止日期而忍受我工作。

## James

这是我与 O’Reilly 的第一本书，我们从团队中收到了巨大的支持，特别是 Melissa Potter，没有她，你们将不得不忍受更多我那复杂的英式拼写。我也要感谢技术审阅者，特别是 AWS 的 David Pallmann 和 Bryan Hogan。

技术行业发展迅速，没有站在巨人的肩膀上，你就不可能有所成就。因此，我要感谢我的两位前任老板 [詹姆斯·伍德利](https://oreil.ly/0ZmWS) 和 [戴夫·伦辛](https://oreil.ly/dlL6x)，感谢他们近年来给予我的建议、辅导、指导和支持。

最后，特别感谢我的不可思议的妻子，卡雅（Kaja），在我所有努力中支持我，我永远是你的理查德·卡斯尔。
