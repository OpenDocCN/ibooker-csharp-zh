# 附录 D. 在线学习资源

本书是 Unity 游戏开发的全面入门，但在此入门之后还有更多东西要学习。您将在网上找到许多有用的资源，您可以在完成本书后进一步使用这些资源。

## D.1 其他教程

许多网站提供了关于 Unity 内部各种主题的定向信息。其中一些甚至是由 Unity 背后的公司官方提供的。

Unity 手册

Unity 提供了全面的用户手册。它不仅对查找信息很有用，而且主题列表本身也很有用，可以全面了解 Unity 的功能。您可以在 [`docs.unity3d.com/Manual/index.html`](http://docs.unity3d.com/Manual/index.html) 找到手册。

脚本参考

Unity 程序员最终会阅读脚本参考比其他任何资源都要多（至少我是这样！）。用户手册涵盖了引擎的功能和编辑器的使用，但脚本参考是 Unity 编程 API 的详尽参考。每个命令都在 [`docs.unity3d.com/ScriptReference/index.html`](http://docs.unity3d.com/ScriptReference/index.html) 列出。

Unity 学习教程

Unity 的官方网站在“学习”部分包含了几个全面的教程。最重要的是，这些教程都是视频形式。这可能是好是坏，取决于您的观点；如果您喜欢观看视频教程，[`learn.unity.com`](https://learn.unity.com) 是一个值得查看的好网站。

Catlike Coding

Catlike Coding 不是通过完整游戏引导学习者，而是提供了一系列有用和有趣的主题。这些主题甚至不一定专门关于游戏开发，但它们是提升 Unity 编程技能的绝佳方式。教程可以在 [catlikecoding.com/unity/tutorials/](https://catlikecoding.com/unity/tutorials/) 找到。

Stack Exchange 上的游戏开发

Stack Exchange 是另一个非常棒的信息网站，其格式与前面列出的不同。它不像一系列自包含的教程那样，Stack Exchange 提供了主要是文本的问答，鼓励搜索。它涵盖了大量主题的章节，[`gamedev.stackexchange.com`](https://gamedev.stackexchange.com) 是该网站专注于游戏开发的区域。就其价值而言，我在那里查找 Unity 信息几乎和我在脚本参考中查找的次数一样多。

Maya LT 指南

如附录 B 所述，外部艺术应用是创建视觉上令人惊叹的游戏的关键部分。有许多关于 Maya、3ds Max、Blender 或其他 3D 艺术应用的教程资源。附录 C 提供了关于 Blender 的教程。有关使用 Maya LT（Maya 的游戏开发导向版本，价格较低）的在线指南可在 [`steamcommunity.com/sharedfiles/filedetails/?id=242847724`](https://steamcommunity.com/sharedfiles/filedetails/?id=242847724) 找到。

## D.2 代码库

虽然之前列出的资源提供了关于 Unity 的教程和/或学习信息，但本节中的网站提供了可以在项目中使用的代码。库和插件是另一种有用的资源，不仅可以直接使用，还可以通过阅读它们的代码来学习。

Unity 社区库

Unity 库是许多开发者的代码贡献的中心数据库，那里托管的脚本涵盖了广泛的功能。该页面的资源部分链接到额外的脚本集合。您可以在[`github.com/UnityCommunity/UnityLibrary`](https://github.com/UnityCommunity/UnityLibrary)浏览内容。

DOTween 和 LeanTween

如第三章简要提到的，在游戏中常用的一种运动效果被称为*tween*。在这种类型的运动中，单个代码命令可以在一定时间内设置一个对象移动到目标位置。可以使用像 DOTween ([`dotween.demigiant.com`](http://dotween.demigiant.com))或 LeanTween ([`github.com/dentedpixel/LeanTween`](https://github.com/dentedpixel/LeanTween))这样的库来添加 tweening 功能。

后处理堆栈

后处理堆栈是向您的游戏添加诸如景深和运动模糊等视觉效果的简单方法。其中许多效果已集成到一个超级组件中。该包的描述可以在[`mng.bz/9aXl`](https://shortener.manning.com/9aXl)找到。

移动通知包

虽然 Unity 的核心已经覆盖了所有游戏平台的各种功能，但对于移动游戏，您可能想要安装具有额外功能的包。Unity Mobile Notifications 包（[`mng.bz/jjvx`](https://shortener.manning.com/jjvx)）专注于通知，即手机应用程序生成的小警报。

Firebase Cloud Messaging

虽然前面提到的 Unity 包可以处理 Android 和 iOS 的本地通知，但它仅在 iOS 上支持远程通知（也称为*推送通知*）。Android 上的推送通知通过名为 Firebase Cloud Messaging 的服务工作，Firebase（[`mng.bz/WBg0`](https://shortener.manning.com/WBg0)）的开发者页面解释了如何使用其 Unity SDK。

Google 的 Play Games 服务

在 iOS 上，Unity 内置了 GameCenter 集成，因此您的游戏可以拥有平台原生的排行榜和成就。Android 上的等效系统称为 Google Play Games；尽管它没有内置在 Unity 中，但 Google 在[`mng.bz/80QP`](http://mng.bz/80QP)维护了一个插件。

FMOD Studio

Unity 内置的音频功能在播放录音方面表现良好，但在高级音效设计工作中可能有限。FMOD Studio 是一个高级音效设计工具，它有一个 Unity 插件。您可以在[www.fmod.com/studio](https://www.fmod.com/studio)找到它。
