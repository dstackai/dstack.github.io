# Quickstart

## How it works

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
    Even though workflows run on users machines, the data on users, runs as well as logs and output artifacts are stored
    with `dstack.ai`. In order to use `dstack`, you have to sign up with `dstack.ai`, and obtain your personal access token.

## Workflows

The very first thing you have to do to use `dstack` is defining the `.dstack/workflows.yaml` file within your project
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

[comment]: <> (&#40;from the [Training GPT-2]&#40;gpt-2.md&#41; tutorial&#41;)
Here's an example of `.dstack/workflows.yaml`:

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
      - curl -O <https://github.com/karpathy/char-rnn/blob/master/data/tinyshakespeare/input.txt>
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
      - PYTHONPATH=src python3 train.py --run_name $job_id --model_name $model --dataset input.npz $variables_as_args
    artifacts:
      - models/$model
      - checkpoint/$job_id
      - samples/$job_id
```

#### Workflow variables

In `.dstack/variables.yaml`, you can define variables (and their default values). Once defined, these variables can be
referenced then from the `.dstack/workflows.yaml` file. 

!!! tip
    Also, variables can be accessed through environment variables from the workflow itself (requires using upper case).

You can define both global variables (shared by all workflows) and individually for each workflow.

[comment]: <> (&#40;from the [Training GPT-2]&#40;gpt-2.md&#41; tutorial&#41;)
Here's an example of `.dstack/variables.yaml`:

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
    memory_saving_gradients: False
    twremat: False
    twremat_memlimit: 12GB
    only_train_transformer_layers: False
    optimizer: adam
    noise: 0.0
    top_k: 40
    top_p: 0.0
    restore_from: latest
    run_name: run1
    sample_every: 100
    sample_length: 1023
    sample_num: 1
    save_every: 1000
    val_dataset: None
    val_batch_size: 2
    val_batch_count: 40
    val_every: 0
```

!!! info
    In addition to the variables that you've defined, `dstack` has the following system variables that are always 
    available for any workflow:
    
    * `$run_name` – the unique ID of the current run
    * `$job_id` – the unique ID of the current job
    * `$variables_as_args` - expands into all variables defined in `.dstack/variables.yaml` for that workflow,
      formatted as `--var_1_name var_1_value, --var_1_name var_1_value ...`; use this variable if you'd like to pass all
      variables into a command

## Runners

The next thing you have to do to use `dstack` is installing the `dstack-runner` daemon on the machines that you'd like to
use to run workflows.

For example, if you have one or several servers (e.g. with GPU) and would like to run workflows there, you'll need to 
launch the `dstack-runner` daemon on these machines. Then, any time when you run a workflow (e.g. using the `dstack` CLI
from your laptop), the `dstack` will create jobs and assign them to run one of these machines (through `dstack-runner` 
daemon). 

Once a runner (the machine that runs the `dstack-runner` daemon) receives a job, it runs in via Docker, stream logs
in realtime, and, after the job is finished, upload output artifacts to the storage.

!!! info "Using your local machine as a runner"
    If you don't plan to use remote machines to run workflows, you can launch the `dstack-runner` daemon locally.

#### Installation

Here's how to install the `dstack-runner` daemon:

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

#### Token

Before you start the `dstack-runner` daemon, you have to configure it with your personal access token:

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

#### Launching

Launching the `dstack-runner` daemon is easy:

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

## CLI

The `dstack` CLI can be used to run workflows (defined with the project files), check status of the runners, 
    and manage jobs.

#### Installation

Here's how to install the CLI via `pip`:

```bash
pip install -i <https://test.pypi.org/simple/> --extra-index-url <https://pypi.org/simple> dstack -U
```

!!! tip 
    The same command can be used to update the `dstack` CLI to the latest version.

#### Token

Next, you have to configure the CLI with your personal access token:

```bash
dstack config --token <your personal access token>
```

!!! note "Project directory"
    Make sure, to run always `dstack` CLI's commands from the directory with your project files (where you have
    `.dstack/workflows.yaml` and `.dstack/variables.yaml` files).

#### Runners

At the previous step, you've set up your runners. You can use the `dstack` CLI to check their status:

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

#### Runs

Once your runners are ready, you can use the CLI to run workflows. 

[comment]: <> (&#40;from the [Training GPT-2]&#40;gpt-2.md&#41; tutorial&#41;)
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

!!! info "What happens when you run a workflow"
    1. When you run a workflow using the `dstack run` command, `dstack` sends the run requests to the `dstack` server
    (hosted with `dstack.ai`). The request contains the information Git repo (incl. the branch, current revision, 
    uncommitted changes, etc.), the name of the workflow, and the overridden variable values. 
    2. Once the `dstack` server receives a run request, it creates a list of jobs associated with the run request. Then,
    it assigns each job to one of the available runners. Since the workflow that you run may depend on other workflows, the
    `dstack` servers creates jobs for these other workflows too (in case they are not cached). 
    3. Each job is assigned by the `dstack` server to available runners.
    4. Once a runner receives a job, it runs, stream logs, and in the end upload output artifacts.

#### Jobs

Once you've run a workflow, you can see the list of jobs associated with it (and their status) by running the following 
command:

```bash
bash jobs
```

If you do, you'll see the following output:

```bash
JOB           WORKFLOW        RUN              RUNNER    STATUS    STARTED    DURATION    ARTIFACTS
275a6207be2e  download-model  wonderful-rat-1  sugar-1   DONE      1 min ago  1 min       models/117M
```

The first column here (`JOB`) is the unique ID of the job. Use that ID when calling other commands, such as `dstack stop`, 
`dstack logs`, and `dstack artifacts`. The third column (`RUN`) is the unique name of the run, associated with a single 
`dstack run` command. You can also use it when calling the commands such as `dstack stop`, and `dstack logs`.

#### Logs

With the CLI, you can see the output of your run. 
Type `dstack logs --help`, to see how to do it:

```bash
usage: dstack logs [(RUN | JOB)]

positional arguments:
  (RUN | JOB)  run name or job id
```

!!! tip 
    You can type `dstack logs` without additional arguments and see all recent logs, or type `dstack logs <job id`>, 
    or `dstack logs <run name>` to filter logs by a job ID or a run name correspondingly.

#### Artifacts

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