# Cluster configuration guidelines and best practices<a name="emr-plan-instances-guidelines"></a>

Use the guidance in this section to help you determine the instance types, purchasing options, and amount of storage to provision for each node type in an EMR cluster\.

## What instance type should you use?<a name="emr-instance-group-size"></a>

There are several ways to add Amazon EC2 instances to a cluster, which depend on whether you use the instance groups configuration or the instance fleets configuration for the cluster\.
+ **Instance Groups**
  + Manually add instances of the same type to existing core and task instance groups\.
  + Manually add a task instance group, which can use a different instance type\.
  + Set up automatic scaling in Amazon EMR for an instance group, adding and removing instances automatically based on the value of an Amazon CloudWatch metric that you specify\. For more information, see [Scaling cluster resources](emr-scale-on-demand.md)\.
+ **Instance Fleets**
  + Add a single task instance fleet\.
  + Change the target capacity for On\-Demand and Spot Instances for existing core and task instance fleets\. For more information, see [Configure instance fleets](emr-instance-fleet.md)\.

One way to plan the instances of your cluster is to run a test cluster with a representative sample set of data and monitor the utilization of the nodes in the cluster\. For more information, see [View and monitor a cluster](emr-manage-view.md)\. Another way is to calculate the capacity of the instances you are considering and compare that value against the size of your data\.

In general, the master node type, which assigns tasks, doesn't require an EC2 instance with much processing power; Amazon EC2 instances for the core node type, which process tasks and store data in HDFS, need both processing power and storage capacity; Amazon EC2 instances for the task node type, which don't store data, need only processing power\. For guidelines about available Amazon EC2 instances and their configuration, see [Configure Amazon EC2 instances](emr-plan-ec2-instances.md)\.

 The following guidelines apply to most Amazon EMR clusters\. 
+ By default, the total number of Amazon EC2 instances you can run on a single AWS account is 20\. This means that the total number of nodes you can have in a cluster is 20\. For more information about how to request a limit increase for your account, see [AWS limits](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-ec2-instances)\. 
+ The master node does not typically have large computational requirements\. For clusters with a large number of nodes, or for clusters with applications that are specifically deployed on the master node \(JupyterHub, Hue, etc\.\), a larger master node may be required and can help improve cluster performance\. For example, consider using an m5\.xlarge instance for small clusters \(50 or fewer nodes\), and increasing to a larger instance type for larger clusters\.
+ The computational needs of the core and task nodes depend on the type of processing your application performs\. Many jobs can be run on general purpose instance types, which offer balanced performance in terms of CPU, disk space, and input/output\. Computation\-intensive clusters may benefit from running on High CPU instances, which have proportionally more CPU than RAM\. Database and memory\-caching applications may benefit from running on High Memory instances\. Network\-intensive and CPU\-intensive applications like parsing, NLP, and machine learning may benefit from running on cluster compute instances, which provide proportionally high CPU resources and increased network performance\.
+ If different phases of your cluster have different capacity needs, you can start with a small number of core nodes and increase or decrease the number of task nodes to meet your job flow's varying capacity requirements\. 
+ The amount of data you can process depends on the capacity of your core nodes and the size of your data as input, during processing, and as output\. The input, intermediate, and output datasets all reside on the cluster during processing\. 

## When should you use Spot Instances?<a name="emr-plan-spot-instances"></a>

When you launch a cluster in Amazon EMR, you can choose to launch master, core, or task instances on Spot Instances\. Because each type of instance group plays a different role in the cluster, there are implications of launching each node type on Spot Instances\. You can't change an instance purchasing option while a cluster is running\. To change from On\-Demand to Spot Instances or vice versa, for the master and core nodes, you must terminate the cluster and launch a new one\. For task nodes, you can launch a new task instance group or instance fleet, and remove the old one\.

**Topics**
+ [Amazon EMR settings to prevent job failure because of task node Spot Instance termination](#emr-plan-spot-YARN)
+ [Master node on a Spot Instance](#emr-dev-master-instance-group-spot)
+ [Core nodes on Spot Instances](#emr-dev-core-instance-group-spot)
+ [Task nodes on Spot Instances](#emr-dev-task-instance-group-spot)
+ [Instance configurations for application scenarios](#emr-plan-spot-scenarios)

### Amazon EMR settings to prevent job failure because of task node Spot Instance termination<a name="emr-plan-spot-YARN"></a>

Because Spot Instances are often used to run task nodes, Amazon EMR has default functionality for scheduling YARN jobs so that running jobs do not fail when task nodes running on Spot Instances are terminated\. Amazon EMR does this by allowing application master processes to run only on core nodes\. The application master process controls running jobs and needs to stay alive for the life of the job\.

Amazon EMR release version 5\.19\.0 and later uses the built\-in [YARN node labels](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/NodeLabel.html) feature to achieve this\. \(Earlier versions used a code patch\)\. Properties in the `yarn-site` and `capacity-scheduler` configuration classifications are configured by default so that the YARN capacity\-scheduler and fair\-scheduler take advantage of node labels\. Amazon EMR automatically labels core nodes with the `CORE` label, and sets properties so that application masters are scheduled only on nodes with the CORE label\. Manually modifying related properties in the yarn\-site and capacity\-scheduler configuration classifications, or directly in associated XML files, could break this feature or modify this functionality\.

Amazon EMR configures the following properties and values by default\. Use caution when configuring these properties\.
+ **yarn\-site \(yarn\-site\.xml\) On All Nodes**
  + `yarn.node-labels.enabled: true`
  + `yarn.node-labels.am.default-node-label-expression: 'CORE'`
  + `yarn.node-labels.fs-store.root-dir: '/apps/yarn/nodelabels'`
  + `yarn.node-labels.configuration-type: 'distributed'`
+ **yarn\-site \(yarn\-site\.xml\) On Master And Core Nodes**
  + `yarn.nodemanager.node-labels.provider: 'config'`
  + `yarn.nodemanager.node-labels.provider.configured-node-partition: 'CORE'`
+ **capacity\-scheduler \(capacity\-scheduler\.xml\) On All Nodes**
  + `yarn.scheduler.capacity.root.accessible-node-labels: '*'`
  + `yarn.scheduler.capacity.root.accessible-node-labels.CORE.capacity: 100`
  + `yarn.scheduler.capacity.root.default.accessible-node-labels: '*'`
  + `yarn.scheduler.capacity.root.default.accessible-node-labels.CORE.capacity: 100`

**Note**  
Beginning with Amazon EMR 6\.x release series, the YARN node labels feature is disabled by default\. The application master processes can run on both core and task nodes by default\. You can enable the YARN node labels feature by configuring following properties:   
`yarn.node-labels.enabled: true`
`yarn.node-labels.am.default-node-label-expression: 'CORE'`

### Master node on a Spot Instance<a name="emr-dev-master-instance-group-spot"></a>

The master node controls and directs the cluster\. When it terminates, the cluster ends, so you should only launch the master node as a Spot Instance if you are running a cluster where sudden termination is acceptable\. This might be the case if you are testing a new application, have a cluster that periodically persists data to an external store such as Amazon S3, or are running a cluster where cost is more important than ensuring the cluster's completion\. 

When you launch the master instance group as a Spot Instance, the cluster does not start until that Spot Instance request is fulfilled\. This is something to consider when selecting your maximum Spot price\.

You can only add a Spot Instance master node when you launch the cluster\. Master nodes cannot be added or removed from a running cluster\. 

Typically, you would only run the master node as a Spot Instance if you are running the entire cluster \(all instance groups\) as Spot Instances\. 

### Core nodes on Spot Instances<a name="emr-dev-core-instance-group-spot"></a>

Core nodes process data and store information using HDFS\. Terminating a core instance risks data loss\. For this reason, you should only run core nodes on Spot Instances when partial HDFS data loss is tolerable\.

When you launch the core instance group as Spot Instances, Amazon EMR waits until it can provision all of the requested core instances before launching the instance group\. In other words, if you request six Amazon EC2 instances, and only five are available at or below your maximum Spot price, the instance group won't launch\. Amazon EMR continues to wait until all six Amazon EC2 instances are available or until you terminate the cluster\. You can change the number of Spot Instances in a core instance group to add capacity to a running cluster\. For more information about working with instance groups, and how Spot Instances work with instance fleets, see [Create a cluster with instance fleets or uniform instance groups](emr-instance-group-configuration.md)\.

### Task nodes on Spot Instances<a name="emr-dev-task-instance-group-spot"></a>

The task nodes process data but do not hold persistent data in HDFS\. If they terminate because the Spot price has risen above your maximum Spot price, no data is lost and the effect on your cluster is minimal\.

When you launch one or more task instance groups as Spot Instances, Amazon EMR provisions as many task nodes as it can, using your maximum Spot price\. This means that if you request a task instance group with six nodes, and only five Spot Instances are available at or below your maximum Spot price, Amazon EMR launches the instance group with five nodes, adding the sixth later if possible\. 

Launching task instance groups as Spot Instances is a strategic way to expand the capacity of your cluster while minimizing costs\. If you launch your master and core instance groups as On\-Demand Instances, their capacity is guaranteed for the run of the cluster\. You can add task instances to your task instance groups as needed, to handle peak traffic or speed up data processing\. 

You can add or remove task nodes using the console, AWS CLI, or API\. You can also add additional task groups, but you cannot remove a task group after it is created\. 

### Instance configurations for application scenarios<a name="emr-plan-spot-scenarios"></a>

The following table is a quick reference to node type purchasing options and configurations that are usually appropriate for various application scenarios\. Choose the link to view more information about each scenario type\.


| Application scenario | Master node purchasing option | Core nodes purchasing option | Task nodes purchasing option | 
| --- | --- | --- | --- | 
| [Long\-running clusters and data warehouses](#emr-dev-when-use-spot-data-warehouses) | On\-Demand | On\-Demand or instance\-fleet mix | Spot or instance\-fleet mix | 
| [Cost\-driven workloads](#emr-dev-when-use-spot-cost-driven) | Spot | Spot | Spot | 
| [Data\-critical workloads](#emr-dev-when-use-spot-data-critical) | On\-Demand | On\-Demand | Spot or instance\-fleet mix | 
| [Application testing](#emr-dev-when-use-spot-application-testing) | Spot | Spot | Spot | 

 There are several scenarios in which Spot Instances are useful for running an Amazon EMR cluster\. 

#### Long\-running clusters and data warehouses<a name="emr-dev-when-use-spot-data-warehouses"></a>

If you are running a persistent Amazon EMR cluster that has a predictable variation in computational capacity, such as a data warehouse, you can handle peak demand at lower cost with Spot Instances\. You can launch your master and core instance groups as On\-Demand Instances to handle the normal capacity and launch the task instance group as Spot Instances to handle your peak load requirements\.

#### Cost\-driven workloads<a name="emr-dev-when-use-spot-cost-driven"></a>

If you are running transient clusters for which lower cost is more important than the time to completion, and losing partial work is acceptable, you can run the entire cluster \(master, core, and task instance groups\) as Spot Instances to benefit from the largest cost savings\.

#### Data\-critical workloads<a name="emr-dev-when-use-spot-data-critical"></a>

If you are running a cluster for which lower cost is more important than time to completion, but losing partial work is not acceptable, launch the master and core instance groups as On\-Demand Instances and supplement with one or more task instance groups of Spot Instances\. Running the master and core instance groups as On\-Demand Instances ensures that your data is persisted in HDFS and that the cluster is protected from termination due to Spot market fluctuations, while providing cost savings that accrue from running the task instance groups as Spot Instances\.

#### Application testing<a name="emr-dev-when-use-spot-application-testing"></a>

When you are testing a new application in order to prepare it for launch in a production environment, you can run the entire cluster \(master, core, and task instance groups\) as Spot Instances to reduce your testing costs\.

## Calculating the required HDFS capacity of a cluster<a name="emr-plan-instances-hdfs"></a>

 The amount of HDFS storage available to your cluster depends on these factors:
+ The number of Amazon EC2 instances used for core nodes\.
+ The capacity of the Amazon EC2 instance store for the instance type used\. For more information on instance store volumes, see [Amazon Amazon EC2 instance store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html) in the *Amazon EC2 User Guide for Linux Instances*\.
+ The number and size of Amazon EBS volumes attached to core nodes\.
+ A replication factor, which accounts for how each data block is stored in HDFS for RAID\-like redundancy\. By default, the replication factor is three for a cluster of 10 or more core nodes, two for a cluster of 4\-9 core nodes, and one for a cluster of three or fewer nodes\.

To calculate the HDFS capacity of a cluster, for each core node, add the instance store volume capacity to the Amazon EBS storage capacity \(if used\)\. Multiply the result by the number of core nodes, and then divide the total by the replication factor based on the number of core nodes\. For example, a cluster with 10 core nodes of type i2\.xlarge, which have 800 GB of instance storage without any attached Amazon EBS volumes, has a total of approximately 2,666 GB available for HDFS \(10 nodes x 800 GB ÷ 3 replication factor\)\.

 If the calculated HDFS capacity value is smaller than your data, you can increase the amount of HDFS storage in the following ways: 
+ Creating a cluster with additional Amazon EBS volumes or adding instance groups with attached Amazon EBS volumes to an existing cluster
+ Adding more core nodes
+ Choosing an Amazon EC2 instance type with greater storage capacity
+ Using data compression
+ Changing the Hadoop configuration settings to reduce the replication factor

Reducing the replication factor should be used with caution as it reduces the redundancy of HDFS data and the ability of the cluster to recover from lost or corrupted HDFS blocks\. 