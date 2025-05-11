# Records & Init-only Properties

This guide covers records and init-only properties in C# 9+ with practical examples and interview questions.

## Table of Contents
1. [Records Basics](#1-records-basics)
2. [Init-only Properties](#2-init-only-properties)
3. [Records with Inheritance](#3-records-with-inheritance)
4. [Records Best Practices](#4-records-best-practices)

## 1. Records Basics

### Question
How do you use records in C# 9+ for immutable data types?

### Suggested Answer
Records provide a concise way to create immutable reference types with value-based equality.

```csharp
using System;

// Positional record (primary constructor)
public record Product(string Name, decimal Price, string Category);

// Record with additional members
public record Order(int Id, string CustomerName, decimal Total)
{
    // Additional property
    public DateTime OrderDate { get; init; } = DateTime.UtcNow;

    // Additional method
    public bool IsHighValue => Total > 1000;

    // Deconstructor
    public void Deconstruct(out int id, out string customerName, out decimal total)
    {
        id = Id;
        customerName = CustomerName;
        total = Total;
    }
}

// Record with validation
public record Customer
{
    public string Name { get; init; } = string.Empty;
    public string Email { get; init; } = string.Empty;
    public int Age { get; init; }

    public Customer(string name, string email, int age)
    {
        if (string.IsNullOrEmpty(name))
            throw new ArgumentException("Name cannot be empty", nameof(name));
        if (string.IsNullOrEmpty(email))
            throw new ArgumentException("Email cannot be empty", nameof(email));
        if (age < 0)
            throw new ArgumentException("Age cannot be negative", nameof(age));

        Name = name;
        Email = email;
        Age = age;
    }
}

// Usage
public class RecordExample
{
    public void DemonstrateRecords()
    {
        // Create records
        var product = new Product("Laptop", 999.99m, "Electronics");
        var order = new Order(1, "John", 1500.00m);
        var customer = new Customer("John Doe", "john@example.com", 30);

        // With expression (creates a new instance with modified properties)
        var updatedProduct = product with { Price = 899.99m };
        var updatedOrder = order with { Total = 1600.00m };

        // Value-based equality
        var product2 = new Product("Laptop", 999.99m, "Electronics");
        Console.WriteLine(product == product2); // True

        // Deconstruction
        var (name, price, category) = product;
        var (id, customerName, total) = order;

        // ToString() implementation
        Console.WriteLine(product); // Product { Name = Laptop, Price = 999.99, Category = Electronics }
    }
}
```

### Follow-up Q&A

1. **Q: What's the difference between a record and a class?**
   - **A**: 
     - Records are immutable by default
     - Records have value-based equality
     - Records have built-in ToString() implementation
     - Records support with expressions
     - Records can use positional syntax

2. **Q: When should you use records?**
   - **A**: Use records when:
     - You need immutable data types
     - You want value-based equality
     - You're working with DTOs or value objects
     - You need to create copies with slight modifications

## 2. Init-only Properties

### Question
How do you use init-only properties to create immutable objects?

### Suggested Answer
Init-only properties allow you to set values only during object initialization.

```csharp
using System;
using System.Collections.Generic;

public class OrderItem
{
    // Init-only properties
    public int ProductId { get; init; }
    public string ProductName { get; init; } = string.Empty;
    public decimal UnitPrice { get; init; }
    public int Quantity { get; init; }
    public List<string> Tags { get; init; } = new();

    // Computed property
    public decimal Total => UnitPrice * Quantity;

    // Method that doesn't modify state
    public string GetDescription() => $"{Quantity} x {ProductName} at {UnitPrice:C}";
}

public class OrderBuilder
{
    public OrderItem CreateOrderItem(
        int productId,
        string productName,
        decimal unitPrice,
        int quantity)
    {
        return new OrderItem
        {
            ProductId = productId,
            ProductName = productName,
            UnitPrice = unitPrice,
            Quantity = quantity,
            Tags = new List<string> { "New" }
        };
    }
}

// Usage
public class InitOnlyExample
{
    public void DemonstrateInitOnly()
    {
        var builder = new OrderBuilder();
        var item = builder.CreateOrderItem(1, "Laptop", 999.99m, 2);

        // This would cause a compile error
        // item.Quantity = 3;

        Console.WriteLine(item.GetDescription());
        Console.WriteLine($"Total: {item.Total:C}");
    }
}
```

### Follow-up Q&A

1. **Q: What's the difference between init-only and readonly properties?**
   - **A**: 
     - Init-only properties can be set during object initialization
     - Readonly properties can only be set in the constructor
     - Init-only properties work with object initializers
     - Readonly properties are more restrictive

2. **Q: Can you modify collections in init-only properties?**
   - **A**: 
     - The property reference cannot be changed
     - The collection contents can be modified
     - Use immutable collections for true immutability
     - Consider using `IReadOnlyCollection<T>` for the property type

## 3. Records with Inheritance

### Question
How do you use inheritance with records in C#?

### Suggested Answer
Records can inherit from other records, but there are some important considerations.

```csharp
using System;

// Base record
public record Payment(decimal Amount, string Currency)
{
    public virtual string GetPaymentType() => "Unknown";
}

// Derived record
public record CreditCardPayment(
    decimal Amount,
    string Currency,
    string CardNumber,
    string ExpiryDate) : Payment(Amount, Currency)
{
    public override string GetPaymentType() => "Credit Card";
}

// Another derived record
public record PayPalPayment(
    decimal Amount,
    string Currency,
    string Email) : Payment(Amount, Currency)
{
    public override string GetPaymentType() => "PayPal";
}

// Usage
public class InheritanceExample
{
    public void DemonstrateInheritance()
    {
        var creditCard = new CreditCardPayment(100.00m, "USD", "1234-5678", "12/25");
        var paypal = new PayPalPayment(200.00m, "EUR", "user@example.com");

        // With expression works with inheritance
        var updatedCreditCard = creditCard with { Amount = 150.00m };

        // Value-based equality works with inheritance
        var sameCreditCard = new CreditCardPayment(100.00m, "USD", "1234-5678", "12/25");
        Console.WriteLine(creditCard == sameCreditCard); // True

        // Type checking
        Console.WriteLine(creditCard is Payment); // True
        Console.WriteLine(creditCard is CreditCardPayment); // True
    }
}
```

### Follow-up Q&A

1. **Q: What are the limitations of record inheritance?**
   - **A**: 
     - Records can only inherit from other records
     - Records cannot inherit from classes
     - Classes cannot inherit from records
     - Records cannot be abstract

2. **Q: How do you handle equality with inherited records?**
   - **A**: 
     - Value-based equality includes all properties
     - Equality considers the actual type
     - With expressions preserve the type
     - Deconstruction works with inherited properties

## 4. Records Best Practices

### Question
What are the best practices for using records in C#?

### Suggested Answer
Follow these best practices to effectively use records in your code.

```csharp
using System;
using System.Collections.Immutable;

// Use positional records for simple DTOs
public record ProductDto(string Name, decimal Price);

// Use full record syntax for complex types
public record Order
{
    // Use init-only for immutability
    public int Id { get; init; }
    public string CustomerName { get; init; } = string.Empty;
    public decimal Total { get; init; }
    public DateTime OrderDate { get; init; } = DateTime.UtcNow;

    // Use immutable collections
    public ImmutableList<OrderItem> Items { get; init; } = ImmutableList<OrderItem>.Empty;

    // Add validation in constructor
    public Order(int id, string customerName, decimal total, ImmutableList<OrderItem> items)
    {
        if (id <= 0)
            throw new ArgumentException("Id must be positive", nameof(id));
        if (string.IsNullOrEmpty(customerName))
            throw new ArgumentException("Customer name cannot be empty", nameof(customerName));
        if (total < 0)
            throw new ArgumentException("Total cannot be negative", nameof(total));
        if (items == null)
            throw new ArgumentNullException(nameof(items));

        Id = id;
        CustomerName = customerName;
        Total = total;
        Items = items;
    }

    // Add computed properties
    public bool IsHighValue => Total > 1000;

    // Add methods that don't modify state
    public string GetSummary() => $"Order {Id} for {CustomerName}: {Total:C}";
}

// Use records for value objects
public record Money
{
    public decimal Amount { get; init; }
    public string Currency { get; init; } = string.Empty;

    public Money(decimal amount, string currency)
    {
        if (amount < 0)
            throw new ArgumentException("Amount cannot be negative", nameof(amount));
        if (string.IsNullOrEmpty(currency))
            throw new ArgumentException("Currency cannot be empty", nameof(currency));

        Amount = amount;
        Currency = currency;
    }

    public static Money operator +(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException("Cannot add money with different currencies");

        return new Money(a.Amount + b.Amount, a.Currency);
    }
}
```

### Follow-up Q&A

1. **Q: When should you use positional records vs full record syntax?**
   - **A**: 
     - Use positional records for simple DTOs
     - Use full syntax when you need:
       - Additional properties
       - Custom validation
       - Complex initialization
       - Additional methods

2. **Q: How do you handle collections in records?**
   - **A**: 
     - Use immutable collections
     - Initialize collections in the constructor
     - Use init-only properties
     - Consider using `IReadOnlyCollection<T>`

## Additional Resources

- [Microsoft Docs: Records](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
- [Microsoft Docs: Init-only Properties](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-9.0/init)
- [Microsoft Docs: C# 9.0 Features](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9)
- [Microsoft Docs: Immutable Collections](https://docs.microsoft.com/en-us/dotnet/standard/collections/thread-safe/immutable-collections) 