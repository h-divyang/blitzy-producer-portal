# 03.0 Models & Relationships - Updated

## Insurance Domain Model Architecture

### Model Architecture Overview
- **Domain-Driven Design**: Models organized by insurance business domains
- **Multi-Tenant Support**: All models include tenant isolation with row-level security
- **Audit Integration**: Comprehensive audit trails for all model changes
- **Event-Driven Architecture**: Models emit events for microservice coordination
- **Cache Strategy**: Redis 7.x integration for frequently accessed model data

### Technology Stack Integration
- **Laravel Version**: 12.x+ with enhanced Eloquent features
- **PHP Version**: 8.4+ for modern language features and performance
- **Database**: MariaDB 12.x LTS with optimized relationships
- **Cache**: Redis 7.x for model caching and session management
- **Search**: Elasticsearch 8.x for full-text search integration

## Core Insurance Domain Models

### Tenant Management Models

#### Tenant Model
```php
// app/Models/Core/Tenant.php
class Tenant extends Model
{
    use HasFactory, SoftDeletes, LogsActivity;
    
    protected $fillable = [
        'name',
        'domain',
        'database_name',
        'configuration',
        'feature_flags',
        'status',
        'trial_ends_at'
    ];
    
    protected $casts = [
        'configuration' => 'array',
        'feature_flags' => 'array',
        'trial_ends_at' => 'datetime',
        'status' => TenantStatus::class
    ];
    
    protected $attributes = [
        'status' => 'active',
        'configuration' => '{}',
        'feature_flags' => '{}'
    ];
    
    /**
     * Tenant has many users
     */
    public function users(): HasMany
    {
        return $this->hasMany(User::class);
    }
    
    /**
     * Tenant has many policies
     */
    public function policies(): HasMany
    {
        return $this->hasMany(Policy::class);
    }
    
    /**
     * Tenant has many claims
     */
    public function claims(): HasMany
    {
        return $this->hasMany(Claim::class);
    }
    
    /**
     * Tenant configuration accessor
     */
    public function getConfig(string $key, $default = null)
    {
        return data_get($this->configuration, $key, $default);
    }
    
    /**
     * Check if feature is enabled
     */
    public function hasFeature(string $feature): bool
    {
        return data_get($this->feature_flags, $feature, false);
    }
    
    /**
     * Scope for active tenants
     */
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('status', TenantStatus::ACTIVE);
    }
}
```

### User Management Models

#### User Model with Multi-Tenant Support
```php
// app/Models/Core/User.php
class User extends Authenticatable implements MustVerifyEmail
{
    use HasApiTokens, HasFactory, Notifiable, SoftDeletes, 
        LogsActivity, HasRoles, HasPermissions, TenantScoped;
    
    protected $fillable = [
        'tenant_id',
        'email',
        'first_name',
        'last_name',
        'phone',
        'preferences',
        'mfa_secret',
        'mfa_recovery_codes',
        'is_active'
    ];
    
    protected $hidden = [
        'password',
        'remember_token',
        'mfa_secret',
        'mfa_recovery_codes'
    ];
    
    protected $casts = [
        'email_verified_at' => 'datetime',
        'last_login_at' => 'datetime',
        'preferences' => 'array',
        'mfa_recovery_codes' => 'encrypted:array',
        'is_active' => 'boolean'
    ];
    
    /**
     * User belongs to tenant
     */
    public function tenant(): BelongsTo
    {
        return $this->belongsTo(Tenant::class);
    }
    
    /**
     * User has many policies as policyholder
     */
    public function policies(): HasMany
    {
        return $this->hasMany(Policy::class, 'policyholder_id');
    }
    
    /**
     * User has many policies as agent
     */
    public function agentPolicies(): HasMany
    {
        return $this->hasMany(Policy::class, 'agent_id');
    }
    
    /**
     * User has many claims as claimant
     */
    public function claims(): HasMany
    {
        return $this->hasMany(Claim::class, 'claimant_id');
    }
    
    /**
     * User has many claims as adjuster
     */
    public function adjusterClaims(): HasMany
    {
        return $this->hasMany(Claim::class, 'adjuster_id');
    }
    
    /**
     * User profile relationship
     */
    public function profile(): HasOne
    {
        return $this->hasOne(UserProfile::class);
    }
    
    /**
     * User licenses (for agents)
     */
    public function licenses(): HasMany
    {
        return $this->hasMany(UserLicense::class);
    }
    
    /**
     * Get user's full name
     */
    public function getFullNameAttribute(): string
    {
        return "{$this->first_name} {$this->last_name}";
    }
    
    /**
     * Check if user is an agent
     */
    public function isAgent(): bool
    {
        return $this->hasRole('agent');
    }
    
    /**
     * Check if user is an adjuster
     */
    public function isAdjuster(): bool
    {
        return $this->hasRole('claims_adjuster');
    }
    
    /**
     * Scope for active users
     */
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('is_active', true);
    }
    
    /**
     * Scope for agents
     */
    public function scopeAgents(Builder $query): Builder
    {
        return $query->whereHas('roles', function ($q) {
            $q->where('name', 'agent');
        });
    }
}
```

### Policy Management Models

#### Policy Model
```php
// app/Models/Policy/Policy.php
class Policy extends Model
{
    use HasFactory, SoftDeletes, LogsActivity, TenantScoped, 
        HasUuids, GeneratesDocuments, EmitsEvents;
    
    protected $fillable = [
        'tenant_id',
        'policy_number',
        'policyholder_id',
        'agent_id',
        'type',
        'status',
        'premium_amount',
        'coverage_amount',
        'effective_date',
        'expiration_date',
        'coverage_details',
        'risk_factors',
        'deductible',
        'notes'
    ];
    
    protected $casts = [
        'effective_date' => 'date',
        'expiration_date' => 'date',
        'premium_amount' => 'decimal:2',
        'coverage_amount' => 'decimal:2',
        'deductible' => 'decimal:2',
        'coverage_details' => 'array',
        'risk_factors' => 'array',
        'bound_at' => 'datetime',
        'cancelled_at' => 'datetime',
        'type' => PolicyType::class,
        'status' => PolicyStatus::class
    ];
    
    protected $attributes = [
        'status' => 'quote',
        'coverage_details' => '{}',
        'risk_factors' => '{}'
    ];
    
    /**
     * Policy belongs to tenant
     */
    public function tenant(): BelongsTo
    {
        return $this->belongsTo(Tenant::class);
    }
    
    /**
     * Policy belongs to policyholder
     */
    public function policyholder(): BelongsTo
    {
        return $this->belongsTo(User::class, 'policyholder_id');
    }
    
    /**
     * Policy belongs to agent
     */
    public function agent(): BelongsTo
    {
        return $this->belongsTo(User::class, 'agent_id');
    }
    
    /**
     * Policy has many claims
     */
    public function claims(): HasMany
    {
        return $this->hasMany(Claim::class);
    }
    
    /**
     * Policy has many endorsements
     */
    public function endorsements(): HasMany
    {
        return $this->hasMany(PolicyEndorsement::class);
    }
    
    /**
     * Policy has many coverages
     */
    public function coverages(): HasMany
    {
        return $this->hasMany(PolicyCoverage::class);
    }
    
    /**
     * Policy has many documents
     */
    public function documents(): MorphMany
    {
        return $this->morphMany(Document::class, 'documentable');
    }
    
    /**
     * Policy has many payments
     */
    public function payments(): HasMany
    {
        return $this->hasMany(Payment::class);
    }
    
    /**
     * Policy renewal history
     */
    public function renewals(): HasMany
    {
        return $this->hasMany(PolicyRenewal::class);
    }
    
    /**
     * Check if policy is active
     */
    public function isActive(): bool
    {
        return $this->status === PolicyStatus::ACTIVE &&
               $this->effective_date <= now() &&
               $this->expiration_date >= now();
    }
    
    /**
     * Check if policy is expired
     */
    public function isExpired(): bool
    {
        return $this->expiration_date < now();
    }
    
    /**
     * Calculate annual premium
     */
    public function getAnnualPremiumAttribute(): float
    {
        return $this->premium_amount * 12;
    }
    
    /**
     * Get remaining term in days
     */
    public function getRemainingTermAttribute(): int
    {
        return max(0, now()->diffInDays($this->expiration_date));
    }
    
    /**
     * Scope for active policies
     */
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('status', PolicyStatus::ACTIVE);
    }
    
    /**
     * Scope for expiring policies
     */
    public function scopeExpiring(Builder $query, int $days = 30): Builder
    {
        return $query->where('expiration_date', '<=', now()->addDays($days))
                    ->where('expiration_date', '>=', now());
    }
    
    /**
     * Scope by policy type
     */
    public function scopeOfType(Builder $query, PolicyType $type): Builder
    {
        return $query->where('type', $type);
    }
}
```

#### Policy Coverage Model
```php
// app/Models/Policy/PolicyCoverage.php
class PolicyCoverage extends Model
{
    use HasFactory, TenantScoped, LogsActivity;
    
    protected $fillable = [
        'tenant_id',
        'policy_id',
        'coverage_type',
        'coverage_name',
        'limit_amount',
        'deductible_amount',
        'premium_amount',
        'is_active'
    ];
    
    protected $casts = [
        'limit_amount' => 'decimal:2',
        'deductible_amount' => 'decimal:2',
        'premium_amount' => 'decimal:2',
        'is_active' => 'boolean',
        'coverage_type' => CoverageType::class
    ];
    
    /**
     * Coverage belongs to policy
     */
    public function policy(): BelongsTo
    {
        return $this->belongsTo(Policy::class);
    }
    
    /**
     * Coverage has many claims
     */
    public function claims(): HasMany
    {
        return $this->hasMany(Claim::class, 'coverage_id');
    }
    
    /**
     * Scope for active coverages
     */
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('is_active', true);
    }
}
```

### Claims Management Models

#### Claim Model
```php
// app/Models/Claims/Claim.php
class Claim extends Model
{
    use HasFactory, SoftDeletes, LogsActivity, TenantScoped, 
        HasUuids, HasStateMachine, EmitsEvents;
    
    protected $fillable = [
        'tenant_id',
        'claim_number',
        'policy_id',
        'coverage_id',
        'claimant_id',
        'adjuster_id',
        'status',
        'type',
        'incident_date',
        'description',
        'claimed_amount',
        'approved_amount',
        'paid_amount',
        'incident_details',
        'supporting_documents',
        'adjuster_notes'
    ];
    
    protected $casts = [
        'incident_date' => 'date',
        'claimed_amount' => 'decimal:2',
        'approved_amount' => 'decimal:2',
        'paid_amount' => 'decimal:2',
        'incident_details' => 'array',
        'supporting_documents' => 'array',
        'reported_at' => 'datetime',
        'closed_at' => 'datetime',
        'status' => ClaimStatus::class,
        'type' => ClaimType::class
    ];
    
    protected $attributes = [
        'status' => 'reported',
        'paid_amount' => 0,
        'incident_details' => '{}',
        'supporting_documents' => '[]'
    ];
    
    /**
     * Claim belongs to policy
     */
    public function policy(): BelongsTo
    {
        return $this->belongsTo(Policy::class);
    }
    
    /**
     * Claim belongs to coverage
     */
    public function coverage(): BelongsTo
    {
        return $this->belongsTo(PolicyCoverage::class);
    }
    
    /**
     * Claim belongs to claimant
     */
    public function claimant(): BelongsTo
    {
        return $this->belongsTo(User::class, 'claimant_id');
    }
    
    /**
     * Claim belongs to adjuster
     */
    public function adjuster(): BelongsTo
    {
        return $this->belongsTo(User::class, 'adjuster_id');
    }
    
    /**
     * Claim has many items
     */
    public function items(): HasMany
    {
        return $this->hasMany(ClaimItem::class);
    }
    
    /**
     * Claim has many payments
     */
    public function payments(): HasMany
    {
        return $this->hasMany(ClaimPayment::class);
    }
    
    /**
     * Claim has many documents
     */
    public function documents(): MorphMany
    {
        return $this->morphMany(Document::class, 'documentable');
    }
    
    /**
     * Claim has many notes
     */
    public function notes(): HasMany
    {
        return $this->hasMany(ClaimNote::class);
    }
    
    /**
     * Get remaining balance to be paid
     */
    public function getRemainingBalanceAttribute(): float
    {
        return max(0, ($this->approved_amount ?? 0) - $this->paid_amount);
    }
    
    /**
     * Check if claim is within policy limits
     */
    public function isWithinPolicyLimits(): bool
    {
        return $this->claimed_amount <= $this->coverage->limit_amount;
    }
    
    /**
     * Check if claim is fully paid
     */
    public function isFullyPaid(): bool
    {
        return $this->approved_amount > 0 && 
               $this->paid_amount >= $this->approved_amount;
    }
    
    /**
     * Scope for open claims
     */
    public function scopeOpen(Builder $query): Builder
    {
        return $query->whereNotIn('status', [
            ClaimStatus::CLOSED,
            ClaimStatus::DENIED
        ]);
    }
    
    /**
     * Scope for high-value claims
     */
    public function scopeHighValue(Builder $query, float $threshold = 10000): Builder
    {
        return $query->where('claimed_amount', '>=', $threshold);
    }
}
```

### Financial Models

#### Payment Model
```php
// app/Models/Financial/Payment.php
class Payment extends Model
{
    use HasFactory, TenantScoped, LogsActivity, HasUuids;
    
    protected $fillable = [
        'tenant_id',
        'payable_type',
        'payable_id',
        'amount',
        'currency',
        'payment_method',
        'payment_reference',
        'status',
        'processed_at',
        'gateway_response',
        'fee_amount',
        'net_amount'
    ];
    
    protected $casts = [
        'amount' => 'decimal:2',
        'fee_amount' => 'decimal:2',
        'net_amount' => 'decimal:2',
        'processed_at' => 'datetime',
        'gateway_response' => 'array',
        'status' => PaymentStatus::class,
        'payment_method' => PaymentMethod::class
    ];
    
    /**
     * Payment belongs to payable (Policy, Claim, etc.)
     */
    public function payable(): MorphTo
    {
        return $this->morphTo();
    }
    
    /**
     * Payment belongs to user
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
    
    /**
     * Check if payment is successful
     */
    public function isSuccessful(): bool
    {
        return $this->status === PaymentStatus::COMPLETED;
    }
    
    /**
     * Scope for successful payments
     */
    public function scopeSuccessful(Builder $query): Builder
    {
        return $query->where('status', PaymentStatus::COMPLETED);
    }
}
```

## Model Relationships and Caching Strategy

### Redis Cache Integration
```php
// app/Traits/CachesModel.php
trait CachesModel
{
    protected static function bootCachesModel(): void
    {
        static::saved(function ($model) {
            $model->clearModelCache();
        });
        
        static::deleted(function ($model) {
            $model->clearModelCache();
        });
    }
    
    /**
     * Get cached model data
     */
    public function getCached(string $key, callable $callback, int $ttl = 3600)
    {
        $cacheKey = $this->getCacheKey($key);
        
        return Cache::remember($cacheKey, $ttl, $callback);
    }
    
    /**
     * Clear model cache
     */
    public function clearModelCache(): void
    {
        $pattern = $this->getCacheKey('*');
        
        foreach (Cache::get($pattern) as $key) {
            Cache::forget($key);
        }
    }
    
    /**
     * Generate cache key
     */
    protected function getCacheKey(string $suffix): string
    {
        return sprintf(
            '%s:%s:%s:%s',
            config('app.name'),
            $this->getTable(),
            $this->getKey(),
            $suffix
        );
    }
}
```

### Event-Driven Model Integration
```php
// app/Traits/EmitsEvents.php
trait EmitsEvents
{
    protected static function bootEmitsEvents(): void
    {
        static::created(function ($model) {
            event(new ModelCreated($model));
        });
        
        static::updated(function ($model) {
            event(new ModelUpdated($model));
        });
        
        static::deleted(function ($model) {
            event(new ModelDeleted($model));
        });
    }
    
    /**
     * Emit custom domain event
     */
    public function emitEvent(string $eventClass, array $payload = []): void
    {
        event(new $eventClass($this, $payload));
    }
}
```

## Advanced Model Features

### Multi-Database Support
```php
// app/Traits/TenantScoped.php
trait TenantScoped
{
    protected static function bootTenantScoped(): void
    {
        static::addGlobalScope(new TenantScope);
        
        static::creating(function ($model) {
            if (!$model->tenant_id) {
                $model->tenant_id = TenantContext::getCurrentTenantId();
            }
        });
    }
    
    /**
     * Tenant relationship
     */
    public function tenant(): BelongsTo
    {
        return $this->belongsTo(Tenant::class);
    }
    
    /**
     * Scope without tenant isolation (use carefully)
     */
    public function scopeWithoutTenantScope(Builder $query): Builder
    {
        return $query->withoutGlobalScope(TenantScope::class);
    }
}
```

### Document Management Integration
```php
// app/Models/Core/Document.php
class Document extends Model
{
    use HasFactory, TenantScoped, LogsActivity, HasUuids;
    
    protected $fillable = [
        'tenant_id',
        'documentable_type',
        'documentable_id',
        'name',
        'file_path',
        'file_size',
        'mime_type',
        'storage_driver',
        'metadata',
        'is_encrypted'
    ];
    
    protected $casts = [
        'file_size' => 'integer',
        'metadata' => 'array',
        'is_encrypted' => 'boolean'
    ];
    
    /**
     * Document belongs to documentable
     */
    public function documentable(): MorphTo
    {
        return $this->morphTo();
    }
    
    /**
     * Get document URL
     */
    public function getUrlAttribute(): string
    {
        return Storage::disk($this->storage_driver)->url($this->file_path);
    }
    
    /**
     * Get formatted file size
     */
    public function getFormattedSizeAttribute(): string
    {
        return $this->formatBytes($this->file_size);
    }
    
    private function formatBytes(int $bytes): string
    {
        $units = ['B', 'KB', 'MB', 'GB'];
        
        for ($i = 0; $bytes > 1024 && $i < count($units) - 1; $i++) {
            $bytes /= 1024;
        }
        
        return round($bytes, 2) . ' ' . $units[$i];
    }
}
```

## Model Testing and Validation

### Model Testing Framework
```php
// tests/Unit/Models/PolicyTest.php
class PolicyTest extends TestCase
{
    use RefreshDatabase, WithFaker;
    
    public function test_policy_relationships(): void
    {
        $tenant = Tenant::factory()->create();
        $policyholder = User::factory()->for($tenant)->create();
        $agent = User::factory()->for($tenant)->agent()->create();
        
        $policy = Policy::factory()
            ->for($tenant)
            ->for($policyholder, 'policyholder')
            ->for($agent, 'agent')
            ->create();
        
        $this->assertInstanceOf(Tenant::class, $policy->tenant);
        $this->assertInstanceOf(User::class, $policy->policyholder);
        $this->assertInstanceOf(User::class, $policy->agent);
        $this->assertEquals($tenant->id, $policy->tenant_id);
    }
    
    public function test_policy_scopes(): void
    {
        $tenant = Tenant::factory()->create();
        
        Policy::factory()
            ->for($tenant)
            ->active()
            ->count(5)
            ->create();
            
        Policy::factory()
            ->for($tenant)
            ->expired()
            ->count(3)
            ->create();
        
        $this->assertEquals(5, Policy::active()->count());
        $this->assertEquals(3, Policy::expiring(0)->count());
    }
    
    public function test_tenant_isolation(): void
    {
        $tenant1 = Tenant::factory()->create();
        $tenant2 = Tenant::factory()->create();
        
        TenantContext::setCurrentTenant($tenant1->id);
        
        $policy1 = Policy::factory()->for($tenant1)->create();
        $policy2 = Policy::factory()->for($tenant2)->create();
        
        $this->assertEquals(1, Policy::count());
        $this->assertEquals($policy1->id, Policy::first()->id);
    }
}
```

This comprehensive models and relationships file provides the complete insurance domain model architecture with multi-tenant support, caching strategies, event-driven integration, and comprehensive relationship management for the Laravel 11.x+ application.