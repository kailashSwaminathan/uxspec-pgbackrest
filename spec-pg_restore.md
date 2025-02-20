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
  - None
    The standard input is used
