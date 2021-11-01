# Spark Catalyst Internals
Spark catalyst is one of the secret sauce of Spark's Operations on the structured data. Let's take
a deep look into its internals.


## TreeNode Abstraction
TreeNode is the fundamental data type  abstraction for the catalyst internals. This abstraction brings 
methods (such as foreach, map, flatmap, collect etc.) that helps to manipulate  scala functional for 
manipulating the internal tree node structure. The contract of the tree node abstraction to the operators/
class that extending it is to define the list of children.

# TreeNode library
![replace_except_with_not_filter_case-1](https://raw.githubusercontent.com/sathiyapk/blog-post/master/images/spark-catalyst-internals/Catalyst-TreeNode-Abstraction.svg)

## Evaluate-able TreeNode

![replace_except_with_not_filter_case-1](https://raw.githubusercontent.com/sathiyapk/blog-post/master/images/spark-catalyst-internals/Catalyst-TreeNode-Expression.svg)

## Query-able TreeNode
![replace_except_with_not_filter_case-1](https://raw.githubusercontent.com/sathiyapk/blog-post/master/images/spark-catalyst-internals/Catalyst-TreeNode-LogicalPlan.svg)


## Transform-able TreeNode


## Low-level RDD TreeNode

