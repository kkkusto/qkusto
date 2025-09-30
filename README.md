flowchart TD
    A[Start Script] --> B[Source Environment & Config Files]
    B --> C[Hive Query: Get Previous Run Date / Partition Prep]
    C --> D[Run Sqoop Extraction (Parquet, Spark, HDFS)]
    D --> E{Sqoop Success?}
    
    E -- No --> F[Retry / Wait Loop]
    F --> E
    E -- Still Fails --> G[Send Failure Email & Exit]
    
    E -- Yes --> H[Move Data to HDFS Archive Directory]
    H --> I[Copy Files Locally & Upload via AzCopy to Azure]
    
    I --> J{File Transfer Success?}
    J -- No --> K[Send Failure Email & Exit]
    J -- Yes --> L[Send Success Email]
    
    L --> M[Trigger Next Pipeline Script: SyncCsoFulfillmentPipelineTrigger.sh]
    M --> N[End]
