---
layout: post
title: FB Learner Flow notes
---

* **FBs ML infrastructure project**
* Not mentioned explicitly, but probably very well integrated with the other parts of fb stack
* There are 3 key concepts:
  * Workflows - pipelines defined in python; they are DAGs of operators
  * Operators - individual building blocks (similar to Operators in Airflow)
  * Channels - represent inputs and outputs (they mark what flows between operators); use a type-system avoid errors when connecting operators
* Platform has 3 main components:
  * Tool to author the workflows
  * Tool to execute workflows (launch experiments) with differents inputs and viewing their results
  * A catalog of ready to use workflows/pipelines
* Platform seems to support only offline models (but hey, the description is from 2016 so by now it is probaly supporting also online learning)
* UI has following functions:
  * Can launch a workflow
    * You can specify dates of data (partitions), features (those come from an internal system), which workflow is being executed, name/desc/tags etc metadata.
  * Visualize output
    * Type system is used to infer how each output can be visualized
    * Output reports can be enhanced with custom plugins (e.g. if a model/algorithm has a special visualization, a plugin which supports it can be added)
  * Manage experiments
    * This is basically running a few versions of the same workflow, but with different hyperparameters of training
    * experiment reports are centrally stored and easy to query (indexed with elasticsearch)
* Core properties:

  ```
  * Each machine learning algorithm should be implemented once in a reusable manner.
  * Engineers should be able to write a training pipeline that parallelizes over many machines and can be reused by many engineers.
  * Training a model should be easy for engineers of varying ML experience, and nearly every step should be fully automated.
  * Everybody should be able to easily search past experiments, view results, share with others, and start new variants of a given experiment.
  ```

* Conceptually this is a set of easy to re-use workflows, which can be plugged in into various sets of features
* Workflow definitions are Python-based (similarly to Airflow).
* Core insight: model kind isn't that important, it is more important to correctly choose the set of features
  * This means it is far more important to make it _easy_ to switch between features

## Key excerpts from blog

Explanation of how the execution works:

```
Rather than execute serially, workflows are run in two stages: 1) the DAG compilation stage and 2) the
operator execution stage. In the DAG compilation stage, operators within the workflow are not actually
executed and instead return futures. Futures are objects that represent delayed pieces of computation.
So in the above example, the dt variable is actually a future representing decision tree training that has
not yet occurred. FBLearner Flow maintains a record of all invocations of operators within the DAG
compilation stage as well as a list of futures that must be resolved before it operates. For example, both
ComputeMetricsOperator and PredictOperator take dt.model as an input, thus the system knows that dt
must be computed before these operators can run, and so they must wait until the completion of
TrainDecisionTreeOperator.
```

Philosophy of the system:

```
In some of our earliest work to leverage AI and ML — such as delivering the most relevant content to each
person — we noticed that the largest improvements in accuracy often came from quick experiments,
feature engineering, and model tuning rather than applying fundamentally different algorithms. An
engineer may need to attempt hundreds of experiments before finding a successful new feature or set of
hyperparameters. Traditional pipeline systems that we evaluated did not appear to be a good fit for our
uses — most solutions did not provide a way to rerun pipelines with different inputs, mechanisms to
explicitly capture outputs and/or side effects, visualization of outputs, and conditional steps for tasks like
parameter sweeps.
```

## Sample workflow

  ```python
  # The typed schema of the Hive table containing the input data
feature_columns = (
    ('petal_width', types.INTEGER),
    ('petal_height', types.INTEGER),
    ('sepal_width', types.INTEGER),
    ('sepal_height', types.INTEGER),
)
label_column = ('species', types.TEXT)
all_columns = feature_columns + (label_column,)

# This decorator annotates that the following function is a workflow within
# FBLearner Flow
@workflow(
    # Workflows have typed inputs and outputs declared using the FBLearner type
    # system
    input_schema=types.Schema(
        labeled_data=types.DATASET(schema=all_columns),
        unlabeled_data=types.DATASET(schema=feature_columns),
    ),
    returns=types.Schema(
        model=types.MODEL,
        mse=types.DOUBLE,
        predictions=types.DATASET(schema=all_columns),
    ),
)
def iris(labeled_data, unlabeled_data):
    # Divide the dataset into separate training and evaluation dataset by random
    # sampling.
    split = SplitDatasetOperator(labeled_data, train=0.8, evaluation=0.2)

    # Train a decision tree with the default settings then evaluate it on the
    # labeled evaluation dataset.
    dt = TrainDecisionTreeOperator(
        dataset=split.train,
        features=[name for name, type in feature_columns],
        label=label_column[0],
    )
    metrics = ComputeMetricsOperator(
        dataset=split.evaluation,
        model=dt.model,
        label=label_column[0],
        metrics=[Metrics.LOGLOSS],
    )

    # Perform predictions on the unlabeled dataset and produce a new dataset
    predictions = PredictOperator(
        dataset=unlabeled_data,
        model=dt.model,
        output_column=label_column[0],
    )

    # Return the outputs of the workflow from the individual operators
    return Output(
        model=dt.model,
        logloss=metrics.logloss,
        predictions=predictions,
    )
  ```

#### Links

* [Source](https://engineering.fb.com/2016/05/09/core-data/introducing-fblearner-flow-facebook-s-ai-backbone/)
