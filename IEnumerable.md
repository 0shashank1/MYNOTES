# Phase 2 — `IEnumerable<T>`

The key idea:

> **`IEnumerable<T>` represents something you can enumerate — one element at a time — as a sequence of `T`.**

---

## 1. Start with `IEnumerable<T>`

Suppose:

```csharp
List<int> numbers = new()
{
    10, 20, 30, 40
};
```

A `List<int>` is a collection of integers.

But it also implements:

```csharp
IEnumerable<int>
```

So you can write:

```csharp
IEnumerable<int> numbers = new List<int>
{
    10, 20, 30, 40
};
```

The important distinction is:

```text
List<int>
    ↓
A concrete collection

IEnumerable<int>
    ↓
A sequence that can be enumerated
```

`IEnumerable<T>` doesn't care **how the data is stored**.

It only promises:

> "You can ask me for an enumerator and walk through my elements."

---

# 2. What does "enumerate" mean?

**Enumerate = visit each element in a sequence.**

For example:

```csharp
IEnumerable<int> numbers = new[] { 10, 20, 30 };

foreach (int number in numbers)
{
    Console.WriteLine(number);
}
```

The `foreach` loop effectively walks through:

```text
10 → 20 → 30
```

That's enumeration.

And this is exactly where LINQ starts becoming interesting.

---

# 3. `foreach` and `IEnumerable<T>`

When you write:

```csharp
foreach (int number in numbers)
{
    Console.WriteLine(number);
}
```

C# needs a mechanism to move through the sequence.

That mechanism is:

```csharp
IEnumerator<T>
```

So the relationship is:

```text
IEnumerable<T>
       │
       │ GetEnumerator()
       ▼
IEnumerator<T>
       │
       │ MoveNext()
       ▼
   Current
       │
       ▼
   Next element
```

This relationship is fundamental.

---

# 4. `IEnumerable<T>` has `GetEnumerator()`

Conceptually, its important contract is:

```csharp
public interface IEnumerable<T>
{
    IEnumerator<T> GetEnumerator();
}
```

There's also a non-generic:

```csharp
IEnumerable
```

But for modern C#, focus primarily on:

```csharp
IEnumerable<T>
```

because generic LINQ operations work heavily with it.

---

# 5. `IEnumerator<T>`

An enumerator represents the **current position while walking through a sequence**.

Its important members are:

```csharp
MoveNext()
Current
```

For example:

```csharp
int[] numbers = { 10, 20, 30 };

IEnumerator<int> enumerator = numbers.GetEnumerator();
```

Initially, you're effectively **before the first element**.

Then:

```csharp
enumerator.MoveNext();
```

moves to the first element.

Now:

```csharp
enumerator.Current
```

is:

```text
10
```

Another:

```csharp
enumerator.MoveNext();
```

moves to:

```text
20
```

Then:

```csharp
enumerator.Current
```

returns:

```text
20
```

---

# 6. Manually doing what `foreach` does

You can manually enumerate:

```csharp
int[] numbers = { 10, 20, 30 };

IEnumerator<int> enumerator = numbers.GetEnumerator();

while (enumerator.MoveNext())
{
    int number = enumerator.Current;

    Console.WriteLine(number);
}
```

Output:

```text
10
20
30
```

This is extremely useful for understanding LINQ.

Conceptually:

```csharp
foreach (int number in numbers)
{
    // ...
}
```

is based on this kind of enumeration process.

You normally **shouldn't write it manually**. The point is to understand what's underneath.

---

# 7. Why does LINQ care about `IEnumerable<T>`?

Now return to Phase 1:

```csharp
var result = numbers.Where(n => n > 20);
```

`Where()` is an extension method that operates on an `IEnumerable<T>`.

Conceptually:

```text
IEnumerable<T>
      ↓
    Where
      ↓
IEnumerable<T>
```

This is important:

### `Where()` doesn't fundamentally require a `List<T>`

It can work with any suitable `IEnumerable<T>`.

For example:

```csharp
int[] numbers = { 10, 20, 30, 40 };

var result = numbers.Where(n => n > 20);
```

An array can be enumerated.

A list can be enumerated.

Many other sequence types can be enumerated.

Therefore LINQ can operate on them.

---

# 8. `IEnumerable<T>` is about capability, not storage

Consider:

```csharp
List<int> list = new() { 1, 2, 3 };
```

The list has many capabilities:

```text
Add
Remove
Indexing
Count
Sorting
...
```

But if you expose it as:

```csharp
IEnumerable<int> sequence = list;
```

you're saying:

> "For this part of the program, I only care that this thing provides a sequence of integers."

You can enumerate:

```csharp
foreach (int n in sequence)
{
    Console.WriteLine(n);
}
```

But you shouldn't expect `IEnumerable<T>` itself to provide:

```csharp
sequence.Add(...)
```

because that's not part of its contract.

---

# 9. Interface vs implementation

This distinction is important in C#.

Suppose:

```csharp
List<int> list = new();
```

`List<int>` is a **concrete type**.

```csharp
IEnumerable<int> sequence = list;
```

`IEnumerable<int>` is an **interface**.

Think:

```text
List<int>
├── Stores elements
├── Add()
├── Remove()
├── Indexing
├── Count
└── Implements IEnumerable<int>
```

The interface describes the capability:

```text
IEnumerable<int>
└── "I can provide an enumeration of int values."
```

This is classic interface-based programming.

---

# 10. The really important part: LINQ returns `IEnumerable<T>`

Consider:

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

IEnumerable<int> result =
    numbers.Where(n => n % 2 == 0);
```

Notice something interesting:

```text
numbers
   ↓
Where()
   ↓
IEnumerable<int>
```

`Where()` doesn't normally return a `List<int>`.

It returns a sequence.

That means you can continue composing operations:

```csharp
var result = numbers
    .Where(n => n % 2 == 0)
    .Select(n => n * 10)
    .OrderBy(n => n);
```

Conceptually:

```text
IEnumerable<int>
       ↓
     Where
       ↓
IEnumerable<int>
       ↓
     Select
       ↓
IEnumerable<int>
       ↓
   OrderBy
       ↓
IEnumerable<int>
```

This is why LINQ feels like a pipeline.

---

# 11. `IEnumerable<T>` enables composition

This is one of the biggest ideas in LINQ.

Suppose:

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6 };
```

You can build:

```csharp
var result = numbers
    .Where(n => n > 2)
    .Where(n => n % 2 == 0)
    .Select(n => n * 100);
```

Think:

```text
Original
1 2 3 4 5 6
      ↓
n > 2
      ↓
3 4 5 6
      ↓
even
      ↓
4 6
      ↓
×100
      ↓
400 600
```

Each operation produces another sequence.

---

# 12. A subtle but critical concept: `IEnumerable<T>` does NOT necessarily mean "all data is already in memory"

This is where people often develop an incorrect mental model.

You might think:

```text
IEnumerable<T>
=
List<T>
```

No.

`IEnumerable<T>` only represents something that can be enumerated.

The underlying source could be:

```text
Array
List
Generated sequence
File-related sequence
Custom iterator
LINQ query
etc.
```

For example:

```csharp
IEnumerable<int> numbers = GetNumbers();
```

`GetNumbers()` could generate values as they're requested.

---

# 13. `yield return`

This is one of the best ways to understand `IEnumerable<T>`.

Consider:

```csharp
IEnumerable<int> GetNumbers()
{
    yield return 10;
    yield return 20;
    yield return 30;
}
```

Now:

```csharp
var numbers = GetNumbers();
```

You have an `IEnumerable<int>`.

You can enumerate it:

```csharp
foreach (int number in numbers)
{
    Console.WriteLine(number);
}
```

Output:

```text
10
20
30
```

The important thing is that the method doesn't need to construct a `List<int>` first.

It can **produce values during enumeration**.

---

# 14. `yield return` and deferred execution

Consider:

```csharp
IEnumerable<int> GetNumbers()
{
    Console.WriteLine("Generating 10");
    yield return 10;

    Console.WriteLine("Generating 20");
    yield return 20;

    Console.WriteLine("Generating 30");
    yield return 30;
}
```

Now:

```csharp
var numbers = GetNumbers();

Console.WriteLine("Before foreach");

foreach (var number in numbers)
{
    Console.WriteLine(number);
}
```

The important concept is that the sequence is **produced as it is enumerated**.

This gives you the connection:

```text
IEnumerable<T>
      ↓
Enumeration
      ↓
yield return
      ↓
Deferred/lazy production
```

This is closely related to what you learned in Phase 1 about **deferred execution**.

---

# 15. `IEnumerable<T>` + LINQ

Now everything starts connecting.

Suppose:

```csharp
IEnumerable<int> numbers = GetNumbers();
```

Then:

```csharp
var result = numbers
    .Where(n => n > 10)
    .Select(n => n * 2);
```

At this point, you have essentially built a **query pipeline**.

```text
GetNumbers()
     ↓
IEnumerable<int>
     ↓
Where
     ↓
Select
     ↓
IEnumerable<int>
```

When you enumerate:

```csharp
foreach (var number in result)
{
    Console.WriteLine(number);
}
```

the pipeline gets executed.

---

# 16. Why `ToList()` changes things

Compare:

```csharp
var query = numbers
    .Where(n => n > 10);
```

with:

```csharp
var list = numbers
    .Where(n => n > 10)
    .ToList();
```

The first is a sequence/query.

The second creates an actual `List<int>`.

Conceptually:

```text
Where()
   ↓
IEnumerable<int>
   ↓
ToList()
   ↓
List<int>
```

`ToList()` is therefore called a **materialization** operation.

Other common materialization operations include:

```csharp
ToArray()
ToList()
ToDictionary()
ToHashSet()
```

---

# 17. The most important distinction so far

You should now distinguish these three things:

### `IEnumerable<T>`

> A sequence that can be enumerated.

### `IEnumerator<T>`

> The mechanism/state used to walk through that sequence.

### `List<T>`

> A concrete collection that stores elements and also supports enumeration.

Relationship:

```text
             List<T>
                │
                │ implements
                ▼
         IEnumerable<T>
                │
                │ GetEnumerator()
                ▼
         IEnumerator<T>
                │
          ┌─────┴─────┐
          ▼           ▼
     MoveNext()     Current
```

---

# 18. How this connects directly to `foreach`

You can mentally translate:

```csharp
foreach (var item in collection)
{
    Console.WriteLine(item);
}
```

into:

```text
Get an enumerator
       ↓
Move to next element
       ↓
Read Current
       ↓
Execute loop body
       ↓
Move to next element
       ↓
Repeat until MoveNext() == false
```

So:

```csharp
foreach
```

and:

```csharp
IEnumerable<T>
```

are deeply connected.

---

# 19. And now LINQ makes more sense

When you write:

```csharp
var adults = people
    .Where(p => p.Age >= 18);
```

you should no longer think:

> "C# somehow filters my list."

Think:

```text
people
  │
  │ IEnumerable<Person>
  ▼
Where(...)
  │
  │ creates another enumerable sequence
  ▼
adults
```

When `adults` is enumerated:

```text
GetEnumerator()
       ↓
MoveNext()
       ↓
Get Current Person
       ↓
Run predicate: Age >= 18
       ↓
If true → yield person
       ↓
Continue
```

That's the conceptual machinery behind LINQ-to-Objects.

---

# 20. One more important distinction: `IEnumerable<T>` vs `IQueryable<T>`

You don't need to master this yet, but you should know why it matters.

### `IEnumerable<T>`

Generally associated with **in-memory enumeration**.

```text
Objects in memory
       ↓
LINQ
       ↓
Enumeration
```

### `IQueryable<T>`

Designed for query providers that can inspect and translate the query.

For example, Entity Framework Core:

```text
C# LINQ
   ↓
IQueryable<T>
   ↓
Expression tree
   ↓
SQL
   ↓
Database
```

We'll study this later.

For now:

> **`IEnumerable<T>` → think "sequence I can enumerate."**

> **`IQueryable<T>` → think "query that a provider can translate."**

---

# Phase 2 mental model

You should now have this picture:

```text
                 DATA SOURCE
                     │
                     ▼
               IEnumerable<T>
                     │
             GetEnumerator()
                     │
                     ▼
               IEnumerator<T>
                     │
              ┌──────┴──────┐
              │             │
          MoveNext()      Current
              │             │
              └──────┬──────┘
                     ▼
                Each element
                     │
                     ▼
                  LINQ
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Where      Select     OrderBy
          │          │          │
          └──────────┼──────────┘
                     ▼
               IEnumerable<T>
                     │
                     ▼
             ToList / ToArray
                     │
                     ▼
             Materialized data
```

## The one sentence to remember

> **`IEnumerable<T>` is the abstraction that says: "This represents a sequence of `T` values that I can enumerate one at a time."**

And that is the foundation upon which **LINQ-to-Objects** is built.

### Next: Phase 3 — Lambda Expressions + Delegates

This is the other half of the puzzle. Once you understand why this works:

```csharp
numbers.Where(n => n > 10)
```

you'll understand how **`n => n > 10` becomes a function that LINQ can execute**.