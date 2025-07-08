# 44.0 Comprehensive Communication Architecture - Updated

## System-Wide Communication Strategy

### Overview
The insurance management system implements a comprehensive communication infrastructure that handles all email, SMS, and phone verification processes across the entire platform. This architecture integrates SendGrid for email delivery, Twilio for SMS and voice services, and HashiCorp Vault for secure credential management, providing a unified communication experience for all insurance workflows.

### Technology Stack Integration
- **Laravel Version**: 12.x+ with built-in notification system
- **Email Provider**: SendGrid API for reliable email delivery
- **SMS Provider**: Twilio API for SMS and voice communications
- **Secret Management**: HashiCorp Vault for secure credential storage
- **Queue System**: Laravel queues with Redis 7.x for asynchronous processing
- **Template Engine**: Laravel Blade with dynamic content replacement

## Multi-Channel Communication Framework

### Unified Communication Service Architecture
```php
// app/Services/Communication/CommunicationService.php
class CommunicationService
{
    use LogsActivity, ManagesTemplates, HandlesFailures;
    
    private SendGridService $sendGrid;
    private TwilioService $twilio;
    private VaultService $vault;
    
    /**
     * Send multi-channel communication
     */
    public function sendCommunication(
        CommunicationRequest $request
    ): CommunicationResult {
        return DB::transaction(function () use ($request) {
            // 1. Create communication record for audit trail
            $communication = Communication::create([
                'tenant_id' => $request->tenantId,
                'user_id' => $request->userId,
                'type' => $request->type,
                'channels' => $request->channels,
                'priority' => $request->priority,
                'scheduled_at' => $request->scheduledAt,
                'metadata' => $request->metadata
            ]);
            
            $results = [];
            
            // 2. Process each communication channel
            foreach ($request->channels as $channel) {
                try {
                    $result = match($channel) {
                        'email' => $this->sendEmail($request, $communication),
                        'sms' => $this->sendSMS($request, $communication),
                        'voice' => $this->makeVoiceCall($request, $communication),
                        'push' => $this->sendPushNotification($request, $communication),
                        default => throw new UnsupportedChannelException("Channel {$channel} not supported")
                    };
                    
                    $results[$channel] = $result;
                    
                } catch (Exception $e) {
                    $this->handleChannelFailure($communication, $channel, $e);
                    $results[$channel] = new ChannelResult(false, $e->getMessage());
                }
            }
            
            // 3. Update communication status
            $communication->update([
                'status' => $this->calculateOverallStatus($results),
                'sent_at' => now(),
                'results' => $results
            ]);
            
            // 4. Log communication action
            $this->logAction('communication_sent', 'communication', $communication->id, null, [
                'channels' => $request->channels,
                'recipient_count' => count($request->recipients),
                'success_channels' => collect($results)->where('success', true)->count()
            ]);
            
            return new CommunicationResult($communication, $results);
        });
    }
}
```

### SendGrid Email Integration
```php
// app/Services/Communication/SendGridService.php
class SendGridService
{
    use LogsActivity, HandlesRetries;
    
    private SendGrid $sendGrid;
    private VaultService $vault;
    
    public function __construct()
    {
        $apiKey = $this->vault->getSecret('sendgrid.api_key');
        $this->sendGrid = new SendGrid($apiKey);
    }
    
    /**
     * Send insurance-specific email
     */
    public function sendInsuranceEmail(
        EmailRequest $request,
        Communication $communication
    ): EmailResult {
        try {
            // 1. Load and compile email template
            $template = $this->loadEmailTemplate($request->templateName, $request->tenantId);
            $compiledContent = $this->compileTemplate($template, $request->data);
            
            // 2. Create SendGrid email object
            $email = new Mail();
            $email->setFrom($request->fromEmail, $request->fromName);
            $email->setSubject($compiledContent['subject']);
            
            // 3. Add recipients with personalization
            foreach ($request->recipients as $recipient) {
                $personalization = new Personalization();
                $personalization->addTo(new To($recipient['email'], $recipient['name']));
                
                // Add dynamic template data
                $templateData = array_merge($request->data, $recipient['data'] ?? []);
                $personalization->addDynamicTemplateData($templateData);
                
                $email->addPersonalization($personalization);
            }
            
            // 4. Set content and template
            if ($template->sendgrid_template_id) {
                $email->setTemplateId($template->sendgrid_template_id);
            } else {
                $email->addContent(new HtmlContent($compiledContent['html']));
                $email->addContent(new PlainTextContent($compiledContent['text']));
            }
            
            // 5. Add custom headers for tracking
            $email->addHeader('X-Communication-ID', $communication->id);
            $email->addHeader('X-Tenant-ID', $request->tenantId);
            $email->addHeader('X-Email-Type', $request->type);
            
            // 6. Configure tracking
            $email->setClickTracking(true);
            $email->setOpenTracking(true);
            $email->setSubscriptionTracking(false); // Insurance emails shouldn't have unsubscribe
            
            // 7. Send email
            $response = $this->sendGrid->send($email);
            
            // 8. Process response
            $success = $response->statusCode() >= 200 && $response->statusCode() < 300;
            
            if ($success) {
                $this->logEmailSuccess($communication, $request, $response);
            } else {
                $this->logEmailFailure($communication, $request, $response);
            }
            
            return new EmailResult($success, $response->statusCode(), $response->body());
            
        } catch (Exception $e) {
            $this->logEmailException($communication, $request, $e);
            throw $e;
        }
    }
    
    /**
     * Handle SendGrid webhooks for delivery tracking
     */
    public function handleWebhook(array $events): void
    {
        foreach ($events as $event) {
            $communicationId = $event['X-Communication-ID'] ?? null;
            
            if (!$communicationId) {
                continue;
            }
            
            $communication = Communication::find($communicationId);
            
            if (!$communication) {
                continue;
            }
            
            // Create email event record
            EmailEvent::create([
                'communication_id' => $communication->id,
                'tenant_id' => $communication->tenant_id,
                'event_type' => $event['event'],
                'email_address' => $event['email'],
                'timestamp' => Carbon::createFromTimestamp($event['timestamp']),
                'sendgrid_message_id' => $event['sg_message_id'] ?? null,
                'metadata' => $event
            ]);
            
            // Update communication status based on events
            $this->updateCommunicationFromEvent($communication, $event);
        }
    }
}
```

### Twilio SMS and Voice Integration
```php
// app/Services/Communication/TwilioService.php
class TwilioService
{
    use LogsActivity, HandlesRetries;
    
    private Client $twilio;
    private VaultService $vault;
    
    public function __construct()
    {
        $accountSid = $this->vault->getSecret('twilio.account_sid');
        $authToken = $this->vault->getSecret('twilio.auth_token');
        $this->twilio = new Client($accountSid, $authToken);
    }
    
    /**
     * Send insurance SMS notification
     */
    public function sendInsuranceSMS(
        SMSRequest $request,
        Communication $communication
    ): SMSResult {
        try {
            $results = [];
            
            foreach ($request->recipients as $recipient) {
                // 1. Load and compile SMS template
                $template = $this->loadSMSTemplate($request->templateName, $request->tenantId);
                $message = $this->compileTemplate($template, array_merge($request->data, $recipient['data'] ?? []));
                
                // 2. Send SMS via Twilio
                $twilioMessage = $this->twilio->messages->create(
                    $recipient['phone'],
                    [
                        'from' => $this->getTenantPhoneNumber($request->tenantId),
                        'body' => $message,
                        'statusCallback' => route('twilio.webhook.status'),
                        'provideFeedback' => true,
                        'maxPrice' => 0.10, // Prevent expensive international charges
                        'validityPeriod' => 3600 // 1 hour validity
                    ]
                );
                
                // 3. Create SMS record
                SMSMessage::create([
                    'communication_id' => $communication->id,
                    'tenant_id' => $request->tenantId,
                    'phone_number' => $recipient['phone'],
                    'twilio_sid' => $twilioMessage->sid,
                    'status' => $twilioMessage->status,
                    'price' => $twilioMessage->price,
                    'sent_at' => now()
                ]);
                
                $results[] = new SMSRecipientResult(true, $twilioMessage->sid, $recipient['phone']);
            }
            
            return new SMSResult(true, $results);
            
        } catch (Exception $e) {
            $this->logSMSException($communication, $request, $e);
            throw $e;
        }
    }
    
    /**
     * Make automated voice call (for critical notifications)
     */
    public function makeInsuranceCall(
        VoiceRequest $request,
        Communication $communication
    ): VoiceResult {
        try {
            $results = [];
            
            foreach ($request->recipients as $recipient) {
                // 1. Create TwiML for the call
                $twiml = $this->generateInsuranceTwiML($request, $recipient);
                
                // 2. Initiate call
                $call = $this->twilio->calls->create(
                    $recipient['phone'],
                    $this->getTenantPhoneNumber($request->tenantId),
                    [
                        'twiml' => $twiml,
                        'statusCallback' => route('twilio.webhook.call-status'),
                        'statusCallbackEvent' => ['initiated', 'ringing', 'answered', 'completed'],
                        'timeout' => 30,
                        'machineDetection' => 'Enable'
                    ]
                );
                
                // 3. Create call record
                VoiceCall::create([
                    'communication_id' => $communication->id,
                    'tenant_id' => $request->tenantId,
                    'phone_number' => $recipient['phone'],
                    'twilio_sid' => $call->sid,
                    'status' => $call->status,
                    'initiated_at' => now()
                ]);
                
                $results[] = new VoiceRecipientResult(true, $call->sid, $recipient['phone']);
            }
            
            return new VoiceResult(true, $results);
            
        } catch (Exception $e) {
            $this->logVoiceException($communication, $request, $e);
            throw $e;
        }
    }
    
    /**
     * Generate TwiML for insurance notifications
     */
    private function generateInsuranceTwiML(VoiceRequest $request, array $recipient): string
    {
        $template = $this->loadVoiceTemplate($request->templateName, $request->tenantId);
        $message = $this->compileTemplate($template, array_merge($request->data, $recipient['data'] ?? []));
        
        $twiml = new VoiceResponse();
        
        // Add pause for answering
        $twiml->pause(['length' => 2]);
        
        // Speak the message
        $twiml->say($message, [
            'voice' => 'alice',
            'language' => 'en-US'
        ]);
        
        // Repeat option
        $gather = $twiml->gather([
            'numDigits' => 1,
            'timeout' => 10,
            'action' => route('twilio.webhook.gather', ['communication' => $request->communicationId])
        ]);
        
        $gather->say('Press 1 to repeat this message, or hang up.');
        
        return $twiml->asXML();
    }
}
```

## Template Management System

### Dynamic Template Engine
```php
// app/Services/Communication/TemplateService.php
class TemplateService
{
    use LogsActivity, CachesTemplates;
    
    /**
     * Insurance-specific template categories
     */
    const TEMPLATE_CATEGORIES = [
        'policy_notifications' => [
            'policy_bound' => 'Policy Binding Confirmation',
            'policy_renewal' => 'Policy Renewal Notice',
            'policy_cancellation' => 'Policy Cancellation Notice',
            'premium_due' => 'Premium Payment Due',
            'coverage_change' => 'Coverage Modification Notice'
        ],
        'claims_communications' => [
            'claim_acknowledged' => 'Claim Acknowledgment',
            'claim_status_update' => 'Claim Status Update',
            'claim_settlement' => 'Claim Settlement Notice',
            'claim_denial' => 'Claim Denial Notification',
            'additional_info_needed' => 'Additional Information Request'
        ],
        'verification_security' => [
            'email_verification' => 'Email Address Verification',
            'phone_verification' => 'Phone Number Verification',
            'password_reset' => 'Password Reset Request',
            'security_alert' => 'Security Alert Notification',
            'login_verification' => 'Login Verification Code'
        ],
        'administrative' => [
            'welcome_message' => 'Welcome to Insurance Portal',
            'account_suspended' => 'Account Suspension Notice',
            'policy_document_ready' => 'Policy Documents Available',
            'appointment_reminder' => 'Appointment Reminder',
            'compliance_notification' => 'Compliance Requirement Notice'
        ]
    ];
    
    /**
     * Load template with tenant customization
     */
    public function loadTemplate(
        string $templateName,
        string $channel,
        int $tenantId
    ): CommunicationTemplate {
        // Try tenant-specific template first
        $template = CommunicationTemplate::where('name', $templateName)
            ->where('channel', $channel)
            ->where('tenant_id', $tenantId)
            ->where('is_active', true)
            ->first();
        
        // Fall back to system default template
        if (!$template) {
            $template = CommunicationTemplate::where('name', $templateName)
                ->where('channel', $channel)
                ->whereNull('tenant_id')
                ->where('is_active', true)
                ->first();
        }
        
        if (!$template) {
            throw new TemplateNotFoundException("Template {$templateName} not found for channel {$channel}");
        }
        
        return $template;
    }
    
    /**
     * Compile template with dynamic data
     */
    public function compileTemplate(
        CommunicationTemplate $template,
        array $data
    ): array {
        $compiled = [];
        
        // Compile subject (for email)
        if ($template->subject) {
            $compiled['subject'] = $this->replaceVariables($template->subject, $data);
        }
        
        // Compile content
        $compiled['content'] = $this->replaceVariables($template->content, $data);
        
        // Generate HTML version (for email)
        if ($template->channel === 'email') {
            $compiled['html'] = $this->generateHTMLContent($compiled['content'], $template->html_template);
            $compiled['text'] = strip_tags($compiled['content']);
        }
        
        return $compiled;
    }
    
    /**
     * Replace variables in template content
     */
    private function replaceVariables(string $content, array $data): string
    {
        // Standard Laravel-style variable replacement
        $content = preg_replace_callback('/\{\{([^}]+)\}\}/', function ($matches) use ($data) {
            $variable = trim($matches[1]);
            return data_get($data, $variable, $matches[0]);
        }, $content);
        
        // Insurance-specific helper functions
        $content = $this->applyInsuranceHelpers($content, $data);
        
        return $content;
    }
    
    /**
     * Apply insurance-specific template helpers
     */
    private function applyInsuranceHelpers(string $content, array $data): string
    {
        // Format currency
        $content = preg_replace_callback('/\@currency\(([^)]+)\)/', function ($matches) use ($data) {
            $value = data_get($data, trim($matches[1]), 0);
            return '$' . number_format($value, 2);
        }, $content);
        
        // Format dates
        $content = preg_replace_callback('/\@date\(([^,]+),?\s*([^)]*)\)/', function ($matches) use ($data) {
            $dateField = trim($matches[1]);
            $format = trim($matches[2]) ?: 'M j, Y';
            $date = data_get($data, $dateField);
            return $date ? Carbon::parse($date)->format($format) : '';
        }, $content);
        
        // Format policy numbers
        $content = preg_replace_callback('/\@policy\(([^)]+)\)/', function ($matches) use ($data) {
            $value = data_get($data, trim($matches[1]));
            return strtoupper($value);
        }, $content);
        
        return $content;
    }
}
```

## Verification and Security Communications

### Multi-Factor Authentication Integration
```php
// app/Services/Communication/VerificationService.php
class VerificationService extends CommunicationService
{
    /**
     * Send email verification
     */
    public function sendEmailVerification(User $user): VerificationResult
    {
        $token = $this->generateSecureToken($user, 'email_verification');
        $verificationUrl = $this->generateVerificationUrl($token);
        
        $request = new EmailRequest([
            'tenantId' => $user->tenant_id,
            'templateName' => 'email_verification',
            'recipients' => [[
                'email' => $user->email,
                'name' => $user->full_name,
                'data' => ['verification_url' => $verificationUrl]
            ]],
            'data' => [
                'user_name' => $user->first_name,
                'company_name' => $user->tenant->name,
                'verification_url' => $verificationUrl,
                'expires_at' => now()->addHours(24)->format('M j, Y g:i A')
            ],
            'type' => 'verification'
        ]);
        
        return $this->sendEmail($request);
    }
    
    /**
     * Send SMS verification code
     */
    public function sendSMSVerification(User $user): VerificationResult
    {
        $code = $this->generateVerificationCode();
        
        // Store verification code
        VerificationCode::create([
            'user_id' => $user->id,
            'tenant_id' => $user->tenant_id,
            'phone_number' => $user->phone,
            'code' => Hash::make($code),
            'type' => 'phone_verification',
            'expires_at' => now()->addMinutes(10)
        ]);
        
        $request = new SMSRequest([
            'tenantId' => $user->tenant_id,
            'templateName' => 'phone_verification',
            'recipients' => [[
                'phone' => $user->phone,
                'data' => ['verification_code' => $code]
            ]],
            'data' => [
                'verification_code' => $code,
                'company_name' => $user->tenant->name,
                'expires_minutes' => 10
            ],
            'type' => 'verification'
        ]);
        
        return $this->sendSMS($request);
    }
    
    /**
     * Generate secure verification token
     */
    private function generateSecureToken(User $user, string $type): string
    {
        $payload = [
            'user_id' => $user->id,
            'tenant_id' => $user->tenant_id,
            'type' => $type,
            'email' => $user->email,
            'issued_at' => now()->timestamp,
            'expires_at' => now()->addHours(24)->timestamp
        ];
        
        return base64_encode(encrypt(json_encode($payload)));
    }
}
```

## Insurance Workflow Communication Integration

### Policy Lifecycle Communications
```php
// app/Services/Communication/PolicyCommunicationService.php
class PolicyCommunicationService extends CommunicationService
{
    /**
     * Send policy binding confirmation
     */
    public function sendPolicyBoundNotification(Policy $policy): CommunicationResult
    {
        $request = new CommunicationRequest([
            'tenantId' => $policy->tenant_id,
            'userId' => $policy->policyholder_id,
            'type' => 'policy_bound',
            'channels' => ['email'],
            'templateName' => 'policy_bound',
            'recipients' => [
                [
                    'email' => $policy->policyholder->email,
                    'name' => $policy->policyholder->full_name,
                    'data' => $this->getPolicyData($policy)
                ]
            ],
            'data' => array_merge($this->getPolicyData($policy), [
                'agent_name' => $policy->agent->full_name ?? 'Customer Service',
                'agent_phone' => $policy->agent->phone ?? 'N/A'
            ]),
            'priority' => 'high'
        ]);
        
        return $this->sendCommunication($request);
    }
    
    /**
     * Send premium due reminder
     */
    public function sendPremiumDueReminder(Policy $policy, Payment $payment): CommunicationResult
    {
        $channels = ['email'];
        
        // Add SMS for high-value policies or overdue payments
        if ($policy->premium_amount > 5000 || $payment->due_date < now()->subDays(5)) {
            $channels[] = 'sms';
        }
        
        $request = new CommunicationRequest([
            'tenantId' => $policy->tenant_id,
            'userId' => $policy->policyholder_id,
            'type' => 'premium_due',
            'channels' => $channels,
            'templateName' => 'premium_due',
            'recipients' => [[
                'email' => $policy->policyholder->email,
                'phone' => $policy->policyholder->phone,
                'name' => $policy->policyholder->full_name,
                'data' => array_merge($this->getPolicyData($policy), $this->getPaymentData($payment))
            ]],
            'data' => [
                'payment_url' => route('payments.pay', ['payment' => $payment->id]),
                'days_overdue' => $payment->due_date < now() ? now()->diffInDays($payment->due_date) : 0
            ],
            'priority' => $payment->due_date < now() ? 'urgent' : 'normal'
        ]);
        
        return $this->sendCommunication($request);
    }
}
```

This comprehensive communication architecture provides a unified, scalable, and secure foundation for all email, SMS, and voice communications across the insurance management system, with proper integration into the Laravel 12.x+ workflow and complete audit capabilities.