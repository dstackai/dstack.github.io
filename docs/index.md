# Overview

dstack is a workflow as code tool for AI researchers 👩🏽‍🔬 to define their workflows and the infrastructure they need declaratively.
Once defined, these workflows can be run interactively 🧪 in a reproducible manner 🧬. 
Infrastructure is provisioned on-demand and torn down when it's not needed 💨.
You're free to use any frameworks, vendors, etc.

## Why dstack?

As an AI researcher 👩🏽‍🔬, you always want to focus on experiments 🧪 and their metrics 📈. 

Training production-ready models 👷🏽‍ however requires high reproducibility 🧬 and involves many things that may 
distract you from your work.

### 📦 Versioning data

The data produced by your workflows should be saved automatically in an immutable storage 💿.

You should be able to chain workflows together ⛓ so one workflow may use the outputs of another workflow at any time.

### 🤖 Infrastructure as code

To ensure your workflows can be reproduced, you should be able to describe the 
infrastructure your workflows need declaratively as code 📝.

When you run your workflows, the infrastructure should be provisioned on-demand 🙏🏽 and torn down once it's not needed  
any more automatically 💨.

You should be able to use any type and vendor of the infrastructure.

### 🧪 Interactivity

While ensuring reproducibility, it should be possible to run workflows interactively from your IDE or Terminal.

### 🧩 Extensibility

You should be allowed to use any languages, libraries, frameworks, experiment trackers, or cloud vendors.

It should be possible to use multiple workflow providers, either created by yourself, or by the community 🌍.

!!! success ""
    With dstack, you get all of it in a simple and easy-to-use to use form 🙌.

## Features

1. 🧬 **Workflows**

     * Define your workflows and infrastructure they need, using declarative configuration files
     * Use multiple workflow providers shared by the community or create your own providers

2. 📦 **Artifacts**

     * Define what output files produced by a workflow should be saved as artifacts
     * Have output data by workflows saved to an immutable storage in real-time as artifacts
     * Define dependencies between workflows to chain them together and pass artifacts of one workflow as inputs to another one
     * Tag successful runs with a name to reuse their artifacts later and share with others

3. 🤖 **Runners**

     * Provide dstack with credentials to provision infrastructure on-demand in your own cloud accounts (such
       as AWS, GCP, Azure, etc.)
     * Manage what compute instances it's allowed to use, in what regions, and at what quantity  
     * Add your own servers to the pool of available infrastructure that can be used by dstack
     
1. 🧬 **CLI**

     * Run workflows interactively from your IDE or Terminal