# Run workflows

Before you can run and manage workflows of your project via the CLI, you have to authorize `dstack` to access
your Git remote repository. 

## Add your Git remote repository

If you're connecting to your Git repository via an SSH key, to authorize `dstack` to access your repository, 
use the following command:

```bash
dstack git remote add --private key <path to your ssh key> 
```

This command sends the URL of your remote repository and your private key to `dstack.ai`. This information will be
securely shared with the runners that will run workflows.

!!! warning "Repository folder"
    Make sure to run all `dstack` CLI commands from the folder where your Git repository is checked out,
    and where your `.dstack/workflows.yaml` and `.dstack/variables.yaml` files are.

## Run workflows

If you type `dstack run --help`, you'll see the syntax of the run command as well as the list of the workflows
defined with `.dstack/workflows.yaml` that you can run. 

Here's an example (from the [GPT-2](gpt-2.md) tutorial):

```bash
usage: dstack run {download-model,encode-dataset,finetune-model} ...

positional arguments:
    download-model      run download-model workflow
    encode-dataset      run encode-dataset workflow
    finetune-model      run finetune-model workflow
```

If you type `dstack run <workflow name> --help`, you'll see that `dstack` is also aware of the variables defined
in `.dstack/variables.yaml` for that workflow. 

Here's an example (again, from the [GPT-2](gpt-2.md) tutorial):

```bash
usage: dstack run download-model [--model [MODEL]] [--models_dir [MODELS_DIR]]

optional arguments:
  --model       [MODEL]       default is 124M
  --models_dir  [MODELS_DIR]  default is model
```

If you want to run `download-model` from the example above and override the `model` variable, try this:

```bash
dstack run download-model --model 117M
```

!!! info "What happens when you run a workflow?"
    1. When you run a workflow using the `dstack run` command, `dstack` sends the run requests to `dstack.ai`. 
    The request contains the information about the repository (incl. the branch, the commit hash of the head,
    uncommitted changes if any, etc.), the name of the workflow, and the overridden variable values.
    2. Once the `dstack` server receives a run request, it creates a list of jobs (one job per workflow that has to run).
    Then, `dstack.ai` assigns each job to one of the available runners. If any of the workflows (one that you run or one
    it depends on) had already run before with the code and variables, `dstack.ai` won't create a new job and instead
    will reuse the one from cache.
    3. Each job is assigned to one of the available runners.
    4. Once a runner receives a job, it runs, upload logs in real-time, and in the end upload output artifacts.

## Check run status

Once you've submitted a run, you can see its status, (incl. the jobs associated with it) with the help
of the `dstack status` command:

```bash
dstack status
```

If call it, you'll see the following output:

```bash
RUN           JOB           WORKFLOW        VARIABLES      RUNNER    STATUS    STARTED      DURATION    ARTIFACTS
angry-rat-1                 download-model  --model 117M   sugar-1   DONE      1 min ago    -
              d0e3d8d0a1ff  download-model  --model 117M   sugar-1   DONE      1 min ago    2 mins      models/117M
```

!!! warning ""
    By default, the `dstack status` command lists all unfinished runs. If there are no unfinished runs,
    it lists the last finished run. If you'd like to see more finished runs, use the `--last <n>` argument to
    specify the number of last runs to show regardless of their status.

## Check logs

With the `dstack` CLI, you can see the output of the entire run or individual of any job associated with it by its job ID.

Type `dstack logs --help`, to see how to do it:

```bash
usage: dstack logs (RUN | JOB)

positional arguments:
  (RUN | JOB)  run name or job id
```

## Check artifacts

Every job at the end of its execution saves its artifacts in the storage.
With the `dstac` CLI, you can do both list the contents of each artifact and download artifacts to your local machine.

Here's how to list the content of the artifacts of a given job:

```bash
dstack artifacts list <job id>
```

By default, this will only show the list of artifacts, the number of files in each, and the total size of the artifact.

In order to see all file stored within each artifact, use `-l` option:

```bash
dstack artifacts list -l <job id>
```

If you'd like to download artifacts, use the following command:

```bash
dstack artifacts download <job id>
```

By default, it will download the artifacts into the current working directory. The output directory can be overridden
with the use of the `--output <path>` argument.

!!! bug "Submit feedback"
    Something doesn't work or is not clear? Would like to suggest a feature? Please, [let us know](https://forms.gle/nhigiDm4FmjZdRkx5).