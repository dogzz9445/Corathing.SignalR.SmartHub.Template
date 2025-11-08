# CLAUDE.md

This document helps AI assistants (like Claude) understand the project structure, conventions, and technical decisions for better code assistance.

## 📦 Project Overview

**Corathing.SignalR.SmartHub.Template** is a WPF-ASP.NET SignalR template that uses attribute-based routing to eliminate manual Hub wrapper methods. It leverages Reflection with caching for dynamic service method invocation.

## 🏗️ Solution Structure
```
Corathing.SignalR.SmartHub.Template/
├── src/
│   ├── Corathing.SignalR.SmartHub.Server/          # ASP.NET Core Server
│   ├── Corathing.SignalR.SmartHub.Client.Wpf/      # WPF Client
│   ├── Corathing.SignalR.SmartHub.Shared/          # Shared DTOs & Attributes
│   └── Corathing.SignalR.SmartHub.Core/            # Core Reflection Engine
├── samples/                                         # Usage examples
├── docs/                                           # Documentation
└── tests/                                          # Unit tests
```

## 🎯 Core Concepts

### 1. HubMethod Attribute
- **Location**: `Shared/Attributes/HubMethodAttribute.cs`
- **Purpose**: Mark service methods for automatic Hub exposure
- **Usage**: `[HubMethod("GetCurrentUser")]`

### 2. SmartHub
- **Location**: `Server/Hubs/SmartHub.cs`
- **Purpose**: Single Hub that routes all method calls dynamically
- **Key Method**: `Task<object> Invoke(string methodName, params object[] args)`

### 3. MethodCacheBuilder
- **Location**: `Core/Reflection/MethodCacheBuilder.cs`
- **Purpose**: Cache Reflection results for performance
- **Initialization**: Scans all registered services on startup

### 4. SignalRClientService
- **Location**: `Client.Wpf/Services/SignalRClientService.cs`
- **Purpose**: Manages SignalR connection and method invocations
- **Usage**: Injected into ViewModels

## 📐 Architecture Patterns

### Server Side
```
Request Flow:
Client → SmartHub.Invoke(methodName, args) 
      → MethodCache.GetMethod(methodName) 
      → Reflection.Invoke(service, method, args) 
      → Return result
```

### Client Side (MVVM)
```
User Action → ViewModel → SignalRClientService 
           → InvokeAsync<T>("Invoke", methodName, args) 
           → Update ViewModel properties
```

## 🔧 Technical Decisions

### Why Reflection?
- **Problem**: Manual Hub wrappers create maintenance burden
- **Solution**: Automatic routing using Reflection
- **Trade-off**: Slight performance overhead (mitigated by caching)

### Why Attribute-Based?
- **Explicit**: Developers choose which methods to expose
- **Secure**: Not all service methods are automatically exposed
- **Documented**: Attributes serve as self-documentation

### Why Method Caching?
- **Performance**: Reflection is slow without caching
- **Strategy**: Cache MethodInfo objects on startup
- **Result**: First call uses Reflection, subsequent calls are fast

## 📝 Coding Conventions

### Naming
- **Services**: `{Domain}Service.cs` (e.g., `UserManagementService.cs`)
- **DTOs**: `{Entity}Dto.cs` (e.g., `UserDto.cs`)
- **ViewModels**: `{View}ViewModel.cs` (e.g., `MainViewModel.cs`)
- **Hub Methods**: Use PascalCase in attribute (e.g., `"GetCurrentUser"`)

### File Organization
```
Services/
  ├── UserManagementService.cs
  ├── DataCollectionService.cs
  └── ReportGenerationService.cs

DTOs/
  ├── UserDto.cs
  ├── DataDto.cs
  └── ReportDto.cs
```

### Attribute Usage Pattern
```csharp
public class ExampleService
{
    // ✅ Good: Clear method name, async, returns Task<T>
    [HubMethod("GetUserById")]
    public async Task<UserDto> GetUserByIdAsync(int userId)
    {
        return await _repository.GetByIdAsync(userId);
    }
    
    // ❌ Bad: Non-async, unclear naming
    [HubMethod("Get")]
    public UserDto GetUser(int id)
    {
        return _repository.GetById(id);
    }
}
```

## 🔑 Key Classes to Understand

### 1. HubMethodAttribute.cs
```csharp
[AttributeUsage(AttributeTargets.Method)]
public class HubMethodAttribute : Attribute
{
    public string MethodName { get; }
    public HubMethodAttribute(string methodName) => MethodName = methodName;
}
```

### 2. MethodCacheBuilder.cs
```csharp
public class MethodCacheBuilder
{
    private readonly Dictionary<string, CachedMethod> _cache = new();
    
    public void BuildCache(IServiceProvider services)
    {
        // Scan all services for [HubMethod] attributes
        // Cache MethodInfo, parameter types, return type
    }
    
    public async Task<object> InvokeMethod(string name, IServiceProvider services, object[] args)
    {
        // Retrieve from cache
        // Get service instance
        // Invoke method via Reflection
        // Handle async Task<T> results
    }
}
```

### 3. SmartHub.cs
```csharp
public class SmartHub : Hub
{
    private readonly IMethodCache _cache;
    private readonly IServiceProvider _services;
    
    public async Task<object> Invoke(string methodName, params object[] args)
    {
        return await _cache.InvokeMethod(methodName, _services, args);
    }
}
```

## 🚨 Common Pitfalls

### 1. Forgetting Async/Await
```csharp
// ❌ Wrong: Returns Task, doesn't await
[HubMethod("GetUser")]
public Task<UserDto> GetUser(int id)
{
    return _repository.GetByIdAsync(id); // Missing await
}

// ✅ Correct
[HubMethod("GetUser")]
public async Task<UserDto> GetUser(int id)
{
    return await _repository.GetByIdAsync(id);
}
```

### 2. Method Name Conflicts
```csharp
// ❌ Wrong: Same method name in different services
public class UserService
{
    [HubMethod("GetData")] // Conflict!
    public async Task<UserDto> GetUserData() { ... }
}

public class ReportService
{
    [HubMethod("GetData")] // Conflict!
    public async Task<ReportDto> GetReportData() { ... }
}

// ✅ Correct: Use unique names
[HubMethod("GetUserData")]
[HubMethod("GetReportData")]
```

### 3. Not Registering Services
```csharp
// ❌ Wrong: Service not registered in DI
public class NewService
{
    [HubMethod("DoSomething")]
    public async Task DoSomething() { ... }
}

// ✅ Correct: Register in Program.cs
builder.Services.AddScoped<NewService>();
```

## 🧪 Testing Guidelines

### Unit Testing Services
```csharp
[Fact]
public async Task GetUserById_ReturnsUser()
{
    // Arrange
    var service = new UserManagementService(mockRepo);
    
    // Act
    var result = await service.GetUserByIdAsync(1);
    
    // Assert
    Assert.NotNull(result);
}
```

### Integration Testing Hub
```csharp
[Fact]
public async Task SmartHub_InvokesServiceMethod()
{
    // Arrange
    var hub = new SmartHub(mockCache, mockServices);
    
    // Act
    var result = await hub.Invoke("GetUserById", 1);
    
    // Assert
    Assert.IsType<UserDto>(result);
}
```

## 📦 Dependencies

### Server
- `Microsoft.AspNetCore.SignalR` (latest)
- `Microsoft.Extensions.DependencyInjection`

### Client
- `Microsoft.AspNetCore.SignalR.Client` (latest)
- `CommunityToolkit.Mvvm`

### Shared
- No external dependencies (pure DTOs and attributes)

### Core
- `System.Reflection`
- `System.Linq.Expressions` (for future optimization)

## 🔄 Development Workflow

### Adding a New Service Method

1. **Create service method**:
```csharp
public class MyService
{
    [HubMethod("MyNewMethod")]
    public async Task<MyDto> MyNewMethodAsync()
    {
        return await DoWorkAsync();
    }
}
```

2. **Register service** (if new):
```csharp
builder.Services.AddScoped<MyService>();
```

3. **Call from client**:
```csharp
var result = await _signalR.InvokeAsync<MyDto>("Invoke", "MyNewMethod");
```

### That's it! No Hub changes needed.

## 🎨 Code Style

- **Async suffix**: Use `Async` for async methods
- **DTO suffix**: Use `Dto` for data transfer objects
- **Private fields**: Use `_camelCase` with underscore
- **Indentation**: 4 spaces (not tabs)
- **Braces**: Always use braces, even for single-line if statements

## 🔐 Security Considerations

- **Authentication**: Implement `[Authorize]` on SmartHub
- **Input Validation**: Validate all parameters in service methods
- **Method Exposure**: Only mark necessary methods with `[HubMethod]`
- **Error Handling**: Don't expose internal exception details to clients

## 📚 Resources

- [SignalR Documentation](https://docs.microsoft.com/aspnet/core/signalr)
- [Reflection Best Practices](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/reflection)
- [MVVM Pattern](https://docs.microsoft.com/dotnet/desktop/wpf/data/data-binding-overview)

## 🤖 AI Assistant Guidelines

When helping with this project:

1. **Always use `[HubMethod]` pattern** for new service methods
2. **Suggest async/await** for any I/O operations
3. **Recommend caching** for expensive Reflection operations
4. **Follow MVVM pattern** in WPF client
5. **Use dependency injection** for all services
6. **Write tests** for new functionality

---

Last Updated: 2025-11-08
Project Maintainer: Dongmin Jang (@Corathing)