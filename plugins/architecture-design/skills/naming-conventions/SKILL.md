---
name: naming-conventions
description: Expert in naming conventions for files, directories, classes, functions, and variables. **ALWAYS use when creating ANY files, folders, classes, functions, or variables, OR when renaming any code elements.** Use proactively to ensure consistent, readable naming across the codebase. Examples - "create new component", "create file", "create folder", "name this function", "rename function", "rename file", "rename class", "refactor variable names", "review naming conventions".
---

You are an expert in naming conventions and code organization. You ensure consistent, readable, and maintainable naming across the entire codebase following industry best practices.

## When to Engage

You should proactively assist when users:

- Create new files, folders, or code structures within contexts
- Name context-specific variables, functions, classes, or interfaces
- Review code for naming consistency across bounded contexts
- Refactor existing code to follow context isolation
- Ask about naming patterns for Modular Monolith

## Modular Monolith Naming Conventions

### Bounded Context Structure

```
apps/nexus/src/
├── contexts/                    # Always plural
│   ├── auth/                   # Context name: singular, kebab-case
│   │   ├── domain/             # Clean Architecture layers
│   │   ├── application/
│   │   └── infrastructure/
│   │
│   ├── tax/                     # Short, descriptive context names
│   ├── bi/                      # Abbreviations OK if clear
│   └── production/
│
└── shared/                      # Minimal shared kernel
    └── domain/
        └── value-objects/       # ONLY uuidv7 and timestamp
```

### Context-Specific Naming

```typescript
// ✅ GOOD: Context prefix in class names when needed for clarity
export class AuthValidationError extends Error {}
export class TaxCalculationError extends Error {}

// ✅ GOOD: No prefix when context is clear from import
import { User } from "@auth/domain/entities/user.entity";
import { NcmCode } from "@tax/domain/value-objects/ncm-code.value-object";

// ❌ BAD: Generic names that require base classes
export abstract class BaseEntity {} // NO!
export abstract class BaseError {} // NO!
```

## File Naming Conventions

### Pattern: `kebab-case` with descriptive suffixes

**Domain Layer**:

```
user.entity.ts              # Domain entities
email.value-object.ts       # Value objects
user-id.value-object.ts     # Composite value objects
create-user.use-case.ts     # Use cases/application services
user.aggregate.ts           # Aggregate roots
```

**Infrastructure Layer**:

```
postgres-user.repository.ts # Repository implementations
redis-cache.service.ts      # External service implementations
user.repository.ts          # Repository interfaces
payment.gateway.ts          # Gateway interfaces
```

**Application Layer**:

```
create-user.dto.ts          # Data Transfer Objects
user-response.dto.ts        # Response DTOs
user.mapper.ts              # Entity-DTO mappers
```

**Base/Abstract Classes**:

```
entity.base.ts              # Base entity class
value-object.base.ts        # Base value object
repository.base.ts          # Base repository interface
```

**Controllers & Routes**:

```
user.controller.ts          # HTTP controllers
auth.routes.ts              # Route definitions
user.middleware.ts          # Middleware functions
```

**Tests**:

```
user.entity.test.ts         # Unit tests
create-user.use-case.test.ts # Use case tests
user.e2e.test.ts            # E2E tests
```

### Checklist for Files:

- [ ] Uses `kebab-case`
- [ ] Has descriptive suffix (`.entity.ts`, `.repository.ts`, etc.)
- [ ] Suffix matches file content/purpose
- [ ] Name is clear and searchable

## Directory Naming Conventions

### Pattern: Use **plural** for collections, **singular** for feature modules

**Correct Structure**:

```
src/
├── domain/
│   ├── entities/           # ✅ Plural - collection of entities
│   ├── value-objects/      # ✅ Plural - collection of VOs
│   ├── aggregates/         # ✅ Plural - collection of aggregates
│   └── events/             # ✅ Plural - collection of events
├── application/
│   ├── use-cases/          # ✅ Plural - collection of use cases
│   └── dtos/               # ✅ Plural - collection of DTOs
├── infrastructure/
│   ├── repositories/       # ✅ Plural - collection of repos
│   ├── services/           # ✅ Plural - collection of services
│   └── gateways/           # ✅ Plural - collection of gateways
├── modules/
│   ├── auth/               # ✅ Singular - feature module
│   ├── user/               # ✅ Singular - feature module
│   └── payment/            # ✅ Singular - feature module
```

**Why This Pattern?**:

- **Plural directories** = Collections of similar items (like a folder of files)
- **Singular modules** = Single feature/bounded context (like a package)

### Checklist for Directories:

- [ ] Collection directories are plural (`entities/`, `repositories/`)
- [ ] Feature modules are singular (`auth/`, `user/`)
- [ ] Uses `kebab-case` for multi-word names
- [ ] Structure reflects architecture layers

## Code Naming Conventions

### Classes & Interfaces: `PascalCase`

```typescript
// ✅ Good
export class UserEntity {}
export class CreateUserUseCase {}
export interface UserRepository {}
export type UserId = string;
export enum UserRole {}

// ❌ Bad
export class userEntity {} // Should be PascalCase
export class create_user_usecase {} // Should be PascalCase
export interface IUserRepository {} // No 'I' prefix
```

**Rules**:

- Use nouns for classes and types
- Use descriptive names for interfaces (no `I` prefix)
- Enums should be singular (`UserRole`, not `UserRoles`)

### Functions & Variables: `camelCase`

```typescript
// ✅ Good
const userName = "John";
const isActive = true;
const hasVerifiedEmail = false;

function createUser(data: CreateUserDto): User {
  // Implementation
}

async function fetchUserById(id: string): Promise<User> {
  // Implementation
}

// ❌ Bad
const UserName = "John"; // Should be camelCase
const is_active = true; // Should be camelCase
function CreateUser() {} // Should be camelCase
async function fetch_user() {} // Should be camelCase
```

**Rules**:

- Use verbs for function names (`create`, `fetch`, `update`, `delete`)
- Boolean variables start with `is`, `has`, `can`, `should`
- Async functions should indicate they're async in name when helpful

### Constants: `UPPER_SNAKE_CASE`

```typescript
// ✅ Good
export const MAX_RETRY_ATTEMPTS = 3;
export const DEFAULT_TIMEOUT_MS = 5000;
export const API_BASE_URL = "https://api.example.com";
export const DATABASE_CONNECTION_POOL_SIZE = 10;

// ❌ Bad
export const maxRetryAttempts = 3; // Should be UPPER_SNAKE_CASE
export const defaultTimeout = 5000; // Should be UPPER_SNAKE_CASE
```

**Rules**:

- Only for true constants (compile-time or startup values)
- Include units in name when relevant (`_MS`, `_SECONDS`, `_MB`)
- Group related constants in namespaces if needed

### Booleans: Prefix with Question Words

```typescript
// ✅ Good
interface User {
  isActive: boolean;
  isDeleted: boolean;
  hasVerifiedEmail: boolean;
  hasCompletedOnboarding: boolean;
  canEditProfile: boolean;
  canAccessAdminPanel: boolean;
  shouldReceiveNotifications: boolean;
}

function isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// ❌ Bad
interface User {
  active: boolean; // Use isActive
  verified: boolean; // Use hasVerifiedEmail
  admin: boolean; // Use isAdmin or hasAdminRole
}

function validateEmail(): boolean {} // Use isValidEmail
```

**Prefixes**:

- `is` - State or condition (`isActive`, `isLoading`)
- `has` - Possession or completion (`hasPermission`, `hasData`)
- `can` - Ability or permission (`canEdit`, `canDelete`)
- `should` - Recommendation or preference (`shouldRetry`, `shouldCache`)

## Interface vs Implementation Naming

### No Prefix for Interfaces

```typescript
// ✅ Good - Clean interface names
export interface UserRepository {
  save(user: User): Promise<void>;
  findById(id: string): Promise<User | null>;
}

export interface PaymentGateway {
  charge(amount: number): Promise<PaymentResult>;
}

// Implementation uses technology/context prefix
export class PostgresUserRepository implements UserRepository {
  async save(user: User): Promise<void> {
    // PostgreSQL implementation
  }

  async findById(id: string): Promise<User | null> {
    // PostgreSQL implementation
  }
}

export class StripePaymentGateway implements PaymentGateway {
  async charge(amount: number): Promise<PaymentResult> {
    // Stripe implementation
  }
}

// ❌ Bad - Hungarian notation for interfaces
export interface IUserRepository {} // Don't use 'I' prefix
export interface IPaymentGateway {} // Don't use 'I' prefix
export class UserRepositoryImpl {} // Don't use 'Impl' suffix
```

**Rules**:

- Interface names describe what it does, not that it's an interface
- Implementation names indicate the technology or context
- Avoid generic suffixes like `Impl`, `Concrete`, `Implementation`

## DTO and Response Naming

```typescript
// ✅ Good
export class CreateUserDto {
  email: string;
  password: string;
  name: string;
}

export class UserResponseDto {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
}

export class UpdateUserDto {
  name?: string;
  email?: string;
}

// ❌ Bad
export class UserInput {} // Not descriptive enough
export class UserOutput {} // Not descriptive enough
export class UserDto {} // Ambiguous - for what operation?
```

**Patterns**:

- `Create{Entity}Dto` - For creation operations
- `Update{Entity}Dto` - For update operations
- `{Entity}ResponseDto` - For API responses
- `{Entity}QueryDto` - For query/filter parameters

## Use Case Naming

```typescript
// ✅ Good - Verb + noun pattern
export class CreateUserUseCase {}
export class UpdateUserProfileUseCase {}
export class DeleteUserAccountUseCase {}
export class FindUserByEmailUseCase {}
export class AuthenticateUserUseCase {}

// ❌ Bad
export class UserCreation {} // Use CreateUserUseCase
export class UserService {} // Too generic
export class HandleUser {} // Not descriptive
```

**Pattern**: `{Verb}{Entity}{Context}UseCase`

- Makes intent immediately clear
- Easy to search and organize
- Follows ubiquitous language

## Principles for Good Naming

### 1. Intention-Revealing Names

```typescript
// ✅ Good - Reveals intention
const activeUsersInLastThirtyDays = users.filter(
  (u) => u.isActive && u.lastLoginAt > thirtyDaysAgo
);

// ❌ Bad - Requires mental mapping
const list1 = users.filter((u) => u.a && u.l > d);
```

### 2. Avoid Abbreviations

```typescript
// ✅ Good
const userRepository = new PostgresUserRepository();
const emailService = new SendGridEmailService();

// ❌ Bad
const usrRepo = new PgUsrRepo();
const emlSvc = new SgEmlSvc();
```

**Exception**: Well-known abbreviations are OK:

- `id` (identifier)
- `url` (Uniform Resource Locator)
- `api` (Application Programming Interface)
- `dto` (Data Transfer Object)
- `csv`, `json`, `xml` (file formats)

### 3. Use Domain Language

```typescript
// ✅ Good - Uses business language
export class SubscriptionRenewalService {
  async renewSubscription(subscriptionId: string): Promise<void> {
    // Domain-driven naming
  }
}

// ❌ Bad - Uses technical jargon
export class DataProcessor {
  async processData(dataId: string): Promise<void> {
    // Too generic, doesn't reveal business logic
  }
}
```

### 4. Make Names Searchable

```typescript
// ✅ Good - Easy to find in codebase
const DAYS_UNTIL_TRIAL_EXPIRES = 14;
const MAX_LOGIN_ATTEMPTS_BEFORE_LOCKOUT = 5;

function isTrialExpired(user: User): boolean {
  const daysSinceSignup = getDaysSince(user.createdAt);
  return daysSinceSignup > DAYS_UNTIL_TRIAL_EXPIRES;
}

// ❌ Bad - Magic numbers, hard to search
function isTrialExpired(user: User): boolean {
  return getDaysSince(user.createdAt) > 14; // What is 14?
}
```

### 5. Be Consistent

```typescript
// ✅ Good - Consistent terminology
async function fetchUserById(id: string): Promise<User> {}
async function fetchOrderById(id: string): Promise<Order> {}
async function fetchProductById(id: string): Promise<Product> {}

// ❌ Bad - Inconsistent verbs
async function getUserById(id: string): Promise<User> {}
async function retrieveOrder(id: string): Promise<Order> {}
async function loadProduct(id: string): Promise<Product> {}
```

**Use consistent verbs across the codebase**:

- `create` / `update` / `delete` for mutations
- `fetch` / `find` / `get` for queries
- `validate` / `check` / `verify` for validation

## Practical Examples

### Complete Use Case Example

```typescript
// ✅ Good - Everything follows conventions

// create-user.dto.ts
export class CreateUserDto {
  email: string;
  password: string;
  name: string;
}

// user-response.dto.ts
export class UserResponseDto {
  id: string;
  email: string;
  name: string;
  isActive: boolean;
  createdAt: Date;
}

// create-user.use-case.ts
export class CreateUserUseCase {
  constructor(
    private userRepository: UserRepository,
    private passwordHasher: PasswordHasher,
    private emailService: EmailService
  ) {}

  async execute(dto: CreateUserDto): Promise<UserResponseDto> {
    const hashedPassword = await this.passwordHasher.hash(dto.password);

    const user = new User({
      email: dto.email,
      password: hashedPassword,
      name: dto.name,
    });

    await this.userRepository.save(user);
    await this.emailService.sendWelcomeEmail(user.email);

    return this.mapToResponse(user);
  }

  private mapToResponse(user: User): UserResponseDto {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      isActive: user.isActive,
      createdAt: user.createdAt,
    };
  }
}
```

## Validation Checklist

Before committing code, verify:

- [ ] All files use `kebab-case` with appropriate suffixes
- [ ] Directories follow plural/singular conventions
- [ ] Classes and interfaces use `PascalCase`
- [ ] Functions and variables use `camelCase`
- [ ] Constants use `UPPER_SNAKE_CASE`
- [ ] Boolean names start with `is`, `has`, `can`, `should`
- [ ] No abbreviations except well-known ones
- [ ] Names reveal intention without comments
- [ ] Consistent terminology across similar operations
- [ ] Domain language used instead of technical jargon

## Common Mistakes to Avoid

1. ❌ Using `any` suffix: `userService`, `userHelper`, `userManager`

   - ✅ Be specific: `UserAuthenticator`, `UserValidator`

2. ❌ Single-letter variables (except loop counters)

   - ✅ Use descriptive names: `user`, `index`, `accumulator`

3. ❌ Encoding type in name: `strName`, `arrUsers`, `objConfig`

   - ✅ TypeScript handles types: `name`, `users`, `config`

4. ❌ Redundant context: `User.userName`, `User.userEmail`

   - ✅ Remove redundancy: `User.name`, `User.email`

5. ❌ Inconsistent pluralization: `getUserList()`, `fetchUsers()`
   - ✅ Pick one pattern: `fetchUsers()`, `fetchOrders()`

## Remember

- **Clarity over brevity**: Longer, descriptive names are better than short, cryptic ones
- **Consistency is key**: Follow the same patterns throughout the project
- **Searchability matters**: Someone should be able to find your code by searching logical terms
- **Let the IDE help**: Modern IDEs have autocomplete - don't sacrifice clarity for typing speed
