# Troubleshooting

This document records issues encountered during deployment and the methods used to diagnose and resolve them.

## 1. MySQL Backup Failed Due to Tablespace Privileges

### Symptom

'mysqldump' returned:

Access denied; you need (at least one of) the PROCESS privilege(s)
for this operation

The error occurred while attempting to dump tablespaces.

### Investigation

The backup was being performed by the NuxBill MySQL user. That account did not have the 'PROCESS' privilege required for the tablespace operation.

### Resolution

The backup process was changed so that tablespace information was not included in the logical database dump.

This avoided granting the application database user unnecessary administrative privileges.

### Lesson

A database backup command may require privileges beyond those needed by the application itself.

When possible, it is preferable to adjust the backup operation rather than unnecessarily expanding the privileges of an application account.

## 2. Restore Failed Due to Database Permissions

### Symptom

Attempting to restore into a test database produced:

ERROR 1044 (42000):
Access denied for user 'nuxbill'@'%' to database 'nuxbill_restore_test'


### Investigation

The 'nuxbill' application account had access to the production database but did not have the required permissions for the separate restore-test database.

The distinction was important:

Application database access
        ≠
Administrative database operations

### Resolution

Administrative privileges were used for the restore-test operation rather than granting the application account unnecessary database-management privileges.

The restored database was then queried and its tables verified.

### Lesson

Database users should have permissions appropriate to their role.

An application account does not automatically need permission to create or manage arbitrary databases.

## 3. Empty Backup File

### Symptom

One backup attempt produced a zero-byte SQL file:

nuxbill_2026-08-26_09-14-19.sql

The file existed but contained no dump data.

A subsequent backup produced a valid approximately 43 KB SQL dump.

### Investigation

The output file's size was checked with:

ls -lh /opt/ispb/backups/

The generated SQL was then inspected to determine whether the dump contained valid MySQL output.

### Resolution

The failed backup was not treated as a valid recovery point.

A successful dump was generated and subsequently used for restore testing.

### Lesson

A backup job succeeding at the process level does not necessarily mean the resulting backup is usable.

Backup verification should include checking that:

* the file exists
* the file is non-empty
* the dump contains expected SQL content
* the backup can actually be restored


## 4. Verifying the Restore

### Symptom

A successful-looking SQL import does not by itself prove that the database is recoverable.

### Investigation

The restored database was queried directly:

docker exec mysql mysql -unuxbill -p... \
  -e "USE nuxbill_restore_test; SHOW TABLES;"


The expected NuxBill tables were present.

### Resolution

The restored database was considered successfully verified after confirming the expected schema.

The temporary restore-test database was then removed.

### Lesson

A restore test is stronger evidence than simply having a backup file.

The recovery process should be tested separately from the production database whenever possible.

## 5. Verifying Scheduled Backups

### Symptom

A backup script can exist without actually running on schedule.

### Investigation

The systemd timer was checked using:

systemctl list-timers

The deployment showed:

nuxbill-backup.timer

triggering:

nuxbill-backup.service

### Resolution

The timer schedule and last/next execution information were verified.

Backup files in '/opt/ispb/backups/' were also checked to confirm that scheduled executions were producing SQL dumps.

### Lesson

Scheduled automation should be verified from both sides:

Scheduler
   ↓
Service execution
   ↓
Backup output

Checking only the timer configuration is insufficient.

## Troubleshooting Approach

The general troubleshooting process used throughout the deployment was:

Observe the error
       ↓
Identify the component involved
       ↓
Inspect configuration/logs/output
       ↓
Determine the permission or configuration boundary
       ↓
Make the smallest appropriate change
       ↓
Test again
       ↓
Verify the result independently

This approach was particularly useful for distinguishing between:

* application configuration problems
* Docker configuration problems
* MySQL permission problems
* backup problems
* scheduling problems
