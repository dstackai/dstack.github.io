# Getting started

## Introduction

Typical ML workflows consist of multiple steps, e.g. pre-processing data, training, fine-tuning, validation, etc.

`dstack` les you to define these steps in a simple YAML format, and then, run any of them on any infrastructure of your 
choice via the CLI.

If you use one or several machines to run ML workflows, you can install the `dstack-runner` 
daemon on all these machines, and submit workflows to run there from your laptop through the `dstack` CLI. 

The `dstack-runner` daemon will manage dependencies, run ML workflows through Docker, and upstream logs and output 
artifacts to the storage.

### Key components 

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

### How it works

1. When you run a workflow using the `dstack run` command, `dstack` sends the run requests to the `dstack` server
(hosted with `dstack.ai`). The request contains the information Git repo (incl. the branch, current revision,
uncommitted changes, etc.), the name of the workflow, and the overridden variable values.
2. Once the `dstack` server receives a run request, it creates a list of jobs associated with the run request. Then,
it assigns each job to one of the available runners. Since the workflow that you run may depend on other workflows, the
`dstack` servers creates jobs for these other workflows too (in case they are not cached).
3. Each job is assigned by the `dstack` server to available runners.
4. Once a runner receives a job, it runs, stream logs, and in the end upload output artifacts.

## Step 1: Define workflows

The first thing you have to do to get started with dstack is creating the `.dstack/workflows.yaml` file in your project
files.

The root element of that file is called workflows. This is a list. Each element in this list may have the following
fields:

* `name` – the name of the workflow; required
* `image` – the Docker image that will be used to run the workflow; required
* `commands` – the list of bash commands to run the workflow; required
* `depends-on` – defines what the workflow depends on; optional, if not defined, by default, the workflow depends on all
  files in the repo, on all workflow variables defined in `.dstack/variables.yaml`, and doesn't depend on other workflows
  ; within depends-on, you may specify the elements `repo`, `variables`, and `workflows`.
* `artifacts` – the list of paths by which all files must be stored as output artifacts in the storage once the workflow
  is done; optional

Here's an example of `.dstack/workflows.yaml` (from the [Training GPT-2](gpt-2.md) tutorial):

```yaml
workflows:
  - name: download-model
    image: tensorflow/tensorflow:1.15.0-py3
    commands:
      - pip3 install -r requirements.txt
      - python3 download_model.py $model
    depends-on:
      repo:
        include:
          - requirements.txt
          - download_model.py
    artifacts:
      - models/$model

  - name: encode-dataset
    image: tensorflow/tensorflow:1.15.0-py3
    commands:
      - curl -O https://github.com/karpathy/char-rnn/blob/master/data/tinyshakespeare/input.txt
      - pip3 install -r requirements.txt
      - PYTHONPATH=src ./encode.py --model_name $model input.txt input.npz
    depends-on:
      repo:
        include:
          - requirements.txt
          - encode.py
          - src/load_dataset.py
          - src/encoder.py
      workflows:
        - download-model
    artifacts:
      - input.npz

  - name: finetune-model
    image: tensorflow/tensorflow:1.15.0-py3
    depends-on:
      workflows:
        - download-model
        - encode-dataset
    commands:
      - pip3 install -r requirements.txt
      - PYTHONPATH=src python3 train.py --run_name $run_name --model_name $model --dataset input.npz $variables_as_args
    artifacts:
      - checkpoint/$run_name
      - samples/$run_name
```

### Workflow variables

In `.dstack/variables.yaml`, you can define variables (and their default values). Once defined, these variables can be
referenced then from the `.dstack/workflows.yaml` file.

!!! tip "Environment variables"
Also, variables can be accessed through environment variables from the workflow itself (requires using upper case).

You can define both global variables (shared by all workflows) and individually for each workflow.

Here's an example of `.dstack/variables.yaml`(from the [Training GPT-2](gpt-2.md) tutorial):

```yaml
variables:
  global:
    model: 124M
    models_dir: model

  finetune-model:
    dataset: input.npz
    combine: 50000
    encoding: utf-8
    batch_size: 1
    learning_rate: 0.00002
    accumulate_gradients: 1
    twremat_memlimit: 12GB
    optimizer: adam
    noise: 0.0
    top_k: 40
    top_p: 0.0
    restore_from: latest
    sample_every: 100
    sample_length: 1023
    sample_num: 1
    save_every: 1000
    val_batch_size: 2
    val_batch_count: 40
    val_every: 0
```

!!! info ""
    In addition to the variables that you've defined, `dstack` has the following system variables that are always
    available for any workflow:

    * `$run_name` – the unique ID of the current run
    * `$job_id` – the unique ID of the current job
    * `$variables_as_args` - expands into all variables defined in `.dstack/variables.yaml` for that workflow,
      formatted as `--var_1_name var_1_value, --var_1_name var_1_value ...`; use this variable if you'd like to pass all
      variables into a command

## Step 2: Install CLI

The next step is installing the `dstack` CLI. This can be done via `pip`:

```bash
pip install -i https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple dstack -U
```

!!! note ""
    The same command can be used to update the `dstack` CLI to the latest version.

### Registering a user

Before you can use the `dstack` CLI or set up runners, you have to register a user with `dstack.ai`.

Here's how to do it via the CLI:

```bash
dstack register
```

It will prompt you to select a user name (only latin characters, digits and underscores are allowed), and specify your
email. Once you provide that, to verify the email address, it will send to this email a verification code that you'll 
have to specify right away.

That's it. Now, you are fully authorized on this machine to perform any operations via the CLI.

!!! note ""
    The authorization is done via the `Personal Access Token` that is stored in `~/.dstack/config.yaml` file.

!!! note "Login as existing user"
    If you'll ever have to authorize the `dstack` CLI again on this or another machine, you'll can do that by using 
    the following command:

    ```bash 
    dstack login
    ```
    
    This command needs the user name and the password. If it's correct, it authorizes the current machine to use 
    the `dstack` CLI.

### Get a token

At the next step, when you'll be setting up runners, you'll need your `Personal Access Token`.
This token can be obtained by the following command:

```bash
dstack token
```

Keep the token that the CLI gives you. You'll need it to configure the `dstack-runner` daemon.

## Step 3: Set up runners

!!! note "What is a runner?"
    A runner is any machine which you'd like to use to run `dstack` workflows. In order to use any machine
    as a runner, you have to launch the `dstack-runner` daemon on that machine. 
    
    All machines that have the `dstack-runner` daemon running form a pool of runners. 

    When you later run workflows via the `dstack` CLI, these workflows are assigned to these runners.

!!! info "Run runners locally"
    If you don't plan to use remote machines, you can launch the `dstack-runner` daemon locally.

Here's how you can install the `dstack-runner` daemon:

=== "Linux"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-linux-amd64"
    sudo chmod +x /usr/local/bin/dstack-runner
    ```

=== "macOS"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-darwin-amd64"
    sudo chmod +x /usr/local/bin/dstack-runner
    ```

If you are using **Windows**, download [dstack-runner.exe](https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-windows-amd64.exe), and run it.

### Configure a token

Before you start the daemon, you have to configure it with your `Personal Access Token`:

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

That's it. Now, you can launch the daemon:

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

!!! warning "Docker is required"
    `dstack-runner` requires that either the standard Docker or the NVIDIA's Docker is installed and running on the
    machine.

### Check runners

After you've set up your runners, you can use the `dstack` CLI to check their status:

```bash
dstack runners 
```

If runners are running properly, you'll see their hosts in the output:

```bash
RUNNER    HOST                    STATUS    UPDATED
sugar-1   MBP-de-Boris.fritz.box  LIVE      3 mins ago
```

!!! warning "Runner is not there?"
    If you don't see your runner in the output, this means the runner is offline or that the `dstack-runner` daemon 
    was not configured or launched properly.

## Step 4: Run workflows

### Run workflows

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

!!! warning ""

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

### Check run status

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

### Check logs

With the CLI, you can see the output of your run. 
Type `dstack logs --help`, to see how to do it:

```bash
usage: dstack logs (RUN | JOB)

positional arguments:
  (RUN | JOB)  run name or job id
```

### Check artifacts

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