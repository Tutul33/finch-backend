# 🛠️ Capstone Project – Full Stack Application with Kubernetes, Monitoring, and CI/CD

This is a full-stack capstone project deployed using Docker and Kubernetes with end-to-end observability, CI/CD pipelines, and secret management.

---

## 📁 Project Structure

```
.
├── .github/workflows/          # GitHub Actions for CI/CD
├── .grafana/dashboards/       # Custom Grafana dashboard JSONs
├── api/                        # Backend Django API
├── core/                       # Core backend logic
├── k8s/                        # Kubernetes manifests
├── static/, templates/, ...   # Static frontend resources
├── docker-compose.yml         # Local development docker config
├── Dockerfile                 # Backend container build config
├── Monitoring_Setup.md        # Prometheus, Grafana, Loki setup
├── Secret_Management.md       # Kubernetes secret handling
├── ci_cd_setup.md             # GitHub Actions CI/CD pipeline
├── Kubernetes_Architecture.md # K8s resource and service layout
```

---

## 🚀 Features

- 🔧 **Backend**: Django REST API
- 🌐 **Frontend**: Static + template files
- 🐳 **Containerized**: Docker & docker-compose
- ☘️ **Kubernetes**: Manifests for backend, frontend, DB, Redis
- 🔐 **Secrets**: Managed via `Secret_Management.md`
- 📊 **Monitoring**: Prometheus, Grafana, Loki
- 🔄 **CI/CD**: Automated builds & deployments via GitHub Actions

---

## 🧪 Local Development

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
```

### 2. Run with Docker Compose

```bash
docker-compose up --build
```

Backend should be available at `http://localhost:8000`

---

## ☘️ Kubernetes Deployment

### 1. Apply Manifests

```bash
kubectl apply -f k8s/
```

### 2. Port Forward Grafana (Monitoring)

```bash
kubectl port-forward svc/prometheus-grafana -n capstone 3000:80
```

Access Grafana at: [http://localhost:3000](http://localhost:3000)

Default login:

- **user**: `admin`
- **password**: `prom-operator` *(or get from secret)*

---

## 🛡️ Secrets

Secrets are managed using Kubernetes `Secret` resources. See [Secret\_Management.md](./Secret_Management.md) for how to create and apply them securely.

---

## 🔄 CI/CD

GitHub Actions workflow defined in `.github/workflows`:

- On push to `main`: builds Docker image, pushes to Docker Hub, deploys to cluster.

See [ci\_cd\_setup.md](./ci_cd_setup.md) for more details.

---

## 📈 Monitoring & Dashboards

Observability stack includes:

- **Prometheus**: Metrics collection
- **Grafana**: Visual dashboards ([`.grafana/dashboards`](./.grafana/dashboards))
- **Loki**: Log aggregation

Details in [Monitoring\_Setup.md](./Monitoring_Setup.md)

---

## 📄 Documentation

- [`Kubernetes_Architecture.md`](./Kubernetes_Architecture.md) — Namespace, pod, service, and ingress structure.
- [`Monitoring_Setup.md`](./Monitoring_Setup.md) — How to deploy Prometheus + Grafana + Loki.
- [`Secret_Management.md`](./Secret_Management.md) — Secure handling of sensitive credentials.
- [`ci_cd_setup.md`](./ci_cd_setup.md) — CI/CD workflow using GitHub Actions.
- [`docker_containerization.md`](./docker_containerization.md) — Dockerfile & containerization strategy.

---

## 👨‍💼 Author

**Tutul33**\
GitHub: [Tutul33](https://github.com/Tutul33)

---

## 📜 License

This project is licensed under the MIT License.

