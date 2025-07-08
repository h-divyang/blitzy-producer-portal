# 06.0 Project Structure & Organization - Updated

## Project Architecture Overview

### Mono-Repo Strategy with Microservice Evolution
- **Unified Repository**: Single repository containing all components for simplified development
- **Microservice Boundaries**: Clear service boundaries within monolith for future extraction
- **Shared Components**: Common libraries and utilities shared across services
- **Independent Deployment**: Structure supports independent service deployment when needed
- **Technology Alignment**: Laravel 12.x+ backend with React 18+ frontend

## Root Directory Structure

```
insurance-system/
├── .env.example                 # Environment configuration template
├── .env.testing                 # Testing environment configuration
├── .gitignore                   # Git ignore patterns
├── .gitlab-ci.yml              # GitLab CI/CD pipeline configuration
├── composer.json               # PHP dependencies and autoloading
├── package.json                # Node.js dependencies and scripts
├── docker-compose.yml          # Local development environment
├── docker-compose.prod.yml     # Production Docker configuration
├── Dockerfile                  # Multi-stage Docker build
├── Makefile                    # Build automation commands
├── README.md                   # Project documentation
├── CHANGELOG.md                # Version history and changes
├── LICENSE                     # Project license
│
├── app/                        # Laravel application core
├── bootstrap/                  # Laravel bootstrap files
├── config/                     # Configuration files
├── database/                   # Database migrations, factories, seeds
├── public/                     # Public web assets
├── resources/                  # Frontend resources and views
├── routes/                     # API and web routes
├── storage/                    # File storage and logs
├── tests/                      # Backend test suites
├── vendor/                     # Composer dependencies (auto-generated)
│
├── frontend/                   # React frontend application
├── services/                   # Microservice modules (future extraction)
├── shared/                     # Shared libraries and utilities
├── docs/                       # Project documentation
├── scripts/                    # Build and deployment scripts
├── kubernetes/                 # Kubernetes deployment manifests
├── helm/                       # Helm chart templates
└── .docker/                    # Docker configuration files
```

## Laravel Backend Structure (app/)

### Core Application Organization
```
app/
├── Console/                    # Artisan commands
│   ├── Commands/               # Custom Artisan commands
│   │   ├── TenantManagement/   # Tenant-specific commands
│   │   ├── DataMigration/      # Data migration utilities
│   │   └── SystemMaintenance/  # Maintenance commands
│   └── Kernel.php              # Command registration
│
├── Events/                     # Domain events
│   ├── Policy/                 # Policy-related events
│   │   ├── PolicyCreated.php
│   │   ├── PolicyBound.php
│   │   └── PolicyCancelled.php
│   ├── Claims/                 # Claims-related events
│   └── User/                   # User-related events
│
├── Exceptions/                 # Custom exceptions
│   ├── Business/               # Business logic exceptions
│   │   ├── PolicyException.php
│   │   ├── UnderwritingException.php
│   │   └── ClaimException.php
│   ├── Integration/            # External service exceptions
│   ├── Security/               # Security-related exceptions
│   └── Handler.php             # Global exception handler
│
├── Http/                       # HTTP layer
│   ├── Controllers/            # API and web controllers
│   │   ├── Api/                # API controllers
│   │   │   ├── V1/             # API version 1
│   │   │   │   ├── Auth/       # Authentication endpoints
│   │   │   │   ├── Policies/   # Policy management
│   │   │   │   ├── Claims/     # Claims management
│   │   │   │   ├── Users/      # User management
│   │   │   │   └── Reports/    # Reporting endpoints
│   │   │   └── V2/             # Future API version
│   │   ├── Admin/              # Administrative controllers
│   │   └── Web/                # Web controllers (if needed)
│   │
│   ├── Middleware/             # HTTP middleware
│   │   ├── TenantIsolation.php # Multi-tenant isolation
│   │   ├── RolePermission.php  # RBAC enforcement
│   │   ├── ApiVersioning.php   # API version handling
│   │   ├── AuditLogging.php    # Request/response auditing
│   │   └── SecurityHeaders.php # Security header enforcement
│   │
│   ├── Requests/               # Form request validation
│   │   ├── Auth/               # Authentication requests
│   │   ├── Policy/             # Policy-related requests
│   │   │   ├── StorePolicyRequest.php
│   │   │   ├── UpdatePolicyRequest.php
│   │   │   └── BindPolicyRequest.php
│   │   ├── Claims/             # Claims-related requests
│   │   └── User/               # User management requests
│   │
│   ├── Resources/              # API resources (transformers)
│   │   ├── Policy/             # Policy API resources
│   │   │   ├── PolicyResource.php
│   │   │   ├── PolicyCollection.php
│   │   │   └── PolicyDetailResource.php
│   │   ├── Claims/             # Claims API resources
│   │   └── User/               # User API resources
│   │
│   └── Kernel.php              # HTTP kernel configuration
│
├── Jobs/                       # Queue jobs
│   ├── Policy/                 # Policy-related background jobs
│   │   ├── ProcessPolicyBinding.php
│   │   ├── GeneratePolicyDocuments.php
│   │   └── SendPolicyNotifications.php
│   ├── Claims/                 # Claims processing jobs
│   ├── Billing/                # Billing and payment jobs
│   ├── Notifications/          # Notification jobs
│   └── DataProcessing/         # Data processing jobs
│
├── Listeners/                  # Event listeners
│   ├── Policy/                 # Policy event listeners
│   ├── Claims/                 # Claims event listeners
│   ├── Audit/                  # Audit logging listeners
│   └── Notification/           # Notification listeners
│
├── Mail/                       # Email templates and classes
│   ├── Policy/                 # Policy-related emails
│   ├── Claims/                 # Claims-related emails
│   ├── Auth/                   # Authentication emails
│   └── System/                 # System notifications
│
├── Models/                     # Eloquent models
│   ├── Core/                   # Core system models
│   │   ├── User.php
│   │   ├── Tenant.php
│   │   ├── Role.php
│   │   └── Permission.php
│   │
│   ├── Policy/                 # Policy domain models
│   │   ├── Policy.php
│   │   ├── Coverage.php
│   │   ├── Endorsement.php
│   │   ├── Quote.php
│   │   └── Renewal.php
│   │
│   ├── Claims/                 # Claims domain models
│   │   ├── Claim.php
│   │   ├── ClaimItem.php
│   │   ├── Adjuster.php
│   │   └── Settlement.php
│   │
│   ├── Financial/              # Financial models
│   │   ├── Invoice.php
│   │   ├── Payment.php
│   │   ├── Commission.php
│   │   └── Premium.php
│   │
│   ├── Configuration/          # System configuration models
│   │   ├── SystemConfig.php
│   │   ├── TenantConfig.php
│   │   └── ApplicationSetting.php
│   │
│   └── Audit/                  # Audit and logging models
│       ├── AuditTrail.php
│       ├── SecurityLog.php
│       └── ComplianceLog.php
│
├── Notifications/              # Laravel notifications
│   ├── Policy/                 # Policy notifications
│   ├── Claims/                 # Claims notifications
│   ├── System/                 # System notifications
│   └── Channels/               # Custom notification channels
│
├── Observers/                  # Model observers
│   ├── PolicyObserver.php     # Policy model observer
│   ├── ClaimObserver.php      # Claim model observer
│   └── AuditObserver.php      # Audit trail observer
│
├── Policies/                   # Authorization policies
│   ├── PolicyPolicy.php       # Policy authorization
│   ├── ClaimPolicy.php        # Claims authorization
│   ├── UserPolicy.php         # User management authorization
│   └── TenantPolicy.php       # Tenant-specific authorization
│
├── Providers/                  # Service providers
│   ├── AppServiceProvider.php # Main application provider
│   ├── AuthServiceProvider.php # Authentication provider
│   ├── EventServiceProvider.php # Event provider
│   ├── RouteServiceProvider.php # Route provider
│   ├── TenantServiceProvider.php # Multi-tenant provider
│   └── CustomServiceProvider.php # Custom services
│
├── Rules/                      # Custom validation rules
│   ├── Policy/                 # Policy validation rules
│   ├── Insurance/              # Insurance-specific rules
│   ├── Financial/              # Financial validation rules
│   └── Security/               # Security validation rules
│
└── Services/                   # Application services
    ├── Policy/                 # Policy domain services
    │   ├── PolicyService.php
    │   ├── UnderwritingService.php
    │   ├── RatingService.php
    │   └── DocumentService.php
    │
    ├── Claims/                 # Claims domain services
    │   ├── ClaimService.php
    │   ├── AdjusterService.php
    │   └── SettlementService.php
    │
    ├── Financial/              # Financial services
    │   ├── BillingService.php
    │   ├── PaymentService.php
    │   └── CommissionService.php
    │
    ├── Integration/            # External service integrations
    │   ├── PaymentGatewayService.php
    │   ├── EmailService.php
    │   ├── SmsService.php
    │   └── DocumentSigningService.php
    │
    ├── Tenant/                 # Multi-tenant services
    │   ├── TenantService.php
    │   ├── TenantConfigService.php
    │   └── TenantIsolationService.php
    │
    └── Shared/                 # Shared utility services
        ├── CacheService.php
        ├── LoggingService.php
        ├── SecurityService.php
        └── ValidationService.php
```

## React Frontend Structure (frontend/)

### Modern React 18+ Application Structure
```
frontend/
├── public/                     # Static public assets
│   ├── index.html              # Main HTML template
│   ├── favicon.ico             # Application favicon
│   ├── manifest.json           # PWA manifest
│   └── robots.txt              # SEO robots configuration
│
├── src/                        # Source code
│   ├── components/             # Reusable UI components
│   │   ├── common/             # Common UI components
│   │   │   ├── Button/         # Button component variations
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Button.test.tsx
│   │   │   │   ├── Button.stories.tsx
│   │   │   │   └── Button.module.css
│   │   │   ├── Input/          # Input component variations
│   │   │   ├── Modal/          # Modal component
│   │   │   ├── Table/          # Data table component
│   │   │   ├── Form/           # Form components
│   │   │   └── Layout/         # Layout components
│   │   │
│   │   ├── domain/             # Domain-specific components
│   │   │   ├── Policy/         # Policy-related components
│   │   │   │   ├── PolicyForm/
│   │   │   │   ├── PolicyList/
│   │   │   │   ├── PolicyDetails/
│   │   │   │   └── PolicyWizard/
│   │   │   ├── Claims/         # Claims-related components
│   │   │   ├── Users/          # User management components
│   │   │   └── Reports/        # Reporting components
│   │   │
│   │   └── navigation/         # Navigation components
│   │       ├── Header/
│   │       ├── Sidebar/
│   │       ├── Breadcrumbs/
│   │       └── Menu/
│   │
│   ├── pages/                  # Page-level components
│   │   ├── auth/               # Authentication pages
│   │   │   ├── Login/
│   │   │   ├── Register/
│   │   │   ├── ForgotPassword/
│   │   │   └── MfaVerification/
│   │   ├── dashboard/          # Dashboard pages
│   │   ├── policies/           # Policy management pages
│   │   │   ├── PolicyList.tsx
│   │   │   ├── PolicyCreate.tsx
│   │   │   ├── PolicyEdit.tsx
│   │   │   └── PolicyDetails.tsx
│   │   ├── claims/             # Claims management pages
│   │   ├── reports/            # Reporting pages
│   │   ├── admin/              # Administrative pages
│   │   └── errors/             # Error pages (404, 500, etc.)
│   │
│   ├── hooks/                  # Custom React hooks
│   │   ├── auth/               # Authentication hooks
│   │   │   ├── useAuth.ts
│   │   │   ├── usePermissions.ts
│   │   │   └── useTenant.ts
│   │   ├── api/                # API-related hooks
│   │   │   ├── usePolicies.ts
│   │   │   ├── useClaims.ts
│   │   │   └── useUsers.ts
│   │   ├── common/             # Common utility hooks
│   │   │   ├── useLocalStorage.ts
│   │   │   ├── useDebounce.ts
│   │   │   └── usePagination.ts
│   │   └── business/           # Business logic hooks
│   │       ├── usePremiumCalculation.ts
│   │       └── useUnderwriting.ts
│   │
│   ├── services/               # API and external services
│   │   ├── api/                # API client and endpoints
│   │   │   ├── client.ts       # Base API client configuration
│   │   │   ├── auth.ts         # Authentication API
│   │   │   ├── policies.ts     # Policy API endpoints
│   │   │   ├── claims.ts       # Claims API endpoints
│   │   │   ├── users.ts        # User management API
│   │   │   └── reports.ts      # Reporting API
│   │   ├── external/           # External service integrations
│   │   │   ├── analytics.ts    # Analytics service
│   │   │   ├── monitoring.ts   # Monitoring service
│   │   │   └── storage.ts      # File storage service
│   │   └── utils/              # Service utilities
│   │       ├── tokenManager.ts
│   │       ├── errorHandler.ts
│   │       └── responseInterceptor.ts
│   │
│   ├── store/                  # State management
│   │   ├── index.ts            # Store configuration
│   │   ├── slices/             # Redux Toolkit slices (if used)
│   │   ├── context/            # React Context providers
│   │   │   ├── AuthContext.tsx
│   │   │   ├── TenantContext.tsx
│   │   │   └── ThemeContext.tsx
│   │   └── providers/          # Combined providers
│   │       └── AppProvider.tsx
│   │
│   ├── types/                  # TypeScript type definitions
│   │   ├── api/                # API response types
│   │   │   ├── auth.types.ts
│   │   │   ├── policy.types.ts
│   │   │   ├── claim.types.ts
│   │   │   └── user.types.ts
│   │   ├── business/           # Business domain types
│   │   │   ├── insurance.types.ts
│   │   │   ├── financial.types.ts
│   │   │   └── compliance.types.ts
│   │   ├── common/             # Common utility types
│   │   │   ├── pagination.types.ts
│   │   │   ├── form.types.ts
│   │   │   └── table.types.ts
│   │   └── global.d.ts         # Global type declarations
│   │
│   ├── utils/                  # Utility functions
│   │   ├── formatters/         # Data formatting utilities
│   │   │   ├── currency.ts
│   │   │   ├── date.ts
│   │   │   ├── phone.ts
│   │   │   └── policy.ts
│   │   ├── validators/         # Client-side validation
│   │   │   ├── forms.ts
│   │   │   ├── business.ts
│   │   │   └── security.ts
│   │   ├── helpers/            # General helper functions
│   │   │   ├── arrays.ts
│   │   │   ├── objects.ts
│   │   │   ├── strings.ts
│   │   │   └── math.ts
│   │   └── constants/          # Application constants
│   │       ├── api.ts
│   │       ├── business.ts
│   │       ├── routes.ts
│   │       └── ui.ts
│   │
│   ├── styles/                 # Global styles and themes
│   │   ├── globals.css         # Global CSS styles
│   │   ├── variables.css       # CSS custom properties
│   │   ├── themes/             # Theme configurations
│   │   │   ├── light.css
│   │   │   └── dark.css
│   │   └── components/         # Component-specific styles
│   │       └── shared.css
│   │
│   ├── assets/                 # Static assets
│   │   ├── images/             # Image assets
│   │   │   ├── logos/
│   │   │   ├── icons/
│   │   │   └── illustrations/
│   │   ├── fonts/              # Custom fonts
│   │   └── documents/          # Document templates
│   │
│   ├── config/                 # Configuration files
│   │   ├── environment.ts      # Environment configuration
│   │   ├── api.config.ts       # API configuration
│   │   ├── router.config.ts    # Router configuration
│   │   └── app.config.ts       # Application configuration
│   │
│   ├── App.tsx                 # Main application component
│   ├── App.test.tsx            # Application tests
│   ├── index.tsx               # Application entry point
│   ├── react-app-env.d.ts      # React TypeScript definitions
│   └── setupTests.ts           # Test setup configuration
│
├── .env                        # Environment variables
├── .env.local                  # Local environment overrides
├── .env.production             # Production environment variables
├── .gitignore                  # Git ignore patterns
├── package.json                # Dependencies and scripts
├── package-lock.json           # Dependency lock file
├── tsconfig.json               # TypeScript configuration
├── tailwind.config.js          # Tailwind CSS configuration
├── vite.config.ts              # Vite build configuration
├── vitest.config.ts            # Vitest testing configuration
├── .eslintrc.js                # ESLint configuration
├── .prettierrc                 # Prettier configuration
├── jest.config.js              # Jest testing configuration (if used)
└── README.md                   # Frontend documentation
```

## Microservice Structure (services/)

### Future Microservice Extraction Preparation
```
services/
├── policy-service/             # Policy management microservice
│   ├── src/
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Services/
│   │   └── Events/
│   ├── routes/
│   ├── config/
│   ├── tests/
│   ├── Dockerfile
│   └── composer.json
│
├── claims-service/             # Claims processing microservice
│   ├── src/
│   ├── routes/
│   ├── config/
│   ├── tests/
│   ├── Dockerfile
│   └── composer.json
│
├── user-service/               # User management microservice
│   ├── src/
│   ├── routes/
│   ├── config/
│   ├── tests/
│   ├── Dockerfile
│   └── composer.json
│
├── billing-service/            # Billing and payments microservice
│   ├── src/
│   ├── routes/
│   ├── config/
│   ├── tests/
│   ├── Dockerfile
│   └── composer.json
│
├── communication-service/      # Notifications and communications
│   ├── src/
│   ├── routes/
│   ├── config/
│   ├── tests/
│   ├── Dockerfile
│   └── composer.json
│
└── shared/                     # Shared microservice libraries
    ├── authentication/
    ├── database/
    ├── logging/
    ├── events/
    └── utilities/
```

## Shared Libraries Structure (shared/)

### Common Components and Utilities
```
shared/
├── php/                        # Shared PHP libraries
│   ├── Authentication/         # Common auth utilities
│   ├── Database/               # Database helpers
│   ├── Validation/             # Validation rules
│   ├── Events/                 # Shared events
│   ├── Exceptions/             # Common exceptions
│   ├── Traits/                 # Reusable traits
│   └── Utilities/              # General utilities
│
├── typescript/                 # Shared TypeScript libraries
│   ├── types/                  # Common type definitions
│   ├── utils/                  # Utility functions
│   ├── constants/              # Shared constants
│   ├── validators/             # Validation schemas
│   └── components/             # Shared React components
│
├── config/                     # Shared configuration
│   ├── database.php            # Database configuration
│   ├── cache.php               # Cache configuration
│   ├── queue.php               # Queue configuration
│   └── logging.php             # Logging configuration
│
└── docs/                       # Shared documentation
    ├── coding-standards.md
    ├── architecture.md
    ├── api-conventions.md
    └── security-guidelines.md
```

## Database Structure (database/)

### Laravel Database Organization
```
database/
├── factories/                  # Model factories for testing
│   ├── PolicyFactory.php
│   ├── ClaimFactory.php
│   ├── UserFactory.php
│   └── TenantFactory.php
│
├── migrations/                 # Database migrations
│   ├── 2024_01_01_000000_create_tenants_table.php
│   ├── 2024_01_01_000001_create_users_table.php
│   ├── 2024_01_01_000002_create_policies_table.php
│   ├── 2024_01_01_000003_create_claims_table.php
│   └── tenant_specific/        # Tenant-specific migrations
│       ├── create_tenant_config_table.php
│       └── create_tenant_audit_table.php
│
├── seeders/                    # Database seeders
│   ├── DatabaseSeeder.php     # Main seeder
│   ├── TenantSeeder.php       # Tenant data seeder
│   ├── RolePermissionSeeder.php # RBAC seeder
│   ├── PolicyTypeSeeder.php   # Insurance policy types
│   ├── TestDataSeeder.php     # Test data seeder
│   └── ProductionSeeder.php   # Production data seeder
│
└── sql/                        # Raw SQL files
    ├── views/                  # Database views
    ├── procedures/             # Stored procedures
    ├── functions/              # Database functions
    └── indexes/                # Index optimization scripts
```

## Configuration Structure (config/)

### Laravel Configuration Organization
```
config/
├── app.php                     # Application configuration
├── auth.php                    # Authentication configuration
├── cache.php                   # Cache configuration
├── database.php                # Database configuration
├── filesystems.php             # File storage configuration
├── logging.php                 # Logging configuration
├── mail.php                    # Email configuration
├── queue.php                   # Queue configuration
├── services.php                # External services configuration
├── session.php                 # Session configuration
│
├── tenant.php                  # Multi-tenant configuration
├── insurance.php               # Insurance-specific configuration
├── security.php                # Security configuration
├── audit.php                   # Audit logging configuration
├── integration.php             # External integration configuration
└── business.php                # Business rules configuration
```

## Testing Structure (tests/)

### Comprehensive Test Organization
```
tests/
├── Feature/                    # Feature/integration tests
│   ├── Auth/                   # Authentication tests
│   ├── Api/                    # API endpoint tests
│   │   ├── V1/                 # API version 1 tests
│   │   │   ├── PolicyTest.php
│   │   │   ├── ClaimTest.php
│   │   │   └── UserTest.php
│   │   └── V2/                 # Future API version tests
│   ├── Admin/                  # Admin functionality tests
│   ├── Tenant/                 # Multi-tenant tests
│   └── Integration/            # External service integration tests
│
├── Unit/                       # Unit tests
│   ├── Models/                 # Model tests
│   │   ├── PolicyTest.php
│   │   ├── ClaimTest.php
│   │   └── UserTest.php
│   ├── Services/               # Service class tests
│   │   ├── PolicyServiceTest.php
│   │   ├── ClaimServiceTest.php
│   │   └── BillingServiceTest.php
│   ├── Rules/                  # Validation rule tests
│   ├── Helpers/                # Helper function tests
│   └── Utilities/              # Utility class tests
│
├── Browser/                    # Browser/E2E tests (Laravel Dusk)
│   ├── PolicyManagementTest.php
│   ├── ClaimProcessingTest.php
│   └── UserAuthenticationTest.php
│
├── Performance/                # Performance tests
│   ├── ApiPerformanceTest.php
│   ├── DatabasePerformanceTest.php
│   └── LoadTest.php
│
├── Security/                   # Security tests
│   ├── AuthenticationSecurityTest.php
│   ├── AuthorizationTest.php
│   └── InputValidationTest.php
│
└── TestCase.php                # Base test case
```

## Development Tools Integration

### IDE and Development Environment Setup
- **Primary IDE Integration**: PhpStorm/VS Code with Laravel 12.x+ plugin support
- **Code Standards**: PSR-12 PHP coding standards with Laravel conventions
- **Debugging Integration**: Xdebug configuration for step-through debugging
- **Testing Framework**: PHPUnit test runner integration with coverage reporting
- **Database Tools**: Direct database connection and query execution tools
- **Version Control**: Git 2.45+ integration with GitLab workflows and merge request support

### Local Development Environment (Laravel Sail)
```yaml
# docker-compose.yml - Laravel Sail configuration for insurance system
version: '3'
services:
    laravel.test:
        build:
            context: './vendor/laravel/sail/runtimes/8.4'
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: sail-8.4/app
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${APP_PORT:-80}:80'
            - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
        depends_on:
            - mariadb
            - redis
            - meilisearch
    mariadb:
        image: 'mariadb:12'
        ports:
            - '${FORWARD_DB_PORT:-3306}:3306'
        environment:
            MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ROOT_HOST: "%"
            MYSQL_DATABASE: '${DB_DATABASE}'
            MYSQL_USER: '${DB_USERNAME}'
            MYSQL_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ALLOW_EMPTY_PASSWORD: 1
        volumes:
            - 'sail-mariadb:/var/lib/mysql'
            - './vendor/laravel/sail/database/mysql/create-testing-database.sh:/docker-entrypoint-initdb.d/10-create-testing-database.sh'
        networks:
            - sail
        healthcheck:
            test: ["CMD", "mysqladmin", "ping", "-p${DB_PASSWORD}"]
            retries: 3
            timeout: 5s
    redis:
        image: 'redis:7-alpine'
        ports:
            - '${FORWARD_REDIS_PORT:-6379}:6379'
        volumes:
            - 'sail-redis:/data'
        networks:
            - sail
        healthcheck:
            test: ["CMD", "redis-cli", "ping"]
            retries: 3
            timeout: 5s
networks:
    sail:
        driver: bridge
volumes:
    sail-mariadb:
        driver: local
    sail-redis:
        driver: local
```

### Build Automation and Task Management (Makefile Integration)
```makefile
# Makefile - Insurance System Build Automation
.PHONY: help install dev build test deploy clean

# Default environment
ENV ?= local

help: ## Show this help message
	@echo 'Usage: make [target] [ENV=environment]'
	@echo ''
	@echo 'Targets:'
	@awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_-]+:.*?## / {printf "  %-15s %s\n", $$1, $$2}' $(MAKEFILE_LIST)

install: ## Install dependencies and setup environment
	@echo "Installing dependencies for $(ENV) environment..."
	composer install --optimize-autoloader $(if $(filter production,$(ENV)),--no-dev)
	npm install
	cp .env.$(ENV) .env || cp .env.example .env
	php artisan key:generate
	php artisan storage:link

dev: ## Start development environment
	@echo "Starting development environment..."
	./vendor/bin/sail up -d
	./vendor/bin/sail npm run dev &
	@echo "Development environment started at http://localhost"

build: ## Build for production
	@echo "Building for $(ENV) environment..."
	npm run build
	composer install --optimize-autoloader --no-dev
	php artisan config:cache
	php artisan route:cache
	php artisan view:cache

test: ## Run test suite
	@echo "Running tests..."
	./vendor/bin/sail test --parallel --coverage
	npm run test

migrate: ## Run database migrations
	@echo "Running migrations for $(ENV)..."
	php artisan migrate $(if $(filter production,$(ENV)),--force)

seed: ## Seed database
	@echo "Seeding database for $(ENV)..."
	php artisan db:seed $(if $(filter production,$(ENV)),--class=ProductionSeeder,--class=DevelopmentSeeder)

deploy: build ## Deploy application
	@echo "Deploying to $(ENV)..."
	docker build -t insurance-system:$(ENV) .
	kubectl apply -f kubernetes/$(ENV)/

clean: ## Clean cache and temporary files
	@echo "Cleaning cache and temporary files..."
	php artisan cache:clear
	php artisan config:clear
	php artisan route:clear
	php artisan view:clear
	npm run clean
```

### Development Environment Configuration
```
.docker/
├── nginx/                      # Nginx configuration
│   ├── default.conf
│   └── ssl/
├── php/                        # PHP configuration
│   ├── php.ini
│   └── xdebug.ini
├── mysql/                      # MySQL configuration
│   └── my.cnf
└── redis/                      # Redis configuration
    └── redis.conf

scripts/
├── build/                      # Build scripts
│   ├── build-prod.sh
│   ├── build-staging.sh
│   └── build-local.sh
├── deploy/                     # Deployment scripts
│   ├── deploy-k8s.sh
│   ├── deploy-staging.sh
│   └── rollback.sh
├── maintenance/                # Maintenance scripts
│   ├── backup.sh
│   ├── restore.sh
│   └── cleanup.sh
└── development/                # Development utilities
    ├── setup-local.sh
    ├── run-tests.sh
    └── generate-docs.sh
```

## Naming Conventions

### File and Directory Naming Standards
- **Laravel Models**: PascalCase (Policy.php, ClaimItem.php)
- **Controllers**: PascalCase with suffix (PolicyController.php)
- **Services**: PascalCase with suffix (PolicyService.php)
- **React Components**: PascalCase directories and files (PolicyForm/PolicyForm.tsx)
- **Hooks**: camelCase with 'use' prefix (usePolicyData.ts)
- **Utilities**: camelCase (formatCurrency.ts)
- **Constants**: UPPER_SNAKE_CASE (API_BASE_URL)
- **Database**: snake_case (policy_items, claim_adjusters)

### Code Organization Principles
- **Domain-Driven Design**: Organize by business domain rather than technical layer
- **Separation of Concerns**: Clear boundaries between presentation, business, and data layers
- **Dependency Inversion**: High-level modules should not depend on low-level modules
- **Single Responsibility**: Each class/component should have one reason to change
- **Open/Closed Principle**: Open for extension, closed for modification

This comprehensive project structure supports the evolution from a Laravel monolith to microservices while maintaining clear organization, proper separation of concerns, and adherence to modern development best practices.