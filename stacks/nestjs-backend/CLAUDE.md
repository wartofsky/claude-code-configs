# NestJS Backend Project

## Tech Stack

- **Runtime**: Node.js 20+ LTS
- **Framework**: NestJS 10+ (Express adapter)
- **Language**: TypeScript 5.x (strict mode)
- **Database**: PostgreSQL 15+ with TypeORM
- **Validation**: class-validator + class-transformer
- **Auth**: Passport.js + JWT (@nestjs/passport, @nestjs/jwt)
- **Docs**: Swagger/OpenAPI (@nestjs/swagger)
- **Testing**: Jest + Supertest
- **Linting**: ESLint + Prettier
- **Containerization**: Docker + Docker Compose

## Project Structure

```
src/
├── main.ts                      # Bootstrap, global pipes, Swagger setup
├── app.module.ts                # Root module
├── config/                      # Configuration module (@nestjs/config)
│   ├── config.module.ts
│   ├── database.config.ts
│   └── jwt.config.ts
├── common/                      # Shared across all modules
│   ├── decorators/              # Custom decorators (@CurrentUser, @Public, @Roles)
│   ├── dto/                     # Shared DTOs (pagination, response wrappers)
│   ├── entities/                # Base entity (id, timestamps, soft delete)
│   ├── exceptions/              # Custom exception classes
│   ├── filters/                 # Exception filters (HttpExceptionFilter, AllExceptionsFilter)
│   ├── guards/                  # Auth guards (JwtAuthGuard, RolesGuard)
│   ├── interceptors/            # Transform, logging, timeout interceptors
│   ├── interfaces/              # Shared interfaces and types
│   ├── middleware/               # Logger, correlation ID middleware
│   └── pipes/                   # Custom validation pipes
├── modules/                     # Feature modules (domain-driven)
│   ├── auth/                    # Authentication module
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── dto/
│   │   ├── guards/
│   │   └── strategies/          # Passport strategies (JWT, Local)
│   ├── users/                   # Users module
│   │   ├── users.module.ts
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── users.repository.ts  # Custom repository (optional)
│   │   ├── dto/
│   │   ├── entities/
│   │   └── interfaces/
│   └── [feature]/               # Other feature modules follow same pattern
├── database/                    # Database configuration
│   ├── migrations/              # TypeORM migrations
│   ├── seeds/                   # Seed data
│   └── data-source.ts           # TypeORM DataSource for CLI
└── shared/                      # Shared services (mail, storage, queue)
    ├── mail/
    ├── storage/
    └── queue/

test/
├── e2e/                         # End-to-end tests
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
└── fixtures/                    # Shared test data and factories
```

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Files | kebab-case | `users.controller.ts`, `create-user.dto.ts` |
| Classes | PascalCase | `UsersController`, `CreateUserDto` |
| Methods | camelCase | `findAll()`, `createUser()` |
| Interfaces | PascalCase with `I` prefix (optional) | `IUserResponse` or `UserResponse` |
| Constants | UPPER_SNAKE_CASE | `MAX_LOGIN_ATTEMPTS` |
| Env vars | UPPER_SNAKE_CASE | `DATABASE_HOST`, `JWT_SECRET` |
| DB tables | snake_case plural | `users`, `user_roles` |
| DB columns | snake_case | `first_name`, `created_at` |
| Modules | PascalCase + Module suffix | `UsersModule` |
| Controllers | PascalCase + Controller suffix | `UsersController` |
| Services | PascalCase + Service suffix | `UsersService` |
| Guards | PascalCase + Guard suffix | `RolesGuard` |
| Pipes | PascalCase + Pipe suffix | `ParseIntPipe` |
| Interceptors | PascalCase + Interceptor suffix | `TransformInterceptor` |
| Filters | PascalCase + Filter suffix | `HttpExceptionFilter` |
| DTOs | PascalCase + Dto suffix | `CreateUserDto` |
| Entities | PascalCase + Entity suffix | `UserEntity` |

## Architecture Layers

```
Request → Middleware → Guards → Interceptors (pre) → Pipes → Controller → Service → Repository → Database
                                                                                         ↓
Response ← Interceptors (post) ← Exception Filters ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ←
```

### Layer Responsibilities

- **Controllers**: Handle HTTP requests, parameter extraction, response formatting. No business logic.
- **Services**: Business logic, orchestration, validation rules, external service calls.
- **Repositories**: Data access, queries, TypeORM operations. Optional custom repos for complex queries.
- **DTOs**: Request/response shape definition, validation decorators.
- **Entities**: Database schema definition, TypeORM decorators, column types.
- **Guards**: Authentication and authorization checks.
- **Interceptors**: Response transformation, logging, caching, timeout handling.
- **Pipes**: Data transformation and validation.
- **Filters**: Exception handling and error response formatting.

## Data Flow Patterns

### CRUD Operations
```
Controller (@Get/@Post/@Patch/@Delete)
  → Pipe (ValidationPipe auto-validates DTO)
    → Service (business logic + rules)
      → Repository (TypeORM query)
        → Entity (DB mapping)
```

### Authentication Flow
```
Request → JwtAuthGuard → JwtStrategy.validate() → @CurrentUser() → Controller
```

### Authorization Flow
```
Request → JwtAuthGuard → RolesGuard (@Roles decorator) → Controller
```

## API Response Format

### Success Response
```json
{
  "data": { },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

### Error Response
```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "details": [
    {
      "field": "email",
      "message": "email must be a valid email address"
    }
  ],
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/api/v1/users"
}
```

## Key Conventions

### Modules
- One module per domain/feature
- Export only what other modules need
- Use `forRoot()` / `forRootAsync()` for dynamic configuration
- Register global modules with `@Global()` sparingly

### DTOs & Validation
- Always use class-validator decorators on DTOs
- Separate Create, Update, and Response DTOs
- Use `PartialType()`, `PickType()`, `OmitType()`, `IntersectionType()` for DTO composition
- Enable `whitelist: true` and `forbidNonWhitelisted: true` in global ValidationPipe
- Use `@Transform()` for data sanitization (trim, lowercase)

### Entities
- Use a shared `BaseEntity` with `id`, `createdAt`, `updatedAt`, `deletedAt`
- Define relations explicitly with `@OneToMany`, `@ManyToOne`, `@ManyToMany`
- Use `@Index()` on frequently queried columns
- Prefer soft delete (`@DeleteDateColumn`) over hard delete

### Error Handling
- Throw NestJS built-in exceptions: `NotFoundException`, `BadRequestException`, `UnauthorizedException`, `ForbiddenException`, `ConflictException`
- Use custom exception filters for consistent error formatting
- Never expose internal errors or stack traces in production

### Configuration
- Use `@nestjs/config` with `.env` files
- Validate env vars with Joi or class-validator at startup
- Type config with interfaces and `registerAs()`
- Never hardcode secrets or connection strings

### API Versioning
- Use URI versioning: `/api/v1/`, `/api/v2/`
- Set global prefix: `app.setGlobalPrefix('api')`
- Enable versioning: `app.enableVersioning({ type: VersioningType.URI })`

### Swagger
- Use `@ApiTags()` on controllers
- Use `@ApiOperation()`, `@ApiResponse()` on endpoints
- Use `@ApiProperty()` on DTO properties
- Group by tags for readability

### Status Codes
| Action | Status Code |
|--------|------------|
| GET (list/single) | 200 OK |
| POST (create) | 201 Created |
| PATCH (update) | 200 OK |
| DELETE | 204 No Content |
| Validation error | 400 Bad Request |
| Authentication failure | 401 Unauthorized |
| Authorization failure | 403 Forbidden |
| Not found | 404 Not Found |
| Conflict (duplicate) | 409 Conflict |
| Server error | 500 Internal Server Error |

## Commands

```bash
# Development
npm run start:dev           # Watch mode
npm run start:debug         # Debug mode

# Database
npm run migration:generate  # Generate migration from entity changes
npm run migration:run       # Run pending migrations
npm run migration:revert    # Revert last migration
npm run seed                # Seed database

# Testing
npm run test                # Unit tests
npm run test:watch          # Watch mode
npm run test:cov            # Coverage report
npm run test:e2e            # End-to-end tests

# Build & Deploy
npm run build               # Production build
npm run start:prod          # Start production server
npm run lint                # ESLint check
npm run format              # Prettier format
```
