# Argonaut - Kubernetes GitOps Deployment - Claude Code Context

## Project Overview
Argonaut is the GitOps repository that manages Kubernetes deployments for all IM Apps services using ArgoCD. It provides declarative, automated deployment and management of staging and production environments on your Kubernetes cluster.

## Architecture
- **GitOps Tool**: ArgoCD for continuous deployment
- **Container Orchestration**: Kubernetes cluster
- **Configuration Management**: Kustomize for environment-specific overlays
- **Service Mesh**: Traefik ingress controller
- **Load Balancer**: MetalLB for bare-metal load balancing
- **Secret Management**: Sealed Secrets for encrypted secrets in Git

## Project Structure
```
argonaut/
├── argocd-apps/                    # ArgoCD Application definitions
│   ├── shoppingo-staging.yaml     # Shoppingo staging deployment
│   ├── shoppingo-production.yaml  # Shoppingo production deployment
│   ├── kivo-staging.yaml          # Kivo auth service staging
│   ├── kivo-production.yaml       # Kivo auth service production
│   ├── jewellery-catalogue-*.yaml # Jewellery catalogue deployments
│   └── metallb.yaml               # Load balancer configuration
└── apps/                          # Kubernetes manifests per application
    ├── shoppingo/                 # Shoppingo K8s resources
    │   ├── base/                  # Base Kustomize configuration
    │   │   ├── deployment.yaml         # API deployment
    │   │   ├── web-deployment.yaml     # Web frontend deployment
    │   │   ├── mongodb-deployment.yaml # MongoDB database
    │   │   ├── service.yaml            # API service
    │   │   ├── web-service.yaml        # Web service
    │   │   ├── ingress.yaml            # Traefik ingress rules
    │   │   ├── *-sealedsecret.yaml     # Encrypted secrets
    │   │   └── kustomization.yaml      # Base resources
    │   └── overlays/              # Environment-specific configurations
    │       ├── staging/           # Staging environment patches
    │       └── production/        # Production environment patches
    ├── kivo/                      # Kivo auth service K8s resources
    │   ├── base/                  # Base configuration
    │   └── overlays/              # Environment overlays
    ├── jewellery-catalogue/       # Jewellery catalogue service
    └── metallb/                   # Load balancer configuration
```

## Deployed Applications

### Core Services
- **shoppingo**: E-commerce application (API + Web frontend)
  - Staging: `shoppingo.imapps.staging`
  - Production: `shoppingo.imapps.co.uk`

- **kivo**: Authentication service
  - Staging: Available internally for shoppingo-staging
  - Production: Available internally for shoppingo-production

- **jewellery-catalogue**: Additional e-commerce service
  - Staging: `jewellerycatalogue.imapps.staging`
  - Production: `jewellerycatalogue.imapps.co.uk`

### Infrastructure Services
- **metallb**: Bare-metal load balancer for service exposure
- **traefik**: Ingress controller for HTTP routing (configured separately)

## GitOps Workflow

### ArgoCD Applications
Each service has ArgoCD Applications for staging and production:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: shoppingo-staging
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/igor-siergiej/shoppingo'
    targetRevision: HEAD
    path: 'apps/shoppingo/overlays/staging'
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: shoppingo-staging
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Deployment Process
1. **Code Changes**: Developers push code to application repositories
2. **CI/CD Pipeline**: GitHub Actions build and push container images
3. **GitOps Update**: CI updates Kubernetes manifests in this repository
4. **ArgoCD Sync**: ArgoCD detects changes and deploys to cluster
5. **Automated Healing**: ArgoCD ensures actual state matches desired state

## Kustomize Configuration

### Base Resources
Each application has a `base/` directory with core Kubernetes resources:
- **Deployments**: Container specifications and replicas
- **Services**: Internal service discovery and load balancing
- **ConfigMaps**: Non-sensitive configuration
- **SealedSecrets**: Encrypted secrets for sensitive data
- **Ingress**: External traffic routing via Traefik

### Environment Overlays
Environment-specific configurations in `overlays/`:
- **staging/**: Development environment patches (single replica, staging domains)
- **production/**: Production environment patches (multiple replicas, production domains)

## Infrastructure Details

### Container Registry
- **Private Registry**: `192.168.68.54:31834`
- **Image Format**: `registry/service-name:tag`
- **Example**: `192.168.68.54:31834/shoppingo-api:v0.2.14`

### Networking
- **Ingress Controller**: Traefik with HTTP entrypoints
- **Load Balancer**: MetalLB for bare-metal cluster
- **DNS Routing**: Host-based routing for different services
- **Internal Communication**: Kubernetes service discovery

### Secrets Management
- **Tool**: Sealed Secrets for GitOps-compatible secret management
- **Encryption**: Secrets encrypted with cluster-specific key
- **Examples**: Database credentials, API keys, JWT secrets

## Environment Configuration

### Staging Environment
- **Namespace Pattern**: `{service}-staging`
- **Domain Pattern**: `{service}.imapps.staging`
- **Database**: Shared or dedicated MongoDB per service
- **Resource Limits**: Lower limits for cost optimization

### Production Environment
- **Namespace Pattern**: `{service}-production`
- **Domain Pattern**: `{service}.imapps.co.uk`
- **Database**: Dedicated MongoDB with persistence
- **Resource Limits**: Higher limits for performance

## Service Architecture

### Shoppingo Deployment
- **API Container**: Node.js Koa API server (port 4001)
- **Web Container**: React frontend served via Nginx (port 80)
- **Database**: MongoDB with persistent volume
- **Storage**: MinIO object storage integration
- **Ingress**: Routes `/api/*` to API, everything else to web

### Kivo Deployment
- **Auth Container**: Node.js authentication service (port 3008)
- **Database**: Dedicated MongoDB for user data
- **Internal Service**: Accessed by other services, not exposed externally

## Monitoring and Operations

### ArgoCD Dashboard
- **Access**: ArgoCD UI for deployment status and synchronization
- **Sync Status**: Real-time view of application health
- **History**: Deployment history and rollback capabilities

### Kubernetes Operations
- **Logs**: `kubectl logs` for application debugging
- **Status**: `kubectl get pods` for pod status
- **Secrets**: Sealed secrets for sensitive configuration

## Useful Commands for Claude

- This is hosted on another server so won't have access to the cluster 

### ArgoCD Operations
- **Check app status**: Via ArgoCD UI or `argocd app get {app-name}`
- **Manual sync**: `argocd app sync {app-name}`
- **App health**: Monitor via ArgoCD dashboard

### Kubernetes Operations
- **Pod status**: `kubectl get pods -n {namespace}`
- **Logs**: `kubectl logs -f deployment/{service} -n {namespace}`
- **Secrets**: Check sealed secrets encryption status

### GitOps Updates
- **Update image**: Modify deployment YAML with new image tag
- **Environment changes**: Update Kustomize overlays for environment-specific config
- **New secrets**: Use `kubeseal` to encrypt secrets before committing

## Integration with Application Repositories
- **Source of Truth**: This repository defines infrastructure, application repos define code
- **Image Updates**: CI/CD pipelines update image tags in deployment manifests
- **Configuration**: Environment-specific config managed through Kustomize overlays
- **Secrets**: Sensitive data encrypted and stored as SealedSecrets

## Security Considerations
- **Network Policies**: Implemented via Kubernetes network policies
- **Secret Management**: All secrets encrypted with Sealed Secrets
- **Access Control**: Kubernetes RBAC for service accounts
- **Image Security**: Private container registry with controlled access
