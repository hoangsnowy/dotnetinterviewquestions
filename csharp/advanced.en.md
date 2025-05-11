# Advanced C# Concepts

This guide covers advanced C# concepts with practical examples and interview questions.

## Table of Contents
1. [Async/Await & Task Parallel Library](#1-asyncawait--task-parallel-library)
2. [Memory Management & Span<T> / Memory<T>](#2-memory-management--spant--memoryt)
3. [Pattern Matching](#3-pattern-matching)
4. [Expression Trees & Source Generators](#4-expression-trees--source-generators)
5. [Threading & Synchronization](#5-threading--synchronization)
6. [Unsafe Code & Pointers](#6-unsafe-code--pointers)
7. [Advanced Reflection & Custom Attributes](#7-advanced-reflection--custom-attributes)
8. [Preprocessor Directives](#8-preprocessor-directives)

## 1. Async/Await & Task Parallel Library

### Question / Câu hỏi
How do you handle asynchronous operations in C# using async/await and TPL?

### Suggested Answer / Câu trả lời gợi ý
Async/await and TPL provide a way to write asynchronous code that's easy to read and maintain. They help prevent blocking operations and improve application responsiveness.

```csharp
public class PaymentProcessor
{
    private readonly IPaymentGateway _gateway;
    private readonly ILogger<PaymentProcessor> _logger;

    public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request, CancellationToken cancellationToken)
    {
        try
        {
            // Validate payment
            await ValidatePaymentAsync(request, cancellationToken);

            // Process payment in parallel
            var tasks = new[]
            {
                _gateway.ProcessPaymentAsync(request, cancellationToken),
                _gateway.ValidateFraudAsync(request, cancellationToken),
                _gateway.CheckBalanceAsync(request, cancellationToken)
            };

            await Task.WhenAll(tasks);

            return new PaymentResult { Success = true };
        }
        catch (OperationCanceledException)
        {
            _logger.LogWarning("Payment processing was cancelled");
            throw;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error processing payment");
            throw;
        }
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: How do you prevent deadlocks in async code?**
   - **A**: Use ConfigureAwait(false) in libraries and avoid blocking on async code.

2. **Q: When should you use Task.Run vs async/await?**
   - **A**: Use Task.Run for CPU-bound operations and async/await for I/O-bound operations.

## 2. Memory Management & Span<T> / Memory<T>

### Question / Câu hỏi
How do you optimize memory usage with Span<T> and Memory<T> in C#?

### Suggested Answer / Câu trả lời gợi ý
Span<T> and Memory<T> provide efficient ways to work with memory without allocations, improving performance for high-throughput scenarios.

```csharp
public class FileProcessor
{
    public async Task ProcessFileAsync(string filePath)
    {
        using var file = File.OpenRead(filePath);
        var buffer = new byte[8192];
        var memory = new Memory<byte>(buffer);

        while (true)
        {
            var bytesRead = await file.ReadAsync(memory);
            if (bytesRead == 0) break;

            ProcessBuffer(memory.Slice(0, bytesRead));
        }
    }

    private void ProcessBuffer(ReadOnlySpan<byte> buffer)
    {
        // Process buffer without allocations
        for (int i = 0; i < buffer.Length; i++)
        {
            // Process each byte
        }
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: What's the difference between Span<T> and Memory<T>?**
   - **A**: Span<T> is stack-only and can't be stored in fields, while Memory<T> can be stored and used across async boundaries.

2. **Q: When should you use stackalloc?**
   - **A**: Use stackalloc for small, temporary buffers that don't need to live beyond the current method.

## 3. Pattern Matching

### Question / Câu hỏi
How do you use pattern matching in C# for complex type checking and data extraction?

### Suggested Answer / Câu trả lời gợi ý
Pattern matching provides a concise way to check types and extract data using switch expressions and property patterns.

```csharp
public class OrderProcessor
{
    public string GetOrderStatus(Order order) => order switch
    {
        { Status: OrderStatus.Pending } => "Order is pending",
        { Status: OrderStatus.Processing, PaymentStatus: PaymentStatus.Paid } => "Order is being processed",
        { Status: OrderStatus.Shipped, TrackingNumber: not null } => $"Order is shipped: {order.TrackingNumber}",
        { Status: OrderStatus.Delivered } => "Order is delivered",
        _ => "Unknown order status"
    };

    public decimal CalculateDiscount(Order order) => (order.CustomerType, order.TotalAmount) switch
    {
        (CustomerType.Premium, var amount) when amount > 1000 => amount * 0.1m,
        (CustomerType.Regular, var amount) when amount > 500 => amount * 0.05m,
        _ => 0m
    };
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: What's the difference between switch statement and switch expression?**
   - **A**: Switch expression is more concise and can return a value, while switch statement is more traditional and can't return values.

2. **Q: How do you handle complex pattern matching?**
   - **A**: Use nested patterns and proper syntax to handle complex matching scenarios.

## 4. Expression Trees & Source Generators

### Question / Câu hỏi
How do you use expression trees and source generators in C#?

### Suggested Answer / Câu trả lời gợi ý
Expression trees allow you to represent code as data structures, and source generators let you generate code at compile time.

```csharp
// Expression tree example
public class CustomQueryProvider : IQueryProvider
{
    public IQueryable<TElement> CreateQuery<TElement>(Expression expression)
    {
        return new CustomQuery<TElement>(this, expression);
    }

    public object Execute(Expression expression)
    {
        var sql = TranslateExpression(expression);
        return ExecuteQuery(sql);
    }

    private string TranslateExpression(Expression expression)
    {
        return expression switch
        {
            MethodCallExpression m => TranslateMethodCall(m),
            BinaryExpression b => TranslateBinary(b),
            MemberExpression m => TranslateMember(m),
            _ => throw new NotSupportedException()
        };
    }
}

// Source generator example
[Generator]
public class CustomGenerator : ISourceGenerator
{
    public void Initialize(GeneratorInitializationContext context)
    {
        // Initialize generator
    }

    public void Execute(GeneratorExecutionContext context)
    {
        // Generate code
        var sourceBuilder = new StringBuilder();
        sourceBuilder.AppendLine("// Generated code");
        context.AddSource("Generated.cs", sourceBuilder.ToString());
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: When should you use expression trees?**
   - **A**: Use expression trees when you need to analyze or transform code at runtime, such as in LINQ providers.

2. **Q: What are the benefits of source generators?**
   - **A**: Source generators can improve performance by generating code at compile time and reduce runtime overhead.

## 5. Threading & Synchronization

### Question / Câu hỏi
How do you handle concurrent operations and synchronization in C#?

### Suggested Answer / Câu trả lời gợi ý
C# provides various synchronization primitives and patterns for handling concurrent operations safely.

```csharp
public class OrderProcessor
{
    private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(10);
    private readonly object _lock = new object();

    public async Task ProcessOrdersAsync(IEnumerable<Order> orders)
    {
        var tasks = orders.Select(async order =>
        {
            await _semaphore.WaitAsync();
            try
            {
                await ProcessOrderAsync(order);
            }
            finally
            {
                _semaphore.Release();
            }
        });

        await Task.WhenAll(tasks);
    }

    public void UpdateInventory(IEnumerable<Product> products)
    {
        Parallel.ForEach(products, product =>
        {
            lock (_lock)
            {
                // Update inventory
            }
        });
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: How do you prevent deadlocks in multi-threaded code?**
   - **A**: Use proper locking order and timeouts to prevent deadlocks.

2. **Q: When should you use Parallel.ForEach vs async/await?**
   - **A**: Use Parallel.ForEach for CPU-bound operations and async/await for I/O-bound operations.

## 6. Unsafe Code & Pointers

### Question / Câu hỏi
How do you use unsafe code and pointers in C#?

### Suggested Answer / Câu trả lời gợi ý
Unsafe code and pointers allow direct memory manipulation, which can be useful for performance-critical scenarios.

```csharp
public unsafe class BufferProcessor
{
    public void ProcessBuffer(byte[] buffer)
    {
        fixed (byte* ptr = buffer)
        {
            for (int i = 0; i < buffer.Length; i++)
            {
                *(ptr + i) = (byte)(*(ptr + i) ^ 0xFF);
            }
        }
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: When should you use unsafe code?**
   - **A**: Use unsafe code only when necessary for performance or interop with native code.

2. **Q: How do you handle pointer arithmetic safely?**
   - **A**: Use proper bounds checking and fixed statements to ensure safe pointer arithmetic.

## 7. Advanced Reflection & Custom Attributes

### Question / Câu hỏi
How do you use reflection and custom attributes in C#?

### Suggested Answer / Câu trả lời gợi ý
Reflection allows you to inspect and manipulate types at runtime, and custom attributes provide metadata for types and members.

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class TableAttribute : Attribute
{
    public string Name { get; }

    public TableAttribute(string name)
    {
        Name = name;
    }
}

[Table("Products")]
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class EntityMapper
{
    public string GetTableName(Type type)
    {
        var attribute = type.GetCustomAttribute<TableAttribute>();
        return attribute?.Name ?? type.Name;
    }

    public T CreateInstance<T>() where T : class
    {
        return Activator.CreateInstance<T>();
    }
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: How do you create and use custom attributes?**
   - **A**: Create an attribute class and use reflection to read it.

2. **Q: When should you use reflection?**
   - **A**: Use reflection when you need to work with types dynamically or access metadata at runtime.

## 8. Preprocessor Directives

### Question / Câu hỏi
How do you use preprocessor directives in C#?

### Suggested Answer / Câu trả lời gợi ý
Preprocessor directives allow you to control compilation and organize code.

```csharp
public class Configuration
{
    public string GetConnectionString()
    {
        #if DEBUG
            return "DebugConnectionString";
        #elif RELEASE
            return "ReleaseConnectionString";
        #else
            return "DefaultConnectionString";
        #endif
    }

    #region Logging
    public void Log(string message)
    {
        #if DEBUG
            Console.WriteLine($"Debug: {message}");
        #endif
    }
    #endregion
}
```

### Follow-up Q&A / Câu hỏi phụ

1. **Q: When should you use preprocessor directives?**
   - **A**: Use them for conditional compilation and code organization.

2. **Q: How do you handle multiple build configurations?**
   - **A**: Use preprocessor directives and build configurations to handle different environments.

### Additional Resources
- [C# Async/Await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)
- [C# Memory Management](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/)
- [C# Pattern Matching](https://docs.microsoft.com/en-us/dotnet/csharp/pattern-matching)
- [C# Expression Trees](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/)
- [C# Source Generators](https://docs.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview)
- [C# Threading](https://docs.microsoft.com/en-us/dotnet/standard/threading/)
- [C# Unsafe Code](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code)
- [C# Reflection](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection)
- [C# Preprocessor Directives](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives)