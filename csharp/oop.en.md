# Object-Oriented Programming in C# – Interview Questions (Ecommerce Domain)

This comprehensive guide covers essential OOP concepts in C# with ecommerce-focused examples, detailed explanations, best practices, and real-world scenarios.

## Table of Contents
1. [Four Principles of OOP](#1-four-principles-of-oop)
2. [Encapsulation](#2-encapsulation)
3. [Inheritance](#3-inheritance)
4. [Polymorphism](#4-polymorphism)
5. [Abstraction](#5-abstraction)
6. [Abstract Class vs Interface](#6-abstract-class-vs-interface)
7. [Overloading vs Overriding](#7-overloading-vs-overriding)
8. [Sealed Keyword](#8-sealed-keyword)
9. [Access Modifiers](#9-access-modifiers)
10. [Class vs Struct](#10-class-vs-struct)

## 1. Four Principles of OOP

### Core Concept
Object-Oriented Programming (OOP) is a programming paradigm based on the concept of "objects" that contain data and code. The four main principles of OOP are:

1. **Encapsulation**: Bundling of data and methods that operate on that data within a single unit (class)
2. **Abstraction**: Hiding complex implementation details and showing only necessary features
3. **Inheritance**: Creating new classes from existing ones
4. **Polymorphism**: Ability of objects to take multiple forms

### Real-world Example
In an ecommerce system:
- **Encapsulation**: Order class encapsulates order details and operations
- **Abstraction**: Payment processing interface hides complex payment gateway details
- **Inheritance**: Different product types (Physical, Digital) inherit from base Product class
- **Polymorphism**: Different shipping methods (Standard, Express) implement same interface

### Code Example
```csharp
// Encapsulation
public class Order
{
    private readonly List<OrderItem> _items;
    private decimal _total;
    
    public Order()
    {
        _items = new List<OrderItem>();
        _total = 0;
    }
    
    public void AddItem(Product product, int quantity)
    {
        // Encapsulated logic for adding items
        var item = new OrderItem(product, quantity);
        _items.Add(item);
        RecalculateTotal();
    }
    
    private void RecalculateTotal()
    {
        _total = _items.Sum(item => item.Subtotal);
    }
    
    public decimal GetTotal() => _total;
}

// Abstraction
public interface IPaymentProcessor
{
    Task<PaymentResult> ProcessPayment(PaymentRequest request);
}

// Inheritance
public abstract class Product
{
    public string Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class PhysicalProduct : Product
{
    public double Weight { get; set; }
    public string Dimensions { get; set; }
}

// Polymorphism
public interface IShippingCalculator
{
    decimal CalculateShippingCost(Order order);
}

public class StandardShipping : IShippingCalculator
{
    public decimal CalculateShippingCost(Order order) => 5.99m;
}

public class ExpressShipping : IShippingCalculator
{
    public decimal CalculateShippingCost(Order order) => 15.99m;
}
```

### Best Practices
1. **Encapsulation**
   - Use private fields with public properties
   - Implement proper validation in setters
   - Use readonly collections for collections
   - Implement proper immutability where possible

2. **Abstraction**
   - Define clear interfaces
   - Hide implementation details
   - Use dependency injection
   - Follow interface segregation principle

3. **Inheritance**
   - Favor composition over inheritance
   - Use abstract classes for shared implementation
   - Keep inheritance hierarchy shallow
   - Use interfaces for multiple inheritance

4. **Polymorphism**
   - Program to interfaces
   - Use strategy pattern for different behaviors
   - Implement proper type checking
   - Use virtual methods judiciously

### Anti-patterns
1. **Encapsulation**
   - Exposing internal state
   - Using public fields
   - Mutable collections
   - Global state

2. **Abstraction**
   - Leaky abstractions
   - Over-abstraction
   - Interface bloat
   - Tight coupling

3. **Inheritance**
   - Deep inheritance hierarchies
   - Inheritance for code reuse only
   - Multiple inheritance of classes
   - Inheritance for implementation

4. **Polymorphism**
   - Type checking with if-else
   - Downcasting
   - Overuse of virtual methods
   - Runtime type checking

### Follow-up Questions

1. **Q: How does encapsulation improve security in an ecommerce system?**
   - **Answer**: Encapsulation helps protect sensitive data like payment information and customer details by controlling access through well-defined interfaces.
   - **Example**: 
     ```csharp
     public class PaymentInfo
     {
         private string _cardNumber;
         public string MaskedCardNumber => $"****-****-****-{_cardNumber.Substring(12)}";
     }
     ```
   - **Best Practice**: Always validate and sanitize data in encapsulated properties.

2. **Q: When should you use abstract classes vs interfaces for abstraction?**
   - **Answer**: Use abstract classes when you need to share implementation details among related classes, and interfaces when you need to define a contract that can be implemented by unrelated classes.
   - **Example**:
     ```csharp
     // Abstract class for shared implementation
     public abstract class BaseProduct
     {
         protected decimal _price;
         public virtual decimal CalculateDiscount() => _price * 0.1m;
     }

     // Interface for unrelated implementations
     public interface IPaymentProcessor
     {
         Task<PaymentResult> ProcessPayment(PaymentRequest request);
     }
     ```
   - **Best Practice**: Use interfaces for defining contracts and abstract classes for sharing implementation.

### Key Takeaways
1. OOP principles help create maintainable and scalable ecommerce systems
2. Proper encapsulation is crucial for security and data integrity
3. Abstraction helps manage complexity in large systems
4. Inheritance should be used carefully and prefer composition
5. Polymorphism enables flexible and extensible code

### Additional Resources
- [Microsoft C# Documentation](https://docs.microsoft.com/en-us/dotnet/csharp/)
- [SOLID Principles in C#](https://www.dofactory.com/net/solid-principles)
- [Design Patterns in C#](https://refactoring.guru/design-patterns/csharp)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 2. Encapsulation

### Core Concept
Encapsulation is the bundling of data and methods that operate on that data within a single unit (class), hiding the internal state and requiring all interaction to be performed through an object's methods.

### Real-world Example
In an ecommerce system, encapsulation is crucial for:
- Order management
- Inventory control
- Payment processing
- Customer data protection

### Code Example
```csharp
public class Order
{
    private readonly List<OrderItem> _items;
    private decimal _total;
    private OrderStatus _status;
    private readonly ILogger<Order> _logger;
    private readonly IInventoryService _inventoryService;

    public Order(ILogger<Order> logger, IInventoryService inventoryService)
    {
        _items = new List<OrderItem>();
        _total = 0;
        _status = OrderStatus.Created;
        _logger = logger;
        _inventoryService = inventoryService;
    }

    public IReadOnlyList<OrderItem> Items => _items.AsReadOnly();
    public decimal Total => _total;
    public OrderStatus Status => _status;

    public async Task AddItem(Product product, int quantity)
    {
        if (product == null)
            throw new ArgumentNullException(nameof(product));
        if (quantity <= 0)
            throw new ArgumentException("Quantity must be positive", nameof(quantity));

        var item = new OrderItem(product, quantity);
        _items.Add(item);
        await RecalculateTotal();
        _logger.LogInformation($"Added item {product.Name} to order");
    }

    private async Task RecalculateTotal()
    {
        _total = _items.Sum(item => item.Subtotal);
        await _inventoryService.UpdateStock(_items);
    }

    public async Task ProcessPayment(IPaymentProcessor paymentProcessor)
    {
        if (_status != OrderStatus.Created)
            throw new InvalidOperationException("Order cannot be processed in current status");

        try
        {
            var result = await paymentProcessor.ProcessPayment(new PaymentRequest(_total));
            if (result.Success)
            {
                _status = OrderStatus.Paid;
                _logger.LogInformation("Order payment processed successfully");
            }
            else
            {
                throw new PaymentException(result.ErrorMessage);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Payment processing failed");
            throw;
        }
    }
}
```

### Best Practices
1. **Data Hiding**
   - Use private fields
   - Expose data through properties
   - Implement proper validation
   - Use readonly collections

2. **Method Encapsulation**
   - Keep methods focused
   - Use proper access modifiers
   - Implement error handling
   - Log important operations

3. **State Management**
   - Maintain object invariants
   - Use immutable objects where possible
   - Implement proper validation
   - Handle concurrent access

4. **Security**
   - Protect sensitive data
   - Implement proper validation
   - Use secure defaults
   - Follow security best practices

### Anti-patterns
1. **Data Exposure**
   - Public fields
   - Mutable collections
   - Global state
   - Exposed internals

2. **Method Issues**
   - Too many responsibilities
   - Poor error handling
   - Missing validation
   - Inconsistent state

3. **State Problems**
   - Mutable state
   - Race conditions
   - Invalid state
   - Poor validation

4. **Security Issues**
   - Exposed sensitive data
   - Missing validation
   - Insecure defaults
   - Poor error handling

### Follow-up Questions

1. **Q: How do you handle concurrent access to encapsulated data?**
   - **Answer**: Use thread-safe collections, implement proper locking mechanisms, and consider using immutable objects.
   - **Example**:
     ```csharp
     public class ThreadSafeOrder
     {
         private readonly object _lock = new object();
         private readonly List<OrderItem> _items;
         
         public void AddItem(OrderItem item)
         {
             lock (_lock)
             {
                 _items.Add(item);
                 RecalculateTotal();
             }
         }
     }
     ```
   - **Best Practice**: Use proper synchronization mechanisms and thread-safe collections.

2. **Q: How do you ensure data integrity in encapsulated classes?**
   - **Answer**: Implement proper validation, use immutable objects where possible, and maintain object invariants.
   - **Example**:
     ```csharp
     public class OrderItem
     {
         public Product Product { get; }
         public int Quantity { get; }
         public decimal UnitPrice { get; }
         
         public OrderItem(Product product, int quantity)
         {
             if (product == null)
                 throw new ArgumentNullException(nameof(product));
             if (quantity <= 0)
                 throw new ArgumentException("Quantity must be positive", nameof(quantity));
                 
             Product = product;
             Quantity = quantity;
             UnitPrice = product.Price;
         }
         
         public decimal Subtotal => UnitPrice * Quantity;
     }
     ```
   - **Best Practice**: Validate input data and maintain object invariants.

### Key Takeaways
1. Encapsulation helps protect data and maintain integrity
2. Proper access control is crucial for security
3. Thread safety is important for concurrent access
4. Validation helps maintain data integrity
5. Immutable objects can simplify state management

### Additional Resources
- [C# Encapsulation](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)
- [C# Thread Safety](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices)
- [C# Immutable Types](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/immutable-types)
- [C# Validation](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/validation)

## 3. Inheritance

### Core Concept
Inheritance is a mechanism that allows a class to inherit properties and methods from another class, enabling code reuse and the creation of a hierarchical relationship between classes.

### Real-world Example
In an ecommerce system, inheritance is used for:
- Product hierarchy (Base Product → Physical Product, Digital Product)
- Payment methods (Base Payment → Credit Card, PayPal)
- Shipping methods (Base Shipping → Standard, Express)
- Order types (Base Order → Regular Order, Subscription Order)

### Code Example
```csharp
// Base class
public abstract class Product
{
    public string Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Description { get; set; }
    public int StockQuantity { get; set; }

    protected Product(string id, string name, decimal price)
    {
        Id = id;
        Name = name;
        Price = price;
    }

    public virtual decimal CalculateDiscount()
    {
        return Price * 0.1m; // 10% default discount
    }

    public abstract string GetProductType();
}

// Derived class
public class PhysicalProduct : Product
{
    public double Weight { get; set; }
    public string Dimensions { get; set; }
    public bool RequiresShipping { get; set; }

    public PhysicalProduct(string id, string name, decimal price, double weight)
        : base(id, name, price)
    {
        Weight = weight;
        RequiresShipping = true;
    }

    public override decimal CalculateDiscount()
    {
        var baseDiscount = base.CalculateDiscount();
        return Weight > 10 ? baseDiscount * 1.5m : baseDiscount;
    }

    public override string GetProductType() => "Physical";
}

// Another derived class
public class DigitalProduct : Product
{
    public string DownloadUrl { get; set; }
    public long FileSize { get; set; }
    public string FileFormat { get; set; }
    public int DownloadLimit { get; set; }
    public int DownloadsCount { get; private set; }

    public DigitalProduct(string id, string name, decimal price, string downloadUrl)
        : base(id, name, price)
    {
        DownloadUrl = downloadUrl;
        RequiresShipping = false;
    }

    public override decimal CalculateDiscount()
    {
        return Price * 0.2m; // 20% discount for digital products
    }

    public override string GetProductType() => "Digital";

    public bool CanDownload()
    {
        return DownloadsCount < DownloadLimit;
    }

    public void IncrementDownloadCount()
    {
        DownloadsCount++;
    }
}
```

### Best Practices
1. **Class Design**
   - Keep inheritance hierarchy shallow
   - Use abstract classes for shared implementation
   - Implement proper constructors
   - Use virtual methods judiciously

2. **Method Overriding**
   - Use override keyword
   - Call base methods when appropriate
   - Maintain Liskov Substitution Principle
   - Document behavior changes

3. **Access Control**
   - Use protected for inherited members
   - Keep private members private
   - Use sealed to prevent further inheritance
   - Implement proper encapsulation

4. **Code Organization**
   - Group related classes
   - Use namespaces effectively
   - Follow naming conventions
   - Document inheritance relationships

### Anti-patterns
1. **Class Design**
   - Deep inheritance hierarchies
   - Inheritance for code reuse only
   - Multiple inheritance of classes
   - Inheritance for implementation

2. **Method Overriding**
   - Breaking base class contract
   - Not calling base methods
   - Overriding without virtual
   - Poor documentation

3. **Access Control**
   - Exposing protected members
   - Overusing protected
   - Not using sealed
   - Poor encapsulation

4. **Code Organization**
   - Unrelated classes in hierarchy
   - Poor namespace organization
   - Inconsistent naming
   - Missing documentation

### Follow-up Questions

1. **Q: How do you handle the "diamond problem" in C#?**
   - **Answer**: C# doesn't support multiple inheritance of classes, but it does support multiple interface implementation. Use interfaces and composition to avoid the diamond problem.
   - **Example**:
     ```csharp
     public interface IShippable
     {
         decimal CalculateShippingCost();
     }

     public interface IDownloadable
     {
         Task<string> GetDownloadUrl();
     }

     public class DigitalProduct : Product, IDownloadable
     {
         public async Task<string> GetDownloadUrl() => DownloadUrl;
     }

     public class PhysicalProduct : Product, IShippable
     {
         public decimal CalculateShippingCost() => Weight * 2.5m;
     }
     ```
   - **Best Practice**: Use interfaces for multiple inheritance and composition for code reuse.

2. **Q: When should you use an interface vs an abstract class?**
   - **Answer**: Use interfaces for defining contracts that can be implemented by unrelated classes, and abstract classes for sharing implementation among related classes.
   - **Example**:
     ```csharp
     // Interface for unrelated implementations
     public interface IPaymentGateway
     {
         Task<PaymentResult> ProcessPayment(PaymentRequest request);
     }

     // Abstract class for related implementations
     public abstract class BaseOrderProcessor
     {
         protected readonly ILogger _logger;
         protected readonly IPaymentGateway _paymentGateway;

         protected BaseOrderProcessor(ILogger logger, IPaymentGateway paymentGateway)
         {
             _logger = logger;
             _paymentGateway = paymentGateway;
         }

         public abstract Task<OrderResult> ProcessOrder(Order order);
     }
     ```
   - **Best Practice**: Use interfaces for contracts and abstract classes for shared implementation.

### Key Takeaways
1. Inheritance enables code reuse and polymorphism
2. Keep inheritance hierarchies shallow
3. Use interfaces for multiple inheritance
4. Follow Liskov Substitution Principle
5. Document inheritance relationships

### Additional Resources
- [C# Inheritance](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/inheritance)
- [C# Abstract Classes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members)
- [C# Interfaces](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 4. Polymorphism

### Core Concept
Polymorphism is the ability of objects to take multiple forms and behave differently based on their type. In C#, polymorphism is achieved through inheritance and interfaces, allowing for flexible and extensible code.

### Real-world Example
In an ecommerce system, polymorphism is used for:
- Payment processing (different payment methods)
- Shipping calculations (different shipping methods)
- Product types (different product behaviors)
- Order processing (different order types)

### Code Example
```csharp
// Interface for polymorphic behavior
public interface IPaymentProcessor
{
    Task<PaymentResult> ProcessPayment(PaymentRequest request);
    decimal CalculateFee(decimal amount);
}

// Concrete implementations
public class CreditCardProcessor : IPaymentProcessor
{
    private readonly ICreditCardGateway _gateway;
    private readonly ILogger<CreditCardProcessor> _logger;

    public CreditCardProcessor(ICreditCardGateway gateway, ILogger<CreditCardProcessor> logger)
    {
        _gateway = gateway;
        _logger = logger;
    }

    public async Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        try
        {
            _logger.LogInformation($"Processing credit card payment for amount: {request.Amount}");
            var result = await _gateway.ProcessCardPayment(request.CardNumber, request.Amount);
            return new PaymentResult { Success = true, TransactionId = result.TransactionId };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Credit card payment failed");
            return new PaymentResult { Success = false, ErrorMessage = ex.Message };
        }
    }

    public decimal CalculateFee(decimal amount)
    {
        return amount * 0.029m + 0.30m; // 2.9% + $0.30
    }
}

public class PayPalProcessor : IPaymentProcessor
{
    private readonly IPayPalGateway _gateway;
    private readonly ILogger<PayPalProcessor> _logger;

    public PayPalProcessor(IPayPalGateway gateway, ILogger<PayPalProcessor> logger)
    {
        _gateway = gateway;
        _logger = logger;
    }

    public async Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        try
        {
            _logger.LogInformation($"Processing PayPal payment for amount: {request.Amount}");
            var result = await _gateway.ProcessPayPalPayment(request.PayPalToken, request.Amount);
            return new PaymentResult { Success = true, TransactionId = result.TransactionId };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "PayPal payment failed");
            return new PaymentResult { Success = false, ErrorMessage = ex.Message };
        }
    }

    public decimal CalculateFee(decimal amount)
    {
        return amount * 0.024m + 0.30m; // 2.4% + $0.30
    }
}

// Usage example
public class OrderService
{
    private readonly IPaymentProcessor _paymentProcessor;
    private readonly ILogger<OrderService> _logger;

    public OrderService(IPaymentProcessor paymentProcessor, ILogger<OrderService> logger)
    {
        _paymentProcessor = paymentProcessor;
        _logger = logger;
    }

    public async Task<OrderResult> ProcessOrder(Order order)
    {
        try
        {
            var paymentRequest = new PaymentRequest
            {
                Amount = order.Total,
                // Other payment details...
            };

            var paymentResult = await _paymentProcessor.ProcessPayment(paymentRequest);
            if (!paymentResult.Success)
            {
                throw new PaymentException(paymentResult.ErrorMessage);
            }

            var fee = _paymentProcessor.CalculateFee(order.Total);
            _logger.LogInformation($"Payment processed successfully. Fee: {fee}");

            return new OrderResult { Success = true, OrderId = order.Id };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Order processing failed");
            throw;
        }
    }
}
```

### Best Practices
1. **Interface Design**
   - Define clear contracts
   - Keep interfaces focused
   - Use interface segregation
   - Document behavior

2. **Implementation**
   - Follow Liskov Substitution Principle
   - Handle errors properly
   - Use dependency injection
   - Implement logging

3. **Type Safety**
   - Use proper type checking
   - Avoid downcasting
   - Use pattern matching
   - Handle null cases

4. **Code Organization**
   - Group related implementations
   - Use proper namespaces
   - Follow naming conventions
   - Document relationships

### Anti-patterns
1. **Interface Design**
   - Interface bloat
   - Unclear contracts
   - Poor documentation
   - Tight coupling

2. **Implementation**
   - Breaking LSP
   - Poor error handling
   - Hard-coded dependencies
   - Missing logging

3. **Type Safety**
   - Excessive type checking
   - Unsafe downcasting
   - Ignoring null cases
   - Poor pattern matching

4. **Code Organization**
   - Scattered implementations
   - Poor namespace organization
   - Inconsistent naming
   - Missing documentation

### Follow-up Questions

1. **Q: How do you handle different payment methods in a polymorphic way?**
   - **Answer**: Use interfaces and dependency injection to handle different payment methods, allowing for easy addition of new payment types.
   - **Example**:
     ```csharp
     public interface IPaymentMethod
     {
         Task<PaymentResult> ProcessPayment(decimal amount);
         decimal CalculateFee(decimal amount);
     }

     public class PaymentService
     {
         private readonly Dictionary<PaymentType, IPaymentMethod> _paymentMethods;

         public PaymentService(IEnumerable<IPaymentMethod> paymentMethods)
         {
             _paymentMethods = paymentMethods.ToDictionary(pm => pm.Type);
         }

         public async Task<PaymentResult> ProcessPayment(PaymentType type, decimal amount)
         {
             if (!_paymentMethods.TryGetValue(type, out var method))
                 throw new ArgumentException($"Unsupported payment type: {type}");

             return await method.ProcessPayment(amount);
         }
     }
     ```
   - **Best Practice**: Use dependency injection and interfaces for flexible payment processing.

2. **Q: How do you ensure type safety when working with polymorphic objects?**
   - **Answer**: Use pattern matching, proper type checking, and handle null cases appropriately.
   - **Example**:
     ```csharp
     public class OrderProcessor
     {
         public decimal CalculateShippingCost(Product product)
         {
             return product switch
             {
                 PhysicalProduct p => p.Weight * 2.5m,
                 DigitalProduct => 0m,
                 _ => throw new ArgumentException($"Unsupported product type: {product.GetType()}")
             };
         }
     }
     ```
   - **Best Practice**: Use pattern matching and proper type checking for safe polymorphic behavior.

### Key Takeaways
1. Polymorphism enables flexible and extensible code
2. Use interfaces for polymorphic behavior
3. Follow Liskov Substitution Principle
4. Handle type safety properly
5. Use dependency injection

### Additional Resources
- [C# Polymorphism](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/polymorphism)
- [C# Pattern Matching](https://docs.microsoft.com/en-us/dotnet/csharp/pattern-matching)
- [C# Interfaces](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 5. Abstraction

### Core Concept
Abstraction is the process of hiding complex implementation details and showing only necessary features. It allows for a clear and concise representation of a system's components.

### Real-world Example
In an ecommerce system, abstraction is used for:
- Payment processing interface
- Product hierarchy
- Shipping methods

### Code Example
```csharp
// Abstraction
public interface IPaymentProcessor
{
    Task<PaymentResult> ProcessPayment(PaymentRequest request);
}

// Inheritance
public abstract class Product
{
    public string Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class PhysicalProduct : Product
{
    public double Weight { get; set; }
    public string Dimensions { get; set; }
}

// Polymorphism
public interface IShippingCalculator
{
    decimal CalculateShippingCost(Order order);
}

public class StandardShipping : IShippingCalculator
{
    public decimal CalculateShippingCost(Order order) => 5.99m;
}

public class ExpressShipping : IShippingCalculator
{
    public decimal CalculateShippingCost(Order order) => 15.99m;
}
```

### Best Practices
1. **Define Clear Interfaces**
   - Use interfaces for polymorphic behavior
   - Hide implementation details
   - Follow interface segregation principle

2. **Hide Implementation Details**
   - Use abstract classes for shared implementation
   - Use dependency injection
   - Follow interface segregation principle

3. **Use Dependency Injection**
   - Use dependency injection
   - Follow interface segregation principle

4. **Follow Interface Segregation Principle**
   - Use interfaces for polymorphic behavior
   - Hide implementation details
   - Follow interface segregation principle

### Anti-patterns
1. **Leaky Abstractions**
   - Over-abstraction
   - Interface bloat
   - Tight coupling

2. **Over-abstraction**
   - Leaky abstractions
   - Interface bloat
   - Tight coupling

3. **Interface Bloat**
   - Too many interfaces
   - Over-abstraction
   - Tight coupling

4. **Tight Coupling**
   - Interface bloat
   - Over-abstraction
   - Leaky abstractions
   - Tight coupling

### Follow-up Questions

1. **Q: How do you handle abstraction in an ecommerce system?**
   - **Answer**: Use interfaces for polymorphic behavior and abstract classes for shared implementation.
   - **Example**:
     ```csharp
     public interface IPaymentProcessor
     {
         Task<PaymentResult> ProcessPayment(PaymentRequest request);
     }

     public abstract class Product
     {
         public string Id { get; set; }
         public string Name { get; set; }
         public decimal Price { get; set; }
     }

     public class PhysicalProduct : Product
     {
         public double Weight { get; set; }
         public string Dimensions { get; set; }
     }
     ```
   - **Best Practice**: Use interfaces for polymorphic behavior and abstract classes for shared implementation.

2. **Q: When should you use an interface vs an abstract class for abstraction?**
   - **Answer**: Use interfaces for defining contracts that can be implemented by unrelated classes, and abstract classes for sharing implementation among related classes.
   - **Example**:
     ```csharp
     // Interface for unrelated implementations
     public interface IPaymentGateway
     {
         Task<PaymentResult> ProcessPayment(PaymentRequest request);
     }

     // Abstract class for related implementations
     public abstract class BaseOrderProcessor
     {
         protected readonly ILogger _logger;
         protected readonly IPaymentGateway _paymentGateway;

         protected BaseOrderProcessor(ILogger logger, IPaymentGateway paymentGateway)
         {
             _logger = logger;
             _paymentGateway = paymentGateway;
         }

         public abstract Task<OrderResult> ProcessOrder(Order order);
     }
     ```
   - **Best Practice**: Use interfaces for contracts and abstract classes for sharing implementation.

### Key Takeaways
1. Abstraction helps manage complexity in large systems
2. Use interfaces for polymorphic behavior
3. Abstract classes for shared implementation
4. Proper encapsulation
5. Follow interface segregation principle

### Additional Resources
- [C# Abstraction](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstraction)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [C# Design Patterns](https://refactoring.guru/design-patterns/csharp)

## 6. Abstract Class vs Interface

### Core Concept
An abstract class is a class that cannot be instantiated and is used as a base class for other classes. An interface is a collection of related methods that define a contract.

### Real-world Example
In an ecommerce system:
- **Abstract Class**: Base Product class
- **Interface**: Payment processing interface

### Code Example
```csharp
// Abstract class
public abstract class Product
{
    public string Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class PhysicalProduct : Product
{
    public double Weight { get; set; }
    public string Dimensions { get; set; }
}

// Interface
public interface IPaymentProcessor
{
    Task<PaymentResult> ProcessPayment(PaymentRequest request);
}
```

### Best Practices
1. **Use Abstract Classes for Shared Implementation**
   - Use abstract classes for shared implementation
   - Keep inheritance hierarchy shallow
   - Use interfaces for multiple inheritance

2. **Use Interfaces for Defining Contracts**
   - Use interfaces for defining contracts that can be implemented by unrelated classes
   - Follow interface segregation principle

3. **Avoid Tight Coupling**
   - Use interfaces for polymorphic behavior
   - Avoid tight coupling
   - Implement strategy pattern

4. **Avoid Interface Bloat**
   - Use interfaces for defining contracts
   - Avoid too many interfaces
   - Follow interface segregation principle

### Anti-patterns
1. **Tight Coupling**
   - Interface bloat
   - Over-abstraction
   - Leaky abstractions
   - Tight coupling

2. **Interface Bloat**
   - Too many interfaces
   - Over-abstraction
   - Leaky abstractions
   - Tight coupling

3. **Over-abstraction**
   - Leaky abstractions
   - Interface bloat
   - Tight coupling

4. **Interface Segregation Principle Violation**
   - Too many methods in interfaces
   - Interface bloat
   - Over-abstraction
   - Tight coupling

### Follow-up Questions

1. **Q: How do you handle the "diamond problem" in C#?**
   - **Answer**: C# doesn't support multiple inheritance of classes, but it does support multiple interface implementation. Use interfaces and composition to avoid the diamond problem.
   - **Example**:
     ```csharp
     public interface IShippable
     {
         decimal CalculateShippingCost();
     }

     public interface IDownloadable
     {
         Task<string> GetDownloadUrl();
     }

     public class DigitalProduct : Product, IDownloadable
     {
         public async Task<string> GetDownloadUrl() => DownloadUrl;
     }

     public class PhysicalProduct : Product, IShippable
     {
         public decimal CalculateShippingCost() => Weight * 2.5m;
     }
     ```
   - **Best Practice**: Use interfaces for multiple inheritance and composition for code reuse.

2. **Q: When should you use an interface vs an abstract class?**
   - **Answer**: Use interfaces for defining contracts that can be implemented by unrelated classes, and abstract classes for sharing implementation among related classes.
   - **Example**:
     ```csharp
     // Interface for unrelated implementations
     public interface IPaymentGateway
     {
         Task<PaymentResult> ProcessPayment(PaymentRequest request);
     }

     // Abstract class for related implementations
     public abstract class BaseOrderProcessor
     {
         protected readonly ILogger _logger;
         protected readonly IPaymentGateway _paymentGateway;

         protected BaseOrderProcessor(ILogger logger, IPaymentGateway paymentGateway)
         {
             _logger = logger;
             _paymentGateway = paymentGateway;
         }

         public abstract Task<OrderResult> ProcessOrder(Order order);
     }
     ```
   - **Best Practice**: Use interfaces for contracts and abstract classes for shared implementation.

### Key Takeaways
1. Use interfaces for defining contracts
2. Use abstract classes for shared implementation
3. Proper encapsulation
4. Avoid tight coupling
5. Follow interface segregation principle

### Additional Resources
- [C# Abstract Classes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members)
- [C# Interfaces](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 7. Overloading vs Overriding

### Core Concept
Overloading is the ability to define multiple methods with the same name but different parameters. Overriding is the ability to provide a specific implementation of a method that is already defined in a base class.

### Real-world Example
In an ecommerce system:
- **Overloading**: Different constructors for the same class
- **Overriding**: Different implementations for the same method in different classes

### Code Example
```csharp
// Overloading
public class Order
{
    private readonly List<OrderItem> _items;
    private decimal _total;
    
    public Order()
    {
        _items = new List<OrderItem>();
        _total = 0;
    }
    
    public void AddItem(Product product, int quantity)
    {
        // Encapsulated logic for adding items
        var item = new OrderItem(product, quantity);
        _items.Add(item);
        RecalculateTotal();
    }
    
    private void RecalculateTotal()
    {
        _total = _items.Sum(item => item.Subtotal);
    }
    
    public decimal GetTotal() => _total;
}

// Overriding
public interface IPaymentProcessor
{
    Task<PaymentResult> ProcessPayment(PaymentRequest request);
}

public class CreditCardPayment : IPaymentProcessor
{
    public Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        // Implementation of credit card payment processing
    }
}

public class PayPalPayment : IPaymentProcessor
{
    public Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        // Implementation of PayPal payment processing
    }
}
```

### Best Practices
1. **Overloading**
   - Use overloading for different constructors
   - Keep methods focused
   - Use proper access modifiers
   - Implement error handling
   - Log important operations

2. **Overriding**
   - Use override keyword
   - Call base methods when appropriate
   - Maintain Liskov Substitution Principle
   - Document behavior changes

3. **Access Control**
   - Use protected for inherited members
   - Keep private members private
   - Use sealed to prevent further inheritance
   - Implement proper encapsulation

4. **Code Organization**
   - Group related classes
   - Use namespaces effectively
   - Follow naming conventions
   - Document inheritance relationships

### Anti-patterns
1. **Overloading**
   - Too many overloaded methods
   - Poor error handling
   - Missing validation
   - Inconsistent state

2. **Overriding**
   - Breaking base class contract
   - Not calling base methods
   - Overriding without virtual
   - Poor documentation

3. **Access Control**
   - Exposing protected members
   - Overusing protected
   - Not using sealed
   - Poor encapsulation

4. **Code Organization**
   - Unrelated classes in hierarchy
   - Poor namespace organization
   - Inconsistent naming
   - Missing documentation

### Follow-up Questions

1. **Q: How do you handle overloading in an ecommerce system?**
   - **Answer**: Use overloading for different constructors and methods.
   - **Example**:
     ```csharp
     public class Order
     {
         private readonly List<OrderItem> _items;
         private decimal _total;
         
         public Order()
         {
             _items = new List<OrderItem>();
             _total = 0;
         }
         
         public void AddItem(Product product, int quantity)
         {
             // Encapsulated logic for adding items
             var item = new OrderItem(product, quantity);
             _items.Add(item);
             RecalculateTotal();
         }
         
         private void RecalculateTotal()
         {
             _total = _items.Sum(item => item.Subtotal);
         }
         
         public decimal GetTotal() => _total;
     }
     ```
   - **Best Practice**: Use overloading for different constructors and methods.

2. **Q: When should you use overloading vs overriding?**
   - **Answer**: Use overloading for different constructors and methods, and overriding for different implementations of the same method.
   - **Example**:
     ```csharp
     public interface IPaymentProcessor
     {
         Task<PaymentResult> ProcessPayment(PaymentRequest request);
     }

     public class CreditCardPayment : IPaymentProcessor
     {
         public Task<PaymentResult> ProcessPayment(PaymentRequest request)
         {
             // Implementation of credit card payment processing
         }
     }

     public class PayPalPayment : IPaymentProcessor
     {
         public Task<PaymentResult> ProcessPayment(PaymentRequest request)
         {
             // Implementation of PayPal payment processing
         }
     }
     ```
   - **Best Practice**: Use overloading for different constructors and methods, and overriding for different implementations of the same method.

### Key Takeaways
1. Overloading enables different constructors and methods
2. Overriding enables different implementations of the same method
3. Proper encapsulation
4. Follow Liskov Substitution Principle
5. Document inheritance relationships

### Additional Resources
- [C# Overloading](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/overloading)
- [C# Overriding](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/inheritance)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 8. Sealed Keyword

### Core Concept
The sealed keyword is used to prevent further inheritance of a class or a method.

### Real-world Example
In an ecommerce system, the sealed keyword is used for:
- Preventing further inheritance of a class
- Preventing further overriding of a method

### Code Example
```csharp
// Sealed class
public sealed class Order
{
    private readonly List<OrderItem> _items;
    private decimal _total;
    
    public Order()
    {
        _items = new List<OrderItem>();
        _total = 0;
    }
    
    public void AddItem(Product product, int quantity)
    {
        // Encapsulated logic for adding items
        var item = new OrderItem(product, quantity);
        _items.Add(item);
        RecalculateTotal();
    }
    
    private void RecalculateTotal()
    {
        _total = _items.Sum(item => item.Subtotal);
    }
    
    public decimal GetTotal() => _total;
}

// Sealed method
public interface IPaymentProcessor
{
    Task<PaymentResult> ProcessPayment(PaymentRequest request);
}

public class CreditCardPayment : IPaymentProcessor
{
    public Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        // Implementation of credit card payment processing
    }
}

public class PayPalPayment : IPaymentProcessor
{
    public Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        // Implementation of PayPal payment processing
    }
}
```

### Best Practices
1. **Use Sealed Classes for Preventing Inheritance**
   - Use sealed classes for preventing further inheritance
   - Keep inheritance hierarchy shallow
   - Use interfaces for multiple inheritance

2. **Use Sealed Methods for Preventing Overriding**
   - Use sealed methods for preventing further overriding
   - Keep methods focused
   - Use proper access modifiers
   - Implement error handling

3. **Access Control**
   - Use protected for inherited members
   - Keep private members private
   - Use sealed to prevent further inheritance
   - Implement proper encapsulation

4. **Code Organization**
   - Group related classes
   - Use namespaces effectively
   - Follow naming conventions
   - Document inheritance relationships

### Anti-patterns
1. **Sealed Classes**
   - Too many sealed classes
   - Over-abstraction
   - Leaky abstractions
   - Tight coupling

2. **Sealed Methods**
   - Too many sealed methods
   - Over-abstraction
   - Leaky abstractions
   - Tight coupling

3. **Access Control**
   - Exposing protected members
   - Overusing protected
   - Not using sealed
   - Poor encapsulation

4. **Code Organization**
   - Unrelated classes in hierarchy
   - Poor namespace organization
   - Inconsistent naming
   - Missing documentation

### Follow-up Questions

1. **Q: How do you handle sealed classes in an ecommerce system?**
   - **Answer**: Use sealed classes for preventing further inheritance.
   - **Example**:
     ```csharp
     public sealed class Order
     {
         private readonly List<OrderItem> _items;
         private decimal _total;
         
         public Order()
         {
             _items = new List<OrderItem>();
             _total = 0;
         }
         
         public void AddItem(Product product, int quantity)
         {
             // Encapsulated logic for adding items
             var item = new OrderItem(product, quantity);
             _items.Add(item);
             RecalculateTotal();
         }
         
         private void RecalculateTotal()
         {
             _total = _items.Sum(item => item.Subtotal);
         }
         
         public decimal GetTotal() => _total;
     }
     ```
   - **Best Practice**: Use sealed classes for preventing further inheritance.

2. **Q: When should you use sealed classes vs sealed methods?**
   - **Answer**: Use sealed classes for preventing further inheritance and sealed methods for preventing further overriding.
   - **Example**:
     ```csharp
     public sealed class Order
     {
         private readonly List<OrderItem> _items;
         private decimal _total;
         
         public Order()
         {
             _items = new List<OrderItem>();
             _total = 0;
         }
         
         public void AddItem(Product product, int quantity)
         {
             // Encapsulated logic for adding items
             var item = new OrderItem(product, quantity);
             _items.Add(item);
             RecalculateTotal();
         }
         
         private void RecalculateTotal()
         {
             _total = _items.Sum(item => item.Subtotal);
         }
         
         public decimal GetTotal() => _total;
     }
     ```
   - **Best Practice**: Use sealed classes for preventing further inheritance and sealed methods for preventing further overriding.

### Key Takeaways
1. Use sealed classes for preventing further inheritance
2. Use sealed methods for preventing further overriding
3. Proper encapsulation
4. Follow Liskov Substitution Principle
5. Document inheritance relationships

### Additional Resources
- [C# Sealed Classes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/sealed-classes-and-class-members)
- [C# Sealed Methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/sealed-methods)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

## 9. Access Modifiers

### Core Concept
Access modifiers control the visibility of classes, methods, and other members.

### Real-world Example
In an ecommerce system, access modifiers are used for:
- Encapsulation
- Security
- Code organization

### Code Example
```csharp
// Access modifiers
public class Order
{
    private readonly List<OrderItem> _items;
    private decimal _total;
    
    public Order()
    {
        _items = new List<OrderItem>();
        _total = 0;
    }
    
    public void AddItem(Product product, int quantity)
    {
        // Encapsulated logic for adding items
        var item = new OrderItem(product, quantity);
        _items.Add(item);
        RecalculateTotal();
    }
    
    private void RecalculateTotal()
    {
        _total = _items.Sum(item => item.Subtotal);
    }
    
    public decimal GetTotal() => _total;
}

// Encapsulation
public interface IPaymentProcessor
{
    Task<PaymentResult> ProcessPayment(PaymentRequest request);
}

// Inheritance
public abstract class Product
{
    public string Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class PhysicalProduct : Product
{
    public double Weight { get; set; }
    public string Dimensions { get; set; }
}

// Polymorphism
public interface IShippingCalculator
{
    decimal CalculateShippingCost(Order order);
}

public class StandardShipping : IShippingCalculator
{
    public decimal CalculateShippingCost(Order order) => 5.99m;
}

public class ExpressShipping : IShippingCalculator
{
    public decimal CalculateShippingCost(Order order) => 15.99m;
}
```

### Best Practices
1. **Use Private Fields**
   - Use private fields with public properties
   - Implement proper validation in setters
   - Use readonly collections for collections
   - Implement proper immutability where possible

2. **Use Public Properties**
   - Expose data through properties
   - Implement proper validation
   - Use readonly collections

3. **Use Protected Members**
   - Use protected for inherited members
   - Keep private members private
   - Use sealed to prevent further inheritance
   - Implement proper encapsulation

4. **Use Private Constructors**
   - Use private constructors for preventing instantiation
   - Implement proper encapsulation

### Anti-patterns
1. **Exposing Internal State**
   - Public fields
   - Mutable collections
   - Global state
   - Exposed internals

2. **Using Public Fields**
   - Public fields
   - Mutable collections
   - Global state
   - Exposed internals

3. **Mutable Collections**
   - Mutable collections
   - Global state
   - Exposed internals
   - Exposing internal state

4. **Global State**
   - Global state
   - Exposed internals
   - Mutable collections
   - Exposing internal state

### Follow-up Questions

1. **Q: How do you handle access modifiers in an ecommerce system?**
   - **Answer**: Use private fields with public properties, implement proper validation in setters, use readonly collections for collections, and implement proper immutability where possible.
   - **Example**:
     ```csharp
     public class Order
     {
         private readonly List<OrderItem> _items;
         private decimal _total;
         
         public Order()
         {
             _items = new List<OrderItem>();
             _total = 0;
         }
         
         public void AddItem(Product product, int quantity)
         {
             // Encapsulated logic for adding items
             var item = new OrderItem(product, quantity);
             _items.Add(item);
             RecalculateTotal();
         }
         
         private void RecalculateTotal()
         {
             _total = _items.Sum(item => item.Subtotal);
         }
         
         public decimal GetTotal() => _total;
     }
     ```
   - **Best Practice**: Use private fields with public properties, implement proper validation in setters, use readonly collections for collections, and implement proper immutability where possible.

2. **Q: When should you use public vs private access modifiers?**
   - **Answer**: Use public access modifiers for exposing data and methods, and private access modifiers for hiding implementation details.
   - **Example**:
     ```csharp
     public class Order
     {
         private readonly List<OrderItem> _items;
         private decimal _total;
         
         public Order()
         {
             _items = new List<OrderItem>();
             _total = 0;
         }
         
         public void AddItem(Product product, int quantity)
         {
             // Encapsulated logic for adding items
             var item = new OrderItem(product, quantity);
             _items.Add(item);
             RecalculateTotal();
         }
         
         private void RecalculateTotal()
         {
             _total = _items.Sum(item => item.Subtotal);
         }
         
         public decimal GetTotal() => _total;
     }
     ```
   - **Best Practice**: Use public access modifiers for exposing data and methods, and private access modifiers for hiding implementation details.

### Key Takeaways
1. Access modifiers help manage visibility and security
2. Use private fields with public properties
3. Implement proper validation in setters
4. Use readonly collections for collections
5. Implement proper immutability where possible

### Additional Resources
- [C# Access Modifiers](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [C# Design Patterns](https://refactoring.guru/design-patterns/csharp)

## 10. Class vs Struct

### Core Concept
A class is a reference type, and a struct is a value type. Classes are stored on the heap, while structs are stored on the stack.

### Real-world Example
In an ecommerce system:
- **Class**: Order, Product
- **Struct**: OrderItem

### Code Example
```csharp
// Class
public class Order
{
    private readonly List<OrderItem> _items;
    private decimal _total;
    
    public Order()
    {
        _items = new List<OrderItem>();
        _total = 0;
    }
    
    public void AddItem(Product product, int quantity)
    {
        // Encapsulated logic for adding items
        var item = new OrderItem(product, quantity);
        _items.Add(item);
        RecalculateTotal();
    }
    
    private void RecalculateTotal()
    {
        _total = _items.Sum(item => item.Subtotal);
    }
    
    public decimal GetTotal() => _total;
}

// Struct
public struct OrderItem
{
    public Product Product { get; }
    public int Quantity { get; }
    public decimal UnitPrice { get; }
    
    public OrderItem(Product product, int quantity)
    {
        if (product == null)
            throw new ArgumentNullException(nameof(product));
        if (quantity <= 0)
            throw new ArgumentException("Quantity must be positive", nameof(quantity));
        
        Product = product;
        Quantity = quantity;
        UnitPrice = product.Price;
    }
    
    public decimal Subtotal => UnitPrice * Quantity;
}
```

### Best Practices
1. **Use Classes for Reference Types**
   - Use classes for reference types
   - Store on the heap
   - Support inheritance
   - Support polymorphism

2. **Use Structs for Value Types**
   - Use structs for value types
   - Store on the stack
   - Immutable by default
   - Support simple types

3. **Avoid Large Structs**
   - Avoid large structs
   - Use classes instead
   - Implement proper encapsulation

4. **Avoid Reference Types**
   - Avoid reference types
   - Use structs instead
   - Implement proper encapsulation

### Anti-patterns
1. **Using Reference Types**
   - Using reference types
   - Storing on the heap
   - Supporting inheritance
   - Supporting polymorphism

2. **Using Value Types**
   - Using value types
   - Storing on the stack
   - Immutable by default
   - Supporting simple types

3. **Large Structs**
   - Large structs
   - Storing on the stack
   - Immutable by default
   - Supporting simple types

4. **Avoiding Reference Types**
   - Avoiding reference types
   - Using structs instead
   - Implementing proper encapsulation

### Follow-up Questions

1. **Q: How do you handle class vs struct in an ecommerce system?**
   - **Answer**: Use classes for reference types and structs for value types.
   - **Example**:
     ```csharp
     public class Order
     {
         private readonly List<OrderItem> _items;
         private decimal _total;
         
         public Order()
         {
             _items = new List<OrderItem>();
             _total = 0;
         }
         
         public void AddItem(Product product, int quantity)
         {
             // Encapsulated logic for adding items
             var item = new OrderItem(product, quantity);
             _items.Add(item);
             RecalculateTotal();
         }
         
         private void RecalculateTotal()
         {
             _total = _items.Sum(item => item.Subtotal);
         }
         
         public decimal GetTotal() => _total;
     }

     public struct OrderItem
     {
         public Product Product { get; }
         public int Quantity { get; }
         public decimal UnitPrice { get; }
         
         public OrderItem(Product product, int quantity)
         {
             if (product == null)
                 throw new ArgumentNullException(nameof(product));
             if (quantity <= 0)
                 throw new ArgumentException("Quantity must be positive", nameof(quantity));
                 
             Product = product;
             Quantity = quantity;
             UnitPrice = product.Price;
         }
         
         public decimal Subtotal => UnitPrice * Quantity;
     }
     ```
   - **Best Practice**: Use classes for reference types and structs for value types.

2. **Q: When should you use class vs struct?**
   - **Answer**: Use classes for reference types and structs for value types.
   - **Example**:
     ```csharp
     public class Order
     {
         private readonly List<OrderItem> _items;
         private decimal _total;
         
         public Order()
         {
             _items = new List<OrderItem>();
             _total = 0;
         }
         
         public void AddItem(Product product, int quantity)
         {
             // Encapsulated logic for adding items
             var item = new OrderItem(product, quantity);
             _items.Add(item);
             RecalculateTotal();
         }
         
         private void RecalculateTotal()
         {
             _total = _items.Sum(item => item.Subtotal);
         }
         
         public decimal GetTotal() => _total;
     }

     public struct OrderItem
     {
         public Product Product { get; }
         public int Quantity { get; }
         public decimal UnitPrice { get; }
         
         public OrderItem(Product product, int quantity)
         {
             if (product == null)
                 throw new ArgumentNullException(nameof(product));
             if (quantity <= 0)
                 throw new ArgumentException("Quantity must be positive", nameof(quantity));
                 
             Product = product;
             Quantity = quantity;
             UnitPrice = product.Price;
         }
         
         public decimal Subtotal => UnitPrice * Quantity;
     }
     ```
   - **Best Practice**: Use classes for reference types and structs for value types.

### Key Takeaways
1. Use classes for reference types and structs for value types
2. Store on the heap for classes and stack for structs
3. Immutable by default for structs
4. Support inheritance and polymorphism for classes
5. Implement proper encapsulation

### Additional Resources
- [C# Classes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/classes)
- [C# Structs](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/structs)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [C# Design Patterns](https://refactoring.guru/design-patterns/csharp)

## Conclusion

This comprehensive guide covers essential OOP concepts in C# with ecommerce-focused examples, detailed explanations, best practices, and real-world scenarios. By understanding these concepts, you can create maintainable and scalable ecommerce systems.

### Key Takeaways
1. OOP principles help create maintainable and scalable ecommerce systems
2. Proper encapsulation is crucial for security and data integrity
3. Abstraction helps manage complexity in large systems
4. Inheritance should be used carefully and prefer composition
5. Polymorphism enables flexible and extensible code

### Additional Resources
- [Microsoft C# Documentation](https://docs.microsoft.com/en-us/dotnet/csharp/)
- [SOLID Principles in C#](https://www.dofactory.com/net/solid-principles)
- [Design Patterns in C#](https://refactoring.guru/design-patterns/csharp)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

Thank you for reading this comprehensive guide on Object-Oriented Programming in C# for the ecommerce domain. I hope you found it helpful and informative. If you have any questions or need further clarification, feel free to ask.

Happy coding! 