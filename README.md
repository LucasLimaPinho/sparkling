# Spark - The Definitive Guide

Higher Level: Structured Streaming, Advanced Analytics & Libraries & Ecosystem

Strucutured APIs: Datasets, Dataframes, SQL

Low-Level APIs: RDDs, Distributed Variables

Spark handles loading data from storage systems and performing computation on it, not permanent storage as the end itself. 

Spark manages and coordinates the execution of tasks on data across a cluster of computers.

The cluster of machines that Spark will use to execute tasks is managed by a cluster manager like Spark's standalone cluster manager, YARN or Mesos. 

The driver process runs your main() function, sits on a node in the cluster and is responsible for three things: maintaning information about the Spark Application, responding to user's program or input and analyzing, distributing and scheduling work across the executors.

The executor is responsible for only two things: executing code assigned to it by the driver and reporting the state of the computation on that executor back to the driver node. 

The driver and executors are simply processes, which means that they can live on the same machine or different machines. In local mode, the driver and executors run (as threads) on your inidivdual computer instead of a cluster. 

The executors, for the most part, will always be running **Spark code**. However, the driver can be "driven" from a number of differente languages through Spark's language APIs. 

When using Spark from Python or R, you don't write explicit JVM instructions; instead, you write Python and R code that Spark translates into code that it then can run on the executor JVMs.

# SparkSession

You control your Spark Application through a driver process called the SparkSession. The SParkSession instance is the way Spark executes user-definied manipulations across the cluster. There is a one-to-one correspondence between a SparkSession and a Spark Application. 

# Dataframe

Dataframe is the monst common Strutured API and simply represents a table of data with rows and columns. The list that defines the columns and the types within those columns is called schema. You can think of a Datagrame as a spreadsheet with named columns.

To allow every executor to perform work in parallel, Spark breaks up the data into chunks called partitions. A partition is a collection of rows that sit on one physical machine in your cluster. If you have ine partition, Spark will have a parallelism of only one, even if you have thousand of executors. If you have many partitions but only one executor, Spark still have a parallelism of only because there is only one computation resource.

**Narrow Transformations**: are those for which each input partition will contribute to only one output partition. (1 --> 1).

**Wide Transformations**: will have input partitions contributing to many output partitions. Shuffle: Spark will exchange partitions across the cluster. When we perform a shuffle, Spark writes the results to disk (1 --> N).

**Lazy Evaluation**: Spark will wait until the very last moment to execute the grapg of computation instructions. 

Transformation allow us to build up our logical transformation plan. To trigger the computation, we run an action. Reading data is a transformation, and is therefore a lazy operation. Spark peeks at only a couple of rows of data to try to guess what types each column should be.

**By default, when we perform a shuffle, Spark outputs 200 shuffle partitions**.

~~~python

spark.conf.set("spark.sql.shuffle.partition", "5")

~~~

