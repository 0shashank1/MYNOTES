
# Phase 22 — Expression Trees in C\#

Expression Trees are one of the most important concepts behind **`IQueryable<T>`**.

You've already seen:

```text
IEnumerable<T>
    ↓
Func<T, TResult>
    ↓
execute C# code
```

and:

```text
IQueryable<T>
    ↓
Expression<Func<T, TResult>>
    ↓
inspect/translate the expression
```

Now we're going underneath `IQueryable`.

> **An expression tree is a data structure that represents C# code as a tree of nodes.**

It turns code from something that can only be **executed** into something that can also be **inspected, analyzed, transformed, or translated**.

---

# 1. The core difference

Consider:

```csharp
Func<int, int> square = x => x * x;
```

This represents executable code.

Think:

```text
x
 ↓
multiply x × x
 ↓
return result
```

Now:

```csharp
Expression<Func<int, int>> square =
    x => x * x;
```

This represents the **structure of that code**.

Think:

```text
Lambda
   │
   └── Multiply
       ├── Parameter: x
       └── Parameter: x
```

That's the fundamental distinction.

---

# 2. `Func` vs `Expression<Func<...>>`

This is worth memorizing.

### Delegate

```csharp
Func<int, int> f = x => x * x;
```

Meaning:

> "Here is a function I can execute."

You can do:

```csharp
var result = f(5);
```

Result:

```text
25
```

---

### Expression

```csharp
Expression<Func<int, int>> expression =
    x => x * x;
```

Meaning:

> "Here is a representation of a function."

You can inspect:

```csharp
Console.WriteLine(expression);
```

Conceptually:

```text
x => (x * x)
```

The expression itself isn't simply being treated as executable machine code.

---

# 3. Why expression trees exist

Imagine a database query:

```csharp
db.Users
    .Where(u => u.Age > 18);
```

The database doesn't understand arbitrary C#.

It needs something like:

```sql
WHERE Age > 18
```

The provider needs to understand:

```text
User
 ↓
Age
 ↓
GreaterThan
 ↓
18
```

An expression tree gives the provider that structure.

So:

```text
C# lambda
     ↓
Expression Tree
     ↓
Query Provider
     ↓
SQL
```

That's one of their most important real-world uses.

---

# 4. The `System.Linq.Expressions` namespace

Expression trees live primarily in:

```csharp
using System.Linq.Expressions;
```

Example:

```csharp
Expression<Func<int, bool>> expression =
    x => x > 10;
```

Now:

```text
expression
```

contains a structured representation of:

```text
x > 10
```

---

# 5. The tree structure

Consider:

```csharp
Expression<Func<int, bool>> expression =
    x => x > 10;
```

Conceptually:

```text
Lambda
│
├── Parameter
│     └── x : int
│
└── GreaterThan
      ├── Parameter
      │     └── x
      │
      └── Constant
            └── 10
```

This is literally a tree-shaped representation.

The root is:

```text
Lambda
```

and underneath it are nodes describing the expression.

---

# 6. Expression trees are composed of nodes

Expression trees contain different node types.

Some important ones:

```text
ParameterExpression
ConstantExpression
BinaryExpression
UnaryExpression
MemberExpression
MethodCallExpression
LambdaExpression
ConditionalExpression
NewExpression
InvocationExpression
```

You don't need to memorize every node type yet.

The important idea is:

> **Different pieces of C# syntax become different expression-tree nodes.**

---

# 7. Parameter expressions

Consider:

```csharp
x => x + 10
```

The `x` is represented by a:

```csharp
ParameterExpression
```

Conceptually:

```text
Parameter
Name = x
Type = int
```

You can access it:

```csharp
var parameter =
    expression.Parameters[0];
```

For:

```csharp
Expression<Func<int, bool>> expression =
    x => x > 10;
```

the parameter is:

```text
Name → x
Type → int
```

---

# 8. Constant expressions

The:

```csharp
10
```

in:

```csharp
x => x > 10
```

is represented as a constant node.

Conceptually:

```text
Constant
Value = 10
Type = int
```

So the tree contains:

```text
x
+
10
```

as structured objects.

---

# 9. Binary expressions

An expression such as:

```csharp
x => x > 10
```

contains a binary operation:

```text
GreaterThan
```

Other examples:

```csharp
x => x + 10
```

→ `Add`

```csharp
x => x * 10
```

→ `Multiply`

```csharp
x => x == 10
```

→ `Equal`

```csharp
x => x && y
```

→ `AndAlso`

So:

```text
C# operator
    ↓
Expression node
```

---

# 10. Member access

Consider:

```csharp
Expression<Func<Employee, decimal>> expression =
    e => e.Salary;
```

The tree conceptually looks like:

```text
Lambda
│
├── Parameter
│     └── e : Employee
│
└── MemberAccess
      │
      └── Salary
```

The `Salary` property becomes a:

```csharp
MemberExpression
```

This is extremely important for query providers.

A provider can inspect:

```text
MemberAccess
    ↓
Employee.Salary
```

and potentially translate it to:

```sql
Salary
```

---

# 11. A complete example

Consider:

```csharp
Expression<Func<Employee, bool>> expression =
    e => e.Salary > 100000;
```

Conceptually:

```text
Lambda
│
├── Parameter
│     └── e
│
└── GreaterThan
      │
      ├── MemberAccess
      │     ├── Parameter
      │     │     └── e
      │     └── Salary
      │
      └── Constant
            └── 100000
```

Notice how much information is available.

A provider can see:

```text
parameter = e
property = Salary
operator = >
constant = 100000
```

That's why translation is possible.

---

# 12. Expression trees aren't strings

This is an important distinction.

An expression tree is **not**:

```text
"x => x.Salary > 100000"
```

stored as plain text.

It's structured objects.

Conceptually:

```text
LambdaExpression
    ↓
BinaryExpression
    ↓
MemberExpression
    ↓
ParameterExpression
```

This means software can programmatically inspect the structure.

---

# 13. Expression trees aren't IL either

An expression tree is also not simply:

```text
compiled machine code
```

or:

```text
IL instructions
```

It is a higher-level representation of an expression.

Think:

```text
Source-like structure
       ↓
Expression Tree
       ↓
can be interpreted/compiled/translated
```

That's why a query provider can inspect the tree before executing anything.

---

# 14. `Expression<TDelegate>`

The main generic type is:

```csharp
Expression<TDelegate>
```

For example:

```csharp
Expression<Func<int, bool>>
```

means:

```text
Expression
    of
Func<int, bool>
```

Another:

```csharp
Expression<Func<Employee, decimal>>
```

means:

```text
Employee → decimal
```

The expression tree describes a function with that signature.

---

# 15. `Expression<T>` can be compiled

An expression tree can be turned into executable code.

Example:

```csharp
Expression<Func<int, int>> expression =
    x => x * x;

Func<int, int> function =
    expression.Compile();

Console.WriteLine(function(5));
```

Output:

```text
25
```

So:

```text
Expression Tree
      ↓
Compile()
      ↓
Delegate
      ↓
Execute
```

This is useful because it demonstrates:

> An expression tree can be converted into executable code.

---

# 16. But compilation isn't translation

These are different operations.

### Compile

```csharp
expression.Compile()
```

means:

> Turn the expression into executable .NET code.

### Translate

A database provider may instead inspect:

```text
Expression Tree
```

and produce:

```sql
SQL
```

So:

```text
Expression Tree
    │
    ├── Compile → .NET delegate
    │
    └── Translate → SQL / other representation
```

This distinction is fundamental.

---

# 17. Why `IQueryable<T>` uses expression trees

Recall:

```csharp
IQueryable<T>
```

uses:

```csharp
Expression<Func<T, bool>>
```

for operations such as:

```csharp
Where()
```

Why?

Because the provider needs to inspect:

```csharp
x => x.Age > 18
```

rather than simply receive opaque executable code.

The provider can then interpret:

```text
Age
>
18
```

and translate it.

---

# 18. `IEnumerable<T>` cannot do this

With:

```csharp
IEnumerable<User>
```

the `Where()` predicate is:

```csharp
Func<User, bool>
```

Suppose:

```csharp
Func<User, bool> predicate =
    u => u.Age > 18;
```

The delegate is primarily executable behavior.

The LINQ-to-Objects implementation simply calls it:

```text
User
 ↓
predicate(user)
 ↓
true / false
```

There's no need to translate it.

---

# 19. `IQueryable<T>` can inspect it

With:

```csharp
IQueryable<User>
```

the provider receives something conceptually like:

```text
Expression<Func<User, bool>>
```

and can inspect:

```text
Parameter → u
Member → Age
Operator → GreaterThan
Constant → 18
```

Then potentially:

```sql
WHERE Age > 18
```

That's the architectural reason expression trees exist in LINQ.

---

# 20. Building an expression tree manually

You don't have to use lambda syntax.

You can construct the tree yourself.

Example:

```csharp
ParameterExpression x =
    Expression.Parameter(typeof(int), "x");

ConstantExpression ten =
    Expression.Constant(10);

BinaryExpression greaterThan =
    Expression.GreaterThan(x, ten);

Expression<Func<int, bool>> expression =
    Expression.Lambda<Func<int, bool>>(
        greaterThan,
        x);
```

This creates:

```text
x => x > 10
```

without writing that lambda directly.

---

# 21. Visualizing the manual construction

The code:

```csharp
Expression.Parameter(...)
```

creates:

```text
Parameter
   x
```

Then:

```csharp
Expression.Constant(10)
```

creates:

```text
Constant
   10
```

Then:

```csharp
Expression.GreaterThan(x, ten)
```

creates:

```text
GreaterThan
├── x
└── 10
```

Finally:

```csharp
Expression.Lambda(...)
```

wraps it:

```text
Lambda
└── GreaterThan
    ├── x
    └── 10
```

---

# 22. Why manually build trees?

Most application code should simply write:

```csharp
x => x > 10
```

Manual construction becomes useful when you need to **generate queries dynamically**.

For example:

```text
User chooses:
    Field = Salary
    Operator = >
    Value = 100000
```

You can construct:

```text
Salary > 100000
```

as an expression tree.

This enables dynamic filtering systems.

---

# 23. Dynamic filtering example

Suppose your UI lets users select:

```text
Field: Salary
Operator: Greater Than
Value: 100000
```

You might dynamically construct:

```csharp
Expression<Func<Employee, bool>>
```

representing:

```csharp
e => e.Salary > 100000
```

Then use it:

```csharp
query = query.Where(expression);
```

If `query` is `IQueryable<Employee>`, the provider can potentially translate the generated expression to SQL.

This is a common advanced application.

---

# 24. Expression tree visitor

Now we reach another important concept:

```csharp
ExpressionVisitor
```

It's designed to traverse expression trees.

Think:

```text
Expression Tree
      ↓
ExpressionVisitor
      ↓
visit every node
```

For example:

```text
Lambda
 ↓
GreaterThan
 ├── MemberAccess
 │     └── Salary
 └── Constant
       └── 100000
```

A visitor can inspect each node.

---

# 25. Why `ExpressionVisitor` matters

Suppose you want to:

- replace a property
    
- rewrite an expression
    
- inspect a query
    
- inject conditions
    
- transform constants
    
- build a custom provider
    
- implement dynamic filtering
    

You can traverse the expression tree using:

```csharp
ExpressionVisitor
```

This is one of the foundations of advanced LINQ infrastructure.

---

# 26. Example visitor

A simple visitor:

```csharp
class MyVisitor : ExpressionVisitor
{
    protected override Expression VisitMember(
        MemberExpression node)
    {
        Console.WriteLine(node.Member.Name);

        return base.VisitMember(node);
    }
}
```

Given:

```csharp
Expression<Func<Employee, bool>> expression =
    e => e.Salary > 100000;
```

you could visit it:

```csharp
var visitor = new MyVisitor();

visitor.Visit(expression);
```

It can encounter:

```text
Salary
```

because `Salary` is represented by a `MemberExpression`.

---

# 27. Expression rewriting

One of the powerful things about expression trees is that you can transform them.

Suppose:

```csharp
e => e.Salary > 100000
```

You might want to transform it into:

```csharp
e => e.Salary > 100000 && e.IsActive
```

Conceptually:

```text
Original expression
        ↓
Visitor / rewriter
        ↓
Modified expression
```

This is a foundation for sophisticated query composition.

---

# 28. Combining expressions

Suppose you have:

```csharp
Expression<Func<Employee, bool>> highSalary =
    e => e.Salary > 100000;
```

and:

```csharp
Expression<Func<Employee, bool>> active =
    e => e.IsActive;
```

You might conceptually want:

```csharp
e => e.Salary > 100000 && e.IsActive
```

But there's a catch.

You can't naïvely combine the bodies if they have separate parameter objects.

You need **parameter rebinding**.

This is one of the places where `ExpressionVisitor` becomes practically useful.

---

# 29. Why parameters are objects

Consider:

```csharp
e => e.Salary > 100000
```

and:

```csharp
e => e.IsActive
```

Even though both parameters are named:

```text
e
```

the parameter objects are distinct.

Think:

```text
Expression A
    Parameter A: e

Expression B
    Parameter B: e
```

They have the same name but aren't necessarily the same `ParameterExpression` instance.

So expression composition often requires replacing one parameter with another.

---

# 30. Parameter rebinding

Conceptually:

```text
Expression A:

e1 => e1.Salary > 100000

Expression B:

e2 => e2.IsActive
```

We want:

```text
e => e.Salary > 100000
     &&
     e.IsActive
```

So we replace:

```text
e1 → e
e2 → e
```

This is called **parameter rebinding**.

It's a common advanced expression-tree technique.

---

# 31. Expression trees have limitations

Not every C# construct can be represented in expression trees.

Expression trees historically support a restricted subset of C# language constructs.

For example, depending on the current .NET/compiler version, newer C# syntax may not be representable in traditional expression trees.

This matters because:

```text
C# language
      >
Expression Tree capabilities
```

The expression-tree representation has its own supported node model.

---

# 32. This explains some `IQueryable` surprises

You might write valid C#:

```csharp
query.Where(x => SomeNewCSharpFeature(x))
```

but encounter:

```text
expression tree cannot contain ...
```

or a provider translation failure.

There are actually **two separate constraints**:

```text
C# expression
    ↓
Can compiler represent it as an expression tree?
    ↓
Can provider translate that expression tree?
```

So:

```text
valid C#
≠
valid expression tree
≠
provider-translatable expression
```

This three-level distinction is extremely important.

---

# 33. Three levels of compatibility

When using `IQueryable<T>`, think:

```text
LEVEL 1
Is this valid C#?
        ↓
LEVEL 2
Can it be represented in an expression tree?
        ↓
LEVEL 3
Can the provider translate it?
```

For example:

```csharp
db.Users.Where(u => u.Age > 18)
```

typically passes all three.

But some advanced C# construct might fail at level 2.

Another expression might pass level 2 but fail at level 3 because EF Core or another provider doesn't know how to translate it.

---

# 34. Expression tree vs execution

A common misconception is:

> "Expression trees execute code."

Not inherently.

They **represent** code.

You can execute one by compiling it:

```csharp
var function = expression.Compile();
```

But until then, the tree itself is data.

Think:

```text
Delegate
→ executable behavior

Expression Tree
→ structured description
```

---

# 35. Expression trees are metadata-like structures

Not metadata in the reflection sense, but they provide **structural information about an expression**.

For:

```csharp
e => e.Salary > 100000
```

you can determine:

```text
Parameter:
    e

Property:
    Salary

Operator:
    >

Constant:
    100000
```

That's exactly the information needed to construct another representation.

For example:

```text
Expression
    ↓
SQL
```

or:

```text
Expression
    ↓
Search API filter
```

or:

```text
Expression
    ↓
custom query language
```

---

# 36. Expression trees aren't only for databases

Databases are the most famous use case, but they're useful for:

### ORMs

```text
LINQ → SQL
```

### Dynamic query systems

```text
user filters → expression tree
```

### Validation

```text
property expression → validation metadata
```

### Mapping systems

```text
source expression → target expression
```

### Authorization

```text
policy expression → executable/queryable rule
```

### Custom providers

```text
expression → external API query
```

The broader idea is:

> **Represent behavior as data.**

---

# 37. Expression trees and reflection

They are related conceptually but different.

Reflection asks:

> "What members does this type have?"

Expression trees ask:

> "What operation does this piece of code represent?"

For example:

```csharp
e => e.Salary
```

Reflection can tell you:

```text
Employee has property Salary
```

An expression tree tells you:

```text
This particular expression accesses Employee.Salary
```

Expression trees preserve the structure of the operation itself.

---

# 38. `Expression.Property`

When constructing trees manually, you can create member access:

```csharp
var parameter =
    Expression.Parameter(typeof(Employee), "e");

var property =
    Expression.Property(parameter, "Salary");
```

Conceptually:

```text
e.Salary
```

Then:

```csharp
var constant =
    Expression.Constant(100000m);

var comparison =
    Expression.GreaterThan(
        property,
        constant);
```

Now you have:

```text
e.Salary > 100000
```

---

# 39. Building a complete predicate manually

Conceptually:

```csharp
var parameter =
    Expression.Parameter(
        typeof(Employee),
        "e");

var salary =
    Expression.Property(
        parameter,
        nameof(Employee.Salary));

var value =
    Expression.Constant(100000m);

var body =
    Expression.GreaterThan(
        salary,
        value);

var predicate =
    Expression.Lambda<Func<Employee, bool>>(
        body,
        parameter);
```

Now:

```csharp
predicate
```

represents:

```csharp
e => e.Salary > 100000m
```

You could potentially use it with:

```csharp
query.Where(predicate);
```

where `query` is `IQueryable<Employee>`.

---

# 40. Why `nameof()` is preferable

Notice:

```csharp
Expression.Property(
    parameter,
    nameof(Employee.Salary));
```

rather than:

```csharp
Expression.Property(
    parameter,
    "Salary");
```

`nameof()` is compile-time checked and refactor-friendly.

If the property is renamed, tooling can update the reference.

This is a small but important C# engineering practice.

---

# 41. Expression tree anatomy

For:

```csharp
Expression<Func<Employee, bool>> predicate =
    e => e.Salary > 100000;
```

think:

```text
Expression<TDelegate>
│
└── LambdaExpression
     │
     ├── Parameters
     │      └── ParameterExpression
     │             └── e
     │
     └── Body
          │
          └── BinaryExpression
               └── GreaterThan
                    │
                    ├── MemberExpression
                    │     └── Salary
                    │
                    └── ConstantExpression
                          └── 100000
```

This diagram is worth understanding.

---

# 42. Expression trees and `GroupBy()`

Now connect this to the LINQ phases you've already studied.

Suppose:

```csharp
db.Orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        Total = g.Sum(o => o.Total)
    });
```

The query provider receives expression representations describing:

```text
GroupBy
   ↓
CustomerId
   ↓
Select
   ↓
Sum(Total)
```

Conceptually:

```text
LINQ
 ↓
Expression Trees
 ↓
Provider
 ↓
SQL GROUP BY
```

So expression trees are the machinery underneath the `IQueryable` phases you've already studied.

---

# 43. Expression trees and `Join()`

Same idea.

```csharp
db.Customers.Join(
    db.Orders,
    c => c.Id,
    o => o.CustomerId,
    (c, o) => new
    {
        c.Name,
        o.Total
    });
```

The provider can inspect:

```text
Join
 ├── Customer.Id
 ├── Order.CustomerId
 └── projection
```

and potentially produce:

```sql
INNER JOIN Orders
    ON Customers.Id = Orders.CustomerId
```

Again:

```text
LINQ syntax
    ↓
Expression tree
    ↓
provider translation
```

---

# 44. Expression trees and deferred execution

Expression trees also help explain why:

```csharp
var query = db.Users
    .Where(u => u.Age > 18);
```

doesn't necessarily execute immediately.

You're effectively building a query representation:

```text
Query
 ↓
Where
 ↓
Age > 18
```

Then:

```csharp
query.ToList();
```

causes the provider to execute the resulting query.

So:

```text
Build expression
    ≠
Execute expression
```

---

# 45. Expression trees are immutable

Expression trees are generally treated as immutable structures.

You don't normally mutate:

```text
node.Operator = ...
```

Instead, transformations create/reuse nodes to produce a new expression tree.

This is why `ExpressionVisitor` is designed around methods that return `Expression`.

Conceptually:

```text
Old Tree
   ↓
Visitor
   ↓
New Tree
```

---

# 46. Why immutability is useful

An immutable expression representation is useful because:

- trees can be safely reused
    
- transformations are easier to reason about
    
- query structures aren't accidentally modified
    
- providers can inspect stable representations
    

This fits naturally with functional-style query construction.

---

# 47. Expression tree compilation vs provider execution

Consider:

```csharp
Expression<Func<Employee, bool>> predicate =
    e => e.Salary > 100000;
```

There are two very different paths:

### Path A — Compile

```text
Expression
   ↓
Compile()
   ↓
Func<Employee,bool>
   ↓
.NET execution
```

### Path B — Query provider

```text
Expression
   ↓
Provider
   ↓
SQL
   ↓
Database
```

Same expression tree.

Completely different execution paths.

---

# 48. Why you shouldn't call `Compile()` on an EF query predicate

Suppose:

```csharp
Expression<Func<Employee, bool>> predicate =
    e => e.Salary > 100000;
```

If you do:

```csharp
var compiled = predicate.Compile();
```

you now have:

```csharp
Func<Employee, bool>
```

You've converted the expression into executable C#.

A database provider can't inspect that delegate in the same way.

So:

```text
Expression
   ↓
Compile()
   ↓
delegate
```

can destroy the provider's ability to translate the predicate.

This is a crucial practical distinction.

---

# 49. The complete LINQ architecture

You can now see the deeper architecture:

```text
                   C# Lambda
                      │
             ┌────────┴────────┐
             │                 │
          IEnumerable       IQueryable
             │                 │
             ▼                 ▼
          Func<T,R>     Expression<Func<T,R>>
             │                 │
             ▼                 ▼
        Execute C#        Expression Tree
                               │
                               ▼
                         Query Provider
                               │
                    ┌──────────┴──────────┐
                    │                     │
                   SQL             Other provider
                    │
                    ▼
                 Database
```

This is the architectural foundation of provider-backed LINQ.

---

# 50. Phase 22 — The mental model

Don't think:

> "Expression trees are some complicated LINQ syntax."

Think:

> **Expression trees turn code structure into data.**

For example:

```csharp
e => e.Salary > 100000
```

becomes conceptually:

```text
          Lambda
             │
        GreaterThan
          /      \
    Salary      100000
```

That tree can then be:

```text
                    Expression Tree
                          │
          ┌───────────────┼───────────────┐
          │               │               │
       Compile          Inspect         Translate
          │               │               │
          ▼               ▼               ▼
      .NET code       Visitor/etc.       SQL
```

---

# Phase 22 checkpoint

Given:

```csharp
Expression<Func<Employee, bool>> predicate =
    e => e.Salary > 100000 && e.IsActive;
```

you should be able to mentally see:

```text
Lambda
│
├── Parameter
│     └── e
│
└── AndAlso
      │
      ├── GreaterThan
      │     ├── e.Salary
      │     └── 100000
      │
      └── MemberAccess
            └── e.IsActive
```

And the crucial chain is:

```text
C# Lambda
    ↓
Expression<Func<Employee, bool>>
    ↓
Expression Tree
    ↓
Query Provider
    ↓
SQL
```

while for `IEnumerable<T>`:

```text
C# Lambda
    ↓
Func<Employee, bool>
    ↓
Execute directly in .NET
```

### The three rules to lock in

1. **`Func<T, TResult>` is executable behavior.**
    
2. **`Expression<Func<T, TResult>>` is a structured representation of behavior.**
    
3. **`IQueryable<T>` uses expression trees so a provider can inspect and potentially translate LINQ into another query language such as SQL.**
    

Once this is solid, the next deeper layer is **Expression Tree Construction & Visitors**—how to dynamically build predicates such as `e => e.Name == "Alice" && e.Age > 18`, how parameter rebinding works, and how to write reusable expression combinators.