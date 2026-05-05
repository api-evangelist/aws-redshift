---
title: "Simplify access management with Amazon Redshift and AWS Lake Formation for users in an External Identity Provider"
url: "https://aws.amazon.com/blogs/big-data/simplify-access-management-with-amazon-redshift-and-aws-lake-formation-for-users-in-an-external-identity-provider/"
date: "Thu, 15 Feb 2024 16:02:07 +0000"
author: "Harshida Patel"
feed_url: "https://aws.amazon.com/blogs/big-data/category/database/amazon-redshift/feed/"
---
<p>Many organizations use identity providers (IdPs) to authenticate users, manage their attributes, and group memberships for secure, efficient, and centralized identity management. You might be modernizing your data architecture using <a href="http://aws.amazon.com/redshift" rel="noopener" target="_blank">Amazon Redshift</a> to enable access to your data lake and data in your data warehouse, and are looking for a centralized and scalable way to define and manage the data access based on IdP identities. <a href="https://aws.amazon.com/lake-formation/" rel="noopener" target="_blank">AWS Lake Formation</a> makes it straightforward to centrally govern, secure, and globally share data for analytics and machine learning (ML). Currently, you may have to map user identities and groups to <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) roles, and data access permissions are defined at the IAM role level within Lake Formation. This setup is not efficient because setting up and maintaining IdP groups with IAM role mapping as new groups are created is time consuming and it makes it difficult to derive what data was accessed from which service at that time.</p> 
<p>Amazon Redshift, <a href="https://aws.amazon.com/quicksight" rel="noopener" target="_blank">Amazon QuickSight</a>, and Lake Formation now integrate with the new trusted identity propagation capability in <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> to authenticate users seamlessly across services. In this post, we discuss two use cases to configure trusted identity propagation with Amazon Redshift and Lake Formation.</p> 
<h2>Solution overview</h2> 
<p>Trusted identity propagation provides a new authentication option for organizations that want to centralize data permissions management and authorize requests based on their IdP identity across service boundaries. With IAM Identity Center, you can configure an existing IdP to manage users and groups and use Lake Formation to define fine-grained access control permissions on catalog resources for these IdP identities. Amazon Redshift supports identity propagation when querying data with <a href="https://docs.aws.amazon.com/redshift/latest/dg/c-getting-started-using-spectrum.html" rel="noopener" target="_blank">Amazon Redshift Spectrum</a> and with <a href="https://aws.amazon.com/redshift/features/data-sharing/" rel="noopener" target="_blank">Amazon Redshift Data Sharing</a>, and you can use <a href="http://aws.amazon.com/cloudtrail" rel="noopener" target="_blank">AWS CloudTrail</a> to audit data access by IdP identities to help your organization meet their regulatory and compliance requirements.</p> 
<p>With this new capability, users can connect to Amazon Redshift from QuickSight with a single sign-on experience and create direct query datasets. This is enabled by using IAM Identity Center as a shared identity source. With trusted identity propagation, when QuickSight assets like dashboards are shared with other users, the database permissions of each QuickSight user are applied by propagating their end-user identity from QuickSight to Amazon Redshift and enforcing their individual data permissions. Depending on the use case, the author can apply additional row-level and column-level security in QuickSight.</p> 
<p>The following diagram illustrates an example of the solution architecture.</p> 
<p><img alt="" class="alignnone wp-image-59364 size-full" height="1017" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-arch.png" width="1594" /></p> 
<p>In this post, we walk through how to configure trusted identity propagation with Amazon Redshift and Lake Formation. We cover the following use cases:</p> 
<ul> 
 <li>Redshift Spectrum with Lake Formation</li> 
 <li>Redshift data sharing with Lake Formation</li> 
</ul> 
<h2>Prerequisites</h2> 
<p>This walkthrough assumes you have set up a Lake Formation administrator role or a similar role to follow along with the instructions in this post. To learn more about setting up permissions for a data lake administrator, see <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/initial-LF-setup.html#create-data-lake-admin" rel="noopener" target="_blank">Create a data lake administrator</a>.</p> 
<p>Additionally, you must create the following resources as detailed in <a href="https://aws.amazon.com/blogs/big-data/integrate-okta-with-amazon-redshift-query-editor-v2-using-aws-iam-identity-center-for-seamless-single-sign-on/" rel="noopener" target="_blank">Integrate Okta with Amazon Redshift Query Editor V2 using AWS IAM Identity Center for seamless Single Sign-On</a>:</p> 
<ul> 
 <li>An Okta account integrated with IAM Identity Center to sync users and groups</li> 
 <li>A Redshift managed application with IAM Identity Center</li> 
 <li>A Redshift source cluster with IAM Identity Center integration enabled</li> 
 <li>A Redshift target cluster with IAM Identity Center integration enabled (you can skip the section to set up Amazon Redshift role-based access)</li> 
 <li>Users and groups from IAM Identity Center assigned to the Redshift application</li> 
 <li>A permission set assigned to AWS accounts to enable Redshift Query Editor v2 access</li> 
 <li>Add the below permission to the IAM role used in Redshift managed application for integration with IAM Identity Center. 
  <div class="hide-language"> 
   <pre><code class="lang-code">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lakeformation:GetDataAccess",
                "glue:GetTable",
                "glue:GetTables",
                "glue:SearchTables",
                "glue:GetDatabase",
                "glue:GetDatabases",
                "glue:GetPartitions",
                "lakeformation:GetResourceLFTags",
                "lakeformation:ListLFTags",
                "lakeformation:GetLFTag",
                "lakeformation:SearchTablesByLFTags",
                "lakeformation:SearchDatabasesByLFTags"
           ],
            "Resource": "*"
        }
    ]
}</code></pre> 
  </div> </li> 
</ul> 
<h2>Use case 1: Redshift Spectrum with Lake Formation</h2> 
<p>This use case assumes you have the following prerequisites:</p> 
<ul> 
 <li> 
  <ul> 
   <li>To store the data, you need an <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) bucket.</li> 
   <li>To run <a href="http://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface</a> (AWS CLI) commands, you need to set up <a href="https://aws.amazon.com/cloudshell/" rel="noopener" target="_blank">AWS CloudShell</a> in your account or the AWS CLI on your workstation. For instructions, refer to <a href="https://docs.aws.amazon.com/cloudshell/latest/userguide/getting-started.html" rel="noopener" target="_blank">Getting started with AWS CloudShell</a> or <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html" rel="noopener" target="_blank">Set up the AWS CLI</a>, respectively.</li> 
   <li>You can use an existing <a href="https://aws.amazon.com/glue" rel="noopener" target="_blank">AWS Glue</a> database and table in your account or complete the following steps logged in as an admin to set up these resources.</li> 
  </ul> </li> 
</ul> 
<ol> 
 <li>Log in to the <a href="http://aws.amazon.com/console" rel="noopener" target="_blank">AWS Management Console</a> as an IAM administrator.</li> 
 <li>Go to CloudShell or your AWS CLI and run the following AWS CLI command, providing your bucket name to copy the data:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws s3 sync s3://redshift-demos/data/NY-Pub/ s3://&lt;bucketname&gt;/data/NY-Pub/</code></pre> 
</div> 
<p>In this post, we use an <a href="https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html" rel="noopener" target="_blank">AWS Glue crawler</a> to create the external table <code>ny_pub</code> stored in Apache Parquet format in the Amazon S3 location <code>s3://&lt;bucketname&gt;/data/NY-Pub/</code>. In the next step, we create the solution resources using <a href="http://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> to create a stack named <code>CrawlS3Source-NYTaxiData</code> in <code>us-east-1</code>.</p> 
<ol start="3"> 
 <li>Download the <a href="https://redshift-demos.s3.amazonaws.com/code/glue-crawler-automate-cfn-use1.yml" rel="noopener" target="_blank">.yml file</a> or <a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=CrawlS3Source-NYTaxiData&amp;templateURL=https://redshift-demos.s3.amazonaws.com/code/glue-crawler-automate-cfn-use1.yml" rel="noopener" target="_blank">launch the CloudFormation stack</a>.</li> 
</ol> 
<p>The stack creates the following resources:</p> 
<ul> 
 <li>The crawler <code>NYTaxiCrawler</code> along with the new IAM role <code>AWSGlueServiceRole-RedshiftAutoMount</code></li> 
 <li>The AWS Glue database <code>automountdb</code></li> 
</ul> 
<p><img alt="" class="alignnone wp-image-59385 size-full" height="466" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-automountdb.png" style="margin: 10px 0px 10px 0px;" width="1252" /></p> 
<p>When the stack is complete, continue with the following steps to finish setting up your resources:</p> 
<ol start="4"> 
 <li>On the AWS Glue console, under <strong>Data Catalog</strong> in the navigation pane, choose <strong>Crawlers</strong>.</li> 
 <li>Open <code>NYTaxiCrawler</code> and choose <strong>Edit</strong>.<img alt="" class="alignnone wp-image-59384 size-full" height="373" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-NYTaxiCrawler.png" style="margin: 10px 0px 10px 0px;" width="1428" /></li> 
</ol> 
<ol start="6"> 
 <li>Under <strong>Choose data sources and classifiers</strong>, choose <strong>Edit</strong>.<br /> <img alt="" class="alignnone wp-image-59383 size-full" height="344" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-classifiers.png" style="margin: 10px 0px 10px 0px;" width="1432" /></li> 
</ol> 
<ol start="7"> 
 <li>For <strong>Data source</strong>, choose <strong>S3</strong>.</li> 
 <li>For <strong>S3 path</strong>, enter <code>s3://&lt;bucketname&gt;/data/NY-Pub/</code>.</li> 
 <li>Choose <strong>Update S3 data source</strong>.<br /> <img alt="" class="alignnone wp-image-59731 size-full" height="630" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/07/BDB-3884-datasource-1.png" style="margin: 10px 0px 10px 0px;" width="500" /></li> 
</ol> 
<ol start="10"> 
 <li>Choose <strong>Next</strong> and choose <strong>Update</strong>.</li> 
 <li>Choose <strong>Run crawler</strong>.<br /> <img alt="" class="alignleft wp-image-59381 size-full" height="684" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-Runcrawler.png" style="margin: 10px 0px 10px 0px;" width="1432" /></li> 
</ol> 
<p>After the crawler is complete, you can see a new table called <code>ny_pub</code> in the Data Catalog under the <code>automountdb</code> database.</p> 
<p>After you create the resources, complete the steps in the next sections to set up Lake Formation permissions on the AWS Glue table <code>ny_pub</code> for the <code>sales</code> IdP group and access them via Redshift Spectrum.</p> 
<h3>Enable Lake Formation propagation for the Redshift managed application</h3> 
<p>Complete the following steps to enable Lake Formation propagation for the Redshift managed application created in <a href="https://aws.amazon.com/blogs/big-data/integrate-okta-with-amazon-redshift-query-editor-v2-using-aws-iam-identity-center-for-seamless-single-sign-on/" rel="noopener" target="_blank">Integrate Okta with Amazon Redshift Query Editor V2 using AWS IAM Identity Center for seamless Single Sign-On</a>:</p> 
<ol> 
 <li>Log in to the console as admin.</li> 
 <li>On the Amazon Redshift console, choose <strong>IAM Identity Center connection</strong> in the navigation pane.</li> 
 <li>Select the managed application that starts with <code>redshift-iad</code> and choose <strong>Edit</strong>.<br /> <img alt="" class="alignnone wp-image-59380 size-full" height="382" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-redshift-iad.png" style="margin: 10px 0px 10px 0px;" width="1431" /></li> 
</ol> 
<ol start="4"> 
 <li>Select <strong>Enable AWS Lake Formation access grants</strong> under <strong>Trusted identity propagation</strong> and save your changes.<br /> <img alt="" class="alignnone wp-image-59379 size-full" height="270" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-Trusted-identity-propagation.png" width="1290" /></li> 
</ol> 
<h3>Set up Lake Formation as an IAM Identity Center application</h3> 
<p>Complete the following steps to set up Lake Formation as an IAM Identity Center application:</p> 
<ol> 
 <li>On the Lake Formation console, under <strong>Administration</strong> in the navigation pane, choose <strong>IAM Identity Center integration</strong>.<br /> <img alt="" class="alignnone wp-image-59378 size-full" height="841" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-idp-integration.png" style="margin: 10px 0px 10px 0px;" width="1429" /></li> 
</ol> 
<ol start="2"> 
 <li>Review the options and choose <strong>Submit</strong> to enable Lake Formation integration.<br /> <img alt="" class="alignnone wp-image-59377 size-full" height="888" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-enable-Lake-Formation.png" style="margin: 10px 0px 10px 0px;" width="1652" /></li> 
</ol> 
<p>The integration status will update to <strong>Success</strong>.<br /> <img alt="" class="alignnone wp-image-59376 size-full" height="668" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-Success.png" style="margin: 10px 0px 10px 0px;" width="1430" /> Alternatively, you can run the following command:</p> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">aws lakeformation create-lake-formation-identity-center-configuration 
--cli-input-json '{"CatalogId": "&lt;catalog_id&gt;","InstanceArn": "&lt;identitycenter_arn&gt;"}'
</code></pre> 
 </div> 
</div> 
<h3>Register the data with Lake Formation</h3> 
<p>In this section, we register the data with Lake Formation. Complete the following steps:</p> 
<ol> 
 <li>On the Lake Formation console, under <strong>Administration</strong> in the navigation pane, choose <strong>Data lake locations</strong>.</li> 
 <li>Choose <strong>Register location</strong>.</li> 
 <li>For <strong>Amazon S3 path</strong>, enter the bucket where the table data resides (<code>s3://&lt;bucketname&gt;/data/NY-Pub/</code>).</li> 
 <li>For <strong>IAM role</strong>, choose a Lake Formation user-defined role. For more information, refer to <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/registration-role.html" rel="noopener" target="_blank">Requirements for roles used to register locations</a>.</li> 
 <li>For <strong>Permission mode</strong>, select <strong>Lake Formation</strong>.</li> 
 <li>Choose <strong>Register location</strong>.<img alt="" class="alignnone wp-image-59375 size-full" height="1256" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-Register-location.png" style="margin: 10px 0px 10px 0px;" width="1292" /></li> 
</ol> 
<p>Next, verify that the <code>IAMAllowedPrincipal</code> group doesn’t have permission on the database.</p> 
<ol start="7"> 
 <li>On the Lake Formation console, under <strong>Data catalog</strong> in the navigation pane, choose <strong>Databases</strong>.</li> 
 <li>Select <code>automountdb</code> and on the <strong>Actions</strong> menu, choose <strong>View permissions</strong>.</li> 
 <li>If <code>IAMAllowedPrincipal</code> is listed, select the principal and choose <strong>Revoke</strong>.<img alt="" class="alignnone wp-image-59374 size-full" height="204" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-IAMAllowedPrincipal.png" style="margin: 10px 0px 10px 0px;" width="1428" /></li> 
 <li>Repeat these steps to verify permissions for the table <code>ny_pub</code>.</li> 
</ol> 
<h3>Grant the IAM Identity Center group permissions on the AWS Glue database and table</h3> 
<p>Complete the following steps to grant database permissions to the IAM Identity Center group:</p> 
<ol> 
 <li>On the Lake Formation console, under <strong>Data catalog</strong> in the navigation pane, choose <strong>Databases</strong>.</li> 
 <li>Select the database <code>automountdb</code> and on the <strong>Actions</strong> menu, choose <strong>Grant</strong>.</li> 
 <li>Choose <strong>Grant database</strong>.</li> 
 <li>Under Principals, select <strong>IAM Identity Center</strong> and choose <strong>Add</strong>.<br /> <img alt="" class="alignnone wp-image-59730 size-full" height="381" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/07/BDB-3884-IAM-Identity-Center-Add-resize.png" style="margin: 10px 0px 10px 0px;" width="500" /></li> 
 <li>In the pop-up window, if this is the first time assigning users and groups, choose <strong>Get started</strong>.<img alt="" class="alignnone wp-image-59733 size-full" height="502" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/07/BDB-3884-Getstarted-resized.png" width="500" /></li> 
 <li>Enter the IAM Identity Center group in the search bar and choose the group.</li> 
 <li>Choose <strong>Assign</strong>.<br /> <img alt="" class="alignnone wp-image-59732 size-full" height="500" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/07/BDB-3884-Assign-resized.png" width="500" /></li> 
 <li>Under <strong>LF-Tags or catalog resources</strong>, <code>automountdb</code> is already selected for <strong>Databases</strong>.</li> 
 <li>Select <strong>Describe</strong> for <strong>Database permissions</strong>.</li> 
 <li>Choose <strong>Grant</strong> to apply the permissions.<img alt="" class="alignnone wp-image-59370 size-full" height="1372" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-LFTag.png" width="1068" /></li> 
</ol> 
<p>Alternatively, you can run the following command:</p> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">aws lakeformation grant-permissions --cli-input-json '
{
    "Principal": {
        "DataLakePrincipalIdentifier": "arn:aws:identitystore:::group/&lt;identitycenter_group_name&gt;"
    },
    "Resource": {
        "Database": {
            "Name": "automountdb"
        }
    },
    "Permissions": [
        "DESCRIBE"
    ]
}'</code></pre> 
 </div> 
</div> 
<p>Next, you grant table permissions to the IAM Identity Center group.</p> 
<ol start="11"> 
 <li>Under <strong>Data catalog</strong> in the navigation pane, choose <strong>Databases</strong>.</li> 
 <li>Select the database <code>automountdb</code> and on the <strong>Actions</strong> menu, choose <strong>Grant</strong>.</li> 
 <li>Under <strong>Principals</strong>, select <strong>IAM Identity Center</strong> and choose <strong>Add</strong>.</li> 
 <li>Enter the IAM Identity Center group in the search bar and choose the group.</li> 
 <li>Choose <strong>Assign</strong>.</li> 
 <li>Under <strong>LF-Tags or catalog resources</strong>, <code>automountdb</code> is already selected for <strong>Databases</strong>.</li> 
 <li>For <strong>Tables</strong>, choose <code>ny_pub</code>. <img alt="" class="alignnone wp-image-59369 size-full" height="938" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-LFTag-ny_pub.png" width="1082" /></li> 
 <li>Select <strong>Describe</strong> and <strong>Select</strong> for <strong>Table permissions</strong>.</li> 
 <li>Choose <strong>Grant</strong> to apply the permissions. <img alt="" class="alignnone wp-image-59368 size-full" height="888" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-Table-permissions.png" style="margin: 10px 0px 10px 0px;" width="1216" /></li> 
 <li> 
  <ol> 
   <li> 
    <ol> 
     <li></li> 
    </ol> </li> 
  </ol> </li> 
</ol> 
<ol start="11"> 
 <li></li> 
</ol> 
<p>Alternatively, you can run the following command:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws lakeformation grant-permissions --cli-input-json '
{
    "Principal": {
        "DataLakePrincipalIdentifier": "arn:aws:identitystore:::group/&lt;identitycenter_group_name&gt;"
    },
    "Resource": {
        "Table": {
            "DatabaseName": "automountdb",
            "Name": "ny_pub "
        }
    },
    "Permissions": [
        "SELECT",
        "DESCRIBE"

    ]
}'
</code></pre> 
</div> 
<h3>Set up Redshift Spectrum table access for the IAM Identity Center group</h3> 
<p>Complete the following steps to set up Redshift Spectrum table access:</p> 
<ol> 
 <li>Sign in to the Amazon Redshift console using the admin role.</li> 
 <li>Navigate to Query Editor v2.</li> 
 <li>Choose the options menu (three dots) next to the cluster and choose <strong>Create connection</strong>.</li> 
 <li>Connect as the admin user and run the following commands to make the <code>ny_pub</code> data in the S3 data lake available to the sales group: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">create external schema if not exists nyc_external_schema from DATA CATALOG database 'automountdb' catalog_id '&lt;accountid&gt;'; 
grant usage on schema nyc_external_schema to role "awsidc:awssso-sales"; 
grant select on all tables in schema nyc_external_schema to role "awsidc:awssso- sales";</code></pre> 
  </div> </li> 
</ol> 
<h3>Validate Redshift Spectrum access as an IAM Identity Center user</h3> 
<p>Complete the following steps to validate access:</p> 
<ol> 
 <li>On the Amazon Redshift console, navigate to Query Editor v2.</li> 
 <li>Choose the options menu (three dots) next to the cluster and choose <strong>Create connection</strong></li> 
 <li>Choose select IAM Identity Center option for Connect option. Provide Okta user name and password in the browser pop-up.</li> 
 <li>Once connected as a federated user, run the following SQL commands to query the ny_pub data lake table:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">select * from nyc_external_schema.ny_pub;</code></pre> 
</div> 
<p><img alt="" class="alignnone wp-image-59367 size-full" height="717" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/BDB-3884-SQL-ny_pub.png" style="margin: 10px 0px 10px 0px;" width="1431" /></p> 
<h2>Use Case 2: Redshift data sharing with Lake Formation</h2> 
<p>This use case assumes you have IAM Identity Center integration with Amazon Redshift set up, with Lake Formation propagation enabled as per the instructions provided in the previous section.</p> 
<h3>Create a data share with objects and share it with the Data Catalog</h3> 
<p>Complete the following steps to create a data share:</p> 
<ol> 
 <li>Sign in to the Amazon Redshift console using the admin role.</li> 
 <li>Navigate to Query Editor v2.</li> 
 <li>Choose the options menu (three dots) next to the Redshift source cluster and choose <strong>Create connection</strong>.</li> 
 <li>Connect as admin user using Temporarily credentials using a database user name option and run the following SQL commands to create a data share: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE DATASHARE salesds; 
ALTER DATASHARE salesds ADD SCHEMA sales_schema; 
ALTER DATASHARE salesds ADD TABLE store_sales; 
GRANT USAGE ON DATASHARE salesds TO ACCOUNT ‘&lt;accountid&gt;’ via DATA CATALOG;</code></pre> 
  </div> </li> 
 <li>Authorize the data share by choosing <strong>Data shares</strong> in the navigation page and selecting the data share <code>salesdb</code>.</li> 
 <li>Select the data share and choose <strong>Authorize</strong>.</li> 
</ol> 
<p>Now you can register the data share in Lake Formation as an AWS Glue database.</p> 
<ol start="7"> 
 <li>Sign in to the Lake Formation console as the data lake administrator IAM user or role.</li> 
 <li>Under <strong>Data catalog</strong> in the navigation pane, choose <strong>Data sharing</strong> and view the Redshift data share invitations on the <strong>Configuration</strong> tab.</li> 
 <li>Select the datashare salesds and choose <strong>Review Invitation</strong>.</li> 
 <li>Once you review the details choose <strong>Accept</strong>.</li> 
 <li>Provide a name for the AWS Glue database (for example, salesds) and choose <strong>Skip to Review and create</strong>.</li> 
</ol> 
<p>After the AWS Glue database is created on the Redshift data share, you can view it under <strong>Shared databases</strong>.</p> 
<h3>Grant the IAM Identity Center user group permission on the AWS Glue database and table</h3> 
<p>Complete the following steps to grant database permissions to the IAM Identity Center group:</p> 
<ol> 
 <li>On the Lake Formation console, under <strong>Data catalog</strong> in the navigation pane, choose <strong>Databases</strong>.</li> 
 <li>Select the database salesds and on the <strong>Actions</strong> menu, choose <strong>Grant</strong>.</li> 
 <li>Choose <strong>Grant database</strong>.</li> 
 <li>Under Principals, select <strong>IAM Identity Center</strong> and choose <strong>Add</strong>.</li> 
 <li>In the pop-up window, enter the IAM Identity Center group awssso in the search bar and choose the awssso-sales group.</li> 
 <li>Choose <strong>Assign</strong>.</li> 
 <li>Under <strong>LF-Tags or catalog resources</strong>, salesds is already selected for <strong>Databases</strong>.</li> 
 <li>Select <strong>Describe</strong> for <strong>Database permissions</strong>.</li> 
 <li>Choose <strong>Grant</strong> to apply the permissions.</li> 
</ol> 
<p>Next, grant table permissions to the IAM Identity Center group.</p> 
<ol start="10"> 
 <li>Under <strong>Data catalog</strong> in the navigation pane, choose <strong>Databases</strong>.</li> 
 <li>Select the database salesds and on the <strong>Actions</strong> menu, choose <strong>Grant</strong>.</li> 
 <li>Under <strong>Principals</strong>, select <strong>IAM Identity Center</strong> and choose <strong>Add</strong>.</li> 
 <li>In the pop-up window, enter the IAM Identity Center group awssso in the search bar and choose the awssso-sales group.</li> 
 <li>Choose <strong>Assign</strong>.</li> 
 <li>Under <strong>LF-Tags or catalog resources</strong>, salesds is already selected for <strong>Databases</strong>.</li> 
 <li>For <strong>Tables</strong>, choose sales_schema.store_sales.</li> 
 <li>Select <strong>Describe</strong> and <strong>Select</strong> for <strong>Table permissions</strong>.</li> 
 <li>Choose <strong>Grant</strong> to apply the permissions.</li> 
</ol> 
<h3>Mount the external schema in the target Redshift cluster and enable access for the IAM Identity Center user</h3> 
<p>Complete the following steps:</p> 
<ol> 
 <li>Sign in to the Amazon Redshift console using the admin role.</li> 
 <li>Navigate to Query Editor v2.</li> 
 <li>Connect as an admin user and run the following SQL commands to mount the AWS Glue database customerds as an external schema and enable access to the sales group:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">create external schema if not exists sales_datashare_schema from DATA CATALOG database salesds catalog_id '&lt;accountid&gt;';
create role "awsidc:awssso-sales"; # If the role was not already created 
grant usage on schema sales_datashare_schema to role "awsidc:awssso-sales";
grant select on all tables in schema sales_datashare_schema to role "awsidc:awssso- sales";
</code></pre> 
</div> 
<h3>Access Redshift data shares as an IAM Identity Center user</h3> 
<p>Complete the following steps to access the data shares:</p> 
<ol> 
 <li>On the Amazon Redshift console, navigate to Query Editor v2.</li> 
 <li>Choose the options menu (three dots) next to the cluster and choose <strong>Create connection</strong>.</li> 
 <li>Connect with IAM Identity Center and the provide IAM Identity Center user and password in the browser login.</li> 
 <li>Run the following SQL commands to query the data lake table:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT *&nbsp;FROM&nbsp;"dev"."sales_datashare_schema"."sales_schema.store_sales";</code></pre> 
</div> 
<p>With Transitive Identity Propagation we can now audit user access to dataset from Lake Formation dashboard and service used for accessing the dataset providing complete trackability. For federated user Ethan whose Identity Center User ID is ‘459e10f6-a3d0-47ae-bc8d-a66f8b054014’ you can see the below event log.</p> 
<p><img alt="" class="alignnone wp-image-59481 size-full" height="393" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/BDB-3884-Event_Log.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<div class="hide-language"> 
 <pre><code class="lang-json">"eventSource": "lakeformation.amazonaws.com",
    "eventName": "GetDataAccess",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "redshift.amazonaws.com",
    "userAgent": "redshift.amazonaws.com",
    "requestParameters": {
        "tableArn": "arn:aws:glue:us-east-1:xxxx:table/automountdb/ny_pub",
        "durationSeconds": 3600,
        "auditContext": {
            "additionalAuditContext": "{\"invokedBy\":\"arn:aws:redshift:us-east-1:xxxx:dbuser:redshift-consumer/awsidc:ethan.doe@gmail.com\", \"transactionId\":\"961953\", \"queryId\":\"613842\", \"isConcurrencyScalingQuery\":\"false\"}"
        },
        "cellLevelSecurityEnforced": true
    },
    "responseElements": null,
    "additionalEventData": {
        "requesterService": "REDSHIFT",
        "LakeFormationTrustedCallerInvocation": "true",
        "lakeFormationPrincipal": "arn:aws:identitystore:::user/459e10f6-a3d0-47ae-bc8d-a66f8b054014",
        "lakeFormationRoleSessionName": "AWSLF-00-RE-726034267621-K7FUMxovuq"
    }
</code></pre> 
</div> 
<h2>Clean up</h2> 
<p>Complete the following steps to clean up your resources:</p> 
<ol> 
 <li>Delete the data from the S3 bucket.</li> 
 <li>Delete the Lake Formation application and the Redshift provisioned cluster that you created for testing.</li> 
 <li>Sign in to the CloudFormation console as the IAM admin used for creating the CloudFormation stack, and delete the stack you created.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this post, we covered how to simplify access management for analytics by propagating user identity across Amazon Redshift and Lake Formation using IAM Identity Center. We learned how to get started with trusted identity propagation by connecting to Amazon Redshift and Lake Formation. We also learned how to configure Redshift Spectrum and data sharing to support trusted identity propagation.</p> 
<p>Learn more about IAM Identity Center with <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/redshift-iam-access-control-idp-connect.html" rel="noopener" target="_blank">Amazon Redshift</a> and <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/identity-center-integration.html" rel="noopener" target="_blank">AWS Lake Formation</a>.&nbsp;Leave your questions and feedback in the comments section.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-59366 alignleft" height="130" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/Harshida.png" width="100" />Harshida Patel</strong>&nbsp;is a Analytics Specialist Principal Solutions Architect, with AWS.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-59365 alignleft" height="131" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/05/Srividya.png" width="100" />Srividya Parthasarathy</strong> is a Senior Big Data Architect on the AWS Lake Formation team. She enjoys building data mesh solutions and sharing them with the community.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-59484 alignleft" height="115" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/06/maneesh.jpg" width="100" />Maneesh Sharma</strong>&nbsp;is a Senior Database Engineer at AWS with more than a decade of experience designing and implementing large-scale data warehouse and analytics solutions. He collaborates with various Amazon Redshift Partners and customers to drive better integration.</p> 
<p style="clear: both;"><strong><img alt="" class="wp-image-33725 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/08/28/Poulomi-Dasgupta.png" width="100" /></strong><strong>Poulomi Dasgupta</strong>&nbsp;is a Senior Analytics Solutions Architect with AWS. She is passionate about helping customers build cloud-based analytics solutions to solve their business problems. Outside of work, she likes travelling and spending time with her family.</p>
