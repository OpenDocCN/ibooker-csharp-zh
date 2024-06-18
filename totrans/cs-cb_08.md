# Chapter 8\. Matching with Patterns

Historically, developers have implemented business rules with various logical checks and comparisons. Sometimes the rules are complex—naturally leading to code that’s difficult to write, read, and maintain. Think about how often you’ve encountered multibranch logic with multivariate comparisons and multiple levels of nesting.

To help ease this complexity, modern programming languages have begun introducing pattern matching—features of the language that help match facts to results with declarative syntax. In C#, pattern matching manifests as a growing list of features added in each new version, especially from C# 7 and later.

The theme of this chapter revolves around hotel scheduling and using patterns for business rules. The criteria is usually around a type of customer such as Bronze, Silver, or Gold, with Gold being the highest level because those customers have more points from more frequent hotel stays.

This chapter discusses pattern matching for properties, tuples, and types. There’s also a couple of sections on logical operations and how they enable and simplify multiconditional patterns. Surprisingly, C# had some form of pattern matching since v1.0\. The first section of this chapter discusses the `is` and `as` operators and shows the new enhancements to the `is` operator.

# 8.1 Converting Instances Safely

## Problem

Your legacy code is weakly typed, relies on procedural patterns, and needs to be refactored.

## Solution

Here’s an interface and implementing classes that produce results we’re looking for:

[PRE0]

Here’s a method representing data returned in legacy nontyped instances:

[PRE1]

And this code processes the legacy collection:

[PRE2]

Here’s more modern code that returns a strongly typed collection:

[PRE3]

And this code processes the strongly typed collection:

[PRE4]

The `Main` methods call both the legacy and modern versions:

[PRE5]

## Discussion

The `as` and `is` operators appeared in C# 1; you’re probably aware of and/or have used them. To recap, the `is` operator tells whether an object’s type is the same as the type being matched. The `as` operator performs a conversion of a reference type object to a specified type. The `as` operator returns `null` if the converted instance isn’t the specified type. This example also demonstrates a recent C# addition that allows both type checking and conversion with the `is` operator.

Most of the code we write today uses generic collections and it’s increasingly unnecessary to use weakly typed collections. I’ll go out on a limb here and say that you might adopt a rule of thumb to use generic collections as a default, with the exception being when you can’t avoid using weakly typed collections. One important situation where you have to use weakly typed collections is when maintaining legacy code that already uses them. Generics weren’t added until C# 2, so you might encounter some old code with weakly typed collections. Another example is when you have a library you need or want to use that uses weakly typed collections. In practical terms, you might not want to rewrite that code because of the time and resources required—especially if it’s already tested and working well.

In the solution, the `GetWeakTypedSchedules` method returns an `ArrayList`, which is a weakly typed collection because it only operates on instances of type `Object`. The `ProcessLegacyCode` method calls `GetWeakTypedSchedules` and shows how to use the `as` and `is` operators.

The first `if` statement in the `foreach` loop uses the `is` operator to determine whether the object is an `IRoomSchedule`. If so, it uses a cast operator to get an `IRoomSchedule` instance and calls `GetSchedule`. You might ask why the `is` operator is necessary if we already know that the collection contains `IRoomSchedule` instances—why don’t we just go straight for the conversion? The problem is that there isn’t a guarantee of what the types in that collection are. What if a developer accidentally loads an object that isn’t `IRoomSchedule` into the collection? The `is` operator improves the reliability of the code.

An alternative to the `is` operator is the `as` operator. In the solution, the `schedule as IRoomSchedule` performs the conversion. If the result isn’t `null`, the object is an `IRoomSchedule`. This approach could perform better because an `is` operation both checks the type and still requires a conversion, whereas the `as` operator only requires a conversion and a `null` check.

The final `if` statement demonstrates the newer `is` operator syntax. It does both the type check and conversion, assigning the result to the `isRoomSchedule` variable. The `isRoomSchedule` variable is `null` if `schedule` wasn’t an `IRoomSchedule`, but since the `is` operator returned a `bool` result, we don’t need to do the extra `null` check.

The `GetStrongTypedSchedules` and `ProcessModernCode` show how you would probably want to write the code today. Notice how it has less ceremony because of the strong typing. Each class implements the same interface, and the collection is that interface, allowing you to write code that operates efficiently on every object.

This example also demonstrates that the new `is` operator can be useful in current code (not only legacy). In `ProcessModernCode`, even though all the objects implement `IRoomSchedule`, the `is` operator lets us check for `GoldSchedule` and do some extra processing.

# 8.2 Catching Filtered Exceptions

## Problem

You need to handle logic for the same exception type with different conditions.

## Solution

This is a demo class that throws exceptions:

[PRE6]

This `Main` method uses exception filters for clean processing:

[PRE7]

## Discussion

An interesting addition to C#, related to pattern matching, is the exception filter. As you know, `catch` blocks operate on the type of exception thrown. However, when the same exception type can be thrown for different reasons, sometimes it’s useful to be able to differentiate processing for each reason. While you could add `if` or `switch` statements in the `catch` block, filters offer a clean way to separate and simplify the different logic.

In the solution, we’re interested in filtering `ArgumentNullException`, depending on which parameter is `null`. The `ScheduleRoom` method checks each parameter and throws `ArgumentNullException` if either is `null`.

The `Main` method wraps the call to `ScheduleRoom` in a `try`/`catch` block. This example has two `catch` blocks, each of type `ArgumentNullException`. The difference between the two is the filter, specified by the `when` clause. The parameter to `when` is a `bool` expression. In the solution, the expression compares the `ParamName` to the parameter name it’s designed to handle.

# 8.3 Simplifying Switch Assignments

## Problem

You want to return a value, based on some criteria, but don’t want to return from every `switch` case.

## Solution

Here’s an interface and implementing classes that are results we’re looking for:

[PRE8]

This enum is used in upcoming logic:

[PRE9]

This class shows the `switch` statement and new `switch` expression:

[PRE10]

The `Main` method tests the code:

[PRE11]

## Discussion

The `switch` statement has been around since C# 1, and a recent addition is a `switch` expression. The main syntactic features of the `switch` expression are a shorthand notation and the ability to assign a result to a variable. If you think about all the times you’ve used a `switch` statement, you might have noticed that producing a value or new instance was a common theme. The `switch` expression streamlines that theme and improves upon it with pattern matching.

The solution has two examples: a `switch` statement and a `switch` expression. Both rely on the `ScheduleType` enum for criteria and produce an `IRoomSchedule` type, based on that criteria.

The `CreateStatement` method uses a `switch` statement, with `case` clauses for each member of the `ScheduleType` enum. Notice how it returns the value from the method and that it requires normal block body syntax (with curly braces).

The `CreateExpression` method uses the new `switch` expression. Notice that the method can be command body (with arrow), returning the expression. Instead of a parameter in parentheses after the `switch` keyword, the parameter precedes the `switch` keyword. Also, instead of `case` clauses, case pattern matches precede an arrow, with the result expression after the arrow. The default case is the discard pattern, `_`.

Whenever the parameter matches the case pattern, the `switch` expression returns the result. In the solution, the patterns are the values of the `ScheduleType` enum. The result of the `switch` expression is the result of the method because the command syntax of the method specifies the `switch` expression.

If you have a use case where there’s logic to process for each case, but don’t need to return, a classic `switch` statement might make more sense. However, if you can use pattern matching and need to return a value, the `switch` expression can be an excellent choice.

# 8.4 Switching on Property Values

## Problem

You need business rules based on strongly typed class properties.

## Solution

Here’s a class with properties that we need to evaluate:

[PRE12]

This enum is the result of the evaluation:

[PRE13]

This method gets the data we need:

[PRE14]

This method uses that data and returns an enum, based on matching pattern:

[PRE15]

The `Main` method drives the program:

[PRE16]

## Discussion

In the past, a `switch` statement matched cases with the value of a single parameter. Now, you can do parameter matching based on the values of properties in an object.

The solution uses an instance of the `Room` class as the parameter to a `switch` expression in the `AssignRoom` method. The pattern is an object with the properties of `Room` and the values to match. The result returned is based on which pattern the properties of the parameter match.

The goal of the program is to find an available room for a customer. The purpose of `AssignRoom` is to return the first room associated with a specific schedule type. That’s why `AssignRoom` compares `roomType` and `scheduleType`, returning if they match.

Property pattern matching is a nice approach because it’s easy to read. This potentially translates into more maintainable code. One trade-off is that it can be verbose if you’re matching a lot of properties. The next recipe offers shorter syntax.

## See Also

[Recipe 8.5, “Switching on Tuples”](#switching_on_tuples)

# 8.5 Switching on Tuples

## Problem

You need business rules and prefer shorter syntax.

## Solution

This class is interesting because it has a deconstructor:

[PRE17]

Here’s an enum that this program will produce:

[PRE18]

Here’s the data the program will work with:

[PRE19]

And this method uses the tuple returned from the class deconstructor to determine which enum to return:

[PRE20]

The `Main` method drives the program:

[PRE21]

## Discussion

Throughout this book, you’ve seen how useful tuples are for situations where you want to manage a set of values without all the ceremony of a custom type. The quick syntax of tuples makes them ideal for simple pattern matching.

In this example, we do have a custom type, `Room`. Notice that `Room` has a custom deconstructor, which we’ll use in this solution. The `GetRooms` method returns a `List<Room>`. `AssignRooms` uses that collection. However, because of the deconstructor, we can use each room as a `switch` expression parameter, which is smart enough to use the deconstructor to produce a tuple for pattern matching.

Except for using tuples through a deconstructor, this demo is the same as [Recipe 8.4](#switching_on_property_values). In this example, the tuple offered a shorter syntax. The property pattern is more verbose but easier to read. One consideration is that if you’re matching scalar values, like `bool` or `int`, the property pattern documents better. If you’re matching strings or enums, a tuple might provide the best of both worlds in terms of readability and shorter syntax. Because the choice between the two approaches is situational, it’s best to evaluate trade-offs in each situation to see what makes sense to you.

## See Also

[Recipe 8.4, “Switching on Property Values”](#switching_on_property_values)

# 8.6 Switching on Position

## Problem

You need business rules based on values but don’t want to create a new single-use class.

## Solution

This enum is the result we’ll look for:

[PRE22]

Here’s a class used for specifying decision criteria:

[PRE23]

These methods simulate getting data from two sources:

[PRE24]

This method joins those data sources to produce a list of tuples:

[PRE25]

This method shows the business logic based on positional pattern matching:

[PRE26]

The `Main` method drives the process:

[PRE27]

## Discussion

The solution here is similar to [Recipe 8.5](#switching_on_tuples), in that it also uses tuples for pattern matching. This solution differs in that it explores the situation where you have two different sources of data, shown in `GetHotel1Rooms` and `GetHotel2Rooms`, simulating what would normally be database queries. This can happen when companies merge or form partnerships and their data is similar but not entirely alike.

The `GetRooms` method shows how to use the LINQ `Union` operator to combine the two lists. Rather than create a new type for the combination of values we need, the method builds a collection of tuples.

When `AssignRooms` calls `GetRooms`, you don’t need a deconstructor on an object because you’re already working with tuples. This is a useful technique if you’re working with third-party types where you can’t modify their members.

Inside of `AssignRoom`, the `switch` expression uses tuples for the match. What’s immediately noticeable here is the first parameter, representing the `Room.Number` property—there’s a discard symbol for each pattern. Clearly, this could have been omitted in `GetRooms`, but I wrote it that way to make a couple of points: value positions must match and every value is required.

Tuple patterns require values to be in the proper positions (e.g., you can’t swap `Number` and `Size`). The position of each pattern value must match the corresponding tuple position. In contrast, property patterns can be in any order and differ between cases.

For tuples, you must include a value for each position of the tuple. Therefore, even if you don’t use a position in a pattern, you must at least specify the discard parameter. Property patterns don’t have this restriction, allowing you to add or ignore whatever properties you want for the pattern.

## See Also

[Recipe 8.5, “Switching on Tuples”](#switching_on_tuples)

# 8.7 Switching on Value Ranges

## Problem

Your business rules are continuous, rather than discrete.

## Solution

Here’s an interface, as well as implementing classes that are results we’re looking for:

[PRE28]

This method uses relational pattern matching to produce results:

[PRE29]

The `Main` method drives the process:

[PRE30]

## Discussion

Previous sections of this chapter explored pattern matching based on discrete values. The pattern had to be exact to match. However, there are a lot of situations where values are continuous, rather than discrete. An example of this is the solution in this section, where hotel customers could have a range of points in their accounts.

In the solution, customers with points from 0 to 4,999 are Bronze. Those with points from 5,000 to 19,999 are Silver. Those with 20,000 points or more are Gold. The `SilverPoints` and `GoldPoints` constants in the solution define the boundaries.

The `Main` method asks how many points a customer has and passes that value to `GetSchedule`. This value can vary, depending on how many times a person booked a room or used other hotel services. Because of this, `GetSchedule` uses a `switch` expression based on those points. Instead of using a discrete pattern for the match, `GetSchedule` uses relational operators.

The first pattern asks if `points` is equal to or higher than `GoldPoints`. If not, `points` must be less, and the code checks to see if the points are equal to or higher than `SilverPoints`. Since we already evaluated the `GoldPoints` case, it follows that the range is between `SilverPoints` and less than `GoldPoints`. The final case, less than `SilverPoints`, documents the meaning of Bronze, but you could have easily replaced that with a discard pattern because the other two cases handled every other possibility, and Bronze is all that’s left.

# 8.8 Switching with Complex Conditions

## Problem

Your business rules are multiconditional.

## Solution

Here’s an interface, as well as implementing classes that are results we’re looking for:

[PRE31]

This class describes the criteria to use:

[PRE32]

This method generates simulated data with various values to exercise our logic:

[PRE33]

Here’s a method using complex logic in a `switch` expression:

[PRE34]

The `Main` method iterates through results:

[PRE35]

## Discussion

Sometimes conditions are so complex that the techniques shown in earlier sections of this chapter are inadequate for solving the problem. An example is the solution in this section that requires multiple conditions involving more than one property. Here, we use a `switch` expression with `when` clauses to specify matches.

This scenario is based on a `Customer` type, indicating number of points and whether the customer has a free upgrade. The free upgrade could have been from a contest or hotel promotion activity. When scheduling a room, we want to make sure that the customer gets a room commensurate with their point level. Additionally, if they have the free upgrade option, they receive a room that is upgraded to the next higher level. For simplicity, we’re conveniently ignoring whether Gold has a free upgrade.

The `GetSchedule` method operates on an instance of `Customer`. Both the cases for Gold and Silver result in a room at that level. Additionally, the `||` operator says that if `customer` is at the next lower level, but `HasFreeUpgrade` is `true`, then the result is a room at this higher level.

Using logic like this can get complex fast. Notice the use of newlines and other spacing to add symmetry and consistency to reading the result.

While this technique can help when the logic is a bit more complex than a discrete pattern match, you might want to consider a threshold where using `if` statements might be a better implementation. One consideration is maintenance, because breaking each piece of the logic out can help with debugging, whereas a single expression with multiple conditions might not be immediately obvious.

# 8.9 Using Logical Conditions

## Problem

You want multiconditional logic to be more readable.

## Solution

Here’s a class to use as criteria:

[PRE36]

This method simulates a data source:

[PRE37]

This method implements business rules with conditional logic in a `switch` expression:

[PRE38]

The `Main` method drives this process:

[PRE39]

## Discussion

[Recipe 8.8](#switching_with_complex_conditions) described how to add complex logic to `switch` expressions. By complex, I’m referring to multiple conditions involving two or more properties. This contrasts with simple pattern matching for previous sections of this chapter that used property and tuple patterns. Somewhere in between these contrasting approaches of simple and complex is a moderate approach where you need logic isolated within individual properties.

The properties of interest in this solution are the `Points` and `Month` of the `Customer` class. Similar to earlier sections, the `Points` property contributes to receiving a room for a customer that has at least a certain number of points. The other condition, `Month`, is the month when the customer wants to book the room. Because of seasonal supply and demand, some months leave the hotel with more open rooms. Therefore, this application provides incentives, based on points, for customers to book rooms in the months with more open rooms.

In the solution, you can see that there are `GoldPoints` and `SilverPoints` constants to tell which level a customer is. Also, there are constants for `May`, `Sep`, and `Dec`—the busy months. The logic will be to give a discount in the months that are not busy.

The pattern for the `switch` expression in `GetDiscount` matches on two properties: `Points` and `Month`. Notice how this code doesn’t rely on an object deconstructor and the original parameter is a class, rather than a tuple. `GetDiscount` creates an inline tuple for the `switch` expression.

The pattern itself relies on relational operators for `Points`, as in [Recipe 8.7](#switching_on_value_ranges).

The `Month` pattern uses the new C# 9 logical operators: `and`, `not`, and `or`. The first expression ensures the customer receives a discount during the winter months, between `Sep` and `May`, except for `Dec`. The second pattern says that a Gold customer still gets a discount in `Dec`, except that it’s 10% instead of 15%.

The last pattern is logically equivalent to the first and uses DeMorgan’s Theorem. That is, it negates the whole result and swaps `and` with `or`. Because the last example applies `not` to the entire expression, it uses parentheses. In the first pattern, `not` applied to `Dec` only.

## See Also

[Recipe 8.7, “Switching on Value Ranges”](#switching_on_value_ranges)

[Recipe 8.8, “Switching with Complex Conditions”](#switching_with_complex_conditions)

# 8.10 Switching on Type

## Problem

You need the type of an object for decision making.

## Solution

Here’s an interface, as well as implementing classes that are results we’re looking for:

[PRE40]

The following types represent criteria:

[PRE41]

This method simulates a data source:

[PRE42]

Here’s a method that implements logic based on type pattern matching:

[PRE43]

The `Main` method iterates through the data to exercise the pattern matching logic:

[PRE44]

## Discussion

It used to be that the only way to make a decision on type was to either use `if` statements or convert the object’s type to a `string` and use a `switch` statement with `string` cases. A popular ask for C# over the years was to allow a `switch` statement with type cases, and now we finally have it.

The solution has a set of classes for `GoldCustomer`, `SilverCustomer`, and `BronzeCustomer`, each deriving from `Customer`. Our goal in this program is to schedule a room, based on the matching class type.

The `GetSchedule` method does the scheduling by accepting an object of type `Customer`, and the `switch` expression has a pattern for each of the classes that derive from `Customer`. All you need to do is specify the name of each class and the `switch` expression matches based on the object type.