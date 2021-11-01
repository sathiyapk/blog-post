# Automating spark pipelines using generic code and configurations files :

A quick background about Transformation and Action in Spark.

### Transformations : 
As the name suggests, transformations transfers a dataset from one format to an another format.
They are lazy in order to be able to be replaced, optimized, staged, grouped, pipelined executed all at once 
until an action is called on that.

*Replaced:*

*Optimized:*

*staged:*

*grouped:*

*pipelined:*

A list of spark Transformation functions:

Narrow Transformations (without Shuffle):
*Map, FlatMap, MapPartition, Filter, Sample, Union*

Wide Transformations (with Shuffle): 
*Intersection, Distinct, ReduceByKey, GroupByKey, Join, Repartition, Coalesce*


### Action :
In contrast to transformation functions that can be detained from executing an action right away, 
Action functions needs the result right away and often they are the final action that completes a pipeline.

A list of spark Action functions:
*Show, Count, Collect, take, foreach, write*

### Pipeline : 
A Spark job is essentially composes of minimum one pipeline until n number of pipelines.


Here is an example of a typical spark pipeline.
````scala
val df = spark.read.parquet("..")
df
 .filter(...)
 .select(...)
 .withColumnRenamed("..", "..")
 .withColumn("..", f(".."))
 .write
 .parquet("..")
````


Transformations: filter, select, withColumnRenamed, withColumn

Action: write


## Datalake : 
I tend to see different definitions of the term datalake, so let's clear out the definition of the datalake 
that I refer to here. The term datalake here refers to the definition from Databricks. 
```
A datalake is a collection of multiple files of different formats varying in size and quality that are processed,
 refined and combiend in order to be able to run sql and datascience algorithms on it. 
```
The concept of datalake is getting more and more popular in almost every industry and sectors.

### Tables, Columns and Schema :
One of the most common pattern in datalake projects that I see often is constructing tables from the ingested data that
often involves hundreds of tables, columns and schemas to maintain.

Creating tables from the ingested data is fairly one of the easiest job. Although, maintaining hundreds of schemas in
java or scala code will lead to cumbersome and one can loose a lot of time .

### Externalising table Schemas in a csv files

A sample schema file :
```shell
source_path,table_name,field_name,column_name,data_type,transformation,index
src1,table1,field1,column1,int,,pk
src1,table1,field2,column2,string,,
src1,table1,field3,column3,float,,btree
src1,table2,field4,column1,float,,pk
src1,table2,field5,column2,int,,
src1,table2,field6,column3,string,,
src2,table3,field1,column1,string,,
src2,table3,field2,column2,string,,
src2,table3,field3,column3,int,,
```

***source_path:*** Path of the source file.

***table_name:*** Name of the target table to create from the input source.

***field_name:*** Name of the field in the source file.

***column_name:*** Name of the column in the target table.

***data_type:*** Column type in the target table.

***transformation:*** Defines the transformation type to apply to obtain the needed column value.
Few example: 
- hash(col1)
- upper(col1)
- date_format(current_timestamp(),"MM/dd/yyyy hh:mm").

***index:*** Whether to create index on the particular column or not. No value implies no indexes on the particular columns.

Configuration file for integrating new source without adding new code :
````shell
source-conf = [
  {
    name = source-1
    enable = true
    specification-path = specification-1.csv
    table-names = [
      "table-1"
      "table-2"
    ]
  }
  {
    name = source-2
    enable = false
    specification-path = specification-2.csv
    table-names = [
      "table-3"
      "table-4"
    ]
  }
  
  ...
  
  {
    name = source-n
    enable = true
    specification-path = specification-n.csv
    table-names = [
      "table-n"
    ]
  }
]
````

Scala Application Main : 
````scala

AppConfig.sourceConfigList.foreach { ds =>
  logger.info(s"Processing Datasource: $ds")
  val rawDF = Reader.readDataset(spark, ds)
  val dsSchema = Specification.load(ds.specificationPath)
  val compliantDF = new SpecConfirmer(rawDF, sourceSpecification).adaptToSchema
  
  DataLoader.load(compliantDF)
}
````

Generic Scala code that performs transformation using the schema file :

```scala

class SpecConfirmer(rawDF: DataFrame, specItem: Seq[SpecificationItem]) {
 var compliantDF = rawDF

 def adaptToSchema(specItem: Seq[SpecificationItem]): DataFrame = specItem.foreach { specItem =>
  compliantDF = compliantDF.withColumn(specItem.columnName, formatColumn(specItem))
 }

 private def formatColumn(specItem: SpecificationItem): Column = specItem.transformation match {
  case r"hash(field)" => hash(col(field))
  case r"concat(field1, field2)" => concat(col(field1), col(field2))
  case r"uuid" => expr("uuid()")
  case r"current_timestamp(format)" => date_format(current_timestamp(), format)
  case _ => col(specItem.fieldName).cast(specItem.columnType)
 }
}

```
The formatColumn method in the SpecConfirmer class above is simplified for better readability.

#### Advantages : 

- ***Data Ingestion without Code:***
Since the code is generic, we can automate ingestion of larger number of sources without further touching/rebuilding the source code.

- ***Manageable Codebase:***
By externalising the schema information and business rules, we can keep the source code
fairly small leading to better code management. Moreover, while dealing with multiple sources with multiple columns, we can easily mistaken the column name/type that
takes more time in debugging and rebuilding the code. External schema files facilitates shorter the debugging the rebuilding chain.

- ***Manageable Schema:***
Since the schema file supposed to contain very less technical details, we can able to exchange the
schema file with business experts like Data Scientist, Business Intelligence teams and even project owners.

####  Limitations : 

- ***Coding new transformation rule:***
Although we can integrate new sources with only a specification file without any code, integrating a new tranformation rule
for the first time may need extra time and effort.

- ***Effortful for complex transformation rules:***
Automating transformations using specification file works like a charm for sources with one-to-one transformations. However, 
integrating sources that includes complex adhoc transformations rules may be effortful. 

