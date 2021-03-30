---
layout: post
title: Feature stores from Data Engineering perspective
---

# What is a feature store?

```
Introduction to feature stores
Feature stores are systems that help to address some of the key challenges that ML teams face when productionizing features

* Feature sharing and reuse: Engineering features is one of the most time consuming activities in building an end-to-end ML system, yet many teams continue to develop features in silos. This leads to a high amount of re-development and duplication of work across teams and projects.

* Serving features at scale: Models need data that can come from a variety of sources, including event streams, data lakes, warehouses, or notebooks. ML teams need to be able to store and serve all these data sources to their models in a performant and reliable way. The challenge is scalably producing massive datasets of features for model training, and providing access to real-time feature data at low latency and high throughput in serving.

* Consistency between training and serving: The separation between data scientists and engineering teams often lead to the re-development of feature transformations when moving from training to online serving. Inconsistencies that arise due to discrepancies between training and serving implementations frequently leads to a drop in model performance in production.

* Point-in-time correctness: General purpose data systems are not built with ML use cases in mind and by extension don’t provide point-in-time correct lookups of feature data. Without a point-in-time correct view of data, models are trained on datasets that are not representative of what is found in production, leading to a drop in accuracy.

* Data quality and validation: Features are business critical inputs to ML systems. Teams need to be confident in the quality of data that is served in production and need to be able to react when there is any drift in the underlying data.
```
[Source](https://www.kubeflow.org/docs/components/feature-store/overview/#introduction-to-feature-stores)

In my opinion most of those reasons boil down to three key things:

* features data needs to be stored in a well specified, consistent format
* features data quality needs to stay high
* features metadata needs to make it clear and easy to find relevant features, check their provenance and understand whether they are fit in a given context

With that in mind I would argue that the existence of feature stores is caused by shortcomings  and inconsistencies of data infrastructures deployed in different companies. Effectively introducing a feature store is just duplicating data (which could/should be stored back in the existing datalake/data warehouse) and introducing complexity (e.g. by introducing another type of a database in your infrastructure). On the other hand, it is perfectly reasonable to use a dedicated system to solve a very specific use-case (especially if the technical requirements are very different, e.g. latency needs to be small comparing to e.g. HDFS used in a traditional Hive warehouse).

Ideally, the data warehouse (and/or datalake) can be used as a feature store (this means your data is clean enough and has good metadata to trust it can be fed to a ML model). If there are performance issues related the technology choices made in the past, I would look into creating a general solution (e.g. introduce an additional layer to the datalake which nicely caches the data in another type of database, but hides the complexity of managing it behind some nice interface).

# Examples of feature stores

## Michelangelo - explicit feature store in the architecture

```
Offline
Uber’s transactional and log data flows into an HDFS data lake and is easily accessible via Spark and Hive SQL compute jobs. We provide containers and scheduling to run regular jobs to compute features which can be made private to a project or published to the Feature Store (see below) and shared across teams, while batch jobs run on a schedule or a trigger and are integrated with data quality monitoring tools to quickly detect regressions in the pipeline–either due to local or upstream code or data issues.

Online
Models that are deployed online cannot access data stored in HDFS, and it is often difficult to compute some features in a performant manner directly from the online databases that back Uber’s production services (for instance, it is not possible to directly query the UberEATS order service to compute the average meal prep time for a restaurant over a specific period of time). Instead, we allow features needed for online models to be precomputed and stored in Cassandra where they can be read at low latency at prediction time.

We support two options for computing these online-served features, batch precompute and near-real-time compute, outlined below:

Batch precompute. The first option for computing is to conduct bulk precomputing and loading historical features from HDFS into Cassandra on a regular basis. This is simple and efficient, and generally works well for historical features where it is acceptable for the features to only be updated every few hours or once a day. This system guarantees that the same data and batch pipeline is used for both training and serving. UberEATS uses this system for features like a ‘restaurant’s average meal preparation time over the last seven days.’
Near-real-time compute. The second option is to publish relevant metrics to Kafka and then run Samza-based streaming compute jobs to generate aggregate features at low latency. These features are then written directly to Cassandra for serving and logged back to HDFS for future training jobs. Like the batch system, near-real-time compute ensures that the same data is used for training and serving. To avoid a cold start, we provide a tool to “backfill” this data and generate training data by running a batch job against historical logs. UberEATS uses this near-realtime pipeline for features like a ‘restaurant’s average meal preparation time over the last one hour.’

Shared feature store
We found great value in building a centralized Feature Store in which teams around Uber can create and manage canonical features to be used by their teams and shared with others. At a high level, it accomplishes two things:

It allows users to easily add features they have built into a shared feature store, requiring only a small amount of extra metadata (owner, description, SLA, etc.) on top of what would be required for a feature generated for private, project-specific usage.
Once features are in the Feature Store, they are very easy to consume, both online and offline, by referencing a feature’s simple canonical name in the model configuration. Equipped with this information, the system handles joining in the correct HDFS data sets for model training or batch prediction and fetching the right value from Cassandra for online predictions.
At the moment, we have approximately 10,000 features in Feature Store that are used to accelerate machine learning projects, and teams across the company are adding new ones all the time. Features in the Feature Store are automatically calculated and updated daily.

In the future, we intend to explore the possibility of building an automated system to search through Feature Store and identify the most useful and important features for solving a given prediction problem.
```

## Feast, the  feature store for Kubeflow

* Deployed directly in the k8s cluster
* After deployment, users can use the API right away
* Also uses a concept of an Entity (this basically defines a primary key in a "table")
* Then you define some features (basically names + data types) ...
* ... and a data source (e.g. the format, physical location, timestamp columns etc)...
* ... and you tie all those things together in a FeatureTable object
* Finally you call the "ingest()" function
* All of the above steps are done in the Python SDK (see demo notebook)
* Data can be retrieved in large offline batches (`get_historical_features`)
* there is an online access mode (`get_online_features`) for low-latency access
  * but this requires triggering loading from offline -> online storage

* [source](https://www.kubeflow.org/docs/components/feature-store/overview/)
* [sample notebook with API demo](https://github.com/feast-dev/feast/blob/master/examples/hello-world/feast-hello-world.ipynb)

## FB Learner Flow

* Does not seem to have a distinct feature store

## Tecton

* paid tool
* seems to be a bit more broader than just a "feature store" - provides also serving and transformations for input data
* provides online and offline serving (batch and API access)
* interactions with the system are implemented in 3 ways:
  * cli - feature definition changes
  * web ui - monitoring of the environment
  * python sdk - allows to access the feature store in Databricks or EMR notebooks or to access the data through Consumption API (rest api ?)
* data model
  * first, tecton introduces Entities (object classes)
  * then every feature value is associated with one or more entities
* input data can be transformed with SQL or PySpark, there can be online transformations too
* [source](https://www.tecton.ai/blog/what-is-a-feature-store/)

## AWS Sage Maker

* **Note:** weirdly similar to Feast
* Fully integrated in Amazon's console (there is ajupter workbook though :))
* Feature records are analogs of db rows
* You work with it in a following way:
  * in a jupyter notebook you load/clean the data
  * then you define the "schema" - you create "features" in a "feature group"
  * there are two key "features":
    * unique ids (transaction id, etc - something that uniquely identify a "row")
    * event time - timestamp of the event/feature
  * then you define the data location
  * ... and call the "ingest()" function to actually load the data
  * there is a batch API and a single record API
* after ingestion data is both in offline and online stores
* you can access the data by:
  * fetching it from s3
  * reading it from a hive table (there is a nice api to generate external hive tables)
  * dumping it to csv

* [source](https://aws.amazon.com/sagemaker/feature-store/#)
* [video](https://www.youtube.com/watch?v=-ydEYWhYlYw)

# More links

* [here the author says DWH/DL is not a good feature store](https://medium.com/data-for-ai/feature-store-vs-data-warehouse-306d1567c100)
* [a lot of additional resources (not only feature stores)](https://github.com/full-stack-deep-learning/course-gitbook/blob/master/course-content/where-to-go-next.md)
* [nice summary of feature stores](https://www.featurestore.org/)
* [direct link to the nice table from previous site](https://docs.featurestore.org/feature-store-comparison)
