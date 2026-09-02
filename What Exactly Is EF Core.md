

> **ORM fundamentals → EF Core architecture → DbContext → CRUD → Dependency Injection → Relationships → Loading strategies**

### Part 1 — ORM + EF Core Architecture

- What is ORM?
    
- Why ORM?
    
- EF Core vs ADO.NET
    
- EF Core architecture
    
- Entity, model, database
    
- `DbContext`
    
- `DbSet<T>`
    
- Change Tracker
    
- LINQ → SQL
    
- Provider concept
    
- Migrations
    

### Part 2 — `DbContext` + CRUD + DI

- What is `DbContext`?
    
- Lifetime of `DbContext`
    
- `DbSet`
    
- Create
    
- Read
    
- Update
    
- Delete
    
- `SaveChanges()` / `SaveChangesAsync()`
    
- Dependency Injection
    
- `AddDbContext`
    
- Scoped lifetime
    
- Repository/service interaction
    

### Part 3 — Relationships

- One-to-one
    
- One-to-many
    
- Many-to-many
    
- Primary key / foreign key
    
- Navigation properties
    
- Required vs optional relationships
    
- Fluent API
    
- Data annotations
    
- Cascade delete
    

### Part 4 — Loading

- Eager loading
    
- Explicit loading
    
- Lazy loading
    
- `Include()`
    
- `ThenInclude()`
    
- N+1 problem
    
- Projection with `Select()`
    
- When each approach should be used
    

---

# 1. Start Here: What is an ORM?

**ORM = Object-Relational Mapper.**

It is a technology that maps:

```text
C# Objects              Relational Database
-----------             -------------------
Student       <------>  Students table
Id            <------>  Id column
Name          <------>  Name column
Age           <------>  Age column
```

Without ORM, you might write SQL manually:

```sql
SELECT Id, Name, Age
FROM Students
WHERE Id = 10;
```

Then manually convert the result into a C# object.

With an ORM such as **Entity Framework Core**, you can write:

```csharp
var student = await context.Students
    .FirstOrDefaultAsync(s => s.Id == 10);
```

EF Core handles much of the mapping between the C# object model and relational database.

---

# 2. Why Do We Need ORM?

Suppose your database has:

```text
Students
----------------
Id
Name
Age
```

and your C# application has:

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}
```

The ORM understands that:

```text
Student class
     ↓
Students table

Id property
     ↓
Id column

Name property
     ↓
Name column

Age property
     ↓
Age column
```

This is called **object-relational mapping**.

---

# 3. What is Entity Framework Core?

**Entity Framework Core (EF Core)** is Microsoft's modern ORM for .NET.

It allows your application to work with databases using:

- C# objects
    
- LINQ
    
- `DbContext`
    
- `DbSet`
    
- Change Tracking
    
- Migrations
    

instead of writing SQL for every operation.

For example:

```csharp
var students = await context.Students
    .Where(s => s.Age >= 18)
    .ToListAsync();
```

Conceptually, EF Core translates this into SQL similar to:

```sql
SELECT *
FROM Students
WHERE Age >= 18;
```

The important interview point:

> **LINQ is not SQL. EF Core translates LINQ expressions into SQL when the query is executed against the database.**

---

# 4. EF Core Architecture

This is an important interview topic.

Think about EF Core like this:

```text
                 Your Application
                       │
                       ▼
                  DbContext
                       │
              ┌────────┴────────┐
              │                 │
           DbSet<T>        Change Tracker
              │                 │
              └────────┬────────┘
                       ▼
                 EF Core Engine
                       │
                 Query Translation
                       │
                       ▼
                Database Provider
              (SQL Server, PostgreSQL,
               SQLite, etc.)
                       │
                       ▼
                    Database
```

Let's understand each piece.

---

## 4.1 Entity

An **entity** is normally a C# class representing data that EF Core maps to the database.

Example:

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}
```

EF Core can map this to:

```text
Students
-----------------
Id
Name
Age
```

---

# 5. `DbContext`

This is probably the **most important EF Core class** for interviews.

`DbContext` represents a session/unit of work with the database.

Example:

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<Student> Students { get; set; }
}
```

You can think of it as:

```text
Application
     │
     ▼
 AppDbContext
     │
     ├── Students
     ├── Courses
     └── Departments
     │
     ▼
 Database
```

### Interview answer

If interviewer asks:

> **What is DbContext?**

A good answer:

> "`DbContext` is the primary class in EF Core for interacting with the database. It manages entity querying, change tracking, persistence of changes, and the database connection/configuration used by the context."

That's much better than:

> "DbContext is used to connect to the database."

Because **connection** is only part of what it does.

---

# 6. What is `DbSet<T>`?

Consider:

```csharp
public DbSet<Student> Students { get; set; }
```

`DbSet<Student>` represents the collection/query root for `Student` entities.

You can use it to query and manipulate students:

```csharp
context.Students
```

For example:

```csharp
var students = await context.Students.ToListAsync();
```

and:

```csharp
context.Students.Add(student);
```

Conceptually:

```text
DbContext
   │
   └── DbSet<Student>
          │
          ▼
      Students table
```

---

# 7. Change Tracker

This is another **very important interview concept**.

Suppose:

```csharp
var student = await context.Students
    .FirstAsync(s => s.Id == 1);
```

EF Core starts tracking that entity.

Initially:

```text
Student
Id = 1
Name = "Rahul"

State = Unchanged
```

Then:

```csharp
student.Name = "Shashank";
```

EF Core detects that the object has changed.

Conceptually:

```text
Before:
Name = Rahul
State = Unchanged

       ↓

student.Name = "Shashank"

       ↓

After:
Name = Shashank
State = Modified
```

Then:

```csharp
await context.SaveChangesAsync();
```

EF Core generates the necessary SQL, conceptually:

```sql
UPDATE Students
SET Name = 'Shashank'
WHERE Id = 1;
```

### Key interview concept

> **Change tracking allows EF Core to detect modifications to tracked entities and persist those changes when `SaveChanges()` or `SaveChangesAsync()` is called.**

---

# 8. CRUD

CRUD means:

|Operation|Meaning|EF Core|
|---|---|---|
|C|Create|`Add()`|
|R|Read|LINQ|
|U|Update|Modify + `SaveChanges()`|
|D|Delete|`Remove()`|

Let's learn each one.

---

## Create

```csharp
var student = new Student
{
    Name = "Rahul",
    Age = 21
};

context.Students.Add(student);

await context.SaveChangesAsync();
```

Flow:

```text
new Student
    ↓
Add()
    ↓
Change Tracker
    ↓
SaveChangesAsync()
    ↓
INSERT
    ↓
Database
```

---

# 9. Read

Simple query:

```csharp
var students = await context.Students
    .ToListAsync();
```

Filter:

```csharp
var students = await context.Students
    .Where(s => s.Age >= 18)
    .ToListAsync();
```

Find one:

```csharp
var student = await context.Students
    .FirstOrDefaultAsync(s => s.Id == id);
```

---

# 10. Update

First retrieve the entity:

```csharp
var student = await context.Students
    .FirstOrDefaultAsync(s => s.Id == id);

if (student == null)
{
    return;
}

student.Name = "Rahul";

await context.SaveChangesAsync();
```

Why does this work without:

```csharp
context.Students.Update(student);
```

Because the entity is already being **tracked**.

This is a classic interview question.

### Question

> Why can EF Core update an entity without explicitly calling `Update()`?

### Answer

Because when the entity is queried through a normal tracking query, EF Core's change tracker tracks it. After modifying its properties, `SaveChanges()` detects the changes and generates the appropriate `UPDATE`.

---

# 11. Delete

```csharp
var student = await context.Students
    .FirstOrDefaultAsync(s => s.Id == id);

if (student == null)
{
    return;
}

context.Students.Remove(student);

await context.SaveChangesAsync();
```

Conceptually:

```text
Remove()
   ↓
Deleted state
   ↓
SaveChanges()
   ↓
DELETE
```

---

# 12. Dependency Injection

Now we connect EF Core to ASP.NET Core.

Instead of doing this inside a controller:

```csharp
var context = new AppDbContext(...);
```

we use **Dependency Injection (DI)**.

In `Program.cs`:

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));
```

Then inject it:

```csharp
public class StudentsController : ControllerBase
{
    private readonly AppDbContext _context;

    public StudentsController(AppDbContext context)
    {
        _context = context;
    }
}
```

Now ASP.NET Core creates and supplies the `AppDbContext`.

---

# 13. Why is `DbContext` Usually Scoped?

This is a **very common interview question**.

`AddDbContext<T>()` registers `DbContext` with a **scoped lifetime by default**.

In a typical ASP.NET Core web application:

```text
HTTP Request
     │
     ▼
Create DbContext
     │
     ├── Query
     ├── Track changes
     ├── Insert/update/delete
     │
     ▼
SaveChanges
     │
     ▼
Request ends
     │
     ▼
Dispose DbContext
```

The key idea:

> A `DbContext` is generally intended to represent a short-lived unit of work.

You normally **should not make a normal request-scoped `DbContext` a singleton**.

---

# 14. Relationships

Now suppose we have:

```text
Department
     │
     │ 1
     │
     │
     │ *
     ▼
Student
```

One department can have many students.

C#:

```csharp
public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }

    public ICollection<Student> Students { get; set; }
        = new List<Student>();
}
```

```csharp
public class Student
{
    public int Id { get; set; }

    public string Name { get; set; }

    public int DepartmentId { get; set; }

    public Department Department { get; set; }
}
```

Here:

```text
Student.DepartmentId
```

is the **foreign key**.

And:

```csharp
Student.Department
```

is a **navigation property**.

---

# 15. Types of Relationships

You need to know these extremely well.

### One-to-One

```text
Person ───── Passport
   1             1
```

Example:

```text
User → UserProfile
```

---

### One-to-Many

```text
Department
     1
     │
     │
     *
  Students
```

One department has many students.

This is probably the most common relationship you'll encounter.

---

### Many-to-Many

```text
Student       Course
   *            *
    \          /
     \        /
      Enrollment
```

A student can take multiple courses.

A course can have multiple students.

Modern EF Core can represent many-to-many relationships using skip navigations, while an explicit join entity is appropriate when the relationship itself has additional data.

For example:

```text
Student
Course
Enrollment
    ├── EnrollmentDate
    └── Grade
```

---

# 16. Loading Related Data

This is the final major topic you listed.

Suppose:

```csharp
Student
   │
   └── Department
```

You retrieve the student:

```csharp
var student = await context.Students
    .FirstAsync();
```

Does `student.Department` automatically contain the department?

**Not necessarily.**

This is where loading strategies come in.

There are three major approaches:

```text
Eager Loading
Explicit Loading
Lazy Loading
```

---

# 17. Eager Loading

You explicitly tell EF Core to load the related data as part of the query.

```csharp
var students = await context.Students
    .Include(s => s.Department)
    .ToListAsync();
```

Conceptually:

```text
Students
   +
Department
```

are fetched as part of the query.

### Interview answer

> **Eager loading means loading related entities as part of the initial query, typically using `Include()` and `ThenInclude()`.**

---

# 18. `ThenInclude()`

Suppose:

```text
Student
  ↓
Department
  ↓
University
```

You can load:

```csharp
var students = await context.Students
    .Include(s => s.Department)
        .ThenInclude(d => d.University)
    .ToListAsync();
```

Meaning:

```text
Student
   │
   └── Department
          │
          └── University
```

---

# 19. Explicit Loading

Here you load related data manually after retrieving the entity.

Example:

```csharp
var student = await context.Students
    .FirstAsync(s => s.Id == id);
```

Then:

```csharp
await context.Entry(student)
    .Reference(s => s.Department)
    .LoadAsync();
```

You explicitly tell EF Core:

> "Now load this student's Department."

For collections:

```csharp
await context.Entry(department)
    .Collection(d => d.Students)
    .LoadAsync();
```

---

# 20. Lazy Loading

Lazy loading means related data is loaded **when you access the navigation property**, assuming lazy-loading has been configured.

Conceptually:

```csharp
var student = await context.Students
    .FirstAsync();

var department = student.Department;
```

Accessing:

```csharp
student.Department
```

can cause another database query.

This sounds convenient, but there is an important danger.

---

# 21. The N+1 Problem

This is an **important interview question**.

Suppose you have 100 students:

```csharp
var students = await context.Students
    .ToListAsync();

foreach (var student in students)
{
    Console.WriteLine(student.Department.Name);
}
```

With inappropriate lazy loading, you might get:

```text
1 query → load students

+ 100 queries → load each department
```

Total:

```text
101 database queries
```

That's the **N+1 query problem**.

A better approach may be:

```csharp
var students = await context.Students
    .Include(s => s.Department)
    .ToListAsync();
```

Or, often even better when you only need specific fields:

```csharp
var students = await context.Students
    .Select(s => new
    {
        s.Id,
        s.Name,
        DepartmentName = s.Department.Name
    })
    .ToListAsync();
```

This is called **projection**.

---

# 22. Eager vs Explicit vs Lazy

Memorize the conceptual difference:

|Strategy|When data loads|
|---|---|
|Eager|Initial query|
|Explicit|When you explicitly request it|
|Lazy|When navigation property is accessed|

### Interview question

> Which loading strategy do you prefer?

Don't simply answer:

> "Eager loading."

A stronger answer:

> "It depends on the use case. I prefer explicit query shaping or projection when I know exactly what data the API needs. Eager loading with `Include` is useful when the related entities are genuinely required. Lazy loading can be convenient but I use it cautiously because it can hide database queries and cause N+1 problems."

That's a **much stronger interview answer**.

---

# Your Mental Model

You should eventually be able to visualize EF Core like this:

```text
                         EF CORE
                            │
             ┌──────────────┴──────────────┐
             │                             │
         DbContext                     Entity Model
             │                             │
       ┌─────┴─────┐                 ┌─────┴─────┐
       │           │                 │           │
    DbSet<T>   Change Tracker      Entities   Relationships
       │           │                 │           │
       └─────┬─────┘                 └─────┬─────┘
             │                             │
             └──────────────┬──────────────┘
                            │
                       LINQ Queries
                            │
                            ▼
                     Query Translation
                            │
                            ▼
                        SQL Provider
                            │
                            ▼
                        Database
```

And for an ASP.NET Core application:

```text
HTTP Request
     │
     ▼
Controller / API
     │
     ▼
Service
     │
     ▼
DbContext
     │
     ├── DbSet
     ├── LINQ
     ├── Change Tracker
     └── SaveChanges
            │
            ▼
         Database
```

---

# Interview Questions You Should Master

After learning these topics, you should be able to answer these **without memorizing a definition**:

### ORM

1. What is ORM?
    
2. Why use an ORM?
    
3. What problem does EF Core solve?
    
4. EF Core vs ADO.NET?
    
5. What is an entity?
    

### Architecture

6. What is `DbContext`?
    
7. What is `DbSet`?
    
8. What is change tracking?
    
9. How does LINQ become SQL?
    
10. What is a database provider?
    
11. What are migrations?
    

### CRUD

12. How do you insert an entity?
    
13. How do you query an entity?
    
14. How does EF Core detect updates?
    
15. Difference between `Add()` and `Update()`?
    
16. How do you delete an entity?
    
17. What does `SaveChanges()` do?
    
18. `SaveChanges()` vs `SaveChangesAsync()`?
    

### DI

19. Why inject `DbContext`?
    
20. Why is `DbContext` scoped?
    
21. What happens if you make it singleton?
    
22. How do you register EF Core in ASP.NET Core?
    

### Relationships

23. What is a navigation property?
    
24. What is a foreign key?
    
25. One-to-one vs one-to-many vs many-to-many?
    
26. How do you configure relationships?
    
27. Data annotations vs Fluent API?
    
28. What is cascade delete?
    

### Loading

29. What is eager loading?
    
30. What is explicit loading?
    
31. What is lazy loading?
    
32. `Include()` vs `ThenInclude()`?
    
33. What is the N+1 problem?
    
34. How can you avoid N+1?
    
35. Why use projection instead of `Include()` sometimes?
    

---

## How I recommend we learn this

Don't try to memorize all of this at once.

We'll do it like an **actual interview-oriented course**, progressively:

```text
LESSON 1
ORM fundamentals
       ↓
LESSON 2
EF Core architecture
       ↓
LESSON 3
DbContext + DbSet + Change Tracking
       ↓
LESSON 4
CRUD with real code
       ↓
LESSON 5
Dependency Injection + DbContext lifetime
       ↓
LESSON 6
Relationships
       ↓
LESSON 7
Eager / Explicit / Lazy loading
       ↓
LESSON 8
N+1 + projection + performance
       ↓
LESSON 9
EF Core interview questions
       ↓
LESSON 10
Mock interview
```

**Start with Lesson 1: ORM fundamentals.** I'll teach it from absolute basics, then give you a small coding exercise and interview questions before moving to EF Core architecture.