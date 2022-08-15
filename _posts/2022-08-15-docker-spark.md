---
layout: post
title: "Deploying Apache Spark, Jupyter Lab with support for Scala/Python"
subtitle: "By Kaus"
author: "Kaus"
header-img: "img/in-post/2022-08-15-spark.jpg"
header-mask: 0.4
tags: 
    - Docker
    - Spark
    - Scala
    - Jupyter-Lab
    - Ammonite
---

# Apache Spark, Scala and Jupyter Lab Docker image
#### [Git Link](https://github.com/kaus98/spark-jupyter-docker)

Apache Spark is popular tool used in Big Data for processing large data at Parallel sequence. It support integration with multiple languages among which  Scala and Python are main ones. In todays industries, it is used by almost every large scale company to format there data, perform ETL Operations, Data migrations and Machine Learning.  
Other important tool, Jupyter is important and most popularly used with Python for Development and Deep Learning. The Availabilty to include Markdown and Code into single file make it ideal for POC and Testings.

## Step 1: Installing Docker and Tools.
If not already installed, you can refer to [instructions](https://www.docker.com/get-started/) at docker site for you OS.  
We will be building base image which will have Apache Spark, Scala and Python Installed and we will import the same base image to different images for our use.  

First we will be building the Main Spark Master and then run another instance of Slave spark instance. We will run another Instance that will Run Jupyter lab server.

## Step 2: Building the Base Image.
For our learning, we will be setting up everything from screth over debian base Image and setup basic variables and Make our shared Workspace.
```
FROM debian:stretch

# Adding Env Vairables which we will need later
RUN mkdir -p /tmp/logs/ && chmod a+w /tmp/logs/ && mkdir /app && chmod a+rwx /app && mkdir /data && chmod a+rwx /data
ENV JAVA_HOME=/usr
ENV SPARK_HOME=/usr/bin/spark-3.3.0-bin-hadoop3
ENV PATH=$SPARK_HOME:$PATH:/bin:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
ENV SPARK_MASTER_HOST spark-master
ENV SPARK_MASTER_PORT 7077
ENV PYSPARK_PYTHON=/usr/bin/python
ENV PYTHONPATH=$SPARK_HOME/python:$PYTHONPATH
ENV APP=/app
ENV SHARED_WORKSPACE=/opt/workspace
RUN mkdir -p ${SHARED_WORKSPACE}
```
For first step, we will install Python=3.8 in debian along with some important dependencies. Since apt install python=3.6 by default, we will be installing Python from scratch.   
We will also install Java-8 (openjdk-8-jdk) along with importany plugins.

```
#Install Python=3.8
RUN apt update && \
    apt install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev \
    libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev curl \
    wget libbz2-dev && \
    wget https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tgz && \
    tar -xf Python-3.8.0.tgz && cd Python-3.8.0 && \
    ./configure --enable-optimizations && \
    make -j 8 && make altinstall && \
    cd .. && rm -rf ./Python-3.8.0* && \
    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python3.8 get-pip.py && rm -rf ./get-pip.py && \
    pip3 install numpy scipy matplotlib && \
    rm -f /usr/bin/python && ln -s /usr/local/bin/python3.8 /usr/bin/python

# System packages
RUN apt-get clean && apt-get update -y && \
    apt-get install -y curl wget unzip procps openjdk-8-jdk coreutils && \
    rm -rf /var/lib/apt/lists/*

```

Now in the next process we can install latest Apache Spark and Scala in our base image. At time of writing Apache Spark had latest 3.3.0. For Scala, we will use 2.12.12 version.

```
# Install Spark
RUN curl https://dlcdn.apache.org/spark/spark-3.3.0/spark-3.3.0-bin-hadoop3.tgz -o spark.tgz && \
    tar -xf spark.tgz && \
    mv spark-3.3.0-bin-hadoop3 /usr/bin/ && \
    mkdir /usr/bin/spark-3.3.0-bin-hadoop3/logs && \
    rm spark.tgz

# Install Scala in Docker
ARG SCALA_VERSION=2.12.12
RUN mkdir -p ${SHARED_WORKSPACE}/data && \
    mkdir -p /usr/share/man/man1 && \
    curl https://downloads.lightbend.com/scala/${SCALA_VERSION}/scala-${SCALA_VERSION}.deb -k -o scala.deb && \
    apt install -y ./scala.deb && \
    rm -rf scala.deb /var/lib/apt/lists/*

# Setup Ammonium Inside Docker
RUN curl -L -o /usr/local/bin/amm https://git.io/vASZm && chmod +x /usr/local/bin/amm

VOLUME ${SHARED_WORKSPACE}
```