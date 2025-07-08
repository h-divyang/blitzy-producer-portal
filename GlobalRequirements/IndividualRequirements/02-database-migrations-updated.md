# 02.0 Database Migrations - Updated

## Multi-Tenant Migration Strategy

### Database Architecture Overview
- **Multi-Tenant Isolation**: Namespace-based tenant isolation with shared infrastructure
- **Primary Database**: MariaDB 12.x LTS for transactional data with row-level security
- **Cache Layer**: Redis 7.x for session management and application caching
- **Search Engine**: Elasticsearch 8.x for full-text search and analytics
- **File Storage**: AWS S3 with tenant-specific bucket organization

### Technology Stack Integration
- **Laravel Version**: 12.x+ with modern migration features
- **PHP Version**: 8.4+ for enhanced database connectivity and security
- **Database Driver**: Laravel's native MariaDB driver with connection pooling
- **Migration Engine**: Laravel's schema builder with tenant-aware extensions

## Core Migration Framework

### Multi-Tenant Database Structure
```php
// TenantAwareMigration.php - Base class for tenant-aware migrations
abstract class TenantAwareMigration extends Migration
{
    /**
     * Run migrations with tenant context
     */
    public function up()
    {
        // Create tenant-isolated table with automatic tenant_id column
        Schema::create($this->getTableName(), function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('tenant_id')->index();
            
            // Call child migration specific schema
            $this->defineSchema($table);
            
            $table->timestamps();
            $table->softDeletes();
            
            // Add tenant isolation foreign key
            $table->foreign('tenant_id')
                  ->references('id')
                  ->on('tenants')
                  ->onDelete('cascade');
            
            // Add tenant-specific unique constraints
            $this->addTenantConstraints($table);
        });
        
        // Add row-level security policies
        $this->addRowLevelSecurity();
    }
    
    /**
     * Define tenant-specific schema (implemented by child classes)
     */
    abstract protected function defineSchema(Blueprint $table): void;
    
    /**
     * Add tenant-specific constraints
     */
    protected function addTenantConstraints(Blueprint $table): void
    {
        // Override in child classes for custom constraints
    }
    
    /**
     * Add row-level security for tenant isolation
     */
    protected function addRowLevelSecurity(): void
    {
        $tableName = $this->getTableName();
        
        // Create RLS policy for tenant isolation
        DB::statement("
            CREATE POLICY tenant_isolation_policy ON {$tableName}
            FOR ALL
            TO app_user
            USING (tenant_id = current_setting('app.current_tenant_id')::bigint)
            WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::bigint)
        ");
        
        // Enable RLS on the table
        DB::statement("ALTER TABLE {$tableName} ENABLE ROW LEVEL SECURITY");
    }
    
    /**
     * Get table name (can be overridden by child classes)
     */
    protected function getTableName(): string
    {
        return $this->table ?? Str::snake(class_basename(static::class));
    }
}
```

### Core Insurance Domain Migrations

#### Tenant Management Migration
```php
// 2024_01_01_000001_create_tenants_table.php
class CreateTenantsTable extends Migration
{
    public function up()
    {
        Schema::create('tenants', function (Blueprint $table) {
            $table->id();
            $table->string('name', 100)->index();
            $table->string('domain', 50)->unique();
            $table->string('database_name', 50)->unique();
            $table->json('configuration')->nullable();
            $table->json('feature_flags')->nullable();
            $table->enum('status', ['active', 'suspended', 'inactive'])->default('active');
            $table->timestamp('trial_ends_at')->nullable();
            $table->timestamps();
            $table->softDeletes();
            
            // Performance indexes
            $table->index(['status', 'created_at']);
            $table->index('domain');
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('tenants');
    }
}
```

#### User Authentication Migration
```php
// 2024_01_01_000002_create_users_table.php
class CreateUsersTable extends TenantAwareMigration
{
    protected function defineSchema(Blueprint $table): void
    {
        $table->string('email', 100)->index();
        $table->timestamp('email_verified_at')->nullable();
        $table->string('password');
        $table->string('first_name', 50);
        $table->string('last_name', 50);
        $table->string('phone', 20)->nullable();
        $table->json('preferences')->nullable();
        $table->string('mfa_secret')->nullable();
        $table->json('mfa_recovery_codes')->nullable();
        $table->timestamp('last_login_at')->nullable();
        $table->ipAddress('last_login_ip')->nullable();
        $table->boolean('is_active')->default(true);
        $table->rememberToken();
    }
    
    protected function addTenantConstraints(Blueprint $table): void
    {
        // Unique email per tenant
        $table->unique(['tenant_id', 'email'], 'users_tenant_email_unique');
        
        // Performance indexes
        $table->index(['tenant_id', 'email', 'is_active']);
        $table->index(['tenant_id', 'last_login_at']);
    }
}
```

#### Policy Management Migration
```php
// 2024_01_01_000003_create_policies_table.php
class CreatePoliciesTable extends TenantAwareMigration
{
    protected function defineSchema(Blueprint $table): void
    {
        $table->string('policy_number', 20)->index();
        $table->unsignedBigInteger('policyholder_id');
        $table->unsignedBigInteger('agent_id')->nullable();
        $table->enum('type', ['auto', 'home', 'business', 'life', 'health']);
        $table->enum('status', ['quote', 'bound', 'active', 'cancelled', 'expired']);
        $table->decimal('premium_amount', 10, 2);
        $table->decimal('coverage_amount', 12, 2);
        $table->date('effective_date');
        $table->date('expiration_date');
        $table->json('coverage_details');
        $table->json('risk_factors')->nullable();
        $table->decimal('deductible', 8, 2)->nullable();
        $table->text('notes')->nullable();
        $table->timestamp('bound_at')->nullable();
        $table->timestamp('cancelled_at')->nullable();
        $table->string('cancellation_reason')->nullable();
    }
    
    protected function addTenantConstraints(Blueprint $table): void
    {
        // Unique policy number per tenant
        $table->unique(['tenant_id', 'policy_number'], 'policies_tenant_number_unique');
        
        // Foreign key constraints
        $table->foreign(['tenant_id', 'policyholder_id'])
              ->references(['tenant_id', 'id'])
              ->on('users')
              ->onDelete('cascade');
              
        $table->foreign(['tenant_id', 'agent_id'])
              ->references(['tenant_id', 'id'])
              ->on('users')
              ->onDelete('set null');
        
        // Performance indexes
        $table->index(['tenant_id', 'status', 'effective_date']);
        $table->index(['tenant_id', 'type', 'status']);
        $table->index(['tenant_id', 'agent_id', 'status']);
        $table->index(['tenant_id', 'expiration_date']);
    }
}
```

#### Claims Management Migration
```php
// 2024_01_01_000004_create_claims_table.php
class CreateClaimsTable extends TenantAwareMigration
{
    protected function defineSchema(Blueprint $table): void
    {
        $table->string('claim_number', 20)->index();
        $table->unsignedBigInteger('policy_id');
        $table->unsignedBigInteger('claimant_id');
        $table->unsignedBigInteger('adjuster_id')->nullable();
        $table->enum('status', ['reported', 'investigating', 'processing', 'approved', 'denied', 'closed']);
        $table->enum('type', ['collision', 'comprehensive', 'liability', 'property', 'injury', 'other']);
        $table->date('incident_date');
        $table->text('description');
        $table->decimal('claimed_amount', 10, 2);
        $table->decimal('approved_amount', 10, 2)->nullable();
        $table->decimal('paid_amount', 10, 2)->default(0);
        $table->json('incident_details');
        $table->json('supporting_documents')->nullable();
        $table->timestamp('reported_at');
        $table->timestamp('closed_at')->nullable();
        $table->text('adjuster_notes')->nullable();
    }
    
    protected function addTenantConstraints(Blueprint $table): void
    {
        // Unique claim number per tenant
        $table->unique(['tenant_id', 'claim_number'], 'claims_tenant_number_unique');
        
        // Foreign key constraints
        $table->foreign(['tenant_id', 'policy_id'])
              ->references(['tenant_id', 'id'])
              ->on('policies')
              ->onDelete('cascade');
              
        $table->foreign(['tenant_id', 'claimant_id'])
              ->references(['tenant_id', 'id'])
              ->on('users')
              ->onDelete('cascade');
              
        $table->foreign(['tenant_id', 'adjuster_id'])
              ->references(['tenant_id', 'id'])
              ->on('users')
              ->onDelete('set null');
        
        // Performance indexes
        $table->index(['tenant_id', 'status', 'incident_date']);
        $table->index(['tenant_id', 'policy_id', 'status']);
        $table->index(['tenant_id', 'adjuster_id', 'status']);
        $table->index(['tenant_id', 'reported_at']);
    }
}
```

## Advanced Migration Features

### Data Encryption Migration
```php
// EncryptedFieldsMigration.php - Handle encrypted sensitive data
abstract class EncryptedFieldsMigration extends TenantAwareMigration
{
    /**
     * Add encrypted fields to table
     */
    protected function addEncryptedFields(Blueprint $table, array $fields): void
    {
        foreach ($fields as $field => $type) {
            switch ($type) {
                case 'text':
                    $table->text($field . '_encrypted')->nullable();
                    break;
                case 'string':
                    $table->string($field . '_encrypted', 500)->nullable();
                    break;
                case 'json':
                    $table->text($field . '_encrypted')->nullable();
                    break;
            }
            
            // Add field hash for searchability
            $table->string($field . '_hash', 64)->nullable()->index();
        }
    }
    
    /**
     * Create encrypted data trigger
     */
    protected function createEncryptionTrigger(string $tableName, array $fields): void
    {
        foreach ($fields as $field => $type) {
            DB::statement("
                CREATE TRIGGER encrypt_{$tableName}_{$field}_trigger
                BEFORE INSERT OR UPDATE ON {$tableName}
                FOR EACH ROW
                BEGIN
                    IF NEW.{$field} IS NOT NULL THEN
                        SET NEW.{$field}_encrypted = AES_ENCRYPT(NEW.{$field}, UNHEX(SHA2(@encryption_key, 256)));
                        SET NEW.{$field}_hash = SHA2(NEW.{$field}, 256);
                        SET NEW.{$field} = NULL;
                    END IF;
                END;
            ");
        }
    }
}
```

### Audit Trail Migration
```php
// 2024_01_01_000010_create_audit_trails_table.php
class CreateAuditTrailsTable extends TenantAwareMigration
{
    protected function defineSchema(Blueprint $table): void
    {
        $table->string('auditable_type', 100);
        $table->unsignedBigInteger('auditable_id');
        $table->string('event', 50);
        $table->unsignedBigInteger('user_id')->nullable();
        $table->json('old_values')->nullable();
        $table->json('new_values')->nullable();
        $table->ipAddress('ip_address')->nullable();
        $table->string('user_agent', 500)->nullable();
        $table->string('session_id', 100)->nullable();
        $table->json('metadata')->nullable();
    }
    
    protected function addTenantConstraints(Blueprint $table): void
    {
        // Composite index for auditable entity
        $table->index(['tenant_id', 'auditable_type', 'auditable_id']);
        
        // Performance indexes
        $table->index(['tenant_id', 'event', 'created_at']);
        $table->index(['tenant_id', 'user_id', 'created_at']);
        $table->index(['tenant_id', 'created_at']);
        
        // Foreign key for user
        $table->foreign(['tenant_id', 'user_id'])
              ->references(['tenant_id', 'id'])
              ->on('users')
              ->onDelete('set null');
    }
}
```

## Migration Testing and Validation

### Migration Testing Framework
```php
// MigrationTestCase.php - Base test class for migration testing
abstract class MigrationTestCase extends TestCase
{
    use RefreshDatabase, WithFaker;
    
    /**
     * Test migration up and down
     */
    public function test_migration_up_and_down(): void
    {
        // Test migration up
        $this->artisan('migrate', ['--path' => $this->getMigrationPath()])
             ->assertExitCode(0);
        
        // Verify table structure
        $this->assertDatabaseHasTable($this->getTableName());
        $this->assertTableStructure();
        
        // Test migration rollback
        $this->artisan('migrate:rollback', ['--path' => $this->getMigrationPath()])
             ->assertExitCode(0);
        
        // Verify table is dropped
        $this->assertDatabaseMissingTable($this->getTableName());
    }
    
    /**
     * Test tenant isolation
     */
    public function test_tenant_isolation(): void
    {
        $tenant1 = Tenant::factory()->create();
        $tenant2 = Tenant::factory()->create();
        
        // Create records for both tenants
        $this->createTenantRecord($tenant1);
        $this->createTenantRecord($tenant2);
        
        // Verify tenant isolation
        $this->assertTenantIsolation($tenant1, $tenant2);
    }
    
    /**
     * Test row-level security
     */
    public function test_row_level_security(): void
    {
        $tenant = Tenant::factory()->create();
        
        // Set tenant context
        DB::statement("SET app.current_tenant_id = ?", [$tenant->id]);
        
        // Create and verify record
        $record = $this->createTenantRecord($tenant);
        $this->assertDatabaseHas($this->getTableName(), ['id' => $record->id]);
        
        // Switch tenant context
        $otherTenant = Tenant::factory()->create();
        DB::statement("SET app.current_tenant_id = ?", [$otherTenant->id]);
        
        // Verify record is not accessible
        $this->assertDatabaseMissing($this->getTableName(), ['id' => $record->id]);
    }
    
    abstract protected function getMigrationPath(): string;
    abstract protected function getTableName(): string;
    abstract protected function assertTableStructure(): void;
    abstract protected function createTenantRecord(Tenant $tenant);
}
```

### Migration Performance Testing
```php
// MigrationPerformanceTest.php - Performance testing for large datasets
class MigrationPerformanceTest extends TestCase
{
    use RefreshDatabase;
    
    /**
     * Test migration performance with large dataset
     */
    public function test_migration_performance_large_dataset(): void
    {
        // Create test data
        $this->createLargeDataset();
        
        // Measure migration time
        $startTime = microtime(true);
        
        $this->artisan('migrate', ['--path' => 'database/migrations/performance'])
             ->assertExitCode(0);
        
        $migrationTime = microtime(true) - $startTime;
        
        // Assert performance requirements
        $this->assertLessThan(30, $migrationTime, 'Migration took too long');
        
        // Verify data integrity
        $this->assertDataIntegrity();
    }
    
    /**
     * Test concurrent migration safety
     */
    public function test_concurrent_migration_safety(): void
    {
        $processes = [];
        
        // Start multiple migration processes
        for ($i = 0; $i < 3; $i++) {
            $processes[] = $this->startMigrationProcess();
        }
        
        // Wait for all processes to complete
        foreach ($processes as $process) {
            $this->waitForProcess($process);
        }
        
        // Verify only one migration succeeded
        $this->assertSingleMigrationSuccess();
    }
}
```

## Environment-Specific Migration Strategies

### Production Migration Safety
```php
// ProductionMigrationCommand.php - Safe production migrations
class ProductionMigrationCommand extends Command
{
    protected $signature = 'migrate:production {--dry-run} {--backup}';
    
    public function handle(): void
    {
        if ($this->option('backup')) {
            $this->createBackup();
        }
        
        if ($this->option('dry-run')) {
            $this->performDryRun();
            return;
        }
        
        // Check maintenance mode
        if (!$this->laravel->isDownForMaintenance()) {
            $this->error('Production migrations require maintenance mode');
            return;
        }
        
        // Validate migration safety
        $this->validateMigrationSafety();
        
        // Execute migrations with monitoring
        $this->executeWithMonitoring();
    }
    
    private function validateMigrationSafety(): void
    {
        $pendingMigrations = $this->getMigrator()->getPendingMigrations();
        
        foreach ($pendingMigrations as $migration) {
            $this->validateMigration($migration);
        }
    }
    
    private function executeWithMonitoring(): void
    {
        $startTime = microtime(true);
        
        try {
            $this->call('migrate', ['--force' => true]);
            
            $duration = microtime(true) - $startTime;
            $this->logMigrationSuccess($duration);
            
        } catch (Exception $e) {
            $this->logMigrationFailure($e);
            
            if ($this->option('backup')) {
                $this->restoreBackup();
            }
            
            throw $e;
        }
    }
}
```

### Multi-Environment Migration Management
```php
// TenantMigrationManager.php - Manage migrations across tenants
class TenantMigrationManager
{
    /**
     * Run migrations for all tenants
     */
    public function migrateAllTenants(): void
    {
        $tenants = Tenant::where('status', 'active')->get();
        
        foreach ($tenants as $tenant) {
            $this->migrateTenant($tenant);
        }
    }
    
    /**
     * Run migrations for specific tenant
     */
    public function migrateTenant(Tenant $tenant): void
    {
        // Set tenant context
        TenantContext::setCurrentTenant($tenant->id);
        
        try {
            // Set database connection for tenant
            $this->setTenantConnection($tenant);
            
            // Run tenant-specific migrations
            Artisan::call('migrate', [
                '--database' => "tenant_{$tenant->id}",
                '--path' => 'database/migrations/tenant',
                '--force' => true
            ]);
            
            // Log successful migration
            Log::info("Tenant migration completed", [
                'tenant_id' => $tenant->id,
                'tenant_name' => $tenant->name
            ]);
            
        } catch (Exception $e) {
            // Log migration failure
            Log::error("Tenant migration failed", [
                'tenant_id' => $tenant->id,
                'error' => $e->getMessage()
            ]);
            
            throw $e;
        } finally {
            // Reset tenant context
            TenantContext::clearCurrentTenant();
        }
    }
    
    /**
     * Rollback migrations for tenant
     */
    public function rollbackTenant(Tenant $tenant, int $steps = 1): void
    {
        TenantContext::setCurrentTenant($tenant->id);
        
        try {
            $this->setTenantConnection($tenant);
            
            Artisan::call('migrate:rollback', [
                '--database' => "tenant_{$tenant->id}",
                '--step' => $steps,
                '--force' => true
            ]);
            
        } finally {
            TenantContext::clearCurrentTenant();
        }
    }
}
```

## Migration Documentation and Standards

### Migration Naming Conventions
- **Format**: `YYYY_MM_DD_HHMMSS_descriptive_migration_name.php`
- **Tenant Tables**: Include "tenant_" prefix or use TenantAwareMigration base class
- **Indexes**: Include purpose in index name (e.g., `policies_tenant_status_idx`)
- **Foreign Keys**: Include referenced table in constraint name

### Migration Best Practices
1. **Atomic Operations**: Each migration should be atomic and reversible
2. **Performance**: Consider impact on large datasets and use chunking for data migrations
3. **Security**: Implement row-level security for tenant isolation
4. **Testing**: Comprehensive test coverage for all migration scenarios
5. **Documentation**: Clear comments explaining complex migration logic
6. **Backward Compatibility**: Ensure migrations don't break existing functionality

This enhanced database migration file provides comprehensive multi-tenant migration strategies with security, performance, and maintainability considerations for the insurance system's evolution from monolith to microservices architecture.