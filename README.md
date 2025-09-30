flowchart TD

  A[SCM20453<br/>Sqoop Ingestion<br/>Source DB → HDFS/Hive] --> B[SCM20455<br/>Hive Loads + Archive<br/>SHIPCAL_xxx + Archive flag]
  B --> C[DMP Job<br/>Spark ETL Fulfillment Extraction<br/>HDFS/Hive → Azure Blob → CSO Pipeline]
