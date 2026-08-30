
# Phase 21 — `IEnumerable<T>` vs `IQueryable<T>`

This is one of the **most important LINQ concepts**.

If you understand only the syntax of LINQ but don't understand the difference between `IEnumerable<T>` and `IQueryable<T>`, you can write code that _looks_ correct but performs terribly against a database.

The core distinction is:

> **`IEnumerable<T>` executes LINQ as C# code over objects.**
> 
> **`IQueryable<T>` builds a query representation that a provider can translate and execute elsewhere, typically a database.**

---

# 1. The big picture

Think of LINQ as having two worlds:

```text
                    LINQ
                     │
          ┌──────────┴──────────┐
          │                     │
     IEnumerable<T>       IQueryable<T>
          │                     │
          ▼                     ▼
   LINQ-to-Objects       Query Provider
          │                     │
          ▼                     ▼
    C# delegates          Expression Trees
          │                     │
          ▼                     ▼
      .NET memory          Database / provider
```

For example:

```csharp
List<Employee> employees;
```

naturally gives you:

```csharp
IEnumerable<Employee>
```

while:

```csharp
DbSet<Employee> employees;
```

in Entity Framework Core behaves as:

```csharp
IQueryable<Employee>
```

---

# 2. `IEnumerable<T>`

`IEnumerable<T>` represents a sequence that can be **enumerated**.

Simplified:

```csharp
public interface IEnumerable<out T>
{
    IEnumerator<T> GetEnumerator();
}
```

The important idea is:

```text
"I have a sequence of T that I can iterate over."
```

For example:

```csharp
var numbers = new List<int>
{
    1, 2, 3, 4, 5
};
```

Then:

```csharp
IEnumerable<int> query = numbers;
```

You can do:

```csharp
var result = query
    .Where(x => x > 2)
    .Select(x => x * 10);
```

The operations happen against the objects in your process.

---

# 3. `IQueryable<T>`

`IQueryable<T>` extends `IEnumerable<T>`.

Conceptually:

```csharp
public interface IQueryable<out T> : IEnumerable<T>
{
    Expression Expression { get; }
    IQueryProvider Provider { get; }
}
```

This is a critical fact:

```text
IQueryable<T>
      ↓
is also IEnumerable<T>
```

But it has additional information describing **how the query should be interpreted**.

The important properties are:

```text
Expression
Provider
```

---

# 4. What is a query provider?

A query provider is the component responsible for taking the LINQ query representation and figuring out how to execute it.

For Entity Framework Core:

```text
LINQ
 ↓
Expression Tree
 ↓
EF Core Query Provider
 ↓
SQL
 ↓
Database
```

So:

```csharp
db.Employees.Where(e => e.Salary > 100000)
```

doesn't necessarily execute the C# lambda directly against every employee.

Instead, EF Core can inspect the expression and translate the operation into SQL.

---

# 5. The crucial difference: delegate vs expression tree

This is the technical heart of the subject.

Consider:

```csharp
employees.Where(e => e.Salary > 100000);
```

For `IEnumerable<T>`, the relevant overload accepts:

```csharp
Func<T, bool>
```

So the lambda becomes executable C# code.

Conceptually:

```text
Employee
   ↓
Func<Employee, bool>
   ↓
execute C# code
   ↓
true/false
```

For `IQueryable<T>`, the relevant overload accepts:

```csharp
Expression<Func<T, bool>>
```

Now the lambda is represented as an **expression tree**.

Conceptually:

```text
Employee.Salary > 100000
          ↓
Expression Tree
          ↓
Query Provider
          ↓
SQL
```

---

# 6. `Func<T, bool>` vs `Expression<Func<T, bool>>`

This distinction is extremely important.

### `Func<T, bool>`

```csharp
Func<Employee, bool> predicate =
    e => e.Salary > 100000;
```

Think:

> "Here is executable code."

### `Expression<Func<T, bool>>`

```csharp
Expression<Func<Employee, bool>> predicate =
    e => e.Salary > 100000;
```

Think:

> "Here is a representation of code that can be inspected."

The expression tree can conceptually look like:

```text
Lambda
 └── GreaterThan
      ├── MemberAccess
      │    └── Salary
      └── Constant
           └── 100000
```

A query provider can inspect this structure.

---

# 7. Why can't a database execute a C# delegate?

Suppose:

```csharp
Func<Employee, bool> predicate =
    e => e.Salary > 100000;
```

The database doesn't understand:

```text
Func<Employee, bool>
```

or arbitrary .NET machine code.

But a provider can inspect:

```csharp
Expression<Func<Employee, bool>>
```

and recognize:

```text
Salary > 100000
```

Then translate it:

```sql
WHERE Salary > 100000
```

That's the fundamental reason expression trees matter.

---

# 8. Example with a List

Consider:

```csharp
var employees = new List<Employee>
{
    new Employee { Name = "Alice", Salary = 90000 },
    new Employee { Name = "Bob", Salary = 120000 },
    new Employee { Name = "Charlie", Salary = 150000 }
};
```

Then:

```csharp
var result = employees
    .Where(e => e.Salary > 100000)
    .Select(e => e.Name);
```

Because `employees` is an in-memory collection:

```text
List<Employee>
      ↓
IEnumerable<Employee>
      ↓
Where
      ↓
Select
      ↓
C# executes against objects
```

No SQL exists.

---

# 9. Example with Entity Framework Core

Suppose:

```csharp
var result = db.Employees
    .Where(e => e.Salary > 100000)
    .Select(e => e.Name);
```

`db.Employees` is an `IQueryable<Employee>`.

Conceptually:

```text
DbSet<Employee>
       ↓
IQueryable<Employee>
       ↓
Where
       ↓
Select
       ↓
Expression Tree
       ↓
EF Core
       ↓
SQL
       ↓
Database
```

Potentially:

```sql
SELECT Name
FROM Employees
WHERE Salary > 100000;
```

The database does the filtering and projection.

---

# 10. This creates a huge performance difference

Imagine the database has:

```text
10,000,000 employees
```

You want:

```text
Salary > 100000
```

### Good query

```csharp
var result = db.Employees
    .Where(e => e.Salary > 100000)
    .Select(e => e.Name)
    .ToList();
```

Conceptually:

```text
Database
10,000,000 rows
       ↓
WHERE Salary > 100000
       ↓
maybe 200,000 rows
       ↓
SELECT Name
       ↓
send results to application
```

### Bad pattern

```csharp
var employees = db.Employees
    .ToList();

var result = employees
    .Where(e => e.Salary > 100000)
    .Select(e => e.Name)
    .ToList();
```

Now:

```text
Database
10,000,000 rows
       ↓
send all rows
       ↓
Application memory
       ↓
Where
       ↓
Select
```

The syntax looks similar.

The execution is radically different.

---

# 11. `ToList()` is a major boundary

This is one of the most important things to recognize:

```csharp
.ToList()
```

materializes the query.

Before:

```csharp
db.Employees
```

you have:

```text
IQueryable<Employee>
```

After:

```csharp
db.Employees.ToList()
```

you have:

```text
List<Employee>
```

Therefore:

```csharp
var result = db.Employees
    .ToList()
    .Where(e => e.Salary > 100000);
```

means:

```text
Database
   ↓
retrieve employees
   ↓
List<Employee>
   ↓
IEnumerable
   ↓
Where executes in memory
```

Whereas:

```csharp
var result = db.Employees
    .Where(e => e.Salary > 100000)
    .ToList();
```

means:

```text
IQueryable
   ↓
build query
   ↓
execute WHERE in database
   ↓
materialize results
```

**The location of `ToList()` matters.**

---

# 12. Materialization operators

Several LINQ operators force execution/materialization.

Common examples:

```csharp
ToList()
ToArray()
ToDictionary()
ToLookup()
Count()
Any()
First()
FirstOrDefault()
Single()
SingleOrDefault()
Sum()
Average()
Max()
Min()
```

For a database-backed `IQueryable`, these can cause the query to execute.

For example:

```csharp
var count = db.Employees.Count();
```

can become something like:

```sql
SELECT COUNT(*)
FROM Employees;
```

rather than retrieving every employee.

---

# 13. Deferred execution

Both `IEnumerable<T>` and `IQueryable<T>` commonly support **deferred execution**.

For example:

```csharp
var query = employees
    .Where(e => e.Salary > 100000);
```

At this point, the query generally hasn't been enumerated yet.

Then:

```csharp
foreach (var employee in query)
{
    ...
}
```

causes enumeration.

Likewise:

```csharp
var list = query.ToList();
```

forces execution.

But what happens during execution depends on the source:

```text
IEnumerable
→ execute C# over objects

IQueryable
→ provider executes translated query
```

---

# 14. The same LINQ syntax can mean different things

This is a crucial concept.

Consider:

```csharp
query.Where(x => x.Price > 100);
```

If:

```text
query : IEnumerable<Product>
```

then:

```text
C# predicate
```

If:

```text
query : IQueryable<Product>
```

then:

```text
Expression tree
```

So don't judge LINQ code only by its syntax.

Always ask:

> **What is the type of the source?**

---

# 15. The source determines the LINQ world

For example:

```csharp
IEnumerable<Employee> employees;
```

means:

```text
LINQ-to-Objects
```

while:

```csharp
IQueryable<Employee> employees;
```

means:

```text
provider-backed LINQ
```

This is why type awareness is so important.

---

# 16. `AsEnumerable()`

Now we introduce an extremely important operator:

```csharp
AsEnumerable()
```

Suppose:

```csharp
var query = db.Employees
    .Where(e => e.Salary > 100000);
```

This is:

```text
IQueryable<Employee>
```

If you write:

```csharp
var result = query
    .AsEnumerable()
    .Where(e => MyCustomMethod(e));
```

then the second `Where()` operates as LINQ-to-Objects.

Conceptually:

```text
IQueryable
    ↓
database filtering
    ↓
AsEnumerable()
    ↓
IEnumerable
    ↓
C# filtering
```

This is a deliberate boundary.

---

# 17. `AsEnumerable()` doesn't immediately fetch everything

This distinction matters.

```csharp
query.AsEnumerable()
```

doesn't necessarily mean:

> "Execute the entire query right now."

It changes the subsequent LINQ operators to use the `IEnumerable<T>` side.

Enumeration still causes execution.

So:

```csharp
var query = db.Employees
    .Where(e => e.Salary > 100000)
    .AsEnumerable()
    .Where(e => SomeCSharpMethod(e));
```

means:

```text
First Where
→ provider-side

AsEnumerable
→ switch to IEnumerable semantics

Second Where
→ client-side
```

That is very different from:

```csharp
db.Employees.ToList()
```

which explicitly materializes the current result.

---

# 18. `AsQueryable()`

There is also:

```csharp
AsQueryable()
```

But be careful.

If you have:

```csharp
var list = new List<Employee>();
```

then:

```csharp
var query = list.AsQueryable();
```

does **not magically turn the List into a database query**.

The data is still in memory.

You now have an `IQueryable<T>` abstraction over an in-memory source/provider.

So:

```text
AsQueryable()
≠
move data to database
```

It simply provides queryable semantics for that source.

---

# 19. The dangerous misconception

Never think:

```text
IQueryable<T> = database
```

Not necessarily.

Think:

```text
IQueryable<T>
=
a query abstraction backed by a query provider
```

Entity Framework Core happens to provide a database-backed implementation.

Other providers can exist.

---

# 20. `IEnumerable<T>` does not mean "slow"

Another misconception:

> "IEnumerable is slower than IQueryable."

That's not generally true.

`IEnumerable<T>` is often exactly what you want for:

- in-memory collections
    
- arrays
    
- lists
    
- objects already loaded
    
- custom C# transformations
    

For example:

```csharp
var numbers = Enumerable.Range(1, 1_000_000);

var result = numbers
    .Where(x => x % 2 == 0)
    .Select(x => x * x);
```

There is no database involved.

`IEnumerable<T>` is the correct abstraction.

---

# 21. `IQueryable<T>` does not automatically mean faster

Likewise:

> "`IQueryable<T>` is faster."

Not automatically.

You can write terrible database queries with `IQueryable<T>`.

For example:

```csharp
var result = db.Employees
    .Where(...)
    .Include(...)
    .Include(...)
    .Include(...)
    .ToList();
```

Depending on the model and generated SQL, you could create:

- huge joins
    
- duplicated rows
    
- excessive data transfer
    
- expensive query plans
    
- Cartesian explosions
    

`IQueryable<T>` gives you **query composition**, not a guarantee of good performance.

---

# 22. Query composition

One of the major advantages of `IQueryable<T>` is that you can keep composing the query.

```csharp
var query = db.Employees;

query = query.Where(e => e.IsActive);

query = query.Where(e => e.Salary > 100000);

query = query.OrderByDescending(e => e.Salary);

var result = query.ToList();
```

Conceptually:

```text
IQueryable
   ↓
Where
   ↓
Where
   ↓
OrderBy
   ↓
ToList
   ↓
provider executes ONE composed query
```

The provider can potentially optimize the whole query.

---

# 23. `IEnumerable` composition

The same syntax works in memory:

```csharp
IEnumerable<Employee> query = employees;

query = query.Where(e => e.IsActive);

query = query.Where(e => e.Salary > 100000);

query = query.OrderByDescending(e => e.Salary);

var result = query.ToList();
```

But:

```text
All operations
     ↓
C# execution
     ↓
Objects in memory
```

No translation happens.

---

# 24. `IQueryable` can only translate what the provider understands

This is a major limitation.

Suppose:

```csharp
bool IsSenior(Employee employee)
{
    return employee.Salary > 100000 &&
           employee.YearsOfExperience >= 5;
}
```

Then:

```csharp
db.Employees.Where(e => IsSenior(e));
```

may not be translatable by your provider.

Why?

Because the provider sees an expression referring to a custom C# method and needs to know how to turn that method into SQL.

A database doesn't know what:

```text
IsSenior()
```

means.

---

# 25. But the equivalent inline expression may translate

Instead:

```csharp
db.Employees.Where(e =>
    e.Salary > 100000 &&
    e.YearsOfExperience >= 5);
```

the provider can recognize:

```text
Salary > 100000
AND
YearsOfExperience >= 5
```

and translate it to SQL.

This gives us an important rule:

> **With `IQueryable<T>`, write expressions in terms the provider can translate.**

---

# 26. Client-side vs server-side evaluation

This distinction is fundamental in database LINQ.

### Server-side

```csharp
db.Employees
    .Where(e => e.Salary > 100000)
```

Potentially:

```text
C# → SQL → database
```

### Client-side

```csharp
db.Employees
    .AsEnumerable()
    .Where(e => MyCustomMethod(e))
```

means:

```text
database
   ↓
fetch data
   ↓
.NET objects
   ↓
MyCustomMethod()
```

You should consciously know where computation happens.

---

# 27. Projection is especially important

Suppose you need only:

```text
Name
Salary
```

Don't necessarily load entire entities.

Prefer:

```csharp
var result = db.Employees
    .Where(e => e.IsActive)
    .Select(e => new
    {
        e.Name,
        e.Salary
    })
    .ToList();
```

Conceptually:

```sql
SELECT Name, Salary
FROM Employees
WHERE IsActive = 1;
```

Instead of:

```text
SELECT every column
FROM Employees
```

Projection can substantially reduce:

- network transfer
    
- memory usage
    
- materialization cost
    
- unnecessary entity tracking
    

---

# 28. `IEnumerable` projection

The same:

```csharp
employees
    .Select(e => new
    {
        e.Name,
        e.Salary
    });
```

still works, but the projection occurs in your application:

```text
Employee objects
      ↓
C# Select
      ↓
anonymous objects
```

Whereas with `IQueryable`:

```text
Expression Tree
      ↓
SQL SELECT
      ↓
database returns only required columns
```

That is a huge conceptual difference.

---

# 29. `IQueryable<T>` and expression trees

This connects directly to the architecture:

```text
Lambda
   ↓
Expression<Func<T,...>>
   ↓
Expression Tree
   ↓
Query Provider
   ↓
Translation
```

For example:

```csharp
db.Employees.Where(e => e.Salary > 100000)
```

can be represented conceptually as:

```text
Call: Where
 │
 ├── Source: Employees
 │
 └── Predicate
      │
      └── Lambda
           │
           └── GreaterThan
                ├── e.Salary
                └── 100000
```

The provider walks this structure.

This is why `IQueryable<T>` is deeply connected to **expression trees**.

---

# 30. `IEnumerable<T>` uses delegates

For:

```csharp
employees.Where(e => e.Salary > 100000)
```

the lambda becomes something like:

```csharp
Func<Employee, bool>
```

Conceptually:

```text
Employee
   ↓
execute delegate
   ↓
true / false
```

There is no need for a provider to understand the internals.

.NET simply executes the function.

---

# 31. A powerful comparison

|Feature|`IEnumerable<T>`|`IQueryable<T>`|
|---|---|---|
|Typical source|List, Array|EF Core `DbSet`|
|Execution|In memory|Provider|
|Lambda representation|Delegate|Expression tree|
|Query translation|No|Yes, if provider supports it|
|Database-aware|No|Potentially|
|LINQ-to-Objects|Yes|Can eventually become yes|
|Deferred execution|Often|Often|
|`ToList()`|Materializes sequence|Executes provider query + materializes|
|Custom C# methods|Fine|May not translate|
|Main concern|CPU/memory|Query translation/database cost|

---

# 32. The boundary operators

You should memorize these:

```text
IQueryable
    │
    │ AsEnumerable()
    ▼
IEnumerable
```

and:

```text
IEnumerable
    │
    │ AsQueryable()
    ▼
IQueryable
```

But remember:

```text
AsEnumerable()
→ changes LINQ operator binding

ToList()
→ materializes

AsQueryable()
→ changes abstraction/provider semantics
  but does not magically create a database
```

---

# 33. A subtle overload-resolution issue

This is one of the reasons `AsEnumerable()` is useful.

Suppose:

```csharp
IQueryable<Employee> query;
```

Then:

```csharp
query.Where(e => ...)
```

resolves to the `Queryable.Where()` family.

After:

```csharp
query.AsEnumerable()
```

you have:

```csharp
IEnumerable<Employee>
```

So:

```csharp
query
    .AsEnumerable()
    .Where(e => ...)
```

resolves to:

```csharp
Enumerable.Where()
```

This changes:

```text
Expression<Func<T,bool>>
```

into:

```text
Func<T,bool>
```

That is the technical reason `AsEnumerable()` creates the client-side boundary.

---

# 34. `Enumerable` vs `Queryable`

You'll often see these classes in documentation:

```csharp
Enumerable.Where(...)
```

and:

```csharp
Queryable.Where(...)
```

They look similar but serve different execution models.

### `Enumerable`

Works with:

```csharp
IEnumerable<T>
```

and generally takes:

```csharp
Func<T, ...>
```

### `Queryable`

Works with:

```csharp
IQueryable<T>
```

and generally takes:

```csharp
Expression<Func<T, ...>>
```

So:

```text
Enumerable
→ execute

Queryable
→ describe a query
```

That's a powerful way to remember it.

---

# 35. Example: same code, different overload

### In-memory

```csharp
IEnumerable<int> numbers = new[] { 1, 2, 3, 4 };

var result = numbers.Where(x => x > 2);
```

Uses:

```csharp
Enumerable.Where(...)
```

### Queryable

```csharp
IQueryable<Employee> employees = db.Employees;

var result = employees.Where(e => e.Salary > 100000);
```

Uses:

```csharp
Queryable.Where(...)
```

The lambda syntax looks identical.

The semantics are not.

---

# 36. Don't prematurely call `AsEnumerable()`

This:

```csharp
var result = db.Employees
    .AsEnumerable()
    .Where(e => e.Salary > 100000)
    .ToList();
```

forces the filtering to happen client-side.

Usually, this is better:

```csharp
var result = db.Employees
    .Where(e => e.Salary > 100000)
    .ToList();
```

unless you **intentionally need client-side logic**.

---

# 37. When `AsEnumerable()` is appropriate

It can be appropriate when:

1. You intentionally need a C# method that cannot be translated.
    
2. You want to perform a local transformation after server-side filtering.
    
3. The amount of data crossing the boundary is known to be reasonable.
    

For example:

```csharp
var result = db.Employees
    .Where(e => e.IsActive)
    .Select(e => new
    {
        e.Name,
        e.Salary
    })
    .AsEnumerable()
    .Where(e => MyComplexCSharpLogic(e))
    .ToList();
```

This is a deliberate architecture:

```text
Database:
    IsActive
    Select columns

        ↓

Application:
    Complex C# logic
```

---

# 38. The dangerous version

Avoid blindly doing:

```csharp
db.Employees
    .AsEnumerable()
    .Where(...)
```

if the database contains millions of rows.

You may have accidentally moved:

```text
WHERE
```

from the database into your application.

Always ask:

> How many rows cross the boundary?

---

# 39. `ToList()` vs `AsEnumerable()`

This distinction is worth memorizing.

### `ToList()`

```csharp
query.ToList()
```

means:

```text
Execute now
+
materialize results
```

### `AsEnumerable()`

```csharp
query.AsEnumerable()
```

means:

```text
Switch subsequent LINQ operations
to IEnumerable semantics.
```

It does not itself mean:

```text
"materialize everything immediately."
```

---

# 40. A complete example

Consider:

```csharp
var query = db.Employees
    .Where(e => e.IsActive)
    .Where(e => e.Salary > 100000)
    .Select(e => new
    {
        e.Name,
        e.Salary
    });
```

At this point:

```text
IQueryable<AnonymousType>
```

Conceptually:

```text
Database
    ↓
WHERE IsActive
    ↓
WHERE Salary > 100000
    ↓
SELECT Name, Salary
```

Then:

```csharp
var result = query.ToList();
```

executes it.

---

# 41. Add `AsEnumerable()`

Now:

```csharp
var query = db.Employees
    .Where(e => e.IsActive)
    .Where(e => e.Salary > 100000)
    .Select(e => new
    {
        e.Name,
        e.Salary
    })
    .AsEnumerable()
    .Where(e => MyCustomMethod(e))
    .ToList();
```

Conceptually:

```text
DATABASE
   │
   ├── WHERE IsActive
   ├── WHERE Salary > 100000
   └── SELECT Name, Salary
   │
   ▼
.NET
   │
   └── MyCustomMethod()
   │
   ▼
List
```

This is a very useful pattern when used intentionally.

---

# 42. A common interview question

### Question:

What is the difference between:

```csharp
IEnumerable<T>
```

and:

```csharp
IQueryable<T>
```

### Strong answer:

> `IEnumerable<T>` represents an enumerable sequence and LINQ operations against it are generally executed in memory using delegates. `IQueryable<T>` represents a queryable sequence backed by an `IQueryProvider`; LINQ operations build expression trees that the provider can translate and execute, such as translating LINQ into SQL. `IQueryable<T>` therefore enables provider-side filtering, projection, joins, aggregation, etc., while `IEnumerable<T>` executes those operations against the objects available in memory.

That's the technically meaningful answer.

---

# 43. Another interview question

### Why is this potentially bad?

```csharp
db.Users.ToList().Where(u => u.IsActive);
```

Because:

```text
ToList()
→ database query executes
→ all users are materialized
→ Where runs in memory
```

Instead:

```csharp
db.Users
    .Where(u => u.IsActive)
    .ToList();
```

allows the provider to potentially translate:

```sql
WHERE IsActive = ...
```

and return only matching rows.

---

# 44. Another important question

### Why can this fail?

```csharp
db.Users.Where(u => MyCustomMethod(u));
```

Because:

```text
MyCustomMethod
```

is arbitrary C# logic.

The query provider may not know how to translate it into SQL.

But:

```csharp
db.Users
    .AsEnumerable()
    .Where(u => MyCustomMethod(u));
```

makes the custom method execute in .NET.

The tradeoff is that the data has crossed from the provider into the application first.

---

# 45. The architecture you should memorize

```text
             LINQ SOURCE
                  │
        ┌─────────┴─────────┐
        │                   │
 IEnumerable<T>       IQueryable<T>
        │                   │
        ▼                   ▼
   Enumerable.*        Queryable.*
        │                   │
     Delegate          Expression Tree
        │                   │
        ▼                   ▼
    .NET runtime       Query Provider
                            │
                            ▼
                           SQL
                            │
                            ▼
                        Database
```

And:

```text
IQueryable
    │
    │ AsEnumerable()
    ▼
IEnumerable
    │
    ▼
C# execution
```

---

# 46. Phase 16 — The rules to memorize

### Rule 1

```csharp
IEnumerable<T>
```

means:

> **Think in-memory sequence.**

---

### Rule 2

```csharp
IQueryable<T>
```

means:

> **Think query provider + expression tree.**

---

### Rule 3

```csharp
Enumerable.Where()
```

generally receives:

```csharp
Func<T, bool>
```

while:

```csharp
Queryable.Where()
```

generally receives:

```csharp
Expression<Func<T, bool>>
```

---

### Rule 4

```csharp
ToList()
```

means:

> **Execute/materialize here.**

---

### Rule 5

```csharp
AsEnumerable()
```

means:

> **Subsequent operators use LINQ-to-Objects semantics.**

---

### Rule 6

With `IQueryable<T>`:

> **Always think about where the computation happens.**

```text
Database?
.NET process?
```

---

# 47. Final mental model

When you see:

```csharp
var query = something
    .Where(...)
    .Select(...)
    .GroupBy(...)
    .Join(...);
```

**don't immediately analyze the operators.**

First ask:

```text
What is `something`?
```

If:

```text
IEnumerable<T>
```

think:

```text
C# → objects → memory → delegates
```

If:

```text
IQueryable<T>
```

think:

```text
C# → expression tree → provider → SQL/database
```

Then ask:

```text
Where is ToList()?
Where is AsEnumerable()?
Where does execution actually happen?
How much data crosses the boundary?
Can the provider translate this expression?
```

That mental model is far more important than memorizing individual LINQ operators.

---

## Phase 16 checkpoint

Given:

```csharp
var result = db.Orders
    .Where(o => o.Total > 10000)
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        Total = g.Sum(o => o.Total)
    })
    .OrderByDescending(x => x.Total)
    .ToList();
```

You should mentally see:

```text
DbSet<Order>
     │
     ▼
IQueryable<Order>
     │
     ├── Where
     ├── GroupBy
     ├── Select
     └── OrderByDescending
     │
     ▼
Expression Tree
     │
     ▼
EF Core / Query Provider
     │
     ▼
SQL
     │
     ▼
Database
     │
     ▼
ToList()
     │
     ▼
List<AnonymousType>
```

Whereas:

```csharp
var result = orders
    .Where(o => o.Total > 10000)
    .GroupBy(o => o.CustomerId)
    .Select(...)
    .ToList();
```

where `orders` is an in-memory `List<Order>` means:

```text
List<Order>
    ↓
IEnumerable<Order>
    ↓
C# delegates
    ↓
.NET memory
    ↓
List<TResult>
```

### The one sentence to remember

> **`IEnumerable<T>` tells LINQ how to execute over data you already have; `IQueryable<T>` gives a provider a structured description of what you want done with the data.**