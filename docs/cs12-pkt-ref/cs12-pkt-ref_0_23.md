# 第二十三章：事件

当您使用委托时，通常会出现两个新兴角色：*广播者*和*订阅者*。*广播者*是包含委托字段的类型。广播者通过调用委托来决定何时进行广播。*订阅者*是方法目标接收者。订阅者通过在广播者的委托上调用`+=`和`-=`来决定何时开始和停止监听。订阅者不知道或干扰其他订阅者。

事件是一种语言特性，正式化了这种模式。`event`是一种只暴露委托功能子集的构造，用于广播者/订阅者模型。事件的主要目的是*防止订阅者互相干扰*。

声明事件的最简单方式是在委托成员前面加上 `event` 关键字：

```cs
public class Broadcaster
{
  public event ProgressReporter Progress;
}
```

`Broadcaster` 类型内部的代码可以完全访问 `Progress` 并将其视为委托。在 `Broadcaster` 外部的代码只能对 `Progress` 事件执行 `+=` 和 `-=` 操作。

在下面的示例中，`Stock` 类在每次 `Stock` 的 `Price` 改变时触发其 `PriceChanged` 事件：

```cs
public delegate void PriceChangedHandler
 (decimal oldPrice, decimal newPrice);

public class Stock
{
  string symbol; decimal price;

  public Stock (string symbol) => this.symbol = symbol;

 public event PriceChangedHandler PriceChanged;

  public decimal Price
  {
    get => price;
    set
    {
      if (price == value) return;
      // Fire event if invocation list isn't empty:
      if (PriceChanged != null)
 PriceChanged (price, value);
      price = value;
    }
  }
}
```

如果我们从示例中删除 `event` 关键字，使 `PriceChanged` 变为普通委托字段，我们的示例将给出相同的结果。但是，`Stock` 在订阅者之间可能会更不健壮，因为订阅者可以执行以下操作之一来干扰彼此：

+   通过重新分配 `PriceChanged`（而不是使用 `+=` 运算符）替换其他订阅者

+   清除所有订阅者（通过将 `PriceChanged` 设置为 `null`）

+   通过调用委托向其他订阅者广播

事件可以是虚拟的、被重写的、抽象的或密封的。它们也可以是静态的。

## 标准事件模式

几乎所有在 .NET 库中定义事件的情况下，它们的定义都遵循一种标准模式，旨在提供对库和用户代码的一致性。以下是使用此模式重构的前述示例：

```cs
public class PriceChangedEventArgs : EventArgs
{
  public readonly decimal LastPrice, NewPrice;

  public PriceChangedEventArgs (decimal lastPrice,
                                decimal newPrice)
  {
    LastPrice = lastPrice; NewPrice = newPrice;
  }
}

public class Stock
{
  string symbol; decimal price;

  public Stock (string symbol) => this.symbol = symbol;

  public event EventHandler<PriceChangedEventArgs>
               PriceChanged;

 protected virtual void OnPriceChanged
 (PriceChangedEventArgs e) =>
    // Shortcut for invoking PriceChanged if not null:
 PriceChanged?.Invoke (this, e);

  public decimal Price
  {
    get { return price; }
    set
    {
      if (price == value) return;
 OnPriceChanged (new PriceChangedEventArgs (price,
 value));
      price = value;
    }  
  }
}
```

标准事件模式的核心是 `System.EventArgs`，这是一个预定义的 .NET 类，没有其他成员（除了静态的 `Empty` 字段）。 `EventArgs` 是传递事件信息的基类。在这个示例中，我们子类化 `EventArgs` 来传递价格变化前后的旧值和新值。

泛型 `System.EventHandler` 委托也是 .NET 的一部分，定义如下：

```cs
public delegate void EventHandler<TEventArgs>
  (object source, TEventArgs e)
```

###### 注意

在 C# 2.0 之前（当泛型添加到语言中时），解决方案是为每个 `EventArgs` 类型编写自定义事件处理委托：

```cs
delegate void PriceChangedHandler
  (object sender,
   PriceChangedEventArgs e);
```

基于历史原因，.NET 库中的大多数事件使用了这种方式定义的委托。

一个名为 `On`*-*`*event-name*` 的受保护虚拟方法集中了事件的触发。这允许子类触发事件（通常是可取的），还允许子类在事件触发前后插入代码。

这是我们如何使用我们的 `Stock` 类：

```cs
Stock stock = new Stock ("THPW");
stock.Price = 27.10M;

stock.PriceChanged += stock_PriceChanged;
stock.Price = 31.59M;

static void stock_PriceChanged
  (object sender, PriceChangedEventArgs e)
{
  if ((e.NewPrice - e.LastPrice) / e.LastPrice > 0.1M)
    Console.WriteLine ("Alert, 10% price increase!");
}
```

对于不携带附加信息的事件，.NET 还提供了一个非泛型的 `EventHandler` 委托。我们可以通过重写我们的 `Stock` 类来演示这一点，使得 `PriceChanged` 事件在价格改变后触发。这意味着事件无需传递额外的信息：

```cs
public class Stock
{
  string symbol; decimal price;

  public Stock (string symbol) => this.symbol = symbol;

  public event EventHandler PriceChanged;

 protected virtual void OnPriceChanged (EventArgs e) =>
 PriceChanged?.Invoke (this, e);

  public decimal Price
  {
    get => price;
    set
    {
      if (price == value) return;
      price = value;
      OnPriceChanged (EventArgs.Empty);      
    }  
  }
}
```

注意，我们还使用了 `EventArgs.Empty` 属性——这样可以节省实例化 `EventArgs` 的开销。

## 事件访问器

事件的 *访问器* 是其 `+=` 和 `-=` 函数的实现。默认情况下，访问器由编译器隐式实现。考虑以下事件声明：

```cs
public event EventHandler PriceChanged;
```

编译器将其转换为以下内容：

+   一个私有委托字段

+   一对公共事件访问器函数，其实现将`+=`和`-=`操作转发到私有委托字段

你可以通过定义*显式*事件访问器来接管这个过程。这里是我们先前示例中`PriceChanged`事件的手动实现：

```cs
EventHandler priceChanged;   // Private delegate
public event EventHandler PriceChanged
{
 add    { priceChanged += value; }
 remove { priceChanged -= value; }
}
```

这个示例在功能上与 C#的默认访问器实现相同（除了 C#还确保在更新委托时线程安全）。通过定义事件访问器，我们指示 C#不生成默认字段和访问器逻辑。

使用显式事件访问器时，可以对底层委托的存储和访问应用更复杂的策略。当事件访问器仅仅是向另一个广播事件的类中继时，或者显式实现声明事件的接口时，这非常有用：

```cs
public interface IFoo { event EventHandler Ev; }
class Foo : IFoo
{
  EventHandler ev;
  event EventHandler IFoo.Ev
  {
    add { ev += value; } remove { ev -= value; }
  }
}
```

