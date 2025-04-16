Sure! Below is a sample `README.md` that outlines the steps for transitioning from a single namespace (default) environment to multiple namespaces (default and prod) in an AKS Kubernetes cluster using FluxCD and Kustomize. It includes the necessary Flux commands, changes to repositories, and how to switch from the `master` branch to `flux-k8s-multi-env`.

---

# Transition from Single Namespace to Multiple Namespaces (default, prod) in AKS Kubernetes with FluxCD
# Switch from the master branch to flux-k8s-multi-env

This documentation outlines the steps to transition from using a single `default` namespace to multiple namespaces, specifically `default` and `prod`, in an Azure Kubernetes Service (AKS) cluster. It leverages FluxCD for GitOps and Kustomize for environment-specific overlays.

## **Prerequisites**

- An AKS cluster is already set up.
- FluxCD is installed on the cluster.
- GitHub repository with Kustomize configurations for Kubernetes deployments.

---

## **Steps to Transition to Multiple Environments with FluxCD**

### 1. **Create a New Git Branch for Multi-Env Support**
Initially, the repository used the `master` branch for deployments. To manage both `default` and `prod` namespaces, create a new branch named `flux-k8s-multi-env`.

```bash
# Create a new branch for multi-env setup
git checkout master
git checkout -b flux-k8s-multi-env
git push origin flux-k8s-multi-env
```

---

### 2. **Update `flux-system` GitRepository Resource**
The FluxCD GitRepository resource for the `flux-system` is still pointing to the `master` branch. We need to change it to point to the `flux-k8s-multi-env` branch.

#### Change the branch in the FluxCD GitRepository:

```bash
# Suspend the GitRepository to avoid auto-reconciliation during changes
flux suspend source git flux-system -n flux-system

# Edit the GitRepository resource
kubectl edit gitrepository flux-system -n flux-system
```

Update the `spec.ref.branch` field:

```yaml
spec:
  ref:
    branch: flux-k8s-multi-env
```

Save the changes.

#### Resume the GitRepository:

```bash
flux resume source git flux-system -n flux-system
```

#### Reconcile the changes:

```bash
flux reconcile source git flux-system -n flux-system
```

This updates the FluxCD source to pull from the `flux-k8s-multi-env` branch.

---

### 3. **Update the PodInfo GitRepository Resource**
Similarly, update the `podinfo` GitRepository resource to point to the new `flux-k8s-multi-env` branch.

#### Change the branch in the `podinfo` GitRepository:

```bash
# Suspend the GitRepository to avoid auto-reconciliation during changes
flux suspend source git podinfo -n flux-system

# Edit the GitRepository resource for podinfo
kubectl edit gitrepository podinfo -n flux-system
```

Update the `spec.ref.branch` field:

```yaml
spec:
  ref:
    branch: flux-k8s-multi-env
```

Save the changes.

#### Resume the `podinfo` GitRepository:

```bash
flux resume source git podinfo -n flux-system
```

#### Reconcile the changes:

```bash
flux reconcile source git podinfo -n flux-system
```

This ensures that FluxCD is using the correct branch for `podinfo`.

---

### 4. **Update Kustomization Resources**

To apply both `default` and `prod` namespaces, create separate Kustomization resources. Ensure the `flux-system` deployment is updated accordingly.

#### `clusters/my-cluster/podinfo-default-kustomisation.yaml`
```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: podinfo-default
  namespace: flux-system
spec:
  interval: 5m0s
  path: ./kustomize/overlays/default
  prune: true
  sourceRef:
    kind: GitRepository
    name: podinfo
  targetNamespace: default
```

#### `clusters/my-cluster/podinfo-prod-kustomisation.yaml`
```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: podinfo-prod
  namespace: flux-system
spec:
  interval: 5m0s
  path: ./kustomize/overlays/prod
  prune: true
  sourceRef:
    kind: GitRepository
    name: podinfo
  targetNamespace: prod
```

Save these files under the `clusters/my-cluster/` directory.

---

### 5. **Change FluxCD Synchronization Settings**

Ensure the `flux-system` synchronization configuration (`gotk-sync.yaml`) is also updated to reflect the new branch:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: flux-system
  namespace: flux-system
spec:
  chart:
    spec:
      chart: flux-system
      sourceRef:
        kind: GitRepository
        name: flux-system
  interval: 5m
  values:
    ref:
      branch: flux-k8s-multi-env
```

---

### 6. **Verify the Deployment**

Once all configurations are updated, you can check the FluxCD Kustomization and GitRepository resources to verify the changes.

```bash
# Check GitRepository sources
flux get sources git -n flux-system

# Check Kustomization resources
flux get kustomizations -A
```

Ensure that the `podinfo` repository is now deployed to both `default` and `prod` namespaces.

---

## **Conclusion**

By following the steps above, you successfully transitioned from a single `default` namespace to a multi-environment setup, supporting both the `default` and `prod` namespaces, using FluxCD with Kustomize.

---

### **Repository Setup**
For reference, the repository is configured as follows:

```plaintext
https://github.com/nanisravankumar/flux-cd.git
```
