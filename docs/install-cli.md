# Install CLI

## Install package

To run workflows or obtain a token to set up runners, you'll need the `dstack` CLI. It can be installed via `pip`:

```bash
pip install dstack -U
```

## Register a user

To set up runners, you'll need to obtain your `Personal Access Token`. To do that, you have to register a user 
with `dstack.ai`. This can be done via the CLI:

```bash
dstack register
```

It will prompt you to select a user name (only latin characters, digits and underscores are allowed), and specify your
email. To verify the email address, it will send you a verification code that you'll have to confirm.

Now, you are fully authorized on this machine to perform any operations via the CLI. The obtained 
`Personal Access Token` can be later used to set up runners.

!!! note ""
    The authorization is done via the `Personal Access Token` that is stored in `~/.dstack/config.yaml` file.

!!! note "Login as existing user"
    If you'll have to authorize the `dstack` CLI again on this or another machine, you'll can do that by using
    the following command:

    ```bash 
    dstack login
    ```
    
    This command needs the user name and the password. If it's correct, it authorizes the current machine to use 
    the `dstack` CLI.

## Obtain a token

To set up runners, you'll need your `Personal Access Token`.
This token can be obtained by the following command:

```bash
dstack token
```

You'll need this token to configure the `dstack-runner` daemon at the next step.

!!! bug "Submit feedback"
    Something didn't work or was unclear? Miss a critical feature? Please, [let me know](https://forms.gle/nhigiDm4FmjZdRkx5). I'll look into it ASAP.