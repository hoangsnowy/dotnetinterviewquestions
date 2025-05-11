# Core C# Concepts

This guide covers essential C# core concepts with practical examples and interview questions.

## Table of Contents
1. [Generics & Constraints](#1-generics--constraints)
2. [Delegates, Events & Lambdas](#2-delegates-events--lambdas)
3. [LINQ](#3-linq)
4. [Tuples & Deconstruction](#4-tuples--deconstruction)
5. [Extension Methods & Indexers](#5-extension-methods--indexers)
6. [Nullable Reference Types](#6-nullable-reference-types)
7. [Custom Exceptions & IDisposable](#7-custom-exceptions--idisposable)
8. [Async/Await Nuances](#8-asyncawait-nuances)
9. [Reflection & Attributes](#9-reflection--attributes)

## 1. Generics & Constraints

### Question
How do you use generics and constraints in C# to create type-safe, reusable code?

### Suggested Answer
Generics allow you to write type-safe code that can work with different data types. Constraints help ensure type safety and provide access to specific members.

```csharp
using System;
using System.Collections.Generic;

public class Repository<T> where T : class, IEntity, new()
{
    private readonly List<T> _items = new List<T>();

    public T GetById(int id)
    {
        return _items.Find(item => item.Id == id);
    }

    public void Add(T item)
    {
        if (item == null)
            throw new ArgumentNullException(nameof(item));
        _items.Add(item);
    }
}

public interface IEntity
{
    int Id { get; set; }
}

// Usage
public class Product : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; }
}

var productRepo = new Repository<Product>();
```

### Follow-up Q&A

1. **Q: What are the different types of constraints in C#?**
   - **A**: Class/struct constraints, interface constraints, constructor constraints, and reference/value type constraints.

2. **Q: When should you use generic constraints?**
   - **A**: Use constraints when you need to ensure type safety or access specific members of the generic type.

## 2. Delegates, Events & Lambdas

### Question
How do you use delegates, events, and lambda expressions in C#?

### Suggested Answer
Delegates are type-safe function pointers, events are a way to implement the observer pattern, and lambdas provide a concise way to write anonymous methods.

```csharp
using System;

public class OrderProcessor
{
    // Delegate definition
    public delegate void OrderProcessedHandler(Order order);

    // Event using the delegate
    public event OrderProcessedHandler OrderProcessed;

    // Lambda expression
    private readonly Func<Order, bool> _validateOrder = order => 
        order != null && order.TotalAmount > 0;

    public void ProcessOrder(Order order)
    {
        if (_validateOrder(order))
        {
            // Process order
            OrderProcessed?.Invoke(order);
        }
    }
}

// Usage
var processor = new OrderProcessor();
processor.OrderProcessed += order => Console.WriteLine($"Order {order.Id} processed");
```

### Follow-up Q&A

1. **Q: What's the difference between delegates and events?**
   - **A**: Events are a wrapper around delegates that provide encapsulation and prevent external invocation.

2. **Q: When should you use lambda expressions?**
   - **A**: Use lambdas for short, one-off functions, especially with LINQ and event handlers.

## 3. LINQ

### Question
How do you use LINQ for complex data operations in C#?

### Suggested Answer
LINQ provides a declarative way to query and transform data using method syntax or query syntax.

```csharp
using System;
using System.Linq;
using System.Collections.Generic;

public class OrderService
{
    public IEnumerable<OrderSummary> GetOrderSummaries(IEnumerable<Order> orders)
    {
        return orders
            .GroupBy(o => o.CustomerId)
            .Select(g => new OrderSummary
            {
                CustomerId = g.Key,
                TotalOrders = g.Count(),
                TotalAmount = g.Sum(o => o.TotalAmount),
                LastOrderDate = g.Max(o => o.OrderDate)
            });
    }

    public IEnumerable<Order> GetRecentOrders(IEnumerable<Order> orders, DateTime cutoff)
    {
        return from order in orders
               where order.OrderDate >= cutoff
               orderby order.OrderDate descending
               select order;
    }

    // Example of joining collections
    public IEnumerable<OrderDetail> GetOrderDetails(IEnumerable<Order> orders, IEnumerable<Customer> customers)
    {
        var query = from order in orders
                   join customer in customers on order.CustomerId equals customer.Id
                   where order.TotalAmount > 100
                   select new OrderDetail
                   {
                       OrderId = order.Id,
                       CustomerName = customer.Name,
                       OrderDate = order.OrderDate,
                       TotalAmount = order.TotalAmount
                   };

        return query;
    }

    // Example of complex grouping
    public IEnumerable<CustomerOrderSummary> GetCustomerOrderSummaries(
        IEnumerable<Order> orders, 
        IEnumerable<OrderItem> items)
    {
        return from order in orders
               join item in items on order.Id equals item.OrderId
               group new { order, item } by order.CustomerId into g
               select new CustomerOrderSummary
               {
                   CustomerId = g.Key,
                   TotalOrders = g.Select(x => x.order.Id).Distinct().Count(),
                   TotalItems = g.Sum(x => x.item.Quantity),
                   TotalAmount = g.Sum(x => x.item.Price * x.item.Quantity)
               };
    }
}
```

### Follow-up Q&A

1. **Q: What's the difference between method syntax and query syntax?**
   - **A**: Method syntax is more concise and can be chained, while query syntax is more readable for complex queries.

2. **Q: How do you handle performance with LINQ?**
   - **A**: Use appropriate methods (First vs FirstOrDefault), consider deferred execution, and use ToList() when needed.

## 4. Tuples & Deconstruction

### Question
How do you use tuples and deconstruction in C# for returning multiple values?

### Suggested Answer
Tuples provide a way to return multiple values without creating a custom type, and deconstruction allows easy extraction of tuple values.

```csharp
using System;

public class OrderProcessor
{
    public (bool IsValid, string ErrorMessage) ValidateOrder(Order order)
    {
        if (order == null)
            return (false, "Order cannot be null");

        if (order.TotalAmount <= 0)
            return (false, "Order total must be greater than zero");

        if (string.IsNullOrEmpty(order.CustomerName))
            return (false, "Customer name is required");

        return (true, string.Empty);
    }

    public void ProcessOrder(Order order)
    {
        var (isValid, errorMessage) = ValidateOrder(order);
        if (!isValid)
        {
            throw new OrderValidationException(errorMessage);
        }

        // Business logic for processing valid order
        Console.WriteLine($"Processing order {order.Id} for {order.CustomerName}");
    }

    // Example of tuple pattern matching
    public string GetOrderStatus(Order order)
    {
        var status = (order.TotalAmount > 1000, order.IsPaid, order.IsShipped);
        return status switch
        {
            (true, true, true) => "Premium order shipped",
            (true, true, false) => "Premium order ready to ship",
            (true, false, _) => "Premium order pending payment",
            (false, true, true) => "Standard order shipped",
            (false, true, false) => "Standard order ready to ship",
            (false, false, _) => "Standard order pending payment"
        };
    }
}
```

### Follow-up Q&A

1. **Q: When should you use tuples vs custom types?**
   - **A**: Use tuples for temporary, internal return values; use custom types for public APIs and complex data.

2. **Q: How do you name tuple elements?**
   - **A**: Use named tuples for better readability: `(string FirstName, string LastName) GetName()`

## 5. Extension Methods & Indexers

### Question
How do you use extension methods and indexers to enhance existing types in C#?

### Suggested Answer
Extension methods allow you to add functionality to existing types, and indexers provide a way to access elements using array-like syntax.

```csharp
using System;
using System.Collections.Generic;

public static class StringExtensions
{
    public static bool IsPalindrome(this string str)
    {
        if (string.IsNullOrEmpty(str)) return false;
        return str.SequenceEqual(str.Reverse());
    }
}

public class CustomCollection<T>
{
    private readonly List<T> _items = new List<T>();

    public T this[int index]
    {
        get => _items[index];
        set => _items[index] = value;
    }

    public T this[string key]
    {
        get => _items.FirstOrDefault(item => item.ToString() == key);
    }
}
```

### Follow-up Q&A

1. **Q: What are the limitations of extension methods?**
   - **A**: They can't access private members and must be in a static class.

2. **Q: When should you use indexers?**
   - **A**: Use indexers when you want to provide array-like access to your collection or class.

## 6. Nullable Reference Types

### Question
How do you use nullable reference types to prevent null reference exceptions?

### Suggested Answer
Nullable reference types help catch potential null reference issues at compile time.

```csharp
#nullable enable

public class Customer
{
    public string Name { get; set; } = string.Empty; // Non-nullable
    public string? Email { get; set; } // Nullable

    public void UpdateEmail(string? newEmail)
    {
        if (newEmail != null)
        {
            Email = newEmail;
        }
    }
}

#nullable disable
```

### Follow-up Q&A

1. **Q: How do you handle nullable reference types in existing code?**
   - **A**: Gradually enable nullable reference types and use the null-forgiving operator (!) when necessary.

2. **Q: What's the difference between nullable reference types and nullable value types?**
   - **A**: Nullable reference types are a compile-time feature, while nullable value types are a runtime feature.

## 7. Custom Exceptions & IDisposable

### Question
How do you create custom exceptions and implement the IDisposable pattern?

### Suggested Answer
Custom exceptions should inherit from Exception, and IDisposable should be implemented using the dispose pattern.

```csharp
using System;

public class OrderProcessingException : Exception
{
    public int OrderId { get; }

    public OrderProcessingException(int orderId, string message) 
        : base(message)
    {
        OrderId = orderId;
    }
}

public class ResourceManager : IDisposable
{
    private bool _disposed;
    private readonly UnmanagedResource _resource;

    public ResourceManager()
    {
        _resource = new UnmanagedResource();
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                _resource.Dispose();
            }
            _disposed = true;
        }
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
}
```

### Follow-up Q&A

1. **Q: When should you create custom exceptions?**
   - **A**: Create custom exceptions when you need to provide specific error information or handle specific error cases.

2. **Q: Why use the dispose pattern?**
   - **A**: The dispose pattern ensures proper cleanup of both managed and unmanaged resources.

## 8. Async/Await Nuances

### Question
How do you handle advanced async/await scenarios in C#?

### Suggested Answer
Advanced async/await usage includes proper cancellation, ConfigureAwait, and exception handling.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class AsyncProcessor
{
    public async Task ProcessDataAsync(
        IEnumerable<Data> data,
        CancellationToken cancellationToken = default)
    {
        try
        {
            await foreach (var item in data.ToAsyncEnumerable()
                .WithCancellation(cancellationToken))
            {
                await ProcessItemAsync(item).ConfigureAwait(false);
            }
        }
        catch (OperationCanceledException)
        {
            // Handle cancellation
        }
    }

    private async Task ProcessItemAsync(Data item)
    {
        using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(30));
        await Task.WhenAny(
            ProcessItemCoreAsync(item),
            Task.Delay(Timeout.Infinite, cts.Token)
        );
    }
}
```

### Follow-up Q&A

1. **Q: When should you use ConfigureAwait(false)?**
   - **A**: Use ConfigureAwait(false) in library code to prevent context switching and potential deadlocks.

2. **Q: How do you handle cancellation in async methods?**
   - **A**: Use CancellationToken and handle OperationCanceledException appropriately.

## 9. Reflection & Attributes

### Question
How do you use reflection and custom attributes in C#?

### Suggested Answer
Reflection allows runtime type inspection, and attributes provide metadata for types and members.

```csharp
using System;
using System.Reflection;

[AttributeUsage(AttributeTargets.Property)]
public class RequiredAttribute : Attribute { }

public class ValidationAttribute : Attribute
{
    public string Message { get; }
    public ValidationAttribute(string message) => Message = message;
}

public class Validator
{
    public bool Validate(object obj)
    {
        var type = obj.GetType();
        foreach (var prop in type.GetProperties())
        {
            if (prop.GetCustomAttribute<RequiredAttribute>() != null)
            {
                var value = prop.GetValue(obj);
                if (value == null)
                {
                    return false;
                }
            }
        }
        return true;
    }
}
```

### Follow-up Q&A

1. **Q: When should you use reflection?**
   - **A**: Use reflection for dynamic type inspection, serialization, or when you need to work with types at runtime.

2. **Q: What are the performance implications of reflection?**
   - **A**: Reflection is slower than direct access, so use it judiciously and consider caching reflection results.

## Additional Resources

- [Microsoft Docs: Generics](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/)
- [Microsoft Docs: Delegates](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Microsoft Docs: LINQ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)
- [Microsoft Docs: Tuples](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
- [Microsoft Docs: Extension Methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)
- [Microsoft Docs: Nullable Reference Types](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references)
- [Microsoft Docs: IDisposable](https://docs.microsoft.com/en-us/dotnet/api/system.idisposable)
- [Microsoft Docs: Async/Await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)
- [Microsoft Docs: Reflection](https://docs.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection) 