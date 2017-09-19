
One of the main benefits of spark-sql as mentioned in their [sigmod paper](https://people.csail.mit.edu/matei/papers/2015/sigmod_spark_sql.pdf) 
is its ability to easily define and plug in user defined adhoc rules for better optimization. 
Spark-sql provides api for adding set of adhoc rules that can be plugged into the query planner during 
runtime. Especially after the data source api is made independent from the spark core, it is very 
useful for different database/ datasource vendors for adding rules that best suit their datasource 
design. This api for adding adhoc rules at run time is moved from SQLContext to separate 
class called `ExperimentalMethods` in [SPARK-5193](https://issues.apache.org/jira/browse/SPARK-5193).
However this api supported only adding rules for optimizing spark `PhysicalPlan` and didn't offer
support for adding rules for the `OptimizedPlan`. In [SPARK-9843](https://issues.apache.org/jira/browse/SPARK-9843)
this support is added and officially made available to the public since Spark 2.0. If you are not familiar
with the difference between spark `PhycialPlan` and `OptimizedPlan` please refer my previous post [here]()

However, the api provided by the `ExperimentalMethods` for adding additional rules provides option
to apply user provided rule only after all the native rules of spark are applied. This may limit the 
ability to do some advanced optimization, also it limits additional optimization on the user
provided custom rule by taking advantage of the spark's native optimization rules. One such examples 
can be found in my previous post on [SparkOptimizer](https://github.com/sathiyapk/Blog-Posts/blob/master/SparkOptimizer.md).


## Experimental Methods
Let's take a look at the `ExperimentalMethods` class and how user provided optimization rules are 
plugged into `SparkOptimizer` class.

ExperimentalMethods class:
```scala
class ExperimentalMethods private[sql]() {

  @volatile var extraStrategies: Seq[Strategy] = Nil

  @volatile var extraOptimizations: Seq[Rule[LogicalPlan]] = Nil

  override def clone(): ExperimentalMethods = {
    val result = new ExperimentalMethods
    result.extraStrategies = extraStrategies
    result.extraOptimizations = extraOptimizations
    result
  }
}
```

Snippet from SparkOptimizer:
```scala
  override def batches: Seq[Batch] = (preOptimizationBatches ++ super.batches :+
    Batch("Optimize Metadata Only Query", Once, OptimizeMetadataOnlyQuery(catalog, conf)) :+
    Batch("Extract Python UDF from Aggregate", Once, ExtractPythonUDFFromAggregate) :+
    Batch("Prune File Source Table Partitions", Once, PruneFileSourcePartitions)) ++
    postHocOptimizationBatches :+
    Batch("User Provided Optimizers", fixedPoint, experimentalMethods.extraOptimizations: _*)
    
```

And here are the changes that facilitate to apply user provided optimization rules before spark's
native rules for `Analyzer` and `Optimizer` are applied:

```scala
class ExperimentalMethods private[sql]() {

  @volatile var extraStrategies: Seq[Strategy] = Nil

  @volatile var extraPreOptimizations: Seq[Rule[LogicalPlan]] = Nil

  @volatile var extraOptimizations: Seq[Rule[LogicalPlan]] = Nil

  override def clone(): ExperimentalMethods = {
    val result = new ExperimentalMethods
    result.extraStrategies = extraStrategies
    result.extraPreOptimizations = extraPreOptimizations
    result.extraOptimizations = extraOptimizations
    result
  }
}
```

```scala
  val experimentalPreOptimizations: Seq[Batch] = Seq(Batch(
    "User Provided Pre Optimizers", fixedPoint, experimentalMethods.extraPreOptimizations: _*))

  val experimentalPostOptimizations: Batch = Batch(
    "User Provided Post Optimizers", fixedPoint, experimentalMethods.extraOptimizations: _*)

  override def batches: Seq[Batch] = experimentalPreOptimizations ++
    (preOptimizationBatches ++ super.batches :+
    Batch("Optimize Metadata Only Query", Once, OptimizeMetadataOnlyQuery(catalog)) :+
    Batch("Extract Python UDF from Aggregate", Once, ExtractPythonUDFFromAggregate) :+
    Batch("Prune File Source Table Partitions", Once, PruneFileSourcePartitions)) ++
    postHocOptimizationBatches :+ experimentalPostOptimizations
    
```

I hope these changes to the codebase would be very helpful in many cases for advanced optimization.
Also it will take full advantage of the spark's simplicity on defining new rules and plug them into 
spark on the fly during run time. At least these changes on the local branch of my laptop made it very 
useful to experiment with the SparkOptimizer without having to recompile the code every time.

