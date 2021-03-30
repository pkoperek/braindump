---
layout: post
title: Model stores in ML platforms
---

# What is a model store/registry?

Component of a ML platform, which is responsible for storage of models and related metadata. The related metadata can be anything from performance metrics, through source code to related datasets. The exact content depends on the use-cases implemented by the model store. Introducing a model store has the following advantages:
* it makes the model development process more structured and systematic: it is possible to easily compare different versions of the same model (e.g. with different training parameters)
* the deployment of the model to production can be simplified: one can prepare a common infrastructure which would have well defined requirements towards model format, those requirements could be fulfilled and enforced centrally in the model store
* it makes it possible to browse through existing models what should encourage knowledge sharing

## Examples of model stores

### Michelangelo

Uber's Michelangelo platform introduced an explicit model store component:

```
For every model that is trained in Michelangelo, we store a versioned object in our model repository in Cassandra that contains a record of:

* Who trained the model
* Start and end time of the training job
* Full model configuration (features used, hyper-parameter values, etc.)
* Reference to training and test data sets
* Distribution and relative importance of each feature
* Model accuracy metrics
* Standard charts and graphs for each model type (e.g. ROC curve, PR curve, and confusion matrix for a binary classifier)
* Full learned parameters of the model
* Summary statistics for model visualization
The information is easily available to the user through a web UI and programmatically through an API, both for inspecting the details of an individual model and for comparing one or more models with each other.
```

[source](https://eng.uber.com/michelangelo-machine-learning-platform/)

### MLFlow

* The Model Registry is one of the central concepts in MLFlow
* Inserting into MLFlow is executed when you include MLFlow sdk's code in model code
* MLFlow introduces its own model format. After you are done with training (in any supported framework) you can simply save (or log - if you are using model registry) the model to storage in that format. Using it enables MLFlow to e.g. easily wrap it with a REST API service (so you can easily deploy it somewhere).
* MLFlow provides infra for easy model serving (although comparing to KFServing looks a bit like a toy)

[source](https://www.mlflow.org/docs/latest/model-registry.html)

### Kubeflow

In kubeflow there is no explicit model registry, although its features seem
to be implemented to some degree in KFServing. KFServing introduces a
description of model deployment (a Custom Resource in k8s) which ties together:

* framework which is used for inference (e.g. pytorch) - this is done through
  different resource specs
* parameters (e.g. weights of the model) - just a resource like file blob in s3
* resource requirements (CPU, mem)
* serving configuration (e.g. the k8s service spec wrapped in a relevant
  kubeflow resource)

This introduces a common description of models, which is kind-of-a model
registry.

### FB Learner flow

* An explicit model registry seems not to be present
* ... however its role is implemented by two other components:
  * the repository of reusable training pipelines (they have a notion of a model embedded, that model just needs to be trained)
  * the experiment/run repository (which allows to compare runs with each other)

## Related concept - version control for ML models

If an environment doesn't provide an explicit model store, one could be  implemented using version control tools, e.g.

* [keepsake](https://github.com/replicate/keepsake)
  * versioning for ML artifacts (code, hyperparameters, weights etc)
  * a little bit similar to MLFLow: installed through embedding in code
* [dvc](https://dvc.org/)
  * "git" for ML
  * similar to keepsake
