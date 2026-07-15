### Task 1: Git Reset — Hands-On
soft heads back to the previous commit but keeps the changes in the working directory and staging area. Mixed resets back to the previous commit and keeps the changes in the working directory but unstages them. Hard resets back to the previous commit and discards all changes in the working directory and staging area.
hard is destructive because it permanently removes changes from the working directory and staging area, making it impossible to recover them unless they were committed elsewhere.
Use `--soft` when you want to undo a commit but keep your changes staged for a new commit. Use `--mixed` when you want to undo a commit and unstage the changes, allowing you to modify them before committing again. Use `--hard` when you want to completely discard changes and reset your working directory to a clean state.
we should avoid using `git reset` on commits that are already pushed to a shared repository, as it can rewrite history and cause issues for other collaborators. Instead, use `git revert` for shared branches.

### Task 2: Git Revert — Hands-On
git revert is different from git reset in that it creates a new commit that undoes the changes of a previous commit, rather than removing the commit from history. This makes it safer for shared branches, as it preserves the commit history and avoids rewriting it.
revert is considered safer than reset for shared branches because it does not alter the commit history, which can lead to confusion and conflicts for other collaborators. Instead, it adds a new commit that negates the changes of the specified commit.
revert is typically used when you want to undo changes in a shared branch without affecting the commit history, while reset is used for local branches or when you want to completely remove commits from history.

### Task 3: Reset vs Revert — Summary
`git reset`: goes back to a previous commit and can modify the working directory and staging area depending on the reset type. It removes commits from history and is not safe for shared/pushed branches. Use it for local branches or when you want to completely remove commits.
`git revert`: creates a new commit that undoes the changes of a previous commit. It does not remove commits from history and is safe for shared/pushed branches. Use it when you want to undo changes without affecting the commit history.

### Task 4: Branching Strategies
`GitFlow` uses multiple long-lived branches. Development happens in feature branches, which are merged into develop. Releases are prepared in release branches, and urgent production fixes are made in hotfix branches. Production code always lives in main.

Flow
                 feature/login
                /
main ---------●----------------------●----------
               \                    /
                develop -----------●---------
                    \             /
                     release ----●
                          \
                           hotfix


Branches
main → Production-ready code
develop → Integration branch
feature/* → New features
release/* → Release preparation
hotfix/* → Emergency production fixes
Used in
Enterprise applications
Banking
Healthcare
Large organizations
Teams with scheduled releases
Pros
Very organized
Stable production branch
Easy release management
Supports parallel development
Cons
Complex workflow
Many branches to manage
Slow release cycle
Not ideal for Continuous Deployment

`GitHub Flow` Everything starts from main. Create a feature branch, make changes, open a Pull Request, review, merge, and deploy.

Flow:
main
 │
 ├── feature/login
 │      │
 │      └── Pull Request
 │              │
 └──────────────┘
       merge

Used in
SaaS companies
Startups
Continuous Deployment
Small to medium teams
Pros
Very simple
Easy to learn
Fast deployments
Encourages code reviews
Cons
No dedicated release branch
Less suitable for multiple production versions
Requires a stable CI/CD pipeline

`Trunk-Based Development (TBD)` Everyone commits directly to main (the trunk) or uses very short-lived feature branches that are merged quickly—typically within a day.

Flow
main
 │
 ├── small feature
 ├── bug fix
 ├── refactor
 ├── documentation
 └── merge quickly

Used in
Google
Meta
Netflix
Amazon
Teams practicing Continuous Integration
Pros
Frequent integration
Fewer merge conflicts
Faster delivery
Excellent for CI/CD
Cons
Requires strong automated testing
Needs feature flags for incomplete work
Riskier without disciplined engineering practices

`1. Which strategy would you use for a startup shipping fast?`

GitHub Flow

Reason:

Simple workflow
Quick feature delivery
Continuous deployment
Easy collaboration
Minimal overhead


`2. Which strategy would you use for a large team with scheduled releases?`

GitFlow

Reason:

Dedicated develop, release, and hotfix branches
Better release planning
Easier maintenance of production versions
Suitable for large teams and regulated environments


`3. Which one does your favorite open-source project use?`

A good example is Kubernetes.

Primary branch: master/main (depending on repository)
Development happens in short-lived feature branches
Changes are submitted via Pull Requests
After review and CI checks, they are merged into the main branch

This closely resembles GitHub Flow. The project also maintains separate release branches (such as release-1.xx) for supported versions, but day-to-day contribution follows a GitHub Flow–style process rather than full GitFlow.