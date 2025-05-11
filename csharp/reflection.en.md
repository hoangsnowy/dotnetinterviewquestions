# Reflection & Attributes in C#

This guide covers advanced reflection concepts and custom attributes with practical examples and interview questions.

## Table of Contents
1. [Basic Reflection](#1-basic-reflection)
2. [Custom Attributes](#2-custom-attributes)
3. [Dynamic Instance Creation](#3-dynamic-instance-creation)
4. [Performance Considerations](#4-performance-considerations)

## 1. Basic Reflection

### Question
How do you use reflection to inspect and manipulate types at runtime?

### Suggested Answer
Reflection allows you to examine and interact with types, properties, and methods at runtime.

```csharp
using System;
using System.Reflection;
using System.Linq;

public class ReflectionExample
{
    public void InspectType(Type type)
    {
        // Get type information
        Console.WriteLine($"Type: {type.Name}");
        Console.WriteLine($"Is Class: {type.IsClass}");
        Console.WriteLine($"Is Interface: {type.IsInterface}");
        Console.WriteLine($"Base Type: {type.BaseType?.Name}");

        // Get properties
        var properties = type.GetProperties(BindingFlags.Public | BindingFlags.Instance);
        foreach (var prop in properties)
        {
            Console.WriteLine($"Property: {prop.Name}, Type: {prop.PropertyType.Name}");
        }

        // Get methods
        var methods = type.GetMethods(BindingFlags.Public | BindingFlags.Instance)
                         .Where(m => !m.IsSpecialName); // Exclude property getters/setters
        foreach (var method in methods)
        {
            Console.WriteLine($"Method: {method.Name}, Return Type: {method.ReturnType.Name}");
        }
    }

    public object? GetPropertyValue(object obj, string propertyName)
    {
        var property = obj.GetType().GetProperty(propertyName);
        return property?.GetValue(obj);
    }

    public void SetPropertyValue(object obj, string propertyName, object value)
    {
        var property = obj.GetType().GetProperty(propertyName);
        property?.SetValue(obj, value);
    }
}

// Usage
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }

    public void UpdatePrice(decimal newPrice)
    {
        Price = newPrice;
    }
}

var example = new ReflectionExample();
var product = new Product { Id = 1, Name = "Laptop", Price = 999.99m };

// Inspect type
example.InspectType(typeof(Product));

// Get property value
var price = example.GetPropertyValue(product, "Price");
Console.WriteLine($"Price: {price}");

// Set property value
example.SetPropertyValue(product, "Price", 899.99m);
Console.WriteLine($"New Price: {product.Price}");
```

### Follow-up Q&A

1. **Q: What are the different types of BindingFlags?**
   - **A**: BindingFlags control how reflection searches for members:
     - `Public/NonPublic`: Access level
     - `Instance/Static`: Member type
     - `DeclaredOnly`: Only declared members, not inherited
     - `FlattenHierarchy`: Include inherited members

2. **Q: How do you handle generic types with reflection?**
   - **A**: Use `GetGenericTypeDefinition()` and `MakeGenericType()`:
     ```csharp
     var listType = typeof(List<>);
     var stringListType = listType.MakeGenericType(typeof(string));
     var instance = Activator.CreateInstance(stringListType);
     ```

## 2. Custom Attributes

### Question
How do you create and use custom attributes in C#?

### Suggested Answer
Custom attributes provide metadata for types and members, useful for validation, serialization, and framework features.

```csharp
using System;

// Define custom attribute
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Property)]
public class TableAttribute : Attribute
{
    public string Name { get; }
    public TableAttribute(string name) => Name = name;
}

[AttributeUsage(AttributeTargets.Property)]
public class RequiredAttribute : Attribute { }

[AttributeUsage(AttributeTargets.Property)]
public class MaxLengthAttribute : Attribute
{
    public int Length { get; }
    public MaxLengthAttribute(int length) => Length = length;
}

// Use attributes
[Table("Products")]
public class Product
{
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; } = string.Empty;

    public decimal Price { get; set; }
}

// Read attributes
public class AttributeReader
{
    public string GetTableName(Type type)
    {
        var tableAttr = type.GetCustomAttribute<TableAttribute>();
        return tableAttr?.Name ?? type.Name;
    }

    public bool ValidateRequiredProperties(object obj)
    {
        var type = obj.GetType();
        var properties = type.GetProperties();

        foreach (var prop in properties)
        {
            if (prop.GetCustomAttribute<RequiredAttribute>() != null)
            {
                var value = prop.GetValue(obj);
                if (value == null || (value is string str && string.IsNullOrEmpty(str)))
                {
                    return false;
                }
            }
        }
        return true;
    }

    public bool ValidateMaxLength(object obj)
    {
        var type = obj.GetType();
        var properties = type.GetProperties();

        foreach (var prop in properties)
        {
            var maxLengthAttr = prop.GetCustomAttribute<MaxLengthAttribute>();
            if (maxLengthAttr != null && prop.PropertyType == typeof(string))
            {
                var value = (string?)prop.GetValue(obj);
                if (value?.Length > maxLengthAttr.Length)
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

1. **Q: What are the different AttributeTargets?**
   - **A**: AttributeTargets specify where attributes can be applied:
     - `Class/Struct/Interface`: Types
     - `Property/Field/Method`: Members
     - `Parameter/ReturnValue`: Method elements
     - `Assembly/Module`: Assembly-level

2. **Q: How do you create an attribute that can be used multiple times?**
   - **A**: Use `AllowMultiple = true` in AttributeUsage:
     ```csharp
     [AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
     public class AuthorizeAttribute : Attribute
     {
         public string Role { get; }
         public AuthorizeAttribute(string role) => Role = role;
     }
     ```

## 3. Dynamic Instance Creation

### Question
How do you create instances of types dynamically using reflection?

### Suggested Answer
Use `Activator.CreateInstance()` or `ConstructorInfo.Invoke()` to create instances at runtime.

```csharp
using System;
using System.Reflection;

public class DynamicInstanceCreator
{
    public T? CreateInstance<T>(params object[] args) where T : class
    {
        return Activator.CreateInstance(typeof(T), args) as T;
    }

    public object? CreateInstance(string typeName, params object[] args)
    {
        var type = Type.GetType(typeName);
        return type != null ? Activator.CreateInstance(type, args) : null;
    }

    public T? CreateInstanceWithConstructor<T>(params object[] args) where T : class
    {
        var type = typeof(T);
        var constructor = type.GetConstructor(args.Select(a => a.GetType()).ToArray());
        return constructor?.Invoke(args) as T;
    }
}

// Usage
public class Order
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public decimal Total { get; set; }

    public Order(int id, string customerName, decimal total)
    {
        Id = id;
        CustomerName = customerName;
        Total = total;
    }
}

var creator = new DynamicInstanceCreator();

// Create instance using Activator
var order1 = creator.CreateInstance<Order>(1, "John", 100.00m);

// Create instance using type name
var order2 = creator.CreateInstance("Namespace.Order", 2, "Jane", 200.00m);

// Create instance using constructor
var order3 = creator.CreateInstanceWithConstructor<Order>(3, "Bob", 300.00m);
```

### Follow-up Q&A

1. **Q: What's the difference between Activator.CreateInstance and ConstructorInfo.Invoke?**
   - **A**: 
     - `Activator.CreateInstance` is simpler but less flexible
     - `ConstructorInfo.Invoke` gives more control over constructor selection and parameter handling

2. **Q: How do you handle constructor parameters with different types?**
   - **A**: Use `GetConstructor` with parameter types:
     ```csharp
     var constructor = type.GetConstructor(new[] { typeof(int), typeof(string) });
     var instance = constructor?.Invoke(new object[] { 1, "Test" });
     ```

## 4. Performance Considerations

### Question
How do you optimize reflection usage to minimize performance impact?

### Suggested Answer
Use caching, compiled expressions, or emit IL to improve reflection performance.

```csharp
using System;
using System.Reflection;
using System.Linq.Expressions;
using System.Collections.Concurrent;

public class OptimizedReflection
{
    private readonly ConcurrentDictionary<string, Delegate> _getterCache = new();
    private readonly ConcurrentDictionary<string, Delegate> _setterCache = new();

    public Func<object, object?> CreateGetter(PropertyInfo property)
    {
        var key = $"{property.DeclaringType?.FullName}.{property.Name}";
        return (Func<object, object?>)_getterCache.GetOrAdd(key, _ =>
        {
            var instance = Expression.Parameter(typeof(object), "instance");
            var instanceCast = Expression.Convert(instance, property.DeclaringType!);
            var propertyAccess = Expression.Property(instanceCast, property);
            var convert = Expression.Convert(propertyAccess, typeof(object));
            return Expression.Lambda<Func<object, object?>>(convert, instance).Compile();
        });
    }

    public Action<object, object?> CreateSetter(PropertyInfo property)
    {
        var key = $"{property.DeclaringType?.FullName}.{property.Name}";
        return (Action<object, object?>)_setterCache.GetOrAdd(key, _ =>
        {
            var instance = Expression.Parameter(typeof(object), "instance");
            var value = Expression.Parameter(typeof(object), "value");
            var instanceCast = Expression.Convert(instance, property.DeclaringType!);
            var valueCast = Expression.Convert(value, property.PropertyType);
            var propertyAccess = Expression.Property(instanceCast, property);
            var assign = Expression.Assign(propertyAccess, valueCast);
            return Expression.Lambda<Action<object, object?>>(assign, instance, value).Compile();
        });
    }
}

// Usage
public class PerformanceExample
{
    public void DemonstratePerformance()
    {
        var product = new Product { Id = 1, Name = "Laptop", Price = 999.99m };
        var optimized = new OptimizedReflection();
        var property = typeof(Product).GetProperty("Price")!;

        // Get cached getter
        var getter = optimized.CreateGetter(property);
        var price = getter(product);

        // Get cached setter
        var setter = optimized.CreateSetter(property);
        setter(product, 899.99m);
    }
}
```

### Follow-up Q&A

1. **Q: When should you use compiled expressions vs direct reflection?**
   - **A**: Use compiled expressions when:
     - The same property/method is accessed frequently
     - Performance is critical
     - The code path is well-defined

2. **Q: What are the alternatives to reflection for dynamic behavior?**
   - **A**: Consider:
     - Source generators
     - Dynamic programming
     - Interface-based design
     - Code generation

## Additional Resources

- [Microsoft Docs: Reflection](https://docs.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection)
- [Microsoft Docs: Attributes](https://docs.microsoft.com/en-us/dotnet/standard/attributes/)
- [Microsoft Docs: Expression Trees](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/) 