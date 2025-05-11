# C# Fundamentals

This guide covers fundamental C# concepts with practical examples and interview questions.

## Table of Contents
1. [Generics & Constraints](#1-generics--constraints)
2. [Delegates & Events](#2-delegates--events)
3. [LINQ & Lambda Expressions](#3-linq--lambda-expressions)
4. [Tuples & Deconstruction](#4-tuples--deconstruction)
5. [Indexers & Extension Methods](#5-indexers--extension-methods)
6. [Nullable Reference Types](#6-nullable-reference-types)

## 1. Generics & Constraints

### Question / Câu hỏi
What are generics in C# and how do you use them effectively?

### Suggested Answer / Câu trả lời gợi ý
Generics allow you to write type-safe code that can work with different data types. They provide type safety at compile time and eliminate the need for type casting.

```csharp
// Generic class example
public class Repository<T> where T : class
{
    private readonly List<T> _items = new();

    public void Add(T item)
    {
        _items.Add(item);
    }

    public T GetById(int id)
    {
        return _items.FirstOrDefault(x => x.Id == id);
    }
}

// Generic method example
public static class Helper
{
    public static T Max<T>(T a, T b) where T : IComparable<T>
    {
        return a.CompareTo(b) > 0 ? a : b;
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: What are the different types of constraints in C# generics?**
   - **A**: 
     - `where T : class` - Reference type
     - `where T : struct` - Value type
     - `where T : new()` - Has default constructor
     - `where T : BaseClass` - Inherits from base class
     - `where T : IInterface` - Implements interface

2. **Q: When should you use generic methods vs generic classes?**
   - **A**: Use generic methods when only specific methods need type parameters, and generic classes when the entire class needs to work with a type parameter.

## 2. Delegates & Events

### Question / Câu hỏi
Explain delegates and events in C# with examples.

### Suggested Answer / Câu trả lời gợi ý
Delegates are type-safe function pointers that can reference methods with matching signatures. Events are a way to provide notifications when something happens.

```csharp
// Delegate definition
public delegate void OrderProcessedHandler(Order order);

// Event publisher
public class OrderProcessor
{
    // Event declaration
    public event OrderProcessedHandler OrderProcessed;

    public void ProcessOrder(Order order)
    {
        // Process order logic
        OnOrderProcessed(order);
    }

    protected virtual void OnOrderProcessed(Order order)
    {
        OrderProcessed?.Invoke(order);
    }
}

// Event subscriber
public class OrderNotifier
{
    public void Subscribe(OrderProcessor processor)
    {
        processor.OrderProcessed += HandleOrderProcessed;
    }

    private void HandleOrderProcessed(Order order)
    {
        Console.WriteLine($"Order {order.Id} processed");
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: What's the difference between delegates and events?**
   - **A**: Events are a special kind of delegate that can only be invoked by the declaring class, providing better encapsulation.

2. **Q: How do you handle multiple subscribers to an event?**
   - **A**: Events can have multiple subscribers, and they are called in the order they were added using the += operator.

## 3. LINQ & Lambda Expressions

### Question / Câu hỏi
How do you use LINQ and lambda expressions in C#?

### Suggested Answer / Câu trả lời gợi ý
LINQ (Language Integrated Query) provides a way to query data using a SQL-like syntax. Lambda expressions are anonymous functions that can be used to create delegates or expression trees.

```csharp
public class ProductService
{
    private readonly List<Product> _products;

    public IEnumerable<Product> GetExpensiveProducts()
    {
        // LINQ query syntax
        var query = from p in _products
                   where p.Price > 100
                   orderby p.Price descending
                   select p;

        // LINQ method syntax with lambda
        return _products
            .Where(p => p.Price > 100)
            .OrderByDescending(p => p.Price)
            .ToList();
    }

    public decimal GetTotalValue()
    {
        // Lambda expression with aggregation
        return _products.Sum(p => p.Price * p.Quantity);
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: What's the difference between query and method syntax in LINQ?**
   - **A**: Query syntax is more readable for complex queries, while method syntax is more concise and flexible.

2. **Q: When should you use deferred execution in LINQ?**
   - **A**: Use deferred execution when you want to optimize performance by not executing the query until the results are actually needed.

## 4. Tuples & Deconstruction

### Question / Câu hỏi
How do you use tuples and deconstruction in C#?

### Suggested Answer / Câu trả lời gợi ý
Tuples allow you to group multiple values together, and deconstruction lets you extract those values into separate variables.

```csharp
public class OrderService
{
    // Tuple return type
    public (bool success, string message) ProcessOrder(Order order)
    {
        try
        {
            // Process order
            return (true, "Order processed successfully");
        }
        catch (Exception ex)
        {
            return (false, ex.Message);
        }
    }

    public void HandleOrder()
    {
        // Deconstruction
        var (success, message) = ProcessOrder(new Order());
        
        // Using tuple directly
        var result = ProcessOrder(new Order());
        if (result.success)
        {
            Console.WriteLine(result.message);
        }
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: When should you use tuples vs custom classes?**
   - **A**: Use tuples for temporary grouping of values, and custom classes for more complex data structures that need to be reused.

2. **Q: How do you name tuple elements?**
   - **A**: You can name tuple elements using the syntax `(string name, int age)` or by using the `ValueTuple.Create` method.

## 5. Indexers & Extension Methods

### Question / Câu hỏi
Explain indexers and extension methods in C#.

### Suggested Answer / Câu trả lời gợi ý
Indexers allow objects to be indexed like arrays, and extension methods let you add methods to existing types without modifying them.

```csharp
// Indexer example
public class CustomCollection<T>
{
    private readonly List<T> _items = new();

    public T this[int index]
    {
        get => _items[index];
        set => _items[index] = value;
    }
}

// Extension method example
public static class StringExtensions
{
    public static bool IsValidEmail(this string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

// Usage
public class Program
{
    public void Example()
    {
        var collection = new CustomCollection<string>();
        collection[0] = "Hello";

        string email = "test@example.com";
        bool isValid = email.IsValidEmail();
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: What are the limitations of extension methods?**
   - **A**: Extension methods can't access private members of the extended type and can't override existing methods.

2. **Q: When should you use indexers?**
   - **A**: Use indexers when your class represents a collection of items and you want to provide array-like access to its elements.

## 6. Nullable Reference Types

### Question / Câu hỏi
How do you use nullable reference types in C#?

### Suggested Answer / Câu trả lời gợi ý
Nullable reference types help prevent null reference exceptions by making the nullability of reference types explicit.

```csharp
#nullable enable

public class UserService
{
    // Non-nullable reference type
    public string GetUserName(User user)
    {
        // Compiler warning if user.Name might be null
        return user.Name;
    }

    // Nullable reference type
    public string? GetOptionalName(User user)
    {
        return user.OptionalName;
    }

    // Null-forgiving operator
    public string GetNameOrDefault(User user)
    {
        return user.Name!; // Assert that Name is not null
    }
}

#nullable disable
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: What's the difference between nullable reference types and nullable value types?**
   - **A**: Nullable reference types are a compile-time feature, while nullable value types (int?, etc.) are actual types that can hold null.

2. **Q: How do you handle nullable reference types in legacy code?**
   - **A**: You can gradually enable nullable reference types by using the #nullable directive and fixing warnings as you go.

### Additional Resources
- [C# Generics](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/)
- [C# Delegates](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [C# LINQ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)
- [C# Tuples](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
- [C# Extension Methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)
- [C# Nullable Reference Types](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references) 