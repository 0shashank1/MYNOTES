
```
C# source
   │
   ▼
Roslyn compiler
   │
   ├── Parse
   ├── Bind / type-check
   ├── Resolve references
   └── Emit
        │
        ▼
   IL + metadata
        │
        ▼
      .dll / .exe
        │
        ▼
   CLR / .NET runtime
        │
        ├── Load assemblies
        ├── Resolve types/methods
        ├── JIT compile
        └── Execute native machine code
```

The interesting part is **assembly references and "linking."**

---

# 1. Start with a tiny example

Suppose we have two projects:

```
MyApp/
│
├── MyApp.Core/
│   └── Calculator.cs
│
└── MyApp/
    └── Program.cs
```

### `MyApp.Core`

```
namespace MyApp.Core;

public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}
```

### `MyApp`

```
using MyApp.Core;

var calculator = new Calculator();

Console.WriteLine(calculator.Add(10, 20));
```

And:

```
MyApp
   │
   └── references
          ↓
     MyApp.Core
```

The question is:

> What actually happens from source code to execution?

---

# 2. C# source code is compiled by Roslyn

The standard C# compiler is **Roslyn**.

You give it:

```
var calculator = new Calculator();

Console.WriteLine(calculator.Add(10, 20));
```

The compiler doesn't directly produce x86/x64/ARM machine code.

Instead, it produces **CIL/IL** (Common Intermediate Language / Intermediate Language).

Conceptually:

```
C#
 ↓
IL
 ↓
Machine code
```

For example, something approximately like this could be generated:

```
newobj     Calculator::.ctor
stloc      calculator

ldloc      calculator
ldc.i4     10
ldc.i4     20
callvirt   Calculator::Add
call       Console::WriteLine
```

The exact IL depends on the compiler and code.

---

# 3. The output is an assembly

After compilation you typically get:

```
MyApp.dll
```

or:

```
MyApp.exe
```

But an assembly is **not simply a DLL containing machine code**.

A .NET assembly contains things such as:

```
Assembly
│
├── IL
├── Type metadata
├── Method metadata
├── Assembly metadata
├── References to other assemblies
├── Manifest
└── Resources
```

This is a critical concept.

### A .NET assembly is both:

**code + metadata + dependency information**

---

# 4. What is metadata?

Suppose you have:

```
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}
```

The compiler emits metadata describing things like:

```
Type:
    MyApp.Core.Calculator

Methods:
    .ctor()
    Add(Int32, Int32) : Int32

Accessibility:
    public

Base type:
    System.Object
```

The runtime can inspect this metadata.

That's why things like reflection are possible:

```
typeof(Calculator)
```

and:

```
typeof(Calculator).GetMethods()
```

The runtime knows what `Calculator` is because the assembly contains metadata describing it.

---

# 5. What does an assembly reference actually mean?

Now suppose:

```
MyApp.dll
```

uses:

```
MyApp.Core.dll
```

The compiler needs to know what `Calculator` means.

When compiling:

```
var calculator = new Calculator();
```

the compiler resolves:

```
Calculator
    ↓
MyApp.Core.Calculator
    ↓
MyApp.Core.dll
```

The resulting assembly contains a reference to `MyApp.Core`.

Conceptually:

```
MyApp.dll
│
├── IL
│
├── Metadata
│
└── AssemblyRef
      │
      ▼
 MyApp.Core
```

This is one of the most important ideas:

> **A project/assembly reference allows the compiler to resolve types and members from another assembly.**

---

# 6. `using` is NOT the assembly reference

This is where beginners often get confused.

You write:

```
using MyApp.Core;
```

and might think:

```
using
  ↓
loads MyApp.Core.dll
```

No.

`using` mainly affects **name resolution**.

You could write:

```
var calculator = new MyApp.Core.Calculator();
```

without:

```
using MyApp.Core;
```

But you still need the assembly containing `MyApp.Core.Calculator` available as a compiler reference.

Think:

```
using
   ↓
"How can I write the name?"

assembly reference
   ↓
"Where is the type definition?"
```

---

# 7. Project references

Suppose your `.csproj` contains:

```
<ItemGroup>
    <ProjectReference Include="..\MyApp.Core\MyApp.Core.csproj" />
</ItemGroup>
```

This establishes a dependency:

```
MyApp
   │
   │ ProjectReference
   ▼
MyApp.Core
```

When building `MyApp`, MSBuild determines that:

```
MyApp.Core
```

must be built/available first.

Then Roslyn compiles `MyApp` against the resulting assembly/reference information.

---

# 8. What happens during compilation?

Let's make it concrete.

You have:

```
MyApp.Core
     ↓
MyApp
```

Build starts.

### Step 1 — Compile Core

```
Calculator.cs
     ↓
Roslyn
     ↓
MyApp.Core.dll
```

Conceptually:

```
MyApp.Core.dll
├── MyApp.Core.Calculator
├── IL
└── metadata
```

---

### Step 2 — Compile MyApp

Compiler sees:

```
using MyApp.Core;

var calculator = new Calculator();
```

It needs to answer:

> What is `Calculator`?

It searches its available references.

Eventually:

```
Calculator
    ↓
MyApp.Core.Calculator
    ↓
MyApp.Core.dll
```

Then it generates IL referencing that type.

# 9. The generated IL doesn't copy the Calculator class

This is important.

You might imagine:

```
MyApp.dll
└── copied Calculator class
```

That is generally **not** what happens.

Instead:

```
MyApp.dll
│
├── its own IL
│
└── references
       ↓
MyApp.Core.Calculator
```

So:

```
MyApp.dll
       │
       │ references
       ▼
MyApp.Core.dll
       │
       └── Calculator
```

The actual implementation remains in `MyApp.Core.dll`.

---

# 10. So where does "linking" happen?

This is where .NET differs significantly from C/C++.

In traditional native compilation, you might have:

```
source
 ↓
object files
 ↓
linker
 ↓
executable
```

The linker resolves symbols and combines native object code.

For .NET:

```
C#
 ↓
IL + metadata
 ↓
assembly
```

The CLR performs much of the equivalent **type/method resolution at runtime**.

Therefore, there isn't usually a traditional native linker that takes all referenced `.dll`s and physically merges their machine code into your executable.

Instead:

```
MyApp.dll
   │
   │ AssemblyRef
   ▼
MyApp.Core.dll
```

At runtime, the CLR loads and resolves the dependency.

---

# 11. Runtime loading

Suppose you execute:

```
dotnet MyApp.dll
```

The .NET runtime starts.

It loads the application's assembly:

```
MyApp.dll
```

It sees that it references:

```
MyApp.Core
```

The runtime then needs to locate the corresponding assembly.

Conceptually:

```
MyApp.dll
   │
   │ needs
   ▼
MyApp.Core
   │
   ▼
MyApp.Core.dll
```

Once loaded, the runtime can resolve:

```
MyApp.Core.Calculator
```

---

# 12. Then JIT enters the picture

Here's another important distinction.

The IL isn't usually executed directly by the CPU.

For example:

```
calculator.Add(10, 20);
```

produces IL.

When the runtime needs to execute a method, the **JIT compiler** can translate the IL into native machine code.

Conceptually:

```
IL:

callvirt MyApp.Core.Calculator::Add
              │
              ▼
             JIT
              │
              ▼
      native machine code
              │
              ▼
             CPU
```

So the full pipeline is approximately:

```
Calculator.cs
     │
     ▼
Roslyn
     │
     ▼
IL + metadata
     │
     ▼
MyApp.Core.dll
     │
     │ loaded by CLR
     ▼
JIT
     │
     ▼
x64 / ARM64 machine code
     │
     ▼
CPU
```

---

# 13. What happens with a method call?

Consider:

```
var calculator = new Calculator();

int result = calculator.Add(10, 20);
```

There are several layers.

### Source level

```
calculator.Add(10, 20)
```

### Semantic/compiler level

The compiler determines:

```
calculator
   ↓
MyApp.Core.Calculator

Add
   ↓
instance method
Add(Int32, Int32) : Int32
```

### IL level

Something conceptually similar to:

```
ldloc calculator
ldc.i4 10
ldc.i4 20
callvirt MyApp.Core.Calculator::Add
```

### Runtime level

CLR resolves:

```
MyApp.Core.Calculator
        +
Add(Int32, Int32)
```

### JIT level

The method is compiled into native code.

### CPU

CPU executes it.

---

# 14. Assembly references are metadata

Inside a .NET assembly, there are tables representing relationships.

Conceptually:

```
MyApp.dll
│
├── TypeDef
│     └── MyApp.Program
│
├── MethodDef
│     └── Main()
│
├── TypeRef
│     └── MyApp.Core.Calculator
│
├── MemberRef
│     └── Calculator.Add(...)
│
└── AssemblyRef
      └── MyApp.Core
```

This is why saying:

> "MyApp.dll links against MyApp.Core.dll"

is useful conceptually, but technically it's more accurate to say:

> **MyApp.dll contains metadata references to types/members defined by MyApp.Core.**

The CLR resolves those references when needed.

---

# 15. What if the referenced DLL isn't present?

Suppose:

```
MyApp.dll
MyApp.Core.dll
```

works.

Then someone deletes:

```
MyApp.Core.dll
```

and runs:

```
dotnet MyApp.dll
```

The runtime can't satisfy the dependency.

Depending on when the missing type/member is needed, you may get an assembly/type loading exception such as:

```
FileNotFoundException
TypeLoadException
MissingMethodException
```

The important point is:

```
Compilation succeeded
        ↓
Runtime dependency missing
        ↓
Runtime failure
```

This is why **compile-time reference resolution and runtime assembly resolution are distinct stages**.

---

# 16. What if the DLL exists but has the wrong version?

Imagine:

```
MyApp.dll
     ↓
expects MyApp.Core v1
```

but deployment contains:

```
MyApp.Core v2
```

Now assembly identity becomes important.

.NET assemblies can have identity information including:

```
Name
Version
Culture
Public key/token (for strong-named assemblies)
```

Modern .NET has assembly-loading behavior that differs substantially from the old .NET Framework, so don't assume all historical .NET assembly-binding rules still apply.

But the fundamental concept remains:

```
Assembly reference
       ↓
Assembly identity / resolution
       ↓
Actual loaded assembly
```

---

# 17. NuGet packages fit into this model

Suppose you install:

```
dotnet add package Newtonsoft.Json
```

Your project gets package metadata/dependencies, and the build can obtain the referenced assemblies.

Conceptually:

```
NuGet package
      │
      ▼
Newtonsoft.Json.dll
      │
      ▼
Project reference
      │
      ▼
YourAssembly.dll
```

Your source:

```
using Newtonsoft.Json;
```

is only the namespace/name convenience.

The actual dependency is an **assembly reference**.

---

# 18. One package ≠ one assembly

Another subtle point.

A NuGet package can contain:

```
Package
│
├── A.dll
├── B.dll
├── C.dll
└── dependencies...
```

And an assembly can contain many namespaces:

```
Some.dll
│
├── Company.Product.Users
├── Company.Product.Orders
├── Company.Product.Payments
└── Company.Product.Infrastructure
```

Therefore:

```
Namespace ≠ Type
Assembly ≠ Namespace
Package ≠ Assembly
Project ≠ Assembly
```

These distinctions become extremely important in real .NET systems.

---

# 19. Project → Assembly relationship

Typically:

```
.csproj
   │
   │ build
   ▼
assembly
```

For example:

```
MyApp.Core.csproj
        │
        ▼
MyApp.Core.dll
```

A project is primarily a **build-time concept**.

An assembly is primarily a **runtime/deployment concept**.

A project can technically produce different outputs depending on configuration/targeting, and modern .NET project structures can be more complex, but this is the useful mental model.

---

# 20. What `dotnet build` actually coordinates

At a high level:

```
dotnet build
      │
      ▼
    MSBuild
      │
      ├── Determine project dependencies
      │
      ├── Restore NuGet packages
      │
      ├── Build referenced projects
      │
      ├── Invoke Roslyn
      │
      ├── Produce assemblies
      │
      └── Copy required runtime dependencies
```

So there are actually several systems involved:

```
dotnet CLI
    ↓
MSBuild
    ↓
Roslyn compiler
    ↓
Assembly
    ↓
CLR
    ↓
JIT
```

Don't think of "the C# compiler" as doing everything.

---

# 21. Where namespaces fit into this

Now connect this with your previous question.

Suppose:

```
namespace MyCompany.Core;

public class Calculator
{
}
```

There are three different concepts here:

### Namespace

```
MyCompany.Core
```

Logical name.

### Type

```
MyCompany.Core.Calculator
```

Fully qualified type name.

### Assembly

```
MyCompany.Core.dll
```

Physical compiled unit containing that type.

So:

```
MyCompany.Core.Calculator
          │
          │ defined in
          ▼
MyCompany.Core.dll
```

The namespace does **not** imply the DLL.

# 22. A useful complete picture

Imagine this application:

```
                    SOURCE CODE
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
    Domain.csproj                Api.csproj
          │                           │
          │                           │
       Roslyn                      Roslyn
          │                           │
          ▼                           ▼
   Domain.dll  ◄──── reference ─── Api.dll
          │                           │
          │                           │
          ▼                           ▼
     IL + metadata              IL + metadata
          │                           │
          └─────────────┬─────────────┘
                        │
                        ▼
                    .NET CLR
                        │
                 Assembly Loader
                        │
                        ▼
                  Type Resolution
                        │
                        ▼
                       JIT
                        │
                        ▼
                Native Machine Code
                        │
                        ▼
                       CPU
```

That's the architecture you should keep in your head.

---

# 23. Compile-time vs runtime

This distinction is probably the most valuable takeaway.

|Stage|Responsibility|
|---|---|
|C# compiler|Parse/type-check C#|
|Roslyn|Generate IL + metadata|
|MSBuild|Coordinate projects/build|
|Assembly|Package IL + metadata + references|
|CLR|Load assemblies|
|CLR|Resolve types/members|
|JIT|Generate native code|
|CPU|Execute native code|

And:

```
COMPILE TIME
─────────────────────────────
Source
  ↓
Compiler
  ↓
Type checking
  ↓
Reference resolution
  ↓
Assembly
```

versus:

```
RUNTIME
─────────────────────────────
Assembly
  ↓
Assembly loading
  ↓
Type/member resolution
  ↓
JIT
  ↓
Native code
  ↓
CPU
```

---

# 24. The really important distinction: compile-time reference vs runtime dependency

Suppose:

```
using MyCompany.Core;

Calculator calculator = new();
```

There are actually several things happening:

```
using MyCompany.Core
        │
        ▼
Compiler name resolution

MyCompany.Core.Calculator
        │
        ▼
Compiler finds type in reference assembly

MyApp.dll
        │
        ▼
Contains metadata referring to MyCompany.Core

Runtime
        │
        ▼
Loads MyCompany.Core assembly

JIT
        │
        ▼
Compiles Calculator.Add()
```

So when someone says:

> "The application references the DLL"

you should mentally expand it into:

> "The build has a reference that lets the compiler resolve types from that assembly, and the resulting assembly contains metadata references that the runtime must resolve when those types/members are used."

That is much closer to how .NET actually works.

---

## One final mental model

If you remember only this:

```
                C# SOURCE
                    │
                    ▼
                 Roslyn
                    │
             ┌──────┴──────┐
             │             │
             ▼             ▼
            IL         METADATA
             │             │
             └──────┬──────┘
                    ▼
                ASSEMBLY
                 .dll
                    │
          Assembly References
                    │
                    ▼
             Other .dlls
                    │
                    ▼
              .NET Runtime
                    │
             Type Resolution
                    │
                    ▼
                   JIT
                    │
                    ▼
            Native Machine Code
                    │
                    ▼
                   CPU
```

And remember:

```
namespace  → naming/organization
project    → build-time unit
assembly   → compiled/runtime unit
reference  → dependency relationship
IL         → intermediate code
metadata   → description of types/members/references
CLR        → runtime
JIT        → IL → native code
```

Once this model is clear, the next layer is **how CLR assembly loading and type resolution actually work**—including `AssemblyLoadContext`, probing, `.deps.json`, `runtimeconfig.json`, shared frameworks, NuGet dependency graphs, and why `FileNotFoundException`, `TypeLoadException`, and `MissingMethodException` happen. That is where .NET dependency management gets genuinely interesting.