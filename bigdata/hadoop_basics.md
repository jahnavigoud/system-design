## Hadoop's basics

**What are the 5 V's in big data**

Volume: Quantity of the data.
Velocity : How much data
Variety : Different types of data.
Veracity : Accuracy of the data.
Value : Importance of the data, vizualization, 

**Types of data**

Structured Data (RDBMS),
Un-structured data (Files, Audio, Video)
Semi-structered (JSON, XML)

**How hadoop differs from traditional system using rdbms?**

RDBMS
ACID properties are not supported (Support available), CRUD 
Suitable for OLTP
Deal with structured data
Reads are fast because schema is already known

**Hadoop is WORM ( Write once Read multiple times)**
Can support all form of data.
Computation speed won't be great with small data.
Since no schema is required for writes, writes are usually faster.



**All the components of Hadoop**
HDFS (Distributed File Storage)
YARN (Cluster resource manager) To schedule MR jobs
NameNode (Stores the metadata) 
DataNode (All the nodes that runs the MR)
NodeManager()
ResourceManager()


**What are the main configuration files.**
hadoop-env.sh (JavaHome, HadoopHome)
core-site.xml 
hdfs-site.xml
yarn-site.xml
mapred-site.xml
masters (secondary name node)
slaves ()



**HDFS assures fault-tolerance?**
Replication assures that HDFS is fault-tolerant.
Block replication is done in different nodes.


**Where is data stored for the NameNode?**
Data is stored in memory in the NameNode.
Metadata is therefore stored in the memory.
RAM is volatile can lose the data any moment? 
At some interval I will take backup of the data in the disk (FsImage)
what at the 23rd hour i lose the data
maintain all the changes in the editlog files which keeps which is the delta for the interval we set.
directory for FsImage and editlog is usually mentioned in the hdfs-site.xml.
NameDirectory -> CurrentDirectory -> SecondaryNameNodeDirectory in the hdfs-site.xml


**What is the problem of having a lot of small files in HDFS? Provide one method to overcome to this problem?**
Small files - too many metadata entries filling up the ram
How to overcome - create a .har file (hadoop archive)
bring all small files together.
hadoop archive -archiveName inputLocation outputLocation


**Suppose there is a file os size 514MB stored in HDFS (Hadoop 2.x) using default block size.**
How many total block created?
default replication factor = 3;
default block mb = 128Mb;


**How to copy file into HDFS with a differecnt block size of that of existing block size configuration?**
hadoop fs -Ddfs.blocksize -copyFromLocal <input> <hdfs_output_path>


**What is a block scanner in HDFS?**
Integrity of data blocks
runs on datanode periodically to check if the data blocks stores are correct or not.


Hadoop Load Balancer - If they are under-replicated it reports to the Namenode.


**Can multiple clients write into HDFS file concurrently.**
Hadoop is Single Write Multiple Read paradigm 
Multiple writes will make file inconsitent.

**What do you mean High-Availability of NameNode? How is achieved?**
Active NameNode, Passive NameNode. 
SecondaryNameNode (Snapshot of NameNode)


MapReduce questions
**Explain the process of spilling in MapReduce?**
Mapper program writes its output to circular memory buffer, which is of 100 mb in size , when this buffer is filled up to its threashold limit it starts emitting the data to files and these files are know as spilled files. By default, it happens when 80% of the buffer space is occupied .

**Difference between blocks, input splits and records? (Important)**
Block is a physical division
Data is stored as blocks in HDFS

**Input splits and records are logical division**
Input split is logical chunks of data to be processed by the mapper phase
Records are each line of the input split is records.


**What is the role of RecordReader in Hadoop MapReduce?**
RecordReader is a parser that parses the input to a key value pair.

**Significance of counters in Mapreduce?**
Helps you to identify statistics.
Application level statistics
All the bad data 
so it is kinda metrics


**Why the o/p of map tasks are stored (spilled) into local disk and not in Hdfs?**
Replicate mapper output if in HDFS which is not necessary. Also it is more expensive because of I/O.

**Define Speculative Execution.?**
If task is running very slow a duplicate equivalent task is launched so as to maintain the critical path of the job.
scheduler monitor all the tasks and launches speculative execution if some tasks are running slower. After the task is completed it 
kills all the duplicates tasks.


**How to prevent a file from splitting if you want it to be processed by the same mapper?**
split size to be larger than the file size
isSpiltable to false 
If only one data node doesn't make sense to split the input file.


**Is it legal to set the number of reducer task to zero? Where is the o/p stored in this case?**
Sqoop doesn't have any reducer.
If you don't need aggregation then you don't need a reducer.
Sqoop is used to copy data from RDBMS to HDFS and vice-versa.
Store to the HDFS you mention.


**Role of Application master in MR job?**
Application masters is checking and deciding the resources needed and sending it to the resource manager.
Inform then node manager with all the details about the job and tells it to run.


**What do you mean by uber mode in map reduce task?**
If the job is small, application master itself will run the tasks in it's JVM.
uber task is a task if it needs less than 10 masters.
requires only one reducer.
input file size is less than the HDFS block size.


**How will you enhance the performance of MR when dealing with too many small files?**
CombineFileInputFormat packs all small files and process by a single mapper.


**Where does the data of hive table gets stored?**
/user/hive/warehouse

**Why HDFS is not used hive metastore for storage (RDBMS)?**
CRUD operations can't be done in HDFS


**What is thrift?**
RPC clients and server communication.


**Suppose you have installed Apache hive on top of hadoop cluster? What will happen if we have multiple clients trying to access hive at the same time?**
Not possible.


**xternal table vs managed table.**
External table - Only metastore gets deleted.
Managed table - metastore and data both deleted


**When to use SORT BY instead of ORDER BY?**
SORTBY - huge data (multiple reducer)
ORDER BY - single reducer.


**What is the difference partition and bucket in hive?**
partition is at the first level (directories)
buckets - sub-partition


**What is dynamic partioning and when is it used?**
Partition the data when loading data into the data. Only during run-time


**How hive distributes the rows into buckets?**
modulo-hash

**When to use buckets when to use partitioning?**
If there are lot of small files in HDFS, hive query performace will degrade, how to better the performance.
Store the files as seq files. Seq files are flat files consisting of binary key value pairs.