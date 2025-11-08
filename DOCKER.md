# Docker Setup Guide

This guide explains how to run the BitsAll application using Docker.

## Prerequisites

- Docker Desktop installed and running
- Docker Compose installed (comes with Docker Desktop)

## Quick Start

1. **Build and start all services:**
   ```bash
   docker-compose up --build
   ```

2. **Access the application:**
   - API: http://localhost:8080
   - PostgreSQL: localhost:5432
   - Azurite (Blob Storage): localhost:10000

3. **Stop all services:**
   ```bash
   docker-compose down
   ```

4. **Stop and remove all data:**
   ```bash
   docker-compose down -v
   ```

## Services

### Application (app)
- **Port:** 8080
- Multi-stage build with Maven and OpenJDK
- Environment variables override application.properties

### PostgreSQL (postgres)
- **Port:** 5432
- **Database:** bitsall
- **Username:** postgres
- **Password:** postgres

### Azurite (Azure Blob Storage Emulator)
- **Port:** 10000
- Emulates Azure Blob Storage for local development
- Data stored in Docker volume

## Useful Commands

### View logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f app
docker-compose logs -f postgres
```

### Restart a service
```bash
docker-compose restart app
```

### Rebuild after code changes
```bash
docker-compose up --build app
```

### Access PostgreSQL
```bash
docker exec -it bitsall-postgres psql -U postgres -d bitsall
```

### Clean rebuild
```bash
docker-compose down -v
docker-compose up --build
```

## Environment Variables

You can override environment variables in the `docker-compose.yml` file:

```yaml
environment:
  SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/bitsalldb
  SPRING_DATASOURCE_USERNAME: postgres
  SPRING_DATASOURCE_PASSWORD: bitsall
  SPRING_CLOUD_AZURE_STORAGE_BLOB_CONNECTION_STRING: <your-connection-string>
  SPRING_CLOUD_AZURE_STORAGE_BLOB_CONTAINER_NAME: bits-all
```

Spring Boot automatically maps environment variables to properties:
- `SPRING_DATASOURCE_URL` → `spring.datasource.url`
- `SPRING_CLOUD_AZURE_STORAGE_BLOB_CONNECTION_STRING` → `spring.cloud.azure.storage.blob.connection-string`
- Use underscores (_) in environment variable names

## Development Workflow

1. Make code changes
2. Rebuild and restart:
   ```bash
   docker-compose up --build app
   ```
3. Test the API
4. View logs if needed:
   ```bash
   docker-compose logs -f app
   ```

## Troubleshooting

### Port already in use
If you get a "port already in use" error:
```bash
# Check what's using the port
lsof -i :8080
lsof -i :5432

# Kill the process or stop the conflicting service
```

### Database connection issues
```bash
# Check if PostgreSQL is ready
docker-compose ps
docker-compose logs postgres

# Restart PostgreSQL
docker-compose restart postgres
```

### Application crashes on startup
```bash
# View detailed logs
docker-compose logs app

# Check if database is healthy
docker-compose ps
```

## Production Deployment

For production, update the following:

1. **Change database credentials:**
   - Use strong passwords
   - Store credentials in secrets management

2. **Update Azure Storage:**
   - Replace Azurite with real Azure Blob Storage
   - Update connection string in environment variables

3. **Add health checks:**
   - Configure proper health endpoints
   - Set up monitoring

4. **Enable HTTPS:**
   - Configure SSL certificates
   - Update security configuration

5. **Set proper resource limits:**
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '2'
         memory: 2G
       reservations:
         cpus: '1'
         memory: 1G
   ```

