# Launch an EMR Cluster with multiple master nodes<a name="emr-plan-ha-launch"></a>

This topic provides configuration details and examples for launching an EMR cluster with multiple master nodes\.

**Note**  
Amazon EMR automatically enables termination protection for all clusters with multiple master nodes, and overrides any auto\-termination settings that you supply when you create the cluster\. To shut down a cluster with multiple master nodes, you must first modify the cluster attributes to disable termination protection\. For instructions, see [Terminate an EMR Cluster with multiple master nodes](#emr-plan-ha-launch-terminate)\.

## Prerequisites<a name="emr-plan-ha-launch-config"></a>
+ You can launch an EMR cluster with multiple master nodes in both public and private VPC subnets\. **EC2\-Classic** is not supported\. To launch an EMR cluster with multiple master nodes in a public subnet, you must enable the instances in this subnet to receive a public IP address by selecting **Auto\-assign IPv4** in the console or running the following command\. Replace *22XXXX01* with your subnet ID\.

  ```
  aws ec2 modify-subnet-attribute --subnet-id subnet-22XXXX01 --map-public-ip-on-launch					
  ```
+ To run Hive, Hue, or Oozie on an EMR cluster with multiple master nodes, you must create an external metastore\. For more information, see [Configuring an external metastore for Hive](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-metastore-external-hive.html), [Using Hue with a remote database in Amazon RDS](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/hue-rds.html), or [Apache Oozie](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-oozie.html)\.
+ To use Kerberos authentication in your cluster, you must configure an external KDC\. For more information, see [Configuring Kerberos on Amazon EMR](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-kerberos-configure.html)\.

## Launch an EMR Cluster with multiple master nodes<a name="emr-plan-ha-launch-examples"></a>

You must specify an instance count value of three for the master node instance group when you launch an EMR cluster with multiple master nodes\. The following examples demonstrate how to launch the cluster using the default AMI or a custom AMI\. 

**Note**  
You must specify the subnet ID when you launch an EMR cluster with multiple master nodes using the AWS CLI\. Replace *22XXXX01* with your subnet ID in the following examples\.

**Example – Launching an EMR cluster with multiple master nodes using a default AMI**  

```
aws emr create-cluster \
--name "ha-cluster" \
--release-label emr-5.33.0 \
--instance-groups InstanceGroupType=MASTER,InstanceCount=3,InstanceType=m5.xlarge InstanceGroupType=CORE,InstanceCount=4,InstanceType=m5.xlarge \
--ec2-attributes KeyName=ec2_key_pair_name,InstanceProfile=EMR_EC2_DefaultRole,SubnetId=subnet-22XXXX01 \
--service-role EMR_DefaultRole \
--applications Name=Hadoop Name=Spark
```

**Example – Launching an EMR cluster with multiple master nodes using a custom AMI**  

```
aws emr create-cluster \
--name "custom-ami-ha-cluster" \
--release-label emr-5.33.0 \
--instance-groups InstanceGroupType=MASTER,InstanceCount=3,InstanceType=m5.xlarge InstanceGroupType=CORE,InstanceCount=4,InstanceType=m5.xlarge \
--ec2-attributes KeyName=ec2_key_pair_name,InstanceProfile=EMR_EC2_DefaultRole,SubnetId=subnet-22XXXX01 \
--service-role EMR_DefaultRole \
--applications Name=Hadoop Name=Spark \
--custom-ami-id ami-MyAmiID
```

**Example – Launching an EMR cluster with multiple master nodes with an external Hive metastore**  
To run Hive on an EMR cluster with multiple master nodes, you must specify an external metastore for Hive, as the following example demonstrates,  

1. Create a temporary hiveConfiguration\.json file that contains credentials for your Hive metastore\.

   ```
   [
     {
       "Classification": "hive-site",
       "Properties": {
         "javax.jdo.option.ConnectionURL": "jdbc:mysql:\/\/hostname:3306\/hive?createDatabaseIfNotExist=true",
         "javax.jdo.option.ConnectionDriverName": "org.mariadb.jdbc.Driver",
         "javax.jdo.option.ConnectionUserName": "username",
         "javax.jdo.option.ConnectionPassword": "password"
       }
     }
   ]
   ```

1. Launch the cluster with the Hive metastore\.

   ```
   aws emr create-cluster \
   --name "ha-cluster-with-hive-metastore" \
   --release-label emr-5.33.0 \
   --instance-groups InstanceGroupType=MASTER,InstanceCount=3,InstanceType=m5.xlarge InstanceGroupType=CORE,InstanceCount=4,InstanceType=m5.xlarge \
   --ec2-attributes KeyName=ec2_key_pair_name,InstanceProfile=EMR_EC2_DefaultRole,SubnetId=subnet-22XXXX01 \
   --service-role EMR_DefaultRole \
   --applications Name=Hadoop Name= Spark Name=Hive \
   --configurations ./hiveConfiguration.json
   ```

## Terminate an EMR Cluster with multiple master nodes<a name="emr-plan-ha-launch-terminate"></a>

To terminate an EMR cluster with multiple master nodes, you must disable termination protection before terminating the cluster, as the following example demonstrates\. Replace *j\-3KVTXXXXXX7UG* with your cluster ID\.

```
aws emr modify-cluster-attributes --cluster-id j-3KVTXXXXXX7UG --no-termination-protected
aws emr terminate-clusters --cluster-id j-3KVTXXXXXX7UG
```