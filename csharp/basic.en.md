# Basic C# Concepts

This guide covers fundamental C# programming concepts that every developer should know.

## Table of Contents
1. [Variables and Data Types](#1-variables-and-data-types)
2. [Control Structures](#2-control-structures)
3. [Methods and Parameters](#3-methods-and-parameters)
4. [Exception Handling](#4-exception-handling)
5. [Collections and LINQ](#5-collections-and-linq)

## 1. Variables and Data Types

### Question
What are the basic data types in C# and how do you use them?

### Suggested Answer
C# provides several built-in data types:

```csharp
// Value Types
int number = 42;                    // 32-bit integer
long bigNumber = 42L;              // 64-bit integer
float pi = 3.14f;                  // 32-bit floating point
double precisePi = 3.14159;        // 64-bit floating point
decimal money = 42.99m;            // 128-bit decimal
bool isTrue = true;                // Boolean
char letter = 'A';                 // 16-bit Unicode character

// Reference Types
string text = "Hello";             // String
object obj = new object();         // Base class for all types
dynamic dynamicVar = "Hello";      // Dynamic type

// Arrays
int[] numbers = new int[5];        // Array of integers
string[] names = new[] { "John", "Jane" }; // Array initialization

// Type Conversion
int intValue = 42;
double doubleValue = intValue;     // Implicit conversion
int backToInt = (int)doubleValue;  // Explicit conversion
```

### Follow-up Q&A

1. **Q: What's the difference between value types and reference types?**
   - **A**: Value types store their data directly in memory, while reference types store a reference to the data. Value types include primitive types and structs, while reference types include classes, interfaces, and delegates.

2. **Q: When should you use var?**
   - **A**: Use var when:
     - The type is obvious from the right side of the assignment
     - Working with anonymous types
     - The exact type name is long or complex
     - The type might change in the future

## 2. Control Structures

### Question
How do you use control structures in C#?

### Suggested Answer
C# provides several control structures for program flow:

```csharp
// If-else statements
if (condition)
{
    // Code
}
else if (anotherCondition)
{
    // Code
}
else
{
    // Code
}

// Switch statement
switch (value)
{
    case 1:
        // Code
        break;
    case 2:
        // Code
        break;
    default:
        // Code
        break;
}

// Loops
for (int i = 0; i < 10; i++)
{
    // Code
}

foreach (var item in collection)
{
    // Code
}

while (condition)
{
    // Code
}

do
{
    // Code
} while (condition);
```

### Follow-up Q&A

1. **Q: When should you use a for loop vs foreach?**
   - **A**: Use for when:
     - You need the index
     - You need to modify the collection
     - You need to iterate in reverse
     Use foreach when:
     - You just need to read elements
     - The collection implements IEnumerable
     - You want cleaner, more readable code

2. **Q: What's the difference between break and continue?**
   - **A**: break exits the loop completely, while continue skips the current iteration and continues with the next one.

## 3. Methods and Parameters

### Question
How do you define and use methods in C#?

### Suggested Answer
Methods in C# can be defined with various parameter types and return values:

```csharp
// Basic method
public void DoSomething()
{
    // Code
}

// Method with parameters
public int Add(int a, int b)
{
    return a + b;
}

// Optional parameters
public void Greet(string name = "World")
{
    Console.WriteLine($"Hello, {name}!");
}

// Params parameter
public int Sum(params int[] numbers)
{
    return numbers.Sum();
}

// Expression-bodied method
public int Multiply(int a, int b) => a * b;

// Local function
public void OuterMethod()
{
    int LocalFunction(int x) => x * 2;
    var result = LocalFunction(5);
}
```

### Follow-up Q&A

1. **Q: What's the difference between ref and out parameters?**
   - **A**: ref parameters must be initialized before passing to the method, while out parameters must be assigned within the method. Both allow the method to modify the original variable.

2. **Q: When should you use expression-bodied members?**
   - **A**: Use expression-bodied members when:
     - The method body is a single expression
     - You want more concise code
     - The logic is simple and clear

## 4. Exception Handling

### Question
How do you handle exceptions in C#?

### Suggested Answer
Exception handling in C# uses try-catch blocks:

```csharp
try
{
    // Code that might throw an exception
    int result = Divide(10, 0);
}
catch (DivideByZeroException ex)
{
    // Handle specific exception
    Console.WriteLine("Cannot divide by zero");
}
catch (Exception ex)
{
    // Handle any other exception
    Console.WriteLine($"An error occurred: {ex.Message}");
}
finally
{
    // Code that always executes
    Console.WriteLine("Cleanup code");
}

// Custom exception
public class CustomException : Exception
{
    public CustomException(string message) : base(message)
    {
    }
}

// Throwing exceptions
public int Divide(int a, int b)
{
    if (b == 0)
        throw new DivideByZeroException("Cannot divide by zero");
    return a / b;
}
```

### Follow-up Q&A

1. **Q: When should you catch exceptions?**
   - **A**: Catch exceptions when:
     - You can handle the error
     - You need to log the error
     - You need to clean up resources
     - You want to provide a user-friendly message

2. **Q: What's the difference between throw and throw ex?**
   - **A**: throw preserves the original stack trace, while throw ex resets the stack trace to the current location.

## 5. Collections and LINQ

### Question
How do you work with collections and LINQ in C#?

### Suggested Answer
C# provides various collection types and LINQ for data manipulation:

```csharp
// Lists
List<string> names = new List<string>();
names.Add("John");
names.Add("Jane");

// Dictionaries
Dictionary<string, int> ages = new Dictionary<string, int>
{
    ["John"] = 30,
    ["Jane"] = 25
};

// LINQ
var numbers = new List<int> { 1, 2, 3, 4, 5 };

// Filtering
var evenNumbers = numbers.Where(n => n % 2 == 0);

// Projection
var doubled = numbers.Select(n => n * 2);

// Aggregation
var sum = numbers.Sum();
var average = numbers.Average();

// Sorting
var sorted = numbers.OrderBy(n => n);

// Grouping
var grouped = numbers.GroupBy(n => n % 2 == 0 ? "Even" : "Odd");
```

### Follow-up Q&A

1. **Q: When should you use List vs Array?**
   - **A**: Use List when:
     - You need dynamic sizing
     - You need to add/remove items
     - You need LINQ methods
     Use Array when:
     - You know the size in advance
     - You need better performance
     - You need multi-dimensional data

2. **Q: What's the difference between IEnumerable and IList?**
   - **A**: IEnumerable is read-only and forward-only, while IList allows random access and modification of the collection.

## Additional Resources

- [Microsoft Docs: C# Programming Guide](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/)
- [Microsoft Docs: C# Language Reference](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/)
- [Microsoft Docs: Collections](https://docs.microsoft.com/en-us/dotnet/standard/collections/)
- [Microsoft Docs: LINQ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)