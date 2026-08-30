
# Phase 10 — Aggregation in LINQ

Aggregation is the process of taking **many elements and reducing them to one result**.

If projection does:

```text
one → one
```

and quantifiers do:

```text
many → bool
```

aggregation generally does:

```text
many → one value
```

The core LINQ aggregation operators are:

```text
Count
LongCount
Sum
Min
Max
Average
Aggregate
```

---

# 1. The basic mental model

Suppose:

```csharp
int[] numbers = { 10, 20, 30, 40 };
```

Aggregation turns:

```text
10
20
30
40
```

into something like:

```text
Count  → 4
Sum    → 100
Min    → 10
Max    → 40
Average → 25
```

Think:

```text
IEnumerable<T>
      ↓
  Aggregation
      ↓
   TResult
```

---

# 2. `Count()`

`Count()` tells you how many elements are in the sequence.

```csharp
int count = numbers.Count();
```

Result:

```text
4
```

Conceptually:

```text
[10, 20, 30, 40]
        ↓
      Count
        ↓
        4
```

---

## `Count(predicate)`

You can count only matching elements:

```csharp
int evenCount = numbers.Count(n => n % 2 == 0);
```

All four are even:

```text
4
```

Another example:

```csharp
int largeCount = numbers.Count(n => n > 20);
```

Result:

```text
2
```

because:

```text
30
40
```

match.

---

# 3. `Count()` vs `Any()`

This is an important distinction from Phase 8.

If you need the actual number:

```csharp
int count = users.Count();
```

If you only need to know whether at least one exists:

```csharp
bool exists = users.Any();
```

Don't write:

```csharp
if (users.Count() > 0)
```

when you only need existence.

Use:

```csharp
if (users.Any())
```

The intent is clearer, and `Any()` can short-circuit.

---

# 4. `LongCount()`

`Count()` returns an `int`.

```csharp
int count = numbers.Count();
```

`LongCount()` returns a `long`:

```csharp
long count = numbers.LongCount();
```

This matters when the sequence could contain more elements than an `int` can represent.

In ordinary application code, `Count()` is usually sufficient.

---

# 5. `Sum()`

`Sum()` adds numeric values.

```csharp
int total = numbers.Sum();
```

For:

```text
10 + 20 + 30 + 40
```

result:

```text
100
```

---

# 6. `Sum()` with a projection

This is extremely common.

Suppose:

```csharp
class Product
{
    public decimal Price { get; set; }
}
```

You can calculate the total:

```csharp
decimal total = products.Sum(p => p.Price);
```

Conceptually:

```text
Products
   ↓
Select Price
   ↓
10.50
20.25
30.00
   ↓
Sum
   ↓
60.75
```

You don't have to explicitly call `Select()` first.

This:

```csharp
products.Sum(p => p.Price);
```

is effectively doing:

```text
project → aggregate
```

in one operator.

---

# 7. `Min()`

Finds the minimum value:

```csharp
int minimum = numbers.Min();
```

Result:

```text
10
```

With objects:

```csharp
decimal cheapest =
    products.Min(p => p.Price);
```

The lambda is the **key/value selector** used for the aggregation.

---

# 8. `Max()`

Similarly:

```csharp
int maximum = numbers.Max();
```

Result:

```text
40
```

With objects:

```csharp
decimal mostExpensive =
    products.Max(p => p.Price);
```

---

# 9. `Average()`

Calculates the arithmetic mean:

```csharp
double average = numbers.Average();
```

For:

```text
10, 20, 30, 40
```

calculation:

```text
(10 + 20 + 30 + 40) / 4
= 25
```

So:

```text
Average → 25
```

With objects:

```csharp
double averageSalary =
    employees.Average(e => e.Salary);
```

---

# 10. `Average()` and numeric types

Pay attention to the result type.

For example:

```csharp
int[] numbers = { 1, 2, 3 };
```

```csharp
var average = numbers.Average();
```

The result is a floating-point value, not integer division.

Conceptually:

```text
(1 + 2 + 3) / 3
= 2
```

But for values that don't divide evenly:

```csharp
int[] numbers = { 1, 2 };

var average = numbers.Average();
```

you get:

```text
1.5
```

This is an important difference from:

```csharp
int result = (1 + 2) / 2;
```

which uses integer division.

---

# 11. Empty sequences and aggregation

This is where aggregation gets interesting.

Consider:

```csharp
var numbers = Array.Empty<int>();
```

What happens with:

```csharp
numbers.Count();
```

Result:

```text
0
```

That's straightforward.

But:

```csharp
numbers.Sum();
```

returns the appropriate zero value for supported numeric types.

Whereas:

```csharp
numbers.Min();
```

and:

```csharp
numbers.Max();
```

generally throw because there is no minimum or maximum of an empty non-nullable numeric sequence.

Similarly:

```csharp
numbers.Average();
```

throws for an empty non-nullable numeric sequence.

So don't assume every aggregation has the same empty-sequence behavior.

---

# 12. Nullable values change aggregation behavior

Suppose:

```csharp
int?[] numbers =
{
    10,
    null,
    30
};
```

Then:

```csharp
var sum = numbers.Sum();
```

The null value is ignored for the numeric aggregation.

Conceptually:

```text
10
null → ignored
30
↓
40
```

Similarly, nullable numeric aggregations can return nullable results where appropriate.

This becomes important when working with database data because nullable columns are common.

---

# 13. `Count` is an aggregation, but not always a full scan

There's a subtle implementation detail worth understanding.

For a plain `IEnumerable<T>`:

```csharp
numbers.Count();
```

may require enumeration.

But if the source is a collection with a known count, LINQ can often use that information directly.

So don't oversimplify this as:

> "`Count()` always loops through everything."

The actual behavior depends on the source and available interfaces.

The semantic model is still:

> Count the elements.

---

# 14. Aggregation after filtering

A very common pattern is:

```csharp
int count = products
    .Where(p => p.Price > 1000)
    .Count();
```

Or more directly:

```csharp
int count = products
    .Count(p => p.Price > 1000);
```

Similarly:

```csharp
decimal revenue = orders
    .Where(o => o.Status == "Completed")
    .Sum(o => o.Total);
```

Pipeline:

```text
Orders
   ↓
Filter completed
   ↓
Project Total
   ↓
Sum
   ↓
Revenue
```

---

# 15. Aggregation after grouping

This is where LINQ becomes extremely powerful.

Suppose:

```csharp
var result = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        Department = g.Key,
        EmployeeCount = g.Count(),
        AverageSalary = g.Average(e => e.Salary),
        MaximumSalary = g.Max(e => e.Salary)
    });
```

Conceptually:

```text
Employees
    ↓
GroupBy Department
    ↓
Engineering group
Sales group
HR group
    ↓
Aggregate each group
    ↓
Department statistics
```

Result could look like:

```text
Engineering → Count: 20, Avg: 95000, Max: 150000
Sales       → Count: 15, Avg: 72000, Max: 110000
HR          → Count:  8, Avg: 68000, Max:  90000
```

This pattern is extremely common.

---

# 16. `Aggregate()` — the general-purpose aggregator

Now we reach the most powerful aggregation operator:

```csharp
Aggregate()
```

It lets you define **how the sequence should be reduced**.

For example, sum manually:

```csharp
int total = numbers.Aggregate(
    0,
    (accumulator, number) => accumulator + number
);
```

For:

```text
10, 20, 30
```

the process is:

```text
acc = 0

0 + 10 → 10
10 + 20 → 30
30 + 30 → 60
```

Final result:

```text
60
```

---

# 17. Understanding the accumulator

This is the most important concept behind `Aggregate()`.

You have:

```text
initial value
      ↓
 accumulator
      ↓
 process next element
      ↓
 new accumulator
      ↓
 process next element
      ↓
 ...
      ↓
 final result
```

For:

```csharp
numbers.Aggregate(
    0,
    (acc, n) => acc + n
);
```

you can visualize:

```text
Start
  acc = 0

n = 10
  acc = 0 + 10
  acc = 10

n = 20
  acc = 10 + 20
  acc = 30

n = 30
  acc = 30 + 30
  acc = 60
```

Final:

```text
60
```

---

# 18. `Aggregate()` doesn't have to return the same type

This is where it becomes much more powerful.

Suppose:

```csharp
int[] numbers = { 1, 2, 3, 4 };
```

You could build a string:

```csharp
string result = numbers.Aggregate(
    "",
    (text, number) => text + number
);
```

Conceptually:

```text
"" + 1
"1" + 2
"12" + 3
"123" + 4
```

Final:

```text
"1234"
```

The accumulator is:

```text
string
```

while the input elements are:

```text
int
```

So conceptually:

```text
IEnumerable<int>
      +
Accumulator: string
      ↓
string
```

---

# 19. A better string example

For strings, you would normally use `string.Join()` rather than `Aggregate()`:

```csharp
string result = string.Join(", ", numbers);
```

But `Aggregate()` is useful for understanding the underlying reduction mechanism.

For example, you can implement custom logic that isn't directly represented by `Sum`, `Count`, `Min`, etc.

---

# 20. `Aggregate()` as a fold

In functional programming, `Aggregate()` is closely related to a **fold/reduce** operation.

The conceptual structure is:

```text
[a, b, c, d]
    ↓
fold(initial, function)
    ↓
result
```

For example:

```text
[1, 2, 3, 4]
    ↓
fold(0, +)
    ↓
10
```

This connection is useful because many languages have an equivalent concept:

```text
C#       → Aggregate
JavaScript → reduce
Python   → reduce
F#       → fold
```

---

# 21. `Aggregate()` overloads

There are several forms.

### With seed

```csharp
numbers.Aggregate(
    0,
    (acc, n) => acc + n
);
```

The `0` is the initial accumulator.

### With seed + result selector

Conceptually:

```csharp
Aggregate(
    seed,
    accumulatorFunction,
    resultSelector
)
```

This allows you to transform the final accumulator into another result.

You don't need to memorize every overload yet. Understand the core concept first:

> **`Aggregate()` repeatedly updates an accumulator.**

---

# 22. Aggregation vs projection

This distinction is fundamental.

### Projection

```csharp
numbers.Select(n => n * 2);
```

Produces:

```text
1 → 2
2 → 4
3 → 6
```

Result:

```text
IEnumerable<int>
```

### Aggregation

```csharp
numbers.Sum();
```

Produces:

```text
1 + 2 + 3
```

Result:

```text
int
```

So:

```text
Projection:
many → many

Aggregation:
many → one
```

---

# 23. Aggregation vs quantifiers

You can now distinguish three major LINQ families:

### Projection

```csharp
Select()
```

```text
many → many
```

### Quantification

```csharp
Any()
All()
Contains()
```

```text
many → bool
```

### Aggregation

```csharp
Count()
Sum()
Min()
Max()
Average()
Aggregate()
```

```text
many → one value
```

This gives you a very useful mental taxonomy.

---

# 24. Aggregation and deferred execution

Most aggregators are **terminal operations**.

Compare:

```csharp
var query = numbers
    .Where(n => n > 10);
```

This gives:

```text
IEnumerable<int>
```

But:

```csharp
var sum = numbers
    .Where(n => n > 10)
    .Sum();
```

returns:

```text
int
```

The `Sum()` has to execute the preceding pipeline to calculate the answer.

Conceptually:

```text
numbers
   ↓
Where
   ↓
IEnumerable<int>
   ↓
Sum
   ↓
int
```

---

# 25. Aggregation can terminate after processing everything

Unlike `Any()` and `First()`, most aggregators need to inspect the relevant elements.

For example:

```csharp
numbers.Sum();
```

must conceptually process:

```text
10
20
30
40
```

to calculate:

```text
100
```

Likewise, `Average()` needs enough information to calculate the mean.

So:

```text
Any()
→ may stop early

First()
→ may stop early

Sum()
→ generally processes all relevant elements
```

---

# 26. `Min()` and `Max()` can conceptually stream

An important performance concept:

You don't need to store every element to calculate a minimum.

Conceptually:

```text
numbers = 50, 20, 80, 10, 40

current min = 50

20 → min = 20
80 → min = 20
10 → min = 10
40 → min = 10
```

So:

```text
Min()
```

can conceptually process the sequence in one pass while keeping only the current minimum.

Likewise for `Max()`.

This is a useful distinction from sorting.

To find the maximum, you don't necessarily need:

```csharp
numbers.OrderByDescending(n => n).First();
```

You can simply use:

```csharp
numbers.Max();
```

---

# 27. `Max()` vs `OrderByDescending().First()`

These may appear logically related:

```csharp
numbers.Max();
```

versus:

```csharp
numbers
    .OrderByDescending(n => n)
    .First();
```

But they're not equivalent from a computational perspective.

The first asks:

> What is the maximum value?

The second says:

> Sort the entire sequence, then give me the first element.

If you only need the maximum **value**, use:

```csharp
Max()
```

Sorting is unnecessary work.

---

# 28. Getting the object with the maximum property

Here's a subtle case.

Suppose:

```csharp
class Employee
{
    public string Name { get; set; }
    public decimal Salary { get; set; }
}
```

You want:

> The employee with the highest salary.

You might write:

```csharp
var employee = employees
    .OrderByDescending(e => e.Salary)
    .First();
```

This gets the **object**.

But:

```csharp
employees.Max(e => e.Salary);
```

gets only the **salary value**.

That's an important distinction:

```text
Max(e => e.Salary)
    ↓
decimal

OrderByDescending(e => e.Salary).First()
    ↓
Employee
```

Depending on your .NET version and available APIs, there are also newer `MaxBy`/`MinBy` operators specifically designed to retrieve the object with the extreme key:

```csharp
var employee = employees.MaxBy(e => e.Salary);
```

That is often a cleaner expression of the intent.

---

# 29. Aggregation after projection

You can explicitly separate projection and aggregation:

```csharp
var total = products
    .Select(p => p.Price)
    .Sum();
```

Or use the selector overload:

```csharp
var total = products.Sum(p => p.Price);
```

Both express:

```text
Product
   ↓
Price
   ↓
Sum
```

The second is concise.

The first can sometimes be easier to reason about when you're learning the pipeline.

---

# 30. Aggregation with filtering

A very common production pattern:

```csharp
var revenue = orders
    .Where(o => o.Status == OrderStatus.Completed)
    .Sum(o => o.Total);
```

Type flow:

```text
IEnumerable<Order>
       ↓
Where
       ↓
IEnumerable<Order>
       ↓
Sum(o => o.Total)
       ↓
decimal
```

The key is that `Sum()` operates on the **filtered sequence**.

---

# 31. Aggregation with grouping

Another highly important pattern:

```csharp
var report = orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        OrderCount = g.Count(),
        TotalSpent = g.Sum(o => o.Total)
    });
```

Think:

```text
Orders
   ↓
Group by Customer
   ↓
Customer 1 → orders
Customer 2 → orders
Customer 3 → orders
   ↓
Aggregate each group
   ↓
Customer statistics
```

This pattern appears everywhere in:

- reporting
    
- analytics
    
- dashboards
    
- financial calculations
    
- data processing
    
- database queries
    

---

# 32. A realistic example

Suppose you have:

```csharp
class Order
{
    public int CustomerId { get; set; }
    public decimal Total { get; set; }
    public bool IsCompleted { get; set; }
}
```

Requirement:

> For each customer, calculate the number of completed orders and their total spending.

LINQ:

```csharp
var report = orders
    .Where(o => o.IsCompleted)
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        OrderCount = g.Count(),
        TotalSpent = g.Sum(o => o.Total)
    });
```

Pipeline:

```text
Orders
   ↓
Where(IsCompleted)
   ↓
Completed Orders
   ↓
GroupBy(CustomerId)
   ↓
Groups
   ↓
Select each group
   ├── Count()
   └── Sum()
   ↓
Customer report
```

This is the point where LINQ starts looking less like "fancy loops" and more like a **declarative data-processing language**.

---

# 33. Aggregation and database queries

This becomes particularly important with Entity Framework Core.

Suppose:

```csharp
var total = db.Orders
    .Where(o => o.IsCompleted)
    .Sum(o => o.Total);
```

When operating against a database query provider, the LINQ expression can be translated into a database aggregation conceptually similar to:

```sql
SELECT SUM(Total)
FROM Orders
WHERE IsCompleted = 1;
```

The major advantage is that the database can perform the aggregation rather than sending every order to your application.

Conceptually:

```text
BAD idea for large data:

Database
   ↓
Millions of rows
   ↓
Application memory
   ↓
Sum in C#


Better query:

Database
   ↓
SUM(...)
   ↓
One result
   ↓
Application
```

This distinction becomes critical when we later study **`IQueryable<T>` and expression trees**.

---

# 34. The aggregation family

Keep this map in your head:

```text
                   AGGREGATION
                        │
        ┌───────────────┼────────────────┐
        │               │                │
      Counting       Numeric           General
        │           aggregation        reduction
        │               │                │
      Count          Sum               Aggregate
      LongCount      Min
                     Max
                     Average
```

And their typical result:

```text
Count      → int
LongCount  → long
Sum        → numeric type
Min        → element/key type
Max        → element/key type
Average    → floating/decimal result depending on source
Aggregate  → whatever type you define
```

---

# 35. The deepest mental model: reduction

Projection:

```text
A → B
A → B
A → B
A → B
```

Aggregation:

```text
A ─┐
A ─┤
A ─┤──→ B
A ─┘
```

For example:

```text
10 ─┐
20 ─┤
30 ─┼──→ 60
40 ─┘
```

That's why aggregation is often called **reduction**.

You're reducing a collection of values to one result.

---

# Phase 10 checkpoint

Given:

```csharp
var result = orders
    .Where(o => o.Status == "Completed")
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        Count = g.Count(),
        Total = g.Sum(o => o.Total),
        Average = g.Average(o => o.Total)
    });
```

You should be able to trace:

```text
IEnumerable<Order>
        ↓
Where
        ↓
IEnumerable<Order>
        ↓
GroupBy
        ↓
IEnumerable<IGrouping<int, Order>>
        ↓
Select
        ↓
IEnumerable<ReportRow>
```

And inside each group:

```text
g.Count()
    → number of orders

g.Sum(o => o.Total)
    → total value

g.Average(o => o.Total)
    → average order value
```

The key distinction from everything so far:

```text
Where       → filter many
Select      → transform many
OrderBy     → reorder many
Any/All     → answer yes/no
First       → retrieve one
Count/Sum   → reduce many → one
GroupBy     → split many → groups
```

### The next major concept

The natural next step is **`GroupBy()` in depth**: how `IGrouping<TKey, TElement>` works, how grouping differs from SQL `GROUP BY`, grouping + aggregation, composite keys, and the common mistake of confusing **grouping** with **sorting**.