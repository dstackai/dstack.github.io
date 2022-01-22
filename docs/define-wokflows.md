# Define workflows

## Workflows

The `.dstack/workflows.yaml` file is a YAML file within your project files where you define your ML 
workflows, their commands, dependencies, output artifacts, etc.

The root element of that file is `workflows`. This is a list, where each element may have the following
fields:

* `name` – the name of the workflow; required
* `image` – the Docker image that will be used to run the workflow; required
* `commands` – the list of bash commands to run the workflow; required
* `depends-on` – defines what the workflow depends on; optional; if not defined, by default, the workflow depends on all 
files in the repo, on all workflow variables defined in `.dstack/variables.yaml`, and doesn't depend on other workflows; 
within `depends-on`, you may specify the elements `repo`, `variables`, and `workflows`.
* `artifacts` – the list of paths by which all files must be stored as output artifacts in the storage once the workflow
  is done; optional

Here's an example of `.dstack/workflows.yaml` (from the [GPT-2](gpt-2.md) tutorial):

```yaml
workflows:
  - name: download-model
    image: tensorflow/tensorflow:1.15.0-py3
    commands:
      - pip3 install -r requirements.txt
      - python3 download_model.py $model
    artifacts:
      - models
  - name: encode-dataset
    image: tensorflow/tensorflow:1.15.0-py3
    commands:
      - curl -O https://github.com/karpathy/char-rnn/blob/master/data/tinyshakespeare/input.txt
      - pip3 install -r requirements.txt
      - mkdir -p datasets
      - PYTHONPATH=src ./encode.py --model_name $model input.txt datasets/input.npy
    depends-on:
      workflows:
        - download-model
    artifacts:
      - datasets
  - name: finetune-model
    image: tensorflow/tensorflow:1.15.0-py3
    depends-on:
      workflows:
        - download-model
        - encode-dataset
    commands:
      - pip3 install -r requirements.txt
      - PYTHONPATH=src python3 train.py --run_name $run_name --model_name $model --dataset datasets/input.npy $variables_as_args
    artifacts:
      - checkpoint
      - samples
```

## Variables

In `.dstack/variables.yaml`, you can define variables and their default values. These variables can be
referenced then from the `.dstack/workflows.yaml` file.

!!! tip "Environment variables"
    Variables can be accessed as environment variables from the workflow itself (must be in upper case).

You can define both global variables (shared by all workflows) and for each workflow individually.

Here's an example of `.dstack/variables.yaml`(from the [GPT-2](gpt-2.md) tutorial):

```yaml
variables:
  global:
    model: 124M
    models_dir: model

  finetune-model:
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

In addition to your own variables (that you've defined in `.dstack/variables`), `dstack` supports the following 
`system variables` that can be also used workflows:

* `$run_name` – the unique ID of the current run
* `$job_id` – the unique ID of the current job
* `$variables_as_args` - expands into all variables defined in `.dstack/variables.yaml` for that workflow,
  formatted as `--var_1_name var_1_value, --var_1_name var_1_value ...`; use this variable if you'd like to pass all
  variables into a command

