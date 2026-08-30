
# Phase 14 — Joining in LINQ

Joining is how LINQ combines elements from **two sequences based on a related key**.

If `GroupBy()` answers:

> "Which elements belong together?"

then `Join()` answers:

> **"Which elements from sequence A correspond to elements from sequence B?"**

This is the LINQ equivalent of the relational-database idea of a **JOIN**.

---

# 1. The basic idea

Suppose you have two collections.

### Customers

```csharp
var customers = new[]
{
    new { Id = 1, Name = "Alice" },
    new { Id = 2, Name = "Bob" },
    new { Id = 3, Name = "Charlie" }
};
```

### Orders

```csharp
var orders = new[]
{
    new { Id = 101, CustomerId = 1, Total = 500 },
    new { Id = 102, CustomerId = 2, Total = 700 },
    new { Id = 103, CustomerId = 1, Total = 300 }
};
```

Relationship:

```text
Customer.Id
      ↕
Order.CustomerId
```

We can join them:

```csharp
var result = customers.Join(
    orders,
    customer => customer.Id,
    order => order.CustomerId,
    (customer, order) => new
    {
        Customer = customer.Name,
        OrderId = order.Id,
        Total = order.Total
    });
```

Result:

```text
Alice   → Order 101 → 500
Alice   → Order 103 → 300
Bob     → Order 102 → 700
```

Charlie has no order, so regular `Join()` doesn't produce a row for Charlie.

---

# 2. The `Join()` signature

The important overload conceptually looks like:

```csharp
Join(
    inner,
    outerKeySelector,
    innerKeySelector,
    resultSelector
)
```

For:

```csharp
customers.Join(
    orders,
    customer => customer.Id,
    order => order.CustomerId,
    (customer, order) => ...
);
```

the pieces are:

```text
customers
   │
   │ outerKeySelector
   ▼
customer.Id
   │
   │       equality
   │          ↕
   │
   ▼
order.CustomerId
   ▲
   │ innerKeySelector
   │
orders
```

Then:

```csharp
(customer, order) => ...
```

determines what the joined result should look like.

---

# 3. The four parts of `Join()`

Memorize this structure:

```csharp
outer.Join(
    inner,
    outerKeySelector,
    innerKeySelector,
    resultSelector
);
```

### 1. Outer sequence

```csharp
customers
```

### 2. Inner sequence

```csharp
orders
```

### 3. Outer key

```csharp
customer => customer.Id
```

### 4. Inner key

```csharp
order => order.CustomerId
```

### 5. Result projection

```csharp
(customer, order) => new { ... }
```

The final selector is essentially:

> "What should each matching pair become?"

---

# 4. Type flow

This is especially important.

Suppose:

```text
customers → Customer
orders    → Order
```

Then:

```csharp
customers.Join(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (c, o) => new { c.Name, o.Total }
);
```

conceptually becomes:

```text
IEnumerable<Customer>
        +
IEnumerable<Order>
        ↓
      Join
        ↓
IEnumerable<AnonymousType>
```

Unlike `GroupBy()`, you're not producing groups.

You're producing **matching pairs**.

---

# 5. Think in terms of pairs

Suppose:

```text
Customers:

1 → Alice
2 → Bob

Orders:

101 → CustomerId 1
102 → CustomerId 1
103 → CustomerId 2
```

Join:

```text
Customer 1 ↔ Order 101
Customer 1 ↔ Order 102
Customer 2 ↔ Order 103
```

Result:

```text
Alice ↔ Order 101
Alice ↔ Order 102
Bob   ↔ Order 103
```

This is the core mental model:

```text
A ──key──┐
         ├── MATCH ──→ result
B ──key──┘
```

---

# 6. `Join()` is an inner join

Regular LINQ:

```csharp
customers.Join(orders, ...)
```

behaves like an **INNER JOIN**.

That means:

> Only records having a matching key on both sides appear.

Suppose:

```text
Customers:

1 Alice
2 Bob
3 Charlie

Orders:

101 → Customer 1
102 → Customer 2
```

Result:

```text
Alice → 101
Bob   → 102
```

Charlie disappears because:

```text
Charlie
   ↓
no matching Order
   ↓
not included
```

---

# 7. SQL mental model

The LINQ:

```csharp
var result = customers.Join(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (c, o) => new
    {
        c.Name,
        o.Total
    });
```

conceptually corresponds to:

```sql
SELECT
    c.Name,
    o.Total
FROM Customers c
INNER JOIN Orders o
    ON c.Id = o.CustomerId;
```

The correspondence is:

```text
LINQ                         SQL

Join()                   →   JOIN

c => c.Id                →   c.Id

o => o.CustomerId        →   o.CustomerId

resultSelector           →   SELECT
```

This is one reason LINQ is so useful for database programming.

---

# 8. One-to-many joins

The most common relationship is:

```text
One Customer
    ↓
Many Orders
```

For:

```text
Alice → Order 101
Alice → Order 103
```

the join produces **two result elements** for Alice.

So a join isn't necessarily:

```text
1 customer → 1 result
```

It can be:

```text
1 customer → N results
```

because every matching inner element produces a result.

---

# 9. Many-to-many joins

Suppose:

```text
Students
Courses
StudentCourses
```

The relationship is:

```text
Student
   ↓
StudentCourse
   ↓
Course
```

You can perform multiple joins:

```csharp
var result = students
    .Join(
        studentCourses,
        student => student.Id,
        sc => sc.StudentId,
        (student, sc) => new
        {
            student,
            sc.CourseId
        })
    .Join(
        courses,
        x => x.CourseId,
        course => course.Id,
        (x, course) => new
        {
            Student = x.student.Name,
            Course = course.Name
        });
```

Conceptually:

```text
Student
   ↓
StudentCourse
   ↓
Course
```

Result:

```text
Alice → C# Programming
Alice → Databases
Bob   → Networking
```

---

# 10. Multiple joins

You can chain joins.

Example:

```csharp
var result = orders
    .Join(
        customers,
        order => order.CustomerId,
        customer => customer.Id,
        (order, customer) => new
        {
            order,
            customer
        })
    .Join(
        products,
        x => x.order.ProductId,
        product => product.Id,
        (x, product) => new
        {
            Customer = x.customer.Name,
            Product = product.Name,
            Total = x.order.Total
        });
```

Pipeline:

```text
Orders
   ↓
Join Customers
   ↓
Customer + Order
   ↓
Join Products
   ↓
Customer + Order + Product
```

This is very common in relational data processing.

---

# 11. Query syntax

LINQ also has query syntax.

The previous join can be written:

```csharp
var result =
    from customer in customers
    join order in orders
        on customer.Id equals order.CustomerId
    select new
    {
        Customer = customer.Name,
        OrderId = order.Id,
        Total = order.Total
    };
```

This is very close to SQL:

```text
FROM customer
JOIN order
ON customer.Id = order.CustomerId
SELECT ...
```

---

# 12. Method syntax vs query syntax

Method syntax:

```csharp
customers.Join(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (c, o) => new
    {
        Customer = c.Name,
        OrderId = o.Id
    });
```

Query syntax:

```csharp
from c in customers
join o in orders
    on c.Id equals o.CustomerId
select new
{
    Customer = c.Name,
    OrderId = o.Id
};
```

Both represent LINQ.

For complex pipelines, method syntax often gives you more uniform composition.

For multiple joins and SQL-like transformations, query syntax can sometimes be easier to read.

Know both.

---

# 13. `join ... equals ...`

Notice this syntax:

```csharp
join order in orders
    on customer.Id equals order.CustomerId
```

Query syntax uses:

```text
equals
```

rather than:

```csharp
==
```

So:

```csharp
on customer.Id equals order.CustomerId
```

is the query-expression syntax for the join key equality.

---

# 14. Joining on different property names

This is extremely common.

```text
Customer:
Id

Order:
CustomerId
```

You can join:

```csharp
customers.Join(
    orders,
    c => c.Id,
    o => o.CustomerId,
    ...
);
```

The property names don't need to be the same.

Only the **key values** need to match according to the equality comparer.

---

# 15. Key types must be compatible

For:

```csharp
c => c.Id
```

and:

```csharp
o => o.CustomerId
```

the key types need to be compatible.

For example:

```text
Customer.Id       → int
Order.CustomerId  → int
```

works naturally.

But:

```text
Customer.Id       → int
Order.CustomerId  → string
```

doesn't directly work as the same join key.

You would need an appropriate conversion, assuming that conversion makes domain sense.

---

# 16. Joining with anonymous composite keys

You can join on multiple columns.

Suppose:

```text
Enrollment:
StudentId
CourseId

Grades:
StudentId
CourseId
Grade
```

The relationship is based on:

```text
StudentId + CourseId
```

Use:

```csharp
var result = enrollments.Join(
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
    (e, g) => new
    {
        e.StudentId,
        e.CourseId,
        g.Grade
    });
```

This is the join equivalent of a composite grouping key.

---

# 17. Why anonymous composite keys work here

Both sides create the same anonymous type shape:

```csharp
new
{
    StudentId,
    CourseId
}
```

So equality is based on both properties.

Conceptually:

```text
(StudentId=1, CourseId=10)
             ↕
(StudentId=1, CourseId=10)
```

match.

But:

```text
(StudentId=1, CourseId=10)
             ↕
(StudentId=1, CourseId=20)
```

do not.

---

# 18. `GroupJoin()` — the important advanced join

Now we reach an important operator:

```csharp
GroupJoin()
```

`Join()` produces:

```text
one matching pair → one result
```

`GroupJoin()` produces:

```text
one outer element → one result containing its matching inner elements
```

This is extremely important.

---

# 19. `Join()` vs `GroupJoin()`

Suppose:

```text
Alice
  ├── Order 101
  └── Order 103

Bob
  └── Order 102
```

### `Join()`

Produces:

```text
Alice + Order 101
Alice + Order 103
Bob   + Order 102
```

### `GroupJoin()`

Produces:

```text
Alice
  → [Order 101, Order 103]

Bob
  → [Order 102]
```

The difference:

```text
Join
→ flatten matching pairs

GroupJoin
→ preserve outer element + collection of matches
```

---

# 20. `GroupJoin()` example

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

Now each result contains:

```text
Customer
   +
IEnumerable<Order>
```

For Alice:

```text
Alice
 ├── Order 101
 └── Order 103
```

For Bob:

```text
Bob
 └── Order 102
```

For Charlie with no orders:

```text
Charlie
 └── empty sequence
```

This last behavior makes `GroupJoin()` extremely useful.

---

# 21. `GroupJoin()` is naturally one-to-many

This makes it ideal for relationships like:

```text
Customer → Orders
Department → Employees
Category → Products
Author → Books
University → Students
```

Example:

```csharp
var departments = employees
    .GroupBy(e => e.Department);
```

is grouping one sequence.

But:

```csharp
departments.GroupJoin(...)
```

can combine two independently sourced sequences while preserving the parent-child relationship.

---

# 22. Simulating a left outer join

LINQ doesn't have a direct `LeftJoin()` in the classic `Enumerable` API.

A traditional pattern is:

```csharp
var result =
    customers
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

The important pieces are:

```text
GroupJoin
    ↓
matching orders per customer

DefaultIfEmpty
    ↓
keep customer even when no order exists

SelectMany
    ↓
flatten the result
```

So conceptually:

```text
Customers
   +
Orders
   ↓
LEFT JOIN
```

---

# 23. Why `DefaultIfEmpty()` matters

Without:

```csharp
DefaultIfEmpty()
```

a customer with zero orders would disappear when flattened.

With it:

```text
Customer with orders
→ one result per order

Customer without orders
→ one result with default/null order
```

This connects directly to the `DefaultIfEmpty()` operator from earlier.

---

# 24. Modern .NET left join

In modern .NET versions, there is also a dedicated `LeftJoin()` API available for LINQ-to-Objects.

Conceptually:

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

This directly expresses:

> Keep every customer, matching an order where one exists.

When targeting a framework/provider version where `LeftJoin()` is available and supported, it is clearer than the older `GroupJoin()` + `SelectMany()` + `DefaultIfEmpty()` pattern.

---

# 25. Full join concept

A **full outer join** means:

```text
all left elements
+
all matching right elements
+
unmatched right elements
```

Classic LINQ-to-Objects doesn't provide a simple built-in `FullJoin()` equivalent in the same way as `Join()`.

It generally requires composing multiple operations or using another abstraction/provider.

Conceptually:

```text
Left Join
    +
Right-only rows
```

You should understand the relational concept even if you don't implement it frequently.

---

# 26. Join cardinality

This is a very important database concept.

Suppose:

```text
Customer A
```

has:

```text
3 Orders
```

Then:

```text
Customer A × Orders
```

produces:

```text
3 joined results
```

If both sides have duplicate matching keys, the number of results can multiply.

Suppose:

```text
A has 2 matching rows
B has 3 matching rows
```

Then an equality join can produce:

```text
2 × 3 = 6
```

pairs.

This is one of the most common sources of unexpected row multiplication in SQL and LINQ.

---

# 27. Example of accidental multiplication

Suppose:

```text
Customer 1
```

appears twice in `customers` and has:

```text
3 orders
```

A join can produce:

```text
2 customer rows × 3 orders
= 6 results
```

The join isn't "wrong."

Your data/cardinality assumptions may be wrong.

Always ask:

> Is this relationship one-to-one, one-to-many, or many-to-many?

---

# 28. Join vs navigation properties

In Entity Framework Core, you may have:

```csharp
customer.Orders
```

through a navigation property.

In that situation, you may not need an explicit LINQ `Join()`.

For example:

```csharp
var result = db.Customers
    .Select(c => new
    {
        c.Name,
        Orders = c.Orders
    });
```

The ORM can understand the relationship.

Explicit `Join()` is still useful when:

- relationships aren't modeled as navigations
    
- joining arbitrary query sources
    
- expressing specific relational logic
    
- working with LINQ-to-Objects
    
- building provider-translatable queries where it improves clarity
    

---

# 29. Join + aggregation

A very common pattern:

> Calculate total order value per customer.

You could write:

```csharp
var result = customers
    .GroupJoin(
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
Customer
   ↓
find matching orders
   ↓
Sum(order.Total)
   ↓
one result per customer
```

Result:

```text
Alice   → ₹800
Bob     → ₹700
Charlie → ₹0
```

This is an extremely useful combination:

```text
Join relationship
+
Aggregation
=
summary per parent
```

---

# 30. Join + filtering

Suppose you only want completed orders:

```csharp
var result = customers
    .Join(
        orders.Where(o => o.IsCompleted),
        c => c.Id,
        o => o.CustomerId,
        (c, o) => new
        {
            Customer = c.Name,
            Order = o
        });
```

Notice:

```csharp
orders.Where(...)
```

happens before the join.

This means only completed orders participate in matching.

---

# 31. Filtering after the join

You can also filter after joining:

```csharp
var result = customers
    .Join(
        orders,
        c => c.Id,
        o => o.CustomerId,
        (c, o) => new
        {
            Customer = c,
            Order = o
        })
    .Where(x => x.Order.Total > 10000);
```

Now the condition operates on the **joined result**.

Again, the placement of `Where()` matters.

---

# 32. Join + projection

The final selector is a projection:

```csharp
(c, o) => new
{
    CustomerName = c.Name,
    OrderTotal = o.Total
}
```

So a join pipeline frequently looks like:

```text
Customers
     +
Orders
     ↓
Join
     ↓
Customer + Order
     ↓
Where
     ↓
Select
     ↓
Final DTO/report
```

---

# 33. Joining with a custom comparer

Like `GroupBy()` and `Contains()`, joins can use an equality comparer.

Conceptually:

```csharp
outer.Join(
    inner,
    outerKeySelector,
    innerKeySelector,
    resultSelector,
    comparer
);
```

This matters for things like:

```text
case-insensitive strings
custom value objects
domain-specific equality
```

For example, if codes should be case-insensitive, a suitable string comparer can define that behavior.

---

# 34. Join internally relies on equality

The conceptual operation is:

```text
outer key
     ↓
compare with inner keys
     ↓
matching keys
     ↓
result
```

For LINQ-to-Objects, you can think in terms of a hash-based lookup:

```text
Inner sequence
     ↓
build key index
     ↓
lookup outer key
     ↓
produce matches
```

This is why joins can be efficient compared with naïvely comparing every possible pair.

The implementation details vary, but the hash-join mental model is useful.

---

# 35. Join complexity

Suppose:

```text
Outer = N elements
Inner = M elements
```

A naïve nested-loop join could conceptually require:

```text
N × M
```

comparisons.

A hash-based approach can often be closer to:

```text
N + M
```

average-case work for equality joins, plus the cost of producing the matching results.

But the actual performance depends on:

- source types
    
- comparer
    
- key distribution
    
- number of matches
    
- provider
    
- database indexes
    
- query translation
    

So don't reduce LINQ performance to one Big-O formula without considering the execution environment.

---

# 36. `Join()` with `IEnumerable<T>`

For LINQ-to-Objects:

```csharp
IEnumerable<Customer>
IEnumerable<Order>
```

the join happens in your application process.

Data must already be available to your application.

Conceptually:

```text
Memory
 ┌───────────────┐
 │ Customers     │
 │ Orders        │
 └───────────────┘
       ↓
      Join
       ↓
     Result
```

---

# 37. `Join()` with `IQueryable<T>`

For:

```csharp
IQueryable<Customer>
IQueryable<Order>
```

the query provider can potentially translate the join to SQL.

Conceptually:

```text
C# LINQ
   ↓
Expression tree
   ↓
Query provider
   ↓
SQL JOIN
   ↓
Database
```

This is a major distinction.

For example:

```csharp
var result = db.Customers
    .Join(
        db.Orders,
        c => c.Id,
        o => o.CustomerId,
        (c, o) => new
        {
            c.Name,
            o.Total
        });
```

can conceptually become:

```sql
SELECT
    c.Name,
    o.Total
FROM Customers c
INNER JOIN Orders o
    ON c.Id = o.CustomerId;
```

The database performs the join.

---

# 38. Why execution context matters

This:

```csharp
IEnumerable<Customer> customers
```

and:

```csharp
IQueryable<Customer> customers
```

may look similar syntactically.

But:

```text
IEnumerable
→ execute C# delegates against objects

IQueryable
→ build expressions for a query provider
```

Therefore the same-looking LINQ code can have very different execution characteristics.

This becomes a major topic when we reach:

> **`IQueryable<T>` + Expression Trees + Entity Framework Core**

---

# 39. Join vs GroupBy

These are easy to confuse.

### `GroupBy`

One sequence:

```text
Employees
    ↓
GroupBy Department
    ↓
Department groups
```

You're partitioning existing elements.

### `Join`

Two sequences:

```text
Customers + Orders
       ↓
     Join
       ↓
matching pairs
```

You're relating data from separate sources.

So:

```text
GroupBy
→ partition

Join
→ relate
```

---

# 40. Join vs `SelectMany`

You can sometimes express relationship traversal using `SelectMany()`.

Suppose:

```csharp
customers.SelectMany(c => c.Orders)
```

This uses an existing object relationship:

```text
Customer
   ↓
Orders
```

A `Join()` instead explicitly relates two independent sequences:

```csharp
customers.Join(
    orders,
    c => c.Id,
    o => o.CustomerId,
    ...
);
```

So:

```text
SelectMany
→ flatten nested relationships

Join
→ match independent sequences by keys
```

---

# 41. Advanced example: Customer sales report

Suppose:

```csharp
class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
}

class Order
{
    public int CustomerId { get; set; }
    public decimal Total { get; set; }
    public bool IsCompleted { get; set; }
}
```

Requirement:

> Show every customer and their completed-order revenue.

A `GroupJoin()` is natural:

```csharp
var report = customers
    .GroupJoin(
        orders.Where(o => o.IsCompleted),
        c => c.Id,
        o => o.CustomerId,
        (customer, customerOrders) => new
        {
            Customer = customer.Name,
            OrderCount = customerOrders.Count(),
            Revenue = customerOrders.Sum(o => o.Total)
        });
```

Result:

```text
Alice    → 2 orders → ₹80,000
Bob      → 3 orders → ₹95,000
Charlie  → 0 orders → ₹0
```

Notice that Charlie remains in the result.

That's the key advantage of `GroupJoin()` here.

---

# 42. Advanced example: department + manager

Suppose:

```text
Employees
Departments
```

and:

```text
Employee.DepartmentId
Department.Id
```

You can join:

```csharp
var result = employees.Join(
    departments,
    e => e.DepartmentId,
    d => d.Id,
    (e, d) => new
    {
        Employee = e.Name,
        Department = d.Name
    });
```

Result:

```text
Alice   → Engineering
Bob     → Sales
Charlie → HR
```

This is a classic **foreign key → primary key** join.

---

# 43. Database terminology

When you see:

```text
Customer.Id
Order.CustomerId
```

think:

```text
Primary Key
     ↕
Foreign Key
```

And:

```csharp
customers.Join(
    orders,
    c => c.Id,
    o => o.CustomerId,
    ...
)
```

means:

```text
PK = FK relationship
```

This is the bridge between LINQ and relational database thinking.

---

# 44. Phase 14 type-flow map

Keep this one:

```text
                    Join
                     │
        ┌────────────┴────────────┐
        │                         │
IEnumerable<TOuter>       IEnumerable<TInner>
        │                         │
        │ outer key               │ inner key
        └────────────┬────────────┘
                     │
                  equality
                     │
                     ▼
               matching pairs
                     │
                     ▼
              resultSelector
                     │
                     ▼
             IEnumerable<TResult>
```

For:

```csharp
customers.Join(
    orders,
    c => c.Id,
    o => o.CustomerId,
    (c, o) => new { c.Name, o.Total }
);
```

the transformation is:

```text
Customer + Order
      ↓
match Customer.Id == Order.CustomerId
      ↓
{ Name, Total }
```

---

# 45. The join family

You should now have this mental map:

```text
                     JOINING
                        │
             ┌──────────┴──────────┐
             │                     │
           Join                GroupJoin
             │                     │
             ▼                     ▼
       matching pairs        outer + matches
             │                     │
             ▼                     ▼
       INNER JOIN             one-to-many
```

And for outer joins:

```text
Left Join
    ↓
keep all outer elements

Right Join
    ↓
keep all inner elements

Full Join
    ↓
keep everything
```

In modern .NET, dedicated `LeftJoin()`/`RightJoin()` APIs may be available; otherwise the traditional `GroupJoin()` + `DefaultIfEmpty()` pattern is the classic LINQ approach.

---

# Phase 14 — The critical mental model

Don't memorize `Join()` as a complicated method signature.

Think:

```text
                    TWO SEQUENCES
                         │
             ┌───────────┴───────────┐
             │                       │
          Sequence A             Sequence B
             │                       │
          extract key             extract key
             │                       │
             └───────────┬───────────┘
                         │
                      MATCH
                         │
                         ▼
                   matching pairs
                         │
                         ▼
                  project result
```

Then distinguish:

```text
Join()
→ A + matching B
→ one result per matching pair

GroupJoin()
→ A + all matching B
→ one result per A
```

And remember the relational equivalents:

```text
Join()
→ INNER JOIN

GroupJoin() + DefaultIfEmpty()
→ LEFT OUTER JOIN
```

---

## Phase 14 checkpoint

Given:

```csharp
var result = customers
    .GroupJoin(
        orders.Where(o => o.IsCompleted),
        c => c.Id,
        o => o.CustomerId,
        (customer, customerOrders) => new
        {
            Name = customer.Name,
            Orders = customerOrders.Count(),
            Revenue = customerOrders.Sum(o => o.Total)
        })
    .OrderByDescending(x => x.Revenue);
```

You should be able to trace:

```text
Customers
    +
Completed Orders
    ↓
GroupJoin(Customer.Id ↔ Order.CustomerId)
    ↓
One group per customer
    ↓
Count + Sum inside each group
    ↓
One report row per customer
    ↓
Order by revenue
```

And the most important distinction from the previous phases is:

```text
GroupBy
→ "Group elements from ONE sequence."

Join
→ "Match elements from TWO sequences."

GroupJoin
→ "Match TWO sequences while preserving the outer element
   and its collection of matches."
```

**Next logical phase: Set Operators — `Distinct`, `Union`, `Intersect`, `Except`, and `Concat`, including how equality affects them and why `Concat` is fundamentally different from `Union`.**