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
    "dev:sequential": "pnpm dev:server & pnpm dev:client:wait",
    "dev:server": "pnpm --filter server dev",
    "dev:client": "pnpm --filter client dev",
    "dev:client:wait": "wait-on http://localhost:3001/api/health && pnpm --filter client dev",
    "build": "pnpm build:shared && pnpm build:server && pnpm build:client",
    "build:shared": "pnpm --filter @my-app/shared build",
    "build:server": "pnpm --filter server build", 
    "build:client": "pnpm --filter client build",
    "start": "pnpm --filter server start",
    "clean": "pnpm --filter @my-app/shared clean && pnpm --filter client clean && pnpm --filter server clean"
  },
  "devDependencies": {
    "concurrently": "^8.2.2",
    "wait-on": "^7.2.0"
  }
}
```

**Install concurrently and wait-on for coordinated server startup:**

```bash
pnpm add -D concurrently wait-on -w
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

**Create `server/package.json` with enhanced dev script:**

```json
{
  "name": "server",
  "version": "1.0.0",
  "main": "dist/server.js",
  "scripts": {
    "start": "node dist/server.js",
    "dev": "nodemon src/server.ts",
    "dev:wait-ready": "nodemon src/server.ts --signal SIGTERM",
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
pnpm add express cors dotenv
pnpm add -D nodemon typescript ts-node @types/node @types/express @types/cors
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

**Create `server/src/server.ts` with startup logging:**

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

const server = app.listen(PORT, () => {
  console.log(`🚀 Server ready on port ${PORT}`);
  console.log(`📡 API endpoints available:`);
  console.log(`   - GET  http://localhost:${PORT}/api/health`);
  console.log(`   - GET  http://localhost:${PORT}/api/users`);
  console.log(`   - POST http://localhost:${PORT}/api/users`);
});

// Graceful shutdown handling for wait-on compatibility
process.on('SIGTERM', () => {
  console.log('📡 Server shutting down gracefully...');
  server.close(() => {
    console.log('✅ Server closed');
    process.exit(0);
  });
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

**Update `client/package.json` with wait-on integration:**

```json
{
  "name": "client",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "dev:wait": "wait-on http://localhost:3001/api/health && vite",
    "build": "tsc && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview",
    "clean": "rm -rf dist"
  },
  "dependencies": {
    "@my-app/shared": "workspace:^",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.15",
    "@types/react-dom": "^18.2.7",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "@vitejs/plugin-react": "^4.0.3",
    "eslint": "^8.45.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.3",
    "typescript": "^5.0.2",
    "vite": "^4.4.5",
    "wait-on": "^7.2.0"
  }
}
```

**Install wait-on in client:**

```bash
cd client
pnpm add -D wait-on
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
```

## Step 8: Run with Sequential Startup

### Available Development Commands

From the root directory, you now have multiple ways to run the application:

**Option 1: Sequential startup (Recommended)**
```bash
# Starts server first, waits for it to be ready, then starts client
pnpm dev:sequential
```

**Option 2: Concurrent startup (Original method)**
```bash
# Starts both server and client simultaneously
pnpm dev
```

**Option 3: Manual control**
```bash
# Start server only
pnpm dev:server

# In another terminal, start client with wait-on
pnpm dev:client:wait

# Or start client immediately (without waiting)
pnpm dev:client
```

### Recommended Workflow

**For development with guaranteed server readiness:**

```bash
# Use sequential startup to ensure server is ready before client starts
pnpm dev:sequential
```

This command will:
1. Start the Express server in the background
2. Wait for the server to respond to `http://localhost:3001/api/health`
3. Once server is confirmed ready, start the Vite dev server
4. Open your browser to `http://localhost:3000`

### Test the Complete Setup

1. Run `pnpm dev:sequential` to start with guaranteed order
2. You should see output like:
   ```
   🚀 Server ready on port 3001
   📡 API endpoints available:
      - GET  http://localhost:3001/api/health
      - GET  http://localhost:3001/api/users  
      - POST http://localhost:3001/api/users
   wait-on(7588) waiting for 1 resource: http://localhost:3001/api/health
   wait-on(7588) complete
   Local:   http://localhost:3000/
   Network: http://192.168.1.x:3000/
   ```
3. Visit `http://localhost:3000` (or the port shown in terminal)
4. You should see:
   - Server status information
   - List of users fetched from the API
   - No connection errors or timeouts

## Benefits of wait-on Integration

1. **Reliable Startup Order**: Client never starts before server is ready
2. **Eliminates Connection Errors**: No more "ECONNREFUSED" errors during development
3. **Faster Development**: No manual waiting or browser refreshing needed
4. **Production-like Behavior**: Mimics production startup sequences
5. **Flexible Options**: Choose between sequential or concurrent startup based on needs

## Common Issues & Solutions

### 1. **wait-on Timeout**
**Error**: `wait-on timeout after 60000ms`
**Solution**: 
```bash
# Check if server port is correct
curl http://localhost:3001/api/health

# Verify server is starting properly
pnpm dev:server

# Check for port conflicts
lsof -i :3001
```

### 2. **Port Conflicts**
**Error**: `EADDRINUSE: address already in use`
**Solution**: Change ports in:
- `server/.env` 
- `client/vite.config.ts` proxy target
- `client/.env` VITE_API_URL
- Root `package.json` wait-on URL

### 3. **Workspace Dependencies Not Found** 
**Error**: `Cannot find module '@my-app/shared'`
**Solution**: 
```bash
# 1. Ensure shared package is built first
pnpm --filter @my-app/shared build

# 2. Check that server/package.json includes the dependency
# Should have: "@my-app/shared": "workspace:^" in dependencies

# 3. Reinstall workspace dependencies
pnpm install
```

### 4. **TypeScript Import Errors**
**Error**: `'User' is a type and must be imported using a type-only import`
**Solution**: Use `import type { User } from '@my-app/shared'`

### 5. **Server Not Responding to Health Check**
**Error**: `wait-on` hangs indefinitely
**Solution**:
```bash
# Check if health endpoint exists and responds
curl -v http://localhost:3001/api/health

# Verify server logs show startup messages
# Should see: "🚀 Server ready on port 3001"

# Check server route configuration in server/src/server.ts
# Ensure: app.get('/api/health', ...)
```

### 6. **Background Process Management**
**Issue**: Server keeps running after stopping `pnpm dev:sequential`
**Solution**:
```bash
# Kill processes by port
lsof -ti:3001 | xargs kill -9
lsof -ti:3000 | xargs kill -9

# Or use process names
pkill -f "node.*server"
pkill -f "vite"
```

## Production Considerations

For production deployment, the wait-on dependency is only needed during development. The production build process remains the same:

```bash
# Production build
pnpm build

# Start production server (serves both API and static files)
pnpm start
```

The production server automatically serves the built React files, so no wait-on coordination is needed.