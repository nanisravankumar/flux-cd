Here’s a well-organized, step-by-step `README.md` file for your PoC using Flux CD with GitHub and Kubernetes:

---

# 🚀 FluxCD PoC with GitHub and AKS

This Proof of Concept demonstrates how to set up [Flux CD](https://fluxcd.io) with GitHub for GitOps-based Kubernetes deployments.  
We will be installing necessary tools, configuring GitHub integration, and deploying a sample application.

---

## 🔧 Step 1: Install Chocolatey (User/Portable Mode)

1. Visit: [https://docs.chocolatey.org/en-us/choco/setup/#non-administrative-install](https://docs.chocolatey.org/en-us/choco/setup/#non-administrative-install)

2. Open PowerShell **as current user (non-admin)**.

3. Run the following to install Chocolatey:

   ```powershell
   $InstallDir='C:\ProgramData\chocoportable'
   $env:ChocolateyInstall="$InstallDir"
   Set-ExecutionPolicy Bypass -Scope Process -Force;
   iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
   ```

---

## 🔧 Step 2: Install Flux CLI Using Chocolatey

```powershell
choco install flux
```

✅ To verify installation:

```bash
flux --version
```

---

## 🔧 Step 3: Install Azure CLI and Required Extensions

```bash
az aks install-cli

az provider register --namespace Microsoft.Kubernetes
az provider register --namespace Microsoft.ContainerService
az provider register --namespace Microsoft.KubernetesConfiguration

az provider show -n Microsoft.KubernetesConfiguration -o table

az extension add -n k8s-configuration
az extension add -n k8s-extension

az extension update -n k8s-configuration
az extension update -n k8s-extension

az extension list -o table
```

---

## 🔐 Step 4: Create GitHub Personal Access Token (PAT)

1. Go to: [https://github.com/settings/tokens](https://github.com/settings/tokens)
2. Click on **"Generate new token"** (classic).
3. Set:
   - **Note:** flux
   - **Expiration:** Choose appropriate
   - **Scopes:**  
     ✅ `repo` – Full control of private repositories  
     ✅ `workflow` – Update GitHub Actions workflows  
4. Click **Generate token** and **copy the token**.

---

## 📦 Step 5: Prepare GitHub Repository

1. Create a repository:  
   👉 [https://github.com/nanisravankumar/flux-cd.git](https://github.com/nanisravankumar/flux-cd.git)

2. Clone the repo locally:

   ```bash
   git clone https://github.com/nanisravankumar/flux-cd.git
   cd flux-cd
   ```

3. Create folder structure:

   ```bash
   mkdir -p clusters/my-cluster/flux-system
   cd clusters/my-cluster/flux-system
   touch gotk-components.yaml gotk-sync.yaml kustomization.yaml
   ```

4. Add the following content to `kustomization.yaml`:

   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   resources:
   - gotk-components.yaml
   - gotk-sync.yaml
   ```

5. Commit and push:

   ```bash
   git add .
   git commit -m "Initial commit for flux-system"
   git push origin main
   ```

---

## 🚀 Step 6: Bootstrap FluxCD with GitHub

```bash
export GITLAB_TOKEN=#token value

flux bootstrap github \
  --owner=nanisravankumar \
  --repository=flux-cd \
  --branch=main \
  --path=clusters/my-cluster \
  --personal
```

---

## 🧱 Step 7: Create Sample Application (Podinfo)

1. Create the following structure:

   ```
   flux-cd/
   ├── clusters/my-cluster/
   │   ├── podinfo-kustomisation.yaml
   │   └── podinfo-repo.yaml
   └── kustomize/
       ├── deployment.yaml
       ├── service.yaml
       └── kustomization.yaml
   ```

2. `clusters/my-cluster/podinfo-repo.yaml`:

   ```yaml
   apiVersion: source.toolkit.fluxcd.io/v1
   kind: GitRepository
   metadata:
     name: podinfo
     namespace: flux-system
   spec:
     interval: 1m0s
     ref:
       branch: master
     secretRef:
       name: flux-system
     url: https://github.com/nanisravankumar/flux-cd.git
   ```

3. `clusters/my-cluster/podinfo-kustomisation.yaml`:

   ```yaml
   apiVersion: kustomize.toolkit.fluxcd.io/v1
   kind: Kustomization
   metadata:
     name: podinfo
     namespace: flux-system
   spec: 
     interval: 5m0s
     path: ../../kustomize/
     prune: true
     sourceRef:
       kind: GitRepository
       name: podinfo
     targetNamespace: default
   ```

4. `kustomize/deployment.yaml`:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: podinfo
     labels:
       app: podinfo
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: podinfo
     template:
       metadata:
         labels:
           app: podinfo
       spec:
         containers:
           - name: podinfo
             image: nginx:latest
             ports:
               - containerPort: 80
   ```

5. `kustomize/service.yaml`:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: podinfo
     labels:
       app: podinfo
   spec:
     selector:
       app: podinfo
     ports:
       - port: 80
         targetPort: 80
     type: LoadBalancer
   ```

6. `kustomize/kustomization.yaml`:

   ```yaml
   resources:
     - deployment.yaml
     - service.yaml
   ```

7. Commit and push changes:

   ```bash
   git add .
   git commit -m "Added podinfo GitRepository and Kustomization"
   git push
   ```

---

## ✅ Verify Deployment

```bash
flux get kustomizations --watch
kubectl get pods
```

🔁 To access the app (once deployed):

```bash
kubectl get svc
kubectl port-forward pod/<podinfo-pod-name> 9898:80 --address 0.0.0.0
```

---

## 🧪 Debugging Commands

```bash
kubectl get gitrepo -n flux-system
kubectl get secrets -n flux-system
kubectl get secret flux-system -n flux-system -o jsonpath='{.data.password}' | base64 -d
kubectl get clusterrolebindings | grep flux
```
