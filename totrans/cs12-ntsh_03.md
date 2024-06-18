# Chapter 3\. Creating Types in C#

In this chapter, we delve into types and type members.

# Classes

A class is the most common kind of reference type. The simplest possible class declaration is as follows:

[PRE0]

A more complex class optionally has the following:

| Preceding the keyword `class` | *Attributes* and *class modifiers*. The non-nested class modifiers are `public`, `internal`, `abstract`, `sealed`, `static`, `unsafe`, and `partial`. |
| Following `YourClassName` | *Generic type parameters* and *constraints*, a *base class*, and *interfaces*. |
| Within the braces | *Class members* (these are *methods*, *properties*, *indexers*, *events*, *fields*, *constructors*, *overloaded operators*, *nested types*, and a *finalizer*). |

This chapter covers all of these constructs except attributes, operator functions, and the `unsafe` keyword, which are covered in [Chapter 4](ch04.html#advanced_chash). The following sections enumerate each of the class members.

## Fields

A *field* is a variable that is a member of a class or struct; for example:

[PRE1]

Fields allow the following modifiers:

| Static modifier | `static` |
| Access modifiers | `public internal private protected` |
| Inheritance modifier | `new` |
| Unsafe code modifier | `unsafe` |
| Read-only modifier | `readonly` |
| Threading modifier | `volatile` |

There are two popular naming conventions for private fields: camel-cased (e.g., `firstName`), and camel-cased with an underscore (`_firstName`). The latter convention lets you instantly distinguish private fields from parameters and local variables.

### The readonly modifier

The `readonly` modifier prevents a field from being modified after construction. A read-only field can be assigned only in its declaration or within the enclosing type’s constructor.

### Field initialization

Field initialization is optional. An uninitialized field has a default value (`0`, `'\0'`, `null`, `false`). Field initializers run before constructors:

[PRE2]

A field initializer can contain expressions and call methods:

[PRE3]

### Declaring multiple fields together

For convenience, you can declare multiple fields of the same type in a comma-separated list. This is a convenient way for all the fields to share the same attributes and field modifiers:

[PRE4]

## Constants

A *constant* is evaluated statically at compile time, and the compiler literally substitutes its value whenever used (rather like a macro in C++). A constant can be `bool`, `char`, `string`, any of the built-in numeric types, or an enum type.

A constant is declared with the `const` keyword and must be initialized with a value. For example:

[PRE5]

A constant can serve a similar role to a `static readonly` field, but it is much more restrictive—both in the types you can use and in field initialization semantics. A constant also differs from a `static readonly` field in that the evaluation of the constant occurs at compile time; thus

[PRE6]

is compiled to

[PRE7]

It makes sense for `PI` to be a constant because its value is predetermined at compile time. In contrast, a `static readonly` field’s value can potentially differ each time the program is run:

[PRE8]

###### Note

A `static readonly` field is also advantageous when exposing to other assemblies a value that might change in a later version. For instance, suppose that assembly `X` exposes a constant as follows:

[PRE9]

If assembly `Y` references `X` and uses this constant, the value `2.3` will be baked into assembly `Y` when compiled. This means that if `X` is later recompiled with the constant set to 2.4, `Y` will still use the old value of 2.3 *until* `Y` *is recompiled*. A `static readonly` field avoids this problem.

Another way of looking at this is that any value that might change in the future is not constant by definition; thus, it should not be represented as one.

Constants can also be declared local to a method:

[PRE10]

Nonlocal constants allow the following modifiers:

| Access modifiers | `public internal private protected` |
| Inheritance modifier | `new` |

## Methods

A method performs an action in a series of statements. A method can receive *input* data from the caller by specifying *parameters* and *output* data back to the caller by specifying a *return type*. A method can specify a `void` return type, indicating that it doesn’t return any value to its caller. A method can also output data back to the caller via `ref`/`out` parameters.

A method’s *signature* must be unique within the type. A method’s signature comprises its name and parameter types in order (but not the parameter *names*, nor the return type).

Methods allow the following modifiers:

| Static modifier | `static` |
| Access modifiers | `public internal private protected` |
| Inheritance modifiers | `new virtual abstract override sealed` |
| Partial method modifier | `partial` |
| Unmanaged code modifiers | `unsafe extern` |
| Asynchronous code modifier | `async` |

### Expression-bodied methods

A method that comprises a single expression, such as

[PRE11]

can be written more tersely as an *expression-bodied method*. A fat arrow replaces the braces and `return` keyword:

[PRE12]

Expression-bodied functions can also have a void return type:

[PRE13]

### Local methods

You can define a method within another method:

[PRE14]

The local method (`Cube`, in this case) is visible only to the enclosing method (`WriteCubes`). This simplifies the containing type and instantly signals to anyone looking at the code that `Cube` is used nowhere else. Another benefit of local methods is that they can access the local variables and parameters of the enclosing method. This has a number of consequences, which we describe in detail in [“Capturing Outer Variables”](ch04.html#capturing_outer_variables).

Local methods can appear within other function kinds, such as property accessors, constructors, and so on. You can even put local methods inside other local methods, and inside lambda expressions that use a statement block ([Chapter 4](ch04.html#advanced_chash)). Local methods can be iterators ([Chapter 4](ch04.html#advanced_chash)) or asynchronous ([Chapter 14](ch14.html#concurrency_and_asynchron)).

### Static local methods

Adding the `static` modifier to a local method (from C# 8) prevents it from seeing the local variables and parameters of the enclosing method. This helps to reduce coupling and prevents the local method from accidentally referring to variables in the containing method.

### Local methods and top-level statements

Any methods that you declare in top-level statements are treated as local methods. This means that (unless marked as `static`) they can access the variables in the top-level statements:

[PRE15]

### Overloading methods

###### Warning

Local methods cannot be overloaded. This means that methods declared in top-level statements (which are treated as local methods) cannot be overloaded.

A type can *overload* methods (define multiple methods with the same name) as long as the signatures are different. For example, the following methods can all coexist in the same type:

[PRE16]

However, the following pairs of methods cannot coexist in the same type, because the return type and the `params` modifier are not part of a method’s signature:

[PRE17]

Whether a parameter is pass-by-value or pass-by-reference is also part of the signature. For example, `Foo(int)` can coexist with either `Foo(ref int)` or `Foo(out int)`. However, `Foo(ref int)` and `Foo(out int)` cannot coexist:

[PRE18]

## Instance Constructors

Constructors run initialization code on a class or struct. A constructor is defined like a method, except that the method name and return type are reduced to the name of the enclosing type:

[PRE19]

Instance constructors allow the following modifiers:

| Access modifiers | `public internal private protected` |
| Unmanaged code modifiers | `unsafe extern` |

Single-statement constructors can also be written as expression-bodied members:

[PRE20]

###### Note

If a parameter name (or any variable name, for that matter) conflicts with a field name, you can disambiguate by prefixing the field with a `this` reference:

[PRE21]

### Overloading constructors

A class or struct may overload constructors. To avoid code duplication, one constructor can call another, using the `this` keyword:

[PRE22]

When one constructor calls another, the *called constructor* executes first.

You can pass an *expression* into another constructor, as follows:

[PRE23]

The expression can access static members of the class but not instance members. (This is enforced because the object has not been initialized by the constructor at this stage, so any methods that you call on it are likely to fail.)

###### Note

This particular example could be better implemented with a single constructor that has `year` as an optional parameter:

[PRE24]

We will offer yet another solution shortly, in [“Object Initializers”](#object_initializers).

### Implicit parameterless constructors

For classes, the C# compiler automatically generates a parameterless public constructor if and only if you do not define any constructors. However, as soon as you define at least one constructor, the parameterless constructor is no longer automatically generated.

### Constructor and field initialization order

We previously saw that fields can be initialized with default values in their declaration:

[PRE25]

Field initializations occur *before* the constructor is executed, and in the declaration order of the fields.

### Nonpublic constructors

Constructors need not be public. A common reason to have a nonpublic constructor is to control instance creation via a static method call. The static method could be used to return an object from a pool rather than creating a new object, or to return various subclasses based on input arguments:

[PRE26]

## Deconstructors

A deconstructor (also called a *deconstructing method*) acts as an approximate opposite to a constructor: whereas a constructor typically takes a set of values (as parameters) and assigns them to fields, a deconstructor does the reverse and assigns fields back to a set of variables.

A deconstruction method must be called `Deconstruct` and must have one or more `out` parameters, such as in the following class:

[PRE27]

The following special syntax calls the deconstructor:

[PRE28]

The second line is the deconstructing call. It creates two local variables and then calls the `Deconstruct` method. Our deconstructing call is equivalent to the following:

[PRE29]

Or:

[PRE30]

Deconstructing calls allow implicit typing, so we could shorten our call to this:

[PRE31]

Or simply this:

[PRE32]

###### Note

You can use C#’s discard symbol (`_`) if you’re uninterested in one or more variables:

[PRE33]

This better indicates your intention than declaring a variable that you never use.

If the variables into which you’re deconstructing are already defined, omit the types altogether:

[PRE34]

This is called a *deconstructing assignment*. You can use a deconstructing assignment to simplify your class’s constructor:

[PRE35]

You can offer the caller a range of deconstruction options by overloading the `Deconstruct` method.

###### Note

The `Deconstruct` method can be an extension method (see [“Extension Methods”](ch04.html#extension_methods)). This is a useful trick if you want to deconstruct types that you did not author.

From C# 10, you can mix and match existing and new variables when deconstructing:

[PRE36]

## Object Initializers

To simplify object initialization, any accessible fields or properties of an object can be set via an *object initializer* directly after construction. For example, consider the following class:

[PRE37]

Using object initializers, you can instantiate `Bunny` objects as follows:

[PRE38]

The code to construct `b1` and `b2` is precisely equivalent to the following:

[PRE39]

The temporary variables are to ensure that if an exception is thrown during initialization, you can’t end up with a half-initialized object.

## The this Reference

The `this` reference refers to the instance itself. In the following example, the `Marry` method uses `this` to set the `partner`’s `mate` field:

[PRE40]

The `this` reference also disambiguates a local variable or parameter from a field; for example:

[PRE41]

The `this` reference is valid only within nonstatic members of a class or struct.

## Properties

Properties look like fields from the outside, but internally they contain logic, like methods do. For example, you can’t tell by looking at the following code whether `CurrentPrice` is a field or a property:

[PRE42]

A property is declared like a field but with a `get`/`set` block added. Here’s how to implement `CurrentPrice` as a property:

[PRE43]

`get` and `set` denote property *accessors*. The `get` accessor runs when the property is read. It must return a value of the property’s type. The `set` accessor runs when the property is assigned. It has an implicit parameter named `value` of the property’s type that you typically assign to a private field (in this case, `currentPrice`).

Although properties are accessed in the same way as fields, they differ in that they give the implementer complete control over getting and setting its value. This control enables the implementer to choose whatever internal representation is needed without exposing the internal details to the user of the property. In this example, the `set` method could throw an exception if `value` was outside a valid range of values.

###### Note

Throughout this book, we use public fields extensively to keep the examples free of distraction. In a real application, you would typically favor public properties over public fields in order to promote encapsulation.

Properties allow the following modifiers:

| Static modifier | `static` |
| Access modifiers | `public internal private protected` |
| Inheritance modifiers | `new virtual abstract override sealed` |
| Unmanaged code modifiers | `unsafe extern` |

### Read-only and calculated properties

A property is read-only if it specifies only a `get` accessor, and it is write-only if it specifies only a `set` accessor. Write-only properties are rarely used.

A property typically has a dedicated backing field to store the underlying data. However, a property can also be computed from other data:

[PRE44]

### Expression-bodied properties

You can declare a read-only property, such as the one in the preceding example, more tersely as an *expression-bodied property*. A fat arrow replaces all the braces and the `get` and `return` keywords:

[PRE45]

With a little extra syntax, `set` accessors can also be expression-bodied:

[PRE46]

### Automatic properties

The most common implementation for a property is a getter and/or setter that simply reads and writes to a private field of the same type as the property. An *automatic property* declaration instructs the compiler to provide this implementation. We can improve the first example in this section by declaring `CurrentPrice` as an automatic property:

[PRE47]

The compiler automatically generates a private backing field of a compiler-generated name that cannot be referred to. The `set` accessor can be marked `private` or `protected` if you want to expose the property as read-only to other types. Automatic properties were introduced in C# 3.0.

### Property initializers

You can add a *property initializer* to automatic properties, just as with fields:

[PRE48]

This gives `CurrentPrice` an initial value of `123`. Properties with an initializer can be read-only:

[PRE49]

Just as with read-only fields, read-only automatic properties can also be assigned in the type’s constructor. This is useful in creating *immutable* (read-only) types.

### get and set accessibility

The `get` and `set` accessors can have different access levels. The typical use case for this is to have a `public` property with an `internal` or `private` access modifier on the setter:

[PRE50]

Notice that you declare the property itself with the more permissive access level (`public`, in this case) and add the modifier to the accessor you want to be *less* accessible.

### Init-only setters

From C# 9, you can declare a property accessor with `init` instead of `set`:

[PRE51]

These *init-only* properties act like read-only properties, except that they can also be set via an object initializer:

[PRE52]

After that, the property cannot be altered:

[PRE53]

Init-only properties cannot even be set from inside their class, except via their property initializer, the constructor, or another init-only accessor.

The alternative to init-only properties is to have read-only properties that you populate via a constructor:

[PRE54]

Should the class be part of a public library, this approach makes versioning difficult, in that adding an optional parameter to the constructor at a later date breaks binary compatibility with consumers (whereas adding a new init-only property breaks nothing).

###### Note

Init-only properties have another significant advantage, which is that they allow for nondestructive mutation when used in conjunction with records (see [“Records”](ch04.html#records-id00087)).

Just as with ordinary `set` accessors, init-only accessors can provide an implementation:

[PRE55]

Notice that the `_pitch` field is read-only: init-only setters are permitted to modify `readonly` fields in their own class. (Without this feature, `_pitch` would need to be writable, and the class would fail at being internally immutable.)

###### Warning

Changing a property’s accessor from `init` to `set` (or vice versa) is a *binary breaking change*: anyone that references your assembly will need to recompile their assembly.

This should not be an issue when creating wholly immutable types, in that your type will never require properties with a (writable) `set` accessor.

### CLR property implementation

C# property accessors internally compile to methods called `get_*XXX*` and `set_*XXX*`:

[PRE56]

An `init` accessor is processed like a `set` accessor, but with an extra flag encoded into the `set` accessor’s “modreq” metadata (see [“Init-only properties”](ch18.html#init_only_properties)).

Simple nonvirtual property accessors are *inlined* by the Just-In-Time (JIT) compiler, eliminating any performance difference between accessing a property and a field. Inlining is an optimization in which a method call is replaced with the body of that method.

## Indexers

Indexers provide a natural syntax for accessing elements in a class or struct that encapsulate a list or dictionary of values. Indexers are similar to properties but are accessed via an index argument rather than a property name. The `string` class has an indexer that lets you access each of its `char` values via an `int` index:

[PRE57]

The syntax for using indexers is like that for using arrays, except that the index argument(s) can be of any type(s).

Indexers have the same modifiers as properties (see [“Properties”](#properties)) and can be called null-conditionally by inserting a question mark before the square bracket (see [“Null Operators”](ch02.html#null_operators)):

[PRE58]

### Implementing an indexer

To write an indexer, define a property called `this`, specifying the arguments in square brackets:

[PRE59]

Here’s how we could use this indexer:

[PRE60]

A type can declare multiple indexers, each with parameters of different types. An indexer can also take more than one parameter:

[PRE61]

If you omit the `set` accessor, an indexer becomes read-only, and you can use expression-bodied syntax to shorten its definition:

[PRE62]

### CLR indexer implementation

Indexers internally compile to methods called `get_Item` and `set_Item`, as follows:

[PRE63]

### Using indices and ranges with indexers

You can support indices and ranges (see [“Indices and Ranges”](ch02.html#indices_and_ranges-id00073)) in your own classes by defining an indexer with a parameter type of `Index` or `Range`. We could extend our previous example, by adding the following indexers to the `Sentence` class:

[PRE64]

This then enables the following:

[PRE65]

## Primary Constructors (C# 12)

From C# 12, you can include a parameter list directly after a class (or struct) declaration:

[PRE66]

This instructs the compiler to automatically build a *primary constructor* using the *primary constructor parameters* (`firstName` and `lastName`), so that we can instantiate our class as follows:

[PRE67]

Primary constructors are useful for prototyping and other simple scenarios. The alternative would be to define fields and write a constructor explicitly:

[PRE68]

The constructor that C# builds is called primary because any additional constructors that you choose to (explicitly) write must invoke it:

[PRE69]

This ensures that primary constructor parameters are *always populated*.

###### Note

C# also provides *records*, which we cover in [“Records”](ch04.html#records-id00087). Records also support primary constructors; however, the compiler takes an extra step with records and generates (by default) a public init-only property for each primary constructor parameter. Should this behavior be desirable, consider using records instead.

Primary constructors are best suited to simple scenarios due to the following limitations:

*   You cannot add extra initialization code to a primary constructor.

*   Although it’s easy to expose a primary constructor parameter as a public property, you cannot easily incorporate validation logic unless the property is read-only.

Primary constructors displace the default parameterless constructor that C# would otherwise generate.

### Primary constructor semantics

To understand how primary constructors work, consider how an ordinary constructor behaves:

[PRE70]

When the code inside this constructor finishes executing, parameters `firstName` and `lastName` disappear out of scope and cannot be subsequently accessed. In contrast, a primary constructor’s parameters do *not* disappear out of scope and can be subsequently accessed from anywhere within the class, for the life of the object.

###### Note

Primary constructor parameters are special C# constructs, not *fields*, although the compiler does end up generating hidden fields behind the scenes to store their values if necessary.

### Primary constructors and field/property initializers

The accessibility of primary constructor parameters extends to field and property initializers. In the following example, we use field and property initializers to assign `firstName` to a public field, and `lastName` to a public property:

[PRE71]

### Masking primary constructor parameters

Fields (or properties) can reuse primary constructor parameter names:

[PRE72]

In this scenario, the field or property takes precedence, thereby masking the primary constructor parameter, *except* on the righthand side of field and property initializers (shown in boldface).

###### Note

Just like ordinary parameters, primary constructor parameters are writable. Masking them with a same-named `readonly` field (as in our example) effectively protects them from subsequent modification.

### Validating primary constructor parameters

Sometimes it’s useful to perform computation in field initializers:

[PRE73]

In the next example, we save an uppercase version of `lastName` to a field of the same name (masking the original value):

[PRE74]

In [“throw expressions”](ch04.html#throw_expressions-id00096), we describe how to throw exceptions when encountering scenarios such as invalid data. Here’s a preview to illustrate how this can be used with primary constructors to validate `lastName` upon construction, ensuring that it cannot be null:

[PRE75]

(Remember that code within a field or property initializer executes when the object is constructed—not when the field or property is accessed.) In the next example, we expose a primary constructor parameter as a read/write property:

[PRE76]

Adding validation to this example is not straightforward in that you must validate in two places: in a (manually implemented) property `set` accessor and in the property initializer. (The same problem exists if the property is defined as init-only.) At this point, it’s easier to abandon the shortcut of primary constructors and define a constructor and backing fields explicitly.

## Static Constructors

A static constructor executes once per *type* rather than once per *instance*. A type can define only one static constructor, and it must be parameterless and have the same name as the type:

[PRE77]

The runtime automatically invokes a static constructor just prior to the type being used. Two things trigger this:

*   Instantiating the type

*   Accessing a static member in the type

The only modifiers allowed by static constructors are `unsafe` and `extern`.

###### Warning

If a static constructor throws an unhandled exception ([Chapter 4](ch04.html#advanced_chash)), that type becomes *unusable* for the life of the application.

###### Note

From C# 9, you can also define *module initializers*, which execute once per assembly (when the assembly is first loaded). To define a module initializer, write a static void method and then apply the `[ModuleInitializer]` attribute to that method:

[PRE78]

### Static constructors and field initialization order

Static field initializers run just *before* the static constructor is called. If a type has no static constructor, static field initializers will execute just prior to the type being used—or *anytime earlier* at the whim of the runtime.

Static field initializers run in the order in which the fields are declared. The following example illustrates this. `X` is initialized to `0`, and `Y` is initialized to `3`:

[PRE79]

If we swap the two field initializers around, both fields are initialized to 3\. The next example prints 0 followed by 3 because the field initializer that instantiates a `Foo` executes before `X` is initialized to `3`:

[PRE80]

If we swap the two lines in boldface, the example prints 3 followed by 3.

## Static Classes

A class marked `static` cannot be instantiated or subclassed, and must be composed solely of static members. The `System.Console` and `System.Math` classes are good examples of static classes.

## Finalizers

Finalizers are class-only methods that execute before the garbage collector reclaims the memory for an unreferenced object. The syntax for a finalizer is the name of the class prefixed with the `~` symbol:

[PRE81]

This is actually C# syntax for overriding `Object`’s `Finalize` method, and the compiler expands it into the following method declaration:

[PRE82]

We discuss garbage collection and finalizers fully in [Chapter 12](ch12.html#disposal_and_garbage_collection).

You can write single-statement finalizers using expression-bodied syntax:

[PRE83]

## Partial Types and Methods

Partial types allow a type definition to be split—typically across multiple files. A common scenario is for a partial class to be autogenerated from some other source (such as a Visual Studio template or designer), and for that class to be augmented with additional hand-authored methods:

[PRE84]

Each participant must have the `partial` declaration; the following is illegal:

[PRE85]

Participants cannot have conflicting members. A constructor with the same parameters, for instance, cannot be repeated. Partial types are resolved entirely by the compiler, which means that each participant must be available at compile time and must reside in the same assembly.

You can specify a base class on one or more partial class declarations, as long as the base class, if specified, is the same. In addition, each participant can independently specify interfaces to implement. We cover base classes and interfaces in [“Inheritance”](#inheritance) and [“Interfaces”](#interfaces).

The compiler makes no guarantees with regard to field initialization order between partial type declarations.

### Partial methods

A partial type can contain *partial methods*. These let an autogenerated partial type provide customizable hooks for manual authoring; for example:

[PRE86]

A partial method consists of two parts: a *definition* and an *implementation*. The definition is typically written by a code generator, and the implementation is typically manually authored. If an implementation is not provided, the definition of the partial method is compiled away (as is the code that calls it). This allows autogenerated code to be liberal in providing hooks without having to worry about bloat. Partial methods must be `void` and are implicitly `private`. They cannot include `out` parameters.

### Extended partial methods

*Extended partial methods* (from C# 9) are designed for the reverse code generation scenario, where a programmer defines hooks that a code generator implements. An example of where this might occur is with *source generators*, a Roslyn feature that lets you feed the compiler an assembly that automatically generates portions of your code.

A partial method declaration is *extended* if it begins with an accessibility modifier:

[PRE87]

The presence of the accessibility modifier doesn’t just affect accessibility: it tells the compiler to treat the declaration differently.

Extended partial methods *must* have implementations; they do not melt away if unimplemented. In this example, both `M1` and `M2` must have implementations because they each specify accessibility modifiers (`public` and `private`).

Because they cannot melt away, extended partial methods can return any type and can include `out` parameters:

[PRE88]

## The nameof operator

The `nameof` operator returns the name of any symbol (type, member, variable, and so on) as a string:

[PRE89]

Its advantage over simply specifying a string is that of static type checking. Tools such as Visual Studio can understand the symbol reference, so if you rename the symbol in question, all of its references will be renamed, too.

To specify the name of a type member such as a field or property, include the type as well. This works with both static and instance members:

[PRE90]

This evaluates to `Length`. To return `StringBuilder.Length`, you would do this:

[PRE91]

# Inheritance

A class can *inherit* from another class to extend or customize the original class. Inheriting from a class lets you reuse the functionality in that class instead of building it from scratch. A class can inherit from only a single class but can itself be inherited by many classes, thus forming a class hierarchy. In this example, we begin by defining a class called `Asset`:

[PRE92]

Next, we define classes called `Stock` and `House`, which will inherit from `Asset`. `Stock` and `House` get everything an `Asset` has, plus any additional members that they define:

[PRE93]

Here’s how we can use these classes:

[PRE94]

The *derived classes*, `Stock` and `House`, inherit the `Name` field from the *base class*, `Asset`.

###### Note

A derived class is also called a *subclass*.

A base class is also called a *superclass*.

## Polymorphism

References are *polymorphic*. This means a variable of type *x* can refer to an object that subclasses *x*. For instance, consider the following method:

[PRE95]

This method can display both a `Stock` and a `House` because they are both `Asset`s:

[PRE96]

Polymorphism works on the basis that subclasses (`Stock` and `House`) have all the features of their base class (`Asset`). The converse, however, is not true. If `Display` was modified to accept a `House`, you could not pass in an `Asset`:

[PRE97]

## Casting and Reference Conversions

An object reference can be:

*   Implicitly *upcast* to a base class reference

*   Explicitly *downcast* to a subclass reference

Upcasting and downcasting between compatible reference types performs *reference conversions*: a new reference is (logically) created that points to the *same* object. An upcast always succeeds; a downcast succeeds only if the object is suitably typed.

### Upcasting

An upcast operation creates a base class reference from a subclass reference:

[PRE98]

After the upcast, variable `a` still references the same `Stock` object as variable `msft`. The object being referenced is not itself altered or converted:

[PRE99]

Although `a` and `msft` refer to the identical object, `a` has a more restrictive view on that object:

[PRE100]

The last line generates a compile-time error because the variable `a` is of type `Asset`, even though it refers to an object of type `Stock`. To get to its `SharesOwned` field, you must *downcast* the `Asset` to a `Stock`.

### Downcasting

A downcast operation creates a subclass reference from a base class reference:

[PRE101]

As with an upcast, only references are affected—not the underlying object. A downcast requires an explicit cast because it can potentially fail at runtime:

[PRE102]

If a downcast fails, an `InvalidCastException` is thrown. This is an example of *runtime type checking* (we elaborate on this concept in [“Static and Runtime Type Checking”](#static_and_runtime_type_checking)).

### The as operator

The `as` operator performs a downcast that evaluates to `null` (rather than throwing an exception) if the downcast fails:

[PRE103]

This is useful when you’re going to subsequently test whether the result is `null`:

[PRE104]

###### Note

Without such a test, a cast is advantageous, because if it fails, a more helpful exception is thrown. We can illustrate by comparing the following two lines of code:

[PRE105]

If `a` is not a `Stock`, the first line throws an `InvalidCastException`, which is an accurate description of what went wrong. The second line throws a `NullReferenceException`, which is ambiguous. Was `a` not a `Stock`, or was `a` null?

Another way of looking at it is that with the cast operator, you’re saying to the compiler: “I’m *certain* of a value’s type; if I’m wrong, there’s a bug in my code, so throw an exception!” Whereas with the `as` operator, you’re uncertain of its type and want to branch according to the outcome at runtime.

The `as` operator cannot perform *custom conversions* (see [“Operator Overloading”](ch04.html#operator_overloading)), and it cannot do numeric conversions:

[PRE106]

###### Note

The `as` and cast operators will also perform upcasts, although this is not terribly useful because an implicit conversion will do the job.

### The is operator

The `is` operator tests whether a variable matches a *pattern*. C# supports several kinds of patterns, the most important being a *type pattern*, where a type name follows the `is` keyword.

In this context, the `is` operator tests whether a reference conversion would succeed—in other words, whether an object derives from a specified class (or implements an interface). It is often used to test before downcasting:

[PRE107]

The `is` operator also evaluates to true if an *unboxing conversion* would succeed (see [“The object Type”](#the_object_type)). However, it does not consider custom or numeric conversions.

###### Note

The `is` operator works with many other patterns introduced in recent versions of C#. For a full discussion, see [“Patterns”](ch04.html#patterns).

### Introducing a pattern variable

You can introduce a variable while using the `is` operator:

[PRE108]

This is equivalent to the following:

[PRE109]

The variable that you introduce is available for “immediate” consumption, so the following is legal:

[PRE110]

And it remains in scope outside the `is` expression, allowing this:

[PRE111]

## Virtual Function Members

A function marked as `virtual` can be *overridden* by subclasses wanting to provide a specialized implementation. Methods, properties, indexers, and events can all be declared `virtual`:

[PRE112]

(`Liability => 0` is a shortcut for `{ get { return 0; } }`. For more details on this syntax, see [“Expression-bodied properties”](#expression_bodied_properties).)

A subclass overrides a virtual method by applying the `override` modifier:

[PRE113]

By default, the `Liability` of an `Asset` is `0`. A `Stock` does not need to specialize this behavior. However, the `House` specializes the `Liability` property to return the value of the `Mortgage`:

[PRE114]

The signatures, return types, and accessibility of the virtual and overridden methods must be identical. An overridden method can call its base class implementation via the `base` keyword (we cover this in [“The base Keyword”](#the_base_keyword)).

###### Warning

Calling virtual methods from a constructor is potentially dangerous because authors of subclasses are unlikely to know, when overriding the method, that they are working with a partially initialized object. In other words, the overriding method might end up accessing methods or properties that rely on fields not yet initialized by the constructor.

### Covariant return types

From C# 9, you can override a method (or property `get` accessor) such that it returns a *more derived* (subclassed) type. For example:

[PRE115]

This is permitted because it does not break the contract that `Clone` must return an `Asset`: it returns a `House`, which *is* an `Asset` (and more).

Prior to C# 9, you had to override methods with the identical return type:

[PRE116]

This still does the job, because the overridden `Clone` method instantiates a `House` rather than an `Asset`. However, to treat the returned object as a `House`, you must then perform a downcast:

[PRE117]

## Abstract Classes and Abstract Members

A class declared as *abstract* can never be instantiated. Instead, only its concrete *subclasses* can be instantiated.

Abstract classes are able to define *abstract members*. Abstract members are like virtual members except that they don’t provide a default implementation. That implementation must be provided by the subclass unless that subclass is also declared abstract:

[PRE118]

## Hiding Inherited Members

A base class and a subclass can define identical members. For example:

[PRE119]

The `Counter` field in class `B` is said to *hide* the `Counter` field in class `A`. Usually, this happens by accident, when a member is added to the base type *after* an identical member was added to the subtype. For this reason, the compiler generates a warning and then resolves the ambiguity as follows:

*   References to `A` (at compile time) bind to `A.Counter`.

*   References to `B` (at compile time) bind to `B.Counter`.

Occasionally, you want to hide a member deliberately, in which case you can apply the `new` modifier to the member in the subclass. The `new` modifier *does nothing more than suppress the compiler warning that would otherwise result*:

[PRE120]

The `new` modifier communicates your intent to the compiler—and other programmers—that the duplicate member is not an accident.

###### Note

C# overloads the `new` keyword to have independent meanings in different contexts. Specifically, the `new` *operator* is different from the `new` *member modifier*.

### new versus override

Consider the following class hierarchy:

[PRE121]

The differences in behavior between `Overrider` and `Hider` are demonstrated in the following code:

[PRE122]

## Sealing Functions and Classes

An overridden function member can *seal* its implementation with the `sealed` keyword to prevent it from being overridden by further subclasses. In our earlier virtual function member example, we could have sealed `House`’s implementation of `Liability`, preventing a class that derives from `House` from overriding `Liability`, as follows:

[PRE123]

You can also apply the `sealed` modifier to the class itself, to prevent subclassing. Sealing a class is more common than sealing a function member.

Although you can seal a function member against overriding, you can’t seal a member against being *hidden*.

## The base Keyword

The `base` keyword is similar to the `this` keyword. It serves two essential purposes:

*   Accessing an overridden function member from the subclass

*   Calling a base-class constructor (see the next section)

In this example, `House` uses the `base` keyword to access `Asset`’s implementation of `Liability`:

[PRE124]

With the `base` keyword, we access `Asset`’s `Liability` property *nonvirtually*. This means that we will always access `Asset`’s version of this property—regardless of the instance’s actual runtime type.

The same approach works if `Liability` is *hidden* rather than *overridden*. (You can also access hidden members by casting to the base class before invoking the function.)

## Constructors and Inheritance

A subclass must declare its own constructors. The base class’s constructors are *accessible* to the derived class but are never automatically *inherited*. For example, if we define `Baseclass` and `Subclass` as follows:

[PRE125]

the following is illegal:

[PRE126]

`Subclass` must hence “redefine” any constructors it wants to expose. In doing so, however, it can call any of the base class’s constructors via the `base` keyword:

[PRE127]

The `base` keyword works rather like the `this` keyword except that it calls a constructor in the base class.

Base-class constructors always execute first; this ensures that *base* initialization occurs before *specialized* initialization.

### Implicit calling of the parameterless base-class constructor

If a constructor in a subclass omits the `base` keyword, the base type’s *parameterless* constructor is implicitly called:

[PRE128]

If the base class has no accessible parameterless constructor, subclasses are forced to use the `base` keyword in their constructors. This means that a base class with (only) a multiparameter constructor burdens subclasses with the obligation to call it:

[PRE129]

### Required members (C# 11)

The requirement for subclasses to invoke a constructor in the base class can become burdensome in large class hierarchies if there are many constructors with many parameters. Sometimes, the best solution is to avoid constructors altogether and rely solely on object initializers to set fields or properties during construction. To help with this, you can mark a field or property as `required` (from C# 11):

[PRE130]

A required member *must* be populated via an object initializer when constructed:

[PRE131]

Should you wish to also write a constructor, you can apply the `[SetsRequired​Mem⁠bers]` attribute to bypass the required member restriction for that constructor:

[PRE132]

Consumers can now benefit from the convenience of that constructor without any trade-off:

[PRE133]

Notice that we also defined a parameterless constructor (for use with the object initializer). Its presence also ensures that subclasses remain under no burden to reproduce any constructor. In the following example, the `House` class chooses not to implement a convenience constructor:

[PRE134]

### Constructor and field initialization order

When an object is instantiated, initialization takes place in the following order:

1.  From subclass to base class:

    1.  Fields are initialized.

    2.  Arguments to base-class constructor calls are evaluated.

2.  From base class to subclass:

    1.  Constructor bodies execute.

For example:

[PRE135]

### Inheritance with primary constructors

Classes with primary constructors can subclass with the following syntax:

[PRE136]

The call to `Baseclass(x)` is equivalent to calling `base(x)` in the following example:

[PRE137]

## Overloading and Resolution

Inheritance has an interesting impact on method overloading. Consider the following two overloads:

[PRE138]

When an overload is called, the most specific type has precedence:

[PRE139]

The particular overload to call is determined statically (at compile time) rather than at runtime. The following code calls `Foo(Asset)`, even though the runtime type of `a` is `House`:

[PRE140]

###### Note

If you cast `Asset` to `dynamic` ([Chapter 4](ch04.html#advanced_chash)), the decision as to which overload to call is deferred until runtime and is then based on the object’s actual type:

[PRE141]

# The object Type

`object` (`System.Object`) is the ultimate base class for all types. Any type can be upcast to `object`.

To illustrate how this is useful, consider a general-purpose *stack*. A stack is a data structure based on the principle of *LIFO*—“last in, first out.” A stack has two operations: *push* an object on the stack and *pop* an object off the stack. Here is a simple implementation that can hold up to 10 objects:

[PRE142]

Because `Stack` works with the object type, we can `Push` and `Pop` instances of *any type* to and from the `Stack`:

[PRE143]

`object` is a reference type, by virtue of being a class. Despite this, value types, such as `int`, can also be cast to and from `object`, and so be added to our stack. This feature of C# is called *type unification* and is demonstrated here:

[PRE144]

When you cast between a value type and `object`, the CLR must perform some special work to bridge the difference in semantics between value and reference types. This process is called *boxing* and *unboxing*.

###### Note

In [“Generics”](#generics), we describe how to improve our `Stack` class to better handle stacks with same-typed elements.

## Boxing and Unboxing

Boxing is the act of converting a value-type instance to a reference-type instance. The reference type can be either the `object` class or an interface (which we visit later in the chapter).^([1](ch03.html#ch01fn5)) In this example, we box an `int` into an object:

[PRE145]

Unboxing reverses the operation by casting the object back to the original value type:

[PRE146]

Unboxing requires an explicit cast. The runtime checks that the stated value type matches the actual object type, and throws an `InvalidCastException` if the check fails. For instance, the following throws an exception because `long` does not exactly match `int`:

[PRE147]

The following succeeds, however:

[PRE148]

As does this:

[PRE149]

In the last example, `(double)` performs an *unboxing* and then `(int)` performs a *numeric conversion*.

###### Note

Boxing conversions are crucial in providing a unified type system. The system is not perfect, however: we’ll see in [“Generics”](#generics) that variance with arrays and generics supports only *reference conversions* and not *boxing conversions*:

[PRE150]

### Copying semantics of boxing and unboxing

Boxing *copies* the value-type instance into the new object, and unboxing *copies* the contents of the object back into a value-type instance. In the following example, changing the value of `i` doesn’t change its previously boxed copy:

[PRE151]

## Static and Runtime Type Checking

C# programs are type-checked both statically (at compile time) and at runtime (by the CLR).

Static type checking enables the compiler to verify the correctness of your program without running it. The following code will fail because the compiler enforces static typing:

[PRE152]

Runtime type checking is performed by the CLR when you downcast via a reference conversion or unboxing:

[PRE153]

Runtime type checking is possible because each object on the heap internally stores a little type token. You can retrieve this token by calling the `GetType` method of `object`.

## The GetType Method and typeof Operator

All types in C# are represented at runtime with an instance of `System.Type`. There are two basic ways to get a `System.Type` object:

*   Call `GetType` on the instance

*   Use the `typeof` operator on a type name

`GetType` is evaluated at runtime; `typeof` is evaluated statically at compile time (when generic type parameters are involved, it’s resolved by the JIT compiler).

`System.Type` has properties for such things as the type’s name, assembly, base type, and so on:

[PRE154]

`System.Type` also has methods that act as a gateway to the runtime’s reflection model, described in [Chapter 18](ch18.html#reflection_and_metadata).

## The ToString Method

The `ToString` method returns the default textual representation of a type instance. This method is overridden by all built-in types. Here is an example of using the `int` type’s `ToString` method:

[PRE155]

You can override the `ToString` method on custom types as follows:

[PRE156]

If you don’t override `ToString`, the method returns the type name.

###### Note

When you call an *overridden* `object` member such as `ToString` directly on a value type, boxing doesn’t occur. Boxing then occurs only if you cast:

[PRE157]

## Object Member Listing

Here are all the members of `object`:

[PRE158]

We describe the `Equals`, `ReferenceEquals`, and `GetHashCode` methods in [“Equality Comparison”](ch06.html#equality_comparison-id00067).

# Structs

A *struct* is similar to a class, with the following key differences:

*   A struct is a value type, whereas a class is a reference type.

*   A struct does not support inheritance (other than implicitly deriving from `object`, or more precisely, `System.ValueType`).

A struct can have all of the members that a class can, except for a finalizer. And because it cannot be subclassed, members cannot be marked as virtual, abstract, or protected.

###### Warning

Prior to C# 10, structs were further prohibited from defining fields initializers and parameterless constructors. Although this prohibition has now been relaxed—primarily for the benefit of record structs (see [“Records”](ch04.html#records-id00087))—it’s worth thinking carefully before defining these constructs, as they can result in confusing behavior that we’ll describe in [“Struct Construction Semantics”](#struct_construction_semantics).

A struct is appropriate when value-type semantics are desirable. Good examples of structs are numeric types, where it is more natural for assignment to copy a value rather than a reference. Because a struct is a value type, each instance does not require instantiation of an object on the heap; this results in useful savings when creating many instances of a type. For instance, creating an array of value type elements requires only a single heap allocation.

Because structs are value types, an instance cannot be null. The default value for a struct is an empty instance, with all fields empty (set to their default values).

## Struct Construction Semantics

###### Note

Prior to C# 11, every field in a struct had to be explicitly assigned in the constructor (or field initializer). This restriction has now been relaxed.

### The default constructor

In addition to any constructors that you define, a struct always has an implicit parameterless constructor that performs a bitwise-zeroing of its fields (setting them to their default values):

[PRE159]

Even when you define a parameterless constructor of your own, the implicit parameterless constructor still exists and can be accessed via the `default` keyword:

[PRE160]

In this example, we initialized `x` to 1 via a field initializer, and we initialized `y` to 1 via the parameterless constructor. And yet with the `default` keyword, we were still able to create a `Point` that bypassed both initializations. The default constructor can be accessed other ways, too, as the following example illustrates:

[PRE161]

###### Note

Having what amounts to two parameterless constructors can be a source of confusion, and is arguably a good reason to avoid defining field initializers and explicit parameterless constructors in structs.

A good strategy with structs is to design them such that their `default` value is a valid state, thereby making initialization redundant. For example, rather than initializing a property as follows:

[PRE162]

consider the following:

[PRE163]

## Read-Only Structs and Functions

You can apply the `readonly` modifier to a struct to enforce that all fields are `readonly`; this aids in declaring intent as well as affording the compiler more optimization freedom:

[PRE164]

If you need to apply `readonly` at a more granular level, you can apply the `readonly` modifier (from C# 8) to a struct’s *functions*. This ensures that if the function attempts to modify any field, a compile-time error is generated:

[PRE165]

If a `readonly` function calls a non-`readonly` function, the compiler generates a warning (and defensively copies the struct to avoid the possibility of a mutation).

## Ref Structs

###### Note

Ref structs were introduced in C# 7.2 as a niche feature primarily for the benefit of the `Span<T>` and `ReadOnlySpan<T>` structs that we describe in [Chapter 23](ch23.html#spanless_thantgreater_than_and-id00089) (and the highly optimized `Utf8JsonReader` that we describe in [Chapter 11](ch11.html#other_xml_and_json_technologies)). These structs help with a micro-optimization technique that aims to reduce memory allocations.

Unlike reference types, whose instances always live on the heap, value types live *in-place* (wherever the variable was declared). If a value type appears as a parameter or local variable, it will reside on the stack:

[PRE166]

But if a value type appears as a field in a class, it will reside on the heap:

[PRE167]

Similarly, arrays of structs live on the heap, and boxing a struct sends it to the heap.

Adding the `ref` modifier to a struct’s declaration ensures that it can only ever reside on the stack. Attempting to use a *ref struct* in such a way that it could reside on the heap generates a compile-time error:

[PRE168]

Ref structs were introduced mainly for the benefit of the `Span<T>` and `ReadOnlySpan<T>` structs. Because `Span<T>` and `ReadOnlySpan<T>` instances can exist only on the stack, it’s possible for them to safely wrap stack-allocated memory.

Ref structs cannot partake in any C# feature that directly or indirectly introduces the possibility of existing on the heap. This includes a number of advanced C# features that we describe in [Chapter 4](ch04.html#advanced_chash), namely, lambda expressions, iterators, and asynchronous functions (because, behind the scenes, these features all create hidden classes with fields). Also, ref structs cannot appear inside non-ref structs, and they cannot implement interfaces (because this could result in boxing).

# Access Modifiers

To promote encapsulation, a type or type member can limit its *accessibility* to other types and other assemblies by adding an *access modifier* to the declaration:

`public`

Fully accessible. This is the implicit accessibility for members of an enum or interface.

`internal`

Accessible only within the containing assembly or friend assemblies. This is the default accessibility for non-nested types.

`private`

Accessible only within the containing type. This is the default accessibility for members of a class or struct.

`protected`

Accessible only within the containing type or subclasses.

`protected internal`

The *union* of `protected` and `internal` accessibility. A member that is `protected internal` is accessible in two ways.

`private protected`

The *intersection* of `protected` and `internal` accessibility. A member that is `private protected` is accessible only within the containing type, or from subclasses *that reside in the same assembly* (making it *less* accessible than `protected` or `internal` alone).

`file` (from C# 11)

Accessible only from within the same file. Intended for use by *source generators* (see [“Extended partial methods”](#extended_partial_methods)). This modifier can be applied only to type declarations.

## Examples

`Class2` is accessible from outside its assembly; `Class1` is not:

[PRE169]

`ClassB` exposes field `x` to other types in the same assembly; `ClassA` does not:

[PRE170]

Functions within `Subclass` can call `Bar` but not `Foo`:

[PRE171]

## Friend Assemblies

You can expose `internal` members to other *friend* assemblies by adding the `System.Runtime.CompilerServices.InternalsVisibleTo` assembly attribute, specifying the name of the friend assembly as follows:

[PRE172]

If the friend assembly has a strong name (see [Chapter 17](ch17.html#assemblies)), you must specify its *full* 160-byte public key:

[PRE173]

You can extract the full public key from a strongly named assembly with a LINQ query (we explain LINQ in detail in [Chapter 8](ch08.html#linq_queries)):

[PRE174]

###### Note

The companion sample in LINQPad invites you to browse to an assembly and then copies the assembly’s full public key to the clipboard.

## Accessibility Capping

A type caps the accessibility of its declared members. The most common example of capping is when you have an `internal` type with `public` members. For example, consider this:

[PRE175]

`C`’s (default) `internal` accessibility caps `Foo`’s accessibility, effectively making `Foo internal`. A common reason `Foo` would be marked `public` is to make for easier refactoring should `C` later be changed to `public`.

## Restrictions on Access Modifiers

When overriding a base class function, accessibility must be identical on the overridden function; for example:

[PRE176]

(An exception is when overriding a `protected internal` method in another assembly, in which case the override must simply be `protected`.)

The compiler prevents any inconsistent use of access modifiers. For example, a subclass itself can be less accessible than a base class but not more:

[PRE177]

# Interfaces

An interface is similar to a class, but only *specifies behavior* and does not hold state (data). Consequently:

*   An interface can define only functions and not fields.

*   Interface members are *implicitly abstract*. (There are exceptions to this rule that we will describe in [“Default Interface Members”](#default_interface_members-id00061) and [“Static Interface Members”](#static_interface_members).)

*   A class (or struct) can implement multiple interfaces. In contrast, a class can inherit from only a single class, and a struct cannot inherit at all (aside from deriving from `System.ValueType`).

An interface declaration is like a class declaration, but it (normally) provides no implementation for its members because its members are implicitly abstract. These members will be implemented by the classes and structs that implement the interface. An interface can contain only functions, that is, methods, properties, events, and indexers (which noncoincidentally are precisely the members of a class that can be abstract).

Here is the definition of the `IEnumerator` interface, defined in `System.Collections`:

[PRE178]

Interface members are always implicitly public and cannot declare an access modifier. Implementing an interface means providing a `public` implementation for all of its members:

[PRE179]

You can implicitly cast an object to any interface that it implements:

[PRE180]

###### Note

Even though `Countdown` is an internal class, its members that implement `IEnumerator` can be called publicly by casting an instance of `Countdown` to `IEnumerator`. For instance, if a public type in the same assembly defined a method as follows:

[PRE181]

a caller from another assembly could do this:

[PRE182]

If `IEnumerator` were itself defined as `internal`, this wouldn’t be possible.

## Extending an Interface

Interfaces can derive from other interfaces; for instance:

[PRE183]

`IRedoable` “inherits” all the members of `IUndoable`. In other words, types that implement `IRedoable` must also implement the members of `IUndoable`.

## Explicit Interface Implementation

Implementing multiple interfaces can sometimes result in a collision between member signatures. You can resolve such collisions by *explicitly implementing* an interface member. Consider the following example:

[PRE184]

Because `I1` and `I2` have conflicting `Foo` signatures, `Widget` explicitly implements `I2`’s `Foo` method. This lets the two methods coexist in one class. The only way to call an explicitly implemented member is to cast to its interface:

[PRE185]

Another reason to explicitly implement interface members is to hide members that are highly specialized and distracting to a type’s normal use case. For example, a type that implements `ISerializable` would typically want to avoid flaunting its `ISerializable` members unless explicitly cast to that interface.

## Implementing Interface Members Virtually

An implicitly implemented interface member is, by default, sealed. It must be marked `virtual` or `abstract` in the base class in order to be overridden:

[PRE186]

Calling the interface member through either the base class or the interface calls the subclass’s implementation:

[PRE187]

An explicitly implemented interface member cannot be marked `virtual`, nor can it be overridden in the usual manner. It can, however, be *reimplemented*.

## Reimplementing an Interface in a Subclass

A subclass can reimplement any interface member already implemented by a base class. Reimplementation hijacks a member implementation (when called through the interface) and works whether or not the member is `virtual` in the base class. It also works whether a member is implemented implicitly or explicitly—although it works best in the latter case, as we will demonstrate.

In the following example, `TextBox` implements `IUndoable.Undo` explicitly, and so it cannot be marked as `virtual`. To “override” it, `RichTextBox` must reimplement `IUndoable`’s `Undo` method:

[PRE188]

Calling the reimplemented member through the interface calls the subclass’s implementation:

[PRE189]

Assuming the same `RichTextBox` definition, suppose that `TextBox` implemented `Undo` *implicitly*:

[PRE190]

This would give us another way to call `Undo`, which would “break” the system, as shown in Case 3:

[PRE191]

Case 3 demonstrates that reimplementation hijacking is effective only when a member is called through the interface and not through the base class. This is usually undesirable in that it can create inconsistent semantics. This makes reimplementation most appropriate as a strategy for overriding *explicitly* implemented interface members.

### Alternatives to interface reimplementation

Even with explicit member implementation, interface reimplementation is problematic for a couple of reasons:

*   The subclass has no way to call the base class method.

*   The base class author might not anticipate that a method will be reimplemented and might not allow for the potential consequences.

Reimplementation can be a good last resort when subclassing hasn’t been anticipated. A better option, however, is to design a base class such that reimplementation will never be required. There are two ways to achieve this:

*   When implicitly implementing a member, mark it `virtual` if appropriate.

*   When explicitly implementing a member, use the following pattern if you anticipate that subclasses might need to override any logic:

[PRE192]

If you don’t anticipate any subclassing, you can mark the class as `sealed` to preempt interface reimplementation.

## Interfaces and Boxing

Converting a struct to an interface causes boxing. Calling an implicitly implemented member on a struct does not cause boxing:

[PRE193]

## Default Interface Members

From C# 8, you can add a default implementation to an interface member, making it optional to implement:

[PRE194]

This is advantageous if you want to add a member to an interface defined in a popular library without breaking (potentially thousands of) implementations.

Default implementations are always explicit, so if a class implementing `ILogger` fails to define a `Log` method, the only way to call it is through the interface:

[PRE195]

This prevents a problem of multiple implementation inheritance: if the same default member is added to two interfaces that a class implements, there is never an ambiguity as to which member is called.

## Static Interface Members

An interface can also declare static members. There are two kinds of static interface members:

*   Static nonvirtual interface members

*   Static virtual/abstract interface members

###### Note

In contrast to *instance* members, static members on interfaces are nonvirtual by default. To make a static interface member virtual, you must mark it with `static abstract` or `static virtual`.

### Static nonvirtual interface members

Static nonvirtual interface members exist mainly to help with writing default interface members. They are not implemented by classes or structs; instead, they are consumed directly. Along with methods, properties, events, and indexers, static nonvirtual members permit fields, which are typically accessed from code inside default member implementations:

[PRE196]

Static nonvirtual interface members are public by default, so they can be accessed from the outside:

[PRE197]

You can restrict this by adding an accessibility modifier (such as `private`, `protected`, or `internal`).

Instance fields are (still) prohibited. This is in line with the principle of interfaces, which is to define *behavior*, not *state*.

### Static virtual/abstract interface members

Static virtual/abstract interface members (from C# 11) enable *static polymorphism*, an advanced feature that we will discuss in [Chapter 4](ch04.html#advanced_chash). Static virtual interface members are marked with `static abstract` or `static virtual`:

[PRE198]

An implementing class or struct must implement static abstract members, and can optionally implement static virtual members:

[PRE199]

In addition to methods, properties, and events, operators and conversions are also legal targets for static virtual interface members (see [“Operator Overloading”](ch04.html#operator_overloading)). Static virtual interface members are called through a constrained type parameter; we will demonstrate this in [“Static Polymorphism”](ch04.html#static_polymorphism) and [“Generic Math”](ch04.html#generic_math-id00069), after covering generics later in this chapter.

# Enums

An enum is a special value type that lets you specify a group of named numeric constants. For example:

[PRE200]

We can use this enum type as follows:

[PRE201]

Each enum member has an underlying integral value. These are by default:

*   Underlying values are of type `int`.

*   The constants `0`, `1`, `2`... are automatically assigned, in the declaration order of the enum members.

You can specify an alternative integral type, as follows:

[PRE202]

You can also specify an explicit underlying value for each enum member:

[PRE203]

###### Note

The compiler also lets you explicitly assign *some* of the enum members. The unassigned enum members keep incrementing from the last explicit value. The preceding example is equivalent to the following:

[PRE204]

## Enum Conversions

You can convert an `enum` instance to and from its underlying integral value with an explicit cast:

[PRE205]

You can also explicitly cast one enum type to another. Suppose that `Horizontal​A⁠lignment` is defined as follows:

[PRE206]

A translation between the enum types uses the underlying integral values:

[PRE207]

The numeric literal `0` is treated specially by the compiler in an `enum` expression and does not require an explicit cast:

[PRE208]

There are two reasons for the special treatment of `0`:

*   The first member of an enum is often used as the “default” value.

*   For *combined enum* types, `0` means “no flags.”

## Flags Enums

You can combine enum members. To prevent ambiguities, members of a combinable enum require explicitly assigned values, typically in powers of two:

[PRE209]

or:

[PRE210]

To work with combined enum values, you use bitwise operators such as `|` and `&`. These operate on the underlying integral values:

[PRE211]

By convention, the `Flags` attribute should always be applied to an enum type when its members are combinable. If you declare such an `enum` without the `Flags` attribute, you can still combine members, but calling `ToString` on an `enum` instance will emit a number rather than a series of names.

By convention, a combinable enum type is given a plural rather than singular name.

For convenience, you can include combination members within an enum declaration itself:

[PRE212]

## Enum Operators

The operators that work with enums are:

[PRE213]

The bitwise, arithmetic, and comparison operators return the result of processing the underlying integral values. Addition is permitted between an enum and an integral type, but not between two enums.

## Type-Safety Issues

Consider the following enum:

[PRE214]

Because an enum can be cast to and from its underlying integral type, the actual value it can have might fall outside the bounds of a legal enum member:

[PRE215]

The bitwise and arithmetic operators can produce similarly invalid values:

[PRE216]

An invalid `BorderSide` would break the following code:

[PRE217]

One solution is to add another `else` clause:

[PRE218]

Another workaround is to explicitly check an enum value for validity. The static `Enum.IsDefined` method does this job:

[PRE219]

Unfortunately, `Enum.IsDefined` does not work for flagged enums. However, the following helper method (a trick dependent on the behavior of `Enum.ToString()`) returns `true` if a given flagged enum is valid:

[PRE220]

# Nested Types

A *nested type* is declared within the scope of another type:

[PRE221]

A nested type has the following features:

*   It can access the enclosing type’s private members and everything else the enclosing type can access.

*   You can declare it with the full range of access modifiers rather than just `public` and `internal`.

*   The default accessibility for a nested type is `private` rather than `internal`.

*   Accessing a nested type from outside the enclosing type requires qualification with the enclosing type’s name (like when accessing static members).

For example, to access `Color.Red` from outside our `TopLevel` class, we’d need to do this:

[PRE222]

All types (classes, structs, interfaces, delegates, and enums) can be nested within either a class or a struct.

Here is an example of accessing a private member of a type from a nested type:

[PRE223]

Here is an example of applying the `protected` access modifier to a nested type:

[PRE224]

Here is an example of referring to a nested type from outside the enclosing type:

[PRE225]

Nested types are used heavily by the compiler itself when it generates private classes that capture state for constructs such as iterators and anonymous methods.

###### Note

If the sole reason for using a nested type is to avoid cluttering a namespace with too many types, consider using a nested namespace, instead. A nested type should be used because of its stronger access control restrictions, or when the nested class must access private members of the containing class.

# Generics

C# has two separate mechanisms for writing code that is reusable across different types: *inheritance* and *generics*. Whereas inheritance expresses reusability with a base type, generics express reusability with a “template” that contains “placeholder” types. Generics, when compared to inheritance, can *increase type safety* and *reduce casting and boxing*.

###### Note

C# generics and C++ templates are similar concepts, but they work differently. We explain this difference in [“C# Generics Versus C++ Templates”](#chash_generics_versus_cplusplus_templat).

## Generic Types

A generic type declares *type parameters*—placeholder types to be filled in by the consumer of the generic type, which supplies the *type arguments*. Here is a generic type `Stack<T>`, designed to stack instances of type `T`. `Stack<T>` declares a single type parameter `T`:

[PRE226]

We can use `Stack<T>` as follows:

[PRE227]

`Stack<int>` fills in the type parameter `T` with the type argument `int`, implicitly creating a type on the fly (the synthesis occurs at runtime). Attempting to push a string onto our `Stack<int>` would, however, produce a compile-time error. `Stack<int>` effectively has the following definition (substitutions appear in bold, with the class name hashed out to avoid confusion):

[PRE228]

Technically, we say that `Stack<T>` is an *open type*, whereas `Stack<int>` is a *closed type*. At runtime, all generic type instances are closed—with the placeholder types filled in. This means that the following statement is illegal:

[PRE229]

However, it’s legal if it’s within a class or method that itself defines `T` as a type parameter:

[PRE230]

## Why Generics Exist

Generics exist to write code that is reusable across different types. Suppose that we need a stack of integers but we don’t have generic types. One solution would be to hardcode a separate version of the class for every required element type (e.g., `IntStack`, `StringStack`, etc.). Clearly, this would cause considerable code duplication. Another solution would be to write a stack that is generalized by using `object` as the element type:

[PRE231]

An `ObjectStack`, however, wouldn’t work as well as a hardcoded `IntStack` for specifically stacking integers. An `ObjectStack` would require boxing and downcasting that could not be checked at compile time:

[PRE232]

What we need is both a general implementation of a stack that works for all element types as well as a way to easily specialize that stack to a specific element type for increased type safety and reduced casting and boxing. Generics give us precisely this by allowing us to parameterize the element type. `Stack<T>` has the benefits of both `ObjectStack` and `IntStack`. Like `ObjectStack`, `Stack<T>` is written once to work *generally* across all types. Like `IntStack`, `Stack<T>` is *specialized* for a particular type—the beauty is that this type is `T`, which we substitute on the fly.

###### Note

`ObjectStack` is functionally equivalent to `Stack<object>`.

## Generic Methods

A generic method declares type parameters within the signature of a method.

With generic methods, many fundamental algorithms can be implemented in a general-purpose way. Here is a generic method that swaps the contents of two variables of any type `T`:

[PRE233]

`Swap<T>` is called as follows:

[PRE234]

Generally, there is no need to supply type arguments to a generic method, because the compiler can implicitly infer the type. If there is ambiguity, generic methods can be called with type arguments as follows:

[PRE235]

Within a generic *type*, a method is not classed as generic unless it *introduces* type parameters (with the angle bracket syntax). The `Pop` method in our generic stack merely uses the type’s existing type parameter, `T`, and is not classed as a generic method.

Methods and types are the only constructs that can introduce type parameters. Properties, indexers, events, fields, constructors, operators, and so on cannot declare type parameters, although they can partake in any type parameters already declared by their enclosing type. In our generic stack example, for instance, we could write an indexer that returns a generic item:

[PRE236]

Similarly, constructors can partake in existing type parameters but not *introduce* them:

[PRE237]

## Declaring Type Parameters

Type parameters can be introduced in the declaration of classes, structs, interfaces, delegates (covered in [Chapter 4](ch04.html#advanced_chash)), and methods. Other constructs, such as properties, cannot *introduce* a type parameter, but they can *use* one. For example, the property `Value` uses `T`:

[PRE238]

A generic type or method can have multiple parameters:

[PRE239]

To instantiate:

[PRE240]

Or:

[PRE241]

Generic type names and method names can be overloaded as long as the number of type parameters is different. For example, the following three type names do not conflict:

[PRE242]

###### Note

By convention, generic types and methods with a *single* type parameter typically name their parameter `T`, as long as the intent of the parameter is clear. When using *multiple* type parameters, each parameter is prefixed with `T` but has a more descriptive name.

## typeof and Unbound Generic Types

Open generic types do not exist at runtime: they are closed as part of compilation. However, it is possible for an *unbound* generic type to exist at runtime—purely as a `Type` object. The only way to specify an unbound generic type in C# is via the `typeof` operator:

[PRE243]

Open generic types are used in conjunction with the Reflection API ([Chapter 18](ch18.html#reflection_and_metadata)).

You can also use the `typeof` operator to specify a closed type:

[PRE244]

Or, you can specify an open type (which is closed at runtime):

[PRE245]

## The default Generic Value

You can use the `default` keyword to get the default value for a generic type parameter. The default value for a reference type is `null`, and the default value for a value type is the result of bitwise-zeroing the value type’s fields:

[PRE246]

From C# 7.1, you can omit the type argument for cases in which the compiler is able to infer it. We could replace the last line of code with this:

[PRE247]

## Generic Constraints

By default, you can substitute a type parameter with any type whatsoever. *Constraints* can be applied to a type parameter to require more specific type arguments. These are the possible constraints:

[PRE248]

In the following example, `GenericClass<T,U>` requires `T` to derive from (or be identical to) `SomeClass` and implement `Interface1`, and requires `U` to provide a parameterless constructor:

[PRE249]

You can apply constraints wherever type parameters are defined, in both methods and type definitions.

###### Note

A *constraint* is a *restriction*; however, the main purpose of type parameter constraints is to enable things that would otherwise be prohibited.

For instance, the constraint `T:Foo` lets you treat instances of `T` as `Foo`, and the constraint `T:new()` lets you construct new instances of `T`.

A *base-class constraint* specifies that the type parameter must subclass (or match) a particular class; an *interface constraint* specifies that the type parameter must implement that interface. These constraints allow instances of the type parameter to be implicitly converted to that class or interface. For example, suppose that we want to write a generic `Max` method, which returns the maximum of two values. We can take advantage of the generic interface defined in the `System` namespace called `IComparable<T>`:

[PRE250]

`CompareTo` returns a positive number if `this` is greater than `other`. Using this interface as a constraint, we can write a `Max` method as follows (to avoid distraction, null checking is omitted):

[PRE251]

The `Max` method can accept arguments of any type implementing `IComparable<T>` (which includes most built-in types such as `int` and `string`):

[PRE252]

###### Note

From C# 11, an interface constraint also lets you call static virtual/abstract members on that interface (see [“Static virtual/abstract interface members”](ch01.html#static_virtualsolidusabstract-id00040)). For example, if interface `IFoo` defines a static abstract method called `Bar`, the `T:IFoo` constraint makes it legal to call `T.Bar()`. We pick up this topic again in [“Static Polymorphism”](ch04.html#static_polymorphism).

The *class constraint* and *struct constraint* specify that `T` must be a reference type or (non-nullable) value type. A great example of the struct constraint is the `System.Nullable<T>` struct (we discuss this class in depth in [“Nullable Value Types”](ch04.html#nullable_value_types)):

[PRE253]

The *unmanaged constraint* (introduced in C# 7.3) is a stronger version of a struct constraint: `T` must be a simple value type or a struct that is (recursively) free of any reference types.

The *parameterless constructor constraint* requires `T` to have a public parameterless constructor. If this constraint is defined, you can call `new()` on `T`:

[PRE254]

The *naked type constraint* requires one type parameter to derive from (or match) another type parameter. In this example, the method `FilteredStack` returns another `Stack`, containing only the subset of elements where the type parameter `U` is of the type parameter `T`:

[PRE255]

## Subclassing Generic Types

A generic class can be subclassed just like a nongeneric class. The subclass can leave the base class’s type parameters open, as in the following example:

[PRE256]

Or, the subclass can close the generic type parameters with a concrete type:

[PRE257]

A subtype can also introduce fresh type arguments:

[PRE258]

###### Note

Technically, *all* type arguments on a subtype are fresh: you could say that a subtype closes and then reopens the base type arguments. This means that a subclass can give new (and potentially more meaningful) names to the type arguments that it reopens:

[PRE259]

## Self-Referencing Generic Declarations

A type can name *itself* as the concrete type when closing a type argument:

[PRE260]

The following are also legal:

[PRE261]

## Static Data

Static data is unique for each closed type:

[PRE262]

## Type Parameters and Conversions

C#’s cast operator can perform several kinds of conversion, including the following:

*   Numeric conversion

*   Reference conversion

*   Boxing/unboxing conversion

*   Custom conversion (via operator overloading; see [Chapter 4](ch04.html#advanced_chash))

The decision as to which kind of conversion will take place happens at *compile time*, based on the known types of the operands. This creates an interesting scenario with generic type parameters, because the precise operand types are unknown at compile time. If this leads to ambiguity, the compiler generates an error.

The most common scenario is when you want to perform a reference conversion:

[PRE263]

Without knowledge of `T`’s actual type, the compiler is concerned that you might have intended this to be a *custom conversion*. The simplest solution is to instead use the `as` operator, which is unambiguous because it cannot perform custom conversions:

[PRE264]

A more general solution is to first cast to `object`. This works because conversions to/from `object` are assumed not to be custom conversions, but reference or boxing/unboxing conversions. In this case, `StringBuilder` is a reference type, so it must be a reference conversion:

[PRE265]

Unboxing conversions can also introduce ambiguities. The following could be an unboxing, numeric, or custom conversion:

[PRE266]

The solution, again, is to first cast to `object` and then to `int` (which then unambiguously signals an unboxing conversion in this case):

[PRE267]

## Covariance

Assuming `A` is convertible to `B`, `X` has a covariant type parameter if `X<A>` is convertible to `X<B>`.

###### Note

With C#’s notion of covariance (and contravariance), “convertible” means convertible via an *implicit reference conversion*—such as `A` *subclassing* `B`, or `A` *implementing* `B`. Numeric conversions, boxing conversions, and custom conversions are not included.

For instance, type `IFoo<T>` has a covariant `T` if the following is legal:

[PRE268]

Interfaces permit covariant type parameters (as do delegates; see [Chapter 4](ch04.html#advanced_chash)), but classes do not. Arrays also allow covariance (`A[]` can be converted to `B[]` if `A` has an implicit reference conversion to `B`) and are discussed here for comparison.

###### Note

Covariance and contravariance (or simply “variance”) are advanced concepts. The motivation behind introducing and enhancing variance in C# was to allow generic interface and generic types (in particular, those defined in .NET, such as `IEnumerable<T>`) to work more as you’d expect. You can benefit from this without understanding the details behind covariance and contravariance.

### Variance is not automatic

To ensure static type safety, type parameters are not automatically variant. Consider the following:

[PRE269]

The following fails to compile:

[PRE270]

That restriction prevents the possibility of runtime failure with the following code:

[PRE271]

Lack of covariance, however, can hinder reusability. Suppose, for instance, that we wanted to write a method to `Wash` a stack of animals:

[PRE272]

Calling `Wash` with a stack of bears would generate a compile-time error. One workaround is to redefine the `Wash` method with a constraint:

[PRE273]

We can now call `Wash` as follows:

[PRE274]

Another solution is to have `Stack<T>` implement an interface with a covariant type parameter, as you’ll see shortly.

### Arrays

For historical reasons, array types support covariance. This means that `B[]` can be cast to `A[]` if `B` subclasses `A` (and both are reference types):

[PRE275]

The downside of this reusability is that element assignments can fail at runtime:

[PRE276]

### Declaring a covariant type parameter

Type parameters on interfaces and delegates can be declared covariant by marking them with the `out` modifier. This modifier ensures that, unlike with arrays, covariant type parameters are fully type-safe.

We can illustrate this with our `Stack<T>` class by having it implement the following interface:

[PRE277]

The `out` modifier on `T` indicates that `T` is used only in *output positions* (e.g., return types for methods). The `out` modifier flags the type parameter as *covariant* and allows us to do this:

[PRE278]

The conversion from `bears` to `animals` is permitted by the compiler—by virtue of the type parameter being covariant. This is type-safe because the case the compiler is trying to avoid—pushing a `Camel` onto the stack—can’t occur, because there’s no way to feed a `Camel` *in*to an interface where `T` can appear only in *out*put positions.

###### Note

Covariance (and contravariance) in interfaces is something that you typically *consume*: it’s less common that you need to *write* variant interfaces.

###### Warning

Curiously, method parameters marked as `out` are not eligible for covariance, due to a limitation in the CLR.

We can leverage the ability to cast covariantly to solve the reusability problem described earlier:

[PRE279]

###### Note

The `IEnumerator<T>` and `IEnumerable<T>` interfaces described in [Chapter 7](ch07.html#collections-id00055) have a covariant `T`. This allows you to cast `IEnumerable<string>` to `IEnumerable<object>`, for instance.

The compiler will generate an error if you use a covariant type parameter in an *input* position (e.g., a parameter to a method or a writable property).

###### Note

Covariance (and contravariance) works only for elements with *reference conversions*—not *boxing conversions*. (This applies both to type parameter variance and array variance.) So, if you wrote a method that accepted a parameter of type `IPoppable<object>`, you could call it with `IPoppable<string>` but not `IPoppable<int>`.

## Contravariance

We previously saw that, assuming that `A` allows an implicit reference conversion to `B`, a type `X` has a covariant type parameter if `X<A>` allows a reference conversion to `X<B>`. *Contravariance* is when you can convert in the reverse direction—from `X<B>` to `X<A>`. This is supported if the type parameter appears only in *input* positions and is designated with the `in` modifier. Extending our previous example, suppose the `Stack<T>` class implements the following interface:

[PRE280]

We can now legally do this:

[PRE281]

No member in `IPushable` *outputs* a `T`, so we can’t get into trouble by casting `animals` to `bears` (there’s no way to `Pop`, for instance, through that interface).

###### Note

Our `Stack<T>` class can implement both `IPushable<T>` and `IPoppable<T>`—despite `T` having opposing variance annotations in the two interfaces! This works because you must exercise variance through the interface and not the class; therefore, you must commit to the lens of either `IPoppable` or `IPushable` before performing a variant conversion. This lens then restricts you to the operations that are legal under the appropriate variance rules.

This also illustrates why *classes* do not allow variant type parameters: concrete implementations typically require data to flow in both directions.

To give another example, consider the following interface, defined in the `System` namespace:

[PRE282]

Because the interface has a contravariant `T`, we can use an `IComparer<**object**>` to compare two *strings*:

[PRE283]

Mirroring covariance, the compiler will report an error if you try to use a contravariant type parameter in an output position (e.g., as a return value or in a readable property).

## C# Generics Versus C++ Templates

C# generics are similar in application to C++ templates, but they work very differently. In both cases, a synthesis between the producer and consumer must take place in which the placeholder types of the producer are filled in by the consumer. However, with C# generics, producer types (i.e., open types such as `List<T>`) can be compiled into a library (such as *mscorlib.dll*). This works because the synthesis between the producer and the consumer that produces closed types doesn’t actually happen until runtime. With C++ templates, this synthesis is performed at compile time. This means that in C++ you don’t deploy template libraries as *.dll*s—they exist only as source code. It also makes it difficult to dynamically inspect, let alone create, parameterized types on the fly.

To dig deeper into why this is the case, consider again the `Max` method in C#:

[PRE284]

Why couldn’t we have implemented it like this?

[PRE285]

The reason is that `Max` needs to be compiled once and work for all possible values of `T`. Compilation cannot succeed because there is no single meaning for `>` across all values of `T`—in fact, not every `T` even has a `>` operator. In contrast, the following code shows the same `Max` method written with C++ templates. This code will be compiled separately for each value of `T`, taking on whatever semantics `>` has for a particular `T`, and failing to compile if a particular `T` does not support the `>` operator:

[PRE286]

^([1](ch03.html#ch01fn5-marker)) The reference type can also be `System.ValueType` or `System.Enum` ([Chapter 6](ch06.html#dotnet_fundamentals)).