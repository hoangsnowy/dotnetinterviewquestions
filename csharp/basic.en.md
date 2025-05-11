# Basic C# Concepts - Interview Questions

This guide covers fundamental C# concepts with practical examples and interview questions.

## Table of Contents
1. [Variables and Data Types](#1-variables-and-data-types)
2. [Control Structures](#2-control-structures)
3. [Methods and Parameters](#3-methods-and-parameters)
4. [Collections](#4-collections)
5. [Exception Handling](#5-exception-handling)

## 1. Variables and Data Types

### Core Concept
C# is a strongly-typed language that supports various data types including value types and reference types.

### Real-world Example
In an ecommerce system:
- `decimal` for prices and money calculations
- `string` for product names and descriptions
- `int` for quantities and IDs
- `bool` for flags and conditions
- `DateTime` for order dates and timestamps

### Code Example
```csharp
public class Product
{
    // Value types
    public int Id { get; set; }
    public decimal Price { get; set; }
    public double Weight { get; set; }
    public bool IsInStock { get; set; }
    public DateTime CreatedAt { get; set; }

    // Reference types
    public string Name { get; set; }
    public string Description { get; set; }
    public List<string> Tags { get; set; }
    public Category Category { get; set; }
}

// Usage
var product = new Product
{
    Id = 1,
    Name = "Laptop",
    Price = 999.99m,
    Weight = 2.5,
    IsInStock = true,
    CreatedAt = DateTime.UtcNow,
    Tags = new List<string> { "Electronics", "Computers" }
};
```

### Best Practices
1. **Type Selection**
   - Use `decimal` for financial calculations
   - Use `string` for text data
   - Use `DateTime` for dates and times
   - Use appropriate numeric types

2. **Variable Naming**
   - Use meaningful names
   - Follow PascalCase for properties
   - Follow camelCase for variables
   - Use descriptive names

3. **Initialization**
   - Initialize reference types
   - Use object initializers
   - Set default values
   - Handle null cases

4. **Type Safety**
   - Use strong typing
   - Avoid type casting
   - Use type inference
   - Handle conversions

### Anti-patterns
1. **Type Selection**
   - Using `float` for money
   - Using `string` for dates
   - Using `object` unnecessarily
   - Using wrong numeric types

2. **Variable Naming**
   - Using single letters
   - Using unclear names
   - Using reserved words
   - Using inconsistent naming

3. **Initialization**
   - Not initializing collections
   - Not handling nulls
   - Using default values
   - Not setting defaults

4. **Type Safety**
   - Using dynamic
   - Excessive casting
   - Ignoring type safety
   - Using var unnecessarily

### Follow-up Questions

1. **Q: Why use decimal instead of double for money?**
   - **Answer**: Decimal provides exact decimal representation and is designed for financial calculations, while double can have rounding errors.
   - **Example**:
     ```csharp
     decimal price = 10.99m;
     double priceDouble = 10.99; // Potential rounding issues
     ```
   - **Best Practice**: Always use decimal for financial calculations.

2. **Q: When should you use var?**
   - **Answer**: Use var when the type is obvious from the right side of the assignment, but not when it makes the code less readable.
   - **Example**:
     ```csharp
     // Good use of var
     var products = new List<Product>();
     var price = 10.99m;

     // Bad use of var
     var x = GetValue(); // Type is not obvious
     ```
   - **Best Practice**: Use var when it improves readability.

### Key Takeaways
1. Choose appropriate data types
2. Follow naming conventions
3. Initialize variables properly
4. Maintain type safety
5. Handle null cases

### Additional Resources
- [C# Types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types)
- [C# Variables](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 2. Control Structures

### Core Concept
Control structures determine the flow of program execution through conditional statements and loops.

### Real-world Example
In an ecommerce system:
- If-else for order validation
- Switch for payment processing
- For/foreach for product iteration
- While for inventory checking

### Code Example
```csharp
public class OrderProcessor
{
    public void ProcessOrder(Order order)
    {
        // If-else
        if (order == null)
        {
            throw new ArgumentNullException(nameof(order));
        }

        // Switch
        switch (order.Status)
        {
            case OrderStatus.Pending:
                ProcessPendingOrder(order);
                break;
            case OrderStatus.Processing:
                ProcessProcessingOrder(order);
                break;
            case OrderStatus.Completed:
                ProcessCompletedOrder(order);
                break;
            default:
                throw new InvalidOperationException($"Unknown order status: {order.Status}");
        }

        // For loop
        for (int i = 0; i < order.Items.Count; i++)
        {
            var item = order.Items[i];
            ValidateItem(item);
        }

        // Foreach
        foreach (var item in order.Items)
        {
            UpdateInventory(item);
        }

        // While
        while (order.HasPendingItems())
        {
            ProcessNextItem(order);
        }
    }
}
```

### Best Practices
1. **Conditional Statements**
   - Use early returns
   - Keep conditions simple
   - Use switch for multiple cases
   - Handle all cases

2. **Loops**
   - Use foreach when possible
   - Avoid modifying collections
   - Use break/continue properly
   - Handle empty collections

3. **Error Handling**
   - Validate inputs
   - Handle edge cases
   - Use appropriate exceptions
   - Log errors

4. **Code Organization**
   - Keep methods focused
   - Use meaningful names
   - Add comments
   - Follow conventions

### Anti-patterns
1. **Conditional Statements**
   - Deep nesting
   - Complex conditions
   - Missing cases
   - Unnecessary conditions

2. **Loops**
   - Infinite loops
   - Modifying collections
   - Unnecessary loops
   - Poor performance

3. **Error Handling**
   - Ignoring errors
   - Catching all exceptions
   - Not logging
   - Poor error messages

4. **Code Organization**
   - Long methods
   - Unclear names
   - Missing comments
   - Inconsistent style

### Follow-up Questions

1. **Q: When should you use switch vs if-else?**
   - **Answer**: Use switch when you have multiple cases for a single value, and if-else for complex conditions or ranges.
   - **Example**:
     ```csharp
     // Good use of switch
     switch (order.Status)
     {
         case OrderStatus.Pending:
             return "Pending";
         case OrderStatus.Processing:
             return "Processing";
         default:
             return "Unknown";
     }

     // Good use of if-else
     if (order.Total > 1000 && order.IsPriority)
     {
         return "High Priority";
     }
     else if (order.Total > 500)
     {
         return "Medium Priority";
     }
     ```
   - **Best Practice**: Use switch for simple value matching and if-else for complex conditions.

2. **Q: How do you handle null checks in conditions?**
   - **Answer**: Use null-conditional operators and null-coalescing operators for cleaner null checks.
   - **Example**:
     ```csharp
     // Traditional null check
     if (order?.Customer?.Address != null)
     {
         var city = order.Customer.Address.City;
     }

     // Using null-conditional operator
     var city = order?.Customer?.Address?.City;

     // Using null-coalescing operator
     var name = order?.Customer?.Name ?? "Unknown";
     ```
   - **Best Practice**: Use modern C# features for null handling.

### Key Takeaways
1. Use appropriate control structures
2. Handle errors properly
3. Keep code readable
4. Follow best practices
5. Use modern C# features

### Additional Resources
- [C# Control Statements](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements)
- [C# Loops](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 3. Methods and Parameters

### Core Concept
Methods are blocks of code that perform specific tasks and can accept parameters and return values.

### Real-world Example
In an ecommerce system:
- Methods for order processing
- Methods for payment handling
- Methods for inventory management
- Methods for customer operations

### Code Example
```csharp
public class OrderService
{
    // Method with parameters and return value
    public async Task<OrderResult> ProcessOrder(Order order, IPaymentProcessor paymentProcessor)
    {
        if (order == null)
            throw new ArgumentNullException(nameof(order));
        if (paymentProcessor == null)
            throw new ArgumentNullException(nameof(paymentProcessor));

        try
        {
            // Process payment
            var paymentResult = await paymentProcessor.ProcessPayment(order.Total);
            if (!paymentResult.Success)
                return new OrderResult { Success = false, Error = paymentResult.Error };

            // Update inventory
            foreach (var item in order.Items)
            {
                await UpdateInventory(item);
            }

            return new OrderResult { Success = true, OrderId = order.Id };
        }
        catch (Exception ex)
        {
            return new OrderResult { Success = false, Error = ex.Message };
        }
    }

    // Method with optional parameters
    public decimal CalculateShippingCost(Order order, bool isExpress = false)
    {
        var baseCost = order.Items.Sum(item => item.Weight * 2.5m);
        return isExpress ? baseCost * 1.5m : baseCost;
    }

    // Method with params
    public decimal CalculateDiscount(params Discount[] discounts)
    {
        return discounts.Sum(d => d.Amount);
    }
}
```

### Best Practices
1. **Method Design**
   - Single responsibility
   - Clear naming
   - Proper parameters
   - Appropriate return type

2. **Parameter Handling**
   - Validate inputs
   - Use optional parameters
   - Use params when appropriate
   - Document parameters

3. **Error Handling**
   - Use exceptions
   - Handle errors
   - Return results
   - Log issues

4. **Code Organization**
   - Keep methods focused
   - Use meaningful names
   - Add comments
   - Follow conventions

### Anti-patterns
1. **Method Design**
   - Too many parameters
   - Unclear names
   - Mixed responsibilities
   - Poor return types

2. **Parameter Handling**
   - Not validating
   - Too many optional
   - Misusing params
   - Poor documentation

3. **Error Handling**
   - Ignoring errors
   - Catching all
   - Not logging
   - Poor messages

4. **Code Organization**
   - Long methods
   - Unclear names
   - Missing comments
   - Inconsistent style

### Follow-up Questions

1. **Q: When should you use async/await?**
   - **Answer**: Use async/await for I/O-bound operations or when you need to keep the UI responsive.
   - **Example**:
     ```csharp
     public async Task<OrderResult> ProcessOrderAsync(Order order)
     {
         // I/O-bound operations
         await _paymentProcessor.ProcessPaymentAsync(order.Total);
         await _inventoryService.UpdateStockAsync(order.Items);
         await _emailService.SendConfirmationAsync(order);

         return new OrderResult { Success = true };
     }
     ```
   - **Best Practice**: Use async/await for I/O operations and long-running tasks.

2. **Q: How do you handle method parameters?**
   - **Answer**: Validate parameters, use appropriate types, and consider optional parameters when needed.
   - **Example**:
     ```csharp
     public void UpdateProduct(
         int id,
         string name,
         decimal price,
         bool isActive = true,
         params string[] tags)
     {
         if (id <= 0)
             throw new ArgumentException("Id must be positive", nameof(id));
         if (string.IsNullOrEmpty(name))
             throw new ArgumentException("Name cannot be empty", nameof(name));
         if (price < 0)
             throw new ArgumentException("Price cannot be negative", nameof(price));

         // Update product
     }
     ```
   - **Best Practice**: Validate parameters and use appropriate parameter types.

### Key Takeaways
1. Design methods properly
2. Handle parameters correctly
3. Use async/await appropriately
4. Follow best practices
5. Document methods

### Additional Resources
- [C# Methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods)
- [C# Parameters](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/passing-parameters)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 4. Collections

### Core Concept
Collections are data structures that store and manage groups of objects.

### Real-world Example
In an ecommerce system:
- List for order items
- Dictionary for product cache
- HashSet for unique tags
- Queue for order processing

### Code Example
```csharp
public class OrderManager
{
    private readonly List<Order> _orders;
    private readonly Dictionary<int, Product> _productCache;
    private readonly HashSet<string> _uniqueTags;
    private readonly Queue<Order> _processingQueue;

    public OrderManager()
    {
        _orders = new List<Order>();
        _productCache = new Dictionary<int, Product>();
        _uniqueTags = new HashSet<string>();
        _processingQueue = new Queue<Order>();
    }

    public void AddOrder(Order order)
    {
        _orders.Add(order);
        _processingQueue.Enqueue(order);
    }

    public Product GetProduct(int id)
    {
        if (_productCache.TryGetValue(id, out var product))
        {
            return product;
        }

        product = FetchProductFromDatabase(id);
        _productCache[id] = product;
        return product;
    }

    public void AddTag(string tag)
    {
        _uniqueTags.Add(tag);
    }

    public void ProcessNextOrder()
    {
        if (_processingQueue.Count > 0)
        {
            var order = _processingQueue.Dequeue();
            ProcessOrder(order);
        }
    }
}
```

### Best Practices
1. **Collection Selection**
   - Use appropriate types
   - Consider performance
   - Handle concurrency
   - Use generics

2. **Collection Operations**
   - Use LINQ
   - Handle nulls
   - Check bounds
   - Use proper methods

3. **Memory Management**
   - Clear unused
   - Use capacity
   - Handle disposal
   - Monitor size

4. **Thread Safety**
   - Use thread-safe
   - Handle locks
   - Use concurrent
   - Avoid deadlocks

### Anti-patterns
1. **Collection Selection**
   - Wrong type
   - Poor performance
   - Not thread-safe
   - Not generic

2. **Collection Operations**
   - No LINQ
   - Not checking
   - Wrong methods
   - Poor performance

3. **Memory Management**
   - Not clearing
   - No capacity
   - Not disposing
   - Memory leaks

4. **Thread Safety**
   - Not thread-safe
   - Deadlocks
   - Race conditions
   - Poor locking

### Follow-up Questions

1. **Q: When should you use List vs Array?**
   - **Answer**: Use List when you need dynamic sizing and Array when you need fixed size and better performance.
   - **Example**:
     ```csharp
     // List - dynamic size
     var orderItems = new List<OrderItem>();
     orderItems.Add(new OrderItem());

     // Array - fixed size
     var productIds = new int[10];
     productIds[0] = 1;
     ```
   - **Best Practice**: Use List for dynamic collections and Array for fixed-size collections.

2. **Q: How do you handle concurrent collections?**
   - **Answer**: Use thread-safe collections from System.Collections.Concurrent namespace.
   - **Example**:
     ```csharp
     private readonly ConcurrentDictionary<int, Product> _productCache;
     private readonly ConcurrentQueue<Order> _processingQueue;

     public void AddProduct(Product product)
     {
         _productCache.TryAdd(product.Id, product);
     }

     public void ProcessOrders()
     {
         while (_processingQueue.TryDequeue(out var order))
         {
             ProcessOrder(order);
         }
     }
     ```
   - **Best Practice**: Use concurrent collections for thread-safe operations.

### Key Takeaways
1. Choose appropriate collections
2. Use LINQ effectively
3. Handle memory properly
4. Consider thread safety
5. Follow best practices

### Additional Resources
- [C# Collections](https://docs.microsoft.com/en-us/dotnet/standard/collections/)
- [C# LINQ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 5. Exception Handling

### Core Concept
Exception handling is a mechanism to handle runtime errors and maintain application stability.

### Real-world Example
In an ecommerce system:
- Handle payment failures
- Handle network errors
- Handle validation errors
- Handle database errors

### Code Example
```csharp
public class OrderProcessor
{
    private readonly ILogger<OrderProcessor> _logger;
    private readonly IPaymentProcessor _paymentProcessor;
    private readonly IInventoryService _inventoryService;

    public OrderProcessor(
        ILogger<OrderProcessor> logger,
        IPaymentProcessor paymentProcessor,
        IInventoryService inventoryService)
    {
        _logger = logger;
        _paymentProcessor = paymentProcessor;
        _inventoryService = inventoryService;
    }

    public async Task<OrderResult> ProcessOrder(Order order)
    {
        try
        {
            // Validate order
            if (order == null)
                throw new ArgumentNullException(nameof(order));

            if (!order.Items.Any())
                throw new InvalidOperationException("Order must contain items");

            // Process payment
            try
            {
                var paymentResult = await _paymentProcessor.ProcessPayment(order.Total);
                if (!paymentResult.Success)
                    throw new PaymentException(paymentResult.Error);
            }
            catch (PaymentException ex)
            {
                _logger.LogError(ex, "Payment processing failed");
                return new OrderResult { Success = false, Error = "Payment failed" };
            }

            // Update inventory
            try
            {
                await _inventoryService.UpdateStock(order.Items);
            }
            catch (InventoryException ex)
            {
                _logger.LogError(ex, "Inventory update failed");
                // Attempt to refund payment
                await _paymentProcessor.RefundPayment(order.Total);
                return new OrderResult { Success = false, Error = "Inventory update failed" };
            }

            return new OrderResult { Success = true, OrderId = order.Id };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Order processing failed");
            return new OrderResult { Success = false, Error = "Internal error" };
        }
    }
}
```

### Best Practices
1. **Exception Types**
   - Use specific exceptions
   - Create custom exceptions
   - Handle exceptions properly
   - Log exceptions

2. **Try-Catch Blocks**
   - Catch specific exceptions
   - Handle exceptions properly
   - Clean up resources
   - Log errors

3. **Error Handling**
   - Validate inputs
   - Handle edge cases
   - Provide error messages
   - Log errors

4. **Resource Management**
   - Use using statements
   - Clean up resources
   - Handle disposal
   - Monitor resources

### Anti-patterns
1. **Exception Types**
   - Catching all exceptions
   - Not logging
   - Poor messages
   - Not handling

2. **Try-Catch Blocks**
   - Empty catch blocks
   - Catching all
   - Not cleaning up
   - Not logging

3. **Error Handling**
   - Not validating
   - Not handling
   - Poor messages
   - Not logging

4. **Resource Management**
   - Not disposing
   - Not cleaning up
   - Memory leaks
   - Resource leaks

### Follow-up Questions

1. **Q: When should you create custom exceptions?**
   - **Answer**: Create custom exceptions when you need to handle specific error cases or provide additional information.
   - **Example**:
     ```csharp
     public class PaymentException : Exception
     {
         public string ErrorCode { get; }

         public PaymentException(string message, string errorCode)
             : base(message)
         {
             ErrorCode = errorCode;
         }
     }

     // Usage
     throw new PaymentException("Payment failed", "PAYMENT_DECLINED");
     ```
   - **Best Practice**: Create custom exceptions for domain-specific errors.

2. **Q: How do you handle async exceptions?**
   - **Answer**: Use try-catch with async/await and handle exceptions appropriately.
   - **Example**:
     ```csharp
     public async Task<OrderResult> ProcessOrderAsync(Order order)
     {
         try
         {
             await _paymentProcessor.ProcessPaymentAsync(order.Total);
             await _inventoryService.UpdateStockAsync(order.Items);
             return new OrderResult { Success = true };
         }
         catch (PaymentException ex)
         {
             _logger.LogError(ex, "Payment failed");
             return new OrderResult { Success = false, Error = ex.Message };
         }
         catch (InventoryException ex)
         {
             _logger.LogError(ex, "Inventory update failed");
             return new OrderResult { Success = false, Error = ex.Message };
         }
     }
     ```
   - **Best Practice**: Handle async exceptions properly and provide meaningful error messages.

### Key Takeaways
1. Use specific exceptions
2. Handle exceptions properly
3. Clean up resources
4. Log errors
5. Follow best practices

### Additional Resources
- [C# Exceptions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [C# Error Handling](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/)