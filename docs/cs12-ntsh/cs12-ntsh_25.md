# 第二十五章：正则表达式

正则表达式语言识别字符模式。支持正则表达式的.NET 类型基于 Perl 5 正则表达式，支持搜索和搜索/替换功能。

正则表达式用于诸如以下任务：

+   验证文本输入，如密码和电话号码。

+   将文本数据解析为更结构化的形式（例如，NuGet 版本字符串）

+   替换文档中的文本模式（例如，仅整个单词）

本章分为概念部分和参考部分，分别介绍.NET 中正则表达式的基础知识和正则表达式语言。

所有正则表达式类型都定义在`System.Text.RegularExpressions`中。

###### 注意

本章中的示例都预装在 LINQPad 中，该工具还包括一个交互式的 RegEx 工具（按 Ctrl+Shift+F1）。在线工具可访问[*http://regexstorm.net/tester*](http://regexstorm.net/tester)。

# 正则表达式基础知识

最常见的正则表达式运算符之一是*量词*。`?`是一个量词，匹配前面的项目 0 或 1 次。换句话说，`?`表示*可选*。项目可以是单个字符或方括号中复杂的字符结构。例如，正则表达式`"colou?r"`匹配`color`和`colour`，但不匹配`colouur`：

```cs
Console.WriteLine (Regex.Match ("color",   @"colou?r").Success);  // True
Console.WriteLine (Regex.Match ("colour",  @"colou?r").Success);  // True
Console.WriteLine (Regex.Match ("colouur", @"colou?r").Success);  // False
```

`Regex.Match`在较大字符串内搜索。返回的对象具有匹配的`Index`和`Length`属性以及实际匹配的`Value`：

```cs
Match m = Regex.Match ("any colour you like", @"colou?r");

Console.WriteLine (m.Success);     // True
Console.WriteLine (m.Index);       // 4
Console.WriteLine (m.Length);      // 6
Console.WriteLine (m.Value);       // colour
Console.WriteLine (m.ToString());  // colour
```

您可以将`Regex.Match`视为`string`的`IndexOf`方法的更强大版本。不同之处在于它搜索*模式*而不是字面字符串。

`IsMatch`方法是调用`Match`并测试`Success`属性的快捷方式。

默认情况下，正则表达式引擎从左到右工作，因此仅返回最左边的匹配项。您可以使用`NextMatch`方法返回更多匹配项：

```cs
Match m1 = Regex.Match ("One color? There are two colours in my head!",
                        @"colou?rs?");
Match m2 = m1.NextMatch();
Console.WriteLine (m1);         // color
Console.WriteLine (m2);         // colours
```

`Matches`方法返回数组中的所有匹配项。我们可以按照前面的示例重写如下：

```cs
foreach (Match m in Regex.Matches
          ("One color? There are two colours in my head!", @"colou?rs?"))
  Console.WriteLine (m);
```

另一个常见的正则表达式运算符是*交替符*，用竖线`|`表示。交替符表示备选项。以下匹配“Jen”，“Jenny”和“Jennifer”：

```cs
Console.WriteLine (Regex.IsMatch ("Jenny", "Jen(ny|nifer)?"));  // True
```

交替符周围的括号将备选项与表达式的其余部分分开。

###### 注意

当匹配正则表达式时，你可以指定超时时间。如果匹配操作超过指定的`TimeSpan`，将抛出`RegexMatchTimeoutException`异常。如果你的程序处理用户提供的正则表达式，这非常有用，因为它可以防止格式错误的正则表达式无限循环。

## 编译正则表达式

在前面的一些示例中，我们反复调用了静态的`Regex`方法，使用相同的模式。在这些情况下的另一种方法是使用模式和`RegexOptions.Compiled`实例化一个`Regex`对象，然后调用实例方法：

```cs
Regex r = new Regex (@"sausages?", RegexOptions.Compiled);
Console.WriteLine (r.Match ("sausage"));   // sausage
Console.WriteLine (r.Match ("sausages"));  // sausages
```

`RegexOptions.Compiled` 指示 `RegEx` 实例使用轻量级代码生成（`Reflection.Emit` 中的 `DynamicMethod`）动态构建和编译特定于该正则表达式的代码。这样做可以实现更快的匹配速度，但需要付出初始编译成本。

您还可以实例化 `Regex` 对象而不使用 `RegexOptions.Compiled`。`Regex` 实例是不可变的。

###### 注意

正则表达式引擎非常快速。即使不进行编译，简单匹配通常少于一微秒。

## RegexOptions

`RegexOptions` 标志枚举允许您调整匹配行为。`RegexOptions` 的常见用法是执行不区分大小写的搜索：

```cs
Console.WriteLine (Regex.Match ("a", "A", RegexOptions.IgnoreCase)); // a
```

这适用于当前文化规则的大小写等价性。`CultureInvariant` 标志允许您请求不变的文化：

```cs
Console.WriteLine (Regex.Match ("a", "A", RegexOptions.IgnoreCase
                                        | RegexOptions.CultureInvariant));
```

您可以在正则表达式中通过单字母代码激活大多数 `RegexOptions` 标志，如下所示：

```cs
Console.WriteLine (Regex.Match ("a", @"(?i)A"));                     // a
```

您可以在表达式中随时打开和关闭选项：

```cs
Console.WriteLine (Regex.Match ("AAAa", @"(?i)a(?-i)a"));            // Aa
```

另一个有用的选项是 `IgnorePatternWhitespace` 或 `(?x)`。这允许您插入空格以使正则表达式更易读，而不会将空格视为字面字符。

`NonBacktracking` 选项（从 .NET 7 开始）指示正则表达式引擎使用前向匹配算法。这通常导致性能较慢，并禁用某些高级特性，如前瞻或后顾。但是，它还可以防止形式不正确或恶意构造的表达式花费近乎无限的时间，从而减轻处理用户提供的正则表达式时可能发生的拒绝服务攻击（*ReDOS* 攻击）。在这种情况下，指定超时也很有用。

表格 25-1 列出所有 `RegExOptions` 值及其单字母代码。

表格 25-1\. 正则表达式选项

| 枚举值 | 正则表达式代码 | 描述 |
| --- | --- | --- |
| `None` |   |   |
| `IgnoreCase` | `i` | 忽略大小写（默认情况下，正则表达式区分大小写） |
| `Multiline` | `m` | 更改 `^` 和 `$`，使其匹配行的开头/结尾而不是字符串的开头/结尾 |
| `ExplicitCapture` | `n` | 仅捕获显式命名或显式编号的组（见 “Groups”） |
| `Compiled` |   | 强制编译为 IL（见 “Compiled Regular Expressions”） |
| `Singleline` | `s` | 使 `.` 匹配每个字符（而不是匹配除 `\n` 外的每个字符） |
| `IgnorePatternWhitespace` | `x` | 从模式中消除未转义的空格 |
| `RightToLeft` | `r` | 从右向左搜索；不能在中间指定 |
| `ECMAScript` |   | 强制执行 ECMAScript 兼容性（默认情况下，实现不符合 ECMAScript） |
| `CultureInvariant` |   | 关闭字符串比较的特定于文化的行为 |
| `NonBacktracking` |   | 禁用回溯以确保可预测（尽管较慢）的性能 |

## 字符转义

正则表达式具有以下元字符，它们具有特殊的而不是字面意义：

> \ * + ? | { [ () ^ $ . # 

要字面使用元字符，必须使用反斜杠进行前缀，即*转义*字符。在下面的例子中，我们转义 `?` 字符来匹配字符串 `"what?"`：

```cs
Console.WriteLine (Regex.Match ("what?", @"what\?")); // what? (correct)
Console.WriteLine (Regex.Match ("what?", @"what?"));  // what  (incorrect)
```

###### 注意

如果字符位于*集合*（方括号）内，则此规则不适用，并且元字符被逐字解释。我们在下一节讨论集合。

`Regex` 的 `Escape` 和 `Unescape` 方法通过将包含正则表达式元字符的字符串替换为转义等效项（反之亦然）来转换字符串：

```cs
Console.WriteLine (Regex.Escape   (@"?"));     // \?
Console.WriteLine (Regex.Unescape (@"\?"));    // ?>
```

本章中所有的正则表达式字符串都用 C# 的 `@` 文字表达。这是为了绕过 C# 的转义机制，该机制也使用反斜杠。没有 `@`，字面上的反斜杠将需要四个反斜杠：

```cs
Console.WriteLine (Regex.Match ("\\", "\\\\"));    // \
```

除非包括 `(?x)` 选项，否则空格在正则表达式中被视为字面量：

```cs
Console.Write (Regex.IsMatch ("hello world", @"hello world"));  // True
```

## 字符集

字符集充当特定字符集的通配符。

| 表达式 | 含义 | 反义（“非”） |
| --- | --- | --- |
| `[abcdef]` | 匹配列表中的单个字符。 | `[^abcdef]` |
| `[a-f]` | 匹配范围内的单个字符。 | `[^a-f]` |
| `\d` | 匹配 Unicode *数字* 类别中的任何内容。在 ECMAScript 模式下，`[0-9]`。 | `\D` |
| `\w` | 匹配一个*单词*字符（默认情况下，根据 `CultureInfo.CurrentCulture` 变化；例如，在英语中，与 `[a-zA-Z_0-9]` 相同）。 | `\W` |
| `\s` | 匹配空白字符；即，`char.IsWhiteSpace` 返回 true 的任何内容（包括 Unicode 空格）。在 ECMAScript 模式下，`[\n\r\t\f\v ]`。 | `\S` |
| `\p{`*category*`}` | 匹配指定*类别*中的字符。 | `\P` |
| `.` | （默认模式）匹配除 `\n` 之外的任何字符。 | `\n` |
| `.` | （`SingleLine` 模式）匹配任何字符。 | `\n` |

要匹配一组字符中的一个，请将字符集放在方括号内：

```cs
Console.Write (Regex.Matches ("That is that.", "[Tt]hat").Count);   // 2
```

要匹配除了集合中字符以外的任何字符，请在方括号中使用 `^` 符号放置集合：

```cs
Console.Write (Regex.Match ("quiz qwerty", "q[^aeiou]").Index);    // 5
```

你可以使用连字符指定一系列字符。以下正则表达式匹配象棋走法：

```cs
Console.Write (Regex.Match ("b1-c4", @"[a-h]\d-[a-h]\d").Success);  // True
```

`\d` 表示数字字符，因此 `\d` 将匹配任何数字。 `\D` 匹配任何非数字字符。

`\w` 表示一个单词字符，包括字母、数字和下划线。`\W` 匹配任何非单词字符。对于非英语字母，如西里尔字母，这些也能正常工作。

`.` 匹配除了 `\n` 之外的任何字符（但允许 `\r`）。

`\p` 匹配指定类别中的字符，例如 `{Lu}` 表示大写字母或 `{P}` 表示标点符号（我们稍后在参考部分列出类别）：

```cs
Console.Write (Regex.IsMatch ("Yes, please", @"\p{P}"));   // True
```

当我们将它们与*量词*结合使用时，我们将发现更多关于 `\d`、`\w` 和 `.` 的用法。

# 量词

量词匹配指定次数的项。

| 量词 | 含义 |
| --- | --- |
| `*` | 零个或多个匹配 |
| `+` | 一次或多次匹配 |
| `?` | 零或一个匹配 |
| `{*n*}` | 正好 `*n*` 次匹配 |
| `{*n*,}` | 至少 `*n*` 次匹配 |
| `{*n*,*m*}` | 匹配 `*n*` 到 `*m*` 次 |

`*` 量词匹配前面的字符或组零次或更多次。下面的示例匹配 *cv.docx*，以及同一文件的任何编号版本（例如 *cv2.docx*，*cv15.docx*）：

```cs
Console.Write (Regex.Match ("cv15.docx", @"cv\d*\.docx").Success);  // True
```

注意，我们必须使用反斜杠转义文件扩展名中的句点。

下面允许在 *cv* 和 *.docx* 之间的任何内容，并且等效于 `dir cv*.docx`：

```cs
Console.Write (Regex.Match ("cvjoint.docx", @"cv.*\.docx").Success); // True
```

`+` 量词匹配前面的字符或组一次或更多次。例如：

```cs
Console.Write (Regex.Matches ("slow! yeah slooow!", "slo+w").Count);  // 2
```

`{}` 量词匹配指定数量（或范围）的重复。以下匹配一个血压读数：

```cs
Regex bp = new Regex (@"\d{2,3}/\d{2,3}");
Console.WriteLine (bp.Match ("It used to be 160/110"));  // 160/110
Console.WriteLine (bp.Match ("Now it's only 115/75"));   // 115/75
```

## 贪婪量词与懒惰量词的比较

默认情况下，量词是 *贪婪* 的，与 *懒惰* 相对。贪婪量词重复尽可能多次，然后再继续。懒惰量词重复尽可能少次，然后再继续。通过在其后缀加上 `?` 符号，你可以使任何量词变成懒惰的。为了说明差异，请考虑以下 HTML 片段：

```cs
string html = "<i>By default</i> quantifiers are <i>greedy</i> creatures";
```

假设我们要提取两个斜体短语。如果我们执行以下操作：

```cs
foreach (Match m in Regex.Matches (html, @"<i>.*</i>"))
  Console.WriteLine (m);
```

结果不是两个匹配，而是*单个*匹配：

```cs
<i>By default</i> quantifiers are <i>greedy</i>
```

问题在于我们的 `*` 量词贪婪地重复尽可能多次，直到匹配 `</i>`。因此，它会跳过第一个 `</i>`，仅在最后一个 `</i>`（*表达式其余部分仍可匹配的最后点*）处停止。

如果我们使量词懒惰，`*` 将在*表达式其余部分仍可匹配的第一个*点处退出：

```cs
foreach (Match m in Regex.Matches (html, @"<i>.*?</i>"))
  Console.WriteLine (m);
```

这是结果：

```cs
<i>By default</i>
<i>greedy</i>
```

# 零宽断言

正则表达式语言允许您在匹配之前或之后对应该发生的条件进行设置，通过*回顾*、*预查*、*锚点*和*词边界*。这些称为*零宽*断言，因为它们不增加匹配本身的宽度（或长度）。

## 预查和回顾

`(?=*expr*)` 构造检查紧随的文本是否与 `*expr*` 匹配，但不包括 `expr` 在结果中。这被称为 *正向预查*。在下面的例子中，我们寻找一个数字后面跟着单词“miles”：

```cs
Console.WriteLine (Regex.Match ("say 25 miles more", @"\d+\s(?=miles)"));

*OUTPUT: 25*
```

注意，单词“miles”未在结果中返回，尽管它是必须*满足*的匹配。

在成功的 *预查* 之后，匹配继续进行，就好像预览从未发生过一样。因此，如果我们像这样追加 `.*` 到我们的表达式中：

```cs
Console.WriteLine (Regex.Match ("say 25 miles more", @"\d+\s(?=miles).*"));
```

结果是 `25 miles more`。

预查可以在强密码规则强制执行中非常有用。假设密码必须至少包含六个字符并至少包含一个数字。通过预查，我们可以实现如下：

```cs
string password = "...";
bool ok = Regex.IsMatch (password, @"(?=.*\d).{6,}");
```

首先执行*预查*以确保数字在字符串的某处出现。如果满足条件，则返回预览开始之前的位置，并匹配六个或更多字符。（在“正则表达式手册”中，我们包括一个更实质的密码验证示例。）

相反的是*负向预查*结构`(?!*expr*)`。这要求匹配*不*后跟`*expr*`。以下表达式匹配“good”—除非后续字符串中出现“however”或“but”：

```cs
string regex = "(?i)good(?!.*(however|but))";
Console.WriteLine (Regex.IsMatch ("Good work! But...",  regex));  // False
Console.WriteLine (Regex.IsMatch ("Good work! Thanks!", regex));  // True
```

`(?<=*expr*)`结构表示*正向回顾*，要求匹配*之前*由指定表达式。相反的结构`(?<!*expr*)`表示*负向回顾*，要求匹配*不*之前有指定的表达式。例如，以下匹配“good”—除非前面字符串中出现“however”：

```cs
string regex = "(?i)(?<!however.*)good";
Console.WriteLine (Regex.IsMatch ("However good, we...", regex)); // False
Console.WriteLine (Regex.IsMatch ("Very good, thanks!", regex));  // True
```

我们可以通过添加*单词边界断言*来改进这些示例，稍后我们将介绍这一点。

## 锚点

锚点`^`和`$`匹配特定的*位置*。默认情况下：

`^`

匹配字符串的*开始*

`$`

匹配字符串的*结尾*

###### 注意

`^`有两个上下文相关的含义：*锚点*和*字符类否定器*。

`$`有两个上下文相关的含义：*锚点*和*替换组指示符*。

例如：

```cs
Console.WriteLine (Regex.Match ("Not now", "^[Nn]o"));   // No
Console.WriteLine (Regex.Match ("f = 0.2F", "[Ff]$"));   // F
```

当您指定`RegexOptions.Multiline`或在表达式中包含`(?m)`时：

+   `^`匹配字符串或*行*的开始（直接在`\n`之后）。

+   `$`匹配字符串或*行*的结尾（直接在`\n`之前）。

在多行模式下使用`$`有一个注意事项：Windows 中的换行符几乎总是用`\r\n`表示，而不仅仅是`\n`。这意味着对于 Windows 文件，要使`$`有用，通常必须同时匹配`\r`，使用*正向预查*：

```cs
(?=\r?$)
```

*正向预查*确保`\r`不成为结果的一部分。以下匹配以`".txt"`结尾的行：

```cs
string fileNames = "a.txt" + "\r\n" + "b.docx" + "\r\n" + "c.txt";
string r = @".+\.txt(?=\r?$)";
foreach (Match m in Regex.Matches (fileNames, r, RegexOptions.Multiline))
  Console.Write (m + " ");

*OUTPUT: a.txt c.txt*
```

匹配字符串`s`中所有空行：

```cs
MatchCollection emptyLines = Regex.Matches (s, "^(?=\r?$)",
                                            RegexOptions.Multiline);
```

以下匹配所有空行或仅包含空白的行：

```cs
MatchCollection blankLines = Regex.Matches (s, "^[ \t]*(?=\r?$)",
                                            RegexOptions.Multiline);
```

###### 注意

因为锚点匹配的是位置而不是字符，所以在其自身上指定锚点会匹配一个空字符串：

```cs
Console.WriteLine (Regex.Match ("x", "$").Length);   // 0
```

## 单词边界

单词边界断言`\b`匹配单词字符(`\w`)与以下之一相邻：

+   非单词字符（`\W`）

+   字符串的开始/结束（`^`和`$`）

`\b`通常用于匹配整个单词：

```cs
foreach (Match m in Regex.Matches ("Wedding in Sarajevo", @"\b\w+\b"))
  Console.WriteLine (m);

*Wedding*
*in*
*Sarajevo*
```

以下语句突出显示单词边界的效果：

```cs
int one = Regex.Matches ("Wedding in Sarajevo", @"\bin\b").Count; // 1
int two = Regex.Matches ("Wedding in Sarajevo", @"in").Count;     // 2
```

下一个查询使用*正向预查*来返回后面跟有“(sic)”的单词：

```cs
string text = "Don't loose (sic) your cool";
Console.Write (Regex.Match (text, @"\b\w+\b\s(?=\(sic\))"));  // loose
```

# 组

有时将正则表达式分解为一系列子表达式或*组*是有用的。例如，考虑以下表示美国电话号码（如 206-465-1918）的正则表达式：

```cs
\d{3}-\d{3}-\d{4}
```

假设我们想将其分成两组：区号和本地号码。我们可以通过使用括号来*捕获*每个组来实现此目的：

```cs
(\d{3})-(\d{3}-\d{4})
```

程序化地检索组：

```cs
Match m = Regex.Match ("206-465-1918", @"(\d{3})-(\d{3}-\d{4})");

Console.WriteLine (m.Groups[1]);   // 206
Console.WriteLine (m.Groups[2]);   // 465-1918
```

第零组表示整个匹配。换句话说，它的值与匹配的`Value`相同：

```cs
Console.WriteLine (m.Groups[0]);   // 206-465-1918
Console.WriteLine (m);             // 206-465-1918
```

组是正则表达式语言的一部分。这意味着您可以在正则表达式内部引用一个组。`\n`语法允许您在表达式内通过组号`n`索引组。例如，表达式`(\w)ee\1`匹配`deed`和`peep`。在以下示例中，我们找到字符串中所有以相同字母开头和结尾的单词：

```cs
foreach (Match m in Regex.Matches ("pop pope peep", @"\b(\w)\w+\1\b"))
  Console.Write (m + " ");  // pop peep
```

`\w`周围的括号指示正则表达式引擎将子匹配存储在一个组中（在本例中是单个字母），以便稍后使用。我们稍后使用`\1`引用该组，表示表达式中的第一个组。

## 命名组

在长或复杂的表达式中，通过*名称*而不是索引来处理组可能更容易。以下是先前示例的重写，使用了我们命名为`'letter'`的组：

```cs
string regEx =
  @"\b"             +  // word boundary
  @"(?'letter'\w)"  +  // match first letter, and name it 'letter'
  @"\w+"            +  // match middle letters
  @"\k'letter'"     +  // match last letter, denoted by 'letter'
  @"\b";               // word boundary

foreach (Match m in Regex.Matches ("bob pope peep", regEx))
  Console.Write (m + " ");  // bob peep
```

这是如何命名捕获组的：

```cs
(?'*group-name*'group-expr)  *or*  (?<*group-name*>group-expr)
```

下面是如何引用一个组：

```cs
\k'*group-name*'  *or*  \k<*group-name*>
```

以下示例通过查找具有匹配名称的起始和结束节点来匹配简单（非嵌套）XML/HTML 元素：

```cs
string regFind =
  @"<(?'tag'\w+?).*>" +  // lazy-match first tag, and name it 'tag'
  @"(?'text'.*?)"     +  // lazy-match text content, name it 'text'
  @"</\k'tag'>";         // match last tag, denoted by 'tag'

Match m = Regex.Match ("<h1>hello</h1>", regFind);
Console.WriteLine (m.Groups ["tag"]);          // h1
Console.WriteLine (m.Groups ["text"]);         // hello
```

允许 XML 结构的所有可能变化，如嵌套元素，这更加复杂。.NET 正则表达式引擎具有称为“匹配平衡构造”的复杂扩展，可用于嵌套标签 - 有关此信息，请查看互联网和 Jeffrey E. F. Friedl 的[*Mastering Regular Expressions*](https://learning.oreilly.com/library/view/mastering-regular-expressions/0596528124/)（O’Reilly）。

# 替换和拆分文本

`RegEx.Replace`方法的工作方式类似于`string.Replace`，但它使用正则表达式。

以下代码将“cat”替换为“dog”。与`string.Replace`不同，"catapult"不会变成"dogapult"，因为我们匹配了单词边界：

```cs
string find = @"\bcat\b";
string replace = "dog";
Console.WriteLine (Regex.Replace ("catapult the cat", find, replace));

*OUTPUT: catapult the dog*
```

替换字符串可以使用`$0`替换构造引用原始匹配项。以下示例在字符串中的数字周围加上尖括号：

```cs
string text = "10 plus 20 makes 30";
Console.WriteLine (Regex.Replace (text, @"\d+", @"<$0>"));

*OUTPUT: <10> plus <20> makes <30>*
```

您可以使用`$1`，`$2`，`$3`等访问任何捕获的组，或`${*name*}`用于命名组。为了说明这可以有多有用，考虑前一节中匹配简单 XML 元素的正则表达式。通过重新排列这些组，我们可以形成一个替换表达式，将元素的内容移到 XML 属性中：

```cs
string regFind =
  @"<(?'tag'\w+?).*>" +  // lazy-match first tag, and name it 'tag'
  @"(?'text'.*?)"     +  // lazy-match text content, name it 'text'
  @"</\k'tag'>";         // match last tag, denoted by 'tag'

string regReplace =
  @"<${tag}"         +  // <tag
  @"value="""        +  // value="
  @"${text}"         +  // text
  @"""/>";              // "/>

Console.Write (Regex.Replace ("<msg>hello</msg>", regFind, regReplace));
```

这是结果：

```cs
<msg value="hello"/>
```

## MatchEvaluator 委托

`Replace`有一个重载，接受一个`MatchEvaluator`委托，每次匹配时调用。当正则表达式语言表达能力不足时，这允许您将替换字符串的内容委托给 C#代码：

```cs
Console.WriteLine (Regex.Replace ("5 is less than 10", @"\d+",
                   m => (int.Parse (m.Value) * 10).ToString()) );

*OUTPUT: 50 is less than 100*
```

在“Cookbook Regular Expressions”中，我们展示如何使用`Match​Eva⁠luator`适当地转义 Unicode 字符以用于 HTML。

## 拆分文本

静态 `Regex.Split` 方法是 `string.Split` 方法的更强大版本，其中正则表达式表示分隔符模式。在此示例中，我们将字符串分割，其中任何数字都作为分隔符：

```cs
foreach (string s in Regex.Split ("a5b7c", @"\d"))
  Console.Write (s + " ");     // a b c
```

此结果不包括分隔符本身。但是，可以通过使用 *正向预查* 将表达式包裹起来来包含分隔符。以下内容将驼峰格式的字符串拆分为单独的单词：

```cs
foreach (string s in Regex.Split ("oneTwoThree", @"(?=[A-Z])"))
  Console.Write (s + " ");    // one Two Three
```

# 食谱正则表达式

## 配方

### 匹配美国社会安全号码/电话号码

```cs
string ssNum = @"\d{3}-\d{2}-\d{4}";

Console.WriteLine (Regex.IsMatch ("123-45-6789", ssNum));      // True

string phone = @"(?x)
 ( \d{3}[-\s] | \(\d{3}\)\s? )
 \d{3}[-\s]?
 \d{4}";

Console.WriteLine (Regex.IsMatch ("123-456-7890",   phone));   // True
Console.WriteLine (Regex.IsMatch ("(123) 456-7890", phone));   // True
```

### 提取“名称 = 值”对（每行一个）

注意，此处使用了 *多行* 指令 `(?m)`：

```cs
string r = @"(?m)^\s*(?'name'\w+)\s*=\s*(?'value'.*)\s*(?=\r?$)";

string text =
  @"id = 3
    secure = true
    timeout = 30";

foreach (Match m in Regex.Matches (text, r))
  Console.WriteLine (m.Groups["name"] + " is " + m.Groups["value"]);
*id is 3 secure is true timeout is 30*
```

### 强密码验证

下面的内容检查密码是否至少包含六个字符，并且是否包含数字、符号或标点符号：

```cs
string r = @"(?x)^(?=.* ( \d | \p{P} | \p{S} )).{6,}";

Console.WriteLine (Regex.IsMatch ("abc12", r));     // False
Console.WriteLine (Regex.IsMatch ("abcdef", r));    // False
Console.WriteLine (Regex.IsMatch ("ab88yz", r));    // True
```

### 至少包含 80 个字符的行

```cs
string r = @"(?m)^.{80,}(?=\r?$)";

string fifty = new string ('x', 50);
string eighty = new string ('x', 80);

string text = eighty + "\r\n" + fifty + "\r\n" + eighty;

Console.WriteLine (Regex.Matches (text, r).Count);   // 2
```

### 解析日期/时间（N/N/N H:M:S AM/PM）

此表达式处理多种数字日期格式，并且无论年份先出现还是后出现都能正常工作。`(?x)` 指令通过允许空白符提高了可读性；`(?i)` 关闭了大小写敏感性（用于可选的 AM/PM 指示符）。然后，可以通过 `Groups` 集合访问每个匹配的组件：

```cs
string r = @"(?x)(?i)
 (\d{1,4}) [./-]
 **(\d{1,2}) [./-]**
 **(\d{1,4}) [\sT]**
 (\d+):(\d+):(\d+) \s? (A\.?M\.?|P\.?M\.?)?";

string text = "01/02/2008 5:20:50 PM";

foreach (Group g in Regex.Match (text, r).Groups)
  Console.WriteLine (g.Value + " ");
*01/02/2008 5:20:50 PM 01 02 2008 5 20 50 PM*
```

（当然，这并不验证日期/时间是否正确。）

### 匹配罗马数字

```cs
string r =
  @"(?i)\bm*"         +
  @"(d?c{0,3}|c[dm])" +
  @"(l?x{0,3}|x[lc])" +
  @"(v?i{0,3}|i[vx])" +
  @"\b";

Console.WriteLine (Regex.IsMatch ("MCMLXXXIV", r));   // True
```

### 删除重复的单词

在此，我们捕获了一个名为 `dupe` 的命名组：

```cs
string r = @"(?'dupe'\w+)\W\k'dupe'";

string text = "In the the beginning...";
Console.WriteLine (Regex.Replace (text, r, "${dupe}"));

*In the beginning*
```

### 单词计数

```cs
string r = @"\b(\w|[-'])+\b";

string text = "It's all mumbo-jumbo to me";
Console.WriteLine (Regex.Matches (text, r).Count);   // 5
```

### 匹配 GUID

```cs
string r =
  @"(?i)\b"           +
  @"[0-9a-fA-F]{8}\-" +
  @"[0-9a-fA-F]{4}\-" +
  @"[0-9a-fA-F]{4}\-" +
  @"[0-9a-fA-F]{4}\-" +
  @"[0-9a-fA-F]{12}"  +
  @"\b";

string text = "Its key is {3F2504E0-4F89-11D3-9A0C-0305E82C3301}.";
Console.WriteLine (Regex.Match (text, r).Index);                    // 12
```

### 解析 XML/HTML 标签

正则表达式对解析 HTML 片段非常有用——特别是当文档可能不完整时：

```cs
string r =
  @"<(?'tag'\w+?).*>"  +  // lazy-match first tag, and name it 'tag'
  @"(?'text'.*?)"      +  // lazy-match text content, name it 'textd'
  @"</\k'tag'>";          // match last tag, denoted by 'tag'

string text = "<h1>hello</h1>";

Match m = Regex.Match (text, r);

Console.WriteLine (m.Groups ["tag"]);       // h1
Console.WriteLine (m.Groups ["text"]);      // hello
```

### 拆分驼峰命名的单词

这需要使用 *正向预查* 来包括大写分隔符：

```cs
string r = @"(?=[A-Z])";

foreach (string s in Regex.Split ("oneTwoThree", r))
  Console.Write (s + " ");    // one Two Three
```

### 获得合法文件名

```cs
string input = "My \"good\" <recipes>.txt";

char[] invalidChars = System.IO.Path.GetInvalidFileNameChars();
string invalidString = Regex.Escape (new string (invalidChars));

string valid = Regex.Replace (input, "[" + invalidString + "]", "");
Console.WriteLine (valid);

*My good recipes.txt*
```

### 用于 HTML 的转义 Unicode 字符

```cs
string htmlFragment = "© 2007";

string result = Regex.Replace (htmlFragment, @"[\u0080-\uFFFF]",
                m => @"&#" + ((int)m.Value[0]).ToString() + ";");

Console.WriteLine (result);        // &#169; 2007
```

### 在 HTTP 查询字符串中取消转义字符

```cs
string sample = "C%23 rocks";

string result = Regex.Replace (
    sample,
    @"%[0-9a-f][0-9a-f]", 
    m => ((char) Convert.ToByte (m.Value.Substring (1), 16)).ToString(),
    RegexOptions.IgnoreCase
);

Console.WriteLine (result);   // C# rocks
```

### 从 web 统计日志中解析 Google 搜索术语

你应该与前面的示例结合使用来在查询字符串中取消转义字符：

```cs
string sample = 
  "http://google.com/search?hl=en&q=greedy+quantifiers+regex&btnG=Search";

Match m = Regex.Match (sample, @"(?<=google\..+search\?.*q=).+?(?=(&|$))");

string[] keywords = m.Value.Split (
  new[] { '+' }, StringSplitOptions.RemoveEmptyEntries);

foreach (string keyword in keywords)
  Console.Write (keyword + " ");       // greedy quantifiers regex
```

# 正则表达式语言参考

表格 25-2 至 25-12 总结了在 .NET 实现中支持的正则表达式语法和语法。

表 25-2\. 字符转义

| 转义代码序列 | 含义 | 十六进制等效 |
| --- | --- | --- |
| `\a` | 响铃符 | `\u0007` |
| `\b` | 退格符 | `\u0008` |
| `\t` | 制表符 | `\u0009` |
| `\r` | 回车符 | `\u000A` |
| `\v` | 垂直制表符 | `\u000B` |
| `\f` | 换页符 | `\u000C` |
| `\n` | 换行符 | `\u000D` |
| `\e` | Escape | `\u001B` |
| `\*nnn*` | ASCII 字符 *nnn*（如 `\n052`） |   |
| `\x*nn*` | ASCII 字符 *nn*（如 `\x3F`） |   |
| `\c*l*` | ASCII 控制字符 *l*（例如，`\cG` 表示 Ctrl-G） |   |
| `\u*nnnn*` | Unicode 字符 *nnnn*（如 `\u07DE`） |   |
| `\*symbol*` | 一个非转义的符号 |   |

特殊情况：在正则表达式中，`\b` 表示单词边界，但在 `[ ]` 集合中，`\b` 表示退格字符。

表 25-3\. 字符集

| 表达式 | 含义 | 反义（“非”） |
| --- | --- | --- |
| `[abcdef]` | 匹配列表中的单个字符 | `[^abcdef]` |
| `[a-f]` | 匹配范围内的单个字符 | `[^a-f]` |
| `\d` | 匹配一个十进制数字，等同于 `[0-9]` | `\D` |
| `\w` | 匹配一个*单词*字符（默认情况下，根据 `CultureInfo.CurrentCulture` 变化；例如，在英语中，与 `[a-zA-Z_0-9]` 相同） | `\W` |
| `\s` | 匹配空白字符，等同于 `[\n\r\t\f\v ]` | `\S` |
| `\p{*category*}` | 匹配指定*category*中的字符（参见表 25-4） | `\P` |
| `.` | （默认模式）匹配除 `\n` 外的任意字符 | `\n` |
| `.` | （*单行*模式）匹配任意字符，不包括 `\n` | `\n` |

表 25-4\. 字符类别

| 量词 | 意义 |
| --- | --- |
| `\p{L}` | 字母 |
| `\p{Lu}` | 大写字母 |
| `\p{Ll}` | 小写字母 |
| `\p{N}` | 数字 |
| `\p{P}` | 标点符号 |
| `\p{M}` | 重音符号 |
| `\p{S}` | 符号 |
| `\p{Z}` | 分隔符 |
| `\p{C}` | 控制字符 |

表 25-5\. 量词

| 量词 | 意义 |
| --- | --- |
| `*` | 零次或多次匹配 |
| `+` | 一次或多次匹配 |
| `?` | 零或一次匹配 |
| `{*n*}` | 恰好 `*n*` 次匹配 |
| `{*n*,}` | 至少 `*n*` 次匹配 |
| `{*n,m*}` | 匹配 `*n*` 到 `*m*` 次 |

可以将 `?` 后缀应用于任何量词，使其变为*懒惰*而不是*贪婪*。

表 25-6\. 替换

| 表达式 | 意义 |
| --- | --- |
| `$0` | 替换匹配的文本 |
| `$*group-number*` | 在匹配的文本中替换索引为 `*group-number*` 的文本 |
| `${*group-name*}` | 在匹配的文本中替换文本 `*group-name*` |

替换只在替换模式中指定。

表 25-7\. 零宽断言

| 表达式 | 意义 |
| --- | --- |
| `^` | 字符串开头（或在*多行*模式下是行首） |
| `$` | 字符串结尾（或在*多行*模式下是行尾） |
| `\A` | 字符串开头（忽略*多行*模式） |
| `\z` | 字符串结尾（忽略*多行*模式） |
| `\Z` | 行尾或字符串结尾 |
| `\G` | 开始搜索的位置 |
| `\b` | 在单词边界上 |
| `\B` | 不在单词边界上 |
| `(?=*expr*)` | 只有右边的表达式 `*expr*` 匹配时继续匹配（*正向先行断言*） |
| `(?!*expr*)` | 只有右边的表达式 `*expr*` 不匹配时继续匹配（*负向先行断言*） |
| `(?<=*expr*)` | 只有左边的表达式 `*expr*` 匹配时继续匹配（*正向后行断言*） |
| `(?<!*expr*)` | 只有左边的表达式 `*expr*` 不匹配时继续匹配（*负向后行断言*） |
| `(?>*expr*)` | 子表达式 `*expr*` 仅匹配一次，不回溯 |

表 25-8\. 分组结构

| 语法 | 意义 |
| --- | --- |
| `(*expr*)` | 将匹配的表达式 `*expr*` 捕获到索引组中 |
| `(?*number*)` | 将匹配的子字符串捕获到指定的组 `*number*` |
| `(?'*name*')` | 将匹配的子字符串捕获到组 `*name*` 中 |
| `(?'*name1-name2*')` | 取消定义`*name2*`并将区间和当前组存储到`*name1*`；如果`*name2*`未定义，则匹配回溯 |
| `(?:*expr*)` | 非捕获组 |

表 25-9\. 后向引用

| 参数语法 | 含义 |
| --- | --- |
| `\*index*` | 根据`*index*`引用先前捕获的组 |
| `\k<*name*>` | 根据`*name*`引用先前捕获的组 |

表 25-10\. 选择

| 表达式语法 | 含义 |
| --- | --- |
| `&#124;` | 逻辑*或* |
| `(?(*expr*)*yes*&#124;*no*)` | 如果表达式匹配，则匹配`*yes*`；否则，匹配`*no*`（`*no*`为可选项） |
| `(?(*name*)*yes*&#124;*no*)` | 如果命名组有匹配，则匹配`*yes*`；否则，匹配`*no*`（`*no*`为可选项） |

表 25-11\. 杂项构造

| 表达式语法 | 含义 |
| --- | --- |
| `(?#*comment*)` | 行内注释 |
| `#*comment*` | 行尾注释（仅在`IgnorePatternWhitespace`模式下有效） |

表 25-12\. 正则表达式选项

| 选项 | 含义 |
| --- | --- |
| `(?i)` | 大小写不敏感匹配（忽略大小写） |
| `(?m)` | 多行模式；改变`^`和`$`以匹配任何行的开头和结尾 |
| `(?n)` | 仅捕获显式命名或编号的组 |
| `(?c)` | 编译为中间语言 |
| `(?s)` | 单行模式；改变“.”的含义以匹配每个字符 |
| `(?x)` | 从模式中消除未转义的空白 |
| `(?r)` | 从右到左搜索；不能在中间指定 |
