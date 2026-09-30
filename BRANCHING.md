# Branching Strategy & Lifecycle Governance

This document establishes the official branching strategy, branch protection rules, lifecycle procedures, and synchronization policies for the `workflow-charter-drill` repository.

---

## 1. Branch Naming Convention

All branches created in this repository must use a mandatory prefix followed by a concise, kebab-case description (and optional issue reference).

| Prefix | Description | Example Branch Name |
| :--- | :--- | :--- |
| `feature/` | New product functionality or enhancement | `feature/auth-jwt-refresh` |
| `bugfix/` | Resolving bugs identified during development/QA | `bugfix/payment-tax-rounding` |
| `hotfix/` | Emergency fixes required directly in production | `hotfix/stripe-webhook-timeout` |
| `release/` | Preparing and validating a new version release candidate | `release/v1.2.0` |
| `chore/` | Refactoring, dependency updates, build/CI configuration | `chore/update-express-deps` |

---

## 2. Protected Branches

### Protected Branch: `main`
The `main` branch represents the production-ready source of truth. Direct pushes and force pushes to `main` are strictly prohibited.

### Required Protection Settings & Rationale

1. **Require Pull Request Before Merging**:
   * *Setting*: Minimum 1 peer review approval required.
   * *Rationale*: Prevents unreviewed code from entering production and ensures peer feedback on all changes.
2. **Require CODEOWNERS Approval**:
   * *Setting*: Mandatory review from assigned code owners for domain-specific paths (`src/payments.js`, `src/auth.js`, `.github/workflows/`).
   * *Rationale*: Guarantees that domain experts (Fintech, Security, DevOps leads) inspect high-risk modules before changes are merged.
3. **Require Status Checks to Pass Before Merging**:
   * *Setting*: CI build, test suite execution, and security scans must succeed (`CI Pipeline`).
   * *Rationale*: Ensures that broken, unformatted, or vulnerable code cannot be merged into `main`.
4. **Require Branches to be Up-to-Date Before Merging**:
   * *Setting*: Enforce branch synchronization with `main` prior to merging.
   * *Rationale*: Prevents post-merge conflicts and ensures tests validate the exact code state that will exist after merge.
5. **Restrict Direct Pushes & Force Pushes**:
   * *Setting*: Enforce rules for all contributors, including administrators.
   * *Rationale*: Protects Git history integrity and prevents accidental overwrite of production code.

---

## 3. Branch Lifecycle

The complete lifecycle of a branch follows these standardized steps:

1. **Branch Creation**:
   * Create the branch from the latest `main`:
     ```bash
     git checkout main
     git pull origin main
     git checkout -b feature/auth-jwt-refresh
     ```
2. **Local Development & Commit Standards**:
   * Work locally using atomic commits following Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`).
3. **Branch Synchronization**:
   * Continuously rebase onto updated `main` to avoid drift.
4. **Pull Request Submission**:
   * Open a PR using `.github/PULL_REQUEST_TEMPLATE.md` once development and local testing are complete.
5. **Review & Validation**:
   * Automated CI gates execute; required peer and CODEOWNERS approvals are obtained.
6. **Squash and Merge**:
   * Merge into `main` using **Squash and Merge** to maintain a clean, linear commit history on `main`.
7. **Branch Deletion**:
   * Delete the feature branch immediately after merge both on GitHub and locally:
     ```bash
     git branch -d feature/auth-jwt-refresh
     git push origin --delete feature/auth-jwt-refresh
     ```

### Maximum Branch Lifetime
* **Maximum Allowed Age**: Feature and bugfix branches must not remain open for more than **3 days**.
* *Rationale*: Long-lived branches lead to severe merge conflicts, stale code bases, and integration bottlenecks.

---

## 4. Sync Policy

### Sync Frequency
* Developers must sync their working branch with `main` **at least once daily** and **immediately before opening or requesting final review on a Pull Request**.

### Mandatory Sync Commands
To maintain a clean linear history, developers must use `git rebase` (or `git pull --rebase`):

```bash
# Fetch latest remote changes
git fetch origin main

# Rebase local working branch onto main
git rebase origin/main
```

*Note: If merge conflicts occur during rebase, resolve them locally, test the code, and continue the rebase using `git rebase --continue`.*
