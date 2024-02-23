# SAIL Winter School 2024 – Evaluating Machine Learning

This directory contains is used for the [SAIL Winter School 2024](https://indico.uni-paderborn.de/event/62/). It comes with a pre-configured HOBBIT platform deployment for local development as well as a very basic benchmark and baseline system implementation for Java and Python, respectively.

## Scenario and Benchmark Design

The target of this tutorial is to implement a benchmark for a regression task. As example, we use the [🍷 wine quality dataset](https://archive.ics.uci.edu/dataset/186/wine+quality) of [Cortez et al](https://repositorium.sdum.uminho.pt/bitstream/1822/10029/1/wine5.pdf). In the following, we will briefly look at the data, define the task that a benchmarked system should fulfill and design of the benchmark and its API.

### Data 

The dataset comprises two files. One file is for red, the second is for white wine. We will handle these two files as two distinct datasets. However, both of them have the same structure. They come as CSV files in the following form:
```
"fixed acidity";"volatile acidity";"citric acid";"residual sugar";"chlorides";"free sulfur dioxide";"total sulfur dioxide";"density";"pH";"sulphates";"alcohol";"quality"
7;0.27;0.36;20.7;0.045;45;170;1.001;3;0.45;8.8;6
6.3;0.3;0.34;1.6;0.049;14;132;0.994;3.3;0.49;9.5;6
8.1;0.28;0.4;6.9;0.05;30;97;0.9951;3.26;0.44;10.1;6
7.2;0.23;0.32;8.5;0.058;47;186;0.9956;3.19;0.4;9.9;6
```
The structure of the file will be important later on. At the moment, it is sufficient to recognize that the file contains several features in a tabular format and that the last column contains the `"quality"`. This is the target value, that a system should predict.

### System's Task

We define the task that a system, which will be evaluated with our benchmark, should fulfill as a regression. The target values are the quality levels 0 (very bad) to 10 (excellent). Note that the data itself only contains labels ranging from 3 to 9 and that the data is unbalanced, i.e., the most wines belong to the quality levels 5, 6 or 7.

We further want to enable the usage of supervised machine learning. Hence, our benchmark has to ensure that the data is split into training and test data. The training data has to be made available for the system to train an algorithm. Further, we define that the split should be done randomly and that 90% of the data should be used for training.

The benchmark should measure the effectiveness and efficiency of a system. While the effectiveness can be measured with a large number of different measures, we focus on the usage of Micro Precision, Recall and F1-measure. The efficiency should be measured in form of the time that the system needs to classify the single examples.

### Benchmark Design

A major part of the work on a benchmark is to define its internal workflow and the API that the system has to implement to be evaluated by the benchmark. For this example, we use a quite simple setup with only two containers—a benchmark and a system container. In addition to the [general API of the HOBBIT platform](system_integration_api.html) that has to be implemented by the system, the benchark's API comprises the following parts:
1. The task definition above already stated that the data has to be split to get training and test data. We define that 10% of the dataset are randomly chosen as test dataset. The remaining 90% are used as training data and are sent to the system at the beginning as a CSV file of the form above.
2. The system should have the time to train an internal algorithm using the training data. Hence, the evaluation of the system should only start after the system signalled that the internal training has finished and that it is ready to be benchmarked.
3. The benchmark will sent the single tasks (i.e., the instances of the test set) as CSV comprising the headline and a single data line. The first column of the line contains the ID of the task, which has to be part of the response. The column with the quality level is removed before submission.
4. The system is expected to send a single CSV line comprising two values: Firstly, the task ID. Secondly, the value that it predicts for the task.

Our benchmark will have several parameters:
* **Dataset**: There are two wine datasets. One for red and a second for white wine. The user should be able to choose which dataset they want to use.
* **Seed**: The split into train and test data will be done randomly. However, the user should be able to define a seed value to ensure that an experiment is repeatable.
* **Test dataset size**: The benchmark should report the size of the randomly created test dataset.

The benchmark should provide the following evaluation results (also called key performance indicators (KPIs)):
* **Runtime**: The average runtime that the system needs to answer a request (including its standard deviation).
* **Faulty responses**: The number of faulty responses that the system may have produced. This avoids to include them into the error calculation and allows the benchmark to report that the system did not always create correctly formed answers.

The figure below gives an overview of the benchmark and its components as well as the type of data that they send to each other.

<p align="center">
  <img src="/images/Components-diagram-wine-benchmark.svg" />
</p>

## Prerequisites

The following software has to be available on the machine on which the platform should be run:

* [Docker](https://www.docker.com/) 19.03.13 or newer
* [Docker Compose](https://docs.docker.com/compose/) 1.19.0 or newer

In addition, an editor for developing Java or Python programs should be available. Additionally, [make](https://www.gnu.org/software/make/manual/html_node/index.html) can be helpful since we provide a Makefile that eases the execution of commands.

## Preparations

You should download / checkout this directory. Within this directory, we already prepared the HOBBIT platform in a local, development-friendly setup for you. Before starting the HOBBIT platform, you should run the following two commands once:
```sh
docker swarm init
make create-networks
```
The first will initialize the swarm mode of Docker. The second will create three Docker overlay networks, which are needed for the components to talk to each other later on.

### HOBBIT Platform

After the initialization above, the platform can be started with:
```sh
make start-hobbit-platform
```
After that, you can find the user interface (UI) of the platform at [http://localhost:8080].

The platform can also be stopped with:
```sh
make stop-hobbit-platform
```

The Platform runs in develop mode. This ensures that you can access Docker containers of the benchmark and system even after they terminated. However, that also means that some of them might be still running. If you want to easily remove containers created by the platform (without stopping the platform itself), you can do that with the following command:
```sh
make remove-hobbit-containers
```
Note that this will also remove containers of a currently running experiment. So please only use it if you are sure that you do not need any of the containers anymore.

### Benchmark and System

We prepared two base implementations for the benchmark and system, respectively. One implementation is in Java and a second in Python. You will find them in the `java` and `python` directories. At the beginning, you may want to build them, to ensure that the build itself works and that you can run them on your local HOBBIT platform.
```sh
make build-java
make build-python
```
After that, you can start an experiment by clicking on `Benchmarks` in the HOBBIT UI, choosing one of the benchmarks and one of the systems and pressing `Submit`.

## Tasks

### Add a new KPI

#### Implementation

#### Meta Data update

### Add another System

#### Meta Data update

### Dataset Extension

Find a dataset, that fits to the task
https://archive.ics.uci.edu/