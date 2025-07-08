# 34.0 Supporting Services - Updated

## Secrets Management

### AWS Secrets Manager
- **Purpose**: Primary secrets management for AWS-specific credentials
- **Features**:
  - Automatic secret rotation (RDS, API keys)
  - Cross-region secret replication
  - Fine-grained access control via IAM
  - CloudFormation/Terraform integration
- **Use Cases**:
  - RDS database credentials
  - Third-party API keys
  - OAuth client secrets
  - Encryption keys

### HashiCorp Vault Integration
- **Purpose**: Application-level secrets and encryption services
- **Deployment**: 
  - EKS StatefulSet with 3+ replicas
  - Auto-unseal using AWS KMS
  - Cross-AZ deployment for HA
- **Features**:
  - Dynamic secret generation
  - PKI certificate management
  - Encryption as a Service
  - Audit logging
- **Integration**: Kubernetes auth for pod identity

### AWS CloudHSM (Optional)
- **Purpose**: Hardware security module for high-security requirements
- **Use Cases**:
  - Payment card data encryption
  - Regulatory compliance (FIPS 140-2 Level 3)
  - Key ceremony requirements
- **Architecture**: CloudHSM cluster across AZs

## API Gateway - Kong

### Deployment Architecture
- **Infrastructure**: Deployed on EKS as Kubernetes Ingress
- **Database**: AWS RDS PostgreSQL for Kong configuration
- **Scaling**: Horizontal pod autoscaling based on traffic
- **High Availability**: Multi-AZ deployment with load balancing

### Kong Features Implementation
- **Plugins**:
  - Rate limiting per client/endpoint
  - JWT authentication
  - Request/response transformation
  - Logging to CloudWatch
  - Prometheus metrics
- **Admin API**: Secured with mTLS and IP restrictions
- **Developer Portal**: Self-service API documentation

## Service Mesh - Istio

### Istio Deployment
- **Control Plane**: Istiod deployed in dedicated namespace
- **Data Plane**: Envoy sidecars auto-injected
- **Integration**: 
  - AWS Load Balancer Controller
  - CloudWatch Container Insights
  - X-Ray for distributed tracing

### Service Mesh Features
- **Traffic Management**:
  - Canary deployments
  - Circuit breakers
  - Retry policies
  - Load balancing
- **Security**:
  - Automatic mTLS between services
  - Fine-grained authorization policies
  - External authorization integration
- **Observability**:
  - Distributed tracing with AWS X-Ray
  - Metrics export to CloudWatch
  - Service topology mapping

## Monitoring & Observability Stack

### Grafana LGTM Stack on EKS
- **Deployment Strategy**:
  - Grafana: 3+ replicas with shared storage
  - Loki: Distributed mode with S3 backend
  - Tempo: S3 storage for traces
  - Mimir: Long-term metrics storage in S3

### AWS CloudWatch Integration
- **Container Insights**: EKS cluster monitoring
- **Log Groups**: Centralized log aggregation
- **Metrics**: Custom application metrics
- **Dashboards**: Unified monitoring view
- **Alarms**: Multi-channel alerting

### Monitoring Configuration
```yaml
grafana:
  replicas: 3
  persistence:
    enabled: true
    storageClass: gp3
  datasources:
    - cloudwatch
    - loki
    - tempo
    - mimir

loki:
  storage:
    s3:
      bucketname: monitoring-loki-data
      region: us-east-1
  
tempo:
  storage:
    trace:
      backend: s3
      s3:
        bucket: monitoring-tempo-traces
```

## Backup Services

### Velero for Kubernetes
- **Deployment**: 
  - Velero server in kube-system namespace
  - S3 bucket for backup storage
  - IAM roles for service accounts (IRSA)
- **Backup Configuration**:
  - Daily full cluster backups
  - Hourly namespace snapshots
  - Volume snapshots via EBS CSI driver
- **Restore Capabilities**:
  - Full cluster restore
  - Selective namespace restore
  - Individual resource restore

### AWS Backup Integration
- **Managed Services**: RDS, EBS, EFS, DynamoDB
- **Backup Policies**: 
  - Centralized backup management
  - Cross-region backup copies
  - Lifecycle management
- **Compliance**: Backup job monitoring and reporting

## Container Registry

### AWS ECR (Elastic Container Registry)
- **Purpose**: Primary container image storage
- **Features**:
  - Image vulnerability scanning
  - Lifecycle policies for cleanup
  - Cross-region replication
  - IAM-based access control
- **Integration**: 
  - EKS pull-through cache
  - GitLab CI/CD push access
  - Image signing with AWS Signer

### GitLab Container Registry
- **Purpose**: Development and staging images
- **Sync Strategy**: Production images promoted to ECR
- **Cleanup**: Aggressive cleanup policies
- **Access**: Developer-friendly access

## Certificate Management

### AWS Certificate Manager (ACM)
- **Public Certificates**: Auto-renewed SSL/TLS certificates
- **Private CA**: Internal service certificates
- **Integration**:
  - ALB/NLB automatic certificate deployment
  - CloudFront distribution certificates
- **Monitoring**: Certificate expiration alerts

### cert-manager for Kubernetes
- **Purpose**: Kubernetes service certificates
- **Issuers**:
  - Let's Encrypt for public endpoints
  - AWS Private CA for internal services
- **Automation**: Automatic certificate renewal

## DNS and Service Discovery

### Route 53
- **Public DNS**: External service endpoints
- **Private DNS**: Internal service discovery
- **Health Checks**: Multi-region failover
- **Traffic Policies**: Geolocation and latency routing

### Kubernetes Service Discovery
- **CoreDNS**: Internal Kubernetes DNS
- **Headless Services**: Direct pod discovery
- **ExternalDNS**: Automatic Route 53 updates
- **Service Mesh**: Istio service registry

## Email and Communication Services

### Amazon SES Integration
- **Purpose**: Transactional email backup
- **Configuration**:
  - Domain verification
  - DKIM signing
  - Bounce/complaint handling
- **Use Case**: Regional email delivery

### SendGrid Primary Integration
- **Purpose**: Primary email delivery service
- **Features**: Templates, analytics, webhooks
- **Failover**: Automatic failover to SES

## Workflow and Orchestration

### AWS Step Functions
- **Purpose**: Complex workflow orchestration
- **Use Cases**:
  - Multi-step claim processing
  - Policy renewal workflows
  - Batch job orchestration
- **Integration**: Lambda, ECS tasks, API calls

### Kubernetes Jobs/CronJobs
- **Scheduled Tasks**: 
  - Report generation
  - Data synchronization
  - Cleanup operations
- **Management**: Kubernetes-native scheduling

## Development Support Services

### AWS Cloud9
- **Purpose**: Cloud-based IDE for quick edits
- **Features**: 
  - Pre-configured development environment
  - Direct AWS service access
  - Collaborative editing

### AWS CloudShell
- **Purpose**: CLI access from browser
- **Tools**: Pre-installed AWS CLI, kubectl, helm
- **Security**: Temporary environments

## Cost Management

### Service Cost Optimization
- **Reserved Capacity**: 
  - RDS reserved instances
  - ElastiCache reserved nodes
  - Compute savings plans
- **Spot Instances**: Non-critical workloads
- **Auto-scaling**: Right-sized resources
- **Monitoring**: Cost allocation tags

### Resource Cleanup
- **Unused Resources**: Automated identification
- **Lifecycle Policies**: Automatic cleanup rules
- **Cost Alerts**: Budget threshold notifications
- **Optimization Reports**: Weekly cost analysis