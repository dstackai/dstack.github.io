# Installation

## Install CLI

The `dstack` CLI can be installed via `pip`:

```bash
pip install dstack -U
```

## Login

Before you use the CLI, you need to authorize with your user credentials. This can be done via a CLI command:

```bash
dstack login
```

The command will prompt you to enter your username and your password. 

If the credentials are correct, the `Personal Access Token` that is associated with your user will be stored in the 
`~/.dstack/config.yaml` file. Now, all your commands of the `dstack` CLI will be authorized and associated with your user.

!!! info "How do I register a user with dstack.ai?"
    Please sign up using the form at [dstack.ai](https://dstack.ai) and wait for an approval. 
    As soon as your request is approved, you'll receive your user credentials to your email.

## Get a token

To see your `Personal Access Token`, you can either open the `~/.dstack/config.yaml` file, or call the following command:

```bash
dstack token
```

!!! danger "Keep the token secure"
    It's important that you keep this token secure and don't share it with others. If the token is compromised, you'll 
    have to change it.

## Artifacts S3 bucket

By default, `dstack` stores output artifacts in its own secure storage that only
your user has access to. If you want to store output artifacts in your own S3 bucket, [configure your AWS account](aws.md) 
and specify the name of the S3 bucket.