# Docker + Kubernetes - Skill Completo

> Guía completa para contenedorización y orquestación moderna.
> Transversal, sin dependencias de proyectos específicos.

---

## Tabla de Contenidos

1. [Docker Fundamentals](#1-docker-fundamentals)
2. [Docker Compose](#2-docker-compose)
3. [Dockerfile Best Practices](#3-dockerfile-best-practices)
4. [Kubernetes Basics](#4-kubernetes-basics)
5. [K8s Resources](#5-k8s-resources)
6. [Helm](#6-helm)
7. [Deployment](#7-deployment)
8. [Monitoring](#8-monitoring)

---

## 1. Docker Fundamentals

### 1.1 Comandos Esenciales

```bash
# Build
docker build -t my-app:latest .
docker build -t my-app:latest --build-arg ENV=production .

# Run
docker run -d -p 8080:80 --name my-app my-app:latest
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:16

# Inspect
docker ps
docker logs -f my-app
docker exec -it my-app sh
docker inspect my-app

# Cleanup
docker system prune -af
docker rmi $(docker images -q)
```

### 1.2 Networks

```bash
# Create network
docker network create my-network

# Run with network
docker run -d --network my-network --name api my-api:latest

# Connect containers
docker network connect my-network container2
```

### 1.3 Volumes

```bash
# Named volume
docker volume create my-data
docker run -v my-data:/app/data my-app:latest

# Bind mount (development)
docker run -v $(pwd):/app my-app:latest

# Read-only mount
docker run -v /data:/app/data:ro my-app:latest
```

---

## 2. Docker Compose

### 2.1 Compose V2

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/app
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api
    networks:
      - backend

networks:
  backend:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
```

### 2.2 Compose Commands

```bash
# Up
docker compose up -d
docker compose up -d --build

# Logs
docker compose logs -f
docker compose logs -f api

# Down
docker compose down
docker compose down -v  # Remove volumes

# Scale
docker compose up -d --scale api=3
```

---

## 3. Dockerfile Best Practices

### 3.1 Multi-stage Build (Node/NPM)

```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Builder
FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Stage 3: Runner
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

# Create non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nodejs

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json

USER nodejs
EXPOSE 3000

CMD ["node", "dist/main.js"]
```

### 3.2 Multi-stage Build (Python/FastAPI)

```dockerfile
# Stage 1: Dependencies
FROM python:3.12-slim AS dependencies
WORKDIR /app
RUN pip install --no-cache-dir poetry
COPY pyproject.toml poetry.lock ./
RUN poetry config virtualenvs.create false \
    && poetry install --no-dev --no-interaction

# Stage 2: Builder
FROM python:3.12-slim AS builder
WORKDIR /app
COPY --from=dependencies /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY . .

# Stage 3: Runner
FROM python:3.12-slim AS runner
WORKDIR /app
RUN groupadd -r appuser && useradd -r -g appuser appuser
COPY --from=builder /app .
USER appuser
EXPOSE 8000
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 3.3 Multi-stage Build (.NET)

```dockerfile
# Stage 1: Restore
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS restore
WORKDIR /src
COPY ["MyApp.csproj", "./"]
RUN dotnet restore MyApp.csproj

# Stage 2: Build
FROM restore AS build
WORKDIR /src
COPY . .
RUN dotnet build MyApp.csproj -c Release -o /app/build

# Stage 3: Publish
FROM build AS publish
WORKDIR /src
RUN dotnet publish MyApp.csproj -c Release -o /app/publish

# Stage 4: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### 3.4 Best Practices Summary

| Práctica | ✅ Hacer | ❌ Evitar |
|----------|---------|----------|
| Base image | alpine/slim | latest |
| Layers | Multiple RUN agrupados | RUN separado por comando |
| Caching | Copy package.json primero | Copy todo antes |
| Usuario | Non-root | Root |
| Secrets | Build args, no env | ENV con secrets |
| Multi-stage | Sí | Single stage grande |

---

## 4. Kubernetes Basics

### 4.1 K8s Architecture

```
┌─────────────────────────────────────────────────────┐
│                  K8s Cluster                         │
│  ┌─────────────┐    ┌─────────────┐               │
│  │   Master    │    │   Master    │               │
│  │  (Control   │    │  (Control   │               │
│  │   Plane)    │    │   Plane)    │               │
│  └──────┬──────┘    └──────┬──────┘               │
│         │                   │                      │
│  ┌──────┴───────────────────┴──────┐               │
│  │         Node Pool               │               │
│  │  ┌────────┐  ┌────────┐       │               │
│  │  │ Worker │  │ Worker │       │               │
│  │  │ Node 1 │  │ Node 2 │       │               │
│  │  └────────┘  └────────┘       │               │
│  └─────────────────────────────────┘               │
└─────────────────────────────────────────────────────┘
```

### 4.2 kubectl Commands

```bash
# Context
kubectl config use-context production
kubectl config current-context

# Pods
kubectl get pods -n namespace
kubectl get pods -o wide
kubectl logs -f pod-name
kubectl exec -it pod-name -- /bin/bash
kubectl describe pod pod-name

# Deployments
kubectl get deployments
kubectl apply -f deployment.yaml
kubectl rollout restart deployment/app
kubectl rollout status deployment/app

# Services
kubectl get svc
kubectl get endpoints
kubectl port-forward svc/app 8080:80

# Debug
kubectl top pod
kubectl events --sort-by='.lastTimestamp'
```

---

## 5. K8s Resources

### 5.1 Pod

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
    - name: app
      image: my-app:latest
      ports:
        - containerPort: 3000
      env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
      resources:
        requests:
          memory: "128Mi"
          cpu: "100m"
        limits:
          memory: "256Mi"
          cpu: "500m"
      livenessProbe:
        httpGet:
          path: /health
          port: 3000
        initialDelaySeconds: 30
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /ready
          port: 3000
        initialDelaySeconds: 5
        periodSeconds: 5
```

### 5.2 Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: my-app
    spec:
      serviceAccountName: app-sa
      containers:
        - name: app
          image: my-app:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 3000
          envFrom:
            - configMapRef:
                name: app-config
            - secretRef:
                name: app-secrets
          volumeMounts:
            - name: cache
              mountPath: /app/cache
      volumes:
        - name: cache
          emptyDir: {}
```

### 5.3 Service

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
```

### 5.4 Service Types

| Type | Descripción | Uso |
|------|-------------|-----|
| **ClusterIP** | Solo interno | Microservicios |
| **NodePort** | Expuesto en cada nodo | Desarrollo |
| **LoadBalancer** | Balanceador externo | Cloud production |
| **ExternalName** | DNS CNAME | External services |

### 5.5 Ingress

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - api.example.com
      secretName: api-tls
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-svc
                port:
                  number: 80
```

### 5.6 ConfigMap & Secrets

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "db-service"
  REDIS_HOST: "redis-service"
  LOG_LEVEL: "info"
---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  DATABASE_PASSWORD: "changeme"
  API_KEY: "secret-key"
```

### 5.7 PVC (Persistent Volume)

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

### 5.8 Horizontal Pod Autoscaler

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### 5.9 RBAC

```yaml
# service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: production
---
# role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "watch"]
---
# role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-role-binding
  namespace: production
subjects:
  - kind: ServiceAccount
    name: app-sa
    namespace: production
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io
```

---

## 6. Helm

### 6.1 Helm Commands

```bash
# Install
helm install my-app ./chart
helm install my-app repo/chart-name
helm upgrade my-app ./chart
helm rollback my-app 1

# Repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Values
helm install my-app ./chart --set image.tag=v1.0
helm install my-app ./chart --values values.yaml
```

### 6.2 Chart Structure

```
my-chart/
├── Chart.yaml
├── values.yaml
├── values-prod.yaml
├── values-staging.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl
│   └── configmap.yaml
└── charts/
```

### 6.3 values.yaml

```yaml
replicaCount: 3

image:
  repository: my-app
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

nodeSelector: {}

tolerations: []

affinity: {}
```

### 6.4 _helpers.tpl

```yaml
{{- define "my-chart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "my-chart.labels" -}}
app.kubernetes.io/name: {{ include "my-chart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion }}
{{- end }}
```

---

## 7. Deployment

### 7.1 Namespace

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
---
apiVersion: v1
kind: Namespace
metadata:
  name: staging
  labels:
    environment: staging
```

### 7.2 Kustomize Overlay

```yaml
# kustomization.yaml (base)
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml

commonLabels:
  app: my-app
```

```yaml
# production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
patches:
  - path: deployment-patch.yaml
images:
  - name: my-app
    newTag: v1.2.0
```

### 7.3 GitOps with ArgoCD

```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: production
  source:
    repoURL: https://github.com/org/repo
    targetRevision: HEAD
    path: k8s/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## 8. Monitoring

### 8.1 Prometheus + Grafana

```yaml
# prometheus.yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
spec:
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      team: backend
  resources:
    requests:
      memory: 1Gi
      cpu: 500m
```

```yaml
# service-monitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
  labels:
    team: backend
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
    - port: metrics
      path: /metrics
```

### 8.2 Logging with Loki

```yaml
# pod with logging
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "3000"
  prometheus.io/path: "/metrics"
```

---

## Checklist

### Docker
- [ ] Multi-stage build
- [ ] Non-root user
- [ ] Alpine images
- [ ] .dockerignore
- [ ] Healthcheck

### Kubernetes
- [ ] Resource limits
- [ ] Liveness/Readiness probes
- [ ] RBAC configured
- [ ] Secrets via Vault o external-secrets
- [ ] HPA configured

### Helm
- [ ] Values por ambiente
- [ ] Helpers definidos
- [ ] Version pinning

### CI/CD
- [ ] Image scanning (Trivy)
- [ ] Resource quotas
- [ ] Network policies

---

*Docker + Kubernetes Modern Skill*
*Versión: 1.0*