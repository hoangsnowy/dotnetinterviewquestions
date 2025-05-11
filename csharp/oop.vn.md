# Lập trình Hướng đối tượng trong C# cho Thương mại điện tử

## Giới thiệu

Lập trình Hướng đối tượng (OOP) là một phương pháp lập trình sử dụng các đối tượng để tổ chức code. Trong C#, OOP được sử dụng rộng rãi để xây dựng các ứng dụng thương mại điện tử. Tài liệu này sẽ giải thích các khái niệm OOP cốt lõi trong C# với ví dụ về thương mại điện tử.

## 1. Trừu tượng hóa (Abstraction)

### Khái niệm cốt lõi
Trừu tượng hóa là quá trình ẩn các chi tiết phức tạp và chỉ hiển thị các tính năng cần thiết. Trong C#, trừu tượng hóa được thực hiện thông qua abstract classes và interfaces.

### Ví dụ thực tế
Trong hệ thống thương mại điện tử:
- **Abstract Class**: Lớp Product cơ sở
- **Interface**: Giao diện xử lý thanh toán

### Ví dụ code
```csharp
// Abstract class
public abstract class Product
{
    public string Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    
    public virtual decimal CalculateDiscount()
    {
        return Price * 0.1m; // Giảm giá mặc định 10%
    }
}

// Interface
public interface IPaymentProcessor
{
    Task<PaymentResult> ProcessPayment(PaymentRequest request);
}

// Implementation
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
}
```

### Best Practices
1. **Thiết kế Interface**
   - Định nghĩa các hợp đồng rõ ràng
   - Giữ interface tập trung
   - Sử dụng interface segregation
   - Ghi chú hành vi

2. **Triển khai**
   - Tuân thủ nguyên lý Liskov Substitution
   - Xử lý lỗi đúng cách
   - Sử dụng dependency injection
   - Triển khai logging

3. **An toàn kiểu**
   - Sử dụng kiểm tra kiểu phù hợp
   - Tránh ép kiểu xuống
   - Sử dụng pattern matching
   - Xử lý trường hợp null

### Anti-patterns
1. **Thiết kế Interface**
   - Interface phình to
   - Hợp đồng không rõ ràng
   - Ghi chú kém
   - Liên kết chặt chẽ

2. **Triển khai**
   - Vi phạm LSP
   - Xử lý lỗi kém
   - Dependency cứng
   - Thiếu logging

3. **An toàn kiểu**
   - Kiểm tra kiểu quá mức
   - Ép kiểu xuống không an toàn
   - Bỏ qua trường hợp null
   - Pattern matching kém

### Câu hỏi mở rộng

1. **Hỏi: Làm thế nào để xử lý các phương thức thanh toán khác nhau một cách trừu tượng?**
   - **Trả lời**: Sử dụng interface và dependency injection để xử lý các phương thức thanh toán khác nhau.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng interface và dependency injection cho các phương thức thanh toán khác nhau.

2. **Hỏi: Khi nào nên sử dụng interface và khi nào nên sử dụng abstract class?**
   - **Trả lời**: Sử dụng interface để định nghĩa các hợp đồng có thể được triển khai bởi các lớp không liên quan, và sử dụng abstract class để chia sẻ triển khai giữa các lớp liên quan.
   - **Ví dụ**:
     ```csharp
     // Interface cho các triển khai không liên quan
     public interface IPaymentGateway
     {
         Task<PaymentResult> ProcessPayment(PaymentRequest request);
     }

     // Abstract class cho các triển khai liên quan
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
   - **Best Practice**: Sử dụng interface cho các hợp đồng và abstract class cho chia sẻ triển khai.

### Ý chính cần nhớ
1. Trừu tượng hóa giúp quản lý độ phức tạp trong hệ thống lớn
2. Sử dụng interface cho hành vi đa hình
3. Abstract class cho chia sẻ triển khai
4. Đóng gói phù hợp
5. Tuân thủ nguyên lý interface segregation

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [Design Patterns trong C#](https://refactoring.guru/design-patterns/csharp)

## 2. Đóng gói (Encapsulation)

### Khái niệm cốt lõi
Đóng gói là quá trình ẩn thông tin và cung cấp quyền truy cập có kiểm soát vào dữ liệu. Trong C#, đóng gói được thực hiện thông qua access modifiers và properties.

### Ví dụ thực tế
Trong hệ thống thương mại điện tử:
- **Private Fields**: Dữ liệu đơn hàng
- **Public Properties**: Truy cập có kiểm soát
- **Private Methods**: Xử lý nội bộ

### Ví dụ code
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
1. **Ẩn dữ liệu**
   - Sử dụng private fields
   - Tiết lộ dữ liệu thông qua properties
   - Triển khai validation phù hợp
   - Sử dụng readonly collections

2. **Đóng gói phương thức**
   - Giữ các phương thức tập trung
   - Sử dụng access modifiers phù hợp
   - Triển khai xử lý lỗi
   - Log các hoạt động quan trọng

3. **Quản lý trạng thái**
   - Duy trì các bất biến đối tượng
   - Sử dụng đối tượng bất biến khi có thể
   - Triển khai validation phù hợp
   - Xử lý truy cập đồng thời

4. **Bảo mật**
   - Bảo vệ dữ liệu nhạy cảm
   - Triển khai validation phù hợp
   - Sử dụng giá trị mặc định an toàn
   - Tuân thủ best practices bảo mật

### Anti-patterns
1. **Tiết lộ dữ liệu**
   - Public fields
   - Collections có thể thay đổi
   - Trạng thái toàn cục
   - Tiết lộ nội bộ

2. **Vấn đề phương thức**
   - Quá nhiều trách nhiệm
   - Xử lý lỗi kém
   - Thiếu validation
   - Trạng thái không nhất quán

3. **Vấn đề trạng thái**
   - Trạng thái có thể thay đổi
   - Điều kiện đua
   - Trạng thái không hợp lệ
   - Validation kém

4. **Vấn đề bảo mật**
   - Tiết lộ dữ liệu nhạy cảm
   - Thiếu validation
   - Giá trị mặc định không an toàn
   - Xử lý lỗi kém

### Câu hỏi mở rộng

1. **Hỏi: Làm thế nào để xử lý truy cập đồng thời đến dữ liệu được đóng gói?**
   - **Trả lời**: Sử dụng collections thread-safe, triển khai cơ chế khóa phù hợp, và xem xét sử dụng đối tượng bất biến.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng cơ chế đồng bộ hóa phù hợp và collections thread-safe.

2. **Hỏi: Làm thế nào để đảm bảo tính toàn vẹn dữ liệu trong các lớp được đóng gói?**
   - **Trả lời**: Triển khai validation phù hợp, sử dụng đối tượng bất biến khi có thể, và duy trì các bất biến đối tượng.
   - **Ví dụ**:
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
   - **Best Practice**: Validate dữ liệu đầu vào và duy trì các bất biến đối tượng.

### Ý chính cần nhớ
1. Đóng gói giúp bảo vệ dữ liệu và duy trì tính toàn vẹn
2. Kiểm soát truy cập phù hợp rất quan trọng cho bảo mật
3. Thread safety quan trọng cho truy cập đồng thời
4. Validation giúp duy trì tính toàn vẹn dữ liệu
5. Đối tượng bất biến có thể đơn giản hóa quản lý trạng thái

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [Thread Safety trong C#](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices)
- [Immutable Types trong C#](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/immutable-types)

## 3. Kế thừa (Inheritance)

### Khái niệm cốt lõi
Kế thừa là khả năng của một lớp để kế thừa các thuộc tính và phương thức từ một lớp khác. Trong C#, kế thừa được thực hiện thông qua từ khóa `:`.

### Ví dụ thực tế
Trong hệ thống thương mại điện tử:
- **Base Class**: Lớp Product
- **Derived Classes**: PhysicalProduct, DigitalProduct

### Ví dụ code
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
        return Price * 0.1m; // Giảm giá mặc định 10%
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
        return Price * 0.2m; // Giảm giá 20% cho sản phẩm số
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
1. **Thiết kế lớp**
   - Giữ phân cấp kế thừa nông
   - Sử dụng abstract classes cho triển khai chung
   - Triển khai các constructor phù hợp
   - Sử dụng virtual methods một cách thận trọng

2. **Ghi đè phương thức**
   - Sử dụng từ khóa override
   - Gọi các phương thức cơ sở khi cần thiết
   - Tuân thủ nguyên lý Liskov Substitution
   - Ghi chú các thay đổi hành vi

3. **Kiểm soát truy cập**
   - Sử dụng protected cho các thành viên kế thừa
   - Giữ các thành viên private
   - Sử dụng sealed để ngăn kế thừa thêm
   - Triển khai đóng gói phù hợp

4. **Tổ chức code**
   - Nhóm các lớp liên quan
   - Sử dụng namespace hiệu quả
   - Tuân thủ quy ước đặt tên
   - Ghi chú các mối quan hệ kế thừa

### Anti-patterns
1. **Thiết kế lớp**
   - Phân cấp kế thừa sâu
   - Kế thừa chỉ để tái sử dụng code
   - Kế thừa nhiều lớp
   - Kế thừa cho triển khai

2. **Ghi đè phương thức**
   - Phá vỡ hợp đồng lớp cơ sở
   - Không gọi phương thức cơ sở
   - Ghi đè không có virtual
   - Ghi chú kém

3. **Kiểm soát truy cập**
   - Tiết lộ thành viên protected
   - Lạm dụng protected
   - Không sử dụng sealed
   - Đóng gói kém

4. **Tổ chức code**
   - Các lớp không liên quan trong phân cấp
   - Tổ chức namespace kém
   - Đặt tên không nhất quán
   - Thiếu ghi chú

### Câu hỏi mở rộng

1. **Hỏi: Làm thế nào để xử lý vấn đề "diamond problem" trong C#?**
   - **Trả lời**: C# không hỗ trợ kế thừa nhiều lớp, nhưng hỗ trợ triển khai nhiều interface. Sử dụng interface và composition để tránh vấn đề diamond.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng interface cho kế thừa nhiều và composition cho tái sử dụng code.

2. **Hỏi: Khi nào nên sử dụng interface và khi nào nên sử dụng abstract class?**
   - **Trả lời**: Sử dụng interface để định nghĩa các hợp đồng có thể được triển khai bởi các lớp không liên quan, và sử dụng abstract class để chia sẻ triển khai giữa các lớp liên quan.
   - **Ví dụ**:
     ```csharp
     // Interface cho các triển khai không liên quan
     public interface IPaymentGateway
     {
         Task<PaymentResult> ProcessPayment(PaymentRequest request);
     }

     // Abstract class cho các triển khai liên quan
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
   - **Best Practice**: Sử dụng interface cho các hợp đồng và abstract class cho chia sẻ triển khai.

### Ý chính cần nhớ
1. Kế thừa cho phép tái sử dụng code và đa hình
2. Giữ phân cấp kế thừa nông
3. Sử dụng interface cho kế thừa nhiều
4. Tuân thủ nguyên lý Liskov Substitution
5. Ghi chú các mối quan hệ kế thừa

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [Abstract Classes trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members)
- [Interfaces trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/)

## 4. Đa hình (Polymorphism)

### Khái niệm cốt lõi
Đa hình là khả năng của các đối tượng để có nhiều hình thức và hành vi khác nhau dựa trên kiểu của chúng. Trong C#, đa hình được thực hiện thông qua kế thừa và interface.

### Ví dụ thực tế
Trong hệ thống thương mại điện tử:
- **Payment Processing**: Các phương thức thanh toán khác nhau
- **Shipping Calculations**: Các phương thức vận chuyển khác nhau
- **Product Types**: Các hành vi sản phẩm khác nhau
- **Order Processing**: Các loại đơn hàng khác nhau

### Ví dụ code
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
1. **Thiết kế Interface**
   - Định nghĩa các hợp đồng rõ ràng
   - Giữ interface tập trung
   - Sử dụng interface segregation
   - Ghi chú hành vi

2. **Triển khai**
   - Tuân thủ nguyên lý Liskov Substitution
   - Xử lý lỗi đúng cách
   - Sử dụng dependency injection
   - Triển khai logging

3. **An toàn kiểu**
   - Sử dụng kiểm tra kiểu phù hợp
   - Tránh ép kiểu xuống
   - Sử dụng pattern matching
   - Xử lý trường hợp null

4. **Tổ chức code**
   - Nhóm các triển khai liên quan
   - Sử dụng namespace phù hợp
   - Tuân thủ quy ước đặt tên
   - Ghi chú các mối quan hệ

### Anti-patterns
1. **Thiết kế Interface**
   - Interface phình to
   - Hợp đồng không rõ ràng
   - Ghi chú kém
   - Liên kết chặt chẽ

2. **Triển khai**
   - Vi phạm LSP
   - Xử lý lỗi kém
   - Dependency cứng
   - Thiếu logging

3. **An toàn kiểu**
   - Kiểm tra kiểu quá mức
   - Ép kiểu xuống không an toàn
   - Bỏ qua trường hợp null
   - Pattern matching kém

4. **Tổ chức code**
   - Triển khai phân tán
   - Tổ chức namespace kém
   - Đặt tên không nhất quán
   - Thiếu ghi chú

### Câu hỏi mở rộng

1. **Hỏi: Làm thế nào để xử lý các phương thức thanh toán khác nhau một cách đa hình?**
   - **Trả lời**: Sử dụng interface và dependency injection để xử lý các phương thức thanh toán khác nhau.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng interface và dependency injection cho các phương thức thanh toán khác nhau.

2. **Hỏi: Làm thế nào để đảm bảo an toàn kiểu khi làm việc với các đối tượng đa hình?**
   - **Trả lời**: Sử dụng pattern matching, kiểm tra kiểu phù hợp, và xử lý trường hợp null.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng pattern matching và kiểm tra kiểu phù hợp cho hành vi đa hình an toàn.

### Ý chính cần nhớ
1. Đa hình cho phép code linh hoạt và mở rộng
2. Sử dụng interface cho hành vi đa hình
3. Tuân thủ nguyên lý Liskov Substitution
4. Xử lý an toàn kiểu phù hợp
5. Sử dụng dependency injection

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [Pattern Matching trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/pattern-matching)
- [Interfaces trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/)

## 5. Abstract Class vs Interface

### Khái niệm cốt lõi
Abstract class là một lớp không thể khởi tạo và được sử dụng làm lớp cơ sở cho các lớp khác. Interface là một tập hợp các phương thức liên quan định nghĩa một hợp đồng.

### Ví dụ thực tế
Trong hệ thống thương mại điện tử:
- **Abstract Class**: Lớp Product cơ sở
- **Interface**: Giao diện xử lý thanh toán

### Ví dụ code
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
1. **Sử dụng Abstract Class cho Triển khai Chung**
   - Sử dụng abstract class cho triển khai chung
   - Giữ phân cấp kế thừa nông
   - Sử dụng interface cho kế thừa nhiều

2. **Sử dụng Interface cho Định nghĩa Hợp đồng**
   - Sử dụng interface cho định nghĩa hợp đồng có thể được triển khai bởi các lớp không liên quan
   - Tuân thủ nguyên lý interface segregation

3. **Tránh Liên kết Chặt chẽ**
   - Sử dụng interface cho hành vi đa hình
   - Tránh liên kết chặt chẽ
   - Triển khai strategy pattern

4. **Tránh Interface Phình to**
   - Sử dụng interface cho định nghĩa hợp đồng
   - Tránh quá nhiều interface
   - Tuân thủ nguyên lý interface segregation

### Anti-patterns
1. **Liên kết Chặt chẽ**
   - Interface phình to
   - Trừu tượng hóa quá mức
   - Trừu tượng hóa rò rỉ
   - Liên kết chặt chẽ

2. **Interface Phình to**
   - Quá nhiều interface
   - Trừu tượng hóa quá mức
   - Trừu tượng hóa rò rỉ
   - Liên kết chặt chẽ

3. **Trừu tượng hóa Quá mức**
   - Trừu tượng hóa rò rỉ
   - Interface phình to
   - Liên kết chặt chẽ

4. **Vi phạm Nguyên lý Interface Segregation**
   - Quá nhiều phương thức trong interface
   - Interface phình to
   - Trừu tượng hóa quá mức
   - Liên kết chặt chẽ

### Câu hỏi mở rộng

1. **Hỏi: Làm thế nào để xử lý vấn đề "diamond problem" trong C#?**
   - **Trả lời**: C# không hỗ trợ kế thừa nhiều lớp, nhưng hỗ trợ triển khai nhiều interface. Sử dụng interface và composition để tránh vấn đề diamond.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng interface cho kế thừa nhiều và composition cho tái sử dụng code.

2. **Hỏi: Khi nào nên sử dụng interface và khi nào nên sử dụng abstract class?**
   - **Trả lời**: Sử dụng interface để định nghĩa các hợp đồng có thể được triển khai bởi các lớp không liên quan, và sử dụng abstract class để chia sẻ triển khai giữa các lớp liên quan.
   - **Ví dụ**:
     ```csharp
     // Interface cho các triển khai không liên quan
     public interface IPaymentGateway
     {
         Task<PaymentResult> ProcessPayment(PaymentRequest request);
     }

     // Abstract class cho các triển khai liên quan
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
   - **Best Practice**: Sử dụng interface cho các hợp đồng và abstract class cho chia sẻ triển khai.

### Ý chính cần nhớ
1. Sử dụng interface cho định nghĩa hợp đồng
2. Sử dụng abstract class cho chia sẻ triển khai
3. Đóng gói phù hợp
4. Tránh liên kết chặt chẽ
5. Tuân thủ nguyên lý interface segregation

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [Abstract Classes trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members)
- [Interfaces trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/)

## 6. Overloading vs Overriding

### Khái niệm cốt lõi
Overloading là khả năng định nghĩa nhiều phương thức với cùng tên nhưng khác tham số. Overriding là khả năng cung cấp một triển khai cụ thể của một phương thức đã được định nghĩa trong lớp cơ sở.

### Ví dụ thực tế
Trong hệ thống thương mại điện tử:
- **Overloading**: Các constructor khác nhau cho cùng một lớp
- **Overriding**: Các triển khai khác nhau cho cùng một phương thức trong các lớp khác nhau

### Ví dụ code
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
   - Sử dụng overloading cho các constructor khác nhau
   - Giữ các phương thức tập trung
   - Sử dụng access modifiers phù hợp
   - Triển khai xử lý lỗi
   - Log các hoạt động quan trọng

2. **Overriding**
   - Sử dụng từ khóa override
   - Gọi các phương thức cơ sở khi cần thiết
   - Tuân thủ nguyên lý Liskov Substitution
   - Ghi chú các thay đổi hành vi

3. **Kiểm soát Truy cập**
   - Sử dụng protected cho các thành viên kế thừa
   - Giữ các thành viên private
   - Sử dụng sealed để ngăn kế thừa thêm
   - Triển khai đóng gói phù hợp

4. **Tổ chức Code**
   - Nhóm các lớp liên quan
   - Sử dụng namespace hiệu quả
   - Tuân thủ quy ước đặt tên
   - Ghi chú các mối quan hệ kế thừa

### Anti-patterns
1. **Overloading**
   - Quá nhiều phương thức overloaded
   - Xử lý lỗi kém
   - Thiếu validation
   - Trạng thái không nhất quán

2. **Overriding**
   - Phá vỡ hợp đồng lớp cơ sở
   - Không gọi phương thức cơ sở
   - Ghi đè không có virtual
   - Ghi chú kém

3. **Kiểm soát Truy cập**
   - Tiết lộ thành viên protected
   - Lạm dụng protected
   - Không sử dụng sealed
   - Đóng gói kém

4. **Tổ chức Code**
   - Các lớp không liên quan trong phân cấp
   - Tổ chức namespace kém
   - Đặt tên không nhất quán
   - Thiếu ghi chú

### Câu hỏi mở rộng

1. **Hỏi: Làm thế nào để xử lý overloading trong hệ thống thương mại điện tử?**
   - **Trả lời**: Sử dụng overloading cho các constructor và phương thức khác nhau.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng overloading cho các constructor và phương thức khác nhau.

2. **Hỏi: Khi nào nên sử dụng overloading và khi nào nên sử dụng overriding?**
   - **Trả lời**: Sử dụng overloading cho các constructor và phương thức khác nhau, và overriding cho các triển khai khác nhau của cùng một phương thức.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng overloading cho các constructor và phương thức khác nhau, và overriding cho các triển khai khác nhau của cùng một phương thức.

### Ý chính cần nhớ
1. Overloading cho phép các constructor và phương thức khác nhau
2. Overriding cho phép các triển khai khác nhau của cùng một phương thức
3. Đóng gói phù hợp
4. Tuân thủ nguyên lý Liskov Substitution
5. Ghi chú các mối quan hệ kế thừa

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [Overloading trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/overloading)
- [Overriding trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/inheritance)

## 7. Sealed Keyword

### Khái niệm cốt lõi
Từ khóa sealed được sử dụng để ngăn chặn kế thừa thêm của một lớp hoặc một phương thức.

### Ví dụ thực tế
Trong hệ thống thương mại điện tử, từ khóa sealed được sử dụng cho:
- Ngăn chặn kế thừa thêm của một lớp
- Ngăn chặn ghi đè thêm của một phương thức

### Ví dụ code
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
1. **Sử dụng Sealed Classes để Ngăn Kế thừa**
   - Sử dụng sealed classes để ngăn kế thừa thêm
   - Giữ phân cấp kế thừa nông
   - Sử dụng interface cho kế thừa nhiều

2. **Sử dụng Sealed Methods để Ngăn Ghi đè**
   - Sử dụng sealed methods để ngăn ghi đè thêm
   - Giữ các phương thức tập trung
   - Sử dụng access modifiers phù hợp
   - Triển khai xử lý lỗi

3. **Kiểm soát Truy cập**
   - Sử dụng protected cho các thành viên kế thừa
   - Giữ các thành viên private
   - Sử dụng sealed để ngăn kế thừa thêm
   - Triển khai đóng gói phù hợp

4. **Tổ chức Code**
   - Nhóm các lớp liên quan
   - Sử dụng namespace hiệu quả
   - Tuân thủ quy ước đặt tên
   - Ghi chú các mối quan hệ kế thừa

### Anti-patterns
1. **Sealed Classes**
   - Quá nhiều sealed classes
   - Trừu tượng hóa quá mức
   - Trừu tượng hóa rò rỉ
   - Liên kết chặt chẽ

2. **Sealed Methods**
   - Quá nhiều sealed methods
   - Trừu tượng hóa quá mức
   - Trừu tượng hóa rò rỉ
   - Liên kết chặt chẽ

3. **Kiểm soát Truy cập**
   - Tiết lộ thành viên protected
   - Lạm dụng protected
   - Không sử dụng sealed
   - Đóng gói kém

4. **Tổ chức Code**
   - Các lớp không liên quan trong phân cấp
   - Tổ chức namespace kém
   - Đặt tên không nhất quán
   - Thiếu ghi chú

### Câu hỏi mở rộng

1. **Hỏi: Làm thế nào để xử lý sealed classes trong hệ thống thương mại điện tử?**
   - **Trả lời**: Sử dụng sealed classes để ngăn kế thừa thêm.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng sealed classes để ngăn kế thừa thêm.

2. **Hỏi: Khi nào nên sử dụng sealed classes và khi nào nên sử dụng sealed methods?**
   - **Trả lời**: Sử dụng sealed classes để ngăn kế thừa thêm và sealed methods để ngăn ghi đè thêm.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng sealed classes để ngăn kế thừa thêm và sealed methods để ngăn ghi đè thêm.

### Ý chính cần nhớ
1. Sử dụng sealed classes để ngăn kế thừa thêm
2. Sử dụng sealed methods để ngăn ghi đè thêm
3. Đóng gói phù hợp
4. Tuân thủ nguyên lý Liskov Substitution
5. Ghi chú các mối quan hệ kế thừa

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [Sealed Classes trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/sealed-classes-and-class-members)
- [Sealed Methods trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/sealed-methods)

## 8. Access Modifiers

### Khái niệm cốt lõi
Access modifiers kiểm soát khả năng truy cập của các lớp, phương thức và các thành viên khác.

### Ví dụ thực tế
Trong hệ thống thương mại điện tử, access modifiers được sử dụng cho:
- Đóng gói
- Bảo mật
- Tổ chức code

### Ví dụ code
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
1. **Sử dụng Private Fields**
   - Sử dụng private fields với public properties
   - Triển khai validation phù hợp trong setters
   - Sử dụng readonly collections cho collections
   - Triển khai tính bất biến phù hợp khi có thể

2. **Sử dụng Public Properties**
   - Tiết lộ dữ liệu thông qua properties
   - Triển khai validation phù hợp
   - Sử dụng readonly collections

3. **Sử dụng Protected Members**
   - Sử dụng protected cho các thành viên kế thừa
   - Giữ các thành viên private
   - Sử dụng sealed để ngăn kế thừa thêm
   - Triển khai đóng gói phù hợp

4. **Sử dụng Private Constructors**
   - Sử dụng private constructors để ngăn khởi tạo
   - Triển khai đóng gói phù hợp

### Anti-patterns
1. **Tiết lộ Trạng thái Nội bộ**
   - Public fields
   - Collections có thể thay đổi
   - Trạng thái toàn cục
   - Tiết lộ nội bộ

2. **Sử dụng Public Fields**
   - Public fields
   - Collections có thể thay đổi
   - Trạng thái toàn cục
   - Tiết lộ nội bộ

3. **Collections Có thể Thay đổi**
   - Collections có thể thay đổi
   - Trạng thái toàn cục
   - Tiết lộ nội bộ
   - Tiết lộ trạng thái nội bộ

4. **Trạng thái Toàn cục**
   - Trạng thái toàn cục
   - Tiết lộ nội bộ
   - Collections có thể thay đổi
   - Tiết lộ trạng thái nội bộ

### Câu hỏi mở rộng

1. **Hỏi: Làm thế nào để xử lý access modifiers trong hệ thống thương mại điện tử?**
   - **Trả lời**: Sử dụng private fields với public properties, triển khai validation phù hợp trong setters, sử dụng readonly collections cho collections, và triển khai tính bất biến phù hợp khi có thể.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng private fields với public properties, triển khai validation phù hợp trong setters, sử dụng readonly collections cho collections, và triển khai tính bất biến phù hợp khi có thể.

2. **Hỏi: Khi nào nên sử dụng public và khi nào nên sử dụng private access modifiers?**
   - **Trả lời**: Sử dụng public access modifiers cho tiết lộ dữ liệu và phương thức, và private access modifiers cho ẩn chi tiết triển khai.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng public access modifiers cho tiết lộ dữ liệu và phương thức, và private access modifiers cho ẩn chi tiết triển khai.

### Ý chính cần nhớ
1. Access modifiers giúp quản lý khả năng truy cập và bảo mật
2. Sử dụng private fields với public properties
3. Triển khai validation phù hợp trong setters
4. Sử dụng readonly collections cho collections
5. Triển khai tính bất biến phù hợp khi có thể

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [Access Modifiers trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)
- [Design Patterns trong C#](https://refactoring.guru/design-patterns/csharp)

## 9. Class vs Struct

### Khái niệm cốt lõi
Class là kiểu tham chiếu, và struct là kiểu giá trị. Classes được lưu trữ trên heap, trong khi structs được lưu trữ trên stack.

### Ví dụ thực tế
Trong hệ thống thương mại điện tử:
- **Class**: Order, Product
- **Struct**: OrderItem

### Ví dụ code
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
1. **Sử dụng Classes cho Kiểu Tham chiếu**
   - Sử dụng classes cho kiểu tham chiếu
   - Lưu trữ trên heap
   - Hỗ trợ kế thừa
   - Hỗ trợ đa hình

2. **Sử dụng Structs cho Kiểu Giá trị**
   - Sử dụng structs cho kiểu giá trị
   - Lưu trữ trên stack
   - Bất biến mặc định
   - Hỗ trợ kiểu đơn giản

3. **Tránh Structs Lớn**
   - Tránh structs lớn
   - Sử dụng classes thay thế
   - Triển khai đóng gói phù hợp

4. **Tránh Kiểu Tham chiếu**
   - Tránh kiểu tham chiếu
   - Sử dụng structs thay thế
   - Triển khai đóng gói phù hợp

### Anti-patterns
1. **Sử dụng Kiểu Tham chiếu**
   - Sử dụng kiểu tham chiếu
   - Lưu trữ trên heap
   - Hỗ trợ kế thừa
   - Hỗ trợ đa hình

2. **Sử dụng Kiểu Giá trị**
   - Sử dụng kiểu giá trị
   - Lưu trữ trên stack
   - Bất biến mặc định
   - Hỗ trợ kiểu đơn giản

3. **Structs Lớn**
   - Structs lớn
   - Lưu trữ trên stack
   - Bất biến mặc định
   - Hỗ trợ kiểu đơn giản

4. **Tránh Kiểu Tham chiếu**
   - Tránh kiểu tham chiếu
   - Sử dụng structs thay thế
   - Triển khai đóng gói phù hợp

### Câu hỏi mở rộng

1. **Hỏi: Làm thế nào để xử lý class vs struct trong hệ thống thương mại điện tử?**
   - **Trả lời**: Sử dụng classes cho kiểu tham chiếu và structs cho kiểu giá trị.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng classes cho kiểu tham chiếu và structs cho kiểu giá trị.

2. **Hỏi: Khi nào nên sử dụng class và khi nào nên sử dụng struct?**
   - **Trả lời**: Sử dụng classes cho kiểu tham chiếu và structs cho kiểu giá trị.
   - **Ví dụ**:
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
   - **Best Practice**: Sử dụng classes cho kiểu tham chiếu và structs cho kiểu giá trị.

### Ý chính cần nhớ
1. Sử dụng classes cho kiểu tham chiếu và structs cho kiểu giá trị
2. Lưu trữ trên heap cho classes và stack cho structs
3. Bất biến mặc định cho structs
4. Hỗ trợ kế thừa và đa hình cho classes
5. Triển khai đóng gói phù hợp

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)
- [Classes trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/classes)
- [Structs trong C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/structs)
- [Design Patterns trong C#](https://refactoring.guru/design-patterns/csharp)

## Kết luận

Tài liệu này bao gồm các khái niệm OOP cốt lõi trong C# với ví dụ về thương mại điện tử, giải thích chi tiết, best practices và các tình huống thực tế. Bằng cách hiểu các khái niệm này, bạn có thể tạo ra các hệ thống thương mại điện tử đơn giản và hiệu quả.

### Ý chính cần nhớ
1. Nguyên lý OOP giúp tạo ra hệ thống thương mại điện tử đơn giản
2. Trừu tượng hóa và đóng gói đúng cách rất quan trọng
3. Kế thừa và đa hình giúp code linh hoạt
4. Tuân thủ best practices cơ bản

### Tài liệu tham khảo
- [Tài liệu C# chính thức](https://docs.microsoft.com/vi-vn/dotnet/csharp/)
- [Best Practices C# cơ bản](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)

Cảm ơn bạn đã đọc tài liệu này về Lập trình Hướng đối tượng trong C# cho lĩnh vực thương mại điện tử. Tôi hy vọng bạn thấy nó hữu ích và cung cấp thông tin. Nếu bạn có bất kỳ câu hỏi nào hoặc cần làm rõ thêm, hãy thoải mái hỏi.

Chúc bạn code vui vẻ!