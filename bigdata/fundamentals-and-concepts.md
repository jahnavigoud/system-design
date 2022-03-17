# Fundamentals and Concepts

## Data Encoding and Schema Evolution

1. [How Protobuf and Avro and Thrift allow schema evolution](https://martin.kleppmann.com/2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html)
    - Avro, follows or forces you to follow stricter guidelines in terms of schema evolution.
    - Protobuf has lax conditions and it can allow schema evolution without enforcing any constraints
    so its upto users to make sure its followed.
    - Use Protobuf, if you have a single service and want to service data between them.
    - If there is no dependency on downstream to be on same version  
2. [How Protobuf actually encodes the data - Internal working](https://medium.com/@yashschandra/an-inner-view-to-protobuf-encoding-e668f37847d5#)

### Advantages of Protocol Buffers - Comparison with JSON and not avro

- String representations of data:
- Require text encode/decode (which can be cheap, but is still an extra step)
- Requires complex parse code, especially if there are human-friendly rules like "must allow whitespace"
- Usually involves more bandwidth - so more actual payload to churn - due to embedding of things like names,and (again) having to deal with human-friendly representations (how to tokenize the syntax, for example)
- Often requires lots of intermediate string instances that are used for member-lookups etc
- Both text-based and binary-based serializers can be fast and efficient (or slow and horrible)... just: binary serializers have the scales tipped in their advantage. This means that a "good" binary serializer will usually be faster than a "good" text-based serializer.

Let's compare a basic example of an integer:

**json:**
`{"id":42}`
9 bytes if we assume ASCII or UTF-8 encoding and no whitespace.

**xml:**
`<id>42</id>`
11 bytes if we assume ASCII or UTF-8 encoding and no whitespace - and no namespace noise like namespaces.

**protobuf:**
`0x08 0x2a`
2 bytes

Now imagine writing a general purpose xml or json parser, and all the ambiguities and scenarios you need to handle just at the text layer, then you need to map the text token "id" to a member, then you need to do an integer parse on "42". In protobuf, the payload is smaller, plus the math is simple, and the member-lookup is an integer (so: suitable for a very fast switch/jump).

## Bucketing and partitioning
Partitioning data is often used for distributing load horizontally, this has performance benefit,
and helps in organizing data in a logical fashion. Example: if we are dealing with a large `employee` table
and often run queries with `WHERE` clauses that restrict the results to a particular country or department .
For a faster query response Hive table can be `PARTITIONED BY (country STRING, DEPT STRING)`.
Partitioning tables changes how Hive structures the data storage and Hive will now create subdirectories reflecting the partitioning structure like

>../employees/country=ABC/DEPT=XYZ.

If query limits for `employee from country=ABC`,
it will only scan the contents of one directory `country=ABC`. This can dramatically improve query performance,
but only if the partitioning scheme reflects common filtering. Partitioning feature is very useful in Hive,
however, a design that creates too many partitions may optimize some queries, but be detrimental for other important queries.
Other drawback is having too many partitions is the large number of Hadoop files and directories that are created
unnecessarily and overhead to NameNode since it must keep all metadata for the file system in memory.

Bucketing is another technique for decomposing data sets into more manageable parts.
For example, suppose a table using date as the top-level partition and `employee_id` as the second-level
partition leads to too many small partitions. Instead, if we bucket the employee table and use `employee_id`
as the bucketing column, the value of this column will be hashed by a user-defined number into buckets. 
Records with the same `employee_id` will always be stored in the same bucket. Assuming the number of employee_id is
much greater than the number of buckets, each bucket will have many `employee_id`. While creating table you can specify
like `CLUSTERED BY (employee_id) INTO XX  BUCKETS`; where `XX` is the number of buckets . Bucketing has several advantages.
The number of buckets is fixed so it does not fluctuate with data. If two tables are bucketed by `employee_id`, Hive can create
a logically correct sampling. Bucketing also aids in doing efficient map-side joins etc.

### Static partitions and dynamic partitions

[How bucketing reduces shuffling? ByteDance](https://databricks.com/session_na20/bucketing-2-0-improve-spark-sql-performance-by-removing-shuffle)


### Bloom filter and how it improves the performance
[What is bloom filter](http://hadoopnalgos.blogspot.com/2017/02/bloom-filter.html)
[Addvantages of bloom filter](https://stackoverflow.com/questions/4282375/what-is-the-advantage-to-using-bloom-filters)
TL;DR - Used to check if element is definitely not there.
[Online bloom filter playground](https://llimllib.github.io/bloomfilter-tutorial/)
```text
A false positive means that the results say you have the condition you were tested for, 
but you really don't. With a false negative, the results say you don't have a condition, but you really do.
```


## Isolation levels
- Serializable: The strongest isolation level. It ensures that committed write operations and all reads are Serializable. Operations are allowed as long as there exists a serial sequence of executing them one-at-a-time that generates the same outcome as that seen in the table. For the write operations, the serial sequence is exactly the same as that seen in the table’s history.
- WriteSerializable: A weaker isolation level than Serializable. It ensures only that the write operations (that is, not reads) are serializable. However, this is still stronger than Snapshot isolation. WriteSerializable is the default isolation level because it provides great balance of data consistency and availability for most common operations.
- Snapshot serialization: In databases, and transaction processing (transaction management), snapshot isolation is a guarantee that all reads made in a transaction will see a consistent snapshot of the database (in practice it reads the last committed values that existed at the time it started), and the transaction itself will successfully commit only if no updates it has made conflict with any concurrent updates made since that snapshot.
[Isolation levels](https://docs.databricks.com/delta/optimizations/isolation-level.html)
    
### Data Skipping and Z-ordering

Z-ordering is an I/O pruning operation.

- [Data Skipping and Z-ordering in Delta lake](https://engineering.salesforce.com/boost-delta-lake-performance-with-data-skipping-and-z-order-75c7e6c59133)
- [Databricks optimization technique](https://docs.databricks.com/delta/optimizations/file-mgmt.html#compaction-bin-packing)

    

## Apache Hudi / Delta Lake / Apache Iceberg.
[Comparison of Apache Hudi vs Apache Iceberg vs Delta](https://eric-sun.medium.com/rescue-to-distributed-file-system-2dd8abd5d80d)
 - [Databricks analysis on the above (slides)](https://www.slideshare.net/databricks/a-thorough-comparison-of-delta-lake-iceberg-and-hudi)
 - [Databricks video and transcript](https://databricks.com/session_na20/a-thorough-comparison-of-delta-lake-iceberg-and-hudi)
 
### Features that we expect Data Lake's to have

1. Transaction or ACID ability.
2. Time travel, concurrence read, and write.
3. Support for both batch and streaming
4. data could be late in streaming, thus we need to have a mechanism like data mutation and data correction which would allow the right data to merge into the base dataset, and the correct base dataset to follow for the business view of the report for end-user.
5. As the table made changes around with the business over time. So we also expect that data lake to have features like Schema Evolution and Schema Enforcements, which could update a Schema over time.
6. Data Lake is, independent of the engines and the underlying storage's.

#### Delta Lake
- [How traansactions work in delta lake](https://databricks.com/blog/2019/08/21/diving-into-delta-lake-unpacking-the-transaction-log.html)
i. Dealing with Multiple Concurrent Reads and Writes, uses optimistic concurrency control.
ii. For Solving Conflicts Optimistically, uses  mutual exclusion.
- [How schema validation and schema evolution works](https://databricks.com/blog/2019/09/24/diving-into-delta-lake-schema-enforcement-evolution.html)
- [Update/Merge/Delete](https://databricks.com/blog/2020/09/29/diving-into-delta-lake-dml-internals-update-delete-merge.html)
  - Scan for records , select those files, update those file, old files are tombstone.
  - TODO : need to understand all performance tuning in this case.
  
- [Optimize File Management (compaction)](https://docs.databricks.com/delta/optimizations/file-mgmt.html#compaction-bin-packing)
- [Data Skipping and Z-ordering](https://engineering.salesforce.com/boost-delta-lake-performance-with-data-skipping-and-z-order-75c7e6c59133)

  
#### Apache Hudi
(TODO)
#### Apache Iceberg
(TODO)

### Natural Keys vs Synthetic Keys.


## Apache Kafka
[Basics of kafka ecosystem]()

[Apache Kafka Schema management](https://docs.confluent.io/platform/current/schema-registry/index.html#)
[Kafka avro vs Kafka proto](https://simon-aubury.medium.com/kafka-with-avro-vs-kafka-with-protobuf-vs-kafka-with-json-schema-667494cbb2af)

[Kafka Exactly once (Effectively once), Atleast once](https://medium.com/@andy.bryant/processing-guarantees-in-kafka-12dd2e30be0e)
[Original confluent blog for kafka exactly once](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)

[How to achieve strict ordering in kafka](https://www.cloudkarafka.com/blog/2018-08-21-faq-apache-kafka-strict-ordering.html)

[How to Read topic from beginning](https://riptutorial.com/apache-kafka/example/19390/how-can-i-read-topic-from-its-beginning)

[Kafka storage internals](https://medium.com/@durgaswaroop/a-practical-introduction-to-kafka-storage-internals-d5b544f6925f#:~:text=To%20confirm%20that%20the%20messages,what's%20inside%20that%20log%20file.&text=Kafka%20stores%20all%20the%20messages,also%20called%20as%20the%20Offset%20.)

[Disaster recovery for kafka multi-region uber](https://eng.uber.com/kafka/) todo

[Read data from closest replica for reducing the latency](https://developers.redhat.com/blog/2020/04/29/consuming-messages-from-closest-replicas-in-apache-kafka-2-4-0-and-amq-streams/)
[Proposal for closest replica changes](https://cwiki.apache.org/confluence/display/KAFKA/KIP-392%3A+Allow+consumers+to+fetch+from+closest+replica)


## Apache Kinesis
[Medium blog on Kinesis](https://medium.com/@yashbindlish1/amazon-kinesis-the-core-of-real-time-streaming-a543085a212f)


### Kinesis vs Kafka
[Benchmarking Kinesis vs Kafka](https://medium.com/flo-engineering/kinesis-vs-kafka-6709c968813)
[Kinesis vs Kafka Analysis](https://medium.com/softkraft/aws-kinesis-vs-kafka-comparison-which-is-right-for-you-8e81374d8166)


### What is compaction 
[Compaction in HDFS](https://medium.com/datakaresolutions/compaction-in-hive-97a1d072400f)

## Good reads
[Why GCP is better than AWS?](https://nandovillalba.medium.com/why-i-think-gcp-is-better-than-aws-ea78f9975bda)

## Data modelling 
[Parent child relationship in data modelling using Avro](https://www.linkedin.com/pulse/parent-child-relationships-you-joshua-hartman/)

[Avro Schema Composition](https://mykidong.medium.com/howto-implement-avro-schema-inheritance-757d2897c1ad)

[Data Governance using Avro and Kafka](https://medium.com/@sderosiaux/governing-data-with-kafka-avro-ecfb665f266c)


## Avro vs Parquet
### COMPARISONS BETWEEN DIFFERENT FILE FORMATS

**AVRO vs PARQUET**
- AVRO is a row-based storage format whereas PARQUET is a columnar based storage format.
- PARQUET is much better for analytical querying i.e. reads and querying are much more efficient than writing.
- Write operations in AVRO are better than in PARQUET.
- AVRO is much matured than PARQUET when it comes to schema evolution. PARQUET only supports schema append whereas AVRO supports a much-featured schema evolution i.e. adding or modifying columns.
- PARQUET is ideal for querying a subset of columns in a multi-column table. AVRO is ideal in case of ETL operations where we need to query all the columns.

**ORC vs PARQUET**
- PARQUET is more capable of storing nested data.
- ORC is more capable of Predicate Pushdown.
- ORC supports ACID properties.
- ORC is more compression efficient.


[Introduction to columnar formats in spark and hadoop](https://blog.matthewrathbone.com/2019/11/21/guide-to-columnar-file-formats.html)
[Intro to hive file format (paid medium)](https://towardsdatascience.com/new-in-hadoop-you-should-know-the-various-file-format-in-hadoop-4fcdfa25d42b)
[Simple high level diff - Avro,ORC,Parquet](https://towardsdatascience.com/demystify-hadoop-data-formats-avro-orc-and-parquet-e428709cf3bb)
[Medium level explanation of all formats](https://towardsdatascience.com/new-in-hadoop-you-should-know-the-various-file-format-in-hadoop-4fcdfa25d42b)
[Medium level explanation of different file formats](https://blog.clairvoyantsoft.com/big-data-file-formats-3fb659903271)

[ORC performance and compression > parquet](https://blog.cloudera.com/orcfile-in-hdp-2-better-compression-better-performance/)

- Spark's optimized for parquet, hive's optimized for orc file format.
- Parquet is implemented using the google dremel paper. Hence, parquet is intended for complex data structure types.(tree like)
- Many of the performance improvements provided in the Stinger initiative are dependent on features of the ORC format including block level index for each column.
  This leads to potentially more efficient I/O allowing Hive to skip reading entire blocks of data if it determines predicate values are not present there.
  Also the Cost Based Optimizer has the ability to consider column level metadata present in ORC files in order to generate the most efficient graph.
[Hive 3.0 ACID compliance only with ORC](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/using-hiveql/content/hive_hive_3_tables.html)

[How compression effects file splitting](http://comphadoop.weebly.com/)

### Avro examples
```json
{
  "type": "record",
  "name": "Address",
  "namespace": "com.company.base.Address",
  "fields": [
    {
      "name": "streetaddress",
      "type": "string"
    },
    {
      "name": "city",
      "type": "string"
    }
  ]
}
```
```json
{
	"name": "traffic_events",
	"type": "record",
	"namespace": "ccom.company.base.traffic_events",
	"fields": [{
			"name": "offerRequestId",
			"type": "string",
			"dot_name": "traffic_events.offerRequestId",
			"ccpa": [
				"orid"
			],
			"is_required": "true",
			"flat_map_source": "offerRequestId"
		},
		{
			"name": "eventType",
			"type": "string",
			"default": "HomeDetailsEvent",
			"dot_name": "traffic_events.eventType",
			"ccpa": [],
			"is_required": "true",
			"flat_map_source": "eventType"
		},
        {
        			"name": "address",
        			"type": "Address",
        			"default": "{}",
        			"dot_name": "traffic_events.address",
        			"ccpa": [],
        			"is_required": "true",
        			"flat_map_source": "address"
        }
	]
}
```
    
### Avro tools and code

### Proto examples 
```proto
syntax = "proto2";

import "google/protobuf/timestamp.proto";
import "shared.proto";
import "godaddy.proto";
import "journal.proto";
import "json.proto";
import "currency.proto";
import "microcents.proto";
import "core/bills/bill.proto";


package ecomm.core.bills;

option java_package = "com.company.project.models.ecomm.bills";
option java_multiple_files = true;

message SellerAccountUri {
    option (shared.type_wrapper)=true;
    optional string value = 1;
}

message PrimaryPayment {
    optional BillUri payment_instrument_uri = 1 [(json.path) = "paymentInstrumentUri"];
    optional BillUri payment_uri = 2 [(json.path) = "paymentUri"];
    optional string authorization_id = 3 [(json.path) = "authorizationId"];
    optional Microcents amount = 4 [(json.path) = "amount"];
    optional string displayed_merchant_country = 5 [(json.path) = "displayedMerchantCountry"];
    optional int32 response_code = 6 [(json.path) = "responseCode"];
    optional int32 reason_code = 7 [(json.path) = "reasonCode"];
}

message BillSchematized {
    option (journal.realtime_stream) = "journal-Bills-v3";
    option (journal.s3_storage_prefix) = "journal-Bills-v3";
    option (journal.storage_region) = "us-west-2";

    optional BillUri uri = 1 [(json.path) = "uri"];
    optional BillId bill_id = 2 [(json.path) = "billId"];
    optional int64 revision = 3 [(json.path) = "metadata.revision"];
    optional godaddy.CustomerId customer_id = 4 [(json.path) = "customerId"];
    optional Currency currency = 5 [(json.path) = "currency"];
    optional BillStatus status = 6 [(json.path) = "status"];
    optional BillTransactionType transaction_type = 7 [(json.path) = "transactionType"];
    optional PricingTotalFull pricing_total = 8 [(json.path) = "pricingTotal"];
    optional google.protobuf.Timestamp created_at = 9 [(json.path) = "metadata.createdAt"];
    optional google.protobuf.Timestamp modified_at = 10 [(json.path) = "metadata.modifiedAt"];
    optional google.protobuf.Timestamp status_updated_at = 11 [(json.path) = "statusUpdatedAt"];
    optional FriendlyId friendly_id = 12 [(json.path) = "friendlyId"];
    optional BillUri updated_uri = 13 [(json.path) = "updatedUri"];
    optional string source = 14 [(json.path) = "source"];
    optional string transaction_id = 15 [(json.path) = "transactionId"];
    optional BillUri original_uri = 16 [(json.path) = "originalUri"];
    optional SellerAccountUri seller_account_uri = 17 [(json.path) = "sellerAccountUri"];
    optional string customer_ip_address = 18 [(json.path) = "customerIpAddress"];
    optional PrimaryPayment primary_payment = 19 [(json.path) = "primaryPayment"];
    repeated Items first_level = 20 [(json.path) = "items[]"];
    repeated Items second_level = 21 [(json.path) = "items[].item.items[]"];
    repeated Items third_level = 22 [(json.path) = "items[].item.items[].item.items[]"];
    optional Microcents discount = 23 [(json.path) = "discount"];
    optional bool test_transaction = 24 [(json.path) = "testTransaction"];
}

message Item {
    optional string originalKey = 1 [(json.path) = "originalKey"];
    optional string source = 2 [(json.path) = "source"];
    optional string item_source = 3 [(json.path) = "itemSource"];
    optional string pay_by = 4 [(json.path) = "payBy"];
    optional string sku = 5 [(json.path) = "sku"];
    optional string description = 6 [(json.path) = "description"];
    optional int32 quantity = 7 [(json.path) = "quantity"];
    optional Microcents price = 8 [(json.path) = "price"];
    optional Microcents shipping = 9 [(json.path) = "shipping"];
    optional TaxesAndFees taxes_and_fees = 10 [(json.path) = "taxesAndFees"];
    optional ProfitCenterData profit_center_data = 11 [(json.path) = "profitCenterData"];
    optional json.JsonNode data = 12 [(json.path) = "data"];
    optional PricingTotalFull pricing_total = 13 [(json.path) = "pricingTotal"];
}

message Items {
    optional string key = 1 [(json.path) = "key"];
    optional Item item = 2 [(json.path) = "item"];
}

message TaxesAndFees {
    optional Microcents taxable_amount = 1 [(json.path) = "taxableAmount"];
    optional Microcents tax_total = 2 [(json.path) = "taxTotal"];
    optional Microcents fee_total = 3 [(json.path) = "feeTotal"];
    optional Taxes taxes = 4 [(json.path) = "taxes"];
    optional Fees fees = 5 [(json.path) = "fees"];
}

message Taxes {
    optional Microcents tax = 1 [(json.path) = "tax"];
    optional int32 tax_code = 2 [(json.path) = "taxCode"];
    optional string jurisdiction_code = 3 [(json.path) = "jurisdictionCode"];
    optional string tax_system = 4 [(json.path) = "taxSystem"];
}

message Fees {
    optional string name = 1 [(json.path) = "name"];
    optional Microcents fee = 2 [(json.path) = "fee"];
}

message ProfitCenterData {
    optional string profit_center_id = 1 [(json.path) = "profitCenterId"];
    optional Cost cost = 2 [(json.path) = "cost"];
    optional Microcents msrp = 3 [(json.path) = "msrp"];
    optional Microcents revenue = 4 [(json.path) = "revenue"];
}

message Cost {
    optional Microcents amount = 1 [(json.path) = "amount"];
    optional core.Currency currency = 2 [(json.path) = "currency"];
}

message PricingTotalFull {
    optional Microcents tax_total = 1 [(json.path) = "taxTotal"];
    optional Microcents shipping = 2 [(json.path) = "shipping"];
    optional Microcents sub_total = 3 [(json.path) = "subTotal"];
    optional Microcents total = 4 [(json.path) = "total"];
    optional Microcents fee_total = 5 [(json.path) = "feeTotal"];
}

```

### Proto tools and code
        
## Data Modelling interview questions
[Interview questions](https://www.guru99.com/data-modeling-interview-questions-answers.html)
[Star schema vs Snowflake schema](https://www.guru99.com/star-snowflake-data-warehousing.html)

Dimension - Dimensions represent qualitative data. For example, product, class, plan, etc.
A dimension table has textual or descriptive attributes. For example, the product category and product name
are two attributes of the product dimension table.

Fact / Fact Table - The fact represents quantitative data. For example, the net amount which is due.
A fact table contains numerical data as well as foreign keys from dimensional tables.

[Types of Normalization](https://www.sqlshack.com/what-is-database-normalization-in-sql-server/)

Types of normalizations are: 

1) first normal form
2) second normal form
3) third normal forms
4) boyce-codd fourth
5) fifth normal forms.            




## SQL Basics
[SQL Window functions](https://www.toptal.com/sql/intro-to-sql-windows-functions)

### Over function
The OVER clause is what specifies a window function and must always be included in the statement.
The default in an OVER clause is the entire rowset. As an example, let’s look at an employee table in a company database
and show the total number of employees on each row, along with each employee’s info, including when they started with the company.

```sql
SELECT COUNT(*) OVER() As NumEmployees, firstname, lastname, date_started
FROM Employee
ORDER BY date_started;
```
But now, let’s say we wish to show the number of employees who started in the same month as the employee in the row.
We will need to narrow or restrict the count to just that month for each row. How is that done?
We use the window `PARTITION` clause, like so:

```sql
SELECT COUNT(*) OVER (PARTITION BY MONTH(date_started),YEAR(date_started)) 
As NumPerMonth, 
DATENAME(month,date_started)+' '+DATENAME(year,date_started) As TheMonth,
firstname, lastname
FROM Employee
ORDER BY date_started;
```

let’s say we not only wanted to find out how many employees started in the same month,
but we want to show in which order they started that month.For that, we can use the familiar `ORDER BY` clause.
 However, within a window function, `ORDER BY` acts a bit diffrently than it does at the end of a query.

```sql
SELECT COUNT(*) OVER (PARTITION BY MONTH(date_started), YEAR(date_started) 
ORDER BY date_started) As NumThisMonth,
    DATENAME(month,date_started)+' '+DATENAME(year,date_started) As TheMonth,
    firstname, lastname, date_started
FROM Employee
ORDER BY date_started;
```

### Rank functions
**ROW_NUMBER()**
- Gives a sequential count within a given partition (but with the absence of a partition, it goes through all rows)
**RANK()**
- Gives the rank of each row based on the ORDER BY clause. It shows ties, and then skips the next ranking.
**DENSE_RANK()**
- Also shows ties, but then continues with the next consecutive value as if there were no tie.

```sql
SELECT 
ROW_NUMBER() OVER (ORDER BY YEAR(date_started),MONTH(date_started)) 
As StartingRank,
    RANK() OVER (ORDER BY YEAR(date_started),MONTH(date_started)) As EmployeeRank,
    DENSE_RANK() OVER (ORDER BY YEAR(date_started),MONTH(date_started)) As DenseRank,
    DATENAME(month,date_started)+' '+DATENAME(year,date_started) As TheMonth,
    firstname, lastname, date_started
FROM Employee
ORDER BY date_started;
```

```text
StartingRank	EmployeeRank	DenseRank	TheMonth	        firstname	    lastname	    date_started
1	                1	            1	    January 2019	    John	        Smith	        2019-01-01
2	                2	            2	    February 2019	    Sally	        Jones	        2019-02-15
3	                2	            2	    February 2019	    Sam	            Gordon	        2019-02-18
4	                4	            3	    March 2019	        Julie	        Sanchez	        2019-03-19                      
```
## Rows and Range

```sql
SELECT OrderYear, OrderMonth, TotalDue,
    SUM(TotalDue) OVER(ORDER BY OrderYear, OrderMonth ROWS BETWEEN 
UNBOUNDED PRECEDING AND 1 PRECEDING) AS 'LaggingRunningTotal'
FROM sales_products;
```

```sql
SELECT OrderYear, OrderMonth, TotalDue,
    SUM(TotalDue) OVER(ORDER BY OrderYear, OrderMonth ROWS BETWEEN 
UNBOUNDED PRECEDING AND 1 PRECEDING) AS 'LaggingRunningTotal'
FROM sales_products;
```
Range will include those rows in the window frame which have the same ORDER BY values as the current row.
Thus, it’s possible that you can get duplicates with RANGE if the ORDER BY is not unique.

Available window options
```text
BETWEEN
{UNBOUNDED PRECEDING | offset { PRECEDING | FOLLOWING }
| CURRENT ROW}
AND
{UNBOUNDED FOLLOWING | offset { PRECEDING | FOLLOWING }
| CURRENT ROW}
```

For example, let’s say you want to show on each row a sales figure for the current month,
and the difference between last month’s sale figure:

```sql
SELECT id, OrderMonth, OrderYear, product, sales, 
sales - LAG(sales,1) OVER (PARTITION BY product ORDER BY OrderYear, OrderMonth) As sales_change
FROM sales_products
WHERE sale_year = 2019;
```