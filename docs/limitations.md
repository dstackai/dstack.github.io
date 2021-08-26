# Limitations

The `dstack` tool is at the early preview stage. There are lots of things it doesn't support yet:

* **Private repositories**: Currently, `dstack` only supports public Git repositories. Private Git repositories
  cannot be used with `dstack` yet.
* **Dashboard interface**: Currently, you can work with `dstack` only via the CLI. There is not yet a user interface
  to sign up, and manage runners, runs, artifacts, and logs.
* **Hardware metrics**: Currently, `dstack` doesn't report hardware metrics for the runners, e.g. the use and
  availability of memory, CPU, GPU, etc.
* **Hardware requirements**: Currently, there is no way to specify hardware requirements per workflow (e.g. GPU, CPU,
  memory, etc.)
* **Own storage**: Currently, `dstack` uses its own storage for artifacts and logs. There is not yet
  a way to use own storage.
* **Own server**: Currently, the `dstack` server is hosted with `dstack.ai` and stores the information about
  the user, runs, and runners on its own server. There is not yet a way to have a `dstack` server up on your own
  machine.
* **Triggers**: Currently, there is no way to configure your own events or webhooks to trigger new runs. All runs must
  be manually started via the `dstack` CLI.
* **Sweeps**: Currently, there is no way to run a multiple combinations of variables in parallel (i.e. hyperparameter
  tuning, e.g. grid search, bayesian search, etc.)

!!! bug "Submit feedback" 
    Something critical is not on the list? Something didn't work or was unclear? Please, [let me know](https://forms.gle/nhigiDm4FmjZdRkx5). I'll look into it ASAP.