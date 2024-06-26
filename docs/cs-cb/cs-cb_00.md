# 前言

# 我为什么写这本书

在职业生涯中，我们收集了许多工具。无论是概念、技术、模式还是可重用代码，这些工具帮助我们完成工作。我们收集的越多，越好，因为我们有很多问题要解决，要构建的应用程序也很多。*C# Cookbook* 通过提供各种配方，为您的工具集增添了一笔。

事物随时间变化，包括编程语言。截至本书写作时，C# 编程语言已经超过 20 年，并且在其生命周期内软件开发已经发生了变化。可以写很多配方，这本书承认了 C#随时间的演变以及现代 C#代码使我们的生产力更高的事实。

本书充满了我在整个职业生涯中使用的配方。除了阐述问题、呈现代码和解释解决方案外，每次讨论还包括更深入的洞察，解释为什么每个配方都很重要。在整本书中，我避免了对流程的倡导或绝对声明“你必须这样做”，因为在创建软件时我们需要做出权衡。事实上，你会发现有几次讨论关于一个配方的后果或权衡。这尊重了你可以考虑一个配方在多大程度上适合你的事实。

# 本书的受众

本书假定你已经了解基本的 C#语法。也就是说，这里有适合各个开发者水平的配方。无论你是初学者、中级还是高级开发者，都应该能找到适合你的内容。如果你是架构师，可能会有一些有趣的配方帮助你迅速掌握最新的 C#技术。

# 本书的组织方式

在为这本书进行头脑风暴时，整个焦点都集中在回答“C# 开发人员需要做什么？”的问题上。查看列表时，某些模式浮现并演变成章节：

+   当我编写代码时，我做的第一件事之一就是构建类型并组织应用程序。因此，我写了第一章，展示如何创建和组织类型。你将看到处理模式的配方，因为这是我编码的方式。

+   创建类型后，我们添加类型成员，如方法及其包含的逻辑，这是第二章中自然的一类配方。

+   代码有何用处，除非它能正常工作？这就是为什么我添加了第三章，其中包含有助于提高代码质量的配方。虽然这一章节充满了有用的配方，你可能会想查看一个展示如何使用可空引用类型的配方。

虽然第一章到第三章遵循“C# 开发人员需要做什么？”的主题，从第四章到书的结尾，我进行了分离，专注于技术特定的重点：

+   许多人认为语言集成查询（LINQ）是一个数据库技术。虽然 LINQ 对于与数据库一起工作很有用，但它也非常适用于内存数据操作和查询。这就是为什么第四章讨论了使用名为 LINQ to Objects 的内存提供程序的功能。

+   反射是 C# 1 的一部分，但动态编程在 C# 4 中稍后出现。我认为在第五章中讨论这两种技术很重要，甚至展示动态编程在某些情况下可能比反射更好。还请查看 Python 互操作食谱。

+   异步编程是 C#的一个重要补充，表面上看似乎很简单。第六章涵盖了涉及几个你可能不知道的重要功能的异步食谱。

+   所有应用程序都使用数据，无论是保护、解析还是序列化。第七章包括涵盖数据处理不同需求的多个食谱。它侧重于一些用于处理数据的新库和算法。

+   C#语言在模式匹配领域在最近几个版本中发生了最大的变化之一。有太多模式匹配的内容，我成功地填充了第八章，仅用于模式匹配的食谱。

+   C#继续发展，第九章涵盖了专门针对 C# 9 的食谱。我们将研究一些新特性并讨论如何应用它们。虽然我在讨论中提供了见解，但请记住，有时新功能在后续版本中可能更加完善。如果您对最前沿感兴趣，这些食谱非常有趣。

# 本书使用的约定

本书使用以下印刷约定：

*斜体*

指示新术语、URL、电子邮件地址、文件名和文件扩展名。

`固定宽度`

用于程序清单，以及段落内用于引用诸如变量或函数名称、数据库、数据类型、环境变量、语句和关键字等程序元素。

**`固定宽度加粗`**

显示用户应按字面输入的命令或其他文本。

*`固定宽度斜体`*

显示应替换为用户提供值或由上下文确定的值的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素指示警告或注意事项。

# 使用代码示例

补充资料（代码示例，练习等）可在[*https://github.com/JoeMayo/csharp-nine-cookbook*](https://github.com/JoeMayo/csharp-nine-cookbook)下载。

如果您有技术问题或使用代码示例时出现问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作。通常情况下，如果本书附带示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分内容，否则您无需征得我们的许可。例如，编写一个使用本书中多个代码片段的程序不需要许可。销售或分发来自 O’Reilly 书籍的示例代码需要许可。引用本书并引用示例代码来回答问题不需要许可。将本书中大量示例代码整合到产品文档中需要许可。

我们感谢，但通常不需要署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*C# Cookbook* by Joe Mayo (O’Reilly). Copyright 2022 Mayo Software, LLC, 978-1-492-09369-5.”

如果您认为您对示例代码的使用超出了合理使用范围或上述授权，请随时通过 *permissions@oreilly.com* 联系我们。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](http://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深入学习路径、交互式编码环境，以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频内容。有关更多信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送给出版商：

+   O’Reilly Media, Inc.

+   Gravenstein Highway North 1005 号

+   Sebastopol, CA 95472

+   800-998-9938 (美国或加拿大)

+   707-829-0515 (国际或本地)

+   707-829-0104 (传真)

我们为这本书制作了一个网页，列出勘误、示例和任何其他信息。您可以访问 [*https://oreil.ly/c-sharp-cb*](https://oreil.ly/c-sharp-cb)。

通过邮件 *bookquestions@oreilly.com* 来评论或提问关于本书的技术问题。

关于我们的书籍和课程的新闻和信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

在 Facebook 找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 观看我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

从概念到交付，有许多人参与创建新书。我想要感谢那些在 *C# Cookbook* 上帮助的人们。

高级内容采购编辑阿曼达·奎因为这本书的概念提供了帮助，并在我概述其内容时提供了反馈意见。内容开发编辑安吉拉·鲁菲诺帮助我制定了标准和工具，对我的写作提出了反馈，并在整个过程中给予了极大的帮助。技术编辑巴萨姆·阿卢吉利、奥克塔维奥·埃尔南德斯和沙德曼·库德奇卡纠正了错误，提供了出色的见解，并分享了新的想法。制作编辑和供应商协调员凯瑟琳·托泽及时通知我新的早期发布情况，并协调其他生产事项。编辑贾斯汀·比林在改善我的写作方面做得非常出色。这本书的实现离不开幕后的许多人的支持。

我想要感谢大家。我对你们的贡献心存感激，祝愿你们一切顺利。
