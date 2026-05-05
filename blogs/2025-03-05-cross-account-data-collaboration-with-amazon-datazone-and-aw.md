---
title: "Cross-account data collaboration with Amazon DataZone and AWS analytical tools"
url: "https://aws.amazon.com/blogs/big-data/cross-account-data-collaboration-with-amazon-datazone-and-aws-analytical-tools/"
date: "Wed, 05 Mar 2025 17:29:17 +0000"
author: "Arun Pradeep Selvaraj"
feed_url: "https://aws.amazon.com/blogs/big-data/category/database/amazon-redshift/feed/"
---
<p>Data sharing has become a crucial aspect of driving innovation, contributing to growth, and fostering collaboration across industries. According to this&nbsp;<a href="https://www.gartner.com/smarterwithgartner/data-sharing-is-a-business-necessity-to-accelerate-digital-business" rel="noopener" target="_blank">Gartner study</a>, organizations promoting data sharing outperform their peers on most business value metrics. A straightforward data access and sharing mechanism is crucial for enabling effective data sharing across an organization. There are challenges such as complexity in managing cross-account permissions and difficulty in discovering the right data across accounts that organizations face when trying to share data products across AWS accounts.&nbsp;<a href="https://aws.amazon.com/datazone/" rel="noopener" target="_blank">Amazon DataZone</a>&nbsp;is a fully managed data management service that customers can use to catalog, discover, share, and govern data stored across <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a>.</p> 
<p>In this post, we will cover how you can use Amazon DataZone to facilitate data collaboration between AWS accounts.</p> 
<h2>Solution overview</h2> 
<p>This solution provides a streamlined way to enable cross-account data collaboration using&nbsp;<a href="https://docs.aws.amazon.com/datazone/latest/userguide/working-with-associated-accounts.html" rel="noopener" target="_blank">Amazon DataZone domain association</a> while maintaining security and governance. This post describes the process of using the business data catalog resource of Amazon DataZone to publish data assets so they’re discoverable by other accounts. After they’ve been published, you can query the published assets from another AWS account using analytical tools such as&nbsp;<a href="https://docs.aws.amazon.com/athena/latest/ug/what-is.html" rel="noopener" target="_blank">Amazon Athena</a> and the <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2.html" rel="noopener" target="_blank">Amazon Redshift query editor</a>,&nbsp;as shown in the following figure.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img1.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-75745 size-full" height="699" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img1.png" width="1020" /></a></p> 
<p>In this solution (as shown in the preceding figure), the AWS account that contains the data assets is referred to as the <em>producer account</em>. The AWS account that needs to access or use the data from the producer account is referred to as the <em>consumer account</em>. The Amazon DataZone domain is created and managed within the producer account and then the consumer account is associated with that domain.</p> 
<p>As part of Amazon DataZone domain association,&nbsp;Amazon DataZone uses&nbsp;<a href="http://aws.amazon.com/ram" rel="noopener" target="_blank">AWS Resource Access Manager (AWS RAM)</a> to share the resource. When the producer and consumer AWS accounts are in the same organization within&nbsp;<a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html" rel="noopener" target="_blank">AWS Organizations</a>, the domain association happens automatically.&nbsp;If the producer and consumer AWS accounts are in different organizations, AWS RAM sends an invitation to the consumer AWS account to accept or reject the resource grant.</p> 
<p>This solution presents three Amazon DataZone user personas as:</p> 
<ul> 
 <li><strong>Data administrators</strong>: Account owners in both producer and consumer AWS accounts. The data administrators are responsible for creating Amazon DataZone domains, configuring domain associations, and accepting domain associations within the Amazon DataZone domain.</li> 
 <li><strong>Data publishers</strong>: Users in producer AWS accounts. The data publishers are responsible for creating Amazon DataZone publish projects and environments, producing and publishing data assets, and accepting subscription requests.</li> 
 <li><strong>Data subscribers</strong>: Users in consumer AWS accounts. The data subscribers are responsible for creating Amazon DataZone subscribe projects and environments, searching for and subscribing to data assets, and querying the data and deriving insights.</li> 
</ul> 
<h2>Prerequisites</h2> 
<p>To follow along with the instructions, you will need:</p> 
<ul> 
 <li>Two AWS accounts, one serving as producer and other account serving as consumer. Create new&nbsp;<a href="https://portal.aws.amazon.com/billing/signup" rel="noopener" target="_blank">AWS accounts</a> if necessary.</li> 
 <li>An&nbsp;<a href="https://docs.aws.amazon.com/redshift/latest/gsg/new-user.html" rel="noopener" target="_blank">Amazon Redshift provisioned cluster</a>&nbsp;or&nbsp;<a href="https://docs.aws.amazon.com/redshift/latest/gsg/new-user-serverless.html#serverless-console-resource-creation" rel="noopener" target="_blank">Amazon Redshift Serverless workgroup</a>&nbsp;in the producer and consumer AWS accounts provisioned by a data administrator.</li> 
 <li>A secret in&nbsp;<a href="https://aws.amazon.com/secrets-manager/" rel="noopener" target="_blank">AWS Secrets Manager</a>&nbsp;storing the master user credentials for the Amazon Redshift cluster or workgroup in the producer and consumer AWS accounts. 
  <ul> 
   <li>The data administrators are responsible for creating secrets.</li> 
   <li>The data producers and consumers can obtain the&nbsp;<a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html" rel="noopener" target="_blank">Amazon Resource Name (ARN)</a>&nbsp;of the secrets from the data administrators during the environment or environment profile creation steps.</li> 
  </ul> </li> 
</ul> 
<p>Amazon DataZone uses&nbsp;<a href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-datashare-using.html" rel="noopener" target="_blank">Amazon Redshift Datashares</a>&nbsp;to share data across clusters and accounts. There are specific requirements and&nbsp;<a href="https://docs.aws.amazon.com/lake-formation/latest/dg/notes-rs-datashare.html" rel="noopener" target="_blank">limitations</a>&nbsp;for using Amazon Redshift datashares.</p> 
<ul> 
 <li>For cross-account data sharing, both the producer and consumer clusters must be encrypted. See <a href="https://docs.aws.amazon.com/redshift/latest/dg/datashare-considerations.html#encryption" rel="noopener" target="_blank">Cluster encryption section of datashare-considerations</a> for more information about the encryption process.</li> 
 <li>Data sharing is supported only for provisioned ra3 cluster types (ra3.16xlarge, ra3.4xlarge, and ra3.xlplus) and Amazon Redshift Serverless.</li> 
</ul> 
<h2>Walkthrough:</h2> 
<p>The following are the high level steps to configure cross-account access. We’ve provided step-by-step instructions in the following sections.</p> 
<ol> 
 <li>Create an Amazon DataZone domain in the producer account. The data administrator creates an Amazon DataZone domain.</li> 
 <li>Request Amazon DataZone domain association from the producer account to the consumer account.</li> 
 <li>Accept the domain association request in the consumer account. The data administrator accepts the domain association.</li> 
 <li>Add data users to the Amazon DataZone domain.</li> 
 <li>Create the necessary publish project for <a href="https://aws.amazon.com/glue" rel="noopener" target="_blank">AWS Glue</a> and Amazon Redshift in the producer account.</li> 
 <li>Create AWS Glue and Amazon Redshift environments to publish the data assets in the producer account.</li> 
 <li>Create and run a data source for AWS Glue and Amazon Redshift to publish assets into the business catalog.</li> 
 <li>Create subscribe projects for AWS Glue and Amazon Redshift.</li> 
 <li>Create AWS Glue and Amazon Redshift environment profiles and environments in the subscribe project</li> 
 <li>Subscribe to AWS Glue and Amazon Redshift tables. Consume the data using Athena and Amazon redshift editors. This step is performed by the data subscriber.</li> 
</ol> 
<h3>Create the Amazon DataZone domain in the producer account</h3> 
<p>Amazon DataZone domains serve as high-level organizational units for assets, users, and projects, facilitating cross-team and cross-account collaboration. This step focusses on creating the Amazon DataZone domain in the producer account.</p> 
<ol> 
 <li>Sign in to the producer account <a href="https://console.aws.amazon.com/datazone" rel="noopener" target="_blank">AWS Management Console for Amazon DataZone</a>&nbsp;using the data administrator credentials.</li> 
 <li>Create an Amazon DataZone domain titled <code>Demo_cross_account_domain</code> using the instructions at&nbsp;<a href="https://docs.aws.amazon.com/datazone/latest/userguide/create-domain.html" rel="noopener" target="_blank">create domains</a>.</li> 
 <li>On the <strong>Create domain</strong> screen, select&nbsp;<strong>Quick setup</strong>&nbsp;checkbox to automate several configuration steps, saving time and reducing the potential for setup errors. Quick setup enables two default blueprints and creates the default environment profiles for the data lake and data warehouse default blueprints.</li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img3.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-75747 size-full" height="746" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img3.png" /></a></p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img5.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-75749 size-full" height="874" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img5.png" /></a><br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img7.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-75751 size-full" height="287" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img7.png" /></a></p> 
<h3>Request Amazon DataZone domain association from the producer account to the consumer account</h3> 
<p>To associate the Amazon DataZone domain with the consumer account, the producer account&nbsp;<a href="https://docs.aws.amazon.com/datazone/latest/userguide/working-with-associated-accounts.html#invite-account-to-associate" rel="noopener" target="_blank">requests a domain association</a>. This involves providing necessary information about the consumer account and granting appropriate permissions for data access and management.</p> 
<ol> 
 <li>Sign in to the&nbsp;<a href="https://console.aws.amazon.com/datazone" rel="noopener" target="_blank">Amazon DataZone console</a>&nbsp;of the producer account using the data administrator credentials.</li> 
 <li>Navigate to the domain detail page, and then scroll down and select the <strong>Associated Accounts</strong> tab.</li> 
 <li>Enter the consumer account IDs that you want to request association. Choose <strong>Add another account</strong> if you want to add more than one account. When you’re satisfied with the list of account IDs, choose&nbsp;<strong>Request association</strong>. 
  <ul> 
   <li>Use the latest (AWS RAM&nbsp;<code>DataZonePortalReadWrite</code>&nbsp;policy when requesting the account association. This policy allows users in the consumer account to execute Amazon DataZone APIs and to use the data portal interface.</li> 
  </ul> </li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img9.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-75753 size-full" height="677" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img9.png" style="margin: 10px 0px 10px 0px;" width="1428" /></a></p> 
<h3>Accept an account association request from an Amazon DataZone domain</h3> 
<p>This step focuses on accepting the account association request from the Amazon DataZone domain in the consumer account. This allows the consumer account to be linked with the Amazon DataZone domain to enable data sharing and collaboration between the producer and consumer accounts.</p> 
<ol> 
 <li>Sign in to the consumer account and go to the <a href="https://console.aws.amazon.com/datazone" rel="noopener" target="_blank">Amazon DataZone console</a> &nbsp;in the same AWS Region as the domain. On the Amazon DataZone home page, choose <strong>View requests</strong>.</li> 
 <li>Select the name of the inviting Amazon DataZone domain and choose <strong>Review request</strong>.</li> 
 <li>Choose&nbsp;<strong>Accept association</strong>, you should see the <code>Demo_cross_account_domain</code>&nbsp;state as associated in the <strong>Associated domains </strong>screen</li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img11.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-75755 size-full" height="409" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img11.png" style="margin: 10px 0px 10px 0px;" width="982" /></a></p> 
<ol start="4"> 
 <li>Choose the domain for which you want to enable an environment blueprint.</li> 
 <li>From the <strong>Blueprints</strong> list, choose either the <strong>DefaultDataLake</strong> blueprint</li> 
 <li>On the Permissions and resources page, for enabling the <strong>DefaultDataLake</strong> blueprint, for <strong>Glue Manage Access role</strong>, specify a new role that grants Amazon DataZone authorization to ingest and manage access to tables in AWS Glue and AWS Lake Formation.</li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img13.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-75757 size-full" height="1085" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img13.png" style="margin: 10px 0px 10px 0px;" width="996" /></a></p> 
<ol start="7"> 
 <li>Repeat steps 4 to 6 to enable the <strong>DefaultDataWarehouse </strong>blueprint<strong> by choosing DefaultDataWarehouse instead of DefaultDataLake</strong></li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img15.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-75759 size-full" height="774" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img15.png" style="margin: 10px 0px 10px 0px;" width="992" /></a></p> 
<h3>Add data users to the Amazon DataZone domain</h3> 
<p>To grant access to the Amazon DataZone data portal from the console for data publisher and data Subscriber IAM users, use the following steps to add them in the User Management section of the Amazon DataZone domain. See&nbsp;<a href="https://docs.aws.amazon.com/datazone/latest/userguide/user-management-console.html" rel="noopener" target="_blank">Manage users in the Amazon DataZone console</a>&nbsp;for additional details.</p> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/datazone" rel="noopener" target="_blank">Amazon DataZone console</a> as a data administrator using the producer account.</li> 
 <li>Select the Amazon DataZone domain and, in the&nbsp;<strong>User management&nbsp;</strong>section,<strong>&nbsp;</strong>choose<strong>&nbsp;</strong><strong>Add</strong><strong>&nbsp;</strong>and select<strong>&nbsp;</strong><strong>Add IAM users</strong>.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img17.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-75761 size-full" height="249" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img17.png" style="margin: 10px 0px 10px 0px;" width="1431" /></a></li> 
 <li>On the&nbsp;<strong>Add users</strong>&nbsp;page, choose&nbsp;<strong>Current account</strong>&nbsp;and add the user ARN of the data producer and choose&nbsp;<strong>Add users</strong>.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img19.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75763" height="544" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img19.png" style="margin: 10px 0px 10px 0px;" width="1428" /></a></li> 
 <li>Next choose<strong>&nbsp;</strong><strong>Associated account</strong>, and enter the data subscriber user’s ARN and add the user by choosing&nbsp;<strong>Add users</strong>.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img21.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75765" height="900" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img21.png" style="margin: 10px 0px 10px 0px;" width="1274" /></a></li> 
</ol> 
<h3>Create the publish project for AWS Glue and Amazon Redshift</h3> 
<p>This step focuses on creating the publish project for AWS Glue and Amazon Redshift in the producer account.&nbsp;The project will be used to publish data from your data sources to the appropriate AWS services.</p> 
<ol> 
 <li>Using the producer account, sign in to the&nbsp;<a href="https://console.aws.amazon.com/datazone" rel="noopener" target="_blank">Amazon DataZone console</a> as a data publisher.</li> 
 <li>Select&nbsp;<strong>View domains</strong>&nbsp;and select the&nbsp;<strong>demo_cross_account_domain</strong>.</li> 
 <li>Choose the&nbsp;<strong>Open data portal</strong>&nbsp;link and sign in to the data portal.</li> 
 <li>Choose <strong>Create New Project</strong>&nbsp;and create a project named&nbsp;<code>Glue_Publish_Project</code>&nbsp;for publishing AWS Glue data assets and create the project under <strong>demo_cross_account_domain</strong>.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img23.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75767" height="1143" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img23.png" style="margin: 10px 0px 10px 0px;" width="1433" /></a></li> 
 <li>Create another project named <code>Redshift_Publish_Project</code> for publishing Amazon Redshift data assets, also under the <strong>demo_cross_account_domain</strong>.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img24.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75768" height="747" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img24.png" style="margin: 10px 0px 10px 0px;" width="936" /></a></li> 
</ol> 
<h3>Create AWS Glue and Amazon Redshift environments to publish the data assets</h3> 
<p>In this step, you set up AWS Glue and Amazon Redshift environments in the producer account to share data assets. The required infrastructure, such as the AWS Glue Data Catalog and Redshift cluster for storing data, should already be in place. After setup, this will allow the consumer account to access and use the shared data assets. See&nbsp;<a href="https://docs.aws.amazon.com/datazone/latest/userguide/create-new-environment.html" rel="noopener" target="_blank">Create a new environment</a>&nbsp;for detailed instructions on creating a new environment.</p> 
<h4>Create the AWS Glue environment and a new AWS Glue table</h4> 
<ol> 
 <li>In the same Amazon DataZone domain <strong>demo_cross_account_domain, </strong>choose<strong> Browse Project </strong>and select the&nbsp;<strong>Glue_Publish_Project</strong>&nbsp;and create&nbsp;<strong>Glue_Publish_Environment</strong>&nbsp;using the default&nbsp;<strong>DataLakeProfile</strong>.</li> 
 <li>Leave the&nbsp;<strong>producer_glue_db_name</strong>, <strong>consumer_glue_db_name</strong> and&nbsp;<strong>Workgroup_name</strong> blank.</li> 
 <li>Choose&nbsp;<strong>Create Environment</strong>&nbsp;and wait for the process to complete.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img27.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75771" height="792" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img27.png" style="margin: 10px 0px 10px 0px;" width="779" /></a></li> 
 <li>After the environment is created, browse the list of available projects and choose <strong>Glue_publish_project</strong>.</li> 
 <li>Next, navigate to the <strong>Glue_Publish_Environment</strong>, and under&nbsp;<strong>Analytics tools</strong>, choose&nbsp;<strong>Amazon Athena</strong>&nbsp;to open the Athena query editor<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img29.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75773" height="903" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img29.png" style="margin: 10px 0px 10px 0px;" width="1007" /></a></li> 
 <li>Choose <strong>Open Athena</strong>&nbsp;and make sure that&nbsp;<strong>Glue_Publish_Environment</strong>&nbsp;is selected in the&nbsp;<strong>Amazon DataZone environment</strong>&nbsp;dropdown at the upper right and that in <strong>Data</strong> on the left, <strong>glue_publish_environment_pub_db</strong> is selected as the <strong>Database</strong>.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img31.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75775" height="945" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img31.png" style="margin: 10px 0px 10px 0px;" width="1001" /></a></li> 
 <li>Create a new AWS Glue table for publishing to Amazon DataZone. Paste the following create table as select (CTAS) query script in the <strong>Query</strong> window and run it to create a new table named <code>mkt_sls_table</code>. The script creates a table with sample marketing and sales data. <pre><code class="lang-sql">CREATE TABLE mkt_sls_table AS
SELECT 146776932 AS ord_num, 23 AS sales_qty_sld, 23.4 AS wholesale_cost, 45.0 as lst_pr, 43.0 as sell_pr, 2.0 as disnt, 12 as ship_mode,13 as warehouse_id, 23 as item_id, 34 as ctlg_page, 232 as ship_cust_id, 4556 as bill_cust_id
UNION ALL SELECT 46776931, 24, 24.4, 46, 44, 1, 14, 15, 24, 35, 222, 4551
UNION ALL SELECT 46777394, 42, 43.4, 60, 50, 10, 30, 20, 27, 43, 241, 4565
UNION ALL SELECT 46777831, 33, 40.4, 51, 46, 15, 16, 26, 33, 40, 234, 4563
UNION ALL SELECT 46779160, 29, 26.4, 50, 61, 8, 31, 15, 36, 40, 242, 4562
UNION ALL SELECT 46778595, 43, 28.4, 49, 47, 7, 28, 22, 27, 43, 224, 4555
UNION ALL SELECT 46779482, 34, 33.4, 64, 44, 10, 17, 27, 43, 52, 222, 4556
UNION ALL SELECT 46779650, 39, 37.4, 51, 62, 13, 31, 25, 31, 52, 224, 4551
UNION ALL SELECT 46780524, 33, 40.4, 60, 53, 18, 32, 31, 31, 39, 232, 4563
UNION ALL SELECT 46780634, 39, 35.4, 46, 44, 16, 33, 19, 31, 52, 242, 4557
UNION ALL SELECT 46781887, 24, 30.4, 54, 62, 13, 18, 29, 24, 52, 223, 4561</code></pre> <p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img33.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75777" height="439" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img33.png" style="margin: 10px 0px 10px 0px;" width="1429" /></a></p></li> 
 <li>Go to the Tables&nbsp;and&nbsp;Views&nbsp;section and verify that the&nbsp;<code>mkt_sls_table</code>&nbsp;table was successfully created.</li> 
</ol> 
<h4>Create the Amazon Redshift publish environment and a new Redshift table</h4> 
<ol> 
 <li>Staying in the same Amazon DataZone domain <strong>demo_cross_account_domain, </strong>choose<strong> Browse Project, </strong>to create an Amazon Redshift publish environment, select the&nbsp;<strong>Redshift_Publish_Project</strong>&nbsp;and create <strong>Redshift_Publish_Environment</strong>&nbsp;using the default&nbsp;data warehouse profile.</li> 
 <li>&nbsp;To configure environment parameters, enter the name of your Amazon Redshift cluster or workgroup, specify the database name and enter the AWS Secrets Manager secret ARN for the Redshift cluster or workgroup. You need to make sure that the secret in Secrets Manager includes the following tags. These tags help Amazon DataZone implement proper access control so that only authorized users within the correct Amazon DataZone project and domain can access the Amazon Redshift resource: 
  <ol type="a"> 
   <li>For <strong>Amazon Redshift cluster</strong>: <code>DataZone.rs.cluster: &lt;cluster_name:database name&gt;</code></li> 
   <li>For <strong>Amazon Redshift Serverless workgroup</strong>: <code>DataZone.rs.workgroup:&nbsp; &lt;workgroup_name:database_name&gt;</code></li> 
   <li><strong>AmazonDataZoneProject:</strong> <code>&lt;projectID&gt;</code></li> 
   <li><strong>AmazonDataZoneDomain</strong>: <code>&lt;domainID&gt;</code>For more information for creating redshift database user secret in secret manager, see&nbsp;Storing database credentials in AWS Secrets Manager.</li> 
  </ol> </li> 
</ol> 
<p style="padding-left: 40px;">For more information for creating redshift database user secret in secret manager, see <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/data-api-secrets.html" rel="noopener" target="_blank">Storing database credentials in AWS Secrets Manager</a>.</p> 
<ol start="3"> 
 <li>Note that the database user you provide in Secrets Manager must have superuser permissions. Data publishers should work with the data administrator to get the details of the Redshift cluster or workgroup, database name, and secret ARN.</li> 
 <li>The schema is optional.</li> 
 <li>Choose <strong>Create Environment</strong> and wait for the process to complete.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img35.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75779" height="1215" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img35.png" style="margin: 10px 0px 10px 0px;" width="1432" /></a></li> 
 <li>Verify that the environment is created successfully without errors.</li> 
 <li>Browse the list of available projects and select <code>Redshift_publish_project</code>. Navigate to <code>Redshift_publish_environment</code>.</li> 
 <li>Under Analytics tools, choose Amazon Redshift to open the Amazon Redshift query editor.</li> 
 <li>Select the Redshift cluster that you want to connect, choose Save and then choose Create Connection using temporary credentials with your IAM identity.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img37.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75781" height="872" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img37.png" style="margin: 10px 0px 10px 0px;" width="925" /></a></li> 
 <li>Create a new Redshift table. You can use the CTAS query to create a new table named <code>rs_sls_tbl</code>. Use the provided CTAS script, which creates a table with sample sales data in the <code>datazone_env_redshift_publish_environment</code> schema. 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE TABLE "datazone_env_redshift_publish_environment"."rs_sls_tbl" AS
SELECT 146776932 AS ord_num, 23 AS sales_qty_sld, 23.4 AS wholesale_cost, 45.0 as lst_pr, 43.0 as sell_pr, 2.0 as disnt, 12 as ship_mode,13 as warehouse_id, 23 as item_id, 34 as ctlg_page, 232 as ship_cust_id, 4556 as bill_cust_id
UNION ALL SELECT 46776931, 24, 24.4, 46, 44, 1, 14, 15, 24, 35, 222, 4551
UNION ALL SELECT 46777394, 42, 43.4, 60, 50, 10, 30, 20, 27, 43, 241, 4565
UNION ALL SELECT 46777831, 33, 40.4, 51, 46, 15, 16, 26, 33, 40, 234, 4563
UNION ALL SELECT 46779160, 29, 26.4, 50, 61, 8, 31, 15, 36, 40, 242, 4562
UNION ALL SELECT 46778595, 43, 28.4, 49, 47, 7, 28, 22, 27, 43, 224, 4555
UNION ALL SELECT 46779482, 34, 33.4, 64, 44, 10, 17, 27, 43, 52, 222, 4556
UNION ALL SELECT 46779650, 39, 37.4, 51, 62, 13, 31, 25, 31, 52, 224, 4551
UNION ALL SELECT 46780524, 33, 40.4, 60, 53, 18, 32, 31, 31, 39, 232, 4563
UNION ALL SELECT 46780634, 39, 35.4, 46, 44, 16, 33, 19, 31, 52, 242, 4557
UNION ALL SELECT 46781887, 24, 30.4, 54, 62, 13, 18, 29, 24, 52, 223, 4561</code></pre> 
  </div> </li> 
 <li>&nbsp;Make sure that the&nbsp;<code>rs_sls_tbl</code>&nbsp;table is successfully created.</li> 
</ol> 
<h3>Publish assets into the common business catalog</h3> 
<p>In this step, you create and run the Amazon DataZone data sources for&nbsp;<a href="https://docs.aws.amazon.com/datazone/latest/userguide/create-glue-data-source.html" rel="noopener" target="_blank">AWS Glue&nbsp;</a>and&nbsp;<a href="https://docs.aws.amazon.com/datazone/latest/userguide/create-redshift-data-source.html" rel="noopener" target="_blank">Amazon Redshift</a>. You will then publish the data assets from these data sources.</p> 
<p>The Amazon DataZone data sources allow you to connect to various data sources, including databases, data warehouses, and data lakes, and ingest metadata into Amazon DataZone. By creating and running these data sources, you can make your data available for analysis, transformation, and sharing within your organization.</p> 
<p>After the data sources are set up, you can publish the data assets from these sources to make them accessible to other users and applications. This process involves mapping the data assets to the appropriate business terms and metadata, making sure that the data is properly described and categorized.</p> 
<h4>Add an AWS Glue data source to publish the new AWS Glue table.</h4> 
<ol> 
 <li>Stay signed in the producer account and <a href="https://console.aws.amazon.com/datazone" rel="noopener" target="_blank">Amazon DataZone console</a> as a data publisher.</li> 
 <li>Choose&nbsp;<strong>Select project</strong>&nbsp;from the top navigation pane and select the&nbsp;<strong>Glue_Publish_Project</strong>&nbsp;that you want to add the data source to.</li> 
 <li>Select the&nbsp;<strong>Glue_Publish_Environment</strong>.</li> 
 <li>Choose&nbsp;<strong>Create data source</strong>. Enter <code>glue-publish-datasource</code> as the name.</li> 
 <li>Under&nbsp;<strong>Data source type</strong>, choose&nbsp;<strong>AWS Glue</strong>.</li> 
 <li>Under&nbsp;<strong>Select an environment</strong>, select&nbsp;<strong>Glue_Publish_Environment</strong>.</li> 
 <li>Under&nbsp;<strong>Data selection</strong>, select the AWS Glue database <strong>glue_publish_environment_pub_db</strong>, enter your table selection criteria as&nbsp;“*“, and then&nbsp;and choose&nbsp;<strong>Next</strong>.</li> 
 <li>Leave all other setting as default and choose&nbsp;<strong>Next</strong>.</li> 
 <li>For&nbsp;<strong>Run Preference</strong>,&nbsp;select <strong>Run on demand&nbsp;</strong>to ingest metadata from the specified AWS Glue tables into Amazon DataZone.</li> 
 <li>Review and choose&nbsp;<strong>Create</strong>.</li> 
 <li>After the data source has been created choose&nbsp;<strong>Run</strong>. The&nbsp;<code>mkt_sls_table</code>&nbsp;will be listed in the inventory and available to publish.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img39.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75783" height="292" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img39.png" style="margin: 10px 0px 10px 0px;" width="1429" /></a></li> 
 <li>Select the <code>mkt_sls_table</code> table and review the metadata that was generated. Choose&nbsp;<strong>Accept All</strong>&nbsp;if you’re satisfied with the metadata.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/21/bdb-4417-img41.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75785" height="569" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img41.png" style="margin: 10px 0px 10px 0px;" width="1429" /></a></li> 
 <li>Choose <strong>Publish Asset</strong>&nbsp;and the&nbsp;<code>mkt_sls_table</code><em>&nbsp;</em>table will be published to the business data catalog, making it discoverable and understandable across your organization.</li> 
</ol> 
<h4>Add an Amazon Redshift data source to publish the new Amazon Redshift table.</h4> 
<ol> 
 <li>Stay signed in the producer account and <a href="https://console.aws.amazon.com/datazone" rel="noopener" target="_blank">Amazon DataZone console</a> as a data publisher.</li> 
 <li>Choose <strong>Select project</strong>&nbsp;from the top navigation pane and select the&nbsp;<strong>Redshift_Publish_Project</strong>&nbsp;that you want to add the data source to.</li> 
 <li>Choose the&nbsp;<strong>Redshift_Publish_Environment</strong>.</li> 
 <li>Choose&nbsp;<strong>Create data source</strong>. Enter <code>rs-publish-datasource</code> as the name.</li> 
 <li>Under&nbsp;<strong>Data source type</strong>, select&nbsp;<strong>Amazon Redshift</strong>.</li> 
 <li>Under&nbsp;<strong>Select an environment</strong>, select&nbsp;<strong>Redshift_Publish_Environment</strong>.</li> 
 <li>Under&nbsp;<strong>Redshift Credentials</strong>, enter the Redshift cluster and secret details provided by the data administrator.</li> 
 <li>Under&nbsp;<strong>Data Selection,</strong>&nbsp;select the database&nbsp;<strong>dev</strong>&nbsp;and schema <strong>datazone_env_redshift_publish_environment</strong>.</li> 
 <li>Keep other setting as default and choose&nbsp;<strong>Next</strong>.</li> 
 <li>For&nbsp;<strong>Run Preference</strong>,&nbsp;select <strong>Run on Demand</strong>.</li> 
 <li>Choose&nbsp;<strong>Save</strong>. After the data source is created, choose&nbsp;<strong>Run</strong>. The data source runs and the&nbsp;<code>rs_sls_tbl</code><em>&nbsp;</em>will be listed in the inventory and available to publish.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img43.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75813" height="309" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img43.png" style="margin: 10px 0px 10px 0px;" width="1431" /></a></li> 
 <li>Select the <code>rs_sls_tbl</code> table and review the metadata that was generated. Choose&nbsp;<strong>Accept All</strong>&nbsp;if you are satisfied with the metadata.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img45.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75815" height="586" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img45.png" style="margin: 10px 0px 10px 0px;" width="1428" /></a></li> 
 <li>Choose&nbsp;<strong>Publish Asset</strong>&nbsp;and the&nbsp;<code>rs_sls_table</code><em>&nbsp;</em>table will be published to the business data catalog.</li> 
</ol> 
<h3>Create subscribe projects for AWS Glue and Amazon Redshift</h3> 
<p>In this step, you create the&nbsp;<a href="https://docs.aws.amazon.com/datazone/latest/userguide/create-new-project.html" rel="noopener" target="_blank">projects</a>&nbsp;for subscribing to AWS Glue and Amazon Redshift data assets within your Amazon DataZone domain.</p> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/datazone" rel="noopener" target="_blank">Amazon DataZone console</a> as a data subscriber IAM user using the consumer account.</li> 
 <li>Choose&nbsp;<strong>Associated domains</strong>&nbsp;and select the&nbsp;<strong>demo_cross_account_domain</strong>.</li> 
 <li>Select the&nbsp;<strong>Open data portal </strong>link&nbsp;and&nbsp;sign in&nbsp;to the&nbsp;data portal.</li> 
 <li>Choose&nbsp;<strong>Create New Project</strong>&nbsp;and create a project named&nbsp;<code>Glue_Subscribe_Project</code>&nbsp;for subscribing to the AWS Glue data assets.</li> 
 <li>Create another project named <code>Redshift_Subscribe_Project</code> for subscribing to the Redshift data assets.</li> 
</ol> 
<h3>Create AWS Glue and Amazon Redshift environment profiles</h3> 
<p>In this step, you will set up the <a href="https://docs.aws.amazon.com/datazone/latest/userguide/create-environment-profile.html" rel="noopener" target="_blank">environment profiles</a>&nbsp;and&nbsp;<a href="https://docs.aws.amazon.com/datazone/latest/userguide/create-new-environment.html" rel="noopener" target="_blank">environments</a>&nbsp;for AWS Glue and Amazon Redshift in your Amazon DataZone projects. This will allow you to connect and interact with resources across AWS accounts.</p> 
<p>The purpose of environment profiles in Amazon DataZone is to streamline the process of environment creation. By using environment profiles, you can preconfigure essential placement information such as AWS account and AWS Region. In this solution, you will configure environment profiles with placement information pointing to your consumer account.</p> 
<p>You will also create an Amazon DataZone environment from the profiles you are about to create. This will provision the necessary resources in the consumer account and establish the connections between the Amazon DataZone domain and the consumer account. After the environments are created, you can work with AWS Glue and Amazon Redshift assets seamlessly across different AWS accounts within your Amazon DataZone ecosystem.</p> 
<h4>Create an AWS Glue profile and environment</h4> 
<ol> 
 <li>Stay signed in the consumer account’s&nbsp;Amazon DataZone console as a data subscriber IAM, select the Environments tab and then choose&nbsp;Create environment<strong> profile</strong>.</li> 
 <li>Configure the fields as follows: 
  <ol type="a"> 
   <li>Name: Enter <code>glue_subscribe-env-profile</code>.</li> 
   <li>Owner: The project where the profile is being created is selected by default in this field. Verify that it’s <code>Glue_Subscribe_Project</code>.</li> 
   <li>Blueprint: Select Default Data Lake.</li> 
   <li>AWS account parameters: Enter the consumer AWS account number and select the Region.</li> 
   <li>Authorized projects: Select All projects.</li> 
   <li>Publishing: Select Publish from any database.</li> 
   <li>Choose Create Environment Profile.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img47-1.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75874" height="946" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img47-1.png" style="margin: 10px 0px 10px 0px;" width="803" /></a><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img49-1.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75863" height="777" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img49-1.png" style="margin: 10px 0px 10px 0px;" width="804" /></a></li> 
  </ol> </li> 
 <li>On the Create environment page, enter the following: 
  <ol type="a"> 
   <li><strong>Name</strong>: Enter <code>glue_subscribe_environment</code>.</li> 
   <li>Verify that the <strong>Environment profile</strong> is set to <strong>glue_subscribe-env-profile</strong>.</li> 
  </ol> </li> 
 <li>(Optional) <strong>Parameters</strong>: Enter the <strong>Producer glue db name, Consumer glue db name</strong>, and <strong>Workgroup name</strong>.</li> 
 <li>Choose <strong>Create environment</strong>. <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img51-1.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75865" height="1105" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img51-1.png" style="margin: 10px 0px 10px 0px;" width="773" /></a></li> 
 <li>It takes a few minutes for the environment to be created. Verify that the environment creation is successful without any errors.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img53-1.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75867" height="835" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img53-1.png" style="margin: 10px 0px 10px 0px;" width="865" /></a></li> 
</ol> 
<ol> 
 <li></li> 
</ol> 
<h4>Create a Redshift environment profile and environment</h4> 
<ol> 
 <li>Staying in the consumer account’s&nbsp;Amazon DataZone management console as a data subscriber IAM user, navigate to the&nbsp;<strong>Redshift_Subscribe_Project</strong>&nbsp;you created previously.</li> 
 <li>Select the <strong>Environments</strong> tab and then choose&nbsp;<strong>Create environment profile</strong>.</li> 
 <li>Configure the fields as follows: 
  <ol type="a"> 
   <li><strong>Name</strong>: Enter&nbsp;<code>redshift_subscribe-env-profile<strong>.</strong></code></li> 
   <li><strong>Owner</strong>: Verify that <strong>Project</strong> is set to&nbsp;<strong>Redshift_Subscribe_Project</strong>.</li> 
   <li><strong>Blueprint</strong>: Select&nbsp;<strong>Default Data Warehouse</strong>.</li> 
   <li><strong>Parameter set</strong>:&nbsp;Select <strong>Enter my own</strong>.</li> 
   <li><strong>AWS account parameters</strong>: Enter the consumer AWS account number and select the Region.</li> 
   <li><strong>Parameters</strong>: Select either&nbsp;<strong>Amazon Redshift Cluster</strong>&nbsp;or&nbsp;<strong>Amazon Redshift Serverless</strong>&nbsp;in the consumer account. 
    <ul> 
     <li><strong>AWS Secret ARN</strong>: Enter the AWS Secrets Manager secret ARN for the Redshift cluster or workgroup. You need to make sure that the secret in Secrets Manager includes the following tags. These tags help Amazon DataZone implement proper access control so that only authorized users within the correct Amazon DataZone project and domain can access the Amazon Redshift resource. 
      <ol type="a"> 
       <li><strong>AmazonDataZoneDomain</strong>: [<code>Domain_ID</code>]</li> 
       <li><strong>AmazonDataZoneProject</strong>:&nbsp; [<code>Project_ID</code>]</li> 
      </ol> </li> 
    </ul> <p>For more information for creating redshift database user secret in secret manager, see&nbsp;<a href="https://docs.aws.amazon.com/redshift/latest/mgmt/data-api-secrets.html" rel="noopener" target="_blank">Storing database credentials in AWS Secrets Manager</a>.</p> <p>Note that the database user you provide in AWS Secrets Manager must have superuser permissions. Data publishers should work with the data administrator to get the details of the Redshift cluster or workgroup, database name, and secret ARN.</p> 
    <ul> 
     <li><strong>Redshift cluster name</strong>: Enter the name of the Amazon Redshift cluster or Amazon Redshift Serverless workgroup.</li> 
     <li><strong>Database name</strong>: Enter the name of the database within the selected Amazon Redshift cluster or Amazon Redshift Serverless workgroup</li> 
    </ul> </li> 
   <li><span style="color: #000000;"><strong>Authorized projects</strong>: Select <strong>All projects</strong>.</span></li> 
   <li><span style="color: #000000;"><strong>Publishing</strong>:&nbsp;Select <strong>Publish any schema</strong>.</span></li> 
  </ol> </li> 
 <li><span style="color: #000000;">Choose <strong>Create environment profile</strong>.</span><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img47-1.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75874" height="946" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img47-1.png" style="margin: 10px 0px 10px 0px;" width="803" /></a><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img49-upd.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75876" height="826" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img49-upd.png" style="margin: 10px 0px 10px 0px;" width="624" /></a><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img59.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75829" height="773" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img59.png" style="margin: 10px 0px 10px 0px;" width="804" /></a></li> 
 <li>Create an environment from this profile: Create an environment from this profile: 
  <ol> 
   <li><strong>Name:</strong> Enter <code>redshift_subscribe_environment</code>.</li> 
   <li>Verify that the <strong>Environment profile</strong> is set to <strong>redshift_subscribe-env-profile</strong>.</li> 
  </ol> </li> 
 <li>Choose <strong>Create Environment</strong>.<a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img61.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75831" height="1056" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img61.png" width="770" /></a></li> 
 <li></li> 
</ol> 
<p>It takes a few minutes for the environment to be created. Verify that the environment creation is successful without any errors.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img63.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75833" height="709" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img63.png" style="margin: 10px 0px 10px 0px;" width="859" /></a></p> 
<h3>Subscribe to the AWS Glue and Redshift tables</h3> 
<p>In this step, you will subscribe AWS Glue and Amazon redshift tables published by the data producer.</p> 
<h4>Subscribe to the AWS Glue table</h4> 
<ol> 
 <li>Sign in to the&nbsp;<a href="https://console.aws.amazon.com/datazone" rel="noopener" target="_blank">Amazon DataZone console</a>&nbsp;of the consumer account using the data subscriber credentials and navigate to the&nbsp;<strong>Glue_Subscribe_project</strong>&nbsp;you created previously.</li> 
 <li>Search for the <strong>Market Sales Table</strong> in the&nbsp;Search bar.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img65.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75835" height="943" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img65.png" style="margin: 10px 0px 10px 0px;" width="762" /></a></li> 
 <li>Select the <strong>Market Sales Table</strong>&nbsp;and choose&nbsp;<strong>Subscribe</strong>.</li> 
 <li>In the <strong>Subscribe</strong>&nbsp;pop-up window, provide the following information: 
  <ul> 
   <li><strong>Project</strong>: Enter the name of the project that you want to subscribe to the asset. By default this will be&nbsp;<strong>Glue_Subscribe_Project</strong>.</li> 
   <li>Enter a justification for your subscription request.</li> 
  </ul> </li> 
 <li>Choose <strong>Subscribe</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img67.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75837" height="538" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img67.png" style="margin: 10px 0px 10px 0px;" width="790" /></a></li> 
 <li>Switch to the data publisher role to approve the subscription request, then back to data subscriber after choosing <strong>Approve</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img69.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75839" height="1284" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img69.png" style="margin: 10px 0px 10px 0px;" width="1427" /></a></li> 
 <li>Select the <strong>Glue_subscribe_project</strong>&nbsp;and choose&nbsp;<strong>Subscribed Assets</strong>. Verify that the&nbsp;<strong>Market Sales Table</strong> is added to your environment.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img71.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75841" height="645" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img71.png" style="margin: 10px 0px 10px 0px;" width="1440" /></a></li> 
 <li>Navigate to the&nbsp;<strong>Amazon Athena query editor</strong>&nbsp;using the link in the project’s home page.</li> 
 <li>Choose <strong>OPEN AMAZON ATHENA</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img73.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75843" height="202" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img73.png" style="margin: 10px 0px 10px 0px;" width="900" /></a></li> 
 <li>You will now be automatically routed to the Athena console, make sure that the <strong>Amazon DataZone Environment</strong>&nbsp;is set to&nbsp;<strong>glue_subscribe_environment</strong>.</li> 
 <li>For <strong>Database</strong>, select&nbsp;<strong>glue_subscribe_environment_sub_db</strong>.</li> 
 <li>You should see the <code>mkt_sls_table</code> in the <strong>Tables</strong> list. Preview the table by choosing the three-dot menu next to the table name and selecting <strong>Preview Table<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img75.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75845" height="983" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img75.png" style="margin: 10px 0px 10px 0px;" width="905" /></a></strong></li> 
 <li>Review the table preview results. You will be able to see all the sales related data from the <code>mkt_sls_table<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img77.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75847" height="739" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img77.png" style="margin: 10px 0px 10px 0px;" width="1429" /></a></code></li> 
</ol> 
<h4>Subscribe to the Redshift table</h4> 
<ol> 
 <li>Stay signed in to the Amazon DataZone management console as the data subscriber,&nbsp;Choose&nbsp;<strong>Select project</strong>&nbsp;from the top navigation pane and select the&nbsp;<strong>Redshift_Subscribe_project</strong>.</li> 
 <li>Search for <strong>Sales Table</strong>&nbsp;in the search bar, and select the&nbsp;<strong>Sales Table</strong>.</li> 
 <li>In the&nbsp;<strong>Subscribe</strong>&nbsp;pop-up window, provide the following information: 
  <ul> 
   <li><strong>Project</strong>: Enter the name of the project that you want to subscribe to the asset. By default this will be&nbsp;<strong>Redshift_Subscribe_Project</strong>.</li> 
   <li>Enter a justification for your subscription request.</li> 
  </ul> </li> 
 <li>Choose&nbsp;<strong>Subscribe</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img79.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75849" height="538" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img79.png" style="margin: 10px 0px 10px 0px;" width="790" /></a></li> 
 <li>Switch back to the data publisher who is the producer of the Market Sales Table choose <strong>Approve</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img81.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75851" height="639" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img81.png" style="margin: 10px 0px 10px 0px;" width="790" /></a></li> 
 <li>After the subscription request is approved, switch back to data subscriber.</li> 
 <li>Select the <strong>Redshift_subscribe_project</strong>&nbsp;and choose <strong>Subscribed Assets</strong>. After the <strong>Sales Table</strong> is added to your environment, you can query the data in the table.</li> 
 <li>Select the <strong>Amazon Redshift</strong>&nbsp;link in the right side panel of the project home page and navigate to the&nbsp;<strong>Amazon Redshift query editor.</strong></li> 
 <li>Select <strong>Open Amazon Redshift</strong>&nbsp;and the&nbsp;<strong>Redshift query editor v2</strong>&nbsp;will open in a new tab.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img83.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75853" height="386" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img83.png" style="margin: 10px 0px 10px 0px;" width="928" /></a></li> 
 <li>In the query editor, right-click your Amazon DataZone environment’s Amazon Redshift cluster and select <strong>Create a connection</strong>.</li> 
 <li>Select <strong>Temporary credentials using your IAM identity</strong>&nbsp;for authentication. 
  <ul> 
   <li>If that authentication method isn’t available, open&nbsp;<strong>Account settings</strong>&nbsp;by choosing the gear icon in the bottom left corner, choose&nbsp;<strong>Authenticate with IAM credentials</strong>&nbsp;and choose <strong>Save</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img85.png" rel="noopener" target="_blank"><img alt="" class="alignnone size-full wp-image-75855" height="883" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img85.png" style="margin: 10px 0px 10px 0px;" width="937" /></a></li> 
  </ul> </li> 
 <li>Enter the name of the Amazon DataZone environment’s database to create the connection.</li> 
 <li>Choose <strong>Create connection</strong>.</li> 
 <li>You can now view the Redshift table <code>rs_sls_tbl</code> in the <strong>datazone_env_redshift_subscribe_environment</strong>.</li> 
 <li>Execute the following query to make sure the data is accessible</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT * FROM "dev"."datazone_env_redshift_subscribe_environment"."rs_sls_tbl";</code></pre> 
</div> 
<p>You will be able to preview the <code>rs_sls_tbl</code> which will show the sale data from the table.</p> 
<h2>Clean up</h2> 
<p>To avoid unnecessary future charges, follow these steps:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/datazone/latest/userguide/delete-project.html" rel="noopener" target="_blank">Delete the Amazon DataZone project</a>&nbsp;if you created it as part of this post.</li> 
 <li><a href="https://docs.aws.amazon.com/datazone/latest/userguide/delete-domain.html" rel="noopener" target="_blank">Delete the Amazon DataZone domain</a>&nbsp;if you created it as part of this post.</li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/mgmt/rs-mgmt-shutdown-delete-cluster.html" rel="noopener" target="_blank">Delete the Redshift clusters</a>&nbsp;and&nbsp;<a href="https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_delete-secret.html" rel="noopener" target="_blank">the redshift secrets</a>&nbsp;in both the producer and consumer accounts if you created them as part of the post.</li> 
</ul> 
<h2>Summary</h2> 
<p>Organizations often face significant challenges when trying to share data products across multiple AWS accounts. These challenges stem from the complexity of configuring proper cross-account access permissions and roles while maintaining robust data governance and security controls.</p> 
<p>You can use the solution described in the post to publish and consume data across AWS accounts and make sure that reliable access and consistent data governance is in place. By combining the power of AWS Glue and Amazon Redshift, you can unlock valuable insights and accelerate your data-driven decision-making processes.</p> 
<p>In this post, you followed a step-by-step guide to set up cross-account data sharing using Amazon DataZone domain association. You learned how to publish data assets from a producer account. You also learned how to subscribe to and query the published assets from a consumer account.&nbsp;You can optionally use <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/security-event-logging.html" rel="noopener" target="_blank">AWS Lake Formation access monitoring</a> to view permissions and data access activities. AWS Lake Formation uses <a href="https://aws.amazon.com/cloudtrail" rel="noopener" target="_blank">AWS CloudTrail</a> for historical analysis and CloudTrail retains logs for 90 days by default.</p> 
<p>Now that you’re familiar with the elements involved in cross-account data sharing using Amazon DataZone and your choice of analytical tool, you’re ready to try it with multiple accounts.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img87.png" rel="noopener" target="_blank"><img alt="" class="wp-image-75980 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/27/arun-1.png" width="100" /></a><strong>Arun Pradeep Selvaraj</strong>&nbsp;is a Senior Solutions Architect at AWS. Arun is passionate about working with his customers and stakeholders on digital transformations and innovation in the cloud while continuing to learn, build and reinvent. He is creative, fast-paced, deeply customer-obsessed, and uses the working backwards process to build modern architectures to help customers solve their unique challenges. Connect with him on&nbsp;<a href="http://www.linkedin.com/in/arun-pradeep-selvaraj-77133112b" rel="noopener" target="_blank">LinkedIn</a>.</p> 
<p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img89.png" rel="noopener" target="_blank"><img alt="" class="wp-image-75981 size-full alignleft" height="97" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/27/piyush.png" width="100" /></a><strong>Piyush Mattoo</strong>&nbsp;is a Senior Solution Architect for the Financial Services Data Provider segment at Amazon Web Services. He’s a software technology leader with over a decade of experience building scalable and distributed software systems to enable business value through the use of technology. He has an educational background in Computer Science with a master’s degree in computer and information science from University of Massachusetts. He is based out of Southern California and current interests include camping and nature walks.</p> 
<p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/24/bdb-4417-img91.png" rel="noopener" target="_blank"><img alt="" class="wp-image-75982 size-full alignleft" height="132" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/27/mani.png" width="100" /></a><strong>Mani Yamaraja</strong>&nbsp;is a Senior Customer Solutions Manager for Financial Services Data Provider segment at Amazon Web Services. He has over a decade long experience working with financial services customers enabling their digital transformation journey. Mani adopts a customer centric approach and provides technology solutions working backwards from customer’s business goals. He is passionate about the financial services industry and helps the customers accelerate their cloud based transformation using the proven mechanisms of AWS.</p>
