# 00.0 Technology Version Standards

## Version Standards Overview

### Technology Version Philosophy
- **Latest Stable Versions**: Use the most recent stable versions for security and performance
- **LTS Preference**: Prefer Long Term Support versions for production stability
- **Security Updates**: Maintain current versions for security patch availability
- **Future Compatibility**: Choose versions that support planned feature development
- **Enterprise Support**: Select versions with commercial support options

## Backend Technology Versions

### PHP & Laravel Framework
- **PHP Version**: 8.4+ (Latest Stable)
  - Minimum: PHP 8.4.0
  - Recommended: Latest PHP 8.4.x patch release
  - Rationale: Performance improvements, enhanced type system, security updates
  - EOL Consideration: PHP 8.3 supported until November 2026

- **Laravel Framework**: 12.x LTS (When Released)
  - Current: Laravel 11.x (until Laravel 12 LTS release)
  - Minimum: Laravel 11.30+
  - Rationale: Latest features, security updates, long-term support
  - Upgrade Path: Migrate to Laravel 12 LTS when available (estimated Q4 2024/Q1 2025)

### Database Technologies
- **MariaDB**: 12.x LTS
  - Minimum: MariaDB 12.0
  - Recommended: Latest MariaDB 12.x stable
  - AWS RDS Support: Verify MariaDB 12.x availability in AWS RDS
  - Alternative: MariaDB 11.x LTS if 12.x not available in AWS RDS

- **Redis**: 7.x
  - Minimum: Redis 7.0
  - Recommended: Redis 7.2+ (Latest stable)
  - AWS ElastiCache: Use latest Redis 7.x available in ElastiCache
  - Features: Enhanced security, improved performance, JSON data type support

### API & Middleware
- **Laravel Passport**: Latest compatible with Laravel 12.x/11.x
  - OAuth2 Server implementation
  - JWT token support
  - Scope-based authorization

- **Laravel Sanctum**: Latest compatible version
  - SPA authentication
  - Mobile app token authentication
  - Simple API token management

## Frontend Technology Versions

### JavaScript Runtime & Framework
- **Node.js**: 24.x LTS
  - Minimum: Node.js 24.0
  - Recommended: Latest Node.js 24.x LTS
  - Rationale: Latest performance improvements, security updates, npm compatibility
  - Alternative: Node.js 22.x LTS for stability if 24.x has issues

- **React**: 18.x (Latest)
  - Minimum: React 18.2
  - Recommended: Latest React 18.x stable
  - Features: Concurrent features, automatic batching, Suspense improvements
  - Future: Monitor React 19 for stable release

- **TypeScript**: 5.x
  - Minimum: TypeScript 5.0
  - Recommended: Latest TypeScript 5.x stable
  - Features: Decorators, const type parameters, improved inference

### Build Tools & Package Management
- **Vite**: 5.x
  - Minimum: Vite 5.0
  - Recommended: Latest Vite 5.x stable
  - Features: Improved HMR, better plugin ecosystem, enhanced build performance

- **npm**: 10.x
  - Included with Node.js 24.x
  - Alternative: pnpm 9.x for improved performance and disk usage

### UI & Styling
- **Tailwind CSS**: 3.x
  - Minimum: Tailwind CSS 3.4
  - Recommended: Latest Tailwind CSS 3.x stable
  - Features: Container queries, dynamic viewport units, modern color spaces

- **Minimal UI Kit**: Latest compatible version
  - Framework: React-based component library
  - Compatibility: Ensure React 18+ compatibility
  - Customization: Tailwind CSS integration support

## Infrastructure Technology Versions

### Container Orchestration
- **Kubernetes**: 1.30+
  - Minimum: Kubernetes 1.30
  - Recommended: Latest stable 1.30.x or 1.31.x
  - AWS EKS: Use latest EKS-supported Kubernetes version
  - Features: Enhanced security, improved performance, new APIs

- **Docker**: 24.x
  - Minimum: Docker 24.0
  - Recommended: Latest Docker 24.x stable
  - Features: Improved security, BuildKit enhancements, multi-platform builds

### Service Mesh & API Gateway
- **Istio**: 1.22+
  - Minimum: Istio 1.22
  - Recommended: Latest Istio 1.22.x stable
  - Features: Enhanced security policies, improved observability, performance optimizations

- **Kong**: 3.7+
  - Minimum: Kong 3.7
  - Recommended: Latest Kong 3.x stable
  - Features: Enhanced plugins, improved performance, better observability

### Observability Stack (LGTM)
- **Grafana**: 11.x
  - Minimum: Grafana 11.0
  - Recommended: Latest Grafana 11.x stable
  - Features: Enhanced dashboards, improved alerting, new visualization options

- **Loki**: 3.x
  - Minimum: Loki 3.0
  - Recommended: Latest Loki 3.x stable
  - Features: Improved query performance, enhanced retention policies

- **Tempo**: 2.x
  - Minimum: Tempo 2.4
  - Recommended: Latest Tempo 2.x stable
  - Features: Enhanced trace search, improved performance

- **Mimir**: 2.x
  - Minimum: Mimir 2.12
  - Recommended: Latest Mimir 2.x stable
  - Features: Improved query performance, enhanced storage efficiency

### Message Queue & Event Processing
- **Apache Kafka**: 3.8+
  - Minimum: Kafka 3.8
  - Recommended: Latest Kafka 3.x stable
  - Migration: From Laravel Queues to Kafka for microservices
  - AWS MSK: Use latest Kafka version available in Amazon MSK

- **Laravel Queues**: Latest with Laravel 11.x/12.x
  - Initial implementation before Kafka migration
  - Drivers: Redis, SQS, Database
  - Migration timeline: 6-12 months to Kafka

## Development & Testing Tools

### Testing Frameworks
- **PHPUnit**: 11.x
  - Minimum: PHPUnit 11.0
  - Laravel Integration: Laravel 11.x/12.x compatible version
  - Features: Improved assertions, better performance

- **Pest PHP**: 2.x
  - Alternative to PHPUnit for expressive testing
  - Laravel Integration: Full Laravel support
  - Features: Elegant syntax, plugin ecosystem

- **Jest**: 29.x
  - Minimum: Jest 29.7
  - React Testing: React Testing Library integration
  - Features: Improved ESM support, better snapshot testing

- **Playwright**: 1.45+
  - Minimum: Playwright 1.45
  - E2E Testing: Cross-browser automation
  - Features: Enhanced debugging, improved test isolation

### Development Tools
- **Composer**: 2.7+
  - PHP Dependency Management
  - Features: Improved performance, better security

- **ESLint**: 9.x
  - JavaScript/TypeScript Linting
  - Configuration: Flat config format
  - Plugins: React, TypeScript, Accessibility

- **Prettier**: 3.x
  - Code Formatting
  - Integration: ESLint, VS Code, CI/CD
  - Configuration: Consistent formatting rules

## AWS Infrastructure Versions

### Compute & Container Services
- **AWS EKS**: Latest supported Kubernetes version
  - Auto-upgrade: Enable automatic version updates
  - Node Groups: Use latest EKS-optimized AMIs
  - Add-ons: Latest compatible versions (VPC CNI, CoreDNS, kube-proxy)

- **AWS Fargate**: Latest platform version
  - Serverless containers
  - Automatic updates managed by AWS

### Database & Storage
- **AWS RDS**: Latest engine versions
  - MariaDB: Latest supported MariaDB 12.x or 11.x
  - Automatic Updates: Enable minor version auto-upgrades
  - Backup: Point-in-time recovery enabled

- **AWS ElastiCache**: Latest Redis engine versions
  - Redis: Latest supported Redis 7.x
  - Cluster Mode: Enable for high availability
  - Automatic Failover: Multi-AZ deployment

- **AWS S3**: Latest service features
  - Intelligent Tiering: Automatic cost optimization
  - Versioning: Enable for data protection
  - Encryption: Server-side encryption (SSE-S3 or SSE-KMS)

### Networking & Security
- **AWS VPC**: Latest networking features
  - IPv6 Support: Dual-stack configuration
  - Flow Logs: Enhanced monitoring
  - Security Groups: Least privilege access

- **AWS ALB**: Application Load Balancer latest features
  - HTTP/2 Support: Enhanced performance
  - WebSocket Support: Real-time applications
  - SSL/TLS: Latest protocol versions

## Security & Compliance Versions

### TLS/SSL
- **TLS Protocol**: 1.3 minimum
  - Cipher Suites: Modern, secure cipher suites only
  - Certificate Management: Automated renewal with Let's Encrypt or AWS ACM
  - HSTS: HTTP Strict Transport Security enabled

### PHP Security
- **Encryption**: OpenSSL 3.x
  - Laravel Encryption: AES-256-CBC
  - Hashing: Argon2id for password hashing
  - Random: Cryptographically secure random number generation

### Database Security
- **Connection Encryption**: TLS 1.3 for database connections
  - SSL Certificates: Valid, trusted certificates
  - Authentication: Strong password policies, IAM integration where possible

## Version Update Strategy

### Update Cadence
- **Security Updates**: Immediate (within 48 hours)
- **Minor Updates**: Monthly review and update cycle
- **Major Updates**: Quarterly evaluation, planned migration
- **LTS Updates**: Annual evaluation for major framework updates

### Testing Strategy
- **Staging Environment**: Mirror production versions exactly
- **Update Testing**: Comprehensive test suite execution before production updates
- **Rollback Plan**: Documented rollback procedures for all version updates
- **Monitoring**: Enhanced monitoring during and after version updates

### Documentation Requirements
- **Version Tracking**: Maintain current version inventory
- **Change Logs**: Document all version changes and rationale
- **Security Advisories**: Track and respond to security advisories
- **Compatibility Matrix**: Maintain compatibility information between components

## Deprecation Timeline

### Current Deprecation Notices
- **PHP 8.3**: End of life November 2026, migrate to PHP 8.4+ by Q2 2026
- **Laravel 11.x**: Migrate to Laravel 12.x LTS when available
- **Node.js 22.x**: End of life April 2027, current Node.js 24.x LTS is preferred
- **Kubernetes 1.29**: AWS EKS deprecation timeline follows Kubernetes releases

### Future Planning
- **React 19**: Monitor for stable release and migration timeline
- **PHP 9**: Monitor development for future migration planning
- **Kubernetes 1.32+**: Plan for annual Kubernetes version upgrades

This technology version standards document serves as the authoritative reference for all technology version decisions across the insurance system. Regular reviews and updates ensure the system remains current with security patches, performance improvements, and feature enhancements.