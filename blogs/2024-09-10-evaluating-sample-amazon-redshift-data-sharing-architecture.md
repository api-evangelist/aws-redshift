---
title: "Evaluating sample Amazon Redshift data sharing architecture using Redshift Test Drive and advanced SQL analysis"
url: "https://aws.amazon.com/blogs/big-data/evaluating-sample-amazon-redshift-data-sharing-architecture-using-redshift-test-drive-and-advanced-sql-analysis/"
date: "Tue, 10 Sep 2024 14:57:16 +0000"
author: "Ayan Majumder"
feed_url: "https://aws.amazon.com/blogs/big-data/category/database/amazon-redshift/feed/"
---
<p>With the launch of <a href="https://aws.amazon.com/redshift/redshift-serverless/" rel="noopener" target="_blank">Amazon Redshift Serverless</a> and the various <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-clusters.html#working-with-clusters-overview" rel="noopener" target="_blank">provisioned instance deployment options</a>, customers are looking for tools that help them determine the most optimal data warehouse configuration to support their Amazon Redshift workloads.</p> 
<p><a href="http://aws.amazon.com/redshift" rel="noopener" target="_blank">Amazon Redshift</a> is a widely used, fully managed, petabyte-scale data warehouse service. Tens of thousands of customers use Amazon Redshift to process exabytes of data every day to power their analytics workloads.</p> 
<p><a href="https://github.com/aws/redshift-test-drive" rel="noopener" target="_blank">Redshift Test Drive</a> is a tool hosted on the GitHub repository that let customers evaluate which data warehouse configurations options are best suited for their workload. The Test Drive <a href="https://github.com/aws/redshift-test-drive/tree/main/core" rel="noopener" target="_blank">Workload Replicator</a> utility consists of scripts that can be used to extract the workload queries from your source warehouse audit logs and replay them on a target warehouse you launched. The Test Drive <a href="https://github.com/aws/redshift-test-drive/tree/main/tools/NodeConfigCompare" rel="noopener" target="_blank">Configuration Comparison</a> utility automates this process by deploying target Amazon Redshift warehouses and orchestrating the replay of the source workload through a combination of <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a> and <a href="https://aws.amazon.com/step-functions/" rel="noopener" target="_blank">AWS StepFunctions</a>.</p> 
<p>Both utilities unload the performance metrics from the replay of the source workload on the target configuration(s) to <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3), which is used as a storage to store the performance metrics. Although the <a href="https://github.com/aws/redshift-test-drive/tree/main/tools/ReplayAnalysis" rel="noopener" target="_blank">Replay Analysis UI</a> and Configuration Comparison utility can provide a preliminary performance comparison, many customers want to dig deeper by analyzing the raw data themselves.</p> 
<p>The walkthrough illustrates an example workload replayed on a single Amazon Redshift data warehouse and data sharing architecture using the Workload Replicator utility, the output of which will be used to evaluate the performance of the workload.</p> 
<h2>Use case overview</h2> 
<p>For the sample use case, we assumed we have an existing 2 x <code>ra3.4xlarge</code> provisioned data warehouse that currently runs extract, transform, and load (ETL), ad hoc, and business intelligence (BI) queries. We’re interested in breaking these workloads apart using data sharing into a 32 base Redshift Processing Unit (RPU) Serverless producer running ETL and a 64 base RPU Serverless consumer running the BI workload. We used Workload Replicator to replay the workload on a replica baseline of the source and target data sharing configuration as specified in the tutorial. The following image shows the process flow.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-arch-dia.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-68180 size-full" height="200" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/26/BDB-3985-arch-dia-resize.png" style="margin: 10px 0px 10px 0px;" width="500" /></a></p> 
<h2>Generating and accessing Test Drive metrics</h2> 
<p>The results of Amazon Redshift Test Drive can be accessed using an external schema for analysis of a replay. Refer to the <a href="https://github.com/aws/redshift-test-drive/tree/main/core" rel="noopener" target="_blank">Workload Replicator README</a> and the <a href="https://github.com/aws/redshift-test-drive/tree/main/tools/NodeConfigCompare" rel="noopener" target="_blank">Configuration Comparison README</a> for more detailed instructions to execute a replay using the respective tool.</p> 
<p>The external schema for analysis is automatically created with the Configuration Comparison utility, in which case you can proceed directly to the SQL analysis in the Deploy the QEv2 SQL Notebook and analyze workload section. If you use Workload Replicator, however, the external schema is not created automatically, and therefore needs to be configured as a prerequisite to the SQL analysis. We demonstrate in the following walkthrough how the external schema can be set up, using sample analysis of the Data Sharing use case.</p> 
<h2>Executing Test Drive Workload Replicator for data sharing</h2> 
<p>To execute Workload Replicator, use <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud</a> (Amazon EC2) to run the automation scripts used to extract the workload from the source.</p> 
<h3>Configure Amazon Redshift Data Warehouse</h3> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/mgmt/managing-clusters-console.html#snapshot-cluster" rel="noopener" target="_blank">Create a snapshot</a> following the guidance in the Amazon Redshift Management Guide.</li> 
 <li>Enable <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/db-auditing-console.html" rel="noopener" target="_blank">audit logging</a> following the guidance in Amazon Redshift Management Guide.</li> 
 <li>Enable the <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/db-auditing.html#db-auditing-enable-logging" rel="noopener" target="_blank">user activity logging</a> of the source cluster following the guidance Amazon Redshift Management Guide.</li> 
</ol> 
<p>Enabling logging requires a change of the parameter group. Audit logging needs to be enabled prior to the workload that will be replayed because this is where the connections and SQL queries of the workload will be extracted from.</p> 
<ol start="4"> 
 <li>Launch the baseline replica from the snapshot by restoring a 2 node <code>ra3.4xlarge</code> <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/managing-snapshots-console.html#snapshot-restore" rel="noopener" target="_blank">provisioned cluster</a> from the snapshot.</li> 
 <li>Launch the producer warehouse by restoring the snapshot to a 32 RPU <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/managing-snapshots-console.html#snapshot-restore-provisioned-to-serverless" rel="noopener" target="_blank">serverless namespace</a>.</li> 
 <li>The consumer should not contain the schema and tables that will be shared from the producer. You can launch the 64 RPU Serverless consumer either from the snapshot and then drop the relevant objects, or you can create a new 64 RPU Serverless consumer warehouse and recreate consumer users.</li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/datashare-creation.html" rel="noopener" target="_blank">Create a datashare</a> from the producer to the consumer and add the relevant objects.</li> 
</ol> 
<p>Data share objects can be read using two mechanisms: using three-part notation (<code>database.schema.table</code>), or by creating an external schema pointing to a shared schema and querying that using two-part notation (<code>external_schema.table</code>). Because we want to seamlessly run the source workload, which uses two-part notation on the local objects, this post demonstrates the latter approach. For each schema shared from the producer, run the following command on the consumer:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE EXTERNAL SCHEMA schema_name 
FROM REDSHIFT DATABASE ‘datashare_database_name’ SCHEMA ‘schema_name’;</code></pre> 
</div> 
<p>Make sure to use the same schema name as the source for the external schema. Also, if any queries are run on the public schema, drop the local public schema first before creating the external equivalent.</p> 
<ol start="8"> 
 <li>Grant usage on the schema for any relevant users.</li> 
</ol> 
<h3>Configure Redshift Test Drive Workload Replicator</h3> 
<ol> 
 <li>Create an S3 bucket to store the artifacts required by the utility (such as the metrics, extracted workload, and output data from running <code>UNLOAD</code> commands).</li> 
 <li>Launch the following three types of EC2 instances using the recommended configuration of <code>m5.8xlarge</code>, 32GB of SSD storage, and Amazon Linux AMI: 
  <ol> 
   <li>Baseline instance</li> 
   <li>Target-producer instance</li> 
   <li>Target-consumer instance</li> 
  </ol> </li> 
</ol> 
<p>Make sure you can connect to the EC2 instance to run the utility.</p> 
<ol start="3"> 
 <li>For each instance, install the required libraries by completing the following steps from the <a href="https://github.com/aws/redshift-test-drive/tree/main/core#:~:text=Install%20Workload%20Replicator%20and%20libraries%20dependencies%20on%20the%20provided%20EC2%20machine" rel="noopener" target="_blank">GitHub repository</a>:<br /> a. 2.i<br /> b. 2.ii (if an ODBC driver should be used—the default is the Amazon Redshift Python driver)<br /> c. 2.iii<br /> d. 2.iv<br /> e. 2.v</li> 
 <li><a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html#roles-creatingrole-service-console" rel="noopener" target="_blank">Create</a> an <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) role for the EC2 instances to access the Amazon Redshift warehouses, to read from the S3 audit logging bucket, and with both read and write access to the new S3 bucket created for storing replay artifacts.</li> 
 <li>If you are going to run COPY and UNLOAD commands, create an IAM role with access to the S3 buckets required, and attach it to the Amazon Redshift warehouses that will execute the load and unload.</li> 
</ol> 
<p>In this example, the IAM role is attached to the baseline replica and producer warehouses because these will be executing the ETL processes. The utility will update <code>UNLOAD</code> commands to unload data to a bucket you define, which as a best practice should be the bucket created for S3 artifacts. Write permissions need to be granted to the Amazon Redshift warehouse for this location.</p> 
<h3>Run Redshift Test Drive Workload Replicator</h3> 
<ol> 
 <li>Run <code>aws configure</code> on the EC2 instances and populate the default Region with the Region the utility is being executed in.</li> 
 <li>Extract only needs to be run once, so connect to the baseline EC2 instance and run <code>vi config/extract.yaml</code> to open the <code>extract.yaml</code> file and configure the extraction details (select <code>i</code> to begin configuring elements, then use <code>escape</code> to leave edit mode and <code>:wq!</code> to leave vi). For more details on the parameters, see <a href="https://github.com/aws/redshift-test-drive/tree/main/core#configure-parameters" rel="noopener" target="_blank">Configure parameters</a>.</li> 
</ol> 
<p>The following code is an example of a configured extract that unloads the logs for a half hour window to the Test Drive artifacts bucket and updates <code>COPY</code> commands to run with the POC Amazon Redshift role.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-extract.jpg" rel="noopener" target="_blank"><img alt="Configuration Extract File" class="alignnone size-full wp-image-67865" height="687" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-extract.jpg" style="margin: 10px 0px 10px 0px;" width="1101" /></a></p> 
<ol start="3"> 
 <li>Run <code>make extract</code> to extract the workload. When completed, make note of the folder created at the path specified for the <code>workload_location</code> parameter in the extract (<code>s3://testdriveartifacts/myworkload/Extraction_xxxx-xx-xxTxx:xx:xx.xxxxxx+00:00</code>).</li> 
 <li>On the same baseline EC2 instance that will run the full workload on the source replica, run <code>vi config/replay.yaml</code> and configure the details with the workload location copied in the previous step 3 and the baseline warehouse endpoint. (See additional details on the parameters <a href="https://github.com/aws/redshift-test-drive/tree/main/core#configure-parameters-1" rel="noopener" target="_blank">Configure parameters to run an extract job</a>. The values after the <code>analysis_iam_role</code> parameter can be left as the default).</li> 
</ol> 
<p>The following code is an example for the beginning of a replay configuration for the source replica.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-replay.jpg" rel="noopener" target="_blank"><img alt="Config Reply File" class="alignnone size-full wp-image-67867" height="732" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-replay.jpg" style="margin: 10px 0px 10px 0px;" width="1132" /></a></p> 
<ol start="5"> 
 <li>On the EC2 instance that will run the target-producer workload, run <code>vi config/replay.yaml</code>. Configure the details with the workload location copied in the previous step 3, the producer warehouse endpoint and other configuration as in step 4. In order to replay only the producer workload, add the appropriate users to include or exclude for the filters parameter.</li> 
</ol> 
<p>The following code is an example of the filters used to exclude the BI workload from the producer.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-producer-1.jpg" rel="noopener" target="_blank"><img alt="Producer Configuration" class="alignnone size-full wp-image-67870" height="191" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-producer-1.jpg" style="margin: 10px 0px 10px 0px;" width="929" /></a></p> 
<ol start="6"> 
 <li>On the EC2 instance that will run the target-consumer workload, run <code>vi config/replay.yaml</code> and configure the details with the workload location copied in the previous step 3, the consumer warehouse endpoint, and appropriate filters as for step 5. The same users that were excluded on the producer workload replay should be included in the consumer workload replay.</li> 
</ol> 
<p>The following is an example of the filters used to only run the BI workload from the consumer.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-consumer.jpg" rel="noopener" target="_blank"><img alt="Consumer Configuration" class="alignnone size-full wp-image-67864" height="191" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-consumer.jpg" style="margin: 10px 0px 10px 0px;" width="930" /></a></p> 
<ol start="7"> 
 <li>Run <code>make replay</code> on the baseline instance, target-producer instance, and target-consumer instance simultaneously to run the workload on the target warehouses.</li> 
</ol> 
<h2>Analyze the Workload Replicator output</h2> 
<ol> 
 <li>Create the folder structure in the S3 bucket that was created in the previous step.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-code">'{comparison_stats_s3_path}/{what_if_timestamp}/{cluster_identifier}/'</code></pre> 
</div> 
<p>For <code>comparison_stats_s3_path</code>, enter the S3 bucket and path name. For <code>what_if_timestamp</code>, enter the replay start time. For <code>cluster_identifier</code>, enter the target cluster name for easy identification.</p> 
<p>The following screenshot shows</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-s3.jpg" rel="noopener" target="_blank"><img alt="Configuration of S3" class="alignnone size-full wp-image-67868" height="444" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-config-s3.jpg" style="margin: 10px 0px 10px 0px;" width="1289" /></a></p> 
<ol start="2"> 
 <li>Use the following script to unload system table data for each target cluster to a corresponding Amazon S3 target path that was created previously in the baseline Redshift cluster using QEv2.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">UNLOAD ($$
SELECT a.*,Trim(u.usename) as username FROM sys_query_history a , pg_user u
WHERE a.user_id = u.usesysid
and a.start_time &gt; to_timestamp('{what_if_timestamp}','YYYY-MM- DD-HH24-MI-SS')
$$) TO '{comparison_stats_s3_path}/{what_if_timestamp}/{cluster_identifier}/'
FORMAT AS PARQUET PARALLEL OFF ALLOWOVERWRITE 
IAM_ROLE '{redshift_iam_role}';</code></pre> 
</div> 
<p>For <code>what_if_timestamp</code>, enter the replay start time. For <code>comparison_stats_s3_path</code>, enter the S3 bucket and path name. For <code>cluster_identifier</code>, enter the target cluster name for easy identification. For <code>redshift_iam_role</code>, enter the Amazon Resource Name (ARN) of the Redshift IAM role for the target cluster.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-unload-s3.jpg" rel="noopener" target="_blank"><img alt="Unload File" class="alignnone size-full wp-image-67869" height="388" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-unload-s3.jpg" style="margin: 10px 0px 10px 0px;" width="1289" /></a></p> 
<ol start="3"> 
 <li>Create an external schema in Amazon Redshift with the name <code>comparison_stats</code>.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE EXTERNAL SCHEMA comparison_stats from DATA CATALOG
DATABASE 'redshift_config_comparison'
IAM_ROLE '{redshift-iam-role}'
CREATE EXTERNAL DATABASE IF NOT EXISTS;</code></pre> 
</div> 
<ol start="4"> 
 <li>Create an external table in Amazon Redshift with the name <code>redshift_config_comparision_aggregate</code> based on the Amazon S3 file location.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE EXTERNAL TABLE comparison_stats.redshift_config_comparision_aggregate 
(user_id int,
query_id bigint,
query_label VARCHAR,
transaction_id bigint,
session_id int,
database_name VARCHAR,
query_type VARCHAR,
status VARCHAR,
result_cache_hit boolean,
start_time timestamp,
end_time timestamp,
elapsed_time bigint,
queue_time bigint,
execution_time bigint,
error_message VARCHAR,
returned_rows bigint,
returned_bytes bigint,
query_text VARCHAR,
redshift_version VARCHAR,
usage_limit VARCHAR,
compute_type VARCHAR,
compile_time bigint,
planning_time bigint,
lock_wait_time bigint,
username VARCHAR)
PARTITIONED BY (cluster_identifier VARCHAR)
STORED AS PARQUET
LOCATION '{comparison_stats_s3_path}/{what_if_timestamp}';</code></pre> 
</div> 
<ol start="5"> 
 <li>After creating a partitioned table, alter the table using the following statement to register partitions to the external catalog.</li> 
</ol> 
<p>When you add a partition, you define the location of the subfolder on Amazon S3 that contains the partition data. Run that statement for each cluster identifier.</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">ALTER&nbsp;TABLE&nbsp;comparison_stats.redshift_config_comparision_aggregate 
ADD PARTITION (cluster_identifier='{cluster_identifier}')
LOCATION '{comparison_stats_s3_path}/{what_if_timestamp}/{cluster_identifier}/';</code></pre> 
</div> 
<p>Example:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">ALTER&nbsp;TABLE&nbsp;comparison_stats.redshift_config_comparision_aggregate 
ADD PARTITION (cluster_identifier='baseline-ra3-4xlarge-2')
LOCATION 's3://workload-replicator-redshift/2024-03-05-21-00-00/baseline-ra3-4xlarge-2/';</code></pre> 
</div> 
<div class="hide-language"> 
 <pre><code class="lang-sql">ALTER&nbsp;TABLE&nbsp;comparison_stats.redshift_config_comparision_aggregate 
ADD PARTITION (cluster_identifier='producer-serverless32RPU')
LOCATION 's3://workload-replicator-redshift/2024-03-05-21-00-00/producer-serverless32RPU/';</code></pre> 
</div> 
<div class="hide-language"> 
 <pre><code class="lang-sql">ALTER&nbsp;TABLE&nbsp;comparison_stats.redshift_config_comparision_aggregate 
ADD PARTITION (cluster_identifier='consumer-serverless64RPU')
LOCATION 's3://workload-replicator-redshift/2024-03-05-21-00-00/consumer-serverless64RPU/';</code></pre> 
</div> 
<h2>Deploy the QEv2 SQL Notebook and analyze workload</h2> 
<p>In this section, we analyze the queries that were replayed in both the baseline and target clusters. We analyze the workload based on the common queries that are executed in the baseline and target clusters.</p> 
<ol> 
 <li>Download the <a href="https://aws-blogs-artifacts-public.s3.amazonaws.com/artifacts/BDB-3985/Analysis+Notebook.ipynb" rel="noopener" target="_blank">analysis notebook</a> from Amazon S3.</li> 
 <li>Import the notebook into the baseline Redshift clusters using QEv2. For guidance, refer to the <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-notebooks.html" rel="noopener" target="_blank">Authoring and running notebooks</a>.</li> 
 <li>Create the stored procedure <code>common_queries_sp</code> in the same database that was used to create the external schema.</li> 
 <li>The stored procedure will create a view called <code>common_queries</code> by querying the external table <code>redshift_config_comparison_aggregate</code> that was created in previous steps.</li> 
</ol> 
<p>The view will identify the queries common to both the baseline and target clusters as mentioned in the notebook.</p> 
<ol start="5"> 
 <li>Execute the stored procedure by passing the cluster identifiers for the baseline and target clusters as parameters to the stored procedure.</li> 
</ol> 
<p>For this post, we passed the baseline and producer cluster identifier as the parameters. Passing the cluster identifiers as parameters will retrieve the data only for those specific clusters.</p> 
<p>Once the <code>common_queries</code> view is created, you can perform further analysis using subsequent queries that are available in the notebook. If you have more than one target cluster, you can follow the same analysis process for each one. For this post, we have two target clusters: producer and consumer. We first performed the analysis between the baseline and producer clusters, then repeated the same process to analyze the data for the baseline versus consumer clusters.</p> 
<p>To analyze our workload, we will use the <code>sys_query_history</code> view. We frequently use several columns from this view, including the following:</p> 
<ul> 
 <li><code>elapsed_time</code>: The end-to-end time of the query run</li> 
 <li><code>execution_time</code>: The time the query spent running. In the case of a SELECT query, this also includes the return time.</li> 
 <li><code>compile_time</code>: The time the query spent compiling</li> 
</ul> 
<p>For more information on <code>sys_query_history</code>, refer to <a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_QUERY_HISTORY.html" rel="noopener" target="_blank">SYS_QUERY_HISTORY</a> in the Amazon Redshift Database Developer Guide. The following table shows the descriptions of the analysis queries.</p> 
<table border="1px" cellpadding="10px" style="height: 808px;" width="696"> 
 <tbody> 
  <tr style="background-color: #000000;"> 
   <td width="31"></td> 
   <td width="195"><span style="color: #ffffff;"><strong>Name of the query</strong></span></td> 
   <td width="397"><span style="color: #ffffff;"><strong>Description</strong></span></td> 
  </tr> 
  <tr> 
   <td width="31">1</td> 
   <td width="195">Overall workload by user</td> 
   <td width="397">Count of common queries between baseline and target clusters based on user</td> 
  </tr> 
  <tr> 
   <td width="31">2</td> 
   <td width="195">Overall workload by query type</td> 
   <td width="397">Count of common queries between baseline and target clusters based on query type</td> 
  </tr> 
  <tr> 
   <td width="31">3</td> 
   <td width="195">Overall workload comparison (in seconds)</td> 
   <td width="397">Compare the overall workload between the baseline and target clusters by analyzing the execution time, compile time, and elapsed time</td> 
  </tr> 
  <tr> 
   <td width="31">4</td> 
   <td width="195">Percentile workload comparison</td> 
   <td width="397">The percentage of queries that perform at or below that runtime (for example, p50_s having the value of 5 seconds means 50% of queries in that workload were 5 seconds or faster)</td> 
  </tr> 
  <tr> 
   <td width="31">5</td> 
   <td width="195">Number of improve/degrade/stay same queries</td> 
   <td width="397">The number of queries degraded/stayed the same/improved when comparing the elapsed time between the baseline and target clusters</td> 
  </tr> 
  <tr> 
   <td width="31">6</td> 
   <td width="195">Degree of query-level performance change (proportion)</td> 
   <td width="397">The degree of change of the query from the baseline to target relative to the baseline performance</td> 
  </tr> 
  <tr> 
   <td width="31">7</td> 
   <td width="195">Comparison by query type (in seconds)</td> 
   <td width="397">Compare the elapsed time of different query types such as <code>SELECT</code>, <code>INSERT</code>, and <code>COPY</code> commands between the baseline cluster and target cluster</td> 
  </tr> 
  <tr> 
   <td width="31">8</td> 
   <td width="195">Top 10 slowest running queries (in seconds)</td> 
   <td width="397">Top 10 slowest queries between the baseline and target cluster by comparing the elapsed time of both clusters</td> 
  </tr> 
  <tr> 
   <td width="31">9</td> 
   <td width="195">Top 10 improved queries (in seconds)</td> 
   <td width="397">The top 10 queries with the most improved elapsed time when comparing the baseline cluster to the target cluster</td> 
  </tr> 
 </tbody> 
</table> 
<h2>Sample Results analysis</h2> 
<p>In our example, the overall workload improvement for workload isolation architecture using data sharing for ETL workload between baseline and producer is 858 seconds (<code>baseline_elapsed_time</code> – <code>target_elapsed_time</code>) for the sample TPC data, as shown in the following screenshots.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-baseline-producer.jpg" rel="noopener" target="_blank"><img alt="Baseline vs Producer Analysis" class="alignnone size-full wp-image-67863" height="527" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-baseline-producer.jpg" style="margin: 10px 0px 10px 0px;" width="1286" /></a></p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-baseline-producer-graph.jpg" rel="noopener" target="_blank"><img alt="Baseline vs Producer Analysis Grpah" class="alignnone size-full wp-image-67862" height="542" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-baseline-producer-graph.jpg" style="margin: 10px 0px 10px 0px;" width="1082" /></a></p> 
<p>The overall workload improvement for workload isolation architecture using data sharing for BI workload between baseline and consumer is 1148 seconds (<code>baseline_elapsed_time</code> – <code>target_elapsed_time</code>) for sample TPC data, as shown in the following screenshots.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-baseline-consumer.jpg" rel="noopener" target="_blank"><img alt="Baseline vs Consumer Analysis" class="alignnone size-full wp-image-67861" height="562" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-baseline-consumer.jpg" style="margin: 10px 0px 10px 0px;" width="1287" /></a></p> 
<h2><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-baseline-consumer-grpah.jpg" rel="noopener" target="_blank"><img alt="Baseline vs Consumer Analysis Graph" class="alignnone size-full wp-image-67860" height="528" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/BDB-3985-baseline-consumer-grpah.jpg" style="margin: 10px 0px 10px 0px;" width="1070" /></a></h2> 
<h2>Cleanup</h2> 
<p>Complete the following steps to clean up your resources:</p> 
<ol> 
 <li>Delete the Redshift provisioned replica cluster and the two Redshift serverless endpoints (32 RPU and 64 RPU)</li> 
 <li>Delete the S3 bucket used to store the artifacts</li> 
 <li>Delete the baseline, target-producer, and target-consumer EC2 instances</li> 
 <li>Delete the IAM role created for the EC2 instances to access Redshift clusters and S3 buckets</li> 
 <li>Delete the IAM roles created for Amazon Redshift warehouses to access S3 buckets for <code>COPY</code> and <code>UNLOAD</code> commands</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this post, we walked you through the process of testing workload isolation architecture using Amazon Redshift Data Sharing and Test Drive utility. We demonstrated how you can use SQL for advanced price performance analysis and compare different workloads on different target Redshift cluster configurations. We encourage you to evaluate your Amazon Redshift data sharing architecture using the Redshift Test Drive tool. Use the provided SQL script to analyze the price-performance of your Amazon Redshift cluster.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="text-align: left;"><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/Ayan-Majumder.jpg" rel="noopener" target="_blank"><img alt="" class="size-full wp-image-67889 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/08/21/Ayan-Majumder.jpg" width="100" /></a><strong>Ayan Majumder</strong> is an Analytics Specialist Solutions Architect at AWS. His expertise lies in designing robust, scalable, and efficient cloud solutions for customers. Beyond his professional life, he derives joy from traveling, photography, and outdoor activities.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/02/ekta-ahuja-1.png" rel="noopener" target="_blank"><img alt="" class="size-full wp-image-36901 alignleft" height="132" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/02/ekta-ahuja-1.png" width="100" /></a><strong>Ekta Ahuja </strong></strong>is an Amazon Redshift Specialist Solutions Architect at AWS. She is passionate about helping customers build scalable and robust data and analytics solutions. Before AWS, she worked in several different data engineering and analytics roles. Outside of work, she enjoys landscape photography, traveling, and board games.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/05/31/BDB-2083-Julia.jpg" rel="noopener" target="_blank"><img alt="Julia Beck" class="size-full wp-image-29577 alignleft" height="132" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/05/31/BDB-2083-Julia.jpg" width="100" /></a><strong>Julia Beck </strong></strong>is an Analytics Specialist Solutions Architect at AWS. She is passionate about supporting customers&nbsp;in validating and optimizing analytics solutions by architecting proof of concept workloads designed to meet their specific needs.</p>
