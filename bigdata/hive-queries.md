
# Hive Queries

## Partitioning

The Hive command for Partitioning is:
```hiveql
CREATE TABLE table_name (column1 data_type, column2 data_type) PARTITIONED BY (partition1 data_type, partition2 data_type);
```

## Bucketing

The Hive command for Bucketing is:
```hiveql
CREATE TABLE table_name PARTITIONED BY (partition1 data_type, partition2 data_type) CLUSTERED BY (column_name1, column_name2, …) SORTED BY (column_name [ASC|DESC])] INTO num_buckets BUCKETS;
```

