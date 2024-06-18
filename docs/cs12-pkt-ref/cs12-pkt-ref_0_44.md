# 第四十四章：XML 文档

*文档注释*是嵌入的 XML 片段，用于文档化类型或成员。文档注释紧接在类型或成员声明之前，并以三斜杠开始：

```cs
/// <summary>Cancels a running query.</summary>
public void Cancel() { ... }
```

可以这样编写多行注释：

```cs
/// <summary>
/// Cancels a running query
/// </summary>
public void Cancel() { ... }
```

或者像这样（注意开头的额外星号）：

```cs
/** 
    <summary> Cancels a running query. </summary>
*/
public void Cancel() { ... }
```

如果使用`/doc`指令进行编译（或在项目文件中启用 XML 文档），编译器会提取并整理文档注释到一个单独的 XML 文件中。这有两个主要用途：

+   如果放置在与编译后程序集相同的文件夹中，Visual Studio 会自动读取 XML 文件，并使用信息为同名程序集的消费者提供 IntelliSense 成员列表。

+   第三方工具（如 Sandcastle 和 NDoc）可以将 XML 文件转换为 HTML 帮助文件。

## 标准 XML 文档标签

这里是 Visual Studio 和文档生成器识别的标准 XML 标签：

`<summary>`

```cs
<summary>*...*</summary>
```

指示 IntelliSense 应显示的工具提示，用于类型或成员。通常是一个短语或句子。

`<remarks>`

```cs
<remarks>*...*</remarks>
```

额外描述类型或成员的文本。文档生成器将其捡起并合并到类型或成员描述的主体中。

`<param>`

```cs
<param name="*name*">*...*</param>
```

解释方法的参数。

`<returns>`

```cs
<returns>*...*</returns>
```

解释方法的返回值。

`<exception>`

```cs
<exception [cref="*type*"]>*...*</exception>
```

列出方法可能抛出的异常（`cref`指向异常类型）。

`<permission>`

```cs
<permission [cref="*type*"]>*...*</permission>
```

指示文档化类型或成员所需的`IPermission`类型。

`<example>`

```cs
<example>*...*</example>
```

表示一个示例（由文档生成器使用）。通常包含描述文本和源代码（源代码通常在 `<c>` 或 `<code>` 标签内）。

`<c>`

```cs
<c>*...*</c>
```

指示内联代码片段。此标签通常用于`<example>`块内。

`<code>`

```cs
<code>*...*</code>
```

指示多行代码示例。此标签通常用于`<example>`块内。

`<see>`

```cs
<see cref="*member*">*...*</see>
```

插入对另一个类型或成员的内联交叉引用。HTML 文档生成器通常会将其转换为超链接。如果类型或成员名称无效，编译器会发出警告。

`<seealso>`

```cs
<seealso cref="*member*">*...*</seealso>
```

交叉引用另一个类型或成员。文档生成器通常会将此写入到单独的“另请参阅”部分在页面底部。

`<paramref>`

```cs
<paramref name="*name*"/>
```

引用 `<summary>` 或 `<remarks>` 标记中的参数。

`<list>`

```cs
<list type=[ bullet | number | table ]>
  <listheader>
    <term>*...*</term>
    <description>*...*</description>
  </listheader>
  <item>
    <term>*...*</term>
    <description>*...*</description>
  </item>
</list>
```

指导文档生成器生成项目符号、编号或表格样式的列表。

`<para>`

```cs
<para>*...*</para>
```

指导文档生成器将内容格式化为单独的段落。

`<include>`

```cs
<include file='*filename*' path='*tagpath*[@name="*id*"]'>
  ...
</include>
```

合并包含文档的外部 XML 文件。路径属性表示该文件中特定元素的 XPath 查询。
