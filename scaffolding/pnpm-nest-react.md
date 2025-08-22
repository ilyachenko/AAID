# NestJS + React Full-Stack Setup Guide

Quick setup for a monorepo with NestJS backend, React frontend, and shared TypeScript types using pnpm workspaces.

## Project Structure
```
my-fullstack-app/
├── package.json          # Root workspace config
├── pnpm-workspace.yaml   # Workspace definition
├── server/               # NestJS backend
├── client/               # React frontend  
└── shared/               # Shared TypeScript types
```

## Setup Commands

### 1. Initialize Project
```bash
mkdir my-fullstack-app && cd my-fullstack-app
git init
pnpm init
```

### 2. Configure Workspace
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
  "private": true,
  "scripts": {
    "dev": "concurrently \"pnpm dev:server\" \"pnpm dev:client:wait\" --kill-others-on-fail --prefix-colors \"blue,green\" --names \"server,client\"",
    "dev:server": "pnpm --filter server start:dev",
    "dev:client": "pnpm --filter client dev",
    "dev:client:wait": "wait-on http://localhost:3001/api/health --timeout 30000 && pnpm dev:client",
    "build": "pnpm build:shared && pnpm build:server && pnpm build:client",
    "build:shared": "pnpm --filter @my-app/shared build",
    "build:server": "pnpm --filter server build", 
    "build:client": "pnpm --filter client build"
  },
  "devDependencies": {
    "concurrently": "^8.2.2",
    "wait-on": "^8.2.0"
  }
}
```

Install dev dependencies:
```bash
pnpm add -D concurrently wait-on -w
```

### 3. Create Shared Types Package
```bash
mkdir shared && cd shared
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
    "clean": "rm -rf dist"
  },
  "devDependencies": {
    "typescript": "^5.2.2"
  }
}
```

Create `shared/tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "declaration": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": ["src/**/*"]
}
```

Create type definitions:
```bash
mkdir -p src/types
```

`shared/src/types/api.ts`:
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
  success: boolean;
}
```

`shared/src/index.ts`:
```typescript
export * from './types/api';
```

Build shared package:
```bash
pnpm add -D typescript
pnpm build
cd ..
```

### 4. Create NestJS Backend
```bash
npx @nestjs/cli new server --package-manager pnpm --skip-git --skip-install
cd server
```

Update `server/package.json` dependencies:
```json
{
  "dependencies": {
    "@my-app/shared": "workspace:^",
    "@nestjs/common": "^10.0.0",
    "@nestjs/core": "^10.0.0",
    "@nestjs/platform-express": "^10.0.0",
    "@nestjs/config": "^3.1.1",
    "@nestjs/swagger": "^7.1.16",
    "reflect-metadata": "^0.1.13",
    "rxjs": "^7.8.1"
  }
}
```

Install additional dependencies:
```bash
pnpm add @nestjs/config @nestjs/swagger
```

Update `server/tsconfig.json` to include shared types:
```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "ES2020",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "paths": {
      "@my-app/shared": ["../shared/src"]
    }
  }
}
```

Create `server/.env`:
```env
NODE_ENV=development
PORT=3001
APP_VERSION=1.0.0
```

Update `server/src/main.ts`:
```typescript
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.enableCors({
    origin: process.env.NODE_ENV === 'production' 
      ? ['https://yourdomain.com'] 
      : ['http://localhost:3000'],
    credentials: true,
  });

  app.setGlobalPrefix('api');

  if (process.env.NODE_ENV === 'development') {
    const config = new DocumentBuilder()
      .setTitle('My NestJS API')
      .setVersion('1.0')
      .build();
    const document = SwaggerModule.createDocument(app, config);
    SwaggerModule.setup('api/docs', app, document);
  }

  const port = process.env.PORT || 3001;
  await app.listen(port);
  console.log(`Server: http://localhost:${port}`);
  console.log(`Docs: http://localhost:${port}/api/docs`);
}
bootstrap();
```

Update `server/src/app.controller.ts`:
```typescript
import { Controller, Get } from '@nestjs/common';
import { ApiTags } from '@nestjs/swagger';
import { AppService } from './app.service';
import type { HealthResponse } from '@my-app/shared';

@ApiTags('health')
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('health')
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

### 5. Create React Frontend
```bash
pnpm create vite client --template react-ts
cd client
pnpm install
pnpm add @my-app/shared --workspace
```

Update `client/vite.config.ts`:
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
      }
    }
  }
})
```

Update `client/src/App.tsx`:
```tsx
import { useState, useEffect } from 'react'
import type { HealthResponse } from '@my-app/shared'

function App() {
  const [health, setHealth] = useState<HealthResponse | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    fetch('/api/health')
      .then(res => res.json())
      .then(setHealth)
      .catch(err => setError(err.message))
      .finally(() => setLoading(false))
  }, [])

  if (loading) return <div>Loading...</div>
  if (error) return <div>Error: {error}</div>

  return (
    <div>
      <h1>Full-Stack App</h1>
      {health && (
        <div>
          <h2>Server Status</h2>
          <p>Message: {health.message}</p>
          <p>Version: {health.version}</p>
          <p>Uptime: {Math.round(health.uptime / 1000)}s</p>
        </div>
      )}
      <p>
        <a href="http://localhost:3001/api/docs" target="_blank">
          View API Docs
        </a>
      </p>
    </div>
  )
}

export default App
```

Navigate back to root:
```bash
cd ..
```

## Running the Application

**Always build shared types first:**
```bash
pnpm build:shared
pnpm dev
```

This starts:
- NestJS server: http://localhost:3001
- React client: http://localhost:3000
- API docs: http://localhost:3001/api/docs

## Key Commands
```bash
# Development
pnpm dev                 # Start both servers
pnpm build:shared       # Build shared types (required first)
pnpm build              # Build everything

# Individual services
pnpm dev:server         # Server only
pnpm dev:client         # Client only

# Production
pnpm start              # Production server
```

## Features
- ✅ Monorepo with pnpm workspaces
- ✅ Shared TypeScript types
- ✅ Hot reload in development
- ✅ Swagger API documentation
- ✅ CORS configured
- ✅ Health check endpoint
- ✅ Proper startup order (server waits for health check)