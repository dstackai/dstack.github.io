# Overview 

### What is dstack? 

dstack allows AI researchers to define their workflows declaratively, and then run these workflows interactively 
and have infrastructure provisioned automatically.

Here's a very simple workflow example:

```yaml
workflows:
  - name: train
    image: tensorflow/tensorflow:latest-gpu
    commands:
      - python3 train.py
    artifacts:
      - checkpoint
    resources:
      gpu: 4
```

Run this workflow via the CLI:

```bash
dstack run train -f
```

Once you do that, dstack provisions an instance that has 4 GPU and assigns this workflow to that instance. 
While the workflow is running, the output artifacts and output logs are tracked in real-time.

!!! info ""
    For more advanced documentation about workflows, proceed to [Workflows](workflows.md).

### Why use dstack?

#### Infrastructure provisioning

Once you run a workflow, dstack provisions infrastructure according to the workflow requirements and account limits. 
Once the workflow is finished, the infrastructure is torn down. You can use either your own servers or connect
dstack to your cloud accounts and have infrastructure set up on demand.

!!! info ""
    To learn more about runners, check out [On-demand runners](on-demand-runners.md) and [Self-hosted runners](self-hosted-runners.md).

#### Data versioning

The output artifacts of any run are stored in an immutable storage. Once a run is finished, it can be marked with a tag and 
used later as a dependency for another workflow.

#### Reproducible pipelines

Typically, complex workflows include multiple steps, such as pre-processing data, training, fine-tuning, validation, etc.
With dstack, it's possible to define each step as a separate workflow, and then run any of these workflows interactively
from the CLI.