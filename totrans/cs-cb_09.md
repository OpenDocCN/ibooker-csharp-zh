# Chapter 9\. Examining Recent C# Language Highlights

The C# programming language is continually evolving. Earlier chapters discussed subjects from C# 1 through C# 8\. The exception is pattern matching in [Chapter 8](ch08.xhtml#matching_with_patterns), where some of the patterns were introduced in C# 9\. This chapter focuses primarily on C# 9, the exception being the subject of [Recipe 9.9](#slicing_arrays), which is a C# 8 feature.

A central concept in this chapter is immutability—the ability to create and operate on types that don’t change. Immutability is important for safe multithreading as well as for the cognitive relief of knowing that code you’ve passed a type to won’t change (mutate) the contents of the object.

The example scenario is working with addresses, such as a mailing address or a shipping address. In many contexts, an address is a value, meaning that it doesn’t have an identity. The address exists as a set of data (value) associated with an entity such as a customer or company. On the other hand, an entity does have an identity, normally modeled as an ID field in a database. Because we’re treating it as a value, address becomes a useful candidate for immutability because we don’t want the value to change once it’s set.

One interesting feature of C# 9 is called *module initialization*. Think about how we use constructors to initialize types or `Main` to initialize an application. Module initialization lets you write initialization code at the scope of an assembly, and you’ll see how that works.

Another C# 9 theme is code simplification. You’ll see a section on how to write code without namespaces or classes, eliminating the ceremony around starting an application in a `Main` method. Another simplification is in instantiating objects, with a new feature to infer type contextually. Let’s start with simplifying application startup.

# 9.1 Simplifying Application Startup

## Problem

You need to eliminate as much code as possible for your application entry point.

## Solution

This is a top-level program:

[PRE0]

Here’s the code that the C# compiler generates:

[PRE1]

## Discussion

A lot of the code we write is boilerplate—standard syntax that we copy over and over again. In the case of a console app with a `Main` method, you have a namespace that generally matches the project name, a class named `Program`, and a `Main` method. While you’re free to remove the namespace and rename the class, people rarely do. Developers have recognized this for years, and in C# 9, we no longer need the boilerplate code.

The solution shows the new top-level statements feature, where the code doesn’t have a namespace, class, or `Main` method. It’s the minimal amount of code required to start the app. The example code requests address details and prints out the results and works exactly as written.

###### Tip

If you’re teaching someone how to program in C#, top-level statements can make the task easier. You don’t need to explain methods, because you’re not writing a `Main` method. You don’t need to explain a class, which is an object with members and more. You can leave out the namespace discussion and all the nuance about naming and organization. Rather than waxing lightly across (or ignoring) all of these complex details, you can discuss them later, when the student is ready.

Top-level statements serve in place of the `Main` method. The solution shows the code that the compiler generates. It has the `CompilerGenerated` attribute, class, and `Main` method. The naming conventions match the typical boilerplate code that Visual Studio, the .NET CLI, and other IDEs produce for console apps.

Interestingly, you can only put top-level statements in a single file. If you try putting them in multiple files, you’ll encounter the following compiler error:

[PRE2]

# 9.2 Reducing Instantiation Syntax

## Problem

Object instantiation is too redundant and verbose.

## Solution

We’re going to instantiate this class:

[PRE3]

Here are different ways to instantiate that class:

[PRE4]

## Discussion

Originally, C# had a single way to instantiate a variable: declaring the type, variable name, new operator, type, and parenthesized constructor parameter list. You can see this in the solution via the `addressOld` field and `addressLocalOld` variable.

Under the C# 3 paradigm, we needed a strongly typed variable (the `var` keyword) to hold anonymous types, especially for LINQ queries. A `var` variable required assignment and became a strongly typed assigned type. Some people saw that `var` looked similar to the JavaScript `var` and were uncomfortable using it. However, as stated earlier, the C# `var` variable is strongly typed, meaning that you can’t declare the variable to be of a different type.

Besides LINQ, a convenient use case for `var` emerged in type instantiation. Developers recognized the ability to eliminate redundancy in defining variables by using `var`. You can see how this works in the solution for the `addressLocalVar` variable.

Because of the popularity of `var` to reduce code in object instantiation, developers looked toward fields for the same experience. However, you can’t use `var` with fields, as demonstrated in the `address` field in the solution, which is commented out.

C# 9 fixes the redundancy concerns with a feature called *target-typed new*. Instead of using `var`, target-typed new declares the type, identifier, and `new` keyword with a parameter list. The `addressNew` field and `addressLocalNew` variable show how this works. Now you can instantiate fields without the redundancy of specifying the same type twice in the same statement.

Target-typed new is shortcut syntax for the same type instantiation we’ve done forever. That means you can still use object initializers and constructor overloads, shown in `addressObjectInit` and `addressCtorInit`, respectively.

Now that we have target-typed new, there’s an argument to be made for preferring that over `var`. The first reason is the cognitive hesitation of developers who eschew `var` because it’s spelled the same way as the JavaScript `var`—even though we know the C# `var` is strongly typed. The other is that since we can use target-typed new for both variables and fields, we have syntactic consistency in how we instantiate types. Some developers will view mixing `var` and target-typed new in the same code as distracting or messy.

###### Note

Even if you don’t use `var` for type instantiation, it’s still useful. When doing LINQ queries, you can reshape data with anonymous type projections in the same method, and that requires a `var` for the results.

Finally, introducing target-typed new into the language doesn’t necessarily imply a preference for direct object instantiation. As explained in [Recipe 1.2](ch01.xhtml#removing_explicit_dependencies), IoC is a powerful mechanism for decoupling code, promoting separation of concerns, and making code more testable.

## See Also

[Recipe 1.2, “Removing Explicit Dependencies”](ch01.xhtml#removing_explicit_dependencies)

# 9.3 Initializing Immutable State

## Problem

You need immutable properties that are populated only during instantiation.

## Solution

Here’s a class with immutable state:

[PRE5]

Here are a couple of ways to instantiate the immutable class:

[PRE6]

## Discussion

Immutability, the ability to create and operate on types that don’t change, is increasingly an important feature for the quality and correctness of code. Imagine the scenario where you pass an object to a method and get the same type of object back. Assuming you don’t own the code for that method, how do you know what that method did to the object you gave it? Short of decompilation or trusting documentation, you don’t know. However, if the object is immutable, it can’t change, and you know that the method didn’t change anything.

Another use case for immutability is in multithreading. The reality of deadlocks and race conditions have plagued developers for a long time. In the deadlock scenario, separate threads wait on each other to release a resource that the other needs for changing. In the race condition scenario, you don’t know which thread will modify an object first, resulting in inconsistent object state. In each case, immutability simplifies the scenario because neither thread can change an existing object—they must rely on their own copy. Multithreading is such a complex topic that it couldn’t be fairly discussed here in depth, but the point is that immutability is part of the solution.

In the solution, the `Address` class is immutable. You can instantiate it with the data you need, but its contents can’t change after that. Notice that the properties have a getter but not a setter. Instead, they have initters. The initters let you instantiate the object but not change it thereafter.

The `Main` method shows how this works. The `addressObjectInit` variable instantiates normally, but setting any of its properties, including `City`, won’t compile. The `addressCtorInit` variable shows a similar situation.

If you have an existing class, making properties init-only can be useful. However, if you’re building new types with C# 9, you can also define records, as discussed in the next recipe.

## See Also

[Recipe 9.4, “Creating Immutable Types”](#creating_immutable_types)

# 9.4 Creating Immutable Types

## Problem

You need an immutable reference type but don’t want to write all the plumbing code.

## Solution

Here is a C# record:

[PRE7]

This code shows how to use that record:

[PRE8]

And this is the output:

[PRE9]

Here’s the synthesized code that the C# compiler generates:

[PRE10]

## Discussion

[Recipe 9.3](#initializing_immutable_state) discusses immutability and its benefits and how to create an immutable class. This works great if you have existing types and want to migrate them to being immutable. However, for new code and types that you want to make immutable, consider using a record.

Records were introduced in C# 9 as a way to create simple immutable types. The solution shows how to do this with the `Address` record. Declare the type as record, give it a type name, list the properties, and terminate with a semicolon. Although this might look similar to defining a constructor or method, the parameters define the properties of this new type, and they follow the common convention of Pascal case naming.

The solution shows how to instantiate the `Address` record. Notice how the `address​C⁠torInit` variable doesn’t allow changing its state, including the `Street` property.

An interesting fact about records is that they are reference types with value semantics. The solution shows that comparing `addressClassic` and `addressCtorInit` with `==` results in `true`. This demonstrates value equality because the properties of both records are identical. However, notice the `ReferenceEquals` comparison. It’s `false` because records are reference types and each refers to separate objects in memory.

Although declaring the `Address` record was short and quick, this is a huge simplification of the real code that the C# compiler generates. The solution shows the synthesized code with many members. The type is a class and it has a constructor overload with parameters for populating each property.

The key to value equality is the implementation of `IEquatable<Address>`. The class has both a weakly typed and strongly typed `Equals` method. [Recipe 2.5](ch02.xhtml#checking_for_type_equality) showed how to implement `IEquatable<T>`, which shares some similarity to this implementation. One difference is the type being stored in the `EqualityContract` property. Since the C# generated class uses `EqualityContract` in both `Equals` and `GetHashCode`, it makes sense to eliminate the redundancy.

The `ToString` and `PrintMembers` implementations might be familiar if you’ve read [Recipe 3.6](ch03.xhtml#customizing_class_string_representation). The implementations are nearly identical. Notice that the `PrintMembers` is `virtual`, and that allows derived types to add their values to the output.

Finally, the synthesized class includes a `Clone` method to get a shallow copy, a deconstructor for representing values as a tuple, and a copy constructor for making a copy of another record. What would also be convenient is a way to get a copy of the current object but with modifications, which is discussed next.

## See Also

[Recipe 2.5, “Checking for Type Equality”](ch02.xhtml#checking_for_type_equality)

[Recipe 3.6, “Customizing Class String Representation”](ch03.xhtml#customizing_class_string_representation)

[Recipe 9.3, “Initializing Immutable State”](#initializing_immutable_state)

# 9.5 Simplifying Immutable Type Assignments

## Problem

You need to change a property of an object but don’t want to mutate the original object.

## Solution

We’ll use this record:

[PRE11]

And this code shows how to make a copy of that record:

[PRE12]

## Discussion

As discussed in [Recipe 9.4](#creating_immutable_types), records have a normal constructor, a copy constructor, and a clone method to create new records of the same type. There’s one scenario that these options don’t easily cover: getting a type with modifications. That is, what if you wanted everything on the object to be the same, except for one or two properties? This section shows a simple way to do that.

Since records are immutable, you can’t modify any of their properties. You could always instantiate a new record and supply all of the properties, but that is wasteful when only a single property changes, especially if it’s an object with a lot of properties.

The solution in C# 9 uses a `with` expression. You have an existing record, add a `with` expression, and only change the properties that need to change. This gives you a new instance of the record type with the changes you want.

The solution does this on the `addressPre` variable. The `with` expression uses a block of property assignments to specify the properties that need to change. This example changes a single property. You can also set multiple properties in the same way you do with object initializers, via a comma-separated list.

## See Also

[Recipe 9.4, “Creating Immutable Types”](#creating_immutable_types)

# 9.6 Designing for Record Reuse

## Problem

You need to avoid duplicating functionality.

## Solution

Here’s an abstract base record:

[PRE13]

These two records derive from that abstract base record:

[PRE14]

Here’s how you can work with those records:

[PRE15]

## Discussion

One of the ways to achieve reuse in C# is via inheritance. Records support inheritance in the same manner as classes.

The solution has a record named `AddressBase`. As its name suggests, `AddressBase` is intended to be a base record. `AddressBase` is also abstract, preventing direct instantiation. It has properties common to all derived types.

`MailingAddress` and `ShippingAddress` derive from `AddressBase`, using the inheritance syntax similar to classes. The difference is that the inherited record declaration includes a parameter list, indicating which parameters from the derived record match the base record.

`MailingAddress` specializes `AddressBase` with two new properties: `Email` and `PreferEmail`. `ShippingAddress` specializes `AddressBase` with an extra `Delivery​In⁠structions` property.

The definition of `ShippingAddress` is different because it explicitly defines members, rather than using default record syntax. It has a constructor, just like a C# class, passing parameters to the base, `AddressBase`. The `ShippingAddress` constructor implementation has validation code that throws an exception to protect against invalid initialization. In this case, it enforces the logic that a P.O. box is not a place you can deliver merchandise to. The constructor also initializes the `DeliveryInstructions` property. This demonstrates that while default record syntax simplifies the code, you still have the ability to customize records all you need.

When customizing records, you can add any member that a class could have. Additionally, you can override default implementations such as equality, `ToString` output, or constructors. Also, customizing as done with `ShippingAddress` doesn’t prevent the C# compiler from generating the default record implementation.

# 9.7 Returning Different Method Override Types

## Problem

You’re overriding a base class method but need to return a more specific type.

## Solution

Here are the records we want to work with:

[PRE16]

This base class has a method, returning a base record:

[PRE17]

These classes have methods returning derived records:

[PRE18]

This code shows how to use those derived classes that return derived records:

[PRE19]

## Discussion

It used to be that method overrides were required to return the same type as the base class virtual method return type. The problem was that derived classes often needed to return specialized information from their overrides. The alternatives were ugly:

1.  Create a new nonpolymorphic method.

2.  Return the base type.

3.  Return a type derived from the base return type and expect the caller to convert.

None of these choices are optimal, and fortunately, C# 9 offers a solution through covariant return types.

The solution has two sets of type hierarchies: one for return types and one for method polymorphism. `AddressBase` and its two derived records, `MailingAddress` and `ShippingAddress`, represent the return types. The `DeliveryBase` class, with its derived classes, `Communications` and `Shipping`, have a `GetAddress` method that operates polymorphically.

###### Note

Notice how the implementation of `GetAddress` returns target-typed new instances. The compiler infers type by context, which is the return type in these examples. You can learn more about target-typed new in [Recipe 9.2](#reducing_instantiation_syntax).

Prior to C# 9, the `GetAddress` in `Communications` and `Shipping` would be forced to return `AddressBase`. However, looking at the solution implementation, the `Get​Ad⁠d​ress` in `Communications` and `Shipping` return `MailingAddress` and `Shipping​A⁠dd​ress`, respectively.

## See Also

[Recipe 9.2, “Reducing Instantiation Syntax”](#reducing_instantiation_syntax)

# 9.8 Implementing Iterators as Extension Methods

## Problem

You need an iterator on a third-party type for which you don’t have the code.

## Solution

Here’s a definition to a type in a third-party library that we don’t have access to:

[PRE20]

This class has an enumerator extension method for that type:

[PRE21]

Here’s how to use that enumerator:

[PRE22]

## Discussion

Sometimes, it’s convenient to add an iterator to an object. Doing so lets you separate the concerns of dissecting, transforming, and returning object data from the consuming code that wants to concentrate on solving the business problem. If you own the code of an object and want to loop over its contents, add an iterator. However, if the object is from a third party and you don’t have access to the code, you used to be forced to add extraneous logic to business code. In C# 9, you now have the ability to add a `GetEnumerator` method as an extension method.

In the solution, the `Address` record is the object we want to iterate over. More specifically, we want to iterate on the members of the `Address` record, similar to the way you can iterate over properties of a JavaScript object.

The `AddressExtensions` method has an extension method named `GetEnumerator` that takes an `Address` parameter and returns an `IEnumerable<T>`. The `this` parameter works just like for any other extension method, specifying the type and instance to operate on. The pattern for the iterator is that the method must be named `GetEnumerator`, and it must return an `IEnumerator<T>`. The type, `T`, can be any type of your choosing—whatever you need. In this example, `T` is `string`. This means that you need to convert each property to a string, which isn’t a problem in `Address` because all properties are already a string. Consistent with C# iterator implementation, the `AddressExtensions` `GetEnumerator` method uses `yield return` for each value and `yield break` to indicate the end of iteration.

After getting a list of `Address`, the `Main` method has a nested `foreach` loop where the inner `foreach` iterates on an instance of `Address`. Because of the extension method, the `foreach` works on `address` the same as it does with arrays and collections—no extra syntax.

# 9.9 Slicing Arrays

## Problem

You want to use ranges to page through data.

## Solution

We’re going to use this record:

[PRE23]

This method populates an array of records:

[PRE24]

This method does paging by slicing an array of records:

[PRE25]

This code iterates through pages of the record:

[PRE26]

## Discussion

Since C# 8, it has been much easier to slice arrays. Specify the beginning index, concatenate two dots, and specify the last index.

This solution looks at slicing from the perspective of paging. Some of the applications we use page by number of rows or number of columns in a row. This solution pages `Address` instances by three per page.

There are two overloads of the `GetAddresses` method. The first, parameterless version, generates unique addresses.

The second `GetAddresses` overload is an iterator that takes an `int` parameter, `perPage`, instructing the method to return that many instances of `Address` at one time. After getting a list of `Address` instances, the `for` loop controls iterating through the list. The `for` initializer sets `i` to the first `Address` and `j` to one more than the last `Address`. Since `i` is the start of the range, the `for` condition ensures that `i` doesn’t exceed the size of the array. The `for` incrementer adjusts `i` and `j` to the next set of `Address` instances (that is, the next page).

`GetAddresses(int` `perPage)` is an iterator, as indicated by the `IEnumerable<Address[]>` return type and the fact that it uses `yield return` on results. While [Recipe 9.8](#implementing_iterators_as_extension_methods) showed how to add an iterator as an extension method, this example assumes you have access to the code and adding an iterator directly to the code is preferable.

The `Main` method shows how to use the `GetAddresses(int perPage)` iterator, returning the page that was sliced out of the original `Address[]`.

## See Also

[Recipe 9.8, “Implementing Iterators as Extension Methods”](#implementing_iterators_as_extension_methods)

# 9.10 Initializing Entire Modules

## Problem

You need IoC to work on a class library without relying on the caller to do it right.

## Solution

Here’s a repository that returns records:

[PRE27]

This module initializer configures an IoC container:

[PRE28]

This service relies on the IoC container:

[PRE29]

This `Main` method is a client that consumes the service:

[PRE30]

## Discussion

C# 9 added a feature called module initialization. Essentially, this allows you to add any kind of initialization code for an assembly. This initialization code runs before any other code in the assembly.

At first glance, this might sound strange because console, Windows Forms, and WPF apps have `Main` methods. Even all versions of ASP.NET and Web API have startup code. I’m not saying that there isn’t a use case for those technologies, though it seems like a rare event for the average professional developer.

###### Note

Another initialization technique that has existed since C# 1 is the use of static constructors. A static constructor only runs whenever code accesses class members, either via type or instance. So, a static constructor isn’t a valid substitute for module initialization because it’s possible that calling code will never access a member of that class, and the static constructor will never run.

That said, there is a set of use cases that involve class libraries. The problem has always been that you don’t know how consuming code will use your library. You can document and set a contract that says the user must call some method or start the library a certain way, and that’s as close as you get to guaranteeing any type of control over initialization.

Module initialization changes that because now you have more control over how to initialize library code, regardless of what the user does. The solution solves the problem of ensuring that IoC gets initialized before any code runs.

The `AddressService` class offers two ways to instantiate an instance of itself, via IoC or with the `Create` method. The user has a choice of whether to use IoC or not. The benefit is that IoC becomes an option for the library developer too for making it easy to write unit tests and write maintainable code.

[Recipe 1.2](ch01.xhtml#removing_explicit_dependencies) explains how IoC works, using `Microsoft.Extensions.Dependency​Injec⁠tion`, and this solution uses the same library and technique. The main difference is where the IoC container gets configured.

The `Initializer` class has a method named `InitAddressUtilities`. The `Module​Ini⁠tializer` attribute indicates that `InitAddressUtilities` is the module initialization code for this class library. The `InitAddressUtilities` method will run before any other code in the class library.

###### Note

In the early days of .NET, a module was a way to group code into a single file, for modularization. You could combine modules into an assembly, where the assembly was defined as being one or more modules with an additional manifest. The manifest contains meta-data for the CLR, the details of which are voluminous and unimportant for the current focus.

Using modules was largely a theoretical capability, as most code is compiled as a single module in an assembly. This is the default behavior for the C# compiler and Visual Studio. In fact, you had to go out of your way to create a module that was potentially useful. While it’s interesting that the `ModuleInitializer` has the word “module” in it, the practical reality is that it applies toward initialization at the level of the assembly.

Because the `InitAddressUtilities` method has already run, the `Create` method in `AddressService` can rely on `Initializer.Container` having a valid container reference for resolving an `AddressService` instance.

## See Also

[Recipe 1.2, “Removing Explicit Dependencies”](ch01.xhtml#removing_explicit_dependencies)