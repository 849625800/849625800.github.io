---
layout: post
title: Apache Spark - Actions and aspects of RDD
date: 2024-03-21 20:43 +0800
last_modified_at: 2024-03-21 20:43 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction
Actions are the operations which will return a final value to the driver program or persist data to an external storage system. It will force the evaluation of the transformations required for the RDD they were called on.

# Different Actions

- `Collect()`: Collect operation retrieves the eitire RDD object back to the driver program as a regular collection or value. For example we can retrieve a string RDD back to a list of string. 

    ```python
    from pyspark import SparkContext, SparkConf

    if __name__ == "__main__":
        conf = SparkConf().setAppName("collect").setMaster("local[*]")
        sc = SparkContext(conf = conf)
        
        inputWords = ["spark", "hadoop", "spark", "hive", "pig", "cassandra", "hadoop"]
        
        wordRdd = sc.parallelize(inputWords)
        
        words = wordRdd.collect()
        
        for word in words:
            print(word)
    ```
    **Note:** Be aware that the collect action will dump the whole dataset into the memory of local machine. We must be sure that the memory is large enough to contain the dataset. Usually, we would not use collect in large dataset.

- `count()` and `countbyValue()`: Count action will return how many rows in an RDD, countbyValue action will look at the unique value in each row and return a map of each unique value to its count.

    ```python
    from pyspark import SparkContext, SparkConf

    if __name__ == "__main__":
        conf = SparkConf().setAppName("count").setMaster("local[*]")
        sc = SparkContext(conf = conf)
        
        inputWords = ["spark", "hadoop", "spark", "hive", "pig", "cassandra", "hadoop"]
        
        wordRdd = sc.parallelize(inputWords)
        print("Count: {}".format(wordRdd.count()))
        
        worldCountByValue = wordRdd.countByValue()
        print("CountByValue: ")
        for word, count in worldCountByValue.items():
            print("{} : {}".format(word, count))
    ```

- `take()`: take action takes n elements from an RDD. It is useful when we want to take a peek at the input RDD for unit tests.

    ```python
    from pyspark import SparkContext, SparkConf

    if __name__ == "__main__":
        conf = SparkConf().setAppName("take").setMaster("local[*]")
        sc = SparkContext(conf = conf)
        
        inputWords = ["spark", "hadoop", "spark", "hive", "pig", "cassandra", "hadoop"]
        wordRdd = sc.parallelize(inputWords)
        
        words = wordRdd.take(3)
        for word in words:
            print(word)

    ```

**Note:** take action will try to reduce the number of access to the original RDD, which means it may return a biased collection of data.


- `saveAsTextFile()`: This action will  write data out to a distributed storage system such as HDFS or Amazon S3. It can also be used to save file on local file system.

    ```python
    cleanedHostIntersection.saveAsTextFile("out/nasa_logs_same_hosts.csv")
    ```

- `reduce()`: One of the most common action we use in spark applications. It takes in that operate on two elements of the type in the input RDD and return a new element of the same type. For example, it can be use to calculate the cumulated product of the whole RDD datas. 

    ```python
    from pyspark import SparkContext, SparkConf

    if __name__ == "__main__":
        conf = SparkConf().setAppName("reduce").setMaster("local[*]")
        sc = SparkContext(conf = conf)
    
        inputIntegers = [1, 2, 3, 4, 5]
        integerRdd = sc.parallelize(inputIntegers)

        # product
        product = integerRdd.reduce(lambda x, y: x * y)
        print("product is :{}".format(product))

        # sum
        sum = integerRdd.reduce(lambda x, y: x + y)
        print("sum is : {}".format(sum))
    ```
# Asbspects of RDD
- **RDDs are distributed.**
   Each RDD are devided into multiple partitions, which can be operated on in each node in parallel and independently. This partitions is automatically done by spark, which keeps us aways from the details of the implementation.

- **RDDs are immutable.**
    They cannot be changed after created. This properties rules out a significant set of potential problems due to updatas from multiple threads at the same time.

- **RDDs are resilient**
    RDDs are deterministic function of their input. Combined with immutability, RDDs can be created at any time. If any node in the cluster goes down, Spark can recover the malfunction node from the input and pick up from where it left off. Spark make sure the RDDs are fault tolerant.

# Summary of RDD operations

- **Lazy Evaluation:** RDD only defined when the first time they are used in an Action.

```python
# Nothing would happen when first see textFile()
lines = sc.textFile("...")

# Nothing would happen when first see fliter()
selected_lines = lines.filter(lanbda line: "some condition")

# Spark start loading file when the "first()" action is called
selected_lines.first()

# Here, spark scans the file only until the first line satisfied the condition is detected.
# It would not go through the entire file.
```

- Instead of thinking the RDDs as the containing of specific data, it might be better to think of each RDD as consisting of instructions on how to compute the data that we bulid up through transformations.

- Spark use lazy evaluation to reduce the number of passes to take over our data by grouping operations together.

- Transformation returns RDDs, while actions returns some other datatypes.

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=4)