# 第三十章：扩展方法

*扩展方法* 允许在不改变原始类型定义的情况下为现有类型添加新方法。扩展方法是静态类的静态方法，其中第一个参数应用了 `this` 修饰符。第一个参数的类型将是扩展的类型。例如：

```cs
public static class StringHelper
{
  public static bool IsCapitalized (this string s)
  {
    if (string.IsNullOrEmpty (s)) return false;
    return char.IsUpper (s[0]);
  }
}
```

`IsCapitalized` 扩展方法可以被调用，就像它是一个字符串上的实例方法一样，如下所示：

```cs
Console.Write ("Perth".IsCapitalized());
```

扩展方法调用在编译时会被转换回普通的静态方法调用：

```cs
Console.Write (StringHelper.IsCapitalized ("Perth"));
```

接口也可以被扩展：

```cs
public static T First<T> (this IEnumerable<T> sequence)
{
  foreach (T element in sequence)
    return element;
  throw new InvalidOperationException ("No elements!");
}
...
Console.WriteLine ("Seattle".First());   // S
```

## 扩展方法链

扩展方法与实例方法一样，提供了一种整洁的方法来链式调用函数。考虑以下两个函数：

```cs
public static class StringHelper
{
  public static string Pluralize (this string s) {...}
  public static string Capitalize (this string s) {...}
}
```

`x` 和 `y` 是等价的，都会评估为`"Sausages"`，但 `x` 使用扩展方法，而 `y` 使用静态方法：

```cs
string x = "sausage".Pluralize().Capitalize();

string y = StringHelper.Capitalize
           (StringHelper.Pluralize ("sausage"));
```

## 歧义和解决方案

任何兼容的实例方法总是优先于扩展方法—即使扩展方法的参数更具体匹配类型。

如果两个扩展方法具有相同的签名，则必须将扩展方法调用为普通静态方法，以消除调用方法的歧义。然而，如果一个扩展方法具有更具体的参数，则更具体的方法优先。

