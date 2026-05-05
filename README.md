# 🧁 CupCat Café — Helm Chart Lab

> *Where pixel cats serve cupcakes, powered by Kubernetes!*

![Helm](https://img.shields.io/badge/Helm-3.x-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.32-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS EKS](https://img.shields.io/badge/AWS%20EKS-us--east--1-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-latest-009639?style=for-the-badge&logo=nginx&logoColor=white)

---

## 📖 About This Lab

This lab walks through the full lifecycle of packaging and deploying a web application on **Kubernetes using Helm** — from writing templates to publishing a public Helm repository via GitHub Pages.

The app itself is a fun, animated **CupCat Café** landing page served by Nginx, but the real learning is in the tooling around it. 🐱

---

## 🏗️ What Was Built

```
cupcatcafe/
├── Chart.yaml           # Chart metadata (name, version, description)
├── values.yaml          # All configurable values (replicas, image, cafeName...)
└── templates/
    ├── deployment.yaml  # Kubernetes Deployment
    ├── service.yaml     # Kubernetes Service (LoadBalancer)
    └── configmap.yaml   # HTML page served by Nginx
```

---

## 🧠 Key Concepts Covered

### 🎯 Helm = The Terraform of Kubernetes

| | Terraform | Helm |
|---|---|---|
| Manages | AWS Infrastructure | Apps inside K8s |
| Config file | `variables.tf` | `values.yaml` |
| Templates | `main.tf` | `templates/` |
| Install | `terraform apply` | `helm install` |
| Destroy | `terraform destroy` | `helm uninstall` |

> Both follow the same philosophy: **define what you want, let the tool handle the rest.**

---

### 📦 Helm Chart Structure

A Helm Chart is a **package** containing everything needed to deploy an app:

- **`Chart.yaml`** → Who: name and version of the chart
- **`values.yaml`** → What: all the configurable settings
- **`templates/`** → How: the Kubernetes resource definitions

```yaml
# values.yaml — change these, everything updates automatically!
replicaCount: 1
cafeName: "CupCat Café"
image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent
```

---

### 🔧 Helm Templates — Dynamic YAML

Templates use Go templating to inject values from `values.yaml`:

```yaml
# templates/deployment.yaml
metadata:
  name: {{ include "cupcatcafe.fullname" . }}
  labels:
    {{- include "cupcatcafe.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  containers:
    - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

The `{{ }}` blocks are replaced with actual values at install time. 🪄

---

### 📬 ConfigMap — Separating Config from Code

The HTML page is stored in a **ConfigMap** and mounted into the Nginx container as a volume:

```yaml
volumes:
  - name: html
    configMap:
      name: {{ include "cupcatcafe.fullname" . }}-html
```

This means you can update the page content **without rebuilding the Docker image**!

---

## 🚀 How to Use This Chart

### Install from the public Helm repository

```bash
# Add the repo
helm repo add cupcatcafe https://joaodmorgadoribeiro-del.github.io/cupcatcafe_Helm_Charts

# Update
helm repo update

# Install
helm install my-cafe cupcatcafe/cupcatcafe --namespace my-namespace --create-namespace
```

### Or install from source

```bash
git clone https://github.com/joaodmorgadoribeiro-del/cupcatcafe_Helm_Charts
helm install my-cafe ./cupcatcafe --namespace my-namespace --create-namespace
```

---

## ⚙️ Deployment on AWS EKS

This chart was deployed on a **Spot Instance EKS cluster** (`spot-eks-lab-joao`) in `us-east-1`.

```bash
# Connect kubectl to EKS
aws eks update-kubeconfig --region us-east-1 --name spot-eks-lab-joao

# Create namespace
kubectl create namespace joao

# Deploy!
helm install my-cafe cupcatcafe/cupcatcafe --namespace joao

# Check pods
kubectl get pods -n joao

# Get external URL
kubectl get svc -n joao
```

---

## 📚 Helm Commands Cheatsheet

```bash
# Package the chart
helm package ./cupcatcafe

# Generate repo index
helm repo index . --url https://joaodmorgadoribeiro-del.github.io/cupcatcafe_Helm_Charts

# Dry run (test before installing)
helm install my-cafe cupcatcafe/cupcatcafe --dry-run

# Upgrade
helm upgrade my-cafe cupcatcafe/cupcatcafe --namespace joao

# List all releases
helm list -A

# Uninstall
helm uninstall my-cafe --namespace joao
```

---

## 💡 Lessons Learned

- ✅ `values.yaml` is not just replica count — it's **all** configurable settings
- ✅ Helm templates use `{{ }}` to inject values dynamically
- ✅ `---` separator allows multiple K8s resources in one YAML file
- ✅ ConfigMaps decouple app config from container images
- ✅ GitHub Pages can host a public Helm repository for free
- ✅ Always `helm lint .` before packaging to catch YAML errors
- ⚠️ `t3.small` nodes fill up fast — scale up when hitting `Too many pods`!

---

## 🏛️ Architecture

```
GitHub Pages
└── Helm Repository (index.yaml + cupcatcafe-0.1.0.tgz)
        ↓
    helm install
        ↓
AWS EKS Cluster (spot-eks-lab-joao)
└── Namespace: joao
    ├── Deployment (1 replica, Nginx)
    │   └── Pod
    │       └── Container (nginx)
    │           └── /usr/share/nginx/html ← mounted from ConfigMap
    ├── Service (LoadBalancer)
    └── ConfigMap (HTML page)
```

---

## 👨‍💻 Author

**João Ribeiro** — Ironhack Cloud & DevOps Bootcamp

[![GitHub](https://img.shields.io/badge/GitHub-joaodmorgadoribeiro--del-181717?style=flat&logo=github)](https://github.com/joaodmorgadoribeiro-del)
[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-joaoribeiro9595-2496ED?style=flat&logo=docker&logoColor=white)](https://hub.docker.com/u/joaoribeiro9595)

---

*Built with ☕, 🐱, and a lot of `kubectl get pods`*
