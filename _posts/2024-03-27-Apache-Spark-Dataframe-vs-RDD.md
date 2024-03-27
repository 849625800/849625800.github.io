---
layout: post
title: Apache Spark - SparkSQL Dataframe Vs. RDD
date: 2024-03-27 17:40 +0800
last_modified_at: 2024-03-27 18:30 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction

Since Spark 1.6, the trand is more to use dataframe instead of RDD. Dataframe becomes the new hotness. So when and where shoule we use Dataframe? How about RDD? This blog will do a comparation between them and provide some insights.

# Comparing
**DataFrame:** MLlib is on a shift to DataFrame based API. Spark steaming is also moving towards, sothing called structured streaming which is heavily based on DataFrame API.

**RDD:** Given those trands, RDD still not being deprecated. The RDDs are still the core and fundamental building block of Spark. Both DataFrames and Datasets are built on top of RDDs.

# Which one to use?

RDD is the primary user-facing API in Spark. At the core, RDD is an immutable distributed collection of elements of the data, partitioned across nodes in the cluster that can be operated in parallel with  a low-level API that offers transformations and actions.

Therefore, we use RDD when:
- We want low-level transformation, actions, and control on our dataset.
- We work with unstructured data, such as media streams or streams of text.
- We need to manipulate our data with functional programming constructs than domain specific expressions.
- We don't nedd the optimization and performance benefits available with DataFrame.

we use DataFrames when:

- We need rich sementics, high-level abstractions, and domain specific APIs.
- Our processing requires aggregation, averages, sum, SQL queries and columnar access on semi-structured data.
- We want the benefit of Catalyst optimization.
- Unification and simplification of APIs across Spark Libraries are needed.


In summary, we should consider suing Dataframes over RDDs if possible because It can optimize SQL queries and provide better performance than old school RDD operations. But RDD will remains to be the one of the most critical core components of Spark, and it is the underlying building block for DataFrames.

# Conversion between DataFrame and RDD

Sometime we need both DataFrame and RDD, we can do the conversion by a simple API call.

```python
import sys
sys.path.insert(0, '.')
from pyspark.sql import SparkSession
from commons.Utils import Utils

def mapResponseRdd(line: str):
    splits = Utils.COMMA_DELIMITER.split(line)
    double1 = None if not splits[6] else float(splits[6])
    double2 = None if not splits[14] else float(splits[14])
    return splits[2], double1, splits[9], double2

def getColNames(line: str):
    splits = Utils.COMMA_DELIMITER.split(line)
    return [splits[2], splits[6], splits[9], splits[14]]

if __name__ == "__main__":

    session = SparkSession.builder.appName("StackOverFlowSurvey").master("local[*]").getOrCreate()
    sc = session.sparkContext

    # load string rdd
    lines = sc.textFile("in/2016-stack-overflow-survey-responses.csv")

    # fliter the header line and map them to a format we need.
    responseRDD = lines \
        .filter(lambda line: not Utils.COMMA_DELIMITER.split(line)[2] == "country") \
        .map(mapResponseRdd)    

    # get column names, which can be used to create dataframe schema
    colNames = lines \
        .filter(lambda line: Utils.COMMA_DELIMITER.split(line)[2] == "country") \
        .map(getColNames)

    # call toDF() method and pass the column names. Then the RDD will be converted to DataFrame.
    responseDataFrame = responseRDD.toDF(colNames.collect()[0])

    print("=== Print out schema ===")
    responseDataFrame.printSchema()

    print("=== Print 20 records of responses table ===")
    responseDataFrame.show(20)

    # converte DF back to RDD.
    for response in responseDataFrame.rdd.take(10):
        print(response)
```

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=37)


