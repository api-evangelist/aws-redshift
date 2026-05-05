---
title: "Five actionable steps to GDPR compliance (Right to be forgotten) with Amazon Redshift"
url: "https://aws.amazon.com/blogs/big-data/five-actionable-steps-to-gdpr-compliance-right-to-be-forgotten-with-amazon-redshift/"
date: "Fri, 28 Jul 2023 17:35:48 +0000"
author: "Yadukishore Tatavarthi"
feed_url: "https://aws.amazon.com/blogs/big-data/category/database/amazon-redshift/feed/"
---
<p>The GDPR (General Data Protection Regulation) right to be forgotten, also known as the right to erasure, gives individuals the right to request the deletion of their personally identifiable information (PII) data held by organizations. This means that individuals can ask companies to erase their personal data from their systems and any third parties with whom the data was shared. Organizations must comply with these requests provided that there are no legitimate grounds for retaining the personal data, such as legal obligations or contractual requirements.</p> 
<p><a href="https://aws.amazon.com/redshift/" rel="noopener" target="_blank">Amazon Redshift</a> is a fully managed, petabyte-scale data warehouse service in the cloud. It is designed for analyzing large volumes of data and performing complex queries on structured and semi-structured data. Many customers are looking for best practices to keep their Amazon Redshift analytics environment compliant and have an ability to respond to GDPR right to forgotten requests.</p> 
<p>In this post, we discuss challenges associated with implementation and architectural patterns and actionable best practices for organizations to respond to the right to be forgotten request requirements of the GDPR for data stored in Amazon Redshift.</p> 
<h2><strong>Who does GDPR apply to?</strong></h2> 
<p>The GDPR applies to all organizations established in the EU and to organizations, whether or not established in the EU, that process the personal data of EU individuals in connection with either the offering of goods or services to data subjects in the EU or the monitoring of behavior that takes place within the EU.</p> 
<p>The following are key terms we use when discussing the GDPR:</p> 
<ul> 
 <li><strong>Data subject</strong> – An identifiable living person and resident in the EU or UK, on whom personal data is held by a business or organization or service provider</li> 
 <li><strong>Processor</strong> – The entity that processes the data on the instructions of the controller (for example, AWS)</li> 
 <li><strong>Controller</strong> – The entity that determines the purposes and means of processing personal data (for example, an AWS customer)</li> 
 <li><strong>Personal data</strong> – Information relating to an identified or identifiable person, including names, email addresses, and phone numbers</li> 
</ul> 
<p>Implementing the right to be forgotten can include the following challenges:</p> 
<ul> 
 <li><strong>Data identification</strong> – One of the main challenges is identifying all instances of personal data across various systems, databases, and backups. Organizations need to have a clear understanding of where personal data is being stored and how it is processed to effectively fulfill the deletion requests.</li> 
 <li><strong>Data dependencies</strong> – Personal data can be interconnected and intertwined with other data systems, making it challenging to remove specific data without impacting the integrity of functionality of other systems or processes. It requires careful analysis to identify data dependencies and mitigate any potential risks or disruptions.</li> 
 <li><strong>Data replication and backups</strong> – Personal data can exist in multiple copies due to data replication and backups. Ensuring the complete removal of data from all these copies and backups can be challenging. Organizations need to establish processes to track and manage data copies effectively.</li> 
 <li><strong>Legal obligations and exemptions</strong> – The right to be forgotten is not absolute and may be subject to legal obligations or exemptions. Organizations need to carefully assess requests, considering factors such as legal requirements, legitimate interests, freedom of expression, or public interest to determine if the request can be fulfilled or if any exceptions apply.</li> 
 <li><strong>Data archiving and retention</strong> – Organizations may have legal or regulatory requirements to retain certain data for a specific period. Balancing the right to be forgotten with the obligation to retain data can be a challenge. Clear policies and procedures need to be established to manage data retention and deletion appropriately.</li> 
</ul> 
<h2>Architecture patterns</h2> 
<p>Organizations are generally required to respond to right to be forgotten requests within 30 days from when the individual submits a request. This deadline can be extended by a maximum of 2 months taking into account the complexity and the number of the requests, provided that the data subject has been informed about the reasons for the delay within 1 month of the receipt of the request.</p> 
<p>The following sections discuss a few commonly referenced architecture patterns, best practices, and options supported by Amazon Redshift to support your data subject’s GDPR right to be forgotten request in your organization.</p> 
<p><img alt="Actionable Steps" class="aligncenter wp-image-50814 size-full" height="348" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/07/17/BDB-3185-overview1.jpg" style="margin: 10px 0px 10px 0px;" width="818" /></p> 
<h2>Data management and governance</h2> 
<p>Addressing the challenges mentioned requires a combination of technical, operational, and legal measures. Organizations need to develop robust data governance practices, establish clear procedures for handling deletion requests, and maintain ongoing compliance with GDPR regulations.</p> 
<p>Large organizations usually have multiple Redshift environments, databases, and tables spread across multiple Regions and accounts. To successfully respond to a data subject’s requests, organizations should have a clear strategy to determine how data is forgotten, flagged, anonymized, or deleted, and they should have clear guidelines in place for data audits.</p> 
<p><em>Data mapping</em> involves identifying and documenting the flow of personal data in an organization. It helps organizations understand how personal data moves through their systems, where it is stored, and how it is processed. By creating visual representations of data flows, organizations can gain a clear understanding of the lifecycle of personal data and identify potential vulnerabilities or compliance gaps.</p> 
<p>Note that putting a comprehensive data strategy in place is not in scope for this post.</p> 
<h2>Audit tracking</h2> 
<p>Organizations must maintain proper documentation and audit trails of the deletion process to demonstrate compliance with GDPR requirements. A typical audit control framework should record the data subject requests (who is the data subject, when was it requested, what data, approver, due date, scheduled ETL process if any, and so on). This will help with your audit requests and provide the ability to roll back in case of accidental deletions observed during the QA process. It’s important to maintain the list of users and systems who may get impacted during this process to ensure effective communication.</p> 
<h2>Data discovery and findability</h2> 
<p>Findability is an important step of the process. Organizations need to have mechanisms to find the data under consideration in an efficient and quick manner for timely response. The following are some patterns and best practices you can employ to find the data in Amazon Redshift.</p> 
<h3>Tagging</h3> 
<p>Consider tagging your Amazon Redshift resources to quickly identify which clusters and snapshots contain the PII data, the owners, the data retention policy, and so on. Tags provide metadata about resources at a glance. Redshift resources, such as namespaces, workgroups, snapshots, and clusters can be tagged. For more information about tagging, refer to <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/amazon-redshift-tagging.html" rel="noopener" target="_blank">Tagging resources in Amazon Redshift</a>.</p> 
<h3>Naming conventions</h3> 
<p>As a part of the modeling strategy, name the database objects (databases, schemas, tables, columns) with an indicator that they contain PII so that they can be queried using system tables (for example, make a list of the tables and columns where PII data is involved). Identifying the list of tables and users or the systems that have access to them will help streamline the communication process. The following sample SQL can help you find the databases, schemas, and tables with a name that contains PII:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT
pg_catalog.pg_namespace.nspname AS schema_name,
pg_catalog.pg_class.relname AS table_name,
pg_catalog.pg_attribute.attname AS column_name,
pg_catalog.pg_database.datname AS database_name
FROM
pg_catalog.pg_namespace
JOIN pg_catalog.pg_class ON pg_catalog.pg_namespace.oid = pg_catalog.pg_class.relnamespace
JOIN pg_catalog.pg_attribute ON pg_catalog.pg_class.oid = pg_catalog.pg_attribute.attrelid
JOIN pg_catalog.pg_database ON pg_catalog.pg_attribute.attnum &gt; 0
WHERE
pg_catalog.pg_attribute.attname LIKE '%PII%';

SELECT datname
FROM pg_database
WHERE datname LIKE '%PII%';

SELECT table_schema, table_name, column_name
FROM information_schema.columns
WHERE column_name LIKE '%PII%'</code></pre> 
</div> 
<h3>Separate PII and non-PII</h3> 
<p>Whenever possible, keep the sensitive data in a separate table, database, or schema. Isolating the data in a separate database may not always be possible. However, you can separate the non-PII columns in a separate table, for example, <code>Customer_NonPII</code> and <code>Customer_PII</code>, and then join them with an unintelligent key. This helps identify the tables that contain non-PII columns. This approach is straightforward to implement and keeps non-PII data intact, which can be useful for analysis purposes. The following figure shows an example of these tables.</p> 
<p><img alt="PII-Non PII Example Tables" class="alignnone wp-image-50811 size-full" height="131" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/07/17/BDB-3185-PII-image2.png" style="margin: 10px 0px 10px 0px;" width="782" /></p> 
<h3>Flag columns</h3> 
<p>In the preceding tables, rows in bold are marked with <code>Forgotten_flag=Yes</code>. You can maintain a <code>Forgotten_flag</code> as a column with the default value as <code>No</code> and update this value to <code>Yes</code> whenever a request to be forgotten is received. Also, as a best practice from HIPAA, do a batch deletion once in a month. The downstream and upstream systems need to respect this flag and include this in their processing. This helps identify the rows that need to be deleted. For our example, we can use the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">Delete from Customer_PII where forgotten_flag=“Yes”</code></pre> 
</div> 
<h3>Use Master data management system</h3> 
<p>Organizations that maintain a master data management system maintain a golden record for a customer, which acts as a single version of truth from multiple disparate systems. These systems also contain crosswalks with several peripheral systems that contain the natural key of the customer and golden record. This technique helps find customer records and related tables. The following is a representative example of a crosswalk table in a master data management system.</p> 
<p><img alt="Example of a MDM Records" class="aligncenter wp-image-50813 size-full" height="190" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/07/17/BDB-3185-MDM-Image3.jpg" style="margin: 10px 0px 10px 0px;" width="1038" /></p> 
<h3>Use AWS Lake Formation</h3> 
<p>Some organizations have use cases where you can share the data across multiple departments and business units and use Amazon Redshift data sharing. We can use <a href="https://aws.amazon.com/lake-formation/" rel="noopener" target="_blank">AWS Lake Formation</a> tags to tag the database objects and columns and define fine-grained access controls on who can have the access to use data. Organizations can have a dedicated resource with access to all tagged resources. With Lake Formation, you can centrally define and enforce database-, table-, column-, and row-level access permissions of Redshift data shares and restrict user access to objects within a data share.</p> 
<p>By sharing data through Lake Formation, you can define permissions in Lake Formation and apply those permissions to data shares and their objects. For example, if you have a table containing employee information, you can use column-level filters to help prevent employees who don’t work in the HR department from seeing sensitive information. Refer to <a href="https://docs.aws.amazon.com/redshift/latest/dg/lake-formation-datashare.html" rel="noopener" target="_blank">AWS Lake Formation-managed Redshift shares</a> for more details on the implementation.</p> 
<h3>Use Amazon DataZone</h3> 
<p><a href="https://aws.amazon.com/datazone/" rel="noopener" target="_blank">Amazon DataZone</a> introduces a business metadata catalog. Business metadata provides information authored or used by businesses and gives context to organizational data. Data discovery is a key task that business metadata can support. Data discovery uses centrally defined corporate ontologies and taxonomies to classify data sources and allows you to find relevant data objects. You can add business metadata in Amazon DataZone to support data discovery.</p> 
<h2>Data erasure</h2> 
<p>By using the approaches we’ve discussed, you can find the clusters, databases, tables, columns, snapshots that contain the data to be deleted. The following are some methods and best practices for data erasure.</p> 
<h3>Restricted backup</h3> 
<p>In some use cases, you may have to keep data backed up to align with government regulations for a certain period of time. It’s a good idea to take the backup of the data objects before deletion and keep it for an agreed-upon retention time. You can use <a href="https://aws.amazon.com/backup/" rel="noopener" target="_blank">AWS Backup</a> to take automatic or manual backups. AWS Backup allows you to define a central backup policy to manage the data protection of your applications. For more information, refer to <a href="https://aws.amazon.com/blogs/aws/new-amazon-redshift-support-in-aws-backup/" rel="noopener" target="_blank">New – Amazon Redshift Support in AWS Backup</a>.</p> 
<h3>Physical deletes</h3> 
<p>After we find the tables that contain the data, we can delete the data using the following code (using the flagging technique discussed earlier):</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">Delete from Customer_PII where forgotten_flag=“Yes”</code></pre> 
</div> 
<p>It’s a good practice to delete data at a specified schedule, such as once every 25–30 days, so that it is simpler to maintain the state of the database.</p> 
<h3>Logical deletes</h3> 
<p>You may need to keep data in a separate environment for audit purposes. You can employ Amazon Redshift row access policies and conditional dynamic masking policies to filter and anonymize the data.</p> 
<p>You can use row access policies on <code>Forgotten_flag=No</code> on the tables that contain PII data so that the designated users can only see the necessary data. Refer to <a href="https://aws.amazon.com/blogs/big-data/achieve-fine-grained-data-security-with-row-level-access-control-in-amazon-redshift/" rel="noopener" target="_blank">Achieve fine-grained data security with row-level access control in Amazon Redshift</a> for more information about how to implement row access policies.</p> 
<p>You can use <a href="https://docs.aws.amazon.com/redshift/latest/dg/t_ddm-conditional.html" rel="noopener" target="_blank">conditional dynamic data masking policies</a> so that designated users can see the redacted data. With <a href="https://docs.aws.amazon.com/redshift/latest/dg/t_ddm.html" rel="noopener" target="_blank">dynamic data masking</a> (DDM) in Amazon Redshift, organizations can help protect sensitive data in your data warehouse. You can manipulate how Amazon Redshift shows sensitive data to the user at query time without transforming it in the database. You control access to data through masking policies that apply custom obfuscation rules to a given user or role. That way, you can respond to changing privacy requirements without altering the underlying data or editing SQL queries.</p> 
<p>Dynamic data masking policies hide, obfuscate, or pseudonymize data that matches a given format. When attached to a table, the masking expression is applied to one or more of its columns. You can further modify masking policies to only apply them to certain users or user-defined roles that you can create with <a href="https://docs.aws.amazon.com/redshift/latest/dg/t_Roles.html" rel="noopener" target="_blank">role-based access control</a> (RBAC). Additionally, you can apply DDM on the cell level by using conditional columns when creating your masking policy.</p> 
<p>Organizations can use conditional dynamic data masking to redact sensitive columns (for example, names) where the forgotten flag column value is TRUE, and the other columns display the full values.</p> 
<h3>Backup and restore</h3> 
<p>Data from Redshift clusters can be transferred, exported, or copied to different AWS services or outside of the cloud. Organizations should have an effective governance process to detect and remove data to align with the GDPR compliance requirement. However, this is beyond the scope of this post.</p> 
<p>Amazon Redshift offers backups and snapshots of the data. After deleting the PII data, organizations should also purge the data from their backups. To do so, you need to <a href="https://docs.aws.amazon.com/aws-backup/latest/devguide/redshift-restores.html" rel="noopener" target="_blank">restore the snapshot</a> to a new cluster, remove the data, and take a fresh backup. The following figure illustrates this workflow.</p> 
<p>It’s good practice to keep the retention period at 29 days (if applicable) so that the backups are cleared after 30 days. Organizations can also set the backup schedule to a certain date (for example, the first of every month).</p> 
<p><img alt="Backup and Restore" class="aligncenter wp-image-50812 size-full" height="104" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/07/17/BDB-3185-Backup-image4.jpg" style="margin: 10px 0px 10px 0px;" width="732" /></p> 
<h3>Communication</h3> 
<p>It’s important to communicate to the users and processes who may be impacted by this deletion. The following query helps identify the list of users and groups who have access to the affected tables:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT
nspname AS schema_name,
relname AS table_name,
attname AS column_name,
usename AS user_name,
groname AS group_name
FROM pg_namespace
JOIN pg_class ON pg_namespace.oid = pg_class.relnamespace
JOIN pg_attribute ON pg_class.oid = pg_attribute.attrelid
LEFT JOIN pg_group ON pg_attribute.attacl::text LIKE '%' || groname || '%'
LEFT JOIN pg_user ON pg_attribute.attacl::text LIKE '%' || usename || '%'
WHERE
pg_attribute.attname LIKE '%PII%'
AND (usename IS NOT NULL OR groname IS NOT NULL);</code></pre> 
</div> 
<h2>Security controls</h2> 
<p>Maintaining security is of great importance in GDPR compliance. By implementing robust security measures, organizations can help protect personal data from unauthorized access, breaches, and misuse, thereby helping maintain the privacy rights of individuals. Security plays a crucial role in upholding the principles of confidentiality, integrity, and availability of personal data. AWS offers a comprehensive suite of services and features that can support GDPR compliance and enhance security measures.</p> 
<p>The GDPR does not change the <a href="https://aws.amazon.com/compliance/shared-responsibility-model/" rel="noopener" target="_blank">AWS shared responsibility model</a>, which continues to be relevant for customers. The shared responsibility model is a useful approach to illustrate the different responsibilities of AWS (as a data processor or subprocessor) and customers (as either data controllers or data processors) under the GDPR.</p> 
<p>Under the shared responsibility model, AWS is responsible for securing the underlying infrastructure that supports AWS services (“Security of the Cloud”), and customers, acting either as data controllers or data processors, are responsible for personal data they upload to AWS services (“Security in the Cloud”).</p> 
<p>AWS offers a GDPR-compliant AWS Data Processing Addendum (AWS DPA), which enables you to comply with GDPR contractual obligations. The <a href="http://aws.amazon.com/blogs/security/aws-gdpr-data-processing-addendum/" rel="noopener" target="_blank">AWS DPA is incorporated into the AWS Service Terms</a>.</p> 
<p>Article 32 of the GDPR requires that organizations must “…implement appropriate technical and organizational measures to ensure a level of security appropriate to the risk, including …the pseudonymization and encryption of personal data[…].” In addition, organizations must “safeguard against the unauthorized disclosure of or access to personal data.” Refer to the <a href="https://docs.aws.amazon.com/whitepapers/latest/navigating-gdpr-compliance/welcome.html" rel="noopener" target="_blank">Navigating GDPR Compliance on AWS</a> whitepaper for more details.</p> 
<h2>Conclusion</h2> 
<p>In this post, we delved into the significance of GDPR and its impact on safeguarding privacy rights. We discussed five commonly followed best practices that organizations can reference for responding to GDPR right to be forgotten requests for data that resides in Redshift clusters. We also highlighted that the GDPR does not change the AWS shared responsibility model.</p> 
<p>We encourage you to take charge of your data privacy today. Prioritizing GPDR compliance and data privacy will not only strengthen trust, but also build customer loyalty and safeguard personal information in digital era. If you need assistance or guidance, reach out to an AWS representative. AWS has teams of Enterprise Support Representatives, Professional Services Consultants, and other staff to help with GDPR questions. You can <a href="https://aws.amazon.com/contact-us/compliance-support/" rel="noopener" target="_blank">contact us</a> with questions. To learn more about GDPR compliance when using AWS services, refer to the <a href="https://aws.amazon.com/compliance/gdpr-center/" rel="noopener" target="_blank">General Data Protection Regulation (GDPR) Center</a>. To learn more about the right to be forgotten, refer to <a href="https://edpb.europa.eu/decision-nr-92_en" rel="noopener" target="_blank">Right to Erasure</a>.</p> 
<p>Disclaimer: The information provided above is not a legal advice. It is intended to showcase commonly followed best practices. It is crucial to consult with your organization’s privacy officer or legal counsel and determine appropriate solutions.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><img alt="YaduKishore Profile" class="size-full wp-image-50820 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/07/17/KishorePicture.jpeg" width="100" /><strong>Yadukishore Tatavarthi </strong>&nbsp;is a Senior Partner Solutions Architect supporting Healthcare and life science customers at Amazon Web Services. He has been helping the customers over the last 20 years in building the enterprise data strategies, advising customers on cloud implementations, migrations, reference architecture creation, data modeling best practices, data lake/warehouses architecture, and other technical processes.</p> 
<p style="clear: both;"><strong><img alt="Sudhir Gupta" class="size-full wp-image-50821 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/07/17/Screenshot-2023-07-17-at-7.20.18-AM.png" width="100" />Sudhir&nbsp;Gupta&nbsp;</strong>is a Principal Partner Solutions Architect, Analytics Specialist at AWS with over 18 years of experience in Databases and Analytics. He helps AWS partners and customers design, implement, and migrate large-scale data &amp; analytics (D&amp;A) workloads. As a trusted advisor to partners, he enables partners globally on AWS D&amp;A services, builds solutions/accelerators, and leads go-to-market initiatives</p> 
<p style="clear: both;"><img alt="Deepak Singh" class="size-full wp-image-50822 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/07/17/Screenshot-2023-07-17-at-7.20.37-AM.png" width="100" /><strong>Deepak Singh</strong> is a Senior Solutions Architect at Amazon Web Services with 20+ years of experience in Data &amp; AIA. He enjoys working with AWS partners and customers on building scalable analytical solutions for their business outcomes. When not at work, he loves spending time with family or exploring new technologies in analytics and AI space.</p>
