# Day 4 — Azure Pipelines Advanced

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** Advanced Azure Pipelines YAML, expressions, parameters, variables, conditions, dependencies, output variables, multi-stage pipelines, deployment jobs, environments, templates, artifacts, Docker, SonarQube, Trivy, ACR authentication, troubleshooting, and senior-level interview preparation  
> **Recommended time:** 3.5–4 hours  
> **Target:** ~6 years DevOps / Azure DevOps experience

---

## 1. Objectives

By the end of Day 4, you should be able to:

- Write advanced Azure Pipelines YAML.
- Understand compile-time vs runtime expressions.
- Explain parameters vs variables.
- Use conditions and dependencies.
- Pass output variables between jobs and stages.
- Build multi-stage CI/CD pipelines.
- Use deployment jobs and environments.
- Create reusable YAML templates.
- Publish and consume artifacts.
- Build and push Docker images.
- Integrate SonarQube and Trivy.
- Understand ACR authentication.
- Troubleshoot common pipeline failures.
- Design secure enterprise pipelines.
- Answer advanced Azure Pipelines interview questions.

## 2. Four-Hour Schedule

| Time | Duration | Topic |
|---|---:|---|
| 00:00–00:30 | 30 min | YAML fundamentals |
| 00:30–01:10 | 40 min | Variables, parameters, expressions |
| 01:10–01:50 | 40 min | Conditions, dependencies, outputs |
| 01:50–02:30 | 40 min | Multi-stage + deployment jobs |
| 02:30–03:10 | 40 min | Docker + ACR + security |
| 03:10–03:40 | 30 min | Templates + artifacts |
| 03:40–04:10 | 30 min | Troubleshooting + interview Q&A |

---

# 3. Azure Pipeline YAML Structure

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
trigger:
- main

pool:
  vmImage: ubuntu-latest

stages:

- stage: Build
  jobs:
  - job: BuildApplication
    steps:
    - script: echo "Build"

- stage: Deploy
  dependsOn: Build
  jobs:
  - job: DeployApplication
    steps:
    - script: echo "Deploy"
```

---

# 4. Pipeline Evaluation Model

Not everything is evaluated at the same time.

```text
Pipeline YAML
     ↓
Template / Compile Time
     ↓
Expanded Pipeline
     ↓
Runtime
     ↓
Jobs / Steps Execute
```

This matters for:

- Parameters
- Variables
- Expressions
- Conditions
- Templates

---

# 5. Compile-Time Expressions

Compile-time expressions use:

```yaml
${{ }}
```

Example:

```yaml
parameters:
- name: environment
  type: string
  default: dev

steps:
- script: |
    echo "Environment: ${{ parameters.environment }}"
```

Parameters are resolved during pipeline/template expansion.

---

# 6. Runtime Variables

Variables are commonly referenced using:

```yaml
$(variableName)
```

Example:

```yaml
variables:
  environment: dev

steps:
- script: |
    echo "Environment: $(environment)"
```

---

# 7. Runtime Expressions

Runtime expressions commonly use:

```yaml
$[ ]
```

Example:

```yaml
variables:
  isMain: $[eq(variables['Build.SourceBranch'], 'refs/heads/main')]
```

Then:

```yaml
steps:
- script: echo "Main branch detected"
  condition: eq(variables.isMain, 'True')
```

---

# 8. Three Expression Forms

| Syntax | Main Use |
|---|---|
| `${{ }}` | Compile-time/template expressions |
| `$( )` | Variable substitution |
| `$[ ]` | Runtime expression evaluation |

Easy memory:

```text
${{ }} → Before runtime
$( )   → Substitute variable
$[ ]   → Evaluate runtime expression
```

---

# 9. Parameters vs Variables

## Parameters

Useful for controlling pipeline/template structure.

```yaml
parameters:
- name: runSecurityScan
  type: boolean
  default: true
```

Use:

```yaml
${{ parameters.runSecurityScan }}
```

## Variables

Better for runtime configuration.

```yaml
variables:
  environment: dev
  imageName: myapp
```

Use:

```yaml
$(environment)
$(imageName)
```

### Interview Answer

> Parameters are generally evaluated during template expansion and are useful for controlling pipeline structure and reusable template behavior. Variables are runtime-oriented configuration values that can change during execution and can come from pipeline settings, variable groups, or other sources.

---

# 10. Parameter Types

```yaml
parameters:
- name: environment
  type: string
  default: dev

- name: runTests
  type: boolean
  default: true

- name: environments
  type: object
  default:
  - dev
  - qa
```

---

# 11. Conditional Template Logic

```yaml
parameters:
- name: runSecurityScan
  type: boolean
  default: true

steps:

- script: echo "Build application"

- ${{ if eq(parameters.runSecurityScan, true) }}:
  - script: echo "Run security scan"
```

This is useful for reusable templates.

---

# 12. Conditions

Conditions control whether stages, jobs, or steps run.

Common examples:

```yaml
condition: succeeded()
```

```yaml
condition: failed()
```

```yaml
condition: always()
```

```yaml
condition: succeededOrFailed()
```

---

# 13. Branch-Based Conditions

Example:

```yaml
condition: and(
  succeeded(),
  eq(variables['Build.SourceBranch'], 'refs/heads/main')
)
```

Flow:

```text
Build
 ↓
Test
 ↓
Is branch main?
 ├── No  → Stop production deployment
 └── Yes → Deploy
```

---

# 14. Pull Request vs CI Build

Feature branch:

```text
Feature Branch
      ↓
Pull Request
      ↓
PR Validation
      ↓
Build + Test + Security
```

After merge:

```text
main
 ↓
CI
 ↓
Artifact
 ↓
CD
```

This prevents feature branches from automatically reaching production.

---

# 15. Dependencies

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
```

Flow:

```text
Build
 ↓
Test
```

Multiple dependencies:

```yaml
- stage: Deploy
  dependsOn:
  - Build
  - Security
```

---

# 16. Parallel Jobs

Independent jobs can run in parallel:

```yaml
jobs:

- job: UnitTests
  steps:
  - script: echo "Unit Tests"

- job: SecurityScan
  steps:
  - script: echo "Security Scan"
```

Concept:

```text
          ┌→ Unit Tests ─┐
Build ────┤              ├→ Deploy
          └→ Security ───┘
```

Parallelization can reduce pipeline duration.

---

# 17. Output Variables

One job can produce a value another job needs.

Example:

```text
Build Job
   ↓
Image Tag
   ↓
Deploy Job
```

Example:

```yaml
jobs:

- job: Build

  steps:

  - bash: |
      echo "##vso[task.setvariable variable=imageTag;isOutput=true]$(Build.BuildId)"
    name: SetTag

- job: Deploy

  dependsOn: Build

  variables:
    imageTag: $[ dependencies.Build.outputs['SetTag.imageTag'] ]

  steps:
  - script: |
      echo "Deploying image: $(imageTag)"
```

Important pieces:

1. `isOutput=true`
2. Step `name`
3. Job dependency
4. Correct `dependencies` syntax

---

# 18. Multi-Stage Pipeline

A realistic flow:

```text
Build
 ↓
Test
 ↓
Security
 ↓
Package
 ↓
Deploy Dev
 ↓
Integration Test
 ↓
Deploy QA
 ↓
Approval
 ↓
Production
```

Example:

```yaml
stages:

- stage: Build

- stage: Test
  dependsOn: Build

- stage: Security
  dependsOn: Build

- stage: DeployDev
  dependsOn:
  - Test
  - Security

- stage: DeployQA
  dependsOn: DeployDev

- stage: Production
  dependsOn: DeployQA
```

---

# 19. Deployment Jobs

Deployment jobs are designed for deployment workflows and environment tracking.

```yaml
jobs:

- deployment: DeployDev

  environment: dev

  strategy:
    runOnce:
      deploy:
        steps:
        - script: echo "Deploying application"
```

Concept:

```text
Build
 ↓
Deployment Job
 ↓
Environment
 ↓
Deployment
```

---

# 20. Deployment Strategies

Deployment jobs can use strategies such as:

```text
runOnce
rolling
canary
```

The exact configuration depends on the target resource and Azure DevOps capabilities.

---

# 21. Environment-Based Deployment

Example:

```yaml
environment: production
```

Concept:

```text
Pipeline
   ↓
Production Environment
   ↓
Approval / Checks
   ↓
Deployment
```

Environments can provide deployment history, permissions, and checks.

---

# 22. Production Approval

Recommended flow:

```text
Build
 ↓
Test
 ↓
Security
 ↓
Dev
 ↓
QA
 ↓
Staging
 ↓
Production Environment
 ↓
Approval / Checks
 ↓
Production
```

Do not rely only on a YAML branch condition.

Use multiple layers:

- Branch policies
- Environment permissions
- Approvals/checks
- Service connection restrictions
- Least-privilege identities

---

# 23. Docker Build Pipeline

Typical flow:

```text
Source
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
ACR Push
```

Example:

```yaml
- task: Docker@2
  displayName: "Build and Push Docker Image"
  inputs:
    command: buildAndPush
    repository: myapp
    dockerfile: Dockerfile
    containerRegistry: my-acr
    tags: |
      $(Build.BuildId)
```

---

# 24. Docker Image Tagging

Avoid depending only on:

```text
latest
```

Prefer immutable tags:

```text
myapp:1001
myapp:1002
myapp:1003
```

Or release versions:

```text
myapp:1.4.2
```

A useful enterprise rule:

> Every deployable image should be traceable back to the source commit and build.

---

# 25. ACR Authentication

Pipeline:

```text
Azure DevOps
      ↓
Service Connection / Identity
      ↓
Azure Container Registry
      ↓
Push Image
```

If push fails, check:

```text
Service Connection
        ↓
Authentication
        ↓
Authorization
        ↓
ACR Permissions
        ↓
Registry Name
        ↓
Repository
        ↓
Image Tag
```

Do not solve authorization problems by blindly granting Owner or broad Contributor permissions.

---

# 26. SonarQube Integration

Typical flow:

```text
Checkout
   ↓
Prepare SonarQube
   ↓
Build
   ↓
Test
   ↓
Analyze
   ↓
Quality Gate
```

Example structure:

```yaml
- task: SonarQubePrepare@7
  inputs:
    SonarQube: 'SonarQube-Service'
    scannerMode: 'cli'
    configMode: 'manual'
    cliProjectKey: 'my-project'
    cliSources: '.'

- script: |
    echo "Build and test"
  displayName: "Build"

- task: SonarQubeAnalyze@7

- task: SonarQubePublish@7
  inputs:
    pollingTimeoutSec: '300'
```

> Task versions can change. Use the task version supported by your Azure DevOps and SonarQube setup.

---

# 27. Trivy Integration

Typical container security flow:

```text
Docker Build
     ↓
Trivy Scan
     ↓
Critical Vulnerability?
 ├── Yes → Fail according to policy
 └── No
      ↓
   Push ACR
```

Example:

```bash
trivy image --severity HIGH,CRITICAL myapp:$(Build.BuildId)
```

Define the vulnerability policy explicitly. Do not automatically fail every finding without considering severity, exploitability, exceptions, and organizational policy.

---

# 28. Secure Docker Pipeline

```text
Git
 ↓
Build
 ↓
Unit Test
 ↓
SonarQube
 ↓
Dependency Scan
 ↓
Docker Build
 ↓
Trivy
 ↓
Image Push
 ↓
Deploy
```

Security should be integrated before deployment.

---

# 29. Artifact vs Container Image

Traditional application:

```text
Source
 ↓
Build
 ↓
JAR / ZIP
 ↓
Pipeline Artifact
 ↓
Deploy
```

Container application:

```text
Source
 ↓
Docker Build
 ↓
Container Image
 ↓
ACR
 ↓
Deploy
```

A container image in ACR is a versioned deployment artifact.

---

# 30. Reusable YAML Templates

Avoid:

```text
Pipeline A → 200 lines
Pipeline B → 200 lines
Pipeline C → 200 lines
```

Prefer:

```text
templates/
├── build.yml
├── test.yml
├── security.yml
└── deploy.yml
```

Then:

```yaml
stages:

- template: templates/build.yml

- template: templates/test.yml
```

---

# 31. Template Example

`templates/security.yml`:

```yaml
parameters:
- name: runTrivy
  type: boolean
  default: true

steps:

- script: |
    echo "Run security checks"

- ${{ if eq(parameters.runTrivy, true) }}:
  - script: |
      echo "Run Trivy"
```

Main pipeline:

```yaml
steps:

- template: templates/security.yml
  parameters:
    runTrivy: true
```

---

# 32. Complete Enterprise Pipeline

```text
Azure Repos
     ↓
Pull Request
     ↓
Branch Policy
     ↓
CI Validation
     ↓
Build
     ↓
Unit Tests
     ↓
SonarQube
     ↓
Dependency Scan
     ↓
Docker Build
     ↓
Trivy
     ↓
ACR
     ↓
Dev
     ↓
Integration Tests
     ↓
QA
     ↓
Staging
     ↓
Production Approval
     ↓
Production
     ↓
Monitoring
```

---

# 33. Complete YAML Example

```yaml
trigger:
- main

variables:
  imageName: 'myapp'
  dockerfile: 'Dockerfile'

stages:

- stage: Build

  jobs:

  - job: Build

    pool:
      vmImage: ubuntu-latest

    steps:

    - checkout: self

    - script: |
        echo "Building application"
      displayName: "Build"

    - script: |
        echo "Running unit tests"
      displayName: "Unit Tests"

    - script: |
        mkdir -p $(Build.ArtifactStagingDirectory)
        cp -r . $(Build.ArtifactStagingDirectory)/
      displayName: "Prepare Artifact"

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: "$(Build.ArtifactStagingDirectory)"
        artifact: "application"

- stage: Security

  dependsOn: Build

  jobs:

  - job: SecurityScan

    pool:
      vmImage: ubuntu-latest

    steps:

    - script: |
        echo "Dependency scan"
      displayName: "Dependency Scan"

    - script: |
        echo "Static security scan"
      displayName: "Security Scan"

- stage: DeployDev

  dependsOn:
  - Build
  - Security

  condition: succeeded()

  jobs:

  - deployment: DeployDev

    environment: dev

    strategy:

      runOnce:

        deploy:

          steps:

          - download: current
            artifact: application

          - script: |
              echo "Deploying to DEV"
            displayName: "Deploy DEV"

- stage: DeployQA

  dependsOn: DeployDev

  condition: succeeded()

  jobs:

  - deployment: DeployQA

    environment: qa

    strategy:

      runOnce:

        deploy:

          steps:

          - script: |
              echo "Deploying to QA"
            displayName: "Deploy QA"

- stage: Production

  dependsOn: DeployQA

  condition: and(
    succeeded(),
    eq(variables['Build.SourceBranch'], 'refs/heads/main')
  )

  jobs:

  - deployment: DeployProduction

    environment: production

    strategy:

      runOnce:

        deploy:

          steps:

          - script: |
              echo "Deploying to Production"
            displayName: "Deploy Production"
```

---

# 34. Troubleshooting — YAML Syntax

Common errors:

```text
Unexpected value
Mapping values are not allowed
Sequence was not expected
```

Check:

- Indentation
- Colons
- Lists
- Quotes
- Stage/job nesting
- Template syntax

Correct:

```yaml
steps:
- script: echo "Hello"
```

---

# 35. Troubleshooting — Variable Not Found

Check:

```text
Variable Name
Variable Scope
Variable Group
Secret Variable
Output Variable
Template Parameter
```

Also check whether you are mixing:

```text
${{ parameters.x }}
$(variable)
$[ expression ]
```

incorrectly.

---

# 36. Troubleshooting — Output Variable

Check:

1. `isOutput=true`
2. Step `name`
3. Correct dependency
4. Correct `dependencies` or `stageDependencies`
5. Correct expression

Example:

```yaml
- bash: |
    echo "##vso[task.setvariable variable=tag;isOutput=true]123"
  name: SetTag
```

Then:

```yaml
variables:
  tag: $[ dependencies.Build.outputs['SetTag.tag'] ]
```

---

# 37. Troubleshooting — Docker Push Unauthorized

Check:

```text
Registry
 ↓
Service Connection
 ↓
Pipeline Authorization
 ↓
Identity
 ↓
ACR Role
 ↓
Repository
 ↓
Tag
```

Avoid immediately granting subscription-wide Owner/Contributor permissions.

---

# 38. Troubleshooting — Slow Pipeline

Measure each stage:

```text
Checkout       1 min
Build          5 min
Tests          12 min
SonarQube      3 min
Docker         8 min
Trivy          5 min
Deploy         2 min
```

Possible improvements:

- Parallelize independent jobs.
- Cache dependencies.
- Optimize Docker layers.
- Avoid unnecessary rebuilds.
- Reuse templates.
- Measure before optimizing.

---

# 39. Troubleshooting — Production Deployment Too Powerful

Bad:

```text
Pipeline
   ↓
Subscription Owner
```

Better:

```text
Pipeline
   ↓
Dedicated Identity
   ↓
Required Resource Permissions
```

Use:

- Least privilege
- Resource-scoped permissions
- Environment approvals
- Service connection restrictions
- Separate identities where appropriate

---

# 40. Day 4 Hands-on Lab

## Objective

Create a realistic pipeline containing:

- Build
- Unit tests
- Security stage
- Artifact
- Dev deployment
- QA deployment
- Production deployment
- Conditions
- Output variable
- Docker image
- Reusable template

## Step 1 — Repository

```text
azure-pipelines-day4/
├── app/
│   └── hello.py
├── tests/
│   └── test_hello.py
├── templates/
│   └── security.yml
├── Dockerfile
└── azure-pipelines.yml
```

## Step 2 — Application

`app/hello.py`:

```python
def hello():
    return "Hello Azure DevOps Day 4!"

if __name__ == "__main__":
    print(hello())
```

## Step 3 — Test

`tests/test_hello.py`:

```python
from app.hello import hello

def test_hello():
    assert hello() == "Hello Azure DevOps Day 4!"
```

## Step 4 — Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app/ /app/

CMD ["python", "hello.py"]
```

For production, improve this with:

- Non-root user
- Pinned dependencies
- Minimal packages
- Health checks
- Vulnerability scanning
- Read-only filesystem where practical

## Step 5 — Security Template

`templates/security.yml`:

```yaml
parameters:
- name: environment
  type: string
  default: dev

steps:

- script: |
    echo "Security scan for ${{ parameters.environment }}"
  displayName: "Security Scan"

- script: |
    echo "Dependency scan"
  displayName: "Dependency Scan"
```

## Step 6 — Pipeline

Start with:

```yaml
trigger:
- main

variables:
  imageName: myapp

stages:

- stage: Build

  jobs:

  - job: Build

    pool:
      vmImage: ubuntu-latest

    steps:

    - checkout: self

    - script: |
        python -m pip install pytest
        pytest
      displayName: "Run Tests"

    - bash: |
        echo "##vso[task.setvariable variable=imageTag;isOutput=true]$(Build.BuildId)"
      name: SetImageTag
      displayName: "Set Image Tag"
```

## Step 7 — Consume Output Variable

```yaml
- stage: DeployDev

  dependsOn: Build

  variables:
    imageTag: $[ stageDependencies.Build.Build.outputs['SetImageTag.imageTag'] ]

  jobs:

  - job: Deploy

    steps:

    - script: |
        echo "Deploying image tag: $(imageTag)"
      displayName: "Deploy"
```

## Step 8 — Add Security Template

```yaml
- stage: Security

  dependsOn: Build

  jobs:

  - job: Security

    steps:

    - template: templates/security.yml
      parameters:
        environment: dev
```

## Step 9 — Add Production Condition

```yaml
condition: and(
  succeeded(),
  eq(variables['Build.SourceBranch'], 'refs/heads/main')
)
```

Production should also be protected by an Azure DevOps environment and its configured approvals/checks.

---

# 41. Interview Questions

## Q1. Difference between `${{ }}`, `$( )`, and `$[ ]`?

> `${{ }}` is mainly compile-time/template expansion, `$( )` is variable substitution, and `$[ ]` is runtime expression evaluation.

## Q2. Parameters vs variables?

> Parameters are useful for template structure and are generally evaluated during compilation. Variables are runtime-oriented configuration values.

## Q3. What are output variables?

> Output variables allow a job or stage to expose a calculated value so a dependent job or stage can consume it.

## Q4. How do you pass an image tag from Build to Deploy?

> Generate the tag in Build, mark it with `isOutput=true`, name the step, make Deploy depend on Build, and consume it through the appropriate `dependencies` or `stageDependencies` expression.

## Q5. How do you prevent production deployment from feature branches?

> Protect the main branch with PR policies, use environment permissions and approvals/checks, and add branch conditions where appropriate. Production should have multiple independent controls.

## Q6. What is a deployment job?

> A deployment job is a job type designed for deployment workflows and environment tracking. It can use deployment strategies such as `runOnce`.

## Q7. Why use environments?

> Environments provide deployment targets, deployment history, permissions, and controls such as approvals/checks.

## Q8. Why use YAML templates?

> Templates reduce duplication, standardize pipeline behavior, centralize security controls, and simplify maintenance across many repositories.

## Q9. How would you design templates for 50 microservices?

> Create centralized templates for common build, test, security, Docker, and deployment logic. Repositories provide service-specific parameters instead of copying entire pipelines.

## Q10. How do you troubleshoot a pipeline variable issue?

> Determine whether the value is a parameter, variable, runtime expression, secret, variable-group value, or output variable. Then verify scope, evaluation timing, dependencies, and syntax.

## Q11. How do you troubleshoot an output variable?

> Check `isOutput=true`, the step name, dependency relationship, and whether `dependencies` or `stageDependencies` is being used correctly.

## Q12. How do you make pipelines faster?

> Parallelize independent jobs, cache dependencies, optimize Docker builds, avoid unnecessary work, reuse templates, and measure stage duration.

## Q13. How do you securely authenticate to ACR?

> Use an appropriate service connection or workload identity/managed identity pattern where supported, with minimum required registry permissions.

## Q14. Why avoid `latest`?

> `latest` is mutable and makes deployments harder to trace and reproduce. Immutable build IDs or release versions provide better traceability.

## Q15. What is build-once-deploy-many?

> Produce one tested artifact/image and promote that exact artifact through environments rather than rebuilding separately.

## Q16. How do you integrate SonarQube?

> Prepare the scanner, build/test, run analysis, and publish/check the quality result using the supported SonarQube integration.

## Q17. How do you integrate Trivy?

> Build or obtain the image, scan it with an explicit vulnerability policy, and prevent promotion when policy requirements are violated.

## Q18. What if a self-hosted agent is offline?

> Check agent service, network, authentication, agent logs, OS health, disk, CPU, and memory.

## Q19. What if a service connection is unauthorized?

> Check pipeline authorization, identity permissions, resource scope, connection status, and whether the intended service connection is being used.

## Q20. How do you secure a production pipeline?

> Use protected branches, PR validation, least-privilege identities, secret management, security scanning, immutable artifacts, protected environments, approvals/checks, and auditability.

## Q21. How would you design CI/CD for 50 microservices?

> Standardize through reusable templates, keep service-specific configuration in each repository, build/test independently, scan source and images, publish versioned images to ACR, deploy consistently, and centralize governance without creating one giant pipeline.

## Q22. Artifact vs container image?

> A pipeline artifact can be any build output such as a ZIP or JAR. A container image is a packaged runtime artifact stored in a registry such as ACR.

## Q23. How do you prevent secrets in logs?

> Use secure secret stores and variables, avoid echoing sensitive values, minimize command-line exposure, and review scripts for accidental logging.

## Q24. Biggest mistake in enterprise YAML?

> Copy-pasting large amounts of pipeline logic across repositories. Reusable templates and centralized governance reduce inconsistent security and deployment behavior.

## Q25. How do you troubleshoot a failed production deployment?

> Determine impact and failing stage, inspect pipeline/deployment/application logs, verify configuration, secrets, identity, networking, image/version, and health checks, then follow rollback/roll-forward procedures and complete RCA.

---

# 42. Senior Scenario Questions

## Scenario 1 — Parameter Works but Variable Does Not

Parameters are compile-time constructs. If a value must change during execution, use an appropriate runtime variable or output variable.

## Scenario 2 — Production Uses Wrong Image

Problem:

```text
Build → myapp:1050
Deploy → myapp:latest
```

Better:

```text
Build
 ↓
imageTag = 1050
 ↓
Output Variable
 ↓
Deploy
 ↓
myapp:1050
```

## Scenario 3 — 50 Copy-Pasted Pipelines

Create:

```text
Central Templates
│
├── build.yml
├── test.yml
├── security.yml
├── docker.yml
└── deploy.yml
```

## Scenario 4 — Production Approval Is Bypassed

Do not rely only on YAML-controlled approvals.

Use:

```text
Protected Environment
        ↓
Approval / Checks
        ↓
Restricted Permissions
```

## Scenario 5 — ACR Works Locally but Fails in Pipeline

Check:

- Pipeline service connection
- Identity permissions
- ACR role assignment
- Registry name
- Pipeline authorization
- Network access
- Credential/token state

Local credentials do not prove that the pipeline identity has access.

---

# 43. Day 4 Checklist

- [ ] I understand compile-time expressions.
- [ ] I understand runtime variables.
- [ ] I understand `$[ ]`.
- [ ] I understand parameters vs variables.
- [ ] I understand conditions.
- [ ] I understand dependencies.
- [ ] I understand output variables.
- [ ] I can build a multi-stage pipeline.
- [ ] I understand deployment jobs.
- [ ] I understand environments.
- [ ] I understand approvals/checks.
- [ ] I can create YAML templates.
- [ ] I understand template parameters.
- [ ] I understand pipeline artifacts.
- [ ] I can build a Docker image in a pipeline.
- [ ] I understand ACR authentication.
- [ ] I understand SonarQube integration.
- [ ] I understand Trivy scanning.
- [ ] I can troubleshoot YAML failures.
- [ ] I can troubleshoot service connection failures.
- [ ] I can troubleshoot ACR authorization failures.
- [ ] I completed the Day 4 lab.
- [ ] I answered at least 20 interview questions aloud.

---

# 44. Homework Before Day 5

- [ ] Build the Day 4 multi-stage pipeline.
- [ ] Create a reusable security template.
- [ ] Practice passing an output variable between jobs.
- [ ] Practice passing an output variable between stages.
- [ ] Add a production environment.
- [ ] Configure appropriate approval/check controls if available.
- [ ] Build a Docker image.
- [ ] Push an image to ACR if available.
- [ ] Practice an explicit Trivy scan.
- [ ] Practice explaining `${{ }}`, `$( )`, and `$[ ]`.
- [ ] Record yourself answering 5 advanced Azure Pipelines questions.
- [ ] Draw your ideal enterprise CI/CD architecture.

---

# 45. Final Day 4 Challenge

Without looking at your notes, explain:

```text
Azure Repos
     ↓
Pull Request
     ↓
Branch Policy
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
Trivy
     ↓
ACR
     ↓
Dev
     ↓
QA
     ↓
Staging
     ↓
Production Approval
     ↓
Production
     ↓
Monitoring
```

Explain:

1. How the pipeline is triggered.
2. Which agent runs each job.
3. Why parameters differ from variables.
4. How output variables pass an image tag.
5. How stages depend on each other.
6. How the production environment is protected.
7. How the pipeline authenticates to ACR.
8. How SonarQube fits into CI.
9. How Trivy fits into container security.
10. How you would troubleshoot a failed deployment.

---

# 46. Day 4 Success Criteria

You are ready for **Day 5 — Docker Deep Dive** when you can write and explain a multi-stage Azure Pipeline without copying a template.

You should understand:

```text
Parameters
    ↓
Templates
    ↓
Variables
    ↓
Conditions
    ↓
Dependencies
    ↓
Output Variables
    ↓
Build
    ↓
Security
    ↓
Artifact/Image
    ↓
Deployment
```

---

# Next Day

## Day 5 — Docker Deep Dive

Topics:

- Docker architecture
- Images vs containers
- Dockerfile instructions
- Build context
- Layers and caching
- Multi-stage Docker builds
- Networking
- Volumes
- Environment variables
- Docker Compose
- Container security
- Non-root containers
- Read-only filesystem
- Linux capabilities
- Health checks
- Image optimization
- Docker registry
- ACR
- Trivy
- Production Dockerfile design
- Docker troubleshooting
- 25+ Docker interview questions
