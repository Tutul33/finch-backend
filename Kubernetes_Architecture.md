# Kubernetes Architecture Documentation

## Overview

This document explains the Kubernetes deployment setup for the **Finch** project, including:

- Deployments and services for frontend, backend, PostgreSQL, and Redis.
- Secret management.
- Ingress configuration.
- Persistent storage.
- Scaling and resource allocation notes.

---

## Namespace

All resources are deployed under the namespace:  
`capstone`

---

## Frontend Deployment and Service

**File:** `frontend-deployment.yaml`

- Deploys the `finch-frontend` container (image: `tutulchakma/finch-frontend:latest`).
- Runs 1 replica, exposing port 80.
- Service of type `NodePort` exposes it on node port `30080`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: finch-frontend
  namespace: capstone
spec:
  replicas: 1
  selector:
    matchLabels:
      app: finch-frontend
  template:
    metadata:
      labels:
        app: finch-frontend
    spec:
      containers:
        - name: finch-frontend
          image: tutulchakma/finch-frontend:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: finch-frontend
  namespace: capstone
spec:
  type: NodePort
  selector:
    app: finch-frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

---

## Backend Deployment and Service

**File:** `backend-deployment.yaml`

- Deploys the `finch-backend` container (image: `tutulchakma/finch-backend:latest`).
- Runs 1 replica, exposing port 8000.
- Uses environment variables from `django-env-secret`.
- Service of type `ClusterIP` exposes backend internally.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: finch-backend
  namespace: capstone
spec:
  replicas: 1
  selector:
    matchLabels:
      app: finch-backend
  template:
    metadata:
      labels:
        app: finch-backend
    spec:
      containers:
        - name: finch-backend
          image: tutulchakma/finch-backend:latest
          ports:
            - containerPort: 8000
          envFrom:
            - secretRef:
                name: django-env-secret
---
apiVersion: v1
kind: Service
metadata:
  name: finch-backend
  namespace: capstone
spec:
  type: ClusterIP
  selector:
    app: finch-backend
  ports:
    - port: 8000
      targetPort: 8000
```

---

## PostgreSQL Deployment, PVC, and Service

**File:** `postgres-deployment.yaml`

- PersistentVolumeClaim requests 5Gi storage.
- Deploys `postgres:15` with environment variables for DB, user, password.
- Uses PVC for persistent data storage.
- Service type `ClusterIP` on port 5432.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: capstone
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: capstone
spec:
  selector:
    matchLabels:
      app: postgres
  replicas: 1
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_DB
              value: fleetdb
            - name: POSTGRES_USER
              value: roy77
            - name: POSTGRES_PASSWORD
              value: asdf1234@77
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgres-storage
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: capstone
spec:
  type: ClusterIP
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
```

---

## Secrets for Backend Environment Variables

**File:** `deps.secrets.yaml`

- Contains database credentials, Stripe keys, and site URLs.
- Used by backend deployment via `envFrom`.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: django-env-secret
  namespace: capstone
type: Opaque
stringData:
  POSTGRES_DB: fleetdb
  POSTGRES_USER: roy77
  POSTGRES_PASSWORD: asdf1234@77
  POSTGRES_HOST: postgres
  POSTGRES_PORT: "5432"  
  PYTHONUNBUFFERED: "1"
  STRIPE_SECRET_KEY: "sk_test_51R2EHqFxSoY5Rjyor0IeGElOkgFZMziWh7YMbEydw7Xrby1GlzZijsW3JXw7sqhTRWTKKTIsG4xxPXhlCEQd5dRU00zvsQANcy"
  STRIPE_PUBLIC_KEY: "pk_test_51R2EHqFxSoY5RjyoISdnIQZkHvt6LmxJdpdEVxG3MYvlhbkgOQPzTUck2pZtcbKLQQKasG7VHZNkzMH7bCv6XmC600QlbooiKJ"
  STRIPE_WEBHOOK_SECRET: "whsec_uAoNngWqlE3qITAux0SEVXMcMdv5yT1I"
  SITE_URL: "http://93.127.195.189:5000"
  FRONTEND_SITE_URL: "https://finch-development.vercel.app"
```

---

## Ingress Configuration

**File:** `ingress.yaml`

- Uses nginx ingress controller.
- Routes traffic to frontend on `frontend.local` (port 80).
- Routes traffic to backend on `backend.local` (port 8000).

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: finch-ingress
  namespace: capstone
spec:
  ingressClassName: nginx
  rules:
    - host: frontend.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: finch-frontend
                port:
                  number: 80
    - host: backend.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: finch-backend
                port:
                  number: 8000
```

---

## Resource Allocation & Scaling

- Currently set to **1 replica** per deployment for simplicity.
- Consider setting resource requests/limits per container for CPU and memory.
- Enable Horizontal Pod Autoscaler (HPA) to scale pods based on CPU/memory in production.

---

## Summary

- Backend and frontend services are isolated and communicate via ClusterIP services.
- PostgreSQL persists data using a PersistentVolumeClaim.
- Secrets securely inject sensitive config into backend.
- Ingress routes public traffic.
- Namespace `capstone` keeps resources scoped.

---

If you want, I can prepare this as a downloadable `.md` file for you too! Just let me know.
