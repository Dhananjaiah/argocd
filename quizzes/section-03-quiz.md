# Section 3 Quiz: Lab Setup (Local + Tooling)

Test your understanding of the tools and setup required for Argo CD.

---

## Question 1: What is Kind?

**A)** A Kubernetes distribution for production  
**B)** A tool to run local Kubernetes clusters using Docker containers  
**C)** A Kubernetes monitoring tool  
**D)** A CI/CD platform  

<details>
<summary>Click to reveal answer</summary>

**Answer: B - A tool to run local Kubernetes clusters using Docker containers**

Kind (Kubernetes IN Docker) runs Kubernetes clusters in Docker containers. It's perfect for:
- Local development and testing
- CI/CD environments
- Learning Kubernetes and Argo CD
- Quick cluster creation and deletion

Kind creates a control plane node (and optionally worker nodes) as Docker containers, providing a full Kubernetes cluster for testing.

</details>

---

## Question 2: What is the primary purpose of kubectl?

**A)** To build Docker images  
**B)** To run GitOps workflows  
**C)** To interact with Kubernetes clusters  
**D)** To manage Git repositories  

<details>
<summary>Click to reveal answer</summary>

**Answer: C - To interact with Kubernetes clusters**

kubectl is the Kubernetes command-line tool that allows you to:
- Deploy applications (`kubectl apply`)
- Inspect resources (`kubectl get`, `kubectl describe`)
- View logs (`kubectl logs`)
- Execute commands in containers (`kubectl exec`)
- Port forward to services (`kubectl port-forward`)
- Manage cluster resources

It's the primary way to interact with any Kubernetes cluster.

</details>

---

## Question 3: What command accesses the Argo CD UI locally?

**A)** `argocd ui`  
**B)** `kubectl port-forward svc/argocd-server -n argocd 8080:443`  
**C)** `argocd open`  
**D)** `kubectl expose argocd`  

<details>
<summary>Click to reveal answer</summary>

**Answer: B - `kubectl port-forward svc/argocd-server -n argocd 8080:443`**

This command:
- Creates a tunnel from your local machine to the Argo CD server service
- Maps local port 8080 to service port 443 (HTTPS)
- Allows you to access the UI at https://localhost:8080

Port forwarding is the standard way to access services in a local Kubernetes cluster without setting up an Ingress controller.

</details>

---

## Question 4: Where is the initial Argo CD admin password stored?

**A)** In a file named `password.txt`  
**B)** In a Kubernetes Secret named `argocd-initial-admin-secret`  
**C)** In the Argo CD ConfigMap  
**D)** It's "admin" by default  

<details>
<summary>Click to reveal answer</summary>

**Answer: B - In a Kubernetes Secret named `argocd-initial-admin-secret`**

The initial admin password is randomly generated during installation and stored in a Secret:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

This ensures the password is secure and unique per installation. You should change it after first login and then delete the secret.

</details>

---

## Question 5: What is Kustomize used for?

**A)** Building Docker images  
**B)** Managing Kubernetes configuration files without templates  
**C)** Running CI/CD pipelines  
**D)** Monitoring applications  

<details>
<summary>Click to reveal answer</summary>

**Answer: B - Managing Kubernetes configuration files without templates**

Kustomize allows you to:
- Define a base set of Kubernetes manifests
- Create overlays for different environments (dev/staging/prod)
- Patch configurations without duplicating entire files
- Manage configurations using pure YAML (no templating language)

Example structure:
```
base/
  deployment.yaml
overlays/
  dev/
    kustomization.yaml
  prod/
    kustomization.yaml
```

Each overlay can modify the base manifests for environment-specific needs without duplicating the entire configuration.

</details>

---

## Scoring

- **5 correct**: Excellent! You're ready for hands-on labs.
- **4 correct**: Good job! Review the missed topics.
- **3 correct**: You're getting there. Re-read the lectures on missed topics.
- **< 3 correct**: Review Section 3 lectures again before proceeding.

---

**Ready to get hands-on?** Continue to [Lab 1: Setting Up Your Argo CD Environment](../labs/lab-01.md)!
