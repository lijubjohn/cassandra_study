

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


- keyspace

```
CREATE KEYSPACE IF NOT EXISTS cycling WITH REPLICATION =
{ 'class' : 'SimpleStrategy', 'replication_factor' : 1 };


-- multi-dc
CREATE KEYSPACE cycling
  WITH REPLICATION = {
   'class' : 'NetworkTopologyStrategy',
   'boston'  : 3 , // Datacenter 1
   'seattle' : 2 , // Datacenter 2
   'tokyo'   : 2   // Datacenter 3
  };

-- with durable write disabled

CREATE KEYSPACE cycling
  WITH REPLICATION = {
   'class' : 'NetworkTopologyStrategy',
   'datacenter1' : 3
  }
AND DURABLE_WRITES = false ;


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


cqlsh:cycling> UPDATE popular_count SET popularity = 10 WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b47;
InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot set the value of counter column popularity (counters can only be incremented/decremented, not set)"


```

- Materialized view

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

```
CREATE [OR REPLACE] FUNCTION [IF NOT EXISTS]
[keyspace_name.]function_name (
    var_name var_type [,...] )
[CALLED | RETURNS NULL] ON NULL INPUT
RETURNS cql_data_type
LANGUAGE language_name AS
'code_block';

- example -
Overwrite or create the fLog function that computes the logarithm of an input value. CALLED ON NULL INPUT ensures that the function will always be executed.

CREATE OR REPLACE FUNCTION cycling.fLog (input double)
CALLED ON NULL INPUT
RETURNS double LANGUAGE java AS
'return Double.valueOf(Math.log(input.doubleValue()));';
```

- UDA

https://docs.datastax.com/en/cql-oss/3.3/cql/cql_reference/cqlCreateAggregate.html
```
CREATE [OR REPLACE] AGGREGATE [IF NOT EXISTS]
keyspace_name.aggregate_name ( cql_type )
SFUNC udf_name
STYPE cql_type
FINALFUNC udf_name
INITCOND [value]

```


- Select from local

```
SELECT toTimestamp(now()) FROM system.local;
SELECT now() FROM system.local;

```


- BATCH

```
BEGIN [ ( UNLOGGED | COUNTER ) ] BATCH
  [ USING TIMESTAMP [ epoch_microseconds ] ]
  dml_statement [ USING TIMESTAMP [ epoch_microseconds ] ] ;
  [ dml_statement [ USING TIMESTAMP [ epoch_microseconds ] ] [ ; ... ] ]
  APPLY BATCH ;

```

```
cqlsh> BEGIN BATCH
  INSERT INTO cycling.cyclist_expenses (cyclist_name, balance) VALUES ('Vera ADRIAN', 0) IF NOT EXISTS;
  INSERT INTO cycling.cyclist_expenses (cyclist_name, expense_id, amount, description, paid) VALUES ('Vera ADRIAN', 1, 7.95, 'Breakfast', false);
  APPLY BATCH;


-- Batching conditional updates
- Conditional batches cannot provide custom timestamps. UPDATE and DELETE statements within a conditional batch cannot use IN conditions to filter rows.

BEGIN BATCH
  INSERT INTO purchases (user, balance) VALUES ('user1', -8) IF NOT EXISTS;
  INSERT INTO purchases (user, expense_id, amount, description, paid)
    VALUES ('user1', 1, 8, 'burrito', false);
APPLY BATCH;

-- Batching counter updates

-- Counter batches cannot include non-counter columns in the DML statements, just as a non-counter batch cannot include counter columns. Counter batch statements cannot provide custom timestamps.

BEGIN COUNTER BATCH
  UPDATE UserActionCounts SET total = total + 2 WHERE keyalias = 523;
  UPDATE AdminActionCounts SET total = total + 2 WHERE keyalias = 701;
APPLY BATCH;

```

- Multiple partition logged batch

```
cqlsh> BEGIN LOGGED BATCH
INSERT INTO cycling.cyclist_names (cyclist_name, race_id) VALUES ('Vera ADRIAN', 100);
INSERT INTO cycling.cyclist_by_id (race_id, cyclist_name) VALUES (100, 'Vera ADRIAN');
APPLY BATCH;
```

- COPY

```
COPY table_name [( column_list )]
FROM 'file_name'[, 'file2_name', ...] | STDIN
[WITH option = 'value' [AND ...]]

COPY table_name [( column_list )]
TO 'file_name'[, 'file2_name', ...] | STDOUT
[WITH option = 'value' [AND ...]]


COPY cycling.cyclist_name (id,lastname)
TO '../cyclist_lastname.csv' WITH HEADER = TRUE ;


COPY cycling.cyclist_name (id,firstname)
FROM '../cyclist_firstname.csv' WITH HEADER = TRUE ;
```

- LIMIT

```

cqlsh> SELECT * From cycling.cyclist_name LIMIT 3;

cqlsh> SELECT * FROM cycling.rank_by_year_and_name PER PARTITION LIMIT 2;
```
