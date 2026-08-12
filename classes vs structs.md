

In C#, both `struct` and `class` are types, but the **main difference is value type vs reference type**.

|Feature|`struct`|`class`|
|---|---|---|
|Type|**Value type**|**Reference type**|
|Stored generally|Stack/inline as part of containing object|Object data generally on managed heap|
|Assignment|Copies the **value**|Copies the **reference**|
|`null`|❌ Not normally allowed*|✅ Allowed|
|Inheritance|Cannot inherit from another struct/class|Supports class inheritance|
|Can implement interfaces|✅ Yes|✅ Yes|
|Default constructor|Always has a default value|Can define constructors|
|Best for|Small, immutable data|Complex/stateful objects|

### 1. Assignment behaves differently

**Struct:**

```c#
struct Point
{
    public int X;
}

Point p1 = new Point { X = 10 };
Point p2 = p1;

p2.X = 20;

Console.WriteLine(p1.X); // 10
Console.WriteLine(p2.X); // 20
```

`p2 = p1` **copies the entire value**.

**Class:**

```c#
class Point
{
    public int X;
}

Point p1 = new Point { X = 10 };
Point p2 = p1;

p2.X = 20;

Console.WriteLine(p1.X); // 20
Console.WriteLine(p2.X); // 20
```

Here, `p1` and `p2` refer to the **same object**.

---

### 2. `struct` is a value type

```c#
int x = 10;
int y = x;

y = 20;

Console.WriteLine(x); // 10
```

`int` itself is a struct (`System.Int32`), so this follows value semantics.

A custom struct works similarly:

```c#
struct EmployeeId
{
    public int Id;
}
```

---

### 3. `class` is a reference type

```c#
class Employee
{
    public string Name;
}

Employee e1 = new Employee();
e1.Name = "John";

Employee e2 = e1;
e2.Name = "Bob";

Console.WriteLine(e1.Name); // Bob
```

Both variables point to the same object.

---

### 4. Inheritance

Classes support inheritance:

```c#
class Animal
{
    public void Eat() { }
}

class Dog : Animal
{
    public void Bark() { }
}
```

But you can't do:

```
struct Dog : Animal // ❌
```

A struct can, however, implement an interface:

```c#
interface IMovable
{
    void Move();
}

struct Car : IMovable
{
    public void Move()
    {
        Console.WriteLine("Moving");
    }
}
```

---

### 5. When should you use each?

Use a **struct** when the type represents a small value:

```c#
struct Point
{
    public int X;
    public int Y;
}
```

Examples from .NET include:

```
int
double
bool
DateTime
Guid
```

Use a **class** for entities/objects with more complex behavior or mutable state:

```c#
class Student
{
    public string Name;
    public int Age;

    public void Study()
    {
        // ...
    }
}
```

### The easiest way to remember

> **Struct = "this value itself"**  
> **Class = "a reference to an object"**

So:

```
struct
p1 ──[ X=10 ]
p2 ──[ X=10 ]   ← separate copy

class
p1 ──┐
     ├──> [ X=10 ]  ← same object
p2 ──┘
```

* A struct can be nullable using `Point?`, which is shorthand for `Nullable<Point>`.