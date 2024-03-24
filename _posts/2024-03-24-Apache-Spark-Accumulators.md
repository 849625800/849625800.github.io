---
layout: post
title: Apache Spark - Accumulators
date: 2024-03-24 12:40 +0800
last_modified_at: 2024-03-24 14:47 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction

Accumulators are variables that are used for aggregating information across the executors. 

# Practical Example: 2016-stack-overflow-survey-responses.csv

We want to answer the following questions by single pass of the dataset:
- How many records do w ehave in this survery result?
- How many records are missing the salary middle point?
- How many reconrds are form Canada?


```python
import sys
sys.path.insert(0, '.')
from pyspark import SparkContext, SparkConf
from commons.Utils import Utils

if __name__ == "__main__":
    conf = SparkConf().setAppName('StackOverFlowSurvey').setMaster("local[*]")
    sc = SparkContext(conf = conf)

    # initialize 2 accumulators in the driver program
    total = sc.accumulator(0)
    missingSalaryMidPoint = sc.accumulator(0)

    # read dataset and create RDD
    responseRDD = sc.textFile("in/2016-stack-overflow-survey-responses.csv")

    # define a fliter function and update cooresponding accumulator. 
    def filterResponseFromCanada(response):
        splits = Utils.COMMA_DELIMITER.split(response)
        total.add(1) # add total count
        if not splits[14]: 
            missingSalaryMidPoint.add(1) # add the one without salary mid point
        # the boolean value for flitering the records from canada
        return splits[2] == "Canada" 
        
    responseFromCanada = responseRDD.filter(filterResponseFromCanada)
    print("Count of responses from Canada: {}".format(responseFromCanada.count()))
    print("Total count of responses: {}".format(total.value))
    print("Count of responses missing salary middle point: {}" \
        .format(missingSalaryMidPoint.value))
```

Here, by intergrating the accumulator into a fliter function, when flitering the data from Canada, the other result can be calculated at the same time. This successfully answer 3 questions with a single pass of the dataset.

# Notes
- Tasks on worker nodes cannot access the accumulator value, which means the accumulators are write-only variables.

- This property allows accumulator to be implemented efficiently without having to communicate every update.

- It is not a only way to answer those questions. We can achieve the same output by using reduce or reduceByKey operation. The benefit is that we usually prefer a simple way to aggregate values within the process of transforming a RDD, or values that are generated at a differen tscale or granularity than that of the RDD itself.

# Practice

Adding another accumulator to count the bytee processed.

```python
import sys
sys.path.insert(0, '.')
from pyspark import SparkContext, SparkConf
from commons.Utils import Utils

if __name__ == "__main__":
    conf = SparkConf().setAppName('StackOverFlowSurvey').setMaster("local[*]")
    sc = SparkContext(conf = conf)

    total = sc.accumulator(0)
    missingSalaryMidPoint = sc.accumulator(0)

    # Adding a Bytes processed accumulator
    processedBytes = sc.accumulator(0)
    responseRDD = sc.textFile("in/2016-stack-overflow-survey-responses.csv")

    def filterResponseFromCanada(response):
        # update byte process accumulator.
        processedBytes.add(len(response.encode('utf-8')))

        splits = Utils.COMMA_DELIMITER.split(response)
        total.add(1)
        if not splits[14]:
            missingSalaryMidPoint.add(1)
        return splits[2] == "Canada"
    responseFromCanada = responseRDD.filter(filterResponseFromCanada)

    print("Count of responses from Canada: {}".format(responseFromCanada.count()))
    print("Number of bytes processed: {}".format(processedBytes.value))
    print("Total count of responses: {}".format(total.value))
    print("Count of responses missing salary middle point: {}".format(missingSalaryMidPoint.value))

```


# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=29)
