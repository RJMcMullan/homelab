## FluxCD Components

FluxCD is a GitOps tool that automates the deployment of applications and infrastructure to Kubernetes. Below is an overview of the main FluxCD components and their roles within the GitOps workflow:

### 1. **GitRepository**

The `GitRepository` resource defines a connection between FluxCD and your Git repository. FluxCD uses this resource to track changes in the repository and automatically apply those changes to your Kubernetes cluster.

#### Example:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 1m
  url: https://gitlab.com/your-repo/your-repo-name.git
  branch: main
  ref:
    branch: main
  secretRef:
    name: git-credentials
```

- interval: How often FluxCD checks for updates in the repository.
- url: The URL of your Git repository.
- branch: The branch that FluxCD will track (usually main).
- secretRef: A reference to a Kubernetes secret for Git repository authentication.

### 2. Kustomization
The Kustomization resource defines how FluxCD should apply the resources in your Git repository to your Kubernetes cluster. It specifies the source, the path to the resources, and the intervals at which to check for updates.

Example:
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

- interval: How often FluxCD checks for updates in the specified path.
- dependsOn: Specifies dependencies between different - Kustomizations to ensure correct deployment order.
- sourceRef: Refers to the GitRepository resource.
- path: The path in the repository that contains the Kubernetes manifests or Helm charts to be applied.
- prune: Ensures that resources deleted from the repository are also removed from the cluster.
- wait: Waits for all resources to become ready before proceeding.
- timeout: Specifies the maximum time to wait for resources to be ready.


### 3. HelmRepository
The HelmRepository resource defines an external Helm chart repository from which FluxCD can pull charts. It allows you to use pre-built Helm charts for deploying applications or infrastructure components.

Example:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: nfs-subdir-external-provisioner
  namespace: flux-system
spec:
  interval: 60m
  url: https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
```
- interval: Specifies how often to check the repository for new charts.
- url: The URL of the Helm repository.

### 4. HelmRelease
The HelmRelease resource manages the lifecycle of a Helm chart deployment. It uses a HelmRepository resource as the source and applies the Helm chart to your cluster.

Example:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta2
kind: HelmRelease
metadata:
  name: nfs-subdir-external-provisioner
  namespace: nfs-subdir-external-provisioner
spec:
  interval: 30m
  chart:
    spec:
      chart: nfs-subdir-external-provisioner
      version: "4.0.18"
      sourceRef:
        kind: HelmRepository
        name: nfs-subdir-external-provisioner
        namespace: flux-system
```

- interval: How often to check for chart updates.
- chart: Specifies the Helm chart and version to deploy.
- sourceRef: References the HelmRepository that contains the chart.

### 5. ImageRepository and ImagePolicy (Optional)
If your deployment involves container images, the ImageRepository and ImagePolicy resources manage the discovery and updates of container images.

ImageRepository: Monitors a container image repository for new tags.
ImagePolicy: Defines rules for selecting which image tag to deploy (e.g., latest stable version).

Example:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1alpha1
kind: ImageRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 5m
  image: my-docker-registry.com/my-app
```

```yaml
apiVersion: image.toolkit.fluxcd.io/v1alpha1
kind: ImagePolicy
metadata:
  name: my-app-policy
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: my-app
  policy:
    semver:
      range: 1.0.x
```

### 6. Notifications (Optional)
FluxCD can also integrate with external systems like Slack or Microsoft Teams to notify you of changes in the GitOps pipeline. This can be configured using the Alert and Receiver resources.

Example:

```yaml
apiVersion: notification.toolkit.fluxcd.io/v1beta1
kind: Alert
metadata:
  name: flux-alert
  namespace: flux-system
spec:
  providerRef:
    name: slack
  eventSeverity: info
  eventSources:
    - kind: Kustomization
      name: apps
```
