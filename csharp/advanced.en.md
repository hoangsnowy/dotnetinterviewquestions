# Advanced C# Concepts - Interview Questions

This guide covers advanced C# concepts with practical examples and interview questions.

## Table of Contents
1. [Delegates and Events](#1-delegates-and-events)
2. [LINQ and Lambda Expressions](#2-linq-and-lambda-expressions)
3. [Async/Await and Tasks](#3-asyncawait-and-tasks)
4. [Reflection and Attributes](#4-reflection-and-attributes)
5. [Generics and Constraints](#5-generics-and-constraints)

## 1. Delegates and Events

### Core Concept
Delegates are type-safe function pointers that can reference methods, while events are a way to provide notifications to subscribers.

### Real-world Example
In an ecommerce system:
- Order status change notifications
- Payment processing callbacks
- Inventory update events
- Price change notifications

### Code Example
```csharp
public class OrderProcessor
{
    // Delegate definition
    public delegate void OrderStatusChangedHandler(Order order, OrderStatus newStatus);
    
    // Event using the delegate
    public event OrderStatusChangedHandler OrderStatusChanged;
    
    // Event using built-in EventHandler
    public event EventHandler<OrderEventArgs> OrderProcessed;

    private readonly ILogger<OrderProcessor> _logger;

    public OrderProcessor(ILogger<OrderProcessor> logger)
    {
        _logger = logger;
    }

    public async Task ProcessOrder(Order order)
    {
        try
        {
            // Process order...
            await UpdateOrderStatus(order, OrderStatus.Processing);
            
            // Notify subscribers
            OnOrderStatusChanged(order, OrderStatus.Processing);
            OnOrderProcessed(new OrderEventArgs(order));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error processing order {OrderId}", order.Id);
            throw;
        }
    }

    protected virtual void OnOrderStatusChanged(Order order, OrderStatus newStatus)
    {
        OrderStatusChanged?.Invoke(order, newStatus);
    }

    protected virtual void OnOrderProcessed(OrderEventArgs e)
    {
        OrderProcessed?.Invoke(this, e);
    }
}

// Usage
public class OrderNotificationService
{
    private readonly ILogger<OrderNotificationService> _logger;

    public OrderNotificationService(ILogger<OrderNotificationService> logger)
    {
        _logger = logger;
    }

    public void SubscribeToOrderEvents(OrderProcessor processor)
    {
        processor.OrderStatusChanged += HandleOrderStatusChanged;
        processor.OrderProcessed += HandleOrderProcessed;
    }

    private void HandleOrderStatusChanged(Order order, OrderStatus newStatus)
    {
        _logger.LogInformation("Order {OrderId} status changed to {Status}", 
            order.Id, newStatus);
    }

    private void HandleOrderProcessed(object sender, OrderEventArgs e)
    {
        _logger.LogInformation("Order {OrderId} processed", e.Order.Id);
    }
}
```

### Best Practices
1. **Delegate Design**
   - Use built-in delegates when possible
   - Follow naming conventions
   - Document delegate parameters
   - Handle null cases

2. **Event Handling**
   - Use EventHandler<T> for standard events
   - Implement proper thread safety
   - Handle exceptions in handlers
   - Unsubscribe when done

3. **Thread Safety**
   - Use thread-safe event invocation
   - Handle concurrent subscriptions
   - Protect event handlers
   - Use proper locking

4. **Error Handling**
   - Handle exceptions in handlers
   - Log errors properly
   - Provide error context
   - Maintain system stability

### Anti-patterns
1. **Delegate Issues**
   - Memory leaks
   - Thread safety issues
   - Poor error handling
   - Not unsubscribing

2. **Event Problems**
   - Too many events
   - Complex event chains
   - Poor error handling
   - Not thread-safe

3. **Thread Safety**
   - Race conditions
   - Deadlocks
   - Not thread-safe
   - Poor locking

4. **Error Handling**
   - Swallowing exceptions
   - Not logging
   - Poor error context
   - System instability

### Follow-up Questions

1. **Q: What's the difference between delegates and events?**
   - **Answer**: Delegates are function pointers that can be invoked directly, while events are a way to provide notifications to subscribers and can only be invoked by the declaring class.
   - **Example**:
     ```csharp
     // Delegate
     public delegate void ProcessOrderDelegate(Order order);
     ProcessOrderDelegate processor = ProcessOrder;
     processor(order); // Can be invoked directly

     // Event
     public event EventHandler<OrderEventArgs> OrderProcessed;
     OrderProcessed?.Invoke(this, new OrderEventArgs(order)); // Can only be invoked by declaring class
     ```
   - **Best Practice**: Use events for notifications and delegates for callbacks.

2. **Q: How do you handle memory leaks with events?**
   - **Answer**: Always unsubscribe from events when the subscriber is no longer needed, and use weak references for long-lived subscriptions.
   - **Example**:
     ```csharp
     public class OrderSubscriber : IDisposable
     {
         private readonly OrderProcessor _processor;

         public OrderSubscriber(OrderProcessor processor)
         {
             _processor = processor;
             _processor.OrderProcessed += HandleOrderProcessed;
         }

         public void Dispose()
         {
             _processor.OrderProcessed -= HandleOrderProcessed;
         }
     }
     ```
   - **Best Practice**: Implement IDisposable and unsubscribe from events.

## 2. LINQ and Lambda Expressions

### Core Concept
LINQ (Language Integrated Query) provides a way to query data using a SQL-like syntax, while lambda expressions are anonymous functions that can be used to create delegates or expression trees.

### Real-world Example
In an ecommerce system:
- Product filtering and sorting
- Order aggregation and grouping
- Customer data analysis
- Inventory management

### Code Example
```csharp
public class ProductService
{
    private readonly List<Product> _products;
    private readonly ILogger<ProductService> _logger;

    public ProductService(ILogger<ProductService> logger)
    {
        _products = new List<Product>();
        _logger = logger;
    }

    public IEnumerable<Product> GetProductsByCategory(string category)
    {
        return _products
            .Where(p => p.Category == category)
            .OrderBy(p => p.Price);
    }

    public IEnumerable<Product> GetProductsInPriceRange(decimal minPrice, decimal maxPrice)
    {
        return _products
            .Where(p => p.Price >= minPrice && p.Price <= maxPrice)
            .OrderByDescending(p => p.Price);
    }

    public IEnumerable<IGrouping<string, Product>> GetProductsByCategory()
    {
        return _products
            .GroupBy(p => p.Category)
            .OrderBy(g => g.Key);
    }

    public decimal GetAveragePriceByCategory(string category)
    {
        return _products
            .Where(p => p.Category == category)
            .Average(p => p.Price);
    }

    public IEnumerable<Product> GetTopSellingProducts(int count)
    {
        return _products
            .OrderByDescending(p => p.SalesCount)
            .Take(count);
    }
}
```

### Best Practices
1. **LINQ Usage**
   - Use appropriate methods
   - Consider performance
   - Handle nulls
   - Use proper filtering

2. **Lambda Expressions**
   - Keep them simple
   - Use meaningful names
   - Handle exceptions
   - Consider readability

3. **Performance**
   - Use deferred execution
   - Avoid multiple iterations
   - Use appropriate methods
   - Consider memory usage

4. **Error Handling**
   - Handle nulls
   - Validate inputs
   - Handle exceptions
   - Log errors

### Anti-patterns
1. **LINQ Issues**
   - N+1 queries
   - Multiple iterations
   - Poor performance
   - Not handling nulls

2. **Lambda Problems**
   - Complex expressions
   - Poor readability
   - Not handling errors
   - Memory leaks

3. **Performance**
   - Eager loading
   - Multiple iterations
   - Poor filtering
   - Memory issues

4. **Error Handling**
   - Not handling nulls
   - Not validating
   - Not logging
   - Poor error messages

### Follow-up Questions

1. **Q: What's the difference between IEnumerable and IQueryable?**
   - **Answer**: IEnumerable is used for in-memory collections, while IQueryable is used for database queries and supports deferred execution.
   - **Example**:
     ```csharp
     // IEnumerable - in-memory
     var products = _products
         .Where(p => p.Price > 100)
         .ToList(); // Executes immediately

     // IQueryable - database
     var products = _dbContext.Products
         .Where(p => p.Price > 100)
         .ToList(); // Executes when enumerated
     ```
   - **Best Practice**: Use IQueryable for database queries and IEnumerable for in-memory collections.

2. **Q: How do you optimize LINQ performance?**
   - **Answer**: Use appropriate methods, avoid multiple iterations, and consider using compiled queries for frequently used queries.
   - **Example**:
     ```csharp
     // Compiled query
     private static readonly Func<DbContext, string, IEnumerable<Product>> GetProductsByCategory =
         EF.CompileQuery((DbContext context, string category) =>
             context.Products
                 .Where(p => p.Category == category)
                 .OrderBy(p => p.Price));

     // Usage
     var products = GetProductsByCategory(_dbContext, "Electronics");
     ```
   - **Best Practice**: Use compiled queries for frequently used queries.

## 3. Async/Await and Tasks

### Core Concept
Async/await is a way to write asynchronous code that looks synchronous, while Tasks represent asynchronous operations.

### Real-world Example
In an ecommerce system:
- API calls
- Database operations
- File I/O
- External service integration

### Code Example
```csharp
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentProcessor _paymentProcessor;
    private readonly IInventoryService _inventoryService;
    private readonly ILogger<OrderService> _logger;

    public OrderService(
        IOrderRepository orderRepository,
        IPaymentProcessor paymentProcessor,
        IInventoryService inventoryService,
        ILogger<OrderService> logger)
    {
        _orderRepository = orderRepository;
        _paymentProcessor = paymentProcessor;
        _inventoryService = inventoryService;
        _logger = logger;
    }

    public async Task<OrderResult> ProcessOrderAsync(Order order)
    {
        try
        {
            // Validate order
            if (order == null)
                throw new ArgumentNullException(nameof(order));

            // Process payment
            var paymentResult = await _paymentProcessor.ProcessPaymentAsync(order.Total);
            if (!paymentResult.Success)
                return new OrderResult { Success = false, Error = paymentResult.Error };

            // Update inventory
            await _inventoryService.UpdateStockAsync(order.Items);

            // Save order
            await _orderRepository.SaveOrderAsync(order);

            return new OrderResult { Success = true, OrderId = order.Id };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error processing order {OrderId}", order.Id);
            return new OrderResult { Success = false, Error = ex.Message };
        }
    }

    public async Task<IEnumerable<Order>> GetOrdersByCustomerAsync(int customerId)
    {
        return await _orderRepository.GetOrdersByCustomerAsync(customerId);
    }

    public async Task<Order> GetOrderByIdAsync(int orderId)
    {
        return await _orderRepository.GetOrderByIdAsync(orderId);
    }
}
```

### Best Practices
1. **Async/Await Usage**
   - Use async/await consistently
   - Avoid async void
   - Handle exceptions
   - Use proper cancellation

2. **Task Management**
   - Use Task.WhenAll/WhenAny
   - Handle task exceptions
   - Use proper cancellation
   - Monitor task status

3. **Performance**
   - Avoid blocking calls
   - Use proper concurrency
   - Handle resource cleanup
   - Monitor memory usage

4. **Error Handling**
   - Handle exceptions
   - Use proper logging
   - Provide error context
   - Maintain system stability

### Anti-patterns
1. **Async/Await Issues**
   - Async void
   - Blocking calls
   - Not handling exceptions
   - Poor cancellation

2. **Task Problems**
   - Not awaiting tasks
   - Poor exception handling
   - Not using cancellation
   - Memory leaks

3. **Performance**
   - Blocking calls
   - Poor concurrency
   - Resource leaks
   - Memory issues

4. **Error Handling**
   - Not handling exceptions
   - Not logging
   - Poor error context
   - System instability

### Follow-up Questions

1. **Q: What's the difference between Task and ValueTask?**
   - **Answer**: Task is a reference type that represents an asynchronous operation, while ValueTask is a value type that can avoid heap allocation for completed operations.
   - **Example**:
     ```csharp
     // Task - reference type
     public async Task<int> GetCountAsync()
     {
         await Task.Delay(100);
         return 42;
     }

     // ValueTask - value type
     public async ValueTask<int> GetCountAsync()
     {
         if (_cachedCount.HasValue)
             return _cachedCount.Value;

         await Task.Delay(100);
         return 42;
     }
     ```
   - **Best Practice**: Use ValueTask for methods that might complete synchronously.

2. **Q: How do you handle cancellation in async methods?**
   - **Answer**: Use CancellationToken to support cancellation and handle it properly in async methods.
   - **Example**:
     ```csharp
     public async Task<OrderResult> ProcessOrderAsync(
         Order order,
         CancellationToken cancellationToken = default)
     {
         try
         {
             cancellationToken.ThrowIfCancellationRequested();

             // Process order...
             await _paymentProcessor.ProcessPaymentAsync(
                 order.Total,
                 cancellationToken);

             return new OrderResult { Success = true };
         }
         catch (OperationCanceledException)
         {
             _logger.LogWarning("Order processing cancelled");
             return new OrderResult { Success = false, Error = "Operation cancelled" };
         }
     }
     ```
   - **Best Practice**: Always support cancellation in long-running async operations.

## 4. Reflection and Attributes

### Core Concept
Reflection allows you to inspect and interact with types at runtime, while attributes provide metadata about types and members.

### Real-world Example
In an ecommerce system:
- Validation attributes
- API documentation
- Dependency injection
- Custom serialization

### Code Example
```csharp
[AttributeUsage(AttributeTargets.Property)]
public class RequiredAttribute : Attribute
{
    public string ErrorMessage { get; }

    public RequiredAttribute(string errorMessage = "This field is required")
    {
        ErrorMessage = errorMessage;
    }
}

public class Product
{
    [Required("Product name is required")]
    public string Name { get; set; }

    [Required("Product price is required")]
    [Range(0, double.MaxValue, ErrorMessage = "Price must be positive")]
    public decimal Price { get; set; }

    [Required("Product category is required")]
    public string Category { get; set; }
}

public class ValidationService
{
    public IEnumerable<ValidationError> Validate(object obj)
    {
        var errors = new List<ValidationError>();
        var type = obj.GetType();

        foreach (var property in type.GetProperties())
        {
            var requiredAttr = property.GetCustomAttribute<RequiredAttribute>();
            if (requiredAttr != null)
            {
                var value = property.GetValue(obj);
                if (value == null || string.IsNullOrEmpty(value.ToString()))
                {
                    errors.Add(new ValidationError
                    {
                        PropertyName = property.Name,
                        ErrorMessage = requiredAttr.ErrorMessage
                    });
                }
            }
        }

        return errors;
    }
}
```

### Best Practices
1. **Reflection Usage**
   - Cache reflection results
   - Handle exceptions
   - Consider performance
   - Use proper security

2. **Attribute Design**
   - Use proper targets
   - Document attributes
   - Handle inheritance
   - Consider performance

3. **Performance**
   - Cache reflection results
   - Use proper caching
   - Handle memory usage
   - Monitor performance

4. **Security**
   - Validate inputs
   - Handle exceptions
   - Use proper security
   - Monitor access

### Anti-patterns
1. **Reflection Issues**
   - Poor performance
   - Not caching
   - Security issues
   - Memory leaks

2. **Attribute Problems**
   - Wrong targets
   - Poor documentation
   - Not handling inheritance
   - Performance issues

3. **Performance**
   - Not caching
   - Poor performance
   - Memory issues
   - Resource leaks

4. **Security**
   - Not validating
   - Security issues
   - Not monitoring
   - Poor access control

### Follow-up Questions

1. **Q: When should you use reflection?**
   - **Answer**: Use reflection when you need to inspect or interact with types at runtime, such as for validation, serialization, or dependency injection.
   - **Example**:
     ```csharp
     public class SerializationService
     {
         public string Serialize(object obj)
         {
             var properties = obj.GetType()
                 .GetProperties()
                 .Where(p => p.CanRead);

             var values = properties
                 .Select(p => $"{p.Name}: {p.GetValue(obj)}");

             return string.Join(", ", values);
         }
     }
     ```
   - **Best Practice**: Cache reflection results and handle exceptions properly.

2. **Q: How do you create custom attributes?**
   - **Answer**: Create a class that inherits from Attribute and use AttributeUsage to specify valid targets.
   - **Example**:
     ```csharp
     [AttributeUsage(AttributeTargets.Property | AttributeTargets.Field)]
     public class MaxLengthAttribute : Attribute
     {
         public int Length { get; }

         public MaxLengthAttribute(int length)
         {
             Length = length;
         }
     }

     // Usage
     public class Product
     {
         [MaxLength(100)]
         public string Name { get; set; }
     }
     ```
   - **Best Practice**: Use proper targets and document attributes.

## 5. Generics and Constraints

### Core Concept
Generics allow you to write type-safe code that can work with different types, while constraints specify what types can be used with generics.

### Real-world Example
In an ecommerce system:
- Repository pattern
- Service layer
- Data access
- Caching

### Code Example
```csharp
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly DbContext _context;
    private readonly DbSet<T> _dbSet;
    private readonly ILogger<Repository<T>> _logger;

    public Repository(DbContext context, ILogger<Repository<T>> logger)
    {
        _context = context;
        _dbSet = context.Set<T>();
        _logger = logger;
    }

    public async Task<T> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(T entity)
    {
        _dbSet.Remove(entity);
        await _context.SaveChangesAsync();
    }
}

// Usage
public class ProductService
{
    private readonly IRepository<Product> _productRepository;
    private readonly ILogger<ProductService> _logger;

    public ProductService(
        IRepository<Product> productRepository,
        ILogger<ProductService> logger)
    {
        _productRepository = productRepository;
        _logger = logger;
    }

    public async Task<Product> GetProductAsync(int id)
    {
        return await _productRepository.GetByIdAsync(id);
    }

    public async Task<IEnumerable<Product>> GetAllProductsAsync()
    {
        return await _productRepository.GetAllAsync();
    }
}
```

### Best Practices
1. **Generic Design**
   - Use proper constraints
   - Document generics
   - Handle type safety
   - Consider performance

2. **Constraint Usage**
   - Use appropriate constraints
   - Document constraints
   - Handle type safety
   - Consider flexibility

3. **Type Safety**
   - Use proper constraints
   - Handle type checking
   - Consider inheritance
   - Monitor type safety

4. **Performance**
   - Use proper caching
   - Handle memory usage
   - Consider boxing
   - Monitor performance

### Anti-patterns
1. **Generic Issues**
   - Wrong constraints
   - Poor documentation
   - Type safety issues
   - Performance problems

2. **Constraint Problems**
   - Too restrictive
   - Not documented
   - Type safety issues
   - Flexibility issues

3. **Type Safety**
   - Not handling types
   - Poor type checking
   - Inheritance issues
   - Type safety problems

4. **Performance**
   - Not caching
   - Memory issues
   - Boxing issues
   - Performance problems

### Follow-up Questions

1. **Q: What are the different types of generic constraints?**
   - **Answer**: C# supports several types of constraints: where T : class, where T : struct, where T : new(), where T : BaseClass, where T : Interface.
   - **Example**:
     ```csharp
     public class Repository<T> where T : class, IEntity, new()
     {
         public T Create()
         {
             return new T();
         }

         public void Save(T entity)
         {
             // Save entity
         }
     }
     ```
   - **Best Practice**: Use appropriate constraints to ensure type safety.

2. **Q: How do you handle generic type constraints?**
   - **Answer**: Use appropriate constraints and handle type safety properly.
   - **Example**:
     ```csharp
     public class Cache<T> where T : class
     {
         private readonly Dictionary<string, T> _cache = new();

         public void Add(string key, T value)
         {
             if (value == null)
                 throw new ArgumentNullException(nameof(value));

             _cache[key] = value;
         }

         public T Get(string key)
         {
             return _cache.TryGetValue(key, out var value) ? value : null;
         }
     }
     ```
   - **Best Practice**: Use proper constraints and handle null cases.

### Key Takeaways
1. Understand advanced C# concepts
2. Use appropriate patterns
3. Handle performance
4. Follow best practices
5. Consider security

### Additional Resources
- [C# Delegates](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [C# Events](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/events/)
- [C# LINQ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)
- [C# Async/Await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)
- [C# Reflection](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection)
- [C# Attributes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/attributes/)
- [C# Generics](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/)