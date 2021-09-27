# Introduction

## What is dstack?

Typical ML workflows consist of multiple steps, e.g. pre-processing data, training, fine-tuning, validation, etc.
The `dstack` tool lets you define these steps in a simple YAML format, and then, run any of them on any infrastructure 
of your choice via the CLI. 

For example. if you have own servers where you'd like to run ML workflows, you may install the `dstack-runner` 
daemon on these servers, and, the next time you run a workflow via the `dstack` CLI, these workflows will 
running on these servers. `dstack` daemon will take care of managing workflow dependencies, 
upstreaming logs and uploading output artifacts to the storage. No manual use of SSH, Git, and Docker is needed. 
Just use the `dstack` CLI to run and monitor workflows over a pool of configured servers.

Alternatively, if you don't want to use own servers, you can provide `dstack` credentials to your cloud,
and `dstack` will automatically set up instances at spot price to run submitted workflows.

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

## How does works

1. When you run a workflow using the `dstack run` command, `dstack` sends the run requests to the `dstack` server. 
The request contains the information Git repo (incl. the branch, current revision, uncommitted changes, etc.), 
the name of the workflow, and the overridden variable values.
2. Once the `dstack` server receives the request, it creates jobs associated with the run request (one per workflow). 
Then, it assigns each job to one of the available runners. If a workflow with the corresponding code and variables
had run before, it will be reused.
3. Each new job is assigned by the `dstack` server to available runners.
4. If the user provided `dstack` credentials to own cloud account and provided the corresponding rules, 
the `dstack` server will automatically set up runners at spot price based on the demand.   
5. Once a runner receives a job, it runs, upstream logs, and in the end upload output artifacts.