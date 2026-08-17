# Day 2 — Advanced Git

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** Advanced Git, branching strategies, merge/rebase, conflict resolution, recovery, tags, release workflows, and senior-level interview preparation  
> **Recommended time:** 3 hours

## 1. Objectives

By the end of Day 2 you should be able to:

- Explain Git's working directory, staging area, repository, HEAD, branches, and remote-tracking branches.
- Explain GitFlow and trunk-based development.
- Use `merge`, `rebase`, interactive rebase, `cherry-pick`, `revert`, `reset`, `stash`, and `reflog`.
- Resolve merge and rebase conflicts.
- Recover work after an accidental reset.
- Create release tags and understand semantic versioning.
- Use `.gitignore` and understand Git hooks.
- Explain enterprise Git workflows in interviews.

## 2. Three-Hour Schedule

| Time | Duration | Topic |
|---|---:|---|
| 00:00–00:30 | 30 min | Git internals and branches |
| 00:30–01:00 | 30 min | Branching strategies |
| 01:00–01:40 | 40 min | Merge, rebase, conflicts |
| 01:40–02:10 | 30 min | Recovery and advanced commands |
| 02:10–02:30 | 20 min | Tags and release workflow |
| 02:30–03:00 | 30 min | Interview Q&A |

---

# 3. Git Mental Model

```text
Working Directory
       ↓ git add
Staging Area
       ↓ git commit
Local Repository
       ↓ git push
Remote Repository
```

### Working Directory

Files you are currently editing.

### Staging Area

Files selected for the next commit:

```bash
git add main.py
```

### Commit

Creates a versioned snapshot:

```bash
git commit -m "Add login validation"
```

### HEAD

HEAD identifies the current checked-out commit or branch.

```bash
git status
git log --oneline --decorate
```

Example:

```text
a8c12ef (HEAD -> main) Add authentication
7b23abc Add database configuration
2d91abc Initial commit
```

### Branch

A branch is a movable reference to a commit:

```text
A---B---C  main
                   D---E  feature/login
```

Create and switch:

```bash
git switch -c feature/login
```

Remote branches:

```bash
git branch -a
```

---

# 4. Fetch vs Pull

## `git fetch`

```bash
git fetch origin
```

Downloads remote information without integrating it into the current branch.

```text
Remote
  ↓
fetch
  ↓
Remote-tracking refs
```

## `git pull`

```bash
git pull origin main
```

Normally means:

```text
fetch + merge/rebase
```

### Interview answer

> `git fetch` downloads remote changes without integrating them into my current branch. `git pull` fetches and then integrates the changes using merge or rebase depending on configuration.

---

# 5. Branching Strategies

## 5.1 GitFlow

Typical branches:

```text
main
develop
feature/*
release/*
hotfix/*
```

Example:

```text
feature
   ↓
develop
   ↓
release
   ↓
main
```

Create feature:

```bash
git switch -c feature/payment
```

Create release:

```bash
git switch develop
git switch -c release/2.0.0
```

Create hotfix:

```bash
git switch main
git switch -c hotfix/payment-timeout
```

### Advantages

- Formal release process
- Useful for multiple supported versions
- Clear release/hotfix branches

### Disadvantages

- More branches
- More merge complexity
- Longer-lived branches can create integration problems

---

# 6. Trunk-Based Development

Developers keep changes close to the main branch.

```text
              short feature
                   ↓
main ────────●─────●─────●─────●
```

Feature branches are normally short-lived.

Typical workflow:

```bash
git switch -c feature/login-validation
git add .
git commit -m "Add login validation"
git push -u origin feature/login-validation
```

Create a PR, run CI, review, and merge quickly.

### GitFlow vs Trunk-Based

| Area | GitFlow | Trunk-Based |
|---|---|---|
| Branches | More | Fewer |
| Feature lifetime | Can be longer | Usually short |
| Releases | Formal branches | Frequent releases |
| Merge complexity | Higher | Usually lower |
| CI requirement | Important | Very important |
| Best fit | Scheduled/multi-version releases | Frequent CI/CD |

### Senior interview answer

> I choose the branching model based on release frequency, number of supported versions, compliance requirements, and team workflow. Trunk-based development is often simpler for continuous delivery, while GitFlow can make sense when formal release or maintenance branches are required.

---

# 7. Merge vs Rebase

## Merge

```text
A---B---C---M
     \     /
      D---E
```

```bash
git switch main
git merge feature/login
```

Advantages:

- Preserves branch history
- Safe for shared branches
- Clearly shows integration points

## Rebase

Before:

```text
A---B---C  main
     \
      D---E  feature
```

After:

```text
A---B---C---D'---E'  feature
```

```bash
git switch feature
git rebase main
```

Advantages:

- Cleaner linear history
- Easier history navigation

Important:

> Rebase rewrites commit identities. Avoid rebasing commits that other developers already depend on unless your team explicitly allows it.

---

# 8. Practical Rebase Exercise

```bash
mkdir git-day2
cd git-day2

git init
git branch -M main

echo "# Git Day 2" > README.md
git add .
git commit -m "Initial commit"

git switch -c feature/login

echo "Login API" >> README.md
git add .
git commit -m "Add login API"

echo "Login validation" >> README.md
git add .
git commit -m "Add login validation"

git switch main

echo "Database" >> README.md
git add .
git commit -m "Add database configuration"

git switch feature/login
git rebase main

git log --oneline --graph --decorate --all
```

---

# 9. Interactive Rebase

Use:

```bash
git rebase -i HEAD~3
```

Common actions:

```text
pick    keep commit
reword  change message
edit    stop and modify
squash  combine with previous commit
fixup   combine and discard message
drop    remove commit
```

Example:

```text
pick   Add login API
squash Fix typo
squash Add validation
```

Result:

```text
Add complete login functionality
```

Use interactive rebase mainly for local/unshared history unless your team has a controlled policy for rewriting shared history.

---

# 10. Cherry-Pick

Suppose a production fix is one commit:

```text
main:   A---B---C
hotfix: A---B---D
```

Apply only `D`:

```bash
git switch main
git cherry-pick <commit-id>
```

Result:

```text
A---B---C---D'
```

Useful for:

- Backporting bug fixes
- Applying isolated production fixes
- Moving a targeted change

Risks:

- Dependencies may be missing
- Conflicts may occur
- Duplicate changes can appear

Always test after cherry-picking.

---

# 11. Merge Conflicts

Git creates a conflict when it cannot automatically combine changes.

Example:

```text
<<<<<<< HEAD
print("Hello")
=======
print("Hello DevOps")
>>>>>>> feature
```

## Resolution

1. Check conflicts:

```bash
git status
```

2. Open the file.
3. Understand both changes.
4. Write the correct final code.
5. Remove conflict markers.
6. Stage:

```bash
git add <file>
```

7. Complete merge:

```bash
git commit
```

For rebase:

```bash
git rebase --continue
```

Abort merge:

```bash
git merge --abort
```

Abort rebase:

```bash
git rebase --abort
```

### Best practice

Do not blindly select "ours" or "theirs". Understand the intended application behavior, resolve the conflict, and run tests.

---

# 12. Reset

Three important modes:

```bash
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

### Soft

Moves HEAD but keeps changes staged.

### Mixed

Moves HEAD and leaves changes in the working tree unstaged. This is the default reset mode.

### Hard

Moves HEAD and changes the working tree.

```bash
git reset --hard HEAD~1
```

This can discard uncommitted changes. Use carefully, especially on shared branches.

---

# 13. Revert

```bash
git revert <commit>
```

Creates a new commit that reverses the selected commit.

```text
A---B---C---R
        bad  ↑
             reverse C
```

Use revert when the commit is already shared.

### Interview answer

> Revert creates a new commit that reverses an earlier commit, so it preserves an auditable history. Reset moves the branch reference and can rewrite history.

---

# 14. Reflog Recovery

If you accidentally run:

```bash
git reset --hard HEAD~3
```

check:

```bash
git reflog
```

Example:

```text
abc123 HEAD@{0}: reset: moving to HEAD~3
def456 HEAD@{1}: commit: Add payment validation
```

Inspect:

```bash
git show def456
```

Recover:

```bash
git switch -c recovery/payment def456
```

Important:

> Reflog is local and entries can eventually be pruned. It is a recovery mechanism, not a substitute for remote backups.

---

# 15. Git Tags

Tags mark important commits, commonly releases.

Lightweight:

```bash
git tag v1.0.0
```

Annotated:

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
```

List:

```bash
git tag
```

Inspect:

```bash
git show v1.0.0
```

Push:

```bash
git push origin v1.0.0
```

All tags:

```bash
git push origin --tags
```

---

# 16. Semantic Versioning

Common format:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
2.4.1
```

- `2` = major
- `4` = minor
- `1` = patch

Typical meaning:

```text
1.5.0 → 2.0.0   Breaking change
1.5.0 → 1.6.0   Backward-compatible feature
1.5.0 → 1.5.1   Bug fix
```

---

# 17. Release Workflow

```text
Feature Branch
      ↓
Pull Request
      ↓
CI Validation
      ↓
Merge
      ↓
Build
      ↓
Version
      ↓
Tag
      ↓
Artifact
      ↓
Deploy
      ↓
Monitor
```

Example:

```bash
git switch main
git pull
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0
```

---

# 18. `.gitignore`

Example:

```gitignore
# Python
__pycache__/
*.pyc
.venv/

# Node
node_modules/

# Environment
.env
.env.*

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Logs
*.log
```

Important:

> `.gitignore` does not remove a secret that has already been committed. If a credential was committed, rotate/revoke it immediately and handle repository history according to your security process.

---

# 19. Git Hooks

Common hooks:

```text
pre-commit
commit-msg
pre-push
post-merge
```

Possible uses:

- Formatting
- Linting
- Unit tests
- Commit message validation
- Secret detection

Example:

```text
git commit
    ↓
pre-commit
    ↓
Lint
    ↓
Tests
    ↓
Commit
```

Important:

> Client-side hooks can be bypassed, so important controls should also run in CI.

---

# 20. Enterprise Remote Workflow

```text
Developer
   ↓
Feature Branch
   ↓
Commit
   ↓
Push
   ↓
Pull Request
   ↓
CI
   ↓
Code Review
   ↓
Approval
   ↓
Merge
   ↓
CD
```

Commands:

```bash
git clone <repository-url>

git switch -c feature/login

git add .
git commit -m "Add login"

git push -u origin feature/login
```

---

# 21. Complete Day 2 Hands-on Lab

## Objective

Practice:

- Feature branching
- Multiple commits
- Rebase
- Merge conflict
- Revert
- Cherry-pick
- Reflog
- Release tagging

### Step 1 — Repository

```bash
mkdir day2-git-lab
cd day2-git-lab

git init
git branch -M main

echo "# Day 2 Git Lab" > README.md
echo "Application" > app.txt

git add .
git commit -m "Initial application"
```

### Step 2 — Feature

```bash
git switch -c feature/login

echo "Login API" >> app.txt
git add .
git commit -m "Add login API"

echo "Login validation" >> app.txt
git add .
git commit -m "Add login validation"
```

### Step 3 — Main Change

```bash
git switch main

echo "Database configuration" >> app.txt
git add .
git commit -m "Add database configuration"
```

### Step 4 — Rebase

```bash
git switch feature/login
git rebase main

git log --oneline --graph --decorate --all
```

### Step 5 — Practice a Conflict

Change the same line on `main` and `feature/login`, commit both changes, then:

```bash
git switch feature/login
git rebase main
```

Resolve the conflict, then:

```bash
git add app.txt
git rebase --continue
```

Or cancel:

```bash
git rebase --abort
```

### Step 6 — Hotfix + Cherry-Pick

```bash
git switch main
git switch -c hotfix/login

echo "Login timeout fix" >> app.txt
git add .
git commit -m "Fix login timeout"

git log --oneline -3
```

Then:

```bash
git switch main
git cherry-pick <commit-id>
```

### Step 7 — Release Tag

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git tag
git show v1.0.0
```

### Step 8 — Revert

Create a bad commit:

```bash
echo "Bad production configuration" >> app.txt
git add .
git commit -m "Bad production change"
```

Reverse it:

```bash
git revert HEAD
```

### Step 9 — Reflog

```bash
git reflog
```

Find an earlier commit:

```bash
git show <commit-id>
```

Recover it:

```bash
git switch -c recovery/test <commit-id>
```

---

# 22. Interview Questions and Answers

## Q1. Git vs GitHub?

> Git is the distributed version-control system. GitHub is a hosting and collaboration platform for Git repositories.

## Q2. What happens during `git commit`?

> Git takes the staged changes, creates the required Git objects and commit metadata, records the parent commit, and moves the current branch reference to the new commit.

## Q3. What is HEAD?

> HEAD identifies the current checked-out commit or branch.

## Q4. What is detached HEAD?

> Detached HEAD occurs when HEAD points directly to a commit instead of a branch.

Example:

```bash
git checkout <commit-id>
```

If you create useful commits there, create a branch:

```bash
git switch -c recovery-branch
```

## Q5. Merge vs rebase?

> Merge combines histories and preserves the branch graph. Rebase replays commits onto another base and produces a linear history, but rewrites commit IDs.

## Q6. When would you use rebase?

> I use rebase to update a short-lived feature branch with the latest main branch and clean up local commits before sharing, according to team policy.

## Q7. When should you avoid rebase?

> Avoid rebasing commits already shared and depended on by other developers because it rewrites history.

## Q8. What is interactive rebase?

> It allows me to reorder, squash, reword, edit, or drop commits.

## Q9. What is cherry-pick?

> It applies the changes from a specific commit to the current branch.

## Q10. Why can cherry-pick be dangerous?

> The commit may depend on other commits, which can create conflicts or incomplete behavior. I verify dependencies and test afterward.

## Q11. Revert vs reset?

> Revert creates a new reversing commit and is safer for shared branches. Reset moves the branch reference and can rewrite history.

## Q12. What is reflog?

> Reflog records local movements of Git references and can help recover work after accidental reset or rebase.

## Q13. Can reflog recover everything?

> No. It is local and entries are eventually pruned. It is not a replacement for backups.

## Q14. What is a Git tag?

> A tag marks a specific point in repository history, commonly a release such as `v1.0.0`.

## Q15. Lightweight vs annotated tag?

> A lightweight tag is a simple reference to a commit. An annotated tag is a Git object containing metadata such as tagger, date, message, and optionally a signature.

## Q16. What is GitFlow?

> GitFlow uses branches such as main, develop, feature, release, and hotfix to organize development and formal releases.

## Q17. What is trunk-based development?

> It keeps changes close to the main branch using short-lived branches or direct integration with strong CI controls.

## Q18. Which is better: GitFlow or trunk-based?

> Neither is universally better. The choice depends on release frequency, supported versions, compliance, and team workflow. Trunk-based development is often simpler for high-frequency CI/CD.

## Q19. How do you resolve a conflict?

> Identify conflicting files, understand both changes, implement the correct final behavior, remove markers, stage the result, complete the merge/rebase, and run tests.

## Q20. How do you abort rebase?

```bash
git rebase --abort
```

## Q21. How do you abort merge?

```bash
git merge --abort
```

## Q22. What is `.gitignore`?

> It specifies files Git should normally not track, such as build output, dependencies, IDE files, logs, and local environment files.

## Q23. Does `.gitignore` remove an already tracked secret?

> No. Rotate/revoke the secret and handle history according to the organization's security process.

## Q24. What are Git hooks?

> Scripts triggered by Git lifecycle events such as pre-commit, commit-msg, and pre-push.

## Q25. Are Git hooks sufficient for security?

> No. Client-side hooks can be bypassed. Critical validation should also run in CI.

---

# 23. Senior-Level Scenarios

## Scenario 1 — Shared Branch Was Rebased

**Question:** A developer rebased a branch already used by several developers. What can happen?

**Answer:**

Rebase changes commit IDs and can cause:

- Duplicate commits
- Conflicts
- Synchronization problems
- Force-push requirements

Avoid rewriting shared history unless the team has an explicit workflow for it.

---

## Scenario 2 — Production Hotfix

**Question:** Production has a critical bug, but `develop` contains unfinished features. How can you release only the fix?

**Answer:**

```text
main
 ↓
hotfix
 ↓
Fix
 ↓
CI
 ↓
Test
 ↓
Release
```

Then synchronize the fix back into development according to the team's workflow. If the fix already exists as an isolated commit, cherry-pick may be appropriate.

---

## Scenario 3 — Accidental Reset

**Question:** A developer accidentally ran `git reset --hard` and lost commits.

**Answer:**

```bash
git reflog
git show <commit-id>
git switch -c recovery <commit-id>
```

Then restore the intended branch state using the team's process.

---

## Scenario 4 — Huge Merge Conflict

**Question:** A feature branch has been open for three months and now has hundreds of conflicts. What does this indicate?

**Answer:**

It indicates delayed integration and potentially oversized changes.

Improve with:

- Short-lived branches
- Frequent integration
- Smaller PRs
- Feature flags
- Trunk-based development
- Strong CI

---

## Scenario 5 — One Production Fix

**Question:** A bug fix exists as one isolated commit on another branch. Production needs only that fix.

**Answer:**

```bash
git cherry-pick <commit-id>
```

Verify dependencies and test the result.

---

## Scenario 6 — Release Strategy

**Question:** Your organization releases every two weeks and maintains three production versions. Would you automatically choose trunk-based development?

**Answer:**

Not automatically. Evaluate:

- Release frequency
- Supported versions
- Backport requirements
- Compliance
- Team workflow
- Deployment automation

Release branches may still be useful for maintaining multiple versions.

---

# 24. Day 2 Self-Assessment

- [ ] I understand Working Directory.
- [ ] I understand Staging Area.
- [ ] I understand commits.
- [ ] I understand HEAD.
- [ ] I understand branches.
- [ ] I understand remote-tracking branches.
- [ ] I understand `fetch` vs `pull`.
- [ ] I understand GitFlow.
- [ ] I understand trunk-based development.
- [ ] I understand merge.
- [ ] I understand rebase.
- [ ] I practiced interactive rebase.
- [ ] I understand cherry-pick.
- [ ] I resolved a merge conflict.
- [ ] I practiced `merge --abort`.
- [ ] I practiced `rebase --abort`.
- [ ] I understand reset.
- [ ] I understand revert.
- [ ] I practiced reflog recovery.
- [ ] I created a Git tag.
- [ ] I understand semantic versioning.
- [ ] I understand `.gitignore`.
- [ ] I understand Git hooks.
- [ ] I completed the Day 2 lab.
- [ ] I answered at least 15 interview questions aloud.

---

# 25. Homework Before Day 3

- [ ] Create `git-day2-lab`.
- [ ] Practice feature branching.
- [ ] Create at least 3 commits.
- [ ] Practice rebase.
- [ ] Create and resolve a merge conflict.
- [ ] Practice `git revert`.
- [ ] Practice `git cherry-pick`.
- [ ] Practice `git reflog`.
- [ ] Create tag `v1.0.0`.
- [ ] Explain GitFlow vs trunk-based development aloud.
- [ ] Record yourself answering 5 Git interview questions.
- [ ] Write down one real Git problem you have experienced and how you solved it.

---

# 26. Final Day 2 Challenge

Without looking at your notes, explain:

```text
Developer
   ↓
Feature Branch
   ↓
Commit
   ↓
Push
   ↓
Pull Request
   ↓
CI
   ↓
Code Review
   ↓
Merge
   ↓
Tag
   ↓
Release
```

Then explain what you would do if:

1. The branch has conflicts.
2. The last commit is bad.
3. A production fix exists on another branch.
4. A developer accidentally resets the branch.
5. A developer rebases a shared branch.
6. You need to release version `2.3.1`.
7. A secret was accidentally committed.

---

# 27. Day 2 Success Criteria

You are ready for **Day 3 — Azure DevOps** when you can confidently explain:

```text
Working Directory
       ↓
Staging Area
       ↓
Commit
       ↓
Branch
       ↓
Push
       ↓
Pull Request
       ↓
CI
       ↓
Merge
       ↓
Tag
       ↓
Release
```

And you can explain:

- `fetch` vs `pull`
- `merge` vs `rebase`
- `revert` vs `reset`
- `cherry-pick`
- `stash`
- `reflog`
- GitFlow
- Trunk-based development
- Merge conflict resolution
- Release tagging

---

# Next Day

## Day 3 — Azure DevOps

Topics:

- Azure DevOps architecture
- Azure Repos
- Azure Boards
- Azure Pipelines
- YAML pipelines
- Pipeline stages and jobs
- Agents and agent pools
- Microsoft-hosted vs self-hosted agents
- Variables and variable groups
- Secrets
- Service connections
- Environments
- Approvals and checks
- Artifacts
- Templates
- Pipeline security
- Real enterprise CI/CD design
- 20+ Azure DevOps interview questions
