# Configure networking<a name="emr-plan-vpc-subnet"></a>

Most clusters launch into a virtual network using Amazon Virtual Private Cloud \(Amazon VPC\)\. A VPC is an isolated virtual network within AWS that is logically isolated within your AWS account\. You can configure aspects such as private IP address ranges, subnets, routing tables, and network gateways\. For more information, see the [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)\.

VPC offers the following capabilities:
+ **Processing sensitive data**

  Launching a cluster into a VPC is similar to launching the cluster into a private network with additional tools, such as routing tables and network ACLs, to define who has access to the network\. If you are processing sensitive data in your cluster, you may want the additional access control that launching your cluster into a VPC provides\. Furthermore, you can choose to launch your resources into a private subnet where none of those resources has direct internet connectivity\.
+ **Accessing resources on an internal network**

  If your data source is located in a private network, it may be impractical or undesirable to upload that data to AWS for import into Amazon EMR, either because of the amount of data to transfer or because of the sensitive nature of the data\. Instead, you can launch the cluster into a VPC and connect your data center to your VPC through a VPN connection, enabling the cluster to access resources on your internal network\. For example, if you have an Oracle database in your data center, launching your cluster into a VPC connected to that network by VPN makes it possible for the cluster to access the Oracle database\. 

****Public and private subnets****  
You can launch Amazon EMR clusters in both public and private VPC subnets\. This means you do not need internet connectivity to run an Amazon EMR cluster; however, you may need to configure network address translation \(NAT\) and VPN gateways to access services or resources located outside of the VPC, for example in a corporate intranet or public AWS service endpoints like AWS Key Management Service\.

**Important**  
Amazon EMR only supports launching clusters in private subnets in release version 4\.2 and later\.

For more information about Amazon VPC, see the [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)\.

**Topics**
+ [Amazon VPC options](emr-clusters-in-a-vpc.md)
+ [Set up a VPC to host clusters](emr-vpc-host-job-flows.md)
+ [Launch clusters into a VPC](emr-vpc-launching-job-flows.md)
+ [Minimum Amazon S3 policy for private subnet](private-subnet-iampolicy.md)
+ [More resources for learning about VPCs](#emr-resources-about-vpcs)

## More resources for learning about VPCs<a name="emr-resources-about-vpcs"></a>

Use the following topics to learn more about VPCs and subnets\.
+ Private Subnets in a VPC
  + [Scenario 2: VPC with Public and Private Subnets \(NAT\)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)
  + [NAT Instances](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html)
  + [High Availability for Amazon VPC NAT Instances: An Example](https://aws.amazon.com/articles/2781451301784570)
+ Public Subnets in a VPC
  + [Scenario 1: VPC with a Single Public Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html)
+ General VPC Information
  + [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
  + [VPC Peering](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-peering.html)
  + [Using Elastic Network Interfaces with Your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html)
  + [Securely connect to Linux instances running in a private VPC](https://blogs.aws.amazon.com/security/post/Tx3N8GFK85UN1G6/Securely-connect-to-Linux-instances-running-in-a-private-Amazon-VPC)