---
title: "Simplify data streaming ingestion for analytics using Amazon MSK and Amazon Redshift"
url: "https://aws.amazon.com/blogs/big-data/simplify-data-streaming-ingestion-for-analytics-using-amazon-msk-and-amazon-redshift/"
date: "Wed, 21 Feb 2024 20:32:14 +0000"
author: "Sebastian Vlad"
feed_url: "https://aws.amazon.com/blogs/big-data/category/database/amazon-redshift/feed/"
---
<p>Towards the end of 2022, <a href="https://aws.amazon.com/about-aws/whats-new/2022/11/amazon-redshift-real-time-streaming-ingestion-kds-msk/" rel="noopener" target="_blank">AWS announced the general availability of real-time streaming ingestion</a> to <a href="https://aws.amazon.com/redshift/" rel="noopener" target="_blank">Amazon Redshift</a> for <a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener" target="_blank">Amazon Kinesis Data Streams</a> and <a href="https://aws.amazon.com/msk/" rel="noopener" target="_blank">Amazon Managed Streaming for Apache Kafka (Amazon MSK)</a>, eliminating the need to stage streaming data in <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> before ingesting it into Amazon Redshift.</p> 
<p><a href="https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-streaming-ingestion.html" rel="noopener" target="_blank">Streaming ingestion</a> from Amazon MSK into Amazon Redshift, represents a cutting-edge approach to real-time data processing and analysis. Amazon MSK serves as a highly scalable, and fully managed service for Apache Kafka, allowing for seamless collection and processing of vast streams of data. Integrating streaming data into Amazon Redshift brings immense value by enabling organizations to harness the potential of real-time analytics and data-driven decision-making.</p> 
<p>This integration enables you to achieve low latency, measured in seconds, while ingesting hundreds of megabytes of streaming data per second into Amazon Redshift. At the same time, this integration helps make sure that the most up-to-date information is readily available for analysis. Because the integration doesn’t require staging data in Amazon S3, Amazon Redshift can ingest streaming data at a lower latency and without intermediary storage cost.</p> 
<p>You can configure Amazon Redshift streaming ingestion on a Redshift cluster using SQL statements to authenticate and connect to an MSK topic. This solution is an excellent option for data engineers that are looking to simplify data pipelines and reduce the operational cost.</p> 
<p>In this post, we provide a complete overview on how to configure <a href="https://aws.amazon.com/redshift/redshift-streaming-ingestion/" rel="noopener" target="_blank">Amazon Redshift streaming ingestion</a> from Amazon MSK.</p> 
<h2>Solution overview</h2> 
<p>The following architecture diagram describes the AWS services and features you will be using.</p> 
<p><img alt="architecture diagram describing the AWS services and features you will be using" class="alignnone wp-image-60012 size-full" height="330" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/13/001_high_level_solution_architecture_BDB-3239-1.png" width="2203" /></p> 
<p>The workflow includes the following steps:</p> 
<ol> 
 <li>You start with configuring an <a href="https://aws.amazon.com/msk/features/msk-connect/" rel="noopener" target="_blank">Amazon MSK Connect</a> source connector, to create an MSK topic, generate mock data, and write it to the MSK topic. For this post, we work with mock customer data.</li> 
 <li>The next step is to connect to a Redshift cluster using the <a href="https://aws.amazon.com/redshift/query-editor-v2/" rel="noopener" target="_blank">Query Editor v2</a>.</li> 
 <li>Finally, you configure an external schema and create a materialized view in Amazon Redshift, to consume the data from the MSK topic. This solution does not rely on an MSK Connect sink connector to export the data from Amazon MSK to Amazon Redshift.</li> 
</ol> 
<p>The following solution architecture diagram describes in more detail the configuration and integration of the AWS services you will be using.<br /> <img alt="solution architecture diagram describing in more detail the configuration and integration of the AWS services you will be using" class="alignnone wp-image-60011 size-full" height="1083" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/13/002_detailed_solution_architecture_BDB-3239-1.png" width="2358" /><br /> The workflow includes the following steps:</p> 
<ol> 
 <li>You deploy an MSK Connect source connector, an MSK cluster, and a Redshift cluster within the private subnets on a VPC.</li> 
 <li>The MSK Connect source connector uses granular permissions defined in an <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#inline-policies" rel="noopener" target="_blank">in-line policy</a> attached to an <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html" rel="noopener" target="_blank">IAM role</a>, which allows the source connector to perform actions on the MSK cluster.</li> 
 <li>The MSK Connect source connector logs are captured and sent to an <a href="https://aws.amazon.com/cloudwatch/" rel="noopener" target="_blank">Amazon CloudWatch</a> <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Working-with-log-groups-and-streams.html" rel="noopener" target="_blank">log group</a>.</li> 
 <li>The MSK cluster uses a <a href="https://docs.aws.amazon.com/msk/latest/developerguide/msk-configuration-properties.html" rel="noopener" target="_blank">custom MSK cluster configuration</a>, allowing the MSK Connect connector to create topics on the MSK cluster.</li> 
 <li>The MSK cluster logs are captured and sent to an Amazon CloudWatch log group.</li> 
 <li>The Redshift cluster uses granular permissions defined in an IAM in-line policy attached to an IAM role, which allows the Redshift cluster to perform actions on the MSK cluster.</li> 
 <li>You can use the Query Editor v2 to connect to the Redshift cluster.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>To simplify the provisioning and configuration of the prerequisite resources, you can use the following <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a> template:</p> 
<p style="padding-left: 25px;"><a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create?stackName=prerequisites&amp;templateURL=https://aws-blogs-artifacts-public.s3.amazonaws.com/artifacts/BDB-3239/RedshiftStreamingIngestionPrerequisites.json" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-51486 size-full" height="22" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/07/28/LaunchStack.png" width="110" /></a></p> 
<p>Complete the following steps when launching the stack:</p> 
<ol> 
 <li>For <strong>Stack name</strong>, enter a meaningful name for the stack, for example, <code>prerequisites</code>.</li> 
 <li>Choose <strong>Next.</strong></li> 
 <li>Choose<strong> Next.</strong></li> 
 <li>Select&nbsp;<b>I acknowledge that AWS CloudFormation might create IAM resources with custom names.</b></li> 
 <li>Choose <strong>Submit.</strong></li> 
</ol> 
<p>The CloudFormation stack creates the following resources:</p> 
<ul> 
 <li>A VPC <code>custom-vpc</code>, created across three Availability Zones, with three <a href="https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html#subnet-types" rel="noopener" target="_blank">public subnets</a> and three <a href="https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html#subnet-types" rel="noopener" target="_blank">private subnets</a>: 
  <ul> 
   <li>The public subnets are associated with a public route table, and outbound traffic is directed to an internet gateway.</li> 
   <li>The private subnets are associated with a private route table, and outbound traffic is sent to a NAT gateway.</li> 
  </ul> </li> 
 <li>An <a href="https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html" rel="noopener" target="_blank">internet gateway</a> attached to the Amazon VPC.</li> 
 <li>A <a href="https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html" rel="noopener" target="_blank">NAT gateway</a> that is associated with an <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html" rel="noopener" target="_blank">elastic IP</a> and is deployed in one of the public subnets.</li> 
 <li>Three <a href="https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html" rel="noopener" target="_blank">security groups</a>: 
  <ul> 
   <li><code>msk-connect-sg</code>, which will be later associated with the MSK Connect connector.</li> 
   <li><code>redshift-sg</code>, which will be later associated with the Redshift cluster.</li> 
   <li><code>msk-cluster-sg</code>, which will be later associated with the MSK cluster. It allows inbound traffic from <code>msk-connect-sg</code>, and <code>redshift-sg</code><strong><em>.</em></strong></li> 
  </ul> </li> 
 <li>Two CloudWatch log groups: 
  <ul> 
   <li><code>msk-connect-logs</code>, to be used for the MSK Connect logs.</li> 
   <li><code>msk-cluster-logs</code>, to be used for the MSK cluster logs.</li> 
  </ul> </li> 
 <li>Two IAM Roles: 
  <ul> 
   <li><code>msk-connect-role</code>, which includes granular IAM permissions for MSK Connect.</li> 
   <li><code>redshift-role</code>, which includes granular IAM permissions for Amazon Redshift.</li> 
  </ul> </li> 
 <li>A <a href="https://docs.aws.amazon.com/msk/latest/developerguide/msk-configuration-properties.html" rel="noopener" target="_blank">custom MSK cluster configuration</a>, allowing the MSK Connect connector to create topics on the MSK cluster.</li> 
 <li>An MSK cluster, with three brokers deployed across the three private subnets of <code>custom-vpc</code><em>.</em> The <code>msk-cluster-sg</code> security group and the <code>custom-msk-cluster-configuration</code> configuration are applied to the MSK cluster. The broker logs are delivered to the <code>msk-cluster-logs</code> CloudWatch log group.</li> 
 <li>A <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-cluster-subnet-groups.html" rel="noopener" target="_blank">Redshift cluster subnet group</a>, which is using the three private subnets of <code>custom-vpc</code>.</li> 
 <li>A Redshift cluster, with one single node deployed in a private subnet within the Redshift cluster subnet group. The <code>redshift-sg</code> security group and <code>redshift-role</code> IAM role are applied to the Redshift cluster.</li> 
</ul> 
<h2>Create an MSK Connect custom plugin</h2> 
<p>For this post, we use an <a href="https://github.com/awslabs/amazon-msk-data-generator" rel="noopener" target="_blank">Amazon MSK data generator</a> deployed in MSK Connect, to generate mock customer data, and write it to an MSK topic.</p> 
<p>Complete the following steps:</p> 
<ol> 
 <li>Download the <a href="https://github.com/awslabs/amazon-msk-data-generator/releases" rel="noopener" target="_blank">Amazon MSK data generator</a> JAR file with dependencies from GitHub.<br /> <img alt="awslabs github page for downloading the jar file of the amazon msk data generator" class="alignnone wp-image-59429 " height="558" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/004_download_JAR_from_GitHub_BDB-3239.png" style="margin: 10px 0px 10px 0px;" width="949" /></li> 
 <li>Upload the JAR file into an S3 bucket in your AWS account.<br /> <img alt="amazon s3 console image showing the uploaded jar file in an s3 bucket" class="alignnone wp-image-59428 " height="439" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/005_upload_JAR_to_S3_BDB-3239.png" style="margin: 10px 0px 10px 0px;" width="947" /></li> 
 <li>On the Amazon MSK console, choose <strong>Custom plugins </strong>under <strong>MSK Connect</strong> in the navigation pane.</li> 
 <li>Choose <strong>Create custom plugin.</strong></li> 
 <li>Choose <strong>Browse S3</strong>, search for the Amazon MSK data generator JAR file you uploaded to Amazon S3, then choose <strong>Choose</strong>.</li> 
 <li>For <strong>Custom plugin name</strong>, enter&nbsp;<code>msk-datagen-plugin</code>.</li> 
 <li>Choose&nbsp;<strong>Create custom plugin.</strong></li> 
</ol> 
<p>When the custom plugin is created, you will see that its status is <strong>Active</strong>, and you can move to the next step.<br /> <img alt="amazon msk console showing the msk connect custom plugin being successfully created" class="alignnone wp-image-59427 " height="376" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/006_plugin_created_BDB-3239.png" style="margin: 10px 0px 10px 0px;" width="1001" /></p> 
<h2>Create an MSK Connect connector</h2> 
<p>Complete the following steps to create your connector:</p> 
<ol> 
 <li>On the Amazon MSK console, choose <strong>Connectors </strong>under <strong>MSK Connect</strong> in the navigation pane.</li> 
 <li>Choose<strong> Create connector.</strong></li> 
 <li>For <strong>Custom plugin type</strong>, choose <strong>Use existing plugin.</strong></li> 
 <li>Select <code>msk-datagen-plugin</code>, then choose <strong>Next.</strong></li> 
 <li>For <strong>Connector name</strong>, enter<em>&nbsp;</em><code>msk-datagen-connector</code>.</li> 
 <li>For <strong>Cluster type</strong>, choose <strong>Self-managed Apache Kafka cluster.</strong></li> 
 <li>For <strong>VPC</strong>, choose <code>custom-vpc</code><strong>.</strong></li> 
 <li>For <strong>Subnet 1</strong>, choose the private subnet within your first Availability Zone.</li> 
</ol> 
<p>For the <code>custom-vpc</code> created by the CloudFormation template, we are using odd CIDR ranges for public subnets, and even CIDR ranges for the private subnets:</p> 
<ul> 
 <li> 
  <ul> 
   <li>The CIDRs for the public subnets are 10.10.1.0/24, 10.10.3.0/24, and 10.10.5.0/24</li> 
   <li>The CIDRs for the private subnets are 10.10.2.0/24, 10.10.4.0/24, and 10.10.6.0/24</li> 
  </ul> </li> 
</ul> 
<ol start="9"> 
 <li>For <strong>Subnet 2</strong>, select the private subnet within your second Availability Zone.</li> 
 <li>For <strong>Subnet 3</strong>, select the private subnet within your third Availability Zone.</li> 
 <li>For <strong>Bootstrap servers</strong>, enter the list of bootstrap servers for TLS authentication of your MSK cluster.</li> 
</ol> 
<p>To <a href="https://docs.aws.amazon.com/msk/latest/developerguide/msk-get-bootstrap-brokers.html#get-bootstrap-console" rel="noopener" target="_blank">retrieve the bootstrap servers for your MSK cluster</a>, navigate to the Amazon MSK console, choose <strong>Clusters</strong>, choose <code>msk-cluster</code>, then choose <strong>View client information</strong>. Copy the TLS values for the bootstrap servers.</p> 
<ol start="12"> 
 <li>For <strong>Security groups</strong>, choose <strong>Use specific security groups with access to this cluster</strong>, and choose <code>msk-connect-sg</code><strong><em>.</em></strong></li> 
 <li>For <strong>Connector configuration</strong>, replace the default settings with the following:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-java">connector.class=com.amazonaws.mskdatagen.GeneratorSourceConnector
tasks.max=2
genkp.customer.with=#{Code.isbn10}
genv.customer.name.with=#{Name.full_name}
genv.customer.gender.with=#{Demographic.sex}
genv.customer.favorite_beer.with=#{Beer.name}
genv.customer.state.with=#{Address.state}
genkp.order.with=#{Code.isbn10}
genv.order.product_id.with=#{number.number_between '101','109'}
genv.order.quantity.with=#{number.number_between '1','5'}
genv.order.customer_id.matching=customer.key
global.throttle.ms=2000
global.history.records.max=1000
value.converter=org.apache.kafka.connect.json.JsonConverter
value.converter.schemas.enable=false</code></pre> 
</div> 
<ol start="14"> 
 <li>For Connector capacity, choose <strong>Provisioned.</strong></li> 
 <li>For <strong>MCU count per worker</strong>, choose&nbsp;<strong>1</strong>.</li> 
 <li>For <strong>Number of workers</strong>, choose<strong>&nbsp;1</strong>.</li> 
 <li>For <strong>Worker configuration</strong>, choose <strong>Use the MSK default configuration</strong>.</li> 
 <li>For <strong>Access permissions</strong>, choose <code>msk-connect-role</code>.</li> 
 <li>Choose <strong>Next.</strong></li> 
 <li>For Encryption, select&nbsp;<strong>TLS encrypted traffic.</strong></li> 
 <li>Choose <strong>Next.</strong></li> 
 <li>For <strong>Log delivery</strong>, choose <strong>Deliver to Amazon CloudWatch Logs</strong>.</li> 
 <li>Choose&nbsp;<strong>Browse</strong>, select&nbsp;<code>msk-connect-logs</code>, and choose&nbsp;<strong>Choose</strong>.</li> 
 <li>Choose <strong>Next.</strong></li> 
 <li>Review and choose <strong>Create connector.</strong></li> 
</ol> 
<p>After the custom connector is created, you will see that its status is <strong>Running</strong>, and you can move to the next step.<br /> <img alt="amazon msk console showing the msk connect connector being successfully created" class="alignnone wp-image-59426 size-full" height="600" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/007_connector_created_BDB-3239.png" style="margin: 10px 0px 10px 0px;" width="1598" /></p> 
<h2>Configure Amazon Redshift streaming ingestion for Amazon MSK</h2> 
<p>Complete the following steps to set up streaming ingestion:</p> 
<ol> 
 <li>Connect to your Redshift cluster using Query Editor v2, and authenticate with the database user name <code>awsuser</code>, and password <code>Awsuser123</code>.</li> 
 <li>Create an external schema from Amazon MSK using the following SQL statement.</li> 
</ol> 
<p>In the following code, enter the values for the <code>redshift-role</code> IAM role, and the <code>msk-cluster</code> <a href="https://docs.aws.amazon.com/msk/latest/developerguide/msk-list-clusters.html" rel="noopener" target="_blank">cluster ARN</a>.</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE EXTERNAL SCHEMA msk_external_schema
FROM MSK
IAM_ROLE '&lt;insert your redshift-role arn&gt;'
AUTHENTICATION iam
CLUSTER_ARN '&lt;insert your msk-cluster arn&gt;';</code></pre> 
</div> 
<ol start="3"> 
 <li>Choose <strong>Run</strong> to run the SQL statement.</li> 
</ol> 
<p><img alt="redshift query editor v2 showing the SQL statement used to create an external schema from amazon msk" class="alignnone wp-image-59425 size-full" height="762" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/008_create_external_schema_BDB-3239.png" style="margin: 10px 0px 10px 0px;" width="1638" /></p> 
<ol start="4"> 
 <li>Create a <a href="https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-overview.html" rel="noopener" target="_blank">materialized view</a> using the following SQL statement:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE MATERIALIZED VIEW msk_mview AUTO REFRESH YES AS
SELECT
    "kafka_partition",
    "kafka_offset",
    "kafka_timestamp_type",
    "kafka_timestamp",
    "kafka_key",
    JSON_PARSE(kafka_value) as Data,
    "kafka_headers"
FROM
    "dev"."msk_external_schema"."customer"</code></pre> 
</div> 
<ol start="5"> 
 <li>Choose <strong>Run</strong> to run the SQL statement.</li> 
</ol> 
<p><img alt="redshift query editor v2 showing the SQL statement used to create a materialized view" class="alignnone wp-image-59424 size-full" height="865" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/009_create_mview_BDB-3239.png" style="margin: 10px 0px 10px 0px;" width="1638" /></p> 
<ol start="6"> 
 <li>You can now query the materialized view using the following SQL statement:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">select * from msk_mview LIMIT 100;</code></pre> 
</div> 
<ol start="7"> 
 <li>Choose <strong>Run</strong> to run the SQL statement.</li> 
</ol> 
<p><img alt="redshift query editor v2 showing the SQL statement used to query the materialized view" class="alignnone wp-image-59423 size-full" height="958" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/010_select_mview_BDB-3239.png" style="margin: 10px 0px 10px 0px;" width="2105" /></p> 
<ol start="8"> 
 <li>To monitor the progress of records loaded via streaming ingestion, you can take advantage of the&nbsp;<a class="c-link" href="https://docs.aws.amazon.com/redshift/latest/dg/r_SYS_STREAM_SCAN_STATES.html" rel="noopener noreferrer" target="_blank">SYS_STREAM_SCAN_STATES</a>&nbsp;monitoring view using the following SQL statement:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">select * from SYS_STREAM_SCAN_STATES;</code></pre> 
</div> 
<ol start="9"> 
 <li>Choose <strong>Run</strong> to run the SQL statement.</li> 
</ol> 
<p><img alt="redshift query editor v2 showing the SQL statement used to query the sys stream scan states monitoring view" class="alignnone wp-image-59422 size-full" height="958" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/011_select_scan_states_BDB-3239.png" style="margin: 10px 0px 10px 0px;" width="2105" /></p> 
<ol start="10"> 
 <li>To monitor errors encountered on records loaded via streaming ingestion, you can take advantage of the&nbsp;<a class="c-link" href="https://docs.aws.amazon.com/redshift/latest/dg/r_SYS_STREAM_SCAN_ERRORS.html" rel="noopener noreferrer" target="_blank">SYS_STREAM_SCAN_ERRORS</a>&nbsp;monitoring view using the following SQL statement:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">select * from SYS_STREAM_SCAN_ERRORS;</code></pre> 
</div> 
<ol start="11"> 
 <li>Choose <strong>Run</strong> to run the SQL statement.<img alt="redshift query editor v2 showing the SQL statement used to query the sys stream scan errors monitoring view" class="alignnone wp-image-59421 size-full" height="958" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/012_select_scan_errors_BDB-3239.png" style="margin: 10px 0px 10px 0px;" width="2105" /></li> 
</ol> 
<h2>Clean up</h2> 
<p>After following along, if you no longer need the resources you created, delete them in the following order to prevent incurring additional charges:</p> 
<ol> 
 <li>Delete the MSK Connect connector <code>msk-datagen-connector</code>.</li> 
 <li>Delete the MSK Connect plugin <code>msk-datagen-plugin</code>.</li> 
 <li>Delete the Amazon MSK data generator JAR file you downloaded, and delete the S3 bucket you created.</li> 
 <li>After you delete your MSK Connect connector, you can delete the CloudFormation template. All the resources created by the CloudFormation template will be automatically deleted from your AWS account.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this post, we demonstrated how to configure Amazon Redshift streaming ingestion from Amazon MSK, with a focus on privacy and security.</p> 
<p>The combination of the ability of Amazon MSK to handle high throughput data streams with the robust analytical capabilities of Amazon Redshift empowers business to derive actionable insights promptly. This real-time data integration enhances the agility and responsiveness of organizations in understanding changing data trends, customer behaviors, and operational patterns. It allows for timely and informed decision-making, thereby gaining a competitive edge in today’s dynamic business landscape.</p> 
<p>This solution is also applicable for customers that are looking to use <a href="https://aws.amazon.com/msk/features/msk-serverless/" rel="noopener" target="_blank">Amazon MSK Serverless</a> and <a href="https://aws.amazon.com/redshift/redshift-serverless/" rel="noopener" target="_blank">Amazon Redshift Serverless</a>.</p> 
<p>We hope this post was a good opportunity to learn more about AWS service integration and configuration. Let us know your feedback in the comments section.</p> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-59469 alignleft" height="150" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/013_sebastian_vlad_BDB-3239.jpeg" width="100" />Sebastian Vlad</strong> is a Senior Partner Solutions Architect with Amazon Web Services, with a passion for data and analytics solutions and customer success. Sebastian works with enterprise customers to help them design and build modern, secure, and scalable solutions to achieve their business outcomes.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-59470 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/014_sharad_pai_BDB-3239.jpeg" width="100" />Sharad Pai</strong> is a Lead Technical Consultant at AWS. He specializes in streaming analytics and helps customers build scalable solutions using Amazon MSK and Amazon Kinesis. He has over 16 years of industry experience and is currently working with media customers who are hosting live streaming platforms on AWS, managing peak concurrency of over 50 million. Prior to joining AWS, Sharad’s career as a lead software developer included 9 years of coding, working with open source technologies like JavaScript, Python, and PHP.</p>
