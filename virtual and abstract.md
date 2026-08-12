

## 1. Virtual Method in C\#

A `virtual` method has a **default implementation**, but a derived class can override it using `override`.

```c#
class Animal
{
    public virtual void Sound()
    {
        Console.WriteLine("Animal makes a sound");
    }
}

class Dog : Animal
{
    public override void Sound()
    {
        Console.WriteLine("Dog barks");
    }
}
```

Usage:

```c#
Animal animal = new Dog();
animal.Sound();
```

Output:

```
Dog barks
```

### Why?

Although the variable is:

```
Animal animal
```

the actual object is:

```c#
new Dog()
```

Therefore, C# calls `Dog.Sound()`.

This is **runtime polymorphism**.

---

# 2. Abstract Class in C#

An `abstract` class **cannot be instantiated**.

```c#
abstract class Animal
{
    public abstract void Sound();
}
```

Here, `Sound()` is an **abstract method**. It has **no implementation**.

A derived class **must** implement it:

```c#
class Dog : Animal
{
    public override void Sound()
    {
        Console.WriteLine("Dog barks");
    }
}
```

You cannot do:

```
Animal a = new Animal();  // ❌ Error
```

But this is valid:

```
Animal a = new Dog();     // ✅
a.Sound();
```

---

## 3. `virtual` vs `abstract`

|Feature|`virtual`|`abstract`|
|---|---|---|
|Has implementation?|✅ Yes|❌ No|
|Must derived class override?|❌ No|✅ Yes|
|Can be in normal class?|✅ Yes|Abstract method requires abstract class|
|Can class be instantiated?|✅ Yes, if class isn't abstract|❌ Abstract class cannot|
|Uses `override` in child?|Optional|Required|
|Main purpose|Provide customizable behavior|Define required behavior|

### Example

**Virtual:**

```c#
class Animal
{
    public virtual void Eat()
    {
        Console.WriteLine("Animal eats");
    }
}

class Dog : Animal
{
    // Optional
    public override void Eat()
    {
        Console.WriteLine("Dog eats");
    }
}
```

The child **may** override `Eat()`.

---

**Abstract:**

```c#
abstract class Animal
{
    public abstract void Eat();
}

class Dog : Animal
{
    // Required
    public override void Eat()
    {
        Console.WriteLine("Dog eats");
    }
}
```

The child **must** override `Eat()`.

---

## 4. Abstract Class Can Have Both

This is important in C#.

An abstract class can contain:

- abstract methods
- virtual methods
- normal methods
- fields
- properties
- constructors

Example:

```c#
abstract class Animal
{
    // Must be implemented
    public abstract void Sound();

    // Can be overridden
    public virtual void Eat()
    {
        Console.WriteLine("Animal eats");
    }

    // Normal method
    public void Sleep()
    {
        Console.WriteLine("Animal sleeps");
    }
}
```

Then:

```c#
class Dog : Animal
{
    public override void Sound()
    {
        Console.WriteLine("Dog barks");
    }

    public override void Eat()
    {
        Console.WriteLine("Dog eats");
    }
}
```

### Simple exam definition

> **Virtual method:** A method with a default implementation that can be overridden by a derived class.

> **Abstract class:** A class that cannot be instantiated and is used as a base class; it can contain abstract members that derived classes must implement.

**Shortcut:**  
`virtual` → **optional override**  
`abstract` → **mandatory override**