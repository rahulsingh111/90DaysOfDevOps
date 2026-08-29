### Task 1: Pull Request Event Types
```
name: pr-lifecycle
on:
    pull_request:
        types: [opened, synchronize, reopened, closed]
jobs:
    print-pr-info:
        runs-on: ubuntu-latest
        steps:
            - name: print event type
              run: 'echo "Event action: ${{ github.event.action }}"'
            - name: print pr title
              run: 'echo "PR Title: ${{ github.event.pull_request.title }}"'
            - name: Print PR Author
              run: 'echo "PR Author: ${{ github.event.pull_request.user.login }}"'
            - name: print branches
              run: |
                echo "Source: ${{ github.head_ref }}"
                echo "Target: ${{ github.base_ref }}"
    run-on-merge:
        if: github.event.pull_request.merged == true
        runs-on: ubuntu-latest
        steps:
            - name: post-merge actions
              run: echo "Pull request was successfully merged from dev"
```

### Task 2: PR Validation Workflow
```
name: pr-checks
on:
    pull_request:
        branches:
            - main
jobs:
    file-size-check:
        runs-on: ubuntu-latest
        steps:
            - name: check the code
              uses: actions/checkout@v5
            - name: Fail if new files are committed over 1 MB
              uses: pelotech/github-action-file-size-checker@v0.2.1
              with:
                max_file_size_kib: '1024'

    branch-name-check:
        runs-on: ubuntu-latest
        steps:
            - name: read the branch
              run: |
                BRANCH_NAME="${{ github.head_ref }}"
                if [[ ! "$BRANCH_NAME" =~ ^(feature|fix|docs)/.+$ ]]; then
                    echo "::error::Branch name '$BRANCH_NAME' does not follow the required format."
                    echo "Expected format: feature/*, fix/*, or docs/*"
                    exit 1
                fi
                echo "Branch name '$BRANCH_NAME' is valid."

    pr-body-check:
        runs-on: ubuntu-latest
        steps:
            - name: Check PR description
              run: |
                PR_BODY="${{ github.event.pull_request.body }}"

                if [ -z "$PR_BODY" ]; then
                    echo "::warning::PR description is empty. Please add a description."
                else
                    echo "PR description is present."
                fi
#pushing from lol
```

### Task 3: Scheduled Workflows (Cron Deep Dive)
```
name: scheduled-tasks
on:
    schedule:
        - cron: '30 2 * * 1'
        - cron: '0 */6 * * *'
    workflow_dispatch:
jobs:
    run-scheduled-task:
        runs-on: ubuntu-latest
        steps:
            - name:  Print Trigger Schedule
              run: 'echo "Schedule triggered: ${{ github.event.schedule }}"'

            - name: Health check
              run: |
                HTTP_STATUS=$(curl -L -s -o /dev/null -w "%{http_code}" https://google.com)

                echo "HTTP response code: $HTTP_STATUS"

                if [ "$HTTP_STATUS" -ne 200 ]; then
                    echo "::error::Health check failed with HTTP status $HTTP_STATUS"
                    exit 1
                fi

                echo "Health check passed."
```

### Task 4: Path & Branch Filters
```
name: smart-triggers
on:
    push:
        branches:
            - main
            - 'release/*'
        paths:
            - 'src/**'
            - 'app/**'
jobs:
    checking-workflow:
        runs-on: ubuntu-latest
        steps:
            - name: checkout code
              uses: actions/checkout@v4
```

I would use paths when I want to trigger a workflow only when specific files or directories are changed, ensuring that the workflow runs only for relevant changes. On the other hand, I would use paths-ignore when I want to skip running a workflow for certain files or directories, such as documentation or configuration files, to avoid unnecessary runs for changes that don't affect the main functionality of the project.

### Task 5: `workflow_run` — Chain Workflows Together
```
name: testing
on:
    push:
        branches:
            - main
jobs:
    run-tests:
        runs-on: ubuntu-latest
        steps:
            - name: test code
              uses: actions/checkout@v4
            - name: checking
              run: |
                echo "running tests"
```

```
name: deployaftertest
on:
    workflow_run:
        workflows: ["testing"]
        types: [completed]
jobs:
    deploy:
        if: ${{ github.event.workflow_run.conclusion == 'success' }}
        runs-on: ubuntu-latest
        steps:
            - name: Deploy
              run: echo "tests succeded. Deploying"
```

### Task 6: `repository_dispatch` — External Event Triggers
```
name: external triggers
on:
    repository_dispatch:
      types: [deploy-request]
jobs:
    client-payload:
        runs-on: ubuntu-latest
        steps:
            - name: Print the client payload
              run: echo "${{ github.event.client_payload.environment }}"
```

I would use an external system to trigger a pipeline when I want to integrate GitHub Actions with other tools or services, such as a Slack bot, monitoring tool, or any external application that needs to initiate a workflow based on specific events or conditions outside of GitHub. This allows for more dynamic and responsive automation workflows that can react to real-time events in the broader ecosystem.

`workflow run` is useful when I want to create a sequence of workflows where one workflow depends on the successful completion of another. This is particularly helpful for scenarios like running tests before deployment, ensuring that the deployment only occurs if the tests pass, thus maintaining the integrity and stability of the application.
`workflow call` is useful when I want to reuse a workflow across multiple workflows or repositories. It allows me to define a common set of steps in a single workflow and call it from other workflows, promoting modularity and reducing duplication. This is beneficial for maintaining consistency and simplifying updates, as changes made to the called workflow will automatically propagate to all workflows that invoke it.