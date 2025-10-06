# Implementation Plan: JWT Authentication with Session & Refresh Tokens

**Date:** 2025-10-06
**Project:** csharp-backend-api
**Status:** Draft - Awaiting Approval

---

## Overview

Implement a complete JWT-based authentication system for the C# Backend API with dual-token architecture:
- **Access Token (Session Token)**: Short-lived JWT for API authentication (15 minutes)
- **Refresh Token**: Long-lived token for extending sessions without re-login (7 days)

The implementation will follow .NET 8 best practices, integrate with the existing architecture, and provide secure user registration, login, token refresh, and logout capabilities.

---

## Requirements Summary

### Functional Requirements
- User registration with email and password
- User login with credentials returning access + refresh tokens
- Access token validation on protected endpoints
- Refresh token rotation for extending sessions
- Secure logout with token revocation
- User profile endpoint to retrieve authenticated user data

### Non-Functional Requirements
- Access tokens expire after 15 minutes
- Refresh tokens expire after 7 days
- Refresh tokens stored server-side in database
- Automatic refresh token rotation on renewal
- Password hashing with BCrypt (already available)
- Token revocation capability
- HTTP-only cookies for refresh tokens (optional/recommended)
- Comprehensive audit logging for auth events

---

## Research Findings

### Best Practices
1. **Short-lived access tokens**: 15-minute expiration balances security and user experience
2. **Server-side refresh token storage**: Enables revocation and security auditing
3. **Refresh token rotation**: Generate new refresh token on each renewal to prevent replay attacks
4. **Revoke descendant tokens**: If compromised token detected, revoke all related tokens
5. **Zero clock skew**: Enforce exact token expiration times
6. **HTTP-only cookies**: Store refresh tokens in HTTP-only cookies to prevent XSS attacks
7. **Fail-fast validation**: Validate all token claims (issuer, audience, expiration, signature)

### Reference Implementations
- **Microsoft Learn**: [Configure JWT Bearer Authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication)
- **C-Sharp Corner**: [Implementing JWT Refresh Tokens in .NET 8.0](https://www.c-sharpcorner.com/article/implementing-jwt-refresh-tokens-in-net-8-0/)
- **Code Maze**: [Using Refresh Tokens in ASP.NET Core Authentication](https://code-maze.com/using-refresh-tokens-in-asp-net-core-authentication/)

### Technology Decisions
- **BCrypt.Net-Next 4.0.3**: Already included for password hashing
- **System.IdentityModel.Tokens.Jwt**: JWT generation and validation (part of Microsoft.AspNetCore.Authentication.JwtBearer)
- **Entity Framework Core 8.0.10**: Already included for database operations
- **Newtonsoft.Json 13.0.3**: Already included for JSON serialization
- **SQL Server (LocalDB)**: Using existing MainConnection database

---

## Technical Design

### Architecture Diagram

```
┌─────────────┐         ┌──────────────────┐         ┌──────────────┐
│   Client    │────1───▶│  AuthController  │────2───▶│  AuthService │
│  (Browser/  │         │   /auth/login    │         │              │
│   Mobile)   │         │   /auth/register │         │ - Generate   │
└─────────────┘         │   /auth/refresh  │         │   Tokens     │
      ▲                 │   /auth/logout   │         │ - Validate   │
      │                 └──────────────────┘         │   Passwords  │
      │                          │                   └──────────────┘
      │                          │                          │
      │                          ▼                          ▼
      │                 ┌──────────────────┐         ┌──────────────┐
      └────────3────────│   JWT Middleware │         │   Database   │
                        │                  │         │              │
                        │ - Validate Token │         │ - Users      │
                        │ - Extract Claims │         │ - RefreshTok │
                        └──────────────────┘         └──────────────┘

Flow:
1. Client sends credentials to /auth/login
2. AuthController → AuthService validates & generates tokens
3. Client includes access token in Authorization header for protected endpoints
4. JWT Middleware validates token and extracts user claims
5. When access token expires, client uses refresh token at /auth/refresh
6. New access token + refresh token issued (rotation)
```

### Database Schema

```sql
-- Users Table
CREATE TABLE Users (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Email NVARCHAR(256) NOT NULL UNIQUE,
    PasswordHash NVARCHAR(MAX) NOT NULL,
    FirstName NVARCHAR(100),
    LastName NVARCHAR(100),
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);

-- RefreshTokens Table
CREATE TABLE RefreshTokens (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    UserId UNIQUEIDENTIFIER NOT NULL,
    Token NVARCHAR(MAX) NOT NULL,
    ExpiresAt DATETIME2 NOT NULL,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CreatedByIp NVARCHAR(50),
    RevokedAt DATETIME2 NULL,
    RevokedByIp NVARCHAR(50) NULL,
    ReplacedByToken NVARCHAR(MAX) NULL,
    ReasonRevoked NVARCHAR(500) NULL,
    IsExpired AS CASE WHEN GETUTCDATE() >= ExpiresAt THEN 1 ELSE 0 END,
    IsRevoked AS CASE WHEN RevokedAt IS NOT NULL THEN 1 ELSE 0 END,
    IsActive AS CASE WHEN RevokedAt IS NULL AND GETUTCDATE() < ExpiresAt THEN 1 ELSE 0 END,
    CONSTRAINT FK_RefreshTokens_Users FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE CASCADE
);

CREATE INDEX IX_RefreshTokens_UserId ON RefreshTokens(UserId);
CREATE INDEX IX_RefreshTokens_Token ON RefreshTokens(Token);
```

### API Contract

#### 1. Register User
```http
POST /auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecureP@ssw0rd",
  "firstName": "John",
  "lastName": "Doe"
}

Response 200:
{
  "status": "Success",
  "statusCode": 200,
  "result": {
    "message": "User registered successfully"
  }
}
```

#### 2. Login
```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecureP@ssw0rd"
}

Response 200:
{
  "status": "Success",
  "statusCode": 200,
  "result": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "a1b2c3d4e5f6...",
    "expiresAt": "2025-10-06T15:30:00Z",
    "user": {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe"
    }
  }
}
```

#### 3. Refresh Token
```http
POST /auth/refresh
Content-Type: application/json

{
  "refreshToken": "a1b2c3d4e5f6..."
}

Response 200:
{
  "status": "Success",
  "statusCode": 200,
  "result": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "z9y8x7w6v5u4...",
    "expiresAt": "2025-10-06T15:45:00Z"
  }
}
```

#### 4. Logout
```http
POST /auth/logout
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "refreshToken": "a1b2c3d4e5f6..."
}

Response 200:
{
  "status": "Success",
  "statusCode": 200,
  "result": {
    "message": "Logged out successfully"
  }
}
```

#### 5. Get Current User
```http
GET /auth/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response 200:
{
  "status": "Success",
  "statusCode": 200,
  "result": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe"
  }
}
```

### JWT Token Structure

**Access Token Claims:**
```json
{
  "sub": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "email": "user@example.com",
  "jti": "unique-token-id",
  "iss": "https://localhost:7258",
  "aud": "https://localhost:5050",
  "exp": 1696604400,
  "iat": 1696603500
}
```

**Refresh Token:** Cryptographically random 64-character string stored in database

---

## Files to Change / Create

### New Files to Create

1. **Models/Entities/**
   - `User.cs` - User entity for database
   - `RefreshToken.cs` - Refresh token entity for database

2. **Models/DTOs/**
   - `RegisterRequestDto.cs` - Registration input
   - `LoginRequestDto.cs` - Login input
   - `LoginResponseDto.cs` - Login output with tokens
   - `RefreshTokenRequestDto.cs` - Refresh input
   - `RefreshTokenResponseDto.cs` - Refresh output
   - `UserDto.cs` - User profile output

3. **Data/**
   - `ApplicationDbContext.cs` - EF Core context
   - `Migrations/` - EF Core migration files (auto-generated)

4. **Services/Interfaces/**
   - `IAuthService.cs` - Authentication service interface
   - `ITokenService.cs` - Token generation/validation interface

5. **Services/Implementations/**
   - `AuthService.cs` - Authentication business logic
   - `TokenService.cs` - JWT generation/validation logic

6. **Controllers/**
   - `AuthController.cs` - Authentication endpoints

7. **Validators/**
   - `RegisterRequestValidator.cs` - FluentValidation for registration
   - `LoginRequestValidator.cs` - FluentValidation for login

8. **Startup/**
   - `DatabaseConfiguration.cs` - **MODIFY** to register DbContext

### Files to Modify

1. **Startup/ServicesConfiguration.cs**
   - Register `IAuthService` and `ITokenService`

2. **Startup/AuthConfiguration.cs**
   - **Already configured** - No changes needed (JWT validation already set up)

3. **appsettings.json**
   - Add token expiration settings:
   ```json
   "JwtSettings": {
     "AccessTokenExpirationMinutes": 15,
     "RefreshTokenExpirationDays": 7
   }
   ```

4. **Constants.cs**
   - **Already has connection strings** - No changes needed

---

## Task Breakdown

### Phase 1: Foundation (Database & Models)

#### Task 1.1: Create Entity Models
- **Description**: Create `User` and `RefreshToken` entities following EF Core conventions
- **Files to create**:
  - `Models/Entities/User.cs`
  - `Models/Entities/RefreshToken.cs`
- **Dependencies**: None
- **Estimated effort**: 30 minutes
- **Complexity**: Low

#### Task 1.2: Create DTO Models
- **Description**: Create all DTOs for authentication requests/responses
- **Files to create**:
  - `Models/DTOs/RegisterRequestDto.cs`
  - `Models/DTOs/LoginRequestDto.cs`
  - `Models/DTOs/LoginResponseDto.cs`
  - `Models/DTOs/RefreshTokenRequestDto.cs`
  - `Models/DTOs/RefreshTokenResponseDto.cs`
  - `Models/DTOs/UserDto.cs`
- **Dependencies**: Task 1.1
- **Estimated effort**: 45 minutes
- **Complexity**: Low

#### Task 1.3: Create Database Context
- **Description**: Create `ApplicationDbContext` with Users and RefreshTokens DbSets
- **Files to create**:
  - `Data/ApplicationDbContext.cs`
- **Files to modify**:
  - `Startup/DatabaseConfiguration.cs` (register DbContext)
- **Dependencies**: Task 1.1
- **Estimated effort**: 30 minutes
- **Complexity**: Low

#### Task 1.4: Create and Apply EF Migrations
- **Description**: Generate initial migration and update database
- **Commands**:
  ```bash
  dotnet ef migrations add InitialAuthSchema
  dotnet ef database update
  ```
- **Dependencies**: Task 1.3
- **Estimated effort**: 15 minutes
- **Complexity**: Low

---

### Phase 2: Core Services Implementation

#### Task 2.1: Create Token Service Interface & Implementation
- **Description**: Implement JWT generation, validation, and refresh token creation
- **Files to create**:
  - `Services/Interfaces/ITokenService.cs`
  - `Services/Implementations/TokenService.cs`
- **Key methods**:
  - `GenerateAccessToken(User user)` → string
  - `GenerateRefreshToken()` → string
  - `GetPrincipalFromExpiredToken(string token)` → ClaimsPrincipal
  - `ValidateRefreshToken(string token)` → bool
- **Dependencies**: Task 1.1, Task 1.2
- **Estimated effort**: 2 hours
- **Complexity**: Medium

#### Task 2.2: Create Auth Service Interface & Implementation
- **Description**: Implement authentication business logic
- **Files to create**:
  - `Services/Interfaces/IAuthService.cs`
  - `Services/Implementations/AuthService.cs`
- **Key methods**:
  - `RegisterAsync(RegisterRequestDto dto)` → Task\<User\>
  - `LoginAsync(LoginRequestDto dto)` → Task\<LoginResponseDto\>
  - `RefreshTokenAsync(RefreshTokenRequestDto dto, string ipAddress)` → Task\<RefreshTokenResponseDto\>
  - `LogoutAsync(string refreshToken)` → Task
  - `GetCurrentUserAsync(Guid userId)` → Task\<UserDto\>
- **Dependencies**: Task 2.1, Task 1.3
- **Estimated effort**: 3 hours
- **Complexity**: High

#### Task 2.3: Register Services in DI Container
- **Description**: Register authentication and token services
- **Files to modify**:
  - `Startup/ServicesConfiguration.cs`
- **Code to add**:
  ```csharp
  services.AddScoped<ITokenService, TokenService>();
  services.AddScoped<IAuthService, AuthService>();
  ```
- **Dependencies**: Task 2.1, Task 2.2
- **Estimated effort**: 10 minutes
- **Complexity**: Low

---

### Phase 3: API Controllers & Validation

#### Task 3.1: Create FluentValidation Validators
- **Description**: Create validators for registration and login DTOs
- **Files to create**:
  - `Validators/RegisterRequestValidator.cs`
  - `Validators/LoginRequestValidator.cs`
- **Validation rules**:
  - Email: Required, valid email format, max 256 chars
  - Password: Required, min 8 chars, contains uppercase, lowercase, digit, special char
  - FirstName/LastName: Max 100 chars
- **Dependencies**: Task 1.2
- **Estimated effort**: 45 minutes
- **Complexity**: Low

#### Task 3.2: Create AuthController
- **Description**: Implement authentication endpoints
- **Files to create**:
  - `Controllers/AuthController.cs`
- **Endpoints**:
  - `POST /auth/register` - [AllowAnonymous]
  - `POST /auth/login` - [AllowAnonymous]
  - `POST /auth/refresh` - [AllowAnonymous]
  - `POST /auth/logout` - [Authorize]
  - `GET /auth/me` - [Authorize]
- **Dependencies**: Task 2.2, Task 3.1
- **Estimated effort**: 2 hours
- **Complexity**: Medium

---

### Phase 4: Configuration & Security

#### Task 4.1: Update Configuration Files
- **Description**: Add JWT token expiration settings
- **Files to modify**:
  - `appsettings.json`
  - `appsettings.Development.json`
- **Settings to add**:
  ```json
  "JwtSettings": {
    "AccessTokenExpirationMinutes": 15,
    "RefreshTokenExpirationDays": 7
  }
  ```
- **Dependencies**: None
- **Estimated effort**: 10 minutes
- **Complexity**: Low

#### Task 4.2: Create Configuration Model
- **Description**: Create strongly-typed configuration model for JWT settings
- **Files to create**:
  - `Models/Configuration/JwtConfigModel.cs`
- **Files to modify**:
  - `Startup/InitConfiguration.cs` (register config model)
- **Dependencies**: Task 4.1
- **Estimated effort**: 20 minutes
- **Complexity**: Low

---

### Phase 5: Testing

#### Task 5.1: Create Unit Tests for TokenService
- **Description**: Test JWT generation and validation logic
- **Files to create**:
  - `Tests/Services/TokenServiceTests.cs`
- **Test cases**:
  - `GenerateAccessToken_ValidUser_ReturnsValidJwt`
  - `GenerateRefreshToken_ReturnsRandomString`
  - `GetPrincipalFromExpiredToken_ValidToken_ReturnsClaimsPrincipal`
  - `GetPrincipalFromExpiredToken_InvalidToken_ThrowsException`
- **Dependencies**: Task 2.1
- **Estimated effort**: 1.5 hours
- **Complexity**: Medium

#### Task 5.2: Create Unit Tests for AuthService
- **Description**: Test authentication business logic
- **Files to create**:
  - `Tests/Services/AuthServiceTests.cs`
- **Test cases**:
  - `RegisterAsync_ValidDto_CreatesUser`
  - `RegisterAsync_DuplicateEmail_ThrowsException`
  - `LoginAsync_ValidCredentials_ReturnsTokens`
  - `LoginAsync_InvalidCredentials_ThrowsException`
  - `RefreshTokenAsync_ValidToken_ReturnsNewTokens`
  - `RefreshTokenAsync_ExpiredToken_ThrowsException`
  - `LogoutAsync_ValidToken_RevokesToken`
- **Dependencies**: Task 2.2
- **Estimated effort**: 2.5 hours
- **Complexity**: High

#### Task 5.3: Create Integration Tests for AuthController
- **Description**: Test end-to-end authentication flows
- **Files to create**:
  - `Tests/Controllers/AuthControllerTests.cs`
- **Test cases**:
  - `Register_ValidDto_Returns200`
  - `Login_ValidCredentials_ReturnsTokens`
  - `Refresh_ValidToken_ReturnsNewTokens`
  - `Logout_ValidToken_Returns200`
  - `GetMe_ValidToken_ReturnsUserProfile`
  - `GetMe_InvalidToken_Returns401`
- **Dependencies**: Task 3.2
- **Estimated effort**: 2 hours
- **Complexity**: Medium

---

## Milestones

### Milestone 1: Database Ready (Tasks 1.1 - 1.4)
- ✅ Entity models created
- ✅ DTOs created
- ✅ Database context configured
- ✅ Migrations applied
- **Duration**: 1 day
- **Deliverable**: Database schema ready for auth operations

### Milestone 2: Services Implemented (Tasks 2.1 - 2.3)
- ✅ Token service operational
- ✅ Auth service operational
- ✅ Services registered in DI
- **Duration**: 2 days
- **Deliverable**: Core authentication logic complete

### Milestone 3: API Endpoints Ready (Tasks 3.1 - 3.2)
- ✅ Validators created
- ✅ AuthController implemented
- **Duration**: 1 day
- **Deliverable**: All authentication endpoints accessible

### Milestone 4: Configuration Complete (Tasks 4.1 - 4.2)
- ✅ Settings configured
- ✅ Config models created
- **Duration**: 0.5 days
- **Deliverable**: Production-ready configuration

### Milestone 5: Tests Passing (Tasks 5.1 - 5.3)
- ✅ Unit tests passing
- ✅ Integration tests passing
- **Duration**: 2 days
- **Deliverable**: Comprehensive test coverage

**Total Estimated Duration**: 6.5 days

---

## Testing Plan

### Unit Tests
- **Framework**: xUnit
- **Mocking**: Moq
- **Coverage Target**: 80%+ for services

### Integration Tests
- **Framework**: xUnit + WebApplicationFactory
- **Database**: In-memory SQLite for testing
- **Coverage**: All API endpoints

### Manual Testing Checklist
- [ ] Register new user with valid data
- [ ] Register with duplicate email (should fail)
- [ ] Login with valid credentials
- [ ] Login with invalid credentials (should fail)
- [ ] Access protected endpoint with valid token
- [ ] Access protected endpoint with expired token (should fail)
- [ ] Refresh token with valid refresh token
- [ ] Refresh token with expired refresh token (should fail)
- [ ] Logout and attempt to use revoked refresh token (should fail)
- [ ] Token rotation: verify old refresh token is revoked

---

## Deployment Plan

### Pre-Deployment
1. **Environment Variables**: Ensure production `appsettings.Production.json` has:
   - Strong SecretKey (min 64 characters)
   - Production database connection string
   - Production issuer/audience URLs

2. **Database Migration**: Run migrations on production database:
   ```bash
   dotnet ef database update --connection "ProductionConnectionString"
   ```

### Deployment Steps
1. Build release version: `dotnet build -c Release`
2. Run tests: `dotnet test`
3. Publish app: `dotnet publish -c Release`
4. Deploy to hosting environment
5. Apply database migrations
6. Smoke test authentication endpoints

### Rollback Plan
1. Revert to previous deployment
2. If database migration was applied:
   ```bash
   dotnet ef migrations remove
   dotnet ef database update PreviousMigration
   ```

---

## Monitoring & Alerts

### Metrics to Track
- **Authentication success/failure rate**: Track login attempts and success percentage
- **Token refresh rate**: Monitor how often tokens are refreshed
- **Failed login attempts per IP**: Detect brute force attacks
- **Token validation errors**: Monitor invalid/expired token usage

### Logging Requirements
- **Info**: Successful login, logout, token refresh
- **Warning**: Failed login attempts, expired token usage
- **Error**: Token validation failures, database errors
- **Audit**: User registration, password changes

### Recommended Log Format
```json
{
  "timestamp": "2025-10-06T15:30:00Z",
  "level": "Info",
  "event": "UserLogin",
  "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "email": "user@example.com",
  "ipAddress": "192.168.1.1",
  "success": true
}
```

---

## Security, Privacy & Performance Considerations

### Security
- ✅ **Password hashing**: BCrypt with salt (already included)
- ✅ **Token signing**: HMAC-SHA256 with secret key
- ✅ **Token expiration**: Short-lived access tokens (15 min)
- ✅ **Refresh token rotation**: New token on each refresh
- ✅ **Token revocation**: Server-side storage enables revocation
- ⚠️ **HTTPS only**: Enforce HTTPS in production
- ⚠️ **Rate limiting**: Implement on `/auth/login` to prevent brute force
- ⚠️ **CORS**: Configure allowed origins properly

### Privacy
- ✅ **Password not logged**: Never log passwords or tokens
- ✅ **PII protection**: Email and personal data encrypted at rest
- ⚠️ **GDPR compliance**: Implement user data deletion endpoint

### Performance
- ✅ **Stateless authentication**: JWT validation doesn't hit database
- ⚠️ **Database indexing**: Index on `Users.Email` and `RefreshTokens.Token`
- ⚠️ **Token caching**: Consider caching user claims for frequently accessed tokens
- ⚠️ **Connection pooling**: EF Core connection pooling enabled by default

---

## Dependencies & Prerequisites

### NuGet Packages (Already Installed)
- ✅ `BCrypt.Net-Next` 4.0.3
- ✅ `Microsoft.AspNetCore.Authentication.JwtBearer` 8.0.20
- ✅ `Microsoft.EntityFrameworkCore.SqlServer` 8.0.10
- ✅ `Microsoft.EntityFrameworkCore.Tools` 8.0.10
- ✅ `FluentValidation.AspNetCore` 11.3.0
- ✅ `Newtonsoft.Json` 13.0.3

### Additional Packages Needed for Testing
- `xunit` - Testing framework
- `xunit.runner.visualstudio` - Test runner
- `Moq` - Mocking framework
- `Microsoft.AspNetCore.Mvc.Testing` - Integration testing
- `Microsoft.EntityFrameworkCore.InMemory` - In-memory database for tests

### Infrastructure Requirements
- SQL Server LocalDB (development)
- SQL Server 2019+ (production)
- .NET 8.0 SDK

### Secrets to Configure
- `AuthServer:SecretKey` - JWT signing key (min 64 chars, cryptographically random)
- `ConnectionStrings:MainConnection` - Database connection string

---

## Risk Assessment

### Risk 1: Token Secret Exposure
- **Impact**: High - Attackers can forge tokens
- **Probability**: Low - If proper secret management is followed
- **Mitigation**:
  - Use environment variables for secrets in production
  - Never commit secrets to source control
  - Rotate secret keys periodically
  - Use Azure Key Vault or similar for production

### Risk 2: Refresh Token Theft
- **Impact**: Medium - Attacker gains extended access
- **Probability**: Low - If HTTPS and HTTP-only cookies are used
- **Mitigation**:
  - Store refresh tokens in HTTP-only cookies
  - Implement token rotation
  - Detect and revoke descendant tokens on suspicious activity
  - Log all refresh token usage with IP tracking

### Risk 3: Brute Force Attacks
- **Impact**: Medium - Account compromise
- **Probability**: Medium - Common attack vector
- **Mitigation**:
  - Implement rate limiting on `/auth/login`
  - Account lockout after N failed attempts
  - CAPTCHA on repeated failures
  - Monitor and alert on failed login patterns

### Risk 4: Database Migration Failures
- **Impact**: High - Service downtime
- **Probability**: Low - EF migrations are generally reliable
- **Mitigation**:
  - Test migrations in staging environment first
  - Backup database before migration
  - Have rollback migration ready
  - Use idempotent migration scripts

### Risk 5: Performance Degradation
- **Impact**: Medium - Slow authentication responses
- **Probability**: Low - With proper indexing
- **Mitigation**:
  - Index `Users.Email` and `RefreshTokens.Token`
  - Implement database query monitoring
  - Set up performance benchmarks
  - Consider Redis caching for high-traffic scenarios

---

## Suggested Branch & PR

### Branch Name
```
feature/jwt-authentication-with-refresh-tokens
```

### PR Title Template
```
feat: Implement JWT Authentication with Session & Refresh Tokens
```

### PR Description Template
```markdown
## Summary
Implements complete JWT authentication system with dual-token architecture (access + refresh tokens).

## Changes
- ✅ User registration and login endpoints
- ✅ JWT token generation and validation
- ✅ Refresh token rotation mechanism
- ✅ Secure logout with token revocation
- ✅ Database schema for Users and RefreshTokens
- ✅ Comprehensive unit and integration tests

## Testing
- [x] Unit tests passing (80%+ coverage)
- [x] Integration tests passing
- [x] Manual testing completed
- [x] Security review completed

## Migrations
- Run `dotnet ef database update` after merge

## Breaking Changes
None - New feature addition

## Related Issues
Closes #XXX
```

---

## Implementation Options

### Option A: Approve Full Plan ✅ Recommended
- Implement all 15 tasks across 5 phases
- Full test coverage
- Production-ready with monitoring
- **Estimated time**: 6.5 days

### Option B: Implement Core Features Only
- Tasks 1.1 - 1.4 (Database)
- Tasks 2.1 - 2.3 (Services)
- Tasks 3.1 - 3.2 (API)
- Tasks 4.1 - 4.2 (Configuration)
- Skip testing phase initially
- **Estimated time**: 4.5 days

### Option C: Iterate on Plan
- Review and refine requirements
- Adjust token expiration times
- Modify security approach
- Add/remove features

### Option D: Cancel
- Do not proceed with implementation

---

## Next Steps

**Please select one of the following options:**

1. **Approve Plan** → I will begin implementation following the detailed task breakdown
2. **Implement Core** → I will implement essential features only (skip testing initially)
3. **Iterate** → Provide feedback and I will refine the plan
4. **Cancel** → Stop the feature planning process

**Awaiting your decision...**
