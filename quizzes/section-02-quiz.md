# Section 2 Quiz: Argo CD Architecture & Concepts

Test your understanding of Argo CD's architecture and core concepts.

---

## Question 1: Which Argo CD component is responsible for monitoring Git repositories?

**A)** API Server  
**B)** Repository Server  
**C)** Application Controller  
**D)** Redis  

<details>
<summary>Click to reveal answer</summary>

**Answer: B - Repository Server**

The Repository Server is responsible for:
- Cloning and monitoring Git repositories
- Generating Kubernetes manifests from Git (Helm, Kustomize, plain YAML)
- Caching repository data
- Providing manifests to the Application Controller

</details>

---

## Question 2: What is the difference between "Sync Status" and "Health Status"?

**A)** They are the same thing  
**B)** Sync Status compares Git vs cluster; Health Status indicates if resources are functioning  
**C)** Health Status is for Git; Sync Status is for cluster  
**D)** Only Health Status matters  

<details>
<summary>Click to reveal answer</summary>

**Answer: B - Sync Status compares Git vs cluster; Health Status indicates if resources are functioning**

**Sync Status** indicates whether the live state in the cluster matches the desired state in Git:
- Synced: Cluster matches Git
- OutOfSync: Cluster differs from Git

**Health Status** indicates whether the resources are functioning correctly:
- Healthy: Resources are running properly
- Progressing: Resources are being created/updated
- Degraded: Resources have issues
- Missing: Resources don't exist

An app can be Synced but Unhealthy (deployed but not working), or OutOfSync but Healthy (manual changes made but app still running).

</details>

---

## Question 3: What is an Argo CD Application?

**A)** The actual application code (frontend/backend)  
**B)** A CRD that defines what to deploy, where, and how  
**C)** A Docker container  
**D)** A Kubernetes namespace  

<details>
<summary>Click to reveal answer</summary>

**Answer: B - A CRD that defines what to deploy, where, and how**

An Argo CD Application is a Custom Resource Definition (CRD) that declares:
- **Source**: Git repository URL, path, and branch/tag
- **Destination**: Target Kubernetes cluster and namespace
- **Sync Policy**: How to synchronize (manual/auto, prune, self-heal)
- **Project**: Which Argo CD Project it belongs to

Example:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    repoURL: https://github.com/myorg/myapp
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

</details>

---

## Question 4: What is the purpose of Argo CD Projects?

**A)** To store application code  
**B)** To provide multi-tenancy and boundaries for applications  
**C)** To run CI/CD pipelines  
**D)** To manage Docker images  

<details>
<summary>Click to reveal answer</summary>

**Answer: B - To provide multi-tenancy and boundaries for applications**

Argo CD Projects provide:
- **Multi-tenancy**: Separate teams can manage their own apps
- **Repository restrictions**: Limit which Git repos can be used
- **Cluster restrictions**: Limit which clusters can be targeted
- **Namespace restrictions**: Limit which namespaces can be deployed to
- **RBAC boundaries**: Control who can access which applications

Example use case: "team-a" project can only deploy to "team-a-*" namespaces from "github.com/team-a/*" repositories.

</details>

---

## Question 5: What happens when Argo CD detects drift (desired state ≠ live state)?

**A)** Argo CD automatically crashes  
**B)** Argo CD shows OutOfSync status; may auto-sync if configured  
**C)** Argo CD ignores it  
**D)** Kubernetes automatically fixes it  

<details>
<summary>Click to reveal answer</summary>

**Answer: B - Argo CD shows OutOfSync status; may auto-sync if configured**

When drift is detected:

**Without automated sync**:
1. Argo CD marks the application as "OutOfSync"
2. UI shows the difference between Git and cluster
3. Manual sync is required to reconcile

**With automated sync**:
1. Argo CD marks the application as "OutOfSync"
2. Argo CD automatically syncs to match Git
3. Application returns to "Synced" status

**With self-heal enabled**:
1. Argo CD detects manual changes in cluster
2. Automatically reverts changes to match Git
3. Prevents configuration drift

This is the core of GitOps: Git is always the source of truth.

</details>

---

## Scoring

- **5 correct**: Excellent! You understand Argo CD architecture.
- **4 correct**: Good job! Review the missed topics.
- **3 correct**: You're getting there. Re-read the lectures on missed topics.
- **< 3 correct**: Review Section 2 lectures again before proceeding.

---

**Ready for more?** Continue to [Section 3: Lab Setup](../sections/section-03/README.md) to set up your environment!
