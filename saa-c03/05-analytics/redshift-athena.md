# Redshift & Athena

## Amazon Redshift

- Columnar data warehouse for petabyte-scale queries
- Not serverless — requires running EC2-based cluster instances
- **Redshift Spectrum** queries S3 data directly with parallel processing, without loading into Redshift

## Amazon Athena

- Serverless SQL queries directly on S3 data
- Inexpensive, pay-per-query option
- Native encryption support (client/server-side)

## S3 Select

- Run SQL analysis directly on S3 objects without spinning up a data warehouse

## Notes

<!-- Your own notes go here. -->
