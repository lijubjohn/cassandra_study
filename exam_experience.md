
### Cassandra developer associate exam 
- clustering order by  (3-4 q) -> mainly checking do we need to include partition key in the order by clause

- secondary index (3-4 q) -> one question on SASIIndex

- Insert / update queries based on primary key / partition key (5-6 q) -> guess the o/p questions

- chebotko  
  - identify clustering column / primary key ( 2-3 q)
  - based on the diagram tell number of copies of a column ( all tables combined) - (2q)

- Queries
  - Q.
    - query1 -> update on column a,b using primary key  
    - query2 -> insert on column a,b using primary key
      - q : what would the result of the row

  - Q.  
    - query1 -> insert on column a,b using primary key
    - query1 -> insert on column c,d using primary key
      - q : what would the result of the row

- Batch (3-4 q) :

```
BATCH

simple insert;

insert with if not exists clause:

APPLY BATCH
```

- when/why is multi-partition batch used ?

- Static column (2 q) -> easy

- counter column (1 q) -> easy

- why is gossip protocol used -> easy

- multi-data center : one node down : how many replicas available -> easy

- what is multicloud deployment in cassandra ?

- what is hybrid architecture ?

- materialized view  (2-3 q) ->
 - Q.
  - when in materialzed view used
 - Q.
 - based on syntax  (check where condition and null check)

- create secondary index  on table with partition key & clustering column and want to query on non-primary key -> syntax (do we need to filter null in where ?)

- Replication factor(2-3)  -> easy

- ERD ( 2 q)
  - network is related to sensor (1:n)
  - question :
     - 1 n/w can have 1 or more sensors
     - 1 n/w can have 0 or more sensors
  - given ERD  : what is the key of the weak entity
    ans: dotted line


- UDT Address ( city ,state , pincode)
  - Q : city and pincode is already there , how to update state of an address UDT
    - syntax of updating UDT

- primary key understanding (2-3q)
  - where clause with partition key
  - where clause with partition + partial clustering key
  - where clause with primary key

- Q : select query with where cause containing only partition key / partition+ partition clustering key/ primary key
   - one row
   - subset of rows in one partition
   - all rows in one partition
   - multiple row in multiple partition


- Q:Given replication factor is unknown , which of the options woudl provide strong consistency or immediate consistency
  - quorum read, quorum write
  - all read, 1 write
  - 1 read. all write
  - Any read, all write


- 5 node DC-1  with RF = 3 ,4 node DC-2 with RF=2 , if one node goes down in each DC how many replicas would be there ?


- Recommended replication strategy for prod

- Definition of primary key

- question on quorum vs local_quorum

- question on uuid & timeuuid

- No question on read or write flow , UDT ,UDA
