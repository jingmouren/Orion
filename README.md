<p align="center">
<img width=30% src="https://dai.lids.mit.edu/wp-content/uploads/2018/08/orion.png" alt=“Orion” />
</p>

<p align="center">
<i>Orion is a machine learning library built for data generated by satellites.</I>
</p>

<p align="center">
<i>A project from Data to AI Lab at MIT.</I>
</p>

# Orion

Orion is a machine learning library built for sensor data collected from Satellites. The library
makes use of a number of automated machine learning tools developed under
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

**Orion Pipelines** work on Time Series that consist of a single table of observations
with two columns:

* `timestamp`: an INTEGER or FLOAT column with the time of the observation in
  [Unix Time Format](https://en.wikipedia.org/wiki/Unix_time)
* `value`: an INTEGER or FLOAT column with the observed value at the indicated timestamp

This is an example of such table:

|    timestamp |                value |
|--------------|----------------------|
| 1222819200.0 | -0.3663589458809165  |
| 1222840800.0 | -0.3941077801118832  |
| 1222862400.0 |  0.4036246025342055  |
| 1222884000.0 | -0.36275905932000696 |
| 1222905600.0 | -0.3707464936807424  |

### Output

The output of the **Orion Pipelines** is another table that contains the detected anomalous
intervals and that has at least two columns:

* `start`: timestamp where the anomalous interval starts
* `end`: timestamp where the anomalous interval ends

Optionally, a third column called `score` can be included with a value that represents the
severity of the detected anomaly.

An example of such a table is:

|        start |          end |              score |
|--------------|--------------|--------------------|
| 1222970400.0 | 1222992000.0 | 0.5726435987608016 |
| 1223013600.0 | 1223035200.0 | 0.5726435987608016 |

## Demo Dataset

Orion includes a demo dataset which includes several satellite signals already formatted
as expected by the Orion Pipelines.

This dataset can be browsed and downloaded directly from the
[d3-ai-orion AWS S3 Bucket](https://d3-ai-orion.s3.amazonaws.com/index.html).

This dataset is an adaptation of the one used for the experiments in the
[Detecting Spacecraft Anomalies Using LSTMs and Nonparametric Dynamic Thresholding paper](https://arxiv.org/abs/1802.04431),
available from download [here](https://s3-us-west-2.amazonaws.com/telemanom/data.zip)

## Orion Pipelines

The main element in the Orion project are the **Orion Pipelines**, which consist of
[MLBlocks Pipelines](https://hdi-project.github.io/MLBlocks/advanced_usage/pipelines.html)
specialized on anomaly detection on time series.

As ``MLPipeline`` instances, **Orion Pipelines**:

* consist of a list of one or more [MLPrimitives](https://hdi-project.github.io/MLPrimitives/)
* can be *fitted* on some data and later on used to *predict* anomalies on more data
* can be *scored* by comparing their predictions with some known anomalies
* have *hyperparameters* that can be *tuned* to improve their anomaly detection performance
* can be stored as a JSON file that includes all the primitives that compose them, as well as
  other required configuration options.

### JSON Pipelines

In the **Orion Project**, the pipelines are included as **JSON** files, which can be found
inside the [orion/pipelines](https://github.com/D3-AI/Orion/tree/master/orion/pipelines) folder.

This is the list of pipelines available so far, which will grow over time:

| name | location | description |
|------|----------|-------------|
| Dummy | [orion/pipelines/dummy.json](https://github.com/D3-AI/Orion/tree/master/orion/pipelines/dummy.json) | Dummy Pipeline to showcase the input and output format and the usage of sample primitives |
| LSTM Dynamic Thresholding | [orion/pipelines/lstm_dynamic_threshold.json](https://github.com/D3-AI/Orion/tree/master/orion/pipelines/lstm_dynamic_threshold.json) | LSTM Based pipeline inspired by the [Detecting Spacecraft Anomalies Using LSTMs and Nonparametric Dynamic Thresholding paper](https://arxiv.org/abs/1802.04431) |

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

In [2]: s1 = load_signal('S-1')

In [3]: s1.head()
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
In [6]: pipeline.fit(s1)
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
In [7]: anomalies = pipeline.predict(s1)

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
