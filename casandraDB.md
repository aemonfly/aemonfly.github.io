#Casandra DB throught

### Casandra Basic Idea

The starting point of casandra is that: disk is the cheapest compared with CPU, RAM. It will occupy a lot of cheap disk. 

Casandra write is really fast, it just write the current value into the DB even include delete data. Which means it has to read the all the write commits and merge them together if you do a query. And you will find the read operation is more and more slower if you write the data more and more times.



### DB Storage

Gernally speaking we can treat it as two levels of Map.

- The first level Map
	
	- key: the parition key.
	- value: the second level Map Pointer or Index in another word.
- The Second Level Map
	- key: the table field's.
	- value: the table field value.

Actually the casandra store the partition key in a separated key space and table. 

`system.schema_columnfamilies`

The partition key is used to compute which node will be use for operation (read or write). One of the big advantage casandra over traditional DB would be its columns is not fixed. Namely if one field has no value, it will not hold space. Think about the traditional DB which will allocated space no matter it has an value or  null vaule.





### DB Model

As for the guid for creating table, two basic rules:

- create table by query.
- create table by decreasing the number of row key (partition key) reading, it would be very good if the query only in one node.

As casandra has the advantage of fast writing speed, we don't need to care about the number of write. And
why casandra has the fast speed of writing based on the fact that a lot of duplicated data. So when do the model with casandra, you should firstly to make sure you have enough disk and don't care about how many time it writes.

When design casandra DB especially with table, we should make full use of the partition key to scatter the data evenly accross nodes(which form the cluster). Of course, you'd better read the partition key as few as possible to make casandra work more quickly.


General way to determine how many tables we need:

1. write down all the possible business queries.
2. try to merge some queries if possible.
3. create tables according to the queries.

And you may find there is data duplication accross the tables, they may have the same fields but with different partition keys. Well, it is normal. No worry, the disk is really cheap compared with your CPU, RAM and more importantly the response speed of query.



### DB Details

Casandra will use partition key to computer which node to do the operation, if two partition keys are equal, they go into the same phsical casandra node. For example, if you want to group student by grade.(Of course, you use the grade as the partition key). However, the different partition keys may goto the same phsical node due to the fact that the phsical nodes are limited. How does the casandra deal with such a case?




### Some Reference Resource.

[Primary Keys Of Casandra](http://thelastpickle.com/blog/2013/01/11/primary-keys-in-cql.html)

[Casandra Data Modeling Table](http://intellidzine.blogspot.co.uk/2013/11/cassandra-data-modelling-tables.html)

[Cassandra Data Modelling - Primary Keys](http://intellidzine.blogspot.com/2014/01/cassandra-data-modelling-primary-keys.html)

[Basic Rules of Cassandra Data Modeling](http://www.datastax.com/dev/blog/basic-rules-of-cassandra-data-modeling)

[The most important thing to know in Cassandra data modeling: The primary key](http://www.planetcassandra.org/blog/the-most-important-thing-to-know-in-cassandra-data-modeling-the-primary-key/)
