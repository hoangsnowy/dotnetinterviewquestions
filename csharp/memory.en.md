# Memory Management in C# - Heap vs Stack

This guide covers advanced memory management concepts in C#, focusing on the differences between Heap and Stack memory.

## Table of Contents
1. [Heap vs Stack Overview](#1-heap-vs-stack-overview)
2. [Value Types vs Reference Types](#2-value-types-vs-reference-types)
3. [Memory Allocation](#3-memory-allocation)
4. [Garbage Collection](#4-garbage-collection)
5. [Performance Considerations](#5-performance-considerations)

## 1. Heap vs Stack Overview

### Core Concept
In C#, memory is managed in two main areas:
- **Stack**: A LIFO (Last In, First Out) data structure that stores value types and reference pointers
- **Heap**: A dynamic memory area that stores reference types and their data

### Key Differences
1. **Memory Management**
   - Stack: Automatically managed, fixed size
   - Heap: Dynamically allocated, garbage collected

2. **Access Speed**
   - Stack: Faster access (direct memory access)
   - Heap: Slower access (indirect memory access)

3. **Memory Size**
   - Stack: Limited size (typically 1MB per thread)
   - Heap: Larger size (limited by available system memory)

4. **Lifetime**
   - Stack: Variables exist only within their scope
   - Heap: Objects exist until garbage collected

### Code Example
```csharp
public class MemoryExample
{
    // Stored on the stack
    private int _stackValue = 42;
    private double _stackDouble = 3.14;

    // Reference stored on stack, object on heap
    private string _heapString = "Hello";
    private List<int> _heapList = new List<int>();

    public void DemonstrateMemory()
    {
        // Stack allocation
        int localValue = 10;
        double localDouble = 2.5;

        // Heap allocation
        var list = new List<int>();
        var dictionary = new Dictionary<string, int>();
    }
}
```

## 2. Value Types vs Reference Types

### Value Types
- Stored on the stack
- Direct memory access
- Include: int, double, bool, struct, enum
- Passed by value

### Reference Types
- Stored on the heap
- Accessed via reference
- Include: class, interface, delegate, string
- Passed by reference

### Code Example
```csharp
public struct Point // Value type
{
    public int X { get; set; }
    public int Y { get; set; }
}

public class Rectangle // Reference type
{
    public Point TopLeft { get; set; }
    public Point BottomRight { get; set; }
}

public void DemonstrateTypes()
{
    // Value type on stack
    Point p1 = new Point { X = 10, Y = 20 };
    Point p2 = p1; // Creates a copy

    // Reference type on heap
    Rectangle r1 = new Rectangle { TopLeft = p1, BottomRight = p2 };
    Rectangle r2 = r1; // Creates a reference
}
```

## 3. Memory Allocation

### Stack Allocation
- Automatic allocation
- Fixed size
- Fast allocation/deallocation
- Scope-based lifetime

### Heap Allocation
- Dynamic allocation
- Variable size
- Slower allocation/deallocation
- Garbage collected

### Code Example
```csharp
public class MemoryAllocation
{
    public void DemonstrateAllocation()
    {
        // Stack allocation
        int[] stackArray = new int[1000]; // Small array on stack
        Span<int> stackSpan = stackalloc int[1000]; // Stack-only allocation

        // Heap allocation
        int[] heapArray = new int[1000000]; // Large array on heap
        List<int> heapList = new List<int>(); // Always on heap
    }
}
```

## 4. Garbage Collection

### Core Concept
The .NET Garbage Collector automatically manages heap memory by:
1. Identifying unused objects
2. Reclaiming their memory
3. Compacting the heap

### Generations
- **Generation 0**: New objects
- **Generation 1**: Survived first collection
- **Generation 2**: Long-lived objects

### Code Example
```csharp
public class GarbageCollectionExample
{
    private List<byte[]> _largeObjects = new List<byte[]>();

    public void DemonstrateGC()
    {
        // Create objects that will be collected
        for (int i = 0; i < 1000; i++)
        {
            var temp = new byte[1000];
            // temp will be collected after this scope
        }

        // Create objects that will survive
        for (int i = 0; i < 10; i++)
        {
            _largeObjects.Add(new byte[1000000]);
        }

        // Force garbage collection (not recommended in production)
        GC.Collect();
        GC.WaitForPendingFinalizers();
    }
}
```

## 5. Performance Considerations

### Best Practices
1. **Value Types**
   - Use for small, simple data structures
   - Avoid large value types
   - Consider struct vs class
   - Use stackalloc for temporary arrays

2. **Reference Types**
   - Use for complex objects
   - Implement IDisposable
   - Use object pooling
   - Minimize allocations

3. **Memory Management**
   - Use using statements
   - Implement proper disposal
   - Monitor memory usage
   - Use memory profiling tools

4. **Garbage Collection**
   - Avoid forcing GC
   - Use weak references
   - Implement finalizers carefully
   - Monitor GC performance

### Anti-patterns
1. **Value Types**
   - Large structs
   - Mutable structs
   - Boxing/unboxing
   - Excessive copying

2. **Reference Types**
   - Memory leaks
   - Not disposing
   - Large object graphs
   - Circular references

3. **Memory Management**
   - Not using using
   - Not disposing
   - Memory leaks
   - Resource leaks

4. **Garbage Collection**
   - Forcing GC
   - Poor finalizers
   - Memory pressure
   - Large object allocations

### Follow-up Questions

1. **Q: When should you use struct vs class?**
   - **Answer**: Use structs for small, immutable value types, and classes for larger, mutable reference types.
   - **Example**:
     ```csharp
     // Good use of struct
     public struct Point
     {
         public int X { get; }
         public int Y { get; }
     }

     // Good use of class
     public class Rectangle
     {
         public Point TopLeft { get; set; }
         public Point BottomRight { get; set; }
     }
     ```
   - **Best Practice**: Use structs for small, immutable value types.

2. **Q: How do you handle memory leaks?**
   - **Answer**: Implement IDisposable, use using statements, and properly clean up resources.
   - **Example**:
     ```csharp
     public class ResourceManager : IDisposable
     {
         private bool _disposed;
         private readonly Stream _stream;

         public ResourceManager()
         {
             _stream = new FileStream("data.txt", FileMode.Open);
         }

         public void Dispose()
         {
             Dispose(true);
             GC.SuppressFinalize(this);
         }

         protected virtual void Dispose(bool disposing)
         {
             if (!_disposed)
             {
                 if (disposing)
                 {
                     _stream?.Dispose();
                 }
                 _disposed = true;
             }
         }
     }
     ```
   - **Best Practice**: Implement proper resource cleanup.

### Key Takeaways
1. Understand heap vs stack differences
2. Choose appropriate types
3. Manage memory properly
4. Handle garbage collection
5. Follow best practices

### Additional Resources
- [C# Memory Management](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/)
- [C# Value Types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)
- [C# Reference Types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types)
- [C# Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/) 