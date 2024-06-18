# 第二十四章 本地和 COM 互操作性

本章描述如何与本地（非托管）动态链接库（DLL）和组件对象模型（COM）组件集成。除非另有说明，本章提到的类型存在于`System`或`System.Runtime.InteropServices`命名空间中。

# 调用本地 DLL

*P/Invoke*，简称*平台调用服务*，允许您访问非托管 DLL 中的函数、结构和回调（Unix 上的共享库）。

例如，考虑在 Windows DLL *user32.dll* 中定义的`MessageBox`函数如下：

```cs
int MessageBox (HWND hWnd, LPCTSTR lpText, LPCTSTR lpCaption, UINT uType);
```

您可以通过声明同名的静态方法、应用`extern`关键字并添加`DllImport`属性来直接调用此函数：

```cs
using System;
using System.Runtime.InteropServices;

MessageBox (IntPtr.Zero,
            "Please do not press this again.", "Attention", 0);

[DllImport("user32.dll")]
static extern int MessageBox (IntPtr hWnd, string text, string caption,
                              int type);
```

`System.Windows`和`System.Windows.Forms`命名空间中的`MessageBox`类本身调用类似的非托管方法。

这是 Ubuntu Linux 的一个`DllImport`示例：

```cs
Console.WriteLine ($"User ID: {getuid()}");

[DllImport("libc")]
static extern uint getuid();
```

CLR 包含一个编组器，知道如何在.NET 类型和非托管类型之间转换参数和返回值。在 Windows 示例中，`int`参数直接转换为函数期望的四字节整数，字符串参数转换为以 UTF-16 编码的空终止 Unicode 字符数组。`IntPtr`是一个设计用来封装非托管句柄的结构体；在 32 位平台上宽度为 32 位，在 64 位平台上宽度为 64 位。在 Unix 上也会进行类似的转换。（从 C# 9 开始，您还可以使用`nint`类型，它映射到`IntPtr`。）

# 类型和参数编组

## 编组常见类型

在非托管端，可以有多种方式来表示给定的数据类型。例如，字符串可以包含单字节的 ANSI 字符或 UTF-16 Unicode 字符，并且可以是长度前缀、空终止或固定长度。使用`MarshalAs`属性，您可以指定 CLR 编组器正在使用的变体，以便它提供正确的转换。以下是一个示例：

```cs
[DllImport("...")]
static extern int Foo ( [MarshalAs (UnmanagedType.LPStr)] string s );
```

`UnmanagedType`枚举包括编组器了解的所有 Win32 和 COM 类型。在本例中，编组器被告知转换为`LPStr`，这是一个以 null 结尾的单字节 ANSI 字符串。

在.NET 端，您也可以选择使用哪种数据类型。例如，非托管句柄可以映射到`IntPtr`、`int`、`uint`、`long`或`ulong`。

###### 注意

大多数非托管句柄封装了一个地址或指针，因此必须映射到`IntPtr`以兼容 32 位和 64 位操作系统。典型示例是 HWND。

在 Win32 和 POSIX 函数中，经常会遇到接受一组常量的整数参数，这些常量在 C++头文件（如*WinUser.h*）中定义。您可以选择将其定义为简单的 C#常量，也可以将其定义为枚举。使用枚举可以使代码更整洁，同时增加静态类型安全性。我们在“共享内存”中提供了一个示例。

###### 注意

在安装 Microsoft Visual Studio 时，请确保安装 C++头文件——即使在 C++类别中未选择其他任何选项。这是定义所有本机 Win32 常量的地方。然后，可以通过在 Visual Studio 程序目录中搜索**.h*来定位所有头文件。

在 Unix 上，POSIX 标准定义了常量的名称，但符合 POSIX 的 Unix 系统的各个实现可能为这些常量分配不同的数值。你必须使用所选操作系统的正确数值。类似地，POSIX 定义了在互操作调用中使用的结构体的标准。结构体中字段的顺序不由标准固定，Unix 实现可能会添加额外的字段。定义函数和类型的 C++头文件通常安装在*/usr/include*或*/usr/local/include*中。

从非托管代码接收字符串返回到.NET 需要进行一些内存管理。如果你使用`StringBuilder`而不是`string`声明外部方法，封送程序会自动执行这项工作，如下所示：

```cs
StringBuilder s = new StringBuilder (256);
GetWindowsDirectory (s, 256);
Console.WriteLine (s);

[DllImport("kernel32.dll")]
static extern int GetWindowsDirectory (StringBuilder sb, int maxChars);
```

在 Unix 上，工作方式类似。以下调用`getcwd`以返回当前目录：

```cs
var sb = new StringBuilder (256);
Console.WriteLine (getcwd (sb, sb.Capacity));

[DllImport("libc")]
static extern string getcwd (StringBuilder buf, int size);
```

虽然`StringBuilder`使用方便，但 CLR 在执行时需要进行额外的内存分配和复制，效率略低。在性能热点处，可以通过使用`char[]`来避免这种开销：

```cs
[DllImport ("kernel32.dll", CharSet = CharSet.Unicode)]
static extern int GetWindowsDirectory (char[] buffer, int maxChars);
```

注意，你必须在`DllImport`属性中指定`CharSet`。调用函数后还必须将输出字符串修剪到指定长度。你可以通过使用数组池（参见“数组池”）来实现这一点，同时最小化内存分配，如下所示：

```cs
string GetWindowsDirectory()
{
  var array = ArrayPool<char>.Shared.Rent (256);
  try
  {
    int length = GetWindowsDirectory (array, 256);
    return new string (array, 0, length).ToString();
  }
  finally { ArrayPool<char>.Shared.Return (array); }
}
```

（当然，这个例子有些刻意，因为你可以通过内置的`Environment.GetFolderPath`方法获取 Windows 目录。）

###### 注意

如果你不确定如何调用特定的 Win32 或 Unix 方法，通常可以在互联网上搜索方法名称和*DllImport*来找到示例。对于 Windows，网站[*http://www.pinvoke.net*](http://www.pinvoke.net)是一个旨在记录所有 Win32 签名的维基。

## 管理类和结构体

有时，你需要将结构体传递给非托管方法。例如，Win32 API 中的`GetSystemTime`定义如下：

```cs
void GetSystemTime (LPSYSTEMTIME lpSystemTime);
```

`LPSYSTEMTIME`符合此 C 结构：

```cs
typedef struct _SYSTEMTIME {
  WORD wYear;
  WORD wMonth;
  WORD wDayOfWeek;
  WORD wDay;
  WORD wHour;
  WORD wMinute;
  WORD wSecond;
  WORD wMilliseconds;
} SYSTEMTIME, *PSYSTEMTIME;
```

要调用`GetSystemTime`，我们必须定义一个与此 C 结构体匹配的.NET 类或结构体：

```cs
using System;
using System.Runtime.InteropServices;

[StructLayout(LayoutKind.Sequential)]
class SystemTime
{
   public ushort Year;
   public ushort Month;
   public ushort DayOfWeek;
   public ushort Day;
   public ushort Hour;
   public ushort Minute;
   public ushort Second;
   public ushort Milliseconds;
}
```

`StructLayout`属性指示封送程序如何将每个字段映射到其非托管对应项。`LayoutKind.Sequential`表示我们希望字段按*pack-size*边界依次对齐（你很快就会明白这是什么意思），就像它们在 C 结构体中一样。这里字段名不重要；字段顺序才是重要的。

现在我们可以调用`GetSystemTime`：

```cs
SystemTime t = new SystemTime();
GetSystemTime (t);
Console.WriteLine (t.Year);

[DllImport("kernel32.dll")]
static extern void GetSystemTime (SystemTime t);
```

同样，在 Unix 上：

```cs
Console.WriteLine (GetSystemTime());

static DateTime GetSystemTime()
{
  DateTime startOfUnixTime = 
    new DateTime(1970, 1, 1, 0, 0, 0, 0, System.DateTimeKind.Utc);

  Timespec tp = new Timespec();
  int success = clock_gettime (0, ref tp);
  if (success != 0) throw new Exception ("Error checking the time.");
  return startOfUnixTime.AddSeconds (tp.tv_sec).ToLocalTime();  
}

[DllImport("libc")]
static extern int clock_gettime (int clk_id, ref Timespec tp);

[StructLayout(LayoutKind.Sequential)]
struct Timespec
{
  public long tv_sec;   /* seconds */
  public long tv_nsec;  /* nanoseconds */
}
```

在 C 和 C# 中，对象的字段位于该对象地址加上 *n* 字节的位置。不同之处在于，在 C# 程序中，CLR 通过查找字段标记来找到偏移量；而 C 中的字段名直接编译为偏移量。例如，在 C 中，`wDay` 只是一个标记，用于表示 `SystemTime` 实例地址加上 24 字节处的内容。

为了访问速度，每个字段都被放置在其大小的倍数的偏移量处。然而，该乘数受到 *x* 字节的限制，其中 *x* 是*包大小*。在当前实现中，默认的包大小是 8 字节，因此由一个 `sbyte` 和一个（8 字节）`long` 组成的结构体占据 16 字节，并且 `sbyte` 后面的 7 字节被浪费了。您可以通过 `StructLayout` 属性的 `Pack` 属性指定一个*包大小*来减少或消除这种浪费：这使得字段对齐到指定包大小的倍数的偏移量上。因此，使用包大小为 1，刚刚描述的结构体将仅占用 9 字节。可以指定包大小为 1、2、4、8 或 16 字节。

`StructLayout` 属性还允许您指定显式字段偏移量（参见“模拟 C 联合”）。

## 进出传递

在前面的示例中，我们将 `SystemTime` 实现为一个类。我们也可以选择使用结构体——前提是 `GetSystemTime` 声明为具有 `ref` 或 `out` 参数：

```cs
[DllImport("kernel32.dll")]
static extern void GetSystemTime (out SystemTime t);
```

在大多数情况下，C# 的参数传递语义与外部方法相同。按值传递的参数被复制进去，C# 的 ref 参数是传入/传出的，C# 的 out 参数则是传出的。然而，对于具有特殊转换的类型，存在一些例外情况。例如，数组类和 `StringBuilder` 类在从函数中输出时需要复制，因此它们是传入/传出的。偶尔会有覆盖此行为的情况，可以使用 `In` 和 `Out` 属性。例如，如果数组应该是只读的，`in` 修饰符表示仅在进入函数时复制数组，而不是在离开函数时复制：

```cs
static extern void Foo ( [In] int[] array);
```

## 调用约定

非托管方法通过堆栈和（可选地）CPU 寄存器接收参数和返回值。由于有多种实现方式，因此出现了多种不同的协议。这些协议称为*调用约定*。

当前 CLR 支持三种调用约定：StdCall、Cdecl 和 ThisCall。

默认情况下，CLR 使用*平台默认*调用约定（该平台的标准约定）。在 Windows 上是 StdCall，在 Linux x86 上是 Cdecl。

如果非托管方法不遵循此默认设置，可以明确声明其调用约定如下：

```cs
[DllImport ("MyLib.dll", CallingConvention=CallingConvention.Cdecl)]
static extern void SomeFunc (...)
```

有些颇具误导性的命名为 `CallingConvention.WinApi` 实际上指的是平台默认。

# 从非托管代码的回调

C# 还允许外部函数通过回调调用 C# 代码。有两种方法可以实现回调：

+   通过函数指针

+   通过委托

为了说明问题，我们将调用以下位于 *User32.dll* 中的 Windows 函数，该函数枚举所有顶级窗口句柄：

```cs
BOOL EnumWindows (WNDENUMPROC lpEnumFunc, LPARAM lParam);
```

`WNDENUMPROC` 是一个回调函数，按顺序触发每个窗口的句柄（或直到回调返回 `false`）。以下是其定义：

```cs
BOOL CALLBACK EnumWindowsProc (HWND hwnd, LPARAM lParam);
```

## 使用函数指针的回调

从 C# 9 开始，当您的回调是静态方法时，最简单且性能最佳的选项是使用 *函数指针*。对于 `WNDENUMPROC` 回调，我们可以使用以下函数指针：

```cs
delegate*<IntPtr, IntPtr, bool>
```

这表示一个接受两个 `IntPtr` 参数并返回 `bool` 的函数。然后，您可以使用 `&` 运算符将静态方法传递给它：

```cs
using System;
using System.Runtime.InteropServices;

unsafe
{
  EnumWindows (&PrintWindow, IntPtr.Zero);

  [DllImport ("user32.dll")]
  static extern int EnumWindows (
    delegate*<IntPtr, IntPtr, bool> hWnd, IntPtr lParam);

  static bool PrintWindow (IntPtr hWnd, IntPtr lParam)
  {
    Console.WriteLine (hWnd.ToInt64());
    return true;
  }
}
```

对于函数指针，回调必须是静态方法（或者是本示例中的静态局部函数）。

### UnmanagedCallersOnly

您可以通过在函数指针声明中应用 `unmanaged` 关键字，以及在回调方法上应用 `[UnmanagedCallersOnly]` 属性来提高性能：

```cs
using System;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

unsafe
{
  EnumWindows (&PrintWindow, IntPtr.Zero);

  [DllImport ("user32.dll")]
  static extern int EnumWindows (
    delegate* unmanaged <IntPtr, IntPtr, byte> hWnd, IntPtr lParam);

  [UnmanagedCallersOnly]
  static byte PrintWindow (IntPtr hWnd, IntPtr lParam)
  {
    Console.WriteLine (hWnd.ToInt64());
    return 1;
  }
}
```

此属性标记 `PrintWindow` 方法，以便它只能从未管理的代码中调用，从而允许运行时采取捷径。请注意，我们还将方法的返回类型从 `bool` 更改为 `byte`：这是因为您应用 `[UnmanagedCallersOnly]` 的方法只能在签名中使用 *可平铺* 的值类型。可平铺类型是那些在托管和未托管世界中都以相同方式表示的类型，因此不需要任何特殊的编组逻辑。这些包括原始整数类型、`float`、`double`，以及仅包含可平铺类型的结构体。如果位于具有指定 `CharSet.Unicode` 的 `StructLayout` 属性的结构体中，则 `char` 类型也是可平铺的：

```cs
[StructLayout (LayoutKind.Sequential, CharSet=CharSet.Unicode)]
```

### 非默认的调用约定

默认情况下，编译器假定未管理的回调遵循平台默认的调用约定。如果情况不是这样，您可以通过 `[UnmanagedCallersOnly]` 属性的 `CallConvs` 参数显式指定其调用约定：

```cs
[UnmanagedCallersOnly (CallConvs = new[] { typeof (CallConvStdcall) })]
static byte PrintWindow (IntPtr hWnd, IntPtr lParam) ...
```

您还必须通过在 `unmanaged` 关键字后插入特殊修饰符来更新函数指针类型：

```cs
delegate* unmanaged[Stdcall] <IntPtr, IntPtr, byte> hWnd, IntPtr lParam);
```

###### 注意

编译器允许您在方括号内放置任何标识符（例如 `XYZ`），只要有一个名为 `CallConv**XYZ**` 的.NET 类型（这是由运行时理解的，并且与您在应用 `[UnmanagedCallersOnly]` 属性时指定的匹配）。这样做可以使微软在未来更容易添加新的调用约定。

在这种情况下，我们指定了 StdCall，这是 Windows 平台的默认值（Cdecl 是 Linux x86 的默认值）。以下是当前支持的所有选项：

| 名称 | 未管理的修饰符 | 支持的类型 |
| --- | --- | --- |
| Stdcall | `unmanaged[Stdcall]` | `CallConvStdcall` |
| Cdecl | `unmanaged[Cdecl]` | `CallConvCdecl` |
| ThisCall | `unmanaged[Thiscall]` | `CallConvThiscall` |

## 使用委托的回调

未管理的回调也可以通过委托来完成。这种方法适用于所有版本的 C#，并允许引用实例方法的回调。

要继续进行，首先声明一个与回调匹配的委托类型。然后，您可以将委托实例传递给外部方法：

```cs
class CallbackFun
{
 delegate bool EnumWindowsCallback (IntPtr hWnd, IntPtr lParam);

  [DllImport("user32.dll")]
  static extern int EnumWindows (EnumWindowsCallback hWnd, IntPtr lParam);

  static bool PrintWindow (IntPtr hWnd, IntPtr lParam)
  {
    Console.WriteLine (hWnd.ToInt64());
    return true;
  }
  static readonly EnumWindowsCallback printWindowFunc = PrintWindow;

  static void Main() => EnumWindows (printWindowFunc, IntPtr.Zero);
}
```

对于未管理的回调，使用委托具有讽刺意味，因为很容易陷入陷阱，允许在委托实例超出范围后发生回调（在这种情况下，委托变得符合垃圾收集）。这可能导致最糟糕的运行时异常之一——没有有用的堆栈跟踪。对于静态方法回调，您可以通过将委托实例分配给只读静态字段（就像我们在这个例子中所做的那样）来避免这种情况。对于实例方法回调，这种模式将无法帮助，因此您必须小心编码，以确保在任何潜在的回调期间至少保持对委托实例的一个引用。即使如此，如果未管理的一侧存在错误——它在您告诉它不要后继续调用回调——您可能仍然需要处理无法跟踪的异常。一个解决方法是为每个未管理的函数定义一个唯一的委托类型：这有助于诊断，因为异常中报告了委托类型。

您可以通过将 `[UnmanagedFunctionPointer]` 属性应用于委托来更改回调的调用约定，从而使其不同于平台默认值：

```cs
[UnmanagedFunctionPointer (CallingConvention.Cdecl)]
delegate void MyCallback (int foo, short bar);
```

# 模拟 C 联合

`struct` 中的每个字段都有足够的空间来存储其数据。考虑一个包含一个 `int` 和一个 `char` 的 `struct`。`int` 可能从偏移量 `0` 开始，并且保证至少有四个字节。因此，`char` 将至少从偏移量 `4` 开始。如果由于某种原因 `char` 从偏移量 `2` 开始，如果给 `char` 赋值，则会更改 `int` 的值。听起来像混乱，不是吗？奇怪的是，C 语言支持一个称为 *union* 的结构体变体，正是做到了这一点。您可以通过使用 `LayoutKind.Explicit` 和 `FieldOffset` 属性在 C# 中模拟这一点。

想象一个这种方法会有用的情况可能会有些挑战。但是，假设您想在外部合成器上播放一个音符。Windows 多媒体 API 通过 MIDI 协议提供了一个可以做到这一点的函数：

```cs
[DllImport ("winmm.dll")]
public static extern uint midiOutShortMsg (IntPtr handle, uint message);
```

第二个参数 `message` 描述要播放的音符。问题在于构造这个 32 位无符号整数：它在内部被分成字节，表示 MIDI 通道、音符以及打击时的速度。一个解决方案是通过位移和掩码操作符 `<<`、`>>`、`&` 和 `|` 将这些字节转换为和从 32 位“打包”消息。尽管更简单的方法是定义一个具有显式布局的结构体：

```cs
[StructLayout (LayoutKind.Explicit)]
public struct NoteMessage
{
  [FieldOffset(0)] public uint PackedMsg;    // 4 bytes long

  [FieldOffset(0)] public byte Channel;      // FieldOffset also at 0
  [FieldOffset(1)] public byte Note;
  [FieldOffset(2)] public byte Velocity;
}
```

`Channel`、`Note`和`Velocity`字段故意与 32 位打包消息重叠。这允许您使用任一字段进行读取和写入。无需计算即可保持其他字段同步：

```cs
NoteMessage n = new NoteMessage();
Console.WriteLine (n.PackedMsg);    // 0

n.Channel = 10;
n.Note = 100;
n.Velocity = 50;
Console.WriteLine (n.PackedMsg);    // 3302410

n.PackedMsg = 3328010;
Console.WriteLine (n.Note);         // 200
```

# 共享内存

内存映射文件，或*共享内存*，是 Windows 中的一项功能，允许同一台计算机上的多个进程共享数据。共享内存非常快速，并且与管道不同，提供对共享数据的*随机*访问。我们在第十五章中看到，您可以使用`MemoryMappedFile`类访问内存映射文件；绕过这一点，直接调用 Win32 方法是演示 P/Invoke 的好方法。

Win32 的`CreateFileMapping`函数分配共享内存。您告诉它需要多少字节以及用于标识共享的名称。然后，另一个应用程序可以通过使用相同名称调用`OpenFileMapping`来订阅这个内存。这两种方法都返回一个*句柄*，您可以通过调用`MapViewOfFile`将其转换为指针。

这是一个封装对共享内存访问的类：

```cs
using System;
using System.Runtime.InteropServices;
using System.ComponentModel;

public sealed class SharedMem : IDisposable
{
  // Here we're using enums because they're safer than constants

  enum FileProtection : uint      // constants from winnt.h
  {
    ReadOnly = 2,
    ReadWrite = 4
  }

  enum FileRights : uint          // constants from WinBASE.h
  {
    Read = 4,
    Write = 2,
    ReadWrite = Read + Write
  }

  static readonly IntPtr NoFileHandle = new IntPtr (-1);

  [DllImport ("kernel32.dll", SetLastError = true)]
  static extern IntPtr CreateFileMapping (IntPtr hFile,
                                          int lpAttributes,
                                          FileProtection flProtect,
                                          uint dwMaximumSizeHigh,
                                          uint dwMaximumSizeLow,
                                          string lpName);

  [DllImport ("kernel32.dll", SetLastError=true)]
  static extern IntPtr OpenFileMapping (FileRights dwDesiredAccess,
                                        bool bInheritHandle,
                                        string lpName);

  [DllImport ("kernel32.dll", SetLastError = true)]
  static extern IntPtr MapViewOfFile (IntPtr hFileMappingObject,
                                      FileRights dwDesiredAccess,
                                      uint dwFileOffsetHigh,
                                      uint dwFileOffsetLow,
                                      uint dwNumberOfBytesToMap);

  [DllImport ("Kernel32.dll", SetLastError = true)]
  static extern bool UnmapViewOfFile (IntPtr map);

  [DllImport ("kernel32.dll", SetLastError = true)]
  static extern int CloseHandle (IntPtr hObject);

  IntPtr fileHandle, fileMap;

  public IntPtr Root => fileMap;

  public SharedMem (string name, bool existing, uint sizeInBytes)
  {
    if (existing)
      fileHandle = OpenFileMapping (FileRights.ReadWrite, false, name);
    else
      fileHandle = CreateFileMapping (NoFileHandle, 0,
                                      FileProtection.ReadWrite,
                                      0, sizeInBytes, name);
    if (fileHandle == IntPtr.Zero)
      throw new Win32Exception();

    // Obtain a read/write map for the entire file
    fileMap = MapViewOfFile (fileHandle, FileRights.ReadWrite, 0, 0, 0);

    if (fileMap == IntPtr.Zero)
      throw new Win32Exception();
  }

  public void Dispose()
  {
    if (fileMap != IntPtr.Zero) UnmapViewOfFile (fileMap);
    if (fileHandle != IntPtr.Zero) CloseHandle (fileHandle);
    fileMap = fileHandle = IntPtr.Zero;
  }
}
```

在本示例中，我们在使用`SetLastError`协议发出错误代码的`DllImport`方法上设置了`SetLastError=true`。这确保当抛出异常时，`Win32Exception`中填充了错误的详细信息。（它还允许您通过调用`Marshal.GetLastWin32Error`显式查询错误。）

要演示这个类，我们需要运行两个应用程序。第一个应用程序创建共享内存如下：

```cs
using (SharedMem sm = new SharedMem ("MyShare", false, 1000))
{
  IntPtr root = sm.Root;
  // I have shared memory!

  Console.ReadLine();         // Here's where we start a second app...
}
```

第二个应用程序订阅共享内存，通过构造同名的`SharedMem`对象，使用`existing`参数设置为`true`：

```cs
using (SharedMem sm = new SharedMem ("MyShare", true, 1000))
{
  IntPtr root = sm.Root;
  // I have the same shared memory!
  // ...
}
```

结果是，每个程序都有一个`IntPtr`——指向同一非托管内存的指针。现在，这两个应用程序需要以某种方式通过这个共享指针读取和写入内存。一种方法是编写一个类来封装所有共享数据，然后使用`UnmanagedMemoryStream`将数据（反）序列化到非托管内存中。然而，如果数据量很大，这种方法效率低下。想象一下，如果共享内存类有一兆字节的数据，而只需更新一个整数。更好的方法是将共享数据结构定义为结构体，然后直接映射到共享内存中。我们将在接下来的部分讨论这个问题。

# 将结构体映射到非托管内存

您可以直接将带有`Sequential`或`Explicit`的`StructLayout`映射到非托管内存中。考虑以下结构体：

```cs
[StructLayout (LayoutKind.Sequential)]
unsafe struct MySharedData
{
  public int Value;
  public char Letter;
  public fixed float Numbers [50];
}
```

`fixed`指令允许我们在内联中定义固定长度的值类型数组，并将我们带入`unsafe`领域。这个结构体中为 50 个浮点数分配了内联空间。与标准的 C#数组不同，`Numbers`不是一个指向数组的引用——它*就是*数组。如果我们运行以下代码：

```cs
static unsafe void Main() => Console.WriteLine (sizeof (MySharedData));
```

结果是 208：50 个四字节浮点数，加上`Value`整数的四个字节，加上`Letter`字符的两个字节。总计 206，由于`floats`在四字节边界上对齐（四个字节是`float`的大小），所以四舍五入为 208。

我们可以在`unsafe`上下文中展示`MySharedData`，最简单的方法是使用栈分配的内存：

```cs
MySharedData d;
MySharedData* data = &d;       // Get the address of d

data->Value = 123;
data->Letter = 'X';
data->Numbers[10] = 1.45f;

or:

// Allocate the array on the stack:
MySharedData* data = stackalloc MySharedData[1];

data->Value = 123;
data->Letter = 'X';
data->Numbers[10] = 1.45f;
```

当然，我们并没有展示任何在受控上下文中无法实现的事情。然而，假设我们想要将`MySharedData`的一个实例存储在*不受 CLR 垃圾收集器管理的非托管堆*上。这时指针变得非常有用：

```cs
MySharedData* data = (MySharedData*)
  Marshal.AllocHGlobal (sizeof (MySharedData)).ToPointer();

data->Value = 123;
data->Letter = 'X';
data->Numbers[10] = 1.45f;
```

`Marshal.AllocHGlobal`在非托管堆上分配内存。以下是如何稍后释放相同内存的方法：

```cs
Marshal.FreeHGlobal (new IntPtr (data));
```

（忘记释放内存的结果是一个老式的内存泄漏。）

###### 注意

从.NET 6 开始，您可以使用`NativeMemory`类来分配和释放非托管内存。`NativeMemory`使用比`AllocHGlobal`更新（更好）的底层 API，并且还包括执行对齐分配的方法。

顾名思义，在这里我们使用`MySharedData`与我们在前一节中编写的`SharedMem`类结合使用。以下程序分配了一块共享内存块，然后将`MySharedData`结构映射到该内存中：

```cs
static unsafe void Main()
{
  using (SharedMem sm = new SharedMem ("MyShare", false, 
                          (uint) sizeof (MySharedData)))
  {
    void* root = sm.Root.ToPointer();
    MySharedData* data = (MySharedData*) root;

    data->Value = 123;
    data->Letter = 'X';
    data->Numbers[10] = 1.45f;
    Console.WriteLine ("Written to shared memory");

    Console.ReadLine();

    Console.WriteLine ("Value is " + data->Value);
    Console.WriteLine ("Letter is " + data->Letter);
    Console.WriteLine ("11th Number is " + data->Numbers[10]);
    Console.ReadLine();
  }
}
```

###### 注意

您可以使用内置的`MemoryMappedFile`类来替代`SharedMem`，如下所示：

```cs
using (MemoryMappedFile mmFile =
       MemoryMappedFile.CreateNew ("MyShare", 1000))
using (MemoryMappedViewAccessor accessor =
       mmFile.CreateViewAccessor())
{
  byte* pointer = null;
  accessor.SafeMemoryMappedViewHandle.AcquirePointer
   (ref pointer);
  void* root = pointer;
  ...
}
```

这是第二个程序，它附加到相同的共享内存中，读取由第一个程序写入的值（必须在第一个程序在等待`ReadLine`语句时运行，因为共享内存对象在离开其`using`语句时被释放）：

```cs
static unsafe void Main()
{
  using (SharedMem sm = new SharedMem ("MyShare", true, 
                          (uint) sizeof (MySharedData)))  
  {
    void* root = sm.Root.ToPointer();
    MySharedData* data = (MySharedData*) root;

    Console.WriteLine ("Value is " + data->Value);
    Console.WriteLine ("Letter is " + data->Letter);
    Console.WriteLine ("11th Number is " + data->Numbers[10]);

    // Our turn to update values in shared memory!
    data->Value++;
    data->Letter = '!';
    data->Numbers[10] = 987.5f;
    Console.WriteLine ("Updated shared memory");
    Console.ReadLine();
  }
}
```

这些程序的输出如下：

```cs
// First program:

Written to shared memory
Value is 124
Letter is !
11th Number is 987.5

// Second program:

Value is 123
Letter is X
11th Number is 1.45
Updated shared memory
```

不要被指针吓到：C++程序员在整个应用程序中都在使用它们，并且大多数时候都能使其正常工作！这种用法相对简单。

正如发生的那样，我们的示例因为另一个原因而不安全——字面上来说。我们没有考虑到两个程序同时访问同一内存时出现的线程安全（或更准确地说，进程安全）问题。要在生产应用程序中使用这一点，我们需要在`MySharedData`结构的`Value`和`Letter`字段中添加`volatile`关键字，以防止这些字段被即时（JIT）编译器（或硬件 CPU 寄存器）缓存。此外，随着我们与字段的交互超出了琐碎的范围，我们很可能需要通过跨进程的`Mutex`来保护它们的访问，就像我们在多线程程序中使用`lock`语句来保护对字段的访问一样。我们在第二十一章中详细讨论了线程安全性。

## fixed and fixed {...}

直接将结构体映射到内存的一个限制是结构体只能包含未管理的类型。例如，如果你需要共享字符串数据，必须使用固定长度的字符数组。这意味着需要手动转换到和从`string`类型。以下是如何做到的：

```cs
[StructLayout (LayoutKind.Sequential)]
unsafe struct MySharedData
{
  ...
  // Allocate space for 200 chars (i.e., 400 bytes).
  const int MessageSize = 200;
  fixed char message [MessageSize];

  // One would most likely put this code into a helper class:
  public string Message
  {
    get { fixed (char* cp = message) return new string (cp); }
    set
    {
      fixed (char* cp = message)
      {
        int i = 0;
        for (; i < value.Length && i < MessageSize - 1; i++)
          cp [i] = value [i];

        // Add the null terminator
        cp [i] = '\0';
      }
    }
  }
}
```

###### 注意

没有指向固定数组的引用；相反，你得到一个指针。当你索引固定数组时，实际上在进行指针算术运算！

使用`fixed`关键字首次使用时，我们在结构体中为 200 个字符分配了空间。然而，同一关键字在后续属性定义中有不同的含义。它指示 CLR *固定* 一个对象，以便如果它决定在`fixed`块内部执行垃圾收集，它将不会移动内存堆上的底层结构（因为其内容通过直接内存指针进行迭代）。看看我们的程序，你可能会想知道`MySharedData`如何在内存中移动，因为它位于不受管理的世界中，垃圾收集器在那里无权利。然而，编译器并不知道这一点，并且担心我们*可能*在托管上下文中使用`MySharedData`，因此坚持我们添加`fixed`关键字，以使我们的`unsafe`代码在托管上下文中安全。编译器确实有一点道理——只需要将`MySharedData`放到堆上：

```cs
object obj = new MySharedData();
```

这导致`MySharedData`在堆上装箱，并且在垃圾收集期间可以传输。

本例说明了如何在映射到未管理内存的结构体中表示字符串。对于更复杂的类型，你也可以使用现有的序列化代码。唯一的注意是序列化数据的长度绝不要超过结构体分配的空间；否则，结果将是与后续字段意外联合。

# COM 互操作性

.NET 运行时为 COM 提供了特殊支持，使得可以从.NET 使用 COM 对象，反之亦然。COM 仅在 Windows 上可用。

## COM 的目的

COM 是 Component Object Model 的缩写，是一种与库进行接口交互的二进制标准，由微软在 1993 年发布。发明 COM 的动机是使组件能够以语言无关和版本宽容的方式相互通信。在 COM 出现之前，Windows 的方法是发布声明使用 C 编程语言的结构和函数的 DLL。这种方法不仅特定于语言，而且很脆弱。在这样的库中，类型的规范与其实现是不可分割的：即使更新具有新字段的结构也意味着破坏其规范。

COM 的优点在于通过称为*COM 接口*的结构从其底层实现中分离出类型的规范。COM 还允许在有状态*对象*上调用方法，而不仅仅限于简单的过程调用。

###### 注意

在某种程度上，.NET 编程模型是 COM 编程原则的进化：.NET 平台也促进跨语言开发，并允许二进制组件在不破坏依赖于它们的应用程序的情况下演变。

## COM 类型系统的基础知识

COM 类型系统围绕接口展开。COM 接口与 .NET 接口类似，但更常见，因为 COM 类型仅通过接口公开其功能。在 .NET 世界中，例如，我们可以简单地声明一个类型，如下所示：

```cs
public class Foo
{
  public string Test() => "Hello, world";
}
```

类型的消费者可以直接使用 `Foo`。如果以后我们更改了 `Test()` 的*实现*，调用方不需要重新编译。在这方面，.NET 将接口与实现分离——而无需接口。我们甚至可以添加一个重载而不会破坏调用者：

```cs
  public string Test (string s) => $"Hello, world {s}";
```

在 COM 世界中，`Foo` 通过接口公开其功能以实现同样的解耦。因此，在 `Foo` 的类型库中，可能存在这样的接口：

```cs
public interface IFoo { string Test(); }
```

（我们通过展示一个 C# 接口（而不是 COM 接口）来说明这一点。然而，原理是相同的——尽管具体实现方式不同。）

调用方随后将与 `IFoo` 交互，而不是 `Foo`。

当涉及添加 `Test` 的重载版本时，使用 COM 比使用 .NET 更复杂。首先，我们将避免修改 `IFoo` 接口，因为这将破坏与前一个版本的二进制兼容性（COM 的原则之一是一旦发布，接口就是*不可变*的）。其次，COM 不允许方法重载。解决方案是让 `Foo` 实现*第二个接口*：

```cs
public interface IFoo2 { string Test (string s); }
```

（同样，我们将其转译为 .NET 接口以便熟悉。）

支持多个接口对于使 COM 库具有版本化能力至关重要。

### IUnknown 和 IDispatch

所有的 COM 接口都使用全局唯一标识符（GUID）来标识。

COM 中的根接口是 `IUnknown`——所有 COM 对象必须实现它。该接口有三个方法：

+   `AddRef`

+   `Release`

+   `QueryInterface`

`AddRef` 和 `Release` 用于生命周期管理，因为 COM 使用引用计数而不是自动垃圾回收（COM 设计用于与不可管理代码一起工作，其中自动垃圾回收不可行）。`Quer⁠y​Interface` 方法返回一个支持该接口的对象引用，如果可以的话。

为了实现动态编程（例如脚本和自动化），COM 对象还可以实现 `IDispatch`。这使得动态语言可以以后期绑定的方式调用 COM 对象——有点像 C# 中的 `dynamic`（尽管仅限于简单调用）。

# 从 C# 调用 COM 组件

CLR 对 COM 的内置支持意味着你不直接使用 `IUnknown` 和 `IDispatch`。相反，你使用 CLR 对象，并且运行时通过 Runtime-Callable Wrappers (RCWs) 将你的调用封送到 COM 世界。运行时还通过调用 `AddRef` 和 `Release`（在 .NET 对象被终结时）来处理生命周期管理，并且处理两个世界之间的原始类型转换。类型转换确保每一方以熟悉的形式看到整数和字符串类型等。

此外，需要一种以静态类型方式访问 RCW 的方法。这是*COM 互操作类型*的任务。 COM 互操作类型是自动生成的代理类型，每个 COM 成员都暴露一个 .NET 成员。类型库导入工具（*tlbimp.exe*）基于你选择的 COM 库从命令行生成 COM 互操作类型，并将它们编译成*COM 互操作程序集*。

###### 注意

如果一个 COM 组件实现多个接口，则 *tlbimp.exe* 工具会生成一个包含所有接口成员并集的单一类型。

你可以在 Visual Studio 中通过转到“添加引用”对话框框，并从 COM 选项卡中选择一个库来创建 COM 互操作程序集。例如，如果安装了 Microsoft Excel，则添加对 Microsoft Excel 对象库的引用允许你与 Excel 的 COM 类互操作。以下是创建和显示工作簿，然后在该工作簿中填充单元格的 C# 代码：

```cs
using System;
using Excel = Microsoft.Office.Interop.Excel;

var excel = new Excel.Application();
excel.Visible = true;
excel.WindowState = Excel.XlWindowState.xlMaximized;
Excel.Workbook workBook = excel.Workbooks.Add();
((Excel.Range)excel.Cells[1, 1]).Font.FontStyle = "Bold";
((Excel.Range)excel.Cells[1, 1]).Value2 = "Hello World";
workBook.SaveAs (@"d:\temp.xlsx");
```

###### 注意

当前需要在应用程序中嵌入互操作类型（否则，运行时无法在运行时找到它们）。可以在 Visual Studio 的解决方案资源管理器中单击 COM 引用，然后在属性窗口中将 Embed Interop Types 属性设置为 true，或者打开 *.csproj* 文件并添加以下行（**加粗**）：

```cs
<ItemGroup>
  <COMReference Include="Microsoft.Office.Excel.dll">
    ...
    <EmbedInteropTypes>true</EmbedInteropTypes>
  </COMReference>
</ItemGroup>
```

`Excel.Application` 类是一个 COM 互操作类型，其运行时类型是一个 RCW。当我们访问 `Workbooks` 和 `Cells` 属性时，会得到更多的互操作类型。

## 可选参数和命名参数

因为 COM API 不支持函数重载，所以经常有函数具有许多参数，其中许多是可选的。例如，这是如何调用 Excel 工作簿的 `Save` 方法的方式：

```cs
var missing = System.Reflection.Missing.Value;

workBook.SaveAs (@"d:\temp.xlsx", missing, missing, missing, missing,
  missing, Excel.XlSaveAsAccessMode.xlNoChange, missing, missing,
  missing, missing, missing);
```

好消息是，C# 对可选参数的支持是 COM 感知的，所以我们可以这样做：

```cs
workBook.SaveAs (@"d:\temp.xlsx");
```

（正如我们在 第三章 中所述，可选参数由编译器“展开”为完整的冗长形式。）

命名参数允许你指定额外的参数，而不管它们的位置如何：

```cs
workBook.SaveAs (@"d:\test.xlsx", Password:"foo");
```

## 隐式 ref 参数

一些 COM API（特别是 Microsoft Word）公开的函数将*每一个*参数声明为按引用传递，无论函数是否修改参数值。这是因为认为不复制参数值会带来性能提升（*实际*性能提升微乎其微）。

从历史上看，从 C# 调用这样的方法一直很笨拙，因为您必须对每个参数指定 `ref` 关键字，这会阻止使用可选参数。例如，要打开 Word 文档，我们过去必须这样做：

```cs
object filename = "foo.doc";
object notUsed1 = Missing.Value;
object notUsed2 = Missing.Value;
object notUsed3 = Missing.Value;
...
Open (ref filename, ref notUsed1, ref notUsed2, ref notUsed3, ...);
```

由于隐式引用参数，您可以省略 COM 函数调用中的 `ref` 修饰符，从而允许使用可选参数：

```cs
word.Open ("foo.doc");
```

缺点是，如果调用的 COM 方法实际上会改变参数值，您既不会得到编译时错误，也不会得到运行时错误。

## 索引器

省略 `ref` 修饰符的能力还有另一个好处：它使带有 `ref` 参数的 COM 索引器可通过普通的 C# 索引器语法访问。否则，这将被禁止，因为 `ref`/`out` 参数在 C# 索引器中不受支持。

您还可以调用接受参数的 COM 属性。在以下示例中，`Foo` 是一个接受整数参数的属性：

```cs
myComObject.Foo [123] = "Hello";
```

在 C# 中自己编写这样的属性仍然是被禁止的：类型只能在自身上（“默认”索引器）公开索引器。因此，如果您想在 C# 中编写使前述语句合法的代码，`Foo` 需要返回另一种公开了（默认）索引器的类型。

## 动态绑定

动态绑定在调用 COM 组件时有两种帮助方式。

第一种方式是允许访问不使用 COM 互操作类型的 COM 组件。为此，请调用 `Type.GetTypeFromProgID` 以获取 COM 实例的 COM 组件名称，然后使用动态绑定从此调用成员。当然，这没有 IntelliSense，并且无法进行编译时检查：

```cs
Type excelAppType = Type.GetTypeFromProgID ("Excel.Application", true);
dynamic excel = Activator.CreateInstance (excelAppType);
excel.Visible = true;
dynamic wb = excel.Workbooks.Add();
excel.Cells [1, 1].Value2 = "foo";
```

（可以用反射而不是动态绑定实现相同的功能，但更加笨拙。）

###### 注意

此主题的一个变种是调用仅支持 `IDispatch` 的 COM 组件。但是，这样的组件非常罕见。

动态绑定在处理 COM `variant` 类型时也可能有用（程度较低）。由于设计不良而非必要原因，COM API 函数经常会用到这种类型，它在 .NET 中大致相当于 `object`。如果在项目中启用了“嵌入互操作类型”（稍后详述），运行时会将 `variant` 映射为 `dynamic`，而不是映射为 `object`，从而避免了需要进行强制类型转换。例如，您可以合法地执行以下操作：

```cs
excel.Cells [1, 1].Font.FontStyle = "Bold";
```

而不是：

```cs
var range = (Excel.Range) excel.Cells [1, 1];
range.Font.FontStyle = "Bold";
```

以这种方式工作的缺点是，您会失去自动完成功能，因此您必须知道名为 `Font` 的属性存在。因此，通常更容易*动态地*将结果分配给其已知的互操作类型：

```cs
Excel.Range range = excel.Cells [1, 1];
range.Font.FontStyle = "Bold";
```

正如您所见，与老式方法相比，这仅节省了五个字符！

将 `variant` 映射为 `dynamic` 是默认设置，并且是启用引用时的一个功能。

# 嵌入互操作类型

我们之前说过，C# 通常通过调用 *tlbimp.exe* 工具（直接或通过 Visual Studio）生成的互操作类型来调用 COM 组件。

历史上，你唯一的选择是像对待任何其他程序集一样*引用*互操作程序集。这可能会麻烦，因为互操作程序集可以因复杂的 COM 组件而变得相当庞大。例如，微软 Word 的一个小插件需要一个比其自身大几个数量级的互操作程序集。

而不是*引用*互操作程序集，你可以选择嵌入你使用的部分。编译器会分析程序集，精确确定应用程序所需的类型和成员，并直接在应用程序中嵌入这些类型和成员的定义。这样既避免了臃肿，又避免了需要额外传送文件。

要启用此功能，可以在 Visual Studio 的解决方案资源管理器中选择 COM 引用，然后在属性窗口中将“嵌入互操作类型”设置为 true，或者像我们之前描述的那样编辑*.csproj*文件（参见“从 C#调用 COM 组件”）。

## 类型等价性

CLR 支持链接互操作类型的*类型等价性*。这意味着如果两个程序集分别链接到一个互操作类型，那么即使这些链接到的互操作程序集是独立生成的，这些类型也会被视为等效，只要它们包装了相同的 COM 类型。

###### 注意

类型等价性依赖于`System.Runtime.InteropServices`命名空间中的`TypeIdentifierAttribute`特性。当链接到互操作程序集时，编译器会自动应用此特性。如果 COM 类型具有相同的 GUID，则这些类型被认为是等效的。

# 将 C#对象公开给 COM

也可以在 C#中编写可以在 COM 世界中消耗的类。CLR 通过称为*COM-Callable Wrapper*（CCW）的代理实现了这一点。CCW 在两个世界之间进行类型的封送（与 RCW 类似），并根据 COM 协议实现了`IUnknown`（和可选的`IDispatch`）。CCW 通过引用计数从 COM 侧进行生命周期控制（而不是通过 CLR 的垃圾收集器）。

你可以将任何公共类公开给 COM（作为“进程内”服务器）。要实现此功能，首先创建一个接口，分配一个唯一的 GUID（在 Visual Studio 中，你可以使用工具 > 创建 GUID），声明其对 COM 可见，然后设置接口类型：

```cs
namespace MyCom
{
  [ComVisible(true)]
  [Guid ("226E5561-C68E-4B2B-BD28-25103ABCA3B1")]  // Change this GUID
  [InterfaceType (ComInterfaceType.InterfaceIsIUnknown)]
  public interface IServer
  {
    int Fibonacci();
  }
}
```

接下来，提供一个接口的实现，并为该实现分配一个唯一的 GUID：

```cs
namespace MyCom
{
  [ComVisible(true)]
  [Guid ("09E01FCD-9970-4DB3-B537-0EC555967DD9")]  // Change this GUID
  public class Server
  {
    public ulong Fibonacci (ulong whichTerm)
    {
      if (whichTerm < 1) throw new ArgumentException ("...");
      ulong a = 0;
      ulong b = 1;
      for (ulong i = 0; i < whichTerm; i++)
      {
        ulong tmp = a;
        a = b;
        b = tmp + b;
      }
      return a;
    }
  }
}
```

编辑你的.*csproj*文件，添加以下行：

```cs
<PropertyGroup>
 <EnableComHosting>true</EnableComHosting>
</PropertyGroup>
```

现在，当你构建你的项目时，会生成一个额外的文件，*MyCom​.com⁠host.dll*，可以注册为 COM 互操作。（请记住，该文件始终是 32 位或 64 位，取决于你的项目配置：在这种情况下不存在“任何 CPU”选项。）从*提升的*命令提示符中，切换到保存 DLL 的目录，并运行*regsvr32 MyCom.comhost.dll*。

然后，您可以从大多数支持 COM 的语言中消费您的 COM 组件。例如，您可以在文本编辑器中创建此 Visual Basic 脚本，并通过在 Windows 资源管理器中双击该文件或从命令提示符中启动它来运行它，就像运行程序一样：

```cs
REM Save file as ComClient.vbs
Dim obj
Set obj = CreateObject("MyCom.Server")

result = obj.Fibonacci(12)
Wscript.Echo result
```

注意，.NET Framework 不能加载到与 .NET 5+ 或 .NET Core 相同的进程中。因此，.NET 5+ COM 服务器无法加载到 .NET Framework COM 客户端进程中，反之亦然。

## 启用无注册表 COM

传统上，COM 将类型信息添加到注册表中。无注册表 COM 使用一个清单文件而不是注册表来控制对象的激活。要启用此功能，请将以下行（加粗）添加到您的 *.csproj* 文件中：

```cs
<PropertyGroup>
  <TargetFramework>netcoreapp3.0</TargetFramework>
  <EnableComHosting>true</EnableComHosting>
 <EnableRegFreeCom>true</EnableRegFreeCom>
</PropertyGroup>
```

然后，您的构建将生成 *MyCom.X.manifest*。

###### 注意

在 .NET 5+ 中不支持生成 COM 类型库 (*.tlb)。您可以手动编写一个 IDL（接口定义语言）文件或者 C++ 头文件来定义接口中的本地声明。
