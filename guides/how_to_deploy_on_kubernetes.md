---
prev: false
next: false
---

# Deploy on Kubernetes

This guide covers deploying an Alchemy app on Kubernetes. It assumes you are familiar with Kubernetes concepts (Pods, Deployments, Services, Ingress) and have a cluster running.

<ServiceAd href="https://blish.cloud/en/solutions/alchemycms">
<template #before>Don't want to manage your own infrastructure? Our sponsor</template>
<template #after>offers <a href="https://blish.cloud/en/solutions/alchemycms" target="_blank">managed Alchemy hosting</a>.</template>
</ServiceAd>

## Docker Image

Use the same `Dockerfile` as any Rails app. The key requirements for Alchemy:

- Install `libvips` or `imagemagick` for image processing
- Run `db:prepare` on startup

~~~ dockerfile
FROM docker.io/library/ruby:3.4-slim AS base
WORKDIR /rails

RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y curl libjemalloc2 libvips && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# ... build stage ...

ENTRYPOINT ["/rails/bin/docker-entrypoint"]
CMD ["./bin/thrust", "./bin/rails", "server"]
~~~

::: tip
Rails 7.1+ generates a production-ready `Dockerfile`. See the [Kamal guide](how_to_deploy_with_kamal#dockerfile) for a full example.
:::

## Deployment

A basic Deployment manifest for the Alchemy web process:

~~~ yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alchemy-web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: alchemy
  template:
    metadata:
      labels:
        app: alchemy
    spec:
      containers:
        - name: web
          image: your-registry/alchemy-app:latest
          ports:
            - containerPort: 3000
          envFrom:
            - secretRef:
                name: alchemy-secrets
            - configMapRef:
                name: alchemy-config
          readinessProbe:
            httpGet:
              path: /admin/login
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /admin/login
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 15
~~~

## Configuration

Split environment variables into a ConfigMap for non-sensitive values and a Secret for credentials:

~~~ yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alchemy-config
data:
  RAILS_ENV: production
  RAILS_LOG_TO_STDOUT: "true"
  RAILS_SERVE_STATIC_FILES: "true"
~~~

~~~ yaml
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: alchemy-secrets
type: Opaque
stringData:
  SECRET_KEY_BASE: "your-secret-key"
  RAILS_MASTER_KEY: "your-master-key"
  DATABASE_URL: "postgres://user:pass@db-host:5432/alchemy_production"
~~~

::: warning
Do not commit secrets to git. Use `kubectl create secret`, Sealed Secrets, or an external secret manager (Vault, AWS Secrets Manager, etc.).
:::

## Database Migrations

Run `db:prepare` as an init container so migrations complete before the web process starts:

~~~ yaml
# Add to spec.template.spec in your Deployment
initContainers:
  - name: migrate
    image: your-registry/alchemy-app:latest
    command: ["bin/rails", "db:prepare"]
    envFrom:
      - secretRef:
          name: alchemy-secrets
      - configMapRef:
          name: alchemy-config
~~~

This ensures migrations run exactly once per deploy, before any web pods receive traffic.

## Service and Ingress

Expose the Deployment with a Service and route external traffic through an Ingress:

~~~ yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: alchemy-web
spec:
  selector:
    app: alchemy
  ports:
    - port: 80
      targetPort: 3000
~~~

~~~ yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alchemy-web
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
    - hosts:
        - example.com
      secretName: alchemy-tls
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: alchemy-web
                port:
                  number: 80
~~~

::: tip
This example uses [cert-manager](https://cert-manager.io) for automatic TLS certificates. Adjust the annotations for your Ingress controller (Nginx, Traefik, etc.).
:::

## Background Worker

Alchemy uses Active Job for page publishing and other tasks. A worker process is required for these jobs to run.

A separate Deployment is recommended because web and worker processes have different scaling and resource needs. Workers are CPU/memory-heavy during image processing, while web pods need low latency. Separating them allows independent scaling and prevents a crashing worker from affecting web traffic.

~~~ yaml
# k8s/worker-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alchemy-worker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alchemy-worker
  template:
    metadata:
      labels:
        app: alchemy-worker
    spec:
      containers:
        - name: worker
          image: your-registry/alchemy-app:latest
          command: ["bin/rails", "solid_queue:start"]
          envFrom:
            - secretRef:
                name: alchemy-secrets
            - configMapRef:
                name: alchemy-config
~~~

Adjust the command for your queue adapter (e.g. `bundle exec sidekiq` for Sidekiq).

::: tip
For simple setups, you can run the worker in the same pod as the web process by setting `SOLID_QUEUE_IN_PUMA=true`. This avoids a separate Deployment at the cost of shared resources.
:::

## Storage

Alchemy needs persistent storage for uploaded pictures and attachments. On Kubernetes you have two options:

**Remote storage (recommended)** — configure ActiveStorage with S3, GCS, or Azure Blob Storage. This is the best option for multi-replica deployments because all pods share the same storage. See the [Deployment guide](deployment#remote-storage) for the setup.

**PersistentVolumeClaim** — for single-replica deployments, you can use a PVC. Note that `ReadWriteOnce` volumes cannot be shared across pods, so this limits you to one replica.

~~~ yaml
# k8s/pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alchemy-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
~~~

Mount it in the Deployment:

~~~ yaml
volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: alchemy-storage

# In the container spec:
volumeMounts:
  - name: storage
    mountPath: /rails/storage
~~~

## Caching

Alchemy sets `Cache-Control` headers on published pages. In a Kubernetes setup, you can place a CDN (Cloudflare, CloudFront) in front of the Ingress, or add a caching reverse proxy like Varnish as a sidecar.

See the [Deployment guide](deployment#caching) for cache configuration options.
