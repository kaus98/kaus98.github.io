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
If not already installed, you can refer to [instructions](https://www.docker.com/get-started/) at docker site for you OS. We will be building one base image which will have Apache Spark, Scala and Python Installed and we will will import the same base image to different images for our use.
First we will be building the Main Spark Master and then run another instance of Slave spark instance. We will run another Instance that will Run Jupyter lab server.

## Step 2: Building the Base Image.
For our learning, we will be setting up everything from screth over debian base Image.
```
FROM debian:stretch
```
