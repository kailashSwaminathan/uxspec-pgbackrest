# Specification of pg_dump

pg_dump is a utility for backing up a PostgreSQL database

  - concurrent backup
    Make consistent backups even if the database is being used concurrently
   
  - Doesnt block other users accessing the database (writers and readers)

  - What can be backup-ed
    - A single database in a cluster
  
  - What cannot be backup-ed
    - Global objects that are common to all databases in a cluster(roles, tablespaces)
	- Entire cluster
	
  - Output format
    - script file (--format=[p | plain])
	  Plain-text files containing the SQL commands required to construct the database to the state it was in at the time it was saved.
	    - To be used by psql tool. 
	- archive file formats (--format=[t | tar])
	  - The archive file formats are designed to be portable across architectures
	  - Must be used with pg_restore to rebuild the database
	- custom format (--format=[c | custom])
	- directory format ((--format=[d | directory])
	  - Allow for selection and reordering of all archived items
	  - supports parallel dump
	  - support parallel restoration
	  - Compressed by default
	  
	- Output
	
	  - Data only
	    Dump only the data, not the schema (data definitions). Table data, large objects and sequence values are dumped
	  - Include large objects
	    Default behaviour
	  - Exclude large objects
	    Output command to DROP all the dumped objects prior to outputting the commands for creating them. This option is useful when the restore is to overwrite an existing database
	  - Create
	    Begin the output with a command to create the database itself and reconnect to the created database. The output also includes the database's comment if any, and any configuration variable
		settings that are specific to this database (ALTER DATABASE ... SET ... / ALTER ROLE ... IN DATABASE ... SET ...). Access privileges for the database itself are also dumped, unless suppressed
	  - Extension(s) matching pattern
	    Dump extensions matching a pattern. By default, all non-system extensions in the target database will be dumped.
		   NOTE
		   No other database objects that the selected extension(s) might depend upon are not dumped. 
	
	- Character Encoding
	  Create the dump in the specific character set encoding.
	  
	- Execution
	  - run parallel jobs ([-j njobs | --jobs=njobs])
	  
	     
	  
