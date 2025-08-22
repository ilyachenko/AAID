# Full-Stack Express + React + TypeScript Setup (pnpm workspace)

## Project Structure
```
my-fullstack-app/
├── package.json          # Root workspace
├── pnpm-workspace.yaml   # Workspace config
├── server/               # Express backend
├── shared/               # Shared TypeScript types
└── client/               # React frontend
```

## Quick Setup Commands

```bash
# 1. Initialize project
mkdir my-fullstack-app && cd my-fullstack-app
git init && pnpm init

# 2. Create workspace config
echo 'packages:\n  - "client"\n  - "server"\n  - "shared"' > pnpm-workspace.yaml

# 3. Install dev tools
pnpm add -D concurrently wait-on -w

# 4. Create directories
mkdir shared server
```

## Root package.json scripts
```json
{
  "scripts": {
    "dev": "concurrently --prefix-colors \"blue,green\" --names \"server,client\" \"pnpm dev:server\" \"pnpm dev:client\"",
    "dev:sequential": "concurrently --prefix-colors \"blue,green\" --names \"server,client\" \"pnpm dev:server\" \"pnpm dev:client:wait\"",
    "dev:server": "pnpm --filter server dev",
    "dev:client": "pnpm --filter client dev",
    "dev:client:wait": "wait-on http://localhost:3001/api/health && pnpm --filter client dev",
    "build": "pnpm build:shared && pnpm build:server && pnpm build:client",
    "build:shared": "pnpm --filter @my-app/shared build",
    "build:server": "pnpm --filter server build",
    "build:client": "pnpm --filter client build"
  }
}
```

## Shared Types Package

**shared/package.json:**
```json
{
  "name": "@my-app/shared",
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

**shared/src/index.ts:**
```typescript
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
```

## Server Setup

**server/package.json:**
```json
{
  "name": "server",
  "scripts": {
    "start": "node dist/server.js",
    "dev": "nodemon src/server.ts",
    "build": "tsc"
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

**server/src/server.ts:**
```typescript
import express from 'express';
import cors from 'cors';
import { HealthResponse } from '@my-app/shared';

const app = express();
const PORT = process.env.PORT || 3001;

app.use(cors());
app.use(express.json());

app.get('/api/health', (req, res) => {
  res.json({
    message: 'Server is running!',
    timestamp: new Date(),
    version: '1.0.0'
  });
});

app.listen(PORT, () => {
  console.log(`🚀 Server ready on port ${PORT}`);
});
```

## Client Setup

```bash
# From project root
pnpm create vite client --template react-ts
cd client && pnpm install
pnpm add @my-app/shared --workspace
pnpm add -D wait-on
cd ..
```

**client/vite.config.ts:**
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

**client/src/App.tsx:**
```tsx
import { useState, useEffect } from 'react'
import type { HealthResponse } from '@my-app/shared'

function App() {
  const [health, setHealth] = useState<HealthResponse | null>(null)

  useEffect(() => {
    fetch('/api/health').then(r => r.json()).then(setHealth)
  }, [])

  return (
    <div>
      <h1>Full-Stack App</h1>
      {health && (
        <div>
          <p>Server: {health.message}</p>
          <p>Version: {health.version}</p>
          <p>Timestamp: {new Date(health.timestamp).toLocaleString()}</p>
        </div>
      )}
    </div>
  )
}

export default App
```

## Build & Run Commands

```bash
# Install dependencies
cd shared && pnpm add -D typescript && cd ..
cd server && pnpm add express cors dotenv && pnpm add -D nodemon typescript ts-node @types/node @types/express @types/cors && cd ..

# Build (required order)
pnpm --filter @my-app/shared build
pnpm build

# Development
pnpm dev:sequential  # Recommended: server first, then client
pnpm dev            # Concurrent startup with colors

# Production
pnpm build && pnpm --filter server start
```

## Key Points for LLMs

1. **Build order matters**: Always build `shared` package first
2. **Use workspace dependencies**: `"@my-app/shared": "workspace:^"`
2. **Type-only imports**: `import type { HealthResponse } from '@my-app/shared'`
3. **Port configuration**: Server on 3001, client on 3000 with proxy
4. **Sequential startup**: Use `wait-on` to ensure server is ready before client
5. **Colored output**: Use `concurrently --prefix-colors` for better visibility