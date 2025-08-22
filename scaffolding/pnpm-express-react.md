# Full-Stack Express + React (Vite) Setup Guide with pnpm and Colored Output

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

Update root `package.json` with enhanced colored scripts:

```json
{
  "name": "my-fullstack-app",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "concurrently --prefix-colors \"blue,green\" --names \"server,client\" \"pnpm dev:server\" \"pnpm dev:client\"",
    "dev:colorful": "concurrently --prefix-colors \"cyan,magenta\" --names \"🚀server,⚛️client\" \"pnpm dev:server\" \"pnpm dev:client\"",
    "dev:rainbow": "concurrently --prefix-colors \"red,yellow,green,cyan,blue,magenta\" --names \"server,client\" \"pnpm dev:server\" \"pnpm dev:client\"",
    "dev:sequential": "concurrently --prefix-colors \"blue,green\" --names \"server,client\" \"pnpm dev:server\" \"pnpm dev:client:wait\"",
    "dev:server": "pnpm --filter server dev",
    "dev:client": "pnpm --filter client dev",
    "dev:client:wait": "wait-on http://localhost:3001/api/health && pnpm --filter client dev",
    "dev:watch": "concurrently --prefix-colors \"yellow,cyan,magenta\" --names \"shared,server,client\" \"pnpm --filter @my-app/shared build:watch\" \"pnpm dev:server\" \"pnpm dev:client:wait\"",
    "build": "pnpm build:shared && pnpm build:server && pnpm build:client",
    "build:shared": "pnpm --filter @my-app/shared build",
    "build:server": "pnpm --filter server build", 
    "build:client": "pnpm --filter client build",
    "build:parallel": "concurrently --prefix-colors \"yellow,blue,green\" --names \"shared,server,client\" \"pnpm build:shared\" \"pnpm build:server\" \"pnpm build:client\"",
    "start": "pnpm --filter server start",
    "clean": "pnpm --filter @my-app/shared clean && pnpm --filter client clean && pnpm --filter server clean",
    "clean:parallel": "concurrently --prefix-colors \"yellow,blue,green\" --names \"shared,server,client\" \"pnpm --filter @my-app/shared clean\" \"pnpm --filter server clean\" \"pnpm --filter client clean\""
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

**Create `server/src/server.ts` with colorful startup logging:**

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
  console.log(`🎨 Running with colored output via concurrently`);
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

Update `client/src/App.tsx` to test the connection with colorful console logs:

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
        console.log('🔄 Fetching data from server...')
        
        // Fetch health status
        const healthResponse = await fetch('/api/health')
        if (!healthResponse.ok) {
          throw new Error('Failed to fetch health status')
        }
        const healthData: HealthResponse = await healthResponse.json()
        setHealth(healthData)
        console.log('✅ Health data fetched successfully')

        // Fetch users
        const usersResponse = await fetch('/api/users')
        if (!usersResponse.ok) {
          throw new Error('Failed to fetch users')
        }
        const usersData: User[] = await usersResponse.json()
        setUsers(usersData)
        console.log('✅ Users data fetched successfully')
        
      } catch (err) {
        console.error('❌ Error fetching data:', err)
        setError(err instanceof Error ? err.message : 'An error occurred')
      } finally {
        setLoading(false)
      }
    }

    fetchData()
  }, [])

  if (loading) return <div>🔄 Loading...</div>
  if (error) return <div>❌ Error: {error}</div>

  return (
    <div className="App">
      <h1>🎨 Full-Stack App with Colored Output</h1>
      
      {health && (
        <div>
          <h2>🚀 Server Status</h2>
          <p>Message: {health.message}</p>
          <p>Version: {health.version}</p>
          <p>Timestamp: {new Date(health.timestamp).toLocaleString()}</p>
        </div>
      )}

      <h2>👥 Users from API:</h2>
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

# Or use parallel build with colors
pnpm build:parallel
```

## Step 8: Run with Colorful Output

### Available Development Commands with Colors

From the root directory, you now have multiple colorful ways to run the application:

**Option 1: Basic colors (Blue server, Green client)**
```bash
pnpm dev
```
This shows output like:
- `[server] 🚀 Server ready on port 3001` (in blue)
- `[client] Local: http://localhost:3000/` (in green)

**Option 2: Enhanced colorful version**
```bash
pnpm dev:colorful
```
This shows output like:
- `[🚀server] 🚀 Server ready on port 3001` (in cyan)
- `[⚛️client] Local: http://localhost:3000/` (in magenta)

**Option 3: Sequential startup with colors**
```bash
pnpm dev:sequential
```
This ensures server starts first, then client, with colored output.

**Option 4: Watch mode with shared types rebuilding**
```bash
pnpm dev:watch
```
This runs three processes with different colors:
- `[shared]` - Rebuilds shared types on changes (in yellow)
- `[server]` - Server with hot reload (in cyan)  
- `[client]` - Client that waits for server (in magenta)

**Option 5: Manual control**
```bash
# Start server only
pnpm dev:server

# In another terminal, start client with wait-on
pnpm dev:client:wait

# Or start client immediately (without waiting)
pnpm dev:client
```

### Available Color Options

You can customize the colors in `package.json` scripts using these color names:
- `red`
- `green` 
- `yellow`
- `blue`
- `magenta`
- `cyan`
- `white`
- `gray`
- `redBright`
- `greenBright`
- `yellowBright`
- `blueBright`
- `magentaBright`
- `cyanBright`
- `whiteBright`

### Custom Color Combinations

Here are some example color combinations you can use:

```json
{
  "scripts": {
    "dev:ocean": "concurrently --prefix-colors \"blue,cyan\" --names \"server,client\"",
    "dev:forest": "concurrently --prefix-colors \"green,yellowBright\" --names \"server,client\"", 
    "dev:sunset": "concurrently --prefix-colors \"red,yellow\" --names \"server,client\"",
    "dev:neon": "concurrently --prefix-colors \"magentaBright,cyanBright\" --names \"server,client\"",
    "dev:minimal": "concurrently --prefix-colors \"gray,white\" --names \"server,client\""
  }
}
```

### Recommended Workflow

**For development with colorful, reliable startup:**

```bash
# Use colorful sequential startup to ensure server is ready before client starts
pnpm dev:sequential
```

**For active development with live type updates:**

```bash
# Use watch mode to automatically rebuild shared types
pnpm dev:watch
```

This command will:
1. Start TypeScript compiler for shared types in watch mode (yellow)
2. Start the Express server in the background (cyan)
3. Wait for the server to respond to `http://localhost:3001/api/health`
4. Once server is confirmed ready, start the Vite dev server (magenta)
5. Open your browser to `http://localhost:3000`

### Test the Complete Setup

1. Run `pnpm dev:colorful` to start with enhanced colored output
2. You should see beautifully colored output like:
   ```
   [🚀server] 🚀 Server ready on port 3001
   [🚀server] 📡 API endpoints available:
   [🚀server]    - GET  http://localhost:3001/api/health
   [🚀server]    - GET  http://localhost:3001/api/users  
   [🚀server]    - POST http://localhost:3001/api/users
   [🚀server] 🎨 Running with colored output via concurrently
   [⚛️client] Local:   http://localhost:3000/
   [⚛️client] Network: http://192.168.1.x:3000/
   ```
3. Visit `http://localhost:3000` (or the port shown in terminal)
4. You should see:
   - Server status information
   - List of users fetched from the API
   - Colored console logs in browser dev tools
   - No connection errors or timeouts

## Benefits of Colored Output Integration

1. **Visual Clarity**: Easily distinguish between server and client logs
2. **Quick Debugging**: Instantly identify which service is producing output
3. **Professional Development**: Makes terminal output look organized and modern
4. **Team Collaboration**: Easier to share screenshots and debug together
5. **Process Identification**: Colored prefixes make it clear which process is running
6. **Enhanced Productivity**: Less mental overhead parsing mixed output

## Advanced Concurrently Options

You can further customize your colored output with these options:

```json
{
  "scripts": {
    "dev:advanced": "concurrently --prefix-colors \"blue,green\" --names \"server,client\" --kill-others-on-fail --restart-tries 3 \"pnpm dev:server\" \"pnpm dev:client\"",
    "dev:timestamps": "concurrently --prefix-colors \"blue,green\" --names \"server,client\" --timestamp-format \"HH:mm:ss\" \"pnpm dev:server\" \"pnpm dev:client\"",
    "dev:no-prefix": "concurrently --prefix none --prefix-colors \"blue,green\" \"pnpm dev:server\" \"pnpm dev:client\""
  }
}
```

Options explained:
- `--kill-others-on-fail`: Kills all processes if one fails
- `--restart-tries 3`: Attempts to restart failed processes up to 3 times
- `--timestamp-format`: Adds timestamps to each line
- `--prefix none`: Removes the `[name]` prefix but keeps colors

## Common Issues & Solutions

### 1. **Colors not showing**
**Issue**: Terminal doesn't show colors
**Solution**: 
```bash
# Check if your terminal supports colors
echo $TERM

# Force color support
export FORCE_COLOR=1
pnpm dev:colorful

# Or use a different terminal (VS Code integrated terminal, iTerm2, etc.)
```

### 2. **Mixed up process names**
**Issue**: Server and client outputs are confusing
**Solution**: Update the names in your script:
```json
{
  "dev": "concurrently --prefix-colors \"blue,green\" --names \"🚀API,⚛️UI\" \"pnpm dev:server\" \"pnpm dev:client\""
}
```

### 3. **Too many colors**
**Issue**: Output is overwhelming with many processes
**Solution**: Use a more subtle color scheme:
```json
{
  "dev:subtle": "concurrently --prefix-colors \"gray,white\" --names \"server,client\" \"pnpm dev:server\" \"pnpm dev:client\""
}
```

### 4. **Concurrently hanging**
**Issue**: Processes don't start or hang
**Solution**:
```bash
# Kill any hanging processes
pkill -f concurrently
pkill -f "pnpm.*dev"

# Clear terminal and restart
clear
pnpm dev:colorful
```

### 5. **Colors in CI/CD**
**Issue**: Colors break in CI environments
**Solution**: Use conditional coloring:
```json
{
  "dev": "concurrently --prefix-colors \"auto\" --names \"server,client\" \"pnpm dev:server\" \"pnpm dev:client\""
}
```

## Production Considerations

For production deployment, the colored output is only used during development. The production build process remains the same:

```bash
# Production build (can use colors for better visibility)
pnpm build:parallel

# Start production server (no colors needed)
pnpm start
```

The production server automatically serves the built React files, so no concurrent processes or colors are needed in production.