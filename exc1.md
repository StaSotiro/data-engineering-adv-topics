# Installing Impala on Docker (Mac)

For this assignment I ended up using Apache's installation of Impala on a Docker container, since I ran into major issues with networking and installation of other attempted versions. The installation of Impala I ended up using was apache/kudu, which [allows Impala's SQL syntax on Kudu tablets](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/kudu_impala.html). My installation was based on [information of this video](https://www.youtube.com/watch?v=CyjarHWEw8k&ab_channel=TechGranth). Docker image available at [docker/apache/impala](https://hub.docker.com/r/apache/impala). Docker presented serious issues with networking and allowing connection to containers, so I used version 4.5.0. This version does not fully resolve, but noticed a slight better behavior as [stated here](https://github.com/docker/for-win/issues/8861). 

I also ended up using docker cli to interact with the Impala instance, as connecting from outside the container's network was not possible. The command used to run and access Impala's shell was:

```
docker exec -it kudu-impala impala-shell
```



## Task 1

Impala is a highly optimised tool that allows fast query execution and metadata caching. It is designed with a very efficient execution framework at run-time, as it makes use of code generation, inter-process communication, massive support for parallelism. This makes it a great choise for something like a Data Mart, where specific business related queries are required and can be run ad-hoc at very low response times.

Hive is built for running complex queries, with heavy transformations and multiple joins. What Hive lacks in speed, it covers with fault tolerance, which allows queries to do not drop the execution in case e.g. a node fails at runtime. The ability to process complex queries make it a better option in something like a Data Warehouse where multiple dimensions and data types exist, and query orchestration and scheduling could be required.

Drill provides a different view of querying data, as it allows querying raw data sources without requiring prior metadata definition. This makes Drill a great choise for a prototyping phase, before proceding to a final implementation of a data warehouse, where discovery plays a big part. Drill is also a great choise for data science jobs, where various sources can be combined, without requiring strict definition and ETL workflows first in order to proceed to modeling and reporting.

Regarding data virtualisation, Drill is getting the closest to the main goal of not strictly defining a schema for our data, and the sources are more than one.

## Task 2

As mentioned above Drill allows us to put great work in connecting various sources in a metadata free 'schema'. Regarding our client's needs, there are three different paths depending the requirments.
1) If the client requires only high level reporting, then Drill is the best option, with low cost and quick implementation setup
2) If the client requires complex processing, metrics calculation etc. A DW would be a better option and for that Hive would work best.
3) If the client requires constant ad-hoc business specific questions to be answerd a DataMart would be the best way to move forward and Impala would work best in this scenario.



## Task 3

### Create statements

The proper table creations should be different, but my installation did not allow me to define Primary keys for some weird reason. I ended up simplyfying everything to fit the scope and be able to run queries on my machine. 

#### Original attempts example
```
create table student(
    sid int PRIMARY KEY,
    name string
) PARTITION BY HASH(sid) PARTITIONS 2
STORED AS KUDU;
;
```

#### Simplyfied queries

```
create table student(
    sid int,
    name string
);

create table courses(
    cid int,
    title string,
    description string
);

create table attended(
    sid int,
    cid int,
    year int,
    grade int
);
```

### Insert statements

```
insert into student values (1, 'stavros sotiropoulos') (2, 'dwayne johnson'), (3, 'John Cena');

insert into courses values (1, 'slam champs', 'the ultimate brawl'), (2, 'cage battles', 'fighting in cage'),  (3, 'Artificial Intelligence', 'AI courses')
;

insert into attended values (1, 1, 2020, 4), (1,1,2021,10), (1,3, 2020,4), (1,3,2021,5), (2,3,2021,5);
```


### Question 3
```
select name from student s
join attended a on a.sid = s.id
join courses c on c.cid = a.cid
where c.title like 'Artificial Intelligence' and a.year=2021;
```

### Question 4
```
with devData as (
    select sid, avg(grade)
    from attended 
    group by sid
)

select c.title, avg(a.grade)
from courses c
join attended a on a.cid = c.cid and a.grade > 5
join devData d on d.sid = a.sid
group by c.title
;
```
