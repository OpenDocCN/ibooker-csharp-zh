# 第二十章：嵌套类型

*嵌套类型* 声明在另一个类型的范围内。例如：

```cs
public class TopLevel
{
  public class Nested { }               // Nested class
  public enum Color { Red, Blue, Tan }  // Nested enum
}
```

嵌套类型具有以下特征：

+   它可以访问封闭类型的私有成员以及封闭类型可以访问的其他所有内容。

+   它可以声明具有全范围访问修饰符，而不仅限于 `public` 和 `internal`。

+   嵌套类型的默认可访问性是 `private` 而不是 `internal`。

+   从封闭类型外部访问嵌套类型需要使用封闭类型名称进行限定（就像访问静态成员时一样）。

例如，要从我们的 `TopLevel` 类外部访问 `Color.Red`，你需要这样做：

```cs
TopLevel.Color color = TopLevel.Color.Red;
```

所有类型都可以嵌套；但是，只有类和结构可以嵌套。

