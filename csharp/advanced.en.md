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

### Core Concepts
- async/await keywords
- Task Parallel Library (TPL)
- Deadlock prevention
- ConfigureAwait(false)
- Task.Run vs async/await
- Task.WhenAll/WhenAny
- CancellationToken

### Real-world Example
Payment processing system with multiple async operations:
```csharp
public class PaymentProcessor
{
    private readonly IPaymentGateway _gateway;
    private readonly ILogger<PaymentProcessor> _logger;

    public PaymentProcessor(IPaymentGateway gateway, ILogger<PaymentProcessor> logger)
    {
        _gateway = gateway;
        _logger = logger;
    }

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

            // Handle results
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

### Best Practices
1. **Async/Await Usage**
   - Use async/await consistently
   - Avoid .Result or .Wait()
   - Use ConfigureAwait(false) in libraries
   - Handle cancellation properly

2. **Task Management**
   - Use Task.WhenAll for parallel operations
   - Handle task exceptions properly
   - Use CancellationToken for cancellation
   - Monitor task completion

3. **Error Handling**
   - Handle async exceptions
   - Use proper logging
   - Implement retry logic
   - Handle cancellation

4. **Performance**
   - Avoid blocking calls
   - Use proper task scheduling
   - Monitor async performance
   - Handle resource cleanup

### Anti-patterns
1. **Async Issues**
   - Blocking on async code
   - Not handling exceptions
   - Not using cancellation
   - Deadlocks

2. **Task Problems**
   - Not awaiting tasks
   - Not handling exceptions
   - Not using cancellation
   - Resource leaks

3. **Error Handling**
   - Not handling async errors
   - Not logging properly
   - No retry logic
   - Poor error context

4. **Performance**
   - Blocking calls
   - Poor task scheduling
   - Resource leaks
   - Memory issues

### Follow-up Questions

1. **Q: How do you prevent deadlocks in async code?**
   - **Answer**: Use ConfigureAwait(false) in libraries and avoid blocking on async code.
   - **Example**:
     ```csharp
     // Bad: Potential deadlock
     public string GetData()
     {
         return GetDataAsync().Result;
     }

     // Good: No deadlock
     public async Task<string> GetDataAsync()
     {
         return await GetDataFromServiceAsync().ConfigureAwait(false);
     }
     ```
   - **Best Practice**: Always use async/await consistently and avoid blocking calls.

2. **Q: How do you handle multiple async operations?**
   - **Answer**: Use Task.WhenAll for parallel operations and handle exceptions properly.
   - **Example**:
     ```csharp
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
     ```
   - **Best Practice**: Use Task.WhenAll for parallel operations and handle exceptions properly.

## 2. Memory Management & Span<T> / Memory<T>

### Core Concepts
- Garbage Collection
- Generations
- Pinning
- Span<T>
- Memory<T>
- Buffer handling

### Real-world Example
Efficient buffer processing in a file system:
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

### Best Practices
1. **Memory Management**
   - Use proper GC settings
   - Handle large objects
   - Monitor memory usage
   - Clean up resources

2. **Span<T> Usage**
   - Use for stack-only operations
   - Avoid heap allocations
   - Handle buffer boundaries
   - Consider performance

3. **Memory<T> Usage**
   - Use for heap operations
   - Handle memory lifetime
   - Consider thread safety
   - Monitor performance

4. **Buffer Handling**
   - Use proper buffer size
   - Handle buffer overflow
   - Consider performance
   - Monitor memory usage

### Anti-patterns
1. **Memory Issues**
   - Memory leaks
   - Large object heap
   - Poor GC settings
   - Resource leaks

2. **Span Problems**
   - Stack overflow
   - Buffer overflow
   - Thread safety issues
   - Performance problems

3. **Memory Problems**
   - Memory leaks
   - Thread safety issues
   - Lifetime issues
   - Performance problems

4. **Buffer Issues**
   - Buffer overflow
   - Memory leaks
   - Performance issues
   - Resource leaks

### Follow-up Questions

1. **Q: When should you use Span<T> vs Memory<T>?**
   - **Answer**: Use Span<T> for stack-only operations and Memory<T> for heap operations.
   - **Example**:
     ```csharp
     // Span<T> - stack-only
     public void ProcessBuffer(ReadOnlySpan<byte> buffer)
     {
         // Process buffer without allocations
     }

     // Memory<T> - heap
     public async Task ProcessBufferAsync(Memory<byte> buffer)
     {
         await ProcessBufferAsync(buffer);
     }
     ```
   - **Best Practice**: Use Span<T> for stack-only operations and Memory<T> for heap operations.

2. **Q: How do you handle large buffers efficiently?**
   - **Answer**: Use proper buffer size and handle memory efficiently.
   - **Example**:
     ```csharp
     public class BufferProcessor
     {
         private const int BufferSize = 8192;

         public async Task ProcessLargeFileAsync(string filePath)
         {
             using var file = File.OpenRead(filePath);
             var buffer = new byte[BufferSize];
             var memory = new Memory<byte>(buffer);

             while (true)
             {
                 var bytesRead = await file.ReadAsync(memory);
                 if (bytesRead == 0) break;

                 ProcessBuffer(memory.Slice(0, bytesRead));
             }
         }
     }
     ```
   - **Best Practice**: Use proper buffer size and handle memory efficiently.

## 3. Pattern Matching

### Core Concepts
- switch expression
- property pattern
- tuple pattern
- type pattern
- constant pattern

### Real-world Example
Order processing with pattern matching:
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

### Best Practices
1. **Pattern Design**
   - Use appropriate patterns
   - Handle all cases
   - Consider readability
   - Document patterns

2. **Switch Expression**
   - Use proper syntax
   - Handle all cases
   - Consider performance
   - Document cases

3. **Property Pattern**
   - Use proper syntax
   - Handle null cases
   - Consider inheritance
   - Document properties

4. **Tuple Pattern**
   - Use proper syntax
   - Handle all cases
   - Consider readability
   - Document tuples

### Anti-patterns
1. **Pattern Issues**
   - Not handling all cases
   - Poor readability
   - Not documented
   - Performance issues

2. **Switch Problems**
   - Not handling all cases
   - Poor syntax
   - Not documented
   - Performance issues

3. **Property Problems**
   - Not handling nulls
   - Poor syntax
   - Not documented
   - Inheritance issues

4. **Tuple Problems**
   - Not handling all cases
   - Poor syntax
   - Not documented
   - Readability issues

### Follow-up Questions

1. **Q: What's the difference between switch statement and switch expression?**
   - **Answer**: Switch expression is more concise and can return a value.
   - **Example**:
     ```csharp
     // Switch statement
     public string GetStatus(Order order)
     {
         switch (order.Status)
         {
             case OrderStatus.Pending:
                 return "Pending";
             case OrderStatus.Processing:
                 return "Processing";
             default:
                 return "Unknown";
         }
     }

     // Switch expression
     public string GetStatus(Order order) => order.Status switch
     {
         OrderStatus.Pending => "Pending",
         OrderStatus.Processing => "Processing",
         _ => "Unknown"
     };
     ```
   - **Best Practice**: Use switch expression for more concise code.

2. **Q: How do you handle complex pattern matching?**
   - **Answer**: Use nested patterns and proper syntax.
   - **Example**:
     ```csharp
     public string GetOrderInfo(Order order) => order switch
     {
         { Status: OrderStatus.Pending, PaymentStatus: PaymentStatus.Unpaid } => "Awaiting payment",
         { Status: OrderStatus.Processing, PaymentStatus: PaymentStatus.Paid } => "Processing payment",
         { Status: OrderStatus.Shipped, TrackingNumber: not null } => $"Shipped: {order.TrackingNumber}",
         _ => "Unknown status"
     };
     ```
   - **Best Practice**: Use nested patterns for complex matching.

## 4. Expression Trees & Source Generators

### Core Concepts
- Expression<Func<T,bool>>
- Custom LINQ providers
- Source generators
- Code generation

### Real-world Example
Custom LINQ provider for database queries:
```csharp
public class CustomQueryProvider : IQueryProvider
{
    public IQueryable<TElement> CreateQuery<TElement>(Expression expression)
    {
        return new CustomQuery<TElement>(this, expression);
    }

    public object Execute(Expression expression)
    {
        // Convert expression to SQL
        var sql = TranslateExpression(expression);
        return ExecuteQuery(sql);
    }

    private string TranslateExpression(Expression expression)
    {
        // Translate expression to SQL
        return expression switch
        {
            MethodCallExpression m => TranslateMethodCall(m),
            BinaryExpression b => TranslateBinary(b),
            MemberExpression m => TranslateMember(m),
            _ => throw new NotSupportedException()
        };
    }
}
```

### Best Practices
1. **Expression Design**
   - Use proper syntax
   - Handle all cases
   - Consider performance
   - Document expressions

2. **LINQ Provider**
   - Use proper translation
   - Handle all operators
   - Consider performance
   - Document operators

3. **Source Generator**
   - Use proper syntax
   - Handle all cases
   - Consider performance
   - Document generation

4. **Code Generation**
   - Use proper syntax
   - Handle all cases
   - Consider performance
   - Document generation

### Anti-patterns
1. **Expression Issues**
   - Not handling all cases
   - Poor syntax
   - Not documented
   - Performance issues

2. **Provider Problems**
   - Not handling all operators
   - Poor translation
   - Not documented
   - Performance issues

3. **Generator Problems**
   - Not handling all cases
   - Poor syntax
   - Not documented
   - Performance issues

4. **Generation Problems**
   - Not handling all cases
   - Poor syntax
   - Not documented
   - Performance issues

### Follow-up Questions

1. **Q: How do you create a custom LINQ provider?**
   - **Answer**: Implement IQueryProvider and translate expressions.
   - **Example**:
     ```csharp
     public class CustomQuery<T> : IQueryable<T>
     {
         private readonly IQueryProvider _provider;
         private readonly Expression _expression;

         public CustomQuery(IQueryProvider provider, Expression expression)
         {
             _provider = provider;
             _expression = expression;
         }

         public IEnumerator<T> GetEnumerator()
         {
             return _provider.Execute<IEnumerable<T>>(_expression).GetEnumerator();
         }
     }
     ```
   - **Best Practice**: Implement IQueryProvider and translate expressions.

2. **Q: How do you use source generators?**
   - **Answer**: Create a generator class and implement ISourceGenerator.
   - **Example**:
     ```csharp
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
   - **Best Practice**: Create a generator class and implement ISourceGenerator.

## 5. Threading & Synchronization

### Core Concepts
- lock statement
- SemaphoreSlim
- Interlocked
- PLINQ
- Parallel.ForEach

### Real-world Example
Parallel order processing:
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

### Best Practices
1. **Thread Safety**
   - Use proper locking
   - Handle deadlocks
   - Consider performance
   - Monitor threads

2. **Synchronization**
   - Use proper primitives
   - Handle contention
   - Consider performance
   - Monitor resources

3. **Parallel Processing**
   - Use proper patterns
   - Handle exceptions
   - Consider performance
   - Monitor resources

4. **Resource Management**
   - Use proper cleanup
   - Handle resources
   - Consider performance
   - Monitor usage

### Anti-patterns
1. **Thread Issues**
   - Deadlocks
   - Race conditions
   - Not thread-safe
   - Resource leaks

2. **Sync Problems**
   - Poor locking
   - Contention issues
   - Not thread-safe
   - Resource leaks

3. **Parallel Problems**
   - Not handling exceptions
   - Poor patterns
   - Not thread-safe
   - Resource leaks

4. **Resource Problems**
   - Not cleaning up
   - Resource leaks
   - Not thread-safe
   - Performance issues

### Follow-up Questions

1. **Q: How do you prevent deadlocks in multi-threaded code?**
   - **Answer**: Use proper locking order and timeouts.
   - **Example**:
     ```csharp
     public class ResourceManager
     {
         private readonly object _lock1 = new object();
         private readonly object _lock2 = new object();

         public void ProcessResources()
         {
             // Always lock in the same order
             lock (_lock1)
             {
                 lock (_lock2)
                 {
                     // Process resources
                 }
             }
         }
     }
     ```
   - **Best Practice**: Use proper locking order and timeouts.

2. **Q: How do you handle parallel processing?**
   - **Answer**: Use proper patterns and handle exceptions.
   - **Example**:
     ```csharp
     public class OrderProcessor
     {
         public void ProcessOrders(IEnumerable<Order> orders)
         {
             Parallel.ForEach(orders, new ParallelOptions
             {
                 MaxDegreeOfParallelism = Environment.ProcessorCount
             }, order =>
             {
                 try
                 {
                     ProcessOrder(order);
                 }
                 catch (Exception ex)
                 {
                     _logger.LogError(ex, "Error processing order");
                 }
             });
         }
     }
     ```
   - **Best Practice**: Use proper patterns and handle exceptions.

## 6. Unsafe Code & Pointers

### Core Concepts
- unsafe block
- fixed statement
- pointer arithmetic
- native interop

### Real-world Example
Native buffer processing:
```csharp
public class BufferProcessor
{
    public unsafe void ProcessBuffer(byte[] buffer)
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

### Best Practices
1. **Unsafe Code**
   - Use sparingly
   - Handle memory
   - Consider safety
   - Document usage

2. **Pointer Usage**
   - Use properly
   - Handle bounds
   - Consider safety
   - Document usage

3. **Native Interop**
   - Use properly
   - Handle memory
   - Consider safety
   - Document usage

4. **Memory Management**
   - Use properly
   - Handle cleanup
   - Consider safety
   - Document usage

### Anti-patterns
1. **Unsafe Issues**
   - Overuse
   - Memory leaks
   - Safety issues
   - Not documented

2. **Pointer Problems**
   - Buffer overflow
   - Memory leaks
   - Safety issues
   - Not documented

3. **Interop Problems**
   - Memory leaks
   - Safety issues
   - Not documented
   - Resource leaks

4. **Memory Problems**
   - Memory leaks
   - Safety issues
   - Not documented
   - Resource leaks

### Follow-up Questions

1. **Q: When should you use unsafe code?**
   - **Answer**: Use unsafe code only when necessary for performance or interop.
   - **Example**:
     ```csharp
     public unsafe class ImageProcessor
     {
         public void ProcessImage(byte[] imageData)
         {
             fixed (byte* ptr = imageData)
             {
                 // Process image data
             }
         }
     }
     ```
   - **Best Practice**: Use unsafe code only when necessary.

2. **Q: How do you handle pointer arithmetic safely?**
   - **Answer**: Use proper bounds checking and fixed statements.
   - **Example**:
     ```csharp
     public unsafe class BufferProcessor
     {
         public void ProcessBuffer(byte[] buffer)
         {
             fixed (byte* ptr = buffer)
             {
                 for (int i = 0; i < buffer.Length; i++)
                 {
                     // Safe pointer arithmetic
                     byte* current = ptr + i;
                     *current = (byte)(*current ^ 0xFF);
                 }
             }
         }
     }
     ```
   - **Best Practice**: Use proper bounds checking and fixed statements.

## 7. Advanced Reflection & Custom Attributes

### Core Concepts
- Custom attributes
- Activator.CreateInstance
- Type information
- Member reflection

### Real-world Example
Custom attribute usage:
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

### Best Practices
1. **Attribute Design**
   - Use properly
   - Handle inheritance
   - Consider performance
   - Document usage

2. **Reflection Usage**
   - Use sparingly
   - Handle exceptions
   - Consider performance
   - Document usage

3. **Type Information**
   - Use properly
   - Handle inheritance
   - Consider performance
   - Document usage

4. **Member Reflection**
   - Use properly
   - Handle exceptions
   - Consider performance
   - Document usage

### Anti-patterns
1. **Attribute Issues**
   - Overuse
   - Poor design
   - Not documented
   - Performance issues

2. **Reflection Problems**
   - Overuse
   - Not handling exceptions
   - Not documented
   - Performance issues

3. **Type Problems**
   - Not handling inheritance
   - Not documented
   - Performance issues
   - Safety issues

4. **Member Problems**
   - Not handling exceptions
   - Not documented
   - Performance issues
   - Safety issues

### Follow-up Questions

1. **Q: How do you create and use custom attributes?**
   - **Answer**: Create an attribute class and use reflection to read it.
   - **Example**:
     ```csharp
     [AttributeUsage(AttributeTargets.Property)]
     public class RequiredAttribute : Attribute
     {
     }

     public class Validator
     {
         public bool Validate(object obj)
         {
             var type = obj.GetType();
             foreach (var property in type.GetProperties())
             {
                 if (property.GetCustomAttribute<RequiredAttribute>() != null)
                 {
                     var value = property.GetValue(obj);
                     if (value == null)
                         return false;
                 }
             }
             return true;
         }
     }
     ```
   - **Best Practice**: Create an attribute class and use reflection to read it.

2. **Q: How do you use Activator.CreateInstance?**
   - **Answer**: Use it to create instances of types dynamically.
   - **Example**:
     ```csharp
     public class Factory
     {
         public T Create<T>(string typeName) where T : class
         {
             var type = Type.GetType(typeName);
             if (type == null)
                 throw new ArgumentException($"Type {typeName} not found");

             return (T)Activator.CreateInstance(type);
         }
     }
     ```
   - **Best Practice**: Use it to create instances of types dynamically.

## 8. Preprocessor Directives

### Core Concepts
- #if DEBUG
- #region
- #warning
- #error
- Conditional compilation

### Real-world Example
Multi-environment build:
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

### Best Practices
1. **Directive Usage**
   - Use properly
   - Handle all cases
   - Consider readability
   - Document usage

2. **Conditional Compilation**
   - Use properly
   - Handle all cases
   - Consider readability
   - Document usage

3. **Region Usage**
   - Use properly
   - Handle organization
   - Consider readability
   - Document usage

4. **Warning Usage**
   - Use properly
   - Handle all cases
   - Consider readability
   - Document usage

### Anti-patterns
1. **Directive Issues**
   - Overuse
   - Not handling all cases
   - Not documented
   - Readability issues

2. **Conditional Problems**
   - Not handling all cases
   - Not documented
   - Readability issues
   - Maintenance issues

3. **Region Problems**
   - Overuse
   - Not documented
   - Readability issues
   - Organization issues

4. **Warning Problems**
   - Overuse
   - Not documented
   - Readability issues
   - Maintenance issues

### Follow-up Questions

1. **Q: When should you use preprocessor directives?**
   - **Answer**: Use them for conditional compilation and code organization.
   - **Example**:
     ```csharp
     public class Logger
     {
         public void Log(string message)
         {
             #if DEBUG
                 Console.WriteLine($"Debug: {message}");
             #elif TRACE
                 Console.WriteLine($"Trace: {message}");
             #endif
         }
     }
     ```
   - **Best Practice**: Use them for conditional compilation and code organization.

2. **Q: How do you handle multiple build configurations?**
   - **Answer**: Use preprocessor directives and build configurations.
   - **Example**:
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
     }
     ```
   - **Best Practice**: Use preprocessor directives and build configurations.

### Key Takeaways
1. Understand async/await and TPL
2. Use memory management effectively
3. Implement pattern matching
4. Use expression trees and source generators
5. Handle threading and synchronization
6. Use unsafe code carefully
7. Implement reflection and attributes
8. Use preprocessor directives properly

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