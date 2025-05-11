# Advanced Pattern Matching & Nullable Reference Types

This guide covers advanced pattern matching features and nullable reference types in C# 8+ with practical examples and interview questions.

## Table of Contents
1. [Pattern Matching](#1-pattern-matching)
2. [Nullable Reference Types](#2-nullable-reference-types)
3. [Pattern Matching with Nullable Types](#3-pattern-matching-with-nullable-types)
4. [Migration Strategies](#4-migration-strategies)

## 1. Pattern Matching

### Question
How do you use advanced pattern matching features in C# 8+?

### Suggested Answer
C# 8+ introduces several pattern matching features that make code more concise and readable.

```csharp
using System;

public class OrderProcessor
{
    // Property pattern
    public string GetOrderStatus(Order order) => order switch
    {
        { Status: OrderStatus.Pending } => "Order is pending",
        { Status: OrderStatus.Processing, PaymentStatus: PaymentStatus.Paid } => "Order is being processed",
        { Status: OrderStatus.Shipped, TrackingNumber: not null } => $"Order is shipped: {order.TrackingNumber}",
        { Status: OrderStatus.Delivered } => "Order is delivered",
        _ => "Unknown order status"
    };

    // Tuple pattern
    public decimal CalculateDiscount(Order order) => (order.CustomerType, order.TotalAmount) switch
    {
        (CustomerType.Premium, var amount) when amount > 1000 => amount * 0.1m,
        (CustomerType.Regular, var amount) when amount > 500 => amount * 0.05m,
        _ => 0m
    };

    // Positional pattern with deconstruction
    public string GetShippingStatus(Order order)
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

    // Type pattern with when clause
    public string ProcessPayment(Payment payment) => payment switch
    {
        CreditCardPayment ccp when ccp.IsValid => $"Processing credit card payment: {ccp.CardNumber}",
        PayPalPayment ppp when ppp.IsVerified => $"Processing PayPal payment: {ppp.Email}",
        BankTransferPayment btp => $"Processing bank transfer: {btp.AccountNumber}",
        _ => "Unknown payment type"
    };

    // Relational pattern (C# 9+)
    public string GetOrderPriority(Order order) => order.TotalAmount switch
    {
        < 100 => "Low",
        >= 100 and < 500 => "Medium",
        >= 500 and < 1000 => "High",
        >= 1000 => "Premium"
    };
}

// Supporting types
public enum OrderStatus { Pending, Processing, Shipped, Delivered }
public enum PaymentStatus { Pending, Paid, Failed }
public enum CustomerType { Regular, Premium }

public class Order
{
    public int Id { get; set; }
    public OrderStatus Status { get; set; }
    public PaymentStatus PaymentStatus { get; set; }
    public string? TrackingNumber { get; set; }
    public CustomerType CustomerType { get; set; }
    public decimal TotalAmount { get; set; }
    public bool IsPaid { get; set; }
    public bool IsShipped { get; set; }
}

public abstract class Payment { }
public class CreditCardPayment : Payment 
{ 
    public string CardNumber { get; set; } = string.Empty;
    public bool IsValid { get; set; }
}
public class PayPalPayment : Payment 
{ 
    public string Email { get; set; } = string.Empty;
    public bool IsVerified { get; set; }
}
public class BankTransferPayment : Payment 
{ 
    public string AccountNumber { get; set; } = string.Empty;
}
```

### Follow-up Q&A

1. **Q: What's the difference between switch statement and switch expression?**
   - **A**: 
     - Switch expression is more concise and returns a value
     - Switch expression uses `=>` instead of `case` and `break`
     - Switch expression requires exhaustive pattern matching
     - Switch expression can be used in expression contexts

2. **Q: When should you use pattern matching vs traditional if-else?**
   - **A**: Use pattern matching when:
     - You need to match against multiple conditions
     - You want to extract and use values in the same expression
     - You need to match against types and properties
     - You want more concise and readable code

## 2. Nullable Reference Types

### Question
How do you use nullable reference types to prevent null reference exceptions?

### Suggested Answer
Nullable reference types help catch potential null reference issues at compile time.

```csharp
#nullable enable

public class Customer
{
    // Non-nullable properties must be initialized
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public List<Order> Orders { get; set; } = new();

    // Nullable properties
    public string? PhoneNumber { get; set; }
    public Address? ShippingAddress { get; set; }

    // Nullable parameter
    public void UpdatePhoneNumber(string? newNumber)
    {
        // Null check required
        if (newNumber != null)
        {
            PhoneNumber = newNumber;
        }
    }

    // Non-nullable parameter
    public void UpdateEmail(string newEmail)
    {
        // No null check needed
        Email = newEmail;
    }

    // Null-forgiving operator (!)
    public string GetFormattedPhoneNumber()
    {
        // We know PhoneNumber is not null here
        return FormatPhoneNumber(PhoneNumber!);
    }

    // Null-conditional operator (?.)
    public string GetShippingCity()
    {
        return ShippingAddress?.City ?? "No shipping address";
    }

    // Null-coalescing operator (??)
    public string GetDisplayName()
    {
        return Name ?? "Unknown Customer";
    }
}

public class Address
{
    public string Street { get; set; } = string.Empty;
    public string City { get; set; } = string.Empty;
    public string Country { get; set; } = string.Empty;
}

// Nullable annotation context
#nullable disable
public class LegacyCustomer
{
    // No nullable warnings
    public string Name { get; set; }
    public string Email { get; set; }
}
#nullable enable
```

### Follow-up Q&A

1. **Q: What's the difference between nullable reference types and nullable value types?**
   - **A**: 
     - Nullable reference types (`string?`) are a compile-time feature
     - Nullable value types (`int?`) are a runtime feature
     - Nullable reference types help prevent null reference exceptions
     - Nullable value types are actual types that can be null

2. **Q: When should you use the null-forgiving operator (!)?**
   - **A**: Use the null-forgiving operator when:
     - You know a nullable reference is not null
     - You're working with legacy code
     - You're using a third-party library
     - You're in a context where null checks are performed elsewhere

## 3. Pattern Matching with Nullable Types

### Question
How do you combine pattern matching with nullable reference types?

### Suggested Answer
Pattern matching can be used to safely handle nullable types and provide type-safe code.

```csharp
#nullable enable

public class OrderValidator
{
    public string ValidateOrder(Order? order)
    {
        return order switch
        {
            null => "Order cannot be null",
            { Customer: null } => "Order must have a customer",
            { Items: null or { Count: 0 } } => "Order must have items",
            { ShippingAddress: null } => "Order must have a shipping address",
            { TotalAmount: <= 0 } => "Order total must be greater than zero",
            _ => "Order is valid"
        };
    }

    public string ProcessCustomer(Customer? customer)
    {
        return customer switch
        {
            null => "No customer",
            { Name: null or "" } => "Customer name is required",
            { Email: null or "" } => "Customer email is required",
            { PhoneNumber: not null } => $"Customer phone: {customer.PhoneNumber}",
            _ => $"Customer: {customer.Name}"
        };
    }

    public string GetShippingInfo(Order order)
    {
        return (order.ShippingAddress, order.TrackingNumber) switch
        {
            (null, null) => "No shipping information",
            (null, _) => "Missing shipping address",
            (_, null) => "No tracking number",
            (var address, var tracking) => $"Shipping to {address.City}, Tracking: {tracking}"
        };
    }
}
```

### Follow-up Q&A

1. **Q: How do you handle nullable types in switch expressions?**
   - **A**: 
     - Use the `null` pattern to match null values
     - Use property patterns to check for null properties
     - Use the `not` pattern to check for non-null values
     - Use tuple patterns to match multiple nullable values

2. **Q: What's the difference between `is` pattern and switch expression?**
   - **A**: 
     - `is` pattern is used for single condition checks
     - Switch expression is used for multiple condition checks
     - `is` pattern can be used in if statements
     - Switch expression must be exhaustive

## 4. Migration Strategies

### Question
How do you migrate legacy code to use nullable reference types?

### Suggested Answer
Migrate code gradually using nullable annotations and proper null handling.

```csharp
#nullable enable

public class LegacyCodeMigration
{
    // Step 1: Add nullable annotations
    public string? GetCustomerName(Customer? customer)
    {
        return customer?.Name;
    }

    // Step 2: Use null-conditional operators
    public string GetCustomerInfo(Customer? customer)
    {
        return customer?.Email ?? "No email";
    }

    // Step 3: Add null checks
    public void UpdateCustomer(Customer? customer, string? newName)
    {
        if (customer == null)
        {
            throw new ArgumentNullException(nameof(customer));
        }

        if (newName != null)
        {
            customer.Name = newName;
        }
    }

    // Step 4: Use pattern matching
    public string ProcessCustomer(Customer? customer)
    {
        return customer switch
        {
            null => "No customer",
            { Name: null or "" } => "Invalid customer name",
            { Email: null or "" } => "Invalid customer email",
            _ => $"Valid customer: {customer.Name}"
        };
    }
}

// Step 5: Update interfaces
public interface ICustomerService
{
    Customer? GetCustomer(int id);
    void UpdateCustomer(Customer customer);
    IEnumerable<Customer> GetAllCustomers();
}

// Step 6: Update implementation
public class CustomerService : ICustomerService
{
    private readonly List<Customer> _customers = new();

    public Customer? GetCustomer(int id)
    {
        return _customers.FirstOrDefault(c => c.Id == id);
    }

    public void UpdateCustomer(Customer customer)
    {
        if (customer == null)
        {
            throw new ArgumentNullException(nameof(customer));
        }

        var existing = _customers.FirstOrDefault(c => c.Id == customer.Id);
        if (existing != null)
        {
            _customers.Remove(existing);
        }
        _customers.Add(customer);
    }

    public IEnumerable<Customer> GetAllCustomers()
    {
        return _customers;
    }
}
```

### Follow-up Q&A

1. **Q: What are the steps to enable nullable reference types in a project?**
   - **A**: 
     - Add `<Nullable>enable</Nullable>` to project file
     - Add `#nullable enable` to files
     - Fix nullable warnings
     - Add proper null checks
     - Update interfaces and implementations

2. **Q: How do you handle third-party libraries with nullable reference types?**
   - **A**: 
     - Use null-forgiving operator when necessary
     - Add proper null checks
     - Consider creating wrapper classes
     - Document assumptions about nullability

## Additional Resources

- [Microsoft Docs: Pattern Matching](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns)
- [Microsoft Docs: Nullable Reference Types](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references)
- [Microsoft Docs: C# 8.0 Features](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8)
- [Microsoft Docs: C# 9.0 Features](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9) 