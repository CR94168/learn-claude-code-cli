# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Development Philosophy

### KISS (Keep It Simple, Stupid)

Simplicity should be a key goal in design. Choose straightforward solutions over complex ones whenever possible. Simple solutions are easier to understand, maintain, and debug.

### YAGNI (You Aren't Gonna Need It)

Avoid building functionality on speculation. Implement features only when they are needed, not when you anticipate they might be useful in the future.

### Design Principles

- **Dependency Inversion**: High-level modules should not depend on low-level modules. Both should depend on abstractions.
- **Open/Closed Principle**: Software entities should be open for extension but closed for modification.
- **Single Responsibility**: Each function, class, and module should have one clear purpose.
- **Fail Fast**: Check for potential errors early and raise exceptions immediately when issues occur.

## 🧱 Code Structure & Modularity

### File and Function Limits

- **Never create a file longer than 500 lines of code**. If approaching this limit, refactor by splitting into modules.
- **Functions should be under 50 lines** with a single, clear responsibility.
- **Classes should be under 100 lines** and represent a single concept or entity.
- **Organize code into clearly separated modules**, grouped by feature or responsibility.

### ⚙️ Project Architecture

### Project Architecture for Vue3 UI
```
vue3-project-name/
  ├── index.html                            # HTML entry point
  ├── vite.config.js                        # Vite build configuration
  ├── package.json                          # Dependencies & scripts
  │
  ├── public/                               # Static assets
  │   └── favicon.ico
  │
  └── src/                                  # Application source code
      ├── main.js                           # App entry point
      ├── App.vue                           # Root component
      │
      ├── assets/                           # Styles & resources
      │   └── scss/
      │
      ├── composables/                      # Reusable composition functions
      │
      ├── layouts/                          # Layout wrappers
      │   ├── BaseLayout.vue                # Standard layout with header/footer
      │   ├── FullScreenLayout.vue          # Full-screen layout for games
      │   └── components/
      │       ├── AppHeader.vue             # Top navigation bar
      │       └── AppFooter.vue             # Bottom navigation bar
      │
      ├── libs/                             # Library initialization
      │   ├── index.js                      # Centralized app setup
      │   ├── bootstrap.js                  # Bootstrap configuration
      │   ├── pinia.js                      # Pinia store setup
      │   └── router.js                     # Router setup
      │
      ├── router/                           # Vue Router configuration
      │   ├── index.js                      # Router instance
      │
      ├── services/                         # External service integrations
      │   ├── client.api.service.js         # Axios API client
      │   ├── storage.service.js            # LocalStorage wrapper
      │
      ├── stores/                  # Pinia state management
      │   ├── auth/                # Authentication store
      │   │   ├── index.js
      │   │   ├── state.js
      │   │   ├── actions.js
      │   │   └── getters.js
      │   ├── user/                # User data store
      │   │   ├── index.js
      │   │   ├── state.js
      │   │   └── actions.js
      │
      ├── utils/                   # Utility functions
      │   ├── formatters.js        # Data formatting helpers
      │
      └── views/                   # Page components
          ├── Home.vue             # Main view
```

🔧 Core Technologies
Dependencies
  | Package            | Version | Purpose                           |
  |--------------------|---------|-----------------------------------|
  | vue                | ^3.x.x  | Progressive JavaScript framework  |
  | vue-router         | ^4.x.x  | Official router for Vue           |
  | pinia              | ^3.x.x  | State management (Vuex successor) |
  | axios              | ^1.12.x | HTTP client for API calls         |
  | bootstrap          | ^5.3.7  | UI component framework            |
  | bootstrap-icons    | ^1.13.1 | Icon library                      |

Dev Dependencies
  | Package                 | Version | Purpose                        |
  |-------------------------|---------|--------------------------------|
  | vite                    | ^7.x.x  | Build tool & dev server        |
  | sass                    | ^1.x.0  | CSS preprocessor               |
  | unplugin-vue-components | ^29.x.x | Auto-import components         |
  | bootstrap-vue-next      | ^0.x.x  | Bootstrap components for Vue 3 |

🎯 Architecture Patterns

  1. Composition API - Uses Vue 3's Composition API with <script setup> syntax for better code organization and reusability.
  2. Modular Composables
  3. Centralized Services
  4. Layered Store Architecture - Pinia stores organized by domain - Each store split into state.js, actions.js, getters.js
  5. Route Organization

🔌 API Integration

    Base Configuration

    // vite.config.js proxy
    {
        "/api": "http://localhost:5000",    // REST API
    }

    Environment Variables .env

    VITE_API_URL=http://localhost:5000


🎨 Styling System

  SCSS Organization
  ```
  assets/scss/
  ├── index.scss              # Main entry
  ├── _variables.scss         # Design tokens
  ├── components/             # Component styles
  └── views/                  # View-specific styles
  ```

  Guidelines
  - Use @use instead of deprecated @import
  - Scoped styles in Vue components
  - External SCSS for large style blocks
  - Bootstrap theming with Telegram theme variables

🚀 Build & Development

  Scripts

  npm run dev       # Dev server on port 3000
  npm run build     # Production build
  npm run preview   # Preview production build
  npm run format    # Prettier formatting

📦 Component Auto-Import

  unplugin-vue-components auto-imports Bootstrap Vue components:
  <!-- No manual import needed -->
  <BButton variant="primary">Click Me</BButton>

🔐 Security Features

  - JWT token in Authorization header
  - Token auto-refresh mechanism
  - Request queue during token refresh
  - Secure logout with server cleanup
  - CORS handled by Vite proxy



### Project Architecture for C-Sharp .NET Backend API
```
csdotnet-api-project-name/
  │
  ├── Controllers/                          # API Controllers
  │   └── BaseController.cs                 # Base controller with common response methods
  │
  ├── Converters/                           # Custom JSON Converters
  │   └── BaseConverter.cs                  # Abstract base for custom JSON serialization
  │
  ├── Middleware/                           # Custom Middleware
  │   └── ExceptionMiddleware.cs            # Global exception handling
  │
  ├── Startup/                              # Startup Configuration (Modular)
  │   ├── AuthConfiguration.cs              # JWT authentication setup
  │   ├── CorsConfiguration.cs              # CORS policies
  │   ├── DatabaseConfiguration.cs          # EF Core DbContext registration
  │   ├── InitConfiguration.cs              # Config model initialization
  │   ├── MiddlewareConfiguration.cs        # Middleware pipeline setup
  │   ├── ServicesConfiguration.cs          # Custom service registration
  │   ├── SetupConfig.cs                    # Constants initialization
  │   └── SwashbuckleConfiguration.cs       # Swagger/OpenAPI setup
  │
  ├── Properties/                           # Project properties
  │   └── launchSettings.json               # Development launch profiles
  │
  ├── Constants.cs                          # Global constants (connection strings)
  ├── Program.cs                            # Application entry point
  ├── appsettings.json                      # Configuration (prod)
  ├── appsettings.Development.json          # Configuration (dev)
  └── csdotnet-api-project-name.csproj      # Project file
```

🎯 Architecture Patterns

  1. Layered Architecture with dependency on:
    - Business Project (Business logic layer)
    - Data Project (Data access layer)
  2. RESTful API with JWT Bearer authentication
  3. Centralized exception handling via middleware
  4. Standardized response format for all endpoints

  🔧 Core Technologies
  
  ```
  .NET 8 Web API
  ├── Entity Framework Core 8.0 (SQL Server)
  ├── JWT Authentication (Microsoft.AspNetCore.Authentication.JwtBearer)
  ├── FluentValidation (for DTO validation)
  ├── Newtonsoft.Json (JSON serialization)
  ├── Swashbuckle/Swagger (API documentation - dev only)
  └── BCrypt.Net-Next (password hashing)
  ```

    ---
    Core Components Deep Dive

    1. Program.cs (Entry Point)

        Configuration Flow:
        1. Services Registration (builder.Services):
            - Controllers with custom behavior options
            - Config constants initialization
            - Database context (EF Core)
            - Swagger (dev only)
            - Custom services
            - JWT configuration
            - CORS policies
        2. Middleware Pipeline (app.Use*):
            - Swagger UI (dev only)
            - HTTPS redirection
            - Custom exception middleware
            - CORS
            - Authentication → Authorization
            - Controller mapping

    ---
    2. Startup Configurations (Modular Pattern)

    DatabaseConfiguration.cs

    - Registers DbContext with SQL Server
    - Connection string from Constants, imported from appsettings configurations
    - Command timeout: 180 seconds
    - Scoped lifetime

    AuthConfiguration.cs

    - JWT Bearer authentication
    - Token validation parameters:
        - Validates: Issuer, Audience, Lifetime, Signing Key
        - Zero clock skew for exact expiration
        - Symmetric key from appsettings AuthServer:SecretKey

    CorsConfiguration.cs

    - Two approaches:
        a. Named policy AllowOrigin from config
        b. Global policy allowing all origins with credentials
    - Allows any method/header with credentials

    SwashbuckleConfiguration.cs

    - Development only Swagger setup
    - XML documentation support
    - Swagger UI at root path (/)
    - Collapsed by default

    SetupConfig.cs

    - Initializes static constants from appsettings:
        - database connection string
        - auth connection string

    InitConfiguration.cs

    - Configures JwtConfigModel from AuthServer section

    ServicesConfiguration.cs

    - Currently empty placeholder for custom service registration

    ---
    3. Controllers

    BaseController.cs

    Features:
    - Base class for all controllers
    - [Authorize] attribute by default
    - Route pattern: [controller]
    - Custom JSON settings (camelCase, ignore nulls/loops)

    Response Methods:
    - SuccessResponse(object? data) → Status: "Success"
    - ErrorResponse(object data, string message) → Status: "Error"
    - AddConverter(JsonConverter) → Add custom JSON converters
    - GetClaimValue(string claimType) → Extract JWT claims

    Response Model:
    HttpResponseModel {
        Status: "Success" | "Error" | "Failed" | "Unauthorized"
        StatusCode: int
        Message: string
        Result: object
        Exception: Exception
    }
    
    ---
    4. Middleware
    ExceptionMiddleware.cs

    Handles exceptions globally:

    | Exception Type              | Response Status | Behavior                                       |
    |-----------------------------|-----------------|------------------------------------------------|
    | InformationException        | Failed          | Returns exception message                      |
    | ValidationException         | Failed          | Returns FluentValidation errors (line-by-line) |
    | UnauthorizedAccessException | Unauthorized    | Returns 401 response                           |
    | Others                      | Error           | Logs error, returns "Server Error"             |

    Key Features:
    - Uses Newtonsoft.Json settings from MVC options
    - All responses return HTTP 200 (status in JSON body)
    - Validation errors concatenated with line breaks

    ---
    5. Converters

    BaseConverter

    - Abstract base for custom JSON converters
    - Generic type constraint for specific types
    - Access to IConfiguration if needed
    - Helper method GenerateJToken() for serialization
    - Handles null and DateTime.MinValue → JSON null

    ---
    6. Configuration Files

    appsettings.json

    {
        "ConnectionStrings": {
        "MidasGroupBotConnection": "localdb connection",
        "MidasAuthConnection": "external auth server DB"
        },
        "AllowedCorsOrigin": "http://localhost:3000",
        "AuthServer": {
        "Authority": "https://localhost:7258",
        "Issuer": "https://localhost:7258",
        "SecretKey": "shared secret with auth server",
        "Audience": "https://localhost:5050"
        },
        "RateLimit": {
        "Enable": true,
        "Limit": 100,
        "WindowInMinutes": 1
        }
    }

    Key Points:
    - Two databases:
        a. main app DB
        b. shared auth DB
    - JWT configured to trust external auth server
    - CORS allows Admin UI origin
    - Rate limiting configured

    ---
    Dependencies & References

    NuGet Packages:

    - BCrypt.Net-Next 4.0.3 - Password hashing
    - FluentValidation.AspNetCore 11.3.0 - DTO validation
    - Microsoft.AspNetCore.Authentication.JwtBearer 8.0.20 - JWT auth
    - Microsoft.AspNetCore.Mvc.NewtonsoftJson 6.0.0 - JSON serialization
    - Microsoft.EntityFrameworkCore.Tools 8.0.10 - EF migrations
    - Newtonsoft.Json 13.0.3 - JSON handling
    - Swashbuckle.AspNetCore 9.0.5 - API docs

    Project References:

    - Business.csproj - Business logic/DTOs
    - Data.csproj - Data access/EF Core

    ---
    Design Patterns Used

    1. Extension Methods Pattern - All startup configs use this IServiceCollection extensions
    2. Base Controller Pattern - Standardized response format
    3. Middleware Pipeline Pattern - Global exception handling
    4. Dependency Injection - All services injected via DI
    5. Repository Pattern - Via TelegramGroup.Data layer
    6. DTO Pattern - Via TelegramGroup.Business layer
    7. Converter Pattern - Custom JSON serialization

    ---
    Security Features

    1. JWT Authentication - Bearer token validation
    2. BCrypt Password Hashing - Secure password storage
    3. CORS Protection - Configured allowed origins
    4. HTTPS Redirection - Enforced in production
    5. FluentValidation - Input validation with Indonesian error messages
    6. Rate Limiting - Configured (implementation pending)


### Project Architecture for C-Sharp .NET Backend Worker
```

```

### Project Architecture for C-Sharp .NET Business Services
```

```

## Repository Purpose

This is a learning and research repository for experimenting with Claude Code CLI. The repository is currently minimal with no established codebase or project structure.

## Current State

- Empty repository initialized for learning purposes
- No build tools, dependencies, or test frameworks configured yet
- Intended for experimentation with Claude Code CLI features


