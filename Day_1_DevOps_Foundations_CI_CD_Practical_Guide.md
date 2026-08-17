# Day 1 — DevOps Foundations + CI/CD

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** DevOps fundamentals, CI/CD, Git, practical implementation, and interview preparation  
> **Recommended time:** 3 hours  
> **Target:** ~6 years DevOps / Azure DevOps experience

---

## 1. Day 1 Objective

By the end of Day 1, you should be able to:

- Explain DevOps clearly in your own words.
- Explain the DevOps lifecycle.
- Explain CI, Continuous Delivery, and Continuous Deployment.
- Design and explain an enterprise CI/CD pipeline.
- Use important Git commands confidently.
- Create a feature branch and merge changes.
- Understand `merge` vs `rebase`.
- Understand `revert` vs `reset`.
- Understand `cherry-pick`, `stash`, and `reflog`.
- Create a basic CI pipeline.
- Explain how to secure a CI/CD pipeline.
- Troubleshoot a failed deployment.
- Answer senior-level DevOps interview questions.

---

# 2. Recommended 3-Hour Schedule

| Time | Duration | Topic | Expected Result |
|---|---:|---|---|
| 00:00–00:30 | 30 min | DevOps fundamentals | Explain DevOps lifecycle |
| 00:30–01:10 | 40 min | CI/CD deep dive | Explain CI/CD clearly |
| 01:10–01:50 | 40 min | Git practice | Branch, commit, merge, revert/reset |
| 01:50–02:30 | 40 min | Hands-on project | Build a small Git project |
| 02:30–03:00 | 30 min | Interview Q&A | Answer scenario questions |

---

# 3. DevOps Fundamentals

## 3.1 What is DevOps?

DevOps is a combination of:

- Culture
- Processes
- Automation
- Collaboration
- Tools
- Monitoring
- Continuous improvement

The main objective is to help development and operations teams deliver software:

- Faster
- More reliably
- More securely
- More consistently
- With less manual effort

### Simple explanation

```text
Development
     +
Operations
     +
Automation
     +
Monitoring
     +
Security
     ↓
Reliable Software Delivery
```

### Interview Answer

> DevOps is a combination of culture, practices, automation, and tools that improves collaboration between development and operations and enables fast, reliable, and repeatable software delivery.

---

# 4. Why Do Companies Use DevOps?

Traditional software delivery can look like:

```text
Developer
   ↓
Manual Build
   ↓
Manual Testing
   ↓
Manual Deployment
   ↓
Operations
```

This can cause:

- Human errors
- Slow releases
- Environment inconsistencies
- Difficult troubleshooting
- Long feedback cycles

DevOps introduces automation:

```text
Developer
   ↓
Git
   ↓
CI Pipeline
   ↓
Build
   ↓
Test
   ↓
Security Scan
   ↓
Artifact
   ↓
CD Pipeline
   ↓
Deployment
   ↓
Monitoring
```

---

# 5. DevOps Lifecycle

A common DevOps lifecycle is:

```text
Plan
  ↓
Code
  ↓
Build
  ↓
Test
  ↓
Release
  ↓
Deploy
  ↓
Operate
  ↓
Monitor
  ↓
Feedback
  └──────────────→ Plan
```

## 5.1 Plan

Activities:

- Requirement gathering
- Sprint planning
- Task creation
- Estimation
- Backlog management

Examples:

- Azure Boards
- Jira

## 5.2 Code

Developers write application code and commit it to Git.

Examples:

- GitHub
- Azure Repos
- GitLab
- Bitbucket

## 5.3 Build

The application is compiled or packaged.

Examples:

```text
Java       → JAR
.NET       → DLL / application package
Node.js    → application package
Python     → package / Docker image
```

## 5.4 Test

Typical tests:

- Unit tests
- Integration tests
- API tests
- Regression tests
- Smoke tests
- End-to-end tests

## 5.5 Release

The validated artifact is prepared for deployment.

```text
Build
  ↓
Artifact
  ↓
Release Candidate
```

## 5.6 Deploy

Typical environments:

```text
Development
    ↓
QA
    ↓
Staging
    ↓
Production
```

## 5.7 Operate

Operations manages:

- Infrastructure
- Applications
- Kubernetes
- Networking
- Databases
- Security
- Availability

## 5.8 Monitor

Monitor:

- CPU
- Memory
- Request rate
- Latency
- Error rate
- Availability
- Logs
- Business metrics

Common tools:

- Prometheus
- Grafana
- ELK
- Azure Monitor

---

# 6. SDLC vs DevOps

| Area | Traditional SDLC | DevOps |
|---|---|---|
| Development | Teams can work separately | Cross-functional collaboration |
| Testing | Often later | Continuous |
| Deployment | More manual | Highly automated |
| Infrastructure | Often manually managed | Infrastructure as Code |
| Feedback | Slow | Fast |
| Security | Often later | Integrated throughout |
| Monitoring | Operations-focused | Continuous feedback |
| Release frequency | Lower | Higher |

---

# 7. CI/CD Deep Dive

## 7.1 What is CI?

CI means **Continuous Integration**.

Developers frequently integrate code into a shared repository.

Every change can automatically trigger:

```text
Git Push / Pull Request
        ↓
Checkout
        ↓
Build
        ↓
Unit Test
        ↓
Code Quality
        ↓
Security Scan
        ↓
Package
        ↓
Artifact
```

### Example

```text
Developer
    ↓
git push
    ↓
Azure DevOps Pipeline
    ↓
Build
    ↓
Test
    ↓
SonarQube
    ↓
Trivy
    ↓
Docker Build
    ↓
Push to ACR
```

---

# 8. Continuous Delivery

Continuous Delivery means the application is kept in a **production-ready state**.

Example:

```text
Build
  ↓
Test
  ↓
Dev
  ↓
QA
  ↓
Staging
  ↓
Approval
  ↓
Production
```

Production deployment can require manual approval.

---

# 9. Continuous Deployment

Continuous Deployment automatically deploys successful changes to production.

```text
Git
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Deploy
 ↓
Production
```

There is no manual production approval when the organization's policy allows fully automated deployment.

---

# 10. CI vs Continuous Delivery vs Continuous Deployment

| Concept | Main Goal | Production |
|---|---|---|
| CI | Validate code changes | No |
| Continuous Delivery | Keep software release-ready | Usually approval |
| Continuous Deployment | Automatically release validated changes | Automatic |

### Easy way to remember

```text
CI
↓
Can we build and validate this?

Continuous Delivery
↓
Can we release this?

Continuous Deployment
↓
Can we automatically deploy this?
```

---

# 11. Enterprise CI/CD Pipeline

A practical enterprise pipeline:

```text
Developer
    ↓
Pull Request
    ↓
PR Validation
    ↓
Build
    ↓
Unit Tests
    ↓
SonarQube
    ↓
Security / Dependency Scan
    ↓
Trivy
    ↓
Docker Build
    ↓
Push Image to ACR
    ↓
Deploy to Dev
    ↓
Integration Tests
    ↓
Approval
    ↓
Production
    ↓
Monitoring
```

---

# 12. Important CI/CD Design Principles

## 12.1 Build Once, Deploy Many

Do not rebuild the application separately for every environment.

Prefer:

```text
Build
 ↓
Versioned Artifact
 ↓
Dev
 ↓
QA
 ↓
Staging
 ↓
Production
```

This ensures that the same tested artifact is promoted.

## 12.2 Secret Management

Never store:

```text
password=MyPassword123
```

inside source code or pipeline YAML.

Use:

- Azure Key Vault
- HashiCorp Vault
- Azure DevOps secret variables
- Managed Identity
- Workload Identity

## 12.3 Least Privilege

Pipeline identities should only have the permissions they need.

Example:

```text
Build Pipeline
     ↓
Read Source
     ↓
Build Image
     ↓
Push to ACR
```

Do not give unnecessary subscription-level permissions.

## 12.4 Quality Gates

A pipeline can stop when quality requirements are not met.

Examples:

- Unit test failure
- Critical vulnerability
- SonarQube quality gate failure
- Secret detected
- Docker image vulnerability

---

# 13. Git Practical Guide

## 13.1 Initialize a Repository

```bash
mkdir devops-30-days
cd devops-30-days

mkdir app
mkdir docs

git init
git status
```

---

# 14. Create the Application

Create:

```text
app/
└── hello.py
```

`hello.py`:

```python
print("Hello DevOps!")
```

---

# 15. First Git Commit

```bash
git add .

git commit -m "Initial DevOps project"

git branch -M main
```

Check:

```bash
git status
git log --oneline
```

---

# 16. Create a Feature Branch

```bash
git switch -c feature/day1
```

Or:

```bash
git checkout -b feature/day1
```

Modify:

```python
print("Hello from Day 1 DevOps!")
```

Check changes:

```bash
git diff
```

Commit:

```bash
git add .
git commit -m "Complete Day 1 exercise"
```

---

# 17. Merge the Feature

Switch to main:

```bash
git switch main
```

Merge:

```bash
git merge feature/day1
```

View history:

```bash
git log --oneline --graph --decorate --all
```

---

# 18. Important Git Commands

| Command | Purpose | Interview Note |
|---|---|---|
| `git fetch` | Download remote references | Does not integrate changes |
| `git pull` | Fetch + integrate | Usually fetch + merge/rebase |
| `git merge` | Combine histories | Normally preserves existing history |
| `git rebase` | Replay commits onto another base | Rewrites commit IDs |
| `git cherry-pick` | Apply one specific commit | Useful for targeted fixes |
| `git revert` | Reverse a commit with a new commit | Safer for shared branches |
| `git reset` | Move branch/HEAD | Can rewrite history |
| `git stash` | Temporarily save changes | Useful when changing tasks |
| `git reflog` | Show local reference movements | Useful for recovery |

---

# 19. Git Merge vs Rebase

## Merge

```text
A---B---C
     \
      D---E
           \
            M
```

Merge combines histories.

Advantages:

- Preserves branch history
- Safe for shared branches
- Easy to understand for many teams

## Rebase

```text
A---B---C---D---E
```

Rebase moves commits onto another base.

Advantages:

- Cleaner linear history
- Easier history navigation

Important:

> Avoid rebasing commits that other developers are already depending on unless your team explicitly follows a workflow that permits it.

---

# 20. Git Revert vs Reset

## Revert

```bash
git revert HEAD
```

Creates a new commit that reverses the previous commit.

Use it when:

- The commit is already shared.
- The branch is production-related.
- You want an auditable rollback.

## Reset

```bash
git reset --soft HEAD~1
```

or:

```bash
git reset --hard HEAD~1
```

Reset moves the branch reference.

`--hard` can discard working-tree changes.

Use carefully on shared branches.

---

# 21. Git Cherry-Pick

Suppose a production fix exists in another branch:

```bash
git log --oneline
```

Example:

```text
a12bc34 Fix authentication timeout
```

Apply it:

```bash
git cherry-pick a12bc34
```

Useful when you need one specific fix without merging the complete feature branch.

---

# 22. Git Stash

Temporarily save changes:

```bash
git stash
```

View:

```bash
git stash list
```

Restore:

```bash
git stash pop
```

Example:

```text
Working on Feature A
       ↓
Urgent production issue
       ↓
git stash
       ↓
Fix production issue
       ↓
Return to Feature A
       ↓
git stash pop
```

---

# 23. Git Reflog

`reflog` is one of the most useful recovery commands.

```bash
git reflog
```

It shows local movements of:

- HEAD
- Branch references

It can help recover from an accidental reset or rebase.

---

# 24. Day 1 Hands-on Lab

## Objective

Build a small Python application and practice:

- Git initialization
- Branching
- Commit
- Merge
- Revert
- Git history

## Step 1 — Verify Prerequisites

```bash
git --version
python --version
```

## Step 2 — Create Project

```bash
mkdir devops-day1
cd devops-day1

mkdir app
mkdir docs
```

## Step 3 — Create Application

Create:

```text
app/hello.py
```

Add:

```python
print("Hello DevOps!")
```

## Step 4 — Initialize Git

```bash
git init

git add .

git commit -m "Initial application"

git branch -M main
```

## Step 5 — Feature Development

```bash
git switch -c feature/greeting
```

Modify:

```python
print("Hello from Day 1 DevOps!")
```

Check:

```bash
git diff
```

Commit:

```bash
git add .
git commit -m "Improve greeting"
```

Merge:

```bash
git switch main
git merge feature/greeting
```

---

# 25. Simulate a Bad Change

Make an intentionally incorrect change.

Commit it:

```bash
git add .

git commit -m "Introduce bad change"
```

Check history:

```bash
git log --oneline -5
```

Safely reverse it:

```bash
git revert HEAD
```

Check:

```bash
git log --oneline -5
```

### What should you observe?

The original bad commit still exists in history.

A new commit reverses its changes.

This is why `git revert` is usually safer for shared branches.

---

# 26. Optional Azure DevOps CI Practice

Create a basic Azure DevOps pipeline:

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:

- checkout: self

- script: |
    python --version
    python app/hello.py
  displayName: "Run application"

- script: |
    python -m compileall app
  displayName: "Validate Python syntax"
```

Basic flow:

```text
Git Push
   ↓
Azure DevOps
   ↓
Pipeline Trigger
   ↓
Checkout
   ↓
Run Python
   ↓
Validate Syntax
```

The goal today is not to build a production-grade pipeline. The goal is to understand the path from Git commit → pipeline trigger → validation.

---

# 27. Interview Questions & Answers

## Q1. What is DevOps?

**Answer:**

> DevOps is a combination of culture, practices, automation, and tools that improves collaboration between development and operations and enables fast, reliable, and repeatable software delivery.

## Q2. Why is DevOps important?

**Answer:**

> DevOps reduces manual work, improves collaboration, shortens feedback cycles, increases deployment consistency, and helps teams release software faster and more reliably.

## Q3. What is Continuous Integration?

**Answer:**

> Continuous Integration means frequently integrating code changes into a shared repository and automatically validating those changes through build, unit testing, quality checks, and security checks.

## Q4. What is Continuous Delivery?

**Answer:**

> Continuous Delivery keeps validated software in a production-ready state. Production deployment can still require a manual approval.

## Q5. What is Continuous Deployment?

**Answer:**

> Continuous Deployment automatically deploys validated changes through the deployment pipeline, including production when the organization allows fully automated production releases.

## Q6. CI vs CD?

**Answer:**

> CI focuses on validating source-code changes and producing trusted artifacts. CD focuses on taking those validated artifacts through deployment environments.

## Q7. What is a CI/CD pipeline?

**Answer:**

> A CI/CD pipeline is an automated workflow that takes source code through stages such as build, testing, security and quality checks, artifact creation, and deployment.

## Q8. What happens after a developer pushes code?

**Answer:**

```text
Git Push
   ↓
Pipeline Trigger
   ↓
Checkout
   ↓
Build
   ↓
Unit Tests
   ↓
Quality Scan
   ↓
Security Scan
   ↓
Artifact/Image
   ↓
Deployment
```

## Q9. What is an artifact?

**Answer:**

> An artifact is a versioned output generated by a build, such as a JAR, ZIP, NuGet package, npm package, or container image.

## Q10. Why should we build once and promote the same artifact?

**Answer:**

> It prevents differences caused by rebuilding the application separately for each environment. The exact artifact that was tested is promoted through Dev, QA, staging, and production.

## Q11. What is a quality gate?

**Answer:**

> A quality gate is a set of rules that determines whether a pipeline can continue. Examples include test failures, critical vulnerabilities, code-quality thresholds, or SonarQube quality-gate status.

## Q12. How do you secure a CI/CD pipeline?

**Answer:**

> I use least-privilege identities, protected branches, approvals, centralized secret management, code and dependency scanning, container scanning, secure service connections, and audit logging.

## Q13. Where should secrets be stored?

**Answer:**

> Secrets should not be hard-coded in source code or pipeline YAML. They should be stored in services such as Azure Key Vault, HashiCorp Vault, or the CI/CD platform's secure secret store.

## Q14. What is Git?

**Answer:**

> Git is a distributed version-control system used to track changes, create branches, collaborate, merge changes, and recover previous versions of code.

## Q15. `git fetch` vs `git pull`?

**Answer:**

> `git fetch` downloads remote references without integrating them into the current branch. `git pull` normally performs a fetch followed by an integration operation such as merge or rebase.

## Q16. Merge vs Rebase?

**Answer:**

> Merge combines histories and preserves the branch structure. Rebase replays commits onto another base and creates a more linear history, but it rewrites commit identities.

## Q17. Revert vs Reset?

**Answer:**

> Revert creates a new commit that reverses an earlier commit and is safer for shared branches. Reset moves the branch reference and can rewrite history, while `--hard` can discard local changes.

## Q18. What is cherry-pick?

**Answer:**

> Cherry-pick applies the changes from a specific commit onto the current branch. It is useful when I need a targeted fix without merging an entire branch.

## Q19. Production deployment failed. What do you do?

**Answer:**

```text
Assess Impact
     ↓
Stop Further Rollout
     ↓
Check Pipeline Logs
     ↓
Check Application Logs
     ↓
Identify Root Cause
     ↓
Rollback / Roll Forward
     ↓
Validate Health
     ↓
Monitor
     ↓
RCA
```

A strong answer:

> First I assess the impact and stop further rollout if necessary. Then I inspect the pipeline and application logs to identify the failing stage. If the previous version is known-good and rollback is safe, I restore it. After service recovery, I validate application health, monitor the system, and perform root-cause analysis.

## Q20. How would you design CI/CD for microservices?

**Answer:**

```text
Git
 ↓
PR Validation
 ↓
Build
 ↓
Unit Tests
 ↓
SonarQube
 ↓
Security Scan
 ↓
Docker Build
 ↓
ACR
 ↓
Helm
 ↓
AKS
 ↓
Argo CD
 ↓
Prometheus/Grafana
```

> Each service should be independently testable and deployable where appropriate. The pipeline builds and tests the service, performs quality and security checks, publishes a versioned image to the registry, and deploys it through Kubernetes and Helm. GitOps tools such as Argo CD can manage the desired deployment state.

## Q21. How do you reduce pipeline execution time?

**Answer:**

> I first identify the slow stages using pipeline metrics. Then I parallelize independent tests, cache dependencies, reuse pipeline templates, avoid unnecessary builds, optimize Docker layers, and run fast validation before expensive stages.

## Q22. How do you handle secrets in pipelines?

**Answer:**

> I use centralized secret management or workload identity instead of hard-coded credentials. Access is restricted using least privilege, secrets are rotated, and pipeline logs are protected from exposing secret values.

## Q23. What is rollback?

**Answer:**

> Rollback means returning the application to a previous known-good version after a problematic deployment.

## Q24. How do you monitor a deployment?

**Answer:**

> I monitor availability, latency, error rate, resource utilization, logs, deployment health, and business-critical metrics. Prometheus and Grafana can be used for metrics and visualization, while centralized logging helps investigate failures.

## Q25. What is DevSecOps?

**Answer:**

> DevSecOps integrates security throughout the software delivery lifecycle instead of treating security as a final step. It includes secure coding, secret detection, dependency scanning, SAST, container scanning, access control, and runtime security.

---

# 28. Scenario-Based Interview Practice

## Scenario 1 — Pipeline succeeds but application fails

### Question

> The CI/CD pipeline succeeded, but the application failed after deployment. What could be wrong?

### Answer

Check:

```text
Configuration
    ↓
Environment Variables
    ↓
Secrets
    ↓
Database Connectivity
    ↓
Network Rules
    ↓
Image / Runtime Compatibility
    ↓
Health Probes
    ↓
Application Logs
```

A successful build does not guarantee runtime success.

---

## Scenario 2 — Merge Conflict

### Question

> Two developers changed the same file and the merge has conflicts. What do you do?

### Answer

```text
Fetch Latest Changes
       ↓
Inspect Conflict
       ↓
Understand Both Changes
       ↓
Resolve Manually
       ↓
Run Tests
       ↓
git add
       ↓
Complete Merge/Rebase
```

Never blindly choose one side without understanding the application behavior.

---

## Scenario 3 — Secret Accidentally Committed

### Question

> A developer accidentally committed a password to Git. What do you do?

### Answer

1. Treat the credential as compromised.
2. Immediately revoke/rotate it.
3. Remove it from repository history according to organizational policy.
4. Check whether it was accessed.
5. Review logs.
6. Add secret scanning.
7. Prevent recurrence.

> Simply deleting the password from the latest file does not mean the secret is gone from Git history.

---

## Scenario 4 — Production Incident

### Question

> Would you always rollback a failed production deployment?

### Answer

No.

Rollback is appropriate when:

- Previous version is known-good.
- Rollback is safe.
- Database/schema compatibility allows it.
- The previous version can handle current data.

Sometimes a forward fix is safer, especially when a database migration cannot safely be reversed.

---

# 29. Senior-Level Interview Answer Framework

For a 6-year experience interview, avoid giving only textbook definitions.

Use:

```text
Situation
   ↓
Problem
   ↓
Approach
   ↓
Tools
   ↓
Security / Reliability
   ↓
Result
```

### Example

> Our team had manual deployments and inconsistent environment changes.

> I standardized the CI/CD workflow and moved deployment configuration into version control.

> We used Azure DevOps, Docker, ACR, Kubernetes, Helm, and Argo CD.

> We added approvals, secret management, image scanning, and deployment health checks.

> This made the deployment process repeatable, auditable, and significantly reduced manual effort.

---

# 30. Day 1 Self-Assessment

Before moving to Day 2:

- [ ] I can explain DevOps in my own words.
- [ ] I can explain the DevOps lifecycle.
- [ ] I understand CI.
- [ ] I understand Continuous Delivery.
- [ ] I understand Continuous Deployment.
- [ ] I can explain CI vs CD.
- [ ] I can design a basic enterprise CI/CD pipeline.
- [ ] I understand build-once/deploy-many.
- [ ] I know how to manage CI/CD secrets.
- [ ] I can initialize a Git repository.
- [ ] I can create a feature branch.
- [ ] I can merge a feature branch.
- [ ] I understand `fetch` vs `pull`.
- [ ] I understand `merge` vs `rebase`.
- [ ] I understand `revert` vs `reset`.
- [ ] I understand `cherry-pick`.
- [ ] I understand `stash`.
- [ ] I understand `reflog`.
- [ ] I completed the hands-on lab.
- [ ] I created a basic CI pipeline.
- [ ] I can answer at least 10 interview questions without notes.

---

# 31. Homework Before Day 2

- [ ] Complete the Day 1 Git lab.
- [ ] Push the project to GitHub or Azure Repos.
- [ ] Create the basic CI pipeline.
- [ ] Practice explaining CI/CD aloud for 3 minutes.
- [ ] Answer the interview questions aloud.
- [ ] Record yourself answering at least 5 questions.
- [ ] Write down 3 CI/CD problems you have personally encountered.
- [ ] Explain how you solved those problems.
- [ ] Revise `merge`, `rebase`, `cherry-pick`, `revert`, `reset`, `stash`, and `reflog`.

---

# 32. Final Day 1 Challenge

Without looking at your notes, explain this architecture for 5 minutes:

```text
Developer
   ↓
Git
   ↓
Pull Request
   ↓
CI Pipeline
   ↓
Build
   ↓
Unit Test
   ↓
SonarQube
   ↓
Trivy
   ↓
Docker Image
   ↓
Azure Container Registry
   ↓
Deployment
   ↓
Kubernetes / AKS
   ↓
Argo CD
   ↓
Prometheus
   ↓
Grafana
```

You should be able to explain:

1. Why each component exists.
2. What happens when the pipeline fails.
3. Where secrets are stored.
4. How security is implemented.
5. How the application is rolled back.
6. How monitoring detects problems.
7. How the same artifact is promoted between environments.

---

# 33. Day 1 Success Criteria

You are ready for **Day 2 — Advanced Git** when you can explain this without reading:

```text
Code
 ↓
Git
 ↓
PR
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Quality
 ↓
Security
 ↓
Artifact
 ↓
CD
 ↓
Deployment
 ↓
Monitoring
 ↓
Feedback
```

---

# Next Day

## Day 2 — Advanced Git

Topics:

- Git branching strategies
- GitFlow
- Trunk-based development
- Merge vs rebase in real projects
- Cherry-pick
- Reset and revert
- Recovering deleted commits
- Merge conflicts
- Interactive rebase
- Git tags
- Release branches
- Git hooks
- Git interview scenarios
- 15+ senior-level Git interview questions
