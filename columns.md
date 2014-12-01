#Columnar Stores

Of all the NoSQL families, Columnar Storage is the most RDBMS-ish. There are rows and there are columns. The database knows about column types. The key difference is how one has to relearn what a row is.

Columnar stores have the reputation of being obtuse. I think this stems from there general appearance. It took me multiple tries to understand how they actually work. My issue, and I think a lot of people's, is approaching a columnar store as a table store in the RDBMS sense. You really have to suspend this mapping.

Suspended? Good. Let's look at the system.

## Architecture
Suspending false equivalences is not the same as forgetting the prior model. Most RDBMS are row-centric. An entity is the table. A user is the `USER` table. 

Read and write operations occur against the entire row. 

Let's say you have a demographic table with 100 columns about your users. You want to know the average age of all the men that purchased an item within the last six months. You only need 3 columns of data: `GENDER`, `BIRTHDATE`, and `LAST_PURCHASED_DT`. Because it's row-centric, the database will actually read in all 100 columns, discard 97 and calculate against the three. That is a lot of wasted I/O.

Another implementation detail of row designs is that every row has to have 100 columns. This causes wasted space due to nulls[^oracle_nulls]. 

It also misrepresents the state of the data from a modeling perspective. A user might not provide a subset of fields. The fields make the user look incomplete. In reality the user is not incomplete, but a different type of user stuck in a fixed model.

Now that we've outlined the relational database storage model. Let's switch to the columnar model.

### Building Blocks
The smallest unit of storage is a column. 

A column has three pieces of data: column name, column value, and column timestamp. A value is typed like `INTEGER` or `STRING`. Of these the timestamp is the most interesting. The system determines the data age from timestamp. Age helps the system determine conflict resolution like the most recent write wins.

Columns are grouped into column families. A family has a row key. Depending on the actual system used, you can have either relatively static or dynamic columns within the family. 

By "relatively static" I mean that a column family can say, "I have these possible columns, but not every row in the family provides them all. They just can't add extras." Dynamic means just that: no fixed schema. Dynamic columns enable you to essentially create pre-calculated rows just as you would with a materialized view in RDBMS land. Let's say you have a project submission  request with comments. A dynamic row might have the submitted document, it's meta-data for the evaluation process and each comment by the reviewers represented by the dynamic columns where the column name is the reviewer id and the column value is the comment.

Super column families are like a view over multiple column families. If you need to analyze two or more column families at once, you should look at using super columns. **Quick Note:** super column families might be deprecated in some engines; RTFM before committing to a modeling paradigm.

### Storage and Distribution

Column families are written in contiguous chunks on the hard drive. This allows the database to optimize read operations by grabbing a large number of records per read cycle. 

Every system leverages replication at a column family level. When one defines a column family, the number of replicas is specified, or a default, like three, is used. 

At this point our friend the CAP theorem comes in. Some systems choose CP. Others will choose AP. Some systems even allow you dial up or down your needs between the two extremes. This is a quorum. Such systems allow you to poll the replicas to see what the common answer is. If two nodes say a row is "X, Y, Z" while a third says "X,C,B' and the quorum setting is 50% or more, the client will get a row of "X,Y,Z". 

The more nodes involved in the quorum, the slower the read operation because it has wait for each node's response. If the quorum is set to one node, that puts you squarely in the AP camp. If the quorum is set to all the nodes, you're in the C camp because you can't necessarily survive a network partition if one of your replicas is on the other side of the partition.


### Joins
There are no joins in such systems. Columns can point to other columns in other tables. It's up to the application to perform another query to get the information in the second table. At first blush this appears limiting. In practice, denormalization is your friend. Rather than storing information in another table, duplicate data into a column family. This gives you essentially materialized views in the row itself.

## Getting to Know the Players
Google's BigTable pushed this concept. They've used it to deal with multiple products like Google Earth and Google Finance[^bigtable_google]. Others have used it as well. It is part of the PaaS offerings by Google[^google_cloud_data]. None one has used it outside of the Google server environment. People liked the idea though, so they worked to replicate it.

### Cassandra
The homepage is http://cassandra.apache.org/. 

Cassandra is a top-level project from Apache by way of Facebook. It is living largely in the AP camp. In 2008 Facebook was looking for a way to build out its data collection infrastructure.  Their solution was a hybrid between BigTable and Amazon DynamoDB. A little while after creating the first pass of Cassandra, Facebook pretty much abandoned the project in favor of HBase. Fortunately, large companies like Apple and Netflix picked up where Facebook left off[^fb_abandon].

Cassandra is a complex, powerful storage system. It consistently wins performance shoutouts focused on I/O speed[^cassandra_shoutout]. It is eventually consistent. Quorums allow the developer to strike the right balance between consistency and speed.  Unlike other NoSQLs, there is no single point of failure in the entire system. It uses a gossip protocol to figure out the structure of the cluster. If its friends are unavailable, it learns to ignore them until contacted later. 

Below are some features that make it standout.
* Adjustable quorum size.
* Secondary indexes.
* True linear scaling (2 boxes run twice as fast as 1).
* Resiliency during node failures.


If your looking to a Bat Phone to a support company, never fear, DataStax[^datastax] is here. Recently called a Visionary by Gartner, they are the premier Cassandra consulting group.

### HBase

The homepage is http://hbase.apache.org/.

HBase is another Apache top-level project. Originally developed by Powerset, it's now used by multiple Fortune 500 companies. Notably Facebook built its messaging platform on it.

HBase sits atop Hadoop's HDFS. While Hadoop gets special treatment later in the book, now is a fine time to take a sneak peak. HDFS is Hadoop's distributed file system. HBase only writes data to HDFS (excluding the local drive). Under the covers HDFS takes care of replication and all of the other bookkeeping associated with tracking data.

Below are some features that make it standout. 
* Easier Hadoop integration.
* Bloom filters.
* Advanced in-memory operations.
* Simpler development model when compared to Cassandra.

If your looking to a Bat Phone to a support company, never fear, Hortonworks[^hortonworks] is here. Hortonworks has the largest number of contributors to the core Hadoop space than any other company. If you want to use HBase or any other to the fullest by working with a great partner, I know from personal experience Hortoworks is the way to go.

## So How Would We Use This?

## Sizing and Cost Considerations

## Further Resources

* Secondary indexes in Cassandra - http://www.wentnet.com/blog/?p=77
* HBase Official Free Book - http://hbase.apache.org/book/book.html

[^bigtable_google]: http://research.google.com/archive/bigtable.html
[^oracle_nulls]: https://community.oracle.com/thread/855964
[^google_cloud_data]: https://cloud.google.com/datastore/
[^fb_abandon]: https://www.facebook.com/note.php?note_id=454991608919
[^cassandra_shoutout]: http://planetcassandra.org/what-is-apache-cassandra/
[^datastax]: http://www.datastax.com/
[^hortonworks]: http://hortonworks.com/hadoop/hbase/