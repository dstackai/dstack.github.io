# Training GPT-2

This tutorial will walk you through the main steps of using `dstack` on the example of training the famous 
OpenAI's GPT-2.

## Introduction

Before using `dstack` you have to understand how `dstack` works. If you're already familiar, skip to 
[Set up a runner](#step-1-set-up-a-runner).

Typical ML workflows consist of multiple steps, e.g. pre-processing data, training, fine-tuning, validation, etc. 
`dstack` les you to define these steps in a simple YAML format, and then, run any of them on any infrastructure of your
choice via the CLI.

#### What are the key components of dstack?

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

#### What happens when you run a workflow?

1. When you run a workflow using the `dstack run` command, `dstack` sends the run requests to the `dstack` server
   (hosted with `dstack.ai`). The request contains the information Git repo (incl. the branch, current revision,
   uncommitted changes, etc.), the name of the workflow, and the overridden variable values.
2. Once the `dstack` server receives a run request, it creates a list of jobs associated with the run request. Then,
   it assigns each job to one of the available runners. Since the workflow that you run may depend on other workflows, the
   `dstack` servers creates jobs for these other workflows too (in case they are not cached).
3. Each job is assigned by the `dstack` server to available runners.
4. Once a runner receives a job, it runs, stream logs, and in the end upload output artifacts.

!!! info "Personal access token"
    Even though workflows run on your machines, logs, output artifacts, and other metadata are stored with `dstack.ai`.
    In order to use `dstack`, you have to sign up with `dstack.ai`, and obtain your personal access token.

## 1. Set up a runner

!!! note "What is a runner?"
    A runner is any machine which you'd like to use to run `dstack` workflows. In order to use any machine
    as a runner, you have to launch the `dstack-runner` daemon on that machine. All machines that have the `dstack-runner`
    daemon running form a pool of runners. When you later run workflows via the `dstack` CLI, these workflows will be
    assigned to run on runners.

!!! info "Run workflows locally"
    If you don't plan to use remote machines to run workflows, you can launch the `dstack-runner` daemon locally.

Here's how you can install the `dstack-runner` daemon:

=== "Linux"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-linux-amd64"
    sudo chmod +x /usr/local/bin/dstack-runner/dstack-runner
    ```

=== "macOS"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-darwin-amd64"
    sudo chmod +x /usr/local/bin/dstack-runner/dstack-runner
    ```

If you are using **Windows**, download [dstack-runner.exe](https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-windows-amd64.exe), and run it.

Note, before you start the daemon, you have to configure it with your personal access token:

=== "Linux"

    ```bash
    dstack-runner config --token <token>
    ```

=== "macOS"

    ```bash
    dstack-runner config --token <token>
    ```

=== "Windows"

    ```cmd
    dstack-runner.exe config --token <token>
    ```

Now you can launch the daemon:

=== "Linux"

    ```bash
    dstack-runner start
    ```

=== "macOS"

    ```bash
    dstack-runner start
    ```

=== "Windows"

    ```cmd
    dstack-runner.exe start
    ```

!!! warning "Runner requires Docker"
    `dstack-runner` requires that either the standard Docker or the NVIDIA's Docker is installed and running on the
    machine.

## 2. Install CLI

Now, that the runners are ready, we can install the `dstack` CLI to run workflows on these runners.

Here's how to install the CLI via `pip`:

```bash
pip install -i <https://test.pypi.org/simple/> --extra-index-url <https://pypi.org/simple> dstack -U
```

!!! tip
    The same command can be used to update the `dstack` CLI to the latest version.

Next, you have to configure the CLI with your personal access token:

```bash
dstack config --token <your personal access token>
```

## 3. Check runner status

Once the CLI is configured, you can use it to check the status of runners:

```bash
dstack runners 
```

You'll see the following output:
```bash
RUNNER    HOST                        STATUS
sugar-1   MacBook-Pro-de-Boris.local  LIVE
```

!!! warning "If your runner is not there"
    If you don't see your runner, this means the runner is offline, and you have to check whether the VM is up or
    that the `dstack-runner` daemon is configured and launched properly.

## 4. Clone Git repo

Now, it's time to clone the [Git repository](https://github.com/dstackai/gpt-2) with our project files.

The Git repository that we are going to use in this tutorial already has the `.dstack` folder with 
the [`workflows.yaml`](https://github.com/dstackai/gpt-2/blob/finetuning/.dstack/workflows.yaml) and 
[`variables.yaml`](https://github.com/dstackai/gpt-2/blob/finetuning/.dstack/variables.yaml) files.

## 5. Run a workflow

!!! note "Project directory"
      Make sure, to run always `dstack` CLI's commands from the directory with your project files (where you have
      `.dstack/workflows.yaml` and `.dstack/variables.yaml` files).

If you type `dstack run --help`, you'll see the following output:

```bash
usage: dstack run [-h] {download-model,encode-dataset,finetune-model} ...

positional arguments:
  {download-model,encode-dataset,finetune-model}
    download-model      run download-model workflow
    encode-dataset      run encode-dataset workflow
    finetune-model      run finetune-model workflow
```

Here, the CLI shows you list of workflows that you have defined in your `.dstack/workflows.yaml` file.

If you type `dstack run download-model --help`, you'll see the following output:

```bash
dstack run download-model --help
usage: dstack run download-model [--model [MODEL]] [--models_dir [MODELS_DIR]]

optional arguments:
  --model [MODEL]              by default, the value is "124M"
  --models_dir [MODELS_DIR]̋̋̋    by default, the value is is "model"
```

Here, the CLI shows you the variables defined for the `download-model` workflow in `.dstack/variables.yaml`.

Now, let's go and run the `finetune-model` workflow:

```bash
dstack run finetune-model 
```

If you want, alternatively, you can override the default variable `model` with another value, e.g. `"117M"`:

```bash
dstack run finetune-model --model 117M 
```

This is how you override any variables that you have defined in `.dstack/variables.yaml`.

## 6. Check jobs

After we call the `dstack run` command, the CLI sends the request with all the information about our run (incl. 
the Git repo, the uncommitted changes, the overridden variable values, etc.) to the `dstack` server. The server
creates job for the requested workflow (in our case it's `finetune-model`) and all the workflows the requested 
workflow depends on (in our case these include `download-model` and `encode-dataset`).

!!! info "Workflow cache"
      In case the requested workflow depends on a workflow that already ran before with the exact same parameters,
      it won't run again. Instead, `dstack` will reuse the previously run job's artifacts.

To see the created job and their status, run the following command:

```bash
dstack jobs
```

Here's what you can see if you do that:

```bash
JOB           RUN             WORKFLOW        STATUS     STARTED    DURATION    RUNNER    ARTIFACTS
0673f7f444b6  pink-fly-1      download-model  DONE       6 min ago  1 min       sugar-1   models/124M
67d53f2daa2e  pink-fly-1      encode-dataset  DONE       5 min ago  1 min       sugar-1   input.npz
4ef42541c7f4  clever-tiger-1  finetune-model  RUNNING    just now   1 sec       sugar-1   -
```

The first column (`JOB`) here is the unique ID of the job. Use that ID when calling other commands, such as 
`dstack stop`, `dstack logs`, and `dstack artifacts`. The third column (`RUN`) is the unique name of the run, 
associated with a single `dstack run` command. You can also use it when calling the commands such as `dstack stop`, 
and `dstack logs`.

!!! warning "If jobs are not there"
      In case you don't see jobs in the list, it may mean one of the following: either, the jobs aren't created yet by 
      the `dstack` server, or there is a problem with your runners. 

## 7. Check logs

With the CLI, you can see the output of your run.
If you type `dstack logs --help`, you'll see the following output:

```bash
usage: dstack logs [(RUN | JOB)]

positional arguments:
  (RUN | JOB)  run name or job id
```

You can run `dstack run` with or without an ID of a job or a name of a run.

Try this command:

```bash
dstack logs
```

In this case, it's supposed to print all the recent logs. 

## 8. Stop job

Any job can be stopped at any time. If you type `dstack stop --help`, you'll see the following:

```bash
usage: dstack stop [--abort] (RUN | JOB)

positional arguments:
  (RUN | JOB)  run name or job id

optional arguments:
  --abort      abort the job (don't upload artifacts)
```

This command requires specifying an ID of a job or a name of a run. Also, if you want, you can specify the `--abort`
argument. In this case, `dstack` will not upload the output artifacts of the stopped job.

## 9. Check artifacts

Every job at the end of its execution stores its artifacts in the storage.
With the CLI, you can both list the contents of each artifact and download it to your local machine.

Here's how to list the content of the artifacts of a given job:

```bash
dstack artifacts list <job id>
```

By default, this will only show the list of artifacts, the number of files in each, and the total size of the artifact.

In order to see all file stored within each artifact, use `-l` option:

```bash
dstack artifacts list -l <job id>
```

!!! info "Something didn't work or was unclear? Miss a critical feature? Please, [let me know](https://forms.gle/nhigiDm4FmjZdRkx5). I'll look into it ASAP." 