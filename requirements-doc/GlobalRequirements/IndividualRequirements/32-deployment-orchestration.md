# 32.0 Deployment & Orchestration

1. Kubernetes Manifests or Helm Charts
    - The images produced by the pipeline are referenced in Kubernetes deployments or Helm charts. Each microservice has a separate deployment configuration using environment variables from ConfigMaps and Secrets.
2. Namespace & Tenant Considerations
    - The same Docker images are reused in different namespaces with tenant-specific environment variables and/or secrets.
3. Integration with Observability
    - The Docker images include instrumentation for Prometheus, Loki, and Tempo, ensuring consistent logging and monitoring in production.