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

Dataframe is the most common Strutured API and simply represents a table of data with rows and columns. The list that defines the columns and the types within those columns is called schema. You can think of a Datagrame as a spreadsheet with named columns.

DataFrames are a distributed collection of objects of type Row that can hold various types of tabular data.

To allow every executor to perform work in parallel, Spark breaks up the data into chunks called partitions. A partition is a collection of rows that sit on one physical machine in your cluster. If you have ine partition, Spark will have a parallelism of only one, even if you have thousand of executors. If you have many partitions but only one executor, Spark still have a parallelism of only because there is only one computation resource.

**Narrow Transformations**: are those for which each input partition will contribute to only one output partition. (1 --> 1).

**Wide Transformations**: will have input partitions contributing to many output partitions. Shuffle: Spark will exchange partitions across the cluster. When we perform a shuffle, Spark writes the results to disk (1 --> N).

**Lazy Evaluation**: Spark will wait until the very last moment to execute the grapg of computation instructions. 

Transformation allow us to build up our logical transformation plan. To trigger the computation, we run an action. Reading data is a transformation, and is therefore a lazy operation. Spark peeks at only a couple of rows of data to try to guess what types each column should be.

**By default, when we perform a shuffle, Spark outputs 200 shuffle partitions**.

~~~python

spark.conf.set("spark.sql.shuffle.partition", "5")

~~~

You can express your business logic in SQL or Dataframes (either in R, Python, Scala, or Java) and Spark will compile that logic down to an underlying plan before actually executing the code. With Spark SQL, you can register anu Dataframe as a table or view and query it using pure SQL. There is no performance difference between writing SQL queries or writing DataFrame code, they both "compile" to the same underlying plan.

Upon submission, the application will run until it exits (completes the task) or encounters an error. 

# Datasets - Type-Safe Structured APIs

The Dataset API is not available in Python and R because those languages are dynamically typed.

When we use User Defined Functions (UDF), Spark will serialize the function on the driver and transfer it over the network to all executors processes. This happens regardless of language.

If the function is written in Python, Spark starts a Python process on the worker, serializes all of the data to a format that Python can understand, executes the function row by row on that data in the Python process, and then finally returns the results of the row operations to the JVM and Spark. Starting this Python process is expensive, but the real cost is in serializing the data to Python.

THis is costly for two reasons: it is an expensive computation, but also, after the data enters Python, Spark cannot manage the memory of the worker. This means that you could potentially cause a worker to failt if it becomes resource constrained (because both the JVM and Python are competing for memory on the same machine). We recommend that you write your UDFs in Scala or Java - the small amount of time it should take you to write the function in Scala will always yeld significant speed ups, and on top of that, you can still use the function from Python!

