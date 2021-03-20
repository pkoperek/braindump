---
layout: post
title: Uber's Michelangelo notes
---

* Michelangelo is built on top open source tools + integrations with the Uber's compute and data infra
* **Unified infrastructure for ML-projects**
* Abstracts one particular version of a ML workflow
* Both Cassandra (for streaming) and Hive (for batch) are mentioned as feature stores - this suggests that "regular" data warehouse/datalake assets are/can be used as features
* Uber supports a concept of a "partitioned" model (basically the same model appplied to a different subset of data e.g. at a city, country level)
* Architecture includes explicit feature and model stores
  * feature store seems like a subset of datalake/warehouse with a special purpose
  * model store is more tailored - this is a db with records of all models trained in Michelangelo (see excerpts for a list)
* Supports offline training, models can be deployed to make batch or online predictions
* After training commences, a set of metrics is computed and assembled into a report; all of that is stored in the model repository
  * popular/important model types have special visualizations implemented
* There is a feature of automatically searching for hyperparameters
* Michelangelo also creates a "feature report" which explains influence of features on the model
* Models can be deployed as:
  * Offline job (spark job) which generates batch predictions (the predictions are stored back to Hive - from there they are copied to target systems)
  * Online prediction service (there is a prediction service cluster - set of machines behind an Load Balancer) - client can send RPC calls to get a prediction about a sample/batch of samples
  * Library deployment - model is embedded into a library and can be invoked as Java API
* Training is triggered through API - people can start it manually or write scripts which schedule it automatically over time (_this seems like not the best idea_)
* Models deployed to prod are uniquely identified when serving predictions (_seems like the online use-case_), client can choose which model it wants to use (this nicely integrates with the client experimentation api)
* Online predictions can be done in P95 5ms (without accessing Cassandra) or P95 10ms (with accessing Cassandra)
* The whole system seems to have an API (you can control everything through an API)
  * _this makes it very similar to kubeflow_
* While a model is being served, Michelangelo can log and store a percentage of predictions and calculate metrics on top of them and use this information later to measure/track model's accuracy over time (numbers go to an internal tracking tool which allows to e.g. configure alerts).

#### Key excerpts

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
```

```
Michelangelo consists of a mix of open source systems and components built in-house. The primary open sourced components used are HDFS, Spark, Samza, Cassandra, MLLib, XGBoost, and TensorFlow. We generally prefer to use mature open source options where possible, and will fork, customize, and contribute back as needed, though we sometimes build systems ourselves when open source solutions are not ideal for our use case.
```

```
Michelangelo is built on top of Uber’s data and compute infrastructure, providing a data lake that stores all of Uber’s transactional and logged data, Kafka brokers that aggregate logged messages from all Uber’s services, a Samza streaming compute engine, managed Cassandra clusters, and Uber’s in-house service provisioning and deployment tools.
```

```
The same general workflow exists across almost all machine learning use cases at Uber regardless of the challenge at hand, including classification and regression, as well as time series forecasting. The workflow is generally implementation-agnostic, so easily expanded to support new algorithm types and frameworks, such as newer deep learning frameworks. It also applies across different deployment modes such as both online and offline (and in-car and in-phone) prediction use cases.
```

```
We designed Michelangelo specifically to provide scalable, reliable, reproducible, easy-to-use, and automated tools to address the following six-step workflow:

1. Manage data
2. Train models
3. Evaluate models
4. Deploy models
5. Make predictions
6. Monitor predictions
```

```
A platform should provide standard tools for building data pipelines to generate feature and label data sets for training (and re-training) and feature-only data sets for predicting. These tools should have deep integration with the company’s data lake or warehouses and with the company’s online data serving systems.
```

Michelangelo seems to be using a similar model to organize workflow execution as kubeflow:

```
Training jobs can be configured and managed through a web UI or an API, often via Jupyter notebook. Many teams use the API and workflow tools to schedule regular re-training of their models.
```

```
When a model is trained and evaluated, historical data is always used. To make sure that a model is working well into the future, it is critical to monitor its predictions so as to ensure that the data pipelines are continuing to send accurate data and that production environment has not changed such that the model is no longer accurate.

To address this, Michelangelo can automatically log and optionally hold back a percentage of the predictions that it makes and then later join those predictions to the observed outcomes (or labels) generated by the data pipeline. With this information, we can generate ongoing, live measurements of model accuracy. In the case of a regression model, we publish R-squared/coefficient of determination, root mean square logarithmic error (RMSLE), root mean square error (RMSE), and mean absolute error metrics to Uber’s time series monitoring systems so that users can analyze charts over time and set threshold alerts, as depicted below:
```

#### Links

* [Source](https://eng.uber.com/michelangelo-machine-learning-platform/)
