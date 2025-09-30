flowchart TD

A[Start] --> B[Source env: Environment.properties]
B --> C[Beeline/Hive sanity check for run_date]
C --> D[Sleep 20m to allow upstream to finish]

D --> E{Wait on SC020055}
E -->|Poll 1| E1[grep in $log_file_path/SC020055_${run_date}.log<br/>"Successfully moved the data to Archive Directory"]
E -->|Poll 2| E2[hdfs dfs -ls $csoFulfillment_hdfs_scooped_location/$run_date<br/>expect 0 files]
E1 --> F
E2 --> F
F{Both conditions met?}
F -- No (after retries) --> Z[Exit: Upstream not complete]
F -- Yes --> G[Spark submit: com.optum.gcp.fulfillmentquoteExtraction<br/>YARN: 30 exec × 2 cores × 10G]

G --> H{Spark status (with retries)}
H -- Fail after 5 tries --> Z[Exit: Spark failed]
H -- Success --> I[Copy results from HDFS ➜ local]
I --> J[AzCopy ➜ Azure Blob]
J --> K{Transfer success?}
K -- No --> Z[Exit: Transfer failed]
K -- Yes --> L[Trigger: CSO_Fulfillment_Pipeline_Trigger.sh]
L --> M[Mark CSO Fulfillment DW complete]
M --> N[End]
