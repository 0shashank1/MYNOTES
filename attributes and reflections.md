# Attributes and Reflection in C\#

These are two important C# features often used together, especially in **frameworks, serialization, dependency injection, testing, ORMs, and metadata-driven programming**.

---

## 1. Attributes in C\#

An **attribute** is metadata that you attach to a class, method, property, parameter, etc.

Think of it as:

> **"Attach extra information to this piece of code."**

### Example

```csharp
[Obsolete("Use NewMethod() instead")]
static void OldMethod()
{
    Console.WriteLine("Old method");
}
```

`[Obsolete]` is an attribute.

The compiler/runtime can use that metadata to determine something about `OldMethod()`.

### Common built-in attributes

```csharp
[Obsolete]
[Serializable]
[DllImport("user32.dll")]
[Conditional("DEBUG")]
```

Attributes can be applied to:

- Classes

- Methods

- Properties

- Fields

- Parameters

- Assemblies

- Events
    
- Constructors
    
- etc.
    

---

# 2. Creating a Custom Attribute

You can create your own attribute by inheriting from `Attribute`.

```csharp
public class AuthorAttribute : Attribute
{
    public string Name { get; }

    public AuthorAttribute(string name)
    {
        Name = name;
    }
}
```

Then use it:

```csharp
[Author("Shashank")]
public class Student
{
}
```

The attribute doesn't automatically _do_ anything. It simply stores metadata.

---

# 3. Reflection in C\#

**Reflection** allows a program to inspect types and their metadata **at runtime**.

For example, you can ask:

- What methods does this class have?
    
- What properties does it have?
    
- What attributes are attached?
    
- What is the type of a property?
    
- Can I create an instance dynamically?
    
- Can I invoke a method dynamically?
    

The main namespace is:

```csharp
using System.Reflection;
```

---

## 4. Basic Reflection Example

Consider:

```csharp
public class Student
{
    public string Name { get; set; }

    public void Study()
    {
        Console.WriteLine("Studying...");
    }
}
```

You can inspect it:

```csharp
Type type = typeof(Student);

Console.WriteLine(type.Name);
```

Output:

```text
Student
```

---

## 5. Getting Properties

```csharp
Type type = typeof(Student);

PropertyInfo[] properties = type.GetProperties();

foreach (PropertyInfo property in properties)
{
    Console.WriteLine(property.Name);
}
```

Output:

```text
Name
```

You can also inspect the property's type:

```csharp
PropertyInfo property = typeof(Student).GetProperty("Name")!;

Console.WriteLine(property.PropertyType);
```

Output:

```text
System.String
```

---

# 6. Getting Methods

```csharp
Type type = typeof(Student);

MethodInfo[] methods = type.GetMethods();

foreach (MethodInfo method in methods)
{
    Console.WriteLine(method.Name);
}
```

You'll see methods such as:

```text
Study
get_Name
set_Name
...
```

The extra methods come from the property and base `object` class.

---

# 7. Reflection + Attributes

This is where attributes become particularly useful.

Suppose we have:

```csharp
[Author("Shashank")]
public class Student
{
    public string Name { get; set; }
}
```

We can retrieve the attribute using reflection:

```csharp
Type type = typeof(Student);

AuthorAttribute? attribute =
    type.GetCustomAttribute<AuthorAttribute>();

Console.WriteLine(attribute?.Name);
```

Output:

```text
Shashank
```

So the relationship is:

```text
Attribute
   ↓
stores metadata
   ↓
Reflection
   ↓
reads metadata at runtime
```

---

# 8. Attribute on a Property

You can also put attributes on properties.

```csharp
public class Student
{
    [Required]
    public string Name { get; set; }

    [Range(18, 60)]
    public int Age { get; set; }
}
```

Frameworks such as ASP.NET Core can inspect these attributes and apply behavior based on them.

For example:

```csharp
PropertyInfo[] properties =
    typeof(Student).GetProperties();

foreach (PropertyInfo property in properties)
{
    Console.WriteLine(property.Name);

    foreach (Attribute attribute in
             property.GetCustomAttributes())
    {
        Console.WriteLine($"  {attribute.GetType().Name}");
    }
}
```

Conceptually:

```text
Student
 ├── Name
 │    └── RequiredAttribute
 │
 └── Age
      └── RangeAttribute
```

---

# 9. Dynamically Creating Objects

Reflection can also create objects at runtime.

```csharp
Type type = typeof(Student);

object student = Activator.CreateInstance(type)!;
```

You can then access its properties dynamically:

```csharp
PropertyInfo property = type.GetProperty("Name")!;

property.SetValue(student, "Shashank");

Console.WriteLine(property.GetValue(student));
```

Output:

```text
Shashank
```

Notice that we didn't write:

```csharp
Student student = new Student();
student.Name = "Shashank";
```

Instead, reflection performed those operations dynamically.

---

# 10. Dynamically Calling a Method

```csharp
Student student = new Student();

MethodInfo method = typeof(Student).GetMethod("Study")!;

method.Invoke(student, null);
```

Output:

```text
Studying...
```

So reflection can perform:

```text
Find type
   ↓
Find method
   ↓
Find property
   ↓
Read/write property
   ↓
Invoke method
   ↓
Read attributes
```

---

# 11. Real-World Example

Attributes + reflection are commonly used in systems like:

### ASP.NET Core

```csharp
[HttpGet]
public IActionResult GetStudents()
{
    ...
}
```

ASP.NET Core can inspect the `[HttpGet]` attribute and determine that the method should handle an HTTP GET request.

### Entity Framework

```csharp
[Key]
public int Id { get; set; }
```

The attribute provides metadata describing the model.

### Serialization

```csharp
[JsonPropertyName("student_name")]
public string Name { get; set; }
```

A serializer can inspect the attribute and determine the JSON property name.

---

# 12. Important Difference

|Feature|Attributes|Reflection|
|---|---|---|
|Purpose|Add metadata|Inspect/use metadata|
|Works mainly at|Declaration/source metadata|Runtime|
|Example|`[Obsolete]`|`GetCustomAttributes()`|
|Can inspect methods?|Metadata attached to them|Yes|
|Can inspect properties?|Metadata attached to them|Yes|
|Can invoke methods?|No|Yes|
|Can create objects dynamically?|No|Yes|

### Easy way to remember

> **Attributes describe code. Reflection examines code.**

For example:

```csharp
[Author("Shashank")]
public class Student
{
}
```

Here:

- `[Author("Shashank")]` → **Attribute**
    
- `typeof(Student)` → obtains type information
    
- `GetCustomAttribute<AuthorAttribute>()` → **Reflection**
    
- `attribute.Name` → retrieves the metadata
    

---

## 13. The Bigger Picture

Attributes and reflection are powerful, but reflection has runtime costs and reduces compile-time guarantees.

For ordinary application code, prefer normal C# constructs:

```csharp
Student student = new Student();
student.Name = "Shashank";
student.Study();
```

Reflection becomes valuable when you're building **generic infrastructure** where the code cannot know the types/methods beforehand:

```text
             Your Classes
                  │
            Attributes
                  │
                  ▼
         Metadata at runtime
                  │
                  ▼
            Reflection
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
   Frameworks   DI/ORM   Serializers
```

This is one of the core mechanisms behind how many .NET frameworks can work with classes **without you explicitly telling the framework every detail about them**.