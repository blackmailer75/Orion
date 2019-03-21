<p align="center">
<img width=30% src="https://dai.lids.mit.edu/wp-content/uploads/2018/08/orion.png" alt=“Orion” />
</p>

<p align="center">
<i>Orion is a machine learning library built for telemetry data generated by satellites.</I>
</p>

<p align="center">
<i>A project from Data to AI Lab at MIT.</I>
</p>

# Orion

Orion is a machine learning library built for telemetry data generated by Satellites. With this data, our interest to develop techniques 
* Identify rare patterns and flag them for expert review. 
* Predict outcomes ahead of time. 

The library makes use of a number of automated machine learning tools developed under
["The human data interaction project"](https://github.com/HDI-Project) within the
[Data to AI Lab at MIT](https://dai.lids.mit.edu/).

The focus, with the ready availability of automated machine learning is on:

* domain expert interaction with the machine learning system
* learning from minimal labels
* explainability of model outputs
* model audit
* scalability

## License

- MIT license

## Data Format

### Input

**Orion Pipelines** work on time Series that consist of a single table of telemetry observations
with two columns:

* `timestamp`: an INTEGER or FLOAT column with the time of the observation in
  [Unix Time Format](https://en.wikipedia.org/wiki/Unix_time)
* `value`: an INTEGER or FLOAT column with the observed value at the indicated timestamp

This is an example of such table:

|  timestamp |                value |
|------------|----------------------|
| 1222819200 | -0.3663589458809165  |
| 1222840800 | -0.3941077801118832  |
| 1222862400 |  0.4036246025342055  |
| 1222884000 | -0.36275905932000696 |
| 1222905600 | -0.3707464936807424  |

### Output

The output of the **Orion Pipelines** is another table that contains the detected anomalous
intervals and that has at least two columns:

* `start`: timestamp where the anomalous interval starts
* `end`: timestamp where the anomalous interval ends

Optionally, a third column called `score` can be included with a value that represents the
severity of the detected anomaly.

An example of such a table is:

|      start |        end |              score |
|------------|------------|--------------------|
| 1222970400 | 1222992000 | 0.5726435987608016 |
| 1223013600 | 1223035200 | 0.5726435987608016 |

## Dataset we use in this library

For development, evaluation of pipelines, we include a dataset which includes several satellite telemetry signals already formatted as expected by the Orion Pipelines.

This formatted dataset can be browsed and downloaded directly from the
[d3-ai-orion AWS S3 Bucket](https://d3-ai-orion.s3.amazonaws.com/index.html).

This dataset is adapted of the one used for the experiments in the
[Detecting Spacecraft Anomalies Using LSTMs and Nonparametric Dynamic Thresholding paper](https://arxiv.org/abs/1802.04431),
available from download [Original source data here](https://s3-us-west-2.amazonaws.com/telemanom/data.zip). We thank NASA for making this data available for public use. 

## Orion Pipelines

The main component in the Orion project are the **Orion Pipelines**, which consist of
[MLBlocks Pipelines](https://hdi-project.github.io/MLBlocks/advanced_usage/pipelines.html)
specialized on anomaly detection on time series.

As ``MLPipeline`` instances, **Orion Pipelines**:

* consist of a list of one or more [MLPrimitives](https://hdi-project.github.io/MLPrimitives/)
* can be *fitted* on some data and later on used to *predict* anomalies on more data
* can be *scored* by comparing their predictions with some known anomalies
* have *hyperparameters* that can be *tuned* to improve their anomaly detection performance
* can be stored as a JSON file that includes all the primitives that compose them, as well as
  other required configuration options.

### Current Available Pipelines

In the **Orion Project**, the pipelines are included as **JSON** files, which can be found
inside the [orion/pipelines](https://github.com/D3-AI/Orion/tree/master/orion/pipelines) folder.

This is the list of pipelines available so far, which will grow over time:

| name | location | description |
|------|----------|-------------|
| Dummy | [orion/pipelines/dummy.json](https://github.com/D3-AI/Orion/tree/master/orion/pipelines/dummy.json) | Dummy Pipeline to showcase the input and output format and the usage of sample primitives |
| LSTM Dynamic Thresholding | [orion/pipelines/lstm_dynamic_threshold.json](https://github.com/D3-AI/Orion/tree/master/orion/pipelines/lstm_dynamic_threshold.json) | LSTM Based pipeline inspired by the [Detecting Spacecraft Anomalies Using LSTMs and Nonparametric Dynamic Thresholding paper](https://arxiv.org/abs/1802.04431) |


### Leaderboard 
In this repository we maintain the current scoring of the pipelines. We will keep this leaderboard up-to-date. Read more about our scoring [here](link to a docs page)

|name| score| rank |
|----|------|------|
|LSTM Dynamic Thresholding| 0.67 |1|


## Getting Started

### Requirements

#### Python

**Orion** has been developed and runs on [Python 3.6](https://www.python.org/downloads/release/python-360/).

Also, although it is not strictly required, the usage of a [virtualenv](https://virtualenv.pypa.io/en/latest/)
is highly recommended in order to avoid interfering with other software installed in the system
where **Orion** is run.

#### MongoDB

In order to be fully operational, **Orion** requires having access to a
[MongoDB](https://www.mongodb.com/) database running version **3.6** or higher.

### Install

Since **Orion** is a private project, the only way to install it is by cloning or downloading
its sources from its [GitHub repository](https://github.com/D3-AI/Orion):

```
git clone git@github.com:D3-AI/Orion.git
```

After cloning the repository and creating and activating a virtualenv, you can install
the project with this command:

```
make install
```

For development, use the following command instead, which will install some additional
dependencies for code linting and testing

```
make install-develop
```

### Tutorial: Run a Pipeline on the Demo Dataset

In the following steps we will show a short guide about how to run one of the **Orion Pipelines**
on one of the signals from the **Demo Dataset**.

**NOTE**: All the examples of this tutorial are run in an [IPython Shell](https://ipython.org/),
which you can install by running the following commands inside your *virtualenv*:

```
pip install ipython
ipython
```

#### 1. Load the Data

In the first step we will load the **S-1** signal from the **Demo Dataset**,

To do so, we need to import the `orion.data.load_signal` function and call it passing
the `'S-1'` name.

```
In [1]: from orion.data import load_signal

In [2]: data = load_signal('S-1')

In [3]: data.head()
Out[3]:
    timestamp     value
0  1222819200 -0.366359
1  1222840800 -0.394108
2  1222862400  0.403625
3  1222884000 -0.362759
4  1222905600 -0.370746
```

#### 2. Load a Pipeline

Once we have the data, we will load the LSTM pipeline using an
[MLPipeline](https://hdi-project.github.io/MLBlocks/api/mlblocks.html#mlblocks.MLPipeline)

```
In [4]: from mlblocks import MLPipeline

In [5]: pipeline = MLPipeline.load('orion/pipelines/lstm_dynamic_threshold.json')
Using TensorFlow backend.
```

#### 3. Fit the Pipeline

Once we have the data and the pipeline, we can call the `pipeline.fit` method and passing
it the loaded data:

```
In [6]: pipeline.fit(data)
Epoch 1/1
9899/9899 [==============================] - 55s 6ms/step - loss: 0.0559 - mean_squared_error: 0.0559
```

**NOTE:** Depending on your system and the exact versions that you might have installed
some *WARNINGS* may be printed. These can be safely ignored as they do not interfere
with the proper behavior of the pipeline.

#### 4. Detect Anomalies

Once the pipeline has been fitted on the data, we can pass the data back to the pipeline
so that it can detect which intervals are anomalous.

This is done by calling the `pipeline.predict` method and passing it the data again:

```
In [7]: anomalies = pipeline.predict(data)

In [8]: anomalies
Out[8]: array([[1.39806000e+09, 1.39944240e+09, 1.68380893e-01]])
```

The output is a 2d numpy array that contains the three columns explained above.
In this case, the matrix has only one row, which once converted into a *pandas.DataFrame*
and properly formatted looks like this:

```
In [9]: import pandas as pd

In [10]: adf = pd.DataFrame(anomalies, columns=['start', 'end', 'score'])

In [11]: adf['start'] = adf['start'].astype(int)

In [12]: adf['end'] = adf['end'].astype(int)

In [13]: adf
Out[13]:
        start         end     score
0  1398060000  1399442400  0.168381
```

#### 5. Evaluate performance

In this next step we will load some already known anomalous intervals and evalulate how
good our anomaly detection was by comparing those with our detected intervals.

For this, we will first load the known anomalies for the signal that we are using:

```
In [8]: from orion.data import load_anomalies

In [9]: truth = load_anomalies('S-1')

In [10]: truth
Out[10]:
        start         end
0  1392768000  1402423200
```

Afterwards, we pass the ground truth, the detected anomalies and the original data
to the `orion.metrics.accuracy_score` in order to compute a score that indicates
how good our anomaly detection was:

```
In [11]: from orion.metrics import accuracy_score

In [11]: accuracy_score(truth, anomalies, data)
Out[11]: 0.956346078044935
```


### Tutorial (Version 2 - Simplified): Run a Pipeline on the Demo Dataset

In the following steps we will show a short guide about how to run one of the **Orion Pipelines**
on one of the signals from the **Demo Dataset**.

**NOTE**: All the examples of this tutorial are run in an [IPython Shell](https://ipython.org/),
which you can install by running the following commands inside your *virtualenv*:

```
pip install ipython
ipython
```

#### 1. Load the data

In the first step we will load the **S-1** signal from the **Demo Dataset**,

To do so, we need to import the `orion.data.load_signal` function and call it passing
the `'S-1'` name.

```
In [1]: from orion.data import load_signal

In [2]: data = load_signal('S-1')

In [3]: data.head()
Out[3]:
    timestamp     value
0  1222819200 -0.366359
1  1222840800 -0.394108
2  1222862400  0.403625
3  1222884000 -0.362759
4  1222905600 -0.370746
```

#### 2. Detect anomalies using a pipeline

Once we have the data, use the LSTM pipeline to analyze it and search for anomalies.

In order to do so, we will have import the `orion.analysis.analyze` function and pass it
the loaded data and the path to the pipeline JSON that we want to use:

```
In [4]: from orion.analysis import analyze

In [5]: pipeline_path = 'orion/pipelines/lstm_dynamic_threshold.json'

In [6]: anomalies = analyze(pipeline_path, data)
Using TensorFlow backend.
Epoch 1/1
9899/9899 [==============================] - 55s 6ms/step - loss: 0.0559 - mean_squared_error: 0.0559
```

**NOTE:** Depending on your system and the exact versions that you might have installed
some *WARNINGS* may be printed. These can be safely ignored as they do not interfere
with the proper behavior of the pipeline.

The output of the previous command will be a ``pandas.DataFrame`` containing a table in the
Output format described above:

```
In [7]: anomalies
Out[7]:
        start         end     score
0  1398060000  1399442400  0.168381
```

#### 3. Evaluate performance

In this next step we will load some already known anomalous intervals and evalulate how
good our anomaly detection was by comparing those with our detected intervals.

For this, we will first load the known anomalies for the signal that we are using:

```
In [8]: from orion.data import load_anomalies

In [9]: truth = load_anomalies('S-1')

In [10]: truth
Out[10]:
        start         end
0  1392768000  1402423200
```

Afterwards, we pass the ground truth, the detected anomalies and the original data
to the `orion.metrics.accuracy_score` in order to compute a score that indicates
how good our anomaly detection was:

```
In [11]: from orion.metrics import accuracy_score

In [11]: accuracy_score(truth, anomalies, data)
Out[11]: 0.956346078044935
```


## Database

**Orion** comes ready to use a MongoDB Database to easily register and explore:

* Multiple Datasets based on signals from one or more satellites.
* Multiple Pipelines, including historical Pipeline versions.
* Pipeline executions on the registered Datasets, including any environment details required to
  later on reproduce the results.
* Pipeline execution results and detected events.
* Comments about the detected events.

This, among other things, allows:

* Providing visibility about the system usage.
* Keeping track of the evolution of the registered pipelines and their performance over multiple datasets.
* Visualizing and browsing the detected events by the pipelines using a web application.
* Collecting comments from multiple domain experts about the detected events to be able to later
  on curate the pipelines based on their knowledge.
* Reproducing previous executions in identical environments to replicate the obtained results.
* Detecting and keeping a history of system failures for later investigation.


### Database Schema

The **Orion Database** contains the following collections and fields:

#### Dataset

The **Dataset** collection contains all the required details to be able to load
the observations from a satellite signal, as well as some metadata about it, such as
the minimum and maximum timestamps that want to be used or the user that registered it.

##### Fields

* \_id (ObjectID): Unique Identifier of this Dataset object
* name (String): Name of the dataset
* signal_set (String): Identifier of the signal
* satellite_id (String): Identifier of the satellite
* start_time (Integer): minimum timestamp of this dataset
* stop_time (Integer): maximum timestamp of this dataset
* data_location (String): URI of the dataset
* timestamp_column (Integer): index of the timestamp column
* value_column (Integer): index of the value column
* created_by (String): Identifier of the user that created this Dataset Object
* insert_time (DateTime): Time when this Dataset Object was inserted

#### Pipeline

The **Pipeline** collection contains all the pipelines registered in the system, including
their details, such as the list of primitives and all the configured hyperparameter values.

##### Fields

* \_id (ObjectID): Unique Identifier of this Pipeline object
* name (String): Name given to this pipeline
* mlpipeline (SubDocument): JSON representation of this pipeline
* created_by (String): Identifier of the user that created this Pipeline Object
* insert_time (DateTime): Time when this Pipeline Object was inserted

#### Datarun

The **Datarun** objects represent single executions of a **Pipeline** on a **Dataset**,
and contain all the information about the environment and context where this execution
took place, which potentially allows to later on reproduce the results in a new environment.

It also contains information about whether the execution was successful or not, when it started
and ended, and the number of events that were found by the pipeline.

##### Fields

* \_id (ObjectID): Unique Identifier of this Datarun object
* dataset_id (ObjectID - Foreign Key): Unique Identifier of the Dataset used
* pipeline_id (ObjectID - Foreign Key): Unique Identifier of the Pipeline used
* start_time (DateTime): When the execution started
* end_time (DateTime): When the execution ended
* software_versions (List of Strings): version of each python dependency installed in the
  *virtualenv* when the execution took place
* budget_type (String): Type of budget used (time or number of iterations)
* budget_amount (Integer): Budget amount
* model_location (String): URI of the fitted model
* metrics_location (String): URI of the metrics
* events (Integer): Number of events detected during this Datarun execution
* status (String): Whether the Datarun is still running, finished successfully or failed
* created_by (String): Identifier of the user that created this Datarun Object
* insert_time (DateTime): Time when this Datarun Object was inserted

#### Event

Each one of the anomalies detected by the pipelines is stored as an **Event**, which
contains the details about the start time, the stop time and the severity score.

##### Fields

* \_id (ObjectID): Unique Identifier of this Event object
* datarun_id (ObjectID - Foreign Key): Unique Identifier of the Datarun during which this
  Event was detected.
* start_time (Integer): Timestamp where the anomalous interval starts.
* stop_time (Integer): Timestamp where the anomalous interval ends.
* score (Float): Severity score given by the pipeline to this Event
* tag (String): User given tag for this Event
* insert_time (DateTime): Time when this Event Object was inserted

#### Comment

Each Event can have multiple **Comments**, from one or more users.
**Comments** are expected to be inserted by the domain experts after the Datarun has
finished and they analyze the results.

##### Fields

* \_id (ObjectID): Unique Identifier of this Comment object
* event_id (ObjectID - Foreign Key): Unique Identifier of the Event to which this Comment relates
* text (String): Comment contents
* created_by (String): Identifier of the user that created this Event Object
* insert_time (DateTime): Time when this Event Object was inserted


### Database Usage

In order to make **Orion** interact with the database we will need to use the `OrionExplorer`,
which provides all the required functionality to register and explore all the database objects,
as well as load pipelines and datasets from it in order to start new dataruns and detect events.

NOTE: Work in Progress
