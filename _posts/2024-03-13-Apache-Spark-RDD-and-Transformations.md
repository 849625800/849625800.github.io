---
layout: post
title: Apache Spark RDD and Transformations
date: 2024-03-13 20:43 +0800
last_modified_at: 2024-03-19 20:43 +0800
math: true
tags: [Data Engineering]
toc:  true
---
# Introduction
This blog will cover some basic concepts and practice of Apache Spark. [Github Rrpo]()

# Environment 
- python 3.6
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

# Create RDDs
- Process an existing collection in the program to SparkContext's parallelize method. All the data in the collection will be copied to form a distributed dataset, which can be operated on in parallel.

```python
list_of_integers = list(range(1,10))
rdd_of_list = sc.parallelize(list_of_integers)
```
But this is not practical in real scenario because the dataset must be fully copied to the RAM of a single machine. 

- Load dataset from external storage by `textFile` method on SparkContext to create RDDs.

```python
sc = SparkContext("local", "textfile")
lines = sc.textFile("path to the textfile")
```

The external storage can be a distributed file system such as `Amazon S3` or HDFS. Other data sources such as JDBC, Cassandra, and Elastisearch can be also integrated with Spark.

# Transformation of RDD
Transformation of RDD will return a subset of the given RDD. The most common transformations are `filter` and `map`.

- **filter():** This transformation takes in a function and return an subset RDD that selected by the filter function. It can also be used to clean up the invalid rows of the origin RDD.

- **map():** Map transformation takes in a function and process each element in the original RDD through the function, then return the RDD with the new processed values. It can be use to make HTTP requests to each URL in the input RDD or simply calculate the squre of each number.

```python
URLs = sc.textFile("url text path")
URLs.map(makeHttpRequest)
```

Note: the Input type of the map function is not necessary the same as the input type. For example, the input can be `str` and the output can be `int`:

```python
lines = sc.textFile("text file path")
lengths = lines.map(lambda line: len(line))
```

# Practice!

## 1. Fliter by location

```
Create a Spark program to read the airport data from in/airports.text, find all the airports which are located in United States
and output the airport's name and the city's name to out/airports_in_usa.text.

Each row of the input file contains the following columns:
Airport ID, Name of airport, Main city served by airport, Country where airport is located, IATA/FAA code,
ICAO Code, Latitude, Longitude, Altitude, Timezone, DST, Timezone in Olson format

Sample output:
"Putnam County Airport", "Greencastle"
"Dowagiac Municipal Airport", "Dowagiac"
```

```python
import sys
sys.path.insert(0, '.')
from pyspark import SparkContext, SparkConf
from commons.Utils import Utils

def splitComma(line: str):
    splits = Utils.COMMA_DELIMITER.split(line)
    return "{}, {}".format(splits[1], splits[2])

if __name__ == "__main__":
    conf = SparkConf().setAppName("airports").setMaster("local[*]")
    sc = SparkContext(conf = conf)

    airports = sc.textFile("in/airports.text")
    airportsInUSA = airports.filter(lambda line : Utils.COMMA_DELIMITER.split(line)[3] == "\"United States\"")

    airportsNameAndCityNames = airportsInUSA.map(splitComma)
    airportsNameAndCityNames.saveAsTextFile("out/airports_in_usa.text")
```

## 2. Fliter by latitude 
```
Create a Spark program to read the airport data from in/airports.text,  find all the airports whose latitude are bigger than 40.
Then output the airport's name and the airport's latitude to out/airports_by_latitude.text.

Each row of the input file contains the following columns:
Airport ID, Name of airport, Main city served by airport, Country where airport is located, IATA/FAA code,
ICAO Code, Latitude, Longitude, Altitude, Timezone, DST, Timezone in Olson format

Sample output:
"St Anthony", 51.391944
"Tofino", 49.082222
```

```python
import sys
sys.path.insert(0, '.')
from pyspark import SparkContext, SparkConf
from commons.Utils import Utils

def splitComma(line: str):
    splits = Utils.COMMA_DELIMITER.split(line)
    return "{}, {}".format(splits[1], splits[6])

if __name__ == "__main__":
    conf = SparkConf().setAppName("airports").setMaster("local[*]")
    sc = SparkContext(conf = conf)
    
    airports = sc.textFile("in/airports.text")

    airportsInUSA = airports.filter(lambda line: float(Utils.COMMA_DELIMITER.split(line)[6]) > 40)
    
    airportsNameAndCityNames = airportsInUSA.map(splitComma)

    airportsNameAndCityNames.saveAsTextFile("out/airports_by_latitude.text")
```

# Special mapping: flatMap()

`flatMap()` will return a flattened results after mapping.

-  If one row of the target RDD is not the combination of many rows in the origin RDD, we should use `Map()`, otherwise, we can use `flatMap()` as a shortcut.

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=4)