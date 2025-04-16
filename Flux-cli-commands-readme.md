### FluxCD Commands and When to Use Them

Here’s a breakdown of the FluxCD commands, their purpose, and when to use them. These commands are typically used for troubleshooting, managing Git repositories, and ensuring that FluxCD is synchronizing correctly with your Kubernetes cluster.

---

### **1. Check GitRepository Sources**

#### Command:
```bash
flux get sources git -n flux-system
```

#### Purpose:
This command shows the current state of all the `GitRepository` resources in the `flux-system` namespace. It lists details such as the current revision, whether the source is suspended or ready, and any messages indicating the status of the Git repository source (i.e., the repository FluxCD is pulling resources from).

#### When to Use:
- **Troubleshooting**: To verify that FluxCD is correctly fetching the Git repository.
- **Check Status**: To confirm that FluxCD is using the correct branch (`flux-k8s-multi-env` in this case) for deployments.
- **Visibility into Repositories**: If there’s any issue with fetching updates from the Git repository, this command helps diagnose if the repository has the correct configuration and revision.

---

### **2. Check Kustomization Resources**

#### Command:
```bash
flux get kustomizations -A
```

#### Purpose:
This command shows all the `Kustomization` resources in your cluster across all namespaces. It provides the current state of each `Kustomization`, including whether it is ready, whether it's suspended, and any relevant messages.

#### When to Use:
- **Troubleshooting**: To check if the `Kustomization` resources are correctly applied and synchronized with your Git repository.
- **Debugging**: If there are issues applying resources (e.g., resources not being deployed to the correct namespace), this command can help verify the status of the `Kustomization` resources.

---

### **3. Suspend GitRepository Source**

#### Command:
```bash
flux suspend source git podinfo -n flux-system
```

#### Purpose:
This command suspends the `GitRepository` source (in this case, the `podinfo` repository), which means FluxCD will no longer pull updates from the Git repository until resumed.

#### When to Use:
- **Maintenance or Troubleshooting**: If you are making updates to your Git repository and want to prevent FluxCD from pulling updates while you are working on it.
- **Prevent Automatic Reconciliation**: If you need to make changes to your configuration but don’t want FluxCD to overwrite or apply changes automatically.

---

### **4. Resume GitRepository Source**

#### Command:
```bash
flux resume source git flux-system -n flux-system
```

#### Purpose:
This command resumes the `GitRepository` source (in this case, `flux-system`), which means FluxCD will resume fetching updates from the Git repository.

#### When to Use:
- **After Editing or Pausing**: Once you have made updates to your Git repository source (e.g., changing the branch or fixing repository settings), you should resume it to allow FluxCD to start pulling updates again.
- **Resuming After a Pause**: If you suspended the repository for troubleshooting, you can use this command to resume normal synchronization.

---

### **5. Reconcile GitRepository Source**

#### Command:
```bash
flux reconcile source git flux-system -n flux-system
```

#### Purpose:
This command manually triggers FluxCD to fetch the latest changes from the Git repository and apply them to the cluster, regardless of the usual reconciliation interval.

#### When to Use:
- **Manual Trigger**: If you made changes to the Git repository (such as switching branches) and need FluxCD to immediately apply those changes.
- **Troubleshooting**: If the Git repository is not automatically synchronizing, or if you’ve made critical updates that you want to ensure are immediately applied.

---

### **6. Viewing GitHub Personal Access Token in Base64 Format**

To view the **GitHub Personal Access Token (PAT)** used by FluxCD (or any Kubernetes service account or secret), you can query the base64-encoded value of the secret that contains the token. FluxCD typically stores this token in a Kubernetes secret.

#### Command:
```bash
kubectl get secret flux-system -n flux-system -o jsonpath='{.data.GitRepository}' | base64 --decode
```

#### Purpose:
This command fetches the value of the `GitRepository` secret in the `flux-system` namespace and decodes it from base64 into a human-readable format.

#### When to Use:
- **Troubleshooting**: If you suspect there is an issue with the GitHub authentication token (e.g., expired or incorrect token), you can use this command to view the token and validate it.
- **Verifying Configuration**: If FluxCD is unable to access the GitHub repository, you can check the stored token for errors or expiration.

---

### **Additional Troubleshooting Tips**

- **Check Flux Logs**:
  If something isn’t working as expected, you can inspect the Flux logs to get more detailed information on what's happening:
  
  ```bash
  kubectl logs -n flux-system -l app=fluxcd
  ```

  This will show the logs from the FluxCD controller, which can help diagnose issues related to Git repository synchronization or Kubernetes resource deployments.

- **Check Kustomization Failures**:
  If there are issues applying Kustomizations, you can inspect the logs for the Kustomization controller:

  ```bash
  kubectl logs -n flux-system -l app.kubernetes.io/name=kustomize-controller
  ```

  This will show logs specifically related to the Kustomization process, which can help you troubleshoot errors in applying resources or building manifests.

---

### **Summary of Commands**:

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `flux get sources git -n flux-system` | Check GitRepository source status | To verify the repository status and configuration |
| `flux get kustomizations -A` | Check Kustomization resources | To verify the status of deployments and reconciliations |
| `flux suspend source git podinfo -n flux-system` | Suspend the GitRepository source | When making updates and preventing automatic pulls |
| `flux resume source git flux-system -n flux-system` | Resume the GitRepository source | After making changes to the source or pausing it |
| `flux reconcile source git flux-system -n flux-system` | Manually trigger reconciliation | To immediately fetch updates from the Git repository |
| `kubectl get secret flux-system -n flux-system -o jsonpath='{.data.GitRepository}' | base64 --decode` | View GitHub PAT in base64 format | If troubleshooting GitHub authentication issues |

---

By using these commands, you can manage and troubleshoot your FluxCD resources, ensuring that the correct configurations are being applied to your Kubernetes cluster.
