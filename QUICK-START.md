# Quick Start Guide - Local Store Platform

**Last Updated:** October 22, 2025  
**Goal:** Get development environment running in 30 minutes

## Prerequisites Check

Before starting, ensure you have:

```bash
# Check Node.js version (need 20+)
node --version  # Should show v20.x.x or higher

# Check Python version (need 3.11+)
python3 --version  # Should show Python 3.11.x or higher

# Check Docker
docker --version
docker-compose --version

# Check Flutter (for mobile development)
flutter --version  # Should show Flutter 3.x
```

If any are missing:

- **Node.js:** Download from <https://nodejs.org/> (LTS version)
- **Python:** Download from <https://python.org/> or use `brew install python@3.11`
- **Docker:** Download from <https://docker.com/get-started>
- **Flutter:** Follow <https://flutter.dev/docs/get-started/install>

---

## Step 1: Initial Project Setup (5 minutes)

### 1.1 Create Backend Projects

```bash
# Navigate to your project root
cd /Users/thaidg/code/local-store-platform

# Install NestJS CLI globally
npm install -g @nestjs/cli

# Create NestJS backend
nest new backend --package-manager npm

# When prompted, select "npm" as package manager
```

### 1.2 Create Python AI Service

```bash
# Create Python virtual environment
mkdir ai-service
cd ai-service
python3.11 -m venv venv

# Activate virtual environment
source venv/bin/activate  # On macOS/Linux
# venv\Scripts\activate   # On Windows

# Create project structure
mkdir -p app/{api/routes,services,ml/models,database,tasks}
touch app/__init__.py app/main.py app/grpc_server.py

# Create requirements.txt
cat > requirements.txt << 'EOF'
fastapi==0.104.1
uvicorn[standard]==0.24.0
grpcio==1.59.3
grpcio-tools==1.59.3
asyncpg==0.29.0
redis==5.0.1
scikit-learn==1.3.2
pandas==2.1.3
numpy==1.26.2
celery==5.3.4
python-dotenv==1.0.0
EOF

# Install dependencies
pip install -r requirements.txt

cd ..
```

### 1.3 Create Proto Directory

```bash
# Create shared proto definitions
mkdir proto
cat > proto/ai_service.proto << 'EOF'
syntax = "proto3";

package ai_service;

service AIService {
  rpc GetRecommendations(RecommendationRequest) returns (RecommendationResponse);
}

message RecommendationRequest {
  string tenant_id = 1;
  string location_id = 2;
  int32 date_range_days = 3;
}

message Recommendation {
  string type = 1;
  string priority = 2;
  string title = 3;
  string description = 4;
  string impact = 5;
  string action_json = 6;
}

message RecommendationResponse {
  repeated Recommendation recommendations = 1;
  int32 data_points_used = 2;
}
EOF
```

---

## Step 2: Docker Development Environment (10 minutes)

### 2.1 Create Docker Compose File

```bash
# In project root
cat > docker-compose.yml << 'EOF'
version: '3.9'

services:
  postgres:
    image: postgres:14-alpine
    container_name: lsp-postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: local_store_platform
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: lsp-redis
    command: redis-server --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

volumes:
  postgres_data:
  redis_data:
EOF

# Start PostgreSQL and Redis
docker-compose up -d

# Verify services are running
docker-compose ps

# Check logs
docker-compose logs -f
```

### 2.2 Test Database Connection

```bash
# Connect to PostgreSQL
docker exec -it lsp-postgres psql -U postgres -d local_store_platform

# Inside psql, run:
# \l  -- List databases
# \q  -- Quit
```

---

## Step 3: Backend (NestJS) Setup (10 minutes)

### 3.1 Install Dependencies

```bash
cd backend

# Core dependencies
npm install @nestjs/graphql @nestjs/apollo @apollo/server graphql
npm install @nestjs/websockets @nestjs/platform-socket.io socket.io
npm install @nestjs/typeorm typeorm pg
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install @nestjs/cache-manager cache-manager cache-manager-redis-store redis
npm install @nestjs/bull bull
npm install @nestjs/config
npm install class-validator class-transformer
npm install @grpc/grpc-js @grpc/proto-loader

# Dev dependencies
npm install -D @types/passport-jwt
```

### 3.2 Create Environment File

```bash
cat > .env << 'EOF'
# Application
NODE_ENV=development
PORT=3000
API_PREFIX=api/v1

# Database
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=postgres
DATABASE_NAME=local_store_platform
DATABASE_SYNC=false

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0

# JWT
JWT_SECRET=dev-secret-change-in-production
JWT_ACCESS_EXPIRATION=15m
JWT_REFRESH_EXPIRATION=30d

# OTP
OTP_EXPIRATION=120
OTP_RATE_LIMIT=3

# gRPC
GRPC_AI_SERVICE_URL=localhost:5000

# CORS
CORS_ORIGINS=http://localhost:3001,http://localhost:8080

# GraphQL
GRAPHQL_PLAYGROUND=true
GRAPHQL_INTROSPECTION=true
EOF
```

### 3.3 Update Main File

Replace `src/main.ts` content:

```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const configService = app.get(ConfigService);

  app.setGlobalPrefix(configService.get('API_PREFIX'));
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      transform: true,
      forbidNonWhitelisted: true,
    }),
  );

  app.enableCors({
    origin: configService.get('CORS_ORIGINS')?.split(',') || '*',
    credentials: true,
  });

  const port = configService.get('PORT') || 3000;
  await app.listen(port);

  console.log(`🚀 NestJS API running on: http://localhost:${port}`);
  console.log(`🔍 GraphQL Playground: http://localhost:${port}/graphql`);
}

bootstrap();
```

### 3.4 Start Development Server

```bash
# Start in watch mode
npm run start:dev

# You should see:
# 🚀 NestJS API running on: http://localhost:3000
# 🔍 GraphQL Playground: http://localhost:3000/graphql
```

---

## Step 4: Python AI Service Setup (5 minutes)

### 4.1 Generate gRPC Code

```bash
cd ../ai-service
source venv/bin/activate

# Generate Python gRPC code from proto
python -m grpc_tools.protoc \
  -I../proto \
  --python_out=./app \
  --grpc_python_out=./app \
  ../proto/ai_service.proto

# Verify generated files
ls app/ai_service_pb2*.py
```

### 4.2 Create Main Application

```bash
cat > app/main.py << 'EOF'
from fastapi import FastAPI
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(
    title="Local Store Platform - AI Service",
    version="1.0.0",
)

@app.get("/health")
async def health_check():
    return {"status": "healthy", "service": "ai-service"}

@app.get("/")
async def root():
    return {
        "service": "Local Store Platform - AI Service",
        "version": "1.0.0",
        "docs": "/docs",
    }

if __name__ == "__main__":
    import uvicorn
    uvicorn.run("app.main:app", host="0.0.0.0", port=8000, reload=True)
EOF
```

### 4.3 Create gRPC Server Stub

```bash
cat > app/grpc_server.py << 'EOF'
import grpc
from concurrent import futures
import logging
import asyncio

logger = logging.getLogger(__name__)

async def serve():
    server = grpc.aio.server(futures.ThreadPoolExecutor(max_workers=10))
    # TODO: Add servicer
    
    port = "0.0.0.0:5000"
    server.add_insecure_port(port)
    
    logger.info(f"🚀 gRPC server starting on {port}")
    await server.start()
    await server.wait_for_termination()

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    asyncio.run(serve())
EOF
```

### 4.4 Create Environment File

```bash
cat > .env << 'EOF'
ENVIRONMENT=development
PORT=5000
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/local_store_platform
REDIS_URL=redis://localhost:6379/0
GRPC_PORT=5000
EOF
```

### 4.5 Start Services

```bash
# Terminal 1: FastAPI HTTP server
python -m app.main

# Terminal 2: gRPC server (separate terminal)
python -m app.grpc_server
```

---

## Step 5: Verify Everything Works (5 minutes)

### 5.1 Check All Services

```bash
# Check Docker services
docker-compose ps
# Should show postgres and redis as "Up"

# Check NestJS
curl http://localhost:3000/api/v1/health
# Should return: Cannot GET /api/v1/health (means server is running, route not implemented yet)

# Check Python FastAPI
curl http://localhost:8000/health
# Should return: {"status":"healthy","service":"ai-service"}

# Check Python gRPC
# Should see log: "🚀 gRPC server starting on 0.0.0.0:5000"

# Check PostgreSQL
docker exec -it lsp-postgres psql -U postgres -c "SELECT version();"

# Check Redis
docker exec -it lsp-redis redis-cli ping
# Should return: PONG
```

### 5.2 Check GraphQL Playground

1. Open browser: `http://localhost:3000/graphql`
2. You should see GraphQL Playground interface
3. Try a test query:

```graphql
query {
  __schema {
    types {
      name
    }
  }
}
```

---

## Next Steps

### Immediate (Day 1-2)

1. **Create Database Schema**
   - Follow `architecture/backend-setup-guide.md` Section: "Database Setup"
   - Run TypeORM migrations
   - Seed test data

2. **Implement Authentication**
   - Create AuthModule (phone OTP)
   - Test OTP flow end-to-end
   - See `planning/sprint-1-implementation.md` Week 1, Day 3-5

3. **Setup GraphQL Schema**
   - Follow `architecture/graphql-schema.md`
   - Implement base resolvers (User, Tenant, Location)
   - See `planning/sprint-1-implementation.md` Week 1, Day 6-7

### Week 1 Goals

- ✅ All services running in Docker
- ✅ NestJS API with authentication
- ✅ GraphQL queries operational
- ✅ Database with seed data
- ✅ Basic health checks passing

### Resources

- **Full Backend Setup:** `architecture/backend-setup-guide.md`
- **GraphQL Schema:** `architecture/graphql-schema.md`
- **Sprint 1 Plan:** `planning/sprint-1-implementation.md`
- **System Architecture:** `architecture/system-diagram.md`

---

## Troubleshooting

### Issue: Docker containers won't start

```bash
# Stop all containers
docker-compose down

# Remove volumes (WARNING: deletes data)
docker-compose down -v

# Rebuild and start
docker-compose up --build -d
```

### Issue: Port already in use

```bash
# Check what's using port 5432 (PostgreSQL)
lsof -i :5432

# Kill the process
kill -9 <PID>

# Or change port in docker-compose.yml
# ports:
#   - "5433:5432"  # Use 5433 externally
```

### Issue: NestJS dependencies won't install

```bash
# Clear npm cache
npm cache clean --force

# Remove node_modules and package-lock.json
rm -rf node_modules package-lock.json

# Reinstall
npm install
```

### Issue: Python virtual environment issues

```bash
# Deactivate current venv
deactivate

# Remove venv directory
rm -rf venv

# Recreate
python3.11 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Issue: gRPC code generation fails

```bash
# Make sure you're in ai-service directory
cd ai-service

# Activate venv
source venv/bin/activate

# Install grpcio-tools if missing
pip install grpcio-tools

# Regenerate
python -m grpc_tools.protoc \
  -I../proto \
  --python_out=./app \
  --grpc_python_out=./app \
  ../proto/ai_service.proto
```

---

## Development Workflow

### Daily Routine

```bash
# Morning: Start all services
docker-compose up -d

# Terminal 1: NestJS backend
cd backend
npm run start:dev

# Terminal 2: Python AI service
cd ai-service
source venv/bin/activate
python -m app.main

# Terminal 3: Python gRPC server
cd ai-service
source venv/bin/activate
python -m app.grpc_server

# Evening: Stop services
docker-compose down
```

### Making Changes

1. **Backend (NestJS)**
   - Edit files in `backend/src/`
   - Hot reload automatically applies changes
   - Run tests: `npm test`

2. **AI Service (Python)**
   - Edit files in `ai-service/app/`
   - FastAPI auto-reloads on changes
   - Run tests: `pytest`

3. **Database Changes**
   - Create migration: `npm run migration:generate -- -n MigrationName`
   - Run migrations: `npm run migration:run`

---

## Success Checklist

After completing this guide, you should have:

- ✅ PostgreSQL running on port 5432
- ✅ Redis running on port 6379
- ✅ NestJS API running on port 3000
- ✅ Python FastAPI running on port 8000
- ✅ Python gRPC server running on port 5000
- ✅ GraphQL Playground accessible
- ✅ All health checks passing
- ✅ Environment variables configured
- ✅ Proto files generated

---

## Getting Help

- **Architecture Questions:** See `architecture/system-diagram.md`
- **API Design:** See `architecture/graphql-schema.md`
- **Sprint Tasks:** See `planning/sprint-1-implementation.md`
- **Setup Issues:** Review `architecture/backend-setup-guide.md`

For Vietnamese-specific implementation details, refer to `.github/copilot-instructions.md`.

Good luck! 🚀
