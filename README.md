# Mini e-Ticarət & Sifariş İdarəetmə Sistemi Tapşırığı

---

## 1. Sistem Arxitekturası və Entity-lər

Sistemdə cəmi 6 əsas Entity olacaq:

1. **User** (İstifadəçi - Id, FullName, Email, Role, CreatedAt)
2. **Product** (Məhsul - Id, Name, Price, StockQuantity, IsActive)
3. **Category** (Kateqoriya - Id, Name, Description)
4. **Order** (Sifariş - Id, UserId, OrderDate, Status, TotalAmount)
5. **OrderItem** (Sifariş Detalı - Id, OrderId, ProductId, Quantity, UnitPrice)
6. **Payment** (Ödəniş - Id, OrderId, PaymentDate, Amount, PaymentMethod, Status)

---

## 2. Enum-lar

```csharp
public enum UserRole
{
    Admin = 1,
    Customer = 2
}

public enum OrderStatus
{
    Pending = 1,
    Processing = 2,
    Shipped = 3,
    Delivered = 4,
    Cancelled = 5
}

public enum PaymentMethod
{
    CreditCard = 1,
    PayPal = 2,
    BankTransfer = 3
}

public enum PaymentStatus
{
    Pending = 1,
    Completed = 2,
    Failed = 3
}
```

---

## 3. EF Core Configurations (Fluent API)

Hər bir Entity üçün ayrı `IEntityTypeConfiguration<T>` faylı yaradılmalıdır. Metod daxilində `Configure` metodunu yazmalısınız:

* **`UserConfiguration`**
  * `Configure(EntityTypeBuilder<User> builder)`
* **`ProductConfiguration`**
  * `Configure(EntityTypeBuilder<Product> builder)`
* **`CategoryConfiguration`**
  * `Configure(EntityTypeBuilder<Category> builder)`
* **`OrderConfiguration`**
  * `Configure(EntityTypeBuilder<Order> builder)`
* **`OrderItemConfiguration`**
  * `Configure(EntityTypeBuilder<OrderItem> builder)`
* **`PaymentConfiguration`**
  * `Configure(EntityTypeBuilder<Payment> builder)`

---

## 4. AppDbContext Strukturu

`AppDbContext` sinfində `DbSet`-lər, konfiqurasiyaların qoşulması və Override ediləcək metodlar yer almalıdır:

* **Xassələr:** `Users`, `Products`, `Categories`, `Orders`, `OrderItems`, `Payments`
* **Metodlar:**
  * `OnConfiguring(DbContextOptionsBuilder optionsBuilder)`
  * `OnModelCreating(ModelBuilder modelBuilder)`
  * `SaveChangesAsync(CancellationToken cancellationToken = default)` *(Audit loglar və ya Soft Delete məntiqləri üçün)*

---

## 5. Interfeyslər və Servislər (Yalnız Metod İmzaları)

### A. Category / Product İdarəetməsi

```csharp
public interface IProductService
{
    Task<ProductDto> GetByIdAsync(int id);
    Task<List<ProductDto>> GetAllAsync();
    Task<List<ProductDto>> GetByCategoryIdAsync(int categoryId);
    Task CreateAsync(CreateProductDto dto);
    Task UpdateAsync(int id, UpdateProductDto dto);
    Task DeleteAsync(int id);
    Task UpdateStockAsync(int productId, int quantityChange);
}
```

### B. Sifariş İdarəetməsi

```csharp
public interface IOrderService
{
    Task<OrderDto> GetOrderDetailsAsync(int orderId);
    Task<List<OrderDto>> GetOrdersByUserIdAsync(int userId);
    Task<int> CreateOrderAsync(CreateOrderDto dto);
    Task UpdateOrderStatusAsync(int orderId, OrderStatus status);
    Task CancelOrderAsync(int orderId);
}
```

### C. Ödəniş və Avtomatlaşdırma

```csharp
public interface IPaymentService
{
    Task<PaymentDto> GetPaymentByOrderIdAsync(int orderId);
    Task<bool> ProcessPaymentAsync(CreatePaymentDto dto);
}
```

---

## Tapşırığın İzahı və Gedişat Ardıcıllığı

Bu tapşırığı sırasıyla icra edərək bazadan servislərə qədər tam funksional arxitektura qura bilərsiniz.

### Addım 1: Domen və Baza Modelinin Qurulması
1. **Entity-ləri təyin edin:** Verilən 6 entity-ni və Enum-ları yaradın. Entity-lər arasında düzgün Naviqasiya xassələrini (Navigation Properties) verin:
   * `Category` $\rightarrow$ `List<Product>` (1-ə Çox)
   * `User` $\rightarrow$ `List<Order>` (1-ə Çox)
   * `Order` $\rightarrow$ `List<OrderItem>` (1-ə Çox)
   * `Order` $\rightarrow$ `Payment` (1-ə 1)

### Addım 2: Fluent API Konfiqurasiyası
1. `IEntityTypeConfiguration<T>` interfeysini tətbiq edin:
   * **Məhdudiyyətlər (Constraints):** `HasMaxLength`, `IsRequired`, `HasPrecision` (qiymətlər üçün `decimal(18,2)`).
   * **Əlaqələr (Relationships):** `HasOne`, `WithMany`, `HasForeignKey` istifadə edərək Cascade Delete davranışlarını (məsələn, `DeleteBehavior.Restrict`) tənzimləyin.
   * **İndekslər:** Email kimi sahələr üçün Unikal İndeks (`IsUnique`) qoyun.

### Addım 3: AppDbContext və Miqrasiya
1. `AppDbContext` faylında Fluent API konfiqurasiyalarını avtomatik yükləmək üçün `modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly())` yazın.
2. `Add-Migration InitialCreate` və `Update-Database` əmrləri ilə verilənlər bazasını yaradın.

### Addım 4: Biznes Məntiqi və Servislərin Yazılması
1. Təyin olunmuş interfeysləri tətbiq edən `ProductService`, `OrderService` və `PaymentService` siniflərini yaradın.
   * **Async/Await & AsNoTracking:** Oxuma metodlarında (`Get...`) `AsNoTracking()` istifadə edərək resurslara qənaət edin.
  




# "Система управления мини-интернет-магазином"

## 1. Архитектура системы и сущности

Система состоит из 6 основных сущностей (Entities):

1. **User** (Пользователь - Id, FullName, Email, Role, CreatedAt)
2. **Product** (Товар - Id, Name, Price, StockQuantity, IsActive)
3. **Category** (Категория - Id, Name, Description)
4. **Order** (Заказ - Id, UserId, OrderDate, Status, TotalAmount)
5. **OrderItem** (Элемент заказа - Id, OrderId, ProductId, Quantity, UnitPrice)
6. **Payment** (Оплата - Id, OrderId, PaymentDate, Amount, PaymentMethod, Status)

## 2. Перечисления (Enums)

```csharp
public enum UserRole
{
    Admin = 1,
    Customer = 2
}

public enum OrderStatus
{
    Pending = 1,
    Processing = 2,
    Shipped = 3,
    Delivered = 4,
    Cancelled = 5
}

public enum PaymentMethod
{
    CreditCard = 1,
    PayPal = 2,
    BankTransfer = 3
}

public enum PaymentStatus
{
    Pending = 1,
    Completed = 2,
    Failed = 3
}
```

## 3. Конфигурации EF Core (Fluent API)

Для каждой сущности должен быть создан отдельный файл конфигурации, реализующий интерфейс `IEntityTypeConfiguration<T>`. Внутри него необходимо реализовать метод `Configure`:

* **`UserConfiguration`**
  * `Configure(EntityTypeBuilder<User> builder)`
* **`ProductConfiguration`**
  * `Configure(EntityTypeBuilder<Product> builder)`
* **`CategoryConfiguration`**
  * `Configure(EntityTypeBuilder<Category> builder)`
* **`OrderConfiguration`**
  * `Configure(EntityTypeBuilder<Order> builder)`
* **`OrderItemConfiguration`**
  * `Configure(EntityTypeBuilder<OrderItem> builder)`
* **`PaymentConfiguration`**
  * `Configure(EntityTypeBuilder<Payment> builder)`

## 4. Структура AppDbContext

Класс `AppDbContext` должен содержать свойства `DbSet`, регистрацию конфигураций и переопределение методов:

* **Свойства:** `Users`, `Products`, `Categories`, `Orders`, `OrderItems`, `Payments`
* **Методы:**
  * `OnConfiguring(DbContextOptionsBuilder optionsBuilder)`
  * `OnModelCreating(ModelBuilder modelBuilder)`
  * `SaveChangesAsync(CancellationToken cancellationToken = default)` *(Для аудит-логов или логики Soft Delete)*

## 5. Интерфейсы и сервисы (Только сигнатуры методов)

### A. Управление категориями и товарами

```csharp
public interface IProductService
{
    Task<ProductDto> GetByIdAsync(int id);
    Task<List<ProductDto>> GetAllAsync();
    Task<List<ProductDto>> GetByCategoryIdAsync(int categoryId);
    Task CreateAsync(CreateProductDto dto);
    Task UpdateAsync(int id, UpdateProductDto dto);
    Task DeleteAsync(int id);
    Task UpdateStockAsync(int productId, int quantityChange);
}
```

### B. Управление заказами

```csharp
public interface IOrderService
{
    Task<OrderDto> GetOrderDetailsAsync(int orderId);
    Task<List<OrderDto>> GetOrdersByUserIdAsync(int userId);
    Task<int> CreateOrderAsync(CreateOrderDto dto);
    Task UpdateOrderStatusAsync(int orderId, OrderStatus status);
    Task CancelOrderAsync(int orderId);
}
```

### C. Оплата и автоматизация

```csharp
public interface IPaymentService
{
    Task<PaymentDto> GetPaymentByOrderIdAsync(int orderId);
    Task<bool> ProcessPaymentAsync(CreatePaymentDto dto);
}
```

---

