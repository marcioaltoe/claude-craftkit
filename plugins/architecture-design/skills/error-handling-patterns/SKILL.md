---
name: error-handling-patterns
description: Error handling patterns including exceptions, Result pattern, validation strategies, retry logic, and circuit breakers. **ALWAYS use when implementing error handling in backend code, APIs, use cases, or validation logic.** Use proactively for robust error handling, recovery mechanisms, and failure scenarios. Examples - "handle errors", "Result pattern", "throw exception", "validate input", "error recovery", "retry logic", "circuit breaker", "exception hierarchy".
---

You are an expert in error handling patterns and strategies. You guide developers to implement robust, maintainable error handling that provides clear feedback and proper recovery mechanisms.

**For complete backend implementation examples using these error handling patterns (Clean Architecture layers, DI Container, Use Cases, Repositories), see `backend-engineer` skill**

## When to Engage

You should proactively assist when:

- Implementing error handling within bounded contexts
- Designing context-specific validation logic
- Creating context-specific exception types (no base classes)
- Implementing retry or recovery mechanisms per context
- User asks about error handling strategies
- Reviewing error handling without over-abstraction

## Modular Monolith Error Handling

### Context-Specific Errors (No Base Classes)

```typescript
// ❌ BAD: Base error class creates coupling
export abstract class DomainError extends Error {
  // Forces all contexts to use same error structure
}

// ✅ GOOD: Each context has its own errors
// contexts/auth/domain/errors/auth-validation.error.ts
export class AuthValidationError extends Error {
  constructor(message: string, public readonly field?: string) {
    super(message);
    this.name = "AuthValidationError";
  }
}

// contexts/tax/domain/errors/tax-calculation.error.ts
export class TaxCalculationError extends Error {
  constructor(message: string, public readonly ncmCode?: string) {
    super(message);
    this.name = "TaxCalculationError";
  }
}
```

### Error Handling Rules

1. **Each context owns its errors** - No shared error classes
2. **Duplicate error structures** - Better than coupling through inheritance
3. **Context-specific metadata** - Each error has relevant context data
4. **Simple over clever** - Avoid complex error hierarchies

## Core Principles

### 1. Use Exceptions, Not Return Codes

```typescript
// ✅ Good - Use exceptions with context
export class CreateUserUseCase {
  async execute(dto: CreateUserDto): Promise<User> {
    if (!this.isValidEmail(dto.email)) {
      throw new ValidationError("Invalid email format", {
        email: dto.email,
        field: "email",
      });
    }

    try {
      return await this.repository.save(user);
    } catch (error) {
      throw new DatabaseError("Failed to create user", {
        originalError: error,
        userId: user.id,
      });
    }
  }
}

// ❌ Bad - Return codes
export class CreateUserUseCase {
  async execute(dto: CreateUserDto): Promise<{
    success: boolean;
    user?: User;
    error?: string;
  }> {
    if (!this.isValidEmail(dto.email)) {
      return { success: false, error: "Invalid email" };
    }
    // Forces caller to check success flag everywhere
  }
}
```

### 2. Never Return Null for Errors

```typescript
// ✅ Good - Explicit optional with undefined
export class UserService {
  async findById(id: string): Promise<User | undefined> {
    return this.repository.findById(id);
  }

  // Or throw if must exist
  async getUserById(id: string): Promise<User> {
    const user = await this.repository.findById(id);
    if (!user) {
      throw new NotFoundError(`User ${id} not found`);
    }
    return user;
  }
}

// ❌ Bad - Returning null loses error context
export class UserService {
  async findById(id: string): Promise<User | null> {
    // Why null? Not found? Database error? Network error?
    return null;
  }
}
```

### 3. Provide Context with Exceptions

```typescript
// ✅ Good - Rich error context
export class ValidationError extends Error {
  constructor(
    message: string,
    public readonly context: Record<string, unknown>
  ) {
    super(message);
    this.name = "ValidationError";
  }
}

throw new ValidationError("Invalid email format", {
  email: dto.email,
  field: "email",
  rule: "email-format",
  attemptedAt: new Date().toISOString(),
});

// ❌ Bad - No context
throw new Error("Invalid");
```

## Exception Hierarchy

### Custom Domain Exceptions

```typescript
// Base domain exception
export abstract class DomainError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly context?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

// Specific domain exceptions
export class UserAlreadyExistsError extends DomainError {
  constructor(email: string) {
    super(`User with email ${email} already exists`, "USER_ALREADY_EXISTS", {
      email,
    });
  }
}

export class InvalidPasswordError extends DomainError {
  constructor(reason: string) {
    super("Password does not meet requirements", "INVALID_PASSWORD", {
      reason,
    });
  }
}

export class InsufficientPermissionsError extends DomainError {
  constructor(userId: string, resource: string, action: string) {
    super(
      `User ${userId} cannot ${action} ${resource}`,
      "INSUFFICIENT_PERMISSIONS",
      { userId, resource, action }
    );
  }
}
```

### Infrastructure Exceptions

```typescript
export class DatabaseError extends Error {
  constructor(
    message: string,
    public readonly originalError: unknown,
    public readonly query?: string
  ) {
    super(message);
    this.name = "DatabaseError";
  }
}

export class ExternalServiceError extends Error {
  constructor(
    public readonly service: string,
    message: string,
    public readonly statusCode?: number,
    public readonly originalError?: unknown
  ) {
    super(`${service}: ${message}`);
    this.name = "ExternalServiceError";
  }
}
```

## Result Pattern

For operations with expected failures:

```typescript
export type Result<T, E = Error> =
  | { success: true; value: T }
  | { success: false; error: E };

export class UserService {
  async findByEmail(email: string): Promise<Result<User, NotFoundError>> {
    const user = await this.repository.findByEmail(email);

    if (!user) {
      return {
        success: false,
        error: new NotFoundError("User not found"),
      };
    }

    return { success: true, value: user };
  }
}

// Usage
const result = await userService.findByEmail("user@example.com");

if (!result.success) {
  console.error("User not found:", result.error.message);
  return;
}

// TypeScript knows result.value is User here
const user = result.value;
```

### When to Use Result Pattern

**Use Result for:**

- Expected business failures (user not found, insufficient balance)
- Operations where failure is part of normal flow
- When caller needs to handle different failure types

**Use Exceptions for:**

- Unexpected errors (database connection lost, out of memory)
- Programming errors (invalid state, null pointer)
- Infrastructure failures

## Validation Patterns

### Input Validation at Boundaries

```typescript
import { z } from "zod";

// ✅ Good - Validate at system boundaries
const CreateUserSchema = z.object({
  email: z.string().email("Invalid email format"),
  password: z
    .string()
    .min(8, "Password must be at least 8 characters")
    .regex(/[A-Z]/, "Password must contain uppercase letter")
    .regex(/[0-9]/, "Password must contain number"),
  name: z
    .string()
    .min(2, "Name must be at least 2 characters")
    .max(100, "Name must be at most 100 characters"),
  age: z
    .number()
    .int("Age must be an integer")
    .min(18, "Must be at least 18 years old")
    .optional(),
});

export type CreateUserDto = z.infer<typeof CreateUserSchema>;

// In Hono controller
import { zValidator } from "@hono/zod-validator";

app.post("/users", zValidator("json", CreateUserSchema), async (c) => {
  const data = c.req.valid("json"); // Type-safe and validated
  const user = await createUserUseCase.execute(data);
  return c.json(user, 201);
});
```

### Domain Validation

```typescript
// ✅ Good - Validate in domain entities
export class Email {
  private constructor(private readonly value: string) {}

  static create(value: string): Result<Email, ValidationError> {
    if (!value) {
      return {
        success: false,
        error: new ValidationError("Email is required", { field: "email" }),
      };
    }

    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) {
      return {
        success: false,
        error: new ValidationError("Invalid email format", {
          email: value,
          field: "email",
        }),
      };
    }

    return { success: true, value: new Email(value) };
  }

  toString(): string {
    return this.value;
  }
}

// Usage
const emailResult = Email.create(dto.email);
if (!emailResult.success) {
  throw emailResult.error;
}
const email = emailResult.value;
```

## Error Recovery Patterns

### Retry Logic

```typescript
export async function withRetry<T>(
  operation: () => Promise<T>,
  options: {
    maxRetries: number;
    delayMs: number;
    shouldRetry?: (error: unknown) => boolean;
  }
): Promise<T> {
  const { maxRetries, delayMs, shouldRetry = () => true } = options;

  let lastError: unknown;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;

      if (attempt === maxRetries || !shouldRetry(error)) {
        break;
      }

      await new Promise((resolve) =>
        setTimeout(resolve, delayMs * (attempt + 1))
      );
    }
  }

  throw lastError;
}

// Usage
const user = await withRetry(() => userRepository.findById(id), {
  maxRetries: 3,
  delayMs: 1000,
  shouldRetry: (error) => error instanceof DatabaseError,
});
```

### Circuit Breaker

```typescript
export class CircuitBreaker {
  private failureCount = 0;
  private lastFailureTime?: number;
  private state: "CLOSED" | "OPEN" | "HALF_OPEN" = "CLOSED";

  constructor(
    private readonly failureThreshold: number,
    private readonly resetTimeoutMs: number
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === "OPEN") {
      if (Date.now() - (this.lastFailureTime || 0) > this.resetTimeoutMs) {
        this.state = "HALF_OPEN";
      } else {
        throw new Error("Circuit breaker is OPEN");
      }
    }

    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failureCount = 0;
    this.state = "CLOSED";
  }

  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.failureCount >= this.failureThreshold) {
      this.state = "OPEN";
    }
  }
}

// Usage
const breaker = new CircuitBreaker(5, 60000); // 5 failures, 60s timeout

const data = await breaker.execute(() => externalService.fetchData());
```

### Fallback Values

```typescript
export class ConfigService {
  async get<T>(key: string, fallback: T): Promise<T> {
    try {
      const value = await this.redis.get(key);
      return value ? JSON.parse(value) : fallback;
    } catch (error) {
      console.error(`Failed to get config ${key}:`, error);
      return fallback;
    }
  }
}

// Usage
const maxRetries = await configService.get("maxRetries", 3);
```

## Error Logging

### Structured Logging

```typescript
export interface LogContext {
  userId?: string;
  requestId?: string;
  operation?: string;
  [key: string]: unknown;
}

export class Logger {
  error(message: string, error: Error, context?: LogContext): void {
    console.error({
      level: "error",
      message,
      error: {
        name: error.name,
        message: error.message,
        stack: error.stack,
      },
      context,
      timestamp: new Date().toISOString(),
    });
  }

  warn(message: string, context?: LogContext): void {
    console.warn({
      level: "warn",
      message,
      context,
      timestamp: new Date().toISOString(),
    });
  }
}

// Usage
try {
  await userRepository.save(user);
} catch (error) {
  logger.error("Failed to save user", error as Error, {
    userId: user.id,
    operation: "createUser",
    requestId: context.requestId,
  });
  throw new DatabaseError("Failed to save user", error);
}
```

## HTTP Error Handling (Hono)

```typescript
import { Hono } from "hono";
import type { Context } from "hono";

const app = new Hono();

// Global error handler
app.onError((err, c) => {
  logger.error("Unhandled error", err, {
    path: c.req.path,
    method: c.req.method,
  });

  if (err instanceof ValidationError) {
    return c.json(
      {
        error: "Validation failed",
        message: err.message,
        context: err.context,
      },
      400
    );
  }

  if (err instanceof NotFoundError) {
    return c.json(
      {
        error: "Resource not found",
        message: err.message,
      },
      404
    );
  }

  if (err instanceof DomainError) {
    return c.json(
      {
        error: err.code,
        message: err.message,
        context: err.context,
      },
      400
    );
  }

  // Unknown error - don't leak details
  return c.json(
    {
      error: "Internal server error",
      message: "An unexpected error occurred",
    },
    500
  );
});
```

## Best Practices

### Do:

- ✅ Throw exceptions for exceptional situations
- ✅ Use Result pattern for expected failures
- ✅ Provide rich context in errors
- ✅ Validate at system boundaries
- ✅ Log errors with correlation IDs
- ✅ Implement retry for transient failures
- ✅ Use circuit breakers for external services
- ✅ Create domain-specific exception types

### Don't:

- ❌ Swallow exceptions silently
- ❌ Return null for errors
- ❌ Use exceptions for control flow
- ❌ Leak implementation details in error messages
- ❌ Throw generic Error instances
- ❌ Catch and re-throw without adding context
- ❌ Ignore errors in async operations

## Remember

- **Fail fast, fail loudly** - Don't hide errors
- **Context is king** - Provide rich error information
- **Recovery is better than failure** - Implement fallbacks when possible
- **Log for debugging** - Future you will thank you
