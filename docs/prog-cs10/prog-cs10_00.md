# 前言

C#现在已经存在大约二十年。它在强大性和规模上稳步增长，但微软始终保持了其基本特性的完整性。每一个新的能力都设计成与其余部分清晰集成，增强语言而不将其变成杂乱无章的特性集合。

虽然 C#在其核心上仍然是一种相当简单的语言，但现在关于它的内容远远超过了它的首次出现。由于需要涵盖如此广泛的内容，本书期望读者具备一定的技术能力。

# 本书的受众

我写这本书是为了有经验的开发者——我多年来一直在编程，并且我决定把这本书设计成我如果在其他语言上有这种经验，并且今天正在学习 C#时想要阅读的书籍。尽管早期版本解释了一些基本概念，如类、多态性和集合，但我假设读者已经知道这些是什么。早期章节仍然描述 C#如何呈现这些常见概念，但重点是特定于 C#的细节，而不是广泛的概念。

# 本书使用的约定

本书使用以下排版约定：

*斜体*

指示新术语，URL，电子邮件地址，文件名和文件扩展名。

`Constant width`

用于程序清单，以及在段落中引用程序元素，例如变量或函数名称，数据库，数据类型，环境变量，语句和关键字。

**`Constant width bold`**

显示用户应直接输入的命令或其他文本。在示例中，突出显示特别感兴趣的代码。

*`Constant width italic`*

显示应替换为用户提供的值或由上下文确定的值的文本。

###### 提示

这个元素表示一个提示或建议。

###### 注意

这个元素表示一般提示。

###### 警告

这个元素表示一个警告或注意事项。

# 使用代码示例

可下载的补充材料（代码示例、练习等）位于[*https://oreil.ly/prog-cs-10-repo*](https://oreil.ly/prog-cs-10-repo)。

如果您有技术问题或使用代码示例遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作。通常情况下，如果本书提供了示例代码，您可以在您的程序和文档中使用它。除非您复制了大量代码，否则无需联系我们请求许可。例如，编写一个使用本书中几个代码片段的程序不需要许可。销售或分发 O'Reilly 书籍中的示例代码需要许可。引用本书并引用示例代码回答问题不需要许可。将本书中大量示例代码整合到您产品的文档中需要许可。

我们感谢您的支持，但通常不要求署名。一般的署名包括标题、作者、出版商和 ISBN。例如：“*Programming C# 10* by Ian Griffiths (O’Reilly). Copyright 2022 by Ian Griffiths, 978-1-098-11781-8.”

如果您认为您对代码示例的使用超出了合理使用范围或上述许可，请随时通过*permissions@oreilly.com*与我们联系。

# O’Reilly 在线学习

###### 注意

超过 40 年来，[*O’Reilly*](http://oreilly.com)为公司提供技术和商业培训、知识和见解，帮助它们取得成功。

我们独特的专家和创新者网络通过书籍、文章、会议和我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深度学习路径、交互式编码环境，以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。更多信息，请访问[*http://oreilly.com*](http://www.oreilly.com)。

# 如何联系我们

关于本书的评论和问题，请联系出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   CA 95472，Sebastopol

+   800-998-9938 (美国或加拿大)

+   707-829-0515 (国际或本地)

+   707-829-0104 (传真)

我们为这本书建立了一个网页，列出勘误、示例和任何额外信息。您可以访问[*https://oreil.ly/prgrmg-c-10*](https://oreil.ly/prgrmg-c-10)获取更多信息。

通过*bookquestions@oreilly.com*向我们发送评论或技术问题的电子邮件。

获取关于我们的书籍和课程的新闻和信息，请访问[*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)。

关注我们的 Twitter 账号：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)。

在 YouTube 上关注我们：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)。

# 致谢

衷心感谢本书的官方技术审稿人：Stephen Toub、Howard van Rooijen 和 Glyn Griffiths。我还要特别感谢那些审阅单独章节或以其他方式提供帮助或信息以改进本书的人：Brian Rasmussen、Eric Lippert、Andrew Kennedy、Daniel Sinclair、Brian Randell、Mike Woodring、Mike Taulty、Bart De Smet、Matthew Adams、Jess Panni、Jonathan George、Mike Larah、Carmel Eve、Ed Freeman、Elisenda Gascon、Jessica Hill、Liam Mooney、Nehemiah Campbell 和 Shahryar Saljoughi。特别感谢 endjin，不仅允许我抽出时间写这本书，还为创造这样一个优秀的工作环境而致谢。

感谢 O’Reilly 公司的所有人员，他们的工作使这本书得以问世。特别感谢 Corbin Collins 在推动这本书问世方面的支持，以及 Amanda Quinn 在启动这个项目方面的支持。还要感谢 Elizabeth Faerm、Cassandra Furtado、Ron Bilodeau、Nick Adams、Kate Dullea、Karen Montgomery 和 Kristen Brown，在完成这项工作中的帮助。进一步感谢 Sue Klefstad 和 WordCo Indexing Services, Inc. 对索引的工作。也要感谢 Kim Cofer 进行了彻底而周到的编辑工作，以及 Kim Sandoval 的勤奋校对工作。最后，感谢 John Osborn，在我写第一本书时成为 O’Reilly 的作者。
