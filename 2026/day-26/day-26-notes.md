### Task 1: Install and Authenticate

1. install the GitHub CLI on your machine
2. authenticate with your GitHub account using `gh auth login`
3. verify you're logged in and check which account is active using `gh auth status`
4. gh supports authentication via web-based login, SSH keys, and personal access tokens (PATs).

### Task 2: Working with Repositories
1. create a new GitHub repo directly from the terminal using `gh repo create <repo-name> --public --description "My test repo" --confirm`
2. clone a repo using `gh repo clone <username>/<repo-name>`
3. view details of one of your repos from the terminal using `gh repo view <username>/<repo-name>`
4. list all your repositories using `gh repo list <username>`
5. open a repo in your browser directly from the terminal using `gh repo view <username>/<repo-name> --web`
6. delete the test repo you created using `gh repo delete <username>/<repo-name> --confirm`

### Task 3: Issues
1. create an issue on one of your repos from the terminal using `gh issue create --title "Issue Title" --body "Issue body description" --label "bug"`
2. list all open issues on that repo using `gh issue list`
3. view a specific issue by its number using `gh issue view <issue-number>`
4. close an issue from the terminal using `gh issue close <issue-number> --comment "Closing this issue as it has been resolved."`
5. gh issue can be used in scripts or automation to create, view, and manage issues programmatically, allowing for integration with CI/CD pipelines or other automated workflows.

### Task 4: Pull Requests
1. create a branch, make a change, push it, and create a pull request entirely from the terminal using:
   - `git checkout -b new-branch`
   - make changes and commit them
   - `git push origin new-branch`
   - `gh pr create --title "My Pull Request" --body "Description of the changes"`
2. list all open PRs on a repo using `gh pr list`
3. view the details of your PR using `gh pr view <pr-number>` to check its status, reviewers, and checks
4. merge your PR from the terminal using `gh pr merge <pr-number> --merge`
5. gh pr merge supports merge methods such as `--merge`, `--squash`, and `--rebase`. To review someone else's PR using `gh`, you can use `gh pr checkout <pr-number>` to check out the PR branch locally, review the changes, and leave comments or approve it.

### Task 5: GitHub Actions & Workflows (Preview)
1. list the workflow runs on any public repo that uses GitHub Actions using `gh run list --repo <username>/<repo-name>`
2. view the status of a specific workflow run using `gh run view <run-id>`
3. gh run and gh workflow can be useful in a CI/CD pipeline to monitor the status of workflow runs, trigger workflows, and manage workflow configurations programmatically, allowing for better automation and integration with other tools.

### Task 6: Useful `gh` Tricks
1. `gh api` — make raw GitHub API calls from the terminal using `gh api <endpoint> --method <HTTP-method> --input <data>`
2. `gh gist` — create and manage GitHub Gists using `gh gist create <file> --public` and `gh gist list`
3. `gh release` — create and manage releases using `gh release create <tag> --title "Release Title" --notes "Release notes"`
4. `gh alias` — create shortcuts for commands you use often using `gh alias set <alias> <command>`
5. `gh search repos` — search GitHub repos from the terminal using `gh search repos <query> --limit <number>`