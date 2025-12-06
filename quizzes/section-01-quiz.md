# Section 1 Quiz: Welcome & GitOps Foundations

Test your understanding of GitOps principles and Argo CD basics.

---

## Question 1: What is GitOps?

**A)** A new Git hosting service like GitHub  
**B)** An operational framework using Git as single source of truth for declarative infrastructure  
**C)** A tool for managing Git repositories  
**D)** A continuous integration tool  

<details>
<summary>Click to reveal answer</summary>

**Answer: B**

GitOps is an operational framework that uses Git as the single source of truth for declarative infrastructure and applications. It applies DevOps best practices (version control, collaboration, compliance, CI/CD) to infrastructure automation.

</details>

---

## Question 2: Which of the following is NOT one of the four core GitOps principles?

**A)** Declarative configuration  
**B)** Automatically pushed from CI/CD  
**C)** Versioned and immutable  
**D)** Continuously reconciled  

<details>
<summary>Click to reveal answer</summary>

**Answer: B**

The four core GitOps principles are:
1. Declarative
2. Versioned and immutable
3. **Automatically pulled** (not pushed)
4. Continuously reconciled

GitOps uses a pull model where the cluster pulls changes from Git, not a push model where CI/CD pushes to the cluster.

</details>

---

## Question 3: What is the main advantage of Argo CD's pull model over traditional push-based CD?

**A)** It's faster to deploy  
**B)** It requires fewer Git commits  
**C)** No cluster credentials needed outside the cluster (better security)  
**D)** It works without Git  

<details>
<summary>Click to reveal answer</summary>

**Answer: C**

Argo CD runs inside the cluster and pulls changes from Git, so cluster credentials never need to leave the cluster. With traditional push-based CD, the CI/CD system needs cluster credentials to push changes, which is a security risk.

</details>

---

## Question 4: In GitOps, how should you rollback a deployment?

**A)** Run kubectl rollout undo  
**B)** Revert the Git commit and push  
**C)** Manually edit the deployment in the cluster  
**D)** Rebuild the previous Docker image  

<details>
<summary>Click to reveal answer</summary>

**Answer: B**

In GitOps, Git is the source of truth. To rollback, you revert the Git commit and push. Argo CD will automatically sync the cluster to match Git. You can also use `argocd app rollback` which essentially does the same thing.

</details>

---

## Question 5: What happens if someone makes a manual change to a deployment using `kubectl edit` when Argo CD has `selfHeal: true` enabled?

**A)** The change is permanent  
**B)** Argo CD shows a warning but keeps the change  
**C)** Argo CD automatically reverts the change to match Git  
**D)** The cluster crashes  

<details>
<summary>Click to reveal answer</summary>

**Answer: C**

With `selfHeal: true`, Argo CD continuously reconciles the cluster state with Git. If it detects drift (like a manual change), it automatically reverts the change to match what's in Git. This ensures the cluster always reflects the desired state in Git.

</details>

---

## Scoring

- **5 correct**: Excellent! You understand GitOps fundamentals.
- **4 correct**: Good job! Review the missed topics.
- **3 correct**: You're getting there. Re-read the lectures on missed topics.
- **< 3 correct**: Review Section 1 lectures again before proceeding.

---

**Ready for more?** Continue to [Section 2: Argo CD Architecture & Concepts](../sections/section-02/README.md)
