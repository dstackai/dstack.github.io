# Installation

## Install CLI

The `dstack` CLI can be installed via `pip`:

```bash
pip install dstack -U
```

## Configure a token

Before you can use the CLI, you have to configure it with your `Personal Access Token`. This can be done via a CLI command:

```bash
dstack config --token <token>
```

The provided `Personal Access Token` will be stored in the `~/.dstack/config.yaml` file. 
After that, all your commands of CLI will be associated with your user.

!!! info "Personal Access Token"
    In order to receive your `Personal Access Token`, please click `Request access` at [dstack.ai](https://dstack.ai). 
    Once your request is approved, you'll be able to create a `dstack` user, and obtain your token.

## Artifacts S3 bucket

By default, `dstack` stores output artifacts in its own secure storage that only
your user has access to. If you want to store output artifacts in your own S3 bucket, [configure your AWS account](aws.md) 
and specify the name of the S3 bucket.