
# Phase 15 — `GroupJoin` in LINQ

`GroupJoin()` deserves its own phase because it is one of the most important operators for understanding **one-to-many relationships** in LINQ.

The central idea:

> **`GroupJoin()` matches every element in the outer sequence with all matching elements from the inner sequence, while preserving the outer element.**

If `Join()` gives you **matching pairs**, `GroupJoin()` gives you **one outer element + a collection of matches**.

---

# 1. `Join()` vs `GroupJoin()`

This is the first thing to lock in.

Suppose:

```text
Customers

1 → Alice
2 → Bob
3 → Charlie
```

and:

```text
Orders

101 → CustomerId 1
102 → CustomerId 1
103 → CustomerId 2
```

### `Join()`

```text
Alice → Order 101
Alice → Order 102
Bob   → Order 103
```

Each matching pair becomes a result.

### `GroupJoin()`

```text
Alice
 ├── Order 101
 └── Order 102

Bob
 └── Order 103

Charlie
 └── empty
```

Each **customer remains one result**, with all matching orders attached.

That is the fundamental difference.

---

# 2. Basic `GroupJoin()`

```csharp
var result = customers.GroupJoin(
    orders,
    customer => customer.Id,
    order => order.CustomerId,
    (customer, customerOrders) => new
    {
        Customer = customer.Name,
        Orders = customerOrders
    });
```

The arguments are:

```text
customers
    ↓
outer sequence

orders
    ↓
inner sequence

customer => customer.Id
    ↓
outer key

order => order.CustomerId
    ↓
inner key

(customer, customerOrders) => ...
    ↓
result selector
```

---

# 3. The type transformation

Suppose:

```text
Customer
Order
```

Then:

```csharp
customers.GroupJoin(...)
```

conceptually produces:

```text
IEnumerable<TResult>
```

where the result selector receives:

```text
Customer
+
IEnumerable<Order>
```

So mentally:

```text
Customer
   +
IEnumerable<Order>
   ↓
one result
```

That's the essence of `GroupJoin()`.

---

# 4. The `GroupJoin()` signature

The important overload is conceptually:

```csharp
GroupJoin(
    inner,
    outerKeySelector,
    innerKeySelector,
    resultSelector
)
```

For example:

```csharp
customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (c, orders) => ...
);
```

The result selector receives:

```text
c
↓
one Customer

orders
↓
IEnumerable<Order>
```

The second parameter is **not one order**.

It's the entire matching sequence.

---

# 5. The most important difference

Compare these two result selectors.

### `Join()`

```csharp
(c, o) => new
{
    Customer = c.Name,
    Order = o.Id
}
```

Here:

```text
o → one Order
```

### `GroupJoin()`

```csharp
(c, orders) => new
{
    Customer = c.Name,
    Orders = orders
}
```

Here:

```text
orders → IEnumerable<Order>
```

This difference is everything.

---

# 6. One-to-many relationships

`GroupJoin()` naturally models:

```text
Customer
   │
   ├── Order
   ├── Order
   └── Order
```

Other examples:

```text
Department
   │
   ├── Employee
   ├── Employee
   └── Employee
```

```text
Author
   │
   ├── Book
   ├── Book
   └── Book
```

```text
Category
   │
   ├── Product
   ├── Product
   └── Product
```

Whenever you have:

```text
ONE parent → MANY children
```

`GroupJoin()` should come to mind.

---

# 7. `GroupJoin()` preserves unmatched outer elements

This is one of its most important properties.

Suppose:

```text
Customers:
Alice
Bob
Charlie

Orders:
Alice → Order 1
Bob   → Order 2
```

Then:

```csharp
customers.GroupJoin(...)
```

produces:

```text
Alice
 └── Order 1

Bob
 └── Order 2

Charlie
 └── empty
```

Charlie remains.

That's because `GroupJoin()` is naturally **outer-oriented**.

---

# 8. This is why `GroupJoin()` is useful for reporting

Suppose you need:

> Show every customer and how many orders they have.

```csharp
var result = customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (customer, customerOrders) => new
    {
        Customer = customer.Name,
        OrderCount = customerOrders.Count()
    });
```

Result:

```text
Alice   → 2
Bob     → 1
Charlie → 0
```

Notice Charlie.

A regular `Join()` would have omitted Charlie because there are no matching orders.

---

# 9. `GroupJoin()` + `Sum()`

Now suppose:

> Calculate total spending per customer.

```csharp
var result = customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (customer, customerOrders) => new
    {
        Customer = customer.Name,
        TotalSpent = customerOrders.Sum(o => o.Total)
    });
```

Conceptually:

```text
Alice
  Orders:
    500
    300
      ↓
    Sum = 800

Bob
  Orders:
    700
      ↓
    Sum = 700

Charlie
  Orders:
    none
      ↓
    Sum = 0
```

This is an extremely useful pattern.

---

# 10. `GroupJoin()` + multiple aggregations

You can calculate several metrics:

```csharp
var result = customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (customer, customerOrders) => new
    {
        Customer = customer.Name,
        OrderCount = customerOrders.Count(),
        TotalSpent = customerOrders.Sum(o => o.Total),
        AverageOrder = customerOrders.Any()
            ? customerOrders.Average(o => o.Total)
            : 0
    });
```

Each customer becomes one report row.

Conceptually:

```text
Customer
   ↓
matching Orders
   ↓
┌───────────────┐
│ Count         │
│ Sum           │
│ Average       │
└───────────────┘
   ↓
one report row
```

---

# 11. Why `Any()` before `Average()`?

Remember:

```csharp
Average()
```

on an empty non-nullable numeric sequence throws.

So:

```csharp
customerOrders.Any()
    ? customerOrders.Average(o => o.Total)
    : 0
```

protects the empty case.

Alternatively, depending on the desired semantics, you can project to a nullable value or use other explicit handling.

The key lesson:

> **GroupJoin guarantees the outer element exists, but its matching inner sequence can be empty.**

---

# 12. Filtering the inner sequence

You can filter orders before the join:

```csharp
var result = customers.GroupJoin(
    orders.Where(o => o.IsCompleted),
    c => c.Id,
    o => o.CustomerId,
    (customer, completedOrders) => new
    {
        Customer = customer.Name,
        Count = completedOrders.Count(),
        Revenue = completedOrders.Sum(o => o.Total)
    });
```

This means:

> For every customer, consider only completed orders.

Result:

```text
Alice   → completed orders
Bob     → completed orders
Charlie → none
```

---

# 13. Filtering after `GroupJoin()`

You can also filter the resulting outer records:

```csharp
var result = customers
    .GroupJoin(
        orders,
        c => c.Id,
        o => o.CustomerId,
        (customer, customerOrders) => new
        {
            Customer = customer,
            Orders = customerOrders
        })
    .Where(x => x.Orders.Any());
```

This means:

> Keep only customers who have at least one order.

Now Charlie disappears.

So:

```text
Filter INNER sequence
→ changes which children participate

Filter RESULT
→ changes which parents remain
```

---

# 14. Filtering children inside the result

You can also leave the outer customer intact and filter its children:

```csharp
var result = customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (customer, customerOrders) => new
    {
        Customer = customer.Name,
        LargeOrders = customerOrders
            .Where(o => o.Total > 10000)
            .ToList()
    });
```

Now:

```text
Alice
 ├── large order
 └── large order

Bob
 └── no large orders

Charlie
 └── no large orders
```

The parent remains even if its filtered child collection is empty.

---

# 15. `GroupJoin()` + `SelectMany()`

This is one of the most important advanced patterns.

`GroupJoin()` produces:

```text
Customer
   +
IEnumerable<Order>
```

But sometimes you want:

```text
Customer + Order
Customer + Order
Customer + Order
```

You can flatten it with:

```csharp
.SelectMany(...)
```

This gives you the conceptual relationship:

```text
GroupJoin
   ↓
parent + collection
   ↓
SelectMany
   ↓
parent + individual child
```

This is the foundation of the traditional LINQ **left outer join** pattern.

---

# 16. Traditional left outer join

The classic pattern:

```csharp
var result = customers
    .GroupJoin(
        orders,
        c => c.Id,
        o => o.CustomerId,
        (customer, customerOrders) => new
        {
            customer,
            customerOrders
        })
    .SelectMany(
        x => x.customerOrders.DefaultIfEmpty(),
        (x, order) => new
        {
            Customer = x.customer.Name,
            OrderId = order?.Id
        });
```

This looks complicated, so understand it in pieces.

### Step 1

```csharp
GroupJoin(...)
```

produces:

```text
Alice → [Order 101, Order 102]
Bob   → [Order 103]
Charlie → []
```

### Step 2

```csharp
DefaultIfEmpty()
```

turns:

```text
Charlie → []
```

into conceptually:

```text
Charlie → [null]
```

### Step 3

`SelectMany()` flattens:

```text
Alice   → Order 101
Alice   → Order 102
Bob     → Order 103
Charlie → null
```

That's a left outer join.

---

# 17. Why `DefaultIfEmpty()` is essential

Without:

```csharp
DefaultIfEmpty()
```

the empty collection:

```text
Charlie → []
```

would produce no flattened row.

With it:

```text
Charlie → null
```

produces one row.

So:

```text
GroupJoin
   ↓
preserve parent
   ↓
DefaultIfEmpty
   ↓
represent missing child
   ↓
SelectMany
   ↓
flatten
```

---

# 18. Modern `LeftJoin()`

In current .NET APIs, a dedicated:

```csharp
LeftJoin()
```

is available for LINQ-to-Objects.

So where supported, you can express the same intent much more directly:

```csharp
var result = customers.LeftJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (customer, order) => new
    {
        Customer = customer.Name,
        Order = order
    });
```

This is conceptually:

```text
Every customer
+
matching order if available
```

For new code targeting a framework version that supports it, this can be clearer than the traditional `GroupJoin()` pattern when you actually want flattened left-join results.

---

# 19. `GroupJoin()` itself is not a left join

This distinction is subtle.

People sometimes say:

> "`GroupJoin()` is a left join."

Not exactly.

`GroupJoin()` produces:

```text
outer element
+
collection of matching inner elements
```

A traditional left outer join is constructed from:

```text
GroupJoin
+
DefaultIfEmpty
+
SelectMany
```

So:

```text
GroupJoin ≠ flattened Left Join
```

But `GroupJoin()` is the key building block for the classic implementation.

---

# 20. `GroupJoin()` vs `GroupBy()`

These can look similar because both produce groups.

But they're fundamentally different.

### `GroupBy()`

One sequence:

```text
Orders
   ↓
GroupBy(CustomerId)
   ↓
CustomerId → Orders
```

### `GroupJoin()`

Two sequences:

```text
Customers + Orders
       ↓
GroupJoin
       ↓
Customer → matching Orders
```

So:

```text
GroupBy
→ partition ONE sequence

GroupJoin
→ relate TWO sequences
```

---

# 21. `GroupJoin()` vs `Join()`

This distinction should be automatic now:

||`Join()`|`GroupJoin()`|
|---|---|---|
|Outer element preserved|Only when matched|Yes|
|Inner matches|Individual|Collection|
|Result per outer element|Potentially many|Exactly one|
|Unmatched outer element|Removed|Preserved|
|Natural use|Pairing|One-to-many|

Example:

```text
Join:

Alice + Order 1
Alice + Order 2
Bob   + Order 3
```

```text
GroupJoin:

Alice + [Order 1, Order 2]
Bob   + [Order 3]
Charlie + []
```

---

# 22. `GroupJoin()` with composite keys

Just like `Join()`, you can use multiple key fields.

Suppose:

```csharp
class Enrollment
{
    public int StudentId { get; set; }
    public int CourseId { get; set; }
}

class Grade
{
    public int StudentId { get; set; }
    public int CourseId { get; set; }
    public string Value { get; set; }
}
```

Join by:

```text
StudentId + CourseId
```

```csharp
var result = enrollments.GroupJoin(
    grades,
    e => new
    {
        e.StudentId,
        e.CourseId
    },
    g => new
    {
        g.StudentId,
        g.CourseId
    },
    (enrollment, gradesForEnrollment) => new
    {
        enrollment.StudentId,
        enrollment.CourseId,
        Grades = gradesForEnrollment
    });
```

The same composite-key principles from advanced `GroupBy()` apply.

---

# 23. `GroupJoin()` with custom equality

You can also provide an:

```csharp
IEqualityComparer<TKey>
```

when the default key equality isn't appropriate.

For example, case-insensitive string matching:

```csharp
outer.GroupJoin(
    inner,
    x => x.Code,
    y => y.Code,
    (x, matches) => ...,
    StringComparer.OrdinalIgnoreCase);
```

Conceptually:

```text
"ABC"
  ↕
"abc"
```

can be considered equal.

Again, the comparer defines **what "matching" means**.

---

# 24. GroupJoin and cardinality

This is an important mental model.

Suppose:

```text
Customers = N
Orders = M
```

`GroupJoin()` produces:

```text
N result elements
```

because it creates one result for every outer element.

The number of orders doesn't directly change the number of outer results.

For example:

```text
Alice   → 100 orders
Bob     → 3 orders
Charlie → 0 orders
```

still produces:

```text
3 outer results
```

The difference is how many elements are inside each matching collection.

This is a crucial difference from `Join()`.

---

# 25. `Join()` cardinality

With regular `Join()`:

```text
Alice → 100 orders
```

produces approximately:

```text
100 result elements
```

assuming all 100 match.

So:

```text
Join
→ result count depends on matching pairs

GroupJoin
→ result count equals outer count
```

That's a very useful invariant.

---

# 26. Nested child processing

Because `customerOrders` is an `IEnumerable<Order>`, you can run an entire LINQ pipeline against it.

For example:

```csharp
var result = customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (customer, customerOrders) => new
    {
        Customer = customer.Name,

        LatestOrder = customerOrders
            .OrderByDescending(o => o.CreatedAt)
            .FirstOrDefault(),

        LargeOrderCount = customerOrders
            .Count(o => o.Total > 10000)
    });
```

Each customer's child collection becomes its own mini-LINQ pipeline.

This is one of the most powerful aspects of `GroupJoin()`.

---

# 27. `GroupJoin()` and hierarchical output

You can build a nested DTO:

```csharp
var result = customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (customer, customerOrders) => new CustomerDto
    {
        Id = customer.Id,
        Name = customer.Name,
        Orders = customerOrders
            .Select(o => new OrderDto
            {
                Id = o.Id,
                Total = o.Total
            })
            .ToList()
    });
```

The resulting structure is:

```text
CustomerDto
│
├── Id
├── Name
└── Orders
      ├── OrderDto
      ├── OrderDto
      └── OrderDto
```

This is particularly useful when constructing:

- API response models
    
- hierarchical DTOs
    
- reports
    
- view models
    
- tree-like structures
    

---

# 28. `GroupJoin()` and `SelectMany()`

These operators are closely related conceptually:

```text
GroupJoin
→ parent + children collection

SelectMany
→ flatten children collections
```

You can think of:

```text
GroupJoin
```

as preserving the hierarchy:

```text
Parent
 └── Children
```

while:

```text
SelectMany
```

can turn it into:

```text
Parent + Child
Parent + Child
Parent + Child
```

Understanding this relationship will make complex LINQ pipelines much easier.

---

# 29. `GroupJoin()` and databases

With:

```csharp
IQueryable<T>
```

a provider such as Entity Framework Core may translate suitable `GroupJoin()` patterns into SQL joins.

However, arbitrary hierarchical `GroupJoin()` projections are not always equivalent to simply asking the database for a flat SQL join.

The provider needs to translate the expression tree into a supported query shape.

Therefore:

```text
LINQ-to-Objects
→ actual grouping in memory

LINQ-to-Entities
→ potentially translated into SQL
```

Always keep the execution environment in mind.

---

# 30. Performance considerations

For LINQ-to-Objects:

```csharp
customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    ...
);
```

is not equivalent to repeatedly doing:

```csharp
orders.Where(o => o.CustomerId == customer.Id)
```

for every customer.

The latter can lead to repeatedly scanning the orders collection.

A join implementation can use an indexed/hash-based strategy internally.

Conceptually:

```text
Orders
   ↓
build key lookup
   ↓
Customer.Id
   ↓
find matching orders
```

So joining is designed around efficient key matching.

---

# 31. Don't accidentally enumerate child groups repeatedly

Consider:

```csharp
var result = customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (customer, customerOrders) => new
    {
        Count = customerOrders.Count(),
        Total = customerOrders.Sum(o => o.Total),
        Average = customerOrders.Average(o => o.Total)
    });
```

Depending on the exact source/provider, repeatedly enumerating `customerOrders` may have different costs.

For LINQ-to-Objects grouped matches, the implementation can often enumerate the underlying grouped collection repeatedly, but conceptually you should still recognize:

```text
Count()
Sum()
Average()
```

are three separate operations.

For expensive or custom sources, materializing once can be appropriate:

```csharp
var result = customers.GroupJoin(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (customer, customerOrders) =>
    {
        var list = customerOrders.ToList();

        return new
        {
            Customer = customer.Name,
            Count = list.Count,
            Total = list.Sum(o => o.Total)
        };
    });
```

But don't materialize blindly—especially against database queries. Provider translation and query shape matter.

---

# 32. `GroupJoin()` isn't always the best tool

Suppose your EF Core model already has:

```csharp
customer.Orders
```

You may simply write:

```csharp
var result = db.Customers
    .Select(c => new
    {
        c.Name,
        OrderCount = c.Orders.Count(),
        TotalSpent = c.Orders.Sum(o => o.Total)
    });
```

This can be clearer than manually joining:

```csharp
db.Customers.GroupJoin(...)
```

So the principle is:

> Use `GroupJoin()` when you need an explicit relationship between independent sequences; don't use it merely because the relationship happens to be one-to-many.

---

# 33. A complete real-world example

Let's build a customer report.

```csharp
var report = customers
    .GroupJoin(
        orders.Where(o => o.IsCompleted),
        customer => customer.Id,
        order => order.CustomerId,
        (customer, customerOrders) =>
        {
            var ordersList = customerOrders.ToList();

            return new
            {
                CustomerId = customer.Id,
                CustomerName = customer.Name,
                OrderCount = ordersList.Count,
                TotalRevenue = ordersList.Sum(o => o.Total),
                AverageOrderValue =
                    ordersList.Count == 0
                        ? 0
                        : ordersList.Average(o => o.Total)
            };
        })
    .OrderByDescending(x => x.TotalRevenue);
```

Read it as:

```text
Customers
     │
     │ GroupJoin
     ▼
Completed Orders
     │
     │ match:
     │ Customer.Id
     │      =
     │ Order.CustomerId
     ▼
Customer + matching completed orders
     │
     ├── Count
     ├── Sum
     └── Average
     │
     ▼
Customer report
     │
     ▼
Order by revenue
```

---

# 34. The three levels of data

After `GroupJoin()`, it's useful to think in three levels:

```text
LEVEL 1
Outer element

Customer
   │
   ▼

LEVEL 2
Matching collection

IEnumerable<Order>
   │
   ▼

LEVEL 3
Individual inner element

Order
```

Your code can move between these levels:

```csharp
customer
```

→ outer element

```csharp
customerOrders
```

→ matching collection

```csharp
customerOrders.Where(o => ...)
```

→ individual orders inside the collection

Understanding these levels is the key to mastering `GroupJoin()`.

---

# 35. `GroupJoin()` mental model

Memorize this:

```text
                OUTER
             Customers
                 │
                 │ key
                 ▼
             Customer.Id
                 │
                 │ MATCH
                 │
                 ▲
                 │ key
                 │
          Order.CustomerId
                 │
                 ▼
              INNER
              Orders
                 │
                 ▼
       matching order collection
                 │
                 ▼
      Customer + IEnumerable<Order>
```

Compare:

```text
Join:

Customer + Order
Customer + Order
Customer + Order
```

with:

```text
GroupJoin:

Customer + IEnumerable<Order>
Customer + IEnumerable<Order>
Customer + IEnumerable<Order>
```

---

# 36. Phase 15 checkpoint

Given:

```csharp
var result = customers
    .GroupJoin(
        orders,
        c => c.Id,
        o => o.CustomerId,
        (customer, customerOrders) => new
        {
            Name = customer.Name,
            Count = customerOrders.Count(),
            Total = customerOrders.Sum(o => o.Total)
        });
```

You should immediately understand:

```text
Customer
   ↓
find ALL matching orders
   ↓
customerOrders : IEnumerable<Order>
   ↓
Count + Sum
   ↓
ONE result per customer
```

And the cardinality rule:

```text
Join()
→ one result per matching pair

GroupJoin()
→ one result per outer element
   containing all matching inner elements
```

The three concepts to lock in are:

```text
1. GroupJoin preserves every outer element.

2. The matching inner elements arrive as an IEnumerable<T>.

3. GroupJoin + DefaultIfEmpty + SelectMany
   gives the classic left-outer-join pattern.
```

Once those are solid, `GroupJoin()` stops looking like a complicated LINQ method and becomes a very natural expression of **parent → children relationships**.