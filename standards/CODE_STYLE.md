# CODE STYLE STANDARDS 📏

> **TypeScript strict, functional core, imperative shell.**  
> Código legible, testeable, mantenible por cualquier ingeniero DARUMA.

---

## 🎯 Principios Fundamentales

| Principio | Regla |
|-----------|-------|
| **TypeScript Strict** | `strict: true` siempre. No `any`. No `@ts-ignore` sin justificación en comentario. |
| **Functional Core** | Lógica de negocio = funciones puras. Sin side effects. Fácil de testear. |
| **Imperative Shell** | Side effects (IO, DB, HTTP, clock) solo en capa externa. Inyectados como dependencias. |
| **Single Responsibility** | Una función/clase = una responsabilidad. Si tiene "y" en el nombre, dividir. |
| **Explicit over Implicit** | Tipos explícitos en APIs públicas. No magic strings. Constantes nombradas. |
| **Fail Fast** | Validar inputs temprano. Lanzar errores descriptivos. No silent failures. |

---

## 📝 TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*", "tests/**/*"],
  "exclude": ["node_modules", "dist", "build"]
}
```

---

## 🏗️ Arquitectura de Carpetas

```
src/
├── domain/           # Lógica de negocio pura (functional core)
│   ├── entities/     # Tipos de dominio, value objects
│   ├── services/     # Reglas de negocio, casos de uso
│   ├── repositories/ # Interfaces (ports) - NO implementaciones
│   └── events/       # Domain events
├── application/      # Orquestación (imperative shell)
│   ├── use-cases/    # Application services
│   ├── dto/          # Data Transfer Objects
│   └── ports/        # Interfaces para adapters
├── infrastructure/   # Implementaciones concretas (adapters)
│   ├── http/         # Clients HTTP, webhooks
│   ├── database/     # Supabase, Redis, repositories impl
│   ├── messaging/    # Queues, event bus
│   ├── scheduler/    # Cron jobs, triggers
│   └── monitoring/   # Logs, metrics, tracing
├── interfaces/       # Entry points
│   ├── api/          # REST endpoints, GraphQL
│   ├── cli/          # Comandos CLI
│   └── workers/      # Background jobs
└── shared/           # Utilidades cross-cutting
    ├── types/        # Tipos compartidos
    ├── errors/       # Error classes personalizados
    ├── validation/   # Zod schemas
    └── utils/        # Helpers puros
```

---

## 🔤 Naming Conventions

| Elemento | Convención | Ejemplo |
|----------|------------|---------|
| **Archivos** | kebab-case | `user-service.ts`, `http-client.ts` |
| **Clases/Interfaces** | PascalCase | `UserService`, `HttpClient`, `IUserRepository` |
| **Funciones/Variables** | camelCase | `getUserById`, `httpClient`, `MAX_RETRIES` |
| **Constantes** | UPPER_SNAKE_CASE | `DEFAULT_TIMEOUT_MS`, `SUPPORTED_LOCALES` |
| **Tipos/Interfaces** | PascalCase + sufijo | `UserDto`, `HttpResponse<T>`, `UserRepository` |
| **Enums** | PascalCase singular | `UserStatus`, `HttpMethod` |
| **Tests** | `*.test.ts` / `*.spec.ts` | `user-service.test.ts`, `http-client.spec.ts` |
| **Git branches** | `type/scope-description` | `feat/n8n-webhook-retry`, `fix/scraper-proxy-rotation` |

---

## 🧱 Patrones Obligatorios

### 1. Result Type (No Exceptions para Control de Flujo)

```typescript
// shared/result.ts
export type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

export const ok = <T>(value: T): Result<T, never> => ({ ok: true, value });
export const err = <E>(error: E): Result<never, E> => ({ ok: false, error });

// Uso
async function fetchUser(id: string): Promise<Result<User, UserNotFoundError>> {
  const user = await userRepo.findById(id);
  if (!user) return err(new UserNotFoundError(id));
  return ok(user);
}

// Consumidor
const result = await fetchUser(id);
if (!result.ok) {
  // Manejo explícito del error
  logger.warn({ error: result.error }, 'User not found');
  return;
}
// result.value es User (type narrowing)
```

### 2. Dependency Injection (Manual, Simple)

```typescript
// domain/repositories/user-repository.ts
export interface IUserRepository {
  findById(id: string): Promise<Result<User, UserNotFoundError>>;
  save(user: User): Promise<Result<void, SaveError>>;
}

// infrastructure/database/supabase-user-repository.ts
export class SupabaseUserRepository implements IUserRepository {
  constructor(private readonly supabase: SupabaseClient) {}
  
  async findById(id: string): Promise<Result<User, UserNotFoundError>> {
    // Implementación concreta
  }
}

// application/use-cases/get-user.ts
export class GetUserUseCase {
  constructor(private readonly userRepo: IUserRepository) {}
  
  async execute(id: string): Promise<Result<UserDto, UserNotFoundError>> {
    const result = await this.userRepo.findById(id);
    if (!result.ok) return err(result.error);
    return ok(toDto(result.value));
  }
}

// Composition root (main.ts)
const supabase = createSupabaseClient();
const userRepo = new SupabaseUserRepository(supabase);
const getUser = new GetUserUseCase(userRepo);
```

### 3. Zod Schemas para Validación en Fronteras

```typescript
// shared/validation/user-schema.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  role: z.enum(['admin', 'user', 'viewer']).default('user'),
  metadata: z.record(z.unknown()).optional(),
});

export type CreateUserInput = z.infer<typeof CreateUserSchema>;

// Uso en boundary (API, CLI, Worker)
export async function createUserHandler(input: unknown) {
  const parsed = CreateUserSchema.safeParse(input);
  if (!parsed.success) {
    return err(new ValidationError(parsed.error.flatten()));
  }
  // parsed.data es CreateUserInput (type-safe)
  return await createUserUseCase.execute(parsed.data);
}
```

### 4. Error Classes Personalizadas

```typescript
// shared/errors/domain-errors.ts
export abstract class DomainError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
  
  constructor(message: string, public readonly context?: Record<string, unknown>) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class UserNotFoundError extends DomainError {
  readonly code = 'USER_NOT_FOUND';
  readonly statusCode = 404;
  constructor(id: string) {
    super(`User not found: ${id}`, { userId: id });
  }
}

export class ValidationError extends DomainError {
  readonly code = 'VALIDATION_ERROR';
  readonly statusCode = 400;
  constructor(public readonly issues: z.ZodIssue[]) {
    super('Validation failed', { issues });
  }
}
```

---

## 🚫 Anti-Patrones (Prohibidos)

| Anti-Patrón | Por Qué | Alternativa |
|-------------|---------|-------------|
| `any` type | Pierde type safety | `unknown` + type guards / Zod |
| `@ts-ignore` | Oculta bugs reales | Fixear el tipo o refactor |
| Try-catch para control de flujo | Exceptions son para excepciones | `Result<T, E>` type |
| Mutación de estado global | Imposible de testear/razonar | Estado inmutable + DI |
| Clases anémicas (solo getters/setters) | No es OOP, es data structures | Tipos + funciones puras |
| Magic strings/numbers | Frágil, no refactorizable | Constantes tipadas / Enums |
| Comentarios que explican "qué" | El código debe ser autoexplicativo | Nombres claros + tipos |
| Tests que solo verifican implementación | Frágiles, no dan confianza | Tests de comportamiento/contrato |

---

## 🔧 Tooling Obligatorio

```json
// package.json scripts
{
  "scripts": {
    "lint": "eslint src/**/*.ts --max-warnings 0",
    "typecheck": "tsc --noEmit",
    "test": "vitest run --coverage",
    "test:watch": "vitest",
    "test:e2e": "playwright test",
    "format": "prettier --write \"**/*.{ts,json,md}\"",
    "security": "npm audit --audit-level=high && codeql analyze",
    "prepare": "husky install"
  },
  "devDependencies": {
    "typescript": "^5.4",
    "eslint": "^8.57",
    "@typescript-eslint/eslint-plugin": "^7.0",
    "@typescript-eslint/parser": "^7.0",
    "prettier": "^3.2",
    "vitest": "^1.3",
    "@vitest/coverage-v8": "^1.3",
    "playwright": "^1.42",
    "zod": "^3.22",
    "husky": "^9.0",
    "lint-staged": "^15.2"
  }
}
```

---

## 📋 Code Review Checklist

- [ ] TypeScript strict pasa sin errores
- [ ] ESLint sin warnings
- [ ] Tests unitarios + integración pasan
- [ ] Coverage >90% en código nuevo
- [ ] No `any`, no `@ts-ignore` sin justificación
- [ ] Nombres claros, sin abreviaturas crípticas
- [ ] Funciones <50 líneas, clases <200 líneas
- [ ] Error handling explícito con `Result<T, E>`
- [ ] Validación Zod en todas las fronteras
- [ ] ADR si hay decisión arquitectónica nueva
- [ ] Docs actualizadas (README, CHANGELOG, API docs)

---

> **"El código se lee 10x más de lo que se escribe. Escríbelo para el próximo ingeniero que lo mantenga (que puede ser tú en 6 meses)."**