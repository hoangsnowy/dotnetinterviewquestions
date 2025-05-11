# C# Version History & Frequently Asked Questions

This guide covers the history of C# versions and common interview questions about C# language features and evolution.

## Table of Contents
1. [C# Version History](#1-c-version-history)
2. [Language Features by Version](#2-language-features-by-version)
3. [Common Interview Questions](#3-common-interview-questions)
4. [Best Practices & Migration](#4-best-practices--migration)

## 1. C# Version History

### Question
What is the history of C# versions and their key features?

### Suggested Answer
C# has evolved significantly since its initial release in 2002. Here's a timeline of major versions:

```csharp
// C# 1.0 (2002)
// Basic object-oriented features
public class Customer
{
    private string name;
    public string Name
    {
        get { return name; }
        set { name = value; }
    }
}

// C# 2.0 (2005)
// Generics, partial types, anonymous methods
public class List<T>
{
    public void Add(T item) { }
}

// C# 3.0 (2007)
// LINQ, lambda expressions, extension methods
var query = customers
    .Where(c => c.Age > 18)
    .OrderBy(c => c.Name)
    .Select(c => c.Email);

// C# 4.0 (2010)
// Dynamic binding, named arguments, optional parameters
public void ProcessOrder(string product, int quantity = 1, bool express = false)
{
    dynamic order = new ExpandoObject();
    order.Product = product;
    order.Quantity = quantity;
}

// C# 5.0 (2012)
// Async/await
public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);
    return "Data";
}

// C# 6.0 (2015)
// String interpolation, null-conditional operator
var message = $"{customer.Name} ordered {order.Quantity} items";
var name = customer?.Name ?? "Unknown";

// C# 7.0 (2017)
// Tuples, pattern matching
var (name, age) = GetPersonInfo();
if (obj is string s) { }

// C# 8.0 (2019)
// Nullable reference types, switch expressions
public string GetStatus(Order order) => order switch
{
    { Status: OrderStatus.Pending } => "Pending",
    { Status: OrderStatus.Processing } => "Processing",
    _ => "Unknown"
};

// C# 9.0 (2020)
// Records, init-only properties
public record Person(string Name, int Age);

// C# 10.0 (2021)
// Global using directives, file-scoped namespaces
global using System;
namespace MyApp;

// C# 11.0 (2022)
// Raw string literals, required members
public class Person
{
    public required string Name { get; init; }
}

// C# 12.0 (2023)
// Primary constructors, collection expressions
public class Customer(string name, string email)
{
    public string Name { get; } = name;
    public string Email { get; } = email;
}
```

### Follow-up Q&A

1. **Q: What was the most significant change in C# history?**
   - **A**: 
     - C# 2.0: Introduction of generics
     - C# 3.0: LINQ and functional programming features
     - C# 5.0: Async/await for asynchronous programming
     - C# 8.0: Nullable reference types for better null safety
     - C# 9.0: Records for immutable data types

2. **Q: How has C# evolved in terms of programming paradigms?**
   - **A**: 
     - Started as a pure object-oriented language
     - Added functional programming features (LINQ, lambda expressions)
     - Introduced declarative programming (pattern matching)
     - Enhanced immutability support (records, init-only)
     - Improved asynchronous programming (async/await)

## 2. Language Features by Version

### Question
What are the key features introduced in each C# version?

### Suggested Answer
Here's a comprehensive list of major features by version:

```csharp
// C# 1.0 (2002)
// - Basic OOP features
// - Properties
// - Events
// - Indexers
// - Attributes

// C# 2.0 (2005)
// - Generics
// - Partial types
// - Anonymous methods
// - Nullable types
// - Static classes

// C# 3.0 (2007)
// - LINQ
// - Lambda expressions
// - Extension methods
// - Anonymous types
// - Object initializers
// - Collection initializers
// - Implicit typing (var)

// C# 4.0 (2010)
// - Dynamic binding
// - Named arguments
// - Optional parameters
// - Generic covariance/contravariance
// - COM interop improvements

// C# 5.0 (2012)
// - Async/await
// - Caller info attributes
// - Breakpoints in lambda expressions

// C# 6.0 (2015)
// - String interpolation
// - Null-conditional operator
// - Expression-bodied members
// - Auto-property initializers
// - Dictionary initializers
// - Exception filters
// - nameof operator

// C# 7.0 (2017)
// - Tuples and deconstruction
// - Pattern matching
// - Local functions
// - Out variables
// - Literal improvements
// - Ref returns and locals
// - Expression-bodied members expansion

// C# 8.0 (2019)
// - Nullable reference types
// - Switch expressions
// - Pattern matching enhancements
// - Using declarations
// - Static local functions
// - Disposable ref structs
// - Nullable reference types
// - Asynchronous streams

// C# 9.0 (2020)
// - Records
// - Init-only properties
// - Top-level statements
// - Pattern matching improvements
// - Target-typed new expressions
// - Static anonymous functions
// - Target-typed conditional expressions
// - Covariant return types

// C# 10.0 (2021)
// - Global using directives
// - File-scoped namespaces
// - Record structs
// - Improvements to structs
// - Interpolated string improvements
// - Constant interpolated strings
// - Record types can seal ToString()
// - Assignment and declaration in the same deconstruction

// C# 11.0 (2022)
// - Raw string literals
// - Required members
// - Generic attributes
// - File-local types
// - List patterns
// - Extended nameof scope
// - Numeric IntPtr
// - Auto-default structs
// - Pattern match on Span<char>
// - Extended fixed statement

// C# 12.0 (2023)
// - Primary constructors
// - Collection expressions
// - Default lambda parameters
// - Alias any type
// - Experimental attribute
// - Interceptors
```

### Follow-up Q&A

1. **Q: What are the most important features for modern C# development?**
   - **A**: 
     - Nullable reference types (C# 8.0)
     - Records and init-only properties (C# 9.0)
     - Pattern matching (C# 7.0+)
     - Async/await (C# 5.0)
     - LINQ (C# 3.0)

2. **Q: How do you choose which C# version to use?**
   - **A**: 
     - Consider target framework compatibility
     - Evaluate required language features
     - Check team's familiarity with features
     - Consider long-term maintenance
     - Review performance implications

## 3. Common Interview Questions

### Question
What are common interview questions about C# versions and features?

### Suggested Answer
Here are some frequently asked questions in C# interviews:

```csharp
// 1. What's the difference between C# and .NET?
// .NET is the framework/platform, while C# is the language
// C# code compiles to IL (Intermediate Language)
// .NET provides the runtime and libraries

// 2. What are the main differences between C# and Java?
// - C# has properties, Java uses getters/setters
// - C# has value types (structs), Java has only reference types
// - C# has async/await, Java uses CompletableFuture
// - C# has extension methods, Java doesn't
// - C# has nullable reference types, Java has Optional

// 3. What's the difference between var and dynamic?
// var is compile-time type inference
// dynamic is runtime type binding
var name = "John"; // Compile-time type: string
dynamic value = GetValue(); // Runtime type checking

// 4. What's the difference between async/await and Task.Run?
// async/await is for I/O-bound operations
// Task.Run is for CPU-bound operations
public async Task<string> GetDataAsync()
{
    // I/O-bound: Use async/await
    return await File.ReadAllTextAsync("data.txt");
}

public Task<string> ProcessDataAsync()
{
    // CPU-bound: Use Task.Run
    return Task.Run(() => HeavyComputation());
}

// 5. What's the difference between struct and class?
// struct is a value type, class is a reference type
// struct is stored on the stack, class on the heap
// struct is passed by value, class by reference
// struct can't inherit, class can
// struct can't be null (unless nullable), class can

// 6. What's the difference between interface and abstract class?
// interface can't have implementation
// abstract class can have implementation
// class can implement multiple interfaces
// class can inherit only one abstract class
// interface members are public by default
// abstract class can have access modifiers

// 7. What's the difference between IEnumerable and IQueryable?
// IEnumerable is for in-memory collections
// IQueryable is for database queries
// IEnumerable executes in memory
// IQueryable translates to SQL
var inMemory = customers.Where(c => c.Age > 18); // IEnumerable
var database = db.Customers.Where(c => c.Age > 18); // IQueryable

// 8. What's the difference between throw and throw ex?
// throw preserves the stack trace
// throw ex resets the stack trace
try
{
    // Some code
}
catch (Exception ex)
{
    // throw; // Preserves stack trace
    // throw ex; // Resets stack trace
}

// 9. What's the difference between String and StringBuilder?
// String is immutable
// StringBuilder is mutable
// String is better for few concatenations
// StringBuilder is better for many concatenations
string s = "Hello" + " " + "World"; // Creates new strings
StringBuilder sb = new StringBuilder();
sb.Append("Hello").Append(" ").Append("World"); // Modifies existing

// 10. What's the difference between == and Equals?
// == is reference equality for reference types
// == is value equality for value types
// Equals is virtual and can be overridden
// Equals is value equality by default
string a = "Hello";
string b = "Hello";
Console.WriteLine(a == b); // True (string interning)
Console.WriteLine(a.Equals(b)); // True
```

### Follow-up Q&A

1. **Q: What are the most important C# concepts to know for interviews?**
   - **A**: 
     - Object-oriented programming
     - Generics and collections
     - LINQ and lambda expressions
     - Async/await and threading
     - Memory management
     - Design patterns
     - SOLID principles
     - Unit testing

2. **Q: How do you stay updated with C# features?**
   - **A**: 
     - Follow Microsoft's C# blog
     - Read release notes
     - Participate in C# community
     - Practice with new features
     - Review code samples
     - Attend conferences
     - Follow C# experts on social media

## 4. Best Practices & Migration

### Question
What are the best practices for using modern C# features and migrating legacy code?

### Suggested Answer
Here are some best practices and migration strategies:

```csharp
// 1. Use modern C# features appropriately
// Use records for DTOs and value objects
public record CustomerDto(string Name, string Email);

// Use init-only properties for immutability
public class Order
{
    public int Id { get; init; }
    public string CustomerName { get; init; } = string.Empty;
}

// Use pattern matching for type checking
public string ProcessOrder(Order order) => order switch
{
    { Status: OrderStatus.Pending } => "Pending",
    { Status: OrderStatus.Processing } => "Processing",
    _ => "Unknown"
};

// 2. Migrate legacy code gradually
// Step 1: Enable nullable reference types
#nullable enable

// Step 2: Update properties to init-only
public class LegacyCustomer
{
    public string Name { get; init; } = string.Empty;
    public string Email { get; init; } = string.Empty;
}

// Step 3: Use modern syntax
public class ModernCustomer
{
    public required string Name { get; init; }
    public required string Email { get; init; }
}

// 3. Follow best practices
// Use expression-bodied members
public string GetFullName() => $"{FirstName} {LastName}";

// Use null-conditional operator
public string GetCustomerName() => customer?.Name ?? "Unknown";

// Use string interpolation
public string GetMessage() => $"Hello, {Name}!";

// Use pattern matching
public bool IsValidCustomer(Customer? customer) =>
    customer is { Name: not null, Email: not null };

// 4. Performance considerations
// Use Span<T> for high-performance scenarios
public void ProcessData(Span<byte> data)
{
    // Process data without allocations
}

// Use ValueTask for async methods
public ValueTask<string> GetDataAsync()
{
    return ValueTask.FromResult("Data");
}

// 5. Testing considerations
// Use records for test data
public record TestData(string Name, int Value);

// Use init-only properties for test setup
public class TestSetup
{
    public string Name { get; init; } = string.Empty;
    public int Value { get; init; }
}
```

### Follow-up Q&A

1. **Q: How do you handle breaking changes when upgrading C# versions?**
   - **A**: 
     - Review release notes
     - Use compiler warnings
     - Update code gradually
     - Use feature flags
     - Maintain backward compatibility
     - Test thoroughly
     - Document changes

2. **Q: What are the common pitfalls when using modern C# features?**
   - **A**: 
     - Overusing nullable reference types
     - Misusing records
     - Incorrect async/await usage
     - Performance issues with LINQ
     - Memory leaks with events
     - Thread safety issues
     - Incorrect exception handling

## Additional Resources

- [Microsoft Docs: C# Version History](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/)
- [Microsoft Docs: C# Language Reference](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/)
- [Microsoft Docs: C# Programming Guide](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/)
- [Microsoft Docs: C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions) 