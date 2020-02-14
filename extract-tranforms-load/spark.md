---
description: Everything related to spark
---

# Spark

![Spark process](../.gitbook/assets/image%20%285%29.png)

## Explanation

* `SparkContext` is one application. 
* Application when running will have multiple executors on workers.
* Each executor will have a number of core as well as it owns memory.

#### Spark config

* Spark driver \(`spark.driver.cores,` `spark.driver.memory`\) is the program that declares the transformations and actions on RDDs of data and submits such requests to the master.
* Number of cores \(`--executor-cores` or `spark.executor.cores`\) is concurrent tasks an executor can run
* Number of executor \(`--num-executors` or `spark.executor.instances`\) is the maximum executors that the cluster can handle
* Executor memory \(`--executor-memory` or `spark.executor.memory`\) is the memory of the executor

## Spark config optimization

For example with the below cluster

```bash
**Cluster Config:**
10 Nodes
16 cores per Node
64GB RAM per Node
```

### 1. for one application

* Number of cores: 5 \# ideal number for HDFS to run 5 concurrent tasks
* Total cores = \[cores per node - 1 \(for application master\) \] x number of node . E.g \(16-1\)x10 = 150
* Number of available executors = total cores / number of core per executors. E.g = 150/5 = 30
* Leaving 1 executor for ApplicationManager =&gt; `--num-executors` = 29
* Number of executors per node = 30/10 = 3
* Memory per executor = 64GB/3 = 21GB
* Counting off heap overhead = 7% of 21GB = 3GB. So, actual `--executor-memory` = 21 - 3 = 18GB

### 2. DynamicAllocation









### References

{% embed url="http://site.clairvoyantsoft.com/understanding-resource-allocation-configurations-spark-application/" %}

{% embed url="https://spoddutur.github.io/spark-notes/distribution\_of\_executors\_cores\_and\_memory\_for\_spark\_application.html" %}

{% embed url="https://stackoverflow.com/questions/24637312/spark-driver-in-apache-spark" %}



