# ASP.NET Core MVC - Complete Reference Guide

## Table of Contents
- [Evolution of .NET Core](#evolution-of-net-core)
- [Benefits of .NET Core](#benefits-of-net-core)
- [Project Structure](#project-structure)
- [Core Files Explained](#core-files-explained)
- [MVC Architecture](#mvc-architecture)
- [Dependency Injection](#dependency-injection)
- [Routing System](#routing-system)
- [Views System](#views-system)
- [Interview Preparation Tips](#interview-preparation-tips)

## Evolution of .NET Core

The journey from WebForms to .NET Core represents a significant evolution in Microsoft's web development framework. Initially, WebForms provided a rapid development model familiar to Windows developers, using concepts like postbacks and viewstate. However, this approach had limitations in terms of control and performance.

ASP.NET MVC introduced in 2009 brought a separation of concerns and better testability. The real transformation came with .NET Core, which was rebuilt from the ground up to be cross-platform, modular, and cloud-ready.

### Timeline:
1. **ASP.NET WebForms (2002)**
   - Event-driven programming model
   - ViewState for state management
   - Server controls
   - Page lifecycle events

2. **ASP.NET MVC (2009)**
   - Model-View-Controller pattern
   - Clean HTML output
   - Full control over markup
   - Better testability

3. **ASP.NET Core (2016+)**
   - Cross-platform support
   - Dependency injection built-in
   - High performance
   - Modular architecture

## Benefits of .NET Core

### Performance
- Just-In-Time compilation
- Side-by-side versioning
- Optimized request pipeline
- Improved garbage collection

### Cross-Platform
- Runs on Windows, Linux, and macOS
- Docker container support
- Cloud-native capabilities
- Consistent development experience

### Modern Architecture
- Built-in dependency injection
- Middleware pipeline
- Configuration system
- Logging framework

### Development Efficiency
- Command-line tools
- Hot reload
- Package management via NuGet
- Modern tooling support

## Project Structure

### Essential Files and Directories

```
MyProject/
├── Program.cs
├── Startup.cs (pre .NET 6)
├── appsettings.json
├── appsettings.Development.json
├── MyProject.csproj
├── Controllers/
├── Models/
├── Views/
└── wwwroot/
```

## Core Files Explained

### Program.cs

The entry point of the application that configures services and middleware.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Service configuration
builder.Services.AddControllersWithViews();

var app = builder.Build();

// Middleware configuration
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### appsettings.json

Configuration file that stores application settings.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=MyDb;Trusted_Connection=True;"
  },
  "AllowedHosts": "*"
}
```

### Project File (.csproj)

Defines project configuration and dependencies.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Mvc" Version="2.2.0" />
  </ItemGroup>
</Project>
```

## MVC Architecture

### Model
- Represents data and business logic
- Independent of the presentation layer
- Can include validation and data access logic

```csharp
public class Product
{
    public int Id { get; set; }
    
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Range(0, 1000)]
    public decimal Price { get; set; }
}
```

### Controller
- Handles user input and interactions
- Updates the model and selects the view
- Contains action methods

```csharp
public class ProductsController : Controller
{
    private readonly IProductService _productService;
    
    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }
    
    public async Task<IActionResult> Index()
    {
        var products = await _productService.GetAllAsync();
        return View(products);
    }
}
```

### View
- Renders the user interface
- Uses Razor syntax
- Can use layouts and partial views

```cshtml
@model IEnumerable<Product>

<h1>Products</h1>

<div class="row">
    @foreach (var product in Model)
    {
        <div class="col-md-4">
            <h3>@product.Name</h3>
            <p>Price: @product.Price.ToString("C")</p>
        </div>
    }
</div>
```

## Dependency Injection

### Registration

Services are registered in Program.cs:

```csharp
// Singleton - One instance throughout the application
builder.Services.AddSingleton<IMyService, MyService>();

// Scoped - New instance for each request
builder.Services.AddScoped<IProductService, ProductService>();

// Transient - New instance each time requested
builder.Services.AddTransient<IEmailService, EmailService>();
```

### Usage

```csharp
public class ProductController : Controller
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductController> _logger;
    
    public ProductController(
        IProductService productService,
        ILogger<ProductController> logger)
    {
        _productService = productService;
        _logger = logger;
    }
    
    public async Task<IActionResult> Index()
    {
        _logger.LogInformation("Retrieving products");
        var products = await _productService.GetAllAsync();
        return View(products);
    }
}
```

## Routing System

### Convention-based Routing

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

### Attribute Routing

```csharp
[Route("api/[controller]")]
public class ProductsController : Controller
{
    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        if (product == null)
            return NotFound();
            
        return View(product);
    }
}
```

## Views System

### View Components

```csharp
public class CartSummaryViewComponent : ViewComponent
{
    private readonly ICartService _cartService;
    
    public CartSummaryViewComponent(ICartService cartService)
    {
        _cartService = cartService;
    }
    
    public async Task<IViewComponentResult> InvokeAsync()
    {
        var items = await _cartService.GetCartItemsAsync();
        return View(items);
    }
}
```

### Partial Views

```cshtml
@* _ProductCard.cshtml *@
@model Product

<div class="card">
    <div class="card-body">
        <h5 class="card-title">@Model.Name</h5>
        <p class="card-text">@Model.Description</p>
        <p class="card-text">@Model.Price.ToString("C")</p>
    </div>
</div>

@* Usage in main view *@
@foreach (var product in Model)
{
    <partial name="_ProductCard" model="product" />
}
```

## Interview Preparation Tips

### Key Concepts to Master
1. Dependency Injection and its lifecycle
2. Middleware pipeline and request processing
3. Configuration system and options pattern
4. Routing mechanisms and conventions
5. MVC design pattern implementation

### Common Interview Questions

**Q: Explain the request pipeline in ASP.NET Core.**  
A: The request pipeline is configured using middleware components. Each middleware component can:
- Process the incoming request
- Modify the request or response
- Pass control to the next middleware
- Short-circuit the pipeline

**Q: What are the benefits of dependency injection?**  
A: Benefits include:
- Loose coupling between components
- Easier unit testing through mocking
- Better code maintenance
- Flexibility in implementation changes
- Lifetime management of services

**Q: Explain the difference between AddTransient, AddScoped, and AddSingleton.**  
A: These are service lifetimes:
- AddTransient: New instance created every time
- AddScoped: Same instance within a request
- AddSingleton: Same instance for application lifetime

### Best Practices
1. Use dependency injection properly
2. Implement repository pattern
3. Keep controllers thin
4. Use async/await for I/O operations
5. Implement proper error handling
6. Use strongly-typed configuration
