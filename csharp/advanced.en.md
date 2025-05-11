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

### Question
How do you handle asynchronous operations in C# using async/await and TPL?

### Suggested Answer
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

### Follow-up Q&A

1. **Q: How do you prevent deadlocks in async code?**
   - **A**: Use ConfigureAwait(false) in libraries and avoid blocking on async code.

2. **Q: When should you use Task.Run vs async/await?**
   - **A**: Use Task.Run for CPU-bound operations and async/await for I/O-bound operations.

## 2. Memory Management & Span<T> / Memory<T>

### Question
How do you optimize memory usage with Span<T> and Memory<T> in C#?

### Suggested Answer
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

### Follow-up Q&A

1. **Q: What's the difference between Span<T> and Memory<T>?**
   - **A**: Span<T> is stack-only and can't be stored in fields, while Memory<T> can be stored and used across async boundaries.

2. **Q: When should you use stackalloc?**
   - **A**: Use stackalloc for small, temporary buffers that don't need to live beyond the current method.

## 3. Pattern Matching

### Question
How do you use pattern matching in C# for complex type checking and data extraction?

### Suggested Answer
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

### Follow-up Q&A

1. **Q: What's the difference between switch statement and switch expression?**
   - **A**: Switch expression is more concise and can return a value, while switch statement is more traditional and can't return values.

2. **Q: How do you handle complex pattern matching?**
   - **A**: Use nested patterns and proper syntax to handle complex matching scenarios.

## 4. Expression Trees & Source Generators

### Question
How do you use expression trees and source generators in C#?

### Suggested Answer
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

### Follow-up Q&A

1. **Q: When should you use expression trees?**
   - **A**: Use expression trees when you need to analyze or transform code at runtime, such as in LINQ providers.

2. **Q: What are the benefits of source generators?**
   - **A**: Source generators can improve performance by generating code at compile time and reduce runtime overhead.

## 5. Threading & Synchronization

### Question
How do you handle concurrent operations and synchronization in C#?

### Suggested Answer
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
                // Update inventory safely
            }
        });
    }
}
```

### Follow-up Q&A

1. **Q: What's the difference between lock and SemaphoreSlim?**
   - **A**: Lock is for mutual exclusion, while SemaphoreSlim allows multiple threads to access a resource up to a limit.

2. **Q: When should you use Parallel.ForEach?**
   - **A**: Use Parallel.ForEach for CPU-bound operations that can be parallelized.

## 6. Unsafe Code & Pointers

### Question
How do you use unsafe code and pointers in C#?

### Suggested Answer
Unsafe code allows direct memory manipulation using pointers, which can be useful for performance-critical scenarios.

```csharp
public unsafe class BufferProcessor
{
    public void ProcessBuffer(byte[] buffer)
    {
        fixed (byte* ptr = buffer)
        {
            // Process buffer using pointers
            for (int i = 0; i < buffer.Length; i++)
            {
                *(ptr + i) = (byte)(*(ptr + i) ^ 0xFF);
            }
        }
    }
}
```

### Follow-up Q&A

1. **Q: When should you use unsafe code?**
   - **A**: Use unsafe code only when necessary for performance optimization and when you understand the risks.

2. **Q: What are the risks of using unsafe code?**
   - **A**: Unsafe code can lead to memory corruption, security vulnerabilities, and platform-specific issues.

## 7. Advanced Reflection & Custom Attributes

### Question
How do you use reflection and custom attributes in C#?

### Suggested Answer
Reflection allows you to inspect and modify code at runtime, while custom attributes provide metadata for your code.

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class LogAttribute : Attribute
{
    public string Message { get; }

    public LogAttribute(string message)
    {
        Message = message;
    }
}

public class ReflectionExample
{
    public void ProcessType(Type type)
    {
        var attributes = type.GetCustomAttributes<LogAttribute>();
        foreach (var attr in attributes)
        {
            Console.WriteLine(attr.Message);
        }
    }
}
```

### Follow-up Q&A

1. **Q: When should you use reflection?**
   - **A**: Use reflection when you need to inspect or modify code at runtime, such as in serialization or dependency injection.

2. **Q: What are the performance implications of reflection?**
   - **A**: Reflection can be slow, so use it judiciously and consider caching reflection results.

## 8. Preprocessor Directives

### Question
How do you use preprocessor directives in C#?

### Suggested Answer
Preprocessor directives allow you to control compilation and include/exclude code based on conditions.

```csharp
#define DEBUG
#define TRACE

public class PreprocessorExample
{
    public void Process()
    {
#if DEBUG
        Console.WriteLine("Debug mode");
#elif TRACE
        Console.WriteLine("Trace mode");
#else
        Console.WriteLine("Release mode");
#endif
    }
}
```

### Follow-up Q&A

1. **Q: What's the difference between #if and #ifdef?**
   - **A**: #if evaluates a condition, while #ifdef checks if a symbol is defined.

2. **Q: When should you use preprocessor directives?**
   - **A**: Use preprocessor directives for conditional compilation and platform-specific code.

## Additional Resources

- [Microsoft Docs: Async/Await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)
- [Microsoft Docs: Span<T>](https://docs.microsoft.com/en-us/dotnet/api/system.span-1)
- [Microsoft Docs: Pattern Matching](https://docs.microsoft.com/en-us/dotnet/csharp/pattern-matching)
- [Microsoft Docs: Expression Trees](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/)
- [Microsoft Docs: Threading](https://docs.microsoft.com/en-us/dotnet/standard/threading/)
- [Microsoft Docs: Unsafe Code](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code)
- [Microsoft Docs: Reflection](https://docs.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection)
- [Microsoft Docs: Preprocessor Directives](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives)