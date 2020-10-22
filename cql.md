

### Local setup

- Installation path

```
LJs-MBP:cassandra liju$ pwd
/Users/liju/Documents/softwares/cassandra

Set cassandra home
export CASSANDRA_HOME=/Users/liju/Documents/softwares/cassandra
```

- Start Cassandra

```
> LJs-MBP:cassandra liju$ cassandra
```

- Check status

```
LJs-MBP:cassandra liju$ nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
UN  127.0.0.1  218.44 KiB  256          100.0%            900dbd7c-1360-4fbd-a09d-57ca185983ba  rack1

```

- connect with cql

```
LJs-MBP:cassandra liju$ cqlsh
Connected to Test Cluster at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.11.5 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh>

```


### DDL

- keyspace

```
CREATE KEYSPACE IF NOT EXISTS cycling WITH REPLICATION =
{ 'class' : 'SimpleStrategy', 'replication_factor' : 1 };


use cycling;
```

- Table

  - Valid

```
CREATE TABLE cycling.cyclist_alt_stats ( id UUID PRIMARY KEY, lastname
text, birthday timestamp, nationality text, weight text, height text );

CREATE TABLE cycling.whimsey ( id UUID PRIMARY KEY, lastname text,
 cyclist_teams set<text>, events list<text>, teams map<int,text> );


CREATE TABLE cycling.route (race_id int, race_name text, point_id
 int, lat_long tuple<text, tuple<float,float>>, PRIMARY KEY (race_id,
 point_id));

CREATE TABLE cyclist_name ( id uuid primary key, lastname text, firstname text);


CREATE TABLE cycling.cyclist_category (
category text,
points int,
id UUID,
lastname text,
PRIMARY KEY (category, points)
) WITH CLUSTERING ORDER BY (points DESC);




```
   - Invalid

```

cqlsh:cycling> CREATE TABLE cyclist_name ( id uuid, lastname text, firstname text );
InvalidRequest: Error from server: code=2200 [Invalid query] message="No PRIMARY KEY specifed (exactly one required)"


```


- Tables & queries

```
CREATE TABLE cyclist_name ( firstname text, lastname text, age int, country text,gender text, rank smallint,
PRIMARY KEY ((firstname, lastname),gender,rank));

insert into cyclist_name(firstname, lastname, age, country, gender, rank) values ('fa', 'la',10,'IN','M',20);
insert into cyclist_name(firstname, lastname, age, country, gender, rank) values ('fb', 'lb',20,'US','M',21);
insert into cyclist_name(firstname, lastname, age, country, gender, rank) values ('fc', 'lc',30,'CH','M',22);
insert into cyclist_name(firstname, lastname, age, country, gender, rank) values ('fd', 'ld',40,'IN','F',23);
insert into cyclist_name(firstname, lastname, age, country, gender, rank) values ('fe', 'le',50,'US','F',24);
insert into cyclist_name(firstname, lastname, age, country, gender, rank) values ('ff', 'lf',60,'CH','F',25);
insert into cyclist_name(firstname, lastname, age, country, gender, rank) values ('fg', 'lg',70,'IN','M',26);
insert into cyclist_name(firstname, lastname, age, country, gender, rank) values ('fh', 'lh',80,'US','M',27);


Works
select * from cyclist_name;
select * from cyclist_name where firstname = 'fa' and lastname='la' ;

select * from cyclist_name where firstname = 'fa' and lastname='la' and gender = 'M';

select * from cyclist_name where firstname = 'fa' and lastname='la' and gender = 'M' and rank=20;
select * from cyclist_name where firstname = 'fa' and lastname='la' and rank=20 and gender = 'M';




Fails:

cqlsh:cycling> select * from cyclist_name where firstname = 'fa';
InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"

cqlsh:cycling> select * from cyclist_name where firstname = 'fa' and lastname='la' and country ='IN';
InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"

cqlsh:cycling> select * from cyclist_name where firstname = 'fa' and lastname='la' and rank = 20;
InvalidRequest: Error from server: code=2200 [Invalid query] message="PRIMARY KEY column "rank" cannot be restricted as preceding column "gender" is not restricted"

cqlsh:cycling> select * from cyclist_name where firstname = 'fa' and lastname='la' and gender = 'M' and country='IN';
InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"
```


- Counter Column
  -  Table containing counter column can have :
      - The primary key (can be one or more columns)
      - The counter column
      - Column(s) that serves as the primary key or partition key cannot be counter column
  - A counter column cannot be indexed or deleted
  - Column(s) that serves as the primary key or partition key cannot be counter column
  - Cassandra rejects USING TIMESTAMP or USING TTL when updating a counter column.
  - INSERT statements are not allowed on counter tables
  - To increase or decrease the value of the counter, use the UPDATE command


```
CREATE TABLE popular_count (
  id UUID PRIMARY KEY,
  popularity counter
  );

insert into popular_count (id,popularity) values (now(),10);

cqlsh:cycling> insert into popular_count (id,popularity) values (now(),10);
InvalidRequest: Error from server: code=2200 [Invalid query] message="INSERT statements are not allowed on counter tables, use UPDATE instead"

---
cqlsh:cycling> select * from popular_count;

 id | popularity
----+------------

(0 rows)

cqlsh:cycling> UPDATE popular_count
           ...  SET popularity = popularity + 10
           ...  WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b47;
cqlsh:cycling> select * from popular_count;

 id                                   | popularity
--------------------------------------+------------
 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b47 |         10

(1 rows)

```

- Materialized view
  -  a materialized view is a table that is built from another table's data with a new primary key and new properties
  - A materialized view automatically receives the updates from its source table.
  - Secondary indexes are suited for low cardinality data. Queries of high cardinality columns on secondary indexes require Cassandra to access all nodes in a cluster, causing high read latency.
  - Materialized views are suited for high cardinality data. The data in a materialized view is arranged serially based on the view's primary key.
  - Materialized views cause hotspots when low cardinality data is inserted.
  - Requirements for a materialized view:
    - The columns of the source table's primary key must be part of the materialized view's primary key
    - Only one new column can be added to the materialized view's primary key
    - Static columns are not allowed.
  - When another INSERT is executed on cyclist_mv, Cassandra updates the source table and both of these materialized views. When data is deleted from cyclist_mv, Cassandra deletes the same data from any related materialized views
  - Materialized views allow fast lookup of the data using the normal Cassandra read path
  - However, materialized views do not have the same write performance as normal table writes. Cassandra performs an additional read-before-write to update each materialized view.
  - The performance of deletes on the source table also suffers.
  - Cassandra can only write data directly to source tables, not to materialized views.Cassandra updates a materialized view asynchronously after inserting data into the source table, so the update of materialized view is delayed.
  - Cassandra performs a read repair to a materialized view only after updating the source table.

```

-- wrong
CREATE MATERIALIZED VIEW cyclist_name_mv_1
AS SELECT age , country
FROM  cyclist_name
WHERE age is not null and country is not null
PRIMARY KEY (age);


cqlsh:cycling> CREATE MATERIALIZED VIEW cyclist_name_mv_1
           ... AS SELECT age , country
           ... FROM  cyclist_name
           ... WHERE age is not null and country is not null
           ... PRIMARY KEY (age);
InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot create Materialized View cyclist_name_mv_1 without primary key columns from base cyclist_name (firstname,lastname,gender,rank)"

-- wrong

CREATE MATERIALIZED VIEW cyclist_name_mv_1
AS SELECT age , country
FROM  cyclist_name
WHERE age is not null and country is not null
PRIMARY KEY ((country,age), firstname, lastname, gender, rank);

cqlsh:cycling> CREATE MATERIALIZED VIEW cyclist_name_mv_1
           ... AS SELECT age , country
           ... FROM  cyclist_name
           ... WHERE age is not null and country is not null
           ... PRIMARY KEY ((country,age), firstname, lastname, gender, rank);
InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot include more than one non-primary key column 'age' in materialized view primary key"


-- wrong
CREATE MATERIALIZED VIEW cyclist_name_mv_1
AS SELECT age , country
FROM  cyclist_name
WHERE age is not null and country is not null
PRIMARY KEY (country, firstname, lastname, gender, rank);

cqlsh:cycling> CREATE MATERIALIZED VIEW cyclist_name_mv_1
           ... AS SELECT age , country
           ... FROM  cyclist_name
           ... WHERE age is not null and country is not null
           ... PRIMARY KEY (country, firstname, lastname, gender, rank);
InvalidRequest: Error from server: code=2200 [Invalid query] message="Primary key column 'firstname' is required to be filtered by 'IS NOT NULL'"


-- wrong
CREATE MATERIALIZED VIEW cyclist_name_mv_1
AS SELECT age , country
FROM  cyclist_name
WHERE firstname is not null and lastname is not null and country is not null and gender is not null and rank is not null
PRIMARY KEY (country, firstname, lastname, gender, rank);

-- correct
cqlsh:cycling> CREATE MATERIALIZED VIEW cyclist_name_mv_1
           ... AS SELECT age , country
           ... FROM  cyclist_name
           ... WHERE firstname is not null and lastname is not null and country is not null and gender is not null and rank is not null
           ... PRIMARY KEY (country, firstname, lastname, gender, rank);

Warnings :
Materialized views are experimental and are not recommended for production use.

cqlsh:cycling>


```


- Collection datatypes

  - set
    - unique elements
    - The values of a set are stored unordered, but will return the elements in sorted order when queried.

  - list
    - can contain duplicates
    - ordered
    - may be inserted or retrieved according to an index value.

  - map
    - key-value pair
    - For each key, only one value may exist, and duplicates cannot be stored

  - tuple
    - allow two or more values to be stored together in a column
    - for simple grouping , for complex grouping use UDT

```
CREATE TABLE cyclist_career_teams(id UUID PRIMARY KEY , lastname text, teams set<text> );

CREATE TABLE upcoming_calander(year int ,month int ,events list<text>, PRIMARY KEY(year,month));

CREATE TABLE cyclist_teams  (id UUDI PRIMARY KEY, lastname text , teams map<int, text>);

CREATE TABLE nation_rank (nation text PRIMARY KEY , info tuple<int,text,int>);


```

- UDT

- UDF

- UDA


- Static column
  - In a table that uses clustering columns, non-clustering columns can be declared static in the table definition.
  - Static columns are only static within a given partition.
  - A table that does not define any clustering columns cannot have a static column. The table having no clustering columns has a one-row partition in which every column is inherently static
  - A table defined with the COMPACT STORAGE directive cannot have a static column.
  - A column designated to be the partition key cannot be static.
