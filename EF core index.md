
# EF Core Complete Learning Roadmap

## 0. Prerequisites — Don't Skip This

Before EF Core, you should be comfortable with:

### C\#

- Classes / objects
    
- Interfaces
    
- Generics
    
- LINQ
    
- `async` / `await`
    
- `IEnumerable<T>` vs `IQueryable<T>`
    
- Lambda expressions
    
- Dependency Injection
    
- Exceptions
    
- `using` / `IDisposable`
    
- Nullable reference types
    

### SQL

Learn:

```text
Database
 ├── Tables
 ├── Rows
 ├── Columns
 ├── Primary Keys
 ├── Foreign Keys
 ├── Indexes
 ├── Constraints
 └── Transactions
```

Then:

- `SELECT`
    
- `WHERE`
    
- `ORDER BY`
    
- `GROUP BY`
    
- `HAVING`
    
- `JOIN`
    
- `INNER JOIN`
    
- `LEFT JOIN`
    
- Subqueries
    
- Aggregations
    
- Indexes
    
- Transactions
    
- Normalization
    

### Critical concept

Understand this difference:

```csharp
IEnumerable<T>
```

vs

```csharp
IQueryable<T>
```

Because EF Core fundamentally relies on **LINQ → expression trees → SQL translation**.

---

# 1. [[What Exactly Is EF Core]]?

Understand:

> **EF Core is an ORM (Object-Relational Mapper) for .NET that allows you to work with relational databases using .NET objects and LINQ.**

Without EF Core:

```text
C# Application
      ↓
SQL string
      ↓
Database
      ↓
DataReader
      ↓
Map manually
      ↓
C# objects
```

With EF Core:

```text
C# Application
      ↓
LINQ
      ↓
EF Core
      ↓
SQL
      ↓
Database
      ↓
EF Core
      ↓
C# objects
```

Learn the terminology:

|Concept|Meaning|
|---|---|
|Entity|C# class mapped to DB|
|DbContext|Unit of work / DB session|
|DbSet|Represents entity collection|
|Model|EF's representation of your domain/database|
|Change Tracker|Tracks entity changes|
|Migration|Version-controls DB schema|
|LINQ|Query language used against entities|
|Provider|Database-specific EF implementation|

---

# 2. Build Your First EF Core Application

Create a small project:

```text
EFCoreLearning
│
├── Models
│   ├── Student.cs
│   └── Course.cs
│
├── Data
│   └── AppDbContext.cs
│
└── Program.cs
```

Start with:

```csharp
public class Student
{
    public int Id { get; set; }

    public string Name { get; set; } = string.Empty;

    public string Email { get; set; } = string.Empty;
}
```

Then:

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Student> Students => Set<Student>();

    protected override void OnConfiguring(
        DbContextOptionsBuilder options)
    {
        options.UseSqlServer(connectionString);
    }
}
```

Learn exactly what happens when:

```csharp
context.Students.Add(student);

context.SaveChanges();
```

Internally:

```text
Student object
     ↓
DbSet.Add()
     ↓
Change Tracker
     ↓
EntityState.Added
     ↓
SaveChanges()
     ↓
SQL INSERT
     ↓
Database
```

This **change-tracking mental model** becomes extremely important later.

---

# 3. DbContext — Master This

`DbContext` is probably the most important EF Core class.

Understand:

### DbContext responsibilities

```text
DbContext
 ├── Database connection
 ├── Entity tracking
 ├── Query execution
 ├── Change detection
 ├── SaveChanges
 ├── Transactions
 └── Model configuration
```

Learn:

```csharp
DbContext
DbSet<T>
DbContextOptions
SaveChanges()
SaveChangesAsync()
Database
ChangeTracker
```

Understand the lifecycle:

```text
HTTP Request
     ↓
Create DbContext
     ↓
Query
     ↓
Modify entities
     ↓
SaveChanges
     ↓
Dispose DbContext
```

### Industry rule

Usually:

> **One DbContext per request/unit of work.**

Avoid treating `DbContext` as a permanent application-wide singleton.

---

# 4. Database Configuration  

Learn:

```csharp
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

Understand:

- Connection strings
    
- SQL Server provider
    
- PostgreSQL provider
    
- SQLite
    
- DbContext configuration
    
- Dependency Injection
    
- Environment-specific configuration
    

Typical:

```text
appsettings.json
appsettings.Development.json
appsettings.Production.json
        ↓
Configuration
        ↓
DI
        ↓
DbContext
```

---

# 5. Code First

This is where EF Core becomes practical.

Learn:

```text
C# Classes
    ↓
EF Core Model
    ↓
Migration
    ↓
Database Schema
```

Example:

```csharp
public class Employee
{
    public int Id { get; set; }

    public string Name { get; set; } = string.Empty;

    public decimal Salary { get; set; }
}
```

Then learn migrations:

```bash
dotnet ef migrations add InitialCreate
```

and:

```bash
dotnet ef database update
```

Understand:

```text
Model change
     ↓
Migration
     ↓
Migration SQL
     ↓
Database schema change
```

---

# 6. EF Core Conventions  [[entity]]

Before configuration, understand what EF Core does automatically.

For example:

```csharp
public int Id { get; set; }
```

typically becomes:

```text
Primary Key
```

And:

```csharp
public int DepartmentId { get; set; }
```

may participate in a foreign-key relationship based on conventions.

Learn:

- Primary key conventions
    
- Foreign key conventions
    
- Table naming
    
- Column naming
    
- Required/optional properties
    
- String length
    
- Relationships
    
- Shadow properties
    

This helps you understand **what EF Core infers versus what you explicitly configure**.

---

# 7. Fluent API

Now learn proper model configuration.

Example:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>()
        .Property(x => x.Name)
        .HasMaxLength(100)
        .IsRequired();
}
```

Learn:

```csharp
HasKey()
HasIndex()
HasMaxLength()
IsRequired()
HasColumnName()
HasColumnType()
HasDefaultValue()
HasDefaultValueSql()
```

And:

```csharp
ToTable()
```

You should become comfortable reading:

```csharp
modelBuilder.Entity<Order>()
    .HasKey(x => x.Id);
```

and:

```csharp
modelBuilder.Entity<Order>()
    .HasIndex(x => x.OrderNumber)
    .IsUnique();
```

---

# 8. Data Annotations

Learn attributes such as:

```csharp
[Key]
[Required]
[MaxLength(100)]
[Column("employee_name")]
[Table("employees")]
```

But understand the industry preference:

> Use **Fluent API** for complex/centralized database configuration.

Data annotations are useful for simple metadata.

---

# 9. CRUD Operations

Now build basic database operations.

## Create

```csharp
context.Students.Add(student);

await context.SaveChangesAsync();
```

## Read

```csharp
var students = await context.Students.ToListAsync();
```

## Update

```csharp
student.Name = "John";

await context.SaveChangesAsync();
```

## Delete

```csharp
context.Students.Remove(student);

await context.SaveChangesAsync();
```

But don't stop at syntax.

Understand:

```text
EntityState
```

Possible states:

```text
Detached
Unchanged
Added
Modified
Deleted
```

This is heavily relevant to interviews.

---

# 10. LINQ — Become Very Strong

This is one of the most important EF Core skills.

Learn:

```csharp
Where()
Select()
OrderBy()
ThenBy()
Skip()
Take()
First()
FirstOrDefault()
Single()
SingleOrDefault()
Any()
All()
Count()
Sum()
Average()
Min()
Max()
GroupBy()
Join()
```

Example:

```csharp
var employees = await context.Employees
    .Where(e => e.Salary > 50000)
    .OrderByDescending(e => e.Salary)
    .Select(e => new
    {
        e.Id,
        e.Name,
        e.Salary
    })
    .ToListAsync();
```

Understand the SQL that EF Core approximately generates.

---

# 11. `IQueryable` vs `IEnumerable`

**Extremely important interview topic.**

Compare:

```csharp
context.Employees
    .Where(x => x.Salary > 50000)
    .ToList();
```

with:

```csharp
context.Employees
    .ToList()
    .Where(x => x.Salary > 50000);
```

First:

```text
Database
    ↓
WHERE Salary > 50000
    ↓
Only matching rows
    ↓
Application
```

Second:

```text
Database
    ↓
ALL rows
    ↓
Application
    ↓
Filter
```

This leads directly into:

> **Server-side vs client-side evaluation**

---

# 12. Query Execution

Learn the difference between:

```csharp
IQueryable<Employee>
```

and:

```csharp
List<Employee>
```

Understand **deferred execution**.

This:

```csharp
var query = context.Employees
    .Where(x => x.Salary > 50000);
```

doesn't necessarily execute SQL immediately.

Execution generally occurs when you materialize:

```csharp
await query.ToListAsync();
```

Other execution operators:

```csharp
FirstAsync()
SingleAsync()
AnyAsync()
CountAsync()
ToListAsync()
```

Mental model:

```text
LINQ expression
      ↓
Expression Tree
      ↓
EF Core query translator
      ↓
SQL
      ↓
Database
      ↓
Results
      ↓
Materialization
```

---

# 13. Projection — Very Important

Learn:

```csharp
.Select()
```

instead of blindly doing:

```csharp
.ToList()
```

Example:

```csharp
var employees = await context.Employees
    .Select(e => new EmployeeDto
    {
        Id = e.Id,
        Name = e.Name,
        Salary = e.Salary
    })
    .ToListAsync();
```

Understand why projection is useful:

```text
Entity
 ├── Id
 ├── Name
 ├── Salary
 ├── Address
 ├── Phone
 ├── ...
 └── 30 more columns
```

versus:

```text
DTO
 ├── Id
 └── Name
```

Projection reduces unnecessary data transfer and is especially important for APIs.

---

# 14. Relationships

This is a major section.

Learn:

### One-to-One

```text
User ───── Profile
```

### One-to-Many

```text
Customer
   │
   ├── Order
   ├── Order
   └── Order
```

### Many-to-Many

```text
Student
   ↕
Course
```

with join table:

```text
StudentCourse
```

Learn:

- Principal entity
    
- Dependent entity
    
- Foreign key
    
- Navigation property
    
- Optional relationship
    
- Required relationship
    
- Cascade delete
    
- Restrict delete
    
- No action
    
- Join entities
    

---

# 15. Navigation Properties

Example:

```csharp
public class Customer
{
    public int Id { get; set; }

    public ICollection<Order> Orders { get; set; }
        = new List<Order>();
}
```

and:

```csharp
public class Order
{
    public int Id { get; set; }

    public int CustomerId { get; set; }

    public Customer Customer { get; set; } = null!;
}
```

Understand the difference between:

```text
CustomerId
```

and:

```text
Customer
```

One is the **foreign key**.

The other is the **navigation property**.

---

# 16. Loading Related Data

Master these three concepts.

## Eager Loading

```csharp
var customers = await context.Customers
    .Include(c => c.Orders)
    .ToListAsync();
```

## Explicit Loading

```csharp
await context.Entry(customer)
    .Collection(c => c.Orders)
    .LoadAsync();
```

## Lazy Loading

Related data is loaded automatically when accessed, when lazy-loading is configured.

Understand the tradeoffs.

Especially:

> **Lazy loading can easily create N+1 query problems.**

---

# 17. N+1 Problem

This is an **industry + interview critical** topic.

Bad pattern:

```text
Get 100 customers
      ↓
1 query

For each customer:
      ↓
Get orders
      ↓
100 queries
```

Total:

```text
101 queries
```

Instead, use appropriate eager loading/projection/query shaping.

Learn to recognize N+1 from:

- SQL logs
    
- profiler
    
- application traces
    
- generated SQL
    

---

# 18. Change Tracking

Now go deep.

EF Core maintains a graph of tracked entities.

Example:

```csharp
var employee = await context.Employees
    .FirstAsync(x => x.Id == 1);

employee.Salary = 100000;

await context.SaveChangesAsync();
```

EF detects:

```text
Original Salary = 80000
Current Salary  = 100000
```

and generates an update.

Learn:

```csharp
context.Entry(entity).State
```

and:

```csharp
context.ChangeTracker
```

Understand:

```text
Added
Modified
Deleted
Unchanged
Detached
```

---

# 19. Tracking vs No Tracking

Learn:

```csharp
AsNoTracking()
```

Example:

```csharp
var employees = await context.Employees
    .AsNoTracking()
    .ToListAsync();
```

Understand why read-only queries often benefit from no tracking.

But don't memorize:

> "Always use AsNoTracking."

That's wrong.

Instead understand:

```text
Need to modify entity?
        │
       Yes → Tracking
        │
       No
        ↓
Read-only query → Consider NoTracking
```

Also learn:

```csharp
AsNoTrackingWithIdentityResolution()
```

at the advanced stage.

---

# 20. `SaveChanges()`

Understand exactly what happens.

```csharp
await context.SaveChangesAsync();
```

Conceptually:

```text
Detect changes
     ↓
Determine entity states
     ↓
Generate commands
     ↓
Execute SQL
     ↓
Database
     ↓
Update generated values
     ↓
Accept changes
```

Learn:

```csharp
SaveChanges()
SaveChangesAsync()
```

and:

```csharp
SaveChanges(bool acceptAllChangesOnSuccess)
```

---

# 21. Transactions

Now learn database transactions.

Example:

```csharp
await using var transaction =
    await context.Database.BeginTransactionAsync();

try
{
    // operation 1
    // operation 2

    await context.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

Understand:

```text
Transaction
 ├── Atomicity
 ├── Consistency
 ├── Isolation
 └── Durability
```

And:

- Commit
    
- Rollback
    
- Isolation levels
    
- Ambient transactions
    
- Multiple operations
    
- Transaction boundaries
    

---

# 22. Concurrency

Another major interview topic.

Learn:

### Optimistic concurrency

For example:

```text
User A reads row
User B reads row

User A updates
User B updates

What happens?
```

Learn concurrency tokens:

```csharp
[Timestamp]
public byte[] RowVersion { get; set; }
```

And:

```csharp
DbUpdateConcurrencyException
```

Understand how EF detects that the database row changed since it was read.

---

# 23. Migrations — Advanced Understanding

Don't just memorize:

```bash
dotnet ef migrations add
```

Understand migration architecture.

Learn:

```text
ModelSnapshot
Migration
Up()
Down()
```

Understand:

```text
Application model
      ↓
Model comparison
      ↓
Migration
      ↓
Schema change
```

Learn how to handle:

- Adding columns
    
- Removing columns
    
- Renaming columns
    
- Data migrations
    
- Breaking schema changes
    
- Production deployment
    
- Migration conflicts
    

---

# 24. Indexes

EF Core doesn't magically make your queries fast.

Understand database indexes.

Configure:

```csharp
modelBuilder.Entity<User>()
    .HasIndex(x => x.Email)
    .IsUnique();
```

Learn:

- Clustered indexes
    
- Non-clustered indexes
    
- Composite indexes
    
- Unique indexes
    
- Covering indexes
    
- Index selectivity
    
- Index tradeoffs
    

Most importantly:

> EF Core performance is ultimately constrained by database performance.

---

# 25. Query Performance

Now enter **industry-level EF Core**.

Learn:

### Avoid unnecessary data

Bad:

```csharp
await context.Users.ToListAsync();
```

if you only need names.

Better:

```csharp
await context.Users
    .Select(x => x.Name)
    .ToListAsync();
```

### Avoid unnecessary tracking

```csharp
AsNoTracking()
```

### Pagination

Learn:

```csharp
Skip()
Take()
```

but also understand why **keyset/seek pagination** can be preferable for large datasets.

### Avoid N+1

### Avoid premature materialization

Bad:

```csharp
var data = context.Users
    .ToList()
    .Where(...);
```

### Understand generated SQL

Use logging and query inspection.

---

# 26. `Include()` vs Projection

This is an important design decision.

You should understand when to use:

```csharp
.Include()
```

versus:

```csharp
.Select()
```

For API read models, projection is frequently preferable:

```csharp
var result = await context.Orders
    .Where(o => o.CustomerId == customerId)
    .Select(o => new OrderDto
    {
        Id = o.Id,
        Total = o.Total,
        CustomerName = o.Customer.Name
    })
    .ToListAsync();
```

This lets the database return exactly what you need.

---

# 27. Split Queries

Learn:

```csharp
AsSplitQuery()
```

and:

```csharp
AsSingleQuery()
```

Why?

Multiple collection includes can produce huge joins and **cartesian explosion**.

Understand:

```text
Single query
    ↓
Large JOIN
    ↓
Duplicated result rows
```

versus:

```text
Split query
    ↓
Multiple SQL queries
    ↓
EF combines results
```

Learn when each strategy makes sense.

---

# 28. Global Query Filters

Very important for certain architectures.

Example:

```csharp
modelBuilder.Entity<Product>()
    .HasQueryFilter(x => !x.IsDeleted);
```

Now:

```csharp
context.Products
```

automatically excludes deleted records.

Useful for:

- Soft deletes
    
- Multi-tenancy
    
- Tenant isolation
    

But understand their dangers, especially when debugging unexpected query results.

---

# 29. Shadow Properties

Learn properties that exist in the EF model but not necessarily on your CLR entity.

Example:

```csharp
modelBuilder.Entity<Order>()
    .Property<DateTime>("CreatedAt");
```

Access:

```csharp
EF.Property<DateTime>(order, "CreatedAt")
```

Useful for infrastructure concerns such as auditing.

---

# 30. Value Conversions

Learn:

```csharp
HasConversion()
```

Example:

```text
C#:
enum OrderStatus

Database:
string / int
```

EF can convert between CLR and database representations.

Useful for:

- Strongly typed values
    
- Enums
    
- Value objects
    
- Custom representations
    

---

# 31. Owned Types / Complex Types

Learn how to model value-like objects.

For example:

```csharp
public class Address
{
    public string City { get; set; }
    public string Country { get; set; }
}
```

Understand how modern EF Core models **complex/value-object-like structures** and how that differs from independent entities.

---

# 32. Raw SQL

You should know how to escape the abstraction when necessary.

Learn:

```csharp
FromSql()
```

and:

```csharp
Database.ExecuteSql(...)
```

But understand SQL injection.

Safe parameterization is critical.

Conceptually:

```text
LINQ
 ↓
EF Core
 ↓
SQL
```

but sometimes:

```text
Complex DB-specific query
 ↓
Raw SQL
 ↓
EF Core
```

Don't treat raw SQL as automatically bad. Treat **unnecessary raw SQL and unsafe string construction** as bad.

---

# 33. Stored Procedures

Understand EF Core's interaction with:

- Stored procedures
    
- Raw SQL
    
- Database functions
    
- Views
    
- Table-valued functions
    

You don't need to become a stored-procedure expert, but an enterprise developer should understand how EF Core fits into an existing database environment.

---

# 34. Database Functions

Learn how CLR methods can map to database functions.

Conceptually:

```text
C# expression
      ↓
EF translation
      ↓
SQL function
```

This becomes useful when working with advanced SQL/database capabilities.

---

# 35. Compiled Queries

Advanced performance topic.

Learn:

```csharp
EF.CompileQuery()
```

and async variants where applicable.

But understand the important interview answer:

> Compiled queries are an optimization for repeated query shapes; they are not a default requirement for every EF Core query.

---

# 36. Bulk Operations

Modern EF Core has set-based operations such as:

```csharp
ExecuteUpdateAsync()
ExecuteDeleteAsync()
```

Understand the difference between:

```csharp
foreach (var entity in entities)
{
    entity.IsActive = false;
}

await context.SaveChangesAsync();
```

and a set-based update.

The latter can avoid materializing and tracking every entity.

---

# 37. Interceptors

Advanced industry topic.

Learn:

```text
DbCommandInterceptor
SaveChangesInterceptor
ConnectionInterceptor
```

Potential uses:

- Auditing
    
- Diagnostics
    
- Query instrumentation
    
- Cross-cutting infrastructure
    
- Custom behavior
    

Don't abuse interceptors to hide business logic.

---

# 38. EF Core Diagnostics & Logging

You need to know how to answer:

> "EF Core query is slow. How do you investigate?"

Your process should look like:

```text
Slow API
   ↓
Application tracing
   ↓
EF Core logging
   ↓
Generated SQL
   ↓
Database execution plan
   ↓
Indexes / joins / cardinality
   ↓
Query optimization
```

Learn:

```csharp
LogTo()
EnableDetailedErrors()
EnableSensitiveDataLogging()
```

Understand why sensitive-data logging should be handled carefully in production.

---

# 39. Query Tags

Learn:

```csharp
.TagWith("Get active customers")
```

Useful when diagnosing SQL in database logs/profilers.

---

# 40. Connection Pooling vs DbContext Pooling

This is a good interview distinction.

Understand:

```text
Database connection pooling
```

versus:

```text
DbContext pooling
```

They are **not the same thing**.

Learn:

```csharp
AddDbContextPool()
```

and when pooling can be useful.

---

# 41. Repository Pattern

Now move from EF Core API knowledge to architecture.

Understand the debate:

```text
Controller
   ↓
Repository
   ↓
DbContext
```

versus:

```text
Controller
   ↓
Application Service
   ↓
DbContext
```

You should understand why many teams don't create generic repositories over EF Core.

EF Core already provides:

```text
Unit of Work
+
Repository-like abstraction through DbSet
```

A repository can still make sense when it encapsulates meaningful domain/application-specific persistence behavior.

---

# 42. Unit of Work

Understand why:

```csharp
DbContext
```

is commonly considered a Unit of Work.

Example:

```text
Operation
 ├── Create Order
 ├── Update Inventory
 ├── Create Payment record
 └── SaveChanges()
```

One unit of work can coordinate these changes.

---

# 43. EF Core in Clean Architecture

Learn a typical architecture:

```text
Presentation
     ↓
Application
     ↓
Domain
     ↑
Infrastructure
```

EF Core normally lives in:

```text
Infrastructure
```

For example:

```text
MyApp.Domain
MyApp.Application
MyApp.Infrastructure
MyApp.API
```

EF Core implementation:

```text
Infrastructure
 ├── AppDbContext
 ├── Configurations
 ├── Migrations
 ├── Repositories
 └── Persistence
```

---

# 44. [[Entity Configuration]] 

Instead of putting everything inside:

```csharp
OnModelCreating()
```

learn:

```csharp
IEntityTypeConfiguration<T>
```

Example:

```csharp
public class EmployeeConfiguration
    : IEntityTypeConfiguration<Employee>
{
    public void Configure(EntityTypeBuilder<Employee> builder)
    {
        builder.HasKey(x => x.Id);

        builder.Property(x => x.Name)
            .HasMaxLength(100)
            .IsRequired();
    }
}
```

Then:

```csharp
modelBuilder.ApplyConfigurationsFromAssembly(
    typeof(AppDbContext).Assembly);
```

This is much cleaner for larger systems.

---

# 45. DTOs vs Entities

Understand why you shouldn't blindly expose EF entities from APIs.

Bad architecture:

```text
EF Entity
    ↓
JSON API
```

Better:

```text
EF Entity
    ↓
Projection / Mapping
    ↓
DTO
    ↓
API
```

Understand:

- Entity
    
- DTO
    
- Request model
    
- Response model
    
- Domain model
    

They serve different purposes.

---

# 46. EF Core + ASP.NET Core

Now combine everything.

Build:

```text
ASP.NET Core Web API
        ↓
Service/Application Layer
        ↓
EF Core
        ↓
SQL Server
```

Implement:

```text
GET    /products
GET    /products/{id}
POST   /products
PUT    /products/{id}
DELETE /products/{id}
```

Then add:

- Validation
    
- DTOs
    
- Pagination
    
- Filtering
    
- Sorting
    
- Searching
    
- Transactions
    
- Error handling
    
- Logging
    
- Concurrency
    

---

# 47. Production-Grade API Querying

Learn to design:

```http
GET /products?page=2&pageSize=20
```

and:

```http
GET /products?category=electronics&sort=price
```

Understand how to safely construct dynamic queries.

Avoid:

```csharp
if (...)
    query = query.Where(...);

if (...)
    query = query.Where(...);
```

being treated as something magical. Learn that you're building an `IQueryable` expression pipeline.

---

# 48. Specification Pattern

Advanced architectural topic.

Understand how reusable query specifications can encapsulate:

```text
Filtering
Sorting
Includes
Projection
Pagination
```

But learn it conceptually before adopting a specification library.

---

# 49. Multi-Tenancy

Industry-level topic.

Understand approaches such as:

```text
TenantId column
      ↓
Global Query Filter
```

and:

```text
Tenant
 ↓
DbContext
 ↓
Queries
```

Learn the security implications.

**Do not rely on developers remembering `Where(x => x.TenantId == tenantId)` everywhere.**

---

# 50. Soft Delete

Typical design:

```text
IsDeleted
DeletedAt
DeletedBy
```

combined with:

```csharp
HasQueryFilter(...)
```

But understand:

- Restoring deleted records
    
- Admin queries
    
- Unique constraints
    
- Indexes
    
- Data retention
    
- Hard deletion
    
- Audit requirements
    

---

# 51. Auditing

Learn patterns for:

```text
CreatedAt
CreatedBy
ModifiedAt
ModifiedBy
```

Possible implementation:

```text
SaveChanges interceptor
        OR
SaveChanges override
        OR
Application-layer behavior
```

Understand the tradeoffs.

---

# 52. Transactions + External Systems

Very important production concept.

Imagine:

```text
Database update
      +
Publish message
      +
Send email
```

A DB transaction cannot automatically make all three atomic.

This leads into:

```text
Outbox Pattern
```

Learn:

```text
Business operation
      ↓
DB transaction
 ├── Business data
 └── Outbox message
      ↓
Commit
      ↓
Background publisher
      ↓
Message broker
```

This is beyond basic EF Core but is **excellent industry-level knowledge**.

---

# 53. Testing EF Core

Learn:

### Unit testing

Don't automatically use EF Core for every unit test.

### Integration testing

Use a real relational database where database behavior matters.

Understand the limitations of:

```text
EF Core InMemory provider
```

It does **not behave like a relational production database**.

For serious relational behavior, consider:

```text
SQLite
Testcontainers
Dedicated test database
```

depending on the test requirements.

---

# 54. EF Core Testing Strategy

A mature strategy:

```text
Pure business logic
       ↓
Unit tests

EF Core + SQL behavior
       ↓
Integration tests

API + DB + infrastructure
       ↓
End-to-end tests
```

Don't try to test everything through mocked `DbSet`s.

---

# 55. Database-First

After Code First, learn Database First.

Understand:

```text
Existing DB
   ↓
EF Core scaffolding
   ↓
Entities
   +
DbContext
```

Learn when organizations use:

- Code First
    
- Database First
    
- Hybrid approaches
    

Enterprise applications often have legacy databases, so this knowledge matters.

---

# 56. EF Core Providers

Understand that EF Core itself isn't the database.

Architecture:

```text
EF Core
   ↓
Provider
   ↓
Database
```

Examples:

```text
SQL Server
PostgreSQL
SQLite
MySQL
Oracle
```

Different providers have different capabilities and SQL translation behavior.

---

# 57. SQL Translation Limitations

This is a major real-world skill.

Not every C# expression can be translated to SQL.

For example:

```csharp
.Where(x => SomeCustomCSharpMethod(x.Name))
```

may not translate.

You need to know:

```text
Can EF translate this?
        │
        ├── Yes → SQL
        │
        └── No → redesign / explicit client-side work
```

Never assume:

> "If LINQ compiles, EF Core will execute it efficiently in SQL."

---

# 58. Client vs Server Evaluation

Understand exactly where computation occurs.

You want:

```text
Database
 ├── Filter
 ├── Join
 ├── Aggregate
 └── Projection
        ↓
Application
```

rather than:

```text
Database
    ↓
Millions of rows
    ↓
Application
    ↓
Filter
    ↓
Projection
```

---

# 59. Cartesian Explosion

Understand why:

```csharp
.Include(x => x.Orders)
.Include(x => x.Payments)
.Include(x => x.Addresses)
```

can become expensive.

Conceptually:

```text
Customer
 × Orders
 × Payments
 × Addresses
```

can produce many duplicated rows.

Learn:

```text
Projection
Split queries
Better query shape
```

---

# 60. Pagination Deep Dive

Understand the difference between:

### Offset pagination

```csharp
.Skip(10000)
.Take(20)
```

and:

### Keyset pagination

```text
WHERE Id > lastSeenId
ORDER BY Id
LIMIT 20
```

For large datasets, keyset pagination can be dramatically better.

This is an excellent **industry interview topic**.

---

# 61. EF Core Performance Checklist

When reviewing a query, ask:

```text
1. Is it executing on the DB?
2. Am I retrieving unnecessary columns?
3. Am I tracking unnecessarily?
4. Is there an N+1?
5. Are joins exploding the result?
6. Are indexes appropriate?
7. Is pagination correct?
8. Is the SQL efficient?
9. Is the database plan efficient?
10. Am I making too many round trips?
```

This mindset is more valuable than memorizing 100 EF methods.

---

# 62. Advanced Query Topics

Eventually learn:

- Expression trees
    
- Dynamic LINQ
    
- Query compilation
    
- Query caching
    
- Global filters
    
- Temporal tables
    
- JSON columns
    
- Database-specific features
    
- Spatial data
    
- Value converters
    
- Complex types
    
- Interceptors
    
- Diagnostics
    
- Compiled queries
    
- Bulk/set-based operations
    

---

# 63. EF Core + Domain-Driven Design

For senior-level interviews, understand:

```text
Entity
Value Object
Aggregate
Aggregate Root
Domain Event
Repository
Unit of Work
```

Then understand how EF Core persistence maps onto these concepts.

Especially:

> Don't let your database model automatically dictate your domain model.

---

# 64. EF Core + CQRS

Understand:

```text
Command
   ↓
Write model
   ↓
DbContext
```

and:

```text
Query
   ↓
Projection
   ↓
DTO
```

For read-heavy systems, you may use EF Core primarily for projections rather than materializing rich domain entities.

---

# 65. EF Core + Microservices

Understand:

```text
Service A
   ↓
Database A

Service B
   ↓
Database B
```

and the principle:

> Each service should own its persistence boundary.

Learn:

- Transaction boundaries
    
- Outbox pattern
    
- Eventual consistency
    
- Integration events
    
- Idempotency
    
- Read models
    

---

# 66. Production Database Migrations

This deserves special attention.

Development:

```bash
dotnet ef database update
```

Production is more complicated.

Learn:

```text
Migration generation
        ↓
Review SQL
        ↓
Deployment pipeline
        ↓
Database migration
        ↓
Application deployment
```

Understand:

- Backward-compatible migrations
    
- Expand/contract pattern
    
- Zero-downtime deployment
    
- Data migrations
    
- Large-table migrations
    
- Rollback limitations
    

---

# 67. Industry-Level EF Core Project

After learning the concepts, build **one serious project**.

I recommend:

# E-Commerce Backend

```text
Users
Products
Categories
Orders
OrderItems
Payments
Inventory
Reviews
Coupons
Addresses
```

Relationships:

```text
User
 │
 ├── Orders
 ├── Addresses
 └── Reviews

Product
 │
 ├── Category
 ├── Reviews
 └── Inventory

Order
 │
 ├── OrderItems
 ├── Payment
 └── Address
```

Implement:

```text
EF Core
SQL Server
ASP.NET Core Web API
Clean Architecture
JWT Authentication
DTOs
Fluent API
Migrations
Transactions
Concurrency
Pagination
Filtering
Sorting
Projection
Global Query Filters
Auditing
Soft Delete
Logging
Integration Tests
```

Then add:

```text
Outbox Pattern
Background Worker
Caching
CQRS-style queries
```

At this point you'll have something that resembles real enterprise work.

---

# 68. Your Learning Sequence

Don't study the topics randomly.

Follow this exact progression:

```text
PHASE 1 — FOUNDATION
│
├── C# LINQ
├── IQueryable
├── SQL
├── Relational DB
└── ORM concepts
        ↓
PHASE 2 — EF CORE BASICS
│
├── DbContext
├── DbSet
├── Entities
├── DI
├── Configuration
├── CRUD
└── Migrations
        ↓
PHASE 3 — QUERYING
│
├── LINQ
├── Projection
├── Filtering
├── Sorting
├── Pagination
├── IQueryable
└── SQL translation
        ↓
PHASE 4 — RELATIONSHIPS
│
├── 1:1
├── 1:N
├── N:N
├── Navigation properties
├── Include
├── Explicit loading
└── Lazy loading
        ↓
PHASE 5 — CHANGE TRACKING
│
├── Entity states
├── ChangeTracker
├── SaveChanges
├── Tracking
└── NoTracking
        ↓
PHASE 6 — DATABASE ENGINEERING
│
├── Transactions
├── Concurrency
├── Indexes
├── Isolation
├── Constraints
└── Migrations
        ↓
PHASE 7 — PERFORMANCE
│
├── N+1
├── Projection
├── Split queries
├── Keyset pagination
├── Query plans
├── SQL optimization
└── Set-based operations
        ↓
PHASE 8 — ARCHITECTURE
│
├── Clean Architecture
├── Repository
├── Unit of Work
├── DTOs
├── CQRS
├── DDD
└── Specifications
        ↓
PHASE 9 — PRODUCTION
│
├── Auditing
├── Soft Delete
├── Multi-tenancy
├── Interceptors
├── Diagnostics
├── Testing
├── Deployment
└── Outbox
        ↓
PHASE 10 — ADVANCED
│
├── Compiled queries
├── Value conversions
├── Complex types
├── Temporal tables
├── JSON
├── Provider-specific features
└── Advanced SQL integration
        ↓
PHASE 11 — INTERVIEW
│
├── Conceptual questions
├── SQL questions
├── EF Core questions
├── Performance scenarios
├── Architecture scenarios
└── Coding problems
```

---

# 69. Interview Preparation Map

Once you've completed the roadmap, you should be able to answer these without memorizing definitions.

### Beginner

- What is EF Core?
    
- What is ORM?
    
- What is `DbContext`?
    
- What is `DbSet`?
    
- What is Code First?
    
- What are migrations?
    
- What is `SaveChanges()`?
    
- What is `Include()`?
    

### Intermediate

- `IQueryable` vs `IEnumerable`?
    
- Tracking vs `AsNoTracking()`?
    
- `First()` vs `FirstOrDefault()`?
    
- `Single()` vs `SingleOrDefault()`?
    
- Eager vs lazy vs explicit loading?
    
- What is change tracking?
    
- What are shadow properties?
    
- What is Fluent API?
    
- What is a navigation property?
    
- How does EF Core detect changes?
    

### Advanced

- How does LINQ become SQL?
    
- What causes N+1?
    
- How do you optimize EF Core queries?
    
- `Include()` vs projection?
    
- Single query vs split query?
    
- What is optimistic concurrency?
    
- How do transactions work?
    
- How do indexes affect EF queries?
    
- What is keyset pagination?
    
- What are compiled queries?
    
- What are interceptors?
    
- How do you handle bulk updates?
    
- How do you diagnose slow EF Core queries?
    

### Senior

- Should you use Repository over EF Core?
    
- How would you structure EF Core in Clean Architecture?
    
- How would you implement multi-tenancy?
    
- How would you implement soft delete?
    
- How would you handle auditing?
    
- How would you deploy migrations safely?
    
- How would you design persistence for microservices?
    
- How would you solve DB + message broker consistency?
    
- How would you test EF Core?
    
- How would you handle a 10-million-row table?
    
- How would you diagnose a production API that suddenly became slow?
    

---

# 70. The Most Important Mental Models

If you remember only a few things, remember these.

### Mental Model 1 — EF Core is not SQL's replacement

```text
C# / LINQ
    ↓
EF Core
    ↓
SQL
    ↓
Database
```

You still need to understand SQL.

---

### Mental Model 2 — `IQueryable` is a query definition

```csharp
var query = context.Users
    .Where(x => x.IsActive);
```

Think:

> "I'm constructing a query."

Not:

> "I already have the users."

---

### Mental Model 3 — `ToListAsync()` is a boundary

```csharp
var query = context.Users
    .Where(...);

var users = await query.ToListAsync();
```

Think:

```text
IQueryable
   │
   │ SQL translation
   ↓
DATABASE
   │
   ↓
Objects
```

---

### Mental Model 4 — `DbContext` tracks a unit of work

```text
Load
 ↓
Modify
 ↓
Track changes
 ↓
SaveChanges
 ↓
Commit
```

---

### Mental Model 5 — Performance is about SQL

When EF Core is slow, don't blindly optimize C#.

Inspect:

```text
LINQ
 ↓
Generated SQL
 ↓
Execution Plan
 ↓
Indexes
 ↓
Database
```

---

# 71. How I Recommend You Actually Study

Use a **30% theory / 70% coding** approach.

For every topic:

```text
1. Learn concept
       ↓
2. Write tiny example
       ↓
3. Inspect generated SQL
       ↓
4. Change the code
       ↓
5. Observe SQL difference
       ↓
6. Build a real feature
       ↓
7. Solve interview questions
```

For example, don't merely learn `Include()`.

Do this:

```csharp
.Include(x => x.Orders)
```

Then inspect the SQL.

Then replace it with:

```csharp
.Select(...)
```

Inspect SQL again.

Then try:

```csharp
.AsSplitQuery()
```

Inspect SQL again.

Now you **understand** the feature instead of memorizing it.

---

# 72. Your Final Skill Target

By the end, you should be able to look at this:

```csharp
var orders = await db.Orders
    .Include(x => x.Customer)
    .Include(x => x.Items)
        .ThenInclude(x => x.Product)
    .Where(x => x.CustomerId == customerId)
    .OrderByDescending(x => x.CreatedAt)
    .Skip(page * size)
    .Take(size)
    .ToListAsync();
```

and immediately reason about:

```text
What SQL is generated?
        ↓
How many tables are joined?
        ↓
How many columns are returned?
        ↓
Is tracking necessary?
        ↓
Could projection be better?
        ↓
Could this create row explosion?
        ↓
Are indexes available?
        ↓
Is offset pagination appropriate?
        ↓
Could there be an N+1 elsewhere?
        ↓
Is this query executed server-side?
        ↓
What does the execution plan look like?
```

**That is the difference between "I know EF Core" and "I can work with EF Core professionally."**

---

## Recommended mastery order

If you're starting from zero, I would divide your learning into **12 stages**:

|Stage|Focus|Target|
|---|---|---|
|1|SQL + LINQ|Foundation|
|2|ORM + EF Core architecture|Beginner|
|3|DbContext + CRUD + DI|Beginner|
|4|Relationships + loading|Intermediate|
|5|Querying + projection|Intermediate|
|6|Change tracking + SaveChanges|Intermediate|
|7|Migrations + transactions + concurrency|Intermediate/Advanced|
|8|Performance + SQL translation|Advanced|
|9|Architecture + Clean Architecture|Advanced|
|10|Testing + production patterns|Advanced|
|11|DDD/CQRS/Microservices integration|Senior|
|12|Interview scenarios + system design|Interview-ready|

**Don't jump to Stage 8 before you genuinely understand Stages 1–6.** Performance problems become much easier to reason about once you understand what EF Core is actually doing underneath.

If you want to learn this with me interactively, the best next step is **Stage 1: SQL + LINQ + `IQueryable`**, followed by building a small EF Core project from scratch and progressively turning it into the production-grade e-commerce API above.