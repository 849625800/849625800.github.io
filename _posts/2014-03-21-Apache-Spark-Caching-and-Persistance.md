---
layout: post
title: Apache Spark - Caching and Persistance
date: 2024-03-21 20:43 +0800
last_modified_at: 2024-03-21 20:43 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction
If we call actions in an RDD for many times and do it naively, the RDDs and all of its dependencies are recomputed at each time we call the action, which can be very expensive, especially for some iterative algorithms.

Calling `persist()` method on the RDD can enable Spark to reuse an RDD in multiple actions. This will keep the first time computed action in memory for multiple uses, which can speed up the action for up to 10 times.

# Example
```python
from pyspark import SparkContext, SparkConf, StorageLevel

if __name__ == "__main__":
    conf = SparkConf().setAppName("persist").setMaster("local[*]")
    sc = SparkContext(conf = conf)

    inputIntegers = [1, 2, 3, 4, 5]
    integerRdd = sc.parallelize(inputIntegers)
    
    integerRdd.persist(StorageLevel.MEMORY_ONLY)
    
    integerRdd.reduce(lambda x, y: x*y)
    
    integerRdd.count()
```

# Storage Level
We can pass a `StorageLevel` object to the persist method to choose the storage type.
Specially, `cache()` is the shortcut for `persist(MEMORY_ONLY)`
```python
RDD.persist(StorageLevel level)

# The following 2 lines are identical
RDD.cache()
RDD.persist(MEMORY_ONLY)
```

- `MEMORY_ONLY`: Store RDD as deserialized Jave objects in the JVM. If the RDD does not fit in memory, some partitions will not be cached and will be recomputed on the fly each time they are needed. This is the default level.

- `MEMORY_AND_DISK`: Store RDD as deserialized Javaobjects in the JVM. If the RDD does not fit in memory, store the partitions that don't fit in memory on disk, and read them form there when they are needed.

- `MEMORY_ONLY_SER` (Java and Scala): Store RDD as serialized Java objects (one byte array per partition). This is generally more space-efficient than deserialized objects, especially when using a fast serializer, but more CPU-intensive to read.

- `MEMORY_AND_DIST_SER` (Java and Scala): Similar to MEMORY_ONLY_SER, but spill partitions that don't fit in memory to disk instead of recomputing them on the fly each time they are needed.

- `DISK_ONLY`: Store the RDD partitions only on disk.

# Which level should we use?

- Different storage level provides different trade-offs between memory usage and CPU efficiency.

- If the memory is sufficient to fit the RDD, `MEMORY_ONLY` is the ideal option, which is the most CPU-efficient option. The operations on the RDDs can be run as fast as possible.

- If the memory is not large enough, we can try `MEMORY_ONLY_SER` to save serialized data to memory. It can save space and still keep the reasonably fast access to the RDD actions we run.

- Don't save to disk unless the functions that computed the datasets are expensive, or they filter a signifivent amount of the data. Otherwise, we can compute the function as fast as reading it from disk.

# What if we attemp to cache too much data to fit in memory?

- Spark will evict old partitions using a Least Recently Used cache policy.
- For the `MEMORY_ONLY` storage level, spark will re-compute these partitions the next time they are needed. 
- For the `MEMORY_AND_DISK` storage level, Spark will write these partitions to disk.
- For In either case, the spark job won't break even if we ask Spark to cache too much data.
- Caching unnecessary data can cause spark to evit useful data and lead to longer re-computation time. We may call the `unpersist()` method to remove them from the cache.

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=4)




    