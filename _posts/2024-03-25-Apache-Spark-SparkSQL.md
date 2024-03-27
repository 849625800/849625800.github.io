---
layout: post
title: Apache Spark - SparkSQL Introduction
date: 2024-03-26 12:40 +0800
last_modified_at: 2024-03-26 14:47 +0800
math: true
tags: [Spark]
toc:  true
---

# Introduction
SparkSQL is a interface on Spark that working with structure and semistructure data.

# Structured data

Structured data is any data that has a schema meaning that a known set of fields for each record. Spark SQL provides a dataset abstraction that simplifies working with structured datsets. Dataset is similar to tables in a relational database. More and more Spark workflow is moving towards Spark SQL. The whole point of SparkSQL and its related technology is dealing with structure data. 

Dataset that has a natural schema lets Spark store data in a more efficient manner and can run SQL queries on it using actual SQL commands.

# Important concepts: DataFrame and Dataset

## **`DataFrame`**: 

- Spark SQL published a tabular data abstraction called DataFrame since v1.3. A Dataframe is a dataabstraction or a domain-specific language for working with structured and semi-structured data. It can store data in a more efficient manner than native RDDs, taking advantage of their schema. 

- It uses the immutable, in-memory, resilient, distributed and parallel capabilities of RDD, and applies a structure called schema to the data, allowing Spark to manage the schema and only pass data between nodes, in a much more efficient way using Java serialization.

- Unlike an RDD, data is organized into named columns, like a table in a relational dataset. Also, it provides new operations not available on RDDs, such as the ability to run SQL queries.

## **Dataset**:

- The Dataset API released since Spark 1.6. it provides the familiar object-oriented programming style, compile-time safety of the RDD API and the benefits of leveraging schema to work with structured data.

- A dataset is a set of structured data, not necessary a row but it could be aof a particular type.

- Java and Spark will know the type of the data in a dataset at complile time.

- Because of its nature, the Dataset API is not available in Python.

# DataFrame and Dataset

Since Spark 2.0, DataFrame APIs merge with Dataset APIs.
- Dataset takes on two distinct APIs characteristics: a strongly-typed API and an untyped API.

- Consider DataFrame as untyped view of a Dataset, which is a Dataset of Row where a Row is a generic untyped JVM object.

- Dataset, by contrast, is a collection of strongly-typed JVM objects.

- The Dataset API is only available on Java and Scala.

- For Python we stick with the DataFrame API.

# Reference:

Thanks for the amazing tutorial by Youtuber [Analytics Excellence](https://www.youtube.com/watch?v=W__Jk83gOyo&list=PL0hSJrxggIQr6wA8buIn1Yxu810ugGed-&index=32)