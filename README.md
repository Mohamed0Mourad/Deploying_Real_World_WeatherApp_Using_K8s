
# 🌍 Deploying Real-World WeatherApp Using Kubernetes ☁️🚀

Welcome to the **WeatherApp Kubernetes Lab Project** — a complete end-to-end microservices application deployed on **Kubernetes (KinD)**
> 💡 **Goal**: Learn how to deploy, scale, secure, and expose real microservices using Kubernetes best practices.

---

## 📦 Project Architecture

This project contains **three core microservices**, fully containerized and deployed on a Kubernetes cluster:

```
                ┌──────────────┐
                │  UI (Python) │
                └──────┬───────┘
                       │
         ┌─────────────┼─────────────┐
         │                             │
┌────────▼────────┐         ┌─────────▼─────────┐
│ Auth Service    │         │ Weather Service   │
│ (Go + MySQL)    │         │ (Node.js + API)   │
└─────────────────┘         └───────────────────┘
```

---

## 🔧 Services Breakdown

### 1️⃣ 🛡️ Authentication Service (GoLang + MySQL)

This service manages user authentication and JWT token generation.

#### 📌 Components:
- **MySQL StatefulSet** with:
  - A **Headless Service** for stable DNS
  - A **Secret** for DB credentials + JWT secret key
  - An **Init Job** to bootstrap the `weatherapp` DB and user
- **GoLang Auth API**:
  - Deployment with 1 replica
  - ClusterIP service on port `8080`

---

### 2️⃣ 🌤️ Weather API Service (Node.js)

A lightweight Node.js service that connects to [RapidAPI](https://rapidapi.com) to fetch real-time weather data.

#### 📌 Components:
- Deployment + Secret (for API key)
- ClusterIP service on port `5000`

---

### 3️⃣ 🖥️ UI Web Application (Python Flask)

Frontend interface that interacts with both Auth and Weather services.

#### 📌 Components:
- Deployment with **2 replicas** (for availability)
- ClusterIP service on port `3000`
- Ingress resource to expose HTTP/HTTPS
- TLS secured using self-signed certificate via OpenSSL

---

## 🧪 Step-by-Step Deployment Guide

### 🔁 Cluster Setup (Clean Start)

If needed, reset your cluster by following the instructions in the [Ingress Lab (Lab 3)](https://github.com/Mohamed0Mourad/Kubernetes-Course-Practical-Labs/tree/main/Lab3).

---

### 📂 MySQL Deployment

#### ✅ Headless Service

```yaml
clusterIP: None
```

#### ✅ Secret Creation

```bash
kubectl create secret generic mysql-secret \
  --from-literal=root-password='secure-root-pw' \
  --from-literal=auth-password='my-secret-pw' \
  --from-literal=secret-key='xco0sr0fh4e52x03g9mv'
```

#### ✅ Init Job (DB Bootstrap)

Creates the `weatherapp` database and user with permissions.

#### ✅ StatefulSet

```yaml
replicas: 1
```

#### ✅ Apply Resources

```bash
kubectl apply -f headless-service.yaml
kubectl apply -f statefulset.yaml
kubectl apply -f init-job.yaml
```

#### ✅ MySQL Validation

```bash
kubectl exec -it mysql-0 -- mysql -u authuser -pmy-secret-pw
SHOW DATABASES;
USE weatherapp;
```

---

### 🔐 Authentication Service

#### ✅ Deployment + Service

Exposes port `8080` inside the cluster.

#### ✅ Test the Auth API

```bash
kubectl run alpine --rm -it --image=alpine -- sh
apk add curl
curl -X POST http://weatherapp-auth:8080/users \
-H "Content-Type: application/json" \
-d '{"username": "testuser", "password": "testpassword"}'
# Response: {"success":"User added successfully"}
```

---

### 🌤️ Weather Service

#### ✅ Deployment + Secret

Store your RapidAPI key securely in a Kubernetes Secret.

#### ✅ Service

Exposed on port `5000` (ClusterIP).

#### ✅ Test Weather API

```bash
curl weatherapp-weather:5000
# Output: "The service is running"

curl weatherapp-weather:5000/cairo
# Output: JSON with current weather for Cairo
```

---

### 🖥️ UI Web App

#### ✅ Deployment + Service

Expose internally on port `3000`, using 2 replicas for availability.

#### ✅ Create TLS Certificate (HTTPS)

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout tls.key -out tls.crt -subj "/CN=weatherapp.local/O=weatherapp"
```

#### ✅ Create Secret for TLS

```bash
kubectl create secret tls weatherapp-tls \
--cert=tls.crt --key=tls.key
```

#### ✅ Ingress Object

Configure Ingress for both HTTP and HTTPS routing:

> 📌 Make sure to edit your `/etc/hosts`:
```bash
127.0.0.1 weatherapp.local
```

```bash
kubectl apply -f ingress.yaml
```

---

## 🔍 Verifying Everything Works

✅ UI accessible at: https://weatherapp.local  
✅ User Registration & Login works via Auth Service  
✅ Weather info loads dynamically from Node.js API  
✅ All services discover each other via internal DNS

---

## 🧰 Tools & Technologies

- Kubernetes (KinD)
- MySQL (StatefulSet, PVC)
- GoLang (Auth API)
- Node.js (Weather Service)
- Python Flask (UI App)
- NGINX Ingress Controller
- OpenSSL for TLS
- RapidAPI (Weather data)

--
