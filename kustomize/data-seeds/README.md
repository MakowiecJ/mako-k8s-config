# Mako Bank - GKE Test Data Initialization

This directory contains Kubernetes Job definitions for initializing test data in your Google Kubernetes Engine (GKE) deployment, similar to the `db-seeds` directory used in local Docker Compose setup.

## Overview

The data seeding process creates:
- **Test Users** in PostgreSQL databases
- **Sample Accounts** with realistic bank account data
- **Transaction History** with various transaction types
- **Notifications** for user interactions
- **Keycloak Realm** with test users for authentication

## Components

### Database Seed Jobs
- `users-seed-job.yaml` - Seeds the users database
- `accounts-seed-job.yaml` - Seeds the accounts database  
- `transactions-seed-job.yaml` - Seeds the transactions database
- `notifications-seed-job.yaml` - Seeds the notifications database

### Keycloak Configuration
- `../keycloak/realm-config.yaml` - Keycloak realm with test users

### ArgoCD Management
- `../../apps/3-data-seeds/data-seeds.yaml` - ArgoCD application for managing seed jobs

## Usage

### Quick Deployment
```powershell
# Deploy all test data automatically
.\deploy-gke-test-data.ps1

# Check status
.\manage-gke-data.ps1 -Action status
```

### Manual Deployment
```bash
# Apply the ArgoCD application
kubectl apply -f ../../apps/3-data-seeds/data-seeds.yaml

# Or apply individual jobs
kubectl apply -f users-seed-job.yaml
kubectl apply -f accounts-seed-job.yaml
kubectl apply -f transactions-seed-job.yaml
kubectl apply -f notifications-seed-job.yaml
```

### Monitoring
```bash
# Check job status
kubectl get jobs -n mako-services

# View job logs
kubectl logs -l job-name=users-db-seed -n mako-services

# Check ArgoCD application
kubectl get application mako-data-seeds -n argocd
```

## Test Data

### Users
| Username | Password | Email | Keycloak ID |
|----------|----------|-------|-------------|
| john | password | john.walter@email.com | cdd4e82f-1ceb-4600-908f-f435df3672bd |
| jane | password | jane.smith@email.com | a5ebbd33-0ef4-4203-abe1-fddc35762b04 |
| bob | password | bob.johnson@email.com | 0edfcbff-b61a-4530-9618-c9f309594b29 |

### Accounts
- **John:** Checking ($5,000) and Savings ($15,000)
- **Jane:** Checking ($3,200)
- **Bob:** Checking ($7,500)

### Transactions
- Salary deposits, grocery shopping, transfers between accounts
- Various transaction types: DEPOSIT, WITHDRAWAL, TRANSFER

## Architecture

### Job Flow
1. **ArgoCD** detects changes and creates Kubernetes Jobs
2. **Jobs** wait for PostgreSQL databases to be ready
3. **PostgreSQL containers** execute SQL seed scripts
4. **Jobs complete** and report success/failure status

### Dependencies
- PostgreSQL databases must be running
- Database secrets must exist
- Services must be accessible via Kubernetes DNS

### Security
- Database passwords are read from Kubernetes secrets
- Jobs run with minimal privileges
- SQL scripts use `ON CONFLICT DO NOTHING` for idempotency

## Troubleshooting

### Job Failed
```bash
# Check job status
kubectl describe job users-db-seed -n mako-services

# View pod logs
kubectl logs -l job-name=users-db-seed -n mako-services
```

### Database Connection Issues
```bash
# Test database connectivity
kubectl run debug-pod --image=postgres:16 --rm -it -- bash
pg_isready -h postgres-users-postgresql.mako-services.svc.cluster.local -p 5432
```

### Reset and Retry
```bash
# Delete failed jobs
kubectl delete jobs -l app.kubernetes.io/name=mako-data-seeds -n mako-services

# Reapply
kubectl apply -f .
```

## Comparison with Local Setup

| Local (Docker Compose) | GKE (Kubernetes) |
|------------------------|------------------|
| `db-seeds/*.sql` | Kubernetes Jobs with embedded SQL |
| `docker-compose.yml` volumes | ConfigMaps and Secrets |
| `keycloak-config/` | Kubernetes ConfigMap |
| Manual `docker-compose up` | ArgoCD automated deployment |
| Direct database access | Jobs with `pg_isready` waiting |

## Next Steps

After successful data seeding:
1. **Test Authentication:** Login with test users (john/password, jane/password, bob/password)
2. **Verify Data:** Check that accounts and transactions appear in the UI
3. **Test Transactions:** Try creating new transactions between accounts
4. **Monitor System:** Use ArgoCD and Kubernetes dashboards to monitor health
