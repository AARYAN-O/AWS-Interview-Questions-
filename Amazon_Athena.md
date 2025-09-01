
Basic Athena Questions

1. What is Amazon Athena and how does it differ from Redshift and EMR?
Answer: Athena is a serverless, interactive query service that allows you to run SQL directly on data in S3 without managing infrastructure. Unlike Redshift, it does not store data in a warehouse; unlike EMR, it does not run persistent clusters—Athena spins up a query engine on demand.

2. What underlying technology does Athena use to run queries?
Answer: Athena uses Presto (Trino) SQL engine to run distributed queries on S3 data.

3. What file formats are supported in Athena? Which ones are most efficient?
Answer: Supports CSV, JSON, Avro, ORC, Parquet, and more. Columnar formats like Parquet or ORC are most efficient for large datasets because they reduce scan size and improve query performance.

4. How does Athena pricing work?
Answer: Pricing is based on data scanned per query. Reducing scanned data via partitioning, compression, or columnar formats lowers costs.

5. Where does Athena store query results by default? Can we change it?
Answer: Default results are stored in an S3 bucket under aws-athena-query-results. You can change the output location in Athena settings or per query.

6. What are the limitations of Athena?
Answer:

Limited DML support (no UPDATE/DELETE natively, except via CTAS/INSERT).

Performance depends on S3 file size and format.

Queries scanning huge unpartitioned CSVs can be slow/costly.



---

Data Modeling & Partitioning

7. How do you create an external table in Athena?
Answer: Use CREATE EXTERNAL TABLE specifying schema, S3 location, and file format. Example:

CREATE EXTERNAL TABLE IF NOT EXISTS logs (
  user_id STRING,
  event_time TIMESTAMP,
  event_type STRING
)
PARTITIONED BY (year INT, month INT)
STORED AS PARQUET
LOCATION 's3://my-bucket/logs/';

8. Difference between partitioning and bucketing in Athena?
Answer:

Partitioning divides data into separate folders by key (reduces data scanned).

Bucketing divides data into files within a partition (helps with joins, distribution) — Athena doesn’t natively support bucketing for performance like Hive.


9. How do you add partitions in Athena?
Answer: Either MSCK REPAIR TABLE table_name if partitions exist in S3, or manually using ALTER TABLE table_name ADD PARTITION.

10. Why is partitioning important for Athena queries?
Answer: It reduces the amount of data scanned, lowering query cost and execution time. Only relevant partitions are scanned.

11. Query only March 2024 data efficiently:

SELECT * FROM logs
WHERE year = 2024 AND month = 3;

Partition pruning ensures only March 2024 folder is scanned.


---

Performance Optimization

12. How does Athena handle large datasets efficiently?
Answer: Distributed Presto engine reads only required partitions and columns (projection pruning) and can handle parallel scans on S3.

13. How can you reduce Athena query costs?
Answer:

Partition data properly.

Use columnar formats (Parquet/ORC).

Compress files (Snappy, Gzip).

Avoid SELECT *; select only necessary columns.


14. Why Parquet/ORC over CSV in Athena?
Answer: Columnar format → less data scanned, better compression, faster queries. CSV is row-based → full scan every time.

15. How does compression affect Athena performance?
Answer: Compression reduces S3 storage scanned → faster queries and lower cost. Example: Snappy or Gzip with Parquet is common.

16. How do you use CTAS to optimize queries?
Answer: CTAS (Create Table As Select) allows you to transform raw data into optimized format (partitioned/compressed) for faster queries:

CREATE TABLE logs_parquet
WITH (format='PARQUET', parquet_compression='SNAPPY', external_location='s3://bucket/optimized/')
AS SELECT * FROM logs_raw;


---

Integrations & Security

17. How does Athena integrate with Glue Data Catalog?
Answer: Athena uses Glue Data Catalog as a metadata store for databases and tables, allowing centralized schema management.

18. Role of AWS Lake Formation with Athena?
Answer: Provides fine-grained access control, row/column-level security, and centralized governance for Athena queries.

19. How do you secure data queried through Athena?
Answer: Use IAM policies, S3 bucket policies, encryption at rest (S3 SSE), and optionally Lake Formation for row-level access control.

20. How do you use Athena with QuickSight?
Answer: Athena can act as a data source for QuickSight, allowing visualization of S3 data without ETL. Queries are run via Athena, and QuickSight dashboards are refreshed as needed.


---

SQL-Specific Athena Questions

21. What SQL engine does Athena support?
Answer: Athena uses Presto SQL (Trino) — ANSI SQL compatible with some limitations.

22. How do you handle schema evolution in Athena?
Answer: Use MSCK REPAIR TABLE to detect new partitions or ALTER TABLE ADD COLUMN to reflect changes. Athena can read new columns added to files automatically if they exist in the schema.

23. Can you update or delete rows in Athena?
Answer: Not directly. Use:

CTAS to create a new table excluding unwanted rows.

INSERT INTO to append data.

Or use AWS Glue / EMR for full DML operations.


24. How do you join multiple S3 datasets in Athena?
Answer: Standard SQL joins (INNER JOIN, LEFT JOIN) on tables created over S3 datasets. Performance improves if tables are partitioned and/or stored in columnar format.

25. How do you query nested JSON data in Athena?
Answer: Use dot notation or json_extract() functions. Example:

SELECT data.user.id, data.event.type
FROM logs_json;


---

Scenario / Case Study Questions

26. Billions of log files in S3 – how to design schema?
Answer: Partition by date (year/month/day), use columnar format (Parquet/ORC), compress with Snappy, catalog in Glue.

27. Optimize query scanning terabytes of CSV?
Answer: Convert to Parquet, partition data, compress, select only required columns.

28. Athena query is slow – troubleshooting steps?
Answer:

Check if partitions are used.

Check file format (CSV → Parquet).

Reduce scanned columns.

Consider splitting/merging very small files.


29. Migrate Hive table to Athena?
Answer: Use Glue Data Catalog to import Hive metadata, ensure table format (Parquet/ORC), and adjust partitioning if needed.

30. Row-level security / fine-grained access?
Answer: Use AWS Lake Formation to restrict access by row/column, or IAM policies to restrict query execution.
