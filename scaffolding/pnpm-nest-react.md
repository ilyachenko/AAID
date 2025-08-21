# Basic Full-Stack NestJS + React (Vite) Setup Guide with pnpm

## Project Structure

```
my-fullstack-app/
├── .gitignore            # Root gitignore
├── package.json          # Root package.json for workspace
├── pnpm-workspace.yaml   # pnpm workspace configuration
├── server/               # NestJS TypeScript backend
│   ├── .gitignore        # Server-specific gitignore
│   ├── package.json
│   ├── tsconfig.json
│   ├── nest-cli.json     # NestJS CLI configuration
│   ├── .env              # Environment variables
│   ├── src/
│   │   ├── main.ts
│   │   ├── app.module.ts
│   │   ├── app.controller.ts
│   │   └── app.service.ts
│   ├── test/             # E2E tests
│   └── dist/             # Built files (ignored by git)
├── shared/               # Shared types package
│   ├── .gitignore        # Shared package gitignore
│   ├── package.json
│   ├── tsconfig.json
│   ├── src/
│   │   ├── index.ts
│   │   └── types/
│   └── dist/             # Built declarations (ignored by git)
└── client/               # React frontend
    ├── .gitignore        # Client gitignore
    ├── package.json
    ├── vite.config.ts
    ├── .env              # Client environment variables
    ├── index.html
    ├── src/
    └── dist/             # Built assets (ignored by git)
```

## Step 1: Initialize the Project

**⚠️ Important**: Run these commands from the project root directory.

Create the project directory and initialize pnpm workspace:

```bash
mkdir my-fullstack-app
cd my-fullstack-app

# Initialize git repository first
git init

# Initialize root package.json
pnpm init
```

## Step 2: Configure Git Ignore (Root Level)

Create root `.gitignore` file:

```gitignore
# Dependencies
node_modules/

# OS
.DS_Store
Thumbs.db
```

## Step 3: Configure pnpm Workspace

Create `pnpm-workspace.yaml`:

```yaml
packages:
  - 'client'
  - 'server' 
  - 'shared'
```

Update root `package.json`:

```json
{
  "name": "my-fullstack-app",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "concurrently \"pnpm dev:server\" \"pnpm dev:client\"",
    "dev:server": "pnpm --filter server start:dev",
    "dev:client": "pnpm --filter client dev",
    "build": "pnpm build:shared && pnpm build:server && pnpm build:client",
    "build:shared": "pnpm --filter @my-app/shared build",
    "build:server": "pnpm --filter server build", 
    "build:client": "pnpm --filter client build",
    "start": "pnpm --filter server start:prod",
    "test": "pnpm --filter server test",
    "test:e2e": "pnpm --filter server test:e2e",
    "clean": "pnpm --filter @my-app/shared clean && pnpm --filter client clean && pnpm --filter server clean"
  },
  "devDependencies": {
    "concurrently": "^8.2.2"
  }
}
```

Install concurrently for running both servers:

```bash
pnpm add -D concurrently -w
```

## Step 4: Set Up Shared Types Package

**⚠️ Build order matters**: Always build shared package first.

Create shared types package:

```bash
mkdir shared
cd shared
```

Create `shared/.gitignore`:

```gitignore
# Build output
dist/

# Dependencies
node_modules/

# TypeScript
*.tsbuildinfo

# Environment variables
.env
```

Create `shared/package.json`:

```json
{
  "name": "@my-app/shared",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts", 
  "scripts": {
    "build": "tsc",
    "build:watch": "tsc --watch",
    "clean": "rm -rf dist"
  },
  "devDependencies": {
    "typescript": "^5.2.2"
  },
  "files": [
    "dist"
  ]
}
```

Install TypeScript for the shared package:

```bash
pnpm add -D typescript
```

Create `shared/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs", 
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Create the shared types structure:

```bash
mkdir -p src/types
```

Create `shared/src/types/api.ts`:

```typescript
export interface HealthResponse {
  message: string;
  timestamp: Date;
  version: string;
  uptime: number;
}

export interface ApiResponse<T> {
  data?: T;
  message?: string;
  error?: string;
  success: boolean;
  statusCode?: number;
}

// NestJS specific types
export interface HttpExceptionResponse {
  statusCode: number;
  message: string | string[];
  error: string;
  timestamp: string;
  path: string;
}
```

Create `shared/src/types/common.ts`:

```typescript
export type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';

export interface ApiError {
  code: string;
  message: string;
  details?: any;
}

// Environment configuration
export interface AppConfig {
  port: number;
  nodeEnv: 'development' | 'production' | 'test';
}
```

Create `shared/src/index.ts`:

```typescript
// API Types
export * from './types/api';
export * from './types/common';

// Utility functions
export const formatDate = (date: Date): string => {
  return date.toISOString().split('T')[0];
};

export const generateId = (): string => {
  return Math.random().toString(36).substring(2) + Date.now().toString(36);
};
```

**Build the shared package (REQUIRED before proceeding):**

```bash
pnpm build
cd ..
```

## Step 5: Set Up NestJS Backend

Create server directory and initialize NestJS project:

```bash
# Create the NestJS app using CLI
npx @nestjs/cli new server --package-manager pnpm --skip-git --skip-install

cd server
```

Update `server/package.json` to include workspace dependency:

```json
{
  "name": "server",
  "version": "1.0.0",
  "description": "",
  "author": "",
  "private": true,
  "license": "UNLICENSED",
  "scripts": {
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "clean": "rm -rf dist"
  },
  "dependencies": {
    "@my-app/shared": "workspace:^",
    "@nestjs/common": "^10.0.0",
    "@nestjs/core": "^10.0.0",
    "@nestjs/platform-express": "^10.0.0",
    "@nestjs/config": "^3.1.1",
    "@nestjs/swagger": "^7.1.16",
    "reflect-metadata": "^0.1.13",
    "rxjs": "^7.8.1"
  },
  "devDependencies": {
    "@nestjs/cli": "^10.0.0",
    "@nestjs/schematics": "^10.0.0",
    "@nestjs/testing": "^10.0.0",
    "@types/express": "^4.17.17",
    "@types/jest": "^29.5.2",
    "@types/node": "^20.3.1",
    "@types/supertest": "^2.0.12",
    "@typescript-eslint/eslint-plugin": "^5.59.11",
    "@typescript-eslint/parser": "^5.59.11",
    "eslint": "^8.42.0",
    "eslint-config-prettier": "^8.8.0",
    "eslint-plugin-prettier": "^4.2.1",
    "jest": "^29.5.0",
    "prettier": "^2.8.8",
    "source-map-support": "^0.5.21",
    "supertest": "^6.3.3",
    "ts-jest": "^29.1.0",
    "ts-loader": "^9.4.3",
    "ts-node": "^10.9.1",
    "tsconfig-paths": "^4.2.0",
    "typescript": "^5.1.3"
  }
}
```

Install additional NestJS dependencies:

```bash
pnpm add @nestjs/config @nestjs/swagger
```

Create `server/.gitignore` (NestJS CLI doesn't generate this automatically):

```gitignore
# Build output
dist/
.nest/

# Dependencies
node_modules/

# Environment variables
.env

# TypeScript
*.tsbuildinfo

# Logs
*.log

# Coverage
coverage/

# IDE
.vscode/
.idea/
```

Update `server/tsconfig.json`:

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2020",
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": false,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "forceConsistentCasingInFileNames": false,
    "noFallthroughCasesInSwitch": false,
    "paths": {
      "@my-app/shared": ["../shared/src"]
    }
  }
}
```

**⚠️ Important Fix**: Update `server/tsconfig.build.json` to prevent build issues:

```json
{
  "extends": "./tsconfig.json",
  "exclude": ["node_modules", "test", "dist", "**/*spec.ts"],
  "compilerOptions": {
    "paths": {}
  }
}
```

Create `server/.env`:

```env
NODE_ENV=development
PORT=3001
APP_NAME=My NestJS App
APP_VERSION=1.0.0
```

Update `server/src/main.ts`:

```typescript
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable CORS
  app.enableCors({
    origin: process.env.NODE_ENV === 'production' 
      ? ['https://yourdomain.com'] // Replace with your production domain
      : ['http://localhost:3000'], // React dev server
    credentials: true,
  });

  // API prefix
  app.setGlobalPrefix('api');

  // Swagger documentation (only in development)
  if (process.env.NODE_ENV === 'development') {
    const config = new DocumentBuilder()
      .setTitle('My NestJS API')
      .setDescription('Basic API with health check')
      .setVersion('1.0')
      .addTag('health')
      .build();
    const document = SwaggerModule.createDocument(app, config);
    SwaggerModule.setup('api/docs', app, document);
  }

  const port = process.env.PORT || 3001;
  await app.listen(port);
  console.log(`Application is running on: http://localhost:${port}`);
  console.log(`Swagger documentation: http://localhost:${port}/api/docs`);
}
bootstrap();
```

Update `server/src/app.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Update `server/src/app.controller.ts`:

```typescript
import { Controller, Get } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';
import { AppService } from './app.service';
import type { HealthResponse } from '@my-app/shared';

@ApiTags('health')
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('health')
  @ApiOperation({ summary: 'Health check endpoint' })
  @ApiResponse({ 
    status: 200, 
    description: 'Server health status',
    type: 'object'
  })
  getHealth(): HealthResponse {
    return this.appService.getHealth();
  }
}
```

Update `server/src/app.service.ts`:

```typescript
import { Injectable } from '@nestjs/common';
import type { HealthResponse } from '@my-app/shared';

@Injectable()
export class AppService {
  private readonly startTime = Date.now();

  getHealth(): HealthResponse {
    return {
      message: 'NestJS server is running!',
      timestamp: new Date(),
      version: process.env.APP_VERSION || '1.0.0',
      uptime: Date.now() - this.startTime,
    };
  }
}
```

Navigate back to root:

```bash
cd ..
```

## Step 6: Set Up React Frontend with Vite (TypeScript)

**⚠️ Critical**: Always run pnpm commands from the project root directory unless explicitly told otherwise.

Navigate back to root and create client with TypeScript template:

```bash
# Ensure you're in the project root (CRITICAL)
pwd # Should show your project root path

# Create client from root directory (IMPORTANT: run this from project root)
pnpm create vite client --template react-ts
cd client
pnpm install

# Add shared package as workspace dependency
pnpm add @my-app/shared --workspace
cd ..
```

Update `client/vite.config.ts` with proxy configuration:

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:3001', // Match server port
        changeOrigin: true,
      }
    }
  }
})
```

Update `client/src/App.tsx` to test the NestJS connection:

```tsx
import { useState, useEffect } from 'react'
import type { HealthResponse } from '@my-app/shared'
import './App.css'

function App() {
  const [health, setHealth] = useState<HealthResponse | null>(null)
  const [loading, setLoading] = useState<boolean>(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('/api/health')
        if (!response.ok) {
          throw new Error('Failed to fetch health status')
        }
        const healthData: HealthResponse = await response.json()
        setHealth(healthData)
      } catch (err) {
        setError(err instanceof Error ? err.message : 'An error occurred')
        console.error('Error:', err)
      } finally {
        setLoading(false)
      }
    }

    fetchData()
  }, [])

  if (loading) return <div>Loading...</div>
  if (error) return <div style={{ color: 'red' }}>Error: {error}</div>

  return (
    <div className="App">
      <h1>Full-Stack NestJS + React App</h1>
      
      {health && (
        <div style={{ 
          background: '#f0f9ff', 
          padding: '1rem', 
          borderRadius: '8px', 
          marginBottom: '2rem' 
        }}>
          <h2>🚀 Server Status</h2>
          <p><strong>Message:</strong> {health.message}</p>
          <p><strong>Version:</strong> {health.version}</p>
          <p><strong>Uptime:</strong> {Math.round(health.uptime / 1000)}s</p>
          <p><strong>Timestamp:</strong> {new Date(health.timestamp).toLocaleString()}</p>
        </div>
      )}

      <div style={{ 
        marginTop: '2rem', 
        padding: '1rem', 
        background: '#fffbeb', 
        borderRadius: '8px' 
      }}>
        <h3>🔗 API Documentation</h3>
        <p>
          Visit <a 
            href="http://localhost:3001/api/docs" 
            target="_blank" 
            rel="noopener noreferrer"
            style={{ color: '#3b82f6' }}
          >
            http://localhost:3001/api/docs
          </a> to view the interactive Swagger API documentation.
        </p>
      </div>
    </div>
  )
}

export default App
```

## Step 7: Running the Application

**⚠️ Important**: Always build the shared package first before running the application.

From the project root:

```bash
# Build shared types first (REQUIRED)
pnpm build:shared

# Start both server and client in development mode
pnpm dev
```

This will start:
- NestJS server on http://localhost:3001
- React client on http://localhost:3000
- Swagger docs on http://localhost:3001/api/docs

## Useful Commands

```bash
# Build everything
pnpm build

# Build individual packages
pnpm build:shared
pnpm build:server  
pnpm build:client

# Run server only
pnpm dev:server

# Run client only
pnpm dev:client

# Clean all build outputs
pnpm clean

# Run tests
pnpm test

# Production start (server only)
pnpm start
```

## Key Features

✅ **Monorepo Structure**: pnpm workspace with shared types package  
✅ **Type Safety**: Full TypeScript support across frontend and backend  
✅ **Hot Reload**: Development mode with automatic restarts  
✅ **API Documentation**: Swagger UI for interactive API testing  
✅ **CORS Configuration**: Proper cross-origin setup for development  
✅ **Health Check**: Basic endpoint to verify server status  
✅ **Build System**: Optimized build process with proper dependencies

The application now provides a solid foundation for full-stack development with NestJS and React, featuring shared TypeScript types and a basic health check endpoint to verify the connection between frontend and backend.