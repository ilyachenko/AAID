# Full-Stack Express + React (Vite) Setup Guide with pnpm

## Project Structure

```
my-fullstack-app/
├── .gitignore            # Root gitignore
├── package.json          # Root package.json for workspace
├── pnpm-workspace.yaml   # pnpm workspace configuration
├── server/               # Express.js TypeScript backend
│   ├── .gitignore        # Server-specific gitignore
│   ├── package.json
│   ├── tsconfig.json
│   ├── .env              # Environment variables
│   ├── src/
│   │   └── server.ts
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

# Build outputs
dist/

# Environment variables
.env
.env.local

# Logs
*.log
.pnpm-debug.log*

# TypeScript
*.tsbuildinfo

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
    "dev:server": "pnpm --filter server dev",
    "dev:client": "pnpm --filter client dev",
    "build": "pnpm build:shared && pnpm build:server && pnpm build:client",
    "build:shared": "pnpm --filter @my-app/shared build",
    "build:server": "pnpm --filter server build", 
    "build:client": "pnpm --filter client build",
    "start": "pnpm --filter server start",
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
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Create the shared types structure:

```bash
mkdir -p src/types
```

### Shared Types Organization

The shared package organizes types into two categories:

- **`api.ts`** - Business Logic & Domain Types: Contains types specific to your application's business domain (User entities, API responses, domain-specific operations)
- **`common.ts`** - Reusable Infrastructure Types: Contains generic, reusable types for common patterns (pagination, error handling, HTTP utilities)

This separation allows you to extend `api.ts` with new business entities while reusing `common.ts` patterns across different features.

Create `shared/src/types/api.ts`:

```typescript
export interface User {
  id: number;
  name: string;
  email?: string;
  createdAt?: Date;
}

export interface HealthResponse {
  message: string;
  timestamp: Date;
  version: string;
}

export interface ApiResponse<T> {
  data?: T;
  message?: string;
  error?: string;
  success: boolean;
}

export interface CreateUserRequest {
  name: string;
  email: string;
}

export interface UpdateUserRequest {
  name?: string;
  email?: string;
}
```

Create `shared/src/types/common.ts`:

```typescript
export interface PaginationParams {
  page: number;
  limit: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
}

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

export type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';

export interface ApiError {
  code: string;
  message: string;
  details?: any;
}
```

Create `shared/src/index.ts`:

```typescript
// API Types
export * from './types/api';
export * from './types/common';

// You can add utility functions here too
export const isValidEmail = (email: string): boolean => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

export const formatDate = (date: Date): string => {
  return date.toISOString().split('T')[0];
};
```

### Usage Examples

```typescript
// Import business types (api.ts)
import type { User, CreateUserRequest } from '@my-app/shared'

// Import infrastructure types (common.ts)  
import type { PaginatedResponse, ApiError } from '@my-app/shared'

// Combined usage
const getUsersPaginated = (): PaginatedResponse<User> => {
  return {
    data: users,
    pagination: { page: 1, limit: 10, total: 100, totalPages: 10 }
  }
}

// Using utility functions
import { isValidEmail, formatDate } from '@my-app/shared'
```

**Build the shared package (REQUIRED before proceeding):**

```bash
pnpm build
cd ..
```

## Step 5: Set Up Express.js Backend

**⚠️ Important**: Choose an available port (avoid common ports like 3000, 5000, 8000).

Create server directory:

```bash
mkdir server
cd server
```

Create `server/.gitignore`:

```gitignore
# Build output
dist/

# Dependencies
node_modules/

# Environment variables
.env

# TypeScript
*.tsbuildinfo
```

Create `server/package.json`:

```json
{
  "name": "server",
  "version": "1.0.0",
  "main": "dist/server.js",
  "scripts": {
    "start": "node dist/server.js",
    "dev": "nodemon src/server.ts", 
    "build": "tsc",
    "build:watch": "tsc --watch",
    "clean": "rm -rf dist"
  },
  "dependencies": {
    "@my-app/shared": "workspace:^",
    "express": "^4.18.2",
    "cors": "^2.8.5", 
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "typescript": "^5.2.2",
    "ts-node": "^10.9.1", 
    "@types/node": "^20.8.0",
    "@types/express": "^4.17.18",
    "@types/cors": "^2.8.14"
  }
}
```

Install server dependencies:

```bash
cd server
pnpm add express cors dotenv
pnpm add -D nodemon typescript ts-node @types/node @types/express @types/cors
cd ..
```

**⚠️ Note**: The shared package dependency is already included in package.json above. If you need to add it manually:

```bash 
cd server
pnpm add @my-app/shared --workspace
cd ..
```

Create `server/tsconfig.json`:

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
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Create the server source directory and main file:

```bash
mkdir src
```

Create `server/src/server.ts`:

```typescript
import express, { Request, Response } from 'express';
import cors from 'cors';
import path from 'path';
import dotenv from 'dotenv';
import { User, HealthResponse, ApiResponse } from '@my-app/shared';

dotenv.config();

const app = express();
const PORT = process.env.PORT || 3001; // Use port 3001 to avoid conflicts

// Middleware
app.use(cors());
app.use(express.json());

// API Routes
app.get('/api/health', (req: Request, res: Response<HealthResponse>) => {
  res.json({ 
    message: 'Server is running!',
    timestamp: new Date(),
    version: '1.0.0'
  });
});

app.get('/api/users', (req: Request, res: Response<User[]>) => {
  const users: User[] = [
    { 
      id: 1, 
      name: 'John Doe',
      email: 'john@example.com',
      createdAt: new Date()
    },
    { 
      id: 2, 
      name: 'Jane Smith',
      email: 'jane@example.com',
      createdAt: new Date()
    }
  ];
  res.json(users);
});

app.post('/api/users', (req: Request, res: Response<ApiResponse<User>>) => {
  // In a real app, you'd validate and save to database
  const newUser: User = {
    id: Date.now(),
    name: req.body.name,
    email: req.body.email,
    createdAt: new Date()
  };

  res.status(201).json({
    success: true,
    message: 'User created successfully',
    data: newUser
  });
});

// Serve static files from React build (for production)
if (process.env.NODE_ENV === 'production') {
  app.use(express.static(path.join(__dirname, '../../client/dist')));
  
  app.get('*', (req: Request, res: Response) => {
    res.sendFile(path.join(__dirname, '../../client/dist/index.html'));
  });
}

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Create `server/.env`:

```env
NODE_ENV=development
PORT=3001
```

Navigate back to root:

```bash
cd ..
```

## Step 6: Set Up React Frontend with Vite (TypeScript)

**⚠️ Important**: Run from project root and ensure correct working directory.

Navigate back to root and create client with TypeScript template:

```bash
# Ensure you're in the project root (CRITICAL)
pwd # Should show your project root path

# Create client from root directory
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

**⚠️ TypeScript Import Fix**: Use type-only imports for shared types.

Update `client/src/App.tsx` to test the connection:

```tsx
import { useState, useEffect } from 'react'
import type { User, HealthResponse } from '@my-app/shared' // Note: type-only import
import './App.css'

function App() {
  const [users, setUsers] = useState<User[]>([])
  const [health, setHealth] = useState<HealthResponse | null>(null)
  const [loading, setLoading] = useState<boolean>(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    const fetchData = async () => {
      try {
        // Fetch health status
        const healthResponse = await fetch('/api/health')
        if (!healthResponse.ok) {
          throw new Error('Failed to fetch health status')
        }
        const healthData: HealthResponse = await healthResponse.json()
        setHealth(healthData)

        // Fetch users
        const usersResponse = await fetch('/api/users')
        if (!usersResponse.ok) {
          throw new Error('Failed to fetch users')
        }
        const usersData: User[] = await usersResponse.json()
        setUsers(usersData)
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
  if (error) return <div>Error: {error}</div>

  return (
    <div className="App">
      <h1>Full-Stack App with Shared Types</h1>
      
      {health && (
        <div>
          <h2>Server Status</h2>
          <p>Message: {health.message}</p>
          <p>Version: {health.version}</p>
          <p>Timestamp: {new Date(health.timestamp).toLocaleString()}</p>
        </div>
      )}

      <h2>Users from API:</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            <strong>{user.name}</strong> - {user.email}
            {user.createdAt && (
              <>
                <br />
                <small>Created: {new Date(user.createdAt).toLocaleString()}</small>
              </>
            )}
          </li>
        ))}
      </ul>
    </div>
  )
}

export default App
```

Create `client/.env`:

```env
VITE_API_URL=http://localhost:3001
```

Navigate back to root:

```bash
cd ..
```

## Step 7: Build and Verify Setup

**⚠️ Important**: Build in correct order (shared → server → client).

Build everything:

```bash
# ALWAYS build shared types first (REQUIRED)
pnpm --filter @my-app/shared build

# Verify shared package built successfully
ls shared/dist # Should show compiled files

# Build server
pnpm --filter server build

# Build client  
pnpm --filter client build

# Or use the root build script (builds in correct order)
pnpm build

# Troubleshooting: If server build fails with "Cannot find module '@my-app/shared'"
# 1. Ensure shared package is built: pnpm --filter @my-app/shared build
# 2. Reinstall dependencies: pnpm install
# 3. Check server/package.json includes "@my-app/shared": "workspace:^"
```

## Step 8: Run the Application

From the root directory, you can now run both servers simultaneously:

```bash
# Development mode (runs both client and server)
pnpm dev

# Run only the server
pnpm dev:server

# Run only the client  
pnpm dev:client

# Build for production
pnpm build

# Start production server
pnpm start
```

## Common Issues & Solutions

### 1. **Port Conflicts**
**Error**: `EADDRINUSE: address already in use`
**Solution**: Change ports in:
- `server/.env` 
- `client/vite.config.ts` proxy target
- `client/.env` VITE_API_URL

### 2. **Workspace Dependencies Not Found** 
**Error**: `Cannot find module '@my-app/shared'`
**Solution**: 
```bash
# 1. Ensure shared package is built first
pnpm --filter @my-app/shared build

# 2. Check that server/package.json includes the dependency
# Should have: "@my-app/shared": "workspace:^" in dependencies

# 3. Reinstall workspace dependencies
pnpm install

# 4. If still failing, manually add the dependency:
cd server && pnpm add @my-app/shared --workspace && cd ..
```

### 3. **TypeScript Import Errors**
**Error**: `'User' is a type and must be imported using a type-only import`
**Solution**: Use `import type { User } from '@my-app/shared'`

### 4. **JSX Fragment Errors**
**Error**: `JSX expressions must have one parent element`
**Solution**: Wrap multiple JSX elements in `<>...</>` or `<div>`

### 5. **Nested Directory Creation**
**Issue**: `pnpm create vite` creates files in wrong location
**Solution**: 
```bash
# Always verify you're in project root before creating client
pwd # Should show /path/to/your-project-name
ls # Should show: shared/ server/ package.json pnpm-workspace.yaml

# If in wrong directory, navigate to project root:
cd /path/to/your-project-name
```

### 6. **Command Execution in Scripts**
**Issue**: Commands like `cd server && pnpm install` may fail in some environments
**Solution**: Use separate commands or full paths:
```bash
# Instead of: cd server && pnpm install
# Use:
cd server
pnpm install
cd ..
```

## Project Benefits

- **Monorepo Structure**: Everything in one repository
- **pnpm Workspaces**: Efficient dependency management
- **Shared Types Package**: Type safety across frontend and backend
- **Development Proxy**: Vite proxies API calls to Express server
- **Production Ready**: Express serves built React files
- **Concurrent Development**: All packages can be developed simultaneously
- **Full TypeScript**: Complete type safety from database to UI
- **Proper Git Configuration**: Build outputs and sensitive files are properly ignored
- **Clean Repository**: Only source code and configuration committed to gi