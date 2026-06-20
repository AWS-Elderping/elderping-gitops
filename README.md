# ElderPing GitOps & Kubernetes Configurations

This repository contains all Kubernetes configurations, Helm charts, ArgoCD application manifests, and environment-specific values files for continuous delivery.

---

## Directory Structure

```text
elderping-gitops/
├── argocd/
│   ├── projects/
│   │   └── elderping-project.yaml      # Consolidated EKS projects (healthcare, platform, etc.)
│   └── applications/
│       ├── root-app.yaml               # Root Application (App-of-Apps pattern)
│       ├── *-app.yaml                  # Individual application configurations for microservices
│       ├── prometheus-app.yaml         # Prometheus monitoring application
│       ├── loki-app.yaml               # Loki log aggregator application
│       ├── grafana-app.yaml            # Grafana dashboard application
│       ├── aws-load-balancer-...yaml   # ALB Ingress Controller application
│       ├── cluster-secret-...yaml      # External Secrets Store application
│       └── namespaces-app.yaml         # Namespaces configuration application
│
├── charts/                             # Reusable Application Helm Charts
│   ├── ai-service/
│   ├── alert-service/
│   ├── appointment-service/
│   ├── audit-service/
│   ├── auth-service/
│   ├── finops-service/
│   ├── health-service/
│   ├── notes-service/
│   ├── notification-service/
│   ├── reminder-service/
│   ├── report-service/
│   └── frontend/                       # React frontend chart (Renamed from ui-service)
│
├── environments/                       # Environment-specific values overrides
│   ├── dev/                            # Development environment (tag updates occur here)
│   ├── stage/                          # Staging environment
│   └── prod/                           # Production environment
│
└── infrastructure/                     # Infrastructure Middlewares & Raw Configurations
    ├── postgresql/                     # PostgreSQL StatefulSet and PVC manifests
    ├── prometheus/                     # Prometheus ServiceMonitor configuration
    ├── monitoring/                     # Grafana dashboards and telemetry
    ├── gateway/                        # KGateway and Ingress controller configurations
    ├── redis/                          # In-cluster Redis setups (Placeholder)
    ├── mongodb/                        # In-cluster MongoDB setups (Placeholder)
    ├── namespace.yaml                  # Merged Kubernetes namespace definitions
    └── secrets.yaml                    # Base64 placeholder secrets manifest
```

---

## ArgoCD App-of-Apps Pattern
ElderPing uses the **ArgoCD App-of-Apps pattern** to manage all cluster resources declaratively:
1. Deploys the AppProjects defined in `argocd/projects/elderping-project.yaml`.
2. Registers `argocd/applications/root-app.yaml` (the root application) in ArgoCD.
3. The root application scans the `argocd/applications/` directory and creates all sub-applications dynamically (namespaces, services, telemetry, operators).

---

## Image Tag Automation & CI/CD
To update microservices on EKS dynamically without changing baseline chart templates:
1. baseline settings are configured inside `charts/<service>/values.yaml`.
2. Environment-specific overrides (replicas, DB targets, ECR tags) are defined inside `environments/<env>/<service>-values.yaml`.
3. In a CI run, when a microservice is successfully built and pushed to ECR, a GitHub Action updates only the `image.tag` key inside `environments/<env>/<service>-values.yaml` in this repository:
   ```yaml
   image:
     repository: 462355914183.dkr.ecr.us-east-1.amazonaws.com/elderpinq-dev-auth-service
     tag: dev-20260620153022 # Updated automatically by CI/CD
   ```
4. ArgoCD automatically detects this commit in the GitOps repository, synchronizes the changes, and performs a rolling update on the corresponding Deployment.
