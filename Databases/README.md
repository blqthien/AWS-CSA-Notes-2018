# Databases

## Types of Databases

### Relational Datbases

Relational databases are what most of us are all used to. They have been around since the 70's and you can think about them like spreadsheets!

- Database
- Tables
- Columns
- Rows

| id        | name    | age    | location   |
| --------- |:-------:| :-----:| :--------:  |
| 1         | nigel   | 30     | San Diego  |
| 2         | jim     | 28     | NYC        |
| 3         | betty   | 31     | San Francisco|

**_Relational Databases Examples_**

- SQL Server
- Oracle
- MySQL
- PostgreSQL
- Aurora
- MariaDB

### Non-Relational (NoSQL)

- Database
  - Collection => Table
  - Document => Row
  - Key, Value Pairs => Columns

**_Non Relational Databases Examples_**

```json
{
  "_id": "394ejojaj903091881dnna",
  "name": "nigel",
  "age": 30,
  "location": "San Diego"
}
```

### Data Warehousing

Used for business intelligence. Tools like Cognos, Jaspersoft, SQL Server, Reporting Services, Oracle Hyperion, SAP NetWeaver.

Used to pull in very large and complex data sets. Usually used by management to do queries on data (such as current performance vs targets etc).

### OLTP (Online Transaction Processing) vs. OLAP (Online Analytics Processing)

OTLP differs from OLAP in terms of the types of queries you will run.

**_OLTP Example_**

Used for transactional type queries.

```
Order number: 2120121

Pulls up a row of data such as Name, Date, Address to Deliver to, Delivery Status etc.
```

**_OLAP Example_**

Used for business logic type queries.

```
Net Profit of given product or device
Pulls in large number of records

Sum of products sold in region
Sum of products sold in continent
Unit cost of product in each region
Sales price of each product
Sales price - unit cost
```

Data Warehousing databases use different type of architecture both from a database perspective and infrastructure layer.

### Elasticache

ElastiCache is a web service that makes it easy to deploy, operate and scale an in-memory cache in the cloud. The service improves the performance of web applications by allowing you to retrieve information from fast, managed, in-memory caches, instead of relying entirely on slower disk-based databases.

ElasticCache supports two open-source in-memory caching engines...

1. Memcached
2. Redis

## Backups, Multi-AZ & Read Replicas

### Automated Backups

Automated Backups allow you to recover your database to any point in time within a 'retention period'. The retention period can be between one and 35 days.

Automated Backups will take a full daily snapshot and will also store transaction logs throughout the day.

When you do a recovery, AWS will first choose the most recent daily backup, and then apply transaction logs relevant to that day. This allows you to do a point in time recovery down to a second, within a retention period.

### Database Snapshots

DB Snapshots are done manually (ie they are user initiated) They are stored even after you delete the original RDS instance, unlike automated backups.

### Restoring Backups

Whenever you restore either an Automatic Backup or a manual Snapshot, the restored version of the database will be a new RDS instance with a new DNS endpoint

`original.us-west-1.rds.amazonaws.com`  ->  `restored.eu-west-1.rds.amazonaws.com`

### Encyrption

Encryption at rest is supported for MySQL, Oracle, SQL Server, PostgreSQL, MariaDB & Aurora.

Encryption is done using the AWS Key Management System (KMS) service. Once your RDS instance is encrypted, the data stored at rest in the underlying storage is encrypted, as are its automated backups, read replicas and snapshots.

At the present time, encrypting an existing DB Instance is not supported. To use RDS encryption for an existing database, you must first create a snapshot, make a copy of that snapshot and encrypt the copy.

### Multi-AZ

Multi-AZ allows you to have an exact copy of your production database in another Availability Zone. AWS handles the replication for you, so when your production database is written to, this write will automatically be synchronized to the stand by database.

In the event of planned database maintenance, DB instance failure, or AZ failure, RDS will automatically failover to the standby so that database operations can resume quickly without admin intervention.

**NOTE:** It is not primarily used for improving performance, really only **disaster recovery**. For performance improvement, you need **Read Replicas**

**Multi-AZ Available DBs**

- SQL Server
- Oracle
- MySQL Server
- PostgreSQL
- MariaDB

### Read Replicas

Read replicas allow you to have a read-only copy of your production database. This is achieved by using async replication from the primay RDS instance to the Read Replica. You use Read Replicas primarily for very read-heavy database workloads.

- Used for scaling, not disaster control!
- Must have auto backups turned on in order to deploy a Read Replica
- You can have up to 5 Read Replica copies of any database.
- You can have Read Replicas of Read Replicas _(inception)_ - mindful of latency
- Each Read Replica will have its own DNS end point.
- You can have Read Replicas that have Multi-AZ
- You can create Read Replicas of Mulit-AZ source databases
- Read Replicas can be promoted to be their own databases. This breaks the replication.
- You can have a Read Replica in a second region.

**Read Replica Available DBs**

- MySQL Server
- PostgreSQL
- MariaDB
- Aurora

## DynamoDB

DynamoDB is a fast and flexible NoSQL database service for all applications that need consistent, single-digit millisecond latency at any scale. It is a fully managed db nd supports both document and key-value data models. Its flexible data model and reliable performance make it a great fit for mobile, web, gaming, ad-tech, IoT etc.

- Stored on SSD Storage
- Spread Across **3** geographically distinct data centers

- Eventual Consistent Read (Default)
  - Consistency across all copies of data is usually reached within a second. Repeating a read after a short amount of time should return the updated data. (Best Read Perf.)


- Strongly Consistent Reads
  - A stronly consistent read returns a result that reflects all writes that received a successful response prior to the read.

**NOTE:** Super easy to scale! Push button scaling

### Pricing

Pricing is based on provision throughput capacity

- Write Throughput $0.0065 per hour for every 10 units
- Read Throughput $0.0065 per hour for every 50 units
- Storage costs of $0.25G per month

_Pricing Example:_

```
Constraint: 1 million WRITEs and 1 million READs per day, while storing 3G of data.

First, calculate how many writes and reads per second you need.

1 million evenly spread writes per day is equivalent to 1,000,000 (writes) /24 (hours) / 60 (minutes) / 60 (seconds) = 11.6 writes per second.

-- BREAKDOWN --

DynamoDB WRITE Capacity Unit - 1 per second = 12
DynamoDB READ Capacity Unit - 1 per second = 12

READ Capacity Units - billed in blocks of 50
WRITE Capacity Units - billed in blocks of 10

Calc WRITE Capacity Units = (0.0065 / 10) x 12 x 24 = $0.1872
Calc READ Capacity Units = (0.0065 / 10) x 12 x 24 = $0.0374
```

## Redshift

Amazon Redshift is a fast and powerful, fully managed petabyte-scale data warehouse service in the cloud. 

Customers can start small for just $0.25 per hour with no commitments or upfront costs and scale to a petabyte or more for $1,000 per terabyte per year, less than 1/10 of most data warehousing solutions.

### Configuration

- Single Node (160Gb)
- Multi-Node
  - Leader Node _(manages client connections and receives queries)_
  - Compute Node _(store data and perform queries and computations)_ - Up to 128 Compute Nodes

### Columns

**Columnar Data Storage** - Instead of storing data as rows, Redshift organizes the data by column.

Unlike row-based systems, which are ideal for transaction processing, column-based systems are ideal for data warehousing and analytics, where queries often invovle aggregates performed over large data sets.

Since only the columns involved in the queries are processing and columnar data is stored sequentially on the storage media, column-based systems require far fewer I/Os, greatly improving query performance.