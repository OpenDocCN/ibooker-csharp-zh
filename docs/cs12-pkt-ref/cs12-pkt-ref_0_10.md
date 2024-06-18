# 第十章：空值操作符

C# 提供了三个操作符，用于更方便地处理空值：*null 合并运算符*，*null 条件运算符* 和 *null 合并赋值运算符*。

## 空值合并运算符

`??` 运算符是 *null**-**合并运算符*。它表示，“如果左边的操作数非空，则将其给我；否则，给我另一个值。”例如：

```cs
string s1 = null;
string s2 = s1 ?? "nothing"; // s2 evaluates to "nothing"
```

如果左侧表达式非空，则不会评估右侧表达式。空值合并运算符也适用于可空值类型（参见“可空值类型”）。

## 空值合并赋值运算符

`??=` 运算符（C# 8 中引入）是 *null 合并赋值运算符*。它表示，“如果左边的操作数为空，则将右操作数赋给左操作数。”考虑以下示例：

```cs
myVariable ??= someDefault;
```

这等效于：

```cs
if (myVariable == null) myVariable = someDefault;
```

## 空值条件运算符

`?.`运算符是*空值条件*或“Elvis”运算符。它允许您调用方法和访问成员，就像标准点运算符一样，但如果左边的操作数为空，则表达式会评估为 null，而不是抛出`NullReferenceException`：

```cs
System.Text.StringBuilder sb = null;
string s = sb?.ToString();   // No error; s is null
```

最后一行等同于此内容：

```cs
string s = (sb == null ? null : sb.ToString());
```

空值条件表达式也适用于索引器：

```cs
string[] words = null;
string word = words?[1];   // word is null
```

遇到空值时，Elvis 运算符会短路表达式的其余部分。在以下示例中，即使在`ToString()`和`ToUpper()`之间使用了标准点运算符，`s`仍然评估为 null：

```cs
System.Text.StringBuilder sb = null;
string s = sb?.ToString().ToUpper();   // No error
```

只有在左边的操作数可能为空时，才需要重复使用 Elvis 运算符。以下表达式可以确保`x`为空或者`x.y`为空时也能正常运行：

```cs
x?.y?.z
```

这等同于以下内容（不同之处在于`x.y`只会被评估一次）：

```cs
x == null ? null 
          : (x.y == null ? null : x.y.z)
```

最后的表达式必须能够接受空值。以下示例是非法的，因为`int`不能接受空值：

```cs
System.Text.StringBuilder sb = null;
int length = sb?.ToString().Length;   // Illegal
```

我们可以通过使用可空值类型来修复此问题（参见“可空值类型”）：

```cs
int? length = sb?.ToString().Length;
// OK : int? can be null
```

您还可以使用空值条件运算符来调用一个`void`方法：

```cs
someObject?.SomeVoidMethod();
```

如果`someObject`为空，这将成为“无操作”，而不是抛出`NullReferenceException`。

空值条件运算符可以与我们在“类”中描述的常用类型成员一起使用，包括*方法*、*字段*、*属性*和*索引器*。它还可以与空值合并运算符很好地结合使用：

```cs
System.Text.StringBuilder sb = null;
string s = sb?.ToString() ?? "nothing";
// s evaluates to "nothing"
```

