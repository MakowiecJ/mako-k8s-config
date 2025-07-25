# 💰 Mako Bank - Ultra Cost-Efficient Configuration

## 🎯 Cel: Maksymalna oszczędność kosztów GKE

Ta konfiguracja została zoptymalizowana pod kątem **minimalnych kosztów** kosztem niektórych aspektów wydajności i redundancji.

## 📊 Oszczędności kosztów

### ✅ **Przed optymalizacją:**
- **Frontend:** 5 deploymentów × 64Mi RAM = 320Mi + 5 × 100m CPU = 500m CPU
- **Backend:** 6 deploymentów × 256Mi RAM = 1.5Gi + 6 × 200m CPU = 1200m CPU
- **Bazy danych:** 2 PostgreSQL instancje
- **Kafka:** Pełny cluster z Zookeeper
- **Łącznie:** ~2Gi RAM, ~1.7 CPU cores

### ✅ **Po optymalizacji:**
- **Frontend:** 1 deployment × 256Mi RAM = 256Mi + 300m CPU = 300m CPU
- **Backend:** 6 deploymentów × 128Mi RAM = 768Mi + 6 × 100m CPU = 600m CPU  
- **Bazy danych:** 1 PostgreSQL instancja (512Mi RAM)
- **Kafka:** Minimal single-node (512Mi RAM)
- **Łącznie:** ~1.5Gi RAM, ~1 CPU core

### 💰 **Szacowane oszczędności: ~40-50% kosztów compute**

## 🏗️ Główne zmiany architektury

### 1. **Konsolidacja Frontend'u**
```yaml
# Zamiast 5 osobnych deploymentów:
# ❌ mako-shell, mako-dashboard-web, mako-accounts-web, mako-transactions-web, mako-users-web

# ✅ Jeden deployment z wieloma kontenerami:
mako-frontend-consolidated:
  containers: [shell, dashboard, accounts, transactions, users]
```

### 2. **Pojedyncza baza danych**
```yaml
# Zamiast:
# ❌ postgres-keycloak + cloudsql-proxy

# ✅ Jeden PostgreSQL z wieloma schematami:
postgres-consolidated:
  schemas: [accounts, transactions, users, notifications, keycloak]
```

### 3. **Minimal Kafka**
```yaml
# Zamiast pełnego cluster'a:
# ❌ 3 brokers + 3 zookeepers

# ✅ Single-node KRaft:
kafka-minimal:
  controller: 1 node
  broker: 0 nodes
  zookeeper: disabled
```

### 4. **Agresywne HPA**
```yaml
# Skalowanie do 0 replik gdy brak ruchu
minReplicas: 0
maxReplicas: 1-2
cpuUtilization: 90-95%  # Bardzo wysokie progi
```

## 🚀 Deployment Ultra-oszczędnej wersji

### 1. **Zastąp obecne aplikacje:**
```bash
# Usuń obecne frontend deployments
kubectl delete application mako-shell -n argocd
kubectl delete application mako-dashboard-web -n argocd
kubectl delete application mako-accounts-web -n argocd
kubectl delete application mako-transactions-web -n argocd
kubectl delete application mako-users-web -n argocd

# Zastąp skonsolidowaną wersją
kubectl apply -f apps/2-services/frontend-consolidated.yaml
```

### 2. **Zastosuj oszczędne resource limits:**
```bash
# Zastosuj ultra-low resource patches
kubectl patch deployment mako-accounts -n mako-services --patch-file optimization-patches/ultra-low-cost-optimization.yaml
kubectl patch deployment mako-api-gateway -n mako-services --patch-file optimization-patches/ultra-low-cost-optimization.yaml
# Powtórz dla wszystkich backend services
```

### 3. **Konsolidacja baz danych:**
```bash
# Backup danych z obecnych baz
kubectl exec -it postgres-keycloak-postgresql-0 -n keycloak -- pg_dump keycloak > keycloak-backup.sql

# Zastąp skonsolidowaną bazą
kubectl apply -f optimization-patches/postgres-consolidation.yaml

# Restore danych do nowej bazy z odpowiednim schematem
```

### 4. **Zminimalizuj Kafka:**
```bash
# Zastąp obecną instalację Kafka
kubectl delete application kafka -n argocd
kubectl apply -f optimization-patches/kafka-cost-optimization.yaml
```

## ⚠️ **Kompromisy i ograniczenia**

### 🔴 **Wydajność:**
- **Longer startup times** - mniej zasobów = wolniejsze uruchamianie
- **Potential OOM kills** - bardzo niskie memory limits
- **CPU throttling** - przy wysokim obciążeniu

### 🔴 **Dostępność:**
- **Single points of failure** - brak redundancji
- **No rolling updates** - tylko 1 replika frontend
- **Database bottleneck** - jedna instancja dla wszystkich serwisów

### 🔴 **Skalowanie:**
- **Manual intervention needed** - HPA z minReplicas=0 wymaga ręcznego startu
- **Slow scale-up** - długie okresy stabilizacji
- **Resource contention** - współdzielone zasoby

## 🔧 **Monitorowanie oszczędnej wersji**

### 1. **Sprawdź resource usage:**
```bash
# Aktualnie użyte zasoby
kubectl top nodes
kubectl top pods --all-namespaces

# Resource requests vs limits
kubectl describe resourcequota -A
```

### 2. **Monitoring kosztów:**
```bash
# GCP Cost monitoring
gcloud billing budgets list
gcloud compute instances list --format="table(name,zone,machineType,status)"
```

### 3. **Sprawdź czy serwisy działają:**
```bash
# Health check wszystkich serwisów
kubectl get pods -A | grep -v Running
kubectl get hpa -A
```

## 📈 **Dalsze optymalizacje**

### 1. **Spot/Preemptible nodes:**
```bash
# Użyj tanich spot instances dla GKE nodes
gcloud container node-pools create spot-pool \
  --cluster=mako-cluster \
  --spot \
  --machine-type=e2-small
```

### 2. **Scheduled scaling:**
```bash
# CronJob do skalowania w dół w nocy
apiVersion: batch/v1
kind: CronJob
metadata:
  name: scale-down-nightly
spec:
  schedule: "0 2 * * *"  # 2 AM scale down
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: kubectl
            command:
            - kubectl
            - scale
            - deployment
            - --replicas=0
            - --all
            - -n
            - mako-services
```

### 3. **Cloud SQL zamiast self-hosted PostgreSQL:**
```yaml
# Rozważ przejście na managed Cloud SQL
# Może być tańsze dla małych aplikacji (no maintenance overhead)
```

## 🎯 **Oczekiwane rezultaty**

- **💰 Koszty:** Spadek o 40-50%
- **⚡ Startup:** +30-60 sekund dłużej
- **🔄 Scaling:** Wolniejsze reagowanie na ruch
- **🛡️ Reliability:** Niższa dostępność

**Idealne dla:** development, staging, demo environments  
**Nie zalecane dla:** production z wysokim ruchem
