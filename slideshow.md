---
marp: true
author: Talmeez Faizy
size: 4:3
theme: uncover

---
<style>
    {
         font-size:18px
     }
</style>

# Github Actions
- A tool that lets you automate your software development workflows.
- You can write individual task, called actions and combine them to create a custom workflow.


### What are workflows?

Custom automated process that you can setup in your repository to build, test, package, release or deploy any code project on GitHub
Workflow can have more than one job and each of them will run on their virtual machine. It can be parallel or in series.

For e.g.- we have to run an app
So one job can run on windows virtual machine building an android version of an app and another job can run on macOS virtual machine building an iOS version of the same app.

---
### What is a runner?
- Any machine with the GitHub actions runner application installed
- A runner is responsible for running your jobs whenever an event happens and displays back the results
- It can be hosted by GitHub or you can host your own runner

### Self hosted runner:
- A machine you manage and maintain with the runner application installed
- Ideal if you need to control hardware

### GitHub hosted runner:
- You cannot customize the hardware to a great extent, can only choose from the options available through GitHub

**For private repo GitHub-actions are not free!**

---
### Yaml introduction:

- It is similar to JSON with slight different format, i.e. key value pair language
- We can use Yaml extension by red hat in VSCode for ease 
- We have to use indentation using spaces and not tab. (Either two space or four)
- Array in Yaml:
```sh
array: 
  - Item1
  - Item2
 
For array of objects-
objectsArray:
  - key: value
    key2: value2
  - key1: value1
```
---
- Longtext in yaml:

```sh
longtext: |
  Hi, this is line 1
  Hi, this is line 2
  Hi, this is line three
  
In practical scenario it is used to run commands:

run: |
  node -v
  cat /etc/os-release
```

---
### Simple workflow:

```
#Name of entire workflow
name: Shell Commands

#After name we define events which will trigger the workflow

on: [push] # it can be array of events

#Git hosted runner
jobs: 
    run-shell-command:
      runs-on: ubuntu-latest
      #put example of self hosted for explaining the case of VM runner
      steps:
        - name: echo a string
          run: echo "Testing an echo command"
        - name: multiline script
          run: |
            node -v
            npm -v
```
---
### Playing with workflows

#### Using published actions, output of steps and checkout action

- Actions are some code that does some specific tasks which can be used in a workflow
- You can either have actions defined in an other folder or repository which you can use in your workflow
- Actions can be published in github and those actions can also be used

```
jobs:
  run-github-actions:
    runs-on: ubuntu-latest
    steps:
      - name: simple JS Action
        uses: actions/hello-world-javascript-action@v1
        with:
          who-to-greet: Faizy

# different ways to use actions:
# actions/hello-world-javascript-action@master
# actions/hello-world-javascript-action@v1
# actions/hello-world-javascript-action@<commit-id>  
```
---
- You can use output of one step in other step
```
jobs:
  run-github-actions:
    runs-on: ubuntu-latest
    steps:
      - name: simple JS Action
        # define id which will be used in another step
        id: greet
        uses: actions/hello-world-javascript-action@v1
        with:
          who-to-greet: Faizy
      # getting output of above action
      - name: Log Greeting Time
        run: echo "${{ steps.greet.outputs.time }}"
```
- Checkout action
  - By default git does not clone your repo in the working directory. All workflows may not require the files of repository
  - The checkout action can also take some inputs as well. You can for instance change the commit that you are checking-out to, override the authentication token with another one, choose the fetch depth and more. [Checout-Action](https://github.com/actions/checkout#usage)
  
---
  Combined action of above explanation:

```
name: Actions workflow

on: [push]

jobs:
  run-github-actions:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout branch
        uses: actions/checkout@v1
      - name: simple JS Action
        id: greet
        uses: actions/hello-world-javascript-action@v1
        with:
          who-to-greet: Faizy
      - name: Log Greeting Time
        run: echo "${{ steps.greet.outputs.time }}"
```

---
#### Events, Schedules and Filters

- Trigerring Workflow:
  - [Triggering-Workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow)
  - Pull request can have many activity types, which can be specified. 
  - Instead of passing as an array, we can also pass triggers as an object
  - We can define branches for the trigger event
  - We can also filter branches using the regex [Filter-Pattern-Cheatsheet](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#filter-pattern-cheat-sheet)
  - We can also schedule triggering of our workflows. 
    - cron: "* * * * *" #any day of the week, month, hour, every minute
    - It consists of 5 things: minutes hours dayofmonth month dayofweek
    - asterisk means any value
    - eg: 
       Cron: "0 12 * * *" at 12:00 every day
       Cron: "0 12 1 1 *" at minute 0 of 12 on day of month 1 in January
    - You can take help here to frame cron expression : [Crontab](https://crontab.guru/)

---
```
name: Trigger workflow

on:
#  schedule:
#   - cron: "0/5 * * * *"
 push:
  branches:
   - master
   - 'feature/*' # matches feature/featA, feature/featB, doesn't match feature/feat/a
   - 'feature/**' # matches feature/feat/c
   - '!feature/featC' # workflow does not run for feature/featC
  # tags:
  #  - v1.*
  # paths:
  #  - '**.js' # runs only javascript files
  # path-ignore:
  #  - '**.js'
 pull_request:
  types: [closed, assigned, reopened, opened]

jobs:
  run-github-actions:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout branch
        uses: actions/checkout@v1
      - name: simple JS Action
        id: greet
        # it is used to refer to action defined in another folder or using published actions
        uses: actions/hello-world-javascript-action@v1
        with:
          who-to-greet: Faizy
```

---

#### Environment Variables, Ecnryption and context

**Environment Variables and Encryption**

- Environment variables can be defined for complete workflow, or a specific job, or for a specific step in a specific job
- Environment variable can be encrypted using github secrets
- GITHUB_TOKEN need not be defined. Secrets inherently has it. ${{ secrets.GITHUB_TOKEN }}

---
```
name: Environment workflow
on:
  push:
env: 
  WF_ENV: Workflow env variable
jobs:
  log-env:
    runs-on: ubuntu-latest
    env:
      JOB_ENV: job env variable
    steps:
      - name: echo variables
        env: 
          STEP_ENV: step env variable
        run: |
          echo "WF_ENV: ${WF_ENV}"
          echo "JOB_ENV: ${JOB_ENV}"
          echo "STEP_ENV: ${STEP_ENV}"
  default-env:
    runs-on: ubuntu-latest
    steps:##
      - name: default env variable values
        run: |
          echo "HOME: ${HOME}"
          echo "GITHUB_WORKFLOW: ${GITHUB_WORKFLOW}"
          echo "GITHUB_ACTIONS: ${GITHUB_ACTIONS}"
          echo "GITHUB_ACTION: ${GITHUB_ACTION}"
          echo "GITHUB_ACTOR: ${GITHUB_ACTOR}"
          echo "GITHUB_REPOSITORY: ${GITHUB_REPOSITORY}"
          echo "GITHUB_EVENT_NAME: ${GITHUB_EVENT_NAME}"
          echo "GITHUB_WORKSPACE: ${GITHUB_WORKSPACE}"
          echo "GITHUB_SHA: ${GITHUB_SHA}"
          echo "GITHUB_REF: ${GITHUB_REF}"

#$GITHUB_SHA - commit id of the specific commit that has triggered the workflow
#$GITHUB_REPOSITORY - the username and repo name
#$GITHUB_WORKSPACE - workspace directory
#https://docs.github.com/en/actions/learn-github-actions/environment-variables#default-environment-variables
```

---

**Context**

[Github_Context](https://docs.github.com/en/actions/learn-github-actions/contexts)
- Contexts are a way to access information about workflow runs, runner environments, jobs, and steps. Each context is an object that contains properties, which can be strings or other objects.
- Contexts, objects, and properties will vary significantly under different workflow run conditions. For example, the matrix context is only populated for jobs in a matrix.
- You can access contexts using the expression syntax. ${{ <context> }}
- There are numerous context like github context, env context, steps context, etc
- As part of an expression, you can access context information using one of two syntaxes.
  - Index syntax: github['sha']
  - Property dereference syntax: github.sha

---

- You can use most contexts at any point in your workflow, including when default environment variables would be unavailable. For example, you can use contexts with expressions to perform initial processing before the job is routed to a runner for execution; this allows you to use a context with the conditional if keyword to determine whether a step should run. Once the job is running, you can also retrieve context variables from the runner that is executing the job, such as runner.os. 
- Context use example:

```
name: CI
on: push
jobs:
  prod-check:
    if: ${{ github.ref == 'refs/heads/main' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production server on branch $GITHUB_REF"
```

---
#### Strategy, matrix and Docker

**ContinueOnError and Timeout**

```
jobs: 
    run-shell-command:
      runs-on: ubuntu-latest 
      timeout-minutes: 360 # by default is 360 minutes you can increase it or decrease it

      continue-on-error: true # this make sure all steps will run even if this job fails 
      # Timeout minutes can be added in steps or in whole job
```

---
**Strategy**

Strategy helps to setup environment matrix.
[Setup-Node-Action-Details](https://github.com/actions/setup-node)

```
name: matirx workflow
on: [push]
jobs:
  node-version:
    runs-on: ubuntu-latest
    steps: 
      - name: Log node version
        run: node -v 
      - uses: actions/setup-node@v1
        with:
          node-version: 6
      - name: Log node new version 
        run: node -v 

```
Above workflow will setup node to 6 version but if we want to have different versions of node, then **strategy** helps.


---

**How to use strategy to create matrix of environment?**

```
name: matirx workflow
on: [push]
# job will run for number of times depending on values present in matrix
# if we define one more parameter with 3 values, then job will run for 
# every node version and every parameter, which means total 9 times it will run
jobs:
  node-version:
    # using strategy
    strategy:
      matrix:
       node_version: [6,8,10]
      # max-parallel: 2 # used to limit max jobs you want to run the matrix in paralell
      # fail-fast: true # write what is the use of it?
    runs-on: ubuntu-latest
    steps: 
      - name: Log node version
        run: node -v 
      - uses: actions/setup-node@v1
        with:
          # node-version: 6
          node-version: ${{ matrix.node_version }}
      - name: Log node new version 
        run: node -v 
```

---

**Include and Exclude in matrix**

```
name: matirx workflow
on: [push]
# job will run for number of times depending on values present in matrix
# if we define one more parameter with 3 values, then job will run for 
# every node version and every parameter, which means total 9 times it will run
jobs:
  node-version:
    # using strategy
    strategy:
      matrix:
       os: [macos-latest, ubuntu-latest, windows-latest]
       node_version: [6,8,10]
       # is_ubuntu_8 will only exist for the below config otherwise it will be an empty string
       # It includes extra variable for certain configuration in the matrix
       include:
         - os: ubuntu-latest
           node-version: 8
           is_ubuntu_8: "true"
       # job will not run for the config defined in exclude
       exclude:
         - os: ubuntu-latest
           node_version: 6
         - os: macos-latest
           node_version: 8
      # max-parallel: 2 # used to limit max jobs you want to run the matrix in paralelle
      # fail-fast: true
    runs-on: ubuntu-latest
    env:
      IS_UBUNTU_8: ${{ matrix.is_ubuntu_8 }}
    steps: 
      - name: Log node version
        run: node -v 
      - uses: actions/setup-node@v1
        with:
          # node-version: 6
          node-version: ${{ matrix.node_version }}
      - name: Log node new version 
        run: |
          node -v 
          echo $IS_UBUNTU_8

```

---

**Docker Container in Jobs**

```
name: Container
on: push
jobs:
  node-docker:
    runs-on: ubuntu-latest
    #This can be image from docker hub
    # how to put: dockerhub username and then image
    container: #node:13.5.0-alpine3.10 
    #this may show error in vs code because it expects to be an object not string
    #other way to pass as an object
      image: node:13.5.0-alpine3.10
      #env: pass env variables
      #ports: ports to be exposed in a container
      #volumes: if there is any volume mapping
      #options: --cpus 1 --host etct
    steps:
      - name: Log node version
        run: |
          node -v 
          cat /etc/os-release
```

---

**Multiple Docker Container in Jobs**

In normal scenario it is done by using docker compose.
But Github actions allows us to run multiple containers

```
name: Multiple Container
on: push

jobs: 
  multiple_containers:
    runs-on: ubuntu-latest
    services: 
      app:
        image: alialaa17/node-api
        ports: 
          - 3001:3000
      mongo:
        image: mongo
        ports: 
          - 27017:27017
    steps:
      - name: Post a user
        run: |
        "curl -X POST http://localhost:3001/api/user -H 'Content-Type: application/json' 
        -d '{\"username\":\"hello\",\"address\":\"dwed\"}'"
      - name: Get users
        run: curl http://localhost:3001/api/users 
```

---

**Running Containers in Steps**
In docker file we have entry point, which is the command that is executed as soon as container starts.
It is usually defined as: 
ENTRYPOINT echo "hello world"  // this is shell form of entrypoint
Other way of Entry Point:

//called executable form of entry point
ENTRYPOINT ['/bin/echo', 'Hello'] //path to executable and args to be passed to executable
CMD ['WORLD'] //it contains additional args which has to be passed to executables

---

The above method is similar to the steps followed in github actions:

```
jobs: 
  docker-steps:
    runs-on: ubuntu-latest
    container:
      image: node:10.18.0-jessie
    steps:
      - name: log node version
        run: node -v 
      - name: Step with docker
        uses: docker://node:12.14.1-alpine3.10
        with:
          # unlike docker file it will have just path to executable
          entrypoint: /bin/echo
          # unlike docker file it can only have single arg
          args: hello world
      - name: Log node version
        uses: docker://node:12.14.1-alpine3.10
        with:
          entrypoint: /usr/local/bin/node
          args: -v
```
