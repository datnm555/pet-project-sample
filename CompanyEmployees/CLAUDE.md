# CompanyEmployees API - Project Documentation for Claude

## Project Overview
This is an ASP.NET Core 8.0 Web API for managing companies and their employees with comprehensive features including authentication, API documentation with Swagger, versioning, and clean architecture.

## Architecture Structure

### Solution Projects
- **CompanyEmployees** - Main API host application
- **CompanyEmployees.Presentation** - Controllers and API endpoints
- **Service** - Business logic implementation
- **Service.Contracts** - Service interfaces
- **Repository** - Data access layer with Repository pattern
- **Contracts** - Repository interfaces
- **Entities** - Domain models and database entities
- **Shared** - DTOs and shared models
- **LoggerService** - NLog logging implementation

## Key Technologies
- .NET 8.0
- Entity Framework Core (SQL Server)
- ASP.NET Core Identity
- JWT Authentication
- Swagger/OpenAPI
- AutoMapper
- NLog
- API Versioning

## Database Configuration
- Connection String: Located in `appsettings.json`
- Database: SQL Server with database name "CompanyEmployee"
- Uses Entity Framework Core with Code First approach
- Seed data configured in `Repository/Configuration/` folder

## API Features

### Authentication & Security
- JWT Bearer token authentication
- Role-based authorization (Manager, Administrator roles)
- Token endpoint: `/api/token/refresh`
- Authentication endpoint: `/api/authentication`

### API Versioning
- Header-based versioning using "api-version" header
- Default version: 1.0
- Version 2.0 available for Companies controller (deprecated)

### Swagger Documentation
- Available at `/swagger` endpoint
- Two versions documented: v1 and v2
- API Explorer groups configured for version separation

### Rate Limiting
- Global rate limiting configured
- Specific policy: "SpecificPolicy" for certain endpoints
- Can be disabled per endpoint with `[DisableRateLimiting]`

### Caching
- Output caching with 120-second default policy
- Per-endpoint caching configuration available
- Response caching support

### Response Formats
- JSON (default)
- XML
- CSV (custom formatter)
- HATEOAS support with custom media types

## Core Entities

### Company
- Properties: Id (Guid), Name, Address, Country
- Relationship: One-to-many with Employees
- Validation: Required fields with max length constraints

### Employee
- Properties: Id (Guid), Name, Age, Position, CompanyId
- Relationship: Many-to-one with Company
- Validation: Required fields with max length constraints

### User (Identity)
- Extended from IdentityUser
- Properties: FirstName, LastName, RefreshToken, RefreshTokenExpiryTime
- Used for authentication and authorization

## API Endpoints Structure

### Companies
- `GET /api/companies` - Get all companies (requires Manager role)
- `GET /api/companies/{id}` - Get company by ID
- `POST /api/companies` - Create new company
- `PUT /api/companies/{id}` - Update company
- `DELETE /api/companies/{id}` - Delete company
- `GET /api/companies/collection/({ids})` - Get multiple companies

### Employees
- `GET /api/companies/{companyId}/employees` - Get employees for company
- `GET /api/companies/{companyId}/employees/{id}` - Get specific employee
- `POST /api/companies/{companyId}/employees` - Create employee
- `PUT /api/companies/{companyId}/employees/{id}` - Update employee
- `DELETE /api/companies/{companyId}/employees/{id}` - Delete employee
- `PATCH /api/companies/{companyId}/employees/{id}` - Partial update

### Authentication
- `POST /api/authentication/login` - User login
- `POST /api/authentication/register` - User registration
- `POST /api/token/refresh` - Refresh JWT token

## Development Commands

### Build and Run
```bash
# Build the solution
dotnet build

# Run the API
dotnet run --project CompanyEmployees/CompanyEmployees.csproj

# Run with hot reload
dotnet watch run --project CompanyEmployees/CompanyEmployees.csproj
```

### Database Commands
```bash
# Add migration
dotnet ef migrations add MigrationName --project Repository --startup-project CompanyEmployees

# Update database
dotnet ef database update --project Repository --startup-project CompanyEmployees

# Remove last migration
dotnet ef migrations remove --project Repository --startup-project CompanyEmployees
```

### Testing
```bash
# Run tests (if test project exists)
dotnet test

# Run with coverage
dotnet test /p:CollectCoverage=true
```

## Common Tasks

### Adding a New Entity
1. Create model in `Entities/Models`
2. Add DbSet in `Repository/RepositoryContext.cs`
3. Create configuration in `Repository/Configuration`
4. Add repository interface in `Contracts`
5. Implement repository in `Repository`
6. Create DTOs in `Shared/DataTransferObjects`
7. Add service interface in `Service.Contracts`
8. Implement service in `Service`
9. Add AutoMapper profile in `CompanyEmployees/MappingProfile.cs`
10. Create controller in `CompanyEmployees.Presentation/Controllers`

### Adding a New API Endpoint
1. Add method to controller in `CompanyEmployees.Presentation/Controllers`
2. Add service method in `Service.Contracts` and `Service`
3. Add repository method if needed in `Contracts` and `Repository`
4. Update Swagger documentation with XML comments
5. Add validation attributes or action filters if needed

### Modifying Authentication/Authorization
1. JWT settings in `appsettings.json` under "JwtSettings"
2. Identity configuration in `ServiceExtensions.ConfigureIdentity()`
3. JWT configuration in `ServiceExtensions.ConfigureJWT()`
4. Role configuration in `Repository/Configuration/RoleConfiguration.cs`

## Important Files and Locations

### Configuration
- `appsettings.json` - Application settings and connection strings
- `nlog.config` - Logging configuration
- `Program.cs` - Application startup and middleware pipeline

### Extensions
- `CompanyEmployees/Extensions/ServiceExtensions.cs` - Service registration
- `CompanyEmployees/Extensions/ExceptionMiddlewareExtensions.cs` - Exception handling

### Action Filters
- `CompanyEmployees.Presentation/ActionFilters/ValidationFilterAttribute.cs` - Model validation
- `CompanyEmployees.Presentation/ActionFilters/ValidateMediaTypeAttribute.cs` - Media type validation

### Utility
- `CompanyEmployees/Utility/EmployeeLinks.cs` - HATEOAS link generation
- `Repository/Extensions/RepositoryEmployeeExtensions.cs` - Employee query extensions

## Environment-Specific Settings

### Development
- Detailed error pages
- Swagger UI enabled
- HTTP allowed
- Detailed logging

### Production
- HSTS enabled
- HTTPS enforced
- Swagger can be disabled
- Minimal logging
- Rate limiting enforced

## Troubleshooting

### Common Issues
1. **Database connection fails**: Check connection string in appsettings.json
2. **JWT token invalid**: Verify JWT settings and secret key configuration
3. **Swagger not loading**: Ensure XML documentation file is generated
4. **CORS issues**: Check CORS policy in ServiceExtensions.cs
5. **Rate limiting blocking requests**: Adjust rate limit settings or disable for testing

### Debugging Tips
- Check `logs` folder for NLog output
- Use Swagger UI for API testing
- Monitor SQL queries with EF Core logging
- Check authentication with JWT debugger (jwt.io)

## Code Style and Conventions
- Use async/await for all database operations
- Follow Repository and Service pattern
- Use DTOs for API responses, never expose entities directly
- Implement global exception handling
- Use action filters for cross-cutting concerns
- Add XML documentation comments for Swagger
- Follow RESTful naming conventions for endpoints

## Notes for Future Development
- Consider implementing unit tests for services and repositories
- Add integration tests for API endpoints
- Implement API response compression
- Consider adding health checks endpoint
- Implement more sophisticated caching strategies
- Add API request/response logging middleware
- Consider implementing CQRS pattern for complex queries
- Add support for pagination metadata in response headers