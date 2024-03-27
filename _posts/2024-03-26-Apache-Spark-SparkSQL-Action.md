---
layout: post
title: Apache Spark - SparkSQL Action
date: 2024-03-26 12:40 +0800
last_modified_at: 2024-03-26 14:47 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction
The most powerful way to use Spark SQL is to intergrate it into a spark application program, in which we not only can run SQL code to make a query to dataframe, but also can combine the SQL query with other implementations. 

In this experiment, we will look into the `2016-stack-overflow-survey-responses.csv` dataset. We will focus on the `country`, `age_mid`, `occupation`, `salary_midpoint` columns.

# Experiments & Explain
This session will walk through a demo on the dataset using SparkSQL. 

## The complete code of this experiment

```python
from pyspark.sql import SparkSession

AGE_MIDPOINT = "age_midpoint"
SALARY_MIDPOINT = "salary_midpoint"
SALARY_MIDPOINT_BUCKET = "salary_midpoint_bucket"

if __name__ == "__main__":

    session = SparkSession.builder.appName("StackOverFlowSurvey").getOrCreate()

    dataFrameReader = session.read

    responses = dataFrameReader \
        .option("header", "true") \
        .option("inferSchema", value = True) \
        .csv("in/2016-stack-overflow-survey-responses.csv")

    print("=== Print out schema ===")
    responses.printSchema()
    
    responseWithSelectedColumns = responses.select("country", "occupation", 
        AGE_MIDPOINT, SALARY_MIDPOINT)

    print("=== Print the selected columns of the table ===")
    responseWithSelectedColumns.show()

    print("=== Print records where the response is from Afghanistan ===")
    responseWithSelectedColumns\
        .filter(responseWithSelectedColumns["country"] == "Afghanistan").show()

    print("=== Print the count of occupations ===")
    groupedData = responseWithSelectedColumns.groupBy("occupation")
    groupedData.count().show()

    print("=== Print records with average mid age less than 20 ===")
    responseWithSelectedColumns\
        .filter(responseWithSelectedColumns[AGE_MIDPOINT] < 20).show()

    print("=== Print the result by salary middle point in descending order ===")
    responseWithSelectedColumns\
        .orderBy(responseWithSelectedColumns[SALARY_MIDPOINT], ascending = False).show()

    print("=== Group by country and aggregate by average salary middle point ===")
    dataGroupByCountry = responseWithSelectedColumns.groupBy("country")
    dataGroupByCountry.avg(SALARY_MIDPOINT).show()

    responseWithSalaryBucket = responses.withColumn(SALARY_MIDPOINT_BUCKET,
        ((responses[SALARY_MIDPOINT]/20000).cast("integer")*20000))

    print("=== With salary bucket column ===")
    responseWithSalaryBucket.select(SALARY_MIDPOINT, SALARY_MIDPOINT_BUCKET).show()

    print("=== Group by salary bucket ===")
    responseWithSalaryBucket \
        .groupBy(SALARY_MIDPOINT_BUCKET) \
        .count() \
        .orderBy(SALARY_MIDPOINT_BUCKET) \
        .show()

    session.stop()
```
## Walk through the code

When working with RDD before, we usually using spark context. Now we need to create a spark session for Spark SQL, which create a single point that interacts with underlining Spark funtionality with SQL style syntax and allows us to work with Dataframe instead of RDD. It internally has a spark context to do spark computation. It is the very first object we have to create on a Spark application.

```python
session = SparkSession.builder.appName("StackOverFlowSurvey").getOrCreate()
```

Spark session use a builder factury design pattern. Here we specified an appName and run on default master `local[*]` which is the local model on all available threads. Then, the `getOrCreate()` method will create the spark session or get the existing one if one already has been created.

```python
dataFrameReader = session.read

responses = dataFrameReader \
    .option("header", "true") \
    .option("inferSchema", value = True) \
    .csv("in/2016-stack-overflow-survey-responses.csv")
```
`dataFrameReader` object is a spark interface used to load the data from the external storage system. Then we can specifiy the options of the reader.

- We set the `header` to true, which means the dataset has a header on the first line, so that the reader will skip the first line.

- We specified `inferSchema` to true, which save us many time from set up schema manually.

- The reader class have many useful APIs for reading different types of files including `csv`, `jebc`, `json`, `parquet` ect. Here, our input file is `csv`. It return a `Dataframe` data. 

```python
print("=== Print out schema ===")
responses.printSchema()
```
The Schema can be printed using the `printSchema()` method in the returned Dataframe. It will include the information of the column names and the datatypes of each column. The `inferSchma` setting will infer the data types such as `string` or `double` based on the actural data, and all allow null values.

Now, we have a SQL database sitting on the memory of our Spark application distributed potentially on the cluster. We can treat the dataframe as a regular SQL database, which allows us to use SQL syntax to quary the dataframe.

For example, if we want to extract 4 different columns from the dataframe, which are `country`, `occupation`, `AGE_MIDPOINT`, `SALARY_MIDPOINT`.

```python
responseWithSelectedColumns = responses.select("country", "occupation", 
    AGE_MIDPOINT, SALARY_MIDPOINT)

print("=== Print the selected columns of the table ===")
responseWithSelectedColumns.show()
```

We can call the `select()` method with a set of column names. It will return a new dataframe with those columns. By calling the `show()` method, we can see what is inside the dataframe. This will show the top 20 rows in the dataframe.

Then, we may want to filter some rows from the new dataframe. Say, select all the data where the `country` is `Afghanistan`, we can use `filter()` method on the dataframe. 

```python
print("=== Print records where the response is from Afghanistan ===")
responseWithSelectedColumns\
    .filter(responseWithSelectedColumns["country"] == "Afghanistan").show()
```

The `filter()` method can take the column object as a condition. To get a column object, we can use a `[]` and a column name `country`. Then we can use the regular python logical operators to set up a condition.

The `groupBy()` operation is alos available on Dataframe, we can group the data on a specified column, so that we can do aggrigation on them. For example, if we want the total count on each occupation:

```python
print("=== Print the count of occupations ===")
groupedData = responseWithSelectedColumns.groupBy("occupation")
groupedData.count().show()
```

After calling the `groupBy()` method, spark returns a `GroupData` object, which provide a set of methods for use to do aggrigation on a dataframe, such as `max`, `min`, `sum`, etc. Here, we use `count()` aggrigation.

We can also setup another condition on the specific column with filter. Such as getting all the data with the `AGE_MIDPOINT` no more than 20:

```python
print("=== Print records with average mid age less than 20 ===")
responseWithSelectedColumns\
    .filter(responseWithSelectedColumns[AGE_MIDPOINT] < 20).show()
```
The `<` operater is also a method for the column object. This will return all the rows wht the age midpoint no more than 20.


Spark could also showing the sorted data by a specific column.

```python
print("=== Print the result by salary middle point in descending order ===")
responseWithSelectedColumns\
    .orderBy(responseWithSelectedColumns[SALARY_MIDPOINT], ascending = False).show()
```

By setting the ascending to false, the result will return all the rows with desending order by salary midpoint.

Next, we could do some aggrigation on the dataframe,such as calculating the average of all the data on a specific column:

```python
print("=== Group by country and aggregate by average salary middle point ===")
dataGroupByCountry = responseWithSelectedColumns.groupBy("country")
dataGroupByCountry.avg(SALARY_MIDPOINT).show()
```

Lastly, we would like to group the data by different buckets based on the salary midpoint. For example, the data with salary middle point within `0 ~ 20000` will be put into the first bucket, within `20000 ~ 40000` will be in the second, and `40000 ~ 60000` will be in the third, ect.

```python
responseWithSalaryBucket = responses.withColumn(SALARY_MIDPOINT_BUCKET,
    ((responses[SALARY_MIDPOINT]/20000).cast("integer")*20000))

print("=== With salary bucket column ===")
responseWithSalaryBucket.select(SALARY_MIDPOINT, SALARY_MIDPOINT_BUCKET).show()

print("=== Group by salary bucket ===")
responseWithSalaryBucket \
    .groupBy(SALARY_MIDPOINT_BUCKET) \
    .count() \
    .orderBy(SALARY_MIDPOINT_BUCKET) \
    .show()
```

- We use the `withColumn()` method to create a new `SALARY_MIDPOINT_BUCKET` column with the value that calculated the cooresponding column bucket value.

- Then print the salary midpoint and midpoint bucket column to check if it is what we want. The ideal result would be the rows like (45000, 40000), (210000, 20000), (5000, 0) etc. The first value is the actural salary midpoint and the second is the bucket value. For example, 4.5K should be in the 40k bucket. 

- Finally, call `groupBy()` in the bucket column and `count()` the values within each bucket, then call `orderby()` to sort the values in ascending order.

At the end, make sure we call the `stop()` method after we have done everything we want in this session. We should do the same in Spark SQL like we do in other dataset connection.

```python
session.stop()
```
# Catalyst Optimizer

- Spark SQL uses an optimizer called Catalyst to optimize all the queries written both in Spark SQL and DataFrame DSL,

- This optimizer makes queries run much faster than their RDD counterparts.

- The Catalyst is a modular library which is bulit as a rule-based system. Each rule in the framework focuses on the specific optimization. For example, rule like ConstantFolding focus on removing constant expression from the query.

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=33)



