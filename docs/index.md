# Introduction

## Use case

Typical ML workflows consist of multiple steps, e.g. pre-processing data, training, fine-tuning, validation, etc.
`dstack` les you to define these steps in a simple YAML format, and then, run any of them on any infrastructure of your 
choice via the CLI.

If you use one or several machines to run ML workflows, you can install the `dstack-runner` 
daemon on all these machines, and submit workflows to run there from your laptop through the `dstack` CLI. 
The `dstack-runner` daemon will manage dependencies, run ML workflows through Docker, and upstream logs and output 
artifacts to the storage.

## Key components 

1. `.dstack/workflows.yaml` – you create this file in your project files, and define there all your workflows that you
   have in your project. For every workflow, you may specify the Docker image, the commands to run the workflow, the
   dependencies to other workflows, output artifacts, etc.
2. `.dstack/variables.yaml` – you create this file in your project files, and define there all the variables (and their
   default values) your workflow may depend on. These variables can be referenced from `.dstack/workflows.yaml`.
3. `dstack-runner` – you install and start this daemon on the machines that you'd like to use to run your workflows; you
   can use any number of machines (to make a pool of available runners); once the daemon is started, it runs assigned
   workflows, stream logs, and upload output artifacts to the storage.
4. `dstack` – you install this CLI (Command Line Interface) via `pip` and use from the terminal (e.g. from your laptop)
   to run the workflows defined in `.dstack/workflows.yaml`. When you run workflows with the CLI, you can override any
   variables.

## How it works

1. When you run a workflow using the `dstack run` command, `dstack` sends the run requests to the `dstack` server
(hosted with `dstack.ai`). The request contains the information Git repo (incl. the branch, current revision,
uncommitted changes, etc.), the name of the workflow, and the overridden variable values.
2. Once the `dstack` server receives a run request, it creates a list of jobs associated with the run request. Then,
it assigns each job to one of the available runners. Since the workflow that you run may depend on other workflows, the
`dstack` servers creates jobs for these other workflows too (in case they are not cached).
3. Each job is assigned by the `dstack` server to available runners.
4. Once a runner receives a job, it runs, stream logs, and in the end upload output artifacts.