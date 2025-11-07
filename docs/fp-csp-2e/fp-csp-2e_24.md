# 附录 A.使用 C#的早期版本

这本书的第二版是为 C# 10 编写的，并利用了语言最新的功能，前提是它们与函数式编程相关。如果你正在处理使用 C# 早期版本的遗留项目，你仍然可以应用这本书中讨论的所有想法。本附录向你展示了如何做到这一点。

## A.1 C# 9 之前的不可变数据对象

在这本书中，我使用了记录和结构体来处理所有数据对象。记录默认是不可变的，结构体在函数间按值复制时也是复制的，因此它们也被视为不可变。如果你想要使用不可变数据对象，但需要使用 C# 9 之前的版本，你必须依赖以下选项之一：

+   按惯例将对象视为不可变。

+   手动定义不可变对象。

+   使用 F# 作为你的领域对象。

为了说明这些策略中的每一个，我将回到编写 `AccountState` 类的任务，以表示 BOC 应用程序中银行账户的状态。我们在第 11.3 节中看到了这个。

### A.1.1 按惯例的不可变性

在引入记录之前，C# 开发者通常使用空构造函数和属性获取器和设置器来定义数据对象。以下列表展示了如何使用这种方法来模拟银行账户的状态。

列表 A.1 银行账户状态的简单模型

```
public enum AccountStatus
{ Requested, Active, Frozen, Dormant, Closed }

public class AccountState
{
   public AccountStatus Status { get; set; }
   public CurrencyCode Currency { get; set; }
   public decimal AllowedOverdraft { get; set; }
   public List<Transaction> TransactionHistory { get; set; }

   public AccountState()
     => TransactionHistory = new List<Transaction>();
}
```

这允许我们以如下列表所示的对象初始化器语法优雅地创建新实例。

列表 A.2 使用方便的对象初始化器语法

```
var account = new AccountState
{
   Currency = "EUR"
};
```

这创建了一个新的账户，其中 `Currency` 属性被显式设置；其他属性初始化为它们的默认值。请注意，对象初始化器语法调用无参数构造函数和 `AccountState` 中定义的公共设置器。

### A.1.2 定义复制方法

如果我们要表示状态的变化，例如账户被冻结，我们将创建一个新的 `AccountState`，并带有新的 `Status`。我们可以通过在 `AccountState` 上添加一个方便的方法来实现，如下列表所示。

列表 A.3 定义复制方法

```
public class AccountState
{
   public AccountState WithStatus(AccountStatus newStatus)
      => new AccountState
      {
         Status = newStatus,                            ❶
         Currency = this.Currency,                      ❷
         AllowedOverdraft = this.AllowedOverdraft,      ❷
         TransactionHistory = this.TransactionHistory   ❷
      };
}
```

❶ 更新字段

❷ 所有其他字段都从当前状态复制。

`WithStatus` 是一个返回实例副本的方法，除了 `Status` 以外，其他方面都与原始实例相同，`Status` 的值如所给。这与使用 `AddDays` 和在 `DateTime` 上定义的类似方法得到的行为相似：它们都返回一个新的实例（参见第 11.2.1 节）。

类似于 `WithStatus` 这样的方法被称为 *复制方法* 或 *with-ers*，因为惯例是给它们命名为 `With[Property]`。以下列表展示了如何调用一个复制方法来表示账户状态的改变。

列表 A.4 获取对象的修改版本

```
var newState = account.WithStatus(AccountStatus.Frozen);
```

复制方法与记录中的 `with` 表达式类似，因为它们返回原始对象的副本，其中一个属性已被更新。

注意：通过复制方法表示更改的成本并不像你想象的那么高，正如第 11.3 节中已经讨论的那样（特别是在“使用不可变对象对性能的影响”侧边栏中）。这是因为像`WithStatus`这样的复制方法会创建原始对象的**浅拷贝**：这是一个快速且足以保证安全性的操作（假设对象的所有子对象也都是不可变的）。

### A.1.3 强制不可变性

到目前为止所示的实施方案使用属性 setter 来最初填充对象（A.1.1 节）和使用复制方法来获取更新版本（A.1.2 节）。这种方法被称为**约定不可变性**：你使用约定和纪律来避免突变。setter 是公开的，但它们应该在对象初始化后 never 被调用。但这并不能阻止一个不认同不可变性的淘气的同事直接设置字段：

```
account.Status = AccountStatus.Frozen;
```

如果你想要防止这种破坏性更新，你必须通过完全移除属性 setter 来使你的对象不可变。然后，新的实例必须通过将所有值作为参数传递给构造函数来填充，如下列所示。

列表 A.5：朝着不可变性重构：移除所有 setter

```
public class AccountState
{
   public AccountStatus Status { get; }
   public CurrencyCode Currency { get; }
   public decimal AllowedOverdraft { get; }
   public List<Transaction> Transactions { get; }

   public AccountState
   (
      CurrencyCode Currency,
      AccountStatus Status = AccountStatus.Requested,
      decimal AllowedOverdraft = 0,
      List<Transaction> Transactions = null
   )
   {
      this.Status = Status;
      this.Currency = Currency;
      this.AllowedOverdraft = AllowedOverdraft;
      this.Transactions = Transactions ?? new List<Transaction>();
   }

   public AccountState WithStatus(AccountStatus newStatus)
      => new AccountState
      (
         Status: newStatus,
         Currency: this.Currency,
         AllowedOverdraft: this.AllowedOverdraft,
         Transactions: this.TransactionHistory
      );
}
```

在构造函数中，我使用了命名参数和默认值，这样我就可以使用类似于我们之前使用的对象初始化器语法来创建一个新的实例。现在我们可以用如下方式创建一个新的账户，并赋予其合理的值：

```
var account = new AccountState
(
   Currency: "EUR",
   Status: AccountStatus.Active
);
```

`WithStatus`复制方法与之前一样工作。注意，我们现在强制为`Currency`提供一个值，这在使用对象初始化器语法时是不可能的。因此，我们在保持可读性的同时，使实现更加健壮。

提示：强制你的代码客户端使用构造函数或工厂函数来实例化对象可以提高代码的健壮性，因为你可以在这个点强制执行业务规则，使得无法创建处于无效状态的对象，例如没有货币的账户。

### A.1.4 一直到底的不可变性

我们还没有完成，因为对于一个对象来说，要成为不可变的，它的所有组成部分也必须是不可变的。在这里，我们使用了一个可变的`List`，所以你的淘气的同事仍然可以通过编写以下代码来有效地突变账户状态：

```
account.Transactions.Clear();
```

防止这种情况最有效的方法是创建构造函数提供的列表的副本，并将内容存储在一个不可变列表中。以下列表显示了如何使用`System.Collections.Immutable`库中的`ImmutableList`类型来完成此操作。¹

列表 A.6：通过使用不可变集合防止突变

```
using System.Collections.Immutable;

public sealed class AccountState                               ❶
{
   public IEnumerable<Transaction> TransactionHistory { get; }

   public AccountState(CurrencyCode Currency
      , AccountStatus Status = AccountStatus.Requested
      , decimal AllowedOverdraft = 0
      , IEnumerable<Transaction> Transactions = null)
   {
      // ...
      TransactionHistory = ImmutableList.CreateRange           ❷
         (Transactions ?? Enumerable.Empty<Transaction>());    ❷
   }
}
```

❶ 将类标记为密封以防止可变子类

❷ 创建并存储给定列表的防御性副本

当创建一个新的`AccountState`时，给定的交易列表会被复制并存储在一个`ImmutableList`中。这被称为*防御性复制*。现在，`AccountState`的交易列表不能被任何消费者更改，即使构造函数中给出的列表在以后被更改，它也不会受到影响。幸运的是，`CreateRange`足够智能，如果它被给定了`ImmutableList`，它就会直接返回它，这样复制方法就不会产生任何额外的开销。

此外，`Transaction`和`Currency`也必须是不可变类型。我还将`AccountState`标记为`sealed`以防止创建可变的子类。现在，`AccountState`在理论上确实是不可变的。在实践中，仍然可以通过反射来修改实例，这样你那淘气的同事仍然可以占据上风。² 但至少现在没有通过错误修改对象的空间了。

你如何将一笔新交易添加到列表中？你不需要这样做。你创建一个新的列表，其中包含新交易以及所有现有交易，并且这个新列表将成为一个新的`AccountState`的一部分，如下面的列表所示。

列表 A.7 向列表添加元素需要一个新的父对象

```
using LaYumba.Functional;                               ❶

public sealed class AccountState
{
   public AccountState Add(Transaction t)
      => new AccountState
      (
         Transactions: TransactionHistory.Prepend(t),   ❷
         Currency: this.Currency,                       ❸
         Status: this.Status,                           ❸
         AllowedOverdraft: this.AllowedOverdraft        ❸
      );
}
```

❶ 将`Prepend`作为`IEnumerable`的扩展方法

❷ 包含现有值和即将添加的一个新的`IEnumerable`

❸ 所有其他字段都按常规复制。

注意，在这个特定的情况下，我们是在列表中*预加*交易。这是特定领域的；在大多数情况下，你感兴趣的是最新的交易，所以最有效的方法是将最新的交易放在列表的前面。

### A.1.5 无样板复制方法？

现在我们已经成功地将`AccountState`实现为一个不可变类型，让我们面对一个痛点：*编写复制方法并不有趣!* 想象一个有 10 个属性的对象，所有这些属性都需要复制方法。如果有任何集合，你需要将它们复制到不可变集合中，并添加复制方法来添加或删除这些集合中的项目。这有很多样板代码！

下面的列表展示了如何通过包含一个带有命名可选参数的单个`With`方法来减轻这种情况，就像我们在列表 A.5 中的`AccountState`构造函数中使用它们一样。

列表 A.8 一个可以设置任何属性的单一`With`方法

```
public AccountState With
(
   AccountStatus? Status = null,                                 ❶
   decimal? AllowedOverdraft = null                              ❶
 )
=> new AccountState
(
   Status: Status ?? this.Status,                                ❷
   AllowedOverdraft: AllowedOverdraft ?? this.AllowedOverdraft,  ❷
   Currency: this.Currency,                                      ❸
   Transactions: this.TransactionHistory                         ❸
);
```

❶ `null`表示该字段未指定。

❷ 如果未指定值，则使用当前实例的值

❸ 你可以防止任意更改。

`null` 的默认值表示值尚未指定；在这种情况下，将使用当前实例的值来填充复制。对于值类型字段，你可以使用相应的可空类型作为参数类型，以允许 `null` 作为默认值。因为默认值 `null` 表示字段尚未指定，因此将使用当前值，所以不可能使用此方法将字段设置为 `null`。鉴于第 5.5.1 节中关于 `null` 与 `Option` 的讨论，你可能已经看出这并不是一个好主意。

注意，在列表 A.8 中，我们只允许更改两个字段，因为我们假设我们永远不能更改银行账户的货币或对交易历史进行任意更改。这种方法使我们能够在保留对想要允许的操作的细粒度控制的同时减少样板代码。使用方法如下：

```
public static AccountState Freeze(this AccountState account)
   => account.With(Status: AccountStatus.Frozen);

public static AccountState RedFlag(this AccountState account)
   => account.With
   (
      Status: AccountStatus.Frozen,
      AllowedOverdraft: 0m
   );
```

这不仅读起来清晰，而且与使用经典的 `With[Property]` 方法相比，性能更好：如果我们需要更新多个字段，则只创建一个新实例。我强烈推荐使用这个单一的 `With` 方法，而不是为每个字段定义复制方法。

另一种方法是定义一个通用的辅助器，它执行复制和更新而不需要任何样板代码。我在 `LaYumba.Functional.Immutable` 类中实现了一个这样的通用 `With` 方法，如下所示。

列表 A.9 使用通用复制方法

```
using LaYumba.Functional;

var oldState = new AccountState("EUR", AccountStatus.Active);
var newState = oldState.With(a => a.Status, AccountStatus.Frozen);

oldState.Status   // => AccountStatus.Active
newState.Status   // => AccountStatus.Frozen
newState.Currency // => "EUR"
```

在这里，`With` 是一个在 `object` 上的扩展方法，它接受一个 `Expression` 来标识要更新的属性和新的值。使用反射，它随后创建原始对象的位复制，识别指定属性的备份字段，并将其设置为给定值。

简而言之，它做了我们想要的事情——对于任何字段和任何类型。优点是，这使我们免于编写繁琐的复制方法。缺点是，反射相对较慢，并且当我们显式选择在 `With` 中可以更新的字段时，我们失去了细粒度的控制。

### A.1.6 比较不可变性的策略

总结来说，在 C# 9 引入记录之前，强制不可变性是一个棘手的问题，并且在函数式编程时也是一个最大的障碍。

这里是我所讨论的两种方法的优缺点：

+   *约定不可变性*——在这种方法中，你不需要做任何额外的工作来*防止*突变；你只是像可能避免使用 `goto`、`unsafe` 指针访问和位操作（仅举几例，这些是语言允许但已被证明有问题的操作）一样避免它。如果你独立工作或与从第一天起就支持这种方法的团队一起工作，这可以是一个可行的选择。当然，缺点是突变可能会悄悄地出现。

+   *在 C# 中定义不可变对象*——这种方法为你提供了一个更健壮的模型，它向其他开发者传达了该对象不应该被修改的信息。如果你在一个项目中工作，其中并没有全面使用不可变性，那么这种方法是首选的。与约定不可变性相比，它至少需要在定义构造函数时做一些额外的工作。

要使事情更加复杂，第三方库可能有限制，这些限制决定了你的选择。传统上，.NET 的反序列化器和 ORM 使用空构造函数和可设置属性来创建和填充对象。如果你依赖于具有此类要求的库，约定不可变性可能就是你的唯一选择。

## A.2 在 C# 8 之前进行模式匹配

模式匹配是一种语言特性，允许你根据某些数据（最重要的是其类型）的 *形状* 执行不同的代码。它是静态类型函数式语言的一个基本特性，我们在书中广泛使用了它，无论是通过 `switch` 表达式还是通过定义一个 `Match` 方法。

在本节中，我将描述模式匹配在 C# 中随版本演变的支持情况，并展示一个解决方案，即使你正在使用较旧的 C# 版本，也可以使用模式匹配。

### A.2.1 C# 对模式匹配的增量支持

很长一段时间，C# 对模式匹配的支持都很差。直到 C# 7，`switch` 语句只支持非常有限的形式的模式匹配，允许你匹配表达式的确切值。那么，匹配表达式的 *类型* 呢？例如，假设你有一个以下简单的领域：

```
enum Ripeness { Green, Yellow, Brown }

abstract class Reward { }

class Peanut : Reward { }
class Banana : Reward { public Ripeness Ripeness; }
```

在 C# 6 之前，计算给定 `Reward` 的描述必须像以下列表所示那样进行。

列表 A.10 在 C# 6 中对表达式类型进行匹配

```
string Describe(Reward reward)
{
   Peanut peanut = reward as Peanut;
   if (peanut != null)
      return "It's a peanut";

   Banana banana = reward as Banana;
   if (banana != null)
      return $"It's a {banana.Ripeness} banana";

   return "It's a reward I don't know or care about";
}
```

对于这样一个简单的操作，这确实非常繁琐且嘈杂。C# 7 引入了一些对模式匹配的有限支持，以便前面的代码可以像以下列表所示那样简化。

列表 A.11 使用 `is` 在 C# 7 中进行类型匹配

```
string Describe(Reward reward)
{
   if (reward is Peanut _)
      return "It's a peanut";

   if (reward is Banana banana)
      return $"It's a {banana.Ripeness} banana";

   return "It's a reward I don't know or care about";
}
```

或者，也可以使用以下列表所示的 `switch` 语句。

列表 A.12 使用 `switch` 在 C# 7 中进行类型匹配

```
string Describe(Reward reward)
{
   switch (reward)
   {
      case Peanut _:
         return "It's a peanut";
      case Banana banana:
         return $"It's a {banana.Ripeness} banana";
      default:
         return "It's a reward I don't know or care about";
   }
}
```

这仍然相当尴尬，尤其是在函数式编程中，我们希望使用表达式，而 `if` 和 `switch` 都在每个分支中期望语句。

最后，C# 8 引入了 `switch` 表达式（你在书中看到了几个例子），允许我们将前面的代码写成以下列表所示的形式。

列表 A.13 C# 8 中的 `switch` 表达式

```
string Describe(Reward reward)
   => reward switch
   {
       Banana banana => $"It's a {banana.Ripeness} banana",
       Peanut _ => "It's a peanut",
       _ => "It's a reward I don't know or care about"
   };
```

### A.2.2 用于模式匹配表达式的自定义解决方案

如果你正在使用 C# 8 之前的版本的代码库，你仍然可以使用我包含在 `LaYumba.Functional` 中的 `Pattern` 类来进行类型匹配。它可以像以下列表所示那样使用。

列表 A.14 用于基于表达式的模式匹配的自定义 `Pattern` 类

```
string Describe(Reward reward)
   => new Pattern<string>                                 ❶
   {
      (Peanut _) => "It's a peanut",                      ❷
      (Banana b) => $"It's a {b.Ripeness} banana"         ❷
   }
   .Default("It's a reward I don't know or care about")   ❸
   .Match(reward);                                        ❹
```

❶ 泛型参数指定调用 `Match` 时返回的类型。

❷ 一个函数列表；第一个匹配类型的函数将被评估。

❸ 可选地添加默认值或处理程序

❹ 提供要匹配的值

这不如一等语言支持性能好，也没有所有像解构这样的功能，但如果您只是对类型匹配感兴趣，这仍然是一个很好的解决方案。

您首先设置处理每个情况的函数（内部，`Pattern` 实质上是一个函数列表，所以我使用了列表初始化语法）。您可以可选地调用 `Default` 来提供一个默认值或一个函数，如果找不到匹配的函数则使用。最后，您使用 `Match` 来提供要匹配的值；这将评估第一个输入类型与给定值类型匹配的函数。

此外，`Pattern` 还有一个非泛型版本，其中 `Match` 返回 `dynamic`。您可以在前面的示例中通过简单地省略 `<string>` 来使用这个版本，从而使语法更加简洁。

提示：在这本书中，您看到了为 `Option`、`Either`、`List`、`Tree` 等实现的 `Match` 方法。这些方法有效地执行模式匹配。当您一开始就知道需要处理的所有情况时（例如，`Option` 只能是 `Some` 或 `None`），定义此类方法是有意义的。相比之下，`Pattern` 类对于开放继承的类型很有用，如 `Event` 或 `Reward`，您可以根据系统的发展添加新的子类。

## A.3 重新审视事件源示例

为了说明之前描述的技术，让我们回顾一下 13.2 节中的事件源场景，并假设我们只能使用 C# 6。我们没有记录，所以为了表示账户的状态，我们将定义 `AccountState` 为一个不可变类。所有属性都将只读，并在构造函数中填充。以下列表显示了实现。

列表 A.15 表示账户状态的不可变类

```
public sealed class AccountState
{
   public AccountStatus Status { get; }                     ❶
   public CurrencyCode Currency { get; }                    ❶
   public decimal Balance { get; }                          ❶
   public decimal AllowedOverdraft { get; }                 ❶

   public AccountState
   (
      CurrencyCode Currency,
      AccountStatus Status = AccountStatus.Requested,
      decimal Balance = 0m,
      decimal AllowedOverdraft = 0m
   )
   {
      this.Currency = Currency;                             ❷
       this.Status = Status;                                ❷
       this.Balance = Balance;                              ❷
       this.AllowedOverdraft = AllowedOverdraft;            ❷
    }

   public AccountState WithStatus(AccountStatus newStatus)  ❸
      => new AccountState
      (
         Status: newStatus,
         Balance: this.Balance,
         Currency: this.Currency,
         AllowedOverdraft: this.AllowedOverdraft
      );

   public AccountState Debit(decimal amount)                ❸
      => Credit(-amount);

   public AccountState Credit(decimal amount)               ❸
      => new AccountState
      (
         Balance: this.Balance + amount,
         Currency: this.Currency,
         Status: this.Status,
         AllowedOverdraft: this.AllowedOverdraft
      );
}
```

❶ 所有属性都是只读的。

❷ 在构造函数中初始化属性

❸ 提供复制方法以创建修改后的副本

除了属性和构造函数之外，`AccountState` 还有一个 `WithStatus` 复制方法，它创建一个新的 `AccountState`，并带有更新的状态。`Debit` 和 `Credit` 也是复制方法，它们创建一个带有更新余额的副本。（这个相当长的类定义替换了列表 13.2 中的记录定义，后者只有七行。）

现在，关于状态转换。记住，我们使用账户历史中的第一个事件来创建一个 `AccountState`，然后使用每个事件来计算事件之后账户的新状态。状态转换的签名是

```
AccountState → Event → AccountState
```

为了实现状态转换，我们根据事件类型进行模式匹配，并相应地更新 `AccountState`：

列表 A.16 使用模式匹配建模状态转换

```
public static class Account
{
   public static AccountState Create(CreatedAccount evt)    ❶
      => new AccountState
         (
            Currency: evt.Currency,
            Status: AccountStatus.Active
         );

   public static AccountState Apply
      (this AccountState account, Event evt)
      => new Pattern                                        ❷
      {
         (DepositedCash e) => account.Credit(e.Amount),
         (DebitedTransfer e) => account.Debit(e.DebitedAmount),
         (FrozeAccount _) => account.WithStatus(AccountStatus.Frozen),
      }
      .Match(evt);
}
```

❶ `CreatedAccount` 是一个特殊情况，因为没有先前的状态。

❷ 根据事件类型调用相关的转换

由于无法依赖语言对模式匹配的支持，此代码有效地使用了 A.2.2 节中展示的模式匹配解决方案。

## A.4 结论

正如你所见，本书中讨论的所有技术都可以用于使用较旧版本的 C# 的遗留项目。当然，如果你能的话，升级到 C# 的最新版本以利用新的语言特性，特别是记录和模式匹配。

* * *

¹ 微软开发了 `System.Collections.Immutable` 库来补充 BCL 中的可变集合，因此它的感觉应该很熟悉。你必须从 NuGet 获取它。

`System.Reflection` 中的实用工具允许你在运行时查看和修改任何字段的值，包括 `private` 和 `readonly` 字段以及自动属性的底层字段。
