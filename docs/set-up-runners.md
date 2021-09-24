# Set up runners

## Install runner

A runner is any machine which you'd like to use to run `dstack` workflows. In order to use any machine
as a runner, you have to launch the `dstack-runner` daemon on that machine.

All machines that have the `dstack-runner` daemon running form a pool of runners. 

When you later run workflows via the `dstack` CLI, these workflows are assigned to these runners.

!!! info "Run runners locally"
    If you don't plan to use remote machines, you can launch the `dstack-runner` daemon locally.

Here's how you can install the `dstack-runner` daemon:

=== "Linux"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads.s3.eu-west-1.amazonaws.com/0.0.1rc5/binaries/dstack-runner-linux-amd64"
    sudo chmod +x /usr/local/bin/dstack-runner
    ```

=== "macOS"

    ```bash
    sudo curl --output /usr/local/bin/dstack-runner "https://dstack-runner-downloads.s3.eu-west-1.amazonaws.com/0.0.1rc5/binaries/dstack-runner-darwin-amd64"
    sudo chmod +x /usr/local/bin/dstack-runner
    ```

If you are using **Windows**, download [dstack-runner.exe](https://dstack-runner-downloads.s3.eu-west-1.amazonaws.com/0.0.1rc5/binaries/dstack-runner-windows-amd64.exe), and run it.

## Configure a token

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

## Check runners

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

!!! bug "Submit feedback"
    Something didn't work or was unclear? Miss a critical feature? Please, [let me know](https://forms.gle/nhigiDm4FmjZdRkx5). I'll look into it ASAP.