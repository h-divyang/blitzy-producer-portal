# 28-29-30.0 Comprehensive Docker Architecture Requirements - Updated

## Docker Strategy Overview

### Container-First Architecture
- **Multi-Stage Builds**: Optimized builds with minimal runtime images
- **Security-First**: Non-root users, minimal permissions, vulnerability scanning
- **Microservice-Ready**: Individual containers supporting monolith-to-microservice evolution
- **EKS-Optimized**: Container design optimized for AWS EKS deployment
- **Performance-Focused**: Layer caching, minimal image sizes, fast startup times

## Dockerfile Standards & Best Practices

### Laravel Backend Dockerfile
```dockerfile
# Dockerfile - Laravel Backend Multi-Stage Build
# Stage 1: Build dependencies and assets
FROM php:8.4-cli-alpine AS builder

# Install system dependencies
RUN apk add --no-cache \
    build-base \
    curl \
    git \
    libzip-dev \
    oniguruma-dev \
    postgresql-dev \
    mariadb-dev \
    nodejs \
    npm \
    && rm -rf /var/cache/apk/*

# Install PHP extensions
RUN docker-php-ext-install \
    bcmath \
    ctype \
    fileinfo \
    json \
    mbstring \
    pdo \
    pdo_mysql \
    pdo_pgsql \
    tokenizer \
    xml \
    zip

# Install Composer
COPY --from=composer:2.7 /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Copy composer files first for better caching
COPY composer.json composer.lock ./
RUN composer install --no-dev --no-scripts --no-autoloader --prefer-dist

# Copy application code
COPY . .

# Generate optimized autoloader and caches
RUN composer dump-autoload --optimize --no-dev \
    && php artisan config:cache \
    && php artisan route:cache \
    && php artisan view:cache

# Build frontend assets if needed
COPY package*.json ./
RUN npm ci --only=production \
    && npm run build \
    && rm -rf node_modules

# Stage 2: Runtime image
FROM php:8.4-fpm-alpine AS runtime

# Create app user
RUN addgroup -g 1001 -S appgroup \
    && adduser -u 1001 -S appuser -G appgroup

# Install runtime dependencies only
RUN apk add --no-cache \
    libzip \
    oniguruma \
    postgresql-libs \
    mariadb-connector-c \
    && rm -rf /var/cache/apk/*

# Install PHP extensions (runtime only)
RUN docker-php-ext-install \
    bcmath \
    ctype \
    fileinfo \
    json \
    mbstring \
    pdo \
    pdo_mysql \
    pdo_pgsql \
    tokenizer \
    xml \
    zip

# Set working directory
WORKDIR /var/www/html

# Copy application from builder stage
COPY --from=builder --chown=appuser:appgroup /var/www/html .

# Copy PHP-FPM configuration
COPY docker/php-fpm.conf /usr/local/etc/php-fpm.d/www.conf
COPY docker/php.ini /usr/local/etc/php/php.ini

# Create necessary directories
RUN mkdir -p /var/www/html/storage/logs \
    && mkdir -p /var/www/html/storage/framework/cache \
    && mkdir -p /var/www/html/storage/framework/sessions \
    && mkdir -p /var/www/html/storage/framework/views \
    && chown -R appuser:appgroup /var/www/html/storage \
    && chmod -R 755 /var/www/html/storage

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD php artisan health:check || exit 1

# Expose port
EXPOSE 9000

CMD ["php-fpm"]

# Stage 3: Nginx frontend proxy
FROM nginx:1.25-alpine AS web

# Copy nginx configuration
COPY docker/nginx.conf /etc/nginx/nginx.conf
COPY docker/default.conf /etc/nginx/conf.d/default.conf

# Copy static assets from builder
COPY --from=builder /var/www/html/public /var/www/html/public

# Create nginx user
RUN addgroup -g 1001 -S nginxgroup \
    && adduser -u 1001 -S nginxuser -G nginxgroup

# Change ownership
RUN chown -R nginxuser:nginxgroup /var/cache/nginx \
    && chown -R nginxuser:nginxgroup /var/log/nginx \
    && chown -R nginxuser:nginxgroup /etc/nginx/conf.d \
    && touch /var/run/nginx.pid \
    && chown nginxuser:nginxgroup /var/run/nginx.pid

USER nginxuser

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
```

### React Frontend Dockerfile
```dockerfile
# Dockerfile.frontend - React Frontend Multi-Stage Build
# Stage 1: Build application
FROM node:24-alpine AS builder

# Set working directory
WORKDIR /app

# Install dependencies first for better caching
COPY package*.json ./
RUN npm ci --only=production --cache /tmp/.npm

# Copy source code
COPY . .

# Build application
ENV NODE_ENV=production
RUN npm run build \
    && npm run optimize

# Stage 2: Runtime with Nginx
FROM nginx:1.25-alpine AS runtime

# Install security updates
RUN apk upgrade --no-cache

# Create non-root user
RUN addgroup -g 1001 -S appgroup \
    && adduser -u 1001 -S appuser -G appgroup

# Copy built application
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY docker/nginx.conf /etc/nginx/nginx.conf
COPY docker/frontend.conf /etc/nginx/conf.d/default.conf

# Copy security headers configuration
COPY docker/security-headers.conf /etc/nginx/conf.d/security-headers.conf

# Update permissions
RUN chown -R appuser:appgroup /usr/share/nginx/html \
    && chown -R appuser:appgroup /var/cache/nginx \
    && chown -R appuser:appgroup /var/log/nginx \
    && touch /var/run/nginx.pid \
    && chown appuser:appgroup /var/run/nginx.pid

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
```

### Microservice Dockerfile Template
```dockerfile
# Dockerfile.microservice - Template for microservices
FROM php:8.4-fpm-alpine AS base

# Install dependencies
RUN apk add --no-cache \
    curl \
    git \
    libzip-dev \
    oniguruma-dev \
    postgresql-dev \
    mariadb-dev \
    && rm -rf /var/cache/apk/*

# Install PHP extensions
RUN docker-php-ext-install \
    bcmath \
    pdo \
    pdo_mysql \
    pdo_pgsql \
    zip

# Install Composer
COPY --from=composer:2.7 /usr/bin/composer /usr/bin/composer

# Create app user
RUN addgroup -g 1001 -S appgroup \
    && adduser -u 1001 -S appuser -G appgroup

WORKDIR /app

# Stage 1: Dependencies
FROM base AS dependencies
COPY composer.json composer.lock ./
RUN composer install --no-dev --no-scripts --prefer-dist

# Stage 2: Application
FROM base AS application
COPY --from=dependencies /app/vendor ./vendor
COPY . .
RUN composer dump-autoload --optimize --no-dev

# Final stage
FROM base AS runtime
COPY --from=application --chown=appuser:appgroup /app .

USER appuser

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD php artisan health:check || exit 1

EXPOSE 9000
CMD ["php-fpm"]
```

## Docker Compose Development Environment

### Primary docker-compose.yml
```yaml
# docker-compose.yml - Local development environment
version: '3.8'

services:
  # Laravel Backend
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: runtime
    container_name: insurance-backend
    ports:
      - "8000:9000"
    volumes:
      - .:/var/www/html
      - ./docker/php-fpm.conf:/usr/local/etc/php-fpm.d/www.conf
      - ./docker/php.ini:/usr/local/etc/php/php.ini
    environment:
      - APP_ENV=local
      - APP_DEBUG=true
      - DB_CONNECTION=mysql
      - DB_HOST=database
      - DB_PORT=3306
      - DB_DATABASE=insurance
      - DB_USERNAME=insurance
      - DB_PASSWORD=secret
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - insurance-network
    healthcheck:
      test: ["CMD", "php", "artisan", "health:check"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Nginx Web Server
  web:
    build:
      context: .
      dockerfile: Dockerfile
      target: web
    container_name: insurance-web
    ports:
      - "80:8080"
    volumes:
      - ./public:/var/www/html/public
      - ./docker/nginx.conf:/etc/nginx/nginx.conf
      - ./docker/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - insurance-network

  # React Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.frontend
    container_name: insurance-frontend
    ports:
      - "3000:8080"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - REACT_APP_API_URL=http://localhost:8000
      - REACT_APP_WS_URL=ws://localhost:8000
    networks:
      - insurance-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # MariaDB Database
  database:
    image: mariadb:12-jammy
    container_name: insurance-database
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=insurance
      - MYSQL_USER=insurance
      - MYSQL_PASSWORD=secret
    volumes:
      - database-data:/var/lib/mysql
      - ./docker/mariadb/init:/docker-entrypoint-initdb.d
      - ./docker/mariadb/conf:/etc/mysql/conf.d
    networks:
      - insurance-network
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: insurance-redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
      - ./docker/redis/redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      - insurance-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Mailhog (Development Email)
  mailhog:
    image: mailhog/mailhog:v1.0.1
    container_name: insurance-mailhog
    ports:
      - "8025:8025"  # Web UI
      - "1025:1025"  # SMTP
    networks:
      - insurance-network

volumes:
  database-data:
    driver: local
  redis-data:
    driver: local

networks:
  insurance-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### Microservices docker-compose.override.yml
```yaml
# docker-compose.override.yml - Microservices development
version: '3.8'

services:
  # Policy Service (Future microservice)
  policy-service:
    build:
      context: ./services/policy
      dockerfile: Dockerfile.microservice
    container_name: insurance-policy-service
    ports:
      - "8001:9000"
    environment:
      - SERVICE_NAME=policy-service
      - DB_HOST=database
      - REDIS_HOST=redis
      - KAFKA_BROKERS=kafka:9092
    depends_on:
      - database
      - redis
      - kafka
    networks:
      - insurance-network
    profiles:
      - microservices

  # Claims Service (Future microservice)
  claims-service:
    build:
      context: ./services/claims
      dockerfile: Dockerfile.microservice
    container_name: insurance-claims-service
    ports:
      - "8002:9000"
    environment:
      - SERVICE_NAME=claims-service
      - DB_HOST=database
      - REDIS_HOST=redis
      - KAFKA_BROKERS=kafka:9092
    depends_on:
      - database
      - redis
      - kafka
    networks:
      - insurance-network
    profiles:
      - microservices

  # User Service (Future microservice)
  user-service:
    build:
      context: ./services/user
      dockerfile: Dockerfile.microservice
    container_name: insurance-user-service
    ports:
      - "8003:9000"
    environment:
      - SERVICE_NAME=user-service
      - DB_HOST=database
      - REDIS_HOST=redis
      - KAFKA_BROKERS=kafka:9092
    depends_on:
      - database
      - redis
      - kafka
    networks:
      - insurance-network
    profiles:
      - microservices

  # Apache Kafka (Event streaming)
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: insurance-kafka
    ports:
      - "9092:9092"
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092
      - KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
    depends_on:
      - zookeeper
    networks:
      - insurance-network
    profiles:
      - microservices

  # Zookeeper (Kafka dependency)
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    container_name: insurance-zookeeper
    ports:
      - "2181:2181"
    environment:
      - ZOOKEEPER_CLIENT_PORT=2181
      - ZOOKEEPER_TICK_TIME=2000
    networks:
      - insurance-network
    profiles:
      - microservices
```

### Testing docker-compose.test.yml
```yaml
# docker-compose.test.yml - Testing environment
version: '3.8'

services:
  # Test Database
  test-database:
    image: mariadb:12-jammy
    container_name: insurance-test-database
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=insurance_test
      - MYSQL_USER=insurance
      - MYSQL_PASSWORD=secret
    volumes:
      - ./docker/mariadb/test-init:/docker-entrypoint-initdb.d
    networks:
      - test-network
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Test Redis
  test-redis:
    image: redis:7-alpine
    container_name: insurance-test-redis
    networks:
      - test-network

  # Backend Tests
  test-backend:
    build:
      context: .
      dockerfile: Dockerfile
      target: runtime
    container_name: insurance-test-backend
    environment:
      - APP_ENV=testing
      - DB_CONNECTION=mysql
      - DB_HOST=test-database
      - DB_DATABASE=insurance_test
      - DB_USERNAME=insurance
      - DB_PASSWORD=secret
      - REDIS_HOST=test-redis
    volumes:
      - .:/var/www/html
      - ./coverage:/var/www/html/coverage
    command: |
      sh -c "
        php artisan migrate --env=testing --force &&
        php artisan db:seed --class=TestingSeeder &&
        php artisan test --coverage --coverage-html=coverage
      "
    depends_on:
      test-database:
        condition: service_healthy
      test-redis:
        condition: service_started
    networks:
      - test-network

  # Frontend Tests
  test-frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.frontend
      target: builder
    container_name: insurance-test-frontend
    volumes:
      - ./frontend:/app
      - /app/node_modules
      - ./frontend/coverage:/app/coverage
    command: |
      sh -c "
        npm run test:unit -- --coverage --watchAll=false &&
        npm run test:integration &&
        npm run lint &&
        npm run type-check
      "
    networks:
      - test-network

  # E2E Tests
  test-e2e:
    image: mcr.microsoft.com/playwright:v1.45.0-focal
    container_name: insurance-test-e2e
    environment:
      - BASE_URL=http://test-app:8080
    volumes:
      - ./e2e:/app
      - ./e2e/test-results:/app/test-results
    command: |
      sh -c "
        npm ci &&
        npx playwright test
      "
    depends_on:
      - test-app
    networks:
      - test-network

  # Test Application Instance
  test-app:
    build:
      context: .
      dockerfile: Dockerfile
      target: web
    container_name: insurance-test-app
    ports:
      - "8080:8080"
    environment:
      - APP_ENV=testing
      - DB_HOST=test-database
    depends_on:
      test-database:
        condition: service_healthy
    networks:
      - test-network

networks:
  test-network:
    driver: bridge
```

## Production-Ready Configuration

### EKS-Optimized Dockerfile
```dockerfile
# Dockerfile.production - Production-optimized builds
FROM php:8.4-fpm-alpine AS production

# Security updates
RUN apk upgrade --no-cache

# Install minimal runtime dependencies
RUN apk add --no-cache \
    curl \
    libzip \
    oniguruma \
    postgresql-libs \
    mariadb-connector-c \
    && rm -rf /var/cache/apk/*

# Install PHP extensions
RUN docker-php-ext-install \
    bcmath \
    opcache \
    pdo \
    pdo_mysql \
    pdo_pgsql \
    zip

# Configure OPcache for production
RUN echo "opcache.enable=1" >> /usr/local/etc/php/php.ini \
    && echo "opcache.memory_consumption=256" >> /usr/local/etc/php/php.ini \
    && echo "opcache.max_accelerated_files=20000" >> /usr/local/etc/php/php.ini \
    && echo "opcache.validate_timestamps=0" >> /usr/local/etc/php/php.ini

# Create app user with specific UID for EKS
RUN addgroup -g 1001 -S appgroup \
    && adduser -u 1001 -S appuser -G appgroup

WORKDIR /var/www/html

# Copy application (from CI build artifacts)
COPY --chown=appuser:appgroup . .

# Set correct permissions
RUN chown -R appuser:appgroup /var/www/html \
    && chmod -R 755 storage bootstrap/cache

# Security hardening
RUN chmod 600 .env 2>/dev/null || true
RUN find /var/www/html -type f -name "*.php" -exec chmod 644 {} \;
RUN find /var/www/html -type d -exec chmod 755 {} \;

USER appuser

# Health check optimized for Kubernetes
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:9000/fpm-ping || exit 1

EXPOSE 9000

CMD ["php-fpm"]
```

### Container Security Configuration
```yaml
# docker/security-scan.yml - Container security scanning
version: '3.8'

services:
  security-scan:
    image: aquasec/trivy:latest
    container_name: insurance-security-scan
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./security-reports:/reports
    command: |
      sh -c "
        trivy image --format json --output /reports/backend-scan.json insurance/backend:latest &&
        trivy image --format json --output /reports/frontend-scan.json insurance/frontend:latest &&
        trivy image --exit-code 1 --severity HIGH,CRITICAL insurance/backend:latest &&
        trivy image --exit-code 1 --severity HIGH,CRITICAL insurance/frontend:latest
      "
```

## Development Workflow Integration

### Docker Development Scripts
```bash
#!/bin/bash
# scripts/docker-dev.sh - Development workflow scripts

set -e

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Functions
log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Development environment setup
setup_dev() {
    log_info "Setting up development environment..."
    
    # Copy environment files
    cp .env.example .env
    
    # Build containers
    docker-compose build
    
    # Start services
    docker-compose up -d
    
    # Wait for database
    log_info "Waiting for database to be ready..."
    until docker-compose exec -T database mysqladmin ping -h"localhost" --silent; do
        sleep 1
    done
    
    # Run migrations and seeders
    docker-compose exec app php artisan migrate --seed
    
    # Install frontend dependencies
    docker-compose exec frontend npm install
    
    log_info "Development environment ready!"
    log_info "Backend: http://localhost:8000"
    log_info "Frontend: http://localhost:3000"
    log_info "Database: localhost:3306"
    log_info "Redis: localhost:6379"
    log_info "Mailhog: http://localhost:8025"
}

# Run tests
run_tests() {
    log_info "Running test suite..."
    
    # Backend tests
    docker-compose -f docker-compose.test.yml up --build test-backend
    
    # Frontend tests
    docker-compose -f docker-compose.test.yml up --build test-frontend
    
    # E2E tests
    docker-compose -f docker-compose.test.yml up --build test-e2e
    
    log_info "All tests completed!"
}

# Clean environment
clean_env() {
    log_info "Cleaning Docker environment..."
    
    docker-compose down -v
    docker-compose -f docker-compose.test.yml down -v
    docker system prune -f
    docker volume prune -f
    
    log_info "Environment cleaned!"
}

# Production build
build_production() {
    log_info "Building production images..."
    
    # Build backend
    docker build -f Dockerfile.production -t insurance/backend:latest .
    
    # Build frontend
    docker build -f frontend/Dockerfile.frontend -t insurance/frontend:latest ./frontend
    
    # Security scan
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
        aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL \
        insurance/backend:latest
        
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
        aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL \
        insurance/frontend:latest
    
    log_info "Production images built and scanned successfully!"
}

# Microservices setup
setup_microservices() {
    log_info "Setting up microservices environment..."
    
    # Start with microservices profile
    docker-compose --profile microservices up -d
    
    # Wait for Kafka
    log_info "Waiting for Kafka to be ready..."
    until docker-compose exec -T kafka kafka-topics --bootstrap-server localhost:9092 --list > /dev/null 2>&1; do
        sleep 2
    done
    
    # Create required topics
    docker-compose exec kafka kafka-topics --create --topic policy-events --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
    docker-compose exec kafka kafka-topics --create --topic claim-events --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
    docker-compose exec kafka kafka-topics --create --topic user-events --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
    
    log_info "Microservices environment ready!"
}

# Main command handler
case "${1:-setup}" in
    setup)
        setup_dev
        ;;
    test)
        run_tests
        ;;
    clean)
        clean_env
        ;;
    build)
        build_production
        ;;
    microservices)
        setup_microservices
        ;;
    *)
        echo "Usage: $0 {setup|test|clean|build|microservices}"
        exit 1
        ;;
esac
```

### Docker Configuration Files
```nginx
# docker/nginx.conf - Optimized Nginx configuration
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    # Performance
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # Security
    server_tokens off;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Gzip
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    include /etc/nginx/conf.d/*.conf;
}
```

```ini
; docker/php.ini - Optimized PHP configuration
[PHP]
; Performance
opcache.enable=1
opcache.memory_consumption=256
opcache.max_accelerated_files=20000
opcache.validate_timestamps=0
opcache.save_comments=1
opcache.load_comments=1

; Security
expose_php=Off
allow_url_fopen=Off
allow_url_include=Off
display_errors=Off
display_startup_errors=Off
log_errors=On
error_log=/var/log/php/php_errors.log

; Memory and execution
memory_limit=512M
max_execution_time=30
max_input_time=60
post_max_size=50M
upload_max_filesize=50M
max_file_uploads=20

; Session security
session.cookie_httponly=On
session.cookie_secure=On
session.use_strict_mode=On
session.cookie_samesite=Strict

; Date
date.timezone=UTC
```

This comprehensive Docker architecture provides a robust foundation for both monolithic and microservices deployments, with security, performance, and maintainability as core principles. The configuration supports the full development lifecycle from local development through production deployment on AWS EKS.