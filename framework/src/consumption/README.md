[//]: # (consumption.redshift-serverless-namespace)
# RedshiftServerlessNamespace

A [Redshift Serverless Namespace](https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-workgroup-namespace.html) with secrets manager integration for admin credentials management and rotation.

## Overview

`RedshiftServerlessNamespace` is a [Redshift Serverless Namespace](https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-workgroup-namespace.html) with the following options:
- Encrypt data with a customer managed KMS Key.
- Create Redshift superuser credentials managed by Redshift service: stored in Secrets Manager, encrypted with a KMS Key, and with automatic rotation.
- Attach multiple IAM roles that can be used by Redshift Serverless users to interact with other AWS services.
- Set an [IAM role as default](https://docs.aws.amazon.com/redshift/latest/mgmt/default-iam-role.html)

## Usage

[example default usage](./examples/redshift-serverless-namespace-default.lit.ts)

## Attaching IAM Roles to Redshift Serverless Namespace

To allow Redshift Serverless to access other AWS services on your behalf (eg. data ingestion from S3 via the COPY command, accessing data in S3 via Redshift Spectrum, exporting data from Redshift to S3 via the UNLOAD command.), the preferred method is to specify an IAM role. High-level steps are as follows:

1. Create an IAM role with a trust relationship of `redshift.amazonaws.com`.
2. Attach policy/permissions to the role to give it access to specific AWS services.
3. Configure the role when creating the Redshift Serverless Namespace
4. Run the relevant SQL command referencing the attached IAM role via its ARN (or the `default` keyword if a default IAM role is configured)

[example default IAM role configuration](./examples/redshift-serverless-namespace-roles.lit.ts)

[//]: # (consumption.redshift-serverless-workgroup)
# RedshiftServerlessWorkgroup

A [Redshift Serverless Workgroup](https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-workgroup-namespace.html) with helpers method for Redshift administration. 

## Overview
`RedshiftServerlessWorkgroup` is a [Redshift Serverless Workgroup](https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-workgroup-namespace.html) with the following options/capabilities:
- Deployed in a VPC in private subnets. The network configuation can be customized.
- Provide helper methods for running SQL commands via the Redshift Data API. Commands can be custom or predefined for common administration tasks like creating and granting roles.
- Initialize a Glue Data Catalog integration with auto crawling via Glue Crawlers. This would allow tables in Redshift Serverless to appear in the [Glue Data Catalog](https://docs.aws.amazon.com/glue/latest/dg/catalog-and-crawler.html) for the purposes of discovery and integration.

## Usage

[example default usage](./examples/redshift-serverless-workgroup-default.lit.ts)

## Bootstrapping Redshift Serverless w/ RedshiftData Construct

The `RedshiftData` construct allows custom SQLs to run against the `RedshiftServerlessWorkgroup` via the Data API. This allows users to bootstrap Redshift directly from CDK.

The `RedshitData` construct provides the following helpers for bootstrapping Redshift databases:
- Run a custom SQL command
- Create Redshift roles
- Grant Redshift roles full access to schemas
- Grant Redshift roles read only access
- Run a COPY command to load data 

[example bootstrap](./examples/redshift-serverless-workgroup-bootstrap.lit.ts)

## Cataloging Redshift Serverless Tables

Redshift tables and databases can also be automatically catalog in Glue Data Catalog using an helper method. This method creates a Glue Catalog database as well as a crawler to populate the database with table metadata from your Redshift database.

The default value of the path that the crawler would use is `<databaseName>/public/%` which translates to all the tables in the public schema. Please refer to the [crawler documentation](https://docs.aws.amazon.com/glue/latest/dg/define-crawler.html#define-crawler-choose-data-sources) for more information for JDBC data sources.

[example catalog](./examples/redshift-serverless-workgroup-catalog.lit.ts)

[//]: # (consumption.athena-workgroup)
# Athena Workgroup

An [Amazon Athena workgroup](https://docs.aws.amazon.com/athena/latest/ug/manage-queries-control-costs-with-workgroups.html) with provided configuration.

`AthenaWorkGroup` provides Athena workgroup configuration with best-practices:
- Amazon S3 bucket for query results, based on [`AnalyticsBucket`](https://awslabs.github.io/data-solutions-framework-on-aws/docs/constructs/library/Storage/analytics-bucket).
- Query results are encrypted using AWS KMS Key.
- Execution Role for the PySpark query engine.
- A grant method to allow principals to run queries.

## Usage

[example default athena usage](./examples/athena-workgroup-default.lit.ts)

## User provided S3 bucket for query results

You can provide your own S3 bucket for query results. If you do so, you are required to provide a KMS Key that will be used to encrypt query results.

:::caution Results encryption
If you provide your own S3 bucket, you also need to provide KMS encryption key to encrypt query results. You also need to 
grant access to this key for AthenaWorkGroup's executionRole (if Spark engine is used), or for principals that were granted to run
queries using AthenaWorkGroup's `grantRunQueries` method.
:::caution

You can also decide to provide your KMS Key to encrypt query results with S3 bucket that is provided by the construct (i.e. if you are not providing your own S3 bucket).

[example usage with user provided bucket](./examples/athena-workgroup-user-bucket.lit.ts)

## Apache Spark (PySpark) Engine version

You can choose Athena query engine from the available options:
- [Athena engine version 3](https://docs.aws.amazon.com/athena/latest/ug/engine-versions-reference-0003.html)
- [PySpark engine version 3](https://docs.aws.amazon.com/athena/latest/ug/notebooks-spark.html)

The `default` is set to `AUTO` which will choose Athena engine version 3. 

If you wish to change query engine to PySpark, you will also be able to access the `executionRole` IAM Role that will be created for you if you don't provide it. 
You can access the execution role via `executionRole` property.

[example usage with pyspark](./examples/athena-workgroup-spark.lit.ts)

## Construct properties

You can leverage different properties to customize your Athena workgroup. For example, you can use `resultsRetentionPeriod` to specify the retention period for your query results. You can provide your KMS Key for encryption even if you use provided results bucket. You can explore other properties available in `AthenaWorkGroupProps`.

[example usage with properties](./examples/athena-workgroup-properties.lit.ts)

## Grant permission to run queries

We provide `grantRunQueries` method to grant permission to principals to run queries using the workgroup.

[example usage with properties](./examples/athena-workgroup-grant.lit.ts)


## Workgroup removal

You can specify if Athena Workgroup construct resources should be deleted when CDK Stack is destroyed using `removalPolicy`. To have an additional layer of protection, we require users to set a global context value for data removal in their CDK applications.

Athena workgroup will be destroyed only if **both** the removal policy parameter of the construct and DSF global removal policy are set to remove objects.

If set to be destroyed, Athena workgroup construct will use `recursiveDeleteOption`, that will delete the workgroup and its contents even if it contains any named queries.

You can set `@data-solutions-framework-on-aws/removeDataOnDestroy` (`true` or `false`) global data removal policy in `cdk.json`:

```json title="cdk.json"
{
  "context": {
    "@data-solutions-framework-on-aws/removeDataOnDestroy": true
  }
}
```