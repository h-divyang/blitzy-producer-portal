# 18.0 Workflow Requirements - Updated

## Comprehensive Insurance Workflow Management System

### Workflow Architecture Overview
The insurance management system implements sophisticated workflow engines that orchestrate complex business processes across policy management, claims processing, underwriting, and financial operations. The system supports both human-driven and automated workflows with comprehensive audit trails and regulatory compliance.

### Technology Integration
- **Workflow Engine**: Laravel 12.x-based state machine integration with PHP 8.4+ features
- **Event System**: Laravel events with Kafka integration for microservice coordination
- **Queue Management**: Laravel queues with Redis 7.x for background processing
- **State Management**: Database-driven state machines with full audit trails
- **API Integration**: RESTful APIs for external system workflow coordination

## Core Workflow Framework

### Universal Action Logging System
Every workflow step creates comprehensive audit records that satisfy insurance regulatory requirements and provide complete operational transparency.

```php
// app/Services/Workflow/WorkflowActionService.php
class WorkflowActionService
{
    use LogsActivity, ManagesTransactions;
    
    /**
     * Execute workflow action with comprehensive logging
     */
    public function executeAction(
        string $entityType,
        int $entityId,
        string $actionType,
        array $data = [],
        ?User $user = null,
        array $metadata = []
    ): WorkflowAction {
        return DB::transaction(function () use ($entityType, $entityId, $actionType, $data, $user, $metadata) {
            // 1. Create comprehensive action record
            $action = Action::create([
                'tenant_id' => TenantContext::getCurrentTenantId(),
                'action_type_id' => $this->resolveActionTypeId($actionType),
                'user_id' => $user?->id ?? $this->getSystemUserId(),
                'status_id' => $this->getStatusId('active'),
                'description' => $this->generateActionDescription($actionType, $data),
                'ip_address' => request()?->ip(),
                'user_agent' => request()?->userAgent(),
                'session_id' => session()?->getId(),
                'request_id' => request()?->header('X-Request-ID'),
                'metadata' => array_merge($metadata, [
                    'workflow_context' => $this->getWorkflowContext(),
                    'environment' => app()->environment(),
                    'timestamp' => now()->toISOString()
                ])
            ]);
            
            // 2. Create entity-action mapping
            $this->createEntityActionMapping($action->id, $entityType, $entityId);
            
            // 3. Execute the actual workflow step
            $result = $this->performWorkflowAction($entityType, $entityId, $actionType, $data);
            
            // 4. Log detailed changes if applicable
            if (isset($result['changes'])) {
                $this->logEntityChanges($action->id, $result['changes']);
            }
            
            // 5. Trigger downstream workflow events
            $this->triggerWorkflowEvents($entityType, $entityId, $actionType, $result);
            
            return new WorkflowAction($action, $result);
        });
    }
    
    /**
     * Create entity-specific action mapping with workflow context
     */
    private function createEntityActionMapping(int $actionId, string $entityType, int $entityId): void
    {
        $mappingTable = "map_{$entityType}_action";
        $entityField = "{$entityType}_id";
        
        DB::table($mappingTable)->insert([
            'tenant_id' => TenantContext::getCurrentTenantId(),
            $entityField => $entityId,
            'action_id' => $actionId,
            'workflow_step' => $this->getCurrentWorkflowStep($entityType, $entityId),
            'previous_state' => $this->getPreviousState($entityType, $entityId),
            'new_state' => $this->getNewState($entityType, $entityId),
            'created_at' => now(),
            'updated_at' => now()
        ]);
    }
    
    /**
     * Generate human-readable workflow action description
     */
    private function generateActionDescription(string $actionType, array $data): string
    {
        return match($actionType) {
            'policy_created' => "Policy created with number {$data['policy_number']} for {$data['policyholder_name']}",
            'policy_bound' => "Policy {$data['policy_number']} bound with premium {$data['premium_amount']}",
            'claim_reported' => "Claim {$data['claim_number']} reported for policy {$data['policy_number']}",
            'claim_assigned' => "Claim {$data['claim_number']} assigned to adjuster {$data['adjuster_name']}",
            'payment_processed' => "Payment of {$data['amount']} processed for {$data['reference']}",
            'underwriting_review' => "Underwriting review completed with decision: {$data['decision']}",
            default => ucfirst(str_replace('_', ' ', $actionType))
        };
    }
}
```

## Insurance-Specific Workflows

### 1. Policy Lifecycle Workflow

#### Quote-to-Bind Workflow
```php
// app/Workflows/PolicyLifecycleWorkflow.php
class PolicyLifecycleWorkflow extends BaseWorkflow
{
    protected array $states = [
        'draft' => 'Draft Quote',
        'quoted' => 'Quote Generated',
        'underwriting' => 'Underwriting Review',
        'approved' => 'Approved for Binding',
        'declined' => 'Declined',
        'bound' => 'Policy Bound',
        'active' => 'Policy Active',
        'cancelled' => 'Policy Cancelled',
        'expired' => 'Policy Expired'
    ];
    
    protected array $transitions = [
        'submit_application' => [
            'from' => ['draft'],
            'to' => 'quoted',
            'conditions' => ['application_complete', 'basic_eligibility_met']
        ],
        'send_to_underwriting' => [
            'from' => ['quoted'],
            'to' => 'underwriting',
            'conditions' => ['requires_underwriting_review']
        ],
        'approve_policy' => [
            'from' => ['quoted', 'underwriting'],
            'to' => 'approved',
            'conditions' => ['underwriting_approved', 'premium_calculated']
        ],
        'bind_policy' => [
            'from' => ['approved'],
            'to' => 'bound',
            'conditions' => ['payment_received', 'documents_signed']
        ],
        'activate_policy' => [
            'from' => ['bound'],
            'to' => 'active',
            'conditions' => ['effective_date_reached']
        ]
    ];
    
    /**
     * Execute quote submission workflow
     */
    public function submitApplication(Policy $policy, array $applicationData): WorkflowResult
    {
        return $this->executeTransition('submit_application', $policy, [
            'application_data' => $applicationData,
            'actions' => [
                function () use ($policy, $applicationData) {
                    // 1. Validate application completeness
                    $this->validateApplicationData($applicationData);
                    
                    // 2. Run basic eligibility checks
                    $eligibility = $this->checkBasicEligibility($policy, $applicationData);
                    
                    // 3. Calculate initial premium quote
                    $premium = $this->calculatePremium($policy, $applicationData);
                    
                    // 4. Update policy with quote information
                    $policy->update([
                        'status' => 'quoted',
                        'premium_amount' => $premium->total,
                        'coverage_details' => $premium->coverages,
                        'quote_expires_at' => now()->addDays(30)
                    ]);
                    
                    // 5. Generate quote documents
                    $this->generateQuoteDocuments($policy);
                    
                    // 6. Send notifications
                    $this->sendQuoteNotifications($policy);
                    
                    return [
                        'changes' => [
                            'status' => ['old' => 'draft', 'new' => 'quoted'],
                            'premium_amount' => ['new' => $premium->total],
                            'quote_expires_at' => ['new' => $policy->quote_expires_at]
                        ],
                        'metadata' => [
                            'eligibility_score' => $eligibility->score,
                            'premium_factors' => $premium->factors
                        ]
                    ];
                }
            ]
        ]);
    }
    
    /**
     * Execute underwriting review workflow
     */
    public function sendToUnderwriting(Policy $policy, array $reviewData): WorkflowResult
    {
        return $this->executeTransition('send_to_underwriting', $policy, [
            'review_data' => $reviewData,
            'actions' => [
                function () use ($policy, $reviewData) {
                    // 1. Create underwriting case
                    $underwritingCase = UnderwritingCase::create([
                        'tenant_id' => $policy->tenant_id,
                        'policy_id' => $policy->id,
                        'underwriter_id' => $this->assignUnderwriter($policy),
                        'priority' => $this->calculateUWPriority($policy),
                        'review_items' => $reviewData['review_items'],
                        'due_date' => now()->addBusinessDays(3)
                    ]);
                    
                    // 2. Update policy status
                    $policy->update([
                        'status' => 'underwriting',
                        'underwriting_case_id' => $underwritingCase->id
                    ]);
                    
                    // 3. Send underwriter notification
                    $this->notifyUnderwriter($underwritingCase);
                    
                    // 4. Schedule follow-up reminders
                    $this->scheduleUnderwritingReminders($underwritingCase);
                    
                    return [
                        'changes' => [
                            'status' => ['old' => 'quoted', 'new' => 'underwriting']
                        ],
                        'metadata' => [
                            'underwriting_case_id' => $underwritingCase->id,
                            'assigned_underwriter' => $underwritingCase->underwriter->name,
                            'due_date' => $underwritingCase->due_date
                        ]
                    ];
                }
            ]
        ]);
    }
}
```

### 2. Claims Processing Workflow

#### First Notice of Loss (FNOL) to Settlement
```php
// app/Workflows/ClaimsProcessingWorkflow.php
class ClaimsProcessingWorkflow extends BaseWorkflow
{
    protected array $states = [
        'reported' => 'Claim Reported',
        'assigned' => 'Assigned to Adjuster',
        'investigating' => 'Under Investigation',
        'documenting' => 'Documenting Damages',
        'evaluating' => 'Evaluating Settlement',
        'negotiating' => 'Settlement Negotiation',
        'approved' => 'Settlement Approved',
        'paying' => 'Processing Payment',
        'closed' => 'Claim Closed',
        'denied' => 'Claim Denied',
        'litigation' => 'In Litigation'
    ];
    
    /**
     * Process First Notice of Loss
     */
    public function reportClaim(Claim $claim, array $reportData): WorkflowResult
    {
        return $this->executeTransition('report_claim', $claim, [
            'report_data' => $reportData,
            'actions' => [
                function () use ($claim, $reportData) {
                    // 1. Validate claim eligibility
                    $this->validateClaimEligibility($claim);
                    
                    // 2. Assign claim number
                    $claim->update([
                        'claim_number' => $this->generateClaimNumber(),
                        'status' => 'reported',
                        'reported_at' => now()
                    ]);
                    
                    // 3. Initial fraud screening
                    $fraudScore = $this->performFraudScreening($claim, $reportData);
                    
                    // 4. Auto-assign adjuster based on claim type and value
                    $adjuster = $this->autoAssignAdjuster($claim);
                    
                    // 5. Create initial claim file
                    $this->createClaimFile($claim, $reportData);
                    
                    // 6. Send acknowledgment to claimant
                    $this->sendClaimAcknowledgment($claim);
                    
                    // 7. Notify adjuster of new assignment
                    $this->notifyAdjusterOfAssignment($claim, $adjuster);
                    
                    return [
                        'changes' => [
                            'claim_number' => ['new' => $claim->claim_number],
                            'status' => ['new' => 'reported'],
                            'adjuster_id' => ['new' => $adjuster->id]
                        ],
                        'metadata' => [
                            'fraud_score' => $fraudScore,
                            'auto_assigned' => true,
                            'adjuster_name' => $adjuster->name
                        ]
                    ];
                }
            ]
        ]);
    }
    
    /**
     * Complete claim investigation workflow
     */
    public function completeInvestigation(Claim $claim, array $investigationData): WorkflowResult
    {
        return $this->executeTransition('complete_investigation', $claim, [
            'investigation_data' => $investigationData,
            'actions' => [
                function () use ($claim, $investigationData) {
                    // 1. Validate investigation completeness
                    $this->validateInvestigationCompleteness($investigationData);
                    
                    // 2. Calculate settlement recommendation
                    $settlement = $this->calculateSettlementRecommendation($claim, $investigationData);
                    
                    // 3. Update claim with investigation results
                    $claim->update([
                        'status' => 'evaluating',
                        'investigation_completed_at' => now(),
                        'settlement_recommendation' => $settlement->amount,
                        'adjuster_notes' => $investigationData['final_notes']
                    ]);
                    
                    // 4. Create investigation report
                    $this->generateInvestigationReport($claim, $investigationData);
                    
                    // 5. Route for settlement approval if needed
                    if ($settlement->requiresApproval()) {
                        $this->routeForSettlementApproval($claim, $settlement);
                    }
                    
                    return [
                        'changes' => [
                            'status' => ['old' => 'investigating', 'new' => 'evaluating'],
                            'settlement_recommendation' => ['new' => $settlement->amount]
                        ],
                        'metadata' => [
                            'settlement_amount' => $settlement->amount,
                            'requires_approval' => $settlement->requiresApproval(),
                            'investigation_duration_days' => $claim->investigation_duration_days
                        ]
                    ];
                }
            ]
        ]);
    }
}
```

### 3. Financial Workflow Management

#### Payment Processing Workflow
```php
// app/Workflows/PaymentProcessingWorkflow.php
class PaymentProcessingWorkflow extends BaseWorkflow
{
    /**
     * Process premium payment workflow
     */
    public function processPremiumPayment(Payment $payment, array $paymentData): WorkflowResult
    {
        return $this->executeWorkflowAction('payment', $payment->id, 'process_premium_payment', [
            'payment_data' => $paymentData,
            'actions' => [
                function () use ($payment, $paymentData) {
                    // 1. Validate payment information
                    $this->validatePaymentData($paymentData);
                    
                    // 2. Process payment through gateway
                    $gatewayResult = $this->processPaymentGateway($payment, $paymentData);
                    
                    // 3. Update payment status based on gateway response
                    $payment->update([
                        'status' => $gatewayResult->isSuccessful() ? 'completed' : 'failed',
                        'gateway_response' => $gatewayResult->toArray(),
                        'processed_at' => now(),
                        'transaction_id' => $gatewayResult->transactionId
                    ]);
                    
                    // 4. If successful, apply payment to policy
                    if ($gatewayResult->isSuccessful()) {
                        $this->applyPaymentToPolicy($payment);
                        $this->updatePolicyStatus($payment->payable);
                        $this->sendPaymentConfirmation($payment);
                    } else {
                        $this->handlePaymentFailure($payment, $gatewayResult);
                    }
                    
                    return [
                        'changes' => [
                            'status' => ['old' => 'pending', 'new' => $payment->status],
                            'processed_at' => ['new' => $payment->processed_at]
                        ],
                        'metadata' => [
                            'gateway_response' => $gatewayResult->summary(),
                            'transaction_id' => $gatewayResult->transactionId
                        ]
                    ];
                }
            ]
        ]);
    }
    
    /**
     * Process claim settlement payment
     */
    public function processClaimPayment(ClaimPayment $payment, array $settlementData): WorkflowResult
    {
        return $this->executeWorkflowAction('claim_payment', $payment->id, 'process_claim_settlement', [
            'settlement_data' => $settlementData,
            'actions' => [
                function () use ($payment, $settlementData) {
                    // 1. Verify settlement authorization
                    $this->verifySettlementAuthorization($payment, $settlementData);
                    
                    // 2. Generate settlement documents
                    $documents = $this->generateSettlementDocuments($payment);
                    
                    // 3. Process payment to claimant
                    $paymentResult = $this->processClaimantPayment($payment, $settlementData);
                    
                    // 4. Update claim status
                    $payment->claim->update([
                        'status' => 'closed',
                        'settled_at' => now(),
                        'settlement_amount' => $payment->amount
                    ]);
                    
                    // 5. Close claim file
                    $this->closeClaimFile($payment->claim);
                    
                    // 6. Update loss reserves
                    $this->updateLossReserves($payment->claim);
                    
                    return [
                        'changes' => [
                            'claim_status' => ['old' => 'approved', 'new' => 'closed'],
                            'settlement_amount' => ['new' => $payment->amount]
                        ],
                        'metadata' => [
                            'settlement_documents' => $documents->pluck('id'),
                            'payment_method' => $settlementData['payment_method']
                        ]
                    ];
                }
            ]
        ]);
    }
}
```

## Automated Workflow Processing

### Scheduled Workflow Tasks
```php
// app/Console/Commands/ProcessScheduledWorkflows.php
class ProcessScheduledWorkflows extends Command
{
    protected $signature = 'workflows:process-scheduled';
    
    public function handle(): void
    {
        $this->processExpiredQuotes();
        $this->processRenewalNotifications();
        $this->processOverdueClaimTasks();
        $this->processPaymentReminders();
        $this->processComplianceDeadlines();
    }
    
    /**
     * Process expired quotes workflow
     */
    private function processExpiredQuotes(): void
    {
        $expiredQuotes = Policy::where('status', 'quoted')
            ->where('quote_expires_at', '<', now())
            ->get();
            
        foreach ($expiredQuotes as $policy) {
            $this->workflowService->executeAction(
                'policy',
                $policy->id,
                'quote_expired',
                ['expired_at' => now()],
                $this->getSystemUser(),
                ['automated' => true, 'trigger' => 'scheduled_task']
            );
            
            $policy->update(['status' => 'expired']);
            
            // Send expiration notification
            $this->sendQuoteExpirationNotification($policy);
        }
    }
    
    /**
     * Process renewal notifications workflow
     */
    private function processRenewalNotifications(): void
    {
        $renewingPolicies = Policy::where('status', 'active')
            ->whereBetween('expiration_date', [now()->addDays(30), now()->addDays(60)])
            ->whereDoesntHave('renewalNotifications', function ($query) {
                $query->where('sent_at', '>=', now()->subDays(30));
            })
            ->get();
            
        foreach ($renewingPolicies as $policy) {
            $this->workflowService->executeAction(
                'policy',
                $policy->id,
                'renewal_notification_sent',
                ['notification_type' => '30_day_notice'],
                $this->getSystemUser(),
                ['automated' => true, 'days_to_expiration' => 30]
            );
            
            // Generate renewal quote
            $this->generateRenewalQuote($policy);
            
            // Send renewal notification
            $this->sendRenewalNotification($policy);
        }
    }
}
```

### Event-Driven Workflow Coordination
```php
// app/Events/WorkflowEvents.php
class PolicyWorkflowEvent extends Event
{
    use SerializesModels;
    
    public function __construct(
        public Policy $policy,
        public string $transition,
        public array $metadata = []
    ) {}
}

// app/Listeners/WorkflowEventListener.php
class WorkflowEventListener
{
    /**
     * Handle policy workflow events
     */
    public function handlePolicyWorkflow(PolicyWorkflowEvent $event): void
    {
        match($event->transition) {
            'policy_bound' => $this->handlePolicyBound($event->policy),
            'policy_cancelled' => $this->handlePolicyCancelled($event->policy),
            'policy_renewed' => $this->handlePolicyRenewed($event->policy),
            'policy_reinstatement_eligible' => $this->handleReinstatementEligible($event->policy),
            'policy_reinstated' => $this->handlePolicyReinstated($event->policy),
            'policy_reinstatement_expired' => $this->handleReinstatementExpired($event->policy),
            default => null
        };
        
        // Publish event to Kafka for microservice coordination
        $this->publishToKafka('policy.workflow', [
            'tenant_id' => $event->policy->tenant_id,
            'policy_id' => $event->policy->id,
            'transition' => $event->transition,
            'timestamp' => now()->toISOString(),
            'metadata' => $event->metadata
        ]);
    }
    
    private function handlePolicyBound(Policy $policy): void
    {
        // 1. Create billing schedule
        $this->createBillingSchedule($policy);
        
        // 2. Generate policy documents
        $this->generatePolicyDocuments($policy);
        
        // 3. Update agent commissions
        $this->calculateAgentCommission($policy);
        
        // 4. Send welcome package
        $this->sendWelcomePackage($policy);
        
        // 5. Schedule first payment reminder
        $this->schedulePaymentReminder($policy);
    }
    
    private function handleReinstatementEligible(Policy $policy): void
    {
        // 1. Calculate reinstatement window expiration
        $this->scheduleReinstatementExpiration($policy);
        
        // 2. Send reinstatement notification
        $this->sendReinstatementNotification($policy);
        
        // 3. Create reinstatement calculation record
        $this->createReinstatementCalculation($policy);
    }
    
    private function handlePolicyReinstated(Policy $policy): void
    {
        // 1. Restore billing schedule with adjusted dates
        $this->restoreBillingSchedule($policy);
        
        // 2. Generate reinstatement documents
        $this->generateReinstatementDocuments($policy);
        
        // 3. Send reinstatement confirmation
        $this->sendReinstatementConfirmation($policy);
        
        // 4. Update agent commissions for reinstated policy
        $this->updateAgentCommissionForReinstatement($policy);
        
        // 5. Resume automated payment reminders
        $this->resumePaymentReminders($policy);
    }
    
    private function handleReinstatementExpired(Policy $policy): void
    {
        // 1. Mark policy as ineligible for reinstatement
        $this->markReinstatementExpired($policy);
        
        // 2. Send expiration notification
        $this->sendReinstatementExpirationNotification($policy);
        
        // 3. Archive reinstatement records
        $this->archiveReinstatementRecords($policy);
    }
}
```

## Advanced Locking and Concurrency Management

### Redis-Based Distributed Locking System
Building on Laravel's distributed locking capabilities with Redis 7.x for optimistic concurrency control aligned with our technology stack.

```php
// app/Services/Workflow/DistributedLockingService.php
class DistributedLockingService
{
    use LogsActivity;
    
    private Redis $redis;
    private int $defaultTtl = 1800; // 30 minutes
    
    /**
     * Acquire distributed lock with comprehensive audit trail
     */
    public function acquireLock(
        string $entityType,
        int $entityId,
        User $user,
        string $operation = 'edit',
        int $ttl = null
    ): LockResult {
        $lockKey = $this->generateLockKey($entityType, $entityId);
        $lockValue = $this->generateLockValue($user, $operation);
        $ttl = $ttl ?? $this->defaultTtl;
        
        // Use Redis SET with NX and EX for atomic lock acquisition
        $acquired = $this->redis->set($lockKey, $lockValue, 'EX', $ttl, 'NX');
        
        if ($acquired) {
            // Log successful lock acquisition
            $this->logAction('lock_acquired', $entityType, $entityId, $user, [
                'lock_key' => $lockKey,
                'operation' => $operation,
                'ttl_seconds' => $ttl,
                'expires_at' => now()->addSeconds($ttl)
            ]);
            
            // Store lock metadata in database for audit trail
            $this->createLockRecord($entityType, $entityId, $user, $operation, $ttl);
            
            return new LockResult(true, $lockKey, $lockValue);
        }
        
        // Lock acquisition failed - get current lock info
        $currentLock = $this->getLockInfo($lockKey);
        
        $this->logAction('lock_acquisition_failed', $entityType, $entityId, $user, [
            'lock_key' => $lockKey,
            'current_lock_holder' => $currentLock['user_id'] ?? 'unknown',
            'lock_expires_at' => $currentLock['expires_at'] ?? null
        ]);
        
        return new LockResult(false, $lockKey, null, $currentLock);
    }
    
    /**
     * Release distributed lock with workflow completion
     */
    public function releaseLock(
        string $lockKey,
        User $user,
        array $workflowResult = []
    ): bool {
        $lockValue = $this->redis->get($lockKey);
        
        if (!$lockValue) {
            return false; // Lock already expired or doesn't exist
        }
        
        $lockData = json_decode($lockValue, true);
        
        // Verify lock ownership
        if ($lockData['user_id'] !== $user->id) {
            throw new UnauthorizedLockReleaseException('User does not own this lock');
        }
        
        // Use Lua script for atomic lock release
        $script = "
            if redis.call('get', KEYS[1]) == ARGV[1] then
                return redis.call('del', KEYS[1])
            else
                return 0
            end
        ";
        
        $released = $this->redis->eval($script, 1, $lockKey, $lockValue);
        
        if ($released) {
            // Extract entity information from lock key
            [$entityType, $entityId] = $this->parseLockKey($lockKey);
            
            $this->logAction('lock_released', $entityType, $entityId, $user, [
                'lock_key' => $lockKey,
                'workflow_result' => $workflowResult,
                'released_at' => now()
            ]);
            
            // Update lock record in database
            $this->completeLockRecord($lockKey, $workflowResult);
        }
        
        return (bool) $released;
    }
    
    /**
     * Check lock status and validate ownership
     */
    public function checkLockStatus(string $entityType, int $entityId, ?User $user = null): LockStatus
    {
        $lockKey = $this->generateLockKey($entityType, $entityId);
        $lockValue = $this->redis->get($lockKey);
        
        if (!$lockValue) {
            return new LockStatus(false, null, null);
        }
        
        $lockData = json_decode($lockValue, true);
        $lockOwner = User::find($lockData['user_id']);
        
        $isOwnedByUser = $user && $lockData['user_id'] === $user->id;
        $ttl = $this->redis->ttl($lockKey);
        
        return new LockStatus(true, $lockOwner, $ttl, $isOwnedByUser, $lockData);
    }
    
    /**
     * Automatic lock cleanup for expired locks
     */
    public function cleanupExpiredLocks(): int
    {
        $cleaned = 0;
        
        // Get all lock records that should have expired
        $expiredLocks = LockRecord::where('expires_at', '<', now())
                                 ->where('status', 'active')
                                 ->get();
        
        foreach ($expiredLocks as $lockRecord) {
            $lockKey = $this->generateLockKey($lockRecord->entity_type, $lockRecord->entity_id);
            
            // Check if Redis lock actually exists (might have been cleaned up already)
            if (!$this->redis->exists($lockKey)) {
                // Mark as expired in database
                $lockRecord->update([
                    'status' => 'expired',
                    'expired_at' => now()
                ]);
                
                $this->logAction('lock_expired', $lockRecord->entity_type, $lockRecord->entity_id, null, [
                    'lock_key' => $lockKey,
                    'original_user_id' => $lockRecord->user_id,
                    'expired_at' => now()
                ]);
                
                $cleaned++;
            }
        }
        
        return $cleaned;
    }
    
    /**
     * Emergency lock override for administrators
     */
    public function overrideLock(
        string $entityType,
        int $entityId,
        User $admin,
        string $reason
    ): bool {
        if (!$admin->hasRole(['super_admin', 'tenant_admin'])) {
            throw new UnauthorizedLockOverrideException('Insufficient permissions for lock override');
        }
        
        $lockKey = $this->generateLockKey($entityType, $entityId);
        $currentLock = $this->getLockInfo($lockKey);
        
        // Force delete the lock
        $this->redis->del($lockKey);
        
        // Log the override action
        $this->logAction('lock_overridden', $entityType, $entityId, $admin, [
            'lock_key' => $lockKey,
            'original_lock_holder' => $currentLock['user_id'] ?? null,
            'override_reason' => $reason,
            'overridden_at' => now()
        ]);
        
        return true;
    }
    
    private function generateLockKey(string $entityType, int $entityId): string
    {
        $tenantId = TenantContext::getCurrentTenantId();
        return "lock:tenant:{$tenantId}:{$entityType}:{$entityId}";
    }
    
    private function generateLockValue(User $user, string $operation): string
    {
        return json_encode([
            'user_id' => $user->id,
            'user_name' => $user->name,
            'operation' => $operation,
            'acquired_at' => now()->toISOString(),
            'session_id' => session()->getId(),
            'ip_address' => request()->ip()
        ]);
    }
}
```

### Workflow Locking Integration
```php
// app/Services/Workflow/LockingWorkflowService.php
class LockingWorkflowService extends WorkflowActionService
{
    use ManagesLocks;
    
    /**
     * Execute workflow action with automatic locking
     */
    public function executeLockedAction(
        string $entityType,
        int $entityId,
        string $actionType,
        array $data = [],
        ?User $user = null,
        int $lockTtl = 1800
    ): WorkflowAction {
        $user = $user ?? Auth::user();
        
        // 1. Acquire distributed lock
        $lockResult = $this->distributedLocking->acquireLock(
            $entityType,
            $entityId,
            $user,
            $actionType,
            $lockTtl
        );
        
        if (!$lockResult->acquired) {
            throw new EntityLockedException(
                "Entity {$entityType}:{$entityId} is locked by another user",
                $lockResult->currentLockInfo
            );
        }
        
        try {
            // 2. Execute the workflow action
            $workflowAction = $this->executeAction(
                $entityType,
                $entityId,
                $actionType,
                $data,
                $user,
                ['lock_key' => $lockResult->lockKey]
            );
            
            // 3. Release lock on successful completion
            $this->distributedLocking->releaseLock(
                $lockResult->lockKey,
                $user,
                ['workflow_result' => 'success', 'action_id' => $workflowAction->action->id]
            );
            
            return $workflowAction;
            
        } catch (Exception $e) {
            // 4. Release lock on failure
            $this->distributedLocking->releaseLock(
                $lockResult->lockKey,
                $user,
                ['workflow_result' => 'error', 'error_message' => $e->getMessage()]
            );
            
            throw $e;
        }
    }
    
    /**
     * Check if entity can be edited by user
     */
    public function canUserEditEntity(string $entityType, int $entityId, User $user): bool
    {
        $lockStatus = $this->distributedLocking->checkLockStatus($entityType, $entityId, $user);
        
        // Entity is not locked - user can edit
        if (!$lockStatus->isLocked) {
            return true;
        }
        
        // Entity is locked by this user - can edit
        if ($lockStatus->isOwnedByUser) {
            return true;
        }
        
        // Entity is locked by another user - cannot edit
        return false;
    }
}
```

### Lock-Aware UI Components (React Integration)
```typescript
// src/hooks/useEntityLocking.ts
import { useEffect, useState } from 'react';
import { useMutation, useQuery } from '@tanstack/react-query';

interface LockStatus {
  isLocked: boolean;
  lockOwner?: User;
  expiresIn?: number;
  isOwnedByCurrentUser: boolean;
}

export const useEntityLocking = (entityType: string, entityId: number) => {
  const [lockStatus, setLockStatus] = useState<LockStatus | null>(null);
  
  // Query lock status
  const { data: lockInfo } = useQuery({
    queryKey: ['lock-status', entityType, entityId],
    queryFn: () => lockApi.checkLockStatus(entityType, entityId),
    refetchInterval: 10000, // Check every 10 seconds
  });
  
  // Acquire lock mutation
  const acquireLockMutation = useMutation({
    mutationFn: (operation: string) => 
      lockApi.acquireLock(entityType, entityId, operation),
    onSuccess: () => {
      // Refresh lock status
      queryClient.invalidateQueries(['lock-status', entityType, entityId]);
    }
  });
  
  // Release lock mutation
  const releaseLockMutation = useMutation({
    mutationFn: () => lockApi.releaseLock(entityType, entityId),
    onSuccess: () => {
      queryClient.invalidateQueries(['lock-status', entityType, entityId]);
    }
  });
  
  // Auto-release lock on component unmount
  useEffect(() => {
    return () => {
      if (lockStatus?.isOwnedByCurrentUser) {
        releaseLockMutation.mutate();
      }
    };
  }, [lockStatus?.isOwnedByCurrentUser]);
  
  return {
    lockStatus: lockInfo,
    acquireLock: acquireLockMutation.mutate,
    releaseLock: releaseLockMutation.mutate,
    isAcquiring: acquireLockMutation.isPending,
    isReleasing: releaseLockMutation.isPending,
    canEdit: lockInfo?.isOwnedByCurrentUser || !lockInfo?.isLocked
  };
};
```

## Workflow Monitoring and Reporting

### Workflow Performance Analytics
```php
// app/Services/WorkflowAnalyticsService.php
class WorkflowAnalyticsService
{
    /**
     * Generate workflow performance report
     */
    public function generatePerformanceReport(string $workflowType, array $filters = []): array
    {
        $actions = Action::with(['actionType', 'entityMappings'])
            ->where('metadata->workflow_type', $workflowType)
            ->when($filters['date_from'] ?? null, fn($q, $date) => $q->where('created_at', '>=', $date))
            ->when($filters['date_to'] ?? null, fn($q, $date) => $q->where('created_at', '<=', $date))
            ->get();
            
        return [
            'total_actions' => $actions->count(),
            'average_processing_time' => $this->calculateAverageProcessingTime($actions),
            'success_rate' => $this->calculateSuccessRate($actions),
            'bottlenecks' => $this->identifyBottlenecks($actions),
            'volume_by_day' => $this->calculateVolumeByDay($actions),
            'user_performance' => $this->calculateUserPerformance($actions)
        ];
    }
    
    /**
     * Identify workflow bottlenecks
     */
    private function identifyBottlenecks(Collection $actions): array
    {
        return $actions
            ->groupBy('action_type_id')
            ->map(function ($typeActions) {
                return [
                    'count' => $typeActions->count(),
                    'avg_duration' => $typeActions->avg('duration_minutes'),
                    'max_duration' => $typeActions->max('duration_minutes')
                ];
            })
            ->sortByDesc('avg_duration')
            ->take(5)
            ->toArray();
    }
}
```

## Cross-References

### Related Global Requirements
- **GR-64**: Policy Reinstatement with Lapse Process - Reinstatement workflow state transitions and event handling
- **GR-20**: Business Logic Standards - Service architecture for workflow processing
- **GR-37**: Action Tracking - Comprehensive audit trail requirements
- **GR-09**: State Management - Frontend workflow state management patterns

This comprehensive workflow requirements document provides the complete framework for managing complex insurance business processes with full audit trails, automated processing capabilities, and regulatory compliance features integrated into the Laravel 11.x+ and React 18+ application architecture.