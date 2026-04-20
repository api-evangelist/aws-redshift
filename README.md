# AWS Redshift

Amazon Redshift is a fast, fully managed, petabyte-scale data warehouse service that makes it simple and cost-effective to analyze all your data using standard SQL and existing Business Intelligence tools.

## APIs

### Amazon Redshift API
Programmatic access to create and manage Amazon Redshift clusters and their associated resources including snapshots, parameter groups, subnet groups, and reserved nodes.
- **Documentation**: https://docs.aws.amazon.com/redshift/latest/APIReference/Welcome.html
- **OpenAPI**: [openapi/aws-redshift-openapi.json](openapi/aws-redshift-openapi.json) (238 operations)

### Amazon Redshift Data API
Run SQL statements without managing connections via a secure HTTP endpoint. Supports synchronous and asynchronous SQL execution against Redshift clusters and Serverless workgroups.
- **Documentation**: https://docs.aws.amazon.com/redshift-data/latest/APIReference/Welcome.html
- **OpenAPI**: [openapi/aws-redshift-data-openapi.json](openapi/aws-redshift-data-openapi.json) (10 operations)

### Amazon Redshift Serverless API
Run analytics workloads without managing data warehouse infrastructure. Automatically provisions and scales data warehouse capacity on demand.
- **Documentation**: https://docs.aws.amazon.com/redshift-serverless/latest/APIReference/Welcome.html

## Artifacts

| Directory | Contents |
|---|---|
| [openapi/](openapi/) | 2 OpenAPI specifications |
| [json-schema/](json-schema/) | 595 JSON Schema files |
| [json-structure/](json-structure/) | 595 JSON Structure files |
| [json-ld/](json-ld/) | 2 JSON-LD context files |
| [examples/](examples/) | 595 example files |
| [rules/](rules/) | Spectral ruleset |
| [capabilities/](capabilities/) | Naftiko capability definitions |
| [vocabulary/](vocabulary/) | Domain vocabulary |

## Features

- **Petabyte-Scale Storage** — Store and query petabytes of structured and semi-structured data with columnar storage.
- **Standard SQL Support** — Query data using standard SQL and connect with existing BI tools via JDBC/ODBC.
- **Massively Parallel Processing** — Distribute SQL operations across multiple nodes for high-performance query execution.
- **Columnar Storage** — Store data in columnar format for efficient analytical query performance and compression.
- **Automated Snapshots** — Automated and manual snapshots for point-in-time recovery of your cluster data.
- **Data Sharing** — Share live data across Redshift clusters and accounts without copying data.
- **ML Integration** — Run Amazon Redshift ML to create, train, and deploy machine learning models using SQL.
- **Serverless Mode** — Run analytics workloads without managing cluster infrastructure with Redshift Serverless.
- **Federated Query** — Query data across operational databases, data warehouses, and data lakes.
- **AQUA** — Advanced Query Accelerator for up to 10x faster query performance.

## Use Cases

- **Business Intelligence** — Power BI dashboards and reports with fast analytical queries over large datasets.
- **Log Analytics** — Analyze application logs and clickstream data for operational insights.
- **Financial Analytics** — Process financial transactions and generate regulatory reports over historical data.
- **Data Lake Analytics** — Query data in Amazon S3 data lakes using Redshift Spectrum without loading.
- **Machine Learning** — Build and deploy ML models directly within the warehouse using SQL with Redshift ML.

## Links

- **Website**: https://aws.amazon.com/redshift/
- **Getting Started**: https://docs.aws.amazon.com/redshift/latest/gsg/getting-started.html
- **Pricing**: https://aws.amazon.com/redshift/pricing/
- **Blog**: https://aws.amazon.com/blogs/big-data/category/database/amazon-redshift/
- **Change Log**: https://aws.amazon.com/releasenotes/Amazon-Redshift/
- **Status**: https://health.aws.amazon.com/health/status

## Maintainers

- **Kin Lane** — kin@apievangelist.com
