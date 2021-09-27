# Training GPT-2

This tutorial will walk you through the main steps of using `dstack` on the example of training the famous 
OpenAI's GPT-2.

## Prerequisites

!!! success "Set up CLI and runners"

    - [ ] [Install CLI and obtain a token](install-cli.md)
    - [ ] Set up [own runners](set-up-own-runners.md)

## Step 1: Clone Git repo

Now, it's time to clone the [Git repository](https://github.com/dstackai/gpt-2) with our project files.

The Git repository that we are going to use in this tutorial already has the `.dstack` folder with 
the [`workflows.yaml`](https://github.com/dstackai/gpt-2/blob/finetuning/.dstack/workflows.yaml) and 
[`variables.yaml`](https://github.com/dstackai/gpt-2/blob/finetuning/.dstack/variables.yaml) files.

## Step 2: Run a workflow

!!! warning "Git repository"
    Make sure, to run always `dstack` CLI's commands from the directory with your project files (where you have
    `.dstack/workflows.yaml` and `.dstack/variables.yaml` files). Your project directory must be a Git repository
    and with a configured remote repository (e.g. `origin`).

    Note, currently `dstack` supports only public Git repositories. Make sure to use HTTPS protocol for authentication.
    The SSH protocol is not supported yet.

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

## Step 2: Check the run status

After we call the `dstack run` command, the CLI sends the request with all the information about our run (incl. 
the Git repo, the uncommitted changes, the overridden variable values, etc.) to the `dstack` server. The server
creates job for the requested workflow (in our case it's `finetune-model`) and all the workflows the requested 
workflow depends on (in our case these include `download-model` and `encode-dataset`).

!!! info "Cached runs"
      In case the requested workflow depends on a workflow that already ran before with the exact same parameters,
      it won't run again. Instead, `dstack` will reuse the previously run job's artifacts, and won't create a new job.

To see created job and their status, run the following command:

```bash
dstack status
```

Here's what you can see if you do that:

```bash
RUN             JOB           WORKFLOW        VARIABLES      RUNNER    STATUS    STARTED      DURATION    ARTIFACTS
clever-tiger-1  -             download-model  --model 117M   sugar-1   RUNNING   6 mins ago   6 mins      -
                0673f7f444b6  download-model  --model 117M   sugar-1   DONE      6 mins ago   2 mins      models/117M
                67d53f2daa2e  download-model  --model 117M   sugar-1   DONE      5 mins ago   2 mins      input.npz
                4ef42541c7f4  finetune-model  --model 117M   sugar-1   RUNNING   just now     2 mins      checkpoint/clever-tiger-1
                                                                                                          samples/clever-tiger-1
```

!!! warning ""
    By default, the `dstack status` command lists all unfinished runs. If there are no unfinished runs,
    it lists the last finished run. If you'd like to see more finished runs, use the `--last <n>` argument to
    specify the number of last runs to show regardless of their status.

The first column (`RUN`) is the unique name of the run, associated with a single `dstack run` command.
The second column (`JOB`) is the unique ID of the job associated with the run. Use that ID when calling other commands,
such as `dstack stop`, `dstack logs`, and `dstack artifacts`. You can also use it when calling the commands such as
`dstack stop`, and `dstack logs`.

!!! warning "Job is not there?"
      In case you don't see your job in the list, it may mean one of the following: either, the job isn't created yet by 
      the `dstack` server, or there is a problem with your runners. 

## Step 3: Check logs

With the CLI, you can see the output of your run.
If you type `dstack logs --help`, you'll see the following output:

```bash
usage: dstack logs (RUN | JOB)

positional arguments:
  (RUN | JOB)  run name or job id
```

## Step 4: Stop the job

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

## Step 5: Check artifacts

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

If you'd like to download the artifacts, this can be done by the following command:

```bash
dstack artifacts download <job id>
```

By default, it will download the artifacts into the current working directory. The output directory can be overridden 
with the use of the `--output <path>` argument.

!!! bug "Submit feedback"
        Something didn't work or was unclear? Miss a critical feature? Please, [let me know](https://forms.gle/nhigiDm4FmjZdRkx5). I'll look into it ASAP.