---
name: clean-architecture
description: Clean Architecture principles for Modular Monolith with bounded contexts and minimal shared kernel. **ALWAYS use when working on backend code, ESPECIALLY when creating files, deciding file locations, or organizing contexts (auth, tax, bi, production).** Use proactively to ensure context isolation and prevent "Core Obesity Syndrome". Examples - "create entity", "add repository", "where should this file go", "modular monolith", "bounded context", "shared kernel", "context isolation", "file location", "layer organization".
---

You are an expert in Clean Architecture for Modular Monoliths. You guide developers to structure applications with isolated bounded contexts, minimal shared kernel ("anoréxico"), and clear boundaries following the principles: "Duplication Over Coupling", KISS, YAGNI, and "Start Ugly, Refactor Later".

## When to Engage

You should proactively assist when:

- Structuring a new module or bounded context
- Deciding where files belong (which context)
- Designing use cases within a context
- Creating domain entities and value objects
- Preventing shared kernel growth
- Implementing context communication patterns
- User asks about Modular Monolith, Clean Architecture or DDD

## Modular Monolith Principles (CRITICAL)

### 1. Bounded Contexts Over Shared Layers

**NEVER use flat Clean Architecture (domain/application/infrastructure shared by all).**

Instead, use isolated bounded contexts:

```
src/
├── contexts/                    # Bounded contexts (NOT shared layers)
│   ├── auth/                   # Complete vertical slice
│   │   ├── domain/
│   │   ├── application/
│   │   └── infrastructure/
│   │
│   ├── tax/                     # Complete vertical slice
│   │   ├── domain/
│   │   ├── application/
│   │   └── infrastructure/
│   │
│   └── [other contexts]/
│
└── shared/                      # Minimal shared kernel
    └── domain/
        └── value-objects/       # ONLY UUIDv7 and Timestamp!
```

### 2. "Anoréxico" Shared Kernel

**Rule: Shared kernel must be minimal (< 5 files)**

Before adding ANYTHING to shared/, it must pass ALL criteria:

- ✅ Used by ALL contexts (not 2, not 3, ALL)
- ✅ EXACTLY identical in all uses
- ✅ Will NEVER need context-specific variation
- ✅ Is truly infrastructure, not domain logic

**Only allowed in shared:**

- `uuidv7.value-object.ts` - Universal identifier
- `timestamp.value-object.ts` - Universal timestamp
- Infrastructure (DI container, HTTP server, database client)

### 3. Duplication Over Coupling

**PREFER code duplication over creating dependencies:**

```typescript
// ✅ GOOD: Each context has its own Money VO
// contexts/tax/domain/value-objects/tax-amount.ts
export class TaxAmount {
  // Tax-specific implementation
}

// contexts/bi/domain/value-objects/revenue.ts
export class Revenue {
  // BI-specific implementation
}

// ❌ BAD: Shared Money VO couples contexts
// shared/domain/value-objects/money.ts
export class Money {} // NO! Creates coupling
```

### 4. No Base Classes

**NEVER create base classes that couple contexts:**

```typescript
// ❌ BAD: Base class creates coupling
export abstract class BaseEntity {
  id: string;
  createdAt: Date;
  // Forces all entities into same mold
}

// ✅ GOOD: Each entity is standalone
export class User {
  // Only what User needs, no inheritance
}
```

### 5. Context Communication Rules

**Contexts communicate through Application Services, NEVER direct domain access:**

```typescript
// ✅ ALLOWED: Call through application service
import { AuthApplicationService } from "@auth/application/services/auth.service";

// ❌ FORBIDDEN: Direct domain import
import { User } from "@auth/domain/entities/user.entity"; // NEVER!
```

## Core Principles

### The Dependency Rule

**Rule**: Dependencies must point inward, toward the domain

```
┌─────────────────────────────────────────┐
│         Infrastructure Layer            │  ← External concerns
│  (DB, HTTP, Queue, Cache, External APIs)│     (Frameworks, Tools)
└────────────────┬────────────────────────┘
                 │ depends on ↓
┌────────────────▼────────────────────────┐
│         Application Layer               │  ← Use Cases
│  (Use Cases, DTOs, Application Services)│     (Business Rules)
└────────────────┬────────────────────────┘
                 │ depends on ↓
┌────────────────▼─────────────────────────┐
│           Domain Layer                   │  ← Core Business
│  (Entities, Value Objects, Domain Rules) │     (Pure, Framework-free)
└──────────────────────────────────────────┘
```

**Key Points:**

- Domain layer has NO dependencies (pure business logic)
- Application layer depends ONLY on Domain
- Infrastructure layer depends on Application and Domain

### Benefits

1. **Independence**: Business logic doesn't depend on frameworks
2. **Testability**: Core logic tested without databases or HTTP
3. **Flexibility**: Easy to swap implementations (Postgres → MongoDB)
4. **Maintainability**: Clear boundaries and responsibilities

## Layer Structure (Within Each Context)

**IMPORTANT: These layers exist WITHIN each bounded context, not as shared layers.**

### 1. Domain Layer (Per Context)

**Purpose**: Pure business logic for this specific context

**Location**: `contexts/[context-name]/domain/`

**Contains:**

- Entities (context-specific business objects)
- Value Objects (context-specific immutable objects)
- Ports (context interfaces - NO "I" prefix)
- Domain Events (context events)
- Domain Services (context-specific logic)
- Domain Exceptions (context errors)

**Rules:**

- ✅ NO dependencies on other layers
- ✅ NO dependencies on other contexts
- ✅ NO framework dependencies
- ✅ Pure TypeScript/JavaScript
- ✅ Duplication over shared abstractions

**Example Structure:**

```
contexts/auth/domain/
├── entities/
│   ├── user.entity.ts
│   └── order.entity.ts
├── value-objects/
│   ├── email.value-object.ts
│   ├── money.value-object.ts
│   └── uuidv7.value-object.ts
├── ports/
│   ├── repositories/
│   │   ├── user.repository.ts
│   │   └── order.repository.ts
│   ├── cache.service.ts
│   └── logger.service.ts
├── events/
│   ├── user-created.event.ts
│   └── order-placed.event.ts
├── services/
│   └── pricing.service.ts
└── exceptions/
    ├── user-not-found.exception.ts
    └── invalid-order.exception.ts
```

**Key Concepts:**

- **Entities** have identity and lifecycle (User, Order)
- **Value Objects** are immutable and compared by value (Email, Money, UUIDv7)
- **Ports** are interface contracts (NO "I" prefix) that define boundaries
- **Domain behavior** lives in entities, not in services

#### Value Object Example: UUIDv7

**UUIDv7 is the recommended identifier for all entities.** It provides:

- Time-ordered IDs (monotonic, better database performance)
- Sequential writes (optimal for B-tree indexes)
- Sortable by creation time
- Uses `Bun.randomUUIDv7()` internally (available since Bun 1.3+)

```typescript
// domain/value-objects/uuidv7.value-object.ts

/**
 * UUIDv7 Value Object (Generic)
 *
 * Generic UUID version 7 implementation that can be used by any entity.
 *
 * Responsibilities:
 * - Generate time-ordered UUIDv7 identifiers
 * - Validate UUID format
 * - Provide type safety
 * - Immutable by design
 *
 * Why UUIDv7?
 * - Time-ordered: Monotonic, better database performance
 * - Sequential writes: Optimal for B-tree indexes
 * - Sortable: Natural ordering by creation time
 * - Encodes: Timestamp + random value + counter
 *
 * Usage:
 * Use as-is for entity identifiers:
 * - UserId
 * - OrderId
 * - ProductId
 * - etc.
 *
 * Available since Bun 1.3+
 */
export class UUIDv7 {
  private readonly value: string;

  private constructor(value: string) {
    this.value = value;
  }

  /**
   * Generates a new UUIDv7 identifier
   *
   * Uses Bun.randomUUIDv7() which generates time-ordered UUIDs.
   *
   * UUIDv7 features:
   * - Time-ordered: Monotonic, suitable for databases
   * - Better B-tree index performance (sequential insertion)
   * - Sortable by creation time
   * - Encodes timestamp + random value + counter
   *
   * Available since Bun 1.3+
   */
  static generate(): UUIDv7 {
    const uuid = Bun.randomUUIDv7();
    return new UUIDv7(uuid);
  }

  /**
   * Creates UUIDv7 from existing string
   *
   * Use when reconstituting from database or external source.
   *
   * @throws {Error} If UUID format is invalid
   */
  static from(value: string): UUIDv7 {
    if (!UUIDv7.isValid(value)) {
      throw new Error(`Invalid UUID format: ${value}`);
    }
    return new UUIDv7(value);
  }

  /**
   * Validates UUID format
   *
   * Accepts standard UUID format (v4, v7, etc.)
   */
  private static isValid(value: string): boolean {
    if (!value || typeof value !== "string") {
      return false;
    }

    const uuidRegex =
      /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
    return uuidRegex.test(value);
  }

  /**
   * Compares two UUIDs for equality
   *
   * Value Objects are equal if their values are equal.
   */
  equals(other: UUIDv7): boolean {
    return this.value === other.value;
  }

  /**
   * Returns string representation
   *
   * Use for serialization (database, JSON, logs).
   */
  toString(): string {
    return this.value;
  }

  /**
   * Returns the raw value
   *
   * Use when you need the typed value explicitly.
   */
  toValue(): string {
    return this.value;
  }
}

/**
 * Type alias for User ID
 *
 * Use this type for all User entity ID references.
 * This provides semantic clarity while using the generic UUIDv7 implementation.
 */
export type UserId = UUIDv7;
```

**Usage in Entities:**

```typescript
// domain/entities/user.entity.ts
import { UUIDv7 } from "@/domain/value-objects/uuidv7.value-object";
import type { Email } from "@/domain/value-objects/email.value-object";

export class User {
  private _isActive: boolean = true;
  private readonly _createdAt: Date;

  constructor(
    private readonly _id: UUIDv7,
    private _email: Email,
    private _name: string,
    private _hashedPassword: string
  ) {
    this._createdAt = new Date();
  }

  deactivate(): void {
    if (!this._isActive) {
      throw new Error(`User ${this._id.toString()} is already inactive`);
    }
    this._isActive = false;
  }

  get id(): UUIDv7 {
    return this._id;
  }

  get email(): Email {
    return this._email;
  }

  get name(): string {
    return this._name;
  }

  get isActive(): boolean {
    return this._isActive;
  }

  get createdAt(): Date {
    return this._createdAt;
  }
}
```

**Usage in Use Cases:**

```typescript
// application/use-cases/create-user.use-case.ts
import { UUIDv7 } from "@/domain/value-objects/uuidv7.value-object";
import { User } from "@/domain/entities/user.entity";
import { Email } from "@/domain/value-objects/email.value-object";

export class CreateUserUseCase {
  async execute(dto: CreateUserDto): Promise<UserResponseDto> {
    // Generate UUIDv7 for new user
    const id = UUIDv7.generate();
    const email = Email.create(dto.email);
    const user = new User(id, email, dto.name, dto.hashedPassword);

    await this.userRepository.save(user);

    return {
      id: user.id.toString(),
      email: user.email.toString(),
      name: user.name,
      isActive: user.isActive,
      createdAt: user.createdAt.toISOString(),
    };
  }
}
```

**Usage in Repositories:**

```typescript
// infrastructure/repositories/user.repository.impl.ts
import { UUIDv7 } from "@/domain/value-objects/uuidv7.value-object";
import { User } from "@/domain/entities/user.entity";

export class UserRepositoryImpl implements UserRepository {
  async findById(id: UUIDv7): Promise<User | null> {
    const row = await this.db
      .select()
      .from(users)
      .where(eq(users.id, id.toString()))
      .limit(1);

    if (!row) return null;

    // Reconstruct domain entity
    const userId = UUIDv7.from(row.id);
    const email = Email.create(row.email);
    return new User(userId, email, row.name, row.hashedPassword);
  }

  async save(user: User): Promise<void> {
    await this.db.insert(users).values({
      id: user.id.toString(),
      email: user.email.toString(),
      name: user.name,
      isActive: user.isActive,
      createdAt: user.createdAt,
    });
  }
}
```

**For complete implementation examples of Entities, Value Objects, and Repositories with Drizzle ORM, see `backend-engineer` skill**

### 2. Application Layer (Use Cases)

**Purpose**: Orchestrate business logic, implement use cases

**Contains:**

- Use Cases / Application Services
- DTOs (Data Transfer Objects)
- Mappers (Entity ↔ DTO)

**Rules:**

- ✅ Depends ONLY on Domain layer
- ✅ Orchestrates entities and value objects
- ✅ NO direct infrastructure dependencies (use interfaces)
- ✅ Stateless services

**Example Structure:**

```
src/application/
├── use-cases/
│   ├── create-user.use-case.ts
│   ├── update-user-profile.use-case.ts
│   └── deactivate-user.use-case.ts
├── dtos/
│   ├── create-user.dto.ts
│   └── user-response.dto.ts
└── mappers/
    └── user.mapper.ts
```

**Use Case Responsibilities:**

1. **Validate** business rules
2. **Orchestrate** domain objects (entities, value objects)
3. **Persist** through repositories (ports)
4. **Coordinate** side effects (events, notifications)
5. **Return** DTOs (never expose domain entities)

**Port (Interface) Example:**

```typescript
// ✅ Port in Domain layer (domain/ports/repositories/user.repository.ts)
// NO "I" prefix
import type { UUIDv7 } from "@/domain/value-objects/uuidv7.value-object";

export interface UserRepository {
  save(user: User): Promise<void>;
  findById(id: UUIDv7): Promise<User | undefined>;
  findByEmail(email: string): Promise<User | undefined>;
}

// Implementation in Infrastructure layer
```

**For complete Use Case examples with DTOs, Mappers, and orchestration patterns, see `backend-engineer` skill**

### 3. Infrastructure Layer (Adapters)

**Purpose**: Implement technical details and external dependencies

**Contains:**

- Repository implementations (implements domain/ports/repositories)
- Adapters (external service implementations)
- Database access (Drizzle ORM)
- HTTP setup (Hono app, middleware, OpenAPI)
- Configuration

**Rules:**

- ✅ Implements interfaces defined in Domain layer (ports)
- ✅ Contains framework-specific code
- ✅ Handles technical concerns
- ✅ NO business logic

**Example Structure:**

```
src/infrastructure/
├── controllers/
│   ├── user.controller.ts
│   ├── order.controller.ts
│   └── schemas/
│       ├── user.schema.ts
│       └── order.schema.ts
├── repositories/
│   ├── user.repository.impl.ts
│   └── order.repository.impl.ts
├── adapters/
│   ├── cache/
│   │   └── redis-cache.adapter.ts
│   ├── logger/
│   │   └── winston-logger.adapter.ts
│   └── queue/
│       ├── sqs-queue.adapter.ts
│       ├── localstack-sqs.adapter.ts
│       └── fake-queue.adapter.ts
├── http/
│   ├── server/
│   │   └── hono-http-server.adapter.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts
│   │   ├── validation.middleware.ts
│   │   └── error-handler.middleware.ts
│   └── plugins/
│       ├── cors.plugin.ts
│       └── openapi.plugin.ts
├── database/
│   ├── drizzle/
│   │   ├── schema/
│   │   │   └── users.schema.ts
│   │   └── migrations/
│   └── connection.ts
└── container/
    └── main.ts
```

**Infrastructure Layer Responsibilities:**

- **Repositories**: Implement ports from `domain/ports/repositories/` using Drizzle ORM
- **Adapters**: Implement external service ports (Cache, Logger, Queue)
- **Controllers**: Self-registering HTTP controllers (thin layer, delegate to use cases)
  - Schemas: Zod validation schemas for HTTP contracts (requests/responses)
- **HTTP Layer**: Framework-specific HTTP handling
  - Server: Hono adapter (implements HttpServer port)
  - Middleware: HTTP middleware (auth, validation, error handling)
  - Plugins: Hono plugins (CORS, compression, OpenAPI, etc.)
- **Database**: Drizzle schemas, migrations, connection management
- **Container**: DI Container (composition root)
- **NO business logic**: Only technical implementation details

**Repository Pattern:**

```typescript
// Port in domain/ports/repositories/user.repository.ts
import type { UUIDv7 } from "@/domain/value-objects/uuidv7.value-object";

export interface UserRepository {
  save(user: User): Promise<void>;
  findById(id: UUIDv7): Promise<User | undefined>;
}

// Implementation in infrastructure/repositories/user.repository.impl.ts
export class UserRepositoryImpl implements UserRepository {
  // Drizzle ORM implementation
}
```

**For complete Repository and Adapter implementations with Drizzle ORM, Redis, and other infrastructure examples, see `backend-engineer` skill**

### 4. HTTP Layer (Framework-Specific, in Infrastructure)

**Purpose**: Handle HTTP requests, WebSocket connections, CLI commands

**Location**: `infrastructure/http/`

**Contains:**

- Server: Hono adapter (implements HttpServer port)
- Controllers: Self-registering HTTP controllers (route registration + handlers)
- Schemas: Zod validation for requests/responses
- Middleware: HTTP middleware (auth, validation, error handling)
- Plugins: Hono plugins (CORS, compression, OpenAPI, etc.)

**Rules:**

- ✅ Part of Infrastructure layer (HTTP is technical detail)
- ✅ Depends on Application layer (Use Cases) and HttpServer port
- ✅ Thin layer - delegates to Use Cases
- ✅ NO business logic
- ✅ Controllers auto-register routes in constructor

**Example Structure:**

```
src/infrastructure/http/
├── server/
│   └── hono-http-server.adapter.ts
├── controllers/
│   ├── user.controller.ts
│   └── order.controller.ts
├── schemas/
│   ├── user.schema.ts
│   └── order.schema.ts
├── middleware/
│   ├── auth.middleware.ts
│   └── error-handler.middleware.ts
└── plugins/
    ├── cors.plugin.ts
    └── openapi.plugin.ts
```

**Controller Responsibilities:**

- **Thin layer**: Validation + Delegation to Use Cases
- **NO business logic**: Controllers should be lightweight
- **Request validation**: Use Zod schemas at entry point
- **Response formatting**: Return DTOs (never domain entities)
- **Self-registering**: Controllers register routes in constructor via HttpServer port

**Controller Pattern (Self-Registering):**

```typescript
// infrastructure/http/controllers/user.controller.ts

/**
 * UserController
 *
 * Infrastructure layer (HTTP) - handles HTTP requests.
 * Thin layer that delegates to use cases.
 *
 * Pattern: Constructor Injection + Auto-registration
 */
import type { HttpServer } from "@/domain/ports/http-server";
import { HttpMethod } from "@/domain/ports/http-server";
import type { CreateUserUseCase } from "@/application/use-cases/create-user.use-case";

export class UserController {
  constructor(
    private readonly httpServer: HttpServer, // ✅ HttpServer port injected
    private readonly createUserUseCase: CreateUserUseCase // ✅ Use case injected
  ) {
    this.registerRoutes(); // ✅ Auto-register routes in constructor
  }

  private registerRoutes(): void {
    // POST /users - Create new user
    this.httpServer.route(HttpMethod.POST, "/users", async (context) => {
      try {
        const dto = context.req.valid("json"); // Validated by middleware
        const user = await this.createUserUseCase.execute(dto);
        return context.json(user, 201);
      } catch (error) {
        console.error("Error creating user:", error);
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

**Key Benefits:**

- ✅ **Framework-agnostic domain** - HttpServer port in domain layer
- ✅ **Testable** - Easy to mock HttpServer for testing controllers
- ✅ **DI-friendly** - Controllers resolve via container
- ✅ **Auto-registration** - Controllers register themselves in constructor
- ✅ **Thin controllers** - Only route registration + delegation
- ✅ **Clean separation** - No routes/ folder needed

**For complete HttpServer implementation (Hono adapter), Zod validation, and middleware patterns, see `backend-engineer` skill**

## Dependency Injection

**Use custom DI Container (NO external libraries like InversifyJS or TSyringe)**

### Why Dependency Injection?

- Enables testability (inject mocks)
- Follows Dependency Inversion Principle
- Centralized dependency management
- Supports different lifetimes (singleton, scoped, transient)

### DI Principles in Clean Architecture

**Constructor Injection:**

```typescript
// ✅ Use cases depend on abstractions (ports), not implementations
export class CreateUserUseCase {
  constructor(
    private readonly userRepository: UserRepository, // Port from domain/ports/
    private readonly passwordHasher: PasswordHasher, // Port from domain/ports/
    private readonly emailService: EmailService // Port from domain/ports/
  ) {}

  async execute(dto: CreateUserDto): Promise<UserResponseDto> {
    // Orchestrate domain logic using injected dependencies
  }
}
```

**Lifetimes:**

- **singleton**: Core infrastructure (config, database, logger, repositories)
- **scoped**: Per-request instances (use cases, controllers)
- **transient**: New instance every time (rarely used)

**For complete DI Container implementation with Symbol-based tokens, registration patterns, and Hono integration, see `backend-engineer` skill**

## Testing Strategy

### Domain Layer Tests (Pure Unit Tests)

```typescript
// ✅ Easy to test - no dependencies
import { describe, expect, it } from "bun:test";
import { User } from "@/domain/entities/user.entity";
import { Email } from "@/domain/value-objects/email.value-object";
import { UUIDv7 } from "@/domain/value-objects/uuidv7.value-object";

describe("User Entity", () => {
  it("should deactivate user", () => {
    const userId = UUIDv7.generate();
    const email = Email.create("user@example.com");
    const user = new User(userId, email, "John Doe", "hashed_password");

    user.deactivate();

    expect(user.isActive).toBe(false);
  });

  it("should throw error when deactivating already inactive user", () => {
    const userId = UUIDv7.generate();
    const email = Email.create("user@example.com");
    const user = new User(userId, email, "John Doe", "hashed_password");
    user.deactivate();

    expect(() => user.deactivate()).toThrow();
  });
});
```

### Application Layer Tests (With Mocks)

```typescript
// ✅ Test use case with mocked ports
import { describe, expect, it, mock } from "bun:test";
import { CreateUserUseCase } from "@/application/use-cases/create-user.use-case";

describe("CreateUserUseCase", () => {
  it("should create user successfully", async () => {
    // Arrange - Mock dependencies
    const mockRepository = {
      save: mock(async () => {}),
      findByEmail: mock(async () => undefined),
    };

    const mockPasswordHasher = {
      hash: mock(async (password: string) => `hashed_${password}`),
    };

    const mockEmailService = {
      sendWelcomeEmail: mock(async () => {}),
    };

    const useCase = new CreateUserUseCase(
      mockRepository as any,
      mockPasswordHasher as any,
      mockEmailService as any
    );

    const dto = {
      email: "test@example.com",
      password: "password123",
      name: "Test User",
    };

    // Act
    const result = await useCase.execute(dto);

    // Assert
    expect(mockRepository.save).toHaveBeenCalledTimes(1);
    expect(mockPasswordHasher.hash).toHaveBeenCalledWith("password123");
    expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledTimes(1);
    expect(result.email).toBe("test@example.com");
  });
});
```

## Common Patterns

### Repository Pattern

```typescript
// Port (Domain layer - domain/ports/repositories/order.repository.ts)
// NO "I" prefix
import type { UUIDv7 } from "@/domain/value-objects/uuidv7.value-object";

export interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: UUIDv7): Promise<Order | undefined>;
  findByUserId(userId: UUIDv7): Promise<Order[]>;
}

// Adapter (Infrastructure layer - infrastructure/repositories/order.repository.impl.ts)
export class OrderRepositoryImpl implements OrderRepository {
  async save(order: Order): Promise<void> {
    // Drizzle ORM implementation
  }
}
```

### Domain Service

```typescript
// ✅ Domain service when logic involves multiple entities
export class PricingService {
  calculateOrderTotal(order: Order, discountRules: DiscountRule[]): Money {
    let total = Money.zero();

    for (const item of order.items) {
      total = total.add(item.price.multiply(item.quantity));
    }

    for (const rule of discountRules) {
      if (rule.appliesTo(order)) {
        total = total.subtract(rule.calculateDiscount(total));
      }
    }

    return total;
  }
}
```

### Event-Driven Communication

```typescript
// Domain Event
import type { UUIDv7 } from "@/domain/value-objects/uuidv7.value-object";
import type { Email } from "@/domain/value-objects/email.value-object";

export class UserCreatedEvent {
  constructor(
    public readonly userId: UUIDv7,
    public readonly email: Email,
    public readonly occurredAt: Date = new Date()
  ) {}
}

// Use Case publishes event
export class CreateUserUseCase {
  async execute(dto: CreateUserDto): Promise<UserResponseDto> {
    // ... create user ...

    await this.eventBus.publish(new UserCreatedEvent(user.id, user.email)); // user.id is UUIDv7

    return UserMapper.toDto(user);
  }
}
```

## Anti-Patterns to Avoid

### ❌ Anemic Domain Model

```typescript
// ❌ Bad - Just data, no behavior
export class User {
  id: string;
  email: string;
  isActive: boolean;
}

// Business logic in service (wrong layer)
export class UserService {
  deactivateUser(user: User): void {
    user.isActive = false;
  }
}
```

**Fix:** Move behavior into entity:

```typescript
// ✅ Good - Rich domain model
export class User {
  deactivate(): void {
    if (!this._isActive) {
      throw new UserAlreadyInactiveError(this._id);
    }
    this._isActive = false;
  }
}
```

### ❌ Domain Layer Depending on Infrastructure

```typescript
// ❌ Bad - Domain depends on infrastructure
import { db } from "@/infrastructure/database";

export class User {
  async save(): Promise<void> {
    await db.insert(users).values(this); // WRONG!
  }
}
```

**Fix:** Keep domain pure, use repository:

```typescript
// ✅ Good - Pure domain, repository handles persistence
export class User {
  // Pure domain logic, no database access
}

// Repository in infrastructure/repositories/
export class UserRepositoryImpl implements UserRepository {
  async save(user: User): Promise<void> {
    await db.insert(users).values(...);
  }
}
```

### ❌ Fat Controllers

```typescript
// ❌ Bad - Business logic in controller
app.post('/users', async (c) => {
  const data = c.req.valid('json');

  // Validation
  if (!data.email.includes('@')) {
    return c.json({ error: 'Invalid email' }, 400);
  }

  // Check if exists
  const exists = await db.select()...;

  // Hash password
  const hashed = await bcrypt.hash(data.password, 10);

  // Save
  await db.insert(users).values(...);

  // Send email
  await sendgrid.send(...);

  return c.json(user, 201);
});
```

**Fix:** Delegate to use case:

```typescript
// ✅ Good - Thin controller
app.post("/users", zValidator("json", CreateUserSchema), async (c) => {
  const dto = c.req.valid("json");
  const user = await createUserUseCase.execute(dto);
  return c.json(user, 201);
});
```

## Migration Strategy

### From Monolith to Clean Architecture

1. **Start with Use Cases** - Extract business logic into use cases
2. **Create Domain Models** - Move entities and value objects to domain layer
3. **Define Ports** - Create interfaces in application layer
4. **Implement Adapters** - Move infrastructure code behind interfaces
5. **Refactor Controllers** - Make them thin, delegate to use cases

## Best Practices

### Do:

- ✅ Keep domain layer pure (no external dependencies)
- ✅ Use interfaces (ports) for all external dependencies
- ✅ Implement rich domain models with behavior
- ✅ Make use cases orchestrate domain logic
- ✅ Test domain logic without infrastructure
- ✅ Use dependency injection at composition root
- ✅ Keep controllers thin (validation + delegation)

### Don't:

- ❌ Put business logic in controllers or repositories
- ❌ Let domain layer depend on infrastructure
- ❌ Create anemic domain models
- ❌ Mix layers (e.g., use Drizzle in domain layer)
- ❌ Skip interfaces (ports) for infrastructure
- ❌ Make use cases depend on concrete implementations

## Remember

- **The Dependency Rule is sacred** - Always point inward
- **Domain is the core** - Everything revolves around it
- **Test the domain first** - It's the most important part
- **Interfaces enable flexibility** - Easy to swap implementations
- **Clean Architecture is about maintainability** - Not perfection
