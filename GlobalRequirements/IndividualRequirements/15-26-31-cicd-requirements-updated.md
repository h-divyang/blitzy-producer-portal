# 15-26-31.0 Comprehensive CI/CD Requirements - Updated

## CI/CD Strategy Overview

### Unified DevOps Philosophy
- **GitOps Approach**: Infrastructure and application deployment managed through Git
- **Multi-Tenant Pipeline**: Single pipeline supporting multiple tenant deployments
- **Microservice-Ready**: Modular pipelines supporting monolith-to-microservice evolution
- **Security-First**: Integrated security scanning and compliance validation
- **Zero-Downtime Deployments**: Blue-green and canary deployment strategies

## GitLab CI/CD Pipeline Architecture

### Pipeline Structure
```yaml
# .gitlab-ci.yml - Comprehensive CI/CD Pipeline
stages:
  - validate
  - build
  - test
  - security
  - package
  - deploy-staging
  - integration-tests
  - deploy-production
  - post-deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  REGISTRY: $CI_REGISTRY
  IMAGE_TAG: $CI_COMMIT_SHA
  HELM_CHART_VERSION: "1.0.0"

# Global configuration
default:
  image: php:8.4-cli
  before_script:
    - echo "Pipeline started for commit $CI_COMMIT_SHA"
    - echo "Branch: $CI_COMMIT_REF_NAME"
    - echo "Environment: $CI_ENVIRONMENT_NAME"

include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
  - local: '.gitlab/ci/microservices.yml'
  - local: '.gitlab/ci/deployment.yml'
```

### Build Stage Configuration
```yaml
# Build stage - Multi-service build support
build:backend:
  stage: build
  image: php:8.4-cli
  services:
    - docker:24-dind
  cache:
    key: composer-cache
    paths:
      - vendor/
  before_script:
    - apt-get update && apt-get install -y git curl libzip-dev unzip
    - curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - composer install --no-dev --optimize-autoloader --no-interaction
    - php artisan config:cache
    - php artisan route:cache
    - php artisan view:cache
    - docker build -t $REGISTRY/insurance/backend:$IMAGE_TAG .
    - docker push $REGISTRY/insurance/backend:$IMAGE_TAG
  artifacts:
    paths:
      - vendor/
      - bootstrap/cache/
    expire_in: 1 hour
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

build:frontend:
  stage: build
  image: node:24-alpine
  cache:
    key: npm-cache
    paths:
      - node_modules/
      - .npm/
  before_script:
    - npm config set cache .npm
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - npm ci --cache .npm --prefer-offline
    - npm run build
    - npm run optimize
    - docker build -f Dockerfile.frontend -t $REGISTRY/insurance/frontend:$IMAGE_TAG .
    - docker push $REGISTRY/insurance/frontend:$IMAGE_TAG
  artifacts:
    paths:
      - dist/
      - build/
    expire_in: 1 hour
    reports:
      dotenv: build.env
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# Microservice builds (future-ready)
build:policy-service:
  extends: .microservice-build
  variables:
    SERVICE_NAME: policy-service
    DOCKERFILE_PATH: services/policy/Dockerfile
  rules:
    - if: $MICROSERVICES_ENABLED == "true"
    - changes:
        - services/policy/**/*

build:claims-service:
  extends: .microservice-build
  variables:
    SERVICE_NAME: claims-service
    DOCKERFILE_PATH: services/claims/Dockerfile
  rules:
    - if: $MICROSERVICES_ENABLED == "true"
    - changes:
        - services/claims/**/*

# Microservice build template
.microservice-build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -f $DOCKERFILE_PATH -t $REGISTRY/insurance/$SERVICE_NAME:$IMAGE_TAG .
    - docker push $REGISTRY/insurance/$SERVICE_NAME:$IMAGE_TAG
    - echo "IMAGE_$SERVICE_NAME=$REGISTRY/insurance/$SERVICE_NAME:$IMAGE_TAG" >> service_images.env
  artifacts:
    reports:
      dotenv: service_images.env
```

### Testing Stage Integration
```yaml
# Testing stage - Comprehensive test execution
test:backend:
  stage: test
  image: php:8.4-cli
  services:
    - mysql:8.0
    - redis:7-alpine
  variables:
    MYSQL_ROOT_PASSWORD: secret
    MYSQL_DATABASE: insurance_test
    MYSQL_USER: insurance
    MYSQL_PASSWORD: secret
    REDIS_URL: redis://redis:6379
  cache:
    key: composer-cache
    paths:
      - vendor/
    policy: pull
  before_script:
    - apt-get update && apt-get install -y git curl libzip-dev unzip mysql-client
    - curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
    - cp .env.testing .env
    - php artisan key:generate
    - sleep 10  # Wait for MySQL to be ready
    - php artisan migrate --env=testing --force
    - php artisan db:seed --class=TestingSeeder
  script:
    - php artisan test --parallel --coverage --min=80
    - php artisan test --group=unit --testdox
    - php artisan test --group=integration --testdox
    - php artisan test --group=feature --testdox
  coverage: '/^\s*Lines:\s*\d+.\d+\%/'
  artifacts:
    reports:
      junit: storage/logs/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
    paths:
      - coverage/
    expire_in: 7 days
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

test:frontend:
  stage: test
  image: node:24-alpine
  cache:
    key: npm-cache
    paths:
      - node_modules/
      - .npm/
    policy: pull
  before_script:
    - npm config set cache .npm
  script:
    - npm ci --cache .npm --prefer-offline
    - npm run lint
    - npm run type-check
    - npm run test:unit -- --coverage --watchAll=false
    - npm run test:integration
    - npm run test:accessibility
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      junit: coverage/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 7 days
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

test:e2e:
  stage: test
  image: mcr.microsoft.com/playwright:v1.45.0-focal
  services:
    - name: $REGISTRY/insurance/backend:$IMAGE_TAG
      alias: backend
    - name: $REGISTRY/insurance/frontend:$IMAGE_TAG  
      alias: frontend
    - mysql:8.0
  variables:
    MYSQL_ROOT_PASSWORD: secret
    MYSQL_DATABASE: insurance_test
    BACKEND_URL: http://backend:8000
    FRONTEND_URL: http://frontend:3000
  dependencies:
    - build:backend
    - build:frontend
  before_script:
    - npm ci
    - npx playwright install
    - sleep 30  # Wait for services to be ready
  script:
    - npm run test:e2e:ci
    - npm run test:e2e:mobile
  artifacts:
    when: always
    paths:
      - test-results/
      - playwright-report/
    expire_in: 7 days
  allow_failure: false
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"
```

### Security & Quality Gates
```yaml
# Security scanning stage
security:sast:
  stage: security
  extends: .sast
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

security:dependency-scan:
  stage: security
  extends: .dependency_scanning
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"

security:container-scan:
  stage: security
  extends: .container_scanning
  variables:
    CS_IMAGE: $REGISTRY/insurance/backend:$IMAGE_TAG
  dependencies:
    - build:backend
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"

security:secrets-detection:
  stage: security
  image: trufflesecurity/trufflehog:latest
  script:
    - trufflehog git file://. --since-commit HEAD~1 --only-verified --fail
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# Custom security scans
security:owasp-zap:
  stage: security
  image: owasp/zap2docker-stable
  variables:
    ZAPROXY_TARGET_URL: http://frontend:3000
  services:
    - name: $REGISTRY/insurance/frontend:$IMAGE_TAG
      alias: frontend
  script:
    - mkdir -p /zap/wrk/
    - zap-baseline.py -t $ZAPROXY_TARGET_URL -g gen.conf -r zap-report.html
  artifacts:
    paths:
      - zap-report.html
    expire_in: 7 days
  allow_failure: true
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# Code quality analysis
quality:sonarqube:
  stage: security
  image: sonarsource/sonar-scanner-cli:latest
  script:
    - sonar-scanner
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```

## Kubernetes Deployment Architecture

### Helm Charts Structure
```yaml
# values.yaml - Helm chart values for insurance application
global:
  registry: registry.gitlab.com/insurance/app
  imageTag: "latest"
  environment: production
  
app:
  backend:
    image: 
      repository: insurance/backend
      tag: ""  # Overridden by CI
    replicas: 3
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    
  frontend:
    image:
      repository: insurance/frontend
      tag: ""  # Overridden by CI
    replicas: 2
    resources:
      requests:
        memory: "128Mi"
        cpu: "50m"
      limits:
        memory: "256Mi"
        cpu: "200m"

database:
  enabled: false  # Use AWS RDS
  external:
    host: insurance-prod.cluster-abc123.us-west-2.rds.amazonaws.com
    port: 3306
    database: insurance_prod

redis:
  enabled: false  # Use AWS ElastiCache
  external:
    host: insurance-prod.cache.amazonaws.com
    port: 6379

ingress:
  enabled: true
  className: kong
  annotations:
    kubernetes.io/ingress.class: kong
    konghq.com/plugins: rate-limiting, cors, jwt
  hosts:
    - host: api.insurance.com
      paths:
        - path: /
          pathType: Prefix
          service: backend
    - host: app.insurance.com  
      paths:
        - path: /
          pathType: Prefix
          service: frontend

# Multi-tenant namespace configuration
tenants:
  - name: tenant-a
    namespace: insurance-tenant-a
    database:
      host: tenant-a.cluster-abc123.us-west-2.rds.amazonaws.com
    domain: tenant-a.insurance.com
    
  - name: tenant-b
    namespace: insurance-tenant-b
    database:
      host: tenant-b.cluster-abc123.us-west-2.rds.amazonaws.com
    domain: tenant-b.insurance.com
```

### Deployment Pipeline Stages
```yaml
# Staging deployment
deploy:staging:
  stage: deploy-staging
  image: alpine/helm:3.12.0
  environment:
    name: staging
    url: https://staging.insurance.com
  before_script:
    - kubectl config use-context staging
    - helm repo add insurance-charts https://charts.insurance.com
  script:
    - helm upgrade --install insurance-staging insurance-charts/insurance
        --namespace insurance-staging
        --create-namespace
        --set global.imageTag=$IMAGE_TAG
        --set global.environment=staging
        --set backend.image.tag=$IMAGE_TAG
        --set frontend.image.tag=$IMAGE_TAG
        --values helm/values-staging.yaml
        --wait --timeout=10m
  dependencies:
    - build:backend
    - build:frontend
    - test:backend
    - test:frontend
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"

# Production deployment with approval
deploy:production:
  stage: deploy-production
  image: alpine/helm:3.12.0
  environment:
    name: production
    url: https://app.insurance.com
  when: manual
  before_script:
    - kubectl config use-context production
    - helm repo add insurance-charts https://charts.insurance.com
  script:
    # Blue-green deployment strategy
    - |
      if helm status insurance-production-blue > /dev/null 2>&1; then
        CURRENT_COLOR=blue
        NEW_COLOR=green
      else
        CURRENT_COLOR=green  
        NEW_COLOR=blue
      fi
    - echo "Deploying to $NEW_COLOR environment"
    - helm upgrade --install insurance-production-$NEW_COLOR insurance-charts/insurance
        --namespace insurance-production
        --create-namespace
        --set global.imageTag=$IMAGE_TAG
        --set global.environment=production
        --set global.deploymentColor=$NEW_COLOR
        --set backend.image.tag=$IMAGE_TAG
        --set frontend.image.tag=$IMAGE_TAG
        --values helm/values-production.yaml
        --wait --timeout=15m
    # Health check and traffic switch
    - sleep 30
    - ./scripts/health-check.sh insurance-production-$NEW_COLOR
    - ./scripts/switch-traffic.sh $NEW_COLOR
    - sleep 60  # Grace period
    - helm uninstall insurance-production-$CURRENT_COLOR
  dependencies:
    - test:e2e
    - security:sast
    - security:container-scan
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

## Multi-Tenant Deployment Strategy

### Tenant Provisioning Automation
```yaml
# Tenant onboarding pipeline
provision:tenant:
  stage: deploy-staging
  image: alpine/helm:3.12.0
  script:
    - |
      TENANT_NAME=$TENANT_NAME
      TENANT_DOMAIN=$TENANT_DOMAIN
      TENANT_NAMESPACE=insurance-$TENANT_NAME
      
      # Create namespace
      kubectl create namespace $TENANT_NAMESPACE --dry-run=client -o yaml | kubectl apply -f -
      
      # Label namespace for tenant isolation
      kubectl label namespace $TENANT_NAMESPACE tenant=$TENANT_NAME
      kubectl label namespace $TENANT_NAMESPACE istio-injection=enabled
      
      # Create tenant-specific secrets
      kubectl create secret generic tenant-config \
        --from-literal=tenant-id=$TENANT_ID \
        --from-literal=database-host=$TENANT_DB_HOST \
        --from-literal=database-name=$TENANT_DB_NAME \
        --namespace=$TENANT_NAMESPACE
        
      # Deploy application for tenant
      helm upgrade --install insurance-$TENANT_NAME insurance-charts/insurance \
        --namespace $TENANT_NAMESPACE \
        --set global.tenant.name=$TENANT_NAME \
        --set global.tenant.domain=$TENANT_DOMAIN \
        --set database.external.host=$TENANT_DB_HOST \
        --set ingress.hosts[0].host=$TENANT_DOMAIN \
        --values helm/values-tenant.yaml
        
      # Configure network policies
      kubectl apply -f k8s/network-policies/tenant-isolation.yaml -n $TENANT_NAMESPACE
      
      # Setup monitoring
      kubectl apply -f k8s/monitoring/tenant-monitoring.yaml -n $TENANT_NAMESPACE
  when: manual
  variables:
    TENANT_NAME: ""      # Set when running
    TENANT_DOMAIN: ""    # Set when running  
    TENANT_ID: ""        # Set when running
    TENANT_DB_HOST: ""   # Set when running
    TENANT_DB_NAME: ""   # Set when running
```

### Canary Deployment for Tenants
```yaml
# Canary deployment to subset of tenants
deploy:canary:
  stage: deploy-production
  image: alpine/helm:3.12.0
  environment:
    name: production-canary
  script:
    - |
      # Get canary tenant list
      CANARY_TENANTS=$(cat deployment/canary-tenants.txt)
      
      for tenant in $CANARY_TENANTS; do
        echo "Deploying canary to tenant: $tenant"
        
        helm upgrade insurance-$tenant insurance-charts/insurance \
          --namespace insurance-$tenant \
          --set global.imageTag=$IMAGE_TAG \
          --set global.canaryDeployment=true \
          --set backend.replicas=1 \
          --wait --timeout=10m
          
        # Monitor canary metrics
        ./scripts/monitor-canary.sh $tenant &
      done
      
      # Wait for canary validation
      sleep 300
      
      # Check canary success metrics
      ./scripts/validate-canary.sh $CANARY_TENANTS
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CANARY_DEPLOYMENT == "true"
```

## Version Control & Rollout Management

### Version Tagging Strategy
```yaml
# Semantic versioning and tagging
tag:version:
  stage: package
  image: alpine/git:latest
  before_script:
    - apk add --no-cache curl jq
  script:
    - |
      # Determine version bump type
      if echo "$CI_COMMIT_MESSAGE" | grep -q "BREAKING CHANGE"; then
        BUMP_TYPE="major"
      elif echo "$CI_COMMIT_MESSAGE" | grep -qE "^feat"; then
        BUMP_TYPE="minor"  
      else
        BUMP_TYPE="patch"
      fi
      
      # Get current version
      CURRENT_VERSION=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
      
      # Calculate new version
      NEW_VERSION=$(./scripts/bump-version.sh $CURRENT_VERSION $BUMP_TYPE)
      
      # Create tag
      git tag -a $NEW_VERSION -m "Release $NEW_VERSION"
      git push origin $NEW_VERSION
      
      # Update container images with version tag
      docker tag $REGISTRY/insurance/backend:$IMAGE_TAG $REGISTRY/insurance/backend:$NEW_VERSION
      docker tag $REGISTRY/insurance/frontend:$IMAGE_TAG $REGISTRY/insurance/frontend:$NEW_VERSION
      docker push $REGISTRY/insurance/backend:$NEW_VERSION
      docker push $REGISTRY/insurance/frontend:$NEW_VERSION
      
      echo "NEW_VERSION=$NEW_VERSION" >> version.env
  artifacts:
    reports:
      dotenv: version.env
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# Create GitHub/GitLab release
create:release:
  stage: package
  image: alpine:latest
  dependencies:
    - tag:version
  before_script:
    - apk add --no-cache curl
  script:
    - |
      # Generate changelog
      ./scripts/generate-changelog.sh $NEW_VERSION > CHANGELOG.md
      
      # Create release
      curl -X POST \
        -H "Authorization: Bearer $GITLAB_TOKEN" \
        -H "Content-Type: application/json" \
        -d "{
          \"name\": \"Release $NEW_VERSION\",
          \"tag_name\": \"$NEW_VERSION\",
          \"description\": \"$(cat CHANGELOG.md | jq -Rs .)\",
          \"assets\": {
            \"links\": [
              {
                \"name\": \"Backend Image\",
                \"url\": \"$REGISTRY/insurance/backend:$NEW_VERSION\"
              },
              {
                \"name\": \"Frontend Image\", 
                \"url\": \"$REGISTRY/insurance/frontend:$NEW_VERSION\"
              }
            ]
          }
        }" \
        "$CI_API_V4_URL/projects/$CI_PROJECT_ID/releases"
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

## Environment & Secrets Management

### HashiCorp Vault Integration
```yaml
# Vault secrets management
.vault-auth: &vault-auth
  - export VAULT_ADDR=$VAULT_URL
  - export VAULT_TOKEN=$(vault write -field=token auth/jwt/login role=gitlab-ci jwt=$CI_JOB_JWT)

secrets:staging:
  stage: deploy-staging
  image: vault:latest
  before_script:
    - *vault-auth
  script:
    - |
      # Retrieve secrets from Vault
      DB_PASSWORD=$(vault kv get -field=password secret/staging/database)
      JWT_SECRET=$(vault kv get -field=secret secret/staging/jwt)
      API_KEYS=$(vault kv get -format=json secret/staging/api-keys)
      
      # Create Kubernetes secrets
      kubectl create secret generic app-secrets \
        --from-literal=db-password="$DB_PASSWORD" \
        --from-literal=jwt-secret="$JWT_SECRET" \
        --from-literal=api-keys="$API_KEYS" \
        --namespace=insurance-staging \
        --dry-run=client -o yaml | kubectl apply -f -
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"

secrets:production:
  stage: deploy-production
  image: vault:latest
  before_script:
    - *vault-auth
  script:
    - |
      # Production secrets with rotation
      DB_PASSWORD=$(vault kv get -field=password secret/production/database)
      JWT_SECRET=$(vault kv get -field=secret secret/production/jwt)
      ENCRYPTION_KEY=$(vault kv get -field=key secret/production/encryption)
      
      # Update secrets with zero-downtime rotation
      kubectl create secret generic app-secrets-new \
        --from-literal=db-password="$DB_PASSWORD" \
        --from-literal=jwt-secret="$JWT_SECRET" \
        --from-literal=encryption-key="$ENCRYPTION_KEY" \
        --namespace=insurance-production \
        --dry-run=client -o yaml | kubectl apply -f -
        
      # Rolling update to use new secrets
      kubectl patch deployment backend \
        -p '{"spec":{"template":{"metadata":{"labels":{"secretsVersion":"'$(date +%s)'"}}}}}'
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

### Environment-Specific Configuration
```yaml
# Environment configuration management
variables:
  # Global variables
  HELM_CHART_VERSION: "1.0.0"
  KUBERNETES_VERSION: "1.30"
  
  # Staging environment
  STAGING_NAMESPACE: "insurance-staging"
  STAGING_DOMAIN: "staging.insurance.com"
  STAGING_REPLICAS: "1"
  STAGING_RESOURCES_CPU: "100m"
  STAGING_RESOURCES_MEMORY: "256Mi"
  
  # Production environment  
  PROD_NAMESPACE: "insurance-production"
  PROD_DOMAIN: "app.insurance.com"
  PROD_REPLICAS: "3"
  PROD_RESOURCES_CPU: "500m"
  PROD_RESOURCES_MEMORY: "1Gi"
  
  # Feature flags
  MICROSERVICES_ENABLED: "false"
  CANARY_DEPLOYMENT: "false"
  BLUE_GREEN_DEPLOYMENT: "true"
```

## Monitoring & Observability Integration

### Pipeline Monitoring
```yaml
# Pipeline metrics and monitoring
monitor:pipeline:
  stage: post-deploy
  image: curlimages/curl:latest
  script:
    - |
      # Send pipeline metrics to monitoring
      curl -X POST $MONITORING_ENDPOINT/api/v1/metrics \
        -H "Authorization: Bearer $MONITORING_TOKEN" \
        -d '{
          "pipeline_id": "'$CI_PIPELINE_ID'",
          "commit_sha": "'$CI_COMMIT_SHA'",
          "branch": "'$CI_COMMIT_REF_NAME'",
          "duration": '$CI_PIPELINE_DURATION',
          "status": "success",
          "tests": {
            "backend_coverage": '$BACKEND_COVERAGE',
            "frontend_coverage": '$FRONTEND_COVERAGE',
            "e2e_tests": '$E2E_TEST_COUNT'
          },
          "deployments": {
            "staging": "'$STAGING_DEPLOYED'",
            "production": "'$PRODUCTION_DEPLOYED'"
          }
        }'
        
      # Update deployment status in Grafana
      curl -X POST $GRAFANA_ENDPOINT/api/annotations \
        -H "Authorization: Bearer $GRAFANA_TOKEN" \
        -d '{
          "dashboardUID": "insurance-deployments",
          "time": '$CI_PIPELINE_CREATED_AT',
          "timeEnd": '$CI_PIPELINE_FINISHED_AT',
          "tags": ["deployment", "'$CI_COMMIT_REF_NAME'"],
          "text": "Deployed '$CI_COMMIT_SHA' to '$CI_ENVIRONMENT_NAME'"
        }'
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"
  when: always
```

### Application Health Monitoring
```bash
#!/bin/bash
# scripts/health-check.sh - Comprehensive health monitoring

DEPLOYMENT_NAME=$1
NAMESPACE=${2:-insurance-production}
TIMEOUT=${3:-300}

echo "Performing health check for $DEPLOYMENT_NAME in namespace $NAMESPACE"

# Wait for rollout to complete
kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE --timeout=${TIMEOUT}s

# Check pod readiness
PODS=$(kubectl get pods -l app=$DEPLOYMENT_NAME -n $NAMESPACE -o jsonpath='{.items[*].metadata.name}')

for pod in $PODS; do
    echo "Checking health of pod: $pod"
    
    # Check pod status
    POD_STATUS=$(kubectl get pod $pod -n $NAMESPACE -o jsonpath='{.status.phase}')
    if [ "$POD_STATUS" != "Running" ]; then
        echo "Pod $pod is not running: $POD_STATUS"
        exit 1
    fi
    
    # Check readiness probe
    READY=$(kubectl get pod $pod -n $NAMESPACE -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}')
    if [ "$READY" != "True" ]; then
        echo "Pod $pod is not ready"
        exit 1
    fi
    
    # Application-specific health checks
    kubectl exec $pod -n $NAMESPACE -- curl -f http://localhost:8000/health || exit 1
done

echo "Health check passed for $DEPLOYMENT_NAME"

# Performance validation
echo "Running performance validation..."
./scripts/performance-check.sh $DEPLOYMENT_NAME $NAMESPACE

echo "Health check and performance validation completed successfully"
```

This comprehensive CI/CD framework provides a robust, scalable pipeline supporting the evolution from monolith to microservices while maintaining security, quality, and operational excellence throughout the development and deployment lifecycle.