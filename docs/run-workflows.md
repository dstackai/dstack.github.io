# Run workflows

## Submit runs

Once your runners are ready, you can use the CLI to run workflows.

!!! note "Project directory"
    Make sure, to run always `dstack` CLI's commands from the directory with your project files (where you have
    `.dstack/workflows.yaml` and `.dstack/variables.yaml` files).

If you type `dstack run --help`, you'll see the syntax of the run command as well as the list of the workflows
defined with `.dstack/workflows.yaml` that you can run. Here's an example:

```bash
usage: dstack run {download-model,encode-dataset,finetune-model} ...

positional arguments:
    download-model      run download-model workflow
    encode-dataset      run encode-dataset workflow
    finetune-model      run finetune-model workflow
```

Now, if you type `dstack run <workflow name> --help`, you'll see that `dstack` is also aware of the variables defined
in `.dstack/variables.yaml` for that workflow. Here's an example:

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
    1. When you run a workflow using the `dstack run` command, `dstack` sends the run requests to the `dstack` server
    (hosted with `dstack.ai`). The request contains the information Git repo (incl. the branch, current revision,
    uncommitted changes, etc.), the name of the workflow, and the overridden variable values.
    2. Once the `dstack` server receives a run request, it creates a list of jobs associated with the run request. Then,
    it assigns each job to one of the available runners. Since the workflow that you run may depend on other workflows, the
    `dstack` servers creates jobs for these other workflows too (in case they are not cached).
    3. Each job is assigned by the `dstack` server to available runners.
    4. Once a runner receives a job, it runs, stream logs, and in the end upload output artifacts.

## Check run status

Once you've run a workflow, you can see the status of the run, the jobs associated with it, and runners with the help
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

The first column (`RUN`) is the unique name of the run, associated with a single `dstack run` command.
The second column (`JOB`) is the unique ID of the job associated with the run. Use that ID when calling other commands,
such as `dstack stop`, `dstack logs`, and `dstack artifacts`. You can also use it when calling the commands such as
`dstack stop`, and `dstack logs`.

## Check logs

With the CLI, you can see the output of your run.
Type `dstack logs --help`, to see how to do it:

```bash
usage: dstack logs (RUN | JOB)

positional arguments:
  (RUN | JOB)  run name or job id
```

## Check artifacts

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