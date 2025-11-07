# 附录 A 练习题答案

本附录提供了书中练习的答案以及答案的解释。

## 第二章：.NET 和它的编译方式

| 练习题号 | 答案 | 解释 |
| --- | --- | --- |
| 2.1 | d | AmigaOS 是最初为 Amiga PC 开发的操作系统。它的最后一个主要版本发布于 2016 年。它不支持 .NET 5。 |
| 2.2 | c |  |
| 2.3 | d |  |
| 2.4 | a |  |
| 2.5 | b, a |  |
| 2.6 | e |  |
| 2.7 | c |  |
| 2.8 | a |  |

## 第四章：管理你的非托管资源！

| 练习题号 | 答案 | 解释 |
| --- | --- | --- |
| 4.1 | 错误 | 属性可以应用于方法、类、类型、属性和字段。 |
| 4.2 | 正确 |  |
| 4.3 | 错误 | 枚举是用 `enum` 关键字创建的。 |
| 4.4 | a, d | 只有 Hufflepuff 才会在便利贴上写连接字符串。 |
| 4.5 | b |  |
| 4.6 | a |  |
| 4.7 | c |  |
| 4.8 | 正确 |  |
| 4.9 | 错误 |  |

## 第五章：使用 Entity Framework Core 设置项目和数据库

| 练习题号 | 答案 | 解释 |
| --- | --- | --- |
| 5.1 | b |  |
| 5.2 | a |  |
| 5.3 | a, d | Tomcat 是 Java Servlet 的开源实现（类似于 WebHost）；JVM 是 Java 虚拟机（类似于 CLR）。 |
| 5.4 | 正确 |  |
| 5.5 | 查看下表以获取答案 |  |
| 5.6 | 每个数据库实体一个 |  |

练习题 5.5 的答案：解决这个练习有多种方法，可以是一行代码。你可以从各种访问修饰符、方法名称和变量名称中选择。然而，有两个常数：返回类型需要是 `integer` 类型，并且我们需要返回两个整数输入参数的乘积，如下所示：

```
public int Product(int a, int b) => a * b;
```

## 第六章：测试驱动开发和依赖注入

| 练习题号 | 答案 | 解释 |
| --- | --- | --- |
| 6.1 | c | 答案 B（“不要在两个不同的地方执行相同的逻辑”）描述了 Don’t Repeat Yourself（DRY）原则。 |
| 6.2 | 正确 | 然而，在书中，我们使用 TDD-lite，有时会打破这个规则。 |
| 6.3 | 错误 | 测试类必须具有 `public` 访问修饰符，才能被测试运行器使用。 |
| 6.4 | c |  |
| 6.5 | 错误 | LINQ 允许我们使用类似 SQL 的语句和方法对集合进行查询。 |
| 6.6 | a |  |
| 6.7 | b |  |
| 6.8 | 正确 |  |
| 6.9 | a |  |
| 6.10 | 正确 |  |

6.2.8 节中找到的练习的解决方案：

```
[TestMethod]
public async Task CreateCustomer_Success()
{
    CustomerRepository repository = new CustomerRepository();
    Assert.IsNotNull(repository);

    bool result = await repository.CreateCustomer("Donald Knuth");
    Assert.IsTrue(result);
}

[TestMethod]
public async Task CreateCustomer_Failure_NameIsNull()
{
    CustomerRepository repository = new CustomerRepository();
    Assert.IsNotNull(repository);

    bool result = await repository.CreateCustomer(null);
    Assert.IsFalse(result);
}

[TestMethod]
public async Task CreateCustomer_Failure_NameIsEmptyString()
{
    CustomerRepository repository = new CustomerRepository();
    Assert.IsNotNull(repository);
    bool result = await repository.CreateCustomer(string.Empty);
    Assert.IsFalse(result);
}

[TestMethod]
[DataRow('#')]
[DataRow('$')]
[DataRow('%')]
[DataRow('&')]
[DataRow('*')]
public async Task CreateCustomer_Failure_NameContainsInvalidCharacters(char
➥ invalidCharacter)
{
    CustomerRepository repository = new CustomerRepository();
    Assert.IsNotNull(repository);

    bool result = await repository.CreateCustomer("Donald Knuth" + invalidCharacter);
    Assert.IsFalse(result);
}
```

## 第七章：比较对象

| 练习题号 | 答案 | 解释 |
| --- | --- | --- |
| 7.1 | 编写一个使用异常发现方法属性的单元测试。 |  |
| 7.2 | b |  |
| 7.3 | `Exception` | 您也可以从一个不同的 `Exception`（自定义或非自定义）派生一个自定义异常，该异常继承自 `Exception` 类。 |
| 7.4 | c | A 和 B 可以是集合底层类型的默认值，因此在某些情况下是正确的。 |
| 7.5 | c | 当比较值类型时，相等运算符比较它们的值。 |
| 7.6 | a | 当比较引用类型时，相等运算符比较它们的内存地址。 |
| 7.7 | 正确 |  |
| 7.8 | a | 当重载运算符时，你还需要重载其逆运算符。 |
| 7.9 | 错误 | 计算机中不存在完美的随机性。 |
| 7.10 | 错误 | 计算机中不存在完美的随机性。 |
| 7.11 | 正确 |  |

## 第八章：存根、泛型和耦合

| 练习编号 | 答案 | 说明 |
| --- | --- | --- |
| 8.1 | c |  |
| 8.2 | 错误 | 两个高度相互依赖的类表示紧密耦合。 |
| 8.3 | a |  |
| 8.4 | c |  |
| 8.5 | 错误 | 字符串是不可变的。对字符串所做的任何更改都会导致新的内存分配，并将结果字符串存储在内存中的相应位置。 |
| 8.6 | 错误 | 你必须 `重写` 基类的方法。 |
| 8.7 | a | DRY 原则代表“不要重复自己”原则。Phragmén–Lindelöf 原理处理在无界域上全纯函数的有界性。 |
| 8.8 | b |  |
| 8.9 | c |  |
| 8.10 | 错误 | 泛型可以与类、方法和集合一起使用。 |
| 8.11 | 错误 |  |
| 8.12 | 错误 |  |
| 8.13 | 正确 |  |
| 8.14 | b |  |
| 8.15 | 错误 | 如果你没有在 `switch` 语句中声明 `default` 情况，并且没有其他情况匹配，则 `switch` 语句内不会执行任何操作。 |

## 第九章：扩展方法、流和抽象类

| 练习编号 | 答案 | 说明 |
| --- | --- | --- |
| 9.1 | b |  |
| 9.2 | a |  |
| 9.3 | 错误 | 你可以使用 `[DataRow]` 属性方法与任意数量的数据点一起使用。 |
| 9.4 | b |  |
| 9.5 | 错误 |  |
| 9.6 | 错误 | 抽象类可以包含抽象方法和常规方法。 |
| 9.7 | 错误 |  |
| 9.8 | 正确 |  |

## 第十章：反射和模拟

| 练习编号 | 答案 | 说明 |
| --- | --- | --- |
| 10.1 | b |  |
| 10.2 | 错误 | 当使用 ORM 时，仓库层通常通过数据库访问层与数据库交互。 |
| 10.3 | 正确 |  |
| 10.4 | 正确 |  |
| 10.5 | a |  |
| 10.6 | c | 我们永远不希望在不担心副作用的情况下删除代码。当遇到注释掉的代码时，要尽职尽责地找出它存在的原因。最好有一个非常好的理由，否则就删除它。绝大多数情况下，你可以无忧地删除代码。 |
| 10.7 | d |  |
| 10.8 | a |  |
| 10.9 | 正确 |  |
| 10.10 | c |  |
| 10.11 | 正确 | 虽然控制器和仓库之间仍然存在一些耦合，但与控制器直接调用仓库相比，这是一种更松散的耦合。 |
| 10.12 | 错误 | 这就是存根的功能。 |
| 10.13 | 错误 | `InternalsVisibleTo` 允许你将程序集的内部结构暴露给另一个程序集。 |
| 10.14 | c |  |
| 10.15 | 正确 |  |
| 10.16 | b |  |
| 10.17 | a | `[MethodImpl(MethodImplOptions.NoInlining)]` 只能应用于方法。 |

## 第十一章：重新审视运行时类型检查和错误处理

| 练习编号 | 答案 | 说明 |
| --- | --- | --- |
| 11.1 | 错误 |  |
| 11.2 | c |  |
| 11.3 | a | 只允许服务调用存储库。存储库不应该调用另一个存储库。 |
| 11.4 | 正确 |  |
| 11.5 | b |  |
| 11.6 | 错误 | 抛弃操作符仍然可能导致内存分配。 |
| 11.7 | a | 第一个`catch`块被进入，因为`ItemSoldOutException`可以用作`Exception`类型。 |
| 11.8 | a |  |

## 第十二章：使用 IAsyncEnumerable<T>和 yield return

| 练习题号 | 答案 | 说明 |
| --- | --- | --- |
| 12.1 | 正确 |  |
| 12.2 | 错误 | 尽管如此，我们可能需要实现`InventoryService`。 |
| 12.3 | b |  |
| 12.4 | a |  |
| 12.5 | 是 | `Dragonfruit`类可以设置`IsFruit`属性，因为`IsFruit`属性有一个`protected`访问修饰符。具有`protected`访问修饰符的属性可以被拥有它的类及其子类（派生类）访问。`Dragonfruit`类是从`Fruit`类派生出来的。 |
| 12.6 | 正确 |  |
| 12.7 | 正确 |  |
| 12.8 | 错误 | 当你向结构体添加构造函数时，需要设置结构体上存在的所有属性，否则编译器将不会编译你的代码。 |

## 第十三章：中间件、HTTP 路由和 HTTP 响应

| 练习题号 | 答案 | 说明 |
| --- | --- | --- |
| 13.1 | 正确 |  |
| 13.2 | c |  |
| 13.3 | a |  |
| 13.4 | 正确 |  |
| 13.5 | c |  |
| 13.6 | b |  |
