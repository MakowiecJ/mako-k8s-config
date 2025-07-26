# Service Communication Map

## Overview
This document outlines the corrected service communication configuration for the Mako Bank microservices architecture.

## Service Port Configuration

### Frontend Services (Web)
- **mako-shell**: Port 80 (service) → 4200 (container)
- **mako-dashboard-web**: Port 4201 (service) → 4201 (container)
- **mako-accounts-web**: Port 4202 (service) → 4202 (container)
- **mako-transactions-web**: Port 4203 (service) → 4203 (container)
- **mako-users-web**: Port 4204 (service) → 4204 (container)

### Backend Services (API)
- **mako-api-gateway**: Port 8081 (service) → 8081 (container)
- **mako-service-registry**: Port 8761 (service) → 8761 (container)
- **mako-users**: Port 8082 (service) → 8082 (container)
- **mako-accounts**: Port 8083 (service) → 8083 (container)
- **mako-transactions**: Port 8084 (service) → 8084 (container)
- **mako-notifications**: Port 8085 (service) → 8085 (container)

### Infrastructure Services
- **keycloak**: Port 80 (service) → 8080 (container)
- **cloudsql-proxy**: Port 5432 (service) → 5432 (container)
- **kafka**: Port 9092 (service) → 9092 (container)
- **redis-master**: Port 6379 (service) → 6379 (container)

## DNS/Service Communication

### All services are deployed in the `argocd` namespace

### Internal Service URLs:
- **Service Registry**: `http://mako-service-registry.argocd.svc.cluster.local:8761/eureka/`
- **API Gateway**: `http://mako-api-gateway.argocd.svc.cluster.local:8081`
- **Database**: `jdbc:postgresql://cloudsql-proxy.argocd.svc.cluster.local:5432/mako_bank`
- **Kafka**: `kafka.argocd.svc.cluster.local:9092`
- **Redis**: `redis-master.argocd.svc.cluster.local:6379`
- **Keycloak**: `http://keycloak.argocd.svc.cluster.local:80`

### External URLs (via Ingress):
- **Main Shell**: `https://mako-bank.com` → `mako-shell:80`
- **API Gateway**: `https://api.mako-bank.com` → `mako-api-gateway:8081`
- **Dashboard**: `https://dashboard.mako-bank.com` → `mako-dashboard-web:4201`
- **Accounts**: `https://accounts.mako-bank.com` → `mako-accounts-web:4202`
- **Transactions**: `https://transactions.mako-bank.com` → `mako-transactions-web:4203`
- **Users**: `https://users.mako-bank.com` → `mako-users-web:4204`
- **Keycloak**: `https://keycloak.mako-bank.com` → `keycloak:80`
- **ArgoCD**: `https://argocd.mako-bank.com` → `argocd-server:80`

## Communication Flow

### Frontend → Backend
1. **Shell App** proxies `/api/*` requests to **API Gateway**
2. **Individual Frontend Apps** communicate with backend via **API Gateway**
3. **API Gateway** routes requests to appropriate backend services

### Backend → Backend
1. **Service Discovery**: All services register with **Service Registry (Eureka)**
2. **Database Access**: All services connect via **CloudSQL Proxy**
3. **Messaging**: Services use **Kafka** for async communication
4. **Caching**: **Users Service** uses **Redis** for session management
5. **Authentication**: Services validate tokens with **Keycloak**

### Inter-Service Communication Examples:
- **Transactions Service** → **Accounts Service**: `http://mako-accounts.argocd.svc.cluster.local:8083/accounts`
- **Transactions Service** → **Users Service**: `http://mako-users.argocd.svc.cluster.local:8082/users`

## Security & HTTPS

### SSL/TLS Configuration:
- **Managed Certificates**: `mako-bank-ssl-cert` for all external domains
- **HTTP to HTTPS Redirect**: Configured via `mako-frontend-http-to-https` FrontendConfig
- **Static IP**: `mako-ingress-ip-global` for load balancer

### Internal Communication:
- All internal service-to-service communication is over HTTP (within cluster)
- External communication is HTTPS-only via Google Load Balancer

## Changes Made

### 1. Fixed Ingress Configuration:
- Corrected `mako-shell` port from `4200` to `80`

### 2. Fixed Service Discovery:
- Updated all Eureka client URLs to use `argocd` namespace
- Fixed service registry URL in all backend services

### 3. Fixed Database Connections:
- Updated CloudSQL proxy URLs to use `argocd` namespace
- Ensured consistent schema naming per service

### 4. Fixed Messaging:
- Updated Kafka broker URLs to use `argocd` namespace

### 5. Fixed Frontend Proxying:
- Updated nginx configuration in shell app to use correct namespaces
- Ensured all microfrontend proxying uses `argocd` namespace

### 6. Added Missing Annotations:
- Added backend config annotation to `mako-api-gateway` service

## Validation

To validate the configuration:

1. **Check service connectivity**:
   ```bash
   kubectl exec -n argocd deployment/mako-api-gateway -- curl http://mako-service-registry.argocd.svc.cluster.local:8761/eureka/apps
   ```

2. **Check database connectivity**:
   ```bash
   kubectl exec -n argocd deployment/mako-accounts -- pg_isready -h cloudsql-proxy.argocd.svc.cluster.local -p 5432
   ```

3. **Check external access**:
   ```bash
   curl -k https://mako-bank.com
   curl -k https://api.mako-bank.com/actuator/health
   ```

## Notes

- All services are now consistently using the `argocd` namespace
- Port configurations match between services, ingress, and deployments
- Service discovery URLs are corrected for proper Eureka registration
- Database connections are properly configured for each service schema
- Frontend routing is properly configured for microfrontend architecture
