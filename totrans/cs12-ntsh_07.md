# Chapter 7\. Collections

.NET provides a standard set of types for storing and managing collections of objects. These include resizable lists, linked lists, and sorted and unsorted dictionaries, as well as arrays. Of these, only arrays form part of the C# language; the remaining collections are just classes you instantiate like any other.

We can divide the types in the .NET BCL for collections into the following categories:

*   Interfaces that define standard collection protocols

*   Ready-to-use collection classes (lists, dictionaries, etc.)

*   Base classes for writing application-specific collections

This chapter covers each of these categories, with an additional section on the types used in determining element equality and order.

The collection namespaces are as follows:

| Namespace | Contains |
| --- | --- |
| `System.Collections` | Nongeneric collection classes and interfaces |
| `System.Collections.Specialized` | Strongly typed nongeneric collection classes |
| `System.Collections.Generic` | Generic collection classes and interfaces |
| `System.Collections.ObjectModel` | Proxies and bases for custom collections |
| `System.Collections.Concurrent` | Thread-safe collections (see [Chapter 23](ch23.html#spanless_thantgreater_than_and-id00089)) |

# Enumeration

In computing, there are many different kinds of collections, ranging from simple data structures such as arrays or linked lists, to more complex ones such as red/black trees and hashtables. Although the internal implementation and external characteristics of these data structures vary widely, the ability to traverse the contents of the collection is an almost universal need. The .NET BCL supports this need via a pair of interfaces (`IEnumerable`, `IEnumerator`, and their generic counterparts) that allow different data structures to expose a common traversal API. These are part of a larger set of collection interfaces illustrated in [Figure 7-1](#collection_interfaces).

![Collection interfaces](assets/cn10_0701.png)

###### Figure 7-1\. Collection interfaces

## IEnumerable and IEnumerator

The `IEnumerator` interface defines the basic low-level protocol by which elements in a collection are traversed—or enumerated—in a forward-only manner. Its declaration is as follows:

[PRE0]

`MoveNext` advances the current element or “cursor” to the next position, returning `false` if there are no more elements in the collection. `Current` returns the element at the current position (usually cast from `object` to a more specific type). `MoveNext` must be called before retrieving the first element—this is to allow for an empty collection. The `Reset` method, if implemented, moves back to the start, allowing the collection to be enumerated again. `Reset` exists mainly for Component Object Model (COM) interoperability; calling it directly is generally avoided because it’s not universally supported (and is unnecessary in that it’s usually just as easy to instantiate a new enumerator).

Collections do not usually *implement* enumerators; instead, they *provide* enumerators, via the interface `IEnumerable`:

[PRE1]

By defining a single method retuning an enumerator, `IEnumerable` provides flexibility in that the iteration logic can be farmed out to another class. Moreover, it means that several consumers can enumerate the collection at once without interfering with one another. You can think of `IEnumerable` as “IEnumeratorProvider,” and it is the most basic interface that collection classes implement.

The following example illustrates low-level use of `IEnumerable` and `IEnumerator`:

[PRE2]

However, it’s rare to call methods on enumerators directly in this manner because C# provides a syntactic shortcut: the `foreach` statement. Here’s the same example rewritten using `foreach`:

[PRE3]

## IEnumerable<T> and IEnumerator<T>

`IEnumerator` and `IEnumerable` are nearly always implemented in conjunction with their extended generic versions:

[PRE4]

By defining a typed version of `Current` and `GetEnumerator`, these interfaces strengthen static type safety, avoid the overhead of boxing with value-type elements, and are more convenient to the consumer. Arrays automatically implement `IEnumerable<T>` (where `T` is the member type of the array).

Thanks to the improved static type safety, calling the following method with an array of characters will generate a compile-time error:

[PRE5]

It’s a standard practice for collection classes to publicly expose `IEnumerable<T>` while “hiding” the nongeneric `IEnumerable` through explicit interface implementation. This is so that if you directly call `GetEnumerator()`, you get back the type-safe generic `IEnumerator<T>`. There are times, though, when this rule is broken for reasons of backward compatibility (generics did not exist prior to C# 2.0). A good example is arrays—these must return the nongeneric (the nice way of putting it is “classic”) `IEnumerator` to avoid breaking earlier code. To get a generic `IEnumerator<T>`, you must cast to expose the explicit interface:

[PRE6]

Fortunately, you rarely need to write this sort of code, thanks to the `foreach` statement.

### IEnumerable<T> and IDisposable

`IEnumerator<T>` inherits from `IDisposable`. This allows enumerators to hold references to resources such as database connections—and ensure that those resources are released when enumeration is complete (or abandoned partway through). The `foreach` statement recognizes this detail and translates the following:

[PRE7]

into the logical equivalent of this:

[PRE8]

The `using` block ensures disposal—more on `IDisposable` in [Chapter 12](ch12.html#disposal_and_garbage_collection).

## Implementing the Enumeration Interfaces

You might want to implement `IEnumerable` or `IEnumerable<T>` for one or more of the following reasons:

*   To support the `foreach` statement

*   To interoperate with anything expecting a standard collection

*   To meet the requirements of a more sophisticated collection interface

*   To support collection initializers

To implement `IEnumerable`/`IEnumerable<T>`, you must provide an enumerator. You can do this in one of three ways:

*   If the class is “wrapping” another collection, by returning the wrapped collection’s enumerator

*   Via an iterator using `yield return`

*   By instantiating your own `IEnumerator`/`IEnumerator<T>` implementation

###### Note

You can also subclass an existing collection: `Collection<T>` is designed just for this purpose (see [“Customizable Collections and Proxies”](#customizable_collections_and_proxies)). Yet another approach is to use the LINQ query operators, which we cover in [Chapter 8](ch08.html#linq_queries).

Returning another collection’s enumerator is just a matter of calling `GetEnumerator` on the inner collection. However, this is viable only in the simplest scenarios in which the items in the inner collection are exactly what are required. A more flexible approach is to write an iterator, using C#’s `yield return` statement. An *iterator* is a C# language feature that assists in writing collections, in the same way the `foreach` statement assists in consuming collections. An iterator automatically handles the implementation of `IEnumerable` and `IEnumerator`—or their generic versions. Here’s a simple example:

[PRE9]

Notice the “black magic”: `GetEnumerator` doesn’t appear to return an enumerator at all! Upon parsing the `yield return` statement, the compiler writes a hidden nested enumerator class behind the scenes and then refactors `GetEnumerator` to instantiate and return that class. Iterators are powerful and simple (and are used extensively in the implementation of LINQ-to-Object’s standard query operators).

Keeping with this approach, we can also implement the generic interface `IEnumerable<T>`:

[PRE10]

Because `IEnumerable<T>` inherits from `IEnumerable`, we must implement both the generic and the nongeneric versions of `GetEnumerator`. In accordance with standard practice, we’ve implemented the nongeneric version explicitly. It can simply call the generic `GetEnumerator` because `IEnumerator<T>` inherits from `IEnumerator`.

The class we’ve just written would be suitable as a basis from which to write a more sophisticated collection. However, if we need nothing above a simple `IEnumerable<T>` implementation, the `yield return` statement allows for an easier variation. Rather than writing a class, you can move the iteration logic into a method returning a generic `IEnumerable<T>` and let the compiler take care of the rest. Here’s an example:

[PRE11]

Here’s our method in use:

[PRE12]

The final approach in writing `GetEnumerator` is to write a class that implements `IEnumerator` directly. This is exactly what the compiler does behind the scenes, in resolving iterators. (Fortunately, it’s rare that you’ll need to go this far yourself.) The following example defines a collection that’s hardcoded to contain the integers 1, 2, and 3:

[PRE13]

###### Note

Implementing `Reset` is optional—you can instead throw a `NotSupportedException`.

Note that the first call to `MoveNext` should move to the first (and not the second) item in the list.

To get on par with an iterator in functionality, we must also implement `IEnumerator<T>`. Here’s an example with bounds checking omitted for brevity:

[PRE14]

The example with generics is faster because `IEnumerator<int>.Current` doesn’t require casting from `int` to `object` and so avoids the overhead of boxing.

# The ICollection and IList Interfaces

Although the enumeration interfaces provide a protocol for forward-only iteration over a collection, they don’t provide a mechanism to determine the size of the collection, access a member by index, or modify the collection. For such functionality, .NET defines the `ICollection`, `IList`, and `IDictionary` interfaces. Each comes in both generic and nongeneric versions; however, the nongeneric versions exist mostly for legacy support.

[Figure 7-1](#collection_interfaces) showed the inheritance hierarchy for these interfaces. The easiest way to summarize them is as follows:

`IEnumerable<T>` (and `IEnumerable`)

Provides minimum functionality (enumeration only)

`ICollection<T>` (and `ICollection`)

Provides medium functionality (e.g., the `Count` property)

`IList<T>`*/*`IDictionary<K,V>` and their nongeneric versions

Provide maximum functionality (including “random” access by index/key)

###### Note

It’s rare that you’ll need to *implement* any of these interfaces. In nearly all cases when you need to write a collection class, you can instead subclass `Collection<T>` (see [“Customizable Collections and Proxies”](#customizable_collections_and_proxies)). LINQ provides yet another option that covers many scenarios.

The generic and nongeneric versions differ in ways over and above what you might expect, particularly in the case of `ICollection`. The reasons for this are mostly historical: because generics came later, the generic interfaces were developed with the benefit of hindsight, leading to a different (and better) choice of members. For this reason, `ICollection<T>` does not extend `ICollection`, `IList<T>` does not extend `IList`, and `IDictionary<TKey, TValue>` does not extend `IDictionary`. Of course, a collection class itself is free to implement both versions of an interface if beneficial (which it often is).

###### Note

Another, subtler reason for `IList<T>` not extending `IList` is that casting to `IList<T>` would then return an interface with both `Add(T)` and `Add(object)` members. This would effectively defeat static type safety because you could call `Add` with an object of any type.

This section covers `ICollection<T>` and `IList<T>` and their nongeneric versions; [“Dictionaries”](#dictionaries) covers the dictionary interfaces.

###### Note

There is no *consistent* rationale in the way the words “collection” and “list” are applied throughout the .NET libraries. For instance, because `IList<T>` is a more functional version of `ICollection<T>`, you might expect the class `List<T>` to be correspondingly more functional than the class `Collection<T>`. This is not the case. It’s best to consider the terms “collection” and “list” as broadly synonymous, except when a specific type is involved.

## ICollection<T> and ICollection

`ICollection<T>` is the standard interface for countable collections of objects. It provides the ability to determine the size of a collection (`Count`), determine whether an item exists in the collection (`Contains`), copy the collection into an array (`ToArray`), and determine whether the collection is read-only (`IsReadOnly`). For writable collections, you can also `Add`, `Remove`, and `Clear` items from the collection. And because it extends `IEnumerable<T>`, it can also be traversed via the `foreach` statement:

[PRE15]

The nongeneric `ICollection` is similar in providing a countable collection, but it doesn’t provide functionality for altering the list or checking for element membership:

[PRE16]

The nongeneric interface also defines properties to assist with synchronization ([Chapter 14](ch14.html#concurrency_and_asynchron))—these were dumped in the generic version because thread safety is no longer considered intrinsic to the collection.

Both interfaces are fairly straightforward to implement. If implementing a read-only `ICollection<T>`, the `Add`, `Remove`, and `Clear` methods should throw a `NotSupportedException`.

These interfaces are usually implemented in conjunction with either the `IList` or the `IDictionary` interface.

## IList<T> and IList

`IList<T>` is the standard interface for collections indexable by position. In addition to the functionality inherited from `ICollection<T>` and `IEnumerable<T>`, it provides the ability to read or write an element by position (via an indexer) and insert/remove by position:

[PRE17]

The `IndexOf` methods perform a linear search on the list, returning `−1` if the specified item is not found.

The nongeneric version of `IList` has more members because it inherits less from `ICollection`:

[PRE18]

The `Add` method on the nongeneric `IList` interface returns an integer—this is the index of the newly added item. In contrast, the `Add` method on `ICollection<T>` has a `void` return type.

The general-purpose `List<T>` class is the quintessential implementation of both `IList<T>` and `IList`. C# arrays also implement both the generic and nongeneric `IList`s (although the methods that add or remove elements are hidden via explicit interface implementation and throw a `NotSupportedException` if called).

###### Warning

An `ArgumentException` is thrown if you try to access a multidimensional array via `IList`’s indexer. This is a trap when writing methods such as the following:

[PRE19]

This might appear bulletproof, but it will throw an exception if called with a multidimensional array. You can test for a multidimensional array at runtime with this expression (more on this in [Chapter 19](ch19.html#dynamic_programming)):

[PRE20]

## IReadOnlyCollection<T> and IReadOnlyList<T>

.NET also defines collection and list interfaces that expose just the members required for read-only operations:

[PRE21]

Because the type parameter for these interfaces is used only in output positions, it’s marked as *covariant*. This allows a list of cats, for instance, to be treated as a read-only list of animals. In contrast, `T` is not marked as covariant with `ICollection<T>` and `IList<T>`, because `T` is used in both input and output positions.

###### Note

These interfaces represent a read-only *view* of a collection or list; the underlying implementation might still be writable. Most of the writable (*mutable*) collections implement both the read-only and read/write interfaces.

In addition to letting you work with collections covariantly, the read-only interfaces allow a class to publicly expose a read-only view of a private writable collection. We demonstrate this—along with a better solution—in [“ReadOnlyCollection<T>”](#readonlycollectionless_thantgreater_tha).

`IReadOnlyList<T>` maps to the Windows Runtime type `IVectorView<T>`.

# The Array Class

The `Array` class is the implicit base class for all single and multidimensional arrays, and it is one of the most fundamental types implementing the standard collection interfaces. The `Array` class provides type unification, so a common set of methods is available to all arrays, regardless of their declaration or underlying element type.

Because arrays are so fundamental, C# provides explicit syntax for their declaration and initialization, which we described in Chapters [2](ch02.html#chash_language_basics) and [3](ch03.html#creating_types_in_chash). When an array is declared using C#’s syntax, the CLR implicitly subtypes the `Array` class—synthesizing a *pseudotype* appropriate to the array’s dimensions and element types. This pseudotype implements the typed generic collection interfaces, such as `IList<string>`.

The CLR also treats array types specially upon construction, assigning them a contiguous space in memory. This makes indexing into arrays highly efficient, but prevents them from being resized later on.

`Array` implements the collection interfaces up to `IList<T>` in both their generic and nongeneric forms. `IList<T>` itself is implemented explicitly, though, to keep `Array`’s public interface clean of methods such as `Add` or `Remove`, which throw an exception on fixed-length collections such as arrays. The `Array` class does actually offer a static `Resize` method, although this works by creating a new array and then copying over each element. As well as being inefficient, references to the array elsewhere in the program will still point to the original version. A better solution for resizable collections is to use the `List<T>` class (described in the following section).

An array can contain value-type or reference-type elements. Value-type elements are stored in place in the array, so an array of three long integers (each 8 bytes) will occupy 24 bytes of contiguous memory. A reference type element, however, occupies only as much space in the array as a reference (4 bytes in a 32-bit environment or 8 bytes in a 64-bit environment). [Figure 7-2](#arrays_in_memory) illustrates the effect, in memory, of the following program:

[PRE22]

![Arrays in memory](assets/cn10_0702.png)

###### Figure 7-2\. Arrays in memory

Because `Array` is a class, arrays are always (themselves) reference types—regardless of the array’s element type. This means that the statement `arrayB = arrayA` results in two variables that reference the same array. Similarly, two distinct arrays will always fail an equality test, unless you employ a *structural equality comparer*, which compares every element of the array:

[PRE23]

Arrays can be duplicated by calling the `Clone` method: `arrayB = arrayA.Clone()`. However, this results in a shallow clone, meaning that only the memory represented by the array itself is copied. If the array contains value-type objects, the values themselves are copied; if the array contains reference type objects, just the references are copied (resulting in two arrays whose members reference the same objects). [Figure 7-3](#shallow_cloning_an_array) demonstrates the effect of adding the following code to our example:

[PRE24]

![Shallow-cloning an array](assets/cn10_0703.png)

###### Figure 7-3\. Shallow-cloning an array

To create a deep copy—for which reference type subobjects are duplicated—you must loop through the array and clone each element manually. The same rules apply to other .NET collection types.

Although `Array` is designed primarily for use with 32-bit indexers, it also has limited support for 64-bit indexers (allowing an array to theoretically address up to 2^(64) elements) via several methods that accept both `Int32` and `Int64` parameters. These overloads are useless in practice because the CLR does not permit any object—including arrays—to exceed two gigabytes in size (whether running on a 32- or 64-bit environment).

###### Warning

Many of the methods on the `Array` class that you expect to be instance methods are in fact static methods. This is an odd design decision, and means that you should check for both static and instance methods when looking for a method on `Array`.

## Construction and Indexing

The easiest way to create and index arrays is through C#’s language constructs:

[PRE25]

Alternatively, you can instantiate an array dynamically by calling `Array.CreateInstance`. This allows you to specify element type and rank (number of dimensions) at runtime as well as allowing nonzero-based arrays through specifying a lower bound. Nonzero-based arrays are not compatible with the .NET Common Language Specification (CLS), and should not be exposed as public members in a library that might be consumed by a program written in F# or Visual Basic.

The `GetValue` and `SetValue` methods let you access elements in a dynamically created array (they also work on ordinary arrays):

[PRE26]

Zero-indexed arrays created dynamically can be cast to a C# array of a matching or compatible type (compatible by standard array-variance rules). For example, if `Apple` subclasses `Fruit`, `Apple[]` can be cast to `Fruit[]`. This leads to the issue of why `object[]` was not used as the unifying array type rather than the `Array` class. The answer is that `object[]` is incompatible with both multidimensional and value-type arrays (and non-zero-based arrays). An `int[]` array cannot be cast to `object[]`. Hence, we require the `Array` class for full type unification.

`GetValue` and `SetValue` also work on compiler-created arrays, and they are useful when writing methods that can deal with an array of any type and rank. For multidimensional arrays, they accept an *array* of indexers:

[PRE27]

The following method prints the first element of any array, regardless of rank:

[PRE28]

###### Note

For working with arrays of unknown type but known rank, generics provide an easier and more efficient solution:

[PRE29]

`SetValue` throws an exception if the element is of an incompatible type for the array.

When an array is instantiated, whether via language syntax or `Array.Create​In⁠stance`, its elements are automatically initialized to their default values. For arrays with reference type elements, this means writing nulls; for arrays with value-type elements, this means bitwise “zeroing” the members. The `Array` class also provides this functionality on demand via the `Clear` method:

[PRE30]

This method doesn’t affect the size of the array. This is in contrast to the usual use of `Clear` (such as in `ICollection<T>.Clear`) whereby the collection is reduced to zero elements.

## Enumeration

Arrays are easily enumerated with a `foreach` statement:

[PRE31]

You can also enumerate using the static `Array.ForEach` method, defined as follows:

[PRE32]

This uses an `Action` delegate, with this signature:

[PRE33]

Here’s the first example rewritten with `Array.ForEach`:

[PRE34]

We can further simplify this with a *collection expression* (from C# 12):

[PRE35]

## Length and Rank

`Array` provides the following methods and properties for querying length and rank:

[PRE36]

`GetLength` and `GetLongLength` return the length for a given dimension (`0` for a single-dimensional array), and `Length` and `LongLength` return the total number of elements in the array—all dimensions included.

`GetLowerBound` and `GetUpperBound` are useful with nonzero indexed arrays. `Get​Up⁠perBound` returns the same result as adding `GetLowerBound` to `GetLength` for any given dimension.

## Searching

The `Array` class offers a range of methods for finding elements within a one-dimensional array:

`BinarySearch` methods

For rapidly searching a sorted array for a particular item

`IndexOf`/`LastIndex` methods

For searching unsorted arrays for a particular item

`Find`/`FindLast`/`FindIndex`/`FindLastIndex`/`FindAll`/`Exists`/`TrueForAll`

For searching unsorted arrays for item(s) that satisfy a given `Predicate<T>`

None of the array-searching methods throws an exception if the specified value is not found. Instead, if an item is not found, methods returning an integer return `−1` (assuming a zero-indexed array), and methods returning a generic type return the type’s default value (e.g., `0` for an `int`, or `null` for a `string` ).

The binary search methods are fast, but they work only on sorted arrays and require that the elements be compared for *order* rather than simply *equality*. To this effect, the binary search methods can accept an `IComparer` or `IComparer<T>` object to arbitrate on ordering decisions (see [“Plugging in Equality and Order”](#plugging_in_equality_and_order)”). This must be consistent with any comparer used in originally sorting the array. If no comparer is provided, the type’s default ordering algorithm will be applied based on its implementation of `IComparable` / `IComparable<T>`.

The `IndexOf` and `LastIndexOf` methods perform a simple enumeration over the array, returning the position of the first (or last) element that matches the given value.

The predicate-based searching methods allow a method delegate or lambda expression to arbitrate on whether a given element is a “match.” A predicate is simply a delegate accepting an object and returning `true` or `false`:

[PRE37]

In the following example, we search an array of strings for a name containing the letter “a”:

[PRE38]

Here’s the same code shortened with a lambda expression:

[PRE39]

`FindAll` returns an array of all items satisfying the predicate. In fact, it’s equivalent to `Enumerable.Where` in the `System.Linq` namespace, except that `FindAll` returns an array of matching items rather than an `IEnumerable<T>` of the same.

`Exists` returns `true` if any array member satisfies the given predicate, and is equivalent to `Any` in `System.Linq.Enumerable`.

`TrueForAll` returns `true` if all items satisfy the predicate, and is equivalent to `All` in `System.Linq.Enumerable`.

## Sorting

`Array` has the following built-in sorting methods:

[PRE40]

Each of these methods is additionally overloaded to also accept the following:

[PRE41]

The following illustrates the simplest use of `Sort`:

[PRE42]

The methods accepting a pair of arrays work by rearranging the items of each array in tandem, basing the ordering decisions on the first array. In the next example, both the numbers and their corresponding words are sorted into numerical order:

[PRE43]

`Array.Sort` requires that the elements in the array implement `IComparable` (see [“Order Comparison”](ch06.html#order_comparison)). This means that most built-in C# types (such as integers, as in the preceding example) can be sorted. If the elements are not intrinsically comparable or you want to override the default ordering, you must provide `Sort` with a custom `comparison` provider that reports on the relative position of two elements. There are ways to do this:

*   Via a helper object that implements `IComparer` /`IComparer<T>` (see [“Plugging in Equality and Order”](#plugging_in_equality_and_order))

*   Via a `Comparison` delegate:

[PRE44]

The `Comparison` delegate follows the same semantics as `IComparer<T>.CompareTo`: if `x` comes before `y`, a negative integer is returned; if `x` comes after `y`, a positive integer is returned; if `x` and `y` have the same sorting position, `0` is returned.

In the following example, we sort an array of integers such that the odd numbers come first:

[PRE45]

###### Note

As an alternative to calling `Sort`, you can use LINQ’s `OrderBy` and `ThenBy` operators. Unlike `Array.Sort`, the LINQ operators don’t alter the original array, instead emitting the sorted result in a fresh `IEnumerable<T>` sequence.

## Reversing Elements

The following `Array` methods reverse the order of all—or a portion of—elements in an array:

[PRE46]

## Copying

`Array` provides four methods to perform shallow copying: `Clone`, `CopyTo`, `Copy`, and `ConstrainedCopy`. The former two are instance methods; the latter two are static methods.

The `Clone` method returns a whole new (shallow-copied) array. The `CopyTo` and `Copy` methods copy a contiguous subset of the array. Copying a multidimensional rectangular array requires you to map the multidimensional index to a linear index. For example, the middle square (`position[1,1]`) in a 3 × 3 array is represented with the index 4, from the calculation: 1 * 3 + 1\. The source and destination ranges can overlap without causing a problem.

`ConstrainedCopy` performs an *atomic* operation: if all of the requested elements cannot be successfully copied (due to a type error, for instance), the operation is rolled back.

`Array` also provides an `AsReadOnly` method that returns a wrapper that prevents elements from being reassigned.

## Converting and Resizing

`Array.ConvertAll` creates and returns a new array of element type `TOutput`, calling the supplied `Converter` delegate to copy over the elements. `Converter` is defined as follows:

[PRE47]

The following converts an array of floats to an array of integers:

[PRE48]

The `Resize` method works by creating a new array and copying over the elements, returning the new array via the reference parameter. However, any references to the original array in other objects will remain unchanged.

###### Note

The `System.Linq` namespace offers an additional buffet of extension methods suitable for array conversion. These methods return an `IEnumerable<T>`, which you can convert back to an array via `Enumerable` ’s `ToArray` method.

# Lists, Queues, Stacks, and Sets

.NET provides a basic set of concrete collection classes that implement the interfaces described in this chapter. This section concentrates on the *list-like* collections (versus the *dictionary-like* collections, which we cover in [“Dictionaries”](#dictionaries)). As with the interfaces we discussed previously, you usually have a choice of generic or nongeneric versions of each type. In terms of flexibility and performance, the generic classes win, making their nongeneric counterparts redundant except for backward compatibility. This differs from the situation with collection interfaces, for which the nongeneric versions are still occasionally useful.

Of the classes described in this section, the generic `List` class is the most commonly used.

## List<T> and ArrayList

The generic `List` and nongeneric `ArrayList` classes provide a dynamically sized array of objects and are among the most commonly used of the collection classes. `ArrayList` implements `IList`, whereas `List<T>` implements both `IList` and `IList<T>` (and the read-only version, `IReadOnlyList<T>`). Unlike with arrays, all interfaces are implemented publicly, and methods such as `Add` and `Remove` are exposed and work as you would expect.

Internally, `List<T>` and `ArrayList` work by maintaining an internal array of objects, replaced with a larger array upon reaching capacity. Appending elements is efficient (because there is usually a free slot at the end), but inserting elements can be slow (because all elements after the insertion point must be shifted to make a free slot), as can removing elements (especially near the start). As with arrays, searching is efficient if the `BinarySearch` method is used on a list that has been sorted, but it is otherwise inefficient because each item must be individually checked.

###### Note

`List<T>` is up to several times faster than `ArrayList` if `T` is a value type, because `List<T>` avoids the overhead of boxing and unboxing elements.

`List<T>` and `ArrayList` provide constructors that accept an existing collection of elements: these copy each element from the existing collection into the new `List<T>` or `ArrayList`:

[PRE49]

In addition to these members, `List<T>` provides instance versions of all of `Array`’s searching and sorting methods.

The following code demonstrates `List`’s properties and methods (for examples on searching and sorting, see [“The Array Class”](#the_array_class)):

[PRE50]

The nongeneric `ArrayList` class requires clumsy casts—as the following example demonstrates:

[PRE51]

Such casts cannot be verified by the compiler; the following compiles successfully but then fails at runtime:

[PRE52]

###### Note

An `ArrayList` is functionally similar to `List<object>`. Both are useful when you need a list of mixed-type elements that share no common base type (other than `object`). A possible advantage of choosing an `ArrayList`, in this case, would be if you need to deal with the list using reflection ([Chapter 19](ch19.html#dynamic_programming)). Reflection is easier with a nongeneric `ArrayList` than a `List<object>`.

If you import the `System.Linq` namespace, you can convert an `ArrayList` to a generic `List` by calling `Cast` and then `ToList`:

[PRE53]

`Cast` and `ToList` are extension methods in the `System.Linq.Enumerable` class.

## LinkedList<T>

`LinkedList<T>` is a generic doubly linked list (see [Figure 7-4](#linkedlistless_thantgreater-id00075)). A doubly linked list is a chain of nodes in which each references the node before, the node after, and the actual element. Its main benefit is that an element can always be inserted efficiently anywhere in the list because it just involves creating a new node and updating a few references. However, finding where to insert the node in the first place can be slow because there’s no intrinsic mechanism to index directly into a linked list; each node must be traversed, and binary-chop searches are not possible.

![LinkedList<T>](assets/cn10_0704.png)

###### Figure 7-4\. LinkedList<T>

`LinkedList<T>` implements `IEnumerable<T>` and `ICollection<T>` (and their nongeneric versions), but not `IList<T>` because access by index is not supported. List nodes are implemented via the following class:

[PRE54]

When adding a node, you can specify its position either relative to another node or at the start/end of the list. `LinkedList<T>` provides the following methods for this:

[PRE55]

Similar methods are provided to remove elements:

[PRE56]

`LinkedList<T>` has internal fields to keep track of the number of elements in the list as well as the head and tail of the list. These are exposed in the following public properties:

[PRE57]

`LinkedList<T>` also supports the following searching methods (each requiring that the list be internally enumerated):

[PRE58]

Finally, `LinkedList<T>` supports copying to an array for indexed processing and obtaining an enumerator to support the `foreach` statement:

[PRE59]

Here’s a demonstration on the use of `LinkedList<string>`:

[PRE60]

## Queue<T> and Queue

`Queue<T>` and `Queue` are first-in, first-out (FIFO) data structures, providing methods to `Enqueue` (add an item to the tail of the queue) and `Dequeue` (retrieve and remove the item at the head of the queue). A `Peek` method is also provided to return the element at the head of the queue without removing it, and a `Count` property (useful in checking that elements are present before dequeuing).

Although queues are enumerable, they do not implement `IList<T>`/`IList`, because members cannot be accessed directly by index. A `ToArray` method is provided, however, for copying the elements to an array from which they can be randomly accessed:

[PRE61]

The following is an example of using `Queue<int>`:

[PRE62]

Queues are implemented internally using an array that’s resized as required—much like the generic `List` class. The queue maintains indexes that point directly to the head and tail elements; therefore, enqueuing and dequeuing are extremely quick operations (except when an internal resize is required).

## Stack<T> and Stack

`Stack<T>` and `Stack` are last-in, first-out (LIFO) data structures, providing methods to `Push` (add an item to the top of the stack) and `Pop` (retrieve and remove an element from the top of the stack). A nondestructive `Peek` method is also provided, as is a `Count` property and a `ToArray` method for exporting the data for random access:

[PRE63]

The following example demonstrates `Stack<int>`:

[PRE64]

Stacks are implemented internally with an array that’s resized as required, as with `Queue<T>` and `List<T>`.

## BitArray

A `BitArray` is a dynamically sized collection of compacted `bool` values. It is more memory efficient than both a simple array of `bool` and a generic `List` of `bool` because it uses only one bit for each value, whereas the `bool` type otherwise occupies one byte for each value.

`BitArray`’s indexer reads and writes individual bits:

[PRE65]

There are four bitwise operator methods (`And`, `Or`, `Xor`, and `Not`). All but the last accept another `BitArray`:

[PRE66]

## HashSet<T> and SortedSet<T>

`HashSet<T>` and `SortedSet<T>` have the following distinguishing features:

*   Their `Contains` methods execute quickly using a hash-based lookup.

*   They do not store duplicate elements and silently ignore requests to add duplicates.

*   You cannot access an element by position.

`SortedSet<T>` keeps elements in order, whereas `HashSet<T>` does not.

The commonality of the `HashSet<T>` and `SortedSet<T>` types is captured by the interface `ISet<T>`. From .NET 5, these classes also implement an interface called `IReadOnlySet<T>`, which is also implemented by the immutable set types (see [“Immutable Collections”](#immutable_collections)).

`HashSet<T>` is implemented with a hashtable that stores just keys; `SortedSet<T>` is implemented with a red/black tree.

Both collections implement `ICollection<T>` and offer methods that you would expect, such as `Contains`, `Add`, and `Remove`. In addition, there’s a predicate-based removal method called `RemoveWhere`.

The following constructs a `HashSet<char>` from an existing collection, tests for membership, and then enumerates the collection (notice the absence of duplicates):

[PRE67]

(The reason we can pass a `string` into `HashSet<char>`’s constructor is because `string` implements `IEnumerable<char>`.)

The really interesting methods are the set operations. The following set operations are *destructive* in that they modify the set:

[PRE68]

whereas the following methods simply query the set and so are nondestructive:

[PRE69]

`UnionWith` adds all the elements in the second set to the original set (excluding duplicates). `IntersectWith` removes the elements that are not in both sets. We can extract all of the vowels from our set of characters as follows:

[PRE70]

`ExceptWith` removes the specified elements from the source set. Here, we strip all vowels from the set:

[PRE71]

`SymmetricExceptWith` removes all but the elements that are unique to one set or the other:

[PRE72]

Note that because `HashSet<T>` and `SortedSet<T>` implement `IEnumerable<T>`, you can use another type of set (or collection) as the argument to any of the set operation methods.

`SortedSet<T>` offers all the members of `HashSet<T>`, plus the following:

[PRE73]

`SortedSet<T>` also accepts an optional `IComparer<T>` in its constructor (rather than an *equality comparer*).

Here’s an example of loading the same letters into a `SortedSet<char>`:

[PRE74]

Following on from this, we can obtain the letters in the set between *f* and *i* as follows:

[PRE75]

# Dictionaries

A dictionary is a collection in which each element is a key/value pair. Dictionaries are most commonly used for lookups and sorted lists.

.NET defines a standard protocol for dictionaries, via the interfaces `IDictionary` and `IDictionary <TKey, TValue>`, as well as a set of general-purpose dictionary classes. The classes each differ in the following regard:

*   Whether or not items are stored in sorted sequence

*   Whether or not items can be accessed by position (index) as well as by key

*   Whether it’s generic or nongeneric

*   Whether it’s fast or slow to retrieve items by key from a large dictionary

[Table 7-1](#dictionary_classes) summarizes each of the dictionary classes and how they differ in these respects. The performance times are in milliseconds and based on performing 50,000 operations on a dictionary with integer keys and values on a 1.5 GHz PC. (The differences in performance between generic and nongeneric counterparts using the same underlying collection structure are due to boxing, and show up only with value-type elements.)

Table 7-1\. Dictionary classes

| Type | Internal structure | Retrieve by index? | Memory overhead (avg. bytes per item) | Speed: random insertion | Speed: sequential insertion | Speed: retrieval by key |
| --- | --- | --- | --- | --- | --- | --- |
| **Unsorted** |  |  |  |  |  |  |
| `Dictionary <K,V>` | Hashtable | No | 22 | 30 | 30 | 20 |
| `Hashtable` | Hashtable | No | 38 | 50 | 50 | 30 |
| `ListDictionary` | Linked list | No | 36 | 50,000 | 50,000 | 50,000 |
| `OrderedDictionary` | Hashtable + array | Yes | 59 | 70 | 70 | 40 |
| **Sorted** |  |  |  |  |  |  |
| `SortedDictionary <K,V>` | Red/black tree | No | 20 | 130 | 100 | 120 |
| `SortedList <K,V>` | 2xArray | Yes | 2 | 3,300 | 30 | 40 |
| `SortedList` | 2xArray | Yes | 27 | 4,500 | 100 | 180 |

In Big-O notation, retrieval time by key is as follows:

*   O(1) for `Hashtable`, `Dictionary`, and `OrderedDictionary`

*   O(log *n*) for `SortedDictionary` and `SortedList`

*   O(*n*) for `ListDictionary` (and nondictionary types such as `List<T>`)

*n* is the number of elements in the collection.

## IDictionary<TKey,TValue>

`IDictionary<TKey,TValue>` defines the standard protocol for all key/value-based collections. It extends `ICollection<T>` by adding methods and properties to access elements based on a key of arbitrary type:

[PRE76]

###### Note

There’s also an interface called `IReadOnlyDictionary<TKey,TValue>` that defines the read-only subset of dictionary members.

To add an item to a dictionary, you either call `Add` or use the index’s set accessor—the latter adds an item to the dictionary if the key is not already present (or updates the item if it is present). Duplicate keys are forbidden in all dictionary implementations, so calling `Add` twice with the same key throws an exception.

To retrieve an item from a dictionary, use either the indexer or the `TryGetValue` method. If the key doesn’t exist, the indexer throws an exception, whereas `TryGetValue` returns `false`. You can test for membership explicitly by calling `Contain⁠s​Key`; however, this incurs the cost of two lookups if you then subsequently retrieve the item.

Enumerating directly over an `IDictionary<TKey,TValue>` returns a sequence of `KeyValuePair` structs:

[PRE77]

You can enumerate over just the keys or values via the dictionary’s `Keys`/`Values` properties.

We demonstrate the use of this interface with the generic `Dictionary` class in the following section.

## IDictionary

The nongeneric `IDictionary` interface is the same in principle as `IDictionary<TKey,TValue>`, apart from two important functional differences. It’s important to be aware of these differences, because `IDictionary` appears in legacy code (including the .NET BCL itself in places):

*   Retrieving a nonexistent key via the indexer returns null (rather than throwing an exception).

*   `Contains` tests for membership rather than `ContainsKey`.

Enumerating over a nongeneric `IDictionary` returns a sequence of `Dictionary​En⁠try` structs:

[PRE78]

## Dictionary<TKey,TValue> and Hashtable

The generic `Dictionary` class is one of the most commonly used collections (along with the `List<T>` collection). It uses a hashtable data structure to store keys and values, and it is fast and efficient.

###### Note

The nongeneric version of `Dictionary<TKey,TValue>` is called `Hashtable`; there is no nongeneric class called `Dictionary`. When we refer simply to `Dictionary`, we mean the generic `Dictionary<TKey,TValue>` class.

`Dictionary` implements both the generic and nongeneric `IDictionary` interfaces, the generic `IDictionary` being exposed publicly. `Dictionary` is, in fact, a “textbook” implementation of the generic `IDictionary`.

Here’s how to use it:

[PRE79]

Its underlying hashtable works by converting each element’s key into an integer hashcode—a pseudo-unique value—and then applying an algorithm to convert the hashcode into a hash key. This hash key is used internally to determine which “bucket” an entry belongs to. If the bucket contains more than one value, a linear search is performed on the bucket. A good hash function does not strive to return strictly unique hashcodes (which would usually be impossible); it strives to return hashcodes that are evenly distributed across the 32-bit integer space. This avoids the scenario of ending up with a few very large (and inefficient) buckets.

A dictionary can work with keys of any type, providing it’s able to determine equality between keys and obtain hashcodes. By default, equality is determined via the key’s `object.Equals` method, and the pseudo-unique hashcode is obtained via the key’s `GetHashCode` method. You can change this behavior either by overriding these methods or by providing an `IEqualityComparer` object when constructing the dictionary. A common application of this is to specify a case-insensitive equality comparer when using string keys:

[PRE80]

We discuss this further in [“Plugging in Equality and Order”](#plugging_in_equality_and_order).

As with many other types of collections, you can improve the performance of a dictionary slightly by specifying the collection’s expected size in the constructor, avoiding or lessening the need for internal resizing operations.

The nongeneric version is named `Hashtable` and is functionally similar apart from differences stemming from it exposing the nongeneric `IDictionary` interface discussed previously.

The downside to `Dictionary` and `Hashtable` is that the items are not sorted. Furthermore, the original order in which the items were added is not retained. As with all dictionaries, duplicate keys are not allowed.

###### Note

When the generic collections were introduced back in 2005, the CLR team chose to name them according to what they represent (`Dictionary`, `List`) rather than how they are internally implemented (`Hashtable`, `ArrayList`). Although this is good because it gives them the freedom to later change the implementation, it also means that the *performance contract* (often the most important criteria in choosing one kind of collection over another) is no longer captured in the name.

## OrderedDictionary

An `OrderedDictionary` is a nongeneric dictionary that maintains elements in the same order that they were added. With an `OrderedDictionary`, you can access elements both by index and by key.

###### Note

An `OrderedDictionary` is not a *sorted* dictionary.

An `OrderedDictionary` is a combination of a `Hashtable` and an `ArrayList`. This means that it has all the functionality of a `Hashtable`, plus functions such as `RemoveAt`, and an integer indexer. It also exposes `Keys` and `Values` properties that return elements in their original order.

This class was introduced in .NET 2.0; yet, peculiarly, there’s no generic version.

## ListDictionary and HybridDictionary

`ListDictionary` uses a singly linked list to store the underlying data. It doesn’t provide sorting, although it does preserve the original entry order of the items. `ListDictionary` is extremely slow with large lists. Its only real “claim to fame” is its efficiency with very small lists (fewer than 10 items).

`HybridDictionary` is a `ListDictionary` that automatically converts to a `Hashtable` upon reaching a certain size, to address `ListDictionary`’s problems with performance. The idea is to get a low memory footprint when the dictionary is small, and good performance when the dictionary is large. However, given the overhead in converting from one to the other—and the fact that a `Dictionary` is not excessively heavy or slow in either scenario—you wouldn’t suffer unreasonably by using a `Dictionary` to begin with.

Both classes come only in nongeneric form.

## Sorted Dictionaries

The .NET BCL provides two dictionary classes internally structured such that their content is always sorted by key:

*   `SortedDictionary<TKey,TValue>`

*   `SortedList<TKey,TValue>`^([1](ch07.html#ch01fn6))

(In this section, we abbreviate `<TKey,TValue>` to `<,>`.)

`SortedDictionary<,>` uses a red/black tree: a data structure designed to perform consistently well in any insertion or retrieval scenario.

`SortedList<,>` is implemented internally with an ordered array pair, providing fast retrieval (via a binary-chop search) but poor insertion performance (because existing values need to be shifted to make room for a new entry).

`SortedDictionary<,>` is much faster than `SortedList<,>` at inserting elements in a random sequence (particularly with large lists). `SortedList<,>`, however, has an extra ability: to access items by index as well as by key. With a sorted list, you can go directly to the *n*th element in the sorting sequence (via the indexer on the `Keys`/`Values` properties). To do the same with a `SortedDictionary<,>`, you must manually enumerate over *n* items. (Alternatively, you could write a class that combines a sorted dictionary with a list class.)

None of the three collections allows duplicate keys (as is the case with all dictionaries).

The following example uses reflection to load all of the methods defined in `System.Object` into a sorted list keyed by name and then enumerates their keys and values:

[PRE81]

Here’s the result of the first enumeration:

[PRE82]

Here’s the result of the second enumeration:

[PRE83]

Notice that we populated the dictionary through its indexer. If we instead used the `Add` method, it would throw an exception because the `object` class upon which we’re reflecting overloads the `Equals` method, and you can’t add the same key twice to a dictionary. By using the indexer, the later entry overwrites the earlier entry, preventing this error.

###### Note

You can store multiple members of the same key by making each value element a list:

[PRE84]

Extending our example, the following retrieves the `MethodInfo` whose key is `"GetHashCode"`, just as with an ordinary dictionary:

[PRE85]

So far, everything we’ve done would also work with a `SortedDictionary<,>`. The following two lines, however, which retrieve the last key and value, work only with a sorted list:

[PRE86]

# Customizable Collections and Proxies

The collection classes discussed in previous sections are convenient in that you can directly instantiate them, but they don’t allow you to control what happens when an item is added to or removed from the collection. With strongly typed collections in an application, you sometimes need this control; for instance:

*   To fire an event when an item is added or removed

*   To update properties because of the added or removed item

*   To detect an “illegal” add/remove operation and throw an exception (for example, if the operation violates a business rule)

The .NET BCL provides collection classes for this exact purpose, in the `System​.Col⁠lections.ObjectModel` namespace. These are essentially proxies or wrappers that implement `IList<T>` or `IDictionary<,>` by forwarding the methods through to an underlying collection. Each `Add`, `Remove`, or `Clear` operation is routed via a virtual method that acts as a “gateway” when overridden.

Customizable collection classes are commonly used for publicly exposed collections; for instance, a collection of controls exposed publicly on a `System​.Win⁠dows.Form` class.

## Collection<T> and CollectionBase

The `Collection<T>` class is a customizable wrapper for `List<T>`.

As well as implementing `IList<T>` and `IList`, it defines four additional virtual methods and a protected property as follows:

[PRE87]

The virtual methods provide the gateway by which you can “hook in” to change or enhance the list’s normal behavior. The protected `Items` property allows the implementer to directly access the “inner list”—this is used to make changes internally without the virtual methods firing.

The virtual methods need not be overridden; they can be left alone until there’s a requirement to alter the list’s default behavior. The following example demonstrates the typical “skeleton” use of `Collection<T>`:

[PRE88]

As it stands, `AnimalCollection` is no more functional than a simple `List<Animal>`; its role is to provide a base for future extension. To illustrate, let’s now add a `Zoo` property to `Animal` so that it can reference the `Zoo` in which it lives and override each of the virtual methods in `Collection<Animal>` to maintain that property automatically:

[PRE89]

`Collection<T>` also has a constructor accepting an existing `IList<T>`. Unlike with other collection classes, the supplied list is *proxied* rather than *copied*, meaning that subsequent changes will be reflected in the wrapping `Collection<T>` (although *without* `Collection<T>`’s virtual methods firing). Conversely, changes made via the `Collection<T>` will change the underlying list.

### CollectionBase

`CollectionBase` is the nongeneric version of `Collection<T>`. This provides most of the same features as `Collection<T>` but is clumsier to use. Instead of the template methods `InsertItem`, `RemoveItem`, `SetItem`, and `ClearItem`, `CollectionBase` has “hook” methods that double the number of methods required: `OnInsert`, `OnInsertComplete`, `OnSet`, `OnSetComplete`, `OnRemove`, `OnRemoveComplete`, `OnClear`, and `OnClearComplete`. Because `CollectionBase` is nongeneric, you must also implement typed methods when subclassing it—at a minimum, a typed indexer and `Add` method.

## KeyedCollection<TKey,TItem> and DictionaryBase

`KeyedCollection<TKey,TItem>` subclasses `Collection<TItem>`. It both adds and subtracts functionality. What it adds is the ability to access items by key, much like with a dictionary. What it subtracts is the ability to proxy your own inner list.

A keyed collection has some resemblance to an `OrderedDictionary` in that it combines a linear list with a hashtable. However, unlike `OrderedDictionary`, it doesn’t implement `IDictionary` and doesn’t support the concept of a key/value *pair*. Keys are obtained instead from the items themselves: via the abstract `GetKeyForItem` method. This means enumerating a keyed collection is just like enumerating an ordinary list.

You can best think of `KeyedCollection<TKey,TItem>` as `Collection<TItem>` plus fast lookup by key.

Because it subclasses `Collection<>`, a keyed collection inherits all of `Collection<>`’s functionality, except for the ability to specify an existing list in construction. The additional members it defines are as follows:

[PRE90]

`GetKeyForItem` is what the implementer overrides to obtain an item’s key from the underlying object. The `ChangeItemKey` method must be called if the item’s key property changes, in order to update the internal dictionary. The `Dictionary` property returns the internal dictionary used to implement the lookup, which is created when the first item is added. This behavior can be changed by specifying a creation threshold in the constructor, delaying the internal dictionary from being created until the threshold is reached (in the interim, a linear search is performed if an item is requested by key). A good reason not to specify a creation threshold is that having a valid dictionary can be useful in obtaining an `ICollection<>` of keys, via the `Dictionary`’s `Keys` property. This collection can then be passed on to a public property.

The most common use for `KeyedCollection<,>` is in providing a collection of items accessible both by index and by name. To demonstrate this, let’s revisit the zoo, this time implementing `AnimalCollection` as a `KeyedCollection<string,​Ani⁠mal>`:

[PRE91]

The following code demonstrates its use:

[PRE92]

### DictionaryBase

The nongeneric version of `KeyedCollection` is called `DictionaryBase`. This legacy class takes a very different approach in that it implements `IDictionary` and uses clumsy hook methods like `CollectionBase`: `OnInsert`, `OnInsertComplete`, `OnSet`, `OnSetComplete`, `OnRemove`, `OnRemoveComplete`, `OnClear`, and `OnClearComplete` (and additionally, `OnGet`). The primary advantage of implementing `IDictionary` over taking the `KeyedCollection` approach is that you don’t need to subclass it in order to obtain keys. But since the very purpose of `DictionaryBase` is to be subclassed, it’s no advantage at all. The improved model in `KeyedCollection` is almost certainly due to the fact that it was written some years later, with the benefit of hindsight. `DictionaryBase` is best considered useful for backward compatibility.

## ReadOnlyCollection<T>

`ReadOnlyCollection<T>` is a wrapper, or *proxy*, that provides a read-only view of a collection. This is useful in allowing a class to publicly expose read-only access to a collection that the class can still update internally.

A read-only collection accepts the input collection in its constructor, to which it maintains a permanent reference. It doesn’t take a static copy of the input collection, so subsequent changes to the input collection are visible through the read-only wrapper.

To illustrate, suppose that your class wants to provide read-only public access to a list of strings called `Names`. We could do this as follows:

[PRE93]

Although `Names` returns a read-only interface, the consumer can still downcast at runtime to `List<string>` or `IList<string>` and then call `Add`, `Remove`, or `Clear` on the list. `ReadOnlyCollection<T>` provides a more robust solution:

[PRE94]

Now, only members within the `Test` class can alter the list of names:

[PRE95]

# Immutable Collections

We just described how `ReadOnlyCollection<T>` creates a read-only view of a collection. Restricting the ability to write (*mutate*) a collection—or any other object—simplifies software and reduces bugs.

The *immutable collections* extend this principle, by providing collections that cannot be modified at all after initialization. Should you need to add an item to an immutable collection, you must instantiate a new collection, leaving the old one untouched.

Immutability is a hallmark of *functional programming* and has the following benefits:

*   It eliminates a large class of bugs associated with changing state.

*   It vastly simplifies parallelism and multithreading, by avoiding most of the thread-safety problems that we describe in Chapters [14](ch14.html#concurrency_and_asynchron), [22](ch22.html#parallel_programming-id00071), and [23](ch23.html#spanless_thantgreater_than_and-id00089).

*   It makes code easier to reason about.

The disadvantage of immutability is that when you need to make a change, you must create a whole new object. This incurs a performance hit, although there are mitigating strategies that we discuss in this section, including the ability to reuse portions of the original structure.

The immutable collections are part of .NET (in .NET Framework, they are available via the *System.Collections.Immutable* NuGet package). All collections are defined in the `System.Collections.Immutable` namespace:

| Type | Internal structure |
| --- | --- |
| `ImmutableArray<T>` | Array |
| `ImmutableList<T>` | AVL tree |
| `ImmutableDictionary<K,V>` | AVL tree |
| `ImmutableHashSet<T>` | AVL tree |
| `ImmutableSortedDictionary<K,V>` | AVL tree |
| `ImmutableSortedSet<T>` | AVL tree |
| `ImmutableStack<T>` | Linked list |
| `ImmutableQueue<T>` | Linked list |

The `ImmutableArray<T>` and `ImmutableList<T>` types are both immutable versions of `List<T>`. Both do the same job but with different performance characteristics that we discuss in [“Immutable Collections and Performance”](#immutable_collections_and_performance).

The immutable collections expose a public interface similar to their mutable counterparts. The key difference is that the methods that appear to alter the collection (such as `Add` or `Remove`) don’t alter the original collection; instead they return a new collection with the requested item added or removed. This is called *nondestructive mutation*.

###### Note

Immutable collections prevent the adding and removing of items; they don’t prevent the items *themselves* from being mutated. To get the full benefits of immutability, you need to ensure that only immutable items end up in an immutable collection.

## Creating Immutable Collections

Each immutable collection type offers a `Create<T>()` method, which accepts optional initial values and returns an initialized immutable collection:

[PRE96]

Each collection also offers a `CreateRange<T>` method, which does the same job as `Create<T>`; the difference is that its parameter type is `IEnumerable<T>` instead of `params T[]`.

You can also create an immutable collection from an existing `IEnumerable<T>`, using appropriately extension methods (`ToImmutableArray`, `ToImmutableList`, `ToImmutableDictionary`, and so on):

[PRE97]

## Manipulating Immutable Collections

The `Add` method returns a new collection containing the existing elements plus the new one:

[PRE98]

The `Remove` method operates in the same fashion, returning a new collection with the item removed.

Repeatedly adding or removing elements in this manner is inefficient, because a new immutable collection is created for each add or remove operation. A better solution is to call `AddRange` (or `RemoveRange`), which accepts an `IEnumerable<T>` of items, which are all added or removed in one go:

[PRE99]

The immutable list and array also defines `Insert` and `InsertRange` methods to insert elements at a particular index, a `RemoveAt` method to remove at an index, and `RemoveAll`, which removes based on a predicate.

## Builders

For more complex initialization needs, each immutable collection class defines a *builder* counterpart. Builders are classes that are functionally equivalent to a mutable collection, with similar performance characteristics. After the data is initialized, calling `.ToImmutable()` on a builder returns an immutable collection.

[PRE100]

You also can use builders to *batch* multiple updates to an existing immutable collection:

[PRE101]

## Immutable Collections and Performance

Most of the immutable collections use an *AVL tree* internally, which allows the add/remove operations to reuse portions of the original internal structure rather than having to re-create the entire thing from scratch. This reduces the overhead of add/remove operations from potentially *huge* (with large collections) to just *moderately large*, but it comes at the cost of making read operations slower. The end result is that most immutable collections are slower than their mutable counterparts for both reading and writing.

The most seriously affected is `ImmutableList<T>`, which for both read and add operations, is 10 to 200 times slower than `List<T>` (depending on the size of the list). This is why `ImmutableArray<T>` exists: by using an array inside, it avoids the overhead for read operations (for which it’s comparable in performance to an ordinary mutable array). The flipside is that it’s *much* slower than (even) `ImmutableList<T>` for add operations because none of the original structure can be reused.

Hence, `ImmutableArray<T>` is desirable when you want unimpeded *read*-performance and don’t expect many subsequent calls to `Add` or `Remove` (without using a builder).

| Type | Read performance | Add performance |
| --- | --- | --- |
| `ImmutableList<T>` | Slow | Slow |
| `ImmutableArray<T>` | Very fast | Very slow |

###### Note

Calling `Remove` on an `ImmutableArray` is more expensive than calling `Remove` on a `List<T>`—even in the worst-case scenario of removing the first element—because allocating the new collection places additional load on the garbage collector.

Although the immutable collections as a whole incur a potentially significant performance cost, it’s important to keep the overall magnitude in perspective. An `Add` operation on an `ImmutableList` with a million elements is still likely to occur in less than a microsecond on a typical laptop, and a read operation, in less than 100 nanoseconds. And, if you need to perform write-operations in a loop, you can avoid the accumulated cost with a builder.

The following factors also work to mitigate the costs:

*   Immutability allows for easy concurrency and parallelization ([Chapter 23](ch23.html#spanless_thantgreater_than_and-id00089)), so you can employ all available cores. Parallelizing with mutable state easily leads to errors and requires the use of locks or concurrent collections, both of which hurt performance.

*   With immutability, you don’t need to “defensively copy” collections or data structures to guard against unexpected change. This was a factor in favoring the use of immutable collections in writing recent portions of Visual Studio.

*   In most typical programs, few collections have enough items for the difference to matter.

In addition to Visual Studio, the well-performing Microsoft Roslyn toolchain was built with immutable collections, demonstrating how the benefits can outweigh the costs.

# Frozen Collections

From .NET 8, the `System.Collections.Frozen` namespace contains the following two read-only collection classes:

[PRE102]

These are similar to `ImmutableDictionary<K,V>` and `ImmutableHashSet<T>`, but lack methods for nondestructive mutation (such as `Add` or `Remove`), allowing for highly optimized read performance. To create a frozen collection, you start with another collection or sequence and then call the `ToFrozenDictionary` or `ToFrozenSet` extension method:

[PRE103]

Frozen collections are great for lookups that are initialized at the start of a program and then used throughout the life of the application:

[PRE104]

The frozen collections implement the standard dictionary/set interfaces, including their read-only versions. In this example, we exposed our `FrozenDictionary<string,string>` as a field of type `IReadOnlyDictionary<string,string>`.

# Plugging in Equality and Order

In the sections [“Equality Comparison”](ch04.html#equality_comparison-id00016) and [“Order Comparison”](ch06.html#order_comparison), we described the standard .NET protocols that make a type equatable, hashable, and comparable. A type that implements these protocols can function correctly in a dictionary or sorted list “out of the box.” More specifically:

*   A type for which `Equals` and `GetHashCode` return meaningful results can be used as a key in a `Dictionary` or `Hashtable`.

*   A type that implements `IComparable` /`IComparable<T>` can be used as a key in any of the *sorted* dictionaries or lists.

A type’s default equating or comparison implementation typically reflects what is most “natural” for that type. Sometimes, however, the default behavior is not what you want. You might need a dictionary whose `string` type key is treated without respect to case. Or you might want a sorted list of customers, sorted by each customer’s postcode. For this reason, .NET also defines a matching set of “plug-in” protocols. The plug-in protocols achieve two things:

*   They allow you to switch in alternative equating or comparison behavior.

*   They allow you to use a dictionary or sorted collection with a key type that’s not intrinsically equatable or comparable.

The plug-in protocols consist of the following interfaces:

`IEqualityComparer` and `IEqualityComparer<T>`

*   Performs plug-in *equality comparison and hashing*

*   Recognized by `Hashtable` and `Dictionary`

`IComparer` and `IComparer<T>`

*   Performs plug-in *order comparison*

*   Recognized by the sorted dictionaries and collections; also, `Array.Sort`

Each interface comes in both generic and nongeneric forms. The `IEquality​Com⁠parer` interfaces also have a default implementation in a class called `Equality​Comparer`.

In addition, there are interfaces called `IStructuralEquatable` and `IStructuralComparable` which allow for the option of structural comparisons on classes and arrays.

## IEqualityComparer and EqualityComparer

An equality comparer switches in nondefault equality and hashing behavior, primarily for the `Dictionary` and `Hashtable` classes.

Recall the requirements of a hashtable-based dictionary. It needs answers to two questions for any given key:

*   Is it the same as another?

*   What is its integer hashcode?

An equality comparer answers these questions by implementing the `IEquality​Com⁠parer` interfaces:

[PRE105]

To write a custom comparer, you implement one or both of these interfaces (implementing both gives maximum interoperability). Because this is somewhat tedious, an alternative is to subclass the abstract `EqualityComparer` class, defined as follows:

[PRE106]

`EqualityComparer` implements both interfaces; your job is simply to override the two abstract methods.

The semantics for `Equals` and `GetHashCode` follow the same rules for `object.Equals` and `object.GetHashCode`, described in [Chapter 6](ch06.html#dotnet_fundamentals). In the following example, we define a `Customer` class with two fields and then write an equality comparer that matches both the first and last names:

[PRE107]

To illustrate how this works, let’s create two customers:

[PRE108]

Because we’ve not overridden `object.Equals`, normal reference type equality semantics apply:

[PRE109]

The same default equality semantics apply when using these customers in a `Dictionary` without specifying an equality comparer:

[PRE110]

Now, with the custom equality comparer:

[PRE111]

In this example, we would have to be careful not to change the customer’s `FirstName` or `LastName` while it was in use in the dictionary; otherwise, its hashcode would change and the `Dictionary` would break.

### EqualityComparer<T>.Default

Calling `EqualityComparer<T>.Default` returns a general-purpose equality comparer that you can use as an alternative to the static `object.Equals` method. The advantage is that it first checks whether `T` implements `IEquatable<T>`, and if so, it calls that implementation instead, avoiding the boxing overhead. This is particularly useful in generic methods:

[PRE112]

### ReferenceEqualityComparer.Instance (.NET 5+)

From .NET 5, `ReferenceEqualityComparer.Instance` returns an equality comparer that always applies referential equality. In the case of value types, its `Equals` method always returns false.

## IComparer and Comparer

Comparers are used to switch in custom ordering logic for sorted dictionaries and collections.

Note that a comparer is useless to the unsorted dictionaries such as `Dictionary` and `Hashtable`—these require an `IEqualityComparer` to get hashcodes. Similarly, an equality comparer is useless for sorted dictionaries and collections.

Here are the `IComparer` interface definitions:

[PRE113]

As with equality comparers, there’s an abstract class that you can subtype instead of implementing the interfaces:

[PRE114]

The following example illustrates a class that describes a wish as well as a comparer that sorts wishes by priority:

[PRE115]

The `object.Equals` check ensures that we can never contradict the `Equals` method. Calling the static `object.Equals` method in this case is better than calling `x.Equals` because it still works if `x` is null!

Here’s how our `PriorityComparer` is used to sort a `List`:

[PRE116]

In the next example, `SurnameComparer` allows you to sort surname strings in an order suitable for a phonebook listing:

[PRE117]

Here’s `SurnameComparer` in use in a sorted dictionary:

[PRE118]

## StringComparer

`StringComparer` is a predefined plug-in class for equating and comparing strings, allowing you to specify language and case sensitivity. `StringComparer` implements both `IEqualityComparer` and `IComparer` (and their generic versions), so you can use it with any type of dictionary or sorted collection.

Because `StringComparer` is abstract, you obtain instances via its static properties. `StringComparer.Ordinal` mirrors the default behavior for string equality comparison and `StringComparer.CurrentCulture` for order comparison. Here are all of its static members:

[PRE119]

In the following example, an ordinal case-insensitive dictionary is created such that `dict["Joe"]` and `dict["JOE"]` mean the same thing:

[PRE120]

In the next example, an array of names is sorted, using Australian English:

[PRE121]

The final example is a culture-aware version of the `SurnameComparer` we wrote in the previous section (to compare names suitable for a phonebook listing):

[PRE122]

## IStructuralEquatable and IStructuralComparable

As we discussed in [Chapter 6](ch06.html#dotnet_fundamentals), structs implement *structural comparison* by default: two structs are equal if all of their fields are equal. Sometimes, however, structural equality and order comparison are useful as plug-in options on other types, as well—such as arrays. The following interfaces help with this:

[PRE123]

The `IEqualityComparer`/`IComparer` that you pass in are applied to each individual element in the composite object. We can demonstrate this by using arrays. In the following example, we compare two arrays for equality, first using the default `Equals` method, then using `IStructuralEquatable`’s version:

[PRE124]

Here’s another example:

[PRE125]

^([1](ch07.html#ch01fn6-marker)) There’s also a functionally identical nongeneric version of this called `SortedList`.