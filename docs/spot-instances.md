# Using AWS spot instances

Spot instances is a special type of AWS EC2 instances that are offered at a cheaper price but may terminate at any time.
With `dstack`, it's very easy to run ML workflows on AWS spot instances.

!!! note ""
    In order use `dstack` with AWS spot instances, obviously you'll need an AWS account. If you don't have one, you
    can sign up for it [here](https://www.google.com/aclk?sa=L&ai=DChcSEwiC2LWV7-zyAhXZFXsKHVd7DyEYABABGgJsZQ&ae=2&sig=AOD64_2EAs7JWkaTdb4h1oTYVZX-VJttJQ&q&adurl&ved=2ahUKEwjoj6yV7-zyAhVDR_EDHUZPATAQ0Qx6BAgCEAE).

## Step 1: Install CLI

In order to be able to set up `dstack` runners on AWS spot instances, you have to install the `dstack` CLI locally 
and obtain your `Personal Access Token`.

!!! tip ""
    If you've already obtained your `Personal Access Token`, feel free to skip to [Step 2](#step-2-create-a-launch-template).

The `dstack` CLI can be installed via `pip`:

```bash
pip install dstack -U
```

!!! note ""
    The same command can be used to update the `dstack` CLI to the latest version.

### Register a user

To obtain your `Personal Access Token`, first, you have to register a user with `dstack.ai`:

```bash
dstack register
```

This command will prompt you to select a user name (only latin characters, digits and underscores are allowed), 
and specify your email. Tt will send you a verification code, and you'll have to confirm it right away.

### Login as existing user

If you'll ever have to authorize the `dstack` CLI again on this or another machine, you'll can do that by using
the following command:

```bash 
dstack login
```

This command needs the user name and the password. If it's correct, it authorizes the current machine to use
the `dstack` CLI.

### Obtain a token

Once you've registered, you can obtain your `Personal Access Token`:

```bash
dstack token
```

You'll need this token at the next step.

## Step 2: Create a Launch Template

Now that you have your `Personal Access Token`, you can set up a `Launch Template`.

Sign in to your AWS Console, go to `EC2`, then `Launch Templates`, and then click `Create launch template`.

Here's what you have to specify:
* A name of the template
* Amazon machine image (AMI), e.g. `Deep Learning AMI (Amazon Linux 2)`
* Instance type (with or without GPU)
* A key pair (optional; required only if you'd like to access your instance via SSH)
* Security group

!!! warning ""
    It's important that, in the rules of the selected `Security groups`, you allow `All TCP` outbound traffic. 

Lastly, in `Advanced details`, specify the following `User data`:

```bash
#!/bin/bash
curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads-stgn.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-linux-amd64"
chmod +x /usr/local/bin/dstack-runner
HOME=/root dstack-runner config --token <token>
HOME=/root nohup dstack-runner start &
```

Click `Create launch template`

## Step 3: Request spot instances 

Within the `EC2` page, go to `Spot Requests`, and then click `Request Spot instances`.

Here, you'll have to&hellip;

* Select your `Launch Template` (and its version)
* Specify the total target capacity (the number of instances)
* Check the hourly price

When done, click `Launch`.

Within short time, AWS will set up the requested amount of instances. You don't have to worry of anything else. 
AWS will set up `dstack-runner` on each instance.

## Step 4: Check runners

Once AWS instances are up, you can use the `dstack` CLI to check the status of runners on these instances:

```bash
dstack runners 
```

If the instances are configured properly, you'll see the hosts of each instance in the output. For example:

```bash
RUNNER    HOST                    STATUS    UPDATED
sugar-1   MBP-de-Boris.fritz.box  LIVE      3 mins ago
```

!!! warning "Runners are not there?"
    If you don't see your host in the output, this means the runner is offline or that the `dstack-runner` daemon
    was not configured or launched properly.

!!! bug "Submit feedback"
    Something didn't work or was unclear? Miss a critical feature? Please, [let me know](https://forms.gle/nhigiDm4FmjZdRkx5). I'll look into it ASAP.