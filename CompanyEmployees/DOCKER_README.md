# Docker Setup for CompanyEmployees API

This guide explains how to run the CompanyEmployees API using Docker and Docker Compose.

## Prerequisites

- Docker Desktop installed (https://www.docker.com/products/docker-desktop)
- Docker Compose (included with Docker Desktop)
- .NET 8.0 SDK (for local development only)

## Quick Start

### 1. Build and Run with Docker Compose

```bash
# Navigate to the project root directory
cd CompanyEmployees

# Build and start all services
docker-compose up -d

# Or build and run in one command (with logs)
docker-compose up --build
```

### 2. Access the Application

Once the containers are running:

- **API Base URL**: http://localhost:5000
- **Swagger UI**: http://localhost:5000/swagger
- **HTTPS**: https://localhost:5001 (requires certificate setup)
- **SQL Server**: localhost,1433 (for external connections)

### 3. Initial Database Setup

The database will be created automatically on first run. To apply migrations and seed data:

```bash
# Connect to the API container
docker exec -it companyemployees-api bash

# Apply migrations (if needed)
dotnet ef database update

# Or run SQL scripts directly on SQL Server container
docker exec -it companyemployees-sqlserver /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U sa -P "YourStrong@Password123" \
  -Q "CREATE DATABASE CompanyEmployee IF NOT EXISTS"
```

## Docker Commands

### Basic Operations

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# Stop and remove volumes (clean database)
docker-compose down -v

# View logs
docker-compose logs -f

# View logs for specific service
docker-compose logs -f companyemployees-api
docker-compose logs -f sqlserver

# Rebuild images
docker-compose build

# Rebuild and start
docker-compose up -d --build
```

### Container Management

```bash
# List running containers
docker ps

# Access API container shell
docker exec -it companyemployees-api bash

# Access SQL Server container
docker exec -it companyemployees-sqlserver bash

# Check API health
curl http://localhost:5000/health
```

### Database Operations

```bash
# Connect to SQL Server from host
# Using SQL Server Management Studio or Azure Data Studio:
# Server: localhost,1433
# Username: sa
# Password: YourStrong@Password123

# Execute SQL command
docker exec -it companyemployees-sqlserver /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U sa -P "YourStrong@Password123" \
  -Q "SELECT name FROM sys.databases"

# Backup database
docker exec companyemployees-sqlserver /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U sa -P "YourStrong@Password123" \
  -Q "BACKUP DATABASE CompanyEmployee TO DISK = '/var/opt/mssql/backup/CompanyEmployee.bak'"
```

## Environment Configuration

### Docker Environment Variables

The `docker-compose.yml` file includes these key environment variables:

- `ASPNETCORE_ENVIRONMENT`: Set to "Docker" for Docker-specific settings
- `ConnectionStrings__sqlConnection`: Database connection string
- `JwtSettings__secretKey`: JWT secret (change in production!)

### Modifying Settings

1. **For development**: Edit `appsettings.Docker.json`
2. **For production**: Use environment variables in `docker-compose.yml`
3. **For secrets**: Consider using Docker secrets or Azure Key Vault

## HTTPS/SSL Setup

### Generate Development Certificate

```bash
# Generate certificate for HTTPS
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p YourCertPassword123
dotnet dev-certs https --trust

# On Windows
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p YourCertPassword123
dotnet dev-certs https --trust
```

## Testing the API

### Using Swagger UI

1. Navigate to http://localhost:5000/swagger
2. Try the available endpoints
3. For authenticated endpoints:
   - Register a user via `/api/authentication/register`
   - Login via `/api/authentication/login`
   - Use the JWT token in the Authorize button

### Using cURL

```bash
# Health check
curl http://localhost:5000/health

# Get all companies (requires authentication)
curl -X GET "http://localhost:5000/api/companies" \
  -H "accept: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"

# Register new user
curl -X POST "http://localhost:5000/api/authentication/register" \
  -H "accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Doe",
    "userName": "johndoe",
    "password": "Password123!",
    "email": "john@example.com",
    "phoneNumber": "1234567890",
    "roles": ["Manager"]
  }'
```

## Troubleshooting

### Common Issues

1. **Port already in use**
   ```bash
   # Change ports in docker-compose.yml or stop conflicting services
   docker-compose down
   lsof -i :5000  # On Mac/Linux
   netstat -an | findstr :5000  # On Windows
   ```

2. **Database connection issues**
   ```bash
   # Check if SQL Server is healthy
   docker-compose ps
   docker logs companyemployees-sqlserver

   # Restart SQL Server
   docker-compose restart sqlserver
   ```

3. **API not starting**
   ```bash
   # Check logs
   docker logs companyemployees-api

   # Rebuild image
   docker-compose build --no-cache companyemployees-api
   ```

4. **Permission issues**
   ```bash
   # Fix log directory permissions
   chmod 777 ./logs
   ```

### Viewing Logs

```bash
# All services
docker-compose logs -f

# Specific service with timestamp
docker-compose logs -f --timestamps companyemployees-api

# Last 100 lines
docker-compose logs --tail=100 companyemployees-api
```

## Production Deployment

### Security Considerations

1. **Change default passwords** in `docker-compose.yml`:
   - SQL Server SA password
   - JWT secret key
   - Certificate password

2. **Use secrets management**:
   ```yaml
   secrets:
     db_password:
       file: ./secrets/db_password.txt
     jwt_secret:
       file: ./secrets/jwt_secret.txt
   ```

3. **Enable HTTPS only**:
   - Remove HTTP port exposure
   - Force HTTPS redirect

4. **Limit SQL Server exposure**:
   - Remove port mapping for SQL Server in production
   - Use internal network only

### Optimization

1. **Multi-stage builds**: Already implemented in Dockerfile
2. **Layer caching**: Order Dockerfile commands for better caching
3. **Health checks**: Configured for both services
4. **Resource limits**: Add to docker-compose.yml:
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '2'
         memory: 2G
   ```

## Cleanup

```bash
# Stop and remove containers, networks
docker-compose down

# Remove containers, networks, and volumes
docker-compose down -v

# Remove all unused Docker resources
docker system prune -a

# Remove specific images
docker rmi companyemployees_companyemployees-api
```

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [.NET Docker Documentation](https://docs.microsoft.com/en-us/dotnet/core/docker/introduction)
- [SQL Server Docker Documentation](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-docker-container-deployment)