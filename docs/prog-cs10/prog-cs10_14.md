# 第十四章 属性

在.NET 中，您可以使用*属性*为组件、类型及其成员添加注解。属性的目的是控制或修改框架、工具、编译器或 CLR 的行为。例如，在 第一章 中，我展示了一个使用 `[TestClass]` 属性进行注释的类。这告诉单元测试框架，被注释的类包含一些作为测试套件一部分运行的测试。

属性是信息的被动容器，在自身不执行任何操作。可以通过与物理世界的类比来理解，如果您打印包含地址和跟踪信息的运输标签，并将其附加到包裹上，该标签本身不会导致包裹自行送达目的地。这样的标签只有在包裹交到运输公司手中时才有用。当公司取走您的包裹时，它会期望找到标签，并使用它来确定如何路由您的包裹。因此，标签很重要，但最终它的唯一工作是提供某些系统所需的信息。.NET 属性的工作方式与此类似——它们只有在某些东西正在寻找它们时才会发挥作用。某些属性由 CLR 或编译器处理，但这些只是少数。大多数属性被框架、库、工具（如单元测试运行器）或您自己的代码消耗。

# 应用属性

为了避免在类型系统中引入额外的概念，.NET 将属性建模为.NET 类型的实例。要作为属性使用，类型必须派生自 `System.Attribute` 类，但除此之外，它可以是完全普通的。要应用属性，您将类型名称放在方括号中，这通常直接放在属性的目标之前。（由于 C#大多数情况下忽略空白字符，当目标是类型或成员时，属性不必位于单独的行上，但这是约定俗成的。）示例 14-1 展示了来自微软测试框架的一些属性。我已经将一个应用到类上，以指示这个类包含我希望运行的测试，并且还将属性应用到单独的方法上，告知测试框架哪些方法代表测试，哪些包含要在每次测试前运行的初始化代码。

##### 示例 14-1 单元测试类中的属性

```cs
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace ImageManagement.Tests;

`[TestClass]`
public class WhenPropertiesRetrieved
{
    private ImageMetadataReader? _reader;

    `[TestInitialize]`
    public void Initialize()
    {
        _reader = new ImageMetadataReader(TestFiles.GetImage());
    }

    `[TestMethod]`
    public void ReportsCameraMaker()
    {
        Assert.AreEqual(_reader!.CameraManufacturer, "Fabrikam");
    }

    `[TestMethod]`
    public void ReportsCameraModel()
    {
        Assert.AreEqual(_reader!.CameraModel, "Fabrikam F450D");
    }
}
```

如果您查看大多数属性的文档，您会发现它们的实际名称以 `Attribute` 结尾。如果在括号中指定的名称没有对应的类，C# 编译器会尝试添加 `Attribute`，所以 示例 14-1 中的 `[TestClass]` 属性指的是 `TestClassAttribute` 类。如果您确实希望如此，可以完整拼写类名，例如 `[TestClassAttribute]`，但更常见的是使用缩写版本。

如果你想应用多个属性，有两种选择。你可以提供多组括号，或者将多个属性放在单一对括号内，用逗号分隔。

一些属性类型可以接受构造函数参数。例如，微软的测试框架包含一个`TestCategoryAttribute`。在运行测试时，您可以选择仅执行某个类别中的测试。此属性要求您将类别名称作为构造函数参数传递，因为如果不指定名称，则应用此属性没有意义。正如 Example 14-2 所示，指定属性的构造函数参数的语法并不令人意外。

##### Example 14-2\. 带构造函数参数的属性

```cs
`[TestCategory("Property Handling")]`
[TestMethod]
public void ReportsCameraMaker()
{
    ...
```

你也可以指定属性或字段的值。一些属性具有只能通过属性或字段控制的特性，而不是构造函数参数。（如果一个属性有很多可选设置，通常更容易将这些设置呈现为属性或字段，而不是为每种可能的设置组合定义构造函数重载。）其语法是在构造函数参数之后写一个或多个`*PropertyOrFieldName*=*Value*`条目（如果没有构造函数参数，则是代替它们）。Example 14-3 展示了另一个在单元测试中使用的属性，`ExpectedExceptionAttribute`，它允许你指定在测试运行时期望它抛出特定异常。异常类型是必需的，因此我们将其作为构造函数参数传递，但这个属性还允许你指定测试运行器是否接受从指定类型派生的异常。 （默认情况下，它只接受完全匹配。）这由`AllowDerivedTypes`属性控制。

##### Example 14-3\. 使用属性指定可选的属性设置

```cs
`[ExpectedException(typeof(ArgumentException), AllowDerivedTypes = true)]`
[TestMethod]
public void ThrowsWhenNameMalformed()
{
    ...
```

应用属性并不会导致它们被实例化。当你应用一个属性时，你所做的只是提供关于如何创建和初始化该属性的指令，如果有需要的话。（有一种常见的误解认为方法属性在方法运行时会被实例化，其实不然。）当编译器为程序集构建元数据时，它会包含关于已应用到哪些项上的属性的信息，包括构造函数参数和属性值的列表，CLR 只有在有需要的时候才会提取出来使用。例如，当你要求 Visual Studio 运行你的单元测试时，它会加载你的测试程序集，然后对每个公共类型，它会向 CLR 查询任何与测试相关的属性。这就是属性被构造的时机。如果你只是简单地加载程序集，比如从另一个项目添加引用然后使用它包含的一些类型，属性就不会存在——它们只会保留在元数据中，作为一组冻结在程序集元数据中的构建指令。

## 属性目标

属性可以应用于许多不同类型的目标。你可以在反射 API 中展示的类型系统特性的任何地方放置属性，就像我在第十三章中展示的那样。具体来说，你可以将属性应用于程序集、模块、类型、方法、方法参数、构造函数、字段、属性、事件和泛型类型参数。此外，你还可以提供目标为方法返回值的属性。

对于大多数情况，你只需在目标前面放置属性即可标识目标。但对于程序集或模块来说不是这样，因为在你的源代码中没有单个特性代表它们——你项目中的所有内容都会进入它生成的程序集中，并且模块同样是一个集合体（通常构成整个程序集，就像我在第十二章中描述的那样）。因此，对于这些情况，我们必须在属性开头明确说明目标。你经常会在像*GlobalSuppressions.cs*文件中看到类似示例 14-4 所示的程序集级别属性。Visual Studio 有时会建议修改你的代码，如果你选择抑制这些警告，它就会使用程序集级别的属性来实现。

##### 示例 14-4. 程序集级别属性

```cs
[assembly: System.Diagnostics.CodeAnalysis.SuppressMessage(
 "Style",
 "IDE0060:Remove unused parameter",
 Justification = "This is just some example code from a book",
 Scope = "member",
 Target = "~M:Idg.Examples.SomeMethod")]
```

你可以在任何文件中放置程序集级别的属性。唯一的限制是它们必须出现在任何命名空间或类型定义之前。程序集级别属性之前应该出现的只有你需要的`using`指令、注释和空白（这些都是可选的）。

模块级别的属性遵循相同的模式，尽管它们非常少见，主要是因为多模块程序集非常罕见，并且在最新版本的.NET 中不受支持——它们仅适用于.NET Framework。示例 14-5 展示了如何配置特定模块的调试性，如果您希望多模块程序集中的一个模块易于调试，但其余模块带有完全优化的 JIT 编译，则可以使用此功能。（这是一个假设的场景，我只是为了展示语法。实际上，您不太可能需要这样做。）我稍后会讨论`DebuggableAttribute`，在“JIT 编译”中。

##### 示例 14-5\. 模块级别属性

```cs
using System.Diagnostics;

[module: Debuggable(DebuggableAttribute.DebuggingModes.DisableOptimizations)]
```

另一种需要资格证书的目标是编译器生成的字段。当您在不为 getter 或 setter 提供代码的属性中使用时，以及在没有显式的`add`和`remove`实现的`event`成员中使用时，您会得到这些字段。示例 14-6 中的属性应用于保存属性值的字段和事件委托；如果没有`field:`限定符，这些位置的属性将适用于属性或事件本身。

##### 示例 14-6\. 编译器生成的属性和事件字段的属性

```cs
[field: NonSerialized]
public int DynamicId { get; set; }

[field: NonSerialized]
public event EventHandler? Frazzled;
```

方法的返回值可以进行注释，这也需要资格证书，因为返回值属性放在方法前面，与适用于方法本身的属性位于同一位置。（参数的属性不需要资格证书，因为这些属性与参数一起出现在括号内。）示例 14-7 展示了一个同时应用于方法和返回类型的属性的方法。（本示例中的属性是支持互操作服务的一部分，这些服务使.NET 代码能够调用外部代码，例如操作系统 API。此示例导入了来自 Win32 DLL 的函数，使您能够从 C#中使用它。非托管代码中存在几种不同的布尔值表示形式，因此我在此处用`MarshalAsAttribute`对返回类型进行了注释，以说明 CLR 应该期望哪个特定形式。）

##### 示例 14-7\. 方法和返回值属性

```cs
[DllImport("User32.dll")]
[return: MarshalAs(UnmanagedType.Bool)]
static extern bool IsWindowVisible(HandleRef hWnd);
```

那么对于我们不显式编写方法声明的情况呢？正如您在第九章中看到的那样，Lambda 语法允许我们编写一个值为委托的表达式。编译器会生成一个正常的方法来保存代码（通常是在一个隐藏的类中），我们可能希望将该方法传递给使用属性来控制其功能的框架，例如 ASP.NET Core Web 框架。示例 14-8 展示了在使用 Lambda 时如何指定这些属性。

##### 示例 14-8\. 带属性的 Lambda

```cs
app.MapGet(
    "/items/{id}",
 [Authorize] ([FromRoute] int id) => $"Item {id} requested");
```

`MapGet` 方法告诉 ASP.NET Core 框架应该在收到与特定模式匹配的 URL 的 `GET` 请求时如何行为。第一个参数指定模式，第二个是定义行为的委托。我在这里使用了 lambda 语法，并应用了一些属性。

第一个属性是 `[Authorize]`。它出现在参数列表之前，因此其目标是整个方法。（您还可以在此位置使用 `return:` 属性。）这会导致 ASP.NET Core 阻止未经身份验证的请求匹配此 URL 模式。`[FromRoute]` 属性位于参数列表的括号内，因此它适用于 `id` 参数，并告诉 ASP.NET Core 我们希望从 URL 模式中同名表达式获取该特定参数的值。因此，如果请求 *https://myserver/items/42* 进来，ASP.NET Core 首先会检查请求是否符合应用程序配置的身份验证和授权要求，如果符合，然后会调用我的 lambda，并将 `42` 作为 `id` 参数传递。

###### 注意

例子 9-22 在 第九章 中展示了在某些情况下可以省略细节。参数列表周围的括号通常对于单参数 lambda 是可选的。但是，如果您要将属性应用于 lambda，则括号 *必须* 存在。要看清楚原因，请想象如果 例子 14-8 在参数列表周围没有括号：那么不清楚属性是应用于方法还是参数。

## 编译器处理的属性

C# 编译器识别特定的属性类型，并以特殊方式处理它们。例如，程序集的名称和版本是通过属性设置的，还有一些关于您的程序集的相关信息。正如第十二章所述，在现代 .NET 项目中，构建过程会为您生成一个隐藏的源文件，其中包含这些信息。如果您感兴趣，它通常会出现在项目的 *obj\Debug* 或 *obj\Release* 文件夹中，并且名称通常类似于 *YourProject.AssemblyInfo.cs*。例子 14-9 展示了一个典型的示例。

##### 例 14-9\. 具有程序集级属性的典型生成文件

```cs
//------------------------------------------------------------------------------
// <auto-generated>
//     This code was generated by a tool.
//     Runtime Version:4.0.30319.42000
//
//     Changes to this file may cause incorrect behavior and will be lost if
//     the code is regenerated.
// </auto-generated>
//------------------------------------------------------------------------------

using System;
using System.Reflection;

[assembly: System.Reflection.AssemblyCompanyAttribute("MyCompany")]
[assembly: System.Reflection.AssemblyConfigurationAttribute("Debug")]
[assembly: System.Reflection.AssemblyFileVersionAttribute("1.0.0.0")]
[assembly: System.Reflection.AssemblyInformationalVersionAttribute("1.0.0")]
[assembly: System.Reflection.AssemblyProductAttribute("MyApp")]
[assembly: System.Reflection.AssemblyTitleAttribute("MyApp")]
[assembly: System.Reflection.AssemblyVersionAttribute("1.0.0.0")]

// Generated by the MSBuild WriteCodeFragment class.
```

旧版本的.NET Framework SDK 在构建时未生成此文件，因此如果您处理旧项目，您经常会在名为*AssemblyInfo.cs*的文件中找到这些属性。（默认情况下，Visual Studio 将其隐藏在解决方案资源管理器中项目的属性节点中，但它仍然只是一个普通的源文件。）现代项目中使用的文件生成优势在于，名称不太可能不同步。例如，默认情况下，程序集产品和标题将与项目文件名相同。如果重新命名项目文件，则生成的*YourRenamedProject.AssemblyInfo.cs*将相应更改（除非您向项目文件添加了`<Product>`和`<AssemblyTitle>`属性，在这种情况下它将使用这些属性），而旧的*AssemblyInfo.cs*方法则可能会意外导致名称不匹配。同样，如果从项目构建 NuGet 包，某些属性最终会出现在 NuGet 包和编译的程序集中。当这些都是从项目文件中的信息生成时，更容易保持一致性。

即使您只间接控制这些属性，理解它们也很有用，因为它们会影响编译器输出。

### 名称和版本

正如您在第十二章看到的，程序集具有复合名称。简单名称通常与文件名相同，但不包括*.exe*或*.dll*扩展名，作为项目设置的一部分进行配置。名称还包括版本号，并且通过属性控制，如示例 14-10 所示。

##### 示例 14-10\. 版本属性

```cs
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]
```

正如您可能从第十二章回忆起的，第一个集合设置了程序集名称的版本部分。第二个与.NET 无关——编译器使用它来生成 Win32 风格的版本资源。这是最终用户在 Windows 资源管理器中选择您的程序集并打开属性窗口时看到的版本号。

区域性也是程序集名称的一部分。如果您正在使用第十二章描述的卫星资源程序集机制，通常会自动设置这一点。您可以使用`AssemblyCulture`属性显式设置它，但对于非资源程序集，通常不应设置区域性。（您通常会显式指定的唯一与文化相关的程序集级属性是我在第十二章展示的`NeutralResourcesLanguageAttribute`。）

强名称程序集在其名称中有一个额外的组件：公钥标记。在 Visual Studio 中设置强名称的最简单方法是使用项目属性页中“强名称”部分（位于“生成”部分内）。如果您正在使用 VS Code 或其他编辑器，您可以简单地在您的 *.csproj* 文件中添加两个属性：`SignAssembly` 设置为 `True`，并且 `AssemblyOriginatorKeyFile` 设置为您密钥文件的路径。然而，您也可以通过源代码管理强命名，因为编译器识别一些特殊的属性用于此目的。`AssemblyKeyFileAttribute` 接受包含密钥的文件名。或者，您可以将密钥安装到计算机的密钥存储区（这是 Windows 加密系统的一部分）。如果您想这样做，您可以使用 `AssemblyKeyNameAttribute`。这两个属性中的任何一个都会导致编译器将公钥嵌入程序集并将该密钥的哈希值作为强名称的公钥标记包含在其中。如果密钥文件包含私钥，编译器还会为您的程序集签名。如果不包含私钥，则编译器将无法编译，除非您还启用了延迟签名或公共签名。您可以通过应用带有参数`true`的`AssemblyDelaySignAttribute`来启用延迟签名。或者，您可以将`<DelaySign>true</DelaySign>`或`<PublicSign>true</PublicSign>`添加到您的 *.csproj* 文件中。

###### 警告

尽管与密钥相关的属性触发编译器的特殊处理，但它们仍然像普通属性一样嵌入到元数据中。因此，如果您使用`AssemblyKeyFileAttribute`，则您密钥文件的路径将显示在最终编译输出中。这并不一定是问题，但您可能更喜欢不公开这些细节，因此使用项目级别的强名称配置可能比基于属性的方法更好。

### 描述及相关资源

由`AssemblyFileVersion`属性生成的版本资源并非 C# 编译器可以嵌入到 Win32 样式资源中的唯一信息。还有几个其他属性提供版权信息和其他描述性文本。示例 14-11 展示了典型的选择。

##### Example 14-11\. 典型的程序集描述属性

```cs
[assembly: AssemblyTitle("ExamplePlugin")]
[assembly: AssemblyDescription("An example plug-in DLL")]
[assembly: AssemblyConfiguration("Retail")]
[assembly: AssemblyCompany("Endjin Ltd.")]
[assembly: AssemblyProduct("ExamplePlugin")]
[assembly: AssemblyCopyright("Copyright © 2022 Endjin Ltd.")]
[assembly: AssemblyTrademark("")]
```

与文件版本一样，所有这些信息都可以在文件的属性窗口的详细信息选项卡中看到，这是 Windows Explorer 可以显示的。对于所有这些属性，您都可以通过编辑项目文件来生成它们。

### 调用者信息属性

有一些编译器处理的属性专为您的方法需要有关其被调用上下文信息的场景而设计。这在某些诊断日志记录或错误处理场景中很有用，当然，在实现通常在 UI 代码中使用的特定接口时也非常有帮助。

示例 14-12 说明了如何在日志记录代码中使用这些属性。如果你用这三个属性中的任何一个注释方法参数，编译器在调用者省略参数时会进行一些特殊处理。我们可以请求调用带有注释方法的成员（方法或属性）的名称，调用该方法的包含代码的文件名，或者调用发生的行号。示例 14-12 请求了这三者，但你可以更有选择地使用。

###### 注意

这些属性仅允许用于可选参数。可选参数需要指定默认值，但是当这些属性存在时，C#会始终替换为不同的值，因此你指定的默认值在从 C#（或支持这些属性的 Visual Basic）调用方法时不会被使用。尽管如此，你必须提供一个默认值，因为没有默认值时，参数就不是可选的，所以我们通常使用空字符串、`null`或数字`0`。

##### 示例 14-12\. 将调用者信息属性应用于方法参数

```cs
public static void Log(
    string message,
 [CallerMemberName] string callingMethod = "",
 [CallerFilePath] string callingFile = "",
 [CallerLineNumber] int callingLineNumber = 0)
{
    Console.WriteLine("Message {0}, called from {1} in file '{2}', line {3}",
        message, callingMethod, callingFile, callingLineNumber);
}
```

如果在调用此方法时提供了所有参数，那么不会发生任何异常。但是如果省略了任何可选参数，C#会生成代码来提供关于调用该方法的位置的信息。在示例 14-12 中，三个可选参数的默认值将是调用此`Log`方法的方法或属性的名称，包含调用方法的源代码的完整路径，以及调用`Log`的行号。

`CallerMemberName`属性与我们在第八章中看到的`nameof`运算符有表面上的相似之处。两者都导致编译器创建包含代码某些特性名称的字符串，但它们的工作方式完全不同。使用`nameof`，你总是知道会得到什么字符串，因为它由你提供的表达式决定（例如，在示例 14-12 中的`Log`中写入`nameof(message)`，它将始终评估为`"message"`）。但是`CallerMemberName`改变了应用它们的方法被编译器调用的方式——`cal⁠lin⁠g​Met⁠hod`有该属性，其值不是固定的。它将取决于调用此方法的位置。

###### 注意

你可以另一种方式发现调用方法：`System.Diagnostics`命名空间中的`StackTrace`和`StackFrame`类可以报告调用堆栈中的上层方法的信息。然而，这些方法在运行时开销较高——调用者信息属性在编译时计算值，因此运行时开销非常低（`nameof`也是如此）。此外，`StackFrame`只能在存在调试符号时确定文件名和行号。

虽然诊断日志记录是显而易见的应用，我还提到了大多数.NET UI 开发人员熟悉的某种场景。运行时库定义了一个称为`INotifyPropertyChanged`的接口。正如示例 14-13 所示，这是一个非常简单的接口，只有一个成员，一个称为`PropertyChanged`的事件。

##### 示例 14-13\. `INotifyPropertyChanged`

```cs
public interface INotifyPropertyChanged
{
    event PropertyChangedEventHandler? PropertyChanged;
}
```

实现此接口的类型在其属性更改时引发`PropertyChanged`事件。`PropertyChangedEventArgument`提供一个字符串，其中包含刚刚更改的属性的名称。这些更改通知在 UI 中非常有用，因为它们使对象能够与数据绑定技术（例如.NET 的 WPF UI 框架提供的技术）一起使用，该技术可以在属性更改时自动更新 UI。数据绑定可以帮助您实现 UI 类型直接处理的代码与包含决定应用程序如何响应用户输入的逻辑的代码之间的清晰分离。

实现`INotifyPropertyChanged`可能既繁琐又容易出错。因为`PropertyChanged`事件以字符串形式指示哪个属性已更改，所以很容易拼写错误属性名称，或者在复制和粘贴实现时意外使用错误的名称。另外，如果重命名属性，很容易忘记更改事件的文本，这意味着以前正确的代码现在在引发`PropertyChanged`事件时会提供错误的名称。`nameof`运算符有助于避免拼写错误，并有助于重命名，但无法始终检测到复制和粘贴错误。（例如，在同一类的属性之间粘贴代码时，它不会注意到您未更新名称的情况。）

调用者信息属性可以帮助减少实现此接口时的错误。您可以参考示例 14-14，该示例显示了一个实现`INotifyPropertyChanged`的基类，提供了一个在利用这些属性之一时引发更改通知的帮助程序。（它还使用了空值条件运算符`?.`，以确保仅在委托非空时才调用事件的委托。顺便说一下，当您以这种方式使用运算符时，C#会生成只在委托非空时才评估委托的`Invoke`方法参数的代码。因此，它会跳过当委托为空时的调用`Invoke`，还会避免构造将作为参数传递的`Pro⁠per⁠ty​Cha⁠nge⁠dEv⁠ent⁠Args`。）此代码还会检测值是否真的已更改，仅在这种情况下引发事件，并且其返回值指示是否发生了更改，以防调用者可能会发现这有用。

##### 示例 14-14\. 可重用的`INotifyPropertyChanged`实现

```cs
public class NotifyPropertyChanged : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler? PropertyChanged;

    protected bool SetProperty<T>(
        ref T field,
        T value,
 [CallerMemberName] string propertyName = "")
    {
        if (Equals(field, value))
        {
            return false;
        }

        field = value;

        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        return true;
    }
}
```

存在`[CallerMemberName]`属性意味着从此类型派生的类如果在属性设置器内部调用`SetProperty`，则无需指定属性名称，如 Example 14-15 所示。

##### Example 14-15\. 触发属性更改事件

```cs
public class MyViewModel : NotifyPropertyChanged
{
    private string? _name;

    public string? Name
    {
        get => _name;
        set => SetProperty(ref _name, value);
    }
}
```

即使有了新的属性，实现`INotifyPropertyChanged`显然比自动属性更费力，其中您只需编写`{ get; set; }`并让编译器为您完成工作。但它只比显式实现的简单字段支持属性复杂一点点，并且比没有`[CallerMemberName]`时简单，因为我能够在请求基类触发事件时省略属性名称。更重要的是，它更少容易出错：现在我可以确信每次都会使用正确的名称，即使将来某个时候重命名属性。

.NET 6.0 添加了一个新的调用者信息属性：`CallerArgumentExpression`。Example 14-16 展示了运行时库的`ArgumentNullException`类的摘录。它声明了一个使用此属性的`ThrowIfNull`方法。

##### Example 14-16\. 在`ArgumentNullException.ThrowIfNull`中的`CallerArgumentExpressionAttribute`

```cs
public class ArgumentNullException
{
    public static void ThrowIfNull(
 [NotNull] object? argument,
 [CallerArgumentExpression("argument")] string? paramName =  null)
    {
...
```

正如您所见，`CallerArgumentExpression`属性接受一个字符串参数。这必须是同一方法中另一个参数的名称——在本例中只有一个叫做`argument`，因此必须引用它。其效果是，如果调用此方法而没有为注释的`paramName`参数提供值，C#编译器将传递包含该属性标识的参数的精确表达式的字符串。Example 14-17 展示了`ThrowIfNull`方法的典型调用方式。

##### Example 14-17\. 调用使用`CallerArgumentExpressionAttribute`的方法

```cs
static void Greet(string greetingRecipient)
{
    ArgumentNullException.ThrowIfNull(greetingRecipient);
    Console.WriteLine($"Hello, {greetingRecipient}");
}

Greet("world");
Greet(null!);
```

`Greet`方法需要`greetingRecipient`不为 null，因此调用`Arg⁠ume⁠nt​Nul⁠lEx⁠cep⁠tio⁠n.T⁠hro⁠wIf⁠Null`，传入`greetingRecipient`。因为这段代码没有为`ThrowIfNull`提供第二个参数，编译器将提供我们用于第一个参数的表达式的完整文本。在这种情况下，即是`"gre⁠eti⁠ng​Rec⁠ipi⁠ent"`。因此，当运行此程序时，效果是抛出一个带有此消息的`Arg⁠ume⁠nt​Nul⁠lEx⁠cep⁠tio⁠n`：

```cs
Value cannot be null. (Parameter 'greetingRecipient')
```

在 C# 10.0 之前，我们通常会使用`nameof(greetingRecipient)`来告诉`ArgumentNullException`有问题的参数名称。这种新技术防止了一种特定的错误：以前在抛出异常时很容易传递错误的参数名称。（如果您需要检查多个参数是否为 null，复制和粘贴相关检查会提供大量机会来犯这种错误。）

此属性支持的一个场景是改进断言消息。例如，单元测试库通常提供机制，用于在测试代码执行后断言某些条件是否为真。其思想是，如果您的测试包含类似 `Assert.IsTrue(answer == 42);` 的代码，测试库可以使用 `[CallerArgumentExpression]` 来在失败时报告确切的表达式 (`answer == 42`)。

您可能期望运行时库中的 `Debug.Assert` 方法以类似的方式使用此属性。但是，要使用 `CallerArgumentExpressionAttribute`，您必须添加一个参数来接收表达式文本（除了接收表达式值的现有参数之外），因此这不是一个二进制兼容的更改。新的 `ThrowIfNull` 方法是.NET 6.0 运行时库唯一使用此属性的地方，在我写这篇文章时，微软的测试框架的 NuGet 包尚未使用此属性。但是很可能测试框架会在未来采用这一特性。

## CLR-Handled Attributes

CLR 在运行时会对某些属性进行特殊处理。目前还没有官方的全面属性列表，所以在接下来的几节中，我将简要描述一些最常用的例子。

### InternalsVisibleToAttribute

可以将 `InternalsVisibleToAttribute` 应用于一个程序集，以声明其定义的任何 `internal` 类型或成员应该对一个或多个其他程序集可见。这种属性的一个常见用途是启用内部类型的单元测试。正如 示例 14-18 所示，您可以将程序集的名称作为构造函数参数传递。

###### 注意

强名称会使事情变得更复杂。强名称的程序集无法将其内部成员对不强命名的程序集可见，反之亦然。当一个强命名的程序集将其内部成员对另一个强命名的程序集可见时，它必须指定不仅仅是简单名称，还包括要授予访问权限的程序集的公钥。这不仅仅是我在 第十二章 中描述的公钥令牌——它是整个公钥的十六进制表示，可能有数百位数字。您可以使用 .NET SDK 的 *sn.exe* 实用程序，使用 `-Tp` 开关，后跟程序集的路径，来发现程序集的完整公钥。

##### 示例 14-18\. `InternalsVisibleToAttribute`

```cs
[assembly:InternalsVisibleTo("ImageManagement.Tests")]
[assembly:InternalsVisibleTo("ImageServices.Tests")]
```

这表明您可以通过多次应用该属性并每次使用不同的程序集名称，使类型对多个程序集可见。

CLR 负责执行访问规则。通常，如果您尝试从另一个程序集使用内部类，您将在运行时收到错误。(C# 甚至不允许您编译这样的代码，但是有可能欺骗编译器。或者您可以直接编写 IL。IL 汇编器 *ILASM* 会执行您告诉它的操作，并且比 C# 实施的限制少得多。一旦您通过了编译时的限制，然后您将遇到运行时限制。) 但是，当存在此属性时，CLR 放宽了对您列出的程序集的规则。编译器也理解此属性，并允许尝试使用外部定义的内部类型的代码编译，只要外部库在 `InternalsVisibleToAttribute` 中命名了您的程序集。

除了在单元测试场景中很有用外，这个属性还可以在您想要将代码分散到多个程序集的情况下帮助您。如果您编写了一个大型类库，可能不想将其放入一个庞大的 DLL 中。如果它有几个区域是您的客户可能想要单独使用的，那么将它分割开来以便他们可以部署他们需要的部分就有意义了。但是，尽管您可能可以对库的公共 API 进行分区，但实现可能并不容易分割，特别是如果您的代码库执行大量重用。您可能有许多不设计为公共消费的类，但却在整个代码中使用。

如果没有`InternalsVisibleToAttribute`，跨程序集重用共享实现细节将会很尴尬。每个程序集都需要包含其相关类的副本，或者您需要在某个公共程序集中将它们作为公共类型。第二种技术的问题在于，将类型公开实际上邀请人们使用它们。您的文档可能说明这些类型是框架内部使用的，不应该使用，但这并不能阻止某些人。

幸运的是，您不必将它们设为 `public`。任何只是实现细节的类型可以保持 `internal`，并且您可以使用 `InternalsVisibleToAttribute` 将它们对所有程序集可见，同时使它们对其他人不可访问。

### JIT 编译

影响 JIT 编译器生成代码的几个属性。您可以将`MethodImplAttribute`应用于方法，并传递`Met⁠hod​Imp⁠lOp⁠tions`枚举中的值。其`NoInlining`值确保每当其他方法调用您的方法时，它将成为完整的方法调用。如果没有这个选项，JIT 编译器有时会直接将方法的代码复制到调用代码中。

一般来说，您应该保留内联功能。JIT 编译器仅内联小方法，对于诸如属性访问器之类的简单小方法尤为重要。对于基于字段的简单属性，通过普通函数调用调用访问器通常需要更多代码，而内联优化则可能产生更小、更快的代码。即使代码大小没有减小，它仍然可能更快，因为函数调用可能会意外地昂贵。现代 CPU 倾向于更有效地处理长顺序指令流，而不是跳跃执行位置的代码。然而，内联是一个带有可观测副作用的优化——内联方法不会得到自己的堆栈帧。之前我提到过一些诊断 API，您可以使用它们来检查堆栈，内联将改变报告的堆栈帧数量。如果您只想询问“是哪个方法在调用我？”，则前面描述的调用者信息属性提供了一种更有效的发现方法，不会受内联的影响，但如果您有检查堆栈的代码，有时可能会因内联而混淆。因此，偶尔禁用它是有用的。

相反，您可以指定 `AggressiveInlining`，这鼓励 JIT 编译器内联那些通常作为普通方法调用的内容。如果您确定某个方法对性能非常敏感，尝试这个设置可能会产生不同效果，尽管请注意它可能使代码变慢或变快——这取决于具体情况。相反，您可以使用 `NoOptimization` 选项禁用所有优化（尽管文档暗示这更多是为了微软的 CLR 团队而不是为了消费者，因为它是为了“调试可能的代码生成问题”）。

另一个对优化产生影响的属性是 `DebuggableAttribute`。在调试版本中，C# 编译器会自动将此属性应用到您的程序集中。该属性告诉 CLR 在某些优化方面要更加谨慎，特别是那些影响变量生命周期和代码执行顺序的优化。通常情况下，编译器可以自由更改这些内容，只要最终代码的结果相同即可，但是如果您在调试器中打断点，可能会引起混乱。这个属性确保在这种情况下变量值和执行流程容易跟踪。

### STAThread 和 MTAThread

只运行在 Windows 上并呈现 UI 的应用程序（例如使用 .NET 的 WPF 或 Windows Forms 框架的任何应用程序）通常在它们的 `Main` 方法上有 `[STAThread]` 属性（尽管您不总是看到它，因为入口点通常由这些应用程序的构建系统生成）。这是对 CLR 的互操作服务的一条指令，用于组件对象模型 (COM)，但它有更广泛的影响：如果您希望主线程托管 UI 元素，则需要在 `Main` 方法上放置此属性。

各种 Windows UI 功能都依赖于 COM。例如，剪贴板使用它，某些类型的控件也是如此。COM 有几种线程模型，只有一种与 UI 线程兼容。其中一个主要原因是 UI 元素具有线程关联性，因此 COM 需要确保在正确的线程上执行某些工作。此外，如果 UI 线程不定期检查消息并处理它们，则可能发生死锁。如果您不告诉 COM 特定线程是 UI 线程，它将省略这些检查，您将遇到问题。

###### 注意

即使你不编写 UI 代码，某些互操作场景需要 `[STAThread]` 属性，因为某些 COM 组件在没有它的情况下无法工作。不过，UI 工作是看到它的最常见原因。

由于 CLR 为您管理 COM，CLR 需要知道它应告诉 COM 特定线程需要作为 UI 线程处理。当您显式地使用 Chapter 16 中显示的技术创建新线程时，可以配置其 COM 线程模式，但主线程是一个特例 —— CLR 在应用程序启动时为您创建它，当您的代码运行时，配置线程已经太迟了。在 `Main` 方法上放置 `[STAThread]` 属性告诉 CLR 您的主线程应初始化为兼容 UI 的 COM 行为。

STA 是 *单线程单元* 的缩写。参与 COM 的线程总是属于 STA 或 *多线程单元* (MTA) 中的一种。还有其他类型的单元，但线程只能在其中暂时成员；当线程开始使用 COM 时，必须选择 STA 或 MTA 模式。因此，也有一个 `[MTAThread]` 属性。

### 互操作

CLR 的互操作服务定义了许多属性。其中大多数由 CLR 直接处理，因为互操作是运行时的内在特性。由于这些属性只在它们支持的机制上下文中有意义，并且数量众多，我在这里不会详细描述它们，但 Example 14-19 展示了它们可以做的事情类型。

##### Example 14-19\. 互操作属性

```cs
[DllImport("advapi32.dll", CharSet = CharSet.Unicode, SetLastError = true,
 EntryPoint = "LookupPrivilegeValueW")]
internal static extern bool LookupPrivilegeValue(
 [MarshalAs(UnmanagedType.LPWStr)] string lpSystemName,
 [MarshalAs(UnmanagedType.LPWStr)] string lpName,
    out LUID lpLuid);
```

这里使用了我们之前在示例 14-7 中看到的两个互操作属性，但使用方式稍显复杂。这调用了*advapi32.dll*中的一个函数，它是 Win32 API 的一部分。`DllImport` 属性的第一个参数告诉我们这一点，但与之前的示例不同，这还提供了互操作层的额外信息。这个 API 处理字符串，所以互操作需要知道使用的字符表示法。这个特定的 API 使用了一个常见的 Win32 习惯：它返回一个布尔值以指示成功或失败，但在失败情况下它还使用了 Windows 的 `SetLastError` API 提供更多信息。属性的 `SetLastError` 属性告诉互操作层在调用此 API 后立即检索这些信息，以便 .NET 代码在必要时进行检查。`EntryPoint` 属性处理了 Win32 API 可能有两种形式的字符串参数的事实，可以使用 8 位或 16 位字符（Windows 95 仅支持 8 位文本以节省内存），而我们希望调用*Wide*形式（因此使用 `W` 后缀）。然后，在两个字符串参数上使用 `MarshalAs` 来告诉互操作层，非托管代码中的许多不同字符串表示中，这个特定的 API 需要哪一种。

# 定义和使用属性

绝大多数属性并非运行时或编译器本身固有的，而是由类库定义的，并且只有在使用相关的类库或框架时才会起作用。在你自己的代码中，你可以自由地做完全相同的事情——你可以定义自己的属性类型。因为属性本身不会自行执行任何操作——除非有什么东西要求查看它们，它们甚至不会被实例化——所以通常只在你编写某种类型的框架时定义属性类型才有用，尤其是那些受反射驱动的框架。

例如，单元测试框架经常通过反射发现你编写的测试类，并允许你使用属性控制测试运行程序的行为。另一个例子是 Visual Studio 如何利用反射发现设计表面上可编辑对象（如 UI 控件）的属性，并且它将查找某些属性，这些属性使你可以自定义编辑行为。属性的另一个应用是选择性地排除静态代码分析工具应用的规则。（.NET SDK 包含用于检测代码潜在问题的内置工具。这是一个可扩展的系统，NuGet 包可以添加扩展分析器，扩展这些工具，从而检测特定库可能出现的常见错误。）有时这些工具会出错，你可以通过用属性标注代码来抑制它们的警告。

这里的共同主题是，某些工具或框架会检查你的代码，并根据发现的内容决定要做什么。这种情况正是属性非常适合的场景。例如，如果你编写一个允许最终用户扩展的应用程序，属性可能会很有用。你可能支持加载扩展你应用程序行为的外部程序集——这通常被称为*插件*模型。定义一个允许插件提供有关自身描述信息的属性可能会很有用。使用属性并不是绝对必要的——你可能会定义至少一个接口，所有插件都必须实现该接口，并且可以在该接口中定义用于检索必要信息的成员。然而，使用属性的一个优点是，你不需要创建插件的实例来检索描述信息。这样可以在加载插件之前向用户显示插件的详细信息，如果构建插件可能会产生用户不想要的副作用，这可能很重要。

## 属性类型

示例 14-20 展示了一个包含有关插件信息的属性可能是什么样子。

##### 示例 14-20\. 一个属性类型

```cs
[AttributeUsage(AttributeTargets.Class)]
public class PluginInformationAttribute : Attribute
{
    public PluginInformationAttribute(string name, string author)
    {
        Name = name;
        Author = author;
    }

    public string Name { get; }

    public string Author { get; }

    public string? Description { get; set; }
}
```

要作为属性，类型必须派生自`Attribute`基类。虽然`Attribute`定义了各种静态方法来发现和检索属性，但对于实例来说并没有提供太多有趣的内容。我们并不是从中派生出任何特定功能；我们这样做是因为编译器不会让你使用一个类型作为属性，除非它派生自`Attribute`。

注意到我的类型名称以`Attribute`一词结尾。这不是绝对要求，但这是一个被广泛使用的约定。正如你之前看到的，即使在应用属性时忘记添加`Attribute`后缀，编译器也会自动添加。因此，通常没有理由不遵循这个约定。

我已经用一个属性注释了我的属性类型。大多数属性类型都使用`AttributeUsageAttribute`进行注释，指示属性可以有用地应用到哪些目标上。C#编译器将强制执行这一点。由于我在示例 14-20 中的属性声明只能应用于类，如果有人尝试将其应用于其他任何东西，编译器将生成错误。

###### 注意

正如您所见，有时候当我们应用一个属性时，我们需要说明它的目标。例如，当一个属性出现在方法之前时，它的目标就是该方法，除非您用`return:`前缀加以限定。您可能希望在使用只能针对特定成员的属性时，能够省略这些前缀。例如，如果一个属性只能应用于程序集，您真的需要`assembly:`限定符吗？然而，C# 不允许您省略它。它仅使用`AttributeUsageAttribute`来验证属性未被错误应用。

我的属性只定义了一个构造函数，因此任何使用它的代码都必须传递构造函数所需的参数，就像示例 14-21 所示。

##### Example 14-21\. 应用属性

```cs
[PluginInformation("Reporting", "Endjin Ltd.")]
public class ReportingPlugin
{
    ...
}
```

属性类可以自由地定义多个构造函数重载，以支持不同的信息集。它们还可以定义属性作为支持可选信息的一种方式。我的属性定义了一个`Description`属性，这不是必需的，因为构造函数不要求为其提供值，但我可以使用本章前面描述的语法来设置它。示例 14-22 展示了我属性的样子。

##### Example 14-22\. 为属性提供可选的属性值

```cs
[PluginInformation("Reporting", "Endjin Ltd.", `Description = "Automated report generation")]`
public class ReportingPlugin
{
    ...
}
```

到目前为止，我展示的内容不会创建`Plu⁠gin⁠Inf⁠orm⁠ation​Att⁠rib⁠ute`类型的实例。这些注解只是指示，告诉属性在需要时应如何初始化。因此，如果要使用此属性，我需要编写一些代码来查找它。

## 获取属性

您可以使用反射 API 来发现特定类型的属性是否已经应用，并且反射 API 也可以为您实例化该属性。在第十三章中，我展示了表示各种属性目标的反射类型，例如`MethodInfo`、`Type`和`PropertyInfo`。它们都实现了一个名为`ICustomAttributeProvider`的接口，如示例 14-23 所示。

##### Example 14-23\. `ICustomAttributeProvider`

```cs
public interface ICustomAttributeProvider
{
    object[] GetCustomAttributes(bool inherit);
    object[] GetCustomAttributes(Type attributeType, bool inherit);
    bool IsDefined(Type attributeType, bool inherit);
}
```

`IsDefined`方法仅告诉您特定属性类型是否存在，而不会实例化它。两个`GetCustomAttributes`重载会创建属性并返回它们。（这是属性被构造以及设置任何注解指定的属性的时机。）第一个重载返回应用于目标的所有属性，而第二个重载允许您仅请求特定类型的属性。

所有这些方法都接受一个`bool`参数，让您可以指定是否仅希望获取直接应用于您正在检查的目标的属性，或者还包括应用于基类型或类型的属性。

这个接口在 .NET 1.0 中引入，因此它不使用泛型，这意味着您需要对返回的对象进行强制转换。幸运的是，`Cus⁠tom⁠Att⁠rib⁠ute​Ext⁠ens⁠ions` 静态类定义了几个扩展方法。它并不是为 `ICustomAttributeProvider` 接口定义这些方法，而是扩展了提供属性的反射类。例如，如果您有一个 `Type` 类型的变量，您可以在其上调用 `GetCustomAttribute<PluginInformationAttribute>()` 方法，这将构造并返回插件信息属性，如果该属性不存在，则返回 `null`。示例 14-24 使用此方法显示了特定文件夹中所有 DLL 的所有插件信息。

##### 示例 14-24\. 显示插件信息

```cs
static void ShowPluginInformation(string pluginFolder)
{
    var dir = new DirectoryInfo(pluginFolder);
    foreach (FileInfo file in dir.GetFiles("*.dll"))
    {
        Assembly pluginAssembly = Assembly.LoadFrom(file.FullName);
        var plugins =
             from type in pluginAssembly.ExportedTypes
             `let` `info` `=` `type``.``GetCustomAttribute``<``PluginInformationAttribute``>``(``)`
             where info != null
             select new { type, info };

        foreach (var plugin in plugins)
        {
            Console.WriteLine($"Plugin type: {plugin.type.Name}");
            Console.WriteLine(
                $"Name: {plugin.info.Name}, written by {plugin.info.Author}");
            Console.WriteLine($"Description: {plugin.info.Description}");
        }
    }
}

```

这样做可能会存在一个潜在问题。我之前说过属性的一个好处是，可以在不实例化它们的目标类型的情况下检索它们。在这里是正确的 —— 我没有在 示例 14-24 中构造任何插件。然而，我正在加载插件程序集，枚举插件的一个可能副作用是运行插件 DLL 中的静态构造函数。因此，虽然我没有故意在这些 DLL 中运行任何代码，但我不能保证这些 DLL 中的代码不会运行。如果我的目标是向用户呈现插件列表，并且仅加载和运行明确选择的插件，那么我已经失败了，因为我给插件代码一个运行的机会。不过，我们可以解决这个问题。

## 仅加载元数据（Metadata-Only Load）

您不需要完全加载一个程序集才能检索属性信息。正如我在 第十三章 中讨论的那样，您可以仅为反射目的加载程序集，使用 `MetadataLoadContext` 类来实现。这样可以防止程序集中的任何代码运行，但可以检查其包含的类型。不过，这对属性造成了挑战。通常检查属性的属性的方法是通过调用 `GetCustomAttributes` 或相关的扩展方法来实例化它。由于这涉及构造属性 —— 也就是运行某些代码 —— 对于由 `MetadataLoadContext` 加载的程序集是不支持的（即使涉及的属性类型是在以正常方式完全加载的不同程序集中定义的）。如果我修改了 示例 14-24 来使用 `Met⁠ada⁠ta​Loa⁠dCo⁠ntext` 加载程序集，调用 `Get⁠Cus⁠tom⁠Att⁠rib⁠ute⁠<Pl⁠ugi⁠nIn⁠for⁠mat⁠ion​Att⁠rib⁠ute>` 将会抛出异常。

当仅加载元数据时，您必须使用`GetCustomAttributesData`方法。这不会为您实例化属性，而是返回存储在元数据中的信息——用于创建属性的指令。Example 14-25 展示了从 Example 14-24 修改为使用这种方式的相关代码版本。（还包括初始化`MetadataLoadContext`所需的代码。）

##### Example 14-25\. 使用`MetadataLoadContext`检索属性

```cs
string[] runtimeAssemblies = Directory.GetFiles(
    RuntimeEnvironment.GetRuntimeDirectory(), "*.dll");
var paths = new List<string>(runtimeAssemblies);
paths.Add(file.FullName);

var resolver = new PathAssemblyResolver(paths);
var mlc = new MetadataLoadContext(resolver);

Assembly pluginAssembly = mlc.LoadFromAssemblyPath(file.FullName);
var plugins =
     from type in pluginAssembly.ExportedTypes
     `let` `info` `=` `type``.``GetCustomAttributesData``(``)``.``SingleOrDefault``(``attrData` `=``>`
            `attrData``.``AttributeType``.``FullName` `=``=` `pluginAttributeType``.``FullName``)`
     where info != null
     let description = info.NamedArguments
                           .SingleOrDefault(a => a.MemberName == "Description")
     select new
     {
         type,
         Name = (string) info.ConstructorArguments[0].Value,
         Author = (string) info.ConstructorArguments[1].Value,
         Description =
             description == null ? null : description.TypedValue.Value
     };

foreach (var plugin in plugins)
{
    Console.WriteLine($"Plugin type: {plugin.type.Name}");
    Console.WriteLine($"Name: {plugin.Name}, written by {plugin.Author}");
    Console.WriteLine($"Description: {plugin.Description}");
}
```

代码非常繁琐，因为我们没有得到属性的实例。`GetCustomAttributesData`返回一个`CustomAttributeData`对象的集合。Example 14-25 使用 LINQ 的`SingleOrDefault`操作符来查找`PluginInformationAttribute`的条目，如果存在，查询中的`info`变量将持有与相关`CustomAttributeData`对象的引用。然后，代码通过`ConstructorArguments`和`NamedArguments`属性逐步选择构造函数参数和属性值，从而能够检索嵌入在属性中的三个描述性文本值。

正如这个例子所示，`MetadataLoadContext`增加了复杂性，因此只有在需要它提供的好处时才应该使用它。其中一个好处是它不会运行您加载的任何程序集。它还可以加载通常会被拒绝加载的程序集（例如，因为它们针对的特定处理器架构与您的进程不匹配）。但是，如果您不需要仅仅是元数据选项，直接访问属性，如 Example 14-24 所示，会更加方便。

# 总结

属性为将自定义数据嵌入程序集元数据提供了一种方法。您可以将属性应用于类型、类型的任何成员、参数、返回值，甚至整个程序集或其模块之一。一些属性由 CLR 特别处理，还有一些控制编译器功能，但大多数属性没有固有行为，仅充当被动信息容器。除非有人要求查看它们，否则属性甚至不会被实例化。所有这些使得属性在反射驱动行为的系统中最为有用——如果您已经有了反射 API 对象，比如`ParameterInfo`或`Type`，您可以直接向其询问属性。因此，您最常见到的是在检查您的代码的框架中使用属性，比如单元测试框架、序列化框架、像 Visual Studio 属性面板这样的数据驱动 UI 元素，或者插件框架。如果您使用这类框架，通常可以通过用框架识别的属性注解代码来配置其行为。如果您正在编写这种类型的框架，那么定义自己的属性类型可能是有意义的。
