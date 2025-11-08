# Corathing.SignalR.SmartHub.Template

> A lightweight WPF-ASP.NET SignalR template featuring attribute-based automatic routing

[![.NET](https://img.shields.io/badge/.NET-8.0-512BD4)](https://dotnet.microsoft.com/)
[![SignalR](https://img.shields.io/badge/SignalR-Latest-00ADD8)](https://dotnet.microsoft.com/apps/aspnet/signalr)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## ✨ Features

- 🎯 **Attribute-Based Routing**: Automatically generate Hub endpoints by adding `[HubMethod]`
- 🔄 **Dynamic Method Invocation**: Auto-mapping service methods using Reflection
- 🏗️ **Clean Architecture**: MVVM pattern + Service Layer separation
- ⚡ **High Performance**: Minimized Reflection overhead with method caching
- 📦 **Production Ready**: Enterprise-grade template

## 🚀 Quick Start

### 1️⃣ Run Server
```bash
cd src/Corathing.SignalR.SmartHub.Server
dotnet run
# Server: https://localhost:5001/hub
```

### 2️⃣ Run Client
```bash
cd src/Corathing.SignalR.SmartHub.Client.Wpf
dotnet run
```

## 📖 Usage

### Server: Add Attribute to Service
```csharp
public class UserManagementService
{
    [HubMethod("GetCurrentUser")]
    public async Task<UserDto> GetCurrentCollectionUserListId()
    {
        // Business logic
        return await GetUserAsync();
    }
    
    [HubMethod("GetAllUsers")]
    public async Task<List<UserDto>> GetUsers()
    {
        return await GetAllUsersAsync();
    }
}
```

### Server: SmartHub Routes Automatically
```csharp
public class SmartHub : Hub
{
    private readonly MethodCacheBuilder _cache;
    private readonly IServiceProvider _services;
    
    // Attribute-based automatic routing
    public async Task<object> Invoke(string methodName, params object[] args)
    {
        return await _cache.InvokeMethod(methodName, _services, args);
    }
}
```

### Client: Invoke Methods
```csharp
public class UserManagementViewModel : ObservableObject
{
    private readonly SignalRClientService _signalR;
    
    public async Task LoadCurrentUser()
    {
        var User = await _signalR.InvokeAsync<UserDto>(
            "Invoke", 
            "GetCurrentUser"
        );
        
        CurrentUser = User;
    }
}
```

## 🏗️ Architecture
```
┌─────────────────────────────────────────────────────┐
│              WPF Client (MVVM)                      │
│  ┌──────────────┐      ┌──────────────────────┐   │
│  │  ViewModel   │──────│  SignalRClient       │   │
│  └──────────────┘      │  Service             │   │
│         │               └──────────────────────┘   │
└─────────┼───────────────────────────────────────────┘
          │ SignalR Connection
          ▼
┌─────────────────────────────────────────────────────┐
│           ASP.NET Core Server                       │
│  ┌──────────────┐                                   │
│  │  SmartHub    │──────┐                           │
│  └──────────────┘      │                           │
│         │               ▼                           │
│         │        ┌─────────────────┐               │
│         └────────│ MethodCache     │               │
│                  │ (Reflection)    │               │
│                  └─────────────────┘               │
│                         │                           │
│                         ▼                           │
│         ┌──────────────────────────────┐           │
│         │ Business Services            │           │
│         │ [HubMethod] Decorated        │           │
│         └──────────────────────────────┘           │
└─────────────────────────────────────────────────────┘
```

## 🎯 Before vs After

### ❌ Before (Manual Wrapper)
```csharp
public class ServerHub : Hub
{
    private readonly UserService _UserService;
    private readonly DataService _dataService;
    private readonly ReportService _reportService;
    // ... 10+ service dependencies
    
    public async Task<string> GetCurrentUser()
        => await _UserService.GetCurrentUser();
    
    public async Task<List<Data>> GetData()
        => await _dataService.GetData();
    
    public async Task<Report> GenerateReport()
        => await _reportService.GenerateReport();
    
    // ... dozens of wrapper methods
}
```

### ✅ After (Automatic Routing)
```csharp
public class SmartHub : Hub
{
    private readonly IMethodCache _cache;
    
    public async Task<object> Invoke(string method, params object[] args)
        => await _cache.InvokeMethod(method, _services, args);
}

// Just add attribute to services
public class UserService
{
    [HubMethod("GetCurrentUser")]
    public async Task<string> GetCurrentUser() { ... }
}
```

## 📊 Performance

- **Method Caching**: Minimal Reflection overhead after first call
- **Async Support**: Full `async/await` support
- **Type Safety**: Compile-time type checking

## 📚 Project Structure

| Project | Description |
|---------|-------------|
| `Server` | ASP.NET Core SignalR Hub server |
| `Client.Wpf` | WPF MVVM client application |
| `Shared` | Shared DTOs and Attributes |
| `Core` | Reflection engine and utilities |

## 🛠️ Tech Stack

- **.NET 8.0**
- **ASP.NET Core SignalR**
- **WPF + CommunityToolkit.Mvvm**
- **Reflection + Expression Trees**

## 📝 Use Cases

- ✅ Desktop apps requiring server-client synchronization
- ✅ Exposing dozens of service methods as Hub endpoints
- ✅ Reducing Hub wrapper code writing/maintenance burden
- ✅ Plugin architecture (dynamic service addition)
- ✅ Real-time communication between microservices

## 🔧 How It Works

1. **Attribute Declaration**: Mark service methods with `[HubMethod("methodName")]`
2. **Cache Building**: On startup, scan all services and cache attributed methods
3. **Dynamic Invocation**: Hub receives method name and invokes via Reflection
4. **Result Return**: Async results are properly awaited and returned to client

## 🚦 Getting Started

### Prerequisites
- .NET 8.0 SDK or later
- Visual Studio 2022 or Rider

### Installation
```bash
git clone https://github.com/yourusername/Corathing.SignalR.SmartHub.Template.git
cd Corathing.SignalR.SmartHub.Template
dotnet restore
```

### Configuration
Update `appsettings.json` in the Server project:
```json
{
  "SignalR": {
    "HubPath": "/hub"
  }
}
```

## 📖 Documentation

- [Architecture](docs/Architecture.md) - Detailed architecture explanation
- [How To Use](docs/HowToUse.md) - Step-by-step usage guide
- [Performance](docs/Performance.md) - Performance considerations

## 🤝 Contributing

Issues and PRs are always welcome!

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👤 Author

**Corathing**
- Dongmin Jang

## 🙏 Acknowledgments

- Inspired by ASP.NET Core's attribute routing
- Built on top of Microsoft SignalR

---

⭐ If this project helped you, please consider giving it a star!

## 📞 Support

If you have any questions or need help, feel free to open an issue.

---

**Happy Coding!** 🚀