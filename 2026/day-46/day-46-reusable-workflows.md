### Task 1: Understand `workflow_call`
a reusable workflow is a workflow that can be called by other workflows, allowing for code reuse and modularization of CI/CD processes. 
The `workflow_call` trigger is used to define a reusable workflow that can be invoked by other workflows. 
Calling a reusable workflow is different from using a regular action because it allows for more complex workflows to be shared and reused across multiple repositories or projects, while regular actions are typically single-purpose and self-contained. 
A reusable workflow file must live in the `.github/workflows/` directory of the repository.

complete reusable workflow:
```
name: reusable-workflow
on:
    workflow_call:
        inputs:
            app_name:
                type: string
                required: true
            environment:
                type: string
                required: true
                default: staging
        secrets:
            DOCKERHUB_TOKEN:
                required: true
        
        outputs:
            build_version:
                description: "Generated build version"
                value: ${{ jobs.check.outputs.build_version }}

jobs:
    check:
        runs-on: ubuntu-latest

        outputs:
          build_version: ${{ steps.version.outputs.build_version }}

        steps:
            - name: checkout code
              uses: actions/checkout@v5
            - name: Build application
              run: |
                echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"
            
            - name: Check Docker token
              env:
                DOCKER_TOKEN: ${{ secrets.DOCKER_TOKEN }}
              run: |
                    if [ -n "$DOCKER_TOKEN" ]; then
                        echo "Docker token is set: true"
                    else
                        echo "Docker token is set: false"
                    fi

            - name: generate build version
              id: version
              run: |
                SHORT_SHA=$(git rev-parse --short HEAD)
                VERSION="v1.0-${SHORT_SHA}"

                echo "Build version: $VERSION"
                echo "build_version=$VERSION" >> "$GITHUB_OUTPUT"
```

complete call-build workflow:
```
name: call-build
on:
    push:
        branches:
            - main
jobs:
    build:
        uses: ./.github/workflows/reusable-workflow.yml
        with:
            app_name: "my-web-app"
            environment: "production"
        secrets:
            DOCKERHUB_TOKEN: ${{ secrets.DOCKER_TOKEN }}

    display-version:
        runs-on: ubuntu-latest
        needs: build
        steps:
            - name: Print build version
              run: 'echo "Build version is: ${{ needs.build.outputs.build_version }}"'
```

action file:
```
name: Setup and Greet
description: Prints a greeting, date, and runner OS

inputs:
  name:
    description: "Name of the person to greet"
    required: true

  language:
    description: "Greeting language"
    required: false
    default: "en"

outputs:
  greeted:
    description: "Whether the greeting was completed"
    value: ${{ steps.greeting.outputs.greeted }}

runs:
  using: composite

  steps:
    - name: Print greeting
      id: greeting
      shell: bash
      run: |
        case "${{ inputs.language }}" in
          en)
            echo "Hello, ${{ inputs.name }}!"
            ;;
          es)
            echo "Hola, ${{ inputs.name }}!"
            ;;
          fr)
            echo "Bonjour, ${{ inputs.name }}!"
            ;;
          de)
            echo "Hallo, ${{ inputs.name }}!"
            ;;
          hi)
            echo "Namaste, ${{ inputs.name }}!"
            ;;
          *)
            echo "Hello, ${{ inputs.name }}!"
            ;;
        esac

        echo "greeted=true" >> "$GITHUB_OUTPUT"

    - name: Print date and runner OS
      shell: bash
      run: |
        echo "Current date: $(date)"
        echo "Runner OS: $RUNNER_OS"
```

### Task 6: Reusable Workflow vs Composite Action
|                                  | **Reusable Workflow**                       | **Composite Action**                       |
| -------------------------------- | ------------------------------------------- | ------------------------------------------ |
| **Triggered by**                 | `workflow_call`                             | `uses:` in a step                          |
| **Can contain jobs?**            | ✅ Yes                                       | ❌ No                                       |
| **Can contain multiple steps?**  | ✅ Yes                                       | ✅ Yes                                      |
| **Lives where?**                 | `.github/workflows/`                        | `.github/actions/<action-name>/action.yml` |
| **Can accept secrets directly?** | ✅ Yes, via `workflow_call.secrets`          | ❌ No, not directly                         |
| **Best for**                     | Reusing **entire CI/CD workflows and jobs** | Reusing **a group of steps within a job**  |
