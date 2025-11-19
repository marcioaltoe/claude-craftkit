---
name: typescript-type-safety
description: TypeScript type safety including type guards and advanced type system features. **ALWAYS use when writing ANY TypeScript code (frontend AND backend)** to ensure strict type safety, avoid `any` types, and leverage the type system. Examples - "create function", "implement class", "define interface", "type guard", "discriminated union", "type narrowing", "conditional types", "handle unknown types".
---

You are an expert in TypeScript's type system and type safety. You guide developers to write type-safe code that leverages TypeScript's powerful type system to catch errors at compile time.

**For development workflow and quality gates (pre-commit checklist, bun commands), see `project-workflow` skill**

## When to Engage

You should proactively assist when:

- Working with `unknown` types in any context
- Implementing type guards for context-specific types
- Using discriminated unions within bounded contexts
- Implementing advanced TypeScript patterns without over-abstraction
- User asks about type safety or TypeScript features

## Modular Monolith Type Safety

### Context-Specific Types

```typescript
// ✅ GOOD: Each context owns its types
// contexts/auth/domain/types/user.types.ts
export interface AuthUser {
  id: string;
  email: string;
  isActive: boolean;
}

// contexts/tax/domain/types/calculation.types.ts
export interface TaxCalculation {
  ncmCode: string;
  rate: number;
  amount: number;
}

// ❌ BAD: Shared generic types that couple contexts
// shared/types/base.types.ts
export interface BaseEntity<T> {
  // NO! Creates coupling
  id: string;
  data: T;
}
```

## Core Type Safety Rules

### 1. NEVER Use `any`

```typescript
// ❌ FORBIDDEN - Disables type checking
function process(data: any) {
  return data.value; // No type safety at all
}

// ✅ CORRECT - Use unknown with type guards
function process(data: unknown): string {
  if (isProcessData(data)) {
    return data.value; // Type-safe access
  }
  throw new TypeError("Invalid data structure");
}

interface ProcessData {
  value: string;
}

function isProcessData(data: unknown): data is ProcessData {
  return (
    typeof data === "object" &&
    data !== null &&
    "value" in data &&
    typeof (data as ProcessData).value === "string"
  );
}
```

### 2. Use Proper Type Guards

```typescript
// ✅ Type predicate (narrows type)
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function isObject(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}

// ✅ Complex type guard
interface User {
  id: string;
  email: string;
  name: string;
}

function isUser(value: unknown): value is User {
  if (!isObject(value)) return false;

  return (
    "id" in value &&
    typeof value.id === "string" &&
    "email" in value &&
    typeof value.email === "string" &&
    "name" in value &&
    typeof value.name === "string"
  );
}

// Usage
function processUser(data: unknown): User {
  if (!isUser(data)) {
    throw new TypeError("Invalid user data");
  }
  // TypeScript knows data is User here
  return data;
}
```

## Discriminated Unions

```typescript
// ✅ Discriminated unions for polymorphic data
type PaymentMethod =
  | {
      type: "credit_card";
      cardNumber: string;
      expiryDate: string;
      cvv: string;
    }
  | {
      type: "paypal";
      email: string;
    }
  | {
      type: "bank_transfer";
      accountNumber: string;
      routingNumber: string;
    };

function processPayment(method: PaymentMethod, amount: number): void {
  switch (method.type) {
    case "credit_card":
      // TypeScript knows method has cardNumber, expiryDate, cvv
      console.log(`Charging ${amount} to card ${method.cardNumber}`);
      break;

    case "paypal":
      // TypeScript knows method has email
      console.log(`Charging ${amount} to PayPal ${method.email}`);
      break;

    case "bank_transfer":
      // TypeScript knows method has accountNumber, routingNumber
      console.log(
        `Charging ${amount} via bank transfer ${method.accountNumber}`
      );
      break;

    default:
      // Exhaustiveness check
      const _exhaustive: never = method;
      throw new Error(`Unhandled payment method: ${_exhaustive}`);
  }
}
```

## Conditional Types

```typescript
// ✅ Conditional types for flexible APIs
type ResponseData<T> = T extends { id: string }
  ? { success: true; data: T }
  : never;

type User = { id: string; name: string };
type UserResponse = ResponseData<User>; // { success: true; data: User }

// ✅ Extract promise type
type Awaited<T> = T extends Promise<infer U> ? U : T;

type UserPromise = Promise<User>;
type UserType = Awaited<UserPromise>; // User

// ✅ Extract function return type
type ReturnType<T> = T extends (...args: unknown[]) => infer R ? R : never;

function getUser(): User {
  return { id: "1", name: "John" };
}

type UserFromFunction = ReturnType<typeof getUser>; // User
```

## Mapped Types

```typescript
// ✅ Make all properties optional
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type User = {
  id: string;
  email: string;
  name: string;
};

type PartialUser = Partial<User>;
// { id?: string; email?: string; name?: string }

// ✅ Make all properties readonly
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type ReadonlyUser = Readonly<User>;

// ✅ Pick specific properties
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type UserPreview = Pick<User, "id" | "name">;
// { id: string; name: string }

// ✅ Omit specific properties
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;

type UserWithoutId = Omit<User, "id">;
// { email: string; name: string }
```

## Template Literal Types

```typescript
// ✅ Type-safe string patterns
type EventName = "user" | "order" | "payment";
type EventAction = "created" | "updated" | "deleted";

type Event = `${EventName}:${EventAction}`;
// 'user:created' | 'user:updated' | 'user:deleted' |
// 'order:created' | 'order:updated' | 'order:deleted' |
// 'payment:created' | 'payment:updated' | 'payment:deleted'

function emitEvent(event: Event): void {
  console.log(`Emitting ${event}`);
}

emitEvent("user:created"); // ✅ OK
emitEvent("user:invalid"); // ❌ Type error
```

## Function Overloads

```typescript
// ✅ Function overloads for different input/output types
function getValue(key: "count"): number;
function getValue(key: "name"): string;
function getValue(key: "isActive"): boolean;
function getValue(key: string): unknown {
  // Implementation
  const values: Record<string, unknown> = {
    count: 42,
    name: "John",
    isActive: true,
  };
  return values[key];
}

const count = getValue("count"); // Type is number
const name = getValue("name"); // Type is string
const isActive = getValue("isActive"); // Type is boolean
```

## Const Assertions

```typescript
// ✅ Const assertions for literal types
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
} as const;

// config.apiUrl type is 'https://api.example.com' (literal)
// config.timeout type is 5000 (literal)
// config is readonly

// ✅ Array as const
const colors = ["red", "green", "blue"] as const;
// Type is readonly ['red', 'green', 'blue']

type Color = (typeof colors)[number];
// Type is 'red' | 'green' | 'blue'
```

## Utility Types

### NonNullable

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>; // string
```

### Extract and Exclude

```typescript
type Extract<T, U> = T extends U ? T : never;
type Exclude<T, U> = T extends U ? never : T;

type Status = "pending" | "approved" | "rejected" | "cancelled";

type PositiveStatus = Extract<Status, "approved" | "pending">;
// 'approved' | 'pending'

type NegativeStatus = Exclude<Status, "approved" | "pending">;
// 'rejected' | 'cancelled'
```

### Record

```typescript
type Record<K extends keyof unknown, T> = {
  [P in K]: T;
};

type UserRoles = "admin" | "user" | "guest";
type Permissions = Record<UserRoles, string[]>;
// {
//   admin: string[];
//   user: string[];
//   guest: string[];
// }

const permissions: Permissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"],
};
```

## Type Narrowing

### typeof Guards

```typescript
function process(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase(); // value is string here
  }
  return value.toFixed(2); // value is number here
}
```

### instanceof Guards

```typescript
class User {
  constructor(public name: string) {}
}

class Admin extends User {
  constructor(name: string, public level: number) {
    super(name);
  }
}

function greet(user: User | Admin): string {
  if (user instanceof Admin) {
    return `Hello Admin ${user.name}, level ${user.level}`;
  }
  return `Hello ${user.name}`;
}
```

### in Operator

```typescript
type Dog = { bark: () => void };
type Cat = { meow: () => void };

function makeSound(animal: Dog | Cat): void {
  if ("bark" in animal) {
    animal.bark(); // animal is Dog here
  } else {
    animal.meow(); // animal is Cat here
  }
}
```

## TypeScript Configuration

```json
{
  "compilerOptions": {
    // Strict type checking
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,

    // Module resolution
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,

    // Interop
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,

    // Other
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,

    // Bun-specific
    "types": ["bun-types"],
    "target": "ES2022",
    "lib": ["ES2022"]
  }
}
```

## Best Practices

### Do:

- ✅ Use `unknown` instead of `any`
- ✅ Implement type guards for runtime checks
- ✅ Leverage discriminated unions for polymorphism
- ✅ Enable strict mode in tsconfig.json
- ✅ Use const assertions for literal types
- ✅ Implement exhaustiveness checks in switch statements

### Don't:

- ❌ Use `any` type
- ❌ Use type assertions (as) without validation
- ❌ Disable strict mode
- ❌ Use `@ts-ignore` or `@ts-expect-error` without good reason
- ❌ Mix types and interfaces unnecessarily
- ❌ Create overly complex type gymnastics

## Common Type Errors and Solutions

### Error: Object is possibly 'null' or 'undefined'

```typescript
// ❌ Error
function getName(user: User | null): string {
  return user.name; // Error: Object is possibly 'null'
}

// ✅ Solution: Check for null
function getName(user: User | null): string {
  if (!user) {
    throw new Error("User is null");
  }
  return user.name; // OK
}

// ✅ Or use optional chaining
function getName(user: User | null): string | undefined {
  return user?.name;
}
```

### Error: Type 'X' is not assignable to type 'Y'

```typescript
// ❌ Error
interface User {
  id: string;
  name: string;
}

const user = {
  id: "1",
  name: "John",
  extra: "field",
};

const typedUser: User = user; // OK (structural typing)

// ✅ Use type annotation or assertion when needed
const exactUser: User = {
  id: "1",
  name: "John",
  // extra: 'field', // Error if uncommented
};
```

## Remember

- **Type safety catches bugs at compile time** - Invest in good types
- **unknown > any** - Always use unknown for truly unknown types
- **Type guards are your friends** - Use them liberally
- **Strict mode is mandatory** - Never disable it
