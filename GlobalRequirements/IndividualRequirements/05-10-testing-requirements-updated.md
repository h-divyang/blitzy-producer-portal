# 05-10.0 Comprehensive Testing Requirements - Updated

## Testing Strategy Overview

### Unified Testing Philosophy
- **Comprehensive Coverage**: Unit, integration, end-to-end, and security testing across full stack
- **Shift-Left Approach**: Early testing integration in development lifecycle  
- **Documentation-Driven Testing**: Tests serve as living documentation of system behavior
- **Risk-Based Testing**: Focus on critical insurance business functions and compliance requirements
- **Multi-Tenant Testing**: Ensure isolation and data integrity across all tenant boundaries
- **Continuous Testing**: Automated testing throughout CI/CD pipeline with quality gates

## Backend Testing Framework (Laravel)

### PHPUnit Integration & Configuration
```php
// phpunit.xml - Comprehensive test configuration
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
         processIsolation="false"
         stopOnFailure="false"
         cacheDirectory=".phpunit.cache"
         backupGlobals="false">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
        <testsuite name="Integration">
            <directory>tests/Integration</directory>
        </testsuite>
    </testsuites>
    
    <source>
        <include>
            <directory>app</directory>
        </include>
        <exclude>
            <file>app/Console/Kernel.php</file>
            <file>app/Http/Kernel.php</file>
        </exclude>
    </source>
    
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="DB_CONNECTION" value="sqlite"/>
        <env name="DB_DATABASE" value=":memory:"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="QUEUE_CONNECTION" value="sync"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="MAIL_MAILER" value="array"/>
    </php>
    
    <coverage includeUncoveredFiles="true"
              processUncoveredFiles="true"
              ignoreDeprecatedCodeUnits="true"
              disableCodeCoverageIgnore="true">
        <report>
            <html outputDirectory="coverage-html" lowUpperBound="50" highLowerBound="80"/>
            <clover outputFile="coverage.xml"/>
        </report>
    </coverage>
</phpunit>
```

### Laravel-Specific Testing Patterns
```php
// PolicyServiceTest.php - Comprehensive service testing
class PolicyServiceTest extends TestCase
{
    use RefreshDatabase, WithFaker;

    protected PolicyService $policyService;
    protected User $user;
    protected Tenant $tenant;

    protected function setUp(): void
    {
        parent::setUp();
        
        $this->tenant = Tenant::factory()->create();
        $this->user = User::factory()->for($this->tenant)->create();
        $this->policyService = app(PolicyService::class);
        
        // Set tenant context for testing
        TenantScope::setTenantId($this->tenant->id);
    }

    /**
     * @test
     * @group unit
     * @group policy-creation
     */
    public function it_creates_policy_with_valid_data(): void
    {
        // Arrange
        $this->actingAs($this->user);
        
        $policyData = PolicyData::fake([
            'tenant_id' => $this->tenant->id,
            'policy_number' => 'POL-' . $this->faker->unique()->numberBetween(100000, 999999),
            'insured_name' => $this->faker->name(),
            'effective_date' => now()->addDays(1),
            'premium' => $this->faker->randomFloat(2, 500, 5000)
        ]);

        // Act
        $policy = $this->policyService->createPolicy($policyData);

        // Assert
        $this->assertInstanceOf(Policy::class, $policy);
        $this->assertDatabaseHas('policies', [
            'tenant_id' => $this->tenant->id,
            'policy_number' => $policyData->policyNumber,
            'status' => PolicyStatus::QUOTED,
            'created_by' => $this->user->id
        ]);

        // Verify events were dispatched
        Event::assertDispatched(PolicyCreated::class, function ($event) use ($policy) {
            return $event->policy->id === $policy->id;
        });

        // Verify audit trail
        $this->assertDatabaseHas('policy_audit_trail', [
            'policy_id' => $policy->id,
            'action' => 'created',
            'user_id' => $this->user->id
        ]);
    }

    /**
     * @test
     * @group unit
     * @group tenant-isolation
     */
    public function it_prevents_cross_tenant_policy_access(): void
    {
        // Arrange
        $otherTenant = Tenant::factory()->create();
        $otherUser = User::factory()->for($otherTenant)->create();
        $policy = Policy::factory()->for($this->tenant)->create();

        $this->actingAs($otherUser);

        // Act & Assert
        $this->expectException(TenantIsolationException::class);
        $this->policyService->getPolicy($policy->id);
    }

    /**
     * @test
     * @group integration
     * @group premium-calculation
     */
    public function it_calculates_premium_with_external_rating_service(): void
    {
        // Arrange
        $this->actingAs($this->user);
        
        Http::fake([
            'rating-service.com/api/rate' => Http::response([
                'premium' => 1250.00,
                'factors' => ['age' => 35, 'zip' => '12345'],
                'breakdown' => [
                    'base_rate' => 1000.00,
                    'age_factor' => 1.15,
                    'territory_factor' => 1.09
                ]
            ], 200)
        ]);

        $riskData = RiskData::fake();

        // Act
        $premium = $this->policyService->calculatePremium($riskData);

        // Assert
        $this->assertEquals(1250.00, $premium->amount);
        $this->assertNotEmpty($premium->factors);
        
        Http::assertSent(function (Request $request) {
            return $request->url() === 'rating-service.com/api/rate' &&
                   $request->hasHeader('Authorization');
        });
    }
}
```

### Model & Relationship Testing
```php
// PolicyModelTest.php - Model testing with relationships
class PolicyModelTest extends TestCase
{
    use RefreshDatabase;

    /**
     * @test
     * @group unit
     * @group model-relationships
     */
    public function it_has_proper_relationships(): void
    {
        $tenant = Tenant::factory()->create();
        $user = User::factory()->for($tenant)->create();
        $policy = Policy::factory()->for($tenant)->for($user, 'creator')->create();
        
        $coverage = Coverage::factory()->for($policy)->create();
        $claim = Claim::factory()->for($policy)->create();

        // Test relationships
        $this->assertInstanceOf(Tenant::class, $policy->tenant);
        $this->assertInstanceOf(User::class, $policy->creator);
        $this->assertTrue($policy->coverages->contains($coverage));
        $this->assertTrue($policy->claims->contains($claim));
    }

    /**
     * @test
     * @group unit
     * @group model-validation
     */
    public function it_validates_policy_attributes(): void
    {
        $this->expectException(ValidationException::class);
        
        Policy::create([
            'policy_number' => 'INVALID', // Should be POL-XXXXXX format
            'insured_name' => '',          // Required field
            'effective_date' => 'invalid-date'
        ]);
    }

    /**
     * @test
     * @group unit
     * @group model-scopes
     */
    public function it_applies_tenant_scope_automatically(): void
    {
        $tenant1 = Tenant::factory()->create();
        $tenant2 = Tenant::factory()->create();
        
        $policy1 = Policy::factory()->for($tenant1)->create();
        $policy2 = Policy::factory()->for($tenant2)->create();

        // Set tenant context
        TenantScope::setTenantId($tenant1->id);

        // Query should only return policies for tenant1
        $policies = Policy::all();
        
        $this->assertCount(1, $policies);
        $this->assertEquals($policy1->id, $policies->first()->id);
    }
}
```

### API Testing with Authentication
```php
// PolicyApiTest.php - Comprehensive API testing
class PolicyApiTest extends TestCase
{
    use RefreshDatabase;

    protected User $user;
    protected Tenant $tenant;

    protected function setUp(): void
    {
        parent::setUp();
        
        $this->tenant = Tenant::factory()->create();
        $this->user = User::factory()->for($this->tenant)->create();
    }

    /**
     * @test
     * @group integration
     * @group api-authentication
     */
    public function it_requires_authentication_for_policy_endpoints(): void
    {
        $response = $this->getJson('/api/v1/policies');
        
        $response->assertUnauthorized()
                 ->assertJson([
                     'error' => [
                         'code' => 'UNAUTHENTICATED',
                         'message' => 'Authentication required'
                     ]
                 ]);
    }

    /**
     * @test
     * @group integration
     * @group api-creation
     */
    public function it_creates_policy_via_api(): void
    {
        $token = $this->createOAuthToken($this->user, ['create-policies']);
        
        $policyData = [
            'policy_number' => 'POL-123456',
            'insured_name' => 'John Doe',
            'effective_date' => '2024-01-01',
            'expiration_date' => '2024-12-31',
            'premium' => 1250.00,
            'policy_type' => 'auto'
        ];

        $response = $this->withToken($token)
                         ->postJson('/api/v1/policies', $policyData);

        $response->assertCreated()
                 ->assertJsonStructure([
                     'data' => [
                         'id',
                         'policy_number',
                         'insured_name',
                         'status',
                         'created_at'
                     ]
                 ]);

        $this->assertDatabaseHas('policies', [
            'tenant_id' => $this->tenant->id,
            'policy_number' => 'POL-123456',
            'insured_name' => 'John Doe'
        ]);
    }

    /**
     * @test
     * @group integration
     * @group api-validation
     */
    public function it_validates_policy_creation_data(): void
    {
        $token = $this->createOAuthToken($this->user, ['create-policies']);
        
        $invalidData = [
            'policy_number' => 'INVALID',  // Wrong format
            'insured_name' => '',          // Required field
            'effective_date' => 'invalid', // Invalid date
            'premium' => -100              // Negative value
        ];

        $response = $this->withToken($token)
                         ->postJson('/api/v1/policies', $invalidData);

        $response->assertStatus(422)
                 ->assertJsonValidationErrors([
                     'policy_number',
                     'insured_name', 
                     'effective_date',
                     'premium'
                 ]);
    }

    private function createOAuthToken(User $user, array $scopes = []): string
    {
        return $user->createToken('test-token', $scopes)->accessToken;
    }
}
```

## Frontend Testing Framework (React)

### React Testing Library Integration
```typescript
// setupTests.ts - Test environment configuration
import '@testing-library/jest-dom';
import { configure } from '@testing-library/react';
import { server } from './mocks/server';

// Establish API mocking before all tests
beforeAll(() => server.listen());

// Reset any request handlers that we may add during the tests
afterEach(() => server.resetHandlers());

// Clean up after the tests are finished
afterAll(() => server.close());

// Configure React Testing Library
configure({
  testIdAttribute: 'data-testid',
  asyncUtilTimeout: 5000,
});

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
};

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

### Component Testing Patterns
```typescript
// PolicyForm.test.tsx - Comprehensive component testing
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { rest } from 'msw';
import { server } from '../../mocks/server';
import { PolicyForm } from './PolicyForm';
import { renderWithProviders } from '../../test-utils';

describe('PolicyForm', () => {
  const mockOnSubmit = jest.fn();
  const mockOnCancel = jest.fn();

  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('Form Validation', () => {
    it('validates required fields', async () => {
      const user = userEvent.setup();
      
      render(
        <PolicyForm 
          mode="create" 
          onSubmit={mockOnSubmit} 
          onCancel={mockOnCancel} 
        />
      );

      const submitButton = screen.getByRole('button', { name: /submit/i });
      await user.click(submitButton);

      await waitFor(() => {
        expect(screen.getByText(/policy number is required/i)).toBeInTheDocument();
        expect(screen.getByText(/insured name is required/i)).toBeInTheDocument();
        expect(screen.getByText(/effective date is required/i)).toBeInTheDocument();
      });

      expect(mockOnSubmit).not.toHaveBeenCalled();
    });

    it('validates policy number format', async () => {
      const user = userEvent.setup();
      
      render(
        <PolicyForm 
          mode="create" 
          onSubmit={mockOnSubmit} 
          onCancel={mockOnCancel} 
        />
      );

      const policyNumberInput = screen.getByLabelText(/policy number/i);
      await user.type(policyNumberInput, 'INVALID-FORMAT');
      await user.tab(); // Trigger validation

      await waitFor(() => {
        expect(screen.getByText(/policy number must be in format POL-XXXXXX/i))
          .toBeInTheDocument();
      });
    });

    it('validates date ranges', async () => {
      const user = userEvent.setup();
      
      render(
        <PolicyForm 
          mode="create" 
          onSubmit={mockOnSubmit} 
          onCancel={mockOnCancel} 
        />
      );

      const effectiveDate = screen.getByLabelText(/effective date/i);
      const expirationDate = screen.getByLabelText(/expiration date/i);

      await user.type(effectiveDate, '2024-12-31');
      await user.type(expirationDate, '2024-01-01');
      await user.tab();

      await waitFor(() => {
        expect(screen.getByText(/expiration date must be after effective date/i))
          .toBeInTheDocument();
      });
    });
  });

  describe('Form Submission', () => {
    it('submits form with valid data', async () => {
      const user = userEvent.setup();
      
      const validPolicyData = {
        policyNumber: 'POL-123456',
        insuredName: 'John Doe',
        effectiveDate: '2024-01-01',
        expirationDate: '2024-12-31',
        premium: 1250.00
      };

      render(
        <PolicyForm 
          mode="create" 
          onSubmit={mockOnSubmit} 
          onCancel={mockOnCancel} 
        />
      );

      // Fill out the form
      await user.type(screen.getByLabelText(/policy number/i), validPolicyData.policyNumber);
      await user.type(screen.getByLabelText(/insured name/i), validPolicyData.insuredName);
      await user.type(screen.getByLabelText(/effective date/i), validPolicyData.effectiveDate);
      await user.type(screen.getByLabelText(/expiration date/i), validPolicyData.expirationDate);
      await user.type(screen.getByLabelText(/premium/i), validPolicyData.premium.toString());

      // Submit the form
      const submitButton = screen.getByRole('button', { name: /submit/i });
      await user.click(submitButton);

      await waitFor(() => {
        expect(mockOnSubmit).toHaveBeenCalledWith(
          expect.objectContaining(validPolicyData)
        );
      });
    });

    it('handles API errors gracefully', async () => {
      const user = userEvent.setup();
      
      // Mock API error response
      server.use(
        rest.post('/api/v1/policies', (req, res, ctx) => {
          return res(
            ctx.status(422),
            ctx.json({
              error: {
                code: 'VALIDATION_ERROR',
                message: 'The given data was invalid',
                validation_errors: {
                  policy_number: ['Policy number already exists']
                }
              }
            })
          );
        })
      );

      render(
        <PolicyForm 
          mode="create" 
          onSubmit={mockOnSubmit} 
          onCancel={mockOnCancel} 
        />
      );

      // Fill and submit form
      await user.type(screen.getByLabelText(/policy number/i), 'POL-123456');
      await user.type(screen.getByLabelText(/insured name/i), 'John Doe');
      await user.click(screen.getByRole('button', { name: /submit/i }));

      await waitFor(() => {
        expect(screen.getByText(/policy number already exists/i)).toBeInTheDocument();
      });
    });
  });

  describe('Accessibility', () => {
    it('has no accessibility violations', async () => {
      const { container } = render(
        <PolicyForm 
          mode="create" 
          onSubmit={mockOnSubmit} 
          onCancel={mockOnCancel} 
        />
      );

      const results = await axe(container);
      expect(results).toHaveNoViolations();
    });

    it('supports keyboard navigation', async () => {
      render(
        <PolicyForm 
          mode="create" 
          onSubmit={mockOnSubmit} 
          onCancel={mockOnCancel} 
        />
      );

      // Tab through form elements
      await userEvent.tab();
      expect(screen.getByLabelText(/policy number/i)).toHaveFocus();

      await userEvent.tab();
      expect(screen.getByLabelText(/insured name/i)).toHaveFocus();

      await userEvent.tab();
      expect(screen.getByLabelText(/effective date/i)).toHaveFocus();
    });
  });
});
```

### Custom Hook Testing
```typescript
// usePolicyForm.test.ts - Custom hook testing
import { renderHook, act } from '@testing-library/react';
import { usePolicyForm } from './usePolicyForm';

describe('usePolicyForm', () => {
  it('initializes with empty form data', () => {
    const { result } = renderHook(() => usePolicyForm({}, 'create'));

    expect(result.current.formData).toEqual({
      policyNumber: '',
      insuredName: '',
      effectiveDate: '',
      expirationDate: '',
      premium: 0,
      coverages: []
    });
    expect(result.current.errors).toEqual({});
    expect(result.current.isValid).toBe(false);
  });

  it('updates form data correctly', () => {
    const { result } = renderHook(() => usePolicyForm({}, 'create'));

    act(() => {
      result.current.handleChange('policyNumber', 'POL-123456');
    });

    expect(result.current.formData.policyNumber).toBe('POL-123456');
  });

  it('validates form data and sets errors', () => {
    const { result } = renderHook(() => usePolicyForm({}, 'create'));

    act(() => {
      result.current.handleChange('policyNumber', 'INVALID');
      result.current.validateForm();
    });

    expect(result.current.errors.policyNumber).toBe(
      'Policy number must be in format POL-XXXXXX'
    );
    expect(result.current.isValid).toBe(false);
  });
});
```

## End-to-End Testing with Playwright

### Playwright Configuration
```typescript
// playwright.config.ts - E2E testing configuration
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['junit', { outputFile: 'test-results/junit.xml' }],
    ['github']
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'mobile-chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'mobile-safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Critical User Journey Testing
```typescript
// policy-management.spec.ts - Complete policy workflow testing
import { test, expect } from '@playwright/test';

test.describe('Policy Management Workflow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('/login');
    await page.fill('[data-testid=email]', 'agent@example.com');
    await page.fill('[data-testid=password]', 'password');
    await page.click('[data-testid=login-button]');
    await expect(page).toHaveURL('/dashboard');
  });

  test('complete policy creation and binding flow', async ({ page }) => {
    // Navigate to new policy form
    await page.click('[data-testid=new-policy-button]');
    await expect(page).toHaveURL('/policies/new');
    await expect(page.locator('h1')).toContainText('Create New Policy');

    // Fill out policy form
    await page.fill('[data-testid=policy-number]', 'POL-123456');
    await page.fill('[data-testid=insured-name]', 'John Doe');
    await page.fill('[data-testid=insured-email]', 'john.doe@example.com');
    await page.fill('[data-testid=effective-date]', '2024-01-01');
    await page.fill('[data-testid=expiration-date]', '2024-12-31');
    
    // Add coverage
    await page.click('[data-testid=add-coverage-button]');
    await page.selectOption('[data-testid=coverage-type]', 'liability');
    await page.fill('[data-testid=coverage-limit]', '100000');
    
    // Calculate premium
    await page.click('[data-testid=calculate-premium]');
    await expect(page.locator('[data-testid=premium-amount]')).toBeVisible();
    
    // Save as quote
    await page.click('[data-testid=save-quote]');
    await expect(page.locator('[data-testid=success-message]'))
      .toContainText('Policy quote created successfully');

    // Navigate to policy details
    const policyId = await page.locator('[data-testid=policy-id]').textContent();
    await page.goto(`/policies/${policyId}`);
    
    // Bind the policy
    await page.click('[data-testid=bind-policy-button]');
    await page.click('[data-testid=confirm-bind]');
    
    // Verify policy is bound
    await expect(page.locator('[data-testid=policy-status]')).toContainText('Bound');
    await expect(page.locator('[data-testid=bind-date]')).toBeVisible();
    
    // Verify policy appears in policy list
    await page.goto('/policies');
    await expect(page.locator(`[data-testid=policy-row-${policyId}]`)).toBeVisible();
    await expect(page.locator(`[data-testid=policy-${policyId}-status]`))
      .toContainText('Bound');
  });

  test('policy search and filtering', async ({ page }) => {
    await page.goto('/policies');
    
    // Test search functionality
    await page.fill('[data-testid=search-input]', 'POL-123456');
    await page.press('[data-testid=search-input]', 'Enter');
    
    await expect(page.locator('[data-testid=policy-list]')).toBeVisible();
    await expect(page.locator('[data-testid=policy-row]')).toHaveCount(1);
    
    // Test status filtering
    await page.selectOption('[data-testid=status-filter]', 'bound');
    await expect(page.locator('[data-testid=policy-row]')).toContainText('Bound');
    
    // Test date range filtering
    await page.fill('[data-testid=date-from]', '2024-01-01');
    await page.fill('[data-testid=date-to]', '2024-12-31');
    await page.click('[data-testid=apply-filters]');
    
    // Verify results
    const policyRows = page.locator('[data-testid=policy-row]');
    expect(await policyRows.count()).toBeGreaterThan(0);
  });

  test('handles errors gracefully', async ({ page }) => {
    // Mock API error
    await page.route('/api/v1/policies', route => {
      route.fulfill({
        status: 500,
        contentType: 'application/json',
        body: JSON.stringify({
          error: {
            code: 'INTERNAL_SERVER_ERROR',
            message: 'Internal server error occurred'
          }
        })
      });
    });

    await page.goto('/policies/new');
    await page.fill('[data-testid=policy-number]', 'POL-123456');
    await page.fill('[data-testid=insured-name]', 'John Doe');
    await page.click('[data-testid=save-quote]');

    // Verify error handling
    await expect(page.locator('[data-testid=error-message]'))
      .toContainText('An error occurred while saving the policy');
    await expect(page.locator('[data-testid=retry-button]')).toBeVisible();
  });
});
```

## Documentation-Driven Testing

### Test Documentation Standards
```typescript
/**
 * @fileoverview Policy Management Test Suite
 * 
 * This test suite covers the complete policy management workflow including:
 * - Policy creation and validation
 * - Premium calculation integration  
 * - Policy binding and status management
 * - Multi-tenant data isolation
 * - Error handling and recovery
 * 
 * @author Development Team
 * @since 2024-01-01
 * @requires Laravel 12.x, React 18+, Playwright 1.45+
 */

/**
 * Test case: Policy creation with valid data
 * 
 * Business Rule: Policies must be created with complete information
 * and proper tenant isolation.
 * 
 * Acceptance Criteria:
 * - Policy number must follow POL-XXXXXX format
 * - Effective date must be in the future
 * - Premium must be calculated before binding
 * - Audit trail must be created
 * 
 * @test
 * @group policy-creation
 * @priority high
 * @risk-level medium
 */
```

### Living Documentation Integration
```markdown
# Policy Management Test Documentation

## Test Coverage Matrix

| Component | Unit Tests | Integration Tests | E2E Tests | Coverage % |
|-----------|------------|-------------------|-----------|------------|
| Policy Service | ✅ | ✅ | ✅ | 95% |
| Premium Calculator | ✅ | ✅ | ❌ | 87% |
| Policy API | ✅ | ✅ | ✅ | 92% |
| Policy Form UI | ✅ | ❌ | ✅ | 89% |

## Critical Test Scenarios

### Policy Creation Flow
1. **Valid Policy Creation** - Verify complete policy creation workflow
2. **Data Validation** - Ensure all business rules are enforced
3. **Tenant Isolation** - Confirm cross-tenant data access prevention
4. **Error Handling** - Test graceful error recovery

### Premium Calculation
1. **Rating Service Integration** - Test external API communication
2. **Calculation Accuracy** - Verify mathematical calculations
3. **Factor Application** - Ensure all rating factors are applied
4. **Caching Behavior** - Test premium calculation caching

## Test Data Management

### Factory Patterns
- Use Laravel factories for consistent test data generation
- Implement realistic data patterns for insurance industry
- Maintain referential integrity across related models
- Support multi-tenant test scenarios

### Test Environment
- In-memory SQLite for fast test execution
- Isolated test database per test run
- Mock external services for reliability
- Parallel test execution support
```

## Continuous Integration Testing

### GitLab CI/CD Integration
```yaml
# .gitlab-ci.yml - Comprehensive testing pipeline
stages:
  - test
  - security
  - performance
  - deploy

variables:
  MYSQL_ROOT_PASSWORD: secret
  MYSQL_DATABASE: insurance_test
  MYSQL_USER: insurance
  MYSQL_PASSWORD: secret

# Backend Testing
test:backend:
  stage: test
  image: php:8.4-cli
  services:
    - mysql:8.0
  cache:
    paths:
      - vendor/
  before_script:
    - apt-get update -qq && apt-get install -y -qq git curl libmcrypt-dev libjpeg-dev libpng-dev libfreetype6-dev libbz2-dev
    - curl -sS https://getcomposer.org/installer | php
    - php composer.phar install --no-interaction --prefer-dist --optimize-autoloader
    - cp .env.testing .env
    - php artisan key:generate
    - php artisan migrate --env=testing
  script:
    - php artisan test --coverage --min=80
    - php artisan test --group=unit
    - php artisan test --group=feature
    - php artisan test --group=integration
  artifacts:
    reports:
      junit: storage/logs/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
    paths:
      - coverage/
    expire_in: 30 days
  coverage: '/^\s*Lines:\s*\d+.\d+\%/'

# Frontend Testing  
test:frontend:
  stage: test
  image: node:24-alpine
  cache:
    paths:
      - node_modules/
  before_script:
    - npm ci
  script:
    - npm run test:unit -- --coverage --watchAll=false
    - npm run test:integration
    - npm run lint
    - npm run type-check
  artifacts:
    reports:
      junit: coverage/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 30 days

# End-to-End Testing
test:e2e:
  stage: test
  image: mcr.microsoft.com/playwright:v1.45.0-focal
  services:
    - name: mysql:8.0
      alias: database
  variables:
    DATABASE_URL: mysql://insurance:secret@database:3306/insurance_test
  before_script:
    - npm ci
    - npx playwright install
    - npm run build
  script:
    - npm run test:e2e:ci
  artifacts:
    when: always
    paths:
      - test-results/
      - playwright-report/
    expire_in: 30 days
  allow_failure: false

# Security Testing
test:security:
  stage: security
  image: php:8.4-cli
  script:
    - composer audit
    - npm audit
    - php artisan security:scan
  allow_failure: true

# Performance Testing
test:performance:
  stage: performance
  image: node:24-alpine
  script:
    - npm run test:performance
    - npm run lighthouse:ci
  artifacts:
    reports:
      performance: lighthouse-report.json
  allow_failure: true
```

## Quality Gates & Metrics

### Coverage Requirements
- **Backend (Laravel)**: Minimum 80% code coverage
- **Frontend (React)**: Minimum 70% code coverage  
- **Critical Paths**: Minimum 95% coverage for policy and claims workflows
- **Security Functions**: 100% coverage for authentication and authorization

### Performance Benchmarks
- **Unit Tests**: Maximum 30 seconds total execution time
- **Integration Tests**: Maximum 2 minutes total execution time
- **E2E Tests**: Maximum 10 minutes total execution time
- **API Response**: 95th percentile under 500ms

### Test Maintenance
- **Flaky Test Detection**: Automatic identification and flagging
- **Test Data Refresh**: Weekly test data cleanup and regeneration
- **Framework Updates**: Monthly review of testing framework updates
- **Documentation Sync**: Automatic documentation updates from test annotations

This comprehensive testing framework ensures robust, reliable, and maintainable insurance software with complete coverage across all system components while maintaining fast feedback cycles for development teams.