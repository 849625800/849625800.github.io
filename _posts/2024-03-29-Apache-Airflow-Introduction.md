---
layout: post
title: Apache Airflow - Introduction
date: 2024-03-29 17:40 +0800
last_modified_at: 2024-03-29 18:30 +0800
math: true
tags: [airflow]
toc:  true
---

# Introduction
The most important task for doing data engineering is to build a data workflow. The workflow will include geathering data from different data sources, do transformation to the raw data and then load the data to the target place.

We can build up this workflow using single python script and create a cron job. But this is only manageable when we have little python script. With increasing of the number of python script, cron job become hard to manage. Apache Airflow is a tool designed to handle this situation.

# Why Apache Airflow?

In our society, a huge amount of data piplines is required to process datas in order to keep different service available. Since the world is fast changing, the data we deal with should be updated with the same speed. So, what if some pipline turn down? And what if we want to change the order of operations? Apache Airflow ensure the process work smoothly. We can easily build, schedule and run our pipline using Apache Airflow.

It enable us to create and manage data pipline as a code. Furthermore, it is opensource so that it is free to use.

# DAG (Directed Acyclic Graph)
Apache Airflow use DAG to represent a data workflow. It created a blueprint to define how task should work step-by-step. The tasks in this graph should only work in one single order and no inner loops.

![img](/assets/post_img/2024-03-29-Apache-Airflow-Introduction/DAG.png)

# Declaring a DAG

There are three ways to declare a DAG - either you can use a context manager, which will add the DAG to anything inside it implicitly:

```python
 import datetime

 from airflow import DAG
 from airflow.operators.empty import EmptyOperator

 with DAG(
     dag_id="my_dag_name",
     start_date=datetime.datetime(2021, 1, 1),
     schedule="@daily",
 ):
     EmptyOperator(task_id="task")
```

Or, you can use a standard constructor, passing the DAG into any operators you use:
```python
 import datetime

 from airflow import DAG
 from airflow.operators.empty import EmptyOperator

 my_dag = DAG(
     dag_id="my_dag_name",
     start_date=datetime.datetime(2021, 1, 1),
     schedule="@daily",
 )
 EmptyOperator(task_id="task", dag=my_dag)
```

Or, you can use the @dag decorator to turn a function into a DAG generator:

```python
import datetime

from airflow.decorators import dag
from airflow.operators.empty import EmptyOperator


@dag(start_date=datetime.datetime(2021, 1, 1), schedule="@daily")
def generate_dag():
    EmptyOperator(task_id="task")


generate_dag()
```

# Operator
An Operator is conceptually a template for a predefined Task, that you can just define declaratively inside your DAG:

```python
# open the DAG
with DAG("my-dag") as dag:
    # initialize predefined tasks
    ping = HttpOperator(endpoint="http://example.com/update/")
    email = EmailOperator(to="admin@example.com", subject="Update complete")
    # define the execution order of each task.
    ping >> email
```
Apache airflow provides many Operators for different purpose. For example: `HttpOperator`,`MySqlOperator`,`PostgresOperator`,`MsSqlOperator`,`OracleOperator`,`JdbcOperator`, `DockerOperator`, `HiveOperator`,`S3FileTransformOperator`, `PrestoToMySqlOperator`,`SlackAPIOperator`, ect.

# Reference 
Apache airflow document : [doc](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html)

Youtube video from [Darshil Parmar](https://www.youtube.com/watch?v=5peQThvQmQk&list=RDCMUCChmJrVa8kDg05JfCmxpLRw&start_radio=1&rv=5peQThvQmQk&t=0)