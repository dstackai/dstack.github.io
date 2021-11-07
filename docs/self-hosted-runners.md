# Use self-hosted runners

A runner is a machine that can run `dstack` workflows. You can host runners on your local machine or
on your remote machines.

In order to host a runner on any machine, you have to launch the `dstack-runner` daemon there. 
The machines that host the `dstack-runner` daemon form a pool of runners, and when the user runs workflows via the 
`dstack` CLI, the workflows will be running on these machines.

!!! tip "Run locally"
    If you don't want to use remote machines, you can host a runner locally.
    All you need to do is to launch the `dstack-runner` daemon locally.

## Install the daemon

Here's how to install the `dstack-runner` daemon:

=== "Linux"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-linux-amd64"
    sudo chmod +x /usr/local/bin/dstack-runner
    ```

=== "macOS"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-darwin-amd64"
    sudo chmod +x /usr/local/bin/dstack-runner
    ```

If you are on **Windows**, simply download [dstack-runner.exe](https://dstack-runner-downloads.s3.eu-west-1.amazonaws.com/latest/binaries/dstack-runner-windows-amd64.exe).

## Configure a token

Before you can start the daemon, you have to configure it with your `Personal Access Token`:

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

!!! tip "Personal Access Token"
    Use the `dstack token` command of the CLI to get your `Personal Access Token`. See [Installation](installation.md#get-a-token)&hellip;    

Once you do it, the daemon is ready to start:

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

!!! danger "Docker is required"
    The `dstack-runner` daemon requires that either the standard Docker or the NVIDIA's Docker is installed and 
    running on the machine.

!!! warning "Internet is required"
    The machine where you run the `dstack-runner` daemon has to have a connection to the Internet. 

    If your machine is an EC2 instance, make sure its security group allows outgoing traffic. 

## Check runners' status

After you've set up runners, you can check their status via the `dstack` CLI:

```bash
dstack runners 
```

If runners are running properly, you'll see their hosts in the output:

```bash
RUNNER    HOST                    STATUS    UPDATED
sugar-1   MBP-de-Boris.fritz.box  LIVE      3 mins ago
```

!!! warning "Runner is not there?"
    Don't see your runner? This may mean the runner is offline or that the `dstack-runner` daemon
    was not configured or started properly.