---
layout: post
title: Apache Spark - Broadcast Variables
date: 2024-03-25 12:40 +0800
last_modified_at: 2024-03-25 14:47 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction
Broadcast Variables are variables we want to share through out clusters. It allows the programmer to keep a read-only variable cached on each machine rather thatn shipping a copy of it with tasks. 

They can be used to give every node a copy of a large input dataset. in an efficient manner.

All broadcast variables will be kept at all the worker nodes for use in one or more Spark operations.

# Example on makerspaces dataset
This dataset includes the name, email adress postcode, number of visiters, etc.

We want to answer how are those maker spaces distributed across different regions in the UK, however, we only have the postcode in each space. We can use the `uk-postcode` dataset to find out the space given postcode.

**Solution:**
- load the postcode dataset and broadcast it across the cluster.
- load the maker space datset and call map operation on the maker space RDD to look up the region using the postcode of the maker space. 

```python
import sys
sys.path.insert(0, '.')
from pyspark import SparkContext, SparkConf
from commons.Utils import Utils

def loadPostCodeMap():
    lines = open("in/uk-postcode.csv", "r").read().split("\n")
    splitsForLines = [Utils.COMMA_DELIMITER.split(line) for line in lines if line != ""]
    return {splits[0]: splits[7] for splits in splitsForLines}

def getPostPrefix(line: str):
    # The post code in the makerspace is the full post cost (e.g. W1T 3AC)
    # while the postcode in the postcode dataset is only the prefix (e.g. W1T)
    splits = Utils.COMMA_DELIMITER.split(line)
    postcode = splits[4]
    return None if not postcode else postcode.split(" ")[0]

if __name__ == "__main__":
    conf = SparkConf().setAppName('UkMakerSpaces').setMaster("local[*]")
    sc = SparkContext(conf = conf)

    # read the postcode RDD and broadcast it to all the clusters.
    postCodeMap = sc.broadcast(loadPostCodeMap())

    # load makerspace dataset as string RDD.
    makerSpaceRdd = sc.textFile("in/uk-makerspaces-identifiable-data.csv")

    regions = makerSpaceRdd \
      .filter(lambda line: Utils.COMMA_DELIMITER.split(line)[0] != "Timestamp") \
      .filter(lambda line: getPostPrefix(line) is not None) \
      .map(lambda line: postCodeMap.value[getPostPrefix(line)] \ 
        if getPostPrefix(line) in postCodeMap.value else "Unknow")
    #If there is a match of the prefix, return the region, else return unknow

    for region, count in regions.countByValue().items():
        print("{} : {}".format(region, count))
```

# Procedures of using Broadcast Variables

- Creat a Broadcast variable T by calling SparkContext.boardcast() on an object of type T. The Broadcast Variable can be any type as long as it's serializable which enable the data pass across the wire to the spark clusters.

- The variable will be sent to each node only once and should be treated as read-only, meaning updates will not be propagated to other nodes.

- The value of the broadcast can be accessed by calling the value method in each node.

# Solve this problem without broadcast.


```python
import sys
sys.path.insert(0, '.')
from pyspark import SparkContext, SparkConf
from commons.Utils import Utils

def loadPostCodeMap():
    lines = open("in/uk-postcode.csv", "r").read().split("\n")
    splitsForLines = [Utils.COMMA_DELIMITER.split(line) for line in lines if line != ""]
    return {splits[0]: splits[7] for splits in splitsForLines}

def getPostPrefix(line: str):
    splits = Utils.COMMA_DELIMITER.split(line)
    postcode = splits[4]
    return None if not postcode else postcode.split(" ")[0]

if __name__ == "__main__":
    conf = SparkConf().setAppName('UkMakerSpaces').setMaster("local[*]")
    sc = SparkContext(conf = conf)

    # Not using broadcast
    postCodeMap = loadPostCodeMap()
    makerSpaceRdd = sc.textFile("in/uk-makerspaces-identifiable-data.csv")

    regions = makerSpaceRdd \
      .filter(lambda line: Utils.COMMA_DELIMITER.split(line)[0] != "Timestamp") \
      .map(lambda line: postCodeMap[getPostPrefix(line)] \
        if getPostPrefix(line) in postCodeMap else "Unknow")

    for region, count in regions.countByValue().items():
        print("{} : {}".format(region, count))

```

**Problem of not using broadcast:**
- Spark will automatically sent all variables referenced in out closures to the worker nodes. While this is convenient, it can also be inefficient because the default task launching mechanism is optimized for small task sizes.
- Because we can potentially use the same variable in multiple parallel operations. If spark send it seperately for each operation, it can lead to some performance issur if the size of data to transfer is too large. If we use the same postcode dictionary later, the same postcode dictionary would be sent again to each node.

**Example:** If the dictionary is adress to region instead of postcode to region, the table will be much larger., which makes the data tranportation expensive from the master to each task. Making the dictionary a broadcast variable will fix the problem. 

The broadcast variable is specifically useful if the application need to send a read only look-up table to all tasks, or it can also be a large feature vector in a machine learning algorithm.

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=31)