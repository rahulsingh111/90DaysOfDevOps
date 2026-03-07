# Day 23 task

# Task 1: Understanding Branches
1. A branch in Git is a separate line of development that allows you to work on different features or fixes without affecting the main codebase. It is essentially a pointer to a specific commit in the repository's history.
2. We use branches to keep our main codebase clean and stable. By working on a branch, we can make changes and test new features without risking the integrity of the main branch. This allows for better collaboration and easier management of code changes.
3. `HEAD` in Git is a reference to the current branch or commit that you are working on. It points to the latest commit in the current branch and allows you to keep track of your position in the repository's history.
4. When you switch branches, Git updates the files in your working directory to match the state of the branch you switched to. This means that any changes you made in the previous branch will not be visible in the new branch unless you commit those changes before switching.


# Task 2: Branching Commands — Hands-On
1. Creating and Switching Branches

```bash
# Create and switch to a new branch immediately
git checkout -b <branch_name>

# Switch back to an existing branch (like main)
git switch main
```


2. Committing Changes
Standard flow for saving your work locally.

```bash
# Check status of files
git status

# Stage all changes
git add .

# Commit with a descriptive message
git commit -m "your message here"
```

3. Pushing to GitHub (Setting Upstream)
If it's a brand new branch, Git needs to know where to send it on the server.

```bash
# Push and link your local branch to the remote (origin)
git push --set-upstream origin <branch_name>
```

1. Deleting Local Branches
Clean up your local workspace after you are done.

```bash
# Delete one or more local branches
git branch -d <branch_1> <branch_2>
```

5. Deleting Remote Branches (GitHub)
Deleting locally does not delete the branch on GitHub. You must push the deletion.

```bash
# Remove the branch from the remote server
git push origin --delete <branch_name>
```


### Task 3: Push to GitHub

What is the difference between `origin` and `upstream`?
- `origin` is the default name for the remote repository that we cloned from or initially set up. It typically refers to our own copy of the repository on GitHub. `upstream` is a convention used to refer to the original repository from which we forked. It allows us to keep track of changes in the original repository and pull updates from it.

### Task 4: Pull from GitHub
What is the difference between `git fetch` and `git pull`?
- `git fetch` retrieves the latest changes from the remote repository but does not merge them into your local branch. It allows you to review the changes before integrating them. `git pull`, on the other hand, is a combination of `git fetch` and `git merge`. It retrieves the latest changes and automatically merges them into your current branch, which can lead to conflicts if there are overlapping changes. 

### Task 5: Clone vs Fork

   - What is the difference between clone and fork? : clone creates a local copy of a repository on your machine, while fork creates a copy of the repository on your GitHub account. Cloning is typically used when you want to contribute to an existing project, while forking is used when you want to create your own version of a project or contribute to a project that you do not have write access to.
   
   - When would you clone vs fork? : would clone when you want to work on a project that you have access to and want to contribute directly. You would fork when you want to create your own version of a project or contribute to a project that you do not have write access to, allowing you to make changes and submit pull requests without affecting the original repository.
  
   - After forking, how do you keep your fork in sync with the original repo? : To keep your fork in sync with the original repository, you can add the original repository as a remote (often named `upstream`) and then fetch and merge changes from it. 