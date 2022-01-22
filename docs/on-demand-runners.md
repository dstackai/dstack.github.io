# On-demand runners in your cloud

A runner is a machine that can run `dstack` workflows. If you provide `dstack` with the credentials to your cloud 
and provide autoscaling rules, `dstack` will be able to set up servers to run your workflows automatically – based on 
the demand and workflow requirements. 

!!! warning "Cloud vendors"
    This tutorial describes how to use the `dstack autoscale` feature with an AWS account. 
    If you use another cloud vendor, such as GCP, Azure (or some other), please write to 
    [hello@dstack.ai](mailto:hello@dstack.ai).

!!! info "How on-demand runners work"

    1. You provide `dstack` credentials to create EC2 instances in your AWS account.
    2. You define what types of EC2 instances `dstack` it's allowed to use, spot or on-demand, and what maximum number 
    of each instance type is allowed to run at one time.
    3. When you submit a workflow, `dstack` creates required EC2 instances to run your workflow automatically.
    4. When your workflow is finished and there is no need in the EC2 instance, `dstack` tears it down.

## Configure your AWS account

Before you'll be able to use on-demand runners via your cloud, you have to provide `dstack` the credentials
to your AWS account. This can be done by the `dstack aws configure` command:

```bash
dstack aws configure
AWS Access Key ID:  
AWS Secret Access Key: 
Region name:
Artifact S3 bucket[None]: 
```

Note, `Artifact S3 bucket` is optional and has to be specified only if you want to use your own S3 bucket to store 
artifacts.

!!! note "Required IAM permissions"
    The `dstack autoscale` feature requires the following permissions:

    ```
    ec2:Describe*
    ec2:RequestSpotInstances
    ec2:TerminateInstances
    ec2:CancelSpotInstanceRequests
    ec2:CreateSecurityGroups
    ec2:AuthorizeSecurityGroupIngress
    ec2:AuthorizeSecurityGroupEgress
    ```

## Manage allowed instance types

With `dstack`, it's possible to configure what instance types it's allowed to use, spot or on-demand, 
and what maximum number of each instance type is allowed to run at one time.

### Add or update allowed instance type

Type the following to see how to add or update allowed instance types:

```bash
dstack autoscale allow --help
usage: dstack autoscale allow [-h] --max MAX [--spot] [--on-demand] INSTANCE_TYPE

positional arguments:
  INSTANCE_TYPE

optional arguments:
  -h, --help         show this help message and exit
  --max MAX, -m MAX  The maximum number of instances
  --spot, -s         Spot instances
  --on-demand, -d    On-demand instances
```

The required positional argument `INSTANCE_TYPE` can be any of the [instance types](https://aws.amazon.com/ec2/instance-types/)
supported by AWS in the region that you've configured.

The required `--max MAX` argument can be any integer number. It specifies the maximum number of spot instances
allowed to create per the given instance type.

Note, you always have to specify `--spot` or `--on-demand to indicate which purchase type is to use for the 
corresponding instance type. 

!!! example "Example"
    Here's a command that allows `dstack` to run in parallel up to one spot instance with the type `m5.xlarge`.
    
    ```bash
    dstack autoscale allow m5.xlarge --max 1 --spot
    ```

    
If you try to add an instance type that is not supported by your AWS account, you'll see an error.

### Remove allowed instance types

To remove an allowed instance type, use `dstack autoscale remove`. Here's how this command works:

```bash
dstack autoscale remove --help
usage: dstack autoscale remove [-h] [--all] [INSTANCE_TYPE]

positional arguments:
  INSTANCE_TYPE

optional arguments:
  -h, --help     show this help message and exit
  --all, -a      Disallow all instances
```

!!! info "" 
    As soon as you decrease the number of allowed instances, `dstack` immediately shuts down extra instances to match
    the maximum number of allowed instances.

### List allowed instance types

You can always see the list of all rules by the following command:

```bash
dstack autoscale info
```

## Disable and enable on-demand runners

You can disable and enable on-demand runnerss with a single command:

```bash
dstack autoscale disable
```

or 

```bash
dstack autoscale enable
```

!!! warning ""
    If you disable on-demand runners, `dstack` will immediately shut down all running instances.