# 01.0 Identity & Access Management (IAM) - Updated

## Authentication Framework Overview

### Unified Authentication Strategy
- **OAuth2 Foundation**: Laravel Passport for enterprise-grade OAuth2 server implementation
- **SPA Integration**: Laravel Sanctum for React frontend and mobile application authentication
- **Multi-Tenant Security**: Comprehensive tenant isolation with token-based access control
- **Microservice Ready**: Authentication strategy supporting monolith-to-microservice evolution
- **Zero Trust Model**: All service communications secured with mTLS via Istio service mesh

## Technology Stack Integration

### Backend Authentication (Laravel 12.x+)
- **Laravel Passport**: 12.x+ compatible version for OAuth2 server implementation
- **Laravel Sanctum**: Latest version for SPA and API token authentication
- **PHP Version**: 8.4+ for enhanced security and performance features
- **Database**: MariaDB 12.x LTS with encrypted authentication tables
- **Cache**: Redis 7.x for session management and token caching

### Frontend Authentication (React 18+)
- **React**: 18.2+ for modern authentication patterns
- **TypeScript**: 5.x for type-safe authentication components
- **State Management**: React Query for authentication state and caching
- **HTTP Client**: Axios with automatic token refresh and error handling
- **Security**: HTTPS-only with secure cookie handling

## Multi-Tenant Authentication Architecture

### Tenant Isolation Strategy
```php
// TenantAuthenticationService.php - Comprehensive tenant-aware authentication
class TenantAuthenticationService
{
    /**
     * Authenticate user with tenant context validation
     */
    public function authenticateWithTenant(array $credentials, ?int $tenantId = null): AuthenticationResult
    {
        // Validate credentials
        if (!Auth::attempt($credentials)) {
            $this->logFailedAttempt($credentials['email'], request()->ip());
            throw new AuthenticationException('Invalid credentials');
        }

        $user = Auth::user();
        
        // Validate tenant access
        if ($tenantId && $user->tenant_id !== $tenantId) {
            $this->logTenantViolation($user->id, $tenantId);
            throw new TenantAccessDeniedException('User does not have access to specified tenant');
        }

        // Generate tenant-specific tokens
        $scopes = $this->getTenantSpecificScopes($user);
        $accessToken = $user->createToken('insurance-app', $scopes);
        
        // Create comprehensive audit trail
        AuditTrail::create([
            'user_id' => $user->id,
            'tenant_id' => $user->tenant_id,
            'action' => 'authentication_success',
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'metadata' => [
                'scopes' => $scopes,
                'token_type' => 'passport',
                'expires_at' => $accessToken->token->expires_at
            ]
        ]);

        return new AuthenticationResult($user, $accessToken->accessToken, $scopes);
    }

    /**
     * Generate tenant-specific OAuth2 scopes
     */
    private function getTenantSpecificScopes(User $user): array
    {
        $baseScopes = $user->roles->flatMap->permissions->pluck('name')->unique();
        
        // Add tenant-specific scopes
        $tenantScopes = [
            "tenant:{$user->tenant_id}:access",
            "tenant:{$user->tenant_id}:read",
        ];

        // Add role-specific scopes
        if ($user->hasRole('admin')) {
            $tenantScopes[] = "tenant:{$user->tenant_id}:admin";
        }

        return $baseScopes->concat($tenantScopes)->toArray();
    }
}
```

### Token Management with Multi-Tenancy
```php
// TenantTokenGuard.php - Custom guard for tenant-aware token validation
class TenantTokenGuard extends TokenGuard
{
    /**
     * Validate token with tenant context
     */
    public function user()
    {
        if ($this->user !== null) {
            return $this->user;
        }

        $user = null;
        $token = $this->getTokenForRequest();

        if (!empty($token)) {
            // Validate token and extract tenant information
            $tokenModel = $this->provider->retrieveByToken($token);
            
            if ($tokenModel && $this->validateTenantAccess($tokenModel)) {
                $user = $tokenModel->user;
                
                // Set tenant context for request
                TenantContext::setCurrentTenant($user->tenant_id);
                
                // Log token usage for audit
                TokenUsageLog::create([
                    'token_id' => $tokenModel->id,
                    'user_id' => $user->id,
                    'tenant_id' => $user->tenant_id,
                    'accessed_at' => now(),
                    'ip_address' => request()->ip(),
                    'endpoint' => request()->path()
                ]);
            }
        }

        return $this->user = $user;
    }

    /**
     * Validate tenant access for token
     */
    private function validateTenantAccess($token): bool
    {
        $requestedTenant = request()->header('X-Tenant-ID');
        
        if ($requestedTenant && $token->user->tenant_id !== (int)$requestedTenant) {
            Log::warning('Cross-tenant access attempt detected', [
                'user_id' => $token->user->id,
                'user_tenant' => $token->user->tenant_id,
                'requested_tenant' => $requestedTenant,
                'ip_address' => request()->ip()
            ]);
            
            return false;
        }

        return true;
    }
}
```

## Kong API Gateway Security Integration

### API Gateway Authentication Layer
```yaml
# Kong API Gateway Configuration for Insurance System
services:
  - name: insurance-api
    url: http://laravel-backend:8000
    plugins:
      - name: jwt
        config:
          uri_param_names: [jwt]
          cookie_names: [jwt]
          header_names: [Authorization]
          claims_to_verify: [exp, iss, aud, tenant_id]
          key_claim_name: iss
          run_on_preflight: true
          
      - name: oauth2
        config:
          scopes: [policy.read, policy.create, claim.read, claim.create, admin]
          mandatory_scope: true
          token_expiration: 3600
          enable_authorization_code: true
          enable_client_credentials: true
          hide_credentials: true
          
      - name: rate-limiting
        config:
          minute: 1000
          hour: 10000
          policy: redis
          redis_host: redis.insurance.internal
          redis_port: 6379
          
      - name: request-transformer
        config:
          add:
            headers:
              - "X-Tenant-ID:$(headers.x-tenant-id)"
              - "X-User-ID:$(headers.x-user-id)"
```

### Kong + Laravel Integration
```php
// app/Http/Middleware/KongAuthenticationMiddleware.php
class KongAuthenticationMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        // Extract Kong-validated user information
        $kongUserId = $request->header('X-Authenticated-UserId');
        $kongScopes = explode(',', $request->header('X-Authenticated-Scope', ''));
        $tenantId = $request->header('X-Tenant-ID');
        
        if (!$kongUserId || !$tenantId) {
            throw new AuthenticationException('Invalid Kong authentication');
        }
        
        // Set Laravel authentication context
        $user = User::where('id', $kongUserId)
                   ->where('tenant_id', $tenantId)
                   ->first();
                   
        if (!$user) {
            throw new AuthenticationException('User not found or invalid tenant');
        }
        
        // Set authenticated user
        Auth::setUser($user);
        
        // Set tenant context
        TenantContext::setCurrentTenant($tenantId);
        
        // Validate scopes against user permissions
        $this->validateUserScopes($user, $kongScopes);
        
        return $next($request);
    }
}
```

## Role-Based Access Control (RBAC)

### Insurance Domain User Hierarchy
The system implements a comprehensive RBAC system that supports both internal users and external stakeholders with granular permissions based on insurance industry roles.

#### User Types and Classification
```php
// app/Models/Core/UserType.php
class UserType extends Model
{
    const INTERNAL_STAFF = 'internal_staff';
    const PRODUCER = 'producer';
    const INSURED = 'insured';
    const THIRD_PARTY = 'third_party';
    
    protected $fillable = ['name', 'description', 'access_level'];
    
    /**
     * Define insurance-specific user types
     */
    public static function getInsuranceUserTypes(): array
    {
        return [
            self::INTERNAL_STAFF => [
                'description' => 'Insurance company employees and contractors',
                'roles' => ['super_admin', 'tenant_admin', 'underwriter', 'claims_adjuster', 'customer_service']
            ],
            self::PRODUCER => [
                'description' => 'Insurance agents, brokers, and agencies',
                'roles' => ['agent', 'broker', 'agency_admin', 'producer_assistant']
            ],
            self::INSURED => [
                'description' => 'Policyholders and beneficiaries',
                'roles' => ['policyholder', 'beneficiary', 'named_insured']
            ],
            self::THIRD_PARTY => [
                'description' => 'External service providers and vendors',
                'roles' => ['vendor', 'service_provider', 'auditor']
            ]
        ];
    }
}
```

#### User Groups and Feature-Based Permissions
```php
// app/Models/Core/UserGroup.php
class UserGroup extends Model
{
    use TenantScoped, LogsActivity;
    
    protected $fillable = [
        'tenant_id', 'name', 'description', 'group_type', 'hierarchy_level'
    ];
    
    /**
     * User group belongs to many users
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class, 'map_user_group')
                    ->withTimestamps()
                    ->withPivot(['assigned_at', 'assigned_by']);
    }
    
    /**
     * User group has many feature permissions
     */
    public function featurePermissions(): BelongsToMany
    {
        return $this->belongsToMany(Feature::class, 'map_user_group_feature_permission')
                    ->withPivot(['permission_id', 'granted_at', 'granted_by'])
                    ->withTimestamps();
    }
    
    /**
     * Get all permissions for this group
     */
    public function getAllPermissions(): Collection
    {
        return $this->featurePermissions()
                    ->with('permissions')
                    ->get()
                    ->flatMap->permissions
                    ->unique('id');
    }
}

// app/Models/Core/Feature.php
class Feature extends Model
{
    protected $fillable = [
        'name', 'description', 'feature_type_id', 'module', 'is_active'
    ];
    
    /**
     * Insurance system features
     */
    const INSURANCE_FEATURES = [
        'policy_management' => [
            'name' => 'Policy Management',
            'permissions' => ['view', 'create', 'update', 'bind', 'cancel', 'renew']
        ],
        'claims_processing' => [
            'name' => 'Claims Processing', 
            'permissions' => ['view', 'create', 'investigate', 'adjust', 'settle', 'deny']
        ],
        'underwriting' => [
            'name' => 'Underwriting',
            'permissions' => ['view', 'review', 'approve', 'decline', 'refer']
        ],
        'financial_management' => [
            'name' => 'Financial Management',
            'permissions' => ['view', 'process_payments', 'calculate_commissions', 'generate_invoices']
        ],
        'reporting_analytics' => [
            'name' => 'Reporting & Analytics',
            'permissions' => ['view', 'generate', 'export', 'schedule']
        ],
        'administration' => [
            'name' => 'System Administration',
            'permissions' => ['view', 'manage_users', 'configure_system', 'audit_access']
        ]
    ];
    
    /**
     * Feature has many permissions
     */
    public function permissions(): BelongsToMany
    {
        return $this->belongsToMany(Permission::class, 'feature_permissions');
    }
}
```

### Producer Management and Access Control
The system includes comprehensive producer (agent/broker) management with hierarchical access control and producer group assignments.

```php
// app/Models/Producer/Producer.php
class Producer extends Model
{
    use TenantScoped, LogsActivity, HasUuids;
    
    protected $fillable = [
        'tenant_id', 'producer_code_id', 'name', 'type_id', 
        'license_number', 'appointment_status', 'contract_terms'
    ];
    
    /**
     * Producer belongs to producer code
     */
    public function producerCode(): BelongsTo
    {
        return $this->belongsTo(ProducerCode::class);
    }
    
    /**
     * Producer has many users
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class, 'map_user_producer')
                    ->withTimestamps()
                    ->withPivot(['role', 'access_level', 'assigned_at']);
    }
    
    /**
     * Producer belongs to many producer groups
     */
    public function producerGroups(): BelongsToMany
    {
        return $this->belongsToMany(ProducerGroup::class, 'map_producer_group')
                    ->withTimestamps()
                    ->withPivot(['membership_type', 'hierarchy_level']);
    }
    
    /**
     * Get aggregated permissions from all producer groups
     */
    public function getAggregatedPermissions(): Collection
    {
        return $this->producerGroups()
                    ->with(['featurePermissions.permissions'])
                    ->get()
                    ->flatMap(function ($group) {
                        return $group->featurePermissions->flatMap->permissions;
                    })
                    ->unique('id');
    }
}

// app/Models/Producer/ProducerGroup.php
class ProducerGroup extends Model
{
    use TenantScoped, LogsActivity;
    
    protected $fillable = [
        'tenant_id', 'producer_group_type_id', 'code', 'name', 
        'description', 'hierarchy_level', 'commission_structure'
    ];
    
    /**
     * Producer group has feature permissions
     */
    public function featurePermissions(): BelongsToMany
    {
        return $this->belongsToMany(Feature::class, 'map_producer_group_feature_permissions')
                    ->withPivot(['permission_id', 'scope_restrictions'])
                    ->withTimestamps();
    }
    
    /**
     * Producer group types (Regional, National, MGA, etc.)
     */
    public function groupType(): BelongsTo
    {
        return $this->belongsTo(ProducerGroupType::class, 'producer_group_type_id');
    }
}
```

### Advanced Permission Validation System
```php
// app/Services/PermissionValidationService.php
class PermissionValidationService
{
    /**
     * Comprehensive permission check with context awareness
     */
    public function validateUserPermission(
        User $user, 
        string $feature, 
        string $permission, 
        array $context = []
    ): bool {
        // 1. Check direct user permissions
        if ($user->hasDirectPermissionTo("{$feature}.{$permission}")) {
            return $this->validateContext($user, $feature, $permission, $context);
        }
        
        // 2. Check user group permissions
        $groupPermissions = $this->getUserGroupPermissions($user, $feature, $permission);
        if ($groupPermissions->isNotEmpty()) {
            return $this->validateContext($user, $feature, $permission, $context);
        }
        
        // 3. Check producer-based permissions
        if ($user->isProducer()) {
            $producerPermissions = $this->getProducerPermissions($user, $feature, $permission);
            if ($producerPermissions->isNotEmpty()) {
                return $this->validateProducerContext($user, $feature, $permission, $context);
            }
        }
        
        return false;
    }
    
    /**
     * Validate producer-specific context
     */
    private function validateProducerContext(
        User $user, 
        string $feature, 
        string $permission, 
        array $context
    ): bool {
        // Validate producer can access specific policies/claims
        if (isset($context['policy_id'])) {
            $policy = Policy::find($context['policy_id']);
            return $policy && $policy->agent_id === $user->id;
        }
        
        if (isset($context['claim_id'])) {
            $claim = Claim::find($context['claim_id']);
            return $claim && $claim->policy->agent_id === $user->id;
        }
        
        return true;
    }
    
    /**
     * Get user permissions through group memberships
     */
    private function getUserGroupPermissions(User $user, string $feature, string $permission): Collection
    {
        return $user->userGroups()
                   ->whereHas('featurePermissions', function ($query) use ($feature, $permission) {
                       $query->where('features.name', $feature)
                             ->whereHas('permissions', function ($permQuery) use ($permission) {
                                 $permQuery->where('name', $permission);
                             });
                   })
                   ->get();
    }
}
```

### Comprehensive Permission System
```php
// InsurancePermissionSystem.php - Insurance-specific RBAC implementation
class InsurancePermissionSystem
{
    /**
     * Insurance industry role definitions
     */
    const ROLES = [
        'super_admin' => 'System administrator with full access',
        'tenant_admin' => 'Tenant administrator with full tenant access',
        'underwriter' => 'Insurance underwriter with policy approval rights',
        'agent' => 'Insurance agent with policy creation and management',
        'customer_service' => 'Customer service representative',
        'claims_adjuster' => 'Claims processing and adjustment',
        'insured' => 'Policy holder with limited self-service access',
        'read_only' => 'Read-only access for reporting and analytics'
    ];

    /**
     * Permission matrix for insurance operations
     */
    const PERMISSIONS = [
        // Policy Management
        'policy.create' => 'Create new insurance policies',
        'policy.read' => 'View policy information',
        'policy.update' => 'Modify existing policies',
        'policy.delete' => 'Delete policies',
        'policy.bind' => 'Bind quoted policies',
        'policy.cancel' => 'Cancel active policies',
        'policy.renew' => 'Process policy renewals',
        
        // Claims Management
        'claim.create' => 'Create new claims',
        'claim.read' => 'View claim information',
        'claim.update' => 'Update claim details',
        'claim.approve' => 'Approve claim payments',
        'claim.investigate' => 'Investigate claims',
        
        // Financial Operations
        'billing.read' => 'View billing information',
        'billing.process' => 'Process payments and refunds',
        'commission.read' => 'View commission information',
        'commission.calculate' => 'Calculate commissions',
        
        // Administrative
        'user.manage' => 'Manage user accounts',
        'tenant.configure' => 'Configure tenant settings',
        'report.generate' => 'Generate reports',
        'audit.access' => 'Access audit logs',
    ];

    /**
     * Check comprehensive permission with context
     */
    public function checkPermission(User $user, string $permission, array $context = []): bool
    {
        // Basic permission check
        if (!$user->hasPermissionTo($permission)) {
            return false;
        }

        // Context-specific validation
        return match($permission) {
            'policy.update', 'policy.delete' => $this->validatePolicyAccess($user, $context),
            'claim.approve' => $this->validateClaimApprovalLimits($user, $context),
            'billing.process' => $this->validateFinancialLimits($user, $context),
            default => true
        };
    }

    /**
     * Validate policy-specific access
     */
    private function validatePolicyAccess(User $user, array $context): bool
    {
        if (!isset($context['policy_id'])) {
            return false;
        }

        $policy = Policy::find($context['policy_id']);
        
        // Tenant isolation check
        if ($policy->tenant_id !== $user->tenant_id) {
            return false;
        }

        // Agent can only modify their own policies
        if ($user->hasRole('agent') && $policy->agent_id !== $user->id) {
            return false;
        }

        // Additional business rules
        if ($policy->status === 'cancelled' && !$user->hasRole(['underwriter', 'tenant_admin'])) {
            return false;
        }

        return true;
    }
}
```

### Dynamic Permission Assignment
```php
// DynamicPermissionService.php - Context-aware permission management
class DynamicPermissionService
{
    /**
     * Assign permissions based on user context and business rules
     */
    public function assignContextualPermissions(User $user): void
    {
        $permissions = collect();

        // Base permissions by role
        $rolePermissions = $this->getRolePermissions($user->roles);
        $permissions = $permissions->merge($rolePermissions);

        // Tenant-specific permissions
        $tenantPermissions = $this->getTenantPermissions($user->tenant);
        $permissions = $permissions->merge($tenantPermissions);

        // Time-based permissions (business hours, etc.)
        $temporalPermissions = $this->getTemporalPermissions($user);
        $permissions = $permissions->merge($temporalPermissions);

        // License-based permissions (for agents)
        if ($user->hasRole('agent')) {
            $licensePermissions = $this->getLicenseBasedPermissions($user);
            $permissions = $permissions->merge($licensePermissions);
        }

        // Update user permissions
        $user->syncPermissions($permissions->unique());
        
        // Log permission changes
        PermissionAuditLog::create([
            'user_id' => $user->id,
            'tenant_id' => $user->tenant_id,
            'permissions_assigned' => $permissions->toArray(),
            'assigned_at' => now(),
            'context' => 'dynamic_assignment'
        ]);
    }
}
```

## Multi-Factor Authentication (MFA)

### Comprehensive MFA Implementation
```php
// MfaService.php - Multi-factor authentication service
class MfaService
{
    /**
     * Initiate MFA challenge
     */
    public function initiateMfaChallenge(User $user, string $method = 'totp'): MfaChallenge
    {
        $challenge = match($method) {
            'totp' => $this->generateTotpChallenge($user),
            'sms' => $this->sendSmsChallenge($user),
            'email' => $this->sendEmailChallenge($user),
            'hardware' => $this->initiateHardwareChallenge($user),
            default => throw new InvalidArgumentException("Unsupported MFA method: {$method}")
        };

        // Log MFA attempt
        MfaAttemptLog::create([
            'user_id' => $user->id,
            'tenant_id' => $user->tenant_id,
            'method' => $method,
            'challenge_id' => $challenge->id,
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'initiated_at' => now()
        ]);

        return $challenge;
    }

    /**
     * Validate TOTP challenge
     */
    private function generateTotpChallenge(User $user): TotpChallenge
    {
        if (!$user->mfa_secret) {
            throw new MfaNotConfiguredException('TOTP not configured for user');
        }

        return TotpChallenge::create([
            'user_id' => $user->id,
            'expires_at' => now()->addMinutes(5),
            'challenge_hash' => hash('sha256', $user->id . now()->timestamp)
        ]);
    }

    /**
     * Verify MFA response
     */
    public function verifyMfaResponse(string $challengeId, string $response): bool
    {
        $challenge = MfaChallenge::findOrFail($challengeId);
        
        if ($challenge->isExpired()) {
            return false;
        }

        $isValid = match($challenge->method) {
            'totp' => $this->verifyTotpResponse($challenge, $response),
            'sms', 'email' => $this->verifyCodeResponse($challenge, $response),
            'hardware' => $this->verifyHardwareResponse($challenge, $response),
            default => false
        };

        // Log verification attempt
        MfaVerificationLog::create([
            'challenge_id' => $challenge->id,
            'user_id' => $challenge->user_id,
            'success' => $isValid,
            'verified_at' => now(),
            'ip_address' => request()->ip()
        ]);

        if ($isValid) {
            $challenge->markAsUsed();
        }

        return $isValid;
    }
}
```

## Microservice Authentication Integration

### Service-to-Service Authentication
```php
// MicroserviceAuthService.php - Service mesh authentication
class MicroserviceAuthService
{
    /**
     * Generate service-to-service authentication token
     */
    public function generateServiceToken(string $fromService, string $toService): string
    {
        $claims = [
            'iss' => $fromService,
            'aud' => $toService,
            'iat' => now()->timestamp,
            'exp' => now()->addHours(1)->timestamp,
            'scope' => $this->getServicePermissions($fromService, $toService),
            'tenant_id' => TenantContext::getCurrentTenantId()
        ];

        return JWT::encode($claims, config('jwt.private_key'), 'RS256');
    }

    /**
     * Validate service-to-service token
     */
    public function validateServiceToken(string $token): bool
    {
        try {
            $payload = JWT::decode($token, config('jwt.public_key'), ['RS256']);
            
            // Validate service permissions
            return $this->validateServicePermissions(
                $payload->iss,
                $payload->aud,
                $payload->scope
            );
        } catch (Exception $e) {
            Log::warning('Service token validation failed', [
                'error' => $e->getMessage(),
                'token_preview' => substr($token, 0, 20) . '...'
            ]);
            
            return false;
        }
    }

    /**
     * Get allowed permissions between services
     */
    private function getServicePermissions(string $fromService, string $toService): array
    {
        $serviceMatrix = [
            'policy-service' => [
                'user-service' => ['user.read', 'profile.read'],
                'billing-service' => ['billing.create', 'billing.read'],
                'document-service' => ['document.create', 'document.read']
            ],
            'claims-service' => [
                'policy-service' => ['policy.read'],
                'document-service' => ['document.create', 'document.read'],
                'communication-service' => ['notification.send']
            ],
            // Additional service permissions...
        ];

        return $serviceMatrix[$fromService][$toService] ?? [];
    }
}
```

### API Gateway Integration (Kong)
```yaml
# Kong authentication plugin configuration
plugins:
  - name: jwt
    config:
      uri_param_names: [jwt]
      cookie_names: [jwt]
      header_names: [Authorization]
      claims_to_verify: [exp, iss, aud]
      key_claim_name: iss
      secret_is_base64: false
      anonymous: null
      run_on_preflight: true
      
  - name: oauth2
    config:
      scopes: [read, write, admin]
      mandatory_scope: true
      provision_key: provision123
      token_expiration: 3600
      enable_authorization_code: true
      enable_client_credentials: true
      enable_implicit_grant: false
      enable_password_grant: true
      hide_credentials: true
      accept_http_if_already_terminated: true

  - name: rate-limiting
    config:
      minute: 1000
      hour: 10000
      policy: redis
      hide_client_headers: false
      redis_host: redis.insurance.local
      redis_port: 6379
      redis_timeout: 2000
```

## Frontend Authentication Integration (React 18+)

### Authentication Context Provider
```typescript
// AuthContext.tsx - Comprehensive authentication context
interface AuthContextType {
  user: User | null;
  tenant: Tenant | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  permissions: string[];
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
  refreshToken: () => Promise<void>;
  hasPermission: (permission: string) => boolean;
  switchTenant: (tenantId: number) => Promise<void>;
}

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [tenant, setTenant] = useState<Tenant | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  // Token management with automatic refresh
  const { data: authData, mutate: refreshAuth } = useQuery({
    queryKey: ['auth', 'current-user'],
    queryFn: getCurrentUser,
    staleTime: 5 * 60 * 1000, // 5 minutes
    retry: false,
    onSuccess: (data) => {
      setUser(data.user);
      setTenant(data.tenant);
      setIsLoading(false);
    },
    onError: () => {
      setUser(null);
      setTenant(null);
      setIsLoading(false);
      // Redirect to login
      window.location.href = '/login';
    }
  });

  const login = async (credentials: LoginCredentials) => {
    try {
      const response = await authApi.login(credentials);
      
      // Store tokens securely
      tokenStorage.setAccessToken(response.access_token);
      tokenStorage.setRefreshToken(response.refresh_token);
      
      // Refresh user data
      await refreshAuth();
      
      // Log successful login
      analyticsService.track('user_login', {
        user_id: response.user.id,
        tenant_id: response.user.tenant_id,
        login_method: 'password'
      });
      
    } catch (error) {
      console.error('Login failed:', error);
      throw error;
    }
  };

  const logout = async () => {
    try {
      await authApi.logout();
    } catch (error) {
      console.error('Logout request failed:', error);
    } finally {
      // Clear tokens and state
      tokenStorage.clearTokens();
      setUser(null);
      setTenant(null);
      
      // Redirect to login
      window.location.href = '/login';
    }
  };

  const hasPermission = (permission: string): boolean => {
    if (!user?.permissions) return false;
    
    // Check for exact permission or wildcard permissions
    return user.permissions.some(userPerm => 
      userPerm === permission || 
      userPerm.endsWith('.*') && permission.startsWith(userPerm.slice(0, -1))
    );
  };

  const switchTenant = async (tenantId: number) => {
    try {
      await authApi.switchTenant(tenantId);
      await refreshAuth();
    } catch (error) {
      console.error('Tenant switch failed:', error);
      throw error;
    }
  };

  const contextValue: AuthContextType = {
    user,
    tenant,
    isAuthenticated: !!user,
    isLoading,
    permissions: user?.permissions || [],
    login,
    logout,
    refreshToken: refreshAuth,
    hasPermission,
    switchTenant
  };

  return (
    <AuthContext.Provider value={contextValue}>
      {children}
    </AuthContext.Provider>
  );
};
```

### Protected Route Component
```typescript
// ProtectedRoute.tsx - Route protection with permissions
interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredPermissions?: string[];
  requiredRoles?: string[];
  fallback?: React.ReactNode;
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  requiredPermissions = [],
  requiredRoles = [],
  fallback = <UnauthorizedPage />
}) => {
  const { isAuthenticated, isLoading, hasPermission, user } = useAuth();

  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  // Check required permissions
  if (requiredPermissions.length > 0) {
    const hasAllPermissions = requiredPermissions.every(hasPermission);
    if (!hasAllPermissions) {
      return fallback;
    }
  }

  // Check required roles
  if (requiredRoles.length > 0) {
    const hasRequiredRole = requiredRoles.some(role => 
      user?.roles?.includes(role)
    );
    if (!hasRequiredRole) {
      return fallback;
    }
  }

  return <>{children}</>;
};
```

## Security Implementation

### Session Security
```php
// SecureSessionService.php - Enhanced session security
class SecureSessionService
{
    /**
     * Create secure session with tenant context
     */
    public function createSecureSession(User $user, array $sessionData = []): string
    {
        $sessionId = Str::random(64);
        
        $sessionData = array_merge($sessionData, [
            'user_id' => $user->id,
            'tenant_id' => $user->tenant_id,
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'created_at' => now(),
            'last_activity' => now(),
            'security_hash' => $this->generateSecurityHash($user)
        ]);

        // Store session in Redis with encryption
        Redis::setex(
            "session:$sessionId",
            config('session.lifetime') * 60,
            encrypt(json_encode($sessionData))
        );

        // Log session creation
        SessionAuditLog::create([
            'session_id' => $sessionId,
            'user_id' => $user->id,
            'tenant_id' => $user->tenant_id,
            'action' => 'session_created',
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'created_at' => now()
        ]);

        return $sessionId;
    }

    /**
     * Validate session security
     */
    public function validateSession(string $sessionId): bool
    {
        $sessionData = $this->getSessionData($sessionId);
        
        if (!$sessionData) {
            return false;
        }

        // Validate IP address (if configured)
        if (config('session.validate_ip') && $sessionData['ip_address'] !== request()->ip()) {
            $this->invalidateSession($sessionId, 'ip_mismatch');
            return false;
        }

        // Validate user agent (if configured)
        if (config('session.validate_user_agent') && $sessionData['user_agent'] !== request()->userAgent()) {
            $this->invalidateSession($sessionId, 'user_agent_mismatch');
            return false;
        }

        // Validate security hash
        $user = User::find($sessionData['user_id']);
        if (!$user || $sessionData['security_hash'] !== $this->generateSecurityHash($user)) {
            $this->invalidateSession($sessionId, 'security_hash_mismatch');
            return false;
        }

        // Update last activity
        $this->updateSessionActivity($sessionId);

        return true;
    }
}
```

## Compliance Integration

### GDPR/CCPA Data Subject Rights
```php
// DataSubjectRightsService.php - Privacy compliance for authentication
class DataSubjectRightsService
{
    /**
     * Export user authentication data for GDPR compliance
     */
    public function exportAuthenticationData(User $user): array
    {
        return [
            'authentication_logs' => AuditTrail::where('user_id', $user->id)
                ->where('action', 'like', 'authentication_%')
                ->select(['action', 'ip_address', 'created_at'])
                ->get(),
            
            'session_history' => SessionAuditLog::where('user_id', $user->id)
                ->select(['action', 'ip_address', 'created_at'])
                ->get(),
                
            'mfa_settings' => [
                'enabled' => !empty($user->mfa_secret),
                'methods' => $user->mfa_methods ?? [],
                'last_setup' => $user->mfa_setup_at
            ],
            
            'permissions' => $user->getAllPermissions()->pluck('name'),
            'roles' => $user->getRoleNames()
        ];
    }

    /**
     * Delete user authentication data for right to erasure
     */
    public function deleteAuthenticationData(User $user): void
    {
        DB::transaction(function () use ($user) {
            // Anonymize audit trails (retain for compliance but remove PII)
            AuditTrail::where('user_id', $user->id)->update([
                'ip_address' => '0.0.0.0',
                'user_agent' => 'ANONYMIZED',
                'metadata' => encrypt(['anonymized' => true])
            ]);

            // Revoke all tokens
            $user->tokens()->delete();

            // Clear MFA data
            $user->update([
                'mfa_secret' => null,
                'mfa_recovery_codes' => null,
                'mfa_methods' => null
            ]);

            // Log deletion for compliance
            ComplianceAuditLog::create([
                'user_id' => $user->id,
                'action' => 'authentication_data_deleted',
                'reason' => 'data_subject_request',
                'deleted_at' => now()
            ]);
        });
    }
}
```

This enhanced IAM file provides comprehensive authentication and authorization for the insurance system with full integration to the microservice architecture, multi-tenant security, and compliance requirements.