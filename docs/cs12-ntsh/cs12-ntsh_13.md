# 第十三章\. 诊断

当出现问题时，重要的是有信息可用以帮助诊断问题。集成开发环境（IDE）或调试器可以极大地帮助此过程，但通常仅在开发期间可用。应用程序发布后，必须收集和记录诊断信息。为了满足此要求，.NET 提供了一组设施来记录诊断信息，监视应用程序行为，检测运行时错误，并在可能时与调试工具集成。

一些诊断工具和 API 是特定于 Windows 的，因为它们依赖于 Windows 操作系统的功能。为了防止平台特定的 API 混乱.NET BCL，微软已经将它们封装在单独的 NuGet 包中，您可以选择性地引用它们。有超过一打的特定于 Windows 的包，您可以使用 *Microsoft.Windows.Compatibility* “主” 包一次引用它们。

本章中的类型主要在 `System.Diagnostics` 命名空间中定义。

# 条件编译

您可以使用*预处理指令*在 C# 中有条件地编译任何代码段。预处理指令是以 `#` 符号开头的特殊指令（与其他 C# 结构不同，必须单独出现在一行上）。逻辑上，在主编译之前执行它们（尽管在实践中，编译器在词法分析阶段处理它们）。用于条件编译的预处理指令包括 `#if`、`#else`、`#endif` 和 `#elif`。

`#if` 指令指示编译器在指定的*符号*已被定义时忽略代码段。您可以通过使用 `#define` 指令在源代码中定义符号（在这种情况下，符号仅适用于该文件），或者在 *.csproj* 文件中使用 `<DefineConstants>` 元素（在这种情况下，符号适用于整个程序集）：

```cs
#define TESTMODE            // #define directives must be at top of file
                            // Symbol names are uppercase by convention.
using System;

class Program
{
  static void Main()
  {
#if TESTMODE
    Console.WriteLine ("in test mode!");     // OUTPUT: in test mode!
#endif
  }
}
```

如果我们删除第一行，则程序将编译，`Console.WriteLine` 语句将完全从可执行文件中删除，就像被注释掉一样。

`#else` 语句类似于 C# 的 `else` 语句，而 `#elif` 相当于 `#else` 后跟 `#if`。`||`、`&&` 和 `!` 运算符执行*或*、*与* 和 *非* 操作：

```cs
#if TESTMODE && !PLAYMODE      // if TESTMODE and not PLAYMODE
  ...
```

但请记住，您并不是在构建普通的 C# 表达式，您操作的符号与*变量*（静态或其他）没有任何关联。

可以通过编辑 *.csproj* 文件（或在 Visual Studio 中，在项目属性窗口的“生成”选项卡中）定义适用于程序集中每个文件的符号。以下定义了两个常量，`TESTMODE` 和 `PLAYMODE`：

```cs
<PropertyGroup>
  <DefineConstants>TESTMODE;PLAYMODE</DefineConstants>
</PropertyGroup>
```

如果您在程序集级别定义了一个符号，然后希望在特定文件中“取消定义”它，可以使用 `#undef` 指令。

## 条件编译与静态变量标志对比

您可以使用简单的静态字段来实现前面的示例：

```cs
static internal bool TestMode = true;

static void Main()
{
  if (TestMode) Console.WriteLine ("in test mode!");
}
```

这具有允许运行时配置的优点。那么，为什么选择条件编译？原因在于条件编译可以带你到变量标志无法到达的地方，比如以下情况：

+   条件包含属性

+   更改变量的声明类型

+   在`using`指令中切换不同的命名空间或类型别名；例如：

    ```cs
    using TestType =
      #if V2
         MyCompany.Widgets.GadgetV2;
      #else
         MyCompany.Widgets.Gadget;
      #endif
    ```

甚至可以在条件编译指令下执行重大重构，因此您可以即时在旧版和新版之间切换，并编写可以针对多个运行时版本进行编译的库，在可用时利用最新功能。

条件编译的另一个优点是调试代码可以引用部署中未包含的程序集中的类型。

## 条件属性

`Conditional`属性指示编译器忽略对特定类或方法的所有调用，如果指定的符号未被定义。

要看到这对您有多有用，请假设您编写了一个用于记录状态信息的方法，如下所示：

```cs
static void LogStatus (string msg)
{
  string logFilePath = ...
  System.IO.File.AppendAllText (logFilePath, msg + "\r\n");
}
```

现在想象一下，您只希望在定义了`LOGGINGMODE`符号时执行此操作。第一种解决方案是将对`LogStatus`的所有调用都包装在`#if`指令中：

```cs
#if LOGGINGMODE
LogStatus ("Message Headers: " + GetMsgHeaders());
#endif
```

这样可以得到理想的结果，但是这很繁琐。第二种解决方案是将`#if`指令放在`LogStatus`方法内部。然而，如果以以下方式调用`LogStatus`，这将会有问题：

```cs
LogStatus ("Message Headers: " + GetComplexMessageHeaders());
```

`GetComplexMessageHeaders`将始终被调用，这可能会带来性能损失。

我们可以通过将`Conditional`属性（定义在`System.Diagnostics`中）附加到`LogStatus`方法来将第一种解决方案的功能与第二种解决方案的便利性结合起来：

```cs
[Conditional ("LOGGINGMODE")]
static void LogStatus (string msg)
{
  ...
}
```

这会指示编译器将对`LogStatus`的调用视为包含在`#if LOGGINGMODE`指令中。如果未定义该符号，则在编译中将完全消除对`LogStatus`的任何调用，包括它们的参数评估表达式。（因此，任何具有副作用的表达式将被跳过。）即使`LogStatus`和调用方位于不同的程序集中，这也适用。

###### 注意

`[Conditional]`的另一个好处是条件检查是在*调用方*编译时进行的，而不是在*被调用方法*编译时进行的。这很有益，因为它允许您编写包含诸如`LogStatus`之类方法的库，并且只构建该库的一个版本。

`Conditional`属性在运行时被忽略——它纯粹是对编译器的指示。

### 条件属性的替代方案

如果需要在运行时动态启用或禁用功能，则`Conditional`属性无效：相反，必须使用基于变量的方法。这留下了在调用条件日志方法时如何优雅地避免参数评估的问题。函数方法可以解决这个问题：

```cs
using System;
using System.Linq;

class Program
{
  public static bool EnableLogging;

  static void LogStatus (Func<string> message)
  {
    string logFilePath = ...
    if (EnableLogging)
      System.IO.File.AppendAllText (logFilePath, message() + "\r\n");
  }
}
```

使用 lambda 表达式可以在不增加语法复杂性的情况下调用此方法：

```cs
LogStatus ( () => "Message Headers: " + GetComplexMessageHeaders() );
```

如果`EnableLogging`为`false`，则`GetComplexMessageHeaders`永远不会被评估。

# 调试和跟踪类

`Debug`和`Trace`是提供基本日志记录和断言能力的静态类。这两个类非常相似；主要区别在于它们的预期用途。`Debug`类用于调试版本；`Trace`类用于调试和发布版本。为此：

```cs
All methods of the Debug class are defined with [Conditional("DEBUG")].
All methods of the Trace class are defined with [Conditional("TRACE")].
```

这意味着，除非定义了`DEBUG`或`TRACE`符号，否则编译器将消除对`Debug`或`Trace`的所有调用（Visual Studio 在项目属性的构建选项卡中提供了定义这些符号的复选框，并默认使用新项目启用 TRACE 符号）。

`Debug`和`Trace`类都提供`Write`、`WriteLine`和`WriteIf`方法。默认情况下，这些方法将消息发送到调试器的输出窗口：

```cs
Debug.Write     ("Data");
Debug.WriteLine (23 * 34);
int x = 5, y = 3;
Debug.WriteIf   (x > y, "x is greater than y");
```

`Trace`类还提供`TraceInformation`、`TraceWarning`和`TraceError`方法。这些方法与`Write`方法的行为差异取决于活动的`TraceListener`（我们在`TraceListener`中介绍了这一点）。

## 失败和断言

`Debug`和`Trace`类都提供`Fail`和`Assert`方法。`Fail`将消息发送给`Debug`或`Trace`类的`Listeners`集合中的每个`TraceListener`（请参见下一节），默认情况下将消息写入调试输出：

```cs
Debug.Fail ("File data.txt does not exist!");
```

`Assert`如果`bool`参数为`false`，则简单调用`Fail`，这被称为*进行断言*，如果违反则表示代码中存在 bug。指定失败消息是可选的：

```cs
Debug.Assert (File.Exists ("data.txt"), "File data.txt does not exist!");
var result = ...
Debug.Assert (result != null);
```

除了消息外，`Write`、`Fail`和`Assert`方法还可以重载接受一个`string`类别，这在处理输出时非常有用。

另一种断言的替代方法是，如果相反条件为真，则抛出异常。这在验证方法参数时是一种常见做法：

```cs
public void ShowMessage (string message)
{
  if (message == null) throw new ArgumentNullException ("message");
  ...
}
```

此类“断言”被无条件编译，而且在控制失败断言的结果上不够灵活，无法通过`TraceListener`来控制。严格来说，它们并不是断言。断言是指当前方法代码中的 bug，如果违反则表明存在问题。基于参数验证抛出异常表明调用者的代码中存在 bug。

## TraceListener

`Trace`类有一个静态的`Listeners`属性，返回一个`TraceListener`实例的集合。这些负责处理`Write`、`Fail`和`Trace`方法发出的内容。

默认情况下，每个的`Listeners`集合都包含一个监听器（`Default​Tra⁠ceListener`）。默认监听器有两个关键特性：

+   当连接到诸如 Visual Studio 之类的调试器时，消息将被写入调试输出窗口；否则，将忽略消息内容。

+   当调用`Fail`方法（或断言失败）时，应用程序将终止。

您可以通过（可选地）移除默认侦听器并添加一个或多个自定义侦听器来更改此行为。您可以从头编写跟踪侦听器（通过子类化`TraceListener`）或使用预定义类型之一：

+   `TextWriterTraceListener`写入到`Stream`或`TextWriter`或附加到文件。

+   `EventLogTraceListener`写入到 Windows 事件日志（仅限 Windows）。

+   `EventProviderTraceListener`写入到 Windows 事件跟踪（ETW）子系统（跨平台支持）。

`TextWriterTraceListener`进一步被子类化为`ConsoleTraceListener`、`DelimitedListTraceListener`、`XmlWriterTraceListener`和`EventSchemaTraceListener`。

以下示例清除`Trace`的默认侦听器，然后添加三个侦听器——一个附加到文件，一个写入控制台，一个写入 Windows 事件日志：

```cs
// Clear the default listener:
Trace.Listeners.Clear();

// Add a writer that appends to the trace.txt file:
Trace.Listeners.Add (new TextWriterTraceListener ("trace.txt"));

// Obtain the Console's output stream, then add that as a listener:
System.IO.TextWriter tw = Console.Out;
Trace.Listeners.Add (new TextWriterTraceListener (tw));

// Set up a Windows Event log source and then create/add listener.
// CreateEventSource requires administrative elevation, so this would
// typically be done in application setup.
if (!EventLog.SourceExists ("DemoApp"))
  EventLog.CreateEventSource ("DemoApp", "Application");

Trace.Listeners.Add (new EventLogTraceListener ("DemoApp"));
```

对于 Windows 事件日志，使用`Write`、`Fail`或`Assert`方法写入的消息始终在 Windows 事件查看器中显示为“信息”消息。但是，通过`TraceWarning`和`TraceError`方法写入的消息将显示为警告或错误。

`TraceListener`还具有`TraceFilter`类型的`Filter`，您可以设置该过滤器以控制是否将消息写入该侦听器。为此，您可以实例化预定义的子类（如`EventTypeFilter`或`SourceFilter`），或者子类化`TraceFilter`并重写`ShouldTrace`方法。例如，您可以使用这个功能按类别进行过滤。

`TraceListener`还定义了用于控制缩进的`IndentLevel`和`IndentSize`属性，以及用于写入额外数据的`TraceOutputOptions`属性：

```cs
TextWriterTraceListener tl = new TextWriterTraceListener (Console.Out);
tl.TraceOutputOptions = TraceOptions.DateTime | TraceOptions.Callstack;
```

使用`Trace`方法时将应用`TraceOutputOptions`：

```cs
Trace.TraceWarning ("Orange alert");

*DiagTest.vshost.exe Warning: 0 : Orange alert*
 *DateTime=2007-03-08T05:57:13.6250000Z*
 *Callstack=   at System.Environment.GetStackTrace(Exception e, Boolean*
*needFileInfo)*
 *at System.Environment.get_StackTrace()     at ...*
```

## 刷新和关闭侦听器

一些侦听器（如`TextWriterTraceListener`）最终会写入到可能受缓存影响的流中。这有两个影响：

+   消息可能不会立即显示在输出流或文件中。

+   您必须在应用程序结束之前关闭——或至少刷新——侦听器；否则，您将丢失缓存中的内容（默认情况下最多 4 KB，如果写入文件）。

`Trace`和`Debug`类提供静态的`Close`和`Flush`方法，这些方法会调用所有侦听器的`Close`或`Flush`（进而调用底层的编写器和流的`Close`或`Flush`）。`Close`隐含调用`Flush`，关闭文件句柄，并防止进一步写入数据。

通常建议在应用程序结束之前调用`Close`，并在希望确保当前消息数据已写入时调用`Flush`。如果使用基于流或文件的侦听器，则适用此规则。

`Trace`和`Debug`还提供`AutoFlush`属性，如果设置为`true`，则在每条消息后强制执行`Flush`。

###### 注意

如果使用任何基于文件或流的侦听器，则将`AutoFlush`设置为`true`是一个好策略，否则，如果发生未处理的异常或关键错误，则可能会丢失最后的 4 KB 诊断信息。

# 调试器集成

有时，如果可用，应用程序与调试器进行交互是有用的。在开发过程中，调试器通常是你的 IDE（如 Visual Studio）；在部署中，调试器更可能是较低级别的调试工具之一，如 WinDbg、Cordbg 或 MDbg。

## 附加和中断

`System.Diagnostics`中的静态`Debugger`类提供了与调试器交互的基本功能，即`Break`、`Launch`、`Log`和`IsAttached`。

调试器必须先附加到一个应用程序才能进行调试。如果你从 IDE 内部启动应用程序，这将自动发生，除非你另有要求（选择“不带调试启动”）。但有时，在 IDE 内部以调试模式启动应用程序可能不方便或不可能。例如，Windows 服务应用程序或（具有讽刺意味的是）Visual Studio 设计器。一个解决方案是正常启动应用程序，然后在 IDE 中选择“调试进程”。但这样做不能让你在程序执行早期设置断点。

解决方法是在应用程序内部调用`Debugger.Break`。这个方法会启动调试器，附加到它，并在那一点暂停执行（`Launch`做同样的事情，但不会暂停执行）。附加后，你可以使用`Log`方法直接向调试器的输出窗口记录消息。你可以通过检查`IsAttached`属性验证是否已连接到调试器。

## 调试器属性

`DebuggerStepThrough`和`DebuggerHidden`属性为调试器提供了关于如何处理特定方法、构造函数或类的单步执行建议。

`DebuggerStepThrough` 请求调试器在没有任何用户交互的情况下步入函数。这个属性在自动生成的方法和代理方法中特别有用，后者将实际工作转发到其他位置的方法。在后一种情况下，如果在“真实”方法内设置了断点，调试器仍会显示代理方法在调用堆栈中，除非你还添加了`DebuggerHidden`属性。你可以在代理上结合这两个属性，帮助用户专注于调试应用逻辑而不是底层管理：

```cs
[DebuggerStepThrough, DebuggerHidden]
void DoWorkProxy()
{
  // setup...
  DoWork();
  // teardown...
}

void DoWork() {...}   // Real method...
```

# 进程和进程线程

我们在第六章的最后一节中描述了如何使用`Process.Start`启动新进程。`Process`类还允许你查询和与运行在同一台或另一台计算机上的其他进程进行交互。`Process`类是.NET Standard 2.0 的一部分，尽管其功能对于 UWP 平台有所限制。

## 检查运行中的进程

`Process.GetProcess*XXX*` 方法通过名称或进程 ID 检索特定进程，或检索当前计算机或指定计算机上运行的所有进程。这包括托管和非托管进程。每个 `Process` 实例具有丰富的属性，映射统计信息，如名称、ID、优先级、内存和处理器利用率、窗口句柄等。以下示例枚举了当前计算机上所有正在运行的进程：

```cs
foreach (Process p in Process.GetProcesses())
using (p)
{
  Console.WriteLine (p.ProcessName);
  Console.WriteLine ("   PID:      " + p.Id);
  Console.WriteLine ("   Memory:   " + p.WorkingSet64);
  Console.WriteLine ("   Threads:  " + p.Threads.Count);
}
```

`Process.GetCurrentProcess`返回当前进程。

您可以通过调用其 `Kill` 方法终止进程。

## 检查进程中的线程

您也可以使用`Process.Threads`属性枚举其他进程的线程。但是，您获取的对象不是`System.Threading.Thread`对象；它们是`ProcessThread`对象，用于管理而不是同步任务。`ProcessThread`对象提供有关底层线程的诊断信息，并允许您控制一些方面，如其优先级和处理器亲和性：

```cs
public void EnumerateThreads (Process p)
{
  foreach (ProcessThread pt in p.Threads)
  {
    Console.WriteLine (pt.Id);
    Console.WriteLine ("   State:    " + pt.ThreadState);
    Console.WriteLine ("   Priority: " + pt.PriorityLevel);
    Console.WriteLine ("   Started:  " + pt.StartTime);
    Console.WriteLine ("   CPU time: " + pt.TotalProcessorTime);
  }
}
```

# StackTrace 和 StackFrame

`StackTrace` 和 `StackFrame` 类提供了执行调用堆栈的只读视图。您可以为当前线程或 `Exception` 对象获取堆栈跟踪信息。这些信息主要用于诊断目的，尽管您也可以在编程（黑客）中使用它。`StackTrace` 表示完整的调用堆栈；`StackFrame` 表示该堆栈内的单个方法调用。

###### 注意

如果你只需知道调用方法的名称和行号，调用者信息属性可以提供更简单和更快的替代方法。我们在“调用者信息属性”中讨论了这个话题。

如果您使用不带参数或带有`bool`参数的`StackTrace`对象进行实例化，您将获得当前线程调用堆栈的快照。如果`bool`参数为`true`，则`StackTrace`将读取程序集的 *.pdb*（项目调试）文件（如果存在），从而使您能够访问文件名、行号和列偏移数据。在使用 `/debug` 开关进行编译时生成项目调试文件。（Visual Studio 默认使用此开关进行编译，除非您通过 *高级构建设置* 请求否定。）

获得`StackTrace`后，可以通过调用`GetFrame`来检查特定的帧，或者通过使用`GetFrames`获取整个堆栈：

```cs
static void Main() { A (); }
static void A()    { B (); }
static void B()    { C (); }
static void C()
{
  StackTrace s = new StackTrace (true);

  Console.WriteLine ("Total frames:   " + s.FrameCount);
  Console.WriteLine ("Current method: " + s.GetFrame(0).GetMethod().Name);
  Console.WriteLine ("Calling method: " + s.GetFrame(1).GetMethod().Name);
  Console.WriteLine ("Entry method:   " + s.GetFrame
                                       (s.FrameCount-1).GetMethod().Name);
  Console.WriteLine ("Call Stack:");
  foreach (StackFrame f in s.GetFrames())
    Console.WriteLine (
      "  File: "   + f.GetFileName() +
      "  Line: "   + f.GetFileLineNumber() +
      "  Col: "    + f.GetFileColumnNumber() +
      "  Offset: " + f.GetILOffset() +
      "  Method: " + f.GetMethod().Name);
}
```

下面是输出：

```cs
Total frames:   4
Current method: C
Calling method: B
Entry method: Main
Call stack:
  File: C:\Test\Program.cs  Line: 15  Col: 4  Offset: 7  Method: C
  File: C:\Test\Program.cs  Line: 12  Col: 22  Offset: 6  Method: B
  File: C:\Test\Program.cs  Line: 11  Col: 22  Offset: 6  Method: A
  File: C:\Test\Program.cs  Line: 10  Col: 25  Offset: 6  Method: Main
```

###### 注意

中间语言（IL）偏移量指示将执行*下一个*指令的偏移量 —— 而不是当前正在执行的指令。奇怪的是，如果存在 *.pdb* 文件，行和列号通常指示实际执行点。

这是因为 CLR 尽其所能推断出从 IL 偏移计算行和列的实际执行点。编译器以使这种可能成为可能的方式发出 IL —— 包括在 IL 流中插入 `nop`（无操作）指令。

使用启用优化编译时，将禁用插入`nop`指令，因此堆栈跟踪可能显示下一条要执行的语句的行号和列号。获得有用的堆栈跟踪进一步受到优化可以引入的其他技巧的阻碍，包括折叠整个方法。

获取整个`StackTrace`的基本信息的快捷方式是对其调用`ToString`。以下是结果的样子：

```cs
   at DebugTest.Program.C() in C:\Test\Program.cs:line 16
   at DebugTest.Program.B() in C:\Test\Program.cs:line 12
   at DebugTest.Program.A() in C:\Test\Program.cs:line 11
   at DebugTest.Program.Main() in C:\Test\Program.cs:line 10
```

您还可以通过将`Exception`对象传递给`StackTrace`的构造函数来获取异常对象（显示导致抛出异常的内容）的堆栈跟踪。

###### 注意

`Exception`已经有一个`StackTrace`属性；然而，该属性返回一个简单的字符串，而不是`StackTrace`对象。在没有*.pdb*文件可用的部署后发生异常时，IL 偏移量比行号和列号更有用。使用 IL 偏移量和*ildasm*，您可以准确定位发生错误的方法内部位置。

# Windows 事件日志

Win32 平台提供了一个集中的日志记录机制，以 Windows 事件日志的形式存在。

我们之前使用的`Debug`和`Trace`类如果注册了`EventLogTraceListener`，则写入 Windows 事件日志。然而，使用`EventLog`类，您可以直接将数据写入 Windows 事件日志，而无需使用`Trace`或`Debug`。您还可以使用此类来读取和监视事件数据。

###### 注意

在 Windows 服务应用程序中写入 Windows 事件日志是有意义的，因为如果发生问题，您无法弹出用户界面，将用户引导到某个特殊文件中，其中包含已写入的诊断信息。此外，由于服务通常写入 Windows 事件日志是一种常见做法，如果您的服务停止运行，管理员可能会首先查看此处。

有三个标准的 Windows 事件日志，它们分别由以下名称标识：

+   应用程序

+   系统

+   安全

应用程序日志通常是大多数应用程序写入的地方。

## 写入事件日志

要写入 Windows 事件日志：

1.  选择三个事件日志之一（通常是*应用程序*）。

1.  决定一个*源名称*，如果需要则创建（创建需要管理员权限）。

1.  使用日志名称、源名称和消息数据调用`EventLog.WriteEntry`。

*源名称*是您的应用程序的易于识别的名称。您必须在使用之前注册源名称——`CreateEventSource`方法执行此功能。然后可以调用`WriteEntry`：

```cs
const string SourceName = "MyCompany.WidgetServer";

// CreateEventSource requires administrative permissions, so this would
// typically be done in application setup.
if (!EventLog.SourceExists (SourceName))
  EventLog.CreateEventSource (SourceName, "Application");

EventLog.WriteEntry (SourceName,
  "Service started; using configuration file=...",
  EventLogEntryType.Information);
```

`EventLogEntryType`可以是`Information`、`Warning`、`Error`、`SuccessAudit`或`FailureAudit`。每种类型在 Windows 事件查看器中显示不同的图标。您还可以选择性地指定类别和事件 ID（每个都是您自己选择的数字），并提供可选的二进制数据。

`CreateEventSource`还允许您指定计算机名称：这是为了将日志写入另一台计算机的事件日志（如果您具有足够的权限）。

## 读取事件日志

要读取事件日志，请使用要访问的日志名称和（可选）日志所在计算机的名称实例化`EventLog`类。然后可以通过`Entries`集合属性读取每个日志条目：

```cs
EventLog log = new EventLog ("Application");

Console.WriteLine ("Total entries: " + log.Entries.Count);

EventLogEntry last = log.Entries [log.Entries.Count - 1];
Console.WriteLine ("Index:   " + last.Index);
Console.WriteLine ("Source:  " + last.Source);
Console.WriteLine ("Type:    " + last.EntryType);
Console.WriteLine ("Time:    " + last.TimeWritten);
Console.WriteLine ("Message: " + last.Message);
```

您可以通过静态方法`EventLog.GetEventLogs`（这需要管理员权限以获取完全访问权限）枚举当前（或另一个）计算机的所有日志：

```cs
foreach (EventLog log in EventLog.GetEventLogs())
  Console.WriteLine (log.LogDisplayName);
```

这通常至少打印*应用程序*、*安全性*和*系统*。

## 监视事件日志

你可以通过`EntryWritten`事件在 Windows 事件日志中写入条目时得到提醒。这适用于本地计算机上的事件日志，并且不管哪个应用程序记录了事件，都会触发此事件。

要启用日志监视：

1.  实例化一个`EventLog`并将其`EnableRaisingEvents`属性设置为`true`。

1.  处理`EntryWritten`事件。

例如：

```cs
using (var log = new EventLog ("Application"))
{
  log.EnableRaisingEvents = true;
  log.EntryWritten += DisplayEntry;
  Console.ReadLine();
}

void DisplayEntry (object sender, EntryWrittenEventArgs e)
{
  EventLogEntry entry = e.Entry;
  Console.WriteLine (entry.Message);
}
```

# 性能计数器

###### 注意

性能计数器是仅适用于 Windows 的功能，需要 NuGet 包`System.Diagnostics​.PerformanceCounter`。如果您的目标是 Linux 或 macOS，请参阅“跨平台诊断工具”以获取替代方案。

我们迄今讨论的日志记录机制对于捕获未来分析的信息很有用。但是，要深入了解应用程序（或整个系统）的当前状态，需要一种更实时的方法。Win32 解决这种需求的方案是性能监控基础设施，它由系统和应用程序公开的一组性能计数器以及用于实时监控这些计数器的 Microsoft Management Console（MMC）插件组成。

性能计数器分为类别，如“系统”、“处理器”、“.NET CLR 内存”等。这些类别有时也被图形用户界面（GUI）工具称为“性能对象”。每个类别分组了一组相关的性能计数器，用于监视系统或应用程序的一个方面。在“.NET CLR 内存”类别中，性能计数器的示例包括“%GC 时间”、“所有堆中的字节数”和“每秒分配的字节数”。

每个类别可以选择性地具有一个或多个可以独立监视的实例。例如，在“处理器”类别的“%处理器时间”性能计数器中，这在监视 CPU 利用率时非常有用。在多处理器计算机上，此计数器支持每个 CPU 的实例，允许您独立监视每个 CPU 的利用率。

以下各节说明了如何执行常见的任务，例如确定公开的计数器、监视计数器以及创建自己的计数器以公开应用程序状态信息。

###### 注意

读取性能计数器或类别可能需要在本地或目标计算机上具有管理员权限，这取决于所访问的内容。

## 枚举可用计数器

以下示例枚举计算机上所有可用的性能计数器。对于具有实例的计数器，它枚举每个实例的计数器：

```cs
PerformanceCounterCategory[] cats =
  PerformanceCounterCategory.GetCategories();

foreach (PerformanceCounterCategory cat in cats)
{
  Console.WriteLine ("Category: " + cat.CategoryName);

  string[] instances = cat.GetInstanceNames();
  if (instances.Length == 0)
  {
    foreach (PerformanceCounter ctr in cat.GetCounters())
      Console.WriteLine ("  Counter: " + ctr.CounterName);
  }
  else   // Dump counters with instances
  {
    foreach (string instance in instances)
    {
      Console.WriteLine ("  Instance: " + instance);
      if (cat.InstanceExists (instance))
        foreach (PerformanceCounter ctr in cat.GetCounters (instance))
          Console.WriteLine ("    Counter: " + ctr.CounterName);
    }
  }
}
```

###### 注意

结果超过 10,000 行长！执行起来也需要一些时间，因为 `PerformanceCounter​Cate⁠gory.Instance​Exists` 的实现效率低下。在实际系统中，您希望仅在需要时检索更详细的信息。

下一个示例使用 LINQ 仅检索 .NET 性能计数器，并将结果写入 XML 文件：

```cs
var x =
  new XElement ("counters",
    from PerformanceCounterCategory cat in
         PerformanceCounterCategory.GetCategories()
    where cat.CategoryName.StartsWith (".NET")
    let instances = cat.GetInstanceNames()
    select new XElement ("category",
      new XAttribute ("name", cat.CategoryName),
      instances.Length == 0
      ?
        from c in cat.GetCounters()
        select new XElement ("counter",
          new XAttribute ("name", c.CounterName))
      :
        from i in instances
        select new XElement ("instance", new XAttribute ("name", i),
          !cat.InstanceExists (i)
          ?
            null
          :
            from c in cat.GetCounters (i)
            select new XElement ("counter",
              new XAttribute ("name", c.CounterName))
        )
    )
  );
x.Save ("counters.xml");
```

## 读取性能计数器数据

要检索性能计数器的值，请实例化 `PerformanceCounter` 对象，然后调用 `NextValue` 或 `NextSample` 方法。`NextValue` 返回一个简单的 `float` 值；`NextSample` 返回一个 `CounterSample` 对象，该对象公开了一组更高级的属性，如 `CounterFrequency`、`TimeStamp`、`BaseValue` 和 `RawValue`。

`PerformanceCounter` 的构造函数接受类别名称、计数器名称和可选的实例。因此，要显示所有 CPU 的当前处理器利用率，您可以执行以下操作：

```cs
using PerformanceCounter pc = new PerformanceCounter ("Processor",
                                                      "% Processor Time",
                                                      "_Total");
Console.WriteLine (pc.NextValue());
```

或者显示当前进程的“真实”（即私有）内存消耗：

```cs
string procName = Process.GetCurrentProcess().ProcessName;
using PerformanceCounter pc = new PerformanceCounter ("Process",
                                                      "Private Bytes",
                                                      procName);
Console.WriteLine (pc.NextValue());
```

`PerformanceCounter` 没有公开 `ValueChanged` 事件，因此如果要监视更改，必须进行轮询。在下一个示例中，我们每 200 毫秒轮询一次，直到通过 `EventWaitHandle` 发出退出信号：

```cs
// need to import System.Threading as well as System.Diagnostics

static void Monitor (string category, string counter, string instance,
                     EventWaitHandle stopper)
{
  if (!PerformanceCounterCategory.Exists (category))
    throw new InvalidOperationException ("Category does not exist");

  if (!PerformanceCounterCategory.CounterExists (counter, category))
    throw new InvalidOperationException ("Counter does not exist");

  if (instance == null) instance = "";   // "" == no instance (not null!)
  if (instance != "" &&
      !PerformanceCounterCategory.InstanceExists (instance, category))
    throw new InvalidOperationException ("Instance does not exist");

  float lastValue = 0f;
  using (PerformanceCounter pc = new PerformanceCounter (category,
                                                      counter, instance))
    while (!stopper.WaitOne (200, false))
    {
      float value = pc.NextValue();
      if (value != lastValue)         // Only write out the value
      {                               // if it has changed.
        Console.WriteLine (value);
        lastValue = value;
      }
    }
}
```

使用这种方法同时监视处理器和硬盘活动：

```cs
EventWaitHandle stopper = new ManualResetEvent (false);

new Thread (() =>
  Monitor ("Processor", "% Processor Time", "_Total", stopper)
).Start();

new Thread (() =>
  Monitor ("LogicalDisk", "% Idle Time", "C:", stopper)
).Start();

Console.WriteLine ("Monitoring - press any key to quit");
Console.ReadKey();
stopper.Set();
```

## 创建计数器和写入性能数据

在写入性能计数器数据之前，需要创建性能类别和计数器。您必须在一步中创建性能类别以及属于它的所有计数器，如下所示：

```cs
string category = "Nutshell Monitoring";

// We'll create two counters in this category:
string eatenPerMin = "Macadamias eaten so far";
string tooHard = "Macadamias deemed too hard";

if (!PerformanceCounterCategory.Exists (category))
{
  CounterCreationDataCollection cd = new CounterCreationDataCollection();

  cd.Add (new CounterCreationData (eatenPerMin,
          "Number of macadamias consumed, including shelling time",
          PerformanceCounterType.NumberOfItems32));

  cd.Add (new CounterCreationData (tooHard,
          "Number of macadamias that will not crack, despite much effort",
          PerformanceCounterType.NumberOfItems32));

  PerformanceCounterCategory.Create (category, "Test Category",
    PerformanceCounterCategoryType.SingleInstance, cd);
}
```

当您选择添加计数器时，新的计数器将显示在 Windows 性能监视工具中。如果稍后要在同一类别中定义更多计数器，则必须首先调用 `Performance​Coun⁠terCategory.Delete` 删除旧类别。

###### 注意

创建和删除性能计数器需要管理员权限。因此，通常作为应用程序设置的一部分来完成。

创建计数器后，可以通过实例化 `PerformanceCounter`，将 `ReadOnly` 设置为 `false`，并设置 `RawValue` 来更新其值。您还可以使用 `Increment` 和 `IncrementBy` 方法来更新现有值：

```cs
string category = "Nutshell Monitoring";
string eatenPerMin = "Macadamias eaten so far";

using (PerformanceCounter pc = new PerformanceCounter (category,
                                                       eatenPerMin, ""))
{
  pc.ReadOnly = false;
  pc.RawValue = 1000;
  pc.Increment();
  pc.IncrementBy (10);
  Console.WriteLine (pc.NextValue());    // 1011
}
```

# Stopwatch 类

`Stopwatch` 类提供了一个方便的机制来测量执行时间。`Stopwatch` 使用操作系统和硬件提供的最高分辨率机制，通常小于一微秒。（相比之下，`DateTime.Now` 和 `Environment.TickCount` 的分辨率约为 15 毫秒。）

要使用 `Stopwatch`，调用 `StartNew` —— 这将实例化一个 `Stopwatch` 并启动它开始计时。（或者，您可以手动实例化然后调用 `Start`。）`Elapsed` 属性以 `TimeSpan` 形式返回经过的时间间隔：

```cs
Stopwatch s = Stopwatch.StartNew();
System.IO.File.WriteAllText ("test.txt", new string ('*', 30000000));
Console.WriteLine (s.Elapsed);       // 00:00:01.4322661
```

`Stopwatch` 还公开了`ElapsedTicks`属性，它以`long`返回经过的“ticks”数。要从 ticks 转换为秒，请除以 `StopWatch​.Fre⁠quency`。还有一个`ElapsedMilliseconds`属性，通常是最方便的。

调用`Stop`会冻结`Elapsed`和`ElapsedTicks`。不会有由“运行中”的`Stopwatch`引起的后台活动，因此调用`Stop`是可选的。

# 跨平台诊断工具

在本节中，我们简要介绍了.NET 可用的跨平台诊断工具：

dotnet-counters

提供运行应用程序状态的概述

dotnet-trace

获取更详细的性能和事件监视信息

dotnet-dump

要在需求或崩溃后获取内存转储

这些工具不需要管理员权限，并且适用于开发和生产环境。

## dotnet-counters

*dotnet-counters*工具监视.NET 进程的内存和 CPU 使用情况，并将数据写入控制台（或文件）。

要安装工具，请从命令提示符或带有*dotnet*路径的终端运行以下命令：

```cs
dotnet tool install --global dotnet-counters
```

您可以按以下方式开始监视进程：

```cs
dotnet-counters monitor System.Runtime --process-id *<<ProcessID>>*
```

`System.Runtime` 表示我们要监视*System.Runtime*类别下的所有计数器。您可以指定类别或计数器名称（`dotnet-counters list`命令列出所有可用的类别和计数器）。

输出将持续刷新并类似于以下内容：

```cs
Press p to pause, r to resume, q to quit.
    Status: Running

[System.Runtime]
    # of Assemblies Loaded                            63
    % Time in GC (since last GC)                       0
    Allocation Rate (Bytes / sec)                244,864
    CPU Usage (%)                                      6
    Exceptions / sec                                   0
    GC Heap Size (MB)                                  8
    Gen 0 GC / sec                                     0
    Gen 0 Size (B)                               265,176
    Gen 1 GC / sec                                     0
    Gen 1 Size (B)                               451,552
    Gen 2 GC / sec                                     0
    Gen 2 Size (B)                                    24
    LOH Size (B)                               3,200,296
    Monitor Lock Contention Count / sec                0
    Number of Active Timers                            0
    ThreadPool Completed Work Items / sec             15
    ThreadPool Queue Length                            0
    ThreadPool Threads Count                           9
    Working Set (MB)                                  52
```

以下是所有可用的命令：

| 命令 | 目的 |
| --- | --- |
| `list` | 显示计数器名称及其描述的列表 |
| `ps` | 显示符合监视条件的 dotnet 进程列表 |
| `monitor` | 显示所选计数器的值（定期刷新） |
| `collect` | 将计数器信息保存到文件 |

支持以下参数：

| 选项/参数 | 目的 |
| --- | --- |
| `--version` | 显示*dotnet-counters*的版本。 |
| `-h, --help` | 显示程序的帮助信息。 |
| `-p, --process-id` | 要监视的 dotnet 进程的 ID。适用于`monitor`和`collect`命令。 |
| `--refresh-interval` | 设置所需的刷新间隔（以秒为单位）。适用于`monitor`和`collect`命令。 |
| `-o, --output` | 设置输出文件名。适用于`collect`命令。 |
| `--format` | 设置输出格式。有效的格式为*csv*或*json*。适用于`collect`命令。 |

## dotnet-trace

跟踪是程序中事件的时间戳记录，例如调用方法或查询数据库。跟踪还可以包括性能指标和自定义事件，并可以包含局部上下文，例如局部变量的值。传统上，.NET Framework 和诸如 ASP.NET 之类的框架使用 ETW。在.NET 5 中，应用程序跟踪在 Windows 上运行时写入 ETW，在 Linux 上运行时写入 LTTng。

要安装工具，请执行以下命令：

```cs
dotnet tool install --global dotnet-trace
```

要开始记录程序的事件，请运行以下命令：

```cs
dotnet-trace collect --process-id *<<ProcessId>>*
```

这会使用默认配置文件运行*dotnet-trace*，该配置文件收集 CPU 和 .NET 运行时事件，并写入名为*trace.nettrace*的文件。您可以使用`--profile`开关指定其他配置文件：*gc-verbose* 跟踪垃圾回收和抽样对象分配，*gc-collect* 以低开销跟踪垃圾回收。`-o`开关允许您指定不同的输出文件名。

默认输出为*.netperf*文件，可以直接在 Windows 机器上使用 PerfView 工具分析。或者，您可以指示*dotnet-trace*创建与 Speedscope 兼容的文件，Speedscope 是一个免费的在线分析服务，位于[*https://speedscope.app*](https://speedscope.app)。要创建 Speedscope（*.speedscope.json*）文件，请使用选项`--format speedscope`。

###### 注：

您可以从[*https://github.com/microsoft/perfview*](https://github.com/microsoft/perfview)下载 PerfView 的最新版本。随 Windows 10 发货的版本可能不支持*.netperf*文件。

支持以下命令：

| Commands | Purpose |
| --- | --- |
| `collect` | 开始将计数器信息记录到文件中。 |
| `ps` | 显示可供监视的 dotnet 进程列表。 |
| `list-profiles` | 列出具有每个提供程序和过滤器描述的预构建跟踪配置文件。 |
| `convert <file>` | 将*nettrace*（*.netperf*）格式转换为另一种格式。当前唯一的目标选项是*speedscope*。 |

### 自定义跟踪事件

您的应用程序可以通过定义自定义`EventSource`来发出自定义事件：

```cs
[EventSource (Name = "MyTestSource")]
public sealed class MyEventSource : EventSource
{
  public static MyEventSource Instance = new MyEventSource ();

  MyEventSource() : base (EventSourceSettings.EtwSelfDescribingEventFormat)
  {
  }

  public void Log (string message, int someNumber)
  {
    WriteEvent (1, message, someNumber);
  }
}
```

`WriteEvent` 方法被重载以接受各种简单类型的组合（主要是字符串和整数）。然后，您可以按以下方式调用它：

```cs
MyEventSource.Instance.Log ("Something", 123);
```

调用*dotnet-trace*时，必须指定要记录的任何自定义事件源的名称：

```cs
dotnet-trace collect --process-id *<<ProcessId>>* --providers MyTestSource
```

## dotnet-dump

*Dump*，有时称为*核心转储*，是进程虚拟内存状态的快照。您可以按需转储正在运行的进程，或者配置操作系统在应用程序崩溃时生成转储。

在 Ubuntu Linux 上，以下命令在应用程序崩溃时启用核心转储（不同 Linux 版本间的必要步骤可能有所不同）：

```cs
ulimit -c unlimited
```

在 Windows 上，使用*regedit.exe*在本地计算机 hive 中创建或编辑以下键：

```cs
SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps
```

在此之下，添加与您的可执行文件相同的键（例如*foo.exe*），并在该键下添加以下键：

+   `DumpFolder`（REG_EXPAND_SZ），其值指示要写入转储文件的路径

+   `DumpType`（REG_DWORD），其值为 2，以请求完整转储

+   （可选）`DumpCount`（REG_DWORD），指示在删除最老的转储文件之前的最大转储文件数

要安装该工具，请运行以下命令：

```cs
dotnet tool install --global dotnet-dump
```

安装完后，您可以按需进行转储（而无需结束进程），如下所示：

```cs
dotnet-dump collect --process-id *<<YourProcessId>>*
```

以下命令启动用于分析转储文件的交互式 shell：

```cs
dotnet-dump analyze *<<dumpfile>>*
```

如果异常导致应用程序崩溃，你可以使用*printexceptions*命令（简写为*pe*）来显示异常的详细信息。dotnet-dump shell 支持多个额外的命令，你可以使用*help*命令列出这些命令。
