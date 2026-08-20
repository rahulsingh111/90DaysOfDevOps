### Task 1: Trigger on Pull Request
```
name: pr-check
on:
    pull_request:
        branches:
            - main
        types:
            - opened
            - reopened
            - synchronize
jobs:
    name_show:
        runs-on: ubuntu-latest
        steps:
            - name: show running branch
              run: 'echo "pr check running for branch: ${{ github.head_ref }}"'
```

yes it is showing in the PR page

### Task 2: Scheduled Trigger
cron expression for every Monday at 9 AM is: `0 9 * * 1`
```
name: scheduled
on:
    schedule:
    - cron: '0 2 * * *'
jobs:
    show-name:
        runs-on: ubuntu-latest
        steps:
            - name: show name
              run: echo " hello Rahul"
```

### Task 3: Manual Trigger
```
name: manual trigger
on:
    workflow_dispatch:
        inputs:
            environment:
                description: "environment to deploy to"
                required: true
                type: choice
                options:
                    - staging
                    - development

jobs:
    environment-input:
        runs-on: ubuntu-latest
        environment: ${{ github.event.inputs.environment }}
    
        steps:
            - name: Deploy to ${{ github.event.inputs.environment }}
              run: echo "Deploying to ${{ github.event.inputs.environment }}"

```

### Task 4: Matrix Builds
```
name: matrix strategy
on:
    push:
        branches:
            - main
    workflow_dispatch:
jobs:
    python-version:
        runs-on: ${{ matrix.os }}
        strategy:
            fail-fast: true
            matrix:
                os: [ubuntu-latest, windows-latest]
                python-version: ["3.10", "3.11", "3.12"]
                exclude:
                    - os: windows-latest
                      python-version: "3.10"
        steps:
            - uses: actions/checkout@v4
            - name: setup python ${{ matrix.python-version }}
              uses: actions/setup-python@v5
              with:
                python-version: ${{ matrix.python-version }}
            - name: Run tests
              run: python --version
```
### Task 5: Exclude & Fail-Fast
fail-fast: true (the default) means that if one job fails, the rest of the jobs will be canceled.
fail-fast: false means that if one job fails, the rest of the jobs will continue to run. This allows you to see the results of all jobs, even if one fails.