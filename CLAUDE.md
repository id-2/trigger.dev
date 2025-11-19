# Trigger.dev Repository Guide for AI Assistants

## Repository Overview

**Trigger.dev** is an open-source platform for creating long-running background jobs with no timeouts. This is a **pnpm monorepo** using **Turborepo** for build orchestration.

- **Repository**: https://github.com/triggerdotdev/trigger.dev
- **License**: MIT (Apache Licence 2.0 for distributed code)
- **Node Version**: v20.11.1 (managed via `.nvmrc`)
- **Package Manager**: pnpm 8.15.5 (enforced via `packageManager` field)
- **Current Version**: 3.0.0-beta.46 (V3 is in developer preview)

## Architecture & Technology Stack

### Monorepo Structure

```
trigger.dev/
├── apps/              # Application services
├── packages/          # Core SDK and tooling packages  
├── integrations/      # Third-party service integrations
├── config-packages/   # Shared build/linting configuration
├── references/        # Example projects and testing catalogs
├── docker/            # Docker compose files for local dev
├── docs/              # Documentation
├── tests/             # E2E tests
└── runtime_tests/     # Runtime compatibility tests
```

**Workspace Configuration** (`pnpm-workspace.yaml`):
- `config-packages/*`
- `packages/*`
- `integrations/*`
- `apps/**`
- `references/*`
- `docs`, `perf`, `runtime_tests`

### Technology Stack

**Core Languages & Frameworks:**
- **TypeScript 5.x** - Primary language across all packages
- **Node.js 18+** - Minimum runtime requirement (20.11.1 for development)
- **React 18** - UI framework
- **Remix 2.1.0** - Full-stack web framework for webapp
- **Express** - Server runtime for webapp

**Build Tools:**
- **Turborepo 1.10.3** - Monorepo build orchestration
- **tsup 8.0.1** - TypeScript bundler (using esbuild internally)
- **esbuild 0.19.11** - Fast JavaScript bundler
- **Vite 4.x** - Build tool and dev server
- **Prisma 5.4.1** - Database ORM and migrations

**Databases & Infrastructure:**
- **PostgreSQL 14** - Primary database
- **Redis 7** - Caching and pub/sub
- **Prisma** - ORM with migrations in `packages/database/prisma/`
- **graphile-worker 0.16.6** - Background job processing (with custom patches)

**Observability:**
- **OpenTelemetry (OTLP)** - Distributed tracing and logging
  - `@opentelemetry/api` 1.8.0
  - `@opentelemetry/sdk-node` 0.49.1
  - Custom OTLP importer package
- **Prometheus** (`prom-client`) - Metrics
- **Socket.io 4.7.4** - Real-time communication

**Testing:**
- **Vitest 0.28.4-1.6.0** - Unit testing (primary)
- **Jest 29.x** - Unit testing (some packages)
- **Playwright 1.36.2** - E2E testing

**Styling & UI:**
- **Tailwind CSS 3.4.1** - Utility-first CSS
- **Radix UI** - Accessible component primitives
- **Headless UI** - Accessible UI components
- **Framer Motion** - Animation library
- **CodeMirror 6** - Code editor

**Other Key Dependencies:**
- **Zod 3.22.3** - Schema validation (pinned version across monorepo)
- **Socket.io** - WebSocket communication
- **Docker** - Containerization
- **Kubernetes** - Container orchestration (provider app)

## Applications (`apps/`)

### 1. **webapp** (Main Platform)
**Path**: `/home/user/trigger.dev/apps/webapp`
**Description**: The core Trigger.dev platform - a Remix-based web application

**Key Features:**
- Remix 2.1.0 full-stack application
- Express server runtime
- PostgreSQL database via Prisma
- Real-time features via Socket.io
- Redis for caching and sessions
- GraphQL Worker for background jobs
- Authentication (GitHub OAuth, Magic Link)
- OpenTelemetry instrumentation

**Tech Stack:**
- Remix (SSR framework)
- React 18
- Tailwind CSS + Radix UI
- Prisma ORM
- Express server
- Socket.io for real-time

**Scripts:**
- `dev` - Development server on port 3030
- `build` - Build Remix app and server
- `start` - Production server
- `db:seed` - Seed database
- `db:migrate` - Run migrations

### 2. **coordinator**
**Path**: `/home/user/trigger.dev/apps/coordinator`
**Description**: Coordinates task execution across workers

**Tech Stack:**
- Node.js service
- Socket.io for communication
- Uses `@trigger.dev/core` and `@trigger.dev/core-apps`
- Built with esbuild (ESM format)

**Build Output**: `dist/index.mjs`

### 3. **docker-provider**
**Path**: `/home/user/trigger.dev/apps/docker-provider`
**Description**: Manages task execution in Docker containers

**Tech Stack:**
- Node.js service
- Docker integration via `execa`
- Uses `@trigger.dev/core` and `@trigger.dev/core-apps`

### 4. **kubernetes-provider**
**Path**: `/home/user/trigger.dev/apps/kubernetes-provider`
**Description**: Manages task execution in Kubernetes

**Tech Stack:**
- `@kubernetes/client-node` 0.20.0
- `p-queue` for request queuing
- Uses `@trigger.dev/core-apps`

### 5. **proxy**
**Path**: `/home/user/trigger.dev/apps/proxy`
**Description**: Cloudflare Workers-based proxy service

**Tech Stack:**
- Cloudflare Workers
- Wrangler 3.x for deployment
- AWS SQS client
- Lightweight (no heavy dependencies)

### 6. **yalt**
**Path**: `/home/user/trigger.dev/apps/yalt`
**Description**: Real-time service (yalt.dev client)

## Core Packages (`packages/`)

### Package Relationships

```
┌─────────────────────────────────────────────────────────────┐
│                     @trigger.dev/sdk                         │
│                   (User-facing SDK)                          │
│                  - V2: Job-based API                         │
│                  - V3: Task-based API                        │
└──────────────────────┬──────────────────────────────────────┘
                       │ depends on
                       ↓
┌─────────────────────────────────────────────────────────────┐
│              @trigger.dev/core-backend                       │
│         (Shared backend code for SDK and server)             │
└──────────────────────┬──────────────────────────────────────┘
                       │ depends on
                       ↓
┌─────────────────────────────────────────────────────────────┐
│                   @trigger.dev/core                          │
│              (Core shared code and types)                    │
│         - Schemas, validation, utilities                     │
│         - V3 infrastructure (OTLP, workers)                  │
└─────────────────────────────────────────────────────────────┘
                       ↑
                       │ used by
                       ↓
┌─────────────────────────────────────────────────────────────┐
│                trigger.dev (cli-v3)                          │
│              (V3 Command Line Interface)                     │
│            - triggerdev dev/deploy/login                     │
└─────────────────────────────────────────────────────────────┘
```

### 1. **@trigger.dev/core**
**Path**: `/home/user/trigger.dev/packages/core`
**Description**: Core shared code used across SDK and platform

**Exports:**
- Main: `./dist/index.js`
- V3 specific: `./v3/*`
- OTLP: `./v3/otel`
- Utilities: `./v3/utils/*`
- Dev/Prod runtimes: `./v3/dev`, `./v3/prod`
- Workers: `./v3/workers`
- Schemas: `./v3/schemas`

**Key Features:**
- Event filtering and matching
- Retry logic
- OpenTelemetry integration
- Socket.io client for real-time
- Zod schemas for validation
- SuperJSON for serialization

**Structure:**
```
src/
├── v3/                    # V3-specific code
│   ├── schemas/           # Zod schemas
│   ├── workers/           # Worker runtime
│   ├── otel/              # OpenTelemetry
│   ├── dev/               # Dev runtime
│   ├── prod/              # Prod runtime
│   ├── utils/             # Utilities
│   └── zodSocket.ts       # Typed Socket.io
└── [v2 code]             # Legacy V2 code
```

### 2. **@trigger.dev/core-backend**
**Path**: `/home/user/trigger.dev/packages/core-backend`
**Description**: Backend code shared between SDK and server

**Dependencies:**
- Minimal: Only `@opentelemetry/api`
- Very lightweight, focused on shared backend utilities

### 3. **@trigger.dev/sdk** (trigger-sdk)
**Path**: `/home/user/trigger.dev/packages/trigger-sdk`
**Description**: The main SDK for users to create tasks/jobs

**Exports:**
- V2 API (default): Jobs, triggers, integrations
- V3 API: `@trigger.dev/sdk/v3` - Tasks, runs, schedules

**V2 Structure** (`src/`):
- `job.ts` - Job definitions
- `triggerClient.ts` - Client setup
- `io.ts` - I/O operations
- `triggers/` - Event, scheduled, webhook triggers

**V3 Structure** (`src/v3/`):
- `tasks.ts` - Task definitions
- `runs.ts` - Run management
- `schedules/` - Scheduled tasks
- `retry.ts` - Retry configuration
- `wait.ts` - Wait operations
- `cache.ts` - Caching utilities

**Key Features:**
- Dual V2/V3 API support
- OpenTelemetry auto-instrumentation
- WebSocket connection to platform
- Git integration for version tracking
- MSW (Mock Service Worker) support

### 4. **trigger.dev** (cli-v3)
**Path**: `/home/user/trigger.dev/packages/cli-v3`
**Description**: V3 Command Line Interface

**Binary**: `triggerdev`

**Commands:**
- `login` - Authenticate with Trigger.dev
- `dev` - Run development server
- `deploy` - Deploy to production
- `init` - Initialize new project

**Key Features:**
- Ink (React) for CLI UI
- Built with tsup
- Chokidar for file watching
- esbuild for bundling user code
- Depot integration for Docker builds
- Profile support for multi-environment

**Structure:**
```
src/
├── cli/                  # Command definitions
├── workers/              # Build workers
├── utilities/            # Helper functions
└── Containerfile.prod   # Production container
```

### 5. **@trigger.dev/cli** (V2 CLI)
**Path**: `/home/user/trigger.dev/packages/cli`
**Description**: Legacy V2 Command Line Interface

**Binary**: `trigger-cli`

**Note**: Separate from V3 CLI, supports V2 job-based API

### 6. **@trigger.dev/database**
**Path**: `/home/user/trigger.dev/packages/database`
**Description**: Prisma database schema and migrations

**Database**: PostgreSQL 14
**ORM**: Prisma 5.4.1

**Key Models:**
- `User` - User accounts
- `Organization` - Organizations
- `Project` - Projects (with V3 flag)
- `Job` - V2 Jobs
- `Task` - V3 Tasks
- `Run` - Task/Job runs
- Many more...

**Scripts:**
- `generate` - Generate Prisma client
- `db:migrate:dev` - Create and apply migration
- `db:migrate:deploy` - Apply migrations
- `db:studio` - Prisma Studio UI

**Migrations**: 527+ migration files in `prisma/migrations/`

### 7. **@trigger.dev/core-apps**
**Path**: `/home/user/trigger.dev/packages/core-apps`
**Description**: Backend core code shared across apps (coordinator, providers)

**Dependencies:**
- `@trigger.dev/core`
- `execa` for process execution

### 8. **@trigger.dev/otlp-importer**
**Path**: `/home/user/trigger.dev/packages/otlp-importer`
**Description**: OpenTelemetry OTLP importer

**Features:**
- Protobuf support
- OTLP log/trace importing
- Used by webapp for telemetry ingestion

### 9. **@trigger.dev/yalt**
**Path**: `/home/user/trigger.dev/packages/yalt`
**Description**: yalt.dev client library for real-time communication

**Dependencies:**
- `partysocket` for WebSocket
- `proxy-agent` for proxy support

### 10. **@trigger.dev/integration-kit**
**Path**: `/home/user/trigger.dev/packages/integration-kit`
**Description**: Helpers for creating third-party integrations

**Used By**: All integration packages in `integrations/`

### 11. **@trigger.dev/testing**
**Path**: `/home/user/trigger.dev/packages/testing`
**Description**: Testing utilities for Trigger.dev tasks

**Dependencies:**
- Vitest
- Workspace SDK and core packages

## Framework Integration Packages

**Location**: `/home/user/trigger.dev/packages/`

### Available Integrations:
1. **@trigger.dev/nextjs** - Next.js integration
2. **@trigger.dev/remix** - Remix integration
3. **@trigger.dev/express** - Express.js integration
4. **@trigger.dev/astro** - Astro integration
5. **@trigger.dev/hono** - Hono integration
6. **@trigger.dev/nestjs** - NestJS integration
7. **@trigger.dev/sveltekit** - SvelteKit integration
8. **@trigger.dev/react** - React components

**Pattern**: Each package provides framework-specific utilities for integrating Trigger.dev

## Third-Party Integrations (`integrations/`)

**Location**: `/home/user/trigger.dev/integrations/`

### Available Integrations:
1. **@trigger.dev/openai** - OpenAI API
2. **@trigger.dev/stripe** - Stripe payments
3. **@trigger.dev/github** - GitHub API
4. **@trigger.dev/slack** - Slack API
5. **@trigger.dev/linear** - Linear issues
6. **@trigger.dev/resend** - Resend email
7. **@trigger.dev/sendgrid** - SendGrid email
8. **@trigger.dev/supabase** - Supabase
9. **@trigger.dev/shopify** - Shopify
10. **@trigger.dev/airtable** - Airtable
11. **@trigger.dev/plain** - Plain support
12. **@trigger.dev/replicate** - Replicate AI
13. **@trigger.dev/typeform** - Typeform

**Pattern**: Each integration:
- Extends `@trigger.dev/integration-kit`
- Provides typed SDK methods
- Handles authentication/rate limiting
- Built with tsup
- Exports both ESM and CJS

## V2 vs V3 Architecture

### V2 (Legacy - Job-based)

**Concepts:**
- **Jobs** - Background tasks triggered by events
- **TriggerClient** - Main client for job registration
- **I/O Operations** - `io.runTask()`, `io.backgroundFetch()`, etc.
- **Triggers** - Event, scheduled, webhook triggers
- **Integrations** - Tight coupling with integration packages

**Import**: `@trigger.dev/sdk`

**Example**:
```typescript
import { TriggerClient } from "@trigger.dev/sdk";

const client = new TriggerClient({ id: "my-app" });

client.defineJob({
  id: "my-job",
  name: "My Job",
  version: "1.0.0",
  trigger: eventTrigger({ name: "user.created" }),
  run: async (payload, io, ctx) => {
    await io.runTask("send-email", async () => {
      // Task code
    });
  },
});
```

### V3 (Current - Task-based)

**Concepts:**
- **Tasks** - Standalone async functions, no event system
- **Runs** - Task executions
- **Schedules** - CRON-based scheduling
- **Triggers** - Trigger tasks programmatically
- **No I/O wrapper** - Direct async/await

**Import**: `@trigger.dev/sdk/v3`

**Example**:
```typescript
import { task } from "@trigger.dev/sdk/v3";

export const myTask = task({
  id: "my-task",
  run: async (payload: { userId: string }) => {
    // Direct async code, no io wrapper
    await sendEmail(payload.userId);
    return { success: true };
  },
});
```

**Key Differences:**

| Feature | V2 | V3 |
|---------|----|----|
| **Primary Unit** | Job | Task |
| **Triggers** | Event-driven | Direct invocation |
| **I/O Pattern** | `io.runTask()` wrappers | Direct async/await |
| **Client** | TriggerClient required | No client needed |
| **Deployment** | Platform-hosted | Containerized |
| **Timeouts** | Resume after timeout | No timeouts |
| **Execution** | Orchestrated steps | Long-running processes |

**V3 Architecture** (`packages/core/src/v3/`):
```
v3/
├── schemas/              # API schemas
├── workers/              # Worker runtime code
├── otel/                 # OpenTelemetry setup
├── dev/                  # Dev runtime
├── prod/                 # Production runtime
├── task-catalog/         # Task registry
├── task-context/         # Execution context
├── runtime/              # Runtime detection
├── logger/               # Structured logging
├── clock/                # Time utilities
└── apiClient/            # API client
```

**Migration Status**: V3 is in beta (3.0.0-beta.46), both versions supported

## Development Patterns

### Monorepo Conventions

**Build System**: Turborepo with dependency graph

**Key Scripts** (root `package.json`):
- `pnpm run dev` - Start all apps in parallel
- `pnpm run build` - Build all packages
- `pnpm run build --filter webapp` - Build specific package
- `pnpm run lint` - Lint all packages
- `pnpm run typecheck` - Type check all packages
- `pnpm run test` - Run all tests
- `pnpm run docker` - Start Docker services

**Turbo Pipeline** (`turbo.json`):
```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],  // Build dependencies first
      "outputs": ["dist/**", "public/build/**", "build/**"]
    },
    "dev": { "cache": false },
    "test": {
      "dependsOn": ["^build"]
    }
  }
}
```

### Package Structure Pattern

**Standard Package Layout:**
```
package-name/
├── src/
│   ├── index.ts           # Main entry
│   └── ...                # Source files
├── dist/                  # Build output (gitignored)
│   ├── index.js           # CJS
│   ├── index.mjs          # ESM
│   ├── index.d.ts         # Types (CJS)
│   └── index.d.mts        # Types (ESM)
├── package.json
├── tsconfig.json
├── tsup.config.ts         # Build config (if using tsup)
└── README.md
```

**Dual Package Exports** (ESM + CJS):
```json
{
  "type": "module",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": {
        "types": "./dist/index.d.mts",
        "default": "./dist/index.mjs"
      },
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  }
}
```

### TypeScript Configuration

**Shared Configs**: `config-packages/tsconfig/`

**Common Pattern**:
```json
{
  "extends": "@trigger.dev/tsconfig/base.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  }
}
```

**Strict Mode**: Most packages use strict TypeScript

**Node Types**: `@types/node` version 18

### Build Tools

**Primary Bundler**: `tsup` 8.0.1
- Fast TypeScript bundler
- Based on esbuild
- Dual ESM/CJS output
- Type declaration generation

**Shared tsup Config**: `config-packages/tsup/`

**Typical tsup.config.ts**:
```typescript
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["cjs", "esm"],
  dts: true,
  clean: true,
  sourcemap: true,
});
```

**Alternative Builders**:
- `esbuild` - Coordinator, providers (single bundle)
- `remix build` - Webapp
- `wrangler` - Proxy (Cloudflare Workers)

### Dependency Management

**Workspace Protocol**: `workspace:*` for internal dependencies
```json
{
  "dependencies": {
    "@trigger.dev/core": "workspace:*",
    "@trigger.dev/sdk": "workspace:3.0.0-beta.46"
  }
}
```

**Version Pinning**:
- **Zod**: Pinned to `3.22.3` across entire monorepo
- **Socket.io**: `4.7.4` (client: `4.7.5`)
- **TypeScript**: Varies by package (4.x - 5.x)

**Patches**: Custom patches in `patches/` directory
- `@changesets/assemble-release-plan`
- `tsup`
- `engine.io-parser`
- `graphile-worker`

### Code Style & Linting

**ESLint Config**: Custom config in `config-packages/eslint-config-custom/`

**Root ESLint** (`.eslintrc.js`):
```javascript
module.exports = {
  extends: ["custom"],
  parserOptions: {
    sourceType: "module",
    ecmaVersion: 2020,
  },
};
```

**Prettier** (`prettier.config.js`):
```javascript
module.exports = {
  semi: true,
  singleQuote: false,
  trailingComma: "es5",
  printWidth: 100,
  tabWidth: 2,
};
```

**Format Command**: `pnpm run format`

### Testing Conventions

**Test Files**:
- Unit tests: `*.test.ts` or `*.spec.ts`
- E2E tests: `tests/e2e/*.spec.ts`
- Colocated with source or in `test/` directory

**Test Frameworks**:
1. **Vitest** (primary) - Most packages
   - Config: `vitest.config.ts`
   - Fast, Jest-compatible
   
2. **Jest** (legacy) - Some packages (core, core-backend, otlp-importer)
   - Config: `jest.config.js`
   - Uses `ts-jest` for TypeScript

3. **Playwright** - E2E tests
   - Config: `playwright.config.ts` (root)

**Test Commands**:
- `pnpm run test` - Run unit tests
- `pnpm run test:dev` - Watch mode
- `pnpm run test:e2e` - E2E tests
- `pnpm run test:e2e:ui` - Playwright UI mode

**Example Test** (Vitest):
```typescript
import { describe, it, expect } from "vitest";

describe("MyFunction", () => {
  it("should return expected value", () => {
    expect(myFunction()).toBe("expected");
  });
});
```

### Naming Conventions

**Packages**:
- Core packages: `@trigger.dev/core`, `@trigger.dev/sdk`
- Integrations: `@trigger.dev/{service}` (e.g., `@trigger.dev/stripe`)
- Framework adapters: `@trigger.dev/{framework}` (e.g., `@trigger.dev/nextjs`)

**Files**:
- React components: PascalCase (e.g., `TaskList.tsx`)
- Utilities: camelCase (e.g., `formatDate.ts`)
- Types: PascalCase (e.g., `Task.ts` or within files)
- Tests: `{name}.test.ts` or `{name}.spec.ts`

**Code**:
- Variables/functions: camelCase
- Types/interfaces: PascalCase
- Constants: UPPER_SNAKE_CASE (for true constants)
- Private members: No underscore prefix (TypeScript private)

### Common Utilities & Patterns

**Zod for Validation**:
```typescript
import { z } from "zod";

const TaskSchema = z.object({
  id: z.string(),
  name: z.string(),
  payload: z.any(),
});

type Task = z.infer<typeof TaskSchema>;
```

**SuperJSON for Serialization**:
- Used for dates, undefined, BigInt, etc.
- Preserves types across serialization boundary

**OpenTelemetry Patterns**:
```typescript
import { trace } from "@opentelemetry/api";

const tracer = trace.getTracer("my-package");

const span = tracer.startSpan("operation-name");
try {
  // Work
} finally {
  span.end();
}
```

**Socket.io Type-safe Communication**:
- Uses custom `zodSocket`, `zodNamespace`, `zodMessageHandler`
- Type-safe event handlers with Zod schemas

**Environment Variables**:
- `.env.example` at root
- `dotenv` package
- Environment schema validation with Zod

## Build & Deployment

### Local Development Setup

**Prerequisites:**
- Node.js 20.11.1
- pnpm 8.15.5
- Docker
- PostgreSQL 14 (via Docker)
- Redis 7 (via Docker)

**Setup Steps:**
1. Clone repository
2. Run `corepack enable`
3. Run `pnpm i`
4. Copy `.env.example` to `.env`
5. Generate `ENCRYPTION_KEY`: `openssl rand -hex 16`
6. Start Docker: `pnpm run docker`
7. Migrate database: `pnpm run db:migrate`
8. Build webapp: `pnpm run build --filter webapp`
9. Run webapp: `pnpm run dev --filter webapp`

**Docker Services** (`docker/docker-compose.yml`):
- PostgreSQL 14 (port 5432)
- pgAdmin 8 (port 5480)
- Redis 7 (port 6379)

**Local URLs:**
- Webapp: http://localhost:3030
- pgAdmin: http://localhost:5480

### Turbo Build Orchestration

**Dependency Graph**: Automatically builds dependencies first

**Example**:
```bash
# Build everything
pnpm run build

# Build specific package and its dependencies
pnpm run build --filter trigger.dev

# Force rebuild (ignore cache)
pnpm run build:force
```

**Caching**:
- Turborepo caches build outputs
- Cache invalidation based on file changes
- Some tasks marked `cache: false` (dev, db operations)

### Build Outputs

**Packages** (tsup):
- `dist/` directory
- Dual ESM/CJS bundles
- Type declarations (`.d.ts`, `.d.mts`)
- Source maps (in dev)

**Webapp** (Remix):
- `public/build/` - Client bundles
- `build/server.js` - Server bundle

**Apps** (esbuild):
- `dist/index.mjs` - Single ESM bundle

### Docker & Containerization

**Containerfiles**:
- `apps/coordinator/Containerfile`
- `apps/docker-provider/Containerfile`
- `apps/kubernetes-provider/Containerfile`
- `packages/cli-v3/src/Containerfile.prod`

**Pattern**: Multi-stage builds with Node.js base images

**Depot Integration**: CLI v3 uses Depot for faster Docker builds

**Build Scripts**:
- `build:image` - Build Docker image
- `docker:build` - Build via Docker Compose

### V3 Deployment

**CLI Commands**:
```bash
# Login to Trigger.dev
pnpm exec triggerdev login -a http://localhost:3030

# Development
pnpm exec triggerdev dev

# Deploy to production
pnpm exec triggerdev deploy
```

**Deployment Process**:
1. Build user tasks
2. Bundle with dependencies
3. Create Docker image
4. Push to registry
5. Deploy to coordinator

**Environment Profiles**:
- `--profile local` - Local development
- `--profile staging` - Staging environment
- `--profile production` - Production

## References & Examples

### V3 Catalog

**Path**: `/home/user/trigger.dev/references/v3-catalog`
**Purpose**: Comprehensive task examples for V3

**Example Tasks**:
- `simple.ts` - Basic task examples
- `batch.ts` - Batch operations
- `retries.ts` - Retry strategies
- `scheduled.ts` - Scheduled tasks
- `subtasks.ts` - Subtask patterns
- `logging.ts` - Logging examples
- `longRunning.ts` - Long-running tasks
- `openai.ts` - OpenAI integration
- `stripe.ts` - Stripe integration

### Other Reference Projects

**Path**: `/home/user/trigger.dev/references/`

1. **job-catalog** - V2 job examples
2. **nextjs-reference** - Next.js integration example
3. **remix-reference** - Remix integration example
4. **astro-reference** - Astro integration example
5. **hono-reference** - Hono integration example
6. **deno-reference** - Deno runtime example
7. **unit-testing** - Testing examples
8. **package-tester** - Package testing utilities

### Testing References

**Path**: `/home/user/trigger.dev/runtime_tests`
- Bun runtime tests
- Deno runtime tests
- Wrangler (Cloudflare Workers) tests
- Node.js tests

## Code Organization Patterns

### File Organization (Webapp)

```
app/
├── routes/              # Remix routes (155+ routes)
├── components/          # React components (18 subdirs)
├── services/            # Business logic (17 services)
├── models/              # Data models
├── presenters/          # View presenters
├── hooks/               # React hooks
├── utils/               # Utilities
├── v3/                  # V3-specific code
├── assets/              # Static assets
└── platform/            # Platform utilities
```

**Route Organization**:
- Nested routes use dot notation: `projects.v3.$projectRef.test.ts`
- Resource routes for API endpoints
- Loader/action pattern for data fetching

### Shared Code Patterns

**API Clients**:
```typescript
// Type-safe API client with Zod
import { apiClient } from "@trigger.dev/core/v3";

const result = await apiClient.createTask({
  id: "task-id",
  payload: { ... },
});
```

**Structured Logging**:
```typescript
import { logger } from "@trigger.dev/sdk/v3";

logger.info("Message", { extra: "metadata" });
logger.error("Error", { error });
```

**Error Handling**:
```typescript
import { ApiError } from "@trigger.dev/core/v3";

try {
  await apiClient.call();
} catch (error) {
  if (error instanceof ApiError) {
    // Handle API error
  }
}
```

## Changesets & Releases

**Version Management**: Changesets for semantic versioning

**Commands**:
```bash
# Add changeset
pnpm run changeset:add

# Version packages
pnpm run changeset:version

# Publish packages
pnpm run changeset:release

# Enter beta prerelease
pnpm run changeset:beta

# Exit prerelease
pnpm run changeset:normal
```

**Process**:
1. Make changes
2. Add changeset: `pnpm run changeset:add`
3. Select affected packages
4. Choose version bump (patch/minor/major)
5. Write changelog entry
6. Commit changeset file

**Changeset Files**: `.changeset/*.md`

## Environment Variables

### Webapp Environment Variables

**Required** (from `turbo.json` globalEnv):
- `DATABASE_URL` - PostgreSQL connection
- `DIRECT_URL` - Direct database connection (for migrations)
- `SESSION_SECRET` - Session encryption
- `ENCRYPTION_KEY` - Two-way encryption for OAuth tokens
- `MAGIC_LINK_SECRET` - Magic link authentication

**Optional**:
- `APP_ORIGIN` - Application origin URL
- `LOGIN_ORIGIN` - Login page origin
- `POSTHOG_PROJECT_KEY` - Analytics
- `AUTH_GITHUB_CLIENT_ID` - GitHub OAuth
- `AUTH_GITHUB_CLIENT_SECRET` - GitHub OAuth
- `RESEND_API_KEY` - Email service
- `FROM_EMAIL` - Sender email
- `REPLY_TO_EMAIL` - Reply-to email
- `TRIGGER_API_KEY` - Trigger.dev API key
- `TRIGGER_API_URL` - Trigger.dev API URL
- `APP_ENV` - Environment (dev/staging/prod)
- `DEBUG` - Debug logging
- `TRIGGER_LOG_LEVEL` - Log level

### V3 Environment Variables

**SDK**:
- `TRIGGER_SECRET_KEY` - API authentication
- `TRIGGER_API_URL` - API endpoint (default: https://api.trigger.dev)

**CLI**:
- `TRIGGER_ACCESS_TOKEN` - User access token
- `TRIGGER_API_URL` - API endpoint

## Key Workflows

### Adding a New Package

1. Create package directory: `packages/my-package/`
2. Add `package.json` with workspace dependencies
3. Add `tsconfig.json` extending shared config
4. Add `tsup.config.ts` for build
5. Implement in `src/index.ts`
6. Add to workspace in `pnpm-workspace.yaml` (auto-included via glob)
7. Build: `pnpm run build --filter my-package`

### Creating a New Integration

1. Create directory: `integrations/my-service/`
2. Copy structure from existing integration (e.g., `openai`)
3. Add dependencies: SDK package + integration-kit
4. Implement integration methods
5. Add tests
6. Add changeset: `pnpm run changeset:add`

### Making a Pull Request

1. Create feature branch
2. Make changes
3. Add changeset (if modifying packages)
4. Run tests: `pnpm run test`
5. Run lint: `pnpm run lint`
6. Run typecheck: `pnpm run typecheck`
7. Commit and push
8. Open PR with description
9. Enable "Allow edits from maintainers"
10. Reference issues: `fixes #123`

### Debugging

**Webapp**:
- Use `DEBUG=*` environment variable
- OpenTelemetry traces in UI
- Console logs with structured logger
- Prisma query logging

**CLI**:
- `--log-level debug` flag
- Source map support enabled

**Tasks**:
- Logging automatically captured
- Trace view in dashboard
- Error stack traces preserved

## Special Considerations

### OpenTelemetry

**Instrumentation**:
- Auto-instrumentation for HTTP, Express, Prisma
- Custom spans with `@opentelemetry/api`
- OTLP export to platform

**Setup** (in apps):
```typescript
import { NodeSDK } from "@opentelemetry/sdk-node";

const sdk = new NodeSDK({
  // configuration
});

sdk.start();
```

### Socket.io

**Type Safety**: Custom zodSocket pattern
```typescript
import { ZodNamespace } from "@trigger.dev/core/v3";

const namespace = ZodNamespace.create({
  onMessage: {
    schema: z.object({ ... }),
    handler: async (message) => { ... }
  }
});
```

**Adapters**: Redis adapter for multi-instance support

### Prisma

**Schema**: `packages/database/prisma/schema.prisma`

**Workflow**:
1. Modify schema
2. `cd packages/database`
3. `pnpm run db:migrate:dev` - Create migration
4. Commit schema + migration
5. Restart TypeScript server in IDE

**Binary Targets**: `["native", "debian-openssl-1.1.x"]` for Docker

### GraphQL Worker

**Used By**: Webapp for background jobs
**Pattern**: PostgreSQL-backed job queue
**Customization**: Custom patches applied

## Troubleshooting

### Port Already in Use
```bash
# Find process on port 3030
lsof -i :3030

# Kill process
sudo kill -9 <PID>
```

### Database Issues
```bash
# Reset database
pnpm run db:reset

# Studio for manual fixes
pnpm run db:studio
```

### Build Cache Issues
```bash
# Clear Turbo cache
pnpm run build:force

# Clean all build outputs
pnpm run clean

# Nuclear option
pnpm run clean:node_modules
pnpm i
```

### V3 Development Issues
- Restart `triggerdev dev` after SDK/core changes
- Rebuild CLI after core changes: `pnpm run build --filter trigger.dev`
- Check project `externalRef` matches in database

## Additional Resources

**Documentation**: `/home/user/trigger.dev/docs`
**Contributing Guide**: `/home/user/trigger.dev/CONTRIBUTING.md`
**Docker Setup**: `/home/user/trigger.dev/DOCKER_INSTALLATION.md`
**Changesets Guide**: `/home/user/trigger.dev/CHANGESETS.md`

**External Links**:
- Website: https://trigger.dev
- Docs: https://trigger.dev/docs
- Discord: https://trigger.dev/discord
- GitHub: https://github.com/triggerdotdev/trigger.dev

---

**Last Updated**: Based on repository state at commit d2d54c9 (v3.0.0-beta.46)
