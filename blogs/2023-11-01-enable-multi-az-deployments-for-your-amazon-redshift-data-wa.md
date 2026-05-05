---
title: "Enable Multi-AZ deployments for your Amazon Redshift data warehouse"
url: "https://aws.amazon.com/blogs/big-data/enable-multi-az-deployments-for-your-amazon-redshift-data-warehouse/"
date: "Wed, 01 Nov 2023 16:42:47 +0000"
author: "Ranjan Burman"
feed_url: "https://aws.amazon.com/blogs/big-data/category/database/amazon-redshift/feed/"
---
<p><em><strong>November 2023: This post was reviewed and updated with the general availability of Multi-AZ deployments for provisioned RA3 clusters.</strong></em><br /> <em>Originally published on December 9th, 2022.</em></p> 
<p><a href="http://aws.amazon.com/redshift" rel="noopener" target="_blank">Amazon Redshift</a> is a fully managed, petabyte scale cloud data warehouse that enables you to analyze large datasets using standard SQL. Data warehouse workloads are increasingly being used with mission-critical analytics applications that require the highest levels of resilience and availability. Amazon Redshift is a cloud-based data warehouse that supports many recovery capabilities to address unforeseen outages and minimize downtime. Amazon Redshift RA3 instance types store their data in Redshift Managed Storage (RMS), which is backed by <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) making it highly available and durable by default. Amazon Redshift also supports automatic backups that can recover a data warehouse, automatically remediate failures and <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/managing-cluster-recovery.html" rel="noopener" target="_blank">relocates</a> clusters to different AZs without changes to applications. Although many customers benefit from these features, enterprise data warehouse customers require a low Recovery Time Objective (RTO) and higher availability to support their business continuity with minimal impact to applications.</p> 
<p>Amazon Redshift just announced the general availability of Multi-AZ deployments for provisioned RA3 clusters that support running your data warehouse in two Availability Zones simultaneously and can continue operating in unforeseen failure scenarios. A Multi-AZ deployment is intended for customers with mission-critical analytics applications that require the highest levels of &nbsp;resilience and availability.</p> 
<p>A Redshift Multi-AZ deployment leverages compute resources in two AZs to scale data warehouse workload processing.&nbsp; In situations where there is a high level or concurrency Redshift will automatically leverage the resources in both AZs to scale the workload for both read and write requests.</p> 
<p>Our pre-launch tests found that&nbsp;Amazon Redshift Multi-AZ deployments reduce recovery time to under 60 seconds&nbsp;or less&nbsp;in the unlikely case of an AZ failure.</p> 
<h2>Single-AZ vs. Multi-AZ deployment</h2> 
<p>Amazon Redshift requires a cluster subnet group to create a cluster in your VPC. The cluster subnet group includes information about the VPC ID and a list of subnets in your VPC. When you launch a cluster, Amazon Redshift either creates a default cluster subnet group automatically or you choose a cluster subnet group of your choice so that Amazon Redshift can provision your cluster in one of the subnets in the VPC. You can configure your cluster subnet group to add subnets from different Availability Zones that you want Amazon Redshift to use for cluster deployment.</p> 
<p>All Amazon Redshift clusters today are created and situated in a particular Availability Zone within an AWS Region and thus called Single-AZ deployments. For a Single-AZ deployment, Amazon Redshift selects the subnet from one of the Availability Zones within a Region and deploys the cluster there. You can choose an Availability Zone for deployment, and Amazon Redshift will deploy your cluster in the chosen Availability Zone based on the subnets provided.</p> 
<p>On the other hand, a multi-AZ deployment is provisioned in two Availability Zones simultaneously. For a Multi-AZ deployment, Amazon Redshift automatically selects two subnets from two different Availability Zones and deploys an equal number of compute nodes in each Availability Zone. All these compute nodes are utilized via a single endpoint as compute nodes from both Availability Zones are used for workload processing.</p> 
<p>As shown in the following diagrams, Amazon Redshift deploys a cluster in a single Availability Zone for Single-AZ deployment, and two Availability Zones for Multi-AZ deployment.</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="" class="alignnone wp-image-55806 size-full" height="348" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image001.png" width="369" /></td> 
   <td><img alt="" class="alignnone wp-image-55807 size-full" height="358" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image003.png" width="381" /></td> 
  </tr> 
 </tbody> 
</table> 
<h2>Auto recovery of multi-AZ deployment</h2> 
<p>In the unlikely event of an Availability Zone failure, Amazon Redshift Multi-AZ deployments continue to serve your workloads by automatically using resources in the other Availability Zone. You are not required to make any application changes to maintain business continuity during unforeseen outages since a multi-AZ deployment is accessed as a single data warehouse with one endpoint. Amazon Redshift Multi-AZ deployments are designed to ensure there is no data loss, and you can query all data committed up until the point of failure.</p> 
<p>As shown in the below diagram, if there is an unlikely event that causes compute nodes in AZ1 to fail, then a multi-AZ deployment automatically recovers to use compute resources in AZ2. Amazon Redshift will also automatically provision identical compute nodes in another availability zone (AZ3) to continue operating simultaneously in two Availability zones (AZ2 and AZ3).</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="" class="alignnone wp-image-55808 size-full" height="437" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image005.png" title="Multi AZ deployment" width="444" /></td> 
   <td><img alt="" class="alignnone wp-image-55809 " height="335" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image006.png" title="Multi AZ deployment after auto recovery" width="360" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>Amazon Redshift Multi-AZ deployment is not only used for protection against the possibility of Availability Zone failures, but it can also maximize your data warehouse performance by automatically distributing workload processing across two Availability Zones. A Multi-AZ deployment will always process an individual query using compute resources only from one Availability Zone, but it can automatically distribute processing of multiple simultaneous queries to both Availability Zones to increase overall performance for high concurrency workloads.</p> 
<p>It’s a good practice to set up automatic retries in your extract, transform, and load (ETL) processes and dashboards so that they can be reissued and served by the cluster in the secondary Availability Zone when an unlikely failure happens in the primary Availability Zone. If a connection is dropped, it can then be retried or reestablished immediately. In addition, queries and loads that were running in the failed Availability Zone will be aborted. New queries issued at or after a failure occurs may experience run delays while the multi-AZ data warehouse is being recovered to a two AZ setup.</p> 
<h2>Overview of solution</h2> 
<p>In this post, we provide a walkthrough of how to create and manage a Multi-AZ deployment for Amazon Redshift using the <a href="http://aws.amazon.com/console" rel="noopener" target="_blank">AWS Management Console</a>. We also test the fault tolerance of an Amazon Redshift Multi-AZ data warehouse and monitor queries in your Multi-AZ deployment.</p> 
<h2>Create a new Multi-AZ deployment from the console</h2> 
<p>You can easily create a new multi-AZ deployments through Amazon Redshift console. Amazon Redshift will deploy the same number of nodes in each of the two Availability Zones for a Multi-AZ deployment. All nodes of a multi-AZ deployment can perform read and write workload processing during normal operation. A Multi-AZ deployment is supported only for provisioned RA3 clusters.</p> 
<p>Follow these steps to create an Amazon Redshift provisioned cluster in two Availability Zones:</p> 
<ol> 
 <li>On the Amazon Redshift console, in the navigation pane, choose <strong>Clusters</strong>.</li> 
 <li>Click on Create cluster.</li> 
</ol> 
<p>For general information about creating clusters, see&nbsp;<a href="https://docs.aws.amazon.com/redshift/latest/mgmt/managing-clusters-console.html#create-cluster" rel="noopener" target="_blank">Creating a cluster</a>.</p> 
<ol start="3"> 
 <li>Choose one of the RA3 node types on the <strong>Node type</strong> drop-down menu. The Multi-AZ deployment option only becomes available when you choose an RA3 node type.</li> 
 <li>For <strong>Multi-AZ deployment</strong>, select <strong>Multi-AZ option</strong>.</li> 
 <li>For <strong>Number of nodes per AZ</strong>, enter the number of nodes that you need for your cluster.</li> 
</ol> 
<p><img alt="" class="alignnone size-full wp-image-55810" height="163" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image008.png" style="margin: 10px 0px 10px 0px;" width="1431" /></p> 
<p><img alt="" class="alignnone wp-image-55811 size-full" height="1403" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image010.png" style="margin: 10px 0px 10px 0px;" title="create preview cluster" width="1641" /></p> 
<ol> 
 <li>Under the <strong>Database configurations</strong>, choose <strong>Admin user name</strong> and <strong>Admin user password</strong>.</li> 
 <li>Turn <strong>Use defaults </strong>on next to <strong>Additional configurations</strong> to modify the default settings.</li> 
 <li>Under <strong>Network and security</strong>, specify the following: 
  <ol type="a"> 
   <li>For <strong>Virtual private cloud (VPC)</strong>, choose the VPC you want to deploy the cluster in.</li> 
   <li>For <strong>VPC security groups</strong>, either leave as default or add the security groups of your choice.</li> 
   <li>For <strong>Cluster subnet group, </strong>either leave as default or add a cluster subnet group of your choice. For a Multi-AZ deployment, a cluster subnet group must include one subnet each from at least three or more different Availability Zones.</li> 
  </ol> </li> 
</ol> 
<p>For general information about managing cluster subnet groups, see&nbsp;<a href="https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-cluster-subnet-groups.html" rel="noopener" target="_blank">Cluster subnet groups</a></p> 
<p><img alt="" class="alignnone wp-image-55812 size-full" height="1416" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image012.png" style="margin: 10px 0px 10px 0px;" title="additional configurations" width="1646" /></p> 
<ol start="9"> 
 <li>Under <strong>Database configuration</strong>, for <strong>Database port</strong>, you either use the default value 5439 or choose a value from the range of 5431–5455 and 8191–8215.</li> 
 <li>Under <strong>Database configuration</strong>, in the <strong>Database encryption</strong> section, to use a custom <a href="http://aws.amazon.com/kms" rel="noopener" target="_blank">AWS Key Management Service</a> (AWS KMS) key other than the default KMS key, choose <strong>Customize encryption settings</strong>. This option is deselected by default.</li> 
 <li>Under <strong>Choose an AWS KMS key</strong>, you can either choose an existing KMS key, or choose <strong>Create an AWS KMS key</strong> to create a new KMS key.</li> 
</ol> 
<p>For more information to create key using KMS, refer to <a href="https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html" rel="noopener" target="_blank">Creating keys</a>.</p> 
<p><img alt="" class="alignnone wp-image-55813 size-full" height="747" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image014.png" style="margin: 10px 0px 10px 0px;" title="database configurations" width="647" /></p> 
<ol start="12"> 
 <li>Choose <strong>Create cluster</strong>.</li> 
</ol> 
<p>When the cluster creation succeeds, you can view the details on the cluster details page.</p> 
<p>Under <strong>General information</strong>, you can see <strong>Multi-AZ</strong> as <strong>Yes</strong>.</p> 
<p><img alt="" class="alignnone wp-image-55814 size-full" height="683" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image016.png" style="margin: 10px 0px 10px 0px;" title="general information" width="1429" /></p> 
<p>On the <strong>Properties </strong>tab, under <strong>Network and security settings</strong>, you can find the details on the primary and secondary Availability Zone.</p> 
<p><img alt="" class="alignnone wp-image-55815 size-full" height="591" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image018.png" style="margin: 10px 0px 10px 0px;" title="network and security settings" width="1429" /></p> 
<h2>Create a new Multi-AZ deployment from the CLI</h2> 
<p>The following create-cluster AWS CLI command shows how to create a Multi-AZ cluster</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws redshift create-cluster 
--port 5439 
    --master-username master
    --master-user-password ######
    --node-type ra3.4xlarge
    --number-of-nodes 2
    --profile maz-test
    --endpoint-url https://redshift.us-east-1.amazonaws.com
    --region eu-west-1
    --cluster-identifier redshift-cluster-1
    --multi-az 
    --maintenance-track-name CURRENT
    --encrypted</code></pre> 
</div> 
<h2>Convert a Single-AZ deployment to Multi-AZ deployment</h2> 
<p>To convert an existing Single-AZ deployment to a Multi-AZ deployment, you can go to the Redshift console and select your Redshift cluster that currently is Single-AZ setup and navigate to Actions and select <strong>Activate Multi-AZ</strong>. Your Single-AZ cluster must be encrypted for a successful conversion to Multi-AZ. During conversion to Multi-AZ, Redshift will double the total number of nodes distributing them equally in each AZ. Redshift will not allow you to split existing number of nodes while converting to Multi-AZ to maintain consistent query performance.</p> 
<p>Complete the following steps to create a Multi-AZ deployment restored from a snapshot:</p> 
<ol> 
 <li>On the Amazon Redshift console, in the navigation pane, choose <strong>Clusters</strong>.</li> 
 <li>Select your cluster and navigate to the cluster details page.</li> 
 <li>On the <strong>Actions </strong>menu, choose<strong> Activate Multi-AZ.</strong></li> 
</ol> 
<p><img alt="" class="alignnone wp-image-55816 size-full" height="725" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image020.png" style="margin: 10px 0px 10px 0px;" title="restore snapshot" width="1429" /></p> 
<ol start="4"> 
 <li>Review the modification summary and confirm by choosing <strong>Activate Multi-AZ</strong>.</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-55817 size-full" height="805" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image022.png" style="margin: 10px 0px 10px 0px;" title="cluster identifier" width="897" /></p> 
<p>Using the below AWS CLI command you can convert a single AZ Redshift data warehouse to Multi-AZ.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws redshift modify-cluster 
    --profile maz-test
    --endpoint-url https://redshift.eu-west-1.amazonaws.com
    --region eu-west-1
    --cluster-identifier redshift-cluster-1
    --multi-az</code></pre> 
</div> 
<h2>Convert a Multi-AZ deployment to Single-AZ deployment</h2> 
<p>Redshift also supports conversion of a Multi-AZ deployment into Single-AZ.&nbsp; This option provides customers with the flexibility to switch between different deployments with few easy steps as follows:</p> 
<ol> 
 <li>On the Amazon Redshift console, in the navigation pane, choose <strong>Clusters</strong>.</li> 
 <li>Select your cluster and navigate to the cluster details page.</li> 
 <li>On the <strong>Actions </strong>menu, choose<strong> Deactivate Multi-AZ.<br /> <img alt="" class="alignnone size-full wp-image-55818" height="742" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image024.png" style="margin: 10px 0px 10px 0px;" width="1430" /><br /> </strong></li> 
 <li>Review the modification summary and confirm by choosing <strong>Deactivate Multi-AZ</strong>.<br /> <img alt="" class="alignnone size-full wp-image-55819" height="900" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image026.png" style="margin: 10px 0px 10px 0px;" width="893" /></li> 
</ol> 
<h2>Creating a Multi-AZ data warehouse restored from a snapshot</h2> 
<p>Existing customers can also create a Multi-AZ deployment by restoring a snapshot from an existing Single-AZ deployment. See the required steps as below.</p> 
<ol> 
 <li>On the Amazon Redshift console, in the navigation pane, choose <strong>Clusters</strong>.</li> 
 <li>Select the cluster and navigate to the cluster details page.</li> 
 <li>Choose the <strong>Maintenance</strong></li> 
 <li>Select a snapshot and choose <strong>Restore snapshot</strong>, <strong>Restore to provisioned cluster</strong>.<br /> <img alt="" class="alignnone size-full wp-image-55820" height="378" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image028.png" style="margin: 10px 0px 10px 0px;" width="1431" /></li> 
 <li>Review the&nbsp;<strong>Cluster configuration </strong>and&nbsp;<strong>Cluster details</strong>&nbsp;values of the new cluster to be created using the snapshot information.</li> 
 <li>Select Multi-AZ option and update the properties of the new cluster, then choose <strong>Restore cluster from snapshot</strong> at the bottom of the page.<br /> <img alt="" class="alignnone size-full wp-image-55821" height="1432" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image030.png" style="margin: 10px 0px 10px 0px;" width="1626" /></li> 
</ol> 
<h2>Resizing a Multi-AZ data warehouse</h2> 
<p>Redshift Multi-AZ feature also supports resizing Multi-AZ Redshift cluster deployments to change the cluster configuration based on scaling needs. You can change both number and type of nodes as per needs.</p> 
<ol> 
 <li>On the Amazon Redshift console, in the navigation pane, choose <strong>Clusters</strong>.</li> 
 <li>Select your cluster and navigate to the cluster details page.</li> 
 <li>On the <strong>Actions </strong>menu, choose</li> 
 <li>Once selected it will bring into another screen to show cluster resize screen where you can choose type and number of nodes and click on <strong>Resize cluster</strong>.</li> 
</ol> 
<p><img alt="" class="alignnone size-full wp-image-55822" height="838" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image032.png" style="margin: 10px 0px 10px 0px;" width="892" /> <img alt="" class="alignnone size-full wp-image-55823" height="915" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image034.png" style="margin: 10px 0px 10px 0px;" width="818" /></p> 
<h2>Failing over Multi-AZ deployment</h2> 
<p>In addition to the automatic recovery process, you can also trigger this process manually for your data warehouse using the Failover primary compute option. This approach can be used to manage operational maintenance and other planned operational procedures as per the needs of the respective environment. When the cluster successfully recovers, Multi-AZ deployment becomes available. Your Multi-AZ deployment also automatically provisions new compute nodes in another Availability Zone as soon as it is available.</p> 
<p>Let’s manually trigger the Failover of your Redshift Multi-AZ deployment.</p> 
<ol> 
 <li>On the Amazon Redshift console, choose <strong>Clusters</strong> in the navigation pane.</li> 
 <li>Navigate to the cluster detail page</li> 
 <li>From&nbsp;<strong>Actions</strong>, choose&nbsp;<strong>Failover primary compute</strong>.</li> 
 <li>When prompted, choose <strong>Confirm</strong>.</li> 
</ol> 
<p><img alt="" class="alignnone size-full wp-image-55824" height="752" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image036.png" style="margin: 10px 0px 10px 0px;" width="1730" /><br /> <img alt="" class="alignnone size-full wp-image-55825" height="420" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image038.png" style="margin: 10px 0px 10px 0px;" width="713" /></p> 
<p>After the cluster is back to <code>Available</code> status, you can observe that the primary and secondary Availability Zones have changed.</p> 
<p>The following screenshot shows the status before injecting failure.<br /> <img alt="" class="alignnone size-full wp-image-55826" height="387" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image040.png" style="margin: 10px 0px 10px 0px;" width="936" /></p> 
<p>The following screenshot shows the status after injecting failure.<br /> <img alt="" class="alignnone size-full wp-image-55827" height="386" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image042.png" style="margin: 10px 0px 10px 0px;" width="936" /></p> 
<h2>Restore a table from snapshot</h2> 
<p>You can restore a single table from a snapshot from your Multi-AZ cluster. When you restore a single table from a snapshot, you specify the source snapshot, database, schema, and table name, and the target database, schema, and a new table name for the restored table.</p> 
<p>To restore a table from a snapshot:</p> 
<ol> 
 <li>On the Amazon Redshift console, in the navigation pane, choose <strong>Clusters</strong>.</li> 
 <li>Select your cluster and navigate to the cluster details page.</li> 
 <li>On the <strong>Actions </strong>menu, choose<strong> Restore table.</strong></li> 
 <li>Enter the information about which snapshot, source table, and target table to use, and then choose&nbsp;Restore table.</li> 
</ol> 
<p><img alt="" class="alignnone size-full wp-image-55828" height="687" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image043.png" style="margin: 10px 0px 10px 0px;" width="1431" /></p> 
<h2>Enable public connections for your Multi-AZ data warehouse</h2> 
<ol> 
 <li>From the navigation menu, choose CLUSTERS.</li> 
 <li>Choose the Multi-AZ cluster that you want to modify.</li> 
 <li>Choose Actions.</li> 
 <li>Choose Turn on Publicly accessible.</li> 
 <li>Choose Elastic IP address, if you do not choose one, an address will be randomly assigned to you.</li> 
 <li>Choose Save changes.</li> 
</ol> 
<p><img alt="" class="alignnone size-full wp-image-55829" height="756" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image045.png" style="margin: 10px 0px 10px 0px;" width="1428" /><br /> <img alt="" class="alignnone size-full wp-image-55830" height="1226" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image047.png" style="margin: 10px 0px 10px 0px;" width="1478" /></p> 
<h2>Monitor queries for Multi-AZ deployments</h2> 
<p>A Multi-AZ deployment uses compute resources that are deployed in both Availability Zones and can continue operating in the event that the resources in a given Availability Zone are not available. All the compute resources are used at all times, which allows full operation across two Availability Zones in both read and write operations.</p> 
<p>You can query <code>SYS_views</code> in the <code>pg_catalog</code> schema to monitor Multi-AZ query runs. The <code>SYS_views</code> cover query run activities and stats from primary and secondary clusters.</p> 
<p>The following are the system tables in the <code>SYS_view</code> list:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_QUERY_HISTORY.html" rel="noopener" target="_blank">SYS_QUERY_HISTORY</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_QUERY_DETAIL.html" rel="noopener" target="_blank">SYS_QUERY_DETAIL</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_CONNECTION_LOG.html" rel="noopener" target="_blank">SYS_CONNECTION_LOG</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_DATASHARE_CHANGE_LOG.html" rel="noopener" target="_blank">SYS_DATASHARE_CHANGE_LOG</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_DATASHARE_USAGE_CONSUMER.html" rel="noopener" target="_blank">SYS_DATASHARE_USAGE_CONSUMER</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_DATASHARE_USAGE_PRODUCER.html" rel="noopener" target="_blank">SYS_DATASHARE_USAGE_PRODUCER</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_EXTERNAL_QUERY_DETAIL.html" rel="noopener" target="_blank">SYS_EXTERNAL_QUERY_DETAIL</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_LOAD_HISTORY.html" rel="noopener" target="_blank">SYS_LOAD_HISTORY</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_LOAD_ERROR_DETAIL.html" rel="noopener" target="_blank">SYS_LOAD_ERROR_DETAIL</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_UNLOAD_HISTORY.html" rel="noopener" target="_blank">SYS_UNLOAD_HISTORY</a></li> 
</ul> 
<p>Follow these steps to monitor the query run on Multi-AZ deployment from the Amazon Redshift Console:</p> 
<ol> 
 <li>On the Amazon Redshift console, connect to the database in your Multi-AZ deployment and <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-databases.html" rel="noopener" target="_blank">run queries</a> through the query editor.</li> 
 <li>Run any sample query on the Multi-AZ Redshift deployment.</li> 
 <li>For a Multi-AZ deployment, you can identify a query and the Availability Zone where it is being run (running on the primary or secondary availability zone) by using the <code>compute_type</code> column in the <code>SYS_QUERY_HISTORY</code> table. The valid values for the compute type column are as follows: 
  <ol type="a"> 
   <li><strong>primary</strong> – When run on primary availability zone in the Multi-AZ deployment.</li> 
   <li><strong>secondary</strong> – When run on secondary availability zone in the Multi-AZ deployment.</li> 
  </ol> </li> 
</ol> 
<p>The following is a sample query using the <code>compute_type</code> column to monitor a query:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">dev=# select (compute_type) as compute_type, left(query_text, 50) query_text from sys_query_history order by start_time desc;

 compute_type | query_text
--------------+----------------------------------------------------
 secondary    | select count(*) from t1;
 primary 	   select count(*) from t2;</code></pre> 
</div> 
<p>You can also access the query history from the console to analyze your query diagnostics.</p> 
<ol start="4"> 
 <li>On the <strong>Query monitoring</strong> tab, choose <strong>Connect to database.</strong></li> 
</ol> 
<p><img alt="" class="alignnone wp-image-55831 size-full" height="406" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image049.png" style="margin: 10px 0px 10px 0px;" title="query monitoring" width="1428" /></p> 
<ol start="5"> 
 <li>For <strong>Connection</strong>, choose <strong>Create a new connection</strong></li> 
 <li>For<strong> Authentication</strong>, choose<strong> Temporary credentials</strong></li> 
 <li>For <strong>Database name</strong>, enter the database name (for example, <code>dev</code>).</li> 
 <li>For <strong>Database user</strong>, enter the database user name (for example, <code>awsuser</code>).</li> 
 <li>Choose<strong> Connect</strong>.</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-55832 size-full" height="1103" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image051.png" style="margin: 10px 0px 10px 0px;" title="connect to database" width="1432" /></p> 
<p>After you’re connected, under <strong>Query Monitoring</strong>, on the<strong> Query history tab</strong>, you can view all the queries and loads, as shown in the following screenshot.</p> 
<p><img alt="" class="alignnone wp-image-55833 size-full" height="522" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image053.png" style="margin: 10px 0px 10px 0px;" title="queries and loads" width="1429" /></p> 
<p>Under <strong>Metric filters</strong>, you can use the various filters in the <strong>Additional filtering options</strong> section to view query history based on <strong>Time interval</strong>, <strong>Users</strong>, <strong>Databases</strong>, or<strong> SQL commands</strong>.</p> 
<p><img alt="" class="alignnone wp-image-55834 size-full" height="475" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image055.png" style="margin: 10px 0px 10px 0px;" title="metric filters" width="1429" /></p> 
<p>There are a few limitations when working with Amazon Redshift Multi-AZ in preview mode, refer <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/managing-cluster-multi-az.html" rel="noopener" target="_blank">here</a> for the limitations.</p> 
<h2>Customer feedback</h2> 
<table> 
 <tbody> 
  <tr> 
   <td style="width: 30%;"><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/08/janssen.png" rel="noopener" target="_blank"><img alt="" class="aligncenter wp-image-39707 size-full" height="212" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/08/janssen.png" width="487" /></a></td> 
   <td><a href="https://www.janssen.com/" rel="noopener" target="_blank">Janssen Pharmaceuticals</a>, a subsidiary of Johnson &amp; Johnson, researches and manufactures medicines with a focus on the changing needs of patients and the healthcare industry.<p></p> 
    <blockquote>
     <p><em>“Janssen Pharmaceutical uses Amazon Redshift to enable critical insights that drive important business decisions for our data scientists, data stewards, business users, and external stakeholders. With Amazon Redshift Multi-AZ, we can be confident that our data warehouse will always be available without any disruptions that might delay impact our ability to make critical business decisions.”</em></p>
    </blockquote> <p><strong>– Shyam Mohapatra, Director of Information Technology – Janssen Pharmaceutical Companies of Johnson &amp; Johnson</strong></p></td> 
  </tr> 
  <tr> 
   <td style="width: 30%;"><img alt="" class="alignnone size-full wp-image-55835" height="101" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/23/BDB-2849-image059.png" width="201" /></td> 
   <td><a href="https://stripe.com/" rel="noopener" target="_blank">Stripe</a> is a technology company that builds economic infrastructure for the internet. Stripe’s products power payments for online and in-person retailers, subscriptions businesses, software platforms and marketplaces, and everything in between.<p></p> <p style="margin-left: .5in;">“<i>Millions of companies use Stripe’s software and APIs to accept payments, send payouts, and manage their businesses online.&nbsp; Access to&nbsp;their&nbsp;Stripe data via leading data warehouses like Amazon Redshift has been a top request from our customers. Our customers needed highly available, secure, fast, and integrated analytics at scale without building complex data pipelines or moving and copying data around. With Stripe Data Pipeline for Amazon Redshift, we’re helping our customers set up a direct and reliable data pipeline in a few clicks.&nbsp;</i></p> <p style="margin-left: .5in;"><i>Stripe Data Pipeline enables our customers to automatically share their complete, up-to-date Stripe data with their Amazon Redshift data warehouse, and take their business analytics and reporting to the next level</i>.”</p> <p><b>– Brian Brunner,</b>&nbsp;<strong>Senior Manager, Engineering at Stripe</strong></p></td> 
  </tr> 
 </tbody> 
</table> 
<h2>Conclusion</h2> 
<p>This post demonstrated how to configure an Amazon Redshift Multi-AZ deployment in two Availability Zones and test the fault tolerance of your workloads during an unlikely failure of an Availability Zone. Amazon Redshift Multi-AZ deployment also helps improve overall performance of your data warehouse because compute nodes in both Availability Zones are used for read and write operations. Amazon Redshift Multi-AZ data warehouse helps meet the demands of customers with mission critical analytics applications that require the highest levels of availability and resiliency. For more details, refer <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/managing-cluster-multi-az.html" rel="noopener" target="_blank">Configuring Multi-AZ deployment</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><img alt="" class="size-full wp-image-38981 alignleft" height="94" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image039-2.jpg" width="100" /> <strong>Ranjan Burman</strong> is an Analytics Specialist Solutions Architect at AWS. He specializes in Amazon Redshift and helps customers build scalable analytical solutions. He has more than 16 years of experience in different database and data warehousing technologies. He is passionate about automating and solving customer problems with cloud solutions.</p> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-39737 size-full" height="143" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/08/saurav-das-100.jpg" width="100" /><strong>Saurav Das </strong>is part of the Amazon Redshift Product Management team. He has more than 16 years of experience in working with relational databases technologies and data protection. He has a deep interest in solving customer challenges centered around high availability and disaster recovery.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/10/13/Anusha-Challa.jpg" rel="noopener" target="_blank"><img alt="" class="size-full wp-image-22677 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/10/13/Anusha-Challa.jpg" width="100" /></a>Anusha Challa</strong> is a Senior Analytics Specialist Solutions Architect focused on Amazon Redshift. She has helped many customers build large-scale data warehouse solutions in the cloud and on premises. She is passionate about data analytics and data science.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/12/09/Nita-Shah.jpg" rel="noopener" target="_blank"><img alt="" class="size-full wp-image-24642 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/12/09/Nita-Shah.jpg" width="100" /></a>Nita Shah</strong> is an Analytics Specialist Solutions Architect at AWS based out of New York. She has been building data warehouse solutions for over 20 years and specializes in Amazon Redshift. She is focused on helping customers design and build enterprise-scale well-architected analytics and decision support platforms.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/09/20/Suresh-Patnam.png" rel="noopener" target="_blank"><img alt="" class="size-full wp-image-22057 alignleft" height="135" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/09/20/Suresh-Patnam.png" width="105" /></a>Suresh Patnam</strong> is a Principal BDM – GTM AI/ML Leader at AWS. He works with customers to build IT strategy, making digital transformation through the cloud more accessible by using data and AI/ML. In his spare time, Suresh enjoys playing tennis and spending time with his family.</p>
