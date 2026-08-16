

# Interfaces in C\##

An **interface** in C# is a contract that defines **what a class or struct must provide**, without specifying the complete implementation of that behavior.

Interfaces are fundamental to **abstraction, polymorphism, loose coupling, dependency injection, and testable software design**.
## 1. Basic Syntax

```c#
interface IAnimal
{
    void MakeSound();
}
```

A class implements an interface using `:`:

```c#
class Dog : IAnimal
{
    public void MakeSound()
    {
        Console.WriteLine("Bark");
    }
}
```

Usage:

```c#
Dog dog = new Dog();
dog.MakeSound();
```

Output:

```
Bark
```

### Important point

The interface says:

> "Any class implementing `IAnimal` must provide `MakeSound()`."

It doesn't necessarily say **how** the method should work.


# 2. Why Use Interfaces?

Suppose we have:

```c#
class Dog
{
    public void MakeSound()
    {
        Console.WriteLine("Bark");
    }
}

class Cat
{
    public void MakeSound()
    {
        Console.WriteLine("Meow");
    }
}
```

Both classes have `MakeSound()`, but there is no common contract.

An interface gives us that common abstraction:

```c#
interface IAnimal
{
    void MakeSound();
}
```

Then:

```c#
class Dog : IAnimal
{
    public void MakeSound()
    {
        Console.WriteLine("Bark");
    }
}

class Cat : IAnimal
{
    public void MakeSound()
    {
        Console.WriteLine("Meow");
    }
}
```

Now we can write:

```c#
IAnimal animal = new Dog();
animal.MakeSound();

animal = new Cat();
animal.MakeSound();
```

Output:

```
Bark
Meow
```

This is **polymorphism through an interface**.


# 3. Interface Members

Modern C# interfaces can contain more than just abstract methods.

Depending on the C# version and design, an interface can contain:

- Methods
- Properties
- Events
- Indexers
- Static members
- Default interface implementations
- Constants
- Nested types

For example:

```c#
interface IEmployee
{
    string Name { get; set; }

    decimal CalculateSalary();

    event EventHandler EmployeeUpdated;
}
```

A class implementing it must provide the required instance members:

```c#
class Employee : IEmployee
{
    public string Name { get; set; }

    public decimal CalculateSalary()
    {
        return 50000;
    }

    public event EventHandler? EmployeeUpdated;
}
```

---

# 4. Interface Methods

```c#
interface ILogger
{
    void Log(string message);
}
```

Implementation:

```c#
class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

Another implementation:

```c#
class FileLogger : ILogger
{
    public void Log(string message)
    {
        File.AppendAllText("log.txt", message + Environment.NewLine);
    }
}
```

Now code can depend on `ILogger` rather than a particular logger:

```c#
ILogger logger = new ConsoleLogger();

logger.Log("Application started");
```

This is an important software-design principle:

> **Program against abstractions, not concrete implementations.**

---

# 5. Interface Properties

Interfaces can define properties.

```c#
interface IPerson
{
    string Name { get; set; }
    int Age { get; }
}
```

Implementation:

```c#
class Student : IPerson
{
    public string Name { get; set; }

    public int Age { get; private set; }

    public Student(string name, int age)
    {
        Name = name;
        Age = age;
    }
}
```

Notice:

```c#
int Age { get; }
```

means consumers of the interface can read `Age`, but the interface doesn't require a public setter.

---

# 6. Interface vs Class

A class can contain implementation and state:

```c#
class Car
{
    private int speed;

    public void Accelerate()
    {
        speed += 10;
    }
}
```

An interface primarily defines a contract:

```c#
interface IVehicle
{
    void Accelerate();
}
```

Conceptually:

|Class|Interface|
|---|---|
|Can contain instance fields|Cannot have ordinary instance fields|
|Can contain implementation|Primarily defines contracts|
|Can have constructors|Cannot have instance constructors|
|Can inherit one class|A type can implement multiple interfaces|
|Represents an implementation/type|Represents a capability/contract|
|Can be instantiated if concrete|Cannot be instantiated directly|

---

# 7. Multiple Interface Inheritance

One of the biggest advantages of interfaces is that a class can implement **multiple interfaces**.

```c#
interface IPrintable
{
    void Print();
}

interface IScannable
{
    void Scan();
}
```

A printer can implement both:

```c#
class Printer : IPrintable, IScannable
{
    public void Print()
    {
        Console.WriteLine("Printing...");
    }

    public void Scan()
    {
        Console.WriteLine("Scanning...");
    }
}
```

Usage:

```c#
Printer printer = new Printer();

printer.Print();
printer.Scan();
```

This is different from class inheritance:

```c#
class A { }
class B { }

// Not allowed:
// class C : A, B
```

C# doesn't support multiple **class** inheritance, but it does support multiple **interfaces**.

---

# 8. Interface Inheritance

An interface can inherit another interface.

```c#
interface IAnimal
{
    void Eat();
}

interface IMammal : IAnimal
{
    void Walk();
}
```

Now a class implementing `IMammal` must implement both:

```c#
class Dog : IMammal
{
    public void Eat()
    {
        Console.WriteLine("Dog eats");
    }

    public void Walk()
    {
        Console.WriteLine("Dog walks");
    }
}
```

The hierarchy is:

```
IAnimal
   ↑
IMammal
   ↑
 Dog
```

---

# 9. Explicit Interface Implementation

This is an important C# feature.

Suppose two interfaces contain methods with the same name:

```c#
interface IPrinter
{
    void Start();
}

interface IScanner
{
    void Start();
}
```

A class can implement them explicitly:

```c#
class Machine : IPrinter, IScanner
{
    void IPrinter.Start()
    {
        Console.WriteLine("Printer started");
    }

    void IScanner.Start()
    {
        Console.WriteLine("Scanner started");
    }
}
```

You cannot do:

```c#
Machine machine = new Machine();

// machine.Start();   // Error
```

Instead:

```c#
IPrinter printer = new Machine();
printer.Start();

IScanner scanner = new Machine();
scanner.Start();
```

Output:

```
Printer started
Scanner started
```

### Why use explicit implementation?

It is useful when:

- Two interfaces have conflicting members.
- You don't want an interface member exposed as part of the class's public API.
- Different interfaces require different implementations.

---

# 10. Interface Reference vs Class Reference

Consider:

```c#
interface IAnimal
{
    void Speak();
}

class Dog : IAnimal
{
    public void Speak()
    {
        Console.WriteLine("Bark");
    }

    public void Fetch()
    {
        Console.WriteLine("Fetching");
    }
}
```

You can write:

```c#
Dog dog = new Dog();

dog.Speak();
dog.Fetch();
```

But:

```c#
IAnimal animal = new Dog();

animal.Speak();
```

This won't compile:

```
animal.Fetch(); // Error
```

Why?

Because the **reference type** is `IAnimal`.

The actual object is still a `Dog`, but through an `IAnimal` reference you can access only the contract exposed by `IAnimal`.

This is a key concept in polymorphism.

---

# 11. Interfaces and Dependency Injection

Interfaces are heavily used in modern .NET applications.

Suppose you have:

```c#
interface IPaymentService
{
    void Pay(decimal amount);
}
```

Implementation:

```c#
class StripePaymentService : IPaymentService
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid {amount} using Stripe");
    }
}
```

Your business logic can depend on the interface:

```c#
class OrderService
{
    private readonly IPaymentService paymentService;

    public OrderService(IPaymentService paymentService)
    {
        this.paymentService = paymentService;
    }

    public void PlaceOrder(decimal amount)
    {
        paymentService.Pay(amount);
    }
}
```

Then:

```c#
IPaymentService payment = new StripePaymentService();

OrderService orderService = new OrderService(payment);

orderService.PlaceOrder(1000);
```

The `OrderService` doesn't need to know that the implementation is Stripe.

It only knows:

```c#
IPaymentService
```

This creates **loose coupling**.

---

# 12. Interfaces and Unit Testing

Interfaces make testing much easier.

Suppose:

```c#
interface IUserRepository
{
    User GetUser(int id);
}
```

Production implementation:

```c#
class SqlUserRepository : IUserRepository
{
    public User GetUser(int id)
    {
        // Database access
        throw new NotImplementedException();
    }
}
```

For testing, you can use:

```c#
class FakeUserRepository : IUserRepository
{
    public User GetUser(int id)
    {
        return new User
        {
            Id = id,
            Name = "Test User"
        };
    }
}
```

Then:

```c#
IUserRepository repository = new FakeUserRepository();
```

You don't need a real database for the test.

This is one reason interfaces are common in:

- ASP.NET Core
- Web APIs
- Enterprise applications
- Clean Architecture
- Domain-Driven Design
- Unit testing

---

# 13. Interfaces and `is`

You can check whether an object implements an interface:

```c#
if (obj is IAnimal)
{
    Console.WriteLine("Object is an animal");
}
```

Better, you can pattern match:

```c#
if (obj is IAnimal animal)
{
    animal.Speak();
}
```

---

# 14. Interfaces and `as`

You can also use `as`:

```c#
IAnimal? animal = obj as IAnimal;

if (animal != null)
{
    animal.Speak();
}
```

However, modern C# generally favors pattern matching:

```c#
if (obj is IAnimal animal)
{
    animal.Speak();
}
```

because it is concise and avoids a separate null check.

---

# 15. Default Interface Implementations

Modern C# allows interfaces to provide implementations for some members.

Example:

```c#
interface ILogger
{
    void Log(string message);

    void LogError(string message)
    {
        Log("ERROR: " + message);
    }
}
```

A class only needs to implement:

```c#
class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

Then:

```c#
ILogger logger = new ConsoleLogger();

logger.Log("Hello");
logger.LogError("Something went wrong");
```

The default implementation of `LogError()` is used.

### Why was this introduced?

One major reason is **versioning interfaces**.

Suppose an interface is already implemented by hundreds of classes. Adding a new abstract member would normally force every implementation to change.

A default implementation can sometimes allow the interface to evolve without immediately breaking all implementations.

---

# 16. Static Members in Interfaces

Modern C# also supports static interface members, particularly useful for generic abstractions.

For example, interfaces can define static abstract members:

```c#
interface IAddable<T>
{
    static abstract T Add(T a, T b);
}
```

A type can implement it:

```c#
class Number : IAddable<Number>
{
    public int Value { get; }

    public Number(int value)
    {
        Value = value;
    }

    public static Number Add(Number a, Number b)
    {
        return new Number(a.Value + b.Value);
    }
}
```

This feature is important in advanced generic programming and is used by parts of the modern .NET generic math ecosystem.

---

# 17. Generic Interfaces

Interfaces can be generic.

```c#
interface IRepository<T>
{
    void Add(T item);
    T GetById(int id);
}
```

Implementation:

```c#
class UserRepository : IRepository<User>
{
    public void Add(User user)
    {
        // Add user
    }

    public User GetById(int id)
    {
        return new User();
    }
}
```

You could also have:

```c#
IRepository<Product>
IRepository<Order>
IRepository<Customer>
```

This gives reusable abstractions.

---

# 18. Generic Variance

C# interfaces also support **covariance** and **contravariance**.

### Covariance: `out`

```c#
interface IProducer<out T>
{
    T Produce();
}
```

An `IProducer<Dog>` can be treated as an `IProducer<Animal>` when the variance rules allow it.

### Contravariance: `in`

```c#
interface IConsumer<in T>
{
    void Consume(T item);
}
```

These features become particularly important when working with:

- Generic collections
- Delegates
- LINQ
- Dependency injection
- Generic APIs

---

# 19. `IEnumerable<T>` Is an Interface

One of the most important interfaces you'll encounter in C# is:

```c#
IEnumerable<T>
```

For example:

```c#
List<int> numbers = new List<int>
{
    10, 20, 30
};
```

You can write:

```c#
IEnumerable<int> numbers = new List<int>
{
    10, 20, 30
};
```

Then:

```c#
foreach (int number in numbers)
{
    Console.WriteLine(number);
}
```

The variable doesn't care whether the actual implementation is:

```c#
List<int>
```

or:

```c#
int[]
```

or another enumerable type.

It only requires the `IEnumerable<int>` contract.

---

# 20. Common .NET Interfaces

You'll encounter these frequently:

|Interface|Purpose|
|---|---|
|`IEnumerable<T>`|Iterating over a sequence|
|`ICollection<T>`|Collection operations|
|`IList<T>`|List-like collection|
|`IDictionary<TKey,TValue>`|Key/value collection|
|`IComparable<T>`|Comparing objects|
|`IEquatable<T>`|Equality comparison|
|`IDisposable`|Resource cleanup|
|`IAsyncDisposable`|Asynchronous resource cleanup|
|`ICloneable`|Cloning objects|
|`IFormattable`|Custom formatting|
|`IQueryable<T>`|Queryable data sources|

For example, `IDisposable` is extremely important:

```c#
class DatabaseConnection : IDisposable
{
    public void Dispose()
    {
        Console.WriteLine("Connection closed");
    }
}
```

Then:

```c#
using (DatabaseConnection connection = new DatabaseConnection())
{
    // Use connection
}
```

`Dispose()` is automatically called when leaving the `using` block.

---

# 21. Interface vs Abstract Class

This is a common interview question.

### Interface

```c#
interface IVehicle
{
    void Start();
}
```

### Abstract class

```c#
abstract class Vehicle
{
    public abstract void Start();

    public void Stop()
    {
        Console.WriteLine("Stopped");
    }
}
```

The major difference is that an abstract class can represent a **shared base implementation/state**, whereas an interface primarily represents a **contract/capability**.

|Feature|Interface|Abstract Class|
|---|---|---|
|Multiple inheritance|✅ Multiple interfaces|❌ One base class|
|Instance fields|❌ Ordinary instance fields|✅|
|Constructors|❌ Instance constructors|✅|
|Abstract methods|✅|✅|
|Concrete methods|✅ Modern C# supports defaults|✅|
|Properties|✅|✅|
|Events|✅|✅|
|Shared state|Generally no instance state|✅|
|Best conceptual use|Contract/capability|Common base/type hierarchy|

### Rule of thumb

Use an **interface** when you're saying:

> "This type can do X."

Use an **abstract class** when you're saying:

> "This type is a specialized form of Y and shares implementation/state with Y."

For example:

```c#
IFlyable
ISwimmable
IPrintable
```

are capabilities.

Whereas:

```c
Animal
Vehicle
Employee
Shape
```

can represent base abstractions.

---

# 22. Interface Segregation Principle

Interfaces are also central to **SOLID**.

The **Interface Segregation Principle (ISP)** says:

> Clients should not be forced to depend on methods they do not use.

Bad:

```c#
interface IMachine
{
    void Print();
    void Scan();
    void Fax();
}
```

Suppose a basic printer only supports printing.

It is forced to implement:

```c
Scan();
Fax();
```

Better:

```c#
interface IPrinter
{
    void Print();
}

interface IScanner
{
    void Scan();
}

interface IFax
{
    void Fax();
}
```

Now:

```c#
class BasicPrinter : IPrinter
{
    public void Print()
    {
        Console.WriteLine("Printing");
    }
}
```

And:

```c#
class MultiFunctionPrinter : IPrinter, IScanner, IFax
{
    public void Print() { }
    public void Scan() { }
    public void Fax() { }
}
```

This is a much cleaner design.

---

# 23. Important Interview Example

Consider:

```c#
interface A
{
    void Test();
}

interface B
{
    void Test();
}

class C : A, B
{
    public void Test()
    {
        Console.WriteLine("Hello");
    }
}
```

Does this compile?

**Yes.**

The single `Test()` implementation satisfies both interfaces.

```c#
A a = new C();
B b = new C();

a.Test();
b.Test();
```

Both call the same implementation.

But if you need different behavior:

```c#
class C : A, B
{
    void A.Test()
    {
        Console.WriteLine("A");
    }

    void B.Test()
    {
        Console.WriteLine("B");
    }
}
```

Now:

```c#
A a = new C();
B b = new C();

a.Test(); // A
b.Test(); // B
```

---

# 24. The Most Important Mental Model

Think of an interface as a **capability contract**.

```
             ┌──────────────┐
             │   IAnimal    │
             │              │
             │   Speak()    │
             └──────┬───────┘
                    │
          ┌─────────┴─────────┐
          │                   │
     ┌────▼────┐         ┌────▼────┐
     │   Dog   │         │   Cat   │
     │         │         │         │
     │ Bark()  │         │ Meow()  │
     └─────────┘         └─────────┘
```

The important abstraction is:

```
IAnimal
```

not:

```
Dog
```

or:

```
Cat
```

Code can therefore operate on **any object satisfying the contract**.

---

## 25. In One Example

Putting the major concepts together:

```c#
interface IPayment
{
    void Pay(decimal amount);
}

class CreditCardPayment : IPayment
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ₹{amount} using Credit Card");
    }
}

class UpiPayment : IPayment
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ₹{amount} using UPI");
    }
}

class Checkout
{
    private readonly IPayment payment;

    public Checkout(IPayment payment)
    {
        this.payment = payment;
    }

    public void CompletePayment(decimal amount)
    {
        payment.Pay(amount);
    }
}
```

Now:

```c#
IPayment payment = new UpiPayment();

Checkout checkout = new Checkout(payment);

checkout.CompletePayment(500);
```

The `Checkout` class doesn't care whether payment is:

```c
UpiPayment
CreditCardPayment
PayPalPayment
BankTransferPayment
```

It only knows:

```
IPayment
```

That is the **core value of interfaces in C#**: **abstraction + polymorphism + loose coupling**, which makes large applications easier to extend, test, and maintain.


