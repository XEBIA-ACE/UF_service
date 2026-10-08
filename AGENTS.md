# AGENTS.md — Unified Financial Management Service

## 1. Stack

| Layer | Technology | Role |
|---|---|---|
| Backend API | Java 21 + Spring Boot 3.x | Core business logic, account/credit card aggregation, auth |
| API Gateway / BFF | Node.js 20 LTS + Express 5.x | Backend-for-frontend, request routing, session management |
| Frontend | React.js 18 + TypeScript | Dashboard UI, customizable views, user interaction |
| Auth | Spring Security + JWT (JJWT 0.12.x) | Authentication, authorization, token issuance |
| Database | PostgreSQL 16 | Persistent storage for accounts, cards, users, preferences |
| Cache | Redis 7 | Session cache, rate limiting, financial data TTL cache |
| Build (Java) | Maven 3.9.x | Dependency management, lifecycle, packaging |
| Build (Node/React) | pnpm 9.x + Vite 5.x | Fast installs, frontend bundling |
| Testing (Java) | JUnit 5, Mockito, Testcontainers | Unit, integration, and DB-level tests |
| Testing (Node) | Jest 29 + Supertest | Unit and HTTP integration tests for BFF |
| Testing (React) | Vitest + React Testing Library | Component and hook tests |
| Containerisation | Docker + Docker Compose | Local dev and CI environment |
| CI | GitHub Actions | Lint, test, build, image push pipeline |
| Code Quality | Checkstyle, SpotBugs (Java); ESLint + Prettier (JS/TS) | Enforce style and catch bugs |

---

## 2. Project Structure

```
unified-financial-management/
│
├── AGENTS.md                          # This file — agent scaffold spec
├── tasks.md                           # Agent-generated task tracker (created before coding)
├── docker-compose.yml                 # Orchestrates all services for local dev
├── docker-compose.test.yml            # Isolated compose for CI integration tests
├── .env.example                       # Template for required environment variables
├── .gitignore
├── README.md
│
├── api/                               # Spring Boot service (Java)
│   ├── Dockerfile
│   ├── pom.xml                        # Maven build descriptor
│   ├── checkstyle.xml                 # Checkstyle rule set
│   ├── spotbugs-exclude.xml           # SpotBugs filter file
│   └── src/
│       ├── main/
│       │   ├── java/com/unifiedfin/api/
│       │   │   ├── UnifiedFinApiApplication.java   # Spring Boot entry point
│       │   │   ├── config/
│       │   │   │   ├── SecurityConfig.java          # Spring Security + JWT filter chain
│       │   │   │   ├── RedisConfig.java             # Redis connection factory
│       │   │   │   └── OpenApiConfig.java           # Springdoc OpenAPI configuration
│       │   │   ├── auth/
│       │   │   │   ├── controller/AuthController.java
│       │   │   │   ├── service/AuthService.java
│       │   │   │   ├── service/JwtService.java
│       │   │   │   ├── dto/LoginRequest.java
│       │   │   │   ├── dto/TokenResponse.java
│       │   │   │   └── filter/JwtAuthFilter.java
│       │   │   ├── account/
│       │   │   │   ├── controller/AccountController.java
│       │   │   │   ├── service/AccountService.java
│       │   │   │   ├── repository/AccountRepository.java
│       │   │   │   ├── domain/Account.java          # JPA entity
│       │   │   │   └── dto/AccountDto.java
│       │   │   ├── creditcard/
│       │   │   │   ├── controller/CreditCardController.java
│       │   │   │   ├── service/CreditCardService.java
│       │   │   │   ├── repository/CreditCardRepository.java
│       │   │   │   ├── domain/CreditCard.java
│       │   │   │   └── dto/CreditCardDto.java
│       │   │   ├── aggregation/
│       │   │   │   ├── controller/AggregationController.java
│       │   │   │   ├── service/AggregationService.java  # Unified financial view logic
│       │   │   │   └── dto/FinancialSummaryDto.java
│       │   │   ├── dashboard/
│       │   │   │   ├── controller/DashboardController.java
│       │   │   │   ├── service/DashboardService.java
│       │   │   │   ├── repository/DashboardPreferenceRepository.java
│       │   │   │   ├── domain/DashboardPreference.java
│       │   │   │   └── dto/DashboardPreferenceDto.java
│       │   │   ├── user/
│       │   │   │   ├── controller/UserController.java
│       │   │   │   ├── service/UserService.java
│       │   │   │   ├── repository/UserRepository.java
│       │   │   │   ├── domain/User.java
│       │   │   │   └── dto/UserDto.java
│       │   │   └── common/
│       │   │       ├── exception/GlobalExceptionHandler.java
│       │   │       ├── exception/ResourceNotFoundException.java
│       │   │       ├── exception/UnauthorizedException.java
│       │   │       └── util/PaginationUtil.java
│       │   └── resources/
│       │       ├── application.yml                  # Base config (no secrets)
│       │       ├── application-local.yml            # Local dev overrides
│       │       ├── application-test.yml             # Test profile config
│       │       └── db/migration/                    # Flyway migrations
│       │           ├── V1__create_users.sql
│       │           ├── V2__create_accounts.sql
│       │           ├── V3__create_credit_cards.sql
│       │           └── V4__create_dashboard_preferences.sql
│       └── test/
│           └── java/com/unifiedfin/api/
│               ├── auth/
│               │   ├── AuthControllerTest.java
│               │   └── AuthServiceTest.java
│               ├── account/
│               │   ├── AccountControllerTest.java
│               │   └── AccountServiceTest.java
│               ├── creditcard/
│               │   ├── CreditCardControllerTest.java
│               │   └── CreditCardServiceTest.java
│               ├── aggregation/
│               │   └── AggregationServiceTest.java
│               ├── dashboard/
│               │   └── DashboardServiceTest.java
│               └── integration/
│                   ├── AccountIntegrationTest.java  # Testcontainers + PostgreSQL
│                   └── AuthIntegrationTest.java
│
├── bff/                               # Node.js / Express BFF service
│   ├── Dockerfile
│   ├── package.json
│   ├── pnpm-lock.yaml
│   ├── tsconfig.json                  # TypeScript config (strict mode)
│   ├── jest.config.ts
│   ├── .eslintrc.json
│   ├── .prettierrc
│   └── src/
│       ├── index.ts                   # Express app entry point
│       ├── app.ts                     # App factory (testable, no listen call)
│       ├── config/
│       │   ├── env.ts                 # Validated env vars (zod schema)
│       │   └── redis.ts              # Redis client setup
│       ├── middleware/
│       │   ├── authProxy.ts          # Forward JWT, validate session
│       │   ├── rateLimiter.ts        # Redis-backed rate limiting
│       │   ├── requestLogger.ts      # Morgan + structured logging
│       │   └── errorHandler.ts       # Centralised error response
│       ├── routes/
│       │   ├── index.ts              # Route registry
│       │   ├── auth.routes.ts
│       │   ├── accounts.routes.ts
│       │   ├── creditCards.routes.ts
│       │   ├── aggregation.routes.ts
│       │   └── dashboard.routes.ts
│       ├── services/
│       │   ├── apiClient.ts          # Axios instance pointing to Spring Boot API
│       │   └── sessionService.ts     # Redis session read/write
│       └── types/
│           └── index.ts              # Shared TypeScript interfaces
│   └── tests/
│       ├── unit/
│       │   ├── middleware/
│       │   │   └── authProxy.test.ts
│       │   └── services/
│       │       └── sessionService.test.ts
│       └── integration/
│           ├── auth.routes.test.ts   # Supertest against app.ts
│           └── accounts.routes.test.ts
│
└── web/                              # React.js frontend
    ├── Dockerfile
    ├── package.json
    ├── pnpm-lock.yaml
    ├── vite.config.ts
    ├── tsconfig.json                 # Strict TypeScript
    ├── tsconfig.node.json
    ├── vitest.config.ts
    ├── .eslintrc.json
    ├── .prettierrc
    ├── index.html                    # Vite entry HTML
    └── src/
        ├── main.tsx                  # React root mount
        ├── App.tsx                   # Router + global providers
        ├── assets/                   # Static images, fonts
        ├── components/
        │   ├── common/               # Reusable UI primitives
        │   │   ├── Button/
        │   │   │   ├── Button.tsx
        │   │   │   └── Button.test.tsx
        │   │   ├── Card/
        │   │   └── LoadingSpinner/
        │   └── layout/
        │       ├── AppShell.tsx
        │       └── Sidebar.tsx
        ├── features/
        │   ├── auth/
        │   │   ├── LoginPage.tsx
        │   │   ├── useAuth.ts        # Auth hook + context
        │   │   └── authSlice.ts      # Redux Toolkit slice
        │   ├── accounts/
        │   │   ├── AccountsPage.tsx
        │   │   ├── AccountCard.tsx
        │   │   ├── AccountCard.test.tsx
        │   │   └── accountsSlice.ts
        │   ├── creditCards/
        │   │   ├── CreditCardsPage.tsx
        │   │   ├── CreditCardItem.tsx
        │   │   └── creditCardsSlice.ts
        │   ├── dashboard/
        │   │   ├── DashboardPage.tsx
        │   │   ├── DashboardGrid.tsx  # Customisable widget grid
        │   │   ├── DashboardGrid.test.tsx
        │   │   ├── widgets/
        │   │   │   ├── NetWorthWidget.tsx
        │   │   │   ├── SpendingWidget.tsx
        │   │   │   └── CreditUtilWidget.tsx
        │   │   └── dashboardSlice.ts
        │   └── aggregation/
        │       ├── FinancialSummary.tsx
        │       └── aggregationSlice.ts
        ├── hooks/
        │   ├── useFinancialData.ts
        │   └── useDashboardPrefs.ts
        ├── services/
        │   └── apiClient.ts          # Axios instance pointing to BFF
        ├── store/
        │   ├── index.ts              # Redux store configuration
        │   └── rootReducer.ts
        ├── router/
        │   └── AppRouter.tsx         # React Router v6 routes + guards
        ├── styles/
        │   └── global.css
        └── types/
            └── index.ts              # Shared frontend types
```

---

## 3. Required Workflow

The agent **must** execute steps in this exact order. Do not skip or reorder.

### Step 1 — Read All Specifications
- Read every file in the `specs/` directory (stories, domain models, acceptance criteria) before writing any code.
- If `specs/` is absent, halt and request specifications.

### Step 2 — Create `tasks.md`
- Create `tasks.md` in the repository root **before writing any implementation code**.
- Structure it as a checklist with sections: `[ ] Scaffold`, `[ ] API (Java)`, `[ ] BFF (Node)`, `[ ] Web (React)`, `[ ] Tests`, `[ ] Docker`, `[ ] CI`.
- Each task must reference the spec story or acceptance criterion it satisfies.
- Update `tasks.md` (mark items complete) as work progresses.

### Step 3 — Scaffold Infrastructure First
1. Create `.env.example` with all required variables (see Constraints).
2. Write `docker-compose.yml` (PostgreSQL, Redis, api, bff, web services).
3. Write all `Dockerfile`s.
4. Write Flyway migration SQL files (`V1` through `V4` minimum).
5. Verify `docker compose up --build` succeeds with no errors before proceeding.

### Step 4 — Implement Backend (Java / Spring Boot)
1. Implement domain entities and JPA repositories.
2. Implement service layer with full business logic.
3. Implement REST controllers with `@Valid` request validation.
4. Implement JWT auth filter and Spring Security configuration.
5. Implement `GlobalExceptionHandler` before controllers go live.
6. Run `./mvnw verify` — all tests must pass before moving on.

### Step 5 — Implement BFF (Node.js / Express)
1. Implement env validation in `config/env.ts` using Zod.
2. Implement middleware in order: `requestLogger` → `rateLimiter` → `authProxy`.
3. Implement all route files, proxying to the Spring Boot API via `apiClient.ts`.
4. Run `pnpm test` — all tests must pass before moving on.

### Step 6 — Implement Frontend (React.js)
1. Implement Redux store, slices, and RTK Query endpoints.
2. Implement `AppRouter` with protected route guards.
3. Implement feature pages and components in dependency order: auth → accounts → credit cards → aggregation → dashboard.
4. Implement dashboard customisation (drag-and-drop widget layout, persisted via BFF).
5. Run `pnpm test` — all tests must pass before moving on.

### Step 7 — Validate End-to-End
1. Run `docker compose -f docker-compose.test.yml up --abort-on-container-exit`.
2. Confirm all services start, health checks pass, and integration tests exit 0.
3. Run coverage reports for all three sub-projects and confirm ≥ 90% unit coverage.
4. Fix any failures before marking tasks complete.

### Step 8 — Final Checklist
- [ ] `tasks.md` fully checked off
- [ ] No secrets committed (only `.env.example`)
- [ ] All Dockerfiles build without warnings
- [ ] CI pipeline file exists and is valid YAML
- [ ] README.md documents how to run locally and how to run tests

---

## 4. Coding Conventions

### General
- All code must be written in **TypeScript** (strict mode) for Node/React and **Java 21** with records and sealed types where appropriate.
- No `any` in TypeScript. No raw types in Java generics.
- All public methods must have Javadoc (Java) or JSDoc (TypeScript) comments.
- Environment variables are **never** hard