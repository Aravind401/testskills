# Day 3 — Azure DevOps

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** Azure DevOps architecture, Azure Repos, Azure Pipelines, YAML, agents, variables, secrets, service connections, environments, approvals, artifacts, templates, security, practical CI/CD, and interview preparation  
> **Recommended time:** 3–3.5 hours  
> **Target:** ~6 years DevOps / Azure DevOps experience

---

# 1. Day 3 Objective

By the end of Day 3, you should be able to:

- Explain Azure DevOps and its major services.
- Explain Azure Repos and Pull Requests.
- Understand Azure Pipelines architecture.
- Understand agents, agent pools, jobs, stages, and steps.
- Build a YAML CI pipeline.
- Understand Microsoft-hosted vs self-hosted agents.
- Use pipeline variables and variable groups.
- Understand secret management.
- Explain Azure service connections.
- Understand environments, approvals, and checks.
- Publish and consume pipeline artifacts.
- Create reusable YAML templates.
- Understand pipeline security and permissions.
- Design a real enterprise CI/CD pipeline.
- Troubleshoot common Azure DevOps pipeline failures.
- Answer senior-level Azure DevOps interview questions.

---

# 2. Recommended 3–3.5 Hour Schedule

| Time | Duration | Topic | Expected Result |
|---|---:|---|---|
| 00:00–00:30 | 30 min | Azure DevOps overview | Understand platform architecture |
| 00:30–01:00 | 30 min | Azure Repos + PRs | Understand Git workflow |
| 01:00–01:45 | 45 min | Azure Pipelines + YAML | Build a CI pipeline |
| 01:45–02:15 | 30 min | Agents, variables, secrets | Understand execution/security |
| 02:15–02:45 | 30 min | Environments, approvals, artifacts | Understand CD |
| 02:45–03:15 | 30 min | Templates + enterprise design | Build reusable pipelines |
| 03:15–03:45 | 30 min | Interview Q&A | Senior-level preparation |

---

# 3. What Is Azure DevOps?

Azure DevOps is a collection of development and delivery services used to plan work, host source code, automate builds/releases, manage packages, and support software delivery.

The major services are:

```text
Azure DevOps
│
├── Azure Boards
│   └── Work items / Planning
│
├── Azure Repos
│   └── Git repositories / Pull Requests
│
├── Azure Pipelines
│   └── CI/CD
│
├── Azure Test Plans
│   └── Testing
│
└── Azure Artifacts
    └── Package management
```

A common enterprise flow:

```text
Azure Boards
      ↓
Azure Repos
      ↓
Azure Pipelines
      ↓
Build / Test / Security
      ↓
Azure Artifacts / ACR
      ↓
Azure Environment
      ↓
AKS / App Service / Container Apps
      ↓
Monitoring
```

---

# 4. Azure DevOps Organization and Project

A simplified hierarchy:

```text
Azure DevOps Organization
          ↓
       Project
          ↓
 ┌────────┼──────────┐
 ↓        ↓          ↓
Repos   Pipelines   Boards
          ↓
       Artifacts
```

A project provides a boundary for related development resources.

Typical project resources include:

- Repositories
- Pipelines
- Boards
- Test Plans
- Artifacts
- Teams
- Permissions

---

# 5. Azure Repos

Azure Repos provides Git repositories for source-code management.

Typical workflow:

```text
Developer
   ↓
Clone Repository
   ↓
Feature Branch
   ↓
Code
   ↓
Commit
   ↓
Push
   ↓
Pull Request
   ↓
CI Validation
   ↓
Code Review
   ↓
Merge
```

Commands:

```bash
git clone <repository-url>
git switch -c feature/login
git add .
git commit -m "Add login feature"
git push -u origin feature/login
```

---

# 6. Pull Requests and Branch Policies

A Pull Request proposes changes before they are merged.

Typical PR controls:

- Reviewer approval
- Build validation
- Branch policies
- Work-item linking
- Comment resolution
- Security checks
- Minimum reviewers
- Merge strategy

A good enterprise workflow:

```text
Feature Branch
      ↓
Pull Request
      ↓
Build Validation
      ↓
Unit Tests
      ↓
SonarQube
      ↓
Security Scan
      ↓
Reviewer Approval
      ↓
Merge
```

Protect `main` from direct pushes where appropriate:

```text
main
 │
 ├── Require Pull Request
 ├── Require reviewers
 ├── Build validation
 ├── Comment resolution
 ├── Work item linking
 └── Limit direct pushes
```

---

# 7. Azure Pipelines

Azure Pipelines provides CI/CD automation.

It can:

- Build applications
- Run tests
- Run security scans
- Create artifacts
- Build Docker images
- Push images to ACR
- Deploy to Azure
- Deploy to Kubernetes
- Run scripts
- Manage releases

Simplified:

```text
Git
 ↓
Pipeline
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Artifact
 ↓
Deploy
 ↓
Monitor
```

---

# 8. Azure Pipeline Structure

A YAML pipeline can contain:

```text
Pipeline
   ↓
Stages
   ↓
Jobs
   ↓
Steps
   ↓
Tasks / Scripts
```

Example:

```yaml
stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: echo "Build"

- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - script: echo "Deploy"
```

### Definitions

- **Pipeline:** complete automation definition.
- **Stage:** logical boundary such as Build, Test, or Deploy.
- **Job:** group of steps executed together on an agent.
- **Step:** individual action.
- **Task:** reusable Azure Pipelines action.

---

# 9. First Azure DevOps YAML Pipeline

Create:

```text
azure-pipelines.yml
```

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
  displayName: "Validate Python"
```

Flow:

```text
Push to main
     ↓
Pipeline Trigger
     ↓
Ubuntu Agent
     ↓
Checkout
     ↓
Run Python
     ↓
Validate
```

---

# 10. Triggers

Example:

```yaml
trigger:
- main
```

Multiple branches:

```yaml
trigger:
- main
- develop
```

Disable CI trigger:

```yaml
trigger: none
```

Pull Request validation can also be configured through repository policies and pipeline settings according to the team's workflow.

---

# 11. Agents and Agent Pools

Example:

```yaml
pool:
  vmImage: ubuntu-latest
```

This uses a Microsoft-hosted Ubuntu agent.

## Microsoft-hosted Agent

Advantages:

- Easy setup
- Fresh execution environment
- No server maintenance
- Common tools available

Considerations:

- Execution environment is ephemeral.
- Custom tools may need installation.
- Private network access requires appropriate architecture.

## Self-hosted Agent

```text
Azure DevOps
     ↓
Self-Hosted Agent
     ↓
Private Network
     ↓
Internal Resources
```

Advantages:

- Custom tools
- Private network access
- Specialized hardware
- Controlled environment

Responsibilities:

- OS patching
- Security
- Agent maintenance
- Capacity
- Credentials
- Monitoring

## Agent Pool

```text
Agent Pool: Production-Build
├── agent-01
├── agent-02
└── agent-03
```

Use:

```yaml
pool:
  name: Production-Build
```

---

# 12. Pipeline Variables

Example:

```yaml
variables:
  appName: "devops-demo"
  environment: "dev"
```

Use:

```yaml
steps:
- script: |
    echo "Application: $(appName)"
    echo "Environment: $(environment)"
```

Variables can exist at pipeline, stage, or job scope. Prefer the narrowest sensible scope.

---

# 13. Secret Management

Never hard-code credentials:

```yaml
variables:
  password: "MyPassword123"
```

Prefer:

- Azure Key Vault
- Protected pipeline secrets
- Variable groups with appropriate permissions
- Managed identity/workload identity where appropriate

Do not intentionally print secrets in logs.

A common design:

```text
Azure Pipeline
      ↓
Managed Identity / Service Connection
      ↓
Azure Key Vault
      ↓
Secret
      ↓
Deployment
```

---

# 14. Variable Groups

A variable group can centralize shared configuration:

```text
Variable Group
│
├── API_URL
├── DATABASE_NAME
├── ENVIRONMENT
└── SECRET_REFERENCE
```

Security practices:

- Restrict access.
- Separate environment-specific values.
- Avoid plaintext credentials in source control.
- Prefer Key Vault for sensitive secrets.

---

# 15. Service Connections

A service connection allows Azure DevOps pipelines to authenticate to external resources.

Examples:

```text
Azure Resource Manager
Azure Container Registry
Kubernetes
Docker Registry
GitHub
```

Flow:

```text
Pipeline
   ↓
Service Connection
   ↓
Azure / Registry / Kubernetes
   ↓
Resource
```

Security principle:

```text
Pipeline
   ↓
Least-Privilege Identity
   ↓
Required Resource Only
```

Do not give every pipeline broad subscription-level access.

---

# 16. Docker + ACR Pipeline

Common enterprise flow:

```text
Source
  ↓
Build
  ↓
Test
  ↓
Docker Build
  ↓
Trivy Scan
  ↓
ACR Push
  ↓
Deploy
```

Example:

```yaml
steps:

- task: Docker@2
  displayName: "Build and Push Image"
  inputs:
    command: buildAndPush
    repository: myapp
    dockerfile: Dockerfile
    containerRegistry: my-acr-service-connection
    tags: |
      $(Build.BuildId)
```

Prefer immutable tags such as:

```text
myapp:1254
```

rather than relying only on:

```text
myapp:latest
```

---

# 17. Pipeline Artifacts

Publish build output:

```yaml
steps:

- script: |
    mkdir -p $(Build.ArtifactStagingDirectory)
    echo "Application package" > $(Build.ArtifactStagingDirectory)/app.txt

- task: PublishPipelineArtifact@1
  inputs:
    targetPath: "$(Build.ArtifactStagingDirectory)"
    artifact: "application"
```

Flow:

```text
Build
 ↓
Artifact Staging
 ↓
Publish Artifact
 ↓
Deploy Stage
```

The same validated artifact should be promoted instead of rebuilding separately for every environment.

---

# 18. Multi-Stage Pipeline

Example:

```yaml
stages:

- stage: Build
  jobs:
  - job: Build
    steps:
    - script: echo "Build"

- stage: Test
  dependsOn: Build
  jobs:
  - job: Test
    steps:
    - script: echo "Test"

- stage: DeployDev
  dependsOn: Test
  jobs:
  - job: Deploy
    steps:
    - script: echo "Deploy Dev"
```

Flow:

```text
Build
 ↓
Test
 ↓
Deploy Dev
```

---

# 19. Conditions and Dependencies

Example:

```yaml
condition: succeeded()
```

Production example:

```yaml
condition: and(
  succeeded(),
  eq(variables['Build.SourceBranch'], 'refs/heads/main')
)
```

Use conditions carefully. Prefer readable pipeline logic over very complicated expressions.

---

# 20. Environments and Approvals

Typical environments:

```text
dev
qa
staging
production
```

Flow:

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
Production
     ↑
Approval / Checks
```

Production should have stronger controls than development.

Possible controls include:

- Manual approvals
- Branch restrictions
- Business-hour checks
- Security checks
- Resource checks
- External checks

Configure production controls at the environment level where possible.

---

# 21. YAML Templates

Templates reduce duplicated YAML.

Structure:

```text
templates/
├── build.yml
├── test.yml
└── deploy.yml
```

Example template:

```yaml
# templates/build.yml
stages:

- stage: Build
  jobs:
  - job: Build
    steps:
    - script: echo "Building application"
      displayName: "Build"
```

Main pipeline:

```yaml
trigger:
- main

stages:
- template: templates/build.yml
```

---

# 22. Template Parameters

Example:

```yaml
parameters:
- name: environment
  type: string
  default: dev

steps:
- script: |
    echo "Deploying to ${{ parameters.environment }}"
```

Use:

```yaml
- template: templates/deploy.yml
  parameters:
    environment: production
```

This makes pipelines reusable.

---

# 23. Enterprise CI/CD Architecture

```text
                         Developer
                            |
                            v
                     Azure Repos / Git
                            |
                            v
                     Pull Request
                            |
                            v
                     Branch Policies
                            |
                            v
                     CI Validation
                            |
          +-----------------+----------------+
          |                 |                |
          v                 v                v
       Build            Unit Tests       SonarQube
          |                 |                |
          +-----------------+----------------+
                            |
                            v
                      Security Scan
                            |
                            v
                       Docker Build
                            |
                            v
                    Trivy Container Scan
                            |
                            v
                           ACR
                            |
                            v
                      Deploy to Dev
                            |
                            v
                    Integration Tests
                            |
                            v
                         Staging
                            |
                            v
                    Approval / Checks
                            |
                            v
                       Production
                            |
                            v
                  Prometheus / Grafana
```

---

# 24. Day 3 Hands-on Lab

## Objective

Create a basic Azure DevOps CI/CD workflow containing:

- Git repository
- Branch trigger
- Python application
- Unit tests
- Build validation
- Pipeline artifact
- Build and deployment stages

## Step 1 — Project Structure

```text
azure-devops-day3/
├── app/
│   └── hello.py
├── tests/
│   └── test_hello.py
└── azure-pipelines.yml
```

`app/hello.py`:

```python
def hello():
    return "Hello Azure DevOps!"

if __name__ == "__main__":
    print(hello())
```

`tests/test_hello.py`:

```python
from app.hello import hello

def test_hello():
    assert hello() == "Hello Azure DevOps!"
```

---

## Step 2 — Git Repository

```bash
git init
git branch -M main
git add .
git commit -m "Initial Azure DevOps project"
```

Create an Azure Repos repository and push:

```bash
git remote add origin <repository-url>
git push -u origin main
```

---

## Step 3 — Basic CI Pipeline

Create `azure-pipelines.yml`:

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:

- checkout: self

- script: |
    python --version
    python -m compileall app
  displayName: "Validate Python"

- script: |
    python -m pip install pytest
    pytest
  displayName: "Run Unit Tests"
```

---

## Step 4 — Publish Artifact

Add:

```yaml
- script: |
    mkdir -p $(Build.ArtifactStagingDirectory)
    cp -r app $(Build.ArtifactStagingDirectory)/
  displayName: "Prepare Artifact"

- task: PublishPipelineArtifact@1
  inputs:
    targetPath: "$(Build.ArtifactStagingDirectory)"
    artifact: "application"
  displayName: "Publish Application Artifact"
```

Now the flow is:

```text
Git
 ↓
Build
 ↓
Test
 ↓
Artifact
```

---

## Step 5 — Multi-Stage Pipeline

```yaml
trigger:
- main

stages:

- stage: Build
  jobs:
  - job: Build
    pool:
      vmImage: ubuntu-latest

    steps:
    - checkout: self

    - script: |
        python --version
        python -m compileall app
        python -m pip install pytest
        pytest
      displayName: "Build and Test"

    - script: |
        mkdir -p $(Build.ArtifactStagingDirectory)
        cp -r app $(Build.ArtifactStagingDirectory)/
      displayName: "Prepare Artifact"

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: "$(Build.ArtifactStagingDirectory)"
        artifact: "application"

- stage: DeployDev
  dependsOn: Build
  jobs:
  - job: Deploy
    pool:
      vmImage: ubuntu-latest

    steps:
    - download: current
      artifact: application

    - script: |
        echo "Deploying application to DEV"
        find "$(Pipeline.Workspace)/application" -type f
      displayName: "Deploy to Dev"
```

---

# 25. Troubleshooting Guide

## Pipeline Does Not Trigger

Check:

```text
Branch
 ↓
YAML trigger
 ↓
Pipeline configuration
 ↓
Repository connection
 ↓
Branch policies
```

Example:

```yaml
trigger:
- main
```

Confirm the commit was actually pushed to the configured branch.

## Agent Cannot Run Job

Check:

- Agent pool
- Agent availability
- Agent status
- Permissions
- Network
- Required tools
- Disk space
- CPU/memory

For self-hosted agents also check:

```text
Agent Service
Agent Connectivity
OS
Tool Versions
Network Access
Credentials
```

## Service Connection Authentication Failure

Check:

- Service connection exists
- Pipeline is authorized to use it
- Identity has required permissions
- Subscription/resource is correct
- Credential/token has not expired
- Scope is sufficient but not excessive

## Artifact Not Found

Check:

```text
Build Stage
 ↓
Artifact Published?
 ↓
Artifact Name
 ↓
Deploy Stage
 ↓
Artifact Download
```

Make sure artifact names match exactly.

## Pipeline Works Locally but Fails on Agent

Common causes:

- Different OS
- Different runtime version
- Missing dependency
- Environment variable difference
- File-path difference
- Permission difference
- Network restrictions

Best practice:

> Make build dependencies explicit and pin important runtime/tool versions.

---

# 26. Azure DevOps Interview Questions

## Q1. What is Azure DevOps?

> Azure DevOps is a set of development and delivery services covering planning, source control, CI/CD, testing, and package management.

## Q2. What are the main Azure DevOps services?

> Azure Boards, Azure Repos, Azure Pipelines, Azure Test Plans, and Azure Artifacts.

## Q3. What is Azure Repos?

> Azure Repos provides Git repositories and collaboration features such as pull requests and branch policies.

## Q4. What is Azure Pipelines?

> Azure Pipelines automates build, test, packaging, and deployment workflows.

## Q5. What is an agent?

> An agent is the compute environment that executes pipeline jobs and steps.

## Q6. Microsoft-hosted vs self-hosted agent?

> Microsoft-hosted agents are managed by Microsoft and provide fresh execution environments. Self-hosted agents are managed by the organization and are useful for custom tools, private networks, or specialized workloads.

## Q7. What is an agent pool?

> An agent pool is a collection of agents that can execute pipeline jobs.

## Q8. What is a stage?

> A stage is a logical boundary in a pipeline, such as Build, Test, or Production deployment.

## Q9. What is a job?

> A job is a group of steps executed together on an agent.

## Q10. What is a step?

> A step is an individual action such as a script or task.

## Q11. What is a task?

> A task is a reusable predefined pipeline action such as Docker build, Azure CLI execution, or artifact publishing.

## Q12. Why use YAML pipelines?

> YAML keeps pipeline definitions as code. They can be reviewed, versioned, reused, and changed through Git workflows.

## Q13. What is a service connection?

> A service connection provides an authentication mechanism that allows Azure DevOps pipelines to interact with external services such as Azure resources, registries, Kubernetes, or other platforms.

## Q14. How do you secure service connections?

> Use least privilege, restrict pipeline authorization, limit scope, review access regularly, and prefer managed identities or workload identities where appropriate.

## Q15. What are pipeline variables?

> Variables store configuration values used during pipeline execution.

## Q16. How do you store secrets?

> Use secure secret management such as Azure Key Vault or protected pipeline secret mechanisms rather than source-controlled plaintext.

## Q17. What are variable groups?

> Variable groups provide shared variables that multiple pipelines can consume, subject to permissions.

## Q18. What is an Azure DevOps environment?

> An environment represents a deployment target or logical environment and can provide deployment history and controls such as approvals and checks.

## Q19. How do you implement production approval?

> I create a production environment and configure appropriate approvals and checks there. The deployment targets that environment only after the configured controls pass.

## Q20. What is a pipeline artifact?

> A pipeline artifact is a build output published by one stage/job and consumed later by another stage or pipeline.

## Q21. Why use artifacts?

> They allow the exact output of a validated build to be promoted across environments instead of rebuilding separately.

## Q22. What are YAML templates?

> Templates allow common pipeline logic to be reused across multiple pipelines.

## Q23. How do you reduce duplicated YAML?

> Move repeated build, test, security, and deployment logic into templates and use parameters for differences between applications or environments.

## Q24. How do you secure Azure DevOps?

> Protect repositories with branch policies, secure service connections, restrict permissions, protect secrets, use environment approvals, scan source and artifacts, and audit pipeline activity.

## Q25. How would you design Azure DevOps for 50 microservices?

> I would standardize repository and pipeline conventions, use reusable YAML templates, centralize common security checks, keep service-specific configuration separate, publish versioned container images to ACR, deploy through Kubernetes/GitOps where appropriate, and implement consistent approvals, observability, and access controls.

---

# 27. Senior-Level Scenario Questions

## Scenario 1 — Pipeline Works in Dev but Not Production

Check:

```text
Environment Variables
        ↓
Secrets
        ↓
Service Connection
        ↓
Network
        ↓
Resource Permissions
        ↓
Runtime Configuration
        ↓
Deployment Logs
        ↓
Application Logs
```

Compare Dev and Production configuration deliberately.

## Scenario 2 — Self-Hosted Agent Is Offline

Investigate:

```text
Agent Service
     ↓
Operating System
     ↓
Network
     ↓
DNS
     ↓
Firewall/Proxy
     ↓
Agent Authentication
     ↓
Disk/CPU/Memory
```

## Scenario 3 — ACR Push Fails

Check:

```text
Docker Build
    ↓
Image Exists?
    ↓
Registry Login
    ↓
Service Connection
    ↓
Identity Permissions
    ↓
ACR Name
    ↓
Repository Name
    ↓
Tag
```

Typical causes:

- Authentication
- Authorization
- Wrong registry
- Wrong repository
- Expired credentials
- Network restrictions

## Scenario 4 — Secret Appears in Pipeline Log

1. Treat the secret as compromised.
2. Revoke/rotate it immediately.
3. Identify where it was printed.
4. Remove unsafe logging.
5. Check exposure and audit information.
6. Improve secret handling.
7. Add prevention controls.

## Scenario 5 — Prevent Feature Branch Production Deployment

Use multiple controls:

```text
Feature Branch
      ↓
CI Only
      ↓
Pull Request
      ↓
Main
      ↓
Build
      ↓
Production Deployment
```

Additionally:

- Protect `main`.
- Require PR approvals.
- Restrict production environment permissions.
- Configure environment checks.
- Restrict service connection usage.
- Use branch-aware deployment conditions where appropriate.

---

# 28. Day 3 Practical Checklist

- [ ] I understand Azure DevOps organization/project concepts.
- [ ] I understand Azure Repos.
- [ ] I can create a feature branch.
- [ ] I understand Pull Requests.
- [ ] I understand branch policies.
- [ ] I understand Azure Pipelines.
- [ ] I understand stages.
- [ ] I understand jobs.
- [ ] I understand steps.
- [ ] I understand tasks.
- [ ] I understand agents.
- [ ] I understand agent pools.
- [ ] I understand Microsoft-hosted agents.
- [ ] I understand self-hosted agents.
- [ ] I can write a basic YAML pipeline.
- [ ] I understand triggers.
- [ ] I understand variables.
- [ ] I understand secret handling.
- [ ] I understand variable groups.
- [ ] I understand service connections.
- [ ] I understand environments.
- [ ] I understand approvals/checks.
- [ ] I understand pipeline artifacts.
- [ ] I understand YAML templates.
- [ ] I completed the Day 3 lab.
- [ ] I can explain an enterprise Azure DevOps pipeline.
- [ ] I answered at least 20 interview questions aloud.

---

# 29. Day 3 Homework

- [ ] Create an Azure DevOps project.
- [ ] Create an Azure Repos Git repository.
- [ ] Push the Day 3 Python application.
- [ ] Create `azure-pipelines.yml`.
- [ ] Run a CI pipeline.
- [ ] Add unit tests.
- [ ] Publish a pipeline artifact.
- [ ] Create Build and Deploy stages.
- [ ] Explore agent pools.
- [ ] Create a test environment.
- [ ] Explore variable groups.
- [ ] Review service connection permissions.
- [ ] Create one reusable YAML template.
- [ ] Explain Microsoft-hosted vs self-hosted agents aloud.
- [ ] Record yourself answering 5 Azure DevOps interview questions.

---

# 30. Final Day 3 Challenge

Without looking at your notes, explain this architecture:

```text
Azure Repos
     ↓
Pull Request
     ↓
Branch Policy
     ↓
Azure Pipeline
     ↓
Microsoft-Hosted / Self-Hosted Agent
     ↓
Build
     ↓
Unit Test
     ↓
SonarQube
     ↓
Trivy
     ↓
Docker Build
     ↓
Azure Container Registry
     ↓
Dev Environment
     ↓
QA
     ↓
Staging
     ↓
Approval / Checks
     ↓
Production
     ↓
Prometheus / Grafana
```

Explain:

1. What triggers the pipeline?
2. Which agent executes the job?
3. Where are secrets stored?
4. How does the pipeline authenticate to Azure?
5. How is the Docker image pushed to ACR?
6. How is the same artifact promoted?
7. How do you protect production?
8. How do you troubleshoot an agent failure?
9. How do you troubleshoot an ACR authentication failure?
10. How do you prevent feature branches from deploying to production?

---

# 31. Day 3 Success Criteria

You are ready for **Day 4 — Azure Pipelines Advanced** when you can confidently explain:

```text
Azure DevOps
     ↓
Azure Repos
     ↓
Pull Request
     ↓
Branch Policy
     ↓
Azure Pipeline
     ↓
Agent
     ↓
Stage
     ↓
Job
     ↓
Step / Task
     ↓
Artifact
     ↓
Environment
     ↓
Approval
     ↓
Deployment
```

You should also be able to write a basic YAML pipeline from memory.

---

# Next Day

## Day 4 — Azure Pipelines Advanced

Topics:

- Advanced YAML
- Runtime vs compile-time expressions
- Parameters vs variables
- Conditions
- Dependencies
- Output variables
- Multi-stage pipelines
- Templates
- Deployment jobs
- Environments
- Approvals and checks
- Artifacts
- Docker CI/CD
- SonarQube integration
- Trivy integration
- ACR authentication
- Azure CLI tasks
- Pipeline troubleshooting
- Enterprise pipeline design
- 25+ advanced Azure Pipelines interview questions
