
# Phase 12 — Grouping in LINQ

Grouping is where LINQ becomes much more powerful for **analytics, reporting, categorization, and aggregation**.

The primary operator is:

```csharp
GroupBy()
```

The core idea:

> **`GroupBy()` takes a sequence and partitions its elements into groups based on a key.**

For example:

```text
Students
   ↓
GroupBy(Age)
   ↓
Age 18 → students aged 18
Age 19 → students aged 19
Age 20 → students aged 20
```

---

# 1. The simplest `GroupBy()`

Suppose:

```csharp
int[] numbers =
{
    1, 2, 3, 4, 5, 6
};
```

Group by even/odd:

```csharp
var groups = numbers.GroupBy(n => n % 2);
```

The key is:

```csharp
n => n % 2
```

So:

```text
1 % 2 = 1
2 % 2 = 0
3 % 2 = 1
4 % 2 = 0
5 % 2 = 1
6 % 2 = 0
```

Conceptually:

```text
Key = 0
    2
    4
    6

Key = 1
    1
    3
    5
```

---

# 2. What exactly does `GroupBy()` return?

This is where understanding types becomes important.

If:

```csharp
var groups = numbers.GroupBy(n => n % 2);
```

then conceptually:

```text
IEnumerable<int>
       ↓
GroupBy(n => n % 2)
       ↓
IEnumerable<IGrouping<int, int>>
```

The important type is:

```csharp
IGrouping<TKey, TElement>
```

It represents:

> **One group with a key and a sequence of elements belonging to that key.**

---

# 3. `IGrouping<TKey, TElement>`

Suppose:

```csharp
var groups = numbers.GroupBy(n => n % 2);
```

Then:

```text
TKey     = int
TElement = int
```

So:

```text
IGrouping<int, int>
```

means:

```text
Group
 ├── Key
 └── Elements
```

For example:

```text
Group
Key = 0

Elements:
2
4
6
```

Another:

```text
Group
Key = 1

Elements:
1
3
5
```

---

# 4. Accessing the group key

You access the key with:

```csharp
group.Key
```

Example:

```csharp
foreach (var group in groups)
{
    Console.WriteLine($"Key: {group.Key}");
}
```

Output:

```text
Key: 1
Key: 0
```

The exact ordering of groups follows the ordering in which distinct keys are first encountered for LINQ-to-Objects.

---

# 5. Enumerating elements inside a group

A group itself is enumerable.

```csharp
foreach (var group in groups)
{
    Console.WriteLine($"Key: {group.Key}");

    foreach (var number in group)
    {
        Console.WriteLine(number);
    }
}
```

Conceptually:

```text
Group 1
 ├── 1
 ├── 3
 └── 5

Group 0
 ├── 2
 ├── 4
 └── 6
```

This is why `IGrouping<TKey, TElement>` is so useful:

```text
IGrouping
    ↓
Key + IEnumerable<TElement>
```

---

# 6. Grouping objects

This is where `GroupBy()` becomes practical.

Suppose:

```csharp
class Employee
{
    public string Name { get; set; }
    public string Department { get; set; }
    public decimal Salary { get; set; }
}
```

And:

```csharp
var employees = new[]
{
    new Employee { Name = "Alice", Department = "Engineering", Salary = 90000 },
    new Employee { Name = "Bob", Department = "Sales", Salary = 70000 },
    new Employee { Name = "Charlie", Department = "Engineering", Salary = 100000 },
    new Employee { Name = "David", Department = "Sales", Salary = 80000 }
};
```

Group by department:

```csharp
var groups = employees
    .GroupBy(e => e.Department);
```

Conceptually:

```text
Engineering
 ├── Alice
 └── Charlie

Sales
 ├── Bob
 └── David
```

---

# 7. Reading a group

You can do:

```csharp
foreach (var group in groups)
{
    Console.WriteLine(group.Key);

    foreach (var employee in group)
    {
        Console.WriteLine(employee.Name);
    }
}
```

Output conceptually:

```text
Engineering
Alice
Charlie

Sales
Bob
David
```

Notice that:

```csharp
group.Key
```

is the department.

And:

```csharp
group
```

contains the employees belonging to that department.

---

# 8. Grouping is not sorting

This distinction is critical.

### Sorting

```csharp
employees.OrderBy(e => e.Department);
```

means:

> Put employees in department order.

You still have:

```text
IEnumerable<Employee>
```

### Grouping

```csharp
employees.GroupBy(e => e.Department);
```

means:

> Create separate groups for each department.

You now have conceptually:

```text
IEnumerable<IGrouping<string, Employee>>
```

So:

```text
OrderBy
→ changes sequence order

GroupBy
→ changes the structure of the data
```

---

# 9. Grouping changes the type

This is one of the most important concepts in Phase 12.

Before:

```text
IEnumerable<Employee>
```

After:

```csharp
employees.GroupBy(e => e.Department)
```

conceptually:

```text
IEnumerable<IGrouping<string, Employee>>
```

Visualize:

```text
Employee
Employee
Employee
Employee
     ↓
   GroupBy
     ↓
┌───────────────┐
│ Engineering   │
│   Employee    │
│   Employee    │
└───────────────┘

┌───────────────┐
│ Sales         │
│   Employee    │
│   Employee    │
└───────────────┘
```

---

# 10. Grouping + `Count()`

This is one of the most common patterns.

```csharp
var result = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        Department = g.Key,
        EmployeeCount = g.Count()
    });
```

Result:

```text
Engineering → 2
Sales       → 2
```

Pipeline:

```text
Employees
    ↓
GroupBy(Department)
    ↓
Groups
    ↓
Select(each group)
    ↓
{ Department, EmployeeCount }
```

---

# 11. Grouping + `Sum()`

Suppose you want total salary per department:

```csharp
var result = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        Department = g.Key,
        TotalSalary = g.Sum(e => e.Salary)
    });
```

Result:

```text
Engineering → 190000
Sales       → 150000
```

Conceptually:

```text
Engineering
  90000
  100000
      ↓
    Sum
      ↓
   190000
```

---

# 12. Grouping + `Average()`

Average salary per department:

```csharp
var result = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        Department = g.Key,
        AverageSalary = g.Average(e => e.Salary)
    });
```

Conceptually:

```text
Engineering
  90000
  100000
      ↓
   Average
      ↓
   95000
```

---

# 13. Grouping + multiple aggregations

This is where LINQ becomes extremely expressive.

```csharp
var report = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        Department = g.Key,
        Count = g.Count(),
        TotalSalary = g.Sum(e => e.Salary),
        AverageSalary = g.Average(e => e.Salary),
        MaximumSalary = g.Max(e => e.Salary),
        MinimumSalary = g.Min(e => e.Salary)
    });
```

Now each group becomes a report row:

```text
Engineering
    Count: 2
    Total: 190000
    Average: 95000
    Max: 100000
    Min: 90000

Sales
    Count: 2
    Total: 150000
    Average: 75000
    Max: 80000
    Min: 70000
```

This pattern is extremely common in analytics code.

---

# 14. `GroupBy()` + projection

You aren't required to keep the original objects.

For example:

```csharp
var result = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        Department = g.Key,
        Employees = g.Select(e => e.Name).ToList()
    });
```

Now the result contains:

```text
Engineering
 ├── Alice
 └── Charlie

Sales
 ├── Bob
 └── David
```

The inner:

```csharp
g.Select(e => e.Name)
```

is another projection.

So you can have **nested LINQ pipelines**.

---

# 15. Grouping after filtering

You can filter before grouping:

```csharp
var result = employees
    .Where(e => e.Salary >= 80000)
    .GroupBy(e => e.Department);
```

Pipeline:

```text
Employees
   ↓
Salary >= 80000
   ↓
Filtered Employees
   ↓
GroupBy Department
   ↓
Groups
```

This is often exactly what you want.

For example:

> Group only employees earning at least ₹80,000 by department.

---

# 16. Filtering groups after grouping

There's another possibility.

Suppose:

> Show only departments containing at least 5 employees.

You can do:

```csharp
var result = employees
    .GroupBy(e => e.Department)
    .Where(g => g.Count() >= 5);
```

Notice what `Where()` receives now.

Before grouping:

```text
Where → Employee
```

After grouping:

```text
Where → IGrouping<string, Employee>
```

This is a very important LINQ skill:

> **Always know what type is flowing through the pipeline at that point.**

---

# 17. Filtering before vs after grouping

These are fundamentally different.

### Filter employees first

```csharp
employees
    .Where(e => e.Salary > 80000)
    .GroupBy(e => e.Department);
```

Meaning:

> Group employees whose salary is greater than 80,000.

### Group first, then filter groups

```csharp
employees
    .GroupBy(e => e.Department)
    .Where(g => g.Count() >= 5);
```

Meaning:

> Create all departments, then keep departments containing at least 5 employees.

Visual:

```text
Filter BEFORE GroupBy
Employee-level filtering


GroupBy
   ↓

Filter AFTER GroupBy
Group-level filtering
```

This distinction maps closely to the difference between **row-level filtering** and **group-level filtering** in SQL.

---

# 18. Grouping by multiple properties

Suppose you want to group employees by:

```text
Department + JobTitle
```

You can use an anonymous object as the key:

```csharp
var groups = employees
    .GroupBy(e => new
    {
        e.Department,
        e.JobTitle
    });
```

Now the key contains both values:

```text
{
    Department = "Engineering",
    JobTitle = "Developer"
}
```

Conceptually:

```text
Engineering + Developer
Engineering + Manager
Sales       + Developer
Sales       + Manager
```

This is called a **composite key**.

---

# 19. Grouping by an anonymous key

You can access the properties:

```csharp
foreach (var group in groups)
{
    Console.WriteLine(group.Key.Department);
    Console.WriteLine(group.Key.JobTitle);
}
```

This works because anonymous types provide appropriate value-based equality semantics for their properties.

That makes them convenient for composite grouping keys.

---

# 20. Grouping by a calculated key

The key doesn't have to be a property.

You can calculate it.

For example, categorize employees by salary:

```csharp
var groups = employees
    .GroupBy(e =>
        e.Salary >= 100000
            ? "High"
            : "Standard");
```

Result:

```text
High
 ├── Charlie
 └── ...

Standard
 ├── Alice
 ├── Bob
 └── ...
```

So:

```text
Employee
   ↓
calculate category
   ↓
GroupBy(category)
```

---

# 21. Grouping strings by first character

Another simple example:

```csharp
var names = new[]
{
    "Alice",
    "Adam",
    "Bob",
    "Charlie",
    "Chris"
};

var groups = names
    .GroupBy(name => name[0]);
```

Conceptually:

```text
A
 ├── Alice
 └── Adam

B
 └── Bob

C
 ├── Charlie
 └── Chris
```

The key is:

```csharp
name[0]
```

---

# 22. Group ordering vs element ordering

Suppose:

```csharp
var groups = employees
    .GroupBy(e => e.Department);
```

You can order the groups:

```csharp
var result = employees
    .GroupBy(e => e.Department)
    .OrderBy(g => g.Key);
```

Now you're sorting the **groups** by their keys.

You can also order elements inside each group:

```csharp
var result = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        Department = g.Key,
        Employees = g.OrderBy(e => e.Name)
    });
```

Now:

```text
Engineering
    Alice
    Charlie

Sales
    Bob
    David
```

This distinction is important:

```text
Order groups
→ OrderBy(g => g.Key)

Order elements inside groups
→ OrderBy() on g
```

---

# 23. `GroupBy()` and `ToLookup()`

There is another related LINQ operation:

```csharp
ToLookup()
```

For example:

```csharp
var lookup = employees
    .ToLookup(e => e.Department);
```

This creates an:

```csharp
ILookup<TKey, TElement>
```

A `Lookup` is useful when you want to **index elements by key** and perform repeated key-based lookups.

Conceptually:

```text
Department
    ↓
Lookup
    ↓
Engineering → employees
Sales       → employees
HR          → employees
```

For now, remember:

```text
GroupBy
→ query/group sequence

ToLookup
→ materialized lookup structure
```

We'll revisit this when discussing collection operators and performance.

---

# 24. Grouping and deferred execution

For LINQ-to-Objects:

```csharp
var groups = employees.GroupBy(e => e.Department);
```

is generally deferred.

The grouping is performed when the result is enumerated.

For example:

```csharp
foreach (var group in groups)
{
    ...
}
```

or:

```csharp
var list = groups.ToList();
```

forces the query to be evaluated.

---

# 25. Grouping is not the same as a database `GROUP BY`

This is subtle but important.

In SQL:

```sql
GROUP BY Department
```

is commonly used with aggregates:

```sql
SELECT Department, COUNT(*)
FROM Employees
GROUP BY Department;
```

In LINQ:

```csharp
employees.GroupBy(e => e.Department)
```

actually gives you **groups of elements**.

You can then decide what to do with those groups:

```csharp
.GroupBy(e => e.Department)
.Select(g => new
{
    Department = g.Key,
    Count = g.Count()
});
```

So LINQ separates:

```text
GROUP
+
WHAT TO DO WITH EACH GROUP
```

That makes the abstraction very flexible.

---

# 26. Grouping + aggregation = reporting

A very common production pattern is:

```csharp
var report = orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        OrderCount = g.Count(),
        TotalRevenue = g.Sum(o => o.Total),
        AverageOrder = g.Average(o => o.Total)
    });
```

This translates naturally to:

```text
Orders
    ↓
Group by customer
    ↓
For each customer:
    ├── count orders
    ├── sum revenue
    └── average order
```

This is one of the most important real-world uses of LINQ.

---

# 27. Nested grouping

You can group again.

For example:

```csharp
var result = employees
    .GroupBy(e => e.Department)
    .Select(g => new
    {
        Department = g.Key,

        BySalaryLevel = g.GroupBy(e =>
            e.Salary >= 100000 ? "High" : "Standard")
    });
```

Conceptually:

```text
Engineering
    High
       employees
    Standard
       employees

Sales
    High
       employees
    Standard
       employees
```

This creates hierarchical data.

It's powerful, but don't use nested grouping unless the output actually requires that hierarchy.

---

# 28. Grouping and equality

`GroupBy()` needs to determine:

> Are these two keys equal?

For:

```csharp
.GroupBy(e => e.Department)
```

keys are strings.

For:

```csharp
.GroupBy(e => e.Age)
```

keys are integers.

For:

```csharp
.GroupBy(e => new
{
    e.Department,
    e.JobTitle
})
```

the anonymous type provides appropriate equality behavior.

You can also supply a custom:

```csharp
IEqualityComparer<TKey>
```

when domain-specific equality is required.

---

# 29. Grouping with case-insensitive keys

Suppose:

```text
Engineering
engineering
ENGINEERING
```

You may want these to be one group.

You can provide a comparer:

```csharp
var groups = departments
    .GroupBy(
        d => d,
        StringComparer.OrdinalIgnoreCase);
```

Now different casing can map to the same grouping key.

This is a good example of why **equality semantics** matter in LINQ.

---

# 30. A realistic analytics query

Suppose:

```csharp
class Order
{
    public int CustomerId { get; set; }
    public string Region { get; set; }
    public decimal Total { get; set; }
    public bool IsCompleted { get; set; }
}
```

Requirement:

> For completed orders, calculate total revenue and order count per region.

```csharp
var report = orders
    .Where(o => o.IsCompleted)
    .GroupBy(o => o.Region)
    .Select(g => new
    {
        Region = g.Key,
        Orders = g.Count(),
        Revenue = g.Sum(o => o.Total)
    })
    .OrderByDescending(x => x.Revenue);
```

Pipeline:

```text
Orders
   ↓
Filter completed
   ↓
Group by Region
   ↓
Aggregate each group
   ├── Count
   └── Sum
   ↓
Report rows
   ↓
Sort by revenue
```

This is a very typical LINQ analytics pipeline.

---

# 31. The type-flow model

This is the most important technical model for `GroupBy()`:

```text
IEnumerable<TSource>
        │
        │ GroupBy(
        │     Func<TSource, TKey>
        │ )
        ▼
IEnumerable<
    IGrouping<TKey, TSource>
>
```

For:

```csharp
employees.GroupBy(e => e.Department)
```

the types are:

```text
TSource = Employee
TKey    = string
```

Therefore:

```text
IEnumerable<Employee>
        ↓
GroupBy(Employee → string)
        ↓
IEnumerable<IGrouping<string, Employee>>
```

Then:

```csharp
.Select(g => ...)
```

receives:

```text
IGrouping<string, Employee>
```

not `Employee`.

That's a critical type-flow transition.

---

# 32. What is inside `IGrouping<TKey, TElement>`?

Conceptually:

```text
IGrouping<TKey, TElement>
│
├── Key : TKey
│
└── IEnumerable<TElement>
       ├── element
       ├── element
       └── element
```

For:

```csharp
employees.GroupBy(e => e.Department)
```

you can think:

```text
IGrouping<string, Employee>
│
├── Key → "Engineering"
│
└── Employees
      ├── Alice
      └── Charlie
```

This mental model makes grouped LINQ much easier to understand.

---

# 33. Common mistake: expecting `GroupBy()` to return a dictionary

This:

```csharp
var groups = employees.GroupBy(e => e.Department);
```

does **not** give you:

```text
Dictionary<string, List<Employee>>
```

It gives you a sequence of groups:

```text
IEnumerable<IGrouping<string, Employee>>
```

If you actually need a dictionary:

```csharp
var dictionary = employees
    .GroupBy(e => e.Department)
    .ToDictionary(
        g => g.Key,
        g => g.ToList());
```

Now you have:

```text
Dictionary<string, List<Employee>>
```

The distinction matters.

---

# 34. Grouping vs dictionary lookup

Use grouping when you're doing a **query/reporting pipeline**:

```csharp
employees
    .GroupBy(...)
    .Select(...);
```

Use a lookup/dictionary when your goal is **repeated key-based access**:

```csharp
dictionary["Engineering"]
```

Different abstraction, different intent.

---

# 35. Phase 12 mental model

Keep this picture:

```text
                 IEnumerable<T>
                       │
                       │
                  GroupBy(key)
                       │
                       ▼
          IEnumerable<IGrouping<TKey,T>>
                       │
             ┌─────────┴─────────┐
             │                   │
          group.Key          group elements
             │                   │
             ▼                   ▼
         grouping key       IEnumerable<T>
             │                   │
             └─────────┬─────────┘
                       ▼
                    Select
                       │
             ┌─────────┼──────────┐
             ▼         ▼          ▼
           Count       Sum      Average
             │         │          │
             └─────────┼──────────┘
                       ▼
                  Report object
```

---

# Phase 12 checkpoint

Given:

```csharp
var result = orders
    .Where(o => o.Status == "Completed")
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        OrderCount = g.Count(),
        TotalSpent = g.Sum(o => o.Total)
    })
    .OrderByDescending(x => x.TotalSpent);
```

You should be able to mentally execute it as:

```text
IEnumerable<Order>
        ↓
Where
        ↓
Completed Orders
        ↓
GroupBy(CustomerId)
        ↓
IEnumerable<IGrouping<int, Order>>
        ↓
Select(each group)
        ↓
{
    CustomerId,
    OrderCount,
    TotalSpent
}
        ↓
OrderByDescending(TotalSpent)
        ↓
Customer report
```

### The three rules to lock in

1. **`GroupBy()` partitions elements by a key.**
    
2. **`IGrouping<TKey, TElement>` = one key + the elements belonging to it.**
    
3. **After `GroupBy()`, your next LINQ operator is operating on groups, not individual elements.**
    

And the most important production pattern is:

```csharp
source
    .Where(...)          // filter elements
    .GroupBy(...)        // create groups
    .Select(g => ...)    // calculate each group's result
    .OrderBy(...)        // order the results
```

That pattern is the backbone of a huge amount of LINQ-based reporting and data transformation.