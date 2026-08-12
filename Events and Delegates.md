

events and delegates c\#

1. **Methods → delegates**
2. **Delegates → multicast delegates**
3. **Anonymous methods / lambdas**
4. **Built-in delegates: `Action`, `Func`, `Predicate`**
5. **Events**
6. **Event publishers/subscribers**
7. **EventHandler patterns**
8. **Custom event accessors**
9. **Delegates/events in real architectures**
10. **Thread safety, memory leaks, async events, and production practices**

---

# 1. Why do delegates exist?

Suppose you have a method:

```c#
static void PrintMessage(string message)
{
    Console.WriteLine(message);
}
```

Normally, you call it directly:

```c#
PrintMessage("Hello");
```

But sometimes you want to **pass a method around as data**.

For example:

```c#
DoSomething(PrintMessage);
```

This is where a **delegate** comes in.

A delegate is essentially a **type-safe reference to a method**.

Think:

```
Method
  ↓
Delegate
  ↓
Pass the method around
  ↓
Invoke it somewhere else
```

---

# 2. Your first delegate

Define a delegate:

```c#
delegate void MessageHandler(string message);
```

This says:

> `MessageHandler` can reference any method that takes a `string` and returns `void`.

Now:

```c#
static void PrintMessage(string message)
{
    Console.WriteLine(message);
}

static void Main()
{
    MessageHandler handler = PrintMessage;

    handler("Hello");
}
```

Output:

```
Hello
```

The important relationship is:

```c#
MessageHandler handler = PrintMessage;
```

Then:

```c#
handler("Hello");
```

is effectively invoking:

```c#
PrintMessage("Hello");
```

---

# 3. Delegate signature matching

This method works:

```c#
static void Print(string message)
{
    Console.WriteLine(message);
}
```

because:

```
string → void
```

matches:

```c#
delegate void MessageHandler(string message);
```

But this doesn't:

```c#
static int Calculate(int x)
{
    return x * 2;
}
```

because its signature is:

```
int → int
```

while our delegate expects:

```
string → void
```

This type safety is one of the major advantages of C# delegates.

---

# 4. Passing delegates to methods

This is where delegates become genuinely useful.

```c#
delegate void MessageHandler(string message);

static void ProcessMessage(
    string message,
    MessageHandler handler)
{
    handler(message);
}
```

Now:

```c#
static void Log(string message)
{
    Console.WriteLine($"LOG: {message}");
}

static void Main()
{
    ProcessMessage("User logged in", Log);
}
```

Output:

```
LOG: User logged in
```

The `ProcessMessage` method doesn't need to know **which method** will process the message.

That's the key idea:

> **Delegates allow behavior to be supplied from outside.**

---

# 5. Multiple methods — multicast delegates

A delegate can reference multiple methods.

```c#
delegate void MessageHandler(string message);

static void Log(string message)
{
    Console.WriteLine($"Log: {message}");
}

static void Save(string message)
{
    Console.WriteLine($"Save: {message}");
}

static void Main()
{
    MessageHandler handler = Log;

    handler += Save;

    handler("Hello");
}
```

Output:

```
Log: Hello
Save: Hello
```

The invocation list is:

```
handler
 ├── Log
 └── Save
```

You can remove a method:

```c#
handler -= Save;
```

Now only `Log` executes.

---

# 6. Delegates with lambdas

Modern C# rarely requires you to explicitly declare custom delegates for simple situations.

Instead:

```c#
Action<string> handler = message =>
{
    Console.WriteLine(message);
};
```

Or:

```c#
handler += message =>
{
    Console.WriteLine($"Received: {message}");
};
```

Lambdas are heavily used throughout modern .NET.

---

# 7. `Action`, `Func`, and `Predicate`

These three are extremely important for industry-level C#.

## `Action`

Represents a method returning `void`.

```c#
Action<string> print = message =>
{
    Console.WriteLine(message);
};

print("Hello");
```

Equivalent conceptually to:

```
delegate void Something(string message);
```

---

## `Func`

Represents a method that **returns a value**.

```
Func<int, int> square = x => x * x;

int result = square(5);

Console.WriteLine(result);
```

Output:

```
25
```

The **last generic parameter is the return type**.

```
Func<int, int, int>
```

means:

```
int + int → int
```

Example:

```
Func<int, int, int> add = (a, b) => a + b;
```

---

## `Predicate`

Represents:

```
input → bool
```

Example:

```
Predicate<int> isEven = x => x % 2 == 0;

Console.WriteLine(isEven(10));
```

Output:

```
True
```

You'll frequently see this with collections.

---

# 8. Now: what is an event?

This is where many beginners get confused.

An **event is built on delegates**, but it adds access restrictions.

Consider:

```
public Action<string>? MessageReceived;
```

Any outside code could do:

```
obj.MessageReceived = null;
```

or:

```
obj.MessageReceived = SomeMethod;
```

That's usually not what you want.

An **event** allows the owning class to control invocation.

```
public event Action<string>? MessageReceived;
```

Subscribers can do:

```
obj.MessageReceived += Handler;
```

but they cannot invoke it:

```
obj.MessageReceived("Hello"); // ❌
```

Only the declaring type can raise the event.

That's the fundamental difference.

---

# 9. Your first event

Let's build something realistic.

```
public class OrderService
{
    public event Action<string>? OrderCreated;

    public void CreateOrder()
    {
        Console.WriteLine("Creating order...");

        OrderCreated?.Invoke("Order created");
    }
}
```

Subscriber:

```
class Program
{
    static void Main()
    {
        var service = new OrderService();

        service.OrderCreated += OnOrderCreated;

        service.CreateOrder();
    }

    static void OnOrderCreated(string message)
    {
        Console.WriteLine(message);
    }
}
```

Flow:

```
Program
   │
   │ subscribes
   ▼
OrderCreated event
   ▲
   │ raised by
   │
OrderService
```

This is the **publisher/subscriber pattern**.

# 10. The industry-standard event pattern

For production .NET code, you'll very often encounter:

```
EventHandler
```

and:

```
EventHandler<TEventArgs>
```

The standard pattern looks like this:

```
public class OrderCreatedEventArgs : EventArgs
{
    public int OrderId { get; }

    public OrderCreatedEventArgs(int orderId)
    {
        OrderId = orderId;
    }
}
```

Then:

```
public class OrderService
{
    public event EventHandler<OrderCreatedEventArgs>? OrderCreated;

    public void CreateOrder(int orderId)
    {
        Console.WriteLine("Creating order...");

        var args = new OrderCreatedEventArgs(orderId);

        OrderCreated?.Invoke(this, args);
    }
}
```

Subscriber:

```
var service = new OrderService();

service.OrderCreated += OnOrderCreated;

service.CreateOrder(123);
```

Handler:

```
static void OnOrderCreated(
    object? sender,
    OrderCreatedEventArgs e)
{
    Console.WriteLine(
        $"Order {e.OrderId} was created.");
}
```

This is much closer to conventional .NET event design.

---

# 11. Why `sender`?

The first parameter:

```
object? sender
```

usually identifies **the object that raised the event**.

For example:

```
OrderService service = new();

service.OrderCreated += OnOrderCreated;
```

When:

```
OrderCreated?.Invoke(this, args);
```

runs:

```
sender
   ↓
OrderService instance
```

This allows one handler to potentially work with multiple publishers.

---

# 12. Why `EventArgs`?

`EventArgs` provides a standard mechanism for event data.

For example:

```
public class PaymentCompletedEventArgs : EventArgs
{
    public decimal Amount { get; }
    public string TransactionId { get; }

    public PaymentCompletedEventArgs(
        decimal amount,
        string transactionId)
    {
        Amount = amount;
        TransactionId = transactionId;
    }
}
```

Then:

```
public event EventHandler<PaymentCompletedEventArgs>?
    PaymentCompleted;
```

This is extensible.

Instead of:

```
Action<decimal, string>
```

you have a meaningful domain type:

```
PaymentCompletedEventArgs
```

which can later gain additional properties.

---

# 13. The protected `On...` pattern

A common .NET design is to separate **raising** an event from the business operation.

Instead of:

```
OrderCreated?.Invoke(this, args);
```

directly inside `CreateOrder`, use:

```
protected virtual void OnOrderCreated(
    OrderCreatedEventArgs e)
{
    OrderCreated?.Invoke(this, e);
}
```

Then:

```
public void CreateOrder(int orderId)
{
    // Business logic

    OnOrderCreated(
        new OrderCreatedEventArgs(orderId));
}
```

Complete example:

```
public class OrderService
{
    public event EventHandler<OrderCreatedEventArgs>?
        OrderCreated;

    public void CreateOrder(int orderId)
    {
        Console.WriteLine("Creating order...");

        OnOrderCreated(
            new OrderCreatedEventArgs(orderId));
    }

    protected virtual void OnOrderCreated(
        OrderCreatedEventArgs e)
    {
        OrderCreated?.Invoke(this, e);
    }
}
```

Why?

Because derived classes can customize the raising behavior when appropriate.

---

# 14. The most important mental model

Don't memorize syntax first.

Understand the architecture.

```
              Publisher
                 │
                 │ raises
                 ▼
              Event
                 │
        ┌────────┼────────┐
        │        │        │
        ▼        ▼        ▼
   Subscriber Subscriber Subscriber
```

Example:

```
OrderService
     │
     │ OrderCreated
     ▼
 ┌───────────────┐
 │               │
 ▼               ▼
EmailService   AuditService
```

`OrderService` doesn't need to know that:

- an email is sent
- an audit record is created
- analytics are updated
- a UI notification appears

It simply says:

> **An order was created.**

Subscribers decide what they care about.

That is **decoupling**.

---

# 15. Delegates vs events

This distinction is essential.

|Feature|Delegate|Event|
|---|---|---|
|Reference methods|✅|✅|
|Multiple subscribers|✅|✅|
|Outside code can invoke|Usually yes|❌|
|Publisher controls invocation|❌|✅|
|Common use|Callbacks|Notifications|
|Typical syntax|`Action`, `Func`|`event EventHandler`|

A useful rule:

> **Delegate = "call this behavior."**

> **Event = "notify whoever is interested."**

---

# 16. Real-world example

Imagine an e-commerce application:

```
OrderService
     │
     │ OrderPlaced
     ▼
 ┌─────────────┬─────────────┬──────────────┐
 ▼             ▼             ▼
Email       Inventory      Analytics
Service     Service         Service
```

The order service:

```
public class OrderService
{
    public event EventHandler<OrderPlacedEventArgs>?
        OrderPlaced;

    public void PlaceOrder(int orderId)
    {
        // Save order

        OnOrderPlaced(
            new OrderPlacedEventArgs(orderId));
    }

    protected virtual void OnOrderPlaced(
        OrderPlacedEventArgs e)
    {
        OrderPlaced?.Invoke(this, e);
    }
}
```

Email service:

```
orderService.OrderPlaced +=
    emailService.SendConfirmation;
```

Inventory:

```
orderService.OrderPlaced +=
    inventoryService.ReserveItems;
```

Analytics:

```
orderService.OrderPlaced +=
    analyticsService.TrackOrder;
```

Now the `OrderService` has no direct dependency on those implementations.

That's where events become architecturally useful.

# 17. But don't use events everywhere

This is an important industry distinction.

Events are excellent for:

- UI notifications
- domain notifications
- lifecycle notifications
- loosely coupled observers
- framework callbacks
- state changes

But events are **not automatically the best solution** for:

- request/response operations
- operations requiring a return value
- complex workflows
- guaranteed message delivery
- distributed communication

For example, don't turn this:

```
decimal CalculatePrice(Order order)
```

into an event.

That's a request for a result.

A delegate/function is more appropriate.

---

# 18. Events vs interfaces

Another important design decision:

### Event

Use when you're saying:

> "Something happened."

```
OrderPlaced
```

### Interface

Use when you're saying:

> "I need an object capable of performing this responsibility."

```
public interface IOrderProcessor
{
    Task ProcessAsync(Order order);
}
```

### Delegate

Use when you're saying:

> "Give me the behavior/callback I should execute."

```
Func<Order, Task>
```

These are different abstractions.

---

# 19. Memory leaks — an important production issue

Events have a subtle lifetime problem.

Suppose:

```
longLivedObject.SomeEvent += shortLivedObject.Handler;
```

The publisher may hold a reference to the subscriber through the delegate.

Therefore:

```
Long-lived publisher
        │
        ▼
event delegate
        │
        ▼
subscriber
```

If the subscriber should have died but remains subscribed, it can remain reachable and therefore not be garbage collected.

This is especially important in:

- desktop applications
- UI applications
- long-running services
- event aggregators
- singleton objects

Sometimes you need:

```
publisher.SomeEvent -= subscriber.Handler;
```

or an appropriate lifetime/disposal strategy.

---

# 20. A good learning progression

I'd recommend learning these in exactly this order:

### Level 1 — Fundamentals

Learn:

```
delegate
```

```
Action<T>
```

```
Func<T>
```

```
Predicate<T>
```

Understand:

- method group conversion
- invocation
- parameters
- return values
- multicast delegates

---

### Level 2 — Lambdas

Learn:

```
x => x * 2
```

and:

```
(x, y) => x + y
```

Understand:

- anonymous functions
- closures
- captured variables
- expression lambdas
- statement lambdas

---

### Level 3 — Events

Learn:

```
event
```

Understand:

- publisher
- subscriber
- subscription
- unsubscription
- event invocation
- event data

---

### Level 4 — .NET event conventions

Master:

```
EventHandler
```

```
EventHandler<TEventArgs>
```

```
EventArgs
```

and:

```
protected virtual OnSomething(...)
```

---

### Level 5 — Advanced delegates

Then study:

```
MulticastDelegate
```

delegate variance:

```
Action<in T>
Func<in T, out TResult>
```

closures:

```
int count = 0;

Action action = () =>
{
    count++;
};
```

and delegate allocation/performance.

---

### Level 6 — Production architecture

Finally learn:

- observer pattern
- event aggregation
- domain events
- application events
- event-driven architecture
- synchronous vs asynchronous events
- event lifetime
- memory leaks
- thread safety
- exception handling
- event ordering
- cancellation
- distributed messaging

---

# 21. One crucial distinction: C# events vs messaging systems

Don't confuse:

```
orderService.OrderPlaced += Handler;
```

with:

```
Kafka
RabbitMQ
Azure Service Bus
AWS SNS/SQS
```

A C# event is generally **in-process**.

```
Application Process
┌─────────────────────────────┐
│                             │
│ Publisher → Event → Handler │
│                             │
└─────────────────────────────┘
```

A message broker can be **distributed**:

```
Service A
   │
   ▼
Message Broker
   │
   ├──────► Service B
   ├──────► Service C
   └──────► Service D
```

The concepts are related, but the reliability and delivery semantics are completely different.

---

# 22. The target you should aim for

By the end, you should be able to look at this:

```
public event EventHandler<OrderPlacedEventArgs>? OrderPlaced;
```

and understand **every part of it**:

```
public
  │
  └── accessible outside the class

event
  │
  └── subscription-based delegate abstraction

EventHandler<T>
  │
  └── standard .NET event delegate

OrderPlacedEventArgs
  │
  └── strongly typed event data

?
  │
  └── nullable reference annotation
```

And you should understand the complete lifecycle:

```
Define event
     ↓
Subscribe
     ↓
Business operation occurs
     ↓
Create EventArgs
     ↓
Raise event
     ↓
Delegate invocation list
     ↓
Subscribers execute
     ↓
Unsubscribe when lifetime requires it
```

## Recommended hands-on path

We can turn this into a **progressive C# masterclass**:

**Part 1:** Delegates from absolute basics  
**Part 2:** `Action`, `Func`, `Predicate`  
**Part 3:** Lambdas + closures  
**Part 4:** Multicast delegates  
**Part 5:** Events from scratch  
**Part 6:** `EventHandler<TEventArgs>` and .NET conventions  
**Part 7:** Build an event-driven e-commerce example  
**Part 8:** Advanced events, memory leaks, thread safety  
**Part 9:** Domain events + clean architecture  
**Part 10:** Production-grade patterns and interview problems

The best way to learn this is **code-first**: each part should have small exercises, progressively harder problems, and then a production-style project.