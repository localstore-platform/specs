# Backend Setup Guide - Hybrid NestJS + Python

**Created:** October 22, 2025  
**Status:** Implementation Ready  
**Prerequisites:** Node.js 20, Python 3.11, Docker, PostgreSQL 14, Redis 6

This guide walks through setting up the complete backend infrastructure for the Local Store Platform.

## Table of Contents

1. [Project Structure](#project-structure)
2. [NestJS API Gateway Setup](#nestjs-api-gateway-setup)
3. [Python AI Service Setup](#python-ai-service-setup)
4. [Database Setup](#database-setup)
5. [Docker Development Environment](#docker-development-environment)
6. [gRPC Communication Setup](#grpc-communication-setup)
7. [Testing Strategy](#testing-strategy)
8. [Deployment](#deployment)

---

## Project Structure

```
local-store-platform/
├── backend/                    # NestJS API Gateway
│   ├── src/
│   │   ├── main.ts
│   │   ├── app.module.ts
│   │   ├── common/            # Shared utilities
│   │   │   ├── decorators/
│   │   │   ├── filters/
│   │   │   ├── guards/
│   │   │   ├── interceptors/
│   │   │   └── pipes/
│   │   ├── config/            # Configuration
│   │   │   ├── database.config.ts
│   │   │   ├── redis.config.ts
│   │   │   └── graphql.config.ts
│   │   ├── modules/
│   │   │   ├── auth/          # Authentication (OTP, JWT)
│   │   │   ├── tenants/       # Multi-tenancy
│   │   │   ├── locations/     # Multi-location management
│   │   │   ├── dashboard/     # Metrics aggregation
│   │   │   ├── menu/          # Items, categories
│   │   │   ├── sales/         # Transactions
│   │   │   ├── recommendations/ # AI integration
│   │   │   ├── notifications/ # FCM, WebSocket
│   │   │   ├── payments/      # Webhook processing
│   │   │   └── integrations/  # gRPC client
│   │   └── database/
│   │       ├── entities/      # TypeORM entities
│   │       ├── migrations/    # Database migrations
│   │       └── seeds/         # Test data
│   ├── proto/                 # gRPC proto definitions
│   ├── test/                  # E2E tests
│   ├── package.json
│   ├── tsconfig.json
│   ├── nest-cli.json
│   └── .env.example
│
├── ai-service/                # Python AI Service
│   ├── app/
│   │   ├── main.py           # FastAPI app
│   │   ├── grpc_server.py    # gRPC service
│   │   ├── api/              # HTTP endpoints (admin)
│   │   │   ├── routes/
│   │   │   └── dependencies.py
│   │   ├── services/         # Business logic
│   │   │   ├── recommendation_service.py
│   │   │   ├── forecasting_service.py
│   │   │   └── pricing_service.py
│   │   ├── ml/               # ML models
│   │   │   ├── models/
│   │   │   ├── pipelines/
│   │   │   └── feature_engineering.py
│   │   ├── database/         # DB access
│   │   │   ├── client.py
│   │   │   └── queries.py
│   │   └── tasks/            # Celery tasks
│   │       ├── train_models.py
│   │       └── generate_reports.py
│   ├── proto/                # gRPC proto (synced with backend/)
│   ├── tests/
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
│
├── database/                  # Database management
│   ├── migrations/           # SQL migrations
│   ├── seeds/                # Seed data
│   └── schema.sql            # Master schema
│
├── proto/                     # Shared proto definitions
│   └── ai_service.proto
│
├── docker-compose.yml        # Local development
└── docker-compose.prod.yml   # Production setup
```

---

## NestJS API Gateway Setup

### Step 1: Initialize NestJS Project

```bash
# Install NestJS CLI globally
npm install -g @nestjs/cli

# Create new project
cd local-store-platform
nest new backend --package-manager npm

# Navigate to backend
cd backend
```

### Step 2: Install Dependencies

```bash
# Core dependencies
npm install @nestjs/graphql @nestjs/apollo @apollo/server graphql
npm install @nestjs/websockets @nestjs/platform-socket.io socket.io
npm install @nestjs/typeorm typeorm pg
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install @nestjs/cache-manager cache-manager
npm install @nestjs/bull bull
npm install class-validator class-transformer
npm install @grpc/grpc-js @grpc/proto-loader

# Redis
npm install cache-manager-redis-store redis

# Utilities
npm install @nestjs/config
npm install bcrypt argon2
npm install twilio  # SMS OTP
npm install firebase-admin  # FCM push notifications
npm install aws-sdk  # S3 uploads

# Dev dependencies
npm install -D @types/node
npm install -D @types/passport-jwt
npm install -D @types/bcrypt
npm install -D @types/cache-manager
```

### Step 3: Project Configuration

**`backend/.env.example`**

```env
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
DATABASE_SYNC=false  # Use migrations in production

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0

# JWT
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_ACCESS_EXPIRATION=15m
JWT_REFRESH_EXPIRATION=30d

# OTP
OTP_EXPIRATION=120  # seconds
OTP_RATE_LIMIT=3    # per 10 minutes

# SMS Provider (Twilio)
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_PHONE_NUMBER=+1234567890

# Firebase (FCM Push Notifications)
FIREBASE_PROJECT_ID=your-firebase-project
FIREBASE_PRIVATE_KEY=your-firebase-private-key
FIREBASE_CLIENT_EMAIL=your-service-account@firebase.gserviceaccount.com

# AWS S3
AWS_REGION=ap-southeast-1
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_S3_BUCKET=local-store-platform-uploads

# gRPC (Python AI Service)
GRPC_AI_SERVICE_URL=localhost:5000

# Rate Limiting
THROTTLE_TTL=60  # seconds
THROTTLE_LIMIT=60  # requests per TTL

# CORS
CORS_ORIGINS=http://localhost:3001,http://localhost:8080

# GraphQL
GRAPHQL_PLAYGROUND=true  # Disable in production
GRAPHQL_INTROSPECTION=true  # Disable in production
```

**`backend/src/main.ts`**

```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Get config service
  const configService = app.get(ConfigService);

  // Global prefix
  app.setGlobalPrefix(configService.get('API_PREFIX'));

  // Validation pipe
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      transform: true,
      forbidNonWhitelisted: true,
    }),
  );

  // CORS
  app.enableCors({
    origin: configService.get('CORS_ORIGINS').split(','),
    credentials: true,
  });

  const port = configService.get('PORT') || 3000;
  await app.listen(port);

  console.log(`🚀 Application is running on: http://localhost:${port}`);
  console.log(`🔍 GraphQL Playground: http://localhost:${port}/graphql`);
}

bootstrap();
```

**`backend/src/app.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { CacheModule } from '@nestjs/cache-manager';
import { BullModule } from '@nestjs/bull';
import * as redisStore from 'cache-manager-redis-store';
import { join } from 'path';

// Modules
import { AuthModule } from './modules/auth/auth.module';
import { TenantsModule } from './modules/tenants/tenants.module';
import { LocationsModule } from './modules/locations/locations.module';
import { DashboardModule } from './modules/dashboard/dashboard.module';
import { MenuModule } from './modules/menu/menu.module';
import { SalesModule } from './modules/sales/sales.module';
import { RecommendationsModule } from './modules/recommendations/recommendations.module';
import { NotificationsModule } from './modules/notifications/notifications.module';
import { PaymentsModule } from './modules/payments/payments.module';
import { IntegrationsModule } from './modules/integrations/integrations.module';

@Module({
  imports: [
    // Configuration
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
    }),

    // TypeORM (PostgreSQL)
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DATABASE_HOST'),
        port: configService.get('DATABASE_PORT'),
        username: configService.get('DATABASE_USERNAME'),
        password: configService.get('DATABASE_PASSWORD'),
        database: configService.get('DATABASE_NAME'),
        entities: [join(__dirname, '**', '*.entity.{ts,js}')],
        synchronize: configService.get('DATABASE_SYNC') === 'true',
        logging: configService.get('NODE_ENV') === 'development',
      }),
      inject: [ConfigService],
    }),

    // GraphQL
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
        sortSchema: true,
        playground: configService.get('GRAPHQL_PLAYGROUND') === 'true',
        introspection: configService.get('GRAPHQL_INTROSPECTION') === 'true',
        context: ({ req, connection }) => {
          // WebSocket connection
          if (connection) {
            return { req: connection.context };
          }
          // HTTP request
          return { req };
        },
        subscriptions: {
          'graphql-ws': true,
          'subscriptions-transport-ws': true,
        },
      }),
      inject: [ConfigService],
    }),

    // Redis Cache
    CacheModule.registerAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        store: redisStore,
        host: configService.get('REDIS_HOST'),
        port: configService.get('REDIS_PORT'),
        password: configService.get('REDIS_PASSWORD'),
        db: configService.get('REDIS_DB'),
        ttl: 60, // Default TTL (seconds)
      }),
      inject: [ConfigService],
      isGlobal: true,
    }),

    // Bull Queue (Background Jobs)
    BullModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        redis: {
          host: configService.get('REDIS_HOST'),
          port: configService.get('REDIS_PORT'),
          password: configService.get('REDIS_PASSWORD'),
        },
      }),
      inject: [ConfigService],
    }),

    // Application Modules
    AuthModule,
    TenantsModule,
    LocationsModule,
    DashboardModule,
    MenuModule,
    SalesModule,
    RecommendationsModule,
    NotificationsModule,
    PaymentsModule,
    IntegrationsModule,
  ],
})
export class AppModule {}
```

### Step 4: Create Base Entity

**`backend/src/database/entities/base.entity.ts`**

```typescript
import {
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  Column,
} from 'typeorm';
import { Field, ID, ObjectType } from '@nestjs/graphql';

@ObjectType({ isAbstract: true })
export abstract class BaseEntity {
  @Field(() => ID)
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Field()
  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @Field()
  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  @Column({ name: 'deleted_at', nullable: true })
  deletedAt?: Date;
}
```

**`backend/src/database/entities/tenant.entity.ts`**

```typescript
import { Entity, Column, OneToMany } from 'typeorm';
import { Field, ObjectType } from '@nestjs/graphql';
import { BaseEntity } from './base.entity';
import { Location } from './location.entity';
import { User } from './user.entity';

@ObjectType()
@Entity('tenants')
export class Tenant extends BaseEntity {
  @Field()
  @Column({ length: 255 })
  name: string;

  @Field()
  @Column({ length: 20, unique: true })
  phone: string;

  @Field({ nullable: true })
  @Column({ length: 255, nullable: true })
  email?: string;

  @Field()
  @Column({ default: 'active' })
  status: string;

  @Field()
  @Column({ name: 'subscription_tier', default: 'free' })
  subscriptionTier: string;

  @Field()
  @Column({ name: 'trial_ends_at', nullable: true })
  trialEndsAt?: Date;

  @Field(() => [Location])
  @OneToMany(() => Location, (location) => location.tenant)
  locations: Location[];

  @Field(() => [User])
  @OneToMany(() => User, (user) => user.tenant)
  users: User[];
}
```

### Step 5: Authentication Module (Phone OTP)

**`backend/src/modules/auth/auth.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { AuthService } from './auth.service';
import { AuthResolver } from './auth.resolver';
import { AuthController } from './auth.controller';
import { JwtStrategy } from './strategies/jwt.strategy';
import { User } from '../../database/entities/user.entity';
import { Tenant } from '../../database/entities/tenant.entity';

@Module({
  imports: [
    PassportModule.register({ defaultStrategy: 'jwt' }),
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        secret: configService.get('JWT_SECRET'),
        signOptions: {
          expiresIn: configService.get('JWT_ACCESS_EXPIRATION'),
        },
      }),
      inject: [ConfigService],
    }),
    TypeOrmModule.forFeature([User, Tenant]),
  ],
  providers: [AuthService, AuthResolver, JwtStrategy],
  controllers: [AuthController],
  exports: [AuthService, JwtStrategy, PassportModule],
})
export class AuthModule {}
```

**`backend/src/modules/auth/auth.service.ts`**

```typescript
import { Injectable, UnauthorizedException, BadRequestException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { ConfigService } from '@nestjs/config';
import { User } from '../../database/entities/user.entity';
import { Tenant } from '../../database/entities/tenant.entity';
import { Cache } from 'cache-manager';
import { Inject } from '@nestjs/common';
import { CACHE_MANAGER } from '@nestjs/cache-manager';

@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
    @InjectRepository(Tenant)
    private tenantsRepository: Repository<Tenant>,
    private jwtService: JwtService,
    private configService: ConfigService,
    @Inject(CACHE_MANAGER)
    private cacheManager: Cache,
  ) {}

  /**
   * Request OTP for phone number
   */
  async requestOtp(phone: string): Promise<{ message: string; expiresIn: number }> {
    // Normalize phone (Vietnamese format)
    const normalizedPhone = this.normalizePhone(phone);

    // Rate limiting check
    const rateLimitKey = `otp:rate:${normalizedPhone}`;
    const attempts = await this.cacheManager.get<number>(rateLimitKey);
    
    if (attempts && attempts >= 3) {
      throw new BadRequestException('Quá nhiều yêu cầu. Vui lòng thử lại sau 10 phút.');
    }

    // Generate 6-digit OTP
    const otp = this.generateOtp();

    // Store OTP in Redis (120s TTL)
    const otpKey = `otp:${normalizedPhone}`;
    const expiresIn = this.configService.get<number>('OTP_EXPIRATION');
    await this.cacheManager.set(otpKey, otp, expiresIn);

    // Increment rate limit counter (10 minutes TTL)
    await this.cacheManager.set(rateLimitKey, (attempts || 0) + 1, 600);

    // Send SMS (via Twilio or local provider)
    await this.sendSms(normalizedPhone, `Mã OTP của bạn: ${otp}. Có hiệu lực trong 2 phút.`);

    // In development, log OTP
    if (this.configService.get('NODE_ENV') === 'development') {
      console.log(`📱 OTP for ${normalizedPhone}: ${otp}`);
    }

    return {
      message: 'Mã OTP đã được gửi đến số điện thoại của bạn.',
      expiresIn,
    };
  }

  /**
   * Verify OTP and issue JWT tokens
   */
  async verifyOtp(phone: string, otp: string): Promise<{ accessToken: string; refreshToken: string; user: User }> {
    const normalizedPhone = this.normalizePhone(phone);
    const otpKey = `otp:${normalizedPhone}`;

    // Get stored OTP
    const storedOtp = await this.cacheManager.get<string>(otpKey);

    if (!storedOtp) {
      throw new UnauthorizedException('Mã OTP đã hết hạn.');
    }

    if (storedOtp !== otp) {
      throw new UnauthorizedException('Mã OTP không chính xác.');
    }

    // Delete OTP after successful verification
    await this.cacheManager.del(otpKey);

    // Find or create user
    let user = await this.usersRepository.findOne({
      where: { phone: normalizedPhone },
      relations: ['tenant'],
    });

    if (!user) {
      // Create new tenant and user
      const tenant = this.tenantsRepository.create({
        name: `Cửa hàng ${normalizedPhone}`,
        phone: normalizedPhone,
        status: 'active',
        subscriptionTier: 'free',
        trialEndsAt: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000), // 30 days trial
      });
      await this.tenantsRepository.save(tenant);

      user = this.usersRepository.create({
        phone: normalizedPhone,
        tenant,
        role: 'owner',
        isActive: true,
      });
      await this.usersRepository.save(user);
    }

    // Generate JWT tokens
    const payload = { sub: user.id, phone: user.phone, tenantId: user.tenant.id };
    const accessToken = this.jwtService.sign(payload);
    const refreshToken = this.jwtService.sign(payload, {
      expiresIn: this.configService.get('JWT_REFRESH_EXPIRATION'),
    });

    // Store refresh token in Redis
    await this.cacheManager.set(`refresh:${user.id}`, refreshToken, 30 * 24 * 60 * 60); // 30 days

    return { accessToken, refreshToken, user };
  }

  /**
   * Refresh access token
   */
  async refreshToken(refreshToken: string): Promise<{ accessToken: string }> {
    try {
      const payload = this.jwtService.verify(refreshToken);
      const storedToken = await this.cacheManager.get<string>(`refresh:${payload.sub}`);

      if (storedToken !== refreshToken) {
        throw new UnauthorizedException('Token không hợp lệ.');
      }

      const newAccessToken = this.jwtService.sign({
        sub: payload.sub,
        phone: payload.phone,
        tenantId: payload.tenantId,
      });

      return { accessToken: newAccessToken };
    } catch (error) {
      throw new UnauthorizedException('Token không hợp lệ hoặc đã hết hạn.');
    }
  }

  /**
   * Helper: Generate 6-digit OTP
   */
  private generateOtp(): string {
    return Math.floor(100000 + Math.random() * 900000).toString();
  }

  /**
   * Helper: Normalize phone number (Vietnamese format)
   */
  private normalizePhone(phone: string): string {
    // Remove spaces, dashes, parentheses
    let normalized = phone.replace(/[\s\-\(\)]/g, '');

    // Convert +84 to 0
    if (normalized.startsWith('+84')) {
      normalized = '0' + normalized.slice(3);
    } else if (normalized.startsWith('84')) {
      normalized = '0' + normalized.slice(2);
    }

    // Validate format (10 digits starting with 0)
    if (!/^0\d{9}$/.test(normalized)) {
      throw new BadRequestException('Số điện thoại không hợp lệ.');
    }

    return normalized;
  }

  /**
   * Helper: Send SMS (Twilio)
   */
  private async sendSms(phone: string, message: string): Promise<void> {
    // TODO: Integrate Twilio or Vietnamese SMS provider
    // For now, just log in development
    if (this.configService.get('NODE_ENV') === 'development') {
      console.log(`📨 SMS to ${phone}: ${message}`);
      return;
    }

    // Production: Send via Twilio
    // const twilio = require('twilio');
    // const client = twilio(this.configService.get('TWILIO_ACCOUNT_SID'), this.configService.get('TWILIO_AUTH_TOKEN'));
    // await client.messages.create({ body: message, from: this.configService.get('TWILIO_PHONE_NUMBER'), to: phone });
  }
}
```

---

## Python AI Service Setup

### Step 1: Initialize Python Project

```bash
cd local-store-platform
mkdir ai-service
cd ai-service

# Create virtual environment
python3.11 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Create project structure
mkdir -p app/{api/routes,services,ml/models,database,tasks}
touch app/__init__.py app/main.py app/grpc_server.py
touch app/api/__init__.py app/api/routes/__init__.py
touch app/services/__init__.py app/ml/__init__.py
touch app/database/__init__.py app/tasks/__init__.py
```

### Step 2: Install Dependencies

**`ai-service/requirements.txt`**

```txt
# FastAPI
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.0
pydantic-settings==2.1.0

# gRPC
grpcio==1.59.3
grpcio-tools==1.59.3
protobuf==4.25.1

# Database
asyncpg==0.29.0
psycopg2-binary==2.9.9
sqlalchemy==2.0.23
alembic==1.12.1

# Redis
redis==5.0.1
hiredis==2.2.3

# ML Libraries
scikit-learn==1.3.2
pandas==2.1.3
numpy==1.26.2
prophet==1.1.5  # Facebook's forecasting library
statsmodels==0.14.0  # Alternative forecasting

# Celery (Background Jobs)
celery==5.3.4
flower==2.0.1  # Celery monitoring

# Utilities
python-dotenv==1.0.0
python-multipart==0.0.6
httpx==0.25.2

# Development
pytest==7.4.3
pytest-asyncio==0.21.1
black==23.11.0
flake8==6.1.0
mypy==1.7.1
```

```bash
pip install -r requirements.txt
```

### Step 3: Configuration

**`ai-service/.env.example`**

```env
# Application
ENVIRONMENT=development
PORT=5000
LOG_LEVEL=INFO

# Database (PostgreSQL)
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/local_store_platform

# Redis
REDIS_URL=redis://localhost:6379/0

# gRPC
GRPC_PORT=5000
GRPC_MAX_WORKERS=10

# Celery
CELERY_BROKER_URL=redis://localhost:6379/1
CELERY_RESULT_BACKEND=redis://localhost:6379/2

# ML Configuration
MODEL_REGISTRY_PATH=./models
MIN_DATA_POINTS=30  # Minimum sales records for ML
RECOMMENDATION_CONFIDENCE_THRESHOLD=0.7
```

**`ai-service/app/main.py`**

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager
import logging
from app.api.routes import recommendations, forecasting, pricing
from app.database.client import get_database_pool

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application lifespan events"""
    # Startup
    logger.info("🚀 Starting AI Service...")
    await get_database_pool()
    logger.info("✅ Database pool initialized")
    
    yield
    
    # Shutdown
    logger.info("🛑 Shutting down AI Service...")


app = FastAPI(
    title="Local Store Platform - AI Service",
    description="Machine learning service for business intelligence",
    version="1.0.0",
    lifespan=lifespan,
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configure properly in production
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(recommendations.router, prefix="/api/recommendations", tags=["Recommendations"])
app.include_router(forecasting.router, prefix="/api/forecasting", tags=["Forecasting"])
app.include_router(pricing.router, prefix="/api/pricing", tags=["Pricing"])


@app.get("/health")
async def health_check():
    """Health check endpoint"""
    return {"status": "healthy", "service": "ai-service"}


@app.get("/")
async def root():
    """Root endpoint"""
    return {
        "service": "Local Store Platform - AI Service",
        "version": "1.0.0",
        "docs": "/docs",
    }


if __name__ == "__main__":
    import uvicorn
    uvicorn.run("app.main:app", host="0.0.0.0", port=5000, reload=True)
```

**`ai-service/app/database/client.py`**

```python
import asyncpg
import os
from typing import Optional

_pool: Optional[asyncpg.Pool] = None


async def get_database_pool() -> asyncpg.Pool:
    """Get or create database connection pool"""
    global _pool
    
    if _pool is None:
        database_url = os.getenv("DATABASE_URL")
        _pool = await asyncpg.create_pool(
            database_url,
            min_size=2,
            max_size=10,
            command_timeout=60,
        )
    
    return _pool


async def close_database_pool():
    """Close database connection pool"""
    global _pool
    
    if _pool is not None:
        await _pool.close()
        _pool = None
```

**`ai-service/app/services/recommendation_service.py`**

```python
import pandas as pd
import numpy as np
from typing import List, Dict, Any
from datetime import datetime, timedelta
import logging
from app.database.client import get_database_pool

logger = logging.getLogger(__name__)


class RecommendationService:
    """
    Business intelligence recommendation engine
    Phase 1: Rule-based recommendations
    Phase 2: ML-based (future)
    """
    
    async def generate_recommendations(
        self,
        tenant_id: str,
        location_id: str,
        date_range_days: int = 7
    ) -> List[Dict[str, Any]]:
        """Generate AI recommendations for a location"""
        
        logger.info(f"Generating recommendations for location {location_id}")
        
        # Fetch sales data from database
        sales_data = await self._fetch_sales_data(tenant_id, location_id, date_range_days)
        
        if len(sales_data) < 10:
            logger.warning(f"Insufficient data for location {location_id}: {len(sales_data)} records")
            return self._default_recommendations()
        
        # Convert to pandas DataFrame
        df = pd.DataFrame(sales_data)
        
        # Generate recommendations
        recommendations = []
        
        # 1. Pricing recommendations (low-performing items)
        pricing_recs = self._analyze_pricing(df)
        recommendations.extend(pricing_recs)
        
        # 2. Inventory recommendations (stock optimization)
        inventory_recs = self._analyze_inventory(df)
        recommendations.extend(inventory_recs)
        
        # 3. Promotion recommendations (demand patterns)
        promo_recs = self._analyze_promotions(df)
        recommendations.extend(promo_recs)
        
        # 4. Cross-selling recommendations
        cross_sell_recs = self._analyze_cross_selling(df)
        recommendations.extend(cross_sell_recs)
        
        # Sort by priority
        recommendations.sort(key=lambda x: self._priority_score(x), reverse=True)
        
        return recommendations[:10]  # Top 10 recommendations
    
    async def _fetch_sales_data(
        self,
        tenant_id: str,
        location_id: str,
        days: int
    ) -> List[Dict]:
        """Fetch sales data from PostgreSQL"""
        pool = await get_database_pool()
        
        start_date = datetime.now() - timedelta(days=days)
        
        query = """
            SELECT
                t.id as transaction_id,
                t.transaction_date,
                t.total_amount,
                ti.menu_item_id,
                ti.quantity,
                ti.unit_price,
                ti.subtotal,
                mi.name as item_name,
                mi.category_id,
                c.name as category_name
            FROM transactions t
            JOIN transaction_items ti ON t.id = ti.transaction_id
            JOIN menu_items mi ON ti.menu_item_id = mi.id
            JOIN categories c ON mi.category_id = c.id
            WHERE t.tenant_id = $1
                AND t.location_id = $2
                AND t.transaction_date >= $3
                AND t.deleted_at IS NULL
            ORDER BY t.transaction_date DESC
        """
        
        async with pool.acquire() as conn:
            rows = await conn.fetch(query, tenant_id, location_id, start_date)
            return [dict(row) for row in rows]
    
    def _analyze_pricing(self, df: pd.DataFrame) -> List[Dict[str, Any]]:
        """Analyze pricing and suggest optimizations"""
        recommendations = []
        
        # Group by item
        item_stats = df.groupby(['menu_item_id', 'item_name']).agg({
            'quantity': 'sum',
            'subtotal': 'sum',
            'unit_price': 'mean'
        }).reset_index()
        
        # Find low-selling items (bottom 20%)
        threshold = item_stats['quantity'].quantile(0.2)
        low_sellers = item_stats[item_stats['quantity'] <= threshold]
        
        for _, item in low_sellers.iterrows():
            recommendations.append({
                'type': 'pricing',
                'priority': 'medium',
                'title': f"Giảm giá {item['item_name']}",
                'description': f"Món này bán chậm (chỉ {int(item['quantity'])} phần trong 7 ngày). "
                               f"Thử giảm giá 10-15% để tăng doanh số.",
                'impact': 'Có thể tăng 20-30% lượng bán',
                'action': {
                    'type': 'adjust_price',
                    'item_id': item['menu_item_id'],
                    'current_price': float(item['unit_price']),
                    'suggested_price': float(item['unit_price'] * 0.85),
                },
            })
        
        return recommendations
    
    def _analyze_inventory(self, df: pd.DataFrame) -> List[Dict[str, Any]]:
        """Analyze inventory and suggest optimizations"""
        recommendations = []
        
        # Daily sales by item
        daily_sales = df.groupby(['menu_item_id', 'item_name', df['transaction_date'].dt.date]).agg({
            'quantity': 'sum'
        }).reset_index()
        
        # Calculate average daily sales
        avg_daily = daily_sales.groupby(['menu_item_id', 'item_name'])['quantity'].mean()
        
        # Find high-volume items (top 20%)
        threshold = avg_daily.quantile(0.8)
        high_volume = avg_daily[avg_daily >= threshold]
        
        for item_id, avg_qty in high_volume.items():
            item_name = item_id[1]  # Tuple: (item_id, item_name)
            recommendations.append({
                'type': 'inventory',
                'priority': 'high',
                'title': f"Tăng tồn kho {item_name}",
                'description': f"Món bán chạy: trung bình {int(avg_qty)} phần/ngày. "
                               f"Đảm bảo nguyên liệu đủ để tránh hết hàng.",
                'impact': 'Tránh mất khách hàng khi hết món',
                'action': {
                    'type': 'increase_stock',
                    'item_id': item_id[0],
                    'current_daily_avg': float(avg_qty),
                    'suggested_stock_level': float(avg_qty * 3),  # 3 days buffer
                },
            })
        
        return recommendations
    
    def _analyze_promotions(self, df: pd.DataFrame) -> List[Dict[str, Any]]:
        """Analyze demand patterns and suggest promotions"""
        recommendations = []
        
        # Hourly sales distribution
        df['hour'] = pd.to_datetime(df['transaction_date']).dt.hour
        hourly_revenue = df.groupby('hour')['subtotal'].sum()
        
        # Find low-traffic hours (bottom 30%)
        threshold = hourly_revenue.quantile(0.3)
        low_hours = hourly_revenue[hourly_revenue <= threshold].index.tolist()
        
        if low_hours:
            # Group consecutive hours
            hour_ranges = self._group_consecutive_hours(low_hours)
            
            for start, end in hour_ranges:
                recommendations.append({
                    'type': 'promotion',
                    'priority': 'medium',
                    'title': f"Khuyến mãi giờ {start}h-{end}h",
                    'description': f"Doanh thu thấp trong khung giờ này. "
                                   f"Chạy chương trình giảm giá 10-20% để thu hút khách.",
                    'impact': 'Tăng 15-25% doanh thu trong giờ thấp điểm',
                    'action': {
                        'type': 'create_promotion',
                        'time_range': f"{start}:00-{end}:00",
                        'discount_percentage': 15,
                    },
                })
        
        return recommendations
    
    def _analyze_cross_selling(self, df: pd.DataFrame) -> List[Dict[str, Any]]:
        """Analyze frequently bought together items"""
        recommendations = []
        
        # Group items by transaction
        transaction_items = df.groupby('transaction_id')['item_name'].apply(list).reset_index()
        
        # Find item pairs (simplified association rules)
        # TODO: Implement proper Apriori algorithm for larger datasets
        
        return recommendations  # Placeholder for Phase 2
    
    def _default_recommendations(self) -> List[Dict[str, Any]]:
        """Default recommendations when data is insufficient"""
        return [
            {
                'type': 'onboarding',
                'priority': 'high',
                'title': "Nhập dữ liệu bán hàng",
                'description': "Hệ thống cần ít nhất 7 ngày dữ liệu để đưa ra gợi ý thông minh. "
                               "Hãy bắt đầu nhập doanh thu hàng ngày.",
                'impact': 'Mở khóa tính năng AI recommendation',
                'action': {
                    'type': 'add_sales_data',
                    'min_records': 30,
                },
            }
        ]
    
    def _group_consecutive_hours(self, hours: List[int]) -> List[tuple]:
        """Group consecutive hours into ranges"""
        if not hours:
            return []
        
        hours = sorted(hours)
        ranges = []
        start = hours[0]
        prev = hours[0]
        
        for hour in hours[1:]:
            if hour != prev + 1:
                ranges.append((start, prev))
                start = hour
            prev = hour
        
        ranges.append((start, prev))
        return ranges
    
    def _priority_score(self, recommendation: Dict) -> int:
        """Calculate priority score for sorting"""
        priority_map = {'high': 3, 'medium': 2, 'low': 1}
        return priority_map.get(recommendation.get('priority', 'low'), 1)
```

### Step 4: gRPC Server

First, create the proto definition (shared between NestJS and Python):

**`proto/ai_service.proto`**

```protobuf
syntax = "proto3";

package ai_service;

// AI Service Definition
service AIService {
  rpc GetRecommendations(RecommendationRequest) returns (RecommendationResponse);
  rpc PredictDemand(DemandPredictionRequest) returns (DemandPredictionResponse);
  rpc AnalyzePricing(PricingAnalysisRequest) returns (PricingAnalysisResponse);
}

// Recommendation Messages
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
  string action_json = 6;  // JSON string of action details
}

message RecommendationResponse {
  repeated Recommendation recommendations = 1;
  int32 data_points_used = 2;
}

// Demand Prediction Messages
message DemandPredictionRequest {
  string location_id = 1;
  string item_id = 2;
  int32 forecast_days = 3;
}

message DemandForecast {
  string date = 1;
  double predicted_quantity = 2;
  double confidence_lower = 3;
  double confidence_upper = 4;
}

message DemandPredictionResponse {
  repeated DemandForecast forecasts = 1;
  string model_used = 2;
}

// Pricing Analysis Messages
message PricingAnalysisRequest {
  string location_id = 1;
  repeated CompetitorPrice competitor_prices = 2;
}

message CompetitorPrice {
  string item_name = 1;
  double price = 2;
  string source = 3;
}

message PricingSuggestion {
  string item_id = 1;
  string item_name = 2;
  double current_price = 3;
  double suggested_price = 4;
  string reasoning = 5;
}

message PricingAnalysisResponse {
  repeated PricingSuggestion suggestions = 1;
}
```

Generate Python code from proto:

```bash
cd ai-service
python -m grpc_tools.protoc \
  -I../proto \
  --python_out=./app \
  --grpc_python_out=./app \
  ../proto/ai_service.proto
```

**`ai-service/app/grpc_server.py`**

```python
import grpc
from concurrent import futures
import logging
import json
from app import ai_service_pb2, ai_service_pb2_grpc
from app.services.recommendation_service import RecommendationService

logger = logging.getLogger(__name__)


class AIServiceServicer(ai_service_pb2_grpc.AIServiceServicer):
    """gRPC service implementation"""
    
    def __init__(self):
        self.recommendation_service = RecommendationService()
    
    async def GetRecommendations(self, request, context):
        """Get AI recommendations for a location"""
        try:
            logger.info(f"GetRecommendations called for location {request.location_id}")
            
            recommendations = await self.recommendation_service.generate_recommendations(
                tenant_id=request.tenant_id,
                location_id=request.location_id,
                date_range_days=request.date_range_days or 7,
            )
            
            # Convert to protobuf messages
            pb_recommendations = []
            for rec in recommendations:
                pb_rec = ai_service_pb2.Recommendation(
                    type=rec['type'],
                    priority=rec['priority'],
                    title=rec['title'],
                    description=rec['description'],
                    impact=rec['impact'],
                    action_json=json.dumps(rec['action']),
                )
                pb_recommendations.append(pb_rec)
            
            return ai_service_pb2.RecommendationResponse(
                recommendations=pb_recommendations,
                data_points_used=len(recommendations),
            )
        
        except Exception as e:
            logger.error(f"Error in GetRecommendations: {e}", exc_info=True)
            context.set_code(grpc.StatusCode.INTERNAL)
            context.set_details(str(e))
            return ai_service_pb2.RecommendationResponse()


async def serve():
    """Start gRPC server"""
    server = grpc.aio.server(futures.ThreadPoolExecutor(max_workers=10))
    ai_service_pb2_grpc.add_AIServiceServicer_to_server(AIServiceServicer(), server)
    
    port = "0.0.0.0:5000"
    server.add_insecure_port(port)
    
    logger.info(f"🚀 gRPC server starting on {port}")
    await server.start()
    await server.wait_for_termination()


if __name__ == "__main__":
    import asyncio
    logging.basicConfig(level=logging.INFO)
    asyncio.run(serve())
```

---

## gRPC Communication Setup

### NestJS gRPC Client

**`backend/src/modules/integrations/grpc-ai.service.ts`**

```typescript
import { Injectable, OnModuleInit, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as grpc from '@grpc/grpc-js';
import * as protoLoader from '@grpc/proto-loader';
import { join } from 'path';

interface Recommendation {
  type: string;
  priority: string;
  title: string;
  description: string;
  impact: string;
  actionJson: string;
}

@Injectable()
export class GrpcAiService implements OnModuleInit {
  private client: any;
  private readonly logger = new Logger(GrpcAiService.name);

  constructor(private configService: ConfigService) {}

  onModuleInit() {
    const PROTO_PATH = join(__dirname, '../../../proto/ai_service.proto');
    
    const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
      keepCase: true,
      longs: String,
      enums: String,
      defaults: true,
      oneofs: true,
    });

    const aiServiceProto = grpc.loadPackageDefinition(packageDefinition).ai_service as any;
    
    const grpcUrl = this.configService.get<string>('GRPC_AI_SERVICE_URL');
    this.client = new aiServiceProto.AIService(
      grpcUrl,
      grpc.credentials.createInsecure(),
    );

    this.logger.log(`✅ gRPC client connected to AI service at ${grpcUrl}`);
  }

  async getRecommendations(
    tenantId: string,
    locationId: string,
    dateRangeDays: number = 7,
  ): Promise<Recommendation[]> {
    return new Promise((resolve, reject) => {
      this.client.GetRecommendations(
        {
          tenant_id: tenantId,
          location_id: locationId,
          date_range_days: dateRangeDays,
        },
        (error: any, response: any) => {
          if (error) {
            this.logger.error('gRPC Error:', error);
            reject(error);
            return;
          }

          const recommendations = response.recommendations.map((rec: any) => ({
            type: rec.type,
            priority: rec.priority,
            title: rec.title,
            description: rec.description,
            impact: rec.impact,
            action: JSON.parse(rec.action_json),
          }));

          resolve(recommendations);
        },
      );
    });
  }
}
```

---

## Docker Development Environment

**`docker-compose.yml`**

```yaml
version: '3.9'

services:
  # PostgreSQL Database
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
      - ./database/schema.sql:/docker-entrypoint-initdb.d/schema.sql
    networks:
      - lsp-network

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: lsp-redis
    command: redis-server --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - lsp-network

  # NestJS API Gateway
  nestjs-api:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    container_name: lsp-nestjs-api
    environment:
      NODE_ENV: development
      DATABASE_HOST: postgres
      REDIS_HOST: redis
      GRPC_AI_SERVICE_URL: python-ai:5000
    ports:
      - "3000:3000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      - postgres
      - redis
    networks:
      - lsp-network
    command: npm run start:dev

  # Python AI Service
  python-ai:
    build:
      context: ./ai-service
      dockerfile: Dockerfile.dev
    container_name: lsp-python-ai
    environment:
      ENVIRONMENT: development
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/local_store_platform
      REDIS_URL: redis://redis:6379/0
    ports:
      - "5000:5000"
      - "8000:8000"  # FastAPI HTTP
    volumes:
      - ./ai-service:/app
    depends_on:
      - postgres
      - redis
    networks:
      - lsp-network
    command: python -m app.grpc_server

  # Celery Worker (Background Jobs)
  celery-worker:
    build:
      context: ./ai-service
      dockerfile: Dockerfile.dev
    container_name: lsp-celery-worker
    environment:
      ENVIRONMENT: development
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/local_store_platform
      CELERY_BROKER_URL: redis://redis:6379/1
      CELERY_RESULT_BACKEND: redis://redis:6379/2
    volumes:
      - ./ai-service:/app
    depends_on:
      - postgres
      - redis
    networks:
      - lsp-network
    command: celery -A app.tasks worker --loglevel=info

networks:
  lsp-network:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
```

**`backend/Dockerfile.dev`**

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "start:dev"]
```

**`ai-service/Dockerfile.dev`**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Generate gRPC code
RUN python -m grpc_tools.protoc \
    -I../proto \
    --python_out=./app \
    --grpc_python_out=./app \
    ../proto/ai_service.proto

EXPOSE 5000 8000

CMD ["python", "-m", "app.grpc_server"]
```

### Running the Development Environment

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Rebuild after code changes
docker-compose up --build
```

---

## Next Steps

1. **Database Migrations**: Create TypeORM migrations for the 23 tables
2. **GraphQL Schema**: Define complete schema with queries, mutations, subscriptions
3. **WebSocket Gateway**: Implement Socket.io rooms and broadcasting
4. **Authentication Guards**: Protect routes with JWT authentication
5. **Unit Tests**: Write tests for services and resolvers
6. **Integration Tests**: Test gRPC communication end-to-end
7. **API Documentation**: Generate Swagger/OpenAPI docs

**Ready to proceed with Sprint 1 implementation!** 🚀
