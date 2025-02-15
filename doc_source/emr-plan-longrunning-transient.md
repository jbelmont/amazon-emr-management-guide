# Configuring a cluster to auto\-terminate or continue<a name="emr-plan-longrunning-transient"></a>

By default, clusters that you create using the console or the AWS CLI continue to run until you shut them down\. To have a cluster terminate after running steps, you need to enable auto\-termination\. In contrast, clusters that you launch using the EMR API have auto\-termination enabled by default\.

**Note**  
Amazon EMR automatically enables termination protection for all clusters with multiple master nodes, and overrides any auto\-termination settings that you supply when you create the cluster\. You cannot enable auto\-termination after the cluster has launched, so multi\-master clusters cannot use auto\-termination\. You can disable termination protection after the cluster has been launched\. See [Configuring termination protection for running clusters](UsingEMR_TerminationProtection.md#emr-termination-protection-running-cluster)\. To shut down a cluster with multiple master nodes, you must first modify the cluster attributes to disable termination protection\. For instructions, see [Terminate an EMR Cluster with multiple master nodes](emr-plan-ha-launch.md#emr-plan-ha-launch-terminate)\.

**To disable auto\-termination using the EMR API**
+ When using the [RunJobFlow](https://docs.aws.amazon.com/ElasticMapReduce/latest/API/API_RunJobFlow.html) action to create a cluster, set the [KeepJobFlowAliveWhenNoSteps](https://docs.aws.amazon.com/ElasticMapReduce/latest/API/API_JobFlowInstancesConfig.html#EMR-Type-JobFlowInstancesConfig-KeepJobFlowAliveWhenNoSteps) property to `true`\.

**To enable auto\-termination using quick options in the AWS Management Console**

1. Open the Amazon EMR console at [https://console\.aws\.amazon\.com/elasticmapreduce/](https://console.aws.amazon.com/elasticmapreduce/)\.

1. Choose **Create cluster**\.

1. Choose **Step execution**\.

1. Choose other settings as appropriate for your application, and then choose **Create cluster**\.

**To enable auto\-termination using Advanced Options in the AWS Management Console**

1. Open the Amazon EMR console at [https://console\.aws\.amazon\.com/elasticmapreduce/](https://console.aws.amazon.com/elasticmapreduce/)\.

1. Choose **Create cluster**\.

1. Choose **Go to advanced options**\.

1. Under **Add steps \(optional\)** select **Auto\-terminate cluster after the last step is completed**\.

1. Choose other settings as appropriate for your application, and then choose **Create cluster**\.

**To enable auto\-termination using the AWS CLI**
+ Specify the `--auto-terminate` parameter when you use the `create-cluster` command to create a transient cluster\.

  The following example demonstrates using the `--auto-terminate` parameter\. You can type the following command and replace *myKey* with the name of your EC2 key pair\.
**Note**  
Linux line continuation characters \(\\\) are included for readability\. They can be removed or used in Linux commands\. For Windows, remove them or replace with a caret \(^\)\.

  ```
  aws emr create-cluster --name "Test cluster" --release-label emr-5.33.0 \
  --applications Name=Hive Name=Pig --use-default-roles --ec2-attributes KeyName=myKey \
  --steps Type=PIG,Name="Pig Program",ActionOnFailure=CONTINUE,\
  Args=[-f,s3://mybucket/scripts/pigscript.pig,-p,\
  INPUT=s3://mybucket/inputdata/,-p,OUTPUT=s3://mybucket/outputdata/,\
  $INPUT=s3://mybucket/inputdata/,$OUTPUT=s3://mybucket/outputdata/]
  --instance-type m5.xlarge --instance-count 3 --auto-terminate
  ```

For more information on using Amazon EMR commands in the AWS CLI, see [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/emr)\.