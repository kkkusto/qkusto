#!/bin/bash

LOGFILE=your_logfile.log
OUTFILE=table_mapping.csv

# Write header
echo "Source_Table,Target" > $OUTFILE

# Look for sqoop import lines
grep -i "sqoop import" $LOGFILE | while read line; do
    # Extract source table
    src=$(echo $line | grep -oP '(?<=--table )\S+')

    # Extract hive target if present
    tgt_hive=$(echo $line | grep -oP '(?<=--hive-table )\S+')

    # Extract HDFS target if present
    tgt_hdfs=$(echo $line | grep -oP '(?<=--target-dir )\S+')

    # Decide target priority: Hive table > HDFS dir
    if [ -n "$tgt_hive" ]; then
        tgt=$tgt_hive
    else
        tgt=$tgt_hdfs
    fi

    # Write to CSV
    echo "$src,$tgt" >> $OUTFILE
done
