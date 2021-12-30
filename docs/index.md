# How dstack works

Typical ML workflows include multiple steps, e.g. pre-processing data, training, fine-tuning, validation, etc. 
With `dstack`, you can define ML workflows in a simple YAML format, and run them via the CLI over a pool of
your own servers or spot instances in your cloud. 

### Workflows

Workflows are defined in the `.dstack/workflows.yaml` file inside a project's Git repository. Each workflow may have 
a name, a Docker image, commands, what files (in the repository) and what other workflows it depends on, and
at what paths the files must be stored as output artifacts. 
[Learn more&hellip;](define-wokflows.md)

### Variables

Each workflow may have variables and their default values. Variables for each workflow are defined in the 
`.dstack/varialbles.yaml` file inside a project's Git repository. When a workflow is run, any of the variables 
can be overridden. [Learn more&hellip;](define-wokflows.md#variables)

### CLI

CLI is an abbreviation of Command Line Interface. `dstack`'s CLI can be installed via `pip`. 
It can be used from a terminal to invoke any command on `dstack`, be it running workflows, browsing logs, 
managing runners, etc. [Learn more&hellip;](run-workflows.md)

### Runs

Runs are single instances of running workflows. If you submit a workflow to run (e.g. via `dstack` CLI), the 
corresponding run refers to the name of the workflow, the state of the Git repository (remote URL, branch name,
commit hash, and local changes), and to the values of the variables if any of them were overridden via the CLI. 

### Jobs

If the submitted run refers to a workflow that depends on other workflows, for every workflow, the `dstack` server
schedules a separate job. If the submitted run refers to a workflow that doesn't have dependencies, then
the `dstack` server schedules only one job. Each job refers to a single workflow that must be executed with a particular 
set of variables, dependencies, and state of the repository. 
Generally speaking, jobs are single units of work that can be executed by one machine (aka runner).

### Runners

Runners are machines (either real or virtual) that host the `dstack-runner` daemon. This daemon listens to the
`dstack` server for scheduled jobs. If a job is assigned to a particular runner, the daemon, based on the provided
information, fetches the repository, download the artifacts of the jobs that the given job depends on, and run
the commands of a given job as a Docker container. While the container is being running, the daemon reports 
the logs to `dstack`'s logs storage and upload output artifacts of the job to the `dstack`'s artifact storage.

As a user of `dstack`, you can either install the `dstack-runner` daemon to your own servers to make a pool of 
[your own runners](self-hosted-runners.md),
or provide `dstack` credentials to your own cloud so `dstack` can create runners on-demand using spot instances.

### Lifecycle

1. You define `.dstack/workflows.yaml` and `.dstack/variables.yaml` files inside your project (must be a Git repository).
2. You install the `dstack` CLI via `pip`.
3. You either install `dstack-runner` daemon on your servers, or use the `dstack aws configure` to authorize
`dstack` to use your own cloud to create runners on-demand using spot instances.
4. You use the `dstack` CLI to run workflows, manage runs, jobs, logs, artifacts, runners.
5. When a workflow is submitted via the CLI (e.g. via `dstack run`) , the request is sent to the `dstack` server. 
The `dstack` server creates jobs for the submitted run, and assign them to available runners (either servers where 
you've installed `dstack-runner` or on-demand spot instances in your cloud that you allowed to create).
6. Runners execute assigned jobs, report their logs in real-time, and upload artifacts once the job is finished.

[//]: # (What can dstack used for)