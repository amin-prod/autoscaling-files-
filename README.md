# Kubernetes HPA Autoscaling with Minikube

This repository demonstrates **Horizontal Pod Autoscaling (HPA)** in a local Kubernetes environment using **Minikube**, **Nginx**, **metrics-server**, and **Grafana**.

The goal is to show how Kubernetes automatically scales pods based on **CPU utilization**, and how this behavior can be **visualized using Grafana**.

---

## Prerequisites

Make sure the following tools are installed:

* **Docker Desktop** (with WSL2 backend enabled)
* **WSL2 + Ubuntu**
* **kubectl** (Kubernetes CLI)
* **Minikube**
* Grafana

---

## Project Files

```
.
├── deployment.yaml   # Nginx Deployment
├── service.yaml      # ClusterIP Service
├── hpa.yaml          # Horizontal Pod Autoscaler
└── README.md
```

---

## Step 1 — Start Minikube

```bash
minikube start --driver=docker
```

**Explanation:**

* `minikube start` → starts a local Kubernetes cluster
* `--driver=docker` → uses Docker as the container runtime

Verify the cluster:

```bash
kubectl get nodes
```

---

## Step 2 — Enable Metrics Server (Required for HPA)

```bash
minikube addons enable metrics-server
```

Verify metrics collection:

```bash
kubectl top pods
```

If this command works, HPA will be able to scale pods.

---

## Step 3 — Deploy the Application (Nginx)

Apply the Deployment:

```bash
kubectl apply -f deployment.yaml
```

Check running pods:

```bash
kubectl get pods
```

---

## Step 4 — Create the Service

```bash
kubectl apply -f service.yaml
```

Verify the service:

```bash
kubectl get svc
```

The service allows internal access to the Nginx pods.

---

## Step 5 — Create the Horizontal Pod Autoscaler (HPA)

```bash
kubectl apply -f hpa.yaml
```

Check HPA status:

```bash
kubectl get hpa
```

Initial state:

* Replicas: `1`
* CPU usage: low

---

## Step 6 — Generate Load (Trigger Autoscaling)

### 6.1 Create a Load Generator Pod

```bash
kubectl run load-generator \
  --image=busybox \
  --restart=Never \
  -- /bin/sh
```

### 6.2 Send Continuous Requests

Inside the pod shell:

```sh
while true; do wget -q -O- http://demo-app; done
```

This loop continuously sends HTTP requests to the Nginx service, increasing CPU usage.

---

## Step 7 — Observe Autoscaling

Watch HPA behavior in real time:

```bash
kubectl get hpa -w
```

Check pod scaling:

```bash
kubectl get pods
```

Expected behavior:

* CPU usage exceeds 10%
* Number of pods increases automatically

---

## Step 8 — Enable and Open Grafana

Port-forward Grafana :
```bash
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
```

open link -> http://localhost:3000/

## Step 9 — Grafana Login Credentials

### Default Credentials

```
Username: admin
Password: admin
```

### If Default Password Does Not Work

Retrieve the password from Kubernetes secrets:

```bash
kubectl get secret grafana -n kube-system -o jsonpath="{.data.admin-password}" | base64 --decode
```

Username remains:

```
admin
```

---

## Step 10 — Visualize Autoscaling in Grafana

In Grafana dashboards:

* Open **Kubernetes / Compute Resources / Pod**
* Monitor:

  * CPU usage per pod
  * Number of replicas
  * Scaling events over time

You should clearly observe:

* CPU spikes
* Pod scaling up
* Stabilization under load

---

## Step 11 — Stop Load and Observe Scale Down

Exit the load generator pod:

```sh
exit
```

Delete the pod:

```bash
kubectl delete pod load-generator
```

Watch HPA scale down:

```bash
kubectl get hpa -w
```

Pods will gradually return to the minimum replica count.

---

## Conclusion

This project demonstrates:

* Kubernetes Deployment and Service
* Metrics-based autoscaling using HPA
* Real-time monitoring with Grafana
* Automatic scale-up and scale-down behavior

This setup is ideal for learning and validating Kubernetes autoscaling concepts in a local environment.

---

## Author

Amin El Mabrouk
