# 04.0 Validation & Data Handling - Updated

## Comprehensive Validation Strategy

### Validation Architecture Overview
- **Multi-Layer Validation**: Client-side, server-side, and database-level validation
- **Insurance Domain Rules**: Industry-specific validation for policies, claims, and underwriting
- **Security-First Approach**: Input sanitization and XSS protection at all levels
- **Real-Time Feedback**: Instant validation feedback with optimistic UI updates
- **Multi-Tenant Validation**: Tenant-specific business rules and data validation

### Technology Stack Integration
- **Laravel Version**: 12.x+ with enhanced validation features and custom rules
- **PHP Version**: 8.4+ for modern validation patterns and performance
- **Frontend**: React 18+ with TypeScript 5.x for type-safe validation
- **Validation Libraries**: Zod for TypeScript schema validation, Laravel Validator
- **Form Management**: React Hook Form with resolver integration

## Server-Side Validation (Laravel)

### Form Request Validation Framework
```php
// app/Http/Requests/BaseFormRequest.php
abstract class BaseFormRequest extends FormRequest
{
    use ValidatesWithTenant, LogsValidationAttempts;
    
    /**
     * Determine if the user is authorized to make this request
     */
    public function authorize(): bool
    {
        return $this->authorizeWithTenant();
    }
    
    /**
     * Get custom validation rules
     */
    public function rules(): array
    {
        $rules = $this->getBaseRules();
        $tenantRules = $this->getTenantSpecificRules();
        $contextRules = $this->getContextualRules();
        
        return array_merge($rules, $tenantRules, $contextRules);
    }
    
    /**
     * Get custom error messages
     */
    public function messages(): array
    {
        return array_merge(
            $this->getBaseMessages(),
            $this->getTenantMessages(),
            $this->getInsuranceMessages()
        );
    }
    
    /**
     * Get custom attributes for error messages
     */
    public function attributes(): array
    {
        return $this->getFieldAttributes();
    }
    
    /**
     * Handle a failed validation attempt
     */
    protected function failedValidation(Validator $validator): void
    {
        $this->logValidationFailure($validator);
        
        if ($this->expectsJson()) {
            throw new HttpResponseException(
                response()->json([
                    'message' => 'The given data was invalid.',
                    'errors' => $validator->errors()->toArray(),
                    'meta' => $this->getValidationMeta()
                ], 422)
            );
        }
        
        parent::failedValidation($validator);
    }
    
    /**
     * Prepare the data for validation
     */
    protected function prepareForValidation(): void
    {
        $this->sanitizeInput();
        $this->setTenantContext();
        $this->normalizeData();
    }
    
    abstract protected function getBaseRules(): array;
    abstract protected function authorizeWithTenant(): bool;
}
```

### Insurance Domain Form Requests

#### Policy Creation Validation
```php
// app/Http/Requests/Policy/StorePolicyRequest.php
class StorePolicyRequest extends BaseFormRequest
{
    protected function getBaseRules(): array
    {
        return [
            // Basic policy information
            'policy_number' => [
                'sometimes',
                'string',
                'max:20',
                Rule::unique('policies', 'policy_number')
                    ->where('tenant_id', $this->getCurrentTenantId())
                    ->whereNull('deleted_at')
            ],
            'type' => [
                'required',
                'string',
                Rule::in(PolicyType::cases())
            ],
            'status' => [
                'sometimes',
                'string',
                Rule::in(['quote', 'bound', 'active'])
            ],
            
            // Policyholder information
            'policyholder_id' => [
                'required',
                'integer',
                Rule::exists('users', 'id')
                    ->where('tenant_id', $this->getCurrentTenantId())
                    ->where('is_active', true)
            ],
            'agent_id' => [
                'nullable',
                'integer',
                Rule::exists('users', 'id')
                    ->where('tenant_id', $this->getCurrentTenantId())
                    ->whereHas('roles', function ($query) {
                        $query->where('name', 'agent');
                    })
            ],
            
            // Financial information
            'premium_amount' => [
                'required',
                'numeric',
                'min:0',
                'max:999999.99',
                new ValidPremiumAmount($this->type)
            ],
            'coverage_amount' => [
                'required',
                'numeric',
                'min:1000',
                'max:99999999.99',
                new ValidCoverageAmount($this->type)
            ],
            'deductible' => [
                'nullable',
                'numeric',
                'min:0',
                new ValidDeductible($this->coverage_amount)
            ],
            
            // Date validation
            'effective_date' => [
                'required',
                'date',
                'after_or_equal:today',
                new ValidEffectiveDate()
            ],
            'expiration_date' => [
                'required',
                'date',
                'after:effective_date',
                new ValidPolicyTerm($this->effective_date)
            ],
            
            // Coverage details
            'coverage_details' => [
                'required',
                'array',
                new ValidCoverageDetails($this->type)
            ],
            'coverage_details.*.type' => [
                'required',
                'string',
                Rule::in(CoverageType::cases())
            ],
            'coverage_details.*.limit' => [
                'required',
                'numeric',
                'min:0'
            ],
            'coverage_details.*.deductible' => [
                'nullable',
                'numeric',
                'min:0'
            ],
            
            // Risk factors
            'risk_factors' => [
                'sometimes',
                'array',
                new ValidRiskFactors($this->type)
            ],
            
            // Notes
            'notes' => [
                'nullable',
                'string',
                'max:2000',
                'sanitized'
            ]
        ];
    }
    
    protected function getTenantSpecificRules(): array
    {
        $tenant = $this->getCurrentTenant();
        $rules = [];
        
        // Tenant-specific policy limits
        if ($tenant->hasFeature('enhanced_coverage_limits')) {
            $rules['coverage_amount'][] = 'max:500000000.00';
        }
        
        // Mandatory agent assignment for certain tenants
        if ($tenant->getConfig('require_agent_assignment', false)) {
            $rules['agent_id'] = array_merge(
                $rules['agent_id'] ?? [],
                ['required']
            );
        }
        
        return $rules;
    }
    
    protected function getContextualRules(): array
    {
        $rules = [];
        
        // Additional validation based on policy type
        if ($this->type === 'auto') {
            $rules = array_merge($rules, $this->getAutoInsuranceRules());
        } elseif ($this->type === 'home') {
            $rules = array_merge($rules, $this->getHomeInsuranceRules());
        }
        
        return $rules;
    }
    
    private function getAutoInsuranceRules(): array
    {
        return [
            'coverage_details.liability' => [
                'required',
                new MinimumLiabilityCoverage()
            ],
            'risk_factors.vehicle_year' => [
                'required',
                'integer',
                'min:1900',
                'max:' . (date('Y') + 1)
            ],
            'risk_factors.vehicle_make' => [
                'required',
                'string',
                'max:50'
            ],
            'risk_factors.driver_age' => [
                'required',
                'integer',
                'min:16',
                'max:100'
            ]
        ];
    }
    
    protected function authorizeWithTenant(): bool
    {
        return $this->user()->can('policy.create') &&
               $this->validateTenantAccess();
    }
}
```

#### Claim Submission Validation
```php
// app/Http/Requests/Claims/StoreClaimRequest.php
class StoreClaimRequest extends BaseFormRequest
{
    protected function getBaseRules(): array
    {
        return [
            // Claim identification
            'claim_number' => [
                'sometimes',
                'string',
                'max:20',
                Rule::unique('claims', 'claim_number')
                    ->where('tenant_id', $this->getCurrentTenantId())
            ],
            
            // Policy reference
            'policy_id' => [
                'required',
                'integer',
                Rule::exists('policies', 'id')
                    ->where('tenant_id', $this->getCurrentTenantId())
                    ->where('status', 'active')
            ],
            
            // Claimant information
            'claimant_id' => [
                'required',
                'integer',
                Rule::exists('users', 'id')
                    ->where('tenant_id', $this->getCurrentTenantId())
            ],
            
            // Incident details
            'type' => [
                'required',
                'string',
                Rule::in(ClaimType::cases())
            ],
            'incident_date' => [
                'required',
                'date',
                'before_or_equal:today',
                'after:' . now()->subYears(2)->toDateString(),
                new ValidIncidentDate($this->policy_id)
            ],
            'description' => [
                'required',
                'string',
                'min:10',
                'max:5000',
                'sanitized'
            ],
            
            // Financial information
            'claimed_amount' => [
                'required',
                'numeric',
                'min:0.01',
                'max:999999999.99',
                new WithinPolicyLimits($this->policy_id, $this->type)
            ],
            
            // Incident details
            'incident_details' => [
                'required',
                'array',
                new ValidIncidentDetails($this->type)
            ],
            'incident_details.location' => [
                'required',
                'string',
                'max:200'
            ],
            'incident_details.weather_conditions' => [
                'sometimes',
                'string',
                'max:100'
            ],
            
            // Supporting documents
            'supporting_documents' => [
                'sometimes',
                'array',
                'max:10'
            ],
            'supporting_documents.*' => [
                'file',
                'mimes:pdf,doc,docx,jpg,jpeg,png',
                'max:10240', // 10MB
                new VirusFree()
            ],
            
            // Additional information
            'police_report_number' => [
                'sometimes',
                'string',
                'max:50',
                new ValidPoliceReportNumber()
            ],
            'witnesses' => [
                'sometimes',
                'array',
                'max:5'
            ],
            'witnesses.*.name' => [
                'required_with:witnesses',
                'string',
                'max:100'
            ],
            'witnesses.*.contact' => [
                'required_with:witnesses',
                'string',
                'max:100'
            ]
        ];
    }
    
    protected function authorizeWithTenant(): bool
    {
        return $this->user()->can('claim.create') &&
               $this->validatePolicyAccess();
    }
    
    private function validatePolicyAccess(): bool
    {
        if (!$this->policy_id) {
            return true; // Will fail validation
        }
        
        $policy = Policy::find($this->policy_id);
        
        return $policy &&
               $policy->tenant_id === $this->getCurrentTenantId() &&
               ($policy->policyholder_id === $this->user()->id ||
                $this->user()->can('claim.create_for_others'));
    }
}
```

### Custom Validation Rules

#### Insurance-Specific Validation Rules
```php
// app/Rules/ValidPremiumAmount.php
class ValidPremiumAmount implements Rule
{
    private string $policyType;
    
    public function __construct(string $policyType)
    {
        $this->policyType = $policyType;
    }
    
    public function passes($attribute, $value): bool
    {
        $minimums = [
            'auto' => 300,
            'home' => 500,
            'business' => 1000,
            'life' => 100,
            'health' => 200
        ];
        
        $minimum = $minimums[$this->policyType] ?? 100;
        
        return is_numeric($value) && $value >= $minimum;
    }
    
    public function message(): string
    {
        return "The premium amount is too low for {$this->policyType} insurance.";
    }
}

// app/Rules/ValidCoverageAmount.php
class ValidCoverageAmount implements Rule
{
    private string $policyType;
    
    public function __construct(string $policyType)
    {
        $this->policyType = $policyType;
    }
    
    public function passes($attribute, $value): bool
    {
        $limits = [
            'auto' => ['min' => 15000, 'max' => 1000000],
            'home' => ['min' => 50000, 'max' => 50000000],
            'business' => ['min' => 100000, 'max' => 100000000],
            'life' => ['min' => 10000, 'max' => 10000000],
            'health' => ['min' => 5000, 'max' => 5000000]
        ];
        
        $limit = $limits[$this->policyType] ?? ['min' => 1000, 'max' => 999999999];
        
        return is_numeric($value) &&
               $value >= $limit['min'] &&
               $value <= $limit['max'];
    }
    
    public function message(): string
    {
        return "The coverage amount is outside acceptable limits for {$this->policyType} insurance.";
    }
}

// app/Rules/WithinPolicyLimits.php
class WithinPolicyLimits implements Rule
{
    private int $policyId;
    private string $claimType;
    
    public function __construct(int $policyId, string $claimType)
    {
        $this->policyId = $policyId;
        $this->claimType = $claimType;
    }
    
    public function passes($attribute, $value): bool
    {
        $policy = Policy::find($this->policyId);
        
        if (!$policy) {
            return false;
        }
        
        // Get coverage limit for this claim type
        $coverageLimit = $this->getCoverageLimit($policy, $this->claimType);
        
        return $value <= $coverageLimit;
    }
    
    private function getCoverageLimit(Policy $policy, string $claimType): float
    {
        $coverageDetails = $policy->coverage_details;
        
        return match($claimType) {
            'collision' => $coverageDetails['collision']['limit'] ?? $policy->coverage_amount,
            'comprehensive' => $coverageDetails['comprehensive']['limit'] ?? $policy->coverage_amount,
            'liability' => $coverageDetails['liability']['limit'] ?? $policy->coverage_amount,
            default => $policy->coverage_amount
        };
    }
    
    public function message(): string
    {
        return 'The claimed amount exceeds the policy coverage limits.';
    }
}
```

## Client-Side Validation (React/TypeScript)

### Schema Validation with Zod
```typescript
// src/schemas/policySchemas.ts
import { z } from 'zod';

export const PolicyType = z.enum(['auto', 'home', 'business', 'life', 'health']);
export const PolicyStatus = z.enum(['quote', 'bound', 'active', 'cancelled', 'expired']);

export const PolicySchema = z.object({
  policy_number: z.string()
    .max(20, 'Policy number must be 20 characters or less')
    .optional(),
  
  type: PolicyType,
  
  status: PolicyStatus.default('quote'),
  
  policyholder_id: z.number()
    .int('Policyholder ID must be an integer')
    .positive('Policyholder ID must be positive'),
  
  agent_id: z.number()
    .int('Agent ID must be an integer')
    .positive('Agent ID must be positive')
    .optional()
    .nullable(),
  
  premium_amount: z.number()
    .positive('Premium amount must be positive')
    .max(999999.99, 'Premium amount is too high')
    .refine((val) => val >= 100, {
      message: 'Premium amount must be at least $100'
    }),
  
  coverage_amount: z.number()
    .positive('Coverage amount must be positive')
    .min(1000, 'Coverage amount must be at least $1,000')
    .max(99999999.99, 'Coverage amount is too high'),
  
  deductible: z.number()
    .nonnegative('Deductible cannot be negative')
    .optional()
    .nullable(),
  
  effective_date: z.string()
    .refine((date) => new Date(date) >= new Date(), {
      message: 'Effective date must be today or later'
    }),
  
  expiration_date: z.string(),
  
  coverage_details: z.record(z.object({
    type: z.string(),
    limit: z.number().positive(),
    deductible: z.number().nonnegative().optional()
  })),
  
  risk_factors: z.record(z.any()).optional(),
  
  notes: z.string()
    .max(2000, 'Notes must be 2000 characters or less')
    .optional()
    .nullable()
}).refine((data) => {
  const effectiveDate = new Date(data.effective_date);
  const expirationDate = new Date(data.expiration_date);
  return expirationDate > effectiveDate;
}, {
  message: 'Expiration date must be after effective date',
  path: ['expiration_date']
});

export type PolicyFormData = z.infer<typeof PolicySchema>;

// Type-specific schema extensions
export const AutoPolicySchema = PolicySchema.extend({
  type: z.literal('auto'),
  coverage_details: z.object({
    liability: z.object({
      limit: z.number().min(15000, 'Minimum liability coverage is $15,000')
    }),
    collision: z.object({
      limit: z.number().positive(),
      deductible: z.number().nonnegative()
    }).optional(),
    comprehensive: z.object({
      limit: z.number().positive(),
      deductible: z.number().nonnegative()
    }).optional()
  }),
  risk_factors: z.object({
    vehicle_year: z.number()
      .int()
      .min(1900, 'Vehicle year must be 1900 or later')
      .max(new Date().getFullYear() + 1, 'Vehicle year cannot be more than next year'),
    vehicle_make: z.string().max(50),
    vehicle_model: z.string().max(50),
    driver_age: z.number().int().min(16).max(100),
    driving_record: z.enum(['clean', 'minor_violations', 'major_violations'])
  })
});
```

### React Form Components with Validation
```typescript
// src/components/forms/PolicyForm.tsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { PolicySchema, PolicyFormData } from '@/schemas/policySchemas';
import { FormField, FormError, FormButton } from '@/components/common/Form';
import { usePolicyMutation } from '@/hooks/usePolicies';
import { useAuth } from '@/hooks/useAuth';

interface PolicyFormProps {
  initialData?: Partial<PolicyFormData>;
  onSuccess?: (policy: Policy) => void;
  onCancel?: () => void;
}

export const PolicyForm: React.FC<PolicyFormProps> = ({
  initialData,
  onSuccess,
  onCancel
}) => {
  const { user } = useAuth();
  const createPolicyMutation = usePolicyMutation();
  
  const {
    register,
    handleSubmit,
    watch,
    setValue,
    formState: { errors, isSubmitting, isValid }
  } = useForm<PolicyFormData>({
    resolver: zodResolver(PolicySchema),
    defaultValues: {
      status: 'quote',
      ...initialData
    },
    mode: 'onChange'
  });
  
  const policyType = watch('type');
  const effectiveDate = watch('effective_date');
  
  // Dynamic validation based on policy type
  React.useEffect(() => {
    if (policyType && effectiveDate) {
      validatePolicyTypeSpecificRules();
    }
  }, [policyType, effectiveDate]);
  
  const onSubmit = async (data: PolicyFormData) => {
    try {
      const policy = await createPolicyMutation.mutateAsync(data);
      onSuccess?.(policy);
    } catch (error) {
      console.error('Policy creation failed:', error);
    }
  };
  
  const validatePolicyTypeSpecificRules = () => {
    // Set minimum premium based on policy type
    const minimumPremiums = {
      auto: 300,
      home: 500,
      business: 1000,
      life: 100,
      health: 200
    };
    
    if (policyType && minimumPremiums[policyType]) {
      // Update validation rules dynamically
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        {/* Policy Type Selection */}
        <FormField label="Policy Type" required error={errors.type?.message}>
          <select
            {...register('type')}
            className="form-select"
            disabled={isSubmitting}
          >
            <option value="">Select Policy Type</option>
            <option value="auto">Auto Insurance</option>
            <option value="home">Home Insurance</option>
            <option value="business">Business Insurance</option>
            <option value="life">Life Insurance</option>
            <option value="health">Health Insurance</option>
          </select>
        </FormField>
        
        {/* Policyholder Selection */}
        <FormField label="Policyholder" required error={errors.policyholder_id?.message}>
          <PolicyholderSelect
            {...register('policyholder_id', { valueAsNumber: true })}
            disabled={isSubmitting}
          />
        </FormField>
        
        {/* Agent Selection */}
        {user?.hasPermission('policy.assign_agent') && (
          <FormField label="Agent" error={errors.agent_id?.message}>
            <AgentSelect
              {...register('agent_id', { valueAsNumber: true })}
              disabled={isSubmitting}
            />
          </FormField>
        )}
        
        {/* Financial Information */}
        <FormField label="Premium Amount" required error={errors.premium_amount?.message}>
          <CurrencyInput
            {...register('premium_amount', { valueAsNumber: true })}
            placeholder="0.00"
            disabled={isSubmitting}
          />
        </FormField>
        
        <FormField label="Coverage Amount" required error={errors.coverage_amount?.message}>
          <CurrencyInput
            {...register('coverage_amount', { valueAsNumber: true })}
            placeholder="0.00"
            disabled={isSubmitting}
          />
        </FormField>
        
        <FormField label="Deductible" error={errors.deductible?.message}>
          <CurrencyInput
            {...register('deductible', { valueAsNumber: true })}
            placeholder="0.00"
            disabled={isSubmitting}
          />
        </FormField>
        
        {/* Date Fields */}
        <FormField label="Effective Date" required error={errors.effective_date?.message}>
          <input
            type="date"
            {...register('effective_date')}
            min={new Date().toISOString().split('T')[0]}
            className="form-input"
            disabled={isSubmitting}
          />
        </FormField>
        
        <FormField label="Expiration Date" required error={errors.expiration_date?.message}>
          <input
            type="date"
            {...register('expiration_date')}
            min={effectiveDate}
            className="form-input"
            disabled={isSubmitting}
          />
        </FormField>
      </div>
      
      {/* Policy Type Specific Fields */}
      {policyType && (
        <PolicyTypeSpecificFields
          policyType={policyType}
          register={register}
          errors={errors}
          setValue={setValue}
          disabled={isSubmitting}
        />
      )}
      
      {/* Coverage Details */}
      <CoverageDetailsSection
        policyType={policyType}
        register={register}
        errors={errors}
        disabled={isSubmitting}
      />
      
      {/* Notes */}
      <FormField label="Notes" error={errors.notes?.message}>
        <textarea
          {...register('notes')}
          rows={4}
          className="form-textarea"
          placeholder="Additional notes or comments..."
          disabled={isSubmitting}
        />
      </FormField>
      
      {/* Form Actions */}
      <div className="flex justify-end space-x-4">
        {onCancel && (
          <FormButton
            type="button"
            variant="secondary"
            onClick={onCancel}
            disabled={isSubmitting}
          >
            Cancel
          </FormButton>
        )}
        <FormButton
          type="submit"
          variant="primary"
          disabled={!isValid || isSubmitting}
          loading={isSubmitting}
        >
          {initialData ? 'Update Policy' : 'Create Policy'}
        </FormButton>
      </div>
      
      {/* Global Form Errors */}
      <FormError error={createPolicyMutation.error?.message} />
    </form>
  );
};
```

### Real-Time Validation Hooks
```typescript
// src/hooks/useRealTimeValidation.ts
import { useCallback, useEffect, useState } from 'react';
import { debounce } from 'lodash';
import { apiClient } from '@/services/api';

interface ValidationResult {
  isValid: boolean;
  errors: Record<string, string[]>;
  warnings: Record<string, string[]>;
}

export const useRealTimeValidation = (
  endpoint: string,
  data: Record<string, any>,
  dependencies: string[] = []
) => {
  const [validationResult, setValidationResult] = useState<ValidationResult>({
    isValid: true,
    errors: {},
    warnings: {}
  });
  const [isValidating, setIsValidating] = useState(false);
  
  const validateData = useCallback(
    debounce(async (dataToValidate: Record<string, any>) => {
      if (!dataToValidate || Object.keys(dataToValidate).length === 0) {
        return;
      }
      
      setIsValidating(true);
      
      try {
        const response = await apiClient.post(`${endpoint}/validate`, dataToValidate);
        setValidationResult(response.data);
      } catch (error) {
        console.error('Real-time validation failed:', error);
        setValidationResult({
          isValid: false,
          errors: { general: ['Validation service unavailable'] },
          warnings: {}
        });
      } finally {
        setIsValidating(false);
      }
    }, 500),
    [endpoint]
  );
  
  useEffect(() => {
    const relevantData = dependencies.reduce((acc, key) => {
      if (data[key] !== undefined) {
        acc[key] = data[key];
      }
      return acc;
    }, {} as Record<string, any>);
    
    if (Object.keys(relevantData).length > 0) {
      validateData(relevantData);
    }
  }, dependencies.map(key => data[key]));
  
  return {
    ...validationResult,
    isValidating
  };
};
```

## Data Sanitization and Security

### Input Sanitization Middleware
```php
// app/Http/Middleware/SanitizeInput.php
class SanitizeInput
{
    public function handle(Request $request, Closure $next): Response
    {
        $input = $request->all();
        
        $sanitized = $this->sanitizeArray($input);
        
        $request->replace($sanitized);
        
        return $next($request);
    }
    
    private function sanitizeArray(array $data): array
    {
        $sanitized = [];
        
        foreach ($data as $key => $value) {
            if (is_array($value)) {
                $sanitized[$key] = $this->sanitizeArray($value);
            } else {
                $sanitized[$key] = $this->sanitizeValue($value, $key);
            }
        }
        
        return $sanitized;
    }
    
    private function sanitizeValue($value, string $key): mixed
    {
        if (!is_string($value)) {
            return $value;
        }
        
        // HTML escape for user-generated content
        if ($this->isUserContent($key)) {
            return htmlspecialchars($value, ENT_QUOTES, 'UTF-8');
        }
        
        // Strip tags for most fields
        $value = strip_tags($value);
        
        // Trim whitespace
        $value = trim($value);
        
        // Remove null bytes
        $value = str_replace("\0", '', $value);
        
        return $value;
    }
    
    private function isUserContent(string $key): bool
    {
        $userContentFields = [
            'notes',
            'description',
            'comments',
            'message',
            'content'
        ];
        
        return in_array($key, $userContentFields);
    }
}
```

This comprehensive validation and data handling file provides multi-layer validation strategies with insurance domain-specific rules, real-time validation, and security-first data handling for the Laravel 11.x+ and React 18+ application.