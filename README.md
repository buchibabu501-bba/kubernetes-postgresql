# kubernetes-postgresql

PostgreSQL deployed on Kubernetes, in two environments:

1. **Local** — Docker Desktop's built-in Kubernetes (done, see [`k8s/local`](k8s/local))
2. **AWS (EKS)** — planned, see [`k8s/aws`](k8s/aws)

## Architecture (local)

A single-node Kubernetes cluster (Docker Desktop) running:

- A `postgres` Namespace, isolating everything below from the rest of the cluster
- A `Deployment` running `postgres:16-alpine`, with liveness/readiness probes via `pg_isready`
- A `PersistentVolumeClaim` so data survives pod restarts
- A `ClusterIP` `Service` exposing Postgres on port 5432 inside the cluster
- Credentials injected from a `Secret`, created imperatively (never committed to git)

Managed as a single [Kustomize](https://kustomize.io/) unit under `k8s/local/`.

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) with Kubernetes enabled (Settings → Kubernetes → Enable Kubernetes)
- `kubectl` (bundled with Docker Desktop)

## Quickstart (local)

```bash
# 1. Create the namespace first (secret below needs it to exist)
kubectl apply -f k8s/local/namespace.yaml

# 2. Create the database credentials (not stored in this repo)
kubectl create secret generic postgres-secret \
  --namespace postgres \
  --from-literal=POSTGRES_USER=postgres \
  --from-literal=POSTGRES_PASSWORD=changeme \
  --from-literal=POSTGRES_DB=appdb

# 3. Deploy everything else (PVC, Deployment, Service)
kubectl apply -k k8s/local/

# 4. Watch it come up
kubectl get pods -n postgres --watch
```

## Connecting

```bash
# From inside the cluster, other pods reach it at:
#   postgres.postgres.svc.cluster.local:5432

# From your Mac, port-forward it:
kubectl port-forward -n postgres svc/postgres 5432:5432

# Then, in another terminal:
psql -h localhost -U postgres -d appdb
```

## Tearing down

```bash
kubectl delete -k k8s/local/
kubectl delete secret postgres-secret -n postgres
kubectl delete namespace postgres
```

## Roadmap

- [x] Local Kubernetes (Docker Desktop) + PostgreSQL
- [ ] AWS EKS cluster (Terraform)
- [ ] PostgreSQL on EKS, backed by EBS-backed persistent volumes
