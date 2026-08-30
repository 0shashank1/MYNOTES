# Phase 3 — Core LINQ Operators

Now that you understand `IEnumerable<T>`, we can study the **actual LINQ operators**.

The key idea:

> A LINQ operator takes a sequence, performs some operation, and usually produces another sequence or a final value.

Think of LINQ as a **data-processing pipeline**.

```text
IEnumerable<T>
      ↓
   operator
      ↓
IEnumerable<TResult>
      ↓
   operator
      ↓
IEnumerable<TResult>
      ↓
terminal operator
      ↓
   final value
```

---

# 1. The core categories

Don't memorize 50+ LINQ methods. First understand these categories:

|Category|Operators|Question answered|
|---|---|---|
|Filtering|`Where`|Which elements?|
|Projection|`Select`|What should each become?|
|Flattening|`SelectMany`|How do I flatten nested sequences?|
|Sorting|`OrderBy`, `ThenBy`|How should they be ordered?|
|Partitioning|`Take`, `Skip`|Which portion?|
|Element|`First`, `Single`, `ElementAt`|Give me an element|
|Quantifiers|`Any`, `All`, `Contains`|Does something exist?|
|Aggregation|`Count`, `Sum`, `Min`, `Max`, `Average`|What is the overall value?|
|Grouping|`GroupBy`|How do I divide elements into groups?|
|Joining|`Join`, `GroupJoin`|How do two sequences relate?|
|Materialization|`ToList`, `ToArray`, `ToDictionary`|Execute and store the result|

We'll start with the most important ones.

---

# 2. `Where()` — Filtering

`Where()` answers:

> **Which elements should survive?**

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6 };

var result = numbers.Where(n => n % 2 == 0);
```

Result:

```text
2
4
6
```

The lambda:

```csharp
n => n % 2 == 0
```

is a **predicate**.

A predicate is essentially:

```text
T → bool
```

For each element, it answers:

```text
Should this element be included?
```

---

## `Where()` does not transform the element

```csharp
var result = numbers.Where(n => n > 3);
```

Input:

```text
1 2 3 4 5 6
```

Output:

```text
4 5 6
```

The elements are still `int`.

So:

```text
IEnumerable<int>
      ↓
    Where
      ↓
IEnumerable<int>
```

---

# 3. `Select()` — Projection

`Select()` answers:

> **What should each element become?**

```csharp
int[] numbers = { 1, 2, 3, 4 };

var result = numbers.Select(n => n * 10);
```

Result:

```text
10
20
30
40
```

Unlike `Where()`, `Select()` transforms.

```text
1 → 10
2 → 20
3 → 30
4 → 40
```

Therefore:

```text
IEnumerable<int>
      ↓
    Select
      ↓
IEnumerable<int>
```

But the type can also change.

```csharp
var result = numbers.Select(n => n.ToString());
```

Now:

```text
IEnumerable<int>
      ↓
    Select
      ↓
IEnumerable<string>
```

This is called **projection**.

---

# 4. `Where()` + `Select()`

This combination is probably the most common LINQ pattern.

```csharp
var result = students
    .Where(s => s.Age >= 18)
    .Select(s => s.Name);
```

Read it:

> Find adult students, then project them into their names.

Pipeline:

```text
Students
   ↓
Where(Age >= 18)
   ↓
Adult Students
   ↓
Select(Name)
   ↓
Names
```

This distinction is fundamental:

```text
Where  → filter
Select → transform
```

---

# 5. `OrderBy()` — Sorting

```csharp
var result = numbers.OrderBy(n => n);
```

Ascending order:

```text
1 2 3 4 5
```

For descending:

```csharp
var result = numbers.OrderByDescending(n => n);
```

Result:

```text
5 4 3 2 1
```

The lambda tells LINQ **what property/value to use as the sort key**.

For objects:

```csharp
var result = students
    .OrderBy(s => s.Age);
```

This means:

> Sort students according to their age.

---

# 6. `ThenBy()` — Secondary sorting

Suppose:

```csharp
var students = new[]
{
    new Student("Alice", 20),
    new Student("Bob", 18),
    new Student("Charlie", 20),
    new Student("David", 18)
};
```

You want:

1. Age ascending
    
2. Within the same age, name ascending
    

```csharp
var result = students
    .OrderBy(s => s.Age)
    .ThenBy(s => s.Name);
```

Conceptually:

```text
Age 18
    Bob
    David

Age 20
    Alice
    Charlie
```

Use:

```text
OrderBy → primary key
ThenBy  → secondary key
ThenBy  → tertiary key
...
```

---

# 7. `Take()` — Take the first N

```csharp
var result = numbers.Take(3);
```

If:

```text
1 2 3 4 5 6
```

you get:

```text
1 2 3
```

Very useful for things like:

```csharp
var topProducts = products
    .OrderByDescending(p => p.Price)
    .Take(10);
```

Meaning:

> Sort by price descending and take the first 10.

---

# 8. `Skip()` — Skip the first N

```csharp
var result = numbers.Skip(3);
```

Input:

```text
1 2 3 4 5 6
```

Output:

```text
4 5 6
```

`Take()` and `Skip()` are often combined for pagination:

```csharp
var page = products
    .Skip(20)
    .Take(10);
```

Conceptually:

```text
Products
   ↓
Skip first 20
   ↓
Take next 10
   ↓
Page 3
```

For a page size of 10:

```text
Page 1 → Skip(0).Take(10)
Page 2 → Skip(10).Take(10)
Page 3 → Skip(20).Take(10)
```

---

# 9. `First()`

`First()` means:

> Give me the first element.

```csharp
var first = numbers.First();
```

If:

```text
10 20 30
```

result:

```text
10
```

But there's an important issue.

If the sequence is empty:

```csharp
var first = emptySequence.First();
```

it throws:

```text
InvalidOperationException
```

---

# 10. `First(predicate)`

You can combine finding and filtering:

```csharp
var firstAdult = students
    .First(s => s.Age >= 18);
```

Conceptually:

```text
Find the first student
whose Age >= 18
```

This is roughly equivalent to:

```csharp
var firstAdult = students
    .Where(s => s.Age >= 18)
    .First();
```

---

# 11. `FirstOrDefault()`

If you don't want an exception when nothing exists:

```csharp
var student = students
    .FirstOrDefault(s => s.Name == "Alice");
```

If Alice exists:

```text
Student object
```

If not:

```text
null
```

For value types, the default can be:

```text
int → 0
bool → false
```

Modern C# also has overloads where you can explicitly specify a fallback value.

---

# 12. `Single()` — An important distinction

`Single()` means:

> **There must be exactly one matching element.**

```csharp
var student = students
    .Single(s => s.Id == 100);
```

Three possibilities:

```text
0 matches → exception
1 match   → return it
2+ matches → exception
```

This makes `Single()` useful when uniqueness is part of your **business invariant**.

For example:

```csharp
var user = users.Single(u => u.Email == email);
```

If your system guarantees email uniqueness, `Single()` can communicate that expectation.

---

# 13. `SingleOrDefault()`

Same idea, except zero matches are allowed:

```text
0 matches → default
1 match   → return it
2+ matches → exception
```

This is different from `FirstOrDefault()`.

### `FirstOrDefault()`

```text
0 → default
1 → first
2+ → first
```

### `SingleOrDefault()`

```text
0 → default
1 → return it
2+ → exception
```

That distinction matters.

---

# 14. `Any()` — Does anything exist?

This is one of the most useful LINQ operators.

```csharp
bool exists = numbers.Any();
```

Question:

> Does this sequence contain at least one element?

You can also provide a condition:

```csharp
bool hasAdults = students.Any(s => s.Age >= 18);
```

Meaning:

> Does at least one student satisfy the condition?

---

## `Any()` vs `Count() > 0`

Prefer:

```csharp
if (students.Any())
{
    ...
}
```

rather than:

```csharp
if (students.Count() > 0)
{
    ...
}
```

Why?

Because `Any()` expresses the actual intent:

> "I only care whether at least one exists."

It can also stop as soon as it finds one matching element.

---

# 15. `All()`

`All()` asks:

> **Do all elements satisfy this condition?**

```csharp
bool allAdults = students.All(s => s.Age >= 18);
```

Conceptually:

```text
Alice → true
Bob   → true
Charlie → true

All → true
```

If even one fails:

```text
Alice → true
Bob   → false
Charlie → true

All → false
```

---

# 16. `Contains()`

Checks whether a value exists:

```csharp
bool exists = numbers.Contains(30);
```

Result:

```text
true
```

For custom objects, `Contains()` depends on equality semantics.

This becomes important when learning:

- `Equals()`
    
- `GetHashCode()`
    
- `IEquatable<T>`
    
- equality comparers
    

We'll get there later.

---

# 17. Aggregation

Aggregation turns many values into **one value**.

Important operators:

```text
Count
Sum
Min
Max
Average
Aggregate
```

Example:

```csharp
int count = numbers.Count();
```

```csharp
int sum = numbers.Sum();
```

```csharp
int min = numbers.Min();
```

```csharp
int max = numbers.Max();
```

```csharp
double average = numbers.Average();
```

So:

```text
Many elements
     ↓
Aggregation
     ↓
One result
```

---

# 18. `Count(predicate)`

You can count only matching elements:

```csharp
int adultCount = students.Count(s => s.Age >= 18);
```

This is equivalent conceptually to:

```csharp
int adultCount = students
    .Where(s => s.Age >= 18)
    .Count();
```

---

# 19. `Sum()` with projection

Suppose:

```csharp
class Product
{
    public decimal Price { get; set; }
}
```

You can do:

```csharp
decimal total = products.Sum(p => p.Price);
```

This means:

```text
Products
   ↓
Extract Price
   ↓
Sum prices
   ↓
decimal
```

---

# 20. `SelectMany()` — Flattening

This operator deserves special attention.

Suppose:

```csharp
var departments = new[]
{
    new
    {
        Name = "Engineering",
        Employees = new[] { "Alice", "Bob" }
    },
    new
    {
        Name = "Sales",
        Employees = new[] { "Charlie", "David" }
    }
};
```

You have:

```text
Engineering → Alice, Bob
Sales       → Charlie, David
```

This is a nested sequence.

You can flatten it:

```csharp
var employees = departments
    .SelectMany(d => d.Employees);
```

Result:

```text
Alice
Bob
Charlie
David
```

Think:

```text
IEnumerable<IEnumerable<T>>
          ↓
     SelectMany()
          ↓
IEnumerable<T>
```

That's the essential purpose of `SelectMany()`.

---

# 21. `Select()` vs `SelectMany()`

This distinction is extremely important.

Suppose:

```csharp
var departments = ...
```

### `Select`

```csharp
var result = departments
    .Select(d => d.Employees);
```

Result is conceptually:

```text
[
    [Alice, Bob],
    [Charlie, David]
]
```

Nested sequence.

### `SelectMany`

```csharp
var result = departments
    .SelectMany(d => d.Employees);
```

Result:

```text
[
    Alice,
    Bob,
    Charlie,
    David
]
```

Flattened.

Mental rule:

```text
Select      → one input → one output

SelectMany  → one input → many outputs → flatten
```

---

# 22. `GroupBy()`

`GroupBy()` answers:

> **How can I divide this sequence into groups based on a key?**

Example:

```csharp
var groups = students
    .GroupBy(s => s.Age);
```

If students are:

```text
Alice   18
Bob     20
Charlie 18
David   20
```

You get groups conceptually:

```text
18
 ├── Alice
 └── Charlie

20
 ├── Bob
 └── David
```

Each group has a key:

```csharp
group.Key
```

and contains the elements belonging to that key.

---

# 23. `GroupBy()` becomes powerful with aggregation

For example:

```csharp
var result = students
    .GroupBy(s => s.Age)
    .Select(g => new
    {
        Age = g.Key,
        Count = g.Count()
    });
```

Result conceptually:

```text
Age 18 → Count 2
Age 20 → Count 2
```

Pipeline:

```text
Students
    ↓
GroupBy(Age)
    ↓
Groups
    ↓
Select each group
    ↓
{ Age, Count }
```

This pattern is extremely common in real applications.

---

# 24. `ToList()` — Materialization

So far, many operators produce:

```text
IEnumerable<T>
```

Eventually you may want an actual list:

```csharp
var result = numbers
    .Where(n => n > 10)
    .ToList();
```

Now:

```text
IEnumerable<int>
      ↓
   ToList()
      ↓
List<int>
```

`ToList()` **forces enumeration** and stores the resulting elements.

This is called:

> **Materialization**

---

# 25. Deferred vs immediate operators

This is an important classification.

Many LINQ operators are **deferred**:

```text
Where
Select
OrderBy
Skip
Take
GroupBy
SelectMany
```

They generally don't produce their final results until enumerated.

Other operations are **immediate/terminal**:

```text
ToList
ToArray
Count
Sum
First
Single
Any
Average
Min
Max
```

They cause the sequence to be evaluated to obtain a result.

Think:

```text
         DEFERRED
            ↓
Where → Select → OrderBy → Take
                              ↓
                         TERMINAL
                              ↓
                           ToList
```

---

# 26. A complete LINQ pipeline

Here's a realistic example:

```csharp
var result = products
    .Where(p => p.Price >= 100)
    .OrderByDescending(p => p.Price)
    .Take(5)
    .Select(p => p.Name)
    .ToList();
```

Let's execute the **logic conceptually**:

### Step 1

```csharp
.Where(p => p.Price >= 100)
```

Filter cheap products out.

### Step 2

```csharp
.OrderByDescending(p => p.Price)
```

Sort remaining products from expensive → cheap.

### Step 3

```csharp
.Take(5)
```

Keep five.

### Step 4

```csharp
.Select(p => p.Name)
```

Turn products into names.

### Step 5

```csharp
.ToList()
```

Execute and materialize.

Pipeline:

```text
Products
   ↓
Where
   ↓
Products >= $100
   ↓
OrderByDescending
   ↓
Most expensive first
   ↓
Take(5)
   ↓
Top 5
   ↓
Select(Name)
   ↓
string sequence
   ↓
ToList()
   ↓
List<string>
```

---

# 27. The three questions you should ask about every LINQ operator

Whenever you encounter a LINQ method, ask:

### 1. What does it do?

Example:

```csharp
Where()
```

→ filters.

### 2. What does it return?

Example:

```csharp
Where()
```

→ generally `IEnumerable<T>`.

### 3. When does it execute?

Example:

```csharp
Where()
```

→ deferred.

This gives you a very powerful way to analyze LINQ code instead of memorizing syntax.

---

# 28. Core operators to master first

For now, prioritize these:

```text
                    LINQ
                     │
       ┌─────────────┼─────────────┐
       │             │             │
    Filtering    Projection     Sorting
       │             │             │
     Where         Select       OrderBy
                                  ThenBy
       
       ┌─────────────┼─────────────┐
       │             │             │
 Partitioning   Elements       Quantifiers
       │             │             │
 Take / Skip   First / Single   Any / All
              ElementAt         Contains

       ┌─────────────┼─────────────┐
       │             │             │
 Aggregation    Grouping       Flattening
       │             │             │
Count/Sum/...  GroupBy       SelectMany

                     ↓
                Materialization
                     ↓
                  ToList()
                  ToArray()
```

---

# 29. The most important mental model

Don't see this:

```csharp
var result = products
    .Where(...)
    .Select(...)
    .OrderBy(...)
    .Take(...);
```

as four unrelated methods.

See it as:

```text
             DATA
              │
              ▼
           FILTER
              │
              ▼
         TRANSFORM
              │
              ▼
            SORT
              │
              ▼
           LIMIT
              │
              ▼
          MATERIALIZE
              │
              ▼
           RESULT
```

That's the **LINQ pipeline model**.

---

## Phase 3 checkpoint

You should now be able to explain these without memorizing definitions:

```csharp
.Where(x => ...)
```

→ **filter elements**

```csharp
.Select(x => ...)
```

→ **transform/project elements**

```csharp
.OrderBy(x => ...)
```

→ **sort elements**

```csharp
.Take(n)
```

→ **take a portion**

```csharp
.Any(...)
```

→ **check whether something exists**

```csharp
.Count(...)
```

→ **count**

```csharp
.GroupBy(...)
```

→ **create groups**

```csharp
.SelectMany(...)
```

→ **flatten nested sequences**

```csharp
.ToList()
```

→ **execute/materialize into a list**

And the central pattern is:

```text
IEnumerable<T>
     ↓
LINQ operator
     ↓
IEnumerable<TResult>
     ↓
LINQ operator
     ↓
...
     ↓
Terminal operation
     ↓
Result
```

**Next logical phase: Lambda Expressions + Delegates.** That's where we'll unpack exactly what this:

```csharp
n => n > 10
```

really is, why `Where()` accepts it, and how C# turns it into executable behavior.