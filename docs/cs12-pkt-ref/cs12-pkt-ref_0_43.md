# 第四十三章：预处理器指令

预处理器指令为编译器提供关于代码区域的附加信息。最常见的预处理器指令是条件指令，提供了一种在编译时包含或排除代码区域的方式。例如：

```cs
#define DEBUG
class MyClass
{
  int x;
  void Foo()
  {
 #if DEBUG
    Console.WriteLine ("Testing: x = {0}", x);
 #endif
 }
  ...
}
```

在此类中，`Foo` 中的语句在编译时条件依赖于 `DEBUG` 符号的存在。如果我们移除 `DEBUG` 符号，则不会编译该语句。预处理符号可以在源文件内（正如我们所做的）或在项目文件中的 `<DefineConstants>` 元素内定义。

使用 `#if` 和 `#elif` 指令，您可以在多个符号上执行 *或*、*与* 和 *非* 操作使用 `||`、`&&` 和 `!` 操作符。以下指令指示编译器在定义 `TESTMODE` 符号且未定义 `DEBUG` 符号时包括随后的代码：

```cs
#if TESTMODE && !DEBUG
  ...
```

请记住，您并非在构建普通的 C# 表达式，并且您操作的符号与 *变量* —— 无论是静态还是其他类型的 —— 没有任何连接。

`#error` 和 `#warning` 符号通过使编译器生成警告或错误来防止条件指令的意外误用，给定一个不良的编译符号集。

表 14 描述了预处理器指令的完整列表。

表 14\. 预处理器指令

| 预处理器指令 | 动作 |
| --- | --- |
| `` #define `*symbol*` `` | 定义 `*symbol*`。 |
| `` #undef `*symbol*` `` | 取消定义 `*symbol*`。 |
| `#if` `*symbol*` `[*operator symbol2*]...` | 条件编译 (`*operator*` 包括 `==`、`!=`、`&&` 和 ` | | `)。 |
| `#else` | 执行到后续 `#endif` 的代码。 |
| `#elif` `*symbol*` `[*operator symbol2*]` | 结合 `#else` 分支和 `#if` 测试。 |
| `#endif` | 结束条件指令。 |
| `` #warning `*text*` `` | `*text*` 的警告将出现在编译器输出中。 |
| `` #error `*text*` `` | `*text*` 的错误将出现在编译器输出中。 |
| ``#line [`*number*` ["`*file*`"] &#124; hidden]`` | `*number*` 指定源代码中的行；`*file*` 是要出现在计算机输出中的文件名；`hidden` 指示调试器跳过此点到下一个 `#line` 指令。 |
| `` #region `*name*` `` | 标记大纲的开始。 |
| `#endregion` | 结束大纲区域。 |
| `#pragma warning` | 参见下一节。 |
| `` #nullable `*option*` `` | 查看“可空引用类型”。 |

## Pragma Warning

当编译器发现您的代码中出现看似不经意的东西时，它会生成警告。与错误不同，警告通常不会阻止应用程序的编译。

编译器警告在发现错误时非常有价值。然而，当出现*虚假*警告时，它们的有效性会受到损害。在大型应用程序中，保持良好的信噪比对于注意到“真正”的警告至关重要。

因此，编译器允许您使用`#pragma warning`指令有选择地禁止警告。在此示例中，我们指示编译器不要警告我们未使用的字段`Message`：

```cs
public class Foo
{
 #pragma warning disable 414
  static string Message = "Hello";
 #pragma warning restore 414
}
```

在`#pragma warning`指令中省略编号会禁用或恢复所有警告代码。

如果您在应用此指令时彻底，可以使用`/warnaserror`开关进行编译——这指示编译器将任何剩余的警告视为错误。

