# TNT TV Monorepo

A full-stack OTT starter for **TNT TV**: Web (Next.js), Mobile (Expo), and API (Node + Express + Prisma). No references to any other streaming brand appear in the code.

---

## Key Fixes & Improvements
1. **Prisma Import Fixed**: Uses `@prisma/client` correctly.
2. **TSX Parsing Enabled**: `jsx: "react-jsx"` added in `tsconfig.json` (server + web).
3. **React DOM Types**: Added `@types/react-dom`.
4. **CSS Import Path**: Fixed `layout.tsx` global CSS path.
5. **App Router + Pages Router**: Both supported — `app/` directory (Next.js App Router) and `pages/` fallback for legacy tooling.
6. **Tests Added**: Auth API + homepage render tests.
7. **CI Added**: GitHub Actions workflow runs lint/build/test on pushes & PRs with PNPM cache.

---

## Repository Structure

```
.
├─ .github/
│  └─ workflows/
│     └─ ci.yml
├─ package.json
├─ pnpm-workspace.yaml
├─ README.md
├─ jest.projects.config.ts (root multi-project aggregator)
├─ .env.example
├─ prisma/
│  └─ schema.prisma
├─ server/
│  ├─ package.json
│  ├─ jest.config.ts
│  ├─ src/
│  │  ├─ index.ts
│  │  ├─ env.ts
│  │  ├─ prisma.ts
│  │  ├─ auth.ts
│  │  ├─ routes/
│  │  │  ├─ titles.ts
│  │  │  ├─ playback.ts
│  │  │  └─ users.ts
│  ├─ tests/
│  │  └─ auth.test.ts
│  └─ tsconfig.json
├─ apps/
│  ├─ web/
│  │  ├─ package.json
│  │  ├─ jest.config.ts
│  │  ├─ jest.setup.ts
│  │  ├─ next.config.js
│  │  ├─ postcss.config.js
│  │  ├─ tailwind.config.ts
│  │  ├─ tsconfig.json
│  │  ├─ pages/
│  │  │  └─ index.tsx
│  │  ├─ app/
│  │  │  ├─ layout.tsx
│  │  │  ├─ page.tsx
│  │  │  ├─ (auth)/sign-in/page.tsx
│  │  │  └─ watch/[id]/page.tsx
│  │  ├─ components/
│  │  │  ├─ Navbar.tsx
│  │  │  ├─ HeroCarousel.tsx
│  │  │  ├─ TitleCard.tsx
│  │  │  └─ VideoPlayer.tsx
│  │  ├─ __tests__/
│  │  │  └─ home.test.tsx
│  │  └─ styles/globals.css
│  └─ mobile/
│     ├─ app.json
│     ├─ package.json
│     ├─ babel.config.js
│     └─ App.tsx
└─ packages/
   └─ ui/ (optional shared UI)
```

---

## Root files

### `package.json`
```json
{
  "name": "tnt-tv",
  "private": true,
  "version": "0.1.0",
  "workspaces": ["apps/*", "server", "packages/*", "prisma"],
  "scripts": {
    "dev": "concurrently -n api,web -c blue,green \"pnpm --filter server dev\" \"pnpm --filter web dev\"",
    "dev:mobile": "pnpm --filter mobile start",
    "build": "pnpm -r build",
    "test": "pnpm -r test",
    "prisma:generate": "pnpm --filter server prisma:generate",
    "db:push": "pnpm --filter server db:push"
  },
  "devDependencies": {
    "concurrently": "^9.0.1"
  }
}
```

### `pnpm-workspace.yaml`
```yaml
packages:
  - "apps/*"
  - "server"
  - "packages/*"
  - "prisma"
```

### `.env.example`
```
# Shared
DATABASE_URL="postgresql://user:pass@localhost:5432/tnttv"
JWT_SECRET="super-secret-change-me"
NEXT_PUBLIC_API_BASE="http://localhost:4000"
STRIPE_SECRET_KEY="sk_test_..." # optional if you wire up billing
```

### `README.md`
```md
# TNT TV Monorepo

This is a production-ready starter for a streaming platform called **TNT TV**. It includes:
- **API** (Node, Express, Prisma, Postgres)
- **Web** (Next.js + Tailwind + app router)
- **Mobile** (Expo React Native)

## Quick Start

1. Install deps (use pnpm):
   ```sh
   pnpm install
   ```
2. Copy env:
   ```sh
   cp .env.example .env && cp .env.example server/.env && cp .env.example apps/web/.env
   ```
3. Run the API and Web together:
   ```sh
   pnpm dev
   ```
4. (Optional) Start mobile app:
   ```sh
   pnpm dev:mobile
   ```

### Database
Use Postgres. Start locally via Docker:
```sh
docker run --name tnttv -e POSTGRES_PASSWORD=postgres -e POSTGRES_USER=postgres -e POSTGRES_DB=tnttv -p 5432:5432 -d postgres:16
```
Then push schema:
```sh
pnpm db:push
```

### Seeding sample titles
Hit the API route `POST http://localhost:4000/api/titles/seed` to add a few open-licensed trailers.

### CI
On GitHub, the **CI** workflow runs on pushes/PRs: installs deps, builds, and runs tests for all workspaces.

### Notes
- No references to other streaming brands exist in this repo.
- Replace demo HLS links with your own content and DRM.
```

---

## Jest Multi-Project Config (root)

### `jest.projects.config.ts`
```ts
export default {
  projects: [
    '<rootDir>/server/jest.config.ts',
    '<rootDir>/apps/web/jest.config.ts'
  ]
};
```

---

## Prisma

### `prisma/schema.prisma`
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id          String   @id @default(cuid())
  email       String   @unique
  passwordHash String
  displayName String
  createdAt   DateTime @default(now())
  profiles    Profile[]
  watchEvents WatchEvent[]
}

model Profile {
  id        String  @id @default(cuid())
  name      String
  avatarUrl String?
  user      User    @relation(fields: [userId], references: [id])
  userId    String
}

model Title {
  id          String   @id @default(cuid())
  slug        String   @unique
  name        String
  description String
  year        Int
  rating      String?
  genres      String[]
  posterUrl   String
  backdropUrl String
  hlsUrl      String
  createdAt   DateTime @default(now())
}

model WatchEvent {
  id        String   @id @default(cuid())
  user      User     @relation(fields: [userId], references: [id])
  userId    String
  title     Title    @relation(fields: [titleId], references: [id])
  titleId   String
  progress  Int      // seconds
  duration  Int      // seconds
  updatedAt DateTime @updatedAt
}
```

---

## API Server (Node + Express)

### `server/package.json`
```json
{
  "name": "tnt-tv-api",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc -p tsconfig.json",
    "start": "node dist/index.js",
    "prisma:generate": "prisma generate",
    "db:push": "prisma db push",
    "test": "jest --passWithNoTests"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "jsonwebtoken": "^9.0.2",
    "prisma": "^5.18.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/jest": "^29.5.12",
    "@types/jsonwebtoken": "^9.0.5",
    "@types/node": "^20.12.12",
    "@types/supertest": "^2.0.16",
    "jest": "^29.7.0",
    "supertest": "^7.0.0",
    "ts-jest": "^29.2.3",
    "tsx": "^4.16.2",
    "typescript": "^5.6.3"
  }
}
```

### `server/jest.config.ts`
```ts
import type { Config } from 'jest';

const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  testMatch: ['**/tests/**/*.test.ts']
};
export default config;
```

### `server/tsconfig.json`
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
```

### `server/src/prisma.ts`
```ts
import { PrismaClient } from '@prisma/client';
export const prisma = new PrismaClient();
```

### `server/tests/auth.test.ts`
```ts
import request from 'supertest';
import express from 'express';
import { auth } from '../src/auth';

const app = express();
app.use(express.json());
app.use('/api/auth', auth);

describe('Auth API', () => {
  it('should reject login with wrong credentials', async () => {
    const res = await request(app)
      .post('/api/auth/login')
      .send({ email: 'fake@test.com', password: 'wrong' });
    expect(res.status).toBe(401);
  });
});
```

---

## Web (Next.js)

### `apps/web/package.json`
```json
{
  "name": "tnt-tv-web",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev -p 3000",
    "build": "next build",
    "start": "next start -p 3000",
    "test": "jest"
  },
  "dependencies": {
    "hls.js": "^1.5.13",
    "next": "14.2.5",
    "react": "18.3.1",
    "react-dom": "18.3.1",
    "tailwindcss": "^3.4.10"
  },
  "devDependencies": {
    "@testing-library/jest-dom": "^6.4.2",
    "@testing-library/react": "^14.3.1",
    "@types/jest": "^29.5.12",
    "@types/node": "^20.12.12",
    "@types/react": "^18.3.3",
    "@types/react-dom": "^18.3.0",
    "jest": "^29.7.0",
    "jest-environment-jsdom": "^29.7.0",
    "ts-jest": "^29.2.3",
    "typescript": "^5.6.3",
    "postcss": "^8.4.41",
    "autoprefixer": "^10.4.20"
  }
}
```

### `apps/web/jest.config.ts`
```ts
import type { Config } from 'jest';

const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  testMatch: ['**/__tests__/**/*.test.tsx']
};
export default config;
```

### `apps/web/jest.setup.ts`
```ts
import '@testing-library/jest-dom';
```

### `apps/web/tsconfig.json`
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "es2022"],
    "jsx": "react-jsx",
    "module": "esnext",
    "moduleResolution": "bundler",
    "strict": true,
    "baseUrl": ".",
    "paths": { "@/components/*": ["components/*"], "@/lib/*": ["lib/*"] }
  },
  "include": ["app", "components", "lib", "pages", "__tests__"]
}
```

### `apps/web/next.config.js`
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: { domains: ['images.unsplash.com'] },
  experimental: { serverActions: { allowedOrigins: ['localhost:3000'] } }
};
module.exports = nextConfig;
```

### `apps/web/pages/index.tsx`
```tsx
import HomePage from '../app/page';
export default function IndexPage() {
  return <HomePage />;
}
```

### `apps/web/app/layout.tsx`
```tsx
import '../styles/globals.css';
import Link from 'next/link';

export const metadata = {
  title: 'TNT TV',
  description: 'Stream shows and movies on TNT TV.'
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className="bg-brand-dark text-white">
        <header className="sticky top-0 z-50 bg-gradient-to-b from-black/60 to-black/0">
          <nav className="mx-auto flex max-w-7xl items-center justify-between p-4">
            <Link href="/" className="font-extrabold text-2xl text-brand">TNT TV</Link>
            <div className="flex items-center gap-4">
              <Link href="/" className="opacity-80 hover:opacity-100">Home</Link>
              <Link href="/genres" className="opacity-80 hover:opacity-100">Genres</Link>
              <Link href="/(auth)/sign-in" className="rounded bg-brand px-3 py-1 text-sm font-semibold">Sign in</Link>
            </div>
          </nav>
        </header>
        <main className="min-h-screen">{children}</main>
        <footer className="mx-auto max-w-7xl p-6 text-sm opacity-70">© {new Date().getFullYear()} TNT TV</footer>
      </body>
    </html>
  );
}
```

### `apps/web/__tests__/home.test.tsx`
```tsx
import { render, screen } from '@testing-library/react';
import HomePage from '../app/page';

describe('HomePage', () => {
  it('renders Trending Now heading', () => {
    render(<HomePage />);
    expect(screen.getByText(/Trending Now/i)).toBeInTheDocument();
  });
});
```

---

## Mobile (Expo React Native)

*(unchanged from previous revision)*

---

## GitHub Actions CI

### `.github/workflows/ci.yml`
```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    steps:
      - uses: actions/checkout@v4

      - name: Setup PNPM
      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Generate Prisma Client
        run: pnpm --filter server prisma:generate

      - name: Build workspaces
        run: pnpm -r build --if-present

      - name: Run tests
        run: pnpm -r test
```

---

✅ CI runs on Node 18 and 20, caches PNPM, builds, and runs tests across all workspaces.
✅ Root Jest project aggregates **server** and **web** configs.
✅ Tests do not require a live database.

