

Absolutely. I’ll explain this as an **EF Core interview + practical development guide**, starting from the mental model and going all the way through `IEntityTypeConfiguration<T>`, Fluent API, keys, relationships, constraints, table/column mapping, assemblies, `DbContext`, migrations, and commonly asked interview questions.

# EF Core Entity Configuration — Complete Guide

## 1. First: What is Entity Configuration?

In **Entity Framework Core**, entity configuration is the process of telling EF Core:

> **“How should this C# class be represented in the database?”**

Suppose you have:

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}
```

EF Core can conventionally create something like:

```text
Students
--------------------------------
Id       int       PK
Name     nvarchar
Email    nvarchar
```

But what if you want:

- Table name = `tbl_students`
    
- Primary key = `StudentId`
    
- `Email` must be unique
    
- `Name` is required
    
- `Name` max length = 100
    
- Composite key
    
- Foreign key
    
- Index
    
- Decimal precision
    
- Column name different from property
    
- Relationship behavior
    
- Database constraints
    

This is where **configuration** comes in.

There are three major ways to configure EF Core:

### 1. Conventions

EF Core automatically infers mappings.

```csharp
public int Id { get; set; }
```

EF Core recognizes `Id` as the primary key.

---

### 2. Data Annotations

Attributes on your entity:

```csharp
public class Student
{
    [Key]
    public int StudentId { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; }
}
```

---

### 3. Fluent API

Configuration inside `OnModelCreating` or separate configuration classes:

```csharp
modelBuilder.Entity<Student>()
    .HasKey(x => x.StudentId);

modelBuilder.Entity<Student>()
    .Property(x => x.Name)
    .IsRequired()
    .HasMaxLength(100);
```

For serious applications, **Fluent API + separate configuration classes** is generally the cleanest approach.

---

# 2. What is `IEntityTypeConfiguration<T>`?

This is the most important concept.

Suppose you have:

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}
```

Instead of putting everything into `DbContext`:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .HasKey(x => x.Id);

    modelBuilder.Entity<Student>()
        .Property(x => x.Name)
        .IsRequired()
        .HasMaxLength(100);
}
```

you can create:

```csharp
public class StudentConfiguration 
    : IEntityTypeConfiguration<Student>
{
    public void Configure(EntityTypeBuilder<Student> builder)
    {
        builder.HasKey(x => x.Id);

        builder.Property(x => x.Name)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(x => x.Email)
            .IsRequired();
    }
}
```

Now the configuration for `Student` is isolated in one class.

---

# 3. Why use `IEntityTypeConfiguration<T>`?

Imagine your project has:

```text
Student
Teacher
Course
Department
Address
Order
OrderItem
Product
Customer
Payment
```

If you put all configuration into:

```csharp
DbContext.OnModelCreating()
```

you might end up with:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Student configuration

    // Teacher configuration

    // Course configuration

    // Department configuration

    // Address configuration

    // Order configuration

    // OrderItem configuration

    // Product configuration

    // Customer configuration

    // Payment configuration
}
```

That becomes difficult to maintain.

Instead:

```text
Configurations/
    StudentConfiguration.cs
    TeacherConfiguration.cs
    CourseConfiguration.cs
    DepartmentConfiguration.cs
    AddressConfiguration.cs
    OrderConfiguration.cs
```

Each entity owns its mapping.

---

# 4. Basic Configuration Structure

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

public class StudentConfiguration 
    : IEntityTypeConfiguration<Student>
{
    public void Configure(EntityTypeBuilder<Student> builder)
    {
        builder.ToTable("Students");

        builder.HasKey(x => x.Id);

        builder.Property(x => x.Name)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(x => x.Email)
            .IsRequired()
            .HasMaxLength(200);
    }
}
```

The important pieces are:

```text
IEntityTypeConfiguration<Student>
            ↓
Configure()
            ↓
EntityTypeBuilder<Student>
            ↓
Fluent API
```

---

# 5. Registering Configuration in `DbContext`

This is critical.

You have:

```csharp
public class StudentConfiguration 
    : IEntityTypeConfiguration<Student>
{
    public void Configure(EntityTypeBuilder<Student> builder)
    {
        builder.HasKey(x => x.Id);
    }
}
```

EF Core needs to know about this configuration.

The manual approach:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfiguration(new StudentConfiguration());
}
```

This works.

But imagine 50 entities.

You don't want:

```csharp
modelBuilder.ApplyConfiguration(new StudentConfiguration());
modelBuilder.ApplyConfiguration(new TeacherConfiguration());
modelBuilder.ApplyConfiguration(new CourseConfiguration());
modelBuilder.ApplyConfiguration(new OrderConfiguration());
...
```

Instead, use assembly scanning.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfigurationsFromAssembly(
        typeof(StudentConfiguration).Assembly);
}
```

This tells EF Core:

> Find all classes in this assembly implementing `IEntityTypeConfiguration<T>` and apply them.

This is extremely common in production applications.

---

# 6. What Does `ApplyConfigurationsFromAssembly` Actually Do?

Suppose your assembly contains:

```text
StudentConfiguration
TeacherConfiguration
CourseConfiguration
OrderConfiguration
ProductConfiguration
```

and each implements:

```csharp
IEntityTypeConfiguration<T>
```

Then:

```csharp
modelBuilder.ApplyConfigurationsFromAssembly(
    typeof(StudentConfiguration).Assembly);
```

automatically discovers and applies them.

A common pattern is:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    modelBuilder.ApplyConfigurationsFromAssembly(
        typeof(ApplicationDbContext).Assembly);
}
```

or:

```csharp
modelBuilder.ApplyConfigurationsFromAssembly(
    typeof(StudentConfiguration).Assembly);
```

### Interview question

**Q: How do you automatically register all EF Core entity configurations?**

Answer:

```csharp
modelBuilder.ApplyConfigurationsFromAssembly(
    typeof(SomeConfiguration).Assembly);
```

---

# 7. Entity Configuration vs DbContext

Think about responsibilities like this:

```text
DbContext
   |
   +---- database connection
   |
   +---- DbSets
   |
   +---- model building
   |
   +---- change tracking
   |
   +---- querying
   |
   +---- SaveChanges
```

While:

```text
IEntityTypeConfiguration<T>
   |
   +---- table mapping
   +---- key configuration
   +---- property configuration
   +---- relationship configuration
   +---- indexes
   +---- constraints
```

---

# 8. `ToTable()` — Change Table Name

Suppose:

```csharp
public class Student
{
    public int Id { get; set; }
}
```

By convention EF might create:

```text
Students
```

You can explicitly specify:

```csharp
builder.ToTable("tbl_students");
```

Database:

```text
tbl_students
```

Complete example:

```csharp
public void Configure(EntityTypeBuilder<Student> builder)
{
    builder.ToTable("tbl_students");
}
```

---

# 9. Schema

You can also specify schema:

```csharp
builder.ToTable("Students", "school");
```

Database:

```text
school.Students
```

This is useful when separating database domains.

Example:

```text
sales.Orders
school.Students
hr.Employees
```

---

# 10. Change Column Name

Entity:

```csharp
public class Student
{
    public int Id { get; set; }

    public string Name { get; set; }
}
```

Configuration:

```csharp
builder.Property(x => x.Name)
    .HasColumnName("student_name");
```

Database:

```text
student_name
```

---

# 11. Column Type

You can explicitly specify the database type:

```csharp
builder.Property(x => x.Name)
    .HasColumnType("varchar(100)");
```

Or:

```csharp
builder.Property(x => x.Price)
    .HasColumnType("decimal(18,2)");
```

However, for decimal values, prefer:

```csharp
builder.Property(x => x.Price)
    .HasPrecision(18, 2);
```

where supported by your EF Core/provider combination.

---

# 12. Required Properties

```csharp
builder.Property(x => x.Name)
    .IsRequired();
```

This generally results in:

```sql
Name NVARCHAR(...) NOT NULL
```

For nullable:

```csharp
builder.Property(x => x.Name)
    .IsRequired(false);
```

Database:

```sql
Name NVARCHAR(...) NULL
```

With modern C# nullable reference types enabled, EF Core can also infer requiredness from CLR nullability.

---

# 13. Maximum Length

```csharp
builder.Property(x => x.Name)
    .HasMaxLength(100);
```

This tells EF:

```text
Name <= 100 characters
```

Depending on provider/type conventions, this can influence generated column types.

---

# 14. Minimum Length

A common misconception:

```csharp
.HasMaxLength(100)
```

does **not** create a minimum-length database constraint.

If you need something like:

```text
Name must contain at least 3 characters
```

you can use a database check constraint:

```csharp
builder.ToTable("Students", table =>
{
    table.HasCheckConstraint(
        "CK_Student_Name_Length",
        "LEN(Name) >= 3");
});
```

The exact SQL expression is **database-provider specific**.

This distinction is important in interviews:

> `HasMaxLength()` is model metadata / column sizing. It is not the same thing as a database `CHECK` constraint.

---

# 15. Primary Key

Simple primary key:

```csharp
builder.HasKey(x => x.Id);
```

Equivalent conceptual SQL:

```sql
PRIMARY KEY (Id)
```

---

# 16. Changing the Primary Key

Suppose:

```csharp
public class Student
{
    public int StudentId { get; set; }
    public string Name { get; set; }
}
```

Configure:

```csharp
builder.HasKey(x => x.StudentId);
```

Now:

```text
StudentId = PK
```

---

# 17. Composite Primary Key

This is very important for interviews.

Suppose:

```csharp
public class StudentCourse
{
    public int StudentId { get; set; }
    public int CourseId { get; set; }

    public DateTime EnrolledAt { get; set; }
}
```

You want:

```text
(StudentId, CourseId)
```

as the primary key.

Use:

```csharp
builder.HasKey(x => new
{
    x.StudentId,
    x.CourseId
});
```

Generated conceptually:

```sql
PRIMARY KEY (StudentId, CourseId)
```

### Interview question

**Q: How do you configure a composite primary key in EF Core?**

Answer:

```csharp
builder.HasKey(x => new
{
    x.StudentId,
    x.CourseId
});
```

---

# 18. Composite Key Order Matters

This:

```csharp
builder.HasKey(x => new
{
    x.StudentId,
    x.CourseId
});
```

is different from:

```csharp
builder.HasKey(x => new
{
    x.CourseId,
    x.StudentId
});
```

The database primary-key/index column ordering can matter for index usage.

Conceptually:

```text
(StudentId, CourseId)
```

is optimized differently from:

```text
(CourseId, StudentId)
```

This matters particularly when the composite key is also used for lookup patterns.

---

# 19. Alternate Key

Don't confuse:

```csharp
HasKey()
```

with:

```csharp
HasAlternateKey()
```

Primary key:

```csharp
builder.HasKey(x => x.Id);
```

Alternate key:

```csharp
builder.HasAlternateKey(x => x.Email);
```

An alternate key represents another unique key that can participate in relationships.

For simply enforcing uniqueness on a property, you often want a **unique index** instead:

```csharp
builder.HasIndex(x => x.Email)
    .IsUnique();
```

### Difference

```text
Primary Key
    ↓
Identity of entity

Alternate Key
    ↓
Candidate key usable for relationships

Unique Index
    ↓
Uniqueness/performance constraint
```

---

# 20. Database Generated Values

Suppose:

```csharp
public int Id { get; set; }
```

You want SQL Server to generate the identity value.

Usually EF Core conventions handle this automatically.

You can explicitly configure:

```csharp
builder.Property(x => x.Id)
    .ValueGeneratedOnAdd();
```

For a non-generated property:

```csharp
builder.Property(x => x.Id)
    .ValueGeneratedNever();
```

For values generated when inserting or updating:

```csharp
builder.Property(x => x.UpdatedAt)
    .ValueGeneratedOnAddOrUpdate();
```

Be careful: actual database behavior depends on provider/database configuration.

---

# 21. Identity / Auto Increment

For SQL Server:

```csharp
builder.Property(x => x.Id)
    .UseIdentityColumn();
```

This is provider-specific.

For portable EF Core code, don't unnecessarily use provider-specific configuration unless you actually need it.

---

# 22. Default Value

Suppose:

```csharp
public bool IsActive { get; set; }
```

You want database default:

```text
true
```

You can configure:

```csharp
builder.Property(x => x.IsActive)
    .HasDefaultValue(true);
```

Conceptually:

```sql
DEFAULT 1
```

---

# 23. SQL Default Value

For database-generated values:

```csharp
builder.Property(x => x.CreatedAt)
    .HasDefaultValueSql("GETUTCDATE()");
```

SQL Server example.

PostgreSQL would use a different expression, for example:

```sql
CURRENT_TIMESTAMP
```

Therefore:

> `HasDefaultValueSql()` is database-provider specific.

---

# 24. Indexes

Create an index:

```csharp
builder.HasIndex(x => x.Email);
```

Unique index:

```csharp
builder.HasIndex(x => x.Email)
    .IsUnique();
```

This gives you:

```text
Email
 ↓
UNIQUE INDEX
```

---

# 25. Composite Index

Suppose:

```text
FirstName
LastName
```

need a composite index.

```csharp
builder.HasIndex(x => new
{
    x.FirstName,
    x.LastName
});
```

Database concept:

```sql
INDEX (FirstName, LastName)
```

Again, **order matters**.

---

# 26. Named Index

You can name it:

```csharp
builder.HasIndex(x => x.Email)
    .HasDatabaseName("IX_Students_Email");
```

---

# 27. Unique Constraint vs Unique Index

This is a subtle interview topic.

You can enforce uniqueness using:

```csharp
builder.HasIndex(x => x.Email)
    .IsUnique();
```

A unique index is primarily an index structure with uniqueness semantics.

A unique constraint is a database constraint.

In EF Core, the exact modeling capabilities and migration output depend on the database provider. For most application-level "email must be unique" requirements, a **unique index** is the common approach.

---

# 28. Check Constraints

Suppose:

```csharp
public decimal Salary { get; set; }
```

You want:

```text
Salary >= 0
```

You can configure:

```csharp
builder.ToTable("Employees", table =>
{
    table.HasCheckConstraint(
        "CK_Employee_Salary",
        "Salary >= 0");
});
```

Database:

```sql
CHECK (Salary >= 0)
```

Another example:

```csharp
table.HasCheckConstraint(
    "CK_Student_Age",
    "Age >= 18 AND Age <= 100");
```

### Important

The SQL expression:

```csharp
"Age >= 18"
```

is database-specific.

Don't assume the same SQL expression works across SQL Server, PostgreSQL, MySQL, SQLite, etc.

---

# 29. Foreign Keys

Suppose:

```csharp
public class Student
{
    public int Id { get; set; }

    public int DepartmentId { get; set; }

    public Department Department { get; set; }
}
```

Configure:

```csharp
builder.HasOne(x => x.Department)
    .WithMany(x => x.Students)
    .HasForeignKey(x => x.DepartmentId);
```

Read it from left to right:

```text
Student
   |
   | HasOne
   ↓
Department
   |
   | WithMany
   ↓
Students
```

---

# 30. One-to-Many Relationship

Typical:

```text
Department
     |
     | 1
     |
     | *
     ↓
Student
```

Configuration:

```csharp
builder.HasOne(x => x.Department)
    .WithMany(x => x.Students)
    .HasForeignKey(x => x.DepartmentId);
```

---

# 31. Required Foreign Key

```csharp
builder.HasOne(x => x.Department)
    .WithMany(x => x.Students)
    .HasForeignKey(x => x.DepartmentId)
    .IsRequired();
```

If:

```csharp
public int DepartmentId { get; set; }
```

it's naturally non-nullable.

---

# 32. Optional Foreign Key

Use nullable FK:

```csharp
public int? DepartmentId { get; set; }
```

Then:

```csharp
builder.HasOne(x => x.Department)
    .WithMany(x => x.Students)
    .HasForeignKey(x => x.DepartmentId)
    .IsRequired(false);
```

---

# 33. Cascade Delete

Suppose:

```text
Department
   |
   +--- Student
   +--- Student
   +--- Student
```

You delete the department.

Should students also be deleted?

Configure:

```csharp
builder.HasOne(x => x.Department)
    .WithMany(x => x.Students)
    .HasForeignKey(x => x.DepartmentId)
    .OnDelete(DeleteBehavior.Cascade);
```

---

# 34. Restrict Delete

Prevent deleting the parent when dependents exist:

```csharp
.OnDelete(DeleteBehavior.Restrict);
```

Depending on the provider, generated database behavior can vary.

Common delete behaviors include:

```csharp
DeleteBehavior.Cascade
DeleteBehavior.Restrict
DeleteBehavior.NoAction
DeleteBehavior.SetNull
DeleteBehavior.ClientCascade
DeleteBehavior.ClientSetNull
```

---

# 35. One-to-One Relationship

Suppose:

```csharp
public class Student
{
    public int Id { get; set; }

    public StudentProfile Profile { get; set; }
}
```

```csharp
public class StudentProfile
{
    public int Id { get; set; }

    public int StudentId { get; set; }

    public Student Student { get; set; }
}
```

Configuration:

```csharp
builder.HasOne(x => x.Profile)
    .WithOne(x => x.Student)
    .HasForeignKey<StudentProfile>(x => x.StudentId);
```

---

# 36. Many-to-Many

Suppose:

```text
Student * -------- * Course
```

Modern EF Core can model this without explicitly defining a join entity:

```csharp
modelBuilder.Entity<Student>()
    .HasMany(x => x.Courses)
    .WithMany(x => x.Students);
```

But if you need additional fields:

```text
StudentCourse
----------------
StudentId
CourseId
EnrolledAt
Grade
```

then explicitly model the join entity.

```csharp
builder.HasKey(x => new
{
    x.StudentId,
    x.CourseId
});
```

This is one reason composite keys are commonly encountered in EF Core.

---

# 37. Navigation Properties

Example:

```csharp
public class Student
{
    public int Id { get; set; }

    public int DepartmentId { get; set; }

    public Department Department { get; set; }
}
```

Here:

```csharp
DepartmentId
```

is the FK.

```csharp
Department
```

is the navigation property.

EF Core uses both to understand the relationship.

---

# 38. Shadow Properties

This is an important EF Core feature.

Suppose your C# class doesn't contain:

```csharp
CreatedAt
```

but you want it stored in the database.

You can define:

```csharp
builder.Property<DateTime>("CreatedAt");
```

This creates a **shadow property**.

EF Core tracks it internally, even though it doesn't exist in the CLR class.

You can access it through:

```csharp
context.Entry(entity)
    .Property("CreatedAt")
    .CurrentValue;
```

---

# 39. Global Query Filters

Extremely useful for soft deletes.

Entity:

```csharp
public class Student
{
    public int Id { get; set; }
    public bool IsDeleted { get; set; }
}
```

Configuration:

```csharp
builder.HasQueryFilter(x => !x.IsDeleted);
```

Now:

```csharp
context.Students.ToList();
```

automatically behaves approximately like:

```sql
WHERE IsDeleted = 0
```

This is called a **global query filter**.

---

# 40. Ignoring a Property

Suppose:

```csharp
public string FullName => FirstName + " " + LastName;
```

You don't want this property mapped.

```csharp
builder.Ignore(x => x.FullName);
```

---

# 41. Ignoring an Entity

Inside model configuration:

```csharp
modelBuilder.Ignore<SomeEntity>();
```

EF won't include it in the model.

---

# 42. Owned Types

Suppose:

```csharp
public class Student
{
    public int Id { get; set; }

    public Address Address { get; set; }
}
```

and:

```csharp
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
}
```

You may want `Address` to be part of `Student`, rather than an independent entity.

You can configure:

```csharp
builder.OwnsOne(x => x.Address);
```

This models `Address` as an **owned type**.

Modern EF Core also has broader complex-type support depending on the EF Core version and scenario, so distinguish **owned entity types** from **complex types** when discussing newer EF Core.

---

# 43. Value Conversion

Suppose your domain model uses:

```csharp
public enum StudentStatus
{
    Active,
    Inactive
}
```

You can convert it to a string:

```csharp
builder.Property(x => x.Status)
    .HasConversion<string>();
```

Database might store:

```text
Active
Inactive
```

instead of:

```text
0
1
```

Another common example is strongly typed IDs or custom value objects.

---

# 44. Enum Conversion

```csharp
builder.Property(x => x.Status)
    .HasConversion<string>()
    .HasMaxLength(20);
```

---

# 45. Decimal Precision

Very common interview question.

```csharp
builder.Property(x => x.Price)
    .HasPrecision(18, 2);
```

Means approximately:

```text
18 total digits
2 digits after decimal
```

Examples:

```text
1234567890123456.78
```

depending on database/provider limits.

---

# 46. String Column Configuration

For example:

```csharp
builder.Property(x => x.Name)
    .HasMaxLength(100)
    .IsUnicode(true);
```

Or:

```csharp
builder.Property(x => x.Code)
    .HasMaxLength(20)
    .IsUnicode(false);
```

For SQL Server this can influence `nvarchar` vs `varchar`.

---

# 47. Concurrency

Optimistic concurrency is another major EF Core interview topic.

For SQL Server, a common pattern is:

```csharp
public byte[] RowVersion { get; set; }
```

Configuration:

```csharp
builder.Property(x => x.RowVersion)
    .IsRowVersion();
```

EF uses this to detect whether another transaction modified the row.

Conceptually:

```text
User A reads row
       |
User B reads row
       |
User B updates row
       |
User A tries update
       ↓
Concurrency conflict
```

EF can throw:

```csharp
DbUpdateConcurrencyException
```

---

# 48. Complete Entity Configuration Example

Let's build a realistic example.

### Entity

```csharp
public class Student
{
    public int Id { get; set; }

    public string Name { get; set; }

    public string Email { get; set; }

    public int Age { get; set; }

    public decimal Fees { get; set; }

    public bool IsActive { get; set; }

    public bool IsDeleted { get; set; }

    public int DepartmentId { get; set; }

    public Department Department { get; set; }

    public DateTime CreatedAt { get; set; }
}
```

### Configuration

```csharp
public class StudentConfiguration
    : IEntityTypeConfiguration<Student>
{
    public void Configure(EntityTypeBuilder<Student> builder)
    {
        // Table
        builder.ToTable("Students");

        // Primary Key
        builder.HasKey(x => x.Id);

        // Name
        builder.Property(x => x.Name)
            .IsRequired()
            .HasMaxLength(100);

        // Email
        builder.Property(x => x.Email)
            .IsRequired()
            .HasMaxLength(200);

        // Unique index
        builder.HasIndex(x => x.Email)
            .IsUnique();

        // Age
        builder.Property(x => x.Age)
            .IsRequired();

        // Fees
        builder.Property(x => x.Fees)
            .HasPrecision(18, 2);

        // Default value
        builder.Property(x => x.IsActive)
            .HasDefaultValue(true);

        // CreatedAt
        builder.Property(x => x.CreatedAt)
            .HasDefaultValueSql("GETUTCDATE()");

        // Relationship
        builder.HasOne(x => x.Department)
            .WithMany(x => x.Students)
            .HasForeignKey(x => x.DepartmentId)
            .OnDelete(DeleteBehavior.Restrict);

        // Global filter
        builder.HasQueryFilter(x => !x.IsDeleted);

        // Check constraint
        builder.ToTable("Students", table =>
        {
            table.HasCheckConstraint(
                "CK_Student_Age",
                "Age >= 0");
        });
    }
}
```

That single class demonstrates a large percentage of real-world EF Core configuration.

---

# 49. `DbContext`

Now let's understand `DbContext`.

A simplified context:

```csharp
public class ApplicationDbContext : DbContext
{
    public DbSet<Student> Students { get; set; }

    public DbSet<Department> Departments { get; set; }

    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(
        ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.ApplyConfigurationsFromAssembly(
            typeof(ApplicationDbContext).Assembly);
    }
}
```

---

# 50. What is `DbContext`?

Think of `DbContext` as EF Core's primary gateway to the database.

It manages:

```text
Application
     |
     ↓
 DbContext
     |
     ├── DbSet<TEntity>
     |
     ├── Change Tracker
     |
     ├── Query translation
     |
     ├── Transactions
     |
     ├── Model metadata
     |
     └── SaveChanges
     |
     ↓
 Database
```

---

# 51. `DbSet<T>`

```csharp
public DbSet<Student> Students { get; set; }
```

Allows:

```csharp
context.Students
```

Examples:

```csharp
var students = await context.Students.ToListAsync();
```

```csharp
var student = await context.Students
    .FirstOrDefaultAsync(x => x.Id == id);
```

---

# 52. Is `DbSet` Mandatory?

Not necessarily.

EF Core can include entities in the model through:

- `DbSet<T>`
    
- relationships
    
- `OnModelCreating`
    
- configuration
    

So you can technically have an entity without exposing a public `DbSet<T>` property.

For example:

```csharp
modelBuilder.Entity<Student>();
```

You can still query using:

```csharp
context.Set<Student>();
```

---

# 53. `Set<T>()`

Very important:

```csharp
context.Set<Student>()
```

returns the `DbSet<Student>` for the entity type.

Example:

```csharp
var students = await context
    .Set<Student>()
    .ToListAsync();
```

Useful for generic repositories and generic infrastructure code.

---

# 54. `Add()`

```csharp
context.Students.Add(student);
```

Marks entity:

```text
Added
```

It does **not necessarily execute INSERT immediately**.

You need:

```csharp
await context.SaveChangesAsync();
```

Then SQL is executed.

---

# 55. `AddAsync()`

```csharp
await context.Students.AddAsync(student);
```

Usually:

```text
Add()
```

is sufficient for most entities.

`AddAsync()` exists mainly for cases involving asynchronous value generation. It does **not** mean database insertion happens asynchronously by itself.

---

# 56. `Update()`

```csharp
context.Students.Update(student);
```

Marks the entity as:

```text
Modified
```

and can cause an update during:

```csharp
SaveChanges();
```

Be careful with disconnected graphs: `Update()` can mark many properties/entities as modified.

---

# 57. `Remove()`

```csharp
context.Students.Remove(student);
```

marks:

```text
Deleted
```

Then:

```csharp
await context.SaveChangesAsync();
```

executes the delete.

---

# 58. `Attach()`

```csharp
context.Students.Attach(student);
```

marks the entity as:

```text
Unchanged
```

This is useful when you have an existing entity and want EF to start tracking it without treating it as newly inserted.

Example:

```csharp
var student = new Student
{
    Id = 10
};

context.Students.Attach(student);
```

EF assumes:

```text
Student 10 already exists
```

---

# 59. EntityState

EF Core tracks entities using states:

```csharp
EntityState.Detached
EntityState.Unchanged
EntityState.Added
EntityState.Modified
EntityState.Deleted
```

The lifecycle:

```text
Detached
   |
   | Add
   ↓
Added
   |
   | SaveChanges
   ↓
Unchanged
```

Update:

```text
Unchanged
    |
    | modify
    ↓
Modified
    |
    | SaveChanges
    ↓
Unchanged
```

Delete:

```text
Unchanged
    |
    | Remove
    ↓
Deleted
    |
    | SaveChanges
    ↓
Detached
```

---

# 60. `Entry()`

Very important.

```csharp
var entry = context.Entry(student);
```

You can inspect:

```csharp
entry.State
```

Example:

```csharp
if (context.Entry(student).State == EntityState.Modified)
{
    // ...
}
```

You can also manipulate individual property states:

```csharp
context.Entry(student)
    .Property(x => x.Name)
    .IsModified = true;
```

This is useful for partial updates.

---

# 61. `SaveChanges()`

This is one of the most important methods.

```csharp
context.SaveChanges();
```

It:

1. Detects changes
    
2. Generates SQL
    
3. Executes commands
    
4. Updates generated values
    
5. Updates entity states
    

Async:

```csharp
await context.SaveChangesAsync();
```

---

# 62. What Does `SaveChanges()` Return?

It returns the number of state entries written to the database.

```csharp
int affected = context.SaveChanges();
```

For example:

```text
2 entities inserted
1 entity updated
```

could result in:

```text
3
```

The exact count semantics can vary with provider/command batching, so don't interpret it as necessarily "number of SQL statements."

---

# 63. `Find()`

```csharp
var student = context.Students.Find(10);
```

`Find()` is special because EF Core first checks the **change tracker/local cache** for the entity by primary key before querying the database.

Async:

```csharp
var student = await context.Students.FindAsync(10);
```

This makes it different from:

```csharp
FirstOrDefault()
```

---

# 64. `First()`

```csharp
var student = context.Students
    .First(x => x.Id == id);
```

If nothing exists:

```text
InvalidOperationException
```

---

# 65. `FirstOrDefault()`

```csharp
var student = context.Students
    .FirstOrDefault(x => x.Id == id);
```

If no record:

```text
null
```

Usually safer when absence is expected.

---

# 66. `Single()`

```csharp
var student = context.Students
    .Single(x => x.Email == email);
```

It expects exactly one result.

If:

```text
0 results → exception
>1 result → exception
```

---

# 67. `SingleOrDefault()`

```csharp
var student = context.Students
    .SingleOrDefault(x => x.Email == email);
```

Expected:

```text
0 → null
1 → entity
2+ → exception
```

This is appropriate when the database/business invariant says there can be **at most one** matching row.

---

# 68. `Any()`

Check whether a record exists:

```csharp
bool exists = await context.Students
    .AnyAsync(x => x.Email == email);
```

Prefer this over:

```csharp
Count() > 0
```

because the intent is existence, and SQL can use an efficient existence check.

---

# 69. `Count()`

```csharp
var count = await context.Students.CountAsync();
```

With condition:

```csharp
var count = await context.Students
    .CountAsync(x => x.IsActive);
```

---

# 70. `Where()`

```csharp
var students = await context.Students
    .Where(x => x.Age >= 18)
    .ToListAsync();
```

Important:

`Where()` doesn't execute the database query immediately.

It's part of an `IQueryable`.

---

# 71. `Select()`

Projection:

```csharp
var students = await context.Students
    .Select(x => new
    {
        x.Id,
        x.Name
    })
    .ToListAsync();
```

This is usually better than retrieving entire entities when you only need a subset of columns.

---

# 72. `Include()`

Load related entities:

```csharp
var students = await context.Students
    .Include(x => x.Department)
    .ToListAsync();
```

SQL conceptually includes a join.

---

# 73. `ThenInclude()`

Nested navigation:

```csharp
var students = await context.Students
    .Include(x => x.Department)
        .ThenInclude(x => x.Head)
    .ToListAsync();
```

---

# 74. `AsNoTracking()`

For read-only queries:

```csharp
var students = await context.Students
    .AsNoTracking()
    .ToListAsync();
```

This tells EF Core:

> Don't track these entities for normal change persistence.

Benefits:

- Less memory
    
- Less change-tracking overhead
    
- Good for read-only operations
    

Don't blindly put `AsNoTracking()` everywhere. If you intend to modify the loaded entity and call `SaveChanges()`, normal tracking is often what you want.

---

# 75. `AsNoTrackingWithIdentityResolution()`

Useful when you want no normal tracking but want EF Core to avoid creating duplicate entity instances for the same identity within the query result.

```csharp
context.Students
    .AsNoTrackingWithIdentityResolution()
```

More specialized, but useful to know for interviews.

---

# 76. `IgnoreQueryFilters()`

If you have:

```csharp
builder.HasQueryFilter(x => !x.IsDeleted);
```

normal queries exclude deleted records.

To include them:

```csharp
var students = await context.Students
    .IgnoreQueryFilters()
    .ToListAsync();
```

---

# 77. `ToList()` vs `ToListAsync()`

Synchronous:

```csharp
var students = context.Students.ToList();
```

Asynchronous:

```csharp
var students = await context.Students.ToListAsync();
```

In ASP.NET Core applications, asynchronous database APIs are generally preferred for I/O operations.

---

# 78. `IQueryable` vs `IEnumerable`

This is an extremely common interview question.

### `IQueryable`

```csharp
IQueryable<Student> query = context.Students;
```

Operations can be translated into SQL.

Example:

```csharp
var students = await context.Students
    .Where(x => x.Age > 18)
    .ToListAsync();
```

Database does filtering.

Conceptually:

```sql
SELECT ...
FROM Students
WHERE Age > 18
```

### `IEnumerable`

Once materialized:

```csharp
var students = await context.Students.ToListAsync();

var result = students.Where(x => x.Age > 18);
```

Filtering happens in memory.

So:

```text
IQueryable
    ↓
Database-side query

IEnumerable
    ↓
Application-memory processing
```

---

# 79. Deferred Execution

This:

```csharp
var query = context.Students
    .Where(x => x.Age > 18);
```

doesn't necessarily execute SQL yet.

Execution happens when you materialize/execute:

```csharp
await query.ToListAsync();
```

or:

```csharp
await query.FirstAsync();
```

or:

```csharp
await query.CountAsync();
```

This is called **deferred execution**.

---

# 80. `ExecuteUpdate()`

Modern EF Core provides set-based updates:

```csharp
await context.Students
    .Where(x => x.IsActive)
    .ExecuteUpdateAsync(setters =>
        setters.SetProperty(x => x.IsActive, false));
```

This can execute a direct SQL `UPDATE` without loading every entity and tracking them.

This is very different from:

```csharp
var students = await context.Students
    .Where(x => x.IsActive)
    .ToListAsync();

foreach (var student in students)
{
    student.IsActive = false;
}

await context.SaveChangesAsync();
```

---

# 81. `ExecuteDelete()`

Similarly:

```csharp
await context.Students
    .Where(x => x.IsDeleted)
    .ExecuteDeleteAsync();
```

This executes a set-based delete directly.

Important:

> `ExecuteUpdate` and `ExecuteDelete` bypass the normal change tracker.

---

# 82. `Database` Property

You can access database-level functionality:

```csharp
context.Database
```

For example:

```csharp
await context.Database.EnsureCreatedAsync();
```

or:

```csharp
await context.Database.MigrateAsync();
```

---

# 83. `EnsureCreated()`

```csharp
context.Database.EnsureCreated();
```

Creates the database/schema if it doesn't exist.

But:

> `EnsureCreated()` bypasses the migrations workflow.

It is generally useful for:

- prototypes
    
- tests
    
- temporary databases
    

It is **not normally the production migrations strategy**.

---

# 84. `Migrate()`

```csharp
context.Database.Migrate();
```

Applies pending migrations.

Async:

```csharp
await context.Database.MigrateAsync();
```

This is the migrations-oriented approach.

---

# 85. Migrations

Suppose you initially have:

```csharp
public class Student
{
    public int Id { get; set; }
}
```

Later:

```csharp
public string Email { get; set; }
```

You create a migration.

CLI:

```bash
dotnet ef migrations add AddStudentEmail
```

Then:

```bash
dotnet ef database update
```

Conceptually:

```text
C# Model
   ↓
Migration
   ↓
SQL
   ↓
Database
```

---

# 86. Model Building Lifecycle

One of the most important concepts:

```text
DbContext created
       ↓
EF builds/obtains model metadata
       ↓
OnModelCreating()
       ↓
Configurations applied
       ↓
Model finalized/cached
       ↓
Queries / SaveChanges
```

`OnModelCreating()` is about **model metadata configuration**, not executing your normal CRUD operations.

---

# 87. `OnModelCreating()`

Typical:

```csharp
protected override void OnModelCreating(
    ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    modelBuilder.ApplyConfigurationsFromAssembly(
        typeof(ApplicationDbContext).Assembly);
}
```

You can also configure directly:

```csharp
modelBuilder.Entity<Student>(builder =>
{
    builder.HasKey(x => x.Id);
});
```

---

# 88. `OnConfiguring()`

Another important `DbContext` method:

```csharp
protected override void OnConfiguring(
    DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseSqlServer(connectionString);
}
```

But in ASP.NET Core, you normally configure the context through dependency injection:

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(
    options =>
        options.UseSqlServer(connectionString));
```

Then your context receives:

```csharp
DbContextOptions<ApplicationDbContext>
```

through its constructor.

---

# 89. `OnConfiguring` vs DI

### `OnConfiguring`

```csharp
protected override void OnConfiguring(
    DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseSqlServer(...);
}
```

### ASP.NET Core DI

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(
    options =>
        options.UseSqlServer(connectionString));
```

Production ASP.NET Core applications commonly use DI because it centralizes configuration and makes testing/configuration cleaner.

---

# 90. DbContext Lifetime

Typical:

```csharp
services.AddDbContext<ApplicationDbContext>();
```

Registers it as **scoped** by default.

In ASP.NET Core:

```text
HTTP Request
     |
     ↓
 DbContext
     |
     ↓
 HTTP Request ends
     |
     ↓
 DbContext disposed
```

This is important because `DbContext` is intended to represent a relatively short-lived **unit of work**.

---

# 91. DbContext Is Not Thread-Safe

Do not do:

```csharp
await Task.WhenAll(
    context.Students.ToListAsync(),
    context.Courses.ToListAsync()
);
```

using the same `DbContext` concurrently.

A `DbContext` instance should generally not be used concurrently across multiple threads/tasks.

---

# 92. `ChangeTracker`

You can access:

```csharp
context.ChangeTracker
```

For example:

```csharp
var entries = context.ChangeTracker.Entries();
```

You can inspect:

```csharp
entry.Entity
entry.State
```

Example:

```csharp
foreach (var entry in context.ChangeTracker.Entries())
{
    Console.WriteLine(
        $"{entry.Entity.GetType().Name}: {entry.State}");
}
```

---

# 93. Detect Changes

EF Core uses change detection to identify modifications.

You can explicitly call:

```csharp
context.ChangeTracker.DetectChanges();
```

Normally you don't need to call this manually.

---

# 94. `SaveChanges` Pipeline

Conceptually:

```text
Entity changes
      ↓
ChangeTracker
      ↓
DetectChanges
      ↓
Generate modification commands
      ↓
Transaction / command execution
      ↓
Database
      ↓
Update generated values
      ↓
Accept state changes
```

---

# 95. Transactions

EF Core supports transactions through:

```csharp
using var transaction =
    await context.Database.BeginTransactionAsync();

try
{
    // operations

    await context.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

Also, a single `SaveChanges()` is normally transactional for the set of operations it writes, subject to provider behavior.

---

# 96. Execution Strategy

For cloud databases, transient failures can occur.

EF Core providers can support execution strategies/retries.

For SQL Server:

```csharp
options.EnableRetryOnFailure();
```

Example:

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(
    options =>
        options.UseSqlServer(
            connectionString,
            sql =>
                sql.EnableRetryOnFailure()));
```

---

# 97. Logging SQL

Useful when debugging:

```csharp
options
    .UseSqlServer(connectionString)
    .LogTo(Console.WriteLine);
```

In production, use structured logging and appropriate log levels rather than dumping sensitive SQL indiscriminately.

---

# 98. `EnableSensitiveDataLogging`

Development/debugging:

```csharp
options.EnableSensitiveDataLogging();
```

Be careful in production because parameter values may contain sensitive information.

---

# 99. `EnableDetailedErrors`

Useful during development:

```csharp
options.EnableDetailedErrors();
```

---

# 100. Complete Project Structure

A clean architecture might look like:

```text
MyApplication
│
├── Domain
│   ├── Entities
│   │   ├── Student.cs
│   │   ├── Department.cs
│   │   └── Course.cs
│
├── Infrastructure
│   ├── Persistence
│   │   ├── ApplicationDbContext.cs
│   │   │
│   │   └── Configurations
│   │       ├── StudentConfiguration.cs
│   │       ├── DepartmentConfiguration.cs
│   │       └── CourseConfiguration.cs
│
└── API
```

Then:

```csharp
public class ApplicationDbContext : DbContext
{
    public DbSet<Student> Students => Set<Student>();
    public DbSet<Department> Departments => Set<Department>();

    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(
        ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.ApplyConfigurationsFromAssembly(
            typeof(ApplicationDbContext).Assembly);
    }
}
```

---

# 101. Complete Fluent API Cheat Sheet

Here's the important Fluent API vocabulary.

## Entity/table

```csharp
builder.ToTable("Students");

builder.ToTable("Students", "school");
```

## Primary key

```csharp
builder.HasKey(x => x.Id);
```

## Composite key

```csharp
builder.HasKey(x => new
{
    x.StudentId,
    x.CourseId
});
```

## Property

```csharp
builder.Property(x => x.Name);
```

## Required

```csharp
builder.Property(x => x.Name)
    .IsRequired();
```

## Max length

```csharp
builder.Property(x => x.Name)
    .HasMaxLength(100);
```

## Column name

```csharp
builder.Property(x => x.Name)
    .HasColumnName("student_name");
```

## Column type

```csharp
builder.Property(x => x.Price)
    .HasColumnType("decimal(18,2)");
```

## Precision

```csharp
builder.Property(x => x.Price)
    .HasPrecision(18, 2);
```

## Default value

```csharp
builder.Property(x => x.IsActive)
    .HasDefaultValue(true);
```

## Default SQL

```csharp
builder.Property(x => x.CreatedAt)
    .HasDefaultValueSql("GETUTCDATE()");
```

## Index

```csharp
builder.HasIndex(x => x.Email);
```

## Unique index

```csharp
builder.HasIndex(x => x.Email)
    .IsUnique();
```

## Composite index

```csharp
builder.HasIndex(x => new
{
    x.LastName,
    x.FirstName
});
```

## Ignore property

```csharp
builder.Ignore(x => x.FullName);
```

## One-to-many

```csharp
builder.HasOne(x => x.Department)
    .WithMany(x => x.Students)
    .HasForeignKey(x => x.DepartmentId);
```

## One-to-one

```csharp
builder.HasOne(x => x.Profile)
    .WithOne(x => x.Student)
    .HasForeignKey<StudentProfile>(x => x.StudentId);
```

## Delete behavior

```csharp
.OnDelete(DeleteBehavior.Cascade);
```

## Query filter

```csharp
builder.HasQueryFilter(x => !x.IsDeleted);
```

## Conversion

```csharp
builder.Property(x => x.Status)
    .HasConversion<string>();
```

## Check constraint

```csharp
builder.ToTable("Students", table =>
{
    table.HasCheckConstraint(
        "CK_Student_Age",
        "Age >= 18");
});
```

---

# 102. Important DbContext Methods / APIs for Interviews

Memorize this group.

|API|Purpose|
|---|---|
|`Add()`|Mark entity Added|
|`AddAsync()`|Async add API|
|`Update()`|Mark entity/graph Modified|
|`Remove()`|Mark entity Deleted|
|`Attach()`|Track as Unchanged|
|`Entry()`|Access tracking metadata|
|`Set<T>()`|Get `DbSet<T>` dynamically|
|`SaveChanges()`|Persist changes|
|`SaveChangesAsync()`|Async persistence|
|`Find()`|Find by primary key, checks tracker first|
|`FindAsync()`|Async PK lookup|
|`Database`|Database-level APIs|
|`ChangeTracker`|Tracking APIs|
|`OnModelCreating()`|Configure EF model|
|`OnConfiguring()`|Configure context options|
|`Migrate()`|Apply migrations|
|`EnsureCreated()`|Create schema without migrations workflow|

---

# 103. Query APIs You Must Know

|API|Meaning|
|---|---|
|`Where()`|Filter|
|`Select()`|Projection|
|`OrderBy()`|Sort ascending|
|`OrderByDescending()`|Sort descending|
|`ThenBy()`|Secondary sort|
|`Include()`|Eager-load navigation|
|`ThenInclude()`|Nested eager loading|
|`First()`|First, exception if none|
|`FirstOrDefault()`|First or null|
|`Single()`|Exactly one|
|`SingleOrDefault()`|Zero/one, exception if multiple|
|`Any()`|Exists?|
|`Count()`|Count|
|`Sum()`|Sum|
|`Average()`|Average|
|`Min()`|Minimum|
|`Max()`|Maximum|
|`ToList()`|Execute/materialize|
|`ToListAsync()`|Async materialization|
|`AsNoTracking()`|Disable normal tracking|
|`IgnoreQueryFilters()`|Ignore global filters|

---

# 104. A Very Important Interview Question

### Q: Where should database configuration live?

Good answer:

> Entity-specific database mapping can be implemented using `IEntityTypeConfiguration<TEntity>` and the Fluent API. Each entity has a dedicated configuration class, and the configurations can be registered using `ApplyConfiguration()` or automatically discovered with `ApplyConfigurationsFromAssembly()` inside `OnModelCreating()`.

---

# 105. Another Interview Question

### Q: Data Annotation vs Fluent API?

### Data Annotation

```csharp
[Required]
[MaxLength(100)]
public string Name { get; set; }
```

### Fluent API

```csharp
builder.Property(x => x.Name)
    .IsRequired()
    .HasMaxLength(100);
```

Fluent API generally provides:

- More expressive configuration
    
- Better separation of domain classes from persistence concerns
    
- Relationship configuration
    
- Composite keys
    
- Indexes
    
- Constraints
    
- Provider-specific configuration
    
- Centralized mapping
    

A useful rule:

```text
Simple metadata
    ↓
Data Annotation can work

Complex persistence mapping
    ↓
Fluent API
```

---

# 106. Fluent API Configuration Precedence

A useful high-level rule is:

```text
Conventions
      ↓
Data Annotations
      ↓
Fluent API
```

Fluent configuration generally takes precedence over conventions and conflicting annotations.

For interviews, say:

> EF Core conventions establish defaults, annotations can override conventions, and Fluent API configuration has the highest precedence when configuring the model.

---

# 107. Entity Configuration vs Migration

These are **not the same thing**.

### Configuration

```csharp
builder.ToTable("Students");
builder.HasKey(x => x.Id);
```

defines the EF model.

### Migration

```bash
dotnet ef migrations add InitialCreate
```

captures a model change and generates operations to transition the database schema.

Think:

```text
Configuration
      ↓
EF Model
      ↓
Migration
      ↓
Database Schema
```

---

# 108. Database Constraint vs C# Validation

Very important architectural distinction.

Suppose:

```csharp
builder.ToTable("Students", table =>
{
    table.HasCheckConstraint(
        "CK_Student_Age",
        "Age >= 18");
});
```

This is a **database-level constraint**.

But:

```csharp
[Range(18, 100)]
public int Age { get; set; }
```

is generally an application/model validation mechanism, not equivalent to a database constraint.

For critical invariants, database constraints can provide protection even when data is written outside your application.

---

# 109. Example: Everything Together

Let's create an `Order`.

```csharp
public class Order
{
    public long Id { get; set; }

    public string OrderNumber { get; set; }

    public decimal TotalAmount { get; set; }

    public DateTime CreatedAt { get; set; }

    public int CustomerId { get; set; }

    public Customer Customer { get; set; }

    public bool IsDeleted { get; set; }
}
```

Configuration:

```csharp
public class OrderConfiguration
    : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.ToTable("Orders");

        builder.HasKey(x => x.Id);

        builder.Property(x => x.OrderNumber)
            .IsRequired()
            .HasMaxLength(50);

        builder.HasIndex(x => x.OrderNumber)
            .IsUnique();

        builder.Property(x => x.TotalAmount)
            .HasPrecision(18, 2);

        builder.Property(x => x.CreatedAt)
            .HasDefaultValueSql("GETUTCDATE()");

        builder.HasOne(x => x.Customer)
            .WithMany(x => x.Orders)
            .HasForeignKey(x => x.CustomerId)
            .OnDelete(DeleteBehavior.Restrict);

        builder.HasQueryFilter(x => !x.IsDeleted);

        builder.ToTable("Orders", table =>
        {
            table.HasCheckConstraint(
                "CK_Order_TotalAmount",
                "TotalAmount >= 0");
        });
    }
}
```

This gives you:

```text
Orders
------------------------------------------------
Id              PK
OrderNumber     NOT NULL
                UNIQUE
TotalAmount     decimal(18,2)
CreatedAt       DB DEFAULT
CustomerId      FK
IsDeleted
------------------------------------------------

Constraints:
    PK
    FK
    UNIQUE
    CHECK
```

---

# 110. Recommended Mental Model

When you see an EF Core entity, think in this order:

```text
ENTITY
  ↓
TABLE
  ↓
COLUMNS
  ↓
PRIMARY KEY
  ↓
FOREIGN KEYS
  ↓
INDEXES
  ↓
CONSTRAINTS
  ↓
DEFAULTS
  ↓
RELATIONSHIPS
  ↓
QUERY BEHAVIOR
  ↓
CHANGE TRACKING
```

And map them to Fluent API:

```text
TABLE          → ToTable()
COLUMN         → Property()
PRIMARY KEY    → HasKey()
FOREIGN KEY    → HasForeignKey()
INDEX          → HasIndex()
CHECK          → HasCheckConstraint()
RELATIONSHIP   → HasOne()/HasMany()
DEFAULT        → HasDefaultValue()
DEFAULT SQL    → HasDefaultValueSql()
FILTER         → HasQueryFilter()
CONVERSION     → HasConversion()
```

---

# 111. The Most Important Interview Questions to Prepare

If you're preparing for an EF Core interview, I would prioritize these:

### Fundamentals

1. What is EF Core?
    
2. What is `DbContext`?
    
3. What is `DbSet`?
    
4. What is change tracking?
    
5. What is `SaveChanges()`?
    
6. What is `IQueryable`?
    
7. What is deferred execution?
    

### Configuration

8. What is Fluent API?
    
9. What is `IEntityTypeConfiguration<T>`?
    
10. Why use separate configuration classes?
    
11. How does `ApplyConfigurationsFromAssembly()` work?
    
12. How do you change table name?
    
13. How do you change column name?
    
14. How do you configure required properties?
    
15. How do you configure max length?
    
16. How do you configure decimal precision?
    

### Keys

17. How do you configure a primary key?
    
18. How do you configure a composite primary key?
    
19. Difference between primary key and alternate key?
    
20. Difference between unique index and alternate key?
    

### Relationships

21. How do you configure one-to-many?
    
22. How do you configure one-to-one?
    
23. How do you configure many-to-many?
    
24. What is a foreign key?
    
25. What is a navigation property?
    
26. What is cascade delete?
    

### Database

27. How do you create indexes?
    
28. How do you create unique indexes?
    
29. How do you create check constraints?
    
30. What are shadow properties?
    
31. What are global query filters?
    
32. What are value conversions?
    

### DbContext

33. Difference between `Add`, `Attach`, `Update`, and `Remove`?
    
34. Difference between `Find` and `FirstOrDefault`?
    
35. Difference between `First` and `Single`?
    
36. What is `AsNoTracking`?
    
37. What is `Include`?
    
38. What is `ChangeTracker`?
    
39. Is `DbContext` thread-safe?
    
40. What is the lifetime of DbContext?
    

### Migrations

41. What is a migration?
    
42. `EnsureCreated()` vs `Migrate()`?
    
43. How does EF Core generate SQL?
    
44. What happens during `SaveChanges()`?
    
45. How do transactions work?
    

---

# 112. The 10 Things I Would Memorize First

If you have limited time, memorize these:

### 1. Configuration interface

```csharp
IEntityTypeConfiguration<Student>
```

### 2. Configuration method

```csharp
public void Configure(EntityTypeBuilder<Student> builder)
```

### 3. Register all configurations

```csharp
modelBuilder.ApplyConfigurationsFromAssembly(
    typeof(ApplicationDbContext).Assembly);
```

### 4. Primary key

```csharp
builder.HasKey(x => x.Id);
```

### 5. Composite key

```csharp
builder.HasKey(x => new
{
    x.StudentId,
    x.CourseId
});
```

### 6. Table

```csharp
builder.ToTable("Students");
```

### 7. Unique index

```csharp
builder.HasIndex(x => x.Email)
    .IsUnique();
```

### 8. Relationship

```csharp
builder.HasOne(x => x.Department)
    .WithMany(x => x.Students)
    .HasForeignKey(x => x.DepartmentId);
```

### 9. Required/max length

```csharp
builder.Property(x => x.Name)
    .IsRequired()
    .HasMaxLength(100);
```

### 10. Save

```csharp
await context.SaveChangesAsync();
```

---

## Final architecture to remember

```text
                  EF CORE
                     │
                     ▼
                 DbContext
                     │
          ┌──────────┴──────────┐
          │                     │
       DbSet<T>            OnModelCreating
          │                     │
          │                     ▼
       CRUD/query       Fluent API Configuration
                                │
                                ▼
                  IEntityTypeConfiguration<T>
                                │
                                ▼
                       EntityTypeBuilder<T>
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
          HasKey()           Property()         HasOne()
             │                  │                  │
             ▼                  ▼                  ▼
        Primary Key         Columns          Relationships
             │
             ├── Composite Key
             │
             ├── Indexes
             │
             ├── Constraints
             │
             └── Foreign Keys
                                │
                                ▼
                         EF Core Model
                                │
                                ▼
                            Migration
                                │
                                ▼
                           SQL Database
```

The **core idea** is that `IEntityTypeConfiguration<T>` is not some special database mechanism by itself. It is a **clean organizational abstraction around EF Core's Fluent API**. The actual mapping is performed through `EntityTypeBuilder<T>`, and `ApplyConfigurationsFromAssembly()` is simply a convenient way of discovering and applying all those mappings to the EF Core model.