# Steps to Write Github Actions(GHA)

## Create `.yaml` or `.yml` File Inside `.github/workflows`

```bash
mkdir -p .github/workflows
touch .github/workflows/first-action.yaml
```
## Write Name and Triggering Events

```yaml
# name of workflow [optional]
name: GHA Example

# `on` determines when this Action shoud run 
on:
  workflow_dispatch: # means manually triggerd via GHA UI
```

## Define Jobs(Actions) to Run

```yaml
# everything from above plus following ...
jobs:
  say_hello: # this is unique ID for your job
    # this is name of Job-> it shows directly in the GitHub Actions (GHA) web interface.
    name: Say Hello in Bash
    # in which Github Hosted machine you want to run this Job  (depends on steps you run in this job -> see below), if app is *.exe, it would have been `windows-latest` instead
    runs_on: ubuntu-latest 
```

## Write Steps to Complete The Job

```yaml
# we already know our Job is to `Say Hello in Bash`, so write steps to complete it 
jobs:
  ...
  runs_on:
  ...
#   here is actual steps -> list of step/actions
  steps:
    - name: Saying Hello # name of step (shows up inside GHA)
      run: echo "Hello In Bash" # run means to run this command in default shell
    # since just one task or step complete our Job motive, no further steps are required, if your Job require more steps you would have done something like:
    # - name: Another Step
    #   run: some_command
```

# Github Actions Example

## 1. Simple Hello World Bash

Available at: [.github/workflows/01-hello-world-bash.yaml](.github/workflows/01-hello-world-bash.yaml)
```yaml
name: Simple Hello World in Bash

on:
  workflow_dispatch:

jobs:
  hello_world_bash:
    name: Say Hello World in Bash
    runs-on: ubuntu-latest

    steps:
      - name: Saying Hello World
        run: echo "Hello World From Bash!"
```
The code will be available in Github once you `git push` and since the workflow trigger is just `workflow_dispatch`, it runs manually when invoked from GHA UI. Click **Run Workflow** on your default branch below:

![Simple Hello World in Bash](/images/01-simple-hello-world-in-bash.png)

The Yellow color is indicating Workflow is running:

![alt text](/images/01-running.png)

The exact step named **Saying Hello World** is run. The other steps are Github's own bootstrap and cleanup:

![alt text](/images/01-success.png)

## 2. Different Step Types

[Previous Example](#1-simple-hello-world-bash) uses default shell to run the command.  But what you can do is choose different shell type or even choose github actions from market place. See example:
```yaml
name: Different Types of Tasks/Steps You Can Run

on:
  workflow_dispatch:

jobs:
  hello_world_bash:
    name: Say Hello World in Different Steps
    runs-on: ubuntu-latest

    steps:
    # hello world in default shell
      - name: Hello World in Bash
        run: echo "Hello World From Bash!"

      # run hello world function inside python interpreter
      - name: Hello World in Python Shell
        run: print("Hello World From Python Shell!")
        shell: python
      
      - name: Hello Wold Using Github Marketplace Action
        uses: sormuras/hello-world-java-action@34113a1c31b4deb2efc4810cd45ad16a90f45c3f
        with:
          who-to-greet: 'From Random Github Actions!' 
```

This example is available at: [.github/workflows/02-different-step-types.yaml](.github/workflows/02-different-step-types.yaml).

The third one may interest you. I browse Github marketplace for random java hello world action which is: `sormuras/hello-world-java-action` that I found. **Github Actions from marketplace is nothing but pre baked set of steps somebody else has written which you are just kind of importing in one of the steps of your workflow**.  You write this in `uses`key. The `with` block determines the input for this action. You should check readme or docs of each action to find what inputs it accepts.

You specify version for the action itself as `ACTION@version`. You can use `@v1` or any version that actions supports. For version pinning, it's recommended to use commit hash as such:
```
uses: sormuras/hello-world-java-action@34113a1c31b4deb2efc4810cd45ad16a90f45c3f
```

`34113a1c31b4deb2efc4810cd45ad16a90f45c3f` is nothing but commit hash you want to pin i.e. **you want exact this version because it is stable and works well for  your project**.

Just like in [first example](#1-simple-hello-world-bash), after `git push`, you trigger this manually because of `workflow_dispatch`. If you run it, you should see:

![alt text](/images/02-success.png)

## 3. Parallel vs Dependent Jobs

When jobs are independent, they run in parallel but dependent jobs can only run after jobs which the specific job depends on is completed. Few things to remember:

- Each step in a job runs only after prior step is success. If any step fails, workflow stops there. You can change default behavior but for now understanding this much is enough.
- Each job runs in different runner (machine) by default, this means artifacts (files) created in one job is not available in another job.
- If you need files for next job, you upload them in current job and download in the next job that requires it.
- If a parallel job fails, GitHub immediately cancels all other currently running parallel jobs by default to save run time. This behavior can also be changed.

```yaml
on:
  workflow_dispatch:

jobs:
  install_node:
    runs-on: ubuntu-latest
    steps:
      - name: Install Node.js
        run: sudo apt-get update && sudo apt-get install nodejs -y

  create_hello_world_js:
    runs-on: ubuntu-latest
    steps:
      - name: Prepare Hello World Program In JavaScript
        run: echo 'console.log("Hello, World!");' > hello.js

      - name: Upload hello.js as an artifact
        uses: actions/upload-artifact@v4
        with:
          name: javascript-code
          path: hello.js

  run_hello_world:
    runs-on: ubuntu-latest
    needs: 
    - install_node
    - create_hello_world_js

    steps:
      - name: Download hello.js artifact
        uses: actions/download-artifact@v4
        with:
          name: javascript-code

      - name: Execute JavaScript File
        run: node hello.js
```

You can see this dependency graph:
- `install_node` and `create_hello_world_js` are independent of eachother, they are running parallelly but `run_hello_world` is waiting for jobs to complete:

    ![alt text](/images/03-dependent-jobs.png)

- After success:

    ![alt text](/images/03-success-jobs.png)

## 4. Different Triggers

```yaml
name: Common Workflow Triggers

on:
  # Run for pushes to master or for version tags
  push:
    branches:
      - master
    tags:
      - "v*"

  # Run when a pull request targets master
  pull_request:
    branches:
      - master

  # Allow a manual run from the Actions tab
  workflow_dispatch:

  # Run once every day at 09:00 UTC
#   schedule:
#     - cron: "0 9 * * *"

jobs:
  show-trigger:
    runs-on: ubuntu-24.04

    steps:
      - name: Show the event that started this workflow
        run: |
          echo "Workflow triggered by: ${{ github.event_name }}"
```

The key idea for learners is:

- `push` → code or tags are pushed (`git push origin master`)
- `pull_request` → a PR activity occurs
- `workflow_dispatch` → someone starts it manually
- `schedule` → runs on a cron schedule
- `push.branches` → limit pushes to particular branches (only on `master` in this example)
- `push.tags` → respond specifically to tags such as `git push origin v1.0.0`

For example when I code push using:
```bash
git push origin master
```

The output shows `Workflow triggered by: push`:

![alt text](/images/04-success-on-push.png)

And when manually triggered, it's `workflow_dispatch`:

![alt text](/images/04-success-on-workflow-dispatch.png)

## 5. Workflow Filters

```yaml
name: Workflow Filters

on:
  push:
    # Only run for these branches
    branches:
      - master
      - develop

    # Only run when files under these paths change
    paths:
      - "images/**"
      - "README.md"

    # Run for version tags such as v1.0.0
    tags:
      - "v*"

  pull_request:
    # Only run when the PR targets master
    branches:
      - master

    # Ignore documentation-only changes
    paths-ignore:
      - "docs/**"
      - "*.md"

jobs:
  test:
    runs-on: ubuntu-24.04

    steps:
      - name: Run tests
        run: echo "Running tests..."
```

In this example:
```bash
git add .gitignore
git commit -m 'Add gitignore'
git push origin master
```
Won't trigger the workflow because of:
```yaml
...
    # Only run when files under these paths change
    paths:
      - "images/**"
      - "README.md"
```

However when `README.md` is pushed:
```bash
git add README.md
git commit -m 'Update README'
git push origin master
```
The workflow is triggered:

The main filters to know:

- `branches` → run only for specific branches.
- `branches-ignore` → exclude specific branches.
- `tags` → run only for matching tags.
- `tags-ignore` → exclude matching tags.
- `paths` → run only when matching files/directories change.
- `paths-ignore` → skip the workflow when only matching files change.

> [!NOTE] 
> **filters are event-specific**. The available filtering options depend on the trigger you're configuring, so `push`, `pull_request`, and other events don't all accept exactly the same filters.

## 6. Environment Variable Scope

```yaml
name: Demonstrate Environment Variable Scope

on:
  workflow_dispatch:

env:
  GLOBAL_MESSAGE: available_to_all_jobs

jobs:
  first-job:
    runs-on: ubuntu-24.04

    env:
      JOB_MESSAGE: available_inside_first_job

    steps:
      - name: Check variables from step one
        env:
          LOCAL_MESSAGE: available_only_in_this_step
        run: |
          echo "Global: $GLOBAL_MESSAGE"
          echo "Job:    $JOB_MESSAGE"
          echo "Step:   $LOCAL_MESSAGE"

      - name: Check variables from step two
        run: |
          echo "Global: $GLOBAL_MESSAGE"
          echo "Job:    $JOB_MESSAGE"
          echo "Step:   ${LOCAL_MESSAGE:-not-defined}"

  second-job:
    runs-on: ubuntu-24.04

    steps:
      - name: Check variables in another job
        run: |
          echo "Global: $GLOBAL_MESSAGE"
          echo "Job:    ${JOB_MESSAGE:-not-defined}"
          echo "Step:   ${LOCAL_MESSAGE:-not-defined}"
```

This demonstrates three levels of scope:

- **Workflow-level:** `GLOBAL_MESSAGE` is inherited by both jobs.
- **Job-level:** `JOB_MESSAGE` exists only within `first-job`.
- **Step-level:** `LOCAL_MESSAGE` exists only during the step where it is declared.

When run manually, the output for the `first-job` is:

![alt text](/images/06-first-job.png)

And the output for the `second-job` is:

![alt text](/images/06-second-job.png)

## 7. Environment, Secrets and Variables

```yaml
name: Use Secrets and Environment Variables

on:
  workflow_dispatch:

jobs:
  deploy-staging:
    runs-on: ubuntu-24.04
    environment: staging

    env:
      # Values configured at the repository level
      REPO_SECRET: ${{ secrets.DEPLOY_SECRET }}
      REPO_SETTING: ${{ vars.DEPLOY_REGION }}

      # Values configured specifically for the staging environment
      STAGING_SECRET: ${{ secrets.STAGING_TOKEN }}
      STAGING_SETTING: ${{ vars.STAGING_URL }}

    steps:
      - name: Display configured values
        run: |
          echo "Repository secret:     $REPO_SECRET"
          echo "Repository variable:   $REPO_SETTING"
          echo "Staging secret:        $STAGING_SECRET"
          echo "Staging variable:      $STAGING_SETTING"

  deploy-production:
    runs-on: ubuntu-24.04
    environment: production

    env:
      # Repository-level values are available here too
      REPO_SECRET: ${{ secrets.DEPLOY_SECRET }}
      REPO_SETTING: ${{ vars.DEPLOY_REGION }}

      # These values come from the production environment
      PRODUCTION_SECRET: ${{ secrets.PRODUCTION_TOKEN }}
      PRODUCTION_SETTING: ${{ vars.PRODUCTION_URL }}

    steps:
      - name: Display configured values
        run: |
          echo "Repository secret:     $REPO_SECRET"
          echo "Repository variable:   $REPO_SETTING"
          echo "Production secret:     $PRODUCTION_SECRET"
          echo "Production variable:   $PRODUCTION_SETTING"
```

What this example teaches:

- **Repository secrets/variables** are configured at the repository level and can be used by jobs that have access to them.
- **Environment secrets/variables** belong to a specific environment such as `staging` or `production`.
- `environment: staging` tells GitHub that the job is associated with the **staging environment**.
- `environment: production` does the same for **production**.
- Secrets are automatically **masked in workflow logs**, so printing a secret isn't a good way to inspect its actual value.
- `${{ secrets.NAME }}` accesses a secret, while `${{ vars.NAME }}` accesses a non-secret configuration variable.

 A nice conceptual distinction for learners is:

```
Repository
├── Secrets
│   └── DEPLOY_SECRET
└── Variables
    └── DEPLOY_REGION

Environments
├── staging
│   ├── Secrets
│   │   └── STAGING_TOKEN
│   └── Variables
│       └── STAGING_URL
│
└── production
    ├── Secrets
    │   └── PRODUCTION_TOKEN
    └── Variables
        └── PRODUCTION_URL
```

This makes it easier to see that **repository-level configuration is shared**, while **environment-level configuration can differ between staging and production**.

### Before running the workflow

In your GitHub repository, go to:

**Settings → Secrets and variables → Actions**

Create these **repository-level** values:

- Repository secret: `DEPLOY_SECRET`
- Repository variable: `DEPLOY_REGION`

Then configure the environments:

**Settings → Environments → New environment**

Create:

- `staging`
- `production`

Inside each environment, add its own values.

For `staging`:

- Environment secret: `STAGING_TOKEN`
- Environment variable: `STAGING_URL`

For `production`:

- Environment secret: `PRODUCTION_TOKEN`
- Environment variable: `PRODUCTION_URL`

For example:

```
Repository
│
├── Secrets
│   └── DEPLOY_SECRET
│
├── Variables
│   └── DEPLOY_REGION
│
└── Environments
    │
    ├── staging
    │   ├── Secrets
    │   │   └── STAGING_TOKEN
    │   └── Variables
    │       └── STAGING_URL
    │
    └── production
        ├── Secrets
        │   └── PRODUCTION_TOKEN
        └── Variables
            └── PRODUCTION_URL
```

Then the workflow can access them using:

```
env:
  REPO_SECRET: ${{ secrets.DEPLOY_SECRET }}
  REPO_SETTING: ${{ vars.DEPLOY_REGION }}
  ENV_SECRET: ${{ secrets.STAGING_TOKEN }}
  ENV_SETTING: ${{ vars.STAGING_URL }}
```

The important relationship is:

**GitHub Settings → create configuration → workflow references it with `${{ secrets.* }}` or `${{ vars.* }}`.**

Also, the `environment: staging` / `environment: production` setting is important: it tells GitHub **which environment's configuration and protection rules apply to that job**.

When you run the workflow manually, you will see all the secrets are masked (`***`) and variables can be `echo`ed:

Staging:

![alt text](/images/07-staging.png)

Production:

![alt text](/images/07-prod.png)

## 8. Runners (Machines Where GHA Run)

You have different option for choosing runners which includes all major OS:
### Github Hosted Runners
- [Github Hosted Runners For Public Repos](https://docs.github.com/en/actions/reference/runners/github-hosted-runners#standard-github-hosted-runners-for-public-repositories)
- [Github Hosted Runners For Private Repos](https://docs.github.com/en/actions/reference/runners/github-hosted-runners#standard-github-hosted-runners-for-prvivate-repositories)

Only Free till limits set by Github. See [free usage limits](https://docs.github.com/en/billing/concepts/product-billing/github-actions#free-use-of-github-actions) for more info.

### Self Hosted Runners

You can use your own runner hosted on K8s, VPC, etc. See more on [self hosted runners](https://docs.github.com/en/actions/concepts/runners/self-hosted-runners)

Some commonly used options include for this:
- **RunsOn** — provisions GitHub Actions runners as temporary AWS EC2 instances.
- **Actions Runner Controller (ARC)** — runs GitHub Actions runners inside Kubernetes clusters.
- **Railway GitHub Actions Runners** — allows you to run self-hosted GitHub Actions runners on Railway.

Example using Github Hosted runners:
```yaml
name: Runner Types

on:
  workflow_dispatch:

jobs:
  ubuntu:
    name: Runs on Ubuntu Latest
    runs-on: ubuntu-latest
    steps:
      - name: Print Runner Information
        run: |
          echo "Architecture: ${{ runner.os }}-${{ runner.arch }}"
          echo "Distro Type: $(grep ID_LIKE /etc/os-release)"
          echo "Runner Name: ${{ runner.name }}"

  windows:
    name: Runs on Windows Latest
    runs-on: windows-latest
    steps:
      - name: Print Runner Information
        run: |
          echo "Architecture: ${{ runner.os }}-${{ runner.arch }}"
          echo "Runner Name: ${{ runner.name }}"

  mac:
    name: Runs on macOS Latest
    runs-on: macos-latest
    steps:
      - name: Print Runner Information
        run: |
          echo "Architecture: ${{ runner.os }}-${{ runner.arch }}"
          echo "Runner Name: ${{ runner.name }}"
```

Example output on manual run:

![alt text](/images/08-success.png)

## 9. Workflow Summary

```yaml
name: Workflow Summary

on:
  workflow_dispatch:

jobs:
  say_hi_and_say_hello:
    runs-on: ubuntu-latest
    steps:
      - name: Say Hi
        run: echo "Saying Hi!"

      - name: Say Hello
        run: echo "Saying Hello!"

      - name: Job Summary
        run: |
          echo "### Summary ###" >> "$GITHUB_STEP_SUMMARY"
          echo "Said Hi and Hello" >> "$GITHUB_STEP_SUMMARY"
```

Summary is shown on the same **Actions** Tab below your workflow:

![alt text](/images/09-workflow-summary.png)

## 10. Expressions

Expressions are written inside `{{ expression }}` and evaluates to `true` or `false`:
```yaml
name: Expressions Example

on:
  workflow_dispatch:


jobs:
  expressions:
    runs-on: ubuntu-latest
    steps:
      - name: is equals to
        run: echo "5==5->${{ 5==5 }}"

      - name: is not equals to
        run: echo "5!=5->${{ 5!=5 }}"

      - name: is less than
        run: echo "4<=3->${{ 4==3 }}"

      - name: github.ref_name is equals to 'main'
        run: echo "Is ref_name 'main':${{ 'main' == github.ref_name }}"

      - name: Combine Operation1(&&)
        run: echo "True/False(&&)->${{ 1<2 && 3<3 }}"

      - name: Combine Operation2(||)
        run: echo "True/False(||)->${{ 1<2 || 2<3 }}"
```
The result:

![alt text](/images/10-expressions-output.png)

## 11. Job Level If Conditions

```yaml
name: Job Level If Conditions

on:
  workflow_dispatch:

jobs:
  needs_linux:
    runs-on: ubuntu-latest
    if: vars.required_os != 'Windows'
    steps:
      - name: Print Requied OS
        run: echo "Runner OS => ${{ vars.required_os }}"

  only_on_master:
    if: github.ref_name == 'master'
    runs-on: windows-latest
    steps:
      - name: Print Branch Name
        run: echo "Branch Name => ${{ github.ref_name }}"
```

## 12. Step Level If Conditions

```yaml
name: If Conditional Step Levl
on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Always runs"

      - if: github.ref_name == 'main'
        run: echo "Only runs on main"

      - if: github.ref_name != 'main'
        run: echo "Only runs on non-main branches"
```