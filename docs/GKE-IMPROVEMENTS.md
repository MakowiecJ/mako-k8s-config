# Mako Bank - GKE Best Practices & Configuration Guide

## 📋 Przegląd ulepszeń

Po przeanalizowaniu konfiguracji clustra na Google Cloud Platform, wprowadziłem następujące ulepszenia:

### 🔒 Bezpieczeństwo

#### 1. **Zarządzanie sekretami**
- ✅ Usunięcie hardkodowanych haseł z konfiguracji Keycloak
- ✅ Wykorzystanie Kubernetes Secrets dla credentials
- ✅ Implementacja Network Policies dla wszystkich namespace'ów

#### 2. **Network Policies**
- ✅ Zdefiniowanie zasad ruchu sieciowego między serwisami
- ✅ Ograniczenie dostępu tylko do niezbędnych portów
- ✅ Izolacja baz danych od niepotrzebnego ruchu

### 📊 Zarządzanie zasobami

#### 1. **Resource Quotas**
- ✅ Limity zasobów na poziomie namespace'ów
- ✅ Zapobieganie wyczerpaniu zasobów klastra
- ✅ Rozsądne podziały CPU/Memory między serwisami

#### 2. **Pod Disruption Budgets**
- ✅ Zapewnienie wysokiej dostępności podczas aktualizacji
- ✅ Minimalna liczba replik podczas maintenance

#### 3. **Horizontal Pod Autoscaler**
- ✅ Automatyczne skalowanie na podstawie CPU i Memory
- ✅ Inteligentne zasady skalowania w górę i w dół
- ✅ Stabilizacja podczas skalowania

### 🏗️ Infrastruktura

#### 1. **Kafka Configuration**
- ✅ Przejście na KRaft mode (bez Zookeeper)
- ✅ Optymalizacja resource limits
- ✅ Persistent storage z odpowiednimi rozmiarami

#### 2. **Health Checks**
- ✅ Proper liveness i readiness probes
- ✅ Spring Boot Actuator endpoints
- ✅ Timeout i failure threshold configuration

### 📈 Monitorowanie

#### 1. **Observability**
- ✅ Konfiguracja dla Prometheus metrics
- ✅ Centralna konfiguracja logowania
- ✅ Service Monitor dla automatycznego discovery

## 🚀 Deployment Guide

### 1. **Zastosowanie ulepszeń**

```bash
# Zastosuj wszystkie ulepszenia
kubectl apply -f apps/0-bootstrap/resource-management.yaml
kubectl apply -f apps/0-bootstrap/network-policies.yaml

# Sprawdź status aplikacji ArgoCD
kubectl get applications -n argocd
```

### 2. **Veryfikacja Resource Quotas**

```bash
# Sprawdź resource quotas
kubectl describe resourcequota -n mako-services
kubectl describe resourcequota -n mako-frontend
kubectl describe resourcequota -n keycloak
kubectl describe resourcequota -n kafka
```

### 3. **Sprawdzenie Network Policies**

```bash
# Lista wszystkich network policies
kubectl get networkpolicies --all-namespaces

# Szczegóły policy dla backend services
kubectl describe networkpolicy mako-backend-services-network-policy -n mako-services
```

### 4. **Monitoring HPA**

```bash
# Status autoscalerów
kubectl get hpa -n mako-services

# Szczegóły skalowania
kubectl describe hpa mako-api-gateway-hpa -n mako-services
```

## ⚠️ Zalecenia dodatkowe

### 1. **Secrety wymagające utworzenia**

```bash
# Keycloak admin credentials
kubectl create secret generic keycloak-admin-credentials \
  --from-literal=admin-password=STRONG_PASSWORD \
  -n keycloak

# Namespace labels dla Network Policies
kubectl label namespace mako-services name=mako-services
kubectl label namespace mako-frontend name=mako-frontend
kubectl label namespace keycloak name=keycloak
kubectl label namespace mako-db name=mako-db
kubectl label namespace kafka name=kafka
```

### 2. **Monitoring setup**

Jeśli chcesz skonfigurować monitoring:

```bash
# Zainstaluj Prometheus Operator
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# Zastosuj monitoring config
kubectl apply -f kustomize/monitoring/
```

### 3. **Backup strategy**

```bash
# Backup ETCD i persistent volumes
# Skonfiguruj automatyczne backup'y w GCP Console
```

## 🔧 Troubleshooting

### 1. **Resource limit exceeded**

```bash
kubectl describe pod <POD_NAME> -n <NAMESPACE>
# Sprawdź events na końcu output'u
```

### 2. **Network policy debugging**

```bash
# Test conectivity między podami
kubectl run debug-pod --image=busybox --rm -it -- sh
# wewnątrz poda:
nslookup <SERVICE_NAME>.<NAMESPACE>.svc.cluster.local
```

### 3. **HPA nie skaluje**

```bash
kubectl describe hpa <HPA_NAME> -n <NAMESPACE>
# Sprawdź metryki i conditions
```

## 📈 Następne kroki

1. **Implementacja service mesh** (Istio/Linkerd) dla advanced networking
2. **Chaos engineering** testing z Chaos Monkey
3. **Advanced monitoring** z custom metrics
4. **GitOps workflows** z ArgoCD Image Updater
5. **Cost optimization** analiza z GCP Cost Management

## 📚 Dokumentacja referencji

- [Kubernetes Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [GKE Best Practices](https://cloud.google.com/kubernetes-engine/docs/best-practices)
