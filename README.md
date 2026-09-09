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

The third one may interest you. I browse github market for random java hello world which is: `sormuras/hello-world-java-action` that I found. You write this in `uses`key. The `with` block determines the input for this action. You should check readme or docs of each action to find what inputs it accepts.

You specify version for the action itself as `ACTION@version`. You can use `@v1` or any version that actions supports. For version pinning, it's recommended to use commit hash as such:
```
uses: sormuras/hello-world-java-action@34113a1c31b4deb2efc4810cd45ad16a90f45c3f
```

`34113a1c31b4deb2efc4810cd45ad16a90f45c3f` is nothing but commit hash you want to pin i.e. **you want exact this version because it is stable and works well for  your project**.

Just like in [first example](#1-simple-hello-world-bash), after `git push`, you trigger this manually because of `workflow_dispatch`: