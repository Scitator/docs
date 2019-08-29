---
title: Databricks
sidebar_label: Databricks
---

W&B integrates with [databricks](https://www.databricks.com) by customizing the W&B jupyter notebook experience in the databricks environment.

## Databricks Configuration

### Install wandb on the cluster

Navigate to your cluster configuration, choose your cluster, click on Libraries, then on Install New, Choose PyPI and add the package `wandb`.

> Use this wandb development branch from PyPI for improved compatibility: `git+git://github.com/wandb/client.git@feature/sweep-jupyter#egg=wandb`

### Authentication

In order to authenticate your W&B account you can add a databricks secret which your notebooks can query.
```shell
# install databricks cli
pip install databricks-cli

# Generate a token from databricks UI
databricks configure --token

# Create a scope with one of the two commands (depending if you have security features enabled on databrick):
# with security add-on
databricks secrets create-scope --scope wandb
# without security add-on
databricks secrets create-scope --scope wandb --initial-manage-principal users

# Add your api_key from: https://app.wandb.ai/authorize
databricks secrets put --scope wandb --key api_key
```

## Examples

### Simple

```python
import os
import wandb

api_key = dbutils.secrets.get("wandb", "api_key")
os.environ['WANDB_API_KEY'] = api_key

wandb.init()
wandb.log({"foo": 1})
```

### Sweeps

Setup required (temporary) for notebooks attempting to use wandb.sweep() or wandb.agent():

```python
import os
api_key = dbutils.secrets.get("wandb", "api_key")
os.environ['WANDB_API_KEY'] = api_key
os.environ['WANDB_ENTITY'] = "my-entity"
os.environ['WANDB_PROJECT'] = "my-project-that-exists"
```

Follow examples:
- [Initialize a sweep](/sweeps/python#initialize-a-sweep)
- [Ran an agent](/sweeps/python#run-an-agent)

