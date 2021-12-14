# Use spot instances in own cloud

A runner is a machine that can run `dstack` workflows. With `dstack` you can run workflows using spot instances 
in your cloud. `dstack` will set up and tear down spot instances automatically based on the need.


!!! warning "Cloud vendors"
    This tutorial describes how to use the `dstack autoscale` feature with an AWS account. 
    If you use another cloud vendor, such as GCP, Azure (or some other), please write to 
    [hello@dstack.ai](mailto:hello@dstack.ai).

!!! info "How autoscaling works"

    1. You provide `dstack` credentials to create spot instances in your AWS account.
    2. You define what types of EC2 instances `dstack` is allowed to use, and what maximum number per instance type.
    3. When you run workflows, `dstack` creates spot instances and automatically configures there `dstack-runner` 
    daemon that picks up assigned workflows to run.
    4. When workflows are finished, `dstack` tears down unecessary spot instances.

## Configure your AWS account

Before you'll be able to use spot instances as runners in your cloud, you have to provide `dstack.ai` the credentials
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

## Manage autoscale rules

Once you've configured the credentials to your AWS account, you'll be able to manage autoscale rules.

### Add or replace rules

Type the following to see how to add or replace autoscale rules:

```bash
dstack autoscale allow --help
usage: dstack autoscale allow --max MAX INSTANCE_TYPE

positional arguments:
  INSTANCE_TYPE

optional arguments:
  --max MAX      The maximum number of instances
```

The required positional argument `INSTANCE_TYPE` can be any of the [instance types](https://aws.amazon.com/ec2/instance-types/)
supported by AWS for spot instances.

The required `--max MAX` argument can be any integer number. It specifies the maximum number of spot instances
allowed to create per the given instance type.

Let's look at an example:

```bash
dstack autoscale allow m5.xlarge --max 1
```

This command instructs `dstack` that it can create at max one instance of the type `m5.xlarge`.

If you try to add a rule that is not supported by your AWS account, you'll get an error.

### Remove rules

To remove a rule for a given instance type is the same as setting the corresponding maximum number of instances to 0:

```bash
dstack autoscale allow m5.xlarge --max 0
```

!!! info "" 
    As soon as you decrease the number of allowed instances, `dstack` immediately shuts down extra spot instances to match
    the maximum number of allowed instances.

### List rules

You can always see the list of all rules by the following command:

```bash
dstack autoscale info
```

## Clear autoscale rules

To quickly delete all defined rules, with the following command:

```bash
dstack autoscale clear
```

!!! info ""
    This command will immediately shut down all running spot instances if there are any.

## Disable and enable autoscaling

You can disable and enable autoscaling with a single command:

```bash
dstack autoscale disable
```

or 

```bash
dstack autoscale enable
```

These commands allow you to pause temporarily autoscaling without deleting the rules.

!!! warning ""
    The `disable` command will immediately shut down all running spot instances if there are any.