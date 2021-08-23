# Working with Amazon EMR\-managed security groups<a name="emr-man-sec-groups"></a>

 Different managed security groups are associated with the master instance and with the core and task instances in a cluster\. An additional managed security group for service access is required when you create a cluster in a private subnet\. For more information about the role of managed security groups with respect to your network configuration, see [Amazon VPC options](emr-clusters-in-a-vpc.md)\.

When you specify managed security groups for a cluster, you must use the same type of security group, default or custom, for all managed security groups\. For example, you can't specify a custom security group for the master instance, and then not specify a custom security group for core and task instances\.

If you use default managed security groups, you don't need to specify them when you create a cluster\. Amazon EMR automatically uses the defaults\. Moreover, if the defaults don't exist in the cluster's VPC yet, Amazon EMR creates them\. Amazon EMR also creates them if you explicitly specify them and they don't exist yet\.

You can edit rules in managed security groups after clusters are created\. When you create a new cluster, Amazon EMR checks the rules in the managed security groups that you specify, and then creates any missing *inbound* rules that the new cluster needs in addition to rules that may have been added earlier\. Unless specifically stated otherwise, each rule for default EMR\-managed security groups is also added to custom EMR\-managed security groups that you specify\.

The default managed security groups are as follows:
+ **ElasticMapReduce\-master**

  For rules in this security group, see [Amazon EMR\-managed security group for the master instance \(public subnets\)](#emr-sg-elasticmapreduce-master)\.
+ **ElasticMapReduce\-slave**

  For rules in this security group, see [Amazon EMR\-managed security group for core and task instances \(public subnets\)](#emr-sg-elasticmapreduce-slave)\.
+ **ElasticMapReduce\-Master\-Private**

  For rules in this security group, see [Amazon EMR\-managed security group for the master instance \(private subnets\)](#emr-sg-elasticmapreduce-master-private)\.
+ **ElasticMapReduce\-Slave\-Private**

  For rules in this security group, see [Amazon EMR\-managed security group for core and task instances \(private subnets\)](#emr-sg-elasticmapreduce-slave-private)\.
+ **ElasticMapReduce\-ServiceAccess**

  For rules in this security group, see [Amazon EMR\-managed security group for service access \(private subnets\)](#emr-sg-elasticmapreduce-sa-private)\.

## Amazon EMR\-managed security group for the master instance \(public subnets\)<a name="emr-sg-elasticmapreduce-master"></a>

The default managed security group for the master instance in public subnets has the **Group Name** of **ElasticMapReduce\-master**\. It has the following rules\. If you specify a custom managed security group, Amazon EMR will add all the same rules to your custom security group\.

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-man-sec-groups.html)

**To allow SSH access for trusted sources for the ElasticMapReduce\-master security group**

To edit your security groups, you must have permission to manage security groups for the VPC that the cluster is in\. For more information, see [Changing Permissions for an IAM User](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_change-permissions.html) and the [Example Policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_ec2_securitygroups-vpc.html) that allows managing EC2 security groups in the *IAM User Guide*\.

1. Open the Amazon EMR console at [https://console\.aws\.amazon\.com/elasticmapreduce/](https://console.aws.amazon.com/elasticmapreduce/)\.

1. Choose **Clusters**\.

1. Choose the **Name** of the cluster you want to modify\.

1. Choose the **Security groups for Master** link under **Security and access**\.

1. Choose **ElasticMapReduce\-master** from the list\.

1. Choose the **Inbound rules** tab and then **Edit inbound rules**\.

1. Check for an inbound rule that allows public access with the following settings\. If it exists, choose **Delete** to remove it\.
   + **Type**

     SSH
   + **Port**

     22
   + **Source**

     Custom 0\.0\.0\.0/0
**Warning**  
Before December 2020, the ElasticMapReduce\-master security group had a pre\-configured rule to allow inbound traffic on Port 22 from all sources\. This rule was created to simplify initial SSH connections to the master node\. We strongly recommend that you remove this inbound rule and restrict traffic to trusted sources\.

1. Scroll to the bottom of the list of rules and choose **Add Rule**\.

1. For **Type**, select **SSH**\.

   Selecting SSH automatically enters **TCP** for **Protocol** and **22** for **Port Range**\.

1. For source, select **My IP** to automatically add your IP address as the source address\. You can also add a range of **Custom** trusted client IP addresses, or create additional rules for other clients\. Many network environments dynamically allocate IP addresses, so you might need to update your IP addresses for trusted clients in the future\.

1. Choose **Save**\.

1. Optionally, choose **ElasticMapReduce\-slave** from the list and repeat the steps above to allow SSH client access to core and task nodes\.

## Amazon EMR\-managed security group for core and task instances \(public subnets\)<a name="emr-sg-elasticmapreduce-slave"></a>

The default managed security group for core and task instances in public subnets has the **Group Name** of **ElasticMapReduce\-slave**\. The default managed security group has the following rules, and Amazon EMR adds the same rules if you specify a custom managed security group\.

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-man-sec-groups.html)

## Amazon EMR\-managed security group for the master instance \(private subnets\)<a name="emr-sg-elasticmapreduce-master-private"></a>

The default managed security group for the master instance in private subnets has the **Group Name** of **ElasticMapReduce\-Master\-Private**\. The default managed security group has the following rules, and Amazon EMR adds the same rules if you specify a custom managed security group\.

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-man-sec-groups.html)

## Amazon EMR\-managed security group for core and task instances \(private subnets\)<a name="emr-sg-elasticmapreduce-slave-private"></a>

The default managed security group for core and task instances in private subnets has the **Group Name** of **ElasticMapReduce\-Slave\-Private**\. The default managed security group has the following rules, and Amazon EMR adds the same rules if you specify a custom managed security group\.

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-man-sec-groups.html)

### Editing outbound rules<a name="private-sg-egress-rules"></a>

By default, Amazon EMR creates this security group with outbound rules that allow all outbound traffic on all protocols and ports\. Allowing all outbound traffic is selected because various Amazon EMR and customer applications that can run on Amazon EMR clusters may require different egress rules\. EMR is not able to anticipate these specific settings when creating default security groups\. You can scope down egress in your security groups to include only those rules that suit your use cases and security policies\. At minimum, this security group requires the following outbound rules, but some applications might need additional egress\.


| Type | Protocol | Port range | Destination | Details | 
| --- | --- | --- | --- | --- | 
| All TCP | TCP | All | pl\-xxxxxxxx | Managed Amazon S3 prefix list com\.amazonaws\.MyRegion\.s3\. | 
| All Traffic | All | All | sg\-xxxxxxxxxxxxxxxxx | The ID of the ElasticMapReduce\-Slave\-Private security group\. | 
| All Traffic | All | All | sg\-xxxxxxxxxxxxxxxxx | The ID of the ElasticMapReduce\-Master\-Private security group\. | 
| Custom TCP | TCP | 9443 | sg\-xxxxxxxxxxxxxxxxx | The ID of the ElasticMapReduce\-ServiceAccess security group\. | 

## Amazon EMR\-managed security group for service access \(private subnets\)<a name="emr-sg-elasticmapreduce-sa-private"></a>

The default managed security group for service access in private subnets has the **Group Name** of **ElasticMapReduce\-ServiceAccess**\. It has inbound rules, and outbound rules that allow traffic over HTTPS \(port 8443, port 9443\) to the other managed security groups in private subnets\. These rules allow the cluster manager to communicate with the master node and with core and task nodes\. The same rules are needed if you are using custom security groups\.


| Type | Protocol | Port range | Source | Details | 
| --- | --- | --- | --- | --- | 
| Inbound rules Required for EMR clusters with Amazon EMR release 5\.30\.0 and later\. | 
| Custom TCP | TCP | 9443 | The Group ID of the managed security group for master instance\.  |  This rule allows the communication between master instance's security group to the service access security group\. | 
| Outbound rules Required for all EMR clusters | 
| Custom TCP | TCP | 8443 | The Group ID of the managed security group for master instance\.  |  These rules allow the cluster manager to communicate with the master node and with core and task nodes\. | 
| Custom TCP | TCP | 8443 | The Group ID of the managed security group for core and task instances\.  |  These rules allow the cluster manager to communicate with the master node and with core and task nodes\.  | 