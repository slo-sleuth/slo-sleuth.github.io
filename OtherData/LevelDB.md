# LevelDB

[LevelDB](https://github.com/google/leveldb) is a Google-created, key-value database.  It maps string keys to string values and stores them in byte arrays.  It doesn't have a command line interface, and it is not relational like SQL databases.  It's goal is to be simple and provide fast, sequential reads over the data.  

LevelDB is used in the Chrome browser and in Chrome-based apps/extensions.  While there are programming libraries (e.g. python's [leveldb](https://pypi.org/project/leveldb/) and golang's [goleveldb](https://github.com/syndtr/goleveldb)) and forensic tools (e.g, [hindsight](https://github.com/obsidianforensics/hindsight)) that can access and read the databases, using them alone can leave a lot of information behind.

The database uses [snappy compression](http://google.github.io/snappy/), so key word searches may not result in matches if your tool set doesn't first decompress them.  However, the log files, which contain uncommitted data, are not compressed and it is keyword hits in these files that first brought the database to my attention.

This explanation is primarily designed to help you recognize a LevelDB implementation and explain what you can expect to find in it's various files.

## Structure

The database is not composed of one file, but [many](https://github.com/google/leveldb/blob/master/doc/impl.md).  They work together to provide fault tolerance and create an ordered structure.

- Database Directory
  - Log Files (*.log)
  
    Not to be confused with files named `LOG` or `LOG.old`, these store recent updates to the database in sequential order.   When it reaches a predetermined size, it is converted to a sorted table and a new log is created.  Log files and sorted tables are numbered sequentially when they are created.
    
    _Opening a database causes the log file to be written to sorted table and a new, empty log file to be created_.
  
  - Sorted tables (*.ldb)
  
    The `*.ldb` sorted tables files store records sorted by key.  The entries contain either a key value or a marker that the key has been deleted.  The tables are organized by levels, the newest data being placed in level-0.  When the number of level-0 files reaches a threshold, they are combined with all older data (level-1).  New level-1 files are generated for every 2mb of data.  As the number of level-1 files reaches a threshold, they are combined into a level-2 file of greater size still, and so on.  Sorted tables are automatically compressed with the snappy compression algorithm. 
    
    _Opening a database triggers the `*.log` files to be written to a new `#.ldb` file.  The old log file is then deleted._
  
  - Manifest
  
    A `MANIFEST-######` file contains a list of sorted tables used to make each level.  It includes metadata such as the key ranges.  It uses the same file formatting logs.
    
    _Whenever the database is opened, a new manifest file is created_.  
    
  - CURRENT
  
    The `CURRENT` files is a text file that contains the current manifest in use.
  
  - Info Logs
  
    The `LOG` and `LOG.old` files are text files containing database management information.
  
  - Others
  
    `LOCK` and `*.dbtmp` files may be found in a database directory.  The LOCK file is zero-byte, and I have not encountered the temp files. 

## Analysis

From the structure, one can see that the log files contain recent database changes and the sorted table files contain previous commits to the database.  Opening the database programmatically _will cause changes to these files_ and you may miss valuable data.  

I recently found Discord chat messages in a LevelDB log file, but upon opening the database, the chat content wasn't present.  This means that in the log, the data was added, and then deleted before being committed to sorted tables, and had I only studied the open database data, I would have missed the chat.

That said, there is always value in seeing the data as intended.  Just make sure your are operating on a copy of the data.

### Examining Individual Database Files vs Opening the Database

To collect the most data from a LevelDB, the individual components need to be read and then considered as a whole.  A read of the Info logs will quickly demonstrate how the database is recording and deleting values on an ongoing basis, but it is doing so by writing the changes (additions and deletions) to the log files before committing the changes to the sorted tables.  However, opening the database merges

### Log File Analysis

Log files are not compressed and can be interpreted in a hex editor.  The file format is [partially documented](https://github.com/google/leveldb/blob/master/doc/log_format.md) by Google.

Generally, the data is recorded in the format of:

- Record header (8 bytes)
- Record data (variable)

The documentation clearly defines the record header format, but not the data format.  I have observed a pattern of start-byte (\x01), length, key content, length, value content, but it is a little more complex.

I am presently attempting to create a python tool to extract the contents of a log file.

### Sorted Table Analysis

Because the sorted tables are compressed, you may not get keyword hits.  But unless your tool set decompresses the tables, you might be missing data.  The file format is [documented](https://github.com/google/leveldb/blob/master/doc/table_format.md) by Google.

I plan to follow the log file parsing tool with a sorted table parsing tool.
