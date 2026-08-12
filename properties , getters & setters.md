
## Properties, Getters, and Setters in C\#

In C#, a **property** is a controlled way to expose data from a class. It commonly uses a **getter** to read the value and a **setter** to change it.

### 1. Basic example

```c#
class Student
{
    private string name;

    public string Name
    {
        get
        {
            return name;
        }

        set
        {
            name = value;
        }
    }
}
```

Usage:

```c#
Student s = new Student();

s.Name = "Shashank";       // setter runs
Console.WriteLine(s.Name); // getter runs
```

When you write:

```c
s.Name = "Shashank";
```

C# effectively calls:

```c#
set
{
    name = "Shashank";
}
```

And:

```c#
Console.WriteLine(s.Name);
```

calls:

```c#
get
{
    return name;
}
```

---

## 2. What is `value`?

Inside a setter, `value` is a **special C# keyword** representing the value being assigned.

```c#
public string Name
{
    get
    {
        return name;
    }

    set
    {
        name = value;
    }
}
```

So:

```c#
s.Name = "John";
```

means that inside the setter:

```c
value == "John"
```

---

## 3. Why not just use a public field?

You could do:

```c#
class Student
{
    public string Name;
}
```

Then:

```c
s.Name = "John";
```

works.

But properties allow you to **control access and validation**.

For example:

```C#
class Student
{
    private int age;

    public int Age
    {
        get
        {
            return age;
        }

        set
        {
            if (value >= 0 && value <= 100)
            {
                age = value;
            }
        }
    }
}
```

Now:

```c
s.Age = 20;   // ✅
s.Age = -10;  // ❌ won't update
```

This is one of the major reasons properties are useful.

---

# 4. Auto-implemented properties

Most of the time, you don't need to manually create the backing field.

Instead:

```c#
class Student
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

C# automatically creates the backing storage for you.

Usage:

```c#
Student s = new Student();

s.Name = "Shashank";
s.Age = 20;

Console.WriteLine(s.Name);
Console.WriteLine(s.Age);
```

This is extremely common in modern C#.

---

# 5. Getter-only property

You can make a property **read-only**:

```c#
class Student
{
    public string Name { get; }

    public Student(string name)
    {
        Name = name;
    }
}
```

You can read it:

```c#
Console.WriteLine(s.Name);
```

But you can't do:

```c
s.Name = "John"; // ❌
```

This is useful when a value shouldn't be changed after initialization.

---

# 6. Private setter

Sometimes you want other classes to **read** a property but only the class itself should be able to modify it.

```c#
class BankAccount
{
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount)
    {
        Balance += amount;
    }
}
```

Outside:

```c#
BankAccount account = new BankAccount();

Console.WriteLine(account.Balance); // ✅

account.Balance = 5000; // ❌
```

But inside `BankAccount`:

```c#
Balance += amount; // ✅
```

This is very useful for **encapsulation**.

---

# 7. Different access modifiers

You can independently control the getter and setter:

```c#
public int Age
{
    get;
    private set;
}
```

Means:

```c#
Outside class     → can GET
Outside class     → cannot SET

Inside class      → can GET
Inside class      → can SET
```

---

# 8. Properties can contain logic

A property doesn't necessarily just store a value.

For example:

```c#
class Rectangle
{
    public double Width { get; set; }
    public double Height { get; set; }

    public double Area
    {
        get
        {
            return Width * Height;
        }
    }
}
```

Usage:

```c#
Rectangle r = new Rectangle();

r.Width = 10;
r.Height = 5;

Console.WriteLine(r.Area); // 50
```

`Area` doesn't need its own stored variable. It is **calculated whenever you access it**.

You can shorten it using an expression-bodied property:

```c#
public double Area => Width * Height;
```

---

## 9. Property vs field vs method

Think of them like this:

```c#
class Student
{
    // Field
    private string name;

    // Property
    public string Name
    {
        get { return name; }
        set { name = value; }
    }

    // Method
    public void Study()
    {
        Console.WriteLine("Studying...");
    }
}
```

|Concept|Purpose|Example|
|---|---|---|
|**Field**|Stores data internally|`private string name;`|
|**Property**|Controls access to data|`public string Name { get; set; }`|
|**Getter**|Reads a property|`get`|
|**Setter**|Changes a property|`set`|
|**Method**|Performs an action|`Study()`|

### A good mental model

```
              Student
                 │
        ┌────────┴────────┐
        │                 │
     Property           Method
      Name              Study()
        │
   ┌────┴────┐
   │         │
 getter    setter
   │         │
 read      write
```

The important distinction is:

> **A property is the public interface. The getter and setter define what happens when that property is read or written.**

And in modern C#, you'll very frequently see:

```c#
public string Name { get; set; }
```

rather than manually writing the getter/setter and backing field.