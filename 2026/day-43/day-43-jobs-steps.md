### Task 1: Multi-Job Workflow
today I learnt how to create a multi-job workflow in GitHub Actions. I created a workflow file named `.github/workflows/multi-job.yml` that contains three jobs: `build`, `test`, and `deploy`. The `test` job is set to run only after the `build` job succeeds, and the `deploy` job runs only after the `test` job succeeds.


### Task 2: Environment Variables
I used a global variable at the workflow level called `APP_NAME` with the value `myapp`. At the job level, I defined an environment variable `ENVIRONMENT` with the value `staging`. Finally, at the step level, I set a variable `VERSION` with the value `1.0.0`. I printed all three variables in a single step to verify that they were accessible. Additionally, I used GitHub context variables to print the commit SHA and the actor who triggered the run.

### Task 3: Job Outputs
i used the `outputs:` feature to pass data between jobs. In the first job, I set an output variable to today's date as a string. In the second job, I read that output and printed it. Passing outputs between jobs is useful for sharing data that is generated in one job and needed in another, allowing for better modularity and separation of concerns in workflows. i would use this feature when I need to pass results, such as build artifacts or computed values, from one job to another.

### Task 4: Conditionals
I used condtional statements in my workflow to control the execution of steps and jobs. I added a step that only runs when the branch is `main`, and another step that only runs if the previous step failed. I also created a job that only runs on push events, not on pull requests. Additionally, I included a step with `continue-on-error: true`, which allows the workflow to continue executing subsequent steps even if that particular step fails.
final worflow error including all the above tasks :

```name: multi-job
on:
    push:
        branches:
            - main
    workflow_dispatch:

env:
    APP_NAME: myapp
jobs:
    build:
        runs-on: ubuntu-latest
        env: 
            ENVIRONMENT: staging
            VERSION: 1.0.0
        steps:
            - name: printing
              run: echo "building the app"

            - name: printing details
              run: |
                echo "Application: $APP_NAME"
                echo "Environment: $ENVIRONMENT"
                echo "Version: $VERSION"
                echo "Commit SHA: $GITHUB_SHA"
                echo "Actor: $GITHUB_ACTOR"

            - name: only runs on main
              if: github.ref == 'refs/heads/main'
              run: echo 'this step only runs on main'
              
    continue-error:
        runs-on: ubuntu-latest
        steps:
            
            - name: allow failure
              continue-on-error: true
              run: |
                echo "this will fail"
                exit 1

            - name: this still runs
              run: echo "the workflow continues"
    test:
        runs-on: ubuntu-latest
        needs: build
        steps:
            - name: printing test
              run: echo "running tests"

    deploy:
        runs-on: ubuntu-latest
        needs: test
        steps:
            - name: printing deploy
              run: echo "deploying"

    set-output:
        runs-on: ubuntu-latest
        outputs:
            today: ${{ steps.date.outputs.today }}
        steps:
            - name: get today's date
              id: date
              run: echo "today=$(date +'%Y-%m-%d')" >> "$GITHUB_OUTPUT"

    display:
        runs-on: ubuntu-latest
        needs: set-output
        steps:
            - name: print date from previous job
              run: |
                echo "Today's date is: ${{ needs.generate.outputs.today }}"

    push-only:
        if: github.event_name == 'push'
        runs-on: ubuntu-latest
        steps:
            - name: push only job
              run: echo "this job runs only after a push"
```

### Task 5: Putting It Together
smart pipeline.yml file that triggers on push to any branch, has a `lint` job and a `test` job running in parallel, and a `summary` job that runs after both, printing whether it's a `main` branch push or a feature branch push, and the commit message:

```name: smart-pipeline
on:
    push:
        branches: 
            - '**'
jobs:
    lint:
      runs-on: ubuntu-latest
      steps:
        - name: running linting
          run: echo "running linting"

    test:
        runs-on: ubuntu-latest
        steps:
            - name: running tests
              run: echo "running tests"

    summary:
        runs-on: ubuntu-latest
        needs: [lint, test]
        steps:
            - name: pipeline summary
              run: |
                if [ "$GITHUB_REF" = "refs/heads/main" ]; then
                    echo " this is a main branch"
                else
                    echo "this is a feature branch"
                fi

                echo "commit message"
                echo "${{github.event.head_commit.message}}"
```