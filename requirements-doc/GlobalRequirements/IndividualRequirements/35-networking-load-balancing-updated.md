# 35.0 Networking & Load Balancing - Updated

## AWS Network Architecture

### Virtual Private Cloud (VPC) Design
- **Multi-AZ Architecture**: Subnets across 3 availability zones
- **Subnet Strategy**:
  - Public subnets: ALB, NAT Gateways, Bastion hosts
  - Private subnets: EKS nodes, RDS, ElastiCache
  - Database subnets: Isolated RDS subnet group
- **CIDR Planning**:
  - VPC: 10.0.0.0/16 (65,536 IPs)
  - Public: 10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24
  - Private: 10.0.10.0/23, 10.0.12.0/23, 10.0.14.0/23
  - Database: 10.0.20.0/24, 10.0.21.0/24, 10.0.22.0/24

### Network Security
- **Security Groups**:
  - ALB: Allow 80/443 from Internet
  - EKS Nodes: Allow from ALB and within cluster
  - RDS: Allow 3306 from EKS nodes only
  - ElastiCache: Allow 6379 from EKS nodes only
- **Network ACLs**: Default allow with logging
- **VPC Flow Logs**: Enabled for security monitoring
- **AWS Network Firewall**: Optional for advanced filtering

## Load Balancing Architecture

### AWS Application Load Balancer (ALB)
- **Purpose**: Primary ingress for all HTTP/HTTPS traffic
- **Features**:
  - SSL/TLS termination with ACM certificates
  - Path-based and host-based routing
  - WebSocket and HTTP/2 support
  - WAF integration for security
- **Target Groups**:
  - EKS NodePort services
  - Health checks per service
  - Connection draining
- **Multi-AZ Deployment**: Automatic cross-zone load balancing

### Network Load Balancer (NLB) - Optional
- **Use Cases**: 
  - TCP/UDP traffic
  - Ultra-low latency requirements
  - Static IP requirements
- **Features**:
  - Layer 4 load balancing
  - Preserve source IP
  - Zonal isolation

### NGINX Ingress Controller
- **Deployment**: DaemonSet on EKS worker nodes
- **Purpose**: Kubernetes-native ingress management
- **Features**:
  - Namespace isolation
  - Path-based routing
  - SSL termination (optional)
  - Rate limiting
  - Custom error pages
- **Integration**: 
  - ALB target group registration
  - Automatic DNS updates via ExternalDNS

## CloudFront CDN Integration

### Global Content Delivery
- **Origin Configuration**:
  - ALB as primary origin
  - S3 buckets for static assets
  - Custom origin headers
- **Cache Behaviors**:
  - Static assets: Cache 1 year
  - API responses: Cache based on headers
  - Dynamic content: No cache
- **Security**:
  - AWS WAF integration
  - Geo-restriction capabilities
  - Field-level encryption

### Performance Optimization
- **Compression**: Automatic gzip/brotli
- **HTTP/2 and HTTP/3**: Enabled by default
- **Origin Shield**: Optional for origin protection
- **Real-time Logs**: CloudWatch real-time logs

## Multi-Region Networking

### AWS Transit Gateway
- **Purpose**: Connect multiple VPCs across regions
- **Architecture**:
  - Hub-and-spoke topology
  - Route tables per attachment
  - Cross-region peering
- **Use Cases**:
  - Multi-region DR setup
  - Shared services VPC
  - Hybrid cloud connectivity

### VPC Peering
- **Purpose**: Direct VPC-to-VPC communication
- **Configuration**:
  - Cross-region peering for DR
  - Route table updates
  - Security group rules
- **Limitations**: Non-transitive routing

## DNS and Traffic Management

### Route 53 Configuration
- **Hosted Zones**:
  - Public zone: example.com
  - Private zone: internal.example.com
- **Record Types**:
  - A records: ALB aliases
  - CNAME: Service endpoints
  - TXT: Domain verification
- **Routing Policies**:
  - Weighted: A/B testing
  - Latency: Optimal region selection
  - Failover: DR automation
  - Geolocation: Compliance requirements

### Health Checks and Failover
- **ALB Health Checks**: Layer 7 health validation
- **Route 53 Health Checks**: Endpoint monitoring
- **Automated Failover**: DNS-based DR switching
- **Notification**: SNS alerts on failures

## Service Mesh Networking

### Istio Traffic Management
- **Virtual Services**: Traffic routing rules
- **Destination Rules**: Load balancing policies
- **Gateways**: Ingress/egress configuration
- **Service Entries**: External service registry

### Network Policies
- **Kubernetes NetworkPolicy**: 
  - Namespace isolation
  - Pod-to-pod communication rules
  - Ingress/egress controls
- **Istio Authorization Policies**:
  - Service-level access control
  - JWT validation
  - mTLS enforcement

## Private Connectivity

### VPC Endpoints
- **Interface Endpoints**:
  - S3, DynamoDB, ECR, Secrets Manager
  - Private connectivity to AWS services
  - No internet gateway required
- **Gateway Endpoints**: S3 and DynamoDB

### AWS PrivateLink
- **Purpose**: Private connectivity to services
- **Use Cases**:
  - SaaS application integration
  - Cross-account service sharing
- **Security**: No internet exposure

### VPN Connectivity
- **Site-to-Site VPN**: 
  - Backup for Direct Connect
  - Remote office connectivity
  - IPSec encryption
- **Client VPN**: 
  - Developer access
  - Emergency access
  - MFA required

## Network Monitoring

### VPC Flow Logs
- **Configuration**: 
  - All traffic capture
  - S3 storage for analysis
  - CloudWatch Logs integration
- **Analysis**: 
  - AWS Athena queries
  - Security analysis
  - Traffic patterns

### Network Performance Monitoring
- **CloudWatch Metrics**:
  - ALB metrics: Request count, latency, errors
  - Network interface metrics
  - VPN connection metrics
- **AWS X-Ray**: Distributed request tracing
- **VPC Reachability Analyzer**: Connectivity testing

## Network Optimization

### Performance Tuning
- **Jumbo Frames**: 9000 MTU within VPC
- **Placement Groups**: Low-latency node placement
- **Enhanced Networking**: SR-IOV for high performance
- **Network Load Balancer**: Ultra-low latency needs

### Cost Optimization
- **NAT Gateway**: High availability vs. cost
- **VPC Endpoints**: Reduce data transfer costs
- **CloudFront**: Reduce origin load
- **Traffic Analysis**: Optimize routing

## Disaster Recovery Networking

### Multi-Region Setup
- **Secondary Region VPC**: Matching architecture
- **Cross-Region Peering**: Private connectivity
- **Route 53 Failover**: Automated DNS switching
- **Database Replication**: Private network paths

### Network Recovery Procedures
- **Failover Testing**: Regular DR drills
- **Documentation**: Network diagrams current
- **Runbooks**: Step-by-step procedures
- **Recovery Time**: Meet RTO requirements

## Compliance and Security

### Network Segmentation
- **Tenant Isolation**: Namespace-level separation
- **Compliance Zones**: PCI/HIPAA segregation
- **DMZ Pattern**: Public-facing services
- **Zero Trust**: Micro-segmentation

### Network Audit and Compliance
- **AWS Config**: Configuration compliance
- **Access Analyzer**: Public access detection
- **Security Hub**: Compliance scoring
- **Audit Logs**: Complete network audit trail