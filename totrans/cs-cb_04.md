# Chapter 4\. Querying with LINQ

LINQ has been around since C# 3\. It gives developers a means to query data sources, using syntax with accents of SQL. Because LINQ is part of the language, you experience features like syntax highlighting and IntelliSense in IDEs.

LINQ is popularly known as a tool for querying databases, with the goal of reducing what is called *impedance mismatch*, which is the difference between database representation of data and C# objects. Really, we can build LINQ providers for any data technology. In fact, the author wrote an open source provider for the Twitter API named [LINQ to Twitter](https://oreil.ly/1YEZ8).

The examples in this chapter take a different approach. Instead of an external data source, they use a provider that specifically focuses on in-memory data sources referred to as LINQ to Objects. While any in-memory data manipulation can be performed with C# loops and imperative logic, using LINQ instead can often simplify the code because of its declarative nature—specifying what to do rather than how to do it. Each section has a unique representation of one or more entities (objects to be queried) and an `InMemoryContext` that sets up the in-memory data to be queried.

A couple of recipes in this chapter are simple, such as transforming object shape and simplifying queries. However, there are important points to be made that also clarify and simplify your code.

Pulling together code from different data sources can result in confusing code. The sections on joins, left joins, and grouping describe how you can simplify these scenarios. There’s also a related section for operating on sets.

A huge security problem with search forms and queries appears when developers build their queries with concatenated strings. While that might sound like a quick and easy solution, the cost is often too high. This chapter has a couple of sections that show how LINQ deferred execution lets you build queries dynamically. Another section explains an important technique for search queries and how they give you the ability to use expression trees for dynamic clause generation.

# 4.1 Transforming Object Shape

## Problem

You want data in a custom shape that differs from the original data source.

## Solution

Here’s the entity to reshape:

[PRE0]

This is the data source:

[PRE1]

This code performs the projection that reshapes the data:

[PRE2]

## Discussion

Transforming object shape is referred to as a *projection* in LINQ. A few common reasons you might want to do this is to create lookup lists, create a view or view model object, or translate data transfer objects (DTOs) to something your app works with better.

When doing database queries using LINQ to Entities (a different provider for databases), or consuming DTOs, data often arrives in a format representing the original data source. However, if you want to work with domain data or bind to UIs, the pure data representation doesn’t have the right shape. Moreover, data representation often has attributes and semantics of the object-relational model (ORM) or data access library. Some developers try to bind these data objects to their UI because they don’t want to create a new object type. While that’s understandable, because no one wants to do more work than is necessary, problems occur because UI code often requires a different shape of the data and requires its own validation and attributes. So, the problem here is that you’re using one object for two different purposes. Ideally, an object should have a single responsibility, and mixing it up like this often results in confusing code that’s not as easy to maintain.

Another scenario that the solution demonstrates is the case where you only want a lookup list, with an ID and displayable value. This is useful when populating UI elements such as checkbox lists, radio button groups, combo boxes, or dropdowns. Querying entire entities is wasteful and slow (in the case of an out-of-process or cross-network database connection) when you only need the ID and something to display to the user.

The `Main` method of the solution demonstrates this. It queries the `SalesPeople` property of `InMemoryContext`, which is a list of `SalesPerson`, and the `select` clause re-shapes the result into a tuple of `ID` and `Name`.

###### Note

The `select` clause in the solution uses a tuple. However, you could project (only the requested fields) into an anonymous type, a `SalesPerson` type, or a new custom type.

Although this was an in-memory operation, the benefit of this technique comes when querying a database with a library like LINQ to Entities. In that case, LINQ to Entities translates the LINQ query into a database query that only requests the fields specified in the select clause.

# 4.2 Joining Data

## Problem

You need to pull data from different sources into one record.

## Solution

Here are the entities to join:

[PRE3]

This is the data source:

[PRE4]

This is the code that joins the entities:

[PRE5]

## Discussion

LINQ joins are useful when data comes from more than one source. A company might have merged and you need to pull in data from each of their databases, you might be using a microservice architecture where the data comes from different services, or some of the data was created in-memory and you need to correlate it with database records.

Often, you can’t use an ID because if the data comes from different sources, they’ll never match anyway. The best you can hope for is that some of the fields line up. That said, if you have a single field that matches, that’s great. The `Main` method of the solution uses a composite key of `Region` and `ProductType`, relying on the value equality inherent in tuples.

###### Note

The `select` clause uses an anonymous type for a custom projection. Another example of shaping object data is discussed in [Recipe 4.1](#transforming_object_shape).

Even though this example uses a tuple for the composite key, you could use an anonymous type for the same results. The tuple uses slightly less syntax.

## See Also

[Recipe 4.1, “Transforming Object Shape”](#transforming_object_shape)

# 4.3 Performing Left Joins

## Problem

You need a join on two data sources, but one of those data sources doesn’t have a matching record.

## Solution

Here are the entities to perform a left join with:

[PRE6]

This is the data source:

[PRE7]

The following code performs the left join operation:

[PRE8]

And here’s the output:

[PRE9]

## Discussion

This solution is similar to the `join`, discussed in [Recipe 4.3](#performing_left_joins). The difference is in the LINQ query in the `Main` method. Notice the `into prodPersonTemp` clause. This is a temporary holder for the joined data. The second `from` clause (below `into`) queries `prodPersonTemp.DefaultIfEmpty()`.

The `DefaultIfEmpty()` causes the left join, where the `prodPerson` range variable receives all of the product objects and only the matching person objects.

The first `from` clause specifies the left side of the query, `Products`. The `join` clause specifies the right side of the query, `SalesPeople`, which might not have matching values.

Notice how the `select` clause checks `prodPerson?.Name` for `null` and replaces it with `(none)`. This ensures the output indicates that there wasn’t a match, rather than relying on later code to check for null.

Demonstrating left join results in the solution output. Notice that output for Product 1 and Product 4 have a Person entry. However, there wasn’t a matching Person, showing as `(none)`, for Products 2 and 3.

# 4.4 Grouping Data

## Problem

You need to aggregate data into custom groups.

## Solution

Here’s the entity to group:

[PRE10]

This is the data source:

[PRE11]

The following code groups the data:

[PRE12]

## Discussion

Grouping is useful when you need a hierarchy of data. It creates a parent/children relationship between data where the parent is the main category and the children are objects (representing data records) in that category.

In the solution, each `SalesPerson` has a `Region` property, whose values are repeated in the `InMemoryContext` data source. This helps show how multiple `SalesPerson` entities can be grouped into a single region.

In the `Main` method query, there’s a `group by` clause, specifying the range variable, `person`, to group and the key, `Region`, to group by. The `personGroup` holds the result. In this example, the `select` clause uses the entire `personGroup`, rather than doing a custom projection.

Inside of `salesPeopleByRegion` is a set of top-level objects, representing each group. Each of those groups has a collection of objects belonging to that group, like this:

[PRE13]

###### Note

LINQ providers targeting databases, such as LINQ to Entities for SQL Server, return `IQueryable<T>`, for nonmaterialized queries. Materialization occurs when you use an operator, such as `Count()` or `ToList()`, that actually executes the query and returns an `int` or `List<T>`, respectively. In contrast, the nonmaterialized type returned by LINQ to Objects is `IEnumerable<T>`.

The `foreach` loop demonstrates this group structure and how it could be used. At the top level, each object has a `Key` property. Because the original query was by `Region`, that key will have the name of the `Region`.

The nested `foreach` loop iterates on the group, reading each `SalesPerson` instance in that group. You can see where it prints out the `Name` of each `SalesPerson` instance in that group.

# 4.5 Building Incremental Queries

## Problem

You need to customize a query based on a user’s search criteria but don’t want to concatenate strings.

## Solution

This is the type to query:

[PRE14]

Here’s the data source:

[PRE15]

This code builds a dynamic query:

[PRE16]

## Discussion

One of the worst things a developer can do from a security perspective is to build a concatenated string from user input to send as a SQL statement to a database. The problem is that string concatenation allows the user’s input to be interpreted as part of the query. In most cases, people just want to perform a search. However, there are malicious users who intentionally probe systems for this type of vulnerability. They don’t have to be professional hackers as there are plenty of novices (often referred to as *script kiddies*) who want to practice and have fun. In the worst case, hackers can access private or proprietary information or even take over a machine. Once into one machine on a network, the hacker is on the inside and can monkey bar into other computers and take over your network. This particular problem is called a *SQL injection attack* and this section explains how to avoid it.

###### Note

From a security point of view, no computer is theoretically 100% secure because there’s always a level of effort, either physical or virtual, where a computer can be broken into. In practice, security efforts can grow to a point that they become prohibitively expensive to build, purchase, and maintain. Your goal is to perform a threat assessment of a system (outside the scope of this book) that’s strong enough to deter potential hackers. In most cases, having not been able to perform the typical attacks, like SQL injection, a hacker will assess their own costs of attacking your system and move on to a different system that is less time consuming or expensive. This section offers a low-cost option to solve a high-cost security disaster.

The scenario for this section imagines a situation where the user can perform a search. They fill in the data and the application dynamically builds a query, based on the criteria the user entered.

In the solution, the `Program` class has a method named `GetCriteriaFromUser`. The purpose of this method is to ask for a matching value for each field inside of `SalesPerson`. This becomes the criteria for building a dynamic query. Any fields left blank aren’t included in the final query.

The `QuerySalesPeople` method starts with a LINQ query for `ctx.SalesPeople`. However, notice that this isn’t in parentheses or calling the `ToList` operator, like previous sections. Calling `ToList` would have materialized the query, causing it to execute. However, we aren’t doing that here—the code is just building a query. That’s why the `salesPersonQuery` has the `IEnumerable<SalesPerson>` type, indicating that it’s a LINQ to Objects result, rather than a `List<SalesPerson>` we would have gotten back via a call to `ToList`.

###### Note

This recipe takes advantage of a feature of LINQ, known as *deferred query execution*, which allows you to build the query that won’t execute until you tell it to. In addition to facilitating dynamic query construction, deferred execution is also efficient because there’s only a single query sent to the database, rather than each time the algorithm calls a specific LINQ operator.

With the `salesPersonQuery` reference, the code checks each `SalesPerson` field for a value. If the user did enter a value for that field, the code uses a `Where` operator to check for equality with what the user entered.

###### Note

You’ve seen LINQ queries with language syntax in previous sections. However, this section takes advantage of another way to use LINQ via a fluent interface, called *method syntax*. This is much like the builder pattern you learned about in [Recipe 1.10](ch01.xhtml#constructing_objects_with_complex_configuration).

So far, the only thing that has happened is that we’ve dynamically built a LINQ query and, because of deferred execution, the query hasn’t run yet. Finally, the code calls `ToList` on `salesPersonQuery`, materializing the query. As the return type of this method indicates, this returns a `List<SalesPerson>`.

Now, the algorithm has built and executed a dynamic query, protected from SQL injection attack. This protection comes from the fact that the LINQ provider always parameterizes user input so it will be treated as parameter data, rather than as part of the query. As a side benefit, you also have a method with strongly typed code, where you don’t have to worry about inadvertent and hard-to-find typos.

## See Also

[Recipe 1.10, “Constructing Objects with Complex Configuration”](ch01.xhtml#constructing_objects_with_complex_configuration)

# 4.6 Querying Distinct Objects

## Problem

You have a list of objects with duplicates and need to transform that into a distinct list of unique objects.

## Solution

Here’s an object that won’t support distinct queries:

[PRE17]

Here’s how to fix that object to support distinct queries:

[PRE18]

Here’s the data source:

[PRE19]

This code filters by distinct objects:

[PRE20]

## Discussion

Sometimes you have a list of entities with duplicates, either because of some application processing or the type of database query that results in duplicates. Often, you need a list of unique objects. For instance, you’re materializing into a `Dictionary` collection that doesn’t allow duplicates.

The LINQ `Distinct` operator helps get a list of unique objects. At first glance, this is easy, as shown in the first query of the `Main` method that uses the `Distinct()` operator. Notice that it doesn’t have parameters. However, an inspection of the results shows that you still have the same duplicates in the data that you started with.

The problem, and subsequent solution, might not be immediately obvious because it relies on combining a few different C# concepts. First, think about how `Distinct` should be able to tell the difference between objects—it has to perform a comparison. Next, consider that the type of `SalesPerson` is class. That’s important because classes are reference types, which have reference equality. When `Distinct` does a reference comparison, no two object references are the same because each object has a unique reference. Finally, you need to write code to compare `SalesPerson` instances to see if they’re equal and tell `Distinct` about that code.

The `SalesPerson` class is a basic class with properties and doesn’t contain any syntax to indicate how to perform equality. In contrast, `SalesPersonComparer` implements `IEqualityComparer<SalesPerson>`. The `SalesPerson` class doesn’t work because it has reference equality. However the `SalesPersonComparer` class that implements `IEqualityComparer<SalesPerson>` compares properly because it has an `Equals` method. In this case, checking `ID` is sufficient to determine that instances are equal, assuming that each entity comes from the same data source with unique `ID` fields.

`SalesPersonComparer` knows how to compare `SalesPerson` instances, but that isn’t the end of the story because there isn’t anything tying it to the query. If you ran the first query in `Main` with `Distinct()` (no parameter), the results will still have duplicates. The problem is that `Distinct` doesn’t know how to compare the objects so it defaults to the instance type, `class`, which, as explained earlier, is a reference type.

The solution is to use the second query in `Main` that uses the call to `Distinct(new SalesPersonComparer())` (with parameter). This uses the `Distinct` operator’s overload with the `IEqualityComparer<T>` overload parameter. Since `SalesPerson​Com⁠parer` implements `IEqualityComparer<SalesPerson>`, this works.

## See Also

[Recipe 2.5, “Checking for Type Equality”](ch02.xhtml#checking_for_type_equality)

# 4.7 Simplifying Queries

## Problem

A query has become too complex and you need to make it more readable.

## Solution

Here’s the entity to query:

[PRE21]

This is the data source:

[PRE22]

The following shows how to simplify a query projection:

[PRE23]

## Discussion

Sometimes LINQ queries get complex. If the code is still hard to read, it’s also hard to maintain. One option is to go imperative and rewrite the query as a loop. Another is to use the `let` clause for simplification.

In the solution, the `Main` method has a query with a custom projection into an anonymous type. Sometimes queries are complex because they have subqueries, or other logic, inside of the projection. For example, look at `FullAddress`, being built in a `let` clause. Without that simplification, the code would have ended up inside the projection.

Another scenario you might face is when parsing object input from string. The example uses a `TryParse` in a `let` clause, which is impossible to put in the projection. This is a little tricky because the `out` parameter, `TotalSales`, is outside of the query. We ignore the results of `TryParse` but can now assign `TotalSales` in the projection.

# 4.8 Operating on Sets

## Problem

You want to combine two sets of objects without duplication.

## Solution

Here’s the entity to query:

[PRE24]

Here’s the data source:

[PRE25]

This code shows how to perform set operations:

[PRE26]

## Discussion

In [Recipe 4.2](#joining_data), we discussed the concept of joining data from two separate data sources. The examples operate in that same spirit and show different manipulations, based on sets.

The first method, `DoUnion`, gets two sets of data, intentionally filtering by `ID` to ensure overlap. From the reference of the first data source, the code calls the `Union` operator with the second data source as the parameter. This results in a set of data from both data sources, including duplicates.

The `DoExcept` method is similar to `DoUnion` but uses the `Except` operator. This results in a set of all the objects in the first data source. However, any objects in the second data source, even if they were in the first, won’t appear in the results.

Finally, `DoIntersect` is similar in structure to `DoUnion` and `DoExcept`. However, it queries objects that are only in both data sources. If any object is in one data source, but not the other, it won’t appear in the result. This operation is called *difference in set theory*.

LINQ has many standard operators that, just like the set operators, are very powerful. Before performing any complex operation in a LINQ query, it’s good practice to review standard operators to see if something exists that will simplify your task.

## See Also

[Recipe 4.2, “Joining Data”](#joining_data)

[Recipe 4.3, “Performing Left Joins”](#performing_left_joins)

# 4.9 Building a Query Filter with Expression Trees

## Problem

The LINQ `where` clause combines via `AND` conditions, but you need a dynamic `where` that works as an `OR` condition.

## Solution

Here’s the entity to query:

[PRE27]

This is the data source:

[PRE28]

Here’s an extension method for a filtered `OR` operation:

[PRE29]

Here’s the code that consumes the new extension method:

[PRE30]

## Discussion

[Recipe 4.5](#building_incremental_queries) showed the power of dynamic queries in LINQ. However, that isn’t the end of what you can do. With expression trees, you can leverage LINQ for any type of query. If the standard operators don’t provide something you need, you can use expression trees. This section does just that, showing how to use expression trees to run a dynamic `WhereOr` operation.

The motivation for `WhereOr` comes from the fact that the standard `Where` operator combines in an `AND` comparison. In [Recipe 4.5](#building_incremental_queries), all of those `Where` operators had an implicit `AND` relationship between them. This means that a given entity must have a value equal to each of the fields (that the user specified in the criteria) to get a match. With the `WhereOr` in this section, all of the fields have an `OR` relationship, and a match on only one of the fields is necessary for inclusion in results.

In the solution, the `GetCriteriaFromUser` method gets the values for each `SalesPerson` property. `QuerySalesPeople` starts a query for deferred execution, as explained in [Recipe 4.5](#building_incremental_queries), and builds a `Dictionary<string, string>` of filters.

The `CookbookExtensions` class has the `WhereOr` extension method that accepts the filters. The high-level description of what `WhereOr` is trying to accomplish comes from the fact that it needs to return an `IEnumerable<SalesPerson>` for the caller to complete a LINQ query.

First, go to the bottom of `WhereOr` and notice that it returns the query with the `Where` operator and has a parameter named `compiledQuery`. Remember that the LINQ `Where` operator takes a C# lambda expression with a parameter and a predicate. We want a filter that returns an object if any one field of an object matches, based on the input criteria. Therefore, `compiledQuery` must evaluate to a lambda of the following form:

[PRE31]

That’s a lambda with `OR` operators for each value in the `Dictionary<string, string> criteria` parameter. To get from the top of this algorithm to the bottom, we need to build an expression tree that evaluates to this form of lambda. [Figure 4-1](#where_or) illustrates what this code does.

![Building a Where expression with clauses separated by OR operators](Images/cscb_0401.png)

###### Figure 4-1\. Building a `Where` expression with clauses separated by `OR` operators

[Figure 4-1](#where_or) shows the expression tree that the solution creates. Here, we assume that the user wants to query four values: `City`, `Name`, `ProductType`, and `Region`. Expression trees read depth-first, from left to right, where each box represents a node. Therefore, LINQ follows the tree down the left side until it finds a leaf node, which is the `City` expression. Then it moves back up the tree to find the `OR`, moves to the right and finds the `Name` expression, and builds the `OR` expression. So far, LINQ has built the following clause:

[PRE32]

LINQ continues reading the expression tree up and to the right until it finally builds the following clause:

[PRE33]

Back to the solution code, the first thing `WhereOr` does is create a `ParameterExpression`. This is the `person` parameter in the lambda. It’s the parameter to every comparison expression because it represents the `TParameter`, which is an instance of `SalesPerson` in this example.

###### Note

This example is called the `ParameterExpression` `person`. However, if this is a generic reusable extension method, you might give it a more general name, like `parameterTerm` because `TParameter` could be any type. The choice of `person` in this example is there to clarify that the `ParameterExpression` represents a `SalesPerson` instance in this example.

The `Expression` `accumulatorExpr`, as its name suggests, gathers all of the clauses for the lambda body.

The `foreach` statement loops through the `Dictionary` collection, which returns `KeyValuePair` instances, which have `Key` and `Value` properties. As shown in the `QuerySalesPeople` method, the `Key` property is the name of the `SalesPerson` property, and the `Value` property is what the user entered.

For each clause of the lambda, the left-hand side is a reference to the property on the `SalesPerson` instance (e.g., `person.Name`). To create that, the code instantiates the `paramMbr` using the `paramExpr` (which is `person`). That becomes a parameter of `leftExpr`. The `rightExpr` expression is a constant that holds the value to compare and its type. Then we need to complete the expression with an `Equals` expression for the left and right expressions (`leftExpr` and `rightExpr`, respectively).

Finally, we need to `OR` that expression with any others. The first time through the `foreach` loop, `accumulatorExpr` will be `null`, so we just assign the first expression. On subsequent expressions, we use an `OR` expression to append the new `Equals` expression to `accumulatorExpr`.

After iterating through each input field, we form the final `LambdaExpression` that adds the parameter that was used in the left side of each `Equals` expression. Notice that the result is an `Expression<Func<TParameter, bool>>`, which has a parameter type matching the lambda delegate type for the original query, which is `Func<SalesPerson, bool>`.

We now have a dynamically built expression tree ready to convert into runnable code, which is a task for the `Expression.Compile` method. This gives us a compiled lambda that we can pass to the `Where` clause.

The calling code receives the `IEnumerable<SalesPerson>` from the `WhereOr` method and materializes the query with a call to `ToList`. This produces a list of `SalesPerson` objects that match at least one of the user’s specified criteria.

## See Also

[Recipe 4.5, “Building Incremental Queries”](#building_incremental_queries)

# 4.10 Querying in Parallel

## Problem

You want to improve performance, and your query could benefit from multithreading.

## Solution

Here’s the entity to query:

[PRE34]

This is the data source:

[PRE35]

This code shows how to perform a parallel query:

[PRE36]

## Discussion

This section considers queries that can benefit from concurrency. Imagine you have a LINQ to Objects query, where the data is in memory. Perhaps work on each instance requires intensive processing, the code runs on a multithreaded/multicore CPU, and/or takes a nontrivial amount of time. Running the query in parallel might be an option.

The `Main` method performs a query, similar to any other query, except for the `AsParallel` operator on the data source. What this does is let LINQ figure out how to split up the work and operate on each range variable in parallel. [Figure 4-2](#plinq) illustrates what this query is doing.

![PLINQ runs members of a collection in parallel](Images/cscb_0402.png)

###### Figure 4-2\. PLINQ runs members of a collection in parallel

[Figure 4-2](#plinq) shows the `salesPeople` collection on the left. When the query runs, it takes multiple collection objects to process in parallel, indicated by the split arrows from `salesPeople` pointing to each instance of `SalesPerson`. After processing, the query combines the responses from processing each object into a new collection, named `result`.

###### Note

This example uses a LINQ technology known as Parallel LINQ (PLINQ). Behind the scenes, PLINQ evaluates the query for various runtime optimizations such as degree of parallelism. It’s even smart enough to figure out when running synchronously is faster than the overhead of starting new threads on a given machine.

This example also demonstrates another type of projection that uses a method to return an object. The assumption here is that the intensive processing occurs in `ProcessPerson`, which has a `Thread.Sleep` to simulate nontrivial processing.

In practice, you would want to do some testing to see if you’re really benefiting from parallelism. [Recipe 3.10](ch03.xhtml#measuring_performance) shows how to measure performance with the `System.Diagnostics.StopWatch` class. If successful, this could be an easy way to boost the performance of your application.

## See Also

[Recipe 3.10, “Measuring Performance”](ch03.xhtml#measuring_performance)