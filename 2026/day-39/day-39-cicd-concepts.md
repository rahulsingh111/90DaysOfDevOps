### Task 1: The Problem
the problem with a team of 5 developers manually deploying code to production is that it can lead to inconsistencies, errors, and downtime. Each developer may have different environments, dependencies, and configurations, which can cause the application to behave differently on each machine. The phrase "it works on my machine" highlights this issue, as it means that the code runs successfully in one developer's environment but may fail in others due to these differences.the team can safely deploy manually a limited number of times a day, depending on the complexity of the application and the risk of introducing bugs. However, frequent manual deployments increase the likelihood of errors and make it difficult to maintain a stable production environment.

### Task 2: CI vs CD
continous integration (CI) is the practice of automatically integrating code changes from multiple developers into a shared repository several times a day. It involves running automated tests to catch bugs early in the development process. A real-world example of CI is using Jenkins to automatically build and test code whenever a developer pushes changes to a GitHub repository.

Continuous delivery (CD) is the practice of automatically preparing code changes for release to production. It ensures that the code is always in a deployable state, allowing teams to release updates quickly and reliably. A real-world example of CD is using GitLab CI/CD to automatically build, test, and package an application for deployment to a staging environment.

Continuous deployment is an extension of continuous delivery where code changes are automatically deployed to production without manual intervention. It allows teams to release new features and bug fixes rapidly, but requires a high level of confidence in the automated testing and deployment processes. A real-world example of continuous deployment is using CircleCI to automatically deploy a web application to AWS whenever changes are pushed to the main branch.

### Task 3: Pipeline Anatomy
Trigger — the event that initiates the pipeline, such as a code push or a pull request.
Stage — a logical phase in the pipeline that groups related jobs, such as build, test, or deploy.
Job — a unit of work within a stage that performs a specific task, such as running tests or building an artifact.
Step — a single command or action within a job, such as executing a script or running a command-line tool.
Runner — the machine or environment that executes the jobs in the pipeline, which can be a physical server, a virtual machine, or a container.
Artifact — the output produced by a job, such as compiled code, test results, or deployment packages, which can be used in subsequent stages of the pipeline.

### Task 5: Explore in the Wild
I checked this repo https://github.com/harry0703/MoneyPrinterTurbo/blob/main/.github/workflows/docker-ghcr.yml 
the triggers here are on push events to the main branch and manual workflow dispatch. 
it has one single job, publish, which builds a Docker image and pushes it to the GitHub Container Registry (GHCR). The job includes steps to check out the code, log in to GHCR, build the Docker image, and push it to the registry.
