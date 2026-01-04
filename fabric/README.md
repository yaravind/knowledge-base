# README

Microsoft Fabric
#fabric

## Decision Tree

### Ingestion

[Reference](https://learn.microsoft.com/en-us/fabric/data-factory/decision-guide-data-integration)
![](i/fab-decision-tree-ingestion.svg)


[Reference](https://milescole.dev/data-engineering/2024/09/17/To-V-Order-or-Not.html)
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
- When Dynamic Allocation option is enabled, for every spark application submitted, **the system reserves executors during the job submission step based on the Max Nodes**, which were specified by the user to support successful auto scale scenarios.

![](i/6b7981fe-ece4-4bf7-aba7-475e3d159d12.png)

## Data Engineering

### Anatomy of a Spark Instance

```mermaid
flowchart TB
    subgraph SparkPool["Fabric Spark Pool Instance"]
        HeadNode["Head Node (Fabric‑managed)
        - Spark Driver
        - Livy
        - YARN Resource Manager
        - ZooKeeper
        - Node Agent
        - YARN Node Manager"]

        subgraph Workers["Worker Nodes (Fabric‑managed, Auto‑scaled)"]
            Worker1["Worker Node
            - Spark Executor
            - Node Agent
            - YARN Node Manager"]

            WorkerN["Worker Node
            - Spark Executor
            - Node Agent
            - YARN Node Manager"]
        end
    end

    HeadNode --> Worker1
    HeadNode --> WorkerN
```

### Starter Pool

- Can be near‑instant when warm
- Can fall back to on‑demand provisioning on certain scenarios
- Subject to regional availability and networking constraints

### Custom Pool

- Nodes are acquired on demand from Azure
- No “warm pool reuse” or regional exhaustion logic
- More predictable and consistent startup behavior than Starter Pools

### Default Pool

- Is **not** a third type of pool alongside Starter Pool and Custom Pool. 
- Instead, it tells which pool Fabric will automatically use when you run Spark workloads and you haven’t explicitly chosen a pool
- Configurable per workspace
	- A _Starter Pool _can be set as the default pool
		- This is the usual setup for most workspaces
		- Fast startup, no cluster management, backed by Fabric-managed resources
	- A _Custom Pool_ can also be set as the default pool
		- Useful when you want all Spark workloads to use specific sizing, autoscaling, or configs by default

**Why this matters**

Choosing the right default pool:
	- Helps control cost and performance
	- Avoids accidental execution on an expensive pool
	- Ensures consistent runtime behavior across notebooks and jobs


https://milescole.dev/data-engineering/2024/08/22/Databricks-to-Fabric-Spark-Cluster-Deep-Dive.html

### Spark session startup flow

- Starter Pool advantage = possible reuse
- Custom Pool advantage = consistent behavior

```mermaid
flowchart TB
    User["User or Pipeline"]

    Artifact["Notebook or
    Spark Job Definition"]

    User --> Artifact

    subgraph Fabric["Microsoft Fabric Workspace"]

        PoolSelect["Spark Pool Selection
        (Default or Explicit)"]

        Artifact -->|"Submit job"| PoolSelect
        subgraph StarterPool["Starter Spark Pool"]
            StarterDecision["Starter Pool orchestration"]

            StarterWarm["Reuse warm Starter Pool
            (5 - 10 seconds)"]

            StarterProvision["Provision cluster
            from Azure
            (2 – 5 minutes)"]

            StarterPersonalize["(Optional) Session setup (Spark configs, libraries)
            (30 seconds – 5 minutes)"]

            StarterDecision -->|"Capacity available"| StarterWarm
            StarterDecision -->|"Starter Pools in your region are fully used"| StarterProvision

            StarterWarm --> StarterPersonalize
            StarterProvision --> StarterPersonalize
        end
        subgraph CustomPool["Custom Spark Pool"]
            CustomProvision["Provision cluster
            from Azure
            (2 – 5 minutes)"]

            CustomPersonalize["Optional) Session setup
            (Spark configs, libraries)"]
        end
        subgraph Execution["Spark Instance"]
            Session["Spark Session (Ready)"]

            Driver["Head Node"]

            Workers["Worker Nodes"]
        end
    end

    PoolSelect -->|"Starter Pool"| StarterDecision
    PoolSelect -->|"Custom Pool"| CustomProvision

    StarterPersonalize --> Session
    CustomPersonalize --> Session
    CustomProvision --> CustomPersonalize

    Session --> Driver
    Driver --> Workers
```

### Key Takeaway

1. Starter Pools optimize for fast and bursty interactive workloads
2. Custom Pools optimize for predictable execution characteristics
3. Both provision compute on demand when needed

#### Decision Tree

```mermaid
flowchart TD
    Start["Spark workload requirement"]

    Interactive["Ad‑hoc or interactive
    analysis?"]

    Networking["Private Link or
    Managed VNet enabled?"]

    Predictability["Need predictable
    startup time?"]

    HighConcurrency["High concurrency or
    multiple teams?"]

    Starter["Use Starter Spark Pool"]

    Custom["Use Custom Spark Pool"]

    Start --> Interactive

    Interactive -->|"Yes"| Networking
    Interactive -->|"No"| Predictability

    Networking -->|"Yes"| Custom
    Networking -->|"No"| Starter

    Predictability -->|"Yes"| Custom
    Predictability -->|"No"| HighConcurrency

    HighConcurrency -->|"Yes"| Custom
    HighConcurrency -->|"No"| Starter
```

## Tools

- [PBI Inspector](https://github.com/NatVanG/PBI-InspectorV2)
- [Python: Package, Distribute & Consume in Fabric](https://milescole.dev/data-engineering/2025/03/26/Packaging-Python-Libraries-Using-Microsoft-Fabric.html)