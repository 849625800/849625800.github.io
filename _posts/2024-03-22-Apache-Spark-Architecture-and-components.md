---
layout: post
title: Apache Spark - Architecture and Components
date: 2024-03-21 20:43 +0800
last_modified_at: 2024-03-21 20:43 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction
This note will look at a high-level of Spark architecture and understand how it execute actions across notes

# Master-Slave Architecutre

![image](/assets/post_img/2024-3-22-Apache-Spark-Architecture/master-slave.png)

- The program with `SparkContext` is a driver program running in a master node, which is responsible to manage the worker Nodes (Executors) that runs in a Java process. 

- In the figure above, the driver program schedule the user script into 2 tasks and each task can be execute by a Executor. Each task is called a Spark Job. 

- This process starts at the beginning of the spark application and presist for the entire life time of the application.

- Once the task has been executed, the result will be send back to the driver program. The driver program will access the spark clusters by the SparkContext object and return a RDD we need.

# Driver program coordinate and controls all the operations on RDD in a spark job.

In the following example, the spark job will return the word count of a RDD.

![image](/assets/post_img/2024-3-22-Apache-Spark-Architecture/driver_program.png)

- The driver program will split the RDD into several partitions. Each partition will be passed into different executors on the cluster. 

- Each executor will count the words respetively. Then return the result to the driver program.

- The driver program will merge the result and return back to the end user.

- The computing power of Spark can be upgraded by adding more worker Node on the spark cluster.

# Spark Components

![img](/assets/post_img/2024-3-22-Apache-Spark-Architecture/components.png)

- `Spark Core`: the fundation of the overall project. It provide distributed tasks despatching, scheduling and basic functionalities exposes through the application programming that centered on the RDD, whcih is the spark basic programming abstraction. Spark Core provides the basic AIPs to maniputate those actions.

- `Spark SQL`: A spark package designed for working with structured data. It provides an SQL-like interface for working with this type of data. With spark SQL, we can write Spark query, which looks pretty much like SQL query, which can trigger several RDD operations on spark. This simplify the working with structure dataset.

- `Spark Streaming`: Specifically designed to process the streaming data in real time instead of batched data. It provides an API for manipulating data streams that closely mathc the Spark Core's RDD API, which enables powerful interactive and analytical applications across both streaming and historical data while inheriting Spark's ease of use and fault torlerance characteristics. It is compatable with variaties of data sources including HDFS, Kafka, and twitter etc.

- `Spark MLlib`: MLlib is a scalable machine learning library that delivers both high-quality algorithms and blazing speed. It is usable in Java, Scala and Python as a part of Spark applications. It also containes several vommoin learning algorithms and utilities including classification, regression, clustering, collaborative filtering, dimensionality reduction, etc.

- `Graph X`: A graph computation engin built on top of spark that enables users to interactively create, transform and reason about graph structured data at scale. It extends the Spark RDD by introductin a new Graph abstraction: a directed multigraph with properties attached to each vertex and edge. It comes with some common algorithms like page ranks and triangle counting.

# Who use spark? How?

- `Data Scientists`: Identify patterns, trends, risks and opportunities in data. They need to tell a story of data and a new actionable insight. Ad hoc analysis is usually their responsibility. Spark helps them by supporting the entire data science workflow, form data access, ad-hoc analysis and integration to machine learning and visulilzation.

- `Data processing application engineers`: They build applications that leverage advanced analytics in partnership with the data scientist. General classed of applications are moving to Spark, including compute-intensive applications and applications that require input from data streams, such as sensors and social data. Spark parallelize these applications across clusters and hide the complecity of distributed systems programming, network communication and fault tolerance.

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=18)

The code can be found in the [Github repository](https://github.com/yu-jinh/Apache-Spark-Playground)