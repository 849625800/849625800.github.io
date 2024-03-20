---
layout: post
title: Apache Spark - Set Operations
date: 2024-03-13 20:43 +0800
last_modified_at: 2024-03-20 20:43 +0800
math: true
tags: [Data Engineering]
toc:  true
---

# Introduction
This blog post will focus on one type of RDD transformations: Set Operations

# Set Operations
Spark support many operation on mathmatical sets.

## For One RDD

- `sample()`: create a random sample from RDD.
    ```python
    def sample(self, withReplacement, fraction, seed=None):
        ...
    ```
    - `withReplacement`: sample with replacement or not.
    - `fraction`: the fraction (sampled size to total size) of the sampling.

- `distinct()`: Return a new RDD containing the distinct elements in this RDD.
    ```python
    def distinct(self, numPartitions=None):
        ...
    ```
    Note: this operation is quite expensive because it must go through all the elements to identify the distinct samples. Use it with care.

## For two RDDs
    All of the following operation requires the two RDDs have the same type.

- `union()`: Return the union of this RDD and another one.
    ```python
    def union(self, other):
        ...
    ```
    It will return an RDD consisting of the data from both input RDDs.
    **Note:** If there are duplicate samples in both input RDDs, the return RDD will have those duplicate elements.

- `intersection()`: Return the intersection of this RDD and other one. 
    ```python
    def intersection(self, other):
        ...
    ```
    **Note:** This operation performs a shuffle internally, which means this operation is pretty expensive. It **will not** contain any duplicate elements in the return RDD if there are any.


- `subtract()`: Return each value that is not contained in the other RDD.
    ```python
    def subtract(self, other, numPartitions=None):
        ...
    ```
    - It only contains elements present in the first RDD not in the second RDD.
    - It is useful when we want to remove some elements from the existing RDD.
    - It require shuffling internally. The operation is expensive.

- `cartesian()`: Return the Cartesian product of this RDD and another one. 
    ```python
    def cartesian(self, other):
        ...
    ```
    - Returns all possible pairs of a and b where a is in the source RDD and b is in the other RDD.
    - It can be use to compare the similarity between all possible pairs.


# Practice!

## Problem 1:

```
"in/nasa_19950701.tsv" file contains 10000 log lines from one of NASA's apache server for July 1st, 1995.
"in/nasa_19950801.tsv" file contains 10000 log lines for August 1st, 1995
Create a Spark program to generate a new RDD which contains the log lines from both July 1st and August 1st,
take a 0.1 sample of those log lines and save it to "out/sample_nasa_logs.tsv" file.

Keep in mind, that the original log files contains the following header lines.
host    logname    time    method    url    response    bytes

Make sure the head lines are removed in the resulting RDD.
```

```python
from pyspark import SparkContext, SparkConf

def isNotHeader(line: str):
    return not (line.startswith("host") and "bytes" in line)

if __name__ == "__main__":
    conf = SparkConf().setAppName("unionLogs").setMaster("local[*]")
    sc = SparkContext(conf = conf)

    julyFirstLogs = sc.textFile("data/nasa_19950701.tsv")
    augustFirstLogs = sc.textFile("data/nasa_19950801.tsv")

    aggregatedLogLines = julyFirstLogs.union(augustFirstLogs)

    cleanLogLines = aggregatedLogLines.filter(isNotHeader)
    sample = cleanLogLines.sample(withReplacement = True, fraction = 0.1)

    sample.saveAsTextFile("out/sample_nasa_logs.csv")
```

## Problem 2:

```
"in/nasa_19950701.tsv" file contains 10000 log lines from one of NASA's apache server for July 1st, 1995.
"in/nasa_19950801.tsv" file contains 10000 log lines for August 1st, 1995
Create a Spark program to generate a new RDD which contains the hosts which are accessed on BOTH days.
Save the resulting RDD to "out/nasa_logs_same_hosts.csv" file.

Example output:
vagrant.vf.mmc.com
www-a1.proxy.aol.com
.....    

Keep in mind, that the original log files contains the following header lines.
host    logname    time    method    url    response    bytes

Make sure the head lines are removed in the resulting RDD.

```

```python
from pyspark import SparkContext, SparkConf

if __name__ == "__main__":
    conf = SparkConf().setAppName("sameHosts").setMaster("local[1]")
    sc = SparkContext(conf = conf)

    julyFirstLogs = sc.textFile("data/nasa_19950701.tsv")
    augustFirstLogs = sc.textFile("data/nasa_19950801.tsv")
    
    julyFirstHosts = julyFirstLogs.map(lambda line: line.split("\t")[0])
    augustFirstHosts = augustFirstLogs.map(lambda line: line.split("\t")[0])
    
    intersection = julyFirstHosts.intersection(augustFirstHosts)
    
    cleanedHostIntersection = intersection.filter(lambda host: host != "host")
    cleanedHostIntersection.saveAsTextFile("out/nasa_logs_same_hosts.csv")
```

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=4)