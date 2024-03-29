---
layout: post
title: Apache Spark - SparkSQL Performance Tuning
date: 2024-03-28 17:40 +0800
last_modified_at: 2024-03-28 18:30 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction

This blog will intruduce some approach to tune the performance of SparkSQL.

# Methods

- **Built-in Optimization:**
Spark SQL has some built-in optimizations such as predicate push-down which allows Spark SQL to move some parts of our query down to the engine we are querying.

- **Caching:** If our application includes performing some queries or transformations on a dataframe repeatedly, we may find caching the dataframe useful. It can be done by calling the cache method on the dataframe. This is similar to caching a RDD.

    ```python
    responseDataFrame.cache()
    ```
    When caching a dataframe, Spark SQL uses an in-memory columnar storage for the dataframe. This storage take less space. 

    If our subsequent queries depend only on subsets of the data, SparkSQL will minimize the data read, and automatically tune compression to reduce garbage collection pressure and memory usage.

- **Configure Spark Properties:**

    - `spark.sql.codegen`: 
        ```python
        session = SparkSession.builder \
        .config("spark.sql.codegen", value = False) \
        .getOrCreate()
        ```
        It will ask Spark SQL to compile each query to Java byte code before executing it.

        This codegen option could take long queries or repeated queries substantially faster, as Spark generates specific code to run them.

        For short queries or some non-repeated ad-hoc queries, this option could add unnecessary overhead, as Spark has to run a compiler for each query.

        It's recommanded to use codegen option for workflows which involves large queries, or with the same repeated query.

    - `spark.sql.inMemoryColumnarStorage.batchSize`:
        ```python
        session = SparkSession.builder \
        .config("spark.sql.inMemoryColumnarStorage.batchSize", value = 1000) \
        .getOrCreate()
        ```
        When caching dataframe, Spark groups together the records in batches of the size given by this option and compresses each batch.

        The default batch size is 1000.

        Having a large batch size can imporve memory utilization and compression.

        A bath with a large number of reconrds might be hard to build up in memory and can lead to an `OutOfMemoryError`. If the size of each data sample is large, we should consider using smaller batch size to avoid `OutOfMemoryError`.

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=38)

The code can be found in the [Github repository](https://github.com/yu-jinh/Apache-Spark-Playground)