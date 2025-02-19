# Specification of pg_dumpall

## Description

***pg_dumpall*** is a utility for dumping all PostgreSQL databases of a cluster into one script file. The script file contains SQL commands that can be used as input to ***psql*** to restore the databases. It does this by calling ***pg_dump*** for each database in the cluster. ***pg_dumpall*** also dumps global objects that are common to all databases, namely database roles, tablespaces, and privilege grants for configuration parameters.

## Output

  - Data Only ([-a | --data-only])
    Dump only data, not the schema (data definitions)
	
	- Disable Triggers ([--disable-triggers])
      This option is relevant only when creating a data-only dump. It instructs pg_dumpall to include commands to temporarily disable triggers on the target tables while the data is restored. Use this if you have referential integrity checks or other triggers on the tables that you do not want to invoke during data restore.
      Presently, the commands emitted for --disable-triggers must be done as superuser. So, you should also specify a superuser name with -S, or preferably be careful to start the resulting script as a superuser.

  - Clean before Create ([-c | --clean])
    Emit SQL commands to DROP all the dumped databases, roles and tablespaces before recreating them. This option is useful when the restore is to overwrite an existing cluster.
	
	- Suppress Errors ([--if-exists])
      Use DROP ... IF EXISTS commands to drop objects in --clean mode. This suppresses “does not exist” errors that might otherwise be reported. This option is not valid unless --clean is also specified.

  - Globals Only ([-g | --globals-only])
    Dump only global objects (roles and tablespaces), no databases
  - No Owner ([-o | --no-owner])
    Do not output commands to set ownership of objects to match the original database. By default, pg_dumpall issues ALTER OWNER or SET SESSION AUTHORIZATION statements to set ownership of created schema elements. These statements will fail when the script is run unless it is started by a superuser (or the same user that owns all of the objects in the script). To make a script that can be restored by any user, but will give that user ownership of all the objects, specify -O.
  - Roles Only ([-r | --roles-only])
    Dump only roles, no databases or tablespaces.
  - Schema Only ([-s | --schema-only])
    Dump only the object definitions (schema), not data.
  - Tablespace Only ([-t | --tablespaces-only])
    Dump only tablespaces, no databases or roles.
  - No Privileges ([-x | --no-privileges | --no-acl])
    Prevent dumping of access privileges (grant/revoke commands).
  - With Column Names ([--column-inserts | --attribute-inserts])
    Dump data as INSERT commands with explicit column names (INSERT INTO table (column, ...) VALUES ...). It is mainly useful for making dumps that can be loaded into non-PostgreSQL databases.
	
	- Do Nothing On Conflict ([--on-conflict-do-nothing])
      Add ON CONFLICT DO NOTHING to INSERT commands. This option is not valid unless --inserts or --column-inserts is also specified.

  - Disable $ Quoting ([--disable-dollar-quoting])
    This option disables the use of dollar quoting for function bodies, and forces them to be quoted using SQL standard string syntax.
  - Exclude Database Matching Pattern ([--exclude-database=pattern])
    Do not dump databases whose name matches pattern. Multiple patterns can be excluded by writing multiple --exclude-database switches. 
  - Specify Float Digits ([--extra-float-digits=ndigits])
    Use the specified value of extra_float_digits when dumping floating-point data, instead of the maximum available precision. Routine dumps made for backup purposes should not use this option.
  - Use INSERT command ([--inserts])
    Dump data as INSERT commands (rather than COPY). This will make restoration very slow; it is mainly useful for making dumps that can be loaded into non-PostgreSQL databases. Note that the restore might fail altogether if you have rearranged column order. The --column-inserts option is safer, though even slower.
	
	- Do Nothing On Conflict ([--on-conflict-do-nothing])
      Add ON CONFLICT DO NOTHING to INSERT commands. This option is not valid unless --inserts or --column-inserts is also specified.
	  
  - Use INSERT with Number of Rows ([--rows-per-insert=nrows])
      Dump data as INSERT commands (rather than COPY). Controls the maximum number of rows per INSERT command. The value specified must be a number greater than zero. Any error during restoring will cause only rows that are part of the problematic INSERT to be lost, rather than the entire table contents.
  - Load VIA Root Partition ([--load-via-partition-root])
    When dumping data for a table partition, make the COPY or INSERT statements target the root of the partitioning hierarchy that contains it, rather than the partition itself. This causes the appropriate partition to be re-determined for each row when the data is loaded. This may be useful when restoring data on a server where rows do not always fall into the same partitions as they did on the original server. That could happen, for example, if the partitioning column is of type text and the two systems have different definitions of the collation used to sort the partitioning column.
  - No Comments ([--no-comments])
    Do not dump comments.
  - No Publications ([--no-publications])
    Do not dump publications.
  - No Role Passwords ([--no-role-passwords])
    Do not dump passwords for roles. When restored, roles will have a null password, and password authentication will always fail until the password is set. Since password values aren't needed when this option is specified, the role information is read from the catalog view pg_roles instead of pg_authid. Therefore, this option also helps if access to pg_authid is restricted by some security policy.
  - No Security Labels ([--no-security-labels])
    Do not dump security labels.
  - No Subscription ([--no-subscriptions])
    Do not dump subscriptions.
  - No Table Access Method ([--no-table-access-method])
    Do not output commands to select table access methods. With this option, all objects will be created with whichever table access method is the default during restore.
  - No Tablespaces ([--no-tablespaces])
    Do not output commands to create tablespaces nor select tablespaces for objects. With this option, all objects will be created in whichever tablespace is the default during restore.
  - No Toast Compression ([--no-toast-compression])
    Do not output commands to set TOAST compression methods. With this option, all columns will be restored with the default compression setting.
  - No unlogged Table Data ([--no-unlogged-table-data])
    Do not dump the contents of unlogged tables. This option has no effect on whether or not the table definitions (schema) are dumped; it only suppresses dumping the table data.
  - Quote All Identifiers ([--quote-all-identifiers])
    Force quoting of all identifiers. This option is recommended when dumping a database from a server whose PostgreSQL major version is different from pg_dumpall's, or when the output is intended to be loaded into a server of a different major version. By default, pg_dumpall quotes only identifiers that are reserved words in its own major version. This sometimes results in compatibility issues when dealing with servers of other versions that may have slightly different sets of reserved words. Using --quote-all-identifiers prevents such issues, at the price of a harder-to-read dump script.


--use-set-session-authorization
Output SQL-standard SET SESSION AUTHORIZATION commands instead of ALTER OWNER commands to determine object ownership. This makes the dump more standards compatible, but depending on the history of the objects in the dump, might not restore properly.	


--lock-wait-timeout=timeout
Do not wait forever to acquire shared table locks at the beginning of the dump. Instead, fail if unable to lock a table within the specified timeout. The timeout may be specified in any of the formats accepted by SET statement_timeout.

--no-sync
By default, pg_dumpall will wait for all files to be written safely to disk. This option causes pg_dumpall to return without waiting, which is faster, but means that a subsequent operating system crash can leave the dump corrupt. Generally, this option is useful for testing but should not be used when dumping data from production installation.
