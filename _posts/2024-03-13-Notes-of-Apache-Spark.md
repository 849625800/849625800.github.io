---
layout: post
title: Notes of Apache Spark
date: 2024-03-13 20:43 +0800
last_modified_at: 2024-03-13 20:43 +0800
math: true
tags: [Actively updating, Data Engineering]
toc:  true
---
# Introduction
This blog will cover some basic concepts and practice of Apache Spark. 

# Environment 
- java version "1.8.0_151"
- spark-2.2.0-bin-hadoop2.7

# RDD (Resilient Distributed Datasets)

RDD is the core object we will be using when developing spark objects. 

- An RDD is a capsulation around a very large dataset. All works in spark are either creating new RDDS, transforming existing RDDs or calling operation on RDDs for computation of a result.

- Spark will automatically distribute the data contrained in RDDs across our cluster and parallelize the operations we perform on them.

We have 2 operations on RDDs:

- **Transformations:** Apply some functions to the data in RDD to create a new RDD (e.g. filter) 

```python
lines = sc.textFile("text_path")
filtered_lines = lines.filter(lambda line: "some conditions" in line)
```

- **Actions:** Compute a result based on an RDD (e.g. first)

```python
lines = sc.textFile("text_path")
firstLine = lines.first()
```

Spark RDD general workflow:
- Generate initial RDDs from data source.
- Apply transformations (like fliters) to get subset.
- Launch actions to do computation of the subset.
# Updating in progress....