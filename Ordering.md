# Phase 6 — Ordering in LINQ

Ordering is the LINQ family used to **sort elements according to one or more keys**.

The core operators are:

```text
OrderBy
OrderByDescending
ThenBy
ThenByDescending
```

The most important concept is:

> **Ordering does not change the elements. It changes their sequence order.**

---

# 1. `OrderBy()`

Suppose:

```csharp
int[] numbers = { 50, 10, 40, 20, 30 };
```

You can sort ascending:

```csharp
var result = numbers.OrderBy(n => n);
```

Result:

```text
10
20
30
40
50
```

Conceptually:

```text
IEnumerable<int>
      ↓
 OrderBy(n => n)
      ↓
IEnumerable<int>
```

The elements are still `int`.

Only their **order** changed.

---

# 2. `OrderByDescending()`

For descending order:

```csharp
var result = numbers.OrderByDescending(n => n);
```

Result:

```text
50
40
30
20
10
```

So:

```csharp
OrderBy(x => key)
```

means:

> Sort by `key` ascending.

While:

```csharp
OrderByDescending(x => key)
```

means:

> Sort by `key` descending.

---

# 3. Ordering objects

Ordering becomes more useful with objects.

```csharp
class Student
{
    public string Name { get; set; }
    public int Age { get; set; }
    public double GPA { get; set; }
}
```

Suppose:

```csharp
var students = new[]
{
    new Student { Name = "Alice", Age = 21, GPA = 3.2 },
    new Student { Name = "Bob", Age = 19, GPA = 3.8 },
    new Student { Name = "Charlie", Age = 20, GPA = 3.5 }
};
```

Sort by age:

```csharp
var result = students.OrderBy(s => s.Age);
```

Result:

```text
Bob      19
Charlie  20
Alice    21
```

The lambda:

```csharp
s => s.Age
```

is the **key selector**.

---

# 4. Key selector

This is an important LINQ concept.

In:

```csharp
students.OrderBy(s => s.Age);
```

you aren't telling LINQ:

> "Compare students directly."

You're saying:

> "For each student, extract the value that should be used for ordering."

```text
Student
   ↓
s => s.Age
   ↓
Age
   ↓
compare ages
```

So:

```csharp
OrderBy<TSource, TKey>(
    Func<TSource, TKey> keySelector
)
```

Conceptually.

For our example:

```text
TSource = Student
TKey    = int
```

Therefore:

```text
IEnumerable<Student>
       +
Func<Student, int>
       ↓
ordered sequence of Student
```

This is a useful type-level way to understand ordering.

---

# 5. Multiple ordering criteria

Suppose two students have the same age.

```text
Alice   20
Bob     20
Charlie 18
David   18
```

If you only do:

```csharp
var result = students.OrderBy(s => s.Age);
```

you've specified only the **primary ordering key**.

What if you also want names alphabetically?

Use:

```csharp
var result = students
    .OrderBy(s => s.Age)
    .ThenBy(s => s.Name);
```

Result:

```text
18 → Charlie
18 → David
20 → Alice
20 → Bob
```

The ordering pipeline is:

```text
Primary key
     ↓
Age
     ↓
Secondary key
     ↓
Name
```

---

# 6. `ThenBy()`

`ThenBy()` means:

> **If two elements have the same previous ordering key, use this new key to break the tie.**

Example:

```csharp
var result = students
    .OrderBy(s => s.Department)
    .ThenBy(s => s.Name);
```

Meaning:

1. Sort by department.
    
2. Within each department, sort by name.
    

---

# 7. `ThenByDescending()`

You can mix ascending and descending ordering.

```csharp
var result = students
    .OrderBy(s => s.Department)
    .ThenByDescending(s => s.GPA);
```

Meaning:

1. Department → ascending
    
2. GPA within department → descending
    

For example:

```text
Engineering
    Alice    3.9
    Bob      3.5
    Charlie  3.1

Sales
    David    4.0
    Eve      3.7
```

---

# 8. The critical distinction: `OrderBy` vs `ThenBy`

This is one of the most common LINQ mistakes.

### Correct

```csharp
var result = students
    .OrderBy(s => s.Age)
    .ThenBy(s => s.Name);
```

### Usually wrong when you intend multi-level sorting

```csharp
var result = students
    .OrderBy(s => s.Age)
    .OrderBy(s => s.Name);
```

Why?

Because the second `OrderBy()` establishes a **new primary ordering**.

Think:

```text
OrderBy
  ↓
primary ordering

ThenBy
  ↓
tie-breaker
```

Whereas:

```text
OrderBy
  ↓
primary ordering

OrderBy
  ↓
REPLACE primary ordering
```

So when adding another sort criterion, use `ThenBy()`.

---

# 9. Example showing the difference

Suppose:

```text
Name      Age
----------------
Alice     20
Bob       18
Charlie   20
David     18
```

### Using `OrderBy` + `ThenBy`

```csharp
students
    .OrderBy(s => s.Age)
    .ThenBy(s => s.Name);
```

Result:

```text
Bob      18
David    18
Alice    20
Charlie  20
```

### Using two `OrderBy`s

```csharp
students
    .OrderBy(s => s.Age)
    .OrderBy(s => s.Name);
```

The second ordering takes precedence, so the result is primarily name-sorted:

```text
Alice
Bob
Charlie
David
```

The previous age ordering isn't being used as a tie-breaker.

---

# 10. Ordering and projection

Remember Phase 4?

```csharp
Select()
```

changes the shape.

Therefore, the position of `Select()` matters.

Suppose:

```csharp
students
    .OrderByDescending(s => s.GPA)
    .Select(s => s.Name);
```

This is straightforward:

```text
Student
   ↓
Order by GPA
   ↓
Student
   ↓
Select Name
   ↓
string
```

But:

```csharp
students
    .Select(s => s.Name)
    .OrderBy(name => name);
```

means:

```text
Student
   ↓
Select Name
   ↓
string
   ↓
OrderBy Name
   ↓
string
```

Both are valid, but they operate on different types.

---

# 11. Ordering before projection

Often this is natural:

```csharp
var result = students
    .OrderByDescending(s => s.GPA)
    .Select(s => s.Name);
```

You first determine the desired order using the original object, then project.

Conceptually:

```text
Students
   ↓
Sort by GPA
   ↓
Ordered Students
   ↓
Select Name
   ↓
Names
```

---

# 12. Ordering after projection

You can also do:

```csharp
var result = students
    .Select(s => new
    {
        s.Name,
        s.GPA
    })
    .OrderByDescending(s => s.GPA);
```

Now the ordering works against the **projected object**.

```text
Student
   ↓
{ Name, GPA }
   ↓
OrderByDescending(GPA)
```

This is useful when you've deliberately created a smaller result shape.

---

# 13. Ordering strings

You can order strings directly:

```csharp
var names = new[]
{
    "Charlie",
    "Alice",
    "Bob"
};

var result = names.OrderBy(name => name);
```

Result:

```text
Alice
Bob
Charlie
```

Descending:

```csharp
var result = names.OrderByDescending(name => name);
```

Result:

```text
Charlie
Bob
Alice
```

The actual ordering behavior for strings is governed by .NET's comparison/equality infrastructure and can involve comparer/culture considerations.

For ordinary LINQ learning, just remember:

```text
OrderBy(string)
→ ascending according to the applicable comparer
```

---

# 14. Custom ordering with a comparer

LINQ ordering also supports an `IComparer<TKey>`.

Conceptually:

```csharp
OrderBy(keySelector, comparer)
```

For example, you might have a custom comparison rule.

```csharp
var result = names.OrderBy(
    name => name,
    StringComparer.OrdinalIgnoreCase
);
```

This lets you explicitly control how keys are compared.

This becomes useful when you need deterministic or domain-specific ordering semantics.

---

# 15. Ordering is deferred

This connects directly to the previous phases.

Consider:

```csharp
var result = numbers.OrderBy(n => n);
```

You should not think of `result` as an already-materialized list.

It represents an ordered sequence.

When you enumerate:

```csharp
foreach (var number in result)
{
    Console.WriteLine(number);
}
```

the ordering operation is evaluated.

If you need a concrete list:

```csharp
var result = numbers
    .OrderBy(n => n)
    .ToList();
```

Now the sequence has been materialized.

---

# 16. Ordering and `IOrderedEnumerable<T>`

Here's a more advanced detail.

`OrderBy()` doesn't merely return:

```csharp
IEnumerable<T>
```

Its result is conceptually an:

```csharp
IOrderedEnumerable<T>
```

This is what allows:

```csharp
.OrderBy(...)
.ThenBy(...)
.ThenBy(...)
```

to work naturally.

Conceptually:

```text
IEnumerable<T>
      ↓
   OrderBy
      ↓
IOrderedEnumerable<T>
      ↓
   ThenBy
      ↓
IOrderedEnumerable<T>
```

You don't usually need to explicitly declare this type:

```csharp
var ordered = students.OrderBy(s => s.Age);
```

But understanding it helps explain why `ThenBy()` is associated with an ordered sequence.

---

# 17. Ordering isn't necessarily cheap

Sorting has computational cost.

For an in-memory collection of `n` elements, comparison-based sorting is generally around:

```text
O(n log n)
```

in typical implementations.

So:

```csharp
var result = hugeCollection
    .OrderBy(x => x.SomeProperty);
```

is not free.

This matters when you're working with large datasets.

---

# 18. `OrderBy()` with `Take()`

A very common pattern:

```csharp
var topFive = products
    .OrderByDescending(p => p.Price)
    .Take(5);
```

Conceptually:

```text
Products
   ↓
Sort by Price DESC
   ↓
Most expensive first
   ↓
Take 5
```

This pattern is common for:

- top 10 products
    
- highest scores
    
- newest records
    
- largest transactions
    
- leaderboard queries
    

---

# 19. Ordering and `Where()`

You can combine filtering and sorting:

```csharp
var result = products
    .Where(p => p.Category == "Laptop")
    .OrderByDescending(p => p.Price);
```

Pipeline:

```text
Products
   ↓
Where(Category == Laptop)
   ↓
Laptop products
   ↓
OrderByDescending(Price)
   ↓
Most expensive laptops first
```

The ordering is applied to the filtered sequence.

---

# 20. Ordering direction can be mixed

For example:

```csharp
var result = students
    .OrderBy(s => s.Department)
    .ThenByDescending(s => s.GPA)
    .ThenBy(s => s.Name);
```

This means:

```text
1. Department ascending
2. GPA descending within department
3. Name ascending when GPA is equal
```

Think of it as a hierarchy:

```text
Department
    │
    ├── GPA DESC
    │      │
    │      └── Name ASC
    │
    └── next department
```

---

# 21. A realistic example

Suppose:

```csharp
class Employee
{
    public string Name { get; set; }
    public string Department { get; set; }
    public decimal Salary { get; set; }
}
```

Requirement:

> Show employees grouped by department ordering, with the highest-paid employees first inside each department, and names alphabetically when salaries are equal.

LINQ:

```csharp
var result = employees
    .OrderBy(e => e.Department)
    .ThenByDescending(e => e.Salary)
    .ThenBy(e => e.Name);
```

The ordering specification is almost a direct translation of the requirement:

```text
Department ASC
       ↓
Salary DESC
       ↓
Name ASC
```

That's one of the strengths of LINQ: **the query often closely resembles the business requirement.**

---

# 22. A subtle point: stable ordering

LINQ ordering preserves the relative order of elements when their keys compare equal under the applicable ordering semantics.

For example:

```text
Alice → 20
Bob   → 20
```

If both have the same key, their relative order can be preserved by the ordering operation.

This is useful when reasoning about multi-stage ordering and ties.

But in application code, if you need a **fully deterministic order**, especially across database queries, it's often better to specify enough keys to uniquely determine the desired order:

```csharp
.OrderBy(x => x.Name)
.ThenBy(x => x.Id)
```

---

# 23. Ordering in LINQ-to-Objects vs databases

This becomes particularly important when you eventually use Entity Framework Core.

With LINQ-to-Objects:

```csharp
employees.OrderBy(e => e.Salary)
```

.NET performs the ordering over the in-memory sequence.

With a database query:

```csharp
db.Employees
    .OrderBy(e => e.Salary)
```

the query provider can translate the operation into SQL conceptually similar to:

```sql
ORDER BY Salary
```

So the **same LINQ syntax can represent different execution mechanisms**, depending on the source.

This is one reason understanding `IEnumerable<T>` before `IQueryable<T>` is important.

---

# 24. The type-flow model

For ordering, keep this model:

```text
IEnumerable<TSource>
        │
        │ OrderBy(Func<TSource, TKey>)
        ▼
IOrderedEnumerable<TSource>
        │
        │ ThenBy(Func<TSource, TKey2>)
        ▼
IOrderedEnumerable<TSource>
```

Notice something important:

> **Ordering normally does not change `TSource`.**

For example:

```text
IEnumerable<Employee>
        ↓
OrderBy(e => e.Salary)
        ↓
IOrderedEnumerable<Employee>
```

Still employees.

Compare that with projection:

```text
IEnumerable<Employee>
        ↓
Select(e => e.Name)
        ↓
IEnumerable<string>
```

So:

```text
Select → changes representation/type
OrderBy → changes order
```

---

# 25. Ordering vs `Where()` vs `Select()`

At this point, these three should be very clear:

|Operator|Question|Changes element type?|
|---|---|--:|
|`Where()`|Should this element remain?|Usually no|
|`Select()`|What should this element become?|Often yes|
|`OrderBy()`|Where should this element appear?|No|

Example:

```csharp
var result = products
    .Where(p => p.Price > 100)
    .OrderByDescending(p => p.Price)
    .Select(p => p.Name)
    .ToList();
```

Read it as:

```text
FILTER
↓
Keep products > 100

ORDER
↓
Most expensive first

PROJECT
↓
Turn Product → Name

MATERIALIZE
↓
Create List<string>
```

---

# Phase 6 mental model

The most important picture is:

```text
                  ORDERING
                     │
                     ▼
              ┌─────────────┐
              │  Sequence   │
              └──────┬──────┘
                     │
                     │ key selector
                     ▼
              ┌─────────────┐
              │   Key       │
              │ TKey        │
              └──────┬──────┘
                     │
                     ▼
                 Compare
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      ascending             descending
      OrderBy                OrderByDescending
          │
          ▼
       tie-break
          │
          ▼
       ThenBy
       ThenByDescending
```

### The key rules

```csharp
OrderBy(x => x.Key)
```

→ primary ascending order

```csharp
OrderByDescending(x => x.Key)
```

→ primary descending order

```csharp
ThenBy(x => x.Key)
```

→ secondary ascending order

```csharp
ThenByDescending(x => x.Key)
```

→ secondary descending order

And the critical rule:

> **Use `ThenBy` for additional sort criteria; another `OrderBy` starts a new primary ordering.**

---

## Phase 6 checkpoint

Given:

```csharp
var result = employees
    .Where(e => e.Salary > 50000)
    .OrderBy(e => e.Department)
    .ThenByDescending(e => e.Salary)
    .Select(e => new
    {
        e.Name,
        e.Salary
    })
    .ToList();
```

You should be able to trace it:

```text
IEnumerable<Employee>
        ↓
Where
        ↓
IEnumerable<Employee>
        ↓
OrderBy(Department)
        ↓
IOrderedEnumerable<Employee>
        ↓
ThenByDescending(Salary)
        ↓
IOrderedEnumerable<Employee>
        ↓
Select
        ↓
IEnumerable<AnonymousType>
        ↓
ToList
        ↓
List<AnonymousType>
```

That **type-flow + data-flow** way of reading LINQ is the skill we're building.

**Next logical phase: Partitioning — `Take`, `Skip`, `TakeWhile`, `SkipWhile`**, including how pagination works and why `Skip().Take()` has important performance implications with databases.