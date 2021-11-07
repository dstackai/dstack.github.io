# GPT-2

This tutorial will walk you through the main steps of using `dstack` on the example of finetuning the famous 
OpenAI's GPT-2.

!!! warning "Prerequisites"

    Before following this tutorial, make sure you've done these required steps:

    1. [Install CLI and get a token](installation.md)
    2. [Set up runners on your own machines](self-hosted-runners.md) or [Connect dstack to your cloud](spot-instances.md)

## Step 1: Clone Git repo

Now, that the `dstack` CLI is installed and runners are set up, go ahead and clone the [`github.com/dstackai/gpt-2`](https://github.com/dstackai/gpt-2)
repository.

## Step 2: Add your Git remote repository

If you're connecting to your Git repository via an SSH key, to authorize `dstack` to access your repository, 
use the following command:

```bash
dstack git remote add --private-key <path to your ssh key> 
```

This command sends the URL of your remote repository and your private key to `dstack.ai`. This information will be
securely shared with the runners that will run workflows.

!!! warning "Repository folder"
    Make sure to run all `dstack` CLI commands from the folder where your Git repository is checked out,
    and where your `.dstack/workflows.yaml` and `.dstack/variables.yaml` files are.

## Step 3: Run a workflow

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

## Step 4: Check run status

Once you've submitted a run, you can see its status, (incl. the jobs associated with it) with the help
of the `dstack status` command:

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

!!! warning "Job is not there?"
      In case you don't see your job in the list, it may mean one of the following: either, the job isn't created yet by 
      the `dstack` server, or there is a problem with your runners. 

## Step 4: Check logs

With the CLI, you can see the output of your run.
If you type `dstack logs --help`, you'll see the following output:

```bash
usage: dstack logs (RUN | JOB)

positional arguments:
  (RUN | JOB)  run name or job id
```

## Step 5: Stop the job

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

## Step 6: Check artifacts

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

## Step 7: Resume the stopped job

Any job that had been stopped can be resumed. This means, the job is restarted with its previously saved output
artifacts. If the job used checkpoints, it will be able to start the work where it stopped and not from the 
beginning.

If you've stopped the previously run `finetune-model` workflow, use this command to resume it:

```bash
usage: dstack resume <stopped finetune-model job id>
```

The job will restore the earlier saved checkpoints and continue finetuning the model.

!!! bug "Submit feedback"
        Something doesn't work or is not clear? Would like to suggest a feature? Please, [let us know](https://forms.gle/nhigiDm4FmjZdRkx5).