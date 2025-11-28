# Core Development Rules

## **CRITICAL SKILLS USAGE:**

**Backend Development (MANDATORY):**

- **ALWAYS** use `clean-architecture` skill when **creating files or deciding file locations**
- **ALWAYS** use `backend-engineer` skill when **implementing ANY backend code** (Hono APIs, routes, services)

**Frontend Development (MANDATORY):**

- **ALWAYS** use `frontend-engineer` skill when **implementing frontend features** (Gateway Pattern, Zustand, TanStack Router)
- **ALWAYS** use `ui-designer` skill when **creating UI components** (shadcn/ui, Tailwind, responsive design)
- **ALWAYS** use `gesttione-design-system` skill for **Gesttione brand colors and metrics**

**Code Quality (MANDATORY - ALL CODE - Backend AND Frontend):**

- **ALWAYS** use `code-standards` skill when **implementing ANY feature, fixing bugs, refactoring, writing functions/classes, or designing classes/modules**
- **ALWAYS** use `naming-conventions` skill when **creating ANY files, folders, classes, functions, or variables**
- **ALWAYS** use `typescript-type-safety` skill when **writing ANY TypeScript code**

## **CRITICAL - NEVER IGNORE:**

- **NEVER** commit, create branches, or create pull requests automatically - ALWAYS wait for explicit user request
- **ALWAYS** run `bun run craft` after creating/moving files to update barrel imports
- **NEVER** use `any` type - use `unknown` with type guards instead
- **NEVER** commit without running quality gates: format, lint, type-check and tests
- **NEVER** commit directly to `main` or `dev` branches
- **ALWAYS** create feature branches from `dev`
- **ALWAYS** create unit test files in `__tests__/` folder inside the same directory as the file being tested
- **ALWAYS** create integration and E2E tests in `tests/` folder at project root

## Tech Stack

### Backend:

- Runtime: Bun
- Framework: Hono
- Database: PostgreSQL com Drizzle ORM
- Cache: Redis (via ioredis)
- Queue: AWS SQS (LocalStack local)

### Frontend:

- Framework: React 19 + Vite 6
- Router: TanStack Router
- UI: shadcn/ui + Tailwind 4
- State Management: Zustand (global client state) + TanStack Query (server state)

**Architecture:** Simplified feature-based organization

- Pages orchestrate business logic (use cases)
- Components are pure UI (no stores/gateways)
- Stores (Zustand) are framework-agnostic, 100% testable
- Gateways injected via Context API for isolated testing

**Structure per feature:**

```
features/[name]/
├── components/   # Pure UI
├── pages/        # Use cases (orchestration)
├── stores/       # Zustand (state + actions)
├── gateways/     # Interface + HTTP + Fake
├── hooks/        # Custom hooks (optional)
└── types/        # TypeScript types
```

**NO Clean Architecture layers (domain/application/infrastructure)**

### Testing:

- Unit: Vitest + React Testing Library
- Integration: Vitest (API routes, database, external services)
- E2E: Playwright
- Coverage: Vitest (v8 provider)

**MANDATORY Test Structure:**

```
# Unit tests - Co-located with source code
src/domain/entities/
├── user.entity.ts
└── __tests__/
    └── user.entity.test.ts

features/auth/stores/
├── auth.store.ts
└── __tests__/
    └── auth.store.test.ts

src/application/use-cases/
├── create-user.use-case.ts
└── __tests__/
    └── create-user.use-case.test.ts

# Integration & E2E tests - Root-level tests/ folder
tests/
├── integration/
│   ├── api/
│   │   └── auth.integration.test.ts
│   ├── database/
│   │   └── user-repository.integration.test.ts
│   └── services/
│       └── email-service.integration.test.ts
│
└── e2e/
    ├── auth/
    │   └── login.e2e.test.ts
    └── users/
        └── user-registration.e2e.test.ts
```

**Critical Testing Rules:**

**Unit Tests (co-located):**

- **ALWAYS** create unit test files in `__tests__/` folder inside the same directory as the file being tested
- **ALWAYS** use `.test.ts` or `.test.tsx` extension for unit test files
- **ALWAYS** name test file same as source file: `user.entity.ts` → `__tests__/user.entity.test.ts`
- **NEVER** place unit tests in root-level `tests/` directory
- **ALWAYS** co-locate unit tests with code for easier navigation and maintenance

**Integration & E2E Tests (root-level):**

- **ALWAYS** create integration tests in `tests/integration/` at project root
- **ALWAYS** create E2E tests in `tests/e2e/` at project root
- **ALWAYS** use `.integration.test.ts` extension for integration tests
- **ALWAYS** use `.e2e.test.ts` extension for E2E tests
- **ALWAYS** organize by feature/domain in subdirectories

### Code Quality:

- Linting/Formatting: Biome (TS/JS/CSS)
- Markdown: Prettier
- TypeScript: Strict mode

## UI/Design System (MANDATORY)

### Quick Reference - When to Use Each Skill

1. **`ui-designer` skill** (from ui plugin)

   - Creating or designing UI components
   - Implementing responsive layouts with Tailwind CSS
   - Setting up shadcn/ui components
   - Building forms with TanStack Form + Zod validation
   - Implementing dark mode and theme support
   - Ensuring WCAG 2.1 AA accessibility compliance
   - Component composition and design patterns
   - Adding animations and transitions

2. **`gesttione-design-system` skill** (from ui plugin)
   - Applying Gesttione brand colors and identity
   - Creating metric visualizations (revenue, CMV, purchases, costs, customers, etc.)
   - Building dashboard components with metric cards
   - Using Gesttione typography system (Geist, Lora, Geist Mono)
   - Implementing Gesttione-specific design tokens
   - Brand-compliant UI components
   - Business metric displays and status badges

### Critical UI Rules

- **ALWAYS** use design tokens (CSS variables) for colors, NEVER hardcoded hex values
- **ALWAYS** ensure WCAG AA contrast ratios (4.5:1 for text, 3:1 for UI components)
- **ALWAYS** implement dark mode support with proper token mapping
- **ALWAYS** start with mobile-first responsive design
- **ALWAYS** use shadcn/ui components when available
- **NEVER** use inline styles (use Tailwind classes)
- **NEVER** skip accessibility attributes (ARIA, semantic HTML, keyboard navigation)
- For Gesttione projects, **ALWAYS** use `gesttione-design-system` skill for brand colors and metrics

## Backend Architecture (MANDATORY)

**Clean Architecture is REQUIRED for ALL backend code.**

**For ALL backend implementation work, use these skills from architecture-design plugin:**

- `clean-architecture` - Layered architecture, dependency rule, DDD patterns, file location decisions
- `backend-engineer` - Hono APIs, HTTP routes, service layer, DI Container, implementation examples

**IMPORTANT**: Also apply Code Quality skills (see "Code Quality & Clean Code" section):

- `code-standards` - SOLID principles (SRP, OCP, LSP, ISP, DIP) + Clean Code patterns (KISS, YAGNI, DRY, TDA)

### Quick Reference - When to Use Each Skill

1. **`clean-architecture` skill** (from architecture-design plugin)

   - **ALWAYS use when creating files** - Deciding file locations and layer organization
   - **ALWAYS use when organizing layers** - Domain/Application/Infrastructure (with HTTP layer)
   - Creating entities, value objects, aggregates, domain events
   - Defining repository patterns and ports (interface contracts)
   - Implementing use cases and application services
   - Understanding dependency flow and dependency rule
   - Domain-driven design (DDD) patterns
   - Ensuring proper layer separation

2. **`backend-engineer` skill** (from architecture-design plugin)

   - **ALWAYS use when implementing ANY backend code** - Hono APIs, HTTP routes, services
   - Creating Hono route handlers and controllers
   - Implementing HTTP endpoints and API design
   - Setting up dependency injection container
   - Configuring DI lifetimes (singleton, scoped, transient)
   - Repository implementations (database layer)
   - Infrastructure adapter implementations (Cache, Logger, Queue)
   - Service layer implementation patterns
   - Backend-specific best practices and examples

### Quick Reference - Clean Architecture Structure

**Structure per module/feature:**

```
src/
├── domain/                 # Layer 1 (innermost, no dependencies)
│   ├── entities/          # Business entities
│   ├── value-objects/     # Immutable value objects
│   ├── aggregates/        # Aggregate roots
│   ├── events/            # Domain events
│   └── ports/             # Interface contracts (NO "I" prefix)
│       ├── repositories/  # Repository interfaces
│       ├── services/      # Service interfaces
│       └── http-server.ts # HTTP server interface
│
├── application/           # Layer 2 (depends on Domain only)
│   ├── use-cases/         # Application business rules
│   └── dtos/              # Data transfer objects
│
└── infrastructure/        # Layer 3 (depends on Application + Domain)
    ├── controllers/       # Self-registering HTTP controllers
    │   └── schemas/       # Zod validation schemas (request/response contracts)
    ├── repositories/      # Repository implementations (Drizzle ORM)
    ├── adapters/          # External service adapters
    │   ├── cache/         # Redis adapter (implements CacheService port)
    │   ├── logger/        # Logger adapter (implements Logger port)
    │   └── queue/         # SQS adapter (implements Queue port)
    ├── http/              # HTTP layer (framework-specific)
    │   ├── server/
    │   │   └── hono-http-server.adapter.ts  # Hono implementation of HttpServer port
    │   ├── middleware/    # HTTP middleware (auth, validation, error handling)
    │   └── plugins/       # Hono plugins (CORS, compression, OpenAPI, etc.)
    ├── database/          # Drizzle schemas, migrations, connection
    └── container/         # DI Container (composition root)
```

**Dependency flow:** Domain ← Application ← Infrastructure

### Quick Reference - Layers

1. **Domain Layer** (innermost, no dependencies)

   - Entities, Value Objects, Aggregates, Domain Events
   - Ports: Interface contracts (repositories, services) - NO "I" prefix

2. **Application Layer** (depends on Domain only)

   - Use Cases: Application-specific business rules
   - DTOs: Data transfer between layers

3. **Infrastructure Layer** (depends on Application + Domain)

   - Repositories: Database implementations (implements domain/ports/repositories)
   - Adapters: External service implementations (Cache, Logger, Queue)
   - Controllers: Self-registering HTTP controllers (thin layer, delegate to use cases)
     - Schemas: Zod validation schemas for HTTP contracts (requests/responses)
   - HTTP Layer: Framework-specific HTTP handling
     - Server: Hono adapter (implements HttpServer port)
     - Middleware: HTTP middleware (auth, validation, error handling)
     - Plugins: Hono plugins (CORS, compression, OpenAPI, etc.)
   - Database: Drizzle schemas, migrations, connection management
   - Container: DI Container (composition root)

### Critical Backend Rules

**NEVER:**

- Import infrastructure in domain layer (violates dependency rule)
- Expose domain entities directly in API responses (use DTOs/ViewModels)
- Use external DI libraries (use custom DI Container only)
- Prefix interfaces with "I" (use semantic names without prefix)
- Skip dependency injection (ALWAYS inject via constructors)
- Create circular dependencies between layers
- Put business logic in controllers (delegate to use cases)

**ALWAYS:**

- **Use `clean-architecture` skill when creating files** - Proper file location and layer organization
- **Use `backend-engineer` skill when implementing backend code** - Hono APIs, routes, services
- **Use `code-standards` skill when writing backend code** - SOLID principles + Clean Code patterns
- Inject dependencies via constructors (constructor injection)
- Define interfaces in `domain/ports/` (NO "I" prefix)
- Use DTOs/ViewModels for HTTP layer (never expose entities)
- Follow dependency rule: Infrastructure (with HTTP) → Application → Domain
- Implement repository pattern for data access
- Use type-safe DI tokens with Symbol
- Separate registration functions by layer (registerDomain, registerApplication, etc.)

### Controller Pattern (Self-Registering)

**Controllers now auto-register routes in the constructor:**

```typescript
// infrastructure/http/controllers/system.controller.ts

/**
 * SystemController
 *
 * Infrastructure layer (HTTP) - handles HTTP requests.
 * Thin layer that delegates to use cases.
 *
 * Responsibilities:
 * 1. Register routes in constructor
 * 2. Validate requests (Zod schemas)
 * 3. Delegate to use cases
 * 4. Format responses (return DTOs)
 *
 * NO business logic here! Controllers should be thin.
 *
 * Pattern: Constructor Injection + Auto-registration
 */

import type { GetSystemInfoUseCase } from "@/application/use-cases/get-system-info.use-case";
import type { HttpServer } from "@/domain/ports/http-server";
import { HttpMethod } from "@/domain/ports/http-server";

export class SystemController {
  constructor(
    private readonly httpServer: HttpServer, // ✅ HttpServer port injected
    private readonly getSystemInfoUseCase: GetSystemInfoUseCase // ✅ Use case injected
  ) {
    this.registerRoutes(); // ✅ Auto-register routes in constructor
  }

  private registerRoutes(): void {
    // Register routes using HttpServer port
    this.httpServer.route(HttpMethod.GET, "/system/info", async (context) => {
      try {
        // Delegate to use case (business logic lives there)
        const systemInfo = await this.getSystemInfoUseCase.execute();
        return context.json(systemInfo, 200);
      } catch (error) {
        console.error("Error getting system info:", error);
        return context.json({ error: "Internal server error" }, 500);
      }
    });
  }
}
```

**HttpServer Port (Domain Layer):**

```typescript
// domain/ports/http-server.ts
export enum HttpMethod {
  GET = "GET",
  POST = "POST",
  PUT = "PUT",
  DELETE = "DELETE",
  PATCH = "PATCH",
}

export type HttpHandler = (context: unknown) => Promise<Response | unknown>;

export interface HttpServer {
  route(method: HttpMethod, url: string, handler: HttpHandler): void;
  listen(port: number): void;
}
```

**HonoHttpServer Implementation (Infrastructure Layer):**

```typescript
// infrastructure/http/server/hono-http-server.adapter.ts
import type { Context } from "hono";
import { Hono } from "hono";
import {
  type HttpHandler,
  HttpMethod,
  type HttpServer,
} from "@/domain/ports/http-server";

export class HonoHttpServer implements HttpServer {
  private readonly app: Hono;

  constructor() {
    this.app = new Hono();
  }

  route(method: HttpMethod, url: string, handler: HttpHandler): void {
    const honoHandler = async (c: Context) => {
      try {
        const result = await handler(c);
        return result instanceof Response ? result : (result as Response);
      } catch (error) {
        console.error("Error handling request:", error);
        return c.json({ error: "Internal server error" }, 500);
      }
    };

    switch (method) {
      case HttpMethod.GET:
        this.app.get(url, honoHandler);
        break;
      case HttpMethod.POST:
        this.app.post(url, honoHandler);
        break;
      // ... other methods
    }
  }

  listen(port: number): void {
    console.log(`Server is running on http://localhost:${port}`);
    Bun.serve({
      fetch: this.app.fetch,
      port,
    });
  }
}
```

**Bootstrap (Entry Point):**

```typescript
// main.ts
import { getAppContainer, TOKENS } from "@/infrastructure/di";

const DEFAULT_PORT = 3000;

async function bootstrap() {
  // Get application container (DI)
  const container = getAppContainer();

  // Initialize controllers (they auto-register routes in constructor)
  container.resolve(TOKENS.systemController);
  container.resolve(TOKENS.userController);

  // Start HTTP server
  const server = container.resolve(TOKENS.httpServer);
  const port = Number(process.env.PORT) || DEFAULT_PORT;

  server.listen(port);
}

bootstrap().catch((error) => {
  console.error("Failed to start server:", error);
  process.exit(1);
});
```

**Key Benefits:**

- ✅ **Thin controllers** - Only route registration + delegation
- ✅ **Auto-registration** - Controllers register themselves in constructor
- ✅ **Framework-agnostic domain** - HttpServer port in domain layer
- ✅ **Testable** - Easy to mock HttpServer for testing
- ✅ **DI-friendly** - Controllers resolve via container
- ✅ **Clean separation** - No routes/ folder needed

## Dependency Injection Container

**Use custom DI Container (NO external libraries like InversifyJS or TSyringe)**

**For complete DI implementation, see `backend-engineer` skill from architecture-design plugin**

**Quick Reference:**

- **Symbol-based tokens**: Type-safe DI with `Symbol('Name') as Token<Type>`
- **Lifetimes**: singleton (core, repos), scoped (use cases), transient (rare)
- **Registration by layer**: Separate register functions per layer
- **Composition root**: `infrastructure/container/main.ts`

## Barrel Files Strategy

**Use barrel files for clean imports:**

```typescript
// ✅ Good - uses barrel files
import { UserIdentity } from "@/domain/aggregate";
import { Email, Password } from "@/domain/value-object";

// ❌ Avoid - relative imports
import { UserIdentity } from "../../domain/aggregate/user-identity.aggregate";
```

## Git Standards

- Always run tests/type-check before committing (use `/quality:check`)
- Use `/git:commit` for conventional commits
- Use `/git:pr-creation` for creating pull requests

**Git Workflow Responsibilities:**

- **Quality Gates (BEFORE commit):** Use `project-workflow` skill - Pre-commit checklist, quality gates execution order, Bun-specific commands
- **Commits (CREATE commit):** Use `/git:commit` command - Analyze changes, generate conventional commit message
- **Pull Requests (CREATE PR):** Use `/git:pr-creation` command - Analyze commits, generate comprehensive PR description

**For complete workflow process (pre-commit checklist, quality gates execution order, Bun-specific commands), see `project-workflow` skill from architecture-design plugin**

## Error Handling Patterns

**For complete error handling guidance, use `error-handling-patterns` skill from architecture-design plugin**

**Quick Reference:**

- Use Result/Either types for expected failures
- Log errors with correlation IDs for tracing
- Implement circuit breakers for external services

## Code Quality & Clean Code (MANDATORY - ALL CODE)

**For ALL code generation (backend AND frontend), use these skills from architecture-design plugin:**

- `code-standards` - SOLID principles (SRP, OCP, LSP, ISP, DIP) + Clean Code patterns (KISS, YAGNI, DRY, TDA)
- `naming-conventions` - File, folder, class, function, and variable naming standards
- `typescript-type-safety` - Type guards, avoiding `any` type

### Quick Reference - When to Use Each Skill

1. **`code-standards` skill** (from architecture-design plugin)

   - **ALWAYS use when implementing ANY feature** - New features, bug fixes, refactoring
   - **ALWAYS use when writing functions or classes** - Ensure KISS, YAGNI, DRY, TDA
   - **ALWAYS use when designing ANY classes or modules** - SOLID principles application
   - **ALWAYS use when implementing domain logic** - Business rules in ANY layer
   - Writing new functions or classes
   - Refactoring existing code
   - Code reviews and quality checks
   - Preventing over-engineering
   - Keeping code simple and maintainable
   - Applying Rule of Three before abstraction (DRY)
   - Implementing Tell, Don't Ask pattern (TDA)
   - Ensuring functions are < 20 lines
   - Applying Single Responsibility Principle (one reason to change)
   - Ensuring Open/Closed Principle (open for extension, closed for modification)
   - Following Liskov Substitution Principle (subtypes are substitutable)
   - Implementing Interface Segregation (small, focused interfaces)
   - Applying Dependency Inversion (depend on abstractions)
   - Designing repository patterns
   - Creating extensible architectures

2. **`naming-conventions` skill** (from architecture-design plugin)

   - **ALWAYS use when creating ANY files, folders, classes, functions, or variables**
   - `kebab-case` for files/folders, `PascalCase` for classes, `camelCase` for functions/variables, `SCREAMING_SNAKE_CASE` for constants
   - NO "I" prefix for interfaces (e.g., `UserRepository`, not `IUserRepository`)

3. **`typescript-type-safety` skill** (from architecture-design plugin)
   - **ALWAYS use when writing ANY TypeScript code** - Frontend AND backend
   - Never use `any` type (use `unknown` with type guards)
   - Implementing type guards for runtime safety
   - Leveraging discriminated unions
   - Applying conditional types
   - Ensuring strict type safety
   - Handling unknown types safely

### Critical Code Quality Rules

**NEVER:**

- Write functions > 20 lines (extract into smaller functions)
- Create abstractions before Rule of Three (wait for 3 occurrences)
- Write clever, complex code (prefer clear, boring code)
- Use magic numbers (use named constants)
- Over-engineer simple solutions (apply KISS)
- Build features not in current requirements (apply YAGNI)
- Skip meaningful naming (code should be self-documenting)
- Create God classes (one responsibility per class)
- Use "I" prefix for interfaces (use semantic names)

**ALWAYS:**

- **Use `code-standards` skill when writing ANY code** - SOLID + Clean Code patterns
- **Use `naming-conventions` skill when creating files/classes/functions** - Consistent naming
- **Use `typescript-type-safety` skill when writing TypeScript** - Strict type safety
- Apply KISS (Keep It Simple, Stupid)
- Apply YAGNI (You Aren't Gonna Need It)
- Apply DRY after Rule of Three (3 occurrences)
- Apply TDA (Tell, Don't Ask)
- Keep functions < 20 lines
- Use meaningful names over comments
- Write self-documenting code
- Prefer early returns to reduce nesting
- Use single level of abstraction per function
- Extract complex logic into separate methods
- Follow SOLID principles for classes and modules

## Security Stack

- Validation: Zod schemas at system boundaries
- Passwords: bcrypt hashing
- Secrets: Environment variables only
- Rate limiting on public endpoints

## MCP Server Usage Rules

**MANDATORY - ALWAYS FOLLOW:**

### Private Repository Access & Code Intelligence

- **ALWAYS** use **Octocode MCP** for:

  - Accessing private GitHub repositories (backend services, microservices)
  - Exploring repository structure and architectural patterns
  - Searching code across organizational repositories
  - Finding implementation examples in internal codebases
  - Analyzing commit history and PR evolution
  - Retrieving file contents from private repos
  - Framework-specific code discovery (e.g., Hono middleware patterns, React patterns)
  - Package dependency research (npm/PyPI metadata)

- **ALWAYS** use **Sourcebot MCP** for:
  - Multi-repository semantic search across all indexed repositories
  - Fast trigram-indexed code search (milliseconds across millions of lines)
  - Regex pattern matching with rich query language
  - Branch-specific searches and symbol definitions
  - Large-scale refactoring and code analysis tasks
  - Complex workflows spanning multiple repositories simultaneously

### Documentation Lookup

- **ALWAYS** use **Context7 MCP** for:
  - Comprehensive documentation search across multiple sources
  - Private documentation or internal resources
  - Library documentation and framework patterns
  - Import statements, API usage, configuration guidance
  - Version-specific implementation requirements
  - Need curated, version-specific framework patterns
  - Framework compliance verification (React, Hono, TanStack Router, etc.)

### Web Research & Documentation

- **ALWAYS** use **Perplexity MCP** for:
  - Technical research & documentation (hundreds of QPS)
  - Code patterns, benchmarks, and technical solutions
  - Real-time semantic search with superior accuracy
  - Developer-focused research and API documentation
  - Complex reasoning validation (30 QPS limit)
  - Conversational synthesis when EXA insufficient
  - Latest trends and emerging technologies

### Using MCP Servers in Skills

**All skills should use MCP servers for up-to-date documentation lookup:**

**Context7 MCP Examples:**

- "How do I configure Biome for monorepos?" → Use Context7 MCP for Biome docs
- "What are Playwright's latest features?" → Use Context7 MCP for Playwright docs
- "How to use React 19 useActionState?" → Use Context7 MCP for React docs
- "What are TanStack Router file-based routing patterns?" → Use Context7 MCP for TanStack Router docs

**Perplexity MCP Examples:**

- "Best practices for E2E testing" → Use Perplexity MCP for research and patterns
- "What are best practices for pre-commit hooks?" → Use Perplexity MCP for research
- "How to structure React applications?" → Use Perplexity MCP for architectural guidance
- "Performance optimization techniques for React 19" → Use Perplexity MCP for latest patterns

**Sourcebot MCP Examples:**

- "Find all authentication middleware patterns across our microservices" → Use Sourcebot MCP with `filterByRepoIds`
- "Search for deprecated API usage in all repositories" → Use Sourcebot MCP for large-scale refactoring
- "Compare error handling patterns between services" → Use Sourcebot MCP with multiple `filterByRepoIds`

### Code Search and Discovery

**TOOL HIERARCHY** - Use tools in this order:

1. **Sourcebot MCP** (if available) - Multi-repository semantic search with regex support
2. **Osgrep** (when Sourcebot MCP is NOT available) - Local semantic search that understands code meaning
3. **Built-in search, grep, or find** (last resort) - Only for exact string pattern matching

<sourcebot>
## Overview

Sourcebot MCP is a Model Context Protocol server that enables AI agents to perform intelligent code search across
thousands of repositories. It uses trigram indexing for millisecond searches across millions of lines of code.

## When to Use

**ALWAYS** use Sourcebot MCP when available for code search and discovery. It's the preferred tool over Osgrep and
traditional search tools.

## Required Workflow

**MANDATORY**: Always use Sourcebot MCP (5-7 times) to search code and find information about libraries, frameworks, and
code patterns

**CRITICAL**: You MUST use `filterByRepoIds` parameter on EVERY `mcp_sourcebot_search_code` call to restrict searches to
the relevant repository. Never call `search_code` without specifying `filterByRepoIds`

**REQUIRED WORKFLOW**:

1. First, call `mcp_sourcebot_list_repos` to get the list of available repository IDs
2. Identify the relevant repository ID(s) for your search (e.g., "github.com/Effect-TS/effect" for EffectTS,
   "github.com/vercel/ai" for AI SDK)
3. Always include `filterByRepoIds` with the appropriate repository ID(s) when calling `mcp_sourcebot_search_code`
4. Use `mcp_sourcebot_get_file_source` to fetch specific file contents when needed (this tool requires `repoId`
   parameter, so repository filtering is already enforced)

## Usage Pattern

When searching for code examples or documentation:

- Use descriptive queries in the `query` parameter (not just keywords, but full descriptions of what you're looking for)
- Always specify `filterByRepoIds` to scope your search to the correct repository
- Use `includeCodeSnippets: true` when you need to see actual code examples

## Examples

- Searching EffectTS patterns: Use `filterByRepoIds: ["github.com/Effect-TS/effect"]`
- Searching AI SDK: Use `filterByRepoIds: ["github.com/vercel/ai"]`
- Searching multiple repos: Use `filterByRepoIds: ["repo1", "repo2"]` when comparing patterns across libraries

## Key Features

- **Multi-repository search**: Search across all indexed repositories simultaneously
- **Regex support**: Use regular expressions for precise pattern matching
- **Rich query language**: Filter by files, repos, languages, and symbol definitions
- **Fast performance**: Millisecond searches using trigram indexing
- **Branch-specific searches**: Search across multiple branches
- **Syntax highlighting**: Support for 100+ programming languages

## Tool Hierarchy

**IMPORTANT**: Sourcebot MCP is the primary code search tool:

1. **Sourcebot MCP** (if available) - Multi-repository semantic search
2. **Osgrep** (when Sourcebot MCP is NOT available) - Local semantic search
3. **grep/find** (last resort) - Only for exact string matching

## Enforcement

**IF YOU DON'T USE THIS** as mentioned above your task will be invalidated </sourcebot>

<osgrep>
## When to use

Use `osgrep` for all code and concept discovery when Sourcebot MCP is NOT available. Do not use `grep` or `find` unless
you must match an exact string and `osgrep` fails.

## How to use

**MANDATORY: Always run `osgrep index` before searching.** This ensures the repository is fully indexed and search
results are accurate and complete.

**Always use the `--json` flag.** The server auto-starts and keeps the index fresh.

### Indexing (Required First Step)

**You MUST run indexing before any search operation:**

```bash
osgrep index              # Index current directory
osgrep index --dry-run    # Preview what would be indexed (optional)
```

This step:

- Pre-warms the cache for faster searches
- Ensures all recent changes are indexed
- Respects `.gitignore` and `.osgrepignore` automatically
- Uses adaptive throttling to prevent system overload

### Basic Search

After indexing, ask a natural language question. Do not `ls` first.

```bash
osgrep --json "How are user authentication tokens validated?"
osgrep --json "Where do we handle retries or backoff?"
```

### Scoped Search

Limit search to a specific directory.

```bash
osgrep --json "auth middleware" src/api
```

### Helpful flags

- `--json`: **Required.** Returns structured data (path, line, score, content).
- `-m <n>`: Max total results (default: 25).
- `--per-file <n>`: Max matches per file (default: 1). Use `--per-file 5` when exploring a specific file.

### Strategy

1. **MANDATORY:** Run `osgrep index` to ensure the repository is indexed.
2. Run `osgrep --json "<question>" [path]`.
3. The output is a dense JSON snippet. If it answers the question, stop.
4. Only use `Read` if you need the full file context for a returned path.
5. If results are vague, refine the query or increase `-m`.
6. If you've made significant changes, re-run `osgrep index` before searching again.

## Tool Hierarchy

**IMPORTANT**: Osgrep is the fallback when Sourcebot MCP is unavailable:

1. **Sourcebot MCP** (if available) - Multi-repository semantic search
2. **Osgrep** (when Sourcebot MCP is NOT available) - Local semantic search
3. **grep/find** (last resort) - Only for exact string matching

## Keywords

semantic search, code search, local search, grep alternative, find code, explore codebase, understand code, search by meaning
</osgrep>
