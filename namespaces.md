
# C# Namespaces

A **namespace** in C# is a logical container used to organize types such as:

- Classes
- Interfaces
- Structs
- Enums
- Delegates
- Other namespaces

The main purpose is **organization and name collision avoidance**.

```c#
namespace MyCompany.MyProduct.Users;

public class UserService
{
}
```

You can then reference it with:

```c#
using MyCompany.MyProduct.Users;

var service = new UserService();
```

---

## 1. Why namespaces exist

Imagine two libraries both define:

```c#
public class Logger
{
}
```

Without namespaces, C# would have difficulty distinguishing them.

With namespaces:

```c#
namespace CompanyA.Logging;

public class Logger
{
}
```

and:

```c#
namespace CompanyB.Logging;

public class Logger
{
}
```

Now they are different fully qualified types:

```c#
CompanyA.Logging.Logger
CompanyB.Logging.Logger
```

You can explicitly use either:

```c#
var logger1 = new CompanyA.Logging.Logger();
var logger2 = new CompanyB.Logging.Logger();
```

So namespaces provide a **logical naming system**.

---

# 2. Namespace vs `using`

This distinction is extremely important.

### Namespace declaration

```c#
namespace MyCompany.Application.Services;

public class UserService
{
}
```

This says:

> `UserService` belongs to `MyCompany.Application.Services`.

### `using`

```c#
using MyCompany.Application.Services;
```

This says:

> Within this file, I want to refer to types in that namespace without writing their fully qualified names.

For example:

```c#
using MyCompany.Application.Services;

UserService service = new UserService();
```

Instead of:

```c#
MyCompany.Application.Services.UserService service =
    new MyCompany.Application.Services.UserService();
```

**`using` does not import code into your application.** It primarily makes names easier to resolve.

---

# 3. Namespace hierarchy

Namespaces commonly form a hierarchy.

For example:

```
MyCompany
│
├── Application
│   ├── Services
│   ├── Interfaces
│   └── DTOs
│
├── Domain
│   ├── Entities
│   ├── ValueObjects
│   └── Enums
│
├── Infrastructure
│   ├── Persistence
│   └── Messaging
│
└── Web
    ├── Controllers
    └── Middleware
```

In C#:

```c#
namespace MyCompany.Application.Services;

public class OrderService
{
}
```

```c#
namespace MyCompany.Domain.Entities;

public class Order
{
}
```

```c#
namespace MyCompany.Infrastructure.Persistence;

public class OrderRepository
{
}
```

This becomes especially useful in **large enterprise applications**.

---

# 4. The "confluence" / convergence of namespaces

If by **confluence** you mean how namespaces fit together and interact, the key idea is that **namespace hierarchy is independent of physical folders**.

For example, these files could physically be anywhere:

```c
SomeFolder/A.cs
AnotherFolder/B.cs
RandomFolder/C.cs
```

but contain:

```c#
namespace Company.Product.Domain;

class A { }
```

```c#
namespace Company.Product.Domain;

class B { }
```

```c#
namespace Company.Product.Domain;

class C { }
```

From the compiler's perspective, all three types belong to:

```
Company.Product.Domain
```

The filesystem structure is a convention, not what defines the namespace.

That distinction matters.

### Namespace

A **logical naming boundary**.

### Folder

A **physical organization mechanism**.

Modern .NET projects usually make the two correspond because it makes the codebase easier for humans to navigate.

---

# 5. File-scoped namespaces

Modern C# generally favors **file-scoped namespaces**.

Instead of:

```c#
namespace MyCompany.Application.Services
{
    public class UserService
    {
    }
}
```

prefer:

```c#
namespace MyCompany.Application.Services;

public class UserService
{
}
```

The second form is cleaner because the entire file belongs to that namespace.

### Industry preference

For modern C# projects:

```c#
namespace Company.Product.Feature;

public class SomeService
{
}
```

is generally preferable to:

```c#
namespace Company.Product.Feature
{
    public class SomeService
    {
    }
}
```

unless you have a specific reason to use block-scoped namespaces.

---

# 6. Namespace naming standards

A common industry convention is:

```
<Company>.<Product>.<Component>
```

For example:

```
Microsoft.AspNetCore.Mvc
```

or your own application:

```
Contoso.ECommerce
Contoso.ECommerce.Domain
Contoso.ECommerce.Application
Contoso.ECommerce.Infrastructure
Contoso.ECommerce.Web
```

### Use PascalCase

Good:

```
namespace Contoso.ECommerce.OrderProcessing;
```

Avoid:

```
namespace contoso.ecommerce.orderprocessing;
```

Avoid underscores:

```
namespace Contoso.ECommerce.Order_Processing;
```

Generally use **PascalCase for namespace components**.

---

# 7. Namespace should describe ownership/context

A good namespace answers:

> "Where does this type logically belong?"

For example:

```c#
namespace ECommerce.Domain.Entities;

public class Customer
{
}
```

```c#
namespace ECommerce.Application.Services;

public class CustomerService
{
}
```

```c#
namespace ECommerce.Infrastructure.Persistence;

public class CustomerRepository
{
}
```

This gives developers immediate context.

---

# 8. Don't make namespaces unnecessarily deep

This is a common mistake.

You could theoretically write:

```c#
namespace Company.Product.Application.Features.CustomerManagement.Services.Implementations.Internal;
```

But that is often excessive.

Prefer something like:

```c#
namespace Company.Product.Application.Customers;
```

Then:

```c#
public class CustomerService
{
}
```

The namespace should provide **useful architectural information**, not encode every implementation detail.



# 9. Namespace and project structure

A common enterprise .NET architecture might look like:

```
ECommerce.sln

src/
├── ECommerce.Domain/
│   ├── Entities/
│   │   └── Order.cs
│   └── ValueObjects/
│       └── Money.cs
│
├── ECommerce.Application/
│   ├── Orders/
│   │   ├── CreateOrder.cs
│   │   └── OrderService.cs
│   └── Interfaces/
│       └── IOrderRepository.cs
│
├── ECommerce.Infrastructure/
│   ├── Persistence/
│   │   └── OrderRepository.cs
│   └── Messaging/
│       └── MessagePublisher.cs
│
└── ECommerce.Api/
    ├── Controllers/
    │   └── OrdersController.cs
    └── Program.cs
```

Namespaces might correspond:

```
ECommerce.Domain.Entities
ECommerce.Domain.ValueObjects

ECommerce.Application.Orders
ECommerce.Application.Interfaces

ECommerce.Infrastructure.Persistence
ECommerce.Infrastructure.Messaging

ECommerce.Api.Controllers
```

This is a very common and maintainable approach.

---

# 10. Namespaces don't create architectural boundaries

This is a subtle but **very important industry concept**.

Suppose you have:

```c#
namespace ECommerce.Domain;

public class Order
{
}
```

and:

```c#
namespace ECommerce.Infrastructure;

public class Database
{
}
```

The namespaces don't prevent this:

```c#
namespace ECommerce.Domain;

public class SomeDomainClass
{
    private ECommerce.Infrastructure.Database _database;
}
```

The compiler doesn't care that `Domain` is supposed to be independent of `Infrastructure`.

**Namespaces organize names.**

They do **not**, by themselves, enforce:

- Dependency direction
- Encapsulation
- Layer boundaries
- Security
- Module boundaries
- Architecture

Those require project references, access modifiers, architecture tests, analyzers, or other mechanisms.

This is one of the most important distinctions between **organization** and **architecture**.

---

# 11. Namespace and assembly are different

Another important distinction:

```
Namespace
    ↓
Logical naming

Assembly
    ↓
Compiled deployment unit
```

For example:

```c#
namespace Company.Product.Users;

public class User
{
}
```

could exist inside:

```
Company.Product.Domain.dll
```

The namespace doesn't determine the assembly.

Conversely, one assembly can contain many namespaces:

```c#
Company.Product.Domain.dll

    Company.Product.Domain
    Company.Product.Domain.Entities
    Company.Product.Domain.ValueObjects
```

And one namespace can technically span multiple assemblies.

---

# 12. Namespace collisions

Suppose you have:

```C#
namespace Company.Internal;

public class Logger
{
}
```

and:

```c#
namespace ThirdParty.Logging;

public class Logger
{
}
```

You could write:

```c#
using Company.Internal;
using ThirdParty.Logging;
```

Then:

```
Logger logger;
```

may become ambiguous.

You can resolve this using an alias:

```c#
using InternalLogger = Company.Internal.Logger;
using ExternalLogger = ThirdParty.Logging.Logger;
```

Then:

```c#
InternalLogger a = new();
ExternalLogger b = new();
```

This is particularly useful when integrating multiple libraries.

---

# 13. Global usings

Modern .NET projects can use:

```C#
global using System;
global using System.Collections.Generic;
global using System.Linq;
```

Then every file in the project can use those namespaces without repeating:

```c#
using System;
using System.Collections.Generic;
using System.Linq;
```

ASP.NET Core projects commonly use this approach.

For example:

```c#
global using Microsoft.Extensions.DependencyInjection;
global using Microsoft.Extensions.Logging;
```

### Industry guideline

Use global usings for **genuinely ubiquitous dependencies**.

Don't put everything into global usings just to make individual files shorter.

Bad:

```c#
global using Company.Product.Everything;
global using Company.Product.Internal;
global using Company.Product.Experimental;
```

This hides dependencies and makes files harder to understand.


# 14. Implicit usings

Modern .NET projects can also enable:

```
<ImplicitUsings>enable</ImplicitUsings>
```

For example, a console project may automatically have common namespaces available.

This reduces boilerplate.

But again, understand what is happening:

```
ImplicitUsings
       ↓
Compiler/project configuration
       ↓
Common namespaces automatically available
```

It doesn't change the fundamental namespace system.

---

# 15. Namespace naming vs folder naming

A good rule is:

```
Folder structure ≈ Namespace structure
```

For example:

```
Services/
    PaymentService.cs
```

```
namespace MyCompany.Application.Services;
```

This makes navigation predictable.

However, don't treat the mapping as a technical requirement.

This is legal:

```
TotallyRandomFolder/
    PaymentService.cs
```

with:

```
namespace MyCompany.Application.Services;
```

But it creates unnecessary cognitive friction.

**Industry practice is to keep them aligned.**

---

# 16. Should every class get its own namespace?

No.

Don't do this:

```
namespace Company.Users.UserService;
```

```
namespace Company.Users.UserRepository;
```

Instead:

```
namespace Company.Users;
```

and:

```
public class UserService
{
}
```

```
public class UserRepository
{
}
```

Namespaces should represent **meaningful conceptual boundaries**, not individual types.

---

# 17. Avoid generic namespaces like `Common`

You will frequently encounter:

```
Company.Common
Company.Utilities
Company.Helpers
Company.Shared
```

These can become dumping grounds.

For example:

```
Company.Common
├── StringHelper.cs
├── DateHelper.cs
├── UserHelper.cs
├── PaymentHelper.cs
├── DatabaseHelper.cs
├── EmailHelper.cs
└── RandomStuff.cs
```

This indicates weak cohesion.

Prefer namespaces based on domain responsibility:

```
Company.Payments
Company.Identity
Company.Email
Company.Persistence
```

A useful principle is:

> **Namespace by cohesion, not convenience.**

---

# 18. A practical industry standard

For a medium/large C# application, I'd recommend something like:

```
Company.Product
│
├── Domain
│   ├── Entities
│   ├── ValueObjects
│   └── Events
│
├── Application
│   ├── Services
│   ├── Interfaces
│   └── DTOs
│
├── Infrastructure
│   ├── Persistence
│   ├── Messaging
│   └── ExternalServices
│
└── Api
    ├── Controllers
    ├── Middleware
    └── Filters
```

Namespaces:

```
Company.Product.Domain.Entities
Company.Product.Domain.ValueObjects
Company.Product.Domain.Events

Company.Product.Application.Services
Company.Product.Application.Interfaces
Company.Product.Application.DTOs

Company.Product.Infrastructure.Persistence
Company.Product.Infrastructure.Messaging
Company.Product.Infrastructure.ExternalServices

Company.Product.Api.Controllers
Company.Product.Api.Middleware
Company.Product.Api.Filters
```

And use:

```
namespace Company.Product.Application.Services;

public class OrderService
{
}
```

rather than deeply nested block syntax.

---

# 19. The deeper architectural idea

Think of a C# application as having several different concepts:

```
                    C# Application
                         │
          ┌──────────────┼──────────────┐
          │              │              │
      Namespace       Assembly       Project
          │              │              │
      Naming          Packaging      Build unit
          │              │              │
          └──────────────┼──────────────┘
                         │
                    Architecture
                         │
              Dependencies / Boundaries
```

They are related, but **not interchangeable**.

For example:

```
Company.Product.Domain.Entities
```

is a namespace.

```
Company.Product.Domain.dll
```

is an assembly.

```
Company.Product.Domain.csproj
```

is a project.

And:

```
Domain → Application → Infrastructure → API
```

could represent an architectural dependency model.

Understanding these distinctions is what separates merely _using namespaces_ from designing a maintainable .NET codebase.

## My recommended rules

|Rule|Recommendation|
|---|---|
|Naming|`Company.Product.Feature`|
|Case|PascalCase|
|File-scoped namespaces|**Prefer**|
|Folder ↔ namespace|Keep aligned|
|Namespace depth|Keep meaningful, avoid excessive depth|
|`Common` / `Helpers`|Use sparingly|
|`global using`|Only for ubiquitous dependencies|
|Namespace = architecture|**No**|
|Namespace = assembly|**No**|
|One namespace per class|**No**|
|Namespace purpose|Organization + name collision avoidance|
|Architecture enforcement|Projects, references, analyzers, tests|

### The mental model to remember

> **A namespace tells you what a type is called and where it logically belongs. It does not tell the compiler what architectural layer it is allowed to depend on.**

If you're learning C# seriously, the next useful step is to understand **`namespace` vs `using` vs assembly vs project vs DLL**, because those five concepts are often conflated even by intermediate developers.