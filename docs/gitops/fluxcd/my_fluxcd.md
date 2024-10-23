# My Configuration
## FluxCD GitOps Bootstrap with GitLab

This section provides instructions for bootstrapping FluxCD with GitLab using a Personal Access Token (PAT). Follow these steps to create a FluxCD setup that automatically synchronizes your Kubernetes manifests with a GitLab repository.

## Steps

### 1. Create the Directory Structure

First, create the necessary directory structure for FluxCD to store the cluster configurations.

```bash
mkdir -p clusters/k8s-prod
```
This command creates the clusters/k8s-prod directory where FluxCD will store and apply your Kubernetes manifests.

### 2. Create a Personal Access Token (PAT) in GitLab
To authenticate FluxCD with GitLab, you need to create a Personal Access Token (PAT) with the api scope in GitLab.

1. Go to GitLab and navigate to User Settings > Access Tokens.
2. Create a new token with the following settings:
    - Name: FluxCD Token
    - Scopes: Select the api scope.
3. Copy the generated token and store it in a safe place. You will use this token to authenticate FluxCD with GitLab.

### 3. Set the Personal Access Token (PAT) as an Environment Variable
Export the GitLab PAT token as an environment variable in your terminal.

```bash
export GITLAB_TOKEN=<token here>
```

### 4. Bootstrap FluxCD with GitLab

Now, bootstrap FluxCD with your GitLab repository using the following command:

```bash
flux bootstrap gitlab --token-auth --ca-file=ca.crt --hostname=https://gitlab-docker01.internal --owner="homelab" --repository "flux-k8s-gitops" --branch=main --path=clusters/k8s-prod
```
### Explanation of Flags:

- `--token-auth`: Specifies that the authentication will be done using the GitLab PAT token.
- `--ca-file=ca.crt`: Specifies the CA file for SSL verification (if using internal GitLab hosting with custom CA).
- `--hostname=https://gitlab-docker01.internal`: The URL of your self-hosted GitLab instance.
- `--owner="homelab"`: The GitLab group or user that owns the repository.
- `--repository "flux-k8s-gitops"`: The GitLab repository where FluxCD will store and track the manifests.
- `---branch=main`: The Git branch to track for changes.
- `--path=clusters/k8s-prod`: The path within the repository where the cluster configurations are stored.

### 5. Verify the Bootstrap
After running the bootstrap command, FluxCD should automatically install its components and begin tracking your GitLab repository for changes.

To verify that FluxCD has been successfully bootstrapped, use the following command:

```bash
flux check
```
This command checks the status of FluxCD and confirms that it has successfully synchronized with your GitLab repository.

# FluxCD GitOps (Monorepo) Deployment with Base and Overlay Structure

My repository is structured for deploying applications and infrastructure using FluxCD in a Kubernetes cluster. Everything which is deployed is contained in a single git repository. It follows a base and overlay pattern, with base Helm configurations and environment-specific patches for production. Additionally, the `flux-system` directory contains the FluxCD bootstrap components, and Flux is configured to automatically apply changes when they are pushed to the main branch.

## Repository Structure

The repository is organized as follows:

- `apps/`
  - `base/`: Contains the base Helm configurations for all applications.
  - `production/`: Contains overlay configurations to patch the base for production deployments.

- `infrastructure/`
  - `base/`: Contains the base configurations for infrastructure components (e.g., Grafana, NGINX Ingress, Loki, MetalLB, NFS Subdir External Provisioner etc).
  - `production/`: Contains overlay configurations for infrastructure components for the production environment.

- `clusters/`
  - `k8s-prod/`
    - `infrastructure.yaml`: Contains the            kustomization files to deploy the infrastructure components.
    - `apps.yaml`: Contains the kustomization files to recursively deploy applications.
    - `flux-system/`: Contains the FluxCD bootstrap components, including the controllers and initial configuration for GitOps integration.

## Key Components

### Base Configurations (`apps/base`)

The `apps/base` directory contains the default Helm configurations for the applications. These are reusable across different environments (e.g., production, staging, development).

### Production Overlays (`apps/production`)

The `apps/production` directory contains the overlays to patch the base configurations specifically for the production environment. These overlays may modify the base configurations to adjust resource limits, replicas, or other production-specific settings.

### Cluster-Specific Kustomization (`clusters/k8s-prod/`)

The `clusters/k8s-prod` directory contains the `Kustomization` resource to apply the production environment configuration to the cluster.

### Flux Bootstrap Components (`clusters/flux-system/`)

The `clusters/flux-system` directory contains the FluxCD bootstrap components, which manage the GitOps workflow, monitor the repository, and apply changes automatically when updates are pushed to the main branch.

### Example `Kustomization.yaml` for Production

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 10m0s
  dependsOn:
    - name: ingress-nginx
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./apps/production
  prune: true
  wait: true
  timeout: 5m0s
```


### Key Parameters

- interval: FluxCD will check for updates every 10 minutes.
- dependsOn: Specifies that the deployment of the applications depends on the successful deployment of the ingress-nginx Kustomization.
- sourceRef: Points to the GitRepository resource (flux-system), which provides the configurations for deployment.
- path: Specifies the path to the production-specific overlay configurations (./apps/production).
- prune: Ensures that resources no longer present in the Git repository are deleted from the cluster.
- wait: Ensures that FluxCD waits for all resources to be fully ready before proceeding.
- timeout: Sets the maximum wait time for resources to become ready (5 minutes).
- Deployment Process

## Infrastructure Kustomizations

The `infrastructure.yaml` file in the `clusters/k8s-prod/` directory includes multiple Kustomizations for various infrastructure components. These Kustomizations specify dependencies, intervals, retry intervals, timeouts, and paths for each component.

### Example Kustomizations in `infrastructure.yaml`

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: ingress-nginx
  namespace: flux-system
spec:
  dependsOn:
  - name: metallb
  interval: 1h
  retryInterval: 1m
  timeout: 15m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/production/ingress-nginx
  prune: true
  wait: true
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: cert-manager
  namespace: flux-system
spec:
  interval: 1h
  retryInterval: 1m
  timeout: 5m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/production/cert-manager
  prune: true
  wait: true
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infra-configs
  namespace: flux-system
spec:
  dependsOn: 
  - name: ingress-nginx
  - name: cert-manager
  interval: 1h
  retryInterval: 1m
  timeout: 5m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/production/configs
  prune: true
  wait: true
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: metallb
  namespace: flux-system
spec:
  interval: 1h
  retryInterval: 1m
  timeout: 5m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/production/metallb
  prune: true
  wait: true
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: grafana
  namespace: flux-system
spec:
  interval: 5m
  dependsOn:
  - name: ingress-nginx
  - name: nfs-subdir-external-provisioner
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/production/grafana
  prune: true
  wait: true
```

Since FluxCD is bootstrapped to monitor the main branch of this repository for changes, any updates pushed to the repository will be automatically applied to the cluster. There’s no need to manually run kubectl apply -k.

### How It Works
Push Changes: When you push new changes or updates to the main branch, FluxCD will automatically detect the changes and apply them to your Kubernetes cluster.
Monitor the Deployment: 

You can monitor the status of the FluxCD deployment using the following commands:

```bash
flux get kustomizations -n flux-system # Gets all the kustomizations in that namespace

flux get helmrelease ingress-nginx -n ingress-nginx # Gets helm releases in that namespace

flux events -n ingress-nginx # This gets all flux events in that namespace
```
