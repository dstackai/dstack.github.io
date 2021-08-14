# Quickstart

## Introduction

Typical ML workflows consist of multiple steps, e.g. pre-processing data, training, fine-tuning, validation, etc.

`dstack` les you to define these steps (in a simple YAML format), and then, run any of them over a pool of remote
machines (from your local machine).

## Ket components

1. `.dstack/workflows.yaml` – you create this file in your project files, and define there all your workflows that you
   have in your project. For every workflow, you may specify the Docker image, the commands to run the workflow, the
   dependencies to other workflows, output artifacts, etc.
2. `.dstack/config.yaml` – you create this file in your project files, and define there all the variables (and their
   default values) your workflow may depend on. These variables can be referenced from `.dstack/workflows.yaml`.
3. `dstack-runner` – you install and start this daemon on the machines that you'd like to use to run your workflows; you
   can use any number of machines (to make a pool of available runners); once the daemon is started, it runs assigned
   workflows, stream logs, and upload output artifacts to the storage.
4. `dstack` – you install this CLI (Command Line Interface) via `pip` and use from the terminal (e.g. from your laptop)
   to run the workflows defined in `.dstack/workflows.yaml`. When running workings with this CLI, you can override any
   variables used in the workflow.

## How it works

1. Once you submit a new run (using the `dstack` CLI), the run request is sent to the `dstack` server. This inclues the
   information on the Git repo (incl. the branch, current revision, uncommited changes, etc), the name of the workflow,
   and the overridden variable values if any.
2. Once the `dstack` server (currently is hosted by `dstack.ai`) receives a run request, it creates a list of jobs
   associated with the run request. Then it assigns each job to one of the available runners.
3. Once a runner (the machine that runs the `dstack-runner` daemon) recieves a job, it runs in via Docker, stream logs
   in realtime, and, after the job is finished, upload output artifacts to the storage.

!!! note "Personal access token"
    Even though workflows run on users machines, the data on users, runs as well as logs and output artifacts are stored
    with `dstack.ai`. In order to use `dstack`, you have to sign up with dstack.ai, and opbtain your personal access token.

## Getting started

### Defining workflows

The very first thing you have to do to use `dstack` is defining the `.dstack/workflows.yaml` file within your project
files.

The root element of that file is called workflows. This is a list. Each element in this list may have the following
fields:

* `name` – the name of the workflow; required
* `image` – the Docker image that will be used to run the workflow; required
* `commands` – the list of bash commands to run the workflow; required
* `depends-on` – defines what the workflow depends on; optional, if not defined, by default, the workflow depends on all
  files in the repo, on all workflow variables defined in `.dstack/configs.yaml`, and doesn't depend on other workflows)
  ; within depends-on, you may specify the elements repo, config, and workflows.
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
      - PYTHONPATH=src python3 train.py --run_name $job_id --model_name $model --dataset input.npz $config_args
    artifacts:
      - models/$model
      - checkpoint/$job_id
      - samples/$job_id
```

!!! note "Workflow variables"
    Within the `.dstack/workflows.yaml` file, you can reference any variables defined in the `.dstack/configs.yaml` file (
    e.g. `$model`).

    In addition to the variables that you've defined in `.dstack/configs.yaml`, you can reference the following reserved variables:

    * `$run_name` – the unique ID of the current run
    * `$job_id` – the unique ID of the current job
    * `$config_args` - expands into all variables defined in `.dstack/configs.yaml` for that workflow,
    formatted as `--var_1_name var_1_value, --var_1_name var_1_value ...`; use this variable if you'd like to pass all
    variables into a command

### Defining configs

In `.dstack/configs.yaml`, you can define variables (and their default values) to reuse from your workflows. Variables
can be referenced from the `.dstack/workflows.yaml` or they can be accessed as environment variables from the workflow
(requires using upper case).

In `.dstack/configs.yaml`, you can define both global variables (shared by all workflows) and individually for each
workflow.

[comment]: <> (&#40;from the [Training GPT-2]&#40;gpt-2.md&#41; tutorial&#41;)
Here's an example of `.dstack/configs.yaml`:

```yaml
configs:
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

### Configuring runners

The next thing you have to do to use `dstacl` is installing the `dstack-runner` daemon on the machines that you'd like to
use to run workflows.

For example, if you have one or several servers (e.g. with GPU) and would like to run workflows there, you'll need to 
launch the `dstack-runner` daemon on these machines. Then, any time when you run a workflow (e.g. using the `dstack` CLI
from your laptop), the `dstack` will create jobs and assign them to run one of these machines (through `dstack-runner` 
daemon). 

Once a runner (the machine that runs the `dstack-runner` daemon) recieves a job, it runs in via Docker, stream logs
in realtime, and, after the job is finished, upload output artifacts to the storage.

!!! note "Using your local machine as a runner"
    If you don't plan to use remote machines to run workflows, you can launch the `dstack-runner` daemon locally.

Here's how to install and launch the `dstack-runner`:

=== "Linux"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-linux-amd64"
    chmod +x dstack-runner
    dstack-runner
    ```

=== "macOS"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-darwin-amd64"
    chmod +x dstack-runner
    dstack-runner
    ```

If you are using **Windows**, download [dstack-runner.exe](https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-windows-amd64.exe), and run it.

!!! warning "Runner requires Docker"
    `dstack-runner` requires that either the standard Docker or the NVIDIA's Docker is installed and running on the machine

### Running workflows

In order to run and manage workflows from the `dstack` CLI, we have to install the CLI and configure it with our 
personal access token.

#### Install the dstack CLI

Here's how to install the `dstack` CLI via `pip`:

```bash
pip install -i <https://test.pypi.org/simple/> --extra-index-url <https://pypi.org/simple> dstack -U
```

#### Configure the CLI with the token

The final thing to set up the CLI is configuring it with your personal access token. You can do that by running the 
following command:

```bash
dstack config --token <your personal access token>
```

That's it. Now are are able to run workflows using the `dstack` CLI. If you type 
`dstack run --help`, you'll see the following information:

```bash
usage: dstack run [-h] {download-model,encode-dataset,finetune-model} ...

positional arguments:
  {download-model,encode-dataset,finetune-model}
    download-model      run download-model workflow
    encode-dataset      run encode-dataset workflow
    finetune-model      run finetune-model workflow

optional arguments:
  -h, --help            show this help message and exit
```

As you see, the command is aware of the workflows defined in `.dstack/workflows.yaml`.

Now, if you type `dstack run download-model --help`, you'll see that `dstack` is also aware of the variables defined
for that workflow:

```bash
usage: dstack run download-model [-h] [--model [MODEL]]
                                 [--models_dir [MODELS_DIR]]

optional arguments:
  -h, --help            show this help message and exit
  --model [MODEL]       default is 124M
  --models_dir [MODELS_DIR]
                        default is model
```

!!! note "Project directory"
    Make sure, to run `dstack` CLI's commands from the directory with your project files (where you have
    `.dstack/workflows.yaml` and `.dstack/configs.yaml` files).

If we want to run `download-model` and override the `model` variable, we would use the following command:

```bash
dstack run download-model --model 117M
```

!!! info "What happens when we run a workflow"
    1. When we run a workflow using the `dstack run` command, `dstack` sends the run requests to the `dstack` server
    (hosted with `dstack.ai`). The request contains the information Git repo (incl. the branch, current revision, uncommited changes, etc), the name of the workflow,
    and the overridden variable values. 
    2. Once the `dstack` server receives a run request, it creates a list of jobs associated with the run request. Then,
    it assigns each job to one of the available runners. Since the workflow that we run may depend on other workflows, the
    `dstack` servers creates jobs for these other workflos too (in case they are not cached). 
    3. Each job is assigned by the `dstack` server to available runners.
    4. Once a runner recieves a job, it runs, stream logs, and in the end upload output artifacts.

### CLI reference

If you run the `dstack --help` command, you'll see what commands it supports:

```bash
usage: dstack [-h] [--version]
              {config,run,runners,runs,jobs,stop,logs,artifacts} ...

positional arguments:
  {config,run,runners,runs,jobs,stop,logs,artifacts}
    config              manage your configuration
    run                 run workflow
    runners             show live runners
    runs                show recent runs
    jobs                show recent jobs
    stop                stop run or job
    logs                see logs of a run or a job
    artifacts           manage artifacts of a given job

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit

Please visit https://docs.dstack.ai for more information
```

To see what arguments are available for each of these command, type `dstack <command> --help`.

### Limitations

There is a long list of the things that `dstack` doesn't support yet:

* **Private repositories**: currently, `dstack` only works with public Git repositories. Private Git repositories 
  cannot be used with `dstack` yet.
* **Dashboard interface**: currently, you can work with `dstack` only via the `dstack`. There is not yet an user interface 
  to sign up, and manage runners, runs, artifacts, and logs.
* **Hardware metrics**: currently, `dstack` doesn't provide the hardware metrics for the runners, e.g. the use and 
  availability of memory, CPU, GPU, etc.
* **Hardware requirements**: currently, there is no way to specify hardware requirements per workflow (e.g. GPU, CPU,
  memory, etc)
* **Own storage**: currently, `dstack` uses its own storage for artifacts and logs. There is not yet
  a way to use own storage.
* **Own server**: currently, the `dstack` server is hosted with `dstack.ai` and stores the information about
  the user, runs, and runners on its own server. There is not yet a way to have a `dstack` server up on your own 
  machine.
* **Triggers**: currently, there is no way to configure your own events or webhooks to trigger new runs. All runs must 
  be manually started via the `dstack` CLI.
* **Sweeps**: currently, there is no way to run a multiple combinations of variables in parallel (i.e. hyperparameter 
  tuning, e.g. grid search, bayesian search, etc).