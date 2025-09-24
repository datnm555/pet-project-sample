# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy solution and project files
COPY *.sln ./
COPY CompanyEmployees/*.csproj ./CompanyEmployees/
COPY CompanyEmployees.Presentation/*.csproj ./CompanyEmployees.Presentation/
COPY Contracts/*.csproj ./Contracts/
COPY Entities/*.csproj ./Entities/
COPY LoggerService/*.csproj ./LoggerService/
COPY Repository/*.csproj ./Repository/
COPY Service/*.csproj ./Service/
COPY Service.Contracts/*.csproj ./Service.Contracts/
COPY Shared/*.csproj ./Shared/

# Restore dependencies
RUN dotnet restore

# Copy everything else
COPY . .

# Build the application
WORKDIR /src/CompanyEmployees
RUN dotnet build -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# Install curl for health checks
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Copy published application
COPY --from=publish /app/publish .

# Copy nlog.config to the app directory
COPY CompanyEmployees/nlog.config .

# Create logs directory
RUN mkdir -p /app/logs

# Expose ports
EXPOSE 80
EXPOSE 443

# Set environment variable
ENV ASPNETCORE_URLS=http://+:80
ENV ASPNETCORE_ENVIRONMENT=Development

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80/health || exit 1

# Run the application
ENTRYPOINT ["dotnet", "CompanyEmployees.dll"]