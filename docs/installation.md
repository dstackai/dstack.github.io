# Installation

!!! info "Request access"

    Before you can install the `dstack` CLI, you have to register a `dstack` user and recieve your 
    `Personal Access Token`.
    To do that, please click `Request access` at [dstack.ai](https://dstack.ai). 
    Once your request is approved, you'll recieve an email with the instruction on how to register a user and
    obtain your `Personal Access Token`.

### Install CLI

The `dstack` CLI can be installed via `pip`:

```bash
pip install dstack -U
```

### Configure a token

Once you've installed the CLI, you have to configure it with your `Personal Access Token`. This can be done via a CLI command:

```bash
dstack config --token <token>
```

Your `Personal Access Token` will be stored in the `~/.dstack/config.yaml` file. 
After that, all your commands of CLI will be associated with your user.