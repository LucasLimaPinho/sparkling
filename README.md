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

# Resilient Distributed Datasets (RDD)

There are times when higher-level manipulation with Structured APIs will not meet the business or engineering problems that you are trying to solve. For those cases, you might need Spark's lower-level APIs, specifically the Resilient Distributed Dataset (RDD), the SparkContext and distributed shared variables like accumulators and broadcast variables. 

You should generally use the lower-level APIs in three situations:

* You need some functionality that you cannot find in the higher-level APIs; for example, if you need very tight control over physical data placement across the cluster;

* You need to maintain some legacy codebase written using RDDs;

* You need to do some custom shared variable manipulation.

All Spark workloads compile down to these fundamental primitives (lower level APIs - RDD and shared variables). When you're calling a DataFrame transformation, it actually just becomes a set of RDD transformations. These tools give you more fine-grained control at the expense of safeguarding you from shooting yourself in the foot!

A SparkContext is the entry point for low-level API functionality. You access it through the SparkSession, which is the tool you use to performe computation across a Spark cluster. 

### About RDDs

Every Spark code you write compiles down to RDD. The Spark UI also describes job execution in terms of RDDs. 

An RDD represents an immutable, partitioned collection of records that can be operated on in parallel. Unlike DataFrames though, where each record is a structured row containining fields with a known schema, in RDDs the records are just Java, Scala or Python objects of the programmer's choosing. 

RDDs give you complete control because every record in an RDD is a just a Java or Python object. You can store anything you want in these objects, in any format you want. 

If you look through Spark's API documentation, you will notice that there are lots of subclasses of RDD. For the most part, these are internal representations that the DataFrame API uses to create optimized physical execution plans. As a user, however, you will likely only be creating two types of RDDs: the "generic" RDD type or a key-value RDD that provides additional functions, such as aggregating by key.

Internally, each RDD is characterized by five main properties:

* A list of partitions

* A function for computing each split

* A list of dependencies on other RDDs

* Optionally, a Partitioner for key-value RDDs (e.g, to say that the RDD is hash-partitioned)

* Optionally, a list of preferred locations on which to compute each split

The Partitioner is problably one of the core reasons why you might want to use RDDs in your code. Specifying your own custom Partitioner can give you significant performance and stability improvements if you use correctly. 

There is no concept of "rows" in RDDs; individual records are just raw Scala/Java/Python objects, and you manipulate those manually instead of tapping into the repository of funcions that you have on Structured APIs.

Python can lose a substantial amount of performance using RDDs. Running Python RDDs equates to running Python user-defined functions (UDF) row by row. We serialize the data to the Python process, operate on it in Python, and then serialize it back to the Java Virtual Machine (JVM). THis causes a high overhead for Python RDD manipulations. Even though many people ran production code with them in the past, we recommend building on the Structured APIs in Python and only dropping down to RDDs if is absolutely necessary.

For the vast majority of use cases, DataFrames will be more efficient, more stable and more expressive than RDDs. The most likely reason for why you'll want to use RDDs is because you need fine-grained control over the physical distribution of data (custom partitioning of data).

With RDDs, you have control over how data is exactly physically distributed across the cluster. Some of these methods are basically the same from what we have in the Structured API but the key addition is the ability to specify a partitioning function.

__Coalesce__ effectively collapses partitions on the same worker in order to avoid a shuffle of the data when repartitioning. 

__Repartition__ operation allows you to repartition your data up or down but performs a shuffle across nodes in the process. Increasing the number of partitions can increase the level of parallelism when operating in map and filter-type operations.

### Custom Partitioning ( the main reason to use RDD lower-level API )

Spark has two built-in Partitioners that you can leverage off in the RDD API - a Hash Partitioner for discrete values and a Range Partitioner. These two work for discrete values and continuous values, respectively. 

# Distributed Shared Variables - Lower Level API

1. Broadcast Variables: let you save a large value on all the worker nodes and reuse it across many Spark actions without re-sending it to the cluster.

2. Accumulators: let you add together data from all the tasks into a shared result.

These are variables you can use in your user-defined functions that have special properties when running on a cluster. 

#### Broadcast Variables

Broadcast variables are a way you can share an immutable value efficiently around the cluster without encapsulating that variable in a function closure. When you use a variable in a closure, it must be deserialized on the worker nodes many times (one per task). Moreover, if tou use the same variable multiple Spark actions and jobs, it will be re-sent to the workers with every job instead of once.

Broadcast variables are shared, immutable variables that are cached on every machine in the cluster instead of serialized with every single task. The canonical use is to pass around a large LOOKUP table that fits in memory on the executors and use that in a function.

#### Accumulators

Accumulators are a way of updating a value inside of a variety of transformatons and propagating that value to the driver node in an efficient and fault-tolerant way.

Accumulators provide a mutable variable that Spark cluster can safely update on a per-row basis. You can use these for debugging purposes (say to track the values of a certain variable per partition in order to intelligently use it over time) or to create a low-level aggregation. 

For accumulator updates performed inside actions only, Spark guarantees that each taks's update to the accumulator will be applied only once, meaning that restarted tasks will not update the value. In transformations, you should be aware that each task's update can be applied more than once if tasks or job stages are reexecuted.

# Spark Streaming

Much like the Resilient Distributed Datasets (RDD) API, however, the DStreams API is based on relatively low-level operations on Java/Python objects that limit opportunities for higher-level optimization. Thus, in 2016, the Spark Project added **Structured Streaming**, a new Streaming API built directly on DataFrames that supports both rich optimizations and significantly simpler integration with other DataFrame and DataSet code. The Structured Streaming API was marked as stable in Apache Spark 2.2, and has also seen swift adoption throughout the Spark community.

Although streaming and batch processing sound different, in practice, they often need to work together. For example, streaming applications often need to join input data against a dataset written periodically by a batch job and the output of streaming jobs is often files or tables that are queried in batch jobs.

Spark batch jobs are often used for Extract, Transform * Load (ETL) workloads that turn raw data into a structured format like Parquet to enable efficient queries. Using Structured Streaming, these jobs can incorporate new data whithin seconds, enabling users to query it faster downstream. 

#### Continuous versus Micro-Batch Execution

In continuous processing-based systems, each node in the system is continually listening to messages from other nodes and outputting new updates to its child nodes. For example, suppose that your application implements a map-reduce computation over several input streams. In a continuous processing system, each of the nodes implementing map would read records one by one from an input source, compute its function on them, and send them to the appropriate reducer. The reducer would then update its state whenever it gets a new record. Continuous processing systems generally have lower maximum throughput, because they incur a significant amount of overhead per-record.

In contrast, micro-batch systems wait to accumulate small batches of input data then process each batch in parallel using a distributed collection of tasks, similar to the execution of a batch job in Spark. Micro-batch systems can ofter achieve high throughput per node because they leverage the same optimizations as batch systems and do not incur any extra per-record overhead.

In practice, the streaming applications that are large-scale enough to need to distribute their computation tend to prioritize throughput, so Spark has traditionally implemented micro-batch processing. In Structured Streaming, however, there is an active development effort to also support a continuous processing mode beneath the same API.

The DStream API in Spark Streaming is purely micro-batch oriented (Structured Streaming API is only available in a stable way in Apache Spark 2.2). It has a declarative (functional-based) API but no support for event time. The Structured Streaming API adds higher-level optimizations, event time and support for continuous processing.

#### The DStream API

It is based purely on Java/Python objects and functions, as opposed to the richer concept of structured tables in DataFrames and DataSets. The API is purely based on processing time - to handle event-time operations, applications need to implement them on their own. Finnaly, DStreams can only operate in a microbatch fashion, and exposes the duration of micro-batches in some parts of its API, making it difficult to support alternative execution modes.

#### Structured Streaming

Structured Streaming is a higher-level streaming API built from the ground up on Spark's Structured API. It is available in all the environments where structured processing runs, including Scala, Java, Python, R and SQL. Like DStreams, it is a declarative API based on high-level operations, but by building on the structured data model introduced in the previous part of the book. Structured Streaming can perform more types of optimizations automatically. However, unlike DStreams, Structured Streaming has native support for event time data.

Structured Streaming is built on the Spark SQL Engine. Rather than introducing a separate API, Structured Streaming uses the existing structured API in Spark (DataFrames, DataSets and SQL), meaning that all operations you are familiar with there are supported. Users express a streaming computation in the same way they'd write a batch compitation on static data. 

The main idea behind Structured Streaming is to treat a stream of data as a tale to which data is continuously appended. A cornerstone of the API is that you should not have to change your query's code when doing batch or stream processing - you should have to specify only whether to run that query in a batch or streaming fashion. 



