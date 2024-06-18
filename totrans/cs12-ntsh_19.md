# Chapter 19\. Dynamic Programming

[Chapter 4](ch04.html#advanced_chash) explained how dynamic binding works in the C# language. In this chapter, we look briefly at the *Dynamic Language Runtime* (DLR) and then explore the following dynamic programming patterns:

*   Dynamic member overload resolution

*   Custom binding (implementing dynamic objects)

*   Dynamic language interoperability

###### Note

In [Chapter 24](ch24.html#native_and_com_interoperabilit), we describe how `dynamic` can improve COM interoperability.

The types in this chapter reside in the `System.Dynamic` namespace, except for `CallSite<>`, which resides in `System.Runtime.CompilerServices`.

# The Dynamic Language Runtime

C# relies on the DLR to perform dynamic binding.

Contrary to its name, the DLR is not a dynamic version of the CLR. Rather, it’s a library that sits atop the CLR—just like any other library such as *System.Xml.dll*. Its primary role is to provide runtime services to *unify* dynamic programming—in both statically and dynamically typed languages. Hence, languages such as C#, Visual Basic, IronPython, and IronRuby all use the same protocol for calling functions dynamically. This allows them to share libraries and call code written in other languages.

The DLR also makes it relatively easy to write new dynamic languages in .NET. Instead of having to emit Intermediate Language (IL), dynamic language authors work at the level of *expression trees* (the same expression trees in `System.Linq.Expressions` that we talked about in [Chapter 8](ch08.html#linq_queries)).

The DLR further ensures that all consumers get the benefit of *call-site caching*, an optimization whereby the DLR prevents unnecessarily repeating the potentially expensive member resolution decisions made during dynamic binding.

# Dynamic Member Overload Resolution

Calling a statically known method with dynamically typed arguments defers member overload resolution from compile time to runtime. This is useful in simplifying certain programming tasks—such as simplifying the *Visitor* design pattern. It’s also useful in working around limitations imposed by C#’s static typing.

## Simplifying the Visitor Pattern

In essence, the *Visitor* pattern allows you to “add” a method to a class hierarchy without altering existing classes. Although useful, this pattern in its static incarnation is subtle and unintuitive compared to most other design patterns. It also requires that visited classes be made “visitor-friendly” by exposing an `Accept` method, which can be impossible if the classes are not under your control.

With dynamic binding, you can achieve the same goal more easily—and without needing to modify existing classes. To illustrate, consider the following class hierarchy:

[PRE0]

Suppose that we want to write a method that programmatically exports a `Person`’s details to an XML `XElement`. The most obvious solution is to write a virtual method called `ToXElement()` in the `Person` class that returns an `XElement` populated with a `Person`’s properties. We would then override it in `Customer` and `Employee` classes such that the `XElement` was also populated with `CreditLimit` and `Salary`. This pattern can be problematic, however, for two reasons:

*   You might not own the `Person`, `Customer`, and `Employee` classes, making it impossible to add methods to them. (And extension methods wouldn’t give polymorphic behavior.)

*   The `Person`, `Customer`, and `Employee` classes might already be quite big. A frequent antipattern is the “God Object,” in which a class such as `Person` attracts so much functionality that it becomes a nightmare to maintain. A good antidote is to avoid adding functions to `Person` that don’t need to access `Person`’s private state. A `ToXElement` method might be an excellent candidate.

With dynamic member overload resolution, we can write the `ToXElement` functionality in a separate class, without resorting to ugly switches based on type:

[PRE1]

The `DynamicVisit` method performs a dynamic dispatch—calling the most specific version of `Visit` as determined at runtime. Notice the line in boldface, in which we call `DynamicVisit` on each person in the `Friends` collection. This ensures that if a friend is a `Customer` or `Employee`, the correct overload is called.

We can demonstrate this class, as follows:

[PRE2]

Here’s the result:

[PRE3]

### Variations

If you plan more than one visitor class, a useful variation is to define an abstract base class for visitors:

[PRE4]

Subclasses then don’t need to define their own `DynamicVisit` method: all they do is override the versions of `Visit` whose behavior they want to specialize. This also has the advantages of centralizing the methods that encompass the `Person` hierarchy and allowing implementers to call base methods more naturally:

[PRE5]

You then can even subclass `ToXElementPersonVisitor` itself.

## Anonymously Calling Members of a Generic Type

The strictness of C#’s static typing is a double-edged sword. On the one hand, it enforces a degree of correctness at compile time. On the other hand, it occasionally makes certain kinds of code difficult or impossible to express, at which point you must resort to reflection. In these situations, dynamic binding is a cleaner and faster alternative to reflection.

An example is when you need to work with an object of type `G<T>` where `T` is unknown. We can illustrate this by defining the following class:

[PRE6]

Suppose that we then write a method as follows:

[PRE7]

This method won’t compile: you can’t invoke members of *unbound* generic types.

Dynamic binding offers two means by which we can work around this. The first is to access the `Value` member dynamically as follows:

[PRE8]

This has the (potential) advantage of working with any object that defines a `Value` field or property. However, there are a couple of problems. First, catching an exception in this manner is somewhat messy and inefficient (and there’s no way to ask the DLR in advance, “Will this operation succeed?”). Second, this approach wouldn’t work if `Foo` were an interface (say, `IFoo<T>`) and either of the following conditions were true:

*   `Value` was implemented explicitly

*   The type that implemented `IFoo<T>` was inaccessible (more on this soon)

A better solution is to write an overloaded helper method called `GetFooValue` and to call it using *dynamic member overload resolution*:

[PRE9]

Notice that we overloaded `GetFooValue` to accept an `object` parameter, which acts as a fallback for any type. At runtime, the C# dynamic binder will pick the best overload when calling `GetFooValue` with a dynamic argument. If the object in question is not based on `Foo<T>`, it will choose the object-parameter overload instead of throwing an exception.

###### Note

An alternative is to write just the first `GetFooValue` overload and then catch the `RuntimeBinderException`. The advantage is that it distinguishes the case of `foo.Value` being null. The disadvantage is that it incurs the performance overhead of throwing and catching an exception.

In [Chapter 18](ch18.html#reflection_and_metadata), we solved the same problem with an interface using reflection—with a lot more effort (see [“Anonymously Calling Members of a Generic Interface”](ch18.html#ch18-anonymously_calling_members_of_a_gener)). The example we used was to design a more powerful version of `ToString()` that could understand objects such as `IEnumerable` and `IGrouping<,>`. Here’s the same example solved more elegantly using dynamic binding:

[PRE10]

Here it is in action:

[PRE11]

Notice that we used dynamic *member overload resolution* to solve this problem. If we instead did the following:

[PRE12]

it would fail because LINQ’s `GroupBy` operator returns a type implementing `IGrouping<,>`, which itself is internal and therefore inaccessible:

[PRE13]

Even though the `Key` property is declared `public`, its containing class caps it at `internal`, making it accessible only via the `IGrouping<,>` interface. And as is explained in [Chapter 4](ch04.html#advanced_chash), there’s no way to instruct the DLR to bind to that interface when invoking the `Value` member dynamically.

# Implementing Dynamic Objects

An object can provide its binding semantics by implementing `IDynamicMetaObjectProvider`—or more easily by subclassing `DynamicObject`, which provides a default implementation of this interface. This is demonstrated briefly in [Chapter 4](ch04.html#advanced_chash) via the following example:

[PRE14]

## DynamicObject

In the preceding example, we overrode `TryInvokeMember`, which allows the consumer to invoke a method on the dynamic object—such as a `Quack` or `Waddle`. `DynamicObject` exposes other virtual methods that enable consumers to use other programming constructs as well. The following correspond to constructs that have representations in C#:

| Method | Programming construct |
| --- | --- |
| `TryInvokeMember` | Method |
| `TryGetMember`, `TrySetMember` | Property or field |
| `TryGetIndex`, `TrySetIndex` | Indexer |
| `TryUnaryOperation` | Unary operator such as `!` |
| `TryBinaryOperation` | Binary operator such as `==` |
| `TryConvert` | Conversion (cast) to another type |
| `TryInvoke` | Invocation on the object itself—e.g., `d("foo")` |

These methods should return `true` if successful. If they return `false`, the DLR will fall back to the language binder, looking for a matching member on the `Dynamic​Ob⁠ject` (subclass) itself. If this fails, a `RuntimeBinderException` is thrown.

We can illustrate `TryGetMember` and `TrySetMember` with a class that lets us dynamically access an attribute in an `XElement` (`System.Xml.Linq`):

[PRE15]

Here’s how to use it:

[PRE16]

The following does a similar thing for `System.Data.IDataRecord`, making it easier to use data readers:

[PRE17]

The following demonstrates `TryBinaryOperation` and `TryInvoke`:

[PRE18]

`DynamicObject` also exposes some virtual methods for the benefit of dynamic languages. In particular, overriding `GetDynamicMemberNames` allows you to return a list of all member names that your dynamic object provides.

###### Note

Another reason to implement `GetDynamicMemberNames` is that Visual Studio’s debugger makes use of this method to display a view of a dynamic object.

## ExpandoObject

Another simple application of `DynamicObject` would be to write a dynamic class that stored and retrieved objects in a dictionary, keyed by string. However, this functionality is already provided via the `ExpandoObject` class:

[PRE19]

`ExpandoObject` implements `IDictionary<string,object>`—so we can continue our example and do this:

[PRE20]

# Interoperating with Dynamic Languages

Although C# supports dynamic binding via the `dynamic` keyword, it doesn’t go as far as allowing you to execute an expression described in a string at runtime:

[PRE21]

This is because the code to translate a string into an expression tree requires a lexical and semantic parser. These features are built into the C# compiler and are not available as a runtime service. At runtime, C# merely provides a *binder*, which instructs the DLR how to interpret an already-built expression tree.

True dynamic languages such as IronPython and IronRuby do allow you to execute an arbitrary string, and this is useful in tasks such as scripting, writing dynamic configuration systems, and implementing dynamic rules engines. So, although you can write most of your application in C#, it can be useful to call out to a dynamic language for such tasks. In addition, you might want to use an API that is written in a dynamic language where no equivalent functionality is available in a .NET library.

###### Note

The Roslyn scripting NuGet package *Microsoft.CodeAnalysis.CSharp.Scripting* provides an API that lets you execute a C# string, although it does so by first compiling your code into a program. The compilation overhead makes it slower than Python interop, unless you intend to execute the same expression repeatedly.

In the following example, we use IronPython to evaluate an expression created at runtime from within C#. You could use the following script to write a calculator.

###### Note

To run this code, add the NuGet packages *Dynamic​LanguageRuntime* (not to be confused with the *System.Dynamic.Runtime* package) and *IronPython* to your application.

[PRE22]

Because we’re passing a string into Python, the expression will be evaluated according to Python’s rules and not C#’s. It also means that we can use Python’s language features, such as lists:

[PRE23]

## Passing State Between C# and a Script

To pass variables from C# to Python, a few more steps are required. The following example illustrates those steps and could be the basis of a rules engine:

[PRE24]

You can also get variables back by calling `GetVariable`:

[PRE25]

Notice that we specified `SourceCodeKind.SingleStatement` in the second example (rather than `Expression`) to inform the engine that we want to execute a statement.

Types are automatically marshaled between the .NET and Python worlds. You can even access members of .NET objects from the scripting side:

[PRE26]