# Define workflows

## Workflows

The `.dstack/workflows.yaml` file is a YAML file within your project files where you define the steps of your ML 
workflows, their commands, dependencies, and output artifacts.

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

## Variables

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

