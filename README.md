# Prometheus + Grafana on Minikube

Sets up a **Kubernetes observability stack** locally using:
- Minikube
- Prometheus
- Grafana
- Helm

## Tools Used
- [Minikube](https://minikube.sigs.k8s.io/) (Docker driver)
- [Helm](https://helm.sh/)
- [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)

---

## What It Does
- Spins up a local Kubernetes cluster
- Deploys the full monitoring stack via Helm
- Port-forwards Grafana to `localhost:3000`
- Optional: Provision Grafana dashboards from ConfigMap

---

## Setup Instructions

### 1. Start Minikube
```bash
minikube start --driver=docker
```

### 2. Add Helm Repo
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 3. Install the Prometheus Stack
```bash
helm install prom-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

### 4. Port-forward Grafana
```bash
kubectl port-forward -n monitoring svc/prom-stack-grafana 3000:80
```
Then visit: [http://localhost:3000](http://localhost:3000)

### 5. Grafana Login
```bash
# Username:
admin

# Password:
kubectl get secret --namespace monitoring prom-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode
```

---

## Dashboard Provisioning (Optional)
If you want to automatically load a dashboard:

1. Export it from the Grafana UI (gear → JSON model)
2. Save it to `grafana-dashboards/your-dashboard.json`
3. Create a ConfigMap:

```bash
kubectl create configmap my-dashboard \
  --from-file=grafana-dashboards/your-dashboard.json \
  -n monitoring --dry-run=client -o yaml > manifests/grafana-dashboard-cm.yaml
```

4. Add label to the YAML:
```yaml
metadata:
  labels:
    grafana_dashboard: "1"
```

5. Apply:
```bash
kubectl apply -f manifests/grafana-dashboard-cm.yaml
```
--

## 🔹 Project Structure
```
minikube-observability/
├── README.md
├── helm-values/          
├── grafana-dashboards/          
├── manifests/                   # Kubernetes YAMLs
└── scripts/  
