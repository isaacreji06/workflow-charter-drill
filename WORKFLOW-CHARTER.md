# Official Engineering Workflow Charter

This Workflow Charter defines the mandatory operational policies, validation gates, release practices, and rollback protocols for the `workflow-charter-drill` project. Compliance with this charter is required for all software contributors, team leads, and administrators.

---

## 1. Automated Validation Gates

Validation gates are automated quality checks executed via continuous integration (CI) prior to merging code into `main`. Every Pull Request must pass all validation gates cleanly.

### Gate 1: Code Linting & Syntax Gate
* **What It Checks**: Verifies JavaScript formatting standards, syntax correctness, unhandled variables, unused dependencies, and code style compliance.
* **What Failure Means**: Merging is immediately blocked. The PR cannot be merged until code style errors and syntax violations are corrected.
* **Configuration Location**: `.github/workflows/ci.yml`

### Gate 2: Automated Unit & Integration Test Gate
* **What It Checks**: Executes the automated test suite across core modules (`src/auth.js`, `src/payments.js`, `src/app.js`) to verify business logic correctness and API response contracts.
* **What Failure Means**: Merging is immediately blocked. Broken unit or integration tests indicate functional regression.
* **Configuration Location**: `.github/workflows/ci.yml`

### Gate 3: Dependency Security & Vulnerability Gate
* **What It Checks**: Runs security audit scans (`npm audit`) to identify known Common Vulnerabilities and Exposures (CVEs) in project dependencies and detects leaked secrets.
* **What Failure Means**: Merging is immediately blocked. High or critical vulnerabilities must be remediated before the PR can be accepted.
* **Configuration Location**: `.github/workflows/ci.yml`

### Gate 4: Service Startup & Health Verification Gate
* **What It Checks**: Launches the Express server instance in an isolated CI container and verifies that the `/status` health endpoint responds with an HTTP `200 OK` status.
* **What Failure Means**: Merging is immediately blocked. Indicates that the application fails to boot or crashes during startup.
* **Configuration Location**: `.github/workflows/ci.yml`

---

### Non-Negotiable Rule on Bypassing Gates
> **STRICT RULE**: Under no circumstances—including tight deadline pressure, executive escalation, or production emergencies—may any engineer, repository administrator, or manager bypass automated validation gates or force-merge unverified code into `main`. 
> 
> Production emergencies must follow the expedited **Hotfix Process** described in this charter. Hotfixes receive priority review and accelerated testing, but passing all automated validation gates remains mandatory.

---

## 2. Release Practices

### Semantic Versioning Format
The team uses **Semantic Versioning (SemVer)** formatted as `vMAJOR.MINOR.PATCH` (e.g., `v1.0.0`):
* **MAJOR** (`v1.0.0` -> `v2.0.0`): Incremented for incompatible API changes or breaking structural updates.
* **MINOR** (`v1.0.0` -> `v1.1.0`): Incremented for new, backward-compatible features.
* **PATCH** (`v1.0.0` -> `v1.0.1`): Incremented for backward-compatible bug fixes or emergency hotfixes.

### Git Tagging Command & Example
Releases are marked directly in Git using annotated tags.

```bash
# Tagging command syntax
git tag -a v1.0.0 -m "Release v1.0.0: Initial stable production release with auth, payments, and workflow charter governance"

# Push tag to remote repository
git push origin v1.0.0
```

### Pre-Release Verification Checklist
Before creating and pushing an official release tag, the following conditions must be verified:
- [ ] All feature PRs targeting the release are merged into `main` with passing CI gates and CODEOWNERS approvals.
- [ ] The `main` branch build and complete automated test suite pass without warnings or failures.
- [ ] Documentation (`README.md`, `BRANCHING.md`, `WORKFLOW-CHARTER.md`) and version numbers in `package.json` are updated.
- [ ] Staging environment smoke tests confirm stability under production-like configurations.
- [ ] Dependency security audits show zero unmitigated high or critical security advisories.

### Hotfix Release Process
When a critical production bug requires an out-of-band release outside the normal release cycle:
1. **Branch Creation**: Create a hotfix branch directly from the current production tag or updated `main`:
   ```bash
   git checkout main
   git pull origin main
   git checkout -b hotfix/payment-tax-rounding
   ```
2. **Implementation & Testing**: Apply the minimal fix and verify locally with automated unit tests.
3. **Expedited Review**: Open a PR to `main` with the `hotfix/` prefix. Notify CODEOWNERS for priority review.
4. **Automated Validation**: All CI validation gates must pass successfully.
5. **Merge & Patch Release**: Merge into `main` and immediately issue a new PATCH release tag (e.g., `v1.0.1`):
   ```bash
   git tag -a v1.0.1 -m "Release v1.0.1: Emergency hotfix for payment tax calculation rounding issue"
   git push origin v1.0.1
   ```

---

## 3. Rollback Readiness

When production failures or regressions occur post-deployment, the engineering team utilizes two distinct rollback mechanisms based on severity and scope.

### Method 1: `git revert` (For Shared Branch & Isolated Bad Commits)
* **When It Applies**: Used when an isolated bug or problematic PR was merged into `main`, but the production environment remains operational and overall history must preserve an audit trail.
* **First Command Engineer Runs**:
  ```bash
  # Identify the bad commit hash, then execute git revert
  git revert -m 1 <merge-commit-hash>
  ```
  *(Example for non-merge commit: `git revert 8db72f8`)*

---

### Method 2: Tag-Based Redeployment (For Widespread Production Outages)
* **When It Applies**: Used during severe production outages, widespread application crashes, or database/API failures where rolling back single commits via Git is too slow or risky. Requires restoring immediate production stability by redeploying a known good release tag.
* **First Command Engineer Runs**:
  ```bash
  # Checkout the latest known stable release tag locally to inspect and trigger redeployment
  git checkout tags/v1.0.0
  ```
  *(In cloud CI/CD deployment pipelines, the engineer executes the redeploy command pointing to the last verified release tag: `gh workflow run deploy.yml -f release_tag=v1.0.0` or redeploys `v1.0.0` from the deployment control panel).*
