# Backup and Restore

## Objective

Maintain recoverable backups of the NuxBill MySQL database and verify that backups can be restored successfully.

## Backup

Database backups are created using 'mysqldump' from the MySQL Docker container.

Backups are stored on the server under:

/opt/ispb/backups/


Example:nuxbill_2026-08-28_02-00-05.sql


The backup files are generated automatically on a scheduled basis.

## Backup Privileges

During the initial backup process, 'mysqldump' reported:

Access denied; you need (at least one of) the PROCESS privilege(s)
for this operation

The error occurred while 'mysqldump' attempted to dump tablespaces.

Rather than granting unnecessary privileges to the application database user, the backup process was adjusted so that tablespace information was not required for the logical backup.

This follows the principle of least privilege: "give an account only the permissions required for its task."

## Backup Verification

A generated SQL dump was inspected to confirm that it contained a valid MySQL dump:

-- MySQL dump 10.13  Distrib 8.0.46
-- Host: localhost    Database: nuxbill
-- Server version: 8.0.46


The backup contained NuxBill table definitions and data.

## Restore Testing

A separate test database was created to verify that the backup could be restored without modifying the production database.

The restore was initially affected by MySQL privileges:


ERROR 1044 (42000):
Access denied for user 'nuxbill'@'%' to database 'nuxbill_restore_test'


The issue was caused by the application database user not having permission to create/use the separate test database.

The restore was subsequently performed with appropriate administrative privileges.

## Restore Verification

After restoration, the test database was queried:

bash:
docker exec mysql mysql -unuxbill -p... \
  -e "USE nuxbill_restore_test; SHOW TABLES;"

The restored database contained the expected NuxBill tables, including:

nas
nasreload
rad_acct
radacct
radcheck
radgroupcheck
radgroupreply
radpostauth
radreply
radusergroup
tbl_appconfig
tbl_bandwidth
tbl_coupons
tbl_customers
tbl_customers_fields
tbl_customers_inbox
tbl_logs
tbl_message_logs
tbl_meta
tbl_odps
tbl_payment_gateway
tbl_plans
tbl_pool
tbl_port_pool
tbl_routers
tbl_transactions
tbl_user_recharges

The test database was then removed after verification.

## Scheduled Backups

Backups are scheduled using a systemd timer:

nuxbill-backup.timer

The timer triggers: nuxbill-backup.service

The relationship is:

systemd timer
      │
      ▼
backup service
      │
      ▼
mysqldump
      │
      ▼
/opt/ispb/backups/


The timer was verified using:

bash
systemctl list-timers

## Recovery Process

A basic recovery procedure is:

1. Identify the required SQL backup.
2. Create or select a recovery database.
3. Import the SQL dump using MySQL.
4. Verify the expected tables and data.
5. If restoring production, update the application connection/configuration as required.
6. Test the application after restoration.

## Important Considerations

Backups contain database data and should not be committed to Git or exposed publicly.

The 'backups/' directory and SQL dump files are excluded from the portfolio repository through '.gitignore'.

The actual '.env' file and NuxBill's generated 'config.php' are also excluded from version control.
