---
layout: post
title:  "Spark Thread Safe"
date:   2018-12-26 20:00:00
categories: Programming
tags: Spark Scala
---

* content
{:toc}

> Spark can be configured to run in different modes, local mode, standalone mode and cluster mode. When running in local mode, we need to pay attention to thread safety.


There's thread safe issue in following code. a SimpleDateFormat instance is defined in driver scope, and it's used to convert String to Date in RDD transformation. The transfermation will be executed in parallel by multiple threads, and the SimpleDateFormat instance will be shared by multiple threads. Issue here is that SimpleDateFormat is not thread safe. You'll get wired exceptions. (Note: following code might run successfully if data amount in RDD is small, in which case Spark will probably use one thread to run the task)
```scala
    val conf = new SparkConf().setAppName("Hello World").setMaster("local[4]")
    val sc = new SparkContext(conf)
    val format = new SimpleDateFormat("dd/MM/yyyy")
    ...
    val finalRDD = filtered.rdd.map(row => Row(
        println(s"${Thread.currentThread().getId()}  $format")
        new java.sql.Date(format.parse(row.getAs[String]("Date of Sale")).getTime())
      ...
    ))
```

To fix the issue, we can move the instance creation into the RDD transformation.
```scala
    val conf = new SparkConf().setAppName("Hello World").setMaster("local[4]")
    val sc = new SparkContext(conf)
    ...
    val finalRDD = filtered.rdd.map(row => Row(
      {
        val format = new SimpleDateFormat("dd/MM/yyyy")
        new java.sql.Date(format.parse(row.getAs[String]("Date of Sale")).getTime())
      },
      ...
    ))
```

##### Other Spark Modes
When you define driver variables and want to use them in workers, the variables will be serialized and streamed to the workers. Make sure the instances can be serialized. More details can be found [here](https://medium.com/onzo-tech/serialization-challenges-with-spark-and-scala-a2287cd51c54) 