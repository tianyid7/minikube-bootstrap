# Minikube Bootstrap
This repository contains a set of steps to bootstrap a minikube cluster with a set of services.

## Prerequisites
- [Minikube v1.35.0](https://minikube.sigs.k8s.io/docs/start/)
- [Docker Desktop (Mem >= 16G CPU >= 8)](https://docs.docker.com/desktop/)
- [Helm](https://helm.sh/)
- [kubectx](https://github.com/ahmetb/kubectx)
- [k9s](https://k9scli.io/)

## Getting Started
1. Start minikube
```bash
# Start minikube with docker driver. Ideally, memory should be >= 16G and CPU should be >= 8.
minikube start --driver=docker --cpus=8 --memory=16g
```
The output should look like this:
```bash
😄  minikube v1.35.0 on Darwin 15.3.2 (arm64)
✨  Using the docker driver based on user configuration
📌  Using Docker Desktop driver with root privileges
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.46 ...
🔥  Creating docker container (CPUs=8, Memory=13312MB) ...
💾  Downloading Kubernetes v1.32.0 preload ...
    > preloaded-images-k8s-v18-v1...:  314.92 MiB / 314.92 MiB  100.00% 11.11 M
    > gcr.io/k8s-minikube/kicbase...:  452.84 MiB / 452.84 MiB  100.00% 8.18 Mi
🐳  Preparing Kubernetes v1.32.0 on Docker 27.4.1 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Check if the context is switched to `minikube` by running the following command:
```bash
>> kubectx minikube
```

2. Installing Ray Helm Chart (https://github.com/ray-project/kuberay/blob/master/helm-chart/ray-cluster/README.md)
```bash
# Step 1: Register a Helm chart repo
helm repo add kuberay https://ray-project.github.io/kuberay-helm/

# Step 2: Install both CRDs and KubeRay operator v1.1.0.
helm install kuberay-operator kuberay/kuberay-operator --version 1.1.0

# Step 3: Install a RayCluster custom resource
# (For x86_64 users)
helm install raycluster kuberay/ray-cluster --version 1.1.0
# (For arm64 users, e.g. Mac M1)
# See here for all available arm64 images: https://hub.docker.com/r/rayproject/ray/tags?page=1&name=aarch64
helm install raycluster kuberay/ray-cluster --version 1.1.0 --set image.tag=nightly-aarch64

# Step 4: Verify the installation of KubeRay operator and RayCluster
kubectl get pods
# NAME                                          READY   STATUS    RESTARTS   AGE
# kuberay-operator-6fcbb94f64-gkpc9             1/1     Running   0          89s
# raycluster-kuberay-head-qp9f4                 1/1     Running   0          66s
# raycluster-kuberay-worker-workergroup-2jckt   1/1     Running   0          66s

# Step 5: Forward the port of Dashboard
kubectl port-forward svc/raycluster-kuberay-head-svc 8265:8265

# Step 6: Check 127.0.0.1:8265 for the Dashboard

# Step 7: Uninstall RayCluster
# helm uninstall raycluster
```

3. Installing Onyx Helm Chart (https://github.com/onyx-dot-app/onyx/tree/main/deployment/helm/charts/onyx)
```bash
# Step1: Installing packaged Onyx Helm Chart (It'll take about 10-15 mins to finish the installation)
helm install onyx-stack onyx-stack-0.2.1.tgz

# Step2: Verify the installation of Onyx stack
kubectl get pods

# Step 3: Minikube assign an external IP to the Onyx Nginx. This will prompt a browser to open the URL
minikube service onyx-stack-nginx

# Step 4: Uninstall Onyx
# helm uninstall onyx-stack
```

> How to package Onyx Helm chart?
>```bash
># Step 1: Clone the Onyx repository
>git clone https://github.com/onyx-dot-app/onyx.git
>
># Step 2: Change directory to the Onyx deployments
>cd onyx/deployment
>
># Step 3: Package the Helm chart
>helm package helm/charts/onyx -u
>```

4. Installing Postgres Helm Chart
```bash
# Step 1: Add the Bitnami Helm chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Step 2: Install the Postgres with persistence storage
helm install postgres bitnami/postgresql --set auth.postgresPassword=postgrespassword,auth.username=postgres,auth.password=postgres,primary.persistence.enabled=true,replica.persistence.enabled=false

# Step 3: Verify the installation of Postgres
kubectl get pods

# Step 4: Port forward the Postgres service to access it locally
kubectl port-forward svc/postgres-postgresql 5432:5432

# Step 5: Connect to the Postgres database: postgres:postgres@localhost:5432/postgres

# Step 6: Uninstall Postgres
# helm uninstall postgres
```