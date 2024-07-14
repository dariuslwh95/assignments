# In the context of a relational database (ex. PostgreSQL, MS-SQL) formulate a backup and restore strategy to allow point of time recovery. Explain the mechanism of backups to be configured, and how a restore may be performed. You may use an example timeline to illustrate the process.

To allow point in time recovery, a backup strategy including (1) full backups, (2) incremental backups.
The write ahead logging (WAL) feature is used for incremental backups for every change to be recorded before being written to the actual database should be configured.
This process will be in a PostgreSQL context.

## WAL configuration
For incrementatal backups, WAL configuration should be setup for streaming in real time for archival through tools such as pg_receivewal or walreceiver. 

## Full backups
cronjobs executing full backup commands using pg_basebackup should be run over a set duration e.g. 4 hours

## example timeline
00:00 last full backup
02:30 point in time for recovery

Stop PostgreSQL and remove old data.
Extract the full backup from midnight.
Set up recovery.conf:

Point to the directory containing archived WAL files.
Specify recovery_target_time as '02:30:00'.
