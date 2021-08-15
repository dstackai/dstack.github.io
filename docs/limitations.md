# Limitations

There is a long list of the things that `dstack` doesn't support yet:

* **Private repositories**: currently, `dstack` only works with public Git repositories. Private Git repositories
  cannot be used with `dstack` yet.
* **Dashboard interface**: currently, you can work with `dstack` only via the `dstack`. There is not yet a user interface
  to sign up, and manage runners, runs, artifacts, and logs.
* **Hardware metrics**: currently, `dstack` doesn't provide the hardware metrics for the runners, e.g. the use and
  availability of memory, CPU, GPU, etc.
* **Hardware requirements**: currently, there is no way to specify hardware requirements per workflow (e.g. GPU, CPU,
  memory, etc.)
* **Own storage**: currently, `dstack` uses its own storage for artifacts and logs. There is not yet
  a way to use own storage.
* **Own server**: currently, the `dstack` server is hosted with `dstack.ai` and stores the information about
  the user, runs, and runners on its own server. There is not yet a way to have a `dstack` server up on your own
  machine.
* **Triggers**: currently, there is no way to configure your own events or webhooks to trigger new runs. All runs must
  be manually started via the `dstack` CLI.
* **Sweeps**: currently, there is no way to run a multiple combinations of variables in parallel (i.e. hyperparameter
  tuning, e.g. grid search, bayesian search, etc.)