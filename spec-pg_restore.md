# Specification of pg_restore 

## Description
***pg_restore*** is a utility for restoring a PostgreSQL database from an archive created by ***pg_dump*** in one of the non-plain-text formats. It will issue the commands necessary to reconstruct the database to the state it was in at the time it was saved. The archive files also allow ***pg_restore*** to be selective about what is restored, or even to reorder the items prior to being restored. The archive files are designed to be portable across architectures.

## Mode of Operation
***pg_restore*** can operate in two modes. 

  - Database name is specified <br>
    ***pg_restore*** connects to that database and restores archive contents directly into the database. 
  - Database name is NOT specified <br>
    A script containing the SQL commands necessary to rebuild the database is created and written to a file or standard output. This script output is equivalent to the plain text output format of ***pg_dump***. Some of the options controlling the output are therefore analogous to ***pg_dump*** options.
	
Obviously, ***pg_restore*** cannot restore information that is not present in the archive file. For instance, if the archive was made using the “dump data as INSERT commands” option, ***pg_restore*** will not be able to load the data using COPY statements.

## Input 
The filename command line argument for ***pg_restore*** can be one of the following
  - Archive file <br>
    Specifies the location of the archive file created by ***pg_dump***.
  - Directory <br>
    Specifies the location of the directory for the directory-format archive created by ***pg_dump***
  - None <br>
    The standard input is used
	
## Selectors
The restoration by ***pg_restore*** is controlled by these options

  - Data Only ([-a | --data-only]) <br>
    Restore only the data, not the schema (data definitions). Table data, large objects, and sequence values are restored, if present in the archive.
	
  - Schema Only ([-s | --schema-only]) <br>
    Restore only the schema (data definitions), not data, to the extent that schema entries are present in the archive. This option is the inverse of --data-only. It is similar to, but for historical reasons not identical to, specifying --section=pre-data --section=post-data. (Do not confuse this with the --schema option, which uses the word “schema” in a different meaning.)
	
  - Clean and Create ([-c | --clean]) <br>
    Before restoring database objects, issue commands to DROP all the objects that will be restored. This option is useful for overwriting an existing database. If any of the objects do not exist in the destination database, ignorable error messages will be reported, unless --if-exists is also specified.
	
	- ([--if-exists])
      Use DROP ... IF EXISTS commands to drop objects in --clean mode. This suppresses “does not exist” errors that might otherwise be reported. This option is not valid unless --clean is also specified.

  - Create ([-C | --create]) <br>
    Create the database before restoring into it. If --clean is also specified, drop and recreate the target database before connecting to it. With --create, ***pg_restore*** also restores the database's comment if any, and any configuration variable settings that are specific to this database, that is, any ALTER DATABASE ... SET ... and ALTER ROLE ... IN DATABASE ... SET ... commands that mention this database. Access privileges for the database itself are also restored, unless --no-acl is specified. When this option is used, the database named with -d is used only to issue the initial DROP DATABASE and CREATE DATABASE commands. All data is restored into the database name that appears in the archive.
	
	-n schema
--schema=schema
Restore only objects that are in the named schema. Multiple schemas may be specified with multiple -n switches. This can be combined with the -t option to restore just a specific table.

-N schema
--exclude-schema=schema
Do not restore objects that are in the named schema. Multiple schemas to be excluded may be specified with multiple -N switches.

When both -n and -N are given for the same schema name, the -N switch wins and the schema is excluded.

-O
--no-owner
Do not output commands to set ownership of objects to match the original database. By default, pg_restore issues ALTER OWNER or SET SESSION AUTHORIZATION statements to set ownership of created schema elements. These statements will fail unless the initial connection to the database is made by a superuser (or the same user that owns all of the objects in the script). With -O, any user name can be used for the initial connection, and this user will own all the created objects.

-P function-name(argtype [, ...])
--function=function-name(argtype [, ...])
Restore the named function only. Be careful to spell the function name and arguments exactly as they appear in the dump file's table of contents. Multiple functions may be specified with multiple -P switches.

-I index
--index=index
Restore definition of named index only. Multiple indexes may be specified with multiple -I switches.


--no-comments
Do not output commands to restore comments, even if the archive contains them.

-t table
--table=table
Restore definition and/or data of only the named table. For this purpose, “table” includes views, materialized views, sequences, and foreign tables. Multiple tables can be selected by writing multiple -t switches. This option can be combined with the -n option to specify table(s) in a particular schema.

Note
When -t is specified, pg_restore makes no attempt to restore any other database objects that the selected table(s) might depend upon. Therefore, there is no guarantee that a specific-table restore into a clean database will succeed.

Note
This flag does not behave identically to the -t flag of pg_dump. There is not currently any provision for wild-card matching in pg_restore, nor can you include a schema name within its -t. And, while pg_dump's -t flag will also dump subsidiary objects (such as indexes) of the selected table(s), pg_restore's -t flag does not include such subsidiary objects.

Note
In versions prior to PostgreSQL 9.6, this flag matched only tables, not any other type of relation.

-T trigger
--trigger=trigger
Restore named trigger only. Multiple triggers may be specified with multiple -T switches.


-l
--list
List the table of contents of the archive. The output of this operation can be used as input to the -L option. Note that if filtering switches such as -n or -t are used with -l, they will restrict the items listed.

-L list-file
--use-list=list-file
Restore only those archive elements that are listed in list-file, and restore them in the order they appear in the file. Note that if filtering switches such as -n or -t are used with -L, they will further restrict the items restored.

list-file is normally created by editing the output of a previous -l operation. Lines can be moved or removed, and can also be commented out by placing a semicolon (;) at the start of the line. See below for examples.



-S username
--superuser=username
Specify the superuser user name to use when disabling triggers. This is relevant only if --disable-triggers is used.


-v
--verbose
Specifies verbose mode. This will cause pg_restore to output detailed object comments and start/stop times to the output file, and progress messages to standard error. Repeating the option causes additional debug-level messages to appear on standard error.

-V
--version
Print the pg_restore version and exit.

-x
--no-privileges
--no-acl
Prevent restoration of access privileges (grant/revoke commands).

-1
--single-transaction
Execute the restore as a single transaction (that is, wrap the emitted commands in BEGIN/COMMIT). This ensures that either all the commands complete successfully, or no changes are applied. This option implies --exit-on-error.

--disable-triggers
This option is relevant only when performing a data-only restore. It instructs pg_restore to execute commands to temporarily disable triggers on the target tables while the data is restored. Use this if you have referential integrity checks or other triggers on the tables that you do not want to invoke during data restore.

Presently, the commands emitted for --disable-triggers must be done as superuser. So you should also specify a superuser name with -S or, preferably, run pg_restore as a PostgreSQL superuser.

--enable-row-security
This option is relevant only when restoring the contents of a table which has row security. By default, pg_restore will set row_security to off, to ensure that all data is restored in to the table. If the user does not have sufficient privileges to bypass row security, then an error is thrown. This parameter instructs pg_restore to set row_security to on instead, allowing the user to attempt to restore the contents of the table with row security enabled. This might still fail if the user does not have the right to insert the rows from the dump into the table.

Note that this option currently also requires the dump be in INSERT format, as COPY FROM does not support row security.



--no-data-for-failed-tables
By default, table data is restored even if the creation command for the table failed (e.g., because it already exists). With this option, data for such a table is skipped. This behavior is useful if the target database already contains the desired table contents. For example, auxiliary tables for PostgreSQL extensions such as PostGIS might already be loaded in the target database; specifying this option prevents duplicate or obsolete data from being loaded into them.

This option is effective only when restoring directly into a database, not when producing SQL script output.

--no-publications
Do not output commands to restore publications, even if the archive contains them.

--no-security-labels
Do not output commands to restore security labels, even if the archive contains them.

--no-subscriptions
Do not output commands to restore subscriptions, even if the archive contains them.

--no-table-access-method
Do not output commands to select table access methods. With this option, all objects will be created with whichever table access method is the default during restore.

--no-tablespaces
Do not output commands to select tablespaces. With this option, all objects will be created in whichever tablespace is the default during restore.

--section=sectionname
Only restore the named section. The section name can be pre-data, data, or post-data. This option can be specified more than once to select multiple sections. The default is to restore all sections.

The data section contains actual table data as well as large-object definitions. Post-data items consist of definitions of indexes, triggers, rules and constraints other than validated check constraints. Pre-data items consist of all other data definition items.

--strict-names
Require that each schema (-n/--schema) and table (-t/--table) qualifier match at least one schema/table in the backup file.

--transaction-size=N
Execute the restore as a series of transactions, each processing up to N database objects. This option implies --exit-on-error.

--transaction-size offers an intermediate choice between the default behavior (one transaction per SQL command) and -1/--single-transaction (one transaction for all restored objects). While --single-transaction has the least overhead, it may be impractical for large databases because the transaction will take a lock on each restored object, possibly exhausting the server's lock table space. Using --transaction-size with a size of a few thousand objects offers nearly the same performance benefits while capping the amount of lock table space needed.

--use-set-session-authorization
Output SQL-standard SET SESSION AUTHORIZATION commands instead of ALTER OWNER commands to determine object ownership. This makes the dump more standards-compatible, but depending on the history of the objects in the dump, might not restore properly.

-?
--help
Show help about pg_restore command line arguments, and exit.

pg_restore also accepts the following command line arguments for connection parameters:

-h host
--host=host
Specifies the host name of the machine on which the server is running. If the value begins with a slash, it is used as the directory for the Unix domain socket. The default is taken from the PGHOST environment variable, if set, else a Unix domain socket connection is attempted.

-p port
--port=port
Specifies the TCP port or local Unix domain socket file extension on which the server is listening for connections. Defaults to the PGPORT environment variable, if set, or a compiled-in default.

-U username
--username=username
User name to connect as.

-w
--no-password
Never issue a password prompt. If the server requires password authentication and a password is not available by other means such as a .pgpass file, the connection attempt will fail. This option can be useful in batch jobs and scripts where no user is present to enter a password.

-W
--password
Force pg_restore to prompt for a password before connecting to a database.

This option is never essential, since pg_restore will automatically prompt for a password if the server demands password authentication. However, pg_restore will waste a connection attempt finding out that the server wants a password. In some cases it is worth typing -W to avoid the extra connection attempt.

--role=rolename
Specifies a role name to be used to perform the restore. This option causes pg_restore to issue a SET ROLE rolename command after connecting to the database. It is useful when the authenticated user (specified by -U) lacks privileges needed by pg_restore, but can switch to a role with the required rights. Some installations have a policy against logging in directly as a superuser, and use of this option allows restores to be performed without violating the policy.

-d dbname
--dbname=dbname
Connect to database dbname and restore directly into the database. The dbname can be a connection string. If so, connection string parameters will override any conflicting command line options.

-e
--exit-on-error
Exit if an error is encountered while sending SQL commands to the database. The default is to continue and to display a count of errors at the end of the restoration.

-f filename
--file=filename
Specify output file for generated script, or for the listing when used with -l. Use - for stdout.

--filter=filename
Specify a filename from which to read patterns for objects excluded or included from restore. The patterns are interpreted according to the same rules as -n/--schema for including objects in schemas, -N/--exclude-schemafor excluding objects in schemas, -P/--function for restoring named functions, -I/--index for restoring named indexes, -t/--table for restoring named tables or -T/--trigger for restoring triggers. To read from STDIN, use - as the filename. The --filter option can be specified in conjunction with the above listed options for including or excluding objects, and can also be specified more than once for multiple filter files.

The file lists one database pattern per row, with the following format:

{ include | exclude } { function | index | schema | table | trigger } PATTERN
The first keyword specifies whether the objects matched by the pattern are to be included or excluded. The second keyword specifies the type of object to be filtered using the pattern:

function: functions, works like the -P/--function option. This keyword can only be used with the include keyword.

index: indexes, works like the -I/--indexes option. This keyword can only be used with the include keyword.

schema: schemas, works like the -n/--schema and -N/--exclude-schema options.

table: tables, works like the -t/--table option. This keyword can only be used with the include keyword.

trigger: triggers, works like the -T/--trigger option. This keyword can only be used with the include keyword.

Lines starting with # are considered comments and ignored. Comments can be placed after an object pattern row as well. Blank lines are also ignored. See Patterns for how to perform quoting in patterns.

-F format
--format=format
Specify format of the archive. It is not necessary to specify the format, since pg_restore will determine the format automatically. If specified, it can be one of the following:

c
custom
The archive is in the custom format of pg_dump.

d
directory
The archive is a directory archive.

t
tar
The archive is a tar archive.


-R
--no-reconnect
This option is obsolete but still accepted for backwards compatibility.




## Execution

-j number-of-jobs
--jobs=number-of-jobs
Run the most time-consuming steps of pg_restore — those that load data, create indexes, or create constraints — concurrently, using up to number-of-jobs concurrent sessions. This option can dramatically reduce the time to restore a large database to a server running on a multiprocessor machine. This option is ignored when emitting a script rather than connecting directly to a database server.

Each job is one process or one thread, depending on the operating system, and uses a separate connection to the server.

The optimal value for this option depends on the hardware setup of the server, of the client, and of the network. Factors include the number of CPU cores and the disk setup. A good place to start is the number of CPU cores on the server, but values larger than that can also lead to faster restore times in many cases. Of course, values that are too high will lead to decreased performance because of thrashing.

Only the custom and directory archive formats are supported with this option. The input must be a regular file or directory (not, for example, a pipe or standard input). Also, multiple jobs cannot be used together with the option --single-transaction.
