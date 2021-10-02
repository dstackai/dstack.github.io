# Installation

Before you can set up runners or run workflows, you need to install the `dstack` CLI and obtain a `Personal Access
Token`. 

## Install CLI

The `dstack` CLI can be installed via `pip`:

```bash
pip install dstack -U
```

## Register a user

To obtain a `Personal Access Token`, you have to register a user with `dstack.ai`. 

This can be done via a CLI command:

```bash
dstack register
```

It will prompt you to select a username (only latin characters, digits and underscores are allowed), and specify your
email. To verify the email address, it will send you a verification code that you'll have to confirm.

The `Personal Access Token` that is associated with your user is now stored in the `~/.dstack/config.yaml` file.
Now, all your commands of the `dstack` CLI will be authorized and associated with your user.

Later, when you'd like to set up runners on other machines, you'll need to use this `Personal Access Token`.

!!! note "Login as existing user"
    If your `Personal Access Token` is not stored in the `~/.dstack/config.yaml` file (e.g. it's another 
    machine), you can restore it by the following command:

    ```bash 
    dstack login
    ```
    
    This command will ask your username and password. If credentials are correct, it will update the
    `~/.dstack/config.yaml` file.

## Get a token

To see your `Personal Access Token`, you can either open the `~/.dstack/config.yaml` file, or call the following command:

```bash
dstack token
```

!!! danger "Keep the token secure"
    It's important that you keep this token secure and don't share it with others. If the token is compromised, you'll 
    have to change it.

!!! bug "Submit feedback"
    Something doesn't work or is not clear? Would like to suggest a feature? Please, [let us know](https://forms.gle/nhigiDm4FmjZdRkx5).