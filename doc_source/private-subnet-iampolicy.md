# Minimum Amazon S3 policy for private subnet<a name="private-subnet-iampolicy"></a>

For private subnets, at a minimum you must provide the ability for Amazon EMR to access Amazon Linux repositories\. This private subnet policy is a part of the VPC endpoint policies for accessing Amazon S3\. With Amazon Amazon EMR 5\.25\.0 or later, to enable one\-click access to persistent Spark history server, you must allow Amazon EMR to access the system bucket that collects Spark event logs\. If you enable logging, provide PUT permissions to a `aws157-logs-*` bucket\. For more information, see [One\-click access to persistent Spark History Server](https://docs.aws.amazon.com/emr/latest/ManagementGuide/app-history-spark-UI.html)\.

It is up to you to determine the policy restrictions that meet your business needs\. For example, you can specify the Region `packages.us-east-1.amazonaws.com` to avoid an ambiguous Amazon S3 bucket name\. The following example policy provides permissions to access Amazon Linux repositories and the Amazon EMR system bucket for collecting Spark event logs\. Replace *MyRegion* with the Region where your log buckets reside, for example `us-east-1`\.

For more information about using IAM policies with Amazon VPC endpoints, see [Endpoint policies for Amazon S3](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html#vpc-endpoints-policies-s3)\.

```
{
   "Version": "2008-10-17",
   "Statement": [
       {
           "Sid": "AmazonLinuxAMIRepositoryAccess",
           "Effect": "Allow",
           "Principal": "*",
           "Action": "s3:GetObject",
           "Resource": [
           	"arn:aws:s3:::packages.MyRegion.amazonaws.com/*",
           	"arn:aws:s3:::repo.MyRegion.amazonaws.com/*",
           	"arn:aws:s3:::repo.MyRegion.emr.amazonaws.com/*"
           ]
       },
       {
           "Sid": "EnableApplicationHistory",
           "Effect": "Allow",
           "Principal": "*",
           "Action": [
               "s3:Put*",
               "s3:Get*",
               "s3:Create*",
               "s3:Abort*",
               "s3:List*"
           ],
           "Resource": [
           	"arn:aws:s3:::prod.MyRegion.appinfo.src/*"
           ]
       }
   ]
}
```

The following example policy provides the permissions required to access Amazon Linux 2 repositories\. Amazon Linux 2 AMI is the default\.

```
{
   "Statement": [
       {
           "Sid": "AmazonLinux2AMIRepositoryAccess",
           "Effect": "Allow",
           "Principal": "*",
           "Action": "s3:GetObject",
           "Resource": [
           	"arn:aws:s3:::amazonlinux.MyRegion.amazonaws.com/*",
           	"arn:aws:s3:::amazonlinux-2-repos-MyRegion/*"
           ]
       }
   ]
}
```