---
title: "Simplify data ingestion from Amazon S3 to Amazon Redshift using auto-copy"
url: "https://aws.amazon.com/blogs/big-data/simplify-data-ingestion-from-amazon-s3-to-amazon-redshift-using-auto-copy/"
date: "Wed, 30 Oct 2024 15:16:32 +0000"
author: "Tahir Aziz"
feed_url: "https://aws.amazon.com/blogs/big-data/category/database/amazon-redshift/feed/"
---
<p><a href="https://aws.amazon.com/redshift/">Amazon Redshift</a> is a fast, scalable, secure, and fully managed cloud data warehouse that makes it simple and cost-effective to analyze your data using standard SQL and your existing business intelligence (BI) tools. Tens of thousands of customers today rely on Amazon Redshift to analyze exabytes of data and run complex analytical queries, making it the most widely used cloud data warehouse.</p> 
<p>Data ingestion is the process of getting data to <a href="https://aws.amazon.com/redshift/">Amazon Redshift</a>. You can leverage one of the many zero-ETL integration methods to make data available in <a href="https://aws.amazon.com/redshift/">Amazon Redshift</a> directly. However, if your data is in your Amazon S3 bucket, then you can simply load data from <a href="http://aws.amazon.com/s3">Amazon Simple Storage Service</a> (Amazon S3) to Amazon Redshift using the COPY command. A <a href="https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html">COPY</a> command is the most efficient way to load a table from S3 because it uses the Amazon Redshift’s massively parallel processing (MPP) architecture to read and load data in parallel.</p> 
<p>Amazon Redshift launched auto-copy support to simplify data loading from Amazon S3 into Amazon Redshift. You can now setup continuous file ingestion rules to track your Amazon S3 paths and automatically load new files without the need for additional tools or custom solutions. This also enables end users to have the latest data available in Amazon Redshift shortly after the source data is available.</p> 
<p>This post shows you how to build automatic file ingestion pipelines in Amazon Redshift when source files are located on Amazon S3 by using a simple SQL command. In addition, we show you how to enable auto-copy using auto-copy jobs, how to monitor jobs, considerations, and best practices.</p> 
<h2>Overview of the auto-copy feature in Amazon Redshift</h2> 
<p>The auto-copy feature in Amazon Redshift leverages the S3 event integration to automatically load data into Amazon Redshift and simplifies automatic data loading from Amazon S3 with a simple SQL command. You can enable Amazon Redshift auto-copy by creating auto-copy jobs. A auto-copy job is a database object that stores, automates, and reuses the COPY statement for newly created files that land in the S3 folder.</p> 
<p>The following diagram illustrates this process.</p> 
<p><img alt="" class="wp-image-70913 size-full aligncenter" height="247" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-img1.png" title="automatic data ingestion from Amazon S3 using auto-copy jobx" width="511" /></p> 
<p>S3 event integration and auto-copy jobs have the following benefits:</p> 
<ul> 
 <li>Users can now load data from Amazon S3 automatically without having to build a pipeline or using an external framework</li> 
 <li>auto-copy jobs offer automatic and incremental data ingestion from an Amazon S3 location without the need to implement a custom solution</li> 
 <li>This functionality comes at no additional cost</li> 
 <li>Existing COPY statements can be converted into auto-copy jobs by appending the <code>JOB CREATE <span style="color: #ff0000;">&lt;job_name&gt;</span></code> parameter</li> 
 <li>It keeps track of loaded files and minimizes data duplication.</li> 
 <li>It can be quickly set up using a simple SQL statement using your choice of JDBC/ODBC clients.</li> 
 <li>It has automatic error handling of bad quality data files.</li> 
 <li>It has a mechanism to load-once for each file. This means that there is no need to generate explicit manifest files.</li> 
</ul> 
<h2>Prerequisites</h2> 
<p>To get started with auto-copy, you need the following prerequisites:</p> 
<ul> 
 <li>An <a href="https://console.aws.amazon.com/?nc2=h_m_mc" rel="noopener" target="_blank">AWS account</a></li> 
 <li>An encrypted Amazon Redshift provisioned cluster or Amazon redshift serverless workgroup</li> 
 <li>An <a href="https://console.aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon S3</a> bucket</li> 
 <li>Add following to the <a href="https://console.aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon S3</a> bucket policy 
  <div class="hide-language"> 
   <pre><code class="lang-json">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Auto-Copy-Policy-01",
            "Effect": "Allow",
            "Principal": {
                "Service":"redshift.amazonaws.com"
                    
                
            },
            "Action": [
                "s3:GetBucketNotification",
                "s3:PutBucketNotification",
                "s3:GetBucketLocation"
            ],
            "Resource": "arn:aws:s3:::&lt;&lt;your-s3-bucket-name&gt;&gt;",
            "Condition": {
                "StringLike": {
                    "aws:SourceArn": "arn:aws:redshift:&lt;region-name&gt;:&lt;aws-account-id&gt;:integration:*",
                    "aws:SourceAccount": "&lt;aws-account-id&gt;"
                }
            }
        }
    ]
}</code></pre> 
  </div> </li> 
</ul> 
<h2>Set up Amazon S3 event Integration</h2> 
<p>An <a href="https://console.aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon S3</a> event integration facilitates seamless and automated data ingestion from S3 buckets into an <a href="https://console.aws.amazon.com/redshift/" rel="noopener" target="_blank">Amazon Redshift</a> data warehouse, streamlining the process of transferring and storing data for analytical purposes</p> 
<ol> 
 <li>Sign in to the <a href="https://aws.amazon.com/free/?gclid=EAIaIQobChMI4YSO3OysiAMVYy7UAR24FzKcEAAYASAAEgKp1_D_BwE&amp;trk=7541ebd3-552d-4f98-9357-b542436aa66c&amp;sc_channel=ps&amp;ef_id=EAIaIQobChMI4YSO3OysiAMVYy7UAR24FzKcEAAYASAAEgKp1_D_BwE:G:s&amp;s_kwcid=AL!4422!3!651751058796!e!!g!!aws%20management%20console!19852662149!145019243977&amp;all-free-tier.sort-by=item.additionalFields.SortRank&amp;all-free-tier.sort-order=asc&amp;awsf.Free%20Tier%20Types=*all&amp;awsf.Free%20Tier%20Categories=*all" rel="noopener" target="_blank">AWS Management Console</a> and Navigate to <a href="https://console.aws.amazon.com/redshift/" rel="noopener" target="_blank">Amazon Redshift</a> home page. Under <strong>Integrations</strong> section choose <strong>S3 event integrations<br /> <img alt="" class="alignnone size-full wp-image-70914" height="590" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image003.png" width="1430" /><br /> </strong></li> 
 <li>Choose <strong>Create S3 event integration<br /> <img alt="" class="alignnone size-full wp-image-70915" height="348" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image005.png" width="1432" /><br /> </strong></li> 
 <li>Enter <strong>Integration name</strong> and <strong>Description, </strong>choose <strong>Next<br /> <img alt="" class="alignnone size-full wp-image-70916" height="514" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image007.png" width="1430" /><br /> </strong></li> 
 <li>Choose <strong>Browse S3 buckets</strong>, a dialog box pops up, select the <a href="https://console.aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon S3</a> bucket and choose <strong>Continue<br /> <img alt="" class="alignnone size-full wp-image-70917" height="291" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image009.png" width="1429" /><br /> </strong></li> 
 <li>Amazon S3 bucket is selected. Choose<strong> Next<br /> <img alt="" class="alignnone size-full wp-image-70918" height="480" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image011.png" width="1600" /></strong></li> 
 <li>Choose <strong>Browse Redshift data warehouse<br /> <img alt="" class="alignnone size-full wp-image-70919" height="573" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image013.png" width="1476" /><br /> </strong></li> 
 <li>Choose the Amazon Redshift data warehouse and choose <strong>Continue<br /> <img alt="" class="alignnone size-full wp-image-70920" height="367" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image015.png" width="1796" /><br /> </strong></li> 
 <li>Then Amazon Redshift resource policy needs access to S3 event integration. In case of Resource policy error, check <strong>Fix it for me </strong>and choose <strong>Next<br /> <img alt="" class="alignnone size-full wp-image-70921" height="732" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image017.png" width="1598" /><br /> </strong></li> 
 <li>Add <strong>Tags</strong> as required and choose <strong>Next<br /> <img alt="" class="alignnone size-full wp-image-70922" height="471" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image019.png" width="1601" /><br /> </strong></li> 
 <li>Review changes and choose <strong><strong>Create S3 event integration<br /> <img alt="" class="alignnone size-full wp-image-70923" height="568" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image021.png" width="1585" /></strong></strong><img alt="" class="alignnone size-full wp-image-70924" height="584" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image023.png" width="1398" /></li> 
 <li>An S3 event integration is created. Wait until the status of S3 event integration is <strong>Active<br /> <img alt="" class="alignnone size-full wp-image-70925" height="602" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image025.png" width="1432" /><br /> </strong></li> 
</ol> 
<h2>Set up auto-copy jobs</h2> 
<p>In this section, we demonstrate how to automate data loading of files from Amazon S3 into Amazon Redshift. With the existing <a href="https://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-data-source-s3.html#copy-parameters-data-source-s3-syntax" rel="noopener" target="_blank">COPY syntax</a>, we add the <code>JOB CREATE</code> parameter to perform a one-time setup for automatic file ingestion. See the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">COPY <em><span style="color: #ff0000;">&lt;table-name&gt;</span></em>
FROM 's3://<em><span style="color: #ff0000;">&lt;s3-object-path&gt;</span></em>'
[COPY PARAMETERS...]
JOB CREATE <em><span style="color: #ff0000;">&lt;job-name&gt;</span></em> [AUTO ON | OFF];</code></pre> 
</div> 
<p>Auto ingestion is enabled by default on auto-copy jobs. Files already present at the S3 location will not be visible to the auto-copy job. Only files added after JOB creation are tracked by <a href="https://aws.amazon.com/redshift/">Amazon Redshift</a>.</p> 
<h3>Automate ingestion from a single data source</h3> 
<p>With a auto-copy job, you can automate ingestion from a single data source by creating one job and specifying the path to the S3 objects that contain the data. The S3 object path can reference a set of folders that have the same key prefix.</p> 
<p>In this example, we have multiple files that are being loaded on a daily basis containing the sales transactions across all the stores in the US. For this we can create a <code>store_sales</code> folder in the bucket.</p> 
<p><img alt="" class="alignnone wp-image-70926 size-full" height="460" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image027.png" title="store_sales" width="1656" /></p> 
<p>The following code creates the <code>store_sales</code> table:</p> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-sql">DROP TABLE IF EXISTS public.store_sales;
CREATE TABLE IF NOT EXISTS public.store_sales
(
  ss_sold_date_sk int4,            
  ss_sold_time_sk int4,     
  ss_item_sk int4 not null,      
  ss_customer_sk int4,           
  ss_cdemo_sk int4,              
  ss_hdemo_sk int4,         
  ss_addr_sk int4,               
  ss_store_sk int4,           
  ss_promo_sk int4,           
  ss_ticket_number int8 not null,        
  ss_quantity int4,           
  ss_wholesale_cost numeric(7,2),          
  ss_list_price numeric(7,2),              
  ss_sales_price numeric(7,2),
  ss_ext_discount_amt numeric(7,2),             
  ss_ext_sales_price numeric(7,2),              
  ss_ext_wholesale_cost numeric(7,2),           
  ss_ext_list_price numeric(7,2),               
  ss_ext_tax numeric(7,2),                 
  ss_coupon_amt numeric(7,2), 
  ss_net_paid numeric(7,2),   
  ss_net_paid_inc_tax numeric(7,2),             
  ss_net_profit numeric(7,2),
  primary key (ss_item_sk, ss_ticket_number)
 ) 
DISTKEY (ss_item_sk) 
SORTKEY (ss_sold_date_sk);</code></pre> 
 </div> 
</div> 
<p>Next, we create the auto-copy job to automatically load the gzip-compressed files into the <code>store_sales</code> table:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">COPY store_sales
FROM 's3://aws-redshift-s3-auto-copy-demo/store_sales'
IAM_ROLE 'arn:aws:iam::**********:role/Redshift-S3'
gzip delimiter '|' EMPTYASNULL
region 'us-east-1'
JOB CREATE job_store_sales AUTO ON;</code></pre> 
</div> 
<p>Each day’s sales transactions are loaded to their own folder in Amazon S3.</p> 
<p><img alt="" class="alignnone size-full wp-image-70927" height="460" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image029.png" width="1656" /></p> 
<p>Now upload the files for transaction sold on <code>2002-12-31</code>. Each folder contains multiple gzip-compressed files.</p> 
<p><img alt="" class="alignnone size-full wp-image-70928" height="588" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image031.png" width="1646" /></p> 
<p>Since the auto-copy job is already created, it automatically loads the gzip-compressed files located in the S3 object path specified in the COPY command to the <code>store_sales</code> table.</p> 
<p>Let’s run a query to get the daily total of sales transactions across all the stores in the US:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT ss_sold_date_sk, count(1)
  FROM store_sales
GROUP BY ss_sold_date_sk;</code></pre> 
</div> 
<p>The output shown comes from the transactions sold on <code>2002-12-31</code>.</p> 
<p><img alt="" class="alignnone wp-image-70929 size-full" height="442" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image033.png" title="transactions shown" width="572" /></p> 
<p>The following day, incremental sales transactions data are loaded to a new folder in the same S3 object path.</p> 
<p><img alt="" class="alignnone wp-image-70930 size-full" height="496" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image035.png" title="incremental sales transactions" width="1653" /></p> 
<p>As new files arrive to the same S3 object path, the auto-copy job automatically loads the unprocessed files to the <code>store_sales</code> table in an incremental fashion.</p> 
<p>All new sales transactions for <code>2003-01-01</code> are automatically ingested, which can be verified by running the following query:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT ss_sold_date_sk, count(1)
  FROM store_sales
GROUP BY ss_sold_date_sk;</code></pre> 
</div> 
<h3><img alt="" class="alignnone wp-image-70931 size-full" height="276" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image037.jpg" title="query 1" width="451" /></h3> 
<h3>Automate ingestion from multiple data sources</h3> 
<p>We can also load an Amazon Redshift table from multiple data sources. When using a pub/sub pattern where multiple S3 buckets populate data to an Amazon Redshift table, you have to maintain multiple data pipelines for each source/target combination. With new parameters in the COPY command, this can be automated to handle data loads efficiently.</p> 
<p>In the following example, the <code>Customer_1</code> folder has Green Cab Company sales data, and the <code>Customer_2</code> folder has Red Cab Company sales data. We can use the COPY command with the JOB parameter to automate this ingestion process.</p> 
<p><img alt="" class="alignnone wp-image-70932 size-full" height="380" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image039.png" title="automate this ingestion process" width="1646" /></p> 
<p>The following screenshot shows sample data stored in files. Each folder has similar data but for different customers.</p> 
<p><img alt="" class="alignnone wp-image-70933 size-full" height="279" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image041.jpg" title="sample data sored in files" width="1419" /></p> 
<p>The target for these files in this example is the Amazon Redshift table <code>cab_sales_data</code>.</p> 
<p>Define the target table <code>cab_sales_data</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">DROP TABLE IF EXISTS cab_sales_data;
CREATE TABLE IF NOT EXISTS cab_sales_data
(
  vendorid                VARCHAR(4),
  pickup_datetime         TIMESTAMP,
  dropoff_datetime        TIMESTAMP,
  store_and_fwd_flag      VARCHAR(1),
  ratecode                INT,
  pickup_longitude        FLOAT4,
  pickup_latitude         FLOAT4,
  dropoff_longitude       FLOAT4,
  dropoff_latitude        FLOAT4,
  passenger_count         INT,
  trip_distance           FLOAT4,
  fare_amount             FLOAT4,
  extra                   FLOAT4,
  mta_tax                 FLOAT4,
  tip_amount              FLOAT4,
  tolls_amount            FLOAT4,
  ehail_fee               FLOAT4,
  improvement_surcharge   FLOAT4,
  total_amount            FLOAT4,
  payment_type            VARCHAR(4),
  trip_type               VARCHAR(4)
)
DISTSTYLE EVEN
SORTKEY (passenger_count,pickup_datetime);</code></pre> 
</div> 
<p>You can define two auto-copy jobs as shown in the following code to handle and monitor the ingestion of sales data belonging to different customers, in our case <code>Customer_1</code> and <code>Customer_2</code>. These jobs monitor the <code>Customer_1</code> and <code>Customer_2</code> folders and load new files that are added here.</p> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-sql">COPY public.cab_sales_data
FROM 's3://aws-redshift-s3-auto-copy-demo/Customer_1'
IAM_ROLE 'arn:aws:iam::**********:role/Redshift-S3'
DATEFORMAT 'auto'
IGNOREHEADER 1
DELIMITER ','
IGNOREBLANKLINES
REGION 'us-east-1'
JOB CREATE job_green_cab AUTO ON;

COPY public.cab_sales_data
FROM 's3:// aws-redshift-s3-auto-copy-demo/Customer_2'
IAM_ROLE 'arn:aws:iam::**********:role/Redshift-S3'
DATEFORMAT 'auto'
IGNOREHEADER 1
DELIMITER ','
IGNOREBLANKLINES
REGION 'us-east-1'
JOB CREATE job_red_cab AUTO ON;</code></pre> 
 </div> 
</div> 
<p>After setting up the two jobs, we can upload the relevant files into their respective folders. This will make sure that the data is loaded efficiently as soon as the files arrive. Each customer is assigned its own <code>vendorid</code>, as shown in the following output:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT vendorid,
       sum(passenger_count) as total_passengers 
  FROM cab_sales_data
GROUP BY vendorid;</code></pre> 
</div> 
<h2><img alt="result 1" class="alignnone wp-image-38098 size-full" height="125" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/23/image017-4.jpg" title="result 1" width="441" /></h2> 
<h2>Manually run a auto-copy job</h2> 
<p>There might be scenarios wherein the auto-copy job needs to be paused, meaning it needs to stop looking for new files, for example, to fix a corrupted data pipeline at the data source.</p> 
<p>In that case, either use the COPY JOB ALTER command to set AUTO to OFF or create a new COPY JOB with AUTO OFF. Once this is set, auto copy will no longer ingest new files automatically.</p> 
<p>If necessary, users can manually invoke auto-copy job which will do the work and ingest if new files are found.</p> 
<p><code>copy job RUN</code>&nbsp;<span style="color: #ff0000;">&lt;auto-copy job Name&gt;</span></p> 
<p>You can disable “AUTO ON” in the existing auto-copy job using the following command:</p> 
<p><code>copy job ALTER</code> <span style="color: #ff0000;">&lt;auto-copy job Name&gt;</span><code> AUTO OFF</code></p> 
<p>The following table compares the syntax and data duplication between a regular copy statement and the new auto-copy job</p> 
<table border="1px"> 
 <tbody> 
  <tr style="background-color: #000000;"> 
   <td>.</td> 
   <td><span style="color: #ffffff;"><strong>Copy</strong></span></td> 
   <td><span style="color: #ffffff;"><strong>Auto-copy job</strong></span></td> 
  </tr> 
  <tr> 
   <td>Syntax</td> 
   <td><code>COPY <span style="color: #ff0000;">&lt;table-name&gt;</span><br /> FROM 's3://<span style="color: #ff0000;">&lt;s3-object-path&gt;</span>'<br /> [COPY PARAMETERS...]</code></td> 
   <td><code>COPY <span style="color: #ff0000;">&lt;table-name&gt;</span><br /> FROM 's3://<span style="color: #ff0000;">&lt;s3-object-path&gt;</span>'<br /> [COPY PARAMETERS...]<br /> JOB CREATE <span style="color: #ff0000;">&lt;job-name&gt;</span>;</code></td> 
  </tr> 
  <tr> 
   <td>Data Duplication</td> 
   <td>If it is run multiple times against the same S3 folder, it will load the data again, resulting in data duplication.</td> 
   <td>It will not load the same file twice, preventing data duplication.</td> 
  </tr> 
 </tbody> 
</table> 
<h2>Error handling and monitoring for auto-copy jobs</h2> 
<p>auto-copy jobs continuously monitor the S3 folder specified during job creation and perform ingestion whenever new files are created. New files created under the S3 folder are loaded exactly once to avoid data duplication.</p> 
<p>By default, if there are data or format issues with the specific files, the auto-copy job will fail to ingest the files with a load error and log details to the system tables. The auto-copy job will remain <code>AUTO ON</code> with new data files and will continue to ignore previously failed files.</p> 
<p>Amazon Redshift provides the following system tables for users to monitor or troubleshoot auto-copy jobs as needed:</p> 
<ul> 
 <li><strong>List auto-copy jobs</strong> – Use <a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_COPY_JOB.html">SYS_COPY_JOB</a> to list the auto-copy jobs stored in the database:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT * 
  FROM sys_copy_job;
</code></pre> 
</div> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/01/bdb-2778-image019.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-70935 size-full" height="125" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image045.png" title="list auto-copy jobs" width="1535" /></a></p> 
<ul> 
 <li><strong>Get a summary of a auto-copy job </strong>– Use the <a href="https://docs.aws.amazon.com/redshift/latest/dg/SYS_LOAD_HISTORY.html">SYS_LOAD_HISTORY</a> view to get the aggregate metrics of a auto-copy job operation by specifying the <code>copy_job_id</code>. It shows the aggregate metrics of the files that have been processed by a auto-copy job.</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT *
  FROM sys_load_history
 WHERE copy_job_id = 274978;</code></pre> 
</div> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/01/bdb-2778-image020.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-70936 size-full" height="116" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image047.png" title="get summary of auto-copy job" width="1484" /></a></p> 
<ul> 
 <li><strong>Get details of a auto-copy job </strong>– Use <a href="https://docs.aws.amazon.com/redshift/latest/dg/r_STL_LOAD_COMMITS.html" rel="noopener" target="_blank">STL_LOAD_COMMITS</a> to get the status and details of each file that was processed by a auto-copy job:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT *
  FROM stl_load_commits
 WHERE copy_job_id = 274978
ORDER BY curtime ASC;</code></pre> 
</div> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/01/bdb-2778-image021.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-70937 size-full" height="180" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image049.png" width="1367" /></a></p> 
<ul> 
 <li><strong>Get exception details of a auto-copy job </strong>– Use <a href="https://docs.aws.amazon.com/redshift/latest/dg/r_STL_LOAD_ERRORS.html" rel="noopener" target="_blank">STL_LOAD_ERRORS</a> to get the details of files that failed to ingest from a auto-copy job:</li> 
</ul> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-sql">SELECT   query,
    slice,
    starttime , 
    filename,
    line_number,
    colname,
    type,
    err_code,
    err_reason,
    copy_job_id,
    raw_line,
    raw_field_value
  FROM stl_load_errors
 WHERE copy_job_id = 274978;</code></pre> 
 </div> 
</div> 
<h2><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/01/bdb-2778-image022.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-70938 size-full" height="70" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/28/REVBLOG-706-image051.png" width="1408" /></a></h2> 
<h2>Auto-copy job best practices</h2> 
<p>In an auto-copy job, when a new file is detected and ingested (automatically or manually), Amazon Redshift stores the file name and doesn’t run this specific job when a new file is created with the same file name.</p> 
<p>The following are the recommended best practices when working with files using the auto-copy job:</p> 
<ul> 
 <li>Use unique file names for each file in a auto-copy job (for example, <code>2022-10-15-batch-1.csv</code>). However, you can use the same file name as long as it’s from different auto-copy jobs: 
  <ul> 
   <li><strong>job_customerA_sales</strong> – <code>s3://&lt;&lt;your-s3-bucket-name&gt;&gt;/sales/customerA/2022-10-15-sales.csv</code></li> 
   <li><strong>job_customerB_sales</strong> – <code>s3://&lt;&lt;your-s3-bucket-name&gt;&gt;/sales/customerB/2022-10-15-sales.csv</code></li> 
  </ul> </li> 
 <li>Do not update file contents. Do not overwrite existing files. Changes in existing files will not be reflected to the target table. The auto-copy job doesn’t pick up updated or overwritten files, so make sure they’re renamed as new file names for the auto-copy job to pick up.</li> 
 <li>Run regular COPY statements (not a job) if you need to ingest a file that was already processed by your auto-copy job. (COPY statement without a JOB CREATE syntax doesn’t track loaded files.) For example, this is helpful in scenarios where you don’t have control of the file name and the initial file received failed. The following figure shows a typical workflow in this case.</li> 
</ul> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/02/bdb-2778-yellow-diag.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-39185 size-full" height="211" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/02/bdb-2778-yellow-diag.png" title="file store_list" width="771" /></a></p> 
<ul> 
 <li>Delete and recreate your auto-copy job if you want to reset file tracking history and start over. You can drop auto-copy job using following command. 
  <div class="hide-language"> 
   <pre><code class="lang-sql">copy job DROP <span style="color: #ff0000;">&lt;auto-copy job Name&gt;</span></code></pre> 
  </div> </li> 
</ul> 
<h2>auto-copy job considerations</h2> 
<p>Here are the main things to consider when using auto-copy:</p> 
<ul> 
 <li>Existing files in Amazon S3 prefix are not loaded, use Copy command to catch up historical data</li> 
 <li>The following features are unsupported: 
  <ul> 
   <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-data-load.html#copy-maxerror" rel="noopener" target="_blank">MAXERROR parameter</a></li> 
   <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/loading-data-files-using-manifest.html" rel="noopener" target="_blank">manifest files</a></li> 
   <li><a href="https://docs.aws.amazon.com/redshift/latest/dg/copy-usage_notes-access-permissions.html#copy-usage_notes-access-key-based" rel="noopener" target="_blank">key-based access control</a></li> 
  </ul> </li> 
</ul> 
<p>For additional details on other considerations for auto-copy, refer to the <a href="https://docs.aws.amazon.com/redshift/latest/dg/loading-data-copy-job.html" rel="noopener" target="_blank">AWS documentation</a>.</p> 
<h2><strong>Customer feedback</strong></h2> 
<p><a href="https://www.geaerospace.com/" rel="noopener" target="_blank">GE Aerospace</a> is a global provider of jet engines, components, and systems for commercial and military aircraft. The company has been designing, developing, and manufacturing jet engines since World War I.</p> 
<blockquote>
 <p>“GE Aerospace uses AWS analytics and Amazon Redshift to enable critical business insights that drive important business decisions. With the support for auto-copy from Amazon S3, we can build simpler data pipelines to move data from Amazon S3 to Amazon Redshift. This accelerates our data product teams’ ability to access data and deliver insights to end users. We spend more time adding value through data and less time on integrations.”</p> 
 <p>– <strong>Alcuin Weidus, Sr. Principal Data Architect at GE Aerospace</strong></p>
</blockquote> 
<h2>Conclusion</h2> 
<p>This post demonstrated how to automate data ingestion from Amazon S3 to Amazon Redshift using the auto-copy feature. This new functionality helps make Amazon Redshift data ingestion easier than ever, and will allow SQL users to get access to the most recent data using a simple SQL command.</p> 
<p>Users can begin ingesting data to Redshift from Amazon S3 with simple SQL commands and gain access to the most up-to-date data without the need for third-party tools or custom implementation.</p> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-27451 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/04/01/Tahir-Aziz.png" width="100" />Tahir Aziz</strong> is an Analytics Solution Architect at AWS. He has worked with building data warehouses and big data solutions for over 15+ years. He loves to help customers design end-to-end analytics solutions on AWS. Outside of work, he enjoys traveling and cooking.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-61663 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/21/omakhurs-1.png" width="100" />Omama Khurshid</strong> is an Acceleration Lab Solutions Architect at Amazon Web Services. She focuses on helping customers across various industries build reliable, scalable, and efficient solutions. Outside of work, she enjoys spending time with her family, watching movies, listening to music, and learning new technologies.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-66452 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/07/24/raza_new_new_new.jpg" width="100" />Raza Hafeez</strong> is a Senior Product Manager at Amazon Redshift. He has over 13 years of professional experience building and optimizing enterprise data warehouses and is passionate about enabling customers to realize the power of their data. He specializes in migrating enterprise data warehouses to AWS Modern Data Architecture.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-38104 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/23/image029-1.jpg" width="100" /> Jason Pedreza</strong> is an Analytics Specialist Solutions Architect at AWS with data warehousing experience handling petabytes of data. Prior to AWS, he built data warehouse solutions at Amazon.com. He specializes in Amazon Redshift and helps customers build scalable analytic solutions.</p> 
<p style="clear: both;"><strong> <img alt="" class="size-full wp-image-38105 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/23/image031.jpg" width="100" /> Nita Shah</strong> is an Analytics Specialist Solutions Architect at AWS based out of New York. She has been building data warehouse solutions for over 20 years and specializes in Amazon Redshift. She is focused on helping customers design and build enterprise-scale well-architected analytics and decision support platforms.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-38106 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/23/image033.jpg" width="100" /> Eren Baydemir</strong>, a Technical Product Manager at AWS, has 15 years of experience in building customer-facing products and is currently focusing on data lake and file ingestion topics in the Amazon Redshift team. He was the CEO and co-founder of DataRow, which was acquired by Amazon in 2020.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-38107 alignleft" height="107" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/23/image035.jpg" width="100" /> Eesha Kumar</strong> is an Analytics Solutions Architect with AWS. He works with customers to realize the business value of data by helping them build solutions using the AWS platform and tools.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-38108 alignleft" height="139" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/23/image037.jpg" width="100" /> Satish Sathiya</strong> is a Senior Product Engineer at Amazon Redshift. He is an avid big data enthusiast who collaborates with customers around the globe to achieve success and meet their data warehousing and data lake architecture needs.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/23/image040.jpg" rel="noopener" target="_blank"><img alt="" class="wp-image-38143 size-full alignleft" height="101" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/23/image040.jpg" width="100" /></a>&nbsp;Hangjian Yuan</strong> is a Software Development Engineer at Amazon Redshift. He’s passionate about analytical databases and focuses on delivering cutting-edge streaming experiences for customers.</p>
