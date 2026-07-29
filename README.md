# helm-lesson

A minimal, hands-on Helm chart (`webapp/`) deploying a stateless nginx web app —
built step by step to learn core Helm chart mechanics: `Chart.yaml`, values,
named templates, conditional/looped resources, and packaging.

## Prerequisites

- [Helm](https://helm.sh/) v3+ (built/tested with v4)
- A Kubernetes cluster (e.g. `kind`, `minikube`, or Rancher Desktop's bundled `kind` cluster)
- `kubectl` configured against that cluster

## Chart structure

```
webapp/
├── Chart.yaml              # chart identity: name, chart version, appVersion (nginx tag)
├── values.yaml              # default values consumed by templates/
└── templates/
    ├── _helpers.tpl          # named templates: naming + label helpers
    ├── deployment.yaml        # Deployment (replicas, image, resources)
    ├── service.yaml           # ClusterIP Service in front of the Deployment
    ├── ingress.yaml            # optional Ingress, gated by values.ingress.enabled
    └── NOTES.txt               # post-install/upgrade usage hints
```

## Usage

```bash
# Validate the chart (structure/syntax)
helm lint webapp

# Render manifests locally without touching a cluster
helm template my-webapp webapp

# Install onto the current kubectl context
helm install my-webapp webapp

# Check status / re-print NOTES.txt
helm status my-webapp
kubectl get pods,svc -l app.kubernetes.io/instance=my-webapp

# Reach the ClusterIP service locally
kubectl port-forward svc/my-webapp-webapp 8080:80

# Apply a values change to a running release
helm upgrade my-webapp webapp --set replicaCount=3

# Roll back to a previous revision
helm rollback my-webapp 1

# Remove the release
helm uninstall my-webapp
```

## Key values

| Key | Default | Purpose |
|---|---|---|
| `replicaCount` | `1` | Number of Deployment replicas |
| `image.repository` / `image.tag` | `nginx` / `""` (falls back to `Chart.yaml` `appVersion`) | Container image |
| `service.port` | `80` | Service port (and container port) |
| `ingress.enabled` | `false` | Toggles rendering the Ingress resource |
| `ingress.hosts` | `chart-example.local` | Hostnames routed by the Ingress |
| `resources` | `{}` | Pod resource requests/limits |

Override any of these with `--set key=value` or `-f custom-values.yaml`.

## Packaging & distribution

```bash
helm package webapp
# → webapp-<version>.tgz, versioned from Chart.yaml
```

From there, push to an OCI registry (`helm push webapp-0.1.0.tgz oci://<registry>/charts`)
or host as a classical chart repo (`helm repo index`) — see chart repo docs for details.
