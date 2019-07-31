# Spark/Scala notes

_Resilient Distributed Dataset (RDD)_ - abstraction of a giant set of data
- Fault tolerant (resillient)

_Spark Context (sc)_ - created by driver program and is responsible for making RDD's resilient and distributed.
- spark-shell creates sc for you, if writing a program need to create explicitly.
- sc.textFile("...") use the necessary prefix to specify source i.e hdfs://, s3n://

_Using hive: hiveCtx = HiveContext(sc)_

_Transforming RDD's_
- Map, flatmap, filter, distinct, sample
- Union, intersection, subtract, cartesian
  - val rdd = sc.parallelize(List(1, 2, 3 ,4))
  - val squares  = rdd.map(x => x * x)
    - squares would contain [1, 4, 9, 16]
  - You can define a function outside of the map call, and pass it when used
    - def square(x: Int): Int = { return x*x } <br/> rdd.map(square)

_Actions on RDD's_
- collect, count, countByValue, take, top, reduce, ....
- Nothing actually happens in your driver program until an action is called
_Spark Internals_
- Jobs are broken into stages based on when data needs to be reorganized
- Stages are broken up into tasks
- Start with execution plan &rarr; stages (based on things that can be processed together in parallel (don't need shuffle)) &rarr; stages are broken into tasks that are distributed to individual nodes in your cluster. Rest is up to YARN.

_Key/Value RDDs_
- reduceByKey: combine values with the same key using some function. rdd.reduceByKey((x,y) => x + y)
  - in that example we take value x from tuple 1, value y from tuple 2 with both sharing same key and add them together.
- groupByKey()
- sortByKey(): sort an RDD by its key values
- keys(), values(): create RDDs of just k | v
- SQL like functions such as joins, cogroup, subtractByKey, etc...
- When dealing with key/value data, use flatmapValues() and mapValues() if the transformations do not affect the keys, it is more efficient

___EXAMPLE___

Input data:

|ID|name|age|# of friends|
|---|---|---|---|
|0|Will|33|385|
|1|Jean-Luc|33|2|
|2|Hugh|55|221|
|3|Deanna|40|465|
|4|Quark|68|21|

_Approach_

- val totalsByAge = rdd.mapValues(x => (x, 1)).reduceByKey((x,y) => (x._1 + y._1, x._2 + y._2))
  - Breakdown
    - mapValues(x => (x, 1)) results in :
    |Key|Value|
    |---|---|
    |33|(385,1)|
    |33|(2,1)|
    |55|(221,1)|
    |40|(465,1)|
    |68|(21,1)|
    - reduceByKey((x,y) => (x._1 + y._1, x._2 + y._2))results in :
    |Key|Value|
    |---|---|
    |33|(387,2)|
    |55|(221,1)|
    |40|(465,1)|
    |68|(21,1)|
