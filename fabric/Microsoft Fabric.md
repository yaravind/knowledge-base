# Microsoft Fabric

## Decision Tree

### Ingestion

[Reference](https://learn.microsoft.com/en-us/fabric/data-factory/decision-guide-data-integration)
![](i/fab-decision-tree-ingestion.svg)

[Reference](https://milescole.dev/data-engineering/2024/09/17/To-V-Order-or-Not.html)
![Vorder Decision Tree](./img/fab-decision-tree-vorder.png)
![](i/fab-decision-tree-vorder.png)
### Transformation

- [Configure Auto Table Stats Collection](https://learn.microsoft.com/en-us/fabric/data-engineering/automated-table-statistics)
- [Adaptive Target File Size Mgmt](https://blog.fabric.microsoft.com/en-us/blog/adaptive-target-file-size-management-in-fabric-spark)
- [Optimized Write in Fabric](https://milescole.dev/data-engineering/2024/08/16/A-Deep-Dive-into-Optimized-Write-in-Microsoft-Fabric.html)

### Autoscal vs Dynamic Allocation

- Node: Nodes refer to the **physical** or virtual machines that provide the computational resources
- Executor: Executors are the **processes** that run on those nodes to execute tasks assigned by the Spark driver.
- In Fabric, there is a defined 1:1 elationship between nodes and executors. A Node will run only 1 executor. (In traditional Spark, a Node can host multipe executors).
- Autoscale manages the number of **nodes** based on **overall activity,** while dynamic allocation manages the number of **executors** based on the workload **demands of individual Spark applications**. Therefore, _dynamic allocation can function without autoscale being enabled,_ allowing Spark to adapt to workload changes by adjusting the number of executors as needed, even if the number of nodes remains constant.

![](i/6b7981fe-ece4-4bf7-aba7-475e3d159d12.png)

## Data Engineering

https://milescole.dev/data-engineering/2024/08/22/Databricks-to-Fabric-Spark-Cluster-Deep-Dive.html

## Tools

- [PBI Inspector](https://github.com/NatVanG/PBI-InspectorV2)
- [Python: Package, Distribute & Consume in Fabric](https://milescole.dev/data-engineering/2025/03/26/Packaging-Python-Libraries-Using-Microsoft-Fabric.html)