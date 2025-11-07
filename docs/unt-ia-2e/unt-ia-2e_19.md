# 附录 B. 与 Unity 一起使用的工具

使用 Unity 开发游戏需要依赖各种外部软件工具来处理各种任务。在第一章中，我们讨论了一个外部工具：Visual Studio，尽管它捆绑在 Unity 中，但技术上是一个独立的应用程序。以类似的方式，开发者依赖一系列外部工具来完成 Unity 内部的工作。

这并不是说 Unity 缺乏它应该拥有的功能。相反，游戏开发过程非常复杂和多元，任何设计良好的软件，具有明确的焦点和清晰的关注点分离，不可避免地会限制自己在过程的一个有限子集上表现出色。在这种情况下，Unity 专注于成为将游戏的所有内容粘合在一起并使其工作的胶水和引擎。创建所有这些内容是通过其他工具完成的；让我们看看一些可能对你有用的软件类别。

## B.1 编程工具

我们已经讨论了与 Unity 一起使用的最重要的编程工具：Visual Studio。但你应该了解其他编程工具，正如你将在本节中看到的。

### B.1.1 Rider

如第一章所述，尽管 Unity 随带一个版本的 Visual Studio，但你可以选择使用不同的 IDE。最常见的选择是 Visual Studio Code 或 JetBrains Rider。Rider ([www.jetbrains.com/lp/dotnet-unity/](https://www.jetbrains.com/lp/dotnet-unity/)) 是一个强大的 C# 编程环境，具有 Unity 集成功能。

### B.1.2 Xcode

Xcode 是苹果公司（特别是 IDE，但也包括 Apple 平台的 SDK）提供的编程环境。尽管你仍然会在 Unity 内部完成大部分工作，但你需要使用 Xcode ([`developer.apple.com/xcode/`](https://developer.apple.com/xcode/)) 来部署游戏到 iOS。这项工作通常涉及使用 Xcode 中的工具进行调试或分析你的应用程序。

### B.1.3 Android SDK

就像你需要安装 Xcode 来部署到 iOS 一样，你需要下载 Android SDK 来部署到 Android。通常，你会在 Unity Hub 中下载 Android 模块的同时下载 SDK。或者，Android SDK 与 Android Studio 一起提供，网址为 [`developer.android.com/studio`](https://developer.android.com/studio)。与构建 iOS 游戏不同，你不需要在 Unity 之外启动任何开发工具——你只需在 Unity 中设置指向 Android SDK 的首选项即可。

### B.1.4 版本控制（Git，SVN）

任何体量较大的软件开发项目都会涉及对代码文件的大量复杂修订，因此程序员开发了一类称为 *版本控制系统*（VCS）的软件来处理这个问题。其中一些最受欢迎的免费系统是 Git ([`git-scm.com`](https://git-scm.com/)) 和 Apache Subversion（也称为 SVN，[`subversion.apache.org`](https://subversion.apache.org)）。

如果你还没有使用版本控制系统（VCS），我强烈建议你开始使用。Unity 会在项目文件夹中填充临时文件和工作区设置，但唯一需要纳入版本控制的文件夹是 Assets（确保你的版本控制系统正在获取 Unity 生成的元文件），Packages 和 ProjectSettings。

## B.2 3D 艺术应用程序

虽然 Unity 完全能够处理 2D 图形（第五章和第六章专注于 2D 图形），但它最初是一个 3D 游戏引擎，并且继续拥有强大的 3D 图形功能。许多 3D 艺术家至少使用本节中描述的软件包之一。

### B.2.1 Maya

Autodesk Maya ([www.autodesk.com/products/maya/overview](https://www.autodesk.com/products/maya/overview)) 是一个根植于电影制作的 3D 艺术和动画软件包。Maya 的功能集涵盖了几乎 3D 艺术家可能遇到的所有任务，从制作美丽的电影动画到制作高效的适用于游戏的模型。在 Maya 中完成的 3D 动画（如角色行走）可以导出到 Unity 中。

### B.2.2 3ds Max

另一个广泛使用的 3D 艺术和动画软件包，Autodesk 3ds Max ([www.autodesk.com/products/3ds-max/overview](https://www.autodesk.com/products/3ds-max/overview)) 提供了几乎相同的功能集，并且在工作流程上与 Maya 相当。3ds Max 仅在 Windows 上运行（而其他工具，包括 Maya，是跨平台的），但在游戏行业中同样被广泛使用。

### B.2.3 Blender

虽然在游戏行业中不像 3ds Max 或 Maya 那样常用，但 Blender ([www.blender.org](https://www.blender.org)) 与那些其他应用程序相当。Blender 还涵盖了几乎所有 3D 艺术任务，而且最好的是，Blender 是开源的！鉴于它在所有平台上都可以免费使用，Blender 是这本书假设可用的唯一 3D 艺术应用程序。

### B.2.4 SketchUp

这个简单易用的建模工具非常适合用于建筑和建筑元素。与之前的工具不同，SketchUp ([www.sketchup.com](https://www.sketchup.com)) 并不涵盖所有或甚至大多数 3D 艺术任务；相反，它专注于使建模建筑和其他简单形状变得容易。这个工具在游戏开发中用于白盒建模和关卡编辑。

## B.3 2D 图像编辑器

2D 图像对于所有游戏都至关重要，无论是直接用于 2D 游戏还是作为 3D 模型表面的纹理。在本节中，你将看到在游戏开发中经常出现的几种 2D 图形工具。

### B.3.1 Photoshop

Adobe Photoshop ([www.adobe.com/products/photoshop.html](https://www.adobe.com/products/photoshop.html)) 是最广泛使用的 2D 图像应用程序。Photoshop 中的工具可以用来修饰现有图像，应用图像过滤器，甚至从头开始绘制图片。Photoshop 支持数十种文件格式，包括 Unity 中使用的所有图像格式。

### B.3.2 GIMP

*GNU Image Manipulation Program*（GIMP）的缩写，GIMP ([www.gimp.org](https://www.gimp.org)) 是最著名的开源 2D 图形应用程序。在功能和易用性方面，GIMP 略逊于 Photoshop，但它仍然是一款有用的图像编辑器，而且价格无法匹敌！

### B.3.3 TexturePacker

与之前提到的工具不同，这些工具都超出了游戏开发领域，而 TexturePacker 仅适用于游戏开发。但它在设计任务上非常出色：将精灵图集组装起来用于 2D 游戏。如果你正在开发 2D 游戏，你可能想尝试 TexturePacker ([www.codeandweb.com/texturepacker](https://www.codeandweb.com/texturepacker))。

### B.3.4 Aseprite, Pyxel Edit

像素艺术是 2D 游戏中最具辨识度的艺术风格之一，Aseprite ([www.aseprite.org](http://www.aseprite.org)) 和 Pyxel Edit ([www.pyxeledit.com](http://www.pyxeledit.com)) 是优秀的像素艺术工具。虽然技术上 Photoshop 也可以用于像素艺术，但它并不专注于这项任务。此外，Aseprite 和 Pyxel Edit 在动画功能方面更为突出。

## B.4 音频软件

可用的音频制作工具琳琅满目，包括音频编辑器（处理原始波形）和序列器（使用音符序列创作音乐）。为了展示可用的音频软件，本节将探讨两种主要的音频编辑工具。除此之外的例子还包括 Logic、Ableton 和 Reason。

### B.4.1 Pro Tools

Pro Tools 音频软件([www.avid.com/en/pro-tools](https://www.avid.com/en/pro-tools))拥有许多实用功能，并被无数音乐制作人和音频工程师视为行业标准。它常用于各种专业音频工作，包括游戏开发。

### B.4.2 Audacity

尽管在专业音频工作中并不像 Audacity ([www.audacityteam.org](https://www.audacityteam.org))那样有用，但它是一款小巧实用的音频编辑器，适用于小规模音频工作，例如准备短音频文件作为游戏中的音效。这是寻找开源音频编辑软件者的热门选择。
