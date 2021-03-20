---
layout: post
title: ML platforms evaluation summary
comments: true
---

Below are my general notes from the recent evaluation of various ML
tools/platforms. At the end I try to summarize the features of an ideal platform (at least
from my humble POV :)).

### Which things I did like?

* KFserving from kubeflow for serving
  * uses knative + istio to provide a feature-full environment for
    microservices, which in this case delivers ML model inference
  * using those tools allows to use advanced techniques like canary deployments
* Concept of a workflow from FB Learner: the complete pipeline stays the same
  and people can just plug-in different features
* MLFlow has a good idea on tracking the experiments on the sdk/framework level
  (e.g. there are defaults for pytorch etc) and in general organizing research
workflow
* MLFlow has a really nice concept of a universal model format. On one side it
  ensures that the right things get stored in Model Registry, on the other it
is a nice, common definition which e.g. can be used for deployment/model
serving.
  * It this could be combined with KFserving, this would be a ML platform
    superpower.

### What did I learn?

* Data infra (datalake/data warehouse) and elastic compute are a foundation for
  an ML platform
* Tight integration with the datalake/warehouse (FB Learner/Michelangelo) is a
  very good idea.
  * Enables using available data sources as features (so the datalake/warehouse
    == feature store)
* The pipelines/workflows should have data-quality checks built-in to allow for
  quick catching of problems with data
* Using DAGs and python as a workflow definition (FB Learner) seems to have the
  best abstraction level, e.g. Airflow DAGs seem like a better abstraction than
a series of interconnected container executions (although seem a bit limited in
the sense that you need to follow specific rules when you integrate)
* kubeflow, michelangelo, fblearner support automated hyperparameter search
  functionality
* Model evaluation reports should be centrally stored and be easy to query
  (e.g. indexed with elasticsearch)
* Model should have a report from training associated
* Support for both online/offline training is a nice feature (although this
  would be more a requirements based decision)
* MLFlow seems to be a good step in making research reproducible, or in general
  just more organized, it needs some improvements though to make it digestable
for wider audience:
  * configuration patterns should be easier (e.g. the environment - Jupyter
    Notebooks, injects the default tracking configuration)
  * some defaults on metrics to track
  * artifact storage for the tracing server setup should be solved a bit better
    (the client should only see the server and not the storage accounts in
AWS/Azure)

### What would be the features of an ideal ML platform

* A solid foundation with a datalake/data warehouse with well organized
  metadata
  * DL/DWH become in general large feature stores - we do not have to have an
    explicit feature store this way
* Pipelines are written as Python code in something similar to Airflow
* There is a range of ready to use ML pipelines (pre-processing piepeline +
  model definition + training procedure + easy to change feature configuration)
* Provides easily accessible Jupyter notebooks which have good integration
  with:
  * Scalable Spark clusters
  * Metadata store from datalake/DWH
  * Default MLFlow setup for remote tracing and artifact and model storage out
    of the box
* The underlying infrastructure is an autoscalable, multitenant Spark cluster
  * This can be achieved with Spark-on-Kubernetes, where k8s effectively
    becomes the queue manager for Spark jobs.
* Universal model format, which can be used by KFserving to expose the model
  through an API or by an automated pipeline to produce batch predictions.
* Support for online and batch training
* Support for automated hyperparameter tuning
* Data Quality checks built into the ML pipelines (something similar to
  `great_expectations`)
* MLFlow Model Registry integrated and extended with easy browsing and
  searching functionality, integrated also with KFserving and batch prediction
calculation pipelines.

This would enable the following:
* A Data Scientist can easily iterate over his model with use of ML Flow, his
  work is nicely tracked and models are ready to deploy in Model Registry.
MLFlow integration is done without any additions from his side.
* ML Engineer can easily deploy those models to production with KFserving or
  batch pipelines.
* If model needs to be periodically retrained, an Airflow DAG can be written to
  create a pipeline
* Both Data Scientist and ML Engineer have full access to all DWH/DL data, so
  their models/pipelines can  be easily adjusted if new data begins to be
captured.
* Quality is ensured on every stage with great_expectations (and its extension
  to cover e.g. accuracy of the production models).
* All information about past workflows and models can be easily searched
  through.
