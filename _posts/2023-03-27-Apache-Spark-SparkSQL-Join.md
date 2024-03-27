---
layout: post
title: Apache Spark - SparkSQL Join
date: 2024-03-27 10:18 +0800
last_modified_at: 2024-03-27 12:32 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction

This post will includes the join operation on Spark SQL. Joining data is an critical operation in many SQL pipeline.

# Spark SQL join Vs. core spark join
Spark SQL supports the same basic join types as core Spark. The benefit of using it is that Spark SQL Catalyst optimizer can do more of the heavy lifting for us to optimize the join performance. Although we also give up some of our control.

For example, Spark SQL can sometimes push down or reorder operations to make the joins more efficient. The downside is that we don't have to controls over the partitioner for DataFrames, so we can't manually avoid shuffles as we did with coreSpark Joins.

# Spark SQL Join Types

The standard SQL join types are supported by Spark SQL and can be specified as the how when performing a join. For example:

```python
def join(self, other, on=None, how=None):
    ...
joined = markerSpace.join(postCode, makerSpace["Postcode"], "left_outer")
```

Spark SQL join types has the following types:
- inner
- outer
- left outer
- right outer
- left semi

Except the `left semi` join type, they all join 2 tables. The difference is they have different actions when handling rows without the same keys in both tables.

`Inner join`: returns the rows with the key that exists in both tables.
`Left outer Join`: join the join the left table with the right table. The key that uniquely exist in the left table will retain in the result and the corrsponding value on the right table would be filled with null.
`Right outer Join`: similar to the left outter join but the condition is reversed.
`Left semi joins`: Returns the key that exist in both table but only have value in the left table. 

# Revisit the maker space postcode example
This time, we use Spark SQL join operation to find the distribution of users on different region using the postcode.

Since the maker space datasorce si the full postcode while the postcode in the postcode dataset source is only the prefix of the post code, we need to join two dataset based on the postcode.

```python
from pyspark.sql import SparkSession, functions as fs

if __name__ == "__main__":
    session = SparkSession.builder.appName("UkMakerSpaces").master("local[*]").getOrCreate()

    makerSpace = session.read.option("header", "true") \
        .csv("in/uk-makerspaces-identifiable-data.csv")

    # replace the 'postcode' column with a modified column
    # Here, we add a " " to the end of the perfix of postcode to aviod mismatching. 
    postCode = session.read.option("header", "true").csv("in/uk-postcode.csv") \
        .withColumn("PostCode", fs.concat_ws("", fs.col("PostCode"), fs.lit(" ")))

    # show the 2 columns of name and postcode from the makerspace dataframe 
    print("=== Print 20 records of makerspace table ===")
    makerSpace.select("Name of makerspace", "Postcode").show()

    # show the 2 columns of the postcode and the region name from the postcode dataframe.
    print("=== Print 20 records of postcode table ===")
    postCode.select("PostCode", "Region").show()

    # Call the join on the makerSpace dataframe
    joined = makerSpace \
    # the postCode df is the column to join with
    # the join expression we use here is the postcode column in makerspace that starts with the postCode perfix.
    # we use left outer join because we want to keep the record in the maker space dataset no matter the post code exists on the postCode dataframe or not. 
        .join(postCode, makerSpace["Postcode"].startswith(postCode["Postcode"]), "left_outer")

    print("=== Group by Region ===")
    # group by rigion and count each number of the region, we will have the historgram
    joined.groupBy("Region").count().show(200)
```

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=35)

