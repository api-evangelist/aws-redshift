---
title: "Accelerate SQL code migration from Google BigQuery to Amazon Redshift using BladeBridge"
url: "https://aws.amazon.com/blogs/big-data/accelerate-sql-code-migration-from-google-bigquery-to-amazon-redshift-using-bladebridge/"
date: "Thu, 07 Nov 2024 16:32:17 +0000"
author: "Ritesh Sinha"
feed_url: "https://aws.amazon.com/blogs/big-data/category/database/amazon-redshift/feed/"
---
<p>Accelerating SQL code migration from Google BigQuery to <a href="https://aws.amazon.com/redshift/" rel="noopener" target="_blank">Amazon Redshift</a> can be a complex and time-consuming task. Businesses often struggle to efficiently translate their existing BigQuery code to Amazon Redshift, which can delay critical data modernization initiatives. However, with the right tools and approach, this migration process can be significantly streamlined.</p> 
<p>This post explores how you can use <a href="https://bladebridge.com/" rel="noopener" target="_blank">BladeBridge</a>, a leading data environment modernization solution, to simplify and accelerate the migration of SQL code from BigQuery to Amazon Redshift. BladeBridge offers a comprehensive suite of tools that automate much of the complex conversion work, allowing organizations to quickly and reliably transition their data analytics capabilities to the scalable Amazon Redshift data warehouse. BladeBridge provides a configurable framework to seamlessly convert legacy metadata and code into more modern services such as Amazon Redshift.</p> 
<p>Amazon Redshift is a fully managed data warehouse service offered by <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services</a> (AWS). Tens of thousands of customers use Amazon Redshift every day to run analytics, processing exabytes of data for business insights. Whether your growing data is stored in operational data stores, data lakes, streaming data services, or third-party datasets, Amazon Redshift helps you securely access, combine, and share data with minimal movement or copying. Amazon Redshift is built for scale and <a href="https://aws.amazon.com/blogs/big-data/amazon-redshift-lower-price-higher-performance/" rel="noopener" target="_blank">delivers up to 7.9 times better price performance</a> than other cloud data warehouses.</p> 
<p>By using the <a href="https://bladebridge.com/analyzer/" rel="noopener" target="_blank">BladeBridge Analyzer</a> and <a href="https://bladebridge.com/converter/" rel="noopener" target="_blank">BladeBridge Converter</a> tools, organizations can significantly reduce the time and effort required to migrate BigQuery code to Amazon Redshift. The Analyzer provides detailed assessments of the complexity and requirements for the migration, and the Converter automates the actual code conversion process, using pattern-based customizable rules to streamline the transition.</p> 
<p>In this post, we walk through the step-by-step process of using BladeBridge to accelerate the migration of BigQuery SQL code to Amazon Redshift.</p> 
<h2>Solution overview</h2> 
<p>The BladeBridge solution is composed of two key components: the BladeBridge Analyzer and the BladeBridge Converter.</p> 
<h3>BladeBridge Analyzer</h3> 
<p>The Analyzer is designed to thoroughly assess the complexities of the existing data environment, in this case, Google BigQuery. After assessment of the source SQL files, it generates a comprehensive report that provides valuable insights into the migration effort. The Analyzer report includes the following:</p> 
<ul> 
 <li>Summary of the total number of SQL scripts, file scripts, data definition language (DDL) statements, and other key metrics</li> 
 <li>Categorization of the SQL code complexity into levels such as low, medium, complex, and very complex</li> 
 <li>Insights that help both the organizations and systems integrators prepare more accurate project estimates and migration plans</li> 
</ul> 
<h3>BladeBridge Converter</h3> 
<p>The Converter is a pattern-based automation tool that streamlines the actual code conversion process from BigQuery to Amazon Redshift. The Converter uses a set of predefined conversion rules and patterns to automatically translate <a href="https://bladebridge.com/converter/" rel="noopener" target="_blank">70–95% of the legacy SQL code</a>. This significantly reduces the manual effort required by developers. The Converter works by doing the following:</p> 
<ul> 
 <li>Parsing the source SQL files and analyzing the code semantically</li> 
 <li>Applying the appropriate translation rules and patterns to convert source database code to the target, in this case, Google BigQuery to Amazon Redshift</li> 
</ul> 
<p>The out-of-the-box code handles most conversions. The Converter allows developers to customize the conversion patterns for more complex transformations.</p> 
<p>The following is the migration procedure:</p> 
<ol> 
 <li><a href="#Prepare">Prepare SQL files</a></li> 
 <li><a href="#Run1">Using BladeBridge Analyzer, create an analyzer report</a></li> 
 <li><a href="#Purchase">Purchase license keys for converter</a></li> 
 <li><a href="#Run2">Using BladeBridge Converter, convert SQL files</a></li> 
</ol> 
<p>The following diagram illustrates these steps.</p> 
<p><img alt="" class="alignnone wp-image-70652 size-full" height="684" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/23/blade.png" style="margin: 10px 0px 10px 0px;" width="892" /></p> 
<h2>Prerequisites</h2> 
<p>You need the following prerequisites to implement the solution:</p> 
<ul> 
 <li>An <a href="https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&amp;client_id=signup" rel="noopener" target="_blank">AWS account</a></li> 
 <li>An <a href="https://docs.aws.amazon.com/redshift/latest/gsg/new-user.html" rel="noopener" target="_blank">Amazon Redshift provisioned cluster</a> or <a href="https://docs.aws.amazon.com/redshift/latest/gsg/new-user-serverless.html" rel="noopener" target="_blank">Amazon Redshift serverless workgroup</a></li> 
 <li>An <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html" rel="noopener" target="_blank">Amazon Elastic Compute Cloud</a> (Amazon EC2) instance, on-premises server, or desktop or laptop with the following requirements: 
  <ul> 
   <li>MacOS, Windows 7 or higher with 32-bit or 64-bit, Linux Redhat, Ubuntu, or similar operating system</li> 
   <li>A minimum of 8 GB RAM is recommended</li> 
  </ul> </li> 
 <li>Visit the <a href="https://community.bladebridge.com/login" rel="noopener" target="_blank">BladeBridge community portal</a> and sign up to create your account. The portal gives you access to a comprehensive suite of resources, including the BladeBridge Analyzer, Converter, and other training materials. This post contains some links that are only accessible to registered members of the BladeBridge community portal.</li> 
 <li><a href="https://bladebridge.com/analyzer/" rel="noopener" target="_blank">Contact BladeBridge</a> through <strong>Request demo </strong>and obtain an Analyzer key for your organization.</li> 
</ul> 
<h2>Solution walkthrough</h2> 
<p>Follow these solution steps:</p> 
<h3 id="Prepare">Prepare SQL files</h3> 
<p>For SQL data warehouses such as BigQuery, code preparation starts by exporting the SQL files out of the data warehouse solution. If your BigQuery SQL code is stored in a single file containing multiple database objects, you need to split them into individual files before using the BladeBridge tools to convert the code to Amazon Redshift. To split into multiple files, you can use the BladeBridge SQL File Splitter utility. The BladeBridge conversion process is optimized to work with each database object (for example, tables, views, and materialized views) and code object (for example, stored procedures and functions) stored in its own separate SQL file. This allows the BladeBridge Analyzer to scan each file individually, gaining a comprehensive understanding of the code patterns, complexity, and structure. To use BladeBridge <a href="https://community.bladebridge.com/t/sql-file-splitter/45" rel="noopener" target="_blank">SQL File Splitter utility</a>, follow these steps:</p> 
<ol> 
 <li>Log in to <a href="https://community.bladebridge.com/" rel="noopener" target="_blank">BladeBridge portal</a> and download the <a href="https://community.bladebridge.com/t/sql-file-splitter/45" rel="noopener" target="_blank">SQL file splitter</a> utility for your operating system.</li> 
 <li>Create an input file directory and place your BigQuery SQL code files in the directory.</li> 
 <li>Create an empty output file directory. The files generated by the splitter will be stored here.</li> 
 <li>Navigate to the directory where you downloaded the bbsqlsplit executable file and run the following command in your terminal (Mac or Linux) or command prompt (Windows), replacing the input and output file directory paths:</li> 
</ol> 
<p><strong>Syntax</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">bbsqlsplit

######## OPTIONS ########

-d &lt;&lt;input file directory path&gt;&gt;

-o &lt;&lt;output file directory path&gt;&gt;

-E sql</code></pre> 
</div> 
<p><strong>Example&nbsp;</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">bbsqlsplit

-d C:\Users\XXXXX\Desktop\BladeBridge\SplitFilesUtility\source

-o C:\Users\XXXXX\Desktop\BladeBridge\SplitFilesUtility\splitFiles

-E sql</code></pre> 
</div> 
<p>For more options of the bbsqlsplit command, refer to the <a href="https://community.bladebridge.com/t/sql-file-splitter/45" rel="noopener" target="_blank">SQL file split documentation</a> in the BladeBridge community portal.</p> 
<h3 id="Run1">Using BladeBridge Analyzer, create an analyzer report</h3> 
<p>The Analyzer provides a detailed assessment of the existing BigQuery code, generating a comprehensive report that outlines the complexity and requirements for the migration to Amazon Redshift.</p> 
<p>To run the BladeBridge Analyzer, follow these steps:</p> 
<ol> 
 <li>Log in to the BladeBridge portal and navigate to the <a href="https://community.bladebridge.com/t/analyzer-download/46" rel="noopener" target="_blank">Analyzer Download</a></li> 
 <li>Download the Analyzer executable file for your operating system (for example, bbanalyzer.exe for Windows, bbanalyzer.gz for Linux and macOS). For macOS and Linux users, you need to deflate the downloaded gzip file.</li> 
 <li>Download the configuration file (general_sql_specs.json) from the BladeBridge community portal, as shown in the following screenshot.</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-70654 size-full" height="706" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/23/blade1.png" style="margin: 10px 0px 10px 0px;" width="1450" /></p> 
<ol start="4"> 
 <li>On the BladeBridge community portal, choose <strong>Assets</strong>. This page should display the Analyzer key for your organization.</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-70655 size-full" height="876" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/23/blade2.png" style="margin: 10px 0px 10px 0px;" width="1472" /></p> 
<ol start="5"> 
 <li>From the assets page, download the Analyzer key as shown in the following screenshot.</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-70656 size-full" height="584" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/23/blade3.png" style="margin: 10px 0px 10px 0px;" width="1602" /></p> 
<p>In the directory where you downloaded the bbanalyzer executable file, run the following command in your terminal (Mac or Linux) or command prompt (Windows), replacing the necessary paths.</p> 
<p><strong>Syntax</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-code">bbanalyzer

######## OPTIONS ########

-c &lt;&lt;path to your analyzer key&gt;&gt;

-t SQL

-d &lt;&lt;path to your source code directory&gt;&gt;

-r &lt;&lt;name for the output report&gt;&gt;.xlsx</code></pre> 
</div> 
<p><strong>Example</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-code">bbanalyzer

-c C:\Users\XXXXX\Desktop\BladeBridge\analyzer_key.txt

-t SQL

-d C:\Users\XXXXX\Desktop\BladeBridge\SplitFilesUtility\splitFiles

-r analyzer_report.xlsx</code></pre> 
</div> 
<p>After running the command successfully, the Analyzer generates a report. Review the report thoroughly, because it provides a summary and in-depth explanations of the SQL analysis. The summary sheet, shown in the following image, provides an overview of the migration, including the number of total SQL scripts, file scripts, and DDLs. Each SQL script is categorized into LOW, MEDIUM, COMPLEX, or VERY_COMPLEX complexities, which are determined by the <a href="https://community.bladebridge.com/t/analyzer-complexity-determination-algorithm/47/1" rel="noopener" target="_blank">Analyzer Complexity Determination Algorithm</a>. The summary will also help with understanding the overall complexity and migration effort before performing the actual conversion.</p> 
<p><img alt="" class="alignnone size-full wp-image-70212" height="660" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/15/Picture5-2.png" style="margin: 10px 0px 10px 0px;" width="537" /></p> 
<p>If you observe an error when running BladeBridge Analyzer, review following troubleshooting tips:</p> 
<ul> 
 <li><strong>Configure the write permission</strong> – You may need to add necessary permission to the analyzer executable file. For Mac and Linux users, run chmod 755 ./bbanalyzer to modify the permission.</li> 
 <li><strong>Allow running third party software</strong> – Because BladeBridge Analyzer is a third-party software, MacOS may raise a warning or an error when running Analyzer. If you’re using Mac, follow the instructions in <a href="https://support.apple.com/en-il/guide/mac-help/mh40616/mac" rel="noopener" target="_blank">Open a Mac app from an unidentified developer</a>.</li> 
 <li><strong>Use local drive</strong> – In some cases, you might encounter an error if the executable is located in a network drive. We recommend that you run the executable on the local drive.</li> 
 <li><strong>Don’t include whitespace in the path</strong> – Make sure the path to the executable doesn’t contain a directory with spaces in the directory name.</li> 
</ul> 
<p>For more details, refer to the <a href="https://community.bladebridge.com/t/running-analyzer-video/163" rel="noopener" target="_blank">BladeBridge Analyzer Demo</a>.</p> 
<h3 id="Purchase">Purchase license keys for convertor</h3> 
<p>To use the BladeBridge Converter and automate the code translation process, you need to purchase the necessary license keys. These license keys are tied to the specific SQL files you are converting, making sure that updates to the source code require the appropriate license.</p> 
<p>To obtain the license keys, follow these steps:</p> 
<ol> 
 <li>Share the output of the BladeBridge Analyzer report and the provided <a href="https://community.bladebridge.com/t/pricing-calculator-bladebridge-conversion/70" rel="noopener" target="_blank">pricing calculator Excel sheet</a> with BladeBridge.</li> 
 <li>The BladeBridge team will review the information and provide you with the required license keys to run the Converter.</li> 
</ol> 
<p>The license key is tied to the file hash of the SQL files you are converting. If you make updates to the source SQL files, you need to purchase new license keys to convert the modified code. Therefore, make sure to purchase the necessary license keys and manage your files with a version control system to have smooth transitions when converting your BigQuery SQL code to Amazon Redshift.</p> 
<h3 id="Run2">Using BladeBridge Converter, convert SQL files</h3> 
<p>The Converter uses the predefined conversion rules that are available in the out-of-the-box configuration files to automatically translate 70–95% of the legacy code, significantly reducing manual effort for your development team. The out-of-the-box configuration file handles conversion for common code patterns from Google BigQuery to Amazon Redshift. For those custom patterns that aren’t covered by an out-of-the-box configuration file, you can create custom conversion rules by creating additional configuration files.</p> 
<p>Follow these steps to run the BladeBridge Converter:</p> 
<ol> 
 <li>Log in to the BladeBridge portal and on the <a href="https://community.bladebridge.com/t/sql-converter/207" rel="noopener" target="_blank">Convertor downloads</a> page, download the Convertor executable file for your operating system (sqlconv.exe for Windows or sqlconv.gz for Mac or Linux)</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-70657 size-full" height="698" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/23/blade4.png" style="margin: 10px 0px 10px 0px;" width="1740" /></p> 
<ol start="2"> 
 <li>From the same page, download the configuration file (general_sql_specs.json)</li> 
 <li>Create an output directory where the converted files will be saved</li> 
 <li>In the folder where you downloaded the Convertor executable sqlconv, run the following command</li> 
</ol> 
<p><strong>Syntax</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-code">sqlconv

-c &lt;&lt;converter license file name obtained from BladeBridge&gt;&gt;

-d &lt;&lt;input folder for SQL files&gt;&gt;

-n &lt;&lt;output folder for converted files&gt;&gt;

-u &lt;&lt;path for the config files/s provide at least one file&gt;&gt;</code></pre> 
</div> 
<p><strong>Example</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-code">sqlconv

-c converter_license.txt

-d C:\Users\XXXXX\Desktop\BladeBridge\SplitFilesUtility\splitFiles

-n C:\Users\XXXXX\Desktop\BladeBridge\SplitFilesUtility\cnvrtdFiles

-u bq2redshift.json</code></pre> 
</div> 
<ol start="5"> 
 <li>Run the generated SQL files in your Amazon Redshift data warehouse. If you encounter errors, analyze them and determine if custom conversion rules, not already covered in the out-of-the-box configuration files, need to be applied.</li> 
 <li>If custom conversion rules are needed, create a new configuration file following the guidelines in the <a href="#Customize">Customize Configuration File</a> section. Provide the new config file name in the -u option and rerun the Converter.</li> 
 <li>Repeat these steps until all files are converted successfully or manually modified.</li> 
</ol> 
<h2 id="Customize">Customize configuration file</h2> 
<p>Customizing a configuration file is an iterative process that can help automate the conversion for occurrences in your codebase. However, manual conversion may be required if the conversion is needed for only a few files and a few occurrences.</p> 
<p>The configuration is defined in a JSON file. There is a general configuration file with common rules and custom configuration files for each client with client-specific rules. Rules can be added to the general configuration file if they are applicable for all clients. For client-specific rules, a separate JSON file should be created and referenced. This keeps the general rules clean and organized.</p> 
<p>The conversion rules in BladeBridge’s configuration file fall into one of three categories:</p> 
<ol> 
 <li><a href="#Line">Line substitution</a></li> 
 <li><a href="#Block">Block substitution</a></li> 
 <li><a href="#Function">Function substitution</a></li> 
</ol> 
<p>Every line ending with a ; is a statement. This line ending also can be replaced with other breakers. Refer to this <a href="https://community.bladebridge.com/t/bladebridge-sql-and-expression-converter-guide/590" rel="noopener" target="_blank">BladeBridge documentation</a> to get more details on SQL and expression conversion.</p> 
<p>The following are considerations while using the customized configuration:</p> 
<ul> 
 <li>Nested functions in BigQuery allow for complex operations within a single SQL statement, which may need to be broken down into multiple steps in Amazon Redshift</li> 
 <li>Array functions in BigQuery provide capabilities for manipulating and transforming array data types, which may require alternative approaches in Amazon Redshift</li> 
 <li>You need to carefully analyze the requirements and implement workarounds or alternative solutions when migrating from BigQuery to Amazon Redshift, especially for advanced functionality not directly supported in Amazon Redshift</li> 
</ul> 
<h3 id="Line">Line substitution</h3> 
<p>Line substitution applies regular expressions to each line of code. This has the from clause, which has the expression to be converted. The to section has the target mapping for which it’ll be converted. Statement_categories limit the application of line substitution to specific statements such as DDL or procedure. For example:</p> 
<ul> 
 <li>The first expression in the following code example replaces the regular expression pattern ROWNUM with the SQL expression row_number() over (order by 1)</li> 
 <li>The second expression in the following code example replaces the regular expression pattern SYSDATE with the SQL expression CURRENT_TIMESTAMP.</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-code">“<strong>line_subst</strong>” : [
{“from” : “\bROWNUM\b”, “to” : “row_number() over (order by 1)”},
{“from” : “SYSDATE”, “to” : “CURRENT_TIMESTAMP”}
]</code></pre> 
</div> 
<h3 id="Block">Block substitution</h3> 
<p>Block substitution applies regular expressions across multiple lines. This applies to statements that stretch over multiple lines, which are generally more complex than the line substitutions. The following expression in the example replaces the block. In this example, the procedure is created in the target database.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">BEGIN

EXECUTE IMMEDIATE(‘SQL Statement’);

EXCEPTION WHEN OTHERS

THEN

NULL

END;

To

CALL SP_DYN_SQL(‘parameters’);

“block_subst” : [
{“from”: “BEGIN(.*?)execute immediate(.*);.*exception\s*when\s*others\s*then\s*null(.*?)end;(.*)”, “to”: “CALL sp_dyn_sql($2);”}
]</code></pre> 
</div> 
<h3 id="Function">Function substitution</h3> 
<p>Function substitution allows replacing one function with an equivalent function in the target data warehouse. The configuration also allows for specifying custom functions.</p> 
<p>Function substitution points to an array of instructions responsible for altering function calls. This section is used when function translations are required or function arguments (function signature) have to be altered. The following expression converts the NVL2 function to CASE function on Amazon Redshift.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">“function_subst” : [
{“from”: “NVL2”, “output_template” : “CASE WHEN $1 IS NOT NULL THEN $2 ELSE $3 END”}
]</code></pre> 
</div> 
<h2>Conclusion</h2> 
<p>In this post, we demonstrated how to use the BladeBridge Analyzer and BladeBridge Converter to streamline the migration of SQL code from Google BigQuery to Amazon Redshift. By using BladeBridge, organizations can significantly reduce the time and effort required to translate their existing BigQuery code for migration to the Amazon Redshift data warehouse. The Analyzer provides a detailed assessment of the source SQL code, and the Converter automates the actual conversion process using a set of predefined, customizable rules and patterns.</p> 
<p>We also covered the customization capabilities of the BladeBridge solution, showcasing how you can tailor the conversion rules to handle more complex transformations. By using the line substitution, block substitution, and function substitution features, you can have a seamless migration that addresses the unique requirements of your data analytics infrastructure.</p> 
<p>We encourage you to try out BladeBridge’s <a href="https://bladebridge.com/configurations/gcp-bigquery-to-aws-redshift/" rel="noopener" target="_blank">GCP BigQuery to Amazon Redshift solution</a> and explore the various configuration options. If you encounter any challenges or have additional requirements, refer to the <a href="https://community.bladebridge.com" rel="noopener" target="_blank">BladeBridge community support portal</a> or reach out to the BladeBridge team for further assistance.</p> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-70418 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/18/ritesh-100.png" width="100" />Ritesh Kumar Sinha</strong> is an Analytics Specialist Solutions Architect based out of San Francisco. He has helped customers build scalable data warehousing and big data solutions for over 16 years. He loves to design and build efficient end-to-end solutions on AWS. In his spare time, he loves reading, walking, and doing yoga.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-70419 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/18/Anusha.png" width="100" />Anusha Challa</strong> is a Senior Analytics Specialist Solutions Architect focused on Amazon Redshift. She has helped many customers build large-scale data warehouse solutions in the cloud and on premises. She is passionate about data analytics and data science.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-70422 alignleft" height="120" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/18/awsyota.png" width="100" />Yota Hamaoka</strong> is an Analytics Solution Architect at Amazon Web Services. He is focused on driving customers to accelerate their analytics journey with Amazon Redshift.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-70421 alignleft" height="129" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/18/Milindo.png" width="100" />Milind Oke</strong> is a Data Warehouse Specialist Solutions Architect based out of New York. He has been building data warehouse solutions for over 15 years and specializes in Amazon Redshift.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-70420 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/10/18/raza_new_new_new.jpg" width="100" />Raza Hafeez</strong> is a Senior Product Manager at Amazon Redshift. He has over 13 years of professional experience building and optimizing enterprise data warehouses and is passionate about enabling customers to realize the power of their data. He specializes in migrating enterprise data warehouses to AWS Modern Data Architecture.</p>
