### Task 1: Git Merge — Hands-On
1. A fast-forward merge occurs when the branch being merged has not diverged from the branch it is being merged into. In this case, Git simply moves the pointer of the main branch forward to the latest commit of the feature branch, resulting in a linear history without a merge commit.
2. Git creates a merge commit when the branches have diverged, meaning there are commits on both branches that are not present in the other. In this case, Git combines the changes from both branches and creates a new commit that represents the merge, preserving the history of both branches.
3. A merge conflict occurs when Git is unable to automatically reconcile differences between two branches being merged. This typically happens when changes have been made to the same line of code in both branches. To resolve a merge conflict, you must manually edit the conflicting files to choose which changes to keep, and then commit the resolved changes.

### Task 2: Git Rebase — Hands-On
- Rebase takes the commits from the feature branch and re-applies them on top of the main branch, effectively rewriting the commit history. This results in a linear history without merge commits, making it appear as if the feature branch was developed directly on top of the main branch.
- The history after a rebase is different from a merge because it does not create a merge commit. Instead, it creates new commits that represent the same changes but with different commit hashes, as they are now based on the latest commit of the main branch.
- You should never rebase commits that have been pushed and shared with others because it rewrites the commit history, which can cause confusion and conflicts for others who have based their work on the original commits. It can lead to lost work and difficulties in collaboration.
- You would use rebase when you want to maintain a clean, linear history, especially for feature branches that are not yet shared with others. Merge is preferred when you want to preserve the context of the branch and its history, especially for collaborative work where multiple developers are contributing to the same branch.

### Task 3: Squash Commit vs Merge Commit
squash merging combines all the commits from a feature branch into a single commit before merging it into the main branch. This results in a cleaner history with fewer commits, making it easier to understand the changes made by the feature branch.
- Squash merging is useful when you want to condense multiple small commits (like typo fixes or formatting changes) into a single commit that represents the overall feature or change. This can make the commit history more concise and easier to read.
- Regular merging preserves the individual commits from the feature branch, which can be useful for understanding the development process and for debugging purposes. It allows you to see the incremental changes made during the development of the feature.

- The trade-off of squashing is that it loses the granular history of the individual commits, which can make it harder to trace specific changes or understand the development process. It can also make it more difficult to revert specific changes if needed.

### Task 5: Cherry Picking
cherry picking allows you to select specific commits from one branch and apply them to another branch. This is useful when you want to incorporate specific changes without merging the entire branch, especially for hotfixes or urgent updates.
- You would use cherry-pick in a real project when you need to apply a specific change from one branch to another without merging all the changes from the source branch. This is often used for bug fixes or urgent updates that need to be applied to multiple branches.
- What can go wrong with cherry-picking is that it can lead to duplicate commits if the same changes are later merged from the source branch. It can also create conflicts if the changes being cherry-picked depend on other changes that are not present in the target branch. Additionally, it can make the commit history more complex and harder to follow if overused and not managed carefully.